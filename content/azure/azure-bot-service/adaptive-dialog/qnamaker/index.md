---
title: "QnA Maker"
date: 2021-01-08T13:27:54+09:00
lastMod: 2021-03-17T20:48:03+09:00
---

## はじめに
参考：

* [BotBuilder-Samples/samples/csharp_dotnetcore/adaptive-dialog/07.qnamaker at main · microsoft/BotBuilder-Samples](https://github.com/microsoft/BotBuilder-Samples/tree/main/samples/csharp_dotnetcore/adaptive-dialog/07.qnamaker)

Adaptive Dialog で QnAMaker を使う方法を記載する。
MS のドキュメントに載っているのは、ボットのソースコードと一緒に `.qna` ファイルを作成し、Q&Aのリストを管理する方法だが、
ここに記載するのは、QnAMaker ポータルサイトに作成済みのナレッジベースを使う方法である。

## サンプル

### Dialog クラスのサンプル

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.AI.QnA;
using Microsoft.Bot.Builder.AI.QnA.Recognizers;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Adaptive;
using Microsoft.Bot.Builder.Dialogs.Adaptive.Actions;
using Microsoft.Bot.Builder.Dialogs.Adaptive.Conditions;
using Microsoft.Bot.Builder.Dialogs.Adaptive.Generators;
using Microsoft.Bot.Builder.Dialogs.Adaptive.Input;
using Microsoft.Bot.Builder.Dialogs.Adaptive.Templates;
using Microsoft.Bot.Builder.LanguageGeneration;
using Microsoft.Extensions.Configuration;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;

namespace FaqBotDemo.Dialogs
{
    public class RootDialog : AdaptiveDialog
    {
        public RootDialog(IConfiguration configuration) : base(nameof(RootDialog))
        {
            Recognizer = new QnAMakerRecognizer()
            {
                HostName = configuration["qna:hostname"],
                EndpointKey = configuration["qna:endpointKey"],
                KnowledgeBaseId = configuration["qna:knowledgeBaseId"],
                QnAId = "turn.qnaIdFromPrompt",
                IncludeDialogNameInMetadata = false
            };

            Triggers = new List<OnCondition>
            {
                new OnQnAMatch()
                {
                    Actions = new List<Dialog>()
                    {
                        new CodeAction(GetTopAnswer),
                        new SendActivity()
                        {
                            Activity = new ActivityTemplate("${SimpleAnswer()}"),
                        }
                    }
                },
                // 回答が複数あるものの、最大のスコアが0.8未満のとき
                new OnQnAMatch()
                {
                    Condition = "count(turn.recognized.answers) > 1 && count(where(turn.recognized.answers, answer, answer.score >= 0.8)) == 0",
                    Actions = new List<Dialog>()
                    {
                        // 回答リストを保存しておく
                        new SetProperty()
                        {
                            Property = "dialog.qnaAnswers",
                            Value = "=turn.recognized.answers"
                        },
                        // どの回答が合っているかユーザーへ尋ねる
                        new TextInput()
                        {
                            Prompt = new ActivityTemplate("${UnconfidentAnswer()}"),
                            Property = "turn.qnaUnconfidentResponse",
                            AllowInterruptions = false,
                            AlwaysPrompt = true
                        },
                        // ユーザーが選んだ質問を使って回答を探す
                        new SetProperty()
                        {
                            Property = "turn.qnaMatchFromAnswers",
                            Value = "=where(dialog.qnaAnswers, answer, answer.questions[0] == turn.qnaUnconfidentResponse)"
                        },
                        // 保存しておいた回答リストを削除する
                         new DeleteProperty()
                        {
                            Property = "dialog.qnaAnswers"
                        },
                        new IfCondition()
                        {
                            Condition = "turn.qnaMatchFromAnswers && count(turn.qnaMatchFromAnswers) > 0",
                            Actions = new List<Dialog>()
                            {
                                new SetProperty()
                                {
                                    Property = "turn.topAnswer",
                                    Value = "=turn.qnaMatchFromAnswers[0]"
                                },
                                new SendActivity("${SimpleAnswer()}")
                            },
                            // follow-up prompt で提示したもの以外が送られてきたとき
                            ElseActions = new List<Dialog>()
                            {
                                new SendActivity("お答えできませんでした。改めてご質問ください。")
                            }
                        }
                    }
                },
                // 回答に follow-up prompt が付いているとき
                new OnQnAMatch()
                {
                    Condition = "count(turn.recognized.answers[0].context.prompts) > 0",
                    Actions = new List<Dialog>()
                    {
                        new SetProperty()
                        {
                            Property = "dialog.qnaContext",
                            Value = "=turn.recognized.answers[0].context.prompts"
                        },
                        new TextInput()
                        {
                            Prompt = new ActivityTemplate("${PromptAnswer()}"),
                            Property = "turn.qnaMultiTurnResponse",
                            AllowInterruptions = false,
                            AlwaysPrompt = true
                        },
                        new SetProperty()
                        {
                            Property = "turn.qnaMatchFromContext",
                            Value = "=where(dialog.qnaContext, item, item.displayText == turn.qnaMultiTurnResponse)"
                        },
                        new DeleteProperty()
                        {
                            Property = "dialog.qnaContext"
                        },
                        new IfCondition()
                        {
                            Condition = "turn.qnaMatchFromContext && count(turn.qnaMatchFromContext) > 0",
                            Actions = new List<Dialog>()
                            {
                                new SetProperty()
                                {
                                    Property = "turn.qnaIdFromPrompt",
                                    Value = "=turn.qnaMatchFromContext[0].qnaId"
                                },
                                new EmitEvent()
                                {
                                    EventName = DialogEvents.ActivityReceived,
                                    EventValue = "=turn.activity"
                                }
                            },
                            // follow-up prompt で提示したもの以外が送られてきたとき
                            ElseActions = new List<Dialog>()
                            {
                                new SendActivity("お答えできませんでした。改めてご質問ください。")
                            }
                        }
                    }
                },
                new OnConversationUpdateActivity()
                {
                    Actions =
                    {
                        new CodeAction(WelcomeUser)
                    }
                },
                new OnUnknownIntent()
                {
                    Actions =
                    {
                        new SendActivity("回答が見つかりませんでした。")
                    }
                },
            };

            string[] paths = { ".", "Dialogs", "RootDialog.lg" };
            string fullPath = Path.Combine(paths);

            Generator = new TemplateEngineLanguageGenerator(Templates.ParseFile(fullPath));

        }

        /// <summary>
        /// QnAMaker から返ってきた回答のうち、一番スコアの高いものを「turn.topAnswer」に保存する
        /// </summary>
        /// <param name="dc"></param>
        /// <param name="options"></param>
        /// <returns></returns>
        private static async Task<DialogTurnResult> GetTopAnswer(DialogContext dc, object options)
        {
            var answers = (JArray) dc.State["turn.recognized.answers"];
            JToken topAnswer = null;
            float topScore = 0;

            foreach (var answer in answers)
            {
                var score = float.Parse(answer["score"].ToString());

                if (topAnswer == null || topScore < score)
                {
                    topScore = score;
                    topAnswer = answer;
                }
            }

            var turn = (JObject) dc.State["turn"];
            turn["topAnswer"] = topAnswer;

            return await dc.EndDialogAsync(options);
        }

        /// <summary>
        /// 新しいユーザーが追加されたときの処理
        /// </summary>
        /// <param name="dc"></param>
        /// <param name="options"></param>
        /// <returns></returns>
        private static async Task<DialogTurnResult> WelcomeUser(DialogContext dc, object options)
        {
            foreach (var member in dc.Context.Activity.MembersAdded)
            {
                if (member.Name != dc.Context.Activity.Recipient.Name)
                {
                    // はじめのメッセージを送信
                    var template = new ActivityTemplate("${WelcomeUser()}");
                    var message = await template.BindAsync(dc, dc.State);
                    await dc.Context.SendActivityAsync(message);
                }
            }
            return await dc.EndDialogAsync(options);
        }

        public override Task<DialogTurnResult> BeginDialogAsync(DialogContext dc, object options = null, CancellationToken cancellationToken = default)
        {
            // debug用
            var debug = dc.State["turn"];
            return base.BeginDialogAsync(dc, options, cancellationToken);
        }
    }
}
```

### lg ファイルのサンプル
GitHub のサンプルでは follow-up prompt に Suggested Action を使っているが、
Teams などでは対応していないため、Adaptive Card の Action を使っている。

````md
# SimpleAnswer
[Activity
    Attachments = ${json(SimpleAnswerCard())}
]

# SimpleAnswerCard
- ```
{
    "type": "AdaptiveCard",
    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
    "version": "1.2",
    "body": [
        {
            "type": "FactSet",
            "facts": [
                {
                    "title": "Q :",
                    "value": "${turn.topAnswer.questions[0]}"
                },
                {
                    "title": "A :",
                    "value": "${turn.topAnswer.answer}"
                }
            ]
        }
    ]
}
```

# PromptAnswer
[Activity
    Attachments = ${json(PromptAnswerCard())}
]

# PromptAnswerCard
- ```
{
    "type": "AdaptiveCard",
    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
    "version": "1.2",
    "body": [
        {
            "type": "TextBlock",
            "text": "${@answer}",
            "wrap": true
        },
        ${join(foreach(turn.recognized.answers[0].context.prompts, x, ActionButton(x.displayText)), ',')}
    ]
}
```

# UnconfidentAnswer
[Activity
    Attachments = ${json(UnconfidentAnswerCard())}
]

# UnconfidentAnswerCard
- ```
{
    "type": "AdaptiveCard",
    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
    "version": "1.2",
    "body": [
        {
            "type": "TextBlock",
            "text": "以下の回答が見つかりました。",
            "wrap": true
        },
        ${join(foreach(turn.recognized.answers, x, ActionButton(x.questions[0])), ',')}
    ]
}
```

# ActionButton (displayText)
- IF: ${turn.activity.channelId == 'msteams'}
    - ```
    {
        "type": "ActionSet",
        "actions": [
            {
                "type": "Action.Submit",
                "title": "${displayText}",
                "data": {
                "msteams": {
                    "type": "imBack",
                    "value": "${displayText}"
                    }
                }
            }
        ]
    }
    ```
- ELSE:
    - ```
    {
        "type": "ActionSet",
        "actions": [
            {
                "type": "Action.Submit",
                "title": "${displayText}",
                "data": "${displayText}"
            }
        ]
    }
    ```
# WelcomeUser
- ```
こんにちは。FAQチャットボットです。
```
````

## 回答の参照
`OnQnAMatch` トリガーが発生した後、QnAMaker から得た回答は以下で参照できる。

* `{@answer}` - 一番スコアの高い回答。文字データ。
* `turn.recognized.answers` - Microsoft.Bot.Builder.AI.QnA.QueryResult[] - QnAMaker が返した回答すべてが入っている。

* QueryResult
  * `questions` - string[]
  * `answer` - string
  * `score` - float
  * `metadata` - Microsoft.Bot.Builder.AI.QnA.Metadata[]
  * `source` - string
  * `id` - int - QnAId
  * `context` - Microsoft.Bot.Builder.AI.QnA.QnAResponseContext
    * `prompts` - Microsoft.Bot.Builder.AI.QnA.QnaMakerPrompt[]

* QnaMakerPrompt
  * `displayOrder` - int
  * `qnaId` - int
  * `displayText` - string
  * `qna` - object

型も一緒に記載したが、CodeAction などで直接 State を取り出す場合、JObject とか JArray とか、JSONの形式になっている。

## 注意点

`QnAMakerRecognizer` の生成時に、`IncludeDialogNameInMetadata` プロパティを `false` にしている。
これを忘れると既定値の `true` が適用され、QnAMaker 検索時に常にダイアログ名がメタデータフィルターにセットされる。
QnAMaker 側にメタデータの設定がない場合、QnAMaker が必ず 400 (Bad Request) を返す。
ボット側でエラーの原因を見つけるのは難しく、QnAMaker 作成時に一緒に作成した AppInsights のログを見てやっと気づけた。

AppInsights に記録されていたエラーメッセージ：

    Exception : Microsoft.CognitiveServices.QnAMaker.Runtime.Exceptions.AzureSearchBadStateException
    Message : Strict filter (dialogname) does not exist in the KB. Please retry with valid strict filters.


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

## QnAMakerRecognizer の追加
まず AdaptiveDialog の Recognizer に `QnAMakerRecognizer` を追加する。

```cs
public RootDialog(IConfiguration configuration) : base(nameof(RootDialog))
{
    Recognizer = new QnAMakerRecognizer()
    {
        HostName = configuration["qna:hostname"],
        EndpointKey = configuration["qna:endpointKey"],
        KnowledgeBaseId = configuration["qna:knowledgeBaseId"],
        QnAId = "turn.qnaIdFromPrompt", // follow-up prompt を使うなら指定が必要
        IncludeDialogNameInMetadata = false // Metadata を使わないなら必ずfalseにする
    };
}
```

HostName, EndpointKey, KnowledgeBaseId の3つは、QnAMaker ポータルサイトで当該ナレッジベースを選び、PUBLISH のページを開くと取得できる。

## OnQnAMatch の追加

### シンプルな回答をする
次に、Triggers に QnAMaker から回答を見つけられたときの処理を追加する。トリガーのクラスは `OnQnAMatch` を使う。下記は、シンプルに一番スコアの高い回答を返す処理のサンプル。

```cs
new OnQnAMatch()
{
    Actions = new List<Dialog>()
    {
        new SendActivity()
        {
            Activity = new ActivityTemplate("${@answer}"),
        }
    }
},
```

`${@answer}` には一番スコアの高い回答のテキストが入っている。

### follow-up prompt に対応する
follow-up prompt に対応する場合、シンプルな回答の処理に、もうひとつ `OnQnAMatch` トリガーを追加する。それぞれの Actions が何をやっているかをコメントに記載した。

```cs
// シンプルな回答を返す処理
new OnQnAMatch()
{
    Actions = new List<Dialog>()
    {
        new SendActivity()
        {
            Activity = new ActivityTemplate("${@answer}"),
        }
    }
},
new OnQnAMatch()
{
    // follow-up prompt がある場合、こちらのトリガーが実行される
    Condition = "count(turn.recognized.answers[0].context.prompts) > 0",
    Actions = new List<Dialog>()
    {
        // follow-up prompt のリストを dialog スコープに保存する
        new SetProperty()
        {
            Property = "dialog.qnaContext",
            Value = "=turn.recognized.answers[0].context.prompts"
        },
        // follow-up prompt をユーザーに見せ、いずれかを選ばせる
        new TextInput()
        {
            Prompt = new ActivityTemplate("${ShowMultiTurnAnswer()}"),
            Property = "turn.qnaMultiTurnResponse",
            AllowInterruptions = false,
            AlwaysPrompt = true
        },
        // ユーザーの選んだものが follow-up prompt のリストの中にあるかどうか調べ、見つかったものを turn スコープに保存する
        new SetProperty()
        {
            Property = "turn.qnaMatchFromContext",
            Value = "=where(dialog.qnaContext, item, item.displayText == turn.qnaMultiTurnResponse)"
        },
        // 最初のステップで保存した follow-up prompt のリストを削除
        new DeleteProperty()
        {
            Property = "dialog.qnaContext"
        },
        // ユーザーの選んだものがfollow-up prompt のリストにある場合
        new IfCondition()
        {
            Condition = "turn.qnaMatchFromContext && count(turn.qnaMatchFromContext) > 0",
            Actions = new List<Dialog>()
            {
                // 選んだ follow-up prompt の qnaId を turn スコープに保存する
                // ここで指定する Property は Recognizer のプロパティである qnaId に指定したもの
                new SetProperty()
                {
                    Property = "turn.qnaIdFromPrompt",
                    Value = "=turn.qnaMatchFromContext[0].qnaId"
                },
                // ActivityReceived のイベントをもう一度発生させ、qnaId を指定した状態でもう一度 QnAMaker に問い合わせを行う。
                // そうすると API は指定した qnaId の回答を返し、一連の処理が繰り返される。
                new EmitEvent()
                {
                    EventName = DialogEvents.ActivityReceived,
                    EventValue = "=turn.activity"
                }
            }
        }
    }
},
```


## Dialog クラスのサンプル

以下に RootDialog.cs のサンプルを記載する。

```cs
using AdaptiveExpressions.Properties;
using Microsoft.Bot.Builder;
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
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Threading.Tasks;

namespace AdaptiveDialogs.Dialogs
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
                QnAId = "turn.qnaIdFromPrompt", // follow-up prompt を使うなら指定が必要
                IncludeDialogNameInMetadata = false // Metadata を使わないなら必ずfalseにする
            };

            Triggers = new List<OnCondition>
            {
                new OnQnAMatch()
                {
                    Actions = new List<Dialog>()
                    {
                        new SendActivity()
                        {
                            Activity = new ActivityTemplate("${@answer}"),
                        }
                    }
                },
                new OnQnAMatch() // follow-up prompt を使うならこの部分が必要。
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
                            Prompt = new ActivityTemplate("${ShowMultiTurnAnswer()}"),
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
                            }
                        }
                    }
                },
                new OnUnknownIntent()
                {
                    Actions =
                    {
                        new SendActivity("nothing!")
                    }
                },
            };

            string[] paths = { ".", "Dialogs", "RootDialog.lg" };
            string fullPath = Path.Combine(paths);

            Generator = new TemplateEngineLanguageGenerator(Templates.ParseFile(fullPath));
        }
    }
}
```

加えて、RootDialog.lg ファイルに以下の定義を追加。

```md
# ShowMultiTurnAnswer
[Activity
    Text = ${@answer}
    SuggestedActions = ${foreach(turn.recognized.answers[0].context.prompts, x, x.displayText)}
]
```

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

一番スコアの高い回答の「質問」を参照したい場合などは、CodeAction を使う方がいいと思う。

## 注意点

`QnAMakerRecognizer` の生成時に、`IncludeDialogNameInMetadata` プロパティを `false` にしている。
これを忘れると既定値の `true` が適用され、QnAMaker 検索時に常にダイアログ名がメタデータフィルターにセットされる。
QnAMaker 側にメタデータの設定がない場合、QnAMaker が必ず 400 (Bad Request) を返す。
ボット側でエラーの原因を見つけるのは難しく、QnAMaker 作成時に一緒に作成した AppInsights のログを見てやっと気づけた。

AppInsights に記録されていたエラーメッセージ：

    Exception : Microsoft.CognitiveServices.QnAMaker.Runtime.Exceptions.AzureSearchBadStateException
    Message : Strict filter (dialogname) does not exist in the KB. Please retry with valid strict filters.

## Teams
Teams は SuggestedActions をサポートしていないので、代わりに AdaptiveCard を使う。
やっつけ気味だが、自分がたどり着いた実装方法を下記に記載する。

まず、RootDialog.cs の一部を上記サンプルから変更する。

```cs {hl_lines=[32]}
namespace AdaptiveDialogs.Dialogs
{
    public class RootDialog : AdaptiveDialog
    {
        public RootDialog(IConfiguration configuration) : base(nameof(RootDialog))
        {
            // 略
            Triggers = new List<OnCondition>
            {
                new OnQnAMatch()
                {
                    Actions = new List<Dialog>()
                    {
                        new SendActivity()
                        {
                            Activity = new ActivityTemplate("Here's what I have from QnA Maker - ${@answer}"),
                        }
                    }
                },
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
                            Prompt = new ActivityTemplate("${ShowMultiTurnAnswer2()}"),
                            Property = "turn.qnaMultiTurnResponse",
                            AllowInterruptions = false,
                            AlwaysPrompt = true
                        },
                        // 略
                    }
                },
                // 略
            };

            string[] paths = { ".", "Dialogs", "RootDialog.lg" };
            string fullPath = Path.Combine(paths);

            Generator = new TemplateEngineLanguageGenerator(Templates.ParseFile(fullPath));

        }
    }
}
```

RootDialog.lg に追加したテンプレートは下記の通り。

````markdown
# ShowMultiTurnAnswer2
[Activity
    Text = ${@answer}
    Attachments = ${json(ShowMultiTurnAnswer3())}
]

# ShowMultiTurnAnswer3
- ```
{
  "type": "AdaptiveCard",
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "version": "1.2",
  "actions": [
    ${join(foreach(turn.recognized.answers[0].context.prompts, x, ActionButton(x.displayText)), ',')}
  ]
}
```

# ActionButton (displayText)
- ```
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
```
````

follow-up prompt があるときだけ Adaptive Card になるのは違和感があるが、しかたなさそう。


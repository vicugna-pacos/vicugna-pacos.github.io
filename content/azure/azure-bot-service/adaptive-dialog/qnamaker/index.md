---
title: "QnA Maker"
date: 2021-01-08T13:27:54+09:00
---

## はじめに
参考：

* [BotBuilder-Samples/samples/csharp_dotnetcore/adaptive-dialog/07.qnamaker at main · microsoft/BotBuilder-Samples](https://github.com/microsoft/BotBuilder-Samples/tree/main/samples/csharp_dotnetcore/adaptive-dialog/07.qnamaker)

Adaptive Dialog で QnAMaker を使う方法を記載する。
MS のドキュメントに載っているのは、ボットのソースコードと一緒に `.qna` ファイルを作成し、Q&Aのリストを管理する方法だが、
ここに記載するのは、QnAMaker ポータルサイトに作成した Q&A のリストを使う方法である。

## サンプル

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
                // HostName, EndpointKey, KnowledgeBaseId は、従来の物と同じ
                HostName = configuration["qna:hostname"],
                EndpointKey = configuration["qna:endpointKey"],
                KnowledgeBaseId = configuration["qna:KnowledgeBaseId"],
                QnAId = "turn.qnaIdFromPrompt", // follow-up prompt を使うなら指定が必要
                IncludeDialogNameInMetadata = false
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


---
title: "Adaptive Dialog"
date: 2021-01-08T13:27:54+09:00
draft: true
---

## はじめに
参考：

* [Introduction to adaptive dialogs - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-adaptive-dialog-introduction)
* [Adaptive Dialog のサンプル (GitHubリポジトリ)](https://github.com/microsoft/BotBuilder-Samples/tree/main/samples/csharp_dotnetcore/adaptive-dialog)

ボットの会話は Dialog を使って実装するが、会話の複雑さが上がるほど中断とか分岐とか考慮しなくてはいけないことが増え、実装が大変になる。
それを解決するために新しく追加されたのが Adaptive Dialog らしい。
いままでの Dialog の実装方法と異なり、「宣言的」に定義できるのが特徴で、ソースコードでも実装できるが json ファイルに定義することもできる。
また、Bot Framework Composer も、この Adaptive Dialog の考え方を元にしているようにみえる。

## 前提条件

* 現在.NET版のBot Framework SDKでのみ使用可能。
* Bot Framework V4 SDK の Dialog の基礎知識
* Bot Framework V4 SDK の Prompt の基礎知識

## Adaptive Dialog の構造

### Trigger
参考：

* [Events and triggers in adaptive dialogs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-adaptive-dialog-triggers)
  * Trigger についての説明が載っている
* [Events and triggers in adaptive dialogs - reference guide - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/adaptive-dialog/adaptive-dialog-prebuilt-triggers)
  * あらかじめ定義されている Trigger の説明が載っている

Adaptive Dialog には、Trigger と言われるイベントハンドラのリストを定義する。
Trigger には Condition (条件) と Action (処理) を定義する。
Trigger のリストを上から順にたどっていって、Condition が合っていると Action が実行され、それ以降の Action は実行されない。
なので、リストの最後には、どの条件にも引っかからなかった場合の処理を用意しておくのが良いっぽい。
下記サンプルの `OnUnknownIntent` の部分がそれにあたる。

```cs
public RootDialog() : base(nameof(RootDialog))
{
    Triggers = new List<OnCondition>
    {
        new OnConversationUpdateActivity()
        {
            Actions = WelcomeUserSteps()
        },
        new OnUnknownIntent
        {
            Actions =
            {
                new SendActivity("Hi, we are up and running!!"),
            }
        },
    };
}
```

### Action
Action は、Trigger の条件が合致したときに行う処理を定義する。
それぞれのステップがメソッドとなっている Waterfall Dialog とは違い、Adaptive dialog の Action は Dialog クラスを拡張したもののリストである。
これは Adaptive Dialog をより強力で柔軟にしつつ、中断や分岐などの管理を容易にする。

Bot Framework SDKは、たくさんのビルトインActionを提供している。例えば、メモリの生成、ダイアログ管理、会話フローの制御などである。
Actionは拡張可能で、自分のカスタム Action を作成できる。

Actionの詳細は、[Actions in adaptive dialogs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-adaptive-dialog-actions) を参照。

### Input
Input は Adaptive Dialog 用の Prompt である。Input は、ユーザーへ情報をリクエスト＆検証するために、Adaptive Dialog で使える特別なActionである。
すべての Input クラスは、下記の機能がある。

* ユーザーへプロンプトする前に、その情報をボットがすでに持っているかどうかをチェックする。
* 入力が想定通りの形式である場合に、指定されたプロパティへ値を保存する。
* 制約を使える。最小、最大など。

Inputの詳細は、[Asking for user input using adaptive dialogs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-adaptive-dialog-inputs) を参照。

[Inputs in adaptive dialogs - reference guide - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/adaptive-dialog/adaptive-dialog-prebuilt-inputs)

### Recognizer

* [Recognizers in adaptive dialogs - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-adaptive-dialog-recognizers)

Recognizerは、ユーザーの入力を解析してくれる。
例えば、LUIS を使ってインテントが見つかるかどうか検証したり、QnA Maker に答えられる質問かどうかを検証したりする。

Recognizer には下記の種類がある：

* __RegexRecognizer__ - 正規表現を使い、ユーザーの入力からインテントを見つける。
* __LUIS recognizer__ - LUIS を使い、ユーザーの入力からインテントを見つける。Azure の同サービスが必要。
* __QnA Maker recognizer__ - QnA Maker を使い、ユーザーの入力が QnA Maker に答えられる質問かどうかを検証する。
* __Multi-language recognizer__ - 言語ごとに Recognizer を指定できる。
* __CrossTrained recognizer set__ - 複数の Recognizer の設定を学習し、解析したりしてくれるもの(？)
* __RecognizerSet__ - 複数の Recognizer を使うときに使う。

### Generator
実装編で解説。

### メモリのスコープと State の管理
参考：

* [Managing state in Adaptive Dialogs - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-adaptive-dialog-memory-states)
  * 説明
* [Managing state in adaptive dialogs - reference guide - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/adaptive-dialog/adaptive-dialog-prebuilt-memory-states)
  * スコープの一覧と使用例が載っている

Adaptive Dialog では、State への読み書きも簡易になっている。State にはいくつかのスコープがあり、User State であれば `user.xxxx` といった具合にアクセスできる。
以下にスコープの一覧を示す：

* __User scope__ (`user`) - 会話に参加したユーザーごと。
* __Conversation scope__ (`conversation`) - 会話ごと。
* __Dialog scope__ (`dialog`) - ダイアログごと。そのダイアログのが終わるとメモリが消えるかもしれない(未確認)。
* __Turn scope__ (`turn`) - ターンごと。
* __Settings scope__ (`settings`) - 設定値。appsettings.json とかの設定値を取得できる。コード上で書き込むことは少ないと思われる。
* __This scope__ (`this`) - その Action クラスのプロパティを取得できる。Input アクションを使用するときによく使ったりするらしい。
* __Class scope__ (`class`) - アクティブなダイアログのプロパティを取得できる。

### Declarative assets
Adaptive Dialog を拡張して、自分でカスタマイズしたdialogクラスを作成できるが、それとは別に、JSONファイルを作って Adaptive dialogを拡張できる。
ソースコードは必要なく、両方のアプローチを一つのボットに入れられる。

[Using declarative assets](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-adaptive-dialog-declarative?view=azure-bot-service-4.0)

## 実装
ドキュメント：https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-dialogs-adaptive
サンプルソース：https://github.com/microsoft/BotBuilder-Samples/tree/main/samples/csharp_dotnetcore/adaptive-dialog/01.multi-turn-prompt

## プロジェクトのセットアップ
[Create a bot project for adaptive dialogs - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-adaptive-dialog-setup)

Empty Bot のテンプレートプロジェクトを元に、Adaptive Dialog を使えるようにする手順を示す。

### NuGet パッケージの追加

`Microsoft.Bot.Builder.Dialogs.Adaptive` を追加する。

必要に応じて下記のパッケージを追加する：

* Adaptive Dialog の単体テスト： `Microsoft.Bot.Builder.Dialogs.Adaptive.Testing`
* LUIS：`Microsoft.Bot.Builder.AI.LUIS`
* QnA Maker language understanding： `Microsoft.Bot.Builder.AI.QnA`

### コンポーネントの登録
Startup.cs の `ConfigureServices` メソッドに、下記を追加してコンポーネントの登録を行う。

必須：

```cs
ComponentRegistration.Add(new AdaptiveComponentRegistration()); // Components common to all adaptive dialogs.
ComponentRegistration.Add(new DialogsComponentRegistration()); // Common memory scopes and path resolvers.
```

任意：

```cs
ComponentRegistration.Add(new AdaptiveTestingComponentRegistration()); // Components used to unit test adaptive dialogs.
ComponentRegistration.Add(new DeclarativeComponentRegistration()); // Components used to consume declarative dialogs.
ComponentRegistration.Add(new LanguageGenerationComponentRegistration()); // Components used for language generation features.
ComponentRegistration.Add(new LuisComponentRegistration()); // Components used for LUIS (language understanding) features.
ComponentRegistration.Add(new QnAMakerComponentRegistration()); // Components used for QnA Maker (language understanding) features.
ComponentRegistration.Add(new TeamsComponentRegistration()); // Components specific to the Teams channel.
```

### State の追加
Startup.cs に UserState と ConversationState を追加する。

```cs
services.AddSingleton<IStorage, MemoryStorage>();
services.AddSingleton<UserState>();
services.AddSingleton<ConversationState>();
```

次に、AdapterWithErrorHandler.cs に下記を追加する。

```cs {hl_lines=[4,7,8,9]}
public class AdapterWithErrorHandler : BotFrameworkHttpAdapter
{
    public AdapterWithErrorHandler(IConfiguration configuration, ILogger<BotFrameworkHttpAdapter> logger
        , IStorage storage, UserState userState, ConversationState conversationState)
        : base(configuration, logger)
    {
        this.UseStorage(storage);
        this.UseBotState(userState);
        this.UseBotState(conversationState);

        OnTurnError = async (turnContext, exception) =>
        {
            // 略
        };
    }
}
```

これを追加すると、turnContext から Storage と State を参照できるようになる。これがないと、DialogManager が動作しない。

### RootDialog の追加
Adaptive Dialog となる RootDialog を作成する。例では Dialogs/RootDialog.cs に作成した。
サンプルは下記の通り。

```cs
using Microsoft.Bot.Builder.Dialogs.Adaptive;
using Microsoft.Bot.Builder.Dialogs.Adaptive.Actions;
using Microsoft.Bot.Builder.Dialogs.Adaptive.Conditions;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace AdaptiveDialogs.Dialogs
{
    public class RootDialog : AdaptiveDialog
    {
        public RootDialog() : base(nameof(RootDialog))
        {
            Triggers = new List<OnCondition>
            {
                new OnUnknownIntent
                {
                    Actions =
                    {
                        new SendActivity("nothing!"),
                    }
                },
            };
        }
    }
}
```

RootDialog を作成したら、Startup.cs のDIに登録する。

```cs
services.AddSingleton<RootDialog>();
```

### ボットの編集
前の手順で作成した RootDialog を、DialogManager を介してボットから呼び出す。以下にボットクラスのサンプルを示す。

```cs
using AdaptiveDialogs.Dialogs;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;

namespace AdaptiveDialogs
{
    public class EmptyBot : ActivityHandler
    {
        private readonly DialogManager _dialogManager;

        public EmptyBot(RootDialog rootDialog)
        {
            _dialogManager = new DialogManager(rootDialog);
        }

        public override async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default)
        {
            await _dialogManager.OnTurnAsync(turnContext, cancellationToken: cancellationToken).ConfigureAwait(false);
        }
    }
}
```

### テストしてみる
ボットを起動し Bot Framework Emulator で接続して、何かメッセージを送信する。
ボットから、「nothing!」と返事が来ればOK。

## Welcome メッセージを追加する
ボットから DialogManager を呼び出す処理を `OnTurnAsync` メソッドに書いたことから分かるように、Adaptive Dialog は全てのイベント、アクティビティに対して処理できる。
ユーザーがボットに初めて接続したときの `OnMembersAddedAsync` メソッドの処理も Adaptive Dialog へ移せる。

サンプルその1を以下に示す。

```cs
public class RootDialog : AdaptiveDialog
{
    public RootDialog() : base(nameof(RootDialog))
    {
        Triggers = new List<OnCondition>
        {
            new OnConversationUpdateActivity()
            {
                Actions = WelcomeUserSteps()
            },
            new OnUnknownIntent
            {
                // 略
            },
        };
    }

    private static List<Dialog> WelcomeUserSteps()
    {
        return new List<Dialog>()
        {
            new Foreach()
            {
                ItemsProperty = "turn.activity.membersAdded",
                Actions = new List<Dialog>()
                {
                    new IfCondition()
                    {
                        Condition = "$foreach.value.name != turn.activity.recipient.name",
                        Actions = new List<Dialog>()
                        {
                            new SendActivity("Hello!")
                        }
                    }
                }
            }
        };
    }
}
```

`WelcomeUserSteps` メソッドの内容が Action にあたる。Foreach とか If まで Action クラスになっていて、それらをネストしたりしながら使っていく。
ただこれだとかえって何をやっているかわかりづらいし、ちょっとしたロジックなのに煩雑にみえる。
そういう場合、`CodeAction` を使うと良い。

```cs
public class RootDialog : AdaptiveDialog
{
    public RootDialog() : base(nameof(RootDialog))
    {
        Triggers = new List<OnCondition>
        {
            new OnConversationUpdateActivity()
            {
                Actions = 
                {
                    new CodeAction(WelcomeUser)
                }
            },
            new OnUnknownIntent
            {
                // 略
            },
        };
    }

    private static async Task<DialogTurnResult> WelcomeUser(DialogContext dc, object options)
    {
        foreach (var member in dc.Context.Activity.MembersAdded)
        {
            if (member.Name != dc.Context.Activity.Recipient.Name)
            {
                var message = MessageFactory.Text("Hello!");
                await dc.Context.SendActivityAsync(message);
            }
        }
        return await dc.EndDialogAsync(options);
    }
}
```

## Language Generator

参考：
* [Generators in adaptive dialogs - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-adaptive-dialog-generators)
  * Language Generator とは、の説明が載っている
* [.lg file format - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/file-format/bot-builder-lg-file-format)
  * lg ファイルの基本的な書き方が載っている
* [Adaptive expressions prebuilt functions - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/adaptive-expressions/adaptive-expressions-prebuilt-functions)
  * lg ファイル内で利用できる関数が載っている

Language Generation を使うと、ボットのセリフをリソースファイルに隔離して管理できる。
リソースファイルには条件式の指定やStateの参照が可能で、より自然なボットのセリフを構築できる。

リソースファイルの拡張子は `.lg` で、通常は利用する Dialog クラスと1対1で作成し、Dialog クラスと同じフォルダに配置する。
lgファイルのプロパティを変更し、「出力ディレクトリにコピー」するようにしておくこと。
文字コードは、UTF-8-BOM にする(BOM無しでも大丈夫っぽい)。

サンプルとして、 Dialogs/RootDialog.lg を作成した。

```markdown
# Nothing
- nothing1!
- nothing2!
```

次に RootDialog をLGを使うように変更する。

```cs {hl_lines=[18,"23-26"]}
public class RootDialog : AdaptiveDialog
{
    public RootDialog() : base(nameof(RootDialog))
    {
        Triggers = new List<OnCondition>
        {
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
                    new SendActivity("${Nothing()}")
                }
            },
        };

        string[] paths = { ".", "Dialogs", "RootDialog.lg" };
        string fullPath = Path.Combine(paths);

        Generator = new TemplateEngineLanguageGenerator(Templates.ParseFile(fullPath));
    }
    // 略
}
```

次に Startup.cs を開き、`ConfigureServices` メソッドにコンポーネントの登録を追加する。
(この記述がなくても動作したが…念のため)

```cs
ComponentRegistration.Add(new LanguageGenerationComponentRegistration()); // Components used for language generation features.
```

これでボットをテストすると、「nothing1!」か「nothing2!」のいずれかを返すようになる。

### CodeAction 内でLGを使う
Adaptive Dialog の Action 類からLGを使うときは `${テンプレート名()}` で良いが、自分で Action の内容を実装したときにLGを使う方法を示す。

```cs
private static async Task<DialogTurnResult> OriginalAction(DialogContext dc, object options)
{
    var template = new ActivityTemplate("${Greeting()}");
    var message = await template.BindAsync(dc, dc.State);
    await dc.Context.SendActivityAsync(message);
}
```

### 条件式や関数
LGファイルに以下のような定義を追加し、時間帯に合わせて挨拶を変えることができたりする。

```md
# timeOfDay
- ${getTimeOfDay(convertFromUTC(utcNow(), 'Asia/Tokyo'))}

# Greeting
- IF: ${timeOfDay() == 'morning'}
  - おはよう！
- ELSEIF: ${timeOfDay() == 'afternoon'}
  - こんにちは！
- ELSE:
  - こんばんは！
```

Dialog の方で `${Greeting()}` を参照すれば、朝なら「おはよう」、昼なら「こんにちは」と喋らせることができる。
`Greeting` のIF文で `timeOfDay` を参照しているように、LGのあるテンプレートから他のテンプレートを参照することが可能。
変数を定義＆参照するような感覚で使える。

## QnAMaker を使う
参考：

* [BotBuilder-Samples/samples/csharp_dotnetcore/adaptive-dialog/07.qnamaker at main · microsoft/BotBuilder-Samples](https://github.com/microsoft/BotBuilder-Samples/tree/main/samples/csharp_dotnetcore/adaptive-dialog/07.qnamaker)

Adaptive Dialog で QnAMaker を使う方法を記載する。
MS のドキュメントに載っているのは、ボットのソースコードと一緒に `.qna` ファイルを作成し、Q&Aのリストを管理する方法だが、
ここに記載するのは、QnAMaker ポータルサイトに作成した Q&A のリストを使う方法である。

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

### 注意点

`QnAMakerRecognizer` の生成時に、`IncludeDialogNameInMetadata` プロパティを `false` にしている。
これを忘れると既定値の `true` が適用され、QnAMaker 検索時に常にダイアログ名がメタデータフィルターにセットされる。
QnAMaker 側にメタデータの設定がない場合、QnAMaker が必ず 400 (Bad Request) を返す。
ボット側でエラーの原因を見つけるのは難しく、QnAMaker 作成時に一緒に作成した AppInsights のログを見てやっと気づけた。

AppInsights に記録されていたエラーメッセージ：

    Exception : Microsoft.CognitiveServices.QnAMaker.Runtime.Exceptions.AzureSearchBadStateException
    Message : Strict filter (dialogname) does not exist in the KB. Please retry with valid strict filters.

### Teams
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

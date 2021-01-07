---
title: "Adaptive Dialog"
date: 2020-11-12T10:02:49+09:00
draft: true
---

## はじめに
参考：[Introduction to adaptive dialogs - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-adaptive-dialog-introduction)

ボットの会話は Dialog を使って実装するが、会話の複雑さが上がるほど実装も大変になっていた。
それを解決するために新しく追加されたのが Adaptive Dialog である。

重要：  
Adaptive Dialog は、現在.NET版のBot Framework SDKでのみ使用可能。Adaptive Dialogを使ったサンプルソースは [GitHubリポジトリ](https://github.com/microsoft/botbuilder-samples/tree/master/samples/csharp_dotnetcore) にある。


## 前提条件

* Bot Framework V4 SDK の Dialog の基礎知識
* Bot Framework V4 SDK の Prompt の基礎知識

## Adaptive Dialog とは

### なぜ adaptive dialog なのか
Adaptive dialog は WaterfallDialog に対してたくさんのアドバンテージがある。主な特徴は下記の通り。

* 会話のフローをコンテキストやイベントによって動的に更新できる柔軟性を提供する。会話の途中でコンテキストが切り替わったり中断される場合に便利。
* Dialog のイベントシステムをサポートする。中断、キャンセル、実行など。
* input recognition とルールベースのイベントハンドリングを提供する。
* 会話モデル(Dialog)と出力生成を一つにまとめる。
* 解析、イベントルール、機械学習の拡張機能を提供する。
* 開始から宣言的に設計された。Bot Framework Composerからツールとして使用できる。

## Adaptive dialog の構造

### Trigger
すべての Adaptive Dialog は、Triggerと言われるイベントハンドラのリストを持っている。そして、トリガーは `Condition` と `Action`のリストを持っている。
Triggerはイベントのキャッチとレスポンスを可能にする。Conditionが満たされるとActionが実行され、それ以降のActionは実行されない。
もしConditionが満たされない場合、イベントは次のイベントハンドラへ処理を渡す。

トリガーについての詳細は、[Events and triggers in adaptive dialogs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-adaptive-dialog-triggers?view=azure-bot-service-4.0)を参照。

### Action
Actionは、特定のイベントがTrigger経由でキャプチャーされたときの会話のフローを定義する。
それぞれのステップが関数となっているWaterfall dialogとは違い、Adaptive dialogのActionはそれ自体がダイアログである。
これはAdaptive dialogをよりパワフルでフレキシブルにしつつ、中断や分岐の状態の管理を容易にする。

Bot Framework SDKは、たくさんのビルトインActionを提供する。例えば、メモリの生成、ダイアログ管理、会話フローの制御などである。
Actionは拡張可能で、自分のカスタムActionを作成できる。

Actionの詳細は、[Actions in adaptive dialogs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-adaptive-dialog-actions?view=azure-bot-service-4.0) を参照。

### Input
Input はAdaptive dialogへの入力であり、PromptはDialogの基底クラスへのPromptである。
Input は、ユーザーへ情報をリクエスト＆検証するために、Adaptive Dialogで使える特別なActionである。
すべての input クラスは、下記の機能がある。

* ユーザーへプロンプトする前に、その情報をボットがすでに持っているかどうかをチェックする。
* 入力が想定通りの形式である場合に、指定されたプロパティへ値を保存する。
* 制約を使える。最小、最大など。

Inputの詳細は、[Asking for user input using adaptive dialogs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-adaptive-dialog-inputs?view=azure-bot-service-4.0) を参照。

### Recognizer
Recognizerは、ボットに、ユーザーの入力を理解し、意味のある断片へ拡張することを可能にする。
すべての Recognizer は、ユーザーの発話からintentを取り出した際に`recognizedIntent`イベントを送出する。
Recognizer の利用は必須ではないが、`recognizedIntent`イベントのかわりに`unknownIntent`が発生するようになる。

Recognizer の詳細は、[Recognizers in adaptive dialogs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-adaptive-dialog-recognizers?view=azure-bot-service-4.0) を参照。

### Generator
Generator は、特定の言語生成システムをAdaptive Dialog に結びつける。
これは、Recognizer とともにダイアログの言語理解と言語生成のクリーンな分離とカプセル化を可能にする。
言語生成機能を使用すると、generator を.lgファイルに関連付けるか、Generator をTemplateEngineインスタンスに設定して、アダプティブダイアログを強化する1つ以上の.lgファイルを明示的に管理できます。

詳細は、[Language Generation in adaptive dialogs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-adaptive-dialog-generators?view=azure-bot-service-4.0) を参照。

### Memory scopes and managing state
アダプティブダイアログは、メモリにアクセスして管理する方法を提供します。すべてのアダプティブダイアログはデフォルトでこのモデルを使用するため、メモリを消費または寄与するすべてのコンポーネントには、適切なスコープで情報を読み書きするための共通の方法があります。すべてのスコープのすべてのプロパティはプロパティバッグであり、保存されているプロパティを動的に変更できます。

[Memory scopes and managing state in adaptive dialogs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-adaptive-dialog-memory-states?view=azure-bot-service-4.0)

### Declarative assets
Adaptive Dialog を拡張して、自分でカスタマイズしたdialogクラスを作成できるが、それとは別に、JSONファイルを作って Adaptive dialogを拡張できる。
ソースコードは必要なく、両方のアプローチを一つのボットに入れられる。

[Using declarative assets](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-adaptive-dialog-declarative?view=azure-bot-service-4.0)

## すべてをまとめる

### The adaptive dialog runtime behavior
下記の架空の旅行代理店ボットのサンプルは、Adaptive Dialogのふるまいを表している。
現実世界のアプリケーションは、複数の機能がある。例えば飛行機の座席やホテルの空室、レンタカーを検索したり、天気までチェックできたりする。

もし、ユーザーがボットとの会話の途中で予期しない行動をしたら何が起こるか？

下記のシナリオを例としてみる：

    User: I’d like to book a flight
    Bot:  Sure. What is your destination city?
    User: How’s the weather in Seattle?
    Bot:  Its 72 and sunny in Seattle
    ...

ユーザーは質問に答えなかっただけでなく、話題を完全に変えている。これを実行するには、別のdialogに存在する別のコード(Action)が必要になる。
Adaptive dialogは、このシナリオの制御を可能にする。

Adaptive_dialog_runtime_behavior

このボットは下記3つのadaptive dialogを持っている。

1. `rootDialog` 自身のLUISモデルとトリガーとアクションのセットを持っている。一部のActionは、特定のリクエスト用に設計された子dialogを呼び出す。
1. `bookFlightDialog` 自身のLUISモデルとトリガーとアクションのセットを持っている。飛行機の予約を扱うdialog。
1. `weatherDialog` 自身のLUISモデルとトリガーとアクションのセットを持っている。天気予報を教えてくれるdialog。

ユーザーが `I'd like to book a flight` と言ったときのフローを示す。

Adaptive_dialog_conversation_flow_example

rootDialog の recognizer が `recognizedIntent` イベントを放出する。このイベントは `OnIntent` トリガーでキャッチできる。
このケースでは、ユーザーが言った　"I’d like to book a flight"　が rootDialogで定義した intentにマッチしたため、bookFlightDialog を呼び出す `BeginDialog`アクションを持った OnIntent トリガーが発生した。
bookFlightDialog は、ユーザーの目的地の都市を質問するという自身のactionを実行する。

ユーザーはどんな返答もできるため、時にはボットの質問と何の関係も無いような返答をすることがある。例えば、 "How's the weather in Seattle?" である。

Adaptive_dialog_interruption_example

ユーザーリクエストに対して、`bookFlightDialog`の`OnIntent`トリガーに引っかからない場合、ボットは会話スタックを、root dialogめがけて上へさかのぼる。そして、`rootDialog`に定義した天気の`OnIntent`トリガーに引っかかったため、`weatherDialog`の`BeginDialog`アクションを実行する。
`weatherDialog`のレスポンスが終わったとき、ボットは元々処理していたdialogに会話フローを戻す。つまり`bookFlightDialog`にもどり、ふたたびユーザーに目的地の都市を聞く。

まとめ:

それぞれのdialogのrecognizerがユーザーの入力を解析し、インテントを求める。
インテントが定まったとき、recognizerは`IntentRecognized`イベントを送出する。このイベントはdialogのOnIntentトリガーとなる。
もしアクティブなdialogに、このintentを処理できるOnIntentトリガーがない場合、ボットはそのdialogの親dialogへインテントを投げる。
もし親dialogにもintentを処理できるtriggerを持っていない場合、intentはroot dialogまでさかのぼる。
triggerがintentを処理し終えると、元々処理していたdialogへ処理が戻る。

## 実装
ドキュメント：https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-dialogs-adaptive
サンプルソース：https://github.com/microsoft/BotBuilder-Samples/tree/main/samples/csharp_dotnetcore/adaptive-dialog/01.multi-turn-prompt

## プロジェクトのセットアップ
[Create a bot project for adaptive dialogs - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-adaptive-dialog-setup)

Empty Bot のテンプレートから Adaptive Dialog を使えるようにする手順を示す。

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

For example, the adaptive multi-turn prompts sample registers these components in ConfigureServices.

```cs
// Register dialog. This sets up memory paths for adaptive.
ComponentRegistration.Add(new DialogsComponentRegistration());

// Register adaptive component
ComponentRegistration.Add(new AdaptiveComponentRegistration());

// Register to use language generation.
ComponentRegistration.Add(new LanguageGenerationComponentRegistration());
```

## State の追加
Startup.cs に UserState と ConversationState を追加する。

```cs
// Create the storage we'll be using for User and Conversation state. (Memory is great for testing purposes.) 
services.AddSingleton<IStorage, MemoryStorage>();

// Create the User state. (Used in this bot's Dialog implementation.)
services.AddSingleton<UserState>();

// Create the Conversation state. (Used by the Dialog system itself.)
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

## RootDialog の追加
Adaptive Dialog の基底となる RootDialog を作成する。例では Dialogs\RootDialog.cs に作成した。
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
                        new SendActivity("Hi, we are up and running!!"),
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

## ボットの編集
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

## テストしてみる
ボットを起動し Bot Framework Emulator で接続して、何かメッセージを送信する。
ボットから、「Hi, we are up and running!!」と返事が来ればOK。

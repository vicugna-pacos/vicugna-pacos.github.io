---
title: "Adaptive Dialog"
date: 2020-11-12T10:02:49+09:00
draft: true
---

https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-adaptive-dialog-introduction?view=azure-bot-service-4.0

Adaptive Dialogは新しいイベントベースの追加機能を提供します。これにより、中断やディスパッチなどの制御が簡単になります。

重要：
Adaptive dialogは、現在.NET版のBot Framework SDKでのみ使用可能。Adaptive dialogを使ったサンプルソースは、[GitHubリポジトリ](https://github.com/microsoft/botbuilder-samples/tree/master/samples/csharp_dotnetcore)で参照可能。


## 前提条件

* Bot Framework V4 SDK の Dialog の基礎知識
* Bot Framework V4 SDK の Prompt の基礎知識

## Adaptive dialogs とは

### なぜ adaptive dialog なのか
Adaptive dialog は WaterfallDialog に対してたくさんのアドバンテージがある。主な特徴は下記の通り。

* 会話のフローをコンテキストやイベントによって動的に更新できる柔軟性を提供する。会話の途中でコンテキストが切り替わったり中断される場合に便利。
* Dialogのイベントシステムをサポートする。中断、キャンセル、実行など。
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


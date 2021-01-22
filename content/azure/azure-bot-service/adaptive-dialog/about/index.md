---
title: "Adaptive Dialog とは"
date: 2021-01-08T13:27:54+09:00
weight: 1
---

## はじめに
参考：

* [Introduction to adaptive dialogs - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-adaptive-dialog-introduction)
* [Adaptive Dialog のサンプル (GitHubリポジトリ)](https://github.com/microsoft/BotBuilder-Samples/tree/main/samples/csharp_dotnetcore/adaptive-dialog)

ボットとユーザーの会話は Dialog を使って実装するが、会話の複雑さが上がるほど中断とか分岐とか考慮しなくてはいけないことが増え、実装が大変になる。
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
それぞれのステップをメソッドで定義する Waterfall Dialog とは違い、Adaptive dialog の Action は Dialog クラスを拡張したもののリストである。
これにより Adaptive Dialog をより強力で柔軟にしつつ、中断や分岐などの管理を容易にする。

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

### Generator
Language Generator は、ボットからのメッセージをテンプレート化したものである。
いくつかの候補からランダムで発言したり、条件式などを指定できる。

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


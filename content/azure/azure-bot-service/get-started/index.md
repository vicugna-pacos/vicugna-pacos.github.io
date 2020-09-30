---
title: "まず始めてみる"
date: 2020-09-23T19:11:50+09:00
weight: 1
---

## 前提条件

* Visual Studio 2019 Community 版で開発
* 言語はC#を選択

## まず始めてみる

参考：[Bot Framework SDK for .NET を使用したボットの作成 - Bot Service | Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0&tabs=vs)

### Bot Service Emulator をインストール
[Bot Service Emulator](https://github.com/microsoft/BotFramework-Emulator/tree/master)

Bot Service Emulatorは、ローカルでボットをテストするためのツール。
上記サイトの「Download」にある「Github Releases」のページへ移動し、最新版のインストーラを取得し、インストールする。

### Bot Framework v4 SDK Templates をインストール

1. Visual Studioを起動し、メニューの「拡張機能」→「拡張機能の管理」をクリック。
1. 「Bot Framework v4 SDK Templates for Visual Studio」を検索し、ダウンロード。
1. Visual Studioを終了し、インストールを実行させる。
1. 再度 Visual Studioを起動する。

### 新しいプロジェクトの作成
Visual Studioのメニューの「ファイル」→「新規作成」→「プロジェクト」をクリック。
テンプレートとして、「Echo Bot (Bot Framework v4 - .NET Core 3.1)」を選ぶ。※.NET Core 2.1を選ばないように注意。

![](2020-09-23-20-59-33.png)

### プロジェクトを実行
ソリューションエクスプローラーで、プロジェクト名を選択して `F5` を押す。
するとプロジェクトがビルド＆実行される。実行されるとブラウザが起動し、`http://localhost:3978/` として以下のページが表示される。

![](2020-09-25-10-33-03.png)

実行時に起動したブラウザは、開いたままにしておく。不要なので閉じてしまいたいが、そうすると実行が終了してしまう。

### Bot Service Emulator でテストする

1. Bot Service Emulator を起動する。
1. 「Open Bot」のボタンを押す。
1. 「Bot URL」に `http://localhost:3978/api/messages` と入力する。他の項目は空白。
1. 「Connect」ボタンを押す。
1. チャットのテストができる。

Echo Bot は、こちらが送ったメッセージをそのまま返すだけのボットなので、送ったメッセージがそのまま返ってくればOK。

### そのほかのテンプレート
参考：[microsoft/BotBuilder-Samples](https://github.com/microsoft/BotBuilder-Samples/tree/main/generators/dotnet-templates)

Bot Framework v4 SDK Templates をインストールすると、Echo Bot の他にもテンプレートが追加される。
それぞれの名前と概要は以下の通り：

* Echo Bot
  * チャットに応答するだけのシンプルなテンプレート。まず初めにチャットボットを触る人向け。
* Core Bot
  * Bot Serviceにまつわる6つの機能が含まれたテンプレート。LUISとかDialogとか色々。
* Empty Bot
  * 最低限のソースが含まれているテンプレート。接続時に「Hello World!」と返す以外は何もしない。一からボットを構築したい人向け。


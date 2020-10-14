---
title: "Azure Bot Service でボットを作成する(VS Code 版)"
date: 2020-10-12T15:25:42+09:00
weight: 2
---

## はじめに
Azure Bot Service でのボット開発を VS Code で行い、GitHubと連携して CI/CD する手順。

前提条件：

* Windows 10
* SDKはC#を選択
* Bot Framework Emulator インストール済
* VS Code インストール済

参考：[Bot Framework SDK for .NET を使用したボットの作成 - Bot Service | Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0&tabs=vc)

## 準備
### .NET Core のインストール

[.NET Core SDK](https://dotnet.microsoft.com/download) をダウンロードし、インストールする。
Runtimeではなく、SDKが必要。

インストール後、コマンドプロンプトで `dotnet --version` を実行し、インストールされていることを確認する。

```
C:\xxx>dotnet --version
3.1.402
```

### テンプレートのインストール
コマンドプロンプトで、下記3つのうち、必要なものを実行する。

```
dotnet new -i Microsoft.Bot.Framework.CSharp.EchoBot
dotnet new -i Microsoft.Bot.Framework.CSharp.CoreBot
dotnet new -i Microsoft.Bot.Framework.CSharp.EmptyBot
```

インストール後の確認は、`dotnet new --list` で行える。

```
C:\xxx>dotnet new --list
(略)

Templates                                         Short Name               Language          Tags
--------------------------------------------------------------------------------------------------------------------------------------------
Bot Framework Echo Bot (v4.10.3)                  echobot                  [C#]              Bot/Bot Framework/Echo Bot/Conversational AI/AI
```

### VS Code に C# プラグインを追加
[C# 拡張機能](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)をインストールする。
これがないと VS Code でデバッグできない。

## ボットの作成
### 新しいプロジェクトを作成
コマンドプロンプトで、プロジェクトを作りたいフォルダまで移動する。
cdを使ってもいいが、エクスプローラーで目的のフォルダを開き、アドレス欄に `cmd` と入力してエンターキーを押すと早い。

その後、コマンドプロンプトで下記コマンドを実行する。

```
dotnet new [テンプレートのShort Name] -n [プロジェクト名]
```

今回は、EchoBotをひな型としてプロジェクトを作成した。しばらく待つと、指定したフォルダにプロジェクトが作成される。

```
C:\bot>dotnet new echobot -n MyEchoBot
The template "Bot Framework Echo Bot (v4.10.3)" was created successfully.

Processing post-creation actions...
No Primary Outputs to restore.
```

### VS Code で実行する
VS Code でプロジェクトフォルダを開き、メニューの「実行」から「デバッグの開始」または「デバッグなしで実行」をクリック。
はじめての場合は環境を選択するダイアログが開くので、「.NET Core」を選択する。

![](2020-10-14-11-31-49.png)

ビルドが実施され、アプリが実行される。ブラウザで「http://localhost:3978/」が開かれると起動完了。

Bot Framework Emulatorで `http://localhost:3978/api/messages` を開くとテストできる。

## ボットをAzureへデプロイ

### ソースコードをGitHubへコミット＆プッシュ
GitHubにリポジトリを作成し、ソースコードをコミット＆プッシュする。リポジトリはprivateでも可。

### Azure で Web App Bot を作成
Azureポータルで「リソースの作成」を選択し、「Web App Bot」を探して「作成」を押す。

![](2020-10-12-15-33-27.png)

ボット作成の画面になるので、必要事項を入力して「作成」を押す。

![](2020-10-12-15-40-10.png)

* ボットハンドル - ボットの名前。おそらくAzure全体で一意でなければいけない。チャンネルに接続するときなどに表示される名前。
* 価格 - F0(無料)推奨。後で変更可能。
* ボットテンプレート - ローカルのソースコードで上書きするのでどれでもいい。言語はC#。
* Application Insights - オンにすると、ボットのログを記録できる。あとで日次データ量の制限をかけておくことを推奨。

しばらく待つとリソースが作成される。

### CI/CD の設定

Azureポータルにて、ボット作成時に一緒に作成された App Service の方のリソースを選択し、「デプロイ センター」→「GitHub」をクリックする。

![](2020-10-12-16-20-13.png)

Azure と GitHub の連携が初めての場合、GitHubにログインしてAzureのアクセスを許可する手順がある。

それが完了した後、ビルドプロバイダーを選択するページになるので「GitHub Actions」を選択する。

次に構成を選択するページになるので、下記の通り選択する：

* 組織 - GitHubの組織、またはアカウントを選択する。
* リポジトリ - ボットのソースコードをコミットしたリポジトリを選択する。
* ブランチ - CI/CDの対象になるブランチを指定。通常、mainになるかと思う。
* ランタイムスタック - アプリの言語を指定。今回は「.NET Core」を選択。
* バージョン - ソースコードのバージョンを確認して、選択する。

![](2020-10-12-16-28-45.png)

最後に確認画面になるので、内容を確認して確定する。

確定後、GitHub側にデプロイ実行用のファイルが追加され、Actionが実行される。

![](2020-10-12-16-32-14.png)

Actionが完了すれば、CI/CDの設定とデプロイが完了する。
今後、mainブランチにコミット＆プッシュするたびに、デプロイが実行される。

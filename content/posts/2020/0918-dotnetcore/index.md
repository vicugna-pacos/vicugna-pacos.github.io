---
title: ".NET Coreを初めて触る"
date: 2020-09-18T17:16:22+09:00
draft: true
---

## はじめに
Azureの学習をしてみようと思うと、アプリケーション類を実装する言語はやはり.NET Coreかなと思って触ってみた。
学習前の経験は以下のような感じ：

* .NET FrameworkはVB.NETで開発経験あり。
* AzureのアプリはNode.js, Javaで実装したことがある。個人学習レベル。

環境などの前提条件：

* Windows10
* Visual Studio Code で開発

## SDKのダウンロード

[Download .NET (Linux, macOS, and Windows)](https://dotnet.microsoft.com/download) へアクセスし、SDKをダウンロードする。
「Download .NET Core SDK」のボタンをクリック。  
![](2020-09-18-17-21-41.png)

ダウンロードされたexeファイルを実行してインストール。

## 製品利用統計情報収集のオプトアウト
.NET Core SDKをインストールすると、既定で利用情報の送信が有効になっている。
これをオプトアウトしたい場合は、Windowsの環境変数に`DOTNET_CLI_TELEMETRY_OPTOUT`を追加し、値を`1`または`true`にする。


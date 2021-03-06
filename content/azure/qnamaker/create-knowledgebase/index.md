---
title: "QnA Maker を作成する"
date: 2020-10-08T09:35:53+09:00
lastMod: 2021-06-25T15:00:30+09:00
weight: 1
---

## 前提条件

* Azureのアカウントを持っている

## 価格レベル
QnA Maker は、Azure の複数のリソース(Cognitive Search, Azure Search, App Service など)を組み合わせて提供されている製品みたいなもので、価格レベルや制限は組み合わされたそれぞれのリソースの影響を受ける。
価格レベルは Free と Standard があるが、Free の特徴は下記の通り：

* サブスクリプション当たり1つしか作成できない。
* 作成できるナレッジベースは2つ。
  * Azure Cognitive Search の Free で作成できるインデックスの最大数は 3 だが、そのうち1つは `testkb` という名前でテスト用のナレッジベースとして使われている。

## QnA Maker のリソースを作成する
Azure ポータルサイトで QnA Maker のリソースを作成する。

* リソースグループ - 特に決まりやこだわりがない場合、QnA Maker用に新しいリソースグループを作ると分かりやすい。
* 価格レベル - お試しの場合は Free を選ぶ。
* App Insights - 会話のログを残したい場合は、「有効」にする。チャットボットアプリ側で詳細なログを記録するのであれば、ここでは無効にしておいた方がよい。

![](2020-10-08-09-46-18.png)

「確認および作成」を押したあと、しばらく待つと各リソースが作成される。

### App Service プランを変更する
QnA Maker と同時に新しい App Service と App Service プランを作成すると、価格が有料の「S1」になっているので、トライアル目的ならこれを無料の「F1」へ変えておく。

手順は下記の通り：  
Azureポータルサイトにて、App Service プランのリソースを選択し、「スケールアップ」→「開発/テスト」→「F1」とクリック。その後、「適用」を押して確定させる。

![](2020-10-08-11-00-16.png)

### App Insights の上限を設定する
App Insightsを有効にした場合、データ量などの上限を設定しておいた方がいい。
データ転送量は1ヶ月あたり5GBまでが無料だが、それ以上はお金がかかる。

参考：[価格 - Azure Monitor | Microsoft Azure](https://azure.microsoft.com/ja-jp/pricing/details/monitor/)

上限を設定する手順は以下の通り：

Azureポータルサイトにて、App Insights のリソースを選択し、「使用とコストの見積もり」→「日次上限」とクリック。
日次ボリュームの上限の欄に、「0.1」と入力して、OKをクリックする。こうしておけば1ヶ月当たりの上限を3GBへ制限できる(5GBギリギリを目指すなら、0.16か？)。

![](2020-10-08-10-43-30.png)

## ナレッジベースを作成する
[QnA Makerのサイト](https://www.qnamaker.ai/)へ移動し、メニューの「Create a knowledge base」をクリック。

![](2020-10-08-09-38-40.png)

さきほど作成した QnAサービスを指定し、次へ進む。

![](2020-10-08-13-31-30.png)

STEP 3 のナレッジベースの名前を入力する。

STEP 4 は初期データとして登録したいFAQのデータがあれば指定する。
Chit-chatは、雑談のQ&Aのデータ。日本語版があるようなので、必要なら追加する。
ただ、無料プランでQnAサービスを作った場合は、ファイルサイズが最大(1MB)を超えて登録できない場合がある。

![](2020-10-08-13-38-42.png)

## QnA Maker で作られるリソース

参考：[Azure リソース - QnA Maker - Azure Cognitive Services | Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/cognitive-services/qnamaker/concepts/azure-resources)

QnAサービスで作られるリソースについては、上記サイトを参照。
求めるパフォーマンスに合わせて、どのリソースの価格レベルを上げればよいか等の参考になると思われる。

---
title: "Luis"
date: 2020-12-25T12:55:45+09:00
lastMod: 2021-01-14T15:55:49+09:00
draft: true
---
参考：
* https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-v4-luis?view=azure-bot-service-4.0&tabs=csharp
* https://qiita.com/annie/items/5fdc9030521f8a0ed61c

## Cognitive Services の作成

自然言語の編集は下記ポータルサイトで行う。

https://www.luis.ai/

はじめてアクセスする場合、Azureアカウントの作成またはサインインをした後、そのアカウントに Cognitive Services を作成する流れになる。
LUISポータルサイトでリソースを作成する場合、新しいリソースグループの作成ができないので、Azureポータルサイトで、リソースグループと Cognitive Services を作っておいた方がいいかもしれない。

## App の作成
appは、ボットなどLUISを使いたい機能の単位で作成する。

## Intent の作成
Intent とは 意図 のこと。ピザの注文をできるボットを作る場合、Intentは下記のようになる。

* `OrderPizza` - ピザの注文内容を受け付ける。「マルゲリータピザとサイドメニューのサラダをください」という感じの文章。
* `Greeting` - ボットと会話を始めるための挨拶。
* `ConfirmOrder` - 注文内容を確定する。「それでいいです」とか「了解」とか。

なお、Intent の名前は日本語も指定できる。

### None について
必ず既定で作成され、削除もできない Intent。
自分が作成した Intent のいずれにも該当しない文章を入れておく。これを上手に入れておくと、学習精度が上がるらしい。

## 例文の追加
Intent を作ったら、その Intent に含めたい文章を登録していく。
「Examples」の下にあるテキストボックスに例文を入力し、エンターキーを押すと追加される。

## Entity の作成
「マルゲリータピザとサイドメニューのサラダをください」と言われたとき、
「マルゲリータピザ」と「シーザーサラダ」を注文したのだと理解したい。
そういうときに、あらかじめ「マルゲリータピザ」と「シーザーサラダ」を固有名詞として Entity に登録しておく。
この場合は、ListタイプのEntityを使う。

### 定義済みのエンティティ

[すべての作成済みエンティティ - LUIS - Azure Cognitive Services | Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/cognitive-services/luis/luis-reference-prebuilt-entities)

電話番号とかメールアドレスとか、汎用的なエンティティはあらかじめ用意されている。
上記ドキュメントに言語ごとの一覧に日本語があるので確認するとよい(サポートされているものが少ない…)。

これ以外に、[Microsoft Recognizers Text の GitHub リポジトリ](https://github.com/microsoft/Recognizers-Text) を調べてみると日本語のエンティティが定義されている。
このリポジトリが LUIS の 定義済みエンティティの元ネタになるっぽいが、実際は存在しないのでよくわからない。
さらにパターンが YAML ファイルになっていて、JSON をインポート形式としている LUIS には取り込めなさそう。

---
title: "LUIS"
date: 2020-12-25T12:55:45+09:00
lastMod: 2021-01-27T11:03:54+09:00
---
参考：
* https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-model
* https://qiita.com/annie/items/5fdc9030521f8a0ed61c

## Cognitive Services の作成

自然言語の編集は下記ポータルサイトで行う。

https://www.luis.ai/

はじめてアクセスする場合、Azureアカウントの作成またはサインインをした後、そのアカウントに Cognitive Services を作成する流れになる。
LUISポータルサイトでリソースを作成する場合、新しいリソースグループの作成ができないので、Azureポータルサイトで、リソースグループと Cognitive Services を作っておいた方がいいかもしれない。

## App の作成
appは、ボットなどLUISを使いたい機能の単位で作成する。

## Intent
Intent とは 意図 のこと。LUISは、まずユーザーの発話を意図で分類する。
ピザの注文をできるボットを作る場合、Intentは下記のようになる。

* `OrderPizza` - ピザの注文内容を受け付ける。
* `Greeting` - ボットと会話を始めるための挨拶。
* `ConfirmOrder` - 注文内容を確定する。「それでいいです」とか「了解」とか。

なお、Intent の名前は日本語も指定できる。

開発者は、`OrderPizza` の Intent に対し、いくつか例文を追加する。例えば下記の通り。

* ピザを注文したいのですが
* ピザください
* マルゲリータピザください

1つのインテントに対し、例文は15個程度用意するのが一番良い。
また、インテントごとの例文の数は、アプリ内でバランスが取れている必要がある。
例えば、特定のインテントに10個の例文があるのに対して、他のインテントの例文が500個もある状態はよくない。
例文の数に偏りがある場合は、多い方をパターンにまとめられないか検討すると良い。
なお、None インテントはこのバランスの母数に含めない。None インテントの例文は、全体のうち10%を占めるようにするのが良い。

### None について
必ず既定で作成され、削除もできない Intent。
自分が作成した Intent のいずれにも該当しない例文を入れておく。
これを入れておかないと、全く関係のない発話「音楽をかけて」に対して、`OrderPizza` の Intent に分類してしまったりする。
それを防ぐため、「音楽をかけて」を `None` の Intent に登録しておくとよい。

## Entity
参考：[Entity types - LUIS - Azure Cognitive Services | Microsoft Docs](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-entity-types)

Entity は、発話からデータを取り出す。
「マルゲリータピザとサイドメニューのサラダをください」と言われたとき、
「マルゲリータピザ」と「シーザーサラダ」を注文したのだと理解したい。
そういうときに、あらかじめ「マルゲリータピザ」と「シーザーサラダ」を固有名詞として Entity に登録しておく。

エンティティにはいくつか種類がある。

* 機械学習 エンティティ
* 非 機械学習 エンティティ
  * 正規表現
  * リスト
  * 定義済み (pre-build)

このうち、一番優先して採用すべきなのが機械学習エンティティとのこと。

### 機械学習エンティティ
機械学習エンティティは、他のエンティティを子要素として構造化できる。

### 定義済みのエンティティ

[すべての作成済みエンティティ - LUIS - Azure Cognitive Services | Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/cognitive-services/luis/luis-reference-prebuilt-entities)

電話番号とかメールアドレスとか、汎用的なエンティティはあらかじめ用意されている。
上記ドキュメントに言語ごとの一覧によると日本語も一部サポートされているっぽいが、実際に使えるかどうかは不明。

これ以外に、[Microsoft Recognizers Text の GitHub リポジトリ](https://github.com/microsoft/Recognizers-Text) を調べてみると日本語のエンティティが定義されている。
このリポジトリが LUIS の 定義済みエンティティの元ネタになるっぽいが、実際は存在しないのでよくわからない。
さらにパターンが YAML ファイルになっていて、JSON をインポート形式としている LUIS には取り込めなさそう。

## パターン
参考：[チュートリアル:パターン - LUIS - Azure Cognitive Services | Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/cognitive-services/luis/luis-tutorial-pattern)

パターンは、発話が似通っている際に精度を向上させるために使用する。パターンを活用すれば、インテントの例文を増やさずに精度を向上できる。

例えば、社員の上司や部下は誰か、という質問に答えてくれるアプリを作るとする。
インテントと例文として下記を登録する。

* インテント：上司を聞く
  * 佐藤は誰の部下ですか
  * 佐藤の上司は誰ですか
* インテント：部下を聞く
  * 佐藤は誰の上司ですか
  * 佐藤の部下は誰ですか
* インテント：None
  * 猫大好き
  * ピザ食べたい

また、「佐藤」は エンティティ：社員名 として登録している。

この状態で　TRAIN → Production へ PUBLISH したあと、下記APIを実行する。
URL内の下記の部分はそれぞれ置き換える。

* `YOUR-CUSTOM-SUBDOMAIN` - LUISを作ったリージョン。endpoint URL から分かる。
* `APP-ID` - LUIS のアプリID。
* `KEY-ID` - サブスクリプションキー。
* `YOUR_QUERY_HERE` - 「佐藤の上司は誰ですか」

```
https://YOUR-CUSTOM-SUBDOMAIN.api.cognitive.microsoft.com/luis/prediction/v3.0/apps/APP-ID/slots/production/predict?subscription-key=KEY-ID&verbose=true&show-all-intents=true&log=true&query=YOUR_QUERY_HERE
```

すると、結果として下記が返ってくる。

```json
{
    "query": "佐藤の上司は誰ですか",
    "prediction": {
        "topIntent": "部下を聞く",
        "intents": {
            "部下を聞く": {
                "score": 0.481271327
            },
            "上司を聞く": {
                "score": 0.481269926
            },
            "None": {
                "score": 0.02978976
            }
        },
        "entities": {
            "社員名": [
                [
                    "佐藤"
                ]
            ],
            "$instance": {
                "社員名": [
                    {
                        "type": "社員名",
                        "text": "佐藤",
                        "startIndex": 0,
                        "length": 2,
                        "modelTypeId": 5,
                        "modelType": "List Entity Extractor",
                        "recognitionSources": [
                            "model"
                        ]
                    }
                ]
            }
        }
    }
}
```

「佐藤の上司は誰ですか」と聞いているにも関わらず、topIntent は僅差で「部下を聞く」になっている。
これは、「上司を聞く」と「部下を聞く」の例文が似通っているのが原因。
パターンを使うとこの状況を改善できる。

パターンとして、下記を登録する。

* インテント：上司を聞く
  * {社員名}は誰の部下ですか
  * {社員名}の上司は誰ですか
* インテント：部下を聞く
  * {社員名}は誰の上司ですか
  * {社員名}の部下は誰ですか

なお、パターンには必ずエンティティが含まれていないといけない。

これで再度 PUBLISH まで実行して、先ほどと同じAPIを実行すると、下記の結果が返ってくる。

```json
{
    "query": "佐藤の上司は誰ですか",
    "prediction": {
        "topIntent": "上司を聞く",
        "intents": {
            "上司を聞く": {
                "score": 0.9999946
            },
            "部下を聞く": {
                "score": 0.0000184611981
            },
            "None": {
                "score": 6.185563e-7
            }
        },
        "entities": {
            "社員名": [
                [
                    "佐藤"
                ]
            ],
            "$instance": {
                "社員名": [
                    {
                        "type": "社員名",
                        "text": "佐藤",
                        "startIndex": 0,
                        "length": 2,
                        "modelTypeId": 5,
                        "modelType": "List Entity Extractor",
                        "recognitionSources": [
                            "model"
                        ]
                    }
                ]
            }
        }
    }
}
```

topIntent が「上司を聞く」になり、他のIntentともスコアが大きく差がついているのが分かる。

パターンは他にも、本のタイトル等どこからどこまでがエンティティなのかが分かりづらい場合にも有用である。

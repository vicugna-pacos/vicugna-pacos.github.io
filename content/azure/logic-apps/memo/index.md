---
title: "色々メモ"
date: 2021-05-18T16:05:53+09:00
lastMod: 2021-06-15T11:10:59+09:00
---

## Excel
SharePoint, OneDrive に保存した Excel ファイルを操作できるが、テーブルの取得やテーブルへの行の追加など、一覧データの取得、更新くらいしかできない。

### テーブル名を指定してデータを取得
「表内に存在する行を一覧表示 (List rows present in a table)」を使う。

![](2021-06-15-10-42-27.png)

テーブル内に日付データがある場合、オプションの「DateTime形式」をどちらにするか検討した方が良い。
既定では「Serial Number」になっていて、日付のシリアル値で取得される。対して「ISO 8601」に変更すると、「2021-04-01T00:00:00.000Z」のような日付形式になる。
Excelの日付にはタイムゾーンがないので、日本時間のつもりで入力していても、末尾にZがついてUTCとして取得される。
Logic App 内で日付を扱いたいときは、replace 関数などで Z を +09:00 に置き換えてから扱うと良い。

オプションの「フィルタークエリ」には ODATA クエリ形式でフィルターを設定できる。
こちらは日付データに対してはシリアル値を指定しないといけない。
Logic App でシリアル値を求めるのが面倒くさい場合は、全データ取得後、「アレイのフィルター処理」でデータを抽出できる。

![](2021-06-15-11-35-00.png)

## 日付

### 現在日時の取得
「現在の時刻」を使う。

![](2021-06-15-11-05-52.png)

タイムゾーンがUTCになっているので、日本時間へ変換してから扱った方がいい場合がある。

### タイムゾーンと書式の変換
「タイムゾーンの変換」を使うと、ついでに書式の変換もできる。

![](2021-06-15-11-09-24.png)

## トラブルシューティング

### メールの添付ファイルを保存できない

現象：  
メール受信をトリガーとしたフローで、添付ファイルの保存がエラーとなるか、保存できても0KBになってしまう。

原因：  
メール受信トリガーで、「添付ファイルを含める」を「はい」にしていない。

## 料金

Logic App のリソースは、従来通りの Consumption と後からできた Standard の二種類がある。
Consumption はマルチテナント または 総合サービス環境 (integration service environment) で実行され、Standard はシングルテナントで実行される。

Consumption (マルチテナント)：

* 簡単に始められる
* 従量課金
* フルマネージド
* 1つの Logic App リソースに対して、ワークフローは1つのみ。


Standard：

* Azure Functions が実行できる場所であれば、ほぼ実行できる。デプロイメントスロットは現在はサポートされていない。
* 1つの Logic App リソースに対して、複数のワークフローを作成できる。ワークフローには ステートレス と ステートフル がある。
* 価格は、Logic App が実行(起動？)されている間の時間で課金される。

制限：
https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-limits-and-config?tabs=azure-portal

### サンプル
Consumption で作った、特定のメールが届いたら転送するワークフローを例にする。

![](2021-09-01-14-56-42.png)

価格表によると、1アクションが ¥0.003136 、Standard コネクタ が ¥0.014560 なので、
このトリガーでは3分に1回メールをチェックする設定になっているので、1ヶ月あたりの料金は

    ¥0.014560 × 20回 × 24時間 × 31日 ＝ およそ¥217

となる。  
3分に1回トリガーが動いているということなので、たとえメールが1通も届かなくても料金が発生する。
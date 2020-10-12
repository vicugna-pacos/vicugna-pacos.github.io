---
title: "Dialog"
date: 2020-10-02T13:55:11+09:00
draft: true
weight: 6
---

`Dialog`はユーザーとの複数回にわたる会話のやり取りを管理するものである。

`Dialog`には、以下のコントロールフローを持つ。begin, continue, end, pause and resume, cancel。

Dialogには、これまでの会話で集めた情報が保存される。そのため、Dialogは会話のターンのたびにストレージに保管＆取得する。

## Dialogの種類

|Type|Description|
|---|---|
|dialog|dialogの基本クラス|
|component dialog|あらゆる目的で使えるDialog|
|waterfall dialog|決まった流れのステップを定義して使う。|
|prompt dialogs|ユーザーへインプットを求め、答えを返す場合に使う。プロンプトは有効な入力値が得られるか、キャンセルされるまで続く。waterfall dialogと一緒に使うために設計された。|
|adaptive dialog|柔軟な会話の流れを作れる。adaptive dialogを開始するには、dialog managerから始めないといけない。|
|action dialogs|会話フローをプログラマチックに定義できる。式とかステートメントとか。adaptive dialogでのみ使用できる。|
|input dialogs|ユーザーにインプットを求めるときに使う。adaptive dialog内でのみ使用可能。|
|skill dialog|skillを使う。|
|QnA Maker dialog|QnA Makerを使う。|

## Dialogのパターン
Dialogの開始、管理には以下の主となる2つのパターンがある。

1. adaptive dialogを開始する場合、まず dialog managerのインスタンスを作成する。
1. 他のdialogの場合、dialog managerを使うか、dialogクラスを直接使う。

## Dialog stack
ダイアログはスタック状に積みあがる。

## Container Dialog
container dialogは、大きなダイアログの一部になりえる。
それぞれのcontainer dialogは、内部にdialogを持つ。

* それぞれのdialog setはdialog IDを解決するためのスコープを持つ。
* SDKは以下の2つのcontainer dialogを実装している。component dialog と adaptive dialog。それぞれの構造は異なるが、共に使うこともできる。

## Dialog ID
dialogをdialog setに追加するとき、ともに一意のIDをセットする。set内に含まれるdialogは、お互いをidで参照する。
dialog context がIDを元にdialogを引っ張ってきてくれる。もし見つからなかった場合は、エラーが投げられる。

## Component dialog
component dialogは、決まった順序で展開される会話で使用する。

## Adaptive dialog
adaptive dialogは、フレキシブルな会話の流れを作りたいときに使う。
adaptive dialogは、いくつかのビルトインの機能がある。中断のハンドリング、言語解析、言語生成など。



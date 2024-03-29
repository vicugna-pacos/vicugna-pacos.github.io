---
title: "XPath"
date: 2020-11-09T15:17:41+09:00
lastMod: 2021-01-05T13:45:34+09:00
---

## 参考

* [xpath cover page - W3C](https://www.w3.org/TR/xpath/)
* [XPath | MDN](https://developer.mozilla.org/ja/docs/Web/XPath)
* [クローラ作成に必須！XPATHの記法まとめ - Qiita](https://qiita.com/rllllho/items/cb1187cec0fb17fc650a)
* [XPath | TECHSCORE(テックスコア)](https://www.techscore.com/tech/XML/XPath/index.html/)

## サンプル

↓ルート要素からツリー構造を指定。

```
/html/body/h1
```

↓途中までのツリー構造の指定を省略。ツリー構造の途中で使うことも可能。

```
//h1
/html//a
//div//a
```

↓属性名と値を指定。

```
//h1[@class='header1']
```

↓属性名を指定。値は問わない。

```
//h1[@class]
//*[@class]
```

↓指定する文字列が含まれる要素を取得。

```
//h1[contains(@class, 'head')]
```

* 第1引数：文字列が含まれているかどうか調査する対象
* 第2引数：文字列

↓タグ内テキストを検索対象にしたい場合。

```
//h1[contains(text(), '見出し')]
```

ただし、この場合はh1タグ直下のテキストしか対象にならない。例えば、h1タグの中にspanタグがあり、そのspanタグのテキストが「見出し」だった場合は引っかからない。
子or孫要素のテキストまで検索したい場合は、下記のようにする。

```
//h1//*[contains(text(), '見出し')]
```

回りくどい書き方になるが、以下のようにも書ける。

```
//h1/descendant::*[contains(text(), '見出し')]
```

↓ 複数の条件を指定したい場合、and でつなげるか、単純に2つ並べて書く。

```
//h1[contains(@class, 'head') and contains(text(), '見出し')]
//h1[contains(@class, 'head')][contains(text(), '見出し')]
```

↓指定した要素が複数ある場合、○番目、を指定できる。序数は 1 から始まる。

```
//ol/li[3]
(//ol/li)[3]    ← 序数以外の部分をカッコで囲まないとうまくいかない場合がある。
```

## 軸 (Axis)
軸とは、起点となるノードから相対的な位置を示すのに使える。例えば、当ノードの子、親、兄弟ノードを指定できる。

例えば下記のサンプルだと「divの子ノードにあるa」を指定している。

```
//div/child::a
```

軸の名前のあとには `::` を付ける。記号によって省略できる場合は `::` は付けない。

* __ancestor__  
指定ノードの親～ルートノードまでの全ての祖先。
* __ancestor-or-self__  
指定ノードと、その親～ルートノードまでの全ての祖先。
* __attribute__  
指定ノードの属性。属性を持つのは要素のみです。この軸はアットマーク (@) によって省略できる。
* __child__  
指定ノードの子ノード。
* __descendant__  
指定ノードの子孫のすべての要素ノード。
* __descendant-or-self__  
指定ノードとその子孫のすべての要素ノード。
* __following__  
指定ノードの後に現れる、descendant、attribute ノードを除く全てのノード。
* __following-sibling__  
指定ノードと同じ親を持ち、ソース文書内で指定ノードの後に現れる全てのノード。
* __parent__  
指定ノードの親ノード。この軸は 2 つのピリオド (..) によって省略できる。
* __preceding__  
文書内で指定ノードの前に現れる、 ancestor、 attribute ノードを除く全てのノード。
* __preceding-sibling__  
指定ノードと同じ親を持ち、ソース文書内で指定ノードの前に現れる全てのノード。
* __self__  
指定ノード自身。 この軸はピリオド (.) によって省略できる。

## 関数
関数を使うと、「class属性に"header1"を含む」というような指定もできる。

参考：[Functions - XPath | MDN](https://developer.mozilla.org/ja/docs/Web/XPath/Functions)

### contains

    contains(param1, param2)

param1 に param2 が含まれていれば true を返す。

例：`contains(@class, "header1")`

### not

    not(expression)

式を真偽値として評価し、その逆の値を返す。
例えば、`not(contains(string))` とすれば、特定の文字を含まないものを検索できる。

### starts-with

    starts-with(param1, param2)

param1 が param2 で始まっていればが含まれていれば true を返す。

### translate

    translate(string, abc, XYZ)

文字列の置換を行う。ただし、他の言語で見られる replace 関数などとは動作が異なる。
置換対象となる abc の1文字目が見つかると、 XYZ の1文字目へ置き換える。

例1：

    translate('The quick brown fox.', 'abcdefghijklmnopqrstuvwxyz', 'ABCDEFGHIJKLMNOPQRSTUVWXYZ')
    ↓ 出力
    THE QUICK BROWN FOX.

例2：

    translate('The quick brown fox.', 'brown', 'red')
    ↓ 出力
    The quick red fdx.

XYZ が abc より短い場合、abcで見つかった部分の文字は削除される。そのため、空白削除に使える。

## Chrome で XPath を取得する
1. 開発者ツールを起動する。
1. Elements タブを開く。
1. HTML ソースから任意の要素を右クリック。
1. 「Copy XPath」をクリック。

![](2021-05-21-09-20-08.png)

この方法で XPath を取得した場合、要素自身や親要素に id 属性があれば `//*[@id="xxxx"]` など短めの XPath になるが、id 属性がない場合などは、
`/html/body/...` というようにルート要素から全ての要素を指定した長い XPath になりがち。なので、色々な属性値などを使い、要素を特定できる XPath を自分で作る方が簡潔に済む場合もある。

## Chrome での検証方法
1. 開発者ツールを起動する。
1. Elements タブを開く。
1. Ctrl + F を押す。
1. 出てきた検索欄に XPath を入力する。
  1. ただし XPath がシンプルな場合、普通の文字検索になってしまうので、先頭に . (ドット) を付けるとよさそう。

![](2021-01-05-10-57-26.png)

![](2021-01-05-10-59-08.png)

他の検証方法もある。

1. 開発者ツールを起動する。
1. Console タブを開く。
1. 入力欄に `$x("調べたいXPath")` と入力してエンターキーを押す。
1. 見つかった要素の一覧が表示される。
1. 一覧のそれぞれにマウスオーバーすると、ページ上のどこにあるかが表示される。また、クリックすると Element タブへ移動し、ソース上のどこにあるかが示される。

![](2021-01-05-11-01-52.png)

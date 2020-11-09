---
title: "XPath"
date: 2020-11-09T15:17:41+09:00
---

## 参考

[XPath | MDN](https://developer.mozilla.org/ja/docs/Web/XPath)

[クローラ作成に必須！XPATHの記法まとめ - Qiita](https://qiita.com/rllllho/items/cb1187cec0fb17fc650a)

## XPathの書き方 

ルート要素からツリー構造を指定。

```
/html/body/h1
```

途中までのツリー構造の指定を省略。

```
//h1
```

属性値を指定。

```
//h1[@class='header1']
```

指定する文字列が含まれる要素を取得。


```
//h1[contains(@class, 'head')]
```

* 第1引数：文字列が含まれているかどうか調査する対象
* 第2引数：文字列

テキストを検索対象にしたい場合。

```
//h1[contains(text(), '見出し')]
```

ただし、この場合はh1タグ直下のテキストしか対象にならない。h1タグの中にspanタグがあり、そのspanタグのテキストが「見出し」だった場合は引っかからない。
子or孫要素のテキストまで検索したい場合は、下記のようにする。

```
//h1[contains(*, '見出し')]
```

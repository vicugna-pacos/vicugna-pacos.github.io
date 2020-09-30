---
title: "こんなときはこうする"
date: 2020-09-03T00:00:00+09:00
tags: ["Hugo"]
weight: 4
---

## 日付のフォーマット
Go独特の書き方をする。  
例えば、年は`yyyy`とかではなく、`2006`と書く。フィールドごとに固定値があるらしい。

以下、Goのページから引っ張ってきた表記のサンプル：

    ANSIC       = "Mon Jan _2 15:04:05 2006"
    UnixDate    = "Mon Jan _2 15:04:05 MST 2006"
    RubyDate    = "Mon Jan 02 15:04:05 -0700 2006"
    RFC822      = "02 Jan 06 15:04 MST"
    RFC822Z     = "02 Jan 06 15:04 -0700" // RFC822 with numeric zone
    RFC850      = "Monday, 02-Jan-06 15:04:05 MST"
    RFC1123     = "Mon, 02 Jan 2006 15:04:05 MST"
    RFC1123Z    = "Mon, 02 Jan 2006 15:04:05 -0700" // RFC1123 with numeric zone
    RFC3339     = "2006-01-02T15:04:05Z07:00"
    RFC3339Nano = "2006-01-02T15:04:05.999999999Z07:00"
    Kitchen     = "3:04PM"
    // Handy time stamps.
    Stamp      = "Jan _2 15:04:05"
    StampMilli = "Jan _2 15:04:05.000"
    StampMicro = "Jan _2 15:04:05.000000"
    StampNano  = "Jan _2 15:04:05.000000000"

参照元：https://golang.org/pkg/time/

## 画像などを参照する
https://gohugo.io/functions/relurl/

```html
<img src="{{ "images/icon.png" | relURL }}">
```

`static`フォルダ配下に置いた画像などを使用する場合、`static`フォルダ内からのファイルパスを、`relURL`という関数へ渡す。
`relURL`は、相対パスを作ってくれる関数。絶対パスが必要な場合は、`absURL`を使用する。

## リンクを新しいタブで開くようにする
参考：[Configure Markup | Hugo](https://gohugo.io/getting-started/configuration-markup#markdown-render-hooks)

`layouts/_default/_markup/render-link.html` を作成する。

内容は以下の通り。

```html
<a href="{{ .Destination | safeURL }}"{{ if strings.HasPrefix .Destination "http" }} target="_blank" rel="noopener"{{ end }}>{{ .Text | safeHTML }}</a>
```

注意点：これで内容が置き換わるのは、`[]()`で記述したリンクのみである。単純にURLを書いて自動的にリンクが貼られた場合には対応していない。

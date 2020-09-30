---
title: "基本文法など"
date: 2020-09-30T19:12:45+09:00
weight: 2
---

## はじめに
Hugo バージョン：v0.74.3

Hugoのテンプレートは、Go言語の `html/template` とか `text/template` というライブラリを使用する。
詳しい使い方については、上記ライブラリのドキュメントを参照。

## 基本の文法
HTMLファイルで作ったテンプレート中に、 `{{  }}` で囲んだところがテンプレート用の構文を書くところになる。

定義済みの変数やプロパティは、以下のようにして記述する。基本的に、変数の内容がそのままHTMLへ出力される。

```go-html-template
{{ .Title }}
{{ $address }}
```

### コメントを書く

```go-html-template
{{/* コメント */}}
```

### 周辺の余白をトリムする
`{{- -}}` のように、かっこの内側に`-`を付けると、前後の余白を無くしてくれる。

例えば、以下のようにテンプレートを書いた場合：

```go-html-template
<div>
  {{- .Title -}}
</div>
```

HTMLの出力結果は以下のようになる。タイトルの前後に入れていた改行と空白がなくなる。

```html
<div>PageTitle!</div>
```

## 変数
テンプレートファイルでは、そのとき処理しているページの情報を、`Page`変数として参照できる。
ページ変数へは、`.`でアクセス可能で、`.Title`であれば、ページのタイトルを取得できる。
ページ変数の定義については、[Page Variables | Hugo](https://gohugo.io/variables/page/) 参照。

また、テンプレートファイル内で変数を定義することも可能。変数名は`$`から始める。

```go-html-template
{{ $arg1 := "aiueo" }}
{{ $arg1 }}
```

(v0.47以前) ちなみに、変数の値をif文の中で変更して、if文の後でその値を参照することはできないらしい。
v0.48以降であれば、`=`を使って条件分岐で変数の値を変えることができるらしい。

```go-html-template
{{ $var := "Hugo Page" }}
{{ if .IsHome }}
    {{ $var = "Hugo Home" }}
{{ end }}
Var is {{ $var }}
```

### Page変数
参考：[Page Variables | Hugo](https://gohugo.io/variables/page/)

よく使いそうなプロパティ：

* `.Title` - ページのタイトル(front matterに書いたもの)
* `.Content` - ページの内容(front matterの後に書かれたもの)
* `.Date`, `.LastMod`, `.ExpiryDate`, `PublishDate` - front matterに書いた日付。
* `.Draft` - 下書きかどうか(front matter)
* `.Next` - 次のページ。`{{with .Next}}{{.Permalink}}{{end}}`と書けば次ページへのリンクを貼れる。
* `.NextInSection` - 同一セクション内での次ページ。

## テンプレートファイルの再利用
テンプレートファイルから、別のテンプレートファイルを取り込むことができる。
どこにでも書くような内容は共通のテンプレートファイルにしておいて、他から使用することが可能。

共通テンプレートファイルは `layouts/partial` フォルダへ置くこと。

使用する側では、以下のように書く。

```go-html-template
{{ partial "header.html" . }}
```

`partial` 関数以外にも `template` という関数があるが、こちらは古いバージョンから使われてきたものらしい。
現在では、internalテンプレート(Hugo内蔵のテンプレート)を使うときに使う。

internalテンプレートの顔ぶれは、[HugoのGitHubリポジトリ](https://github.com/gohugoio/hugo/tree/master/tpl/tplimpl/embedded/templates)で参照できる。

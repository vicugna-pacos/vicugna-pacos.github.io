---
title: "Hugoのテーマを作る"
date: 2020-08-26T00:00:00+09:00
lastMod: 2020-09-03T00:00:00+09:00
tags: ["Hugo"]
draft: true
---

## はじめに
Hugoテーマ自作にチャレンジ。  
Hugo バージョン：v0.74.3

## 準備
### 空のサイトを作る
まずテーマを表示するためのサイトを作成する。公式で用意されているサンプル用のサイトを使うのが手っ取り早い。

gohugoio/hugoBasicExample: Example site to use with Hugo & Hugo Themes  
https://github.com/gohugoio/hugoBasicExample

ただ初めてテーマを触ってみる場合は、サイトの内容を把握できていないとテーマの動きも把握しづらいので、自分で空サイトを作成し、mdファイルをいくつか作ってから取り掛かるのも良いかもしれない。

### 空のテーマを作る
サイト用フォルダで、下記コマンドを実行する。

```
hugo new theme [テーマ名]
```

`themes`フォルダにテーマ名のフォルダが作成され、基本的なファイル達も作成されている。

以下、作成されるファイルの一覧：

    テーマ名フォルダ
    ├ archetypes
    │   └ default.md
    ├ layouts
    │   ├ _default
    │   │   ├ baseof.html  <- 全てのページのベースになるファイル
    │   │   ├ list.html  <- セクションページ
    │   │   └ single.html  <- 単体ページ
    │   ├ partials
    │   │   ├ footer.html
    │   │   ├ head.html
    │   │   └ header.html
    │   ├ 404.html
    │   └ index.html     <- トップページ
    ├ static
    │   ├ css
    │   └ js
    ├ LICENSE
    └ theme.toml

### テーマをGitリポジトリにする
テーマ名のフォルダをGitリポジトリにする。
そうすることで、バージョン管理はもとより、他のサイトにもテーマを使えるようになる。

![](2020-08-26-10-46-24.png)

## 使用できる変数
https://gohugo.io/variables/page/

各テンプレートでは、`Page`変数というものが使用できる。
`{{ .Title }}`など、ドットで始まる。

よく使いそうな変数：

* `.Title` - ページのタイトル(front matterに書いたもの)
* `.Content` - ページの内容(front matterの後に書かれたもの)
* `.Date`, `.LastMod`, `.ExpiryDate`, `PublishDate` - front matterに書いた日付。
* `.Draft` - 下書きかどうか(front matter)
* `.Next` - 次のページ。`{{with .Next}}{{.Permalink}}{{end}}`と書けば次ページへのリンクを貼れる。
* `.NextInSection` - 同一セクション内での次ページ。
* `.Pages` - Collection of regular pages and only first-level section pages under the current list page.

## baseof.htmlを作る
このファイルが全てのページのベースとなる。
`layouts\_default\baseof.html`を開くと、すでに内容が書かれているので、必要に応じて編集していく。

```html
<!DOCTYPE html>
<html>
    {{- partial "head.html" . -}}
    <body>
        {{- partial "header.html" . -}}
        <div id="content">
        {{- block "main" . }}{{- end }}
        </div>
        {{- partial "footer.html" . -}}
    </body>
</html>
```

## head.htmlを作る
`layouts/partials/head.html`を開く。内容は空なので、`head`タグとその中身を書く。

### generator情報を追加する
`head`タグ内に以下を記述しておく。

```html
{{ hugo.Generator }}
```

これを入れると `<meta name="generator" content="Hugo 0.18" />`といった感じのタグになる。
このサイトはHugoで作りました、というのを教える情報で、Hugoのサイトで配布されているテーマはほぼすべてこのタグが入っているらしい。
`generator`を教えるのは本来はセキュリティ上好ましくないとされているが、Hugoは静的ジェネレーターだし、入れてあげると良いのではと思う。

### cssなどを追加する
例えば、cssファイルを読み込ませたい場合は、`テーマフォルダ/static/css/main.css`にファイルを置き、htmlでは以下のように参照する。

```html
<link rel="stylesheet" href="{{ .Site.BaseURL }}css/skeleton.css">
```

## index.htmlを作る
トップページとなる `layouts\index.html` を開く。内容は空。

### 記事の一覧を表示する
トップページには、メインセクションの記事一覧を表示すると良いと思う。以下はそのサンプル。

```html
{{ define "main" }}
<h1>Posts</h1>
{{ range (where site.RegularPages "Type" "in" site.Params.mainSections) }}
  <article>
    <h2><a href="{{ .RelPermalink }}">{{ .Title }}</a></h2>
    {{ .Summary }}
    {{ if .Truncated }}
    <div>
      <a href="{{ .RelPermalink }}">続きを読む</a>
    </div>
    {{ end }}
  </article>
{{ end }}
{{ end }}
```

記事の順番は、既定では `Weight > Date > LinkTitle > FilePath` となっている。
Weightとは、front matterで指定できるプロパティ値。そして、Dateは降順が既定。

### ページネーションを入れる
https://gohugo.io/templates/pagination/

```html
{{ define "main" }}
<h1>Posts</h1>
{{ range ( .Paginate (where site.RegularPages "Type" "in" site.Params.mainSections)).Pages }}
  <article>
    <h2><a href="{{ .Permalink }}">{{ .Title }}</a></h2>
    {{ .Content }}
  </article>
{{ end }}
{{ template "_internal/pagination.html" . }}
{{ end }}
```

`_internal/pagination.html`を使うと、お手軽にページのナビゲーションを作れる。
このファイルの実体は、Hugo のソースコードを漁ると見られる。
[hugo/tpl/tplimpl/embedded/templates at master · gohugoio/hugo](https://github.com/gohugoio/hugo/tree/master/tpl/tplimpl/embedded/templates)

生成されるページネーションのHTMLは以下のような感じ。Bootstrap互換があるらしい。

```html
<ul class="pagination">
  <li class="page-item">
    <a href="/" class="page-link" aria-label="First"><span aria-hidden="true">&laquo;&laquo;</span></a>
  </li>
  <li class="page-item disabled">
    <a  class="page-link" aria-label="Previous"><span aria-hidden="true">&laquo;</span></a>
  </li>
  <li class="page-item active">
    <a class="page-link" href="/">1</a>
  </li>
  <li class="page-item">
    <a class="page-link" href="/page/2/">2</a>
  </li>
  <li class="page-item">
    <a class="page-link" href="/page/3/">3</a>
  </li>
  <li class="page-item disabled">
    <span aria-hidden="true">&nbsp;&hellip;&nbsp;</span>
  </li>
  <li class="page-item">
    <a class="page-link" href="/page/5/">5</a>
  </li>
  <li class="page-item">
    <a href="/page/2/" class="page-link" aria-label="Next"><span aria-hidden="true">&raquo;</span></a>
  </li>
  <li class="page-item">
    <a href="/page/5/" class="page-link" aria-label="Last"><span aria-hidden="true">&raquo;&raquo;</span></a>
  </li>
</ul>
```

### 記事一覧の件数を絞る
`range`の後に`first 10`などを指定する。

```html
<div>
    <p>Recent Posts</p>
    <ul>
        {{ $p1 := where site.RegularPages "Type" "in" site.Params.mainSections }}
        {{ range first 5 $p1 }}
        <li>
            <a href="{{ .Permalink }}">{{ .Title }}</a>
        </li>
        {{ end }}
    </ul>
</div>
```

### タグ一覧を作る

```html
<ul>
    {{ range $termName, $entries := .Site.Taxonomies.tags }}
    <li><a href="{{ "/tags/" | relLangURL }}{{ $termName | urlize }}">{{ $termName }}</a>
    {{ end }}
</ul>
```

## 単体ページを作る
`layouts/_default/single.html`が、記事単体を表示するときのテンプレートとなる。

```html
{{ define "main" }}
<article>
	<h1>{{ .Title }}</h1>
	{{ .Content }}
</article>
{{ end }}
```

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

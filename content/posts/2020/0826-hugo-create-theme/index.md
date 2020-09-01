---
title: "Hugoのテーマを作る"
date: 2020-08-26T00:00:00+09:00
tags: ["Hugo"]
draft: true
---

# はじめに
Hugoテーマ自作にチャレンジ。  
Hugo バージョン：v0.74.3

# 準備
## 空のサイトを作る
まずテーマを表示するためのサイトを作成する。公式で用意されているサンプル用のサイトを使うのが手っ取り早い。

gohugoio/hugoBasicExample: Example site to use with Hugo & Hugo Themes  
https://github.com/gohugoio/hugoBasicExample

ただ初めてテーマを触ってみる場合は、サイトの内容を把握できていないとテーマの動きも把握しづらいので、自分で空サイトを作成し、mdファイルをいくつか作ってから取り掛かるのも良いかもしれない。

## 空のテーマを作る
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

## テーマをGitリポジトリにする
テーマ名のフォルダをGitリポジトリにする。
そうすることで、バージョン管理はもとより、他のサイトにもテーマを使えるようになる。

![](2020-08-26-10-46-24.png)

# 使用できる変数
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

# baseof.htmlを作る
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
# index.htmlを作る
トップページとなる `layouts\index.html` を開く。内容は空。

## 記事の一覧を表示する
トップページには、メインセクションの記事一覧を表示すると良いと思う。以下はそのサンプル。

```html
{{ define "main" }}
<h1>Posts</h1>
{{ range (where site.RegularPages "Type" "in" site.Params.mainSections) }}
  <article>
    <h2>{{ .Title }}</h2>
    {{ .Content }}
  </article>
{{ end }}
{{ end }}
```
## ページネーションを入れる
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

# 単体ページを作る

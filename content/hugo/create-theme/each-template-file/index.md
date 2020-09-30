---
title: "各テンプレートファイルについて"
date: 2020-08-26T00:00:00+09:00
lastMod: 2020-09-03T00:00:00+09:00
tags: ["Hugo"]
weight: 3
---

## はじめに
Hugoのテーマの作り方。  
Hugo バージョン：v0.74.3

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


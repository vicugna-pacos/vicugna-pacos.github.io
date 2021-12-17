---
title: "Brower Automation"
date: 2021-11-21T16:20:40+09:00
draft: true
---

## 基本
アクションは「ブラウザー自動化」の中にある。

![](2021-11-21-18-18-06.png)

## UI要素の管理
アクションごとにUI要素を指定するのではなく、フロー全体で一括してUI要素を管理し、それをアクションで使うイメージ。各アクションで「UI要素の追加」ができるが、セレクタの編集などは、画面右側の「UI要素」のエリアで行う。

![](2021-11-23-17-00-24.png)

UI要素はセレクタとは別に、名前を付けられる。

![](2021-11-23-17-04-04.png)

名前を付けておくとわかりやすくなる。

## UI要素のセレクタ
ブラウザ上のUI要素を指定するときは、Ctrl キーを押しながら目的の要素をクリックする。
このとき、マウスオーバーしたときだけ class 属性を変えるような要素だったりすると、そのフローをいざ実行したときに「要素が見つからない」というエラーになる。そういう場合は、セレクタを編集する必要がある。

![](2021-11-23-17-14-41.png)

もしくは、そのUI要素をクリック等する前に、「Webページの要素にマウスをホバーします」アクションを使い、UI要素を指定した時と同じ状況を作り出すのも解決方法だと思われる。
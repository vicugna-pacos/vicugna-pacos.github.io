---
title: "Memo"
date: 2022-04-08T15:13:57+09:00
draft: true
---

参照設定
「Microsoft PowerPoint xx.x Object Library」

https://docs.microsoft.com/en-us/office/vba/api/powerpoint.application

https://docs.microsoft.com/en-us/office/vba/api/office.msoshapetype

shapeのtypeがplaceholderだと、タイトルとか本文とかになる。
そのタイトルとか本文のどれなのかは、placeholderformat.type で分かる。
https://docs.microsoft.com/en-us/office/vba/api/powerpoint.shape.placeholderformat
https://docs.microsoft.com/en-us/office/vba/api/powerpoint.placeholderformat.type

slide の追加は Slides.Add と Slides.AddSlide の2種類がある。
前者は以下にサンプルがあるけど、Addメソッドのドキュメントがない。
https://docs.microsoft.com/en-us/office/vba/api/powerpoint.slides
後者
https://docs.microsoft.com/en-us/office/vba/api/powerpoint.slides.addslide

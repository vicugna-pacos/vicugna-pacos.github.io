---
title: "Memo"
date: 2022-04-08T15:13:57+09:00
draft: true
---

## 参照設定
Excel VBA など PowerPoint 以外から PowerPoint を操作する場合は、下記を参照設定に追加する。

「Microsoft PowerPoint xx.x Object Library」

## ファイルの新規作成
[Application](https://docs.microsoft.com/en-us/office/vba/api/powerpoint.application) オブジェクトの [Presentations.Add](https://docs.microsoft.com/en-us/office/vba/api/powerpoint.presentations.add) メソッドを使う。
PowerPoint の場合、pptx ファイル ＝ [Presentation](https://docs.microsoft.com/en-us/office/vba/api/powerpoint.presentation) オブジェクトになる。

```vb
Dim oApp As PowerPoint.Application
Dim oPres As PowerPoint.Presentation

Set oApp = New PowerPoint.Application
Set oPres = oApp.Presentations.Add()
```

## ファイルを開く
[Presentations.Open](https://docs.microsoft.com/en-us/office/vba/api/powerpoint.presentations.open) メソッドを使う。

```vb
Dim oApp As PowerPoint.Application
Dim oPres As PowerPoint.Presentation

Set oApp = New PowerPoint.Application
Set oPres = oApp.Presentations.Open("C:\test\sample.pptx")
```

## スライドの追加
Presentation.Slides.[AddSlide](https://docs.microsoft.com/en-us/office/vba/api/powerpoint.slides.addslide) メソッドを使う。

```vb
Dim oApp As PowerPoint.Application
Dim oPres As PowerPoint.Presentation
Dim oItem As Variant
Dim oLayout As PowerPoint.CustomLayout


Set oApp = New PowerPoint.Application
Set oPres = oApp.Presentations.Add

For Each oItem In oPres.SlideMaster.CustomLayouts
    Set oLayout = oItem
    
    If oLayout.Name = "タイトルとコンテンツ" Then
        Exit For
    End If
Next

oPres.Slides.AddSlide 1, oLayout

'oPres.Close
'oApp.Quit

Set oApp = Nothing
```

一部のドキュメントのサンプルでは Slides.Add メソッドが使われているが、Office 365 (もしくはそれより前のバージョン) ではこのメソッドは廃止されている。
(無理やり書けば実行されるっぽい)

AddSlide メソッドの第2引数の CustomLayout は、スライドマスタのうちどのスライドを使うか、を指定する。
Presentation.SlideMaster.[CustomLayouts](https://docs.microsoft.com/en-us/office/vba/api/powerpoint.customlayouts) で取得できる。

どの CustomLayout を使うかは、Name プロパティで特定するしかないと思われる。つまり、操作対象の pptx のスライドマスタにあるスライドの名前を知っていないといけない。
名前は、PowerPoint の画面でスライドのレイアウト一覧を表示したときの名前と同じ。

![](2022-04-12-14-48-23.png)

## スライドに書き込む








https://docs.microsoft.com/en-us/office/vba/api/office.msoshapetype

shapeのtypeがplaceholderだと、タイトルとか本文とかになる。
そのタイトルとか本文のどれなのかは、placeholderformat.type で分かる。
https://docs.microsoft.com/en-us/office/vba/api/powerpoint.shape.placeholderformat
https://docs.microsoft.com/en-us/office/vba/api/powerpoint.placeholderformat.type

slide の追加は Slides.Add と Slides.AddSlide の2種類がある。
前者は以下にサンプルがあるけど、Addメソッドのドキュメントがない。
https://docs.microsoft.com/en-us/office/vba/api/powerpoint.slides
後者


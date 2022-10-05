---
title: "描画キャンバスをPNGへ変換"
date: 2022-10-05T20:53:39+09:00
draft: true
---

## 概要
ドキュメント内の描画キャンバスを、PNGへ変換する。

変換方法はいたって原始的：

1. 描画キャンバスをコピーする
1. 形式を選択して貼り付けで、PNGを指定する
1. 描画キャンバスを削除

```vb
Public Sub 描画キャンバスをPNGへ変換()
    
    Dim wShapes As VBA.Collection
    Dim wShape As Shape
    
    
    Set wShapes = New VBA.Collection
    
    
    ' 描画キャンバスを別リストに取っておく
    For Each wShape In ThisDocument.Shapes
        If wShape.AutoShapeType = -2 Then  ' 描画キャンバス
            wShapes.Add wShape
        End If
    Next
    
    For Each wShape In wShapes
        wShape.Select
        Selection.Copy
        Selection.Collapse
        Selection.PasteSpecial DataType:=15, Placement:=wdInLine
        wShape.Delete
    Next
    
End Sub
```

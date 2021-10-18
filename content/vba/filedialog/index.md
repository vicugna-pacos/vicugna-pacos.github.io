---
title: "ファイルを開く＆保存するダイアログ"
date: 2021-10-18T15:55:23+09:00
---

## はじめに
VBA でファイルを指定するダイアログを表示する方法はいくつかあるが、
一番柔軟性があるのは [Application.FileDialog](https://docs.microsoft.com/en-us/office/vba/api/office.filedialog) だと思われる。

Excel の Application オブジェクトに付いているプロパティで、Outlook の Application オブジェクトにはない。
そのため、Outlook VBA でファイル選択ダイアログを使いたい場合、Excel.Application オブジェクトを生成して使う必要がある。

## サンプル

```vb
Public Sub Sample1()
    Dim oDialog As FileDialog
    Dim iAction As Integer
    
    ' 実際は String 型が入る。For Eachで使う関係上 Variant で定義
    Dim vSelectedItem As Variant
    
    
    Set oDialog = Application.FileDialog(msoFileDialogFilePicker)
    
    oDialog.InitialFileName = "C:\test\"  ' 既定のフォルダを指定する
    
    iAction = oDialog.Show  ' ダイアログを表示する
    
    ' Showの戻り値が -1 の時がユーザーが「開く」ボタンを押したとき。
    '  キャンセルされた場合は 0
    If iAction <> -1 Then
        Set oDialog = Nothing
        Exit Sub
    End If
    
    For Each vSelectedItem In oDialog.SelectedItems
        Debug.Print vSelectedItem
    Next
    
    Set oDialog = Nothing
    
End Sub
```

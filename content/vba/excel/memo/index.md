---
title: "Memo"
date: 2021-12-17T14:49:24+09:00
draft: true
---

## フィルタに一致した行を削除する

```vb
Public Sub test()
    
    Dim oSheet As Worksheet
    Dim oList As ListObject
    Dim listName As String
    Dim filterIndex As Integer
    Dim criteria As String
    Dim visibleCount As Integer
    
    Set oSheet = Sheet1
    listName = "テーブル1"
    filterIndex = 1
    criteria = "りんご"
    
    Set oList = oSheet.ListObjects(listName)
    
    ' フィルタの設定
    ClearAutoFilter oList.Range
    oList.Range.AutoFilter filterIndex, criteria
    
    ' 対象データがあるかチェック
    visibleCount = oList.Range.SpecialCells(xlCellTypeVisible).Rows.Count
    
    If visibleCount > 0 Then
        ' Delete メソッドは表示されている行のみ削除する
        oList.DataBodyRange.EntireRow.Delete xlShiftUp
    End If
    
    ClearAutoFilter oList.Range
    
End Sub

Private Sub ClearAutoFilter(ByRef oRange As Range)
    Dim idx As Integer
    
    For idx = 1 To oRange.Columns.Count
        oRange.AutoFilter idx
    Next

End Sub
```

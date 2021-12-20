---
title: "列番号と列文字の変換"
date: 2021-12-20T16:18:08+09:00
---

## 列番号→列文字への変換

```vb
Private Function ColNumToLetter(colIndex As Integer) As String
    Dim result As String
    Dim colNum As Integer
    Dim colMod As Integer
    Dim cnt As Integer
    
    result = ""
    colNum = colIndex
    cnt = 1
    
    Do While colNum > 0 And cnt < 10
        colMod = (colNum - 1) Mod 26
        colNum = WorksheetFunction.RoundDown((colNum - 1) / 26, 0)
        
        result = Chr(65 + colMod) & result
        cnt = cnt + 1
    Loop
    
    ColNumToLetter = result
    
End Function
```

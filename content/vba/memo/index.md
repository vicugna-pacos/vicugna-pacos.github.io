---
title: "色々メモ"
date: 2021-10-27T16:28:14+09:00
lastmod: 2021-11-02T14:05:20+09:00
draft: true
---

## 余りを求める
整数同士の割り算であれば、[Mod 演算子](https://docs.microsoft.com/en-us/office/vba/language/reference/user-interface-help/mod-operator) を使える。
ただし、数字どちらかが小数で、小数ありの余りを求めたいときは使えない。
Mod 演算子は計算結果の小数を丸めてしまうのが理由。

小数ありの余りを求めたいときは、下記計算式を使う。

```vb
' 割り算の余りを求める
Private Function Modulus(num1 As Double, num2 As Double) As Double
    Modulus = num1 - num2 * Int(num1 / num2)
End Function
```

## For ループの特徴
For ループの条件に Collection.Count などを指定している場合、ループ内部で Collection の要素数を変動させてもループの回数は変わらない。

```vb
Public Sub Sample1()
    Dim oList As Collection
    Dim idx As Integer
    
    Set oList = New Collection
    oList.Add "str1"
    oList.Add "str2"
    
    For idx = 1 To oList.Count
        Debug.Print oList(idx)
        
        If idx = 1 Then
            oList.Add "str3"  ' str3 はイミディエイトウィンドウに出力されない
        End If
    Next
    
End Sub
```

Do While ループであれば、ループ内で要素数が変更されても、それに応じてループ回数が変わる。

```vb
Public Sub Sample2()
    Dim oList As Collection
    Dim idx As Integer
    
    Set oList = New Collection
    oList.Add "str1"
    oList.Add "str2"
    
    idx = 1
    Do While idx <= oList.Count
        Debug.Print oList(idx)
        
        If idx = 1 Then
            oList.Add "str3"
        End If
        
        idx = idx + 1
    Loop
    
End Sub
```
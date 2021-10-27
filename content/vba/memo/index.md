---
title: "Memo"
date: 2021-10-27T16:28:14+09:00
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

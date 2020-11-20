---
title: "「HRESULT からの例外:0x800AC472」のエラー"
date: 2020-11-20T09:25:33+09:00
---

## 前提条件

* Windows 10
* PowerShell v5.1
* Excelのバージョン：Office 365

## 概要
PowerShellからExcelを操作しているとき、特定の状況で下記のようなエラーが発生することがある。

    "0" 個の引数を指定して "Quit" を呼び出し中に例外が発生しました: "HRESULT からの例外:0x800AC472"

私がこのエラーに直面したのはPowerShellをいじっていた時だが、COM経由でExcelを操作するもの (例えば .NET 系、さらには UiPath とか) でも出てくるエラーのようにみえる。

## エラーの原因
端的にいうと、Excelがビジー状態で要求した操作ができないときに発生するらしい。

参考：

* [HRESULT 800ac472 from set operations in Excel](https://social.msdn.microsoft.com/Forums/vstudio/en-US/9168f9f2-e5bc-4535-8d7d-4e374ab8ff09/hresult-800ac472-from-set-operations-in-excel?forum=vsto)
* [excel - Does the inability of IMessageFilter to handle 0x800AC472 (VBA_E_IGNORE) make implementing IMessageFilter irrelevant? - Stack Overflow](https://stackoverflow.com/questions/14596337/does-the-inability-of-imessagefilter-to-handle-0x800ac472-vba-e-ignore-make-im)

## 解決方法
エラーが出る処理をループで囲み、エラーが発生したらリトライする。

ただ、必ずしもエラーが出る箇所が根本原因ではないことがある。
私の場合、最初は `$excel.Quit()` でExcelを終了させるところでエラーが起きていたが、
本当の原因はブックをOneDriveで共有しているフォルダに作成していたことだった。
OneDrive の同期の処理と、Excelを開いたり編集したりする処理がぶつかっていたと思われる。
ローカルのフォルダにブックを作成するようにしたら、エラーは起きなくなった。

---
title: "モジュール (psm1)"
date: 2021-05-10T11:36:14+09:00
weight: 2
---

## 概要

参考：[PowerShell のモジュール詳解とモジュールへのコマンドレット配置手法を考える - tech.guitarrapc.cóm](https://tech.guitarrapc.com/entry/2013/12/03/014013)

参考：[PSModulePath のインストールパスを変更する - PowerShell | Microsoft Docs](https://docs.microsoft.com/ja-jp/powershell/scripting/developer/module/modifying-the-psmodulepath-installation-path?view=powershell-5.1)

### モジュールを読み込む

モジュールを方法を2つ記載する。

まず1つ目は、モジュールを所定のフォルダへ置き、自動で読み込ませる方法。
フォルダパスは環境変数の `$env:PSModulePath` を参照すると分かる。大体、以下のパスが設定されているはず。

* `[ドキュメントフォルダ]\WindowsPowerShell\Modules`
* `C:\Program Files\WindowsPowerShell\Modules`

上記以外のフォルダを追加したい場合は、単純に `$env:PSModulePath` へパスを追加すると良い。ただし、この方法はセッション内でのみ有効。
永続的にフォルダを追加したい場合は、Windowsの環境変数に書き加える。

また、2つ目の方法として、個別にモジュールファイルを指定する方法もある。
ps1 ファイルのコメントを除いた先頭に、下記を追加する。

```powershell
using module ".\module1.psm1"
```

### モジュールを作成する
`$env:PSModulePath` に psm1 を置く場合、設定したフォルダに、モジュールファイル名と同じ名前のフォルダを作成する。

    hoge
    └ hoge.psm1

psm1 ファイルに関数を書いたら、エクスポートして外部から使えるようにする。

```powershell
Export-ModuleMember -Function hogeFunc
```

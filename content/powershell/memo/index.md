---
title: "Memo"
date: 2020-10-27T15:42:25+09:00
draft: true
---

## スクリプトのパスを取得する

```powershell
# スクリプトのフルパス
$MyInvocation.MyCommand.Path
# スクリプトのフォルダパス
Split-Path $MyInvocation.MyCommand.Path -Parent
```

## 関数

参考：[Approved Verbs for PowerShell Commands - PowerShell | Microsoft Docs](https://docs.microsoft.com/en-us/powershell/scripting/developer/cmdlet/approved-verbs-for-windows-powershell-commands?view=powershell-5.1)

命名規則がある。`動詞-名詞` の形。
動詞に使える単語は決まっている。`Get-Verb` というコマンドを実行すると一覧が表示される。


## モジュール (psm1)

参考：[PowerShell のモジュール詳解とモジュールへのコマンドレット配置手法を考える - tech.guitarrapc.cóm](https://tech.guitarrapc.com/entry/2013/12/03/014013)

参考：[PSModulePath のインストールパスを変更する - PowerShell | Microsoft Docs](https://docs.microsoft.com/ja-jp/powershell/scripting/developer/module/modifying-the-psmodulepath-installation-path?view=powershell-5.1)

### モジュールを読み込む

モジュールは所定のフォルダへ置くと、自動で読み込まれる。
パスは `$env:PSModulePath` を参照すると分かる。
`[ドキュメントフォルダ]\WindowsPowerShell\Modules` とか `C:\Program Files\WindowsPowerShell\Modules` が設定されている。

それ以外にフォルダを追加したい場合は、単純に `$env:PSModulePath` へパスを追加すると良い。ただし、この方法はセッション内でのみ有効。
永続的にフォルダを追加したい場合は、Windowsの環境変数に書き加える。

### モジュールを作成する
`$env:PSModulePath` に設定したフォルダに、モジュールファイル名と同じ名前のフォルダを作成する。

    hoge
    └ hoge.psm1

psm1 ファイルに関数を書いたら、エクスポートして外部から使えるようにする。

```powershell
Export-ModuleMember -Function hogeFunc
```

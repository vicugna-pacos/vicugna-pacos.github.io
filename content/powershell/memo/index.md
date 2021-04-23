---
title: "色々メモ"
date: 2020-10-27T15:42:25+09:00
lastMod: 2021-04-23T15:28:31+09:00
---

## 前提条件
基本的に PowerShell v5.1 を使っている。

## スクリプトのパスを取得する

```powershell
# スクリプトのフルパス
$MyInvocation.MyCommand.Path
# スクリプトのフォルダパス
Split-Path $MyInvocation.MyCommand.Path -Parent
```

## クラスの名前空間を省略する
参考：[about_Using - PowerShell | Microsoft Docs](https://docs.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_using?view=powershell-5.1)

using namespace を使うと、.NET のクラスを呼び出すときの名前空間を省略できる。C#などでコーディングするときの using と全く同じ働きをする。

```powershell
using namespace System.Text
using namespace System.IO

[string]$string = "Hello World"
## Valid values are "SHA1", "SHA256", "SHA384", "SHA512", "MD5"
[string]$algorithm = "SHA256"

[byte[]]$stringbytes = [UnicodeEncoding]::Unicode.GetBytes($string)

[Stream]$memorystream = [MemoryStream]::new($stringbytes)
$hashfromstream = Get-FileHash -InputStream $memorystream `
  -Algorithm $algorithm
$hashfromstream.Hash.ToString()
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

上記以外のフォルダを追加したい場合は、単純に `$env:PSModulePath` へパスを追加すると良い。ただし、この方法はセッション内でのみ有効。
永続的にフォルダを追加したい場合は、Windowsの環境変数に書き加える。

また、個別にモジュールファイルを指定する方法もある。
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

## ドットソース
例えば、`main.ps1` に書いたスクリプトの量が多くなり、`sub.ps1` に分割したい場合に使える。
`main.ps1` の任意の場所に下記サンプルを挿入すると、その場所に `sub.ps1` が挿入される感じになる。

```powershell
. ".\sub.ps1"
```

基本的には相対パスで指定してもいいが、PowerShellの場合、カレントフォルダはps1ファイルがある場所ではなく、__ps1ファイルを実行したときのカレントフォルダ__ になるので注意。
`main.ps1`では、自ファイルのパスを取得しつつ、絶対パスで `sub.ps1` をドットソースで読み込んだ方がよさそう。

### ドットソースと変数のスコープ
ドットソースは、記述した場所にそのままps1ファイルの中身をコピペするようなイメージで動作する。

例えば、下記のような `sub.ps1` があるとする。

```powershell
$CONST1 = "CONST1sub"

function Get-Test2() {
    Write-Host $CONST1
}
```

それを、`main.ps1` でドットソースで読込み、実行する。

```powershell
. ".\sub.ps1"

$CONST1 = "CONST1"

Get-Test2
```

すると、コンソールには `CONST1` が出力される。

理由としては、下記のような感じになるからだと思われる。

```powershell
# sub.ps1 ここから
$CONST1 = "CONST1sub"

function Get-Test2() {
    Write-Host $CONST1
}
# sub.ps1 ここまで

$CONST1 = "CONST1"

Get-Test2
```

なので、ドットソースの読込の部分を、`Get-Test2` 関数を実行する直前に移すと、出力結果は `CONST1sub` に変わる。

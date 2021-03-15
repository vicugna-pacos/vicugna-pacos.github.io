---
title: "スクリプトのサンプル"
date: 2021-03-11T13:54:46+09:00
---

## はじめに
WinActor の「スクリプト実行」で利用できるスクリプトのサンプル集。
同梱で用意されているものを少し改変しただけのものも含む。

前提条件：

* WinActor v7.2.0 で動作確認

## 相対パスを絶対パスへ変換
`$` で始まる変数は WinActor であらかじめ用意された特殊変数。
`$PARSE_FILE_PATH` に相対パスを設定すると、絶対パスへ変換してくれる。
このとき、`$FILE_PATH_TYPE` の値によって、パスの存在チェックが行われ、存在しない場合は空文字が返却される。

特殊変数は操作マニュアルに一覧が載っている。

```vb
SetUMSVariable "$FILE_PATH_TYPE", "13"
SetUMSVariable "$PARSE_FILE_PATH", foldername
folder = GetUMSVariable("$PARSE_FILE_PATH")
```

$FILE_PATH_TYPE の値：

* 0, 10 - 補完なし
* 1, 11 - 補完あり。指定したファイルの存在を確認
* 2, 12 - 補完あり。指定したファイルを含むフォルダの存在を確認
* 3, 13 - 補完あり。指定したフォルダの存在を確認
* 4, 14 - 補完あり。ファイル・フォルダの存在確認なし

0～4 はローカルパス・UNC パス・http/https を許容。
10～14 はローカルパス・UNC パスのみ許容。
初期値は0 。

## ファイルリスト作成
既存の「ファイルリスト作成」はサブフォルダのファイルもリストに含めるが、それを指定したフォルダ直下のファイルのみにしたもの ( dir コマンドの /S オプションを除いただけ )。
出力される内容はファイルの絶対パスではなく、ファイル名のみになるので注意。

```vb
folder = !フォルダ名!
outputfile = !ファイルリスト出力先!

If folder = "" Then
  folder = "."
End If

' 「！フォルダ名！」のフォルダパスを作る
If Right(folder, 1) = "\" Then
  foldername = Left(folder, Len(folder) - 1)
Else
  foldername = folder
End If
SetUMSVariable "$FILE_PATH_TYPE", "13"
SetUMSVariable "$PARSE_FILE_PATH", foldername
folder = GetUMSVariable("$PARSE_FILE_PATH")

' 「!ファイルリスト出力先!」のファイルパスを作る
If Right(outputfile, 1) = "\" Then
  fname = Left(outputfile, Len(outputfile) - 1)
Else
  fname = outputfile
End If
SetUMSVariable "$FILE_PATH_TYPE", "12"
SetUMSVariable "$PARSE_FILE_PATH", fname
outputfile = GetUMSVariable("$PARSE_FILE_PATH")

cmd = "cmd.exe /c dir /S /a-d """ & folder & """ > """ & outputfile & """"

Set objShell = WScript.CreateObject("WScript.Shell")
Set objExec = objShell.Exec(cmd)

Do While objExec.Status = 0
  WScript.Sleep 300
Loop
```

## フォルダ作成 (再帰的)
同梱されているライブラリは、1階層分しかフォルダを作ってくれないので、複数階層のフォルダを作成するようにした。

```vb
str_folder = !作成フォルダ名!

If str_folder= "" Then
  WScript.Quit
End If

SetUMSVariable "$FILE_PATH_TYPE", "14"
SetUMSVariable "$PARSE_FILE_PATH", str_folder
str_folder = GetUMSVariable("$PARSE_FILE_PATH")

Set objFS = CreateObject("Scripting.FileSystemObject")

' 再帰的に親フォルダも作成する
Sub CreateFolder(ByVal folder)
  'フォルダが存在しない場合は新規作成する
  If Not objFS.FolderExists(folder) Then
    ' 親フォルダが存在するか確認する
    parentFolder = objFS.GetParentFolderName(folder)
    
    If parentFolder <> "" Then
      If Not objFS.FolderExists(parentfolder) Then
        CreateFolder(parentFolder)
      End If
    End If
    
    objFS.CreateFolder(folder)
  End If
  
End Sub

CreateFolder str_folder
```

## ファイル拡張子変換
変更前ファイルパスの拡張子を、指定した拡張子へ変換する。
変換後ファイル名にはパスが付いていないので注意する。

```
filePath = !変更前ファイル名!
extension = !拡張子!

Set objFS = CreateObject("Scripting.FileSystemObject")

baseName = objFS.GetBaseName(filePath)
newFileName = baseName + "." + extension

SetUmsVariable $変更後ファイル名$, newFileName
```

## 一時ファイル作成
拡張子を指定して、ユーザーのTempフォルダを使った一時ファイルのパスを作成する。
実際にファイルは作成せず、パスのみを作成する。

```vb
Dim FSO
Set FSO = CreateObject("Scripting.FileSystemObject")

Dim tempFolderPath
Dim tempFileName
Dim tempFilePath
Dim extension

extension = !拡張子!

tempFolderPath = FSO.GetSpecialFolder(2)
tempFileName = FSO.GetTempName
tempFileName = FSO.GetBaseName(tempFileName) & "." & extension

tempFilePath = tempFolderPath & "\" & tempFileName

SetUmsVariable $Tempファイル名$, tempFilePath

Set FSO = Nothing
```
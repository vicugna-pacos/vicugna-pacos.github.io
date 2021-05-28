---
title: "スクリプトのサンプル"
date: 2021-03-11T13:54:46+09:00
lastMod: 2021-05-28T10:21:15+09:00
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

相対パスの場所は、シナリオファイルが未保存の場合、「ドキュメント\WinActor」になる。

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

## フォルダ存在チェック
指定されたフォルダが存在するかチェックする。

「フォルダパス」：有無を確認したいフォルダの絶対パスか相対パス。  
「チェック結果」：確認した結果を格納する変数を指定。True：存在する、False：存在しない。

※操作対象のファイルを相対パスで指定する場合、開いているシナリオのフォルダが起点となる。

```vb
Option Explicit

Dim strFolder
Dim result
Dim fname

strFolder = !フォルダパス!

SetUMSVariable "$FILE_PATH_TYPE", "13"
SetUMSVariable "$PARSE_FILE_PATH", strFolder
strFolder = GetUMSVariable("$PARSE_FILE_PATH")

If Err.Number = 0 Then
    If Len(strFolder) = 0 Then
        result = False
    Else
        result = True
    End If
Else
  Err.Raise 1, "", "ライブラリの実行に失敗しました。"
End If

SetUMSVariable $チェック結果$, result
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

Set objFS = Nothing
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

## ファイル拡張子変換＆結合

ファイル名の拡張子を指定したものへ変換しつつ、フォルダパスと結合する。  
例：  
  フォルダパス：C:\test  
  ファイル名：sample.txt  
  拡張子：csv  
  ↓  
  連結結果：C:\test\sample.csv

```
Option Explicit

Dim str1
Dim str2
Dim extension

Dim baseName
Dim newFileName
Dim objFS

str1 = !フォルダパス!
str2 = !ファイル名!
extension = !拡張子!

Set objFS = CreateObject("Scripting.FileSystemObject")

baseName = objFS.GetBaseName(str2)
newFileName = baseName + "." + extension

Set objFS = Nothing


result = str1 & "\" & newFileName

SetUmsVariable $連結結果$, result
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

SetUmsVariable $ファイルパス格納先$, tempFilePath

Set FSO = Nothing
```

## ファイル読み込み (UTF-8)

```vb
Dim stream
Dim buf
Dim filePath
filePath = !読込ファイルパス!

Set stream = CreateObject("ADODB.Stream")
stream.Charset = "UTF-8"
stream.Open
stream.LoadFromFile filePath
buf = stream.ReadText
stream.Close
Set stream = Nothing

SetUmsVariable $読込データ$, buf

```

## ログ記録 (UTF-8)

__※ 条件は分からないが、何回か実行する or 実行のタイミングによって、ファイルに書き込めなくなるエラーが出るので、実用は無理そう。__

ログファイルの書き込み(追記)を行います。  
その際、書き込んだ日時、指定されたラベルも記録します。  
※「書き込みラベル」が未設定の場合、ラベルは記録されません。  
※操作対象のファイルを相対パスで指定する場合、開いているシナリオのフォルダが起点となります。 

```vb
Dim fso
Dim writeLabel
Dim writeData
Dim filePath
Dim lineData

writeLabel = !書き込みラベル!
writeData = !書き込みデータ!
filePath = !書き込みファイルパス!

SetUMSVariable "$FILE_PATH_TYPE", "14"
SetUMSVariable "$PARSE_FILE_PATH", filePath
filePath = GetUMSVariable("$PARSE_FILE_PATH")

Set fso = CreateObject("Scripting.FileSystemObject")
Dim stream1
Set stream1 = CreateObject("ADODB.Stream")
stream1.Type = 2
stream1.Charset = "UTF-8"
stream1.Open

If fso.FileExists(filePath) Then
	stream1.LoadFromFile(filePath)
	stream1.Position = stream1.Size
End If

If Len(writeLabel) = 0 Then
	lineData = Now & " " & writeData
Else
	lineData = Now & " " & writeLabel & " " & writeData
End If
stream1.WriteText lineData, 1

' 先頭のBOMを除去する
Dim binData

' Position をゼロにしてバイナリモードにする
stream1.Position = 0
stream1.Type = 1

' 先頭3バイトを除去して読込
stream1.Position = 3
binData = stream.Read()
stream1.Close
Set stream1 = Nothing

' バイナリデータを書き込む
Dim stream2
Set stream2 = CreateObject("ADODB.Stream")
stream2.Type = 1
stream2.Open
stream2.Write(binData)
stream2.SaveToFile filePath, 2
stream2.Close
Set stream2 = Nothing

Set fso = Nothing
```

## エラーログ記録

エラー情報収集とログ記録を同時に行います。

```vb
Dim objFSO
Dim objFile
Dim writeFilePath
Dim fileFormat

Dim errorName
Dim errorId
Dim errorMessage

writeFilePath = !書き込みファイルパス!
fileFormat = GetFileFormat(!ファイルフォーマット|Ascii形式,Unicode形式,システム既定!)

If writeFilePath = "" Then
  Err.Raise 1, "", "書き込みファイルパスを指定して下さい。"
End If

' エラー情報取得 -------------
' エラー発出ノード名
errorName = GetUmsVariable("$ERROR_NODE_NAME")
If IsNull(errorName) Then
  errorName = ""
End If

' エラー発出ノードID
errorId = GetUmsVariable("$ERROR_NODE_ID")
If IsNull(errorId) Then
  errorId = ""
End If

' エラーメッセージ
errorMessage = GetUmsVariable("$ERROR_MESSAGE")
If IsNull(errorMessage) Then
  errorMessage = ""
End If

' ログファイルに書き込み -----
'ファイルシステムオブジェクトを生成
Set objFSO = WScript.CreateObject("Scripting.FileSystemObject")

fname = writeFilePath
SetUMSVariable "$FILE_PATH_TYPE", "14"
SetUMSVariable "$PARSE_FILE_PATH", fname
writeFilePath = GetUMSVariable("$PARSE_FILE_PATH")

'ファイルをオープン
Set objFile = objFSO.OpenTextFile(writeFilePath, 8, True, fileFormat)
'書き込み
objFile.WriteLine(Now & " ERROR " & "エラーが発生したためシナリオを終了します。エラー情報は下記の通りです。")
objFile.WriteLine(Now & " ERROR " & "エラー発出ノード名：" & errorName)
objFile.WriteLine(Now & " ERROR " & "エラー発出ノードID：" & errorId)
objFile.WriteLine(Now & " ERROR " & "エラーメッセージ：" & errorMessage)

objFile.Close

Set objFile = Nothing
Set objFSO = Nothing

'**************************************************
' 概要: ファイルフォーマット文字列を Tristate 定数に変換する
' 引数: param ファイルフォーマット文字列
' 戻値: Tristate 定数
'**************************************************
Function GetFileFormat(param)
  Dim result
  Select Case param
  Case "Ascii形式"
    result = 0 'TristateFalse
  Case "Unicode形式"
    result = -1 'TristateTrue
  Case "システム規定"
    result = -2 'TristateUseDefault
  Case Else
    result = 0 'TristateFalse
  End Select
  GetFileFormat = result
End Function
```

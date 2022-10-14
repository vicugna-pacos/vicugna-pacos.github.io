---
title: "Graph PowerShell SDK"
date: 2022-10-14T20:34:37+09:00
---

## はじめに
PowerShell のコマンドレットは、2022年12月以降に廃止される予定。そのため、Graph PowerShell SDK への移行が必要。

[MSOnline / AzureAD PowerShell から Graph PowerShell SDK への移行について 1_概要 | Japan Azure Identity Support Blog](https://jpazureid.github.io/blog/azure-active-directory/azuread-module-retirement1/)

## インストール
[Microsoft Graph SDK をインストールする - Microsoft Graph | Microsoft Learn](https://learn.microsoft.com/ja-jp/graph/sdks/sdk-installation#install-the-microsoft-graph-powershell-sdk)

```powershell
Install-Module Microsoft.Graph
```

Graph 全部をインストールするためか、インストールに時間がかかる。

## 認証
2つの方法がある。

* とりあえず `Connect-MgGraph` を実行してサインイン画面を表示し、自分でログインする
* PC で証明書をつくり、Azure AD にアプリとして登録する

## 接続

```powershell
Connect-MgGraph -Scopes User.Read.All
```

スコープの指定必須。このあと使う API によってスコープは異なる。

### スコープ
必要なスコープは、各コマンドのドキュメントに書いてある。

例：[Get-MgUser (Microsoft.Graph.Users) | Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/microsoft.graph.users/get-mguser?view=graph-powershell-1.0&preserve-view=true)

スコープの種類の詳細：  
[Microsoft Graph permissions reference - Microsoft Graph | Microsoft Learn](https://learn.microsoft.com/en-us/graph/permissions-reference)

### 切断
`Connect-MgGraph` を実行すると、ユーザーフォルダにキャッシュファイル (みたいなもの) が保存され、PowerShell を終了した後も残る。
つまり再度 PowerShell を起動した後でも接続を再利用できるが、切断したい場合は以下を実行する。

```powershell
Disconect-MgGraph
```

参考：[MSOnline / AzureAD PowerShell から Graph PowerShell SDK への移行について 3_インストール・接続編 | Japan Azure Identity Support Blog](https://jpazureid.github.io/blog/azure-active-directory/azuread-module-retirement3/#%E3%82%88%E3%81%8F%E3%81%82%E3%82%8B%E8%B3%AA%E5%95%8F)

## コマンドを探す
下記公開情報に、既存のコマンドレットと Graph の対照表がある。

[Find Azure AD and MSOnline cmdlets in Microsoft Graph PowerShell | Microsoft Learn](https://learn.microsoft.com/en-us/powershell/microsoftgraph/azuread-msoline-cmdlet-map?view=graph-powershell-1.0)

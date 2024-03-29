---
title: "Visual Studio Code で開発する"
date: 2021-04-23T16:03:28+09:00
lastMod: 2021-08-19T14:40:54+09:00
weight: 1
---

参考：[Using Visual Studio Code for PowerShell Development - PowerShell | Microsoft Docs](https://docs.microsoft.com/en-us/powershell/scripting/dev-cross-plat/vscode/using-vscode)

## はじめに
VS Code で PowerShell を開発する方法を記載する。

Windows には PowerShell ISE というツールがあるが、ISE がサポートするのは PowerShell v5.1 以下で、それ以降のバージョンには対応していない。
そのため、なるべく VS Code で開発することが推奨される。
なお、ISE はバージョンアップはしないものの、セキュリティ対策などのアップデートは続けられるとのこと。

前提条件：

* PC に PowerShell がインストールされている
* PC に VS Code がインストールされている

## 拡張機能の追加
1. VSCodeでps1ファイルを開く。
2. PowerShellの拡張機能をインストールするか聞かれるので、インストールする。

または

1. VS Code を起動し、`Ctrl` + `P` を押し、Quick Open を起動する。
1.  `ext install powershell` と入力し、Enter キーを押す。
1. サイドバーに拡張機能のビューが開く。拡張機能の一覧から、Microsoft が提供している PowerShell 拡張機能を選択し、インストールする。
1. インストール完了後、インストールボタンが「Reload」へ変わったら、「Reload」ボタンを押す。
1. VS Code の再起動が完了すれば、拡張機能のインストールが完了する。

### 拡張機能で使う PowerShell のバージョンを指定する

1. `Ctrl` + `Shift` + `P` を押し、コマンドパレットを開く。
1. 「Session」と検索する。
1. `PowerShell: Show Session Menu` をクリック。
1. 使いたい PowerShell のバージョンをリストから選ぶ。

上記以外にも、VS Code の画面の右下にある、緑色で PowerShell のバージョンが書かれている場所をクリックしてバージョンを選べる。

## 文字コードの設定を変更
PowerShell ISE の既定の文字コードは UTF-8 bom付 だが、VSCodeは UTF-8。この違いが原因で日本語を出力した際などに文字化けするため、設定を変更する。

1. `F1` または `Ctrl` + `Shift` + `P` を押してコマンドパレットを表示する。
1. `Configure Language Specific Settings` と入力、出てきたものをクリック。<br>![](2021-04-23-16-05-58.png)
1. `PowerShell` を選ぶ。<br>![](2021-04-23-16-06-20.png)
1. Settings.jsonが開くので、`"files.encoding": "utf8bom"` を追記する。
1. Settings.jsonを保存して閉じる。

## デバッグの設定

### 開いているスクリプトを起動するようにする
エディタウィンドウで開いているファイルをデバッグするように設定する手順。

「実行」のタブを開く(`Ctrl` + `Shift` + `D`)。

「launch.jsonファイルを作成します」のリンクを押す。

「Launch Current File」を選ぶ。

![](2021-04-23-16-07-44.png)

launch.jsonが作成されて表示されるので、閉じる。

これで、デバッグしたいps1ファイルを開いた状態でF5を押すと、そのファイルが実行されるようになる。

### デバッグ時は一時的なセッションを使う
VSCode でスクリプトをデバッグするときは、ターミナルに表示されている「PowerShell Integrated Console」で実行される。
このままだとずっと同じセッションが再利用されるので、例えば psm1 ファイルを編集してデバッグしても、変更後の内容が反映されなかったりする。
これを防ぐために、拡張機能の設定を変更し、デバッグ実行の度に一時的なセッションを開くようにする。

手順は下記の通り：

1. Ctrl + , で設定を開く。

2. 検索欄に「session」と入力し、カテゴリ 拡張機能 → PowerShell Configuration の中にある「PowerShell > Debugging: Create Temporary Integrated Console」のチェックボックスをオンにする。

![](2021-04-23-16-11-54.png)


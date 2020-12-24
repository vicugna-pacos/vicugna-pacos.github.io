---
title: "Ngrok"
date: 2020-12-18T16:53:15+09:00
draft: true
---

## ngrok 
sign up したあと、Windows用のアプリケーションをダウンロードする。

ダウンロードした zip ファイルを解凍すると、exeファイルが1つ入っているので、任意のフォルダへ置いておく。

ngrok.exe を置いたフォルダでコマンドプロンプトを起動する。

下記コマンドを実行する。キーの部分は、サインインした状態で [Setup](https://dashboard.ngrok.com/get-started/setup) のページを見るとスニペットに書いてある。

```
ngrok authtoken [キー]
```

実行に成功すると、ユーザーフォルダ配下にngrokのファイルが作成される。

---
title: "逆引き"
date: 2020-10-26T20:16:37+09:00

---

## はじめに

* Windows 10

## Node.js のインストールおよびアップデート

[Node.js ダウンロードページ](https://nodejs.org/ja/download/) で最新のダウンロードしてインストールする。
npmも一緒にインストールまたはアップデートされる。

## Node.js のバージョン確認

```
node -v
```

## npm install したパッケージの確認

インストールしたパッケージの名前とバージョンが分かる。

```
npm list --depth=0
```

	xxxx@1.0.0 C:\xxxx
	+-- config@3.3.1
	+-- date-utils@1.2.21
	+-- googleapis@49.0.0
	`-- puppeteer-core@3.3.0

バージョンが古くなっているパッケージのみが表示される。

```
npm outdated
```

	Package         Current  Wanted  Latest  Location
	config            3.3.1   3.3.2   3.3.2  xxx
	googleapis       49.0.0  49.0.0  61.0.0  xxx
	puppeteer-core    3.3.0   3.3.0   5.4.0  xxx

##  パッケージの更新

```
npm update
```

必ず最新バージョンになるわけではなく、`npm outdated` で __Wanted__ の列に示されたバージョンまでしかアップデートしない。
インストールしたパッケージは package.json に `"config": "^3.3.2"` などと書かれているが、`^`がついていると、メジャーバージョンを超えるアップデートはしないようになっている。

とにかく最新バージョンにしたい場合は、一度パッケージをアンインストールしてからインストールしなおす。

```
npm uninstall googleapis
npm install googleapis
```

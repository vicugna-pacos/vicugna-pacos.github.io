---
title: "Storage"
date: 2021-02-05T15:16:22+09:00
draft: true
---

## ストレージアカウントの作成
参考：[Storage account overview - Azure Storage | Microsoft Docs](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-overview?toc=/azure/storage/blobs/toc.json)

Azure にデータベースなどを作成する場合、まず初めに ストレージアカウント を作成する。
ストレージアカウントに設定する名前空間がURLのエンドポイントとなる。
ストレージアカウントにはいくつか種類があり、それぞれサポートするデータ形式や価格が異なる。

* General-purpose v2 accounts: blob, ファイル、キュー、テーブル が作成できる基本的なアカウント。一番のおすすめ。
* General-purpose v1 accounts: 古いバージョンなので、v2がおすすめ。
* BlockBlobStorage accounts: block blobs と append blobs について特別なパフォーマンスを有する。トランザクションが特段に多いとか、特別にレイテンシを抑えたいとかの場合におすすめ。
* FileStorage accounts: ファイルだけ保存でき、特別なパフォーマンスを有する。エンタープライズか高性能なアプリケーションにおすすめ。
* BlobStorage accounts: 古いので general-purpose v2 がおすすめ。

### レプリケーション オプション
ストレージアカウントを作るときに、レプリケーションのオプションを選べる。

* Locally redundant storage (LRS): シンプルで安価。データは、主リージョンの単一の場所に同期的に3回コピーされる。
* Zone-redundant storage (ZRS): 高可用性が求められるときに使う。データは、主リージョンの3つのAZへ同期的にコピーされる。
* Geo-redundant storage (GRS): リージョン丸ごとの停止に備えたいときに使う。データは、主リージョンへ同期的に3回コピーされ、副リージョンへ非同期的にコピーされる。
* Geo-zone-redundant storage (GZRS): 高い可用性と耐久性の両方が欲しいときに使う。データは、主リージョンの3つのAZへ同期的にコピーされ、副リージョンへ非同期的にコピーされる。

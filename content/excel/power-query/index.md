---
title: "Power Query"
date: 2022-03-10T16:50:29+09:00
draft: true
---

https://docs.microsoft.com/en-us/power-query/power-query-what-is-power-query

## Power Query とは
Excel で使えるクエリ言語。
Excel に作った一覧表を検索・加工して、新たな一覧表を作れる。

検索対象はExcelに限らず、csvファイル、データベース、SharePointのリスト等々色々ある。
さらに Power Query は Power BI 等、Excel以外でも使える。

## 基本的な構造
クエリは、まず最初にデータソースの定義をする。
その後、細かい加工の設定を順番に積み上げていくイメージ。その積み上げた結果がクエリの結果になる。

1. データソースを定義する。これを表1とする。
1. 表1 から 列1 と 列2 だけを取得する。これを表2とする。
1. 表2 の 列1 の値が「Apple」の行だけ取得する。これを表3とする。


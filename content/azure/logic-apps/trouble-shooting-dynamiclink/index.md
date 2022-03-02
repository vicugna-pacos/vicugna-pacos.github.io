---
title: "チャットのメッセージに動的なリンクを挿入する"
date: 2022-03-02T16:36:48+09:00
---

## 発端
Power Automate で Teams のメッセージを送る時に、先のアクションで取得したURLを送りたいときがある。
そんな時、「チャットまたはチャネルでメッセージを投稿する」アクションで以下のようなメッセージを作った。

![](2022-03-02-16-58-57.png)

しかし、このアクションを一度最小化して戻すと、以下のようになってしまう。

![](2022-03-02-17-03-59.png)

## 解決策
concat 関数を挿入し、リンクのタグと変数を結合する。

![](2022-03-02-17-05-09.png)

挿入後の見た目はこんな感じ。

![](2022-03-02-17-06-10.png)

分かりづらくなってしまうが、いたしかたない。

参考：[Flow Bot unable to add clickable URL to Teams - Page 2 - Power Platform Community](https://powerusers.microsoft.com/t5/Power-Automate-Ideas/Flow-Bot-unable-to-add-clickable-URL-to-Teams/idc-p/710571/highlight/true#M23005)
---
title: "アクティブラーニング"
date: 2020-11-17T13:20:33+09:00
weight: 3
---
## アクティブラーニングとは
参考：[Active learning suggestions - QnA Maker - Azure Cognitive Services | Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/cognitive-services/qnamaker/concepts/active-learning-suggestions)

QnA Maker のアクティブラーニングを有効にすると、システムがナレッジベースに追加したほうがいいQ&Aを提案してくれるようになる。
管理者はその提案を採用するかどうかを判断する必要があり、ナレッジベースが勝手に変更されることはない。

アクティブラーニングは2種類のフィードバックがある。

* 暗黙的なフィードバック - ユーザーの質問にたいしてスコアが非常に近い回答が複数ある場合、これをフィードバックとする。アクティブラーニングを有効にしていると自動的に判定してくれる。
* 明示的なフィードバック - スコアに少しバラつきがある回答が複数見つかった場合、クライアントアプリケーションがユーザーに対し、どの質問が意図したものかを確認する。このときユーザーが返したフィードバックが明示的なフィードバックとなる。

## アクティブラーニングを有効にする
参考：[Use active learning with knowledge base - QnA Maker - Azure Cognitive Services | Microsoft Docs](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/use-active-learning)

アクティブラーニングは既定では無効なので、使いたいなら有効にする必要がある。

1. まず、一度でもナレッジベースをPUBLISHしておく。
1. QnA Maker ポータルサイトの右上にある自分のアイコンをクリックする。
1. 「Service settings」をクリック。
1. サービスの一覧が表示されるので、「Active Learning」のトグルボタンをオンにする。

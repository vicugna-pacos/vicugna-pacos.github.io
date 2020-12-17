---
title: "Activity Handler"
date: 2020-12-17T13:56:57+09:00
draft: true
---

## OnConversationUpdateActivityAsync

`OnTurnAsync(ITurnContext, CancellationToken)` の既定の動作をそのまま使う場合、
conversation update アクティビティをチャネルから受信したときに実行されるメソッド。
conversation update アクティビティは、会話に追加または削除されたユーザーに対して何か処理をする際に使える。
例えば、会話に追加されたユーザーに対して、挨拶を送ることができる。
既定では、このメソッドは、ユーザーが追加された場合は `OnMembersAddedAsync(IList<ChannelAccount>, ITurnContext<IConversationUpdateActivity>, CancellationToken)` を呼び出し、
ユーザーが削除された場合は `OnMembersRemovedAsync(IList<ChannelAccount>, ITurnContext<IConversationUpdateActivity>, CancellationToken)` を呼び出す。
メソッドはメンバーIDをチェックし、ボット以外のメンバーが更新されたときのみ応答する。

このメソッドには既定の動作があるので、オーバーライドするときは親クラスの同メソッドを呼び出す必要がある。

## OnMembersRemovedAsync
ユーザーがいなくなったときに呼ばれるメソッド。

しかし、本当にこのメソッドが呼び出されるかどうかは、チャネルによって異なる。
PCにインストールした Bot Framework Emulator では、チャットのタブを閉じてもこのイベントが呼び出されなかった。

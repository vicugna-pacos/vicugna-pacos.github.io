---
title: "メッセージングサービスを比較する"
date: 2020-10-06T17:23:22+09:00
draft: true
---

参考：[Compare Azure messaging services - Azure Event Grid | Microsoft Docs](https://docs.microsoft.com/en-us/azure/event-grid/compare-messaging-services)

Azureには3つのメッセージングサービスがある。

* Event Grid
* Event Hubs
* Service Bus

## イベントとメッセージ
メッセージングサービスの形態として、「イベント」と「メッセージ」がある。

イベントは、状態変化の通知などで使う。イベント発信元は、そのイベントがどう受け取られて、どう処理されるかには関知しない。

メッセージは、データの送信などで使う。発信元によって、そのメッセージを受信側でどういう処理をしてほしいかが決まっている。


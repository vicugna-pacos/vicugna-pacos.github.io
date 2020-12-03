---
title: "Azure Service Bus"
date: 2020-11-18T09:07:09+09:00
---

## 前提条件

* アプリはC#で実装

## 価格のメモ

無料プランはない。
価格レベルによって利用できる機能が違う。
Basic → Standard → Premium

キューはいずれのプランでも利用可能だが、トピックは Standard 以上の場合利用可能。

## 準備
Azure ポータルで、Service Bus のリソースを作成する。
作成後は、リソースのメニューの「キュー」へ移動し、キュー(メッセージの入れ物)を作成する。

## キュー
FIFO。メッセージ受信側は、メッセージが登録された順番でメッセージを取り出す。
受信側は基本的に1つ。同じことをするクライアントを複数作ってもよいが、どんなメッセージを取り出すかは選べないので、負荷分散的な使い方になる。

### キューの送信
参考：[Azure portal を使用して Service Bus キューを作成する - Azure Service Bus | Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/service-bus-messaging/service-bus-quickstart-portal)

参考：[Azure Service Bus キューの使用 - Azure Service Bus | Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/service-bus-messaging/service-bus-dotnet-get-started-with-queues)

Azureポータルを開き、作成した Service Bus のリソースを選択する。
「共有アクセスポリシー」を選び、ポリシーのプライマリ接続文字列を取得する。さらに、送信用と受信用の複数のポリシーを作っておくと良い。
また、準備の手順で作成したキューの名前も取得しておく。

送信側アプリにNugetパッケージの `Microsoft.Azure.ServiceBus` を追加する。

```
dotnet add package Microsoft.Azure.ServiceBus
```

メッセージ送信のサンプルは以下の通り。

```csharp
using Microsoft.Azure.ServiceBus;
using System.Text;
using System.Threading.Tasks;

// 略

const string ServiceBusConnectionString = "接続文字列";
const string QueueName = "キューの名前";

public static async Task SendQueueAsync()
{
    var queueClient = new QueueClient(ServiceBusConnectionString, QueueName);

    string messageBody = "メッセージ本体";
    var message = new Message(Encoding.UTF8.GetBytes(messageBody));
    await queueClient.SendAsync(message);

    await queueClient.CloseAsync();
}
```

### キューの受信
[QueueClientを使ったサンプル](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/GettingStarted/Microsoft.Azure.ServiceBus/BasicSendReceiveUsingQueueClient) は、一度起動すると、キューにあるメッセージをすべて取り出す。
受信したメッセージごとに実行されるコールバックメソッドを定義する形。

キューのメッセージを1件だけ取り出して処理したい場合は、[こちらのサンプル](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/GettingStarted/Microsoft.Azure.ServiceBus/SendReceiveUsingMessageSenderReceiver) または下記を参考にする。

```csharp
using Microsoft.Azure.ServiceBus;
using Microsoft.Azure.ServiceBus.Core;
using System;
using System.Text;
using System.Threading.Tasks;

// 略

const string ServiceBusConnectionString = "接続文字列";
const string QueueName = "キューの名前";

static IMessageReceiver messageReceiver;

static async Task Main(string[] args)
{
    messageReceiver = new MessageReceiver(ServiceBusConnectionString, QueueName, ReceiveMode.PeekLock);

    // メッセージを受信
    Message message = await messageReceiver.ReceiveAsync();
    Console.WriteLine($"Received message: SequenceNumber:{message.SystemProperties.SequenceNumber} Body:{Encoding.UTF8.GetString(message.Body)}");

    await messageReceiver.CompleteAsync(message.SystemProperties.LockToken);

    await messageReceiver.CloseAsync();
}
```

キューにメッセージがない場合、タイムアウトに指定された分だけ受信を待機した後、戻り値にnullを返す。
タイムアウトは、`ReceiveAsync` メソッドの引数に指定可能。

### キューの数を取得
https://github.com/Azure/azure-service-bus-dotnet/blob/master/test/Microsoft.Azure.ServiceBus.UnitTests/Management/ManagementClientTests.cs#L262

## トピック
複数の受信者を作れる。
受信側でどんなメッセージを取り出すか、SQLみたいに指定して選べる。
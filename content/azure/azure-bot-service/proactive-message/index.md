---
title: "ボットからメッセージを送る"
date: 2020-09-23T19:11:50+09:00
draft: true
---

## 前提条件

* Windows 10
* Bot Framework SDK v4
* .NET Core C#

## ボット側からメッセージを送る
チャットボットは、基本的にユーザーから会話が始まりボットは返信するだけだが、ボット側からメッセージを送ることもできる。
「プロアクティブな」メッセージというらしい。

参考：[ユーザーへのプロアクティブな通知の送信 - Bot Service | Microsoft Docs](https://docs.microsoft.com/ja-jp/azure/bot-service/bot-builder-howto-proactive-message?view=azure-bot-service-4.0&tabs=csharp)

参考：[Intro](https://microsoft.github.io/botframework-solutions/solution-accelerators/tutorials/enable-proactive-notifications/1-intro/)

## テスト用クライアント

```
dotnet new console -n client
```

```
dotnet add package Microsoft.Bot.Connector.DirectLine
dotnet add package Microsoft.Rest.ClientRuntime
```




## ボット

```
dotnet add package Microsoft.Bot.Solutions
```

Adapterクラスを作り、 `ProactiveStateMiddleware` を追加する。

DefaultAdapter.cs
```csharp
public class DefaultAdapter : BotFrameworkHttpAdapter
{

    public DefaultAdapter(IServiceProvider provider)
    {
        var proactiveState = provider.GetService<ProactiveState>();
        Use(new ProactiveStateMiddleware(proactiveState));
    }
}
```

Startup.cs
```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Adapterクラスを自作のクラスへ変更する
    services.AddSingleton<IBotFrameworkHttpAdapter, DefaultAdapter>();

    // storageはDIに登録しないこともあるが、ProactiveStateのコンストラクタで必要なため、ProactiveStateを使用するならstorageもDIへ登録しておく必要がある。
    services.AddSingleton<IStorage>(storage);
    services.AddSingleton<ProactiveState>();
}
```

---
title: "ボットからメッセージを送る (プロアクティブなメッセージ)"
date: 2020-09-23T19:11:50+09:00
draft: true
---

## 前提条件

* Windows 10
* Bot Framework SDK v4
* .NET Core C#

## ボット側からメッセージを送る
チャットボットは、基本的にユーザーから会話が始まりボットは返信するだけだが、ボット側からメッセージを送ることもできる。
ボットから話しかけることを、「プロアクティブな」メッセージという。

基本的には、ボットと会話したことのあるユーザーにしか話しかけることができない。
ただし、Teamsの場合、組織単位でボットをアプリとして許可していれば、組織内の一度も会話したことのないユーザーへ話しかけられるらしい。
その際は、Graph API を使ってユーザー情報を取得するらしい。

参考：[Send proactive notifications to users - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-proactive-message?view=azure-bot-service-4.0&tabs=csharp)

参考：[Intro](https://microsoft.github.io/botframework-solutions/solution-accelerators/tutorials/enable-proactive-notifications/1-intro/)

参考：[Send proactive messages - Teams | Microsoft Docs](https://docs.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/send-proactive-messages?tabs=dotnet)



## ConversationReference を保存
ユーザーがボットとの会話を始めると、`ConversationReference` という情報が作られる。
これを保存しておけば、一度会話したことのあるユーザーに対してメッセージを送れる。

本サンプルでは、ユーザーが会話に参加した時点で `ConversationReference` を保存する。

### Startup.cs
まず、`ConversationReference` を保存するオブジェクトをDIに登録する。
サンプルでは、ユーザーごとに振られるIDをキーとして、Dictionary型で保持するようにしている。

実運用の際は、オブジェクトをただDIに登録するのではなく、追加でストレージに保存した方が良いと思う。

```cs
namespace ProactiveBot
{
    public class Startup
    {
        // 略
        public void ConfigureServices(IServiceCollection services)
        {
            // 略

            // ConversationReferenceを保存しておくオブジェクト
            // (実際はどこかStorageに保存したほうが良い)
            services.AddSingleton<ConcurrentDictionary<string, ConversationReference>>();
        }

        // 略
    }
}
```

### ボット (ActiveHandler)



## Controller を追加


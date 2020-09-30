---
title: "Activity Handler"
date: 2020-09-23T19:11:50+09:00
weight: 3
---

## 前提条件

* Visual Studio 2019 Community 版で開発
* 言語はC#を選択

## Activity Handler
アクティビティは、ActivityHandlerで処理する。ボットを作る場合、このActivityHandlerを拡張して会話を実装していく。
ActivityHandlerには各イベントごとにメソッドがあるので、それぞれをオーバーライドして拡張していく感じ。
C#でテンプレートを使って新しいプロジェクトを作った場合、ActivityHandlerを継承したクラスが既に作られている。`EmptyBot` とか、`EchoBot`がそれにあたる。

### 基本形
Echo Bot のActivityHandlerが参考になる。

```csharp
namespace EchoBot1.Bots
{
    public class EchoBot : ActivityHandler
    {
        protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
        {
            var replyText = $"Echo: {turnContext.Activity.Text}";
            await turnContext.SendActivityAsync(MessageFactory.Text(replyText, replyText), cancellationToken);
        }

        protected override async Task OnMembersAddedAsync(IList<ChannelAccount> membersAdded, ITurnContext<IConversationUpdateActivity> turnContext, CancellationToken cancellationToken)
        {
            var welcomeText = "Hello and welcome!";
            foreach (var member in membersAdded)
            {
                if (member.Id != turnContext.Activity.Recipient.Id)
                {
                    await turnContext.SendActivityAsync(MessageFactory.Text(welcomeText, welcomeText), cancellationToken);
                }
            }
        }
    }
}
```

### イベントハンドラ
主に使うのは、おそらく以下の2つ。

|メソッド名|説明|
|---|---|
|`OnMembersAddedAsync`|ユーザーが新しくボットに接続した(会話をはじめた)|
|`OnMessageActivityAsync`|メッセージアクティビティを受信した。メッセージのやり取りはここで処理する。|

---
title: "テキストメッセージの送受信"
date: 2020-10-02T15:09:42+09:00
weight: 5
---

## メッセージの受信
ユーザーが送ってきたメッセージは、以下のようにして受け取る。

```csharp
var responseMessage = turnContext.Activity.Text;
```

## メッセージの送信
ボットからシンプルなテキストを送るには、以下のようにする。

```csharp
await turnContext.SendActivityAsync("Welcome!");
```

または、以下のようにする。

```csharp
var text = "Welcome!";
var msg = MessageFactory.Text(text, text);
await turnContext.SendActivityAsync(msg);
```

`MessageFactory`は、ボットから送るメッセージを作るためのユーティリティクラス。
添付ファイルとかボタンとか色々追加できるので、たくさん使うことになる。

## 「入力しています」のメッセージの送信
もし即座に返事を返せない場合、Teamsで出てくるような「○○が入力しています」のようなメッセージを返すことができる。
以下のサンプルは、ユーザーが「wait」と送ってきたときに、まず「入力しています」を送り、3秒後にメッセージを送信する。

※ Bot Framework Emulatorは非対応のためテストできない

```csharp
private async Task SendTypingAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    if (string.Equals(turnContext.Activity.Text, "wait", System.StringComparison.InvariantCultureIgnoreCase))
    {
        await turnContext.SendActivitiesAsync(
            new Activity[] {
                new Activity { Type = ActivityTypes.Typing },
                new Activity { Type = "delay", Value= 3000 },
                MessageFactory.Text("Finished typing", "Finished typing"),
            },
            cancellationToken);
    }
    else
    {
        var replyText = $"Echo: {turnContext.Activity.Text}. Say 'wait' to watch me type.";
        await turnContext.SendActivityAsync(MessageFactory.Text(replyText, replyText), cancellationToken);
    }
}
```

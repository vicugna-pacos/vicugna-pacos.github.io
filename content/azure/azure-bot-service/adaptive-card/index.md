---
title: "Adaptive Card を送る"
date: 2020-11-13T12:47:14+09:00
weight: 6
---

## Adaptive Card とは
ボットからユーザーへ送信できるメッセージの一つで、ボタンとか画像とか色々含めたカードのようなUI。
JSON形式で定義する。

[Adaptive Cardのサイト](https://adaptivecards.io/)

Bot Framework SDK においては、コードでカードを作ることもできるし、JSONをパースしてカードを作ることもできる。

## JSONからカードを作る

[Adaptive Card の Designer](https://adaptivecards.io/designer/) で、カードを作る。

左上の「Select host app:」の部分が「Bot Framework WebChat」になっていることを確認する。
また、右上の「Target Version」はおそらく最新バージョンが選択されるようになっているが、
Bot Framework WebChatがサポートしているバージョンがそれより低い場合、「Target Version」を合わせる必要がある。
私が試したとき、Target Version は1.3で、Bot Framework WebChatがサポートしているバージョンは1.2だった。
その状態で作ったJSONをボットに読み込ませても、空のカードになる。

作ったカードのJSONは、ボットのプロジェクトへjsonファイルとして配置する。
今回は、`Cards/AdaptiveCard.json` に置いたと仮定する。

そのjsonファイルを読み込んで送信するサンプルが下記。

```csharp
var paths = new[] { ".", "Cards", "AdaptiveCard.json" };
var adaptiveCardJson = File.ReadAllText(Path.Combine(paths));

var adaptiveCardAttachment = new Attachment()
{
    ContentType = "application/vnd.microsoft.card.adaptive",
    Content = JsonConvert.DeserializeObject(adaptiveCardJson),
};

var message = MessageFactory.Attachment(adaptiveCardAttachment);

await turnContext.SendActivityAsync(message, cancellationToken);
```

---
title: "Teams"
date: 2020-12-17T10:47:10+09:00
draft: true
---

[How bots for Microsoft Teams work - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-basics-teams?view=azure-bot-service-4.0&tabs=csharp)

ボットを実装する際、`ActivityHandler` の代わりに `Microsoft.Bot.Builder.Teams.TeamsActivityHandler` を拡張する。
`TeamsActivityHandler` には、Teams用のイベントハンドラメソッドが定義されている。


Teamsのユーザー情報を取得する
https://docs.microsoft.com/ja-jp/microsoftteams/platform/bots/how-to/get-teams-context?tabs=dotnet
---
title: "Virtual Assistant"
date: 2020-11-04T15:04:39+09:00
draft: true
---

https://microsoft.github.io/botframework-solutions/virtual-assistant/tutorials/create-assistant/csharp/1-intro/

Bot Service のテンプレート。
https://marketplace.visualstudio.com/items?itemName=BotBuilder.VirtualAssistantTemplate
VS 2017, 2019 で使える。
VS Code の場合は、GitHubからソースをダウンロードする必要あり。
https://github.com/microsoft/botframework-solutions/tree/master/samples/csharp/assistants/virtual-assistant/VirtualAssistantSample

カスタマイズする。

まずローカルでテストしたい場合は、StorageをMemoryStorageにする。

Startup.cs

```cs
// Configure storage
// Uncomment the following line for local development without Cosmos Db
 services.AddSingleton<IStorage, MemoryStorage>();
// services.AddSingleton<IStorage>(new CosmosDbPartitionedStorage(settings.CosmosDb));
```


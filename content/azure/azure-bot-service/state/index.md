---
title: "State (記憶) の管理"
date: 2020-10-02T12:04:50+09:00
lastMod: 2020-10-15T14:49:00+09:00
weight: 6
---

## Stateの保存場所

ボットは、Webアプリケーションと同じように基本的にはステートレスである。
1回目のターンでやりとりした内容は、基本的には2回目ではもう覚えていない。
しかし、より充実した機能を提供するには「今までどんな会話をしてきたか」を覚えなくてはいけない場合がある。

そういった情報を保存する場所は、インメモリとかDBとかどこでも良いが、
Bot Framework SDKには以下のストレージに対する実装がある:

* Memory storage - テスト用のインメモリのストレージ。ローカルでボットのテストをしたいときに使う。ボットが再起動すると消える。
* Azure Blob Storage
* Azure Cosmos DB

## Stateのスコープ

Stateのスコープは3つある。

* User state - ユーザーとボットの間でずっと有効なスコープ。
* Conversation state - 特定の会話で有効なスコープ。ユーザーは問わない(例えばグループ会話)
* Private conversation state - 特定の会話とユーザーで有効なスコープ。

ユーザーと会話は1つのチャネル内でのみ認識できる。同じユーザーが異なるチャネルからボットにアクセスした場合、異なるユーザーと判断する。

## 実装サンプル
参考ドキュメント：[Save user and conversation data - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-v4-state?view=azure-bot-service-4.0&tabs=csharp)  
サンプルソース：[BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management at main · microsoft/BotBuilder-Samples](https://github.com/microsoft/BotBuilder-Samples/tree/main/samples/csharp_dotnetcore/45.state-management)

このサンプルでは、conversation state と user state の2種類のスコープを扱う。

### データを格納するクラスの作成

まず最初に、保存したいデータを保持するためのクラスを作成する。

```csharp
// user stateで保持するデータ。ユーザー名などが相当する。
public class UserProfile
{
    public string Name { get; set; }
}
```

```csharp
// conversation state で保存するデータ。
public class ConversationData
{
    // The time-stamp of the most recent incoming message.
    public string Timestamp { get; set; }

    // The ID of the user's channel.
    public string ChannelId { get; set; }

    // Track whether we have already asked the user's name
    public bool PromptedUserForName { get; set; } = false;
}
```

### storageクラスなどの準備
次に、`Startup.cs` の `ConfigureServices` メソッドに、下記の処理を追加する。今回はローカルでのテスト用の `MemoryStorage` を使用する。
Azure Blob Storage などは別のクラスを使う。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Bot States
    var storage = new MemoryStorage();

    services.AddSingleton(new UserState(storage));
    services.AddSingleton(new ConversationState(storage));
}
```

### ボットクラスでデータを扱う
ボットクラスのサンプルは以下の通り。

```csharp
private BotState _conversationState;
private BotState _userState;

public StateManagementBot(ConversationState conversationState, UserState userState)
{
    // コンストラクタで、各stateごとのオブジェクトを引数で受け取る。
    _conversationState = conversationState;
    _userState = userState;
}

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{

    // accessorを取得→accessor.GetAsyncを呼び出し で保存データを取得できる
    var conversationStateAccessors =  _conversationState.CreateProperty<ConversationData>(nameof(ConversationData));
    var conversationData = await conversationStateAccessors.GetAsync(turnContext, () => new ConversationData());

    var userStateAccessors = _userState.CreateProperty<UserProfile>(nameof(UserProfile));
    var userProfile = await userStateAccessors.GetAsync(turnContext, () => new UserProfile());

    // (略)

    // 保存する値を変えるときは、オブジェクトに値をセットすればよい
    conversationData.Timestamp = localMessageTime.ToString();
    conversationData.ChannelId = turnContext.Activity.ChannelId.ToString();
}

public override async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    await base.OnTurnAsync(turnContext, cancellationToken);

    // ターンが終わる直前に、データを保存する
    await _conversationState.SaveChangesAsync(turnContext, false, cancellationToken);
    await _userState.SaveChangesAsync(turnContext, false, cancellationToken);
}
```


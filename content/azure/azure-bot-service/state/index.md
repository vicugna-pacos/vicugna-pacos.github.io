---
title: "記憶の管理 (State)"
date: 2020-10-02T12:04:50+09:00
lastMod: 2021-02-09T15:32:26+09:00
weight: 6
---

## Stateの保存場所

ボットは、Webアプリケーションと同じように基本的にはステートレスである。
1回目のターンでやりとりした内容は、基本的には2回目ではもう覚えていない。
しかし、より充実した機能を提供するには「今までどんな会話をしてきたか」を覚えなくてはいけない場合がある。

そういった情報を保存する場所として、Bot Framework SDKには以下の3つが用意されている：

* Memory Storage - テスト用のインメモリのストレージ。ローカルでボットのテストをしたいときに使う。ボットが再起動すると消える。
* Azure Blob Storage
* Azure Cosmos DB

## Stateのスコープ

Stateのスコープは3つ用意されている。

* User state - ユーザーとボットの間でずっと有効なスコープ。
* Conversation state - 特定の会話で有効なスコープ。ユーザーは問わない(例えばグループ会話)
* Private conversation state - 特定の会話とユーザーで有効なスコープ。

ユーザーと会話は1つのチャネル内でのみ認識できる。同じユーザーが異なるチャネルからボットにアクセスした場合、異なるユーザーと判断する。

## 実装サンプル
参考ドキュメント：[Save user and conversation data - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-v4-state?view=azure-bot-service-4.0&tabs=csharp)  
サンプルソース：[BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management at main · microsoft/BotBuilder-Samples](https://github.com/microsoft/BotBuilder-Samples/tree/main/samples/csharp_dotnetcore/45.state-management)

このサンプルでは、conversation state と user state の2種類のスコープを扱う。

### データを格納するクラスの作成
まず最初に、保存したいデータを保持するためのクラスを作成する。この工程は必須ではないが、クラスを作成しておくと管理が楽になると思われる。

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
    public string Timestamp { get; set; }

    public string ChannelId { get; set; }

    public bool PromptedUserForName { get; set; } = false;
}
```

### Storageクラスなどの準備
次に、`Startup.cs` の `ConfigureServices` メソッドに下記の処理を追加して、ストレージやStateをDIに登録する。今サンプルではテスト用の `MemoryStorage` を使用する。
MemoryStorage はデータをただのメモリに保持するだけなので、アプリが再起動すると消えてしまう。
本番環境では必ず Azure Blob Storage などを使うこと。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<IStorage, MemoryStorage>();
    services.AddSingleton<UserState>();
    services.AddSingleton<ConversationState>();
}
```

### TurnContext に登録する
BotAdapter クラス (AdapterWithErrorHandler.cs) のコンストラクタに下記を追加し、TurnContext に各StateクラスとStorageクラスを登録する。
こうすると、State を DI で注入してもらわなくても、TurnContext さえあれば State などを使用できるようになる。  
※ この手順は Adaptive Dialog を使えるようにする手順と重複しているため、そちらが済んでいるのなら改めて実装する必要はない。

```cs {hl_lines=[4,7,8,9]}
public class AdapterWithErrorHandler : BotFrameworkHttpAdapter
{
    public AdapterWithErrorHandler(IConfiguration configuration, ILogger<BotFrameworkHttpAdapter> logger
        , IStorage storage, UserState userState, ConversationState conversationState)
        : base(configuration, logger)
    {
        this.UseStorage(storage);
        this.UseBotState(userState);
        this.UseBotState(conversationState);

        OnTurnError = async (turnContext, exception) =>
        {
            // 略
        };
    }
}
```

#### TurnContext から取得する
Bot クラス等からStateを使いたい場合は、下記のように記述する。

```cs {hl_lines=[9,10]}
using Microsoft.Bot.Builder;
using System.Threading;
using System.Threading.Tasks;

public class EmptyBot : ActivityHandler
{
    public override async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default)
    {
        var storage = turnContext.TurnState.Get<IStorage>(nameof(IStorage));
        var userState = turnContext.TurnState.Get<UserState>(typeof(UserState).FullName);
    }
}
```

Storage クラスは `nameof(IStorage)` がキーになっているが、各 State クラスは `typeof(UserState).FullName` と、名前空間も含めたクラス名がキーになっている点に注意。

### State を読み書きする
State クラスへデータを読み書きするには、`IStatePropertyAccessor` インターフェイスを経由して行う。
ボットクラスのサンプルは以下の通り。

```csharp
private BotState _conversationState;
private BotState _userState;

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

State はターンが終わるまでにデータを保存しないといけない。
DialogManager クラスを使用している場合は、ConversationState と UserState に限り自動的に保存される。
それ以外の場合は、`AutoSaveStateMiddleware` というミドルウェアの使用を検討すると良い。
このミドルウェアはコンストラクタに指定した BotState をターンの最後に自動で保存してくれる。
ミドルウェアの使い方については、[こちらの記事]({{<ref "azure/azure-bot-service/middleware/index.md">}}) を参照。

## Azure Blob Storage を使う

### ストレージアカウントの準備
State の保存場所に Azure Blob Storage を使う場合、まずストレージアカウントをAzureに作成する。
作成後、ストレージアカウントのリソースを表示し、左側メニューの「設定」→「アクセスキー」をクリック。

key1 または key2 の接続文字列をコピーする。コピーした接続文字列は secrets.json へ貼り付ける。

```json
{
  "storage": {
    "connectionString": "接続文字列を貼り付ける",
    "containerName": "bot-state"
  }
}
```

`containerName` は好きな名前を付ける。ボット動作時に自動的に作成されるため、コンテナをあらかじめ作成しておく必要はない。

### Startup.cs の編集

NuGet パッケージ `Microsoft.Bot.Builder.Azure.Blobs` を追加する。  
※ `Microsoft.Bot.Builder.Azure` を追加して `AzureBlobStorage` クラスを使おうとしたら非推奨といわれた。

ConfigureService メソッドに下記を追加する。

```cs
using Microsoft.Bot.Builder.Azure.Blobs;

// 略

var storage = new BlobsStorage(Configuration["storage:connectionString"], Configuration["storage:containerName"]);
services.AddSingleton<IStorage>(storage);
services.AddSingleton<UserState>();
services.AddSingleton<ConversationState>();
```


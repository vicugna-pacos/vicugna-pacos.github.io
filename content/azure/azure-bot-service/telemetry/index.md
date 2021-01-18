---
title: "App Insights でログを記録"
date: 2021-01-15T15:42:29+09:00
weight: 11
---

## はじめに

参考：

* [Add telemetry to your bot - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-telemetry)

Azure に Web アプリを置く場合、ログは Application Insights に記録するのが一つの方法である。
Application Insights にログを記録しておくと、あとから検索・分析ができたりする。
Bot Framework SDK にも、Application Insights へログを記録する機能が備わっていて、後述の編集を行うと大体必要なログが自動的に記録される。

ちなみに、Azure ポータルサイトで Web アプリボットのリソースを作成する際、「Application Insights を有効にする」かどうかのオプションがある。
しかし、これを有効にした際に記録されるログはあまり役に立たない気がするので、リソース作成時は無効にしておき、あとで自分で Application Insights のリソースを作成しておくとよい。

また、ボットが QnAMaker を使っている場合、QnAMaker のリソース作成時にも Application Insights を有効するかどうかのオプションがある。
本記事の手順で QnAMaker の結果もログ記録できるので、QnAMaker 側の Application Insights は必要ないと思われる。

## 環境設定

### NuGet パッケージの追加

`Microsoft.Bot.Builder.Integration.ApplicationInsights.Core` を追加する。

### Startup.cs の編集
Startup.cs の ConfigureServices メソッドに下記を追加する。

```cs
public void ConfigureServices(IServiceCollection services)
{
    ...
        // Add Application Insights services into service collection
        services.AddApplicationInsightsTelemetry();

        // Create the telemetry client.
        services.AddSingleton<IBotTelemetryClient, BotTelemetryClient>();

        // Add telemetry initializer that will set the correlation context for all telemetry items.
        services.AddSingleton<ITelemetryInitializer, OperationCorrelationTelemetryInitializer>();

        // Add telemetry initializer that sets the user ID and session ID (in addition to other bot-specific properties such as activity ID)
        services.AddSingleton<ITelemetryInitializer, TelemetryBotIdInitializer>();

        // Create the telemetry middleware to initialize telemetry gathering
        services.AddSingleton<TelemetryInitializerMiddleware>();

        // Create the telemetry middleware (used by the telemetry initializer) to track conversation events
        services.AddSingleton<TelemetryLoggerMiddleware>();
    ...
}
```

#### ユーザーが送ったメッセージをログに含める
上記の通りに実装すると、アクティビティの送受信がログに記録されるようになるが、ユーザーが送ってきたメッセージやユーザーIDなど、個人的な情報は既定では記録されない。
それらを有効にするには、`TelemetryLoggerMiddleware` のインスタンス生成時の引数の指定を追加する。

```cs
// 変更前
services.AddSingleton<TelemetryLoggerMiddleware>();
// ↓
// 変更後
services.AddSingleton<TelemetryLoggerMiddleware>(sp =>
{
    var telemetryClient = sp.GetService<IBotTelemetryClient>();
    return new TelemetryLoggerMiddleware(telemetryClient, logPersonalInformation: true);
});
```

このオプションを有効にする場合、ボットがメッセージやユーザーIDなどを収集することを、あらかじめユーザーに通知しておくとよさそう。
ボットのウェルカムメッセージにその旨を載せておくのが簡単かと思われる。

### AdapterWithErrorHandler.cs の編集
AdapterWithErrorHandler.cs のコンストラクタ引数に `TelemetryInitializerMiddleware` を追加し、`Use` メソッドの呼び出しを追加する。

```cs {hl_lines=2,6}
public AdapterWithErrorHandler(IConfiguration configuration, ILogger<BotFrameworkHttpAdapter> logger, ConversationState conversationState = null
    , TelemetryInitializerMiddleware telemetryInitializerMiddleware
    ) : base(configuration, logger)
{
    ...
    Use(telemetryInitializerMiddleware);
}
```

### appsettings.json の編集
appsettings.json またはユーザーシークレットに、Application Insights のインストルメンテーションキーを追加する。

```json
{
    "ApplicationInsights": {
        "InstrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
}
```

これで、ログを Application Insights に記録する準備が整った。

## Dialog に Application Insights を追加
ComponentDialog や AdaptiveDialog のコンストラクタ引数に `IBotTelemetryClient` を追加してフィールド変数に代入しておくと、Dialog のログが出力されるようになる。

```cs
public MainDialog(IConfiguration configuration, ILogger<MainDialog> logger, IBotTelemetryClient telemetryClient)
    : base(nameof(MainDialog))
{
    this.TelemetryClient = telemetryClient;
    ...
}
```

## Recognizer に Application Insights を追加
AdaptiveDialog で使う QnAMaker や LUIS の Recognizer に `IBotTelemetryClient` を渡すと、QnAMaker や LUIS の検証結果がログ出力されるようになる。

下記は、AdaptiveDialog のコンストラクタで、Recognizer に `IBotTelemetryClient` を渡しているサンプル。

```cs {hl_lines=[3,16,23]}
public RootDialog(IConfiguration configuration, IBotTelemetryClient telemetryClient) : base(nameof(RootDialog))
{
    this.TelemetryClient = telemetryClient;

    Recognizer = new RecognizerSet()
    {
        Recognizers =
        {
            new QnAMakerRecognizer()
            {
                HostName = configuration["qna:hostname"],
                EndpointKey = configuration["qna:endpointKey"],
                KnowledgeBaseId = configuration["qna:KnowledgeBaseId"],
                QnAId = "turn.qnaIdFromPrompt",
                IncludeDialogNameInMetadata = false,
                TelemetryClient = telemetryClient
            },
            new LuisAdaptiveRecognizer()
            {
                ApplicationId = configuration["luis:applicationId"],
                EndpointKey = configuration["luis:endpointKey"],
                Endpoint = configuration["luis:endpoint"],
                TelemetryClient = telemetryClient
            }
        }
    };
    // 略
}
```

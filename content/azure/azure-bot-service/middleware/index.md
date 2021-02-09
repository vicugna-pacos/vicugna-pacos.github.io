---
title: "ミドルウェア"
date: 2021-02-09T13:33:23+09:00
---

## ミドルウェアを使う

BotAdapter クラス (AdapterWithErrorHandler.cs) のコンストラクタで `Use` メソッドを実行する。

```cs {hl_lines=[7]}
public class AdapterWithErrorHandler : BotFrameworkHttpAdapter
{
    public AdapterWithErrorHandler(IConfiguration configuration, ILogger<BotFrameworkHttpAdapter> logger
        , IStorage storage, UserState userState, ConversationState conversationState)
        : base(configuration, logger)
    {
        this.Use(new AutoSaveStateMiddleware(userState, conversationState));
    }
}
```

## ビルトインのミドルウェア
AutoSaveStateMiddleware - コンストラクタに指定した BotState をターンの最後に自動で保存してくれる。DialogManager を使っている場合、ConversationState と UserState は DialogManager で自動保存されているので、このミドルウェアに自動保存させる必要はない。

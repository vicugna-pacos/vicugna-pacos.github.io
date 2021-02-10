---
title: "ボットからメッセージを送る (プロアクティブなメッセージ)"
date: 2020-12-17T15:42:46+09:00
weight: 9
---

## 前提条件

* Windows 10
* Bot Framework SDK v4
* .NET Core C#

## ボット側からメッセージを送る
チャットボットは、基本的にユーザーから会話が始まりボットは返事をするだけだが、ボット側からメッセージを送ることもできる。
ボットから話しかけることを、「プロアクティブな」メッセージという。

基本的には、ボットに接続したことのあるユーザーにしか話しかけることができない。
ただし、Teamsの場合、組織単位でボットをアプリとして許可していれば、組織内の一度も会話したことのないユーザーへ話しかけられるらしい。
その際は、Graph API を使ってユーザー情報を取得するらしい。

参考：

* [Send proactive notifications to users - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-howto-proactive-message?view=azure-bot-service-4.0&tabs=csharp)
* [Send proactive messages - Teams | Microsoft Docs](https://docs.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/send-proactive-messages?tabs=dotnet)

## ConversationReference を保存
ユーザーがボットに接続すると、`ConversationReference` という情報が作られる。これを保存しておけば、接続済みのユーザーに対してメッセージを送れる。
保存する際は IStorage クラスを保管場所にしておくといい。ストレージにDBなどを使っていれば、一緒に永続化される。

下記サンプルでは、保存の処理をAdaptive Dialog の CodeAction のメソッドとして定義している。
このメソッドをユーザーが会話に参加したときに実行するとよい。
なお、このサンプルはボットのチャネルはTeamsであることを大前提としている。

```cs
private async Task<DialogTurnResult> SaveConversationReference(DialogContext dc, object options)
{
    var member = await TeamsInfo.GetMemberAsync(dc.Context, dc.Context.Activity.From.Id, default);
    if (member != null)
    {
        var email = member.Email;
        var value = dc.Context.Activity.GetConversationReference();

        var storage = dc.Context.TurnState.Get<IStorage>();
        IDictionary<string, object> changes = new Dictionary<string, object>
        {
            { $"ConversationReference_{email}", value }
        };

        await storage.WriteAsync(changes);
    }

    return await dc.EndDialogAsync(options);
}
```

このサンプルでは、保存のキーにメールアドレスを使用している。
特定の個人に宛ててプロアクティブなメッセージを送りたいので、個人ごとにキーを分けていて、なおかつ外部から指定のしやすいメールアドレスを採用した。
対して、例えばボットに接続しているユーザー全員に、通知のようなイメージでプロアクティブなメッセージを送りたい場合は、
自動で振られるユーザーIDをキーとしてもいい。

#### ConversationReference の削除
メンバーがボットから切断したら ConversationReference を削除するのが順当だが、`OnMembersRemovedAsync` のイベントが呼び出されるかどうかはチャネルによって違うため、
環境によっては保存した ConversationReference が削除されずにずっと残るかもしれない。

## メッセージの受け口を作る
次に、プロアクティブなメッセージを受け付ける場所を作る。
ボットから発言するといっても、実態はWebアプリなので勝手には動き出さない。
トリガーは外部にあるため、メッセージを受け付けるための Controller をボットに用意する。

### Controller を追加
サンプルでは、URL `api/proactive` を追加して、パラメータでメッセージ送信先のメールアドレスを受け取る。
メールアドレスに一致する ConversationReference がある場合、そのユーザーに対してメッセージを送るなどの処理を行う。

```cs
using DemoBot1.Models;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Bot.Schema;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.Logging;
using Microsoft.Extensions.Primitives;
using System;
using System.Net.Mime;
using System.Threading;
using System.Threading.Tasks;

namespace DemoBot1.Controllers
{
    [Route("api/proactive")]
    [ApiController]
    public class ProactiveController : ControllerBase
    {
        private readonly IBotFrameworkHttpAdapter _adapter;
        private readonly string _appId;
        private readonly ILogger _logger;
        private readonly IStorage _storage;

        public ProactiveController(
            IBotFrameworkHttpAdapter adapter,
            IConfiguration config,
            ILogger<ProactiveController> logger, 
            IStorage storage)
        {
            _adapter = adapter;
            _appId = config["MicrosoftAppId"];
            _logger = logger;
            _storage = storage;
        }

        [HttpPost]
        [Consumes(MediaTypeNames.Application.Json)]
        [ProducesResponseType(StatusCodes.Status400BadRequest)]
        [ProducesResponseType(StatusCodes.Status404NotFound)]
        public async Task<IActionResult> PostAsync(ProactiveParam param)
        {
            if (StringValues.IsNullOrEmpty(param.Email))
            {
                _logger.LogWarning("クエリパラメータの email が指定されていません。");
                return BadRequest();
            }

            string key = $"ConversationReference_{param.Email}";
            var values = await _storage.ReadAsync(new[] { key });
            ConversationReference value = null;
            if (values.ContainsKey(key))
            {
                value = (ConversationReference) values[key];
            }

            if (value == null)
            {
                _logger.LogWarning($"{param.Email} の ConversationReference は保存されていません。");
                return NotFound();
            }

            await ((BotAdapter)_adapter).ContinueConversationAsync(_appId, value, BotCallback, default);

            return Ok();
        }

        private Task BotCallback(ITurnContext turnContext, CancellationToken cancellationToken)
        {
            // 何か処理をする
            throw new NotImplementedException();
        }
    }
}
```

### テストする
ボットを起動し、Teams で接続する。ConversationReference が保存されたはずなので、PowerShell で下記サンプルを実行してメッセージを送ってみる。
200 OK が返ってきて、ボットが何かメッセージを発言したりすれば実装が上手くいっている。

```ps
$url = "http://localhost:3978/api/proactive"
$body = @{
    "Email" = "xxx@example.com"
}
$bodyStr = ConvertTo-Json $body
$bodyBytes = [Text.Encoding]::UTF8.GetBytes($bodyStr)

Invoke-WebRequest $url -Body $bodyBytes -Method "Post" -ContentType "application/json"
```

## メッセージ送信制限に気を付ける
外部からのリクエストに応じるプロアクティブなメッセージの場合、一度に大量のリクエストが送られるとボットからのメッセージも大量になる。
ボットのチャネルは、それぞれ一定時間に送れるメッセージ量を制限していることがある。
下記に Teams での制限に対してどのように実装したらよいかのサンプルが載っている。

[Rate limiting - Teams | Microsoft Docs](https://docs.microsoft.com/en-us/microsoftteams/platform/bots/how-to/rate-limit)

一定時間待機してからリトライ、という感じ。

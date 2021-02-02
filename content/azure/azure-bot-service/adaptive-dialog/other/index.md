---
title: "その他いろいろ"
date: 2021-01-08T13:27:54+09:00
---

## CodeAction を使う
Adaptive Dialog の Action 部分は Dialog クラスをリストにしたりネストにしたりして実装するが、
処理が複雑でそれだと実装しづらい場合は、`Microsoft.Bot.Builder.Dialogs.Adaptive.Actions.CodeAction` クラスを使って自分でコードを書ける。
`CodeAction` のコンストラクタに実行させたいメソッドを指定すればよい。

```cs
public class RootDialog : AdaptiveDialog
{
    public RootDialog() : base(nameof(RootDialog))
    {
        Triggers = new List<OnCondition>
        {
            new OnConversationUpdateActivity()
            {
                Actions = 
                {
                    new CodeAction(WelcomeUser)
                }
            },
            new OnUnknownIntent
            {
                // 略
            },
        };
    }

    private static async Task<DialogTurnResult> WelcomeUser(DialogContext dc, object options)
    {
        // ここに処理を書く
    }
}
```

## Welcome メッセージを追加する
ボットから DialogManager を呼び出す処理を `OnTurnAsync` メソッドに書いたことから分かるように、Adaptive Dialog は全てのイベント、アクティビティに対して処理できる。
ユーザーがボットに初めて接続したときの `OnMembersAddedAsync` メソッドの処理も Adaptive Dialog へ移せる。

サンプルその1を以下に示す。

```cs
public class RootDialog : AdaptiveDialog
{
    public RootDialog() : base(nameof(RootDialog))
    {
        Triggers = new List<OnCondition>
        {
            new OnConversationUpdateActivity()
            {
                Actions = WelcomeUserSteps()
            },
            new OnUnknownIntent
            {
                // 略
            },
        };
    }

    private static List<Dialog> WelcomeUserSteps()
    {
        return new List<Dialog>()
        {
            new Foreach()
            {
                ItemsProperty = "turn.activity.membersAdded",
                Actions = new List<Dialog>()
                {
                    new IfCondition()
                    {
                        Condition = "$foreach.value.name != turn.activity.recipient.name",
                        Actions = new List<Dialog>()
                        {
                            new SendActivity("Hello!")
                        }
                    }
                }
            }
        };
    }
}
```

`WelcomeUserSteps` メソッドの内容が Action にあたる。Foreach とか If まで Action クラスになっていて、それらをネストしたりしながら使っていく。
ただこれだとかえって何をやっているかわかりづらいし、ちょっとしたロジックなのに煩雑にみえる。
そういう場合、`CodeAction` を使うと良い。

```cs
public class RootDialog : AdaptiveDialog
{
    public RootDialog() : base(nameof(RootDialog))
    {
        Triggers = new List<OnCondition>
        {
            new OnConversationUpdateActivity()
            {
                Actions = 
                {
                    new CodeAction(WelcomeUser)
                }
            },
            new OnUnknownIntent
            {
                // 略
            },
        };
    }

    private static async Task<DialogTurnResult> WelcomeUser(DialogContext dc, object options)
    {
        foreach (var member in dc.Context.Activity.MembersAdded)
        {
            if (member.Name != dc.Context.Activity.Recipient.Name)
            {
                var message = MessageFactory.Text("Hello!");
                await dc.Context.SendActivityAsync(message);
            }
        }
        return await dc.EndDialogAsync(options);
    }
}
```


---
title: "Language Generator"
date: 2021-01-08T13:27:54+09:00
lastMod: 2021-07-02T16:08:15+09:00
---

## はじめに

参考：
* [Generators in adaptive dialogs - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-concept-adaptive-dialog-generators)
  * Language Generator とは、の説明が載っている
* [.lg file format - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/file-format/bot-builder-lg-file-format)
  * lg ファイルの基本的な書き方が載っている
* [Adaptive expressions prebuilt functions - Bot Service | Microsoft Docs](https://docs.microsoft.com/en-us/azure/bot-service/adaptive-expressions/adaptive-expressions-prebuilt-functions)
  * lg ファイル内で利用できる関数が載っている

Language Generator は、ボットからのメッセージをテンプレート化する仕組み。
ボットのセリフをリソースファイルに隔離して管理できる。
リソースファイルには条件式の指定やStateの参照が可能で、より自然なボットのセリフを構築できる。

リソースファイルの拡張子は `.lg` で、通常は利用する Dialog クラスと1対1で作成し、Dialog クラスと同じフォルダに配置する。
lgファイルのプロパティを変更し、「出力ディレクトリにコピー」するようにしておくこと。
文字コードは、UTF-8-BOM にする(BOM無しでも大丈夫っぽい)。

サンプルとして、 Dialogs/RootDialog.lg を作成した。

```markdown
# Nothing
- nothing1!
- nothing2!
```

次に RootDialog をLGを使うように変更する。

```cs {hl_lines=[18,"23-26"]}
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
            new OnUnknownIntent()
            {
                Actions =
                {
                    new SendActivity("${Nothing()}")
                }
            },
        };

        string[] paths = { ".", "Dialogs", "RootDialog.lg" };
        string fullPath = Path.Combine(paths);

        Generator = new TemplateEngineLanguageGenerator(Templates.ParseFile(fullPath));
    }
    // 略
}
```

これでボットをテストすると、「nothing1!」か「nothing2!」のいずれかを返すようになる。

## CodeAction 内でLGを使う
Adaptive Dialog の Action 類からLGを使うときは `${テンプレート名()}` で良いが、自分で Action の内容を実装したときにLGを使う方法を示す。

```cs
private static async Task<DialogTurnResult> OriginalAction(DialogContext dc, object options)
{
    var template = new ActivityTemplate("${Greeting()}");
    var message = await template.BindAsync(dc, dc.State);
    await dc.Context.SendActivityAsync(message);
}
```

## 条件式や関数
LGファイルに以下のような定義を追加し、時間帯に合わせて挨拶を変えることができたりする。

```md
# timeOfDay
- ${getTimeOfDay(convertFromUTC(utcNow(), 'Asia/Tokyo'))}

# Greeting
- IF: ${timeOfDay() == 'morning'}
  - おはよう！
- ELSEIF: ${timeOfDay() == 'afternoon'}
  - こんにちは！
- ELSE:
  - こんばんは！
```

Dialog の方で `${Greeting()}` を参照すれば、朝なら「おはよう」、昼なら「こんにちは」と喋らせることができる。
`Greeting` のIF文で `timeOfDay` を参照しているように、LGのあるテンプレートから他のテンプレートを参照することが可能。
変数を定義＆参照するような感覚で使える。

## Adaptive Card をつかう
`.lg` ファイルには Adaptive Card の付いたメッセージも定義できる。
まずカードの json をテンプレートとして定義する。

````md
# SampleCard
- ```
{
    "type": "AdaptiveCard",
    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
    "version": "1.3",
    "body": [
        {
            "type": "TextBlock",
            "text": "Hello!",
            "wrap": true
        }
    ]
}
```
````

次に、カードの json を Attachment として取り込むテンプレートを定義する。
```markdown
# Greeting
[Activity
    Attachments = ${json(SampleCard())}
]
```

SendActivity のアクションで、このテンプレート(`Greeting`)を指定するとカードが送られる。

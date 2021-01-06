---
title: "Adaptive Card Blog"
date: 2021-01-06T10:35:41+09:00
draft: true
---

https://blog.botframework.com/2019/07/02/using-adaptive-cards-with-the-microsoft-bot-framework/

## はじめに
Adaptive Cards はチャットクライアントなどにおいて、自己完結型のUIを表示する仕組みです。
Bot Framework の rich card のほぼすべての機能を持っていて、なおかついくつかの独自機能も備えています。
Web Chat、コルタナ、Teamsでサポートされていて、さらにはWindows Timeline や Outlook Actionable Messages でもサポートされています。
さらには、ユーザー独自のアプリケーションでも描画することができます。
Adaptive Cards はどの環境でも使用できるように作られたものです。もしこれが初めて Adaptive Card に触れる機会なのであれば、この記事が助けになるでしょう。
そしてすでに Adaptive Cards を使っている場合は、いくつかの新しいトリックを教えられるでしょう。

### Schema
すべての Adaptive Card は、特定のスキーマで定義されたjsonオブジェクトであらわされます。
スキーマには Adaptive Card に配置できるすべてが定義されています。
すべてのホストアプリケーションは、Adaptive Card のスキーマを適切なルールで解釈しなくてはいけません。しかし、Adaptive Cards の nature は、アプリケーションごとの柔軟性やバリエーションを許容しています。
スキーマエクスプローラを参照すれば、スキーマがどんなものか調べることができます。

現在、Adaptive Card のスキーマには2つのバージョンがあります(1.1 と 1.2)。スキーマは累積的になっているので、古いバージョンのカードを新しいバージョンで使用することができます。

Adaptive Card のスキーマを指定するには、スキーマのバージョンごとのパス (例：https://adaptivecards.io/schemas/1.2.0/adaptive-card.json) を指定します。
バージョンの入っていないパス https://adaptivecards.io/schemas/adaptive-card.json もありますが、こちらは常に最新バージョンのスキーマを指定しています。
下記に、Adaptive Card を JSON で表したサンプルを記載します。スキーマの指定も含まれています。

```json
{
  "$schema": "https://adaptivecards.io/schemas/1.1.0/adaptive-card.json",
  "type": "AdaptiveCard",
  "version": "1.0",
  "body": [
    {
      "type": "TextBlock",
      "text": "Example card"
    }
  ]
}
```

もしAdaptive Card のJSONが実際にどんな見た目になるか知りたい場合は、Adaptive Card designer へ JSON をコピー＆貼り付けしてみてください。

You can find more samples of Adaptive Cards here.

### Packages
Adaptive Cards は JSON で記述してHTTPのコンテンツとして送信できるため、いずれの言語で実装されたボットに対応できます。
しかしながら、Adaptive Cards をプログラムで扱う助けになるパッケージがいくつか用意されています。

もしボットを C# で実装している場合、AdaptiveCards Nuget パッケージが必要となるでしょう。古い方の Microsoft.AdaptiveCards は非推奨なので使用しないでください。
同じようにボットを Node.js で実装している場合は、adaptivecards を使用してください。microsoft-adaptivecards ではありません。
これらのパッケージにはクライアントサイドでAdaptive Card を描画するための機能が入っていますが、それらはボットサイドの実装でも役に立ちます。
例えば、Nuget パッケージに含まれている型を使えば、JSONを一から書かずに Adaptive Card を構築することができます。

```cs
var card = new AdaptiveCard(new AdaptiveSchemaVersion(1, 0))
{
    Body = new List<AdaptiveElement>()
    {
        new AdaptiveTextBlock("Example card"),
    },
};
```

### Body & Actions
Adaptive Card の body は、情報を表示しユーザーの入力を集める場所です。
カードのレイアウトを定義するのにコンテナを使うことができます。
カードエレメントは 例えばテキストブロックなどのAdaptive Card のコンポーネントのことです。
すべてのコンポーネントはスキーマから参照できます。

card action は カード上でクリックしたときに何かしら処理を発動できる箇所のことです。
他の rich cards とは違い、Adaptive Cards は以下のaction type をサポートしています：OpenUrl, ShowCard, Submit。
Adaptive Cards 1.2 では4つ目のアクションが追加されました: ToggleVisibility.

card action はカードのbodyで、elementがselectAction プロパティを持っているときに使用できます。
adaptive card はさらにbodyとは別にaction list を持っています。そして、それらはボタンとして表示されます。
adaptive card 1.2 では、actionset を使ってどのようにactionを表示させるか指定することができます。

## Tips & Tricks
adaptive card は bot framework のためだけに作られたものではありませんが、ボットにとって非常に便利であり、使用する際に知っておくべき重要なことがいくつかあります。

### Submit Actions
スキーマで Action.Submit を見ると、submit アクションのdataプロパティが文字列またはオブジェクトのいずれかであることがわかります。
submit アクションには、これら2つのデータ型に対応する2つの動作があります。
文字列を submit アクションの data プロパティとして使用する場合、submit アクションは「string submit action」と呼ばれる方法で動作します。
オブジェクトを submit アクションの data プロパティとして使用する場合、または data プロパティを省略した場合、submit アクションは「object submit action」と呼ばれる方法で動作します。

「string submit action」は、あたかもユーザーがメッセージを入力して手動で送信したかのように、クライアントアプリケーションの会話履歴に表示されるメッセージをユーザーからボットに自動的に送信します。
「object submit action」は、非表示のデータをユーザーからボットへ送信します。 
（リッチカードのアクションに例えると、文字列送信アクションとオブジェクト送信アクションは、それぞれimBackとpostBackと呼ばれるカードアクションにほぼ対応します。）

以下に、ユーザーが string submit action をクリックしたときにどうなるかを示します。

※ 画像

このときのカードのjsonは下記の通りです。submit action の data プロパティに文字列が指定されている点に注目してください。

```json
{
  "$schema": "https://adaptivecards.io/schemas/adaptive-card.json",
  "type": "AdaptiveCard",
  "version": "1.0",
  "actions": [
    {
      "type": "Action.Submit",
      "title": "Click here to say \"Hello\"",
      "data": "Hello"
    }
  ]
}
```

Adaptive card に入力フィールドが含まれている場合は object submit action になるため、data プロパティに文字列を指定できません。

Bot Framework を使用している場合、submit アクションをクリックすると、message activity がボットに送信されます。
string submit action は、その文字列データをアクティビティの text プロパティに送信するだけです。
object submit action はやや複雑で、textプロパティは空で、アクティビティのvalueプロパティを使用します。
object submit action のカードの各入力フィールドは、それぞれobjectのプロパティとなって送信されます。
たとえば、次のようなアダプティブカードがあるとします。

```json
{
  "$schema": "https://adaptivecards.io/schemas/adaptive-card.json",
  "type": "AdaptiveCard",
  "version": "1.0",
  "body": [
    {
      "type": "Input.Text",
      "id": "id_text"
    },
    {
      "type": "Input.Number",
      "id": "id_number"
    }
  ],
  "actions": [
    {
      "type": "Action.Submit",
      "title": "Submit",
      "data": {
        "prop1": true,
        "prop2": []
      }
    }
  ]
}
```

There are two input fields for the user to fill out:

画像

ユーザーがsubmitボタンをクリックしたとき、見えないメッセージactivityがユーザーからボットへ送信されます。activityのvalueプロパティには以下のようなオブジェクトが設定されます。

```json
{
  "id_number": "30",
  "id_text": "Kyle",
  "prop1": true,
  "prop2": []
}
```

2つの入力フィールド (id_number, id_text) と、submit action の json に書かれている2つのプロパティ (prop1, prop2) がオブジェクトに含まれています。
また、入力フィールドが数値であっても、文字列が送信される点に注意してください。入力フィールドの種類に関係なく、データは文字列として送信されるため、入力値のチェックが必要です。

object submit action はメッセージタイプのアクティビティを生成するため、そのようなアクティビティを、通常のテキストベースのメッセージアクティビティと区別する方法が必要になります。
ほとんどの場合、「valueプロパティが設定されている かつ textプロパティが設定されていない」かどうかで区別できます。
必要に応じて、値オブジェクト内のデータが期待を満たしているかどうかを確認することで、追加のチェックを実行できます。
ただし、通常、ボットが文字列送信アクションからのメッセージとユーザーがチャットクライアントに入力したメッセージを区別する方法はありません。
これらのメッセージは同じように扱われる必要があるため、これは仕様によるものです。

これらすべてを考慮に入れると、C＃では、次のようにアクティビティのvalueプロパティから数値を取得できます。

```cs
var txt = turnContext.Activity.Text;
dynamic val = turnContext.Activity.Value;
// Check if the activity came from a submit action
if (string.IsNullOrEmpty(txt) && val != null)
{
    // Retrieve the data from the id_number field
    var num = double.Parse(val.id_number);
    // . . .
}
```

In JavaScript you might do it like this:

```js
var txt = turnContext.activity.text;
var val = turnContext.activity.value;
// Check if the activity came from a submit action
if (!txt && val) {
    // Retrieve the data from the id_number field
    var num = parseFloat(val.id_number);
    // . . .
}
```

Other types of input can be retrieved in a similar fashion.

### Adaptive Cards in Dialogs

Dialogsは、Bot Framework の重要な部分です。
ダイアログは通常、あらかじめ決められた手順で会話が進んでいきますが、ユーザーが会話を「中断」することも考慮しなくてはいけません。
カードは会話履歴に残るように設計されているため、古くなったカードに対して後からアクションを起こすこともできてしまいます。これは Adaptive Card だけではなく他のカードにも当てはまります。

多くのチャネルは、会話の1ターンだけボタンを表示することでカードの問題を解決する、何らかの形の「suggested actions」をサポートしています。
ただし、カードを適切に使用すれば、これを問題と考える理由はまったくないかもしれません。
ユーザーが会話の前のポイントから古いカードをクリックできるようにしたい場合があります。古いカードに何もさせたくない場合は、ボットがそれらを無視することを選択できます。

プロンプトは非常に一般的な形式のダイアログです。プロンプトにより、ユーザー入力の検証と自動型変換が可能になります。
プロンプトが表示されると、ユーザーが適切な種類の情報を提供するまでダイアログは続行されません。
ダイアログでアダプティブカードを使用している場合は、プロンプトを使用することをお勧めします。

プロンプトに任意の種類のカードを含めるには、プロンプトで送信されるアクティビティにカードを添付するだけです。たとえば、次のようにC＃のウォーターフォールステップの選択プロンプトでアダプティブカードを使用できます。

```cs
private async Task<DialogTurnResult> PromptWithAdaptiveCardAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    // Define choices
    var choices = new[] { "One", "Two", "Three" };
    // Create card
    var card = new AdaptiveCard(new AdaptiveSchemaVersion(1, 0))
    {
        // Use LINQ to turn the choices into submit actions
        Actions = choices.Select(choice => new AdaptiveSubmitAction
        {
            Title = choice,
            Data = choice,  // This will be a string
        }).ToList<AdaptiveAction>(),
    };
    // Prompt
    return await stepContext.PromptAsync(
        CHOICEPROMPT,
        new PromptOptions
        {
            Prompt = (Activity)MessageFactory.Attachment(new Attachment
            {
                ContentType = AdaptiveCard.ContentType,
                // Convert the AdaptiveCard to a JObject
                Content = JObject.FromObject(card),
            }),
            Choices = ChoiceFactory.ToChoices(choices),
            // Don't render the choices outside the card
            Style = ListStyle.None,
        },
        cancellationToken);
}
```

The card (along with the message that gets sent when the user clicks on a choice) will look something like this:

画像

ここで行っていることについて注意すべき重要なことがいくつかあります。
カードをJObjectに変換する理由は、Bot Builder SDKがオブジェクトをメッセージにシリアル化するときに、型データを保持するためです。これにより、実際に送信する必要があるのはアダプティブカードスキーマに適合する生のJavaScriptオブジェクトだけであるため、送信する必要のない多くの追加情報を送信する可能性があります。シリアル化するクラスにカスタムのシリアル化/逆シリアル化機能がある場合、追加の型情報によって例外が発生することもあります。カードオブジェクトをJObjectに変換することは、特別な型情報が関連付けられていない、通常の生のjsonオブジェクトとそのプロパティが必要であるという言い方です。

また、StyleをListStyle.Noneとして設定していることにも注意してください。選択したリストスタイルに応じて、選択プロンプトにプロンプ​​トの選択肢が自動的に表示されます。たとえば、スタイルが「インライン」または「リスト」の場合、選択肢はアクティビティのテキストに直接追加されます。カード自体に選択肢がすでに含まれているため、ここでこれが発生することは望ましくありません。そのため、リストスタイルとして「なし」を設定しています。

アダプティブカードの入力フィールドをプロンプトに組み込みたい場合は、オブジェクト送信アクションのみを使用でき、文字列送信アクションは使用できません（[アクションの送信]セクションで説明されている制限のため）。ほとんどのプロンプトはアクティビティのテキストプロパティに基づいて動作し、オブジェクト送信アクションの場合はテキストが空になることを思い出してください。プロンプトにテキストなしのアクティビティからの値を受け入れるようにするために使用できるトリックは、ダイアログを続行する前に自分でテキストプロパティを設定することです。複数の入力フィールドがある場合は、JSON文字列にシリアル化することで、それらをテキストプロパティに組み合わせることができます。この場合、テキストプロンプトを使用することだけが意味があります。

着信アクティビティのtextプロパティに独自の値を割り当てた後、ユーザーがその情報を入力したかのようにプロンプ​​トが動作するように、そのアクティビティを効果的に変更します。その変更されたアクティビティがターンコンテキストに残っている限り、ターンコンテキストで行うことはすべて、その変更されたアクティビティを使用します。したがって、そのターンコンテキストをダイアログコンテキストに割り当ててから、アクティブなダイアログを続行すると、ダイアログはtextプロパティに入力したものをすべて使用します。 C＃では、次のようになります。

```cs
private async Task SendValueToDialogAsync(
    ITurnContext turnContext,
    CancellationToken cancellationToken)
{
    // Serialize value
    var json = JsonConvert.SerializeObject(turnContext.Activity.Value);
    // Assign the serialized value to the turn context's activity
    turnContext.Activity.Text = json;
    // Create a dialog context
    var dc = await _dialogSet.CreateContextAsync(
        turnContext, cancellationToken);
    // Continue the dialog with the modified activity
    await dc.ContinueDialogAsync(cancellationToken);
}
```

アクティブなダイアログがテキストプロンプトの場合、そのテキストプロンプトは、アダプティブカードの送信アクションからの情報を返します。

### Adaptive Cards in Teams

Microsoft Teamsチャネルに固有の特別なアダプティブカード機能がいくつかあります。この機能にアクセスするために、オブジェクト送信アクションのデータプロパティ内のオブジェクトにmsteamsという名前の特別なプロパティを指定できます。そうすることで、送信アクションを選択したアクションのように動作させることができます。 msteamsプロパティに配置するオブジェクトには、送信アクションの特別な動作を指定するためのtypeプロパティが必要です。

typeプロパティが「messageBack」の場合、送信アクションは、imBackとpostBackの組み合わせのようなmessageBackカードアクションのように動作します。これは、会話履歴に表示される表示テキストと、舞台裏でボットに送信される非表示データの両方を指定できることを意味します。

```json
{
  "type": "Action.Submit",
  "title": "Click me for messageBack",
  "data": {
    "msteams": {
        "type": "messageBack",
        "displayText": "I clicked this button",
        "text": "text to bots",
        "value": "{\"bfKey\": \"bfVal\", \"conflictKey\": \"from value\"}"
    },
    "extraData": {}
  }
}
```

displayText文字列は、会話履歴に表示される文字列です。テキスト文字列はアクティビティのテキストプロパティに入力されますが、ユーザーには表示されません。アクティビティのvalueプロパティは、入力フィールドの値をdatapropertyのオブジェクトの追加のプロパティと組み合わせることで通常の方法で入力されますが、msteamsプロパティのオブジェクトのvalueプロパティにシリアル化されたプロパティも含まれます。

typeプロパティが「imBack」の場合、送信アクションはimBackカードアクションのように動作します。これはもちろん文字列送信アクションと同様に機能しますが、カードに入力フィールドが含まれている場合に壊れないという利点があります。ただし、ユーザーがこのアクションをクリックしても入力フィールドの値はボットに送信されず、追加のプロパティも送信されません。データプロパティのオブジェクトを入力します。また、この記事の執筆時点では、Microsoft Teamsは文字列送信アクションをサポートしていないため、文字列送信アクションをシミュレートする場合は、この特別なTeams機能を使用する必要があります。

```json
{
  "type": "Action.Submit",
  "title": "Click me for imBack",
  "data": {
    "msteams": {
        "type": "imBack",
        "value": "Text to reply in chat"
    },
    "extraData": "(this will be ignored)"
  }
}
```

typeプロパティが「サインイン」の場合、送信アクションはサインインカードアクションのように動作します。サインインアクションは通常、非常に単純なカードで使用されるため、これはほとんどの場合必要ない場合があります。したがって、アダプティブカードよりもサインインカードを使用する方が理にかなっている場合があります。それでも、Microsoft Teamsのアダプティブカードにサインインアクションを含めたい場合は、msteamsプロパティのオブジェクトのvalueプロパティにサインインURLを入力するだけです。

```json
{
  "type": "Action.Submit",
  "title": "Click me for signin",
  "data": {
    "msteams": {
        "type": "signin",
        "value": "https://yoursigninurl.com/signinpath?parames=values",
    },
    "extraData": "(this will be ignored)"
  }
}
```

## Conclusion
アダプティブカードは多くのことを実行でき、多くの目的を果たします。最近、アダプティブカードに新機能が追加されており、今後も追加される可能性があります。また、人気が高まるにつれて、より多くのアプリケーションがアダプティブカードのサポートを強化する可能性があります。この投稿の情報を使用して、特にボットに組み込む場合に、アダプティブカードの使用方法を拡張できることを願っています。

Happy Making!

Kyle Delaney and the Azure Bot Service Team
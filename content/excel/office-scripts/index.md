---
title: "Office スクリプト"
date: 2021-09-06T16:12:21+09:00
---

## はじめに
Office スクリプトは、Web 版 Excel で使用できるスクリプトの仕組み。VBA の Web 版と捉えられる。
言語は TypeScript。

Office スクリプトを使うには、Office 365 の商用または教育向けライセンスが必要。もしOffice スクリプトを使えるなら、Web 版 Excel を開いたときにリボンに「自動化」タブが表示される。

![](2021-09-07-09-26-16.png)

表示されない場合、ライセンスを持っていないか、持っていたとしても組織の管理者が機能を制限している可能性がある。

Office スクリプトの利点：

* マクロを Web 版で実行できる
* Power Automate や Azure Logic Apps から呼び出せる。
  * Power Automate で Excel のデータを色々いじりたいときは、Office スクリプトに実装し、Power Automate から呼び出すだけにした方が効率がいい場合がある。さらには、細かい処理を Power Automate で一生懸命作るよりも、まるっと Office Script に書いてしまうという使い方もできるかもしれない。
* 一つのスクリプトを色々なブックへ使いまわせる。
  * スクリプトはユーザー個人ごとに保存され、それを色々なブックで使うイメージ。

Office スクリプトのデメリット：

* デスクトップアプリでは利用できない。
* 共有設定をしない限りスクリプト作成者本人しか使用できない。
  * スクリプトはブックには含まれず、ユーザー個人の OneDrive に保存されるため.

## 始め方
Web 版 Excel で新しいファイルを作成するか、OneDrive または SharePoint 上に作成した Excel ファイルを Web 版で開いてもいい。
リボンに「自動化」タブがあるので、クリックする。
画面右側に表れるウィンドウで、スクリプトの管理を行う。

## スクリプトの基本
[Scripting fundamentals for Office Scripts in Excel on the web - Office Scripts | Microsoft Docs](https://docs.microsoft.com/en-us/office/dev/scripts/develop/scripting-fundamentals)

### main 関数
すべてのスクリプトには、main 関数が必要。この関数がスクリプトのエントリーポイントになる。

```typescript
function main(workbook: ExcelScript.Workbook) {
  // ここに処理を書く
}
```
[Office Scripts API reference - Office Scripts | Microsoft Docs](https://docs.microsoft.com/en-us/javascript/api/office-scripts/overview?view=office-scripts)

## スクリプトが保存される場所

[Office Scripts file storage and ownership - Office Scripts | Microsoft Docs](https://docs.microsoft.com/en-us/office/dev/scripts/overview/script-storage)

VBA と違い、Office Script はブックとは別の場所に保存される。

スクリプトは、スクリプトを作成したユーザーの OneDrive に保存される。そのため、共有設定をしない限り、他の人はそのスクリプトを編集したり実行することはできない。

![](2021-09-09-16-52-08.png)

共有設定をしても保存場所は変わらないため、スクリプト作成者のアカウントを削除するとスクリプトも一緒に削除されると考えられる。
複数人でスクリプトを共有する場合は、バックアップを共有フォルダ (SharePoint とか) に取っておくと良いかもしれない。

![](2021-09-09-16-32-09.png)

## エディタ
[Office Scripts Code Editor environment - Office Scripts | Microsoft Docs](https://docs.microsoft.com/en-us/office/dev/scripts/overview/code-editor-environment)

下記がスクリプトの編集画面。

![](2021-09-07-16-44-05.png)

VS Code がベースになっているので、VS Code みたいに IntelliSense が使えたりする。

### 文字を大きくする
エディタの文字が小さくて見づらい場合は、大きくすることができる。

1. F1 キーを押してコマンドパレットを表示する。
1. Font と入力し、「Editor Font Zoom In」を選ぶ。
1. 文字が少し大きくなる。もっと大きくしたい場合は、1～3の手順を繰り返す。

![](2021-09-07-16-55-07.png)

## Power Automate から実行する

1. アクションの一覧から「Excel Online (Business)」を選ぶ。<br>![](2021-09-08-15-21-50.png)
1. 「スクリプトの実行」を選ぶ。<br>![](2021-09-08-15-22-54.png)
1. スクリプトを書いた Excel ファイルを指定すると、「スクリプト」のリストがファイルに定義したスクリプトの一覧になる。
1. さらにスクリプトを選択すると、そのスクリプトの main 関数に追加した引数が入力項目として現れる。<br>![](2021-09-08-15-27-44.png)

## サンプル

### 日付のシリアル値を変換する
日付が格納されたセルの値を取ると、シリアル値になっている。
これを JavaScript の日付型に変換するサンプル。

```typescript
function convertDate(excelDateValue: number) {
    let javaScriptDate = new Date(Math.round((excelDateValue - 25569) * 86400 * 1000));
    return javaScriptDate;
}
```

### テーブルにフィルタを設定し結果を取得する

```typescript
function main(workbook: ExcelScript.Workbook)
{
  let sheet = workbook.getWorksheet("Sheet1");
  let table = sheet.getTable("テーブル1");
  
  // フィルタ
  table.clearFilters();
  let column = table.getColumnByName("列1");
  column.getFilter().applyValuesFilter(["検索したい値"]);

  // 結果を取得
  let range = table.getRange().getVisibleView();
  if (range.getRowCount() == 0) {
    return;
  }

  // values は Object[][] 型。1行目にヘッダー、2行目以降にデータ行。
  console.log(range.getValues());
}
```

### テーブルをJSONへ変換する

```javascript
function main(workbook: ExcelScript.Workbook): TableData[] {
  const table = workbook.getWorksheet("Sheet1").getTable("テーブル1");
  // テーブルの値を表示されているテキストの形式で取得
  const texts = table.getRange().getTexts();
  
  let result = convertToJson(texts);

  console.log(result);
}

// テーブルの値(二次元配列)をJSONにする。
function convertToJson(values: Object[][]) {
  let result = {};

  if (values == null || values.length < 2) {
    return null;
  }

  // 1行目はヘッダー行なので2行目から始める
  for (let i:number = 1; i<values.length; i++) {
    for (let j:number = 0; j<values[i].length; j++) {
      let key = values[0][j] as string;
      let value = values[i][j];

      result[key] = value;
    }
  }

  return result;
}
```


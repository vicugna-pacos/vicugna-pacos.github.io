---
title: "繰り返しの予定"
date: 2020-10-28T13:39:53+09:00
---

## はじめに
Outlookの予定、タスクに設定できる「繰り返し」について。

## 概要
参照：[RecurrencePattern object (Outlook) | Microsoft Docs](https://docs.microsoft.com/en-us/office/vba/api/outlook.recurrencepattern)

`AppointmentItem` または `TaskItem` で設定できる。
上記2つには `IsRecurring`(Boolean) プロパティがあり、まずこれで繰り返しのアイテムかどうか確認できる。
繰り返しの詳細は、`GetRecurrencePattern` メソッドで取得できる。
`GetRecurrencePattern` または `ClearRecurrencePattern` メソッドを呼び出すと、`IsRecurring` プロパティが自動的に変更される。
つまり、何も気にせず `GetRecurrencePattern` を呼び出すと、`IsRecurring` が勝手に true になるので注意。

## 繰り返しの予定の参照

例えば 2020/10/1 12:00～13:00 に作った予定「test昼休み」を繰り返しの予定にしたとする。

* 開始日 - 2020/10/1
* 終了日 - 2020/10/31
* 繰り返しのパターン - 毎日
* 間隔 - 1日ごと

これに対して、10/1～10/31の条件で予定を検索しても、`AppointmentItem` は1つしか取れない。

```powershell
$filter = "[Start] >= '2020/10/01 00:00'"
$filter = $filter + " AND [End] <= '2020/10/31 00:00'"
$filter = $filter + " AND [Subject] = 'test昼休み'"

$items = $folder.Items.Restrict($filter)

foreach ($item in $items) {
    Write-Host ("■" + $item.Subject) # $itemsには1件しかない
    Write-Host ("Start:" + $item.Start.toString("yyyy/MM/dd") + "  End:" + $item.End.toString("yyyy/MM/dd"))
}
```

    ■test昼休み
    Start:2020/10/01  End:2020/10/01

次に、10/10を検索条件にして検索してみる。

```powershell
$filter = "[Start] >= '2020/10/10 00:00'"
$filter = $filter + " AND [End] <= '2020/10/11 00:00'"
$filter = $filter + " AND [Subject] = 'test昼休み'"

$items = $folder.Items.Restrict($filter)

foreach ($item in $items) {
    Write-Host ("■" + $item.Subject)
    Write-Host ("Start:" + $item.Start.toString("yyyy/MM/dd") + "  End:" + $item.End.toString("yyyy/MM/dd"))
}
```

    ■test昼休み
    Start:2020/10/01  End:2020/10/01

「test昼休み」は毎日繰り返すので、10/10の予定として検索できる。しかしStartとEndは10/01になっている。


### GetOccurrence メソッド
`AppointmentItem.RecurrencePattern().GetOccurrence(DateTime)` メソッドを使うと、
繰り返しの予定のうち、指定した日付の `AppointmentItem` を取得できる。
指定した日に予定がない場合はエラーになる。

    この定期的なアイテムの 1 回分に変更を加えたので、この回はもう存在しません。開いているアイテムをすべて閉じて、再度実行してください。
    発生場所 C:\xxxtest.ps1:39 文字:17
    + ...             $newItem = $item.GetRecurrencePattern().GetOccurrence($dt ...
    +                 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        + CategoryInfo          : OperationStopped: (:) [], COMException
        + FullyQualifiedErrorId : System.Runtime.InteropServices.COMException

引数に指定する日付は、__予定の開始時刻と同時刻でなければいけない__ 。「test昼休み」の場合、単純に「2020/10/08」と日付だけ指定している下記のサンプルではエラーになる。

```powershell
# $item は「test昼休み」のAppointmentItem
[DateTime]$dt = "2020/10/08"
$item2 = $item.GetRecurrencePattern().GetOccurrence($dt)
```

正しくは、「test昼休み」が始まる12:00も指定しなくてはいけない。

```powershell
# $item は「test昼休み」のAppointmentItem
[DateTime]$dt = "2020/10/08 12:00"
$item2 = $item.GetRecurrencePattern().GetOccurrence($dt)
```

参考：[vb.net - GetOccurrence always throws exception - Stack Overflow](https://stackoverflow.com/questions/12167921/getoccurrence-always-throws-exception)

### 繰り返しの予定をすべて取得する
「test昼休み」が繰り返される日付をすべて取得したいとき、以下の実装方法が考えられる：

* 全ての日付に Filter または Restrict を実行して総当たり。
* 全ての日付に GetOccurrence を実行して総当たり。

その他、繰り返しパターンなどから自力で計算する方法もあるが、例外パターンもあるため大変になるので推奨しない。
GetOccurrence メソッドを使う方は予定がない場合にエラーが発生するため、個人的には好きではない。

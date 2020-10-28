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

これに対して、10/01～10/31すべての予定を取りたい場合は、以下のサンプルのように検索する。

```powershell
$OlDefaultFolders = [Microsoft.Office.Interop.Outlook.OlDefaultFolders]

$namespace = $outlook.GetNamespace("MAPI")
$folder = $namespace.GetDefaultFolder($OlDefaultFolders::olFolderCalendar)

$allItems = $folder.Items
$allItems.Sort("[Start]")
$allItems.IncludeRecurrences = $true

$filter = "[Start] >= '2020/10/01 00:00'"
$filter = $filter + " AND [Start] <= '2020/11/01 00:00'"
$filter = $filter + " AND [Subject] = 'test昼休み'"

$items = $allItems.Restrict($filter)
```

Filter や Restrict を使う前に、Items を Start の昇順で並べ替えてから IncludeRecurrences プロパティを true にする。
もし Items に終了日のない繰り返しの予定があるときに IncludeRecurrences をtrueにすると、Items.Count の値がundefinedになるので注意。

参考：[Items.IncludeRecurrences property (Outlook) | Microsoft Docs](https://docs.microsoft.com/en-us/office/vba/api/outlook.items.includerecurrences)

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

検索したい全ての日付について GetOccurrence メソッドで総当たりすれば、繰り返しの予定が全て取得できる。

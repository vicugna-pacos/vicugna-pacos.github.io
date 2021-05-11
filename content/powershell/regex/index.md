---
title: "正規表現"
date: 2021-05-11T09:22:30+09:00
draft: true
---

## はじめに

参考：[about_Regular_Expressions - PowerShell | Microsoft Docs](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_regular_expressions?view=powershell-5.1)

PowerShell での正規表現について記載する。ただし、この記事では PowerShell で使用できる正規表現の文法にはふれていない。文法についての詳しい情報は、[.NET の正規表現のリファレンス](https://docs.microsoft.com/en-us/dotnet/standard/base-types/regular-expression-language-quick-reference) を参照。

PowerShell の正規表現は、既定では大文字小文字を区別しない。そのため、大文字小文字を区別させたい場合は、各メソッドでオプションなどを指定する必要がある。

|Method|Case Sensitivity|
|---|---|
|Select-String|-CaseSensitive スイッチを使う|
|switch 式|-casesensitive オプションを使う|
|演算子|頭に 'c' が付いている演算子を使う (-cmatch, -csplit, -creplace)|

## -match 演算子

    <string[]> -match    <regular-expression>
    <string[]> -notmatch <regular-expression>

左側に指定した文字列が、右側で指定した正規表現に一致している/一致していないかをチェックする。
左側に単一の文字列を指定するか、複数の文字列を指定するかで戻り値が異なる。

```powershell
"PowerShell" -match '^Power\w+'
 # Output: True

"PowerShell", "Super PowerShell", "Power's hell" -match '^Power\w+'
# Output: PowerShell
```

## パターンに一致した部分を取り出す
-match 演算子を使った直後に、`$Matches` という変数を参照すると、正規表現に一致した文字列を取り出すことができる。

下記のコードを例とする：

```powershell
'The last logged on user was CONTOSO\jsmith' -match '(.+was )(.+)'
```

`$Matches` では下記のような結果になる：

```powershell
$Matches.0    # The last logged on user was CONTOSO\jsmith
$Matches.1    # The last logged on user was 
$Matches.2    # CONTOSO\jsmith
```

`$Matches.0` はグルーピング等関係なく、正規表現に一致した部分全てが取得できる。
そして、`$Matches.1` 以降はグルーピングした部分を先頭から取得できる。

その他、グループに名前を付けることができる。  
コード例：

```powershell
'The last logged on user was CONTOSO\jsmith' -match 'was (?<domain>.+)\\(?<user>.+)'
```

結果：

```powershell
$Matches.domain    # CONTOSO
$Matches.user      # jsmith
$Matches.0         # was CONTOSO\jsmith
```


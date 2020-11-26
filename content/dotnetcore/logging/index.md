---
title: "コンソールアプリでログ出力"
date: 2020-11-26T09:34:54+09:00
---

## 概要

前提条件：

* .NET Core 3.1
* Visual Studio 2019 で開発

ASP.NET なら初めから使える(っぽい)ログ出力の機能を、コンソールアプリで使用する方法。

`Microsoft.Extensions.Logging` パッケージを使用するが、一番お手軽(と思われる)なファイルへの出力機能がない。
コンソール出力やイベントログへの出力はできる。

参考：[.NET Core および ASP.NET Core でのログ記録 | Microsoft Docs](https://docs.microsoft.com/ja-jp/aspnet/core/fundamentals/logging/?view=aspnetcore-5.0#built-in-logging-providers)

## NuGet パッケージの追加

下記パッケージをプロジェクトに追加する。

`Microsoft.Extensions.Logging`

これと併せて、使用したいプロバイダー(出力先)ごとにパッケージを追加する。

* `Microsoft.Extensions.Logging.Console` - コンソール
* `Microsoft.Extensions.Logging.EventLog` - イベントログ
* `Microsoft.Extensions.Logging.ApplicationInsights` - Azure の AppInsights
* など

## コンソールへログ出力
下記2つのパッケージをプロジェクトへ追加する。

```
Microsoft.Extensions.Logging
Microsoft.Extensions.Logging.Console
```

下記がロガーの準備とログ出力のサンプル。

```csharp
using Microsoft.Extensions.Logging;

namespace ConsoleApp1
{
    class Program
    {
        public static void Main(string[] args = null)
        {
            using var loggerFactory = LoggerFactory.Create(builder =>
            {
                builder
                    .AddFilter("Microsoft", LogLevel.Warning)
                    .AddFilter("System", LogLevel.Warning)
                    .AddFilter("ConsoleApp1.Program", LogLevel.Debug)
                    .AddConsole();
            });
            ILogger logger = loggerFactory.CreateLogger<Program>();
            logger.LogInformation("Example log message");
        }
    }
}
```

このサンプルを実行すると、下記のログがコンソール上に出力される。

```
info: ConsoleApp1.Program[0]
      Example log message
```

## イベントログへ出力
下記2つのパッケージをプロジェクトへ追加する。

```
Microsoft.Extensions.Logging
Microsoft.Extensions.Logging.EventLog
```

下記がロガーの準備とログ出力のサンプル。

```csharp
using Microsoft.Extensions.Logging;

namespace ConsoleApp1
{
    class Program
    {
        public static void Main(string[] args = null)
        {
            using var loggerFactory = LoggerFactory.Create(builder =>
            {
                builder
                    .AddFilter("Microsoft", LogLevel.Warning)
                    .AddFilter("System", LogLevel.Warning)
                    .AddFilter("ConsoleApp1.Program", LogLevel.Debug)
                    .AddEventLog();
            });
            ILogger logger = loggerFactory.CreateLogger<Program>();
            logger.LogInformation("Example log message");
        }
    }
}
```

イベントビューアの「Windows ログ」→「Application」の中にログが記録される。

![](2020-11-26-11-24-22.png)

`AddEventLog` メソッドの引数でログのソース名などを指定できるが、ソースは事前に登録が必要。
登録するには、PowerShell のコマンドを実行するのが一番手っ取り早い方法と思われる。

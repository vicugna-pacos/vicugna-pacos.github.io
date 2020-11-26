---
title: "汎用ホストでバッチ処理"
date: 2020-11-26T15:01:47+09:00
---

## 概要

前提条件：

* .NET Core 3.1
* Visual Studio 2019 で開発

ASP.NETで提供されているいくつかの機能を、コンソールアプリでも使えるよう汎用的にしたもの。
使えるようになる機能の例は下記の通り。

* 依存性の注入 (DI)
* ログ
* 設定ファイル

参考：[.NET Generic Host | Microsoft Docs](https://docs.microsoft.com/ja-jp/dotnet/core/extensions/generic-host)

## 実装手順
コンソールアプリケーションをテンプレートとしたプロジェクトを作成し、下記パッケージをプロジェクトに追加。

```
Microsoft.Extensions.Hosting
```

下記がMainメソッドサンプル。

```cs
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace ConsoleApp1
{
    class Program
    {
        public static void Main(string[] args = null)
        {
            var builder = Host.CreateDefaultBuilder(args)
                .ConfigureServices((hostContext, services) =>
                {
                    // AddHostedServiceメソッドは、using Microsoft.Extensions.DependencyInjection
                    // がないと呼び出せない
                    services.AddHostedService<TestService>();
                });

            builder.Build().Run();
        }
    }
}
```

処理のエントリーポイントとなるのは、`IHostedService` を実装したクラス。

```cs
using Microsoft.Extensions.Hosting;
using System;
using System.Threading;
using System.Threading.Tasks;

namespace ConsoleApp1
{
    class TestService : IHostedService
    {
        public Task StartAsync(CancellationToken cancellationToken)
        {
            Console.WriteLine("StartAsync");
            return Task.CompletedTask;
        }

        public Task StopAsync(CancellationToken cancellationToken)
        {
            Console.WriteLine("StopAsync");
            return Task.CompletedTask;
        }
    }
}
```

`IHostLifetime` の実装クラスが、いつホストが起動していつ終了するかを制御している。
既定では `Microsoft.Extensions.Hosting.Internal.ConsoleLifetime` が使われていて、
コンソールで Ctrl + C を押すまでホストが起動し続ける。

[ConsoleLifetime のソース](https://github.com/dotnet/runtime/blob/master/src/libraries/Microsoft.Extensions.Hosting/src/Internal/ConsoleLifetime.cs)

処理が終了次第アプリケーションを終了させたい場合は、`IHostedService` の `StartAsync` メソッドの最後で `IHostApplicationLifetime` の `StopApplication`を実行する。
`IHostApplicationLifetime` のインスタンスは、DIで受け取っておく。

下記はそのサンプル。

```cs {hl_lines=["21"]}
using Microsoft.Extensions.Hosting;
using System;
using System.Threading;
using System.Threading.Tasks;

namespace ConsoleApp1
{
    class TestService : IHostedService
    {

        private readonly IHostApplicationLifetime _appLifeTime;

        public TestService(IHostApplicationLifetime appLifeTime)
        {
            _appLifeTime = appLifeTime;
        }

        public Task StartAsync(CancellationToken cancellationToken)
        {
            Console.WriteLine("StartAsync");
            _appLifeTime.StopApplication();

            return Task.CompletedTask;
        }

        public Task StopAsync(CancellationToken cancellationToken)
        {
            Console.WriteLine("StopAsync");
            return Task.CompletedTask;
        }
    }
}
```

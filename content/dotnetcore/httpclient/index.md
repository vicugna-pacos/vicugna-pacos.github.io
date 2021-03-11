---
title: "HttpClient"
date: 2021-03-08T15:49:54+09:00
---

## はじめに
.NET Core (C#) で API を実行したい場合などは、`System.Net.Http.HttpClient` を使う。

HttpClient のインスタンスはアプリケーションで1つになるように定義する。
IDisposable インターフェイスを実装しているが、自分で using 句で囲んだり Dispose メソッドを実行する必要はない。

## GET

```cs
using System;
using System.Collections.Generic;
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;

namespace Samples
{
    public class Class1
    {
        private static readonly HttpClient client = new HttpClient();

        public static async Task Sample1(string url)
        {
            var response = await client.GetAsync(url);
            response.EnsureSuccessStatusCode();

            var responseContent = response.Content.ReadAsStringAsync();
        }
    }
}
```

### ステータスの確認方法
HttpClient の各メソッドを実行したとき、HTTPステータスコードがエラーになっても勝手に Exception がスローされない。
下記のいずれかの方法でステータスコードをチェックする必要がある。

```cs
var response = await client.GetAsync(url);
// ステータスコードがエラーの場合、Exception がスローされる
response.EnsureSuccessStatusCode();
```

```cs
if (!response.IsSuccessStatusCode)
{
    // エラー時の処理
}
```

## POST

フォームデータを送信する (application/x-www-form-urlencoded) 場合は、`FormUrlEncodedContent` クラスを使う。以下サンプル。

```cs
using System;
using System.Collections.Generic;
using System.Net.Http;
using System.Threading.Tasks;

namespace Samples
{
    public class Class1
    {
        private static readonly HttpClient client = new HttpClient();

        public async Task Sample1(string url)
        {
            var values = new Dictionary<string, string>
            {
                { "param1", "value1" },
                { "param2", "value2" }
            };

            HttpContent content = new FormUrlEncodedContent(values);
            var response = await client.PostAsync(url, content);

            if (!response.IsSuccessStatusCode)
            {
                // 失敗
            }
            // 成功
        }
    }
}
```

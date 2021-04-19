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

### Multipart データを送る
ファイルアップロードがある場合は、こちらの方法を使う。

```cs
public async Task ExampleAsync(string url, List<string> filePaths)
{
    MultipartFormDataContent rootContent = new MultipartFormDataContent();

    // ファイル以外のデータをセット
    StringContent content1 = new StringContent("value1");
    content1.Headers.ContentDisposition = new ContentDispositionHeaderValue("form-data");
    content1.Headers.ContentDisposition.Name = "name1";
    rootContent.Add(content1);

    StringContent content2 = new StringContent("value2");
    content2.Headers.ContentDisposition = new ContentDispositionHeaderValue("form-data");
    content2.Headers.ContentDisposition.Name = "name2";
    rootContent.Add(content2);

    // アップロードファイルを送信データに追加する
    var streams = new List<IDisposable>();
    try
    {
        foreach (var filePath in filePaths)
        {
            FileInfo info = new FileInfo(filePath);

            // ファイルを開く
            var stream = new FileStream(filePath, FileMode.Open, FileAccess.Read);
            streams.Add(stream);

            // 送信データに追加
            var streamContent = new StreamContent(stream);
            streamContent.Headers.ContentDisposition = new ContentDispositionHeaderValue("form-data")
            {
                Name = "file",
                FileName = info.Name
            };

            rootContent.Add(streamContent);
        }

        // 送信
        using var request = new HttpRequestMessage(HttpMethod.Post, url);
        request.Content = rootContent;

        var response = await client.SendAsync(request);
    }
    finally
    {
        streams.ForEach(x => x.Dispose());
    }
}
```

#### 日本語ファイル名が文字化けする場合
大体の場合は上記サンプルでよいが、場合によってはアップロードしたファイル名が以下のようになったりする。

    =?utf-8?B?xxxxx?=

これは HttpClient が仕様に従って filename を filename* にしてエンコードしてくれているが、リクエストを受け取るサーバー側がこの仕様に対応していなかったりするのが原因。
(参考：[Content-Disposition - HTTP | MDN](https://developer.mozilla.org/ja/docs/Web/HTTP/Headers/Content-Disposition))

対策としては、一度ファイル名をバイトコードへ変換し、それをさらに文字型にする方法がある。以下がサンプルコード。

```cs
// ヘッダー値を作る
streamContent.Headers.Add("Content-Type", "application/octet-stream");
streamContent.Headers.Add("Content-Disposition", GetContentDisposition(info.Name));

private static string GetContentDisposition(string fileName)
{
    var headerValue = "form-data; name=\"file\"; filename=\"" + fileName + "\"";
    var bytes = Encoding.UTF8.GetBytes(headerValue);
    var sb = new StringBuilder();
    foreach (var b in bytes)
    {
        sb.Append((char)b);
    }
    return sb.ToString();
}
```

参考：[c# - How to disable base64-encoded filenames in HttpClient/MultipartFormDataContent - Stack Overflow](https://stackoverflow.com/questions/21928982/how-to-disable-base64-encoded-filenames-in-httpclient-multipartformdatacontent)

## ヘッダーを指定する
リクエストのヘッダーを指定したい場合、 GetAsync や PostAsync メソッドは使用できない。代わりに SendAsync メソッドを使用する。サンプルは下記の通り。

```cs
using (HttpRequestMessage request = new HttpRequestMessage(HttpMethod.Post, "http://xxx"))
{
    // body
    var content = new StringContent("aaa");
    request.Content = content;

    // header
    request.Headers.Add("name", "value");

    // 送信
    using (var response = await client.SendAsync(request))
    {
        // TODO : レスポンスを受け取る
    }
}
```

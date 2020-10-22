---
title: "Google Fit"
date: 2020-09-24T20:40:18+09:00
draft: true
---

## Google Fit のコンポーネント

Google Fit は、以下のコンポーネントで構成される：

* The fitness store
  * 様々なアプリやデバイスからのデータを保存する場所。クラウド上にある。
* The sensor framework
  * スマホとかスマートウォッチのセンサーとやり取りするフレームワーク。Google Fit API 経由で使用する。
* Permissions and user controls
  * Fitness storeのデータを読み書きするためのパーミッションなど。
* Google Fit APIs
  * Android用APIと、REST API の2種類がある。Android以外はREST API を使う。

今回はREST APIを使ってデータ連携したいので、REST APIについて調べる。

## REST API
参考：[REST API  |  Google Fit  |  Google Developers](https://developers.google.com/fit/rest?hl=ja)

REST APIでは、以下のことができる：

* data source の作成、取得、検索、更新。data source とは、データの提供元ごとに一意に作られる。fitness store に保存されるデータは、すべて data source の中に保存される。
* dataset の作成、取得、集計、削除。dataset とは、data point のセット。
* data point を列挙して、datasetへ追加できる。data pointとは、data source から取り出したある一点のデータ。
* session の作成、取得、削除。session とは、時間のインターバルと共に構成されたデータのこと。

### Get Started
参考：[Getting Started with the REST API  |  Google Fit  |  Google Developers](https://developers.google.com/fit/rest/v1/get-started?hl=ja)

このチュートリアルでは、Fitness REST API のアクティベート方法、OAuth アクセストークンの取得方法、HTTPリクエストを使ったAPIメソッドの実行方法、について解説する。

Fitness REST API を使うには、RESTfulなWebサービスの基本と、JSON構文について理解している必要がある。

#### Google アカウントの取得
Fitness REST API を使用するなら、Google アカウントが必要である。APIテスト用に、別のアカウントも用意した方がいいかもしれない。

#### OAuth 2.0 client ID の取得
下記の手順を参考に、Fitness APIのクライアントを取得する。

1. [Google API Console](https://console.developers.google.com/flows/enableapi?apiid=fitness) を開く。
1. プロジェクトを選択するか、新しいプロジェクトを作成する。AndroidアプリとRESTを使ったアプリの両方を作るなら、両者は同じプロジェクトに作成すること。
1. __Continue__ をクリックし、Fitness API を有効化する。
1. __Go to credentials__ をクリックする。
1. __New credentials__ をクリックし、__OAuth Client ID__ を選択する。
1. __Application type__ は、__Web application__ を選択する。
1. __Authorized JavaScript origins__ は、APIを実行するWebサイトのベースURLを入力する。(例えば、OAuth Playground では、`https://developers.google.com` がURLになる)
1. __Authorized redirect URI__ は、認証後のレスポンスを受け付けるURLを入力する。(OAuth Playground では、`https://developers.google.com/oauthplayground` が使われている)

Click Create. Your new OAuth 2.0 Client ID and secret appear in the list of IDs for your project. An OAuth 2.0 Client ID is a string of characters, something like this:

780816631155-gbvyo1o7r2pn95qc4ei9d61io4uh48hl.apps.googleusercontent.com

Try the REST API in the OAuth Playground
The OAuth Playground is the easiest way to familiarize yourself with the Fitness REST API by submitting HTTP requests and observing the responses before you write any client code.

To authorize the Fitness REST API in the OAuth Playground:

Go to the OAuth Playground.
Under Step 1 Select & authorize APIs, expand Fitness v1 and select the Fitness scopes to use.
Click the Authorize APIs button, select the Google API Console project to use, and click Allow when prompted. You will be able to access and modify data associated with the selected Google API Console account.
Click the Exchange authorization code for tokens button. The OAuth Playground automatically includes this header in the Authorization: request header when you submit HTTP requests. Note that the access token will expire after 60 minutes (3600 seconds).
Submit HTTP requests
The following examples demonstrate how to send HTTP requests to list all available data sources, and to create a new data source. For the Fitness REST API, the URI format is:

https://www.googleapis.com/fitness/v1/resourcePath?parameters

To list all available data sources:

In HTTP Method, select GET.
In Request URI, enter https://www.googleapis.com/fitness/v1/users/me/dataSources
Click Send the request.
The request and the response appear on the right side of the page. If the request is successful, the response shows the data source from the previous example in JSON format.

To create a data source:

In HTTP Method, select POST.
In Request URI, enter https://www.googleapis.com/fitness/v1/users/me/dataSources
Click Enter request body.
In the Request Body window, copy and paste the following JSON:

{
  "dataStreamName": "MyDataSource",
  "type": "derived",
  "application": {
    "detailsUrl": "http://example.com",
    "name": "Foo Example App",
    "version": "1"
  },
  "dataType": {
    "field": [
      {
        "name": "steps",
        "format": "integer"
      }
    ],
    "name": "com.google.step_count.delta"
  },
  "device": {
    "manufacturer": "Example Manufacturer",
    "model": "ExampleTablet",
    "type": "tablet",
    "uid": "1000001",
    "version": "1"
  }
}


In the Request Body window, click Close.

Click Send the request.

The request and the response appear on the right side of the page. The request includes the OAuth access token in the Authorization header:

Authorization: Bearer ya29.OAuthTokenValue

If the request is successful, the first line of the response is:

HTTP/1.1 200 OK

Use cURL to access the Fit REST API
You can use the cURL command line tool to access the Fit REST API. You will need an OAuth access token to make requests using cURL (see the preceding instructions). Note that access tokens expire after an hour. The following example shows a simple bash script to list all data sources.

#!/bin/bash
ACCESS_TOKEN=""
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  https://www.googleapis.com/fitness/v1/users/me/dataSources

Next steps
To learn more about the REST API, see these pages:

Fitness Data Types
Sessions
How to Record a Workout

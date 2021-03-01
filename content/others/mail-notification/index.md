---
title: "メール受信をトリガーにする方法"
date: 2021-03-01T13:37:36+09:00
---

## はじめに
業務の効率化をしたくて、メール受信をトリガーにスクリプトを実行させる方法がないかと調べた記録。

前提条件：

* 職場では Microsoft 365 を使っている
* 組織アカウントを使っている
* 私は組織の管理者ではない

## Graph API
Graph API とは、Microsoft 365 のリソースにアクセスできる API である。
Graph API であれば、Outlookのメール受信はもちろん、予定登録など他のリソースに追加・変更があった際に Webhook を実行させることができる。
利用には Azure AD が使えること、APIを使うアプリケーションが組織の管理者によって許可される必要がある。
個人でお手軽に実装するには向いていない。

参考：

* [Microsoft Graph API を使用して変更通知を取得する - Microsoft Graph v1.0 | Microsoft Docs](https://docs.microsoft.com/ja-jp/graph/api/resources/webhooks?view=graph-rest-1.0)

## Power Automate
Power Automate とは、Microsoft 365 に含まれている製品で、様々なトリガーとアクションを用いて自動で動くフローを作成できるツールである。
トリガーに「新しいメールが届いたとき」というものがあるので、メール受信をトリガーとしたフローも作成できる。
しかし、Webhook の実行や Azure など他サービスへ向けたアクションを実行するには、PREMIUM プランが必要。

## Outlook VBA
Excel などと同様に、Outlook にも VBA がある。
そして、メール受信イベントも存在する。
Outlook アプリケーションがあれば使用できるが、PC が起動かつネットワークに接続していて、さらに Outlook が起動している必要がある。

参考：

* [NewMail イベント (Outlook) | Microsoft Docs](https://docs.microsoft.com/ja-jp/office/vba/api/outlook.application.newmail)
* [NewMailEx イベント (Outlook) | Microsoft Docs](https://docs.microsoft.com/ja-jp/office/vba/api/outlook.application.newmailex)

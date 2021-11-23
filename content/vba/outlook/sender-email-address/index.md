---
title: "差出人のメールアドレスを取得する"
date: 2021-11-10T09:34:55+09:00
---

## 概要
受信したメール (MailItem) の差出人のメールアドレスを取得するには、SenderEmailAddress プロパティを参照する。
ただし、Exchange を使用してる かつ 差出人が同じ組織内のユーザーである場合は、SenderEmailAddress からメールアドレスを取得できない。
そのような場合は、PropertyAccessor オブジェクトを使用する必要がある。

参考：[Get the SMTP address of the sender of a mail item | Microsoft Docs](https://docs.microsoft.com/en-us/office/client-developer/outlook/pia/how-to-get-the-smtp-address-of-the-sender-of-a-mail-item)

差出人の SenderEmailType が "EX" の場合、その人は同じ組織内の Exchange 利用者ということになる。
下記サンプルでは、その場合に PropertyAccessor オブジェクトを使用してメールアドレスを取得する。

## サンプル

```vb
' ----------------------------------------
' 送信者のメールアドレスを取得する
' ----------------------------------------
Private Function GetSenderEmailAddress(ByRef oItem As MailItem) As String

    Dim PR_SMTP_ADDRESS  As String
    Dim oSender As AddressEntry
    Dim oExUser As ExchangeUser
    
    
    PR_SMTP_ADDRESS = "http://schemas.microsoft.com/mapi/proptag/0x39FE001E"
    
    ' Exchange 以外
    If oItem.SenderEmailType <> "EX" Then
        GetSenderEmailAddress = oItem.SenderEmailAddress
        Exit Function
    End If
    
    Set oSender = oItem.Sender
    
    If oSender.AddressEntryUserType = olExchangeUserAddressEntry _
        Or oSender.AddressEntryUserType = olExchangeRemoteUserAddressEntry Then
        
        Set oExUser = oSender.GetExchangeUser
        GetSenderEmailAddress = oExUser.PrimarySmtpAddress
        Exit Function
    End If
    
    GetSenderEmailAddress = oSender.PropertyAccessor.GetProperty(PR_SMTP_ADDRESS)

End Function
```

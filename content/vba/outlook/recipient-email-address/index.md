---
title: "TO、CCなどのメールアドレスを取得する"
date: 2022-01-26T10:13:55+09:00
weight: 3
---

## 概要
メールの TO、CC、BCCを取得するときは [MailItem.Recipients](https://docs.microsoft.com/en-us/office/vba/api/outlook.mailitem.recipients) を参照する。
MailItem には [To プロパティ](https://docs.microsoft.com/en-us/office/vba/api/outlook.mailitem.to) もあるが、こちらは宛先の表示名しか取得できない。
加えて、[差出人の場合]({{< ref "/vba/outlook/sender-email-address/index.md" >}})と同じように、同じ組織内の Exchange ユーザーの場合はアドレスの形式が異なる。

## サンプル

下記のサンプルでは、TO のメールアドレスを取得する。
Exchange ユーザーの場合でもメールアドレスが取得できるようにしている。

```vb
' 受信フォルダにあるメールの TO のメールアドレスを取得し、イミディエイトウィンドウに出力するサンプル
Public Sub Sample1()
    Dim oNs As NameSpace
    Dim oFolder As Folder
    Dim oItem As Variant
    Dim oMailItem As MailItem
    Dim oRecipient As Recipient
    Dim i As Integer
    
    
    Set oNs = Application.GetNamespace("MAPI")
    Set oFolder = oNs.GetDefaultFolder(olFolderInbox)
    
    For Each oItem In oFolder.Items
        If TypeName(oItem) = "MailItem" Then
            Set oMailItem = oItem
            Debug.Print oMailItem.Subject
            
            For i = 1 To oMailItem.Recipients.Count
                Set oRecipient = oMailItem.Recipients(i)
                If oRecipient.Type = OlMailRecipientType.olTo Then
                    Debug.Print GetRecipientEmailAddress(oRecipient.AddressEntry)
                End If
            Next
        End If
    Next

End Sub

Private Function GetRecipientEmailAddress(ByRef oAddressEntry As AddressEntry) As String

    Dim oSender As AddressEntry
    Dim oExUser As ExchangeUser
    
    
    If oAddressEntry.AddressEntryUserType = olExchangeUserAddressEntry _
        Or oAddressEntry.AddressEntryUserType = olExchangeRemoteUserAddressEntry Then
        
        Set oExUser = oAddressEntry.GetExchangeUser
        GetRecipientEmailAddress = oExUser.PrimarySmtpAddress
        Exit Function
    End If
    
    GetRecipientEmailAddress = oAddressEntry.Address

End Function
```

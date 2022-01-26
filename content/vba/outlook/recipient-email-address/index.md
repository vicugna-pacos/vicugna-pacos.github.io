---
title: "Recipient Email Address"
date: 2022-01-26T10:13:55+09:00
draft: true
---

TO、CC、BCC のメールアドレスを取得する。

```vb
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

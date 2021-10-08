---
title: "Memo"
date: 2021-08-26T16:51:00+09:00
draft: true
---

## キャンセル済みの通知を元に会議を削除する
自分以外が開催した会議がキャンセルされると、参加者には「キャンセル済み」の通知が来る。
その通知を開けば該当する会議予定を削除できるが、一度に一つの通知に対してしか操作できない。

https://outlooklab.wordpress.com/2015/08/01/%E5%87%BA%E5%B8%AD%E8%80%85%E3%81%8B%E3%82%89%E8%BE%9E%E9%80%80%E3%81%AE%E8%BF%94%E4%BF%A1%E3%81%8C%E6%9D%A5%E3%81%9F%E9%9A%9B%E3%81%AB%E3%82%AD%E3%83%A3%E3%83%B3%E3%82%BB%E3%83%AB%E9%80%9A%E7%9F%A5/
https://docs.microsoft.com/en-us/office/vba/outlook/concepts/forms/item-types-and-message-classes

## AppointmentItem
https://docs.microsoft.com/en-us/office/vba/api/outlook.appointmentitem

## MeetingItem
ある人が作った会議アイテムに参加者として追加された人に届くアイテムのこと。会議通知とか、その会議のキャンセル通知とかが該当する。

MailItem や Appointment と異なり、MeetingItem を直接作成することはできない。
AppointmentItem のプロパティ MeetingStatus を olMeeting に設定すると、参加者に指定されている人に送信される。
そして、参加者の人たちの受信ボックスに、MeetingItem が届く。

MeetingItem の GetAssociatedAppointment メソッドを使えば、MeetingItem と紐づいた AppointmentItem を取得できる。


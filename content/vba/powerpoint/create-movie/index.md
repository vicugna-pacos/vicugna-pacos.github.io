---
title: "Create Movie"
date: 2022-04-21T16:27:59+09:00
draft: true
---

## 自動読み上げソフトを使って動画を作る

参考：

* [PowerPointによるVOICEROID音声付きプレゼン動画の作成法 | 知識のサラダボウル](https://salad-bowl-of-knowledge.github.io/hp/presentation/2020/04/13/voiceroid_powerpoint.html)
* [Using VBA to insert and automatically play audio files in Powerpoint - Microsoft Community](https://answers.microsoft.com/en-us/msoffice/forum/all/using-vba-to-insert-and-automatically-play-audio/a56ac636-2a83-4c37-88d1-068ef01b52b9)

```vb
Sub SampleTest()
	Call InsertAudio("G:\Music\track1.mp3", ActivePresentation.Slides(1))
	Call InsertAudio("G:\Music\track2.mp3", ActivePresentation.Slides(2))

End Sub

Sub InsertAudio(Track As String, oSlide As Slide)
	Dim oShp As Shape
	Dim oEffect As Effect

	'Add the audio shape
	Set oShp = oSlide.Shapes.AddMediaObject2(Track, True, False, 10, 10)

	'Set audio to play automatically
	Set oEffect = oSlide.TimeLine.MainSequence.AddEffect(oShp, msoAnimEffectMediaPlay, , msoAnimTriggerWithPrevious)
	oEffect.MoveTo 1

	'Hide during slide show
	With oEffect
	    .EffectInformation.PlaySettings.HideWhileNotPlaying = True
	End With

End Sub
```


ファイルのプロパティ一覧。
https://docs.microsoft.com/en-us/windows/win32/properties/props

その他
http://www.thom.jp/vbainfo/refsetting.html
https://excel-ubara.com/excelvba4/EXCEL_VBA_426.html

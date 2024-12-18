---
tags:
  - windows
  - tts
---

Windows has two implementations of TTS.

## SAPI 5

SAPI 5 is like some kind of legacy built in system that is used since Windows XP. This uses the robotic voices for TTS. It seems that no more development is being done on SAPI 5 and no 'natural voices' are available.

## Windows Runtime (WinRT) / OneCore speech

WinRT is the newer alternative available since Windows 10. This is the backend used when using the 'Navigator' application. There are quite some 'natural voices' available for WinRT. Without some tinkering however, they can only be used via the Navigator app.

## Converting WinRT to SAPI 5

Since WinRT only seems to be usable via the Navigator application, there is no default way to use non-robotic voices via Text to Speech in Windows (without using the Navigator application).

If you are using external applications (such as the DialogueUI WoW Addon) for TTS, you can only select SAPI 5 voices and they all suck because they are extremely robotic.

Luckily there is open source! Some great dude created [NaturalVoiceSAPIAdapter](https://github.com/gexgd0419/NaturalVoiceSAPIAdapter). This uses shenanigans that I don't understand (who understands Windows?) to convert WinRT voices to SAPI 5 voices. Effectively meaning that you can use the natural sounding voices from Windows Navigator (e.g. Microsoft Guy, Microsoft Jenny) in external applications such as the DialogueUI WoW Addon for text to speech.

Gone are the days of robotic voices, long live the future!
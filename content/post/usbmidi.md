---
title: "usb midiを使う"
date: "2021-01-30"
description: ""
tags: ["Tips"]
draft: true
---

NXP社が提供するMCUXpressoSDK USB StackでUSB MIDIを実装する方法の紹介。
<!--more-->
# Introduction
音楽機器(特にエフェクタ、シンセ)では、他の機器との通信にMIDI通信[^1]が使われることが多くなりました。  
近年では、データ量の増加、通信速度の変化、汎用コネクタ化の要望などによりUSBを物理層としたMIDI通信も一般的になっています。
そんなこともあり趣味の工作でもUSB-MIDIを使いたいといった声をよく聞くようになりました。

そこでこの記事では、NXP社のi.MXRTシリーズでUSB-MIDIを実装してみたいと思います。

# Basics
いきなり実装方法を説明するより基礎的な話を多少やっておいた方が良いと感じたので読むべき資料、参考になるサイトなんかを紹介します。

まず組み込みでUSBを扱うにあたり必要一般知識を学ぶためにはインターフェイス株式会社が公開してくれている[技術解説](#sankou)の２つを読むといいと思います。

* Road to USB Master
* USB AUDIO関連情報

次に[usb.org](https://www.usb.org/)が発行する本家規格書を読むことになりますが全部読むというよりは辞書的な使い方をしたほうが良いと思います。[^2]

* Universal Serial Bus Device Class Definition for Audio Devices1.0
* USB MIDI Devices 1.0

最後に規格書を片手にシンセアンプラグドさんの[USB-MIDI(1)~(13)](#sankou)を読むとより実装に近い理解が得られると思います。

# MCUXpresso SDK USB Stack Device
i.MXRTシリーズでUSB扱うには、MCUXpressoSDK USB Stackを使用することになります。
USB Stackに関しても公式ドキュメントが存在しますのでダウンロードをしておくと良いです。
ダウンロードはSDK Builderのページで行うことができます。

* MCUXpresso SDK USB Stack User’s Guide
* MCUXpresso SDK USB Stack Device Reference Manual

MCUXpresso SDK USB Stack User’s Guideにサンプルの取り扱いだったりが載っているのでまずはAUDIO関連のサンプルを動かしてみるのも良いかもしれません。

# USB-MIDI implementation 

# Ref
<a id="sankou"></a>
1. [インターフェイス株式会社　技術解説](https://www.itf.co.jp/tech)  
2. [シンセアンプラグド:USB-MIDI(1)~(13)](https://pcm1723.hateblo.jp/entry/20141206/1417861340)
3. [USB MIDI Devices 1.0](https://www.usb.org/document-library/usb-midi-devices-10)
4. [USB Class Definition for MIDI Devices v2.0](https://www.usb.org/document-library/usb-class-definition-midi-devices-v20)
5. [Universal Serial Bus Device Class Definition for Audio Devices1.0](https://www.usb.org/document-library/audio-device-document-10)
6. [Audio Devices Rev. 2.0 and Adopters Agreement](https://www.usb.org/document-library/audio-devices-rev-20-and-adopters-agreement)
7.  MCUXpresso SDK USB Stack User’s Guide


[^1]:2020年にはMIDI自体の規格もMIDI2.0に推進され分解能の向上、速度規定撤廃などが行われています。

[^2]:USB AUDIO CLASSは最新では2.0 , 3.0が存在しUSBMIDIはMIDI2.0に対応したUSB-MIDI2.0が存在するがこの記事は両方とも1.0を対象に実装している。

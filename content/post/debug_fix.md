---
title: "debuggerが繋がらなくなったとき"
date: "2020-05-26"
description: ""
tags: ["Tips","RT1010"]
---

i.MXRTシリーズは外部ROMから起動する仕組みのためか変なコードを書き込んだりするとデバッガが繋がらなくなる事が多々あります。  
今回はそれの対処法の一つを紹介したいと思います。
<!--more-->
今回の内容は**MIMXRT1010-EVK**を使って説明します。  
異なるデバイスを使用する際は、データシートをよく読んでください。

作業スキームを箇条書きで示します。

- BOOTスイッチをQSPI Flashブート以外の設定にする。
- Linkserver GUI Flash Programmerでフラッシュを消去する。
- BOOTスイッチをQSPI Flashブートに戻す。
- プログラムをデバッグする。
# BOOTスイッチを変更する(0010 -> 0000)
MIMXRTシリーズでは、起動時にどのデバイスから起動するかをBOOTピンで設定する。  
評価ボードでは画像に示すDIPスイッチがそのピンに接続されており設定が可能です。  
BOOTピンには2種類の設定があります。

- 0001 : Serial ダウンロードモード
- 0010 : Boot from SPI Flash

今回は0000に設定する。 (QSPI Flash意外であれば問題ない)   
尚、スイッチのどちらがONであるかは基板のシルクを読めば書いてあるのでそちらを参考にしてください。

{{< figure src="../../img/debugger1.png"class="center" width="100%" height="100%" >}}



# 外部SPI Flashの値を消去する
次にフラッシュの消去を行っていきます。

フラッシュの消去には、Linkserver GUI Flash ProgrammerというGUIツールを利用する。  
Linkserver GUI Flash Programmerはフラッシュの書き換えや消去ができるNXPの純正ツールであり、MCUXPressoIDEに内蔵されています。

早速使い方を説明していきます。

評価ボードをPCと接続してください。（デバッガ側のUSB）  
次にMCUXpressoIDEの上部メニューにあるICの形をしたアイコンをクリックしLinkserver GUI Flash Programmerを開きます。  
うまく接続できていれば、DAPLINKが読み込まれ表示されます。

{{< figure src="../../img/debugger5.png"class="center" width="100%" height="100%" >}}

続いてOKをクリックしメインメニューを表示します。

{{< figure src="../../img/debugger4.png"class="center" width="100%" height="100%" >}}

最後にmass eraseを選択しRunをクリックする。  
消去がうまくいくとウィンドウが表示されると思います。

消去に成功したらBOOTスイッチをもとに戻します。(0000 -> 0010)   

# デバッガ接続を確認する。

フラッシュの消去が完了したので再度デバッガがつながるか確認します。  
このとき前回のコードが悪さをしてデバッガが繋がらなくなっている可能性があります。  
そのため同様のプロジェクトを再度デバッグすると再び繋がらなくなる可能性が高く、
同様の手順を踏んで再度フラッシュ消去を行う必要があるので開発効率が非常に悪いです。

以上よりデバッガ接続の確認は、Lチカのサンプルプログラムなど必ず動作するプロジェクトで行ってください。


# Ref  
フラッシュツールについて詳しく知りたい方はMCUXPressoIDEのリファレンスマニュアルの14章GUI Flash Toolを読んでください。

- [MCUXpresso IDE User Guide - NXP Semiconductors](http://www.nxp.com/docs/en/user-guide/MCUXpresso_IDE_User_Guide.pdf)
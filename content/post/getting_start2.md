---
author: "Takeshi Ogawa"
title: "Getting start(2) : サンプルプロジェクトの実行とデバッグ"
date: "2020-05-29"
description: ""
tags: ["RT1010"]
draft: false
---
この記事は、Getting started with i.MXRT microcontroller (1)の続きです。  
サンプルプロジェクトのインポートとその実行方法について説明します。 
<!--more-->

前回の環境構築が終わっていることを前提にすすめますので、済んでいない方は環境構築から行ってください。

扱う環境は以下の通りです。

- MCUXpressoIDE
- MCUXpresso SDK(MIMXRT1010-EVK)
- MIMXRT1010-EVK 評価ボード


# サンプルプログラムのインポート
i.MXRTシリーズのSDKには、豊富なサンプルプログラムが含まれており、これを利用することで自身のプロジェクトを円滑に進めたり、効率的に学習を行う事が可能です。

サンプルプログラムのインポートは、IDE左下のMCUXpressoIDE QuickStart Panelペイン内のImport SDK exampleをクリックすることで行うことができます。

{{< figure src="../../img/get_start5.png"class="center" width="100%" height="100%" >}}

Import SDK exampleを開くと自身がインストールしたSDKがサムネイル画像とともに表示されます。

今回は、MIMXRT1010-EVKを扱うので、それを選択肢しNEXTをクリックします。

{{< figure src="../../img/get_start6.png"class="center" width="100%" height="100%" >}}

サンプル一覧が表示されるので、  
<font color="CadetBlue">driver_examples/gpio/igpio_led_output</font>を選択しNEXTをクリックします。

{{< figure src="../../img/get_start7.png"class="center" width="100%" height="100%" >}}

次にAdvanced Settingsという画面に遷移しますが、<font color="MediumVioletRed">今回は入門なので</font>何もいじらずFinishをクリックしましょう。

{{< figure src="../../img/get_start8.png"class="center" width="100%" height="100%" >}}

# プロジェクトのビルド

サンプルプロジェクトがインポートされるとProject Explorerに<font color="CadetBlue">evkmimxrt1010_igpio_led_output</font>という名前で表示されます。

プロジェクトを展開すると多くのフォルダが格納されていますが、source/gpio_led_output.cがメインプログラムになります。

今回はサンプルプログラムの実行が目的なので、プログラムは編集しないでビルドします。

プロジェクトのビルドは、IDE上部のメニューバーの金槌のアイコンをクリックすることで行います。

{{< figure src="../../img/get_start9.png"class="center" width="100%" height="100%" >}}

ビルドが完了するとコンソールにメモリ使用量などが表示されます。

```
Memory region         Used Size  Region Size  %age Used
     BOARD_FLASH:       24800 B        16 MB      0.15%
        SRAM_DTC:        4356 B        32 KB     13.29%
        SRAM_ITC:          0 GB        32 KB      0.00%
         SRAM_OC:          0 GB        64 KB      0.00%
   NCACHE_REGION:          0 GB         0 GB       nan%
Finished building target: evkmimxrt1010_igpio_led_output.axf
```

# プログラムの書き込みとデバッグ
次にMIMXRT1010-EVKにプログラムを書き込みます。
プログラムの書き込みと同時にデバッガを起動してプログラムが実際に動いていることを確認します。

PCでの操作に移る前に、MIMXRT1010-EVKをPCと接続します。(windowsの場合初回ドライバのインストールなどが行われる)

{{< figure src="../../img/get_start10.png"class="center" width="100%" height="100%" >}}

次にIDE上部のメニューバー内の青色の虫のアイコンをクリックします。

ボードが正常に接続されていればウインドウが開いてDAPLink CMSIS-DAPという表示がされると思うのでOKをクリックしてすすめます。

{{< figure src="../../img/get_start11.png"class="center" width="100%" height="100%" >}}

しばらく待っていると、プログラムが書き込まれデバッガが起動します。  

int mainの先頭でプログラムが止まると思うので上部のRusumeをクリックしプログラムを進めます。

{{< figure src="../../img/get_start12.png"class="center" width="100%" height="100%" >}}

ここまででエラーが出ていなければ、ボード上のLEDが点滅すると思います。  
以上でサンプルプログラムのインポート、プログラムの書き込みとデバッグは終了となります。

# 参考文献
- MCUXpresso IDE User Guide.pdf
- Getting Started with MCUXpresso SDK for MIMXRT1011.pdf
- MCUXpresso SDK Release Notes Supporting MIMXRT1010-EVK.pdf

MCUXpresso IDE User Guide.pdfは公式のページ or インストール後のフォルダに入っています。
下2つは、SDKのdocsフォルダに内包されています。
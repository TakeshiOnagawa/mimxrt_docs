---
author: "Takeshi Ogawa"
title: "Getting start(1) : MIMXRTのための環境構築"
date: "2020-05-27"
description: ""
tags: ["RT1010"]
draft: false
---

この記事では、2020/05/27時点でのMIMXRTシリーズの環境構築を紹介する。  
<!--more-->

# i.MXRTとは？
NXP社のマイクロコントローララインナップの一つでありクロスオーバープロセッサーと呼ばれている。

通常のマイコンとi.MX8シリーズ(CortexA53)の中間に位置するような高性能マイコンなのでクロスオーバーなんて名前が付いているみたいです。

i.MXRTシリーズには現在8種類のラインナップが存在する。
{{< figure src="../../img/get_start1.png"class="center" width="100%" height="100%" >}}


もう少し正確なスペックが知りたい方は、[NXP i.MXRT Series](https://www.nxp.com/products/processors-and-microcontrollers/arm-microcontrollers/i-mx-rt-crossover-mcus:IMX-RT-SERIES)に飛んで各種製品ページを見てみてください。

ただ単に、i.MXRT使いましょうと言われても採用モチベーションがないと思います。  
そのため自分が使用している理由やモチベーションを箇条書きでまとめて見ました。

- ARM CortexM系の中で最もスペックが高い
- チップ単価がものすごく安い
- SAI/I2Sのようなオーディオ機能がある
- USB 2.0 OTG を使えるしかも内蔵PHY付き

上げればきりがないですが、オーディオ開発を行うには十分なスペックを持っていると言えるでしょう。

簡単ではありますがi.MXRTシリーズの紹介を終えて次の項から環境構築に移ります。


# 環境構築

ここからはi.MXRTシリーズを扱うための環境を構築していきます。必要なものを箇条書きで示します。

- MCUXpressoIDE : NXP社の統合開発環境
- MCUXPressoSDK : 開発に必要なSDK(MIMXRT1010-EVK向けのもの)
- MIMXRT1010-EVK : RT1010の公式評価ボード

この入門記事では、一番安価に手に入るMIMXRT1010-EVKを用いることにします。  
5000円ぐらいで手に入るためDigikeyなどで購入してください。
# MCUXpresso IDEのインストール

MCUXpressoIDEをダウンロードしてインストールします。アカウントを持っていなければこの時点で作成することになります。

[MCUXpressoIDEのトップページ](https://www.nxp.com/design/software/development-software/mcuxpresso-software-and-tools/mcuxpresso-integrated-development-environment-ide:MCUXpresso-IDE)

リンク先でWindows,Mac,Linux向けのものを選択できるので、自身の環境にあったものを選んでください。

ダウンロードが終わりましたら各自インストールを進めてください。

# MCUXpresso SDKの作成

次に開発に必要なSDKをダウンロードしていきます。  
MCUXpressoの場合,自身が必要なSDKをブラウザ上で作成し、ダウンロードするという仕組みをとっています。

[MCUXPressoSDK Builder](https://mcuxpresso.nxp.com/en/welcome)

リンク先にアクセスするとトップページが表示されると思いますのでSelect Development Boardをクリックしてボード選択画面に移動します。

ボード選択画面でMIMXRT1010-EVKのものを選択します。

{{< figure src="../../img/get_start2.png"class="center" width="100%" height="100%" >}}

次にミドルウエアを選択する画面が現れます。

{{< figure src="../../img/get_start3.png"class="center" width="100%" height="100%" >}}

ミドルウエアは、デフォルトで選択されているままでも良いですが以下のものを選択しておくと後々楽です。

- CMSIS DSP Library : ARM社純正の信号処理ライブラリ
- FatFs : SDカード、USBメモリを扱う際に使う
- USB stack : USB通信を行うライブラリ
- Amazon-FreeRTOS : リアルタイムOS

ミドルウエアは今後使う可能性があるものを選んでいるのでこの場で理解していなくても構いません。

次にToolchain / IDEでMCUXpressoIDEを選択しOSは各自自分にあうものを選択してください。

すべての項目が設定し終わったら底部のDownload SDKをクリックしSDKを作成します。(これには多少時間がかかる場合があります)

ビルドが終わったらSDKのzipファイルをダウンロードします。

# SDKのインストール
次にMCUXpressoIDEに開きます。
IDE下部のInstall SDKというタブがあるのでそこに先程ダウンロードしたzipファイルをD&DしSDKをインストールします。　

{{< figure src="../../img/get_start4.png"class="center" width="100%" height="100%" >}}

以上で環境構築は完了です。


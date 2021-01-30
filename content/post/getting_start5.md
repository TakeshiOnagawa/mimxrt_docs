---
author: "Takeshi Ogawa"
title: "Getting start(5) : ペリフェラルの組み合わせでアプリケーションを作る"
date: "2020-06-05"
description: ""
tags: ["RT1010"]
draft: false
---
この記事は、Getting started with i.MXRT microcontroller (4)の続きです。

<!--more-->
前回までの内容で以下に示すIDEのツール機能を学習した。
- <font color="MediumBlue">Import SDK example機能</font>
- <font color="MediumBlue">New project機能</font>
- <font color="MediumBlue">MCUXpresso Config Tools Pins</font>
- <font color="MediumBlue">MCUXpresso Config Tools Clocks</font>
- <font color="MediumBlue">MCUXpresso Config Tools Peripherals</font>
  
今回は各種ツールを駆使し、ペリフェラル同士を組み合わせた一つのアプリケーションを作成します。  
今回行う内容をリストで示します。

- スイッチを押したらLEDがつく
- 再度スイッチを押したらLEDが消える
- スイッチにはチャタリング対策を入れる

一見簡単なように見えますが、GPIO入出力、タイマーを使ったチャタリング対策といったようにボリュームとしては十分かなと思います。


# 新規プロジェクト作成とピンの設定

新規プロジェクトの作成を行います。(PITのドライバを選択する。)

プロジェクトウィザートでドライバを選択し忘れたり、あとから使いたいペリフェラルが増えるといったことがあると思います。  
まずプロジェクトエクスプローラー上でプロジェクトを選択します。  
次に、画像示すアイコンをクリックするとSDK manageのウィンドウが開きます。  
使いたいペリフェラルにチェックを入れてOK押すとプロジェクトにドライバの追加を行うことができます。

{{< figure src="../../img/get_start29.png"class="center" width="100%" height="100%" >}}

プロジェクト作成とドライバの追加が完了したらピンの設定を行います。  
今回はスイッチもあるので2本のピンを設定する必要があります。(GPIO_11,GPIO_SD_05)

{{< figure src="../../img/get_start30.png"class="center" width="100%" height="100%" >}}

ここで注意したいのが、スイッチのピン設定です。  
MIMXRT1010-EVKの場合はスイッチの反対側のピンはGNDにつながっているのでマイコン側はプルアップの設定にしておきます。  
このように設定を行うことでスイッチが押されたときにはピンの状態がLOWとなります。  
設定ができたらUpdate codeをクリックしコードを出力してください。

次にGPIOの入出力が正しく設定されているかテストコードを実行して確認します。

```c++
int main(void) {

  	/* Init board hardware. */
    BOARD_InitBootPins();
    BOARD_InitBootClocks();
    BOARD_InitBootPeripherals();
  	/* Init FSL debug console. */
    BOARD_InitDebugConsole();

    PRINTF("Hello World\n");

    /* Force the counter to be placed into memory. */
    volatile static int i = 0 ;
    volatile int state = 0;
    /* Enter an infinite loop, just incrementing a counter. */
    while(1) {
        i++ ;
        /* 'Dummy' NOP to allow source level single stepping of
            tight while() loop */
        __asm volatile ("nop");
        state = GPIO_PinRead(GPIO2,5);//スイッチの状態をスキャン

        if(state == 0){
        	GPIO_PinWrite(GPIO1, 11, 1);//押されていたらLEDがON
        }
        else{
        	GPIO_PinWrite(GPIO1, 11, 0);//押されていなければLEDはOFF
        }
    }
    return 0 ;
}
```
スイッチを押している間だけLEDが光ると思います。
{{< figure src="../../img/get_start31.png"class="center" width="100%" height="100%" >}}

こういった単機能でのテストは重要なので、必ず行いましょう。

# PITの設定
次にタイマーのセットアップをしていこうと思います。
クロックは前回同様に12.5MHzに設定します。

Channel period/freqは、スイッチのスキャン周期を1kHzにしたいので1msに設定しました。  
その他の設定などは前回同様です。
{{< figure src="../../img/get_start32.png"class="center" width="100%" height="100%" >}}

# コードの記述
各種ペリフェラルの設定ができたのでコードを記述していきます。

```c++
/*
 * Copyright 2016-2020 NXP
 * All rights reserved.
 *
 * Redistribution and use in source and binary forms, with or without modification,
 * are permitted provided that the following conditions are met:
 *
 * o Redistributions of source code must retain the above copyright notice, this list
 *   of conditions and the following disclaimer.
 *
 * o Redistributions in binary form must reproduce the above copyright notice, this
 *   list of conditions and the following disclaimer in the documentation and/or
 *   other materials provided with the distribution.
 *
 * o Neither the name of NXP Semiconductor, Inc. nor the names of its
 *   contributors may be used to endorse or promote products derived from this
 *   software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
 * ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
 * WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR
 * ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
 * (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
 * LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON
 * ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
 * (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
 * SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 */
 
/**
 * @file    switch_test.c
 * @brief   Application entry point.
 */
#include <stdio.h>
#include "board.h"
#include "peripherals.h"
#include "pin_mux.h"
#include "clock_config.h"
#include "MIMXRT1011.h"
#include "fsl_debug_console.h"
/* TODO: insert other include files here. */

/* TODO: insert other definitions and declarations here. */

/*
 * @brief   Application entry point.
 */
#define DEBOUNCE_TIME       0.02 // 0.02s
#define SAMPLE_FREQUENCY    1000 // 1msに一回サンプリング(PITに依存)
#define MAXIMUM         	(DEBOUNCE_TIME * SAMPLE_FREQUENCY)//integratorの最大値が決まる
#define SW_ON  0
#define SW_OFF 1
uint32_t input;//スイッチ入力
uint32_t integrator;//ONカウント
uint32_t output;//出力
uint32_t counter;//長押し判定用
uint32_t old_output = 0;
uint32_t state = 0;

void ScanSwitch_Task(){
	input = GPIO_PinRead(GPIO2,5); // 入力をスキャン
	if (input == SW_OFF)
	{
		if (integrator > 0){//入力がLOWだったら
		  integrator--;
		}
	}
	else if (integrator < MAXIMUM){
		integrator++;
	}

	if (integrator == 0){
		output = 0;
		counter = 0;
	}
	else if (integrator >= MAXIMUM)
	{
	output = 1;
	counter++;
	integrator = MAXIMUM;  /* defensive code if integrator got corrupted */
	}
}

void LedTask(){
	if(output == 1 && old_output == 0){
	    		state = !state;
	    	}
	    	old_output = output;

	    	if(state == 1){
	    		GPIO_PinWrite(GPIO1, 11, 1);

	    	}
	    	else{
	    		GPIO_PinWrite(GPIO1, 11, 0);
	    	}
}

/* PIT_IRQn interrupt handler */
void PIT_IRQHANDLER(void) {
	/*  Place your code here */
	/* Add for ARM errata 838869, affects Cortex-M4, Cortex-M4F
	 Store immediate overlapping exception return operation might vector to incorrect interrupt. */
	ScanSwitch_Task();
	LedTask();
	PIT_ClearStatusFlags(PIT_PERIPHERAL, PIT_CHANNEL_0,kPIT_TimerFlag);
	#if defined __CORTEX_M && (__CORTEX_M == 4U)
	__DSB();
	#endif
}


int main(void) {

  	/* Init board hardware. */
    BOARD_InitBootPins();
    BOARD_InitBootClocks();
    BOARD_InitBootPeripherals();
  	/* Init FSL debug console. */
    BOARD_InitDebugConsole();

    PRINTF("Hello World\n");

    /* Force the counter to be placed into memory. */
    /* Enter an infinite loop, just incrementing a counter. */
    while(1) {

    }
    return 0 ;
}

```


ScanSwitch_Taskの中でチャタリング除去を行っています。
今回はMAXIMUMが20となるように値をセットしていますが、GPIOのスキャン周期(PITのインターバル設定)を変更した場合は調整してください。

LedTaskでは、自己保持の記述をしています。具体的にはスイッチを押したらLEDが付き続け、もう一度押すとLEDが消えるという動作です。

今回は使っていませんがcounterという変数を追加しているため長押しの検出もできます。

チャタリング対策に関してはRefにもリンクを張りましたがDEBOUNCE CODEのKENNETH KUHNさんが実装されたものをカスタマイズして使用しています。

# 実行とデバッグ
プログラムの記述ができたのでビルドを行いボードにプログラムを書き込み実行します。。
  
ボード上のタクトスイッチを押すごとにLEDがついたり消えたりすることが確認できると思います。

次にデバッグをしてみます。 
今回はGlobal Variablesというオンタイムで変数の値を確認できるデバッグ機能を使ってみます。

{{< figure src="../../img/get_start33.png"class="center" width="100%" height="100%" >}}

Global Variablesは、画像を参考に開いてください。  
次にデバッガを立ち上げます。Global Variablesのウィンドウ内のメガネのアイコンをクリックし、変数選択を行います。  
ここでは、counter,integratorを選択します。

{{< figure src="../../img/get_start34.png"class="center" width="100%" height="100%" >}}

変数のチェックを有効にし、更新頻度を500msに設定します。  
最後に再生ボタンをクリックするとオンタイムで変数の変化が見れるようになります。  


{{< figure src="../../img/get_start35.png"class="center" width="100%" height="100%" >}}
ボード上のタクトスイッチを長押ししてみてください。  
画像のように長押ししている間counter変数が増加する様子が確認できます。

# まとめ
Getting start(1) ~ (4)で学んできたことを組み合わせることで一つのアプリケーションを組むことを体験しました。

各種コンフィグ機能を使って開発を行うことで複数ペリフェラルを組み合わせた際でも少ない労力でアプリケーションを作成可能であること、IDEの強力なデバッグ機能を駆使することで効率的な開発が可能であることを学べたと思います。

以上でGetting startシリーズは終了しますが、その他のTips記事などを読むことで更に上位レベルの開発を行う事ができるようになります。

今回学んだMCUXpressoのPins,Clocks,Peripheralsの機能は今後の開発にも重要なツールになっていますので記事を読み返して復習したりご自分で新しいペリフェラルに挑戦するなりして知見を増やしていただけたら幸いです。

# Ref

- [スイッチのチャタリングとは](https://www.marutsu.co.jp/pc/static/large_order/1405_311_ph)
- [DEBOUNCE CODE – ONE POST TO RULE THEM ALL](https://hackaday.com/2010/11/09/debounce-code-one-post-to-rule-them-all/)
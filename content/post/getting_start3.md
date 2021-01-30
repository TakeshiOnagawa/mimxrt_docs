---
title: "Getting start(3) : ピン設定ツールの使い方"
date: "2020-05-30"
description: ""
tags: ["RT1010"]
draft: false
---

この記事は、Getting started with i.MXRT microcontroller (2)の続きです。  
<!--more-->
前回は、サンプルプロジェクトをインポートし実行する方法を学習しました。  
本記事では、新規プロジェクトを作成します。  
作業としては、クロックの設定、ピン機能の設定、ペリフェラルの初期化を行う必要がありより本格的な内容になります。

<font color="Crimson">作業環境は、Getting started with i.MXRT microcontroller (1)のものを利用します。</font>

# 新規プロジェクトを作成
MCUXpressoIDEを立ち上げて左下のQuickstart PanelのNew projectをクリックしウィザードを開きます。

次に評価ボードのサムネイルが表示されるので、evkmimxrt1010を選択してNextをクリックします。

Configure the projectの画面に遷移しますのでプロジェクトの名前を入力してください。  
今回は、GPIOのみの使用なのでドライバはデフォルトのままで良いのでそのままNEXTをクリックします。

{{< figure src="../../img/get_start13.png"class="center" width="100%" height="100%" >}}

Advanced project settingsも特に操作しなくていいと思います。

プロジェクトが作成されたらビルドをして正しく作成されていることを確認します。

# ピンの設定をする。
次にLEDに使うピンを出力ピンに設定する作業を行います。  
一本ぐらいなら手動でコードを書いてもいいのですが、今後の開発で設定するピンが増えてくるとバグの原因にもなるのでコンフィグツールを使います。  

PinコンフィグレータはIDEの右上のプラスアイコンをクリックし表示されるウィンドウ内のPinsを選択することで開くことができます。

{{< figure src="../../img/get_start14.png"class="center" width="100%" height="100%" >}}

Pinコンフィグレータが開くとなにもないウィンドウが開かれるので、先程作成したプロジェクトを選択します(ピンク色で示した部分)。  

プロジェクトが読み込まれると複雑な画面が表示されると思います。  

{{< figure src="../../img/get_start15.png"class="center" width="100%" height="100%" >}}

ここで画面構成について少し解説します。(<font color="CadetBlue">青文字の番号と対応しています。</font>)

1. Functional Group : コンフィグレータで設定し出力されるコードをどの関数で実行するか選べるタブ
2. Pinsタブ : Route済みのピンを表示したり、フリーなピンを表示したりできる。またそのピンにどんな機能がついているか確認することができる。
3. Packageタブ : パッケージのピンを見ながらルーティングできる画面。中央のペリフェラルをクリックすると黄色くハイライトされどこから信号を出すか選択できる。
4. Routed Pinsタブ : 自身がRouteしたピンがここに表示される。GPIOの入出力もここでできる。
5. Problemsタブ : ピンで重複が発生したりするとエラーが表示される。
6. ウィンドウ切り替え : メイン画面に戻りたいときは工具のアイコンを押す

画面構成が理解できたと思うのでLEDのピンを設定していきます。  

{{< figure src="../../img/get_start16.png"class="center" width="100%" height="100%" >}}

画像の手順通りにGPIO_11にRouteしてDirectionをOutputに設定します。

{{< figure src="../../img/get_start17.png"class="center" width="100%" height="100%" >}}

次に上部メニューのUpdate Codeをクリックして現在追加した設定コードを出力します。

そうすると画面が元のワークスペースに戻る。
実際に出力されたコードを確認する。(<font color="CadetBlue">board/pin_mux.c</font>)


{{< highlight c "linenos=inline" >}}
void BOARD_InitPins(void) {
  CLOCK_EnableClock(kCLOCK_Iomuxc);           /* iomuxc clock (iomuxc_clk_enable): 0x03U */

  /* GPIO configuration of GPIO_11 on GPIO_11 (pin 1) */
  gpio_pin_config_t GPIO_11_config = {
      .direction = kGPIO_DigitalOutput,
      .outputLogic = 0U,
      .interruptMode = kGPIO_NoIntmode
  };
  /* Initialize GPIO functionality on GPIO_11 (pin 1) */
  GPIO_PinInit(GPIO1, 11U, &GPIO_11_config);

  IOMUXC_SetPinMux(
      IOMUXC_GPIO_11_GPIOMUX_IO11,            /* GPIO_11 is configured as GPIOMUX_IO11 */
      0U);                                    /* Software Input On Field: Input Path is determined by functionality */
  IOMUXC_GPR->GPR26 = ((IOMUXC_GPR->GPR26 &
    (~(IOMUXC_GPR_GPR26_GPIO_SEL_MASK)))      /* Mask bits to zero which are setting */
      | IOMUXC_GPR_GPR26_GPIO_SEL(0x00U)      /* Select GPIO1 or GPIO2: 0x00U */
    );
}
{{< /highlight >}}

2行目でGPIOにクロック供給を行っています。  
11行目で構造体を引数にGPIOを初期化  
それ以降はPinMuxの設定をしています。  
ピン一つを出力にするのに毎回このコードを書かないといけないと考えるとGUIコンフィグツールを使うほかないと思います。

ここまででピン設定は終了のため一度プロジェクトをビルドしておきましょう。

# Lチカプログラムの作成
最後にLチカを行うコードを記述します。  
前回のサンプルコードでは、for文の無限ループにより点滅のインターバルを作成をしていましたが今回はタイマーを使ってみます。

タイマーはSystickタイマーというものを使います。   
Systickタイマーはこのチップ特有の機能ではなくCortexM4以上のマイコンであれば必ず内蔵されているタイマーになります。(CortexM0などはものによる)   

使い方に関しては、説明するよりコードを見てもらったほうが早いと思うので、メインコードを貼ります。

```c++
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
uint32_t delayCount;

void SysTick_Handler(void)
{
    if(delayCount != 0x00)
    {
    	delayCount--;
    }
}

void Systick_delay(uint32_t ms){
	SysTick->LOAD  = (SystemCoreClock/1000 & SysTick_LOAD_RELOAD_Msk) - 1;
	delayCount = ms;
	while(delayCount!=0x00);
}

int main(void) {

  	/* Init board hardware. */
    BOARD_InitBootPins();
    BOARD_InitBootClocks();
    BOARD_InitBootPeripherals();
  	/* Init FSL debug console. */
    BOARD_InitDebugConsole();

    PRINTF("Hello World\n");
    SysTick_Config(SystemCoreClock/1000-1);
    /* Force the counter to be placed into memory. */
    volatile static int i = 0 ;
    /* Enter an infinite loop, just incrementing a counter. */
    while(1) {
        i++ ;
        GPIO_PinWrite(GPIO1, 11, 0);
        Systick_delay(200);
        GPIO_PinWrite(GPIO1, 11, 1);
        Systick_delay(200);
        /* 'Dummy' NOP to allow source level single stepping of
            tight while() loop */
        __asm volatile ("nop");
    }
    return 0 ;
}
```

ディレイ関数はms単位で入力できるように作りました。  
```
Systick_delay(200);
```
のように書くと200msディレイと言うことになります。   
GPIOの操作に関しては、<font color="CadetBlue">GPIO_PinWrite</font>というAPIを使います。

```
GPIO_PinWrite(GPIO1, 11, 0);　 // GPIO OFF
GPIO_PinWrite(GPIO1, 11, 1);　 // GPIO ON
```

以上のプログラムが用意できたら、前回学習したようにプロジェクトをビルドして書き込みを行ってください。

うまく書き込みができればボード上のLEDが光ると思います。  
今回の作業はこれで終了です。

# Ref
- MIMXRT1010-EVK Schematic
- [STM32SystickTimerを稼働させる](https://qiita.com/ekd/items/731c16c314c52cca0d53)
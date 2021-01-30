---
title: "Getting start(4) : クロックおよびペリフェラル設定ツールの使い方"
date: "2020-06-03"
description: ""
tags: ["RT1010"]
draft: false
---

この記事は、Getting started with i.MXRT microcontroller (3)の続きです。  
<!--more-->
前回までで新規プロジェクトからLチカプログラムを作成しました。  
ここで今まで学習したIDEの機能と未学習の機能を整理したいと思います。

- <font color="MediumBlue">Import SDK example機能</font>
- <font color="MediumBlue">New project機能</font>
- <font color="MediumBlue">MCUXpresso Config Tools Pins</font>
- MCUXpresso Config Tools Clocks
- MCUXpresso Config Tools Peripherals

青色で示した部分が学習済みの機能で以下2つはまだ学習していない機能です。  
今回は下2つのClocks,Peripheralsの2つを学習するために内蔵ペリフェラルの一つであるPIT(Periodic Interrupt Timer)を使ったLチカをやってみようと思います。

今回PITを選んだ理由は、クロックおよび、ペリフェラルの設定が必須であるが比較的容易に扱えるので、ツール学習に支障をきたさないだろうと考えたためです。

では、本題に入りましょう。
 
# 新規作成と前回の復習
新規プロジェクトを作成します。

Configure the projectのウインドウで、<font color="Crimson">pitのドライバにチェックを入れます。</font>
{{< figure src="../../img/get_start18.png"class="center" width="100%" height="100%" >}}
次のAdvanced project settingsは何もいじらずFinishをクリックします。  
プロジェクトが生成されたら、前回同様にLEDがつながっているGPIO_11の設定をしてください。  
その後ビルドを行いエラーがないことを確認します。

# クロックの設定
次にPITのクロックを設定していきます。  
ここで初めてクロック設定ツールが出てきます。  
クロックツールの開き方はPinsツール同様にIDE右上のアイコンから行います。

{{< figure src="../../img/get_start19.png"class="center" width="100%" height="100%" >}}

クロックツールを開くと複雑な画面構成となっているため解説を入れます。

{{< figure src="../../img/get_start20.png"class="center" width="100%" height="100%" >}}

多くのタブがありますが、主に使用するのは画像に表示している物が多いです。  
基本的には設定したいクロックを右のClock Consumerタブで選択しハイライトされる部分をいじってクロックを設定します。

ここではPITクロックを<font color="Crimson">12.5MHz</font>に設定してみることにします。
設定方法についてですが、二種類ほどあります。
<details><summary><font color="MediumBlue">設定方法(1):クロック図上で行こなう方法</font></summary><div>
この方法はクロック図を直接クリックして設定していく方法です。

各分周器をクリックすることでGUI上で設定を行うことができます。
{{< figure src="../../img/get_start21.png"class="center" width="100%" height="100%" >}}

右クリックで設定画面を出すこともできます。
{{< figure src="../../img/get_start22.png"class="center" width="100%" height="100%" >}}

目的クロックを直接入力して設定させることもできますが、LOCKされてない分周器は値が変わってしまうので注意してください。

{{< figure src="../../img/get_start23.png"class="center" width="100%" height="100%" >}}

設定値を満たせない場合エラーが出るのでこの方法はあまり使わないほうが無難です。

</div></details>

<details><summary><font color="MediumBlue">設定方法(2):Detailsタブより行う方法</font></summary><div>
次にDetailsタブより設定する方法です。  
クロック図側でPITをクリックしハイライトさせた上でDetailsタブを確認すると該当部分が黄色く強調されるのでその部分を設定します。
{{< figure src="../../img/get_start24.png"class="center" width="100%" height="100%" >}}
</div></details>
 
以上の例では、12.5MHzにクロックを設定していますが、この部分はどんなクロックに設定してもOKです。  
クロックを早くすれば1カウントあたりの時間が短くなるので時間分解能が上がります。
逆に長いインターバルを作成したければクロックを下げればいいです。


# ペリフェラルの設定
次にペリフェラルの設定を行います。  
ペリフェラルツールはピンツール、クロックツール同様に右上のアイコンから開くことが可能です。

ペリフェラルウィンドウが表示されましたら左のペリフェラルタブよりPITを選択します。  
その後PITの上をダブルクリックするか、右クリックでOpenするとPITの設定画面が表示されます。

{{< figure src="../../img/get_start25.png"class="center" width="100%" height="100%" >}}

PITの設定が画面が表示されたと思うので実際に設定をしていきます。

{{< figure src="../../img/get_start26.png"class="center" width="100%" height="100%" >}}

設定する項目は、主に以下の2つになります。
- 割り込み有効化
- channel period/frequency

詳しい設定は画像を参考にしながら行ってください。

以上でPITの設定は終了なので上部メニューのupdate codeをクリックしてコードを出力してください。  
このように目的の値を設定するだけでレジスタ設定のコードを出力してくれるので時間をかけることなく開発をすすめることができます。


<details><summary><font color="MediumBlue">割り込みについて</font></summary><div>

途中で割り込みという言葉が出てきますが、話が脱線してしまうのでここでは解説しません。  
気になる方のために参考文献に[とても良い資料(割り込みとポーリング)](#link1)があったので貼っておきます。

</div></details>

# Lチカのためのコードを書く
最後にコードを書いていきます。  
ほとんどのコードをツールが出力してくれたので私達が書かないといけないのは割り込みが発生したときにどういった動作をするか？という部分だけです。

まず先程のペリフェラルツールの画面に戻り、Handler templateにあるボタンをクリックします。 

{{< figure src="../../img/get_start27.png"class="center" width="100%" height="100%" >}}

ここでHandlerという言葉が初めて出てきます。  
ARMマイコンでは割り込みが発生するとhandlerと呼ばれるルーチンに飛びます。  
handlerの名称はメーカーのSDKごとに異なっておりNXPのPITではPIT_IRQHANDLERという名前で定義されています。

今回はLチカを行いたいので、Handler内ではGPIOの反転動作と割り込みのクリアを行います。  
割り込みのクリアは割り込みが発生するとフラグが立つことから、それをクリア状態にするため絶対に必要です。

実際の記述例を示します。
```c++
/* PIT_IRQn interrupt handler */
void PIT_IRQHANDLER(void) {
	/*  Place your code here */
	/* Add for ARM errata 838869, affects Cortex-M4, Cortex-M4F
		Store immediate overlapping exception return operation might vector to incorrect interrupt. */
	/*ユーザーコードはここに書く*/
	GPIO_PortToggle(GPIO1, 1U<<11); //GPIO_11を反転
	PIT_ClearStatusFlags(PIT_PERIPHERAL, PIT_CHANNEL_0, kPIT_TimerFlag);
	#if defined __CORTEX_M && (__CORTEX_M == 4U)
	__DSB();
	#endif
}
```

GPIOの操作に関しては、反転を行うAPIで該当bitを反転させています。
PIT_ClearStatusFlagsは、割り込みのフラグ解除を行うAPIです。  
前2つの引数はperipherals.hでマクロ定義されているもので、自分が使うチャンネルに応じて書きます。  
3つ目の引数は、どのフラグをクリアするのかを設定します。  
PITの場合kPIT_TimerFlag以外の選択肢がないので上記のように記述すればよいです。
割り込みの条件が複数あるようなペリフェラルの場合は該当する動作のフラグをクリアにする必要があります。

以上でコードの定義が終わりです。

# 実行してみる
コードの記述まで終わったのでビルドを行いエラーがないことを確認します。  
ビルドが無事に通ったらデバッガを起動して割り込みがきちんと動作しているか確認します。
{{< figure src="../../img/get_start28.png"class="center" width="100%" height="100%" >}}

ハンドラー内でブレークポイントを有効にすると、1秒毎にプログラムが停止すると思います。



# Ref<a name="link1"></a>
- [MIMXRT10xx Periodic Interrupt Timer(基礎編)](https://gsmcustomeffects.hatenablog.com/entry/2018/02/06/000117)
- [MIMXRT10xx Periodic Interrupt Timer(タイマーのチェイン)](https://gsmcustomeffects.hatenablog.com/entry/2018/02/10/002550)
- [割り込みとポーリング](https://www.uquest.co.jp/embedded/learning/lecture02.html)
 
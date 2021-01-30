---
title: "アプリケーションのRAM配置について"
date: "2020-06-09"
description: ""
tags: ["Tips"]
draft: false
---

MCUXpressoの機能を使って簡単なRAM実行を行う手法の紹介
<!--more-->

i.MXRTシリーズは、本体内部にフラッシュメモリを持っておらず外部フラッシュより起動するのが基本となる。  
近年のマイコンでは、外部メモリから直接実行できる仕組みが備わっておりその機能のことをeXecute In Placeの頭文字をとってXIPと呼ぶ。
こういった実行機能は非常に便利であるが外部メモリバスを介して実行を行うためバス速度がネックとなってしまいます。  
そんなわけで、i.MXRTシリーズで高速にプログラムを実行するには内臓のRAMにコードを配置し実行する必要があります。
# RAM配置
MCUXpressoでは、デフォルトで外部フラッシュにプログラムを配置するようになっています。
一般にプログラムをRAMに配置するためにはリンカースクリプトの修正を行う必要がありますが、MCUXPressoにはリンカースクリプトを自動生成する機能が備わっているため簡単に行うことができます。

プロジェクトを右クリックしてpropertiesを開く。  
次にC/C++ Build/Setting/MCU Linker/Managed Linker Scriptタブを開く。
最後にLink application to RAMにチェックを入れる。
{{< figure src="../../img/ram_execute1.png"class="center" width="100%" height="100%" >}}

終わったらapply and closeします。

プロジェクトを再ビルドするとSRAMにリンクされていることがわかると思います。
```
Memory region         Used Size  Region Size  %age Used
     BOARD_FLASH:          0 GB        16 MB      0.00%
        SRAM_ITC:       22112 B        32 KB     67.48%
        SRAM_DTC:          0 GB        32 KB      0.00%
         SRAM_OC:          0 GB        32 KB      0.00%
   NCACHE_REGION:          0 GB        32 KB      0.00%
Finished building target: switch_test.axf
```

# 実行と確認
先程のプログラムでデバッガを立ち上げるとRAM上でプログラムが実行されていることがわかります。

{{< figure src="../../img/ram_execute2.png"class="center" width="100%" height="100%" >}}

ビルド後のバイナリを解析することでも詳細情報を確認することが可能です。

{{< figure src="../../img/ram_execute3.png"class="center" width="100%" height="100%" >}}

ITCMは0x0番地から始まるのできちんとリンクされていることが確認できると思います。


# advanced topics

<details><summary><font color="MediumBlue">Q:ITCM以外にプログラムをリンクすることはできないの？</font></summary><div>

A:できます
</font>

Properties/C/C++ Build/MCU settingにてmemory detailで順序を変更することで可能です。
{{< figure src="../../img/ram_execute4.png"class="center" width="100%" height="100%" >}}

```
Memory region         Used Size  Region Size  %age Used
     BOARD_FLASH:          0 GB        16 MB      0.00%
        SRAM_DTC:       22112 B        32 KB     67.48%
        SRAM_ITC:          0 GB        32 KB      0.00%
         SRAM_OC:          0 GB        32 KB      0.00%
   NCACHE_REGION:          0 GB        32 KB      0.00%
```
</div></details>

<details><summary><font color="MediumBlue">Q:関数ごとに配置を切り替えたりできないの？</font></summary><div>

A:できます  
__attribute__((section("name")))というGNU コンパイラの拡張機能を使って配置を明示的に指定できます。

"name"にはリンカースクリプト内で宣言されているシンボルを書きます。

```c++
__attribute__((section(".ramfunc.$SRAM_ITC"))) void helloTask();

void helloTask(){
	PRINTF("Hello World\n");
}
```

```
Memory region         Used Size  Region Size  %age Used
     BOARD_FLASH:          0 GB        16 MB      0.00%
        SRAM_DTC:       22132 B        32 KB     67.54%
        SRAM_ITC:          32 B        32 KB      0.10%
         SRAM_OC:          0 GB        32 KB      0.00%
   NCACHE_REGION:          0 GB        32 KB      0.00%
```
</div></details>


# まとめ
今回は、RAMにプログラムをリンクする方法を紹介した。

この手法はデバッガでRAMにバイナリを送り込むためアルゴリズムの速度検証とかには便利かと思います。 

<font color="Crimson">当たり前ですが電源を落とすとプログラムは消えます。

本格的にRAM展開を考えるならXIPで起動後にRAM転送しRAMにジャンプするようなブートローダーを作成する必要があります。
</font>

個人的にはRAMが少ないような機種では高速化したいものだけRAM配置をしたり、CortexM7のキャッシュに頼るといった方法が考えられます。
# Ref

- [Execute In Place](https://www.design-reuse.com/articles/41861/execute-in-place-xip-nor-flash-spi-protocol.html)
- [ARMリファレンスガイド](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0348bj/Caccache.html)
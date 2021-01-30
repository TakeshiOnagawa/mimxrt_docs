---
title: ".incbinを利用したバイナリファイルのリンク"
date: "2020-05-04"
description: ""
tags: ["Tips"]
---

ARMのアセンブラ.incbinにより実行ファイルにバイナリファイルをリンクする方法の紹介です。
<!--more-->

音声ファイル(wav),binファイルだったりを実行ファイルにリンクしプログラム内から利用できるのでちょっとした実験などに便利です。

# マクロの作成

使い回せると便利なのでマクロを使用します。  
下記のdefineマクロをソース冒頭に貼り付けてください


```c 
#define	IMPORT_BIN(sect, file, sym , ofs) asm (\
".section " #sect "\n"                  /* Change section */\
".balign 4\n"                           /* Word alignment */\
".global " #sym "\n"                    /* Export the object address to other modules */\
#sym ":\n"                              /* Define the object label */\
".incbin \""file"\","#ofs" \n"                /* Import the file */\
".global _sizeof_" #sym "\n"            /* Export the object size to other modules */\
".set _sizeof_" #sym ", . - " #sym "\n" /* Define the object size */\
".balign 4\n"                           /* Word alignment */\
".section \".text\"\n")                 /* Restore section */
```
各種引数は、以下の表で解説しています。  

|  引数  |  説明  |
| ---- | ---- |
|  sect  |  リンカースクリプトで定義されているsection  |
|  file  |  fileのパスを指定  |
|  sym   |  変数シンボル   |
|  ofs    |  オフセット   |

# 使い方
PATHの部分は適宜読み替えてもらって、このようにして使用します。
FooBinを参照すればリンクしたデータを扱うことができます。
```c 
IMPORT_BIN(".rodata","../source_transfer/ss.wav",FooBin,40);
extern const char FooBin[], _sizeof_FooBin[];
```

あとはコンパイルしてみてメモリ使用量と相談しながら使ってください.  
(wavなんかをリンクすると普通に数MB行くので注意)


# 参考文献
- [RealView Compilation Tools アセンブラガイド](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0204ij/Caccaghf.html)
- [プログラムにバイナリーファイルを組み込む方法](http://www.narimatsu.net/blog/?p=4650)

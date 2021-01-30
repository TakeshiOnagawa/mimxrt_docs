---
author: "Takeshi Ogawa"
title: "sai peripheralsのメモ"
date: "2020-06-10"
description: ""
tags: ["Tips"]
draft: true
---
マスクレジスタとかよくわからなすぎるので設定のメモ  
<!--more-->

<font color="Crimson">今の所自分用メモです。理解したらまた書き直します。</font>

## ワードマスクについて
ワードマスクレジスタ(TMR,RMR)でセットした値が破棄される機能  
おそらくTDMとかで2ch以上のデータ受け取るとき特定のワードだけ破棄するためにあると思う。

## クロック周り
GUIツール側のBLCKの値が半分に表示されることがある。

## FIFO
FIFO Error flag : receive overflow detect
FIFO Warining flag : Enable receive FIFO is full
FIFO Request : Receive FIFO watermark has been reached.
---
author: "Takeshi Ogawa"
title: "メモリサイズの考え方"
date: "2020-05-03"
description: ""
tags: ["Tips"]
---

組み込み開発をやっているとリンカースクリプトだったりでメモリのサイズを規定したりすることがある。
<!--more-->
i.MXRTのリンカースクリプトをリストに示すが、メモリーのサイズがhexで書かれており慣れていないとわかりにくいと思う

```c
MEMORY
{
  /* Define each memory region */
  BOARD_FLASH (rx) : ORIGIN = 0x60000000, LENGTH = 0x800000 /* 8M bytes (alias Flash) */  
  BOARD_SDRAM (rwx) : ORIGIN = 0x80000000, LENGTH = 0x1e00000 /* 30M bytes (alias RAM) */  
  NCACHE_REGION (rwx) : ORIGIN = 0x81e00000, LENGTH = 0x200000 /* 2M bytes (alias RAM2) */  
  SRAM_DTC (rwx) : ORIGIN = 0x20000000, LENGTH = 0x10000 /* 64K bytes (alias RAM3) */  
  SRAM_ITC (rwx) : ORIGIN = 0x0, LENGTH = 0x10000 /* 64K bytes (alias RAM4) */  
  SRAM_OC (rwx) : ORIGIN = 0x20200000, LENGTH = 0x20000 /* 128K bytes (alias RAM5) */  
}
```

0x800000の例を計算してみる。

- 0 bit : 16^0 * 0
- 1 bit : 16^1 * 0
- 2 bit : 16^2 * 0
- 3 bit : 16^3 * 0
- 4 bit : 16^4 * 0
- 5 bit : 16^5 * 8

これを全部足してあげると8388608となりこれをMbyte(1024*1024)で割ると8MBとなる。

桁数でどのぐらいの大きさか知っておくとコードも読みやすくなると思う

|  hex  |  byte  |
| ---- | ---- |
|  0x800000  |  8M  |
|  0x80000  |  512K  |
|  0x8000   |  32K   |
|  0x800    |  2K   |

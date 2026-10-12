
# 用 `BEEP` 做的《送别》

> 原文：[`www.pediy.com/kssd/pediy10/86291.html`](https://www.pediy.com/kssd/pediy10/86291.html)

看到有网上的朋友发的一篇帖子说是以前病毒用 `BEEP` 做歌。。。

我找个谱子 问了问朋友怎么看简谱。发一个《送别》 我很喜欢的一首歌...![](img/8e0d77ee22c3d8438aad7d42c7829e53.jpg "使眼色") 前面有一点有点不像 不知道哪里发音有点错误。。。

代码:

```cpp
// Music.cpp : 定义控制台应用程序的入口点。
//

#include "stdafx.h"

#define ONE_BEEP    600
#define HALF_BEEP   300

#define NOTE_1      440
#define NOTE_2      495
#define NOTE_3      550
#define NOTE_4      587
#define NOTE_5      660
#define NOTE_6      733
#define NOTE_7      825
#define NOTE_NONE   Sleep(ONE_BEEP);

int _tmain(int argc, _TCHAR* argv[])
{
  Beep(NOTE_5, ONE_BEEP);
  Beep(NOTE_3, HALF_BEEP);
  Beep(NOTE_5, HALF_BEEP);
  Beep(NOTE_1*2, ONE_BEEP);
  Beep(NOTE_6, ONE_BEEP);
  Beep(NOTE_1, ONE_BEEP);
  Beep(NOTE_5, ONE_BEEP*3);
  Beep(NOTE_5, ONE_BEEP);
  Beep(NOTE_1, HALF_BEEP);
  Beep(NOTE_2, HALF_BEEP);
  Beep(NOTE_3, ONE_BEEP);
  Beep(NOTE_2, HALF_BEEP);
  Beep(NOTE_1, HALF_BEEP);
  Beep(NOTE_2, ONE_BEEP*4);

  Beep(NOTE_5, ONE_BEEP);
  Beep(NOTE_3, ONE_BEEP);
  Beep(NOTE_5, ONE_BEEP);
  Beep(NOTE_1*2, ONE_BEEP);
  Beep(NOTE_7, ONE_BEEP);
  Beep(NOTE_6, ONE_BEEP);
  Beep(NOTE_1*2, ONE_BEEP);
  Beep(NOTE_5, ONE_BEEP*3);
  Beep(NOTE_5, ONE_BEEP);
  Beep(NOTE_2, HALF_BEEP);
  Beep(NOTE_3, HALF_BEEP);
  Beep(NOTE_4, ONE_BEEP);
  Beep(NOTE_7, ONE_BEEP);
  Beep(NOTE_1, ONE_BEEP*4);

  Beep(NOTE_6, ONE_BEEP);
  Beep(NOTE_1*2, ONE_BEEP);
  Beep(NOTE_1*2, ONE_BEEP*3);
  Beep(NOTE_7, ONE_BEEP);
  Beep(NOTE_6, HALF_BEEP);
  Beep(NOTE_7, HALF_BEEP);
  Beep(NOTE_1, ONE_BEEP*3);
  Beep(NOTE_6, HALF_BEEP);
  Beep(NOTE_7, HALF_BEEP);
  Beep(NOTE_1*2, HALF_BEEP);
  Beep(NOTE_6, HALF_BEEP);
  Beep(NOTE_6, HALF_BEEP);
  Beep(NOTE_5, HALF_BEEP);
  Beep(NOTE_3, HALF_BEEP);
  Beep(NOTE_1, HALF_BEEP);
  Beep(NOTE_2, ONE_BEEP*4);
  Beep(NOTE_5, ONE_BEEP);
  Beep(NOTE_3, HALF_BEEP);
  Beep(NOTE_5, HALF_BEEP);
  Beep(NOTE_1*2, ONE_BEEP*1/2);
  Beep(NOTE_7, HALF_BEEP);
  Beep(NOTE_6, ONE_BEEP);
  Beep(NOTE_1*2, ONE_BEEP);
  Beep(NOTE_5, ONE_BEEP*3);

  Beep(NOTE_5, ONE_BEEP);
  Beep(NOTE_2, HALF_BEEP);
  Beep(NOTE_3, HALF_BEEP);
  Beep(NOTE_4, ONE_BEEP*1/2);
  Beep(NOTE_7, HALF_BEEP);
  Beep(NOTE_1, ONE_BEEP*3);
  return 0;
}
```

`ONE_BEEP` 是一个拍子

`HALF_BEEP` 是半拍

`NOTE_X` *2 是高 8 度

听了一遍，有些谱不对。我对照简谱改了一下。

```cpp
// BeepMusic.cpp : Defines the entry point for the console application.
//

#include "stdafx.h"

#include "windows.h"

#define ONE_BEEP    600

#define HALF_BEEP   300

#define NOTE_1      440

#define NOTE_2      495

#define NOTE_3      550

#define NOTE_4      587

#define NOTE_5      660

#define NOTE_6      733

#define NOTE_7      825

#define NOTE_NONE   Sleep(ONE_BEEP);

int main(int argc, char* argv[])

{

  Beep(NOTE_5, ONE_BEEP);

  Beep(NOTE_3, HALF_BEEP);

  Beep(NOTE_5, HALF_BEEP);

  Beep(NOTE_1*2, ONE_BEEP*2);

  Beep(NOTE_6, ONE_BEEP);

  Beep(NOTE_1*2, ONE_BEEP);

  Beep(NOTE_5, ONE_BEEP*2);

  Beep(NOTE_5, ONE_BEEP);

  Beep(NOTE_1, HALF_BEEP);

  Beep(NOTE_2, HALF_BEEP);

  Beep(NOTE_3, ONE_BEEP);

  Beep(NOTE_2, HALF_BEEP);

  Beep(NOTE_1, HALF_BEEP);

  Beep(NOTE_2, ONE_BEEP*4);

  Beep(NOTE_5, ONE_BEEP);

  Beep(NOTE_3, HALF_BEEP);

  Beep(NOTE_5, HALF_BEEP);

  Beep(NOTE_1*2, HALF_BEEP*3);

  Beep(NOTE_7, HALF_BEEP);

  Beep(NOTE_6, ONE_BEEP);

  Beep(NOTE_1*2, ONE_BEEP);

  Beep(NOTE_5, ONE_BEEP*2);

  Beep(NOTE_5, ONE_BEEP);

  Beep(NOTE_2, HALF_BEEP);

  Beep(NOTE_3, HALF_BEEP);

  Beep(NOTE_4, HALF_BEEP*3);

  Beep(NOTE_7/2, HALF_BEEP);

  Beep(NOTE_1, ONE_BEEP*4);

  Beep(NOTE_6, ONE_BEEP);

  Beep(NOTE_1*2, ONE_BEEP);

  Beep(NOTE_1*2, ONE_BEEP*2);

  Beep(NOTE_7, ONE_BEEP);

  Beep(NOTE_6, HALF_BEEP);

  Beep(NOTE_7, HALF_BEEP);

  Beep(NOTE_1*2, ONE_BEEP*2);

  Beep(NOTE_6, HALF_BEEP);

  Beep(NOTE_7, HALF_BEEP);

  Beep(NOTE_1*2, HALF_BEEP);

  Beep(NOTE_6, HALF_BEEP);

  Beep(NOTE_6, HALF_BEEP);

  Beep(NOTE_5, HALF_BEEP);

  Beep(NOTE_3, HALF_BEEP);

  Beep(NOTE_1, HALF_BEEP);

  Beep(NOTE_2, ONE_BEEP*4);

  Beep(NOTE_5, ONE_BEEP);

  Beep(NOTE_3, HALF_BEEP);

  Beep(NOTE_5, HALF_BEEP);

  Beep(NOTE_1*2, HALF_BEEP*3);

  Beep(NOTE_7, HALF_BEEP);

  Beep(NOTE_6, ONE_BEEP);

  Beep(NOTE_1*2, ONE_BEEP);

  Beep(NOTE_5, ONE_BEEP*2);

  Beep(NOTE_5, ONE_BEEP);

  Beep(NOTE_2, HALF_BEEP);

  Beep(NOTE_3, HALF_BEEP);

  Beep(NOTE_4, HALF_BEEP*3);

  Beep(NOTE_7/2, HALF_BEEP);

  Beep(NOTE_1, ONE_BEEP*3);

  return 0;

}
```


# 一个处理 SVKP1.32 Stolen code 的小脚本，自学用

> 原文：[`www.pediy.com/kssd/pediy10/81310.html`](https://www.pediy.com/kssd/pediy10/81310.html)

我在看 `3800cc` 的教程，其中中级的第九篇脱的是 `SVKP 1.32` 的壳。看完教程后，一头雾水，最后是脱成功了，感觉最难的部分是找 `stolen code` 那里。所以这个小脚本就是按照教程把一些手工的东西变成自动，最后把 `stolen code` 那部分要分析的代码放在运行跟踪里面。这是第一次写的脚本，也算是一次学习吧。

```
//

// im only a beginner...

//

var addrGetModuleHandleA

GPA "GetModuleHandleA", "Kernel32.dll"

mov addrGetModuleHandleA, $RESULT

BP addrGetModuleHandleA + 9

RUN

BC addrGetModuleHandleA + 9

RTU

// 来到 IAT 特殊处理的地方

// 特征码：

// cmp dword ptr ds:\[ebx\],2D66B1C5

FIND eip, #813BC5B1662D#

mov \[$RESULT+6\], #eb5890909090#

mov \[$RESULT+7e\], #eb4c90909090#

// 特征码：

// mov dword ptr ds:\[edi\],eax

// popad

FIND eip, #890761#

mov \[$RESULT\], #618907#

bp $RESULT

RUN

bc $RESULT

BP addrGetModuleHandleA + 9

RUN

BC addrGetModuleHandleA + 9

RTU

// 下硬件断点在 12ffb0，断三次后取消开始 trace

BPHWS 12ffb0, "r"

RUN

RUN

RUN

BPHWC

TICND "ebp==12FFC0"

StO

// 开始处理特征码

bp 4052B6

TC

TOCND ""

// 脚本结束，可以在运行跟踪窗口中分析 stolen code

RET
```


# [原创]Code Virtualizer逆向工程浅析

论坛前最大字数限制，发不出去，只好将不紧要的部分略了，就这样吧。有意细究的，去拖未删减的TXT原文看。

**《Code Virtualizer逆向工程浅析》**

```
创建: 2022-06-30 16:56
更新: 2022-07-03 11:07
a59K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4U0P5W2)9J5k6e0j5I4y4#2)9J5k6h3y4F1i4K6y4m8z5q4)9J5c8X3#2A6M7$3y4Q4x3V1j5J5x3o6t1J5x3o6j5K6x3o6p5$3y4e0k6Q4x3X3g2@1P5s2b7
```

- 背景介绍
- Code Virtualizer 2.2.2.0
  - CV SDK
  - cvtest.c
  - 编译
  - CV虚拟化
- TTD
- CV虚拟机框架概览
  - 从`cv_entry[0]`到`cv_entry[4]`
  - 定位`cv_entry[1]`
  - 在`cv_entry[4]`处获取关键信息
  - 定位`func_array[]`
  - 确定`func_array[]`元素个数
  - 推断`VM_CONTEXT`结构
  - 从`VM_DATA`复制数据到`VM_CONTEXT`
  - 定位`cv_entry[5]`
- CV虚拟机逆向工程经验
  - `VM_CONTEXT.dispatch_data`
  - 跟踪`func_array[i]`
  - `VM_CONTEXT.efl`
  - `VM_CONTEXT.rsp`
  - `deref`操作
  - 库函数调用
  - 分析`func_array[i]`
    - 复杂`func_array[i]`的简单示例
  - 静态定位`func_array[i]`出口
    - 全连接图的非递归广度优先遍历
  - 寻找流程分叉点
  - 反向执行寻找`pushfq`
  - `cv_entry[3]`的函数化
- 后记

## 背景介绍

"Code Virtualizer"的资料不多，可能与它不如VMP被广泛使用有关，OSForensics 9用了CV。若非现实世界有实用软件用CV保护，鬼才有兴趣对之进行逆向工程。之前没有接触过CV，用TTD调试OSF时被绕得七荤八素，后来无意中确认OSF用CV保护。上网搜了些CV资料，都比较老，适用于1.3.8或更早期版本，与OSF所用CV版本差别较大。还有一点，老资料出现在32位时代，现在是64位时代。

CV将CFG扁平化，实际上没有调用栈回溯一说。CV处处是间接转移，主要是`jmp`寄存器这种形式，其次是将目标地址压入栈中，靠`ret`转移，这样一来，IDA中几乎没有显式交叉引用。敏感字符串是可以混淆存放的，这条路也断了。

本文分享一些CV逆向工程经验，基于网上能公开下到的CV 2.x。OSF所用CV是何版本，我不知道，但实测发现本文的经验大多也适用于OSF逆向工程。我只关心C语言，其他语言不在本文范畴。

## Code Virtualizer 2.2.2.0

### CV SDK

公开能下到的只有CV 2.x，以此为研究对象。压缩包展开后主要关注这些文件和目录

(略)

### cvtest.c

没有实际意义，只是示例，要点如下

```
#include "VirtualizerSDK.h"
/*
 * 链接时需要，下面那对宏会转换成对该库中函数的调用，用以占坑
 */
#pragma comment( lib, "VirtualizerSDK64.lib" )
/*
 * 将需要保护的代码片段置于这对宏中间
 */
VIRTUALIZER_EAGLE_BLACK_START
/*
 * 被保护代码片段位于两个宏中间
 */
...
VIRTUALIZER_EAGLE_BLACK_END
```

`EAGLE_BLACK`是CV虚拟机的一种，看上去保护强度最高。SDK自带的`vc_example`用的是`TIGER_WHITE`，保护强度很低。

### 编译

(略)

### CV虚拟化

执行`Virtualizer.exe`

(略)

缺省`LastSectionName`可能是其他值，比如`.vlizer`。`cvtest_p.exe`有2个`.scz`。

同一个`cvtest.exe`，每次CV虚拟化生成的`cvtest_p.exe`都不一样。

前面简介了CV SDK的使用，最好是自己整一个`cvtest.c`，生成`cvtest_p.exe`，对后者进行逆向工程，积累经验后再去对付现实世界的例子，比如OSF。

## TTD

CV虚拟化本身不会增加反调试检查，调试`cvtest_p.exe`时不需要反"反调试"。OSF有反调试，但OSF没有考虑到TTD技术的出现，其反调试措施没有针对TTD录制。

即便不考虑反"反调试"，对CV保护过的代码进行逆向工程时，条件允许的情况下，强烈建议TTD录制。若对CV有过经验积累，再动用TTD，能极大地抵消CV保护。

## CV虚拟机框架概览

### 从`cv_entry[0]`到`cv_entry[4]`

`VIRTUALIZER_EAGLE_BLACK_START`那一对宏在编译后化身为两个`call`

```
/*
 * VIRTUALIZER_EAGLE_BLACK_START
 */
000000014000108F FF 15 03 10 00 00           call    cs:VirtualizerSDK64_151
/*
 * 被保护代码片段位于两个call中间
 */
...
/*
 * VIRTUALIZER_EAGLE_BLACK_END
 */
0000000140001152 FF 15 48 0F 00 00           call    cs:VirtualizerSDK64_551
```

这是`cvtest.exe`中的效果，`cvtest.exe`只是从C编译成PE，尚未进行CV虚拟化处理。151、551这种数字无关紧要，要点是它们成对出现。

`Virtualizer.exe`靠这两个`call`识别出待保护代码片段，对之CV虚拟化，将11KB的`cvtest.exe`膨胀成1649KB的`cvtest_p.exe`，这是加了多少垃圾代码？

CV虚拟化时将`call VirtualizerSDK64_151`就地转成`jmp`，这是`cvtest_p.exe`中的效果

```
/*
 * VIRTUALIZER_EAGLE_BLACK_START
 *
 * 为叙述方便，此处定义成cv_entry[0]
 */
000000014000108F E9 32 BB 19 00              jmp     cv_entry_1_14019CBC6
/*
 * 中间的字节流是啥我也不知道，反正不是原来的代码
 */
...
/*
 * VIRTUALIZER_EAGLE_BLACK_END
 *
 * 将"call VirtualizerSDK64_551"就地转成类似nop的填充指令，模式不固定
 *
 * 为叙述方便，此处定义成cv_exit[0]
 */
0000000140001152 88 C9                       mov     cl, cl
0000000140001154 88 C9                       mov     cl, cl
0000000140001156 88 C9                       mov     cl, cl
```

`cv_entry[0]`还在`.text`中，但`jmp`的目标地址`cv_entry[1]`已离开`.text`，进入`.scz`。

```
000000014019CBC6                         cv_entry_1_14019CBC6
/*
 * 为叙述方便，此处定义成cv_entry[1]
 */
000000014019CBC6 9C                          pushfq
...
/*
 * 从pushfq到jmp，无任何分支转移指令，二者就是块首、块尾
 *
 * 为叙述方便，此处定义成cv_entry[2]
 */
000000014019CD1C E9 18 DD FE FF              jmp     cv_entry_3_14018AA39
```

`cv_entry[1]`的特点是`pushfq`，`cv_entry[1]`、`cv_entry[2]`之间无任何分支转移指令，二者就是块首、块尾，在IDA中用图形模式查看，非常明显，这是第二个特点。

```
/*
 * 位于第一个".scz"中，Ctrl-S确认
 */
000000014018AA39                         cv_entry_3_14018AA39
/*
 * 用到自定位技巧，shellcode常用套路
 *
 * 为叙述方便，此处定义成cv_entry[3]
 */
000000014018AA39 E8 00 00 00 00              call    $+5
...
/*
 * 为叙述方便，此处定义成cv_entry[4]
 */
000000014018BE28 FF 20                       jmp     qword ptr [rax]
```

`cv_entry[3]`的特点是`call $+5`，一种自定位技巧，`shellcode`常用套路。在IDA中`Alt-B`搜索字节流`E8 00 00 00 00`，找出所有`call $+5`，基本上都是`cv_entry[3]`。

`cv_entry[4]`的特点是`jmp [rax]`。

即使在C代码中只使用了一对`VIRTUALIZER_START`/`VIRTUALIZER_END`，`cvtest_p.exe`仍有可能出现多个`cv_entry[3]`，为什么？因为只要进入CV虚拟机一次，就会有一个`cv_entry[3]`等着经过，从CV虚拟机中调用外部库函数时，会临时离开CV虚拟机，执行完外部库函数，重新回到CV虚拟机。在这些进出CV虚拟机过程中，自然出现多个`cv_entry[3]`，有些进出流程可能共用一个`cv_entry[3]`，有些可能用自己的`cv_entry[3]`。

`cv_entry_3_14018AA39`可以`p`操作成函数，图形化查看时非常复杂，但把握住前述入口与出口特点，搞几次后就能轻松定位。

IDA可能缺省未将`cv_entry[1]`与`cv_entry[3]`识别成函数，我的事后复盘经验是，一定将它们`p`成函数，以降低静态分析难度，IDA的图形模式只能看函数。

CV虚拟机官方没有`cv_entry[0]`、`cv_entry[4]`这些概念，这是为了叙述方便自己给的定义。回顾一下流程框架

```
/*
 * cv_entry[0]
 */
jmp     cv_entry[1]
-------------------
/*
 * cv_entry[1]
 */
pushfq
...
/*
 * cv_entry[2]
 */
jmp     cv_entry[3]
-------------------
/*
 * cv_entry[3]
 */
call    $+5
...
/*
 * cv_entry[4]
 */
jmp     [rax]
```

逻辑上`cv_entry[0]`在CV虚拟机外，一般在`.text`中，这个不绝对。之后`cv_entry[1]`至`cv_entry[4]`全部在CV虚拟机中，一般在`.vlizer`中。

### 定位`cv_entry[1]`

已知从`cv_entry[0]`转向`cv_entry[1]`，会从`.text`转向`.scz`（本例中的名字），可以用`x64dbg`调试，对`.scz`设内存访问断点，以此快速定位`cv_entry[1]`。这段话假设目标程序比较复杂，现在还在`.text`中，静态分析一时半会儿找不到`cv_entry[0]`。若肉眼就能发现`cv_entry[0]`，则无需前述技巧。

定位`cv_entry[1]`之后，静态分析就能定位`cv_entry[2]`到`cv_entry[4]`。

### 在`cv_entry[4]`处获取关键信息

假设调试器停在`cv_entry[4]`

```
rax=000000014018120e rbx=0000000000000360 rcx=0000000140000000
rdx=0000000000180eae rsi=5555555555555555 rdi=6666666666666666
rip=000000014018be28 rsp=000000000014fe08 rbp=00000001400ab9d6
 r8=8888888888888888  r9=9999999999999999 r10=aaaaaaaaaaaaaaaa
r11=bbbbbbbbbbbbbbbb r12=cccccccccccccccc r13=dddddddddddddddd
r14=eeeeeeeeeeeeeeee r15=ffffffffffffffff
iopl=0         nv up ei pl nz na pe nc
cs=0033  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00000202
```

在`cv_entry[4]`处有

```
rsp 0x14fe08    VM_DATA
rbp 0x1400ab9d6 VM_CONTEXT
rax 0x14018120e &func_array[0x6c]
rbx 0x360       0x360/8=0x6c
rcx 0x140000000 ImageBase
rdx 0x180eae    VM_CONTEXT.func_array 0x140180eae
```

在`cv_entry[4]`处可以找到`VM_CONTEXT`，这是CV虚拟机的核心组件之一，后面再说。

(略)

从`cv_entry[0]`到`cv_entry[4]`真正干的大事就是将`cv_entry[0]`处各寄存器压栈，一堆眼花缭乱的操作都是为了掩盖这个事实。最初我还老老实实在TTD调试中一步步跟，后来意识到它的意图后，采用污点追踪的思想快速定位`cv_entry[4]`处栈中诸数据。

前文所用术语都是自己瞎写的，结合上下文对得上就成。

### 定位`func_array[]`

`func_array[]`就是老资料里说的`handler[]`，CV虚拟化将每一条位于保护区的汇编指令转换成许多个`func_array[i]`组合。

在`cv_entry[4]`处有多种办法定位`func_array[]`，比如

```
> ? @rcx+@rdx
> ? @rax-qwo(@rsp+0x88)*8
Evaluate expression: 5370285742 = 00000001`40180eae
```

`0x140180eae`即`func_array[]`起始地址。

有个取巧的办法定位`func_array[]`起始地址。假设已知`VM_CONTEXT`在`0x1400ab9d6`，本例中该结构占`0x174`字节，但该结构大小并不固定，有可能是其他大小。在IDA中查看`0x1400ab9d6`处hexdump，大片的0，只有一处非零，就是`VM_CONTEXT.func_array`字段所在，静态查看时该值是重定位前的偏移值，加上基址才是内存地址。

IDA中看`func_array[i]`，是重定位之前的偏移值，加上`ImageBase`才是函数地址。应在IDA中静态Patch，人工完成重定位，使得IDA分析出更多代码。`func_array[]`比较大，很可能没有以`qword`形式展现，一个一个手工加基址Patch不现实，写`IDAPython`脚本完成。

### 确定`func_array[]`元素个数

没有简单办法确定`func_array[]`元素个数。在IDA中肉眼识别、逐步逼近当然可以，但不够放心，怕不精确。

有个辅助办法，图形化查看`cv_entry[4]`，往低址方向找如下`cmp`、`test`指令，还是比较容易定位的。

```
/*
 * rax=0x140180eae VM_CONTEXT.func_array+ImageBase
 * rdx=0x180eae VM_CONTEXT.func_array
 */
000000014018B9D4 48 39 C2                    cmp     rdx, rax
000000014018B9D7 0F 84 72 03 00 00           jz      loc_14018BD4F

/*
 * rbx=0x97 func_array[]元素个数
 */
000000014018BABD 48 85 DB                    test    rbx, rbx
000000014018BAC0 0F 84 7D 01 00 00           jz      loc_14018BC43
```

找到`0x14018B9D4`、`0x14018BABD`这两个地址后，在TTD调试中对之设断点，从`cv_entry[3]`处正向执行，断点命中时查看寄存器，注释中写了。不一定TTD调试，普通调试就可以，但我一上来就TTD录制了，后面的分析都是在反复鞭尸，更方便。

精确知道`func_array[]`元素个数后，写`IDAPython`脚本对之批量`qword`化、加基址。这还不够，应该对每个`func_array[i]`加`repeatable FUNCTION comment`，比如这种效果

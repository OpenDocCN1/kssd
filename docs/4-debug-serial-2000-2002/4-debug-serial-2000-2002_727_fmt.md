
# 雅信 2.5

> 原文：[`www.pediy.com/kssd/pediy03/forum661.htm`](https://www.pediy.com/kssd/pediy03/forum661.htm)

`http://brucez.coolbi.com/soft/YaxinCat25.rar`

大客们我上载的雅典最新缩小版！只有 1.25mb，功能当然不全了！

不过可以注册！我小弟试破解怀疑 `bpx 004c724e`，下面有一个 `call` 可能是关键！原来我离功能只差这么一步！都是 `win98` 不好！

## 破解入门 5

**gave bye 雅信 2.5!**

**作者:** `PeterChen`

一定要解决问题才对,不是为了要注册码!

这是本人破解过程出分析的最详细了!

破解步骤有两种:

### 第一种原始方法

1. `bpx hmemcpy`
2. `F12` 14 次
3. `F10` 26 次

### 第二种

1. `bpx hmemcpy`
2. `pmodule`
3. `F10` 40 几次有多!

想不到原始的方法比新方法还要快!

还有一种方法等一会儿再说!

好不好容易才来到这里?

.........................................................................

```
:004C71E4 8D4DEC                   lea ecx, dword ptr [ebp-14] //->d eax=ID
:004C71E7 A1C4E84F00               mov eax, dword ptr [004FE8C4]
:004C71EC 8B00                     mov eax, dword ptr [eax]
:004C71EE 8BD3                     mov edx, ebx
:004C71F0 E88BFEFFFF               call 004C7080
:004C71F5 8B45EC                   mov eax, dword ptr [ebp-14]
:004C71F8 50                       push eax //->才是真出现注册码的地方!
:004C71F9 8D55E4                   lea edx, dword ptr [ebp-1C]
:004C71FC 8B86D8020000             mov eax, dword ptr [esi+000002D8]
:004C7202 E89DB5F6FF               call 004327A4
:004C7207 8B45E4                   mov eax, dword ptr [ebp-1C]
:004C720A 8D55E8                   lea edx, dword ptr [ebp-18]//->d eax=假注册码
:004C720D E81627F4FF               call 00409928
:004C7212 8B55E8                   mov edx, dword ptr [ebp-18]
:004C7215 58                       pop eax
:004C7216 E8A9CEF3FF               call 004040C4
:004C721B 7517                     jne 004C7234

:004C721D 8D55E0                   lea edx, dword ptr [ebp-20]
:004C7220 8BC3                     mov eax, ebx
:004C7222 E8E929F4FF               call 00409C10
:004C7227 8B45E0                   mov eax, dword ptr [ebp-20]
:004C722A 8D55F8                   lea edx, dword ptr [ebp-08]
:004C722D E8F626F4FF               call 00409928
:004C7232 EB06                     jmp 004C723A

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C721B(C)
|

:004C7234 43                       inc ebx
:004C7235 83FB0A                   cmp ebx, 0000000A
:004C7238 75AA                     jne 004C71E4 //->循环 9 次?
..................................................................
```

为什么？老是才这里来来回回了?

开始分析一下:

`004C7238 75AA jne 004C71E4` -> `:004C71E4 8D4DEC lea ecx, dword ptr [ebp-14]`

看到了吗？`jne 004c71e4` 见鬼去为啥这样了? 究竟要反复多少次了? 不继的按 `F10` 不要怕手软啊?

***

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C7232(U)
|

:004C723A 8B45F8                   mov eax, dword ptr [ebp-08] //-> F10 第 9 次来到这里!

* Possible StringData Ref from Code Obj ->"100"
                                  |

:004C723D BAD4734C00               mov edx, 004C73D4
:004C7242 E87DCEF3FF               call 004040C4
:004C7247 750F                     jne 004C7258

* Possible StringData Ref from Code Obj ->"您输入的授权号不正确，请检查输入（要确保提供给"
                                 ->"本公司的序列号正确）！"
                                  |

:004C7249 B8E0734C00               mov eax, 004C73E0
:004C724E E82D2DF9FF               call 00459F80//->game over!
:004C7253 E9F6000000               jmp 004C734E
```

为什么这样的了?再分析一下!

`:004C7238 75AA  jne 004C71E4` (jump)第 8 次还有黄色"jump"
第 9 次就变成 (not jump)

这里一定有问题?

```
:004C71E4 8D4DEC                   lea ecx, dword ptr [ebp-14] //->d eax=ID
:004C71E7 A1C4E84F00               mov eax, dword ptr [004FE8C4]
:004C71EC 8B00                     mov eax, dword ptr [eax]
:004C71EE 8BD3                     mov edx, ebx
:004C71F0 E88BFEFFFF               call 004C7080
:004C71F5 8B45EC                   mov eax, dword ptr [ebp-14]
:004C71F8 50                       push eax //->才是真出现注册码的地方!
```

为使初学者容易明白,一次把它们说出来!

假设有一个数值 123,把 `[ebp-14]` 中 123 送到 `ecx` 中.再把 `[004FE8C4]` 中 123 转到 `eax` 中然后把 `[eax]` 的 123 再转到 `eax` 中! `mov eax, dword ptr [ebp-14]` 同理!

123 是假设不是相等的! 最后输入 123! `push eax` 是输入在破解中有 `push` 跟寄存器多数是注册码或假码!

`D eax` 出现数码不是我的假码是不是注册码? 试一试! 果然是个人版注册成功!

注册码在这里到了哪为什么要循环了? 再分析一下,

```
:004C71E4 8D4DEC  lea ecx, dword ptr [ebp-14] //->d eax=ID
:004C71F8 50      push eax //->才是真出现注册码的地方!
:004C720A 8D55E8  lea edx, dword ptr [ebp-18]//->d eax=假注册码
```

现在知道了注册原理了?

```
:004C71E4 8D4DEC                   lea ecx, dword ptr [ebp-14] //->d eax=ID
:004C71E7 A1C4E84F00               mov eax, dword ptr [004FE8C4]
:004C71EC 8B00                     mov eax, dword ptr [eax]
:004C71EE 8BD3                     mov edx, ebx
:004C71F0 E88BFEFFFF               call 004C7080
:004C71F5 8B45EC                   mov eax, dword ptr [ebp-14]
:004C71F8 50                       push eax //->才是真出现注册码的地方!
:004C720A 8D55E8  lea edx, dword ptr [ebp-18]//->d eax=假注册码
```

把 id 直接换算后输出真注册码才与你输入的注册码进行比较!

`:004C71F8 50      push eax //->才是真出现注册码的地方!`

为什么我这么破解定这里才是注册码了?

这里不但有 `eax=注册码 edx=假码`!现在我明白为什么在破解过程中要看寄存器变化了?

就如:

```
:004C71E7 A1C4E84F00               mov eax, dword ptr [004FE8C4] ss:[]=?
eax=?
:004C71EC 8B00                     mov eax, dword ptr [eax]
```

当然我们看到上面有 `ss:[]=?`在下面 `eax` 发光 `d` 一下就是"?"了 `ds:[]` 也同理!

为什么要这样看了? 因为这个看能知道两句汇编是否有相同的寄存器值?(寄存器发光才要看)

而出现相等的值多数是 id 假注册码或正注册码!

还有:`:004C71F8 50  push eax //->才是真出现注册码的地方!`

`Ds:[79ee48]=01cbe4ec` 与 `Stack Prot 32` 窗口的 `:0079ee48 >01cbe4ec` 相等! 原来这个窗口也有一点用?

我发现在真注册码时才有相等的值!

到上技巧对找到关键才有用?

`:004C71F8 50  push eax` (9 次来出这里注册码都不同)

昨晚有一位大客说在 `:004C7216 E8A9CEF3FF call 004040C4`

在这里下 `d eax` 就找到注册码! 已经我多次试验在 `:004C71F8 50  push eax`

"`eax=`"的值都发生变化来到大客所说时它的是相等的,所以真正计算出注册码是 `push eax` 才对!

说一下最快的方法就是借助 `W32dsm89` 反译后找到出现错误窗的 `call`!

```
:004C724E E82D2DF9FF               call 00459F80 向上分析!
:004C7247 750F                     jne 004C7258
:004C7238 75AA                     jne 004C71E4
```

为什么找不到跳不过些 `call` 汇编了? 好分析一下两个跳过的命!一个是向下跳,另一个是向上跳!

程序执行有须序的都是从上到下才对! 所以第一个执行是 `jne 004C71E4->004C71E4 8D4DEC`

分析一个就知道有循环了? 就要着重分析最后就能 `kill it`!这样是不是更快了,但是要有一定的汇编

功力才行! 如何要暴力就要分析一个不些的! 这个软件比较奇怪以前暴力不跳多数在不远的地方而且

这个了看来要暴力比破解注册码才要麻烦了! 不过试暴力过但是没有成功过? 它不要其它的破解一样

的错误提示,而是能告诉你哪里有修改过? 看来应该是本身有监察功能吧？也许不能暴力或者我暴力

功力不足!

最后发现这个软件不是想像中的很简单!

你再破解时 `eax` 的值 `ds:[]`、`ss:[]`都变化也 `id`、`假注册码`、`code` 所出现的地方也变了！

就算好像游戏一样用动态方式！

### 最后整理

我的 ID:1572546

sn: 19003697
 21261621
 18577266
 15892912
 13208557
 10524203
 78398485
 51554940
 24711394

9 个注册码？

其中有鬼？

123 分别是个版、专业版、网络版
456 分别是个未注册版、专未注册业版、网络未注册版
789 分别又是是个版、专业版、网络版

在注册表找到:

```
REGEDIT4
[HKEY_CURRENT_USER\yaxincat20cec]
[HKEY_CURRENT_USER\yaxincat20cec\addins]
"func1"=""
"func2"=""
"func3"=""
"func4"=""
"func5"=""
"func6"=""
"func7"=""
"func8"=""
"func9"=""
```

就是这看到有 9 个注册码!

删除又能注册了!

有一点不明白 `id` 是如何出现的了，要想修改一个这样就有自己的

`id` 了可惜不在注册表和 `*.ini` 中请如何改它的 `id` 了？

### 注册机

```c
#include "stdio.h"

main()
{
    char string[81];
    long real[81];
    long i;

    printf("******YaXin CAT 2.5******\nThis keygen is maked by Pretender\nPlease input your ID  ");
    gets(string);

    if(string[7]==57) {string[7]=48;string[6]=string[6]+1;}
    else string[7]+=1;

    real[0]=(string[7]-48)*268435456;
    real[1]=(string[0]-48)*16777216;
    real[2]=(string[6]-48)*1048576;
    real[3]=(string[1]-48)*65536;
    real[4]=(string[5]-48)*4096;
    real[5]=(string[2]-48)*256;
    real[6]=(string[4]-48)*16;
    real[7]=string[3]-48;

    printf("Your Register code is ");
}
```



```
i=real\[0\]+real\[1\]+real\[2\]+real\[3\]+real\[4\]+real\[5\]+real\[6\]+real\[7\];   

if(i>99999999) i/=100;   

 else if(i>9999999) i/=10;   

printf("%ld\\n",i);   

printf("Welcome to WWW.CRACKNOW.COM\\n");}
```

问题~我认为这个东西每个 `ID` 有 10 个注册码。

虽然我全都可以找到，但是我不知道它们的作用都是什么~`

谁能告诉我？

`4dd466d0e1e0cc123afbe504da2462ca`

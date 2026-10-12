
# TraceMe 堆栈查看分析

> 原文：[`www.pediy.com/kssd/pediy11/118022.html`](https://www.pediy.com/kssd/pediy11/118022.html)

**【文章作者】**: Gall  
**【作者主页】**: `http://hi.baidu.com/8ohack`  
**【使用工具】**: `Ollydbg`  
**【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 详细过程

堆栈查看分析，非常常用的方法，很多时候这个方法就是捷径.  

废话不多说了,看下面的分析吧！  

`Ollydbg` 载入程序 `TraceMe.exe`.`F9` 运行程序，输入下面的信息  

【用户名】：`Gall_pediy`  

【序列号】：`1234567`  

点击 `Check` 按钮,然后一 `Ollydbg` 为兑现按下 `F12`,接着再按下 `Alt+K` 查看堆栈的信息.  

```
Call stack of main thread

Address    Stack      Procedure / arguments                            Called from                     Frame

0012F8C0   77D19418   Includes ntdll.KiFastSystemCallRet               USER32.77D19416                 0012F8F4
0012F8C4   77D2770A   USER32.WaitMessage                               USER32.77D27705                 0012F8F4
0012F8F8   77D249C4   USER32.77D2757B                                  USER32.77D249BF                 0012F8F4
0012F920   77D24A06   USER32.77D2490E                                  USER32.77D24A01                 0012F91C
0012F940   77D3B190   USER32.DialogBoxIndirectParamAorW                USER32.77D3B18B                 0012F93C
0012F96C   00401287   USER32.DialogBoxParamA                           TraceMe.00401281                0012F968
0012F970   00400000     hInst = 00400000
0012F974   00000079     pTemplate = 79
0012F978   0003092C     hOwner = 0003092C ('TraceMe 动态分析技术',class=
0012F97C   00401060     DlgProc = TraceMe.00401060
0012F980   00000000     lParam = NULL
0012FA84   77D18734   Includes TraceMe.00401287                        USER32.77D18731                 0012FAAC
0012FAB0   77D23CE4   ? USER32.77D1870C                                 USER32.77D23CDF                 0012FAAC
0012FB1C   77D23B30   ? USER32.77D23C2F                                 USER32.77D23B2B                 0012FB18
0012FB64   77D3E599   USER32.77D23AB1                                    USER32.77D3E594                 0012FB60
0012FB80   77D18734   Includes USER32.77D3E599                          USER32.77D18731                 0012FB7C
0012FBAC   77D18816   ? USER32.77D1870C                                 USER32.77D18811                 0012FBA8
0012FBB0   77D3E577   Includes USER32.77D18816                          USER32.77D3E571                 0012FC10
```

记住我们调试的是程序，而不是系统文件.注意看 `Stack` 对应的数据,加以区分，在这里我以 `00401287` 为目标

Call stack of main thread, item 5

```
Address=0012F96C
Stack=00401287
Procedure / arguments=USER32.DialogBoxParamA
Called from=TraceMe.00401281
Frame=0012F968
```

右键 `Show Call`,显示如下:

```
00401276   .  6A 00         push    0                                      ; /lParam = NULL
00401278   .  68 60104000  push    00401060                                 ; |DlgProc = TraceMe.00401060
0040127D   .  56            push    esi                                    ; |hOwner
0040127E   .  6A 79         push    79                                       ; |pTemplate = 79
00401280   .  51            push    ecx                                    ; |hInst => 00400000
00401281   .  FF15 C8404000 call    dword ptr [<&USER32.DialogBoxPar>; \【Show Call 后来到这里】
00401287   .  5B            pop     ebx
00401288   .  5F            pop     edi
00401289   .  33C0          xor     eax, eax
0040128B   .  5E            pop     esi
```

滚动查看一下,打击就会发现,上面就是关键信息

```
004011E4   .  50            push    eax
004011E5   .  E8 56010000  call    00401340                                 ;  关键位置[算法过程]
004011EA   .  8B3D BC404000 mov     edi, dword ptr [<&USER32.GetDlgI>;  USER32.GetDlgItem
004011F0   .  83C4 0C       add     esp, 0C
004011F3   .  85C0          test    eax, eax
004011F5   .  74 37         je      short 0040122E                           ;  关键跳转
004011F7   .  8D4C24 0C     lea     ecx, dword ptr [esp+C]
004011FB   .  51            push    ecx                                    ; /String2
004011FC   .  68 E4544000  push    004054E4                                 ; |String1 = TraceMe.004054E4
00401201   .  FF15 60404000 call    dword ptr [<&KERNEL32.lstrcpyA>] ; \lstrcpyA
```

有人肯定要问，你怎么知道关键信息就在上面呢？

不错!查看堆栈的时候,大家难道没有疑问吗？为什么没有 `MessageBoxA`【或 `MessageBoxW`】函数呢?

明明有提示框出现.再次说明一下,不是所有的注册错误提示都是通过函数 `MessageBoxA`【或 `MessageboxW`】来完成的,在本例中的错误信息窗口就是函数 `DialogBoxParamA` 建立的.

大家如果不信，可以利用快捷键 `Ctrl+N` 查看一下是否有函数 `MessageBoxA`【或 `MessageBoxW`】.

既然知道错误提示窗口是由函数 `DialogBoxParamA` 建立的，那么就不难想到,肯定是注册错误后跳到这里的。所以就需要对函数 `DialogBoxParamA` 前面的代码进行分析了

---

## 版权声明

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2010 年 08 月 05 日 15:41:25

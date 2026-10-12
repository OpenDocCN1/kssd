
# 获取 QQ2008 密码 附测试源码

> 原文：[`www.pediy.com/kssd/pediy10/100885.html`](https://www.pediy.com/kssd/pediy10/100885.html)

本人小菜一个，呵呵 属于是瞎子逆向，不过还是分析出了一些东西，本着学习研究的态度下面就给大家分享一下。

首先 QQ 密码是采用了 xxxx 先进的键盘加密技术，所以如果再从键盘截取这条路截取密码显然是比较难的了，而且成本也比较高，所以我们要从另外一条路下手。在 QQ 协议里面，QQ 的密码是先 MD5 然后 BASE 编码 然后再发送到服务器验证的。所以这里就是关键了，无论那个先进的键盘加密怎样加密，他最终还是要还原成原来的密码然后再进行 MD5+BASE 的编码，如果我们能在 MD5 加密之前截获他，嘿嘿~~~这里就留给了我们获取密码的机会![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif")，我的思路大概就是这样的了，下面就是具体的实现过程

我们首先要定位到 MD5 加密代码的附近，如何定位呢？这样用 PEID 里面的插件 Krypto ANALyzer 比较方便（当然也可以自己找），首先到目录里面看看有没有什么可疑的文件，LoginCtrl.dll 一看就应该是目标了，然后查到了 MD5 加密代码在 0x608C5F87 附近，然后直接把 QQ 扔 OD 里面到 0x608C5F87 去看看（呵呵，QQ 无壳无反调试 还真轻松）

```
608C5F84   |.  8D943A 56B7C7E8    |lea     edx, dword ptr [edx+edi+E8C7B75>
608C5F8B   |.  8BFA              |mov     edi, edx
..................(很长的加密函数)
```

然后在函数末尾下断，先跟一下 乱输个密码登陆 QQ 就断下了

返回来到这里

```
608C6897   |.  8B45 08           mov     eax, dword ptr [ebp+8]
608C689A   |.  8B0E             mov     ecx, dword ptr [esi]
。。。。。。。。（又是一个长长的加密函数）
```

然后再跟到返回 来到了 608B6D1E 很明显，608b6d19 应该就是 MD5 的 call 了，这个 call 有两个参数，一个应该是加密的字符，另外一个应该就是加密后保存的缓冲区；

```
608B6D15   |>  \53               |push    ebx
608B6D16   |.  FF75 DC           |push    dword ptr [ebp-24]
608B6D19   |.  E8 D9FA0000       |call    608C67F7
608B6D1E   |.  6A 5C            |push    5C                           ; /n = 5C (92.)
608B6D20   |.  6A 00            |push    0                            ; |c = 00
608B6D22   |.  53              |push    ebx                           ; |s
608B6D23   |.  E8 9EE90000      |call    <jmp.&MSVCRT.memset>         ; \memset
```

这里想都想得到，为了密码的安全 memset 应该就是清除密码

密码地址的内容如下，很奇怪 uczvhcncfbsgdbms 不是我输入的密码，先不管这个，继续跟下去

```
01195528  77 99 93 EF 1F 3D 18 E4 95 51 1D FC DC CF 4B F0  w?=Q
01195538  80 00 00 00 00 00 00 00 75 63 7A 76 68 63 6E 63  .......uczvhcnc
01195548  66 62 73 67 64 62 6D 73 80 00 00 00 00 00 00 00  fbsgdbms.......
```

再跟下去就到了下一轮循环了整个循环内容如下。

```
608B6C85   |> /8B45 CC           /mov     eax, dword ptr [ebp-34]
608B6C88   |. |8B4D E8           |mov     ecx, dword ptr [ebp-18]
608B6C8B   |. |03C1             |add     eax, ecx
608B6C8D   |. |99               |cdq
608B6C8E   |. |F7FE             |idiv    esi
608B6C90   |. |837D D4 00        |cmp     dword ptr [ebp-2C], 0
608B6C94   |. |8BFA             |mov     edi, edx
608B6C96   |. |75 35            |jnz     short 608B6CCD
608B6C98   |. |68 ECBF8D60       |push    608DBFEC
608B6C9D   |. |8D4D D8          |lea     ecx, dword ptr [ebp-28]
608B6CA0   |. |E8 01E40000      |call    <jmp.&MFC42.#CString::CString_5>
608B6CA5   |. |8B45 D8          |mov     eax, dword ptr [ebp-28]
608B6CA8   |. |8365 FC 00       |and     dword ptr [ebp-4], 0
608B6CAC   |. |C1E7 04          |shl     edi, 4
608B6CAF   |. |FF70 F8          |push    dword ptr [eax-8]
608B6CB2   |. |037D E4          |add     edi, dword ptr [ebp-1C]
608B6CB5   |. |50              |push    eax
608B6CB6   |. |57              |push    edi
608B6CB7   |. |E8 58FC0000     |call    608C6914
608B6CBC   |. |834D FC FF       |or      dword ptr [ebp-4], FFFFFFFF
608B6CC0   |. |83C4 0C          |add     esp, 0C
608B6CC3   |. |8D4D D8          |lea     ecx, dword ptr [ebp-28]
608B6CC6   |. |E8 23E10000      |call    <jmp.&MFC42.#CString::~CString_>
608B6CCB   |. |EB 5E            |jmp     short 608B6D2B
608B6CCD   |> |8BC7             |mov     eax, edi
608B6CCF   |. |8B4D EC          |mov     ecx, dword ptr [ebp-14]
608B6CD2   |. |6BC0 5C          |imul    eax, eax, 5C
608B6CD5   |. |8D1C08           |lea     ebx, dword ptr [eax+ecx]
608B6CD8   |. |53              |push    ebx
608B6CD9   |. |E8 87EF0000     |call    608C5C65
608B6CDE   |. |8B45 D4          |mov     eax, dword ptr [ebp-2C]
608B6CE1   |. |59              |pop     ecx
608B6CE2   |. |85C0            |test    eax, eax
608B6CE4   |. |7E 2F            |jle     short 608B6D15
608B6CE6   |. |8365 E0 00       |and     dword ptr [ebp-20], 0
608B6CEA   |. |8945 D0          |mov     dword ptr [ebp-30], eax
608B6CED   |> |8B45 C8          |/mov     eax, dword ptr [ebp-38]
608B6CF0   |. |6A 01           ||push    1
608B6CF2   |. |8B40 44          ||mov     eax, dword ptr [eax+44]
608B6CF5   |. |8B00             ||mov     eax, dword ptr [eax]
608B6CF7   |. |0345 E0          ||add     eax, dword ptr [ebp-20]
608B6CFA   |. |8A0438           ||mov     al, byte ptr [eax+edi]
608B6CFD   |. |8845 F3          ||mov     byte ptr [ebp-D], al
608B6D00   |. |8D45 F3          ||lea     eax, dword ptr [ebp-D]
608B6D03   |. |50              ||push    eax
608B6D04   |. |53              ||push    ebx
608B6D05   |. |E8 99EF0000     ||call    608C5CA3
608B6D0A   |. |0175 E0          ||add     dword ptr [ebp-20], esi
608B6D0D   |. |83C4 0C          ||add     esp, 0C
608B6D10   |. |FF4D D0          ||dec     dword ptr [ebp-30]
608B6D13   |.^|75 D8            |\jnz     short 608B6CED
608B6D15   |> |53              |push    ebx
608B6D16   |. |FF75 DC          |push    dword ptr [ebp-24]
608B6D19   |. |E8 D9FA0000      |call    608C67F7
608B6D1E   |. |6A 5C            |push    5C                           ; /n = 5C (92.)
608B6D20   |. |6A 00            |push    0                            ; |c = 00
608B6D22   |. |53              |push    ebx                           ; |s
608B6D23   |. |E8 9EE90000      |call    <jmp.&MSVCRT.memset>         ; \memset
608B6D28   |. |83C4 14          |add     esp, 14
608B6D2B   |> |FF45 E8          |inc     dword ptr [ebp-18]
608B6D2E   |. |8345 DC 10       |add     dword ptr [ebp-24], 10
608B6D32   |. |3975 E8          |cmp     dword ptr [ebp-18], esi
608B6D35   |.^\\0F8C 4AFFFFFF    \\jl      608B6C85
```

很奇怪，不知道为什么密码要计算这么多次，（很可能是混有假密码在里面），在 memset 这里下断 多跟踪几次，最终在 memset 的参数的指令里面发现了输入的密码（输入的密码是 N 个 d），欣喜中......

```
01195750  F8 D9 02 4E 1E FC 5C 82 B0 39 69 4D 66 70 42 64  N9iMfpBd
01195760  80 00 00 00 00 00 00 00 64 64 64 64 64 64 64 64  .......dddddddd
01195770  64 64 64 64 64 64 64 64 80 00 00 00 00 00 00 00  ddddddd...
```

这里是密码的 HASH

```
01195818  F8 D9 02 4E 1E FC 5C 82 B0 39 69 4D 66 70 42 64  N9iMfpBd
```

接下来的问题是密码是在其中找到了，关键是我们怎么才知道哪个才是正确的密码呢？继续跟...

跳出循环后来到了这里

```
608B6D3B   |> \\2B75 CC         sub     esi, dword ptr [ebp-34]
608B6D3E   |.  6A 10            push    10
608B6D40   |.  8D4D DC         lea     ecx, dword ptr [ebp-24]
608B6D43   |.  4E             dec     esi
608B6D44   |.  C1E6 04         shl     esi, 4
608B6D47   |.  0375 E4         add     esi, dword ptr [ebp-1C]
608B6D4A   |.  56             push    esi
608B6D4B   |.  E8 12E50000    call    <jmp.&MFC42.#CString::CString_538>
608B6D50   |.  8D45 DC         lea     eax, dword ptr [ebp-24]
608B6D53   |.  C745 FC 01000000 mov    dword ptr [ebp-4], 1
608B6D5A   |.  50             push    eax
608B6D5B   |.  FF15 3CC08C60  call    dword ptr [<&BasicCtrlDll.Encode16>]  ;  BasicCtr.Encode16
```

这里看到将十六个字节转换字符串 应该就是密码的 HASH 了，然后下面的 BasicCtrlDll.Encode16 再编码，跟到 CString 看看堆栈

```
0012EFA4   01195818
0012EFA8   00000010
```

这里的 01195818 就是刚刚 MD5 那里算出来的 hash。

到这里，整个密码处理的流程差不多就清楚了，我们就可以根据这个拦截密码了，首先在 HOOK memset 的这个 call 然后获取密码和密码的 hash，然后再 HOOK 后面的 Cstring 根据 HASH 找出真的密码。这里有个问题是由于 QQ2008 的版本众多所以还要利用特征码来动态定位，下面就是实现的效果，和测试代码

(由于本人小菜一个，逆向功夫不到家，很多东西都吃不准，如果有什么失误，不足的还望各位高手指出，让我等小菜学习学习)

代码:

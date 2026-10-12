
# Calendar Builder 3.45

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-408.htm`](https://www.pediy.com/kssd/pediy08/pediy8-408.htm)

## 非常简单的 Calendar Builder 3.45 注册算法

**作者**：XXNB

**邮箱**：

**主页**：[`free.ys168.com/?binbinbin7456`](http://free.ys168.com/?binbinbin7456)

**工具**：`OD`

**平台**：`xpsp2`

**软件名称**：`Calendar Builder 3.45`

**大小**：1727KB

**下载**：[`www.onlinedown.net/soft/3467.htm`](http://www.onlinedown.net/soft/3467.htm)

**保护方式**：名＋码

**简介**：非常棒的日历制作软件，程序内建多种的日历模式样板，如每月、每年、每周等等，它支持中文字形，可加入文字及喜欢的图片，更可将每个节日放上喜欢的 ICON 来表示，相当不错，设计完并可将它打出来使用。

**声明**：向大侠们学习！！！

## 破解过程

献给像我一样想学算法的同志们

### 1、字符串查找

字符串查找“`invalid serial number or name.\n\nyou may need to remove one or more periods from the name`”，可以找到如下代码：

```
004DAFE8  /.  55             push    ebp
004DAFE9  |.  8BEC          mov     ebp, esp
004DAFEB  |.  33C9          xor     ecx, ecx
004DAFED  |.  51            push    ecx
004DAFEE  |.  51            push    ecx
004DAFEF  |.  51            push    ecx
004DAFF0  |.  51            push    ecx
004DAFF1  |.  51            push    ecx
004DAFF2  |.  51            push    ecx
004DAFF3  |.  53            push    ebx
004DAFF4  |.  56            push    esi
004DAFF5  |.  57            push    edi
004DAFF6  |.  8945 F4       mov     [ebp-C], eax
004DAFF9  |.  33C0          xor     eax, eax
004DAFFB  |.  55            push    ebp
004DAFFC  |.  68 D2B14D00   push    004DB1D2
004DB001  |.  64:FF30       push    dword ptr fs:[eax]
004DB004  |.  64:8920       mov     fs:[eax], esp
004DB007  |.  8D4D F0       lea     ecx, [ebp-10]
004DB00A  |.  8B45 F4       mov     eax, [ebp-C]
004DB00D  |.  8B80 D0010000 mov     eax, [eax+1D0]             ;  CB
004DB013  |.  BA 20000000   mov     edx, 20
004DB018  |.  E8 1BEBFFFF   call    004D9B38
004DB01D  |.  8D55 EC       lea     edx, [ebp-14]
004DB020  |.  8B45 F4       mov     eax, [ebp-C]
004DB023  |.  8B80 AC010000 mov     eax, [eax+1AC]
004DB029  |.  E8 46ACF3FF   call    00415C74
004DB02E  |.  8B45 EC       mov     eax, [ebp-14]              ;  用户名
004DB031  |.  8D55 FC       lea     edx, [ebp-4]
004DB034  |.  E8 B3CEFFFF   call    004D7EEC
004DB039  |.  8B45 FC       mov     eax, [ebp-4]              ;  用户名
004DB03C  |.  E8 D787F2FF   call    00403818                   ;  计算位数函数
004DB041  |.  85C0          test    eax, eax
004DB043  |.  0F84 56010000 je      004DB19F
004DB049  |.  8D55 F8       lea     edx, [ebp-8]
004DB04C  |.  8B45 F4       mov     eax, [ebp-C]
004DB04F  |.  8B80 B0010000 mov     eax, [eax+1B0]
004DB055  |.  E8 1AACF3FF   call    00415C74
004DB05A  |.  8B45 F4       mov     eax, [ebp-C]
004DB05D  |.  8B88 D0010000 mov     ecx, [eax+1D0]             ;  CB
004DB063  |.  8B55 F8       mov     edx, [ebp-8]               ;  假码
004DB066  |.  8B45 FC       mov     eax, [ebp-4]               ;  用户名
004DB069  |.  E8 5AF0FFFF   call    004DA0C8                   ;  想都不用想了。这个关键 call
004DB06E  |.  84C0          test    al, al                     ；标志位
004DB070  |.  74 12         je      short 004DB084             ；关键跳
```

### 2、跟进 004DB069 call 004DA0C8

跟进 `004DB069 call 004DA0C8` 得到：

```
004DA0C8  /$  55            push    ebp
004DA0C9  |.  8BEC          mov     ebp, esp
004DA0CB  |.  6A 00         push    0
004DA0CD  |.  6A 00         push    0
004DA0CF  |.  6A 00         push    0
004DA0D1  |.  6A 00         push    0
004DA0D3  |.  6A 00         push    0
004DA0D5  |.  6A 00         push    0
004DA0D7  |.  6A 00         push    0
004DA0D9  |.  6A 00         push    0
004DA0DB  |.  53            push    ebx
004DA0DC  |.  56            push    esi
004DA0DD  |.  57            push    edi
004DA0DE  |.  894D F4       mov     [ebp-C], ecx
004DA0E1  |.  8955 F8       mov     [ebp-8], edx
004DA0E4  |.  8945 FC       mov     [ebp-4], eax
004DA0E7  |.  8B45 FC       mov     eax, [ebp-4]
004DA0EA  |.  E8 DD98F2FF   call    004039CC
004DA0EF  |.  8B45 F8       mov     eax, [ebp-8]
004DA0F2  |.  E8 D598F2FF   call    004039CC
004DA0F7  |.  8B45 F4       mov     eax, [ebp-C]
004DA0FA  |.  E8 CD98F2FF   call    004039CC
004DA0FF  |.  33C0          xor     eax, eax
004DA101  |.  55            push    ebp
004DA102  |.  68 31A44D00   push    004DA431
004DA107  |.  64:FF30       push    dword ptr fs:[eax]
004DA10A  |.  64:8920       mov     fs:[eax], esp
004DA10D  |.  33DB          xor     ebx, ebx
004DA10F  |.  8D55 E8       lea     edx, [ebp-18]
004DA112  |.  8B45 FC       mov     eax, [ebp-4]
004DA115  |.  E8 D2DDFFFF   call    004D7EEC
004DA11A  |.  8D55 E4       lea     edx, [ebp-1C]
004DA11D  |.  8B45 F8       mov     eax, [ebp-8]
004DA120  |.  E8 87C0F2FF   call    004061AC                   ;  对输入的注册码的格式化
004DA125  |.  8B55 E4       mov     edx, [ebp-1C]
004DA128  |.  8D45 F8       lea     eax, [ebp-8]
004DA12B  |.  E8 0896F2FF   call    00403738
004DA130  |.  8B45 E8       mov     eax, [ebp-18]
004DA133  |.  E8 E096F2FF   call    00403818                   ;  位数
004DA138  |.  85C0          test    eax, eax
004DA13A  |.  0F8E D6020000 jle     004DA416
004DA140  |.  8B45 F8       mov     eax, [ebp-8]
004DA143  |.  E8 D096F2FF   call    00403818                   ;  位数
004DA148  |.  85C0          test    eax, eax
004DA14A  |.  0F8E C6020000 jle     004DA416
004DA150  |.  8D45 EC       lea     eax, [ebp-14]
004DA153  |.  50            push    eax
004DA154  |.  B9 20000000   mov     ecx, 20
004DA159  |.  8B55 F4       mov     edx, [ebp-C]
004DA15C  |.  8B45 FC       mov     eax, [ebp-4]
004DA15F  |.  E8 88FEFFFF   call    004D9FEC                   ;  第一次调用，最关键的地方 里 嘻嘻！
004DA164  |.  8D45 F0       lea     eax, [ebp-10]
004DA167  |.  50            push    eax
004DA168  |.  B9 10000000   mov     ecx, 10
004DA16D  |.  8B55 F4       mov     edx, [ebp-C]
004DA170  |.  8B45 FC       mov     eax, [ebp-4]
004DA173  |.  E8 74FEFFFF   call    004D9FEC                   ;  第二次调用关键代码
004DA178  |.  8D55 E0       lea     edx, [ebp-20]
004DA17B  |.  8B45 F8       mov     eax, [ebp-8]
004DA17E  |.  E8 F1DCFFFF   call    004D7E74
004DA183  |.  8B45 E0       mov     eax, [ebp-20]
004DA186  |.  8D55 E4       lea     edx, [ebp-1C]
004DA189  |.  E8 66DCFFFF   call    004D7DF4                   ;  这个出现了。
004DA18E  |.  8B55 E4       mov     edx, [ebp-1C]              ;  假码
004DA191  |.  8B45 F0       mov     eax, [ebp-10]              ;  真码"RKS-5202132"
004DA194  |.  E8 8F97F2FF   call    00403928                   ;  比较函数
004DA199  |.  74 27         je      short 004DA1C2
004DA19B  |.  8D55 E0       lea     edx, [ebp-20]
004DA19E  |.  8B45 F8       mov     eax, [ebp-8]
004DA1A1  |.  E8 CEDCFFFF   call    004D7E74
004DA1A6  |.  8B45 E0       mov     eax, [ebp-20]
004DA1A9  |.  8D55 E4       lea     edx, [ebp-1C]
004DA1AC  |.  E8 43DCFFFF   call    004D7DF4
004DA1B1  |.  8B55 E4       mov     edx, [ebp-1C]
004DA1B4  |.  8B45 EC       mov     eax, [ebp-14]              ；同样是真码“RKS-1300038”
004DA1B7  |.  E8 6C97F2FF   call    00403928                   ；比较函数
004DA1BC  |.  74 04         je      short 004DA1C2
004DA1BE  |.  33DB          xor     ebx, ebx
004DA1C0  |.  EB 02         jmp     short 004DA1C4
004DA1C2  |>  B3 01         mov     bl, 1
004DA1C4  |>  84DB          test    bl, bl
004DA1C6  |./0F85 1A010000 jnz     004DA2E6
004DA1CC  |. |8B45 F4       mov     eax, [ebp-C]               ;  固定字符串“CB”
```

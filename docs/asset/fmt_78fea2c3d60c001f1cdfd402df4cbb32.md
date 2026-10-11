
# 资料收集管理专家 V1.50

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-502.htm`](https://www.pediy.com/kssd/pediy08/pediy8-502.htm)

**【破文标题】** 资料收集管理专家(`DataCollector`) 版本: `V1.50`

**【破文作者】** `inbreak`

**【破解工具】** `OD`，`W32Dasm`，`ASPackDie`

**【破解平台】** `XP` `SP2`

**【软件名称】** 资料收集管理专家(`DataCollector`)

**【软件大小】** `1.41MB`

**【原版下载】** `http://www.delphidak.com/`

**【保护方式】** 壳，注册码

**【编写语言】** `DELPHI`

**【破解感言】** 没想到超新大菜鸟有一天也能写一篇破文。汗！

--------------------------------------------------------------------------------

**【破解声明】** 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

**【破解过程】**

运行软件后出理提示，跟着是一个注册提示窗口，随便输入以下信息：

- 注册名：`inberak`
- 注册人：随便，注册码与此值无关
- 注册码：`789789789789`

点注册，出现有错误提示。

然后用 `W32Dasm` 反编译找一个错误提示的内容，很容易就找到地址：

```
005038AA  |.  /75 2C          jnz     short 005038D8            ;  关键跳，不跳就失败！
```

那么我们再向前看

```
0050384C  /.   55             push    ebp                       ;在这里下断点吧
0050384D  |.   8BEC          mov     ebp, esp
0050384F  |.   33C9          xor     ecx, ecx
00503851  |.   51            push    ecx
00503852  |.   51            push    ecx
00503853  |.   51            push    ecx
00503854  |.   51            push    ecx
00503855  |.   53            push    ebx
00503856  |.   56            push    esi
00503857  |.   8BD8          mov     ebx, eax
00503859  |.   33C0          xor     eax, eax
0050385B  |.   55            push    ebp
0050385C  |.   68 63395000   push    00503963
00503861  |.   64:FF30       push    dword ptr fs:[eax]
00503864  |.   64:8920       mov     dword ptr fs:[eax], esp
00503867  |.   8D55 F8       lea     edx, dword ptr [ebp-8]
0050386A  |.   8B83 18030000 mov     eax, dword ptr [ebx+318]
00503870  |.   E8 C3A3F4FF   call    0044DC38                    ;  得到假注册码 789789789
00503875  |.   8B45 F8       mov     eax, dword ptr [ebp-8]
00503878  |.   50            push    eax
00503879  |.   8D55 F4       lea     edx, dword ptr [ebp-C]
0050387C  |.   8B83 14030000 mov     eax, dword ptr [ebx+314]
00503882  |.   E8 B1A3F4FF   call    0044DC38
00503887  |.   8B45 F4       mov     eax, dword ptr [ebp-C]       ;  注册人：KAKAKAKA
0050388A  |.   50            push    eax
0050388B  |.   8D55 F0       lea     edx, dword ptr [ebp-10]
0050388E  |.   8B83 10030000 mov     eax, dword ptr [ebx+310]
00503894  |.   E8 9FA3F4FF   call    0044DC38
00503899  |.   8B55 F0       mov     edx, dword ptr [ebp-10]      ;  注册名：inbreak
0050389C  |.   8B83 2C030000 mov     eax, dword ptr [ebx+32C]
005038A2  |.   59            pop     ecx
005038A3  |.   E8 8CF0FFFF   call    00502934                    ;  关键 CALL
005038A8  |.   84C0          test    al, al
005038AA  |.   75 2C         jnz     short 005038D8              ;  关键跳，不跳就失败！
```

--------------------------

`F7` 跟进 `005038A3 |. E8 8CF0FFFF call 00502934 ; 关键 CALL`

```
00502934  /$  55            push    ebp
00502935  |.  8BEC          mov     ebp, esp
00502937  |.  83C4 F0       add     esp, -10
0050293A  |.  53            push    ebx
0050293B  |.  33DB          xor     ebx, ebx
0050293D  |.  895D F0       mov     dword ptr [ebp-10], ebx
00502940  |.  895D F4       mov     dword ptr [ebp-C], ebx
00502943  |.  894D F8       mov     dword ptr [ebp-8], ecx
00502946  |.  8955 FC       mov     dword ptr [ebp-4], edx
00502949  |.  8BD8          mov     ebx, eax
0050294B  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0050294E  |.  E8 5D28F0FF   call    004051B0
00502953  |.  8B45 F8       mov     eax, dword ptr [ebp-8]
00502956  |.  E8 5528F0FF   call    004051B0
0050295B  |.  8B45 08       mov     eax, dword ptr [ebp+8]
0050295E  |.  E8 4D28F0FF   call    004051B0
00502963  |.  33C0          xor     eax, eax
00502965  |.  55            push    ebp
00502966  |.  68 1E2A5000   push    00502A1E
0050296B  |.  64:FF30       push    dword ptr fs:[eax]
0050296E  |.  64:8920       mov     dword ptr fs:[eax], esp
00502971  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
00502974  |.  E8 4F26F0FF   call    00404FC8
00502979  |.  3B43 4C       cmp     eax, dword ptr [ebx+4C]
0050297C  |.  7F 19         jg      short 00502997
0050297E  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
00502981  |.  E8 4226F0FF   call    00404FC8
00502986  |.  3B43 50       cmp     eax, dword ptr [ebx+50]
00502989  |.  7C 0C         jl      short 00502997
0050298B  |.  8B45 08       mov     eax, dword ptr [ebp+8]
0050298E  |.  E8 3526F0FF   call    00404FC8
00502993  |.  85C0          test    eax, eax
00502995  |.  75 04         jnz     short 0050299B
00502997  |>  33DB          xor     ebx, ebx
00502999  |.  EB 60         jmp     short 005029FB
0050299B  |>  8D55 F4       lea     edx, dword ptr [ebp-C]
0050299E  |.  8B45 08       mov     eax, dword ptr [ebp+8]
005029A1  |.  E8 926BF0FF   call    00409538
005029A6  |.  8B55 F4       mov     edx, dword ptr [ebp-C]
005029A9  |.  8D45 08       lea     eax, dword ptr [ebp+8]
005029AC  |.  E8 F723F0FF   call    00404DA8
005029B1  |.  8D4D F0       lea     ecx, dword ptr [ebp-10]
005029B4  |.  8B55 FC       mov     edx, dword ptr [ebp-4]
005029B7  |.  8BC3          mov     eax, ebx
005029B9  |.  E8 46FBFFFF   call    00502504                    ;  关键 CALL
```

这里再用 `F7` 跟进

```
00502504  /$  55            push    ebp
00502505  |.  8BEC          mov     ebp, esp
00502507  |.  51            push    ecx
00502508  |.  B9 04000000   mov     ecx, 4
0050250D  |>  6A 00         /push   0
0050250F  |.  6A 00         |push   0
00502511  |.  49            |dec    ecx
00502512  |.^ 75 F9         \jnz    short 0050250D
00502514  |.  874D FC       xchg    dword ptr [ebp-4], ecx
00502517  |.  53            push    ebx
00502518  |.  56            push    esi
00502519  |.  57            push    edi
0050251A  |.  8BF9          mov     edi, ecx
0050251C  |.  8955 FC       mov     dword ptr [ebp-4], edx
0050251F  |.  8BF0          mov     esi, eax
00502521  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
00502524  |.  E8 872CF0FF   call    004051B0
00502529  |.  33C0          xor     eax, eax
0050252B  |.  55            push    ebp
0050252C  |.  68 A4265000   push    005026A4
00502531  |.  64:FF30       push    dword ptr fs:[eax]
00502534  |.  64:8920       mov     dword ptr fs:[eax], esp
00502537  |.  8D55 DC       lea     edx, dword ptr [ebp-24]
0050253A  |.  8BC6          mov     eax, esi
0050253C  |.  E8 070F0000   call    00503448                    ;  得到的机器码
00502541  |.  8B45 DC       mov     eax, dword ptr [ebp-24]
00502544  |.  8D55 EC       lea     edx, dword ptr [ebp-14]
00502547  |.  E8 3C72F0FF   call    00409788
0050254C  |.  837D EC 00    cmp     dword ptr [ebp-14], 0
00502550  |.  75 0D         jnz     short 0050255F
00502552  |.  8D45 E0       lea     eax, dword ptr [ebp-20]
00502555  |.  8B55 FC       mov     edx, dword ptr [ebp-4]
00502558  |.  E8 4B28F0FF   call    00404DA8
0050255D  |.  EB 5D         jmp     short 005025BC
0050255F  |>  8B45 EC       mov     eax, dword ptr [ebp-14]
00502562  |.  E8 612AF0FF   call    00404FC8
00502567  |.  8BD8          mov     ebx, eax
00502569  |.  8D45 E8       lea     eax, dword ptr [ebp-18]
0050256C  |.  50            push    eax
0050256D  |.  8BCB          mov     ecx, ebx
0050256F  |.  D1F9          sar     ecx, 1
00502571  |.  79 03         jns     short 00502576
00502573  |.  83D1 00       adc     ecx, 0
00502576  |>  BA 01000000   mov     edx, 1
0050257B  |.  8B45 EC       mov     eax, dword ptr [ebp-14]
0050257E  |.  E8 9D2CF0FF   call    00405220
00502583  |.  8D45 E4       lea     eax, dword ptr [ebp-1C]
```


# 闹钟和笑话 Ver1.0 注册分析(上)(下）

> 原文：[`www.pediy.com/kssd/pediy12/86782.html`](https://www.pediy.com/kssd/pediy12/86782.html)

【文章标题】闹钟和笑话 Ver1.0 注册分析

【文章作者】`jackozoo`

【作者邮箱】`jackozoo@163.com`

【软件名称】闹钟和笑话

【软件大小】`480K`

【原版下载】`http://catman.iglobalserver.com/`

【保护方式】机器码-序列号

【软件简介】用于日常事务的提醒, 闹钟, 以及播放笑话的功能 .

【作者声明】本文仅供研究学习，本人对因这篇文章而导致的一切后果，不承担任何法律责任。本文中的不足之处请各位多多指教

---

P.S : 我发现上面的原版下载地址有时打不开,所以我将原版安装程序放在续贴的附件中供需要的朋友下载. 同时为了便于阅读,分为上下帖. ![](img/c1d8523a7842d7b67917c210ac6dcf18.jpg "微笑")

### 分析过程

#### 大体简介

这个软件结构比较简单, `Delphi` 写的 . 注册机制是: 先是机器码自动生成, 然后我们输入注册码进行验证, 其中注册码还分试用版的注册码和正式版的注册码.

不过判断部分太失败了, 竟然是明码比较的方式, 当然为了不太扫兴, 我大概分析了下整个程序的算法.

#### 主要目标

1. 分析正式版注册码的生成.
2. 分析试用版注册码的生成.
3. 分析程序注册信息的保存.
4. 分析程序的验证方式.
5. 分析机器码的生成方式.
6. 心得与体会

#### 详细过程

##### 0. 初探门径

`PEID` 之, `ASPack`, 脱之. 再 `PEID`, `Borland` `Delphi` `4.0` - `5.0`.

运行之, 窗口标题栏显示 "闹钟与笑话(未注册)", 点击看笑话按钮, 发现可看的序号有限制. 点注册按钮,

机器码已自动显示出来. 我们只需输入序列号. 随便输入 `ABCDEF`. 确定. 弹错.

开始动手了:

`DeDe` 载入, 切换到 `RegForm` 窗体. 得到注册按钮事件的 `VA`. `OD` 下断跟踪. 试炼码: `ABCDEF`

如下:

```
00482530  >/.   55            push    ebp
00482531   |.   8BEC          mov     ebp, esp
00482533   |.   33C9          xor     ecx, ecx
00482535   |.   51            push    ecx
00482536   |.   51            push    ecx
00482537   |.   51            push    ecx                             ;  这里在开辟局部变量空间,和 VC 很不同.
00482538   |.   51            push    ecx
00482539   |.   51            push    ecx
0048253A   |.   53            push    ebx
0048253B   |.   8BD8          mov     ebx, eax                        ;  保存 RegForm 句柄至 ebx
0048253D   |.   33C0          xor     eax, eax
0048253F   |.   55            push    ebp
00482540   |.   68 27264800   push    dumped\_.00482627
00482545   |.   64:FF30       push    dword ptr fs:[eax]
00482548   |.   64:8920       mov     fs:[eax], esp                   ;  SEH, try 块.
0048254B   |.   8D55 FC       lea     edx, [ebp-4]                    ;  缓冲区. 存放我们的注册码.
0048254E  >|.   8B83 E0020000 mov     eax, [ebx+2E0]                  ;  Edit 控件.
00482554  >|.   E8 4F00FBFF   call    dumped\_.004325A8               ;  TControl.GetText(TControl):TCaption;
00482559   |.   8D55 F0       lea     edx, [ebp-10]                   ;  存放机器码的缓冲区.
0048255C  >|.   8B83 E8020000 mov     eax, [ebx+2E8]                  ;  机器码的 Edit 控件
00482562  >|.   E8 4100FBFF   call    dumped\_.004325A8               ;  TControl.GetText(TControl):TCaption;
00482567   |.   8B45 F0       mov     eax, [ebp-10]
0048256A   |.   8D55 F8       lea     edx, [ebp-8]
0048256D  >|.   E8 1AFAFFFF   call    dumped\_.00481F8C               ;  ->:TfrmSetAwake._PROC_00481F8C()
00482572   |.   8D55 EC       lea     edx, [ebp-14]
00482575  >|.   8B83 E8020000 mov     eax, [ebx+2E8]                  ;  *Edit2:TEdit
0048257B  >|.   E8 2800FBFF   call    dumped\_.004325A8               ;  ->controls.TControl.GetText(TControl):TCaption;
00482580   |.   8B45 EC       mov     eax, [ebp-14]
00482583   |.   8D55 F4       lea     edx, [ebp-C]
00482586  >|.   E8 69FAFFFF   call    dumped\_.00481FF4               ;  这个 call 会得到一个注册码,但是是试用版的.
0048258B   |.   8B45 FC       mov     eax, [ebp-4]                    ;  试炼码
0048258E   |.   8B55 F8       mov     edx, [ebp-8]                    ;  试用版注册码
00482591  >|.   E8 5E19F8FF   call    dumped\_.00403EF4               ;  比较
00482596   |.   75 28         jnz     short dumped\_.004825C0         ;  如果相等的话, 则为试用版.(概率太小..)
00482598   |.   8B55 FC       mov     edx, [ebp-4]
0048259B   |.   8BC3          mov     eax, ebx
0048259D  >|.   E8 5E010000   call    dumped\_.00482700               ;  这里是正式版的注册码生成 call
004825A2   |.   84C0          test    al, al
004825A4   |.   74 1A         je      short dumped\_.004825C0
004825A6   |.   B8 3C264800   mov     eax, dumped\_.0048263C          ;  试用版提示信息.
004825AB   |.   E8 14D4FFFF   call    dumped\_.0047F9C4               ;  MessageBox
004825B0   |.   C683 F4020000>mov     byte ptr [ebx+2F4], 0
004825B7   |.   8BC3          mov     eax, ebx
004825B9  >|.   E8 5EAEFCFF   call    dumped\_.0044D41C               ;  关闭窗口.
004825BE   |.   EB 3F         jmp     short dumped\_.004825FF
004825C0  |>   8B45 FC       mov     eax, [ebp-4]
004825C3   |.   8B55 F4       mov     edx, [ebp-C]
004825C6  >|.   E8 2919F8FF   call    dumped\_.00403EF4               ;  这里才是和正式版的注册码进行比较.
004825CB   |.   75 28         jnz     short dumped\_.004825F5
004825CD   |.   8B55 FC       mov     edx, [ebp-4]                    ;  参数二为注册码.
004825D0   |.   8BC3          mov     eax, ebx                        ;  参数一为 Reg 窗体的句柄.
004825D2  >|.   E8 29010000   call    dumped\_.00482700               ;  到这里就是没跳,而软件要保持了注册信息,下面又无其他陌生 call,所以这里就是写入注册信息.
004825D7   |.   84C0          test    al, al
004825D9   |.   74 1A         je      short dumped\_.004825F5         ;  写入失败,跳走.
004825DB   |.   C683 F4020000>mov     byte ptr [ebx+2F4], 0
004825E2   |.   B8 68264800   mov     eax, dumped\_.00482668
004825E7   |.   E8 D8D3FFFF   call    dumped\_.0047F9C4               ;  完成注册提示!
004825EC   |.   8BC3          mov     eax, ebx                        ;  ebx 保存的为 Reg 窗口的句柄.给 eax 作为参数.
004825EE  >|.   E8 29AEFCFF   call    dumped\_.0044D41C               ;  关闭窗口.
004825F3   |.   EB 0A         jmp     short dumped\_.004825FF
004825F5  |>   B8 98264800   mov     eax, dumped\_.00482698
004825FA   |.   E8 C5D3FFFF   call    dumped\_.0047F9C4               ;  弹错,窗口保留.
004825FF  |>   33C0          xor     eax, eax
00482601   |.   5A            pop     edx
00482602   |.   59            pop     ecx
00482603   |.   59            pop     ecx
00482604   |.   64:8910       mov     fs:[eax], edx
00482607   |.   68 2E264800   push    dumped\_.0048262E
0048260C  |>   8D45 EC       lea     eax, [ebp-14]
0048260F   |.   BA 02000000   mov     edx, 2
00482614  >|.   E8 6F15F8FF   call    dumped\_.00403B88               ;  清理
00482619   |.   8D45 F4       lea     eax, [ebp-C]
0048261C   |.   BA 03000000   mov     edx, 3
00482621  >|.   E8 6215F8FF   call    dumped\_.00403B88
00482626   \.   C3            retn
00482627    .^ E9 4C0FF8FF   jmp     dumped\_.00403578
0048262C    .^ EB DE         jmp     short dumped\_.0048260C
0048262E    .  5B            pop     ebx
0048262F    .  8BE5          mov     esp, ebp
00482631    .  5D            pop     ebp
00482632    .  C3            retn
```

##### 1. 正式版注册码生成算法的分析.

```
00481FF4  /$  55            push    ebp                         ;  试用版注册码生成 Call
00481FF5   |.   8BEC          mov     ebp, esp
00481FF7   |.   83C4 F8       add     esp, -8
00481FFA   |.   53            push    ebx
00481FFB   |.   33C9          xor     ecx, ecx
00481FFD   |.   894D F8       mov     [ebp-8], ecx
00482000   |.   8BDA          mov     ebx, edx
00482002   |.   8945 FC       mov     [ebp-4], eax
00482005   |.   8B45 FC       mov     eax, [ebp-4]
00482008   |.   E8 8B1FF8FF   call    dumped\_.00403F98
0048200D   |.   33C0          xor     eax, eax
0048200F   |.   55            push    ebp
00482010   |.   68 4F204800   push    dumped\_.0048204F
00482015   |.   64:FF30       push    dword ptr fs:[eax]
00482018   |.   64:8920       mov     fs:[eax], esp
0048201B   |.   8D4D F8       lea     ecx, [ebp-8]              ;  参数 3:存放试用版注册码. (也即输出参数)
0048201E   |.   66:BA 31D4    mov     dx, 0D431                 ;  参数 2:常熟一个, 作为参数
00482022   |.   8B45 FC       mov     eax, [ebp-4]              ;  参数 1:机器码.
```



# 代码片段与逻辑分析

## 初始调用序列

```
00482025  |.  E8 32000000   call    dumped_.0048205C                      ; call_1
0048202A  |.  8B45 F8       mov     eax, [ebp-8]
0048202D  |.  8BD3          mov     edx, ebx
0048202F  |.  E8 8C000000   call    dumped_.004820C0                      ; call_2
00482034  |.  33C0          xor     eax, eax
00482036  |.  5A            pop     edx
00482037  |.  59            pop     ecx
00482038  |.  59            pop     ecx
00482039  |.  64:8910       mov     fs:[eax], edx
0048203C  |.  68 56204800   push    dumped_.00482056
00482041  |>  8D45 F8       lea     eax, [ebp-8]
00482044  |.  BA 02000000   mov     edx, 2
00482049  |.  E8 3A1BF8FF   call    dumped_.00403B88
0048204E  |\.  C3           retn
```

可以看到，就那两个 `call` 是关键。一个个来：

## `call_1` 分析

### 代码段

```
//call_1

0048205C  /$  53            push    ebx
0048205D  |.  56            push    esi
0048205E  |.  57            push    edi
0048205F  |.  55            push    ebp
00482060  |.  51            push    ecx
00482061  |.  8BE9          mov     ebp, ecx
00482063  |.  8BF2          mov     esi, edx                      ; 这里把那个常数给 esi.
00482065  |.  890424        mov     [esp], eax
00482068  |.  8BC5          mov     eax, ebp
0048206A  |.  8B1424        mov     edx, [esp]
0048206D  |.  E8 461BF8FF   call    dumped_.00403BB8
00482072  |.  8B0424        mov     eax, [esp]
00482075  |.  E8 6A1DF8FF   call    dumped_.00403DE4                ; 求字符串长度.
0048207A  |.  8BF8          mov     edi, eax
0048207C  |.  85FF          test    edi, edi
0048207E  |.  7E 39         jle     short dumped_.004820B9          ; 空串则玩完.
00482080  |.  BB 01000000   mov     ebx, 1                          ; 循环计数 i
00482085  |>  8BC5          /mov    eax, ebp
00482087  |.  E8 281FF8FF   |call   dumped_.00403FB4
0048208C  |.  8B1424        |mov    edx, [esp]
0048208F  |.  8A541A FF     |mov    dl, [edx+ebx-1]
00482093  |.  0FB7CE        |movzx   ecx, si                       ; 到这里知道那个是加密 key 了.
00482096  |.  C1E9 08       |shr     ecx, 8
00482099  |.  32D1          |xor     dl, cl
0048209B  |.  885418 FF     |mov     [eax+ebx-1], dl                 ; 这里就是写入.
0048209F  |.  8B45 00       |mov     eax, [ebp]
004820A2  |.  0FB64418 FF   |movzx   eax, byte ptr [eax+ebx-1]
004820A7  |.  66:03F0       |add     si, ax
004820AA  |.  66:69C6 09CE  |imul    ax, si, 0CE09
004820AF  |.  66:05 5B58    |add     ax, 585B
004820B3  |.  8BF0          |mov     esi, eax                      ; 这几句对 key 进行变化.
004820B5  |.  43            |inc     ebx
004820B6  |.  4F            |dec     edi
004820B7  |\.^  75 CC       \jnz    short dumped_.00482085
004820B9  |>  5A            pop     edx
004820BA  |.  5D            pop     ebp
004820BB  |.  5F            pop     edi
004820BC  |.  5E            pop     esi
004820BD  |.  5B            pop     ebx
004820BE  |\\.  C3           retn
```

### 逻辑描述

这个 `call_1` 比较简单。就是一个简单变换，它生成的是一个同机器码相同长度的序列。记为 `machine ---> machine_e`：

大致过程：

```c
for (i=0; i<strlen((char*)machine); ++i)
{
    machine_e[i] = machine[i] ^ ((WORD)key >> 8);
    key += (BYTE)machine_e[i];
    key = (WORD)((WORD)(key * 0xCE09) + 0x585B);
}
```

## `call_2` 分析

再来看 `call_2`：

```
004820C0  /$  53            push    ebx
004820C1  |.  56            push    esi
004820C2  |.  57            push    edi
004820C3  |.  55            push    ebp
004820C4  |.  83C4 F4       add     esp, -0C
004820C7  |.  891424        mov     [esp], edx
004820CA  |.  8BF0          mov     esi, eax
004820CC  |.  8BC6          mov     eax, esi
004820CE  |.  E8 111DF8FF   call    dumped_.00403DE4                ; 得到 machine_e 长度
004820D3  |.  83C0 02       add     eax, 2
004820D6  |.  B9 03000000   mov     ecx, 3
004820DB  |.  99            cdq
004820DC  |.  F7F9          idiv    ecx
004820DE  |.  8BD0          mov     edx, eax
004820E0  |.  C1E2 02       shl     edx, 2                          ; 参数 2: strlen(m)+2/3*4
004820E3  |.  8B0424        mov     eax, [esp]                      ; 参数 1:这里就是存放试用版注册码的部分.
004820E6  |.  E8 2D20F8FF   call    dumped_.00404118                ; 设置长度, 忽略之.
004820EB  |.  8BC6          mov     eax, esi                        ; 变形后的机器码.
004820ED  |.  E8 F21CF8FF   call    dumped_.00403DE4                ; 求长度 call.
004820F2  |.  83C0 02       add     eax, 2
004820F5  |.  B9 03000000   mov     ecx, 3
004820FA  |.  99            cdq
004820FB  |.  F7F9          idiv    ecx
004820FD  |.  8BF8          mov     edi, eax                        ; 循环次数.
004820FF  |.  85FF          test    edi, edi
00482101  |.  0F8E 00010000 jle     dumped_.00482207
00482107  |.  BB 01000000   mov     ebx, 1                          ; 循环计数.
0048210C  |>  8BC6          /mov    eax, esi
0048210E  |.  E8 D11CF8FF   |call   dumped_.00403DE4                ; 又求长度, 为 8
00482113  |.  8D145B        |lea     edx, [ebx+ebx*2]
00482116  |.  3BC2          |cmp     eax, edx
00482118  |.  7D 25         |jge     short dumped_.0048213F          ; strlen(machine_e) >= 3*i 则跳
0048211A  |.  8BC6          |mov     eax, esi
0048211C  |.  E8 C31CF8FF   |call    dumped_.00403DE4                ; 还是求长度 call, 大家可以把这个弄个 lable
00482121  |.  8BC8          |mov     ecx, eax
00482123  |.  8BC3          |mov     eax, ebx
00482125  |.  48            |dec     eax
00482126  |.  8D0440        |lea     eax, [eax+eax*2]
00482129  |.  2BC8          |sub     ecx, eax                        ; Move 函数的 arg3 = strlen(machine_e) -3*(i-1)
0048212B  |.  8D5424 04     |lea     edx, [esp+4]                    ; 这个缓冲区暂记为 temp[7]
0048212F  |.  8BC3          |mov     eax, ebx
00482131  |.  48            |dec     eax
00482132  |.  8D0440        |lea     eax, [eax+eax*2]
00482135  |.  8D0406        |lea     eax, [esi+eax]                  ; 这是源串的起始地址.
00482138  |.  E8 0307F8FF   |call    dumped_.00402840                ; 这个是 Move. 大家可以当成是 memcpy 函数.
0048213D  |.  EB 17         |jmp     short dumped_.00482156
0048213F  |>  8D5424 04     |lea     edx, [esp+4]                    ; 循环中的前面几次会到这里来的.
00482143  |.  8BC3          |mov     eax, ebx
00482145  |.  48            |dec     eax
00482146  |.  8D0440        |lea     eax, [eax+eax*2]
00482149  |.  8D0406        |lea     eax, [esi+eax]
0048214C  |.  B9 03000000   |mov     ecx, 3                          ; 这是这次的 size 为常数 3.
00482151  |.  E8 EA06F8FF   |call    dumped_.00402840                ; Move
00482156  |>  8A4424 04     |mov     al, [esp+4]                     ; 取第一个字符.
0048215A  |.  8BD0          |mov     edx, eax
0048215C  |.  80E2 FC       |and     dl, 0FC
0048215F  |.  81E2 FF000000 |and     edx, 0FF
00482165  |.  C1EA 02       |shr     edx, 2                          ; edx 其实是作为偏移使用的.
00482168  |.  B9 8C224800   |mov     ecx, dumped_.0048228C           ; ASCII "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
0048216D  |.  8A1411        |mov     dl, [ecx+edx]
00482170  |.  885424 07     |mov     [esp+7], dl                     ; temp[3] 赋值. 注意:第一次循环的时候,temp[0]至 temp[2]被 Move 函数赋值.
00482174  |.  24 03         |and     al, 3
00482176  |.  8BE8          |mov     ebp, eax
00482178  |.  81E5 FF000000 |and     ebp, 0FF
0048217E  |.  C1E5 04       |shl     ebp, 4
00482181  |.  8A5424 05     |mov     dl, [esp+5]
00482185  |.  8BC2          |mov     eax, edx
00482187  |.  24 F0         |and     al, 0F0
00482189  |.  25 FF000000   |and     eax, 0FF
```



0048218E  |.  C1E8 04       |shr     eax, 4  

00482191  |.  0BE8          |or      ebp, eax                         ;  以上 N 行为生成新的偏移  

00482193  |.  B8 8C224800   |mov     eax, dumped\_.0048228C          ;  ASCII "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"  

00482198  |.  8A0428        |mov     al, \[eax+ebp\]  

0048219B  |.  884424 08     |mov     \[esp+8\], al                     ;  temp\[4\]赋值.  

0048219F  |.  80E2 0F       |and     dl, 0F                           ;  Ok, 下面的这些都差不多了.  

004821A2  |.  8BEA          |mov     ebp, edx  

004821A4  |.  81E5 FF000000 |and     ebp, 0FF  

004821AA  |.  C1E5 02       |shl     ebp, 2  

004821AD  |.  8A4424 06     |mov     al, \[esp+6\]  

004821B1  |.  8BD0          |mov     edx, eax  

004821B3  |.  80E2 C0       |and     dl, 0C0  

004821B6  |.  81E2 FF000000 |and     edx, 0FF  

004821BC  |.  C1EA 06       |shr     edx, 6  

004821BF  |.  0BEA          |or      ebp, edx  

004821C1  |.  BA 8C224800   |mov     edx, dumped\_.0048228C          ;  ASCII "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"  

004821C6  |.  8A142A        |mov     dl, \[edx+ebp\]  

004821C9  |.  885424 09     |mov     \[esp+9\], dl  

004821CD  |.  24 3F         |and     al, 3F  

004821CF  |.  25 FF000000   |and     eax, 0FF  

004821D4  |.  BA 8C224800   |mov     edx, dumped\_.0048228C          ;  ASCII "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"  

004821D9  |.  8A0402        |mov     al, \[edx+eax\]  

004821DC  |.  884424 0A     |mov     \[esp+A\], al  

004821E0  |.  8B0424        |mov     eax, \[esp\]  

004821E3  |.  E8 CC1DF8FF   |call    dumped\_.00403FB4  

004821E8  |.  8BD3          |mov     edx, ebx  

004821EA  |.  4A            |dec     edx  

004821EB  |.  C1E2 02       |shl     edx, 2  

004821EE  |.  8D1410        |lea     edx, \[eax+edx\]                 ;  这是目的地址,连接注册码: &reg\[4\*(i-1)\]  

004821F1  |.  8D4424 07     |lea     eax, \[esp+7\]                   ;  源地址,即&temp\[3\]  

004821F5  |.  B9 04000000   |mov     ecx, 4                           ;  这里是个关键了, ecx 为 copy 的字节数.  

004821FA  |.  E8 4106F8FF   |call    dumped\_.00402840                ;  Move.每次 4 个.  

004821FF  |.  43            |inc     ebx  

00482200  |.  4F            |dec     edi  

00482201  |.^ 0F85 05FFFFFF \\jnz     dumped\_.0048210C  

00482207  |>   8BC6         |mov     eax, esi                         ;  machine\_e  

00482209  |.  E8 D61BF8FF   |call    dumped\_.00403DE4                ;  还是 strlen  

0048220E  |.  B9 03000000   |mov     ecx, 3  

00482213  |.  99            |cdq  

00482214  |.  F7F9         |idiv    ecx  

00482216  |.  4A            |dec     edx                                ;  除以 3 的余数来 switch-case  

00482217  |.  75 34         |jnz     short dumped\_.0048224D  

00482219  |.  8B0424        |mov     eax, \[esp\]                         ;  余数为 1 处理:  

0048221C  |.  8B00          |mov     eax, \[eax\]  

0048221E  |.  E8 C11BF8FF   |call    dumped\_.00403DE4  

00482223  |.  8BD8          |mov     ebx, eax  

00482225  |.  8B0424        |mov     eax, \[esp\]  

00482228  |.  E8 871DF8FF   |call    dumped\_.00403FB4  

0048222D  |.  C64418 FE 3D  |mov     byte ptr \[eax+ebx-2\], 3D        ;  就这个 mov 特殊点, 看到\[eax+ebx-1\]就知道是填充注册码的尾部了.  

00482232  |.  8B0424        |mov     eax, \[esp\]  

00482235  |.  8B00          |mov     eax, \[eax\]  

00482237  |.  E8 A81BF8FF   |call    dumped\_.00403DE4  

0048223C  |.  8BD8          |mov     ebx, eax  

0048223E  |.  8B0424        |mov     eax, \[esp\]  

00482241  |.  E8 6E1DF8FF   |call    dumped\_.00403FB4  

00482246  |.  C64418 FF 3D  |mov     byte ptr \[eax+ebx-1\], 3D        ;  同上个 mov byte ptr ...  

0048224B  |.  EB 2D         |jmp     short dumped\_.0048227A  

0048224D  |>   8BC6         |mov     eax, esi  

0048224F  |.  E8 901BF8FF   |call    dumped\_.00403DE4                ;  strlen  

00482254  |.  B9 03000000   |mov     ecx, 3  

00482259  |.  99            |cdq  

0048225A  |.  F7F9         |idiv    ecx  

0048225C  |.  83FA 02       |cmp     edx, 2  

0048225F  |.  75 19         |jnz     short dumped\_.0048227A  

00482261  |.  8B0424        |mov     eax, \[esp\]                         ;  余数为 2 处理:  

00482264  |.  8B00          |mov     eax, \[eax\]  

00482266  |.  E8 791BF8FF   |call    dumped\_.00403DE4  

0048226B  |.  8BD8          |mov     ebx, eax  

0048226D  |.  8B0424        |mov     eax, \[esp\]  

00482270  |.  E8 3F1DF8FF   |call    dumped\_.00403FB4  

00482275  |.  C64418 FF 3D  |mov     byte ptr \[eax+ebx-1\], 3D        ;  也是填尾巴的  

0048227A  |>   83C4 0C       |add     esp, 0C                             ;  整除则不处理.  

0048227D  |.  5D            |pop     ebp  

0048227E  |.  5F            |pop     edi  

0048227F  |.  5E            |pop     esi  

00482280  |.  5B            |pop     ebx  

00482281  \\.  C3            |retn  

这个 `call_2` 有点长，不过总的来说，思路很清晰，即为通过 `call_1` 得到的变形码来生成，其中会有一个大循环。

在这个循环里面，有个 `temp[7]`。每次循环进去的时候，在 `machine_m` 中都复制 3 个字节到 `temp` 的内存中(不足则复制零头)。然后根据这三个字节的值计算出一个偏移地址，然后用这个偏移地址加上一个常量字符串表的地址来得到一个字符。

字符表为：

```
const char table[] = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
```

依次得到 4 个字符放入到 `temp` 剩余的缓冲区中。最后用 `Move` 函数将这 4 个字节复制到注册码缓冲区中。

其中最为关键的代码为：

004821EE  |.  8D1410        |lea     edx, \[eax+edx\]                 ;  这是目的地址,连接注册码: &reg\[4\*(i-1)\]  

004821F1  |.  8D4424 07     |lea     eax, \[esp+7\]                   ;  源地址,即&temp\[3\]  

004821F5  |.  B9 04000000   |mov     ecx, 4                           ;  这里是个关键了, ecx 为 copy 的字节数.  

004821FA  |.  E8 4106F8FF   |call    dumped\_.00402840                ;  Move.每次 4 个.  

这个 call 的功能类似于 `memcpy(souce,dest,size)`，注意 Delphi 中常用 `eax`, `edx`, `ecx` 来传参。用 C 语言可以表示为：

```
strcpy((char*)&reg_code[4*(i-1)], &temp[3], 4);
```

索引生成大致过程：

```
mlen = strlen((char*)machine_e);
regLen = (mlen + 2) / 3 * 4;
for (i=0;i<regLen/4;i++)
{
  if ((i+1)*3<mlen)
    strncpy((char*)temp,(char*)&machine_e[3*i],3);
  else
    strncpy((char*)temp,(char*)&machine_e[3*i],mlen-3*i);
  temp[3] = table[(temp[0] & 0xFC) >> 2];
  temp[4] = table[((temp[0] & 0x3) << 4) | ((temp[1] & 0xF0) >> 4)];
  temp[5] = table[((temp[1] & 0xF) << 2) | ((temp[2] & 0xC0) >> 6)];
  temp[6] = table[temp[2] & 0x3F];
  strncpy((char*)&reg[i*4],(char*)&temp[3],4);
}
```

### 2. 试用版的注册码如何生成.

00481F8C  /$  55            |push    ebp                                ;  试用版注册码生成.  

00481F8D  |.  8BEC          |mov     ebp, esp  

00481F8F  |.  83C4 F8      |add     esp, -8  

00481F92  |.  53            |push    ebx  

00481F93  |.  33C9          |xor     ecx, ecx  

00481F95  |.  894D F8      |mov     \[ebp-8\], ecx  

00481F98  |.  8BDA          |mov     ebx, edx  

00481F9A  |.  8945 FC      |mov     \[ebp-4\], eax  

00481F9D  |.  8B45 FC      |mov     eax, \[ebp-4\]  

00481FA0  |.  E8 F31FF8FF  |call    dumped\_.00403F98  

00481FA5  |.  33C0          |xor     eax, eax  

00481FA7  |.  55            |push    ebp  

00481FA8  |.  68 E71F4800   |push    dumped\_.00481FE7  

00481FAD  |.  64:FF30       |push    dword ptr fs:\[eax\]  

00481FB0  |.  64:8920       |mov     fs:\[eax\], esp  

00481FB3  |.  8D4D F8      |lea     ecx, \[ebp-8\]  

00481FB6  |.  66:BA 3930    |mov     dx, 3039                           ;  看到这你吃惊了一下吗? 别慌继续.  

00481FBA  |.  8B45 FC      |mov     eax, \[ebp-4\]  

00481FBD  |.  E8 9A000000   |call    dumped\_.0048205C  

00481FC2  |.  8B45 F8      |mov     eax, \[ebp-8\]  

00481FC5  |.  8BD3          |mov     edx, ebx  

00481FC7  |.  E8 F4000000   |call    dumped\_.004820C0                ;  这里的两个 call 不是如此的熟悉吗? 回去看看正式版的.. . 啊 ?!原来真是那两个 call 了.  

00481FCC  |.  33C0          |xor     eax, eax



```
00481FCE  |.  59                  pop     edx  
00481FCF  |.  59                  pop     ecx  
00481FD0  |.  59                  pop     ecx  
00481FD1  |.  64:8910            mov     fs:[eax], edx  
00481FD4  |.  68 EE1F4800        push    dumped_.00481FEE  
00481FD9  |>  8D45 F8            lea     eax, [ebp-8]  
00481FDC  |.  BA 02000000        mov     edx, 2  
00481FE1  |.  E8 A21BF8FF        call    dumped_.00403B88  
00481FE6  \.  C3                  retn  
```

所以，现在我们明白，其实正式版与试用版的区别只在于两者使用的加密 `key` 不同而已，算法的 `call` 完全一样。

所以到了这里，我们对于注册码的生成就分析完毕了。

### 3. 分析程序注册信息的保存方式

```
//这个也很简单了, 上面我标出了那个注册信息保存 call.

00482700   $  55                  push    ebp                             ;  注册信息保存 call  
00482701   .  8BEC                mov     ebp, esp  
...... //无关信息  

...... //  

0048274D   .  8D55 F0             lea     edx, [ebp-10]  
00482750   .  A1 64624800         mov     eax, [486264]  
00482755   .  8B00                mov     eax, [eax]  
00482757   .  E8 A4E4FCFF         call    dumped_.00450C00                ;  这个是 GetModuleFileName 类似的函数,得到完全路径.前面都是无关紧要的.  
0048275C   .  8B45 F0             mov     eax, [ebp-10]  
0048275F   .  8D55 F4             lea     edx, [ebp-C]  
00482762   .  E8 7566F8FF         call    dumped_.00408DDC                ;  提取出文件名.  
00482767   .  8B4D F4             mov     ecx, [ebp-C]  
0048276A   .  8D45 F8             lea     eax, [ebp-8]  
0048276D   .  BA E4274800         mov     edx, dumped_.004827E4           ;  ASCII "\\SoftWare\\Microsoft\\"  
00482772   .  E8 B916F8FF         call    dumped_.00403E30                ;  字符串连接  
00482777   .  8B55 F8             mov     edx, [ebp-8]  
0048277A   .  B1 01               mov     cl, 1  
0048277C   .  8BC3                mov     eax, ebx  
0048277E   .  E8 F545FFFF         call    dumped_.00476D78                ;  打开注册表.  
00482783   .  8B4D FC             mov     ecx, [ebp-4]                    ;  这是参数 3: 即我们的注册码  
00482786   .  BA 04284800         mov     edx, dumped_.00482804           ;  ASCII "Pwd"  
0048278B   .  8BC3                mov     eax, ebx                        ;  句柄.  
0048278D   .  E8 A247FFFF         call    dumped_.00476F34                ;  写入一个键值.  
00482792   .  8BC3                mov     eax, ebx  
00482794   .  E8 7F06F8FF         call    dumped_.00402E18                ;  释放资源.  
00482799   .  B3 01               mov     bl, 1  
0048279B   .  33C0                xor     eax, eax  

// ...... 
```

所以，程序是用最常用的手法以键值对的形式将注册码保存在注册表中的。

好了，到了这里为了避免此页过长影响大家阅读，剩下的分析（4-7 四个目标的实现）我帖在了下贴上：![](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色") 

下贴地址：[ http://bbs.pediy.com/showthread.php?p=609591](http://bbs.pediy.com/showthread.php?p=609591) 

(此为续贴， 上帖地址：[`bbs.pediy.com/showthread.php?t=86782`](http://bbs.pediy.com/showthread.php?t=86782)) 

### 4. 分析程序的验证方式

程序要验证，这里肯定是刚运行时验证，程序未注册时窗口标题为"闹钟与笑话(未注册)" ，注册了则不会显示未注册这三个字. 而 `Delphi` 一般都是 `FormCrete` 里面完成初始代码的，验证肯定是在那里的（注意窗体要选择主窗体）。

`OK`，我们用 `DeDe` 找出 `FormCreate` 地址.跟进.

如下:

```
......  

004831C6 >|.  E8 35DAFCFF         call    dumped_.00450C00                ;  得到全路径.  
004831CB  |.  8B45 F4             mov     eax, [ebp-C]  
004831CE  |.  8D55 F8             lea     edx, [ebp-8]  
004831D1 >|.  E8 065CF8FF         call    dumped_.00408DDC                ;  得到文件名.  
004831D6  |.  8B4D F8             mov     ecx, [ebp-8]  
004831D9  |.  8D45 FC             lea     eax, [ebp-4]  
004831DC  |.  BA 20334800         mov     edx, dumped_.00483320           ;  ASCII "\\SoftWare\\Microsoft\\"  
004831E1 >|.  E8 4A0CF8FF         call    dumped_.00403E30  
004831E6  |.  8B45 FC             mov     eax, [ebp-4]  
004831E9  |.  BA 40334800         mov     edx, dumped_.00483340           ;  ASCII "Pwd"  
004831EE >|.  E8 B9EBFFFF         call    dumped_.00481DAC                ;  这里是 Check Call 了.  
004831F3  |.  83F8 02             cmp     eax, 2                          ;  有此知 eax 函数返回值需为 2  
004831F6  |.  A1 7C614800         mov     eax, [48617C]  
004831FB  |.  0F9400              sete    [eax]                           ;  关键!:这里设置标志位.  
004831FE  |.  A1 7C614800         mov     eax, [48617C]  
00483203  |.  8038 00             cmp     byte ptr [eax], 0  
00483206  |.  75 0C               jnz     short dumped_.00483214  

......  
```

到 `Check Call` 中看看:

我这截取一段代码吧:

```
00481E17  |.  E8 D8010000         call    dumped_.00481FF4  
00481E1C  |.  8B55 E8             mov     edx, [ebp-18]  
00481E1F  |.  8B45 F0             mov     eax, [ebp-10]  
00481E22  |.  E8 CD20F8FF         call    dumped_.00403EF4  
00481E27  |.  75 07               jnz     short dumped_.00481E30  
00481E29  |.  BB 02000000         mov     ebx, 2                          ;  等会给 eax.  

......  
```

看到这里你别吐啊 ,, 呵呵.

`00481FF4` 不是很熟悉吗... 对，它是正式版注册码生成 `call`. 对作者彻底无语了...

竟然这样比较...

现在一切都明了了...

对，程序调用正式版注册码生成的 `Call`（里面当然还会调用机器码的生成 `call`）来生成一个注册码，然后从注册表读出我们的注册码，将两者直接比较，对作者相当的无语.

### 5. 分析机器码的生成方式

```
  //RegForm 的 FormCreate 事件:  

004826A4 >/.  55                  push    ebp                     ;  注册窗体的 FormCreate 事件.  
004826A5  |.  8BEC                mov     ebp, esp  
004826A7  |.  6A 00               push    0  
004826A9  |.  53                  push    ebx  
004826AA  |.  8BD8                mov     ebx, eax  
004826AC  |.  33C0                xor     eax, eax  
004826AE  |.  55                  push    ebp  
004826AF  |.  68 ED264800        push    dumped_.004826ED  
004826B4  |.  64:FF30            push    dword ptr fs:[eax]  
004826B7  |.  64:8920            mov     fs:[eax], esp  
004826BA  |.  8D45 FC            lea     eax, [ebp-4]  
004826BD >|.  E8 9AF7FFFF         call    dumped_.00481E5C          ;  很明显.这个 call 就是得到机器码的,然后放到下面的 Edit 控件中  
004826C2  |.  8B55 FC            mov     edx, [ebp-4]  
004826C5 >|.  8B83 E8020000      mov     eax, [ebx+2E8]            ;  机器码 Edit 控件  
004826CB >|.  E8 08FFFAFF         call    dumped_.004325D8          ;  TControl.SetText  

......  

//  

00481E5C  /$  55                  push    ebp  
... ...  

00481E7B  |.  6A 00               push    0                         ; /pFileSystemNameSize = NULL  
00481E7D  |.  6A 00               push    0                         ; |pFileSystemNameBuffer = NULL  
00481E7F  |.  8D45 F4            lea     eax, [ebp-C]              ; |  
00481E82  |.  50                  push    eax                       ; |pFileSystemFlags  
00481E83  |.  8D45 F8            lea     eax, [ebp-8]              ; |  
00481E86  |.  50                  push    eax                       ; |pMaxFilenameLength  
00481E87  |.  8D45 FC            lea     eax, [ebp-4]              ; |  
00481E8A  |.  50                  push    eax                       ; |pVolumeSerialNumber  
00481E8B  |.  6A 00               push    0                         ; |MaxVolumeNameSize = 0  
00481E8D  |.  6A 00               push    0                         ; |VolumeNameBuffer = NULL  
00481E8F  |.  68 F81E4800        push    dumped_.00481EF8          ; |RootPathName = "c:\\"  
00481E94  |.  E8 134DF8FF        call    <jmp.&kernel32.GetVolumeInformat>; \GetVolumeInformationA  

... ...  
```

呵呵，作者够懒的了，直接使用 `GetVolumeInformation` 来得到机器码. 而且作者还把好好的一个 `DWORD` 分成两份来 `sprintf`，再 `strcat` 起来，够无语的了.

这机器码分析的也太没意思了....

### 6. 心得与体会

在分析这个程序之前，我对 `Delphi` 是一窍不通的， `pascal` 也不通， 弄完了这个程序， 有了一些心得. 希望对新入门的朋友有点帮助.

1. `Delphi` 是使用 `Object Pascal` 编程的， 参数一般为 `eax`, `edx`, `ecx`, ...这样寄存器传参的.

2. `Delphi` 的程序最好结合 `DeDe` 看分析.

在我弄完了后，我想看看 `IDA` 来分析是什么效果， 结果很多函数识别不了. 于是在看雪搜索了一圈， 发现别人是在 `DeDe` 中导出 `map` 文件，再用 `mapsym` 工具将 `map` 转换为 `sym`，再用 `IDA` 自带的 `loadsym` 来装载进 `IDA`， 不过效果不好.

于是我写了一个 `loadmap.idc` 的脚本， 大部分函数都能很好的搞定.

具体信息见这个帖子:

`http://bbs.pediy.com/showthread.php?t=86623`

手都打酸了， ![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!") .不过要谢谢大家的支持 , ![](img/c1d8523a7842d7b67917c210ac6dcf18.jpg "微笑") .

我发现我在上贴中贴的原软件下载地址有时打不开， 所以我把安装程序也放到附件了， 供需要的朋友学习使用.![](img/c1d8523a7842d7b67917c210ac6dcf18.jpg "微笑")

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || 软件安装程序.rar  
|

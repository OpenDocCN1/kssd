
# 从程序中剥离 GameGuard

> 原文：[`www.pediy.com/kssd/pediy10/71616.html`](https://www.pediy.com/kssd/pediy10/71616.html)

![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬") ![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")

这个我一直不想发，今天就发了吧~~

希望能用的着![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")

为了给大家对对~~我把整个函数都贴出来了。

如果有什么不合适的地方，管理直接删掉或移往它处。。。。![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!") ![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")

今天，就给大家说说怎么跳掉 `GameGuard`，目前这个方法，我已经用了几年了，刚开始是学来的。现在我发出来给大家，希望对大家有帮助，这里只是如何取掉 `GameGuard` 的方法。为的是给大家一个方便，请不要有其它想法的。

估计带着 `NP` 玩游戏，也不会好到那里去~~，今天咱们就送它回老家。

首先，需要了解几个函数:

```c
#define    NP\_SUCCESS  0x0755  //只要执行成功就是这个值.

          //根据汇编来看，估计是个类，而且不是很复杂，新旧版的差别不大。

          //可能是开发商提供的是 LIB 之类的库。

//NP 初始化函数:

DWORD  Init(LPCTSTR lpszName);

//NP 清除函数:

DWORD  Exit();

//NP 加载函数:

DWORD  Load(HWND  hWnd);

//NP 检测函数:

DWORD  Check();

//NP 验证函数:

BOOL  Auth(DWORD  dwAuthCode);

//NP 回调函数:

BOOL CALLBACK NP\_Callback(DWORD dwMsg, DWORD dwUnk);
```

估计类为如下:

```cpp
class CGameGuard
{
public:
  CGameGuard(LPCTSTR lpszName)
  {
    Init(lpszGameName);
  }
  ~CGameGuard()
  {
    Exit();
  }
public:
  DWORD  Init(HWND  hWnd)
  {
    return Load(hWnd);
  }
    DWORD Check()
  {
    return Check();
  }
  BOOL  Auth(DWORD  dwAuthCode)
  {
    return Auth(dwAuthCode);
  }
};
```

`NP` 的创建主要，是 `Init()`函数来执行，这个函数执行就会在线更新，检查非法程序等等。

我把整个函数给大家贴出来，方便大家对对~~~~！这里需要说的是 `Init()`有 `W` 和 `A` 两中函数的。

```asm
//------------------------------------------------------------------------------------------------
00803250  /$  6A FF          push    -1    ;在这里会看到有两个调用，一个是 UN 的转换。
00803252  |.  68 643F8200   push    00823F64                          ;  SE 处理程序安装
00803257  |.  64:A1 0000000>mov     eax, dword ptr fs:[0]
0080325D  |.  50            push    eax
0080325E  |.  64:8925 00000>mov     dword ptr fs:[0], esp
00803265  |.  51            push    ecx
00803266  |.  A1 80062808   mov     eax, dword ptr [8280680]
0080326B  |.  53            push    ebx
0080326C  |.  33DB         xor     ebx, ebx
0080326E  |.  3BC3         cmp     eax, ebx
00803270  |.  74 12        je      short 00803284
00803272  |.  33C0         xor     eax, eax
00803274  |.  5B            pop     ebx
00803275  |.  8B4C24 04     mov     ecx, dword ptr [esp+4]
00803279  |.  64:890D 00000>mov     dword ptr fs:[0], ecx
00803280  |.  83C4 10       add     esp, 10
00803283  |.  C3            retn
00803284  |>  55            push    ebp
00803285  |.  56            push    esi
00803286  |.  68 D06A8000   push    00806AD0                          ; /pTopLevelFilter = main.00806AD0
0080328B  |.  FF15 B8428200 call    dword ptr [<&KERNEL32.SetUnhandl>; \\SetUnhandledExceptionFilter
00803291  |.  68 8C120000   push    128C
00803296  |.  8BE8          mov     ebp, eax
00803298  |.  E8 8DC3FEFF   call    007EF62A
0080329D  |.  8BF0          mov     esi, eax
0080329F  |.  83C4 04       add     esp, 4
008032A2  |.  897424 0C     mov     dword ptr [esp+C], esi
008032A6  |.  3BF3          cmp     esi, ebx
008032A8  |.  895C24 18     mov     dword ptr [esp+18], ebx
008032AC  |.  0F84 AD000000 je      0080335F
008032B2  |.  8D46 2C       lea     eax, dword ptr [esi+2C]
008032B5  |.  881E          mov     byte ptr [esi], bl
008032B7  |.  885E 01       mov     byte ptr [esi+1], bl
008032BA  |.  895E 04       mov     dword ptr [esi+4], ebx
008032BD  |.  895E 08       mov     dword ptr [esi+8], ebx
008032C0  |.  895E 0C       mov     dword ptr [esi+C], ebx
008032C3  |.  895E 10       mov     dword ptr [esi+10], ebx
008032C6  |.  895E 14       mov     dword ptr [esi+14], ebx
008032C9  |.  895E 18       mov     dword ptr [esi+18], ebx
008032CC  |.  885E 1C       mov     byte ptr [esi+1C], bl
008032CF  |.  895E 20       mov     dword ptr [esi+20], ebx
008032D2  |.  885E 24       mov     byte ptr [esi+24], bl
008032D5  |.  885E 25       mov     byte ptr [esi+25], bl
008032D8  |.  885E 26       mov     byte ptr [esi+26], bl
008032DB  |.  50            push    eax                           ; /pCriticalSection
008032DC  |.  C746 28 FFFFF>mov     dword ptr [esi+28], -1          ; |
008032E3  |.  FF15 78428200 call    dword ptr [<&KERNEL32.Initialize>; \\InitializeCriticalSection
008032E9  |.  899E 60100000 mov     dword ptr [esi+1060], ebx
008032EF  |.  8D8E 64100000 lea     ecx, dword ptr [esi+1064]
008032F5  |.  C64424 18 01  mov     byte ptr [esp+18], 1
008032FA  |.  E8 215E0000   call    00809120
008032FF  |.  8D8E BC110000 lea     ecx, dword ptr [esi+11BC]
00803305  |.  C64424 18 02  mov     byte ptr [esp+18], 2
0080330A  |.  E8 41650000   call    00809850
0080330F  |.  899E 04120000 mov     dword ptr [esi+1204], ebx
00803315  |.  899E 28120000 mov     dword ptr [esi+1228], ebx
0080331B  |.  899E 2C120000 mov     dword ptr [esi+122C], ebx
00803321  |.  889E 70120000 mov     byte ptr [esi+1270], bl
00803327  |.  889E 71120000 mov     byte ptr [esi+1271], bl
0080332D  |.  C786 74120000>mov     dword ptr [esi+1274], 3E8
00803337  |.  899E 78120000 mov     dword ptr [esi+1278], ebx
0080333D  |.  899E 7C120000 mov     dword ptr [esi+127C], ebx
00803343  |.  889E 80120000 mov     byte ptr [esi+1280], bl
00803349  |.  899E 84120000 mov     dword ptr [esi+1284], ebx
0080334F  |.  899E 88120000 mov     dword ptr [esi+1288], ebx
00803355  |.  889E 30120000 mov     byte ptr [esi+1230], bl
0080335B  |.  8BCE          mov     ecx, esi
0080335D  |.  EB 02         jmp     short 00803361
0080335F  |>  33C9          xor     ecx, ecx
00803361  |>  8B4424 20     mov     eax, dword ptr [esp+20]
00803365  |.  C74424 18 FFF>mov     dword ptr [esp+18], -1
0080336D  |.  50            push    eax
0080336E  |.  890D 80062808 mov     dword ptr [8280680], ecx
00803374  |.  E8 77030000   call    008036F0      //进入 N 长的主函数加载
00803379  |.  8B0D 80062808 mov     ecx, dword ptr [8280680]
0080337F  |.  8941 10       mov     dword ptr [ecx+10], eax
00803382  |.  8B15 80062808 mov     edx, dword ptr [8280680]
00803388  |.  8B42 18       mov     eax, dword ptr [edx+18]
0080338B  |.  3BC3          cmp     eax, ebx
0080338D  |.  74 07         je      short 00803396
0080338F  |.  50            push    eax                           ; /hEvent
00803390  |.  FF15 98438200 call    dword ptr [<&KERNEL32.SetEvent>] ; \\SetEvent
00803396  |>  3BEB          cmp     ebp, ebx
00803398  |.  74 07         je      short 008033A1
0080339A  |.  55            push    ebp                           ; /pTopLevelFilter
0080339B  |.  FF15 B8428200 call    dword ptr [<&KERNEL32.SetUnhandl>; \\SetUnhandledExceptionFilter
008033A1  |>  A1 80062808   mov     eax, dword ptr [8280680]
008033A6  |.  8B4C24 10     mov     ecx, dword ptr [esp+10]
008033AA  |.  5E            pop     esi
008033AB  |.  5D            pop     ebp
008033AC  |.  8B40 10       mov     eax, dword ptr [eax+10]
008033AF  |.  5B            pop     ebx
008033B0  |.  64:890D 00000>mov     dword ptr fs:[0], ecx
008033B7  |.  83C4 10       add     esp, 10
008033BA  \\.  C3            retn

//上面函数的调用，看看是不是我分析的类函数^\_^：

005D4E50  /$  55            push    ebp
005D4E51  |.  8BEC          mov     ebp, esp
005D4E53  |.  51            push    ecx
005D4E54  |.  894D FC       mov     dword ptr [ebp-4], ecx
005D4E57  |.  8B45 08       mov     eax, dword ptr [ebp+8]
005D4E5A  |.  50            push    eax    //这里传递了一个参数:
005D4E5B  |.  E8 F0E32200   call    00803250
005D4E60  |.  83C4 04       add     esp, 4
005D4E63  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
```



# 代码段分析与修改记录

## 第一部分：代码定位与修改
以下是需要查看的代码片段：

```
005D4E66  |.  8BE5         mov     esp, ebp  
005D4E68  |.  5D          pop     ebp  
005D4E69  \.  C2 0400     retn    4  
```

再上面看看：

```
005D2C4D  .  A1 549A8600   mov     eax, dword ptr [869A54]  
005D2C52  .  50          push    eax  
005D2C53  .  8B8D 34F2FFFF mov     ecx, dword ptr [ebp-DCC]  
005D2C59  .  E8 F2210000   call    005D4E50    //调用在这里呢~~  
```

上面是到了，`NP` 主要加载了。我修改如下：

```
005D2C4D    B8 55070000   mov     eax, 755  
005D2C52    90          nop  
005D2C53    90          nop  
005D2C54    90          nop  
005D2C55    90          nop  
005D2C56    90          nop  
005D2C57    90          nop  
005D2C58    90          nop  
005D2C59    90          nop  
005D2C5A    90          nop  
005D2C5B    90          nop  
005D2C5C    90          nop  
005D2C5D    90          nop  
```

//改的方法有很多的，这个不是好的方法，但是，这样保证可以的

## 第二部分：Load() 函数分析
我们再来看 `Load()` 函数，这个函数比较特别点，从分析参数来看应该在窗口创建后不远的地方。`NP` 的很多函数很类似，搞不好很容易混淆的。

看到提示，窗口创建成功。

```
//--------------------------------------------------------------------------------------------
005D3004  .  A3 38E4FE07   mov     dword ptr [7FEE438], eax  
005D3009  .  8B4D 14      mov     ecx, dword ptr [ebp+14]  
005D300C  .  51          push    ecx  
005D300D  .  8B55 08      mov     edx, dword ptr [ebp+8]  
005D3010  .  52          push    edx  
005D3011  .  E8 C5E8FFFF   call    005D18DB    ;创建窗口，不用跟进去看就知道  
005D3016  .  83C4 08      add     esp, 8  
005D3019  .  A3 34E4FE07   mov     dword ptr [7FEE434], eax   ;EAX 肯定是 HWND 了。  
005D301E  .  68 A4A18600   push    0086A1A4                 ;  ASCII "> Start window success.  
"  
005D3023  .  68 F8DFFE07   push    07FEDFF8  
005D3028  .  E8 447D0B00   call    0068AD71  
```

//往下看看那个是吧 `HWND` 传递进去的。

```
005D30ED  .  52          push    edx                       ; /ShowState  
005D30EE  .  A1 34E4FE07   mov     eax, dword ptr [7FEE434]     ; |  
005D30F3  .  50          push    eax                       ; |hWnd => NULL  
005D30F4  .  FF15 E0448200 call    dword ptr [<&USER32.ShowWindow>] ; \ShowWindow  
005D30FA  .  8B0D 34E4FE07 mov     ecx, dword ptr [7FEE434]  
005D3100  .  51          push    ecx                       ; /hWnd => NULL  
005D3101  .  FF15 74458200 call    dword ptr [<&USER32.UpdateWindow>; \UpdateWindow  
005D3107  .  8B15 34E4FE07 mov     edx, dword ptr [7FEE434]  
005D310D  .  52          push    edx  
005D310E  .  E8 FF590B00   call    00688B12  ;一般会在显示窗口之后的.  
```

## 第三部分：非 NP 函数调用
//看样子不是 `NP` 函数的

```
00688B12 /$  55          push    ebp  
00688B13 |.  8BEC        mov     ebp, esp  
00688B15 |.  833D 30E4FE07 >cmp    dword ptr [7FEE430], 0  
00688B1C |.  74 0F       je      short 00688B2D  
00688B1E |.  8B45 08     mov     eax, dword ptr [ebp+8]  
00688B21 |.  50          push    eax  
00688B22 |.  8B0D 30E4FE07 mov     ecx, dword ptr [7FEE430]  
00688B28 |.  E8 830A0000   call    006895B0  
00688B2D |>  5D          pop     ebp  
00688B2E \.  C3          retn  
```

//再看看，^_^这个是类函数，找到了:

```
006895B0 /$  55          push    ebp  
006895B1 |.  8BEC        mov     ebp, esp  
006895B3 |.  51          push    ecx  
006895B4 |.  894D FC     mov     dword ptr [ebp-4], ecx  
006895B7 |.  8B45 08     mov     eax, dword ptr [ebp+8]  
006895BA |.  50          push    eax  
006895BB |.  E8 309C1700   call    008031F0  
006895C0 |.  83C4 04     add     esp, 4  
006895C3 |.  8BE5        mov     esp, ebp  
006895C5 |.  5D          pop     ebp  
006895C6 \.  C2 0400     retn    4  
```

## 第四部分：NP 代码分析
// `NP` 的代码一般在程序最末尾，看到 `0080XXXX` 程序的末尾就证明是 `NP` 了，前面提到了由于 `NP` 是 `LIB` 库的形式，所以很多游戏都是一模一样，只有版本不同会不同的。

```
008031F0 /$  8B0D 80062808 mov     ecx, dword ptr [8280680]  
008031F6 |.  85C9        test    ecx, ecx  
008031F8 |.  74 0A       je      short 00803204  
008031FA |.  8B4424 04   mov     eax, dword ptr [esp+4]  
008031FE |.  50          push    eax  
008031FF |.  E8 8C200000   call    00805290  
00803204 \>  C3          retn  
```

//需要的是如下形式的，上面的那中类似形式 `NP` 有好多个

```
00805290 /$  56          push    esi  
00805291 |.  8BF1        mov     esi, ecx  
00805293 |.  57          push    edi  
00805294 |.  68 D0F28700   push    0087F2D0  
00805299 |.  8D7E 28     lea     edi, dword ptr [esi+28]  
0080529C |.  E8 DF580000   call    0080AB80  
008052A1 |.  83C4 04     add     esp, 4  
008052A4 |.  50          push    eax  
008052A5 |.  57          push    edi  
008052A6 |.  E8 55FEFFFF   call    00805100  
008052AB |.  83C4 08     add     esp, 8  
008052AE |.  E8 4D160000   call    00806900  
008052B3 |.  E8 28170000   call    008069E0  
008052B8 |.  8B4424 0C   mov     eax, dword ptr [esp+C]  
008052BC |.  8D8E 64100000 lea     ecx, dword ptr [esi+1064]  
008052C2 |.  50          push    eax  
008052C3 |.  68 12060000   push    612  
008052C8 |.  68 B1080000   push    8B1  
008052CD |.  8946 04     mov     dword ptr [esi+4], eax  
008052D0 |.  E8 CB3D0000   call    008090A0  
008052D5 |.  8B46 08     mov     eax, dword ptr [esi+8]  
008052D8 |.  85C0        test    eax, eax  
008052DA |.  74 44       je      short 00805320  
008052DC |.  8B4E 0C     mov     ecx, dword ptr [esi+C]  
008052DF |.  51          push    ecx  
008052E0 |.  50          push    eax  
008052E1 |.  68 B0F28700   push    0087F2B0  
008052E6 |.  E8 95580000   call    0080AB80  
008052EB |.  83C4 04     add     esp, 4  
008052EE |.  50          push    eax  
008052EF |.  57          push    edi  
008052F0 |.  E8 0BFEFFFF   call    00805100  
008052F5 |.  8B56 0C     mov     edx, dword ptr [esi+C]  
008052F8 |.  8B46 08     mov     eax, dword ptr [esi+8]  
008052FB |.  83C4 10     add     esp, 10  
008052FE |.  52          push    edx  
008052FF |.  50          push    eax  
00805300 |.  E8 5DBDB8FF   call    MGDLL.00391062  ;这里不算，这里是被我给黑掉的。为了研究么办法^_^  
00805305 |.  84C0        test    al, al  
00805307 |.  75 17       jnz     short 00805320  
00805309 |.  68 94F28700   push    0087F294  
0080530E |.  E8 6D580000   call    0080AB80  
00805313 |.  83C4 04     add     esp, 4  
00805316 |.  50          push    eax  
00805317 |.  57          push    edi  
00805318 |.  E8 E3FDFFFF   call    00805100  
0080531D |.  83C4 08     add     esp, 8  
00805320 |>  68 74F28700   push    0087F274  
00805325 |.  E8 56580000   call    0080AB80  
0080532A |.  83C4 04     add     esp, 4  
0080532D |.  50          push    eax  
0080532E |.  57          push    edi  
0080532F |.  E8 CCFDFFFF   call    00805100  
00805334 |.  83C4 08     add     esp, 8  
00805337 |.  5F          pop     edi  
00805338 |.  5E          pop     esi  
00805339 \.  C2 0400     retn    4  
```

//我就不说怎么跳了，地方都告诉了，想怎么跳随你了。

## 第五部分：Check() 函数分析
已经两个函数了，再看第三个了。`Check()` 这个函数我找了好一会才发现的。由于检测的返回正确就是前面提到的那个 `755` 所以，直接查找 `mov eax, 755`

发现如下，下面就是检测的函数体了:

```
//------------------------------------------------------------------------------
008054C0  > \51         push    ecx  
008054C1  .  53          push    ebx  
008054C2  .  8BD9        mov     ebx, ecx  
008054C4  .  8B0D 84062808 mov     ecx, dword ptr [8280684]  
008054CA  .  56          push    esi  
008054CB  .  41          inc     ecx  
008054CC  .  57          push    edi  
008054CD  .  890D 84062808 mov     dword ptr [8280684], ecx  
008054D3  .  8A43 01     mov     al, byte ptr [ebx+1]  
008054D6  .  84C0        test    al, al  
008054D8  .  74 0A       je      short 008054E4  
008054DA  .  5F          pop     edi  
008054DB  .  5E          pop     esi  
008054DC  .  B8 55070000   mov     eax, 755  
```



```asm
008054E1   .  5B             pop     ebx  

008054E2   .  59             pop     ecx  

008054E3   .  C3             retn  

008054E4   >  803B 00        cmp     byte ptr [ebx], 0  

008054E7   .  75 37          jnz     short 00805520  

008054E9   .  833D 84062808 >cmp     dword ptr [8280684], 3  

008054F0   .  73 0A          jnb     short 008054FC  

008054F2   .  5F             pop     edi  

008054F3   .  5E             pop     esi  

008054F4   .  B8 55070000    mov     eax, 755  

008054F9   .  5B             pop     ebx  

008054FA   .  59             pop     ecx  

008054FB   .  C3             retn  

008054FC   >  68 F8F38700    push    0087F3F8  

00805501   .  E8 7A560000    call    0080AB80  

00805506   .  83C4 04        add     esp, 4  

00805509   .  83C3 28        add     ebx, 28  

0080550C   .  50             push    eax  

0080550D   .  53             push    ebx  

0080550E   .  E8 EDFBFFFF    call    00805100  

00805513   .  83C4 08        add     esp, 8  

00805516   .  B8 62020000    mov     eax, 262  

0080551B   .  5F             pop     edi  

0080551C   .  5E             pop     esi  

0080551D   .  5B             pop     ebx  

0080551E   .  59             pop     ecx  

0080551F   .  C3             retn  

00805520   >  A1 70062808    mov     eax, dword ptr [8280670]  

00805525   .  85C0           test    eax, eax  

00805527   .  75 24          jnz     short 0080554D  

00805529   .  68 DCF38700    push    0087F3DC  

0080552E   .  E8 4D560000    call    0080AB80  

00805533   .  83C4 04        add     esp, 4  

00805536   .  83C3 28        add     ebx, 28  

00805539   .  50             push    eax  

0080553A   .  53             push    ebx  

0080553B   .  E8 C0FBFFFF    call    00805100  

00805540   .  83C4 08        add     esp, 8  

00805543   .  B8 8A020000    mov     eax, 28A  

00805548   .  5F             pop     edi  

00805549   .  5E             pop     esi  

0080554A   .  5B             pop     ebx  

0080554B   .  59             pop     ecx  

0080554C   .  C3             retn  

0080554D   >  A1 7C062808    mov     eax, dword ptr [828067C]  

00805552   .  85C0           test    eax, eax  

00805554   .  74 46          je      short 0080559C  

00805556   .  8D4C24 0C      lea     ecx, dword ptr [esp+C]  

0080555A   .  C74424 0C 0000>mov     dword ptr [esp+C], 0  

00805562   .  51             push    ecx                       ; /pExitCode  

00805563   .  50             push    eax                       ; |hProcess => NULL  

00805564   .  FF15 C0428200  call    dword ptr [<&KERNEL32.GetExitCod>; \GetExitCodeProcess  

0080556A   .  85C0           test    eax, eax  

0080556C   .  74 2E          je      short 0080559C  

0080556E   .  817C24 0C 0301>cmp     dword ptr [esp+C], 103  

00805576   .  74 24          je      short 0080559C  

00805578   .  68 C0F38700    push    0087F3C0  

0080557D   .  E8 FE550000    call    0080AB80  

00805582   .  83C4 04        add     esp, 4  

00805585   .  83C3 28        add     ebx, 28  

00805588   .  50             push    eax  

00805589   .  53             push    ebx  

0080558A   .  E8 71FBFFFF    call    00805100  

0080558F   .  83C4 08        add     esp, 8  

00805592   .  B8 6C020000    mov     eax, 26C  

00805597   .  5F             pop     edi  

00805598   .  5E             pop     esi  

00805599   .  5B             pop     ebx  

0080559A   .  59             pop     ecx  

0080559B   .  C3             retn  

0080559C   >  68 ACF38700    push    0087F3AC  

008055A1   .  E8 DA550000    call    0080AB80  

008055A6   .  8B35 E0428200  mov     esi, dword ptr [<&KERNEL32.OpenM>;   kernel32.OpenMutexA  

008055AC   .  83C4 04        add     esp, 4  

008055AF   .  50             push    eax                       ; /MutexName  

008055B0   .  6A 00         push    0                         ; |Inheritable = FALSE  

008055B2   .  68 00001000    push    100000                    ; |Access = 100000  

008055B7   .  FFD6           call    esi                       ; \OpenMutexA  

008055B9   .  85C0           test    eax, eax  

008055BB   .  75 1B          jnz     short 008055D8  

008055BD   .  68 A0F38700    push    0087F3A0  

008055C2   .  E8 B9550000    call    0080AB80  

008055C7   .  83C4 04        add     esp, 4  

008055CA   .  50             push    eax  

008055CB   .  6A 00         push    0  

008055CD   .  68 00001000    push    100000  

008055D2   .  FFD6           call    esi  

008055D4   .  85C0           test    eax, eax  

008055D6   .  74 77          je      short 0080564F  

008055D8   >  8B3D 84418200  mov     edi, dword ptr [<&KERNEL32.Close>;   kernel32.CloseHandle  

008055DE   .  50             push    eax                       ; /hObject  

008055DF   .  FFD7           call    edi                       ; \CloseHandle  

008055E1   .  68 8CF38700    push    0087F38C  

008055E6   .  E8 95550000    call    0080AB80  

008055EB   .  83C4 04        add     esp, 4  

008055EE   .  50             push    eax  

008055EF   .  6A 00         push    0  

008055F1   .  68 00001000    push    100000  

008055F6   .  FFD6           call    esi  

008055F8   .  85C0           test    eax, eax  

008055FA   .  75 1B          jnz     short 00805617  

008055FC   .  68 80F38700    push    0087F380  

00805601   .  E8 7A550000    call    0080AB80  

00805606   .  83C4 04        add     esp, 4  

00805609   .  50             push    eax  

0080560A   .  6A 00         push    0  

0080560C   .  68 00001000    push    100000  

00805611   .  FFD6           call    esi  

00805613   .  85C0           test    eax, eax  

00805615   .  74 0D          je      short 00805624  

00805617   >  50             push    eax  

00805618   .  FFD7           call    edi  

0080561A   .  5F             pop     edi  

0080561B   .  5E             pop     esi  

0080561C   .  B8 55070000    mov     eax, 755  

00805621   .  5B             pop     ebx  

00805622   .  59             pop     ecx  

00805623   .  C3             retn  

00805624   >  FF15 90418200  call    dword ptr [<&KERNEL32.GetLastErr>; [GetLastError  

0080562A   .  50             push    eax  

0080562B   .  68 5CF38700    push    0087F35C  

00805630   .  E8 4B550000    call    0080AB80  

00805635   .  83C4 04        add     esp, 4  

00805638   .  83C3 28        add     ebx, 28  

0080563B   .  50             push    eax  

0080563C   .  53             push    ebx  

0080563D   .  E8 BEFAFFFF    call    00805100  

00805642   .  83C4 0C        add     esp, 0C  

00805645   .  B8 76020000    mov     eax, 276  

0080564A   .  5F             pop     edi  

0080564B   .  5E             pop     esi  

0080564C   .  5B             pop     ebx  

0080564D   .  59             pop     ecx  

0080564E   .  C3             retn  

0080564F   >  FF15 90418200  call    dword ptr [<&KERNEL32.GetLastErr>; [GetLastError  

00805655   .  50             push    eax  

00805656   .  68 38F38700    push    0087F338  

0080565B   .  E8 20550000    call    0080AB80  

00805660   .  83C4 04        add     esp, 4  

00805663   .  83C3 28        add     ebx, 28  

00805666   .  50             push    eax  

00805667   .  53             push    ebx  

00805668   .  E8 93FAFFFF    call    00805100  

0080566D   .  83C4 0C        add     esp, 0C  

00805670   .  B8 94020000    mov     eax, 294  

00805675   .  5F             pop     edi  

00805676   .  5E             pop     esi  

00805677   .  5B             pop     ebx  

00805678   .  59             pop     ecx  

00805679   .  C3             retn  
```

往上找到了，没想到就在`Init()`下面，郁闷早知道就不花那大工夫了。

```asm
008033C0   $  8B0D 80062808  mov     ecx, dword ptr [8280680]  

008033C6   .  85C9           test    ecx, ecx  

008033C8   .  75 03          jnz     short 008033CD  

008033CA   .  33C0           xor     eax, eax  

008033CC   .  C3             retn  
```



```
008033CD   >  E9 EE200000    jmp     008054C0  
```

往上到了类函数：

```
006895D0  /$  55             push    ebp  
006895D1  |.  8BEC           mov     ebp, esp  
006895D3  |.  51             push    ecx  
006895D4  |.  894D FC        mov     dword ptr [ebp-4], ecx  
006895D7  |.  E8 E49D1700    call    008033C0  //实际这里直接来个 mov eax,755 代表执行成功，也算简单.  
006895DC  |.  8BE5           mov     esp, ebp  
006895DE  |.  5D             pop     ebp  
006895DF  \.  C3             retn  
```

我不说怎么跳了，麻烦死了~~大家自己看情况跳吧。。。只要不调用它就是了~~~~~~~

第四个函数 `Auth(DWORD dwAuthCode)`，也是最麻烦的一个，这个验证码我玩的游戏会用到的。

```
00803450  /$  8B0D 80062808  mov     ecx, dword ptr [8280680]  
00803456  |.  85C9           test    ecx, ecx  
00803458  |.  75 03          jnz     short 0080345D  
0080345A  |.  32C0           xor     al, al  
0080345C  |.  C3             retn  
0080345D  |>  8B4424 04      mov     eax, dword ptr [esp+4]  
00803461  |.  50             push    eax  
00803462  |.  E8 19220000    call    00805680  
00803467  \.  C3             retn  
```

真正函数体：

```
00805680  /$  56             push    esi  
00805681  |.  8BF1           mov     esi, ecx  
00805683  |.  803E 00        cmp     byte ptr [esi], 0  
00805686  |.  74 3B          je      short 008056C3  
00805688  |.  57             push    edi  
00805689  |.  8B7C24 0C      mov     edi, dword ptr [esp+C]  
0080568D  |.  57             push    edi  
0080568E  |.  68 14F48700    push    0087F414  
00805693  |.  E8 E8540000    call    0080AB80  
00805698  |.  83C4 04        add     esp, 4  
0080569B  |.  50             push    eax  
0080569C  |.  8D46 28        lea     eax, dword ptr [esi+28]  
0080569F  |.  50             push    eax  
008056A0  |.  E8 5BFAFFFF    call    00805100  
008056A5  |.  83C4 0C        add     esp, 0C  
008056A8  |.  8D8E 64100000  lea     ecx, dword ptr [esi+1064]  
008056AE  |.  57             push    edi  
008056AF  |.  68 16060000    push    616  
008056B4  |.  68 B1080000    push    8B1  
008056B9  |.  E8 E2390000    call    008090A0  
008056BE  |.  5F             pop     edi  
008056BF  |.  5E             pop     esi  
008056C0  |.  C2 0400        retn    4  
008056C3  |>  32C0           xor     al, al  
008056C5  |.  5E             pop     esi  
008056C6  \.  C2 0400        retn    4  
```

地方已经给大家了，大家看情况来做吧！是在那改也很容易的。

最后一个 `Exit()`，么什么了，这个函数跳不跳都不影响的。

```
008034B0  /$  8B0D 80062808  mov     ecx, dword ptr [8280680]  
008034B6  |.  85C9           test    ecx, ecx  
008034B8  |.  75 03          jnz     short 008034BD  
008034BA  |.  32C0           xor     al, al  
008034BC  |.  C3             retn  
008034BD  |>  53             push    ebx  
008034BE  |.  56             push    esi  
008034BF  |.  E8 BC220000    call    00805780  
008034C4  |.  8B0D 80062808  mov     ecx, dword ptr [8280680]  
008034CA  |.  8AD8           mov     bl, al  
008034CC  |.  85C9           test    ecx, ecx  
008034CE  |.  8BF1           mov     esi, ecx  
008034D0  |.  74 0E          je      short 008034E0  
008034D2  |.  E8 19000000    call    008034F0  
008034D7  |.  56             push    esi  
008034D8  |.  E8 99D7FDFF    call    007E0C76  
008034DD  |.  83C4 04        add     esp, 4  
008034E0  |>  8AC3           mov     al, bl  
008034E2  |.  5E             pop     esi  
008034E3  |.  C705 80062808 >mov     dword ptr [8280680], 0  
008034ED  |.  5B             pop     ebx  
008034EE  \.  C3             retn  
```

也许大家会说少个回调函数，说句很不好意思的话，那个玩意我还么怎么搞明白的。大家去试试吧~~

按这 5 步做下来，基本上 NP 会不见的。而且，有的游戏就完全可以玩了，有的不知道为什么，竟然会掉线~~

郁闷中~~~~~~~~

![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif") ![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif") 

大家就留着自己用吧，请不要转载了~~~![](img/32b12ba0f4c0e6a8843aaa06ba62fbdd.jpg "皱眉") 

免的以后不好使了哈哈~~~~~~~~~

-By EasyStudy  
2008 年 07 月 20 日 凌晨

类是这样的。

```cpp
class CNPGameLib
{
public:
explicit CNPGameLib(LPCTSTR lpszGameName)
{
#ifdef NO_GAMEGUARD
return;
#endif
PreInitNPGameMon(lpszGameName);
}

~CNPGameLib()
{
#ifdef NO_GAMEGUARD
return;
#endif
CloseNPGameMon();
}

public:
DWORD Init()
{
#ifdef NO_GAMEGUARD
return NPGAMEMON_SUCCESS;
#endif
return (InitNPGameMon());
}

void SetHwnd(HWND hWnd)
{
#ifdef NO_GAMEGUARD
return;
#endif
SetHwndToGameMon(hWnd);
}

DWORD Check()
{
#ifdef NO_GAMEGUARD
return NPGAMEMON_SUCCESS;
#endif
return (CheckNPGameMon());
}

DWORD Send(LPCTSTR lpszUserId)
{
#ifdef NO_GAMEGUARD
return NPGAMEMON_SUCCESS;
#endif
return (SendUserIDToGameMon(lpszUserId));
}

LPCSTR GetInfo()
{
#ifdef NO_GAMEGUARD
return NULL;
#endif
return (GetInfoFromGameMon());
}

DWORD Auth(DWORD dwAuth)
{
#ifdef NO_GAMEGUARD
return NPGAMEMON_SUCCESS;
#endif
return (SendCSAuthToGameMon(dwAuth));
}

DWORD Auth2(PGG_AUTH_DATA pAuth)
{
#ifdef NO_GAMEGUARD
return NPGAMEMON_SUCCESS;
#endif
return (SendCSAuth2ToGameMon(pAuth));
}

    void RunFAQ(DWORD dwErrcode)
    {
#ifdef NO_GAMEGUARD
return;
#endif
        RunFAQWebPage(dwErrcode);
    }

BOOL ResAuth(LPCTSTR lpszFilename, bool bCheckFilename = true)
{
#ifdef NO_GAMEGUARD
return TRUE;
#endif
return (ResourceAuth(lpszFilename, bCheckFilename));
}

private:
// No copies
CNPGameLib(const CNPGameLib &rhs);
CNPGameLib &operator=(const CNPGameLib &rhs);
};
```

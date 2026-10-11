
# 扫雷辅助工具完全制作

> 原文：[www.pediy.com/kssd/pediy09/pediy09-337.htm](https://www.pediy.com/kssd/pediy09/pediy09-337.htm)
>
> 【文章标题】: 扫雷辅助工具完全制作  
> 【文章作者】: fqucuo  
> 【作者邮箱】: fqucuo@163.com  
> 【作者 QQ 号】: 389990968  
> 【软件名称】: `UnWinmine`  
> 【下载地址】: 联系我  
> 【加壳方式】: 无  
> 【保护方式】: 无  
> 【编写语言】: `Win32ASM`  
> 【使用工具】: `PEiD`, `OD`, `WinHex`  
> 【操作平台】: `WinXp Sp2`  
> 【软件介绍】: 扫雷游戏的辅助工具，可以秒杀的哦！  
> 【作者声明】: 初学反汇编，就拿这个小程序试手，估计很多搞定了吧，不过作为初学者还是要搞一下！
>
> \--------------------------------------------------------------------------------
>
> 【详细过程】
>
> 闲话我会在后面说（因为没想好说什么），先来看过程吧！
>
> 找到 `Winmine`（扫雷游戏，别告诉我你不知道它在哪儿哦），`PEiD` 一下，获得第一步信息，`VC++7.0` 编写（`VC++6.0` 以上的版本不是很清楚）。
>
> 先谈下思路，打开 `Winmine.exe` 后，我们知道雷区是由 `X * Y` 组成的正方形区域，每次点击都会有改变其目标方块当前状态，那么势必就会调用 `GetDC`, `ReleaseDC`, `Bitblt` 等 GDI 函数，好，顺着这个思路继续。
>
> 打开 `OD`，载入 `Winmine`，查找->当前模块中的名称（标签），找到 `Bitblt`，`bpx Bitblt`，将所有调用都下断点，有人要问了，为什么就一定是 `Bitblt` 而不是其他的 API 呢？我只能说一般来说，使用 `GetDC` 直接画的不多，多数都是使用这种双缓冲方式绘图，能有效防止闪屏，所以我们直接就找 `Bitblt` 了。
>
> `F9` 跑起来，`OD` 这时候断到第一个 `Bitblt`，上下翻翻。。。，呵呵 是不是另一个 `Bitblt` 也看到了（我是喜欢上下翻翻的），因为总共就 2 个 `Bitblt` 调用。
>
> 对比我们可以看到，两个 `Bitblt` 上下有明显的区别，
>
> 上面的 `Bitblt`：
>
> ```
> 01002646   /$  56           push    esi
> 01002647   |.  FF35 245B0001 push    dword ptr [1005B24]                ; /hWnd = 00060598 ('扫雷',class='扫雷')
> 0100264D   |.  FF15 2C110001 call    dword ptr [<&USER32.GetDC>]        ; \GetDC
> 01002653   |.  8B4C24 0C    mov     ecx, dword ptr [esp+C]
> 01002657   |.  68 2000CC00  push    0CC0020                          ; /ROP = SRCCOPY
> 0100265C   |.  8BF0         mov     esi, eax                       ; |
> 0100265E   |.  8B4424 0C    mov     eax, dword ptr [esp+C]         ; |
> 01002662   |.  8BD1         mov     edx, ecx                       ; |
> 01002664   |.  6A 00        push    0                              ; |YSrc = 0
> 01002666   |.  C1E2 05      shl     edx, 5                         ; |
> 01002669   |.  0FBE9402 4053>movsx   edx, byte ptr [edx+eax+1005340]  ; |
> 01002671   |.  6A 00        push    0                              ; |XSrc = 0
> 01002673   |.  83E2 1F      and     edx, 1F                        ; |
> 01002676   |.  FF3495 205A00>push    dword ptr [edx*4+1005A20]      ; |hSrcDC
> 0100267D   |.  C1E1 04      shl     ecx, 4                         ; |
> 01002680   |.  6A 10        push    10                             ; |Height = 10 (16.)
> 01002682   |.  6A 10        push    10                             ; |Width = 10 (16.)
> 01002684   |.  83C1 27      add     ecx, 27                        ; |
> 01002687   |.  C1E0 04      shl     eax, 4                         ; |
> 0100268A   |.  51           push    ecx                            ; |YDest
> 0100268B   |.  83E8 04      sub     eax, 4                         ; |
> 0100268E   |.  50           push    eax                            ; |XDest
> 0100268F   |.  56           push    esi                            ; |hDestDC
> 01002690   |.  FF15 5C100001 call    dword ptr [<&GDI32.BitBlt>]   ; \BitBlt // 第一个断点
> 01002696   |.  56           push    esi                            ; /hDC
> 01002697   |.  FF35 245B0001 push    dword ptr [1005B24]           ; |hWnd = 00060598 ('扫雷',class='扫雷')
> 0100269D   |.  FF15 28110001 call    dword ptr [<&USER32.ReleaseDC>] ; \ReleaseDC
> 010026A3   |.  5E           pop     esi
> 010026A4   \.  C2 0800      retn    8
> ```
>
> 下面的 `Bitblt`：
>
> ```
> 010026A7   /$  55           push    ebp
> 010026A8   |.  8BEC        mov     ebp, esp
> 010026AA   |.  83EC 0C     sub     esp, 0C
> 010026AD   |.  33C0        xor     eax, eax
> 010026AF   |.  40          inc     eax
> 010026B0   |.  3905 38530001 cmp     dword ptr [1005338], eax
> 010026B6   |.  C745 F8 37000>mov     dword ptr [ebp-8], 37
> 010026BD   |.  8945 F4     mov     dword ptr [ebp-C], eax
> 010026C0   |.  7C 68       jl      short 0100272A
> 010026C2   |.  53          push    ebx
> 010026C3   |.  56          push    esi
> 010026C4   |.  BB 60530001  mov     ebx, 01005360
> 010026C9   |>  33F6        /xor     esi, esi
> 010026CB   |.  46          |inc     esi
> 010026CC   |.  3935 34530001 |cmp     dword ptr [1005334], esi            ; 比较 1
> 010026D2   |.  C745 FC 0C000>|mov     dword ptr [ebp-4], 0C
> 010026D9   |.  7C 38       |jl      short 01002713
> 010026DB   |>  68 2000CC00  |/push    0CC0020                        ; /ROP = SRCCOPY
> 010026E0   |.  33C0        ||xor     eax, eax                     ; |
> 010026E2   |.  8A0433      ||mov     al, byte ptr [ebx+esi]         ; |
> 010026E5   |.  6A 00       ||push    0                              ; |YSrc = 0
> 010026E7   |.  6A 00       ||push    0                              ; |XSrc = 0
> 010026E9   |.  83E0 1F     ||and     eax, 1F                        ; |
> 010026EC   |.  FF3485 205A00>||push    dword ptr [eax*4+1005A20]    ; |hSrcDC
> 010026F3   |.  6A 10       ||push    10                             ; |Height = 10 (16.)
> 010026F5   |.  6A 10       ||push    10                             ; |Width = 10 (16.)
> 010026F7   |.  FF75 F8     ||push    dword ptr [ebp-8]            ; |YDest
> 010026FA   |.  FF75 FC     ||push    dword ptr [ebp-4]            ; |XDest
> 010026FD   |.  FF75 08     ||push    dword ptr [ebp+8]            ; |hDestDC
> 01002700   |.  FF15 5C100001 ||call    dword ptr [<&GDI32.BitBlt>]   ; \BitBlt // 第二个断点,先被断下的
> 01002706   |.  8345 FC 10  ||add     dword ptr [ebp-4], 10
> 0100270A   |.  46          ||inc     esi
> 0100270B   |.  3B35 34530001 ||cmp     esi, dword ptr [1005334]          ; 比较 2
> 01002711   |.^ 7E C8       |\jle     short 010026DB              ; 跳转 2
> 01002713   |>  FF45 F4     |inc      dword ptr [ebp-C]
> 01002716   |.  8B45 F4     |mov     eax, dword ptr [ebp-C]
> 01002719   |.  8345 F8 10  |add     dword ptr [ebp-8], 10
> 0100271D   |.  83C3 20     |add     ebx, 20
> 01002720   |.  3B05 38530001 |cmp     eax, dword ptr [1005338]
> 01002726   |.^ 7E A1       \jle     short 010026C9                ; 跳转 1
> 01002728   |.  5E          pop     esi
> 01002729   |.  5B          pop     ebx
> 0100272A   |>  C9          leave
> 0100272B   \.  C2 0400     retn    4
> ```
>
> 可以看到在先被断下来的 `Bitblt` 有明显的判断和跳转，我们可以这样理解：在一个 `X*Y` 的矩阵（雷区）中，分别处理每个元素的方式，常理是一个嵌套循环。好，为了验证这个道理，我们将带“循环嵌套”的 `Bitblt` 断点禁用，然后 `F9` 跑起来，然后在扫雷窗口中随便点一个方块，被断下，再跑，这个时候再去点扫雷窗口就不行了，当鼠标经过的时候就会断下，可以初步判断出这个 `Bitblt` 是用来重绘的，那么我们就取消这个 `Bitblt` 的断点，启用带“循环嵌套的”`Bitblt` 断点，不过这个时候再跑将处于无限断下状态。o(∩\_∩)o...
>
> 当时也挺郁闷的，不过看到这段反汇编不是很长，就决定尝试推一下源码，好，说干就干！
>
> ```
> 010026AD   |.  33C0        xor     eax, eax
> 010026AF   |.  40          inc     eax
> 010026B0   |.  3905 38530001 cmp     dword ptr [1005338], eax
> ```
>
> 在 `OD` 内存中，我们可以查到 `1005338` 地址内容为：`09 00 00 00`，暂时不知道干什么的，先不管它
>
> ```
> 010026B6   |.  C745 F8 37000>mov     dword ptr [ebp-8], 37
> 010026BD   |.  8945 F4     mov     dword ptr [ebp-C], eax
> 010026C0   |.  7C 68       jl      short 0100272A
> ```
>
> `[ebp - 8] = 37 [ebp - c] = eax = 1;` 暂时也不清楚这两个变量是干什么的，继续
>
> ```
> 010026C4   |.  BB 60530001 mov     ebx, 01005360
> ```
>
> `ebx = 1005360`，可以明显看出是个地址，先在内存窗口中看一下，发现了一些一大块的 `0F 10 8F` 等组成的内存值
>
> ```
> 01005330   0A 00 00 00 09 00 00 00 09 00 00 00 00 00 00 00   ................
> 01005340   10 10 10 10 10 10 10 10 10 10 10 0F 0F 0F 0F 0F  
> 01005350   0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F  
> ```



```
01005360  10 8F 8F 0F 0F 0F 0F 0F 0F 0F 10 0F 0F 0F 0F 0F   
01005370  0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F   
01005380  10 0F 0F 0F 0F 0F 0F 0F 0F 0F 10 0F 0F 0F 0F 0F   
01005390  0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F   
010053A0  10 0F 0F 0F 8F 0F 0F 0F 0F 0F 10 0F 0F 0F 0F 0F  ?  
010053B0  0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F   
010053C0  10 0F 0F 0F 0F 0F 8F 0F 0F 0F 10 0F 0F 0F 0F 0F  ?  
010053D0  0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F   
010053E0  10 8F 0F 8F 8F 0F 0F 0F 0F 0F 10 0F 0F 0F 0F 0F  ?  
010053F0  0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F   
01005400  10 0F 8F 0F 0F 0F 0F 0F 0F 0F 10 0F 0F 0F 0F 0F  ?  
01005410  0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F   
01005420  10 0F 0F 0F 0F 0F 0F 0F 0F 0F 10 0F 0F 0F 0F 0F   
01005430  0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F   
01005440  10 0F 0F 8F 0F 0F 0F 0F 0F 0F 10 0F 0F 0F 0F 0F  ?  
.......
（其实这就是双缓冲对应方块的内存，往下我们再来证明这一点）
```

通过 `OD` 的跳转标记我们可以明显的看到循环嵌套的过程，并且根据矩阵 `X * Y` 的理论，我们可以得知，内层循环是一个 `x` 轴（列坐标），外层循环是个 `y` 轴（行坐标），具体是使用了 `for` 循环或者是 `while` 循环我们不必关心，就当他是 `for` 循环好了，实现功能为主，我们不妨定义两个变量 `x` 和 `y`，并且通过

```
010026C9  |>  33F6           /xor     esi, esi
010026CB  |.  46             |inc     esi                                             ;x = 1，x++
```

```
010026AD  |.  33C0           xor     eax, eax
010026AF  |.  40             inc     eax                                               ;y = 1
......
01002713  |>  FF45 F4        |inc     dword ptr [ebp-C]                                 ;y++
01002716  |.  8B45 F4        |mov     eax, dword ptr [ebp-C]
```

```
01002720  |.  3B05 38530001 |cmp     eax, dword ptr [1005338]                          ; 比较 y
01002726  |.^ 7E A1          \jle     short 010026C9                                     ; 跳转 y
```

```
0100270B  |.  3B35 34530001 ||cmp     esi, dword ptr [1005334]                         ; 比较 x
01002711  |.^ 7E C8          |\jle     short 010026DB                                     ; 跳转 x
```

我们得知两个循环都是从 `1` 开始，步长为 `1`，`jle` 推出比较运算符为 `<=`, 内存窗口查找 `[1005338]` 与 `[1005334]` 的值，得到

```
[1005338]  = 9 ; y 轴
[1005334]  = 9 ; x 轴
```

可以知道矩阵的 `y` 轴界限为 `9`，`x` 轴界限为 `9`，也知道了这两个内存地址是用来存放矩阵的高和宽的值，那么我们就有（开始我是用 `Win32MASM` 环境下写的，为了直观我用 `C` 语法表示）

定义变量 `int nHeight = [1005338], nWidth = [1005334];`

```
int nHeight, nWidth; // 这里不赋初值是因为现在只是反汇编, 后面写工具的时候会从内存中读初值
for(int y = 1; y <= nHeight; y++)
{
  for(int x = 1; x <= nWidth; x++)
  {
    ;
  }
}
```

我们继续分析循环体内部的代码

```
010026DB  |>  68 2000CC00    |/push    0CC0020                  ; /ROP = SRCCOPY
010026E0  |.  33C0           ||xor     eax, eax                   ; |
010026E2  |.  8A0433         ||mov     al, byte ptr [ebx+esi]     ; |
010026E5  |.  6A 00          ||push    0                          ; |YSrc = 0
010026E7  |.  6A 00          ||push    0                          ; |XSrc = 0
010026E9  |.  83E0 1F        ||and     eax, 1F                    ; |
010026EC  |.  FF3485 205A00>||push    dword ptr [eax*4+1005A20]    ; |hSrcDC
010026F3  |.  6A 10          ||push    10                         ; |Height = 10 (16.)
010026F5  |.  6A 10          ||push    10                         ; |Width = 10 (16.)
010026F7  |.  FF75 F8        ||push    dword ptr [ebp-8]           ; |YDest
010026FA  |.  FF75 FC        ||push    dword ptr [ebp-4]           ; |XDest
010026FD  |.  FF75 08        ||push    dword ptr [ebp+8]           ; |hDestDC
01002700  |.  FF15 5C100001 ||call    dword ptr [<&GDI32.BitBlt>] ; \BitBlt // 第二个断点
01002706  |.  8345 FC 10     ||add     dword ptr [ebp-4], 10
```

这是内层循环的代码

可以看到主要就是调用了一个 `Bitblt`，现在我们还原 `Bitblt`，通过查看 `MSDN`，可以得知其声明方式

```
BOOL BitBlt(
HDC hdcDest, // handle to destination DC
int nXDest,  // x-coord of destination upper-left corner
int nYDest,  // y-coord of destination upper-left corner
int nWidth,  // width of destination rectangle
int nHeight, // height of destination rectangle
HDC hdcSrc,  // handle to source DC
int nXSrc,   // x-coordinate of source upper-left corner
int nYSrc,   // y-coordinate of source upper-left corner
DWORD dwRop  // raster operation code
);
```

那么我们就有：

```
Bitblt(hDestDC,//目的 DC
          XDest, // 目的 x 坐标
          YDest, // 目的 y 坐标
          10, // 
          10, // 重绘区域的高和宽
          hSrcDC, // 源 DC
          0,
          0,
          SRCCOPY);// 指定操作方式
```

这里我们可以得到 `[ebp - 4] = XDest, [ebp - 8] = YDest`,那么我们可以解释

```
010026D2  |.  C745 FC 0C000> |mov     dword ptr [ebp-4], 0C
010026B6  |.  C745 F8 37000> |mov     dword ptr [ebp-8], 37
```

这两句估计是用来确定有效写入区域的 `x` 轴与 `y` 轴坐标的,(因为要算上边框和显示时间和排雷数区域的高度)

这时我们在定义变量 `XDest` 和 `YDest`

```
int XDest = 0, YDest = 0x37;   
int nHeight, nWidth;  
for(int y = 1; y <= nHeight; y++)
{
  XDest  = 0x0C;
  for(int x = 1; x <= nWidth; x++)
  {
```



```
Bitblt(hDestDC, XDest, YDest, 0x10, 0x10, hSrcDC, 0, 0, SRCCOPY);

// 01002706  .|  8345 FC 10    ||add      dword ptr [ebp-4], 10

XDest += 0x10;

//  01002719  .|  8345 F8 10    |add      dword ptr [ebp-8], 10

YDest += 0x10;
```

基本上就这些,下面我们来看关键的 `ebx`

上面我们知道 `ebx` 用来存储缓冲区首地址,其实整个矩阵首地址并不是 `1005360`,而是 `1005340`,我们可以试想一下,矩阵内部是有效数据,但是对于一个矩阵游戏来说,其外层会有一圈定义值限制其范围,通过内存我们可以得知其定义为 `0x10`,这个并不是重点,知道即可.

那么 `ebx` 到底是做什么的呢?

```
010026E0  .|  33C0           ||xor      eax, eax                       ; |
010026E2  .|  8A0433         ||mov      al, byte ptr [ebx+esi]         ; |
010026E5  .|  6A 00          ||push     0                               ; |YSrc = 0
010026E7  .|  6A 00          ||push     0                               ; |XSrc = 0
010026E9  .|  83E0 1F        ||and      eax, 1F                        ; |
010026EC  .|  FF3485 205A00> ||push     dword ptr [eax*4+1005A20]      ; |hSrcDC
```

通过反汇编我们可以看到它是在做一个位与查表的方式获取源资源 DC 句柄,也就得到了我们平时玩游戏是提示数字、插旗、？号等等，这里我们不去深究，知道即可(把地雷 DC 换到数字 DC 里也是挺有意思的...)

有：

```
0100271D  .|  83C3 20        |add      ebx, 20

010026E2  .|  8A0433         ||mov      al, byte ptr [ebx+esi]         ; |
```

我们得知每次地址+`0x20`，那么我们就定义地址变量 `dwBaseAddress = 1005360 = ebx`, 内层循环每次 +x(+esi)，外层循环每次+`0x20`

```
int XDest = 0, YDest = 0;   

int nHeight, nWidth; 

DWORD dwBaseAddress = 0x1005360;// 这里我们就用一个地址常量了

DWORD dwTempBaseAddress = 0; //临时变量

for(int y = 1; y <= nHeight; y++)

{

  XDest  = 0x0C; 

  dwTempBaseAddress = dwBaseAddress; 

  for(int x = 1; x <= nWidth; x++)

  {

    dwTempBaseAddress += x; 

    ...// 其源资源 DC 查表过程与本例无关，所以省去

    Bitblt(hDestDC, XDest, YDest, 0x10, 0x10, hSrcDC, 0, 0, SRCCOPY); 

    // 01002706  .|  8345 FC 10    ||add      dword ptr [ebp-4], 10

    XDest += 0x10; 

  }

  // 01002719  .|  8345 F8 10    |add      dword ptr [ebp-8], 10

  YDest += 0x10; 

  dwBaseAddress  += 0x20; 

}
```

呼。。。反汇编分析基本上就这些了

这个时候我们可以新开一个 `Winmine`，打开 `WinHex` 查看内存，找到地址 `1005360`，可以看到很多 `10 0F 8F` 等组成的数值，这时候我们每点一个方块，就看一次内存，我们可以得到

*   `10` 为边框
*   `8F` 为有雷且并未点取的
*   `0F` 为无雷且并未点取的
*   `4X` 为点取了无雷区域，x 为 1-9 的数值，也就是游戏中的提示数字
*   `8E` 为插旗并有雷 `0E` 为插旗但无雷
*   `8D` 为问号并有雷 `0D` 为问号但无雷
*   ....

好了，知道这些就够了，我们现在就可以做插旗和秒杀的功能了,具体的实现代码偶就不多说了

--------------------------------------------------------------------------------

### 【经验总结】

惭愧啊，学逆向 2 个月了，在看雪偶还是第一次发帖，无奈本人水平太菜，还望高手多多指教啊~，千万别扔鸡蛋哦！

希望高手、和我一样的菜菜一起进步，我的 QQ：389990968 

在实现代码中我用的是模拟鼠标操作，不知道有没有更好的办法，过程中一定会有不正确的地方，还望高手指点！

感谢看雪提供这么好的平台，希望越办越好！没了。。。

### 【版权声明】

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 12 月 03 日 22:50:33 

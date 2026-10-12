
# 三国志 5 日文加强版去除 256 色限制

> 原文：[`www.pediy.com/kssd/pediy12/98366.html`](https://www.pediy.com/kssd/pediy12/98366.html)

先备份 `San5W95.exe`

用 OD 载入

命令行下断点 `bp MessageBoxA`

F9 运行到断点，F2 清除断点

`Alt+F9` 执行返回到用户代码

在弹出的窗口点确定

按两次 F7 返回到上一级函数，此时 `EIP=004CF1DA`

```
004CF190   .  6A 00         push    0                           ; /hWnd = NULL
004CF192   .  8935 804A5200 mov     dword ptr [524A80], esi     ; |
004CF198   .  FF15 10965200 call    dword ptr [<&USER32.GetDC>] ; \GetDC
004CF19E   .  8BE8          mov     ebp, eax
004CF1A0   .  85ED         test    ebp, ebp
004CF1A2   .  74 45        je      short 004CF1E9
004CF1A4   .  6A 26        push    26                            ; /Index = RASTERCAPS
004CF1A6   .  8B3D E4935200 mov     edi, dword ptr [<&GDI32.GetDevic>; |GDI32.GetDeviceCaps
004CF1AC   .  55          push    ebp                           ; |hDC
004CF1AD   .  FFD7        call    edi                           ; \GetDeviceCaps
004CF1AF   .  8BD8        mov     ebx, eax                      ;GetDeviceCaps(dc, RASTERCAPS)
004CF1B1   .  6A 68        push    68                            ; /Index = SIZEPALETTE
004CF1B3   .  80E7 01     and     bh, 1                         ; GetDeviceCaps(dc, RASTERCAPS) & RC_PALETTE
004CF1B6   .  55          push    ebp                           ; |hDC
004CF1B7   .  FFD7        call    edi                           ; \GetDeviceCaps
004CF1B9   .  8BF8        mov     edi, eax                      ;系统调色板中的入口数目
004CF1BB   .  55          push    ebp                           ; /hDC
004CF1BC   .  6A 00        push    0                           ; |hWnd = NULL
004CF1BE   .  FF15 0C965200 call    dword ptr [<&USER32.ReleaseDC>] ; \ReleaseDC
004CF1C4   .  84FF        test    bh, bh                        ;是否使用调色板
004CF1C6   .  74 08        je      short 004CF1D0
004CF1C8   .  81FF 00010000 cmp     edi, 100
004CF1CE   .  74 19        je      short 004CF1E9
004CF1D0   > \68 105F5100  push    00515F10                    ;弹出窗口显示的字符串地址
004CF1D5   .  E8 AB2FF3FF  call    00402185                    ;弹出窗口
004CF1DA   .  83C4 04      add     esp, 4
004CF1DD   .  33C0         xor     eax, eax
004CF1DF   .  5D          pop     ebp
004CF1E0   .  5F          pop     edi
004CF1E1   .  5E          pop     esi
004CF1E2   .  5B          pop     ebx
004CF1E3   .  83C4 30      add     esp, 30
004CF1E6   .  C2 0800      retn    8
004CF1E9   >  8D4424 10     lea     eax, dword ptr [esp+10]      ;应该跳到这里
```

往上查看代码，发现有两处跳转

```
004CF1C6   .  74 08        je      short 004CF1D0
004CF1C8   .  81FF 00010000 cmp     edi, 100
004CF1CE   .  74 19        je      short 004CF1E9
```

第一处跳或第二处不跳，都会弹出窗口

因此，我们可以直接将第一处强行跳到第二处要跳的地址(可这样改的依据见下面说明)：

`004CF1C6` 处： `je short 004CF1D0`

改为： `jmp short 004CF1E9`

选中后右键选复制到可执行文件，保存覆盖原文件。

`RC_PALETTE` is set in palettized color modes and clear in nonpalettized modes. Generally speaking, the `RC_PALETTE` bit is set in 8-bit color modes and clear in 4-bit and 24-bit color modes. The `RC_PALETTE` bit is also clear if the adapter is running in 16-bit color ("high color") mode, which for most applications produces color output every bit as good as true color. Don't make the mistake some programmers have made and rely on bit counts to tell you whether to use a palette. As sure as you do, you'll run across an oddball video adapter that defies the normal conventions and fools your application into using a palette when a palette isn't needed or not using a palette when a palette would help.

What happens if you ignore the `RC_PALETTE` setting and use a logical palette regardless of color depth? The application will still work because the palette manager works even on nonpalettized devices. If `RC_PALETTE` is 0, palettes can still be created and selected into device contexts, but calls to `RealizePalette` do nothing. `PALETTEINDEX` values are dereferenced and converted into RGB colors in the logical palette, and `PALETTERGB` values are simply treated as if they were standard RGB color values. `OnQueryNewPalette` and `OnPaletteChanged` aren't called because no `WM_QUERYNEWPALETTE` and `WM_PALETTECHANGED` messages are sent. As explained in an excellent article, "The Palette Manager: How and Why," available on the Microsoft Developer Network (MSDN), "The goal is to allow applications to use palettes in a device-independent fashion and to not worry about the actual palette capabilities of the device driver."

摘自： `http://www.moon-soft.com/doc/3124.htm`

如果是三国志 5 简体中文版的话，这样修改后就可以正常运行。

对于三国志 5 日文视窗加强版，仅这样修改还不够

`Ctrl+F2` 重新载入，F9 运行弹出错误，内存地址 `10101010` 不可读，此时堆栈内容为一堆 `10101010`。

（我们知道这是在修改 256 色才引起问题，因此可以直接上网查找与 256 色相关的 API 后下断点，下面用的最笨的方法）

按 F12 暂停，堆栈停在 `ESP` 处，往下拉看到第一个指向代码段的地址：

```
0012FBA4   004E7D21  San5W95.004E7D21
```

选中按回车，来到 CPU 窗口，在此处下断点

```
004E7D21  |.  F3:AB         rep     stos dword ptr es:[edi]
```

`Ctrl+F2` 重新载入，F9 运行，断在了 `004E7D21`，清除断点，并在下一条指令（`004E7D23`）下断点

F9 运行，弹出错误窗口，说明确实是 `004E7D21` 处指行有问题。

往上查看代码

```
004E7CE4  |.  8B4C24 04     mov     ecx, dword ptr [esp+4]
004E7CF3  |.  8BF9          mov     edi, ecx
```

`edi` 的值是由外部传入的第一个参数。直接在这段函数开始处 `004E7CE0` 下断点

重新载入，运行到断点

堆栈窗口：

```
0012FE84   004E6158  返回到 San5W95.004E6158 来自 San5W95.004E7CE0
0012FE88   0012FAA0
0012FE8C   00000010
0012FE90   0004D800
0012FE94   00000004
0012FE98   004E2FEB  返回到 San5W95.004E2FEB 来自 San5W95.004E6136
0012FE9C   00000280
0012FEA0   000001F0
0012FEA4   0012FAA0
0012FEA8   00000003
0012FEAC   004E2CDE  返回到 San5W95.004E2CDE 来自 San5W95.004E2FC2
```

在堆栈顶部按回车，CPU 窗口来到函数返回地址 `004E6158`

`[esp+4]=0012FAA0` 即为传给 `edi` 的值(对应 `004E6150` 处的 `push dword ptr [ebp+10]`)

从 OD 的分析（或从 `ebp+10`）可以看出，是上一个函数传入的第三个参数。

```
004E6136  /$  55          push    ebp
004E6137  |.  8B4424 0C     mov     eax, dword ptr [esp+C]
004E613B  |.  0FAF4424 08   imul    eax, dword ptr [esp+8]
004E6140  |.  837C24 14 07  cmp     dword ptr [esp+14], 7
004E6145  |.  8BEC          mov     ebp, esp
004E6147  |.  50          push    eax
004E6148  |.  75 04        jnz     short 004E614E
004E614A  |.  6A 00        push    0
004E614C  |.  EB 02        jmp     short 004E6150
004E614E  |>  6A 10        push    10
004E6150  |>  FF75 10       push    [arg.3]                  ;  push dword ptr [ebp+10]
004E6153  |.  E8 881B0000   call    004E7CE0
004E6158  |.  8BE5          mov     esp, ebp
```

堆栈窗口往下拉，选中第二个返回地址 `004E2FEB` 按回车

```
004E2FC2  /$  8B0D 68805200 mov     ecx, dword ptr [528068]
004E2FC8  |.  A1 98735200   mov     eax, dword ptr [527398]
004E2FCD  |.  50          push    eax
004E2FCE  |.  8B048D 705752>mov     eax, dword ptr [ecx*4+525770]
004E2FD5  |.  FF35 B05B5200 push    dword ptr [525BB0]
004E2FDB  |.  83C0 10       add     eax, 10
004E2FDE  |.  50          push    eax
004E2FDF  |.  FF348D 985B52>push    dword ptr [ecx*4+525B98]
004E2FE6  |.  E8 4B310000   call    004E6136
004E2FEB  |.  83C4 10       add     esp, 10
```

`call 004E6136` 往上数第三个 push

```
004E2FD5  |.  FF35 B05B5200 push    dword ptr [525BB0]
```

选中这一行，CPU 提示窗口显示

`ds:[00525BB0]=0012FAA0`

说明这个值正是传给先前 `edi` 的

下硬件写入断点： `hw 00525BB0`

重新载入，运行，断在 `004E2799`， 清除硬件断点： `hd 00525BB0`

```
004E276C  /$  8B4424 04     mov     eax, dword ptr [esp+4]
004E2770  |.  81EC 28040000 sub     esp, 428
004E2776  |.  3B05 74655100 cmp     eax, dword ptr [516574]
004E277C  |.  7D 1B        jge     short 004E2799
004E277E  |.  8D4C24 00     lea     ecx, dword ptr [esp]
004E2782  |.  A3 68805200   mov     dword ptr [528068], eax
004E2787  |.  51          push    ecx
004E2788  |.  FF3485 AC5B52>push    dword ptr [eax*4+525BAC]
004E278F  |.  E8 63370000   call    004E5EF7
```



# 逆向分析与调试笔记

## 1. 获取 GDI 对象信息

在地址 `004E2794` 处：
```
004E2794  |.  A3 B05B5200   mov     dword ptr [525BB0], eax  
004E2799  |.\81C4 28040000  add     esp, 428  
```
显然要检查 `call 004E5EF7` 的返回值。

进入 `004E5EF7`，发现其主要作用是调用 `call dword ptr [<&GDI32.GetObjectA>]` (即 `GetObjectA`)。

### GetObject 函数说明

The `GetObject` function obtains information about a specified graphics object. Depending on the graphics object, the function places a filled-in `BITMAP`, `DIBSECTION`, `EXTLOGPEN`, `LOGBRUSH`, `LOGFONT`, or `LOGPEN` structure, or a count of table entries (for a logical palette), into a specified buffer.

```c
int GetObject(  
    HGDIOBJ hgdiobj,  // handle to graphics object of interest  
    int cbBuffer,  // size of buffer for object information  
    LPVOID lpvObject   // pointer to buffer for object information   
   );   
```

#### Parameters

*   **`hgdiobj`**: A handle to the graphics object of interest. This can be a handle to one of the following: a logical bitmap, a brush, a font, a palette, a pen, or a device independent bitmap created by calling the `CreateDIBSection` function.

既然是游戏，估计是 `hgdiobj` 最后一个，试着下断点：`bp CreateDIBSection`。

(不难发现传给 `GetObject` 的 `hgdiobj` 存放于 `[4C0960]`，对其下内存（或硬件）写入断点，定位到)

```
0048BE4A  |.  899F 60094C00 mov     dword ptr [edi+4C0960], ebx  
```

往上找 `ebx` 值：

```
0048BE31  |.  E8 8A050000   call    0048C3C0  
0048BE36  |.  8BD8          mov     ebx, eax  
```

点击进入 `0048C3C0` 就可以发现 `ebx` 值就是 `CreateDIBSection` 的返回值。

## 2. 断点调试与结构体分析

重新载入，运行，断住了，清除断点，`Alt+F9` 执行，发现 `eax` 返回值是 0，说明没创建成功。

```
004E5EDD  /$  33C0          xor     eax, eax  
004E5EDF  |.  50            push    eax                             ; /Offset => 0  
004E5EE0  |.  50            push    eax                             ; |hSection => NULL  
004E5EE1  |.  FF7424 14     push    dword ptr [esp+14]              ; |ppBits  
004E5EE5  |.  50            push    eax                             ; |Usage => DIB_RGB_COLORS  
004E5EE6  |.  FF7424 18     push    dword ptr [esp+18]              ; |pBitmapInfo  
004E5EEA  |.  FF7424 18     push    dword ptr [esp+18]              ; |hDC  
004E5EEE  |.  FF15 4C945200 call    dword ptr [<&GDI32.CreateDIBSection>]    ; \CreateDIBSection  
004E5EF4  \.  C2 0C00       retn    0C  
```

在 `004E5EEE` 处下断点，重新载入运行到断点，看堆栈：

```
0012F56C   3C011BAC  |hDC = 3C011BAC  
0012F570   0012F5A0  |pBitmapInfo = 0012F5A0  
0012F574   00000000  |Usage = DIB_RGB_COLORS  
0012F578   0012F9C8  |ppBits = 0012F9C8  
0012F57C   00000000  |hSection = NULL  
0012F580   00000000  \Offset = 0  
```

使用命令 `d [esp+4]`，内存窗口转到 `pBitmapInfo` 结构。

### BITMAPINFO 结构说明

The `BITMAPINFO` structure defines the dimensions and color information for a Windows device-independent bitmap (`DIB`).

```c
typedef struct tagBITMAPINFO { // bmi   
   BITMAPINFOHEADER bmiHeader; 
   RGBQUAD          bmiColors[1]; 
} BITMAPINFO;  
```

The `BITMAPINFOHEADER` structure contains information about the dimensions and color format of a device-independent bitmap (`DIB`).

```c
typedef struct tagBITMAPINFOHEADER{ // bmih   
   DWORD  biSize; 
   LONG   biWidth; 
   LONG   biHeight; 
   WORD   biPlanes; 
   WORD   biBitCount 
   DWORD  biCompression; 
   DWORD  biSizeImage; 
   LONG   biXPelsPerMeter; 
   LONG   biYPelsPerMeter; 
   DWORD  biClrUsed; 
   DWORD  biClrImportant; 
} BITMAPINFOHEADER;  
```

#### `biBitCount`

Specifies the number of bits per pixel. This value must be 1, 4, 8, 16, 24, or 32.

#### `biCompression`

Specifies the type of compression for a compressed bottom-up bitmap (top-down `DIB`s cannot be compressed). It can be one of the following values:

| Value | Description |
| :--- | :--- |
| `BI_RGB` | An uncompressed format. |
| `BI_BITFIELDS` | Specifies that the bitmap is not compressed and that the color table consists of three doubleword color masks that specify the red, green, and blue components, respectively, of each pixel. This is valid when used with 16- and 32-bits-per-pixel bitmaps. |

也就是说：
*   `biBitCount`：颜色的位数
*   `biCompression`：`BI_RGB`=0，`BI_BITFIELDS`=3 (使用 16 位、32 位位图有效)

### 查看 `pBitmapInfo` 结构值

*   （`0012F5AE`）`+0Eh`: `biBitCount` 值为 `0020h`（32 位）
*   （`0012F5B0`）`+10h`: `biCompression` 值为 `00000003h`

游戏使用的是 256 色，应该将上面 `+0Eh` 处改为 8，`+10h` 处改为 0 才对，可以考虑在这里添加代码实现。

现在来看 `pBitmapInfo` 结构是什么时候生成的。按两下 `F8` 返回到 `004E599B`。

在该段函数入口处 `004E5907` 下断点，重新载入，运行到断点。内存窗口转到 `0012F5AE` 处，使用 `d 0012F5AE` 发现其值还不是 20，下内存写入（或硬件写入）断点。

`F9` 运行断在：

```
004E5E93  |.  66:8970 0E    mov     word ptr [eax+E], si  
```

其附近的代码:

```
004E5E6C  |.  6A 0C         push    0C                              ; /Index = BITSPIXEL  
004E5E6E  |.  53            push    ebx                               ; |hDC  
004E5E6F  |.  FF15 E4935200 call    dword ptr [<&GDI32.GetDeviceCaps>]         ; \GetDeviceCaps  
004E5E75  |.  66:8BF0       mov     si, ax                            ;    颜色位数： 改为: push 8 与 pop esi  
004E5E78  |.  53            push    ebx                               ; /hDC  
004E5E79  |.  57            push    edi                               ; |hWnd => NULL  
004E5E7A  |.  FF15 0C965200 call    dword ptr [<&USER32.ReleaseDC>]             ; \ReleaseDC  
004E5E80  |.  8B4424 10     mov     eax, dword ptr [esp+10]  
004E5E84  |.  B9 01000000   mov     ecx, 1  
004E5E89  |.  66:8948 0C    mov     word ptr [eax+C], cx  
004E5E8D  |.  C700 28000000 mov     dword ptr [eax], 28  
004E5E93  |.  66:8970 0E    mov     word ptr [eax+E], si                 ;    biBitCount  
004E5E97  |.  8948 04       mov     dword ptr [eax+4], ecx  
004E5E9A  |.  66:83FE 10    cmp     si, 10  
004E5E9E  |.  8948 08       mov     dword ptr [eax+8], ecx  
004E5EA1  |.  74 0B         je      short 004E5EAE  
004E5EA3  |.  66:83FE 20    cmp     si, 20  
004E5EA7  |.  74 05         je      short 004E5EAE  
004E5EA9  |.  8978 10       mov     dword ptr [eax+10], edi              ;    biCompression  
004E5EAC  |.  EB 07         jmp     short 004E5EB5  
004E5EAE  |>  C740 10 03000>mov     dword ptr [eax+10], 3                ;    biCompression  
004E5EB5  |>  33C9          xor     ecx, ecx  
```

显然要使:
`004E5E93` 处变成 `mov word ptr [eax+E], 8`
`004E5EAE` 处变成 `mov dword ptr [eax+10], 0`

最简单的修改方法就是将 `004E5E75` 处的 `mov si, ax` 改为：
```
push 8
pop esi
```
选中后右键选复制到可执行文件，保存覆盖原文件。`F9` 运行，正常。当然还没有免 cd，进不了游戏。

再次查看 `004E5E6C` 处的代码：
```
004E5E6C  |.  6A 0C         push    0C                              ; /Index = BITSPIXEL  
004E5E6E  |.  53            push    ebx                               ; |hDC  
004E5E6F  |.  FF15 E4935200 call    dword ptr [<&GDI32.GetDeviceCaps>]         ; \GetDeviceCaps  
```
这个 API 返回值就是每一像素用到的颜色数。

事先查到这个 API 的话，就可以直接下条件断点：`BP GetDeviceCaps, [esp+8] == 0C`。

按两次 `F9` 和 `alt+F9` 就可以定位到上面代码了。

## 3. 窗口激活与调色板动画

至此，可运行正常，但进入游戏后（可直接将 `00466170` 处改为 `retn` 实现免 cd），有时显示不正常，而最小化切换下窗口又能正常显示。

这显然和 windows 消息 `WM_ACTIVATE` 和 `WM_SETFOCUS` 有关。

（可下消息断点再跟进，我是直接查找 `0x202`, 定位到那些与 `0x202` 比较的，再到附近找）

定位到对 windows 消息 `WM_ACTIVATE` 和 `WM_SETFOCUS` 的处理函数，发现 `WM_SETFOCUS` 调用了 `InvalidateRect`。

### InvalidateRect 函数说明

The `InvalidateRect` function adds a rectangle to the specified window's update region. The update region represents the portion of the window's client area that must be redrawn.

可以考虑在适当的位置（比如消息循环）也调用这个函数来更新。

其实这是播放 palette animation 造成的。

What does it take to do palette animation in Windows? Just these three steps:

1.  Call `GetDeviceCaps`, and check `RC_PALETTE` to verify that palettes are supported. Palette animation won't work if the `RC_PALETTE` bit isn't set.
2.  Create a logical palette containing the colors you want to animate, and mark each palette entry with a `PC_RESERVED` flag. Only palette entries marked `PC_RESERVED` can be used for palette animation.
3.  Draw an image using colors in the logical palette, and then call `CPalette::AnimatePalette` repeatedly to change the palette colors. Each time you change the palette with `AnimatePalette`, the colors in the image will change accordingly.



摘自： `http://www.moon-soft.com/doc/3124.htm`  

而三国志 5 简体版不存在这种问题，跟踪发现，其在调用了 `AnimatePalette` 后，对是否为 8 位颜色做了判断，  

高于 8 位时调用了一个函数，该函数中调用了 `BitBlt`  

`The AnimatePalette function replaces entries in the specified logical palette.`  

`The BitBlt function performs a bit-block transfer of the color data corresponding to a rectangle of pixels`  

`from the specified source device context into a destination device context.`  

查找 `AnimatePalette` 发现日文加强版中在 `004E51C3` 和 `004E53CA` 两处调用到，只要在其后面添加上一段代码：  

(只改 `004E51C3` 就可以不过为了保险起见，还是都改了吧)  

```
mov     edx, dword ptr  [528068]  

push    dword ptr [edx*4+525770]  

push    dword ptr [edx*4+525B98]  

push    0  

push    0  

push    edx  

call    004E2603                  

add     esp,14  
```

(上面这段代码是比较两个版本弄出来的。也可以这样找：  

查找调用 `BitBlt` 的，定位到那个函数代码较短的，上一级函数的入口就是 `004E2603`  

传给 `004E2603` 的是区域范围，可查找 `0x320(800)` 几次 `alt+F8` 看到 `0x280(600)` 和 `0x1e0(480)` 定位到  

```
0042A75A   .  E8 50B10B00   call    004E58AF  
```

点击进去就可以看到选择的窗口大小保存的位置了。  

)  

三国志 5 的简体版和日文版可到本人主页下载  

`http://hi.baidu.com/goafteru`  

日文加强版原文件见附件。  

（附件文档，着色有点乱，大家就将就着看吧）  

题外话：  

   本来打算写一篇三国志 5 简体中文版免 `cd` 添加 `mp3` 背景音乐作为处女帖的。刚好正在去除 `35pk` 版的 `256` 限制，就改写这个了。写着写着头都大了。  

本人对 `window` 的 `api` 了解的很少，更不用说那些游戏要用到的 `api`，对文中的一些错处，请大家不啬指教。  

上传的附件  

| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) || `san5pk\_256.zip`  
|
| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) || `S5\_jpk\_old.zip`  
|

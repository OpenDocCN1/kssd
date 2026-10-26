
# [原创]修改扫雷程序实现悬停显雷

本人最近刚刚开始学习逆向分析，突发奇想想弄一个Windows XP扫雷的辅助。发帖前，在论坛上一搜，好多人都做过这个了，还有人做过Windows7 32位，64位的，突然感觉自己好低级……于是我又认真地把windowsXP扫雷的几乎所有汇编代码都分析了一遍，最后做了一个修改扫雷内部代码而实现悬停显示雷的操作。

效果如下：![](https://bbs.kanxue.com/upload/attach/202001/880853_W9B2R25DUH2UM6M.png)

进入正题，这是我的预计任务

1. 找到回调函数
2. 找到并分析雷表
   找到存放时间的位置
3. 分析绘图代码

首先，用OllyDbg打开windows xp的扫雷程序。

一开始如下图所示

```
01003E21 > $  6A 70         push    70

01003E23   .  68 90130001   push    01001390

01003E28   .  E8 DF010000   call    0100400C                         ;  SEH安装

01003E2D   .  33DB          xor     ebx, ebx

01003E2F   .  53            push    ebx                              ; /pModule => NULL

01003E30   .  8B3D 8C100001 mov     edi, dword ptr [<&KERNEL32.GetMo>; |KERNEL32.GetModuleHandleA

01003E36   .  FFD7          call    edi                              ; \GetModuleHandleA

01003E38   .  66:8138 4D5A  cmp     word ptr [eax], 5A4D             ;  MZ头确认

01003E3D   .  75 1F         jnz     short 01003E5E

01003E3F   .  8B48 3C       mov     ecx, dword ptr [eax+3C]          ;  ecx=elfanew

01003E42   .  03C8          add     ecx, eax

01003E44   .  8139 50450000 cmp     dword ptr [ecx], 4550            ;  PE头确认

01003E4A   .  75 12         jnz     short 01003E5E

01003E4C   .  0FB741 18     movzx   eax, word ptr [ecx+18]           ;  eax=OptionalHeader

01003E50   .  3D 0B010000   cmp     eax, 10B                         ;  Magic
```

显然，是属于程序的入口部分代码，包含有初始化SHE，获取ImageBase，测试PE文件的有效性等等等等。

## 一：找窗口回调函数

为了实现找到回调函数这一个目标，首先需要知道获取来源。

首先想到`WNDCLASSA`或`WNDCLASSW`和`WNDCLASSEX`这两个结构体，`WNDCLASSW`定义如下

```
typedef struct tagWNDCLASSW {

    UINT        style;

    WNDPROC     lpfnWndProc;

    int         cbClsExtra;

    int         cbWndExtra;

    HINSTANCE   hInstance;

    HICON       hIcon;

    HCURSOR     hCursor;

    HBRUSH      hbrBackground;

    LPCWSTR     lpszMenuName;

    LPCWSTR     lpszClassName;

} WNDCLASSW, *PWNDCLASSW, NEAR *NPWNDCLASSW, FAR *LPWNDCLASSW;
```

这两个结构体都是用来初始化窗口类的，在`RegisterClass`中被引用，根据该结构体的定义，在此结构的`(+0x04)`位置处，就是窗口回调函数的地址。所以，从设置`WNDCLASS`成员时指定的地址或者从`RegisterClass`调用时的栈窗口，可以找到`WNDCLASS`的地址，进而找到`lpfnWndProc`的值。

`RegisterClass`一般在主线程的`WinMain`中被调用，我向下跟踪，来到此处

```
01003F8F   .  50            push    eax

01003F90   .  E8 5BE2FFFF   call    010021F0

01003F95   .  8BF0          mov     esi, eax

01003F97   .  8975 84       mov     dword ptr [ebp-7C], esi

01003F9A   .  395D E4       cmp     dword ptr [ebp-1C], ebx

01003F9D   .  75 07         jnz     short 01003FA6

01003F9F   .  56            push    esi                              ; /status

01003FA0   .  FF15 94110001 call    dword ptr [<&msvcrt.exit>]       ; \exit

01003FA6   >  FF15 9C110001 call    dword ptr [<&msvcrt._cexit>]     ; [msvcrt._cexit
```

目前窗口并没有创建，而接下来的第二个函数是`exit`的调用，标志着程序的退出，因此可以确认，下面的第一个函数，就是程序的`WinMain`函数。

跟踪进去

```
010021F0  /$  55            push    ebp

010021F1  |.  8BEC          mov     ebp, esp

010021F3  |.  83EC 4C       sub     esp, 4C

010021F6  |.  8B45 08       mov     eax, dword ptr [ebp+8]

010021F9  |.  53            push    ebx

010021FA  |.  56            push    esi

010021FB  |.  57            push    edi

010021FC  |.  A3 305B0001   mov     dword ptr [1005B30], eax

01002201  |.  E8 AA180000   call    01003AB0                         ;  设置随机数种子

01002206  |.  33DB          xor     ebx, ebx

01002208  |.  43            inc     ebx

01002209  |.  33FF          xor     edi, edi

0100220B  |.  837D 14 07    cmp     dword ptr [ebp+14], 7

0100220F  |.  74 0C         je      short 0100221D

01002211  |.  837D 14 02    cmp     dword ptr [ebp+14], 2

01002215  |.  893D 385B0001 mov     dword ptr [1005B38], edi

0100221B  |.  75 06         jnz     short 01002223

0100221D  |>  891D 385B0001 mov     dword ptr [1005B38], ebx

01002223  |>  8D45 F8       lea     eax, dword ptr [ebp-8]

01002226  |.  50            push    eax                              ; /pInitEx

01002227  |.  C745 F8 08000>mov     dword ptr [ebp-8], 8             ; |

0100222E  |.  C745 FC FD160>mov     dword ptr [ebp-4], 16FD          ; |

01002235  |.  FF15 1C100001 call    dword ptr [<&COMCTL32.InitCommon>; \InitCommonControlsEx

0100223B  |.  6A 64         push    64                               ; /RsrcName = 100.

0100223D  |.  FF35 305B0001 push    dword ptr [1005B30]              ; |hInst = NULL

01002243  |.  FF15 AC100001 call    dword ptr [<&USER32.LoadIconW>]  ; \LoadIconW

01002249  |.  8B0D 305B0001 mov     ecx, dword ptr [1005B30]

0100224F  |.  68 007F0000   push    7F00                             ; /RsrcName = IDC_ARROW

01002254  |.  57            push    edi                              ; |hInst

01002255  |.  A3 285B0001   mov     dword ptr [1005B28], eax         ; |

0100225A  |.  897D B4       mov     dword ptr [ebp-4C], edi          ; |

0100225D  |.  C745 B8 C91B0>mov     dword ptr [ebp-48], <lpfnWndProc>; |

01002264  |.  897D BC       mov     dword ptr [ebp-44], edi          ; |

01002267  |.  897D C0       mov     dword ptr [ebp-40], edi          ; |

0100226A  |.  894D C4       mov     dword ptr [ebp-3C], ecx          ; |

0100226D  |.  8945 C8       mov     dword ptr [ebp-38], eax          ; |

01002270  |.  FF15 BC100001 call    dword ptr [<&USER32.LoadCursorW>>; \LoadCursorW

01002276  |.  53            push    ebx                              ; /ObjType

01002277  |.  8945 CC       mov     dword ptr [ebp-34], eax          ; |

0100227A  |.  FF15 60100001 call    dword ptr [<&GDI32.GetStockObjec>; \GetStockObject

01002280  |.  8945 D0       mov     dword ptr [ebp-30], eax

01002283  |.  8D45 B4       lea     eax, dword ptr [ebp-4C]

01002286  |.  BE A05A0001   mov     esi, 01005AA0

0100228B  |.  50            push    eax                              ; /pWndClass

0100228C  |.  897D D4       mov     dword ptr [ebp-2C], edi          ; |

0100228F  |.  8975 D8       mov     dword ptr [ebp-28], esi          ; |

01002292  |.  FF15 CC100001 call    dword ptr [<&USER32.RegisterClas>; \RegisterClassW
```

在第一个`CALL`处我进行了查看，发现是设置随机数种子，接下来明显地看到调用了`LoadIconW`，`LoadCursorW`，和`GetStockObject`三个API函数对`WNDCLASSW`进行设置，往下翻，立刻可以看见`RegisterClassW`。事实上，`hIcon`的位置在`lpfnWndProc(+0x10)`的位置，从`LoadIcon`的返回值的设置位置(2个地方)

```
01002255  |.  A3 285B0001   mov     dword ptr [1005B28], eax         ; |
```

一开始我以为是这里，结果发现`1005B18`没有写入的代码，接着向下看：

```
0100226D  |.  8945 C8       mov     dword ptr [ebp-38], eax          ; |
```

可知`ebp-38-10=ebp-48`就是回调函数的地址，接下来看到

```
0100225D  |.  C745 B8 C91B0>mov     dword ptr [ebp-48], 01001BC9     ; |
```

可以确认，`01001BC9`就是回调函数的地址了，添加标签为`lpfnWndProc`

## 二：

进入窗口回调函数，马上看到一个`switch`

```
01001C05   .  48            dec     eax                              ;  Switch (cases 2..47)
```

根据对回调函数的整体浏览，发现对`WM_LBUTTONDOWN`的处理部分，有将（记住这个地址）`1005144`处的数据设为0这估计是为了实现左键按下时显示格子0的效果，添加标签为`bLButtonUp`(Down的非是Up (`へ´\*)ノ)

接着对`WM_LBUTTONDOWN`和`WM_LBUTTONUP`这两个消息的处理部分下断

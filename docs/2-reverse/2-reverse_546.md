
# 逆向入口切入【原创】

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-75.htm`](https://www.pediy.com/kssd/pediy07/pediy7-75.htm)

逆向入口切入

作者：nbw

谨以此菜文献给 NE365 和 FCG。以资新年庆贺！也祝所有圈内人士新年吉祥！

逆向分析，最开始的一步就是寻找入口点。虽然可以破墙而入，但为了防止“墙后面还是一堵墙”，因此我们还是喜欢去撬门锁。一般来说，破解的话，找一些 `MessageBox` 注册提示，网络程序会考虑搞网络 api，找到了正确的入口，就可以登堂入室，为所欲为。

但是一般的软件逆向分析，入口或许稍微难找一些，有时因为找不到正确的分析入口而放弃分析。本次目标是某股票分析软件。

该软件将根据股市信息计算出的数据以柱状显示在坐标系中。我们的目标是找到计算这些数据的代码。找到了这些代码，要弄懂他们就只是时间问题了。

既然是柱形图显示，希望大家先看一下 GDI 编程，稍微瞅瞅就行，毕竟这里的关键是找到计算柱形图数据的代码。柱形图变换菜单为“查看”/“显示柱状参数”，因此可以从这个菜单消息入手。

找菜单消息处理有很多方法，我比较常用相近处理法。因为菜单消息处理函数都离得很近，所以如果可以找到一个菜单的处理地点，其他菜单函数也相应不远了，这就是我所谓的相近处理法，因为我不会脱壳，也懒得学 OD 那些高级技巧，被逼无奈而已……….

先运行股票分析软件，经过研究，`F7` 键对应的“今日提示”菜单，有点 `MessageBox` 的味道，于是用 OD 附加进程。下 `MessageBox` 断点，在软件中按 `F7`，被 OD 拦截，`Ctrl + F9` 返回调用处：

```
0041A64B    push 0                        
0041A64D    push 0043B2FC                   ; |Title = "今日提示"
0041A652    push 00491E80                   ; |Text = "  ..?...
0041A657    mov ecx,dword ptr ss:[ebp+8]      ; |
0041A65A    push ecx                      ; |hOwner
0041A65B    call dword ptr ds:[42D17C]      ; \MessageBoxA
0041A661    jmp 00422EB2
```

继续 `Ctrl + F9` 返回上一层调用：

```
77E1A411    push dword ptr ss:[ebp+18]
77E1A414    push dword ptr ss:[ebp+14]
77E1A417    push dword ptr ss:[ebp+10]
77E1A41A    push dword ptr ss:[ebp+C]
77E1A41D    call dword ptr ss:[ebp+8]           
77E1A420    cmp dword ptr ss:[esp+4],DCBAABCD
77E1A428    je short user32.77E1A43B
```

幸运的话这个 `call dword ptr ss:[ebp+8]`或许是菜单消息主处理函数，如果在这里下断点，拦不住其他菜单操作，那可以尝试继续 `Ctrl + F9` 返回更上一层调用函数，一直到发现主处理函数。但不幸的是上面代码处于 `User32.dll` 领空，更不幸的是对软件的任何操作都会被这个函数拦截，事实上这个函数是消息处理函数，如 B 哥所说，晃一下鼠标都要被他拦住。

不过正所谓柳暗花明又一村，如果你乐意，可以在这里下条件断点，但我更愿意继续翻一下最开始的 `MessageBox` 。光标定位在 `0041A64B    push 0    ; /Style = MB_OK|MB_APPLMODAL` 根据 OD 提示，该句代码被引发自：

```
0041A513    mov ecx,dword ptr ss:[ebp-7C]
0041A516    xor eax,eax
0041A518    mov al,byte ptr ds:[ecx+42303F]
0041A51E    jmp dword ptr ds:[eax*4+422FDF]  ;-------典型的消息 Table 跳转
```

上面的 `jmp`，是比较典型的条件跳转，根据 `eax` 不同，跳向不同位置。在这个地方下断点，会发现一般的操作，软件不会被中断了，说明大部分消息处理都不经过这个地方。但是随便找一个菜单点一下，很明显被断在这个地方。

那么点一下需要分析的柱形参数菜单，被 OD 拦截在此，继续单步运行，如下：

```
0041C47E    movsx edx,word ptr ds:[447102]
0041C485    neg edx
0041C487    sbb edx,edx
0041C489    inc edx
0041C48A    mov word ptr ds:[447102],dx
0041C491    movsx eax,word ptr ds:[447102]
0041C498    test eax,eax
0041C49A    jnz short .0041C4DE    ;------跳
```

到此：

```
0041C4DE    push .0043B3A4           
```

省略几句…….

```
0041C4EF     mov eax,dword ptr ds:[446C98]
0041C4F4    push eax                ; |hMenu => 02170742
0041C4F5    call dword ptr ds:[42D180]        ; \ModifyMenuA
0041C4FB    push 0043B3B4           
0041C500    push 91                 ; |NewItemID = 91 (145.)
0041C505     push 0    ; |Flags = MF_BYCOMMAND|MF_ENABLED|MF_STRING
```

省略几句…….

```
0041C510    call dword ptr ds:[42D174]            ; |\GetMenu
0041C516    push eax                         ; |hMenu
0041C517    call dword ptr ds:[42D180]            ; \ModifyMenuA
0041C545    add edx,97
```

省略几句…….

```
0041C54B    mov dword ptr ds:[446CDC],edx
0041C551    push 1            ; /Erase = TRUE
0041C553    push  .00446CD0      ; |pRect = 00446CD0 {732.,-80.,837.,636.}
0041C558    mov eax,dword ptr ss:[ebp+8]      ; |
0041C55B    push eax                ; |hWnd
0041C55C    call dword ptr ds:[42D188]        ; \InvalidateRect
0041C562    xor eax,eax
0041C564    jmp  .00422EB2
```

清注意右边的注释，上面说白了就是修改一下菜单提示内容，然后调用 `InvalidateRect` 函数。再往下就跳出当前函数：

```
00422EB2    pop esi                     ;  0012FF20
00422EB3    mov esp,ebp
00422EB5     pop ebp
00422EB6    retn 10
```

`F8` 单步跟出去：

```
77E1A41D    call dword ptr ss:[ebp+8]        ;-----刚才所在的函数
77E1A420    cmp dword ptr ss:[esp+4],DCBAABCD  ;-----返回处
77E1A428    je short user32.77E1A43B        ;-----跳
```

到此：

```
77E1A43B    add esp,8
77E1A43E    pop ebp
77E1A43F    retn 14
```

继续跟出去，再返回几次，回到主进程领空：

```
00419B4D    push 0                          ; /MsgFilterMax = 0
00419B4F    push 0                          ; |MsgFilterMin = 0
00419B51    push 0                          ; |hWnd = NULL
00419B53    lea edx,dword ptr ss:[ebp-1C]        ; |
00419B56    push edx                      ; |pMsg
00419B57    call dword ptr ds:[42D1D0]           ; \GetMessageA
00419B5D    test eax,eax
00419B5F    je short  .00419B77
00419B61    lea eax,dword ptr ss:[ebp-1C]
00419B64    push eax                     ; /pMsg
00419B65    call dword ptr ds:[42D1D4]           ; \TranslateMessage
00419B6B    lea ecx,dword ptr ss:[ebp-1C]
00419B6E    push ecx                     ; /pMsg
00419B6F    call dword ptr ds:[42D1D8]           ; \DispatchMessageA
00419B75    jmp short  .00419B4D        ;-----跳到上面
```

花 3 秒钟看一下，上面就是往消息队列放消息。这样看来对于柱形参数菜单软件的处理流程就是：

1、  修改该菜单内容；

2、  调用 `InvalidateRect` 函数；

3、  继续日常消息传送。

由于这个菜单点下以后，可以看到经过计算的柱形股票数据分析。但上面即没有什么高深的计算，连柱形图显示都没发现。

那么先泡一包方便面，郁闷 10 分钟………..

吃完面，继续研究。既然这个菜单没有什么金子，目前来说，可以考虑一下处理画坐标系的方法，比如可以拦截一下 GDI 的画图函数，看看软件怎么画那个坐标系。

但翻了一下 GDI 教程，发现 `InvalidateRect` 函数可以引发 `WM_PAINT` 消息，从而导致窗口重画。那么看来就是这个函数引发窗体上的坐标系重画。既然如此，下一步就是找到处理 `WM_PAINT` 消息的地方。

寻找窗体消息处理，同样有 N 种方法。但我又要说我不会脱壳，也不会用 OD 的高级功能。因此慢慢来吧。

这个软件的典型 SDK 风格让我这个没写过 SDK 的人也忍不住去看典型的窗体消息处理：

```
Mov    @stWndClass.lpfnWndProc,offset _ProcWinMain
Mov    @stWndClass.hbrBackground,COLOR_WINDOW + 1
Mov    @stWndClass.lpszClassName,offset szClassName
Invoke  RegisterClassEx,addr @stWndClass
invoke  CreateWindowEx,WS_EX_CLIENTEDGE,offset……（略几个参数）
```

既然如此，就应该找到 `CreateWindowEx`，然后看一下上面的 `offset _ProcWinMain`。由于 N 多程序喜欢把关键界面采用子窗口处理，因此还要防止上面的 `InvalidateRect` 是处理的子窗口，那样就需要从 N 个 `CreateWindowEx` 函数中找到创建目标子窗口的那一个。

窗口的创建一般在程序初始化时候进行，因此退出软件。用 OD 加载软件，提示都不理会，中断在入口后，用 `Ctrl+A` 让 OD 分析一下代码。下 `CreateWindowEx` 断点，`F9` 运行 OD。被中断在 `CreateWindowEx`，返回到主程序领空的调用处（注意用 `Ctrl+A` 分析一下那些杂乱 op 就会显示代码）。

```
00419C25    push eax                       ; |Width
00419C26    push 32                       ; |Y = 32 (50.)
00419C28    push 64                       ; |X = 64 (100.)
00419C2A    push 0CF0000                 
00419C2F    push  .0043B030              
00419C34    push  .0043B048              
00419C39    push 0                      ; |ExtStyle = 0
00419C3B    call dword ptr ds:[42D214]            ; \CreateWindowExA
00419C41     mov dword ptr ss:[ebp-4],eax
```

考虑到有子窗口的问题，为了确定该次 `CreateWindowEx` 是否创建的是画坐标系的窗口，因此记下返回的窗口句柄值：`018D0238 H`。

`F9` 继续运行，用窗口 spy 之类的东东查看一下坐标系所在窗口的句柄，也是 `018D0238 H`，其实坐标系所在窗口就是主窗体。这样，上面分析的 `WM_PAINT` 消息触发的就是主窗体，然后重画主窗体上的坐标系。

再温习一下上面的那几句注册窗口的汇编代码，用 OD 重新加载一下软件，在启动时候找到这个 `CreateWindowEx` 上面的 `RegisterClass`，就是这个窗口的注册函数：

```
00419BD5    mov dword ptr ss:[ebp-8], .0043B014       
00419BDC    mov dword ptr ss:[ebp-4], .0043B020        
00419BE3    lea edx,dword ptr ss:[ebp-28]
00419BE6    push edx                 ; /pWndClass
00419BE7    call dword ptr ds:[42D1CC]    ; \RegisterClassA
00419BED    and eax,0FFFF
```

上面的 `00419BE6    push edx` 指向窗口的注册信息结构，查一下 MSDN，结构定义如下：

```
typedef struct _WNDCLASS { 
    UINT    style; 
    WNDPROC lpfnWndProc;     窗口消息函数地址 
    int     cbClsExtra; 
    int     cbWndExtra; 
    HANDLE  hInstance; 
    HICON   hIcon; 
    HCURSOR hCursor; 
    HBRUSH  hbrBackground; 
    LPCTSTR lpszMenuName; 
    LPCTSTR lpszClassName; 
} WNDCLASS
```

该结构第 2 个 word 就是窗口消息处理函数。结构如下：

```
0012FEE4   08 00 00 00 6D 9C 41 00 00 00 00 00 00 00 00 00   ...m 淎.........
```

根据第 2 个字节，找到地址 `419C6D` 。代码如下：

```
00419C6D    push ebp
00419C6E    mov ebp,esp
00419C70    sub esp,100
00419C76    push esi
00419C77     movsx eax,byte ptr ds:[446DC9]
00419C7E    test eax,eax
00419C80     je short  .00419D01        ;此处跳
00419C82    mov ecx,dword ptr ss:[ebp+C]
00419C85    mov dword ptr ss:[ebp-74],ecx
```



### 汇编代码片段

```
00419C88    cmp dword ptr ss:[ebp-74],114  

00419C8F    ja short  .00419CBF  

00419C91     cmp dword ptr ss:[ebp-74],114        


00419C98    je short  .00419CDC  

00419C9A    cmp dword ptr ss:[ebp-74],102  

00419CA1    ja short  .00419CB4  

00419CA3    cmp dword ptr ss:[ebp-74],100  

00419CAA    jnb short  .00419CDC  

00419CAC    cmp dword ptr ss:[ebp-74],10  

00419CB0    je short  .00419CDC  

00419CB2    jmp short  .00419CE6
```

上面的比较，就是比较熟悉的消息比较。如果在最开始地方（`00419C6D` 处）下断点，就会被频繁中断，因为是消息处理最开头嘛。

### 观察跳转点

尝试在几个跳转比较点下断点，会发现上面的：

```
00419C80     je short  .00419D01        ;此处跳
```

会跳转，因此，观察跳转处：

```
00419D01    mov eax,dword ptr ss:[ebp+C]    ；获取传来的消息  
00419D04    mov dword ptr ss:[ebp-78],eax  

00419D07    cmp dword ptr ss:[ebp-78],111  

00419D0E    ja short  .00419D7E  

00419D10    cmp dword ptr ss:[ebp-78],111  

00419D17    je  .0041A4DB  

00419D1D    cmp dword ptr ss:[ebp-78],4E  

00419D21    ja short  .00419D52  

00419D23    cmp dword ptr ss:[ebp-78],4E  

00419D27    je  .0041A2A4  

00419D2D    mov ecx,dword ptr ss:[ebp-78]  

00419D30    sub ecx,1                     ;  wm_paint = 0F h  

00419D33    mov dword ptr ss:[ebp-78],ecx  

00419D36    cmp dword ptr ss:[ebp-78],0E  

00419D3A    ja  .00422E98  

00419D40    mov eax,dword ptr ss:[ebp-78]        ; 如果 wm_paint，此处为 0E h  

00419D43    xor edx,edx  

00419D45    mov dl,byte ptr ds:[eax+422ECD]  

00419D4B     jmp dword ptr ds:[edx*4+422EB9]
```

由于我们关心 `WM_PAINT` 消息，在 `windows.h` 查一下该消息值为：`0F H`。

### 代码流向分析

上面的 `00419D01 mov eax,dword ptr ss:[ebp+C]` 用来获取传来的消息。我们假定该处获得 `WM_PAINT` 消息，也就是 `0FH`。观察一下代码的流向。这里很好观察，但是为了避免出错，你可以把传来的参数硬性改成 `0FH`。到了：

```
00419D4B     jmp dword ptr ds:[edx*4+422EB9]
```

再往下：

```
0041EFEA    mov eax,dword ptr ds:[43CFD4]  

0041EFEF    cmp eax,dword ptr ds:[43CFDC]  

0041EFF5    jge short  .0041F00B
```

然后不用细说，`F8` 执行几步就到了处理数据的地方，如果再往下跟踪，就是 `GDI` 画图函数，用来将计算出来的数据显示在坐标系里面。

### 核心算法分析

下面就正式登堂入室，可以分析软件如何计算数据，然后将这些数据以图形形式打印在界面上。

分析这些入口点，一般来说需要一些技巧，但是更需要扎实的功底，我一般不太乐意看一些花哨的插件或者高级的用法，当然并不是说那样东西不好，只不过是说采用一些基础的知识，其实可以达到那些目的。基础的知识才是我们需要加强的。

再往下就可以分析软件的算法了。

很多程序的核心算法，都是枯燥的数据运算，如果要搞清楚他们，关键有 3 点：

1. 找到他们所在的地方；  
   这就是本文所讲的东西。
2. 可以理解整个运算过程；  
   这是枯燥的跟踪调试，也是最消耗时间很反映个人基础的过程；
3. 理解算法的逻辑。  
   看起来这一点没必要，我以前也这么认为。但是后来发现，有些东西，即使理解运算过程，也很难理解运算逻辑，也就是不晓得为什么要那么运算。要做到这一点，就靠触类旁通的天赋+平时的积累。

因此，诚如一位大侠所说：“终极逆向工程，在于理解对方的整个运算流程和逻辑思维。”吾等小辈，唯有孜孜不倦，才有可能达到终极境界。

`4dd466d0e1e0cc123afbe504da2462ca`

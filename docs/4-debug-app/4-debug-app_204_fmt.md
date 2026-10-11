
# 用 IDA Pro + OD 来分析扫雷

> 原文：[`www.pediy.com/kssd/pediy12/138855.html`](https://www.pediy.com/kssd/pediy12/138855.html)

【声明】小弟菜鸟一只![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")，潜水多年，酷爱游戏，更爱逆向，喜欢修改游戏。坛子里关于扫雷的文章已经很多了，各位前辈都已经分析透彻了。小弟发此文，只是给和我一样的初级逆向爱好者提供一些思路，看看如何分析一个游戏，做出修改器，甚至是修改游戏规则。有不对的地方，各位前辈请指教，多谢![](img/2aa97d64a7e0b211d2f8f207eb382aa7.jpg "酷")。

工具：`IDA Pro`，`OllyDbg`，`exeScope` 或者 `resource hacker`

首先，我们知道，对于 `VC` 写的 `windows` 编程来说，一般都是创建窗体，注册窗体，显示窗体，更新窗体，再来一个消息处理函数。我们先用 `IDA Pro` 来看看，至于为什么要用 `IDA`，是因为我觉得 `IDA` 在某些方面比 `OD` 方便，比如函数名，变量名替换之类的，和 `OD` 配合使用，效果很好。

打开 `IDA Pro`，自动进入 `EP`，一开始肯定是一些加载器的初始化过程，没有必要仔细看，一路向下走。

```assembly
.text:01003E21                 public start

.text:01003E21 start          proc near

.text:01003E21                 push    70h
.text:01003E23                 push    offset dword_1001390
.text:01003E28                 call    sub_100400C
.text:01003E2D                 xor     ebx, ebx
.text:01003E2F                 push    ebx             ; lpModuleName
.text:01003E30                 mov     edi, ds:GetModuleHandleA
.text:01003E36                 call    edi ; GetModuleHandleA
.text:01003E38                 cmp     word ptr [eax], 5A4Dh ; MZ
.text:01003E3D                 jnz     short loc_1003E5E
.text:01003E3F                 mov     ecx, [eax+3Ch]
.text:01003E42                 add     ecx, eax
.text:01003E44                 cmp     dword ptr [ecx], 4550h
.text:01003E4A                 jnz     short loc_1003E5E
.text:01003E4C                 movzx   eax, word ptr [ecx+18h]
.text:01003E50                 cmp     eax, 10Bh
.text:01003E55                 jz      short loc_1003E76
.text:01003E57                 cmp     eax, 20Bh
.text:01003E5C                 jz      short loc_1003E63
```

当到达这里的时候：

```assembly
text:01003F89 loc_1003F89:                      ; CODE XREF: start+158j

.text:01003F89                 push    eax             ; hAccTable
.text:01003F8A                 push    esi             ; int
.text:01003F8B                 push    ebx             ; int
.text:01003F8C                 push    ebx             ; lpModuleName
.text:01003F8D                 call    edi ; GetModuleHandleA
.text:01003F8F                 push    eax             ; int
.text:01003F90                 call    sub_10021F0    ;这是咱们的 Main 函数。
.text:01003F95                 mov     esi, eax
.text:01003F97                 mov     [ebp-7Ch], esi
.text:01003F9A                 cmp     [ebp-1Ch], ebx
.text:01003F9D                 jnz     short loc_1003FA6
.text:01003F9F                 push    esi             ; int
.text:01003FA0                 call    ds:exit
```

我们就发现了 `main` 函数，`10021F0`，为什么是这个？你看这个函数执行完了，程序就退出了啊。我们可以用 `IDA` 给它改名字，叫做 `Main`。不废话，进入 `main` 函数看看。

看！`IDA` 多智能，告诉我们了参数和临时变量，有 `WndClass` 和 `MSG`，大家是不是眼前一亮。

```assembly
.text:010021F0 Main            proc near               ; CODE XREF: start+16Fp

.text:010021F0

.text:010021F0 WndClass        = WNDCLASSW ptr -4Ch
.text:010021F0 Msg             = MSG ptr -24h
.text:010021F0 var_8           = tagINITCOMMONCONTROLSEX ptr -8
.text:010021F0 arg_0           = dword ptr  8
.text:010021F0 hAccTable       = dword ptr  14h
.text:010021F0

.text:010021F0                 push    ebp
.text:010021F1                 mov     ebp, esp
.text:010021F3                 sub     esp, 4Ch
.text:010021F6                 mov     eax, [ebp+arg_0]
.text:010021F9                 push    ebx
```

继续走，往下几行就到这里了：

```assembly
text:0100225A                  mov     [ebp+WndClass.style], edi
.text:0100225D                  mov     [ebp+WndClass.lpfnWndProc], offset WndProccess  ;就是她！
.text:01002264                  mov     [ebp+WndClass.cbClsExtra], edi
.text:01002267                  mov     [ebp+WndClass.cbWndExtra], edi
.text:0100226A                  mov     [ebp+WndClass.hInstance], ecx
.text:0100226D                  mov     [ebp+WndClass.hIcon], eax
.text:01002270                  call    ds:LoadCursorW
.text:01002276                  push    ebx             ; int
.text:01002277                  mov     [ebp+WndClass.hCursor], eax
.text:0100227A                  call    ds:GetStockObject
.text:01002280                  mov     [ebp+WndClass.hbrBackground], eax
.text:01002283                  lea     eax, [ebp+WndClass]
.text:01002286                  mov     esi, offset AppName
.text:0100228B                  push    eax             ; lpWndClass
.text:0100228C                  mov     [ebp+WndClass.lpszMenuName], edi
.text:0100228F                  mov     [ebp+WndClass.lpszClassName], esi
.text:01002292                  call    ds:RegisterClassW
```

`RegisterClassW`，哈哈，我们这下子就知道消息处理函数了。先继续往下走，过一会儿我们看看这个消息处理函数。

```assembly
.text:010022E1                  push    edi             ; lpParam
.text:010022E2                  push    hInstance       ; hInstance
.text:010022E8                  add     ecx, eax
.text:010022EA                  push    edi             ; hMenu
.text:010022EB                  push    edi             ; hWndParent
.text:010022EC                  push    ecx             ; nHeight
.text:010022ED                  mov     ecx, dword_1005A90
.text:010022F3                  add     edx, ecx
.text:010022F5                  push    edx             ; nWidth
.text:010022F6                  mov     edx, Y
.text:010022FC                  sub     edx, eax
.text:010022FE                  mov     eax, X
.text:01002303                  push    edx             ; Y
.text:01002304                  sub     eax, ecx
.text:01002306                  push    eax             ; X
.text:01002307                  push    0CA0000h        ; dwStyle
.text:0100230C                  push    esi             ; lpWindowName
.text:0100230D                  push    esi             ; lpClassName
.text:0100230E                  push    edi             ; dwExStyle
.text:0100230F                  call    ds:CreateWindowExW
.text:01002315                  cmp     eax, edi
.text:01002317                  mov     hWnd, eax
.text:0100231C                  jnz     short loc_1002325
.text:0100231E                  push    3E8h
.text:01002323                  jmp     short loc_1002336
```

嗯，创建窗体了。接着走，看返回的 `EAX` 不为空的话就跳。跳到这里：

```assembly
text:01002325 loc_1002325:                      ; CODE XREF: Main+12Cj

.text:01002325                  push    ebx
.text:01002326                  call    sub_1001950
.text:0100232B                  call    sub_1002B14
.text:01002330                  test    eax, eax
.text:01002332                  jnz     short loc_1002342
.text:01002334                  push    5

.....

.....

.text:01002342 loc_1002342:                      ; CODE XREF: Main+142j

.text:01002342                  push    dword_10056C4
.text:01002348                  call    sub_1003CE5
.text:0100234D                  call    sub_100367A
.text:01002352                  push    ebx             ; nCmdShow
.text:01002353                  push    hWnd            ; hWnd
.text:01002359                  call    ds:ShowWindow
.text:0100235F                  push    hWnd            ; hWnd
.text:01002365                  call    ds:UpdateWindow
.text:0100236B                  mov     esi, ds:GetMessageW
.text:01002371                  mov     dword_1005B38, edi
.text:01002377                  jmp     short loc_10023A4
```

发现 4 个函数，我们要进去看看，因为凭咱们玩扫雷的经验来看，雷的产生是在窗体绘制之前就做好了，也就是 `showWindow` 之前，会有函数来布雷，所以，这四个函数，`sub_1001950`, `sub_1002B14`, `sub_1003CE5`，`sub_100367A` 应该有一个或者多个来处理布雷。一个一个看看。

先看 `sub_1001950`，函数太长![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")，就不都贴了，贴一部分：

```assembly
....

....

.text:01001978                  mov     edi, ds:GetMenuItemRect
.text:0100197E                  mov     dword_1005B88, eax
.text:01001983                  jnz     short loc_10019DB
```



### .text:01001985

```
.text:01001985                    mov     edx, dword_1005B34  

.text:0100198B                    add     edx, eax  

.text:0100198D                    mov     eax, hMenu  

.text:01001992                    cmp     eax, ebp  

.text:01001994                    mov     dword_1005B88, edx  

.text:0100199A                    jz      short loc_10019DB  

.text:0100199C                    lea     edx, [esp+40h+rcItem]  

.text:010019A0                    push    edx             ; lprcItem  

.text:010019A1                    push    ebp             ; uItem  

.text:010019A2                    push    eax             ; hMenu  

.text:010019A3                    push    ecx             ; hWnd  

.text:010019A4                    call    edi ; GetMenuItemRect  

.text:010019A6                    test    eax, eax  

.text:010019A8                    jz      short loc_10019DB  

.text:010019AA                    lea     eax, [esp+40h+var_20]  

.text:010019AE                    push    eax             ; lprcItem  

.text:010019AF                    push    ebx             ; uItem  

.text:010019B0                    push    hMenu           ; hMenu  

.text:010019B6                    push    hWnd            ; hWnd  

.text:010019BC                    call    edi ; GetMenuItemRect  

....  

....  
```

从函数上看，是处理菜单那一块的东西，（如果错了，请各位指正，我当时一看函数不像，就没仔细看）。这个函数不像，那就下一个 `sub_1002B14`：

```
.text:01002B14 sub_1002B14     proc near               ; CODE XREF: Main+13Bp  

.text:01002B14                   call    sub_1002414  

.text:01002B19                   test    eax, eax  

.text:01002B1B                   jnz     short loc_1002B1E  

.text:01002B1D                   retn  
```

她居然 call 了另外一个函数，`sub_1002414`。进入 `sub_1002414` 看了就知道，是处理 resource 的，就不贴了，很明显的。

下一个函数，`sub_1003CE5`：

```
.text:01003CE5 arg_0           = dword ptr  4  

.text:01003CE5  

.text:01003CE5                   mov     eax, [esp+arg_0]  

.text:01003CE9                   mov     dword_10056C4, eax  

.text:01003CEE                   call    sub_1001516  

.text:01003CF3                   mov     eax, dword_10056C4  

.text:01003CF8                   and     al, 1  

.text:01003CFA                   neg     al  

.text:01003CFC                   sbb     eax, eax  

.text:01003CFE                   not     eax  

.text:01003D00                   and     eax, hMenu  

.text:01003D06                   push    eax             ; hMenu  

.text:01003D07                   push    hWnd            ; hWnd  

.text:01003D0D                   call    ds:SetMenu      ; Assign a new menu to the specified window  

.text:01003D13                   push    2  

.text:01003D15                   call    sub_1001950  

.text:01003D1A                   retn    4  
```

注意到一个 `SetMenu`，应该是设置菜单的，其中，call 了一个函数 `sub_1001516`。不放心的话，就进去看看。

`sub_1001516`：

```
.text:01001516 sub_1001516     proc near               ; CODE XREF: sub_1001B49+24p  

.text:01001516                                ; sub_1003CE5+9p  

.text:01001516                   xor     eax, eax  

.text:01001518                   cmp     word ptr dword_10056A0, ax  

.text:0100151F                   setz    al  

.text:01001522                   push    eax  

.text:01001523                   push    209h            ;521  

.text:01001528                   call    sub_1003CC4  

.text:0100152D                   xor     eax, eax  

.text:0100152F                   cmp     word ptr dword_10056A0, 1  

.text:01001537                   setz    al  

.text:0100153A                   push    eax  

.text:0100153B                   push    20Ah            ;522  

.text:01001540                   call    sub_1003CC4  

.text:01001545                   xor     eax, eax  

.text:01001547                   cmp     word ptr dword_10056A0, 2  

.text:0100154F                   setz    al  

.text:01001552                   push    eax  

.text:01001553                   push    20Bh            ;523  

.text:01001558                   call    sub_1003CC4  

.text:0100155D                   xor     eax, eax  

.text:0100155F                   cmp     word ptr dword_10056A0, 3  

.text:01001567                   setz    al  

.text:0100156A                   push    eax  

.text:0100156B                   push    20Ch            ;524  

.text:01001570                   call    sub_1003CC4  

.text:01001575                   push    dword_10056C8  

.text:0100157B                   push    211h            ;529  

.text:01001580                   call    sub_1003CC4  

.text:01001585                   push    Data  

.text:0100158B                   push    20Fh            ;527  

.text:01001590                   call    sub_1003CC4  

.text:01001595                   push    dword_10056B8  

.text:0100159B                   push    20Eh            ;526  

.text:010015A0                   call    sub_1003CC4  

.text:010015A5                   retn  

.text:010015A5 sub_1001516     endp  
```

发现每次都是压入一个数，然后 call `sub_1003CC4`。因为她的父函数是处理菜单的，因此，我们怀疑压入的数是控件 ID，用 resource hacker 看看。

果不其然：

```
500 MENU  

LANGUAGE LANG_ENGLISH, SUBLANG_ENGLISH_US  

{  

POPUP "&Game"  

{  

  MENUITEM "&New\tF2",  510  

  MENUITEM SEPARATOR  

  MENUITEM "&Beginner",  521  

  MENUITEM "&Intermediate",  522  

  MENUITEM "&Expert",  523  

  MENUITEM "&Custom...",  524  

  MENUITEM SEPARATOR  

  MENUITEM "&Marks (?)",  527  

  MENUITEM "Co&lor",  529  

  MENUITEM "&Sound",  526  

  MENUITEM SEPARATOR  

  MENUITEM "Best &Times...",  528  

  MENUITEM SEPARATOR  

  MENUITEM "E&xit",  512  

}  

POPUP "&Help"  

{  

  MENUITEM "&Contents\tF1",  590  

  MENUITEM "&Search for Help on...",  591  

  MENUITEM "Using &Help",  592  

  MENUITEM SEPARATOR  

  MENUITEM "&About Minesweeper...",  593  

}  

}  
```

是处理用户选的是初级，中级，高级之类的。所以，`sub_1003CE5` 也不是。就剩下 `sub_100367A` 了：

`sub_100367A` 函数很长，慢慢分析吧：

```
.text:0100367A sub_100367A     proc near               ; CODE XREF: sub_100140C+CAp  

.text:0100367A                                ; sub_1001B49+33j ...  

.text:0100367A                   mov     eax, dword_10056AC  

.text:0100367F                   mov     ecx, uValue  

.text:01003685                   push    ebx  

.text:01003686                   push    esi  

.text:01003687                   push    edi  

.text:01003688                   xor     edi, edi  

.text:0100368A                   cmp     eax, dword_1005334  

.text:01003690                   mov     dword_1005164, edi  

.text:01003696                   jnz     short loc_10036A4  

.text:01003698                   cmp     ecx, dword_1005338  

.text:0100369E                   jnz     short loc_10036A4  

.text:010036A0                   push    4  

.text:010036A2                   jmp     short loc_10036A6  
```

发现最终会进入 `loc_10036A6`，走着：

```
.text:010036A6 loc_10036A6:                  ; CODE XREF: sub_100367A+28j  

.text:010036A6                   pop     ebx  

.text:010036A7                   mov     dword_1005334, eax  

.text:010036AC                   mov     dword_1005338, ecx  

.text:010036B2                   call    sub_1002ED5  

.text:010036B7                   mov     eax, dword_10056A4  

.text:010036BC                   mov     dword_1005160, edi  

.text:010036C2                   mov     dword_1005330, eax  

.text:010036C7  

.text:010036C7 loc_10036C7:                  ; CODE XREF: sub_100367A+74j  

.text:010036C7                                ; sub_100367A+89j  

.text:010036C7                   push    dword_1005334  

.text:010036CD                   call    RandomReminder  

.text:010036D2                   push    dword_1005338  

.text:010036D8                   mov     esi, eax  

.text:010036DA                   inc     esi  

.text:010036DB                   call    RandomReminder  

.text:010036E0                   inc     eax  

.text:010036E1                   mov     ecx, eax  

.text:010036E3                   shl     ecx, 5  

.text:010036E6                   test    byte ptr dword_1005340[ecx+esi], 80h  

.text:010036EE                   jnz     short loc_10036C7  
```



```
.text:010036F0                   shl     eax, 5  

.text:010036F3                   lea     eax, dword_1005340[eax+esi]  

.text:010036FA                   or      byte ptr [eax], 80h  

.text:010036FD                   dec     dword_1005330  

.text:01003703                   jnz     short loc_10036C7  

.text:01003705                   mov     ecx, dword_1005338  

.text:0100370B                   imul    ecx, dword_1005334  

.text:01003712                   mov     eax, dword_10056A4  

.text:01003717                   sub     ecx, eax  

.text:01003719                   push    edi  

.text:0100371A                   mov     dword_100579C, edi  

.text:01003720                   mov     dword_1005330, eax  

.text:01003725                   mov     dword_1005194, eax  

.text:0100372A                   mov     dword_10057A4, edi  

.text:01003730                   mov     dword_10057A0, ecx  

.text:01003736                   mov     dword_1005000, 1  

.text:01003740                   call    sub_100346A  

.text:01003745                   push    ebx  

.text:01003746                   call    sub_1001950  

.text:0100374B                   pop     edi  

.text:0100374C                   pop     esi  

.text:0100374D                   pop     ebx  

.text:0100374E                   retn  

.text:0100374E sub_100367A      endp  
```

在这一段中，我们发现了 `Rand()` 函数，就在 `01003940`，我已经将她改名成了 `RandomReminder`。进入发现就是随机产生一个数，然后除以参数，返回余数，这个应该就是随机布雷，而且发现调用 2 次，很容易想到是分别产生 X 和 Y 坐标的。两次调用的参数分别放在 `dword_1005334` 和 `dword_1005338` 中，怀疑是雷区的大小，进入 OD，选择不同难度，然后断此函数看看参数，发现，果不其然，就是雷区大小。可以给 `dword_1005334` 和 `dword_1005338` 改名了，以后在遇到这两个参数就好识别了。接着走，发现这段话：

```
.text:010036E6                   test    byte ptr dword_1005340[ecx+esi], 80h  

.text:010036EE                   jnz     short loc_10036C7  

.text:010036F0                   shl     eax, 5  

.text:010036F3                   lea     eax, dword_1005340[eax+esi]  

.text:010036FA                   or      byte ptr [eax], 80h  
```

哈哈，`dword_1005340[ecx+esi]`，雷区地址。是这个样子的，`dword_1005340[Y*32 + X]`; `dword_1005330` 很明显是雷数。

OK，到这里就找到雷区了，比较麻烦，还是各位前辈的直接下断 `Rand()` 简单，但是我比较笨笨，一开始怕万一不是用 `Rand()` 怎么办呢，于是就自己分析看看。先写到这里，下次主要靠 OD 分析按钮事件。谢谢。![](img/755d44573ea0045d4eb19f2164c82d4e.jpg "扮鬼脸")

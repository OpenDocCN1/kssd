
# 加快 ODICE 鼠标滚动速度

> 原文：[`www.pediy.com/kssd/pediy10/65867.html`](https://www.pediy.com/kssd/pediy10/65867.html)

在使用 ODICE 时，滚动鼠标，感觉太慢了，每次只能滚动 2 行，想加快一点，但在 OD 的设置菜单中没有找到相应的设置，只有自己动手 DIY 了。

使用原版 OD 加载 ODICE，让程序执行，使用 OD 在 ODICE 的 ICPUASM 窗口下 `WM_MOUSEWHEEL` 消息断点，然后在 ODICE 中滚动鼠标轮，可惜没有中断成功。

再查手册，知道 MDI 窗口是使用 `WM_VSCROLL` 消息来滚动的，所以使用 OD 在 ODICE 的 ICPUASM 窗口下 `WM_VSCROLL` 消息断点，中断成功

```
0041E604  >  $  55            push    ebp            //中断在这里（会中断两次，说明 OD 每次滚动了两行）
0041E605   .  8BEC          mov     ebp, esp
0041E607   .  50            push    eax
0041E608   .  B8 03000000   mov     eax, 3
0041E60D   >  81C4 04F0FFFF add     esp, -0FFC
0041E613   .  50            push    eax
0041E614   .  48            dec     eax
0041E615   .^ 75 F6         jnz     short 0041E60D
0041E617   .  8B45 FC       mov     eax, [ebp-4]                     ;  Ollydbg3.0041E604
0041E61A   .  83C4 F0       add     esp, -10
0041E61D   .  53            push    ebx
0041E61E   .  56            push    esi                              ;  Ollydbg3.0041E604
```

一路按 F8 来到下面代码

```
0041E7A3   >  \8B45 14       mov     eax, [ebp+14]
0041E7A6   .  50            push    eax                              ; /Arg5 = 00110310
0041E7A7   .  8B55 10       mov     edx, [ebp+10]                    ; |
0041E7AA   .  52            push    edx                              ; |Arg4 = 00000000
0041E7AB   .  8B4D 0C       mov     ecx, [ebp+C]                     ; |
0041E7AE   .  51            push    ecx                              ; |Arg3 = 00000115
0041E7AF   .  8B45 08       mov     eax, [ebp+8]                     ; |
0041E7B2   .  50            push    eax                              ; |Arg2 = 00110310
0041E7B3   .  68 A8D64C00   push    004CD6A8                         ; |Arg1 = 004CD6A8
0041E7B8   .  E8 9F9B0300   call    _Tablefunction // OD 自己的_Tablefunction 函数，各个参数可查 OD 的手册得到,按 F7 进入
```

```
0045944F  |>  \817D 10 15010>cmp     dword ptr [ebp+10], 115         //比较是否 WM_VSCROLL 消息
00459456  |.  0F85 8B010000 jnz     004595E7
0045945C  |.  6A 10         push    10                              ; /Key = VK_SHIFT
0045945E  |.  E8 FF5F0500   call    <jmp.&USER32.GetKeyState>        //检测是否一起按下了 VK_SHIFT
00459463  |.  0FBFD0        movsx   edx, ax
00459466  |.  81E2 00800000 and     edx, 8000
0045946C  |.  8955 D4       mov     [ebp-2C], edx
0045946F  |.  66:8B4D 14    mov     cx, [ebp+14]
00459473  |.  66:81E1 FFFF  and     cx, 0FFFF
00459478  |.  0FB7C1        movzx   eax, cx                          //eax 为滚动方向，1 向上，0 向下
0045947B  |.  83F8 05       cmp     eax, 5                           ;  Switch (cases 0..5)
0045947E  |.  0F87 26010000 ja      004595AA
00459484  |.- FF2485 8B9445>jmp     [eax*4+45948B]                    //分支
0045948B  |.  E6944500      dd      Ollydbg3.004594E6                ;  分支表  被用于 00459484
0045948F  |.  A3944500      dd      Ollydbg3.004594A3
00459493  |.  2D954500      dd      Ollydbg3.0045952D
00459497  |.  56954500      dd      Ollydbg3.00459556
0045949B  |.  AA954500      dd      Ollydbg3.004595AA
0045949F  |.  7D954500      dd      Ollydbg3.0045957D
```

一路按 F7 来到下面代码

```
004594C7  |.  /E9 E1000000   jmp     004595AD
004594CC     |53            push    ebx                             //向下一次滚动的行数（默认 1）
004594CD     |8B45 FC       mov     eax, [ebp-4]
004594D0     |50            push    eax
004594D1  |.  |68 68040000   push    468                             ; |Message = MSG(468)
004594D6  |.  |8B55 0C       mov     edx, [ebp+C]                    ; |
004594D9  |.  |52            push    edx                             ; |hWnd = NULL
004594DA  |.  |E8 91600500   call    <jmp.&USER32.SendMessageA>      ; \\SendMessageA
004594DF  |.  |8BD8          mov     ebx, eax
004594E1  |.  |E9 C7000000   jmp     004595AD
004594E6  |>  |837D D4 00    cmp     dword ptr [ebp-2C], 0           ;  Case 0 of switch 0045947B
004594EA  |.  |74 25         je      short 00459511
004594EC  |.  |F687 35020000>test    byte ptr [edi+235], 80
004594F3  |.  |74 1C         je      short 00459511
004594F5  |.  |F7DB          neg     ebx
004594F7  |.  |53            push    ebx                             ; /lParam = FFFFFFFE
004594F8  |.  |8B4D FC       mov     ecx, [ebp-4]                    ; |
004594FB  |.  |51            push    ecx                             ; |wParam = 110310
004594FC  |.  |68 6A040000   push    46A                             ; |Message = MSG(46A)
00459501  |.  |8B45 0C       mov     eax, [ebp+C]                    ; |
00459504  |.  |50            push    eax                             ; |hWnd = NULL
00459505  |.  |E8 66600500   call    <jmp.&USER32.SendMessageA>      ; \\SendMessageA
0045950A  |.  |8BD8          mov     ebx, eax
0045950C  |.  |E9 9C000000   jmp     004595AD
00459511     |F7DB          neg     ebx
00459513     |53            push    ebx                              // 向上一次滚动的行数（默认-1）
00459514     |8B55 FC       mov     edx, [ebp-4]
00459517     |52            push    edx
00459518  |.  |68 68040000   push    468                             ; |Message = MSG(468)
0045951D  |.  |8B4D 0C       mov     ecx, [ebp+C]                    ; |
00459520  |.  |51            push    ecx                             ; |hWnd = 110310
00459521  |.  |E8 4A600500   call    <jmp.&USER32.SendMessageA>      ; \\SendMessageA
```

**********************************************************************************************************************************************************************************************************

分析完就动手开始 DIY 了，

```
004594C7  |.  /E9 E1000000   jmp     004595AD
004594CC     |6A 02         push    2                               //默认为 1，我就改为 2，增加一倍
004594CE     |FF75 FC       push    dword ptr [ebp-4]               //执行原来代码
004594D1  |.  |68 68040000   push    468                             ; |Message = MSG(468)
004594D6  |.  |8B55 0C       mov     edx, [ebp+C]                    ; |
004594D9  |.  |52            push    edx                             ; |hWnd = NULL
004594DA  |.  |E8 91600500   call    <jmp.&USER32.SendMessageA>      ; \\SendMessageA
004594DF  |.  |8BD8          mov     ebx, eax
004594E1  |.  |E9 C7000000   jmp     004595AD
004594E6  |>  |837D D4 00    cmp     dword ptr [ebp-2C], 0           ;  Case 0 of switch 0045947B
004594EA  |.  |74 25         je      short 00459511
004594EC  |.  |F687 35020000>test    byte ptr [edi+235], 80
004594F3  |.  |74 1C         je      short 00459511
004594F5  |.  |F7DB          neg     ebx
004594F7  |.  |53            push    ebx                             ; /lParam = FFFFFFFE
004594F8  |.  |8B4D FC       mov     ecx, [ebp-4]                    ; |
004594FB  |.  |51            push    ecx                             ; |wParam = 110310
004594FC  |.  |68 6A040000   push    46A                             ; |Message = MSG(46A)
00459501  |.  |8B45 0C       mov     eax, [ebp+C]                    ; |
00459504  |.  |50            push    eax                             ; |hWnd = NULL
00459505  |.  |E8 66600500   call    <jmp.&USER32.SendMessageA>      ; \\SendMessageA
0045950A  |.  |8BD8          mov     ebx, eax
0045950C  |.  |E9 9C000000   jmp     004595AD
00459511     |43            inc     ebx                              //默认为 1，我就改为 2，增加一倍
00459512     |F7DB          neg     ebx
00459514     |53            push    ebx
00459515     |FF75 FC       push    dword ptr [ebp-4]
00459518  |.  |68 68040000   push    468                             ; |Message = MSG(468)
0045951D  |.  |8B4D 0C       mov     ecx, [ebp+C]                    ; |
00459520  |.  |51            push    ecx                             ; |hWnd = 110310
```



`00459521`  `|. |E8 4A600500   call    <jmp.&USER32.SendMessageA>       ; \\SendMessageA`

修改后使用 `ODICE` 就快了一倍，一次能滚动 4 行了，如感觉不够快，自己根据需要再改更大就可以了。

最后还是没有找到 `OD` 是怎样处理 `WM_MOUSEWHEEL` 消息，哪位兄弟如找到，请指点一下

********************************************************************************

再认真分析一下，找到了处理 `WM_MOUSEWHEEL` 消息的地方

```
00439349   > \\81BD 3CF6FFFF>cmp     dword ptr [ebp-9C4], 20A            //比较鼠标 WM_MOUSEWHEEL 消息
00439353   .  0F85 D1000000  jnz     0043942A
00439359   .  66:8B8D 44F6F>mov     cx, [ebp-9BC]
00439360   .  66:81E1 FFFF  and     cx, 0FFFF
00439365   .  0FB7C1        movzx   eax, cx
00439368   .  8945 A4       mov     [ebp-5C], eax
0043936B   .  8B95 44F6FFFF mov     edx, [ebp-9BC]
00439371   .  C1EA 10       shr     edx, 10
00439374   .  0FB7CA        movzx   ecx, dx
00439377   .  894D A8       mov     [ebp-58], ecx
0043937A   .  FF75 A8       push    dword ptr [ebp-58]               ; /pt.Y = 99 (153.)
0043937D   .  FF75 A4       push    dword ptr [ebp-5C]               ; |pt.X = D2 (210.)
00439380   .  E8 7B620700   call    <jmp.&USER32.WindowFromPoint>    ; \\WindowFromPoint
00439385   .  8BF8          mov     edi, eax
00439387   .  3B3D 803B4D00 cmp     edi, [4D3B80]
0043938D   .  75 08         jnz     short 00439397
0043938F   .  8BBD 38F6FFFF mov     edi, [ebp-9C8]
00439395   .  EB 20         jmp     short 004393B7
00439397   >  8BC7          mov     eax, edi
00439399   .  EB 06         jmp     short 004393A1
0043939B   >  50           push    eax                              ; /hWnd = FFFFFFFE
0043939C   .  E8 DF600700   call    <jmp.&USER32.GetParent>          ; \\GetParent
004393A1   >  3B05 7C3B4D00 cmp     eax, [4D3B7C]                    ;  Ollydbg3.004001E0
004393A7   .  74 04         je      short 004393AD
004393A9   .  85C0          test    eax, eax
004393AB   .^ 75 EE         jnz     short 0043939B
004393AD   >  85C0          test    eax, eax
004393AF   .  75 06         jnz     short 004393B7
004393B1   .  8BBD 38F6FFFF mov     edi, [ebp-9C8]
004393B7   >  8B85 40F6FFFF mov     eax, [ebp-9C0]                  //取出 WM_MOUSEWHEEL 消息参数
004393BD   .  B9 78000000   mov     ecx, 78                        //想加快滚动可修改这里，例如改成 mov ecx,28 就能一次滚动 6 行
004393C2   .  C1E8 10       shr     eax, 10
004393C5   .  0FBFC0        movsx   eax, ax
004393C8   .  03C0          add     eax, eax
004393CA   .  99           cdq
004393CB   .  F7F9          idiv    ecx
004393CD   .  F685 40F6FFFF>test    byte ptr [ebp-9C0], 10
004393D4   .  8BD8          mov     ebx, eax
004393D6   .  74 05         je      short 004393DD
004393D8   .  8D045B        lea     eax, [ebx+ebx*2]
004393DB   .  89C3          mov     ebx, eax
004393DD   >  83FB F0       cmp     ebx, -10
004393E0   .  7D 07         jge     short 004393E9
004393E2   .  BB F0FFFFFF   mov     ebx, -10
004393E7   .  EB 0A         jmp     short 004393F3
004393E9   >  83FB 10       cmp     ebx, 10
004393EC   .  7E 05         jle     short 004393F3
004393EE   .  BB 10000000   mov     ebx, 10
004393F3   >  85DB          test    ebx, ebx
004393F5   .^ 0F84 7CFCFFFF je      00439077
004393FB   >  85DB          test    ebx, ebx                         //ebx 为滚动方向，2 向上，-2 向下
004393FD   .  7D 12         jge     short 00439411
004393FF   .  6A 00         push    0                                ; /lParam = 0
00439401   .  6A 01         push    1                                ; |wParam = 1  
00439403   .  68 15010000   push    115                              ; |Message = WM_VSCROLL
00439408   .  57           push    edi                              ; |hWnd = 3302F0
00439409   .  E8 26610700   call    <jmp.&USER32.PostMessageA>       ; \\PostMessageA
0043940E   .  43           inc     ebx
0043940F   .  EB 10         jmp     short 00439421
00439411   >  6A 00         push    0                                ; /lParam = 0
00439413   .  6A 00         push    0                                ; |wParam = 0
00439415   .  68 15010000   push    115                              ; |Message = WM_VSCROLL
0043941A   .  57           push    edi                              ; |hWnd = 3302F0
0043941B   .  E8 14610700   call    <jmp.&USER32.PostMessageA>       ; \\PostMessageA
00439420   .  4B           dec     ebx
00439421   >  85DB          test    ebx, ebx                         //检测是否已经滚动了 2 次
00439423   .^\\75 D6       jnz     short 004393FB                   //不够两次就再次跳回去滚动
00439425   .^ E9 4DFCFFFF  jmp     00439077
```

```
004595FF        |.  8945 CC       mov     dword ptr [ebp-34], eax
00459602        |.  F645 CE 20    test    byte ptr [ebp-32], 20
00459606        |.  74 54         je      short 0045965C
00459608        |.  6A 00         push    0                                    ; /Redraw = FALSE
0045960A        |.  68 00400000   push    4000                                 ; |MaxPos = 4000 (16384.)
0045960F        |.  6A 00         push    0                                    ; |MinPos = 0
00459611        |.  6A 01         push    1                                    ; |BarType = SB_VERT
00459613        |.  8B45 0C       mov     eax, dword ptr [ebp+C]               ; |
00459616        |.  50           push    eax                                  ; |hWnd
00459617        |.  E8 8A5F0500   call    <jmp.&USER32.SetScrollRange>         ; \\SetScrollRange
0045961C        |.  6A 00         push    0                                    ; /lParam = 0
0045961E        |.  8B55 FC       mov     edx, dword ptr [ebp-4]               ; |
00459621        |.  52           push    edx                                  ; |wParam
00459622        |.  68 68040000   push    468                                 ; |Message = MSG(468)
00459627        |.  8B4D 0C       mov     ecx, dword ptr [ebp+C]               ; |
0045962A        |.  51           push    ecx                                  ; |hWnd
0045962B        |.  E8 405F0500   call    <jmp.&USER32.SendMessageA>           ; \\SendMessageA
00459630        |.  8BF0          mov     esi, eax
00459632        |.  85DB          test    ebx, ebx
00459634        |.  7D 09         jge     short 0045963F
00459636        |.  B8 00400000   mov     eax, 4000
0045963B        |.  2BC6          sub     eax, esi
0045963D        |.  8BF0          mov     esi, eax
0045963F        |>  6A 01         push    1                                    ; /BarType = SB_VERT
00459641        |.  8B55 0C       mov     edx, dword ptr [ebp+C]               ; |
00459644        |.  52           push    edx                                  ; |hWnd
00459645        |.  E8 3C5E0500   call    <jmp.&USER32.GetScrollPos>           ; \\GetScrollPos
0045964A        |.  3BF0          cmp     esi, eax
0045964C        |.  74 0E         je      short 0045965C
0045964E        |.  6A 01         push    1                                    ; /Redraw = TRUE
00459650        |.  56           push    esi                                  ; |Pos
00459651        |.  6A 01         push    1                                    ; |BarType = SB_VERT
00459653        |.  8B4D 0C       mov     ecx, dword ptr [ebp+C]               ; |
00459656        |.  51           push    ecx                                  ; |hWnd
00459657        |.  E8 445F0500   call    <jmp.&USER32.SetScrollPos>           ; \\SetScrollPos
0045965C        |>  F645 CE 10    test    byte ptr [ebp-32], 10
00459660        |.  0F84 5F0C0000 je      0045A2C5
00459666        |.  6A 00         push    0                                    ; /Redraw = FALSE
```



```
00459668        |.  68 00400000   push    4000                                 ; |MaxPos = 4000 (16384.)  
0045966D        |.  6A 00         push    0                                    ; |MinPos = 0  
0045966F        |.  6A 00         push    0                                    ; |BarType = SB\_HORZ  
00459671        |.  8B45 0C       mov     eax, dword ptr \[ebp+C\]               ; |  
00459674        |.  50            push    eax                                  ; |hWnd  
00459675        |.  E8 2C5F0500   call    <jmp.&USER32.SetScrollRange>         ; \\SetScrollRange  
0045967A        |.  8B95 64FEFFFF mov     edx, dword ptr \[ebp-19C\]  
00459680        |.  F7DA          neg     edx  
00459682        |.  8955 F0       mov     dword ptr \[ebp-10\], edx  
00459685        |.  33F6          xor     esi, esi  
00459687        |.  8B45 C8       mov     eax, dword ptr \[ebp-38\]  
0045968A        |.  8D50 08       lea     edx, dword ptr \[eax+8\]  
0045968D        |.  8955 8C       mov     dword ptr \[ebp-74\], edx  
00459690        |.  EB 0D         jmp     short 0045969F  
00459692        |>  8B4D 8C       /mov     ecx, dword ptr \[ebp-74\]  
00459695        |.  8B01          |mov     eax, dword ptr \[ecx\]  
00459697        |.  0145 F0       |add     dword ptr \[ebp-10\], eax  
0045969A        |.  46            |inc     esi  
0045969B        |.  8345 8C 04    |add     dword ptr \[ebp-74\], 4  
0045969F        |>  8B55 C8        mov     edx, dword ptr \[ebp-38\]  
004596A2        |.  3B32          |cmp     esi, dword ptr \[edx\]  
004596A4        |.^ 7C EC         \\jl      short 00459692  
004596A6        |.  6A 01         push    1                                    ; /Redraw = TRUE  
004596A8        |.  8B4D F0       mov     ecx, dword ptr \[ebp-10\]              ; |  
004596AB        |.  51            push    ecx                                  ; |/Divisor  
004596AC        |.  68 00400000   push    4000                                 ; ||Multiplier = 4000 (16384.)  
004596B1        |.  8B87 45020000 mov     eax, dword ptr \[edi+245\]             ; ||  
004596B7        |.  50            push    eax                                  ; ||Multiplicand  
004596B8        |.  E8 935A0500   call    <jmp.&KERNEL32.MulDiv>               ; |\\MulDiv  
004596BD        |.  50            push    eax                                  ; |Pos  
004596BE        |.  6A 00         push    0                                    ; |BarType = SB\_HORZ  
004596C0        |.  8B55 0C       mov     edx, dword ptr \[ebp+C\]               ; |  
004596C3        |.  52            push    edx                                  ; |hWnd  
004596C4        |.  E8 D75E0500   call    <jmp.&USER32.SetScrollPos>           ; \\SetScrollPos  
004596C9        |.  E9 F70B0000   jmp     0045A2C5
```

Trace 出来的，没细看。

引用: *最初由 `Bughoho`发布* ![查看帖子](img/78d57e1ac1f1fc075f84e78a6a789d46.jpg)004595FF        |.  8945 CC       mov     dword ptr \[ebp-34\], eax  

00459602        |.  F645 CE 20    test    byte ptr \[ebp-32\], 20  

00459606        |.  ... 

这个我也分析过，`SetScrollPos` 函数只是设置滚动条的滑动块位置，不能滚动窗口。

```
00459645        |.  E8 3C5E0500   call    <jmp.&USER32.GetScrollPos>           ; \\GetScrollPos  
0045964A        |.  3BF0          cmp     esi, eax                                                //比较新旧位置  
0045964C        |.  74 0E         je      short 0045965C                                  //没有滚动就相等并跳  
0045964E        |.  6A 01         push    1                                    ; /Redraw = TRUE  
00459650        |.  56            push    esi                                  ; |Pos  
00459651        |.  6A 01         push    1                                    ; |BarType = SB\_VERT  
00459653        |.  8B4D 0C       mov     ecx, dword ptr \[ebp+C\]               ; |  
00459656        |.  51            push    ecx                                  ; |hWnd  
00459657        |.  E8 445F0500   call    <jmp.&USER32.SetScrollPos>           ; \\SetScrollPos
```

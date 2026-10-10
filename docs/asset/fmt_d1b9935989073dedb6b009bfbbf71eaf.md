
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

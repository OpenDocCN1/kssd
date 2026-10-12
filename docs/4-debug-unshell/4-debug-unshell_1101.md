
# Themida Logo 显示分析

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-553.htm`](https://www.pediy.com/kssd/pediy08/pediy8-553.htm)

作者: `KernelKiller`

目标: 分析 `Themida` 如何显示 Logo 窗口

系统平台: `Windows 2000 Server`

软件: `Themida Demo 1.0.0.5 driver version`

日期: `2006-05-01`

声明: 本文只是技术探讨。

为了不浪费大家宝贵时间，直接进入正题，省略加壳后分析的 Logo 代码，直接来到主程序内代码。

```
// 创建一个线程 显示 Logo 窗口

___:0057C98F Themida_Show_Logo proc near           ; CODE XREF: ___:0057B145p
___:0057C98F                                      ; ___:0057B2A5p
___:0057C98F                                      ; DATA XREF: ...
___:0057C98F               mov     ss:dword_57C687[ebp], esi
___:0057C995               lea     eax, Themida_Show_Logo[ebp]
___:0057C99B               push    eax
___:0057C99C               push    0
___:0057C99E               push    0
___:0057C9A0               lea     eax, Themida_Show_Logo_Fun[ebp]
___:0057C9A6               push    eax
___:0057C9A7               push    0
___:0057C9A9               push    0
___:0057C9AB               call    ss:Themida_Call_CreateThread[ebp]
___:0057C9B1               lea     ebx, loc_580539[ebp]
___:0057C9B7               call    ebx
___:0057C9B9               push    28h
___:0057C9BB               call    ss:Themida_Call_Sleep[ebp]
___:0057C9C1               retn
___:0057C9C1 Themida_Show_Logo endp

// 主显示 Logo 功能.

___:0057C9C2 Themida_Show_Logo_Fun:                ; DATA XREF: Themida_Show_Logo+11r
___:0057C9C2               call    $+5
___:0057C9C7               pop     ebp
___:0057C9C8               sub     ebp, 57C9C7h
___:0057C9CE               inc     ss:dword_58B7BC[ebp]
___:0057C9D4               mov     esi, ss:dword_57C687[ebp]
___:0057C9DA               push    ebp
___:0057C9DB               mov     ebp, esp
___:0057C9DD               add     esp, 0FFFFFFB4h
___:0057C9E0               call    $+5
___:0057C9E5
___:0057C9E5 loc_57C9E5:                      ; DATA XREF: ___:0057C9E6o
___:0057C9E5               pop     edx
___:0057C9E6               sub     edx, offset loc_57C9E5
___:0057C9EC               mov     dword_57B5B9[edx], esi
___:0057C9F2               call    sub_57CD95
___:0057C9F7               lea     ebx, dword_57B5CF[edx]
___:0057C9FD               mov     eax, [ebx+4]
___:0057CA00               mov     eax, [ebx+8]
___:0057CA03               mov     dword ptr [ebp-30h], 30h
___:0057CA0A               mov     dword ptr [ebp-2Ch], 3
___:0057CA11               lea     eax, WindowsProc[edx]
___:0057CA17               mov     [ebp-28h], eax
___:0057CA1A               mov     dword ptr [ebp-24h], 0
___:0057CA21               mov     dword ptr [ebp-20h], 0
___:0057CA28               push    dword_58A78C[edx]
___:0057CA2E               pop     dword ptr [ebp-1Ch]
___:0057CA31               mov     dword ptr [ebp-10h], 6
___:0057CA38               mov     dword ptr [ebp-0Ch], 0
___:0057CA3F               lea     eax, unk_57B548[edx]
___:0057CA45               mov     [ebp-8], eax
___:0057CA48               mov     dword ptr [ebp-18h], 0
___:0057CA4F               mov     dword ptr [ebp-4], 0
___:0057CA56               push    edx
___:0057CA57               push    7F89h
___:0057CA5C               push    0
___:0057CA5E               call    dword_57B59D[edx]
___:0057CA64               pop     edx
___:0057CA65               mov     [ebp-14h], eax
___:0057CA68               push    edx
___:0057CA69               lea     eax, [ebp-30h]
___:0057CA6C               push    eax
___:0057CA6D               call    dword_57B559[edx]
___:0057CA73               pop     edx
___:0057CA74               push    edx
___:0057CA75               lea     ebx, dword_57B5CF[edx]
___:0057CA7B               push    0
___:0057CA7D               push    dword_58A78C[edx]
___:0057CA83               push    0
___:0057CA85               push    0
___:0057CA87               push    dword ptr [ebx+8]
___:0057CA8A               push    dword ptr [ebx+4]
___:0057CA8D               push    80000000h
___:0057CA92               push    80000000h
___:0057CA97               push    80000000h
___:0057CA9C               push    0
___:0057CA9E               lea     eax, unk_57B548[edx]
___:0057CAA4               push    eax
___:0057CAA5               push    0
___:0057CAA7               call    Themida_Call_CreateWindows[edx]
___:0057CAAD               pop     edx
___:0057CAAE               mov     hLogoWindows[edx], eax
___:0057CAB4               push    edx
___:0057CAB5               push    hLogoWindows[edx]
___:0057CABB               call    dword_57B555[edx]
___:0057CAC1               pop     edx
___:0057CAC2               mov     dword_57B5BD[edx], eax
___:0057CAC8               push    dword_57B5BD[edx]
___:0057CACE               call    sub_57CDB8
___:0057CAD3               push    edx
___:0057CAD4               push    1
___:0057CAD6               push    hLogoWindows[edx]
___:0057CADC               call    Themida_Call_ShowWindows[edx]
___:0057CAE2               pop     edx
___:0057CAE3               push    edx
___:0057CAE4               push    hLogoWindows[edx]
___:0057CAEA               call    Themida_Call_UpdateWindow[edx]
___:0057CAF0               pop     edx
___:0057CAF1
___:0057CAF1 loc_57CAF1:                      ; CODE XREF: ___:0057CB1Fj
___:0057CAF1               push    edx
___:0057CAF2               push    0
___:0057CAF4               push    0
___:0057CAF6               push    0
___:0057CAF8               lea     eax, [ebp-4Ch]
___:0057CAFB               push    eax
___:0057CAFC               call    dword_57B569[edx]
___:0057CB02               pop     edx
___:0057CB03               or      eax, eax
___:0057CB05               jz      short loc_57CB21
___:0057CB07               push    edx
___:0057CB08               lea     eax, [ebp-4Ch]
___:0057CB0B               push    eax
___:0057CB0C               call    dword_57B56D[edx]
___:0057CB12               pop     edx
___:0057CB13               push    edx
___:0057CB14               lea     eax, [ebp-4Ch]
___:0057CB17               push    eax
___:0057CB18               call    dword_57B571[edx]
___:0057CB1E               pop     edx
___:0057CB1F               jmp     short loc_57CAF1
___:0057CB21 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
___:0057CB21
___:0057CB21 loc_57CB21:                      ; CODE XREF: ___:0057CB05j
___:0057CB21               push    edx
___:0057CB22               push    dword_57B5BD[edx]
___:0057CB28               push    hLogoWindows[edx]
___:0057CB2E               call    dword_57B575[edx]
___:0057CB34               pop     edx
___:0057CB35               mov     ebp, edx
___:0057CB37               dec     dword_58B7BC[edx]
___:0057CB3D               mov     byte ptr dword_58C4EC[edx], 45h
```



```
__::0057CB44                  push    0DA6038C2h  

__::0057CB49                  push    dword_58AA90[edx]  

__::0057CB4F                  lea     eax, loc_57FAE9[edx]  

__::0057CB55                  call    eax  

__::0057CB57                  push    0  

__::0057CB59                  call    eax  

__::0057CB5B  

__::0057CB5B WindowsProc:                          ; DATA XREF: __::0057CA11r  

__::0057CB5B                  push    ebp  

__::0057CB5C                  mov     ebp, esp  

__::0057CB5E                  add     esp, 0FFFFFFB4h  

__::0057CB61                  push    edx  

__::0057CB62                  call    $+5  

__::0057CB67  

__::0057CB67 loc_57CB67:                       ; DATA XREF: __::0057CB68o  

__::0057CB67                  pop     edx  

__::0057CB68                  sub     edx, offset loc_57CB67  

__::0057CB6E                  cmp     dword ptr [ebp+0Ch], 2  

__::0057CB72                  jnz     short Local_1  

__::0057CB74                  push    edx  

__::0057CB75                  mov     Themida_Exit_Logo_Flag[edx], 1  

__::0057CB7F                  push    0  

__::0057CB81                  call    dword_57B579[edx]  

__::0057CB87                  pop     edx  

__::0057CB88                  jmp     Exit  

__::0057CB8D ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  

__::0057CB8D  

__::0057CB8D Local_1:                         ; CODE XREF: __::0057CB72j  

__::0057CB8D                  cmp     dword ptr [ebp+0Ch], WM_LBUTTONDOWN  

__::0057CB94                  jnz     short Local_2   //鼠标按下  

__::0057CB96                  mov     Themida_Exit_Logo_Flag[edx], 1  

__::0057CBA0                  jmp     Exit  

__::0057CBA5 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  

__::0057CBA5  

__::0057CBA5 Local_2:                         ; CODE XREF: __::0057CB94j  

__::0057CBA5                  cmp     dword ptr [ebp+0Ch], 1  

__::0057CBA9                  jnz     short Local_3  

__::0057CBAB                  push    edx  

__::0057CBAC                  lea     eax, [ebp-3Ch]  

__::0057CBAF                  push    eax  

__::0057CBB0                  push    dword ptr [ebp+8]  

__::0057CBB3                  call    dword_57B57D[edx]  

__::0057CBB9                  pop     edx  

__::0057CBBA                  push    edx  

__::0057CBBB                  call    dword_57B581[edx]  

__::0057CBC1                  pop     edx  

__::0057CBC2                  mov     ecx, eax  

__::0057CBC4                  push    edx  

__::0057CBC5                  lea     eax, [ebp-4Ch]  

__::0057CBC8                  push    eax  

__::0057CBC9                  push    ecx  

__::0057CBCA                  call    dword_57B57D[edx]  

__::0057CBD0                  pop     edx  

__::0057CBD1                  push    edx  

__::0057CBD2                  push    0  

__::0057CBD4                  mov     eax, [ebp-30h]  

__::0057CBD7                  sub     eax, [ebp-38h]  

__::0057CBDA                  mov     [ebp-28h], eax  

__::0057CBDD                  push    eax  

__::0057CBDE                  mov     eax, [ebp-34h]  

__::0057CBE1                  sub     eax, [ebp-3Ch]  

__::0057CBE4                  mov     [ebp-2Ch], eax  

__::0057CBE7                  push    eax  

__::0057CBE8                  mov     eax, [ebp-40h]  

__::0057CBEB                  sub     eax, [ebp-28h]  

__::0057CBEE                  shr     eax, 1  

__::0057CBF0                  push    eax  

__::0057CBF1                  mov     eax, [ebp-44h]  

__::0057CBF4                  sub     eax, [ebp-2Ch]  

__::0057CBF7                  shr     eax, 1  

__::0057CBF9                  push    eax  

__::0057CBFA                  push    dword ptr [ebp+8]  

__::0057CBFD                  call    dword_57B585[edx]  

__::0057CC03                  pop     edx  

__::0057CC04                  jmp     short Exit  

__::0057CC06 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  

__::0057CC06  

__::0057CC06 Local_3:                         ; CODE XREF: __::0057CBA9j  

__::0057CC06                  cmp     dword ptr [ebp+0Ch], 0Fh  

__::0057CC0A                  jnz     short RetSysMessage  

__::0057CC0C                  push    edx  

__::0057CC0D                  lea     eax, dword_57C643[edx]  

__::0057CC13                  push    eax  

__::0057CC14                  push    dword ptr [ebp+8]  

__::0057CC17                  call    dword_57B589[edx]  

__::0057CC1D                  pop     edx  

__::0057CC1E                  push    edx  

__::0057CC1F                  push    0CC0020h  

__::0057CC24                  push    0  

__::0057CC26                  push    0  

__::0057CC28                  push    dword_57B60B[edx]  

__::0057CC2E                  push    dword_57B5FB[edx]  

__::0057CC34                  push    dword_57B5F7[edx]  

__::0057CC3A                  push    0  

__::0057CC3C                  push    0  

__::0057CC3E                  push    dword_57B5BD[edx]  

__::0057CC44                  call    dword_57B58D[edx]  

__::0057CC4A                  pop     edx  

__::0057CC4B                  push    edx  

__::0057CC4C                  lea     eax, dword_57C643[edx]  

__::0057CC52                  push    eax  

__::0057CC53                  push    dword ptr [ebp+8]  

__::0057CC56                  call    dword_57B591[edx]  

__::0057CC5C                  pop     edx  

__::0057CC5D                  jmp     short Exit  

__::0057CC5F ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  

__::0057CC5F  

__::0057CC5F RetSysMessage:                   ; CODE XREF: __::0057CC0Aj  

__::0057CC5F                  push    edx  

__::0057CC60                  push    dword ptr [ebp+14h]  

__::0057CC63                  push    dword ptr [ebp+10h]  

__::0057CC66                  push    dword ptr [ebp+0Ch]  

__::0057CC69                  push    dword ptr [ebp+8]  

__::0057CC6C                  call    dword_57B595[edx]  

__::0057CC72                  pop     edx  

__::0057CC73                  pop     edx  

__::0057CC74                  leave  

__::0057CC75                  retn    10h  

__::0057CC78 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  

__::0057CC78  

__::0057CC78 Exit:                          ; CODE XREF: __::0057CB88j  

__::0057CC78                              ; __::0057CBA0j ...  

__::0057CC78                  pop     edx  

__::0057CC79                  xor     eax, eax  

__::0057CC7B                  leave  

__::0057CC7C                  retn    10h  
```

//等待退出 Logo
```
__::0057B3BD loc_57B3BD:                       ; DATA XREF: sub_57B473+Do  

__::0057B3BD                              ; sub_57B473+56o ...  

__::0057B3BD                  mov     eax, 0  

__::0057B3C2                  push    eax  

__::0057B3C3                  cmp     eax, 2  

__::0057B3C6                  jnz     short loc_57B3DB  

__::0057B3C8                  jmp     short loc_57B3D2  

__::0057B3CA ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  

__::0057B3CA  

__::0057B3CA Themida_Loop_Wati_Logo_Exit:    ; CODE XREF: __::0057B3D9j  

__::0057B3CA                  push    50  

__::0057B3CC                  call    ss:Themida_Call_Sleep[ebp]  

__::0057B3D2  

__::0057B3D2 loc_57B3D2:                       ; CODE XREF: __::0057B3C8j  

__::0057B3D2                  cmp     ss:Themida_Exit_Logo_Flag[ebp], 0  
```



`____:0057B3D9                    jz      short Themida_Loop_Wati_Logo_Exit`

显示 Logo 过程是这样：`等待`->`创建 Logo`->`等待用户鼠标点击`->`结束 Logo`->`退出等待`->`继续...`

`crack`

`code->call Themida_Show_Logo`

`crack->`

`nop`

`nop`

`nop`

`///////////////////////////////`

`code->`

`Themida_Loop_Wati_Logo_Exit:`

`push    50`

`call    ss:Themida_Call_Sleep[ebp]`

`loc_57B3D2:`

`cmp     ss:Themida_Exit_Logo_Flag[ebp], 0`

`jz      short Themida_Loop_Wati_Logo_Exit`

`crack->`

`nop`

`        ...`

全文完。

`4dd466d0e1e0cc123afbe504da2462ca`

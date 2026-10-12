
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


```
.text:101AD3AA                 mov     edi, edi
.text:101AD3AC                 push    ebp
.text:101AD3AD                 mov     ebp, esp
.text:101AD3AF                 sub     esp, 0Ch
.text:101AD3B2                 push    ebx
.text:101AD3B3                 push    esi
.text:101AD3B4                 push    edi
.text:101AD3B5                 lea     edx, \[ebp+pUnk\]
.text:101AD3B8                 push    edx
.text:101AD3B9                 push    offset stru\_100D5E20
.text:101AD3BE                 mov     edi, ecx
.text:101AD3C0                 mov     ecx, \[eax\]
.text:101AD3C2                 push    eax
.text:101AD3C3                 call    dword ptr \[ecx\]
.text:101AD3C5                 xor     ebx, ebx
.text:101AD3C7                 mov     \[ebp+var\_4\], eax
.text:101AD3CA                 cmp     eax, ebx
.text:101AD3CC                 jl      loc\_101AD46D
.text:101AD3D2                 push    3               ; MaxCount
.text:101AD3D4                 push    offset aMk      ; "mk:"
.text:101AD3D9                 push    \[ebp+szURL\]     ; Str1
.text:101AD3DC                 call    ds:\_\_imp\_\_wcsnicmp
.text:101AD3E2                 add     esp, 0Ch
.text:101AD3E5                 test    eax, eax
.text:101AD3E7                 jnz     short loc\_101AD41B
.text:101AD3E9                 mov     eax, \[ebp+pUnk\]
.text:101AD3EC                 mov     ecx, \[eax\]
.text:101AD3EE                 push    eax
.text:101AD3EF                 call    dword ptr \[ecx+8\]
.text:101AD3F2                 push    ebx             ; reserved
.text:101AD3F3                 lea     eax, \[ebp+pUnk\]
.text:101AD3F6                 push    eax             ; ppBC
.text:101AD3F7                 push    ebx             ; pEnum
.text:101AD3F8                 push    ebx             ; pBSCb
.text:101AD3F9                 push    ebx             ; dwOptions
.text:101AD3FA                 push    ebx             ; pbc
.text:101AD3FB                 mov     \[ebp+pUnk\], ebx
.text:101AD3FE                 call    CreateAsyncBindCtxEx
.text:101AD404                 mov     esi, \[edi\]
.text:101AD406                 mov     \[ebp+var\_4\], eax
.text:101AD409                 cmp     esi, ebx
.text:101AD40B                 jz      short loc\_101AD41B
.text:101AD40D                 call    sub\_10347BB6
.text:101AD412                 push    esi             ; lpMem
.text:101AD413                 call    sub\_100135D9
.text:101AD418                 pop     ecx
.text:101AD419                 mov     \[edi\], ebx
.text:101AD41B
.text:101AD41B loc\_101AD41B:                           ; CODE XREF: sub\_101AD3AA+3Dj
.text:101AD41B                                         ; sub\_101AD3AA+61j
.text:101AD41B                 cmp     \[ebp+var\_4\], ebx
.text:101AD41E                 jl      short loc\_101AD46D
.text:101AD420                 mov     eax, \[edi\]
.text:101AD422                 cmp     eax, ebx
.text:101AD424                 jz      short loc\_101AD430
.text:101AD426                 mov     ecx, \[ebp+arg\_0\]
.text:101AD429                 mov     \[ecx+18h\], eax
.text:101AD42C                 mov     \[edi\], ebx
.text:101AD42E                 jmp     short loc\_101AD464
.text:101AD430 ; ---------------------------------------------------------------------------
.text:101AD430
.text:101AD430 loc\_101AD430:                           ; CODE XREF: sub\_101AD3AA+7Aj
.text:101AD430                 push    4               ; dwBytes
.text:101AD432                 call    sub\_1000E771
.text:101AD437                 pop     ecx
.text:101AD438                 cmp     eax, ebx
.text:101AD43A                 jz      short loc\_101AD440
.text:101AD43C                 mov     \[eax\], ebx
.text:101AD43E                 jmp     short loc\_101AD442
.text:101AD440 ; ---------------------------------------------------------------------------
.text:101AD440
.text:101AD440 loc\_101AD440:                           ; CODE XREF: sub\_101AD3AA+90j
.text:101AD440                 xor     eax, eax
.text:101AD442
.text:101AD442 loc\_101AD442:                           ; CODE XREF: sub\_101AD3AA+94j
.text:101AD442                 mov     ecx, \[ebp+arg\_0\]
.text:101AD445                 mov     \[ecx+18h\], eax
.text:101AD448                 mov     \[ebp+var\_4\], 8007000Eh
.text:101AD44F                 cmp     eax, ebx
.text:101AD451                 jz      short loc\_101AD464
.text:101AD453                 push    eax             ; szURL
.text:101AD454                 push    \[ebp+pUnk\]      ; pUnk
.text:101AD457                 push    offset stru\_1009ABFC ; riid
.text:101AD45C                 call    CoMarshalInterThreadInterfaceInStream
.text:101AD461                 mov     \[ebp+var\_4\], eax
.text:101AD464
.text:101AD464 loc\_101AD464:                           ; CODE XREF: sub\_101AD3AA+84j
.text:101AD464                                         ; sub\_101AD3AA+A7j
.text:101AD464                 mov     eax, \[ebp+pUnk\]
.text:101AD467                 mov     ecx, \[eax\]
.text:101AD469                 push    eax
.text:101AD46A                 call    dword ptr \[ecx+8\]
.text:101AD46D
.text:101AD46D loc\_101AD46D:                           ; CODE XREF: sub\_101AD3AA+22j
.text:101AD46D                                         ; sub\_101AD3AA+74j
.text:101AD46D                 call    sub\_1010CD9F
.text:101AD472                 test    al, al
.text:101AD474                 jz      short loc\_101AD4CF
.text:101AD476                 lea     eax, \[ebp+var\_C\]
.text:101AD479                 push    eax
.text:101AD47A                 call    ds:iertutil\_58
.text:101AD480                 test    eax, eax
.text:101AD482                 js      short loc\_101AD4CF
.text:101AD484                 lea     eax, \[ebp+szURL\]
.text:101AD487                 push    eax             ; szURL
.text:101AD488                 push    \[ebp+var\_C\]     ; pUnk
.text:101AD48B                 push    offset stru\_100A7FDC ; riid
.text:101AD490                 call    ds:\_\_imp\_CoMarshalInterThreadInterfaceInStream
.text:101AD496                 mov     \[ebp+var\_4\], eax
.text:101AD499                 cmp     eax, ebx
.text:101AD49B                 jnz     short loc\_101AD4C6
.text:101AD49D                 mov     edi, \[ebp+arg\_0\]
.text:101AD4A0                 mov     eax, \[edi+14h\]
.text:101AD4A3                 mov     esi, \[ebp+szURL\]
.text:101AD4A6                 cmp     eax, ebx
.text:101AD4A8                 jz      short loc\_101AD4B0
.text:101AD4AA                 mov     ecx, \[eax\]
.text:101AD4AC                 push    eax
.text:101AD4AD                 call    dword ptr \[ecx+8\]
.text:101AD4B0
.text:101AD4B0 loc\_101AD4B0:                           ; CODE XREF: sub\_101AD3AA+FEj
.text:101AD4B0                 mov     \[edi+14h\], esi
.text:101AD4B3                 cmp     esi, ebx
.text:101AD4B5                 jz      short loc\_101AD4BD
.text:101AD4B7                 mov     eax, \[esi\]
.text:101AD4B9                 push    esi
.text:101AD4BA                 call    dword ptr \[eax+4\]
.text:101AD4BD
.text:101AD4BD loc\_101AD4BD:                           ; CODE XREF: sub\_101AD3AA+10Bj
.text:101AD4BD                 mov     eax, \[ebp+szURL\]
.text:101AD4C0                 mov     ecx, \[eax\]
.text:101AD4C2                 push    eax
.text:101AD4C3                 call    dword ptr \[ecx+8\]
.text:101AD4C6
.text:101AD4C6 loc\_101AD4C6:                           ; CODE XREF: sub\_101AD3AA+F1j
.text:101AD4C6                 mov     eax, \[ebp+var\_C\]
.text:101AD4C9                 mov     ecx, \[eax\]
.text:101AD4CB                 push    eax
.text:101AD4CC                 call    dword ptr \[ecx+8\]
.text:101AD4CF
.text:101AD4CF loc\_101AD4CF:                           ; CODE XREF: sub\_101AD3AA+CAj
.text:101AD4CF                                         ; sub\_101AD3AA+D8j
.text:101AD4CF                 mov     eax, \[ebp+var\_4\]
.text:101AD4D2                 pop     edi
.text:101AD4D3                 pop     esi
.text:101AD4D4                 pop     ebx
.text:101AD4D5                 leave
.text:101AD4D6                 retn    8
```

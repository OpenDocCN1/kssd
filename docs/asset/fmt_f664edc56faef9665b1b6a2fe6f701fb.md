
SDPI:0047BF2A                         ; 用于加密或解密代码,加解密操作是互换的

SDPI:0047BF2A                         ; 是只简单的 xor,因此可以用来加密也可以

SDPI:0047BF2A                         ; 用来解密

SDPI:0047BF2F                 call    loc_47BF35

SDPI:0047BF34                 nop

SDPI:0047BF35

SDPI:0047BF35 loc_47BF35:                                 ; CODE XREF: FNDDBG_47BF14+1B p

SDPI:0047BF35                 pop     eax

SDPI:0047BF36                 add     eax, 3168h

SDPI:0047BF3B                 call    loc_47BF41

SDPI:0047BF40                 nop

SDPI:0047BF41

SDPI:0047BF41 loc_47BF41:                                 ; CODE XREF: FNDDBG_47BF14+27 p

SDPI:0047BF41                 pop     ecx

SDPI:0047BF42                 add     ecx, 3209h

SDPI:0047BF48                 push    0

SDPI:0047BF4A                 push    ecx

SDPI:0047BF4B                 push    eax

SDPI:0047BF4C                 push    0

SDPI:0047BF4E                 call    loc_47BF54

SDPI:0047BF53                 nop

SDPI:0047BF54

SDPI:0047BF54 loc_47BF54:                                 ; CODE XREF: FNDDBG_47BF14+3A p

SDPI:0047BF54                 pop     eax

SDPI:0047BF55                 add     eax, 11h

SDPI:0047BF5A                 push    eax

SDPI:0047BF5B                 jmp     MessageBoxA         ; MessageBoxA 函数，壳用的宏来处理，

SDPI:0047BF5B                         ; 判断函数的前 5 位是否为 CC

SDPI:0047BF5B                         ; 也就是判断有没有下 int3 断点

SDPI:0047BF5B ; ----------------------------------------------------------------------------

SDPI:0047BF60                 dd 90909090h

SDPI:0047BF64 ; ----------------------------------------------------------------------------

SDPI:0047BF64                 push    7

SDPI:0047BF66                 call    loc_47BF6C

SDPI:0047BF6B                 nop

SDPI:0047BF6C

SDPI:0047BF6C loc_47BF6C:                                 ; CODE XREF: FNDDBG_47BF14+52 p

SDPI:0047BF6C                 pop     eax

SDPI:0047BF6D                 add     eax, 11h

SDPI:0047BF72                 push    eax

SDPI:0047BF73                 jmp     ExitProcess

SDPI:0047BF73 ; ----------------------------------------------------------------------------

SDPI:0047BF78 JUNK_47BF78     db '悙悙',0,10h,'@',0,'鄩?,6

SDPI:0047BF78 FNDDBG_47BF14   endp

SDPI:0047BF78

SDPI:0047BF84 ; ----------------------------------------------------------------------------

SDPI:0047BF84

SDPI:0047BF84 INT3_47BF84:                                   ; CODE XREF: SDPI:0047BE98 p

SDPI:0047BF84                 call    loc_47BF8A

SDPI:0047BF89                 nop

SDPI:0047BF8A

SDPI:0047BF8A loc_47BF8A:                                 ; CODE XREF: SDPI:INT3_47BF84 p

SDPI:0047BF8A                 pop     edi

SDPI:0047BF8B                 add     edi, 0FFFFFF07h

SDPI:0047BF91                 mov     [edi], ebx

SDPI:0047BF93                 mov     [edi+4], edx

SDPI:0047BF96                 pop     eax

SDPI:0047BF97                 call    loc_47BF9D

SDPI:0047BF9C                 nop

SDPI:0047BF9D

SDPI:0047BF9D loc_47BF9D:                                 ; CODE XREF: SDPI:0047BF97 p

SDPI:0047BF9D                 pop     eax

SDPI:0047BF9E                 add     eax, 124h

SDPI:0047BFA3                 push    eax

SDPI:0047BFA4                 xor     eax, eax

SDPI:0047BFA6                 push    dword ptr fs:[eax]

SDPI:0047BFA9                 mov     fs:[eax], esp

SDPI:0047BFAC                 mov     ebp, 300EF1D3h

SDPI:0047BFB1                 add     ebp, 12345678h

SDPI:0047BFB7                 mov     ax, 17h

SDPI:0047BFBB                 sub     ax, 13h

SDPI:0047BFBB ; ----------------------------------------------------------------------------

SDPI:0047BFBF JUNK_47BFBF     db '|',7,'?,7,0,10h,'@',0,'鑤鱮',12h,'s',10h,0,10h,'@',0,'?,3,'?

SDPI:0047BFBF                 db 'r',1Bh,'s',19h,0,10h,'@',0,'鑖淧 3?豻扈',5,0,0,0,0,10h,'@',0

SDPI:0047BFBF                 db '鑈 Xf 潗悙悙'

SDPI:0047BFFA ; ----------------------------------------------------------------------------

SDPI:0047BFFA                 int     3                     ; Trap to Debugger

SDPI:0047BFFB                 nop

SDPI:0047BFFC                 cmp     al, 4                 ; SEH 句柄:0047C0C0

SDPI:0047BFFE                 jz      short Done_47C071

SDPI:0047C000

SDPI:0047C000 ; ************* S U B R O U T I N E ***********

SDPI:0047C000

SDPI:0047C000

SDPI:0047C000 FNDDBG_47C000   proc near                        ; CODE XREF: SDPI:0047C08A j

SDPI:0047C000                         ; SDPI:0047C0A2 j ...

SDPI:0047C000                 nop

SDPI:0047C001                 nop

SDPI:0047C002                 nop

SDPI:0047C003                 nop

SDPI:0047C004                 nop

SDPI:0047C005                 call    loc_47C00B

SDPI:0047C00A                 nop

SDPI:0047C00B

SDPI:0047C00B loc_47C00B:                                 ; CODE XREF: FNDDBG_47C000+5 p

SDPI:0047C00B                 pop     eax

SDPI:0047C00C                 add     eax, 5Eh

SDPI:0047C011                 mov     edx, eax

SDPI:0047C013                 add     edx, 32h

SDPI:0047C016                 call    Crypt_Decrypt_CODE    ; 传入 edx(结束地址)传入 EAX(起始地址)

SDPI:0047C016                         ; 用于加密或解密代码,加解密操作是互换的

SDPI:0047C016                         ; 是只简单的 xor,因此可以用来加密也可以

SDPI:0047C016                         ; 用来解密

SDPI:0047C01B                 call    loc_47C021

SDPI:0047C020                 nop

SDPI:0047C021

SDPI:0047C021 loc_47C021:                                 ; CODE XREF: FNDDBG_47C000+1B p

SDPI:0047C021                 pop     eax

SDPI:0047C022                 add     eax, 307Ch

SDPI:0047C027                 call    loc_47C02D

SDPI:0047C02C                 nop

SDPI:0047C02D

SDPI:0047C02D loc_47C02D:                                 ; CODE XREF: FNDDBG_47C000+27 p

SDPI:0047C02D                 pop     ecx

SDPI:0047C02E                 add     ecx, 311Dh

SDPI:0047C034                 push    0

SDPI:0047C036                 push    ecx

SDPI:0047C037                 push    eax

SDPI:0047C038                 push    0

SDPI:0047C03A                 call    loc_47C040

SDPI:0047C03F                 nop

SDPI:0047C040

SDPI:0047C040 loc_47C040:                                 ; CODE XREF: FNDDBG_47C000+3A p

SDPI:0047C040                 pop     eax

SDPI:0047C041                 add     eax, 11h

SDPI:0047C046                 push    eax

SDPI:0047C047                 jmp     MessageBoxA         ; MessageBoxA 函数，壳用的宏来处理，

SDPI:0047C047                         ; 判断函数的前 5 位是否为 CC

SDPI:0047C047                         ; 也就是判断有没有下 int3 断点

SDPI:0047C047 ; ----------------------------------------------------------------------------

SDPI:0047C04C                 dd 90909090h

SDPI:0047C050 ; ----------------------------------------------------------------------------

SDPI:0047C050                 push    7

SDPI:0047C052                 call    loc_47C058

SDPI:0047C057                 nop

SDPI:0047C058

SDPI:0047C058 loc_47C058:                                 ; CODE XREF: FNDDBG_47C000+52 p

SDPI:0047C058                 pop     eax

SDPI:0047C059                 add     eax, 11h

SDPI:0047C05E                 push    eax

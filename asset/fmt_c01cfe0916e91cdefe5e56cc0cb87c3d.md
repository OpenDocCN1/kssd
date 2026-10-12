
`SDPI:0047CA87` ; 用于加密或解密代码，加解密操作是互换的

`SDPI:0047CA87` ; 是只简单的 xor，因此可以用来加密也可以

`SDPI:0047CA87` ; 用来解密

```
SDPI:0047CA8C                 call    loc_47CA92
SDPI:0047CA91                 nop
SDPI:0047CA92
SDPI:0047CA92 loc_47CA92:                      ; CODE XREF: Over_47ca71+1B p
SDPI:0047CA92                 pop     eax
SDPI:0047CA93                 add     eax, 260Bh
SDPI:0047CA98                 call    loc_47CA9E
SDPI:0047CA9D                 nop
SDPI:0047CA9E
SDPI:0047CA9E loc_47CA9E:                      ; CODE XREF: Over_47ca71+27 p
SDPI:0047CA9E                 pop     ecx
SDPI:0047CA9F                 add     ecx, 26ACh
SDPI:0047CAA5                 push    0
SDPI:0047CAA7                 push    ecx
SDPI:0047CAA8                 push    eax
SDPI:0047CAA9                 push    0
SDPI:0047CAAB                 call    loc_47CAB1
SDPI:0047CAB0                 nop
SDPI:0047CAB1
SDPI:0047CAB1 loc_47CAB1:                      ; CODE XREF: Over_47ca71+3A p
SDPI:0047CAB1                 pop     eax
SDPI:0047CAB2                 add     eax, 11h
SDPI:0047CAB7                 push    eax
SDPI:0047CAB8                 jmp     MessageBoxA                 ; MessageBoxA 函数，壳用的宏来处理，
SDPI:0047CAB8                                             ; 判断函数的前 5 位是否为 CC
SDPI:0047CAB8                                             ; 也就是判断有没有下 int3 断点
SDPI:0047CAB8 ; ----------------------------------------------------------------------------
SDPI:0047CABD                 db 4 dup(90h)
SDPI:0047CAC1 ; ----------------------------------------------------------------------------
SDPI:0047CAC1                 push    7
SDPI:0047CAC3                 call    loc_47CAC9
SDPI:0047CAC8                 nop
SDPI:0047CAC9
SDPI:0047CAC9 loc_47CAC9:                      ; CODE XREF: Over_47ca71+52 p
SDPI:0047CAC9                 pop     eax
SDPI:0047CACA                 add     eax, 11h
SDPI:0047CACF                 push    eax
SDPI:0047CAD0                 jmp     ExitProcess
SDPI:0047CAD0 ; ----------------------------------------------------------------------------
SDPI:0047CAD5                 db 4 dup(90h), 0, 10h, 40h, 0, 0BEh, 56h, 5Ch, 1
SDPI:0047CAD5 Over_47ca71     endp
SDPI:0047CAD5
SDPI:0047CAE1 ; ----------------------------------------------------------------------------
SDPI:0047CAE1                 mov     esp, [esp+8]
SDPI:0047CAE5                 pop     large dword ptr fs:0
SDPI:0047CAEC                 call    loc_47CAF2
SDPI:0047CAF1                 nop
SDPI:0047CAF2
SDPI:0047CAF2 loc_47CAF2:                      ; CODE XREF: SDPI:0047CAEC p
SDPI:0047CAF2                 pop     eax
SDPI:0047CAF3                 add     eax, 11h
SDPI:0047CAF8                 push    eax
SDPI:0047CAF9                 jmp     GetTickCount
SDPI:0047CAF9 ; ----------------------------------------------------------------------------
SDPI:0047CAFE                 db 4 dup(90h)
SDPI:0047CB02 ; ----------------------------------------------------------------------------
SDPI:0047CB02                 call    loc_47CB08
SDPI:0047CB07                 nop
SDPI:0047CB08
SDPI:0047CB08 loc_47CB08:                      ; CODE XREF: SDPI:0047CB02 p
SDPI:0047CB08                 pop     edx
SDPI:0047CB09                 add     edx, 0FFFFFA49h
SDPI:0047CB0F                 mov     ecx, [edx]
SDPI:0047CB11                 cmp     ecx, 0
SDPI:0047CB14                 jz      Over_47ca71
SDPI:0047CB1A                 sub     eax, ecx
SDPI:0047CB1C                 js      Over_47ca71
SDPI:0047CB22                 sub     eax, 7D0h
SDPI:0047CB27                 jns     Over_47ca71
SDPI:0047CB2D                 mov     eax, 0E801276h
SDPI:0047CB32                 mov     [edx], eax
SDPI:0047CB34                 call    loc_47CB3A
SDPI:0047CB39                 nop
SDPI:0047CB3A
SDPI:0047CB3A loc_47CB3A:                      ; CODE XREF: SDPI:0047CB34 p
SDPI:0047CB3A                 pop     edx
SDPI:0047CB3B                 add     edx, 0A2h
SDPI:0047CB41                 call    loc_47CB47
SDPI:0047CB46                 nop
SDPI:0047CB47
SDPI:0047CB47 loc_47CB47:                      ; CODE XREF: SDPI:0047CB41 p
SDPI:0047CB47                 pop     eax
SDPI:0047CB48                 add     eax, 0FFFFD652h
SDPI:0047CB4D                 mov     ecx, 10h                   ; 再次解密代码
SDPI:0047CB52                 call    De_Code                     ; invoke De_Code,ecx(counter),eax(key),edx(end)
SDPI:0047CB52                                             ; 用于解密代码，解密起始地址就是 call 进来
SDPI:0047CB52                                             ; 的下一行代码地址
SDPI:0047CB52 ; ----------------------------------------------------------------------------
SDPI:0047CB57                 db 0Fh dup(90h)
SDPI:0047CB66 ; ----------------------------------------------------------------------------
SDPI:0047CB66                 call    loc_47CB6C
SDPI:0047CB6B                 nop
SDPI:0047CB6C
SDPI:0047CB6C loc_47CB6C:                      ; CODE XREF: SDPI:0047CB66 p
SDPI:0047CB6C                 pop     eax
SDPI:0047CB6D                 add     eax, 11h
SDPI:0047CB72                 push    eax
SDPI:0047CB73                 jmp     Get_Version
SDPI:0047CB73 ; ----------------------------------------------------------------------------
SDPI:0047CB78                 db 4 dup(90h)
SDPI:0047CB7C ; ----------------------------------------------------------------------------
SDPI:0047CB7C                 call    loc_47CB82
SDPI:0047CB81                 nop
SDPI:0047CB82
SDPI:0047CB82 loc_47CB82:                      ; CODE XREF: SDPI:0047CB7C p
SDPI:0047CB82                 pop     edx
SDPI:0047CB83                 add     edx, 0FFFFD483h
SDPI:0047CB89                 cmp     eax, [edx]                 ; 又判断是否修改了 GetVersion 的返回值
SDPI:0047CB8B                 jnz     OVER_47C43A
SDPI:0047CB91                 cmp     eax, 80000000h
SDPI:0047CB96                 jb      NotIsWin9x_47D312          ; 再次判断是否为 Winnt 系统
SDPI:0047CB9C                 mov     ah, 43h                    ; 如果是 Win9x 则产生 int 68 异常
SDPI:0047CB9E                 int     68h                      ;  - APPC/PC
SDPI:0047CBA0                 cmp     ax, 0F386h
SDPI:0047CBA4                 jnz     NODBG_47CC43
SDPI:0047CBAA                 jz      short near ptr aU4Rrrrr+6
SDPI:0047CBAC                 push    ebx
SDPI:0047CBAD                 push    edi
SDPI:0047CBAE                 push    es
SDPI:0047CBAE ; ----------------------------------------------------------------------------
SDPI:0047CBAF a9               db '~',6,'?',4,'9 窿',0Dh
SDPI:0047CBB7 ; ----------------------------------------------------------------------------
SDPI:0047CBB7                 xor     di, di
SDPI:0047CBBA                 db      66h
SDPI:0047CBBA                 mov     es, di
SDPI:0047CBBD                 mov     ax, 1684h
SDPI:0047CBC1                 mov     bx, 202h
SDPI:0047CBC5                 int     2Fh
SDPI:0047CBC7                 mov     ax, es
SDPI:0047CBCA                 add     ax, di
SDPI:0047CBCD                 pop     es
SDPI:0047CBCE                 pop     edi
SDPI:0047CBCF                 pop     ebx
SDPI:0047CBD0                 test    ax, ax
SDPI:0047CBD3                 jz      short NODBG_47CC43
```

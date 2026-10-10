

# PECompact 壳分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-501.htm`](https://www.pediy.com/kssd/pediy09/pediy09-501.htm)

> 以前的精都没了，今天拿 PECompact 开个刀，小分析一下，高手勿笑。\*\_^  
>   
> 废话少说，拿出神器 IDA,OD....  
>   
> OD 载入加壳文件到入口点:  
>   
> 01001000 notepad\_.>  B8 04150101          mov     eax, 01011504  
> 01001005             50                   push    eax                            ; SEH 句柄  
> 01001006             64:FF35 00000000     push    dword ptr fs:\[0\]  
> 0100100D             64:8925 00000000     mov     dword ptr fs:\[0\], esp           ; 添加 SEH 链  
> 01001014             33C0                 xor     eax, eax  
> 01001016             8908                 mov     dword ptr ds:\[eax\], ecx        ; 内存访问异常  
> 01001018             50                   push    eax  
>   
> 程序一开始就添加一个结构化异常处理，在 1011504 地址处下断点(bp 1011504), F9 掉转到  
> 异常处理(1011504)  
>   
>   
> .rsrc:01011504 SEH\_handler:  
> .rsrc:01011504                 mov     eax, 0F10102CEh  
> .rsrc:01011509                 lea     ecx, \[eax+10001259h\]            ; 取 PEC\_start 地址  
> .rsrc:0101150F                 mov     \[ecx+1\], eax                    ; 修改 PEC\_start 地址处的 12345678 为 0F10102CE  
> .rsrc:01011512                 mov     edx, \[esp+4\]  
> .rsrc:01011516                 mov     edx, \[edx+0Ch\]                  ; 取异常发生处地址 1001016  
> .rsrc:01011519                 mov     byte ptr \[edx\], 0E9h            ; 修改异常地址处(1001016)指令为 JMP  
> .rsrc:0101151C                 add     edx, 5  
> .rsrc:0101151F                 sub     ecx, edx  
> .rsrc:01011521                 mov     \[edx-4\], ecx                    ; 计算并修改异常地址到 PEC\_start 处的偏移量  
> .rsrc:01011524                 xor     eax, eax                        ; 指令修改后从异常处继续执行(修改后的指令为 JMP PEC\_start)  
> .rsrc:01011526                 retn  
>   
> .rsrc:01011527 PEC\_start:  
> .rsrc:01011527                 mov     eax, 12345678h  
> .rsrc:0101152C                 pop     large dword ptr fs:0  
> .rsrc:01011533                 add     esp, 4  
>   
>   
> 修改后的 PEC\_start  
> .rsrc:01011527 PEC\_start:  
> .rsrc:01011527                 mov     eax, 0F10102CEh  
> .rsrc:0101152C                 pop     large dword ptr fs:0  
> .rsrc:01011533                 add     esp, 4                          ; 清除 SEH 链  
> .rsrc:01011536                 push    ebp  
> .rsrc:01011537                 push    ebx  
> .rsrc:01011538                 push    ecx  
> .rsrc:01011539                 push    edi  
> .rsrc:0101153A                 push    esi  
> .rsrc:0101153B                 push    edx  
> .rsrc:0101153C                 lea     ebx, \[eax+10001212h\]            ; ebx 指向一个结构(10114E0)  
>   
> 结构如下:  
> .rsrc:010114E0 PEC\_encode\_data        dd 10DB4h                        ; 编码后的 PEC 代码  
> .rsrc:010114E4 size                   dd 0DACh  
> .rsrc:010114E8                        dd 178h  
> .rsrc:010114EC depack\_fun\_rva         dd 11435h                        ; PECompact 的解压缩引擎代码相对虚拟地址  
> .rsrc:010114F0 imp\_VirtualAlloc\_rva   dd 10D08h                        ; 导入表中 VirtualAlloc 的相对虚拟地址  
> .rsrc:010114F4 imp\_VirtualFree\_rva    dd 10D0Ch                        ; 导入表中 VirtualFree 的相对虚拟地址  
> .rsrc:010114F8 orig\_ImageBase\_rva     dd offset \_\_ImageBase            ; 基址  
> .rsrc:010114FC imp\_LoadLibrary\_rva    dd 10D00h                        ; 导入表中 LoadLibrary 的相对虚拟地址  
> .rsrc:01011500 imp\_GetProcAddress\_rva dd 10D04h                        ; 导入表中 GetProcAddress 的相对虚拟地址  
>   
>   
> .rsrc:01011542                 mov     edx, \[ebx+18h\]                  ; 取原程序 ImageBase  
> .rsrc:01011545                 push    edx  
> .rsrc:01011546                 mov     ebp, eax  
> .rsrc:01011548                 push    40h  
> .rsrc:0101154A                 push    1000h  
> .rsrc:0101154F                 push    dword ptr \[ebx+4\]               ;   
> .rsrc:01011552                 push    0  
> .rsrc:01011554                 mov     ecx, \[ebx+10h\]                  ; 取 imp\_VirtualAlloc\_rva  
> .rsrc:01011557                 add     ecx, edx                        ; 计算导入表中 VirtualAlloc 的地址  
> .rsrc:01011559                 mov     eax, \[ecx\]  
> .rsrc:0101155B                 call    eax                             ; 调用 VirtualAlloc 分配内存  
> .rsrc:0101155D                 pop     edx  
> .rsrc:0101155E                 mov     edi, eax  
> .rsrc:01011560                 push    eax  
> .rsrc:01011561                 push    edx  
> .rsrc:01011562                 mov     esi, \[ebx\]                      ; 取 PEC\_encode\_data  
> .rsrc:01011564                 mov     eax, \[ebx+20h\]                  ; 取 imp\_GetProcAddress\_rva  
> .rsrc:01011567                 add     eax, edx                        ; 计算导入表中 GetProcAddress 的地址  
> .rsrc:01011569                 mov     ecx, \[eax\]                      ; Kernel32 导出函数 GetProcAddress 的地址  
> .rsrc:0101156B                 mov     \[ebx+20h\], ecx                  ; 保存到 EBX 指向的结构中 imp\_GetProcAddress\_rva  
> .rsrc:0101156E                 mov     eax, \[ebx+1Ch\]                  ; 取导入表中 LoadLibrary 的相对虚拟地址  
> .rsrc:01011571                 add     eax, edx                        ; 计算导入表中 LoadLibrary 的地址  
> .rsrc:01011573                 mov     ecx, \[eax\]                      ; Kernel32 导出函数 LoadLibrary 的地址  
> .rsrc:01011575                 mov     \[ebx+1Ch\], ecx                  ; 把 LoadLibrary 的地址保存到 imp\_LoadLibrary\_rva  
> .rsrc:01011578                 add     esi, edx  
> .rsrc:0101157A                 mov     ecx, \[ebx+0Ch\]                  ; 取解压缩引擎的相对虚拟地址  
> .rsrc:0101157D                 add     ecx, edx                        ; 计算解压缩引擎地址  
> .rsrc:0101157F                 lea     eax, \[ebx+1Ch\]  
> .rsrc:01011582                 push    eax  
> .rsrc:01011583                 push    edi  
> .rsrc:01011584                 push    esi  
> .rsrc:01011585                 call    ecx                             ; 调用解压缩引擎, 解压缩 PEC\_encode\_data  
> .rsrc:01011587                 pop     edx  
> .rsrc:01011588                 pop     eax  
> .rsrc:01011589                 add     eax, \[ebx+8\]  
> .rsrc:0101158C                 mov     edi, eax  
> .rsrc:0101158E                 push    edx  
> .rsrc:0101158F                 mov     esi, eax  
> .rsrc:01011591                 mov     eax, \[esi-4\]  
> .rsrc:01011594                 add     eax, 4  
> .rsrc:01011597                 sub     esi, eax  
> .rsrc:01011599                 mov     \[esi+8\], edx  
> .rsrc:0101159C                 mov     ecx, \[ebx+0Ch\]  
> .rsrc:0101159F                 mov     \[esi+14h\], ecx  
> .rsrc:010115A2                 call    edi                             ; 关键，调用解压缩后的 PEC 代码 PEC\_unpack  
> .rsrc:010115A4                 mov     \[ebp+100012FAh\], eax  
> .rsrc:010115AA                 mov     esi, eax  
> .rsrc:010115AC                 mov     ecx, \[ebx+14h\]  
> .rsrc:010115AF                 pop     edx  
> .rsrc:010115B0                 jmp     short loc\_10115BE  
> .rsrc:010115B0  
> .rsrc:010115B2 ; ---------------------------------------------------------------------------  
> .rsrc:010115B2                 add     ecx, edx  
> .rsrc:010115B4                 push    8000h  
> .rsrc:010115B9                 push    0  
> .rsrc:010115BB                 push    edi  
> .rsrc:010115BC                 call    dword ptr \[ecx\]  
> .rsrc:010115BC  
> .rsrc:010115BE  
> .rsrc:010115BE loc\_10115BE:  
> .rsrc:010115BE                 mov     eax, esi  
> .rsrc:010115C0                 pop     edx  
> .rsrc:010115C1                 pop     esi  
> .rsrc:010115C2                 pop     edi  
> .rsrc:010115C3                 pop     ecx  
> .rsrc:010115C4                 pop     ebx  
> .rsrc:010115C5                 pop     ebp  
> .rsrc:010115C6                 jmp     eax                             ; 跳转到 OEP  
> .rsrc:010115C6  
> .rsrc:010115C6 ; ---------------------------------------------------------------------------  
> .rsrc:010115C8                 dd 0Eh dup(0)  
> .rsrc:01011600                 dd 280h dup(?)  
> .rsrc:01011600 \_rsrc           ends  
>   
>   
> 解压缩后的 PEC 代码:  
> seg000:00000178 PEC\_unpack      proc near  
> seg000:00000178  
> seg000:00000178 arg\_F           = dword ptr  23h  
> seg000:00000178 arg\_34          = dword ptr  48h  
> seg000:00000178  
> seg000:00000178                 push    ebx  
> seg000:00000179                 push    edi  
> seg000:0000017A                 push    esi  
> seg000:0000017B                 push    ebp  
> seg000:0000017C                 call    $+5  
> seg000:00000181                 pop     ebp  
> seg000:00000182                 sub     ebp, 10001307h  
> seg000:00000188                 lea     esi, \[ebp+100012FEh\]           ; 计算 PEC\_unpack 地址  
> seg000:0000018E                 mov     eax, \[esi-4\]  
> seg000:00000191                 add     eax, 4  
> seg000:00000194                 sub     esi, eax  
> seg000:00000196                 cld  
> seg000:00000197                 mov     ebx, esi  
> seg000:00000199                 mov     edx, \[esi+8\]  
> seg000:0000019C                 mov     esi, \[esi+1Ch\]  
> seg000:0000019F                 add     esi, edx  
> seg000:000001A1                 lea     edi, \[ebp+10001E21h\]  
> seg000:000001A7                 lodsd                                  ; +0C9B 构建 PEC 用导入表   
> seg000:000001A8                 stosd                                  ; imp\_LoadLibrary  
> seg000:000001A9                 lodsd  
> seg000:000001AA                 stosd                                  ; imp\_GetProcAddress  
> seg000:000001AB                 lodsd  
> seg000:000001AC                 stosd                                  ; imp\_VirtualAlloc  
> seg000:000001AD                 lodsd  
> seg000:000001AE                 stosd                                  ; imp\_VirtualFree  
> seg000:000001AF                 nop  
> seg000:000001B0                 cmp     dword ptr \[ebx+48h\], 1  
> seg000:000001B4                 jz      short loc\_1CB  
> seg000:000001B4  
> seg000:000001B6                 mov     esi, \[ebx+44h\]  
> seg000:000001B9                 test    esi, esi  
> seg000:000001BB                 jz      short loc\_1CB  
> seg000:000001BB  
> seg000:000001BD                 mov     ecx, 23h  
> seg000:000001C2                 add     esi, edx  
> seg000:000001C4                 mov     edi, \[ebx+40h\]  
> seg000:000001C7                 add     edi, edx  
> seg000:000001C9                 rep movsb  
> seg000:000001CB loc\_1CB:  
> seg000:000001CB                 mov     esi, ebx  
> seg000:000001CD                 lea     edi, \[ebp+10001E0Dh\]  
> seg000:000001D3                 add     \[edi\], ebp  
> seg000:000001D5                 add     \[edi+4\], ebp  
> seg000:000001D8                 add     \[edi+8\], ebp  
> seg000:000001DB                 lea     ecx, \[ebp+10001D78h\]  
> seg000:000001E1                 push    ecx                            ; +0BF2  
> seg000:000001E2                 call    PEC\_import                     ; 构建 PEC 使用的导入函数表  
>   
>   
> PEC 用导入表构建函数:  
> seg000:00000321 PEC\_import      proc near  
> seg000:00000321  
> seg000:00000321 hModule         = dword ptr -4  
> seg000:00000321 arg\_offset\_BF2  = dword ptr  8  
> seg000:00000321  
> seg000:00000321                 push    ebp  
> seg000:00000322                 mov     ebp, esp  
> seg000:00000324                 add     esp, 0FFFFFFFCh  
> seg000:00000327                 push    ebx  
> seg000:00000328                 push    edi  
> seg000:00000329                 push    esi  
> seg000:0000032A                 call    $+5  
> seg000:0000032F                 pop     ebx                            ; EBX = +32F  
> seg000:00000330                 sub     ebx, 100014B5h  
> seg000:00000336                 mov     esi, \[ebp+arg\_offset\_BF2\]  
> seg000:00000339 loc\_339:    
> seg000:00000339                 mov     ecx, \[esi\]                     ; ECX = 10001D94  
> seg000:0000033B                 add     ecx, ebx                       ; ECX = 10001D94 + +32F -100014B5  
> seg000:0000033B                                                        ;     = +0C0E = offset s\_Kernel32  
> seg000:0000033D                 push    ecx  
> seg000:0000033E                 call    dword ptr \[ebx+10001E21h\]      ; +0C9B: imp\_LoadLibrary  
> seg000:00000344                 mov     \[ebp+hModule\], eax  
> seg000:00000347                 mov     edx, \[esi+4\]  
> seg000:0000034A                 mov     edi, \[esi+8\]  
> seg000:0000034D                 add     edx, ebx  
> seg000:0000034F                 add     edi, ebx  
> seg000:00000351 loc\_351:  
> seg000:00000351                 xor     eax, eax  
> seg000:00000353                 add     eax, \[edx\]  
> seg000:00000355                 jz      short loc\_36D  
> seg000:00000357                 push    edx  
> seg000:00000358                 mov     eax, \[edx\]  
> seg000:0000035A                 add     eax, ebx  
> seg000:0000035C                 push    eax  
> seg000:0000035D                 push    \[ebp+hModule\]  
> seg000:00000360                 call    dword ptr \[ebx+10001E25h\]      ; +0C9F: imp\_GetProcAddress  
> seg000:00000366                 stosd  
> seg000:00000367                 pop     edx  
> seg000:00000368                 add     edx, 4  
> seg000:0000036B                 jmp     short loc\_351  
> seg000:0000036D ; ---------------------------------------------------------------------------  
> seg000:0000036D loc\_36D:  
> seg000:0000036D                 add     esi, 0Ch  
> seg000:00000370                 add     eax, \[esi\]  
> seg000:00000372                 jnz     short loc\_339                  ; ECX = 10001D94  
> seg000:00000374                 pop     esi  
> seg000:00000375                 pop     edi  
> seg000:00000376                 pop     ebx  
> seg000:00000377                 leave  
> seg000:00000378                 retn    4  
> seg000:00000378 PEC\_import      endp  
>   
>   
> 构建后的导入数据如下:  
> seg000:00000BC1 BT\_s\_Messageboxa      dd 10001D62h  
> seg000:00000BC5 BT\_s\_Wsprintfa        dd 10001D6Eh  
> seg000:00000BC9                       dd 0  
> seg000:00000BCD imp\_MessageBoxA       dd 0  
> seg000:00000BD1 imp\_wsprintfA         dd 0  
> seg000:00000BD5 s\_User32              db 'user32',0  
> seg000:00000BDC s\_Messageboxa         db 'MessageBoxA',0  
> seg000:00000BE8 s\_Wsprintfa           db 'wsprintfA',0  
> seg000:00000BF2 BT\_s\_Kernel32         dd 10001D94h                     ; 字符串 s\_Kernel32 的变形偏移地址  
> seg000:00000BF6                       dd 10001DE1h  
> seg000:00000BFA BT\_iat\_kernel32       dd 10001DF9h                     ; 构建从 Kernel32 导入的函数表变形偏移  
> seg000:00000BFE BT\_s\_User32           dd 10001D5Bh                     ; 字符串 s\_User32 的变形偏移地址  
> seg000:00000C02                       dd 10001D47h  
> seg000:00000C06                       dd 10001D53h  
> seg000:00000C0A                       dd 0  
> seg000:00000C0E s\_Kernel32            db 'kernel32',0  
> seg000:00000C17 s\_Exitprocess         db 'ExitProcess',0  
> seg000:00000C23 s\_Closehandle         db 'CloseHandle',0  
> seg000:00000C2F s\_Openprocess         db 'OpenProcess',0  
> seg000:00000C3B s\_Getmodulehandlea    db 'GetModuleHandleA',0  
> seg000:00000C4C s\_Virtualprotect      db 'VirtualProtect',0  
> seg000:00000C5B BT\_s\_Exitprocess      dd 10001D9Dh                     ; 字符串 s\_Exitprocess 的变形偏移  
> seg000:00000C5F BT\_s\_Getmodulehandlea dd 10001DC1h                     ; 字符串 s\_Getmodulehandlea 的变形偏移  
> seg000:00000C63 BT\_s\_Closehandle      dd 10001DA9h                     ; 字符串 s\_Closehanle 的变形偏移  
> seg000:00000C67 BT\_s\_Openprocess      dd 10001DB5h                     ; 字符串 s\_Openprocess 的变形偏移  
> seg000:00000C6B BT\_s\_VirtualProtect   dd 10001DD2h                     ; 字符串 s\_Virtualprotect 的变形偏移  
> seg000:00000C6F                       dd 0  
> seg000:00000C73 imp\_ExitProcess       dd 0                             ; 构建从 Kernel32 导入的函数表  
> seg000:00000C77 imp\_GetModuleHandleA  dd 0  
> seg000:00000C7B imp\_CloseHandle       dd 0  
> seg000:00000C7F imp\_OpenProcess       dd 0  
> seg000:00000C83 imp\_VirtualProtect    dd 0  
> seg000:00000C87 func                  dd 10001C27h  
> seg000:00000C8B func\_0                dd 10001C49h  
> seg000:00000C8F func\_1                dd 10001952h  
> seg000:00000C93                       dd 0  
> seg000:00000C97                       dd 0  
> seg000:00000C9B imp\_LoadLibrary       dd 0  
> seg000:00000C9F imp\_GetProcAddress    dd 0  
> seg000:00000CA3 imp\_VirtualAlloc      dd 0  
> seg000:00000CA7 imp\_VirtualFree       dd 0  
>   
>   
>   
> seg000:000001E1                 push    ecx                            ; +0BF2  
> seg000:000001E2                 call    PEC\_import                     ; 构建 PEC 使用的导入函数  
> seg000:000001E2  
> seg000:000001E7                 nop  
> seg000:000001E8                 nop  
> seg000:000001E9                 nop  
> seg000:000001EA                 nop  
> seg000:000001EB                 nop  
> seg000:000001EC                 nop  
> seg000:000001ED                 nop  
> seg000:000001EE                 nop  
> seg000:000001EF                 mov     ecx, \[esi+2Ch\]                 ; +0E0  
> seg000:000001F2                 mov     \[ebp+10001E1Dh\], ecx           ; +0C97  
> seg000:000001F8                 push    40h  
> seg000:000001FA                 push    1000h  
> seg000:000001FF                 push    ecx  
> seg000:00000200                 push    0  
> seg000:00000202                 call    dword ptr \[ebp+10001E29h\]      ; +0CA3: imp\_VirtualAlloc  
> seg000:00000208                 mov     \[ebp+10001E19h\], eax  
> seg000:0000020E                 push    esi                            ; ESI = +0B4  
> seg000:0000020F                 call    depack\_section                 ; 调用解压缩引擎解压缩节数据  
>   
> 解压缩引擎用 PEC+0 偏移处的一个 DWORD 偏移值指向 PEC 压缩时定义的解压缩引擎  
> seg000:00000000 engine\_offset   dd 8                                   ; 这里是 8, 就是从+8 处是解压缩引擎  
> seg000:00000004                 dd 0  
> seg000:00000008  
> seg000:00000008 ; =============== S U B R O U T I N E =======================================  
> seg000:00000008  
> seg000:00000008 depack\_engine   proc near  
> ....解压缩代码略.  
>   
>   
> seg000:0000020F  
> seg000:00000214                 lea     ecx, \[ebp+10001CC8h\]  
> seg000:0000021A                 test    eax, eax  
> seg000:0000021C                 jnz     loc\_2B6  
> seg000:0000021C  
> seg000:00000222                 push    esi                            ; ESI = +0B4  
> seg000:00000223                 call    rebuild\_section                ; 重新构建节的内存映象  
> seg000:00000223  
> seg000:00000228                 push    esi                            ; ESI = +0B4  
> seg000:00000229                 call    code\_fix                       ; 修复变形的 JMP, CALL 指令  
> seg000:00000229  
> seg000:0000022E                 nop  
> seg000:0000022F                 nop  
> seg000:00000230                 nop  
> seg000:00000231                 nop  
> seg000:00000232                 nop  
> seg000:00000233                 nop  
> seg000:00000234                 nop  
> seg000:00000235                 nop  
> seg000:00000236                 nop  
> seg000:00000237                 nop  
> seg000:00000238                 nop  
> seg000:00000239                 nop  
> seg000:0000023A                 nop  
> seg000:0000023B                 nop  
> seg000:0000023C                 mov     ecx, \[esi+34h\]  
> seg000:0000023F                 test    ecx, ecx  
> seg000:00000241                 jz      loc\_2D0  
> seg000:00000241  
> seg000:00000247                 add     ecx, \[esi+8\]  
> seg000:0000024A                 push    ecx  
> seg000:0000024B                 push    esi  
> seg000:0000024C                 call    rebuild\_IAT                    ; 重新构建源程序导入表  
> seg000:0000024C  
> seg000:00000251                 test    eax, eax  
> seg000:00000253                 jz      short loc\_2D0  
> seg000:00000253  
> seg000:00000255                 mov     edx, \[ebp+10001A57h\]  
> seg000:0000025B                 mov     ecx, \[ebp+10001A5Bh\]  
> seg000:00000261                 test    ecx, ecx  
> seg000:00000263                 jnz     short loc\_26D  
> seg000:00000263  
> seg000:00000265                 lea     ecx, \[ebp+10001D45h\]  
> seg000:0000026B                 jmp     short loc\_29A  
> seg000:0000026B  
> seg000:0000026D ; ---------------------------------------------------------------------------  
> seg000:0000026D  
> seg000:0000026D loc\_26D:   
> seg000:0000026D                 test    ecx, 80000000h  
> seg000:00000273                 jz      short loc\_293  
> seg000:00000273  
> seg000:00000275                 push    edx  
> seg000:00000276                 and     ecx, 7FFFFFFFh  
> seg000:0000027C                 push    ecx  
> seg000:0000027D                 lea     eax, \[ebp+10001D12h\]  
> seg000:00000283                 push    eax  
> seg000:00000284                 lea     eax, \[ebp+10001E31h\]  
> seg000:0000028A                 push    eax  
> seg000:0000028B                 call    dword ptr \[ebp+10001D57h\]  
> seg000:00000291                 jmp     short loc\_2B0  
> seg000:00000291  
> seg000:00000293 ; ---------------------------------------------------------------------------  
> seg000:00000293  
> seg000:00000293 loc\_293:      
> seg000:00000293                 mov     eax, \[esi+8\]  
> seg000:00000296                 add     ecx, eax  
> seg000:00000298                 inc     ecx  
> seg000:00000299                 inc     ecx  
> seg000:00000299  
> seg000:0000029A  
> seg000:0000029A loc\_29A:      
> seg000:0000029A                 push    edx  
> seg000:0000029B                 push    ecx  
> seg000:0000029C                 lea     eax, \[ebp+10001CDDh\]  
> seg000:000002A2                 push    eax  
> seg000:000002A3                 lea     eax, \[ebp+10001E31h\]  
> seg000:000002A9                 push    eax  
> seg000:000002AA                 call    dword ptr \[ebp+10001D57h\]  
> seg000:000002AA  
> seg000:000002B0  
> seg000:000002B0 loc\_2B0:       
> seg000:000002B0                 lea     ecx, \[ebp+10001E31h\]  
> seg000:000002B6  
> seg000:000002B6 loc\_2B6:    
> seg000:000002B6                 push    10h  
> seg000:000002B8                 lea     eax, \[ebp+10001CB6h\]  
> seg000:000002BE                 push    eax  
> seg000:000002BF                 push    ecx  
> seg000:000002C0                 push    0  
> seg000:000002C2                 call    dword ptr \[ebp+10001D53h\]  
> seg000:000002C8                 push    0  
> seg000:000002CA                 call    dword ptr \[ebp+10001DF9h\]  
> seg000:000002CA  
> seg000:000002D0  
> seg000:000002D0 loc\_2D0:    
> seg000:000002D0                 mov     edi, \[ebx+8\]  
> seg000:000002D3                 mov     ebx, esi  
> seg000:000002D5                 cmp     dword ptr \[ebx+48h\], 1  
> seg000:000002D9                 jnz     short loc\_2F0  
> seg000:000002D9  
> seg000:000002DB                 mov     eax, \[ebx+0Ch\]  
> seg000:000002DE                 mov     ecx, \[ebx+40h\]  
> seg000:000002E1                 mov     esi, ecx  
> seg000:000002E3                 add     esi, edi  
> seg000:000002E5                 mov     byte ptr \[esi\], 0E9h  
> seg000:000002E8                 add     ecx, 5  
> seg000:000002EB                 sub     eax, ecx  
> seg000:000002ED                 mov     \[esi+1\], eax  
> seg000:000002ED  
> seg000:000002F0  
> seg000:000002F0 loc\_2F0:    
> seg000:000002F0                 mov     esi, ebx  
> seg000:000002F2                 nop  
> seg000:000002F3                 nop  
> seg000:000002F4                 nop  
> seg000:000002F5                 nop  
> seg000:000002F6                 nop  
> seg000:000002F7                 nop  
> seg000:000002F8                 nop  
> seg000:000002F9                 nop  
> seg000:000002FA                 nop  
> seg000:000002FB                 nop  
> seg000:000002FC                 nop  
> seg000:000002FD                 nop  
> seg000:000002FE                 push    edi  
> seg000:000002FF                 call    sub\_A39                        ; 修改节属性不可写  
> seg000:000002FF  
> seg000:00000304                 push    8000h  
> seg000:00000309                 push    0  
> seg000:0000030B                 push    dword ptr \[ebp+10001E19h\]  
> seg000:00000311                 call    dword ptr \[ebp+10001E2Dh\]      ; +0CA7: imp\_VirtualFree  
> seg000:00000317                 mov     eax, \[esi+0Ch\]  
> seg000:0000031A                 add     eax, edi  
> seg000:0000031C                 pop     ebp  
> seg000:0000031D                 pop     esi  
> seg000:0000031E                 pop     edi  
> seg000:0000031F                 pop     ebx  
> seg000:00000320                 retn  
> seg000:00000320                 PEC\_unpack      endp  
>   
>   
>         Berglob  
>         2007-04-23 

 4dd466d0e1e0cc123afbe504da2462ca
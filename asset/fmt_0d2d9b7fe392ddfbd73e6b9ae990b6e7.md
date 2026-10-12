
```.rsrc:010115C6  
.rsrc:010115C6 ; ---------------------------------------------------------------------------  
.rsrc:010115C8               dd 0Eh dup(0)  
.rsrc:01011600               dd 280h dup(?)  
.rsrc:01011600 _rsrc            ends  
```

解压缩后的 PEC 代码:

```
seg000:00000178 PEC_unpack     proc near  
seg000:00000178  
seg000:00000178 arg_F            = dword ptr  23h  
seg000:00000178 arg_34           = dword ptr  48h  
seg000:00000178  
seg000:00000178                push    ebx  
seg000:00000179                push    edi  
seg000:0000017A                push    esi  
seg000:0000017B                push    ebp  
seg000:0000017C                call    $+5  
seg000:00000181                pop     ebp  
seg000:00000182                sub     ebp, 10001307h  
seg000:00000188                lea     esi, [ebp+100012FEh]            ; 计算 PEC_unpack 地址  
seg000:0000018E                mov     eax, [esi-4]  
seg000:00000191                add     eax, 4  
seg000:00000194                sub     esi, eax  
seg000:00000196                cld  
seg000:00000197                mov     ebx, esi  
seg000:00000199                mov     edx, [esi+8]  
seg000:0000019C                mov     esi, [esi+1Ch]  
seg000:0000019F                add     esi, edx  
seg000:000001A1                lea     edi, [ebp+10001E21h]  
seg000:000001A7                lodsd                              ; +0C9B 构建 PEC 用导入表  
seg000:000001A8                stosd                              ; imp_LoadLibrary  
seg000:000001A9                lodsd  
seg000:000001AA                stosd                              ; imp_GetProcAddress  
seg000:000001AB                lodsd  
seg000:000001AC                stosd                              ; imp_VirtualAlloc  
seg000:000001AD                lodsd  
seg000:000001AE                stosd                              ; imp_VirtualFree  
seg000:000001AF                nop  
seg000:000001B0                cmp     dword ptr [ebx+48h], 1  
seg000:000001B4                jz      short loc_1CB  
seg000:000001B4  
seg000:000001B6                mov     esi, [ebx+44h]  
seg000:000001B9                test    esi, esi  
seg000:000001BB                jz      short loc_1CB  
seg000:000001BB  
seg000:000001BD                mov     ecx, 23h  
seg000:000001C2                add     esi, edx  
seg000:000001C4                mov     edi, [ebx+40h]  
seg000:000001C7                add     edi, edx  
seg000:000001C9                rep movsb  
seg000:000001CB loc_1CB:  
seg000:000001CB                mov     esi, ebx  
seg000:000001CD                lea     edi, [ebp+10001E0Dh]  
seg000:000001D3                add     [edi], ebp  
seg000:000001D5                add     [edi+4], ebp  
seg000:000001D8                add     [edi+8], ebp  
seg000:000001DB                lea     ecx, [ebp+10001D78h]  
seg000:000001E1                push    ecx                        ; +0BF2  
seg000:000001E2                call    PEC_import                 ; 构建 PEC 使用的导入函数表  
```

PEC 用导入表构建函数:

```
seg000:00000321 PEC_import     proc near  
seg000:00000321  
seg000:00000321 hModule         = dword ptr -4  
seg000:00000321 arg_offset_BF2  = dword ptr  8  
seg000:00000321  
seg000:00000321                push    ebp  
seg000:00000322                mov     ebp, esp  
seg000:00000324                add     esp, 0FFFFFFFCh  
seg000:00000327                push    ebx  
seg000:00000328                push    edi  
seg000:00000329                push    esi  
seg000:0000032A                call    $+5  
seg000:0000032F                pop     ebx                        ; EBX = +32F  
seg000:00000330                sub     ebx, 100014B5h  
seg000:00000336                mov     esi, [ebp+arg_offset_BF2]  
seg000:00000339 loc_339:   
seg000:00000339                mov     ecx, [esi]                 ; ECX = 10001D94  
seg000:0000033B                add     ecx, ebx                   ; ECX = 10001D94 + +32F -100014B5  
seg000:0000033B                                                       ;     = +0C0E = offset s_Kernel32  
seg000:0000033D                push    ecx  
seg000:0000033E                call    dword ptr [ebx+10001E21h]  ; +0C9B: imp_LoadLibrary  
seg000:00000344                mov     [ebp+hModule], eax  
seg000:00000347                mov     edx, [esi+4]  
seg000:0000034A                mov     edi, [esi+8]  
seg000:0000034D                add     edx, ebx  
seg000:0000034F                add     edi, ebx  
seg000:00000351 loc_351:  
seg000:00000351                xor     eax, eax  
seg000:00000353                add     eax, [edx]  
seg000:00000355                jz      short loc_36D  
seg000:00000357                push    edx  
seg000:00000358                mov     eax, [edx]  
seg000:0000035A                add     eax, ebx  
seg000:0000035C                push    eax  
seg000:0000035D                push    [ebp+hModule]  
seg000:00000360                call    dword ptr [ebx+10001E25h]  ; +0C9F: imp_GetProcAddress  
seg000:00000366                stosd  
seg000:00000367                pop     edx  
seg000:00000368                add     edx, 4  
seg000:0000036B                jmp     short loc_351  
seg000:0000036D ; ---------------------------------------------------------------------------  
seg000:0000036D loc_36D:  
seg000:0000036D                add     esi, 0Ch  
seg000:00000370                add     eax, [esi]  
seg000:00000372                jnz     short loc_339              ; ECX = 10001D94  
seg000:00000374                pop     esi  
seg000:00000375                pop     edi  
seg000:00000376                pop     ebx  
seg000:00000377                leave  
seg000:00000378                retn    4  
seg000:00000378 PEC_import     endp  
```

构建后的导入数据如下:

```
seg000:00000BC1 BT_s_Messageboxa     dd 10001D62h  
seg000:00000BC5 BT_s_Wsprintfa       dd 10001D6Eh  
seg000:00000BC9                    dd 0  
seg000:00000BCD imp_MessageBoxA      dd 0  
seg000:00000BD1 imp_wsprintfA        dd 0  
seg000:00000BD5 s_User32             db 'user32',0  
seg000:00000BDC s_Messageboxa        db 'MessageBoxA',0  
seg000:00000BE8 s_Wsprintfa          db 'wsprintfA',0  
seg000:00000BF2 BT_s_Kernel32        dd 10001D94h               ; 字符串 s_Kernel32 的变形偏移地址  
seg000:00000BF6                    dd 10001DE1h  
seg000:00000BFA BT_iat_kernel32      dd 10001DF9h               ; 构建从 Kernel32 导入的函数表变形偏移  
seg000:00000BFE BT_s_User32          dd 10001D5Bh               ; 字符串 s_User32 的变形偏移地址  
seg000:00000C02                    dd 10001D47h  
seg000:00000C06                    dd 10001D53h  
seg000:00000C0A                    dd 0  
seg000:00000C0E s_Kernel32          db 'kernel32',0  
seg000:00000C17 s_Exitprocess       db 'ExitProcess',0  
seg000:00000C23 s_Closehandle       db 'CloseHandle',0  
seg000:00000C2F s_Openprocess       db 'OpenProcess',0  
seg000:00000C3B s_Getmodulehandlea  db 'GetModuleHandleA',0  
seg000:00000C4C s_Virtualprotect    db 'VirtualProtect',0  
seg000:00000C5B BT_s_Exitprocess    dd 10001D9Dh               ; 字符串 s_Exitprocess 的变形偏移  
seg000:00000C5F BT_s_Getmodulehandlea dd 10001DC1h              ; 字符串 s_Getmodulehandlea 的变形偏移  
```


# 学习PE输出表

```asm
; searchapi.asm

.386

.model flat, stdcall

option casemap:none

include         windows.inc

include         kernel32.inc

includelib      kernel32.lib

.data

szOk                    db ":)  Ok !",0

szCap                   db "[sapi] -icytear- ",0

szUser32Dll             db "user32.dll",0

szLoadLibraryA          db "LoadLibraryA",0

szMsgBox                db "MessageBoxA",0

szGetProcAddress        db "GetProcAddress",0

Kernel32Base    dd 0

User32Base      dd 0

fGetProcAddress dd 0

fLoadLibraryA   dd 0

fMessageBoxA    dd 0

fExitProcess    dd 0

.code

start:

        mov    ecx, [esp]

        xor    edx, edx

        assume  fs:nothing

        push   offset \_exception\_handle

        push   fs:[0]

        mov    fs:[0],esp

        ;----------------------------

        ; get kernel32 module handle

        ;----------------------------

getK32Base:

        dec    ecx

        cmp    word ptr [ecx],IMAGE\_DOS\_SIGNATURE       ;0x5A4D

        jnz    getK32Base

        mov    dx,word ptr [ecx+IMAGE\_DOS\_HEADER.e\_lfanew]

        cmp    dword ptr [ecx+edx],IMAGE\_NT\_SIGNATURE   ;0x00004550

        jnz    getK32Base

        cmp    ecx,dword ptr [ecx+edx+IMAGE\_NT\_HEADERS.OptionalHeader.ImageBase]

        jnz    getK32Base

        mov    Kernel32Base,ecx

        ;-----------------------

        ; search GetProcAddress

        ;-----------------------

        mov    esi,Kernel32Base

        mov    edx,[esi+IMAGE\_DOS\_HEADER.e\_lfanew]

        add    esi,edx

        add    esi,78h ;IMAGE\_DATA\_DIRECTORY [IMAGE\_DIRECTORY\_ENTRY\_EXPORT]

        assume  esi: ptr IMAGE\_DATA\_DIRECTORY

        mov    edi,[esi].VirtualAddress

        add    edi,Kernel32Base

        assume edi: ptr IMAGE\_EXPORT\_DIRECTORY

        mov    edx,[edi].AddressOfNames

        add    edx,Kernel32Base

        mov    ebx,[edi].NumberOfNames

        push    ebx

        push    edx

        push    offset szGetProcAddress

        push    Kernel32Base

        call    SearchFunName

        mov    ebx,[edi].AddressOfNameOrdinals

        add    ebx,Kernel32Base

        lea    ebx,[ebx+eax\*2]

        movzx    ebx,word ptr [ebx]

        mov    esi,[edi].AddressOfFunctions

        add    esi,Kernel32Base

        lea    esi,[esi+ebx\*4]

        mov    ebx,dword ptr [esi]

        add    ebx,Kernel32Base

        mov    fGetProcAddress,ebx ;save address of GetProcAddress

        ;---------------------

        ;search LoadLibraryA

        ;---------------------

        mov    ebx,[edi].NumberOfNames

        push   ebx

        push   edx

        push   offset szLoadLibraryA

        push    Kernel32Base

        call    SearchFunName

        mov    ebx,[edi].AddressOfNameOrdinals

        add    ebx,Kernel32Base

        lea    ebx,[ebx+eax\*2]

        movzx  ebx,word ptr [ebx]

        mov    esi,[edi].AddressOfFunctions

        add    esi,Kernel32Base

        lea    esi,[esi+ebx\*4]

        mov    ebx,dword ptr [esi]

        add    ebx,Kernel32Base

        mov    fLoadLibraryA,ebx

        push    offset szUser32Dll

        call    fLoadLibraryA

        mov     User32Base,eax

        push    offset szMsgBox

        push    User32Base

        call    fGetProcAddress

        push    MB\_OK

        push    offset szCap

        push    offset szOk

        push    NULL

        call    eax

        invoke   ExitProcess, NULL

SearchFunName   proc uses ebx ecx esi edi edx \
        base:DWORD, pFunName:DWORD, pArray:DWORD, num:DWORD

        mov    ecx,-1

        mov    edi,pFunName

        xor    eax,eax

        repnz  scasb

        not    ecx

        dec    ecx

        mov    edx,ecx

        xor    ebx,ebx

        .while(ebx<num)

                mov    ecx,edx

                mov    esi,pFunName

                mov    edi,pArray

                lea    edi,[edi+ebx\*4]

                mov    edi,dword ptr [edi]

                add    edi,base

                repz   cmpsb

                jz     @F

                inc    ebx

        .endw

        mov    eax,-1 ;not found

        jmp    @exit

        @@:

        mov    eax,ebx ;return index of AddressOfNameOrdinals.

        @exit:

        ret

SearchFunName   endp

\_exception\_handle   proc   uses ebx esi \
        pExcept:DWORD, pFrame:DWORD, pContext:DWORD, pDispatch:DWORD

        mov    esi,pContext

        assume esi:ptr CONTEXT

        lea    ebx,getK32Base

        mov    [esi].regEip,ebx ;change eip, execute continue

        mov    eax,0

        ret

\_exception\_handle  endp

end start
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

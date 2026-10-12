
# API 暴力获取

> 原文：[`www.pediy.com/kssd/pediy06/pediy6228.htm`](https://www.pediy.com/kssd/pediy06/pediy6228.htm)

这篇是老调重弹了。看老罗之作，见原程序（`asm.chm` 中）未完，用了很长时间补完，想想长期蒙受了各位大虾的赐教，特感激！！

```
.386

.model flat, stdcall

option casemap:none

include masm32includewindows.inc

GetKernelBase   proto   :DWORD

GetApiAddress   proto   :DWORD, :DWORD

.data

szMyMsg             db  "--=   暴力搜索内存空间获得 Api 的线性地址  =--", 13, 10, 13, 10,
                    "请注意：", 13, 10,
                    "\* 本对话框的线性地址是通过暴力搜索得来 \*", 13, 10, 13, 10,
                    "老罗的缤纷天地",13, 10, "http://www.luocong.com/", 0

szMyCaption         db  "老罗的病毒基础教程系列 by LC", 0

aKernel32Base       dd  0

szUser32            db  "user32.dll", 0

szExitProcess       db  "ExitProcess", 0

aExitProcess        dd  0

szLoadLibraryA      db  "LoadLibraryA", 0

aLoadLibraryA       dd  0

szGetProcAddress    db  "GetProcAddress", 0

aGetProcAddress     dd  0

szMessageBoxA       db  "MessageBoxA", 0

aMessageBoxA        dd  0

.code

main:

call delta

delta:

    pop ebp

    sub ebp, offset delta

    ;获得 Kernel32.dll 的基地址：

    invoke GetKernelBase, [esp]

    mov aKernel32Base, eax

    ;获得 Kernel32.dll 中的所需的 Api 的线性地址：

    invoke GetApiAddress, aKernel32Base, addr szExitProcess

    mov aExitProcess, eax

    invoke GetApiAddress, aKernel32Base, addr szLoadLibraryA

    mov aLoadLibraryA, eax

    invoke GetApiAddress, aKernel32Base, addr szGetProcAddress

    mov aGetProcAddress, eax

    ;载入 User32.dll ：

    push offset szUser32

    call [ebp + aLoadLibraryA]

    ;获得 User32.dll 中的 MessageBoxA 的线性地址：

    push offset szMessageBoxA

    push eax

    call [ebp + aGetProcAddress]

    mov aMessageBoxA, eax

    ;呵呵，千呼万唤始出来，高兴了吧？？

    push MB_OK or MB_ICONINFORMATION

    push offset szMyCaption

    push offset szMyMsg

    push NULL

    call [ebp + aMessageBoxA]

    ;退出：

    push 0

    call [ebp + aExitProcess]

;****************************************************************

;函数功能：查找 Kernel32.dll 的基地址

;****************************************************************

GetKernelBase   proc uses esi edi dwKernelRet:DWORD

    LOCAL dwReturn: DWORD

    mov edi, dwKernelRet               ; edi = 堆栈顶

    and edi, 0ffff0000h               ; 用 AND 获得初始页

    .while TRUE

        .if word ptr [edi] == IMAGE_DOS_SIGNATURE       ; 等于“MZ”吗？

            mov esi, edi                      ; Yes, next...

            add esi, [esi + IMAGE_DOS_HEADER.e_lfanew]  ; 就是 esi + 3ch

            .if word ptr [esi] == IMAGE_NT_SIGNATURE    ; 等于“PE”吗？

                mov dwReturn, edi                 ; Yes, we got it.

                .break

            .endif

        .endif

        ;以下等同于 sub edi, 010000h，即每次减少 64k：

        dec edi

        xor di, di

        .break .if edi < 070000000h    ; 基地址一般不可能小于 70000000h

    .endw

    mov eax, dwReturn

    ret

GetKernelBase   endp

;****************************************************************

;函数功能：从内存中 Kernel32.dll 的导出表中获取某个 API 的入口地址

;****************************************************************

GetApiAddress   proc uses ecx ebx edx esi edi hModule:DWORD, szApiName:DWORD

    LOCAL dwReturn: DWORD

    LOCAL dwApiLength: DWORD

    mov dwReturn, 0

    ;计算 API 字符串的长度（带尾部的 0）

    mov esi, szApiName

    mov edx, esi

Continue_Searching_Null:

    cmp byte ptr [esi], 0            ; 是否为 Null-terminated char ？

    jz We_Got_The_Length             ; Yeah, we got it.  :)

    inc esi                    ; No, continue searching.

    jmp Continue_Searching_Null     ; searching.......

We_Got_The_Length:

    inc esi                    ; 呵呵, 别忘了还有最后一个“0”的长度。

    sub esi, edx               ; esi = API Name size

    mov dwApiLength, esi           ; dwApiLength = API Name size

    ;从 PE 文件头的数据目录获取输出表的地址

    mov esi, hModule

    add esi, [esi + 3ch]

    assume esi: ptr IMAGE_NT_HEADERS

    mov esi, [esi].OptionalHeader.DataDirectory.VirtualAddress

    add esi, hModule

    assume esi:ptr IMAGE_EXPORT_DIRECTORY

    mov edi,szApiName

    mov ecx,[esi].AddressOfNameOrdinals

    add ecx,hModule;取得 AddressOfNameOrdinals 数组的指针

    ;invoke RVAToFileMap,hModule,[esi].AddressOfNames

    mov ebx,[esi].AddressOfNames

    add ebx,hModule;取得 AddressOfNames 数组的指针，特要注意 AddressOfNames 为指向指针的指针数组（个人之见），且都为 RVA

Continue_look:

    mov eax,[ebx]

    add eax,hModule

    push ebx

    mov ebx,eax

    push edi

    push ecx

    mov ecx,dwApiLength;字符串的长度

Continue:

    mov al,[edi]

    mov ah,[ebx]

    xor al,ah;字符串的比较

    jnz again

    dec ecx

    inc edi

    inc ebx

    jcxz found

    jmp  Continue

again:

    pop ecx

    pop edi

    pop ebx

    add ebx,4 ;ebx 指向 AddressOfNames 名的数组，值为双字型，所以加 4

    add ecx,2;ecx 指向序数数组，值是字类型,所以加 2

    jmp Continue_look

found:

    pop ecx

    pop edi

    pop ebx

    mov ebx,ecx

    mov dx,[ebx] ;ebx 指向序数数组，值是字类型

    movzx edx,dx;因此将其转换成双字

    mov ebx,[esi].AddressOfFunctions

    add ebx,hModule;取得 AddressOfFunctions 的指针

    shl edx, 2;索引乘以 4 (AddressOfFunctions 数组中每个元素都是 4 字节大小)

    add edx,ebx;然后加上数组首地址

    mov eax,[edx];eax 就是所要函数的 RVA 了

    add eax,hModule

    ret

GetApiAddress       endp

end main
```

4dd466d0e1e0cc123afbe504da2462ca

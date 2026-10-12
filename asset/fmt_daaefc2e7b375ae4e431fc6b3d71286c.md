
```assembly
CryptResource proc pFileName : LPSTR
    LOCAL hFile : HANDLE
    LOCAL hMap : HANDLE
    LOCAL pMem : LPVOID
    LOCAL dwAppendSize : DWORD     
    LOCAL dwNTHeaderAddr : DWORD
    LOCAL dwImageBase : DWORD
    LOCAL dwOrigEntryAddr : DWORD
    LOCAL dwResSecVA : DWORD
    
    ;; init data
    xor eax, eax
    mov g_bError, al
    mov eax, offset EndDecryptResource - offset DecryptResource
    mov g_dwDecryptResourceSize, eax
    
    ;; get the append size
    invoke CountAppendSize, pFileName
    .IF eax == 0
        jmp GetFileSizeFailed
    .ENDIF
    mov dwAppendSize, eax
    
    ;; open file
    invoke CreateFile, pFileName,\
                    GENERIC_WRITE + GENERIC_READ,\
                    FILE_SHARE_WRITE + FILE_SHARE_READ,\
                    NULL,\
                    OPEN_EXISTING,\
                    FILE_ATTRIBUTE_NORMAL,\
                    0
    .IF eax == INVALID_HANDLE_VALUE
        jmp OpenFileFailed
    .ENDIF
    mov hFile, eax
;    ;; get file size
;    invoke GetFileSize, eax, NULL
;    .IF eax == 0
;        jmp GetFileSizeFailed         
;    .ENDIF
    mov eax, dwAppendSize
    xchg eax, ecx
    ;; create memory map
    xor ebx, ebx
    invoke CreateFileMapping, hFile, ebx, PAGE_READWRITE, ebx, ecx, ebx
    .IF eax == 0
        invoke CloseHandle, hFile
        jmp CreateMapFailed               
    .ENDIF
    mov hMap, eax
    ;; map file to memory
    xor ebx, ebx
    invoke MapViewOfFile, hMap, 
                    FILE_MAP_WRITE + FILE_MAP_READ + FILE_MAP_COPY,
                    ebx, ebx, ebx
    .IF eax == 0
        invoke CloseHandle, hMap
        invoke CloseHandle, hFile
        jmp MapFileFailed         
    .ENDIF                 
    mov pMem, eax        
    mov g_pMem, eax                    
    ;; check it's PE file or not ?
    xchg eax, esi
    assume esi : ptr IMAGE_DOS_HEADER
    .IF [esi].e_magic != 'ZM'
        invoke UnmapViewOfFile, pMem
        invoke CloseHandle, hMap
        invoke CloseHandle, hFile
        jmp InvalidPE         
    .ENDIF         
    add esi, [esi].e_lfanew
    assume esi : ptr IMAGE_NT_HEADERS    
    .IF word ptr [esi].Signature != 'EP'
        invoke UnmapViewOfFile, pMem
        invoke CloseHandle, hMap
        invoke CloseHandle, hFile
        jmp InvalidPE         
    .ENDIF
    
    ;; store basic PE orig information
    mov dwNTHeaderAddr, esi
    assume esi : ptr IMAGE_NT_HEADERS
    mov eax, dword ptr [esi].OptionalHeader.ImageBase
    mov dwImageBase, eax
    mov eax, dword ptr [esi].OptionalHeader.AddressOfEntryPoint
    add eax, dwImageBase
    mov dwOrigEntryAddr, eax
    
    ;; 判断是否存在资源节
    mov eax, dword ptr [esi].OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_RESOURCE * sizeof IMAGE_DATA_DIRECTORY].VirtualAddress
    .IF eax == 0
        jmp NoResourceSec         
    .ENDIF
    mov ebx, eax
    add ebx, dwImageBase
    mov dwResSecVA, ebx     ; ebx 为资源节在内存中的地址
    ;; 转变为 FVA
    invoke RVA2Offset, pMem, eax
    add eax, pMem
    mov g_pResSection, eax
    
    ;; 加密资源节
    invoke EncryptResSec, eax, 0
           
    ;; 升级数据
    lea eax, DecryptRes_dwOrigEntryAddress
    push dwOrigEntryAddr
    pop dword ptr [eax]
    
    lea eax, DecryptRes_ResVA
    push dwResSecVA
    pop dword ptr [eax]
    
    lea eax, DecryptRes_ImageBase
    push dwImageBase
    pop dword ptr [eax]
    
    ;; 添加解密节
    invoke AddSection, pMem, NULL, g_dwDecryptResourceSize
    mov esi, eax
    assume esi : ptr IMAGE_SECTION_HEADER
    
    ;; 更新入口点与 SizeOfImage
    mov edi, dwNTHeaderAddr
    assume edi : ptr IMAGE_NT_HEADERS
    
    ;; 更改资源节属性
    mov eax, dword ptr [edi].OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_RESOURCE * sizeof IMAGE_DATA_DIRECTORY].VirtualAddress
    invoke FindSectionTable, pMem, eax
    assume eax : ptr IMAGE_SECTION_HEADER
    push 0E00000E0h ; 可读可写可执行
    pop dword ptr [eax].Characteristics   
    
    push dword ptr [esi].VirtualAddress
    pop dword ptr [edi].OptionalHeader.AddressOfEntryPoint
    
    mov eax, dword ptr [esi].VirtualAddress
    add eax, dword ptr [esi].Misc.VirtualSize
    mov ebx, dword ptr [edi].OptionalHeader.SectionAlignment
    invoke PEAlign, eax, ebx
    
    push eax
    pop dword ptr [edi].OptionalHeader.SizeOfImage
    
    ;; 将解密节写入
    mov edi, dword ptr [esi].PointerToRawData
    add edi, pMem
    lea esi, DecryptResource
    mov ecx, g_dwDecryptResourceSize
    cld
    rep movsb
                 
LogicShellExit:
    ;; close handle & write it
    invoke UnmapViewOfFile, pMem
    invoke CloseHandle, hMap
    invoke CloseHandle, hFile
    .IF g_bError == 0
        ;; show success message
        invoke crt_printf, offset g_szOutFormat, offset g_szDone
    .ENDIF
    assume eax : nothing
    assume esi : nothing
    assume edi : nothing
    ret
    
;; ----- Show error message ----- 
OpenFileFailed:
    lea eax, g_szOpenFileFailed
    jmp ShowErr
GetFileSizeFailed:
    lea eax, g_szGetFileSizeFailed
    jmp ShowErr     
CreateMapFailed:
    lea eax, g_szCreateMapFailed
    jmp ShowErr
MapFileFailed:
    lea eax, g_szMapFileFailed
    jmp ShowErr         
InvalidPE:
    lea eax, g_szInvalidPE
    jmp ShowErr    
NoResourceSec:
    lea eax, g_szNoResTbl
    jmp ShowErr     
ShowErr:
    invoke crt_printf, offset g_szOutFormat, eax
    mov al, 1
    mov g_bError, al
    jmp LogicShellExit
    
;; ----- 解密资源节 -----
DecryptResource:
DecryptResourceStackSize    equ 10h
DecryptResource_Eip         equ -04h
DecruptResource_dwFlags     equ -08h
    push ebp
    mov ebp, esp
    sub esp, DecryptResourceStackSize
    ;; 清 0 堆栈
    mov edi, esp
    mov ecx, DecryptResourceStackSize
    cld
    xor eax, eax
    rep stosb
    call GetEip
    GetEip:
    pop eax
    sub eax, offset GetEip - offset DecryptResource
    mov dword ptr [ebp+DecryptResource_Eip], eax     
    add eax, offset DecryptRes_ResVA - offset DecryptResource
    push 0               ; Level
    push dword ptr [eax]    ; ResVA
    push dword ptr [eax]    ; pEntry
    call RecDecryptRes
                 
            
    ;; 读取原入口点
    mov eax, offset DecryptRes_dwOrigEntryAddress - offset DecryptResource
    add eax, dword ptr [ebp+DecryptResource_Eip]
    mov eax, dword ptr [eax]
    mov esp, ebp
    pop ebp
    jmp eax
    ;; ----- 解密资源节数据 -----
    DecryptRes_dwOrigEntryAddress           dd  0
    DecryptRes_ResVA                       dd  0
    DecryptRes_bDecrypt                    dd  0
    DecryptRes_ImageBase                   dd  0
    ;; ----- 递归解密资源节的函数 -----
    
    RecDecryptRes:
    RecDecryptResArg_Entry          equ 08h
    RecDecryptResArg_ResVA          equ 0ch
    ResDecryptRes_dwLevel           equ 10h
    
    ResDecryptResStackSize          equ 04h
    ResDecryptRes_Eip               equ -04h
    
        push ebp
        mov ebp, esp
        sub esp, ResDecryptResStackSize
        
        push ebx
        push ecx
        push edx
        push edi
        push esi
        
        ;; 获取 eip
        call GetRecDecryptResEip
        GetRecDecryptResEip:
        pop ebx
        sub ebx, offset GetRecDecryptResEip - offset RecDecryptRes
        mov dword ptr [ebp+ResDecryptRes_Eip], ebx
                 
        mov esi, dword ptr [ebp+RecDecryptResArg_Entry]
        assume esi : ptr IMAGE_RESOURCE_DIRECTORY
        ;; 计算有几个资源在当前目录下
        mov cx, word ptr [esi].NumberOfNamedEntries
        add cx, word ptr [esi].NumberOfIdEntries
        movzx ecx, cx
        ;; 这里存在着 ecx 个 IMAGE_RESOURCE_DIRECTORY_ENTRY
        add esi, sizeof IMAGE_RESOURCE_DIRECTORY
        mov edi, esi
        assume edi : ptr IMAGE_RESOURCE_DIRECTORY_ENTRY
    ResDeDirEntryLoop:
        ;; 解密资源层
        ;; 是 0 级别则清除
        cmp dword ptr [ebp+ResDecryptRes_dwLevel], 0
        jnz SkipClearDecryptFlag
        push 0
        mov eax, dword ptr [ebp+ResDecryptRes_Eip]
        sub eax, offset RecDecryptRes - offset DecryptRes_bDecrypt
        pop dword ptr [eax]
        SkipClearDecryptFlag:
        push dword ptr [ebp+ResDecryptRes_dwLevel]
        push edi
        call JudgeDecryptResType
        mov eax, dword ptr [edi].OffsetToData
        test eax, FIRSTBIT
        jz FoundTheDeDataEntry
        and eax, 0000FFFFh
        add eax, dword ptr [ebp+RecDecryptResArg_ResVA]
        mov edx, dword ptr [ebp+ResDecryptRes_dwLevel]
        inc edx
        push edx
        push dword ptr [ebp+RecDecryptResArg_ResVA]
        push eax
        call RecDecryptRes
        jmp ContinueResDeDirEntryLoop
        FoundTheDeDataEntry:
        ;; 取出资源文件
        mov ebx, dword ptr [ebp+ResDecryptRes_Eip]
        sub ebx, offset RecDecryptRes - offset DecryptRes_bDecrypt
        mov ebx, dword ptr [ebx]
        test ebx, ebx
        jz ContinueResDeDirEntryLoop
        
        ;; ebx 不为 0 则解密
        add eax, dword ptr [ebp+RecDecryptResArg_ResVA]
        assume eax : ptr IMAGE_RESOURCE_DATA_ENTRY
        mov edx, dword ptr [eax].Size1
        mov eax, dword ptr [eax].OffsetToData
        mov ebx, dword ptr [ebp+ResDecryptRes_Eip]
        sub ebx, offset RecDecryptRes - offset DecryptRes_ImageBase
        mov ebx, dword ptr [ebx]
        add eax, ebx
        
        ;; 解密资源
        push edi
        push esi
        push ecx
        mov edi, eax
        mov esi, edi
        mov ecx, edx
        cld
        DeResourceLoop:
        lodsb
        xor al, CRYPT_KEY
        stosb
        loop DeResourceLoop
        
        mov esi, dword ptr [ebp+ResDecryptRes_Eip]
        sub esi, offset RecDecryptRes - offset DecryptRes_bDecrypt
        push 0
        pop dword ptr [esi]
        
        pop ecx
        pop esi
        pop edi
        ContinueResDeDirEntryLoop:
        add edi, sizeof IMAGE_RESOURCE_DIRECTORY_ENTRY       
        dec ecx
        test ecx, ecx
        jnz ResDeDirEntryLoop         
         
        assume esi : nothing
        assume edi : nothing         
        
        pop esi
        pop edi
        pop edx
        pop ecx
        pop ebx         
        
        mov esp, ebp
        pop ebp
        retn 0ch
            
    ;; ----- 判断资源类型 -----
    JudgeDecryptResType:
    JudgeDecryptResArg_Entry         equ 08h
    JudgeDecryptResArg_dwLevel       equ 0ch
    
        push ebp
        mov ebp, esp 
       
        push eax
        push edi
        push ebx
        push esi
        
        ;; 获取 EIP
        call GetJudgeDecryptEip
        GetJudgeDecryptEip:
        pop esi
        sub esi, offset GetJudgeDecryptEip - offset JudgeDecryptResType
        ;; esi 为函数头地址     
       
        ;; 这里不加密第一个 ICON 与第一个 ICON GROUP
        ;; 在 PE 加载器加载时要读取这些资源
        ;; 否则程序会崩溃
        
        ;; 清除加密标志
        ;xor eax, eax
        mov ebx, esi
        sub ebx, offset JudgeDecryptResType - offset DecryptRes_bDecrypt
        ;mov dword ptr [ebx], eax
        
         
        ;; ebx 为解密标志
        ;; edx 为解密标志的地址
        ;; 用两个寄存器来代替变量
        
        mov edi, dword ptr [ebp+JudgeDecryptResArg_Entry]
        assume edi : ptr IMAGE_RESOURCE_DIRECTORY_ENTRY
        mov eax, dword ptr [ebp+JudgeDecryptResArg_dwLevel]
        cmp eax, 0
        jnz ExitJudgeLevel
        ;; 显示类型
        mov eax, dword ptr [edi].Name1
        cmp eax, RT_ICON
        jnz CmpICON_GROUP     
        jmp ExitJudgeLevel
    CmpICON_GROUP:
        cmp eax, RT_GROUP_ICON
        jnz CmpVERSION       
        jmp ExitJudgeLevel
    CmpVERSION:
        cmp eax, RT_VERSION
        jnz DecryptData
        
    ExitJudgeLevel:
        assume eax : nothing
        assume edi : nothing     
 
        
        pop esi
        pop ebx
        pop edi
        pop eax
             
        mov esp, ebp
        pop ebp
        retn 08h     
        
        DecryptData:         
        push 1
        pop dword ptr [ebx]
        jmp ExitJudgeLevel            
EndDecryptResource:
    
CryptResource endp
```

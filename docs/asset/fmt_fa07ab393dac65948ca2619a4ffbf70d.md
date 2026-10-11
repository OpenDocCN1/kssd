
```
ModifyTLS proc szFileName : LPSTR
    LOCAL hFile : HANDLE
    LOCAL hMap : HANDLE
    LOCAL pMem : LPVOID
    LOCAL dwOrigFileSize : DWORD
    LOCAL dwNTHeaderAddr : DWORD
    LOCAL dwNewTlsSectionOffset : DWORD
    LOCAL dwOrigFileSize : DWORD
    LOCAL dwNTHeaderAddr : DWORD
    LOCAL dwNewTlsSectionOffset : DWORD

    ;; init data
    xor eax, eax
    mov g_bError, al
    mov eax, OWNTLS_SIZE
    mov g_dwNewSectionSize, eax

    ;; open file
    invoke CreateFile, szFileName,\
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
    invoke GetFileSize, hFile, NULL
    .IF eax == 0
        invoke CloseHandle, hFile   
        jmp GetFileSizeFailed
    .ENDIF
    mov dwOrigFileSize, eax
    add eax, APPEND_SIZE
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
    invoke MapViewOfFile, hMap,
                     FILE_MAP_WRITE+FILE_MAP_READ+FILE_MAP_COPY, 
                     ebx, ebx, ebx
    .IF eax == 0
        invoke CloseHandle, hMap
        invoke CloseHandle, hFile
        jmp MapFileFailed
    .ENDIF
    mov pMem, eax                                        
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
    mov dwNTHeaderAddr, esi
    
    ;; 增加一个新节，此节为我们自己的 TLS 节
    invoke AddSection, pMem, offset g_szNewSectionName, g_dwNewSectionSize
    mov dwNewTlsSectionOffset, eax     
    
    ;; 寻找原始的 TLS 节并且记录它
    mov eax, dword ptr [esi].OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_TLS * sizeof IMAGE_DATA_DIRECTORY].VirtualAddress
    .IF eax == 0
        ;; 如果没有 TLS 节则汇编一个新的
        ;; 设置标志变量
        mov eax, offset OwnTLS_OrigYes
        push 0
        pop dword ptr [eax]
        jmp NoTLSCallBack
    .ENDIF
    invoke RVA2Offset, pMem, eax
    add eax, pMem
    ;; 复制这个表到我们自己的内存
    mov esi, eax
    lea edi, Orig_TLS
    mov ecx, sizeof IMAGE_TLS_DIRECTORY
    cld
    rep movsb
    ;; 汇编一个新的 TLS 节并设置回调函数    
    ;; 这里检验是否存在回调函数
    mov eax, offset Orig_TLS
    assume eax : ptr IMAGE_TLS_DIRECTORY
    mov eax, dword ptr [eax].AddressOfCallBacks
    ;; 首先将 VA 转换为 RVA 然后转换为 FVA
    mov esi, dwNTHeaderAddr
    assume esi : ptr IMAGE_NT_HEADERS
    sub eax, dword ptr [esi].OptionalHeader.ImageBase
    invoke RVA2Offset, pMem, eax
    add eax, pMem
    mov ebx, dword ptr [eax]
    test ebx, ebx
    jz NoTLSCallBack
    ;; 保存原回调函数的地址
    mov edx, offset OwnTLS_OrigCallBackEntry
    mov dword ptr [edx], ebx
    ;; 有回调函数这修改，修改位置为 eax 指向的地方
    mov ebx, dwNewTlsSectionOffset
    assume ebx : ptr IMAGE_SECTION_HEADER
    mov ebx, dword ptr [ebx].VirtualAddress
    add ebx, dword ptr [esi].OptionalHeader.ImageBase
    mov dword ptr [eax], ebx
    ;; 设置 TLS 回调跳转标志
    mov eax, offset OwnTLS_OrigYes
    push 1
    pop dword ptr [eax]
    jmp ContinueNoTLSCallBack
NoTLSCallBack:
    ;; 如果没有回调函数，则重新建立一个回调函数地址表
    ;; 并且设置新的 TLS 目录的字段
    mov ebx, dwNewTlsSectionOffset
    assume ebx : ptr IMAGE_SECTION_HEADER
    mov edx, dword ptr [ebx].VirtualAddress
    add edx, offset OwnTLS_CallBackStart - offset OwnTLS_Section
    add edx, dword ptr [esi].OptionalHeader.ImageBase
    mov eax, offset Orig_TLS
    assume eax : ptr IMAGE_TLS_DIRECTORY
    mov dword ptr [eax].AddressOfCallBacks, edx
    ;; 设置 AddressOfCallBacks
    sub edx, offset OwnTLS_CallBackStart - offset OwnTLS_CallBack
    mov ebx, offset OwnTLS_CallBackStart
    mov dword ptr [ebx], edx
    ;; 设置 AddressOfIndex
    add edx, offset OwnTLS_Index - offset OwnTLS_CallBack
    mov dword ptr [eax].AddressOfIndex, edx
    ;; 设置 StartAddressOfRawData
    add edx, offset OwnTLS_Start - offset OwnTLS_Index
    mov dword ptr [eax].StartAddressOfRawData, edx
    ;; 设置 EndAddressOfRawData
    add edx, offset OwnTLS_End - offset OwnTLS_Start
    mov dword ptr [eax].EndAddressOfRawData, edx
    ;; 设置 SizeOfZeroFill
    push 0
    pop dword ptr[eax].SizeOfZeroFill
ContinueNoTLSCallBack:
    ;; 这里复制 TLS 到新的 TLS 表中
    mov eax, dwNewTlsSectionOffset
    assume eax : ptr IMAGE_SECTION_HEADER
    ;; 将数据目录设置到我们自己的 TLS 节内
    mov eax, dword ptr [eax].VirtualAddress
    add eax, offset Orig_TLS - offset OwnTLS_Section
    mov dword ptr [esi].OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_TLS * sizeof IMAGE_DATA_DIRECTORY].VirtualAddress, eax
    push sizeof IMAGE_TLS_DIRECTORY
    pop dword ptr [esi].OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_TLS * sizeof IMAGE_DATA_DIRECTORY].isize
     
    ;; 复制内容到新的 TLS 节
    mov edi, dwNewTlsSectionOffset
    assume edi : ptr IMAGE_SECTION_HEADER
    mov edi, dword ptr [edi].PointerToRawData
    add edi, pMem
    mov esi, offset OwnTLS_Section
    mov ecx, g_dwNewSectionSize
    cld
    rep movsb
     
    ;; close handle & write it
    invoke UnmapViewOfFile, pMem
    invoke CloseHandle, hMap
    invoke CloseHandle, hFile                                           
 LogicShellExit:
     .IF g_bError == 0
         ;; show success message   
         invoke MessageBox, NULL, offset g_szDone, offset g_szDoneCap, MB_ICONINFORMATION
     .ENDIF         
     assume eax : nothing
     assume ebx : nothing
     assume edx : nothing
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
ShowErr:
     invoke MessageBox, NULL, eax, offset g_szErr, MB_ICONERROR
     mov al, 1
     mov g_bError, al
     jmp LogicShellExit

;; ----- 自己的 TLS 数据 -----
OwnTLS_Section:
;; 我们自己的 TLS 回调函数
OwnTLS_CallBack:
    ;; 只判断它的启动状态
    OwnTLS_CallBackArg_Reason           equ 08h
    ;; 这里没有做堆栈帧所以 esp 指向的是返回地址和
    ;; 而我们所用的参数 Reason 是第 2 个所以这里加 08h
    mov eax, dword ptr [esp+OwnTLS_CallBackArg_Reason]
    cmp eax, DLL_PROCESS_ATTACH
    jnz ExitOwnTLS_CallBack
    ;; 在这里停下,把注释停掉，可以在 ollydbg 里调试查看
    ;int 3
ExitOwnTLS_CallBack:         
    ;; 这里做一个判断，如果是新的 TLS 则直接退出如果是修改的 TLS 则跳入原来的
    call GetEip
    GetEip:
    pop eax
    mov edx, eax
    add eax, offset OwnTLS_OrigYes - offset GetEip
    mov eax, dword ptr [eax]
    test eax, eax   ; 0 为原来没有 TLS,1 为有 TLS 回调
    ;; 这里取出原始的 TLS 回调函数入口并跳入
    jz NotHaveOrigTlsCallBack
    add edx, offset OwnTLS_OrigCallBackEntry - offset GetEip
    jmp dword ptr [edx]       ; 跳入正常的回调函数入口
NotHaveOrigTlsCallBack:
    retn 03h                   ; 清除参数造成的堆栈开销     
    ;; ----- 原始回调函数入口 -----          
;; 原来的程序是否有回调函数
OwnTLS_OrigYes               dd  0   
;; 原 TLS 回调函数入口点
OwnTLS_OrigCallBackEntry     dd  0
;; TLS 变量索引
OwnTLS_Index                 dd  0
;; TLS 回调函数地址表
OwnTLS_CallBackStart         dd  0
OwnTLS_CallBackEnd           dd  0
;; TLS 变量区域
OwnTLS_Start:
    db 80h dup ("tls")
OwnTLS_End:
;; 保存原始的 TLS 节目录
Orig_TLS IMAGE_TLS_DIRECTORY <0>
EndOwnTLS_Section:

ModifyTLS endp

```

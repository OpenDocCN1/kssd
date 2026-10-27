
```
EnCryptIID proc uses ebx ecx edx esi edi, pMem : LPVOID, pCurrentIID : LPVOID

;; encrypt image import directory

    LOCAL pIID[IMAGE_IMPORT_TABLE_SIZE] : BYTE

    LOCAL pImportFVA : LPVOID

    LOCAL pImportRVA : DWORD

    LOCAL dwLoadLibraryThunkData : DWORD

    LOCAL dwGetProcAddressThunkData : DWORD

    

    [COLOR=Red];; 将IID_PRIVATE_DATA结构清0[/COLOR]

    mov edi, pCurrentIID

    mov eax, sizeof IID_PRVATE_DATA

    mov ecx, MAX_IID_NUM

    imul ecx

    xchg eax, ecx

    xor eax, eax

    cld

    rep stosb



    [COLOR=Red];; 定位引入表的位置[/COLOR]

    mov esi, pMem

    add esi, dword ptr [esi+03ch]

    assume esi : ptr IMAGE_NT_HEADERS

    mov ecx, dword ptr [esi].OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT * sizeof IMAGE_DATA_DIRECTORY].isize    

    test ecx, ecx

    jz ExitEnCryptIID

    mov esi, dword ptr [esi].OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT * sizeof IMAGE_DATA_DIRECTORY].VirtualAddress

    test esi, esi

    jz ExitEnCryptIID

    ;; change RVA to offset

    invoke RVA2Offset, pMem, esi

    add eax, pMem

    xchg eax, edi           ; esi = Import Symbol Table on file

    [COLOR=Red];; 清除原始的引入表[/COLOR]

    assume edi : ptr IMAGE_IMPORT_DESCRIPTOR

    mov esi, pCurrentIID

    assume esi : ptr IID_PRIVATE_DATA

    xor ebx, ebx            ; ebx = max of IID

ClearOrigIIDLoop:

    mov eax, dword ptr [edi].Name1

    test eax, eax           ; eax = dll name string RVA

    jz ExitClearOrigIIDLoop

    inc ebx

    test ebx, MAX_IID_NUM

    jnz ExitClearOrigIIDLoop

    

    [COLOR=Red];; 储存原始IID的信息,保存到我们自己的IID结构,供解密用[/COLOR]

    push dword ptr [edi].Name1

    pop dword ptr [esi].Name1

    push dword ptr [edi].OriginalFirstThunk

    pop dword ptr [esi].OriginalFirstThunk

    push dword ptr [edi].FirstThunk

    pop dword ptr [esi].FirstThunk

    

    ;; change Name1 RVA to offset

    invoke RVA2Offset, pMem, eax

    add eax, pMem



    [COLOR=Red];; 加密DLL库文件的文件名, 这个函数将字符串的每个字节异或上99h[/COLOR]

    invoke EnCryptString, eax

    

    [COLOR=Red];; 加密引入表和显示引入表的情况相同，把显示换成加密即可[/COLOR]

    mov eax, dword ptr [edi].OriginalFirstThunk

    test eax, eax

    jnz UseOriginalFirstThunk

    mov eax, dword ptr [edi].FirstThunk

UseOriginalFirstThunk:

    ;; change Api Name string RVA to offset

    invoke RVA2Offset, pMem, eax

    add eax, pMem

    xchg edx, eax       ; edx = Api offset in file

    push esi

EnCryptApiNameLoop:

    mov esi, dword ptr [edx]

    test esi, esi

    jz EndEnCryptApiNameLoop    

    ;; judege IMAGE_ORDINAL_FLAG32 flags

    test esi, IMAGE_ORDINAL_FLAG32

    jnz SkipEncrypt

    invoke RVA2Offset, pMem, esi

    test eax, eax

    jz SkipEncrypt 

    add eax, pMem

    add eax, 02h        ; skip HINT

    invoke EnCryptString, eax

SkipEncrypt:

    add edx, 04h   

    jmp EnCryptApiNameLoop

EndEnCryptApiNameLoop:

    pop esi             ; esi = the point of current own IID

    [COLOR=Red];; 这里是毁坏原始IID[/COLOR]

    push 0

    pop dword ptr [edi].Name1

    push 0

    pop dword ptr [edi].OriginalFirstThunk

    push 0

    pop dword ptr [edi].FirstThunk

    push 0

    pop dword ptr [edi].TimeDateStamp

    push 0

    pop dword ptr [edi].ForwarderChain

    

    ;; mov to next IID

    add edi, sizeof IMAGE_IMPORT_DESCRIPTOR

    add esi, sizeof IID_PRIVATE_DATA

    jmp ClearOrigIIDLoop

ExitClearOrigIIDLoop:



    ;; clear the new IID

    lea edi, pIID

    mov ecx, IMAGE_IMPORT_TABLE_SIZE

    cld

    xor al, al

    rep stosb

    sub edi, IMAGE_IMPORT_TABLE_SIZE            ; set edi back to start point

    

    [COLOR=Red];; 增加一个我们自己的IID节,随后会将构建的IID写入到此节中[/COLOR]

    invoke AddSection, pMem, NULL, IMAGE_IMPORT_TABLE_SIZE

    mov pImportFVA, eax    

    assume eax : ptr IMAGE_SECTION_HEADER

[COLOR=Red]    ;; 这里创建一个新的IID

    ;; 我们的IID表只导入一个DLL那就是基本的kernel32.dll引入的函数也只有两个函数

    ;; LoadLibraryA,GetProcAddress两个函数。在解密节中利用这两个最基本的函数获取

    ;; 其他的API地址与DLL句柄

    ;; 结构如下

    ;; 构建新的IID按照以下结构填写即可[/COLOR]

    ;; make new IID

    ;; ----- Image Import Descriptor -----

    ;; IMAGE_IMPORT_DESCRIPTOR

    ;; IMAGE_IMPORT_DESCRIPTOR(0)

    ;; IMAGE_THUNK_DATA

    ;; IMAGE_THUNK_DATA

    ;; IMAGE_THUNK_DATA(0)

    ;; kernel32.dll,0

    ;; IMAGE_IMPORT_BY_NAME(LoadLibraryA)

    ;; IMAGE_IMPORT_BY_NAME(GetProcAddress)

    lea edx, pIID

    assume edx : ptr IMAGE_IMPORT_DESCRIPTOR

    mov ecx, sizeof IMAGE_IMPORT_DESCRIPTOR

    add ecx, sizeof IMAGE_IMPORT_DESCRIPTOR

    add ecx, sizeof IMAGE_THUNK_DATA

    add ecx, sizeof IMAGE_THUNK_DATA

    add ecx, sizeof IMAGE_THUNK_DATA

    mov eax, dword ptr [eax].VirtualAddress

    mov pImportRVA, eax

    add eax, ecx                                ; ecx = kernel32.dll string offset

    mov dword ptr [edx].Name1, eax

    

    [COLOR=Red];; 拷贝kernel32.dll 字符串到 文件[/COLOR]

    mov esi, offset g_szKernelDll

    add edi, ecx

CopyKernel32StrLoop:

    mov al, byte ptr [esi]

    test al, al

    jz EndCopyKernel32StrLoop

    mov byte ptr [edi], al

    inc esi

    inc edi

    jmp CopyKernel32StrLoop

EndCopyKernel32StrLoop:

    mov byte ptr [edi], al

    inc edi

    

    [COLOR=Red];; 设置LoadLibraryA的IMAGE_IMPORT_BY_NAME[/COLOR]

    mov eax, edi

    sub eax, edx

    mov dwLoadLibraryThunkData, eax

    xor eax, eax

    mov word ptr [edi], ax

    add edi, 02h

    mov esi, offset g_szLoadLibrary

CopyLoadLibraryLoop:

    mov al, byte ptr [esi]

    test al, al

    jz EndCopyLoadLibraryLoop

    mov byte ptr [edi], al

    inc esi

    inc edi

    jmp CopyLoadLibraryLoop

EndCopyLoadLibraryLoop:

    mov byte ptr [edi], al

    inc edi

    

    [COLOR=Red];; 设置GetProcAddress的IMAGE_IMPORT_BY_NAME结构[/COLOR]

    mov eax, edi

    sub eax, edx

    mov dwGetProcAddressThunkData, eax  

    xor eax, eax

    mov word ptr [edi], ax

    add edi, 02h

    mov esi, offset g_szGetProcAddress

CopyGetProcAddressLoop:

    mov al, byte ptr [esi]

    test al, al

    jz EndCopyGetProcAddressLoop

    mov byte ptr [edi], al

    inc esi

    inc edi

    jmp CopyGetProcAddressLoop

EndCopyGetProcAddressLoop:   

    mov byte ptr [edi], al

    inc edi

     

[COLOR=Red]    ;; 设置IMAGE_IMPORT_DESCRIPTOR

    ;; 我们这里设置自己的IMAGE_IMPORT_DESCRIPTOR结构，单使用FirstThunk字段

    ;; 将OriginalFirstThunk设置为0[/COLOR]

    xor eax, eax

    mov dword ptr [edx].Characteristics, eax

    mov dword ptr [edx].TimeDateStamp, eax

    mov dword ptr [edx].ForwarderChain, eax

    push 0

    pop dword ptr [edx].OriginalFirstThunk

    mov eax, pImportRVA

    add eax, sizeof IMAGE_IMPORT_DESCRIPTOR

    add eax, sizeof IMAGE_IMPORT_DESCRIPTOR

    mov dword ptr [edx].FirstThunk, eax

    sub eax, pImportRVA                         ;  eax = 2 of IMAGE_IMPORT_DESCRIPTOR SIZE

    add edx, eax

    mov eax, dwLoadLibraryThunkData

    add eax, pImportRVA

    mov dword ptr [edx], eax

    add edx, 04h                                ; move to next point

    mov eax, dwGetProcAddressThunkData

    add eax, pImportRVA

    mov dword ptr [edx], eax

    

    [COLOR=Red];; 拷贝新的IID到文件[/COLOR]

    lea esi, pIID

    mov edi, pImportFVA

    assume edi : ptr IMAGE_SECTION_HEADER

    mov edi, dword ptr [edi].PointerToRawData    

    add edi, pMem

    mov ecx, IMAGE_IMPORT_TABLE_SIZE

    cld

    rep movsb

    

[COLOR=Red]    ;; 修改IT表的虚拟地址和尺寸

    ;; 设置这个数据目录以便PE加载器可以寻找到IID表[/COLOR]

    mov esi, pMem

    add esi, dword ptr [esi+03ch]

    assume esi : ptr IMAGE_NT_HEADERS

    mov eax, pImportFVA

    assume eax : ptr IMAGE_SECTION_HEADER

    push dword ptr [eax].VirtualAddress

    pop dword ptr [esi].OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT * sizeof IMAGE_DATA_DIRECTORY].VirtualAddress

    push dword ptr [eax].Misc.VirtualSize

    pop dword ptr [esi].OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT * sizeof IMAGE_DATA_DIRECTORY].isize

    

    assume edx : nothing

    assume eax : nothing

    assume esi : nothing

    assume edi : nothing

    xor eax, eax

ExitEnCryptIID:

    ret

EnCryptIID endp
```

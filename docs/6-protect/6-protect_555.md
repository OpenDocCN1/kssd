

# 【成果 6.2】软件保护壳技术专题 - 加密引入表

> 原文：[`www.pediy.com/kssd/pediy10/67098.html`](https://www.pediy.com/kssd/pediy10/67098.html)

导入表的加密应该是壳对应用程序加密时最重要的部分了。首先我们先了解一下怎样寻找引入表，并列出其中的导入的函数与 DLL 命。其次，进行讲解如何加密引入表。最后，讲解在我们自定义的节表中恢复引入表并重定位函数地址。  
  
1.寻找引入表  
在 IMAGE\_NT\_HEADERS 结构中的 OptionalHeader 字段中有一组数据目录，数据目录数组的第二个元素就是引入表的目录索引。  
数据目录的结构是这样的  
IMAGE\_DATA\_DIRECTORY STRUCT   
  VirtualAddress dd ?   
  isize dd ?   
IMAGE\_DATA\_DIRECTORY ENDS   
VirtualAddress 是此表的 RVA,也就是相对于模块加载的偏移量，此地址指向一个由 IMAGE\_IMPORT\_DESCRIPTOR 结构组成的数组。  
isize 是此表的大小  
我们利用以下算法寻找引入表  
1.从 DOS header 找到 PE header   
2.从 数据目录中 读取 data directory 的地址。 第二个索引就为引入表的地址。  
3.IMAGE\_DATA\_DIRECTORY 的虚拟地址偏移转化为文件偏移  
4.文件偏移加上文件映射基址就为引入表的地址  
寻找到引入表后，接下来的工作就是展开引入表,此时的指针指向一个 IMAGE\_IMPORT\_DESCRIPTOR 结构的数组，我们可以认为一个 IMAGE\_IMPORT\_DESCRIPTOR 结构就是一个 DLL，如果你的程序引用了 5 个 DLL 那么你的程序用就有 5 个 IMAGE\_IMPORT\_DESCRIPTOR 结构，并且这个数组以一个全 0 的 IMPORT\_IMPORT\_DESCRIPTOR 为结束。为了让读者好理解在以下称 IMAGE\_IMPORT\_DESCRIPTOR 为 DLL 结构。  
对于加密导入表最重要的有三个属性,Name1,OriginalFirstThunk,FirstThunk。  
Name1 也是一个偏移量,指向这个 DLL 的 DLL 字符串的内存偏移。  
OriginalFirstThunk 与 FirstThunk 两个字段也同属于指针类型的。并且在文件中都指向一个位置。当加载到内存时，OriginalFirstThunk 还指向原来指向的地方，FirstThunk 指向一个 API 函数的地址的数据。(由 PE 加载器帮助定位并修改的)  
在文件中，OriginalFirstThunk 指向一组地址的偏移,这个地址偏移被称为 IMAGE\_THUNK\_DATA  
这个偏移值指向一组 IMAGE\_IMPORT\_BY\_NAME 结构的数组。这个结构读者可以认为是这个 DLL 文件中的 API。有几个 API 代表有几个这样的结构。  
IMAGE\_IMPORT\_BY\_NAME STRUCT   
  Hint dw ?   
  Name1 db ?   
IMAGE\_IMPORT\_BY\_NAME ENDS  
Hint:代表此 API 是 DLL 中的第几个函数  
Name1:为此 API 的名字，最后以 NULL 结尾。我们加密 API 的名字就是加密 Name1 的字段。  
另一个 FirstThunk 如同复制一样也一模一样的指向了与 OrigFirstThunk 一样的地方。但是当文件加载到内存中后，FirstThunk 会指向函数的地址。这个转换由 PE 加载器加载。  
列出引入表代码如下  

代码:

```
ListIID proc pFilename : LPSTR
    ;; map file to memory
    LOCAL hFile : HANDLE
    LOCAL hMap : HANDLE
    LOCAL pMem : LPVOID
    LOCAL dwNTHeaderAddr : DWORD
    LOCAL szTmpBuf\[MAX\_PATH\] : BYTE
    
    ;; open file
    invoke CreateFile, pFilename,\\
                      GENERIC\_WRITE + GENERIC\_READ,\\
                      FILE\_SHARE\_WRITE + FILE\_SHARE\_READ,\\
                      NULL,\\
                      OPEN\_EXISTING,\\
                      FILE\_ATTRIBUTE\_NORMAL,\\
                      0
    .IF eax == INVALID\_HANDLE\_VALUE
        jmp OpenFileFailed                
    .ENDIF
    mov hFile, eax 
    invoke GetFileSize, hFile, NULL
    .IF eax == 0
        invoke CloseHandle, hFile  
        jmp GetFileSizeFailed
    .ENDIF  
    
    ;; create memory map
    xor ebx, ebx     
    invoke CreateFileMapping, hFile, ebx, PAGE\_READWRITE, ebx, eax, ebx
    .IF eax == 0
        invoke CloseHandle, hFile
        jmp CreateMapFailed                
    .ENDIF
    mov hMap, eax
    ;; map file to memory
    invoke MapViewOfFile, hMap,
                       FILE\_MAP\_WRITE+FILE\_MAP\_READ+FILE\_MAP\_COPY, 
                       ebx, ebx, ebx
    .IF eax == 0
        invoke CloseHandle, hMap
        invoke CloseHandle, hFile
        jmp MapFileFailed
    .ENDIF
    mov pMem, eax                               
    ;; check it's PE file or not ?
    xchg eax, esi
    assume esi : ptr IMAGE\_DOS\_HEADER
    .IF \[esi\].e\_magic != 'ZM'
        invoke UnmapViewOfFile, pMem
        invoke CloseHandle, hMap
        invoke CloseHandle, hFile
        jmp InvalidPE        
    .ENDIF       
    add esi, \[esi\].e\_lfanew
    assume esi : ptr IMAGE\_NT\_HEADERS   
    .IF word ptr \[esi\].Signature != 'EP'
        invoke UnmapViewOfFile, pMem
        invoke CloseHandle, hMap
        invoke CloseHandle, hFile
        jmp InvalidPE        
    .ENDIF
    mov dwNTHeaderAddr, esi
    
    ;; 寻找引入表
    assume esi : ptr IMAGE\_NT\_HEADERS
    mov eax, dword ptr \[esi\].OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IMPORT \* sizeof IMAGE\_DATA\_DIRECTORY\].VirtualAddress
    ;; 将内存偏移转换为文件偏移,再加上文件头的内存映射就等于引入表在文件的地址了
    invoke RVA2Offset, pMem, eax
    xchg ebx, eax
    add ebx, pMem
    assume ebx : ptr IMAGE\_IMPORT\_DESCRIPTOR
    ;; 这里直到一个全 0 的 IMAGE\_IMPORT\_DESCRIPTOR 结构为结束
    ;; 这里判断 Name1 是否为 NULL
    ListIIDLoop:
        mov eax, dword ptr \[ebx\].Name1
        test eax, eax
        jz EndListIIDLoop
        ;; 此时 eax 指向一个 DLL 名字的 RVA,我们将 RVA 转换为文件的偏移
        invoke RVA2Offset, pMem, eax
        ;; 文件的偏移加上文件的起始指针就为文件中的地址
        add eax, pMem
        ;; 打印 DLL 名
        invoke PrintLine, offset g\_szOutFormat, offset g\_szOutLine        
        invoke PrintLine, offset g\_szOutFormat, eax
        invoke PrintLine, offset g\_szOutFormat, offset g\_szOutLine
        ;; 检查 OriginalFirstThunk 是否为 0。如果为 0 则使用 FirstThunk        
        mov edx, dword ptr \[ebx\].OriginalFirstThunk
        test edx, edx
        jnz UseOrignalFirstThunk
        mov edx, dword ptr \[ebx\].FirstThunk
    UseOrignalFirstThunk:        
        ;; 转换 RVA 转换文件偏移
        invoke RVA2Offset, pMem, edx
        add eax, pMem
        mov edx, eax
    DisplayApiName:
        ;; 查看 edx 的最高位是否是 1 确定是否以序数引出
        test dword ptr \[edx\], IMAGE\_ORDINAL\_FLAG32
        jnz DisPlayOrd
        ;; 这里 edx 指向 IMAGE\_IMPORT\_BY\_NAME 结构的 RVA,继续将它转换
        mov eax, dword ptr \[edx\]
        invoke RVA2Offset, pMem, eax
        add eax, pMem
        assume eax : ptr IMAGE\_IMPORT\_BY\_NAME
        ;; 打印 API 字符串,将 Name1 的地址设置给 eax 寄存器
        lea eax, \[eax\].Name1
        invoke PrintLine, offset g\_szOutFormat, eax
        jmp NextAPI
        DisPlayOrd:
        ;; 取出序数,低 2 个字节为序数
        mov eax, dword ptr \[edx\]
        and eax, 0FFFFh        
        invoke PrintLine, offset g\_szOutOrdFormat, eax
    NextAPI:
        ;; 取下一个 IMAGE\_THUNK\_DATA 的值
        add edx, 04h
        ;; 直到 edx 指向一个 0
        mov eax, dword ptr \[edx\]
        test eax, eax
        jnz DisplayApiName
        ;; 指向下一个 DLL
        add ebx, sizeof IMAGE\_IMPORT\_DESCRIPTOR
        jmp ListIIDLoop
    EndListIIDLoop:   
LogicShellExit:
    ;; close handle & write it
    invoke UnmapViewOfFile, pMem
    invoke CloseHandle, hMap
    invoke CloseHandle, hFile
    assume ebx : nothing
    assume esi : nothing
    ret
;; ----- Show error message ----- 
OpenFileFailed:
     lea eax, g\_szOpenFileFailed
     jmp ShowErr
GetFileSizeFailed:
     lea eax, g\_szGetFileSizeFailed
     jmp ShowErr    
CreateMapFailed:
     lea eax, g\_szCreateMapFailed
     jmp ShowErr
MapFileFailed:
     lea eax, g\_szMapFileFailed
     jmp ShowErr        
InvalidPE:          
     lea eax, g\_szInvalidPE
     jmp ShowErr   
ShowErr:
     invoke MessageBox, NULL, eax, offset g\_szErr, MB\_ICONERROR
     jmp LogicShellExit
ListIID endp

```

列出引入表不知道我讲的是否清楚，总之当从 PE 头结构中获取到引入表地址将它的 RVA 转换为 FVA 再加上文件映射基址就可以按照微软定义的引入表结构进行加密与销毁。接下来我们要做的就是将引入表中每个 DLL 名字与 API 的名字都进行加密，后记录每个 IMAGE\_IMPORT\_DESCRIPTOR 中的 FirstThunk 值。并记录到我们自己的定义的结构中。RVA2Offset 函数是转换 RVA 到 FVA。此函数在附件代码中定义。  
  
前面的只是给加密引入表做个铺垫。加密引入表和列出引入表算法都一样。只不过将显示函数加密而已。这里直接将代码列出。加密字符串的算法也很简单只是异或上 99h 而已。其次和列出不一样的地方是，加密引入表记录了引入表每个 IMAGE\_IMPORT\_DESCRIPTOR 的 Name1,OrigFirstThunk,FirstThunk 三个字段。然后在解密段中按此三个字段在解密引入表。并重定位 API 的地址表。  
这里是我们自己的引入表结构。这个结构安置在解密体内  
代码:

```
IID\_PRIVATE\_DATA struct
    Name1                   dd  0
    OriginalFirstThunk      dd  0
    FirstThunk              dd  0
IID\_PRIVATE\_DATA ends

```

下面列出其加密引入表的代码  
代码:

```
EnCryptIID proc uses ebx ecx edx esi edi, pMem : LPVOID, pCurrentIID : LPVOID
;; encrypt image import directory
    LOCAL pIID\[IMAGE\_IMPORT\_TABLE\_SIZE\] : BYTE
    LOCAL pImportFVA : LPVOID
    LOCAL pImportRVA : DWORD
    LOCAL dwLoadLibraryThunkData : DWORD
    LOCAL dwGetProcAddressThunkData : DWORD
    
    ;; 将 IID\_PRIVATE\_DATA 结构清 0
    mov edi, pCurrentIID
    mov eax, sizeof IID\_PRVATE\_DATA
    mov ecx, MAX\_IID\_NUM
    imul ecx
    xchg eax, ecx
    xor eax, eax
    cld
    rep stosb

    ;; 定位引入表的位置
    mov esi, pMem
    add esi, dword ptr \[esi+03ch\]
    assume esi : ptr IMAGE\_NT\_HEADERS
    mov ecx, dword ptr \[esi\].OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IMPORT \* sizeof IMAGE\_DATA\_DIRECTORY\].isize    
    test ecx, ecx
    jz ExitEnCryptIID
    mov esi, dword ptr \[esi\].OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IMPORT \* sizeof IMAGE\_DATA\_DIRECTORY\].VirtualAddress
    test esi, esi
    jz ExitEnCryptIID
    ;; change RVA to offset
    invoke RVA2Offset, pMem, esi
    add eax, pMem
    xchg eax, edi           ; esi = Import Symbol Table on file
    ;; 清除原始的引入表
    assume edi : ptr IMAGE\_IMPORT\_DESCRIPTOR
    mov esi, pCurrentIID
    assume esi : ptr IID\_PRIVATE\_DATA
    xor ebx, ebx            ; ebx = max of IID
ClearOrigIIDLoop:
    mov eax, dword ptr \[edi\].Name1
    test eax, eax           ; eax = dll name string RVA
    jz ExitClearOrigIIDLoop
    inc ebx
    test ebx, MAX\_IID\_NUM
    jnz ExitClearOrigIIDLoop
    
    ;; 储存原始 IID 的信息,保存到我们自己的 IID 结构,供解密用
    push dword ptr \[edi\].Name1
    pop dword ptr \[esi\].Name1
    push dword ptr \[edi\].OriginalFirstThunk
    pop dword ptr \[esi\].OriginalFirstThunk
    push dword ptr \[edi\].FirstThunk
    pop dword ptr \[esi\].FirstThunk
    
    ;; change Name1 RVA to offset
    invoke RVA2Offset, pMem, eax
    add eax, pMem

    ;; 加密 DLL 库文件的文件名, 这个函数将字符串的每个字节异或上 99h
    invoke EnCryptString, eax
    
    ;; 加密引入表和显示引入表的情况相同，把显示换成加密即可
    mov eax, dword ptr \[edi\].OriginalFirstThunk
    test eax, eax
    jnz UseOriginalFirstThunk
    mov eax, dword ptr \[edi\].FirstThunk
UseOriginalFirstThunk:
    ;; change Api Name string RVA to offset
    invoke RVA2Offset, pMem, eax
    add eax, pMem
    xchg edx, eax       ; edx = Api offset in file
    push esi
EnCryptApiNameLoop:
    mov esi, dword ptr \[edx\]
    test esi, esi
    jz EndEnCryptApiNameLoop    
    ;; judege IMAGE\_ORDINAL\_FLAG32 flags
    test esi, IMAGE\_ORDINAL\_FLAG32
    jnz SkipEncrypt
    invoke RVA2Offset, pMem, esi
    test eax, eax
    jz SkipEncrypt 
    add eax, pMem
    add eax, 02h        ; skip HINT
    invoke EnCryptString, eax
SkipEncrypt:
    add edx, 04h   
    jmp EnCryptApiNameLoop
EndEnCryptApiNameLoop:
    pop esi             ; esi = the point of current own IID
    ;; 这里是毁坏原始 IID
    push 0
    pop dword ptr \[edi\].Name1
    push 0
    pop dword ptr \[edi\].OriginalFirstThunk
    push 0
    pop dword ptr \[edi\].FirstThunk
    push 0
    pop dword ptr \[edi\].TimeDateStamp
    push 0
    pop dword ptr \[edi\].ForwarderChain
    
    ;; mov to next IID
    add edi, sizeof IMAGE\_IMPORT\_DESCRIPTOR
    add esi, sizeof IID\_PRIVATE\_DATA
    jmp ClearOrigIIDLoop
ExitClearOrigIIDLoop:

    ;; clear the new IID
    lea edi, pIID
    mov ecx, IMAGE\_IMPORT\_TABLE\_SIZE
    cld
    xor al, al
    rep stosb
    sub edi, IMAGE\_IMPORT\_TABLE\_SIZE            ; set edi back to start point
    
    ;; 增加一个我们自己的 IID 节,随后会将构建的 IID 写入到此节中
    invoke AddSection, pMem, NULL, IMAGE\_IMPORT\_TABLE\_SIZE
    mov pImportFVA, eax    
    assume eax : ptr IMAGE\_SECTION\_HEADER
    ;; 这里创建一个新的 IID
    ;; 我们的 IID 表只导入一个 DLL 那就是基本的 kernel32.dll 引入的函数也只有两个函数
    ;; LoadLibraryA,GetProcAddress 两个函数。在解密节中利用这两个最基本的函数获取
    ;; 其他的 API 地址与 DLL 句柄
    ;; 结构如下
    ;; 构建新的 IID 按照以下结构填写即可
    ;; make new IID
    ;; ----- Image Import Descriptor -----
    ;; IMAGE\_IMPORT\_DESCRIPTOR
    ;; IMAGE\_IMPORT\_DESCRIPTOR(0)
    ;; IMAGE\_THUNK\_DATA
    ;; IMAGE\_THUNK\_DATA
    ;; IMAGE\_THUNK\_DATA(0)
    ;; kernel32.dll,0
    ;; IMAGE\_IMPORT\_BY\_NAME(LoadLibraryA)
    ;; IMAGE\_IMPORT\_BY\_NAME(GetProcAddress)
    lea edx, pIID
    assume edx : ptr IMAGE\_IMPORT\_DESCRIPTOR
    mov ecx, sizeof IMAGE\_IMPORT\_DESCRIPTOR
    add ecx, sizeof IMAGE\_IMPORT\_DESCRIPTOR
    add ecx, sizeof IMAGE\_THUNK\_DATA
    add ecx, sizeof IMAGE\_THUNK\_DATA
    add ecx, sizeof IMAGE\_THUNK\_DATA
    mov eax, dword ptr \[eax\].VirtualAddress
    mov pImportRVA, eax
    add eax, ecx                                ; ecx = kernel32.dll string offset
    mov dword ptr \[edx\].Name1, eax
    
    ;; 拷贝 kernel32.dll 字符串到 文件
    mov esi, offset g\_szKernelDll
    add edi, ecx
CopyKernel32StrLoop:
    mov al, byte ptr \[esi\]
    test al, al
    jz EndCopyKernel32StrLoop
    mov byte ptr \[edi\], al
    inc esi
    inc edi
    jmp CopyKernel32StrLoop
EndCopyKernel32StrLoop:
    mov byte ptr \[edi\], al
    inc edi
    
    ;; 设置 LoadLibraryA 的 IMAGE\_IMPORT\_BY\_NAME
    mov eax, edi
    sub eax, edx
    mov dwLoadLibraryThunkData, eax
    xor eax, eax
    mov word ptr \[edi\], ax
    add edi, 02h
    mov esi, offset g\_szLoadLibrary
CopyLoadLibraryLoop:
    mov al, byte ptr \[esi\]
    test al, al
    jz EndCopyLoadLibraryLoop
    mov byte ptr \[edi\], al
    inc esi
    inc edi
    jmp CopyLoadLibraryLoop
EndCopyLoadLibraryLoop:
    mov byte ptr \[edi\], al
    inc edi
    
    ;; 设置 GetProcAddress 的 IMAGE\_IMPORT\_BY\_NAME 结构
    mov eax, edi
    sub eax, edx
    mov dwGetProcAddressThunkData, eax  
    xor eax, eax
    mov word ptr \[edi\], ax
    add edi, 02h
    mov esi, offset g\_szGetProcAddress
CopyGetProcAddressLoop:
    mov al, byte ptr \[esi\]
    test al, al
    jz EndCopyGetProcAddressLoop
    mov byte ptr \[edi\], al
    inc esi
    inc edi
    jmp CopyGetProcAddressLoop
EndCopyGetProcAddressLoop:   
    mov byte ptr \[edi\], al
    inc edi
     
    ;; 设置 IMAGE\_IMPORT\_DESCRIPTOR
    ;; 我们这里设置自己的 IMAGE\_IMPORT\_DESCRIPTOR 结构，单使用 FirstThunk 字段
    ;; 将 OriginalFirstThunk 设置为 0
    xor eax, eax
    mov dword ptr \[edx\].Characteristics, eax
    mov dword ptr \[edx\].TimeDateStamp, eax
    mov dword ptr \[edx\].ForwarderChain, eax
    push 0
    pop dword ptr \[edx\].OriginalFirstThunk
    mov eax, pImportRVA
    add eax, sizeof IMAGE\_IMPORT\_DESCRIPTOR
    add eax, sizeof IMAGE\_IMPORT\_DESCRIPTOR
    mov dword ptr \[edx\].FirstThunk, eax
    sub eax, pImportRVA                         ;  eax = 2 of IMAGE\_IMPORT\_DESCRIPTOR SIZE
    add edx, eax
    mov eax, dwLoadLibraryThunkData
    add eax, pImportRVA
    mov dword ptr \[edx\], eax
    add edx, 04h                                ; move to next point
    mov eax, dwGetProcAddressThunkData
    add eax, pImportRVA
    mov dword ptr \[edx\], eax
    
    ;; 拷贝新的 IID 到文件
    lea esi, pIID
    mov edi, pImportFVA
    assume edi : ptr IMAGE\_SECTION\_HEADER
    mov edi, dword ptr \[edi\].PointerToRawData    
    add edi, pMem
    mov ecx, IMAGE\_IMPORT\_TABLE\_SIZE
    cld
    rep movsb
    
    ;; 修改 IT 表的虚拟地址和尺寸
    ;; 设置这个数据目录以便 PE 加载器可以寻找到 IID 表
    mov esi, pMem
    add esi, dword ptr \[esi+03ch\]
    assume esi : ptr IMAGE\_NT\_HEADERS
    mov eax, pImportFVA
    assume eax : ptr IMAGE\_SECTION\_HEADER
    push dword ptr \[eax\].VirtualAddress
    pop dword ptr \[esi\].OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IMPORT \* sizeof IMAGE\_DATA\_DIRECTORY\].VirtualAddress
    push dword ptr \[eax\].Misc.VirtualSize
    pop dword ptr \[esi\].OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IMPORT \* sizeof IMAGE\_DATA\_DIRECTORY\].isize
    
    assume edx : nothing
    assume eax : nothing
    assume esi : nothing
    assume edi : nothing
    xor eax, eax
ExitEnCryptIID:
    ret
EnCryptIID endp

```

最后要讲解的就是解密引入表了。解密引入表比起加密来要复杂一些。我们的具体思路是这样的。建立两个节，一个为新的引入表，一个为解密节，这个节也设置为起始的节。  
解密引入表，两种解密的方式，一是直接解密后将获取的 API 地址填写入 FirstThunk 字段指向的区域，另一种方式是重定位引入地址表。  
例如，在我们的程序中调用 CreateFileA 这个函数。call dword ptr \[XXXX\], 这个 XXXX 其实就是 FirstThunk 指向的那片区域中的一个地址而已，XXXX 指向的地址，如果没有经过我们重定位那么直接就是跟的这个 API 的地址，如果经过我们重定位将这个 XXXX 指向的地址指到我们分配的内存中例如 YYYY,那么最终将会转到 YYYY 指向，我们再在 YYYY 这处地址写入例如 JMP ZZZZ 样的跳转指令,其中 ZZZZ 代表的是系统 API 地址到 YYYY 这个地址的偏移量。那么当程序调用 API 时最终通过我们设置的一篇代理代码将跳转到 API 内执行。  
这里主要的是，我们只重定位系统自身的 DLL，如果是第三方的 DLL，我们直接将它的地址设置到其 FirstThunk 指向的区域就好了。在 NT 系统下，加载第三方 DLL 是在小于 070000000h,大于 077FFFFFFh,9x 下为小于 080000000h。  
解密节的代码如下  
代码:

```
;; ----- 解密 IID -----
DecryptIID:
DecryptIIDStackSize         equ 20h
DecryptIIDEip               equ -04h
DecryptIIDImageBase         equ -08h
DecryptIIDKernel32Dll       equ -0ch
DecryptIIDLoadLibraryA      equ -10h
DecryptIIDGetProcAddress    equ -14h
DecryptIIDGlobalAlloc       equ -18h
DecryptIIDGetVersion        equ -1ch
DecryptIIDIsNT              equ -20h
        
    push ebp
    mov esp, ebp
    sub esp, DecryptIIDStackSize
    ;; 获取新的 EIP
    call GetEip
    GetEip:
    pop eax
    sub eax, offset GetEip - offset DecryptIID
    mov dword ptr \[ebp+DecryptIIDEip\], eax
    ;; 获取 ImageBase
    add eax, offset DecryptIID\_ImageBase - offset DecryptIID
    mov eax, dword ptr \[eax\]
    mov dword ptr \[ebp+DecryptIIDImageBase\], eax
    ;; 获取引入表地址地址
    add eax, dword ptr \[eax+3ch\]
    assume eax : ptr IMAGE\_NT\_HEADERS
    mov eax, dword ptr \[eax\].OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IMPORT \* sizeof IMAGE\_DATA\_DIRECTORY\].VirtualAddress
    ;; 这里就获得了引入表在内存中的位置
    add eax, dword ptr \[ebp+DecryptIIDImageBase\]
    ;; 获取最基本的 API 地址
    assume eax : ptr IMAGE\_IMPORT\_DESCRIPTOR
    mov eax, dword ptr \[eax\].FirstThunk
    add eax, dword ptr \[ebp+DecryptIIDImageBase\]
    ;; 从 FirstThunk 中取出 API 的地址
    push dword ptr \[eax\]
    pop dword ptr \[ebp+DecryptIIDLoadLibraryA\]
    push dword ptr \[eax+04h\]
    pop dword ptr \[ebp+DecryptIIDGetProcAddress\]
    
    ;; get kernel32.dll 的句柄
    call strKernel32Dll
        db 'kernel32.dll',0
    strKernel32Dll:
    call dword ptr \[ebp+DecryptIIDLoadLibraryA\]
    mov dword ptr \[ebp+DecryptIIDKernel32Dll\], eax
    call strGlobalAlloc
        db 'GlobalAlloc',0
    strGlobalAlloc:
    push eax
    call dword ptr \[ebp+DecryptIIDGetProcAddress\]
    mov dword ptr \[ebp+DecryptIIDGlobalAlloc\], eax
    call strGetVersion
        db 'GetVersion',0
    strGetVersion:
    push dword ptr \[ebp+DecryptIIDKernel32Dll\]
    call dword ptr \[ebp+DecryptIIDGetProcAddress\]
    mov dword ptr \[ebp+DecryptIIDGetVersion\], eax
    ;; 获取操作系统
    push eax
    call GetSystemVersion
    mov dword ptr \[ebp+DecryptIIDIsNT\], eax
            
    ;; 初始化 引入表
    mov esi, dword ptr \[ebp+DecryptIIDEip\]
    add esi, offset IID\_Private\_Data - offset DecryptIID
    assume esi : ptr IID\_PRIVATE\_DATA
    ;; 重定位 API 地址
    push esi
    mov edi, dword ptr \[ebp+DecryptIIDEip\]
    add edi, offset NEW\_Thunk\_Data - offset DecryptIID
    assume edi : ptr NEW\_THUNK\_DATA
    xor ecx, ecx
    ;; 计算每个 FirstThunk 列表的值
    CountEachOrigFirstThunkNum:
    mov eax, dword ptr \[esi\].FirstThunk
    test eax, eax
    jz EndCountEachOrigFirstThunkNum
    mov edx, eax
    add edx, dword ptr \[ebp+DecryptIIDImageBase\]
        CountFirstThunkListNum:
        mov eax, dword ptr \[edx\]
        test eax, eax
        jz EndCountFirstThunkListNum
        inc ecx
        add edx, 04h
        jmp CountFirstThunkListNum
        EndCountFirstThunkListNum:
    add esi, sizeof IID\_PRIVATE\_DATA
    jmp CountEachOrigFirstThunkNum
    EndCountEachOrigFirstThunkNum:
    ;; 分配内存为新的 API 地址表
    xor edx, edx
    mov eax, sizeof IMPORT\_API\_INSTRUCTION
    imul ecx
    push eax
    push GMEM\_FIXED
    call dword ptr \[ebp+DecryptIIDGlobalAlloc\]
    mov dword ptr \[edi\].NewFirstThunk, eax
    mov dword ptr \[edi\].NextFirstThunk, eax
    assume edi : nothing
    pop esi
    
    ;; 开始处理地址表
HandleFirstThunk:
    mov eax, dword ptr \[esi\].FirstThunk
    test eax, eax
    jz EndHandleFirstThunk
    ;; 加载库
    mov ebx, dword ptr \[esi\].Name1
    add ebx, dword ptr \[ebp+DecryptIIDImageBase\]
    ;; 解密库文件名
    push ebx
    call DeCryptString
    ;; 加载库文件
    push ebx
    call dword ptr \[ebp+DecryptIIDLoadLibraryA\]
    ;; 销毁 DLL 文件名
    push ebx
    call ZeroString
    mov ebx, eax    ; ebx = dll handle
    
    ;; 处理 OriginalFirstThunk 与 FirstThunk
    mov ecx, dword ptr \[esi\].OriginalFirstThunk
    test ecx, ecx
    jnz UseOriginalFirstThunk
    mov ecx, dword ptr \[esi\].FirstThunk
    UseOriginalFirstThunk:
    ;; ecx 指向正确的 ThunkData
    add ecx, dword ptr \[ebp+DecryptIIDImageBase\]
    mov edx, dword ptr \[esi\].FirstThunk
    add edx, dword ptr \[ebp+DecryptIIDImageBase\]
CreateNewApiAddrTbl:
    mov eax, dword ptr \[ecx\]
    test eax, eax
    jz EndCreateNewApiAddrTbl
    ;; 判断是否按序数引出
    test eax, IMAGE\_ORDINAL\_FLAG32
    jnz OnOrdinalImport
    ;; 处理 API 名字表
    add eax, dword ptr \[ebp+DecryptIIDImageBase\]
    add eax, 02h    
    ;; 解密 API 名字
    push eax
    call DeCryptString
    ;; 保存 API 字符串的指针
    push eax
    ;; 获取 API 的地址
    push edx
    push ecx
    push eax
    push ebx            ;  ebx = dll handle
    call dword ptr \[ebp+DecryptIIDGetProcAddress\]
    pop ecx
    pop edx
    ;; 销毁 API 名字
    call ZeroString
    ;; 设置 API 地址
    mov dword ptr \[edx\], eax
    ;; 转到处理下一个 IMAGE\_THUNK\_DATA
    jmp HandleNextThunkData
OnOrdinalImport:
    push edx
    push ecx
    sub eax, IMAGE\_ORDINAL\_FLAG32
    push eax
    push ebx
    call dword ptr \[ebp+DecryptIIDGetProcAddress\]
    pop ecx
    pop edx
    ;; 设置 API 地址
    mov dword ptr \[edx\], eax
HandleNextThunkData:
    ;; 以 DLL 基址判断是否是系统的 DLL，如非系统 DLL 则不用重定位
    test dword ptr \[ebp+DecryptIIDIsNT\], 1
    jz NowIs9x
    cmp ebx, 070000000h
    jb SkipMakeNewThunkTbl
    cmp ebx, 077FFFFFFh
    ja SkipMakeNewThunkTbl
    jmp StartHandleNextThunkData
    NowIs9x:
    cmp ebx, 080000000h
    jb SkipMakeNewThunkTbl
    
    StartHandleNextThunkData:
    push edi
    push esi
    mov edi, dword ptr \[ebp+DecryptIIDEip\]
    add edi, offset NEW\_Thunk\_Data - offset DecryptIID
    assume edi : ptr NEW\_THUNK\_DATA
    mov esi, dword ptr \[edi\].NextFirstThunk
    ;; 指向新的 API 地址表
    mov dword ptr \[edx\], esi
    ;; 相减获取偏移量
    sub eax, esi
    sub eax, sizeof IMPORT\_API\_INSTRUCTION
    assume esi : ptr IMPORT\_API\_INSTRUCTION
    mov byte ptr \[esi\].JmpOpcode, LONG\_JMP\_OPCODE
    mov dword ptr \[esi\].JmpAddr, eax
    ;; 移动到下一个 FirstThunk
    add dword ptr \[edi\].NextFirstThunk, sizeof IMPORT\_API\_INSTRUCTION
    assume esi : nothing
    assume edi : nothing
    pop esi
    pop edi
    ;; 移动到下一个
    SkipMakeNewThunkTbl:
    add ecx, 04h
    add edx, 04h
    jmp CreateNewApiAddrTbl    
    EndCreateNewApiAddrTbl:
    add esi, sizeof IID\_PRIVATE\_DATA
    jmp HandleFirstThunk
    EndHandleFirstThunk:
    
    ;; 设置返回地址并跳入原入口节
    mov eax, dword ptr \[ebp+DecryptIIDEip\]
    add eax, offset DecryptIID\_OrigEntryPoint - offset DecryptIID
    mov eax, dword ptr \[eax\]
    mov esp, ebp
    pop ebp
    jmp eax
    ;; ----- 结束解密 IID -----
    
    ;; 原入口点
    DecryptIID\_OrigEntryPoint       dd  0
    DecryptIID\_ImageBase            dd  0
    ;; 自己的 IID,最多 MAX\_IID\_NUM 个 DLL
    IID\_Private\_Data    db (MAX\_IID\_NUM \* sizeof IID\_PRIVATE\_DATA) dup (0)
    ;; 重定位的 API 地址表结构
    NEW\_Thunk\_Data      NEW\_THUNK\_DATA <0>
            
        ;; ----- ZeroString -----
        ZeroString:
        ZeroStringArg\_String        equ 08h
            push ebp
            mov ebp, esp
            push esi
            push edi
            push eax
            
            ;; 将字符串清 0
            mov edi, dword ptr \[ebp+ZeroStringArg\_String\]
            mov esi, edi
            cld
            ZeroStringLoop:
            lodsb
            test al, al
            jz EndZeroStringLoop
            xor al, al
            stosb
            jmp ZeroStringLoop
            EndZeroStringLoop:
            
            pop eax
            pop edi
            pop esi
            mov esp, ebp
            pop ebp
            retn 4
        
        ;; ----- 解密字符串 -----
        DeCryptString:
        DeCryptStringArg\_String     equ 08h
            push ebp
            mov ebp, esp
            push esi
            push edi
            push eax
            
            ;; 解密字符串
            mov edi, dword ptr \[ebp+DeCryptStringArg\_String\]
            mov esi, edi
            cld
            DeCryptStringLoop:            
            lodsb
            test al, al
            jz EndDeCryptStringLoop
            xor al, ENCRYPT\_STRING\_KEY
            stosb
            jmp DeCryptStringLoop
            EndDeCryptStringLoop:
            
            pop eax
            pop edi
            pop esi
            mov esp, ebp
            pop ebp
            retn 04h
        ;; ----- 判断操作系统 -----
        GetSystemVersion:
        GetSystemVersionArg\_GetVersion      equ 08h
        ;; nt: eax = 1
        ;; 9x: eax = 0

            push ebp
            mov ebp, esp
            push ecx
            push edx
            
            call dword ptr \[ebp+GetSystemVersionArg\_GetVersion\]
            test eax, 080000000h
            jz GetSystemVersion\_IsNT
            xor eax, eax
            jmp ExitGetSystemVersion
            GetSystemVersion\_IsNT:
            xor eax, eax
            inc eax
            ExitGetSystemVersion:
            
            pop edx
            pop ecx
            mov esp, ebp
            pop ebp
            retn 04h            
EndDecryptIID: 

```

最后要注意的就是有时候 FirstThunk 指向的区域是不能写的，所以当我们做完全部工作后要将所有节的属性赋予它可写的权限。  
代码:

```
    ;; 修改所有节为可写属性
    mov esi, dwNTHeaderAddr
    assume esi : ptr IMAGE\_NT\_HEADERS
    mov cx, word ptr \[esi\].FileHeader.NumberOfSections
    movzx ecx, cx
    add esi, sizeof IMAGE\_NT\_HEADERS
    assume esi : ptr IMAGE\_SECTION\_HEADER
    .WHILE ecx != 0
        mov eax, dword ptr \[esi\].Characteristics
        or eax, IMAGE\_SCN\_MEM\_WRITE
        mov dword ptr \[esi\].Characteristics, eax
        dec ecx
        add esi, sizeof IMAGE\_SECTION\_HEADER
    .ENDW

```

IMAGE\_SCN\_MEM\_WRITE 是写属性的常量值。  
  
终于写完了。累。。。![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")  
这节的原理也可以用在 R3 下的 HOOK 方面，大家应该很轻松的想到如何做 R3 下的 HOOK 了，如果有刚刚接触研究病毒的朋友也可以联想到一种模糊入口点的方式。呵呵！  
...才知道看雪论坛一贴只能传两个附件，我将列出引入表的附件穿到自己的沙发了 呵呵。。

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || CryptIID\_bin.rar  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || CryptIID\_src.rar  
 |

列出引入表的附件。。。![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || ListIID\_bin.rar  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || ListIID\_src.rar  
 |
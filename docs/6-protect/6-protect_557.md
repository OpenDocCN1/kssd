

# 【成果 6.4】软件保护壳技术专题 - 加密资源节

> 原文：[`www.pediy.com/kssd/pediy10/68262.html`](https://www.pediy.com/kssd/pediy10/68262.html)

什么是 WINDOWS 的资源这里就不提了。写过 WIN 程序的人应该都晓得了。如果没写过，那么。。。建议去写哈。  
关于加密资源节，也是加壳过程中的一个可选项而已。加和不加都不太对破解造成影响。但是如果  
你的程序中，有一些重要数据在资源节里面而你又不想让其他人通过资源浏览器参看到，那么这个  
时候加密资源节就有必要了。在这里节，我们首先了解一下资源节在 PE 文件中的结构。其次，编写  
一个简单的资源浏览器用于熟悉资源节的结构。最后，完成一个加密与解密资源节的小程序。  
  
资源节可以说在 PE 结构中最复杂的一个结构了。整体是一个 2 叉树结构。  
总共有 3 层。第一层是类型，第二层是名称，第三层是语言,在语言后面就是一个指向资源数据的位置的结构了。  
下面将资源节的结构如图:  
  
资源节结构图:  
附件中查看 呵呵。。。  
  
资源节处于数据目录的第三项可以通过微软预定义的宏 IMAGE\_DIRECTORY\_ENTRY\_RESOURCE 来定位资源  
节的数据目录。通过 VirtualAddress 转换文件偏移 + 文件映射地址就得到了资源节所在文件的位置  
节内容的起始是一个被称为资源目录表的结构，如下：  
IMAGE\_RESOURCE\_DIRECTORY STRUCT  
    Characteristics dd      ?  
    TimeDateStamp dd        ?  
    MajorVersion dw         ?  
    MinorVersion dw         ?  
    NumberOfNamedEntries dw ?  
    NumberOfIdEntries dw    ?  
IMAGE\_RESOURCE\_DIRECTORY ENDS  
Characteristics:资源标志，保留，设置为 0  
TimeDataStamp:时间戳,由资源编译器设定  
MajorVersion:主版本号，由用户设定  
MinorVersion:次版本号，由用户设定  
以下这两项是我们说要关心的  
NumberOfNamedEntries:通过字符串来定义的资源  
NumberOfIdEntries:用过序号来定义的资源  
这两项的和为引入资源的总数，在这个结构后面是一组由 IMAGE\_RESOURCE\_DIRECTORY\_ENTRY 组成的数组  
具体个数为 NumberOfNamedEntries + NumberOfIdEntries  
以下为结构的说明:  
IMAGE\_RESOURCE\_DIRECTORY\_ENTRY STRUCT  
    union  
        rName    RECORD NameIsString:1,NameOffset:31  
        Name1 dd ?  
        Id dw ?  
    ends  
    union  
        OffsetToData dd ?  
          rDirectory    RECORD DataIsDirectory:1,OffsetToDirectory:31  
    ends  
IMAGE\_RESOURCE\_DIRECTORY\_ENTRY ENDS  
这个结构在不同的位置是不同的意义。  
在第一层的为类型，Name1 代表的是一个类型，在资源类型方面有 16 个基本类型  
"RT\_CURSOR"  
"RT\_BITMAP"  
"RT\_ICON"  
"RT\_MENU"  
"RT\_DIALOG"  
"RT\_STRING"  
"RT\_FONTDIR"  
"RT\_FONT"  
"RT\_ACCELERATOR"  
"RT\_RCDATA"  
"DIFFERENCE"  
"RT\_GROUP\_CURSOR"  
"RT\_GROUP\_ICON"  
"RT\_VERSION"  
也有更多的类型，这方面的资料很少。我也没有能收集全，RT\_RCDATA 类型表示为自定义的。  
这里为标准的类型值。  
在第二层时：如果 Name 的 2 进制最高位时候 1,那么它表示一个以名字引出的资源，此时 Name 的低 2 位表示一个从资源节开头到它的偏移  
使用 Name 的低 2 位值 + 资源节开头的地址 指向一个 IMAGE\_RESOURCE\_DIRECTORY\_STRING 结构。资源的名字使用 UNICODE 编码。  
IMAGE\_RESOURCE\_DIRECTORY\_STRING STRUCT  
    Length1 dw      ?  
    NameString db   ?  
IMAGE\_RESOURCE\_DIRECTORY\_STRING ENDS  
Length1 表示有几个字母  
NameString 为以双 0 结尾的 UNICODE 字符串。  
在第三层时,Name 表示资源所用语言的代码。  
  
OffsetToData：这个字段在不同层都表示一个到资源节开头的偏移。  
不同的是，如果 OffsetToData 的 2 进制最高位是 1,表示还有下面的其他层(语言层或者名字层,类型层总是存在)。  
如果是 0,则表示一个到 IMAGE\_RESOURCE\_DATA\_ENTRY 的偏移  
IMAGE\_RESOURCE\_DATA\_ENTRY STRUCT  
    OffsetToData dd ?  
    Size1 dd        ?  
    CodePage dd     ?  
    Reserved dd     ?  
IMAGE\_RESOURCE\_DATA\_ENTRY ENDS  
OffsetTOData 表示到资源数据的偏移，这个偏移是到加载地址(ImageBase)的偏移。  
Size1 为数据的长度  
CodePage:一般为 0  
Reserved:一般为 0  
  
接下来看段代码可能会更清晰一些。  

代码:

```
ResViewEntry proc uses ebx ecx edx edi esi pEntry : LPVOID, dwLevel : DWORD
    LOCAL szOutFileName\[MAX\_PATH\] : BYTE
;; level = 0,1,2

    mov esi, pEntry
    assume esi : ptr IMAGE\_RESOURCE\_DIRECTORY
    ;; 计算有几个资源在当前目录下
    mov cx, word ptr \[esi\].NumberOfNamedEntries
    add cx, word ptr \[esi\].NumberOfIdEntries
    movzx ecx, cx
    ;; 这里存在着 ecx 个 IMAGE\_RESOURCE\_DIRECTORY\_ENTRY
    add esi, sizeof IMAGE\_RESOURCE\_DIRECTORY
    mov edi, esi
    assume edi : ptr IMAGE\_RESOURCE\_DIRECTORY\_ENTRY
ResDirEntryLoop:
    ;; 打印资源层
    invoke JudegeResourceEntryLevel, edi, dwLevel
    mov eax, dword ptr \[edi\].OffsetToData
    test eax, FIRSTBIT
    jz FoundTheDataEntry
    and eax, 0000FFFFh
    add eax, g\_pResSection
    mov edx, dwLevel
    inc edx
    invoke ResViewEntry, eax, edx
    jmp ContinueResDirEntryLoop
    FoundTheDataEntry:
    ;; 取出资源文件
    add eax, g\_pResSection
    assume eax : ptr IMAGE\_RESOURCE\_DATA\_ENTRY
    mov edx, dword ptr \[eax\].Size1
    mov eax, dword ptr \[eax\].OffsetToData
    ;; 转化 RVA 到偏移
    invoke RVA2Offset, g\_pMem, eax
    push eax
    push ecx
    push edx
    invoke wsprintf, addr szOutFileName, offset g\_szOutOrdFormat, eax
    pop edx
    pop ecx
    pop eax
    add eax, g\_pMem
    invoke OutputToFile, addr szOutFileName, eax, edx
    ContinueResDirEntryLoop:
    add edi, sizeof IMAGE\_RESOURCE\_DIRECTORY\_ENTRY      
    dec ecx
    test ecx, ecx
    jnz ResDirEntryLoop
    
    
    assume esi : nothing
    assume edi : nothing
    ret

ResViewEntry endp

JudegeResourceEntryLevel proc uses ebx ecx edx edi esi pResEntry : LPVOID, dwLevel : DWORD
    LOCAL szTmpBuf\[MAX\_PATH\] : BYTE
    
    ;; init local variable
    mov ecx, MAX\_PATH
    lea edi, szTmpBuf
    xor eax, eax
    cld
    rep stosb
    
    mov edi, pResEntry
    assume edi : ptr IMAGE\_RESOURCE\_DIRECTORY\_ENTRY
    mov eax, dwLevel
       
    cmp eax, 0
    jnz JudgeLevelOne
    ;; 显示类型
    mov eax, dword ptr \[edi\].Name1
    invoke ShowResourceType, eax
    jmp ExitJudgeLevel
JudgeLevelOne:
    cmp eax, 1
    jnz JudgeLevelTwo
    ;; 显示名称
    mov eax, dword ptr \[edi\].Name1
    ;; 最高位是 0 时表示使用 ID
    test eax, FIRSTBIT
    jz UseID
    and eax, 0000FFFFh
    add eax, g\_pResSection
    assume eax : ptr IMAGE\_RESOURCE\_DIR\_STRING\_U
    lea esi, \[eax\].NameString
    lea edi, szTmpBuf
    ;; 拷贝字符串到局部变量
    mov cx, word ptr \[eax\].Length1
    movzx ecx, cx
    cld
    rep movsw
    invoke crt\_printf, offset g\_szOutFormat, offset g\_szResNameByStr
    invoke crt\_wprintf, addr szTmpBuf
    invoke crt\_printf, offset g\_szOutFormat, offset g\_szEndLine
    jmp ExitJudgeLevel                
    UseID:
    invoke crt\_printf, offset g\_szResNameById, eax   
    jmp ExitJudgeLevel
JudgeLevelTwo:
    ;; 第三次是语言,这里输出语言定义的常量
    mov eax, dword ptr \[edi\].Name1
    invoke crt\_printf, offset g\_szResLang, eax
ExitJudgeLevel:
    assume eax : nothing
    assume edi : nothing
    ret

JudegeResourceEntryLevel endp

```

很简单对吧!。。。 这段在附件中的程序打印资源的类型，名称/ID，语言代码，还有将资源 dump 到一个目录中。  
接下来就改讲解一下加解密资源节了，其实与列出资源节的算法一样。这里使用的加密算法也很简单只是简单的 XOR 了一哈。  
不过要注意的是。有些资源是不可以加密的。例如 ICON 还有寻找 ICON 偏移的 ICON\_GROUP 还有版本信息等，还有其他的，总之在 PE 加载器在代码段之前加载的资源是不可以加密，否则程序不能运行。这里的加密程序不加密 ICON ICON\_GROUP VERSION 这三种类型，如果你遇到加密资源后程序无法运行那可以就是加了不该加的资源了。呵呵，自行修改下代码吧。加密在 JudegeResourceType 处添加过滤，解密在 JudgeDecryptResType 处修改，代码很简单一看就明白  
下面列出加解密代码  
首先是加密的  
代码:

```
EncryptResSec proc uses ebx ecx edx edi esi pEntry : LPVOID, dwLevel : DWORD
;; level = 0,1,2

    mov esi, pEntry
    assume esi : ptr IMAGE\_RESOURCE\_DIRECTORY
    ;; 计算有几个资源在当前目录下
    mov cx, word ptr \[esi\].NumberOfNamedEntries
    add cx, word ptr \[esi\].NumberOfIdEntries
    movzx ecx, cx
    ;; 这里存在着 ecx 个 IMAGE\_RESOURCE\_DIRECTORY\_ENTRY
    add esi, sizeof IMAGE\_RESOURCE\_DIRECTORY
    mov edi, esi
    assume edi : ptr IMAGE\_RESOURCE\_DIRECTORY\_ENTRY
ResDirEntryLoop:
    ;; 加密资源层
    ;; 清除加密标志,如果是 0 级别则清除
    cmp dwLevel, 0
    jnz SkipClearEncryptFlag
    push 0
    pop g\_dwEncrypt
    SkipClearEncryptFlag:
    invoke JudegeResourceType, edi, dwLevel
    mov eax, dword ptr \[edi\].OffsetToData
    test eax, FIRSTBIT
    jz FoundTheDataEntry
    and eax, 0000FFFFh
    add eax, g\_pResSection
    mov edx, dwLevel
    inc edx
    invoke EncryptResSec, eax, edx
    jmp ContinueResDirEntryLoop
    FoundTheDataEntry:
    ;; 加密资源文件
    mov ebx, g\_dwEncrypt
    test ebx, ebx
    jz ContinueResDirEntryLoop
    
    add eax, g\_pResSection
    assume eax : ptr IMAGE\_RESOURCE\_DATA\_ENTRY
    mov edx, dword ptr \[eax\].Size1
    mov eax, dword ptr \[eax\].OffsetToData
    ;; 转化 RVA 到偏移
    invoke RVA2Offset, g\_pMem, eax
    add eax, g\_pMem
    
    ;; 加密资源
    EncryptResourceData:
    push edi
    push esi
    push ecx
    mov edi, eax
    mov esi, edi
    mov ecx, edx    
    cld
    EnResourceLoop:
    lodsb
    xor al, CRYPT\_KEY
    stosb
    loop EnResourceLoop    
    pop ecx
    pop esi
    pop edi
    ContinueResDirEntryLoop:
    add edi, sizeof IMAGE\_RESOURCE\_DIRECTORY\_ENTRY
    dec ecx
    test ecx, ecx
    jnz ResDirEntryLoop
    
    
    assume esi : nothing
    assume edi : nothing
    ret    
EncryptResSec endp

JudegeResourceType proc uses eax edi pResEntry : LPVOID, dwLevel : DWORD
    
    ;; 不加密 ICON,ICON\_GROUP,还有版本信息
    ;; 在 PE 加载器加载时要读取这些资源
    ;; 否则程序会崩溃    
    
    mov edi, pResEntry
    assume edi : ptr IMAGE\_RESOURCE\_DIRECTORY\_ENTRY
    mov eax, dwLevel      
    ;; 不是类型层,直接退出 
    cmp eax, 0
    jnz ExitJudgeLevel
    ;; 显示类型
    mov eax, dword ptr \[edi\].Name1
    cmp eax, RT\_ICON
    jnz CmpICON\_GROUP
    jmp ExitJudgeLevel
CmpICON\_GROUP:
    cmp eax, RT\_GROUP\_ICON
    jnz CmpVERSION 
    jmp ExitJudgeLevel
CmpVERSION:
    cmp eax, RT\_VERSION
    jnz EncryptData
         
ExitJudgeLevel:
    assume eax : nothing
    assume edi : nothing
    ret

EncryptData:
    push 1
    pop g\_dwEncrypt
    jmp ExitJudgeLevel

JudegeResourceType endp

```

解密代码如下:  
代码:

```
CryptResource proc pFileName : LPSTR
    LOCAL hFile : HANDLE
    LOCAL hMap : HANDLE
    LOCAL pMem : LPVOID
    LOCAL dwAppendSize : DWORD    
    LOCAL dwNTHeaderAddr : DWORD
    LOCAL dwImageBase : DWORD
    LOCAL dwOrigEntryAddr : DWORD
    LOCAL dwResSecVA : DWORD
    
    ;; init data
    xor eax, eax
    mov g\_bError, al
    mov eax, offset EndDecryptResource - offset DecryptResource
    mov g\_dwDecryptResourceSize, eax
    
    ;; get the append size
    invoke CountAppendSize, pFileName
    .IF eax == 0
        jmp GetFileSizeFailed
    .ENDIF
    mov dwAppendSize, eax
    
    ;; open file
    invoke CreateFile, pFileName,\\
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
;    ;; get file size
;    invoke GetFileSize, eax, NULL
;    .IF eax == 0
;        jmp GetFileSizeFailed        
;    .ENDIF
    mov eax, dwAppendSize
    xchg eax, ecx
    ;; create memory map
    xor ebx, ebx
    invoke CreateFileMapping, hFile, ebx, PAGE\_READWRITE, ebx, ecx, ebx
    .IF eax == 0
        invoke CloseHandle, hFile
        jmp CreateMapFailed                
    .ENDIF
    mov hMap, eax
    ;; map file to memory
    xor ebx, ebx
    invoke MapViewOfFile, hMap, 
                          FILE\_MAP\_WRITE + FILE\_MAP\_READ + FILE\_MAP\_COPY,
                          ebx, ebx, ebx
    .IF eax == 0
        invoke CloseHandle, hMap
        invoke CloseHandle, hFile
        jmp MapFileFailed        
    .ENDIF                          
    mov pMem, eax       
    mov g\_pMem, eax                        
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
    
    ;; store basic PE orig information
    mov dwNTHeaderAddr, esi
    assume esi : ptr IMAGE\_NT\_HEADERS
    mov eax, dword ptr \[esi\].OptionalHeader.ImageBase
    mov dwImageBase, eax
    mov eax, dword ptr \[esi\].OptionalHeader.AddressOfEntryPoint
    add eax, dwImageBase
    mov dwOrigEntryAddr, eax
    
    ;; 判断是否存在资源节
    mov eax, dword ptr \[esi\].OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_RESOURCE \* sizeof IMAGE\_DATA\_DIRECTORY\].VirtualAddress
    .IF eax == 0
        jmp NoResourceSec        
    .ENDIF
    mov ebx, eax
    add ebx, dwImageBase
    mov dwResSecVA, ebx     ; ebx 为资源节在内存中的地址
    ;; 转变为 FVA
    invoke RVA2Offset, pMem, eax
    add eax, pMem
    mov g\_pResSection, eax
    
    ;; 加密资源节
    invoke EncryptResSec, eax, 0
          
    ;; 升级数据
    lea eax, DecryptRes\_dwOrigEntryAddress
    push dwOrigEntryAddr
    pop dword ptr \[eax\]
    
    lea eax, DecryptRes\_ResVA
    push dwResSecVA
    pop dword ptr \[eax\]
    
    lea eax, DecryptRes\_ImageBase
    push dwImageBase
    pop dword ptr \[eax\]
    
    ;; 添加解密节
    invoke AddSection, pMem, NULL, g\_dwDecryptResourceSize
    mov esi, eax
    assume esi : ptr IMAGE\_SECTION\_HEADER
    
    ;; 更新入口点与 SizeOfImage
    mov edi, dwNTHeaderAddr
    assume edi : ptr IMAGE\_NT\_HEADERS
    
    ;; 更改资源节属性
    mov eax, dword ptr \[edi\].OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_RESOURCE \* sizeof IMAGE\_DATA\_DIRECTORY\].VirtualAddress
    invoke FindSectionTable, pMem, eax
    assume eax : ptr IMAGE\_SECTION\_HEADER
    push 0E00000E0h ; 可读可写可执行
    pop dword ptr \[eax\].Characteristics  
    
    push dword ptr \[esi\].VirtualAddress
    pop dword ptr \[edi\].OptionalHeader.AddressOfEntryPoint
    
    mov eax, dword ptr \[esi\].VirtualAddress
    add eax, dword ptr \[esi\].Misc.VirtualSize
    mov ebx, dword ptr \[edi\].OptionalHeader.SectionAlignment
    invoke PEAlign, eax, ebx
    
    push eax
    pop dword ptr \[edi\].OptionalHeader.SizeOfImage
    
    ;; 将解密节写入
    mov edi, dword ptr \[esi\].PointerToRawData
    add edi, pMem
    lea esi, DecryptResource
    mov ecx, g\_dwDecryptResourceSize
    cld
    rep movsb
                
LogicShellExit:
    ;; close handle & write it
    invoke UnmapViewOfFile, pMem
    invoke CloseHandle, hMap
    invoke CloseHandle, hFile
    .IF g\_bError == 0
        ;; show success message
        invoke crt\_printf, offset g\_szOutFormat, offset g\_szDone
    .ENDIF
    assume eax : nothing
    assume esi : nothing
    assume edi : nothing
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
NoResourceSec:
    lea eax, g\_szNoResTbl
    jmp ShowErr    
ShowErr:
    invoke crt\_printf, offset g\_szOutFormat, eax
    mov al, 1
    mov g\_bError, al
    jmp LogicShellExit
    
;; ----- 解密资源节 -----
DecryptResource:
DecryptResourceStackSize    equ 10h
DecryptResource\_Eip         equ -04h
DecruptResource\_dwFlags     equ -08h
    push ebp
    mov ebp, esp
    sub esp, DecryptResourceStackSize
    ;; 清 0 堆栈
    mov edi, esp
    mov ecx, DecryptResourceStackSize
    cld
    xor eax, eax
    rep stosb
    call GetEip
    GetEip:
    pop eax
    sub eax, offset GetEip - offset DecryptResource
    mov dword ptr \[ebp+DecryptResource\_Eip\], eax    
    add eax, offset DecryptRes\_ResVA - offset DecryptResource
    push 0                  ; Level
    push dword ptr \[eax\]    ; ResVA
    push dword ptr \[eax\]    ; pEntry
    call RecDecryptRes
    
    
    
    ;; 读取原入口点
    mov eax, offset DecryptRes\_dwOrigEntryAddress - offset DecryptResource
    add eax, dword ptr \[ebp+DecryptResource\_Eip\]
    mov eax, dword ptr \[eax\]
    mov esp, ebp
    pop ebp
    jmp eax
    ;; ----- 解密资源节数据 -----
    DecryptRes\_dwOrigEntryAddress           dd  0
    DecryptRes\_ResVA                        dd  0
    DecryptRes\_bDecrypt                     dd  0
    DecryptRes\_ImageBase                    dd  0
    ;; ----- 递归解密资源节的函数 -----
    
    RecDecryptRes:
    RecDecryptResArg\_Entry              equ 08h
    RecDecryptResArg\_ResVA              equ 0ch
    ResDecryptRes\_dwLevel               equ 10h
    
    ResDecryptResStackSize              equ 04h
    ResDecryptRes\_Eip                   equ -04h
    
        push ebp
        mov ebp, esp
        sub esp, ResDecryptResStackSize
        
        push ebx
        push ecx
        push edx
        push edi
        push esi
        
        ;; 获取 eip
        call GetRecDecryptResEip
        GetRecDecryptResEip:
        pop ebx
        sub ebx, offset GetRecDecryptResEip - offset RecDecryptRes
        mov dword ptr \[ebp+ResDecryptRes\_Eip\], ebx
                
        mov esi, dword ptr \[ebp+RecDecryptResArg\_Entry\]
        assume esi : ptr IMAGE\_RESOURCE\_DIRECTORY
        ;; 计算有几个资源在当前目录下
        mov cx, word ptr \[esi\].NumberOfNamedEntries
        add cx, word ptr \[esi\].NumberOfIdEntries
        movzx ecx, cx
        ;; 这里存在着 ecx 个 IMAGE\_RESOURCE\_DIRECTORY\_ENTRY
        add esi, sizeof IMAGE\_RESOURCE\_DIRECTORY
        mov edi, esi
        assume edi : ptr IMAGE\_RESOURCE\_DIRECTORY\_ENTRY
    ResDeDirEntryLoop:
        ;; 解密资源层
        ;; 是 0 级别则清除
        cmp dword ptr \[ebp+ResDecryptRes\_dwLevel\], 0
        jnz SkipClearDecryptFlag
        push 0
        mov eax, dword ptr \[ebp+ResDecryptRes\_Eip\]
        sub eax, offset RecDecryptRes - offset DecryptRes\_bDecrypt
        pop dword ptr \[eax\]
        SkipClearDecryptFlag:
        push dword ptr \[ebp+ResDecryptRes\_dwLevel\]
        push edi
        call JudgeDecryptResType
        mov eax, dword ptr \[edi\].OffsetToData
        test eax, FIRSTBIT
        jz FoundTheDeDataEntry
        and eax, 0000FFFFh
        add eax, dword ptr \[ebp+RecDecryptResArg\_ResVA\]
        mov edx, dword ptr \[ebp+ResDecryptRes\_dwLevel\]
        inc edx
        push edx
        push dword ptr \[ebp+RecDecryptResArg\_ResVA\]
        push eax
        call RecDecryptRes
        jmp ContinueResDeDirEntryLoop
        FoundTheDeDataEntry:
        ;; 取出资源文件
        mov ebx, dword ptr \[ebp+ResDecryptRes\_Eip\]
        sub ebx, offset RecDecryptRes - offset DecryptRes\_bDecrypt
        mov ebx, dword ptr \[ebx\]
        test ebx, ebx
        jz ContinueResDeDirEntryLoop
        
        ;; ebx 不为 0 则解密
        add eax, dword ptr \[ebp+RecDecryptResArg\_ResVA\]
        assume eax : ptr IMAGE\_RESOURCE\_DATA\_ENTRY
        mov edx, dword ptr \[eax\].Size1
        mov eax, dword ptr \[eax\].OffsetToData
        mov ebx, dword ptr \[ebp+ResDecryptRes\_Eip\]
        sub ebx, offset RecDecryptRes - offset DecryptRes\_ImageBase
        mov ebx, dword ptr \[ebx\]
        add eax, ebx
        
        ;; 解密资源
        push edi
        push esi
        push ecx
        mov edi, eax
        mov esi, edi
        mov ecx, edx
        cld
        DeResourceLoop:
        lodsb
        xor al, CRYPT\_KEY
        stosb
        loop DeResourceLoop
        
        mov esi, dword ptr \[ebp+ResDecryptRes\_Eip\]
        sub esi, offset RecDecryptRes - offset DecryptRes\_bDecrypt
        push 0
        pop dword ptr \[esi\]
        
        pop ecx
        pop esi
        pop edi
        ContinueResDeDirEntryLoop:
        add edi, sizeof IMAGE\_RESOURCE\_DIRECTORY\_ENTRY      
        dec ecx
        test ecx, ecx
        jnz ResDeDirEntryLoop         
         
        assume esi : nothing
        assume edi : nothing        
        
        pop esi
        pop edi
        pop edx
        pop ecx
        pop ebx       
        
        mov esp, ebp
        pop ebp
        retn 0ch
                  
    ;; ----- 判断资源类型 -----
    JudgeDecryptResType:
    JudgeDecryptResArg\_Entry            equ 08h
    JudgeDecryptResArg\_dwLevel          equ 0ch
        
        push ebp
        mov ebp, esp 
      
        push eax
        push edi
        push ebx
        push esi
        
        ;; 获取 EIP
        call GetJudgeDecryptEip
        GetJudgeDecryptEip:
        pop esi
        sub esi, offset GetJudgeDecryptEip - offset JudgeDecryptResType
        ;; esi 为函数头地址     
      
        ;; 这里不加密第一个 ICON 与第一个 ICON GROUP
        ;; 在 PE 加载器加载时要读取这些资源
        ;; 否则程序会崩溃
        
        ;; 清除加密标志
        ;xor eax, eax
        mov ebx, esi
        sub ebx, offset JudgeDecryptResType - offset DecryptRes\_bDecrypt
        ;mov dword ptr \[ebx\], eax
        
        
        ;; ebx 为解密标志
        ;; edx 为解密标志的地址
        ;; 用两个寄存器来代替变量
        
        mov edi, dword ptr \[ebp+JudgeDecryptResArg\_Entry\]
        assume edi : ptr IMAGE\_RESOURCE\_DIRECTORY\_ENTRY
        mov eax, dword ptr \[ebp+JudgeDecryptResArg\_dwLevel\]
        cmp eax, 0
        jnz ExitJudgeLevel
        ;; 显示类型
        mov eax, dword ptr \[edi\].Name1
        cmp eax, RT\_ICON
        jnz CmpICON\_GROUP     
        jmp ExitJudgeLevel
    CmpICON\_GROUP:
        cmp eax, RT\_GROUP\_ICON
        jnz CmpVERSION      
        jmp ExitJudgeLevel
    CmpVERSION:
        cmp eax, RT\_VERSION
        jnz DecryptData
        
    ExitJudgeLevel:
        assume eax : nothing
        assume edi : nothing     

        
        pop esi
        pop ebx
        pop edi
        pop eax
             
        mov esp, ebp
        pop ebp
        retn 08h    
        
    DecryptData:        
        push 1
        pop dword ptr \[ebx\]
        jmp ExitJudgeLevel           
EndDecryptResource:
    
CryptResource endp

```

个人觉的加密资源节在软件保护方面没有什么太大的意义。对于加密资源节最大的意义应该是对于壳加密木马的时候，有些木马将 rootkit 以资源的形式存放，杀毒软件会遍历资源节来寻找特征码。如果此时加密了这个。在解密时恢复，那么就可以通过杀毒软件的查杀。  
  
  
![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")...   
那个资源节结构图是用 EDraw 画的，原文件也一并上传了。画的难看了点，见谅。。。

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || ResViewer\_src.rar  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || ResViewer\_bin.rar  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || CryptRes\_src.rar  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || CryptRes\_bin.rar  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 资源节结构图.rar  
 |
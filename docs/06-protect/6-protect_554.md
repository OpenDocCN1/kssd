
# 【成果 6.1】软件保护壳技术专题 - 添加新节

> 原文：[`www.pediy.com/kssd/pediy10/66612.html`](https://www.pediy.com/kssd/pediy10/66612.html)

壳要附加到软件本身，有很多方式进行。在这里使用最常用的一种方式，添加一个新节。这里我们先用文字的形式描述一下添加新节的算法。然后给出一段代码并在注释中详细给出每条指令的解释。在这之前，首先给出一些名词的解释，以便刚接触的朋友可以熟悉。熟悉的朋友可以直接跳过。

名词解释：

*   `Offset`
    相对偏移。常指在文件中到文件头的距离。

*   `RVA`
    相当内存偏移。常指在内存中到内存加载起始地址的距离。

*   `VA`
    虚拟地址。常指在内存中确切的地址也就是 `RVA + 内存加载起始地址`。

*   `Alignment`
    对齐粒度。当作"最小单位"就可以了例如：对齐粒度为 `200h`。可以这样理解，一辆装货的车上面装的全是箱子，这箱子的大小是 `200h`。如果最后一箱子没有装满，但是它仍然占着一个箱子。

添加新节相关的 `PE` 头属性：

位于 `IMAGE_NT_HEADERS` 结构中的属性：

*   `ImageBase` (4 字节)
*   `SizeOfImage` (4 字节)
*   `NumberOfSections` (2 字节)
*   `AddressOfEntryPoint` (4 字节)
*   `SectionAlignment` (4 字节)
*   `FileAlignment` (4 字节)

位于 `IMAGE_SECTION_HEADER` 结构的属性：

*   最后节表 `VirtualSize` (4 字节)
*   最后节表的 `VirtualAddress` (4 字节)
*   最后节表的 `SizeOfRawData` (4 字节)
*   最后节表的 `PointerToRawData` (4 字节)
*   最后节表的 `Characteristics` (4 字节)

添加新节算法描述：

1.  建立文件映射
2.  判断是否是 `PE` 文件
3.  移动到最后一个节表
4.  添加新节节表
5.  设置新节的 `VirtualAddress`, `VirtualSize`, `SizeOfRawData`, `PointerToRawData`, `Characteristics` 等属性
6.  将新节的内容写入文件
7.  增加 `NumberOfSections` 属性
8.  设置 `SizeOfImage`, `AddressOfEntryPoint` 属性
9.  将内存映射回文件

注：代码中讲解的部分用红色标出

首先是建立文件映射，也可以直接读写文件，不过这样做操作起来会方便一些。

代码：

```
CryptFile proc szFname : LPSTR
    LOCAL hFile : HANDLE
    LOCAL hMap : HANDLE
    LOCAL pMem : LPVOID
    LOCAL dwOrigFileSize : DWORD
    LOCAL dwNTHeaderAddr : DWORD
    
    ;; init data
    xor eax, eax
    mov g_bError, al
    mov eax, offset EndNewSection - offset NewSection
    mov g_dwNewSectionSize, eax
    
    ;; open file
    invoke CreateFile, szFname,\
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
    
    ;; 这里的 dwOrigFileSize 是原始的文件大小
    ;; 因为你要添加新节，所以要多上那么一点
    ;; 点尺寸.这个尺寸就是 APPEND_SIZE,如果
    ;; 这个尺寸也许会让你最后添加的程序后有
    ;; 一些多余的数据。也可以没有，不过有一 
    ;; 点点麻烦。这就要计算添加后的
    ;; SizeOfImage 了。等到以后讲解吧。
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

    ;; 在建立映射后
    ;; 这段代码将映射后文件的指针存放在局部变量 pMem.
    ;; 而指定的 NT 结构头指针存放到 esi 寄存器处。
    ;; 现在我们调用添加节函数添加一个新节,AddSection 是
    ;; 我们这节的主要函数，将在下面讲解。
    ;; 增加一个新节
    ;; pMem:文件映射内存指针
    ;; g_szNewSectionName:新节的节名,这里是(.new)
    ;; g_dwNewSectionSize:新节的长度,这里是 offset EndNewSection - offset NewSection
    invoke AddSection, pMem, offset g_szNewSectionName, g_dwNewSectionSize
    push eax
    mov esi, dwNTHeaderAddr
    assume esi : ptr IMAGE_NT_HEADERS
    ;; 下面做的就是设置新节的中的原代码入口点,就是真正的入口地址.
    mov ebx, dword ptr [esi].OptionalHeader.AddressOfEntryPoint
    add ebx, dword ptr [esi].OptionalHeader.ImageBase
    ;; OrigAddressOfEntry 这个变量在 CryptFile 底部的 NewSection 节中
    mov eax, offset OrigAddressOfEntry
    mov dword ptr [eax], ebx
    ;; 更新入口点,将新节的入口点设置到 NT 头结构的 AddressOfEntryPoint
    ;; 哪个节的 VirusAddress 被设置到 AddressOfEntryPoint 处,哪个节将会被先执行
    ;; 这也是最通常的入口点技术
    pop eax
    assume eax : ptr IMAGE_SECTION_HEADER
    push dword ptr [eax].VirtualAddress
    pop dword ptr [esi].OptionalHeader.AddressOfEntryPoint
    ;; 下面的代码利用新节节表将新节的代码写入文件
    mov esi, offset NewSection
    mov edi, dword ptr [eax].PointerToRawData
    add edi, pMem
    mov ecx, g_dwNewSectionSize
    cld
    rep movsb
LogicShellExit:
    ;; close handle & write it
    invoke UnmapViewOfFile, pMem
    invoke CloseHandle, hMap
    invoke CloseHandle, hFile
    .IF g_bError == 0
        ;; show success message
        invoke MessageBox, NULL, offset g_szDone, offset g_szDoneCap, MB_ICONINFORMATION
    .ENDIF
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
;; ----- 新节代码 -----
NewSection:
   ;; 在这里获取地址
   ;; call 指令会将下条指令的地址压入堆栈
   ;; 注意此指令的 OPCODE 为 EB00000000
   ;; 病毒与 Shellcode 等常用此指令定位
   ;; 杀毒软件的启发式搜索常将此特征作为查找特征
   ;; 聪明的读者可以自己修改定位代码来躲过
   ;; 这类的查杀
   call GetEip
   GetEip:
   ;; eax 中有保存着当前的地址，标号为 GetEip
   pop eax
   add eax, offset OrigAddressOfEntry - offset GetEip
   ;; 两个偏移的差就是这两个地址之间的距离,它的距离 + 起始地址
   ;; 就为 OrigAddressOfEntry 的地址
   ;; 最后将 OrigAddressOfEntry 保存的值,也就是原来的入口节的地址
   ;; 送回 eax 中。
   mov eax, dword ptr [eax]
   ;; 跳到原入口点地址
   jmp eax
;; ----- 新节数据 -----
OrigAddressOfEntry  dd ?

EndNewSection:

CryptFile endp
```

上面的新节代码没有任何作用，只是直接跳入到原入口的地址。

下面的代码就是这次主题的主要的代码。

代码：



```assembly
AddSection proc uses ebx ecx edx esi edi, pMem : LPVOID,
                      pSectionName : LPVOID,
                      dwSectionSize : DWORD
;; add a new section
;; ret: eax =  new section table file offset
    LOCAL dwNTHeader : LPVOID
    LOCAL dwLastSecTbl : LPVOID     
    LOCAL dwFileAlig : DWORD
    LOCAL dwSecAlig : DWORD
                                                                    
    ;; move to section table
    mov esi, pMem
    ;; 将映射地址传给 esi,再将 esi 设置为 PE 头的地址
    ;; 这里这个 3ch 是 DOS 头中 e_lfanew 的偏移。e_lfanew 保存着
    ;; 从 MZ 头到 PE 头的偏移,所以这样的相加使 esi 指向 PE 头
    ;; 也可以使用下面同等指令替换
    ;; assume esi : ptr IMAGE_DOS_HEADER
    ;; add esi, dword ptr [esi].e_lfanew
    add esi, dword ptr [esi+3ch]
    mov dwNTHeader, esi   
    assume esi : ptr IMAGE_NT_HEADERS
    ;; update the number of section
    mov cx, word ptr [esi].FileHeader.NumberOfSections
    movzx ecx, cx
    ;; 增加节的数目
    inc word ptr [esi].FileHeader.NumberOfSections
    push dword ptr [esi].OptionalHeader.FileAlignment
    pop dwFileAlig
    push dword ptr [esi].OptionalHeader.SectionAlignment
    pop dwSecAlig           
    ;; move esi point to section table
    ;; 在 NT 头结构下面跟着的就是 N 个节表街头.加上 NT 头结构的长度就为第一个节表
    add esi, sizeof IMAGE_NT_HEADERS
    ;; store the last section table
    ;; 在这里保存最后一个节表的偏移,因为一会在计算新节的 RVA 和 offset 要应用。
    mov eax, sizeof IMAGE_SECTION_HEADER
    mov ebx, ecx
    imul ebx
    ;; esi 为新节节表的偏移
    ;; 到这里可以判断下 SizeOfHeader 大小以便检测是否有空间加入新节,这里没有做判断
    ;; 一般没有加过壳的程序都会有空间加入,这个也是按照 File Alignment 对齐的所以就会有
    ;; 剩余的空隙加入
    add esi, eax                         ; esi = the end of orig last section fva
    push esi
    ;; 这里保存了原最后节表的偏移，为了设置新节的地址做准备
    sub esi, sizeof IMAGE_SECTION_HEADER     ; esi = the orig last section fva
    mov dwLastSecTbl, esi
    pop esi
    ;; set new section table
    ;; 设置新节节表名
    assume esi : ptr IMAGE_SECTION_HEADER
    ;; set section name
    push esi
    lea edi, [esi].Name1
    mov esi, pSectionName
CopySectionNameLoop:    
    lodsb
    test al, al
    jz EndCopySectionNameLoop
    stosb
    jmp CopySectionNameLoop
EndCopySectionNameLoop:   
    pop esi   
    ;; set section characteristics
    ;; 设置设置节的属性,一般关心的有三个属性,读写执行
    ;; 0E00000E0h 为设置可读可写可执行三个属性的或运算的值
    ;; 有些杀毒软件的启发式搜索也会检查入口点节是否存在写权限
    ;; 来判断是否被病毒感染,有些多态病毒需要自解密自身所有需要
    ;; 写权限,在壳被恶意程序利用后，现在的杀毒软件有可能报告为
    ;; Heru.XXX 或者 XCrypt 之类，都是启发式惹的祸,过此类检测也是有
    ;; 方法的,例如启动后将的代码写到栈中执行.现在的病毒做的越来越像壳
    ;; 壳整的越来越像病毒,两者只有目的上的区别了。
    push 0E00000E0h
    pop dword ptr [esi].Characteristics
    ;; set section virtualsize
    ;; 这里设置节的真实大小,VirtualSize 表示新节的真实大小,没有经过对齐后的
    push dwSectionSize
    pop dword ptr [esi].Misc.VirtualSize
    ;; set section sizeofrawdata
    ;; 节的 SizeOfRawData 为此节在文件中经过文件对齐后的大小
    ;; PEAlign 函数用于计算节对齐后的大小,将在附件的代码中给出
    invoke PEAlign, dwSectionSize, dwFileAlig
    mov dword ptr [esi].SizeOfRawData, eax
    ;; set section virtualaddress
    ;; 设置新节的内存偏移和文件偏移需要上一节的一个信息
    ;; 新节的内存偏移 = 上一节的内存偏移 + 上一节经过节对齐后的长度
    ;; 新节的文件偏移 = 上一节的文件偏移 + 上一节进过文件对齐后的长度
    mov eax, dwLastSecTbl                   ; eax = orig last section table fva
    assume eax : ptr IMAGE_SECTION_HEADER
    mov ecx, dword ptr [eax].VirtualAddress
    add ecx, dword ptr [eax].Misc.VirtualSize        ; ecx = new section rva
    mov ebx, dword ptr [eax].PointerToRawData
    add ebx, dword ptr [eax].SizeOfRawData           ; ebx = new section fva
    invoke PEAlign, ecx, dwSecAlig
    mov dword ptr [esi].VirtualAddress, eax
    ;; set section pointertorawdata
    invoke PEAlign, ebx, dwFileAlig
    mov dword ptr [esi].PointerToRawData, eax
    ;; update the sizeofimage
    ;; SizeofImage 表示从文件到内存映射文件的内容通过节对齐的大小
    ;; 这个值等于当前最后一节的内存偏移 + 当前最后一节的经过节对齐的大小
    ;; 大家可以思考一下。这个值很有用，可以利用此值做一些特殊的感染来躲过启发式
    ;; 搜索。呵呵...
    mov eax, dword ptr [esi].VirtualAddress
    add eax, dword ptr [esi].Misc.VirtualSize
    invoke PEAlign, eax, dwSecAlig
    mov edx, dwNTHeader
    assume edx : ptr IMAGE_NT_HEADERS
    mov dword ptr [edx].OptionalHeader.SizeOfImage, eax
    push dword ptr [esi].PointerToRawData
    pop edi
    add edi, pMem
    ;; clear the new sec
    ;; 在这里做一下清 0 工作 ZeroMemory
    mov ecx, dwSectionSize
    xor eax, eax
    cld
    rep stosb
                                                                
    ;; 此函数的返回值,新节节表的文件偏移
    mov eax, esi
    assume esi : nothing
    assume eax : nothing
    assume edx : nothing
    ret
AddSection endp
```

添加新节为最基本的感染方式，感染这个词用在这里虽然有些不太恰当，但是我实在想不到用什么更形象词来描述此等行为。感染算法在病毒领域有很多的讨论，感染技术很多配合入口点技术来一起讨论。如果真要详细讨论此技术怕是要在开一个专题讨论才够。  

在附件中的代码由于要修改代码节自身,所以添加新节的程序在连接时要用到 `/SECTION:.text,ERW` 的选项.如果在 `RadAsm` 中配置连接选项时要改为 `/SECTION:.text|ERW` 才可以正常连接。  

附件程序的使用为在 `AddSection` 同目录下放置一名为 `target.exe` 程序直接运行 `AddSection` 后即可.可使用 `ollydbg` 调试 `target.exe` 查看.  

平常很少发帖，在专题文章的质量和可读性上希望大家多多提要求。。。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `AddSection_src.rar`  
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `AddSection_bin.rar`  
|

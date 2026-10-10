

# 【成果 6.3】软件保护壳专题 - 处理 TLS 表

> 原文：[`www.pediy.com/kssd/pediy10/67627.html`](https://www.pediy.com/kssd/pediy10/67627.html)

一开始想这节的名字叫做加密 TLS 表，后来感觉 TLS 表的加密实在是有些不符合。主要是出于 TLS 节加密的必要性考虑。所以改成处理 TLS 表了。我自身喜欢把 TLS 利用成模糊入口点来利用。此节规划是这样的，我们先理解一下什么是 TLS 以及它的结构。接下来汇编一个 TLS 节及相关在 MASM 中的应用。最后，我们讲解如何修改现有的 TLS 以及制作 TLS 入口点。  
  
什么是 TLS 表  
这个问题最有力的参考莫过于 MSDN 的文档了。这种源的东西比起用我的语言归纳要清楚的多。  
MSDM 文档的连接为  
ms-help://MS.MSDNQTR.v80.chs/MS.MSDN.v80/MS.VisualStudio.v80.chs/dv\_vclang/html/a0f1b109-c953-4079-aa10-e47f5483173d.htm  
引用自 MSDN  

引用: 线程本地存储 (TLS) 是一个方法，通过该方法，给定的多线程进程中的每个线程都可以分配存储线程特定数据的位置。动态绑定（运行时）线程特定数据是通过 TLS API（TlsAlloc、TlsGetValue、TlsSetValue 和 TlsFree）的方式支持的。除了现有的 API 实现，Win32 和 Visual C++ 编译器现在还支持静态绑定（加载时间）基于线程的数据。  
  
TLS 的 API 实现  
线程本地存储是通过 Win32 API 层和编译器实现的。有关更多信息，请参见 Win32 API 文档中的 TlsAlloc、TlsGetValue、TlsSetValue 和 TlsFree。  
  
Visual C++ 编译器提供了一个关键字（而不通过 API 层）使 TLS 操作更加自动化。将在下一节（TLS 的编译器实现）描述此语法。  
  
TLS 的编译器实现  
为了支持 TLS，已将新属性 thread 添加到了 C 和 C++ 语言，并由 Visual C++ 编译器支持。此属性是一个扩展存储类修饰符，如上一节中所述。使用 \_\_declspec 关键字声明 thread 变量。例如，以下代码声明了一个整数线程局部变量，并用一个值对其进行初始化：  
\_\_declspec( thread ) int tls\_i = 1; 

其实我们把 TLS 理解成为可以在预先给每条线程所使用并预先规划出一片区域的数据。  
文档中还有些关于 TLS 变量的使用地点这里总结一下:  
TLS 变量只能定义变量  
TLS 变量只能声明为公共静态的  
TLS 变量不能修饰函数  
不能在函数内定义，不能作为函数的参数  
在正常代码内不能引用 TLS 变量  
这些编译方面上的限制，应该是为代码安全与稳定性上设定的。毕竟，TLS 是作为多线程一部分使用的。在内存访问上没有明确的限制(在运行代码中自行定位 TLS 变量并引用是没有问题的)。  
下面我们了解一下 TLS 表的结构。  
TLS 表的目录索引处在数据目录的第 9 个索引上，可以使用微软提供的宏 IMAGE\_DIRECTORY\_ENTRY\_TLS 来直接定位。  
这个目录的 VirtualAddress 指向一个 IMAGE\_TLS\_DIRECTORY32 的结构，如下所示：  
IMAGE\_TLS\_DIRECTORY32 STRUCT  
    StartAddressOfRawData dd    ?  
    EndAddressOfRawData dd      ?  
    AddressOfIndex dd           ?  
    AddressOfCallBacks dd       ?  
    SizeOfZeroFill dd           ?  
    Characteristics dd          ?  
IMAGE\_TLS\_DIRECTORY32 ENDS  
这里解释一下  
StartAddressofRawData 为 TLS 数据起始的地址  
EndAddressOfRawData 为结束的地址  
AddressOfIndex 是 TLS 索引的地址  
AddressOfCallBacks 指向一组以 NULL 结尾的 IMAGE\_TLS\_CALLBACK 函数的地址数组  
以上都为地址,这里使用的是 VA，而不是 RVA 了。当修改时这是要注意的地方。  
SizeOfZeroFill 表明填充 TLS 变量区域的大小  
Characteristics 用作保留，可能用作 TLS 标志。  
在一个 EXE 程序访问 TLS 变量时要经过以下步骤。  
（这里需要注意的是 DLL 访问 TLS 变量除非这个 DLL 是静态链接到这个 EXE 中，否则使用 LoadLibrary 动态加载的 DLL 是不会加载 TLS 变量的）  
1.链接时。连接器设置 TLS 目录的 AddressOfIndex 字段。这个字段指向一个地址。在这个地址保存了程序所用的 TLS 索引。  
2.当线程创建的时候，通常在 TEB 开得 2ch 处位置防止一个指向 TLS 数组的指针。TEB 的地址放入 FS 中保存所以可以使用 FS\[2ch\]引用此 TLS 数组。  
3.我们在可执行代码中使用 1 提供的 TLS 索引与 2 提供的 TLS 数组的地址来使用 TLS 所提供的数据。  
需要注意的是 TLS 数组在每条线程都有维护的一个区域。这个数组的每个地址指出了程序中给定模块的 TLS 数据区的位置。TLS 索引指出了这个数组的哪个元素。  
这里给出一段使用 TLS 的代码。读者可以自行参阅 MSDN 的文档深入理解。  
代码:

```
Start:
    invoke MainFunc
    invoke ExitProcess, 1
CommonFunc proc
    LOCAL lpvData : LPVOID
    LOCAL dwThreadId : DWORD
    
    invoke TlsGetValue, dwTlsIndex
    .IF eax == 0
        ret
    .ENDIF
    mov lpvData, eax
    invoke Sleep, 5000
    ;; 显示信息
    invoke GetCurrentThread
    mov dwThreadId, eax
    invoke PrintLine, offset g\_szOutThread, addr dwThreadId
    invoke PrintLine, offset g\_szOutData, addr lpvData
    ret
    
CommonFunc endp 
  
ThreadFunc proc
    LOCAL lpvData : LPVOID
    LOCAL dwThreadId : DWORD
    invoke LocalAlloc, LPTR, 256
    mov lpvData, eax
    invoke TlsSetValue, dwTlsIndex, eax
    .IF eax == 0
        ret
    .ENDIF
    invoke GetCurrentThread
    mov dwThreadId, eax
    invoke PrintLine, offset g\_szOutThread, addr dwThreadId
    invoke PrintLine, offset g\_szOutData, addr lpvData  
    ;; 调用公共函数
    invoke CommonFunc
    ;; 释放内存
    invoke TlsGetValue, dwTlsIndex
    .IF eax != 0
        invoke LocalFree, lpvData
    .ENDIF
    ret

ThreadFunc endp  
  
MainFunc proc
    LOCAL IDThread : DWORD
    LOCAL hThread\[THREADCOUNT\] : HANDLE
    
    invoke TlsAlloc
    .IF eax == TLS\_OUT\_OF\_INDEXES
        ret
    .ENDIF
    mov dwTlsIndex, eax
    
    xor ecx, ecx
    mov ebx, THREADCOUNT
CreateThreadLoop:
    cmp ecx, ebx
    jz EndCreateThreadLoop
    push ecx
    invoke CreateThread, NULL, 0, ThreadFunc, NULL, 0, addr IDThread
    pop ecx
    mov hThread\[ecx\*sizeof HANDLE\], eax
    inc ecx
    jmp CreateThreadLoop           
EndCreateThreadLoop:    

    xor ecx, ecx
    mov ebx, THREADCOUNT
WaitThreadLoop:
    cmp ecx, ebx
    jz EndWaitThreadLoop
    push ecx
    invoke WaitForSingleObject, hThread\[ecx\*sizeof HANDLE\], INFINITE
    pop ecx
    inc ecx
    jmp WaitThreadLoop
    ;; 是否 TLS 索引
    invoke TlsFree, dwTlsIndex
    ret    
    
EndWaitThreadLoop:

    ret

MainFunc endp

PrintLine proc uses eax ebx ecx edx edi esi pFormat : LPSTR, pBuffer : LPSTR
;; it's must be include masm32.inc and masm32.lib
    LOCAL szBuf\[MAX\_PATH\] : BYTE
    
    mov ebx, pFormat
    mov edx, pBuffer
    invoke wsprintf, addr szBuf, ebx, edx
    invoke StdOut, addr szBuf
    mov ebx, offset CRLF
    invoke StdOut, ebx
    ret
    
CRLF db 0Dh,0Ah,0    
PrintLine endp 
end Start

```

这段代码很简单是 TLS 函数相关使用 C++的代码可以从 MSDN 文档处获取。  
接下来我们介绍一下关于 TLS 回调函数这个东西，这个东西对于编写病毒来说直安逸的。大多数调试器是从 PE 加载器开始加载，几乎都跳过读取此入口的。呵呵。话说壳技术也是从病毒技术发展而来的。是先有病毒后有壳的，他们两者有太多相似的地方。不过貌似安全圈里很多的技术都是从 Anti-安全到安全过渡的。想想 rootkit 的技术不也一样吗！  
这个回调函数的地址是由 AddressOfCallBacks 字段指出的。指向的地址是一个由 IMAGE\_TLS\_CALLBACK 函数组成的数组并以 NULL 做结束符。回调函数的顺序是依次调用。这个 TLS 回调函数是在加载引入表之后，所以在引入表中的函数都可以直接使用。如果加密了引入表，在这里就需要做一些手段了。不过要注意的是切不可在这里解密引入表。否则，加密引入表就没有意义了。  
回调函数的原型与 DLL 启动函数的样子很像。  
TLS\_CALLBACK proto Dllhandle : LPVOID, Reason : DWORD, Reserved : LPVOID  
参数如下:  
Dllhandle : 为模块的句柄  
Reason 可取以下值:  
DLL\_PROCESS\_ATTACH 1 : 启动一个新进程被加载  
DLL\_THREAD\_ATTACH 2 : 启动一个新线程被加载  
DLL\_THREAD\_DETACH 3 : 终止一个新线程被加载  
DLL\_PROCESS\_DETACH 0 : 终止一个新进程被加载  
Reserverd:用于保留，设置为 0  
函数的参数是被加载器设置。我们的函数只管判断就可以。  
下面就让我们使用 MASM 打造一个带有回调功能的 TLS 节的程序，最初使用见到使用 MASM 编写 TLS 相关是在一个俄国人的论坛的代码里。随后的附件中也会附带此代码名为:TlsInAsm。这里我们还是自己打造一个。在 TlsInAsm 中我学习到了一些关于 MASM 的编译选项。  
代码如下  
代码:

```
.data?
dwTLS\_Index dd  ?

 ;; 这里的 OPTION DOTNAME 的意思为让 ML 可以使用以“.”开头的变量，结构，联合，标号等
;; 开启这个选项也可以在 ML 中使用/Zm 选项来启动。
OPTION    DOTNAME
 ;; 定义一个 TLS 节
;; SEGMENT 标志在 ML 里是用于定义一个新节。
.tls  SEGMENT                        
TLS\_Start LABEL  DWORD
 dd    0100h    dup ("slt.")
TLS\_End   LABEL  DWORD
.tls   ENDS
OPTION    NODOTNAME

.data
TLS\_CallBackStart   dd TlsCallBack0
TLS\_CallBackEnd     dd 0

g\_szTitle           db "Hello TLS",0
g\_szInTls           db "我在 TLS 里",0
g\_szInNormal        db "我在正常代码内",0

 ;; 这里需要注意的是，
;; 必须要将此结构声明为 PUBLIC,用于让连接器连接到指定的位置，
;; 其次结构名必须为\_tls\_uesd 这是微软的一个规定。编译器引入的位置名称也如此。
PUBLIC \_tls\_used
\_tls\_used IMAGE\_TLS\_DIRECTORY <TLS\_Start, TLS\_End, dwTLS\_Index, TLS\_CallBackStart, 0, ?>

.code
Start:
    ;; 在正常代码中使用
    invoke MessageBox, NULL, addr g\_szInNormal, addr g\_szTitle, MB\_OK
    invoke ExitProcess, 1
    ret

;; TLS 的回调函数
TlsCallBack0 proc Dllhandle : LPVOID, dwReason : DWORD, lpvReserved : LPVOID
   ;; 这里可以判断 dwReason 发生的条件
   mov eax, dwReason
   ;; 在进行加载时被调用
   cmp eax, DLL\_PROCESS\_ATTACH
   jnz ExitTlsCallBack0
   invoke MessageBox, NULL, addr g\_szInTls, addr g\_szTitle, MB\_OK
   mov dword ptr\[TLS\_Start\],0  
   xor eax, eax
   inc eax
ExitTlsCallBack0:   
   ret
TlsCallBack0   ENDP

end Start

```

以上代码给出了如何汇编一个具有回调函数的 TLS 节程序。代码很简单有关 MASM 的选项可参照 masm32 中 help 的自带文档。可惜没中文也没有例子，此文档显的非常鸡肋。不过有总比没好。如果有对 MASM 语法熟悉的大哥，请赐教小弟一二。我使用 MASM 也是从今年开始，纯属小白一个。  
下面该轮到修改 TLS 了。这段也可以作为加解密 TLS 用，但是个人感觉不到 TLS 加解密的必要，纯属一个可选的选项。不过用作 Anti-debug 是个用途。yC 壳中也是将 TLS 进行了一个简单的移动。而在加密时掠过以.tls 为名的节，这个就会造成不稳定性。而完美的处理 TLS 比较麻烦，如果 TLS 回调函数不止一个，我们必须要读取所有的 TLS 回调函数并判断它的结束当然这需要一个反汇编引擎来做。而有些加入 Anti-反汇编代码的也会产生读取错误，最后还要做一些重定位的工作。考虑至此我们此刻的代码功能是这样的，首先复制 TLS 表，然后进行一个 TLS 入口点的替换。最后跳转回 TLS 真正的回调函数中。由于 TLS 程序并不常见，读者可以使用以上提供的程序进行测试。  
现在我们来描述一下算法:  
现在我们有这样几种情况  
1.有 TLS 表  
  1.1 有 TLS 回调函数  
     修改 AddressOfCallBacks 字段指向区域的第一个回调函数的 VA 为我们自己回调函数的 VA  
  1.2 无 TLS 回调函数  
     设置一个新的 TLS 回调函数地址表,并将新的回调函数地址表的 VA 设置给它  
2.无 TLS 表  
  汇编一个新的 TLS 表  
在我们自己的回调函数中也有不一样的地方。  
如果是 1.1 的情况那么要记录原先第一个回调函数的 VA 并且最后进行跳转  
如果是 1.2 的情况那么直接返回即可.(最后不要忘记清除参数所造成的堆栈开销)  
代码:

```
ModifyTLS proc szFileName : LPSTR
    LOCAL hFile : HANDLE
    LOCAL hMap : HANDLE
    LOCAL pMem : LPVOID
    LOCAL dwOrigFileSize : DWORD
    LOCAL dwNTHeaderAddr : DWORD
    LOCAL dwNewTlsSectionOffset : DWORD
    
    ;; init data
    xor eax, eax
    mov g\_bError, al
    mov eax, OWNTLS\_SIZE
    mov g\_dwNewSectionSize, eax
    
    ;; open file
    invoke CreateFile, szFileName,\\
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
    mov dwOrigFileSize, eax
    add eax, APPEND\_SIZE
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
    
    ;; 增加一个新节，此节为我们自己的 TLS 节
    invoke AddSection, pMem, offset g\_szNewSectionName, g\_dwNewSectionSize
    mov dwNewTlsSectionOffset, eax    
    
    ;; 寻找原始的 TLS 节并且记录它
    mov eax, dword ptr \[esi\].OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_TLS \* sizeof IMAGE\_DATA\_DIRECTORY\].VirtualAddress
    .IF eax == 0
        ;; 如果没有 TLS 节则汇编一个新的
        ;; 设置标志变量
        mov eax, offset OwnTLS\_OrigYes
        push 0
        pop dword ptr \[eax\]
        jmp NoTLSCallBack
    .ENDIF
    invoke RVA2Offset, pMem, eax
    add eax, pMem
    ;; 复制这个表到我们自己的内存
    mov esi, eax
    lea edi, Orig\_TLS
    mov ecx, sizeof IMAGE\_TLS\_DIRECTORY
    cld
    rep movsb
    ;; 汇编一个新的 TLS 节并设置回调函数   
    ;; 这里检验是否存在回调函数
    mov eax, offset Orig\_TLS
    assume eax : ptr IMAGE\_TLS\_DIRECTORY
    mov eax, dword ptr \[eax\].AddressOfCallBacks
    ;; 首先将 VA 转换为 RVA 然后转换为 FVA
    mov esi, dwNTHeaderAddr
    assume esi : ptr IMAGE\_NT\_HEADERS
    sub eax, dword ptr \[esi\].OptionalHeader.ImageBase
    invoke RVA2Offset, pMem, eax
    add eax, pMem
    mov ebx, dword ptr \[eax\]
    test ebx, ebx
    jz NoTLSCallBack
    ;; 保存原回调函数的地址
    mov edx, offset OwnTLS\_OrigCallBackEntry
    mov dword ptr \[edx\], ebx
    ;; 有回调函数这修改，修改位置为 eax 指向的地方
    mov ebx, dwNewTlsSectionOffset
    assume ebx : ptr IMAGE\_SECTION\_HEADER
    mov ebx, dword ptr \[ebx\].VirtualAddress
    add ebx, dword ptr \[esi\].OptionalHeader.ImageBase
    mov dword ptr \[eax\], ebx
    ;; 设置 TLS 回调跳转标志
    mov eax, offset OwnTLS\_OrigYes
    push 1
    pop dword ptr \[eax\]
    jmp ContinueNoTLSCallBack
NoTLSCallBack:
    ;; 如果没有回调函数，则重新建立一个回调函数地址表
    ;; 并且设置新的 TLS 目录的字段
    mov ebx, dwNewTlsSectionOffset
    assume ebx : ptr IMAGE\_SECTION\_HEADER
    mov edx, dword ptr \[ebx\].VirtualAddress
    add edx, offset OwnTLS\_CallBackStart - offset OwnTLS\_Section
    add edx, dword ptr \[esi\].OptionalHeader.ImageBase
    mov eax, offset Orig\_TLS
    assume eax : ptr IMAGE\_TLS\_DIRECTORY
    mov dword ptr \[eax\].AddressOfCallBacks, edx
    ;; 设置 AddressOfCallBacks
    sub edx, offset OwnTLS\_CallBackStart - offset OwnTLS\_CallBack
    mov ebx, offset OwnTLS\_CallBackStart
    mov dword ptr \[ebx\], edx
    ;; 设置 AddressOfIndex
    add edx, offset OwnTLS\_Index - offset OwnTLS\_CallBack
    mov dword ptr \[eax\].AddressOfIndex, edx
    ;; 设置 StartAddressOfRawData
    add edx, offset OwnTLS\_Start - offset OwnTLS\_Index
    mov dword ptr \[eax\].StartAddressOfRawData, edx
    ;; 设置 EndAddressOfRawData
    add edx, offset OwnTLS\_End - offset OwnTLS\_Start
    mov dword ptr \[eax\].EndAddressOfRawData, edx
    ;; 设置 SizeOfZeroFill
    push 0
    pop dword ptr\[eax\].SizeOfZeroFill
ContinueNoTLSCallBack:
    ;; 这里复制 TLS 到新的 TLS 表中
    mov eax, dwNewTlsSectionOffset
    assume eax : ptr IMAGE\_SECTION\_HEADER
    ;; 将数据目录设置到我们自己的 TLS 节内
    mov eax, dword ptr \[eax\].VirtualAddress
    add eax, offset Orig\_TLS - offset OwnTLS\_Section
    mov dword ptr \[esi\].OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_TLS \* sizeof IMAGE\_DATA\_DIRECTORY\].VirtualAddress, eax
    push sizeof IMAGE\_TLS\_DIRECTORY
    pop dword ptr \[esi\].OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_TLS \* sizeof IMAGE\_DATA\_DIRECTORY\].isize
    
    ;; 复制内容到新的 TLS 节
    mov edi, dwNewTlsSectionOffset
    assume edi : ptr IMAGE\_SECTION\_HEADER
    mov edi, dword ptr \[edi\].PointerToRawData
    add edi, pMem
    mov esi, offset OwnTLS\_Section
    mov ecx, g\_dwNewSectionSize
    cld
    rep movsb
    
    ;; close handle & write it
    invoke UnmapViewOfFile, pMem
    invoke CloseHandle, hMap
    invoke CloseHandle, hFile                           
 LogicShellExit:
     .IF g\_bError == 0
         ;; show success message  
         invoke MessageBox, NULL, offset g\_szDone, offset g\_szDoneCap, MB\_ICONINFORMATION
     .ENDIF        
     assume eax : nothing
     assume ebx : nothing
     assume edx : nothing
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
ShowErr:
     invoke MessageBox, NULL, eax, offset g\_szErr, MB\_ICONERROR
     mov al, 1
     mov g\_bError, al
     jmp LogicShellExit

;; ----- 自己的 TLS 数据 -----
OwnTLS\_Section:
;; 我们自己的 TLS 回调函数
OwnTLS\_CallBack:
    ;; 只判断它的启动状态
    OwnTLS\_CallBackArg\_Reason           equ 08h
    ;; 这里没有做堆栈帧所以 esp 指向的是返回地址和
    ;; 而我们所用的参数 Reason 是第 2 个所以这里加 08h
    mov eax, dword ptr \[esp+OwnTLS\_CallBackArg\_Reason\]
    cmp eax, DLL\_PROCESS\_ATTACH
    jnz ExitOwnTLS\_CallBack
    ;; 在这里停下,把注释停掉，可以在 ollydbg 里调试查看
    ;int 3
ExitOwnTLS\_CallBack:        
    ;; 这里做一个判断，如果是新的 TLS 则直接退出如果是修改的 TLS 则跳入原来的
    call GetEip
    GetEip:
    pop eax
    mov edx, eax
    add eax, offset OwnTLS\_OrigYes - offset GetEip
    mov eax, dword ptr \[eax\]
    test eax, eax   ; 0 为原来没有 TLS,1 为有 TLS 回调
    ;; 这里取出原始的 TLS 回调函数入口并跳入
    jz NotHaveOrigTlsCallBack
    add edx, offset OwnTLS\_OrigCallBackEntry - offset GetEip
    jmp dword ptr \[edx\]      ; 跳入正常的回调函数入口
NotHaveOrigTlsCallBack:
    retn 03h                 ; 清除参数造成的堆栈开销    
    ;; ----- 原始回调函数入口 -----         
;; 原来的程序是否有回调函数
OwnTLS\_OrigYes              dd  0  
;; 原 TLS 回调函数入口点
OwnTLS\_OrigCallBackEntry    dd  0
;; TLS 变量索引
OwnTLS\_Index                dd  0
;; TLS 回调函数地址表
OwnTLS\_CallBackStart        dd  0
OwnTLS\_CallBackEnd          dd  0
;; TLS 变量区域
OwnTLS\_Start:
    db 80h dup ("tls")
OwnTLS\_End:
;; 保存原始的 TLS 节目录
Orig\_TLS IMAGE\_TLS\_DIRECTORY <0>
EndOwnTLS\_Section:

ModifyTLS endp

```

到这里 TLS 相关就讲解完毕了。在 TLS 回调函数中可以用异常链的方法取出 ntdll.dll 的句柄并用过引出表找到你想用的函数并使用它。例如加载驱动。呵呵！！！  
这个程序在加有 TLS 节的程序会修改 TLS 回调函数地址。  
对无 TLS 节的程序会给他添加一个。  
  
这节对 TLS 节本身的使用并没有过多的探讨。TLS 回调函数与加密解思路是在这里了。怎样使用就看读者的想象力了。![](img/8e0d77ee22c3d8438aad7d42c7829e53.jpg "使眼色")

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || UseTLS\_src.rar  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || UseTLS\_bin.rar  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || TestTLS\_src.rar  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || TestTLS\_bin.rar  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || ModifyTLS\_src.rar  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || ModifyTLS\_bin.rar  
 |

一个国外玩家关于 TLS 的代码。  
最初汇编一个 TLS 节就是从中学习的。。。 这里也上传到此![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || TlsInAsm.rar  
 |
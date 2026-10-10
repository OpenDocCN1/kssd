

# 笑解 API 函数 －－ API 绝密档案系列之三

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-326.htm`](https://www.pediy.com/kssd/pediy08/pediy8-326.htm)

笑解 API 函数 －－ API 绝密档案系列之三  高处不胜寒  
  
"怎么又笑了，防热涂的血" －－ 摘自悍匪座山雕语录。  
在这一篇中，我们首先简单介绍另一个函数 GetProcAddress， GetProcAddress 和 GetModuleHandle 函数是兄弟的关系，我个人认为这两个函数的命名非常容易让用户产生混乱，而且有些颠倒的感觉，似乎如下命名更好：  

> ```
> 代码:
> 
> * * *
> 
> 将 GetModuleHandle 改为
> GetProcBaseAddress        //GetModuleHandle 实质是获取进程加载的基地址
> 将 GetProcAddress 改为
> GetFunctionAddress        //GetProcAddress  实质是获取函数的调用地址
> 
> * * *
> 
> 
> ```
> 
>     

  
不过老盖是老大，老大喜欢将太阳叫成月亮，你只能跟着喊“月亮(太阳)升起来咯，喜洋洋咯赫唉，挑起扁担啷当彩......"。  
Windows 操作系统进入 2K 时代，开始全面支持 Unicode 编码方式，我们从 GetModuleHandle 的分析中已经看到。说起 Unicode，不得不提一句，Unicode 编码方式对 C 语言来说，简直就是悲剧。大家都知道 C 格式的字符串是以 0x0 作为终止符号的。即有如下表示形式：  

> ```
> 代码:
> 
> * * *
> 
> db  "GetProcAddress",0
> 在内存中的映象为：
> 47 65 74 50 72 6F 63 41 64 64 72 65 73 73 00
> 如果是 Uncode 在内存中的映象为：
> 47 00 65 00 74 00 50 00 72 00 6F 00 63 00 41 00 64 00 64 00 72 00 65 00 73 00 73 00
> 
> * * *
> 
> 
> ```
> 
>     

  
哈哈， C 格式的字符串王朝一瞬间就灰飞烟灭了。  
如果你留心就会发现，Win2K 以后版本的函数凡是涉及字符串的基本都有 A 和 W 的区分，但这个 GetProcAddress 就没有 A 和 W 的区分，即没有 GetProcAddressA 和 GetProcAddressW，为什么呢？其实就是上面的原因，C 编译器生成的程序输出表都是以 C 格式字符串来表示函数名，GetProcAddress 是如何获得函数的调用地址的？就是利用输出表，所以就没有那个 A 和 W 了，哈哈，真有意思。  
说了半天废话，言归正传，GetProcAddress 的代码如下：  

> ```
> 代码:
> 
> * * *
> 
> 77E80CAB ; Exported entry 344. GetProcAddress
> 77E80CAB ; ======== S U B R O U T I N E ====================
> 77E80CAB ; FARPROC \_\_stdcall GetProcAddress(HMODULE hModule,LPCSTR lpProcName)
> 77E80CAB       public \_GetProcAddress@8
> 77E80CAB \_GetProcAddress@8 proc near
> 77E80CAB ProcNameString= dword ptr -8
> 77E80CAB hModule= dword ptr  8
> 77E80CAB lpProcName= dword ptr  0Ch
> 77E80CAB       push ebp
> 77E80CAC       mov ebp, esp
> 77E80CAE       push ecx
> 77E80CAF       push ecx
> 77E80CB0       push ebx
> 77E80CB1       push edi
> 77E80CB2       mov edi, \[ebp+lpProcName\]           ; 获取指向函数名的指针(地址)
> 77E80CB5       mov ebx, 0FFFFh
> 77E80CBA       cmp edi, ebx                        ; 如果 lpProcName 是字符串，这指向这个字符串的地址一定
> 77E80CBA                                           ; 大于 0FFFFh
> 77E80CBC       jbe short IfProcNameIsNull
> 77E80CBE       lea eax, \[ebp+ProcNameString\]
> 77E80CC1       push edi                            ; SourceString
> 77E80CC2       push eax                            ; DestinationString
> 77E80CC3       call ds:\_\_imp\_\_RtlInitString@8      ; \_\_declspec(dllimport) RtlInitString(x,x)
> 77E80CC9       lea eax, \[ebp+lpProcName\]           ; 将 lpProcName 作为变量使用
> 77E80CCC       push eax                            ; ProcedureAddress
> 77E80CCD       lea eax, \[ebp+ProcNameString\]
> 77E80CD0       push 0                              ; Ordinal
> 77E80CD2       push eax                            ; Name
> 77E80CD3       jmp short GetProcAddress
> 77E80CD5 IfProcNameIsNull:
> 77E80CD5       lea eax, \[ebp+lpProcName\]
> 77E80CD8       push eax                            ; ProcedureAddress
> 77E80CD9       push edi                            ; Ordinal
> 77E80CDA       push 0                              ; Name
> 77E80CDC GetProcAddress:                           ; flag
> 77E80CDC       push 0
> 77E80CDE       push \[ebp+hModule\]                  ; hModule
> 77E80CE1       call \_BasepMapModuleHandle@8        ; BasepMapModuleHandle(x,x)
> 77E80CE6       push eax                            ; BaseAddress
> 77E80CE7       call \_LdrGetProcedureAddress@16     ; LdrGetProcedureAddress(x,x,x,x)
> 77E80CEC       test eax, eax
> 77E80CEE       jl short IfFunAddrEquZero
> 77E80CF0       push 0                              ; flag
> 77E80CF2       push \[ebp+hModule\]                  ; hModule
> 77E80CF5       call \_BasepMapModuleHandle@8        ; BasepMapModuleHandle(x,x)
> 77E80CFA       cmp \[ebp+lpProcName\], eax
> 77E80CFD       jnz short IfFunAddrIsOk
> 77E80CFF       cmp ebx, edi
> 77E80D01       sbb eax, eax
> 77E80D03       neg eax
> 77E80D05       add eax, STATUS\_ORDINAL\_NOT\_FOUND
> 77E80D0A IfFunAddrEquZero:
> 77E80D0A       push eax                            ; Status
> 77E80D0B       call \_BaseSetLastNTError@4          ; BaseSetLastNTError(x)
> 77E80D10       xor eax, eax
> 77E80D12       jmp short Exit
> 77E80D14 IfFunAddrIsOk:                            ; 变量参数 lpProcName 的内容为 ProcAddress
> 77E80D14       mov eax, \[ebp+lpProcName\]           ; 变量参数 lpProcName 的内容为 ProcAddress
> 77E80D17 Exit:
> 77E80D17       pop edi
> 77E80D18       pop ebx
> 77E80D19       leave
> 77E80D1A       retn 8
> 77E80D1A \_GetProcAddress@8 endp
> 
> 77E68210 ; ======== S U B R O U T I N E ====================
> 77E68210 ; int \_\_stdcall BasepMapModuleHandle(HMODULE hModule,BOOLEAN flag)
> 77E68210 \_BasepMapModuleHandle@8 proc near
> 77E68210 hModule= dword ptr  8
> 77E68210 flag  = byte ptr  0Ch
> 77E68210       push ebp
> 77E68211       mov ebp, esp
> 77E68213       mov eax, \[ebp+hModule\]
> 77E68216       test eax, eax
> 77E68218       jz short loc\_77E68229
> 77E6821A       test al, 1
> 77E6821C       jz short loc\_77E68235
> 77E6821E                                            ;if flag == 0
> 77E6821E                                            ;  ecx = 0
> 77E6821E                                            ;else
> 77E6821E                                            ;  ecx = 0xFFFFFFFF
> 77E6821E       mov cl, \[ebp+flag\]
> 77E68221       neg cl
> 77E68223       sbb ecx, ecx
> 77E68225       and eax, ecx
> 77E68227       jmp short loc\_77E68235
> 77E68229 loc\_77E68229:
> 77E68229       mov eax, large fs:18h
> 77E6822F       mov eax, \[eax+TEB.Peb\]
> 77E68232       mov eax, \[eax+PEB.ImageBaseAddress\]
> 77E68235 loc\_77E68235:
> 77E68235       pop ebp
> 77E68236       retn 8
> 77E68236 \_BasepMapModuleHandle@8 endp
> 
> * * *
> 
> 
> ```
> 
>     

  
我们来简单的分析一下这段代码，首先我们看看老盖对参数 lpProcName 的解释(英文不好的朋友可以参考 qduwg 老师的翻译)  

引用:    
lpProcName   
Pointer to a null-terminated string containing the function name, or specifies the function's ordinal value. If this parameter is an ordinal value, it must be in the low-order word; the high-order word must be zero.   

  
变参 lpProcName 即可以是函数的名称，也可以是函数的序号，这里说明一下，在你的开发中，千万不要使用函数的序号，这个功能其实是老盖的专用，因为 Updata 或升级将导致函数的序号发生变换，你的程序立刻就可能彻底完蛋，你只有使用函数名字。其实这个参数还有一个功能，就是返回函数的调用地址，和 EAX 中返回的相同。  
  
函数的头部首先判断参数是什么性质，字符串还是序号：  

> ```
> 代码:
> 
> * * *
> 
> 77E80CB2       mov edi, \[ebp+lpProcName\]           ; 获取指向函数名的指针(地址)
> 77E80CB5       mov ebx, 0FFFFh
> 77E80CBA       cmp edi, ebx                        ; 如果 lpProcName 是字符串，这指向这个字符串的地址一定
> 77E80CBA                                           ; 大于 0FFFFh
> 77E80CBC       jbe short IfProcNameIsNull
> 
> * * *
> 
> 
> ```
> 
>     

  
  
**1.如果是字符串**  
如果是字符串，首先调用 RtlInitString:  

> ```
> 代码:
> 
> * * *
> 
> 77E80CBE       lea eax, \[ebp+ProcNameString\]
> 77E80CC1       push edi                            ; SourceString
> 77E80CC2       push eax                            ; DestinationString
> 77E80CC3       call ds:\_\_imp\_\_RtlInitString@8      ; \_\_declspec(dllimport) RtlInitString(x,x)
>                ;下面是为调用 LdrGetProcedureAddress(x,x,x,x) 而设置入口参数
> 77E80CC9       lea eax, \[ebp+lpProcName\]           ; 将 lpProcName 作为变量使用
> 77E80CCC       push eax                            ; ProcedureAddress
> 77E80CCD       lea eax, \[ebp+ProcNameString\]
> 77E80CD0       push 0                              ; Ordinal
> 77E80CD2       push eax                            ; Name
> 77E80CD3       jmp short GetProcAddress             ; 跳到获取函数地址的具体调用
> 
> * * *
> 
> 
> ```
> 
>     

  
这里补充说明上一篇的一个类似的函数 RtlInitUnicodeString，我们首先来看两个结构：  

> ```
> 代码:
> 
> * * *
> 
> UNICODE\_STRING struc ; (sizeof=0X8, standard type)
>   Length           dw ?
>   MaximumLength   dw ?
>   Buffer           dd ?                             ;指向字符串的指针
> UNICODE\_STRING ends
> 
> STRING struc ; (sizeof=0X8)
>   Length           dw ?
>   MaximumLength   dw ?
>   Buffer           dd ?                             ;指向字符串的指针
> STRING ends
> 
> * * *
> 
> 
> ```
> 
>     

  
所谓初始化字符串，其实就是构造这样一个结构，将 Length 中填上字符串的长度，将 MaximumLength 中填上字符串允许的最大长度，Buffer 中填上指向字符串的地址。我不知老盖是怎样思维的，这里字符串的长度单位使用 Word ，明显没有战略眼光，限制了字符串的最大长度，局限了字符串的使用功能。我们来看看 Borland 的工程师们是怎样定义字符串的。  

> ```
> 代码:
> 
> * * *
> 
> strings struc ; (sizeof=0X8, variable size)
>   top   dd ?
>   Len   dd ?
>   Text   db 0 dup(?)                        ; string(C)
> strings ends
> 
> * * *
> 
> 
> ```
> 
>     

  
其中 top 中为 0FFFFFFFFh，表示此字符串已经初始化，也是字符串的标志。Len 字符串的长度，单位是 DWORD，最后是一个不定长的字符串，以零结尾，但在 Delphi 中操作字符串，实际是以 Len 来实现，并不关心字符串中是否有 0，这个 0 是用来兼容 C 字符串的，如果要把 Delphi 的字符串当作 C 字符串来使用，这个 0 就不能乱放了。由于使用 DWORD 来定义字符串的长度，所以 Delphi 中字符串的操作远比我们想象的要来得强大，在 Delphi 中你可以将整个文件读入一个字符串中(包括一个可执行文件)，然后用字符串指针去随意操作，所有字符串的函数都可以使用，比使用流更方便。这就是被称为巨型字符串的东西了。字符串的长度在 32 位系统只和内存的大小有关，即 DWORD 的 4G 寻址空间，Borland 的工程师给出一个比较安全的数值，小于 2G，防止字符串覆盖了老盖的操作系统，哈哈。  
  
**2.如果是序号**  
序号的处理比较简单，直接拿过来使用就完了。  

> ```
> 代码:
> 
> * * *
> 
> 77E80CD5 IfProcNameIsNull:
>                ;下面是为调用 LdrGetProcedureAddress(x,x,x,x) 而设置入口参数
> 77E80CD5       lea eax, \[ebp+lpProcName\]
> 77E80CD8       push eax                            ; ProcedureAddress
> 77E80CD9       push edi                            ; Ordinal
> 77E80CDA       push 0                              ; Name
> 
> * * *
> 
> 
> ```
> 
>     

  
  
**3.获取函数调用地址**  
作为系统操作，首要考虑的就是运行稳定、强壮，这里调用 BasepMapModuleHandle 基本就是起这个作用，如果打造自己的 API 函数，就没有必要怎样复杂了。  

> ```
> 代码:
> 
> * * *
> 
> 77E80CDC GetProcAddress:                           ; flag
> 77E80CDC       push 0
> 77E80CDE       push \[ebp+hModule\]                  ; hModule
> 77E80CE1       call \_BasepMapModuleHandle@8        ; 检查 hModule 的合法性
> 77E80CE6       push eax                            ; BaseAddress
> 77E80CE7       call \_LdrGetProcedureAddress@16     ; LdrGetProcedureAddress(x,x,x,x)
> 77E80CEC       test eax, eax
> 77E80CEE       jl short IfFunAddrEquZero
> 77E80CF0       push 0                              ; flag
> 77E80CF2       push \[ebp+hModule\]                  ; hModule
> 77E80CF5       call \_BasepMapModuleHandle@8        ; 检查函数调用地址的合法性
> 77E80CFA       cmp \[ebp+lpProcName\], eax
> 77E80CFD       jnz short IfFunAddrIsOk
> 77E80CFF       cmp ebx, edi
> 77E80D01       sbb eax, eax
> 77E80D03       neg eax
> 77E80D05       add eax, STATUS\_ORDINAL\_NOT\_FOUND
> 77E80D0A IfFunAddrEquZero:                         ; Status
> 77E80D0A       push eax
> 77E80D0B       call \_BaseSetLastNTError@4          ; BaseSetLastNTError(x)
> 77E80D10       xor eax, eax
> 77E80D12       jmp short Exit
> 
> * * *
> 
> 
> ```
> 
>     

  
不管 lpProcName 变参是函数名还是序号，最后都是调用位于 ntdll.dll 中的 LdrGetProcedureAddress ，这个函数的过程也是很复杂的，但主要工作就是通过输出表来获得函数调用地址，我们只列出相应部分的代码，这部分代码在一个叫着 LdrpSnapThunk 的子过程中。  
我不知大家对这个函数名有什么看法，你能从函数的命名上猜出这个函数做了 Get Function Address 的工作吗？如果你能，你的智商肯定超过爱因斯坦，将来没准给我们弄出一个什么“背对论”来。昨天为了这个函数名，我请教了我们那位密码语言天才伊万大叔。将我对 LdrSnapThunk 的译文“调遣一部带有巨大象鼻吸盘新型设备，用外形似象鼻子的那么一个吸盘吸住一根将要打到树上的大铁棍”，告诉伊万大叔，多么富有诗意的名字。当我正在为我伟大的翻译和理解而沾沾自喜的时候，伊万大叔大吼一声：“错，你知道老盖生活在什么地方吗？”，我想，我还没有愚蠢到连老盖先生住在那里都不知道的地步。伊万大叔接着说：“你知道那里是什么文化最出名吗？，那就是多元文化，写这段程序的先生没准来自于爪哇(Java)国，你知道 Java 吗？”，我答：“这个知道，不就是能让很多黄色下流的东西流入每个家庭的一种下流语言吗”。伊万大叔微笑道：“孺子可教也，在爪哇国，LdrpSnapThunk 的含义就是 Get Function Address，明白了吗，就象我弄一个俄文名字，然后用英文对应的字母写出，你能看懂吗，哈哈”。哇赛!，天才就是不同，连这都解的出，可惜天下父母要倒霉了，“背对论”所要研究的就是当一个男孩和一个女孩背对背靠在一起是，时空所发生的变化。  
马掌钉在膝盖上，唉! 离蹄(题)太远了，打住。  
  
下面我们来看看这个“象鼻子”是怎么吸到函数调用地址的：(这个函数的调用参数都是我瞎掰的，我不知道到那里去找这个调用参数)  

> ```
> 代码:
> 
> * * *
> 
> 77F8C0B8 ; NTSTATUS \_\_stdcall LdrpSnapThunk(PVOID BaseAddress,        \\
>                                             ANSI\_STRING \*lpProcName,  \\
>                                             ULONG Ordinal,            \\
>                                             PVOID \*ProceAddr,         \\
>                                             PVOID ExportTabAddr,      \\
>                                             int,                      \\
>                                             BOOLEAN ShowNotFound,     \\
>                                             int Unknown)
> 77F8C0B8 \_LdrpSnapThunk@32 proc near
> 77F8C0B8 var\_24= dword ptr -24h
> 77F8C0B8 var\_20= dword ptr -20h
> 77F8C0B8 var\_18= dword ptr -18h
> 77F8C0B8 var\_10= dword ptr -10h
> 77F8C0B8 ProcName= STRING ptr -8
> 77F8C0B8 BaseAddress= dword ptr  8
> 77F8C0B8 lpProcName= dword ptr  0Ch
> 77F8C0B8 Ordinal= dword ptr  10h
> 77F8C0B8 ProceAddr= dword ptr  14h
> 77F8C0B8 ExportTabAddr= dword ptr  18h
> 77F8C0B8 arg\_14= dword ptr  1Ch
> 77F8C0B8 ShowNotFound= byte ptr  20h
> 77F8C0B8 Unknown= dword ptr  24h
> 77F8C0B8       push ebp
> 77F8C0B9       mov ebp, esp
> 77F8C0BB       sub esp, 24h
> 77F8C0BE       mov eax, \[ebp+Ordinal\]
> ......
> ......
> 77F8C157 RealGetFunctionAddress:
> 77F8C157       push edi                                    ; AddressOfNameOrdinals
> 77F8C158       push eax                                    ; AddressOfNames
> 77F8C159       push \[ebp+BaseAddress\]                      ; BaseAddress
> 77F8C15C       push \[esi+IMAGE\_EXPORT\_DIRECTORY.NumberOfNames\] ; NumberOfNames
> 77F8C15F       push \[ebp+lpProcName\]                       ; lpProcName
> 77F8C162       call \_LdrpNameToOrdinal@20                  ; LdrpNameToOrdinal(x,x,x,x,x)
> 77F8C167 loc\_77F8C167:
> 77F8C167       movzx ecx, ax
> 77F8C16A       cmp ecx, \[esi+IMAGE\_EXPORT\_DIRECTORY.NumberOfFunctions\]
> 77F8C16D       jnb loc\_77F8C248
> 77F8C173       mov eax, \[esi+IMAGE\_EXPORT\_DIRECTORY.AddressOfFunctions\]
> 77F8C176       mov edx, \[ebp+BaseAddress\]
> 77F8C179       add eax, edx                                ; Offset + BaseAddress
> 77F8C17B       lea eax, \[eax+ecx\*4\]                        ; Offset + Ordinal \* 4
> 77F8C17E       mov ecx, \[eax\]                              ; OffsetFunction
> 77F8C180       add ecx, edx                                ; FunctionAddress = OffsetFunction + BaseAddress
> 77F8C182       mov edx, \[ebp+ProceAddr\]
> 77F8C185       cmp ecx, esi
> 77F8C187       mov \[edx\], ecx
> 77F8C189       jbe loc\_77F8C243
> ......
> ......
> 
> * * *
> 
> 
> ```
> 
>     

  
下面我们弄一个 PE 头来和 Export 表来看看：  

> ```
> 代码:
> 
> * * *
> 
> 77F800D0 PEHead db 'PE',0,0                     ; Signature
> 77F800D0       dw 14Ch                          ; FileHeader.Machine
> 77F800D0       dw 7                             ; FileHeader.NumberOfSections
> 77F800D0       dd 41E648E0h                     ; FileHeader.TimeDateStamp
> 77F800D0       dd 0                             ; FileHeader.PointerToSymbolTable
> 77F800D0       dd 0                             ; FileHeader.NumberOfSymbols
> 77F800D0       dw 0E0h                          ; FileHeader.SizeOfOptionalHeader
> 77F800D0       dw 230Eh                         ; FileHeader.Characteristics
> 77F800D0       dw 10Bh                          ; OptionalHeader.Magic
> ......                                          
> ......                                          
> 77F800D0       dd offset ExportTableBase        ; OptionalHeader.Export.VirtualAddress
> 77F800D0       dd 83D7h                         ; OptionalHeader.Export.Size
> 77F800D0       dd offset ImportTableBase        ; OptionalHeader.Import.VirtualAddress
> 77F800D0       dd 0                             ; OptionalHeader.Import.Size
> 77F800D0       dd offset ResourceTableBase      ; OptionalHeader.Resource.VirtualAddress
> 77F800D0       dd 0DCA0h                        ; OptionalHeader.Resource.Size
> ......
> ......
> 
> 77FBBE20 ExportTableBase 
> 77FBBE20       dd 0                              ; Characteristics ; "ntdll.dll"
> 77FBBE20       dd 41AFD201h                      ; TimeDateStamp
> 77FBBE20       dw 0                              ; MajorVersion
> 77FBBE20       dw 0                              ; MinorVersion
> 77FBBE20       dd offset aNtdll\_dll\_0            ; Name
> 77FBBE20       dd 1                              ; Base
> 77FBBE20       dd 4A5h                           ; NumberOfFunctions
> 77FBBE20       dd 4A5h                           ; NumberOfNames
> 77FBBE20       dd offset AddrOfFunBase           ; AddressOfFunctions
> 77FBBE20       dd offset AddrOfNamesBase         ; AddressOfNames
> 77FBBE20       dd offset AddrOfNameOrdinalsBase  ; AddressOfNameOrdinals
> 
> 77FBBE48 AddrOfFunBase 
> 77FBBE48       dd offset PropertyLengthAsVariant
> 77FBBE4C       dd offset RtlConvertPropertyToVariant
> 77FBBE50       dd offset RtlConvertVariantToProperty
> 77FBBE54       dd offset @RtlUlongByteSwap@4
> 77FBBE58       dd offset @RtlUlonglongByteSwap@4
> 77FBBE5C       dd offset @RtlUshortByteSwap@4
> 77FBBE60       dd offset \_CsrAllocateCaptureBuffer@8
> ......
> 
> 77FBD0DC AddrOfNamesBase 
> 77FBD0DC       dd offset aCsrallocatecapturebuff   ; "CsrAllocateCaptureBuffer"
> 77FBD0E0       dd offset aCsrallocatemessagepoin   ; "CsrAllocateMessagePointer"
> 77FBD0E4       dd offset aCsrcapturemessagebuffe   ; "CsrCaptureMessageBuffer"
> 77FBD0E8       dd offset aCsrcapturemessagestrin   ; "CsrCaptureMessageString"
> 77FBD0EC       dd offset aCsrcapturetimeout        ; "CsrCaptureTimeout"
> 77FBD0F0       dd offset aCsrclientcallserver      ; "CsrClientCallServer"
> 77FBD0F4       dd offset aCsrclientconnecttoserv   ; "CsrClientConnectToServer"
> ......
> 
> 77FBE370 AddrOfNameOrdinalsBase 
> 77FBE370       dw      6,     7,     8,     9,   0Ah,   0Bh,   0Ch,   0Dh
> 77FBE370       dw    0Eh,   0Fh,   10h,   11h,   12h,   13h,   14h,   15h
> 77FBE370       dw    16h,   17h,   18h,   19h,   1Ah,   1Bh,   1Ch,   1Dh
> 77FBE370       dw    1Eh,   1Fh,   20h,   21h,   22h,   23h,   24h,   25h
> 77FBE370       dw    26h,   27h,   28h,   29h,   2Ah,   2Bh,   2Ch,   2Dh
> ......
> 
> * * *
> 
> 
> ```
> 
>     

  
代码首先是将函数名转换成序号，这件工作由函数 LdrpNameToOrdinal 完成，阅读代码时，请参照上面的输出表来理解。  

> ```
> 代码:
> 
> * * *
> 
> 77F8C157 RealGetFunctionAddress:
> 77F8C157       push edi                                    ; AddressOfNameOrdinals
> 77F8C158       push eax                                    ; AddressOfNames
> 77F8C159       push \[ebp+BaseAddress\]                      ; BaseAddress
> 77F8C15C       push \[esi+IMAGE\_EXPORT\_DIRECTORY.NumberOfNames\] ; NumberOfNames
> 77F8C15F       push \[ebp+lpProcName\]                       ; lpProcName
> 77F8C162       call \_LdrpNameToOrdinal@20                  ; LdrpNameToOrdinal(x,x,x,x,x)
> ......
> ......
> 
> 77F859A4 ; USHORT \_\_stdcall LdrpNameToOrdinal(ANSI\_STRING \*lpProcName,
>                                               ULONG NumberOfNames,       \\
>                                               ULONG BaseAddress,         \\
>                                               ULONG AddressOfNames,      \\
>                                               ULONG AddressOfNameOrdinals)
> 77F859A4 \_LdrpNameToOrdinal@20 proc near
> 77F859A4 Count = dword ptr -4
> 77F859A4 lpProcName= dword ptr  8
> 77F859A4 NumberOfNames= dword ptr  0Ch
> 77F859A4 BaseAddress= dword ptr  10h
> 77F859A4 AddressOfNames= dword ptr  14h
> 77F859A4 AddressOfNameOrdinals= dword ptr  18h
> 77F859A4
> 77F859A4       push ebp
> 77F859A5       mov ebp, esp
> 77F859A7       push ecx
> 77F859A8       mov eax, \[ebp+NumberOfNames\]
> 77F859AB       and \[ebp+Count\], 0
> 77F859AF       push ebx
> 77F859B0       push esi
> 77F859B1       lea ecx, \[eax-1\]              ; ecx = NumberOfNames - 1
> 77F859B4       push edi
> 77F859B5       test ecx, ecx
> 77F859B7       jl short loc\_77F85A0C         ; 检测 ECX 是否为零
> 77F859B9
> 77F859B9 Loop1:
> 77F859B9       mov eax, \[ebp+Count\]
> 77F859BC       mov edi, \[ebp+lpProcName\]
> 77F859BF       lea esi, \[eax+ecx\]            ; ecx = NumberOfName - 1
> 77F859BF                                     ; esi = Count + ecx
> 77F859C2       mov eax, \[ebp+AddressOfNames\] ; eax = AddressOfNamesBase
> 77F859C5       sar esi, 1                    ; 这段代码写的相当精彩，可歌可泣，可以非常快的查找函
> 77F859C5                                     ; 数所在位置，其先觉条件是函数名必须是严格的排序，包
> 77F859C5                                     ; 括大小写敏感。
> 77F859C5                                     ; 首先将函数的个数除以二，即从中间开始搜索。
> 77F859C5                                     ;
> 77F859C7       mov eax, \[eax+esi\*4\]          ; 在 AddressOfNames 偏移地址表中取函数名的偏移量
> 77F859CA       add eax, \[ebp+BaseAddress\]    ; 加上基地址得到函数名的地址
> 77F859CD
> 77F859CD Loop\_If\_dl\_IsNotTerminalChar:       ; 从函数名中取一个字符
> 77F859CD       mov bl, \[edi\]
> 77F859CF       mov dl, bl
> 77F859D1       cmp bl, \[eax\]                 ; 和函数名列表中的函数名在相应的位置上进行比较，特别
> 77F859D1                                     ; 需要注意，当两个字符进行比较时：
> 77F859D1                                     ;  if bl >= \[eax\]
> 77F859D1                                     ;     CF = 0          //这里将 CF 标志位置 0
> 77F859D1                                     ;  else               //bl < \[eax\]
> 77F859D1                                     ;     CF = 1
> 77F859D1                                     ;  end
> 77F859D1                                     ; 这里将 CF(进位标志) 置位，根据函数名排序的条件，即
> 77F859D1                                     ; 决定了是向前搜索，还是向后搜索：
> 77F859D1                                     ;   jnz short IfCharNotEqu
> 77F859D1                                     ; 不影响标志位，这个标志位将在后面的代码中使用
> 77F859D3       jnz short IfCharNoEqu
> 77F859D5       test dl, dl
> 77F859D7       jz short If\_dl\_IsTerminalChar
> 77F859D9       mov bl, \[edi+1\]               ; 取下一个字符
> 77F859DC       mov dl, bl
> 77F859DE       cmp bl, \[eax+1\]               ; 和下一个函数名字符进行比较，这里和上面的比较相同，
> 77F859DE                                     ; 通过设置 CF 标志位来决定搜索的方向。
> 77F859E1       jnz short IfCharNoEqu
> 77F859E3       inc edi                       ; 调整指针，指向下二个字符
> 77F859E4       inc edi
> 77F859E5       inc eax
> 77F859E6       inc eax
> 77F859E7       test dl, dl
> 77F859E9       jnz short Loop\_If\_dl\_IsNotTerminalChar
> 77F859EB
> 77F859EB If\_dl\_IsTerminalChar:
> 77F859EB       xor eax, eax
> 77F859ED       jmp short loc\_77F859F4
> 77F859EF
> 77F859EF IfCharNoEqu:                        ; 指令
> 77F859EF       sbb eax, eax                  ;   sbb eax, eax    //带进位减
> 77F859EF                                     ; 其结果只有两种可能
> 77F859EF                                     ;   if CF == 0
> 77F859EF                                     ;      eax = 0
> 77F859EF                                     ;      set CF = 0
> 77F859EF                                     ;   else        //CF = 1
> 77F859EF                                     ;      eax = 0xFFFFFFFF
> 77F859EF                                     ;      set CF = 1
> 77F859EF                                     ;   end;
> 77F859F1       sbb eax, 0FFFFFFFFh           ; 指令
> 77F859F1                                     ;   sbb eax, 0FFFFFFFFh    //带进位减
> 77F859F1                                     ; 其结果也是两种可能
> 77F859F1                                     ;   if  CF == 0
> 77F859F1                                     ;      eax = 0
> 77F859F1                                     ;      set SF = 0
> 77F859F1                                     ;   else         //CF = 1
> 77F859F1                                     ;      eax = 0xFFFFFFFF
> 77F859F1                                     ;      set SF = 1
> 77F859F1                                     ;   end
> 77F859F1                                     ; 这里 SF 是符号标志位，即判断结果是正数还是负数。
> 77F859F4
> 77F859F4 loc\_77F859F4:                       ; 指令
> 77F859F4       test eax, eax                 ;   test eax, eax
> 77F859F4                                     ; 一般我们用来测试某一位是 1 还是 0，但这个操作同样影
> 77F859F4                                     ; 响符号标志位：
> 77F859F4                                     ;   if eax >= 0
> 77F859F4                                     ;     set SF = 0
> 77F859F4                                     ;   else
> 77F859F4                                     ;     set SF = 1
> 77F859F4                                     ;   end
> 77F859F6       jge short ForewardSearch      ; Jump if Greater or Equal (SF=0)
> 77F859F8
> 77F859F8 BackwardSearch:
> 77F859F8       lea ecx, \[esi-1\]
> 77F859FB       jmp short NextSearch
> 77F859FD
> 77F859FD ForewardSearch:
> 77F859FD       jle short SearchEnd
> 77F859FF       lea eax, \[esi+1\]
> 77F85A02       mov \[ebp+Count\], eax
> 77F85A05
> 77F85A05 NextSearch:
> 77F85A05       cmp ecx, \[ebp+Count\]
> 77F85A08       jge short Loop1
> 77F85A0A       jmp short SearchEnd
> 77F85A0C
> 77F85A0C loc\_77F85A0C:
> 77F85A0C       mov esi, \[ebp+NumberOfNames\]
> 77F85A0F
> 77F85A0F SearchEnd:
> 77F85A0F       cmp ecx, \[ebp+Count\]
> 77F85A12       jge short GetOrdinal
> 77F85A14       or ax, 0FFFFh
> 77F85A18       jmp short Exit
> 77F85A1A
> 77F85A1A GetOrdinal:
> 77F85A1A       mov eax, \[ebp+AddressOfNameOrdinals\]
> 77F85A1D       mov ax, \[eax+esi\*2\]           ; 序号列表的单位是 WORD 所以 esi\*2 即得到序号的偏移
> 77F85A1D                                     ; 地址，加上基地址，即得到函数的序号。
> 77F85A21
> 77F85A21 Exit:
> 77F85A21       pop edi
> 77F85A22       pop esi
> 77F85A23       pop ebx
> 77F85A24       leave
> 77F85A25       retn 14h
> 77F85A25 \_LdrpNameToOrdinal@20 endp
> 
> * * *
> 
> 
> ```
> 
>     

  
在这里我对 LdrpNameToOrdinal 的汇编代码进行了详细的解释，个人总有种捞过界的感觉，这种内容应当属于汇编教材的范畴。可是下面又要捞过界了，不过也管不了那么多了。  
汇编语言的条件转移指令有几类，我只对其中两类容易混淆的加以说明：  

> ```
> 代码:
> 
> * * *
> 
>   jg  xxxx            //Jump short if greater (ZF=0 and SF=OF)
>   ja  xxxx            //Jump short if above (CF=0 and ZF=0)
>   
>   jl  xxxx            //Jump short if less (SF<>OF)
>   jb  xxxx            //Jump short if below (CF=1)
> 
> * * *
> 
> 
> ```
> 
>     

  
这里我们要讨论的其实就是这几个英文单词的含义，如果你能正确的理解这些单词，对你今后的汇编开发和阅读会有很多好处。  
首先我们看 greater 和 above，greater 是较大的意思，而 above 是在什么之上的意思，两者似乎非常类似，但从数学的角度看，却有着本质的区别。  
greater 是指一个有符号数的大小，例如：  
0x80000000 < 0x7FFFFFFF  
而 above 是指一个无符号数的大小，例如：  
0x80000000 > 0x7FFFFFFF  
同理，less 较小，而 below 是在什么之下，有了这样清醒的认识，将来在阅读和使用是就不容易犯错误。  
是不是捞过界了，说起英文单词了。  
  
下面我们接着说我们的代码分析，其实已经没有什么好说的，请看下面的代码：  

> ```
> 代码:
> 
> * * *
> 
> 77F8C167       movzx ecx, ax
> 77F8C16A       cmp ecx, \[esi+IMAGE\_EXPORT\_DIRECTORY.NumberOfFunctions\]
> 77F8C16D       jnb loc\_77F8C248
> 77F8C173       mov eax, \[esi+IMAGE\_EXPORT\_DIRECTORY.AddressOfFunctions\]
> 77F8C176       mov edx, \[ebp+BaseAddress\]
> 77F8C179       add eax, edx                  ; Offset + BaseAddress
> 77F8C17B       lea eax, \[eax+ecx\*4\]          ; Offset + Ordinal \* 4
> 77F8C17E       mov ecx, \[eax\]                ; OffsetFunction
> 77F8C180       add ecx, edx                  ; FunctionAddress = OffsetFunction + BaseAddress
> 77F8C182       mov edx, \[ebp+ProceAddr\]
> 77F8C185       cmp ecx, esi
> 77F8C187       mov \[edx\], ecx                ;将函数调用地址从变参 ProceAddr 返回(最后还要将这个数送给 EAX)
> 
> * * *
> 
> 
> ```
> 
>     

  
对于这段代码，你只要能正确的理解 IMAGE\_EXPORT\_DIRECTORY 结构的含义，读懂是非常容易的，最近看雪论坛上连续出了两篇重量级的关于 PE 文件头的翻译文章，各位不仿仔细读读，不过我老人家是绝对不读的，哈哈。  
  
看来今天又要告一段落，下一篇我们将谈如何打造这两个函数。  
  
后话：每次接受伊万大叔教导时总有一种 【即生谕，何生亮，无事生非】的感觉，你看象 LdrpSnapThunk = Get Function Address 这种东西都能想的出来，气死我也。下面我也出道题考考你，你这个鬼子不是总认为自己中文天下无敌吗？  
题目：周谕的老母是谁？诸葛亮的老母是谁？张非的老母是谁？哈哈，答案在上文寻找。  
如果答不出，天下父母皆安也，何解，“背对论”流产了，哈哈哈哈。  
另：下一篇将公布正确的答案。 

 4dd466d0e1e0cc123afbe504da2462ca
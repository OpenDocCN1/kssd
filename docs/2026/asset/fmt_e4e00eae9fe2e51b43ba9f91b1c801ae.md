
其中`top`中为 `0FFFFFFFFh`，表示此字符串已经初始化，也是字符串的标志。`Len`字符串的长度，单位是 `DWORD`，最后是一个不定长的字符串，以零结尾，但在Delphi中操作字符串，实际是以`Len`来实现，并不关心字符串中是否有 `0`，这个 `0` 是用来兼容 C 字符串的，如果要把Delphi的字符串当作 C 字符串来使用，这个 `0` 就不能乱放了。由于使用 `DWORD` 来定义字符串的长度，所以 Delphi 中字符串的操作远比我们想象的要来得强大，在 Delphi 中你可以将整个文件读入一个字符串中(包括一个可执行文件)，然后用字符串指针去随意操作，所有字符串的函数都可以使用，比使用流更方便。这就是被称为巨型字符串的东西了。字符串的长度在32位系统只和内存的大小有关，即 `DWORD` 的 4G 寻址空间，Borland 的工程师给出一个比较安全的数值，小于2G，防止字符串覆盖了老盖的操作系统，哈哈。

## 2.如果是序号

序号的处理比较简单，直接拿过来使用就完了。

```
77E80CD5 IfProcNameIsNull:

               ;下面是为调用 LdrGetProcedureAddress(x,x,x,x) 而设置入口参数

77E80CD5       lea eax, [ebp+lpProcName]

77E80CD8       push eax                            ; ProcedureAddress

77E80CD9       push edi                            ; Ordinal

77E80CDA       push 0                              ; Name
```

## 3.获取函数调用地址

作为系统操作，首要考虑的就是运行稳定、强壮，这里调用 `BasepMapModuleHandle` 基本就是起这个作用，如果打造自己的API 函数，就没有必要怎样复杂了。

```
77E80CDC GetProcAddress:                           ; flag

77E80CDC       push 0

77E80CDE       push [ebp+hModule]                  ; hModule

77E80CE1       call _BasepMapModuleHandle@8        ; 检查 hModule 的合法性

77E80CE6       push eax                            ; BaseAddress

77E80CE7       call _LdrGetProcedureAddress@16     ; LdrGetProcedureAddress(x,x,x,x)

77E80CEC       test eax, eax

77E80CEE       jl short IfFunAddrEquZero

77E80CF0       push 0                              ; flag

77E80CF2       push [ebp+hModule]                  ; hModule

77E80CF5       call _BasepMapModuleHandle@8        ; 检查函数调用地址的合法性

77E80CFA       cmp [ebp+lpProcName], eax

77E80CFD       jnz short IfFunAddrIsOk

77E80CFF       cmp ebx, edi

77E80D01       sbb eax, eax

77E80D03       neg eax

77E80D05       add eax, STATUS_ORDINAL_NOT_FOUND

77E80D0A IfFunAddrEquZero:                         ; Status

77E80D0A       push eax

77E80D0B       call _BaseSetLastNTError@4          ; BaseSetLastNTError(x)

77E80D10       xor eax, eax

77E80D12       jmp short Exit
```

不管`lpProcName`变参是函数名还是序号，最后都是调用位于`ntdll.dll`中的 `LdrGetProcedureAddress`，这个函数的过程也是很复杂的，但主要工作就是通过输出表来获得函数调用地址，我们只列出相应部分的代码，这部分代码在一个叫着 `LdrpSnapThunk` 的子过程中。

我不知大家对这个函数名有什么看法，你能从函数的命名上猜出这个函数做了 Get Function Address 的工作吗？如果你能，你的智商肯定超过爱因斯坦，将来没准给我们弄出一个什么“背对论”来。昨天为了这个函数名，我请教了我们那位密码语言天才伊万大叔。将我对 `LdrSnapThunk` 的译文“调遣一部带有巨大象鼻吸盘新型设备，用外形似象鼻子的那么一个吸盘吸住一根将要打到树上的大铁棍”，告诉伊万大叔，多么富有诗意的名字。当我正在为我伟大的翻译和理解而沾沾自喜的时候，伊万大叔大吼一声：“错，你知道老盖生活在什么地方吗？”，我想，我还没有愚蠢到连老盖先生住在那里都不知道的地步。伊万大叔接着说：“你知道那里是什么文化最出名吗？，那就是多元文化，写这段程序的先生没准来自于爪哇(Java)国，你知道 Java 吗？”，我答：“这个知道，不就是能让很多黄色下流的东西流入每个家庭的一种下流语言吗”。伊万大叔微笑道：“孺子可教也，在爪哇国，`LdrpSnapThunk` 的含义就是 Get Function Address，明白了吗，就象我弄一个俄文名字，然后用英文对应的字母写出，你能看懂吗，哈哈”。哇赛!，天才就是不同，连这都解的出，可惜天下父母要倒霉了，“背对论”所要研究的就是当一个男孩和一个女孩背对背靠在一起是，时空所发生的变化。

马掌钉在膝盖上，唉! 离蹄(题)太远了，打住。

下面我们来看看这个“象鼻子”是怎么吸到函数调用地址的：(这个函数的调用参数都是我瞎掰的，我不知道到那里去找这个调用参数)

```
77F8C0B8 ; NTSTATUS __stdcall LdrpSnapThunk(PVOID BaseAddress,        \
                                            ANSI_STRING *lpProcName,  \
                                            ULONG Ordinal,            \
                                            PVOID *ProceAddr,         \
                                            PVOID ExportTabAddr,      \
                                            int,                      \
                                            BOOLEAN ShowNotFound,     \
                                            int Unknown)

77F8C0B8 _LdrpSnapThunk@32 proc near

77F8C0B8 var_24= dword ptr -24h
77F8C0B8 var_20= dword ptr -20h
77F8C0B8 var_18= dword ptr -18h
77F8C0B8 var_10= dword ptr -10h
77F8C0B8 ProcName= STRING ptr -8
77F8C0B8 BaseAddress= dword ptr  8
77F8C0B8 lpProcName= dword ptr  0Ch
77F8C0B8 Ordinal= dword ptr  10h
77F8C0B8 ProceAddr= dword ptr  14h
77F8C0B8 ExportTabAddr= dword ptr  18h
77F8C0B8 arg_14= dword ptr  1Ch
77F8C0B8 ShowNotFound= byte ptr  20h
77F8C0B8 Unknown= dword ptr  24h

77F8C0B8       push ebp
77F8C0B9       mov ebp, esp
77F8C0BB       sub esp, 24h
77F8C0BE       mov eax, [ebp+Ordinal]
......
......
77F8C157 RealGetFunctionAddress:

77F8C157       push edi                                    ; AddressOfNameOrdinals
77F8C158       push eax                                    ; AddressOfNames
77F8C159       push [ebp+BaseAddress]                      ; BaseAddress
77F8C15C       push [esi+IMAGE_EXPORT_DIRECTORY.NumberOfNames] ; NumberOfNames
77F8C15F       push [ebp+lpProcName]                       ; lpProcName
77F8C162       call _LdrpNameToOrdinal@20                  ; LdrpNameToOrdinal(x,x,x,x,x)
77F8C167 loc_77F8C167:

77F8C167       movzx ecx, ax
77F8C16A       cmp ecx, [esi+IMAGE_EXPORT_DIRECTORY.NumberOfFunctions]
77F8C16D       jnb loc_77F8C248
77F8C173       mov eax, [esi+IMAGE_EXPORT_DIRECTORY.AddressOfFunctions]
77F8C176       mov edx, [ebp+BaseAddress]
77F8C179       add eax, edx                                ; Offset + BaseAddress
77F8C17B       lea eax, [eax+ecx*4]                        ; Offset + Ordinal * 4
77F8C17E       mov ecx, [eax]                              ; OffsetFunction
77F8C180       add ecx, edx                                ; FunctionAddress = OffsetFunction + BaseAddress
77F8C182       mov edx, [ebp+ProceAddr]
77F8C185       cmp ecx, esi
77F8C187       mov [edx], ecx
77F8C189       jbe loc_77F8C243
......
......
```

下面我们弄一个PE头来和Export表来看看：

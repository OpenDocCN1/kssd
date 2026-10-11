
# 【成果 6.4】软件保护壳技术专题 - 加密资源节

> 原文：[`www.pediy.com/kssd/pediy10/68262.html`](https://www.pediy.com/kssd/pediy10/68262.html)

什么是`WINDOWS`的资源这里就不提了。写过`WIN`程序的人应该都晓得了。如果没写过，那么。。。建议去写哈。

关于加密资源节，也是加壳过程中的一个可选项而已。加和不加都不太对破解造成影响。但是如果你的程序中，有一些重要数据在资源节里面而你又不想让其他人通过资源浏览器参看到，那么这个时候加密资源节就有必要了。在这里节，我们首先了解一下资源节在`PE`文件中的结构。其次，编写一个简单的资源浏览器用于熟悉资源节的结构。最后，完成一个加密与解密资源节的小程序。

资源节可以说在`PE`结构中最复杂的一个结构了。整体是一个`2`叉树结构。

总共有`3`层。第一层是类型，第二层是名称，第三层是语言，在语言后面就是一个指向资源数据的位置的结构了。

下面将资源节的结构如图:

资源节结构图:

附件中查看 呵呵。。。

资源节处于数据目录的第三项可以通过微软预定义的宏`IMAGE_DIRECTORY_ENTRY_RESOURCE`来定位资源节的数据目录。通过`VirtualAddress`转换文件偏移 + 文件映射地址就得到了资源节所在文件的位置

节内容的起始是一个被称为资源目录表的结构，如下：

```
IMAGE_RESOURCE_DIRECTORY STRUCT
    Characteristics dd       ?
    TimeDateStamp dd        ?
    MajorVersion dw         ?
    MinorVersion dw         ?
    NumberOfNamedEntries dw ?
    NumberOfIdEntries dw    ?
IMAGE_RESOURCE_DIRECTORY ENDS
```

`Characteristics`: 资源标志，保留，设置为`0`

`TimeDateStamp`: 时间戳，由资源编译器设定

`MajorVersion`: 主版本号，由用户设定

`MinorVersion`: 次版本号，由用户设定

以下这两项是我们说要关心的

`NumberOfNamedEntries`: 通过字符串来定义的资源

`NumberOfIdEntries`: 用过序号来定义的资源

这两项的和为引入资源的总数，在这个结构后面是一组由`IMAGE_RESOURCE_DIRECTORY_ENTRY`组成的数组

具体个数为`NumberOfNamedEntries + NumberOfIdEntries`

以下为结构的说明:

```
IMAGE_RESOURCE_DIRECTORY_ENTRY STRUCT
    union
        rName    RECORD NameIsString:1,NameOffset:31
        Name1 dd ?
        Id dw ?
    ends
    union
        OffsetToData dd ?
          rDirectory    RECORD DataIsDirectory:1,OffsetToDirectory:31
    ends
IMAGE_RESOURCE_DIRECTORY_ENTRY ENDS
```

这个结构在不同的位置是不同的意义。

在第一层的为类型，`Name1`代表的是一个类型，在资源类型方面有`16`个基本类型

```
"RT_CURSOR"
"RT_BITMAP"
"RT_ICON"
"RT_MENU"
"RT_DIALOG"
"RT_STRING"
"RT_FONTDIR"
"RT_FONT"
"RT_ACCELERATOR"
"RT_RCDATA"
"DIFFERENCE"
"RT_GROUP_CURSOR"
"RT_GROUP_ICON"
"RT_VERSION"
```

也有更多的类型，这方面的资料很少。我也没有能收集全，`RT_RCDATA`类型表示为自定义的。

这里为标准的类型值。

在第二层时：如果`Name`的`2`进制最高位时候`1`, 那么它表示一个以名字引出的资源，此时`Name`的低`2`位表示一个从资源节开头到它的偏移

使用`Name`的低`2`位值 + 资源节开头的地址 指向一个`IMAGE_RESOURCE_DIRECTORY_STRING`结构。资源的名字使用`UNICODE`编码。

```
IMAGE_RESOURCE_DIRECTORY_STRING STRUCT
    Length1 dw      ?
    NameString db   ?
IMAGE_RESOURCE_DIRECTORY_STRING ENDS
```

`Length1`表示有几个字母

`NameString`为以双`0`结尾的`UNICODE`字符串。

在第三层时，`Name`表示资源所用语言的代码。

`OffsetToData`：这个字段在不同层都表示一个到资源节开头的偏移。

不同的是，如果`OffsetToData`的`2`进制最高位是`1`, 表示还有下面的其他层(语言层或者名字层，类型层总是存在)。

如果是`0`, 则表示一个到`IMAGE_RESOURCE_DATA_ENTRY`的偏移

```
IMAGE_RESOURCE_DATA_ENTRY STRUCT
    OffsetToData dd ?
    Size1 dd        ?
    CodePage dd     ?
    Reserved dd     ?
IMAGE_RESOURCE_DATA_ENTRY ENDS
```

`OffsetToData`表示到资源数据的偏移，这个偏移是到加载地址(`ImageBase`)的偏移。

`Size1`为数据的长度

`CodePage`: 一般为`0`

`Reserved`: 一般为`0`

接下来看段代码可能会更清晰一些。

代码:

```
ResViewEntry proc uses ebx ecx edx edi esi pEntry : LPVOID, dwLevel : DWORD
    LOCAL szOutFileName[MAX_PATH] : BYTE
;; level = 0,1,2

    mov esi, pEntry
    assume esi : ptr IMAGE_RESOURCE_DIRECTORY
    ;; 计算有几个资源在当前目录下
    mov cx, word ptr [esi].NumberOfNamedEntries
    add cx, word ptr [esi].NumberOfIdEntries
    movzx ecx, cx
    ;; 这里存在着 ecx 个 IMAGE_RESOURCE_DIRECTORY_ENTRY
    add esi, sizeof IMAGE_RESOURCE_DIRECTORY
    mov edi, esi
    assume edi : ptr IMAGE_RESOURCE_DIRECTORY_ENTRY
ResDirEntryLoop:
    ;; 打印资源层
    invoke JudegeResourceEntryLevel, edi, dwLevel
    mov eax, dword ptr [edi].OffsetToData
    test eax, FIRSTBIT
    jz FoundTheDataEntry
    and eax, 0000FFFFh
    add eax, g_pResSection
    mov edx, dwLevel
    inc edx
    invoke ResViewEntry, eax, edx
    jmp ContinueResDirEntryLoop
    FoundTheDataEntry:
    ;; 取出资源文件
    add eax, g_pResSection
    assume eax : ptr IMAGE_RESOURCE_DATA_ENTRY
    mov edx, dword ptr [eax].Size1
    mov eax, dword ptr [eax].OffsetToData
    ;; 转化 RVA 到偏移
    invoke RVA2Offset, g_pMem, eax
    push eax
    push ecx
    push edx
    invoke wsprintf, addr szOutFileName, offset g_szOutOrdFormat, eax
    pop edx
    pop ecx
    pop eax
    add eax, g_pMem
    invoke OutputToFile, addr szOutFileName, eax, edx
    ContinueResDirEntryLoop:
    add edi, sizeof IMAGE_RESOURCE_DIRECTORY_ENTRY       
    dec ecx
    test ecx, ecx
    jnz ResDirEntryLoop
     

     
    assume esi : nothing
    assume edi : nothing
    ret

ResViewEntry endp

JudegeResourceEntryLevel proc uses ebx ecx edx edi esi pResEntry : LPVOID, dwLevel : DWORD
    LOCAL szTmpBuf[MAX_PATH] : BYTE
     
    ;; init local variable
    mov ecx, MAX_PATH
    lea edi, szTmpBuf
    xor eax, eax
    cld
    rep stosb
     
    mov edi, pResEntry
    assume edi : ptr IMAGE_RESOURCE_DIRECTORY_ENTRY
    mov eax, dwLevel
       
    cmp eax, 0
    jnz JudgeLevelOne
    ;; 显示类型
    mov eax, dword ptr [edi].Name1
    invoke ShowResourceType, eax
    jmp ExitJudgeLevel
JudgeLevelOne:
    cmp eax, 1
    jnz JudgeLevelTwo
    ;; 显示名称
    mov eax, dword ptr [edi].Name1
    ;; 最高位是 0 时表示使用 ID
    test eax, FIRSTBIT
    jz UseID
    and eax, 0000FFFFh
    add eax, g_pResSection
    assume eax : ptr IMAGE_RESOURCE_DIR_STRING_U
    lea esi, [eax].NameString
    lea edi, szTmpBuf
    ;; 拷贝字符串到局部变量
    mov cx, word ptr [eax].Length1
    movzx ecx, cx
    cld
    rep movsw
    invoke crt_printf, offset g_szOutFormat, offset g_szResNameByStr
    invoke crt_wprintf, addr szTmpBuf
    invoke crt_printf, offset g_szOutFormat, offset g_szEndLine
    jmp ExitJudgeLevel                
    UseID:
    invoke crt_printf, offset g_szResNameById, eax    
    jmp ExitJudgeLevel
JudgeLevelTwo:
    ;; 第三次是语言,这里输出语言定义的常量
    mov eax, dword ptr [edi].Name1
    invoke crt_printf, offset g_szResLang, eax
ExitJudgeLevel:
    assume eax : nothing
    assume edi : nothing
    ret

JudegeResourceEntryLevel endp
```

很简单对吧!。。。 这段在附件中的程序打印资源的类型，名称/ID，语言代码，还有将资源 dump 到一个目录中。

接下来就改讲解一下加解密资源节了，其实与列出资源节的算法一样。这里使用的加密算法也很简单只是简单的`XOR`了一哈。

不过要注意的是。有些资源是不可以加密的。例如`ICON`还有寻找`ICON`偏移的`ICON_GROUP`还有版本信息等，还有其他的，总之在`PE`加载器在代码段之前加载的资源是不可以加密，否则程序不能运行。这里的加密程序不加密`ICON`、`ICON_GROUP`、`VERSION`这三种类型，如果你遇到加密资源后程序无法运行那可以就是加了不该加的资源了。呵呵，自行修改下代码吧。加密在`JudegeResourceType`处添加过滤，解密在`JudgeDecryptResType`处修改，代码很简单一看就明白

下面列出加解密代码

首先是加密的

代码:

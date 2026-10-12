
# 软件保护壳专题 - 处理引出表

> 原文：[`www.pediy.com/kssd/pediy10/80406.html`](https://www.pediy.com/kssd/pediy10/80406.html)

这节与后面的给感染驱动都是新添加的章节,并且把第二部分的关于变形方面的章节合成为一节 "变形引擎的构建"。变形引擎可能要慢些。争取打造一份实用的东西出来。过了年在写，呵呵。这节主要讲解输出表的结构,输出表的 HOOK,并实现了一个 HOOK 输出的小程序。

### 目录

1. 什么是输出表
2. 输出表的结构
3. 输出表的 HOOK
4. 分析 ExportHandler 代码分析

### 正文

#### 1. 什么是输出表

用于定制一些函数,并提供给其他程序使用,建立一个函数与调用地址的映射。这种表一般存在与 DLL 文件当中。EXE 文件中少有,不过不是不可以存在与 EXE 文件中。

#### 2. 输出表的结构

首先说明一下输出表的定位,输出表的目录存在于 `PE`->可选头->数据目录的第一项中。可以通过此目录获取输出表的 `RVA` 以及输出表的长度。通过 `RVA` 获取到它在文件中的偏移。读取偏移的第一个结构为输出目录表,如下:

```
IMAGE_EXPORT_DIRECTORY STRUCT
  ;; 总是为 0
  Characteristics         DWORD       ?
  ;; 文件创建的时间
  TimeDateStamp           DWORD       ?
  ;; 主版本号,一般为 0
  MajorVersion            WORD        ?
  ;; 次版本号,一般为 0
  MinorVersion            WORD        ?
  ;; DLL 名称的 RVA
  nName                   DWORD       ?
  ;; 序号基值, 一般为 0
  nBase                   DWORD       ?
  ;; 输出函数的数量
  NumberOfFunctions       DWORD       ?
  ;; 以名字输出的数量
  NumberOfNames           DWORD       ?
  ;; 地址表的 RVA
  AddressOfFunctions      DWORD       ?
  ;; 名字表的 RVA
  AddressOfNames          DWORD       ?
  ;; 名字序号表的 RVA
  AddressOfNameOrdinals   DWORD       ?
IMAGE_EXPORT_DIRECTORY ENDS
```

这里我们详细解释一下这个结构,输出表的结构很简单.

`Characteristics`;`TimeDateStamp`;`MajorVersion`;`MinorVersion` 都不重要,这里就不解释了。

`nName`:标示这个 `DLL` 的名字,通过这个值获取到文件的偏移可以看到是一个形如 `xxx.dll` 的字符串

`nBase`:这个值大多数情况下为 `1`,他的作用是,如果通过序数引出的函数必须减去这个值后再通过地址表取出地址.

`NumberOfFunctions`:总共输出表函数的数量.

`NumberOfNames`:以名字输出函数的数量.

`AddressOfFunctions`:这个值就是地址表的 `RVA` 了,是一个 `4` 字节的数组。

`AddressOfNames`:这个值的指向一个存放函数名字 `RVA` 的数组。通过对它的遍历输出所有函数。

`AddressOfNameOrdinals`:这个值指向一个存放名字索引以 `2` 字节为单位的数组。此表与 `AddressOfNames` 表同步使用.当从索引为 `3` 的 `AddressOfNames` 中对比名字,并从 `3` 的 `AddressOfNameOrdinals` 中获取序号 `X`,在从 `AddressOfFunction[X]` 取出此函数的地址.

#### 3. 输出表的 HOOK

输出表的 `HOOK` 很简单.只需要把位于 `AddressOfFunctions` 对应的 `RVA` 替换为你自己函数的 `RVA` 就可以了。运行完毕后直接记的 `JMP` 回去,不过也可以不用...

#### 4. 分析 ExportHandler 代码分析

下面分析一下代码了.程序毕竟是工程还是写出点程序才实在.这个小工具使用很简单就一条命令

`usage:ExportHandle dllname <scriptname>`

当 `ExportHandler dllname` 的时候会遍历函数名字表并且打印所有输出表的函数

当 `ExportHandler dllname scriptname` 的时候会对前面的 `DLL` 使用后面一个简单的配置脚本进行 `HOOK`.脚本大体内容如下:

```
// export handlder script
// 一个简单的配置脚本

// 新节属性
$section_size = 1024;
$section_name = logic;
$section_characteristics = W|R|X;

// 新函数
$hostname = 53 55 56 57 8B F9;
```

这里做一下简单的解释

`//` 是注释

`$section_size`,`$section_name`,`$section_characteristics` 是这个脚本专有的变量名对应的新节大小,名称,还有属性.`W`=可写.`R`=可读.`X`=可执行.

`$hostname = 53 55 56 57 8B F9;` 是这样的 `hostname` 是这个 `DLL` 或者 `EXE` 的一个输出函数的名字后面是要 `HOOK` 新函数的 `SHELLCODE`. 这个小程序会添加一个新节并且把这些 `SHELLCODE` 记录到新节并在后添加 `JMP 原函数的偏移` 这样的语句. 然后修改输出表的地址。可以添加多个这样的变量 `HOOK` 不同的函数.要说明的是这个脚本的 `SHELLCODE` 必须是大写,而且读脚本也没有做什么错误处理(汇编不太适合做词法分析,写起来太麻烦了)

这里解释几个程序中重要的函数.

显示输出函数名字

```
ShowExportTable proc uses ebx ecx edx esi edi, szFilePath : LPSTR
  LOCAL pMem : LPVOID
  LOCAL hFile : HANDLE
  LOCAL hMap : HANDLE
  LOCAL PeHeader : LPVOID
  LOCAL pName : LPVOID
  LOCAL dwRVA : DWORD
  LOCAL dwOffset : DWORD
  LOCAL dwCount : DWORD
  ;; 映射文件
  invoke MapFile2MemNotClose, szFilePath, 0, addr hFile, addr hMap, addr pMem, 1
  mov esi, pMem
        ;; 获取 PE 头
  add esi, dword ptr [esi+03ch]
  assume esi : ptr IMAGE_NT_HEADERS
  mov PeHeader, esi
        ;; 获取输出表的位置
  lea edi, [esi].OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT * sizeof IMAGE_DATA_DIRECTORY]
  assume edi : ptr IMAGE_DATA_DIRECTORY
        ;; 确定输出表是否存在
  cmp [edi].VirtualAddress, 0
  jz Err_NoExportTable
  mov eax, dword ptr [edi].VirtualAddress
  invoke RVA2Offset, pMem, eax
  add eax, pMem
  xchg eax, edi
        ;; 打印输出表目录
  assume edi : ptr IMAGE_EXPORT_DIRECTORY
  invoke crt_printf, offset g_szOutFormat, offset g_szExportDirectory
  mov eax, dword ptr [edi].Characteristics
  invoke crt_printf, offset g_szCharacteristics, eax
  mov eax, dword ptr [edi].TimeDateStamp
  invoke crt_printf, offset g_szTimeDateStamp, eax
  movzx eax, word ptr [edi].MajorVersion
  invoke crt_printf, offset g_szMajorVersion, eax
  movzx eax, word ptr [edi].MinorVersion
  invoke crt_printf, offset g_szMinorVersion, eax
  mov eax, dword ptr [edi].nName
  invoke crt_printf, offset g_szName, eax
  mov eax, dword ptr [edi].nBase
  invoke crt_printf, offset g_szBase, eax
  mov eax, dword ptr [edi].NumberOfFunctions
  invoke crt_printf, offset g_szNumberOfFunctions, eax
  mov eax, dword ptr [edi].NumberOfNames
  invoke crt_printf, offset g_szNumberOfNames, eax
  mov eax, dword ptr [edi].AddressOfFunctions
  invoke crt_printf, offset g_szAddressOfFunctions, eax
  mov eax, dword ptr [edi].AddressOfNames
  invoke crt_printf, offset g_szAddressOfNames, eax
  mov eax, dword ptr [edi].AddressOfNameOrdinals
  invoke crt_printf, offset g_szAddressOfNameOrdinals, eax
  invoke crt_printf, offset g_szOutLine
  mov eax, dword ptr [edi].nName
  invoke RVA2Offset, pMem, eax
  add eax, pMem
  invoke crt_printf, offset g_szDllName, eax 
  invoke crt_printf, offset g_szListFunName
        ;; 遍历输出函数名字表 
  mov ecx, dword ptr [edi].NumberOfNames
  mov edx, dword ptr [edi].AddressOfFunctions
  invoke RVA2Offset, pMem, edx
  add eax, pMem
  mov edx, eax
        ;; ebx = 输出名字表在文件中的位置
        ;; edx = 输出函数表在文件中的位置
        ;; esi = 输出名字表索引表在文件中的位置
        ;; ecx = 输出名字表的个数 
  mov ebx, dword ptr [edi].AddressOfNames
  invoke RVA2Offset, pMem, ebx
  add eax, pMem
  mov ebx, eax
  mov esi, dword ptr [edi].AddressOfNameOrdinals
  invoke RVA2Offset, pMem, esi
  add eax, pMem
  mov esi, eax
  ;; zero ?
  test ecx, ecx
  jz Exit_ShowExportTable
  xor eax, eax
  mov dwCount, eax
   
Loop_NameTable:
        ;; 获取函数名称
  mov eax, dword ptr [ebx]
  invoke RVA2Offset, pMem, eax
  add eax, pMem
  mov pName, eax
        ;; 从序号表中获取名称的序号
  push esi
  mov eax, dwCount
  imul eax, eax, 02h
  add esi, eax
  movzx eax, word ptr [esi]
  imul eax, eax, 04h
  mov esi, edx
  add esi, eax
  mov eax, dword ptr [esi]
        ;; 从地址表中获取函数的 RVA
  mov dwRVA, eax
  invoke RVA2Offset, pMem, eax
  mov dwOffset, eax
  pop esi
  ;; printf result
  push ecx
  push edx
        ;; 打印结果
  invoke crt_printf, offset g_szFunInfo, pName, dwRVA, dwOffset
  pop edx
  pop ecx
   
  add ebx, 04h
  inc dwCount
  loop Loop_NameTable

Exit_ShowExportTable:
  invoke FreeFileMemory, addr hFile, addr hMap, addr pMem
  assume esi : nothing
  assume edi : nothing
  ret
Err_NoExportTable:
  lea eax, g_szNoExportTbl
  invoke crt_printf, offset g_szOutFormat, eax
  xor eax, eax
  jmp Exit_ShowExportTable
ShowExportTable endp
```

HOOK 输出函数

```
FixExportFunction proc uses ebx ecx edx esi edi, pMem : LPVOID, pShellCodeTo : LPVOID, pNewSectionRVA : LPVOID
  LOCAL pExportFunctionPointer : LPVOID
  LOCAL dwExportFunctionRVA : DWORD
  LOCAL dwShellCodeRVA : DWORD
   
  ;; 获取新节在文件中的位置
  mov edi, pNewSectionRVA
  invoke RVA2Offset, pMem, edi
  add eax, pMem
  mov edx, eax
   
  mov ebx, pShellCodeTo
  assume ebx : ptr SHELLCODETO
   
  ;; 查找要 HOOK 函数的 RVA
  lea eax, [ebx].szFunctionName
  invoke FindExportFunction, pMem, eax
  test eax, eax
  jz Error_FixExportFunction
  mov pExportFunctionPointer, eax
  mov eax, dword ptr [eax]
  mov dwExportFunctionRVA, eax
   
  ;; 获取 shellcode 的 RVA
  mov eax, pNewSectionRVA
  add eax, dword ptr [ebx].dwShellCodeOffset
  mov dwShellCodeRVA, eax
  mov eax, pExportFunctionPointer
  push dwShellCodeRVA
  pop dword ptr [eax]
   
  ;; 制作 SHELLCODE,在末尾添加 jmp 原函数偏移的 5 字节指令
  invoke MakeShellCode, ebx, dwExportFunctionRVA, dwShellCodeRVA
   
  ;; 写入 shellcode
  mov edi, edx
  add edi, dword ptr [ebx].dwShellCodeOffset
  lea esi, [ebx].pShellCode
  mov ecx, dword ptr [ebx].dwShellCodeSize
  cld
  push edi
  rep movsb
  pop eax    ; set ret value -> shellcode location in file
Exit_FixExportFunction:
  assume ebx : nothing
  ret
Error_FixExportFunction:
  xor eax, eax
  jmp Exit_FixExportFunction
FixExportFunction endp
```

制作 shellcode

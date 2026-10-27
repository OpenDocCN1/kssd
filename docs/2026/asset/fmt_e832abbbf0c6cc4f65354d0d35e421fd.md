
注释：

| 字段值 | 用途 |
| --- | --- |
| 00000020h | 包含代码，常与10000000h一起设置 |
| 00000040h | 包含已初始化数据 |
| 00000080h | 包含未初始化数据 |
| 02000000h | 可以被丢弃 |
| 10000000h | 共享块 |
| 20000000h | 可执行 |
| 40000000h | 可读 |
| 80000000h | 可写 |

# 4 块

## 4-1 `.text`

此区段是一段汇编代码的16进制形式，功能是弹出一个`MessageBox`提示框。

-->HEX

```
6A 00 68 00 30 40 00 68  07 30 40 00 6A 00 E8 07
00 00 00 6A 00 E8 06 00  00 00 FF 25 08 20 40 00
FF 25 00 20 40 00
```

注释：这是下面汇编指令的16进制模式。

-->ASM

```
00401000  6A 00         PUSH 0
00401002  68 00304000   PUSH first_PE.00403000
00401007  68 07304000   PUSH first_PE.00403007
0040100C  6A 00         PUSH 0
0040100E  E8 07000000   CALL <JMP.&user32.MessageBoxA>
00401013  6A 00         PUSH 0
00401015  E8 06000000   CALL <JMP.&kernel32.ExitProcess>
0040101A  FF25 08204000 JMP DWORD PTR DS:[<&user32.MessageBoxA>]
00401020  FF25 00204000 JMP DWORD PTR DS:[<&kernel32.ExitProcess>]
```

## 4-2 `.rdata`

该区块包含输入表。

### 4-2-1 `IMAGE_THUNK_DATA32` (IAT 1)

注释：这里有很多人都搞不明白，其实`IMAGE_THUNK_DATA32`是一个联合体，可以同时代表`IAT`与`INT`。

- `AddressOfData` [`DWORD`] --> `76 20 00 00`（指向`IMAGE_IMPORT_BY_NAME`的RVA）

注释：作为`IAT`时我们就会使用他的`AddressOfData`成员，用来存放指向`IMAGE_IMPORT_BY_NAME`的RVA，当程序装入内存后，只与`IAT`交换信息，输入表的其他部分就不再需要了。

【由于本例子两个API函数引自与两个不同的DLL文件，所以要补00000000h结束。】

注释：这里同样有很多人不明白，每当属于某一个DLL文件的API罗列完毕后，就要输入00加以结束。

### 4-2-3 `IMAGE_IMPORT_DESCRIPTOR` (IID 1)

注释：这里稍微复杂些，它的作用就是使用`INT`指定某个DLL文件中的API函数，并配合`IAT`指向相关API的地址。

- `OriginalFirstThunk` [`DWORD`] --> `4C 20 00 00`（指向输入名称表`INT`的RVA）

注释：这里指定某个系统DLL文件中的API函数。

- `Name` [`DWORD`] --> `6A 20 00 00`（指向DLL名字的RVA与指针）

注释：这里指定某个系统DLL文件。

- `FirstThunk` [`DWORD`] --> `08 20 00 00`（指向输入地址表`IAT`的RVA）

注释：这里指定相关的`IAT`，并由`IAT`在`IMAGE_IMPORT_BY_NAME`中获得相应API的地址。

### 4-2-4 `IMAGE_IMPORT_DESCRIPTOR` (IID 2)

- `OriginalFirstThunk` [`DWORD`] --> `54 20 00 00`（指向输入名称表`INT`的RVA）
- `Name` [`DWORD`] --> `84 20 00 00`（指向DLL名字的RVA与指针）
- `FirstThunk` [`DWORD`] --> `00 20 00 00`（指向输入地址表`IAT`的RVA）

【填充20个00h空字节做结尾标记】

注释：关于填充20个字节作为`IID`的结尾标记只是一个规律，还没找到相关资料证实。

### 4-2-5 `IMAGE_THUNK_DATA32` (INT 1)

- `ForwarderString` [`DWORD`] --> `5C 20 00 00`（指向一个转向字符的RVA）

注释：直接指向相关API函数。

【由于本例子两个API函数引自与两个不同的DLL文件，所以要补00000000h结束。】

注释：与上面的`IAT`原理一样，就是这样的一个格式。

### 4-2-7 `IMAGE_IMPORT_BY_NAME` (1)

- `Name` [`BYTE`] --> `4D 65 73 73 61 67 65 42 6F 78 41`（`MessageBoxA`的16进制码）

注释：相关API函数的16进制代码。

【后跟输出此函数的DLL名称16进制码00 75 73 65 72 33 32 2E 64 6C 6C 00 00 user32.dll】

注释：相关系统的API函数罗列完毕后，通常都在最后一个API后空00h，并跟着这个DLL名称的16进制数据。

总结：这里罗列了一些需要手工构建的部分结构，如果想看更加详细的请关注下面的内容。

# 三、PE文件结构字段清单

## 1 DOS头部

### 1-1 DOS Header

- `e_magic` [`WORD`] --> `4D 5A`（DOS可执行文件头标记）
- `e_cblp` [`WORD`] -> `00 00`（文件最后页的字节数）
- `e_cp` [`WORD`] -> `00 00`（文件页数）
- `e_crlc` [`WORD`] -> `00 00`（重定位元素个数）
- `e_cparhdr` [`WORD`] -> `00 00`（以段落为单位的头部大小）
- `e_minalloc` [`WORD`] -> `00 00`（所需的最小附加段）
- `e_maxalloc` [`WORD`] -> `00 00`（所需的最大附加段）
- `e_ss` [`WORD`] -> `00 00`（初始的堆栈段(SS)相对偏移量值）
- `e_sp` [`WORD`] -> `00 00`（初始的堆栈指针(SP)值）
- `e_csum` [`WORD`] -> `00 00`（校验和）
- `e_ip` [`WORD`] -> `00 00`（初始的指令指针(IP)值）
- `e_cs` [`WORD`] -> `00 00`（初始的代码段(CS)相对偏移量值）
- `e_lfarlc` [`WORD`] -> `00 00`（重定位表在文件中的偏移地址）
- `e_ovno` [`WORD`] -> `00 00`（覆盖号）
- `e_res` [`WORD`] 4 dup -> `00 00`（保留字，一般都是为确保对齐而预留）
- `e_oemid` [`WORD`] -> `00 00`（OEM 标识符，相对于 `e_oeminfo`）
- `e_oeminfo` [`WORD`] -> `00 00`（OEM 信息，即 `e_oemid` 的细节）
- `e_res2` [`WORD`] 10 dup -> `00 00`（保留字，一般都是为确保对齐而预留）
- `e_lfanew` [`DWORD`] --> `B0 00 00 00`（指向PE文件头的偏移量。0xB0=64+112）

### 1-2 DOS Stub

全部填00h。

## 2 PE文件头

### 2-1 "PE"00

- `Signature` [`DWORD`] --> `50450000h`（PE文件头标记）

### 2-2 `IMAGE_FILE_HEADER`

- `Machine` [`WORD`] --> `4C 01`（可执行文件的目标CPU类型）
- `NumberOfSections` [`WORD`] --> `03 00`（区块数目）
- `TimeDateStamp` [`DWORD`] -> `00 00 00 00`（文件创建的时间与日期）
- `PointerToSymbolTable` [`DWORD`] -> `00 00 00 00`（指向符号表，用于调试）
- `NumberOfSymbols` [`DWORD`] -> `00 00 00 00`（符号表中的符号个数，用于调试）
- `SizeOfOptionalHeadr` [`WORD`] --> `E0 00`（PE头(`IMAGE_OPTIONAL_HEADER32`)大小）
- `Characteristics` [`WORD`] --> `0F 01`（文件属性）

### 2-3 `IMAGE_OPTIONAL_HEADER32`

- `Magic` [`WORD`] --> `0B 01`（标记字）
- `MajorLinkerVersion` [`BYTE`] -> `00`（连接程序主版本号）
- `MinorLinkerVersion` [`BYTE`] -> `00`（连接程序次版本号）
- `SizeOfCode` [`DWORD`] -> `00 00 00 00`（所有含代码区块的总大小）
- `SizeOfInitializedData` [`DWORD`] -> `00 00 00 00`（所有初始化数据区块大总大小）
- `SizeOfUninitializedData` [`DWORD`] -> `00 00 00 00`（所有未初始化数据区块大总大小）
- `AddressOfEntryPoint` [`DWORD`] --> `00 10 00 00`（程序执行入口RAV）
- `BaseOfCode` [`DWORD`] -> `00 00 00 00`（代码区块起始RAV）
- `BaseOfData` [`DWORD`] -> `00 00 00 00`（数据区块起始RAV）
- `ImageBase` [`DWORD`] --> `00 00 40 00`（程序默认装入基地址）
- `SectionAlignment` [`DWORD`] --> `00 10 00 00`（内存中区块对齐值）
- `FileAlignment` [`DWORD`] --> `00 02 00 00`（文件中区块对齐值）
- `MajorOperatingSystemVersion` [`WORD`] -> `00 00`（操作系统主版本号）
- `MinorOperatingSystemVersion` [`WORD`] -> `00 00`（操作系统次版本号）
- `MajorImageVersion` [`WORD`] -> `00 00`（用户自定义主版本号）
- `MinorImageVersion` [`WORD`] -> `00 00`（用户自定义次版本号）
- `MajorSubsystemVersion` [`WORD`] --> `04 00`（运行所需最低子系统主版本号）
- `MinorSubsystemVersion` [`WORD`] --> `00 00`（运行所需最低子系统次版本号）
- `Win32VersionValue` [`DWORD`] -> `00 00 00 00`（保留值，通常为0）
- `SizeOfImage` [`DWORD`] --> `00 40 00 00`（映像装入内存后的总尺寸）
- `SizeOfHeaders` [`DWORD`] --> `00 40 00 00`（DOS头 PE头 区块表的总大小）
- `CheckSum` [`DWORD`] -> `00 00 00 00`（映像效验和）
- `Subsystem` [`WORD`] --> `03 00`（文件子系统）
- `DllCharacteristics` [`WORD`] -> `00 00`（显示DLL特性的旗标）
- `SizeOfStackReserve` [`DWORD`] -> `00 00 00 00`（初始化堆栈总大小）
- `SizeOfStackCommit` [`DWORD`] -> `00 00 00 00`（初始化实际提交堆栈大小）
- `SizeOfHeapReserve` [`DWORD`] -> `00 00 00 00`（初始化保留堆栈大小）
- `SizeOfHeapCommit` [`DWORD`] -> `00 00 00 00`（初始化实际保留堆栈大小）
- `LoaderFlags` [`DWORD`] -> `00 00 00 00`（与调试相关，默认值为0）
- `NumberOfRvaAndSizes` [`DWORD`] --> `10 00 00 00`（数据目录标的项数，默认总为16）

### 2-4 数据目录表

- `2-4-1 Export Table`
  - `VirtualAddress` [`DWORD`] -> `00 00 00 00`（数据块的起始RAV）
  - `Size` [`DWORD`] -> `00 00 00 00`（数据块的长度）
- `2-4-2 Import Table`

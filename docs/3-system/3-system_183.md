

# 文件格式之 PE 知识

> 原文：[`www.pediy.com/kssd/pediy10/100823.html`](https://www.pediy.com/kssd/pediy10/100823.html)

自己最近正在看加密解密中的 PE 文件格式，原来也一直看过 PE 格式，但是很快就忘了，这次特地自己把 WINNT.H 找出来，对每个结构参照书上面的说明研究了下，把 PE 的框架结构里面的一些结构体整理了下，分享给大家！  

代码:

```
//MZ 格式
typedef struct \_IMAGE\_DOS\_HEADER {      // DOS .EXE header
    WORD   e\_magic;                     // Magic number
    WORD   e\_cblp;                      // Bytes on last page of file
    WORD   e\_cp;                        // Pages in file
    WORD   e\_crlc;                      // Relocations
    WORD   e\_cparhdr;                   // Size of header in paragraphs
    WORD   e\_minalloc;                  // Minimum extra paragraphs needed
    WORD   e\_maxalloc;                  // Maximum extra paragraphs needed
    WORD   e\_ss;                        // Initial (relative) SS value
    WORD   e\_sp;                        // Initial SP value
    WORD   e\_csum;                      // Checksum
    WORD   e\_ip;                        // Initial IP value
    WORD   e\_cs;                        // Initial (relative) CS value
    WORD   e\_lfarlc;                    // File address of relocation table
    WORD   e\_ovno;                      // Overlay number
    WORD   e\_res\[4\];                    // Reserved words
    WORD   e\_oemid;                     // OEM identifier (for e\_oeminfo)
    WORD   e\_oeminfo;                   // OEM information; e\_oemid specific
    WORD   e\_res2\[10\];                  // Reserved words
    LONG   e\_lfanew;                    // File address of new exe header  这个最有用指出了真正的 PE 文件头 位于开始偏移 3CH 字节处
  } 


//IMAGE\_NT\_HEADERS 结构(PE 头由三部分组成)
typedef struct \_IMAGE\_NT\_HEADERS {
    DWORD Signature;
    IMAGE\_FILE\_HEADER FileHeader;  //结构体
    IMAGE\_OPTIONAL\_HEADER32 OptionalHeader;//结构体
} 

//IMAGE\_FILE\_HEADER 结构
typedef struct \_IMAGE\_FILE\_HEADER {
    WORD    Machine;   //运行平台 intel i386 一般为 14Ch
    WORD    NumberOfSections;    //块(section)数目 
    DWORD   TimeDateStamp;    //时间日期标记
    DWORD   PointerToSymbolTable;  //COFF 符号指针，这是程序调试信息
    DWORD   NumberOfSymbols;  //符号数
    WORD    SizeOfOptionalHeader;  //可选部首长度，是 IMAGE\_OPTIONAL\_HEADER 的长度
    WORD    Characteristics;  //文件属性
} 

//IMAGE\_OPTIONAL\_HEADER 结构(可选映像头)
typedef struct \_IMAGE\_OPTIONAL\_HEADER {
    //
    // Standard fields.
    //
    WORD    Magic;  //幻数，一般为 10BH  
    BYTE    MajorLinkerVersion;  //链接程序的主版本号
    BYTE    MinorLinkerVersion;  //链接程序的次版本号
    DWORD   SizeOfCode;  //代码段大小
    DWORD   SizeOfInitializedData;  //已初始化数据块的大小
    DWORD   SizeOfUninitializedData;  //未初始化数据库的大小
    DWORD   AddressOfEntryPoint;  //程序开始执行的入口地址,这是一个 RVA（相对虚拟地址）
    DWORD   BaseOfCode; //代码段的起始 RVA
    DWORD   BaseOfData; //数据段的起始 RVA
    //
    // NT additional fields.
    //
    DWORD   ImageBase; //可执行文件默认装入的基地址
    DWORD   SectionAlignment; //内存中块的对齐值（默认的块对齐值为 1000H，4KB 个字节）
    DWORD   FileAlignment;//文件中块的对齐值（默认值为 200H 字节，为了保证块总是从磁盘的扇区开始的）
    WORD    MajorOperatingSystemVersion;//要求操作系统的最低版本号的主版本号
    WORD    MinorOperatingSystemVersion;//要求操作系统的最低版本号的次版本号
    WORD    MajorImageVersion;//该可执行文件的主版本号
    WORD    MinorImageVersion;//该可执行文件的次版本号
    WORD    MajorSubsystemVersion;//要求最低之子系统版本的主版本号
    WORD    MinorSubsystemVersion;//要求最低之子系统版本的次版本号
    DWORD   Win32VersionValue;//保留字
    DWORD   SizeOfImage;//映像装入内存后的总尺寸
    DWORD   SizeOfHeaders;//部首及块表的大小
    DWORD   CheckSum;//CRC 检验和
    WORD    Subsystem;//程序使用的用户接口子系统
    WORD    DllCharacteristics;//DLLmain 函数何时被调用，默认为 0
    DWORD   SizeOfStackReserve;//初始化时堆栈大小
    DWORD   SizeOfStackCommit;//初始化时实际提交的堆栈大小
    DWORD   SizeOfHeapReserve;//初始化时保留的堆大小
    DWORD   SizeOfHeapCommit;//初始化时实际提交的对大小
    DWORD   LoaderFlags;//与调试有关，默认为 0
    DWORD   NumberOfRvaAndSizes;//数据目录结构的数目
    IMAGE\_DATA\_DIRECTORY DataDirectory\[IMAGE\_NUMBEROF\_DIRECTORY\_ENTRIES\];//数据目录表
} 

//数据目录表
typedef struct \_IMAGE\_DATA\_DIRECTORY {
    DWORD   VirtualAddress;//数据块的起始 RVA
    DWORD   Size;//数据块的长度
} 


//块表（）
typedef struct \_IMAGE\_SECTION\_HEADER {
    BYTE    Name\[IMAGE\_SIZEOF\_SHORT\_NAME\];
    union {
            DWORD   PhysicalAddress;
            DWORD   VirtualSize;
    } Misc;
    DWORD   VirtualAddress;//该块的 RVA
    DWORD   SizeOfRawData;//在文件中对齐后的尺寸
    DWORD   PointerToRawData;//在文件中偏移
    DWORD   PointerToRelocations;//重定位偏移
    DWORD   PointerToLinenumbers;//行号表的偏移
    WORD    NumberOfRelocations;//重定位项数目
    WORD    NumberOfLinenumbers;//行号表中行号的数目
    DWORD   Characteristics;//块属性
} 

//输入表结构
typedef struct \_IMAGE\_IMPORT\_DESCRIPTOR {
    union {
        DWORD   Characteristics;            // 0 for terminating null import descriptor
        DWORD   OriginalFirstThunk;         // RVA to original unbound IAT (PIMAGE\_THUNK\_DATA)
    };
    DWORD   TimeDateStamp;                  // 0 if not bound,
                                            // -1 if bound, and real date\\time stamp
                                            //     in IMAGE\_DIRECTORY\_ENTRY\_BOUND\_IMPORT (new BIND)
                                            // O.W. date/time stamp of DLL bound to (Old BIND)

    DWORD   ForwarderChain;                 // -1 if no forwarders
    DWORD   Name;
    DWORD   FirstThunk;                     // RVA to IAT (if bound this IAT has actual addresses)
} 

//输出表结构
typedef struct \_IMAGE\_EXPORT\_DIRECTORY {
    DWORD   Characteristics;
    DWORD   TimeDateStamp;
    WORD    MajorVersion;
    WORD    MinorVersion;
    DWORD   Name;
    DWORD   Base;
    DWORD   NumberOfFunctions;
    DWORD   NumberOfNames;
    DWORD   AddressOfFunctions;     // RVA from base of image
    DWORD   AddressOfNames;         // RVA from base of image
    DWORD   AddressOfNameOrdinals;  // RVA from base of image
} 

```

附件下载：

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || PE 结构分析.rar  
 |
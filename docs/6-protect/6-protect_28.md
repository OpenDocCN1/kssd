

# 根据 PE 文件格式获取 LoadLibraryA()/GetProcAddress()地址 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6683.htm`](https://www.pediy.com/kssd/pediy06/pediy6683.htm)

作者:scz 出处:[`bbs.nsfocus.net/index.php?act=ST&f=3&t=147799`](http://bbs.nsfocus.net/index.php?act=ST&f=3&t=147799)  
  
本节与 PE 文件格式相关的术语以<<微软 PE/COFF 规范>>为准(\[13\])，不再强调该点。  
winnt.h 中定义了部分相关数据结构，后面如未单独注明来自哪个头文件，均隐指来  
自 winnt.h。执行 vulnerable\_0.exe，进入 windbg 调试状态。  
  
\> !list -t \_LIST\_ENTRY.Flink -x "dd" -a "+18 L1" 241ec0  
00241ed8  00400000  
00241f30  77f50000  
00241fd8  77e60000  
... ...  
  
从上节可知，ntdll.dll 基址是 0x77f50000，kernel32.dll 基址是 0x77e60000。最开  
始的 64 字节按如下数据结构解析:  
  
\--------------------------------------------------------------------------  
#define IMAGE\_DOS\_SIGNATURE 0x5A4D      // MZ  
  
typedef struct \_IMAGE\_DOS\_HEADER        // DOS .EXE header  
{  
    WORD   e\_magic;                     // +0x00 Magic number  
    WORD   e\_cblp;                      // +0x02 Bytes on last page of file  
    WORD   e\_cp;                        // +0x04 Pages in file  
    WORD   e\_crlc;                      // +0x06 Relocations  
    WORD   e\_cparhdr;                   // +0x08 Size of header in paragraphs  
    WORD   e\_minalloc;                  // +0x0a Minimum extra paragraphs needed  
    WORD   e\_maxalloc;                  // +0x0c Maximum extra paragraphs needed  
    WORD   e\_ss;                        // +0x0e Initial (relative) SS value  
    WORD   e\_sp;                        // +0x10 Initial SP value  
    WORD   e\_csum;                      // +0x12 Checksum  
    WORD   e\_ip;                        // +0x14 Initial IP value  
    WORD   e\_cs;                        // +0x16 Initial (relative) CS value  
    WORD   e\_lfarlc;                    // +0x18 File address of relocation table  
    WORD   e\_ovno;                      // +0x1a Overlay number  
    WORD   e\_res\[4\];                    // +0x1c Reserved words  
    WORD   e\_oemid;                     // +0x24 OEM identifier (for e\_oeminfo)  
    WORD   e\_oeminfo;                   // +0x26 OEM information; e\_oemid specific  
    WORD   e\_res2\[10\];                  // +0x28 Reserved words  
    LONG   e\_lfanew;                    // +0x3c File address of new exe header  
                                        // +0x40  
} IMAGE\_DOS\_HEADER, \*PIMAGE\_DOS\_HEADER;  
\--------------------------------------------------------------------------  
  
\> db 77e60000 L0n64  
77e60000  4d 5a 90 00 03 00 00 00-04 00 00 00 ff ff 00 00  MZ..............  
77e60010  b8 00 00 00 00 00 00 00-40 00 00 00 00 00 00 00  ........@.......  
77e60020  00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................  
77e60030  00 00 00 00 00 00 00 00-00 00 00 00 f8 00 00 00  ................  
\> dd 77e60000+3c L1 (显示 e\_lfanew 成员)  
77e6003c  000000f8  
  
其中 e\_lfanew 成员用于定位 PE 头。从注释中理解，e\_lfanew 是 File pointer，非 RVA，  
不过在这里当成 RVA 处理也没关系。e\_lfanew 值为 0x000000f8，PE 头在基址加 0xf8 的  
位置。PE 头最开始是标识"PE\\0\\0"，占 4 字节，然后是 20 字节固定头。  
  
\--------------------------------------------------------------------------  
#define IMAGE\_NT\_SIGNATURE       0x00004550  // PE00  
#define IMAGE\_SIZEOF\_FILE\_HEADER 20  
  
typedef struct \_IMAGE\_FILE\_HEADER  
{  
    WORD    Machine;               // +0x00  
    WORD    NumberOfSections;      // +0x02  
    DWORD   TimeDateStamp;         // +0x04  
    DWORD   PointerToSymbolTable;  // +0x08  
    DWORD   NumberOfSymbols;       // +0x0c  
    WORD    SizeOfOptionalHeader;  // +0x10  
    WORD    Characteristics;       // +0x12  
                                   // +0x14  
} IMAGE\_FILE\_HEADER, \*PIMAGE\_FILE\_HEADER;  
\--------------------------------------------------------------------------  
  
\> db 77e60000+f8 L0n24  
77e600f8  50 45 00 00 4c 01 04 00-28 fa 6d 3d 00 00 00 00  PE..L...(.m=....  
77e60108  00 00 00 00 e0 00 0e 21  
                      ^^^^^  
接下来是可选头，SizeOfOptionalHeader 成员表明可选头占用了 224 字节(0x00e0)。  
  
\--------------------------------------------------------------------------  
typedef struct \_IMAGE\_OPTIONAL\_HEADER  
{  
    WORD    Magic;                        // +0x00  
    BYTE    MajorLinkerVersion;           // +0x02  
    BYTE    MinorLinkerVersion;           // +0x03  
    DWORD   SizeOfCode;                   // +0x04  
    DWORD   SizeOfInitializedData;        // +0x08  
    DWORD   SizeOfUninitializedData;      // +0x0c  
    DWORD   AddressOfEntryPoint;          // +0x10  
    DWORD   BaseOfCode;                   // +0x14  
    DWORD   BaseOfData;                   // +0x18  
    DWORD   ImageBase;                    // +0x1c  
    DWORD   SectionAlignment;             // +0x20  
    DWORD   FileAlignment;                // +0x24  
    WORD    MajorOperatingSystemVersion;  // +0x28  
    WORD    MinorOperatingSystemVersion;  // +0x2a  
    WORD    MajorImageVersion;            // +0x2c  
    WORD    MinorImageVersion;            // +0x2e  
    WORD    MajorSubsystemVersion;        // +0x30  
    WORD    MinorSubsystemVersion;        // +0x32  
    DWORD   Win32VersionValue;            // +0x34  
    DWORD   SizeOfImage;                  // +0x38  
    DWORD   SizeOfHeaders;                // +0x3c  
    DWORD   CheckSum;                     // +0x40  
    WORD    Subsystem;                    // +0x44  
    WORD    DllCharacteristics;           // +0x46  
    DWORD   SizeOfStackReserve;           // +0x48  
    DWORD   SizeOfStackCommit;            // +0x4c  
    DWORD   SizeOfHeapReserve;            // +0x50  
    DWORD   SizeOfHeapCommit;             // +0x54  
    DWORD   LoaderFlags;                  // +0x58  
    DWORD   NumberOfRvaAndSizes;          // +0x5c Number of data-dictionary entries in the remainder of the Optional Header  
                                          // +0x60  
    IMAGE\_DATA\_DIRECTORY DataDirectory\[IMAGE\_NUMBEROF\_DIRECTORY\_ENTRIES\];  
                                          // +0xe0  
} IMAGE\_OPTIONAL\_HEADER32, \*PIMAGE\_OPTIONAL\_HEADER32;  
  
typedef struct \_IMAGE\_DATA\_DIRECTORY  
{  
    DWORD   VirtualAddress;  // +0x00 RVA  
    DWORD   Size;            // +0x04 The size in bytes  
                             // +0x08  
} IMAGE\_DATA\_DIRECTORY, \*PIMAGE\_DATA\_DIRECTORY;  
  
#define IMAGE\_NUMBEROF\_DIRECTORY\_ENTRIES     16  
  
#define IMAGE\_DIRECTORY\_ENTRY\_EXPORT          0   // Export Directory  
#define IMAGE\_DIRECTORY\_ENTRY\_IMPORT          1   // Import Directory  
#define IMAGE\_DIRECTORY\_ENTRY\_RESOURCE        2   // Resource Directory  
#define IMAGE\_DIRECTORY\_ENTRY\_EXCEPTION       3   // Exception Directory  
#define IMAGE\_DIRECTORY\_ENTRY\_SECURITY        4   // Security Directory  
#define IMAGE\_DIRECTORY\_ENTRY\_BASERELOC       5   // Base Relocation Table  
#define IMAGE\_DIRECTORY\_ENTRY\_DEBUG           6   // Debug Directory  
#define IMAGE\_DIRECTORY\_ENTRY\_ARCHITECTURE    7   // Architecture Specific Data  
#define IMAGE\_DIRECTORY\_ENTRY\_GLOBALPTR       8   // RVA of GP  
#define IMAGE\_DIRECTORY\_ENTRY\_TLS             9   // TLS Directory  
#define IMAGE\_DIRECTORY\_ENTRY\_LOAD\_CONFIG    10   // Load Configuration Directory  
#define IMAGE\_DIRECTORY\_ENTRY\_BOUND\_IMPORT   11   // Bound Import Directory in headers  
#define IMAGE\_DIRECTORY\_ENTRY\_IAT            12   // Import Address Table  
#define IMAGE\_DIRECTORY\_ENTRY\_DELAY\_IMPORT   13   // Delay Load Import Descriptors  
#define IMAGE\_DIRECTORY\_ENTRY\_COM\_DESCRIPTOR 14   // COM Runtime descriptor  
\--------------------------------------------------------------------------  
  
IMAGE\_DATA\_DIRECTORY.VirtualAddress 是 RVA。一般情况 DataDirectory\[\]是含有 16  
个元素的结构数组。前两个元素分别对应 Export Directory 与 Import Directory。但  
是规范 3.4.3 小节指出元素个数不固定，需要检查 NumberOfRvaAndSizes 成员确定元素  
个数。  
  
此外，不要假设 IMAGE\_DATA\_DIRECTORY.VirtualAddress 指向所在 section 的起始位置，  
比如 Import Directory 一般位于.idata section 中，但不能假设 RVA 指向.idata 的起  
始位置。不要假设 Import Directory 所在 section 一定拥有".idata"这个名字。  
  
\> db 77e60000+f8+0n24 L0n224  
77e60110  0b 01 07 00 00 56 07 00-00 dc 06 00 00 00 00 00  .....V..........  
77e60120  60 ae 01 00 00 10 00 00-00 20 07 00 00 00 e6 77  \`........ .....w  
77e60130  00 10 00 00 00 02 00 00-05 00 01 00 05 00 01 00  ................  
77e60140  04 00 00 00 00 00 00 00-00 60 0e 00 00 04 00 00  .........\`......  
77e60150  d3 7e 0e 00 03 00 00 00-00 00 04 00 00 10 00 00  .~..............  
77e60160  00 00 10 00 00 10 00 00-00 00 00 00 10 00 00 00  ................  
77e60170  40 d0 06 00 39 6b 00 00-7c 3b 07 00 28 00 00 00  @...9k..|;..(...  
77e60180  00 a0 07 00 d8 5e 06 00-00 00 00 00 00 00 00 00  .....^..........  
77e60190  00 00 00 00 00 00 00 00-00 00 0e 00 54 53 00 00  ............TS..  
77e601a0  fc 63 07 00 38 00 00 00-00 00 00 00 00 00 00 00  .c..8...........  
77e601b0  00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................  
77e601c0  a8 76 07 00 40 00 00 00-90 02 00 00 1c 00 00 00  .v..@...........  
77e601d0  00 10 00 00 0c 06 00 00-00 00 00 00 00 00 00 00  ................  
77e601e0  00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00  ................  
\> dd 77e60000+f8+0n24+5c L1 (检查 NumberOfRvaAndSizes 成员)  
77e6016c  00000010  
\> dd 77e60000+f8+0n24+60 L2 (定位 Export Directory，当前基是 16)  
77e60170  0006d040 00006b39  
          ^^^^^^^^  
\> ? 77e60000+0006d040+00006b39  
Evaluate expression: 2012035961 = 77ed3b79  
\> ? 77e60000+0006d040  
Evaluate expression: 2012008512 = 77ecd040  
  
现在我们知道 Export Directory 在"77e60000+0006d040"，格式如下:  
  
\--------------------------------------------------------------------------  
typedef struct \_IMAGE\_EXPORT\_DIRECTORY  
{  
    DWORD   Characteristics;        // +0x00  
    DWORD   TimeDateStamp;          // +0x04  
    WORD    MajorVersion;           // +0x08  
    WORD    MinorVersion;           // +0x0a  
    DWORD   Name;                   // +0x0c Name of the DLL  
    DWORD   Base;                   // +0x10 Starting ordinal number for exports  
    DWORD   NumberOfFunctions;      // +0x14 Number of entries in the EAT  
    DWORD   NumberOfNames;          // +0x18 Number of entries in the ENPT/EOT  
    DWORD   AddressOfFunctions;     // +0x1c RVA from base of image  
    DWORD   AddressOfNames;         // +0x20 RVA from base of image  
    DWORD   AddressOfNameOrdinals;  // +0x24 RVA from base of image  
                                    // +0x28  
} IMAGE\_EXPORT\_DIRECTORY, \*PIMAGE\_EXPORT\_DIRECTORY;  
\--------------------------------------------------------------------------  
  
AddressOfFunctions  
  
    指向 Export Address Table。  
  
    DWORD EAT\[NumberOfFunctions\];  
  
EAT\[i\]  
  
    可选头中 IMAGE\_DATA\_DIRECTORY 结构决定了引出信息范围。如果 EAT\[i\]的值不在  
    引出信息范围，则为函数地址(Export RVA)。否则，该值指向形如 dllname.#27  
    或者 dllname.exportfunc 的 ASCIZ 串，用 MS 术语说，这是一个 Forwarder RVA，表  
    示由另外一个 dll 实际引出某函数。  
  
AddressOfNames  
  
    指向 Export Name Pointer Table  
  
    DWORD ENPT\[NumberOfNames\];  
  
ENPT\[i\]  
  
    指向 Export Name Table 中某个位置  
  
    ENT 由 ASCIZ 串构成  
  
AddressOfNameOrdinals  
  
    指向 Export Ordinal Table  
  
    WORD EOT\[NumberOfNames\];  
  
参看规范 6.3.4 小节，这几个数组之间的关系用伪 C 语言描述如下:  
  
\--------------------------------------------------------------------------  
/\*  
\* 怀疑该规范与 MS 最终实现有出入，规范中 EOT\[0\]应该等于 ordinal\_base 的，但 MS  
\* 最终实现里 EOT\[0\]等于 0。这里的伪代码符合 MS 的最终实现。看来，理论->实践->  
\* 再理论的学习过程永远都要牢记，否则死菜了都不知道怎么死菜的。  
\*/  
index         = Search\_ENPT( function\_name );  
index         = EOT\[ index \];  
function\_addr = EAT\[ index \];  
ordinal       = ordinal\_base + index;  
\--------------------------------------------------------------------------  
  
为何多出个 EOT，因为可能不同的函数名对应同一个函数地址，参\[14\]的 12.5 小节。  
  
假设源代码中有如下声明:  
  
\_\_declspec(dllexport) int \_\_stdcall PublicFunc ( int a, int b, int c, int d );  
  
假设没有使用 DEF 文件:  
  
EXPORTS  
  
    PublicFunc  
  
同时却又在源代码出现:  
  
#pragma comment( linker, "/EXPORT:PublicFunc=\_PublicFunc@32" )  
  
此时生成 PE 文件时会同时引出两个符号，"PublicFunc"与"\_PublicFunc@32"，这两个  
符号对应同一个函数。反映在上述几个数组中，即 EOT\[i\]等于 EOT\[j\]。  
  
NumberOfFunctions 与 NumberOfNames 在这种情形下不等。除此之外，还有一种不等情  
形。NumberOfNames 为 0，NumberOfFunctions 不为 0，表示模块仅通过 ordinal 引出函  
数，这是相当极端却有可能出现的情形。  
  
VC 有个现成的工具 dumpbin，可用于观察引出(export)信息:  
  
\> dumpbin X:\\XP\\system32\\kernel32.dll /exports  
  
  Section contains the following exports for KERNEL32.dll  
  
    00000000 characteristics  
    3D6DE616 time date stamp Thu Aug 29 17:15:02 2002  
        0.00 version  
           1 ordinal base  
         942 number of functions  
         942 number of names  
  
    ordinal hint RVA      name  
  
          1    0 000137E8 ActivateActCtx  
          2    1 000093FE AddAtomA  
          3    2 0000D496 AddAtomW  
          4    3 000607C5 AddConsoleAliasA  
          5    4 0006078E AddConsoleAliasW  
          6    5 0004E0A1 AddLocalAlternateComputerNameA  
          7    6 0004DF8C AddLocalAlternateComputerNameW  
          8    7 00035098 AddRefActCtx  
          9    8          AddVectoredExceptionHandler (forwarded to NTDLL.RtlAddVectoredExceptionHandler)  
         10    9 00036909 AllocConsole  
        ... ...  
        401  190 0001B332 GetProcAddress  
        ... ...  
        571  23A 0001D961 LoadLibraryA  
        572  23B 0001D941 LoadLibraryExA  
        573  23C 0001D839 LoadLibraryExW  
        574  23D 00013B38 LoadLibraryW  
        ... ...  
  
回 windbg 验证一下:  
  
\> db 77e60000+0006d040 L28  
77ecd040  00 00 00 00 16 e6 6d 3d-00 00 00 00 34 f5 06 00  ......m=....4...  
77ecd050  01 00 00 00 ae 03 00 00-ae 03 00 00 68 d0 06 00  ............h...  
77ecd060  20 df 06 00 d8 ed 06 00  
\> da 77e60000+poi(0x77e60000+0x0006d040+0xc) (Name)  
77ecf534  "KERNEL32.dll"  
\> dd 0x77e60000+0x0006d040+0x10 L1 (Base)  
77ecd050  00000001  
\> dd 0x77e60000+0x0006d040+0x14 L1 (NumberOfFunctions)  
77ecd054  000003ae  
\> dd 0x77e60000+0x0006d040+0x18 L1 (NumberOfNames)  
77ecd058  000003ae  
\> dd 0x77e60000+0x0006d040+0x1c L3 (AddressOfFunctions、AddressOfNames、AddressOfNameOrdinals)  
77ecd05c  0006d068 0006df20 0006edd8  
  
套用这个公式:  
  
\--------------------------------------------------------------------------  
0x190      = Search\_ENPT( "GetProcAddress" );  
0x190      = EOT\[ 0x190 \];  
0x77e7b332 = EAT\[ 0x190 \];  
0x191      = 1 + 0x190;  
\--------------------------------------------------------------------------  
  
\> da 77e60000+poi(77e60000+0006df20+0x190\*4) (访问 ENPT、ENT)  
77ed1393  "GetProcAddress"  
\> dw 77e60000+0006edd8+0x190\*2 L1 (访问 EOT)  
77ecf0f8  0190  
\> ? 77e60000+poi(77e60000+0006d068+0x190\*4) (访问 EAT)  
Evaluate expression: 2011673394 = 77e7b332  
\> u 77e7b332 (这个地址不在\[77ecd040, 77ed3b79)内)  
kernel32!GetProcAddress:  
77e7b332 55               push    ebp  
77e7b333 8bec             mov     ebp,esp  
77e7b335 51               push    ecx  
77e7b336 51               push    ecx  
77e7b337 53               push    ebx  
77e7b338 57               push    edi  
77e7b339 8b7d0c           mov     edi,\[ebp+0xc\]  
77e7b33c bbffff0000       mov     ebx,0xffff  
  
再来验证一下 Forwarder RVA 的情形:  
  
\--------------------------------------------------------------------------  
8          = Search\_ENPT( "AddVectoredExceptionHandler" );  
8          = EOT\[ 8 \];  
0x77ed38ad = EAT\[ 8 \];  
9          = 1 + 8;  
\--------------------------------------------------------------------------  
  
\> da 77e60000+poi(77e60000+0006df20+8\*4) (访问 ENPT、ENT)  
77ecf5cf  "AddVectoredExceptionHandler"  
\> dw 77e60000+0006edd8+8\*2 L1 (访问 EOT)  
77ecede8  0008  
\> ? 77e60000+poi(77e60000+0006d068+8\*4) (访问 EAT)  
Evaluate expression: 2012035245 = 77ed38ad  
\> da 77e60000+poi(77e60000+0006d068+8\*4) (这个地址在\[77ecd040, 77ed3b79)内)  
77ed38ad  "NTDLL.RtlAddVectoredExceptionHan"  
77ed38cd  "dler"  
  
现在总结一下"根据 PE 文件格式获取 LoadLibraryA()/GetProcAddress()地址"全过程:  
  
a. 通过 TEB/PEB 获取 kernel32.dll 基址  
  
b. 在(基址+0x3c)处获取 e\_lfanew  
  
c. 在(基址+e\_lfanew+0x78)处获取 Export Directory 地址(后面为描述方便简称 export)  
  
d. 在(基址+export+0x1c)处获取 AddressOfFunctions、AddressOfNames、AddressOfNameOrdinals  
  
e. 搜索 ENPT，确定"LoadLibraryA"、"GetProcAddress"所对应的 index  
  
f. index = EOT\[ index \];  
  
g. function\_addr = EAT\[ index \];  
  
下面是完整的 C 语言演示程序，汇编化留到编写完整 shellcode 时进行。  
  
\--------------------------------------------------------------------------  
/\*  
\* -----------------------------------------------------------------------  
\* Compile : For x86/EWindows XP SP1 & VC 7  
\*         : cl GetAddr.c /nologo /Os /G6 /W3 /D "WIN32" /D "NDEBUG" /D "\_CONSOLE" /link /RELEASE  
\*         :  
\* Create  : 2003-08-14 15:11  
\* Modify  :  
\* -----------------------------------------------------------------------  
\*/  
  
#include <stdio.h>  
#include <stdlib.h>  
#include <windows.h>  
  
#pragma comment( linker, "/INCREMENTAL:NO"    )  
#pragma comment( linker, "/subsystem:console" )  
  
static void \* \_\_stdcall SearchAPI ( char \*BASE, DWORD \*EAT, DWORD \*ENPT, WORD \*EOT, DWORD num, char \*name )  
{  
    DWORD  index         = 0;  
    char  \*function\_name = NULL;  
    void  \*function\_addr = NULL;  
  
    for ( index = 0; index < num; index++ )  
    {  
        function\_name = ENPT\[index\] + BASE;  
        /\*  
         \* 大小写敏感比较  
         \*/  
        if ( 0 == strcmp( function\_name, name ) )  
        {  
            index         = EOT\[index\];  
            function\_addr = EAT\[index\] + BASE;  
            return( function\_addr );  
        }  
    }  /\* end of for \*/  
    return( NULL );  
}  /\* end of SearchAPI \*/  
  
int \_\_cdecl main ( int argc, char \* argv\[\] )  
{  
    void  \*PEB                   = NULL,  
          \*Ldr                   = NULL,  
          \*Flink                 = NULL,  
          \*kernel32\_BaseAddress  = NULL,  
          \*kernel32\_BaseDllName  = NULL,  
          \*ExportDirectory       = NULL,  
          \*PrivateLoadLibraryA   = NULL,  
          \*PrivateGetProcAddress = NULL;  
    DWORD  NumberOfNames         = 0,  
          \*AddressOfFunctions    = NULL,  
          \*AddressOfNames        = NULL;  
    WORD  \*AddressOfNameOrdinals = NULL;  
    LONG   e\_lfanew              = 0;  
  
    \_\_asm  
    {  
        mov     eax,fs:\[0x30\]  
        mov     PEB,eax  
    }  
    printf( "PEB                   = 0x%08X\\n", PEB );  
    Ldr                   = \*( ( void \*\* )( ( unsigned char \* )PEB + 0x0c ) );  
    printf( "Ldr                   = 0x%08X\\n", Ldr );  
    Flink                 = \*( ( void \*\* )( ( unsigned char \* )Ldr + 0x1c ) );  
    printf( "Flink                 = 0x%08X\\n", Flink );  
    Flink                 = \*( ( void \*\* )Flink );  
    kernel32\_BaseAddress  = \*( ( void \*\* )( ( unsigned char \* )Flink + 0x08 ) );  
    kernel32\_BaseDllName  = \*( ( void \*\* )( ( unsigned char \* )Flink + 0x20 ) );  
    printf( "kernel32\_BaseAddress  = 0x%08X\\n", kernel32\_BaseAddress );  
    wprintf( L"kernel32\_BaseDllName  = %s\\n", kernel32\_BaseDllName );  
    /\*  
     \* 根据 PE 文件格式进行解析  
     \*/  
    e\_lfanew              = \*( ( LONG \* )( ( unsigned char \* )kernel32\_BaseAddress + 0x3c ) );  
    printf( "e\_lfanew              = 0x%08X\\n", e\_lfanew );  
    ExportDirectory       = \*( ( DWORD \* )( ( unsigned char \* )kernel32\_BaseAddress + e\_lfanew + 0x78 ) ) +  
                            ( unsigned char \* )kernel32\_BaseAddress;  
    printf( "ExportDirectory       = 0x%08X\\n", ExportDirectory );  
    NumberOfNames         = \*( ( DWORD \* )( ( unsigned char \* )ExportDirectory + 0x18 ) );  
    printf( "NumberOfNames         = %u\\n", NumberOfNames );  
    AddressOfFunctions    = ( DWORD \* )  
                            (  
                                \*( ( DWORD \* )( ( unsigned char \* )ExportDirectory + 0x1c ) ) +  
                                ( unsigned char \* )kernel32\_BaseAddress  
                            );  
    printf( "AddressOfFunctions    = 0x%08X\\n", AddressOfFunctions );  
    AddressOfNames        = ( DWORD \* )  
                            (  
                                \*( ( DWORD \* )( ( unsigned char \* )ExportDirectory + 0x20 ) ) +  
                                ( unsigned char \* )kernel32\_BaseAddress  
                            );  
    printf( "AddressOfNames        = 0x%08X\\n", AddressOfNames );  
    AddressOfNameOrdinals = ( WORD \* )  
                            (  
                                \*( ( DWORD \* )( ( unsigned char \* )ExportDirectory + 0x24 ) ) +  
                                ( unsigned char \* )kernel32\_BaseAddress  
                            );  
    printf( "AddressOfNameOrdinals = 0x%08X\\n", AddressOfNameOrdinals );  
    PrivateLoadLibraryA   = SearchAPI  
                            (  
                                kernel32\_BaseAddress,  
                                AddressOfFunctions,  
                                AddressOfNames,  
                                AddressOfNameOrdinals,  
                                NumberOfNames,  
                                "LoadLibraryA"  
                            );  
    printf( "PrivateLoadLibraryA   = 0x%08X\\n", PrivateLoadLibraryA );  
    printf( "LoadLibraryA          = 0x%08X\\n", LoadLibraryA );  
    PrivateGetProcAddress = SearchAPI  
                            (  
                                kernel32\_BaseAddress,  
                                AddressOfFunctions,  
                                AddressOfNames,  
                                AddressOfNameOrdinals,  
                                NumberOfNames,  
                                "GetProcAddress"  
                            );  
    printf( "PrivateGetProcAddress = 0x%08X\\n", PrivateGetProcAddress );  
    printf( "GetProcAddress        = 0x%08X\\n", GetProcAddress );  
    return( EXIT\_SUCCESS );  
}  /\* end of main \*/  
  
#if 0  
  
/\*  
\* 按初始化顺序前向遍历链表，第二个节点对应 kernel32.dll。  
\*/  
  
#include <stdio.h>  
#include <stdlib.h>  
  
#pragma comment( linker, "/INCREMENTAL:NO"    )  
#pragma comment( linker, "/subsystem:console" )  
  
int \_\_cdecl main ( int argc, char \* argv\[\] )  
{  
    void \*PEB         = NULL,  
         \*Ldr         = NULL,  
         \*Flink       = NULL,  
         \*p           = NULL,  
         \*BaseAddress = NULL,  
         \*BaseDllName = NULL;  
  
    \_\_asm  
    {  
        mov     eax,fs:\[0x30\]  
        mov     PEB,eax  
    }  
    printf( "PEB   = 0x%08X\\n", PEB );  
    Ldr   = \*( ( void \*\* )( ( unsigned char \* )PEB + 0x0c ) );  
    printf( "Ldr   = 0x%08X\\n", Ldr );  
    Flink = \*( ( void \*\* )( ( unsigned char \* )Ldr + 0x1c ) );  
    printf( "Flink = 0x%08X\\n", Flink );  
    p     = Flink;  
    do  
    {  
        BaseAddress = \*( ( void \*\* )( ( unsigned char \* )p + 0x08 ) );  
        BaseDllName = \*( ( void \*\* )( ( unsigned char \* )p + 0x20 ) );  
        printf( "p     = 0x%08X 0x%08X ", p, BaseAddress );  
        wprintf( L"%s\\n", BaseDllName );  
        p = \*( ( void \*\* )p );  
    }  
    while ( Flink != p );  
    return( EXIT\_SUCCESS );  
}  /\* end of main \*/  
  
#endif  
\--------------------------------------------------------------------------  
  
执行效果如下，对比 PrivateLoadLibraryA 与 LoadLibraryA 的值，应该相等。  
  
\> GetAddr  
PEB                   = 0x7FFDF000  
Ldr                   = 0x00241E90  
Flink                 = 0x00241F28  
kernel32\_BaseAddress  = 0x77E60000  
kernel32\_BaseDllName  = kernel32.dll  
e\_lfanew              = 0x000000F8  
ExportDirectory       = 0x77ECD040  
NumberOfNames         = 942  
AddressOfFunctions    = 0x77ECD068  
AddressOfNames        = 0x77ECDF20  
AddressOfNameOrdinals = 0x77ECEDD8  
PrivateLoadLibraryA   = 0x77E7D961  
LoadLibraryA          = 0x77E7D961  
PrivateGetProcAddress = 0x77E7B332  
GetProcAddress        = 0x77E7B332  
  
这里演示的技巧不只用于 exploit、shellcode、virus，还大量用于特殊 Driver 编程  
中。根据 PE 文件格式解析内存中的模块映像是很基础的知识，我今天才接触了一下，  
惭愧。  
  
SearchAPI()用到了被搜索函数名，直接在 shellcode 中保存完整函数名会占用不少空  
间。可以考虑在 shellcode 中保存函数名的某种哈希值，同时 SearchAPI()比较哈希值  
而非函数名。任何一种哈希算法都会丢失部分原有信息，以致不同函数名产生的哈希  
值相同，所谓"碰撞"。virus 编程中为此常常使用标准 CRC32 算法(\[8\])，参 29A-4.227。  
但是 LSD 认为 CRC32 的汇编算法太长了(\[15\])，他们用了一个相当简单的算法:  
  
while ( \*c )  
{  
    h = ( ( h << 5 ) | ( h >> 27 ) ) + \*c++;  
}  
  
不超过 10 行汇编代码。据 LSD 的报告称，对超过 5000 个不同的 dll 测试，覆盖 50000 个  
不同的函数名，该哈希算法未产生一次碰撞。若真是如此，对于编写 shellcode 来讲，  
完全足够了。  
  
下面是完整的 C 语言演示程序，真正汇编化后应提前计算函数名的哈希值。  
  
\--------------------------------------------------------------------------  
/\*  
\* -----------------------------------------------------------------------  
\* Compile : For x86/EWindows XP SP1 & VC 7  
\*         : cl GetAddr\_0.c /nologo /Os /G6 /W3 /D "WIN32" /D "NDEBUG" /D "\_CONSOLE" /link /RELEASE  
\*         :  
\* Create  : 2003-08-14 17:24  
\* Modify  :  
\* -----------------------------------------------------------------------  
\*/  
  
#include <stdio.h>  
#include <stdlib.h>  
#include <windows.h>  
  
#pragma comment( linker, "/INCREMENTAL:NO"    )  
#pragma comment( linker, "/subsystem:console" )  
  
static DWORD \_\_stdcall GetHash ( char \*c )  
{  
    DWORD h = 0;  
  
    while ( \*c )  
    {  
        h = ( ( h << 5 ) | ( h >> 27 ) ) + \*c++;  
    }  
    return( h );  
}  /\* end of GetHash \*/  
  
static void \* \_\_stdcall SearchAPI ( char \*BASE, DWORD \*EAT, DWORD \*ENPT, WORD \*EOT, DWORD num, DWORD NameHash )  
{  
    DWORD  index         = 0,  
           h             = 0;  
    char  \*function\_name = NULL;  
    void  \*function\_addr = NULL;  
  
    for ( index = 0; index < num; index++ )  
    {  
        function\_name = ENPT\[index\] + BASE;  
        h             = GetHash( function\_name );  
        if ( h == NameHash )  
        {  
            index         = EOT\[index\];  
            function\_addr = EAT\[index\] + BASE;  
            return( function\_addr );  
        }  
    }  /\* end of for \*/  
    return( NULL );  
}  /\* end of SearchAPI \*/  
  
int \_\_cdecl main ( int argc, char \* argv\[\] )  
{  
    void  \*PEB                   = NULL,  
          \*Ldr                   = NULL,  
          \*Flink                 = NULL,  
          \*kernel32\_BaseAddress  = NULL,  
          \*kernel32\_BaseDllName  = NULL,  
          \*ExportDirectory       = NULL,  
          \*PrivateLoadLibraryA   = NULL,  
          \*PrivateGetProcAddress = NULL;  
    DWORD  NumberOfNames         = 0,  
          \*AddressOfFunctions    = NULL,  
          \*AddressOfNames        = NULL,  
           NameHash              = 0;  
    WORD  \*AddressOfNameOrdinals = NULL;  
    LONG   e\_lfanew              = 0;  
  
    \_\_asm  
    {  
        mov     eax,fs:\[0x30\]  
        mov     PEB,eax  
    }  
    printf( "PEB                   = 0x%08X\\n", PEB );  
    Ldr                   = \*( ( void \*\* )( ( unsigned char \* )PEB + 0x0c ) );  
    printf( "Ldr                   = 0x%08X\\n", Ldr );  
    Flink                 = \*( ( void \*\* )( ( unsigned char \* )Ldr + 0x1c ) );  
    printf( "Flink                 = 0x%08X\\n", Flink );  
    Flink                 = \*( ( void \*\* )Flink );  
    kernel32\_BaseAddress  = \*( ( void \*\* )( ( unsigned char \* )Flink + 0x08 ) );  
    kernel32\_BaseDllName  = \*( ( void \*\* )( ( unsigned char \* )Flink + 0x20 ) );  
    printf( "kernel32\_BaseAddress  = 0x%08X\\n", kernel32\_BaseAddress );  
    wprintf( L"kernel32\_BaseDllName  = %s\\n", kernel32\_BaseDllName );  
    /\*  
     \* 根据 PE 文件格式进行解析  
     \*/  
    e\_lfanew              = \*( ( LONG \* )( ( unsigned char \* )kernel32\_BaseAddress + 0x3c ) );  
    printf( "e\_lfanew              = 0x%08X\\n", e\_lfanew );  
    ExportDirectory       = \*( ( DWORD \* )( ( unsigned char \* )kernel32\_BaseAddress + e\_lfanew + 0x78 ) ) +  
                            ( unsigned char \* )kernel32\_BaseAddress;  
    printf( "ExportDirectory       = 0x%08X\\n", ExportDirectory );  
    NumberOfNames         = \*( ( DWORD \* )( ( unsigned char \* )ExportDirectory + 0x18 ) );  
    printf( "NumberOfNames         = %u\\n", NumberOfNames );  
    AddressOfFunctions    = ( DWORD \* )  
                            (  
                                \*( ( DWORD \* )( ( unsigned char \* )ExportDirectory + 0x1c ) ) +  
                                ( unsigned char \* )kernel32\_BaseAddress  
                            );  
    printf( "AddressOfFunctions    = 0x%08X\\n", AddressOfFunctions );  
    AddressOfNames        = ( DWORD \* )  
                            (  
                                \*( ( DWORD \* )( ( unsigned char \* )ExportDirectory + 0x20 ) ) +  
                                ( unsigned char \* )kernel32\_BaseAddress  
                            );  
    printf( "AddressOfNames        = 0x%08X\\n", AddressOfNames );  
    AddressOfNameOrdinals = ( WORD \* )  
                            (  
                                \*( ( DWORD \* )( ( unsigned char \* )ExportDirectory + 0x24 ) ) +  
                                ( unsigned char \* )kernel32\_BaseAddress  
                            );  
    printf( "AddressOfNameOrdinals = 0x%08X\\n", AddressOfNameOrdinals );  
    NameHash              = GetHash( "LoadLibraryA" );  
    printf( "NameHash              = 0x%08X\\n", NameHash );  
    PrivateLoadLibraryA   = SearchAPI  
                            (  
                                kernel32\_BaseAddress,  
                                AddressOfFunctions,  
                                AddressOfNames,  
                                AddressOfNameOrdinals,  
                                NumberOfNames,  
                                NameHash  
                            );  
    printf( "PrivateLoadLibraryA   = 0x%08X\\n", PrivateLoadLibraryA );  
    printf( "LoadLibraryA          = 0x%08X\\n", LoadLibraryA );  
    NameHash              = GetHash( "GetProcAddress" );  
    printf( "NameHash              = 0x%08X\\n", NameHash );  
    PrivateGetProcAddress = SearchAPI  
                            (  
                                kernel32\_BaseAddress,  
                                AddressOfFunctions,  
                                AddressOfNames,  
                                AddressOfNameOrdinals,  
                                NumberOfNames,  
                                NameHash  
                            );  
    printf( "PrivateGetProcAddress = 0x%08X\\n", PrivateGetProcAddress );  
    printf( "GetProcAddress        = 0x%08X\\n", GetProcAddress );  
    return( EXIT\_SUCCESS );  
}  /\* end of main \*/  
\--------------------------------------------------------------------------  
  
\> GetAddr\_0  
PEB                   = 0x7FFDF000  
Ldr                   = 0x00241E90  
Flink                 = 0x00241F28  
kernel32\_BaseAddress  = 0x77E60000  
kernel32\_BaseDllName  = kernel32.dll  
e\_lfanew              = 0x000000F8  
ExportDirectory       = 0x77ECD040  
NumberOfNames         = 942  
AddressOfFunctions    = 0x77ECD068  
AddressOfNames        = 0x77ECDF20  
AddressOfNameOrdinals = 0x77ECEDD8  
NameHash              = 0x331ADDDC <- "LoadLibraryA"的哈希值  
PrivateLoadLibraryA   = 0x77E7D961  
LoadLibraryA          = 0x77E7D961  
NameHash              = 0x99C95590 <- "GetProcAddress"的哈希值  
PrivateGetProcAddress = 0x77E7B332  
GetProcAddress        = 0x77E7B332  
  
☆ 参考资源  
  
\[13\] Microsoft Portable Executable and Common Object File Format Specification  
     [`www.microsoft.com/whdc/hwdev/download/hardware/pecoff.doc`](http://www.microsoft.com/whdc/hwdev/download/hardware/pecoff.doc)  
     [`www.microsoft.com/whdc/hwdev/download/hardware/pecoff.pdf`](http://www.microsoft.com/whdc/hwdev/download/hardware/pecoff.pdf)  
  
\[14\] <<Programming Applications for Microsoft Windows, Fourth Edition>> - Jeffrey Richter  
  
\[15\] [`www.lsd-pl.net/documents/winasm-1.0.1.pdf`](http://www.lsd-pl.net/documents/winasm-1.0.1.pdf)  
     [`www.lsd-pl.net/documents/winasm.ppt`](http://www.lsd-pl.net/documents/winasm.ppt)  
     [`lsd-pl.net/projects/winasm-1.0.1.tar.gz`](http://lsd-pl.net/projects/winasm-1.0.1.tar.gz)

 4dd466d0e1e0cc123afbe504da2462ca
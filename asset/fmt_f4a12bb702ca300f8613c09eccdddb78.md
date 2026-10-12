
# 资源目录与重定位

要找出资源目录的下级目录是否是另一个目录，检查偏移量的高位，如果置 1，其余 31 位是从资源原始数据开始到下一个目录的偏移量。格式还是 `IMAGE_RESOURCE_DIRECTORY` 后面跟着 `IMAGE_RESOURCE_DIRECTORY_ENTRYs` 项目。

如果高位清 0，偏移量是从节开始到资源原始数据描述结构（一个 `IMAGE_RESOURCE_DATA_ENTRY`）的偏移量。一个 `IMAGE_RESOURCE_DATA_ENTRY` 包括：

```
IMAGE_RESOURCE_DATA_ENTRY STRUCT  
    OffsetToData dd ?  
    Size1 dd        ?  
    CodePage dd     ?  
    Reserved dd     ?  
IMAGE_RESOURCE_DATA_ENTRY ENDS  
```

32 位 `OffsetToData` 从资源节开始到原始数据的偏移量，32 位数据大小 `Size`，32 位 `CodePage` 和 32 保留。

原始数据格式依赖于资源类型，任何字符串资源都是 `UNICODE` 格式。

## 重定位 relocations

最后一个数据目录是重定位，基重定位目录被 `IMAGE_DIRECTORY_ENTRY_BASERELOC` 指向，典型的包括一个自己的节，名字是 `".reloc"` 以及 `IMAGE_SCN_CNT_INITIALIZED_DATA`, `IMAGE_SCN_MEM_DISCARDABLE` 和 `IMAGE_SCN_MEM_READ` 位集合。

如果映象没有被加载器调入优先地址，则该数据被加载器使用。这种情况下给链接器填入的地址无效了。加载器必须修复用于静态变量的绝对地址，串等。

```
IMAGE_BASE_RELOCATION STRUCT  
    VirtualAddress dd   ?  
    SizeOfBlock dd      ?  
IMAGE_BASE_RELOCATION ENDS  
```

重定位目录是一系列块，每个块包括重定位信息针对 4K 的映象。一个块起始于一个 `IMAGE_BASE_RELOCATION` 结构，包括 32 位的 `VirtualAddress` 和 32 位的 `SizeOfBlock`. 随后是实际的重定位数据每个都是 16 位的。`VirtualAddress` 是本块要应用的基地址。`SizeOfBlock` 是整个块的大小。后面的重定位的数目是 (`SizeOfBlock` - sizeof(`IMAGE_BASE_RELOCATION`))/2，重定位信息以 `VirtualAddress` = 0 的 `IMAGE_BASE_RELOCATION` 结构结束。

每个 16 位重定位信息包括低 12 位的重定位位置和高 4 位的重定位类型。要得到重定位的 `RVA`，`IMAGE_BASE_RELOCATION` 的 `VirtualAddress` 需要加上 12 位位置偏移量。类型是下列之一：

- `IMAGE_REL_BASED_ABSOLUTE` (0) 使块按照 32 位对齐，位置为 0。
- `IMAGE_REL_BASED_HIGH` (1) 高 16 位必须应用于偏移量所指高字 16 位。
- `IMAGE_REL_BASED_LOW` (2) 低 16 位必须应用于偏移量所指低字 16 位。
- `IMAGE_REL_BASED_HIGHLOW` (3) 全部 32 位应用于所有 32 位。
- `IMAGE_REL_BASED_HIGHADJ` (4) 需要 32 位，高 16 位位于偏移量，低 16 位位于下一个偏移量数组元素，组合为一个带符号数，加上 32 位的一个数，然后加上 8000 然后把高 16 位保存在偏移量的 16 位域内。
- `IMAGE_REL_BASED_MIPS_JMPADDR` (5) Unknown
- `IMAGE_REL_BASED_SECTION` (6) Unknown
- `IMAGE_REL_BASED_REL32` (7) Unknown

举例：

```
0x00004000       (32 bits, starting RVA)  
0x00000010       (32 bits, size of chunk)  
0x3012           (16 bits reloc data)  
0x3080           (16 bits reloc data)  
0x30f6           (16 bits reloc data)  
0x0000           (16 bits reloc data)  
0x00000000       (next chunk's RVA)  
0xff341234  
```

第一块描述重定位起始于 `RVA` `0x4000` 长度 16 字节。因为头用去 8 字节，一个重定位用 2 个字节，总共（16－8)/2＝4 个重定位，第一个重定位被用于 `0x4012`，下一个重定位于 `4080`，第三个定位于 `0x40f6`。最后一个无用。最后一个结束。

# 附录：Appendix: hello world

The program will be the equivalent of

```
#include <stdio.h>  
int main(void)  
{  
    puts(hello, world);  
    return 0;  
}  
```

用 Win32 函数代替 C runtime:

```
#define STD_OUTPUT_HANDLE -11UL  
#define hello "hello, world\n"  
__declspec(dllimport) unsigned long __stdcall  
GetStdHandle(unsigned long hdl);  
__declspec(dllimport) unsigned long __stdcall  
WriteConsoleA(unsigned long hConsoleOutput,  
                const void *buffer,  
                unsigned long chrs,  
                unsigned long *written,  
                unsigned long unused  
                );  
static unsigned long written;  
void startup(void)  
{  
    WriteConsoleA(GetStdHandle(STD_OUTPUT_HANDLE), hello, sizeof(hello)-1, &written, 0);  
    return;  
}  
```

## 汇编语言

```
startup:  
        ; 参数 WriteConsole(), 逆向  
6A 00                  push      0x00000000  
68 ?? ?? ?? ??         push      offset _written  
6A 0D                  push      0x0000000d  
68 ?? ?? ?? ??         push      offset hello  
        ; 参数 GetStdHandle()  
6A F5                  push      0xfffffff5  
2E FF 15 ?? ?? ?? ??    call      dword ptr cs:__imp__GetStdHandle@4  
        ; result is last parameter for WriteConsole()  
50                     push      eax  
2E FF 15 ?? ?? ?? ??    call      dword ptr cs:__imp__WriteConsoleA@20  
C3                     ret        
hello:  
68 65 6C 6C 6F 2C 20 77 6F 72 6C 64 0A   "hello, world\n"  
_written:  
00 00 00 00  
```

## 下面是链接器：

我们需要找出函数 `WriteConsoleA()` and `GetStdHandle()`. 它们在 `".kernel32.dll"`. 此为输入库部分。我们现在生成可执行文件。问号在后面被找出。

### DOS-stub, 起始于 0x0，0x40 字节:

```
00 | 4d 5a 00 00 00 00 00 00 00 00 00 00 00 00 00 00  
10 | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  
20 | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  
30 | 00 00 00 00 00 00 00 00 00 00 00 00 40 00 00 00  
```

只是一个头，前面带有签字 `"MZ"` 后面是 `e_lfanew` 指针。然后是 PE 签名，起始于 `0x40`，0x4 字节:

```
50 45 00 00  
```

### 下面是文件头，起始于 0x44,0x14 字节:

```
Machine                   4c 01       ; i386  
NumberOfSections          02 00       ; code and data  
TimeDateStamp             00 00 00 00 ; who cares?  
PointerToSymbolTable      00 00 00 00 ; unused  
NumberOfSymbols           00 00 00 00 ; unused  
SizeOfOptionalHeader      e0 00       ; constant  
Characteristics          02 01       ; executable on 32-bit-machine  
```

### 下面是 optional header, 起始于 0x58，0x60 bytes long:

```
Magic                   0b 01       ; constant  
MajorLinkerVersion      00          ; I'm version 0.0 :-)  
MinorLinkerVersion      00          ;  
SizeOfCode              20 00 00 00 ; 32 bytes of code  
SizeOfInitializedData   ?? ?? ?? ?? ; yet to find out  
SizeOfUninitializedData 00 00 00 00 ; we don't have a BSS  
AddressOfEntryPoint     ?? ?? ?? ?? ; yet to find out  
BaseOfCode              ?? ?? ?? ?? ; yet to find out  
BaseOfData              ?? ?? ?? ?? ; yet to find out  
ImageBase               00 00 10 00 ; 1 MB, chosen arbitrarily  
SectionAlignment        20 00 00 00 ; 32-bytes-alignment  
FileAlignment           20 00 00 00 ; 32-bytes-alignment  
MajorOperatingSystemVersion  04 00      ; NT 4.0  
MinorOperatingSystemVersion  00 00      ;  
MajorImageVersion        00 00       ; version 0.0  
MinorImageVersion        00 00       ;  
MajorSubsystemVersion    04 00       ; Win32 4.0  
MinorSubsystemVersion    00 00       ;  
Win32VersionValue        00 00 00 00 ; unused?  
SizeOfImage             ?? ?? ?? ?? ; yet to find out  
SizeOfHeaders           ?? ?? ?? ?? ; yet to find out  
CheckSum                00 00 00 00 ; not used for non-drivers  
Subsystem               03 00       ; Win32 console  
DllCharacteristics      00 00       ; unused (not a DLL)  
SizeOfStackReserve      00 00 10 00 ; 1 MB stack  
SizeOfStackCommit       00 10 00 00 ; 4 KB to start with  
SizeOfHeapReserve       00 00 10 00 ; 1 MB heap  
SizeOfHeapCommit        00 10 00 00 ; 4 KB to start with  
LoaderFlags             00 00 00 00 ; unknown  
NumberOfRvaAndSizes     10 00 00 00 ; constant  
```

计划 2 个节，一个是代码，一个是数据（包括数据，常量和输入目录），没有重定位，没有其他资源。没有 BSS 段。变量 `written` 填入初始化数据。节对齐和文件对齐都是 32 字节。

### 现在建立数据目录，开始于 0xb8，0x80 字节长。只有引入目录被使用。

```
Address        Size  
00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_EXPORT (0)  
?? ?? ?? ??    ?? ?? ?? ??         ; IMAGE_DIRECTORY_ENTRY_IMPORT (1)  
00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_RESOURCE (2)  
00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_EXCEPTION (3)  
00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_SECURITY (4)  
00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_BASERELOC (5)  
00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_DEBUG (6)  
00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_COPYRIGHT (7)  
00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_GLOBALPTR (8)  
00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_TLS (9)  
```

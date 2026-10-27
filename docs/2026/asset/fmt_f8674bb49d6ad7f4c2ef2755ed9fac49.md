
```
0x30f6          (16 bits reloc data)
0x0000          (16 bits reloc data)
0x00000000      (next chunk's RVA)
0xff341234
```

第一块描述重定位起始于RVA `0x4000`长度16字节。因为头用去8字节，一个重定位用2个字节，总共（16－8)/2＝4个重定位，第一个重定位被用于`0x4012`，下一个重定位于`4080`，第三个定位于`0x40f6`。最后一个无用。最后一个结束。

# 附录：Appendix: hello world

---

The program will be the equivalent of

```
#include <stdio.h>

int main(void)
{
    puts(hello,world);
    return 0;
}
```

用Win32函数代替C runtime:

```
#define STD\_OUTPUT\_HANDLE -11UL
#define hello "hello, world\n"

\_\_declspec(dllimport) unsigned long \_\_stdcall
GetStdHandle(unsigned long hdl);

\_\_declspec(dllimport) unsigned long \_\_stdcall
WriteConsoleA(unsigned long hConsoleOutput,
                const void \*buffer,
                unsigned long chrs,
                unsigned long \*written,
                unsigned long unused
                );

static unsigned long written;

void startup(void)
{
    WriteConsoleA(GetStdHandle(STD\_OUTPUT\_HANDLE),hello,sizeof(hello)-1,&written,0);
    return;
}
```

汇编语言

```
startup:
            ; 参数WriteConsole(), 逆向

6A 00                     push      0x00000000

68 ?? ?? ?? ??            push      offset \_written

6A 0D                     push      0x0000000d

68 ?? ?? ?? ??            push      offset hello

            ; 参数GetStdHandle()

6A F5                     push      0xfffffff5

2E FF 15 ?? ?? ?? ??      call      dword ptr cs:\_\_imp\_\_GetStdHandle@4

            ; result is last parameter for WriteConsole()

50                        push      eax

2E FF 15 ?? ?? ?? ??      call      dword ptr cs:\_\_imp\_\_WriteConsoleA@20

C3                        ret

hello:

68 65 6C 6C 6F 2C 20 77 6F 72 6C 64 0A   "hello, world\n"

\_written:

00 00 00 00
```

下面是链接器：

我们需要找出函数`WriteConsoleA()`和`GetStdHandle()`。他们在`kernel32.dll`。此为输入库部分。我们现在生成可执行文件。问号在后面被找出。

DOS-stub, 起始于`0x0`，`0x40`字节:

```
00 | 4d 5a 00 00 00 00 00 00 00 00 00 00 00 00 00 00
10 | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
20 | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
30 | 00 00 00 00 00 00 00 00 00 00 00 00 40 00 00 00
```

只是一个头，前面带有签字`"MZ"`后面是`e_lfanew`指针。然后是PE签名，起始于`0x40`，`0x4`字节:

```
50 45 00 00
```

下面是文件头，起始于`0x44`, `0x14`字节:

```
Machine                     4c 01       ; i386
NumberOfSections            02 00       ; code and data
TimeDateStamp               00 00 00 00 ; who cares?
PointerToSymbolTable        00 00 00 00 ; unused
NumberOfSymbols             00 00 00 00 ; unused
SizeOfOptionalHeader        e0 00       ; constant
Characteristics             02 01       ; executable on 32-bit-machine
```

下面是optional header,起始于`0x58`，`0x60` bytes long:

```
Magic                       0b 01       ; constant
MajorLinkerVersion          00          ; I'm version 0.0 :-)
MinorLinkerVersion          00          ;
SizeOfCode                  20 00 00 00 ; 32 bytes of code
SizeOfInitializedData       ?? ?? ?? ?? ; yet to find out
SizeOfUninitializedData     00 00 00 00 ; we don't have a BSS
AddressOfEntryPoint         ?? ?? ?? ?? ; yet to find out
BaseOfCode                  ?? ?? ?? ?? ; yet to find out
BaseOfData                  ?? ?? ?? ?? ; yet to find out
ImageBase                   00 00 10 00 ; 1 MB, chosen arbitrarily
SectionAlignment            20 00 00 00 ; 32-bytes-alignment
FileAlignment               20 00 00 00 ; 32-bytes-alignment
MajorOperatingSystemVersion  04 00      ; NT 4.0
MinorOperatingSystemVersion  00 00      ;
MajorImageVersion           00 00       ; version 0.0
MinorImageVersion           00 00       ;
MajorSubsystemVersion       04 00       ; Win32 4.0
MinorSubsystemVersion       00 00       ;
Win32VersionValue           00 00 00 00 ; unused?
SizeOfImage                 ?? ?? ?? ?? ; yet to find out
SizeOfHeaders               ?? ?? ?? ?? ; yet to find out
CheckSum                    00 00 00 00 ; not used for non-drivers
Subsystem                   03 00       ; Win32 console
DllCharacteristics          00 00       ; unused (not a DLL)
SizeOfStackReserve          00 00 10 00 ; 1 MB stack
SizeOfStackCommit           00 10 00 00 ; 4 KB to start with
SizeOfHeapReserve           00 00 10 00 ; 1 MB heap
SizeOfHeapCommit            00 10 00 00 ; 4 KB to start with
LoaderFlags                 00 00 00 00 ; unknown
NumberOfRvaAndSizes         10 00 00 00 ; constant
```

计划2个节，一个是代码，一个是数据（包括数据，常量和输入目录），没有重定位，没有其他资源。没有BSS段。变量`written`填入初始化数据。节对齐和文件对齐都是32字节。

现在建立数据目录，开始于`0xb8`，`0x80`字节长。只有引入目录被使用。

```
Address        Size

00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_EXPORT (0)

?? ?? ?? ??    ?? ?? ?? ??         ; IMAGE\_DIRECTORY\_ENTRY\_IMPORT (1)

00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_RESOURCE (2)

00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_EXCEPTION (3)

00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_SECURITY (4)

00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_BASERELOC (5)

00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_DEBUG (6)

00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_COPYRIGHT (7)

00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_GLOBALPTR (8)

00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_TLS (9)

00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_LOAD\_CONFIG (10)

00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_BOUND\_IMPORT (11)

00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_IAT (12)

00 00 00 00    00 00 00 00         ; 13

00 00 00 00    00 00 00 00         ; 14

00 00 00 00    00 00 00 00         ; 15
```

下面是节头，首先是代码节，包括上述汇编，32字节长，开始于`0x138`，长度x28：

```
Name            2e 63 6f 64 65 00 00 00     ; ".code"
VirtualSize         00 00 00 00             ; unused
VirtualAddress      ?? ?? ?? ??             ; yet to find out
SizeOfRawData       20 00 00 00             ; size of code
PointerToRawData    ?? ?? ?? ??             ; yet to find out
PointerToRelocations 00 00 00 00            ; unused
PointerToLinenumbers 00 00 00 00            ; unused
NumberOfRelocations  00 00                  ; unused
NumberOfLinenumbers  00 00                  ; unused
Characteristics     20 00 00 60             ; code, executable, readable
```

第二个节包括数据，起始于`0x160`，长度x28：

```
Name            2e 64 61 74 61 00 00 00     ; ".data"
VirtualSize         00 00 00 00             ; unused
VirtualAddress      ?? ?? ?? ??             ; yet to find out
SizeOfRawData       ?? ?? ?? ??             ; yet to find out
PointerToRawData    ?? ?? ?? ??             ; yet to find out
PointerToRelocations 00 00 00 00            ; unused
PointerToLinenumbers 00 00 00 00            ; unused
NumberOfRelocations  00 00                  ; unused
NumberOfLinenumbers  00 00                  ; unused
Characteristics     40 00 00 c0             ; initialized, readable, writeable
```

下一个是`0x188`,由于节必须按照32字节对齐，所有填0到`0x1a0`:

```
00 00 00 00 00 00       ; padding
00 00 00 00 00 00
00 00 00 00 00 00
00 00 00 00 00 00
```

现在第一节代码节来到了，开始于`0x1a0`，`0x20`字节长。

```
6A 00                    ; push      0x00000000
68 ?? ?? ?? ??           ; push      offset \_written
6A 0D                    ; push      0x0000000d
68 ?? ?? ?? ??           ; push      offset hello\_string
6A F5                    ; push      0xfffffff5
2E FF 15 ?? ?? ?? ??     ; call      dword ptr cs:\_\_imp\_\_GetStdHandle@4
50                       ; push      eax
2E FF 15 ?? ?? ?? ??     ; call      dword ptr cs:\_\_imp\_\_WriteConsoleA@20
C3                       ; ret
```


# 你知道第一块描述的重定位开始于 RVA 0x4000 处，有 16 字节长。因为头用掉了 8 字节，并且一个重定位要用 2 字节，所以块中计有 `(16-8)/2=4` 个重定位。第一个重定位被应用于 `0x4012` 处的 `DWORD`，第二个于 `0x4080` 处的 `DWORD`，第三个于 `0x40f6` 处的 `DWORD`。最后一个不需操作。

下一块的 `RVA` 是 `0`，列表结束。

好，你怎么处理一个重定位呢？

你能知道映象文件“被”重定位到可选头“`ImageBase`（映象基址）”的优先载入地址；你也能知道你真正载入的地址。如果它们相同，你什么也不用做。如果它们不同，你需计算出实际基址-优先基址的差并加上重定位位置的值（有符号，可能为负值），此值你可通过上面讲述的方法找到。

## 九、致谢（Acknowledgments）

感谢 David Binette 的调试和校读。（剩下的错误全部都是我的。）

也感谢 wotsit.org 网站让我将此文放到他们的网站上。

## 十、版权（Copyright）

本文的版权属于 B. Luevelsmeyer，1999 年。它是免费的，你可以任意的使用，但后果自负。它含有错误并不完整，特此警告。

## 十一、Bug 报告（Bug reports）

Bug 报告（或其他建议）请发送至：`bernd.luevelsmeyer@iplan.heitec.net`

## 十二、版本（Versions）

你可在文件的顶部找到当前的版本号。

- 1998-04-06
  - 第一次公开发表
- 1998-07-29
  - 将映象文件版本和子系统版本中错误的“`byte`”改为“`word`”
  - 更正“栈只限于 1 MB”的错误（实际上没有上限）
  - 更正一些输入错误
- 1999-03-15
  - 更正输出目录的描述，原来非常不全
  - 调整输入目录的描述，原来讲的不清
  - 更正输入错误并为其它节改了一些词句

## 十三、参考文献（Literature）

1. "Peering Inside the PE: A Tour of the Win32 Portable Executable File Format" (M. Pietrek), in: Microsoft Systems Journal 3/1994
2. "Why to Use _declspec(dllimport) & _declspec(dllexport) In Code", MS Knowledge Base Q132044
3. 《Windows 问与答》 "Windows Q&A" (M. Pietrek), in: Microsoft Systems Journal 8/1995
4. 《编写多语言资源》 "Writing Multiple-Language Resources", MS Knowledge Base Q89866
5. "The Portable Executable File Format from Top to Bottom" (Randy Kath), in: Microsoft Developer Network
6. 《Windows 下 TIS 格式规范 1.0 版》 Tool Interface Standard (TIS) Formats Specification for Windows Version 1.0 (Intel Order Number 241597, Intel Corporation 1993)

## 附录（Appendix: hello world）

在这个附录中我将给大家展示一下怎样手工建立一个程序。因为我不会 DEC Alpha 的，本例将使用 Intel 汇编语言。

本程序相当于

```
#include <stdio.h>
int main(void)
{
    puts(hello,world);
    return 0;
}
```

首先，我使用 Win32 函数来翻译它以取代 C 运行时库：

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
    WriteConsoleA(GetStdHandle(STD_OUTPUT_HANDLE),hello,sizeof(hello)-1,&written,0);
    return;
}
```

现在我将笨拙的将它汇编出来：

```
startup:
                ; WriteConsole()的参数, 反向的
6A 00                   push      0x00000000
68 ?? ?? ?? ??          push      offset _written
6A 0D                   push      0x0000000d
68 ?? ?? ?? ??          push      offset hello
                ; GetStdHandle()的参数
6A F5                   push      0xfffffff5
2E FF 15 ?? ?? ?? ??    call      dword ptr cs:__imp__GetStdHandle@4
                ; 结果是 WriteConsole()的参数
50                      push      eax
2E FF 15 ?? ?? ?? ??    call      dword ptr cs:__imp__WriteConsoleA@20
C3                      ret        
hello:
68 65 6C 6C 6F 2C 20 77 6F 72 6C 64 0A   "hello, world\n"
_written:
00 00 00 00
```

以上就是编译的部分。任何人都能做到这点。从现在起让我们扮演起链接器的角色，这会非常有趣 :-)

我需要先找出函数 `WriteConsoleA()` 和 `GetStdHandle()`。碰巧它们都在“`kernel32.dll`”中。（这是“输入库”部分。）

现在我开始做可执行文件。问号代表待定的值；它们将在以后被修正。

首先是 DOS-根，开始于 `0x0`，有 `0x40` 字节长：

```
00 | 4d 5a 00 00 00 00 00 00 00 00 00 00 00 00 00 00
10 | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
20 | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
30 | 00 00 00 00 00 00 00 00 00 00 00 00 40 00 00 00
```

正如你所见到的，这不是真正的 MS-DOS 程序。它只是一个开始部分有“`MZ`”签名的头和紧跟在头后面的 `e_lfanew` 指针，没有任何代码。这是因为它并非打算运行于 MS-DOS 之上；它之所以在这里只是因为规范的需要。

然后是 PE 签名，开始于 `0x40`，有 `0x4` 字节长：

```
50 45 00 00
```

现在到了文件头，开始于 `0x44`，有 `0x14` 字节长：

```
Machine                     4c 01       ; i386
NumberOfSections            02 00       ; 代码段和数据段
TimeDateStamp               00 00 00 00 ; 谁管它？
PointerToSymbolTable        00 00 00 00 ; 未用
NumberOfSymbols             00 00 00 00 ; 未用
SizeOfOptionalHeader        e0 00       ; 常量
Characteristics             02 01       ; 32 位机器上的可执行文件
```

接着是可选头，开始于 `0x58`，有 `0x60` 字节长：

```
Magic                       0b 01       ; 常量
MajorLinkerVersion          00          ; 我是 0.0 版:-)
MinorLinkerVersion          00          ;
SizeOfCode                  20 00 00 00 ; 32 字节代码
SizeOfInitializedData       ?? ?? ?? ?? ; 待找出
SizeOfUninitializedData     00 00 00 00 ; 我们没有 BSS 节
AddressOfEntryPoint         ?? ?? ?? ?? ; 待定
BaseOfCode                  ?? ?? ?? ?? ; 待定
BaseOfData                  ?? ?? ?? ?? ; 待定
ImageBase                   00 00 10 00 ; 1 MB, 随意选
SectionAlignment            20 00 00 00 ; 32 字节对齐
FileAlignment               20 00 00 00 ; 32 字节对齐
MajorOperatingSystemVersion 04 00      ; NT 4.0
MinorOperatingSystemVersion 00 00      ;
MajorImageVersion           00 00       ; 0.0 版
MinorImageVersion           00 00       ;
MajorSubsystemVersion       04 00       ; Win32 4.0
MinorSubsystemVersion       00 00       ;
Win32VersionValue           00 00 00 00 ; 未使用?
SizeOfImage                 ?? ?? ?? ?? ; 待定
SizeOfHeaders               ?? ?? ?? ?? ; 待定
CheckSum                    00 00 00 00 ; 非驱动不用
Subsystem                   03 00       ; Win32 控制台
DllCharacteristics          00 00       ; 未用 (不是一个 DLL)
SizeOfStackReserve          00 00 10 00 ; 1 MB 栈
SizeOfStackCommit           00 10 00 00 ; 开始时 4 KB
SizeOfHeapReserve           00 00 10 00 ; 1 MB 堆
SizeOfHeapCommit            00 10 00 00 ; 开始时 4 KB
LoaderFlags                 00 00 00 00 ; 未知
NumberOfRvaAndSizes         10 00 00 00 ; 常量
```

正如你所见，我计划只用 2 个节，一个用于代码，一个用于所有剩余的东西（数据、常量和输入目录等）。没有重定位和象资源之类其它东西。我也不用 BSS 节并将变量“`written`”放入已初始化数据。文件和 RAM 中的节对齐都是一样的（32 字节）；这将有助于使任务简单，否则我就得来回地计算 `RVA` 很多次。

现在我们设置数据目录，开始于 `0xb8` 字节，有 `0x80` 字节长:

```
       地址            大小
00 00 00 00     00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_EXPORT (0)
?? ?? ?? ??     ?? ?? ?? ??         ; IMAGE_DIRECTORY_ENTRY_IMPORT (1)
00 00 00 00     00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_RESOURCE (2)
00 00 00 00     00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_EXCEPTION (3)
00 00 00 00     00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_SECURITY (4)
00 00 00 00     00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_BASERELOC (5)
00 00 00 00     00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_DEBUG (6)
00 00 00 00     00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_COPYRIGHT (7)
00 00 00 00     00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_GLOBALPTR (8)
00 00 00 00     00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_TLS (9)
00 00 00 00     00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG (10)
00 00 00 00     00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT (11)
00 00 00 00     00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_IAT (12)
00 00 00 00     00 00 00 00         ; 13
00 00 00 00     00 00 00 00         ; 14
00 00 00 00     00 00 00 00         ; 15
```

仅使用输入目录。

下一个使节头。首先我们做代码节的，代码节将包含前面所编的汇编语句。它有 32 字节长，所以代码节也就是这么长。节头从 `0x138` 处开始，有 `0x28` 字节长：

```
Name                2e 63 6f 64 65 00 00 00     ; ".code"的 ASCII 码值
VirtualSize           00 00 00 00               ; 未用
VirtualAddress        ?? ?? ?? ??               ; 待定
```

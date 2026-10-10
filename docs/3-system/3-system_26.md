

# 一个 coff 文件格式的观察

> 原文：[`www.pediy.com/kssd/pediy10/74060.html`](https://www.pediy.com/kssd/pediy10/74060.html)

一个 coff 文件格式的观察   
                                      -- 2008-10-4 ljhhh0123  
  
    我采用自底向上的方法来学习 coff 文件格式,并体会汇编器的行为,  
这是一个好方法.用 vc6.0sp6 编译的 lcc 将 hello1.c  
编译成汇编语言,再用 masm32v8 的汇编器得到的.obj 文件,倾印下来,  
本文的二进制字节序为 little-endlian.  
  
    基本工具:  记事本+WinHex+ida+各种 pe&coff 学习资料  
  
    基本方法是: 选用 winhex 打开.obj 文件,将内容以二进制方式复制到  
记事本,然后就是边看资料,边从 winhex 上找到位置,边在记事本里断行和加  
注释,另外还动用了 masm32v8 自带的工具 dumpbin.exe 和 ida 来找到各部分之  
间的分界点.最后的结果如下.  
  
\-----可选文件头----  
  
4C01           CPU 识别码  
0200           段数  
43C5E548　日期戳  
A6000000    符号表指针  
0E000000    符号个数  
0000           可选头大小  
0000           标志   
  
\----段表--2 项--  
  
2E74657874000000 .text\\0\\0\\0\\0  
00000000               第一个段的位置  
00000000               在 obj 中总为 0  
2E000000               段大小  
64000000               段的起始位置，相对于 obj 文件  
92000000               重定位信息的位置  
00000000               行号表的位置  
0200                      重定位项目数  
0000                      行号个数  
20005060               段标志字   IMAGE\_SCN\_MEM\_EXECUTE | IMAGE\_SCN\_MEM\_READ  
                                 | IMAGE\_SCN\_ALIGN\_16BYTES | IMAGE\_SCN\_CNT\_CODE    
  
2E64617461000000 .data\\0\\0\\0  
2E000000               第二个段的位置(第一个段起址加上第一个段的大小)  
00000000  
00000000  
00000000  
00000000  
00000000  
0000  
0000  
400050C0             IMAGE\_SCN\_MEM\_READ | IMAGE\_SCN\_MEM\_WRITE    
                     | IMAGE\_SCN\_ALIGN\_16BYTES   
                     | E\_SCN\_CNT\_INITIALIZED\_DATA   
  
\----具体的段.text 长度 0x2E----  
  
535657558BEC8D3D0000000057E80000  
000083C404B8000000008BE55D5F5E5B  
C348656C6C6F20576F726C642100  
  
\----重定位表----2 项----  
  
08000000              段内偏移  
0C000000              在符号表内的索引  
0600                  属性  重定位目标的 32 位虚拟地址  
  
0E000000  
0B000000              在符号表内的索引  
1400                  属性  重定位目标的 32 位相对偏移  
                              
  
\----符号表--0xe 项--  
  
2E66696C65000000".file\\0\\0\\0"  
00000000  
FEFF                     只是个调试符号  
0000  
67                       源文件符号记录。后面跟着给出文件名的辅助符号表记录。  
03                       附加记录数,就是下面三项数据不作具体的符号用.  
  
433A5C444F43554D  
457E315C6C6A685C4C4F  
43414C537E315C54  
656D705C6C6363313435  
32312E61736D0000  
00000000000000000000  
  
40636F6D702E6964        "@comp.id"  
FC201200  
FFFF                    这个符号值是个常量.  
0000  
03                      存储类型为 static  
00  
  
2E74657874000000       ".text\\0\\0\\0"  
00000000  
0100                   属于第一个段  
0000  
03  
01                     附加记录 1 项  
2E00000002000000  
00000000000000000000  
  
2E64617461000000        ".data\\0\\0\\0"  
00000000  
0200                   段号,即第二个段  
0000  
03  
01                     附加记录 1 个  
0000000000000000  
00000000000000000000  
  
00000000  
04000000                以字符串表为开始,偏移四的位置取出.  
00000000  
0000  
2000                    函数类型  
02                      external  
00  
  
5F5F66746F6C0000   "\_\_ftol\\0\\0"  
00000000  
0000  
2000                函数类型  
02                  external  
00  
5F70757473000000   "\_puts\\0\\0\\0"  
00000000  
0000  
2000                          
02                   external  
00  
4C33000000000000   "L3"  
21000000  
0100                 符号所在段的索引  
0000  
03                   static  
00  
5F6D61696E000000 "\_main\\0\\0\\0"  
00000000  
0100                 段号  
2000                 函数类型  
02                   external  
00  
  
\----字符串表----  
  
0E000000             总长度=本长度占的 4 字节+以 0 结尾的字符串的长度(包括 0)  
5F5F666C747573656400  "\_\_fltused\\0"  
  
\--------  
附录 A 为开发环境所带的数据结构定义,  
附录 B 为 masm32v8 下所执行的"dumpbin.exe /all hello1.obj"列印.  
附录 C 为 hello1.c 文件和 lcc 生成的汇编语言文件倾印.  
  
注:将些文章内的二进制数据复制到 winhex 内即可还原成 hello1.obj 文件.大小为 0x1b0.  
  
附录 A:  
typedef struct \_IMAGE\_FILE\_HEADER {  
    WORD    Machine;  
    WORD    NumberOfSections;  
    DWORD   TimeDateStamp;  
    DWORD   PointerToSymbolTable;  
    DWORD   NumberOfSymbols;  
    WORD    SizeOfOptionalHeader;  
    WORD    Characteristics;  
} IMAGE\_FILE\_HEADER, \*PIMAGE\_FILE\_HEADER;  
  
typedef struct \_IMAGE\_SECTION\_HEADER {  
    BYTE    Name\[IMAGE\_SIZEOF\_SHORT\_NAME\];  
    union {  
            DWORD   PhysicalAddress;  
            DWORD   VirtualSize;  
    } Misc;  
    DWORD   VirtualAddress;  
    DWORD   SizeOfRawData;  
    DWORD   PointerToRawData;  
    DWORD   PointerToRelocations;  
    DWORD   PointerToLinenumbers;  
    WORD    NumberOfRelocations;  
    WORD    NumberOfLinenumbers;  
    DWORD   Characteristics;  
} IMAGE\_SECTION\_HEADER, \*PIMAGE\_SECTION\_HEADER;  
  
typedef struct \_IMAGE\_RELOCATION {  
    union {  
        DWORD   VirtualAddress;  
        DWORD   RelocCount;               
    };  
    DWORD   SymbolTableIndex;  
    WORD    Type;  
} IMAGE\_RELOCATION;  
  
typedef struct \_IMAGE\_SYMBOL {  
    union {  
        BYTE    ShortName\[8\];  
        struct {  
            DWORD   Short;     // if 0, use LongName  
            DWORD   Long;      // offset into string table  
        } Name;  
        PBYTE   LongName\[2\];  
    } N;  
    DWORD   Value;  
    SHORT   SectionNumber;  
    WORD    Type;  
    BYTE    StorageClass;  
    BYTE    NumberOfAuxSymbols;  
} IMAGE\_SYMBOL;  
  
附录 B:  
Microsoft (R) COFF Binary File Dumper Version 5.12.8078  
Copyright (C) Microsoft Corp 1992-1998. All rights reserved.  
  
Dump of file hello1.obj  
  
File Type: COFF OBJECT  
  
FILE HEADER VALUES  
             14C machine (i386)  
               2 number of sections  
        48E5C543 time date stamp Fri Oct 03 15:09:55 2008  
              A6 file pointer to symbol table  
               E number of symbols  
               0 size of optional header  
               0 characteristics  
  
SECTION HEADER #1  
   .text name  
       0 physical address  
       0 virtual address  
      2E size of raw data  
      64 file pointer to raw data  
      92 file pointer to relocation table  
       0 file pointer to line numbers  
       2 number of relocations  
       0 number of line numbers  
60500020 flags  
         Code  
         16 byte align  
         Execute Read  
  
RAW DATA #1  
  00000000: 53 56 57 55 8B EC 8D 3D 00 00 00 00 57 E8 00 00  SVWU...=....W...  
  00000010: 00 00 83 C4 04 B8 00 00 00 00 8B E5 5D 5F 5E 5B  ............\]\_^\[  
  00000020: C3 48 65 6C 6C 6F 20 57 6F 72 6C 64 21 00        .Hello World!.  
  
RELOCATIONS #1  
                                                Symbol    Symbol  
 Offset    Type              Applied To         Index     Name  
 --------  ----------------  -----------------  --------  ------  
 00000008  DIR32                      00000000         C  L3  
 0000000E  REL32                      00000000         B  \_puts  
  
SECTION HEADER #2  
   .data name  
      2E physical address  
       0 virtual address  
       0 size of raw data  
       0 file pointer to raw data  
       0 file pointer to relocation table  
       0 file pointer to line numbers  
       0 number of relocations  
       0 number of line numbers  
C0500040 flags  
         Initialized Data  
         16 byte align  
         Read Write  
  
COFF SYMBOL TABLE  
000 00000000 DEBUG  notype       Filename     | .file  
    C:\\DOCUME~1\\ljh\\LOCALS~1\\Temp\\lcc14521.asm  
004 001220FC ABS    notype       Static       | @comp.id  
005 00000000 SECT1  notype       Static       | .text  
    Section length   2E, #relocs    2, #linenums    0, checksum        0  
007 00000000 SECT2  notype       Static       | .data  
    Section length    0, #relocs    0, #linenums    0, checksum        0  
009 00000000 UNDEF  notype ()    External     | \_\_fltused  
00A 00000000 UNDEF  notype ()    External     | \_\_ftol  
00B 00000000 UNDEF  notype ()    External     | \_puts  
00C 00000021 SECT1  notype       Static       | L3  
00D 00000000 SECT1  notype ()    External     | \_main  
  
String Table Size = 0xE bytes  
  
Summary  
          0 .data  
        2e .text  
  
附录 C: hello1.c and hello1.asm  
  
#include "stdio.h" /\*借用 djgpp 的 include 目录.\*/  
main()  
{puts("Hello World!");  
}  
  
.486  
.model flat  
extrn \_\_fltused:near  
extrn \_\_ftol:near  
public \_main  
\_TEXT segment  
\_main:  
push ebx  
push esi  
push edi  
push ebp  
mov ebp,esp  
lea edi,(L3)  
push edi  
call \_puts  
add esp,4  
mov eax,0  
L2:  
mov esp,ebp  
pop ebp  
pop edi  
pop esi  
pop ebx  
ret  
\_TEXT ends  
extrn \_puts:near  
\_TEXT segment  
\_TEXT ends  
\_TEXT segment  
align 1  
L3 label byte  
db 72  
db 101  
db 108  
db 108  
db 111  
db 32  
db 87  
db 111  
db 114  
db 108  
db 100  
db 33  
db 0  
\_TEXT ends  
end  
  
\----本文完毕----
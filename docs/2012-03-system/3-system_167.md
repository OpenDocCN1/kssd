
# 最小 PE 133 bytes

> 原文：[`www.pediy.com/kssd/pediy10/85469.html`](https://www.pediy.com/kssd/pediy10/85469.html)

发现个最小的 PE 文件

比以前这个帖子的更精简

`http://bbs.pediy.com/showthread.php?t=48590`

因为比正常的 PE 文件少了很多结构

看了半天没明白是怎么加载的

求大侠指点

```
00000000h: 4D 5A FF FF 50 45 00 00 4C 01 01 00 75 73 65 72 ; MZPE..L...user
00000010h: 33 32 2E 64 6C 6C 00 FF 40 00 0F 01 0B 01 FF FF ; 32.dll.@.....
00000020h: 4D 65 73 73 61 67 65 42 00 44 00 00 00 ; MessageBoxA.D...
00000030h: FF 15 7C 00 40 00 C3 FF 00 00 40 00 04 00 00 00 ; .|.@.?..@.....
00000040h: 04 00 00 00 B8 20 00 40 00 EB 03 FF 04 00 6A 41 ; ....?.@.?..jA
00000050h: 50 50 EB 20 89 00 00 00 85 00 00 00 00 00 00 00 ; PP??..?......
00000060h: 02 00 FF 00 7C 00 00 00 0C 00 00 00 7C 00 00 00 ; ...|.......|...
00000070h: 0C 00 00 00 6A 00 EB B8 02 00 00 00 1E 00 00 00 ; ....j.敫........
00000080h: 00 00 00 00 5C                                      ; ....\\
```

引用: `cham` 发布 ![查看帖子](http://bbs.pediy.com/showthread.php?p=602035#post602035) 不知是不是这个 `http://blog.csdn.net/njhhack/archive/2008/02/25/2120019.aspx`

好像是的

```
;goto end

;****************************************************************

;*        133 字节，带 MessageBOX 信息框的 PE 程序 for  2k/xp         *

;*        Tasm /m spe.txt                                       *

;*        Tlink /3 /t spe, spe.exe                              *

;*        njhhack 2007.8.30 制作                                 *

;*        qq:10772919  e-mail:njhhack@126.com                   *

;****************************************************************

                  .386p

IBase=400000h            ;基地址

Section           SEGMENT        ;

ASSUME CS:Section

BCode:

       ;****************DOS Header****************

DosSignature        dw 5a4dh        ;MZ 标志

                dw 0ffffh

       ;****************PE File Header****************

WinSignature        dd 4550h        ;PE 标志

Machine             dw 014ch        ;Intel 80386

NumberOfSections    dw 1            ;节段数

user32              db "user32.dll",0,0ffh

 ;TimeDateStamp          dd 0            ;任意数值

 ;PointerToSymbolTable   dd 0            ;任意数值

 ;NumberOfSymbols       dd 0            ;任意数值

SizeOfOptionalHeader    dw OHSize         ;可选头部大小

Characteristics       dw 010fh          ;机器标志

          ;****************Optional Header****************

Magic               dw 10bh           ;

LinkerVersion        dw 0ffffh          ;任意数值

MessafeBoxA           db "MessageBoxA",0

 ;SizeOfCode            dd 0            ;任意数值

 ;SizeOfInitializedData  dd 0            ;任意数值

 ;SizeOfUninitializedData dd 0           ;任意数值

AddressOfEntryPoint   dd start          ;初始代码地址

next3:

dw 15ffh

dd  IBase+IAT1

ret

db 0ffh

 ;BaseOfCode            dd 0            ;任意数值

 ;BaseOfData            dd 0            ;任意数值

ImageBase            dd IBase          ;映象加载地址 1 MB

SectionAlignment      dd 4             ;PE 标志(3ch 位置=从 4 开始)

FileAlignment         dd 4             ;

start:

mov eax,offset IBase+MessafeBoxA

jmp short next1

db 0ffh

 ;OperatingSystemVersion   dd 0ffffffffh ;NT 4.0(l.h)

 ;ImageVersion           dd 0ffffffffh ;任意数值

 dw 4

next1:

push 65

push eax

push eax

jmp short next2

 ;SubsystemVersion       dd 0ffff0004h  ;Win32 4.0

 ;Win32VersionValue      dd 0ffffffffh ;任意数值

SizeOfImage           dd endimage       ;任意数值,要求大于 SizeOfHeaders

SizeOfHeaders         dd endhead        ;文件头大小

 OHSize=$-Magic

 IAT:

CheckSum             dd 0            ;任意数值,                OriginalFirstThunk

Subsystem            dw 2            ;                          TimeDateStamp

DllCharacteristics   dw 0ffh          ;

SizeOfStackReserve    dd IAT1          ;任意数值,(Virtual Size),    ForwarderChain

SizeOfStackCommit     dd user32        ;任意数值,(Virtual Address),Name1

SizeOfHeapReserve     dd IAT1          ;任意数值,(Raw Data Size),   FirstThunk

SizeOfHeapCommit      dd user32        ;任意数值,(Raw Data Offset)

next2:

 push 0

 jmp short next3

 ;LoaderFlags          dd 0ffffffffh ;任意数值

NumberOfRvaAndSizes    dd 2h           ;

      ;****************Image Data Directories

 IAT1:

Ide00_Export           dd MessafeBoxA-2,0 ;

Ide01_Import           db IAT-BCode       ;

endhead=$

endimage=endhead+4

endc=$

Section             ENDS

                END

;:end

;del tiny.exe

;Tasm /m spe.bat

;Tlink /3 /t spe.obj,tiny.exe

;del tiny.map

;del spe.obj
```

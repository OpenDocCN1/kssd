
# PEDIY 技术之新思路（二）\_ 用'高级'编译器 MASM 实现自定义 PE 文件结构

> 原文：[`www.pediy.com/kssd/pediy10/86657.html`](https://www.pediy.com/kssd/pediy10/86657.html)

【文章标题】: PEDIY 技术之新思路（二）\_ 用'高级'编译器 MASM 实现自定义 PE 文件结构  

【文章作者】: moonife  

【作者邮箱】: moonife@163.com  

【作者 QQ 号】: 765496322  

【下载地址】: 附件  

【编写语言】: `win32 asm`  

【使用工具】: `MASM`  

【操作平台】: `xp-sp3`  

【作者说明】: 这两天被一个不知道是加了什么壳的东西搞郁闷了，所以一直想找个有意思的东东爽一下心情 ，把乐趣融入到学习当中，这个很重要！还有就是我写了 PEDIY 技术之新思路（一），怎么说都得在表示一下吧！今天学习了 `xfish` 的《PE 结构，SEH 相关知识掌握》一文，心理面就特痒痒，也想完整自定义一个 PE 程序，但是我习惯的 `MASM` 却没有 `FASM` 那样可以自定义 PE 结构（一般高级编译器都是编译好的 PE 头部,例如 `MASM`,`TASM` 等，一直都说 `NASM`,`FASM` 是低级编译器，可以自定义 PE 结构）的功能，想过装一个 `FASM`，但是很多习惯就得改，这样的话，我就不想干了，那我可不可以用 `MASM` 实现这样的定义呢？当然可以了，事实上，是上一篇文章给了我这样的思路，就是利用 `MASM` 编译器变通的编译二进制型汇编代码来实现，自觉还有点新意，所以就拿出来分享一下！好了，不说了，下面看实现过程，不足的地方还请高手指正，谢谢！  

`* * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * 实现过程 * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * *`

## 一：定义 PE 文件结构程序的实现

`* * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * pe.asm * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * *`

```asm
REMOTE_CODE_START  equ this BYTE 

;**************************PE_HEADER_START**************************

PE_HEADER_START equ this BYTE     

DOS_HEADER:                 ;>>>>>>>>>>>>>DOS 头部  
e_magic    db  'MZ'                 ;-----------------------------说明---------------------------------------------  
e_cblp          dw     0        ;事实上，从这里到 e_res2 无用部分可以用一句 DUP(0)来完成，在这里编译器不会检查  
e_cp            dw     0        ;你结构的定义是否正确，这些变量名也是没有意义的，这样写只是有利于我们认识  
e_crlc          dw     0        ;PE 结构而已，除了关键字，你可以任意写，因为它相当于写二进制型汇编代码  
e_cparhdr       dw     0        ;事实上，和直接写二进制是一样一样的，只是这样就轻松多了。当然了，  
e_minalloc      dw     0        ;这种方式的定义给我们带来的问题就是重定位，所以需要重定位的地方要小心了,  
e_maxalloc      dw     0        ;你也可以把 PE 头部提上来到 DOS 头部里面来变形 PE 头，或者整个最小 PE 什么的，  
e_ss            dw     0        ;随你怎么高兴怎么玩，但是要保证 e_lfanew 定位的正确，你想怎么来就怎么来，你自由了！  
e_sp            dw     0        ;但是，没有绝对的自由，俗话说得好，任悟空本领再高，他也跳不出如来佛(OS)的手掌心啊  
e_csum          dw     0        ;------------------------------------------------------------------------------  
e_ip            dw     0  
e_cs            dw     0  
e_lfarlc          dw     0  
e_ovno          dw     0  
e_res           dw   4 dup(0)  
e_oemid         dw     0  
e_oeminfo       dw     0  
e_res2          dw  10 dup(0)  
e_lfanew        dd  NT_HEADERS-401000h  

Dos_Stub:                 ;>>>>>>>>>>>>>>>>>>DOS 小程序，这个写不写都行  
  mov ah, 4ch               
  int 21h  

NT_HEADERS:                         ;>>>>>>>>>>>>>PE 文件头  
Signature    dd  4550h     ;'PE'  
Machine    dw  14ch     ;.Inten I386    
NumberOfSections   dw  2  ;定义了两个节区  
TimeDateStamp    dd  0  
PointerToSymbolTable  dd  0  
NumberOfSymbols  dd  0   
SizeOfOptionalHeader  dw  0E0h  
Characteristics  dw  010fh    ;.exe Signature  
Magic     dw  10Bh   ;.exe file  
MajorLinkerVersion  db  0  
MinorLinkerVersion  db  0  
SizeOfCode    dd  200h  
SizeOfInitializedData  dd  0  
SizeOfUninitializedData dd  0  
AddressOfEntryPoint  dd  1000h ;oep  
BaseOfCode    dd  1000h  ;Code Section Rva  
BaseOfData     dd  0    ;Data Section Rva  
ImageBase    dd  400000h  
SectionAlignment  dd  1000h  ;Section Mem Align  
FileAlignment    dd  200h  ;Section Disk Align  
MajorOperSystemVersion dw  0    ;system version 主  
MinorOperSystemVersion dw  0    ;system version 次  
MajorImageVersion  dw  0    ;user version   主  
MinorImageVersion  dw  0    ;user version   次  
MajorSubsystemVersion  dw  4  
MinorSubsystemVersion  dw  0  
Win32VersionValue  dd  0    ;Reserved 0  
SizeOfImage    dd  3000h     ;pe 加载到内存后的映像大小  
SizeOfHeaders    dd  200h  ;DosHeader + DosStub + NtHeader + Section Header  
_CheckSum    dd  0  
SubSystem    dw  2    ;Gui  
DllCharacteristics  dw  0  
SizeOfStackReserve  dd  100000h     
SizeOfStackCommit  dd  1000h    ;Stack = 4kb  
SizeOfHeapReserve  dd  100000h  
SizeOfHeapCommit  dd  1000h    ;Heap = 4kb  
LoaderFlags    dd  0  
NumberOfRvaAndSizes  dd  10h    ;16  
DirectoryData1 dq 0       ;输出表填 0  
ImportTableAdress dd  IMPORT_START-401000h-400h+2000h ;定义输入表，这里的重定位稍微复杂，结合 MakeData.asm，读者自己思考为什么！  
ImportTableSize  dd  IMPORT_LENGTH  
DirectoryData2 dq 14 dup(0)     ;剩下的数据目录表成员统统填 0,  

SECTION_HEADER1:           ;>>>>>>>>>>>>块表 1  
Name1      db  'CODE', 0, 0, 0,0          ;节区名，八个字节大小  
VirtualSize    dd  CODE_LENGTH  
VirtualAddress    dd  1000h  
SizeOfRawData    dd  200h  
PointerToRawData  dd  CODE_START-401000h        ;也可以直接写成是 200h，因为定义的 pe 头大小为 200h  
PointerToRelocations   dd   0  
PointerToLinenumbers   dd   0  
NumberOfRelocations    dw   0  
NumberOfLinenumbers    dw   0  
_Characteristics  dd    0E0000020h  

SECTION_HEADER2:          ;>>>>>>>>>>>>>>>>>>块表 2，为了自建输入表，再定义一个块  
Name2      db  'IMPORT', 0, 0          ;可不可以去掉这个块，把输入表放到代码块里面呢？可以，自己玩玩吧        
VirtualSize2    dd  IMPORT_LENGTH          ;当然了，也可以不建输入表，直接用 PEB 结构查找 API 也是可以的  
VirtualAddress2    dd  2000h  
SizeOfRawData2    dd  200h  
PointerToRawData2  dd  IMPORT_START-401000h        ;也可以直接写成是 400h  
PointerToRelocations2   dd   0  
PointerToLinenumbers2   dd   0  
NumberOfRelocations2    dw   0  
NumberOfLinenumbers2    dw   0  
_Characteristics2  dd    0E0000020h  

PE_HEADER_END equ this BYTE  
PE_HEADER_LENGTH equ offset PE_HEADER_END - offset PE_HEADER_START  

ZeroSpace1 db 200h-PE_HEADER_LENGTH dup(0)           ;>>>>空白部分填充 0,这个是必须的，这里的编译器不管这事，自己动手丰衣足食！  

;**************************CODE_SECTION_START**************************

CODE_START  equ this BYTE  
                   lea eax, [szContextR-200h]          ;想一下，这里为什么可以这样定位呢？  
                   lea ebx,[szCpationR-200h]  
                   push MB_OK  
                   push  ebx  
                   push eax  
                   push 0  
                   call DWORD ptr [IAT_1-1000h-400h+2000h]    ;CALL PE 加载器填好的地址，我们在做应该是编译器做的工作  
                   push 0  
                   call DWORD ptr  [IAT_2-1000h-400h+2000h]  
                   szContextR db 'Congratulations! You make it!',0dh,0ah  
                            db '           By:moonife',0  
                   szCpationR db 'OK',0  

CODE_END  equ this BYTE  
CODE_LENGTH  equ  offset CODE_END - offset  CODE_START  
ZeroSpace2 db 200h-CODE_LENGTH  dup(0)            ;空白部分填充 0  

;**************************IMPORT_SECTION_START**************************

IMPORT_START  equ this BYTE  

IID_1:  
OriginalFirstThunk  dd IAT_1-401000h-400h+2000h  
TimeDateStemp   dd  0  
ForwarderChain  dd  0  
DllName         dd  DllName1-401000h-400h+2000h  
FirstThunk      dd  IAT_1-401000h-400h+2000h     ;这里为了方便，把它和 OriginalFirstThunk 指向同一个地址的 IAT 结构  
IID_2:  
OriginalFirstThunk2  dd IAT_2-401000h-400h+2000h  
TimeDateStemp2   dd  0  
ForwarderChain2  dd  0  
DllName2         dd  _DllName2-401000h-400h+2000h  
FirstThunk2      dd  IAT_2-401000h-400h+2000h  

IID_END:  
IIDEND  dd 5 dup(0)  

IAT_1:  
AddressOfData1  dd  IIBN_1-401000h-400h+2000h  
AddressOfDataEnd1  dd  0  

IAT_2:  
AddressOfData2  dd  IIBN_2-401000h-400h+2000h  
AddressOfDataEnd2  dd  0  

IIBN_1:  
Hint1  dw  0  
Nama1  db  'MessageBoxA',0  
DllName1 db  'user32.dll',0,0  

IIBN_2:  
Hint2  dw  0  
Nama2  db  'ExitProcess',0  
_DllName2  db  'kernel32.dll',0,0  

IMPORT_END   equ  this BYTE  
IMPORT_LENGTH  equ offset IMPORT_END - offset  IMPORT_START  
ZeroSpace3 db 200h- IMPORT_LENGTH dup(0)            ;空白部分填充 0，我也学学小鱼哥，问句：这个 PE 大小是多少 BYTE 呢？贼简单！嘿嘿！@
```



## 二：编译产生 PE 文件

```
MakeData.asm
```

```assembly
.586

.model flat,stdcall

option casemap:none

;****************************************************************

include windows.inc

include user32.inc

include kernel32.inc

includelib user32.lib

includelib kernel32.lib

;****************************************************************

.data

hOutFile dd 0

BytesWritten dd 0

.const

szCaption db 'Info',0

szContext db 'success',0

szOutFileName  db 'pe.exe',0

;****************************************************************

.code

include   pe.asm  ;这里把实现 PE 数据代码引入，编译并产生 pe.exe 文件，下面的就应该都看懂，就不讲了

start:

invoke  CreateFile,offset szOutFileName,GENERIC_READ or GENERIC_WRITE,\
         FILE_SHARE_READ,\
         NULL,CREATE_ALWAYS,\
         FILE_ATTRIBUTE_NORMAL,NULL

   mov  hOutFile,eax

invoke  WriteFile,hOutFile,offset REMOTE_CODE_START,REMOTE_CODE_LENGTH,addr BytesWritten,NULL

invoke  MessageBox,0,offset szContext,offset szCaption,MB_OK

invoke ExitProcess,0

end start

MakeData_END
```

## 三：至此，一个干干净净的 PE 就新鲜出炉了，来个成功截图：

![](img/1143e16f9ab1ec42b9aa06656cd69a9d.jpg)

Oh！ye！还等什么，赶紧自己弄一个吧！

## 附件清单

`pe.asm`, `MakeData.asm`, `pe.exe`, 本文档

PS: 感谢小鱼哥给我们带来了这么精彩的系列文章，特别感谢 `kanxue` 大哥和 `pediy` 团队已经各位版主辛勤的奉献给我们带来了这么好的交流平台！

重要更正：小鱼不是哥，是姐！ ![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `pe.rar`
|

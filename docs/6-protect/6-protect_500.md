

# 软件保护壳技术专题 - 添加新节(练习笔记)

> 原文：[`www.pediy.com/kssd/pediy11/118651.html`](https://www.pediy.com/kssd/pediy11/118651.html)

**跟着玩命的保护壳技术专题学习,随便解决了一些问题,代码并非按玩命的方式写,只是跟着他的原理按自己的编写风格写的,纯新手,高手莫笑!!**  
  
**添加节 基本步骤:  
1.  创建一个映射文件,大小为原文件大小+新节大小(需要对齐)  
2.  保存 SizeOfImage 的地址,保存 NumberOfSections 的值,然后+1,保存 SectionAlignment 和 FileAlignment 的值,用于对齐计算!  
3.  定位到节表(IMAGE\_SECTION\_HEADER),一般用头文件地址+头文件大小, 当然在无法确定数据目录段的数量的时候还是通过\[NumberOfRvaAndSizes\]\*8 来定位  
4.  NumberOfSections\*IMAGE\_SECTION\_HEADER 就是新节表地址,然后就是把节表的结构参数填上, 新节的内存偏移 = 上一节的内存偏移 + 上一节经过节对齐后的长度, 新节的文件偏移 = 上一节的文件偏移 + 上一节进过文件对齐后的长度, 如果文件偏移为 0,就要取上面的上面,这种取法可能不标准,然后设置 SizeofImage,表示从文件到内存映射文件的内容通过节对齐的大小,这个值等于当前最后一节的内存偏移 + 当前最后一节的经过节对齐的大小,然后把数据移到新节位置,大功告成!**  
主体部分:  

代码:

```
.386
.model flat, stdcall
option casemap:none

include windows.inc
include kernel32.inc
include user32.inc
include macro.asm
includelib kernel32.lib
includelib user32.lib
include    comdlg32.inc
includelib  comdlg32.lib
.data
szSectionName  db '.Alone',0
SecData      db  200 dup('A')

.code
include kernel.asm

\_OpenExeFile proc
local OFN:OPENFILENAME,szFileNameBuff\[MAX\_PATH\]:BYTE 
local Map:MapFileHandle
    ;以下是打开通用对话框的操作
    invoke RtlZeroMemory,addr OFN,sizeof OFN  ;初始化结构
    invoke RtlZeroMemory,addr szFileNameBuff,MAX\_PATH
    mov  OFN.lStructSize,sizeof  OFN
    lea  eax,szFileNameBuff
    mov  OFN.lpstrFile,eax
    mov  OFN.nMaxFile,MAX\_PATH
    mov  OFN.Flags,OFN\_PATHMUSTEXIST or OFN\_FILEMUSTEXIST
    invoke  GetOpenFileName,addr OFN  ;打开通用对话框
    ;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
.if  eax
invoke  \_CalcAlign,sizeof SecData,200H  ;以 200H 为对齐方式
invoke  \_CreatMapFile,addr szFileNameBuff, addr Map,eax;创建一个内存映射文件
mov  eax,Map.lpMemory  ;简单判断下 PE 文件有效性
assume  eax:ptr IMAGE\_DOS\_HEADER
  .if  \[eax\].e\_magic == IMAGE\_DOS\_SIGNATURE
    add  eax,\[eax\].e\_lfanew ;到文件头
    assume  eax:ptr IMAGE\_NT\_HEADERS
    .if  \[eax\].Signature==IMAGE\_NT\_SIGNATURE
      invoke  \_AddSection,Map.lpMemory,addr szSectionName,sizeof szSectionName,addr SecData,sizeof SecData
      invoke  MessageBox,NULL,CTXT("添加节成功!"),NULL,NULL
      jmp @F
    .endif
  .endif
  invoke MessageBox,NULL,CTXT("不是有效的 PE 文件!"),NULL,NULL
@@:
invoke  \_CloseMapFile, addr Map
.endif
ret
\_OpenExeFile endp

;OEP
  Start:
  call \_OpenExeFile
  invoke ExitProcess,NULL
  end Start

```

核心部分,主要便于二次开发  
  
代码:

```
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
; 用一个结构来保存创始内存映射后的句柄
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
MapFileHandle struct
  hFile      dd  ?
  dwFileSize  dd ?
  hMapFile    dd  ?
  lpMemory    dd  ?
MapFileHandle ends
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
; 计算对齐值
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
\_CalcAlign proc dwval,dwalign;(dwval+dwalign-1)\\dwalign\*dwalign
mov  eax,dwalign
dec  eax
add  eax,dwval
xor  edx,edx
mov  ecx,dwalign
div  ecx
imul  ecx
ret
\_CalcAlign endp
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
; 增加节 
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
\_AddSection proc uses esi edi lpMem,lpSecName,dwSecNameSize,lpSecAddr,DwSecSize;参数:内存映射地址,新节名及大小,节数据及大小
;添加新节相关的 PE 头属性位于 IMAGE\_NT\_HEADERS 结构中的属性:
;SizeOfImage(4 字节)  NumberOfSections(2 字节)  SectionAlignment(4 字节)  FileAlignment(4 字节)
local SizeOfImageAddr,NumberOfSections,SectionAlignment,FileAlignment
mov  esi,lpMem
assume  esi:ptr IMAGE\_DOS\_HEADER
add  esi,\[esi\].e\_lfanew  ;到文件头
      ;初始化一些变量
      assume  esi:ptr IMAGE\_NT\_HEADERS
      lea  eax,\[esi\].OptionalHeader.SizeOfImage
      mov  SizeOfImageAddr,eax  ;这个地址的值要修改
      movzx  eax,\[esi\].FileHeader.NumberOfSections
      mov  NumberOfSections,eax;节个数
      inc  \[esi\].FileHeader.NumberOfSections  ;加一个节
      push  \[esi\].OptionalHeader.SectionAlignment
      pop  SectionAlignment
      push  \[esi\].OptionalHeader.FileAlignment
      pop  FileAlignment
add  esi,sizeof IMAGE\_NT\_HEADERS;到节表地址,当然在无法确定数据目录段的数量的时候还是通过\[NumberOfRvaAndSizes\]\*8 来定位节表好
mov  ecx,NumberOfSections
mov  eax,sizeof IMAGE\_SECTION\_HEADER
imul  ecx
lea  esi,\[esi+eax\] ; 28=sizeof IMAGE\_SECTION\_HEADER
assume  esi:ptr IMAGE\_SECTION\_HEADER
lea  eax,\[esi\].Name1
invoke RtlMoveMemory,eax,lpSecName,dwSecNameSize ;设置节名
push  DwSecSize
pop  \[esi\].Misc.VirtualSize;设置内存中节大小
invoke  \_CalcAlign,DwSecSize,FileAlignment;计算文件中节大小
mov  \[esi\].SizeOfRawData,eax  ;设置文件中节大小
mov  edi,esi
sub  edi,sizeof IMAGE\_SECTION\_HEADER  ;上一个节指针,不知道这样算标准不?
assume  edi:ptr IMAGE\_SECTION\_HEADER
; 新节的内存偏移 = 上一节的内存偏移 + 上一节经过节对齐后的长度
mov  eax,\[edi\].VirtualAddress
add  eax,\[edi\].Misc.VirtualSize
invoke \_CalcAlign,eax,SectionAlignment
mov  \[esi\].VirtualAddress,eax
; 新节的文件偏移 = 上一节的文件偏移 + 上一节进过文件对齐后的长度
@@:
mov  eax,\[edi\].PointerToRawData
.if  !eax  ;如果文件偏移=0...      ;貌似能解决 cham 说的文件偏移为 0 的问题
sub  edi,sizeof IMAGE\_SECTION\_HEADER
jmp @B
.endif
add  eax,\[edi\].SizeOfRawData
invoke \_CalcAlign,eax,FileAlignment
mov  \[esi\].PointerToRawData,eax
;; SizeofImage 表示从文件到内存映射文件的内容通过节对齐的大小,这个值等于当前最后一节的内存偏移 + 当前最后一节的经过节对齐的大小
mov  eax,\[esi\].VirtualAddress
add  eax,\[esi\].Misc.VirtualSize
invoke \_CalcAlign,eax,SectionAlignment
mov  ecx,SizeOfImageAddr
mov  \[ecx\],eax  ;设置 SizeOfImage
;然后把数据放到节区
mov  edi,\[esi\].PointerToRawData
add  edi,lpMem
invoke  RtlMoveMemory,edi,lpSecAddr,DwSecSize
ret
\_AddSection endp
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
; 关闭创建的内存映射文件相关句柄
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
\_CloseMapFile proc uses esi lpmapFuct
mov  esi,lpmapFuct
assume  esi:ptr MapFileHandle
  .if  \[esi\].hFile!=INVALID\_HANDLE\_VALUE ;文件句柄是否有效
    invoke CloseHandle,\[esi\].hFile
    .if  \[esi\].hMapFile
      invoke CloseHandle,\[esi\].hMapFile
      .if  \[esi\].lpMemory
        invoke UnmapViewOfFile,\[esi\].lpMemory
      .endif  
    .endif  
  .endif
ret
\_CloseMapFile endp
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
; 创建一个内存映射文件,可读写操作,独占式打开
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
\_CreatMapFile proc uses esi lpPath,lpmapFuct,dwSizeEx;dwSizeEx 附加大小
mov  esi,lpmapFuct
invoke  RtlZeroMemory,esi,sizeof MapFileHandle;初始化自己定义的结构
invoke  CreateFile, lpPath,GENERIC\_WRITE + GENERIC\_READ,FILE\_SHARE\_DELETE,NULL,OPEN\_EXISTING,FILE\_ATTRIBUTE\_NORMAL,0
    .if eax
      mov  \[esi\].hFile,eax
      invoke  GetFileSize,eax,NULL
      mov  \[esi\].dwFileSize,eax
      add  eax,dwSizeEx
      invoke CreateFileMapping,\[esi\].hFile, 0, PAGE\_READWRITE, 0, eax, 0
      .if eax
        mov  \[esi\].hMapFile,eax
        invoke  MapViewOfFile,eax,FILE\_MAP\_READ or FILE\_MAP\_WRITE,NULL,NULL,NULL
        .if  eax
          mov  \[esi\].lpMemory,eax
          xor  eax,eax  ;返回值
          jmp  @F
        .endif  
      .endif  
    .endif  
  invoke \_CloseMapFile,lpmapFuct  ;中途的失败操作,都会默认这里来关闭文件句柄
  mov  eax,1  ;返回值
@@:    
ret
\_CreatMapFile endp

```

好像也解决了如果最后一个节的文件大小为 0 的情况  
http://bbs.pediy.com/showthread.php?t=66612&page=2

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || 壳编写学习.rar  
 |
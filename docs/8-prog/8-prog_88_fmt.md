
# 汇编学习之剪贴版操作

> 原文：[`www.pediy.com/kssd/pediy10/77992.html`](https://www.pediy.com/kssd/pediy10/77992.html)

剪贴板是一组消息和函数的组合，通过它能实现应用程序的数据交换，但是怎么实现这个过程呢？

复制：

1. `OpenClipBoard` 函数打开剪贴板
2. `EmptyClipboard` 函数清空剪贴板
3. 调用 `SetClipboardData` 函数
4. 调用 `CloseClipBoard` 关闭剪贴板

代码:

```asm
.386
.model flat,stdcall
option casemap:none

include \masm32\include\windows.inc
include \masm32\include\kernel32.inc
include \masm32\include\user32.inc

includelib \masm32\lib\user32.lib
includelib \masm32\lib\kernel32.lib

.data
;some err message

nothing db "Nothing",0

noopen db "Can't open clipbaord",0

noalloc db "Unable to allocate memory",0

MsgCaption db "Clipboard",0
CmdLn db "c:\download",0
.data?

hMemory HANDLE ? ;handle to allocated memory
pMemory DWORD ? ;pointer to allocate memory
len DWORD ? ;length of argument(path)

.code
start:
  push edi
  push esi
  invoke lstrlen,ADDR CmdLn
  .if eax ==0
    jmp noline
  .endif
   
  inc eax
  mov len,eax
  invoke OpenClipboard,NULL
  .if eax ==0
    jmp erroropen
  .endif
   
  invoke GlobalAlloc,GMEM_MOVEABLE,len
  .if eax == 0
    jmp erroralloc
  .endif
   
  mov hMemory,eax
  invoke GlobalLock,hMemory
   
  mov pMemory,eax
  mov esi,CmdLn    ;start of string
  mov edi,pMemory  ;and to momory
  mov ecx,len
  rep movsb
  invoke EmptyClipboard
  invoke GlobalUnlock,hMemory
  invoke SetClipboardData,,CF_TEXT,hMemory 
  invoke CloseClipboard
  jmp endit
   
noline:
  mov eax,offset nothing
  jmp mess
erroropen:
  mov eax,offset noopen
  jmp mess
erroralloc:
  mov eax,offset noalloc
mess:
  invoke MessageBox,NULL,eax,addr MsgCaption,MB_OK
endit:
  pop esi
  pop edi
  invoke ExitProcess,NULL
   
end start
```

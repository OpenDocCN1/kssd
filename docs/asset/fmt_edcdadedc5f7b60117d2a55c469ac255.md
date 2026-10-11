
# SEH 在程序中的运用

> 原文：[www.pediy.com/kssd/pediy10/101488.html](https://www.pediy.com/kssd/pediy10/101488.html)

结构化异常处理

当某一线程发生异常时，程序的控制权会立即进入 `Ring0` 异常处理程序，这是属于操作系统的部分， 如果发生的异常是如页异常之类的异常，`Ring0` 处理程序可以处理完它后重新回到程序中执行，而被中断 过的进程可能根本就不知道发生过异常。

但事情并不总是如此，有时进程会发生一些始料不及的异常，例如访问不存在的内存，被 0 除等， 这些异常 `Ring0` 处理程序不知该如何处理它，而进程本身也可能想自己处理这些情况，这是就要用到结构化异常处理（`SEH`）。在 `C/C++` 中也有异常处理的语句如 `_try`、`_catch` 等，这些语句的实现也与 `SEH` 紧密联系。

当系统遇到一个它不知道如何处理的异常时，它就查找异常处理链表，注意每个线程都有它自己的异 常处理链表。异常链表以 `FS:[0]` 所指向的位置为链表头。

异常处理开始时，系统把一些与当前线程和与异常有关的内容传给链头所指向的处理程序；处理程序 由用户编写或编译器生成，它的返回值可以是告诉系统：异常处理以完成，可以继续执行程序，或未处理 异常，可由链表的下一个处理程序处理等，可以一次传递下去。

下面我以以前写的一个 `WinMD5` 程序来说明：

下面是未运用 `SEH` 链的情况下的部分关键代码：

```
GetMd5Thread PROC PFile:DWORD
    LOCAL @FileText[MAX_PATH]:BYTE
LOCAL @TEMP[MAX_PATH]:BYTE
LOCAL @hFileRead,@hMapFile,@pMemory,@FileSize    
   invoke CreateFile, PFile,\  
                                GENERIC_READ ,\  
                                0,\  
                                NULL,OPEN_EXISTING,FILE_ATTRIBUTE_ARCHIVE,\  
                                NULL  
    .if eax==0  
      jmp reterr  
    .endif  
   mov @hFileRead,eax  
   invoke ExtractFileName,PFile,addr @FileText  ; 取短文件名  
   invoke CreateFileMapping,@hFileRead,NULL,PAGE_READONLY,0,0,NULL  
     .if eax==0  
         invoke CloseHandle,@hFileRead  
         jmp reterr  
     ret  
     .endif     
   mov @hMapFile,eax  
   invoke MapViewOfFile,@hMapFile,FILE_MAP_READ,0,0,0  
     .if eax==0  
         invoke CloseMapFile,@hMapFile,@hFileRead  
         jmp reterr  
     .endif  
                          mov @pMemory,eax   
                          invoke  SendDlgItemMessage,hWinMain,IDC_PGB,PBM_SETPOS,1,0   ;进度开始（注意：这里并不是实时进度）                           
                          invoke  GetFileSize,@hFileRead,0   
                          mov @FileSize,eax                                                  
   invoke  _MD5,@pMemory,eax                                                                
                          invoke wsprintf ,addr  @TEMP,addr  szReceive, addr @FileText, @FileSize,eax                                                                                                                      
   invoke  SendDlgItemMessage,hWinMain,IDC_EDT_OUT,EM_REPLACESEL,0,addr @TEMP     
   invoke  SendDlgItemMessage,hWinMain,IDC_PGB,PBM_SETPOS,100,0  ;进度结束  
   invoke  RtlZeroMemory,addr FileName,MAX_PATH              ;  清空数据     
   INVOKE SendDlgItemMessage,hWinMain,IDC_EDT_FILE,WM_SETTEXT,0, addr FileName           
   invoke UnmapViewOfFile,@pMemory  
   invoke CloseMapFile,@hMapFile,@hFileRead
                          ret
      reterr:
      invoke MessageBox,0,CTEXT("文件打开失败，请检查是否为有效文件！"),CTEXT("WinMd5 for ASM v1.1"),MB_ICONERROR OR MB_OK
       ret
GetMd5Thread endp
```

上面红色的部分，是我们自己预防错误的代码，这个程序不算复杂，如果复杂点的程序，其代码肯定会显得很臃肿，并且可能会忽略些错误，给用户带来不好的使用感受，那么有没有一种机制可以统一处理这些错误了？当然有，这就是我们的 `SHE` 机制了

下面是上面的程序采用了 `SEH` 机制的全部源码：

```
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;  Programmed by nohacks, nohacks@163.com
;  Website: http://hi.baidu.com/nohacks
;          Win32 ASM is Masm
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;  版本信息
;  WinMD5 for ASM V1.1   - 可以拖放取得文件的 MD5 值
;     
;                       2007 年 11 月 21 日
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
.386
.model  flat,  stdcall
option  casemap  :none
include    windows.inc
include    user32.inc
include         shell32.inc
include    kernel32.inc
includelib  user32.lib
includelib  kernel32.lib
include    comdlg32.inc
includelib  comdlg32.lib
includelib      shell32.lib
include          debug.inc
include        Stdlib.Inc
includelib  Stdlib.lib

m2m  Macro M1,M2
  push  M2
  pop   M1
endm

SEH struct 
PrevLink dd ?    ; 用来保存 SHE 原地址
CurrentHandler dd ?    ; SEH 处理函数地址
SafeOffset dd ?    ; 执行恢复地址  
PrevEsp dd ?     ; 用来保存 SEH 前的 esp  
PrevEbp dd ?     ;  用来保存 SEH 前的 ebp 
SEH ends

.const          
 ICO_ICO            EQU           1
 DLG_MAIN    EQU    1000
 IDC_EDT_FILE         EQU         1001
 IDC_EDT_OUT          EQU        1002
 IDC_PGB            EQU         1003
 IDC_BTN_EXIT         EQU          1004

.data? 

hWinMain    dd    ?
hInstance    dd    ?
hMenu HANDLE dword ? 
TheThread DWORD ?
 
ValidPE dd ?                      

.data

FileName         db      MAX_PATH  dup(0)
temp             db      '%s',0DH,0AH,0
FilterString   db   "全部文件(*.*)",0,"*.*",0,0

szReceive         db       '文件名   ：%s',0dh,0ah
                 db       '文件大小 ：%d','字节',0dh,0ah
                 db       '文件 MD5 值：%s ',0dh,0ah,0
TEMP             db       '.',0

include    md5.asm

_SetWindowCenter proc _hWnd:DWORD
LOCAL swidth,sheight,dwidth,dheight
LOCAL rect:RECT
invoke GetSystemMetrics,SM_CXSCREEN     
       mov swidth,eax                    
       invoke GetSystemMetrics,SM_CYSCREEN     
       mov sheight,eax                   
       invoke GetWindowRect,_hWnd,addr  rect    
       mov eax,rect.right               
       sub eax,rect.left               
       mov dwidth,eax                 
       sub swidth,eax                 
       mov eax,rect.bottom           
       sub eax,rect.top         
       mov dheight,eax               
       sub sheight,eax                 
       shr sheight,1                
       shr swidth,1                 
       invoke SetWindowPos,_hWnd,HWND_NOTOPMOST,swidth,sheight,dwidth,dheight,SWP_SHOWWINDOW
ret
_SetWindowCenter endp

;*****************************************************************************************
CloseMapFile PROC hMapFile:DWORD,hFileRead:DWORD
    invoke CloseHandle,hMapFile 
    ;mov    hMapFile,0 
    invoke CloseHandle,hFileRead 
    ret  
CloseMapFile endp 

GetMd5Thread PROC PFile:DWORD
    LOCAL seh:SEH
    LOCAL md5 ,ProcessId
    LOCAL   @FileText[MAX_PATH]:BYTE
  LOCAL   @TEMP[MAX_PATH]:BYTE
  LOCAL  @hFileRead,@hMapFile,@pMemory,@FileSize   
    
      ;SEH 异常处理  
       assume fs:nothing   
           push fs:[0] 
           pop seh.PrevLink
           mov seh.CurrentHandler,offset SEHHandler 
           mov seh.SafeOffset,offset FinalExit 
           lea eax,seh 
           mov fs:[0], eax 
           mov seh.PrevEsp,esp 
           mov seh.PrevEbp,ebp 
       invoke CreateFile, PFile,\  
                                GENERIC_READ ,\  
                                0,\  
                                NULL,OPEN_EXISTING,FILE_ATTRIBUTE_ARCHIVE,\  
                                0    
   mov    @hFileRead,eax
   invoke ExtractFileName,PFile,addr @FileText  ; 取短文件名 
   invoke  GetFileSize,@hFileRead,0     ;取文件尺寸
                          mov @FileSize,eax  
   invoke CreateFileMapping,@hFileRead,NULL,PAGE_READONLY,0,0,NULL 
   mov @hMapFile,eax
                          invoke MapViewOfFileEx,@hMapFile,FILE_MAP_READ,0,0,0,0
                          mov @pMemory,eax
                          invoke  SendDlgItemMessage,hWinMain,IDC_PGB,PBM_SETPOS,1,0   ;进度开始（注意：这里并不是实时进度）                           
```

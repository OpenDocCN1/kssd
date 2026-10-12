
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



### 代码实现

```asm
invoke   _MD5,@pMemory,  @FileSize  

  ;MD5 转为大写  

  mov md5,eax  

  invoke lstrlen, md5  

  invoke CharUpperBuff,md5,eax  

                 invoke wsprintf ,addr  @TEMP,addr  szReceive, addr @FileText, @FileSize,md5                      

  invoke  SendDlgItemMessage,hWinMain,IDC_EDT_OUT,EM_REPLACESEL,0,addr @TEMP      

  invoke  SendDlgItemMessage,hWinMain,IDC_PGB,PBM_SETPOS,100,0  ;进度结束   

  invoke  RtlZeroMemory,addr FileName,MAX_PATH                 ;  清空数据     

  INVOKE SendDlgItemMessage,hWinMain,IDC_EDT_FILE,WM_SETTEXT,0, addr FileName           

  invoke UnmapViewOfFile,@pMemory   

  invoke CloseMapFile,@hMapFile,@hFileRead  

  ;断开 SHE 链  

   push seh.PrevLink   

                 pop fs:[0]     

                 ret  

  SEH 执行恢复地址：   

   FinalExit:  

   invoke MessageBox,0,CTEXT("文件打开失败，请检查是否为有效文件！"),CTEXT("WinMd5 for ASM v1.1"),MB_ICONERROR OR MB_OK  

    ret  

GetMd5Thread endp  

_ProcDlgMain  proc  uses ebx edi esi, \  

              hWnd:DWORD,wMsg:DWORD,wParam:DWORD,lParam:DWORD  

  LOCAL  @hMem,@hFile,@Size,@Read   

        mov  eax,wMsg  

  cmp     eax ,WM_DROPFILES  

  je      GetFile   

  cmp  eax,WM_COMMAND                             

  je  Exit  

  cmp  eax,WM_INITDIALOG                          

  je  boxStart  

  cmp  eax,WM_CLOSE  

  je  boxClose   

retFalse:  

  mov  eax,FALSE  

  ret  

boxClose:  

   invoke  EndDialog,hWnd,NULL  

   jmp  retTrue  

boxStart:  

        push  hWnd  

  pop  hWinMain  

    invoke LoadIcon,hInstance,ICO_ICO  

    invoke SendMessage,hWnd,WM_SETICON,ICON_BIG,eax    ;设置窗口图标   

    invoke SetWindowPos,hWnd,HWND_TOPMOST,0,0,0,0,  SWP_NOMOVE or SWP_NOSIZE ;窗口置顶  

    INVOKE _SetWindowCenter,hWinMain     ;使窗体出现在屏幕中心  

    invoke DragAcceptFiles,hWnd,TRUE     ;允许拖放文件  

    jmp  retTrue  

GetFile:  

     invoke DragQueryFile,wParam,0,addr FileName,MAX_PATH                      ;取拖放文件名  

     invoke SendDlgItemMessage,hWnd,IDC_EDT_FILE,EM_REPLACESEL,0,  addr FileName    ;输出到编辑框  

     invoke  DragFinish,wParam                                                   ;释放拖放资源  

     invoke CreateThread,NULL,0,addr  GetMd5Thread,addr FileName,0,NULL         ;创建线程  

  Exit:  

   mov     eax,wParam  

    .if           eax==IDC_BTN_EXIT  

   invoke  ExitProcess,NULL  

   .endif  

retTrue:  

  mov  eax,TRUE  

  ret  

_ProcDlgMain  endp  

;--------------------------------------------------------------------;  

;函数功能：SEH 回调函数  

;-------------------------------------------------------------------;  

SEHHandler proc uses edx pExcept:DWORD, pFrame:DWORD, pContext:DWORD, pDispatch:DWORD  

    mov edx,pFrame   

    assume edx:ptr SEH   

    mov eax,pContext   

    assume eax:ptr CONTEXT   

    push [edx].SafeOffset   

    pop [eax].regEip   

    push [edx].PrevEsp   

    pop [eax].regEsp   

    push [edx].PrevEbp   

    pop [eax].regEbp   

    mov eax,ExceptionContinueExecution   

    ret   

SEHHandler endp   

Start:  

  invoke  GetModuleHandle,NULL  

  mov    hInstance,eax  

  invoke  DialogBoxParam,hInstance,DLG_MAIN,NULL,_ProcDlgMain,0  

  invoke  ExitProcess,NULL  

End  Start  
```

### 分析：SEH 结构定义

```asm
SEH struct   

PrevLink dd ?    ; 用来保存 SHE 原地址  

CurrentHandler dd ?    ; SEH 处理函数地址  

SafeOffset dd ?    ; 执行恢复地址   

PrevEsp dd ?     ; 用来保存 SEH 前的 esp   

PrevEbp dd ?     ;  用来保存 SEH 前的 ebp   

SEH ends  
```

### 程序逻辑解析

程序开始先注册一个 SEH 结构

```asm
GetMd5Thread PROC PFile:DWORD  

 LOCAL seh:SEH  

        LOCAL md5 ,ProcessId  

        LOCAL   @FileText[MAX_PATH]:BYTE  

  LOCAL   @TEMP[MAX_PATH]:BYTE  

  LOCAL  @hFileRead,@hMapFile,@pMemory,@FileSize   

   ;SEH 异常处理   

assume fs:nothing   

                 push fs:[0]   

                 pop seh.PrevLink                        ;备份当前 SEH  

                 mov seh.CurrentHandler,offset SEHHandler     ;SEH 处理函数地址  

                 mov seh.SafeOffset,offset FinalExit             ;执行恢复地址  

                 lea eax,seh                                     

                 mov fs:[0], eax                                  ;装载我们的 SHE  

                 mov seh.PrevEsp,esp                        ;备份 ESP  

                 mov seh.PrevEbp,ebp                    ;备份 EBP  
```

程序打开文件前前建立 SEH，一开始就假设寄存器 `fs` 为空（`assume fs:nothing`）。 记住这一步不能省却，因为 MASM 假设 `fs` 寄存器为 `ERROR`。接下来保存 `Windows` 使用的旧 SEH 处理函数地址到我们自己定义的结构中，同时保存我们的 SEH 处理函数地址和异常处理时的执行恢复地址，这样一旦错误发生就能由异常处理函数安全地恢复执行了。同时还保存当前 `esp` 及 `ebp` 的值，以便我们的 SEH 处理函数将堆栈恢复到正常状态。

```asm
 invoke CloseMapFile,@hMapFile,@hFileRead  

   ;断开 SHE 链  

  push seh.PrevLink   

             pop fs:[0]    
```

一旦 SEH 不再使用，必须从 SEH 链上断开。

```asm
FinalExit:  

       invoke MessageBox,0,CTEXT("文件打开失败，请检查是否为有效文件！"),CTEXT("WinMd5 for ASM v1.1"),MB_ICONERROR OR MB_OK  

  ret  
```

这里就是我们在前面定义的 SEH 执行恢复地址，当程序从错误中恢复过来，就会执行这里。

在本例中，如果用户拖放的是个文件夹，程序就会发生异常，就会进入了我们指定的处理程序

```asm
SEHHandler proc uses edx esi pExcept:DWORD, pFrame:DWORD, pContext:DWORD, pDispatch:DWORD   

   LOCAL ExceptionCode   

    mov edx,pFrame   

    assume edx:ptr SEH   

    mov eax,pContext   

    assume eax:ptr CONTEXT   

    mov esi, pExcept  

    m2m ExceptionCode ,[esi][EXCEPTION_RECORD.ExceptionCode]   

   ;you code                     错误代码记录在变量 ExceptionCode 里  

    debug TEXTEQU <调试信息：SEH 安装成功！>  

    %echo debug  
```

我们处理的第一件事就是输出相应的调试信息，之后开始真正的异常处理。

```asm
   ;执行恢复地址  

    push [edx].SafeOffset   

    pop [eax].regEip   

   ;恢复 ESP   

    push [edx].PrevEsp   

    pop [eax].regEsp   

   ;恢复 EBP  

    push [edx].PrevEbp   

    pop [eax].regEbp   

    mov eax,ExceptionContinueExecution   

    ret  
```

在本例中异常处理只是简单地恢复 `esp`、`ebp` 寄存器的值并将 `eip` 寄存器的值置为一个能使线程安全地继续执行的地址。这些处理信息都是由系统从我们预先填充的 `seh` 结构体中抽出并保存在了 `CONTEXT` 结构体中，`CONTEXT` 结构体的指针又传递给了系统。

我们以 `ExceptionContinueExecution` 的值来返回(该值为零)，就是告诉系统应该恢复线程的上下文并继续执行。即 `eip` 的值等于标记 `FinalExit` 的地址，而 `esp` 和 `ebp` 寄存器的值恢复为原值，线程从标记 `FinalExit` 处继续其执行。

上面程序的 SEH 相关代码，您完全可以直接 COPY 到您的代码中。

经过上面 2 个代码的比较，我们应该知道了 SEH 的妙用，熟练运用能使您的代码更简洁高效。

### 附件

| ![](img/fb8ade91a9e54b7bd6690c3ae5b1d1cc.jpg) || `Md5Win091118.zip`  

 |

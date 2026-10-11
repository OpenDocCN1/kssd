
\_GetRVASection endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

前面有一个朋友给我留言说要搞个链接，好的，等我把第七篇文章写完之后，会在最后一篇文章的最后把前面的六篇文章的地址都链接上去，也样第一方便了大家，也同时方便了自己阅读学习，给人方便，就是给自己方便！！`![](img/cda69129882b40377a87144c782680e1.jpg "微笑")`

参考文献《Win32 汇编语言程序设计》第二版

前面的五篇文章，所讲的都是对 PE 文件的结构进行了分析，以及对 PE 工具的编写进行了介绍，但在实际的应用中还未有涉及到很多方面的内容，其实学好 PE 在很多方面都有很重要的应用，比如文件的加密，压缩，编写病毒等都涉及到修改及重组 PE 文件，另外，像 API HOOK，PE 文件的内存映像 DUMP 等应用则涉及分析内存中的 PE 映像。在接下来的两篇文章中我将用几个例子来进一步说明这方面的应用，可以说这是编写病毒的基础吧！！

首先讲在病毒中用到的一种很常用的技术，就是如何从内存中动态获取某个 API 的地址。

## 动态获取 API 的入口地址

在 Win32 环境下编程，不便用 API 几乎是不可能的事情，一般情况下，在代码中使用 API 不外乎两种办法：第一是编译链接的时候使用导入库，那么生成的 PE 文件中就会包含导入表，这样程序执行时会由 Windows 装载器根据导入表中的信息来修正 API 调用语句中的地址；第二种方法是使用`LoadLibrary`函数动态装入某个 DLL 模块，并使用`GetProcAddress`函数从被装载入的模块中获取 API 函数的地址。

先讲解一下原理吧！`![](img/6821a6175013046370981b6035d43ab4.jpg "使眼色")`

在 DOS 环境下，一个可执行文件既可以用`INT 21h/4ch`来结束程序，也可以用一个`Ret`指令来结束程序，实际上，在 Win32 下也可以用这种方法来结束程序，虽然大部分的 Win32 程序都使用`ExitProcess`函数来终止执行，但是使用`Ret`指令确实也是有效的。

如下图所示，当父进程要创建一个子进程的时候，它会调用`Kernel32.dll`中的`CreateProcess`函数，`CreateProcess`函数在完成装载应用程序后，会将一个返回地址压入堆栈并转而执行应用程序，如果应用程序用`ExitProcess`函数来终止，那么这个返回地址没有什么用途，但如果应用程序使用`Ret`指令的话，程序就会返回`CreateProcess`函数设定的地址。也就是说，应用程序的主程序可以看作是被 Windows 调用的一个子程序。

`![](img/dfb33a1b9826ff1a4c03797621ce6664.jpg)`

图中表示 Win32 可执行文件退出的示意图

那么`Ret`指令返回到的地址上究竟有什么指令呢？用 Soft-ICE 看看就会发现，它包含一句`push eax`指令和一句`call ExitThread`，也就是说，假如用`Ret`指令返回的话，Windows 会替程序去调用`ExitThread`函数，如果这是进程的最后一个线程的话，`ExitThread`函数又会自动去调用`ExitProcess`，这样程序就会被终止执行。

从这个过程可以得到一个很重要的数据，那就是堆栈中的返回地址，这个地址只要在程序入口的地方用`[esp]`就可以将它读出，说它重要是因为它位于`Kernel32.dll`模块中，而`LoadLibrary`和`GetProcAddress`函数正是处于`Kernel32.dll`模块中，换句话说就是，我们得到的地址和这两个函数近在咫尺，完全可以从这个地址经过某种算法来找到这两个函数的入口地址，得到这两个函数的入口地址以后，什么问题都解决了。

结合本章前面内容中提到过的两个事实，可以确定这种想法是可行的。

首先，PE 文件被装入内存后（包括`Kernel32.dll`文件），除了一些可丢弃的节如重定位节以外，其他的内容都会被装入内存，这样获取导出函数地址所需的 PE 文件头、导出表等数据都存在于内存中；第二，PE 文件被装入内存时是按内存页对齐的，只要从`Ret`指令返回的地址按照页对齐的边界一页页地向低地址搜寻，就必然可以找到`Kernel32.dll`文件的文件头位置。

下面请看具体的源代码，我会在其中作出注释，以方便大家理解！

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

```
; Sample code for < Win32ASM Programming 2nd Edition>

; by 罗云彬

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

; NoImport.asm

; 以从内存中动态获取的办法使用 API

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

    .386

    .model flat,stdcall

    option casemap:none

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

include    windows.inc

_ProtoGetProcAddress  typedef  proto  :dword,:dword

_ProtoLoadLibrary  typedef  proto  :dword

_ProtoMessageBox  typedef  proto  :dword,:dword,:dword,:dword

_ApiGetProcAddress  typedef  ptr  _ProtoGetProcAddress

_ApiLoadLibrary    typedef  ptr  _ProtoLoadLibrary

_ApiMessageBox    typedef  ptr  _ProtoMessageBox

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

; 数据段

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

    .data?

hDllKernel32  dd  ?

hDllUser32  dd  ?

_GetProcAddress  _ApiGetProcAddress  ?

_LoadLibrary  _ApiLoadLibrary    ?

_MessageBox  _ApiMessageBox    ?

    .const

szLoadLibrary  db  'LoadLibraryA',0

szGetProcAddress db  'GetProcAddress',0

szUser32  db  'user32',0

szMessageBox  db  'MessageBoxA',0

szCaption  db  'A MessageBox !',0

szText    db  'Hello, World !',0

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

; 代码段

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

    .code

include    _GetKernel.asm                             ;包含获取 Kernel32.dll 的基址的源代码

start:

;**********************************************************************

; 从堆栈中的 Ret 地址转换 Kernel32.dll 的基址，并在 Kernel32.dll

; 的导出表中查找 GetProcAddress 函数的入口地址

;**********************************************************************

    invoke  _GetKernelBase,[esp]                   ;获取 Kernel32.dll 的基址

    .if  eax
      mov  hDllKernel32,eax                  ;保存 Kernel32.dll 的基址
      invoke  _GetApi,hDllKernel32,addr szGetProcAddress ;在 Kernel32.dll 的导出表中查找 GetProcAddress 函数的入口地址
      mov  _GetProcAddress,eax              ;保存 GetProcAddress 函数的入口地址
    .endif

;**********************************************************************

; 用得到的 GetProcAddress 函数得到 LoadLibrary 函数地址并装入其他 Dll

;**********************************************************************

    .if  _GetProcAddress                            ;如果 LoadLibrary 函数地址不为 NULL
      invoke  _GetProcAddress,hDllKernel32,addr szLoadLibrary  ;通过 GetProcAddress 得到 LoadLibrary 函数的地址
      mov  _LoadLibrary,eax                   ;保存 LoadLibrary 函数的地址
      .if  eax
        invoke  _LoadLibrary,addr szUser32         ;使用 LoadLibrary 函数装载 User32.dll
        mov  hDllUser32,eax                  ;保存 User32.dll
        invoke  _GetProcAddress,hDllUser32,addr szMessageBox  ;获取 User32.dll 中 MessageBox 函数的地址
        mov  _MessageBox,eax                  ;保存 MessageBox 函数的地址
      .endif
    .endif

;**********************************************************************

    .if  _MessageBox                                ;如果 MessageBox 函数的地址不为 NULL
      invoke  _MessageBox,NULL,offset szText,offset szCaption,MB_OK
    .endif

    ret

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

    end  start
```

上面这个函数很简单也没什么好说的，我已经注释的很详细了，这里就不多说了，我们来看看它所包含的一个源文件`_GetKernel.asm` 是怎么样获取`Kernel32.dll` 基址的

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

```
; Sample code for < Win32ASM Programming 2nd Edition>

; by 罗云彬

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

; 公用模块：_GetKernel.asm

; 根据程序被调用的时候堆栈中有个用于 Ret 的地址指向 Kernel32.dll

; 而从内存中扫描并获取 Kernel32.dll 的基址

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

; 错误 Handler

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

_SEHHandler  proc  C _lpExceptionRecord,_lpSEH,_lpContext,_lpDispatcherContext  ;这里我不讲了前面已经说过 SEH 异常处理
    pushad
    mov  esi,_lpExceptionRecord
    mov  edi,_lpContext
    assume  esi:ptr EXCEPTION_RECORD,edi:ptr CONTEXT
    mov  eax,_lpSEH
    push  [eax + 0ch]
    pop  [edi].regEbp
    push  [eax + 8]
    pop  [edi].regEip
    push  eax
    pop  [edi].regEsp
    assume  esi:nothing,edi:nothing
    popad
    mov  eax,ExceptionContinueExecution
    ret
_SEHHandler  endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

; 在内存中扫描 Kernel32.dll 的基址

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

_GetKernelBase  proc  _dwKernelRet                          ;传入一个参数
    local  @dwReturn
    pushad
    mov  @dwReturn,0
;**********************************************************************

; 重定位

;**********************************************************************

    call  @F                          ;这里我就不说了，在病毒中经常用到，很多病毒开始就这经典的四句
    @@:                              ;主要就于重定位
    pop  ebx
    sub  ebx,offset @B
```


# Exploit 编写系列教程第三篇_ 基于 SEH 的 Exploit（+3b）

> 原文：[`www.pediy.com/kssd/pediy12/102040.html`](https://www.pediy.com/kssd/pediy12/102040.html)

又学到东西了 ![](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || Exploit 编写系列教程第三篇_ 基于 SEH 的 Exploit.rar |

| ![](img/aabdec8de01ec04d2bc68959c61abb93.jpg) || Exploit 编写教程第三篇 b：基于 SEH 的 Exploit-又一个实例.pdf |

我写了一个简单的例子来演示文章作者所使用的关键技术，

阅读本篇的朋友可以先用 `OD` 跟踪下下面代码出来的程序，

用这个配合着阅读文章，就应该不会被作者翻来覆去的讲解弄晕了 呵呵

```asm
引用: .386  
.model flat, stdcall  
option casemap :none  
;====================================  
include windows.inc  
include user32.inc  
includelib user32.lib  
include kernel32.inc  
includelib kernel32.lib  
;====================================  
.data  
szCap db "By:moonife",0  
szInfo db "SEH Based Exploit Demo...",0  
szMsg db "Nothing...",0  
;====================================  
.code  
Start:  
  ASSUME FS:NOTHING  
        push offset SEH_Handler     ;这里写这个纯粹是为了展示 不然可以直接安装 Fake_Handler  
        push fs:[0]       
        mov fs:[0],esp                      ;安装 SEH  
;====================================  
;下面模拟缓冲区覆盖过程  
        mov DWORD ptr[esp],909006EBh    ;覆盖 “next  SEH”为 jmp $+6 / nop / nop  
        mov DWORD ptr[esp+4],offset Fake_Handler     ;覆盖“SE Handler”为指向 pop pop ret 指令串的指针  
        mov BYTE  ptr[esp+8],0E9h             ;相当于向缓冲区中写入 shellcode  
        lea ebx,[esp+8]                          
        mov eax,offset FakeShellcode-5  
        sub eax,ebx  
        mov DWORD ptr[esp+9],eax            ;这里写入的“shellcode”为 jmp FakeShellcode  
;====================================  
        xor eax,eax  
        mov [eax],eax    ;制造异常并跳到 SEH_Handler 进行处理  
        invoke MessageBox,NULL,addr szMsg,addr szCap,MB_OK   ;这里不会被执行了  
        pop fs:[0]  
        add esp,4  
        invoke ExitProcess,NULL         
;====================================  
SEH_Handler:   
      invoke MessageBox,NULL,addr szMsg,addr szCap,MB_OK  
      mov eax,1  ;ExceptionContinueSearch 不处理,需要其他例程或系统接着处理  
      ret  
;====================================  
Fake_Handler:    ;pop pop ret 指令串   OD 跟踪的时候在这里下个断，然后单步  
      pop edi  
      pop esi  
      ret        
;====================================  
FakeShellcode:  
        invoke MessageBox,NULL,addr szInfo,addr szCap,MB_OK  
        invoke ExitProcess,NULL  
;====================================  
end Start
```

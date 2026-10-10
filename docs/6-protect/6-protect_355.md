

# 用单步跟踪 Anti-Debugger

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-264.htm`](https://www.pediy.com/kssd/pediy08/pediy8-264.htm)

在调试器中，拙劣的 debugger 是用 0xcc 来跟踪代码，但 od 就是用标志位进行单步跟踪,简单的说就是把 TF 置位，如果我们有自己的单步跟踪代码，od 如果同时也跟踪了，就会“无视”我们自己的代码....达到 anti 的目的  
代码如下:  
;===============================cut==============================  
;Author:Comic.Liu  
  .386  
  .model  flat,stdcall  
  option  casemap  :none  
  
  include    windows.inc  
  include    user32.inc  
  includelib  user32.lib  
  include    kernel32.inc  
  includelib  kernel32.lib  
  .const  
  
  .data  
    szT    db  'Title',0  
    szM    db  'You Are Passed',0  
  .code  
SEH  proc  pExcept:DWORD,pFrame:DWORD,pContext:DWORD,pDispatch:DWORD  
    pushad  
    assume  esi:ptr EXEPTION\_RECORD,edi:ptr CONTEXT  
  
    mov  esi,\[pExcept\]  
    mov  edi,\[pContext\]  
  
    mov  ebx,\[edi\].regEip  
    inc  ebx  
    cmp  byte ptr\[ebx\],0  
    org  $-1  
    popfd  
    jz  fin  
    or  \[edi\].regFlag,100h  
    jmp  fin1  
fin:    inc  \[edi\].regEip  
fin1:    popad  
    mov  eax,ExceptionContinueExecution  
    ret  
  
  
  
SEH  endp  
  
start:  
    assume  fs:nothing  
    push  offset SEH  
    push  fs:\[0\]  
    mov  fs:\[0\],esp  
  
    pushfd  
    pushfd  
    or  dword ptr\[esp\],100h  
    popfd  
    ;进入这一段代码就翘翘了  
    nop  
    ret  
    ;=======End============  
    ;不过可以用 OD 里的 F4 直接跳过上面那一段,到下面  
    popfd  
    invoke  MessageBox,NULL,offset szM,offset szT,NULL  
  
    pop  fs:\[0\]  
    add  esp,4  
    invoke  ExitProcess,0  
    
  end  start  
;==========================finish===========================  
程序在 nop 处以 Single Step 事件进入 seh  
在 seh 里面，修改 eip 跳过代码:ret  
  
如果 od 跟踪,就截断了 single step,seh 就不会运行了，然后就出错 

这个太普通了，有经验的遇到 popfd 会不由自主看一下\[esp+1\]里面那个是不是奇数。
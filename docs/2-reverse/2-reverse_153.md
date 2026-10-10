

# 让 OD 不靠 INT3 调试

> 原文：[`www.pediy.com/kssd/pediy12/126015.html`](https://www.pediy.com/kssd/pediy12/126015.html)

dbSTI  db 0FBh  
  
MyWaitForDebugEvent proc uses esi lpDebugEvent,dwMilliseconds  
    
  invoke WaitForDebugEvent,lpDebugEvent,dwMilliseconds  
  mov esi, lpDebugEvent  
  assume esi:ptr DEBUG\_EVENT  
  .if \[esi\].dwDebugEventCode == EXCEPTION\_DEBUG\_EVENT &&\\  
    \[esi\].u.Exception.pExceptionRecord.ExceptionCode == EXCEPTION\_PRIV\_INSTRUCTION  
    mov \[esi\].u.Exception.pExceptionRecord.ExceptionCode, EXCEPTION\_BREAKPOINT  
  .endif  
  ret  
    
MyWaitForDebugEvent endp  
  
  
  local tempbuf\[5\]:BYTE  
    
comment\*  
004197AD    |. C645 F3 CC     mov byte ptr \[ebp-D\],0CC                 ;  INT 3  
00418CA4    |. 81FA CC000000  cmp edx,0CC                           ;  INT 3  
00418D4F    |. 68 CC000000    push 0CC                              ;  INT 3  
00418D06    |. 68 CC000000    push 0CC                              ;  INT3  
\*  
    
  invoke WriteProcessMemory,-1,004197ADh+3,offset dbSTI,1,NULL  
  invoke WriteProcessMemory,-1,00418CA4h+2,offset dbSTI,1,NULL  
  invoke WriteProcessMemory,-1,00418D4Fh+1,offset dbSTI,1,NULL  
  invoke WriteProcessMemory,-1,00418D06h+1,offset dbSTI,1,NULL  
    
comment\*  
00439618   . 68 14574D00    push 004D5714                         ; |pDebugEvent = OllyICE.004D5714  
0043961D   . E8 E85B0700    call <jmp.&KERNEL32.WaitForDebugEvent>; \\WaitForDebugEvent  
00439622   . 85C0           test eax,eax  
\*  
  
  lea eax, tempbuf  
  mov ecx, offset MyWaitForDebugEvent  
  sub ecx, 0043961Dh+1  
  sub ecx, 4   
  mov DWORD ptr\[eax\], ecx  
  invoke WriteProcessMemory,-1,0043961Dh+1,eax,4,0  
  
把 OD 写内存的 BUF(INT 3)换成了 sti,,并且 HOOK 了 WaitForDebugEvent 伪造消息可以达到不用 INT3 调试的目的,当然完全不用 INT 3 还有很多事情要做,但这段代码可以对抗低级 anti debug,特别是检测 CC 的  
由于用的是硬编码,也不放编译好的东东上来了,还有为了避免,,部分代码没发出来


# test anti

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-829.htm`](https://www.pediy.com/kssd/pediy08/pediy8-829.htm)

把新 3.EXE 哄出来.  
注意请先解压.![](img/77a5f7cfd3ec840c98be3aecc80df838.jpg) ![](img/77a5f7cfd3ec840c98be3aecc80df838.jpg) ![](img/77a5f7cfd3ec840c98be3aecc80df838.jpg)  
  
附件：3.rar 

  

是这个吗![](img/f5d9879e4703d3dcc963d0743af268d2.jpg)   
  
    .386  
      .model flat, stdcall  
      option casemap :none   ; case sensitive  
  
      include \\masm32\\include\\windows.inc  
      include \\masm32\\include\\user32.inc  
      include \\masm32\\include\\kernel32.inc  
  
      includelib \\masm32\\lib\\user32.lib  
      includelib \\masm32\\lib\\kernel32.lib  
  
    .data  
  Alloc dd ?  
  OLDProtect dd 02040001h  
  msgTitle db "Execution status:",0h  
  msgText1 db "No debugger detected!",0h  
  msgText2 db "Debugger detected!",0h  
    .code  
  
 start:  
  
;  MASM32   antiOlly example   
;      coded by ap0x  
;      Reversing Labs: http://ap0x.headcoders.net  
  
;  The idea is simple. OllyDBG interprets PAGE\_GUARD as a   
;  Memory break-point. If we set SEH and execute PAGE\_GUARDed  
;  code exception will occure. If debugger is present it  
;  will execute MemBpx and continue executing code after it.  
;  If debugger is not present handleing will be forwarded to SEH.  
  
;  Setup SEH  
  
  ASSUME FS:NOTHING  
  PUSH offset @Check  
  PUSH FS:\[0\]  
  MOV FS:\[0\],ESP  
  
;  Allocate new space  
  
  PUSH PAGE\_READWRITE  
  PUSH MEM\_COMMIT  
  PUSH 10000h  
  PUSH 0  
  CALL VirtualAlloc  
  
;  Write RET there  
  MOV BYTE PTR\[EAX\],0C3h  
  MOV DWORD PTR\[Alloc\],EAX  
  
;  Place Memory break-point  
  PUSH offset OLDProtect  
  PUSH PAGE\_EXECUTE\_READ OR PAGE\_GUARD  
  PUSH 00000010h  
  PUSH EAX  
  CALL VirtualProtect  
  
;  Execute  
  CALL \[Alloc\]  
  
  PUSH 30h  
  PUSH offset msgTitle  
  PUSH offset msgText2  
  PUSH 0  
  CALL MessageBox  
  
  PUSH 0  
  CALL ExitProcess  
  
;  SEH handler  
 @Check:  
  POP FS:\[0\]  
  ADD ESP,4  
  
  PUSH 40h  
  PUSH offset msgTitle  
  PUSH offset msgText1  
  PUSH 0  
  CALL MessageBox  
  
  PUSH 0  
  CALL ExitProcess  
  
 end start 

OK,表示正常;"怎么是你"是在 OD 中的第一提示,也就是 softworm 说的,之后大家都会跳过这个;接着会进入由 2 个 ANTI 组成的检测,出现"怎么老是你".![](img/f5d9879e4703d3dcc963d0743af268d2.jpg) ![](img/f5d9879e4703d3dcc963d0743af268d2.jpg) ![](img/f5d9879e4703d3dcc963d0743af268d2.jpg)  
正常情况下,只要父进程不对就会被检测出来,而且即使使用了注入也没有用;但在特殊情况下就不好说了,比如有流氓软件或 XX 驱动的情况下也许会挂,只是我还没遇见. 

if (PEB.ProcessParameters.StandardOutput == 0)  
{  
} 

引用:  *最初由** heXer **发布*  
if (PEB.ProcessParameters.StandardOutput == 0)  
{  
} 

  
  
这个不知道，学习了顶  
我只到 80000001~
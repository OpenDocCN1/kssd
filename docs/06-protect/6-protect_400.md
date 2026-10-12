
# Anti OllyDbg

> 原文：[`www.pediy.com/kssd/pediy10/93316.html`](https://www.pediy.com/kssd/pediy10/93316.html)

在论坛看到一老兄求助地址（[http://bbs.pediy.com/showthread.php?t=92330](http://bbs.pediy.com/showthread.php?t=92330)），对其开始异常处理分析了一下。

该 anti 手段是利用 `OD` 的单步机制来实现的。

所以 anti 的前提是:单步![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")

在钱老师的鼓励之下逆向了 `OD` 的 `step`。

如有不妥 请勿见怪。

测试请见附件中的 `demo`。

既然说是利用单步来的，请先走到开头处构造的异常语句处，然后 单步一下 最后 `Run` 起来就可以了。

先上逆向分析过程的代码。

首先用 `OD` `load` 官方原版 `od 1.10` `F9` `run` 起来。

被调试 `od` 载入附件中 `demo` 程序 事先走到 `mov eax,eax ;eax=0` 这句来。

然后回到调试 `od` 中 `bp SetThreadContext` 和 `WaitForDebugEvent`。

为了方便 在调试 `od` 中忽略了内存访问异常。

然后在被调试 `od` 中 `f8` 一下。

```
0042EB1A  |> \53             |PUSH EBX                     ; /pContext  
0042EB1B  |.  8B45 EC        |MOV EAX,DWORD PTR SS:[EBP-14]                  ; |  
0042EB1E  |.  8B50 0C        |MOV EDX,DWORD PTR DS:[EAX+C]                  ; |  
0042EB21  |.  52             |PUSH EDX                     ; |hThread  
0042EB22  |.  E8 83060800    |CALL <JMP.&KERNEL32.SetThreadContext>          ; \SetThreadContext
```

这段代码处于 `Go` 函数里面 该 `Go` 函数被调用于单步异常模块中。

先不管它 `F9` 让它跑下那句异常代码先。

```
00439616   > \6A 00         PUSH 0                          ; /Timeout = 0. ms  
00439618   .  68 14574D00   PUSH OFFSET <OLLYDBG.DebugEvent.dwDebugEventCode>     ; |pDebugEvent = OFFSET <OLLYDBG.DebugEvent.dwDebugEventCode>  
0043961D   .  E8 E85B0700    CALL <JMP.&KERNEL32.WaitForDebugEvent>             ; \WaitForDebugEvent  
00439622   .  85C0           TEST EAX,EAX  
00439624   .  75 44          JNZ SHORT OLLYDBG.0043966A
```

;004D5714 为全局的 `DEBUG_EVENT` 结构

```
004D5714>   00000001    00000558    0000078C    C0000005 ;ExceptionCode  
004D5724     00000000    00000000    00401011  ;ExceptionAddress
```

这时候就捕获了内存访问异常，跟着它的处理 来到

```
00439743   .  833D 14574D00>CMP DWORD PTR DS:[<DebugEvent.dwDebugEventCode>],1  ;判断是不是 EXCEPTION_DEBUG_EVENT  
0043974A   .  75 0E         JNZ SHORT OLLYDBG.0043975A  
0043974C   .  8B0D 2C574D00 MOV ECX,DWORD PTR DS:[4D572C]                     ;  OLLYDBG.00401011        ;当前指令地址  
00439752   .  3B0D 30814D00 CMP ECX,DWORD PTR DS:[4D8130]                     ;  OLLYDBG.00401013  ;理想中的下一条指令地址  
00439758   .  /74 0A        JE SHORT OLLYDBG.00439764  
0043975A   > |C705 7C7C4D00>MOV DWORD PTR DS:[4D7C7C],1  
00439764   > \33C0         XOR EAX,EAX  
00439766   .  33D2         XOR EDX,EDX  
00439768   .  A3 30814D00   MOV DWORD PTR DS:[4D8130],EAX  
0043976D   .  8D4D AC       LEA ECX,DWORD PTR SS:[EBP-54]  
00439770   .  C605 203A4E00>MOV BYTE PTR DS:[4E3A20],0  
00439777   .  8915 543B4E00 MOV DWORD PTR DS:[4E3B54],EDX  
0043977D   .  51            PUSH ECX  
0043977E   .  E8 4D54FFFF    CALL OLLYDBG.0042EBD0  ;执行一些前奏 比如此时是内存访问异常 这里会先判断下是否处于 kernel32 和是否选择了忽略在 kernel32 内存中的异常 和 sprintf 一些 UI 字符串"Access violation when %s [%08lX]%s"这些
```

然后就到了这里 因为要单步了。

```
00439964   .  /75 24        JNZ SHORT OLLYDBG.0043998A  
00439966   .  |833D CC574D00>CMP DWORD PTR DS:[4D57CC],8  
0043996D   .  |74 1B        JE SHORT OLLYDBG.0043998A  
0043996F   .  |6A 00        PUSH 0                          ; /Arg5 = 00000000  
00439971   .  |6A 00        PUSH 0                          ; |Arg4 = 00000000  
00439973   .  |6A 00        PUSH 0                          ; |Arg3 = 00000000  
00439975   .  |6A 00        PUSH 0                          ; |Arg2 = 00000000  
00439977   .  |A1 1C574D00   MOV EAX,DWORD PTR DS:[<DebugEvent.dwThreadId>]     ; |  
0043997C   .  |50           PUSH EAX                        ; |Arg1 => 0000078C  
0043997D   .  |E8 92B0FFFF    CALL OLLYDBG._Go              ; \\_Go
```

`Go` 的原型:

```
int Go(ulong threadid,ulong tilladdr,int stepmode,int givechance,int backupregs);
```

关于参数三:

`stepmode - stepping mode, one of the following:`

*   `STEP_SAME(0)`: Same action as on previous call to `Go`
*   `STEP_RUN(1)`: Run program
*   `STEP_OVER(2)`: Step over (execute calls at once)
*   `STEP_IN(3)`: Step in (enter subroutines)
*   `STEP_SKIP(4)`: Skip sequence till specified address

OllyDbg Plugin API v1.10

由于上一步为单步，所以在函数实现内部。

```
00434A80  |> \837D 10 00    CMP DWORD PTR SS:[EBP+10],0   ;判断是否为 STEP_SAME  
00434A84  |.  75 11         JNZ SHORT OLLYDBG.00434A97  
00434A86  |.  8B56 7C       MOV EDX,DWORD PTR DS:[ESI+7C] ;取出上步操作  
00434A89  |.  8955 10       MOV DWORD PTR SS:[EBP+10],EDX ;STEP_OVER(2)
```

所以此时变成了单步了。

```
00434C88  |.  837D 10 02    CMP DWORD PTR SS:[EBP+10],2  
00434C8C  |.  0F94C0        SETE AL  
00434C8F  |.  83E0 01       AND EAX,1  
00434C92  |.  83FF 70       CMP EDI,70  
00434C95  |.  8945 F8       MOV DWORD PTR SS:[EBP-8],EAX  
00434C98  |.  0F85 41010000 JNZ OLLYDBG.00434DDF   ;判断是否为单步
```

最后来到此处:

```
0043534F  |.  6A 00         PUSH 0                          ; /Arg4 = 00000000  
00435351  |.  A3 583B4E00   MOV DWORD PTR DS:[4E3B58],EAX           ; |  
00435356  |.  8B55 18       MOV EDX,DWORD PTR SS:[EBP+18]           ; |  
00435359  |.  C705 543B4E00>MOV DWORD PTR DS:[4E3B54],1            ; |  
00435363  |.  52            PUSH EDX                        ; |Arg3  
00435364  |.  8B4D 14       MOV ECX,DWORD PTR SS:[EBP+14]           ; |  
00435367  |.  51            PUSH ECX                        ; |Arg2  
00435368  |.  8B45 08       MOV EAX,DWORD PTR SS:[EBP+8]            ; |  
0043536B  |.  50            PUSH EAX                        ; 线程 ID  
0043536C  |.  E8 BFC0FFFF    CALL OLLYDBG.00431430                ; \\OLLYDBG.00431430    ;跟进  
00431498  |.  E8 3FD3FFFF    CALL <OLLYDBG.SetContext>             ; \\OLLYDBG.0042E7DC    ; 该函数遍历被调试进程的各个线程 然后设置其 CONTEXT  
0042E8CA  |.  F680 F4020000>|TEST BYTE PTR DS:[EAX+2F4],1  
0042E8D1  |.  74 11         |JE SHORT OLLYDBG.0042E8E4  
0042E8D3  |.  818B C0000000>|OR DWORD PTR DS:[EBX+C0],100
```

`eax` 是一个 `t_thread` 结构

```c
typedef struct t_thread {           // Information about active threads  
  ulong          threadid;         // Thread identifier  
  ulong          dummy;            // Always 1  
  ulong          type;             // Service information, TY_xxx  
  HANDLE         thread;           // Thread handle  
  ulong          datablock;        // Per-thread data block  
  ulong          entry;            // Thread entry point  
  ulong          stacktop;         // Working variable of Listmemory()  
  ulong          stackbottom;      // Working variable of Listmemory()  
  CONTEXT        context;          // Actual context of the thread  
  t_reg          reg;              // Actual contents of registers  
  int            regvalid;         // Whether reg is valid  
  t_reg          oldreg;           // Previous contents of registers  
  int            oldregvalid;      // Whether oldreg is valid  
  int            suspendcount;     // Suspension count (may be negative)  
  long           usertime;         // Time in user mode, 1/10th ms, or -1  
  long           systime;          // Time in system mode, 1/10th ms, or -1  
  ulong          reserved[16];     // Reserved for future compatibility  
} t_thread;  

typedef struct t_reg {            // Excerpt from context  
  int            modified;        // Some regs modified, update context  
  int            modifiedbyuser;  // Among modified, some modified by user  
  int            singlestep;      // Type of single step, SS_xxx  
  ulong          r[8];            // EAX,ECX,EDX,EBX,ESP,EBP,ESI,EDI
```



```
  ulong          ip;                   // Instruction pointer (EIP)  

  ulong          flags;                // Flags  

  int            top;                  // Index of top-of-stack  

  long double    f[8];                 // Float registers, f[top] - top of stack  

  char           tag[8];               // Float tags (0x3 - empty register)  

  ulong          fst;                  // FPU status word  

  ulong          fcw;                  // FPU control word  

  ulong          s[6];                 // Segment registers ES,CS,SS,DS,FS,GS  

  ulong          base[6];              // Segment bases  

  ulong          limit[6];             // Segment limits  

  char           big[6];               // Default size (0-16, 1-32 bit)  

  ulong          dr6;                  // Debug register DR6  

  ulong          threadid;             // ID of thread that owns registers  

  ulong          lasterror;            // Last thread error or 0xFFFFFFFF  

  int            ssevalid;             // Whether SSE registers valid  

  int            ssemodified;          // Whether SSE registers modified  

  char           ssereg[8][16];        // SSE registers  

  ulong          mxcsr;                // SSE control and status register  

  int            selected;             // Reports selected register to plugin  

  ulong          drlin[4];             // Debug registers DR0..DR3  

  ulong          dr7;                  // Debug register DR7  

} t_reg;  
```

最后也就是开头处的  

```
0042EB1A  |> \53             |PUSH EBX                                  ; /pContext  

0042EB1B  |.   8B45 EC       |MOV EAX,DWORD PTR SS:[EBP-14]            ; |  

0042EB1E  |.   8B50 0C       |MOV EDX,DWORD PTR DS:[EAX+C]             ; |  

0042EB21  |.   52            |PUSH EDX                                  ; |hThread  

0042EB22  |.   E8 83060800   |CALL <JMP.&KERNEL32.SetThreadContext>     ; \SetThreadContext  
```

偏移`2F4`处就是`singlestep`字段 将此线程`CONTEXT.EFlags |= 100h`了  

走到系统处理异常领空之后 虽然这时的`TF`被清位了  

但此时的`CONTEXT`结构被作为异常处理回调函数的参数了  

当回调函数返回值为`ExceptionContinueExection` 也即`0`的时候  

系统就会把线程环境设置为`CONTEXT`里的结构并继续执行  

导致异常处理函数返回值 又增加了一个中断异常 这个也是该`anti`所使用的机制  

由于此`anti`是利用了调试器的单步机制  

所以避免方法就是:  

`F9`过去 也就只是避免单步执行异常语句![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")  

菜鸟玩逆向 难免嗦了点  

感谢钱老师的启发![](img/8e0d77ee22c3d8438aad7d42c7829e53.jpg "使眼色")  

by 科锐三期学员

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `anti_od_demo.rar`  

 |

另附 demo 源码  

```
.386  

.model flat,stdcall  

option casemap:none  

include   user32.inc  

includelib  user32.lib  

.code  

start:  

  xor   eax,eax  

  assume fs:nothing  

  push  _SEH_PROC  

  push   fs:[eax]  

  mov   fs:[0],esp  

  mov   [eax],eax  

  jmp   _NOT_FOUND  

  jmp    _FOUND  

_NOT_FOUND:  

  push  0  

  call   @f  

  db 'Demo',0  

@@:  

  call  @f  

  db 'Not Found!',0  

@@:   

  push   0  

  call   MessageBox  

  add   esp,8  

  ret  

_FOUND:  

  push  0  

  call   @f  

  db 'Demo',0  

@@:  

  call  @f  

  db 'Found OllyDbg!',0  

@@:   

  push   0  

  call   MessageBox  

  add   esp,8  

  ret    

_SEH_PROC:  

  mov   edx,[esp+4]  

  mov   edx,DWORD ptr ds:[edx+0Ch]  

  mov   word ptr ds:[edx],0EBh  

  xor   eax,eax  

  ret  

end start
```

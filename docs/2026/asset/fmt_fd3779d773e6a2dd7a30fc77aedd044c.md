

```
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

最后也就是开头处的：

```
0042EB1A  |> \53            |PUSH EBX                                              ; /pContext
0042EB1B  |.  8B45 EC       |MOV EAX,DWORD PTR SS:[EBP-14]                         ; |
0042EB1E  |.  8B50 0C       |MOV EDX,DWORD PTR DS:[EAX+C]                          ; |
0042EB21  |.  52            |PUSH EDX                                              ; |hThread
0042EB22  |.  E8 83060800   |CALL <JMP.&KERNEL32.SetThreadContext>                 ; \SetThreadContext
```

偏移2F4处就是`singlestep`字段，将此线程`CONTEXT.EFlags |= 100h`了。

走到系统处理异常领空之后，虽然这时的`TF`被清位了，但此时的`CONTEXT`结构被作为异常处理回调函数的参数了。

当回调函数返回值为`ExceptionContinueExection`，也即0的时候，系统就会把线程环境设置为`CONTEXT`里的结构并继续执行。

导致异常处理函数返回值，又增加了一个中断异常，这个也是该anti所使用的机制。

由于此anti是利用了调试器的单步机制，所以避免方法就是：

F9过去，也就只是避免单步执行异常语句![](https://bbs.kanxue.com/view/img/face/16.gif)

菜鸟玩逆向，难免啰嗦了点。

感谢钱老师的启发![](https://bbs.kanxue.com/view/img/face/41.gif)

by科锐三期学员

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [anti\_od\_demo.rar](javascript:void(0);)（0.48kb，78次下载）

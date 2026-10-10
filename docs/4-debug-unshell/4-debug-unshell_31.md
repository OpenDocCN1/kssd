

# ESP 定律背后的原理之一：0x12FFC4，为什么是这个数

> 原文：[`www.pediy.com/kssd/pediy10/59352.html`](https://www.pediy.com/kssd/pediy10/59352.html)

ESP 定律背后的原理之一：0x12FFC4  
ESP 定律：如果压缩壳执行前后的堆栈会维持平衡，在 12FFC0 上下读写断点，会中断在 OEP 的第一条指令 push ebp 上。那么各位有没有探究过 ESP 的尾数值为什么不是 0xFFFC，而是 0x0FFC4？这是由 windows 源代码的实现所确定的。  
  
Windows 创建进程的函数链为 kernel32.dll!\_CreateProcessInternalW -> …　-> kernel32.dll!\_BaseProcessStartThunk() -> kernel32.dll!\_BaseProcessStart() -> EP  
  
在进入\_BaseProcessStartThunk()时，堆栈的状态确实为  
+ 0012FFFC   00000000                <- ESP  
  
\---------------------------------------------------------<code>  
   \_BaseProcessStartThunk@8 proc near  
xor     ebp, ebp  
push    eax  
push    0  
jmp     \_BaseProcessStart@4 ; BaseProcessStart(x)  
\_BaseProcessStartThunk@8 endp  
\----------------------------------------------------------</code>  
  
在执行完\_BaseProcessStartThunk 时，堆栈的状态为  
+ 0012FFFC   00000000                  
+ 0012FFF8   EAX  
+ 0012FFF4   0x00000000             <- ESP  
      
\---------------------------------------------------------<code>  
    \_BaseProcessStart proc  
push    0Ch  
push    offset dword\_7C816FE0  
call    \_\_SEH\_prolog  
and     \[ebp+ms\_exc.disabled\], 0  
push    4  
lea     eax, \[ebp+8\]  
push    eax  
push    9  
push    0FFFFFFFEh  
call    ds:\_\_imp\_\_NtSetInformationThread@16 ; NtSetInformationThread(x,x,x,x)  
call    dword ptr \[ebp+8\]  
push    eax             ; dwExitCode  
call    \_ExitThread@4  
nop  
nop  
nop  
\---------------------------------------------------------</code>  
  
在执行完 BaseProcessStart()后，堆栈的状态为  
  
+ 0012FFFC   0x00000000  
+ 0012FFF8   OldEAX -2-> EntryPoint By NtSetInformationThread()  
+ 0012FFF4   reserved, inited to 0x00000000  
+ 0012FFF0   OldEBP(0x00000000)    <-p- NowEBP  
+ 0012FFEC   TryLevel, inited to 0xFFFFFFFF  
+ 0012FFE8   ScopeTable  
+ 0012FFE4   \_\_except\_handler3 in kernel32.dll  
+ 0012FFE0   0xFFFFFFFF            <-p- fs:00h  
+ 0012FFDC   ExceptionInfo, Parameter of UnhandledExceptionFilter()  
+ 0012FFD8   OldESP(0x0012FFC8)  
+ 0012FFD4   ExceptionCode, Parameter of kernel32.dll!ExitProcess  
+ 0012FFD0   EBX(pPEB)  
+ 0012FFCC   ESI  
+ 0012FFC8   EDI                   <-p- OldESP  
+ 0012FFC4   retaddr in kernel32.dll!\_BaseProcessStartThunk  
                                               <-p- NowESP  
  
  
附：  
    DWORD   SizeOfStackReserve;                         // 0x48-4b  
    DWORD   SizeOfStackCommit;                          // 0x4c-4f  
    DWORD   SizeOfHeapReserve;                          // 0x50-53  
    DWORD   SizeOfHeapCommit;                           // 0x54-57  
  
这是 IOH 中的四个域。  
Reserved 保留的, Committed 提交的是两种类型的虚拟内存。  
Reserved 指这段内存已有用途，不能再被分配；Committed 提交的内存指的是已经与物理内存有映射的虚拟内存段（当然也可以被移到硬盘上的缓存文件 pagefile.sys 中）。因为 Stack, Heap 这两种类型的内存段都有随着程序的运行尺寸不断增大的特性，所以一开始不必占用太多宝贵的物理内存，当它们用完了，会引发内存异常，windows 内存管理器就又为程序分配一部分。  
  
SizeOfStackReserve  程序开始时保留用作堆栈空间的内存段的大小  
SizeOfStackCommit   程序开始时在保留的堆栈内存段中与物理内存映射的提交的内存段的大小  
SizeOfHeapReserve  程序开始时保留用作堆空间的内存段的大小  
SizeOfHeapCommit   程序开始时在保留的堆内存段中与物理内存映射的提交的内存段的大小 

－－－－－－－－－－－－－－－－－－－  
注意：并不是所有程序加载时，ESP 的值是 0012FFC4，这个值是什么由操作系统决定，将 SizeOfStackCOmmit 改大 ESP 就会变，这是因为操作系统从这个页向上找一个足够大地方当作 stack 了（感谢 forgot 解释）。  
－－－－－－－－－－－－－－－－－－－  
  
我曾仔细看过这段话，正是这句话引发了我探究的兴趣，这句话的错误在于错误理解了域 SizeOfStackCommit, SizeOfStackReserve 的作用，SizeOfStackCommit 表示提交到物理内存的虚拟内存的大小，SizeOfStackReserve 表示分配给当前程序的大小，注意是大小，不是地址。它们决定了程序初始化时堆栈空间的 Commited 内存的大小和 Reserved 内存的大小，但不是地址。  
  
堆栈空间的顶端的值是否固定，我没有逆向 windows，所以没有发言权，但不管堆栈空间的顶端是 0x00130000，还是 0x00120000，还是 0x00110000，只要这里我提到的几个函数的源代码实现不变，最后几位的值就不会变：0x0012FFC4, 0x0011FFC4, 0x0010FFC4。 

操作系统的不同，源代码的实现有可能不同，这个值有可能不同。这个道理，我在文章中说明了。  
2K, XP, Vista, Checked Version, Free Version 都有可能不同。  
我明白你们的意思，我写下来只是想弄清楚在堆栈上面到底存储了些什么东西。并不是探讨堆栈的顶端是否固定的问题。
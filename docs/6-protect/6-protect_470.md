

# ProcDump 中的隐藏自己的方法

> 原文：[`www.pediy.com/kssd/pediy03/forum869.htm`](https://www.pediy.com/kssd/pediy03/forum869.htm)

欢迎到小站坐坐 http://st0ne.yeah.net   
『技术研习』  『以前更新』    
  
\--------------------------------------------------------------------------------   
  
  
ProcDump 中的隐藏自己的方法   
  
【声明】   
我写文章以交流为主，希望大家在转载时能保持文章的完整性。   
  
【前言】   
运行 ProcDump 的 Script 时，总能看到其开始时会显示一段"Hiding Debugger to host ..."的信息，这到底是做什么用的呢？我们来看看吧。   
  
作者：        ljtt   
写作日期：    2001-04-19   
  
以下是用 W32Dasm 反汇编得到的结果。   
  
:00408E15 6A00                    push 00000000   
  
\* Possible StringData Ref from Data Obj ->"Hiding Debugger to host ..."   
                                  |   
:00408E17 6833C44000              push 0040C433   
:00408E1C E89E88FFFF              call 004016BF                            <--显示"Hiding Debugger to host ..."   
:00408E21 6814204100              push 00412014   
:00408E26 FF35E4204100            push dword ptr \[004120E4\]   
  
\* Reference To: KERNEL32.GetThreadContext, Ord:0152h   
                                  |   
:00408E2C E8150A0000              Call 00409846                            <--取得线程上下文内容到 00412014 地址处   
:00408E31 52                      push edx   
:00408E32 53                      push ebx   
:00408E33 50                      push eax   
:00408E34 8CC0                    mov ax, es   
:00408E36 50                      push eax   
:00408E37 A1A4204100              mov eax, dword ptr \[004120A4\]            <--取得 FS 段选择器的内容   
:00408E3C 8EC0                    mov es, ax   
:00408E3E 26C7052000000000000000  mov dword ptr es:\[00000020\], 00000000    <--设置 DebugContext = 0   
:00408E49 58                      pop eax   
:00408E4A 8EC0                    mov es, ax   
:00408E4C 58                      pop eax   
:00408E4D 5B                      pop ebx   
:00408E4E 5A                      pop edx   
  
我们不妨来分析一下吧。   
  
程序首先用 GetThreadContext 函数来取得线程上下文的内容，此也就是 \_CONTEXT 结构的内容，并保存在 00412014 地址处。   
( \_CONTEXT 结构的定义附后 )   
  
  
然后，程序取 \[004120A4\] 的内容到 EAX 中，并赋给 ES 段选择器中。   
  
:00408E37 A1A4204100              mov eax, dword ptr \[004120A4\]            <--取得 FS 选择器的内容   
:00408E3C 8EC0                    mov es, ax   
  
\[004120A4\]中保存的到底是什么呢？从 \_CONTEXT 结构的定义中我们知道其偏移 90h 处保存的是当前 FS 段选择器的值，而 \[00412014 + 90\] = \[004120A4\] ，因此 \[004120A4\] 保存的即为 FS 段选择器的值。这两条指令的作用即为把 FS 段选择器的值赋给 ES 段选择器。   
  
然后，程序赋 ES:\[20\] 的值为 0，这有何用意呢？我们知道 Windows 在调入进程，创建线程时，操作系统均会为每个线程分配 TEB，而且都将 FS 段选择器(i386)指向当前线程的 TEB 数据(单 CPU 机器在任何时刻系统中只有一条线程在执行)，因此此时 FS 段选择器即为 TEB 结构的指针。   
看来，我们需要了解一下 TEB 结构的知识了，以下是有关 TEB 结构的知识及相关定义：   
  
    TEB 在 Windows 9x 系列中称为 TIB(Thread Information Block)，她纪录着线程的重要信息，每一个线程对应一个 TEB 结构。其格式如下(摘自 Matt Pietrek 的 Under the Hood 专栏－MSJ 1996):   
  
    typedef struct \_TIB   
    {   
        PEXCEPTION\_REGISTRATION\_RECORD pvExcept;   
        // 00h Head of exception record list   
        PVOID  pvStackUserTop;    // 04h  Top of user stack   
        PVOID  pvStackUserBase;    // 08h  Base of user stack   
  
        union                      // 0Ch (NT/Win95 differences)   
        {   
            struct  // Win95 fields   
                {   
                    WORD    pvTDB;        // 0Ch TDB   
                    WORD    pvThunkSS;      
                // 0Eh SS selector  used for thunking to 16 bits   
                    DWORD  unknown1;      // 10h   
                } WIN95;   
  
            struct  // WinNT fields   
                {   
                    PVOID SubSystemTib;    // 0Ch   
                    ULONG FiberData;        // 10h   
                } WINNT;   
        } TIB\_UNION1;   
  
        PVOID  pvArbitrary;          
        // 14h Available for application use   
        struct \_tib \*ptibSelf;        
        // 18h Linear address of TIB structure   
  
        union                      // 1Ch (NT/Win95 differences)   
        {   
                struct  // Win95  fields   
                {   
                    WORD    TIBFlags;          // 1Ch   
                    WORD    Win16MutexCount;    // 1Eh   
                    DWORD  DebugContext;      // 20h   
                    DWORD  pCurrentPriority;  // 24h   
                    DWORD  pvQueue;              
                    // 28h  Message Queue selector   
                } WIN95;   
  
                struct  // WinNT  fields   
                {   
                    DWORD unknown1;            // 1Ch   
                    DWORD processID;            // 20h   
                    DWORD threadID;            // 24h   
                    DWORD unknown2;            // 28h   
                } WINNT;   
        } TIB\_UNION2;   
  
        PVOID\*  pvTLSArray;        // 2Ch Thread Local Storage array   
  
        union                      // 30h (NT/Win95 differences)   
        {   
            struct  // Win95 fields   
            {   
                PVOID\*  pProcess;      
                // 30h Pointer to  owning process database   
            } WIN95;   
        } TIB\_UNION3;   
      
    } TIB, \*PTIB;   
  
了解了这些以后，我们再理解上面指令的作用不再困难，TEB 结构的偏移 20h 处在 Win9x 的定义中为 DebugContext，因此 ES:\[20\] = 0， 即为 FS:\[20\] = 0，即为设置 TEB 结构的 DebugContext 为 0。   
  
现在，我们明白了为什么有些情况 ProcDump 可以加载并进行脱壳的程序，而你编写的程序却无法把它作为一个子进程来加载它，原来 ProcDump 有这一手。:-)   
  
【后记】   
虽然这种方法现在已经不足以对付外壳加密程序的反跟踪保护了，但是我们学习的目的正在于了解它，掌握它，从而有所创新。   
我想当我们对此类知识了解得更多的的时候，对与此相关的新的变化的跟踪方法也就能成竹在胸了。如果你有更多、更好的跟踪方法也请告诉我哦。:-)   
  
\[附：\_CONTEXT 结构定义\]   
\_CONTEXT            STRUC   
cx\_ContextFlags    DD ?   
  
;CONTEXT\_DEBUG\_REGISTERS   
cx\_Dr0                DD ?        ;04   
cx\_Dr1                DD ?        ;08   
cx\_Dr2                DD ?        ;0C   
cx\_Dr3                DD ?        ;10   
cx\_Dr6                DD ?        ;14   
cx\_Dr7                DD ?        ;18   
  
;CONTEXT\_FLOATING\_POINT   
cx\_ControlWord            DD ?   
cx\_StatusWord            DD ?   
cx\_TagWord                DD ?   
cx\_ErrorOffset            DD ?   
cx\_ErrorSelector        DD ?   
cx\_DataOffset            DD ?   
cx\_DataSelector        DD ?   
SIZE\_OF\_80387\_REGISTERS        EQU 80   
cx\_RegisterArea        DB SIZE\_OF\_80387\_REGISTERS DUP (?)   
cx\_Cr0NpxState            DD ?   
  
;CONTEXT\_SEGMENTS   
cx\_SegGs            DD ?        ;8C   
cx\_SegFs            DD ?        ;90   
cx\_SegEs            DD ?        ;94   
cx\_SegDs            DD ?        ;98   
  
;CONTEXT\_INTEGER   
cx\_Edi                DD ?        ;9C   
cx\_Esi                DD ?        ;A0   
cx\_Ebx                DD ?        ;A4   
cx\_Edx                DD ?        ;A8   
cx\_Ecx                DD ?        ;AC   
cx\_Eax                DD ?        ;B0   
  
;CONTEXT\_CONTROL   
cx\_Ebp                DD ?        ;B4   
cx\_Eip                DD ?        ;B8   
cx\_SegCs            DD ?        ;BC   
cx\_EFlags            DD ?        ;C0   
cx\_Esp                DD ?        ;C4   
cx\_SegSs            DD ?        ;C8   
\_CONTEXT            ENDS   
;size of CONTEXT is 0CCH bytes   
  
  
\--------------------------------------------------------------------------------   
  
《加密及解密技术交流站》由 ljtt 制作 版权所有   
&copy;2000 -2001 All Rights Reserved   
    
转载本站所有文章请注明出处，尊重作者的劳动也是尊重你自己。让我们一同撑起绿色的天空！  4dd466d0e1e0cc123afbe504da2462ca
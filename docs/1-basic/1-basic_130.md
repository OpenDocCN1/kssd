

# 调试寄存器（DRx）理论与实践

> 原文：[`www.pediy.com/kssd/pediy06/pediy6751.htm`](https://www.pediy.com/kssd/pediy06/pediy6751.htm)

  
调试寄存器（DRx）理论与实践  
By Hume/冷雨飘心  
前言+牢骚：  
生活的苦痛就象烈火，时时煎熬着伤痕累累疲惫不堪的那颗心。我拼力挣扎，然而却无济于事……  
太残酷了….上帝也在苦笑。  
                                              题记  
  
很多人问 Drx 调试寄存器的用法，网上实际上有很多资料，但是很多人还是不肯去翻，于是我来写点东西，算是给对 DRx 尚有疑惑的解答吧。不要在 QQ 上问我任何问题，如果有错误，请来信指明：Humewen@21cn.com。  
  
1.一点理论  
＝＝＝＝＝  
本文假设你已经知道调试器的使用方法以及 bpx，bpm，int 3，异常都分别是什么，本文不是一篇超级扫盲教程。  
Intel 80386 以上的 CPU 提供了调试寄存器以用于软件调试。386 和 486 包括 6 个调试寄存器：Dr0，Dr1，Dr2，Dr3，Dr6 和 Dr7。这些寄存器全是 32 位，如下图所示：  
  
   |－－－－－－－－－－－－－－－|－－－－－－－－－－－－－－－－|  
Dr0|                 用于一般断点的线性地址                      
   |－－－－－－－－－－－－－－－|－－－－－－－－－－－－－－－－|  
Dr1|                 用于一般断点的线性地址                      
   |－－－－－－－－－－－－－－－|－－－－－－－－－－－－－－－－|  
Dr2|                 用于一般断点的线性地址                      
   |－－－－－－－－－－－－－－－|－－－－－－－－－－－－－－－－|  
Dr3|                 用于一般断点的线性地址                      
   |－－－－－－－－－－－－－－－|－－－－－－－－－－－－－－－－|  
Dr4|                     保留                                  
   |－－－－－－－－－－－－－－－|－－－－－－－－－－－－－－－－|  
Dr5|                     保留                                  
   |－－－－－－－－－－－－－－－|－－－－－－－－－－－－－－－－|  
Dr6|                              |BBB                     BBB B |  
   |                              |TSD                      3 2 1 0 |  
   |－－－－－－－－－－－－－－－|－－－－－－－－－－－－－－－－|  
Dr7|RWE LEN   ...    RWE LEN    |  G               GLGLGLGLGL |  
   | 3   3    ...        0    0     |  D               E E 3 3 2 21 100 |  
   |－－－－－－－－－－－－－－－|－－－－－－－－－－－－－－－－|  
31                            15                                0  
  
  
Dr0~3 用于设置硬件断点，即在调试器中经常使用的 bpm 断点，由于只有 4 个断点寄存器，所以最多只能设置 4 个 bpm 断点。Dr7 是一些控制位，用于控制断点的方式，Dr6 用于显示是哪些引起断点的原因，如果是 Dr0～3 或单步（EFLAGS 的 TF）或由于 GD 置位时访问调试寄存器引起 1 号调试陷阱的话，则相应设置对应的位。下面对 Dr6 和 Dr7 的对应位做一些详细介绍：  
  
调试控制寄存器 Dr7：  
＝＝＝＝＝＝＝＝＝＝  
位 0 L0 和位 1 G0：用于控制 Dr0 是全局断点还是局部断点，如果 G0 置位则是全局断点，L0 置位则是局部断点。  
G1L1～G3L3 用于控制 D1～Dr3，其功能同上。  
  
LEN0：占两个位，开始于位 15，用于控制 Dr0 的断点长度，可能取值：  
00  1 字节  
01  2 字节  
10  保留  
11  4 字节  
RWE0：从第 17 位开始，占两个位，控制 Dr0 的断点是读、写还是执行断点或是 I/O 端口断点：  
00  只执行  
01 写入数据断点  
10 I/O 端口断点（只用于 pentium+，需设置 CR4 的 DE 位）  
11 读或写数据断点  
RWE1～3，LEN1～3 分别用于控制 Dr1～3 的断点方式，含义如上。  
  
还有一个 GD 位：用于保护 DRx，如果 GD 位为 1，则对 Drx 的任何访问都会导致进入 1 号调试陷阱。即 IDT 的对应入口，这样可以保证调试器在必要的时候完全控制 Drx。  
  
调试状态寄存器 Dr6：  
＝＝＝＝＝＝＝＝＝  
该寄存器用于表示进入陷阱 1 的原因，各个位的含义如下：  
B0～B3，如果其中任何一个位置位，则表示是相应的 Dr0~3 断点引发的调试陷阱。但还需注意的是，有时候不管 GiLi 如何设置，只要是遇到 Drx 指定的断点，总会设置 Bi，如果看到多个 Bi 置位，则可以通过 GiLi 的情况判断究竟是哪个 Dr 寄存器引发的调试陷阱。  
BD 置位表示是 GD 位置位情况下访问调试寄存器引发的陷阱。  
BT 置位表示是因为 TS 置位即任务切换时 TSS 中 TS 位置 1 时切到第二个任务时第一条指令时引发的。  
BS 置位表示是单步中断引发的断点。。。。即 EFLAGS 的 TF 置位时引发的调试陷阱。  
  
注意 I/O 端口断点是 586+以上 CPU 才有的功能，受 CR4 的 DE 位的控制，DE 为 1 才有效。（DE 是 CR4 的第 3 位）。  
  
2、一点常识  
＝＝＝＝＝＝  
如果你使用调试器的话，一定清楚 bpx 断点，bpx 实际上就通过在代码中插入 int 3（0xCC 或 0xCD03），将引发 int 3 中断。具体的 intel IA-32 保护模式异常机制并不是我三言两语能解释清楚的，如必要请参照相关资料。  
Bpm 和 BPIO 断点是利用 CPU 的硬件调试器设置的断点。  
有些调试功能只在 586+以上 CPU 才能使用，为增强兼容性用 CPUID 测试。（测试方法见我后续文章。）  
关于调试 API，是 Windows 提供给开发者的调试原 API。具体实现涉及 windows 的内部机制，不是简单的 int x 就能解释清楚的，有兴趣者可参阅相关资料，或等我有时间胡说一通。  
  
  
3、一点实践  
  
设置 BPM 断点很简单，只要相应设置 Drx 即可，产生的异常是 STATUS\_SINGLE\_STEP，只要用调试 API 或 SEH 或 VEH 处理一下即可。下面是 9X 下测试 BPM 断点的一个例子：  
  
A：：－>  
\-----------  
COMMENT/\*  
  
仅工作于 9X 下，BPM 产生的异常类型是 SINGLE\_STEP  
可用来对付调试器，反跟踪，至于效果，试试就知道了  
这里采用 SEH 来设置 9x 下的 Drx 寄存器  
还可采用 GetThreadContext、SetThreadContext 设置 Drx 的值  
NT 类系统由于安全机制，无法使用该办法  
需使用 Debug 技术  
  
\*/  
include c:\\hd\\hhd.h  
include c:\\hd\\drx.h  
ASSUME FS:NOTHING  
;~~~~~~~~~~~~~~~~~~~  
.CODE  
\_StArT:  
        int 3  
        SLDT    cx  
        JCXZ    isNT  
        JMP     @F  
     isNT:  
        MsgBox  CTEXT("NT Series Not Work! only 9X!")  
        JMP     \_XXX\_  
     @@:  
        call    instSEH  
  
     XH01       PROC    C  pExcept,pFrame,pContext,pDispatch    Not minimal form  
        ASSUME  ESI:PTR EXCEPTION\_RECORD,EDI:PTR CONTEXT  
        MOV     ESI,pExcept  
        MOV     EDI,pContext  
        MOV     EAX,1  
        TEST    \[ESI\].ExceptionFlags,7  
        JNZ     @@Not\_handled  
        cmp     \[esi\].ExceptionCode,STATUS\_ILLEGAL\_INSTRUCTION  
        jz      illegal\_instr  
        cmp     \[esi\].ExceptionCode,STATUS\_SINGLE\_STEP  
        JZ      BPM0\_ISOK  
        jmp     @@Not\_handled  
      BPM0\_ISOK:  
        MOV     \[EDI\].regEip,OFFSET MSGbpmOK  
        JMP     SEHexit  
  
        //Set the Dr0 bpm Global BreakPoint  
      illegal\_instr:  
        MOV     \[EDI\].ContextFlags,CONTEXT\_DEBUG\_REGISTERS or CONTEXT\_FULL  
        MOV     \[EDI\].iDr7,M\_INSTR0 or M\_GDR0 or M\_BYTE0  
        MOV     \[EDI\].iDr0,OFFSET bpm01  
        //ByPASS the INVALID INSTRS  
        ADD     \[EDI\].regEip,2             
  
      SEHexit:  
        DEC     EAX  
      @@Not\_handled:  
        ret  
     XH01       ENDP  
  
     instSEH:  
        LEA     eax,\[esp-4\]  
        XCHG    eax,fs:\[0\]  
        push    eax  
  
        NOP  
        DB 0Fh,0Bh              INVALID INSTRS ON ALL PLATFORMS ＝UD2  
  
     bpm01:                     SIMPLE ANTI DEBUGER  
        NOP  
        jmp     bpm01  
  
     \_XXX\_:  
        POP     fs:\[0\]  
        pop     EAX     
invoke ExitProcess,0  
     MSGbpmOK:  
        MsgBox  CTEXT("Hello BPM01 TEST SUC,Prepare TO EXIT")  
        JMP     \_XXX\_  
END \_StArT  
  
WINNT 下设置断点并非如此简单，由于严格的检查机制，原来的 SEH 的方法或简单的 GetThreadContext 方法已经失效。那么要我们写驱动吗？大可不必，因为 WIndows 还提供了调试 API，使用调试 API 可以为目标程序设置断点：  
  
WINNT 下设置断点并非如此简单，由于严格的检查机制，原来的 SEH 的方法或简单的 SetThreadContext 方法已经失效。那么要我们写驱动吗？大可不必，因为 WIndows 还提供了调试 API，使用调试 API 可以为目标程序设置断点。使用调试程序设置断点仍然使用 GetThreadContext 和 SetThreadContext 两个 API。  
  
在看下一个例子之前，还是让我们看看著名的 IczeLion 的教程中利用调试 API 检测程序执行了多少指令这一例子，这个例子在 9X 下运行正常，但在 2K/Xp 下却显示程序初始化失败。究其原因在于在 2K/Xp 下程序收到第一个断点异常时被调试程序的主线程内容并未准备好，是无效的！很奇怪但确实如此，这时根本不能通过 CONTEXT 设置断点或单步等。解决方案之一是 Elicz 提出来的，就是先在调试程序收到的第一个断点中断中给 NtContinue 设一 bpm 断点，截获其第一个参数，这个参数就是指向即将初始化为主线程有效 CONTEXT 内容的指针，通过在其内设置一个 bpm 断点，就可以中断在我们想要中断的任何位置。  
下面是改造过的计算程序执行指令条数的例子，测试结果显示，一个很简单的程序，  
include c:\\hd\\hhd.h  
.DATA?  
hInstance dd ?  
;;-----------------------------------------  
.CODE  
\_StArT:  
        nop  
        nop  
mov hInstance,$invoke(GetModuleHandle,0)  
MsgBox  CTEXT("Hello World")  
  
invoke ExitProcess,0  
END \_StArT  
  
  
执行的内核代码+程序代码 19196764 条，如果只想获得程序执行的程序代码，还是有办法的，自己去想啦。  
本文不是讲述调试 API 的使用，使用见 IczeLion 的教程：  
;  
include c:\\hd\\hhd.h  
include c:\\hd\\drx.h  
  
BREAK\_RVA EQU   401000H         断点。。。。  
;~~~~~~~~~~~~~~~~~~~  
.DATA  
dwSScnt dd 0  
sinstr  dd 0          
pi      PROCESS\_INFORMATION <>  
sif     STARTUPINFO   
Dev     DEBUG\_EVENT <>  
Rs      CONTEXT   
buf     db 256 dup(?)  
dwBuf   dd 0  
dwBytes dd 0  
;;-----------------------------------------  
.CODE  
WipeContextBPdr0        Proc  
        invoke GetThreadContext,pi.hThread,addr Rs  
        mov        Rs.iDr0,0  
        mov        Rs.iDr7,0  
        invoke SetThreadContext,pi.hThread,addr Rs  
        ret  
WipeContextBPdr0        Endp  
;-----------------------------------------  
  
\_StArT:  
        SLDT    CX  
        JCXZ    @F  
        MsgBox  CTEXT("9x Not supported")  
        JMP     pExit  
        @@:  
        invoke GetStartupInfo,addr sif  
        SUB     EAX,EAX  
invoke CreateProcess,0,CTEXT("target01.exe"),EAX,EAX,EAX,\\  
                DEBUG\_PROCESS or DEBUG\_ONLY\_THIS\_PROCESS,EAX,EAX,addr sif,addr pi  
        JEAXZ   crp\_fail  
  
            //some basic text macro   
            //why so many ppl use long names,aren't they tired ?  
            wmDevent    EQU Dev.dwDebugEventCode  
            excCode     EQU Dev.u.Exception.pExceptionRecord.ExceptionCode  
            excAddr     EQU Dev.u.Exception.pExceptionRecord.ExceptionAddress  
            pDllName    EQU Dev.u.LoadDll.lpImageName  
            lpBase      EQU Dev.u.LoadDll.lpBaseOfDll  
            //  
  
        .WHILE TRUE  
            invoke WaitForDebugEvent,addr Dev,INFINITE  
            mov           
            .IF wmDevent==EXIT\_PROCESS\_DEBUG\_EVENT  
                MsgBox CTEXT("target Exit...")  
                .break  
            .ELSEIF wmDevent==LOAD\_DLL\_DEBUG\_EVENT  
                //to see how many DLLS were LOADED  
                invoke wsprintf,addr buf,CTEXT("DLL BASE =: %08X"),lpBase                                         
                invoke MessageBox,0,addr buf,0,0  
                JMP  DBG\_con  
            .ELSEIF wmDevent==EXCEPTION\_DEBUG\_EVENT  
  
                .if  excCode==STATUS\_BREAKPOINT  
                     invoke GetThreadContext,pi.hThread,addr Rs  
                     invoke GetProcAddress,$invoke(GetModuleHandle,CTEXT("NTDLL.DLL")),CTEXT("NtContinue")  
                     JEAXZ      end\_debug  
                     mov        Rs.iDr0,EAX             设置 NtContinue 的断点  
                     mov        Rs.iDr7,M\_LDR0 or M\_INSTR0  
  
                     invoke SetThreadContext,pi.hThread,addr Rs  
                     JMP  DBG\_con  
                .elseif excCode==STATUS\_SINGLE\_STEP  
                     INC        dwSScnt  
                     .if        dwSScnt==1  
  
                        //清除 NtContinue 的 bpm 断点                           
                         invoke WipeContextBPdr0  
                         MOV    EDX,Rs.regEsp  
                         ADD    EDX,4           //获取 NtContinue 的第一个参数 pContext  
  
                         //读入 pContext 值，因为是两个进程要用 ReadProcessMemory  
                         invoke ReadProcessMemory,pi.hProcess,EDX,addr dwBuf,sizeof DWORD,addr dwBytes  
                         //读入真正的 CONTEXT 值  
                         invoke ReadProcessMemory,pi.hProcess,dwBuf,addr Rs,sizeof CONTEXT,addr dwBytes  
                         mov    Rs.iDr0,BREAK\_RVA  
                         mov    Rs.iDr7,M\_LDR0 or M\_INSTR0  
                         invoke WriteProcessMemory,pi.hProcess,dwBuf,addr Rs,sizeof CONTEXT,addr dwBytes  
                         JMP    DBG\_con  
                     .elseif    dwSScnt==2  
                         inc    sinstr  
                         invoke WipeContextBPdr0        //清除断点  
                         invoke wsprintf,addr buf,CTEXT("Break Address: %08X"),excAddr  
                         MsgBox addr buf  
                     .endif  
  
                     comment $  
                     //ICZELION's Example，uncomment this  
                     inc        sinstr  
                     invoke GetThreadContext,pi.hThread,addr Rs  
                     invoke wsprintf,addr buf,CTEXT("XINSTR ADDR %08X"),Rs.regEip  
                     MsgBox     addr buf  
                     or         Rs.regFlag,100h  
                     invoke SetThreadContext,pi.hThread,offset Rs  
                     $  
  
                     JMP  DBG\_con  
                .endif       
  
            .ENDIF  
          
      DBG\_no:     //continue Debug events without handling  
            invoke ContinueDebugEvent,Dev.dwProcessId,Dev.dwThreadId,DBG\_EXCEPTION\_NOT\_HANDLED  
            .continue  
      DBG\_con:  
            invoke ContinueDebugEvent,Dev.dwProcessId,Dev.dwThreadId,DBG\_CONTINUE  
        .ENDW  
          
        invoke wsprintf,addr buf,CTEXT("total : %08d Instructions"),sinstr  
        MsgBox  addr buf  
    end\_debug:  
        invoke CloseHandle,pi.hProcess  
        invoke CloseHandle,pi.hThread  
    pExit:  
invoke ExitProcess,0  
    crp\_fail:  
        MsgBox  CTEXT("Create Process Failed!")  
        jmp     pExit  
END \_StArT  
  
只要能使用 Drx 的断点功能就可以配合 SEH、调试 API 进行一些反跟踪等，具体怎么用，取决于你自己了。  
  
〔完〕  
\=================================================================================================  
一些常量定义（FOR MASM）  
; for DR6  
M\_HIT0                         EQU 1  
M\_HIT1                         EQU 2  
M\_HIT2                         EQU 4  
M\_HIT3                         EQU 8  
M\_BD                         EQU 2000H         DRX access  
M\_BS                         EQU 4000H         single step  
M\_BT                         EQU 8000H         task switch  
;-----------------------------------------  
  
; for DR7  
M\_LDR0                         EQU 01           Li mask for Dr7  
M\_LDR1                         EQU M\_LDR0    SHL 02  
M\_LDR2                         EQU M\_LDR1    SHL 02  
M\_LDR3                         EQU M\_LDR2    SHL 02  
M\_LDRALL EQU M\_LDR0 or M\_LDR1 or M\_LDR2 or M\_LDR3  
  
M\_GDR0                         EQU 02           Gi mask for Dr7  
M\_GDR1                         EQU M\_GDR0    SHL 02  
M\_GDR2                         EQU M\_GDR1    SHL 02  
M\_GDR3                         EQU M\_GDR2    SHL 02  
M\_GDRALL EQU M\_GDR0 or M\_GDR1 or M\_GDR2 or M\_GDR3  
  
;-----------------------------------------  
  
M\_LE                         EQU 01      SHL 08   局部断点精确相符  
M\_GE                         EQU M\_LE     SHL 01   全局断点精确相符  
  
;DRX access  
M\_GD                         EQU M\_BD              drx 保护位置一即使在 ring0 也产生 int 1  
  
;instruction fetch  
M\_INSTR0                        EQU 00  
M\_INSTR1                        EQU 00  
M\_INSTR2                        EQU 00  
M\_INSTR3                        EQU 00  
  
;memory write  
M\_WRITE0                        EQU 01 SHL 16  
M\_WRITE1                        EQU M\_WRITE0 SHL 04  
M\_WRITE2                        EQU M\_WRITE1 SHL 04  
M\_WRITE3                        EQU M\_WRITE2 SHL 04  
  
;port watches: 586+ only and CR4 bit DE (=04  bit 3 in termiology) must be 1  
;M\_DE EQU 04  
M\_DE EQU 08                  
M\_PORT0                         EQU 02 SHL 16  
M\_PORT1                         EQU M\_PORT0  SHL 04  
M\_PORT2                         EQU M\_PORT1  SHL 04  
M\_PORT3                         EQU M\_PORT2  SHL 04  
  
;memory access Read or Write!  
M\_RW0                         EQU 03 SHL 16  
M\_RW1                         EQU M\_RW0    SHL 04  
M\_RW2                         EQU M\_RW1    SHL 04  
M\_RW3                         EQU M\_RW2    SHL 04  
M\_RWALL EQU M\_RW0 or M\_RW1 or M\_RW2 or M\_RW3  
  
;lengths  
M\_BYTE0                         EQU 00           use byte for instruction by default always  
M\_BYTE1                         EQU 00            for data BreakPoints,these exist  
M\_BYTE2                         EQU 00  
M\_BYTE3                         EQU 00  
  
M\_WORD0                         EQU 01 SHL 18  
M\_WORD1                         EQU M\_WORD0  SHL 04  
M\_WORD2                         EQU M\_WORD1  SHL 04  
M\_WORD3                         EQU M\_WORD2  SHL 04  
  
M\_DWORD0                        EQU 03 SHL 18  
M\_DWORD1                        EQU M\_DWORD0 SHL 04  
M\_DWORD2                        EQU M\_DWORD1 SHL 04  
M\_DWORD3                        EQU M\_DWORD2 SHL 04  
\=============================================================================================
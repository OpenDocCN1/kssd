

# 如何分析堆栈出错的 dmp 文件

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-261.htm`](https://www.pediy.com/kssd/pediy09/pediy09-261.htm)

> 如何分析堆栈出错的 dmp 文件  
>   
>   
> 分析程序出错生成的 dmp 文件是事后分析的主要工作。第一步往往都是使用 WinDbg 自带的 !analyze -v 命令先进行初步分析，得到出错地址和出错堆栈后再进行详细分析。  
>   
> 本文介绍一个方法，当 !analyze -v 不好使的时候应该怎么得到出错地址和出错堆栈。  
>   
>     int sum(int x, int y)  
>     {  
>         \_\_asm mov ebp, 0  
>   
>         return (x + y);  
>     }  
>   
>     int sumstub(int x, int y)  
>     {  
>         int  tmp = 0;  
>   
>         printf("enter fun() ...\\n");  
>   
>         tmp = sum(x, y);  
>   
>         printf("leave fun() ...\\n");  
>   
>         return tmp;  
>     }  
>   
>     int main(int argc, char\* argv\[\])  
>     {  
>         printf("enter main() ...\\n");  
>   
>         printf("sum = %d\\n", sumstub(0x1234, 0x5678));  
>   
>         printf("leave main() ...\\n");  
>   
>         return 0;  
>     }  
>   
> 示例程序比较简单，在 sum 函数里面会把 ebp 清零，下面取 x 或者 y 的值时就会出错。  
>   
> 用 WinDbg 打开出错得到的 dmp 文件，先用 !analyze -v 分析，结果如下：  
>   
>     0:000> !analyze -v  
>     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>     \*                                                                             \*  
>     \*                        Exception Analysis                                   \*  
>     \*                                                                             \*  
>     \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
>   
>     \*\*\* WARNING: Unable to verify checksum for Dump01.exe  
>     \*\*\* ERROR: Symbol file could not be found.  Defaulted to export symbols for lpk.dll -   
>     \*\*\* ERROR: Symbol file could not be found.  Defaulted to export symbols for Sysfer.dll -   
>     \*\*\* ERROR: Symbol file could not be found.  Defaulted to export symbols for usp10.dll -   
>     \*\*\* ERROR: Symbol file could not be found.  Defaulted to export symbols for imm32.dll -   
>     \*\*\* ERROR: Symbol file could not be found.  Defaulted to export symbols for apphelp.dll -   
>     \*\*\* ERROR: Symbol file could not be found.  Defaulted to export symbols for version.dll -   
>     \*\*\* ERROR: Symbol file could not be found.  Defaulted to export symbols for advapi32.dll -   
>     \*\*\* ERROR: Symbol file could not be found.  Defaulted to export symbols for shlwapi.dll -   
>   
>     FAULTING\_IP:   
>     +0  
>     00000000 ??              ???  
>   
>     EXCEPTION\_RECORD:  ffffffff -- (.exr 0xffffffffffffffff)  
>     ExceptionAddress: 00000000  
>        ExceptionCode: 80000007 (Wake debugger)  
>       ExceptionFlags: 00000000  
>     NumberParameters: 0  
>   
>     BUGCHECK\_STR:  80000007  
>   
>     PROCESS\_NAME:  Dump01.exe  
>   
>     ERROR\_CODE: (NTSTATUS) 0x80000007 - {  
>   
>     NTGLOBALFLAG:  0  
>   
>     APPLICATION\_VERIFIER\_FLAGS:  0  
>   
>     DERIVED\_WAIT\_CHAIN:    
>   
>     Dl Eid Cid     WaitType  
>     -- --- ------- --------------------------  
>        0   62c.928 Unknown                  
>   
>     WAIT\_CHAIN\_COMMAND:  ~0s;k;;  
>   
>     BLOCKING\_THREAD:  00000928  
>   
>     DEFAULT\_BUCKET\_ID:  APPLICATION\_HANG\_HungIn\_ExceptionHandler  
>   
>     PRIMARY\_PROBLEM\_CLASS:  APPLICATION\_HANG\_HungIn\_ExceptionHandler  
>   
>     LAST\_CONTROL\_TRANSFER:  from 7c92e9ab to 7c92eb94  
>   
>     FAULTING\_THREAD:  00000928  
>   
>     STACK\_TEXT:    
>     0012f3b8 7c92e9ab 7c86372c 00000002 0012f53c ntdll!KiFastSystemCallRet  
>     0012f3bc 7c86372c 00000002 0012f53c 00000001 ntdll!ZwWaitForMultipleObjects+0xc  
>     0012fb38 00401dda 0012fb74 0012ffb0 0012ffc0 kernel32!UnhandledExceptionFilter+0x8e4  
>     0012fb48 00401198 c0000005 0012fb74 0040261b Dump01!\_XcptFilter+0x13e  
>     0012ffc0 7c816fd7 011dd65c 011dd664 7ffd6000 Dump01!mainCRTStartup+0xd1  
>     0012fff0 00000000 004010c7 00000000 00000000 kernel32!BaseProcessStart+0x23  
>   
>   
>     FOLLOWUP\_IP:   
>     Dump01!\_XcptFilter+13e  
>     00401dda 5b              pop     ebx  
>   
>     SYMBOL\_STACK\_INDEX:  3  
>   
>     SYMBOL\_NAME:  Dump01!\_XcptFilter+13e  
>   
>     FOLLOWUP\_NAME:  MachineOwner  
>   
>     MODULE\_NAME: Dump01  
>   
>     IMAGE\_NAME:  Dump01.exe  
>   
>     DEBUG\_FLR\_IMAGE\_TIMESTAMP:  46de4ed1  
>   
>     STACK\_COMMAND:  ~0s ; kb  
>   
>     FAILURE\_BUCKET\_ID:  80000007\_Dump01!\_XcptFilter+13e  
>   
>     BUCKET\_ID:  80000007\_Dump01!\_XcptFilter+13e  
>   
>     Followup: MachineOwner  
>     ---------  
>   
> 分析得到的出错地址为 0，堆栈也在内核里面。很明显这次 !analyze -v 命令出问题了，需要手动分析才能得到想要的信息。  
>   
>     0:000> ~\*kv  
>   
>     .  0  Id: 62c.928 Suspend: 1 Teb: 7ffdf000 Unfrozen  
>     ChildEBP RetAddr  Args to Child                
>     0012f3b8 7c92e9ab 7c86372c 00000002 0012f53c ntdll!KiFastSystemCallRet (FPO: \[0,0,0\])  
>     0012f3bc 7c86372c 00000002 0012f53c 00000001 ntdll!ZwWaitForMultipleObjects+0xc (FPO: \[5,0,0\])  
>     0012fb38 00401dda 0012fb74 0012ffb0 0012ffc0 kernel32!UnhandledExceptionFilter+0x8e4 (FPO: \[Non-Fpo\])  
>     0012fb48 00401198 c0000005 0012fb74 0040261b Dump01!\_XcptFilter+0x13e  
>     0012ffc0 7c816fd7 011dd65c 011dd664 7ffd6000 Dump01!mainCRTStartup+0xd1  
>     0012fff0 00000000 004010c7 00000000 00000000 kernel32!BaseProcessStart+0x23 (FPO: \[Non-Fpo\])  
>   
>     0:000> !teb  
>     TEB at 7ffdf000  
>         ExceptionList:        0012fb28  
>         StackBase:            00130000  
>         StackLimit:           0012a000  
>         SubSystemTib:         00000000  
>         FiberData:            00001e00  
>         ArbitraryUserPointer: 00000000  
>         Self:                 7ffdf000  
>         EnvironmentPointer:   00000000  
>         ClientId:             0000062c . 00000928  
>         RpcHandle:            00000000  
>         Tls Storage:          00000000  
>         PEB Address:          7ffd6000  
>         LastErrorValue:       0  
>         LastStatusValue:      103  
>         Count Owned Locks:    0  
>         HardErrorMode:        0  
>   
> 先查看所有线程的堆栈信息，然后找出比较像出了问题的线程。本次示例只有一个线程，所以肯定是该线程出错。然后显示出错线程的 TEB 信息。  
>   
>     0:000> dps 0x0012a000 0x00130000  
>   
> 根据堆栈的位置和大小，显示堆栈的所有内容。  
>   
> 根据 Windows 异常处理流程可知，所有没被调试器处理的异常最终都会转到 ntdll!KiUserExceptionDispatcher 函数查找 SEH 异常处理例程来处理异常。所以在显示的堆栈信息中查找 ntdll!KiUserExceptionDispatcher 字符串。  
>   
>     0012fc50  00000000  
>     0012fc54  7c92eafa ntdll!KiUserExceptionDispatcher+0xe  
>     0012fc58  00000000  
>     0012fc5c  0012fc84  
>   
> 再根据 KiUserExceptionDispatcher 函数的原型得到本次异常发生时保存的 CONTEXT 结构信息。  
>   
>     ; VOID  
>     ; KiUserExceptionDispatcher (  
>     ;    IN PEXCEPTION\_RECORD ExceptionRecord,  
>     ;    IN PCONTEXT ContextRecord  
>     ;    )  
>   
> 第二个参数指向 CONTEXT 结构，利用 WinDbg 的 .cxr 命令显示/切换 CONTEXT 结构。  
>   
>     0:000> .cxr 0x0012fc84  
>     eax=00005678 ebx=7ffd6000 ecx=00001234 edx=7c92eb94 esi=011dd664 edi=011dd65c  
>     eip=0040100b esp=0012ff50 ebp=00000000 iopl=0         nv up ei pl nz na pe nc  
>     cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00010206  
>     Dump01!sum+0xb:  
>     0040100b 8b4508          mov     eax,dword ptr \[ebp+8\] ss:0023:00000008=????????  
>   
>     0:000> kv  
>       \*\*\* Stack trace for last set context - .thread/.cxr resets it  
>     ChildEBP RetAddr  Args to Child                
>     00000000 00000000 00000000 00000000 00000000 Dump01!sum+0xb (CONV: cdecl) \[E:\\Works\\Dump01\\Dump01.cpp @ 10\]  
>   
> 现在已经找到出错地址为 0x0040100b，下面恢复正确的出错堆栈。  
>   
>     0:000> ?? sizeof(ntdll!\_CONTEXT)  
>     unsigned int 0x2cc  
>   
>     0:000> ? 0x0012fc84 + 0x2cc  
>     Evaluate expression: 1245008 = 0012ff50  
>   
> 计算可知，出错前的堆栈位置在 0x0012ff50 处。  
>   
>     0:000> ub 0x0040100b L 6  
>     Dump01!sum \[E:\\Works\\Dump01\\Dump01.cpp @ 7\]:  
>     00401000 55              push    ebp  
>     00401001 8bec            mov     ebp,esp  
>     00401003 53              push    ebx  
>     00401004 56              push    esi  
>     00401005 57              push    edi  
>     00401006 bd00000000      mov     ebp,0  
>   
>     0:000> dps 0x0012ff50 L 0x10  
>     0012ff50  011dd65c  
>     0012ff54  011dd664  
>     0012ff58  7ffd6000  
>     0012ff5c  0012ff70  
>     0012ff60  0040103b Dump01!sumstub+0x25 \[E:\\Works\\Dump01\\Dump01.cpp @ 19\]  
>     0012ff64  00001234  
>     0012ff68  00005678  
>     0012ff6c  00000000  
>     0012ff70  0012ff80  
>     0012ff74  00401074 Dump01!main+0x1f \[E:\\Works\\Dump01\\Dump01.cpp @ 30\]  
>     0012ff78  00001234  
>     0012ff7c  00005678  
>     0012ff80  0012ffc0  
>     0012ff84  0040117b Dump01!mainCRTStartup+0xb4  
>     0012ff88  00000001  
>     0012ff8c  00520eb0  
>   
>     0:000> r  
>     Last set context:  
>     eax=00005678 ebx=7ffd6000 ecx=00001234 edx=7c92eb94 esi=011dd664 edi=011dd65c  
>     eip=0040100b esp=0012ff50 ebp=00000000 iopl=0         nv up ei pl nz na pe nc  
>     cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00010206  
>     Dump01!sum+0xb:  
>     0040100b 8b4508          mov     eax,dword ptr \[ebp+8\] ss:0023:00000008=????????  
>   
> 反汇编出错地址前的几条指令，可以知道出错原因是 0x00401006 处的指令导致 ebp 被赋零，所以接下来取参数的指令出错。再根据堆栈信息，出错前往堆栈中压入了 ebx/esi/edi 几个寄存器的值，对比 0x0012ff50 处的堆栈，可知 0x0012ff50 正好是程序出错前的堆栈地址。同时还可以得到保存在堆栈上的 ebp 的值，从而得到正确的出错堆栈。  
>   
>     0:000> kv L = 0x0012ff5c  
>     ChildEBP RetAddr  Args to Child                
>     0012ff5c 0040103b 00001234 00005678 00000000 Dump01!sum+0xb (CONV: cdecl)  
>     0012ff70 00401074 00001234 00005678 0012ffc0 Dump01!sumstub+0x25 (CONV: cdecl)  
>     0012ff80 0040117b 00000001 00520eb0 00520e20 Dump01!main+0x1f (CONV: cdecl)  
>     0012ffc0 7c816fd7 011dd65c 011dd664 7ffd6000 Dump01!mainCRTStartup+0xb4  
>     0012fff0 00000000 004010c7 00000000 00000000 kernel32!BaseProcessStart+0x23 (FPO: \[Non-Fpo\])  
>   
> 从这个堆栈来看，起始地址从 kernel32!BaseProcessStart 开始，结束地址也正好在出错地址处，应该是正确的出错堆栈。  
>   
> \==========  
> 小喂  
> 2007.09.05 

> dmp 文件是怎么产生的?存在哪? 

> 可以把即时调试器设置为 ntsd 或者 windbg，程序出错后就可以用 .dump 命令生成 dmp 文件。  
>   
> 下面是我的设置：  
>   
> Windows Registry Editor Version 5.00  
>   
> \[HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\AeDebug\]  
> "Auto"="0"  
> "Debugger"="\\"D:\\\\WinDBG\\\\ntsd.exe\\" -p %ld -e %ld -g -noio -c \\".dump /ma /u d:\\\\dbgdmp\\\\dmp.dmp; q"  
> "UserDebuggerHotKey"=dword:00000000 

 4dd466d0e1e0cc123afbe504da2462ca
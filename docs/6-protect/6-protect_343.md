

# 反调试技巧总结-原理和实现(1)(2)(3)(4)(5)(6)......

> 原文：[`www.pediy.com/kssd/pediy10/70470.html`](https://www.pediy.com/kssd/pediy10/70470.html)

反调试技巧总结-原理和实现  
\-------------------------------------------------------------------------------------------------------  
2008.8.7  shellwolf  
一、 前言  
    前段学习反调试和 vc，写了 antidebug-tester，经常会收到 message 希望交流或索要实现代码，我都没有回复。其实代码已经在编程版提供了 1 个版本，另其多是 vc 内嵌 asm 写的，对 cracker 而言，只要反下就知道了。我想代码其实意义不是很大，重要的是理解和运用。  
    做个简单的总结，说明下实现原理和实现方法。也算回复了那些给我发 Message 的朋友。  
  
    部分代码和参考资料来源：  
1、<<脱壳的艺术>> hawking  
2、<<windows anti-debugger reference>> Angeljyt  
3、http://bbs.pediy.com   
4、<<软件加密技术内幕>> 看雪学院  
5、<<ANTI-UNPACKER TRICKS>> Peter Ferrie  
  
我将反调试技巧按行为分为两大类，一类为检测，另一类为攻击，每类中按操作对象又分了五个小类：  
1、 通用调试器     包括所有调试器的通用检测方法  
2、 特定调试器     包括 OD、IDA 等调试器，也包括相关插件，也包括虚拟环境  
3、 断点           包括内存断点、普通断点、硬件断点检测  
4、 单步和跟踪     主要针对单步跟踪调试  
5、 补丁           包括文件补丁和内存补丁  
反调试函数前缀  
              检测        攻击  
通用调试器     FD\_        AD\_  
特定调试器     FS\_        AS\_  
断点           FB\_        AB\_  
单步和跟踪     FT\_        AT\_  
补丁           FP\_        AP\_  
  
声明：  
1、本文多数都是摘录和翻译，我只是重新组合并翻译，不会有人告侵权吧。里面多是按自己的理解来说明，可能有理解错误，或有更好的实现方法，希望大家帮忙指出错误。  
2、我并没有总结完全，上面的部分分类目前还只有很少的函数甚至空白，等待大家和我一起来完善和补充。我坚信如果有扎实的基础知识，丰富的想像力，灵活的运用，就会创造出更多的属于自己的反调试。而最强的反调试，通常都是自己创造的，而不是来自别人的代码。  
  
二、 查找-通用调试器(FD\_)  
函数列表如下，后面会依次说明，需事先说明的是，这些反调试手段多数已家喻户晓，目前有效的不多，多数已可以通过 OD 的插件顺利通过，如果你想验证它们的有效性，请关闭 OD 的所有反反调试插件：  
bool FD\_IsDebuggerPresent();  
bool FD\_PEB\_BeingDebuggedFlag();  
bool FD\_PEB\_NtGlobalFlags();  
bool FD\_Heap\_HeapFlags();  
bool FD\_Heap\_ForceFlags();  
bool FD\_Heap\_Tail();  
bool FD\_CheckRemoteDebuggerPresent();  
bool FD\_NtQueryInfoProc\_DbgPort();  
bool FD\_NtQueryInfoProc\_DbgObjHandle();  
bool FD\_NtQueryInfoProc\_DbgFlags();  
bool FD\_NtQueryInfoProc\_SysKrlDbgInfo();  
bool FD\_SeDebugPrivilege();  
bool FD\_Parent\_Process();  
bool FD\_DebugObject\_NtQueryObject();  
bool FD\_Find\_Debugger\_Window();  
bool FD\_Find\_Debugger\_Process();  
bool FD\_Find\_Device\_Driver();  
bool FD\_Exception\_Closehandle();  
bool FD\_Exception\_Int3();  
bool FD\_Exception\_Popf();  
bool FD\_OutputDebugString();  
bool FD\_TEB\_check\_in\_Vista();  
bool FD\_check\_StartupInfo();  
bool FD\_Parent\_Process1();  
bool FD\_Exception\_Instruction\_count();  
bool FD\_INT\_2d();  
  
2.1 FD\_IsDebuggerPresent()  
对调试器来说，IsDebuggerPresent 是臭名昭著的恶意函数。不多说了，它是个检测调试的 api 函数。实现更简单，只要调用 IsDebuggerPresent 就可以了。在调用它之前，可以加如下代码，以用来检测是否在函数头有普通断点，或是否被钩挂。  
  //check softbreak  
  if(\*(BYTE\*)Func\_addr==0xcc)  
    return true;  
  //check hook  
  if(\*(BYTE\*)Func\_addr!=0x64)  
    return true;  
2.2 FD\_PEB\_BeingDebuggedFlag  
我们知道，如果程序处于调试器中，那么在 PEB 结构中有个 beingDegug 标志会被设置，直接读取它就可判断是否在调试器中。实际上 IsDebuggerPresent 就是这么干的。  
  \_\_asm  
  {  
    mov eax, fs:\[30h\] ;EAX =  TEB.ProcessEnvironmentBlock  
    inc eax  
    inc eax  
    mov eax, \[eax\]  
    and eax,0x000000ff  ;AL  =  PEB.BeingDebugged  
    test eax, eax  
    jne rt\_label  
    jmp rf\_label  
  }  
  
2.3 FD\_PEB\_NtGlobalFlags  
PEB 中还有其它 FLAG 表明了调试器的存在，如 NtGlobalFlags。它位于 PEB 环境中偏移为 0x68 的位置，默认情况下该值为 0，在 win2k 和其后的 windows 平台下，如果在调试中，它会被设置为一个特定的值。使用该标志来判断是否被调试并不可靠（如在 winnt 中），但这种方法却也很常用。这个标志由下面几个标志组成：  
\*\*\*\_HEAP\_ENABLE\_TAIL\_CHECK (0x10)  
\*\*\*\_HEAP\_ENABLE\_FREE\_CHECK (0x20)  
\*\*\*\_HEAP\_VALIDATE\_PARAMETERS (0x40)  
检测 NtGlobalFlags 的方法如下，这个方法在 ExeCryptor 中使用过。  
\_\_asm  
  {  
    mov eax, fs:\[30h\]  
    mov eax, \[eax+68h\]  
    and eax, 0x70  
    test eax, eax  
    jne rt\_label  
    jmp rf\_label  
  }  
  
2.4 FD\_Heap\_HeapFlags()  
同样，调试器也会在堆中留下痕迹，你可以使用 kernel32\_GetProcessHeap()函数，如果你不希望使用 api 函数（以免暴露），则可以直接在 PEB 中寻找。同样的，使用 HeapFlags 和后面提到的 ForceFlags 来检测调试器也不是非常可靠，但却很常用。  
这个域由一组标志组成，正常情况下，该值应为 2。  
  \_\_asm  
  {  
    mov eax, fs:\[30h\]  
    mov eax, \[eax+18h\] ;PEB.ProcessHeap  
    mov eax, \[eax+0ch\] ;PEB.ProcessHeap.Flags  
    cmp eax, 2  
    jne rt\_label  
    jmp rf\_label  
  }  
  
2.5 FD\_Heap\_ForceFlags  
进程堆里另外一个标志，ForceFlags，它也由一组标志组成，正常情况下，该值应为 0。  
  \_\_asm  
  {  
    mov eax, fs:\[30h\]  
    mov eax, \[eax+18h\] ;PEB.ProcessHeap  
    mov eax, \[eax+10h\] ;PEB.ProcessHeap.ForceFlags  
    test eax, eax  
    jne rt\_label  
    jmp rf\_label  
  }  
  
2.6 FD\_Heap\_Tail  
如果处于调试中，堆尾部也会留下痕迹。标志 HEAP\_TAIL\_CHECKING\_ENABLED 将会在分配的堆块尾部生成两个 0xABABABAB。如果需要额外的字节来填充堆尾，HEAP\_FREE\_CHECKING\_ENABLED 标志则会生成 0xFEEEFEEE。  
  
据说 Themida 使用过这个反调试  
  \_\_asm  
  {  
    mov eax, buff  
    ;get unused\_bytes  
    movzx ecx, byte ptr \[eax-2\]  
    movzx edx, word ptr \[eax-8\] ;size  
    sub eax, ecx  
    lea edi, \[edx\*8+eax\]  
    mov al, 0abh  
    mov cl, 8  
    repe sca\*\*  
    je rt\_label  
    jmp rf\_label  
  }  
  
2.7 FD\_CheckRemoteDebuggerPresent  
CheckRemoteDebuggerPresent 是另一个检测调试的 api，只是可惜它似乎只能在 winxp sp1 版本以后使用。它主要是用来查询一个在 winnt 时就有的一个数值，其内部会调用 NtQueryInformationProcess()，我是这样实现的：  
  FARPROC Func\_addr ;  
  HMODULE hModule = GetModuleHandle("kernel32.dll");  
  if (hModule==INVALID\_HANDLE\_VALUE)  
    return false;  
  (FARPROC&) Func\_addr =GetProcAddress(hModule, "CheckRemoteDebuggerPresent");  
  if (Func\_addr != NULL)   
  {  
    \_\_asm   
    {  
      push  eax;  
      push  esp;  
      push  0xffffffff;  
      call  Func\_addr;  
      test  eax,eax;  
      je    rf\_label;  
      pop    eax;  
      test  eax,eax  
      je    rf\_label;  
      jmp    rt\_label;  
    }  
  }  
  
2.8 FD\_NtQueryInfoProc\_DbgPort  
使用 ntdll\_NtQueryInformationProcess()来查询 ProcessDebugPort 可以用来检测反调试。如果进程被调试，其返回值应为 0xffffffff。  
下面的代码应该是从 pediy 里 copy 过来的，时间太长，不记得是哪位兄弟的代码了。  
  HMODULE hModule = GetModuleHandle("ntdll.dll");   
  ZW\_QUERY\_INFORMATION\_PROCESS ZwQueryInformationProcess;   
    ZwQueryInformationProcess = (ZW\_QUERY\_INFORMATION\_PROCESS)GetProcAddress(hModule, "ZwQueryInformationProcess");   
    if (ZwQueryInformationProcess == NULL)   
    return false;  
  PROCESS\_DEBUG\_PORT\_INFO ProcessInfo;   
  if (STATUS\_SUCCESS != ZwQueryInformationProcess(GetCurrentProcess( ), ProcessDebugPort, &ProcessInfo, sizeof(ProcessInfo), NULL))   
    return false;  
  else   
    if(ProcessInfo.DebugPort)  
      return true;  
    else  
      return false;  
        
2.9 FD\_NtQueryInfoProc\_DbgObjHandle  
  在 winxp 中引入了"debug object".当一个调试活动开始，一个"debug object"被创建，同也相应产生了一个句柄。使用为公开的 ProcessDebugObjectHandle 类，可以查询这个句柄的数值。  
  代码可能还是从 pediy 里复制的，不记得了。  
  HMODULE hModule = GetModuleHandle("ntdll.dll");   
  ZW\_QUERY\_INFORMATION\_PROCESS ZwQueryInformationProcess;   
    ZwQueryInformationProcess = (ZW\_QUERY\_INFORMATION\_PROCESS)GetProcAddress(hModule, "ZwQueryInformationProcess");   
    if (ZwQueryInformationProcess == NULL)   
    return false;  
  \_PROCESS\_DEBUG\_OBJECTHANDLE\_INFO ProcessInfo;   
  if (STATUS\_SUCCESS != ZwQueryInformationProcess(GetCurrentProcess( ), (PROCESS\_INFO\_CLASS)0x0000001e, &ProcessInfo, sizeof(ProcessInfo), NULL))   
    return false;  
  else   
    if(ProcessInfo.ObjectHandle)  
      return true;  
    else  
      return false;  
  
2.10 FD\_NtQueryInfoProc\_DbgFlags();  
同样的未公开的 ProcessDebugFlags 类，当调试器存在时，它会返回 false。  
  HMODULE hModule = GetModuleHandle("ntdll.dll");   
  ZW\_QUERY\_INFORMATION\_PROCESS ZwQueryInformationProcess;   
    ZwQueryInformationProcess = (ZW\_QUERY\_INFORMATION\_PROCESS)GetProcAddress(hModule, "ZwQueryInformationProcess");   
    if (ZwQueryInformationProcess == NULL)   
    return false;  
  \_PROCESS\_DEBUG\_FLAGS\_INFO ProcessInfo;   
  if (STATUS\_SUCCESS != ZwQueryInformationProcess(GetCurrentProcess( ), (PROCESS\_INFO\_CLASS)0x0000001f, &ProcessInfo, sizeof(ProcessInfo), NULL))   
    return false;  
  else   
    if(ProcessInfo.Debugflags)  
      return false;  
    else  
      return true;  
  
2.11 FD\_NtQueryInfoProc\_SysKrlDbgInfo()  
这个方法估计对大家用处不大，SystemKernelDebuggerInformation 类同样可以用来识别调试器，只是可惜在 windows 下无效，据称可以用在 reactOS 中。  
   HMODULE hModule = GetModuleHandle("ntdll.dll");   
    ZW\_QUERY\_SYSTEM\_INFORMATION ZwQuerySystemInformation;   
    ZwQuerySystemInformation = (ZW\_QUERY\_SYSTEM\_INFORMATION)GetProcAddress(hModule, "ZwQuerySystemInformation");   
    if (ZwQuerySystemInformation == NULL)   
        return false;  
    SYSTEM\_KERNEL\_DEBUGGER\_INFORMATION Info;   
    if (STATUS\_SUCCESS == ZwQuerySystemInformation(SystemKernelDebuggerInformation, &Info, sizeof(Info), NULL))   
    {   
        if (Info.DebuggerEnabled)   
        {   
            if (Info.DebuggerNotPresent)   
                return false;  
            else   
                return true;  
        }   
        else   
            return false;  
    }   
    else   
       return true;  
  
  
  
2.12 FD\_SeDebugPrivilege()  
  当一个进程获得 SeDebugPrivilege，它就获得了对 CSRSS.EXE 的完全控制，这种特权也会被子进程继承，也就是说一个被调试的程序如果获得了 CSRSS.EXE 的进程 ID,它就可以使用 openprocess 操作 CSRSS.EXE。获得其进程 ID 有很多中方法，如 Process32Next，或 NtQuerySystemInformation，在 winxp 下可以使用 CsrGetProcessId。  
    hTmp=OpenProcess(PROCESS\_ALL\_ACCESS,false,PID\_csrss);  
    if(hTmp!=NULL)  
    {  
      CloseHandle(hProcessSnap );  
      return true;  
    }  
  
2.13 FD\_Parent\_Process()  
通常我们都直接在 windows 界面下运行应用程序，这样的结果就是它的父进程为"explorer.exe"，这个反调试就是检测应用程序的父进程是否为"explorer.exe"，如不是则判定为处于调试器中，这也不是百分百可靠，因为有的时候你的程序是在命令行提示符下运行的。  
Yoda 使用了这个反调试，它使用 Process32Next 检测父进程，目前很多插件已经通过使 Process32Next 始终返回 false 来越过这个反调试(比如 HideOD)。不过可以对代码做些简单的修正来处理这个反反调试。  
  
2.14 FD\_DebugObject\_NtQueryObject();  
  如前面所描述的，当一个调试活动开始，一个"debug object"被创建，同也相应产生了一个句柄。我们可以查询这个调试对象列表，并检查调试对象的数量，以实现调试器的检测。  
  HMODULE hModule = GetModuleHandle("ntdll.dll");   
  PNtQueryObject NtQueryObject;  
  NtQueryObject = (PNtQueryObject)GetProcAddress(hModule,"NtQueryObject");  
  
  if(NtQueryObject==NULL)  
    return false;  
  unsigned char szdbgobj\[25\]=  
  "\\x44\\x00\\x65\\x00\\x62\\x00\\x75\\x00\\x67\\x00\\x4f\\x00\\x62\\x00\\x6a\\x00\\x65\\x00\\x63\\x00\\x74\\x00\\x00\\x00";  
  unsigned char \*psz=&szdbgobj\[0\];  
  \_\_asm  
  {  
    xor    ebx,ebx;  
    push  ebx;  
    push  esp;  
    push  ebx;  
    push  ebx;  
    push  3;  
    push  ebx;  
    Call  dword ptr \[NtQueryObject\];  
    pop  edi;  
    push  4;  
    push  1000h;  
    push  edi;  
    push  ebx;  
      call  dword ptr \[VirtualAlloc\];  
    push  ebx;  
    push  edi;  
    push  eax;  
    push  3;  
    push  ebx;  
    xchg  esi,eax;  
    Call  dword ptr \[NtQueryObject\];  
    lodsd;  
    xchg  ecx,eax;  
lable1:  lodsd;  
    movzx  edx,ax;  
    lodsd;  
    xchg  esi,eax;  
    cmp    edx,16h;  
    jne    label2;  
    xchg  ecx,edx;  
    mov    edi,psz;  
    repe  cmp\*\*;  
    xchg  ecx,edx;  
    jne    label2;  
    cmp    dword ptr \[eax\],edx  
    jne    rt\_label;  
lable2:  add    esi,edx  
    and    esi,-4;  
    lodsd  
    loop  label1;  
  }  
  return false;  
rt\_label:  
  return true;  
2.15 FD\_Find\_Debugger\_Window();  
通过列举运行的应用程序的窗口，并于常用调试相关工具比对的方法，应该很常用了，就不多说了。这个也是个可以自行增加项目的函数，你可以将一些常用的调试工具归入其中，比如 OD,IDA,WindBG,SoftICE 等，你也可以添加任何你需要的，比如"Import REConstructor v1.6 FINAL (C) 2001-2003 MackT/uCF"，"Registry Monitor - Sysinternals: www.sysinternals.com"等等。  
  //ollyice  
    hWnd=CWnd::FindWindow(\_T("1212121"),NULL);  
    if (hWnd!=NULL)  
    return true;  
  //ollydbg v1.1  
    hWnd=CWnd::FindWindow(\_T("icu\_dbg"),NULL);  
    if (hWnd!=NULL)  
    return true;  
  //ollyice pe--diy  
    hWnd=CWnd::FindWindow(\_T("pe--diy"),NULL);  
    if (hWnd!=NULL)  
    return true;  
  //ollydbg ?-°?  
    hWnd=CWnd::FindWindow(\_T("ollydbg"),NULL);  
    if (hWnd!=NULL)  
    return true;  
  //ollydbg ?-°?  
    hWnd=CWnd::FindWindow(\_T("odbydyk"),NULL);  
    if (hWnd!=NULL)  
    return true;  
  //windbg  
    hWnd=CWnd::FindWindow(\_T("WinDbgFrameClass"),NULL);  
    if (hWnd!=NULL)  
    return true;  
  //dede3.50  
    hWnd=CWnd::FindWindow(\_T("TDeDeMainForm"),NULL);  
    if (hWnd!=NULL)  
    return true;  
  //IDA5.20  
    hWnd=CWnd::FindWindow(\_T("TIdaWindow"),NULL);  
    if (hWnd!=NULL)  
    return true;  
  //others  
    hWnd=CWnd::FindWindow(\_T("TESTDBG"),NULL);  
    if (hWnd!=NULL)  
    return true;  
    hWnd=CWnd::FindWindow(\_T("kk1"),NULL);  
    if (hWnd!=NULL)  
    return true;  
    hWnd=CWnd::FindWindow(\_T("Eew75"),NULL);  
    if (hWnd!=NULL)  
    return true;  
    hWnd=CWnd::FindWindow(\_T("Shadow"),NULL);  
    if (hWnd!=NULL)  
    return true;  
  //PEiD v0.94  
    hWnd=CWnd::FindWindow(NULL,"PEiD v0.94");  
    if (hWnd!=NULL)  
    return true;  
  //RegMON  
    hWnd=CWnd::FindWindow(NULL,"Registry Monitor - Sysinternals: www.sysinternals.com");  
    if (hWnd!=NULL)  
    return true;  
  //File Monitor  
    hWnd=CWnd::FindWindow(NULL,"File Monitor - Sysinternals: www.sysinternals.com");  
    if (hWnd!=NULL)  
    return true;  
  //Import Rec v1.6  
    hWnd=CWnd::FindWindow(NULL,"Import REConstructor v1.6 FINAL (C) 2001-2003 MackT/uCF");  
    if (hWnd!=NULL)  
    return true;  
  return false;  
    
2.16 FD\_Find\_Debugger\_Process();  
  与上面的方法类似，区别是这个反调试用通过查询进程名字与已知的常用调试器应用程序名字进行比对，以确定是否有调试器处于运行状态。  
    if(strcmp(pe32.szExeFile,"OLLYICE.EXE")==0)  
        return true;  
    if(strcmp(pe32.szExeFile,"IDAG.EXE")==0)  
        return true;  
    if(strcmp(pe32.szExeFile,"OLLYDBG.EXE")==0)  
        return true;  
    if(strcmp(pe32.szExeFile,"PEID.EXE")==0)  
        return true;  
    if(strcmp(pe32.szExeFile,"SOFTICE.EXE")==0)  
        return true;  
    if(strcmp(pe32.szExeFile,"LORDPE.EXE")==0)  
        return true;  
    if(strcmp(pe32.szExeFile,"IMPORTREC.EXE")==0)  
        return true;  
    if(strcmp(pe32.szExeFile,"W32DSM89.EXE")==0)  
        return true;  
    if(strcmp(pe32.szExeFile,"WINDBG.EXE")==0)  
        return true;  
  
  
2.17 FD\_Find\_Device\_Driver()  
  调试工具通常会使用内核驱动，因此如果尝试是否可以打开一些调试器所用到的设备，就可判断是否存在调试器。常用的设备名称如下：  
\\\\.\\SICE  （SoftICE）  
\\\\.\\SIWVID（SoftICE）      
\\\\.\\NTICE  （SoftICE）      
\\\\.\\REGVXG（RegMON）  
\\\\.\\REGVXD（RegMON）  
\\\\.\\REGSYS（RegMON）  
\\\\.\\REGSYS（RegMON）  
\\\\.\\FILEVXG（FileMON）  
\\\\.\\FILEM（FileMON）  
\\\\.\\TRW（TRW2000）  
  
2.18 FD\_Exception\_Closehandle()  
  如果给 CloseHandle()函数一个无效句柄作为输入参数，在无调试器时，将会返回一个错误代码，而有调试器存在时，将会触发一个 EXCEPTION\_INVALID\_HANDLE (0xc0000008)的异常。  
  \_\_try    
  {  
    CloseHandle(HANDLE(0x00001234));  
    return false;  
  }  
  \_\_except(1)  
  {  
    return true;  
  }  
  
  
  
2.19 FD\_Exception\_Int3()  
  通过 Int3 产生异常中断的反调试比较经典。当 INT3 被执行到时, 如果程序未被调试, 将会异常处理器程序继续执行。而 INT3 指令常被调试器用于设置软件断点，int 3 会导致调试器误认为这是一个自己的断点，从而不会进入异常处理程序。  
  \_\_asm   
  {  
    push   offset exception\_handler; set exception handler  
    push  dword ptr fs:\[0h\]  
    mov    dword ptr fs:\[0h\],esp    
    xor   eax,eax;reset EAX invoke int3  
    int    3h  
    pop    dword ptr fs:\[0h\];restore exception handler  
    add   esp,4  
  
    test   eax,eax; check the flag   
    je    rt\_label  
    jmp    rf\_label  
  
exception\_handler:  
    mov   eax,dword ptr \[esp+0xc\];EAX = ContextRecord  
    mov    dword ptr \[eax+0xb0\],0xffffffff;set flag (ContextRecord.EAX)  
    inc   dword ptr \[eax+0xb8\];set ContextRecord.EIP  
    xor   eax,eax  
    retn  
  
rt\_label:  
    xor eax,eax  
    inc eax  
    mov esp,ebp  
    pop ebp  
    retn  
rf\_label:  
    xor eax,eax  
    mov esp,ebp  
    pop ebp  
    retn  
  }  
  
2.20 FD\_Exception\_Popf()  
我们都知道标志寄存器中的陷阱标志，当该标志被设置时，将产生一个单步异常。在程序中动态设置这给标志，如果处于调试器中，该异常将会被调试器捕获。  
可通过下面的代码设置标志寄存器。  
    pushf   
    mov dword ptr \[esp\], 0x100  
    popf  
  
2.21 FD\_OutputDebugString()  
  在有调试器存在和没有调试器存在时，OutputDebugString 函数表现会有所不同。最明显的不同是， 如果有调试器存在，其后的 GetLastError()的返回值为零。  
  OutputDebugString("");  
  tmpD=GetLastError();  
  if(tmpD==0)  
    return true;  
  return false;  
  
2.22 FD\_TEB\_check\_in\_Vista();  
  这是从 windows anti-debug reference 里拷贝出来的，据说是适用于 vista 系统下检测调试器。我没有 vista 所以也没有测试。有条件的可以试下，有问题帮忙反馈给我。多谢。  
    //vista  
    \_\_asm  
    {  
      push   offset exception\_handler; set exception handler  
      push  dword ptr fs:\[0h\]  
      mov    dword ptr fs:\[0h\],esp    
      xor   eax,eax;reset EAX invoke int3  
      int    3h  
      pop    dword ptr fs:\[0h\];restore exception handler  
      add   esp,4  
      mov eax, fs:\[18h\] ; teb  
      add eax, 0BFCh   
      mov ebx, \[eax\] ; pointer to a unicode string   
      test ebx, ebx ; (ntdll.dll, gdi32.dll,...)   
      je      rf\_label  
      jmp    rt\_label  
  exception\_handler:  
      mov   eax,dword ptr \[esp+0xc\];EAX = ContextRecord  
      inc   dword ptr \[eax+0xb8\];set ContextRecord.EIP  
      xor   eax,eax  
      retn  
    }   
  
2.23 FD\_check\_StartupInfo();  
  这是从 pediy 上拷贝来的。Window 创建进程的时候会把 STARTUPINFO 结构中的值设为 0,而通过调试器创建进程的时候会忽略这个结构中的值，也就是结构中的值不为 0，所以可以利用这个来判断是否在调试程序。  
  STARTUPINFO si;  
  ZeroMemory( &si, sizeof(si) );  
  si.cb = sizeof(si);  
  GetStartupInfo(&si);  
  if ( (si.dwX != 0) || (si.dwY !=0)   
    || (si.dwXCountChars != 0) || (si.dwYCountChars !=0 )   
    || (si.dwFillAttribute != 0) || (si.dwXSize != 0)   
    || (si.dwYSize != 0) )  
    return true;  
  else    
    return false;  
      
2.24 FD\_Parent\_Process1()  
与前面的 FD\_Parent\_Process 原理一样，唯一不同的是使用 ZwQueryInformationProcess 检测父进程，而没有使用 Process32Next，这有一个好处是可以绕过 OD 的 HideOD 插件。  
  
2.25 FD\_Exception\_Instruction\_count()  
  好像《软件加解密技术》中有提到这个反调试。  
  通过注册一个异常句柄，在特定地址设置一些硬件断点，当通过这些地址时都会触发 EXCEPTION\_SINGLE\_STEP (0x80000004)的异常，在异常处理程序中，将会调整指令指针到一条新指令，然后恢复运行。可以通过进入进程 context 结构来设置这些断点，有些调试器不能处理那些不是自己设置的硬件断点，从而导致一些指令将会被漏掉计数，这就形成了一个反调试。  
  \_\_asm  
  {  
    xor    eax,eax;  
    cdq;  
    push  e\_handler;  
    push  dword ptr fs:\[eax\];  
    mov    fs:\[eax\],esp;  
    int 3;  
hwbp1:  nop  
hwbp2:  nop  
hwbp3:  nop  
hwbp4:  nop  
    div    edx  
    nop  
    pop    dword ptr fs:\[0\]  
    add    esp,4  
    cmp    al,4;  
    jne    rt\_label;  
    jmp    rf\_label;  
  
e\_handler:  
    xor    eax,eax;  
    ;ExceptionRecord  
    mov    ecx,dword ptr\[esp+0x04\]  
    ;Contextrecord  
    mov    edx,dword ptr\[esp+0x0c\]  
    ;ContextEIP  
    inc    byte ptr\[edx+0xb8\];  
      
    ;ExceptionCode  
    mov    ecx,dword ptr\[ecx\];  
  
    ;1.EXCEPTION\_INT\_DIVIDE\_BY\_ZERO  
    cmp    ecx,0xc0000094;  
    jne    Ex\_next2;  
    ;Context\_eip  
    inc    byte ptr\[edx+0xb8\];  
    mov    dword ptr\[edx+0x04\],eax;dr0  
    mov    dword ptr\[edx+0x08\],eax;dr1  
    mov    dword ptr\[edx+0x0c\],eax;dr2  
    mov    dword ptr\[edx+0x10\],eax;dr3  
    mov    dword ptr\[edx+0x14\],eax;dr6  
    mov    dword ptr\[edx+0x18\],eax;dr7  
    ret  
  
    ;2.EXCEPTION\_BREAKPOINT  
Ex\_next2:  
    cmp    ecx,0x80000003;  
    jne    Ex\_next3;  
  
    mov    dword ptr\[edx+0x04\],offset hwbp1;dr0  
    mov    dword ptr\[edx+0x08\],offset hwbp2;dr1  
    mov    dword ptr\[edx+0x0c\],offset hwbp3;dr2  
    mov    dword ptr\[edx+0x10\],offset hwbp4;dr3  
    mov    dword ptr\[edx+0x18\],0x155;dr7  
    ret  
  
    ;3.EXCEPTION\_SINGLE\_STEP  
Ex\_next3:  
    cmp  ecx,0x80000004  
    jne    rt\_label  
    ;CONTEXT\_Eax  
    inc    byte ptr\[edx+0xb0\]  
    ret  
  }  
  
2.26 FD\_INT\_2d()  
在 windows anti-debug reference 中指出，如果程序未被调试这个中断将会生产一个断点异常. 被调试并且未使用跟踪标志执行这个指令, 将不会有异常产生程序正常执行. 如果被调试并且指令被跟踪, 尾随的字节将被跳过并且执行继续. 因此, 使用 INT 2Dh 能作为一个强有力的反调试和反跟踪机制。  
  \_\_try  
  {  
    \_\_asm  
    {  
        int 2dh  
      inc eax;any opcode of singlebyte.  
      ;or u can put some junkcode,"0xc8"..."0xc2"..."0xe8"..."0xe9"  
    }  
  return true;  
  }  
  \_\_except(1)  
  {  
    return false;  
  }  
  
  
  
三、  检测-专用调试器(FS\_)  
    这一部分是我比较喜欢的，但内容还不是很丰富，比如：  
1、  针对 SoftIce 的检测方法有很多，但由于我从没使用过 Softice，也没有条件去测试，所以没有给出太多，有兴趣的可以自己查阅资料进行补充，针对 softice 网上资料较多，或查阅《软件加解密技术》。  
2、  同样，这里也没有给出 windbg 等等其它调试器的检测方法。  
3、  而针对 Odplugin，也只给了几种 HideOD 的检测。事实上，目前 OD 的使用者通常都使用众多的强大插件，当 OD 的反调试越来越普遍时，自己设计几款常用的 OD 插件的反调试，将会是非常有效的反调试手段。  
4、  对 VME 的检测也只给出了两种，如想丰富这一部分可以参考 Peter Ferrie 的一篇 anti-vme 的文章（http://bbs.pediy.com/showthread.php?t=68411）。里面有非常多的 anti-vme 方法。  
  
    针对专用调试器的函数列表如下：  
//find specific debugger  
bool FS\_OD\_Exception\_GuardPages();  
bool FS\_OD\_Int3\_Pushfd();  
bool FS\_SI\_UnhandledExceptionFilter();  
bool FS\_ODP\_Process32NextW();  
bool FS\_ODP\_OutputDebugStringA();  
bool FS\_ODP\_OpenProcess();  
bool FS\_ODP\_CheckRemoteDebuggerPresent();  
bool FS\_ODP\_ZwSetInformationThread();  
bool FS\_SI\_Exception\_Int1();  
bool IsInsideVMWare\_();  
bool FV\_VMWare\_VMX();  
bool FV\_VPC\_Exception();  
int FV\_VME\_RedPill();//0:none,1:vmvare;2:vpc;3:others  
  
3.1 FS\_OD\_Exception\_GuardPages  
    “保护页异常”是一个简单的反调试技巧。当应用程序尝试执行保护页内的代码时，将会产生一个 EXCEPTION\_GUARD\_PAGE(0x80000001)异常，但如果存在调试器，调试器有可能接收这个异常，并允许该程序继续运行，事实上，在 OD 中就是这样处理的，OD 使用保护页来实现内存断点。  
最开始实现时忘记了 free 申请的空间，多谢 sessiondiy 提醒。  
  
  SYSTEM\_INFO sSysInfo;  
  LPVOID lpvBase;  
  BYTE \* lptmpB;  
  GetSystemInfo(&sSysInfo);  
  DWORD dwPageSize=sSysInfo.dwPageSize;  
  DWORD flOldProtect;  
  
  DWORD dwErrorcode;  
  
  lpvBase=VirtualAlloc(NULL,dwPageSize,MEM\_COMMIT,PAGE\_READWRITE);  
  if(lpvBase==NULL)  
    return false;  
    
  lptmpB=(BYTE \*)lpvBase;  
  \*lptmpB=0xc3;//retn  
  
  VirtualProtect(lpvBase,dwPageSize,PAGE\_EXECUTE\_READ | PAGE\_GUARD,&flOldProtect);  
    
  \_\_try  
  {  
    \_\_asm  call dword ptr\[lpvBase\];  
    VirtualFree(lpvBase,0,MEM\_RELEASE);  
    return true;  
  }  
  \_\_except(1)  
  {  
    VirtualFree(lpvBase,0,MEM\_RELEASE);  
    return false;  
  }  
  
3.2 FS\_OD\_Int3\_Pushfd  
    这是个最近比较牛 X 的反调试，据称是 vmp1.64 里发现的，好像 ttprotect 里面也有使用，我没有验证。Pediy 里有帖子详细讨论，我是看到 gkend 的分析，才搞懂一些。下面摘自 gkend 分析  

代码:

```
    int3，pushfd 和 int3，popfd 一样的效果。只要修改 int3 后面的 popfd 为其他值，OD 都能通过。老掉牙的技术又重新被用了。SEH 异常机制的运用而已。
    原理：在 SEH 异常处理中设置了硬件断点 DR0=EIP+2，并把 EIP 的值加 2，那么应该在 int3,popfd 后面的指令执行时会产生单步异常。但是 OD 遇到前面是 popfd/pushfd 时，OD 会自动在 popfd 后一指令处设置硬件断点，而 VMP 的 seh 异常处理会判断是否已经设置硬件断点，如果已经有硬件断点就不产生单步异常，所以不能正常执行。

```

    http://bbs.pediy.com/showthread.php?t=67737  
    大家也可以仔细研究下 OD 下的 pushfd,popfd 等指令，相信利用它们可以构造很多反调试，下面是我实现的一个，不过现在看起来有点没看懂，不知当时为什么用了两个 int3。  
\_\_asm  
  {  
    push   offset e\_handler; set exception handler  
    push  dword ptr fs:\[0h\]  
    mov    dword ptr fs:\[0h\],esp    
    xor   eax,eax;reset EAX invoke int3  
    int    3h  
    pushfd  
    nop  
    nop  
    nop  
    nop  
    pop    dword ptr fs:\[0h\];restore exception handler  
    add   esp,4  
  
    test   eax,eax; check the flag   
    je    rf\_label  
    jmp    rt\_label  
  
e\_handler:  
    push   offset e\_handler1; set exception handler  
    push  dword ptr fs:\[0h\]  
    mov    dword ptr fs:\[0h\],esp    
    xor   eax,eax;reset EAX invoke int3  
    int    3h  
    nop  
    pop    dword ptr fs:\[0h\];restore exception handler  
    add   esp,4  
    ;EAX = ContextRecord  
    mov    ebx,eax;dr0=>ebx  
    mov   eax,dword ptr \[esp+0xc\]  
    ;set ContextRecord.EIP  
    inc   dword ptr \[eax+0xb8\];  
    mov    dword ptr \[eax+0xb0\],ebx;dr0=>eax  
    xor    eax,eax  
    retn  
  
e\_handler1:  
    ;EAX = ContextRecord  
    mov   eax,dword ptr \[esp+0xc\]  
    ;set ContextRecord.EIP  
    inc   dword ptr \[eax+0xb8\];  
    mov    ebx,dword ptr\[eax+0x04\]  
    mov    dword ptr \[eax+0xb0\],ebx;dr0=>eax  
    xor    eax,eax  
    retn  
rt\_label:  
    xor  eax,eax  
    inc eax  
    mov esp,ebp  
    pop  ebp  
    retn  
rf\_label:  
    xor eax,eax  
    mov esp,ebp  
    pop ebp  
    retn  
  }  
  
3.3 FS\_SI\_UnhandledExceptionFilter  
    这个针对 SoftIce 的反调试很简单，好像是 SoftIce 会修改 UnhandledExceptionFilter 这个函数的第一个字节为 CC。因此判断这个字节是否为 cc，就是一种检查 softice 的简便方法。  
  
FARPROC Uaddr ;  
BYTE tmpB = 0;  
(FARPROC&) Uaddr =GetProcAddress ( GetModuleHandle("kernel32.dll"),"UnhandledExceptionFilter");  
tmpB = \*((BYTE\*)Uaddr);   // 取 UnhandledExceptionFilter 函数第一字节  
tmpB=tmpB⁰x55;  
if(tmpB ==0x99)           // 如该字节为 CC，则 SoftICE 己加载  
returntrue;  
else  
returnfalse;  
  
3.4 FS\_ODP\_Process32NextW  
    当我在调试 FD\_parentprocess 时，感觉总是怪怪的，使用 OD 时运行 Process32NextW 总是返回失败，搞了一个晚上，才搞懂原来是 OD 的插件 HideOD 在作怪。当 HideOD 的 Process32NextW 的选项被选中时，它会更改 Process32NextW 的返回值，使其始终返回 false，这主要是 HideOD 针对 FD\_parentprocess 这个反调试的一个反反调试。但也正是这一点暴露的它的存在。  
int OSVersion;  
  FARPROC Func\_addr;  
  WORD tmpW;  
//1.Process32Next  
  HMODULE hModule = GetModuleHandle("kernel32.dll");   
  (FARPROC&) Func\_addr =GetProcAddress ( hModule,"Process32NextW");  
if (Func\_addr != NULL)   
  {  
    tmpW=\*(WORD\*)Func\_addr;  
    OSVersion=myGetOSVersion();  
switch(OSVersion)  
    {  
case OS\_winxp:  
if(tmpW!=0xFF8B)//maybe option of Process32Next is selected.  
returntrue;  
break;  
default:  
if(tmpW==0xC033)  
returntrue;  
break;  
    }  
  }  
    但上面的代码并不完美，因为有跨平台问题，所以要先获得当前操作系统版本。目前只在 win2k 和 winxp 下进行了测试。  
  
3.5 FS\_ODP\_OutputDebugStringA  
    同样,HIDEOD 的 OutputDebugStringA 选项，也对 OutputDebugStringA 这个 api 做了处理，具体修改内容我记不得了，大家可以自己比对一下。我的代码如下：  
int OSVersion;  
  FARPROC Func\_addr;  
  WORD tmpW;  
//2.OutputDebugStringA  
  HMODULE hModule = GetModuleHandle("kernel32.dll");   
  (FARPROC&) Func\_addr =GetProcAddress ( hModule,"OutputDebugStringA");  
if (Func\_addr != NULL)   
  {  
    tmpW=\*(WORD\*)Func\_addr;  
    OSVersion=myGetOSVersion();  
switch(OSVersion)  
    {  
case OS\_winxp:  
if(tmpW!=0x3468)//maybe option of OutputDebugStringAt is selected.  
returntrue;  
break;  
default:  
if(tmpW==0x01e8)  
returntrue;  
break;  
    }  
  }  
returnfalse;  
  
3.6 FS\_ODP\_OpenProcess  
    这个据称这个是针对 HideDebugger 这个插件的，当这个插件开启时，它会挂钩 OpenProcess 这个函数，它修改了 OpenProcess 的前几个字节。因此检测这几个字节就可实现这个反调试。  
  FARPROC Func\_addr;  
  BYTE tmpB;  
//OpenProcess  
  HMODULE hModule = GetModuleHandle("kernel32.dll");   
  (FARPROC&) Func\_addr =GetProcAddress ( hModule,"OpenProcess");  
if (Func\_addr != NULL)   
  {  
    tmpB=\*((BYTE\*)Func\_addr+6);  
if(tmpB==0xea)//HideDebugger Plugin of OD is present  
returntrue;  
  }  
returnfalse;  
  
3.7 FS\_ODP\_CheckRemoteDebuggerPresent  
    和前面提到的两个 HideOD 的反调试类似，不多说了。大家可以自行比对一下开启和不开启 HideOD 时，CheckRemoteDebuggerPresent 函数的异同，就可以设计反这个插件的反调试了。  
int OSVersion;  
  FARPROC Func\_addr;  
  BYTE tmpB;  
//2.CheckRemoteDebuggerPresent  
  HMODULE hModule = GetModuleHandle("kernel32.dll");   
  (FARPROC&) Func\_addr =GetProcAddress ( hModule,"CheckRemoteDebuggerPresent");  
if (Func\_addr != NULL)   
  {  
    tmpB=\*((BYTE\*)Func\_addr+10);  
    OSVersion=myGetOSVersion();  
switch(OSVersion)  
    {  
case OS\_winxp:  
if(tmpB!=0x74)//HideOD is present  
returntrue;  
break;  
default:  
break;  
    }  
  }  
returnfalse;  
  
3.8 FS\_ODP\_ZwSetInformationThread  
    和前面提到的几个 HideOD 的反调试类似，大家可以自行比对一下开启和不开启 HideOD 时，ZwSetInformationThread 函数的异同，就可以设计反这个插件的反调试了。  
int OSVersion;  
  FARPROC Func\_addr;  
  WORD tmpW;  
  BYTE tmpB0,tmpB1;  
//2.CheckRemoteDebuggerPresent  
  HMODULE hModule = GetModuleHandle("ntdll.dll");   
  (FARPROC&) Func\_addr =GetProcAddress ( hModule,"ZwSetInformationThread");  
if (Func\_addr != NULL)   
  {  
    tmpW=\*((WORD\*)Func\_addr+3);  
    tmpB0=\*((BYTE\*)Func\_addr+9);  
    tmpB1=\*((BYTE\*)Func\_addr+10);  
    OSVersion=myGetOSVersion();  
switch(OSVersion)  
    {  
case OS\_winxp:  
if(tmpW!=0x0300)//HideOD is present  
returntrue;  
break;  
case OS\_win2k:  
if((tmpB0!=0xcd)&&(tmpB1!=0x2e))  
returntrue;  
break;  
default:  
break;  
    }  
  }  
returnfalse;  
  
3.9 FS\_SI\_Exception\_Int1  
    通常 int1 的 DPL 为 0,这表示"cd 01"机器码不能在 3 环下执行。如果直接执行这个中断将会产生一个保护错误，windows 会产生一个 EXCEPTION\_ACCESS\_VIOLATION (0xc0000005)异常。然而，如果 SOFTICE 正在运行，它挂钩了 int1，并调整其 DPL 为 3。这样 SoftICE 就可以在用户模式执行单步操作了。  
    当 int 1 发生时，SoftICE 不检查它是由于陷阱标志位还是由软件中断产生，SoftICE 总是去调用原始中断 1 的句柄，此时将会产生一个 EXCEPTION\_SINGLE\_STEP (0x80000004)而不是 EXCEPTION\_ACCESS\_VIOLATION (0xc0000005)异常，这就形成了一个简单的反调试方法。  
  
\_\_asm  
  {  
    push   offset eh\_int1; set exception handler  
    push  dword ptr fs:\[0h\]  
    mov    dword ptr fs:\[0h\],esp    
    xor   eax,eax;reset flag(EAX) invoke int3  
    int    1h  
    pop    dword ptr fs:\[0h\];restore exception handler  
    add   esp,4  
  
    cmp    eax,0x80000004; check the flag   
    je    rt\_label\_int1  
    jmp    rf\_label\_int1  
  
eh\_int1:  
    mov    eax,\[esp+0x4\];  
    mov    ebx,dword ptr \[eax\];  
    mov   eax,dword ptr \[esp+0xc\];EAX = ContextRecord  
    mov    dword ptr \[eax+0xb0\],ebx;set flag (ContextRecord.EAX)  
  
    inc   dword ptr \[eax+0xb8\];set ContextRecord.EIP  
    inc   dword ptr \[eax+0xb8\];set ContextRecord.EIP  
    xor   eax,eax  
    retn  
  }  
  
3.10 FV\_VMWare\_VMX  
    这是一个针对 VMWare 虚拟机仿真环境的反调试，我从网上直接拷贝的代码。  
    VMWARE 提供一种主机和客户机之间的通信方法，这可以被用做一种 VMWare 的反调试。Vmware 将会处理 IN （端口为 0x5658/’VX’）指令，它会返回一个 majic 数值“VMXh”到 EBX 中。  
    当在保护模式操作系统的 3 环下运行时，IN 指令的执行将会产生一个异常，除非我们修改了 I/O 的优先级等级。然而，如果在 VMWare 下运行，将不会产生任何异常，同时 EBX 寄存器将会包含’VMXh’，ECX 寄存器也会被修改为 Vmware 的产品 ID。  
    这种技巧在一些病毒中比较常用。  
    针对 VME 的反调试，在 peter Ferrie 的另一篇文章<<Attacks on More Virtual Machine Emulators>>中有大量的描述，有兴趣的可以根据这篇文章，将 FV\_ 反调试好好丰富一下。  
  
bool IsInsideVMWare\_()  
{  
bool r;  
\_asm  
  {  
    push   edx  
    push   ecx  
    push   ebx  
  
    mov    eax, 'VMXh'  
    mov    ebx, 0 // any value but MAGIC VALUE  
    mov    ecx, 10 // get VMWare version  
    mov    edx, 'VX' // port number  
    in     eax, dx // read port  
// on return EAX returns the VERSION  
    cmp    ebx, 'VMXh' // is it a reply from VMWare?  
    setz   \[r\] // set return value  
  
    pop    ebx  
    pop    ecx  
    pop    edx  
  }  
return r;  
}  
  
bool FV\_VMWare\_VMX()  
{  
\_\_try  
  {  
return IsInsideVMWare\_();  
  }  
\_\_except(1) // 1 = EXCEPTION\_EXECUTE\_HANDLER  
  {  
returnfalse;  
  }  
}  
  
3.11 FV\_VPC\_Exception  
    这个代码我也是完整从网上拷贝下来的，具体原理在<<Attacks on More Virtual Machine Emulators>>这篇文章里也有详细描述。与 VMWare 使用一个特殊端口完成主机和客户机间通信的方法类似的是，VirtualPC 靠执行非法指令产生一个异常供内核捕获。这个代码如下：  
代码:

```
0F 3F x1 x2
0F C7 C8 y1 y2

```

    由这两个非法指令引起的异常将会被应用程序捕获，然而，如果 VirtualPC 正在运行，将不会产生异常。X1,x2 的允许的数值还不知道，但有一部分已知可以使用，如 0A 00,11 00…等等。  
\_\_declspec(naked) bool FV\_VPC\_Exception()  
{  
\_asm  
  {  
    push ebp  
    mov  ebp, esp  
  
    mov  ecx, offset exception\_handler  
  
    push ebx  
    push ecx  
  
    push dword ptr fs:\[0\]  
    mov  dword ptr fs:\[0\], esp  
  
    mov  ebx, 0 // Flag  
    mov  eax, 1 // VPC function number  
  }  
  
// call VPC   
\_asm \_\_emit 0Fh  
\_asm \_\_emit 3Fh  
\_asm \_\_emit 07h  
\_asm \_\_emit 0Bh  
  
\_asm  
  {  
    mov eax, dword ptr ss:\[esp\]  
    mov dword ptr fs:\[0\], eax  
  
    add esp, 8  
  
    test ebx, ebx  
  
    setz al  
  
    lea esp, dword ptr ss:\[ebp-4\]  
    mov ebx, dword ptr ss:\[esp\]  
    mov ebp, dword ptr ss:\[esp+4\]  
  
    add esp, 8  
  
    jmp ret1  
exception\_handler:  
    mov ecx, \[esp+0Ch\]  
    mov dword ptr \[ecx+0A4h\], -1 // EBX = -1 -> not running, ebx = 0 -> running  
    add dword ptr \[ecx+0B8h\], 4 // -> skip past the call to VPC  
    xor eax, eax // exception is handled  
    ret  
 ret1:  
    ret  
  }  
}  
  
3.12 FV\_VME\_RedPill  
    这个方法似乎是检测虚拟机的一个简单有效的方法，虽然还不能确定它是否是 100%有效。名字很有意思，红色药丸（为什么不是 bluepill,哈哈）。我在网上找到了个 ppt 专门介绍这个方法，可惜现在翻不到了。记忆中原理是这样的，主要检测 IDT 的数值，如果这个数值超过了某个数值，我们就可以认为应用程序处于虚拟环境中，似乎这个方法在多 CPU 的机器中并不可靠。据称 ScoobyDoo 方法是 RedPill 的升级版。代码也是在网上找的，做了点小改动。有四种返回结果，可以确认是 VMWare，还是 VirtualPC，还是其它 VME，或是没有处于 VME 中。  
//return value:  0:none,1:vmvare;2:vpc;3:others  
unsignedchar matrix\[6\];  
  
unsignedchar redpill\[\] =   
        "\\x0f\\x01\\x0d\\x00\\x00\\x00\\x00\\xc3";  
  
    HANDLE hProcess = GetCurrentProcess();  
  
    LPVOID lpAddress = NULL;  
    PDWORD lpflOldProtect = NULL;  
  
\_\_try  
    {  
        \*((unsigned\*)&redpill\[3\]) = (unsigned)matrix;  
  
        lpAddress = VirtualAllocEx(hProcess, NULL, 6, MEM\_RESERVE|MEM\_COMMIT , PAGE\_EXECUTE\_READWRITE);  
  
if(lpAddress == NULL)  
return 0;  
  
        BOOL success = VirtualProtectEx(hProcess, lpAddress, 6, PAGE\_EXECUTE\_READWRITE , lpflOldProtect);  
  
if(success != 0)  
return 0;  
  
        memcpy(lpAddress, redpill, 8);  
  
        ((void(\*)())lpAddress)();  
  
if (matrix\[5\]>0xd0)   
        {  
if(matrix\[5\]==0xff)//vmvare  
return 1;  
elseif(matrix\[5\]==0xe8)//vitualpc  
return 2;  
else  
return 3;  
        }  
else  
return 0;  
    }  
\_\_finally  
    {  
        VirtualFreeEx(hProcess, lpAddress, 0, MEM\_RELEASE);  
    }  
  
四、  检测-断点(FB\_)  
这一部分内容较少，但实际上可用的方法也比较多，我没有深入研究，不敢乱写，照抄了几个常用的方法：  
//find breakpoint  
bool FB\_HWBP\_Exception();  
DWORD FB\_SWBP\_Memory\_CRC();  
bool FB\_SWBP\_ScanCC(BYTE \* addr,int len);  
bool FB\_SWBP\_CheckSum\_Thread(BYTE \*addr\_begin,BYTE \*addr\_end,DWORD sumValue);  
  
4.1 FB\_HWBP\_Exception  
  在异常处理程序中检测硬件断点，是比较常用的硬件断点检测方法。在很多地方都有提到。  
\_\_asm  
  {  
    push   offset exeception\_handler; set exception handler  
    push   dword ptr fs:\[0h\]  
    mov    dword ptr fs:\[0h\],esp    
    xor    eax,eax;reset EAX invoke int3  
    int    1h  
    pop    dword ptr fs:\[0h\];restore exception handler  
    add    esp,4  
    ;test if EAX was updated (breakpoint identified)   
    test   eax,eax  
    jnz     rt\_label  
    jmp    rf\_label  
  
exeception\_handler:  
    ;EAX = CONTEXT record  
    mov     eax,dword ptr \[esp+0xc\]  
  
    ;check if Debug Registers Context.Dr0-Dr3 is not zero  
    cmp     dword ptr \[eax+0x04\],0  
    jne     hardware\_bp\_found  
    cmp     dword ptr \[eax+0x08\],0  
    jne     hardware\_bp\_found  
    cmp     dword ptr \[eax+0x0c\],0  
    jne     hardware\_bp\_found  
    cmp     dword ptr \[eax+0x10\],0  
    jne     hardware\_bp\_found  
    jmp     exception\_ret  
  
hardware\_bp\_found:  
    ;set Context.EAX to signal breakpoint found  
    mov     dword ptr \[eax+0xb0\],0xFFFFFFFF  
exception\_ret:  
    ;set Context.EIP upon return  
    inc       dword ptr \[eax+0xb8\];set ContextRecord.EIP  
    inc       dword ptr \[eax+0xb8\];set ContextRecord.EIP  
    xor     eax,eax  
    retn  
  }  
4.2 FB\_SWBP\_Memory\_CRC()  
  由于在一些常用调试器中，比如 OD，其是将代码设置为 0xcc 来实现普通断点，因此当一段代码被设置了普通断点，则其中必定有代码的修改。因此对关键代码进行 CRC 校验则可以实现侦测普通断点。但麻烦的是每次代码修改，或更换编译环境，都要重新设置 CRC 校验值。  
  下面的代码拷贝自《软件加解密技术》，里面完成的是对整个代码段的 CRC 校验，CRC 校验值保存在数据段。CRC32 算法实现代码网上有很多，就不列出来了。  
DWORD FB\_SWBP\_Memory\_CRC()  
{  
//打开文件以获得文件的大小  
  DWORD fileSize,NumberOfBytesRW;  
  DWORD CodeSectionRVA,CodeSectionSize,NumberOfRvaAndSizes,DataDirectorySize,ImageBase;  
  BYTE\* pMZheader;  
  DWORD pPEheaderRVA;  
  TCHAR  \*pBuffer ;  
  TCHAR szFileName\[MAX\_PATH\];   
  
  GetModuleFileName(NULL,szFileName,MAX\_PATH);  
//打开文件  
  HANDLE hFile = CreateFile(  
    szFileName,  
    GENERIC\_READ,  
    FILE\_SHARE\_READ,   
    NULL,  
    OPEN\_EXISTING,  
    FILE\_ATTRIBUTE\_NORMAL,  
    NULL);  
if ( hFile != INVALID\_HANDLE\_VALUE )  
   {  
//获得文件长度 :  
    fileSize = GetFileSize(hFile,NULL);  
if (fileSize == 0xFFFFFFFF) return 0;  
    pBuffer = new TCHAR \[fileSize\];     //// 申请内存,也可用 VirtualAlloc 等函数申请内存  
    ReadFile(hFile,pBuffer, fileSize, &NumberOfBytesRW, NULL);//读取文件内容  
    CloseHandle(hFile);  //关闭文件  
   }  
else  
return 0;  
  pMZheader=(BYTE\*)pBuffer; //此时 pMZheader 指向文件头  
  pPEheaderRVA = \*(DWORD \*)(pMZheader+0x3c);//读 3ch 处的 PE 文件头指针  
///定位到 PE 文件头（即字串“PE\\0\\0”处）前 4 个字节处，并读出储存在这里的 CRC-32 值：  
  
  NumberOfRvaAndSizes=\*((DWORD \*)(pMZheader+pPEheaderRVA+0x74));//得到数据目录结构数量  
  DataDirectorySize=NumberOfRvaAndSizes\*0x8;//得到数据目录结构大小  
  ImageBase=\*((DWORD \*)(pMZheader+pPEheaderRVA+0x34));//得到基地址  
//假设第一个区块就是代码区块  
  CodeSectionRVA=\*((DWORD \*)(pMZheader+pPEheaderRVA+0x78+DataDirectorySize+0xc));//得到代码块的 RVA 值  
  CodeSectionSize=\*((DWORD \*)(pMZheader+pPEheaderRVA+0x78+DataDirectorySize+0x8));///得到代码块的内存大小  
delete pBuffer;  // 释放内存  
return CRC32((BYTE\*)(CodeSectionRVA+ImageBase),CodeSectionSize);  
}  
  
4.3 FB\_SWBP\_ScanCC  
扫描 CC 的方法，比照前面校验代码 CRC 数值的方法更直接一些，它直接在所要检测的代码区域内检测是否有代码被更改为 0xCC，0xcc 对应汇编指令为 int3 ,对一些常用的调试器(如 OD)其普通断点就是通过修改代码为 int3 来实现的。但使用时要注意是否正常代码中就包含 CC。通常这个方法用于扫描 API 函数的前几个字节，比如检测常用的 MessageBoxA、GetDlgItemTextA 等。  
  
bool FB\_SWBP\_ScanCC(BYTE \* addr,int len)  
{  
  FARPROC Func\_addr ;  
  HMODULE hModule = GetModuleHandle("USER32.dll");   
  (FARPROC&) Func\_addr =GetProcAddress ( hModule, "MessageBoxA");  
if (addr==NULL)  
    addr=(BYTE \*)Func\_addr;//for test  
  BYTE tmpB;  
int i;  
  \_\_try  
  {  
for(i=0;i<len;i++,addr++)  
    {  
      tmpB=\*addr;  
      tmpB=tmpB⁰x55;  
if(tmpB==0x99)// cmp 0xcc  
returntrue;  
    }  
  }  
  \_\_except(1)  
returnfalse;  
returnfalse;  
}  
  
4.4 FB\_SWBP\_CheckSum\_Thread(BYTE \*addr\_begin,BYTE \*addr\_end,DWORD sumValue);  
此方法类似 CRC 的方法，只是这里是检测累加和。它与 CRC 的方法有同样的问题，就是要在编译后，计算累加和的数值，再将该值保存到数据区，重新编译。在这里创建了一个单独的线程用来监视代码段。  
DWORD WINAPI CheckSum\_ThreadFunc( LPVOID lpParam )   
{   
  DWORD dwThrdParam\[3\];  
  BYTE tmpB;  
  DWORD Value=0;  
  dwThrdParam\[0\]=\* ((DWORD \*)lpParam);  
     dwThrdParam\[1\]=\* ((DWORD \*)lpParam+1);  
      dwThrdParam\[2\]=\* ((DWORD \*)lpParam+2);  
  BYTE \*addr\_begin=(BYTE \*)dwThrdParam\[0\];  
  BYTE \*addr\_end=(BYTE \*)dwThrdParam\[1\];  
  DWORD sumValue=dwThrdParam\[2\];  
for(int i=0;i<(addr\_end-addr\_begin);i++)  
    Value=Value+\*(addr\_begin+i);  
/\* //if sumvalue is const,it should be substract.  
  DWORD tmpValue;  
  Value=Value-(sumValue&0x000000FF);  
  tmpValue=(sumValue&0x0000FF00)>>8;  
  Value=Value-tmpValue;  
  tmpValue=(sumValue&0x0000FF00)>>16;  
  Value=Value-tmpValue;  
  tmpValue=(sumValue&0x0000FF00)>>24;  
  Value=Value-tmpValue;\*/  
if (Value!=sumValue)  
    MessageBox(NULL,"SWBP is found by CheckSum\_ThreadFunc","CheckSum\_ThreadFunc",MB\_OK|MB\_ICONSTOP);  
return 1;   
}  
  
bool FB\_SWBP\_CheckSum\_Thread(BYTE \*addr\_begin,BYTE \*addr\_end,DWORD sumValue)  
{  
    DWORD dwThreadId;  
  DWORD dwThrdParam\[3\];  
  dwThrdParam\[0\]=(DWORD)addr\_begin;  
  dwThrdParam\[1\]=(DWORD)addr\_end;  
  dwThrdParam\[2\]=sumValue;  
    HANDLE hThread;   
  
    hThread = CreateThread(   
        NULL,                        // default security attributes   
        0,                           // use default stack size    
        CheckSum\_ThreadFunc,         // thread function   
        &dwThrdParam\[0\],                // argument to thread function   
        0,                           // use default creation flags   
        &dwThreadId);                // returns the thread identifier   
// Check the return value for success.   
  
if (hThread == NULL)   
returnfalse;  
else  
   {  
      Sleep(1000);  
      CloseHandle( hThread );  
returntrue;  
   }  
}  
五、  检测-跟踪(FT\_)  
个人认为，反跟踪的一些技巧，多数不会非常有效，因为在调试时，多数不会被跟踪经过，除非用高超的技巧将关键代码和垃圾代码及这些反跟踪技巧融合在一起，否则很容易被发现或被无意中跳过。  
函数列表如下：  
//Find Single-Step or Trace  
bool FT\_PushSS\_PopSS();  
void FT\_RDTSC(unsignedint \* time);  
DWORD FT\_GetTickCount();  
DWORD FT\_SharedUserData\_TickCount();  
DWORD FT\_timeGetTime();  
LONGLONG FT\_QueryPerformanceCounter(LARGE\_INTEGER \*lpPerformanceCount);  
bool FT\_F1\_IceBreakpoint();  
bool FT\_Prefetch\_queue\_nop1();  
bool FT\_Prefetch\_queue\_nop2();  
  
5.1 FT\_PushSS\_PopSS  
这个反调试在<<windows anti-debug reference>>里有描述，如果调试器跟踪经过下面的指令序列：  
\_\_asm  
  {  
    push ss    //反跟踪指令序列  
    ;junk  
    pop  ss    //反跟踪指令序列  
    pushf    //反跟踪指令序列  
    ;junk  
    pop eax    //反跟踪指令序列  
}  
Pushf 将会被执行，同时调试器无法设置压进堆栈的陷阱标志，应用程序通过检测陷阱标志就可以判断处是否被跟踪调试。  
  
\_\_asm  
  {  
    push ebp  
    mov ebp,esp  
    push ss    //反跟踪指令序列  
    ;junk  
    pop  ss    //反跟踪指令序列  
    pushf    //反跟踪指令序列  
    ;junk  
    pop eax    //反跟踪指令序列  
    and  eax,0x00000100  
    jnz  rt\_label  
  
    xor eax,eax  
    mov esp,ebp  
    pop ebp  
    retn  
rt\_label:  
    xor eax,eax  
    inc eax  
    mov esp,ebp  
    pop ebp  
    retn  
  }  
  
5.2 FT\_RDTSC  
通过检测某段程序执行的时间间隔，可以判断出程序是否被跟踪调试，被跟踪调试的代码通常都有较大的时间延迟，检测时间间隔的方法有很多种。比如 RDTSC 指令，kernel32\_GetTickCount 函数，winmm\_ timeGetTime 函数等等。  
下面为 RDTSC 的实现代码。  
  
int time\_low,time\_high;  
\_\_asm  
  {  
    rdtsc  
    mov    time\_low,eax  
    mov    time\_high,edx  
  }  
  
5.3 FT\_GetTickCount  
  GetTickCount 函数检测时间间隔简单且常用。直接调用即可。具体可查 MSDN。  
  
5.4 FT\_SharedUserData\_TickCount  
  直接调用 GetTickCount 函数来检测时间间隔的方法，虽然简单却容易被发现。而使用 GetTickCount 的内部实现代码，直接读取 SharedUserData 数据结构里的数据的方法，更隐蔽些。下面的代码是直接从 GetTickCount 里扣出来的，其应该是在位于 0x7FFE0000 地址处的 SharedUserData 数据接口里面直接取数据，不过这个代码应该有跨平台的问题，我这里没有处理。大家可以完善下。  
  DWORD tmpD;  
\_\_asm  
  {  
    mov     edx, 0x7FFE0000  
    mov     eax, dword ptr \[edx\]  
    mul     dword ptr \[edx+4\]  
    shrd    eax, edx, 0x18  
    mov    tmpD,eax  
  }  
return tmpD;  
  
5.5 FT\_timeGetTime  
  使用 winmm 里的 timeGetTime 的方法也可以用来检测时间间隔。直接调用这个函数即可。具体可查 MSDN。  
  
5.6 FT\_QueryPerformanceCounter  
  这是一种高精度时间计数器的方法，它的检测刻度最小，更精确。  
if(QueryPerformanceCounter(lpPerformanceCount))  
return lpPerformanceCount->QuadPart;  
else  
return 0;  
  
5.7 FT\_F1\_IceBreakpoint  
  在<<Windows anti-debug reference>>中有讲述这个反跟踪技巧。这个所谓的"Ice breakpoint" 是 Intel 未公开的指令之一, 机器码为 0xF1.执行这个指令将产生单步异常.,如果程序已经被跟踪, 调试器将会以为它是通过设置标志寄存器中的单步标志位生成的正常异常. 相关的异常处理器将不会被执行到.下面是我的实现代码：  
  
\_\_asm  
  {  
  push   offset eh\_f1; set exception handler  
     push  dword ptr fs:\[0h\]  
     mov    dword ptr fs:\[0h\],esp    
     xor   eax,eax;reset EAX invoke int3  
     \_emit 0xf1  
     pop    dword ptr fs:\[0h\];restore exception handler  
     add    esp,4  
  test  eax,eax  
  jz    rt\_label\_f1  
  jmp    rf\_label\_f1  
  
eh\_f1:  
     mov eax,dword ptr\[esp+0xc\]  
  mov    dword ptr \[eax+0xb0\],0x00000001;set flag (ContextRecord.EAX)  
     inc dword ptr \[eax+0xb8\]  
     xor eax,eax  
     retn  
rt\_label\_f1:  
  inc    eax  
  mov    esp,ebp  
     pop    ebp  
     retn  
rf\_label\_f1:  
  xor    eax,eax  
  mov    esp,ebp  
     pop    ebp  
     retn  
  }  
  
  
5.8 FT\_Prefetch\_queue\_nop1  
这个反调试是在<<ANTI-UNPACKER TRICKS>>中给出的，它主要是基于 REP 指令，通过 REP 指令来修改自身代码，在非调试态下，计算机会将该指令完整取过来，因此可以正确的执行 REP 这个指令，将自身代码完整修改，但在调试态下，则在修改自身的时候立即跳出。  
这个反跟踪技巧个人觉得用处不大，因为只有在 REP 指令上使用 F7 单步时，才会触发这个反跟踪，而我个人在碰到 REP 时，通常都是 F8 步过。下面是利用这个 CPU 预取指令的特性的实现反跟踪的一种方法，正常情况下，REP 指令会修改其后的跳转指令，进入正常的程序流程，但在调试态下，其无法完成对其后代码的修改，从而实现反调试。  
  
   DWORD oldProtect;  
   DWORD tmpProtect;  
\_\_asm  
   {  
    lea eax,dword ptr\[oldProtect\]  
    push eax  
    push 0x40  
    push 0x10  
    push offset label3;  
    call dword ptr \[VirtualProtect\];  
    nop  
label3:  
    mov al,0x90  
    push 0x10  
    pop ecx  
    mov edi,offset label3  
    rep stosb  
    jmp rt\_label  
    nop  
    nop  
    nop  
    nop  
    nop  
rf\_label:  
    ;write back  
    mov dword ptr\[label3\],0x106a90b0  
    mov dword ptr\[label3+0x4\],0x205CBF59  
    mov dword ptr\[label3+0x8\],0xAAF30040  
    mov dword ptr\[label3+0xc\],0x90909090  
    mov dword ptr\[label3+0x6\],offset label3  
    lea eax, dword ptr\[tmpProtect\];  
    ;restore protect  
    push eax  
    push oldProtect  
    push 0x10  
    push offset label3;  
    call dword ptr \[VirtualProtect\];  
  
    xor eax,eax  
    mov esp,ebp  
    pop ebp  
    retn  
rt\_label:  
    ;write back  
    mov dword ptr\[label3\],0x106a90b0  
    mov dword ptr\[label3+0x4\],0x205CBF59  
    mov dword ptr\[label3+0x8\],0xAAF30040  
    mov dword ptr\[label3+0xc\],0x90909090  
    mov dword ptr\[label3+0x6\],offset label3  
    lea eax, dword ptr\[tmpProtect\];  
    ;restore protect  
    push eax  
    push oldProtect  
    push 0x10  
    push offset label3;  
    call dword ptr \[VirtualProtect\];  
  
    xor eax,eax  
    inc eax  
    mov esp,ebp  
    pop ebp  
    retn  
  }  
  
  
5.9 FT\_Prefetch\_queue\_nop2  
  与 5.8 节类似，这是根据 CPU 预取指令的这个特性实现的另一种反跟踪技巧。原理是通过检测 REP 指令后的 ECX 值，来判断 REP 指令是否被完整执行。在正常情况下，REP 指令完整执行后，ECX 值应为 0；但在调试态下，由于 REP 指令没有完整执行，ECX 值为非 0 值。通过检测 ECX 值，实现反跟踪。  
  DWORD oldProtect;  
  DWORD tmpProtect;  
\_\_asm  
  {  
    lea eax,dword ptr\[oldProtect\]  
    push eax  
    push 0x40  
    push 0x10  
    push offset label3;  
    call dword ptr \[VirtualProtect\];  
    mov ecx,0  
label3:  
    mov al,0x90  
    push 0x10  
    pop ecx  
    mov edi,offset label3  
    rep stosb  
    nop  
    nop  
    nop  
    nop  
    nop  
    nop  
    push ecx  
    ;write back  
    mov dword ptr\[label3\],0x106a90b0  
    mov dword ptr\[label3+0x4\],0x201CBF59  
    mov dword ptr\[label3+0x8\],0xAAF30040  
    mov dword ptr\[label3+0xc\],0x90909090  
    mov dword ptr\[label3+0x6\],offset label3  
    lea eax, dword ptr\[tmpProtect\];  
    ;restore protect  
    push eax  
    push oldProtect  
    push 0x10  
    push offset label3;  
    call dword ptr \[VirtualProtect\];  
    pop ecx  
  
    test ecx,ecx  
    jne rt\_label  
  }  
rf\_label:  
  return false;  
rt\_label:  
  return true;  
  
六、  检测-补丁(FP\_)  
这部分内容也较少，方法当然也有很多种，原理都差不多，我只选了下面三种。这几种方法通常在一些壳中较常用，用于检验文件是否被脱壳或被恶意修改。  
函数列表如下：  
//find Patch  
bool FP\_Check\_FileSize(DWORD Size);  
bool FP\_Check\_FileHashValue\_CRC(DWORD CRCVALUE\_origin);  
bool FP\_Check\_FileHashValue\_MD5(DWORD MD5VALUE\_origin);  
  
6.1 FP\_Check\_FileSize(DWORD Size)  
  通过检验文件自身的大小的方法，是一种比较简单的文件校验方法，通常如果被脱壳，或被恶意修改，就可能影响到文件的大小。我用下面的代码实现。需注意的是，文件的大小要先编译一次，将首次编译得到的数值写入代码，再重新编译完成。  
  DWORD Current\_Size;  
  TCHAR szPath\[MAX\_PATH\];  
  HANDLE hFile;  
  
if( !GetModuleFileName( NULL,szPath, MAX\_PATH ) )  
return FALSE;  
  
  hFile = CreateFile(szPath,   
    GENERIC\_READ ,  
    FILE\_SHARE\_READ,   
    NULL,  
    OPEN\_ALWAYS,   
    FILE\_ATTRIBUTE\_NORMAL,   
    NULL);  
if (hFile == INVALID\_HANDLE\_VALUE)  
returnfalse;  
  Current\_Size=GetFileSize(hFile,NULL);  
  CloseHandle(hFile);  
if(Current\_Size!=Size)  
returntrue;  
returnfalse;  
  
6.2 FP\_Check\_FileHashValue\_CRC  
  检验文件的 CRC 数值，是比较常用的文件校验方法，相信很多人都碰到过了，我是在《软件加解密技术》中了解到的。需注意的是文件原始 CRC 值的获得，及其放置位置，代码编写完成后，通常先运行一遍程序，使用调试工具获得计算得到的数值，在将这个数值写入文件中，通常这个数值不参加校验，可以放置在文件的尾部作为附加数据，也可以放在 PE 头中不用的域中。  
  下面的代码只是个演示，没有保存 CRC 的真实数值，也没有单独存放。  
  
  DWORD fileSize,NumberOfBytesRW;  
  DWORD CRCVALUE\_current;  
  TCHAR szFileName\[MAX\_PATH\];   
  TCHAR  \*pBuffer ;  
  GetModuleFileName(NULL,szFileName,MAX\_PATH);  
  HANDLE hFile = CreateFile(  
    szFileName,  
    GENERIC\_READ,  
    FILE\_SHARE\_READ,   
    NULL,  
    OPEN\_EXISTING,  
    FILE\_ATTRIBUTE\_NORMAL,  
    NULL);  
if (hFile != INVALID\_HANDLE\_VALUE )  
  {  
    fileSize = GetFileSize(hFile,NULL);  
if (fileSize == 0xFFFFFFFF) returnfalse;  
    pBuffer = new TCHAR \[fileSize\];    
    ReadFile(hFile,pBuffer, fileSize, &NumberOfBytesRW, NULL);  
    CloseHandle(hFile);  
  }  
  CRCVALUE\_current=CRC32((BYTE \*)pBuffer,fileSize);  
if(CRCVALUE\_origin!=CRCVALUE\_current)  
returntrue;  
returnfalse;  
  
6.3 FP\_Check\_FileHashValue\_MD5  
与 6.2 节的原理相同，只是计算的是文件的 MD5 数值。仍要注意 6.2 节中同样的 MD5 真实数值的获得和存放问题。  
  
未完。
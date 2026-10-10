

# PhantOm 驱动浅析

> 原文：[`www.pediy.com/kssd/pediy10/100983.html`](https://www.pediy.com/kssd/pediy10/100983.html)

\[目录\]  
  
一. 前言  
二. PhantOm 在驱动层的保护方式  
三. 挂钩的 SSDT 函数  
四. 挂钩的 SSDT Shadow 函数  
五. 结语  
  
\[内容\]  
  
一. 前言  
  
相信脱壳和破解界的朋友们大部分都用过 PhantOm 这个隐藏 OD 的插件，以前可以说是过 TMD 这  
种加密壳的利器，它可以在应用层和驱动层对 OD 进行保护。貌似应用层的保护方式已经给某  
强人分析过了，这里就不再重复了，现在分析一下 PhantOm 的驱动保护。拿来开刀的是  
PhantOm V1.54 版本的。  
  
PhantOm 插件的驱动文件在 PhantOm.dll 的资源中，在 OD 运行时自动从资源中释放驱动到临时  
文件夹中，我们可以通过 PE\_Stud 等 PE 工具进行提取。  
  
二. PhantOm 的驱动层保护方式  
  
当一个程序被调试，驱动将会将 OD 和被调试程序的相关信息存储在一个结构中，并以双链表  
的形式存放。其结构如下：  

代码:

```
typedef struct \_DEBUG\_INFO
{
  struct \_DEBUG\_INFO \*Prev;
  struct \_DEBUG\_INFO \*Next;
  PLONG  Debugger\_Id;  // OD 的 id 号
  PEPROCESS  Debugger\_Eprocess;  // OD 的 EPROCESS 结构
  PEPROCESS  Debuggee\_Eprocess;  // 被调试程序的 EPROCESS 结构
} DEBUG\_INFO, \*PDEBUG\_INFO;

```

此结构非常重要，驱动中主要就是查询此结构来判断是否对 OD 进程进行保护。  
  
当用 OD 加载或重新运行一个程序的时候，应用层将会传递 OD 和被调试程序的 id 号到驱动层中，  
此时驱动程序将会查询双链表看相关的结构是否已经存在，不存在这创建一个节点并加入到  
双链表中。  
代码:

```
VOID SetDebugInfo (PVOID DebugIdBuf)
{
  NTSTATUS status;
  PEPROCESS pEprocess;
  PDEBUG\_INFO NodeAddr, LastNode;
  ULONG Debugger\_id, Debuggee\_id;
  Debugger\_id = \*(PULONG)DebugIdBuf;
  Debuggee\_id = \*(PULONG)((ULONG)DebugIdBuf+1);
  if (DebugIdBuf == NULL)
    return;
  
  // 判断相关结构是否已经存在
  if ( !GetDebugInfoByPid(\*(PULONG)DebugIdBuf)
  {
    // 建立 OD 与被调试程序的关联
    // 传入被调试程序的 id
    status = PsLookupProcessByProcessId(Debuggee\_id), 
      EprocessOfDebuggee);
    if (status == STATUS\_SUCCESS)
    {
      Debug\_Info.Debuggee\_Eprocess = EprocessOfDebuggee;
      ObDereferenceObject(EprocessOfDebuggee);
    }
  }
  else   // 相关结构不存在
  {
    // 为节点分配空间
    NodeAddr = (PDEBUG\_INFO)ExAllocatePool(PagedPool, sizeof(DEBUG\_INFO));
    if (NodeAddr == NULL)
      return;
      
    RtlZeroMemory(NodeAddr, 16);
    // 查找链表最后一个节点
    LastNode = GetLast();
    if (LastNode)
    {
      // 非空链表, 插入到表尾
      LastNode->Next = NodeAddr;
      NodeAddr->Prev = LastNode;
    }
    else
      // 空链表，插入到表头
      DebugInfoHeader = NodeAddr;
    // 填充链表结构
    NodeAddr->Debugger\_Id = Debugger\_id;
    if (STATUS\_SUCCESS == PsLookupProcessByProcessId(Debugger\_id, pEprocess))
    {
      NodeAddr->Debugger\_Eprocess = pEprocess;
      ObDereferenceObject(pEprocess);
    }
    if (STATUS\_SUCCESS == PsLookupProcessByProcessId(Debuggee\_id, pEprocess))
    {
      NodeAddr->Debuggee\_Eprocess = pEprocess;
      ObDereferenceObject(pEprocess);
    }
  }
}

```

  
三. 挂钩的 SSDT 函数  
PhantOm 一共挂钩了 9 个与 anti-debug 相关的 SSDT 函数，分别列举如下：  
  
(1) NtQuerySystemInformation  
此函数的第一个参数 SystemInformationClass 是一个类型信息，可由它指定我们所查询的系统  
信息。在 anti-debug 领域主要关注 3 种类型，分别是 SystemProcessesAndThreadsInformation，  
可通过此参数枚举所有进程；SystemKernelDebuggerInformation，是否存在 windbg 等内核调  
试器；SystemHandleInformation，可用于枚举进程句柄。  
所以我们需要 hook 这个函数，对以上三种情况进行处理。  
代码:

```
NTSTATUS WINAPI New\_NtQuerySystemInformation(
  \_\_in       SYSTEM\_INFORMATION\_CLASS SystemInformationClass,
  \_\_inout    PVOID SystemInformation,
  \_\_in       ULONG SystemInformationLength,
  \_\_out\_opt  PULONG ReturnLength
  )
{
  NTSTATUS status;
  PEPROCESS cur\_eprocess;
  PSYSTEM\_PROCESSES system\_process;
  PSYSTEM\_PROCESSES system\_process\_prev;
  PSYSTEM\_HANDLE\_INFORMATION\_EX handle\_info;
  ULONG NextEntryDelta;
  ULONG handle\_count, index, total\_size;
  status = Org\_NtQuerySystemInformation(
    SystemInformationClass,
    SystemInformation,
    SystemInformationLength,
    ReturnLength);
  if (status != STATUS\_SUCCESS | IS\_HOOK != TRUE)
    return status;

  cur\_eprocess = IoGetCurrentProcess();
  // 当前进程是 OD
  if (FindDebugInfoByEprocess(cur\_eprocess))
    return status;

  switch (SystemInformationClass)
  {
  case SystemProcessesAndThreadsInformation:  // 5
    {
      system\_process = (PSYSTEM\_PROCESSES)SystemInformation;
      system\_process\_prev = system\_process;
      NextEntryDelta = 0;
      do 
      {
        system\_process = (PSYSTEM\_PROCESSES)((ULONG)system\_process + NextEntryDelta);
        if (GetDebugInfoByPid(system\_process->ProcessId)) // 下面断链
        {
          // 非最后一项
          if (system\_process->NextEntryDelta)
            system\_process\_prev->NextEntryDelta += system\_process->NextEntryDelta;
          // 最后一项
          else
            system\_process\_prev->NextEntryDelta = 0;
          // 抹掉进程名字信息
          RtlZeroMemory(system\_process->ProcessName.Buffer, 
            system\_process->ProcessName.Length\*2);
          // 然后将整个 SYSTEM\_PROCESS 抹去
          total\_size = system\_process->ThreadCount << 6 + 0xB4;
          RtlZeroMemory(&system\_process, total\_size);
        }
        system\_process\_prev = system\_process;
        NextEntryDelta = system\_process->NextEntryDelta;
      } while (NextEntryDelta);
      system\_process = (PSYSTEM\_PROCESSES)SystemInformation;
      NextEntryDelta = 0;
      do 
      {
        system\_process = (PSYSTEM\_PROCESSES)((ULONG)system\_process + NextEntryDelta);
        if (GetDebugInfoByPid(system\_process->InheritedFromProcessId))
          // 如果父进程是 OD，则将父进程的 ID 改为 explorer.exe 的 ID 号
          system\_process->InheritedFromProcessId = EXPLORER\_ID;
        NextEntryDelta = system\_process->NextEntryDelta;
      } while (NextEntryDelta);
    }
    break;

  case SystemKernelDebuggerInformation:  // 35
    RtlZeroMemory(SystemInformation, SystemInformationLength);
    break;

  case SystemHandleInformation:  // 16
    {
      handle\_info = (PSYSTEM\_HANDLE\_INFORMATION\_EX)SystemInformation;
      handle\_count = handle\_info->NumberOfHandles;
      index = 0;
      while (handle\_count)
      {
        if (GetDebugInfoByPid(handle\_info->Information\[index\].ProcessId))
          handle\_info->Information\[index\].ProcessId = EXPLORER\_ID;
        handle\_count--;
        index++;
      }
    }
    break;
  }

  return STATUS\_SUCCESS;
}

```

(2) NtOpenProcess  
挂钩此函数防止程序打开 OD 和 csrss.exe 的句柄  
代码:

```
NTSTATUS  New\_NtOpenProcess (    
             \_\_out PHANDLE  ProcessHandle,    
             \_\_in ACCESS\_MASK  DesiredAccess,    
             \_\_in POBJECT\_ATTRIBUTES  ObjectAttributes,    
             \_\_in\_opt PCLIENT\_ID  ClientId    
             )
{
  NTSTATUS status;
  if (IS\_HOOK == TRUE && !FindDebugInfoByEprocess(IoGetCurrentProcess()))
  {
    status = STATUS\_INVALID\_PARAMETER;
    if (!MmIsAddressValid(ClientId) ||
      ClientId.UniqueProcess == CSRSS\_ID ||
      GetDebugInfoByPid(ClientId->UniqueProcess))
      return status;
  }
  return Org\_NtOpenProcess(ProcessHandle,
               DesiredAccess,
               ObjectAttributes,
               ClientId);
}

```

(3) NtSetInformationThread  
在程序保护中，当参数 ThreadInformationClass 的值为 ThreadHideFromDebugger(0x11)时，  
此函数可以用来防止调试事件被发往调试器。  
代码:

```
NTSTATUS
NTAPI
New\_NtSetInformationThread(
               IN HANDLE               ThreadHandle,
               IN THREAD\_INFORMATION\_CLASS ThreadInformationClass,
               IN PVOID                ThreadInformation,
               IN ULONG                ThreadInformationLength 
               )
{
  NTSTATUS status;
  PVOID Object;
  if (IS\_HOOK == TRUE)
  {
    status = ObReferenceObjectByHandle(ThreadHandle, 0, 0, KernelMode, Object, 0);
    if (status != STATUS\_SUCCESS)
      return STATUS\_INVALID\_HANDLE;
    ObDereferenceObject(Object);
    if (!FindDebugInfoByEprocess(IoGetCurrentProcess()) &&
      ThreadInformationClass == ThreadHideFromDebugger)
      return STATUS\_SUCCESS;
  }
  return Org\_NtSetInformationThread(ThreadHandle,
                    ThreadInformationClass,
                    ThreadInformation,
                    ThreadInformationLength);
}

```

(4) NtClose  
当进程被调试, 使用一个无效的句柄调用 NtClose 将会产生一个 STATUS\_INVALID\_HANDLE  
 (0xC0000008) 异常。  
代码:

```
NTSTATUS 
NTAPI
New\_NtClose(HANDLE ObjectHandle)
{
  PVOID Object;
  NTSTATUS status;
  if (IS\_HOOK == FALSE)
    return Org\_NtClose(ObjectHandle);
  // 防止使用无效句柄来检测调试器
  InterlockedIncrement(Lock\_Number);
  status = ObReferenceObjectByHandle(ObjectHandle, 0, 0, 0, Object, 0);
  if (status == STATUS\_SUCCESS)
  {
    ObDereferenceObject(Object);
    status = Org\_NtClose(ObjectHandle);
  }
  else
    status = STATUS\_SUCCESS;
  InterlockedDecrement(Lock\_Number);
  return status;
}

```

(5) NtYieldExecution  
此函数 hook 来做什么用的？不明真相中，那位能人异士知道的话恳求相告，感激不尽。  
直接翻译代码  
代码:

```
NTSTATUS New\_NtYieldExecution()
{
  Org\_NtYieldExecution();
  return STATUS\_NO\_YIELD\_PERFORMED;
}

```

(6) NtQueryInformationProcess  
此函数用于返回目标进程的各类信息。在软件保护中，参数 ProcessInformationClass  
有几个值必须特殊对待：  
ProcessBasicInformation(0x0)--可检测目标进程的父进程，如果程序被调试，则  
父进程是调试器，而对应一般窗口程序，其父进程是 explorer.exe。  
ProcessDebugPort(0x07) -- 如果目标进程正在被调试，系统会为进程分配一个调试  
端口。通过此参数调用 NtQueryInformationProcess 则返回调试端口号，返回 0 表示当前  
无调试器附在进程上。  
ProcessDebugFlags(0x1f) -- 此时函数返回 EPROCESS->NoDebugInherit 域的值。为 0  
表示进程正处于调试状态。  
代码:

```
NTSTATUS WINAPI New\_NtQueryInformationProcess(
  \_\_in       HANDLE ProcessHandle,
  \_\_in       PROCESSINFOCLASS ProcessInformationClass,
  \_\_out      PVOID ProcessInformation,
  \_\_in       ULONG ProcessInformationLength,
  \_\_out\_opt  PULONG ReturnLength
)
{
  NTSTATUS status;
  PPROCESS\_BASIC\_INFORMATION ProcessBasicInfo;
  PEPROCESS pEprocess;
  status = Org\_NtQueryInformationProcess(ProcessHandle,
    ProcessInformationClass,
    ProcessInformation,
    ProcessInformationLength,
    ReturnLength)
  if (STATUS\_SUCCESS != status)
    return status;
  if (!FindDebugInfoByEprocess(IoGetCurrentProcess()))
  {
    // 调用此函数的进程不是 OD
    if (ProcessInformationClass == ProcessBasicInformation)
    {
      // BrocessBasicInformation(0)
      ProcessBasicInfo = (PROCESS\_BASIC\_INFORMATION)ProcessInformation;
      // 检查此句柄的父进程是否为 OD
      if (GetDebugInfoByPid(ProcessBasicInfo->InheritedFromUniqueProcessId))
        // 对于带窗口类进程，其父进程一般为 explorer.exe，如果程序
        // 被调试，则其父进程为调试器。
        ProcessBasicInfo->InheritedFromUniqueProcessId = EXPLORER\_ID;
      // 此句柄所属的进程是否为 OD
      else if (GetDebugInfoByPid(ProcessBasicInfo->UniqueProcessId))
      {
        // 返回的信息清 0
        RtlZeroMemory(ProcessInformation, sizeof(PROCESS\_BASIC\_INFORMATION));
        if (ReturnLength != NULL)
          // 返回长度置 0
          \*ReturnLength = 0;
      }
    }
    else
    {
      // 根据句柄得到进程对象
      if (ObReferenceObjectByHandle(ProcessHandle,
        0, 0, KernelMode, pEprocess, NULL) == STATUS\_SUCCESS)
      {
        // 判断此进程是否是被调试
        if (GetDebuggeeByEprocess(pEprocess))
        {
          if (ProcessInformationClass == ProcessDebugPort 
            || ProcessInformationClass == ProcessDebugObjectHandle)
            //  调试端口信息或调试句柄清 0
            \*ProcessInformation = 0;
          // 它将返回 EPROCESS->NoDebugInherit 的值，
          // 当调试器存在的时候，其值为 FALSE，表示
          // 进程正在被调试
          else if (ProcessInformationClass == ProcessDebugFlags)
          {
            if (ProcessInformation != NULL)
              \*(PDWORD)(ProcessInformation) = TRUE;
          }
        }
        ObDereferenceObject(pEprocess);
      }
    }
  }
  return status;
}

```

(7) NtQueryObject  
每当一个应用程序被调试的时候，将会为调试对话在内核中创建一个 DebugObject 类型的对象。  
程序可以检查 DebugObject10 类型内核对象的数量来确定是否有调试器的存在。DebugObject 的  
数量可以通过 NtQueryObject 函数来检索所有对象类型的信息获得，此时需要将  
ObjectInformationClass 的参数设为 ObjectAllTypeInformation(0x03)。PhantOm 在内核中的  
解决方法是 hook 此函数，如果发现检索的是 ObjectAllTypeInfomation 类型则抹掉名称为  
"DebugObject"的调试对象的相关信息。  
代码:

```
NTSTATUS New\_NtQueryObject(
               \_\_in\_opt   HANDLE Handle,
               \_\_in       OBJECT\_INFORMATION\_CLASS ObjectInformationClass,
               \_\_out\_opt  PVOID ObjectInformation,
               \_\_in       ULONG ObjectInformationLength,
               \_\_out\_opt  PULONG ReturnLength
               )
{
  NTSTATUS status;
  POBJECT\_ALL\_INFORMATION pObjectAllInfo;
  ULONG NumObjects;
  status = Org\_NtQueryObject(Handle,
                 ObjectInformationClass,
                 ObjectInformation,
                 ObjectInformationLength,
                 ReturnLength);
  if (IS\_HOOK == FALSE || ObjectInformation == NULL)
    return status;

  //判断当前进程是否为 OD
  if (!FindDebugInfoByEprocess(IoGetCurrentProcess()))
    return status;

  switch (ObjectInformationClass)
  {
  case ObjectTypeInformation:
    // 把返回信息全部清 0，一了百了
    RtlZeroMemory(ObjectInformation, ObjectInformationLength);
    break;

    // 如果是检索所以对象类型信息
  case ObjectAllInformation:
    {
      pObjectAllInfo = (POBJECT\_ALL\_INFORMATION)ObjectInformation;
      PUCHAR pObjInfo = (PUCHAR)pObjectAllInfo->ObjectTypeInformation;
      NumObjects = pObjectAllInfo->NumberOfObjectsTypes;
      for (UINT i = 0; i < NumObjects; i++)
      {
        POBJECT\_TYPE\_INFORMATION pObjectTypeInfo = 
          (POBJECT\_TYPE\_INFORMATION)pObjInfo;
        ULONG TypeNameLength = pObjectTypeInfo->TypeName.Length;
        PUCHAR TypeNameBuffer = pObjectTypeInfo->TypeName.Buffer;

        // 检查 debug object 是否存在
        if (TypeNameLength == 0x16)
        {
          if (wcscmp(L"DebugObject", pObjectTypeInfo->TypeName.Buffer) == 0)
            \*pObjectTypeInfo = NULL;
        }

        pObjInfo = TypeNameBuffer;
        // 加上字符串的长度
        pObjInfo += TypeNameLength;
        // 双字节对齐
        ULONG tmp = (ULONG)pObjInfo & 0xFFFFFFFC;
        pObjInfo = ((PUCHAR)tmp + sizeof(ULONG));
      }
    }
    break;
  }
}

```

(8) NtSetContextThread  
反调试程序可能会通过此函数获得并修改 CPU 中调试寄存器的内容，所以如果程序下了  
硬件断点的话就会失效，并且可以检测到调试器的存在。我们可以检查 ContextFlags  
中的调试寄存器组的标志位，看程序是否在查询调试寄存器。  
代码:

```
NTSTATUS
NTAPI
New\_NtSetContextThread(IN HANDLE ThreadHandle,
             IN PCONTEXT Context)
{
  if (IS\_HOOK == TRUE)
  {
    if (!FindDebugInfoByEprocess(IoGetCurrentProcess()) && 
      MmIsAddressValid(Context))
      // 清除 ContextFlags 中查询调试寄存器组的标志位
      Context->ContextFlags & ~CONTEXT\_DEBUG\_REGISTERS;
  }
  return Org\_NtSetContextThread(ThreadHandle, Context);
}

```

(9) NtQueryInformationThread  
查询线程信息，可以由此获得线程所对应的进程  
  
代码:

```
NTSTATUS NTAPI New\_NtQueryInformationThread(
  IN HANDLE               ThreadHandle,
  IN THREAD\_INFORMATION\_CLASS ThreadInformationClass,
  OUT PVOID               ThreadInformation,
  IN ULONG                ThreadInformationLength,
  OUT PULONG              ReturnLength OPTIONAL 
  )
{
  NTSTATUS status;
  PTHREAD\_BASIC\_INFORMATION ThreadBasicInfo;
  status = Org\_NtQueryInformationThread(ThreadHandle,
    ThreadInformationClass,
    ThreadInformation,
    ThreadInformationLength,
    ReturnLength);

  if (status == STATUS\_SUCCESS && IS\_HOOK == 1)
  {
    ThreadBasicInfo = (PTHREAD\_BASIC\_INFORMATION)ThreadInformation;
    // 获得线程对应的进程 ID，并判断是否在查询 OD 进程
    if (GetDebugInfoByPid(ThreadBasicInfo->ClientId.UniqueProcess))
    {
      RtlZeroMemory(ThreadInformation, sizeof(THREAD\_BASIC\_INFORMATION));
      if (ReturnLength != NULL)
        \*ReturnLength = 0;
      status = STATUS\_ACCESS\_DENIED;
    }
  }

  return status;
}

```

  
四. 挂钩的 SSDT Shadow 函数  
PhantOm 挂钩了 4 个 SSDT Shadow 函数，主要用于防止程序检测到 OD 的窗口  
  
(1) NtUserGetForegroundWindow  
此函数返回用户正在工作的窗口的句柄，如果程序被调试，则顶层窗口是 OD 的窗口。  
代码:

```
ULONG New\_NtUserGetForegroundWindow(VOID)
{
  NTSTATUS status;
  status = Org\_NtUserGetForegroundWindow();

  if (IS\_HOOK == TRUE)
  {
    // 判断所查询的窗口是否为 OD
    if (GetDebugInfoByPid(status))
    {
      // 判断当前进程是否为 OD
      if (!FindDebugInfoByEprocess(IoGetCurrentProcess()))
        status = LastForegroundWindow;
      else
        LastForegroundWindow = status;
    }
  }

  return status;
}

```

(2) NtUserQueryWindow  
查询给定窗口句柄所属的进程  
代码:

```
INT\_PTR New\_NtUserQueryWindow(
                IN ULONG WindowHandle,
                IN ULONG TypeInformation)
{
  ULONG ProcessID;

  // 查询窗口句柄所属的进程
  ProcessID = Org\_NtUserQueryWindow(WindowHandle, 0);

  if (ProcessID && IS\_HOOK == TRUE)
  {
    // 查询窗口句柄是否属于 OD
    if(!GetDebugInfoByPid(ProcessID))
    {
      if (!FindDebugInfoByEprocess(IoGetCurrentProcess()))
        return 0;
    }
  }

  return ProcessID;
}

```

(3) NtUserBuildHwndList  
此函数用于枚举桌面上所有窗口的句柄，所以在 hook 程序中，我们必须在返回值结构中把  
OD 所属的句柄抹掉。  
代码:

```
NTSTATUS New\_NtUserBuildHwndList(
                   IN HDESK hdesk, 
                   IN HWND hwndNext, 
                   IN ULONG fEnumChildren, 
                   IN DWORD idThread, 
                   IN UINT cHwndMax, 
                   OUT HWND \*phwndFirst, 
                   OUT ULONG\* pcHwndNeeded)
{
  NTSTATUS status;
  ULONG i, ProcessId;
  if (fEnumChildren == 1 && IS\_HOOK == TRUE)
  {
    ProcessId = Org\_NtUserQueryWindow((ULONG)hwndNext, 0);
    if (GetDebugInfoByPid(ProcessId))
    {
      if (!FindDebugInfoByEprocess(IoGetCurrentProcess())
        return STATUS\_UNSUCCESSFUL;
    }
  }

  status = Org\_NtUserBuildHwndList(hdesk, 
                   hwndNext, 
                   fEnumChildren, 
                   idThread, 
                   cHwndMax, 
                   phwndFirst, 
                   pcHwndNeeded);

  if (status == STATUS\_SUCCESS)
  {
    if (IS\_HOOK == 1 && !FindDebugInfoByEprocess(IoGetCurrentProcess()))
    {
      i = 0;
      while (i < \*pcHwndNeeded)
      {
        // 获取句柄所属的进程 ID
        ProcessId = Org\_NtUserQueryWindow((ULONG)phwndFirst\[i\], 0);
        // 判断句柄是否属于 OD
        if (GetDebugInfoByPid(ProcessId))
        {
          // 将此后的句柄信息前移，覆盖掉 OD 的句柄信息
          RtlCopyMemory(&phwndFirst\[i+1\], &phwndFirst\[i\], \*pcHwndNeeded-i);
          // 最后一项清 0
          phwndFirst\[\*pcHwndNeeded-1\] = 0;
          // 总数减 1
          (\*pcHwndNeeded)--;
          continue; 
        }
        i++;
      }
    }
  }

  return status;
}

```

(4) NtUserFindWindowEx  
此函数返回由窗口名或窗口类标识的窗口句柄。如果查询的是 OD 窗口，则返回 0  
代码:

```
NTSTATUS New\_NtUserFindWindowEx(
                  IN HWND hwndParent, 
                  IN HWND hwndChild, 
                  IN PUNICODE\_STRING pstrClassName OPTIONAL, 
                  IN PUNICODE\_STRING pstrWindowName OPTIONAL, 
                  IN DWORD dwType)
{
  NTSTATUS status;

  status = Org\_NtUserFindWindowEx(hwndParent, hwndChild, 
    pstrClassName, pstrWindowName, dwType);

  if (status == 0 || IS\_HOOK == FALSE)
    return status;

  // 查询此窗口所属的进程的 ID
  ProcessId = Org\_NtUserQueryWindow(status, 0);
  // 判断窗口是否属于 OD
  if (GetDebugInfoByPid(ProcessId))
  {
    // 当前进程是否为 OD
    if (!FindDebugInfoByEprocess(IoGetCurrentProcess()))
      return 0;
  }

  return status;
}

```

五. 结语  
很多朋友问 PhantOm 可不可以调 NP 或 TexSafe 保护的游戏，看看上面的程序就一目了然了，  
很明显不能。PhantOm 在驱动层只是做了很基本的服务函数表的 hook，用很简单的 ssdt 表  
还原它的功能就消失了。而 NP 这些猥琐东西不但 inline hook 了 n 个函数，还会采用各种  
手段检测游戏运行中有无异常情况发生，如果有它觉得不爽的情况就给你一个 BSOD。  
但是用它来对付运行在 ring3 层的程序也绰绰有余了。  
  
上面的代码均未经过测试，直接看汇编代码翻译过来的，有错误是必然的。偶很懒，  
而且 BSOD 的话会很心疼偶滴电脑，想想还是不调代码了。如果同志们计划打造一个山寨版  
的 PhantOm 驱动，最好经过严格修改并加上适当的错误处理，不然你的电脑会蓝得很难看。  
  
  
附件是 IDA 文件，加了详细的注解，这个逆得比较彻底，100%全逆。  
还有很多函数未列出来，有兴趣的朋友可以根据 IDA 文件参考一下。  
  
欢迎拍砖!!!

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || PhantOm1.54.rar  
 |
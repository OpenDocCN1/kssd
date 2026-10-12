
# PhantOm 驱动浅析

> 原文：[`www.pediy.com/kssd/pediy10/100983.html`](https://www.pediy.com/kssd/pediy10/100983.html)

## 目录

一. 前言
二. PhantOm 在驱动层的保护方式
三. 挂钩的 SSDT 函数
四. 挂钩的 SSDT Shadow 函数
五. 结语

## 一. 前言

相信脱壳和破解界的朋友们大部分都用过 PhantOm 这个隐藏 OD 的插件，以前可以说是过 TMD 这种加密壳的利器，它可以在应用层和驱动层对 OD 进行保护。貌似应用层的保护方式已经给某强人分析过了，这里就不再重复了，现在分析一下 PhantOm 的驱动保护。拿来开刀的是 PhantOm V1.54 版本的。

PhantOm 插件的驱动文件在 `PhantOm.dll` 的资源中，在 OD 运行时自动从资源中释放驱动到临时文件夹中，我们可以通过 `PE_Stud` 等 PE 工具进行提取。

## 二. PhantOm 的驱动层保护方式

当一个程序被调试，驱动将会将 OD 和被调试程序的相关信息存储在一个结构中，并以双链表的形式存放。其结构如下：

```c
typedef struct _DEBUG_INFO
{
  struct _DEBUG_INFO *Prev;
  struct _DEBUG_INFO *Next;
  PLONG   Debugger_Id;  // OD 的 id 号
  PEPROCESS   Debugger_Eprocess;  // OD 的 EPROCESS 结构
  PEPROCESS   Debuggee_Eprocess;  // 被调试程序的 EPROCESS 结构
} DEBUG_INFO, *PDEBUG_INFO;
```

此结构非常重要，驱动中主要就是查询此结构来判断是否对 OD 进程进行保护。

当用 OD 加载或重新运行一个程序的时候，应用层将会传递 OD 和被调试程序的 id 号到驱动层中，此时驱动程序将会查询双链表看相关的结构是否已经存在，不存在这创建一个节点并加入到双链表中。

```c
VOID SetDebugInfo (PVOID DebugIdBuf)
{
  NTSTATUS status;
  PEPROCESS pEprocess;
  PDEBUG_INFO NodeAddr, LastNode;
  ULONG Debugger_id, Debuggee_id;
  Debugger_id = *(PULONG)DebugIdBuf;
  Debuggee_id = *(PULONG)((ULONG)DebugIdBuf+1);
  if (DebugIdBuf == NULL)
    return;
  
  // 判断相关结构是否已经存在
  if ( !GetDebugInfoByPid(*(PULONG)DebugIdBuf)
  {
    // 建立 OD 与被调试程序的关联
    // 传入被调试程序的 id
    status = PsLookupProcessByProcessId(Debuggee_id), 
      EprocessOfDebuggee);
    if (status == STATUS_SUCCESS)
    {
      Debug_Info.Debuggee_Eprocess = EprocessOfDebuggee;
      ObDereferenceObject(EprocessOfDebuggee);
    }
  }
  else   // 相关结构不存在
  {
    // 为节点分配空间
    NodeAddr = (PDEBUG_INFO)ExAllocatePool(PagedPool, sizeof(DEBUG_INFO));
    if (NodeAddr == NULL)
      return;
      
    RtlZeroMemory(NodeAddr, 16);
    // 查找链表最后一个节点
    LastNode = GetLast();
    if (LastNode)
    {
      // 非空链表, 插入到表尾
      LastNode->Next = NodeAddr;
      NodeAddr->Prev = LastNode;
    }
    else
      // 空链表，插入到表头
      DebugInfoHeader = NodeAddr;
    // 填充链表结构
    NodeAddr->Debugger_Id = Debugger_id;
    if (STATUS_SUCCESS == PsLookupProcessByProcessId(Debugger_id, pEprocess))
    {
      NodeAddr->Debugger_Eprocess = pEprocess;
      ObDereferenceObject(pEprocess);
    }
    if (STATUS_SUCCESS == PsLookupProcessByProcessId(Debuggee_id, pEprocess))
    {
      NodeAddr->Debuggee_Eprocess = pEprocess;
      ObDereferenceObject(pEprocess);
    }
  }
}
```

## 三. 挂钩的 SSDT 函数

PhantOm 一共挂钩了 9 个与 anti-debug 相关的 SSDT 函数，分别列举如下：

### (1) NtQuerySystemInformation

此函数的第一个参数 `SystemInformationClass` 是一个类型信息，可由它指定我们所查询的系统信息。在 anti-debug 领域主要关注 3 种类型，分别是 `SystemProcessesAndThreadsInformation`，可通过此参数枚举所有进程；`SystemKernelDebuggerInformation`，是否存在 windbg 等内核调试器；`SystemHandleInformation`，可用于枚举进程句柄。

所以我们需要 hook 这个函数，对以上三种情况进行处理。

```c
NTSTATUS WINAPI New_NtQuerySystemInformation(
  __in       SYSTEM_INFORMATION_CLASS SystemInformationClass,
  __inout    PVOID SystemInformation,
  __in       ULONG SystemInformationLength,
  __out_opt  PULONG ReturnLength
  )
{
  NTSTATUS status;
  PEPROCESS cur_eprocess;
  PSYSTEM_PROCESSES system_process;
  PSYSTEM_PROCESSES system_process_prev;
  PSYSTEM_HANDLE_INFORMATION_EX handle_info;
  ULONG NextEntryDelta;
  ULONG handle_count, index, total_size;
  status = Org_NtQuerySystemInformation(
    SystemInformationClass,
    SystemInformation,
    SystemInformationLength,
    ReturnLength);
  if (status != STATUS_SUCCESS | IS_HOOK != TRUE)
    return status;

  cur_eprocess = IoGetCurrentProcess();
  // 当前进程是 OD
  if (FindDebugInfoByEprocess(cur_eprocess))
    return status;

  switch (SystemInformationClass)
  {
  case SystemProcessesAndThreadsInformation:  // 5
    {
      system_process = (PSYSTEM_PROCESSES)SystemInformation;
      system_process_prev = system_process;
      NextEntryDelta = 0;
      do 
      {
        system_process = (PSYSTEM_PROCESSES)((ULONG)system_process + NextEntryDelta);
        if (GetDebugInfoByPid(system_process->ProcessId)) // 下面断链
        {
          // 非最后一项
          if (system_process->NextEntryDelta)
            system_process_prev->NextEntryDelta += system_process->NextEntryDelta;
          // 最后一项
          else
            system_process_prev->NextEntryDelta = 0;
          // 抹掉进程名字信息
          RtlZeroMemory(system_process->ProcessName.Buffer, 
            system_process->ProcessName.Length*2);
          // 然后将整个 SYSTEM_PROCESS 抹去
          total_size = system_process->ThreadCount << 6 + 0xB4;
          RtlZeroMemory(&system_process, total_size);
        }
        system_process_prev = system_process;
        NextEntryDelta = system_process->NextEntryDelta;
      } while (NextEntryDelta);
      system_process = (PSYSTEM_PROCESSES)SystemInformation;
      NextEntryDelta = 0;
      do 
      {
        system_process = (PSYSTEM_PROCESSES)((ULONG)system_process + NextEntryDelta);
        if (GetDebugInfoByPid(system_process->InheritedFromProcessId))
          // 如果父进程是 OD，则将父进程的 ID 改为 explorer.exe 的 ID 号
          system_process->InheritedFromProcessId = EXPLORER_ID;
        NextEntryDelta = system_process->NextEntryDelta;
      } while (NextEntryDelta);
    }
    break;

  case SystemKernelDebuggerInformation:  // 35
    RtlZeroMemory(SystemInformation, SystemInformationLength);
    break;

  case SystemHandleInformation:  // 16
    {
      handle_info = (PSYSTEM_HANDLE_INFORMATION_EX)SystemInformation;
      handle_count = handle_info->NumberOfHandles;
      index = 0;
      while (handle_count)
      {
        if (GetDebugInfoByPid(handle_info->Information[index].ProcessId))
          handle_info->Information[index].ProcessId = EXPLORER_ID;
        handle_count--;
        index++;
      }
    }
    break;
  }

  return STATUS_SUCCESS;
}
```

### (2) NtOpenProcess

挂钩此函数防止程序打开 OD 和 `csrss.exe` 的句柄

```c
NTSTATUS  New_NtOpenProcess (     
           __out PHANDLE  ProcessHandle,     
           __in ACCESS_MASK  DesiredAccess,     
           __in POBJECT_ATTRIBUTES  ObjectAttributes,     
           __in_opt PCLIENT_ID  ClientId     
           )
{
  NTSTATUS status;
  if (IS_HOOK == TRUE && !FindDebugInfoByEprocess(IoGetCurrentProcess()))
  {
    status = STATUS_INVALID_PARAMETER;
    if (!MmIsAddressValid(ClientId) ||
      ClientId.UniqueProcess == CSRSS_ID ||
      GetDebugInfoByPid(ClientId->UniqueProcess))
      return status;
  }
  return Org_NtOpenProcess(ProcessHandle,
           DesiredAccess,
           ObjectAttributes,
           ClientId);
}
```

### (3) NtSetInformationThread

在程序保护中，当参数 `ThreadInformationClass` 的值为 `ThreadHideFromDebugger`(0x11)时，此函数可以用来防止调试事件被发往调试器。

```c
NTSTATUS
NTAPI
New_NtSetInformationThread(
           IN HANDLE             ThreadHandle,
           IN THREAD_INFORMATION_CLASS ThreadInformationClass,
           IN PVOID              ThreadInformation,
           IN ULONG              ThreadInformationLength 
           )
{
  NTSTATUS status;
  PVOID Object;
  if (IS_HOOK == TRUE)
  {
    status = ObReferenceObjectByHandle(ThreadHandle, 0, 0, KernelMode, Object, 0);
    if (status != STATUS_SUCCESS)
      return STATUS_INVALID_HANDLE;
    ObDereferenceObject(Object);
    if (!FindDebugInfoByEprocess(IoGetCurrentProcess()) &&
      ThreadInformationClass == ThreadHideFromDebugger)
      return STATUS_SUCCESS;
  }
  return Org_NtSetInformationThread(ThreadHandle,
             ThreadInformationClass,
             ThreadInformation,
             ThreadInformationLength);
}
```

### (4) NtClose

当进程被调试, 使用一个无效的句柄调用 `NtClose` 将会产生一个 `STATUS_INVALID_HANDLE` (0xC0000008) 异常。

```c
// 代码块在此处截断
```



### (5) NtYieldExecution

此函数 hook 来做什么用的？不明真相中，那位能人异士知道的话恳求相告，感激不尽。

直接翻译代码：

```
NTSTATUS New_NtYieldExecution()
{
  Org_NtYieldExecution();
  return STATUS_NO_YIELD_PERFORMED;
}
```

### (6) NtQueryInformationProcess

此函数用于返回目标进程的各类信息。在软件保护中，参数 `ProcessInformationClass` 有几个值必须特殊对待：

- `ProcessBasicInformation`(`0x0`)：可检测目标进程的父进程，如果程序被调试，则父进程是调试器，而对应一般窗口程序，其父进程是 `explorer.exe`。
- `ProcessDebugPort`(`0x07`)：如果目标进程正在被调试，系统会为进程分配一个调试端口。通过此参数调用 `NtQueryInformationProcess` 则返回调试端口号，返回 `0` 表示当前无调试器附在进程上。
- `ProcessDebugFlags`(`0x1f`)：此时函数返回 `EPROCESS->NoDebugInherit` 域的值。为 `0` 表示进程正处于调试状态。

代码：

```
NTSTATUS WINAPI New_NtQueryInformationProcess(
  __in       HANDLE ProcessHandle,
  __in       PROCESSINFOCLASS ProcessInformationClass,
  __out      PVOID ProcessInformation,
  __in       ULONG ProcessInformationLength,
  __out_opt  PULONG ReturnLength
)
{
  NTSTATUS status;
  PPROCESS_BASIC_INFORMATION ProcessBasicInfo;
  PEPROCESS pEprocess;
  status = Org_NtQueryInformationProcess(ProcessHandle,
    ProcessInformationClass,
    ProcessInformation,
    ProcessInformationLength,
    ReturnLength)
  if (STATUS_SUCCESS != status)
    return status;
  if (!FindDebugInfoByEprocess(IoGetCurrentProcess()))
  {
    // 调用此函数的进程不是 OD
    if (ProcessInformationClass == ProcessBasicInformation)
    {
      // BrocessBasicInformation(0)
      ProcessBasicInfo = (PROCESS_BASIC_INFORMATION)ProcessInformation;
      // 检查此句柄的父进程是否为 OD
      if (GetDebugInfoByPid(ProcessBasicInfo->InheritedFromUniqueProcessId))
        // 对于带窗口类进程，其父进程一般为 explorer.exe，如果程序
        // 被调试，则其父进程为调试器。
        ProcessBasicInfo->InheritedFromUniqueProcessId = EXPLORER_ID;
      // 此句柄所属的进程是否为 OD
      else if (GetDebugInfoByPid(ProcessBasicInfo->UniqueProcessId))
      {
        // 返回的信息清 0
        RtlZeroMemory(ProcessInformation, sizeof(PROCESS_BASIC_INFORMATION));
        if (ReturnLength != NULL)
          // 返回长度置 0
          *ReturnLength = 0;
      }
    }
    else
    {
      // 根据句柄得到进程对象
      if (ObReferenceObjectByHandle(ProcessHandle,
        0, 0, KernelMode, pEprocess, NULL) == STATUS_SUCCESS)
      {
        // 判断此进程是否是被调试
        if (GetDebuggeeByEprocess(pEprocess))
        {
          if (ProcessInformationClass == ProcessDebugPort 
            || ProcessInformationClass == ProcessDebugObjectHandle)
            //  调试端口信息或调试句柄清 0
            *ProcessInformation = 0;
          // 它将返回 EPROCESS->NoDebugInherit 的值，
          // 当调试器存在的时候，其值为 FALSE，表示
          // 进程正在被调试
          else if (ProcessInformationClass == ProcessDebugFlags)
          {
            if (ProcessInformation != NULL)
              *(PDWORD)(ProcessInformation) = TRUE;
          }
        }
        ObDereferenceObject(pEprocess);
      }
    }
  }
  return status;
}
```

### (7) NtQueryObject

每当一个应用程序被调试的时候，将会为调试对话在内核中创建一个 `DebugObject` 类型的对象。

程序可以检查 `DebugObject` 类型内核对象的数量来确定是否有调试器的存在。`DebugObject` 的数量可以通过 `NtQueryObject` 函数来检索所有对象类型的信息获得，此时需要将 `ObjectInformationClass` 的参数设为 `ObjectAllTypeInformation`(`0x03`)。`PhantOm` 在内核中的解决方法是 hook 此函数，如果发现检索的是 `ObjectAllTypeInfomation` 类型则抹掉名称为 `"DebugObject"` 的调试对象的相关信息。

代码：

```
NTSTATUS New_NtQueryObject(
          __in_opt   HANDLE Handle,
          __in       OBJECT_INFORMATION_CLASS ObjectInformationClass,
          __out_opt  PVOID ObjectInformation,
          __in       ULONG ObjectInformationLength,
          __out_opt  PULONG ReturnLength
          )
{
  NTSTATUS status;
  POBJECT_ALL_INFORMATION pObjectAllInfo;
  ULONG NumObjects;
  status = Org_NtQueryObject(Handle,
                    ObjectInformationClass,
                    ObjectInformation,
                    ObjectInformationLength,
                    ReturnLength);
  if (IS_HOOK == FALSE || ObjectInformation == NULL)
    return status;

  //判断当前进程是否为 OD
  if (!FindDebugInfoByEprocess(IoGetCurrentProcess()))
    return status;

  switch (ObjectInformationClass)
  {
  case ObjectTypeInformation:
    // 把返回信息全部清 0，一了百了
    RtlZeroMemory(ObjectInformation, ObjectInformationLength);
    break;

    // 如果是检索所以对象类型信息
  case ObjectAllInformation:
    {
      pObjectAllInfo = (POBJECT_ALL_INFORMATION)ObjectInformation;
      PUCHAR pObjInfo = (PUCHAR)pObjectAllInfo->ObjectTypeInformation;
      NumObjects = pObjectAllInfo->NumberOfObjectsTypes;
      for (UINT i = 0; i < NumObjects; i++)
      {
        POBJECT_TYPE_INFORMATION pObjectTypeInfo = 
          (POBJECT_TYPE_INFORMATION)pObjInfo;
        ULONG TypeNameLength = pObjectTypeInfo->TypeName.Length;
        PUCHAR TypeNameBuffer = pObjectTypeInfo->TypeName.Buffer;

        // 检查 debug object 是否存在
        if (TypeNameLength == 0x16)
        {
          if (wcscmp(L"DebugObject", pObjectTypeInfo->TypeName.Buffer) == 0)
            *pObjectTypeInfo = NULL;
        }

        pObjInfo = TypeNameBuffer;
        // 加上字符串的长度
        pObjInfo += TypeNameLength;
        // 双字节对齐
        ULONG tmp = (ULONG)pObjInfo & 0xFFFFFFFC;
        pObjInfo = ((PUCHAR)tmp + sizeof(ULONG));
      }
    }
    break;
  }
}
```

### (8) NtSetContextThread

反调试程序可能会通过此函数获得并修改 CPU 中调试寄存器的内容，所以如果程序下了硬件断点的话就会失效，并且可以检测到调试器的存在。我们可以检查 `ContextFlags` 中的调试寄存器组的标志位，看程序是否在查询调试寄存器。

代码：

```
NTSTATUS
NTAPI
New_NtSetContextThread(IN HANDLE ThreadHandle,
             IN PCONTEXT Context)
{
  if (IS_HOOK == TRUE)
  {
    if (!FindDebugInfoByEprocess(IoGetCurrentProcess()) && 
      MmIsAddressValid(Context))
      // 清除 ContextFlags 中查询调试寄存器组的标志位
      Context->ContextFlags & ~CONTEXT_DEBUG_REGISTERS;
  }
  return Org_NtSetContextThread(ThreadHandle, Context);
}
```

### (9) NtQueryInformationThread

查询线程信息，可以由此获得线程所对应的进程。

代码：

```
NTSTATUS NTAPI New_NtQueryInformationThread(
  IN HANDLE               ThreadHandle,
  IN THREAD_INFORMATION_CLASS ThreadInformationClass,
  OUT PVOID               ThreadInformation,
  IN ULONG                ThreadInformationLength,
  OUT PULONG              ReturnLength OPTIONAL 
  )
{
  NTSTATUS status;
  PTHREAD_BASIC_INFORMATION ThreadBasicInfo;
  status = Org_NtQueryInformationThread(ThreadHandle,
    ThreadInformationClass,
    ThreadInformation,
    ThreadInformationLength,
    ReturnLength);

  if (status == STATUS_SUCCESS && IS_HOOK == 1)
  {
    ThreadBasicInfo = (PTHREAD_BASIC_INFORMATION)ThreadInformation;
    // 获得线程对应的进程 ID，并判断是否在查询 OD 进程
    if (GetDebugInfoByPid(ThreadBasicInfo->ClientId.UniqueProcess))
    {
      RtlZeroMemory(ThreadInformation, sizeof(THREAD_BASIC_INFORMATION));
      if (ReturnLength != NULL)
        *ReturnLength = 0;
      status = STATUS_ACCESS_DENIED;
    }
  }

  return status;
}
```

### 四. 挂钩的 SSDT Shadow 函数

`PhantOm` 挂钩了 4 个 `SSDT Shadow` 函数，主要用于防止程序检测到 `OD` 的窗口。

#### (1) NtUserGetForegroundWindow

此函数返回用户正在工作的窗口的句柄，如果程序被调试，则顶层窗口是 `OD` 的窗口。

代码：

```
ULONG New_NtUserGetForegroundWindow(VOID)
{
  NTSTATUS status;
  status = Org_NtUserGetForegroundWindow();

  if (IS_HOOK == TRUE)
  {
    // 判断所查询的窗口是否为 OD
    if (GetDebugInfoByPid(status))
    {
      // 判断当前进程是否为 OD
      if (!FindDebugInfoByEprocess(IoGetCurrentProcess()))
        status = LastForegroundWindow;
      else
        LastForegroundWindow = status;
    }
  }

  return status;
}
```

#### (2) NtUserQueryWindow

查询给定窗口句柄所属的进程。

代码：



## 代码实现

```c
INT_PTR New_NtUserQueryWindow(
            IN ULONG WindowHandle,
            IN ULONG TypeInformation)
{
  ULONG ProcessID;

  // 查询窗口句柄所属的进程
  ProcessID = Org_NtUserQueryWindow(WindowHandle, 0);

  if (ProcessID && IS_HOOK == TRUE)
  {
    // 查询窗口句柄是否属于 OD
    if(!GetDebugInfoByPid(ProcessID))
    {
      if (!FindDebugInfoByEprocess(IoGetCurrentProcess()))
        return 0;
    }
  }

  return ProcessID;
}
```

## NtUserBuildHwndList

此函数用于枚举桌面上所有窗口的句柄，所以在 `hook` 程序中，我们必须在返回值结构中把 `OD` 所属的句柄抹掉。

```c
NTSTATUS New_NtUserBuildHwndList(
               IN HDESK hdesk, 
               IN HWND hwndNext, 
               IN ULONG fEnumChildren, 
               IN DWORD idThread, 
               IN UINT cHwndMax, 
               OUT HWND *phwndFirst, 
               OUT ULONG* pcHwndNeeded)
{
  NTSTATUS status;
  ULONG i, ProcessId;
  if (fEnumChildren == 1 && IS_HOOK == TRUE)
  {
    ProcessId = Org_NtUserQueryWindow((ULONG)hwndNext, 0);
    if (GetDebugInfoByPid(ProcessId))
    {
      if (!FindDebugInfoByEprocess(IoGetCurrentProcess())
        return STATUS_UNSUCCESSFUL;
    }
  }

  status = Org_NtUserBuildHwndList(hdesk, 
               hwndNext, 
               fEnumChildren, 
               idThread, 
               cHwndMax, 
               phwndFirst, 
               pcHwndNeeded);

  if (status == STATUS_SUCCESS)
  {
    if (IS_HOOK == 1 && !FindDebugInfoByEprocess(IoGetCurrentProcess()))
    {
      i = 0;
      while (i < *pcHwndNeeded)
      {
        // 获取句柄所属的进程 ID
        ProcessId = Org_NtUserQueryWindow((ULONG)phwndFirst[i], 0);
        // 判断句柄是否属于 OD
        if (GetDebugInfoByPid(ProcessId))
        {
          // 将此后的句柄信息前移，覆盖掉 OD 的句柄信息
          RtlCopyMemory(&phwndFirst[i+1], &phwndFirst[i], *pcHwndNeeded-i);
          // 最后一项清 0
          phwndFirst[*pcHwndNeeded-1] = 0;
          // 总数减 1
          (*pcHwndNeeded)--;
          continue; 
        }
        i++;
      }
    }
  }

  return status;
}
```

## NtUserFindWindowEx

此函数返回由窗口名或窗口类标识的窗口句柄。如果查询的是 `OD` 窗口，则返回 0。

```c
NTSTATUS New_NtUserFindWindowEx(
            IN HWND hwndParent, 
            IN HWND hwndChild, 
            IN PUNICODE_STRING pstrClassName OPTIONAL, 
            IN PUNICODE_STRING pstrWindowName OPTIONAL, 
            IN DWORD dwType)
{
  NTSTATUS status;

  status = Org_NtUserFindWindowEx(hwndParent, hwndChild, 
    pstrClassName, pstrWindowName, dwType);

  if (status == 0 || IS_HOOK == FALSE)
    return status;

  // 查询此窗口所属的进程的 ID
  ProcessId = Org_NtUserQueryWindow(status, 0);
  // 判断窗口是否属于 OD
  if (GetDebugInfoByPid(ProcessId))
  {
    // 当前进程是否为 OD
    if (!FindDebugInfoByEprocess(IoGetCurrentProcess()))
      return 0;
  }

  return status;
}
```

## 结语

很多朋友问 `PhantOm` 可不可以调 `NP` 或 `TexSafe` 保护的游戏，看看上面的程序就一目了然了，很明显不能。`PhantOm` 在驱动层只是做了很基本的服务函数表的 `hook`，用很简单的 `ssdt` 表还原它的功能就消失了。而 `NP` 这些猥琐东西不但 `inline hook` 了 n 个函数，还会采用各种手段检测游戏运行中有无异常情况发生，如果有它觉得不爽的情况就给你一个 `BSOD`。

但是用它来对付运行在 `ring3` 层的程序也绰绰有余了。

上面的代码均未经过测试，直接看汇编代码翻译过来的，有错误是必然的。偶很懒，而且 `BSOD` 的话会很心疼偶滴电脑，想想还是不调代码了。如果同志们计划打造一个山寨版的 `PhantOm` 驱动，最好经过严格修改并加上适当的错误处理，不然你的电脑会蓝得很难看。

附件是 `IDA` 文件，加了详细的注解，这个逆得比较彻底，100%全逆。还有很多函数未列出来，有兴趣的朋友可以根据 `IDA` 文件参考一下。

欢迎拍砖!!!

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `PhantOm1.54.rar` |
|----------------------------------------------------------|-------------------|

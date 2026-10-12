
# 现代 dump 技术及保护措施

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-717.htm`](https://www.pediy.com/kssd/pediy09/pediy09-717.htm)

## 本文目的

我们都喜欢使用免费的软件，这也就意味着需要有人对它们进行破解。而破解的时候就要对付各种各样的壳和 `protectors`。壳的工作原理和脱壳的基本方法在《Packer 终结篇》（NEOx, Volodya）一文中有不错的讲解。在此文中详细的讲解了 `PE` 文件格式以及 `protectors` 对它的利用方法。无疑，脱壳过程中一个重要的部分就是取得 `dump`。关于 `dump` 的取得以及反 `dump` 的方法在那些文章中也有介绍，而所有讲到的这些 `PE Tools` 中的 anti-dumping 的方法都有绕过的办法。但是，遗憾的是，所有这些只能用于简单的保护，而更为复杂的 `protectors`（`eXtreme Protector`, `Armadillo`）都使用了完全不同的反 `dump` 的方法，`PE Tools` 已经是相形见绌了。在本文中，我想来研究一下现代的反 `dump` 方法以及绕过的办法。这对于那些想学习如何脱掉比 `ASPack` 更复杂的壳的人们来说无疑是有用的。

## ring 0 下的 anti-dump

所有的进程 `dumpers` 都是由 `OpenProcess`/`ReadProcessMemory`/`VirtualQueryEx` 等函数来实现的。为了取得进程加载的模块列表通常要使用 `ToolHelp API`，`ToolHelp API` 函数使用 `ReadProcessMemory` 来读取进程的内存。在 `NativeAPI` 级别上调用的是函数 `ZwOpenProcess` 和 `ZwReadVirtualMemory`。一个很显然的反 `dump` 的方法就是创建驱动程序，利用驱动在内核模式下拦截这些函数并禁止它们对所保护的进程进行访问。

最简单的办法就是只拦截 `ZwOpenProcess` 函数，因为要读取进程的内存首先得把进程打开。HOOK 的 handler 一般是这样的：

```
NTSTATUS NewNtOpenProcess (  
    OUT PHANDLE ProcessHandle,  
    IN ACCESS_MASK DesiredAccess,  
    IN POBJECT_ATTRIBUTES ObjectAttributes,  
    IN PCLIENT_ID ClientId OPTIONAL)  
{  
  HANDLE ProcessId;  

  if ((ULONG)ClientId > *MmUserProbeAddress) return STATUS_INVALID_PARAMETER;  
  __try  
  {  
    ProcessId = ClientId->UniqueProcess;  
  }  
  __except(EXCEPTION_EXECUTE_HANDLER)  
  {  
    DPRINT("Exception");  
    return STATUS_INVALID_PARAMETER;  
  }  

  if (IsAdded(wLastItem, ProcessId))  
  {  
    DPRINT("Access Denied!");  
    return STATUS_ACCESS_DENIED;  
  } else  

  return TrueNtOpenProcess(ProcessHandle, DesiredAccess,  
                           ObjectAttributes, ClientId);  
}
```

这段代码首先用保险的办法取出要打开的进程的 `ProcessID`，然后判断是该传递控制权还是返回 `STATUS_ACCESS_DENIED`。因为系统中受保护的进程可能不止一个，所以必须建立进程的列表并向其中添加新的进程并删除已经搞定的进程。为了建立这个列表我们来讲一下用来描述受保护进程的结构体：

```
typedef struct _ProcessList  
{  
  PVOID NextItem;  
  HANDLE Pid;  
} TProcessList, *PProcessList;
```

下面是管理结构体链表的代码：

```
BOOLEAN IsAdded(PProcessList List, HANDLE Pid)  
{  
  PProcessList Item = List;  
  while (Item)  
  {  
    if (Pid == Item->Pid) return TRUE;  
    Item = Item->NextItem;  
  }  
  return FALSE;  
}  

void DelItem(PProcessList *List, HANDLE Pid)  
{  
  PProcessList Item = *List;  
  PProcessList Prev = NULL;  
  while (Item)  
  {  
      if (Pid == Item->Pid)  
    {  
                 if (Prev) Prev->NextItem =  
                   Item->NextItem; else *List = Item->NextItem;  
                 ExFreePool(Item);  
                 return;  
    }  
    Prev = Item;  
    Item = Item->NextItem;  
  }  
  return;  
}  


void FreePointers(PProcessList List)  
{  
    PProcessList Item = List;  
  PVOID Mem;  
  while (Item)  
  {  
    Mem = Item;     
    Item = Item->NextItem;  
    ExFreePool(Mem);  
  }  
  return;  
}  


void AddItem(PProcessList *List, HANDLE Pid)  
{  
  PProcessList wNewItem;  
  wNewItem = ExAllocatePool(NonPagedPool, sizeof(TProcessList));  
  wNewItem->NextItem = *List;  
  *List = wNewItem;  
  wNewItem->Pid = Pid;  
  return;  
}
```

当发现新进程时，为了向链表中添加此进程我们将使用 `IOCTL` 向驱动程序发出请求，而完成后则用同样的办法将其删除。实现这种 anti-dump 的驱动程序的完整代码位于本文的附件程序中。

更为可靠的保护就是拦截 `ZwReadVirtualMemory`/`ZwWriteVirtualMemory` 和 `ZwCreateThread`，但是这样就必须通过进程的句柄来取得它的 `ProcessID`。为此可以使用 `ZwQueryInformationProcess` 函数，但是对于那些有 `PROCESS_QUERY_INFORMATION` 访问标志的句柄来说这招就不灵了，所以最好是使用 `ObReferenceObjectByHandle` 来取得 `EPROCESS` 结构体指针，然后直接从 `EPROCESS` 结构体中取出 `ProcessID`。代码如下：

```
ULONG GetPid(HANDLE PHanlde)  
{  
    NTSTATUS st = 15;  
  PEPROCESS process = 0;  
  ULONG pId;  
  
  st = ObReferenceObjectByHandle(PHanlde, 0, NULL, UserMode, &process, NULL);  

  if (st == STATUS_SUCCESS)  
  {  
    pId = *(PULONG)(((ULONG)process) + pIdOffset);  

    ObDereferenceObject(process);  

    return pId;  
  }  
  return 0;  
}
```

`pIdOffset` - `ProcessId` 在 `EPROCESS` 结构体中的偏移量，这个 `constant` 在不同版本系统的内核中是不一样的，所以在驱动启动的时候要检查内核的版本并相应的为变量赋值。在写 handler 时要考虑到，如果进程试图用 `ZwOpenProcess` 来打开自己时也应该能工作（比较 `PsGetCurrentProcessId`），而在处理接收句柄的函数时要考虑到用于表示当前进程的伪句柄（-1）的存在性。

说起理论来都挺容易，但实际实现起来却是一路的磕磕绊绊。比如，在 `Windows XP` 中有 style 服务。为了实现它需要允许子系统服务进程（`csrss.exe`）访问受保护进程的内存。由此就产生了问题——如何确定这个进程的 `Id`？这个问题初看上去好像简单，其实不然。用进程名来确定是不行的，因为系统中可能有不止一个的 `csrss.exe` 进程（例如，`cracker` 可以给自己的 `dumper` 也取这个名字），所以还需要更为可靠的唯一确定进程的方法。为此我决定使用这样的办法，子系统 sever 拥有某些命名对象，通过这些命名对象可以确定这个进程。例如，我们取 `LPC` 端口 `\\Windows\\ApiPort`，在所有版本的 `Windows NT` 上它都是由 `csrss` 创建的。为了确定它需要使用 `ZwQuerySystemInformation` 枚举所有已打开的句柄，将它们拷贝到内核句柄表中，调用 `ZwQueryObject` 并比较所取得的名称与所寻找的是否一致。若一致，则拥有此句柄的进程的 `Id` 就是 `csrss` 的。下面是实现代码：

```
PVOID GetInfoTable(ULONG ATableType)  
{  
  ULONG mSize = 0x4000;  
  PVOID mPtr = NULL;  
  NTSTATUS St;  

  do  
  {  
    mPtr = ExAllocatePool(PagedPool, mSize);  

    if (mPtr != NULL)  
    {  
      St = ZwQuerySystemInformation(ATableType, mPtr, mSize, NULL);  

    } else return NULL;  

    if (St == STATUS_INFO_LENGTH_MISMATCH)  
    {  
      ExFreePool(mPtr);  
      mSize = mSize * 2;  
    }  

  } while (St == STATUS_INFO_LENGTH_MISMATCH);  

  if (St == STATUS_SUCCESS)   
  {  
    DPRINT("GetInfoTable Success!");  
    DPRINT("Info table in memory size - %d", mSize);  
    return mPtr;   

  } else ExFreePool(mPtr);  

  DPRINT("Error on GetInfoTable %X", St);  

  return NULL;  
}  

ULONG GetCsrPid()  
{  
int r;  
HANDLE Process, hObject;  
NTSTATUS St;  
ULONG CsrId = 0;  
OBJECT_ATTRIBUTES obj;  
CLIENT_ID cid;  
POBJECT_NAME_INFORMATION ObjName;  
UNICODE_STRING ApiPortName;  
  
  PSYSTEM_HANDLE_INFORMATION_EX Handles;  

  RtlInitUnicodeString(&ApiPortName, L"\\Windows\\ApiPort");  

  DPRINT("Get handles info");  

  Handles = GetInfoTable(SystemHandleInformation);  

  if (Handles == NULL) return 0;  

  ObjName = ExAllocatePool(PagedPool, 0x2000);  

  DPRINT("Number of handles %d", Handles->NumberOfHandles);  

  for (r = 0; r != Handles->NumberOfHandles; r++)  
  {  
    if (Handles->Information[r].ObjectTypeNumber == 21) //Port object  
    {  
  InitializeObjectAttributes(&obj, NULL, OBJ_KERNEL_HANDLE, NULL, NULL);  
      
      cid.UniqueProcess = (HANDLE)Handles->Information[r].ProcessId;  
      cid.UniqueThread  = 0;  

      if (ZwOpenProcess(&Process, PROCESS_DUP_HANDLE, &obj, &cid)  
          == STATUS_SUCCESS)  
      {  
        if (ZwDuplicateObject(Process,  
          (HANDLE)Handles->Information[r].Handle,  
          NtCurrentProcess(), &hObject,  
          0, 0, DUPLICATE_SAME_ACCESS) == STATUS_SUCCESS)  
        {  
          if (ZwQueryObject(hObject, ObjectNameInformation,  
              ObjName, 0x2000, NULL) == STATUS_SUCCESS)  
          {  
            if (ObjName->Name.Buffer != NULL)  
            if (wcsncmp(ApiPortName.Buffer, ObjName->Name.Buffer, 20) == 0)  
            {  
              DPRINT("Csrss %d", Handles->Information[r].ProcessId);  
              DPRINT("csr port - %ws", ObjName->Name.Buffer);  
              CsrId = Handles->Information[r].ProcessId;  
              ZwClose(Process);  
              ZwClose(hObject);  
              CsrId = Handles->Information[r].ProcessId;  
              ExFreePool(Handles);  
              ExFreePool(ObjName);  
              return CsrId;  
            }  
```



```
    } else DPRINT("Error in Query Object");
    ZwClose(hObject);
    } else DPRINT("Error on duplicating object");
    ZwClose(Process);
  } else DPRINT("Could not open process");
}
```

```
ExFreePool(Handles);
ExFreePool(ObjName);
```

```
return 0;
}
```

`Themida` (`Extreme Protector`) 使用的就是这种方法。在早期版本中为了处理好 `GUI` 的问题，使用的就是上面介绍的办法，但在后来的版本中却放弃了它（使得保护效果变差）。新版的 `Themida` 并未禁止对受保护进程全部地址空间的访问，而只是禁止了对 `EXE` 文件所驻留的内存范围的访问，而所有加载到内存里的 `DLL` 未收到任何的保护。

在本文所附的程序中可以找到编写好的驱动程序，这个驱动程序可以保护进程不会被用这种方法 `dump` 出来。如果有人想把这个驱动添加到自己的 `protector` 里并开始大卖自己的 `Xprot` 的话，我会感到悲哀的——这种方法已经过时了，这里讲到它只是为了让文章完整。

当然，这种方法看上去还是相当好的，因为大多数 `cracker` 都不在内核模式下进行创作。摘除这种保护的办法之一就是找到原始处理程序的地址，`patch` 掉 `SDT` 摘除 `HOOK`。问题在于，这种方法并不通用，因为 `HOOK` 不只是能通过 `SDT` 建立，还可以通过拦截 `int 2Eh` (`win2000`) 或是修改 `sysenter` (`win XP`) 的处理程序，再有就是可以对原始处理程序的代码进行 `splicing`。摘除 `protector` 的 `HOOK` 并不是最好的办法，因为 `protector` 可以检查 `HOOK` 是否还存在，而且还会采取措施来阻止 `HOOK` 被摘除（`StarForce` 就是这样），所以我给出另一种方法——不在驱动程序中使用 `ZwReadVirtualMermory` 来进行 `dump`。

在 `Windows NT` 内核中有两个未公开的函数 `KeAttachProcess` / `KeDetachProcess`，驱动程序可以调用这两个函数来改变当前地址空间。函数原型如下：

```
extern
void KeAttachProcess(PEPROCESS Process);

extern
void KeDetachProcess(void);
```

通过这两个函数可以 `attach` 到受保护的进程并读取它的内存，但遗憾的是最新版的 `Themida` 对这俩个函数进行了 `patch`，禁止用这种方法来 `dump` 自己的进程。但是它的保护并不全面，因为能 `attach` 到别的进程的函数还有 `KeStackAttachProcess` / `KeUnstackDetachProcess`，而这两个函数却常被 `protectors` 遗忘（也容易产生问题，因为系统驱动程序会用到这两个函数），所以我们就用它们来读取内存。以下是函数原型和用到的结构体：

```
typedef struct _KAPC_STATE
{
  LIST_ENTRY ApcListHead[2];
  PVOID Process;
  BOOLEAN KernelApcInProgress;
  BOOLEAN KernelApcPending;
  BOOLEAN UserApcPending;
} KAPC_STATE, *PKAPC_STATE;

extern
NTKERNELAPI void KeStackAttachProcess(PVOID Process, PKAPC_STATE ApcState);

extern
NTKERNELAPI void KeUnstackDetachProcess(PKAPC_STATE ApcState);
```

即使下一版的 `Themida` 实现了对这两个函数的拦截，`attach` 到所需的进程也不难，手动修改 `cr3` 寄存器就行了。

为了读取进程的内存，我们需要使用 `IOCTL` 向我们的驱动程序传递进程的句柄、要读取的内存地址、当前进程缓冲区的地址以及要读取的字节数，在驱动获得 `dump` 进程的 `EPROCESS` 指针之后就会读取请求的数据。也就是说，实际上我们自行手工实现了 `ZwReadVirtualMemory`。代码如下：

```
void CopyProcessMem(HANDLE hProcess, PVOID SrcAddr, PVOID DstAddr, ULONG *Size)
{
  PEPROCESS process = NULL;
  NTSTATUS  st;
  PUCHAR pMem = NULL;
  ULONG Addr, Bytes;
  PUCHAR cPtr, dPtr;
  KAPC_STATE ApcState;

  st = ObReferenceObjectByHandle(hProcess, 0, NULL, UserMode, &process, NULL);

  if (NT_SUCCESS(st))
  {
    Bytes = *Size;
    pMem = ExAllocatePool(NonPagedPool, Bytes);
    dPtr = pMem;

    cPtr = (PUCHAR)SrcAddr;

    KeStackAttachProcess(process, &ApcState);

    __try
    {
      while (Bytes)
      {
        *dPtr = *cPtr;
        cPtr++;
        dPtr++;
        Bytes --;
      }
    }

    __except(EXCEPTION_EXECUTE_HANDLER)
    {
    }

    KeUnstackDetachProcess(&ApcState);

    Bytes = *Size - Bytes;

    __try
    {
      memcpy(DstAddr, pMem, Bytes);
      *Size = Bytes;
    }
    __except(EXCEPTION_EXECUTE_HANDLER)
    {
    }

    ExFreePool(pMem);

    ObDereferenceObject(process);
  }
  return;
}
```

对于驱动方式的 `dumping` 来说，要建立针对它的保护措施就要难多了，因为实际上来讲很难确定传递的信息究竟是来自于 `dumper` 还是来自于系统。但也不是不可能做到，您不要以为这招可以对付所有使用驱动 `dump` 的 `protectors`（尽管目前还是有用的）。

遗憾的是，自行替换 `ZwWriteVirtualMemory` 在很多情况下是不够的，因为有些 `protectors` 还拦截了 `ZwOpenProcess`。要绕过它，可以不去打开进程，而是通过在子系统的 `server` 进程里搜索来获得它的句柄（参见《使用 Code Injection 拦截 Windows NT API 函数》）。在我的 `advApiHook` 库里，对应这项功能的函数叫 `OpenProcessEx`。有些保护程序会修改进程的 `security marker`，以此来禁止打开内存读取。对这种保护，这个方法也可以对付。如果这种获取进程句柄的方法流行起来的话，保护程序的作者们就会最先在子系统 `server` 中把句柄关闭。要在驱动程序中获取进程句柄，只需使用 `PsLookupProcessByProcessId` 来取得指向其 `EPROCESS` 的指针，此后可以用 `ObOpenObjectByPointer` 把它加入到我们进程的句柄表中。实现代码如下：

```
HANDLE MyOpenProcess(HANDLE ProcessId)
{
  PEPROCESS Process;
  NTSTATUS St;
  HANDLE hProcess = NULL;

  PsLookupProcessByProcessId(ProcessId, &Process);

  ObOpenObjectByPointer(Process, 0, NULL, 0, NULL, UserMode, &hProcess);

  ObDereferenceObject(Process);

  return hProcess;
}
```

保护程序的作者们当然可以拦截 `PsLookupProcessByProcessId`，但这对于获取 `dump` 只能起到 5 分钟的拖延作用，因为句柄的获取还可以通过手工挖掘内核结构体来完成，或者干脆就不用句柄也不用 `API`。

`ring 0` 下的保护手段就丰富多彩了（想象无限，创意无限）。例如，一个非常不错的 `anti-dump` 的方法就是破坏受保护进程的页表。为此需要干预调度器的工作并拦截某些 `SwapContext` 的未导出函数，这些函数都是在线程切换时要调用的。在拦截处理程序中，在切换入受保护进程时需要恢复页表，而在切换出时就把页表破坏掉。更为简单的是可以对抗 `dumper` 驱动。绕过这种保护也不困难，既可以在内核中进行（拦截中断处理程序），也可以在用户模式下进行（`Code Injection`）。我想，只要肯动脑，大家都能想出进行 `dump` 的自己原创的方法。

### Dynamic unpacking

另一种对付 `dump` 的常用方法就是的 `dynamic packing`。其思想就是，`protector` 并不将受保护的程序完全 `unpack`，而只是 `unpack` 一部分。首先 `unpack` 第一页，当快要进行完时 `protector` 对异常进行拦截并 `unpack` 所请求的页，这时它就可以将上一页从内存中删除。这样受保护进程的 `image` 在内存里从来都不曾完整过，因此一般的 `dumper` 是没法进行 `dump` 的。这种办法在 `protector` 程序 `Armadillo` 中被广泛使用，被其称为 `CopyMem`。为了拦截异常并对代码进行解码，程序建立了独立的进程，这个进程使用 `DebugAPI` 来调试受护进程。

对于摘除这种保护，大多数 `cracker` 采用的方法都不是最优的。他们的方法就是 `reverse` 并 `patch` 掉 `protector` 的代码，以此来迫使它解出完整的代码。这实在是太费事了，更让人难以接受的是，在每个新版本中，作者们都会修改这种保护机制的代码，这样旧的 `dump` 法就不再有效，从而就被人们毫不犹豫地扔到博物馆里。但幸运的是，我们还有更简单的办法——从内部来 `dump` 进程。为此需要将 `dump` 的代码注入到受保护进程的地址空间中并读取它的内存。在此过程中将会出现异常，处理异常的时候 `protector` 会向我们呈现全部解密后的代码。这种摘除 `CopyMem` 保护的方法已经是可以想出的最简单的办法了。`CopyMem` 至今仍在被使用，原因就是这种方法还没有广泛流行。

### 获取 kernel module 的 dump

在逆向 `ring 0` 下的 `protector` 的时候经常需要 `dump` 已加载到核心内存的 `PE` 文件。例如，如果 `protector` 对系统内核进行了 `patch`，我们可以将内核的 `dump` 与磁盘文件做个比较，从而很容易地发现这个情况。抑或是 `dump` 出 `protector` 的驱动程序并研究它的解密方法。就是做到这些就已经不太容易了，因为所有现在的 `protectors` 早已经学会了在所有系统结构体中删除对自身驱动的引用，以此来实现驱动的隐藏，所以 `dump` 这种驱动的最现实的办法就是 `dump` 所有核心内存，之后在此 `dump` 中查找我们需要的东西。但是这个办法常常又不是很有效，因为保护程序的设计者们已经停止了对驱动的 `pack`（比如 `StarForce 3`），而是转向了更有前景的保护方法，比如多态代码以及虚拟机。

初看上去，`dump` 内核模块只需要用 `copymem` 进行拷贝就行了，但实际上并没有这么简单，因为这里存在着一个陷阱。若要试图从头到尾读取模块内存的话，所能得到的就只是个蓝屏。若要尝试为所需内存段创建 `MDL` 并执行 `MmProbeAndLockPages`，则会引发异常，而如果调用 `MmBuildMdlForNonPagedPool`，则 `MDL` 会被建立，但要读取它会再一次引发蓝屏。与此相关的是，在 `native PE` 文件里，`section` 可以有个“Discardable as needed”属性，这样的 `section` 在 `Driver Entry` 结束后会被立即删除。这样的系统可以节约 `NonPaged Pool`，因为 `NonPaged Pool` 在系统里可是稀缺的。于是尝试访问这个地址就会引发异常，这个异常 `SEH` 不予处理，于是引起系统的崩溃。所以在读取内存之前我们需要用 `MmIsAddressValid` 来验证地址的有效性。以下是安全读取核心内存的代码：

```
NTSTATUS DumpKernelMemory(PVOID SrcAddr, PVOID DstAddr, ULONG Size)
{
    PMDL  pSrcMdl, pDstMdl;
    PUCHAR pAddress, pDstDddress;
    NTSTATUS st = STATUS_UNSUCCESSFUL;
    ULONG r;

    pSrcMdl = IoAllocateMdl(SrcAddr, Size, FALSE, FALSE, NULL);

    if (pSrcMdl)
    {
        MmBuildMdlForNonPagedPool(pSrcMdl);

        pAddress = MmGetSystemAddressForMdlSafe(pSrcMdl, NormalPagePriority);

        if (pAddress)
        {
            pDstMdl = IoAllocateMdl(DstAddr, Size, FALSE, FALSE, NULL);

            if (pDstMdl)
            {
                __try
                {
                    MmProbeAndLockPages(pDstMdl, UserMode, IoWriteAccess);

                    pDstDddress = MmGetSystemAddressForMdlSafe(
                                       pDstMdl, NormalPagePriority);

                    if (pDstDddress)
                    {
                        memset(pDstDddress, 0, Size);

                        for (r = 1; r < Size; r++)
                        {
                            if (MmIsAddressValid(pAddress)) *pDstDddress = *pAddress;
                            pAddress++;
                            pDstDddress++;
                        }

                        st = STATUS_SUCCESS;
                    }

                    MmUnlockPages(pDstMdl);
                }

                __except(EXCEPTION_EXECUTE_HANDLER)
```



```
                 {                 
                 
                 }
                 
                 IoFreeMdl(pDstMdl);
           
           }
       
           }
       
           IoFreeMdl(pSrcMdl);
       
     }
       
     return st;
}
```

除内存读取之外，我们还需要获取模块的信息来知道读取的地址。获取此信息的方法将在后面介绍。

### 使 dump 过程更为便利

编写自己的 dumper 绝对是个费事的活儿，所以我决定使用 `PE Tools`，因为它考虑了许多 `PE` 文件的细节并能方便快捷地进行 dump。所以我写了一个叫 `eXtreme dumper`（与 `Extreme Protector` 相对）的 plugin，这个 plugin 可以实现驱动 dump 和使用 `DllInjection` 的 dump。为此我在 `PE Tools` 中拦截了 `ZwOpenProcess` 和 `ZwReadVirtualMemory` 函数并用自己的函数做了替换，它们都是用上面介绍的方法写成的。作为例子，我这里给出 `ZwReadVirtualMemory` 处理程序的代码：

```
function NewZwReadVirtualMemory(ProcessHandle: dword; BaseAddress: pointer;
                                Buffer: pointer; BufferLength: dword;
                                ReturnLength: pdword): NTStatus; stdcall;
var
 hPipe, Bytes, Len: dword;
 Req: TXDumpRequest;
 PipeName: array [0..128] of Char;
 sPid: array [0..8] of Char;
 ProcessId: dword;
 Query: TDriverQuery;
begin
 if DriverMethod then
  begin
   Result := dword(-1);
   Len := BufferLength;
   Query.QueryType := 2;
   Query.Param1    := ProcessHandle;
   Query.Param2    := dword(@Len);
   Query.Param3    := dword(BaseAddress);
   Query.Param4    := dword(Buffer);
   WriteFile(hDriver, Query, SizeOf(TDriverQuery), Bytes, nil);
   if ReturnLength <> nil then ReturnLength^ := Len;
   if Len > 0 then Result := STATUS_SUCCESS;
  end else
  begin
   ProcessId := GetPid(ProcessHandle);
   StrCpy(PipeName, bPipeName);
   ToHex(ProcessId, 8, sPid);
   StrCat(PipeName, sPid);
   hPipe := CreateFile(@PipeName,
                     GENERIC_WRITE or GENERIC_READ,
                     0, nil, OPEN_EXISTING, 0, 0);
   if hPipe <> INVALID_HANDLE_VALUE then
    begin
     Req.Address := BaseAddress;
     Req.Length  := BufferLength;
     WriteFile(hPipe, Req, SizeOf(TXDumpRequest), Bytes, nil);
     ReadFile(hPipe, Len, SizeOf(dword), Bytes, nil);
     ReadFile(hPipe, Buffer^, Len, Bytes, nil);
     if ReturnLength <> nil then ReturnLength^ := Len;
     Result := 0;
     CloseHandle(hPipe);
    end else Result := TrueZwReadVirtualMemory(ProcessHandle,
                                  BaseAddress, Buffer,
                                  BufferLength, ReturnLength);
  end;
end;
```

我们看到，为了使 dumper 与注入的 DLL 互动在受保护的进程中使用了 named pipes，其名字就是字符串 `eXtremeDumper` 和进程 Id 的 HEX 代码。读取内存的请求发向被 dump 的进程，在那里 server 端读取请求的内存并将数据和读取的字节数返回。dumper 的 server 端的代码如下：

```
library xDump;

uses
  Windows,
  advApiHook,
  NativeApi;

{$include string.inc}

type
 PXDumpRequest = ^TXDumpRequest;
 TXDumpRequest = packed record
  ReqType: dword;
  Address: pointer;
  Length: dword;
 end;

const
 bPipeName = '\\.\pipe\eXtremeDumper'#0;

function SafeReadMemory(Addr, Buff: pointer; Size: dword): dword;
asm
 push ebx
 push edx
 push ecx
 push esi
 push edi
 push ebp
 push offset @Handler
 push fs:[0]
 mov fs:[0], esp
 mov esi, eax
 mov edi, edx
 rep movsb
 mov eax, ecx
 pop fs:[0]
 add esp, 4
 pop ebp
 pop edi
 pop esi
 pop ecx
 pop edx
 pop ebx
 ret
@handler:
 mov ecx, [esp + $0C]
 add [ecx + $B8], 2
 ret
end;

function PipeThread(hPipe: dword): dword; stdcall;
var
 Req: TXDumpRequest;
 Bytes, Len: dword;
 pBuff: pointer;
begin
  ReadFile(hPipe, Req, SizeOf(TXDumpRequest), Bytes, nil);
  GetMem(pBuff, Req.Length);
  Len := Req.Length - SafeReadMemory(Req.Address, pBuff, Req.Length);
  WriteFile(hPipe, Len, SizeOf(dword), Bytes, nil);
  WriteFile(hPipe, pBuff^, Req.Length, Bytes, nil);
  FreeMem(pBuff);
  CloseHandle(hPipe);
end;

var
 TrId: dword;

procedure PipeServerThread();
var
 hPipe: dword;
 PipeName: array [0..128] of Char;
 sPid: array [0..8] of Char;
begin
 StrCpy(PipeName, bPipeName);
 ToHex(GetCurrentProcessId(), 8, sPid);
 StrCat(PipeName, sPid);
 repeat
  hPipe := CreateNamedPipe(PipeName,
                          PIPE_ACCESS_DUPLEX or WRITE_DAC,
                          PIPE_TYPE_MESSAGE or PIPE_READMODE_MESSAGE or PIPE_WAIT,
                          PIPE_UNLIMITED_INSTANCES, 1024, 1024, 5000, nil);
  if hPipe = INVALID_HANDLE_VALUE then Exit;
  if ConnectNamedPipe(hPipe, nil) then
  CreateThread(nil, 0, @PipeThread, pointer(hPipe), 0, TrId);
 until false;
end;

begin
 CreateThread(nil, 0, @PipeServerThread, nil, 0, TrId);
end.
```

在 client 端连接 pipe 时会创建一个线程，该线程在 SEH 处理程序内部读取请求的数据，此后数据就流向 client 端。此代码唯一的缺点就是效率比较低，因为每个请求都是由单独的线程服务的。但是，实际上太高的效率也是多余，因为重要的是 dump 的获取，尽管有人会不满足，会编写更好的版本，但我这里就费点傻劲吧 :)。

现在我们回到内核模块信息的获取上来。为了实现这个目的，我将使用函数 `ZwQuerySystemInformation`，class 为 `SystemModuleInformation`，这样函数会向我返回以下类型的结构体数组：

```
 PSYSTEM_MODULE_INFORMATION = ^SYSTEM_MODULE_INFORMATION;
 SYSTEM_MODULE_INFORMATION = packed record // Information Class 11
    Reserved: array[0..1] of ULONG;
    Base: PVOID;
    Size: ULONG;
    Flags: ULONG;
    Index: USHORT;
    Unknown: USHORT;
    LoadCount: USHORT;
    ModuleNameOffset: USHORT;
    ImageName: array [0..255] of Char;
   end;

 PSYSTEM_MODULE_INFORMATION_EX = ^SYSTEM_MODULE_INFORMATION_EX;
 SYSTEM_MODULE_INFORMATION_EX = packed record
    ModulesCount: dword;
    Modules: array[0..0] of SYSTEM_MODULE_INFORMATION;
   end;
```

我们感兴趣的域是 `ImageName`、`Base` 和 `Size`。我发现，在 `KernelMode` 下还可以通过链表 `PsLoadedModulesList` 来获取这项信息，但是用在这里就没什么意义了。

为了枚举进程加载的模块，`PE Tools` 使用了 `Procs32.dll` 中的函数 `GetModuleFirst`/`GetModuleNext`。为了映射自己的列表，我们将在 `System` 进程 zhong 拦截并替换成我们的版本。处理这些函数的代码如下：

```
function NewGetModuleNext(dwPID: dword; mEntry: PMODULE_ENTRY): bool; stdcall;
begin
  if dwPID = SystemPid then
   begin
    Result := false;
    lstrcpy(mEntry^.lpFileName, Modules^.Modules[CurrentModule].ImageName);
    mEntry^.dwImageBase := dword(Modules^.Modules[CurrentModule].Base);
    mEntry^.dwImageSize := Modules^.Modules[CurrentModule].Size;
    mEntry^.bSystemProcess := true;
    Inc(CurrentModule);
    if CurrentModule > Modules^.ModulesCount then
   ReleaseModulesInfo() else Result := true;
   end else Result := TrueGetModuleNext(dwPID, mEntry);
end;

function NewGetModuleFirst(dwPID: dword; mEntry: PMODULE_ENTRY): bool; stdcall;
begin
  if dwPID = SystemPid then
   begin
    if CurrentModule > 0 then ReleaseModulesInfo();
    Modules := GetInfoTable(SystemModuleInformation);
    Result := NewGetModuleNext(dwPID, mEntry);
   end else Result := TrueGetModuleFirst(dwPID, mEntry);
end;
```

为了进行 dump，`PE Tools` 使用了 `NDump.dll` 中的 `DumpProcess` 函数，我们需要拦截这个函数并将 dump 请求发向我们自己的驱动里。处理程序的代码如下：

```
function NewDumpProcess(dwProcessId: dword; pStartAddr: pointer;
                      dwcBytes: dword; pDumpedBytes: pointer
                    bool; stdcall;
var
 Query: TDriverQuery;
 Bytes: dword;
begin
  if dwProcessId = SystemPid then
   begin
    Query.QueryType := IOCTL_DUMP_KERNEL_MEM;
    Query.Param1    := dword(pStartAddr);
    Query.Param2    := dwcBytes;
    Query.Param3    := dword(pDumpedBytes);
```



```
Result := WriteFile(hDriver, Query, SizeOf(TDriverQuery), Bytes, nil);  

end else Result := TrueDumpProcess(dwProcessId, pStartAddr,  

                              dwcBytes, pDumpedBytes);  

end;  
```

--------------------------------------------------------------------------------

eXtreme dumper 可以分为以下部分：

- 两种 dump 的方法（`Dll-Injection` 法和驱动法）
- 获取句柄的隐式方法（`OpenProcessEx`）
- 防止 PE Tools 受其它进程影响（`Protect PE Tools`）
- 对内核模块的 dump（`Enable kernel modules dumping`）

简言之，这个 plugin 无疑是很有用的。

--------------------------------------------------------------------------------

我想提醒的是，写破解程序这类文章的作者们请不要进行违反俄联邦法规的活动。本文内容只能用于教学目的，对任何由此引起的非法行为，本作者不负有任何责任。（对使用本译文进行违反中华人民共和国法规的行为，译者亦不负任何责任）

`[C] Ms-Rem`

greatdong 译

http://greatdong.blog.edu.cn

`4dd466d0e1e0cc123afbe504da2462ca`


```
{
  PUCHAR cPtr, pOpcode;
  ULONG Length;
  for (cPtr = (PUCHAR)KeSetAffinityThread;
       cPtr < (PUCHAR)KeSetAffinityThread + PAGE_SIZE;
             cPtr += Length)
  {
    Length = SizeOfCode(cPtr, &pOpcode);
    if (!Length) break;       
    if (*(PUSHORT)pOpcode == 0x048D && *(pOpcode + 2) == 0xCD && *(pOpcode + 7) == 0x39)
    {
      KiDispatcherReadyListHead = *(PVOID *)(pOpcode + 3);
      break;
    }
  }
  return;
}
```

不幸的是，Windows XP 内核完全不同于 Windows 2000 内核。XP 下的调度程序（`scheduler`）只有两个线程链表：`KiWaitListHead` 和 `KiDispatcherReadyListHead`。我们可以通过搜索 `KeDelayExecutionThread` 函数来查找 `KeWaitListHead`：

Code:

```
.text:004055B5             mov     dword ptr [ebx], offset KiWaitListHead
.text:004055BB             mov     [ebx+4], eax
```

搜索代码如下：

Code:

```
void XPGetKiWaitListHead()
{
  PUCHAR cPtr, pOpcode;
  ULONG Length;
  for (cPtr = (PUCHAR)KeDelayExecutionThread;
       cPtr < (PUCHAR)KeDelayExecutionThread + PAGE_SIZE;
             cPtr += Length)
  {
    Length = SizeOfCode(cPtr, &pOpcode);
    if (!Length) break;
    if (*(PUSHORT)cPtr == 0x03C7 && *(PUSHORT)(pOpcode + 6) == 0x4389)
    {
      KiWaitInListHead = *(PLIST_ENTRY *)(pOpcode + 2);
      break;
    }
  }
  return;
}
```

最困难的是查找 `KiDispatcherReadyListHead`。主要的问题是 `KiDispatcherReadyListHead` 的地址并没有被任何一个导出的函数使用。因此就要用更加复杂的搜索算法搞定它。就从 `KiDispatchInterrupt` 函数开始，我们感兴趣的地方只有这里：

Code:

```
.text:00404E72             mov     byte ptr [edi+50h], 1
.text:00404E76             call    sub_404C5A
.text:00404E7B             mov     cl, 1
.text:00404E7D             call    sub_404EB9
```

这段代码中的第一个函数调用指向的就是包含 `KiDispatcherReadyListHead` 引用的函数。尽管如此，搜索 `KiDispatcherReadyListHead` 的地址却变的更加复杂，因为这个函数的相关代码在 Windows XP SP1 和 SP2 中是不同的。在 SP2 中它是这个样子：

Code:

```
.text:00404CCD             add     eax, 60h
.text:00404CD0             test    bl, bl
.text:00404CD2             lea     edx, KiDispatcherReadyListHead[ecx*8]
.text:00404CD9             jnz     loc_401F12
.text:00404CDF             mov     esi, [edx+4]
```

And in SP1:

SP1 中是这样的：

Code:

```
.text:004180FE             add     eax, 60h
.text:00418101             cmp     [ebp+var_1], bl
.text:00418104             lea     edx, KiDispatcherReadyListHead[ecx*8]
.text:0041810B             jz      loc_418760
.text:00418111             mov     esi, [edx]
```

仅仅查找一个“`lea`”指令是不可靠的，因此我们也应该检查“`lea`”后面的指令（`LDasm` 中的 `IsRelativeCmd` 函数）。搜索 `KiDispatcherReadyListHead` 的全部代码如下：

Code:

```
void XPGetKiDispatcherReadyListHead()
{
  PUCHAR cPtr, pOpcode;
  PUCHAR CallAddr = NULL;
  ULONG Length;
  for (cPtr = (PUCHAR)KiDispatchInterrupt;
       cPtr < (PUCHAR)KiDispatchInterrupt + PAGE_SIZE;
             cPtr += Length)
  {
    Length = SizeOfCode(cPtr, &pOpcode);
    if (!Length) return;
    if (*pOpcode == 0xE8 && *(PUSHORT)(pOpcode + 5) == 0x01B1)
    {
      CallAddr = (PUCHAR)(*(PULONG)(pOpcode + 1) + (ULONG)cPtr + Length);
      break;
    }
  }
  if (!CallAddr || !MmIsAddressValid(CallAddr)) return;
  for (cPtr = CallAddr; cPtr < CallAddr + PAGE_SIZE; cPtr += Length)
  {
    Length = SizeOfCode(cPtr, &pOpcode);
    if (!Length) return;
    if (*(PUSHORT)pOpcode == 0x148D && *(pOpcode + 2) == 0xCD && IsRelativeCmd(pOpcode + 7))
    {
      KiDispatcherReadyListHead = *(PLIST_ENTRY *)(pOpcode + 3);
      break;
    }
  }
  return;
}
```

找到线程链表地址之后我们就可以非常简单地枚举出那些进程了，代码如下：

Code:

```
void ProcessListHead(PLIST_ENTRY ListHead)
{
  PLIST_ENTRY Item;
  if (ListHead)
  {
    Item = ListHead->Flink;
    while (Item != ListHead)
    {
      CollectProcess(*(PEPROCESS *)((ULONG)Item + WaitProcOffset));
      Item = Item->Flink;
    }
  }
  return;
}
```

`CollectProcess` 是一个非常有用的函数，它可以增加一个进程到进程列表中去。

### 通过拦截系统调用得到进程列表。

任何一个进程都要通过 `API` 来和系统进行交互，而大多数交互都通过系统调用传递给了内核。当然，进程也可以不使用任何 `API` 而存在，但是这样一来它也就不能做任何有用（或有害）的事情。一般而言，我们的思路是使用系统调用管理器拦截系统调用，然后得到管理器中当前进程的 `EPROCESS` 指针。应该在某段时间收集指针列表，这个表不会包含信息收集时没有使用任何系统调用的进程（比如，进程的线程都处于等待状态）。

Windows 2000 平台使用 `2Eh` 中断进行系统调用，因此我们需要修改 `IDT` 中的相应的中断描述符来拦截系统调用，这就要用 `sidt` 指令得到 `IDT` 在内存中的位置。该指令返回这样一个结构：

Code:

```
typedef struct _Idt
{
  USHORT Size;
  ULONG  Base;
} TIdt;
```

修改 `2Eh` 中断向量的代码如下：

Code:

```
void Set2kSyscallHook()
{
  TIdt Idt;
  __asm
  {
    pushad
    cli
    sidt [Idt]
    mov esi, NewSyscall
    mov ebx, Idt.Base
    xchg [ebx + 0x170], si
    rol esi, 0x10
    xchg [ebx + 0x176], si
    ror esi, 0x10
    mov OldSyscall, esi
    sti
    popad
  }
}
```

当然在卸载驱动之前还要保存原始状态的信息：

Code:

```
void Win2kSyscallUnhook()
{
  TIdt Idt;
  __asm
  {
    pushad
    cli
    sidt [Idt]
    mov esi, OldSyscall
    mov ebx, Idt.Base
    mov [ebx + 0x170], si
    rol esi, 0x10
    mov [ebx + 0x176], si
    sti
    xor eax, eax
    mov OldSyscall, eax
    popad
  }
}
```

Windows XP 使用 `sysenter`/`sysexit` 指令（出现在 `Pentium 2` 处理器中）实现系统调用。这些指令的功能由 `model-specific registers (MSR)` 控制。系统调用管理器的地址保存在 `MSR` 寄存器，`SYSENTER_EIP_MSR (0x176)` 中。用 `rdmsr` 指令读取 `MSR` 寄存器，同时设置 `ECX = 要读取的寄存器的号码`，结果保存在两个积存器 `EDX:EAX` 中。在我们这里，`SYSENTER_EIP_MSR` 积存器是 32 位积存器，所以 `EDX` 为 0，`EAX` 内是系统调用管理器的地址。同样地，我们也可以用 `wrmsr` 指令写 `MSR` 积存器。有一个地方需要注意：当写 32 位 `MSR` 积存器的时候，`EDX` 应该被清空，否则将引起异常并且导致系统立即崩溃。

考虑到所有的事情之后，替代系统调用管理器的代码如下：

Code:

```
void SetXpSyscallHook()
{
  __asm
  {
    pushad
    mov ecx, 0x176
    rdmsr
    mov OldSyscall, eax
    mov eax, NewSyscall
    xor edx, edx
    wrmsr
    popad
  }
}
```

恢复原始的系统调用管理器代码：

Code:

```
void XpSyscallUnhook()
{
  __asm
  {
    pushad
    mov ecx, 0x176
    mov eax, OldSyscall
    xor edx, edx
    wrmsr
    xor eax, eax
    mov OldSyscall, eax
    popad
  }
}
```

Windows XP 的另外一个特性是它既可以使用 `sysenter` 也可以使用 `int 2Eh` 来进行系统调用，所以我们要替换这两种情况下的系统调用管理器。我们的新的系统调用管理器应该得到当前进程的 `EPROCESS` 指针，并且如果是一个新的进程，我们要把这个新的进程加到我们的进程列表中。新的系统调用管理器代码如下：

Code:

```
void __declspec(naked) NewSyscall()
{
  __asm
  {
    pushad
    pushfd
    push fs
    mov di, 0x30
    mov fs, di
    mov eax, fs:[0x124]
    mov eax, [eax + 0x44]
    push eax
    call CollectProcess
    pop fs
    popfd
    popad
    jmp OldSyscall
  }
}
```

得到进程列表的这段代码应该在某个时间段内工作，所以我们有这样的问题：如果在列表中的进程被删除掉，在随后的时间内我们将保留一些无效指针，结果就是检测隐藏进程失败或者导致系统 `BSOD`。解决这个问题的办法是，用 `PsSetCreateProcessNotifyRoutine` 函数注册我们的回调函数，这个回调函数将会在系统创建或者销毁一个进程的时候被调用。当进程被销毁时，我们也应该把它从我们的表中删除掉。

回调函数的原型如下：

Code:

```
VOID
(*PCREATE_PROCESS_NOTIFY_ROUTINE) (
    IN HANDLE  ParentId,
    IN HANDLE  ProcessId,
    IN BOOLEAN Create
    );
```

安装回调函数的代码如下：

Code:

```
 PsSetCreateProcessNotifyRoutine (NotifyRoutine, FALSE);
```

取消回调函数的代码：

Code:

```
 PsSetCreateProcessNotifyRoutine (NotifyRoutine, TRUE);
```

这里有一个问题，回调函数总是在系统被销毁的时候创建，因此我们不可能直接在这个回调函数中删除进程列表中的相应进程。这样我们就要用系统的 `work items`，首先调用 `IoAllocateWorkItem` 函数为 `work item` 分配内存，然后调用 `IoQueueWorkItem` 函数（俄文翻译者 `kao` 注：这一句我不太确定...）将任务放置到工作线程队列中。在处理过程中我们不仅仅从进程列表中删除掉已经终止的进程，而且还要加入新创建的线程。处理代码如下：

Code:

```
void WorkItemProc(PDEVICE_OBJECT DeviceObject, PWorkItemStruct Data)
{
  KeWaitForSingleObject(Data->pEPROCESS, Executive, KernelMode, FALSE, NULL);
  DelItem(&wLastItem, Data->pEPROCESS);
  ObDereferenceObject(Data->pEPROCESS);
  IoFreeWorkItem(Data->IoWorkItem);
  ExFreePool(Data);
  return;
}

void NotifyRoutine(IN HANDLE  ParentId,
```

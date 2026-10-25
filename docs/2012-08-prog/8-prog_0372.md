
# Rootkit Hook 之\[六\] -- SYSENTER Hook

> 原文：[`www.pediy.com/kssd/pediy10/60247.html`](https://www.pediy.com/kssd/pediy10/60247.html)

呵呵，今天这篇内容少，比较简单。

`SYSENTER` 是一条汇编指令，它是在 Pentium II 处理器及以上处理器中提供的，是快速系统调用的一部分。`SYSENTER`/`SYSEXIT` 这对指令专门用于实现快速调用。在这之前是采用 `INT 0x2E` 来实现的。`INT 0x2E` 在系统调用的时候，需要进行栈切换的工作。由于 Interrupt/Exception Handler 的调用都是通过 `call`/`trap`/`task` 这一类的 gate 来实现的，这种方式会进行栈切换，并且系统栈的地址等信息由 `TSS` 提供。这种方式可能会引起多次内存访问（来获取这些切换信息），因此，从 PentiumII 开始，IA-32 引入了新指令：`SYSENTER`/`SYSEXIT`。有了这两条指令，

从用户级到特权级的堆栈以及指令指针的转换，可以通过这一条指令来实现，并且，需要切换到的新堆栈的地址，以及相应过程的第一条指令的位置，都有一组特殊寄存器来实现，这类特殊寄存器在 IA-32 中称为 MSR(Model Specific Register)。这里牵涉到 3 个特殊寄存器：

`SYSENTER_CS_MSR`: New code segment selector 0x174

`SYSENTER_ESP_MSR`: New Stack Pointer 0x175

`SYSENTER_EIP_MSR`: New Instruction Pointer 0x176

这里标出的 3 个 16 进制数分别对应这 3 个寄存器的地址，该地址用于 Kernel debug 时，通过 `rdmsr`/`wrmsr` 指令来读/写这 3 个寄存器。步骤如下：

![](img/c396e44b16c12f1225f8190146ed03e5.jpg)

1. 装载 `SYSENTER_CS_MSR` 到 `CS` 寄存器，设置目标代码段
2. 装载 `SYSENTER_EIP_MSR` 到 `EIP` 寄存器，设置目标指令
3. `SYSENTER_CS_MSR+8` 装载到 `SS` 寄存器，设置栈段
4. 装载 `SYSENTER_ESP_MSR` 到 `ESP` 寄存器，设置栈帧
5. 切换 `RING0`
6. 清除 `EFLAGS` 的 `VM` 标志
7. 执行 `RING0` 例程

![](img/41af03325513b82de9f5438ca52b6e4a.jpg)

1. `SYSENTER_CS_MSR+16` 装载到 `CS` 寄存器
2. 将 `EDX` 的值送入 `EIP`
3. `SYSENTER_CS_MSR+24` 装载到 `SS` 寄存器
4. 将 `ECX` 的值送入 `ESP`
5. 切换回 `RING3`
6. 执行 `EIP` 处的 `RING3` 指令

我们在 `windbg` 中可以看看这个三个寄存器的情况，这个是我机器里的情况。

```
lkd> rdmsr 176
msr[176] = 00000000`8053dad0

lkd> rdmsr 175
msr[175] = 00000000`ba4e0000

lkd> rdmsr 174
msr[174] = 00000000`00000008
```

可以看到，我的机器里面当前 `SYSENTER_EIP_MSR`，`SYSENTER_ESP_MSR`，`SYSENTER_CS_MSR` 这三个寄存器的值。

我们在微软公开的内核 WRK 中发现关于这三个寄存器的设置，其中 `SYSENTER_EIP_MSR` 设置的值是 `KiFastCallEntry`。

代码如下：

```
VOID
KiLoadFastSyscallMachineSpecificRegisters(
    IN PLONG Context
    )
/*++
Routine Description:
    Load MSRs used to support Fast Syscall/return.  This routine is
    run on all processors.
Arguments:
    None.
Return Value:
    None.
--*/
{
    PKPRCB Prcb;
    UNREFERENCED_PARAMETER (Context);
    if (KiFastSystemCallIsIA32) {
        Prcb = KeGetCurrentPrcb();
        //
        // Use Intel defined way of doing this.
        //
        WRMSR(MSR_SYSENTER_CS,  KGDT_R0_CODE);
        WRMSR(MSR_SYSENTER_EIP, (ULONGLONG)(ULONG)KiFastCallEntry);
        WRMSR(MSR_SYSENTER_ESP, (ULONGLONG)(ULONG)Prcb->DpcStack);
    }
}
```

看看我电脑的情况如下：

```
lkd> rdmsr 176
msr[176] = 00000000`8053dad0

lkd> u 8053dad0
nt!KiFastCallEntry:
8053dad0 b923000000      mov     ecx,23h
8053dad5 6a30            push    30h
8053dad7 0fa1            pop     fs
8053dad9 8ed9            mov     ds,cx
8053dadb 8ec1            mov     es,cx
8053dadd 8b0d40f0dfff    mov     ecx,dword ptr ds:[0FFDFF040h]
8053dae3 8b6104          mov     esp,dword ptr [ecx+4]
8053dae6 6a23            push    23h
```

下面是 `rootkit.com` 上的一个例子，这个例子有点不厚道，在你卸载的时候会 `bsod`。我简单修改了下，贴代码如下：

```
#include "ntddk.h"

ULONG d_origKiFastCallEntry; // Original value of ntoskrnl!KiFastCallEntry

VOID OnUnload( IN PDRIVER_OBJECT DriverObject )
{
    _asm
    {
      mov ecx, 0x176
      xor edx,edx
      mov eax, d_origKiFastCallEntry     // Hook function address
      wrmsr                 // Write to the IA32_SYSENTER_EIP register
    }
}

// Hook function
__declspec(naked) MyKiFastCallEntry()
{
  __asm {
    jmp [d_origKiFastCallEntry]
  }
}

NTSTATUS DriverEntry( IN PDRIVER_OBJECT theDriverObject, IN PUNICODE_STRING theRegistryPath )
{
  theDriverObject->DriverUnload  = OnUnload;
  __asm {
              mov ecx, 0x176
      rdmsr          // read the value of the IA32_SYSENTER_EIP register
      mov d_origKiFastCallEntry, eax
      mov eax, MyKiFastCallEntry     // Hook function address
      wrmsr                 // Write to the IA32_SYSENTER_EIP register
  }
  return STATUS_SUCCESS;
}
```

注意一点，大家用 `windbg` 的时候，配置 `symbol path`，如图：

![](img/ae602d9ac165e6ca9da8a1d5af4ff33a.jpg)

后面贴上一篇堕落天才写的文章链接：`http://bbs.pediy.com/showthread.php?t=42705`，

他 `inline hook` 了 `KiFastCallEntry`，采用 `detour` 方式，写得很不错。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `SysEnterHook.rar` |
| :--- | :--- |

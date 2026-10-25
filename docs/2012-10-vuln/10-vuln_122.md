
# windows 驱动漏洞发现与利用

> 原文：[`www.pediy.com/kssd/pediy10/86002.html`](https://www.pediy.com/kssd/pediy10/86002.html)

\[翻译\]windows 驱动漏洞发现与利用

声明：本文原先发表于《黑客防线》，后经译者发至看雪论坛，如有不正之处，望看雪的各位大虾指正。

作者：Anibal Sacco

译者：riusksk（泉哥）

翻译自国外著名安全杂志《(IN)SECURE Magazine》

设备驱动程序是 windows 模块的一个基本组成部分，它可以与硬件进行交互，或者执行内核操作。通过用户层接口，用户模式下的进程可以与驱动程序建立一个沟通渠道，从而以预定的方式发送和接收数据。

近来新的驱动程序漏洞不断地被暴出,但这已经不是什么新鲜事了,毕竟驱动中总是会有漏洞的,只是只有少数人去挖掘它而已。而现在很少有程序员致力于驱动程序和 ring 0 软件的开发了,这是可以理解的,毕竟这是一项有难度的工作。一直以来这方面的官方资料都很不完善，而一些团体组织又常常隐藏他们的研究发现，但是还是有很多未公开的函数被发现，并被文档化，这些都是经一些团体组织逆向分析 windows 程序后，接着再去寻找一些被泄漏的源代码而得来的。究于这个原因，很长时间里面（甚至直至今日）很多驱动程序开发者都把精力投入到程序的稳定性及可靠性上面，以致有时忽略了一些非常基本的安全检测。windows 驱动程序漏洞暴露于一些可在用户模式下正常执行的进程，比如 `MS Word`, `MS Messenger`, 甚至是计算器。这与在 ring 0 进程下利用漏洞获取执行权限不同，它隐含着最大的可执行权限，通过该漏洞，攻击者就可能控制或破坏整个系统。

本文主要针对 windows 驱动程序建立通讯渠道的方式进行讲述，以为后面对解释如何通过它们具体的设计特点来处理一些常见的驱动程序漏洞作个铺垫。同时，我也将讲述一些利用这类漏洞，从而获得代码执行权限的攻击方式。

## 驱动程序结构

驱动程序不像用户模式下的进程，它并没有充分利用它所有的功能来执行。通常情况下，它主要是通过 `DriverEntry()`函数来构造的，相当于动态链接库中的 `DllMain()`函数，因为只有当驱动程序被加载时，它才进行内存映射。但当操作系统加载模块时，它仅执行一次。下面简单看一下这个函数，它主要是负责驱动程序的初始化，比如创造符号链接（帮助用户模式下的进程打开句柄），以及"Function Dispatch Table"（在 `DRIVER_OBJECT` 结构体中包含的一个指针列表，而 `DRIVER_OBJECT` 是用于实现驱动程序的真实功能，用户模式进程通过 `IOManager` 来调用这些指针，进而执行在内核中希望执行的代码）的初始化。

### DRIVER_OBJECT

每个驱动程序加载时，就意味着内核数据结构需要调用 `DRIVER_OBJECT`。驱动对象指针是驱动程序中 `DriverEntry()`函数的一个输入参数，当 `DriverEntry()`函数被调用时就会被初始化。

`DRIVER_OBJECT` 结构如下：

```
typedef struct _DRIVER_OBJECT  
          {  
          SHORT Type;  
          SHORT Size;  
          PDEVICE_OBJECT DeviceObject;  
          ULONG Flags;  
          PVOID DriverStart;  
          ULONG DriverSize;  
          PVOID DriverSection;  
          PDRIVER_EXTENSION DriverExtension;  
          UNICODE_STRING DriverName;  
          PUNICODE_STRING HardwareDatabase;  
          PFAST_IO_DISPATCH FastIoDispatch;  
          LONG * DriverInit;  
          PVOID DriverStartIo;  
          PVOID DriverUnload;  
          LONG * MajorFunction[28];  
          } DRIVER_OBJECT, *PDRIVER_OBJECT;
```

其中 `MajorFunction` 数组指针会被驱动程序初始化，用于指向它自身的函数。这个结构相当重要，因为这些函数将被 `IO Manager` 调用，这主要依赖于来自用户模式下的各类 `IRP` 请求。例如使用 `CloseFile()`这个 `API` 函数关闭驱动程序时,`Majorfunction[IRP_MJ_CLOSE]`指向的函数指针将会被调用.

### IRPs

来源于 `MSDN`：“Microsoft Windows 家族操作系统通过发送 I/O 请求数据包 `IRP` 与驱动程序通信。封装 `IRP` 的数据结构不仅描述 I/O 请求，而且在 I/O 请求经过处理它的驱动程序时维护请求的状态信息。因为该数据结构具有两个用途，所以 IRP 可以定义为 ：
* I/O 请求的容器，或者
* 线程独立的调用堆栈。”

现在回顾一下前面的内容：

用户模式的进程是通过请求包来与驱动程序进行通讯的，这些请求包‘告诉’驱动程序应该调用 `MajorFunction` 数组指针中的哪个函数，必要时可对用于发送和接收数据的缓冲区进行管理。这些请求包被称为 `IRP Major requests`。

### IOCTLs (or IRP_MJ_DEVICE_CONTROL) 请求

这是一关键请求，驱动程序常通过 `DeviceIoControl` 来发送和接收数据。它的原型如下：

```
BOOL WINAPI DeviceIoControl(
__in               HANDLE   hDevice,
__in               DWORD    dwIoControlCode,
__in_opt         LPVOID    lpInBuffer,
__in               DWORD    nInBufferSize,
__out_opt       LPVOID    lpOutBuffer,
__in               DWORD    nOutBufferSize,
__out_opt      LPDWORD    lpBytesReturned,
__inout_opt    LPOVERLAPPED  lpOverlapped
);
```

当用户层通过已打开的驱动程序句柄去调用 `DeviceIoControl` 函数时，它先将一个指向 `IRP object` 的指针作为参数，然后去调用 `MajorFunction[IRP_MJ_DEVICE_CONTROL]`中定义的函数。这个函数将会通过 `DeviceIoControl` 这个数据结构来接收重要数据，比如输入缓冲区、输出缓冲区，及其相应的长度。依靠这些已定义的方式，`IOManager` 可以采用各种不同的方式对缓冲区进行处理。

来源于 Microsoft knowledge Base Q115758:

“驱动程序可以使用下列三种不同的 I/O 方式："buffered," "direct," 或者 "neither"。当你使用一个内核模式驱动程序去创建一个设备对象时，可以在设备对象中的 Flags 域指定你想使用的 I/O 方式。这里可以将 `DO_BUFFERED_IO` 与 `DO_DIRECT_IO` 其中一个值赋予 flag 域，或者你也可以不选择指定的方式。在这种情况下，我们称驱动程序将 flag 指定为"neither"方式。通过驱动程序的读写分发例程，这种方式可以影响分配给设备对象的 I/O 读写请求。”

这里讲的主要是 `METHOD_NEITHER` 方式。当最后 XXX bits 被打开时，`IO_Manager` 就会使用这种方法。但这也存在一个特殊问题，与其它 I/O 方式不同的是（`IO_Manager` 使用其它方式来管理缓冲区，这对驱动程序进行内核缓冲区的分配是相对安全的），在这里 `IO_Manager` 并未对缓冲区进行任何检测。这仅需要通过 `DeviceIOControl` 去调用指向驱动函数的用户层缓冲区指针，就可以绕过任何检测，对内核区域进行非法访问。

## 漏洞

先不谈用户模式与内核模式请求方式的选择，我们来看看这些方式中相同之处：

* 用户进程打开一个句柄去访问驱动程序。
* 通过 `DeviceIoControl` 结构中 `input buffer` 存储的数据以及指定的 `output buffer` 来发送一个 `IOCTL` 请求。
* 驱动程序接收 `IOCTL`，并根据 `Inputbuffer` 中的数据进行一些操作，同时返回数据给 `Output buffer`。
* 用户进程接收数据，并继续运行。

当驱动程序并未对来自用户层的指针进行充分地检测时（或者一点也未对其进行检测），问题就出现了。如果检测不当，驱动程序将会检索输出缓冲区中的数据，并直接将其写入用户进程指定的内存，同时依据该地址来发送数据，这可能会写入到一个无效内存地址，从而导致蓝屏 `Blue Screen of Death (BSOD)`，或者被攻击者利用。下面将会对此进行讲述，通过修改特定的内核模式结构，从而允许未授权的用户进程在 ring 0 下执行任意代码，这样做的目的就是为了提升权限。

在各种不同情况下，普遍不一样的是驱动程序在输出缓冲区中返回的地址，但在大多情况下，这个地址并不是都那么重要。有时仅需要一点想像力（有时甚至需要点巫术），就可以将可预测值写入或替换掉内核中的内存数据，从而获得执行代码的权限。

为了更清楚地解释这个问题，下面举个漏洞实例（`CVE-2007-5756`），是关于 `Winpacap 4.x` 的一个驱动程序漏洞（`Winpacap` 是基于 windows 操作系统下的一个软件包，为网络链路层的实时访问提供方便）。

看看下面驱动程序例程的主要部分，如前所述，该例程中实现了用于初始化 `MajorFunctions` 数组指针的驱动函数。在这里相对我们最为重要的是 `IRP_MJ_DEVICE_CON-TROL entry` 的初始化，它提供了 `NPF_IoControl` 函数，用于处理来自用户层的 `IOCTLs`。

```
NTSTATUS DriverEntry( IN PDRIVER_OBJECT DriverObject, IN PUNICODE_STRING  
RegistryPath)  
           {  
           ...  
                // 设置设备驱动程序入口指针.  
                DriverObject->MajorFunction[IRP_MJ_CREATE] = NPF_Open;  
                DriverObject->MajorFunction[IRP_MJ_CLOSE]  = NPF_Close;  
                DriverObject->MajorFunction[IRP_MJ_READ]                   = NPF_Read;  
                DriverObject->MajorFunction[IRP_MJ_WRITE]  = NPF_Write;  
                DriverObject->MajorFunction[IRP_MJ_DEVICE_CONTROL]  =  
NPF_IoControl;  
                DriverObject->DriverUnload = NPF_Unload;
```

漏洞函数代码如下：

```
NTSTATUS NPF_IoControl(IN PDEVICE_OBJECT DeviceObject,IN PIRP  
Irp)  
           {  
           ...  
                IrpSp = IoGetCurrentIrpStackLocation(Irp);                           (1)  
FunctionCode=IrpSp->Parameters.DeviceIoControl.IoControlCode;  
                Open=IrpSp->FileObject->FsContext;  
           ...  
           ...  
                case BIOCGSTATS: //function to get the capture stats  (2)  
                      TRACE_MESSAGE(PACKET_DEBUG_LOUD, "BIOCGSTATS");  
                      if(IrpSp->Parameters.DeviceIoControl.OutputBufferLength  
< 4*sizeof(UINT))         (3)  
                      {  
                           SET_FAILURE_BUFFER_SMALL();  
                           break;  
                      }  
                       pStats = (PUINT)(Irp->UserBuffer);              (4)  
                       pStats[3] = 0;   (5)  
                       pStats[0] = 0;  
                       pStats[1] = 0;  
                       pStats[2] = 0;               // Not yet supported  
                       for(i = 0 ; i < NCpu ; i++) (6)  
                       {  
                             pStats[3] += Open->CpuData[i].Accepted;  
                             pStats[0] += Open->CpuData[i].Received;  
                             pStats[1] += Open->CpuData[i].Dropped;  
                             pStats[2] += 0;              // Not yet supported             (7)  
                       }  
                       SET_RESULT_SUCCESS(4*sizeof(UINT));  
                       break;
```

在（1）处，`IRP Stack Pointer` 通过 `IoGetCurrentIrpStackLocation` 进行检索，该结构还包含了用户层的其它参数 。`IOCTL` 参数存储在 `FunctionCode` 变量中，在 `switch;case` 语句中用于选择将进行的操作。在这里，我们感兴趣的数值是(2) `BIOCGSTATS`。

在（3）处，检测 `OutputBufferLength` 参数，以确定是否可将数据写入（四个符号整数），若不行，则跳出 `switch;case` 语句。

在（4）处，获取用户层地址，以作为输出缓冲区。接着，在（5）处我们就可以看到漏洞本身了。该驱动程序将 16 个 0 写入用户模式下指定的地址，但并未对其做任何检测。在正常情况下，该地址是一个用户地址范围内有效的缓冲区指针，但这里可以提供一个无效地址，以触发访问异常，从而在 ring 0 下执行操作，进而导致蓝屏（BSOD）。

在（6）处，是一个循环，用于在数组中迭代添加数值。在整个循环中，除了第三个 `DWORD` 值，其它值均置 0。接下来，离开 `switch; case` 语句，继续执行其它操作。

## 漏洞利用

通过上面的讲解，现在大家应该可以利用该漏洞致使整个系统崩溃了。这仅需要发送一个 `ioctl` 去指定一个无效的内核空间地址作为输出缓冲区就可以了，比如地址 `0x80808080`。下面让我们更深入地探究一下：

通过该漏洞，我们可以在一些可写的内核地址中修改 16 bytes。在这种情况下，我们无法确切地知道是哪个数值，但不需要进一步分析，我们就可以知道第三个 `DWORD` 值总为 0。那么现在的问题就是：我们该如何在这一数值中获得代码执行的权限？

### 篡改 SSDT



# System Service Descriptor Table (SSDT)

`System Service Descriptor Table` (`SSDT`) 是内核中的一个数据结构，其中包含了函数指针列表。当一些用户模式下特定的的 API 函数想要在内核空间执行操作时，这些函数指针就会被系统服务分配器调用。例如，当在用户进程中调用 `AddAtom()` 函数时，在 `DLL` 中的代码就会负责验证一些参数，然后利用 `int 2e` 或者 `sysenter`（依赖于 `windows` 版本）切换上下文到 `ring 0`，从而通过分配表中的索引号来引用函数，接着系统服务分配器转向对应的指针重发（有时终止它，或者甚至修改它）用户模式参数。在内核调试器 `KD` 中查看一下 `SSDT`，可以发现指针指向的地址在各 `windows` 版本中始终保持不变，如下所示：

```
kd> dds poi(KeServiceDescriptorTable)
...
8050104c  805e8f86
        nt!NtAccessCheckByTypeResultListAndAuditAlarmByHandle
80501050  8060a5da nt!NtAddAtom
80501054  8060b84e nt!NtQueryBootOptions
80501058  805e0a08 nt!NtAdjustGroupsToken
8050105c  805e0660 nt!NtAdjustPrivilegesToken
80501060  805c9684 nt!NtAlertResumeThread
80501064  805c9634 nt!NtAlertThread
80501068  8060ac00 nt!NtAllocateLocallyUniqueId
8050106c  805aa088 nt!NtAllocateUserPhysicalPages
80501070  8060a218 nt!NtAllocateUuids
80501074  8059c910 nt!NtAllocateVirtualMemory
...
```

## 攻击原理

通过这个漏洞，利用我们可控制的数据去修改 `SSDT` 表中的指针，使其指向用户空间中分配的内存区域，从而发动攻击（被广泛使用的攻击方式之一）。在这种情况下，我们已经知道了作为输出缓冲区的地址，驱动程序将会写入：

* `8 bytes` 的未知数据（写入的内容实际上很明显，但我们不需要知道它）
* `4 bytes 0`
* `4 bytes` 未知数据。

由上可知，可预测值只有 4 个 `0`，但我们如何通过数值 `0` 来篡改指针呢？这里有个小技巧：就是将代码置入在页 `0` 中分配的内存里面。这个可以通过 1 中的基址来调用 `NtAllocateVirtualMemory`，因为这个函数是在低内存页（`lower page`）中分配数值，它是从 `0x0` 开始分配内存的。

```c
PVOID Addr=(PVOID)0x1;
NtAllocateVirtualMemory((HANDLE)-1, &Addr, 0, &Size, MEM_RESERVE|MEM_COMMIT|MEM_TOP_DOWN, PAGE_EXECUTE_READWRITE);
```

利用这四个 `0` 值，我们就可以篡改需要入口地址。我们只需发送 `BIOCGSTATS ioctl` 去触发漏洞，然后通过驱动程序来修改函数地址即可 - 8。之后被选择的函数指针将会被 `0` 值替换掉，从而指向我们分配的缓冲区。

```c
DeviceIoControl(hDevice, 0x9031, lpInBuffer, nInBufferSize, (Address of the selected function - 8), nOutBufferSize, &ret, NULL)
```

但是这种方法有个小问题，因为我们破坏了四个连续的函数指针，因此我们必须精心地挑选好要被修改的函数。这些函数被调用的次数要少，而且并不是很重要的函数才行。我们可以附加调试器去设置一些断点，然后去查找那些最不经常被调用的函数。

最后我们只需调用用户模式下对应的函数，但该函数需要使系统服务分配器去调用 `kernel-patched-pointer` 才行。通过这种方法，我们就可以获得我们构造的用户层代码的执行权限了。正常情况下，`0x0` 地址中的代码将会以已知的方式去提升指定进程的执行权限，但这已经不属于本文的主题了。

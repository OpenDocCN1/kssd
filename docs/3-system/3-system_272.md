

# 关于 Int 2E

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-682.htm`](https://www.pediy.com/kssd/pediy08/pediy8-682.htm)

下面是反汇编 ntdll.dll 的 NtCreateEvent 部分 NtCreateEvent 调用了 int 2E  
  
Exported fn(): NtCreateEvent - Ord:005Ah  
Exported fn(): ZwCreateEvent - Ord:02E2h  
:77F83219 B81E000000         mov eax, 0000001E  
:77F8321E 8D542404           lea edx, dword ptr \[esp+04\]  
:77F83222 CD2E             int 2E  
:77F83224 C21400             ret 0014  
  
  
  
int 2e 的使用方法:  
  
mov eax, service\_id  
lea edx, service\_param  
int 2e  
  
  
Windows 2000 int 2e 功能表  
共 248 个  
EAX = function number  
EDX = address of parameter block  
0x0   AcceptConnectPort  
0x1   AccessCheck  
0x2   AccessCheckAndAuditAlarm  
0x3   AccessCheckByType  
0x4   AccessCheckByTypeAndAuditAlarm  
0x5   AccessCheckByTypeResultList  
0x6   AccessCheckByTypeResultListAndAuditAlarm  
0x7   AccessCheckByTypeResultListAndAuditAlarmByHandle  
0x8   AddAtom  
0x9   AdjustGroupsToken  
0xa   AdjustPrivilegesToken  
0xb   AlertResumeThread  
0xc   AlertThread  
0xd   AllocateLocallyUniqueId  
0xe   AllocateUserPhysicalPages  
0xf   AllocateUuids  
0x10 AllocateVirtualMemory  
0x11 AreMappedFilesTheSame  
0x12 AssignProcessToJobObject  
0x13 CallbackReturn  
0x14 CancelIoFile  
0x15 CancelTimer  
0x16 CancelDeviceWakeupRequest  
0x17 ClearEvent  
0x18 Close  
0x19 CloseObjectAuditAlarm  
0x1a CompleteConnectPort  
0x1b ConnectPort  
0x1c Continue  
0x1d CreateDirectoryObject  
0x1e CreateEvent                 请注意这里  
  
  
  
  
Ntdll.dll 通过软件中断 int 2Eh 进入 ntoskrnl.exe，就是通过中断门切换 CPU 特权级。比如 kernel32.dll 导出的函数 DeviceIoControl()实际上调用 ntdll.dll 中导出的 NtDeviceIoControlFile()，反汇编一下这个函数可以看到，EAX 载入 magic 数 0x38，实际上是系统调用号，然后 EDX 指向堆栈。目标地址是当前堆栈指针 ESP+4，所以 EDX 指向返回地址后面一个，也就是指向在进入 NtDeviceIoControlFile()之前存入堆栈的东西。事实上就是函数的参数。下一个指令是 int 2Eh，转到中断描述符表 IDT 位置 0x2E 处的中断处理程序。  
  
反编汇这个函数得到：  
  
mov eax, 38h  
  
lea edx, \[esp+4\]  
  
int 2Eh  
  
ret 28h  
  
当然 int 2E 接口不仅仅是简单的 API 调用调度员，他是从用户模式进入内核模式的 main gate。  
  
W2k Native API 由 248 个这么处理的函数组成，比 NT 4.0 多了 37 个。可以从 ntdll.dll 的导出列表中很容易认出来：前缀 Nt。Ntdll.dll 中导出了 249 个，原因在于 NtCurrentTeb()为一个纯用户模式函数，所以不需要传给内核。令人惊奇的是，仅仅 Native API 的一个子集能够从内核模式调用。而另一方面，ntoskrnl.exe 导出了两个 Nt\*符号，它们不存在于 ntdll.dll 中: NtBuildNumber, NtGlobalFlag。它们不指向函数，事实上，是指向 ntoskrnl.exe 的变量，可以被使用 C 编译器 extern 关键字的驱动模块导入。 Ntdll.dll 和 ntoskrnl.exe 中都有两种前缀 Nt\*,Zw\*。事实上 ntdll.dll 中反汇编结果两者是一样的。而在 ntoskrnl.exe 中，nt 前缀指向真正的代码，而 zw 还是一个 int 2Eh 的 stub。也就是说 zw\*函数集通过用户模式到内核模式门传递的，而 Nt\*符号直接指向模式切换以后的代码。Ntdll.dll 中的 NtCurrentTeb()没有相对应的 zw 函数。Ntoskrnl 并不导出配对的 Nt/zw 函数。有些函数只以一种方式出现。  
  
2Eh 中断处理程序把 EAX 里的值作为查找表中的索引，去找到最终的目标函数。这个表就是系统服务表 SST，C 的结构 SYSTEM\_SERVICE\_TABLE 的定义如下:清单也包含了结构 SERVICE\_DESCRIPTOR\_TABLE 中的定义，为 SST 数组第四个成员，前两个有着特别的用途。  
  
typedef NTSTATUS (NTAPI \*NTPROC) ( ) ；  
  
typedef NTPROC \*PNTPROC；  
  
#define NTPROC\_ sizeof (NTPROC)  
  
typedef struct \_SYSTEM\_SERVICE\_TABLE  
  
{ PNTPROC ServiceTable; // 这里是入口指针数组  
  
PDWORD CounterTable; // 此处是调用次数计数数组  
  
DWORD ServiceLimit ; // 服务入口的个数  
  
PBYTE ArgumentTable; // 服务参数字节数的数组  
  
) SYSTEM\_SERVICE\_TABLE ,  
  
\* PSYSTEM\_SERVICE\_TABLE ,  
  
\* \* PPSYSTEM\_SERVICE\_TABLE ;  
  
/ / \_ \_ \_ \_ \_ \_ \_ \_ \_ \_ \_ \_  
  
typedef struct \_SERVICE\_DESCRIPTOR\_TABLE  
  
{ SYSTEM\_SERVICE\_TABLE ntoskrnl ; // ntoskrnl 所实现的系统服务，本机的 API}  
  
SYSTEM\_SERVICE\_TABLE win32k; // win32k 所实现的系统服务  
  
SYSTEM\_SERVICE\_TABLE Table3; // 未使用  
  
SYSTEM\_SERVICE\_TABLE Table4; // 未使用  
  
} SERVICE\_DESCRIPTOR\_TABLE ,  
  
\* PSERVICE\_DESCRIPTOR\_TABLE,  
  
\* PPSERVICE\_DESCRIPTOR\_TABLE ；  
  
ntoskrnl 通过 KeServiceDescriptorTable 符号,导出了主要 SDT 的一个指针。内核维护另外的一个 SDT，就是 KeServiceDescriptorTableShadow。但这个符号没有导出。要想在内核模式组件中存取主要 SDT 很简单，只需两行 C 语言的代码:  
  
extern PSERVICE\_DESCRIPTOR\_TABLE KeServiceDescriptorTable;  
  
PSERVICE\_DESCRIPTOR\_TABLE psdt= KeServiceDescriptorTable;  
  
NTPROC 为本机 API 的方便的占位符，他类似于 Win32 编程中的 PROC。Native API 正常的返回应该是一个 NTSTATUS 代码，他使用 NTAPI 调用约定，它和\_stdcall 一样。ServiceLimit 成员有在 ServiceTable 数组里找到的入口数目。在 2000 下，默认值是 248。ArgumentTable 为 BYTEs 的数组，每一个对应于 ServiceTable 的位置并显示了在调用者堆栈里的参数比特数。这个信息与 EDX 结合，这是内核从调用者堆栈 copy 参数到自己的堆栈所需的。 CounterTable 成员在 free buid 的 2000 中并没有使用到，在 debug build 中，这个成员指向代表所有函数使用计数的 DWORDS 数组，这个信息能用于性能分析。  
  可以使用这个命令来显示:dd KeServiceDescriptorTable，调试器把此符号解析为 0x8046e0c0。只有前四行是最重要的，对应那四个 SDT 成员。  
  运行这个命令:ln 8046e100，显示符号是 KeServiceDescriptorTableShadow，说明第五个开始确实为内核维护的第二个 SDT。主要的区别在于后一个包含了 win32k.sys 的入口，前一个却没有。在这两个表中，Table3 与 Table4 都是空的。Ntoskrnl.exe 提供了一个方便的 API 函数。这个函数的名字为：  
  
  KeAddSystemServiceTable  
此函数去填充这些位置。  
  
2Eh 的中断处理标记是 KisystemService()。这也是 ntoskrnl.exe 没有导出的内部的符号，但包含在 2k 符号文件中。关于 KisystemService 的操作如下：  
  
1 从当前的线程控制块检索 SDT 指针  
  
2 决定使用 SDT 中 4 个 SST 的其中一个。通过测试 EAX 中递送 ID 的第 12 和 13 位来决定。ID 在 0x0000-0x0fff 的映射至 ntoskrnl 表格，ID 在  
  
0x1000 与 0x1ffff 的分配给 win32k 表格。剩下的 0x2000-0x2ffff 与  
  
0x3000-0x3ffff 则是 Table3 和 Table4 保留。  
  
3 通过选定 SST 中的 ServiceLimit 成员检查 EAX 的 0－11 位。如果 ID 超过了范围，返回错误代码为 STATUS\_INVALID\_SYSTEM\_SERVICE。  
  
4 检查 EAX 中的参数堆栈指针与 MmUserProbeAddress。这是一个 ntoskrnl 导出的全局变量。通常等于 0x7FFF0000，如果参数指针不在这个地址之下，返回 STATUS\_ACCESS\_VIOLATION。  
  
5 查找 ArgumentTable 中的参数堆栈的字节数，从调用者的堆栈 copy 所有的参数至当前内核模式堆栈。  
  
6 搜索 serviceTable 中的服务函数指针，并调用这个函数。  
  
7 控制转到内部的函数 KiserviceExit，在此次服务调用返回之后。 

 4dd466d0e1e0cc123afbe504da2462ca
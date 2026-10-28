
```
sc create E1_OSVersion type= kernel binPath= "\\vmware-host\Shared Folders\MyDriver\x64\Debug\E1_OSVersion.sys"
sc start E1_OSVersion
sc stop E1_OSVersion
```

## 3 内核编程基础

研究一些内核的API、结构和定义，以及一些驱动程序中的机制

### 3.1 通用内核编程指南

用户模式和内核模式调试的重要区别

|  | 用户模式 | 内核模式 |
| --- | --- | --- |
| 未处理异常 | 进程崩溃 | 系统崩溃 |
| 终止 | 当进程终止，所有内存和资源都会被自动释放 | 当驱动卸载，如果没有手动释放，会造成泄露直到重启 |
| 返回值 | API错误有时候会忽略 | 应该不忽略任何错误 |
| IRQL | 总是 `PASSIVE_LEVEL` (0) | 可能为更高 |
| 错误代码 | 通常只会影响本进程 | 影响整个系统 |
| 测试和调试 | 通常在开发机器上调试 | 需要双机调试 |
| 库（Lib） | 可以使用C/C++库（如STL、boost） | 大多数标准库无法使用 |
| 异常处理 | 可以使用C++异常或SEH | 只能使用SEH |
| C++支持 | 完全的C++支持 | 不支持C++ runtime |

#### 3.1.1 未处理异常

未处理异常会导致蓝屏，原因是防止继续执行代码、对系统造成不可逆转的伤害

内核代码不应该跳过任何细节或错误检查

#### 3.1.2 终止

如果驱动程序卸载时仍保留分配的内存或打开的内核句柄，这些资源不会自动释放，只会在下次系统启动时释放

原因是驱动程序可以分配一些缓冲区，然后将其传递给另一个与之合作的驱动程序

#### 3.1.3 函数返回值

忽略内核API的返回值很危险，应该总是检查返回值

#### 3.1.4 IRQL

中断请求级（Interrupt Request Level, IRQL）通常为0

用户模式下始终为0，内核模式下大部分时间为0

#### 3.1.5 C++使用

没有C++ runtime

一些不支持的C++特性：

- 不支持`new`和`delete`，这正常是在用户模式堆分配的

- 不会调用具有非默认构造函数的全局变量

  - 避免在构造函数中使用代码，创建一些要显式调用的`Init`函数

  - 仅将指针分配为全局变量，动态创建实例

- 不支持C++异常处理（`try`、`catch`、`throw`）

- 不可使用标准C++库，如`std::vector<>`、`std::wstring`等

一些支持的C++特性：

- `nullptr`关键字

- `auto`关键字

- 模板将在有意义时使用

- 重载`new`和`delete`运算符

- 构造函数和析构函数，尤其是用于构建 RAII 类型

#### 3.1.6 测试和调试

内核调试需要双机调试，一台作为调试者、另一台作为被调试者运行驱动程序

### 3.2 Debug vs. Release 生成

内核术语是 Checked（Debug）和 Free（Release）

Debug意味着可以使用DBG符号

### 3.3 内核API

内核API常用前缀的意义：

- Ex：一般执行函数

- Ke：一般内核函数

- Mm：内存管理

- Rtl：一般运行时库

- FsRtl：文件系统运行时库

- Flt：文件系统迷你过滤库

- Ob：对象管理

- Io：I/O管理

- Se：安全

- Ps：进程结构

- Po：电源管理

- Wmi：Windows管理工具

- Zw：native API 包装

- Hal：硬件抽象层

- Cm：配置管理器（注册表）

Nt前缀的内核函数对应`NtDll.Dll`的函数，会根据`KTHREAD`结构的标记（调用者是否来自内核）对参数进行检查

Zw前缀的内核函数先将调用者模式设为`KernelMode(0)`，然后调用Nt前缀的内核函数

### 3.4 函数和错误代码

可以在`ntstatus.h`中找到`NTSTATUS`值的定义

大多数代码并不关心错误具体是什么，仅测试最高位即可，可以使用`NT_SUCCESS`宏

当返回到用户层时，会由`STATUS_xxx`转成`ERROR_yyy`，用户模式通过`GetLastError`可以得到这些错误

通常遇到错误时，会返回相同的 NTSTATUS 到调用函数

### 3.5 字符串

内核使用`UNICODE_STRING`

```c
typedef struct _UNICODE_STRING {
    USHORT Length;
    USHORT MaximumLength;
    PWCH Buffer;
} UNICODE_STRING;
typedef UNICODE_STRING *PUNICODE_STRING;
typedef const UNICODE_STRING *PCUNICODE_STRING;
```

`Length`是字符串的字节数（不包括`\x00\x00`结束符）

`MaximumLength`是不需要重新分配内存的情况下、字符串字节数上限

需要注意的是，`UNICODE_STRING`并**不总是有`\x00\x00`结尾**

一些常用的字符串操作函数：

- `RtlInitUnicodeString`

- `RtlCopyUnicodeString`

- `RtlCompareUnicodeString`

- `RtlEqualUnicodeString`

- `RtlAppendUnicodeStringToString`

- `RtlAppendUnicodeToString`

### 3.6 动态内存分配（S2）

内核提供两种通用内存池（general memory pools）给驱动使用：

- 页池（Paged pool）：可能会被换出（paged out）的内存池

- 非页池（Non Paged Pool）：一直在RAM中的内存池

枚举类型`POOL_TYPE`表示池类型，只有三种是可以用于驱动的：

`PagedPool`、`NonPagedPool`、`NonPagedPoolNx`

（non-page pool没有可执行权限）

常用内存池函数：

- `ExAllocatePool`（已过时，将被下面的函数取代）

- `ExAllocatePoolWithTag`

- `ExAllocatePoolWithQuotaTag`

- `ExFreePool`

tag是4字节的值

可以在`PoolMon`（WDK的Windows Kits中）中观察到有tag的内存池（tag以大端序字符串显示）

给`ustring`分配页池内存：

```c
UNICODE_STRING strA;
int length;
// allocate
strA.Buffer = (WCHAR*)ExAllocatePoolWithTag(PagedPool,
        length, 'dcba');
if (strA.Buffer == nullptr) {
    KdPrint(("Failed to allocate memory\n"));
    return STATUS_INSUFFICIENT_RESOURCES;
}
strA.MaximumLength = length;
```

```shell
sc create S2_DynMemAlloc type= kernel binPath= "\\vmware-host\Shared Folders\MyDriver\x64\Debug\S2_DynMemAlloc.sys"
sc start S2_DynMemAlloc
sc stop S2_DynMemAlloc
```

### 3.7 链表

内核使用循环双向链表：

```c
typedef struct _LIST_ENTRY {
    struct _LIST_ENTRY *Flink;
    struct _LIST_ENTRY *Blink;
} LIST_ENTRY, *PLIST_ENTRY;
```

`CONTAINING_RECORD`宏执行适当的偏移计算并转换为实际数据类型

`CONTAINING_RECORD(pvoid, type, entry_member_name)`

```c
struct MyDataItem {
    // some data members
    LIST_ENTRY Link;
    // more data members
};

MyDataItem* GetItem(LIST_ENTRY* pEntry) {
    return CONTAINING_RECORD(pEntry, MyDataItem, Link);
}
```

常用链表函数（时间复杂度都是常数）：

- `InitializeListHead`

- `InsertHeadList`

- `InsertTailList`

- `IsListEmpty`

- `RemoveHeadList`

- `RemoveTailList`

- `RemoveEntryList`

- `ExInterlockedInsertHeadList`

- `ExInterlockedInsertTailList`

- `ExInterlockedRemoveHeadList`

后三个关于自旋锁，在第6章详细讨论

### 3.8 驱动对象（The Driver Object）

常用major function代码：

- `IRP_MJ_CREATE` (0)

- `IRP_MJ_CLOSE` (2)

- `IRP_MJ_READ` (3)

- `IRP_MJ_WRITE` (4)

- `IRP_MJ_DEVICE_CONTROL` (14)

- `IRP_MJ_INTERNAL_DEVICE_CONTROL` (15)

- `IRP_MJ_PNP` (31)

- `IRP_MJ_POWER` (22)

`MajorFunction`数组由内核初始化指向内核内部例程`IopInvalidDeviceRequest`，该例程直接返回失败，表示不支持该操作

### 3.9 设备对象（Device Objects）

驱动通过设备与r3代码通信，驱动应该至少创建一个设备对象并为其命名

`CreateFile`可以打开设备，第一个参数为设备对象名称

打开文件或设备的句柄会创建内核结构 `FILE_OBJECT` 的实例，这是个半文档化的结构。

更准确的说，`CreteFile`接受一个`symbolic link`（符号链接）

对象管理器中名为`??`的目录下的符号链接都可被用户模式代码通过`CreateFile`或`Createfile2`调用

可以通过`WinObj`查看（`WinObj`中目录名为`Global??`）

使用符号链接的`CreateFile`的文件名（第一个参数），必须加上前缀`\\.\`（c++中是`"\\\\.\\"`）

如果创建了多个设备对象，将形成一个单向链表，添加设备时是头插法，所以第一个创建的设备在链表的最后

## 4 驱动从头到尾（Driver from Start to Finish）（S3）

将完成一个完整的驱动及客户端程序，利用驱动完成只能在内核模式下完成的功能（设置任意级别的线程优先级）

### 4.1 绪论

`线程优先级 = 进程优先级 + 相对线程优先级`

用户模式下，设置进程优先级可以用`SetPriorityClass`，共有6个级别

设置相对线程优先级可以用`SetThreadPriority`，共有7个级别

下面是线程优先级合法值的表（通过windows api设置），据别的书说是个未文档化的东西，windows不建议开发时考虑线程优先级，该表的值随windows版本变化可能发生改变

| 进程优先级 | -Sat | -2 | -1 | 0 | +1 | +2 | +sat |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Idle(low) | 1 |  |  | 4 |  |  | 15 |
| Below Normal | 1 |  |  | 6 |  |  | 15 |
| Normal | 1 |  |  | 8 |  |  | 15 |
| Above Normal | 1 |  |  | 10 |  |  | 15 |
| High | 1 |  |  | 13 |  |  | 15 |
| Real-time | 16 |  |  | 24 |  |  | 31 |

进程优先级枚举：级别+`_PRIORITY_CLASS`

线程优先级枚举：`THREAD_PRIORITY_`+级别

### 4.2 驱动初始化

大多数驱动需要在`DriverEntry`中做如下操作：

- 设置`Unload`例程

- 设置驱动支持的调度例程

- 创建一个设备对象

- 创建一个指向设备对象的符号链接

所有驱动必须支持`IRP_MJ_CREATE`和`IRP_MJ_CLOSE`，不然无法打开一个驱动的设备的句柄，通常这两个调度例程是相同的

调度例程的函数原型：`NTSTATUS Function(_In_ PDEVICE_OBJECT DeviceObject, _In_ PIRP Irp)`

#### 4.2.1 将信息传给驱动

用户模式客户端可用的三个基础函数：`WriteFile`、`ReadFile`、`DeviceIoControl`

#### 4.2.2 客户端/驱动程序通信协议

必须使用`CTL_CODE`宏来构建控制代码

```c
#define CTL_CODE( DeviceType, Function, Method, Access ) (                 \
((DeviceType) << 16) | ((Access) << 14) | ((Function) << 2) | (Method) \
)
```

- DeviceType：设备类型标识，`FILE_DEVICE_xxx`，第三方应以`0x8000`开头

- Function：指示特定操作的升序数字，第三方应该以`0x800`开头

- Method：指示客户端提供的输入和输出缓冲区如何传递给驱动程序（将在第6章详细讨论）

- Access：指示对驱动来说这个操作是什么？

示例：

```c
#define MY_DEVICE 0x800
#define IOCTL_MY_OP CTL_CODE(\
MY_DEVICE, 0x800, METHOD_NEITHER, FILE_ANY_ACCESS)
```

#### 4.2.3 创建一个设备对象

**创建设备名：**

在创建一个设备对象前，需要先创建一个`UNICODE_STRING`存储内部设备名称

下面是两种初始化方式：

```c
// plan A
UNICODE_STRING devName = RTL_CONSTANT_STRING(L"\\Device\\YourName");

// plan B
UNICODE_STRING devName;
RtlInitUnicodeString(&devName, L"\\Device\\YourName");
```

设备名称需要在设备对象管理器目录下

（`RtlInitUnicodeString`函数内部字符串的长度，`RTL_CONSTANT_STRING`宏在编译时计算长度）

**创建设备对象：**

创建设备对象需要调用`IoCreateDevice`

```c
NTSTATUS IoCreateDevice(
    _In_ PDRIVER_OBJECT DriverObject,
    _In_ ULONG DeviceExtensionSize,
    _In_opt_ PUNICODE_STRING DeviceName,
    _In_ DEVICE_TYPE DeviceType,
    _In_ ULONG DeviceCharacteristics,
    _In_ BOOLEAN Exclusive,
    _Outptr_ PDEVICE_OBJECT *DeviceObject);
```

**创建设备完整示例：**


> 在MSDN上搜索该函数，发现其所属模块为`kernel32.dll`，但是当你去追`kernel32`里的流程，你会发现最终都是执行了一个`return 0`。这是历史遗留问题，所以不能盲目相信MSDN。

## `KernelBase->ntdll`

在WIN7镜像中，随便找个程序拖到OD里，Ctrl+G定位至`ReadProcessMemory`函数。发现该函数已经隶属于`KernelBase`模块了。且其内部没有什么有用的代码，只调用了`ntdll`的`NtReadVirtualMemory`函数。该函数是`KernelBase`的导入函数。

![image-20210923104255726](https://bbs.kanxue.com/upload/attach/202109/867232_BWM6NUX57AG796B.jpg)

使用IDA打开`ntdll`，在导出函数选项卡中搜索`NtReadVirtualMemory`。

![image-20210923104652359](https://bbs.kanxue.com/upload/attach/202109/867232_9M7MG8CW4CVNKAM.jpg)

跟过去看一下该函数的实现。发现函数内部仅调用了`[7FFE0300]`处的函数，同时为`eax`赋值。

![image-20210923104717960](https://bbs.kanxue.com/upload/attach/202109/867232_KJZ9AQ3STWD34AS.jpg)

## 总结

至此，3环绝大部分告一段落。可以发现`ReadProcessMemory`函数的3环执行流程就是跳来跳去，然后执行一个`call [7FFE0300]`。接下来我们需要探求`7FFE0300`里面存的是什么。

# 4.实战分析-3环进入0环

## `_KUSER_SHARED_DATA`

在继续分析之前，我们需要补充一个知识点：`_KUSER_SHARED_DATA`。这是一个3环与0环共享的结构体。

在3环，它的地址为`7ffe0000`，具备只读权限。在0环，它的地址为`ffdf0000`，具备读写权限。这两个地址指向同一个物理页，只是访问权限不同。该结构会在操作系统启动时由操作系统自己填充。

结构体信息如下：

```
//0x5f0 bytes (sizeof)
struct _KUSER_SHARED_DATA
{
    ULONG TickCountLowDeprecated; //0x0
    ULONG TickCountMultiplier; //0x4
    volatile struct _KSYSTEM_TIME InterruptTime; //0x8
    volatile struct _KSYSTEM_TIME SystemTime; //0x14
    volatile struct _KSYSTEM_TIME TimeZoneBias; //0x20
    USHORT ImageNumberLow; //0x2c
    USHORT ImageNumberHigh; //0x2e
    WCHAR NtSystemRoot[260]; //0x30
    ULONG MaxStackTraceDepth; //0x238
    ULONG CryptoExponent; //0x23c
    ULONG TimeZoneId; //0x240
    ULONG LargePageMinimum; //0x244
    ULONG Reserved2[7]; //0x248
    enum _NT_PRODUCT_TYPE NtProductType; //0x264
    UCHAR ProductTypeIsValid; //0x268
    ULONG NtMajorVersion; //0x26c
    ULONG NtMinorVersion; //0x270
    UCHAR ProcessorFeatures[64]; //0x274
    ULONG Reserved1; //0x2b4
    ULONG Reserved3; //0x2b8
    volatile ULONG TimeSlip; //0x2bc
    enum _ALTERNATIVE_ARCHITECTURE_TYPE AlternativeArchitecture; //0x2c0
    ULONG AltArchitecturePad[1]; //0x2c4
    union _LARGE_INTEGER SystemExpirationDate; //0x2c8
    ULONG SuiteMask; //0x2d0
    UCHAR KdDebuggerEnabled; //0x2d4
    UCHAR NXSupportPolicy; //0x2d5
    volatile ULONG ActiveConsoleId; //0x2d8
    volatile ULONG DismountCount; //0x2dc
    ULONG ComPlusPackage; //0x2e0
    ULONG LastSystemRITEventTickCount; //0x2e4
    ULONG NumberOfPhysicalPages; //0x2e8
    UCHAR SafeBootMode; //0x2ec
    union
    {
        UCHAR TscQpcData; //0x2ed
        struct
        {
            UCHAR TscQpcEnabled:1; //0x2ed
            UCHAR TscQpcSpareFlag:1; //0x2ed
            UCHAR TscQpcShift:6; //0x2ed
        };
    };
    UCHAR TscQpcPad[2]; //0x2ee
    union
    {
        ULONG SharedDataFlags; //0x2f0
        struct
        {
            ULONG DbgErrorPortPresent:1; //0x2f0
            ULONG DbgElevationEnabled:1; //0x2f0
            ULONG DbgVirtEnabled:1; //0x2f0
            ULONG DbgInstallerDetectEnabled:1; //0x2f0
            ULONG DbgSystemDllRelocated:1; //0x2f0
            ULONG DbgDynProcessorEnabled:1; //0x2f0
            ULONG DbgSEHValidationEnabled:1; //0x2f0
            ULONG SpareBits:25; //0x2f0
        };
    };
    ULONG DataFlagsPad[1]; //0x2f4
    ULONGLONG TestRetInstruction; //0x2f8
    ULONG SystemCall; //0x300
    ULONG SystemCallReturn; //0x304
    ULONGLONG SystemCallPad[3]; //0x308
    union
    {
        volatile struct _KSYSTEM_TIME TickCount; //0x320
        volatile ULONGLONG TickCountQuad; //0x320
        ULONG ReservedTickCountOverlay[3]; //0x320
    };
    ULONG TickCountPad[1]; //0x32c
    ULONG Cookie; //0x330
    ULONG CookiePad[1]; //0x334
    LONGLONG ConsoleSessionForegroundProcessId; //0x338
    ULONG Wow64SharedInformation[16]; //0x340
    USHORT UserModeGlobalLogger[16]; //0x380
    ULONG ImageFileExecutionOptions; //0x3a0
    ULONG LangGenerationCount; //0x3a4
    ULONGLONG Reserved5; //0x3a8
    volatile ULONGLONG InterruptTimeBias; //0x3b0
    volatile ULONGLONG TscQpcBias; //0x3b8
    volatile ULONG ActiveProcessorCount; //0x3c0
    volatile USHORT ActiveGroupCount; //0x3c4
    USHORT Reserved4; //0x3c6
    volatile ULONG AitSamplingValue; //0x3c8
    volatile ULONG AppCompatFlag; //0x3cc
    ULONGLONG SystemDllNativeRelocation; //0x3d0
    ULONG SystemDllWowRelocation; //0x3d8
    ULONG XStatePad[1]; //0x3dc
    struct _XSTATE_CONFIGURATION XState; //0x3e0
};
```

其中偏移`0x300`处的成员为`SystemCall`，该成员指明了当发生系统调用时，3环进入0环走哪个函数，该成员就存储对应函数的地址。

> 由于现在的处理器均支持快速调用（`sysenter`指令），而旧处理器不支持，所以此处存储的函数地址有两种情况。后文仅演示快速调用。（中断门（`0x2E`）进0环已经在历史的长河中被替代了，有兴趣的可以了解下。）

为了查看`7FFE0300`的值，我们需要借助windbg动态调试。

在windbg中输入 `!process 0 0` 遍历所有进程，随便附加其中一个即可。

![image-20210923105840035](https://bbs.kanxue.com/upload/attach/202109/867232_GQM9F3CUYDPHURK.jpg)

输入 `dt _KUSER_SHARED_DATA 7FFE0000` 查看该结构体的成员信息。可以发现偏移`0x300`处存储的值为`0x77a470b0`。

![image-20210923112639484](https://bbs.kanxue.com/upload/attach/202109/867232_TK9AYTE8DUX23RU.jpg)

使用命令 `u 77a470b0` 查看该地址的函数。可以看到windbg帮我们识别出来是`ntdll`模块中的`KiFastSystemCall`函数。回到IDA中，定位至该函数，观察函数执行流程。

![image-20210923110011447](https://bbs.kanxue.com/upload/attach/202109/867232_UDFSTZUCP3GG6DD.jpg)

## `KiFastSystemCall`

可以发现该函数只有两行代码：

```
mov edx, esp
sysenter
```

`sysenter`将会进入0环。

![image-20210923110211147](https://bbs.kanxue.com/upload/attach/202109/867232_FTXHQH3MV36H4SJ.jpg)

## `sysenter`指令

在保护模式章节中，我们知道3环进0环必备的几个条件：

- 权限发生切换时，需要新的`CS`
- 与`CS`保持权限一致，也需要新的`SS`
- 0环的堆栈：`ESP`
- 0环代码从哪执行：`EIP`

在旧CPU中，使用中断门（`0x2E`）进行系统调用，需要查询`TSS`来获取这几个寄存器的值。内存查询是很慢的。

而`sysenter`指令将会直接从`MSR`寄存器中读取并写入至`CS`、`SS`、`EIP`、`ESP`中，速度很快，因此称为快速调用。

## `MSR`寄存器

`MSR`寄存器很大，存了很多的数据。

在偏移`0x174`处，存储着系统调用时0环的`CS`。

在偏移`0x175`处，存储着系统调用时0环的`ESP`。

在偏移`0x176`处，存储着系统调用时0环的`EIP`。

`SS`的值通过`CS+8`得到（具体原因参考保护模式章节，`CS`与`SS`的段描述符是相邻的）。

通过指令`RDMSR`/`WRMSR`可以对`MSR`寄存器进行读写。

在windbg中查看`CS` `ESP` `EIP`的值：

![image-20210923132008269](https://bbs.kanxue.com/upload/attach/202109/867232_7XA4H9XSEPDZY8E.jpg)

在windbg中输入命令 `u 83e420c0` 查看这个函数

![image-20210923132143813](https://bbs.kanxue.com/upload/attach/202109/867232_H3EE9K6YUT6X8PK.jpg)

可以得知当我们在3环调用`sysenter`指令时，将会执行内核模块（`ntoskrnl`、`ntkrnlpa`）中的`KiFastCallEntry`函数

## 总结

当发生系统调用时，若CPU支持快速调用，则走`KiFastSystemCall`函数，内部执行`sysenter`指令。并在0环执行`KiFastCallEntry`函数。同时携带了`edx`（3环堆栈）、`eax`（系统调用号）

# 5.前置知识-内核结构体

> 以下结构体均在内核空间，3环无法直接访问

## `TrapFrame`

在进入0环时，该结构体用于保存3环寄存器。由windows进行维护。

结构体成员：

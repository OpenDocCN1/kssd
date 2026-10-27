
# [原创]内核学习-系统调用上

双机调试配置参看我的另一篇文章[[原创\]内核学习-双机调试环境搭建-软件逆向-看雪论坛-安全社区|安全招聘|bbs.pediy.com](https://bbs.pediy.com/thread-268519.htm)，基于xp sp3系统

# 系统调用的学习

![在这里插入图片描述](upload/attach/202107/921642\_KZQ25B2F9JP37TP.jpg)

分析`ReadProcessMemory`了解系统调用

## 一：分析`ReadProcessMemory`

ida打开`kernel32.dll`（`ReadProcessMemory`函数在这个dll里）

![在这里插入图片描述](upload/attach/202107/921642\_S3HSG2AAZAXEN7Q.jpg)

参数压栈，`call NTReadVirtualMemory`，跳转至`loc_7C802204`

`loc_7C802204`调用了`sub_7C8093FD`

![在这里插入图片描述](upload/attach/202107/921642\_WJ8S28UXPEUJ2W9.jpg)

`sub_7C8093FD`内部调用了另一个函数`RtlNtStatusToDosError`

![在这里插入图片描述](upload/attach/202107/921642\_K9273XGPQYU2C59.jpg)

这个函数的作用是设置错误号

从`sub_7C8093FD`出来后，将`eax`清零，跳转到`loc_7C8021F9`

![在这里插入图片描述](upload/attach/202107/921642\_AV3K9K7S7BEU859.jpg)

返回

![](upload/attach/202107/921642\_Z3N6Q4GQMWPYNY3.jpg)

当调用`ReadProcessMemory`失败时，返回结果为0，若`NtReadVirtualMemory`返回结果大于等于0，返回结果为1

`ReadProcessMemory`函数在`kernel32.dll`中只是调用了`NtReadVirtualMemory`，然后设置了返回值

## 二：分析`NtReadVirtualMemory`

在`ntdll.dll`中，`Ntdll.dll`:大多数API都会通过这个DLL**进入内核**(0环)

![在这里插入图片描述](upload/attach/202107/921642\_DF2P4Z2PNHPZATU.jpg)

![在这里插入图片描述](upload/attach/202107/921642\_WCZ2ANXR66BH9WZ.jpg)

```
.text:7C92D9E0 mov eax, 0BAh 系统调用号，对应操作系统内核中某个函数的编号
.text:7C92D9E5 mov edx, 7FFE0300h 函数地址，该函数决定了我们用什么方式进0环
.text:7C92D9EA call dword ptr [edx]
.text:7C92D9EC retn 14h
```

真正读取进程内存的函数在**0环**实现，我们所用的函数只是系统提供给我们的**函数接口**

## 三：进入0环的两种方式

![在这里插入图片描述](upload/attach/202107/921642\_7W4KBZKUW3U3KR4.jpg)

### 基本概念：

#### `_KUSER_SHARED_DATA`结构体

```
nt!\_KUSER\_SHARED\_DATA
+0x000 TickCountLow : Uint4B
+0x004 TickCountMultiplier : Uint4B
+0x008 InterruptTime : \_KSYSTEM\_TIME
+0x014 SystemTime : \_KSYSTEM\_TIME
+0x020 TimeZoneBias : \_KSYSTEM\_TIME
+0x02c ImageNumberLow : Uint2B
+0x02e ImageNumberHigh : Uint2B
+0x030 NtSystemRoot : [260] Uint2B
+0x238 MaxStackTraceDepth : Uint4B
+0x23c CryptoExponent : Uint4B
+0x240 TimeZoneId : Uint4B
+0x244 Reserved2 : [8] Uint4B
+0x264 NtProductType : \_NT\_PRODUCT\_TYPE
+0x268 ProductTypeIsValid : UChar
+0x26c NtMajorVersion : Uint4B
+0x270 NtMinorVersion : Uint4B
+0x274 ProcessorFeatures : [64] UChar
+0x2b4 Reserved1 : Uint4B
+0x2b8 Reserved3 : Uint4B
+0x2bc TimeSlip : Uint4B
+0x2c0 AlternativeArchitecture : \_ALTERNATIVE\_ARCHITECTURE\_TYPE
+0x2c8 SystemExpirationDate : \_LARGE\_INTEGER
+0x2d0 SuiteMask : Uint4B
+0x2d4 KdDebuggerEnabled : UChar
+0x2d5 NXSupportPolicy : UChar
+0x2d8 ActiveConsoleId : Uint4B
+0x2dc DismountCount : Uint4B
+0x2e0 ComPlusPackage : Uint4B
+0x2e4 LastSystemRITEventTickCount : Uint4B
+0x2e8 NumberOfPhysicalPages : Uint4B
+0x2ec SafeBootMode : UChar
+0x2f0 TraceLogging : Uint4B
+0x2f8 TestRetInstruction : Uint8B
+0x300 SystemCall : ntdll.dll!KiFastSystemCall()/ ntdll.dll!KiIntSystemCall()
+0x304 SystemCallReturn : Uint4B
+0x308 SystemCallPad : [3] Uint8B
+0x320 TickCount : \_KSYSTEM\_TIME
+0x320 TickCountQuad : Uint8B
+0x330 Cookie : Uint4B
```

该数据结构是三环与零环的共享内存。

它们使用固定的地址值映射，`_KUSER_SHARED_DATA`结构区域在**User层**和**Kernel层**地址分别为：

`User`：`0x7ffe0000`

`Kernnel`：`0xffdf0000`

指向的是同一个物理页，但是在user层是只读，在kernel层是可读可写的。

此结构体由操作系统负责初始化，其偏移`0x300`处有一个`SystemCall`属性，是个函数指针。

#### `sysenter/sysexit`指令

`sysenter`指令可用于特权级 3 的用户代码调用特权级 0 的系统内核代码，而`SYSEXIT`指令则用于特权级 0 的系统代码返回用户空间中。`sysenter`指令可以在 3，2，1 这三个特权级别调用（Linux 中只用到了特权级 3），而`SYSEXIT`指令只能从特权级 0 调用。

描述：当操作系统初始化时，其会通过`CPUID`指令查询当前CPU是否支持快速调用，如果支持则这里填写通过快速调用进入零环，否则通过中断进入零环。

可以设置`eax=1`，执行`cpuid`指令，处理器的特征信息被放在`ecx`和`edx`寄存器中，其中`edx`包含了一个`SEP`位（第11位），该位指明了当前处理器知否支持`sysenter/sysexit`指令，`SEP=1`，说明当前CPU支持`sysenter` / `sysexit`指令。以此来自行查看前处理器是否支持`sysenter/sysexit`指令。

当CPU支持快读调用，`SystemCall`指向`ntdll.dll!KiFastSystemCall()`

当CPU不支持快速调用，`SystemCall`指向`ntdll.dll!KiIntSystemCall()`

### R3------>R0

1. CS的权限由3变为0，意味着需要新的`CS`
2. SS与CS的权限永远一致，需要新的`SS`
3. 权限发生切换的时候，堆栈也一定会切换，需要新的`ESP`
4. 进0环后会修改`EIP`

### （1）`int 2e`

#### 分析`KiIntSystemCall`

![](upload/attach/202107/921642\_SCJSBNC4BZP8Z84.jpg)

所有的API进内核时，统一的中断号为`0x2e`

在执行`KiIntSystemCall`函数前，系统调用号已被写入`EAX`。

#### 分析 `INT 0x2E`

在`IDT`表中找到`0x2E`号门描述符：`0x2E`右移3位（或者乘8）得到`0x170`，加上`r idtr`获取的`IDT`基址，可以计算出`0x2E`号中断对应的描述符：``8053ee00`0008e481``

门描述符：``8053ee00`0008e481``

指向地址：`8053e481`

![](upload/attach/202107/921642\_CBX6AK3RGSVCGRZ.jpg)

![](upload/attach/202107/921642\_RMEYCS33CEHX4R7.jpg)

> `CS`：门描述符的**段选择子**部分（`0008`）（系统代码段）

查看`eip`

```
kd> u 8053e481
```

![在这里插入图片描述](upload/attach/202107/921642\_NYCTP4SKE7V2UAB.jpg)

![](upload/attach/202107/921642\_9UVNSAWPCQWTJNQ.jpg)

![](upload/attach/202107/921642\_H9V62ASUXHMXB6K.jpg)

`TSS`描述符是``80008b04`200020ab``，所以`TSS`的地址就是`80042000`

![](upload/attach/202107/921642\_63NMSTZYSWPY59B.jpg)

所以`ESP = 8054acf0, SS = 0010`

小结：

> 1. 固定中断号为`0x2E`
> 2. `CS`/`EIP`由**门描述符**提供，`ESP`/`SS`由`TSS`提供
> 3. 进入0环后执行的内核函数：`NT!KiSystemService`

#### 进0环后，**原来的寄存器**存在哪里？

进入0环后执行的内核函数`NT!KiSystemService`

![](upload/attach/202107/921642\_W27S6KZKWKUH92S.jpg)

### （2）`sysenter`

![](upload/attach/202107/921642\_8N757N4AGS6Q2SU.jpg)

> `mov edx,esp` 三环栈顶值传给`edx`，此时`eax`中已经是系统调用号
> `sysenter`指令 寄存器数据传递

`sysenter`是从`MSR`寄存器里读取`CS`,`ESP`,`EIP`（`MSR`寄存器里的值，则是系统启动时就已经填好）

相关的`MSR`寄存器的值

![](upload/attach/202107/921642\_P8XFA6AND975G28.jpg)

可以通过`RDMSR`/`WRMST`来进行读写（操作系统使用`WRMST`写该寄存器）：

```
kd> rdmsr 174 //查看CS
kd> rdmsr 175 //查看ESP
kd> rdmsr 176 //查看EIP
```

查看`eip`所在地址的反汇编：

![](upload/attach/202107/921642\_5KVG3ZP5QVBN4JK.jpg)

![](upload/attach/202107/921642\_C27MDPHPA72MDHV.jpg)

`SS` = `IA32_SYSENTER_CS` + `8`

## 四：小结

3环进0环的两种方式，分别是中断门和快速调用，CPU支持快速调用，那么`_KUSER_SHARED_DATA`结构体的`SystemCall`属性指向的函数是`KiFastSystemCall`，执行`KiFastSystemCall`，使用快速调用的方式进0环；如果不支持，那么`SystemCall`指向的函数是`KiIntSystemCall`，执行`KiIntSystemCall`，使用中断门的方式进0环。

`KiFastSystemCall`：![](upload/attach/202107/921642\_8N757N4AGS6Q2SU.jpg)`KiIntSystemCall`：

![](upload/attach/202107/921642\_SCJSBNC4BZP8Z84.jpg)

两个函数的共同点：

1.更改4个寄存器：`SS` `CS` `EIP` `ESP`，进入0环。

两个函数的区别：

1.快速调用不需要访问内存，而中断门需要读`TSS`和`IDT`表

2.`int 0x2e`和`sysenter`指令进0环后，分别调用了两个函数`KiSystemService`和`KiFastCallEntry`。

## 五：填充`_KTRAP_FRAME`结构体

### 中断门

`int 0x2e`进0环后调用函数`KiSystemService`

#### 分析`KiSystemService`

`_Trap_Frame`和0环栈密切相关。用户定义中断进0环，涉及提权时，CPU会把5个寄存器的值压入0环堆栈。`EIP`、`CS`、`SS`、`EFLAG`、`ESP`

线程切换时会修改`TSS`表，确保每个线程执行时，`TSS`里的`ESP`,`SS`都对应当前线程。

基本概念：

`_KTRAP_FRAME`

```
nt!\_KTRAP\_FRAME
+0x000 DbgEbp : Uint4B
+0x004 DbgEip : Uint4B
+0x008 DbgArgMark : Uint4B
+0x00c DbgArgPointer : Uint4B
+0x010 TempSegCs : Uint4B
+0x014 TempEsp : Uint4B
+0x018 Dr0 : Uint4B
+0x01c Dr1 : Uint4B
+0x020 Dr2 : Uint4B
+0x024 Dr3 : Uint4B
+0x028 Dr6 : Uint4B
+0x02c Dr7 : Uint4B
+0x030 SegGs : Uint4B
+0x034 SegEs : Uint4B
+0x038 SegDs : Uint4B
+0x03c Edx : Uint4B
+0x040 Ecx : Uint4B
+0x044 Eax : Uint4B
+0x048 PreviousPreviousMode : Uint4B
+0x04c ExceptionList : Ptr32 \_EXCEPTION\_REGISTRATION\_RECORD
+0x050 SegFs : Uint4B
+0x054 Edi : Uint4B
+0x058 Esi : Uint4B
+0x05c Ebx : Uint4B
+0x060 Ebp : Uint4B
+0x064 ErrCode : Uint4B
+0x068 Eip : Uint4B
+0x06c SegCs : Uint4B
+0x070 EFlags : Uint4B
+0x074 HardwareEsp : Uint4B
+0x078 HardwareSegSs : Uint4B
+0x07c V86Es : Uint4B
+0x080 V86Ds : Uint4B
+0x084 V86Fs : Uint4B
+0x088 V86Gs : Uint4B
```

`_KPCR`

```
nt!\_KPCR
+0x000 NtTib : \_NT\_TIB
+0x01c SelfPcr : Ptr32 \_KPCR
+0x020 Prcb : Ptr32 \_KPRCB
+0x024 Irql : UChar
+0x028 IRR : Uint4B
+0x02c IrrActive : Uint4B
+0x030 IDR : Uint4B
+0x034 KdVersionBlock : Ptr32 Void
+0x038 IDT : Ptr32 \_KIDTENTRY
+0x03c GDT : Ptr32 \_KGDTENTRY
+0x040 TSS : Ptr32 \_KTSS
+0x044 MajorVersion : Uint2B
+0x046 MinorVersion : Uint2B
+0x048 SetMember : Uint4B
+0x04c StallScaleFactor : Uint4B
+0x050 DebugActive : UChar
+0x051 Number : UChar
+0x052 Spare0 : UChar
+0x053 SecondLevelCacheAssociativity : UChar
+0x054 VdmAlert : Uint4B
+0x058 KernelReserved : [14] Uint4B
+0x090 SecondLevelCacheSize : Uint4B
+0x094 HalReserved : [16] Uint4B
+0x0d4 InterruptMode : Uint4B
+0x0d8 Spare1 : UChar
+0x0dc KernelReserved2 : [17] Uint4B
+0x120 PrcbData : \_KPRCB
```

`_NT_TIB`

主要存储了`SEH`结构化异常链表和一个指向自己的指针。

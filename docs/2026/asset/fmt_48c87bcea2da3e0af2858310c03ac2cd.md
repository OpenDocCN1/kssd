
# [原创]X86内核笔记_2_驱动开发

> 0707：更新驱动遍历、隐藏。更换图床
>
> 0712：更新驱动通信
>
> 0922：更新内存加载
>
> 内存加载驱动的代码放在评论区了

[TOC]

# 0. 创建驱动项目

## 创建项目

WDK 和 VS 安装参考文章：[X86内核笔记_0_配置双机调试环境](https://bbs.pediy.com/thread-267943.htm)

打开 VS2017，新建项目选择 Visual C++ -> Windows Drivers -> Legacy -> Empty WDM Driver

![image-20210707113559574](upload/attach/202201/867232_YRT3JVPGD3C4B7A.jpg)

右键 `SourceFiles` 目录，新建项。创建一个扩展名为 C 的 C++ 文件（不要用 `cpp` 扩展名）。文件名随意起，不是非要和项目名一样。

![image-20210707113710643](upload/attach/202201/867232_SY7KPTG8PX7MQK5.jpg)

在 `.c` 文件中先引入头文件 `ntifs.h`。

```c
#include <ntifs.h> //这个头文件内包含了大量驱动相关的头文件，一次性包含，省时省力。
```

删除 `INF` 文件

![image-20210707113737860](upload/attach/202201/867232_BGCG2YU8UMSN98V.jpg)

如果引入头文件出现找不到头文件，就如下图设置一下 SDK 版本。

![image-20210707113757099](upload/attach/202201/867232_JY8DFXVE5SA84HM.jpg)

然后设置一些项目属性：

- 取消警告视为错误。

![image-20210707113817359](upload/attach/202201/867232_MU3AN9EGTKUCDJM.jpg)

- DriverSetting 中将驱动平台改为 Win7。（我们要在 Win7 上做实验。）

![image-20210707113916405](upload/attach/202201/867232_PEQT7AC57XB6PU2.jpg)

## 驱动分类简述

驱动大体分为三种，分别是：NT 式驱动、WDM 式驱动、WDF 式驱动（KWDF 内核驱动，UWDF 用户驱动）。

### NT式驱动

NT 虚拟驱动，老式驱动，从 WIN95 开始使用 NT 式驱动。若所开发的驱动不与硬件打交道，建议使用 NT 式驱动或 WDM 式驱动。如果 NT 式驱动出现了绑定设备的情况，该驱动将无法卸载。只能通过重启系统进行卸载。对于服务器来说重启很伤。

### WDM式驱动

相对于 NT 式驱动来讲，WDM 式驱动支持卸载（热拔插）。无需重启即可卸载。并且 WDM 式驱动对于 NT 式驱动进行了一些封装和优化。本质区别不大。

### WDF式驱动

WDF 式驱动相较前两种，其最大的意义是简化开发。不像 NT 与 WDM 驱动那么底层化。WDF 式驱动将 WDM 式驱动进行了封装，做成了一套架构，使得开发驱动变得更简单。同时带来的弊端就是无法掌控底层。

由于开发简便，不容易蓝屏，所以公司开发驱动一般选用 WDF 式驱动。

想要学习 WDF 式驱动，需要了解 COM 相关知识。

只有系统中存在 `WDFLDR.sys` 驱动，我们编写的 WDF 驱动才可以跑起来。并且项目中需要一个 `inf` 文件，NT/WDM 式驱动则不需要这个 `inf` 文件。

![image-20210707114938747](upload/attach/202201/867232_S35R5U42NTWNU38.jpg)

# 1. 编写驱动程序

## 驱动入口函数（`DriverEntry`）

```c
#include "ntifs.h"

NTSTATUS DriverEntry(PDRIVER_OBJECT DriverObject, PUNICODE_STRING RegistryPath) {
    //代码
    return STATUS_UNSUCCESSFUL;
}
```

`DriverEntry` 是我们写代码时的入口函数。其编译生成的 `sys` 文件真正的入口点并不是 `DriverEntry`。在 IDA 中可以看到驱动真正的入口点函数是 `GsDriverEntry`。其内部调用了我们的 `DriverEntry` 函数。

![image-20210707113939582](upload/attach/202201/867232_5K2E5DMNJ2PJJ6P.jpg)

### 指定入口函数

如果不想让编译器生成 `GsDriverEntry`，而是直接将入口函数设置为 `DriverEntry`，可以按照下图设置。

![image-20210707114010768](upload/attach/202201/867232_4AG3Q7E7V7E7CJ7.jpg)

## 编写代码

```c
#include "ntddk.h"

void UnloadDriver(PDRIVER_OBJECT driver);

NTSTATUS DriverEntry(PDRIVER_OBJECT DriverObject, PUNICODE_STRING RegistryPath) {
    DbgBreakPoint(); //相当于 __asm{int 3}
    DbgPrint("驱动加载了。\r\n"); //驱动的打印函数，相当于3环的printf
    DriverObject->DriverUnload = UnloadDriver; //为驱动指定卸载函数
    return STATUS_SUCCESS;
}

//驱动卸载函数
void UnloadDriver(PDRIVER_OBJECT driver) {
    DbgPrint("驱动停止了。\r\n");
}
```

## 打印字符串对象

如果想要打印字符串对象中的字符串，可以使用如下格式：

```c
NTSTATUS DriverEntry(PDRIVER_OBJECT DriverObject, PUNICODE_STRING pReg) {
    DbgPrint("-------%wZ--------", pReg); //传入字符串对象指针。
    return STATUS_SUCCESS;
}
```

## 生成驱动

点击生成解决方案即可。若报一些格式错误，就删除一些特殊符号之类的东西。

## 加载驱动（部署-启动-停止-卸载）

使用 `InstDrv.exe` 加载驱动。使用 `DbgView.exe` 查看输出（必须选中监视核心，否则无法监视驱动层输出）。

![1](upload/attach/202201/867232_DMT37TQZP5DM9UE.jpg)

## 调试驱动

通过调用函数 `DbgBreakPoint` 为驱动增加一个断点。这个函数相当于 `int 3` 指令。这样我们就可以在 WinDbg 中断下。并且 WinDbg 会自动识别 PE 结构中的 PDB 路径，自动加载 PDB 文件识别出我们的源码。

![image-20210707114036228](upload/attach/202201/867232_QQ95BRQW4XTGH9T.jpg)

## 解决刷屏

如果 WinDbg 调试过程中，出现了刷屏的情况，执行以下命令可以关闭刷屏。

```
kd> ed nt!Kd_SXS_Mask 0;ed nt!Kd_FUSION_Mask 0
```

# 2. 驱动对象 `PDRIVER_OBJECT` 初识

在成功断在我们的代码中后，在 WinDbg 中查看驱动对象结构。

![image-20210707114100203](upload/attach/202201/867232_FTVY6SPW7ZFR7T5.jpg)

- `Type`：驱动对象类型。
- `Size`：驱动对象大小
- `DeviceObject`：设备对象，我们这里没添加设备，因此是 `null`
- `DriverStart`：驱动文件基址，也就是 PE 格式中的 `ImageBase`。通过 `db` 命令可以看到 `4D 5A`。
- `DriverSize`：驱动模块大小，也就是 PE 格式中的 `SizeOfImage`。
- `DriverExtension`：驱动扩展对象。使用 `dt` 命令查看该对象

```
kd> dt _DRIVER_EXTENSION 0x8831b790

ntdll!_DRIVER_EXTENSION
+0x000 DriverObject : 0x8831b6e8 _DRIVER_OBJECT
+0x004 AddDevice : (null)
+0x008 Count : 0
+0x00c ServiceKeyName : _UNICODE_STRING "hellodriver"
+0x014 ClientDriverExtension : (null)
+0x018 FsFilterCallbacks : (null)
```

- `DriverObject`：指向当前驱动对象首地址。
- `ServiceKeyName`：驱动服务注册表文件夹名。
- `DriverName`：驱动名，也就是驱动的文件名前面加个 `\Driver\`。这个名字是个字符串结构体。

查看该字符串结构：

```
kd> dt _UNICODE_STRING 8831b6e8 +1c

ntdll!_UNICODE_STRING
"\Driver\hellodriver"
+0x000 Length : 0x26 //字符串长度
+0x002 MaximumLength : 0x26 //字符串最大长度
+0x004 Buffer : 0x87fc36c8 "\Driver\hellodriver" //字符串内容
```

- `HardwareDatabase`：驱动服务注册表路径。前往注册表查看该路径，可以发现一个名为“hellodriver”的文件夹，这就是我们的驱动。

![image-20210707114126468](upload/attach/202201/867232_DD9K75H2EU6TM8G.jpg)

- `DisplayName`：驱动名
- `ErrorControl`：当驱动加载失败时会设置这个值。
- `ImagePath`：驱动文件路径。`\??\` 是设备路径，我们平时访问各种文件夹其实都带这个 `\??\`，只是 Windows 底层帮我们补充了。
- `Start`：驱动加载类型。手动启动为 3，开机自启为 2，BIOS 自启为 1。
- `Type`：服务类型。1 为驱动。
- `DriverInit`：驱动入口点，也就是 PE 文件的 `AddressOfEntryPoint`。
- `DriverUnload`：驱动卸载函数地址。

# 3. 驱动加载方法

加载驱动大体分为两种：服务加载和直接加载。实际应用中可以将两种方法都利用上。

## 服务加载

1. 调用 `OpenSCManager` 打开服务控制。
2. 调用 `CreateService` 创建服务。实际上就是创建注册表相关键值。在执行完该 API 后，驱动已经被注册为服务了。这时我们通过 `CMD` 执行 `net start XXXX` 也可以加载我们的驱动。
3. 调用 `OpenService` 打开现有服务。
4. 调用 `StartService` 启动服务

这种方式实际上加载该驱动的进程，并不是调用 API 的进程。而是通过 API 向系统通知我要加载一个驱动。系统进程接收到通知后加入到系统中的一个队列。并由系统进程在某时某刻加载该驱动。

也就是这种方式是通知系统进程来进行加载。

## 直接加载

调用 `ZwLoadDriver` 或 `NtLoadDriver` 加载一个已被正确注册的驱动。

这种方法需要我们自己手动去注册表内注册该驱动的相关信息。这样该驱动才可以被加载。

直接加载的方式在调用 API 后就会直接加载该驱动，所以该驱动的加载者就是调用该 API 的进程。相比于服务加载会留下痕迹。

# 4. 第一个练习

编写两个驱动 A 和 B，在 A 中定义全局变量值为 100，打印 A 的地址 `pA`。在 B 中打印 `pA` 的数据，观察是否与 A 中定义的相同。

A 代码：

```c
#include <ntifs.h>

void UnloadDriver(PDRIVER_OBJECT driver);

UINT32 i = 100;

NTSTATUS DriverEntry(PDRIVER_OBJECT DriverObject, PUNICODE_STRING RegistryPath) {
    DbgPrint("i addr = %08x\r\n", &i);
    DriverObject->DriverUnload = UnloadDriver;
    return STATUS_SUCCESS;
}

//驱动卸载函数
void UnloadDriver(PDRIVER_OBJECT driver) {
    DbgPrint("驱动停止了。\r\n");
}
```

B 代码：

```c
#include <ntifs.h>

void UnloadDriver(PDRIVER_OBJECT driver);

NTSTATUS DriverEntry(PDRIVER_OBJECT DriverObject, PUNICODE_STRING RegistryPath) {
    PUINT32 p = (PUINT32)0x8e315000;
    DbgPrint("i value = %d\r\n", *p);
    DriverObject->DriverUnload = UnloadDriver;
    return STATUS_SUCCESS;
}

//驱动卸载函数
void UnloadDriver(PDRIVER_OBJECT driver) {
    DbgPrint("驱动停止了。\r\n");
}
```

# 5. 驱动常用类型及 API

在驱动中写代码与 3 环不同，一些数据类型及常用 API 也最好使用驱动开发专用的版本。这算是一种代码规范。

## 基本数据类型

在驱动中，原数据类型 `int`、`char` 等均被封装、重定义。在驱动开发中应使用如下数据类型：

```
UINT8,PUINT8 -> unsigned char
UINT16,PUINT16 -> unsigned short
UINT32,PUINT32 -> unsigned int
UINT64,PUINT64 -> unsigned __int64
INT8,PINT8 -> char
INT16,PINT16 -> short
INT32,PINT32 -> int
INT64,PINT64 -> __int64
LONG32,PLONG32 -> int
ULONG32,PULONG32 -> unsigned int
DWORD32,PDWRD32 -> int
```

## 错误码返回值

绝大多数内核函数都会有一个返回值，类型为 `NTSTATUS`。该类型本质就是一个 `LONG`。

如 `GetLastError` 这种取错误码的函数，取到的值其实就是 `NTSTATUS` 转化后的错误码。

常用的 `NTSTATUS` 宏如下，负数（大于 `0X80000000`）的返回值为错误，大于等于 0 为成功：

```
STATUS_SEVERITY_SUCCESS 0x0
STATUS_SEVERITY_INFORMATIONAL 0x1
STATUS_SEVERITY_WARNING 0x2
STATUS_SEVERITY_ERROR 0x3
STATUS_UNSUCCESSFUL 0xC0000001
STATUS_ACCESS_VIOLATION 0xC0000005
```

同时有一个宏用于判断返回值是成功还是失败：

```c
NT_SUCCESS(NTSATUS类型参数) //#define NT_SUCCESS(Status) (((NTSTATUS)(Status)) >= 0)
```

## 字符串相关

在内核开发中，字符串不要定义为 `char* x = "xx"`，WDK 为我们准备了一些字符串相关的 API。

### 定义字符串

```c
UNICODE_STRING uStr = {0}; //定义一个unicode字符串，类型为UNICODE_STRING
STRING aStr = {0}; //定义一个ascii字符串，类型为STRING
ANSI_STRING aStr = {0}; //所有ANSI与直接STRING 作用相同
```

### 初始化字符串

```c
RtlInitUnicodeString(&uStr, L"unicode string"); //初始化unicode字符串，为其赋值。不会申请内存。
RtlInitString(&aStr, "ascii string"); //初始化ascii字符串，为其赋值，不会申请内存。
RtlInitAnsiString(&aStr, "ascii string"); //初始化ascii字符串，为其赋值，不会申请内存。
```

### 字符串转化

```c
RtlAnsiStringToUnicodeString(&uStr, &aStr, true); //将ascii字符串转为unicode字符串，无需为unicode字符串做初始化，第三个参数为true则自动申请内存。为false则不申请，仅修改unicode现有空间。若为true，则需要手动释放字符串内存。
RtlUnicodeStringToAnsiString(); //将unicode字符串转为ascii字符串，用法与上面相同。
```

### 释放字符串

```c
RtlFreeUnicodeString(); //释放unicode字符串内存，当字符串初始化中为其分配了内存时，需要释放内存。
RtlFreeAnsiString(); //释放ascii字符串内存，当字符串初始化中为其分配了内存时，需要释放内存。
```

### 字符串格式化

```c
#include <ntstrsafe.h> //使用格式化API需要引入此头文件

char aStr[0x1000] = {0};

RtlStringCbPrintfA(aStr, 0x1000, "%d---%s", 123, "test"); //参数1： Ascii字符串指针
```

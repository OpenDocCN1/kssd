
# [原创]病毒分析日志_Rootkit

## Rootkit常用隐藏技术

### 1 查壳：

![](https://bbs.kanxue.com/upload/attach/202010/830337_965GKR37N3A8BZB.jpg)

`VC`编写的,无壳

### 2 静态分析

`strings`分析字符串

![](https://bbs.kanxue.com/upload/attach/202010/830337_RYFQZYPFSZJGVXK.jpg)

看到了文件操作，`WriteFile`表明将会在某个时间写文件

![](https://bbs.kanxue.com/upload/attach/202010/830337_XKFACJF8J5UAPJ3.jpg)

看到了`OpenSCManagerA` `StartServiceA` `CreateServiceA` `CloseServiceHandle`表明创建了一个服务

看到了`GetCommandLineA`推测可能读取了命令行某个命令执行某个操作

![](https://bbs.kanxue.com/upload/attach/202010/830337_CWE9RES2G8UNUZT.jpg)

看到`virtualalloc`猜测可能执行了远程线程注入

这里不全部列出，这里还要注意两个API调用，`LoadResource`和`SizeOfResource`表明对该程序的资源节做了某些处理

`C:\Windows\System32\Mlwx486.sys`表明程序在该位置创建了这个文件

可以多用一些工具

![](https://bbs.kanxue.com/upload/attach/202010/830337_AQ9KG9NCVT66RNT.jpg)

这里我们查看下资源节，因为我们前面发现了一些资源操作函数(有些骚的作者会在资源节隐藏shellcode)

![](https://bbs.kanxue.com/upload/attach/202010/830337_7GM6GQQ836NXQ98.jpg)

资源节中包含一个PE头，资源节中是另一个可执行文件

向下拉

![](https://bbs.kanxue.com/upload/attach/202010/830337_QBKVTYNFRBPPDCU.jpg)

`MmGetSystemRoutineAddress`获取`ntoskrnl`模块导出的函数地址，原生API

### 3 行为分析

打开`procmon`

查看进程树

![](https://bbs.kanxue.com/upload/attach/202010/830337_PE7NA5AYVN5M4EF.jpg)

`Operation`过滤

![](https://bbs.kanxue.com/upload/attach/202010/830337_DN5CYZGF285Q29V.jpg)

![](https://bbs.kanxue.com/upload/attach/202010/830337_8XGJT5UUAZKFSE4.jpg)

第一幅图的倒数第二行，创建了`c:\WINDOWS\system32\Mlwx486.sys`这个文件，第二幅图第一行创建了一个文件

我们猜测这是一个`rootkit`，原因有下：

1 根据我们用`strings`发现的API和在资源节发现的`MmGetSystemRoutineAddress`因为服务是进入内核的方式之一，其他的还有`hook`这里不做过多讨论。
2. `sys`后缀,
3. 我们运行这个恶意程序，没有任何症状发生，说明用了隐藏技术
4. 当我们去该目录下找该文件的时候发现找不到

`Mlwx486.sys`创建了一个服务，该服务名为`486 WS Driver`,如下图所示

![](https://bbs.kanxue.com/upload/attach/202010/830337_2R8MY6URVK7588R.jpg)

### 4 动态分析

载入`IDA`

在`IDA`中，同样可以看到服务名，调用的函数

![](https://bbs.kanxue.com/upload/attach/202010/830337_UM5GV3DB7NWDWYK.jpg)

![](https://bbs.kanxue.com/upload/attach/202010/830337_M7U5VERHE4HT3JN.jpg)

检查该内核驱动是否被加载

![](https://bbs.kanxue.com/upload/attach/202010/830337_GXCCWPUYCBVBEVM.jpg)

驱动仍然在运行

在真机上打开`WInDBG`连接到虚拟机查看`SSDT`表

![](https://bbs.kanxue.com/upload/attach/202010/830337_2JM2NB2CFR7PMZT.jpg)

如图，有一个地址位于`nt`模块的边界之外

恢复虚拟机到之前的状态，查看该位置原来的值

![](https://bbs.kanxue.com/upload/attach/202010/830337_XA754HA873PKQKJ.jpg)

![](https://bbs.kanxue.com/upload/attach/202010/830337_J3WTKQUTP4NTMH6.jpg)

被覆盖的函数是`NtQueryDirectoryFile`,在`MSDN`查看以下该函数的功能

![](https://bbs.kanxue.com/upload/attach/202010/830337_6JU5W5Y75HYMQB8.jpg)

![](https://bbs.kanxue.com/upload/attach/202010/830337_5W6HGBH5A8Q2H8E.jpg)

该函数的功能是返回指定文件目录的各种信息，`HOOK`了这个函数就可以隐藏文件信息，这也解释了前面为什么看不到文件，替换函数必须与原函数有相同的接口，即要与`NtQueryDirectoryFile`有相同的参数，对`f8d9e486`下断

![](https://bbs.kanxue.com/upload/attach/202010/830337_WKMKJCEWQM3952V.jpg)

替换的函数开头调用了`f8d9e514`,这里在`WinDBG`中分析的话不知道调用的是什么函数，我们已经知道这个`rootkit`在资源节中，所以，我们可以试着把它从资源节中提取出来，将恶意代码重新载入`ResourceHacker`,然后另存为`bin`文件然后加载入`IDA`中

![](https://bbs.kanxue.com/upload/attach/202010/830337_UWMXDPWY84VCEDA.jpg)

![](https://bbs.kanxue.com/upload/attach/202010/830337_C86E8DM8HWENEHS.jpg)

在`DriverEntry`中调用了`Sub_10772`

![](https://bbs.kanxue.com/upload/attach/202010/830337_ZYGRMA7S43DK2X4.jpg)

下面看看`sub_10706`这个函数

![](https://bbs.kanxue.com/upload/attach/202010/830337_A7BQA4V4DYFF8F9.jpg)

这里，`RtlInitUnicodeString`函数用于创建内核态的字符串，第二个参数是要创建的`UNICODE_STRING`类型以NULL结束的宽字符串，因为Windows内核使用的是`UNICODE_STRING`结构，与用户态的不同，所以这里要转换一下，双击倒数第二个参数，跟进

![](https://bbs.kanxue.com/upload/attach/202010/830337_42GX3FTDTVRGV2R.jpg)

这里创建的是`NtQueryDirectoryFile`字符串，还有`KeServiceDescriptorTable`字符串，然后调用`MmGetSystemRoutineAddress`函数查找`NtQueryDirectoryFile`函数和`KeServiceDescriptorTable`的导出地址，它们都由`ntoskrnl.exe`模块导出，然后被内核驱动导入，`MmGetSystemRoutineAddress`这个函数类似于R3下的`GetProcAddress`函数，我们来看一下`MSDN`的解释

![](https://bbs.kanxue.com/upload/attach/202010/830337_CUYN53WJCYNFZQA.jpg)

继续往下看

![](https://bbs.kanxue.com/upload/attach/202010/830337_DK8TVQCNV2RTXUP.jpg)

接下来的代码循环遍历`SSDT`直到找到`NtQueryDirectoryFile`地址，然后使用`HOOK`将该地址覆盖，其中，`00010760`处的`mov`指令，将挂钩函数的地址复制到一个内存位置，什么意思那，就是安装挂钩。

接下来再来看看`sub_10486`这个函数，

![](https://bbs.kanxue.com/upload/attach/202010/830337_9MNDEBPXCVZAC6G.jpg)

注意光标处，是不是想起了什么，没错，我们在`WinDBG`中的，和对`f8d9e514`处的调用一摸一样，好了，这样我们就解决了前面的问题，在`WInDBG`中的时候我们不知道这里调用的是什么函数，这里`IDA`给出了注释，这里调用的是`NtQueryDirectoryFile`这个函数，有人可能会有疑问，我是怎么知道这里对用的是`WInDBG` 中的`call f8d9e514`处，其实就是参数，上面一大堆的`push`,其实就算我们不用`IDA`也可以猜到是`NtQueryDirectoryFile`,因为我们去`MSDN`查了这个函数，对比一下就知道了，遇到不熟悉的函数就去查`MSDN`，可以提高对函数的熟悉程度，这样不仅可以提高分析恶意代码的效率，对编程了能力的提高也有好处。

![](https://bbs.kanxue.com/upload/attach/202010/830337_KDP9K66DCHU2WU7.jpg)

这里，我们把中重点放在`sub_10486`这个函数上，毫无疑问，这个就是挂钩函数，剩下的是安装挂钩的函数

`000104b6`处，挂钩函数检查参数`FileInformationClass`是否为3，如果是3之外的值，便会返回`NtQueryDirectoryFile`的原始返回值，也检查`RestartScan`与第九个参数`ReturnSingleEntry`

![](https://bbs.kanxue.com/upload/attach/202010/830337_R2CEDH6NW5CJERY.jpg)

挂钩函数过滤一些特殊请求，其余请求则传递给原始的`NtQueryDirectoryFile`函数，当`ReturnSingleEntry`为0时，`RtlCompareMemory`会被调用，这也是挂钩函数的过滤结果。

![](https://bbs.kanxue.com/upload/attach/202010/830337_CGSKCHWDE8HZS33.jpg)

![](https://bbs.kanxue.com/upload/attach/202010/830337_V3N392VBXF28V6S.jpg)

![](https://bbs.kanxue.com/upload/attach/202010/830337_2Z3U88JCQ6WG8EW.jpg)

`RtlCompareMemory`返回两个块中匹配的字节数，若所有字节都匹配到指定的`Length`值，则返回`Length`值

我们看一下`source2`是什么

![](https://bbs.kanxue.com/upload/attach/202010/830337_RTCNMS292PQXYKH.jpg)

`'Mlwx'`

这里的`Length`参数是8，`esi`是`FileInformation`,0x5E偏移量存储的是宽字符串文件名的地址

![](https://bbs.kanxue.com/upload/attach/202010/830337_RFZAVAAUF55Y5YD.jpg)

![](https://bbs.kanxue.com/upload/attach/202010/830337_2HB9AKSZFFNZV75.jpg)

![](https://bbs.kanxue.com/upload/attach/202010/830337_YTXEYJJKN5CU72Y.jpg)

因为`IDA`是静态分析器，这里我们看不到`esi+5eh`处的值,

`RtlCompareMemory`调用了指定大小是8字节，`Unicode`则是4，这段代码比较每个文件的文件名，以查看它们开头的四个字节是不是`Mlwx`,如果是，则对该文件进行隐藏。

`_FILE_ID_BOTH_DIR_INFORMATION`结构的第一个参数指向下一个`_FILE_ID_BOTH_DIR_INFORMATION`结构的偏移量，挂钩函数就是通过修改这个域使其指向下下个`_FILE_ID_BOTH_DIR_INFORMATION`结构来隐藏文件，若当前项拥有一个以`Mlwx`开头的文件，则移动前面的偏移量，使其指向当前项的下一个结构。`_FILE_ID_BOTH_DIR_INFORMATION`是`NtQueryDirectoryFile`的返回值。

![](https://bbs.kanxue.com/upload/attach/202010/830337_RW95N54ECDUMPST.jpg)

![](https://bbs.kanxue.com/upload/attach/202010/830337_5879HGGGCCTJEC2.jpg)

![](https://bbs.kanxue.com/upload/attach/202010/830337_TH8HJ9JJEUZ8MSC.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)

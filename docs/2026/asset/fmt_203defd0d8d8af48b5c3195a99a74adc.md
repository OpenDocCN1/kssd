
### Lab1-2

> 分析`Lab01-02.exe`文件
>
> **问题**
>
> 1、将`Lab01-02.exe`文件上传至<mark class="encrypted">358K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6g2k6A6M7Y4g2K6g2r3!0@1j5h3I4Q4x3X3g2U0L8$3#2Q4y4U0m8Q4c8e0S2Q4b7V1k6Q4z5f1u0Q4c8e0S2Q4b7e0q4Q4z5p5y4Q4c8e0k6Q4z5e0k6Q4z5o6N6Q4c8e0c8Q4b7V1u0Q4b7U0k6Q4c8e0g2Q4z5p5y4Q4b7U0W2Q4c8e0W2Q4z5o6g2Q4z5p5c8Q4c8e0g2Q4z5o6S2Q4b7U0m8Q4c8e0c8Q4b7V1q4Q4z5o6k6Q4c8e0g2Q4b7U0N6Q4b7U0u0Q4c8e0k6Q4z5f1y4Q4z5o6W2Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0g2Q4z5p5k6Q4z5p5c8Q4c8e0N6Q4z5e0N6Q4z5o6g2Q4c8e0k6Q4b7f1k6Q4z5e0u0Q4c8e0S2Q4b7V1c8Q4b7f1k6Q4c8e0c8Q4b7V1u0Q4b7U0k6Q4c8e0N6Q4z5o6W2Q4b7U0W2Q4c8e0g2Q4b7V1g2Q4z5o6q4Q4c8e0g2Q4z5e0m8Q4z5e0N6Q4c8f1k6Q4b7V1y4Q4z5f1j5</mark>.
>
> 2、是否有这个文件被加壳或混淆的任何迹象？如果是这样，这些迹象是什么？如果该文件被加壳，请进行脱壳，如果可能的话。
>
> 3、有没有任何导入函数能够暗示出这个程序的功能？如果是，是哪些导入函数，它们会告诉你什么？
>
> 4、哪些基于主机或基于网络的迹象可以被用来确定被这个恶意代码所感染的机器？

#### 解题

1、在官网<mark class="encrypted">347K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6g2k6A6M7Y4g2K6g2r3!0@1j5h3I4Q4x3X3g2U0L8$3#2Q4y4U0m8Q4c8e0S2Q4b7V1k6Q4z5f1u0Q4c8e0S2Q4b7e0q4Q4z5p5y4Q4c8e0k6Q4z5e0k6Q4z5o6N6Q4c8e0c8Q4b7V1u0Q4b7U0k6Q4c8e0c8Q4b7U0S2Q4z5p5q4Q4c8e0c8Q4b7V1y4Q4b7e0m8Q4c8f1k6Q4b7V1y4Q4z5f1t1</mark>.

`Lab01-01.exe`检测结果：

![image-20210403160148067](upload/attach/202104/856790_HXSMK6H5M8SQ537.jpg)

+ 这个就有点多了，有52个安全厂商标识为病毒，看来此程序操作还是挺敏感的。

2、使用`CFF`、`Exeinfo PE`工具进行查壳检测；

+ 首先习惯性先用`CFF`查看一下PE：

![image-20210403160638788](upload/attach/202104/856790_HJ3DF47BJ4USFW9.jpg)

+ 这个好玩了，`UPX`节区，很明显了，但是还是用`Exeinfo PE`查一下确认一波：

![image-20210403160815441](upload/attach/202104/856790_N5HRGM7NX47NWWR.jpg)

+ 很显然了，虽然`CFF`有自动脱`UPX`压缩壳的功能，但是为了学习，还是遵从题意，下载`UPX`工具，使用`upx -d`命令脱壳；

![image-20210403161539751](upload/attach/202104/856790_3TE6TGQDG5KCSM9.jpg)

+ 出现了正确的编译环境，就说明脱壳成功了；

3、使用`Dependency Walker`工具查看`Lab01-02.exe`文件的导入函数；

![image-20210403161915933](upload/attach/202104/856790_27K4TFENSQTVVFW.jpg)

+ `Kernel32.dll`文件包含了核心系统功能，包括但不限于访问和操作内存、文件和硬件；

+ 因为篇幅原因，文件内导入的函数功能直接给出，细节请查看MSDN；

```c
SystemTimeToFileTime(...);//将系统时间转换为文件时间。
GetModuleFileNameA(...);//获取包含指定模块的可执行文件的完整路径和文件名。
CreateWaitableTimerA(...);//函数创建了一个“可等待的”计时器对象。
ExitProcess(...);//结束一个进程及其所有线程。
OpenMutexA(...);//返回一个已存在的命名互斥对象的句柄。
SetWaitableTimer(...);//激活指定的“可等待”计时器。当到期时间到达时，计时器被通知，设置
//计时器的线程调用可选的完成例程。
WaitForSingleObject(...);//WaitForSingleObject函数在下列情况之一发生时返回:
//指定的对象处于有信号状态。
//超时时间过期。
CreateMutexA(...);//创建一个命名的或未命名的互斥对象。
CreateThread(...);//创建一个线程，在调用进程的地址空间中执行。
```

![image-20210403161941645](upload/attach/202104/856790_3YT22M6A2M6PQ7N.jpg)

+ `Advapi32.dll`对windows核心组件的访问，比如服务管理器和注册表；

+ 因为篇幅原因，文件内导入的函数功能直接给出，细节请查看MSDN；

```c
CreateServiceA(...);//创建一个服务对象，并将其添加到指定的服务控制管理器数据库中。
StartServiceCtrlDispatcherA(...);//服务进程的主线程连接到服务控制管理器，这将使线程成
//为调用进程的服务控制调度程序线程。
OpenSCManagerA(...);//在指定的计算机上建立到业务控制管理中心的连接，并打开指定的业务控制管理中心数据库。
```

![image-20210403162019151](upload/attach/202104/856790_WGJAE9NKTPDVNFP.jpg)

+ `Msvcrt.dll`是微软的C运行环境库，这并非核心功能函数，可以跳过；

![image-20210403162038590](upload/attach/202104/856790_NNMMYCYSG7H6VSY.jpg)

+ `Wininet.dll`是Windows应用程序网络相关模块，层级相对更高，实现了例如FTP、HTTP和NTP等网络协议，这个模块也是病毒对抗中的经典了，改日再细说；

+ 因为篇幅原因，文件内导入的函数功能直接给出，细节请查看MSDN；

```c
InternetOpenUrlA(...);//InternetOpenUrl函数解析URL字符串，建立到服务器的连接，并准备 下载由URL标识的数据。
InternetOpenA(...);//是Win32应用程序调用的第一个Internet函数。它告诉Internet DLL初 始化内部数据结构，并为将来从应用程序调用做准备。当应用程序结束使 用Internet函数时，它应该调用InternetCloseHandle来释放该句 柄和任何相关的资源。
```

总结：这个程序会进行联网操作并且创建服务；

4、使用`IDA`工具查看；

![image-20210403165659092](upload/attach/202104/856790_TTD9NZNGHTNHVKJ.jpg)

+ 同样盲猜一波，啧，<mark class="encrypted">839K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2S2L8s2N6S2M7X3g2S2L8X3q4D9P5i4y4A6M7$3u0G2L8$3E0Q4x3X3g2U0L8$3#2Q4y4U0m8Q4c8e0g2Q4b7f1c8Q4z5e0N6Q4c8e0N6Q4b7f1y4Q4b7e0k6Q4c8e0c8Q4b7U0S2Q4b7U0u0Q4c8e0c8Q4b7V1u0Q4b7e0g2Q4c8e0g2Q4z5p5k6Q4z5p5q4Q4y4U0m8y4j5h3I4e0k6i4u0$3K9h3y4W2i4K6j5H3i4@1f1#2i4@1q4p5i4K6V1%4i4@1f1%4i4@1q4o6i4@1p5$3i4@1f1@1i4@1t1^5i4@1t1J5i4@1f1&6i4K6R3K6i4@1u0p5i4@1f1#2i4K6S2r3i4@1q4r3i4@1f1@1i4@1u0n7i4@1p5#2i4@1f1^5i4@1p5I4i4@1p5^5i4@1f1$3i4K6V1^5i4K6S2q4i4@1f1^5i4@1u0r3i4K6V1&6i4@1f1$3i4K6V1^5i4@1q4r3i4@1f1@1i4@1t1^5i4K6R3H3i4@1f1@1i4@1t1^5i4@1q4m8i4@1f1#2i4K6W2r3i4@1u0m8i4@1f1@1i4@1u0m8i4K6S2q4i4@1f1%4i4@1u0p5i4K6V1I4i4@1f1%4i4@1u0n7i4K6W2o6i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1%4i4K6V1%4i4K6R3#2i4@1f1$3i4@1q4r3i4K6V1J5i4@1f1@1i4@1u0n7i4@1p5K6i4@1f1%4i4@1p5H3i4K6R3I4i4@1g2r3i4@1u0o6i4K6W2n7</mark>

---

### Lab1-3

> 分析`Lab01-03.exe`文件
>
> **问题**
>
> 1、将`Lab01-03.exe`文件上传至<mark class="encrypted">60cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6g2k6A6M7Y4g2K6g2r3!0@1j5h3I4Q4x3X3g2U0L8$3#2Q4y4U0m8Q4c8e0S2Q4b7V1k6Q4z5f1u0Q4c8e0S2Q4b7e0q4Q4z5p5y4Q4c8e0k6Q4z5e0k6Q4z5o6N6Q4c8e0c8Q4b7V1u0Q4b7U0k6Q4c8e0g2Q4z5p5y4Q4b7U0W2Q4c8e0W2Q4z5o6g2Q4z5p5c8Q4c8e0g2Q4z5o6S2Q4b7U0m8Q4c8e0c8Q4b7V1q4Q4z5o6k6Q4c8e0g2Q4b7U0N6Q4b7U0u0Q4c8e0k6Q4z5f1y4Q4z5o6W2Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0g2Q4z5p5k6Q4z5p5c8Q4c8e0N6Q4z5e0N6Q4z5o6g2Q4c8e0k6Q4b7f1k6Q4z5e0u0Q4c8e0S2Q4b7V1c8Q4b7f1k6Q4c8e0c8Q4b7V1u0Q4b7U0k6Q4c8e0N6Q4z5o6W2Q4b7U0W2Q4c8e0g2Q4b7V1g2Q4z5o6q4Q4c8e0g2Q4z5e0m8Q4z5e0N6Q4c8f1k6Q4b7V1y4Q4z5f1j5</mark>.
>
> 2、是否有这个文件被加壳或混淆的任何迹象？如果是这样，这些迹象是什么？如果该文件被加壳，请进行脱壳，如果可能的话。
>
> 3、有没有任何导入函数能够暗示处这个程序的功能？如果是，是哪些导入函数，它们会告诉你什么？
>
> 4、有哪些基于主机或基于网络的迹象，可以被用来确定被这个恶意代码所感染的机器？

#### 解题

1、在官网<mark class="encrypted">408K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6g2k6A6M7Y4g2K6g2r3!0@1j5h3I4Q4x3X3g2U0L8$3#2Q4y4U0m8Q4c8e0S2Q4b7V1k6Q4z5f1u0Q4c8e0S2Q4b7e0q4Q4z5p5y4Q4c8e0k6Q4z5e0k6Q4z5o6N6Q4c8e0c8Q4b7V1u0Q4b7U0k6Q4c8e0c8Q4b7U0S2Q4z5p5q4Q4c8e0c8Q4b7V1y4Q4b7e0m8Q4c8f1k6Q4b7V1y4Q4z5f1t1</mark>.

![image-20210403170248341](upload/attach/202104/856790_J6PJURZNNQRXGTE.jpg)

+ 有五十个安全厂商标识为病毒文件；

2、使用`CFF`、`Exeinfo PE`、`Dependency Walker`工具进行查壳检测；

![image-20210403171800569](upload/attach/202104/856790_DYFWRGSPZXS22MN.jpg)

+ 首先看节表，节表名被抹了，所以PE文件肯定被修改了。

![image-20210403172238740](upload/attach/202104/856790_VE5CWZYBHDETYPR.jpg)

+ 使用`CFF`工具也可以，但是我觉得太长了。我就使用了`WinHex`工具进行截图，发现只有导入表有文件，其他表都为`0x00`，这是不正常的；

![image-20210403172606946](upload/attach/202104/856790_SBWARCUKYYEPJFH.jpg)

+ 加壳程序往往只有这两个导入函数，这种情况也可以被石锤为百分百加壳了。(就是最简单的Hello world程序也不止这两个导入函数。)

![image-20210403172926727](upload/attach/202104/856790_PB3RFYDT32UTKA5.jpg)

+ 既然知道被加壳，我们就使用`Exeinfo PE`工具看看具体是哪种壳，图中显示为`FSG 1.10`；

+ 暂时还没到脱壳章节，书后答案显示，18章课后习题将学习手动脱壳，再详细套路；

3、暂无，后面章节续写；

4、暂无，后面章节续写；

---

### Lab1-4

> 分析`Lab01-04.exe`文件
>
> **问题**
>
> 1、将`Lab01-04.exe`文件上传至<mark class="encrypted">91dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6g2k6A6M7Y4g2K6g2r3!0@1j5h3I4Q4x3X3g2U0L8$3#2Q4y4U0m8Q4c8e0S2Q4b7V1k6Q4z5f1u0Q4c8e0S2Q4b7e0q4Q4z5p5y4Q4c8e0k6Q4z5e0k6Q4z5o6N6Q4c8e0c8Q4b7V1u0Q4b7U0k6Q4c8e0g2Q4z5p5y4Q4b7U0W2Q4c8e0W2Q4z5o6g2Q4z5p5c8Q4c8e0g2Q4z5o6S2Q4b7U0m8Q4c8e0c8Q4b7V1q4Q4z5o6k6Q4c8e0g2Q4b7U0N6Q4b7U0u0Q4c8e0k6Q4z5f1y4Q4z5o6W2Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0g2Q4z5p5k6Q4z5p5c8Q4c8e0N6Q4z5e0N6Q4z5o6g2Q4c8e0k6Q4b7f1k6Q4z5e0u0Q4c8e0S2Q4b7V1c8Q4b7f1k6Q4c8e0c8Q4b7V1u0Q4b7U0k6Q4c8e0N6Q4z5o6W2Q4b7U0W2Q4c8e0g2Q4b7V1g2Q4z5o6q4Q4c8e0g2Q4z5e0m8Q4z5e0N6Q4c8f1k6Q4b7V1y4Q4z5f1j5</mark>.
>
> 2、是否有这个文件被加壳或混淆的任何迹象？如果是这样，这些迹象是什么？如果该文件被加壳，请进行脱壳，如果可能的话。
>
> 3、这个文件是什么时候被编译的？
>
> 4、有没有任何导入函数能够暗示处这个程序的功能？如果是，是哪些导入函数，它们会告诉你什么？
>
> 5、有哪些基于主机或基于网络的迹象，可以被用来确定被这个恶意代码所感染的机器？
>
> 6、这个文件在资源段中包含一个资源。使用`Resource Hacker`工具来检查资源，然后抽取资源。从资源中你能发现什么吗？

#### 解题

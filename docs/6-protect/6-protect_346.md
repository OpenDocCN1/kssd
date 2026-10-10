

# Anti-Debug

> 原文：[`www.pediy.com/kssd/tutorial/chap6-2-1.htm`](https://www.pediy.com/kssd/tutorial/chap6-2-1.htm)

**Anti-Debug**

1．MeltICE 子类型  
类型：检测 SoftICE、TRW2000  
平台：Windows9x、Windows NT  
原理：用 CreateFileA( )或\_lopen( )函数试图获得 SoftICE 的驱动程序"\\\\.\\SICE"（Windows9X 版本）、"\\\\.\\SIWDEBUG"、"\\\\.\\NTICE"（Windows NT 版本）、"\\\\.\\SIWVID"等的句柄，如果成功则说明 SoftICE 驻留在内存中。

2．VWIN32\_Int41Dispatch 子类型  
类型：检测 SoftICE  
平台：Windows9x  
原理：VWIN32.VxD（其 VxD ID 为 0x002A）提供一个名为 VWIN32\_Int41Dispatch 的 VxD service（其 service ID 为 0x002A），系统内核使用此服务来与系统级调试器如 WinDBG、SoftICE 等进行通信。其中 0x4F 号子功能是用来查询调试器是否已经驻留内存并能否处理保护模式程序，如果是的话则调试器应返回 0xF386。

3．给 SoftICE 发送命令  
类型：检测 SoftICE  
平台：Windows9x、Windows NT  
原理：通过调试中断 int 3 给 SoftICE 发送命令让其执行，其中 SI 和 DI 寄存器中放的分别是固定值 0x4647("FG")和 0x4A4D("JM")。AX 中存放的是子功能号，值为 0x0911 则表示让 SoftICE 执行命令，此时 DX 指向一个命令字符串如"HBOOT"等。AX 还可以为其它子功能号，比如让 SoftICE 修改断点设置等。

4、BoundsChecker 后门  
类型：检测 SoftICE  
平台：Windows9x、Windows NT  
原理：这是 SoftICE 为 BoundsChecker 留的一个公开的接口，入口参数 EBP = 0x4243484B(即"BCHK")，AL =4，如果 SoftICE 在内存中则应返回 AL = 0。

这种方法一般也要结合 SEH?（结构异常处理）来实现，否则当 SoftICE 不存在时就会引起非法操作。

5．ICECream 子类型  
类型：检测 SoftICE、TRW2000  
平台：Windows9x  
原理：调试器驻留后修改 INT 1 和 INT 3 的入口，指向它自己的处理程序，所以入口高位偏移与其他中断不同。其他所有中断入口高位偏移都相同。

6．INT 68h 子类型  
类型：检测 SoftICE  
平台：Windows9x  
原理：

> MOV AH, 43h  
> INT 68h  
> CMP AX, 0F386h ；检测此处是否被调试器设置 0F386h  
> JZ SoftICE\_is\_here

7．搜索特征串  
类型：检测 SoftICE  
平台：Windows9x  
原理：通过在内存中搜索 SoftICE 的特征串来发现 SoftICE，这一般要结合 SEH 一起使用，以防止引起内存保护出错而使得程序被终止。这种方法在 DOS 下是可行的。由于 Windows95 之后的操作系统中的每个 ring 3 进程的地址空间是独立的，使得这种方法受到限制。比如在内存中搜索"WINICE.BR"。

8．IsDebuggerPresent 子类型  
类型：检测 SoftICE  
平台：Windows NT  
原理：调用 kernel32.dll 输出的函数 IsDebuggerPresent()来检测是否有调试器存在。这个函数只能检查使用 Debug API 来跟踪程序的调试器，无法检测 SoftICE 之类的系统级调试器。 

 

 4dd466d0e1e0cc123afbe504da2462ca
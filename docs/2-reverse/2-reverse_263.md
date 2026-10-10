

# IceDump 和 NticeDump 使用

> 原文：[`www.pediy.com/kssd/tutorial/chap8-5-3.htm`](https://www.pediy.com/kssd/tutorial/chap8-5-3.htm)

　　IceDump 和 NticeDump 是一款配合 SoftICE 扩展其内存操作的工具，IceDump 支持 Windows 9x、Windows Millennium 系统，NticeDump 支持 Windows NT/2000。它们的出现，使 SoftICE 如虎添翼，TRW2000 的许多特色功能在 SoftICE 里也可实现了。

1．Icedump 操作简介  
　　运行 IceDump 前，首先要确定 SoftICE 版本号，按 Ctrl+D 切换到 SoftICE 下命令：VER，查看版本号。然后在相应 SoftICE 版本号目录下运行 icedump.exe 文件，它会调用自身的 VXD 文件，装载成功出现图 7.16 所示画面。如果发现 SoftICE 没运行或版本不符，就拒绝运行。 如果想从内存中卸载它，可以在 DOS 下键入"icedump u"。

1）/DUMP <起始地址> \[<长度> <文件名>\]  
　　抓取内存中的数据到文件里，类似 TRW2000 中的 W 命令。<文件名>参数可以指定盘符和路径，当在 Ring-0 下还原时最好清除还原区域内的全部断点，否则会给 SoftICE 带来不必要麻烦。（这一点在所有的 IceDump 命令里都应该值得注意）  
在 Win32 系统下读者可能会想到用/BHRAMA 或/PEDUMP 从内存内中重建一个可用的 PE 镜像。请看下面关于/OPTION 命令的说明。  
  
* 注意： IceDump 6.015 以前类似的命令是 PAGEIN D <address> \[<length> <filename>\]*

2）/LOAD <地址> <长度> <文件名>  
　　把<文件>指定长度的字节内容调入到内存中的<地址>处。与/DUMP 的作用相反，同样需要注意的是不要设置断点。

3）/BHRAMA <Bhrama dumper server 窗口名>  
　　用 Procdump 的 Bhrama(由 G-Rom 出品的著名脱壳工具）来初始化 dumping。用户必须提供窗口的名称，可以从标题条找到它。为了使工作简单化，可以在 winice.dat 里设置 F3 键：  
F3="/BHRAMA ProcDump32 - Dumper Server;"

4） /TRACEX <low EIP> \[<high EIP>\]  
　　控制跟踪器并退出 SoftICE。注意该命令只能用于跟踪当前线程，如果要跟踪其它线程，请使用/TRACE 命令。  

　　/TRACEX <low EIP>： 跟踪当前线程。注意，如果跟踪当前线程弹出 SoftICE 窗口后想继续跟踪，必须使用/TRACEX 命令，否则跟踪器会失去对当前线程的控制。  
　　 当线程的 EIP 到达<low EIP>时，跟踪停止并弹出 SoftICE 窗口。这也要求 EIP 真正可以到达，否则 SoftICE 不会弹出。  

　　/TRACEX <low EIP> <high EIP> 跟踪当前线程，注意事项同上。  
　　当线程的 EIP 到达<low EIP>与<high EIP>之间的区域内时停止并弹出 SoftICE 窗口。注意这里没有作<low EIP>和<high EIP>的边界检查，所以错误的参数地址会使 SoftICE 不能中断。

5） /SCREENDUMP \[<文件名>\]  
　　把 SoftICE 屏幕内容保存到一个文件中。注意该功能只支持通用显示驱动模式。这个命令的用法类似于/DUMP，如果没有指定<文件名>，IceDump 将在模式 0、1、2、3 和 4 中切换。  
　　模式 1：默认模式，将以 ASCII 格式输出。  
　　模式 0：字节属性也将被抓取。  
　　模式 2：可以把屏幕内容保存成一个 HTML 文件。  
　　模式 3：会把屏幕内容保存成 LaTeX 格式的文件。  
　　模式 4 ：把屏幕内容保存为 EPS (encapsulated Postscript)格式。

2．NticeDump 操作简介  
　　Nticedump 远不如 IceDump 功能强大，并且 Nticedump 装载方式不同于 IceDump，它是通过给 SoftICE 打补丁来实现 0 特权级控制权的，这是因为在 Windows 2000 上，要切换到 0 特权级不象 Windows9x 那么容易了。  
　　要打补丁的文件是\\WINNT\\SYSTEM32\\DRIVERS\\Ntice.sys，在 Nticedump 目录里有一补丁工具 ntid.exe，把安装目录下相应 SoftICE 版本的 Icedump 文件与 ntid.exe 一同复制到\\WINNT\\SYSTEM32\\DRIVERS\\目录下，然后运行 ntid.exe 程序就能正确补丁 Ntice.sys。这样 Nticedump 和 SoftICE 就完全结合了。

1） 抓取内存数据：PAGEIN D 基地址 长度 文件名  
例: PAGEIN D 400000 512 \\??\\C:\\memory.dmp  

注意： 在 NT 输入输出管理系统中，象"C:\\memory.dmp"不是合法路径。"\\??\\C:\\filename.dmp"是在 C 盘根目录下创建"filename.dmp"文件。

2） 抓取进程: PAGEIN B <Bhrama 窗口名>  
例: PAGEIN B ProcDump32 - Dumper Server

3） 导入文件: PAGEIN L 基地址 长度 文件名  
Example: PAGEIN L 400000 512 \\??\\C:\\memory.dmp  

4） 帮助: PAGEIN 例: PAGEIN 

 4dd466d0e1e0cc123afbe504da2462ca
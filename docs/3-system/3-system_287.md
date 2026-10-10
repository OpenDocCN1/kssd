

# 我个人认识的 windows 启动过程（求验证版）

> 原文：[`www.pediy.com/kssd/pediy11/115171.html`](https://www.pediy.com/kssd/pediy11/115171.html)

个人认识的 windows 启动过程。。。![](img/b0e6af7069663aededb11e4e29a3e6d1.jpg "碌眼")![](img/b0e6af7069663aededb11e4e29a3e6d1.jpg "碌眼")![](img/b0e6af7069663aededb11e4e29a3e6d1.jpg "碌眼")![](img/b0e6af7069663aededb11e4e29a3e6d1.jpg "碌眼")  
  
因为最近要用到这些，发出来求各位验证下，希望多多指教。。。网上资料的个人理解版。请莫笑。  
  
我把 windows 启动看成三部分，windows 引导过程、内核加载过程、用户登录会话建立过程。  
  
一：windows 引导过程  
  
    当机箱接通电源，首先得到控制的是 BIOS 代码。其中的功能主要有两个：一是执行必要的硬件检测工作，二是允许用户通过一些功能键来配置当前系统的硬件设置（BIOS 设置）。。。  
  
  
完成这两个功能之后，BIOS 将控制权交给硬盘主引导记录（MBR），MBR 是由 windows 安装程序在安装 windows 的时候填充的，里面的代码被成为引导代码。引导代码主要完成的功能是检索分区以得到引导分区，然后将引导分区的第一个扇区读入内存。。。。、  
  
之后，MBR 将控制权交与引导扇区。  
  
引导扇区的主要职责是给 windows 提供有关该硬盘上卷的结构与文件系统格式方面的信息，并且在当前卷的根目录中，读入 windows 的加载程序，即 ntldr 文件。这两个功能就要求了引导扇区必须具备能够识别文件系统和读取文件的全部代码。为了摆脱引导扇区盘块的限制，微软的做法是，使引导扇区的内容去加载其它扇区的程序来完成该操作，在这里，引导扇区就相当于一个 loader.  
  
在将 ntldr 加载入内存后，引导扇区将控制权交给 ntldr 的入口点  
  
在 ntldr 之前，系统是工作在 dos 实模式之下，而 windows 必须工作在保护模式。为了引导 windows 的启动，ntldr 首要的工作就是做好实模式和保护模式的切换。  
ntldr 分两个部分组成。第一部分为实模式代码，另一部分为 windows 可执行文件代码，其中第二部分也叫做 os loader。  
  
在引导扇区交控制权给 ntldr 之后，首先得到控制权的是实模式代码它完成实模式下设备的初始化，为切换到保护模式做好环境的准备之后，便通知处理器将环境切换为保护模式，然后交控制权与 os loader。  
  
由于刚刚进入保护模式，虚拟地址转移机制尚未开启，所以 os loader 首先要把物理内存管理起来，用一个内存描述符数组把每一段内存的大小和用途全都记录下来，然后构建页目录，页表，再设置好也目录寄存器，最后打开页面映射机制。在这个过程中，有些初始化必须在实模式下进行，所以 os loader 有时候会进行实模式和保护模式的切换。  
  
os loader 随后就在引导分区的根目录下面读入 boot.ini 文件，然后，清楚屏幕，并检查根目录下有没有 hiberfil.sys 文件。这个文件是休眠标志。如果这个文件存在，则说明这次计算机启动是一次休眠唤醒的过程，这个时候，os loader 会交控制权给系统唤醒代码。  
  
如果 hiberfil.sys 不存在，则 os loader 则解析 boot.ini 的引导选项来显示引导选择菜单（如果只有一项菜单便跳过显示）。  
  
注：解释一下 boot.ini 文件。  
  
\[boot loader\]  
timeout=3  
default=multi(0)disk(0)rdisk(0)partition(1)\\WINDOWS  
\[operating systems\]  
multi(0)disk(0)rdisk(0)partition(1)\\WINDOWS="Microsoft Windows XP Professional" /noexecute=optin /fastdetect  
  
格式分两部分，boot loader（引导加载），Operating Systems(操作系统）。  
timeout：系统选择菜单的等待时间，default 为默认操作系统  
Multi 表示一个非 SCSI 硬盘设备或一个由 SCSI BIOS 访问的 SCSI 硬盘，此处也可以为 SCSI，表示一个 SCSI BIOS 禁止的 SCSI 硬盘，后面跟的数字表示操作系统的系统根目录所在分区所在的硬盘所在的磁盘控制器在同一磁盘控制器上的序号。  
disk：对于 SCSI 硬盘，y 表示操作系统的系统根目录所在分区所在的硬盘在同一磁盘控制器上的硬盘序号。对于 multi，disk 的值恒为 0；  
rdisk：表示磁盘的序列号。  
Partition：表示分区序号。  
（boot.ini 的资料出自百度百科）。  
  
当用户选择所要登录的操作系统之后，os loader 加载执行 NTDETECT.COM，这是 16 位程序，主要功能是利用操作系统的 BIOS 来查询系统的基本设备和配置信息。这些信息在注册表初始化之后会放在 HKLM\\HARD WARE\\DESCRIPTION 中。  
  
等待 NTDETECT.COM 执行完毕，os loader 开始加载内核模块映像文件，默认为 ntoskrnl.exe。以及 HAL 映像文件（HAL.DLL）。然后在加载服务启动中标记为 SERVICE\_BOOT\_START 启动的设备驱动，以及文件系统驱动程序。到这里位置，引导操作系统所需要的前提才算全部被加载，同是 OS LOADER 构造参数块 LOADER\_PARAMETER\_BLOCK。该参数块定义如下：  
  
typedef struct \_LOADER\_PARAMETER\_BLOCK {  
    LIST\_ENTRY LoadOrderListHead;  
    LIST\_ENTRY MemoryDescriptorListHead;  
    LIST\_ENTRY BootDriverListHead;  
    ULONG\_PTR KernelStack;  
    ULONG\_PTR Prcb;  
    ULONG\_PTR Process;  
    ULONG\_PTR Thread;  
    ULONG RegistryLength;  
    PVOID RegistryBase;  
    PCONFIGURATION\_COMPONENT\_DATA ConfigurationRoot;  
    PCHAR ArcBootDeviceName;  
    PCHAR ArcHalDeviceName;  
    PCHAR NtBootPathName;  
    PCHAR NtHalPathName;  
    PCHAR LoadOptions;  
    PNLS\_DATA\_BLOCK NlsData;  
    PARC\_DISK\_INFORMATION ArcDiskInformation;  
    PVOID OemFontFile;  
    struct \_SETUP\_LOADER\_BLOCK \*SetupLoaderBlock;  
    PLOADER\_PARAMETER\_EXTENSION Extension;  
  
    union {  
        I386\_LOADER\_BLOCK I386;  
        // ALPHA\_LOADER\_BLOCK Alpha;  
        // IA64\_LOADER\_BLOCK Ia64;  
    } u;  
  
  
} LOADER\_PARAMETER\_BLOCK, \*PLOADER\_PARAMETER\_BLOCK;  
  
该参数块一般是由 OS LOADER 填充，剩下的由内核初始化的时候填充。（哪些是由 OS LOADER 填充，那些是由内核初始化填充？？？）  
  
到此为止，ntldr 任务完成，交出控制权给内核模块的出口函数：kiSystemStartup。  
  
windows 引导过程结束，内核初始化开始。  
  
  
二：内核初始化  
  
ntldr 交控制权于 ntoskrnl.exe。内核初始化开始。  
  
ntoskrnl 的入口函数为 kiSystemStartup。由于模块间耦合的关系，内核初始化没有办法线性执行。而是将过程分为两个阶段。阶段 0 和阶段 1，启动模块靠着一个标识位来确定当前启动是处于阶段 0 还是阶段 1。  
  
阶段 0：初始化阶段 1 要用到的数据，在该阶段，中断被禁止。kiSystemStartup 首先初始化处理器状态。包括 IDT、TSS、PCR、然后调用 HalInitializeProcess 为当前处理器初始化其 HAL 中的 PCR 和处理器间中断向量，接着调用 kiinitialineKernel 初始化内核，完成之后即变为空闲。在阶段 0 初始化的时候，进程管理器 PCR 创建一个进程对象 idle，创建 system.exe 进程。  
  
kiInitializeKernel 返回后，kiSystemStartup 启动中断。开始允许新线程的创建。  
  
到此时，阶段 0 启动完毕。阶段 1 开始启动。初始化函数 phase1Initialization，间接调用 KeStartAllProcess 启动其它的处理器。其它的处理器启动的过程和主处理器相似，但少了很多初始化的过程。  
  
在阶段 1 的初始化过程中，在屏幕上开始显示 windows 图标和进度。  
  
过程如下：  
  
1、设置标识为 1，表示阶段 1 开始。  
2、HalInitSystem，执行 HAL 的阶段 1 初始化。  
3、初始化图形引导驱动。  
4、调用 PoInitSystem 完成电源初始化。  
5、调用 HalQueryRealTimeCloce,完成时间的初始化；  
  调用 keSetSystemTime 设置时间。  
6、SeStartAllProcessers，启动所有的处理器。  
7、完成阶段 1 初始化。  
8、进度 10%  
9、创建\\SystemRoot 符号链接，完成内存管理器初始化。  
10、加载国家语言支持，初始化缓存管理器，初始化配置管理器，初始化注册表编辑器，初始化内核预读器。  
11、进度 15%  
12、初始化文件系统支持库。初始化调试器(kdcom.dll），初始化即插即用设备。  
13、进度 20%  
14、初始化 LPC 子系统，初始化执行本地组件。  
15、进度 25%  
16、调用 IoInitSystem 函数来初始化 I/O 子系统。初始化 I/O 子系统中的状态变量，创建驱动程序对象和设备对象。加载“引导-启动”类型驱动。加载“系统-启动”类型驱动。  
17、进度 75%。  
18、内核模块标记为可分页。  
19、进度 80%  
20、电源管理器，设备管理器完全初始化。  
21、进度 85%  
22、完全引用监视器初始化（SRM），创建安全引用监视器的命令服务线程，该线程创建一个“引用监视器命令端口”的 LPC 端口，接受 lsass.exe 进程传来的安全服务命令。  
23、进度 90%  
24、创建会话管理器（smss.exe）进程。  
25、进度 100%。  
26、启动 ZwWaitForSigltObject 等待 smss.exe 进程五秒钟，五秒钟内 smss 退出则系统崩溃。(注意：会话管理器虽然为系统进程，但是在系统中并没有做内核守护，平时结束了 smss.exe，是不会引起系统崩溃的，原因就在这里。)  
  
  
内核初始化完毕。用户登录会话开始。  
  
  
三：用户登录会话建立  
  
smss.exe 获得控制权，用户登录会话开始。  
  
首先，初始化注册表内容。（内核启动阶段 1 仅仅启动了注册表管理器和 system 储巢）。。  
  
smss.exe 要启动的东西在注册表中的位置：  
  
HKLM\\SYSTEM\\CurrentControlSet\\Control\\Session Manager  
  
主要键值按启动顺序排列介绍如下：  
  
BootExecute  运行在启动时执行的程序。自动校验程序。  
  
FileRenameOperation  启动时文件删除重命名。  
  
\\KnownDlls  打开已知 dll（系统，全局共享）  
  
\\Memory Management\\PagingFiles  创建页面文件，也就是虚拟内存。  
  
Environment  全局环境变量。  
  
\\SabSystems\\kmode  windows 子系统的内核模块。  
  
\\subSystems\\Windows  启动 windows 子系统进程(csrss.exe）  
  
创建 LPC 端口对象为启动 lsass 做好准备。  
  
启动 winlogon.exe,初始化登录窗体和默认桌面。  
  
内核 ZwWaitForSigleObject 来等待 winlogon.exe 和 csrss.exe 进程，有一个结束则系统崩溃。  
  
启动 Services.exe，启动自启动的服务和设备。  
  
启动本地权威子系统 lsass.exe 进程，建立 LPC 连接。  
  
Winlogon 获得用户名和口令后，首先调用 lsass.exe 的 lsaLookupAuthenticationPackage 函数以获得认证包，然后由 lsass 来验证。  
  
Winlogon 检查 UseInit 的值，启动进程（默认为 UseInit.exe）。  
  
UserInit.exe 检查 Shell 的值(默认为 explore.exe），启动 explore。  
  
UserInit 退出。  
  
启动 Run 键值下，自启动文件夹下的启动项。windows 启动完成。。。![](img/cb6288ad81b189b02921e98a92da6abe.jpg "124.gif")![](img/cb6288ad81b189b02921e98a92da6abe.jpg "124.gif")![](img/cb6288ad81b189b02921e98a92da6abe.jpg "124.gif")![](img/cb6288ad81b189b02921e98a92da6abe.jpg "124.gif")![](img/cb6288ad81b189b02921e98a92da6abe.jpg "124.gif")


# 虚拟机检测技术剖析

> 原文：[`www.pediy.com/kssd/pediy11/119969.html`](https://www.pediy.com/kssd/pediy11/119969.html)

标题：【原创】虚拟机检测技术剖析  
作者：riusksk (泉哥)  
主页：[`riusksk.blogbus.com`](http://riusksk.blogbus.com)  
  
**前言**  
在当今信息安全领域，特别是恶意软件分析中，经常需要利用到虚拟机技术，以提高病毒分析过程的安全性以及硬件资源的节约性，因此它在恶意软件领域中是应用越来越来广泛。这里我们所谓的虚拟机（Virtual Machine）是指通过软件模拟的具有完整硬件系统功能的、运行在一个完全隔离环境中的完整计算机系统。通过虚拟机软件（比如 VMware，Virtual PC ,VirtualBox），你可以在一台物理计算机上模拟出一台或多台虚拟的计算机，这些虚拟机完全就像真正的计算机那样进行工作，例如你可以安装操作系统、安装应用程序、访问网络资源等等。攻击者为了提高恶意程序的隐蔽性以及破坏真实主机的成功率，他们都在恶意程序中加入检测虚拟机的代码，以判断程序所处的运行环境。当发现程序处于虚拟机（特别是蜜罐系统）中时，它就会改变操作行为或者中断执行，以此提高反病毒人员分析恶意软件行为的难度。本文主要针对基于 Intel CPU 的虚拟环境 VMware 中的 Windows XP SP3 系统进行检测分析，并列举出当前常见的几种虚拟机检测方法。  
  
**方法一：通过执行特权指令来检测虚拟机**  
Vmware 为真主机与虚拟机之间提供了相互沟通的通讯机制，它使用“IN”指令来读取特定端口的数据以进行两机通讯，但由于 IN 指令属于特权指令，在处于保护模式下的真机上执行此指令时，除非权限允许，否则将会触发类型为“EXCEPTION\_PRIV\_INSTRUCTION”的异常，而在虚拟机中并不会发生异常，在指定功能号 0A（获取 VMware 版本）的情况下，它会在 EBX 中返回其版本号“VMXH”；而当功能号为 0x14 时，可用于获取 VMware 内存大小，当大于 0 时则说明处于虚拟机中。VMDetect 正是利用前一种方法来检测 VMware 的存在，其检测代码分析如下：  

代码:

```
bool IsInsideVMWare()
{
  bool rc = true;

  \_\_try
  {
    \_\_asm
    {
      push   edx
      push   ecx
      push   ebx

      mov    eax, 'VMXh'
      mov    ebx, 0  // 将 ebx 设置为非幻数’VMXH’的其它值
      mov    ecx, 10 // 指定功能号，用于获取 VMWare 版本，当它为 0x14 时用于获取 VMware 内存大小
      mov    edx, 'VX' // 端口号
      in     eax, dx // 从端口 dx 读取 VMware 版本到 eax
//若上面指定功能号为 0x14 时，可通过判断 eax 中的值是否大于 0，若是则说明处于虚拟机中
      cmp    ebx, 'VMXh' // 判断 ebx 中是否包含 VMware 版本’VMXh’，若是则在虚拟机中
      setz   \[rc\] // 设置返回值

      pop    ebx
      pop    ecx
      pop    edx
    }
  }
  \_\_except(EXCEPTION\_EXECUTE\_HANDLER)  //如果未处于 VMware 中，则触发此异常
  {
    rc = false;
  }

  return rc;
}

```

测试结果：  
![](img/6a78c3baead7d42c80181efee3a59587.jpg)  
                 图 1  
如图 1 所示，VMDetect 成功检测出 VMWare 的存在。  
  
**方法二：利用 IDT 基址检测虚拟机**  
利用 IDT 基址检测虚拟机的方法是一种通用方式，对 VMware 和 Virtual PC 均适用。中断描述符表 IDT（Interrupt Descriptor Table）用于查找处理中断时所用的软件函数，它是一个由 256 项组成的数据，其中每一中断对应一项函数。为了读取 IDT 基址，我们需要通过 SIDT 指令来读取 IDTR（中断描述符表寄存器，用于 IDT 在内存中的基址），SIDT 指令是以如下格式来存储 IDTR 的内容：  
代码:

```
typedef struct
{
    WORD IDTLimit;    // IDT 的大小
    WORD LowIDTbase;  // IDT 的低位地址
    WORD HiIDTbase;  // IDT 的高位地址
} IDTINFO;

```

由于只存在一个 IDTR，但又存在两个操作系统，即虚拟机系统和真主机系统。为了防止发生冲突，VMM（虚拟机监控器）必须更改虚拟机中的 IDT 地址，利用真主机与虚拟机环境中执行 sidt 指令的差异即可用于检测虚拟机是否存在。著名的“红丸”（redpill）正是利用此原理来检测 VMware 的。Redpill 作者在 VMware 上发现虚拟机系统上的 IDT 地址通常位于 0xFFXXXXXX，而 Virtual PC 通常位于 0xE8XXXXXX，而在真实主机上正如图 2 所示都位于 0x80xxxxxx。Redpill 仅仅是通过判断执行 SIDT 指令后返回的第一字节是否大于 0xD0，若是则说明它处于虚拟机，否则处于真实主机中。Redpill 的源码甚是精简，源码分析如下：  
代码:

```
#include <stdio.h>
int main () {

  unsigned char m\[2+4\], rpill\[\] = "\\x0f\\x01\\x0d\\x00\\x00\\x00\\x00\\xc3";  //相当于 SIDT\[adrr\],其中 addr 用于保存 IDT 地址
  \*((unsigned\*)&rpill\[3\]) = (unsigned)m;  //将 sidt\[addr\]中的 addr 设为 m 的地址
  ((void(\*)())&rpill)();  //执行 SIDT 指令，并将读取后 IDT 地址保存在数组 m 中

  printf ("idt base: %#x\\n", \*((unsigned\*)&m\[2\]));   //由于前 2 字节为 IDT 大小，因此从 m\[2\]开始即为 IDT 地址
  if (m\[5\]>0xd0) printf ("Inside Matrix!\\n", m\[5\]); //当 IDT 基址大于 0xd0xxxxxx 时则说明程序处于 VMware 中
  else printf ("Not in Matrix.\\n");
  return 0;
}

```

测试结果如图 2 所示：  
![](img/5bd1c01239d399ed218dbb9a8de188db.jpg)  
                                     图 2  
利用此 IDT 检测的方法存在一个缺陷，由于 IDT 的值只针对处于正在运行的处理器而言，在单 CPU 中它是个常量，但当它处于多 CPU 时就可能会受到影响了，因为每个 CPU 都有其自己的 IDT，这样问题就自然而然的产生了。针对此问题，Offensive Computing 组织成员提出了两种应对方法，其中一种方法就是利用 Redpill 反复地在系统上循环执行任务，以此构造出一张当前系统的 IDT 值变化统计图，但这会增加 CPU 负担；另一种方法就是 windows API 函数 SetThreadAffinityMask()将线程限制在单处理器上执行，当执行此测试时只能准确地将线程执行环境限制在本地处理器，而对于将线程限制在 VM 处理器上就可能行不通了，因为 VM 是计划在各处理器上运行的，VM 线程在不同的处理器上执行时，IDT 值将会发生变化，因此此方法也是很少被使用的。为此，有人提出了使用 LDT 的检测方法，它在具有多个 CPU 的环境下检测虚拟机明显优于 IDT 检测方法，该方法具体内容参见下节内容。  
  
**方法三：利用 LDT 和 GDT 的检测方法**  
在 《Intel 64 and IA-32  Architecture Software Developer’s Manual Volume 3A: System Programming Guide》第二章的 Vol.3 2-5 一页（我的 Intel 开发手册是 2008 版的）中对于 LDT 和 GDT 的描述如下（以下内容为个人翻译）：  
在保护模式下，所有的内存访问都要通过全局描述符表（GDT）或者本地描述符表（LDT）才能进行。这些表包含有段描述符的调用入口。各个段描述符都包含有各段的基址，访问权限，类型和使用信息，而且每个段描述符都拥有一个与之相匹配的段选择子，各个段选择子都为软件程序提供一个 GDT 或 LDT 索引（与之相关联的段描述符偏移量），一个全局/本地标志（决定段选择子是指向 GDT 还是 LDT），以及访问权限信息。  
若想访问段中的某一字节，必须同时提供一个段选择子和一个偏移量。段选择子为段提供可访问的段描述符地址（在 GDT 或者 LDT 中）。通过段描述符，处理器从中获取段在线性地址空间里的基址，而偏移量用于确定字节地址相对基址的位置。假定处理器在当前权限级别（CPL）可访问这个段，那么通过这种机制就可以访问在 GDT 或 LDT 中的各种有效代码、数据或者堆栈段，这里的 CPL 是指当前可执行代码段的保护级别。  
……  
GDT 的线性基址被保存在 GDT 寄存器（GDTR）中，而 LDT 的线性基址被保存在 LDT 寄存器（LDTR）中。   
  
由于虚拟机与真实主机中的 GDT 和 LDT 并不能相同，这与使用 IDT 的检测方法一样，因此虚拟机必须为它们提供一个“复制体”。关于 GDT 和 LDT 的基址可通过 SGDT 和 SLDT 指令获取。虚拟机检测工具 Scoopy suite 的作者 Tobias Klein 经测试发现，当 LDT 基址位于 0x0000（只有两字节）时为真实主机，否则为虚拟机，而当 GDT 基址位于 0xFFXXXXXX 时说明处于虚拟机中，否则为真实主机。具体实现代码如下：  
代码:

```
#include <stdio.h>

void LDTDetect(void)
{
    unsigned short ldt\_addr = 0;
    unsigned char ldtr\[2\];

    \_asm sldt ldtr
    ldt\_addr = \*((unsigned short \*)&ldtr);
    printf("LDT BaseAddr: 0x%x\\n", ldt\_addr);

    if(ldt\_addr == 0x0000)
    {
        printf("Native OS\\n");
    }
    else
        printf("Inside VMware\\n");
}

void GDTDetect(void)
{
    unsigned int gdt\_addr = 0;
    unsigned char gdtr\[4\];

    \_asm sgdt gdtr
    gdt\_addr = \*((unsigned int \*)&gdtr\[2\]);
    printf("GDT BaseAddr:0x%x\\n", gdt\_addr);

    if((gdt\_addr >> 24) == 0xff)
    {
        printf("Inside VMware\\n");
    }
    else
        printf("Native OS\\n");
}

int main(void)
{
    LDTDetect();
    GDTDetect();
    return 0;
}

```

测试结果如图 3 所示：  
![](img/2639c63d405da5ee443c258e135c4256.jpg)  
                   图 3  
  
**方法四：基于 STR 的检测方法**  
在保护模式下运行的所有程序在切换任务时，对于当前任务中指向 TSS 的段选择器将会被存储在任务寄存器中，TSS 中包含有当前任务的可执行环境状态，包括通用寄存器状态，段寄存器状态，标志寄存器状态，EIP 寄存器状态等等，当此项任务再次被执行时，处理器就会其原先保存的任务状态。每项任务均有其自己的 TSS，而我们可以通过 STR 指令来获取指向当前任务中 TSS 的段选择器。这里 STR（Store task register）指令是用于将任务寄存器 (TR) 中的段选择器存储到目标操作数，目标操作数可以是通用寄存器或内存位置，使用此指令存储的段选择器指向当前正在运行的任务的任务状态段 (TSS)。在虚拟机和真实主机之中，通过 STR 读取的地址是不同的，当地址等于 0x0040xxxx 时，说明处于虚拟机中，否则为真实主机。实现代码如下：  
代码:

```
#include <stdio.h>
int main(void)
{
    unsigned char mem\[4\] = {0};
    int i;

    \_\_asm str mem;
    printf (" STR base: 0x");
    for (i=0; i<4; i++)
    {
        printf("%02x",mem\[i\]);
    }

    if ( (mem\[0\]==0x00) && (mem\[1\]==0x40))
        printf("\\n INSIDE MATRIX!!\\n");
    else
        printf("\\n Native OS!!\\n");
    return 0;
}

```

测试结果如图 4 所示：  
![](img/f678704bcdfc402bc2edc347186ef9b0.jpg)  
                          图 4  
  
**方法五：基于注册表检测虚拟机**  
在 windows 虚拟机中常常安装有 VMware Tools 以及其它的虚拟硬件（如网络适配器、虚拟打印机，USB 集线器……），它们都会创建任何程序都可以读取的 windows 注册表项，因此我们可以通过检测注册表中的一些关键字符来判断程序是否处于虚拟机之中。关于这些注册表的位置我们可以通过在注册表中搜索关键词“vmware”来获取，下面是我在 VMware 下的 WinXP 中找到的一些注册表项：  
  
项名：HKEY\_CLASSES\_ROOT\\Applications\\VMwareHostOpen.exe  
项名：HKEY\_CLASSES\_ROOT\\Installer\\Products\\C2A6F2EFE6910124C940B2B12CF170FE\\ProductName  
键值“VMware Tools”  
项名：HKEY\_CLASSES\_ROOT\\Installer\\Products\\C2A6F2EFE6910124C940B2B12CF170FE\\SourceList\\PackageName  
键值：VMware Tools.msi  
项名：HKEY\_CURRENT\_USER\\Printers\\DeviceOld  
键值：\_#VMwareVirtualPrinter,winspool,TPVM:  
项名：HKEY\_LOCAL\_MACHINE\\HARDWARE\\DEVICEMAP\\Scsi\\Scsi Port 0\\Scsi Bus 0\\Target Id 0\\Logical Unit Id 0\\Identifier  
键值：VMware Virtual IDE Hard Drive  
项名：HKEY\_LOCAL\_MACHINE\\HARDWARE\\DEVICEMAP\\Scsi\\Scsi Port 1\\Scsi Bus 0\\Target Id 0\\Logical Unit Id 0\\Identifier  
键值：NECVMWar VMware IDE CDR10  
项名：HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Classes\\Installer\\Products\\C2A6F2EFE6910124C940B2B12CF170FE\\ProductName  
键值：VMware Tools  
项名：HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Installer\\UserData\\S-1-5-18\\Products\\C2A6F2EFE6910124C940B2B12CF170FE\\InstallProperties\\DisplayName  
键值：VMware Tools  
项名：HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Reinstall\\0002\\DeviceDesc  
键值：VMware SVGA II  
项名：HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\NetworkCards\\2\\Description  
键值：VMware Accelerated AMD PCNet Adapter  
项名：HKEY\_LOCAL\_MACHINE\\SOFTWARE\\VMware, Inc.\\VMware Tools  
项名：HKEY\_LOCAL\_MACHINE\\SYSTEM\\ControlSet001\\Control\\Class\\{4D36E968-E325-11CE-BFC1-08002BE10318}\\0000\\DriverDesc  
键值：VMware SVGA II  
项名：HKEY\_LOCAL\_MACHINE\\SYSTEM\\ControlSet001\\Control\\Class\\{4D36E968-E325-11CE-BFC1-  
08002BE10318}\\0000\\ProviderName  
键值：VMware, Inc.  
项名：HKEY\_LOCAL\_MACHINE\\SYSTEM\\ControlSet001\\Control\\Class\\{4D36E972-E325-11CE-BFC1-08002bE10318}\\0001\\DriverDesc  
键值：VMware Accelerated AMD PCNet Adapter  
项名：HKEY\_LOCAL\_MACHINE\\SYSTEM\\ControlSet001\\Control\\Class\\{4D36E97B-E325-11CE-BFC1-08002BE10318}\\0000\\DriverDesc  
键值：VMware SCSI Controller  
项名：HKEY\_LOCAL\_MACHINE\\SYSTEM\\ControlSet001\\Control\\Print\\Monitors\\ThinPrint Print Port Monitor for VMWare  
  
除以上这些表项之外，还有很多地方可以检测，特别是虚拟机提供的虚拟化软硬件、服务之类，比如文件共享服务，VMware 物理磁盘助手服务，VMware Ethernet Adapter Driver，VMware SCSI Controller 等等的这些信息都可作为检测虚拟机的手段。这里我们就以其中某表项为例编程举例一下，其它表项检测方法同理，具体代码如下：  
代码:

```
.386
.model flat, stdcall
option casemap:none

   include  windows.inc
   include  user32.inc
   include  kernel32.inc
   include  advapi32.inc

   includelib  user32.lib
   includelib  kernel32.lib
   includelib  advapi32.lib

.data
szCaption     db "VMware Detector ",0
szInside         db "Inside VMware!",0
szOutside              db "Native OS!",0
szSubKey      db "software\\VMWare, Inc.\\VMware tools",0
hKey              dd    ?

.code
start:
  invoke RegOpenKeyEx, HKEY\_LOCAL\_MACHINE, addr szSubKey, 0,\\
                 KEY\_WRITE or KEY\_READ, addr hKey
  .if eax == ERROR\_SUCCESS
  invoke MessageBox, NULL,addr szInside, addr szCaption, MB\_OK
  .else
  invoke MessageBox, NULL,addr szOutside, addr szCaption, MB\_OK
  .endif
  invoke RegCloseKey,hKey
  invoke ExitProcess,NULL
end start

```

测试结果如图 5 所示：  
![](img/3a2f504f8efe446959dedd57ac5e0f58.jpg)  
            图 5  
  
**方法六：基于时间差的检测方式**  
本方法通过运行一段特定代码，然后比较这段代码在虚拟机和真实主机之中的相对运行时间，以此来判断是否处于虚拟机之中。这段代码我们可以通过 RDTSC 指令来实现，RDTSC 指令是用于将计算机启动以来的 CPU 运行周期数存放到 EDX：EAX 里面，其中 EDX 是高位，而 EAX 是低位。下面我们以 xchg    ecx,  eax 一句指令的运行时间为例，这段指令在我的真实主机 windows 7 系统上的运行时间为 0000001E，如图 6 所示：  
![](img/57f7041e1f95d5792f1da1c258c10ee3.jpg)  
              图 6  
而该指令在虚拟机 WinXP 下的运行时间为 00000442，如图 7 所示：  
![](img/68dbc1d06a4b4f10e44ac9db58641c52.jpg)  
               图 7  
两者之间的运行时间明显差别很多，在虚拟机中的运行速度远不如真实主机的，一般情况下，当它的运行时间大于 0xFF 时，就可以确定它处于虚拟机之中了，因此不难写出检测程序，具体实现代码如下：  
代码:

```
.586p
.model flat, stdcall
option casemap:none

include      windows.inc
include       kernel32.inc
include      user32.inc

includelib    kernel32.lib
includelib    user32.lib

      
.data
szTitle      db  "VMDetect With RDTSC", 0h
szInsideVM    db  "Inside VMware!", 0h
szOutsideVM    db  "Native OS!", 0h

.code

start:
  RDTSC
  xchg     ecx, eax
  RDTSC  
  sub    eax, ecx
  cmp    eax, 0FFh
  jg    Detected
  
  invoke  MessageBox, 0, offset szOutsideVM, offset szTitle, 0
  ret
  
Detected:
  invoke   MessageBox, 0, offset szInsideVM, offset szTitle, 0
  ret
end start

```

测试结果如图 8 所示：  
![](img/6c4f32cf9cbe3d46259eaa585c8a3153.jpg)  
              图 8  
  
**方法七：利用虚拟硬件指纹检测虚拟机**  
利用虚拟硬件指纹也可用于检测虚拟机的存在，比如 VMware 默认的网卡 MAC 地址前缀为“00-05-69，00-0C-29 或者 00-50-56”，这前 3 节是由 VMware 分配的唯一标识符 OUI，以供它的虚拟化适配器使用。在我的 VMWare WinXP 下的 MAC 地址为 00-0C-29-5B-D7-67，如图 9 所示：  
![](img/eb0763cb8381be9425a7d925dc135150.jpg)  
                                 图 9  
但由于这些可经过修改配置文件来绕过检测。另外，还可通过检测特定的硬件控制器，BIOS，USB 控制器，显卡，网卡等特征字符串进行检测，这些在前面使用注册表检测方法中已有所涉及。另外之前在看雪论坛上也有朋友提到通过检测硬盘 Model Number 是否含有“vmware”或“virtual”等字样来实现检测虚拟机的功能，网址见这（附源码）：[`bbs.pediy.com/showthread.php?t=110046。`](http://bbs.pediy.com/showthread.php?t=110046。)  
  
**总结**  
国外 SANS 安全组织的研究人员总结出当前各种虚拟机检测手段不外乎以下四类：  
●  搜索虚拟环境中的进程，文件系统，注册表；  
●  搜索虚拟环境中的内存  
●  搜索虚拟环境中的特定虚拟硬件  
●  搜索虚拟环境中的特定处理器指令和功能  
因为现代计算系统大多是由文件系统，内存，处理器及各种硬件组件构成的，上面提到的四种检测手段均包含了这些因素。纵观前面各种检测方法，也均在此四类当中。除此之外，也有人提出通过网络来检测虚拟机，比如搜索 ICMP 和 TCP 数据通讯的时间差异，IP ID 数据包差异以及数据包中的异常头信息等等。随着技术研究的深入，相信会有更多的检测手段出现，与此同时，虚拟机厂商也会不断进化它们的产品，以增加 anti-vmware 的难度，这不也正是一场永无休止的无烟战争！  
  
文档下载：  
虚拟机检测技术剖析.doc
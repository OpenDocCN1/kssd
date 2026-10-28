
# [原创]Carberp Bootkit源码解析

**Carberp Bootkit 源码解析**

By boywhp@126.com

## 一、BootKit 安装程序生成

Carberp 主体代码位于 `BKGen`、`BkSetup` 和 `FJ` 文件夹主要是安装程序和文件拼接工具，

`BkBuild` 目录 `bkbuild.bat` 批处理完成具体的安装程序生成，如下：

![](https://bbs.kanxue.com/upload/attach/201312/118994_wsty13gcl0702en.png)

1、用 `bkgen` 生成 `vbr.com` 这个就是实际的 Bootkit 功能代码

2、使用 `fj` 文件拼接 `bksetup` 安装程序，其配置文件是 `bksetup.cfg`，其配置大致如下：

![](https://bbs.kanxue.com/upload/attach/201312/118994_cpfflj109cz3l53.png)

`BKInstall` 安装程序 = `BkSetup` + `vbr.com` + `payloader`

为了防止被人分析以及特征码扫描，`BkGen` 会对生成 `vbr.com` 执行随机加密变形处理，不方便我们接下来的调试分析，需要给 `BkGen` 的 `bme.c` [`Block-mixing engine`]打几个补丁。

补丁 1：关闭代码块乱序

![](https://bbs.kanxue.com/upload/attach/201312/118994_d5mvmhzxg8fhsq2.png)

补丁 2：关闭随机生成 jmp 指令

![](https://bbs.kanxue.com/upload/attach/201312/118994_tr3g2h24d0e7ff2.png)

补丁 3：关闭随机 XOR 加密

![](https://bbs.kanxue.com/upload/attach/201312/118994_edb3rbe00lny8j2.png)

## 二、BootKit 调试环境配置

这里我走了些坑爹的弯路，IDA+VMWARE，我调试的目标机是 windows 2003 x86，具体的配置见下面官方教程：

a3bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2Z5k6i4S2Q4x3X3c8J5j5i4W2K6i4K6u0W2j5$3!0E0i4K6u0r3M7s2u0G2k6s2g2U0N6s2y4Q4x3V1k6A6k6r3q4Q4x3V1k6K6N6i4m8H3L8%4u0@1i4K6u0r3N6s2g2@1L8%4u0A6j5h3I4K6i4K6u0r3k6r3g2T1N6h3N6Y4K9h3&6Y4i4K6g2X3k6$3c8T1i4K6g2X3L8r3W2F1N6i4S2Q4y4h3j5`.

vmware.pdf

这里尤其要注意的是 VMWARE 的 CPU 设置，一定不要设置成什么单处理器多核心啥的，否则 IDA 调试的时候会出现无法单步执行或者奇怪的跑飞现象，我因为这个原因都换成了 IDA+Bochs 调试，坑爹啊，所以请务必注意下 VMWARE 的 CPU 设置。

![](https://bbs.kanxue.com/upload/attach/201312/118994_0c338h5iy4jk933.png)

## 三、BootKit 源码解析

### （一）生成器整体结构

Carberp bookit 主体功能代码使用 masm 汇编编写，分为 `vbs16.com` 以及 `vbr32.asm` 其中 `vbs16.com` 运行于实模式，`vbr32.asm` 代码运行于保护模式，其主代码块定义在 `vbr32.asm`。

![](https://bbs.kanxue.com/upload/attach/201312/118994_eje20iddkym894h.png)

`BME` 块混淆引擎通过读取该表，对代码块进行乱序，随机插入跳转指令以及 XOR 加密操作，第三列指明了该块的标志，具体含义如下：

![](https://bbs.kanxue.com/upload/attach/201312/118994_dehuuq43jvglib9.png)

`BME` 使用常数映射完成生成器同生成代码的参数传递，`variables.h` 具体如下：

![](https://bbs.kanxue.com/upload/attach/201312/118994_zt8c294co9jo1sn.png)

Carberp bookit 函数重定位实现（CALL EBP/BP 机制），Carberp 使用了一个类似 API 导入函数表的机制，来完成运行时函数之间相互的调用，具体来说就是：

1、`BME` 引擎额外生成了一个函数偏移地址块![](https://bbs.kanxue.com/upload/attach/201312/118994_0e43iw0f445tuz1.png)，该数据块记录了各个函数块的相对 `CorePtr` 的地址。

![](https://bbs.kanxue.com/upload/attach/201312/118994_7gp7g9h1vu9mja1.png)

![](https://bbs.kanxue.com/upload/attach/201312/118994_kd072if15oc3o3c.png)

2、`Core16_start` 核心重定位函数

![](https://bbs.kanxue.com/upload/attach/201312/118994_h4pu2fgf9ss2sa8.png)

![](https://bbs.kanxue.com/upload/attach/201312/118994_4hxuiki1tsxy2x3.png)

`var_offset_core` 变量指向该函数偏移，bk初始化时将ebp/bp指向该函数地址，然后通过如下方式：![](https://bbs.kanxue.com/upload/attach/201312/118994_0j2fguvig3o7xql.png)执行函数调用

`Core16` 函数会读取返回地址+1 的函数索引 ID，然后通过查函数偏移表得到函数地址并调用该函数，该机制同时会混淆调试器对汇编代码的解析，调试的时候需要注意。

### （二）入口点

Carberp 实际上是感染 NTFS `"$Boot"` Sectors，该扇区位于分区的第二个扇区，安装程序会从第二个扇区+0x20 开始搜索 jmp 指令，具体如下：

![](https://bbs.kanxue.com/upload/attach/201312/118994_8xusah9e6zhmaq8.png)

然后根据偏移地址，执行扇区数据替换为 `vbr.com`，该扇区最终被加载到 0xD200 偏移位置。

IDA 启动调试器，断点 0x7C00，F9 执行，第二次命中断点单步跟踪一会即可看到：

![](https://bbs.kanxue.com/upload/attach/201312/118994_37z1y9ei6zfkm34.png)

入口 0xD00:26A = 0xD26A，IDA 调试时，注意 ALT+S 设置为 16 位段 汇编模式

![](https://bbs.kanxue.com/upload/attach/201312/118994_e0buvgvfrqtkmze.png)

这里有一点比较有意思就是这段入口点的汇编代码和原始的 NTLDR 入口点汇编完全一致，除了那个 CALL near 地址不一样，我估计应该是增加欺骗性用的吧,对应源码如下：

![](https://bbs.kanxue.com/upload/attach/201312/118994_mjefhxyugh5hp4y.png)

最后 `call bp;db F_Startup` 会被 `BME` 引擎替换成一个 `call near` 指令，因为该入口块 `B_NOEBP` 标识表明了此时 bp 尚未初始化，不能使用 `call bp` 调用方式，`F_Startup` 对应的正是 `Startup16_start` 的索引。

### （三）Startup16_start

![](https://bbs.kanxue.com/upload/attach/201312/118994_v959lp06hk99qr2.png)

获取运行时基地址，使用最常见的 `call /pop` 定位代码位置。

BK 接下来需要分配内存，同时需要把自己的内存空间保留起来，以免内存被正常的启动代码覆盖掉，

![](https://bbs.kanxue.com/upload/attach/201312/118994_on6g4hzeec72m78.png)

![](https://bbs.kanxue.com/upload/attach/201312/118994_4hpg2tnbge4rj5j.png)

BK 从 BIOS Data Area 可用内存空间最高地址保留了 3KB 用来存放 BK 自身代码

![](https://bbs.kanxue.com/upload/attach/201312/118994_8h1vxqxj10jfzui.png)

接下来 BK 将自身拷贝到该内存地址的同时完成 XOR 解码

![](https://bbs.kanxue.com/upload/attach/201312/118994_ny7plpiagvrtte3.png)

最后初始化 bp->CORE pointer 、跳转到 `var_block0_ofs` 函数继续执行，`var_block0_ofs` 是代码块索引 0 相对于 `CORE_PTR` 的偏移地址，`var_block0_ofs` + `var_offset_core` = `Main16_start`【索引 0】。

![](https://bbs.kanxue.com/upload/attach/201312/118994_vnb7xpjh08fm6uy.png)

### （四）Main16_start

该函数主要完成原始 Loader 的解压、`INT15 HOOK` 以及 payload 加载。BK 安装程序将原始 Loader 压缩后保存在 BK 代码之后，PAYLOAD 代码保存在磁盘的 NTFS 保留扇区内，BK 代码通过 `FileName` 变量获取 PAYLOAD 的位置信息，`FileName` 由 `BkSetup` 程序根据实际扇区位置写入【搜索 `0x33333333` 定位`FileName`】。

`FileName` 定义如下：

![](https://bbs.kanxue.com/upload/attach/201312/118994_n9cvxc3pxkncryr.png)

实际磁盘布局如下：

![](https://bbs.kanxue.com/upload/attach/201312/118994_nw1krjzzbk26h74.png)

![](https://bbs.kanxue.com/upload/attach/201312/118994_l5jbwwginqa7vkn.png)

首先将 packed loader 向后挪动，以免解压时原始数据被破坏

![](https://bbs.kanxue.com/upload/attach/201312/118994_erabn8qsx211ma0.png)

![](https://bbs.kanxue.com/upload/attach/201312/118994_mrrk0aapzndkner.png)

0xD00:BE8 复制到 0xD00:206A，腾出 `NTFS_LOADER_SIZE` 空间，以便接下来解压 Loader。然后执行解压，BK 使用了开源轻量级压缩库 aPLib 对原始 LOADER 进行压缩，网址如下：

25eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3W2T1M7$3g2F1M7$3!0X3N6s2N6S2M7X3g2Q4x3X3g2U0L8$3#2Q4x3V1k6H3M7X3!0V1N6h3y4@1M7#2)9#2k6X3q4b7e0r3W2T1i4K6u0W2K9s2c8E0L8l9`.`.

![](https://bbs.kanxue.com/upload/attach/201312/118994_xsjy1us1h38cppw.png)

具体解压算法就不研究了，aPLib 本身也提供了 asm 版的解压实现，估计该 BK 的作者也就简单修改了下。解压完成后，D00:26A 入口点就是原始代码数据了。

接下来调用 `LoadData` 函数把 PAYLOAD 数据加载到内存。

![](https://bbs.kanxue.com/upload/attach/201312/118994_xnyjbdxbp0qom0y.png)

`LoadData` 根据 PAYLOAD 大小，申请 BDA 保留内存空间，然后使用 `INT13` 服务读磁盘，加载 PAYLOAD 到申请到的内存地址，PAYLOAD 内存地址同时保存到 `FileName` 数据结构中，以便以后使用。

最后执行 `INT15 HOOK`

![](https://bbs.kanxue.com/upload/attach/201312/118994_4eot4oqyqys8csh.png)

`GETPTR` 宏获取指定函数 ID 的偏移地址，返回结果为 EAX，定义如下：

![](https://bbs.kanxue.com/upload/attach/201312/118994_ziko7lqac27yq0n.png)

`SALC` 会被 `BME` 引擎重新解析为 `MOV CL，XX`，实际汇编如下：

![](https://bbs.kanxue.com/upload/attach/201312/118994_htt8xlkhwqe39zi.png)

### （五）Handler15

该函数完成 `INT15` 的 HOOK，BK 需要处理额外处理下 BIOS INT 15，0xE820 内存检测，欺骗 windows 系统，以防自身保留的代码空间被操作系统意外使用了。

a39K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6A6K9$3W2Q4x3X3g2G2M7$3c8W2N6W2)9J5k6h3!0J5k6#2)9J5c8V1c8W2N6r3g2U0N6r3W2F1k6#2)9#2k6V1#2W2L8h3!0J5P5g2)9#2k6W2)9J5z5s2R3^5y4W2)9J5z5b7`.`.

![](https://bbs.kanxue.com/upload/attach/201312/118994_jkbvt9ab3v1ycgf.png)

BK 首先测试 `INT15` 调用者的 cs，实际 NTLDR 会被加载到 cs = 0x2000 的位置，从而执行 `H15_check_ldr` 功能代码。

![](https://bbs.kanxue.com/upload/attach/201312/118994_cjt8guxae3dqfqp.png)

上述代码搜索 NTLDR 的特征码 `0F-22-C0-87-DB`，对应汇编为 `MOV CR0, EAX` / `XCHG BX, BX`，实际对应位置如下：

![](https://bbs.kanxue.com/upload/attach/201312/118994_eae9zybb5vaoz2r.png)

这段代码设置 CR0 寄存器，开启 CPU 换页和保护模式，该函数实际为 `EnableProtectPaging`。

接下就是对 LDR 代码执行补丁操作了。

![](https://bbs.kanxue.com/upload/attach/201312/118994_9j3wqaozk839hnu.png)

`PatchLdr` 首先保存原始特征码到 BK 的尾部空闲内存，然后将其替换为 `9a CALL FAR JMP` 到 `PmInit`。

![](https://bbs.kanxue.com/upload/attach/201312/118994_pjf1vaw39mvgdgr.png)

现在 BK 已基本完成了前期的准备工作，他悄悄的在 `EnableProtectPaging`（开启换页和保护模式）关键代码上种下了一个诅咒 `PmInit`。

### （六）PmInit

一旦 Windows 系统准备好，开启换页的时候，BK 的 `PmInit` 函数就会得到执行，实际上，这个时间点是在 OsLoader 完成内存子系统初始化后，第一次调用 Su 导出函数 `HardwareCursor` 完成后执行的。

![](https://bbs.kanxue.com/upload/attach/201312/118994_ecyi07f4n6obhf1.png)

Su 在每个导出函数前后都加了一个一小段代码，每个导出函数调用完成后，都会执行一次 `EnableProtectPaging`。

`PmInit` 首先从 BK 尾部内存取出原始数据，恢复先前的 Hook 点。

![](https://bbs.kanxue.com/upload/attach/201312/118994_l4ic3t0ic0lz6tm.png)

由于 BK 使用了 DR 寄存器硬件断点功能，一旦断点命中，CPU 会执行 IDT INT1 处理，因此首先需要对 IDT（中断分发表）INT1 执行 HOOK，使其指向 `Int1Handler` 函数。

![](https://bbs.kanxue.com/upload/attach/201312/118994_h02ai1ctqcyisxj.png)

`HookIdt` 中完成 INT1 HOOK，同时下了硬件断点 DR0 （IDT 读写 - 针对 NT6）：

![](https://bbs.kanxue.com/upload/attach/201312/118994_knqjxnevqhbk42s.png)

硬件断点 DR2（0x401000 执行 - 针对 NT5）

![](https://bbs.kanxue.com/upload/attach/201312/118994_fb51s3fzkm95u30.png)

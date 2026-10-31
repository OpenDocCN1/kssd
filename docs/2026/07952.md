
# [原创]Windows相关保护模式分析笔记(1)

首先声明本帖首发于赏金论坛，希望大家能多多关注这个正在成长的论坛[a0cN6%4N6%4i4K6u0W2M7$3N6G2L8r3c8U0L8W2)9J5k6h3y4G2L8b7`.`.](https://bbs.kanxue.com/elink@cc3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4Y4L8$3I4V1j5$3&6Q4x3X3g2U0L8$3@1`.)

最近正在出一套编写操作系统雏形的教程([1f2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4Y4L8$3I4V1j5$3&6Q4x3X3g2U0L8$3#2Q4x3V1k6@1K9s2u0W2j5h3c8Q4x3X3g2H3K9s2m8Q4x3@1k6X3K9h3c8Q4x3@1b7^5y4l9`.`.](https://bbs.kanxue.com/elink@f2bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4Y4L8$3I4V1j5$3&6Q4x3X3g2U0L8$3#2Q4x3V1k6@1K9s2u0W2j5h3c8Q4x3X3g2H3K9s2m8Q4x3@1k6X3K9h3c8Q4x3@1b7^5y4l9`.`.)),顺便整理了下保护模式的知识,逆向分析了下Windows的保护模式相关代码。

调试工具：`vmware`+`windbg`双机调试，`bochs`。

`vmware`+`windbg`双机调试网上有很多资料，我这里就不说了（缺点是不能分析Windows设置GDT、IDT的过程）。

`bochs`我这里说下，大家可以去下载最新版本的`bochs`，我没有制作`bochs`虚拟磁盘，比较麻烦，想到一个最简单的方法，直接使用`xp2 winpe`的ISO镜像，最新版的`bochs`不需要自己编写脚本，用`bochs.exe`可以直接生成配置文件。

设置步骤如下：

1. 双击`bochs.exe`设置虚拟机引导和磁盘

![](https://bbs.kanxue.com/upload/attach/201108/162380_sjkckxajrdpif92.jpg)

2. 设置引导设备为CD-ROM

![](https://bbs.kanxue.com/upload/attach/201108/162380_xup1n0t5a86hh0u.jpg)

3. 设置引导设备为`*.iso`镜像

![](https://bbs.kanxue.com/upload/attach/201108/162380_3ul04b4el31yi0q.jpg)

点击save可以保存生成`*.bxrc`配置文件。

用`bochs`主要是能看到一些硬件信息，相当于硬件调试器，但是从`bochsdbg`的输出结果来看`bochs`现在还没支持`MSR`指令，`windbg`无法观察内核设置分段、分页等代码，而`bochs`可以模拟硬件，能详尽分析这部分代码；所以我在分析时大部分用依赖与`bochs`。

（`bochs`的命令是兼容`GDB`，与`windbg`不同，我下了份源码，试着改下命令，发现找不到头绪，如果有牛人改好，请发出了分享下。）

做好配置文件`*.bxrc`可以使用bat脚本：`"C:\Program Files\Bochs-2.4.6\bochsdbg" -q -f "D:\内核笔记\winpe XP.bxrc"` 来启动调试器`bochsdbg`，`bochs`的命令可以在网上搜索下，有很多参考整理，`bochs`的文档是最好的指南。

## Windows进入保护模式的步骤

1. 分段准备GDT，加载GDT
2. 分页式寻址支持虚拟地址

## 一. 保护模式分段寻址选择

如果未开启分页，任然沿用`seg:offset`方式寻址。

GDT（Global Descriptor Table Register，全局描述符表寄存器）：描述符数组，有以下特点：

1. 段寄存器中存放段选择子Selector
2. `GDTR`中存放着段描述符表的首地址
3. 通过选择子根据`GDTR`中的首地址，就能找到对应的段描述符
4. 段描述符中有段的物理首地址，就得到段在内存中的首地址
5. 加上偏移量，就找到在这个段中存放的数据的真正物理地址。

寻址示意图如下：

![](https://bbs.kanxue.com/upload/attach/201108/162380_temhn3g7iyh042i.jpg)

分段寻址有一个重要的数据结构，GDT数组，在进入保护模式前准备好这个数组，由`lgdt`设置`gdt`。

IDT（Interrupt Descriptor Table，中断描述符）也是类似于GDT的一个数组，是为了描述中断异常而设置的。

这里我先不解释描述符（Descriptor），我们可以暂时理解为描述内存信息（段基地、段界限、段属性等）的数据结构。

### (1) bochs观察GDT表与IDT表

第一次打开`bochsdbg`，会在准备好BIOS时断下输入命令`c`（contine）。

```
<bochs:1> c
```

跑一分钟左右（注意时间不要太短），就可以用`bochs`命令查看：

```
<bochs:2> info gdt 

Global Descriptor Table (base=0x000000000003f000, limit=1023):           (GDTR信息)

GDT[0x00]=??? descriptor hi=0x00000000, lo=0x00000000              (GDT表第一项不用,每位设置为0)

GDT[0x01]=Code segment, base=0x00000000, limit=0xffffffff, Execute/Read, 32-bit         

GDT[0x02]=Data segment, base=0x00000000, limit=0xffffffff, Read/Write, 

GDT[0x03]=Code segment, base=0x00000000, limit=0xffffffff, Execute/Rea 

GDT[0x04]=Data segment, base=0x00000000, limit=0xffffffff, Read/Write 

GDT[0x05]=32-Bit TSS (Busy) at 0x00024460, length 0x00077 

GDT[0x06]=Data segment, base=0x00000000, limit=0x00001fff, Read/Write 

GDT[0x07]=Data segment, base=0x00000000, limit=0x00000fff, Read/Write, 

GDT[0x08]=Data segment, base=0x00000400, limit=0x0000ffff, Read/Write 

GDT[0x09]=??? descriptor hi=0x00000000, lo=0x00000000 

GDT[0x0a]=32-Bit TSS (Available) at 0x00023b7e, length 0x0006f 

GDT[0x0b]=Code segment, base=0x00020000, limit=0x0000ffff, Execute/Read, 16-bit
```

```
<bochs:3> info idt 

Interrupt Descriptor Table (base=0x000000008003f400, limit=2047              (IDTR 信息)

IDT[0x00]=32-Bit Interrupt Gate target=0x0008:0x80408bff, DPL=0 

IDT[0x01]=32-Bit Interrupt Gate target=0x0008:0x80408d7c, DPL=0
```

注意这两个命令都可以直接加索引查看指定项或者范围，如下：

```
<bochs:4> info gdt 0 2 

Global Descriptor Table (base=0x000000000003f000, limit=1023): 

GDT[0x00]=??? descriptor hi=0x00000000, lo=0x00000000 

GDT[0x01]=Code segment, base=0x00000000, limit=0xffffffff, Execute/Read, Accesse 

GDT[0x02]=Data segment, base=0x00000000, limit=0xffffffff, Read/Write, Accessed 

GDT[0x03]=Code segment, base=0x00000000, limit=0xffffffff, Execute/Read, 32-bit
```

### (2) windbg查看gdt表，idt表查看GDT表基地址

```
kd> r gdtr

gdtr=8003f000                     (注意这时已经打开分页机制,基地址是虚拟地址)
```

查看GDT表界限

```
kd> r gdtl

gdtl=000003ff
```

同样查看idt表有

```
kd> r idtr

idtr=8003f400    （基地址,基地址是虚拟地址）
```

```
kd> r idtl

idtl=000007ff      （界限）
```

观察gdt数组表项（使用`dg [n] [n]`）

```
kd> dg 0 20

                                  P Si Gr Pr Lo

Sel    Base     Limit     Type    l ze an es ng Flags

---- -------- -------- ---------- - -- -- -- -- --------

0000 00000000 00000000 <Reserved> 0 Nb By Np Nl 00000000

0008 00000000 ffffffff Code RE Ac 0 Bg Pg P  Nl 00000c9b

0010 00000000 ffffffff Data RW Ac 0 Bg Pg P  Nl 00000c93

0018 00000000 ffffffff Code RE    3 Bg Pg P  Nl 00000cfa

0020 00000000 ffffffff Data RW Ac 3 Bg Pg P  Nl 00000cf3
```

观察idt数组表项（使用`!idt [-a]`）

```
  kd> !idt -a 

Dumping IDT: 

00: 8053f1ac nt!KiTrap00

01: 8053f324 nt!KiTrap01

02: Task Selector = 0x0058

03: 8053f6f4 nt!KiTrap03

04: 8053f874 nt!KiTrap04

05: 8053f9d0 nt!KiTrap05

06: 8053fb44 nt!KiTrap06

07: 805401ac nt!KiTrap07
```

综合`bochs`调试结果可以看出，Windows设置GDT表大小为1024，即1024/8 = 128个描述符；IDT表大小为2048，共有2048/8 = 256个表项。

下一节我教大家怎么分析Windows设置分段分页的代码。

winpe光盘镜像我上传到网盘：[6a3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2Q4x3X3f1I4x3e0g2Q4x3X3g2U0L8$3#2Q4x3V1k6X3K9h3I4W2i4K6u0r3k6r3&6Y4N6$3A6U0K9Y4f1`.](https://bbs.kanxue.com/elink@32fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2Q4x3X3f1I4x3e0g2Q4x3X3g2U0L8$3#2Q4x3V1k6X3K9h3I4W2i4K6u0r3k6r3&6Y4N6$3A6U0K9Y4f1`.)

我`bochs`配置文件与启动脚本，大家可以参考下：[bochs.rar](https://bbs.kanxue.com/attach-download-59484.htm)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [1.JPG](javascript:void(0);)（40.92kb，220次下载）
- [2.JPG](javascript:void(0);)（33.23kb，221次下载）
- [3.JPG](javascript:void(0);)（41.39kb，220次下载）
- [4.JPG](javascript:void(0);)（25.53kb，222次下载）
- [bochs.rar](javascript:void(0);)（1.22kb，31次下载）


# Windows 相关保护模式分析笔记(1)

> 原文：[`www.pediy.com/kssd/pediy12/138031.html`](https://www.pediy.com/kssd/pediy12/138031.html)
>
> 首先声明本帖首发于赏金论坛，希望大家能多多关注这个正在成长的论坛 `www.sgoldcn.com`,最近正在出一套编写操作系统雏形的教程 `http://www.sgoldcn.com/thread.php?fid=84`,顺便整理了下保护模式的知识,逆向分析了下 Windows 的保护模式相关代码。

> 调试工具: `vmware`+`windbg` 双机调试, `bochs`

> `vmware`+`windbg` 双机调试网上有很多资料,我这里就不说了(缺点是不能分析 windows 设置 `GDT`,`IDT` 的过程)

> `bochs` 我这里说下,大家可以去下载最新的版本的 `bochs`,我没有制作 `bochs` 虚拟磁盘,比较麻烦,想到一个最简单的方法,直接使用 `xp2` `winpe` 的 `ISO` 镜像,最新版的 `bochs` 不需要自己编写脚本,用 `bochs.exe` 可以直接生成配置文件

> 设置步骤如下:

> 1.双击 `bochs.exe` 设置虚拟机引导和磁盘

![](img/6e222c6985ea0f605ccd69c09f37c4dd.jpg)

> 2.设置引导设备为 `CD-ROM`

![](img/5937e16b4d8878d714c060836706a719.jpg)

> 3．设置引导设备为`*.iso` 镜像

![](img/6685677d474407847a119469131b64ef.jpg)

> 点击 `save` 可以保存生成`*.bxrc` 配置文件

> 用 `bochs` 主要是能看到一些硬件信息,相当于硬件调试器,但是从 `bochsdbg` 的输出结果来看 `bochs` 现在还没支持 `MSR` 指令,`windbg` 无法观察内核设置分段,分页等代码,而 `bochs` 可以模拟硬件,能详尽分析这部分代码;所以我在分析时大部分用依赖与 `bochs`.

> (`bochs` 的命令是兼容 `GDB`,与 `windbg` 不同,我下了份源码,试着改下命令,发现找不到头绪,如果有牛人改好,请发出了分享下.)

> 做好配置文件`*.bxrc` 可以使用 `bat` 脚本 `"C:\\Program Files\\Bochs-2.4.6\\bochsdbg" -q -f "D:\\内核笔记\\winpe XP.bxrc"` 来启动调试器 `bochsdbg`,`bochs` 的命令可以在网上搜索下,有很多参考整理,`bochs` 的文档是最好的指南.

## Windows 进入保护模式的步骤：

1. 分段准备 `GDT`，加载 `GDT`
2. 分页式寻址支持虚拟地址

## 一.保护模式分段寻址选择

如果未开启分页，任然沿用 `seg`：`offset` 方式寻址;

`GDT`（Global Descriptor Table Register，全局描述符表寄存器）：描述符数组 有以下特点:

1. 段寄存器中存放段选择子 `Selector`
2. `GDTR` 中存放着段描述符表的首地址
3. 通过选择子根据 `GDTR` 中的首地址，就能找到对应的段描述符
4. 段描述符中有段的物理首地址，就得到段在内存中的首地址
5. 加上偏移量，就找到在这个段中存放的数据的真正物理地址。

寻址示意图如下:

![](img/1cbd42134b06f336083c5307c1e0402.jpg)

分段寻址有一个重要的数据结构,`GDT` 数组，在进入保护模式前准备好这个数组，由 `lgdt` 设置 `gdt`

`IDT`（Interrupt Descriptor Table，中断描述符）也是类似于 `GDT` 的一个数组，是为了描述中断异常而设置的

这里我先不解释描述符(`Descriptor`),我们可以暂时理解为描述内存信息(段基地,段界限,段属性等)的数据结构.

### (1.)`bochs` 观察 `GDT` 表与 `IDT` 表

第一次打开 `bochsdbg`,会在准备好 `BIOS` 时断下输入命令 `c`(continue)

代码:

```
<bochs:1> c
```

跑一分钟左右（注意时间不要太短），就可以用 `bochs` 命令查看：

代码:

```
<bochs:2> info gdt 
Global Descriptor Table (base=0x000000000003f000, limit=1023):          (GDTR 信息)
GDT[0x00]=??? descriptor hi=0x00000000, lo=0x00000000                   (GDT 表第一项不用,每位设置为 0)
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

代码:

```
<bochs:3> info idt 
Interrupt Descriptor Table (base=0x000000008003f400, limit=2047           (IDTR 信息)
IDT[0x00]=32-Bit Interrupt Gate target=0x0008:0x80408bff, DPL=0 
IDT[0x01]=32-Bit Interrupt Gate target=0x0008:0x80408d7c, DPL=0

```

注意这两个命令都可以直接加索引查看指定项或者范围，如下：

代码:

```
<bochs:4> info gdt 0 2 
Global Descriptor Table (base=0x000000000003f000, limit=1023): 
GDT[0x00]=??? descriptor hi=0x00000000, lo=0x00000000 
GDT[0x01]=Code segment, base=0x00000000, limit=0xffffffff, Execute/Read, Accesed 
GDT[0x02]=Data segment, base=0x00000000, limit=0xffffffff, Read/Write, Accessed 
GDT[0x03]=Code segment, base=0x00000000, limit=0xffffffff, Execute/Read, 32-bit

```

### （2）`windbg` 查看 `gdt` 表，`idt` 表查看 `GDT` 表基地址

代码:

```
kd> r gdtr
gdtr=8003f000                 (注意这时已经打开分页机制,基地址是虚拟地址)
```

查看 `GDT` 表界限

代码:

```
kd> r gdtl
gdtl=000003ff
```

同样查看 `idt` 表有

代码:

```
kd> r idtr
idtr=8003f400    （基地址,基地址是虚拟地址）
```

代码:

```
kd> r idtl
idtl=000007ff      （界限）
```

观察 `gdt` 数组表项（使用 `dg [n] [n]`）

代码:

```
kd> dg 0 20
                                 P Si Gr Pr Lo
Sel    Base     Limit     Type    l ze an es ng Flags
\---- -------- -------- ---------- - -- -- -- -- --------
0000 00000000 00000000 <Reserved> 0 Nb By Np Nl 00000000
0008 00000000 ffffffff Code RE Ac 0 Bg Pg P   Nl 00000c9b
0010 00000000 ffffffff Data RW Ac 0 Bg Pg P   Nl 00000c93
0018 00000000 ffffffff Code RE    3 Bg Pg P   Nl 00000cfa
0020 00000000 ffffffff Data RW Ac 3 Bg Pg P   Nl 00000cf3
```

观察 `idt` 数组表项(使用 `!idt [-a]`)

代码:

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

综合 `bochs` 调试结果可以看出,windows 设置 `GDT` 表大小为 1024 即 1024/8 = 128 个描述符,`IDT` 表大小为 2048,共有 2048/8 = 256 个表项.

下一节我教大家怎么分析 windows 设置分段分页的代码.

`winpe` 光盘镜像我上传到网盘:[`u.115.com/file/dngwjcju`](http://u.115.com/file/dngwjcju)

我 `bochs` 配置文件与启动脚本，大家可以参考下:`bochs.rar`

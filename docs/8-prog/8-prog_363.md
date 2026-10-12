
# BIOS Rootkit 实现分析与检测技术研究系列之－－简单 ISA 模块编写

> 原文：[`www.pediy.com/kssd/pediy10/75479.html`](https://www.pediy.com/kssd/pediy10/75479.html)

## 本节目录

1. ISA 总线简介
2. BIOS 文件分析
3. 编写简单 ISA 模块
4. 小结
5. 附录
6. 相关附件

## 引言

作为目前 `BIOS Rootkit` 主要存在方式，在本小节中，我们一起来讨论一下简单 `ISA` 模块的编写。在这个小节里，我们主要讨论 `Award BIOS`（资料相对多一些 :p）。

## ISA 总线简介

20 世纪 80 年代初期，`IBM` 在推出自己的微机系统 `IBM PC/XT` 时，就定义了一种总线结构，称为 `XT` 总线。这是 8 位数据宽度的总线。随着 `IBM` 采用 `80286 CPU`，推出 `IBM PC/AT` 微机系统，又定义了与 `XT` 总线兼容的 16 位的 `AT` 总线。`ISA` 总线 (`Industrial Standard Architecture`) 即 `AT` 总线，它是在 8 位的 `XT` 总线基础上扩展而成的 16 位的总线体系结构。

目前 `ISA` 总线标准基本被 `PCI`（`Periphearl Component Interconnect`）总线标准替代。但在 `BIOS` 中还保留有对 `ISA` 调用的接口，主要是考虑到兼容老旧主板的需要。目前 `BIOS Rootkit` 多采用 `ISA` 模块编写方式，主要还是考虑到编写简单、向下兼容性好的优点。

下面我们逐步来进行分析。

## BIOS 文件分析

### 1. BIOS 文件的静态分析

#### 1) BIOS 文件一般结构分析

固化在 `BIOS` 芯片中的 `BIOS` 文件保存着计算机最重要的基本输入输出的程序、系统设置信息、开机上电自检程序和系统启动自举程序。这些程序采用模块化方式组织在一起，`Award`、`AMI` 或者 `Phoenix` 都采用这种组织方式。为了节约成本，这些模块一般都采用压缩的方式来进行存储。在需要调用模块时，在解压缩到内存执行。压缩算法主要采用 `LHA` 类似的压缩算法（对于 `LHA` 压缩算法还没深入了解![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬") ）。用 16 进制软件打开 `BIOS` 文件，我们可以发现 “-lh5-” 的字样，通过搜索这个字符串，也可以找出这个 `BIOS` 文件存在多少被压缩的模块。

通常来说 `BIOS` 文件一般都包含着 `BOOTBLOCK` 和各种初始化微程序，自动检测硬盘和 `CD` 驱动器、输出 `PnP/PCI` 设备表等硬件设备程序，支持 `ACPI` 高级电源管理的启动程序。另外还有各个主板厂家自己开发的功能扩展程序模块，以及各个主板厂家的 `BIOS` 启动 `logo` 等。

#### 2) Award BIOS 文件结构分析

`Award BIOS` 文件结构可以通过 `cbrom` 程序来查看。`cbrom` 的查看功能可以正常运行于 `DOS` 模式或 `Windows` 的虚拟 86 模式。在 `Windows XP` 下运行虚拟 86 控制台程序 `cmd.exe`（本文实验环境请参阅附录），把需要分析的 `BIOS` 文件（这里为 `awdbios.bin`）和 `cbrom` 所在路径放在一起，然后在 `cbrom` 所在路径下输入：

```
cbrom.exe awdbios.bin /D
```

![](img/4a101a20ee4e1ba3040a56c51ff25be9.jpg)

结果如下：

```
CBROM V2.20 (C)Phoenix Technologies, Ltd 2001 All Rights Reserved.

**************** awdbios.bin BIOS component ****************
No. Item-Name           Original-Size     Compressed-Size       Original-File-Name
====================================================================
0. System BIOS         20000h(128.00K)   13E89h(79.63K)         9a69vpab.BIN
1. XGROUP CODE         0B540h(45.31K)    07E64h(31.60K)         awardext.rom
2. CPU micro code      04000h(16.00K)    03F9Fh(15.91K)        CPUCODE.BIN
3. ACPI table           04067h(16.10K)    01994h(6.39K)         ACPITBL.BIN
4. YGROUP ROM          05F00h(23.75K)    03DC5h(15.44K)        awardeyt.rom
5. GROUP ROM[ 0]       038A0h(14.16K)    01A4Dh(6.58K)         _EN_CODE.BIN
6. Flash ROM           0A016h(40.02K)    0577Bh(21.87K)        AWDFLASH.EXE
7. PCI ROM[A]          0A000h(40.00K)    05F74h(23.86K)        RTSpxe_m.lom
8. ISA ROM[1]          10000h(64.00K)    0F062h(60.10K)        romos.bin

  Total compress code space  = 5A000h(360.00K)
  Total compressed code size = 41583h(261.38K)
  Remain compress code space = 18A7Dh(98.62K)
                      ** Micro Code Information **
Update ID CPUID  |  Update ID CPUID  |  Update ID CPUID  |  Update ID CPUID
-------------------------+---------------------------+----------------------------+-------------------------
PGA478 14    0F0A|  PGA478 2E    0F12|  PGA478 01    0F21|  PGA478 08    0F23
PGA478 1E    0F24|  PGA478 05    0F13 |  PGA478 15    0F25|  PGA478 37    0F27
PGA478 17    0F29|
```

从显示结果可以看出，该 `BIOS` 中包含 9 个模块。`BIOS` 通常以压缩的形式保存微程序（一般使用 `LHA` 压缩算法），这在显示结果中可以清楚的看到：每个模块类型、原始大小、压缩后的大小以及初始名字。

第一个模块为 `9a69vpab.bin`，`cbrom` 识别为 `System BIOS`。`BIOS` 主要的代码都集中在这里。其中包括 `BOOTBLOCK` 和各种初始化微程序。`BOOTBLOCK` 是 `BIOS` 的加载器，它负责在启动时初始化芯片组，把主要的 `BIOS` 代码解开，放入主内存。它也负责检查 `BIOS` 校验和，启动紧急恢复程序或切换到备用 `BIOS`。`BIOS` 加载入内存后，`BOOTBLOCK` 被加载到 `0FE000h`～`0FFFFFh` 这段地址中。为了和以往的 `IBM PC XT/AT` 达成撤销性兼容，程序结构、流程、跳跃点和若干变量参考几乎已经固定。一般 `BIOS` 版本在做日后的追加和修改都基本不会改动这个部分。业界一般称这部分为“激活区段”，即“`Boot Block`”。

第二个模块 `awardext.rom` 为 `BIOS` 的扩展部分，它在 `BIOS` 启动过程的最后阶段初始化设备。这个模块主要负责自动检测硬盘和 `CD` 驱动器、输出 `PnP/PCI` 设备表等工作。

`CPUCODE.BIN` 模块只是一组微程序。这组微代码用于修正开发中发生的错误。`Intel` 公司在 1996 年推出的 `Pentium Pro` 处理器，具备一项称为“微码更新”（`Microcode Update`）的功能。在某一批集成电路存在设计中的 `Bug` 时，可以用微码的方式加以修正。这就是这个模块的功能。在 `AMD` 和 `Intel` 的网站有最新的用于插入 `BIOS` 中的微代码，提供微码更新的功能。

`ACPITBL.BIN` 包含 `ACPI` 表。`ACPI`（`Advanced Configuration and Power Interface`）不仅是简单的电源管理器，它还能控制所有设备的主总线，甚至控制 `BIOS`。可以参考 `John Heasman` 的文章《Implementing and Detecting an ACPI BIOS Rootkit》。

`awardeyt.rom` 类似于 `awardext.rom`，也是 `BIOS` 扩展部分。它是各个厂商自己定制的不同于标准 `AwardBIOS` 的功能，实际上几乎所有的厂商都会增加这一部分内容。

`_EN_CODE.BIN` 模块是 `BIOS` 使用的一组 `ASCII` 字符串。

`AWDFLASH.EXE` 是嵌入到 `BIOS` 中的 `BIOS` 刷新工具，即上篇文章介绍的 `AwardBIOS Flash Utility` 程序。它允许在开机时运行该程序，对 `BIOS` 进行刷新和备份操作。

`RTSpxe_m.lom` 为主板厂家在 `BIOS` 中内置的程序模块，包括一些集成设备的驱动程序集合。

`romos.bin` 为我们加入的 `BIOS` 代码。可以注意到该模块显示为 `ISA` 模块，这正是目前 `BIOS Rootkit` 广泛采用的模块嵌入方式。实际上这个模块来自于一个开源的 `BIOS ROM` 嵌入项目 `ROMOS`（`Martin. The ROMOS Project. http://rayer.ic.cz/romos/romos.htm`）。我们后面的许多程序实例都借鉴于这个项目。

`cbrom` 并不能显示其他隐藏的模块，如 `decomp_blk.bin` 是独立的 `LHA` 解包器，负责对压缩模块的解压缩工作；又如 `boot_blk.bin` 模块，是真正的 `BOOTBLOCK` 模块，其包含在 `BIOS` 主模块 `9a69vpab.bin` 中等等。这些模块都可以通过 `Award BIOS Editor 1.0` 来查看和提取。

上面对于模块的具体描述主要来自 `Kris Kaspersky` 的《Shellcoder 编程揭秘》一书，上面对于 `BIOS` 模块的叙述很详细，不乏许多真知灼见，大家可以参考一下。

接下来，我们对 `BIOS` 文件中的模块进行详细剖析，以对其有一个直观的认识。

主模块 `9a69vpab.bin` 中包含着 `BOOTBLOCK`。而 `BOOTBLOCK` 加载到内存后位于 `000FE000h`～`000FFFFFh` 这个区段，而 `BIOS` 的入口点却在 `000FFFF0h`。入口点为何不在区段开始？其实，在 `000FFFF0h`～`000FFFFF` 这 16 个字节中存放着一条跳转指令，用于调转到 `BIOS` 真正的执行入口。

用 `Award BIOS Editor 1.0` 打开 `BIOS` 文件 `awdbios.bin`，将上述模块分别提取出来（选择菜单“`Actions->Extract All`”）。然后交给 `WinHex` 和 `IDA Pro` 做进一步分析。用 `WinHex` 打开提取出的文件 `9a69vpab.bin`，找到文件末尾，可以看到在文件尾部向前 16 个字节处的一段 16 进制数值：

```
EA 5B E0 00 F0
```

如图：

![](img/99f89f4ec48fb6e5e1970c7f2c29cd19.jpg)

用 `IDA Pro` 打开文件 `9a69vpab.bin`，选择 16 位汇编模式，找到文件尾部的这段数值，在 `EA` 字符上按 `C` 键（进行反汇编操作），我们得到这样的汇编指令：

```
jmp far ptr 0F000h : 0E05Bh
```

如图：

![](img/99eaa0c7f9dd24454d908c1d2147d451.jpg)

这句指令表示一个实模式下的跨段跳跃，跳到段地址为 `0F000h`，偏移地址为 `0E05Bh` 的地方继续执行，即物理地址 `0FE05Bh` 处（`BIOS` 内存分布请参看下面的分布图）。这基本是所有 `BIOS` 的 `BOOTBLOCK` 的执行方式。

`BIOS` 内存分布图（来源于互联网）：

![](img/1484d1e20dbc40b86dc7eab7b002c5da.jpg)

对于其他的模块我们更关心 `ISA/PCI` 类型的模块，如 `ISA` 模块 `romos.bin`。用 `IDA Pro` 可以顺利的进行反汇编。`IDA Pro` 不能识别模块的进入点，需要对 `ISA` 模块格式有清楚了解后进行手动反汇编（`ISA` 总线知识见上面描述）。`ISA` 模块一般用于管理集成的 `ISA` 控制器（例如额外的 `COM` 端口，当然也可以是我们的 `BIOS Rootkit`）。现在的主板上早就没有了 `ISA` 控制器，然而 `BIOS` 为了向后支持的需要，同样提供了对 `ISA` 模块的支持。`ISA` 模块编写相对简单易行，因此成为目前 `BIOS Rootkit` 主要采用的方式。

主 `BIOS` 模块 `9a69vpab.bin` 执行后，载入 `ISA` 模块，它将得到所有设备的完全控制权，包括 `PCI` 总线。也有一些 `BIOS Rootkit` 和 `PCI Rootkit` 对其增加了定制的 `PCI` 模块。定制的 `PCI` 模块伪造了 `PCI` 设备标志符，使它与真实设备的标志符相对应。`PCI` 头伪造技术在后面的系列中有详细分析。`ISA` 模块文件格式如下：

```
==============================================
  文件头部分  55h 0AAh            开头必须是这两个字节作为启动标志
               01h                 紧接的一个字节代表 ISA 模块的大小为
                                   N*512B 这里就是 1*512=512B
          jmp 程序进入点          一条跳转指令进入模块主体
-----------------------------------------------------------------------------------
  文件运行主体  程序进入点：      ISA 模块主体
               …
               …
               …
-----------------------------------------------------------------------------------
  文件结尾    一个字节          必须为整个模块的校验和
==============================================
```

另外，整个 `ISA` 模块大小必须为 512 字节的倍数。

`Kris Kaspersky` 在《Shellcoder 编程揭秘》一书中，编写了一个简单的用于嵌入 `BIOS` 的 `ISA` 模块程序。程序采用 `FASM` 编写，作用是在 `BIOS` 启动的时候设置一个登录界面，只有输入正确的密码才能继续进行系统加载，否则会循环提示用户输入登录密码。

由于 `FASM` 语法的特殊性，这个程序可能很多人编译通过后，在实际运行测试的时候都不能正常运行。经过摸索，发现问题出在开头的第一句：

```
DB    55h, 0Ah
```

上面我们刚说过 `ISA` 模块的开头必须为 `55h AAh` 这样的标志，否则系统将视为非法。但由于 `FASM` 语法的特殊性，如果程序写成：`DB    55h, 0AAh` 的话，将编译不通过，大家可以试试。这样，我们在后面进行 `ISA` 模块校验和计算的时候就会出错。所以在进行这个程序的校验和计算时，一定要先通过 16 进制编辑软件（如 `WinHex`），手动将模块开头的 `0A` 字节变为 `AA` 字节，然后进行校验和的计算，并写入模块文件末尾。这样完成的模块才能够正常运行。

对于模块校验的问题，`Kris Kaspersky` 给出了一个 `IDA` 脚本来进行。校验方法很简单，按照《Shellcoder 编程揭秘》一书的叙述就是：“把所有的字节加起来（求和），除了 100h，取余数：`sum=(sum+next_byte) & 0xFF`。整个块的校验和必须等于 0：因此，最后的字节应该是 `(100h-sum) & 0xFF`。” 用程序实现起来也很简单，这里就不贴了，大家可以自己动手试试。把校验和写入模块文件的操作，在书中有详细描述，这里就不多说了，大家可以对照书进行参考。

我把文章中的程序改为 `NASM` 语法格式，程序如下：

```
;------------------------------------------------------------------------
;
;  ISAOEM_NASM.ASM
;
```



### 1. 程序代码说明

改编自 Kris Kaspersky《Shellcoder 编程揭秘》一书中例子程序的 `ISAOEM.ASM`。

用 `NASM` 编译。

```
nasm o isaoem.bin isaoem_nasm.asm
```

```asm
;------------------------------------------------------------------------

org  0h        ; 运行于 BIOS 中

db  55h,0AAh   ; ISA 模块头

db  01h        ; 大小为 1*512 字节

jmp  x_code    ; 跳转到程序进入点

;------------------------------------------------------------------------

text  db  'Matrix has you!',0  ; 定义提示字符串

;------------------------------------------------------------------------

x_code:

    ; Preparing the registers
    ; ----------------------------------------
    mov  dx,101dh  ; Where to ouput (DH  Y, DL  X)
    mov  si,text   ; What to output
    xor    bx,bx   ; Initial color of characters -1
    mov  cx,1      ; Output one character at a time.
    ; Display color string.
    ; ----------------------------------------
  print_string:
    mov  ah,02h    ; Function for controlling the cursor
    int    10h     ; Position the cursor.
    inc    dl      ; Move to the next position.
    lodsb         ; Load the next character.
    test    al,al    ; Is this the end of the line?
    jz    input    ; Exit if yes.
    mov  ah,09h    ; Function for Printing a character
    inc    bl      ; Use all colors, one by one.
    int    10h     ; Print a character.
    jmp    print_string  ; Loop
  input:  ; Wait for the password.
      ; ----------------------------
    xor    dx,dx    ; Checksum
  enters:
    xor    ax,ax    ; Function for reading a character from the keyboard
    int    16h      ; Read the character.
    cmp    al,0dh    ; Is this ENTER?
    jz    input    ; If yes, start the input again.
    xor    ah,ah    ; Clear the scan code.
    add    dx,ax    ; Compute the CRC.
    cmp    dx,'m'+'s'+'o'+']'+'\'
    jnz    enters    ; If  the password is incorrect, continue.
    retf

times  512-($-$$)  db  0  ; 添加文件至 512 字节大小

;------------------------------------------------------------------------
```

程序编译后，用校验工具求和写入，然后用 `AwardBIOS Flash Utility` 刷写进 `BIOS` 中。重启计算机后，运行效果如图：

![图片不是源程序中出现的字符，提示字符换作“Hello XPM！”。“XPM”是是女朋友的称呼，密码当然设为他的生日，其他没变（照片是用手机抓拍的，手机质量差，图像照下来不太好看 :P）。由于这个密码登录框程序位于 BIOS 中，所以通过给 CMOS 放电的方法是破解不了的。诚如 Kris Kaspersky 所说，只有对 BIOS 进行刷写操作，否则无法破解（当然换一个相同的 BIOS 芯片也行 ^\_^）。](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")

### 2. BIOS 的动态分析

这个部分的内容主要参考陈文钦编著的《BIOS 研发技术剖析》，目的是让大家对 `BIOS` 运行后的一些具体操作有个感性的认识。

#### 1）. `BIOS` 一般运行流程分析

当计算机从开机的一瞬间开始，硬件特性就由 `CPU` 从主板的 `BIOS` 芯片内取得程序代码，用 `BIOS` 内部的程序代码获得控制权并且发挥作用。从 `CPU` 内外部的检测设置、激活 `DRAM` 以及针对芯片组与各种外围设备做初始化设置之后，最后驱动软盘或硬盘，直到把操作系统（如 `DOS`, `Win98/NT/2000/XP`, `Linux`）加载成功，`BIOS` 的开机引导工作就此告一段落，转向从事幕后的支持、协调工作，并帮助操作系统或应用程序，来处理外围设备沟通的细部操作。

#### 2）. `Award BIOS` 运行流程分析

`Intel 8086/8088` 处理器在 `CPU` 加电是所做的第一个动作就是设置：

```
CS=DS=ES=SS=0FFFFh  IP=0000h
```

即是将指令寄存器指向 `FFFF:0000h`，转换成物理地址也就是 `000FFFF0h`，随即开始获取指令码执行。`BIOS` 加载进内存后分布图参见上面。

从分布图中我们可以看出：`BOOTBLOCK` 位于 `000FE000h`～`000FFFFFh` 这个区段，而 `BIOS` 的入口点却在 `000FFFF0h`。确切的说，在 `000FFFF0h`～`000FFFFF` 这 16 个字节中存放着一个跳转指令，用于调转到 `BIOS` 的执行入口。这句跳转指令可以在 `BIOS` 文件的最尾部的 16 个字节看到）。

就主板 `BIOS` 而言，其最重要的工作集中在以下几个方面：

a. `POST`（`Power On Self Test` 开机自检测）。  
开机系统将控制权交给 `BIOS` 时，它会针对 `CPU` 各项寄存器，先检查其是否运行正常，接下来会检查 `8254 timer`（可编程外围计时芯片）、`9259A`（可编程中断器）、`8237 DMA Controller`（`DMA` 控制器）的状态。

b. `Initial`。  
`BIOS` 会针对动态内存（`DRAM`）、主板芯片组、显卡以及相关外围的寄存器（`register`）做初始化（`initialize`）设置，并检测是否能够正常工作。

c. 记录系统的设置值。  
`BIOS` 会记录系统的设置值，并且会存贮在非挥发性内存（`Non-Volatile RAM`），如 `CMOS` 或 `Flash Memory`（`ESCD` 区域）等。

d. 初始化常驻程序库。  
`BIOS` 会将常驻程序库（`Runtime Program`）常驻于某一段内存中。并将其提供给操作系统或应用程序进行调用，如 `int 10h`、`int 13h`、`int 16h` 等中断调用函数。

这些工作完成以后，`BIOS` 会检查是否存在 `ISA/PCI` 附加模块，如果有则加载运行（如以 `ISA/PCI` 模块形式存在的 `BIOS Rootkit`）。完成模块加载并成功运行后，`BIOS` 将调用 `int 19h` 中断，把磁盘第 0 柱头 0 磁道 1 扇区的系统启动数据，即 `MBR`（`Master Boot Record`）提取到内存 `0000:7c00` 处运行，从此完成 `BIOS` 向操作系统的 `CPU` 控制权移交工作。

### 小结

在这个小节里，我们讨论了 `BIOS` 文件结构特性，以及对作为主要 `BIOS Rootkit` 存在方式的 `ISA` 模块，进行了详细描述。这为进一步深入探讨 `BIOS Rootkit` 的实现，先做一个热身，积累些感性的认识。

### 附录

#### 附录：我的实验平台

======================================================
```
OS          Microsoft Windows XP (Service Pack 2)
CPU         Intel Celeron Northwood Processor (0.13um) 2.02GH2
BIOS        Phoenix Technologies Co.Ltd 6.00 PG 12/17/2003 512KB
主板编号  I845PE-W83627-9A69VPABC-00
主板芯片组  Intel 82845G/GL/GE Brookdale Host-Hub Interface Bridge
                          (B1-step) Epox Computer Co.Ltd
Bochs         V 2.3.7
NASM        V 2.03.01
IDA Pro       V 4.7
Cbrom        V 2.20
Award BIOS Editor  V 1.0
AwardBIOS Flash Utility   V 8.80
WinFlash            V 1.8
======================================================
```

#### 1. 硬件环境

*   `CPU`：`Intel Celeron Northwood Processor (0.13um) 2.02GH2`
*   内存：`DDR2 512M`

#### 2. 软件环境

*   操作系统：`Windows XP Professional SP2`；
*   开发平台：`NASM`
*   编程语言：`NASM`

======================================================

### 相关附件

附件中提供了文中实验用的 `awdbios.bin` 这个 `BIOS` 文件、`cbrom` 程序和 `ISAOEM_NASM.asm` 文件。供大家做实验用。

[附件.rar](附件.rar)

文中的程序在写入 `BIOS` 时存在一定风险，请谨慎使用！

待续...

![微笑](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")

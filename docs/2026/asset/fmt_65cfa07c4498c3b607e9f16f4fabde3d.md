
### 内存图

```Assembly
物理内存布局 (启动阶段):
0x00000000 ┌─────────────────────────────────────────┐
│ BIOS 数据区域 (BDA) │
0x00000500 ├─────────────────────────────────────────┤
│ 实模式代码/数据 │
0x00007C00 ├─────────────────────────────────────────┤
│ Bootloader (512字节) │
0x00010000 ├─────────────────────────────────────────┤
│ boot\_params (zeropage, 4KB) │ <- %esi 指向这里
0x00011000 ├─────────────────────────────────────────┤
│ 命令行参数 │
0x00090000 ├─────────────────────────────────────────┤
│ 实模式代码段 │
0x00100000 ├═════════════════════════════════════════┤
(1MB) ║ 压缩内核 vmlinuz ║ <- 典型加载位置
║ ┌─────────────────────────────┐ ║
║ │ startup\_32 (入口点) │ ║
║ │ startup\_64 │ ║
║ │ 压缩数据 │ ║
║ │ boot\_stack (16KB) │ ║
║ │ boot\_heap │ ║
║ └─────────────────────────────┘ ║
0x01000000 ├─────────────────────────────────────────┤
│ (示例: %ebp 加载地址) │
│ │
0x02000000 ├═════════════════════════════════════════┤
(示例) ║ 解压目标区域 (计算的 %ebx) ║
║ ┌─────────────────────────────┐ ║
║ │ pgtable (24KB) │ ║ <- 页表
║ │ PML4 (4KB) │ ║
║ │ PDPT (4KB) │ ║
║ │ PD×4 (16KB) │ ║
║ ├─────────────────────────────┤ ║
║ │ 解压后的内核 vmlinux │ ║
║ │ .text (代码段) │ ║
║ │ .data (数据段) │ ║
║ │ .bss (未初始化数据) │ ║
║ └─────────────────────────────┘ ║
0x???????? ├─────────────────────────────────────────┤
│ initrd (如果存在) │
... │ │
0xFFFFFFFF └─────────────────────────────────────────┘
(4GB)
```

## 实模式与保护模式

Real Address Mode (实模式):在此模式下地址访问的是真实地内存地址所在位置。

- 在`realmode`模式下,数据地址是根据`ds:x`和`es:x` (`ES`是一个"额外"的数据段寄存器。当需要同时访问两个不同的数据段时，`DS`用于一个，`ES`就用于另一个。)来进行访问的

- 比如`ds`为`0x7c0`，`si`为0则

```Assembly
ds<<4+si
0x7c00+0 = 0x7c00
```

- 指令地址是根据`cs:x`寄存器进行寻址的

Protected Address Mode (保护模式):采用虚拟内存，页等机制对内存进行保护。

## BIOS 阶段

### CPU状态

电源接通时,CPU的寄存器值为:

`CS` : `0xfffff000` 即此时代码执行的地址是`0xfffffff0` (BIOS程序所在的ROM区域)

- `CS` selector = `0xf000`

- `CS` base = `0xffff0000`

### BIOS 启动

BIOS 的固件程序会将硬盘启动区 512 B的数据原封不动复制给`0x7c00`这个位置,并且跳转到`0x7c00`

> 最后两字节也就是检查第一扇区的最后两字节 boot Signature 和 Magic Number 分别是否为`0x55`和`0xAA` (01010101 10101010这种交叉数据最容易检查传输是否错误)

>

> 或许你会疑惑为什么是`0x7c00` ,这里可以看一下这个文章:https://blog.csdn.net/hbuxiaofei/article/details/134587909

## Bootloader阶段

> 当 BIOS 执行完毕,就会将 bootloader 复制给`0x7c00`处

>

> 并且 BIOS 会将控制权给 bootloader

>

> 在早期版本下 bootloader 一般是 Linux 的 `boot/bootsect.s` 中定义的

>

> 在后面一般 bootloader 单独存在的软件

对于 x86/x86-64 架构：

- `GRUB2` (Grand Unified Bootloader 2) - 最常用

- `systemd-boot` (以前叫 `gummiboot`) - 简单轻量

- `rEFInd` - 主要用于 UEFI 系统

- `SYSLINUX`/`ISOLINUX` - 用于 Live CD/USB

对于 ARM 架构：

- `U-Boot` (Das U-Boot) - 嵌入式系统主流

- `GRUB2` - 也在一些 ARM 设备上使用

### GRUB

> 一般来说 `GRUB` 是最常见的 bootloader,其实 bootloader 干的事情基本一致,所以我们主要分析 `GRUB`

>

> 下文主要是引用的这个文章感兴趣可以直接看原文:https://xinqiu.gitbooks.io/linux-insides-cn/content/Booting/linux-bootstrap-1.html

`grub\_main` 初始化控制台，计算模块基地址，设置 root 设备，读取 `grub` 配置文件，加载模块。最后，将 `GRUB` 置于 normal 模式，在这个模式中，`grub\_normal\_execute` (from `grub-core/normal/main.c`) 将被调用以完成最后的准备工作，然后显示一个菜单列出所用可用的操作系统。(这个界面我们就比较熟悉了对吧,比如进入网吧我们就会看见这个界面)

### 内存布局

- 经过引导后，内存界面大概就是下方所示（对于具有启动协议版本 >= 2.02 的现代 `bzImage` 内核，一般使用的是下方的内存布局）,kernel 的 setup 程序(realmode code)加载到了内存中

- 当是 `bzImage` 时,保护模式的下的内核被重定位到`0x100000`

```SQL
| Protected-mode kernel |
100000 +------------------------+
| I/O memory hole |
0A0000 +------------------------+
| Reserved for BIOS | Leave as much as possible unused
~ ~
| Command line | (Can also be below the X+10000 mark)
X+10000 +------------------------+
| Stack/heap | For use by the kernel real-mode code.
X+08000 +------------------------+
| Kernel setup | The kernel real-mode code.
| Kernel boot sector | The kernel legacy boot sector.
X +------------------------+
| Boot loader | <- Boot sector entry point 0x7C00
001000 +------------------------+
| Reserved for MBR/BIOS |
000800 +------------------------+
| Typically used by MBR |
000600 +------------------------+
| BIOS use only |
000000 +------------------------+
```

- 内核加载器的传统内存映射，用于 Image 或 zImage 内核，通常如下所示

```SQL
| |
0A0000 +------------------------+
| Reserved for BIOS | Do not use. Reserved for BIOS EBDA.
09A000 +------------------------+
| Command line |
| Stack/heap | For use by the kernel real-mode code.
098000 +------------------------+
| Kernel setup | The kernel real-mode code.
090200 +------------------------+
| Kernel boot sector | The kernel legacy boot sector.
090000 +------------------------+
| Protected-mode kernel | The bulk of the kernel image.
010000 +------------------------+
| Boot loader | <- Boot sector entry point 0000:7C00
001000 +------------------------+
| Reserved for MBR/BIOS |
000800 +------------------------+
| Typically used by MBR |
000600 +------------------------+
| BIOS use only |
000000 +------------------------+
```

- 根据 `arch/x86/boot/header.S` 定义,可以知道 setup 一般在`0x90000`,`code32\_start`一般在`0x100000`

```YAML
setup\_move\_size: .word 0x8000 # size to move, when setup is not
# loaded at 0x90000. We will move setup
# to 0x90000 then just before jumping
# into the kernel. However, only the
# loader knows how much data behind
# us also needs to be loaded.
code32\_start: # here loaders can put a different
# start address for 32-bit code.
.long 0x100000 # 0x100000 = default for big kernel
```

- 总结

```Bash
┌─────────────────────────────────────────────────────────────┐
│ [时刻1] Bootloader (GRUB) 加载阶段 │
├─────────────────────────────────────────────────────────────┤
│ │
│ bzImage 文件结构： │
│ ┌──────────────────────┐ │
│ │ Setup.bin (32KB) │ ← header.S, main.c, pm.c 编译的 │
│ ├──────────────────────┤ │
│ │ vmlinux.bin.gz │ ← head\_64.S + 压缩的内核 │
│ └──────────────────────┘ │
│ │
│ Bootloader 的操作： │
│ 1. 读取 Setup.bin → 加载到物理地址 0x90000 │
│ 2. 读取 vmlinux.bin.gz → 加载到物理地址 0x100000 (1MB) │
│ 3. 跳转到 0x90000 开始执行 (header.S::\_start) │
│ │
│ 此时内存布局： │
│ 0x90000: ★Setup代码★ (header.S, main.c, pm.c...) │
│ 0x100000: ★压缩内核★ (head\_64.S::startup\_32) │
└─────────────────────────────────────────────────────────────┘
```

## Setup 阶段

> 当 bootloader 完成工作后,我们的 kernel setup代码(realmode)就被加载进入了内存中了

>

> 然后 bootloader 会将执行权限交给 kernel （也就是 `arch/x86/boot/header.S` `\_start` 函数开始执行）

`vmlinuz`是压缩后的kernel，于是可以将它分为两部分

- `setup.bin`

- 实模式的代码

- `vmlinux.bin`

- 保护模式的代码

`setup.bin` 程序的入口为`\_\_start`

```Assembly
OBJCOPYFLAGS\_setup.bin := -O binary
$(obj)/setup.bin: $(obj)/setup.elf FORCE$(call if\_changed,objcopy)LDFLAGS\_setup.elf := -T
$(obj)/setup.elf: $(src)/setup.ld $(SETUP\_OBJS) FORCE$(call if\_changed,ld)SETUP\_OBJS = $(addprefix $(obj)/,$(setup-y))setup-y += a20.o bioscall.o cmdline.o copy.o cpu.o cpucheck.o edd.o
setup-y += header.o main.o mca.o memory.o pm.o pmjump.o
setup-y += printf.o regs.o string.o tty.o video.o video-mode.o
setup-y += version.o
setup-$(CONFIG\_X86\_APM\_BOOT) += apm.o
setup-y += video-vga.o
setup-y += video-vesa.o
setup-y += video-bios.o
```

### \_\_start

> 执行 `arch/x86/boot/header.S` `\_\_start`代码

>

> 在 `\_start` 函数开始之前，还有很多的代码，一般这些代码就是 kernel 自带的 bootloader (现在已经不再使用,只是会输出一些错误信息)

```YAML
.globl \_start
\_start:
# Explicitly enter this as bytes, or the assembler
# tries to generate a 3-byte jump here, which causes
# everything else to push off to the wrong offset.
.byte 0xeb # short (2-byte) jump
.byte start\_of\_setup-1f
```

- Start 后紧跟着是一个 `jmp` 指令(`0xeb`) 这里相当于是跳转到 `start\_of\_setup`

```Assembly
jmp start\_of\_setup-1f
```

### start\_of\_setup

> 这里算得上 Linux 真正执行的第一个代码

>

> 主要做的事情就是:

>

> 初始化段寄存器、设置堆栈、代码段规范化、验证设置签名、清空BSS段

>

> 然后就是跳转到 main 函数(`arch/x86/boot/main.c`)

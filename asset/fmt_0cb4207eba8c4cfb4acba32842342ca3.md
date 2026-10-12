
# Pinczakko 的 AwardBIOS 逆向工程指导

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-723.htm`](https://www.pediy.com/kssd/pediy09/pediy09-723.htm)

> Pinczakko 的 AwardBIOS 逆向工程指导

> 作者：Pinczakko

> 翻译：beiyu `http://beiyu.bokee.com`

> Email: `beiyuly@gmail.com`

> 时间：2006.6.6

> `ida` 的使用和最后展望没有翻译，希望有兴趣的朋友能够补上。

> 目录
>
> Pinczakko 的 AwardBios 逆向工程指导  1
>
> 1．序言  2
>
> ２．准备工作  2
>
> 2．1．PCI BUS  3
>
> 2．2．ISA BUS  4
>
> 3．一些硬件特性  4
>
> 3.1. BIOS 芯片地址  5
>
> 3.2. 晦涩的硬件接口（Port）  6
>
> 3.3. "可重定位" 硬件 Port  8
>
> 3.4. Expansion ROM Handling  9
>
> 4．一些软件特性  10
>
> 4．1．call 指令特性  10
>
> 4.2. retn Instruction Peculiarity  10
>
> 5. 用到的工具  13
>
> 5.1. 我们的需求  13
>
> 5.2. IDA Pro 技术介绍  13
>
> 5.2.1. IDA Pro 介绍  13
>
> 5.2.2. IDA Pro Scripting And Key Bindings  19
>
> 6. Award BIOS 文件结构  26
>
> 6.1.压缩部分  26
>
> 6.2. 纯二进制部分  27
>
> 6.3. 真实系统(Mainboard)中的内存印象  27
>
> 7. 反汇编 BIOS  28
>
> 7.1. Bootblock  29
>
> 7.1.1. "Virtual Shutdown" routine  29
>
> 7.1.2. Chipset_Reg_Early_Init routine  29
>
> 7.1.3. Init_Interrupt_n_PwrMgmt routine  35
>
> 7.1.4. Call To "Early Silicon Support" Routine  36
>
> 7.1.5. Bootblock Is Copied And Executed In RAM  37
>
> 7.1.6. Call to bios decompression routine and the jump into decompressed system bios  39
>
> 7.1.6.1. Enable FFF80000h-FFFDFFFFh decoding  40
>
> 7.1.6.2. Copy lower 128KB of BIOS code from ROM chip into RAM  40
>
> 7.1.6.3. Disable FFF8_0000h-FFFD_FFFFh decoding  40
>
> 7.1.6.4. Verify checksum of the whole compressed BIOS image  40
>
> 7.1.6.5. Look for the decompression engine  41
>
> 7.1.6.6. Decompress the compressed BIOS components  41
>
> 7.1.6.7. Shadow the BIOS code  60
>
> 7.1.6.8. Enable the microprocessor cache then jump into the decompressed system BIOS  60
>
> 7.2. System BIOS a.k.a Original.tmp  61
>
> 7.2.1. Entry point from "Bootblock in RAM"  61
>
> 7.2.2. The awardext.rom and Extension BIOS Components (lower 128KB bios-code) Relocation Routine  62
>
> 7.2.3. Call to the POST routine a.k.a "POST jump table execution"  64
>
> 7.2.4. The "segment vector" Routines  68
>
> 7.2.5. "chksum_ROM" Procedure  72
>
> 7.2.6. Original.tmp Decompression Routine for The "Extension_BIOS Components"  72
>
> 7.2.7. Microcode Update Routine  90
>
> 8. 激昂展望  92
>
> 9. 结束语  92

### 1．序言

我非常欢迎你能够来实践复杂的 `Award Bios` 的代码研究工作。本文不是一篇官方的 `Award Bios` 逆向工程的文章，也不是由 `Award` 公司内部人员编辑的。我只是一个好奇的普通人，我真的很喜欢搞清楚我的电脑的 `Bios` 是怎样工作的。我写这篇文章的是为了公开我的发现和研究，从而回报那些我所犯的错误，都是我在逆向工程进程当中所犯的。你有几个可能性来读这篇文章，也许你是一个老资格的黑客，也许你是一个像我一样的系统程序设计爱好者，也许你只是一个好奇的外行。只有一点是肯定的，你肯定可以从这篇文章有所收获，可以提高你的技巧。无论如何，我已经写了一个准备章节，来保证你吸收这篇文章所具备的知识。

除非你自己反汇编了 `Bios` 的文件，你是不会理解搞清楚 `BIOS` 的工作的。

这篇文章的目的是消除疑惑，定位好你自己，在开始对 `BIOS` 的逆向工程工作中，为你提供一个参考。

### 2．准备工作

1.  我必须承认，这个工作需要 `x86` 的知识。
2.  保护模式下的编成开发知识。你必须学会怎样让 `x86` 机器从实模式转移到保护模式。也就是说，你必须学会初步的 `x86` 保护模式 `OS` 开发。`www.osdever.net` 是一个很好的学习这方面知识的网站。最重要的事情是保护模式的数据结构是怎样工作的。我的意思是 `GDT`、`IDT`、`x86` 控制寄存器和段寄存器是怎样工作的，特别是 `award bios` 用他们来实现他的奇妙的地方稍后文章解释。
3.  什么是 `x86` 的不真实模式。他是一个 `x86` 机器在真是模式和保护模式之间的状态稍后文章解释。
4.  `X86` 直接硬件编程开发。你需要知道怎样编程直接制硬件，特别是在你主板上面的。你可以联系这个，通过 `windows` 上的直接访问硬件程序开发练习。这个不是必需的，但是如果你懂的话，会给你带来很多方便。你也需要知道一些 `x86` 总线协议，比如 `PCI` 和 `ISA` 稍后文章解释。
5.  你必须理解大部分你的主板芯片的手册。比如北桥和南桥控制寄存器。

#### 2.1．PCI BUS

官方的 `PCI` 总线标准系统是由 `PCISIG` (`PCI Special Interest Group`) 维持的。他可能是某种公司，他介于 `Intel` 和其他大公司，比如 `Microsoft`。他将要被 `Arapahoe` (`PCI-Express a.k.a PCI-e`) and `Hypertransport` 代替。但是 `PCI` 曾经是在保持一种标准。`Hypertransport` 向后兼容 `PCI`。`Arapahoe` 也是一样。只是这个 `PCI` 的标准是没有公开的。

首先，`PCI BUS` 是一个 32 位宽度的总线。通讯需要 32bit 的地址模式。读写操作需要 32 位地址。64 位 `PCI Bus` 不是天生就是，他使用了双重地址回路实现。所以你可以说 `PCI` 就是一个 32 位总线的系统。

其次，这个总线系统定义位置是，控制端口 `PORT CF8h CFBh`，数据端口 `CFCh CFFh`。这些端口用来配置相应的 `PCI` 芯片，比如读写 `PCI` 芯片的配置寄存器值。

第三，这个总线系统强制我们和 `PCI` 通讯需要遵守下面的法则（从用户 `CPU` 观点）：

1.  写目标总线号，设备号，功能号和偏移/寄存器号到配置地质端口，然后使能 bit 置 1。通俗讲就是，写寄存器的地址到你想要写入的 `PCI` 地址端口。
2.  从一个配置数据端口执行一个 one-byte, two-byte, or four-byte I/O 读操作或者写操作。通俗讲就是，读写数据从你想要读写的 `PCI` 端口。

作为一个提示，据我所知，每一个今天用到的 BUS/通讯协议，使用简单的法则来使芯片互相通讯，而这些芯片有一个复杂的总线协议。

有了上面的定义，这里提供一个 `x86` 的汇编码片断，来说明怎样使用这些配置端口。

```
No.  Mnemonic (masm syntax)  Comment
1  Pushad  保存所有通用寄存器的值
2  mov eax,80000064h  把将要访问的 PCI 芯片寄存器的地址放入 eax
(offset 64h device 00:00:00 or hostbridge)
3  mov dx,0CF8h  地址端口放入 dx。因为是 PCI，我们用 CF8h 作为端口，来打开访问这个设备。
4  out dx,eax  发送 PCI 地址端口到 processor 的 I/O 空间
5  mov dx,0CFCh  数据端口放入 dx。因为是 PCI，我们用 CFCh 作为端口，来和这个设备数据通信。
6  in eax,dx  将从这个设备读出的数据放入 eax
7  or eax, 00020202  改变数据(this is only example, don't try this in your machine, it may hang or even destroy your machine)
8  out dx,eax  将数据发送回设备
9  ............  -
10  Popad  出栈所有寄存器值
11  Ret  返回
```

我想上面的代码已经非常清晰了。这里有一个 `PCI` 寄存器地址格式例子：

```
mov eax,80000064h
```

`the 80000064h is the address. The meaning of these bits are:`

`bit position  31  30  29  28  27  26  25  24  23  22  21  20  19  18  17  16  15  14  13  12  11  10  9  8  7  6  5  4  3  2  1  0`
`binary value  1  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  1  1  0  0  1  0  0`
`hexadecimal value  8  0  0  0  0  0  6  4`

`Bit 31` 是一个使能标志。如果这个位置设置了，我们就给与 `PCI bus` 读写通信的权利了，否则就是禁止。那就是为什么我们在最左边有一个 8 的原因。

`Bits 30 - 24` 保留 bits。

`Bits 23 - 16` 是 `PCI Bus` 号。

`Bits 15 - 11` 是 `PCI` 设备号。

`Bits 10 - 8` 是 `PCI` 功能号。

`Bits 7 - 0` 是偏移地址。

`80000064h` 的意思就是我们通讯的设备是 `bus 0, device 0, function 0`, 偏移地址是 `64h`。实际上这个是我们主板上面的北桥芯片中的存储控制配置寄存器。大多数环境下，`bus 0, device 0, function 0` 是 `Hostbridge`，你需要参考自己的芯片数据表来改变这个。大概来讲，他们要作如下工作：读取偏移地址，改写数据，写回设备。

#### 2.2．ISA BUS

`AFAIK`(恕我直言)，`ISA bus` 不是标准的总线。因此，实际上任何 `ISA` 设备可以存在于系统的 16-bit I/O 地址空间。我对 `ISA bus` 的经验很有限（`CMOS chip`，`mainboard's hardware monitoring chip- Winbond W83781D`）。这两个芯片用了上面提到的 `PCI bus` 通用算法：

1.  先送出你想要读写的设备的地址。只有那样，你才可以通过这个设备的数据端口发送接收数据。
2.  通过数据端口，发送接收将要通过设备读写的数据。

我的硬件监视芯片用端口 `295h` 作为地址端口，`296h` 作为数据端口。`CMOS` 用 `70h` 作为地址端口，`71h` 作为数据端口。

### 3．一些硬件特性

`X86` 平台存在很多 hack，特别是他的 `bios`。这个要归功于向下兼容。这章要讨论一对在我 `BIOS` 反汇编中遇到的问题。

#### 3.1. BIOS 芯片地址

最重要的负责 `bios` 代码处理的芯片是南桥和北桥芯片。由于这方面，北桥负责系统地址空间管理，比如 `bios shadowing`，处理访问 `RAM` 和处理事务，用 `bios ROM` 作为南桥的目标，南桥最后积存 `bios rom`。南桥主要负责使能 `rom` 解码控制，这将要寄存要访问的 `bios rom` 的存储地址。下面展示的地址可以存在于系统 `DRAM` 和 `bios rom` 芯片中的任何一个，这取决于在 `bios` 代码执行时，南桥和北桥寄存器的设置。

| Physical Address | Also Known As | Used by | Address Aliasing Note |
| :--- | :--- | :--- | :--- |
| `000F_0000h - 000F_FFFFh` | `F_seg / F_segment` | 1 Mbit, 2 MBit, and 4 MBit BIOS | alias to `FFFF_0000h - FFFF_FFFFh` in all chipset just after power-up |
| `000E_0000h - 000E_FFFFh` | `E_seg / E_segment` | 1 Mbit, 2 MBit, and 4 MBit BIOS | alias to `FFFE_0000h - FFFE_FFFFh` in some chipset just after power-up |

上面的地址范围包含了 `bios` 代码和很多的系统特性。所以你不得不参考你的芯片数据表来理解它。而且，在 `bios` 代码运行后的时间里，注意上面的地址要被 `bios` 代码占据的是 `F_seg` i.e. `F_0000h - F_FFFFh`。无论怎样，相当的操作系统可能会认为这段地址没有用，而且会把它用于自己的目的。上面提到的地址只是当 `bios` 代码访问或者其他代码直接访问 `bios rom` 的时候，反映了 `bios rom` 芯片到系统地址空间映射。就像我们要看到的一样，这个映射可以通过程序设计一些芯片寄存器来改变。

超过 1m 的 `Bios` 芯片，比如 2m 和 4m 的芯片有一个非常与众不同的低 `bios` 区域地址，i.e. `C_seg`, `D_seg` 和其他低 "segment(s)"。大多数情况，这个区域被映射到了靠近 4GB 地址范围。这个地址范围处理是从类似北桥到 `PCI` 地址范围来解决。这个配置下芯片行为如下：

*   北桥作为一个地址传送装置：在不同方式和普通内存地址比较的状态下，它对这个特别的内存地址有反应，内存地址直接指向 `RAM`。相反，这个特别的内存地址由北桥转到南桥，从而解码。
*   南桥作为地址解码器：它解码这个特别的内存地址，这个地址指向正确的芯片，比如 `bios` 芯片。这方面，如果地址范围不被允许在南桥控制寄存器解码，南桥要返回“void”（`bus` 地址周期结束）。

下面是一个例子：

| Physical Address | Also Known As | Used by | Address Aliasing Note |
| :--- | :--- | :--- | :--- |
| `000F_0000h - 000F_FFFFh` | `F_seg / F_segment` | 1 Mbit, 2 MBit, and 4 Mbit BIOS | alias to `FFFF_0000h - FFFF_FFFFh` in all chipset just after power-up |
| `000E_0000h - 000E_FFFFh` | `E_seg / E_segment` | 1 Mbit, 2 Mbit, and 4 Mbit BIOS | alias to `FFFE_0000h - FFFE_FFFFh` in some chipset just after power-up |
| `FFFD_0000h - FFFD_FFFFh` | `D_seg / D_segment` | 2 Mbit, and 4 Mbit BIOS | - |
| `FFFC_0000h - FFFC_FFFFh` | `C_seg / C_segment` | 2 Mbit, and 4 Mbit BIOS | - |
| `FFF8_0000h - FFFB_FFFFh` | - | 4 Mbit BIOS | - |

结论是：现代芯片组表现为效法 `F_seg` and `E_seg` 处理。这是一个证据，证明现代 `x86` 系统保持着向下兼容。无论如何，卖主已经远离 `x86`，这些“杂牌电脑（`cludge`）”往往被认为是过去的东西。

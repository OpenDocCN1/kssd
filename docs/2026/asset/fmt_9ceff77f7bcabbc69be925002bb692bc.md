
# ARM指令介绍

## DATA Types

与高级语言相似，ARM支持操作不同类型的数据类型。我们可以load（or stroe）的数据类型有signed and unsigned words、halfwords、or bytes. 这些数据类型的extensions分别是：halfwords是-h or -sh，bytes是-b or -sb，word没有extension. signed and unsinged 数据的区别：

1. signed data type 能储存正数和负数
2. unsigned data type 可以保存大的正值（包括“零”），但不能保存负值。

以下是一些load和store指令的操作指令的一些例子：

```c
ldr = Load Word
ldrh = Load unsigned Half Word
ldrsh = Load signed Half Word
ldrb = Load unsigned Byte
ldrsb = Load signed Bytes
str = Store Word
strh = Store unsigned Half Word
strsh = Store signed Half Word
strb = Store unsigned Byte
strsb = Store signed Byte
```

查看内存中的字节有两种基本方式：Little-Endian (LE) 或 Big-Endian (BE)。 区别在于对象的每个字节存储在内存中的字节顺序。 在像 Intel x86 这样的小端机器上，最低有效字节存储在最低地址（最接近零的地址）。 在 big-endian 机器上，最高有效字节存储在最低地址。 ARM 体系结构在第 3 版之前是小端的，从那时起它是双端的，这意味着它具有允许可切换的端的设置。 例如，在 ARMv6 上，指令是固定的 little-endian，数据访问可以是 little-endian 或 big-endian，由程序状态寄存器（`CPSR`）的第 9 位（`E` 位）控制。

寄存器的数量是依赖于ARM的版本的，通过查询ARM Reference Manual，除了基于 ARMv6-M 和 ARMv7-M 的处理器外，都有 30 个通用 32 位寄存器，前 16 个寄存器（`r1-r15`）可在用户级模式下访问，其他寄存器可在特权软件执行中使用（ARMv6-M 和 ARMv7-M 除外）。

| # | Alias | Purpose |
| --- | --- | --- |
| `R0` | - | General purppose |
| `R1` | - | General purppose |
| `R2` | - | General purppose |
| `R3` | - | General purppose |
| `R4` | - | General purppose |
| `R5` | - | General purppose |
| `R6` | - | General purppose |
| `R7` | - | General purppose |
| `R8` | - | General purppose |
| `R9` | - | General purppose |
| `R10` | - | General purppose |
| `R11` | `FP` | Frame Pointer |
| `R12` | `IP` | Instra Procedural Call |
| `R13` | `SP` | Stack Pointer |
| `R14` | `LR` | Link Register |
| `R15` | `PC` | Program Counter |
| `CPSR` | - | Current Program Status Register |

下表只是快速了解 ARM 寄存器如何与 Intel 处理器中的寄存器相关联。

| ARM | Description | x86 |
| --- | --- | --- |
| `R0` | General Purpose | `EAX` |
| `R1-R5` | General Purpose | `EBX`, `ECX`, `EDX`, `ESI`, `EDI` |
| `R6-R10` | General Purpose | - |
| `R11` (`FP`) | Frame Pointer | `EBP` |
| `R12` | Intra Procedural Call | - |
| `R13` (`SP`) | Stack Pointer | `ESP` |
| `R14` (`LR`) | Link Register | - |
| `R15` (`PC`) | Program Counter / Instruction Pointer | `EIP` |
| `CPSR` | Current Program State Register/Flags | `EFLAGS` |

`R0-R12`：在普通的操作中可以被用作储存临时数据、指针等等。例如，`R0` 可在算术运算期间称为累加器或用于存储先前调用函数的结果。 `R7` 在处理系统调用时变得有用，因为它存储系统调用编号，而 `R11` 帮助我们跟踪用作帧指针的堆栈边界。 此外，ARM 上的函数调用约定指定函数的前四个参数存储在寄存器 `r0-r3` 中。

`R13`：`SP`(Stack Pointer). 这个栈指针指向栈顶，栈是用于特定于函数的存储的内存区域，在函数返回时被回收。 因此，栈指针用于在栈上分配空间，方法是从栈指针中减去我们想要分配的值（以字节为单位）。 换句话说，如果我们想分配一个 32 位的值，我们从栈指针中减去 4。

`R14`：`LR`(Link Register). 当一个函数调用被执行了，链接寄存器就会更新为下一条指令的内存地址。止痒允许程序在子函数完成后返回到父函数中接着执行。

`R15`：`PC`(Program Counter). 程序计数器根据所执行指令的大小自动递增。 此大小在 ARM 状态下始终为 4 个字节，在 THUMB 模式下始终为 2 个字节。 当执行分支指令时，`PC` 持有目的地址。 在执行过程中，`PC` 在 ARM 状态下存储当前指令加 8（两条 ARM 指令）的地址，在 Thumb(v1) 状态下存储当前指令加 4（两条 Thumb 指令）的地址。 这与 x86 不同，在 x86 中 `PC` 总是指向要执行的下一条指令。

### 下面在调试器下看一下PC寄存器的行为

源程序如下：

```arm
.section .text
.global \_start
\_start:
mov r0, pc
mov r1, #2
add r2, r1, r1
bkpt
```

在GDB中我们设置断点`\_start`然后运行

```s
gef> b \_start
Breakpoint 1 at 0x10054
gef> r
```

断点的现场状态如下：

```s
Breakpoint 1, 0x00010054 in \_start ()
---------------------------------------------------------------[ registers ]----
$r0 : 0x00000000
$r1 : 0x00000000
$r2 : 0x00000000
$r3 : 0x00000000
$r4 : 0x00000000
$r5 : 0x00000000
$r6 : 0x00000000
$r7 : 0x00000000
$r8 : 0x00000000
$r9 : 0x00000000
$r10 : 0x00000000
$r11 : 0x00000000
$r12 : 0x00000000
$sp : 0xbefff3b0 -> 0x00000001
$lr : 0x00000000
$pc : 0x00010054 -> <\_start+0> mov r0, pc
$cpsr : [thumb fast interrupt overflow carry zero negative]
-------------------------------------------------------------------[ stack ]----
0xbefff3b0|+0x00: 0x00000001 <-$sp
0xbefff3b4|+0x04: 0xbefff51c -> "/home/pi/asm/test\_pc"
0xbefff3b8|+0x08: 0x00000000
0xbefff3bc|+0x0c: 0xbefff531 -> 0x49464e49
0xbefff3c0|+0x10: 0xbefff56b -> "XDG\_SESSION\_ID=c2"
0xbefff3c4|+0x14: 0xbefff57d -> "SHELL=/bin/bash"
0xbefff3c8|+0x18: 0xbefff58d -> "TERM=xterm"
0xbefff3cc|+0x1c: 0xbefff598 -> 0x49464e49
-------------------------------------------------------------[ code:armv5t ]----
0x1003c andeq r0, r1, r0
0x10040 andeq r0, r1, r0
0x10044 andeq r0, r0, r4, rrx
0x10048 andeq r0, r0, r4, rrx
0x1004c andeq r0, r0, r5
0x10050 andeq r0, r1, r0
-> 0x10054 <\_start+0> mov r0, pc
0x10058 <\_start+4> mov r1, #2
0x1005c <\_start+8> add r2, r1, r1
0x10060 <\_start+12> bkpt 0x0000
0x10064 andeq r1, r0, r1, asr #6
0x10068 cmnvs r5, r0, lsl #2
-----------------------------------------------------------------[ threads ]----
[#0] Id 1, Name: "test\_pc", stopped, reason: BREAKPOINT
-------------------------------------------------------------------[ trace ]----
[#0] 0x10054->Name: \_start()
--------------------------------------------------------------------------------
```

单步运行后：

```s
0x00010058 in \_start ()
--------------------------------------------------------------------------------------------------------------------------------------------------------------------[ registers ]----
$r0 : 0x0001005c -> <\_start+8> add r2, r1, r1
$r1 : 0x00000000
$r2 : 0x00000000
$r3 : 0x00000000
$r4 : 0x00000000
$r5 : 0x00000000
$r6 : 0x00000000
$r7 : 0x00000000
$r8 : 0x00000000
$r9 : 0x00000000
$r10 : 0x00000000
$r11 : 0x00000000
$r12 : 0x00000000
$sp : 0xbefff3b0 -> 0x00000001
$lr : 0x00000000
$pc : 0x00010058 -> <\_start+4> mov r1, #2
$cpsr : [thumb fast interrupt overflow carry zero negative]
------------------------------------------------------------------------------------------------------------------------------------------------------------------------[ stack ]----
0xbefff3b0|+0x00: 0x00000001 <-$sp
0xbefff3b4|+0x04: 0xbefff51c -> "/home/pi/asm/test\_pc"
0xbefff3b8|+0x08: 0x00000000
0xbefff3bc|+0x0c: 0xbefff531 -> 0x49464e49
0xbefff3c0|+0x10: 0xbefff56b -> "XDG\_SESSION\_ID=c2"
0xbefff3c4|+0x14: 0xbefff57d -> "SHELL=/bin/bash"
0xbefff3c8|+0x18: 0xbefff58d -> "TERM=xterm"
0xbefff3cc|+0x1c: 0xbefff598 -> 0x49464e49
------------------------------------------------------------------------------------------------------------------------------------------------------------------[ code:armv5t ]----
0x10040 andeq r0, r1, r0
0x10044 andeq r0, r0, r4, rrx
0x10048 andeq r0, r0, r4, rrx
0x1004c andeq r0, r0, r5
0x10050 andeq r0, r1, r0
0x10054 <\_start+0> mov r0, pc
-> 0x10058 <\_start+4> mov r1, #2
0x1005c <\_start+8> add r2, r1, r1
0x10060 <\_start+12> bkpt 0x0000
0x10064 andeq r1, r0, r1, asr #6
0x10068 cmnvs r5, r0, lsl #2
0x1006c tsteq r0, r2, ror #18
----------------------------------------------------------------------------------------------------------------------------------------------------------------------[ threads ]----
[#0] Id 1, Name: "test\_pc", stopped, reason: SINGLE STEP
------------------------------------------------------------------------------------------------------------------------------------------------------------------------[ trace ]----
[#0] 0x10058->Name: \_start()
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
```

奇妙的事情发生了，查看`R0`中的地址。 虽然我们希望 `R0` 包含先前读取的 `PC` 值 (0x10054)，但它保存的是我们先前读取的 `PC` (0x1005c) 前两条指令的值。 从这个例子可以看出，当我们直接读取`PC`时，它遵循`PC`指向下一条指令的定义； 但在调试时，`PC` 指向当前 `PC` 值（0x10054 + 8 = 0x1005C）之前的两条指令。 这是因为较旧的 ARM 处理器总是在当前执行的指令之前获取两条指令。 ARM 保留此定义的原因是为了确保与早期处理器的兼容性。

### 状态寄存器

寄存器 `$CPSR` 显示当前程序状态寄存器（`CPSR`）的值，在其下方您可以看到标志拇指、快速中断、溢出、进位、零和负数。 这些标志代表 `CPSR` 寄存器中的某些位，根据 `CPSR` 的值设置，激活时变为1。 `N`、`Z`、`C` 和 `V` 位与 x86 上 `EFLAG` 寄存器中的 `SF`、`ZF`、`CF` 和 `OF` 位相同。 这些位用于在程序集级别支持条件和循环中的条件执行

| N | Z | C | V | Q | - | J | - | GE | - | E | A | I | F | T | M |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Negative | Zero | Carry | Overflow | underflow |  | Jazelle |  | Greater than or Equal for SIMD |  | Endianness | Abort disable | IRQ disable | FIQ disable | Thumb | Processor mode (Privilege mode) |

| Flag | Description |
| --- | --- |
| `N`(Negative) | 如果指令结果是负数则置1. |
| `Z`(Zero) | 如果指令结果产生零值则置1. |
| `C`(Carry) | 如果指令的结果产生一个需要第 33 位才能完全表示的值，则置1. |
| `V`(Overflow) | 如果该指令的结果产生不能在32位二进制补码表示的值. |
| `E`(Endian-bit) | ARM 可以以小端或大端方式运行。 对于小端模式，该位设置为 0，对于大端模式设置为 1. |
| `T`(Thumb-bit) | 如果是Thumb模式置1，如果是ARM模式置0. |
| `M`(Mode-bits) | 这些位指定当前特权模式（USR、SVC 等）. |
| `J`(Jazelle) | 允许某些 ARM 处理器在硬件中执行 Java 字节码的第三种执行状态. |


# ARM指令介绍

## 多寄存器加载/存储指令

```arm
ldmib r0, {r4-r6} /\* words[4] -> r4 = 0x04; words[5] -> r5 = 0x05; words[6] -> r6 = 0x06 \*/
stmib r1, {r4-r6} /\* r4 -> array\_buff[1] = 0x04; r5 -> array\_buff[2] = 0x05; r6 -> array\_buff[3] = 0x06 \*/
```

执行上述两条指令后，寄存器`R4`-`R6`和内存地址`0x100D4`、`0x100D8`和`0x100DC`包含值 0x4、0x5 和 0x6。

当我们使用`LDMDA`指令时，一切都开始向后运行。`R0`指向`word[3]`。当加载开始时，我们向后移动并将`word[3]`、`word[2]`和`word[1]`加载到`R6`、`R5`、`R4`中。是的，寄存器也是反向加载的。所以在指令完成后`R6 = 0x00000003`，`R5 = 0x00000002`，`R4 = 0x00000001`。

```arm
ldmda r0, {r4-r6} /\* words[3] -> r6 = 0x03; words[2] -> r5 = 0x02; words[1] -> r4 = 0x01 \*/
```

加载多个，递减后：

```arm
ldmdb r0, {r4-r6} /\* words[2] -> r6 = 0x02; words[1] -> r5 = 0x01; words[0] -> r4 = 0x00 \*/
```

store多个，递减后：

```arm
stmda r2, {r4-r6} /\* r6 -> array\_buff[2] = 0x02; r5 -> array\_buff[1] = 0x01; r4 -> array\_buff[0] = 0x00 \*/
```

### Push and Pop

`PUSH`：

1. `SP -= 4`

2. 信息store到新地址

`POP`：

1. `SP`地址处的value被load到指示的register中

2. `SP += 4`

## 条件执行

| Condition Code | Meaning (for `cmp` or `subs`) | Status of Flags |
| --- | --- | --- |
| `EQ` | Equal | `Z==1` |
| `NE` | Not Equal | `Z==0` |
| `GT` | Signed Greater Than | `(Z==0) && (N==V)` |
| `LT` | Signed Less Than | `N!=V` |
| `GE` | Signed Greater Than or Equal | `N==V` |
| `LE` | Signed Less Than or Equal | `(Z==1) \|\| (N!=V)` |
| `CS` or `HS` | Unsigned Higher or Same (or Carry Set) | `C==1` |
| `CC` or `LO` | Unsigned Lower (or Carry Clear) | `C==0` |
| `MI` | Negative (or Minus) | `N==1` |
| `PL` | Positive (or Plus) | `N==0` |
| `AL` | Always executed | – |
| `NV` | Never executed | – |
| `VS` | Signed Overflow | `V==1` |
| `VC` | No signed Overflow | `V==0` |
| `HI` | Unsigned Higher | `(C==1) && (Z==0)` |
| `LS` | Unsigned Lower or same | `(C==0) \|\| (Z==0)` |

```arm
.global main
main:
mov r0, #2 /\* setting up initial variable \*/
cmp r0, #3 /\* comparing r0 to number 3. Negative bit get's set to 1 \*/
addlt r0, r0, #1 /\* increasing r0 IF it was determined that it is smaller (lower than) number 3 \*/
cmp r0, #3 /\* comparing r0 to number 3 again. Zero bit gets set to 1. Negative bit is set to 0 \*/
addlt r0, r0, #1 /\* increasing r0 IF it was determined that it is smaller (lower than) number 3 \*/
bx lr
```

上面代码中的第一条`CMP`指令触发负位被置位（`2 – 3 = -1`）表示`r0`中的值低于数字 3。随后执行`ADDLT`指令，因为当`V != N`（`CPSR`中溢出位和负位的值不同）时，`LT`条件已满足。在执行第二个`CMP`之前，我们的`r0 = 3`。这就是为什么第二个`CMP`清除负位（因为`3 – 3 = 0`，不需要设置负标志）并设置零标志（`Z = 1`）。现在我们有`V = 0`和`N = 0`，这导致`LT`条件失败。因此，不会执行第二个`ADDLT`，并且`r0`保持不变。程序退出，结果为 3。

### Thumb 条件执行

在允许条件执行的 Thumb 版本（`Thumb-2`）中，某些 ARM 处理器版本支持`IT`指令，该指令允许在 Thumb 状态下有条件地执行最多 4 条指令。

语法：`IT{x{y{z}}} cond`

1. `cond` 指定 `IT` 块中第一条指令的条件

2. `x` 指定 `IT` 块中第二条指令的条件开关

3. `y` 指定 `IT` 块中第三条指令的条件开关

4. `z` 指定 `IT` 块中第四条指令的条件开关

`IT` 指令的结构是“`IF-Then-(Else)`”，语法是两个字母 `T` 和 `E` 的结构：

1. `IT` 指的是 `If-Then`（下一条指令是有条件的）

2. `ITT` 指的是 `If-Then-Then`（接下来的 2 条指令是有条件的）

3. `ITE` 指的是 `If-Then-Else`（接下来的 2 条指令是有条件的）

4. `ITTE` 指的是 `If-Then-Then-Else`（接下来的 3 条指令是有条件的）

5. `ITTEE` 指的是 `If-Then-Then-Else-Else`（接下来的 4 条指令是有条件的）

`IT` 块内的每条指令都必须指定一个条件后缀，该后缀相同或逻辑相反。这意味着，如果您使用`ITE`，则第一条和第二条指令（`If-Then`）必须具有相同的条件后缀，而第三条（`Else`）必须具有前两条的逻辑逆。

```arm
ITTE NE ; Next 3 instructions are conditional
ANDNE R0, R0, R1 ; ANDNE does not update condition flags
ADDSNE R2, R2, #1 ; ADDSNE updates condition flags
MOVEQ R2, R3 ; Conditional move
ITE GT ; Next 2 instructions are conditional
ADDGT R1, R0, #55 ; Conditional addition in case the GT is true
ADDLE R1, R0, #48 ; Conditional addition in case the GT is not true
ITTEE EQ ; Next 4 instructions are conditional
MOVEQ R0, R1 ; Conditional MOV
ADDEQ R2, R2, #10 ; Conditional ADD
ANDNE R3, R3, #1 ; Conditional AND
BNE.W dloop ; Branch instruction can only be used in the last instruction of an IT block
```

以下是条件代码及其对立的条件：

| Code | Meaning | Code | Meaning |
| --- | --- | --- | --- |
| `EQ` | Equal | `NE` | Not Equal |
| `HS` (or `CS`) | Unsigned higher or same (or carry set) | `LO` (or `CC`) | Unsigned lower (or carry clear) |
| `MI` | Negative | `PL` | Positive or Zero |
| `VS` | Signed Overflow | `VC` | No Signed Overflow |
| `HI` | Unsigned Higher | `LS` | Unsigned Lower or Same |
| `GE` | Signed Greater Than or Equal | `LT` | Signed Less Than |
| `GT` | Signed Greater Than | `LE` | Signed Less Than or Equal |
| `AL` (or omitted) | Always Executed | There is no opposite to `AL` | – |

```arm
.syntax unified @ this is important!
.text
.global \_start
\_start:
.code 32
add r3, pc, #1 @ increase value of PC by 1 and add it to R3
bx r3 @ branch + exchange to the address in R3 -> switch to Thumb state because LSB = 1
.code 16 @ Thumb state
cmp r0, #10
ite eq @ if R0 is equal 10...
addeq r1, #2 @ ... then R1 = R1 + 2
addne r1, #3 @ ... else R1 = R1 + 3
bkpt
```

`.code 32`

此示例代码以 ARM 状态开始。第一条指令将 `PC` 加 1 指定的地址添加到 `R3`，然后分支到 `R3` 中的地址。这将导致切换到 Thumb 状态，因为 `LSB`（最不重要的位）是 1，因此不是 4 字节对齐的。为此使用`bx`（分支+交换）很重要。在分支之后设置 `T`（Thumb）标志，我们处于 Thumb 状态。

`.code 16`

在 Thumb 状态下，我们首先将`R0`与`#10`进行比较，这将设置负标志（`0–10 = –10`）。然后我们使用`If-Then-Else`块。此块将跳过`ADDEQ`指令，因为没有设置 `Z`（零）标志，并将执行`ADDNE`指令，因为结果为 `NE`（不等于）为 10。

## 分支

### B / BX / BLX

存在三种类型的分支指令：

`B`（Branch）

简单跳转到函数

`BL`（Branch link）

在`LR`中保存`PC+4`并跳转到函数

`BX`（Branch exchange）和`BLX`（Branch link exchange）

与`B`/`BL` + 交换指令集相同（`ARM` <-> `Thumb`）

需要一个寄存器作为第一个操作数：`BX/BLX reg`

## 栈

| Stack Type | Store | Load |
| --- | --- | --- |
| Full descending | `STMFD` (`STMDB`, Decrement Before) | `LDMFD` (`LDM`, Increment After) |
| Full ascending | `STMFA` (`STMIB`, Increment Before) | `LDMFA` (`LDMDA`, Decrement After) |
| Empty descending | `STMED` (`STMDA`, Decrement After) | `LDMED` (`LDMIB`, Increment Before) |
| Empty ascending | `STMEA` (`STM`, Increment After) | `LDMEA` (`LDMDB`, Decrement Before) |

◎ Full descending 满递减堆栈

堆栈首部是高地址，堆栈向低地址增长。栈指针总是指向堆栈最后一个元素（最后一个元素是最后压入的数据）。

ARM-Thumb 过程调用标准和 ARM、Thumb C/C++ 编译器总是使用 Full descending 类型堆栈。

◎ Full ascending 满递增堆栈

堆栈首部是低地址，堆栈向高地址增长。栈指针总是指向堆栈最后一个元素（最后一个元素是最后压入的数据）。

◎ Empty descending 空递减堆栈

堆栈首部是低地址，堆栈向高地址增长。栈指针总是指向下一个将要放入数据的空位置。

◎ Empty ascending 空递增堆栈

堆栈首部是高地址，堆栈向低地址增长。栈指针总是指向下一个将要放入数据的空位置。

## DATA Types

与高级语言相似，ARM 支持操作不同类型的数据类型。我们可以 load（or store）的数据类型有 signed and unsigned words、halfwords、or bytes。这些数据类型的 extensions 分别是：halfwords 是 `-h` or `-sh`，bytes 是 `-b` or `-sb`，word 没有 extension。signed and unsigned 数据的区别：

1. signed data type 能储存正数和负数

2. unsigned data type 可以保存大的正值（包括“零”），但不能保存负值。

以下是一些 load 和 store 指令的操作指令的一些例子：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11 | `ldr` = `Load Word`<br>`ldrh` = `Load unsigned Half Word`<br>`ldrsh` = `Load signed Half Word`<br>`ldrb` = `Load unsigned Byte`<br>`ldrsb` = `Load signed Bytes`<br>`str` = `Store Word`<br>`strh` = `Store unsigned Half Word`<br>`strsh` = `Store signed Half Word`<br>`strb` = `Store unsigned Byte`<br>`strsb` = `Store signed Byte` |

查看内存中的字节有两种基本方式：Little-Endian (LE) 或 Big-Endian (BE)。区别在于对象的每个字节存储在内存中的字节顺序。在像 Intel x86 这样的小端机器上，最低有效字节存储在最低地址（最接近零的地址）。在 big-endian 机器上，最高有效字节存储在最低地址。ARM 体系结构在第 3 版之前是小端的，从那时起它是双端的，这意味着它具有允许可切换的端设置。例如，在 `ARMv6` 上，指令是固定的 little-endian，数据访问可以是 little-endian 或 big-endian，由程序状态寄存器 (`CPSR`) 的第 9 位（`E` 位）控制。

寄存器的数量是依赖于 ARM 版本的，通过查询 ARM Reference Manual，除了基于 `ARMv6-M` 和 `ARMv7-M` 的处理器外，都有 30 个通用 32 位寄存器，前 16 个寄存器（`r1-r15`）可在用户级模式下访问，其他寄存器可在特权软件执行中使用（`ARMv6-M` 和 `ARMv7-M` 除外）。

| # | Alias | Purpose |
| --- | --- | --- |
| `R0` | - | General purpose |
| `R1` | - | General purpose |
| `R2` | - | General purpose |
| `R3` | - | General purpose |
| `R4` | - | General purpose |
| `R5` | - | General purpose |
| `R6` | - | General purpose |
| `R7` | - | General purpose |
| `R8` | - | General purpose |
| `R9` | - | General purpose |
| `R10` | - | General purpose |
| `R11` | `FP` | Frame Pointer |
| `R12` | `IP` | Intra Procedural Call |
| `R13` | `SP` | Stack Pointer |
| `R14` | `LR` | Link Register |
| `R15` | `PC` | Program Counter |
| `CPSR` | - | Current Program Status Register |

下表只是快速了解 ARM 寄存器如何与 Intel 处理器中的寄存器相关联。

| ARM | Description | x86 |
| --- | --- | --- |
| `R0` | General Purpose | `EAX` |
| `R1`-`R5` | General Purpose | `EBX`, `ECX`, `EDX`, `ESI`, `EDI` |
| `R6`-`R10` | General Purpose | – |
| `R11` (`FP`) | Frame Pointer | `EBP` |
| `R12` | Intra Procedural Call | – |
| `R13` (`SP`) | Stack Pointer | `ESP` |
| `R14` (`LR`) | Link Register | – |
| `R15` (`PC`) | Program Counter / Instruction Pointer | `EIP` |
| `CPSR` | Current Program State Register/Flags | `EFLAGS` |

`R0`-`R12`：在普通的操作中可以被用作储存临时数据、指针等等。例如，`R0` 可在算术运算期间称为累加器或用于存储先前调用函数的结果。`R7` 在处理系统调用时变得有用，因为它存储系统调用编号，而 `R11` 帮助我们跟踪用作帧指针的堆栈边界。此外，ARM 上的函数调用约定指定函数的前四个参数存储在寄存器 `r0-r3` 中。

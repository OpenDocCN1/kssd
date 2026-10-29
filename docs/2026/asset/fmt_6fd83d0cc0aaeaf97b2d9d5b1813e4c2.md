
```asm
.data
array_buff:
    .word 0x00000000             /* array_buff[0] */
    .word 0x00000000             /* array_buff[1] */
    .word 0x00000000             /* array_buff[2]. This element has a relative address of array_buff+8 */
    .word 0x00000000             /* array_buff[3] */
    .word 0x00000000             /* array_buff[4] */

.text
.global _start

_start:
    adr r0, words+12             /* address of words[3] -> r0 */
    ldr r1, array_buff_bridge    /* address of array_buff[0] -> r1 */
    ldr r2, array_buff_bridge+4  /* address of array_buff[2] -> r2 */
    ldm r0, {r4,r5}              /* words[3] -> r4 = 0x03; words[4] -> r5 = 0x04 */
    stm r1, {r4,r5}              /* r4 -> array_buff[0] = 0x03; r5 -> array_buff[1] = 0x04 */
    ldmia r0, {r4-r6}            /* words[3] -> r4 = 0x03, words[4] -> r5 = 0x04; words[5] -> r6 = 0x05; */
    stmia r1, {r4-r6}            /* r4 -> array_buff[0] = 0x03; r5 -> array_buff[1] = 0x04; r6 -> array_buff[2] = 0x05 */
    ldmib r0, {r4-r6}            /* words[4] -> r4 = 0x04; words[5] -> r5 = 0x05; words[6] -> r6 = 0x06 */
    stmib r1, {r4-r6}            /* r4 -> array_buff[1] = 0x04; r5 -> array_buff[2] = 0x05; r6 -> array_buff[3] = 0x06 */
    ldmda r0, {r4-r6}            /* words[3] -> r6 = 0x03; words[2] -> r5 = 0x02; words[1] -> r4 = 0x01 */
    ldmdb r0, {r4-r6}            /* words[2] -> r6 = 0x02; words[1] -> r5 = 0x01; words[0] -> r4 = 0x00 */
    stmda r2, {r4-r6}            /* r6 -> array_buff[2] = 0x02; r5 -> array_buff[1] = 0x01; r4 -> array_buff[0] = 0x00 */
    stmdb r2, {r4-r5}            /* r5 -> array_buff[1] = 0x01; r4 -> array_buff[0] = 0x00; */
    bx lr

words:
    .word 0x00000000             /* words[0] */
    .word 0x00000001             /* words[1] */
    .word 0x00000002             /* words[2] */
    .word 0x00000003             /* words[3] */
    .word 0x00000004             /* words[4] */
    .word 0x00000005             /* words[5] */
    .word 0x00000006             /* words[6] */

array_buff_bridge:
    .word array_buff             /* address of array_buff, or in other words - array_buff[0] */
    .word array_buff+8           /* address of array_buff[2] */
```

在开始之前，请记住，`.word` 指的是 32 位 = 4 BYTES 的数据（内存）块。这对于理解偏移很重要。因此该程序由 `.data` 部分组成，我们在其中分配了一个具有 5 个元素的空数组（`array_buff`）。我们将使用它作为存储数据的可写内存位置。`.text` 部分包含我们的代码以及内存操作指令和一个包含两个标签的只读数据池：一个用于具有 7 个元素的数组，另一个用于“桥接” `.text` 和 `.data` 部分，以便我们可以访问驻留在 `.data` 部分的 `array_buff`。

```asm
adr r0, words+12             /* address of words[3] -> r0 */
```

我们使用 `ADR` 指令将第 4 个元素（`words[3]`）的地址放入 `R0`。我们指向 `words` 数组的中间，因为我们将从那里向前和向后操作。

我们用 `array_buff` 数组的第一个（`array_buff[0]`）和第三个（`array_buff[2]`）元素的地址准备 `R1` 和 `R2`。一旦获得地址，我们就可以开始对其进行操作。

下一条指令使用 `LDM` 从 `R0` 指向的内存中加载两个字值。因此，因为我们之前让 `R0` 指向 `words[3]` 元素，所以 `words[3]` 值转到 `R4`，`words[4]` 值转到 `R5`。

下一条指令让我们执行 `STM` 指令将多个值存储到内存中。我们代码中的 `STM` 指令从寄存器 `R4` 和 `R5` 获取值（`0x3` 和 `0x4`），并将这些值存储到 `R1` 指定的内存位置。

变体的类型由指令的后缀定义。示例中使用的后缀是：`-IA`（之后增加）、`-IB`（之前增加）、`-DA`（之后减少）、`-DB`（之前减少）。这些变体的不同之处在于它们如何访问由第一个操作数（存储源地址或目标地址的寄存器）指定的内存。实际上，`LDM` 与 `LDMIA` 相同，这意味着每次加载后都会增加下一个要加载的元素的地址。通过这种方式，我们从第一个操作数（存储源地址的寄存器）指定的内存地址中获得顺序（前向）数据加载。

```asm
ldmia r0, {r4-r6}            /* words[3] -> r4 = 0x03, words[4] -> r5 = 0x04; words[5] -> r6 = 0x05; */
stmia r1, {r4-r6}            /* r4 -> array_buff[0] = 0x03; r5 -> array_buff[1] = 0x04; r6 -> array_buff[2] = 0x05 */
```

执行上述两条指令后，寄存器 `R4-R6` 和内存地址 `0x000100D0`、`0x000100D4` 和 `0x000100D8` 包含值 `0x3`、`0x4` 和 `0x5`。

`LDMIB` 指令首先将源地址增加 4 个字节（一个字值），然后执行第一次加载。通过这种方式，我们仍然可以顺序（向前）加载数据，但第一个元素与源地址有 4 个字节的偏移量。这就是为什么在我们的示例中，通过 `LDMIB` 指令从内存加载到 `R4` 的第一个元素是 `0x00000004`（`word[4]`）而不是 `R0` 指向的 `0x00000003`（`word[3]`）。

```asm
ldmib r0, {r4-r6}            /* words[4] -> r4 = 0x04; words[5] -> r5 = 0x05; words[6] -> r6 = 0x06 */
stmib r1, {r4-r6}            /* r4 -> array_buff[1] = 0x04; r5 -> array_buff[2] = 0x05; r6 -> array_buff[3] = 0x06 */
```

执行上述两条指令后，寄存器 `R4-R6` 和内存地址 `0x100D4`、`0x100D8` 和 `0x100DC` 包含值 `0x4`、`0x5` 和 `0x6`。

当我们使用 `LDMDA` 指令时，一切都开始向后运行。`R0` 指向 `word[3]`。当加载开始时，我们向后移动并将 `word[3]`、`word[2]` 和 `word[1]` 加载到 `R6`、`R5`、`R4` 中。是的，寄存器也是反向加载的。所以在指令完成后 `R6 = 0x00000003`，`R5 = 0x00000002`，`R4 = 0x00000001`。

```asm
ldmda r0, {r4-r6}            /* words[3] -> r6 = 0x03; words[2] -> r5 = 0x02; words[1] -> r4 = 0x01 */
```

加载多个，递减后：

```asm
ldmdb r0, {r4-r6}            /* words[2] -> r6 = 0x02; words[1] -> r5 = 0x01; words[0] -> r4 = 0x00 */
```

store多个，递减后：

```asm
stmda r2, {r4-r6}            /* r6 -> array_buff[2] = 0x02; r5 -> array_buff[1] = 0x01; r4 -> array_buff[0] = 0x00 */
```

### `push` and `pop`

`PUSH`：

1. `SP` -= 4
2. 信息 store 到新地址

`POP`：

1. `SP` 地址处的 value 被 load 到指示的 register 中
2. `SP` += 4

## 条件执行

| Condition Code | Meaning (for cmp or subs) | Status of Flags |
| --- | --- | --- |
| `EQ` | Equal | `Z==1` |
| `NE` | Not Equal | `Z==0` |
| `GT` | Signed Greater Than | `(Z==0) && (N==V)` |
| `LT` | Signed Less Than | `N!=V` |
| `GE` | Signed Greater Than or Equal | `N==V` |
| `LE` | Signed Less Than or Equal | `(Z==1)` \|\| `(N!=V)` |
| `CS` or `HS` | Unsigned Higher or Same (or Carry Set) | `C==1` |
| `CC` or `LO` | Unsigned Lower (or Carry Clear) | `C==0` |
| `MI` | Negative (or Minus) | `N==1` |
| `PL` | Positive (or Plus) | `N==0` |
| `AL` | Always executed | – |
| `NV` | Never executed | – |
| `VS` | Signed Overflow | `V==1` |
| `VC` | No signed Overflow | `V==0` |
| `HI` | Unsigned Higher | `(C==1) && (Z==0)` |
| `LS` | Unsigned Lower or same | `(C==0)` \|\| `(Z==0)` |

```asm
.global main

main:
    mov     r0, #2     /* setting up initial variable */
    cmp     r0, #3     /* comparing r0 to number 3. Negative bit get's set to 1 */
    addlt   r0, r0, #1 /* increasing r0 IF it was determined that it is smaller (lower than) number 3 */
    cmp     r0, #3     /* comparing r0 to number 3 again. Zero bit gets set to 1. Negative bit is set to 0 */
    addlt   r0, r0, #1 /* increasing r0 IF it was determined that it is smaller (lower than) number 3 */
    bx      lr
```

上面代码中的第一条 `CMP` 指令触发负位被置位（`2 - 3 = -1`）表示 `r0` 中的值低于数字 3。随后执行 `ADDLT` 指令，因为当 `V != N` 时 `LT` 条件已满足（`CPSR` 中溢出位和负位的值不同）。在我们执行第二个 `CMP` 之前，我们的 `r0 = 3`。这就是为什么第二个 `CMP` 清除负位（因为 `3 - 3 = 0`，不需要设置负标志）并设置零标志（`Z = 1`）。现在我们有 `V = 0` 和 `N = 0`，这导致 `LT` 条件失败。因此，不会执行第二个 `ADDLT`，并且 `r0` 保持不变。程序退出，结果为 3。

### Thumb条件执行

在允许条件执行的 Thumb 版本（Thumb-2）中。某些 ARM 处理器版本支持“IT”指令，该指令允许在 Thumb 状态下有条件地执行最多 4 条指令。

语法：`IT{x{y{z}}} cond`

1. `cond` 指定 `IT` 块中第一条指令的条件
2. `x` 指定 `IT` 块中第二条指令的条件开关
3. `y` 指定 `IT` 块中第三条指令的条件开关
4. `z` 指定 `IT` 块中第四条指令的条件开关

`IT` 指令的结构是“IF-Then-(Else)”，语法是两个字母 `T` 和 `E` 的结构：

1. `IT` 指的是 If-Then（下一条指令是有条件的）
2. `ITT` 指的是 If-Then-Then（接下来的 2 条指令是有条件的）
3. `ITE` 指的是 If-Then-Else（接下来的 2 条指令是有条件的）
4. `ITTE` 指的是 If-Then-Then-Else（接下来的 3 条指令是有条件的）
5. `ITTEE` 指的是 If-Then-Then-Else-Else（接下来的 4 条指令是有条件的）

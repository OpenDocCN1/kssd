
# [翻译] ARM汇编简介（三）内存指令-加载和存储（下）

本文是内存指令-加载和存储的下篇，接上文https://bbs.pediy.com/thread-228602.htm继续翻译。

## 2. Offset form: Register as the offset. 偏移形式：寄存器的值用作偏移

```
STR    Ra, [Rb, Rc]

LDR    Ra, [Rb, Rc]
```

该偏移形式将寄存器的值用作偏移量。该偏移形式的一个示例用法是，你的代码想访问一个数组，此时正在计算数组的下标索引。

```
.data

var1: .word 3

var2: .word 4

.text

.global _start

_start:

    ldr r0, adr_var1  @ load the memory address of var1 via label adr_var1 to    R0
                      @ 透过adr_var1标签，将var1变量的内存地址加载进R0

    ldr r1, adr_var2  @ load the memory address of var2 via label adr_var2 to    R1
                      @ 透过adr_var2标签，将var2变量的内存地址加载进R1

    ldr r2, [r0]      @ load the value (0x03) at memory address found in R0 to R2
                      @ 将R0里的值作为内存地址，把地址里的数值（0x03）载入r2

    str r2, [r1, r2]  @ address mode: offset. Store the value found in R2 (0x03) to the memory address found in R1 with the offset R2 (0x03). Base register unmodified.
                      @ 寻址模式：偏移寻址。将R2里的值存入：R1+R2（0x03，偏移）的结果所指向的内存空间中。基址寄存器不更新。

    str r2, [r1, r2]! @ address mode: pre-indexed. Store value found in R2 (0x03) to the memory address found in R1 with the offset R2 (0x03). Base register modified: R1 = R1+R2.
                      @ 寻址模式：先索引模式。将R2的值（0x03）存入：R1+R2（0x03，用作偏移）得出的结果所指向的内存空间中。基址寄存器R1更新为R1 = R1+R2。

    ldr r3, [r1], r2  @ address mode: post-indexed. Load value at memory address found in R1 to register R3. Then modify base register: R1 = R1+R2.
                      @ 寻址模式：后索引模式。将R2的值作为地址取出里面的值，并将其存入寄存器R3。基址寄存器R1更新为R1=R1+R2。

    bx lr

adr_var1: .word var1

adr_var2: .word var2
```

用偏移寻址模式执行完第一个`STR`（`str r2, [r1, r2]`）指令后，`R2`的值（`0x00000003`）会被存储在`0x0001009c + 0x00000003 = 0x0001009F`指向的内存空间。

```
gef> x/w 0x0001009F

 0x1009f <var2+3>: 0x00000003
```

第二个`STR`使用先索引模式执行（`str r2, [r1, r2]!`），该指令做了同样的事，不同的是，第二处`str`会用计算得出的内存地址（`R1+R2`）更新基址寄存器（`R1`）。

```
gef> info register r1

 r1     0x1009f      65695
```

最后一处`LDR`操作（`ldr r3, [r1], r2`）使用后索引寻址模式，将`r1`里的值作为地址取出里面的值并将这个值存入寄存器`r2`。然后更新基址寄存器`R1`（`R1+R2 = 0x1009f + 0x3 = 0x100a2`）。

![](https://bbs.kanxue.com/upload/attach/201806/803510_PBJQSREAQM84D5P.gif)

## 3. Offset form: Scaled register as the offset. 偏移形式：移位寄存器作为偏移

```
LDR    Ra, [Rb, Rc, <shifter>]

STR    Ra, [Rb, Rc, <shifter>]
```

第三偏移形式使用移位寄存器作为偏移量。在本例中，`Rb`是基寄存器，`Rc`里面是立即数偏移量（或是寄存器里的一个立即数），用于左/右移位（`<移位寄存器>`）。这意味着移位寄存器被用来存放移位的偏移量。这种偏移形式的一个示例用法是循环遍历数组。下面是一个简单的例子，可以在`GDB`中运行：

```
.data

var1: .word 3

var2: .word 4

.text

.global _start

_start:

    ldr r0, adr_var1         @ load the memory address of var1 via label adr_var1 to R0
                             @ 透过adr_var1标签，将var1变量的内存地址加载进R0

    ldr r1, adr_var2         @ load the memory address of var2 via label adr_var2 to R1
                             @ 透过adr_var2标签，将var2变量的内存地址加载进R1

    ldr r2, [r0]             @ load the value (0x03) at memory address found in R0 to R2
                             @ 将R0里的值作为内存地址，把里面的值（0x03）载入r2

    str r2, [r1, r2, LSL#2]  @ address mode: offset. Store the value found in R2 (0x03) to the memory address found in R1 with the offset R2 left-shifted by 2. Base register (R1) unmodified.
                             @ 寻址模式：偏移寻址。将R2里的值（0x03）存入：R1的值+偏移量（R2左移2位后的值）的结果所指向的内存空间中。基址寄存器（R1）不更新。

    str r2, [r1, r2, LSL#2]! @ address mode: pre-indexed. Store the value found in R2 (0x03) to the memory address found in R1 with the offset R2 left-shifted by 2. Base register modified: R1 = R1 + R2<<2
                             @ 寻址模式：先索引寻址。将R2里的值（0x03）存入：R1的值+偏移量（R2左移2位后的值）的结果所指向的内存空间中。基址寄存器（R1）更新为R1 = R1 + R2<<2。

    ldr r3, [r1], r2, LSL#2  @ address mode: post-indexed. Load value at memory address found in R1 to the register R3. Then modifiy base register: R1 = R1 + R2<<2
                             @ 寻址模式：后索引寻址。将R1的值作为地址取出里面的值，并将其存入寄存器R3。基址寄存器R1更新为R1=R1+R2<<2。

    bkpt

adr_var1: .word var1

adr_var2: .word var2
```

第一个`STR`操作（`str r2, [r1, r2, LSL#2]`）使用偏移寻址模式，并将在`R2`中找到的值作为地址，取出里面的值存储在：`[R1, R2, LSL#2]`计算出的内存地址指向的内存空间中。这意味着它将`R1`中的值用作基址（此时`R1`包含了`var2`变量的内存地址），然后它取出`R2`（`0x3`）中的值向左移动两位，把这个值作为偏移量。下面的图片将`[R1, R2, LSL#2]`计算内存地址的方法可视化了。

![](https://bbs.kanxue.com/upload/attach/201806/X57RDR4KTB5UYFT.jpg)

第二个操作`STR`（`str r2, [r1, r2, LSL#2]!`）使用先索引地址模式。这意味着，它执行与先前操作相同的动作，不同之处在于，用随后计算出的内存地址更新了基址寄存器`R1`。换言之，它首先将存储器地址`R1`（`0x1009c`）加上偏移：将`R1`向左偏移2位（`0x03 LSL#2 = 0xC`，这是偏移量）得到`0x100a8`，将`0x100a8`作为地址，取出里面的值给`R2`，以及用`0x100a8`更新`R1`。

```
gef> info register r1

r1      0x100a8      65704
```

最后一个`LDR`操作（`ldr r3, [r1], r2`）使用后索引寻址模式。意思是说，将`R1`里的数（`0x100a8`）作为地址，取出该地址空间里的值并将它载入寄存器`R3`，然后，用`r2, LSL#2`计算所得的结果更新基址寄存器`R1`。换言之，使用`R1`的值（`0x100a8`），加上偏移`R2`（`0x3`）左移两位得到的值（`0xC`），等于`0x100b4`，用`0x100b4`更新`R1`。

```
gef> info register r1

r1      0x100b4      65716
```

## Summary 总结

记住`LDR/STR`用到的三种偏移模式：

1. offset mode uses an immediate as offset 用立即数作为偏移的偏移模式

```
ldr   r3, [r1, #4]
```

2. offset mode uses a register as offset 用寄存器的值作为偏移的偏移模式

```
ldr   r3, [r1, r2]
```

3. offset mode uses a scaled register as offset 用移位寄存器的值作为偏移的偏移模式

```
ldr   r3, [r1, r2, LSL#2]
```

如何记住`LDR/STR`中的不同寻址模式：

如果有一个`!`，就是先寻址模式：

```
ldr   r3, [r1, #4]!

ldr   r3, [r1, r2]!

ldr   r3, [r1, r2, LSL#2]!
```

如果方括号里只有基址寄存器它自己，就是后寻址模式：

```
ldr   r3, [r1], #4

ldr   r3, [r1], r2

ldr   r3, [r1], r2, LSL#2
```

其他的都是偏移寻址模式：

```
ldr   r3, [r1, #4]

ldr   r3, [r1, r2]

ldr   r3, [r1, r2, LSL#2]
```

# LDR FOR PC-RELATIVE ADDRESSING  LDR用于PC相对寻址

`LDR`是唯一一个用于将内存中的数据载入寄存器的指令。有时你会看到类似下面所示的语法：

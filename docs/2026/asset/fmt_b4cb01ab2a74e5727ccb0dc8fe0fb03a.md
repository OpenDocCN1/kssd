
# [翻译]ARM汇编简介（三）内存指令-加载和存储 （上）

本篇是ARM系列基础教程的第四篇，加载和存储。

## MEMORY INSTRUCTIONS: LOAD AND STORE 内存指令：加载和存储

ARM uses a load-store model for memory access which means that only load/store (`LDR` and `STR`) instructions can access memory. While on x86 most instructions are allowed to directly operate on data in memory, on ARM data must be moved from memory into registers before being operated on. This means that incrementing a 32-bit value at a particular memory address on ARM would require three types of instructions (load, increment, and store) to first load the value at a particular address into a register, increment it within the register, and store it back to the memory from the register.

ARM使用载入-存储模型来访问内存，意味着只有加载/存储（`LDR`和`STR`）指令才可以访问内存。在X86中，大多数指令允许直接操作内存中的数据，而在ARM中，在操作数据之前，必须把数据从内存移动到寄存器中。这意味着在ARM下，若要
增加特定内存地址里的32位的数值，将需要用到三种类型的指令（载入、增加和存储）：首先将特定地址里的数值加载到寄存器中，然后在寄存器中增加它，最后将数据从寄存器返存回内存里

To explain the fundamentals of Load and Store operations on ARM, we start with a basic example and continue with three basic offset forms with three different address modes for each offset form. For each example we will use the same piece of assembly code with a different `LDR`/`STR` offset form, to keep it simple. The best way to follow this part of the tutorial is to run the code examples in a debugger (`GDB`) on your lab environment.

为了解释ARM的加载和存储操作的基本原理，我们从一个基本示例开始，使用三种基本偏移形式，每个偏移形式都用三种不同的寻址模式表示。对于每个示例，我们将借助不同的`LDR`/`STR`偏移形式， 并使用含义相同汇编代码让问题简化。学习教程的这一部分的最佳方法是在你的实验环境中用调试器（`GDB`）来运行示例代码。

### 1.Offset form: Immediate value as the offset 偏移形式：将立即数作为偏移

Addressing mode: Offset 寻址模式：偏移寻址

Addressing mode: Pre-indexed 寻址模式：先索引寻址

Addressing mode: Post-indexed
寻址模式：后索引寻址

### 2.Offset form:Register as the offset

偏移形式：将寄存器作为偏移

Addressing mode: Offset
寻址模式：偏移寻址

Addressing mode: Pre-indexed 寻址模式：先索引寻址

Addressing mode: Post-indexed
寻址模式：后索引寻址

### 3.Offset form:Scaled register as the offset 偏移形式：将移位寄存器作为偏移

Addressing mode: Offset
寻址模式：偏移寻址

Addressing mode: Pre-indexed 寻址模式：先索引寻址

Addressing mode: Post-indexed 寻址模式：后索引寻址

## First basic example 第一个基本示例

Generally, `LDR` is used to load something from memory into a register, and `STR` is used to store something from a register to a memory address.

通常，`LDR`用于将内存数据加载到寄存器中，`STR`用于从寄存器的值存储到内存地址对应的内存中。

![](https://bbs.kanxue.com/upload/attach/201806/5GUBK76KEGQPC7J.jpg)

```
LDR R2, [R0]   @ [R0] - origin address is the value found in R0.

                         @[R0] - 原始地址是R0里的数值

STR R2, [R1]   @ [R1] - destination address is the value found in R1.

                         @[R1] - 目标地址是R1里的数值
```

`LDR` operation: loads the value at the address found in `R0` to the destination register `R2`.

`STR` operation: stores the value found in `R2` to the memory address found in `R1`.

`LDR`操作：将在`R0`中找到的地址的值加载到目标寄存器`R2`。

`STR`操作：将`R2`中找到的值存储在`R1`中找到的内存地址中。

This is how it would look like in a functional assembly program:

下面是它在功能汇编程序中的样子：

```
.data          /* the .data section is dynamically created and its addresses cannot be easily predicted   .data 区段是动态创建的，它的区段地址不容易被预测*/

var1: .word 3  /* variable 1 in memory    定义第一个内存变量*/

var2: .word 4  /* variable 2 in memory    定义第二个内存变量*/

.text          /* start of the text (code) section    文本（代码）段的起始*/

.global _start



_start:

    ldr r0, adr_var1  @ load the memory address of var1 via label adr_var1 into R0 @通过 adr_var1这个标签，将var1变量的地址存储进R0里        

    ldr r1, adr_var2  @ load the memory address of var2 via label adr_var2 into R1 @将var2变量的内存地址通过 adr_var2 标签载入R1里

    ldr r2, [r0]      @ load the value (0x03) at memory address found in R0 to register R2  @将r0里找到的数值作为地址，把它里面的数值（0x03）加载到R2里

    str r2, [r1]      @ store the value found in R2 (0x03) to the memory address found in R1  @将r2里找到的值（0x03）存入r1里的值为内存地址指向的内存空间

    bkpt                 中断在此处



adr_var1: .word var1  /* address to var1 stored here */ var1变量的内存地址存储在这

adr_var2: .word var2  /* address to var2 stored here */ var2变量的内存地址存储在这
```

At the bottom we have our Literal Pool (a memory area in the same code section to store constants, strings, or offsets that others can reference in a position-independent manner) where we store the memory addresses of `var1` and `var2` (defined in the data section at the top) using the labels `adr_var1` and `adr_var2`. The first `LDR` loads the address of `var1` into register `R0`. The second `LDR` does the same for `var2` and loads it to `R1`. Then we load the value stored at the memory address found in `R0` to `R2`, and store the value found in `R2` to the memory address found in `R1`.

在底部，我们有一个文本池（和代码段在同一内存区段，用于存储常量，字符串，或者偏移。其他可以以单独定位的形式来引用它们），在本例中，我们使用标签
`adr_var1`和
`adr_var2`来存储`var1`变量和`var2`变量（在顶部的数据段中定义的）的内存地址。第一个`LDR`指令将`var1`的地址加载到寄存器`R0`中。第二个`LDR`指令对`var2`做了同样的事并将其加载到`R1`。然后，将存储在`R0`中的内存地址加载到`R2`里，并将`R2`中找到的值存储在`R1`中找到的内存地址中。

When we load something into a register, the brackets (`[ ]`) mean: the value found in the register between these brackets is a memory address we want to load something from.

When we store something to a memory location, the brackets (`[ ]`) mean: the value found in the register between these brackets is a memory address we want to store something to.

当我们将某数据加载到寄存器中时，方括号（[]）表示：从这些括号之间的寄存器找出的值是一个内存地址，我们要加载的数据就是从该内存地址中取出的。

当我们将某数据存储到内存位置时，方括号（[]）表示：
从这些括号之间的寄存器找出的值是一个内存地址，我们要存储的数据就是从该内存地址中取出的。

This sounds more complicated than it actually is, so here is a visual representation of what’s going on with the memory and the registers when executing the code above in a debugger:

上述表述和实际情况相比要复杂得多，所以贴出下图，是调试器上执行代码时对内存和寄存器进行的可视化表示：

![](https://bbs.kanxue.com/upload/attach/201806/803510_RMZ4UXGSF5TCW5R.gif)

Let’s look at the same code in a debugger.

我们来看看同样的代码在调试器里是什么样子的

```
gef> disassemble _start

Dump of assembler code for function _start:

 0x00008074 <+0>:      ldr  r0, [pc, #12]   ; 0x8088 <adr_var1>

 0x00008078 <+4>:      ldr  r1, [pc, #12]   ; 0x808c <adr_var2>

 0x0000807c <+8>:      ldr  r2, [r0]

 0x00008080 <+12>:     str  r2, [r1]

 0x00008084 <+16>:     bx   lr

End of assembler dump.
```

The labels we specified with the first two `LDR` operations changed to `[pc, #12]`. This is called PC-relative addressing. Because we used labels, the compiler calculated the location of our values specified in the Literal Pool (`PC+12`).  You can either calculate the location yourself using this exact approach, or you can use labels like we did previously. The only difference is that instead of using labels, you need to count the exact position of your value in the Literal Pool. In this case, it is 3 hops (4+4+4=12) away from the effective PC position. More about PC-relative addressing later in this chapter.

前两个`LDR`操作中用到的标签变成了 [pc, #12]
。这叫做PC相对寻址。因为我们要使用标签，所以编译器计算出了文本池（`PC+12`）中指定标签的位置。你既可以用这个方法自己计算出数据所在的精确的位置，也可以像之前那样使用标签。唯一的区别是，比起使用标签的办法，你需要自己计算你的数值在文字池中的确切位置。在当前这种情形下，文本池是在PC有效位的3个指令之后（4 + 4 + 4＝12）。更多关于PC相对寻址的知识将本章稍后讲解。

Side note: In case you forgot why the effective PC is located two instructions ahead of the current one, it is described in Part 2 [… During execution, PC stores the address of the current instruction plus 8 (two ARM instructions) in ARM state, and the current instruction plus 4 (two Thumb instructions) in Thumb state. This is different from x86 where PC always points to the next instruction to be executed…].

边注：如果你忘记了为什么有效PC位于当前的两个指令之后，在本教程第二部分中曾介绍过：[……在执行期间，ARM状态下`PC`里存储的是当前指令的地址加上8（两个ARM指令的长度），在Thumb状态下存储着当前指令加上4（两个Thumb指令的长度）的值。这与x86不同里PC总是指向下一个要执行的指令不同。如下图所示

![](https://bbs.kanxue.com/upload/attach/201806/ZYN29KTZP5S6445.jpg)

译者注：可参考之前的翻译https://bbs.pediy.com/thread-228309.htm

## 1. Offset form:Immediate value as the offset 第一种偏移形式：立即数用作偏移

```
STR    Ra, [Rb, imm]

LDR    Ra, [Rc, imm]
```

Here we use an immediate (integer) as an offset. This value is added or subtracted from the base register (`R1` in the example below) to access data at an offset known at compile time.


之前说过，`LDM`和`STM`有多种不同的使用形式。具体是哪一种使用形式由指令的后缀所决定。这个示例列出了后缀的几种形式：`-IA` (之后增加), `-IB` (之前增加), `-DA` (之后减少), `-DB` (之前减少)。这么多种类型，他们之间是如何区分的呢？由指令的第一个字段：运算指令规定了访问内存的方式（寄存器作为源地址还是目标地址）。实际上，由于`LDM`和`LDMIA`作用相同，所以每次载入完成后，地址指针会自己增加，从而指向下一个被载入的元素。用这种方法可以从某内存地址中获取（
前向
）序列化的数据，并载入寄存器，

```
ldmia r0, {r4-r6} /* words[3] -> r4 = 0x03, words[4] -> r5 = 0x04; words[5] -> r6 = 0x05;  */ 

stmia r1, {r4-r6} /* r4 -> array_buff[0] = 0x03; r5 -> array_buff[1] = 0x04; r6 -> array_buff[2] = 0x05 */
```

After executing the two instructions above, the registers `R4-R6` and the memory addresses `0x000100D0`, `0x000100D4`, and `0x000100D8` contain the values `0x3`, `0x4`, and `0x5`.

执行完上面的两条指令后，寄存器`R4-R6`分别包含了
`0x3`，`0x4,0x5`
，内存地址
`0x000100D0, 0x000100D4,和 0x000100D8`中也包含了`0x3`，`0x4,0x5`

```
gef> info registers r4 r5 r6

r4     0x3     3

r5     0x4     4

r6     0x5     5

gef> x/3w 0x000100D0

0x100d0 <array_buff>: 0x00000003  0x00000004  0x00000005
```

The `LDMIB` instruction first increases the source address by 4 bytes (one word value) and then performs the first load. In this way we still have a sequential (forward) loading of data, but the first element is with a 4 byte offset from the source address. That’s why in our example the first element to be loaded from the memory into the `R4` by `LDMIB` instruction is `0x00000004` (the words[4]) and not the `0x00000003` (words[3]) as pointed by the `R0`.

`LDMIB`指令先将源地址增加4字节（一个字），然后开始载入数据。该方法仍能让我们载入一串前向数据，但第一个数据的起始地址相较源地址有4个字节的偏移。这就是为什么我们用`LDMIB`指令将内存中的第一个元素载入`R4`后，`R4`是`0x04`（word[4]），而不是`R0`指向的`0x3`（word[3]）

```
ldmib r0, {r4-r6}            /* words[4] -> r4 = 0x04; words[5] -> r5 = 0x05; words[6] -> r6 = 0x06 */

stmib r1, {r4-r6}            /* r4 -> array_buff[1] = 0x04; r5 -> array_buff[2] = 0x05; r6 -> array_buff[3] = 0x06 */
```

After executing the two instructions above, the registers `R4-R6` and the memory addresses `0x100D4`, `0x100D8`, and `0x100DC` contain the values `0x4`, `0x5`, and `0x6`.

执行完上面两条指令后，寄存器`R4-R6`存储了`0x4,0x5,0x6`，
`0x100D4, 0x100D8, 和0x100DC`也存储了
`0x4,0x5,0x6`。

```
gef> x/3w 0x100D4

0x100d4 <array_buff+4>: 0x00000004  0x00000005  0x00000006

gef> info register r4 r5 r6

r4     0x4    4

r5     0x5    5

r6     0x6    6
```

When we use the `LDMDA` instruction everything starts to operate backwards. `R0` points to words[3]. When loading starts we move backwards and load the words[3], words[2] and words[1] into `R6`, `R5`, `R4`. Yes, registers are also loaded backwards. So after the instruction finishes `R6 = 0x00000003`, `R5 = 0x00000002`, `R4 = 0x00000001`. The logic here is that we move backwards because we Decrement the source address AFTER each load. The backward registry loading happens because with every load we decrement the memory address and thus decrement the registry number to keep up with the logic that higher memory addresses relate to higher registry number. Check out the `LDMIA` (or `LDM`) example, we loaded lower registry first because the source address was lower, and then loaded the higher registry because the source address increased.

使用`LDMDA`向相反方向执行运算（译者注：`ldmda r0, {r4-r6}`       /* words[3] -> r6 = 0x03; words[2] -> r5 = 0x02; words[1] -> r4 = 0x01 */）。`R0`指向word[3]，载入操作开始后，我们向后将
words[3], words[2]和words[1]载入`R6, R5, R4`。是的你没有看错，连寄存器都是反着被载入的。所以指令将
`R6`赋值为`0x00000003, R5`赋值为`0x00000002, R4`赋值为`0x00000001`。这里的逻辑是，每次载入操作完成后向后减少内存地址所以是反向赋值。寄存器之所以也是反向载入的是由于我们减少了内存地址，所以对应的寄存器的编号值也跟着下降，因为它需要遵循更高的内存地址关联了更高的寄存器编号这一逻辑。回头检查一下
`LDMIA` (或是 `LDM`)那个例子（译者注：`ldmia r0, {r4-r6}` /* words[3] -> r4 = 0x03, words[4] -> r5 = 0x04; words[5] -> r6 = 0x05;  */ ），我们先载入编号更低的寄存器，因为源地址更低。然后载入到编号更高的寄存器，因为源地址增加了。

Load multiple, decrement after:

载入值，之后降低地址（译者注：建议不要纠结字面含义，建议对照代码理解）

```
ldmda r0, {r4-r6} /* words[3] -> r6 = 0x03; words[2] -> r5 = 0x02; words[1] -> r4 = 0x01 */
```

Registers `R4`, `R5`, and `R6` after execution:

`R4`，`R5`，`R6`的变化：

```
gef> info register r4 r5 r6

r4     0x1    1

r5     0x2    2

r6     0x3    3
```

Load multiple, decrement before:

先降低地址，再载入多个值
（译者注：建议不要纠结字面含义，建议对照代码理解）

```
ldmdb r0, {r4-r6} /* words[2] -> r6 = 0x02; words[1] -> r5 = 0x01; words[0] -> r4 = 0x00 */
```

Registers `R4`, `R5`, and `R6` after execution:

`R4`，`R5`，`R6`的变化

```
gef> info register r4 r5 r6

r4 0x0 0

r5 0x1 1

r6 0x2 2
```

Store multiple, decrement after.

存储值，之后降低地址
（译者注：建议不要纠结字面含义，建议对照代码理解）

```
stmda r2, {r4-r6} /* r6 -> array_buff[2] = 0x02; r5 -> array_buff[1] = 0x01; r4 -> array_buff[0] = 0x00 */
```

Memory addresses of `array_buff[2]`, `array_buff[1]`, and `array_buff[0]` after execution:

观察
`array_buff[2], array_buff[1], 和 array_buff[0]`执行后的改变

```
gef> x/3w 0x100D0

0x100d0 <array_buff>: 0x00000000 0x00000001 0x00000002
```

Store multiple, decrement before:

先给地址指针减4，再一边赋值一边降序

```
stmdb r2, {r4-r5} /* r5 -> array_buff[1] = 0x01; r4 -> array_buff[0] = 0x00; */
```

Memory addresses of `array_buff[1]` and `array_buff[0]` after execution:

观察一下
`array_buff[1]`和`array_buff[0]`执行后的结果

```
gef> x/2w 0x100D0

0x100d0 <array_buff>: 0x00000000 0x00000001
```

# PUSH AND POP push和pop

There is a memory location within the process called Stack. The Stack Pointer (`SP`) is a register which, under normal circumstances, will always point to an address wihin the Stack’s memory region. Applications often use Stack for temporary data storage. And As mentioned before, ARM uses a Load/Store model for memory access, which means that the instructions `LDR` / `STR` or their derivatives (`LDM..` /`STM..`) are used for memory operations. In x86, we use `PUSH` and `POP` to load and store from and onto the Stack. In ARM, we can use these two instructions too:

进程中有使用一块内存区域叫堆栈。堆栈指针（`SP`）是一个寄存器，在正常情况下，它总是指向栈内存区域中的一个地址。应用程序通常使用堆栈进行临时数据存储。之前提过，ARM使用加载/存储模型进行内存访问，这意味着指令`LDR/STR`或它们的派生指令（`LDM.../STM...`）用于内存操作。在x86中，我们使用`PUSH`和`POP`从堆栈中加载和存储。在ARM中，我们也可以使用这两个指令：

When we `PUSH` something onto the Full Descending (more about Stack differences inPart 7: Stack and Functions) stack the following happens:

当我们把数据压入降序分布的栈区（详见第7部分：栈和函数）后会发生：

> 1.First, the address in `SP` gets DECREASED by 4.
>
> 首先，`SP`寄存器里的地址值减4
>
> 2.Second, information gets stored to the new address pointed by `SP`.
>
> 第二步，将信息储存在`SP`指向的新的地址空间中

When we `POP` something off the stack, the following happens:

当把数据弹出栈时会发生：

> 1.The value at the current `SP` address is loaded into a certain register,
>
> 当前`SP`地址所指向的内存里的值内载入到相应寄存器中
>
> 2.Address in `SP` gets INCREASED by 4.
>
> `SP`里的值加4

In the following example we use both `PUSH/POP` and `LDMIA/STMDB`:

下面的例子中，我们既使用了
`PUSH/POP`，也使用了
`LDMIA/STMDB`

```
.text

.global _start



_start:

   mov r0, #3

   mov r1, #4

   push {r0, r1}

   pop {r2, r3}

   stmdb sp!, {r0, r1}

   ldmia sp!, {r4, r5}

   bkpt
```

Let’s look at the disassembly of this code. 看看汇编代码是什么样的

```
azeria@labs:~$ as pushpop.s -o pushpop.o

azeria@labs:~$ ld pushpop.o -o pushpop

azeria@labs:~$ objdump -D pushpop

pushpop: file format elf32-littlearm



Disassembly of section .text:



00008054 <_start>:

 8054: e3a00003 mov r0, #3

 8058: e3a01004 mov r1, #4

 805c: e92d0003 push {r0, r1}

 8060: e8bd000c pop {r2, r3}

 8064: e92d0003 push {r0, r1}

 8068: e8bd0030 pop {r4, r5}

 806c: e1200070 bkpt 0x0000
```

As you can see, our `LDMIA` and `STMDB` instuctions got translated to `PUSH` and `POP`. That’s because `PUSH` is a synonym for `STMDB sp!, reglist` and `POP` is a synonym for `LDMIA sp! reglist` (seeARM Manual)

正如你看到的，我们的
`LDMIA` 和`STMDB`指令被转换成了
`PUSH`和`POP`。这是因为push有同义词`stmdb SP!`，而pop有同义词
`LDMIA sp!`（见arm手册）

Let’s run this code in GDB.在GDB里运行一下

```
gef> break _start

gef> run

gef> nexti 2

[...]

gef> x/w $sp

0xbefff7e0: 0x00000001
```

After running the first two instructions we quickly checked what memory address and value `SP` points to. The next `PUSH` instruction should decrease `SP` by 8, and store the value of `R1` and `R0` (in that order) onto the Stack.

运行两条指令后我们快速检查一下`SP`指向哪，这个地址里面的值是多少。下条`PUSH`指令（`push {r0, r1}`）应该会把`SP`减8，并将`R0`和`R1`的值压入堆栈

```
gef> nexti

[...] ----- Stack -----

0xbefff7d8|+0x00: 0x3 <- $sp

0xbefff7dc|+0x04: 0x4

0xbefff7e0|+0x08: 0x1

[...] 

gef> x/w $sp

0xbefff7d8: 0x00000003
```

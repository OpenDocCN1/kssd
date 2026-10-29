
2. `vm_begin`

3. `vm_run`

4. `vm_return`

5. `vm_do_ops1`

6. `vm_do_ops2`

7. `vm_lshift_op`

8. `vm_rshift_op`

9. `vm_mathop`

可能是这样吧，我画个图说明下:

![图片描述](https://bbs.kanxue.com/upload/attach/202102/630711_W2ZV6KU79HE8WNH.png)

用黄色标明的方块和整个灰色背景的块，都是函数。

`Vm_do_cbranch`, `vm_do_lshift`, … 没有边指向它，是因为我不想把图画的太乱，实际上有不同的case分支指向它。

打开ida 用`libjiagu.so`做对照，`_Z10**arm_a_21v`地址83d0:

![图片描述](https://bbs.kanxue.com/upload/attach/202102/630711_CQEV3QPVH395572.png)

然后我们打开 `_Z9**arm_a_0v`

![图片描述](https://bbs.kanxue.com/upload/attach/202102/630711_RGF64GSZXAXS7YU.png)

然后我们从最外层的`vm_enter`开始，inline掉 `vm_begin`和`vm_run`函数。

让框架上变成这样:

![图片描述](https://bbs.kanxue.com/upload/attach/202102/630711_KUSXTMRHDDBJE9C.png)

严格上来说，所有的vm框架函数都要inline，但是这样会导致代码体积膨胀的非常厉害。我们采取部分完全inline，然后还有部分按需inline的方式来做优化

## 2种handler

360加固在把代码转成了`vm_bytecode`以后，生成了2种handler:

第一种handler搞不清作用，是在最外层的`vm_engine`上执行的case，我们称为`main_handler`

第二种handler看起来是vm op。

初步怀疑外层的handler可能是用来描述原始汇编指令的，比如一个`mov`指令，可能转成2个vm操作

|  |  |
| --- | --- |
| 1  2  3  4 | `mov r0, r1` `/``/` `原始的mov可能有个编号，在外层` `VM_PUSH r1` `/``/` `然后虚拟的hanlder也有个编号` `VM_POP r0` |

但我们不关心这些细节。反正不停的循环展开即可。

## 删除可计算循环

## VM_BYTEINDEX的0生成

我们开始走vm的虚拟机，在360中，`r12`是`vm_byteindex`，需要先初始化他为0，360的初始化采用了一些特殊规则来生成。

1. ![图片描述](https://bbs.kanxue.com/upload/attach/202102/630711_5PZ6X4QT5EAW7E8.png)

2. ![图片描述](https://bbs.kanxue.com/upload/attach/202102/630711_BX63V2PMJ25E4R5.png)

3. ![图片描述](https://bbs.kanxue.com/upload/attach/202102/630711_F7ZKB8E4H5YBRE7.png)

以上的代码简化以后如下:

|  |  |
| --- | --- |
| 1  2  3  4  5 | `r0` `=` `lrand48();` `r6` `=` `r0` `r2` `=` `r6` `*` `r6` `+` `r6 (这里r2一定为偶数)` `r2` `=` `r2 &` `1` `-``> r2` `=` `0` `r12` `=` `r2` `-``> r12` `=` `0` |

推导出`r12`为0

## VM_BYTEINDEX和归纳变量判断

这个地方我偷懒了，没有严格按照书上来，做了一些硬编码，按如下规则来:

1. 生成spanning tree，标注边类型，`a_tree_edge`, `forward`, `cross`, `back edge`

2. 扫描整个block list，找到哪个block的回边最多的，就是vmhead block

3. Ghidra中根据 [400K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8%4u0W2i4K6u0W2j5h3y4Q4x3X3g2#2K9#2)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0r3M7r3c8X3i4K6u0r3z5o6t1H3x3K6t1H3x3K6g2Q4x3X3g2H3k6r3j5`.](https://bbs.kanxue.com/elink@fddK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8%4u0W2i4K6u0W2j5h3y4Q4x3X3g2#2K9#2)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0r3M7r3c8X3i4K6u0r3z5o6t1H3x3K6t1H3x3K6g2Q4x3X3g2H3k6r3j5`.) 这篇论文来识别不可归约流图，我在这个上面又改了下，变成识别循环的函数。

4. 根据哪些节点属于循环来标识循环内外，主要用来标识loop的exit节点，正向编译器开发中，这些都是现成的，上层的ir传给低层的ir，会使用`gated-ssa`来强化ssa的一些表达能力。

5. 找到vmhead，直接扫描比较指令，扫描到，比如`cmp r9, r12`

6. 根据loop的pre节点，获取它反向输入索引，`pre->get_rev_index(vmhead)`，替换到vmhead的phi节点中，得到`r9` 和 `r12`的值，

7. `R9`为`0x192`，`r12`为0，我直接判定`r12`为归纳变量，或者`vm_byteindex`，这里其实不是很对

## 循环展开

基本算法:

1. 获取头的pre节点

2. 根据pre的inslot值，跟新node上的phi节点

3. 一直在cfg上走

4. 假如直接走到了vmhead，复制经过的所有节点，形成一个cfg块

5. 假如走到了undefined bcond，复制经过的所有节点，形成一个cfg节点a，然后从undefined bcond节点的out边开始复制，把整个web递归复制到vmhead节点为止。和前面的cfg a拼接在一起。行为一个cfg块

6. 删除pre节点和vmhead节点的把，把cfg块加入进去。

### 普通展开

估计你们看的头晕，直接上图吧，针对4的情况

![图片描述](https://bbs.kanxue.com/upload/attach/202102/630711_622EYM8PFTZVXPT.png)

我们假设先走的case5，直接按途中1, 2 ,3 ,4 ,5的顺序把代码全走出来。然后生成一个新的绿色节点。重新插入到pre和`vm_engine_head`中。

![图片描述](https://bbs.kanxue.com/upload/attach/202102/630711_XSE87WPFKM27M93.png)

这是好的情况，大部分运气都非常糟糕，走的是上图中case1的分支，接下去我们直接上`libjiagu.so`中的真实数据。

### 复杂展开:

我们直接把`_Z10__arm_a_21v`的第一个`vm_bytecode`展开

它经过了以下节点:

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8 | `process flowblock sub_cc10` `process flowblock sub_cc18` `process flowblock sub_cc20` `process flowblock sub_cc24` `process flowblock sub_cc88` `process flowblock sub_d278` `process flowblock sub_d29c` `process flowblock sub_d2ac` |

![图片描述](https://bbs.kanxue.com/upload/attach/202102/630711_YTZ3K4YVZY7EE7C.png)

`Cc24`还有`cc28`，以及最后的3个节点。

![图片描述](https://bbs.kanxue.com/upload/attach/202102/630711_QETRQSSKZZYGVBE.png)

我在cfg流图中用绿色或者粉红表示有循环的点，`d2ac`已经无法分析了，我们直接把从它的出边开始到vmhead的所有node以及关系都复制出去。

![图片描述](https://bbs.kanxue.com/upload/attach/202102/630711_H2SAG8T95WYFSQD.png)

那到了这里该如何优化呢，看下面的lazy inline

## lazy inline

我在框架分析中说过，原则上来说要inline掉框架内所有的框架函数，但是假如一开始就inline全部函数，会导致代码体积过大。所以我们是一边循环展开一边inline。我们看下上图中，那个被展开的节点:

![图片描述](https://bbs.kanxue.com/upload/attach/202102/630711_6RFTCD82VXCQ9ZW.png)

我们到了这一步才开始决定inline它

## argument inline

传统的inline功能会把整个代码全部inline入函数，这样会导致整个程序流图变的巨大，

1. 我们直接把整个参数全部传进去

2. 别名分析时，可以直接搜caller的堆栈

3. 让它优化完了，在传送出来。这样可以巨大的减小体积。

4. 假如函数内还有调用其他的call，只要时框架函数全部带参数inline掉

上图中的`vmp_math_op1`没优化前长这样:

![图片描述](https://bbs.kanxue.com/upload/attach/202102/630711_P4CG34UUE4URK7N.png)

优化以后长这样:

![图片描述](https://bbs.kanxue.com/upload/attach/202102/630711_NGN965X6FD4XBQC.png)

就只剩下一个唯一的块了，这样就方便多了。最后嵌入到原先的代码中。

嵌入到原先的块后，

![图片描述](https://bbs.kanxue.com/upload/attach/202102/630711_PHXS9TCSKDN23ZT.png)

原先块的末尾那个cbranch就变得可以计算了。会删除其中一边

1. 假如删除左边，那么左边整个模块都变成unreachable了，可以开始下一轮对vmhead的循环展开了。

2. 假如是右边的话，左边保留了下来，因为左边长这样:

![图片描述](https://bbs.kanxue.com/upload/attach/202102/630711_HWES4FWZSPCFQG5.png)

它是一个循环，所以需要继续对do..while代码做循环展开。展开的方式都是一样的。一直到展开不动为止。

最后上面那一坨循环就会变成下面这样一个标准快了。

![图片描述](https://bbs.kanxue.com/upload/attach/202102/630711_J6SDR6QCFW432T6.png)

然后我们发现，上面箭头指向的`vmeip(vm_bytecode_index)`的值被跟新到`0x19`了

这个图好像不是很清晰的样子

# 值和格

![图片描述](https://bbs.kanxue.com/upload/attach/202102/630711_JFTBV7T4JY8QEGW.png)

我们需要扩展数据的一些类型定义

我们扩展了数据类型的定义，增加了一种namespace的概念。为什么要加呢？

|  |  |
| --- | --- |
| 1 | `store [esp], eax ; esp` `=` `esp.``0` `-` `200` |

我们在分析别名的时候，你打算如何标注函数的数据类型为`esp – 200`，`esp-200`是一个我们推导出的值。

我们不打算把`esp`设为一个默认的常量，这其实是一种模拟执行的思路，在编译优化时，可能会引入一些意想不到的bug。

新的类型定义变成这样:

![图片描述](https://bbs.kanxue.com/upload/attach/202102/630711_V8AQ7XYFRXEYFN5.png)

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10 | `struct` `type` `{` `enum {` `a_top,` `a_constant,` `a_rel_constant,` `bottom` `}` `int` `con;` `void` `*``space;` `}` |

常量不能和相对常量比较，相同空间的常量可以互相比较，不同空间是正交的。相同空间的常量可以比较大小。

比如 : `esp – 200` 会大于 `esp – 100`

上面还有2个奇怪的空间，其中一个是alloc space，这个来源于另外一个问题

|  |  |
| --- | --- |
| 1  2  3 | `1.` `store [sp` `+` `200``], r0` `2.` `store [r1], r3` `3.` `r4` `=` `load [sp` `+` `200``]` |

假如这个`r1`是alloc space的，那么3可以关联到1，因为alloc space，代表`r1`是malloc或者new出来的，malloc出的内存，你在上面操作，不会和堆栈和其他的空间发生重叠。Caller space也类似。

现在只支持stack space.

# 别名分析

## 别名分析.正（抛弃）

这一章是用比较传统的别名分析来做的，但是在深入分析vmp时碰到了一些难以解决的问题，在成功解码一个被vmp保护的函数后抛弃了。

正向的别名分析，就是严格计算哪些别名时安全，哪些是may def, may use，这个在正向编译开发中很有用，因为编译器可以拿到全部的数据结构，和全部的调用链，但是你在逆向的编译开发中拿不到全部的信息。

**更重要的一点是，内存的位置是由开发者决定的，逆向的时候，你没资格决定哪个结构在哪里。**

## 别名分析.反

我们不再追求分析出每一个别名的must def, must use，may def, may use。而是假设某一块内存区域绝对安全。或者说是隔离。

什么是别名的绝对安全，也就是说它不会被may def, may use。它的所有must def , must use都是可以计算的，假如碰到不能计算的节点的store, load，那么操作的也一定不是这块区域的内存。

先来看一段示例代码

|  |  |
| --- | --- |
| 1  2  3 | `1.` `store [sp], r0` `2.` `store [r1], r2; r1` `=` `T` `3.` `r3` `=` `load [sp]` |

指令1,3都在操作 `sp + 0`的位置，指令2在操作一个未知位置，并且在常量传播以后，也依然是 `T` 的。

接下我我们使用安全区域标注:

|  |  |
| --- | --- |
| 1 | `set_safe_room(sp,` `0``,` `4``);` |

直接标注这个区域是安全，那么我们认为所有计算不出来得位置和安全区域都是正交的。这样指令3的load和指令1的store就关联起来，并且可以近一步转换成:

|  |  |
| --- | --- |
| 1  2  3 | `1.` `store [sp], r0` `2.` `store [r1], r2` `3.` `mov r3, r0` |

## 实际项目中如何应用？

标注安全区域确实方便，但是如何知道哪些区域绝对安全呢？

VMP在生成`vm_enter`的时候，会生成一个区域保护用户堆栈，在生成一个堆栈操作operand， 我们认为这个区域，这个是相对于整个算法的 vmp-dependent 的部分。我们认为这个区域的操作不会和任何非vm-dependent call产生交互。

## 函数如何处理

|  |  |
| --- | --- |
| 1  2  3  4 | `store [sp], r0` `mov r0, sp` `call.mathop (r0)` `load r1, [r0]` |

假如这个函数是vmp生成的框架函数必须得inline掉，具体如何inline参考章节 《360加固.lazy inline》 和 《360加固.argument inline》

## 引入的问题

|  |  |
| --- | --- |
| 1  2  3 | `1.` `store [sp], r0` `2.` `store [r2], r3` `3.` `r5` `=` `load [sp]` |

假如我们在没有充分计算的前提下就把指令 1, 3关联起来。那么也许在常量传播到后面，我们才发现

|  |  |
| --- | --- |
| 1  2  3 | `1.` `store [sp], r0` `2.` `store [r2], r3 ; r2` `=` `sp` `3.` `r5` `=` `load [sp]` |

实际上应该关联的是 3和2。

所以我们必须得计算到完全不能在计算为止，才能进行关联，或者标注某个store指令不可计算。

下一节会讲述什么是充分性计算。

## 充分性计算

定义:

1. 假如一个pccode的所有in节点都是

1.1 常量

1.2 输入参数

1.3 充分计算过的

2. 那么此pcode的output也都是充分计算过的

3. 递归遍历此output的use，重复1-3过程

伪代码:

|  |  |
| --- | --- |

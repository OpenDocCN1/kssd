
- `path2`：`a<=1`，false。

  - 这条路径的条件约束是`a<=1`，如果走这条路径，输出`b`的约束条件就是`a+1`。

如果我们想要程序能够走到两条路径，达到100%执行覆盖率，那很简单，求解器会随便取两个值，一个是大于1的值，另一个是小于等于1的值，这两个输入就足够覆盖两条路径。

如果我们想要`b`输出一个特定值，也很简单。

假如我们想要`b`输出一个值“2”，那么我们有两条约束：

- 1）`a>1`且`a-1=2`；
- 2）`a<=1`且`a+1=2`。

可计算出结果：

- 1）`a=3`
- 2）`a=1`

也就说约束求解器会给出两个答案：`a=0`或`a=3`。又如我们想让`b`输出0，那么两条约束：

- 1）`a>1`且`a-1=0`；
- 2）`a<=1`且`a+1=0`。

这就只能算出一个答案`a=-1`。

### 1.3 复杂例子

下面这个例子来自`angr_ctf`的ppt。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12 | `#define SECRET 100`  `int` `check_code(` `int` `input`) {`  `if` `(` `input` `>=` `SECRET` `+` `88`)` `return` `0`;`  `if` `(` `input` `> SECRET` `+` `100`)` `return` `0`;`  `if` `(` `input` `==` `SECRET` `+` `68`)` `return` `0`;`  `if` `(` `input` `< SECRET)` `return` `0`;`  `if` `(` `input` `<=` `SECRET` `+` `78`)` `return` `0`;`  `if` `(` `input` `&` `0x1`)` `return` `0`;`  `if` `(` `input` `&` `0x2`)` `return` `0`;`  `if` `(` `input` `&` `0x4`)` `return` `0`;`  `return` `1`;`  `}` |

这段代码有哪些可能的路径？可以把这些路径表示成一个树，然后使用搜索算法来找到`return 1`节点。广度优先遍历是个好选择，`angr`默认使用的也是它。

![image-20201229215457286](https://bbs.kanxue.com/upload/attach/202101/830671_9PXD37ZRBJPW5DH.jpg)

一旦我们选择了一条执行路径，那么就可以构建一个SMT公式来求解符号。当然，在现实世界中，二进制文件会更复杂，随之，符号也就会更多，路径也会更多。二进制文件的复杂性呈指数增长是符号执行的最大问题。

![image-20201230090344641](https://bbs.kanxue.com/upload/attach/202101/830671_A8TGHUFRT6HJVF4.jpg)

> **SMT**:
>
> SAT（satisfiability）问题指的是命题逻辑公式的可满足性问题。但是 SAT 在表达能力上有很大的局限性，需要比 SAT 更强的表达方式。在这种形势下，将SAT问题扩展为SMT，经过扩展，SMT 能比 SAT 更好地表达一些人工智能和形式化方法领域内的问题，比如在资源规划、时序推理、编译器优化等很多方面用到了SMT。
>
> SMT 的全称是 Satisfiability Modulo Theories，可被翻译为“可满足性模理论”、“多理论下的可满足性问题”或者“特定（背景）理论下的可满足性问题”，其判定算法被称为 SMT 求解器。简单地说，一个 SMT公式是**结合了理论背景的逻辑公式，其中的命题变量可以代表理论公式**。
>
> 这里举个例子，命题逻辑公式是这样：
>
> ![image-20210101230246017](https://bbs.kanxue.com/upload/attach/202101/830671_KXGV7DDM8JWAQA9.jpg)
>
> 即只允许出现变量、变量的否定和逻辑连接词
>
> 而SMT公式可以是这样：
>
> ![image-20210101230313903](https://bbs.kanxue.com/upload/attach/202101/830671_RY77FQP5MGRDUGK.jpg)
>
> 即可以出现非逻辑符号，比如函数`g(a)`、`f(g(a))`和常量`c`、`d`，其中`g(a) = c` 这些理论公式可以替代命题变量。
>
> P.S. 希望看雪的md编辑器能加上latex公式的功能。

那么怎么去遍历程序，找到我们想要的执行路径，然后求解出符号`λ`呢？ 正在学习`angr`，那这里肯定就是用`angr`了。`angr`是一个符号执行引擎。它能1）遍历程序并找到所有分支；2）搜索程序里符合给定条件的状态；3）通过给定路径约束，求解符号变量。

### 1.4 angr中的符号执行

符号执行有两个基础：执行路径和符号。从这两者入手来学习。

#### 1.4.1 执行路径

一条执行路径代表程序从某处开始执行，再到某处结束的一个执行流程。在`angr`中，**执行路径的节点**由**`SimState`对象**表示，它存储程序的状态以及之前状态的历史。**将这些`SimState`连接起来就会创建一条执行路径**。

但是我们关注的不仅仅是一条路径，而应该关注多条路径以搜索到我们需要的路径。**在`angr`中如何表示以及构建一组路径呢？**

`angr`存储和操作给定程序的一组路径是通过**`Simulation Manager（SM）`**对象完成的。`SM`提供了函数去遍历程序以生成可能的路径和状态。**生成一组路径**的过程如下：

1. `angr`可以在指定的位置启动程序，此时是第一个`active`状态；
2. 在每个`active`（未终止）状态下执行指令，直到到达分支点或者该状态终止；
3. 在每个分支点处，将状态分为多个状态，并将它们添加到`active` 状态集合中；
4. 重复第2和第3步，直到找到我们想要的，或者所有状态都终止。

![SymbolicExecution-part1](https://bbs.kanxue.com/upload/attach/202101/830671_T38R59JNY6KZQB3.jpg)

生成了路径，那么**其中如何搜索我们想要的呢**？在生成路径过程中，搜索是否有哪个`active`状态符合我们的条件，这个条件可以是指令的地址，也可以是任何确定我们是否已达到所需状态的函数。

关于执行路径，符号执行中有一个很大的问题，就是**状态爆炸（State Explosion）**。对于每个`if`语句，分支路径可能会加倍，路径的增长与程序的“大小”呈指数关系。当前并没有非常好的解决方案。

![image-20201230111316712](https://bbs.kanxue.com/upload/attach/202101/830671_5FW2H33Z2FVFYY3.jpg)

一个好的方案是：如果可以确定继续运行不太可能到达成功状态的条件，则可以立即终止那部分路径并舍弃状态图的大部分。至于如何确定可能会失败的状态，可以使用各种启发式算法，比如：Veritesting。

![image-20201230111619428](https://bbs.kanxue.com/upload/attach/202101/830671_DN55R8FU4GHPUHT.jpg)

接着，我们小结一下搜索和避免路径的方法

- 1.加载二进制
- 2.指定**起点**并创建**`Simulation Manager（SM）`**
- 3.当我们还**没有找到**想要的状态的时候，对以下操作进行循环：
  - 3.1 一步一步进入所有`active`状态
  - 3.2 在每个`active`状态下运行**`should_accept_state`**谓词
    - 如果有状态接受，我们就**找到了想要的状态**！退出循环
  - 3.3 在每个活动状态下运行**`should_avoid_state`**谓词
    - 对于每个接受的状态，将其标记为**终止（termination）**
  - 3.4 从`active`状态集中**删除**所有标记为终止的状态

最后，我们看看`angr`中关于这部分的函数：**`explore`函数**。它会将所有符合条件的状态放到`simulation.found`列表中。

|  |  |
| --- | --- |
| 1 | `Simulation.explore(find=` `should_accept_path, avoid=` `should_avoid_path)` |

另外，搜索或者避开一个特定指令的地址很常用，所以，`find`和`void`参数都可以是地址：

|  |  |
| --- | --- |
| 1  2 | `Simulation.explore(find=` `0x80430a`, avoid=` `0x9aa442`)`  `# 目标搜索地址是0x80430a，并会终止任何会到达0x9aa442的状态` |

#### 1.4.2 符号和约束

当用户输入是从标准输入（`stdin`）中获得，比如简单调用`scanf`函数的时候，`angr`会自动注入符号进行符号化；然而，当用户输入更复杂，比如`scanf`函数的格式化字符串很复杂，或者从文件、网络或者UI中获取输入的时候，`angr`则不会自动注入符号，这时我们需要手动在需要的地方进行注入符号。

**`angr`中的符号是由`bitvector`（位向量）表示的**。位向量具有大小（`size`），即它们的位数。位向量最常表示的数据是`n`位整数或字符串。位向量和典型变量之间的区别在于：典型变量仅仅存储一个值，但是位向量存储满足某些约束的每个值。比如：设位向量`λ`为8位，并受以下约束：

![image-20210101230353061](https://bbs.kanxue.com/upload/attach/202101/830671_5YZ7QDQV2C3DQWG.jpg)

上面是位向量的存储方式。但是，如果我们要具体化位向量，则它可以取以下任意值：2、4或1。

> 按我自己的理解，这个位向量应该是这样：[00010110]。如有错误欢迎指正^_^。

**关于位向量的一些定义**：

- 具体的位向量（a concrete bitvector）：一个可以正好取1个值的位向量。
  - 例如：{`λ`：`λ= 1`}
- 符号位向量：可以采用多个值的位向量。
  - 例如：{`λ`：`λ> 10`}
- 不能满足的位向量：不能采用任何值的位向量。
  - 例如：{`λ`：`λ= 10`，`λ≠10`}
- 无约束的位向量：在向量范围内可以采用任何值的位向量

`angr`为开源约束求解器`z3`提供了一个不错的前端，用它来**求解符号约束**。它具有如下功能（不止）：

- 查找一个位向量的任何（单个）值
- 查找一个位向量的`n`个可能值
- 查找一个位向量的最大或最小值
- 确定位向量是“ true”还是“ false”
- 确定位向量是否可满足

只要位向量的大小等于变量的大小，我们就可以把符号注入变量中。引擎在执行给定路径的时候，会自动生成约束（例如，对于下面的程序，如果要执行`Success`那条路径， `λ= 'hunter2'`，或者对于另外一条路径就是 `λ` 不等于 `'hunter2'`）。如果需要，可以在程序执行期间的任何时候手动将约束添加到任何位向量。

|  |  |
| --- | --- |
| 1  2  3  4  5 | `user_input` `=` `λ`  `if` `user_input` `==` `'hunter2'`:  `print` `'Success.'`  `else`:  `print` `'Try again.'` |

前面学习了符号的注入和表示，接下来我们看看**符号的传播（Symbol Propagation）**，直接用一个例子来学习。

|  |  |
| --- | --- |
| 1  2  3  4 | `user_input` `=` `λ`  `encrypted_input0` `=` `user_input` `-` `3`  `encrypted_input1` `=` `encrypted_input0` `+` `15`  `encrypted_input2` `=` `encrypted_input1` `*` `7` |

下图代表内存，其中变量`user_input`和`encrypted_inputX`都已经标注；所有符号都用绿色表示。很明显，变量`encrypted_inputX`的值完全由`user_input`决定，如箭头所示。传输值的时候，符号也可以传播。

![image-20201230172250159](https://bbs.kanxue.com/upload/attach/202101/830671_MZQ22U92XKJHCCD.jpg)

约束也是可以随着符号值进行传播的（Constraint Propagation）。如果添加约束`λ= 10`，那么我们可以求解出`encrypted_input2`。

|  |  |
| --- | --- |
| 1  2  3  4 | `user_input` `=` `λ` `=` `10`  `encrypted_input0` `=` `user_input` `-` `3` `=` `10` `–` `3` `=` `7`  `encrypted_input1` `=` `encrypted_input0` `+` `15` `=` `7` `+` `15` `=` `22`  `encrypted_input2` `=` `encrypted_input1` `*` `7` `=` `22` `*` `7` `=` `154` |

而逆过来的话，即约束逆向传播（Constraint Reverse-Propagation）也是当然可以的，比如我们给传播的符号值添加约束：`encrypted_input2 = 14`，那么可以求解出`λ= -10`。也就是我们通过给定的结果求解出了初始条件。

|  |  |
| --- | --- |
| 1  2  3  4 | `encrypted_input1` `*` `7` `=` `encrypted_input2` `=` `14``, encrypted_input1` `=` `2`  `encrypted_input0` `+` `15` `=` `encrypted_input1` `=` `2``, encrypted_input0` `=` `-``13`  `user_input` `-` `3` `=` `encrypted_input0` `=` `-``13``, user_input` `=` `-``10`  `λ` `=` `user_input, λ` `=` `-``10` |

前面对于执行路径，符号执行有个大问题就是路径爆炸。那么对于符号，这里也有个大问题，那就是**符号执行依赖于解决复杂的约束系统**——约束满足问题是NP完全问题。

## 2. 00_angr_find

在`angr_ctf`项目的`solutions`目录下都给出了解题脚本，在注释中也对这些文件进行了说明：

- 二进制文件
  - 每个二进制文件在运行时都会要求输入密码，可以通过`stdin`输入该密码（将其输入控制台）。目标是找到一个适用于每个二进制文件的密码。
  - 如果输入的密码不正确，程序将打印“Try again.”。如果你输入正确的密码，程序将打印“ Good Job.”。
- `scaffoldXX.py`
  - 每个挑战都有`scaffoldXX.py`，它就是个脚手架（解题模板）。它会提供指导以及可能的解决方案。
- `solve.py`
  - 解题脚本（需要修改一下，里面用的是`python2`，而环境是`python3`）

### 步骤一：查看文件属性和检查安全机制

- 32位程序
- 开启了`Canary`和`NX`，关闭了`PIE` ，那么`code`段地址不会改变。

![image-20201225142852004](https://bbs.kanxue.com/upload/attach/202101/830671_ZUJX5CJKQ2AQY48.jpg)

### 步骤二：IDA静态分析

![image-20201225153746191](https://bbs.kanxue.com/upload/attach/202101/830671_YSEEFU5RZVDENE4.jpg)

![image-20201225153808446](https://bbs.kanxue.com/upload/attach/202101/830671_R5E6G92V7UF3VPH.jpg)

- `puts("Good Job.")` 地址：

![image-20201229164546525](https://bbs.kanxue.com/upload/attach/202101/830671_VFHGPKPE7HK3YXS.jpg)

> IDA graph界面中指令前显示地址的方法：Options->General->Disassembly->勾选 Line prefixes(graph)

### 步骤三：使用angr解题

解题脚本：修改一下原有脚本，把`python2`改成`python3`。

|  |  |
| --- | --- |
| 1 | `Simulation.explore(find=` `should_accept_path, avoid=` `should_avoid_path)` |

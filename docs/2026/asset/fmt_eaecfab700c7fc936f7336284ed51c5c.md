
# [原创]符号执行---KLEE论文阅读笔记与详细安装过程

论文《KLEE: Unassisted and Automatic Generation of High-Coverage Tests for Complex Systems Programs》的阅读笔记如下

## 2.1 KLEE的用法

KLEE的使用：用户用公开的GNU C的LLVM编译器去编译成字节码

```
llvm-gcc --emit-llvm -c tr.c -o tr.bc
```

然后KLEE运行字节码文件。

```
klee --max-time 2 --sym-args 1 10 10
--sym-files 2 2000 --max-fail 1 tr.bc
```

`max-time`：测试的分钟数

`--sym-args 1 10 10`：表示使用0至3个命令行参数，第一个参数是一字符长，剩下两个是10字符长。

`--sym-files 2 2000`：表示使用标准输入和一个文件，每个数据包含2000字节的符号数据。

`--max-fail 1`：选项表示每个程序路径最多一个系统调用失败

示例程序
```
1 : void expand(char *arg, unsigned char *buffer) { 8
2 : int i, ac; 9
3 : while (*arg) { 10*
4 : if (*arg == '\') { 11*
5 : arg++;
6 : i = ac = 0;
7 : if (*arg >= '0' && *arg <= '7') {
8 : do {
9 : ac = (ac << 3) + *arg++ - '0';
10: i++;
11: } while (i<4 && *arg>= '0' && *arg<= '7');
12: *buffer++ = ac;
13: } else if (*arg != '\0')
14: *buffer++ = *arg++;
15: } else if (*arg == '[') { 12*
16: arg++; 13
17: i = *arg++; 14
18: if (*arg++ != '-') { 15!
19: *buffer++ = '[';
20: arg -= 2;
21: continue;
22: }
23: ac = *arg++;
24: while (i <= ac) *buffer++ = i++;
25: arg++; /* Skip ']' */
26: } else
27: *buffer++ = *arg++;
28: }
29: }
30: . . .
31: int main(int argc, char* argv[ ]) { 1
32: int index = 1; 2
33: if (argc > 1 && argv[index][0] == '-') { 3*
34: . . . 4
35: } 5
36: . . . 6
37: expand(argv[index++], index); 7
38: . . .
39: }
```

第18行有缓冲区溢出，

## 2.2 KLEE符号执行的具体过程。

1.命令行运行

```
klee --max-time 2 --sym-args 1 10 10
--sym-files 2 2000 --max-fail 1 tr.bc
```

2.首先会遇到第33行的分支，会把`argc>1`和`argc<=1`约束用STP约束求解器求解，然后发现两个约束都可以，然后会生成两个执行路径。

3.有多个执行路径的话，KLEE要选择一条路径线去执行。（详细的算法会在3.4节讲）这里我们假设会往到达bug的路径执行，然后后续会添加关于5个`argv`的约束（分别在代码的33，3，4，15行）。

4.在代码17，18行中，`arg`自增了两次，导致了越界。

5.测试用例

```
[ "" ""
```

会触这个错误。这是三个字符串。

# 3.The KLEE Architecture

KLEE是一个基于之前符号执行系统EXE的更完整的一个工作。KLEE更像是一个为符号进程的设计操作系统和解释器的混合体。每个符号进程都有寄存器、栈、堆、程序计数器、路径条件（path condition)。为了不与Unix中进程的概念混淆，我们把KLEE中的符号进程称为`state`(状态)。程序被编译成LLVM汇编语言（RISC虚拟指令集），然后KLEE解释这些指令，并将指令映射到约束（不包含近似，如符号浮点，longjmp,threads，and assembly code）。

## 3.1 Basic architecture

在任何时候，KLEE都可能正在执行大量的状态。KLEE的核心是一个解释器循环，它选择一个要运行的状态，然后在该状态的上下文中符号化地执行一条指令。此循环将持续，直到没有剩余状态，或达到用户定义的超时。与普通进程不同，状态寄存器、堆栈和堆对象的存储位置指的是表达式（树），而不是原始数据值。表达式的叶子节点是符号变量或常量，内部节点是LLVM汇编语言的操作指令（例如，算术操作、按位操作、比较和内存访问）。存储常量表达式的存储位置是具体的。大部分指令的符号性执行都是很简单的。例如，要符号执行下面的LLVM加法指令：

```
%dst = add i32 %src0, %src1
```

KLEE会从`%src0`寄存器和`%src1`寄存器取回加数，然后写入一个新的表达式`Add(%src0, %src1)`到`%dst`寄存器。为了提高效率，构建表达式的代码会检查是否所有给定的操作数都是具体的（即常数），如果是，则在直接执行该操作，并返回一个常量表达式。即如果操作数是1和2则返回3给`%dst`寄存器。

条件分支采用一个布尔表达式（分支条件），并根据该条件是为真还是false来更改该状态的指令指针。KLEE会去调用约束求解器去求解这个表达式，以确定分支条件在当前路径上是可满足的还是不可满足的；如果有一条路径是不可满足的，则指令指针将更新到适当的位置。否则，如果这两个分支都是可能的：KLEE克隆状态，以便它可以探索这两条路径，并且更新每个路径上的指令指针和路径条件。

存在潜在危险的操作指令会隐式地生成分支去检查是否存在可能导致错误的输入值。例如，一个除法指令生成一个检查零除数的分支。这些分支与正常分支相同。当检查成功，即检测出错误的时候，会继续执行条件为假的路径，也就是添加相反的约束，比如使得除数不为0的约束。此外，还会为错误生成测试样例，并且终止检测出错误的状态。

与其他危险操作一样，加载和存储指令也会生成检查：在这种情况下，要检查地址是否在有效内存对象的范围内。然而，加载和存储操作还存在额外的复杂性。所选代码所使用的内存的最直接的表示方式将是一个平面字节数组。在这种情况下，加载和存储会简单的映射成数组读表达式和写表达式。不幸的是，我们的约束求解器STP几乎永远无法解决由此产生的约束（而且我们所知的其他约束求解器也不能）。因此，与EXE中一样，KLEE将检查代码中的每个内存对象映射到不同的STP数组（在某种意义上，将平面地址空间映射到分段地址空间）。这种表示法显著提高了性能，因为它允许STP忽略未被给定表达式引用的所有数组。

许多操作比如边界检查，对象级的copy-on-write）需要对象特定的信息。如果一个指针指向很多对象，这些操作就很难实现。为了简化实现，KLEE用以下方式回避了这个问题。当一个解引用的指针`p`指向N个对象时，KLEE复制当前状态N次。在每个状态，KLEE限制指针`p`在其对象的边界内，然后实现读或写操作。虽然这个方法对于有很大points-to 集合的指针的开销很大，但实际情况中，大部分的符号指针只指向1个对象，所以这个优化还是可以的。（没太看懂，为什么一个指针会被指向很多个对象？？）

## 3.2 Compact state representation（紧凑状态表示）

在实践中，状态的数量增长得相当迅速：通常即使是小程序也会在interpretation的最初几分钟内产生数万甚至数十万的并发状态。当我们运行具有1GB内存上限的重复使用时，记录的并发状态的最大数量为95,982个（对于敌对状态），每个工具的这个最大值的平均值为51,385个。这次爆炸使状态大小至关重要。

由于KLEE保存所有内存对象，因此它可以在对象级别（而不是页面粒度）上实现对写的复制，从而大大减少了每个状态的内存需求。通过将堆实现为不可变映射，堆结构的一些部分也可以在多个状态之间共享(类似于`fork()`的共享部分页表)。此外，这个堆结构可以在恒定的时间内被复制，考虑到这个操作的频率，这很重要。

## 3.3 Query optimization（查询优化）

约束求解的时间代价是最高的，因为KLEE以Np完全的逻辑生成约束。因此，我们花了大量的精力在简化表达式的技巧上，并在查询到达STP之前理想消除查询（没有查询是最快的查询）。简化的查询可使解决速度更快，减少内存消耗，并提高查询缓存的命中率（见下文）。主要的查询优化方法包括：

**表达式重写**：这是最基本的优化，跟编译器的优化类似，如

```
x+0改成x,
x*2^n改成x<<n
2*x-x改成x
```

**约束集简化**：符号执行通常涉及到在路径条件中添加大量的约束。程序的自然结构意味着对相同变量的约束往往会变得更加具体。例如，通常会添加一个不精确的约束，如

```
x<10
```

，然后在一段时间后又会添加约束

```
x=5
```

当向约束集添加新的等式约束时，KLEE通过重写以前的约束来积极地简化约束集。在本例中，将第一个约束简化为true。

**隐式的值具体化**：例如当一个约束

```
x+1=10
```

加入路径条件时，它已经隐式的表面x等于9了，这时候把这个具体的9写入内存，后续的访问这个x时可以返回一个具体的常量表达式。

**约束的独立性（名字取自EXE）**：根据约束集所引用的符号变量，将约束集划分为不相交的独立子集。通过显式地跟踪这些子集，KLEE可以在向约束求解器发送查询之前消除不相关的约束。例如，给定约束集

**Counter-example Cache**：冗余查询非常频繁，而一个简单的缓存可以有效地减少大量的冗余查询由于约束集的特定结构，需要构建一个更复杂的缓存。Counter-example Cache将约束集映射counter-examples（即变量分配），以及在一组约束没有解决方案时使用的特殊哨兵。这个cache存储在一个自定义数据结构中——该结构来自Hoffmann和Hoehler[28]提出的UBTree结构，它能有效地搜索约束集的子集和超集的缓存条目。通过以这种方式存储缓存，counter-examples有三种方式减少查询。

例如，我们假设 counter-example cashe有下面两个条目

```
{i<10，i-10}(没有解)
{i<10，j=8}(有解，例如当分配i=5,j=8)
```

1.当一个约束集的子集没有解的时候，则这个约束也没有解。在一个没有解的约束集中增加约束不能使其变成有解的。例如：

```
{i < 10, i = 10, j = 12}（依然是无解的）
```

2.当约束集的超集有一个解时，该解也满足原始约束集。从约束集中删除约束并不会使该约束集的解无效（不是很理解）。例如

```
i→5，j→8分别是i<10或j=8的解
```

3.当约束集的一个子集有一个解时，这很可能也是该约束集的一个解。因为额外的约束通常不会使子集的解无效。因为找一个潜在的解开销不大，所以KLEE尝试将所有的解替换为该约束集的子集的解，如果找到了，则返回一个令人满意的解。例如，

```
约束集{i<10，j=8，i！=3}的解还是i→5，j→8。
```

查询i是否等于20只需要前两个约束。

**实验结果**：正如预期的那样，独立性优化本身并没有消除任何查询，但它所执行的简化使总体运行时间减少了近一半（45%）。counter-example cashe将运行时间和STP查询的次数都减少了40%。然而，当两种优化都启用时，真正的胜利来了；在这种情况下，由于查询首先通过独立性简化了，counter-example cashe的命中率大大提高。对于示例运行应用程序-STP查询的平均数减少到原始数量的5%，平均运行时减少了超过一个数量级。同样值得注意的是，STP时间（用于解决查询的时间）在运行时的主导程度。对于原始运行，STP平均占总执行时间的92%（组合优化减少了近300%）。启用两种优化后，此百分比下降到41%。最后，图2显示了KLEE优化的有效性随着时间的推移而增加——随着反例缓存被填充和查询大小的增加，优化的速度也会增加。

## 3.4 state scheduling

KLEE会在每条指令执行时，通过交替使用两个启发式算法选择一个state。

### 3.41 Random Path Selection

这个方法存有一棵二叉树，叶子节点是当前的state,内部节点是之前执行的路径。state会在各个分支中随机选择出来。因此，当到达一个分支点时，每个子树中的状态集被选择的概率相同，而不管它们的子树的大小如何。

这个方法偏爱在分支中的较高的state。因为这些states对其符号输入的约束较少，因此有更大的自由来获取未覆盖的代码。第二，可以避免饥饿（当程序进入包含符号约束的循环时，会有路径fork爆炸）。

### 3.42 Coverage-Optimized Search

覆盖-优化搜索试图选择可能在不久的将来覆盖新代码的状态。它使用启发式方法计算每个状态的权值，然后根据这些权值随机选择一个状态。目前，这些启发式方法考虑了到未公开指令的最小距离、状态的调用堆栈以及该状态最近是否覆盖了新代码。

KLEE以循环的方式使用每种策略。虽然这可以增加一个特别有效的策略来实现高覆盖率的时间，但它可以防止个别策略陷入困境的情况。此外，由于策略从同一个状态池中选择，交错它们可以提高整体有效性。

# 4 Environment Modeling环境建模

## 在虚拟机Ubuntu20.04中安装KLEE

### 安装一些必备工具

```
sudo apt-get install build-essential curl libcap-dev git cmake libncurses5-dev python-minimal python-pip unzip libtcmalloc-minimal4 libgoogle-perftools-dev libsqlite3-dev doxygen
```

```
sudo apt-get install python3 python3-pip gcc-multilib g++-multilib
```

安装了pip3后发现用Pip3去安装东西出现找不到Pip3的问题，先找到Pip3的安装路径。

```
bigeast@ubuntu:~$ locate pip3
/usr/local/lib/python3.6/dist-packages/bin/pip3
/usr/local/lib/python3.6/dist-packages/bin/pip3.6
/usr/share/man/man1/pip3.1.gz
```

然后创建软链接到bin目录下

```
sudo ln -s /usr/local/lib/python3.6/dist-packages/bin/pip3.6 /usr/local/bin/pip3
```

即可使用pip3来安装ellvm

```
pip3 install lit tabulate wllvm
```

```
sudo apt-get install clang-9 llvm-9 llvm-9-dev llvm-9-tools
```

### 安装Z3约束求解器

```
git clone https://github.com/Z3Prover/z3.git
cd z3/
python scripts/mk_make.py
sudo make install
```

### （可选）构建 uClibc 和 POSIX 环境模型（macOS 不支持）、

默认情况下，KLEE 适用于封闭程序（不使用任何外部代码的程序，例如 C 库函数）。但是，如果您想使用 KLEE 来运行实际程序，您将需要启用 KLEE POSIX 运行时，它构建在uClibc C 库之上。

```
bigeast@ubuntu:~$ git clone https://github.com/klee/klee-uclibc.git
bigeast@ubuntu:~$ cd klee-uclibc
bigeast@ubuntu:~/klee-uclibc$ ./configure --make-llvm-lib
INFO:Disabling assertions
INFO:Configuring for Debug build
INFO:Configuring for LLVM bitcode archive
INFO:Using llvm-config at...None
ERROR:llvm-config cannot be found
```

报错，把命令改成如下手动配置路径即可。

```
./configure --make-llvm-lib --with-llvm-config /usr/bin/llvm-config-9
```

### （可选）获取 Google 测试源：

对于单元测试，我们使用 Google 测试库。如果要运行单元测试，则需要执行此步骤，并-DENABLE_UNIT_TESTS=ON在步骤 8 中配置 KLEE 时传递给 CMake。

我们现在依赖一个版本，1.7.0所以获取它的来源。

```
curl -OL https://github.com/google/googletest/archive/release-1.7.0.zip
unzip release-1.7.0.zip
```

### 安装gcc9

```
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt-get update
sudo apt-get install gcc-9 g++-9
```

在klee-2.1目录下

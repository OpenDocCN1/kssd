
## 2.7.1 PST

`Harpoon`，构建`PST`树进行控制流分析

## 2.7.2 RPST

`abcd`，java开源反编译器，用支配树来建立`RPST`树，好像并不能称为构建`RPST`树，只能称为构建控制树。

### 2.8 Enhanced Region Structural Analysis

传统结构化分析的扩展，传统结构化分析只能处理只包含`SESE(Single-Entry-Single-Exit)`区域的流图，扩展结构化分析新定义了`SESS(Single-Exit-Single-Successor)`区域类型，好像能处理部分流图是`Multi-Exit`区域类型的情况

#### 2.8.1 Type 1

可以从论文"`Enhanced Structural Analysis for C Code Reconstruction from IR Code`"学习了解

#### 2.8.2 Type 2

可以从论文"`Native x86 Decompilation using Semantics-Preserving Structural Analysis and Iterative Control-Flow Structuring`"了解学习

### 2.9 Normalized Region Structural Analysis

因为区间分析或结构化分析只能处理只包含`SESE(Single-Entry-Single-Exit)`区间类型的流程图，因此需要把原来的控制流图转化成只包含`SESE (Single-Entry-Single-Exit)`区间类型的流程图，当前主要有以下两种方法：

#### 2.9.1 Type 1

可以从论文"`Emscripten: An LLVM-to-JavaScript Compiler`"学习了解。

#### 2.9.2 Type 2

可以从论文"`No More Gotos: Decompilation Using Pattern-Independent Control-Flow Structuring and Semantics-Preserving Transformations`"学习了解

### 2.10 Elimination

反编译器的控制流分析，是结构化分析，是基于控制树的控制流消去法，有些类似于数据流分析。

#### 2.10.1 Normalized

反编译器的流图的区间类型有四种类型：`SESE(Single-Entry-Single-Exit)`、`SEME(Single-Entry-Multi-Exit)`、`MESE(Multi-Entry-Single-Exit)`、`MEME(Multi-Entry-Multi-Exit)`。为了能够进行结构化分析，需要把原来的流图都转化成只包含`SESE(Single-Entry-Single-Exit)`区间的流程图。

当前有三种方法：第一种，构建`RPST`树，然后进行结构化分析；第二种，控制流平坦化，然后用`Emscripten`实现的`ReLooper`算法进行控制流分析；第三种，使用论文"`No More Gotos: Decompilation Using Pattern-Independent Control-Flow Structuring and Semantics-Preserving Transformations`"的方法进行结构化分析。

#### 2.10.2 Structuring Acyclic Regions

如何最小化控制流图重建后`goto`语句的数量的大小，无环区域的简化和优化的实现非常关键。

- `IF Regions`

条件跳转区域简化和合并主要有以下两种方法:

- 一、`Rules Matching`

根据规则进行模式匹配，开始进行类型归类时有些想当然了，如何用规则匹配进行`If`语句合并将在另一篇"`if-else分支的识别和合并`"说明

- 二、`Path Finding`

根据规则匹配归约合并条件语句的方法有最大的缺陷：并不能覆盖所有的规则，由于二进制代码经过编译器优化，代码混淆的原因，匹配的规则数量是不确定的，则控制流图重建后不可避免有`goto`语句。

因此，需要考虑一种不使用规则模式匹配的方法来进行条件归约合并，

可以使用类似于符号执行所使用的`Path Finding`方法进行条件语句归约合并，论文"`No More Gotos: Decompilation Using Pattern-Independent Control-Flow Structuring and Semantics-Preserving Transformations`"中所使用的条件归约合并方法类似于`Path Finding`，论文有些晦涩，并不容易理解，有些类似于数据流分析自顶向下（`Top-Down`）向前问题（`Forward`）的到达-定值(`Reaching Definitions`)。

- `Switch Regions`

`Switch Regions`三种表现形式：位计算，跳转表，`If`跳转或二叉树`If`跳转。

控制流重建时主要考虑跳转表，需要考虑两种特殊的情况：`case`块没有用`Break`语句，直接`Fallthrough`到下一个`case`块；`case`块没有用`Break`语句，而是使用`Return`语句。

- `Return/Finish Regions`

某一无环区域有多个`Return/Finish`(`Finish`块通常用于资源释放)，编译优化后二进制代码可能会共用一个`Return/Finish`块，结构化分析时需要复制`Return/Finish`块。

- `Other Regions`

`C++`异常和`C`运行库函数`setjump()`/`longjump()`所代表的区域，暂时不考虑。

#### 2.10.3 Structuring Cyclic Regions

有环区域极简化和合并有两种情况：第一种，有环区域`Loop`和无环区域`Seq`相互转化；第二种，有环区域`Loop`三种归约类型的规则匹配：`While`、`DoWhile`、`For`。

对于有环区域`Loop`，需要考虑两种特殊情况：第一种，前一个区域是多个`Return/Finish(Multi-Exit)`区域，假设前一个区域不进行复制`Return/Finish`块，则结构化分析归约后当前`Loop`区域的`If`条件语句将带有多个前一个区域`Return/Finish`块的前驱`If Condition`条件语句的反转`not(Condition)`语句；第二种，当前`Loop`循环体内部多个`Return/Finish(Multi-Exit)`区域。这两种类型都会使得控制流重建后不可避免有`goto`语句，并且可读性差，大部分的反编译器是没有考虑这种情况的，比如`Hex-Rays Decompiler`就没有考虑这种情况。

## 2.11 Decompiler Paper

已经公布存在的闭源和开源反编译器相关的论文

#### 2.11.1 Type 1

`DREAM++`，相关论文"`No More Gotos: Decompilation Using Pattern-Independent Control-Flow Structuring and Semantics-Preserving Transformations`"，还有相关后端优化的论文"`Helping Johnny to Analyze Malware A Usability-Optimized Decompiler and Malware Analysis User Study`"

`DREAM`，相关论文"`No More Gotos: Decompilation Using Pattern-Independent Control-Flow Structuring and Semantics-Preserving Transformations`"

`Hex-Rays Decompiler`，相关论文"`Decompilers and Beyond White Paper`"，还有相关中端IR的PPT `"Decompiler internals: microcode"`

`Phoenix`，相关论文"`Native x86 Decompilation using Semantics-Preserving Structural Analysis and Iterative Control-Flow Structuring`"

`Mirtoc`，相关论文"`Enhanced Structural Analysis for C Code Reconstruction from IR Code`"

#### 2.11.2 Type 2

`fcd`，开源，相关论文"`No More Gotos: Decompilation Using Pattern-Independent Control-Flow Structuring and Semantics-Preserving Transformations`"

`reko`，开源，相关论文"`Native x86 Decompilation using Semantics-Preserving Structural Analysis and Iterative Control-Flow Structuring`"

`Emscripten`，开源，相关论文"`Emscripten: An LLVM-to-JavaScript Compiler`"

`dcc`，开源，相关论文"`Reverse Compilation Techniques`"

`axtor`，开源，相关论文"`Decompilation of LLVM IR`"

`Boomerang`，开源，相关论文"`Structuring Assembly Programs`"还有变量和类型重建的相关论文"`SSA for Decompilation`"

`BAP-0.8`，开源，相关论文"`Structural Analysis: A New Approach to Flow Analysis in Optimizing Compilers`"

`SmartDec/Snowman`，开源，相关论文"`SmartDec: Approaching C++ Decompilation`"

`RetDec`，开源，相关论文"`Static Decompilation of Unobfuscated Native Code`"

## 总结

主流的重新构建控制流图的方法都是对控制流程图进行结构化分析生成控制树，然后尝试将函数流程图的可归约的一小部分与已知模式匹配，并在不起作用时使用`goto`语句。

因此，反编译二进制程序时控制流结构恢复后`goto`语句的数量是衡量反编器表现的一个重要标准。然而，论文"`No More Gotos: Decompilation Using Pattern-Independent Control-Flow Structuring and Semantics-Preserving Transformations`"开发了一种称为模式无依赖的控制流结构分析新技术，它重新构建控制流图时没有使用任何预定义模式，控制流结构恢复后没有`goto`语句。

综上所述，实现没有`goto`语句的`C`语言反编译器是可能的，该反编译器重建流图的大概操作简述如下:

- `Stage 1`

构建`RPST`树，确保控制流图只包含`SESE(Single-Entry-Single-Exit)`区域类型。

- `Stage 2`

结构化分析，在控制流图中查找区域，生成控制树。

- `Stage 3`

路径查找(`Path Finding`)，计算每个块的到达条件。

- `Stage 4`

简化和合并控制流语句。

## 附录

`fcd`实现相关文章：

1. `The Region Problem`
2. `The Loop Problem`

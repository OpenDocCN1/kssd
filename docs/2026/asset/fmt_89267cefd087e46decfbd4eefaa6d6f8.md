
stage can be attacked by obfuscating the `call` instruction `C`, say, by using a combination of the `push` and `jump` instructions.

由于过程是大多数分析算法的基本单位，所以病毒作者会设法使得识别二进制程序中的过程更加困难从而挫败静态分析。如果分析不能够确定过程边界，就可能被攻击。比如，使用进栈指令和跳转指令的一个组合，就可以混淆调用指令而实现对这个阶段的威胁。

There is also no sanctity in the tradition of placing the code of a procedure in contiguous memory locations. In fact, this is just a tradition followed by compilers. There is no guarantee that hand-written assembly programs follow this tradition. The virus `Win32.Evol` is a classic example. Use IDA Pro to identify its procedures and you'd find that it beats the heuristic used by IDA Pro.

把过程代码放在连续的存储区域也不是绝对的规矩。事实上，这只是编译器的习惯做法。并不保证手写的汇编程序也保持这个传统。病毒`Win32.Evol`就是个典型的例子。使用IDA Pro鉴别它的过程，你将发现该病毒挫败了IDA Pro所用的启发式。

There is no necessity for such mangled code to be hand-written. It will not be too hard to modify a compiler such as `gcc` so that it mangles the code for a procedure into non-contiguous blocks. Or such mangling can be performed automatically after an executable is created.

这类代码碎片不需要手工编写。修改象`gcc`这类的编译器不会太难，可以改变成把一个过程切碎放到不连续的块。或者也可以等到可执行文件生成之后再自动化切碎。

## 3.3 Attack on control flow graph generation（对控制流程图生成的攻击）

The construction of CFG can be attacked by fooling the CFG generation algorithm in creating redundant edges in the CFG. The creation of extra edges may jeopardize the precision of the analysis in the later stages. The CFG generation process can also be attacked by obfuscating the assembly code such that one cannot determine the correct target of a `jump` instruction, such as using a `jump` through register.

通过在CFG中制造冗余边欺骗CFG生成算法，从而使CFG的建立受到威胁。多余边的产生可能危害以后阶段分析的准确性。攻击CFG生成进程也可以是通过混淆汇编代码以致它不能够正确地确定跳转指令的目标，比如使用寄存器指示跳转目标。

One method for resolving the targets is to assume that the `jump` will transfer control to every instruction (or to every block). This assumption, though safe for various static analyses for compiler optimizations, could spell doom for an anti-virus technology by increasing the time and space costs of the analyses.

解析这些跳转目标的一个方法是，假定跳转会把控制传送给每条指令（或每个指令块）。这个假定，虽然对编译器最优化使用的各种静态分析来说是安全的，但是由于增加了时间和空间的代价，会致反病毒技术于不利处境。

## 3.4 Attack on data flow analysis（对数据流分析的攻击）

An example of attack on the flow analysis stage was discussed in the section entitled 'promises of formal analyses'. Most flow analysis algorithms propagate sets of data from one node of a CFG to another. When data flows into a node from two different predecessors, the two sets of data are merged to create a single set. In the process of merging the data some information is lost, leading to an imprecise analysis.

在‘形式分析的承诺’这个小节中讨论了一个攻击流分析阶段的例子。大多数的流分析算法从CFG的一个结点到另一个结点传送数据组。当数据从两个不同的前趋流入一个结点的时候，两组数据被合并产生一个单个组。在合并数据的处理中，一些信息被遗失，导致不精确的分析。

The data flow analysis stage can be attacked by moving some computation from a block (of a CFG) into the preceding nodes and obfuscating the computation along each path. The analysis can also be attacked by using data that resides outside the scope of the program, such as in the areas managed by operating system. The known constant values in these areas may be used in the program for computation, thus thwarting accurate analysis. For instance, `Win32.Evol` and other viruses utilize knowledge of the specific address where `kernel32.dll` is loaded. The same addresses could also be used to access constant data from `kernel32.dll` code.

通过把(CFG的)一个语句块中一些计算移到前面的结点而且沿每条路径混淆计算，能够攻击数据流分析阶段。对该分析的攻击也能够藉由使用驻存在程序作用域以外的数据，比如在操作系统所管理的区域中。这些区域中已知的常数数值可被用于程序中的计算，以此阻挠正确分析。例如，`Win32.Evol`和其他病毒利用了`kernel32.dll`被装入特定地址的这一知识。也可能使用这些相同地址从`kernel32.dll`代码里面存取常数数据。

## 3.5 Attack on property verification（对特性验证的攻击）

Property verification is carried out using theorem proving systems. Typically, these systems depend on human guidance to prevent them from getting into infinite loops. Completely automated theorem provers, such as model checkers, operate on an abstraction of the problem. Sometimes the abstraction itself may be so large that the theorem prover may take an inordinate amount of time and resources to complete the proof. Or else, the abstraction may throw away so much information that the theorem prover may yield results that are correct for the abstraction, but incorrect for the program being analysed.

特性验证使用定理求证系统来实现。典型地，这些系统依赖人类的引导防止它们进入无限循环。象模型检查程序这种完全自动化的定理证明程序是在问题的一个抽象上操作。有时抽象本身可能是很大的工作量，以致定理证明程序可能花费过多时间和资源来完成证明。要不然，抽象会丢掉那么多信息，以致于定理证明程序产生的结果对抽象而言是正确的，但是对于正在被分析的程序而言则是不正确的了。

A virus writer can attack the mechanism using the knowledge of how the theorem prover used in an anti-virus technology determines the existence of the atomic properties and how it composes atomic properties into more complex properties.

通过了解在反病毒技术中使用的定理证明程序如何确定原子特性的存在，以及它如何把原子特性组成更复杂的特性，病毒作者能够攻击这个机制。

For instance, consider an anti-virus tool that uses the presence of calls to system library as atomic properties. This AV tool may look at the address of the target of a `call` instruction to determine whether a system library function is being called. `Win32.Evol` will escape such a virus detector because it obfuscates calls to system library functions. Instead of using the `call` instruction, this virus uses the `return` instruction to make the call. Before the `return` instruction is executed, though, the address of the function to be called is pushed on the stack. If the analyser cannot determine whether a virus calls a specific library function, the analyser has two choices. One, assume that the program actually does not call the Win32 API, thus letting the virus pass through. Two, assume that the program does call the library, thus generating false positives for programs that actually do not call the API.

例如，设想有一个反病毒工具使用是否存在对系统程序库的调用作为原子特性。这个AV工具可能搜索调用指令的目标地址以判定是否有调用一个系统程序库函数。由于`Win32.Evol`混淆对系统程序库函数的调用，所以它能逃脱这类病毒检测程序。这个病毒不使用调用指令而是使用返回指令实施调用。不过在返回指令被执行之前，被调用的函数的地址被推入栈顶上。如果分析器不能够确定一个病毒是否调用某个特定的库函数，那么分析器有两个选择。一、假定程序实际上没有调用Win32 API，因此让病毒蒙过去。二、假定程序确实调用库，对于实际上没有调用API的程序则造成假阳性。

# 4 Conclusions（结论）

We have argued that formal analysis methods used by optimizing compilers and other programming tools, though they appear promising in the detection of metamorphic viruses, are not directly suitable for use in anti-virus technologies.

我们已经论证了，最优化编译器和其他编程工具所使用的形式分析方法，尽管在变形病毒检测中的应用前景乐观，但是在反病毒技术中直接使用它们是不合适的。

The common approach for analysing a binary consists of the following stages: assembly, procedure abstraction, control flow graph generation, data flow analysis, and property verification. Compiler optimization-based methods for each of these stages can easily be attacked. Thus, even if the processing in each stage was correct 90 per cent of the time, the overall system will be correct only 59 per cent of the time, which is pretty close to the results offered by flipping a coin.

分析一个二进制代码的通常方式有以下几个阶段：反汇编，过程抽象，控制流程图生成，数据流分析和签名验证。这每一阶段中基于编译器最优化的方法会被轻易地攻击。因此，即使在每一阶段中的处理有90%的正确率，整个系统也只会有59%的正确率，这近乎是掷硬币的结果。

In order to use these formal analysis methods in anti-virus technologies, we may have to rethink the whole process. Unlike in the context of optimizing compilers and other similar tools, the analysis tool and the programmer (virus writer) do not have a common objective. Hence, assumptions made by analysis methods used by such compilers can be exploited by the virus writer.

为了在反病毒技术中使用这些形式分析方法，我们可能必须重新考虑整个过程。与最优化编译器及其他类似工具的情况不同，分析工具和程序设计者（病毒作者）没有共同目标。因此，这些编译器使用分析方法的假设前提可能会被病毒作者利用

The good news is that a virus writer is confronted with the same theoretical limit as are anti-virus technologies. To keep ahead of anti-virus technologies, a metamorphic virus writer has to address some of the same challenges that AV technologies face. This is likely to have an effect on the pace at which new metamorphic transformations can be introduced. It may be worth contemplating how this could be used to the advantage of AV technologies.

好消息是，病毒作者和反病毒技术一样也遇到相同的理论限制。为了保持领先于反病毒技术，变形病毒作者必须处理与AV技术共同面对的某些难题。这可能影响到变形病毒花样翻新的速度。如何才能让它为AV技术所用，这是值得深思的。

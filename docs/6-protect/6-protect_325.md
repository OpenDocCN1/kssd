

# 变形的初步方案－－两篇变形文章的读后感

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-802.htm`](https://www.pediy.com/kssd/pediy08/pediy8-802.htm)

   

　　　　**变形的初步方案**  
　　　　　　　　　　　　－－两篇变形文章的读后感  
　　Mental Driller 的“Metamorphism in practice or "How I made MetaPHOR and what I've learnt"”一文(以下简称 M 文)所述变形引擎，全部代码虽然经过变形，但毕竟已经是机器语言了，有些行为如果处理不好，可能引起怀疑。如果直接使用宏语言编写程序并且携带，只把其中一部分模块翻译成可直接执行的机器语言，也许更好。宏命令可以相当于一条到三条指令组合，也可以只是个标记；可能有一个或几个参数，也可能没有。并且因此可以省略把机器语言反汇编成伪汇编语言这一部分，自定义的宏语言程序比较容易控制。这个想法一部分来自 Navrhar 的“Assembly Language or HLL”一文。  
　　宏程序包括入口模块、汇编模块、变形模块、传播模块、X 模块。可以放在 PE 文件的一个只读不执行的数据节。宏程序由宏命令组成，宏命令的基本内容有宏代号、参数个数、参数列表、和连接位置(指向下一条命令，其作用与 M 文中伪指令的 Pointer 不同)。其中翻译成机器语言的部分(入口模块和汇编模块)附加到 PE 文件的可执行不可写入的代码节(利用空白区或者新节)。  
　　入口模块取得控制权，并做些准备工作，需要的时候它启动汇编模块翻译出其他模块并执行。例如，汇编模块从数据节读宏程序，先翻译变形模块。然后执行变形模块把宏程序变形，这样，新一代的数据节就和上一代的不同，再从中翻译出的机器语言代码也就变形了。  
　　宏程序变形主要分两步：置换和收缩/展开。  
　　置换：把宏程序所有命令打乱重新随机排列，命令中的连接位置字段内容必须随之修改，以保持连续性，并跟踪程序入口位置。收缩和展开：一条宏命令用几条宏命令替换、执行相同任务，或者几条宏命令合并用一条宏命令替换。这与宏语言的设计有密切关系，可以参考 M 文的指令对和三联组，只不过改成宏代号之间的对应。  
　　变形模块的收缩/展开部分隐含地使用一个宏替换表，展开比较容易做，收缩需要判断在逻辑上连续的几个宏代号是不是可以用一个宏代号替换。注意，不同之处：在 M 文中收缩/展开是一对互补操作，在这里一般情况下是随机决定把一个宏展开成几个宏、或者和后面的宏一起、判断能不能合并(如果后面的宏是引用或跳转目标就不能合并)。相同之处：能递归，所以要设定代码大小上下限，不要让若干代以后宏程序变得太大或太小，接近下限就多倾向于展开一些宏，接近上限就多倾向于收缩一些宏，达到界限就要反向操作或者只使用一条等效的宏代替(如 XOR EBX,EBX 用 SUB EBX,EBX，因为宏可以根据参数不同改变寄存器，所以可用的替换比较多)。应该防止万一收缩了不是原来展开的宏，造成很多宏不能收缩，因此要设计好宏替换表。  
　　\[注：“隐含地使用表”就是根据它编程，但不储存表。不过如果显式地使用表，编码更简单，可以通过分散存放、混洗、加密等等把表变形。\]  
　　递归性与宏的设计有关，如 M 文的例子，PUSH Reg <--> MOV Mem,Reg/PUSH Mem，而其中 MOV Mem,Reg <--> PUSH Reg/POP Mem，又递归使用到 PUSH Reg。这里一条指令或一个指令对都可以作为一个宏。  
　　变形以后，汇编模块就可以从新的宏程序翻译了。汇编模块的翻译部分也隐含地使用一个宏命令表，其中每个宏代号对应着一个指令格式串(1 至 3 条机器指令，已填好操作符和部分操作数，留空的操作数将用宏命令的参数填入)，如 M 文中的 MakeMOVMemReg()和 MakePUSHMem()等等。  
　　按宏命令的排列顺序(已置换)取下一条命令翻译，根据连接位置字段用无条件跳转连接。跳转指令可以用 JMP xxxx 或 CMP EAX,EAX/JZ xxxx 或 PUSH xxxx/RETN 或 PUSH EAX/XOR EAX,EAX/POP EAX/JNC xxxx，等等；跳转地址 xxxx 需要计算，实现方法至少有两个，用最大长度存放各指令组，再根据连接位置就可以计算出 xxxx，或者，翻译命令时用临时表记录其地址，最后按表修改跳转地址，在 M 文中对此有更多描述。如果有时又按连接位置取下一条命令翻译(此时不用插入跳转)，这样可使得代码整体架构改变，而实际算法没有改变。  
　　以上只是初步设想，很多问题不可能涉及，下一步详细设计时合理地设计宏是关键，然后编码首先要保证汇编模块能正确翻译，之后才变形。  
　  
　　　　　　　　　　　　　　　　　　　　月中人　　原创于 2006.8.22.

变形实践－“我如何做 MetaPHOR 以及我学会了什么”

| 译    者： || 月中人 |
| --- | --- |
| 日    期： ||  \[if supportFields\]><span lang=EN-US style='font-family: 黑体'><span style='mso-element:field-begin'><span style='mso-spacerun:yes'>&nbsp;DATE<span style='mso-spacerun:yes'>&nbsp; \\@ &quot;yyyy-MM-dd&quot;<span style='mso-spacerun:yes'>&nbsp; \\\* MERGEFORMAT <span style='mso-element: field-separator'><!\[endif\] 2007-01-11 \[if supportFields\]><span lang=EN-US style='font-family:黑体'><span style='mso-element:field-end'><!\[endif\]  |
| 发    布： || Pediy.com |
| 原文标题： || [Metamorphism in practice or "How I made MetaPHOR and what I've learnt"](http://vx.netlux.org/lib/vmd01.html) |
| 原文作者： || The Mental Driller (29A#6) |
| 原文日期： || February 2002 |
| 关 键 词： || 变形,宏,伪汇编,置换,收缩,展开 |
| 原文链接： || http://vx.netlux.org/lib/vmd01.html |

目录

 0.         基础知识

 a.         什么是变形作用？

 b.         变形代码的结构

 1.         规划

 a.         心理演算：我们用宏编码！

 b.         我们要做什么？

                         i.              简单化的：置换/代替

                       ii.              “手风琴”模型(收缩/展开)

                     iii.              伪汇编语言

 2.         编码

 a.         反汇编器/反置换器(反模糊器)

 b.         收缩器/仿真器

 c.         置换器

 d.         展开器(模糊器)

 e.         重新汇编器

 3.         已知问题(和解决方案)

 a.         调试你的引擎

 b.         API 调用

 c.         内存

 4.         未来

 a.         插件

 b.         多平台交叉感染

 c.         针对不同处理器重新汇编

 5.         结论

致谢

感谢所有的 29A 成员，感谢 Vecna 和 Z0MBiE、为了他们作为先行者在变形领域所做每一点探索，再次感谢 Vecna、为了他在文章的结构上给我的那些重要建议。

 

 0)      基础知识

 a)        什么是变形作用？

L 此类文章必不可少的问答 J。

变形作用是极度变异的人工技术。这意味着，我们变异代码中每个东西，而不仅仅变异一个可能存在的解密器。变形作用是多态的自然进化结果－多态象是在逃避病毒扫描器。一个病毒加上变形，检测它的难度呈指数上升。

那么，为什么变形病毒没有那么多呢？简单说：因为制作它们非常困难，如本文中所示(不仅由于所使用的技术，也由于我们编码会遇到的许多 ring0 级问题)。不管怎样，我们将努力理解这一点：可能关键是要有好点子(Vecna、Z0MBiE 等代码人具有的那些东西 - 大家好！J。

 b)        变形代码的结构

变形病毒就象一辆 49cc 摩托车载着一艘太空梭的燃料储备(如果你能想象出来)。事实上，90%以上的代码是变形引擎，用来变异那个致力于感染的一小部分代码，这有点荒谬。引擎必须能够变异它本身和附属代码－这些代码让引擎可以独自地传播(oh)。完全不同于小巧的多态引擎，它用一个 200 字节代码长的引擎去变异一个 8KB 的病毒：这次正相反！因为我们不能用 200 字节做一个完整的反汇编器(噢，也许超人可以 K。那么，其结构是：

| Engine || Virus |
| --- | --- |

其引擎的(典型)结构是：

| Disassembler || Shrinker || Permutator || Expander || Assembler |
| --- | --- | --- | --- | --- |

现在详细说明：

反汇编器 Disassembler

引擎的真正开始。反汇编器将译码每条指令以了解它的长度、使用的寄存器和所有有关它本身的信息。同样，它必须能译码那些改变 IP 的指令，象 x86 的 JMP 和 CALL(或者如其他的 BSR－Branch Subroutine 转移子程序)。

收缩器 Shrinker

也叫做压缩器，这部分将压缩已经过反汇编的代码－－病毒在这一代的代码也就是上一代生成的。即使你使用一个非展开的变异技术，也要这样做，以避免病毒一代一代地变得越来越大，在很少几代以后就会变成一个有许多兆字节的病毒。显然，这部分依赖你的变形代码样板\[注：原文 type，或译“类型”，意指是所使用的宏定义或其他与变形方法相关的东西\]，而且它也是最难做的：事实上，很少病毒有这部分。基本上，它是把展开器编码的许多条指令压缩成一条指令。它其实也是一个仿真器，消除无用和冗余指令，把多个操作压缩成一个操作(例如，MOV Reg,1234/ADD Reg,4321 --> MOV Reg,5555)。

置换器 Permutator

变形引擎的一个基本部分，许多病毒作者制作变形病毒都曾经编写这部分，虽然它根本不是变形，其实你指令没有改变。通常结合使用其他变形方法，象指令替换(例如，XOR EAX,EAX 代替 SUB EAX,EAX 等等)。它的原理非常简单，但是非常有效，因为它打断所有能被用来检测病毒的扫描字串。

展开器 Expander

这部分仅当有收缩器时才有(嗯，有些病毒有这部分，但是它们的代码增长失去控制)。它做的与收缩器做的正相反：它把一条指令重新编码成许多条指令，但执行相同的功能。

汇编器 Assembler

它重新编码我们用展开器构造出来的指令。它连接那些 JMP、CALL 之类转移指令，计算指令长度，改变寄存器等等。或者，如果你使用一个内部伪汇编器，那么它把伪代码重新汇编为目标处理器语言。

OK，这些就是常规部分。总之，决定你要做什么，你需要做一个规划！别犯傻(对不起 J，不能不规划你要做的事，就冒然制作一个困难的变形代码(容易做的可以 K。否则，最可能的结果就是你永远完成不了你的代码。在编码 MetaPHOR 病毒的时候，我为之规划了大约两个月，而且我想先在脑子里把自己想做的事弄清楚。嘿，相信我，它对我帮助很大，我节省了大量的工作。因此，让我们做一遍我的规划(就当作帮你规划你的代码)。

 1)      规划

 a)        心理演算：我们用宏编码！

这部分非常重要：你要当作不是在机器层次接触指令。你要表述成“移动这个数值到这个寄存器”或者“把这个寄存器的内容加到这个变量”。这样你对于编写变形代码会感觉更容易些(事实上，这和你编写多态代码时是一样的)。所有指令和指令组都是宏操作的真正实现。目的是学会把宏代码看作一束指令，宏不是最终让机器执行的代码，而是代表一个操作，必须被执行以制造出一个更重要的操作。

 b)        我们要做什么？

 i)          简单化的：置换/代替

如果你想用这种方法做变形(或者你不想做一个 400Kb 的引擎那么辛苦)，那么这就是你的选择。计划好怎样置换代码(JMP 链接、单指令轮换和 NOP 填充，等等)。必需的前提是这个引擎只能用于你自己编写的病毒，因为整个代码必须与该引擎的样板兼容。例如，假设你选择用 NOP 填充的方法置换代码，就必须使所有指令占用的空间大小相同，并且用 NOP 填充多出来的空间。这是最简单的变形方法\[注：原文 meta 应是 metamorphism 的略写\]，同时也是不需要做很多规划的方法：只要直接编码，在你完成第一版本编码后(并且在你测试之前)，用 NOP 填充即可。NOP 填充的另一个方法是用宏做：

        db 10h dup (90h) *; NOPs*

        org $-10h

        <instruction>

        .align 4

或者类似的东西。如果你是个正常人就编码一个宏，或者如果你是疯狂的就直接做 K。

与 NOP 填充不同的另一个方法是在运行时取得每条指令的长度，然后在缓冲区弄乱这些指令的排列顺序。这需要反汇编以调整 JMPs、CALLs 和 Jccs(条件跳转)，所以它也许不象说的这么简单。

  

 

 ii)          “手风琴”模型(收缩/展开)

多好的技术名称! XD 这种变形的威力在于，可能用不着置换，而代码永远都不一样的(而且能与产生“绝对变形”的置换法完美地结合使用)。要做这种变形，你必须先决定，是使用迷你型仿真器，还是仅仅反汇编成伪汇编码。在 MetaPHOR 中我采用后者。

对于后者，你首先定义你自己的汇编语言(可以基于 x86 操作码)。要记住，这种汇编语言越象 x86 操作码，就越容易操纵。那么，让我们结合下一节来学习...

 iii)       伪汇编语言

...继续。MetaPHOR 的内部伪汇编码遵守以下规则：

 

 a.          所有指令是 16 字节长(但将来操作象 Itanium 这样的 64 位处理器时这可以改)。

 b.          指令的结构全都是一样的：

一般结构：

每条指令 16 字节，

00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

OP \*----- instruction data ----\* LM \*-pointer-\*

OP 是指令的操作符，它决定我们使用哪种指令数据结构。

LM 是“标签记号”。当有一个标签指向这条指令时其值为 1，它的用处很多，例如用于了解两条指令能不能被收缩(如果第二条指令有标签那么就不能收缩)。它在指令中的第 0B 字节。\[注：16 字节伪指令是从第 00 到第 0F，下同\]

第 0C 字节的双字(dword)是一个指针，意即“上次代码引用”。在反汇编时它表示 EIP，即这条指令用它储存原始编码的位置；而我们进一步加工代码的时候，用它储存该指令上次位置的引用。这用于辅助修改标签表、重新编码转移指令(JMP、CALL 等)，等等。

该引擎用于指令的结构：\[注：即伪指令的 instruction data 部分\]

Memory\_address\_struct:

| +01 || First index |
| --- | --- |
| +02 || Second index, bits 7&6 are the multiplicator (00=\*1,01=\*2,10=\*4,11=\*8) |
| +03 || DWORD addition to indexes |

 

依据操作符(即所要执行的操作)，用下面算法(判断指令数据结构中的操作数含义)：

  

 

 

 l         如果该操作没有操作数(如 NOP、RET 等操作)，那么指令数据是没意义的。

 l         如果该操作有一个操作数：

Register operand:

+01: Register

Memory address: 

+01: Memory address struct 

Immediate value: 

+07: DWORD value, zero extended if it's a byte operation 

Destiny address (JMP, CALL, etc.) 

+01: Label to jump to (DWORD) 

 l         如果该操作有两个操作数：

Reg,Imm:

+01: Register

+07: DWORD immediate value, zero extended if it's a 8-bits op. 

Reg,Reg: 

+01: Source register

+07: Destiny register 

Reg,Mem: 

+01: Memory address struct

+07: Destiny register 

Mem,Reg: 

+01: Memory address struct

+07: Source register 

Mem,Imm: 

+01: Memory address struct

+07: DWORD immediate value, zero extended if it's a 8-bits op. 

\[注：以上算法说明：+01 表示第 01 字节是什么－冒号后面，如有 Register 则是寄存器，如有 DOWRD 则为 4 字节立即数或标签地址，如果是 Memory address struct 则根据上面那个表。+07 与之类似。如果是 8 位操作而且有立即数，在伪指令中立即数零扩展为双字\]

根据这个规则，现在我们使用下面的伪操作符：

00: ADD, 08: OR, 20: AND, 28: SUB, 30: XOR, 38: CMP, 40: MOV, 48: TEST

设定规则：

| +00 || Reg,Imm |
| --- | --- |
| +01 || Reg,Reg |
| +02 || Reg,Mem |
| +03 || Mem,Reg |
| +04 || Mem,Imm |
| +80 || 8 bits operation |

所以，操作码 83 表示 ADD Mem,Reg 使用 8 位操作数，以此类推。

\[注：根据伪操作符和上面规则可知，83 是 00+03+80；以下继续是伪操作符及其含义\]

| 50 || PUSH Reg |
| --- | --- |
| 51 || PUSH Mem |
| 58 || POP Reg |
| 59 || POP Mem |
| 68 || PUSH Imm |
| 70\-7F || Conditional jumps |
| E0 || NOT Reg |
| E1 || NOT Mem |
| E2 || NOT Reg8 |
| E3 || NOT Mem8 |
| E4 || NEG Reg |
| E5 || NEG Mem |
| E6 || NEG Reg8 |
| E7 || NEG Mem8 |
| E8 || CALL label |
| E9 || JMP label |
| EA || CALL Mem (used for API calls) |
| EB || JMP Mem (used for obfuscation in API calls) |
| EC || CALL Reg (obfuscation of API calls) |
| ED || JMP Reg (idem) |
| F0 || SHIFT Reg,Imm |
| F1 || SHIFT Mem,Imm |
| F2 || SHIFT Reg8,Imm |
| F3 || SHIFT Mem8,Imm For all SHIFTs:

| +07 || Byte with the value of rotation/shifting |
| --- | --- |
| +08 || Operation performed: 0: ROL, 8: ROR, 20: SHL, 28: SHR |

 |
| F4 || APICALL\_BEGIN Special operation meaning PUSH EAX/PUSH ECX/PUSH EDX that avoids the recoding of these registers, always remaining the same. |
| F5 || APICALL\_END The complementary of APICALL\_BEGIN, it means POP EDX/POP ECX/POP EAX |
| F6 || APICALL\_STORE 

| +01 || Memory address struct |
| --- | --- |

This always means: MOV \[Mem\],EAX <-- Avoiding the recoding of EAX |
| F7 || SET\_WEIGHT 

| +01 || Memory address struct |
| --- | --- |
| +07 || Byte with the weight identificator |
| +08 || Register 1 in code structure |
| +09 || Register 2 in code structure |

 |
| F8 || MOVZX Memory address struct is a 8-bits operand, while +07 is a 32 bit reg. |
| FC || LEA |
| FE || RET |
| FF || NOP |

这些操作码会在反汇编输出中出现。此外，我还定义了另外一些操作码用于内部操作：

| 4F || Exists only While shrinking, and means a MOV Mem,Mem, being: 

| +01 || Source memory address struct |
| --- | --- |
| +07 || Pointer to the instruction that holds the destiny memory address struct, which has the format: 

| +00 || FF (NOP) |
| --- | --- |
| +01 || Destiny memory address struct |
| +07 || Ptr to source memory address holder (the 4F instruction) |

 |

 |
| --- | --- |

因为 4F 只是一个转换操作码，在收缩这一步以后可以用来表示其他含义。另外有三个操作码只用于重新汇编：

| 4E || INC/DEC Register 

| +01 || Register |
| --- | --- |
| +07 || 0 if INC, 8 if DEC |

 |
| --- | --- |
| 4F || INC/DEC memory address 

| +01 || Memory address struct |
| --- | --- |
| +07 || 0 if INC, 8 if DEC |

 |
| FD || Literal byte 

| +01 || Byte to insert directly into the code |
| --- | --- |

 |

以上就是我所做规划的开始部分。如你所见，操作码与 x86 的非常相似，所以很容易记 (事实上，我写这个列表只是为了回忆它 J。

现在，规划的第二部分：收缩器。我们该如何收缩？我们将在它那部分学习，因为我不想写两遍 L。

 2)      编码

 a)        反汇编器/反置换器

这是引擎的入口点！当然，做任何东西之前，我们必须反汇编。反汇编器从原理上讲是很容易的，却是一个实实在在的苦差事。因为我们的目标是生成绝对变形的代码，所以不能使用散列表，虽然我们可以编一段程序变异散列、把它变得更强。我的解决方案是前者，而且我在反汇编器中隐含地实现反置换。译码原理如下：

我们有一个内存缓冲区，其大小为容纳我们将要反汇编的代码，我们把代码入口点赋值给 ESI。这个缓冲区(其中各地址储存于变量 PathMarks)让我们容易控制已经反汇编的代码。我们还有另外两个表：LabelTable 和 FutureLabelTable，也已经初始化。这两个表各有一个计数器变量，其值为表中元素序号。此外我们把指令译码到 DisassembledCode，这个地址放在 EDI。

LabelTable 表中每个元素是两个双字长。第一个双字储存指针指向的真实 EIP，第二个双字储存指针指向相应的已反汇编代码。然后，当我们译码一个 JMP 时，我们在该表中设定一个指针作为标签。这样，我们移动该标签的内部指针时，引用该标签的所有指令就被自动更新。

FutureLabelTable 是一个缓冲区表，只用于反汇编。它用来储存那些跳转、调用等指令的目的地，指向我们还没有反汇编的代码。每译码一条指令，我们都要检查该表中有没有该指令的地址，如果该地址出现在表中，我们就可以把所有引用该地址的指令弄完整。

有了以上这些，再看看算法：

 1.       初始化 PathMarks 映射表(即清零)，并初始化标签的个数(LabelTable 的计数器)和预测标签的个数(FutureLabelTable 的计数器)。

 2.       把当前 EIP(在 ESI 中)直接转化为 PathMarks 映射表中一个条目。

 ¡        如果映射表中已经有该条目，那么在 ESI 中储存为一个 JMP，跳转目的地是这条已反汇编指令\注：然后会被当作“没有该条目\->它是 JMP”的情况再次处理\。

 ¡        如果映射表中没有该条目，那么标记这个地址为已经反汇编并且译码这条指令。接着，我们根据该指令不同处理：

 n         如果它是 JMP：

 ²        如果它指向一个已经被译码的地址，那么写入一个 JMP 指令，并在 LabelTable 表中插入一个跳转目的地的标签，再从 FutureLabelTable 表中取出一个新的 EIP。如果 LabelTable 表中已经有这个标签，那么这个 JMP 指令就使用该标签。

 ²        如果它指向一个还没有被译码的地址，那么暂时先写一条 NOP 指令(以防万一某个标签直接引用这个地址)，然后把跳转目的地赋值给 ESI 作为新的 EIP。这样，我们消除了一个可能的置换 JMP。

 n         如果它是 Jcc(条件跳转)：

 ²        如果它指向一个已经被译码的地址，那么写入这个 Jcc，而且如果这个 Label 还不存在就插入一个跳转目的地的 Label(如果该 Label 已经插入到表中，就使用该 Label)。

 ²        如果该目的地还没有被反汇编，那么把地址储存到 FutureLabelTable 表并且继续(处理下一条指令)。

 n         如果它是 CALL，那么采用类似 Jcc 的处理办法(只不过写入 CALL 而不是写入 Jcc)。

 n         如果它是 RET、JMP Reg 或者是 JMP \Mem\，那么储存该指令并且从 FutureLabelTable 表中取一个新的 EIP。

\[注：“置换 JMP”－－在进行置换时插入的跳转指令 JMP。算法中可能消除置换用的 JMP，也可能消除其他作用的 JMP。PathMarks map 意为“执行路线标记映射表”。\]

从 FutureLabelTable 表中取出一个新的 EIP 的时候，我们检查存于该表的标签是不是已经译码了\[注：意即该标签所指示地址的指令是否已经被反汇编\]。如果是，那么我们把相应的标签插入到 LabelTable 表中，并从 FutureLabelTable 表中去掉该条目。如果不是，我们就处理这个新的 EIP(即，我们把新的入口点赋给 ESI)，在 LabelTable 表中插入新标签并继续。

你可以推理出，当 FutureLabelTable 表中再没有条目时，反汇编就结束了，因为没有条目意味着我们是从代码流的终端叶子出发。完成这样的“仿真过程”以后，我们已经：

 1.    消除了置换和置换跳转(因为我们直接改变 ESI 中的 EIP 而消除了那些 JMP)。

 2.    消除了所有绝对无法达到的代码。

 3.    整个代码译解为我们的伪汇编码

 4.    用指向表条目的指针代替标签。

反汇编器做好了！我们不必编写一个反置换器代码或一个仿真器代码来检测死码区，因为我们隐含地消除它们了。一个反置换器执行的例子是：

       CODE         PASSES

      ------        ------

        xxx1        1) Decode xxx1

        xxx2        2) Decode xxx2

        xxx3        3) Decode xxx3

        jmp @A      4) Change EIP to @A (don't store label)

        yyy1        5) Decode xxx7

        yyy2        6) Decode xxx8

    @B: xxx4        7) Decode xxx9

        xxx5        8) Change EIP to @B (don't store label)

        xxx6        9) Decode xxx4

        jmp @C     10) Decode xxx5

        yyy3       11) Decode xxx6

        yyy4       12) Change EIP to @C (don't store label)

    @A: xxx7       13) Decode xxx10

        xxx8       14) Decode xxx11

        xxx9       15) Decode JZ and store @D in FutureLabelTable

        jmp @B     16) Decode xxx12

    @D: xxx13      17) Decode RET, get @D from FutureLabelTable and

        xxx14        complete the JZ at pass 15 (@D = current EIP)

        RET        18) Decode xxx13

        yyy5       19) Decode xxx14

    @C: xxx10      20) Decode RET and get an item from FutureLabelTable.

        xxx11        Since it's empty, we have decoded everything, so finish.

        jz @D

        xxx12

        RET

  

 

反汇编结果将会是：

        xxx1

        xxx2

        xxx3

        xxx7

        xxx8

        xxx9

        xxx4

        xxx5

        xxx6

        xxx10

        xxx11

        jz @D

        xxx12

        RET

  @D:   xxx13

        xxx14

        RET

我想现在更清楚了。所有垃圾和填空代码(那些 yyy?指令)都被隐含地消除了，因此以后不必寻找它们。

这里的问题是，一旦被收缩，代码构架(在此已经反置换了)会被检测病毒的利用。或许不会，但是我们还是想让构架每代都不相同(西班牙语是“rizar el rizo”，或者西英混合语叫做“to loop the loop” J。因此，我们这一步插入三维指令，我把它叫作“3D instructions”。什么？嗯哼...好，这不是哗众取宠，它只是形象化表述：想象一下你在各维做的变形：第一维是当前代码(你正在反汇编的东西)。第二维是下面马上要做出的编码(这次变形的结果)。第三维是将来下一代所译出的代码。

那么，第一维和第二维是明确的，但是...第三维是什么呢？简单说：例如，把一些 JMP 编码为 CMP EAX,EAX/JZ @xxx。你看出关键了吗？收缩器(我们在下一部分马上要讲到)必须能够把“CMP EAX,EAX/JZ @xxx”对压缩成“JMP @xxx”，但是这要在下一代才发生，而在这一代并不压缩它(在更远的后代有一些结构将被完全压缩掉)。我们唯一必须要注意的是，我们不能在这类跳转的后面放垃圾指令，因为它们也将会被译码。让我们再用上面的代码为例，这次先去掉那些 yyy?，并用 CMP/JZ 对代替其中第一个 JMP，如下面左边代码：

\[注：CMP/JZ 将在这一代的收缩器中被压缩成 JMP @A，然后在下一代的反汇编中被隐式消除。这里 CMP/JZ 就是所谓的 3D instruction，在第三维被消除，所以它的主要作用是让代码在这一代的处理过程中永远保持一定的变形复杂性。\]

  

 

          xxx1       Result of the disassembly:

          xxx2          xxx1

          xxx3          xxx2

          CMP X,X       xxx3

          JZ @A         CMP X,X

      @B: xxx4          JZ @A    --> This will be compressed to JMP @A on this

          xxx5      @B: xxx4         generation and then eliminated implicitly

          xxx6          xxx5         in the disassembly of next generation.

          jmp @C        xxx6

      @A: xxx7          xxx10

          xxx8          xxx11

          xxx9          jz @D

          jmp @B        xxx12

      @D: xxx13         RET

          xxx14     @A: xxx7

          RET           xxx8

      @C: xxx10         xxx9

          xxx11         jmp @B

          jz @D     @D: xxx13

          xxx12         xxx14

          RET           RET

这意味着绝对变形。构架改变，但是代码算法没变。必需要经历若干代才能变回到最初的代码，但是因为其他 JMPs 同样被变异，实际上你永远无法达到，而且你需要经历若干代来消除一个置换 JMP，而其时另外一些新的又被插入。好在这种插入不是无限的，达到极致时这个代码稳定了，但是之前那些代的代码是变化的，我们会看到我们的代码因为不断插入这些跳转而体积增长。

这里还可以做的另一件事，就是用这个反汇编器把一些指令译码成易懂形式：例如，INC Reg --> ADD Reg,1。这样，我们只须处理一种指令，不用处理它的所有变体－即该处理器中所有执行相同操作的指令(虽然这也可以用收缩器来做，但是在这里做我们就可以少设计一些伪操作码)。

 b)        收缩器/仿真器(反模糊器) 

在此，我们要仿真还是压缩？仿真更先进更强大，但是它意味着编码非常复杂，而且有很多问题，比如循环后数值的控制。比较容易且有更好的质量/数量压缩比的方法，是压缩已知的指令对和指令三联组：和展开器部分所做的正好相反。

规划必须有这部分。你必须列出所有可能的单条指令、指令对、指令三联组，并且决定哪些是要压缩/展开以及哪些是不压缩/不展开。收缩器可以被用来消除“智能垃圾”：在算法中插入一些似乎成为算法的一部分但实际上无用的代码。现在我列出一些单条/对/三联组指令，它们在 MetaPHOR 中有机会被压缩，同时它们也是展开器可以用一条指令生成的那些指令组合。

图注：

 

Reg    寄存器

Mem    内存地址

Imm    立即数

当一条指令是 Reg,Reg 这种类型的，两个 Reg 表示同一寄存器。(举个例子)如果不是同一寄存器，我就会写成 Reg,Reg2。

对一条指令的变形：

 

| XOR Reg,-1  || \--> NOT Reg |
| --- | --- |
| XOR Mem,-1  || \--> NOT Mem |
| MOV Reg,Reg  || \--> NOP |
| SUB Reg,Imm  || \--> ADD Reg,-Imm |
| SUB Mem,Imm  || \--> ADD Mem,-Imm |
| XOR Reg,0  || \--> MOV Reg,0 |
| XOR Mem,0  || \--> MOV Mem,0 |
| ADD Reg,0  || \--> NOP |
| ADD Mem,0  || \--> NOP |
| OR Reg,0  || \--> NOP |
| OR Mem,0  || \--> NOP |
| AND Reg,-1  || \--> NOP |
| AND Mem,-1  || \--> NOP |
| AND Reg,0  || \--> MOV Reg,0 |
| AND Mem,0  || \--> MOV Mem,0 |
| XOR Reg,Reg  || \--> MOV Reg,0 |
| SUB Reg,Reg  || \--> MOV Reg,0 |
| OR Reg,Reg  || \--> CMP Reg,0 |
| AND Reg,Reg  || \--> CMP Reg,0 |
| TEST Reg,Reg  || \--> CMP Reg,0 |
| LEA Reg,\[Imm\]  || \--> MOV Reg,Imm |
| LEA Reg,\[Reg+Imm\]  || \--> ADD Reg,Imm |
| LEA Reg,\[Reg2\]  || \--> MOV Reg,Reg2 |
| LEA Reg,\[Reg+Reg2\]  || \--> ADD Reg,Reg2 |
| LEA Reg,\[Reg2+Reg2+xxx\]  || \--> LEA Reg,\[2\*Reg2+xxx\] |
| MOV Reg,Reg  || \--> NOP |
| MOV Mem,Mem  || \--> NOP (result of a compression of PUSH Mem/POP Mem, with pseudoopcode 4F) |

被消去的指令\注：指箭头左边的\被当作垃圾指令使用，与可执行代码放在一起。由于每个 NOP 指令都能被展开(例如，MOV Reg,Reg 可以被做成 PUSH Reg/POP Reg，而每一个 PUSH 和 POP 也都可以被展开，以此类推)，所以你无法知道哪些是垃圾，哪些不是，除非你把全部都压缩了。

MetaPHOR 能压缩的指令对有：

| PUSH Imm / POP Reg  || \--> MOV Reg,Imm |
| --- | --- |
| PUSH Imm / POP Mem  || \--> MOV Mem,Imm |
| PUSH Reg / POP Reg2  || \--> MOV Reg2,Reg |
| PUSH Reg / POP Mem  || \--> MOV Mem,Reg |
| PUSH Mem / POP Reg  || \--> MOV Reg,Mem |
| PUSH Mem / POP Mem2  || \--> MOV Mem2,Mem (codificated with pseudoopcode 4F) |
| MOV Mem,Reg/PUSH Mem  || \--> PUSH Reg |
| POP Mem / MOV Reg,Mem  || \--> POP Reg |
| POP Mem2 / MOV Mem,Mem2  || \--> POP Mem |
| MOV Mem,Reg / MOV Reg2,Mem  || \--> MOV Reg2,Reg |
| MOV Mem,Imm / PUSH Mem  || \--> PUSH Imm |
| MOV Mem,Imm / OP Reg,Mem  || \--> OP Reg,Imm |
| MOV Reg,Imm / ADD Reg,Reg2  || \--> LEA Reg,\[Reg2+Imm\] |
| MOV Reg,Reg2 / ADD Reg,Imm  || \--> LEA Reg,\[Reg2+Imm\] |
| MOV Reg,Reg2 / ADD Reg,Reg3  || \--> LEA Reg,\[Reg2+Reg3\] |
| ADD Reg,Imm / ADD Reg,Reg2  || \--> LEA Reg,\[Reg+Reg2+Imm\] |
| ADD Reg,Reg2 / ADD Reg,Imm  || \--> LEA Reg,\[Reg+Reg2+Imm\] |
| OP Reg,Imm / OP Reg,Imm2  || \--> OP Reg,(Imm OP Imm2) (must be calculated) |
| OP Mem,Imm / OP Mem,Imm2  || \--> OP Mem,(Imm OP Imm2) (must be calculated) |
| LEA Reg,\[Reg2+Imm\] / ADD Reg,Reg3  || \--> LEA Reg,\[Reg2+Reg3+Imm\] |
| LEA Reg,\[(RegX+)Reg2+Imm\] / ADD Reg,Reg2 || \--> LEA Reg,\[(RegX+)2\*Reg2+Imm\] |
| POP Mem / PUSH Mem  || \--> NOP |
| MOV Mem2,Mem / MOV Mem3,Mem2  || \--> MOV Mem3,Mem |
| MOV Mem2,Mem / OP Reg,Mem2  || \--> OP Reg,Mem |
| MOV Mem2,Mem / MOV Mem2,xxx  || \--> MOV Mem2,xxx |
| MOV Mem,Reg / CALL Mem  || \--> CALL Reg |
| MOV Mem,Reg / JMP Mem  || \--> JMP Reg |
| MOV Mem2,Mem / CALL Mem2  || \--> CALL Mem |
| MOV Mem2,Mem / JMP Mem2  || \--> JMP Mem |
| MOV Mem,Reg / MOV Mem2,Mem  || \--> MOV Mem2,Reg |
| OP Reg,xxx / MOV Reg,yyy  || \--> MOV Reg,yyy |
| Jcc @xxx / !Jcc @xxx  || \--> JMP @xxx (this applies to (Jcc & 0FEh) with (Jcc | 1) |
| NOT Reg / NEG Reg  || \--> ADD Reg,1 |
| NOT Reg / ADD Reg,1  || \--> NEG Reg |
| NOT Mem / NEG Mem  || \--> ADD Mem,1 |
| NOT Mem / ADD Mem,1  || \--> NEG Mem |
| NEG Reg / NOT Reg  || \--> ADD Reg,-1 |
| NEG Reg / ADD Reg,-1  || \--> NOT Reg |
| NEG Mem / NOT Mem  || \--> ADD Mem,-1 |
| NEG Mem / ADD Mem,-1  || \--> NOT Mem |
| CMP X,Y / != Jcc (CMP without Jcc)  || \--> NOP |
| TEST X,Y / != Jcc  || \--> NOP |
| POP Mem / JMP Mem  || \--> RET |
| PUSH Reg / RET  || \--> JMP Reg |
| CALL Mem / MOV Mem2,EAX  || \--> CALL Mem / APICALL\_STORE Mem2 |
| MOV Reg,Mem / CALL Reg  || \--> CALL Mem |
| XOR Reg,Reg / MOV Reg8,\[Mem\]  || \--> MOVZX Reg,byte ptr \[Mem\] |
| MOV Reg,\[Mem\] / AND Reg,0FFh  || \--> MOVZX Reg,byte ptr \[Mem\] |

也许有更多，但是至少对我们的命题来说这套足够了。我们必需领悟的是，对于此等情形，扫描代码然后用等效指令替换指令对的第一条指令，再用 NOP 覆盖第二条，这样就把指令对压缩了。

但有更多：三联组：

| MOV Mem,Reg   
OP Mem,Reg2  
MOV Reg,Mem  || \--> OP Reg,Reg2 |
| --- | --- |
| MOV Mem,Reg   
OP Mem,Imm   
MOV Reg,Mem  || \--> OP Reg,Imm |
| MOV Mem,Imm   
OP Mem,Reg   
MOV Reg,Mem  || \--> OP Reg,Imm (it can't be SUB) |
| MOV Mem2,Mem  
OP Mem2,Reg  
MOV Mem,Mem2  || \--> OP Mem,Reg |
| MOV Mem2,Mem  
OP Mem2,Imm  
MOV Mem,Mem2  || \--> OP Mem,Imm |
| CMP Reg,Reg   
JO/JB/JNZ/JA/JS/JNP/JL/JG @xxx  
!= Jcc  || \--> NOP |
| CMP Reg,Reg   
JNO/JAE/JZ/JBE/JNS/JP/JGE/JLE @xxx  
!= Jcc  || \--> JMP @xxx |
| MOV Mem,Imm   
CMP/TEST Reg,Mem  
Jcc @xxx  || \--> CMP/TEST Reg,Imm  
Jcc @xxx |
| MOV Mem,Reg   
SUB/CMP Mem,Reg2  
Jcc @xxx  || \--> CMP Reg,Reg2  
Jcc @xxx |
| MOV Mem,Reg   
AND/TEST Mem,Reg2  
Jcc @xxx  || \--> TEST Reg,Reg2  
Jcc @xxx |
| MOV Mem,Reg   
SUB/CMP Mem,Imm  
Jcc @xxx  || \--> CMP Reg,Imm  
Jcc @xxx |
| MOV Mem,Reg   
AND/TEST Mem,Imm  
Jcc @xxx  || \--> TEST Reg,Imm  
Jcc @xxx |
| MOV Mem2,Mem  
CMP/TEST Reg,Mem2  
Jcc @xxx  || \--> CMP/TEST Reg,Mem  
Jcc @xxx |
| MOV Mem2,Mem  
AND/TEST Mem2,Reg  
Jcc @xxx  || \--> TEST Mem,Reg  
Jcc @xxx |
| MOV Mem2,Mem  
SUB/CMP Mem2,Reg  
Jcc @xxx  || \--> CMP Mem,Reg  
Jcc @xxx |
| MOV Mem2,Mem  
AND/TEST Mem2,Imm  
Jcc @xxx  || \--> TEST Mem,Imm  
Jcc @xxx |
| MOV Mem2,Mem  
SUB/CMP Mem2,Imm  
Jcc @xxx  || \--> CMP Mem,Imm  
Jcc @xxx |
| PUSH EAX  
PUSH ECX  
PUSH EDX  || \--> APICALL\_BEGIN |
| POP EDX  
POP ECX  
POP EAX  || \--> APICALL\_END |

也可能有更多，上面这些只是我使用的。我们采取和指令对相同的机制：我们检查指针所指的三条指令是否构成一个已定义的三联组，然后我们压缩它、用两组 NOP 覆盖后两条指令。

我们定义好单条指令、指令对和指令三联组，接着就看压缩算法了。如果我们是递归展开的(就是说，假如我们编码 PUSH Imm/POP Reg，而 PUSH Imm 可以被进一步编码为 MOV Mem,Imm/PUSH Mem，所以就变成是“MOV Mem,Imm/PUSH Mem/POP Reg”，而 POP Reg 也可以被进一步展开)，那么这里也不能使用定义直接收缩，所以压缩算法如下：

          CurrentPointer = FirstInstruction

       @@Loop:

          if (\[CurrentPointer\] == MATCHING\_SINGLE)

          {

             Convert it

             if (CurrentPointer != FirstInstruction) call DecreasePointer

             if (CurrentPointer != FirstInstruction) call DecreasePointer

             if (CurrentPointer != FirstInstruction) call DecreasePointer

             goto @@Loop

          }

          if (\[CurrentPointer\] == MATCHING\_PAIR)

          {

             Convert it

             if (CurrentPointer != FirstInstruction) call DecreasePointer

             if (CurrentPointer != FirstInstruction) call DecreasePointer

             if (CurrentPointer != FirstInstruction) call DecreasePointer

             goto @@Loop

          }

          if(\[CurrentPointer\] == MATCHING\_TRIPLET)

          {

             Convert it

             if (CurrentPointer != FirstInstruction) call DecreasePointer

             if (CurrentPointer != FirstInstruction) call DecreasePointer

             if (CurrentPointer != FirstInstruction) call DecreasePointer

             goto @@Loop

          }

          do (CurrentPointer++) while \[CurrentPointer\] == NOP

          if(CurrentPointer != LastInstruction) goto @@Loop

 

      DecreasePointer:

          do (CurrentPointer--) while ((\[CurrentPointer\] == NOP) &&

                                       (\[CurrentPointer.Label == FALSE))

          return

我们不必使用一些 NOP 填充后续指令，这样才能去掉由前面反汇编产生的那些不受欢迎的垃圾。这是因为压缩算法没有处理那种以 RET、JMP 之类指令作为指令对/三联组第一条指令的情况，而且我们非常肯定代码是以这些指令中的某一个作为结束的。并请注意我们总是忽略掉那些 NOP！

也要小心有标签的那些指令：如果有一个标签指向一组指令中的某一条(第一条除外)，我们绝不压缩这组指令(这就是为什么我们要在规划中定义的指令结构中设置一个 LM 字节)。有标签的指令意味着，有某一个跳转或调用等等指向这条指令，如果我们将它与其上一条指令合并，就会把代码破坏了。

压缩时消除的内存地址只不过是临时变量，在重新汇编中预留它们用来存放数值以执行该操作\[注：只在实现该操作的一组指令内使用\]。因此，我们绝不能在任何指令对或三联组中使用真正的变量(储存重要数据的内存地址)，否则该变量将会被消除而导致引擎损坏。只要稍微注意到这一点，我们就不用担心内存变量了，而且不必检查他们是重要变量还是过渡变量，我们可以根据代码随意地重新定义这些变量的位置、并且可以和临时变量混放在一起。

一个收缩器执行的例子：

  Original code:

 

  MOV \[Var1\], ESI  \* PUSH ESI       \* MOV EAX,ESI

  PUSH \[Var1\]      \* nop              nop

  POP EAX            POP EAX        \* nop

  PUSH EBX           PUSH EBX         PUSH EBX        =====>

  POP \[Var2\]         POP \[Var2\]       POP \[Var2\]

  ADD EAX,\[Var2\]     ADD EAX,\[Var2\]   ADD EAX,\[Var2\]

 

 

 

 

             MOV EAX,ESI        MOV EAX,ESI   \* LEA EAX,\[ESI+EBX\]

             nop                nop             nop

             nop                nop             nop

  ====>    \* MOV \[Var2\],EBX   \* ADD EAX,EBX   \* nop

           \* nop                nop             nop

             ADD EAX,\[Var2\]   \* nop             nop

把上面这个算法传给伪汇编器以后，我们得到的最终代码将杂有大量 NOP 指令。但是不要紧，因为展开的时候，我们就忽略掉这些 NOP 指令，做一个真正的最优化(虽然实现的最优化又被展开器削弱，但谁会在意呢...)。

 c)        置换器

使用一个内部汇编器使得我们在这一步很容易做，而且我们不必关照所有指令相同尺寸以及类似问题，因为重新汇编器会给我们计算。

置换代码最容易的方法是定义“代码框架”：我们构造一个表，在其中定义代码各分区，给每个分区指定一个起始偏移量和一个结束偏移量，这样：

    ESI = Initial address of instructions

    EDI = Address of last instruction

 

    Given ESI = 00000000h,

          EDI = 00000060h

 

    while(ESI < EDI)

       Store ESI

       ESI += Random(8)+8

       Store ESI

       if((ESI+0F > EDI)

          Store ESI,EDI

          break;

       end if

    end while

       

    Result (for example):

          DD    00000000h,0000000Ah

          DD    0000000Ah,00000017h

          DD    00000017h,00000023h

          DD    00000023h,00000032h

          DD    00000032h,0000003Dh

          DD    0000003Dh,00000049h

          DD    00000049h,00000052h

          DD    00000052h,00000060h

现在我们混洗这个数组的元素。混洗很容易(有很多算法可用)。混洗以后我们得到：

\[注：原文 shuffle 意即“混洗”，即弄乱排列顺序\]

          DD    00000032h,0000003Dh

          DD    00000023h,00000032h

          DD    0000000Ah,00000017h

          DD    00000000h,0000000Ah

          DD    00000017h,00000023h

          DD    00000052h,00000060h

          DD    0000003Dh,00000049h

          DD    00000049h,00000052h

混洗时，我们跟踪第一个框架，这是代码的入口点。如果第一个框架不是入口点，我们在该框架中插入一个 JMP。因为所有跳转指令的目的地未知，所以我们先把它们存入一个表中，在复制所有的指令之后，完成它们。所以，我们首先要做的是插入一个跳转到入口点的 JMP，然后我们开始复制指令。根据第一个框架的定义，我们必须复制地址偏移量从 32h 到 3Dh 的那些指令，然后插入一个跳转到下一个框架的 JMP，以此类推。

经过以上处理，我们就得到一置换过的代码。此时我们只须根据先前的储存表完成所有的 JMP，这一步就圆满完成了。

可以在这里尝试别的做法：举个例子，假如你留着收缩器生成的那些 NOP 指令，会导致更随机的代码分布，因为那些 NOP 指令也参予置换，而下一步代码处理又把这些 NOP 消除了。

 d)       展开器(模糊器)

展开器的工作与收缩器是相反的。它准确地执行反操作(当然，以随机方式)。只取已定义的单条指令、指令对和指令三联组，并且把指令用等效编码代替(每条指令除了它们的直接编码之外都有一个可选的等效替代)。

在此，我们将把全部代码递归地编码。例如，见到一个 50h 操作码(PUSH Reg)的时，我们将调用 MakePUSHReg()，它将随机决定是直接编码成 PUSH，还是使用一个已定义的指令对或三联组绕着弯实现。如此，该函数决定用 MOV Mem,Reg/PUSH Mem，所以调用 MakeMOVMemReg()和 MakePUSHMem()。可这样太没有规律了！函数 MakeMOVMemReg() 调用可能换成 MakePUSHReg()+MakePOPMem()，于是又调用 MakePUSHReg()了。这会让代码增长太多，因此我们给它加一个递归控制，即一个变量，当递归调用一个函数时增量，离开时减量。我们检查该变量，看它是不是达到一定值，如果是，就直接编码该指令(在本例中，也就是使用 50h+Reg 并且 EIP++ 继续处理下一条指令)。

如果我们使用内部的汇编语言，展开器就不必处理最终编码了，它会更好。最终编码的任务就由重新汇编器完成(下一节)。为了保证代码展开结果的品质，我们要生成尽可能类似最终代码的伪汇编码(这就是为什么我在操作码列表中使用象 4E 和 4F 这样的操作码来编码 INC 和 DEC)。所以，如果我们在展开器中见到 CMP EAX,0，我们把它编成伪汇编码 OR EAX,EAX 或者 TEST EAX,EAX，如此，重新汇编器只须直接编码指令，除了操作码生成的某些随机性之外，不必解决其他任何东西。

展开器还要做另外一些操作，如：

换掉寄存器

我们选择一个新的寄存器换掉我们一直用到现在的那个寄存器。较容易的方法是在一个列表中放入 0,1,2,3,5,6,7 序列并打乱它，然后根据那些数字换掉每个寄存器。这样，执行那些操作时寄存器永远是不同的，甚至使得内存地址也不同，因为如果我们把 ESI 换成 EBP(举个例子)，这些内存引用的方法被汇编成 x86 是不同的(或者说可能是不同的)。我们可以在 Vecna 的 Regswap 中看到这一点。

重新选择变量

为了使用内存变量，我们必须拥有一个内存缓冲区(用 VirtualAlloc 预留，或用宿主本身的.bss 节，等等)。如果有变量，我们习惯在一个固定位置储存重要数据，忘了它吧！把它们全部一起放在一个缓冲区。这样，我们只须明白一个内存地址是一个变量(就象它是\[DeltaRegister+12345678h\]形式的一个内存地址)，把它储存在一个缓冲区，就象我们对代码标签所做的那样(但这次是内存地址)，弄乱它们的排列顺序，并为每个变量重新赋值一个地址。我们这样做，而不使用固定变量。唯一不方便的是，我们必须由其他地方提供某些值，比如由解密器(如果我们使用某个)。

也举个 LEA 的例子：如果我们把 LEA 仅仅用来重新编码 MOV Reg,Value、ADD Reg,Value、MOV Reg,Reg 和 ADD Reg,Reg，会比较好。为什么？因为如果你看到一个 LEA，意味着到了这里，因为它是更多一些指令的收缩，象 MOV EAX,EBX / ADD EAX,12345678h。所以，如果我们不用 LEA 编码，把这个任务分解成更简单的操作来实现(随机地)，那么我们相当于内含一个交换器模块。一个例子：

     MOV EAX,ECX                                              

     ADD EAX,3   -> (shrink) -> LEA EAX,\[ECX+3\] -> (expand) -> MOV EAX,3

                                                               ADD EAX,ECX

 

                                                   (expand) -> MOV EAX,ECX

                                                               ADD EAX,3

如果我们花力气在引擎某部分编写一个交换器，我们会发现这些情况正是所期望的，所以对于最终结果而言再编写一个显式的交换器是多余的。不管怎样，都能在展开时发生一个交换，但是小心，因为很多情况下－比如象下面这些－两个元素间的交换如果没有控制好就会破坏代码：

     MOV **EAX**,1234

     MOV **EBX**,2345  <-- check if a label is pointing to this!

 

     MOV **EAX**,1234

     MOV **EBX**,\[**EAX**\] <-- check if the second instruction uses the elements of

                       the first instruction

 

     MOV **EAX**,\[**EBX**\]

     MOV **EBX**,1234  <-- check if the first instruction uses the elements of

                       the second instruction

 

     MOV **EAX**,\[**EBX**\]

     MOV \[**ECX**\],**EDX** <-- **EBX** and **ECX** has the same value? We can't know this

                       without total emulation

还有很多很多。我的经验是，有太多因素使得，即使交换器只做了最微小简单的改变都能让代码崩溃，虽然它看上去完全正确。我们唯一可以安全地变异的是那些用 LEA 插入的及其后来解出来的东西。而且，我们又避免了一个大程序的编码 J。\[注：这里 mutate“变异”是指 swap“交换”\]

 e)        重新汇编器

重新汇编器是引擎的结束边缘。这块代码将生成能被处理器所理解的指令。如果我们能理解把代码展开成伪汇编语言的基本原理，这就是小菜一碟，因为我们在编写多态引擎时已经做过很多次了。而且，想象着自己就要完工的引擎会令你士气大增，相信我 J。

重新汇编器是一个友好的模块，容易得到其他模块辅助，比如展开器。我们可以把它编成照字面意义读取伪汇编码，直接把它的表示写出来，而不用管它的含意。

但是我们编码它的时候，发现有些东西不是看上去那么容易的，例如那些 EIP 转移指令。我们该如何编码那些向前的 JMP、CALL 和 Jcc 呢？我们必须(再一次)使用一个表，并且把所有指令储存进去，我们必须在伪码全部被汇编后再完成那些目的地地址。

但是真正棘手的是那些向前短跳转 JMP/Jcc。我们可以不用它，但这不合理 K。我的解决方案(非常简洁)是看指令是否指向前面 11 或 12 条指令的最大长度，所以如果低于那个分数，我们可以任意决定编码成短或长(当然，随机地)。对于向后的跳转我们没有问题，因为我们无例外地知道长度。那些 CALL 不必由我们决定，但是当它们向前的时候我们也必须解决它们，因为那里的代码还没被汇编。

引擎在这部分的随机性是由跳转(当我们能决定我们是使用短跳转还是长跳转)和操作码重新汇编(对于同一操作码我们有几个可能选择时)实现的。下面的列表显示一些指令，可以被这样处理：

  B0+Reg

  C6 C0+Reg   --> MOV Reg8,Value

 

  B8+Reg

  C7 C0+Reg   --> MOV Reg,Value

 

  50+Reg

  FF F0+Reg   --> PUSH Reg

 

  58+Reg

  8F C0+Reg   --> POP Reg

 

  40+Reg

  FF C0+Reg   --> INC Reg

 

  48+Reg

  FF C8+Reg   --> DEC Reg

这是一个例子。其他的就是那些能够使用 EAX 的指令(独占使用 EAX 的操作码或普通的操作码)，还有那些采用符号扩展字节到双字操作数(操作符 83，并使用\-80 到 7F 之间的数值)的操作码，等等。所有它们只是在操作码层次上的随机，因为所有操作的可选替代(复杂的移动之类)由展开器处理。

嘿，完工了！我们得到一段重新汇编过的代码！现在开始难的部分：调试。

 3)      已知问题(和解决方案)

 a)        调试你的引擎

如果你从未编写过这样一个引擎，你会觉得调试变形代码是地狱。在你调试第一代代码的时候感觉很好，因为你了解你做什么，但是当你必须调试生成的代码时问题就来了。汇编程序是如此如此地混乱，以致你都快要发疯了，因此解决办法是：从初期开始调试！直到所有其他部分完全工作以后才编码收缩器/展开器。

你必须考虑到的另一件事是，调试器使用 INT 3 补丁从那些 CALL 返回。如果你编写一个单独函数的反汇编码，在跟踪/单步时要留神，因为：

  MOV **EDI**,\[VirusEntry\]

  CALL Disassembly

  MOV **EAX**,12345678  *;--> The disassembler will see:  *

                                                    MOV **EDI**,\[VirusEntry\]

                                                    CALL Disassembly

                                                    INT 3

                                                    JS @xxx

                                                    ...

你会发现，调试器把代码破坏了。这是好的一面，因为代码隐含地反调试 J，但是不好的一面是如果你不知道这种情况，你会以为是某些地方出错，并且白花时间检查。解决办法是：

 1.         跟进到调用里面，并且跟踪所有的调试(觉得不可接受的那些地方)，或者

 2.         使用硬件断点(噢~！噼唎啪啦－鼓掌声)

因此，按下“运行到这里”键的时候要留神，因为如果你在运行反汇编器以后这样，它是 OK 的，但是如果在反汇编以前、而你指向的代码要在反汇编以后执行，那么它就会崩溃。在“抵达指令”上使用硬件断点将没有这个问题，但是更让人生气(需要按更多次键，等等)。把它做成一个宏。

 b)        API 调用

检测 API 调用真是苦不堪言。因为所有调用的参数个数都不同，我们不知道参数在什么点上开始(但是通过这个 CALL 我们知道 API 的结束点，啊，现在我们安详地休息了 L)。这类调用是脆弱的，在它们附近我们不能随意改变寄存器，因为返回值总是放在 EAX 中，而且它们总是修改 ECX 和 EDX。所以，在一个正常代码中，我们恰当地设置寄存器数值，没有使用这三个，但是当我们“自由变形”改变寄存器的时候\[注：原文“a go-go”\]，我们没办法不用这三个寄存器(否则，我们只能用其他寄存器玩玩，而这样使得可能选择的范围受到局限)。另外我们还必须检测什么时候 EAX 被使用以便及时保存返回值，限制使用这些寄存器引起许多问题，同我们的引擎格格不入。

容易的解决方案是使用一个“代码标记”：它们代表一个特定指令序列，用特定的操作数；它们在代码中不重复，除非它们标记的东西出现。我使用的那些指令序列名为 APICALL\_BEGIN、APICALL\_END 和 APICALL\_STORE。

APICALL\_BEGIN 只是 PUSH EAX/PUSH ECX/PUSH EDX。它会被收缩器发现，因为这个结构的每条指令和其他指令一样都能被展开。收缩器会检测到这个指令系列，并且把它们改成伪操作码 F4，这是一个标记，指示展开器编码 PUSH EAX/PUSH ECX/PUSH EDX。寄存器必须总是 EAX、ECX 和 EDX。这样，我们确保在 API 调用前保存这些寄存器内容。

同样地，APICALL\_END 是 POP EDX/POP ECX/POP EAX。这也被收缩器翻译成一个伪操作码 F5，作为标记指示展开器编码 POP EDX/POP ECX/POP EAX。这标志一个 API 调用结束后恢复保存的寄存器内容。因此，对这些寄存器来说，就好象从来没有执行过 API 调用一样。

APICALL\_STORE 是另一个伪结构。收缩器会检测它，在 API 调用之后马上取得一个 MOV \[Mem\],EAX。这样做是为了避免转换 EAX，所以这条指令将总是编码成 MOV \[Mem\],EAX，这样就不用管寄存器 EAX 必须被转换成什么。内存地址是储存返回数值的变量，可以在处理 APICALL\_END 时取出。

  

 

用下一个例子举例说明该技术的使用：

  PUSH **EAX**

  PUSH **ECX**

  PUSH **EDX**            -------------------> APICALL\_BEGIN

  MOV **EAX**,\[**EBP**+AddressOfNewDirectory\]

  PUSH **EAX**

  CALL DWORD PTR \[**EBP**+RVA\_SetCurrentDirectoryA\]

  MOV \[**EBP**+ReturnValue\],**EAX** -------------> APICALL\_STORE \[ReturnValue\]

  POP **EDX**

  POP **ECX**

  POP **EAX**             -------------------> APICALL\_END

  MOV **EAX**,\[**EBP**+ReturnValue\]

  CMP **EAX**,**EDX**          *; Get the return value and check it*

  JZ @X

  ...

现在改变寄存器：

  PUSH **EAX**

  PUSH **ECX**

  PUSH **EDX**            -------------------> APICALL\_BEGIN

  MOV **ESI**,\[**EBX**+AddressOfNewDirectory\]

  PUSH **ESI**

  CALL DWORD PTR \[**EBX**+SetCurrentDirectoryA\]

  MOV \[**EBX**+ReturnValue\],**EAX**   -----------> APICALL\_STORE \[ReturnValue\]

  POP **EDX**

  POP **ECX**

  POP **EAX**             -------------------> APICALL\_END

  MOV **ESI**,\[**EBX**+ReturnValue\]

  CMP **ESI**,**ECX**          *; Get the return value and check it*

  JZ @X

  ...

出于明显的原因，你绝对不能使用 EAX、ECX 或 EDX 作为病毒重新编码时的增量寄存器，因为如果你用了，那么 API 函数调用将会覆盖它，然后返回值也将不知被写入什么地方，因此有 99%的可能抛出一个异常。在此前提下(增量不是用 EAX、ECX 或 EDX)我们可以编码展开 CALL DWORD PTR \[Mem\]，我们打算用 MOV Reg,Mem/CALL Reg：我们使用 EAX、ECX 或 EDX 完美代替 Reg，不保存任何东西，因为寄存器数值将被 API 调用销毁。

  

 

至于 Linux(对于想在这个系统下变形的那些)API 调用就更复杂了，因为我们在寄存器(EAX、EBX、ECX 等)中传递参数。好，我们可以这样定义一个结构：

  MOV \[**EBP**+Parameter1\], XXX

  MOV \[**EBP**+Parameter2\], YYY

  MOV \[**EBP**+Parameter3\], ZZZ

  PUSH **EAX**

  PUSH **EBX**

  PUSH **ECX**

  PUSH **EDX**

  PUSH **EBP**       --------------> LINUX\_SYSCALL\_BEGIN DeltaReg

  MOV **EAX**,\[**EBP**+Parameter1\]

  MOV **EBX**,\[**EBP**+Parameter2\]

  MOV **ECX**,\[**EBP**+Parameter3\]

  MOV **EDX**,\[**EBP**+Parameter4\]  ---> LINUX\_SYSCALL\_LOADPARAM

  INT 80h

  POP **EBP**

  MOV \[**EBP**+ReturnValue\],**EAX**  --> LINUX\_SYSCALL\_STORE DeltaReg,\[ReturnValue\]

  POP **EDX**

  POP **ECX**

  POP **EBX**

  POP **EAX**        --------------> LINUX\_SYSCALL\_END

它更大，但我不是上帝 J。

有个东西虽然不是一个 API 调用，但也在这里用上了：SET\_WEIGHT 伪指令。这条指令对应下列代码结构：

  PUSH Reg1

  MOV Reg1,WEIGHT\_IDENT

  MOV Reg2,xxyyzztt

  MOV \[Mem\],Reg2

  POP Reg1

这被压缩成 SET\_WEIGHT \[Mem\],IDENT,Reg1,Reg2。这条指令被用于把其中数据从一代传送到另一代。我们只能用它传送这个，因为我们在全部重新汇编之前获取这些数值，所以我们不能用它取代栈传递参数。

其中 weights 代表一个小遗传算法的成分，让病毒成为“适应的”，因为“自然选择”将使幸存者做出最好的选择(感染的类型、解密器的结构、感染率，等等)以能生存。算法不是非常先进，但是确实有效。

 c)        内存

是的，变形作用确实附带产生大量内存需求，用来储存表、反汇编、临时改变的代码、执行路线标记、局部变量，以及很多等等东西。使用 ESP 是一个十分遗憾的技巧，对于储存不太大的东西可能有用，但是当你认识到需要大约 3 或 4Mb 来做一份象样的工作(只要看看 z0mbie 的 Mistfall：预留了 32Mb！)，情况就大大地不同了。唯一的解决办法是使用 VirtualAlloc 而且预留你需要的数量。

因为我们预留了内存，复制代码到内存再无阻碍，如同动态地生成代码一样。而且，(如果你在复制时解密)，在反病毒仿真器看来它是复制，而不是解密。因此，我们必须编码一个多态引擎至少生成一个“复制器”(仅当你不想加密代码时)。

复制到内存可以让我们做得更好：我们可以把代码复制到该内存框架内的任何地方，而且把内存变量放在我们希望的偏移地址。因为我们使用一个增量(Delta)寄存器，所以我们不需要关心代码在哪里！它的问题是取得我们的内部偏移量(另外只要关注通过操作系统动态取得的内存块首地址)。我们可以在重新汇编时重新编码所有缓冲区和表的地址，于是甚至连最终代码的“外形”都是不同的！这里的问题是我们必须提供代码被存放的地址，在预留的虚拟分配的内存中数据分区，等等，我们必须从唯一来源(解密器/复制器，但我们又可以随心所欲地变化)提供这些，例如我们制造一些数值入栈然后在引擎中一次把它们全部弹出栈(看看 MetaPHOR 源代码)。

 4)      未来

这部分是可选的。表达一些想法－－我正在考虑提高病毒 MetaPHOR 的功能性，对你的实现也许有用。

 a)        插件

有一天我肯定要在我的病毒中实现(我希望！)。我了解这类变形引擎的插件是很容易实现的(虽然“容易”不意味着“编码又少又快”)。插件能用“代码标记”实现，同样的方法我们用来表示 API 调用。我们的做法是制作一个信息头(它使用的变量等等)和一段用伪汇编语言写的程序(我们的汇编语言)。一个代码标记，表示一个插件，可以是“PUSH Reg/MOV Reg,12345678/POP Reg”(这是一段无用代码，但我们不让收缩器消除它)。事实上，通过那个寄存器传递的数字是一个插件 ID 号和一个版本号，(使用高字和低字)，这使我们可以比较当前安装的插件版本和一个新插件的版本，并且决定新插件是否必须代替旧的或者被加入执行代码。标记会被重复两次(一次用于开始，一次用于结束)，后一次其中数值的高位设置为 1，所以我们能用新的插件代替当前插件。收缩器必须检测这个结构，而且设置两个新指令，名为“PLUGIN\_BEGIN Version”和“PLUGIN\_END Version”。当收缩器做完它的任务，紧接在置换器之后，插件注入器马上开始工作：它搜索新插件，无论何处，解密它们并且检查它们的签名(象 Vecna 在他的 Hybris 中所做的)。如果你使用公\-私钥签名系统，插件不需要被加密，也几乎不可能修改或放入一个不属于你的插件。

依据插件的类型和版本，我们能直接替换插件入口(如果类型一致而且版本更高)，覆盖对前一插件的调用。这是一个 3D-编码技术\[注：原文 3D-coding，请参考前面的 3D instructions\]，我在文章开始不久提到过：旧插件的代码保留着，而且它被汇编成新病毒的组成部分，但是在下一代，病毒被反汇编时，反汇编器到不了那部分，所以旧插件被消除(同时，代码被混淆并与绝对无用的代码混合一起 J。

整个病毒全部能用插件组成，或者至少在所有程序中，甚至在插件装入器中，可以有插件 ID 号。既然那样，如果有一天我支持 Linux(我想在插件完成前就能 J，只须编码插件并把它放到相应地方。

而且，我们应该必须有一个字段指明我们想放一个新插件的地方(只是以防万一我们想在置换器之后展开器之前马上插入一个插件搞点新花样，或者只是我们忘记编码一个函数，现在我们想放入它)。这个字段就是插件 ID 号，而且它是简单的：仅仅是为原始病毒中的每个插件做一个唯一标识号，前后两个值分得很开，例如：反汇编器：10000001，收缩器：20000001，等等。(所有它们都有 ID 号和版本)。如果我们想把一个新函数放在反汇编器与收缩器之间，我们使用 ID 号 18000001，以此类推。

拭目以待吧，也许某一天我把它做出来 J。

 b)        多平台交叉感染

 

在 MetaPHOR 1.0 版我没有实现，因为要赶上 29A#6 发行就没时间做了。但它是很容易的(事实上，我们已经在 Benny 的 Winux 中看到其原理的实证)。我们只是必须根据运行它的系统调用不同的 API 函数，并且准备一个函数用于把我们的已变异代码附加到一个 PE 文件或是 ELF 文件。API 函数能以同样的方式被使用(例如，我编码一个叫 MapFile 的函数，它返回给我一个文件映像，在 Windows 操作系统平台上是使用 CreateFileMapping 和 MapViewOfFile，在 Linux 平台上是使用相关的 int 80h 功能)。

 c)        针对不同处理器重新汇编

我不知道是否要在插件之前编写它(这种情况下我宁可编一个插件也不重新写一个病毒)。

希望你已经认识到，伪汇编语言是非常一般化的，所以它能被改编以适应我们想要的任何处理器！当然，仅当它们使用 32 位体系架构才行。64 位体系架构要求改变内部伪汇编器所有的处理手段，但这不会很复杂(我只须扩充指令长度从 16 字节到 32 字节，让我有足够空间用于 QWORDs 和一些必被使用的新字段)。

我确实考虑过，并认为一定能做到，而且第一步可能是使用 Alpha 汇编器和 Alpha 处理器下的 WinNT。为什么？因为除了我使用的反汇编器和汇编器之外，我什么都不必改变，因为感染方法和算法将是完全相同的！(因为我们使用 ring 3 级)。收缩器、置换器、展开器等等，都将保持原状。

我们必须做的唯一一件事是，重新定义一些指令和这个汇编器下的展开，因为例如 TEST 指令不是所有的处理器都有(我想 Itaniums 中没有它)。

真正要做的第一件事是，编码一个新的反汇编器(但保留当前的)。所以我们只须知道，我们的代码是 Alpha 还是 x86，以便调用相应的程序。此外，因为我们只使用我们的内部汇编语言，所以到达再汇编器之前都是处在共同部分(算法方面，而非编码方面)，可以根据 PE 头中指明的目标处理器选择再汇编器(用于 Alpha 的还是用于 x86 的)。我们能使用的处理器类型很多(这就意味着大量的工作！)：x86、Alpha、Itanium、680x0、PowerPC、PA-RISC，等等。(所有能运行 WinNT 或 Unix/Linux、甚至 MacOS 的平台)。

最主要的是，代码完全被改变了：它不是象 Mr. Sandman 的 Esperanto 那样，一个病毒由几个部分组成、每部分用于一种处理器(向 Sandy 致敬 J，他的病毒是另一类代码)。整个病毒将重新汇编为新处理器的汇编语言，所以一个反病毒软件想为 x86 处理器拦截病毒，必须处理可能的到 Alpha 汇编器的转化，而另一方面，对于 Alpha 处理器又要转化到 x86，完全跳过反病毒软件(例如，在一个被传染的服务器上)。因为它是变形的，他们不能用字符串检测它。咂咂！J

 5)      结论

变形：古往今来最强的病毒技术。

还要补充吗？  
　

  4dd466d0e1e0cc123afbe504da2462ca
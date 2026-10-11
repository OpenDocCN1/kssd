
# 重建虚拟机

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-546.htm`](https://www.pediy.com/kssd/pediy09/pediy09-546.htm)

> 引用:
>
> 说明下, 由于文章有点长, 我是跳着翻译的. 还有我的五笔打字的时候可能会带字.
>
> 有什么问题希望大家说出来我修正.
>
> 感觉翻译得真糟糕. 有的好拗口哦
>
> 虚拟机是当今最有争议的保护技术之一。我试着通过检查 T2'06 挑战中所使用的虚拟机（价值 1500 美元）的完整逆向源代码来展现虚拟机怎么样被创造 的。解释怎样编写虚拟机以及为哪些想分析这个挑战 的人提供直接访问源代码和结构的帮助。 T2'06 的逆向源代码在附录里。
>
> **说明**
>
> 逆向虚拟机事实上并不需要完全知道虚拟机的结构。通常，有可能的话我们使用反汇编器来一次快速分析它的结构，然后调试看看活动的数据移动，它们是怎样适合我们的初始化描述，真正做了些什么。T2'06 是个小程序，我正好完整的逆向了它，奉献给你它的重建源代码。解决它并没有理由需要提供源代码。主要的理由就是和我以前的虚拟机教程有关。我收到很多的请求关于更复杂和更简单的事，由于没有理论支持很多好的逆向者靠近这个新技术还有问题。所以， 这篇短文直接讲这一点，提供关于虚拟机的逆向和深度观察虚拟机的结构，它们（RE）的编码以及创造。
>
> 作为基本的通向简单的 VM (虚拟机的缩略词) 的路径，我强烈建议你阅读以前我的关于这个主题的基础教程。在 Code-Breakers 杂志 (CBM) 你能找到它。请注意我建议你拥有基本的 IDA/WDASM 知识， 因为在这篇文章中，由于我将直接引用目标应用程序的代码，要么以它的逆向 C 形式，要么以它的汇编代码。 许多按原始代码完成的‘完整性检查’(原文'sanity checks') 在逆向 C 中被忽略了，因为它们对代码综合理解没有用处为了简短被忽略掉。 还有，注意我并没有调试 T2'06 挑战：我仅仅使用没有带调试器的 IDA 4.3 ¨C，也没有烦恼的运行 Olly 并行逆向它。1
>
> 说到这点，让我响起 MP3 的声音然后开始。
>
> **常规步骤, 结构**
>
> 在我打开 T2'06 几秒之后我就意识到我所面对的是一个虚拟机：你可能会问我我是怎么知道的。如果你仔细查看 \_main() 函数， 你能注意到一个普通的循环带有一个转发器，它调用一个函数要在函数列表里进行挑选。这听起来极像一个基本的 VM 核心。
>
> 虚拟机通常是围绕一个解码 和执行 VM 指令的主循环构建的。一般可以用这种方式 描绘出来：
>
> 代码:
>
> ```
> Virtual Machine Loop Start
> ...
> ...
> ...
> --> Decode VM Instruction's
> ...
> --> Execute the Decoded VM Instruction's
> ...
> ...
> --> Check Special Conditions
> ...
> ...
> ...
> Virtual Machine Loop End
> ```
>
> T2'06 是围绕一个虚拟机上下文结构构建的，这个结构持有虚拟机的寄存器以及一系列的引用和 VM 数据的指针。另一个重要的虚拟机结构 是用作解码器和 VM 指令接口的翻译层。这个层只是一个 解码器放置指令流解码出来的数据的结构，因此实现它们的函数能轻松的访问这些信息。
>
> 让我们快速浏览一下 VM 上下文结构，它是机器的核心：
>
> 代码:
>
> ```
> struct TVMContext {
>     int Register_IOType, // 通用寄存器，能持有 I/O 方向
>     int Register_IO, //
>     int Register_IOAddress, //通用寄存器，能寻址内存引用的地址
>     int Register_IOCount, // 通用寄存器，能字节传输传递
>     int GenericRegisters[12], // 其它的通用寄存器
>     int *Registers, //指向寄存器区域
>     int VM_ESP , // 虚拟堆栈指针
>     int VM_EIP , //虚拟指令指针
>     int VMEIP_Saved_Prior_InstrExec, // 操持跟踪当前执行的指令 IP
>     TVMFLAGS VM_EFLAGS, // VM 标志
>     int InstructionCounter, //指令数
>     int InitCode,
>     int MemorySize,
>     void* ProgramMemoryAddr,
>     int Original_ESP,
>     int StackMemorySize,
>     void* StackMemoryAddr,
>     int MachineControl, // 保持跟踪虚拟机状态
>     int VM_ResumeExec // 虚拟机的一种异常处理器
> }
> ```
>
> 虚拟机上下文包含一串通用的和特殊的寄存器，引用虚拟 IP 和 SP 的，标志和引用虚拟机虚拟内存 (这个虚拟机以它自己的寻址模式使用私有，虚拟内存空间) 的。
>
> 另一方面，虚拟机的解码器在解码阶段填充下面的结构 ：
>
> 代码:
>
> ```
> struct TInstructionBuffer{
>     int Length, // 指令长度
>     int InstructionData, // 通过查表传递
>     char InstrType,
>     //char Fillers1[3], //如果结构对齐 为 1
>     int Operand_Size,
>     char InstructionParamsCount,
>     //char Fillers2[3], //如果结构对齐为 1
>     SubInstr ParamDest,
>     SubInstr paramSrc,
>     SubInstr ParamThird,
>     SubInstr *WorkSubField // 指向 SubInstr 参数 (类似于“Registers[]”所做的)
> };
> ```
>
> 它包含主要的解码指令参数， 以及一些解码器用来填充的特殊数据。这两个结构被传递给虚拟机指令，对它们进行操作使虚拟机活动起来。
>
> 例如， 让我们设想一下 NOP 指令会被怎样实现。我们知道 NOP 指令的功能 ： 它简单的跳到代码流中的下一个指令 - 换句话说，它仅仅更新 EIP 指向下一条指令。
>
> 在这个虚拟机上， NOP 可以简单的实现为：
>
> `TVMContext.VM_EIP = TVMContext.VM_EIP + TInstructionBuffer.Length`
>
> 虚拟机的 IP 寄存器 得到待处理的指令长度增量，所以它会指向下一条指令 。
>
> VM 常常利用指令表来分派指令执行。正是这个观点 滚动代码- 带领我找到了机器码表 (而不是只滚动数据区域找\*\*的指针列表). 通常实现为一个索引的 jump 或者 call，但并不总是这样的：
>
> 代码:
>
> ```
> Opcode = *RealEIP;
> MachineCheck = (*OpcodeProc[(char)Opcode])(&InstBuff,&VMContext); // VM dispatcher
>
> 我们可以看到下面的 IDA 代码片断：
> Execute_VM_Opcode: ; CODE XREF: _main+13D#j
> .text:004021B8 0D4 mov edx, [esp+0D4h+VMInstructionBuff_VM_Opcode]
> .text:004021BF 0D4 lea eax, [esp+0D4h+VM_Context] ; Load Effective Address
> .text:004021C3 0D4 lea ecx, [esp+0D4h+VM_InstructionBuff_Body_Ptr] ; Load Effective Address
> .text:004021CA 0D4 and edx, 0FFh ; VM Opcode is 1 byte only
> .text:004021D0 0D4 push eax ; VM Context
> .text:004021D1 0D8 push ecx ; VM Instr Ptr
> .text:004021D2 0DC call VM_Opcode_Table[edx*4] ; Indirect Call Near Procedure
> .text:004021D2
> .text:004021D9 0DC add esp, 8 ; Add
> .text:004021DC 0D4 test eax, eax ; Logical Compare
> .text:004021DE 0D4 jz VM_Loop_Head_Default ; Jump if Zero (ZF=1)
> ```
>
> 你可能想知道我怎样解码前面代码中的“RealEIP”名字。 通过检查在 IDA 代码片断中用来跳转到 VM 机器码表的值你就知道了。 这个索引是按一个字节从一个地址中取到的。 明显地，这个字节区别哪条指令被执行，所以它只可能是指令机器码本身。 还有，它形似于 VM 指令表中的转发器使用的直接索引。
>
> 现在，我们正好需要标签出 VM 指令表并且 移动到虚拟的机器码，试着定位 VM 的 Context 对象，一些最常用的 VM 寄存器像指令指针， 一些指令等等， 直到我们开始获得清晰的映像。 下面你会看到从 IDA 来源中取出的这个结构的代码片断：
>
> 代码:
>
> ```
> .data:00407430 VM_Opcode_Table dd offset VM_MOV ; DATA XREF: _main+162#r
> .data:00407434 dd offset VM_Multiple_op2
> .data:00407438 dd offset VM_Multiple_op2
> .data:0040743C dd offset VM_Multiple_op2
> .data:00407440 dd offset VM_Multiple_op2
> .data:00407444 dd offset VM_Multiple_op2
> .data:00407448 dd offset VM_PUSH
> .data:0040744C dd offset VM_POP
> .data:00407450 dd offset VM_JMP
> .data:00407454 dd offset VM_CALL
> ... ...
> ```
>
> 这个 VM 的指令表包含一个很长的函数列表。让我第一眼就受打击的是事实上它们之间有很多重复的。这意味着一些机器码丢失了，或者多个 VM 指令在同一个函数过程中编码。所以， 我只标记出重复的哪些赋以临时的名字。然后，我开始寻找被这个表.引用的最简单(我认为’简单’是指它使用很少的内部调用)的 VM 指令。
>
> 通过扫描 VM 指令表，你能够快速地到达下面的实现：
>
> 代码:
>
> ```
> ; int __cdecl VM_NOP(int Param1,int Param2)
> .text:00401F80 VM_NOP proc near ; CODE XREF: _main+162#p
> .text:00401F80 ; DATA XREF: .data:0040746C#o ...
> .text:00401F80
> .text:00401F80 Param1 = dword ptr 4
> .text:00401F80 Param2 = dword ptr 8
> .text:00401F80
> .text:00401F80 000 mov ecx, [esp+Param1]
> .text:00401F84 000 mov eax, [esp+Param2]
> .text:00401F88 000 mov edx, [ecx]
> .text:00401F8A 000 mov ecx, [eax+Param2.Something]
> .text:00401F8D 000 add ecx, edx ; Add
> .text:00401F8F 000 mov [eax+Param2.Something], ecx
> .text:00401F92 000 xor eax, eax ; Logical Exclusive OR
> .text:00401F94 000 retn ; Return Near from Procedure
> .text:00401F94
> .text:00401F94 VM_NOP endp
> ```
>
> 不难理解它就是我们大名鼎鼎的 NOP 指令。我第一眼就选择它是因为它是一个令人非常喜欢的指令：它不调用内部函数，并且很短。正如你注意到它：
>
> 1. 接受两个参数：最可能其中之一就是 VM 上下文结构。
> 2. 取第一个参数值 ( mov edx,\[ecx\] ).
> 3. 从第二个参数一定偏移处取一个值 ( mov ecx, \[eax+Param2.Something\] ).
> 4. 把从参数一获得的值加到从参数二读到的值上
> 5. 保存前面的和，然后终止。
>
> 这个代码导致一些考虑： 我知道 虚拟 IP 必须 在某些地方被增加- 这一步能以几种方式完成。 用普通的 C 写的顺序：
>
> `Struct2.FieldX = Struct2.FieldX + Struct1.Field0`
>
> 看上去非常像
>
> `VMContext.VM_EIP = VMContext.VM_EIP + TInstructionBuffer.Length`
>
> 解决这个问题好的方式就是检查其它的指令然后看是否是周期性模式。在指令表中到处逛逛将会告诉你这个代码片断是很被使用的 ，通常在指令末。所以，你可以猜到它就是 我们的 VM EIP 增量。这也能给我们一个主意哪一个结构可能包含通用 VM 寄存器。
>
> 如果你比我聪明，你可能早已注意到包含 VM_EIP 的结构是分配在调用者的堆栈空间里，这意味着 VM 结构是作为\_main()函数的局部变量持有的。由于首先没注意到这点，我不得不重命名所有的 IDA \_main() 模块的字段来匹配 恢复的 VM 结构名。顺便说下，这让 \_main() 函数更加难于理解 。
>
> 解码 VM_NOP 指令帮助我 恢复了下面的元素：
>
> 1. VM Context：因为我们知道持有它的参数，所有能在 main 中原路找到它并且标签其它指令适当的参数。
> 2. 虚拟 IP (VMContext.VM_EIP)：因为它得到在 NOP 指令中的增量 。
> 3. 解码出的指令放置的区域。如果你检查其它的 VM 指令， 你能注意到第二个参数是作为一个结构来引用的。既然 VM 机器码转发器在所有指令中都一样并且传递给它们非常相似的参数，我们能猜到(但还不确定!)它就是一个结构。
>
> 然而，这只不过是冰山一角。 复杂的事来了， 尤其是如果灵感(原文是 Zen)没有及时帮助我们 (因为我正好遇到，需要更多的逆向时间). 我们可能移动到各种不同的方向了。例如，我们可能检查什么过程在机器码转发器之前被调用 ： 指令的解码器一定就是这了，因为它们没有在每条指令内运行时解码。我选择了深度检查指令集，这是我在这个过程中学习到的虚拟机知识。我希望它不仅仅是帮助你理解 T2'06 VM，而是给你一些分析它的思维方式 (我认为需要约 6 个工作日对于一般的逆向而言， 4-6 个工作日用于 C 重写以及写文章，嗨!)。
>
> 在分析这个 VM 过程中，我试着定位 Jump(s) 指令：它对于得到虚拟标志结构能很有用。 VM_JCC 定位于 .text:00401C80。它能轻易地被认出是 JCC 因为它执行一些条件测试，此外，它通过增加或者改变我们的 VM_EIP 来响应测试结果。 VM Jcc 是个有诸多意义的指令，因为在 VM 指令表中它跨越超过一个机器码。
>
> 滚动代码，我们能看到很多的指令调用内部函数来做许多未知的工作。当分析刚开始的时候离它们远点是个好主意，集中精力于更简单的函数。 然而这并不是说我们应该断绝关系去在哪些函数上寻找令人感兴趣的模式！
>
> 例如，我们能找到一个执行许多数学操作的复杂指令 ，在字节/字/双字的基础上作出区分。 就是最初我称做 VM_MultipleOp2 的操作码。这个指令最有趣的一点 就是下面的特色代码：
>
> 代码:
>
> ```
> case 2: // XOR
> switch(DecodedInstr->OperandSize) {
>     case 1: VMValueEval = (char)VMValueSrc ^ (char)VMValueThird; break;
>     case 2: VMValueEval = (word)VMValueSrc ^ (word)VMValueThird;break;
>     case 4: VMValueEval = VMValueSrc ^ VMValueThird;
> }
> ```



## 指令分析与逆向工程

你能注意到，它检查模式值（1-2-4）并且根据字节数大小做出相应的动作，在不同的数据大小上完成同样相关的操作。当看到这个模式，你能轻易地识别出来它的用法，所以你能发现指令的用法和参数。明显地，上面代码中提供的参数名是在对这个机器有了很好的了解之后的，但是你能注意到即使在最初的逆向步骤中这个参数（`DecodedInstr->OperandSize`）是用来辨别操作数大小的，并且你还能看到代码的结构是用来对多个操作数大小执行异或操作。在 `IDA` 中看起来像这样：

### 代码

```
VM_XOR_case_multi_3: ; CODE XREF: VM_Multiple_op2+70#j   .text:00402336 ; DATA XREF: .text:004026E8#o   .text:00402336 
014 mov eax, [ebx_is_InstrBuf+VM_InstrBuffer.Operand_Size]   .text:00402339 
014 dec eax ; Decrement by 1   .text:0040233A 
014 jz short loc_40236E ; Jump if Zero (ZF=1)   .text:0040233A 
   .text:0040233C 
014 dec eax ; Decrement by 1   .text:0040233D 
014 jz short loc_402355 ; Jump if Zero (ZF=1)   .text:0040233D 
   .text:0040233F 
014 sub eax, 2 ; Integer Subtraction   .text:00402342 
014 jnz Finalize_Instruction_end_of_case_0Ch ; Jump if Not Zero (ZF=0)   .text:00402342 
   .text:00402348 
014 mov eax, [esp+14h+Hold_34h_param]   .text:0040234C 
014 mov esi, edi_is_Param_24h   .text:0040234E 
014 xor esi, eax ; Logical Exclusive OR   .text:00402350 
014 jmp Finalize_Instruction_end_of_case_0Ch ; Jump   .text:00402350 
   .text:00402355 
; -------------------------------------------------------------------------   .text:00402355 
   .text:00402355 
loc_402355: ; CODE XREF: VM_Multiple_op2+12D#j   .text:00402355 
014 mov esi, [esp+14h+Hold_34h_param]   .text:00402359 
014 mov ecx, edi_is_Param_24h   .text:0040235B 
014 and esi, 0FFFFh ; Logical AND   .text:00402361 
014 and ecx, 0FFFFh ; Logical AND   .text:00402367 
014 xor esi, ecx ; Logical Exclusive OR   .text:00402369 
014 jmp Finalize_Instruction_end_of_case_0Ch ; Jump   .text:00402369 
   .text:0040236E 
; -------------------------------------------------------------------------   .text:0040236E 
   .text:0040236E 
loc_40236E: ; CODE XREF: VM_Multiple_op2+12A#j   .text:0040236E 
014 mov esi, [esp+14h+Hold_34h_param]   .text:00402372 
014 mov edx, edi_is_Param_24h   .text:00402374 
014 and esi, 0FFh ; Logical AND   .text:0040237A 
014 and edx, 0FFh ; Logical AND   .text:00402380 
014 xor esi, edx ; Logical Exclusive OR   .text:00402382 
014 jmp Finalize_Instruction_end_of_case_0Ch ; Jump
```

然而，这个发现对 `VM` 深度的了解还是不足够的。我使用的是 `IDA 4.3`，因此我不能够调试，就意味着我不知道在这个片断前后的代码做些什么，特别是哪些调用一些函数的。并不是大问题，我可以使用更好的调试器 `Olly`，但是我并不想哪样做。这让逆向更加复杂，但也更加的有趣。因此，让我们总结下通过检查代码对这个 `VM` 所发现的：

1. 我们知道 `VM` 上下文结构在哪里，并且至少知道它的一个字段：`VM_EIP`。
2. 我们知道解码 `VM` 指令放在哪，以及指令字节长度存放位置在这个结构的第一个字段里。
3. 我们发现 `VM` 解码器结构的子字段操作数大小（`OperandSize`），通过定位看上去像对不同大小的数据执行 `XOR` 的代码。

其它有趣的指令在开始分析时我们可能会检查 `VM_DEC` 和 `VM_INC`。当然，通过使用 `...inc(x)` 和 `dec(x)` 可以漂亮地识别出来。还有，`VM_NOT` 也可以用这种方式找出来。

但是，迟早我们必须挖掘出真正的 `VM` 暗礁。所以，明智地返回来检查 `VM` 主循环，尝试着定位和理解它的函数过程。如果查看 `_main()` 过程，你能注意到在操作码执行之前它精心制作了一个调用... 你能猜到这就是 `VM` 指令解码器。

### 代码

```
.text:00402196 
0D4 mov eax, [esp+0D4h+RealVMAddr__and_decoded_VMEIP]   .text:0040219A 
0D4 lea ecx, [esp+0D4h+VM_InstructionBuff_Body_Ptr] 
   .text:004021A1 
0D4 push eax 
   .text:004021A2 
0D8 push ecx 
   .text:004021A3 
0DC call VMInstructionDecoder ; Call Procedure   .text:004021A3 
   .text:004021A8 
0DC add esp, 8 ; Add   .text:004021AB 
0D4 test eax, eax ; Logical Compare   .text:004021AD 
0D4 jnz short Execute_VM_Opcode ; Jump if Not Zero (ZF=0)
   if (!MachineCheck) {
      MachineCheck = VMInstructionDecoder(&InstBuff,RealEIP);
      if (!MachineCheck) // check for opposite behavior...
```

跟随我们的分析，原路返回 `_main()` 函数，一直到在解码器之前被调用的小函数。让我们看看主函数中动作的一般顺序：其中一个函数接受 `VM_EIP` 地址并靠着 2 个块边界测试它。然后，跟随它到内存地址。

我猜想：其中一块代表 `VM` 内存空间，另一块是 `VM` 堆栈空间。地址是这个函数的一个一般参数，函数接受一个虚拟地址并且靠 `VM` 内存空间对它进行测试以保证正确的解释。当然我是对的。

所以，即使在主函数中真正的调用，我们把参数看作接收虚拟 `EIP`，函数接受任何一般的 `VM` 指针并且找到它所引用的真正的 `x86` 指针。这样的话，我们揭示出了用来在内存中找到指针所指内容的函数：

### 代码

```
bool VMAddress2Real(TVMContext *VMContext,int VMAddress,int *RealAddr) { // .text:004011D0
   if( RANGE(VMAddress,VMContext->InitCode,VMContext->MemorySize) ) {
      *RealAddr = (VM_Address-VMContext->InitCode)+VMContext->ProgramMemoryAddr;
      return 1;
   }
   if( RANGE(VMAddress,VMContext->Original_ESP,VMContext->StackMemorySize) ) {
      *RealAddr = (VM_Address-VMContext->Original_ESP)+VMContext->StackMemoryAddr;
      return 1;
   }
   VMContext->MachineControl = mcWrongAddress;
   return 0;
}
```

通过检查代码你可以注意到，有一个机器控制（`MachineControl`）寄存器我们还未揭开它的庐山真面目，以及它的一个状态。这个寄存器在许多错误条件出现的地方（这意味着我在作出假设之前前后对照它!）被设置，所以对我来说把它和机器控制器配对是很自然的事。注意它不仅仅是个错误状态寄存器，因为它用来表明其它的机器条件不同于错误：例如，`VM` 向外界输入/输出是使用机器控制值来作为信号的，我们可以从下面的实现中看出：

### 代码

```
// :00401F60 VM_ALLOW_IO
int __cdecl VM_ALLOW_IO(TVMContext* VMContext, TInstructionBuffer* DecodedInstr) {
   VMContext->MachineControl = mcInputOutput;
   NextInstr(VMContext,DecodedInstr);
   // very ugly: for having IO you must force a MachineControl check, as if error were in.
   return 1;
}
```

上面的 `C` 代码相应于下面的：

### 代码

```
:00401F60 VM_ALLOW_IO proc near ; CODE XREF: _main+162#p   .text:00401F60 ; DATA XREF: .data:0040752C#o   .text:00401F60 
   .text:00401F60 
arg_0 = dword ptr 4   .text:00401F60 
arg_4 = dword ptr 8   .text:00401F60 
   .text:00401F60 
000 mov eax, [esp+arg_4]   .text:00401F64 
000 mov ecx, [esp+arg_0]   .text:00401F68 
000 mov [eax+VM_Context.maybe_MachineControl], mcInputOutput   .text:00401F6F 
000 mov edx, [ecx]   .text:00401F71 
000 mov ecx, [eax+VM_Context.VM_EIP]   .text:00401F74 
000 add ecx, edx ; Add   .text:00401F76 
000 mov [eax+VM_Context.VM_EIP], ecx   .text:00401F79 
000 mov eax, 1   .text:00401F7E 
000 retn ; Return Near from Procedure   .text:00401F7E 
   .text:00401F7E 
VM_ALLOW_IO endp
```

#### 个人注释

我并不欣赏这个 `VM` 创造者使用的这个解决方案。我认为这是个不雅的解决方案，因为它对错误和非错误使用了相同的检查，并且需要指令返回错误代码（`'1'` 值）来正确处理不同的机器状态。下面的片断显示这个机器怎么样执行机器控制检查：

### 代码

```
.text:004021E4 
0D4 cmp [esp+0D4h+Ctx_var_54_zeroed_on_loop_head_R70_MachineControl], edi_mcInputOutput ; Compare Two Operands   .text:004021EB 
0D4 jnz VM_MachineErrCheck_OrEndOfVM ; jump to test if we need NOT to read/write output!   .text:004021EB 
   .text:004021F1 
0D4 lea edx, [esp+0D4h+VM_Context] ; Load Effective Address   .text:004021F5 
0D4 push edx ; VM_Context_Ptr   .text:004021F6 
0D8 call CheckForInputOutput ; Call Procedure
   if (MachineCheck && VMContext.maybe_MachineControl==c2) { // VM loop end. c2==mcInputOutput
      CheckForInputOutput(&VMContext);
      continue;
```

原始的应用程序看上去并未和我使用一样的机器检查（原文 “MachineCheck”）。`_main()` 里的代码让我对原始代码想了很多。我想据我看来，我希望这是 `#defining` 外部代码的结果，并且没有 `goto` 语义出现。哪代表一个糟糕的设计（和编码）选择。

继续分析这个虚拟机，我们能够试着定位哪些看上去引用堆栈结构的指令，希望我们的初始猜想和现有的堆栈是符合的：这样的话，改变 `VM_ESP` 的指令和管理堆栈的就出来了。在细节上我们可能不会明白 `VM` 指令用堆栈来做什么，因为它们调用了需要我们了解的内部过程，但是至少我们知道它们产生什么作用，适当地标注它们。定位 `CALL/RET` 这对指令不到我们开始检查内部函数不是很容易的，因为通过检查下面的 `VM_RET` 指令你就明白了:

### 代码

```
（此处原文未提供具体代码块内容，保留标题以维持结构完整性）
```



# 虚拟机分析

## 1. VM_RET 指令分析

```c
int __cdecl VM_RET(TVMContext* VMContext, TInstructionBuffer* DecodedInstr) {
    int VMValue;
    Read_VMMemory_To(VMContext->VM_ESP, &VMValue);
    VMContext->VM_ESP -= 4;
    VMContext->VM_EIP = VMValue;
}
```

```asm
.text:00401EC0 VM_RET proc near ; CODE XREF: _main+162#p
; DATA XREF: .data:0040745C#o
.text:00401EC0
.text:00401EC0 VMCOntext_Ptr = dword ptr 0Ch
.text:00401EC0
.text:00401EC0 000 push esi
.text:00401EC1 004 mov esi, [esp+VMCOntext_Ptr]
.text:00401EC5 004 push esi ; vm_context_ptr
.text:00401EC6 008 lea eax, [esp+4+VMCOntext_Ptr] ; Load Effective Address
.text:00401ECA 008 mov ecx, [esi+VM_Context.VM_ESP]
.text:00401ECD 008 push 4 ; AddressDataSize
.text:00401ECF 00C push eax ; Write_VMValue_in_LE_At
.text:00401ED0 010 push ecx ; VMAddress
.text:00401ED1 014 call Read_VMMemory_To ; was Set_RealAddress_To
.text:00401ED1
.text:00401ED6 014 add esp, 10h ; Add
.text:00401ED9 004 test eax, eax ; Logical Compare
.text:00401EDB 004 jnz short loc_401EE4 ; Jump if Not Zero (ZF=0)
.text:00401EDB
.text:00401EDD 004 mov eax, 1
.text:00401EE2 004 pop esi
.text:00401EE3 000 retn ; Return Near from Procedure
.text:00401EE3
.text:00401EE4 ; ------------------------------------------------------------
.text:00401EE4
.text:00401EE4 loc_401EE4: ; CODE XREF: VM_RET+1B#j
.text:00401EE4 004 mov eax, [esi+VM_Context.VM_ESP]
.text:00401EE7 004 mov edx, [esp+VMCOntext_Ptr]
.text:00401EEB 004 add eax, -4 ; Add
.text:00401EEE 004 mov [esi+VM_Context.VM_EIP], edx
.text:00401EF1 004 mov [esi+VM_Context.VM_ESP], eax
.text:00401EF4 004 xor eax, eax ; Logical Exclusive OR
.text:00401EF6 004 pop esi
.text:00401EF7 000 retn ; Return Near from Procedure
.text:00401EF7
.text:00401EF7 VM_RET endp
```

恢复从虚拟堆栈来的值是由一个特殊的过程完成的，并且它处理这个 `VM` 分配的虚拟内存。

不用想也知道，逆向这种指令问题多多。尽管如此，如果你看得够仔细，可以注意到这个被调用的函数返回值是放在 `EIP` 中。既然函数似乎是对我们可能的 `VM_ESP` 操作，甚至不用知道内部函数的动作你就可以开始有点想像(还有，我们可能注意到假想的堆栈使用一种不同的长大方向！)。

我用其它的方式：直接进攻内部函数，看它们到底做些什么。在分析过程中，当整个画面几乎失踪时，我不明白指令中 `DEC EAX` 开始于 `text:00401E10` 的原因。几分钟后我放弃了它。你也许希望看一看它并试图猜猜这个指令做什么——这也许是个不错的训练。

我希望通过突出在我分析中出现的两段来结束这个 `VM` 的一般介绍。

## 2. 内存初始化与字节序

第一个就是在刚开始分析时有件事让我感到模糊，初始化值是在哪里赋值给堆栈和应用程序区域的。例如，堆栈是从 `0x80000000` 开始，我感觉这是个非常古怪的值。在这我也没什么灵感，所以我被迫用其它方式寻找答案。

四处逆向，我来到了下面的代码片断：

```asm
.text:00401261 004 jnz short loc_4012AD ; here below, operandsize is 4
.text:00401261
.text:00401263 004 mov ecx, ebx_param_vmvalue
.text:00401265 004 mov edx, ebx_param_vmvalue
.text:00401267 004 mov eax, ebx_param_vmvalue ;
; this code simply swap ebx bytes
; from 4321 Little endian to 1234 big endian,
; and write VMAddress2RealAddr the BE value
.text:00401269 004 and ecx, 0FF0000h ; take 3rd byte
.text:0040126F 004 shr edx, 10h ; Shift Logical Right
.text:00401272 004 and eax, 0FF00h ; take 2nd byte
.text:00401277 004 or ecx, edx ; Logical Inclusive OR
.text:00401279 004 mov edx, [esp+4+Ptr_ValueToWriteAndSwap]
.text:0040127D 004 shl ebx_param_vmvalue, 10h ; Shift Logical Left
.text:00401280 004 or eax, ebx_param_vmvalue ; Logical Inclusive OR
.text:00401282 004 pop ebx_param_vmvalue
.text:00401283 000 shr ecx, 8 ; Shift Logical Right
.text:00401286 000 shl eax, 8 ; Shift Logical Left
.text:00401289 000 or ecx, eax ; Logical Inclusive OR
.text:0040128B 000 mov eax, 1
.text:00401290 000 mov [edx], ecx
.text:00401292 000 retn ; Return Near from Procedure
```

如果你注意到，这个代码把双字的字节从小端交换到大端(反之亦然)。在我找到（并理解了）这个代码，它马上让我想到 `0x80000000` 值不同的画面。它使用大端法书写，而不是小端法！在我刚见到它时非常的郁闷因为我想不明白。好了，毕竟人无完人。

最后一点我要强调的是这个虚拟机的寻址模式。和常见的处理器一样，这个 `VM` 使用不同的寻址模式——例如直接寻址，寄存器寻址等。

用虚拟机的跳转指令作为参考来理解指令内值的使用方式是个好办法：你能肯定地找到实际的寻址模式：如果检查 `VM_JMP` 指令，你能看到虚拟机 `EIP` 是怎样被改变，以及 `VM` 至少使用部分寻址模式来解码。例如，通过检查 `VM` 跳转指令，我们能看到参数测试，如果测试成功，它被加到当前的 `EIP` 上。这听起来难道不像按位移跳转？用你的双眼看看：

```asm
.text:00401D79 loc_401D79: ; CODE XREF: VM_JMP+1F#j
.text:00401D79 008 cmp [esi+SubInstr.AddressType], vmaVMValue_orC4__or_displacement ; Compare Two Operands
.text:00401D7C 008 jnz short make_jmp ; Jump if Not Zero (ZF=0)
.text:00401D7C
.text:00401D7E 008 mov edx, [esp+8+InstrBuf_Then_Addr_WriteTo] ; relative jump!
.text:00401D82 008 mov eax, [edi+VM_Context.VM_EIP]
.text:00401D85 008 add eax, edx ; Add
.text:00401D87 008 mov [edi+VM_Context.VM_EIP], eax
.text:00401D8A 008 pop edi
.text:00401D8B 004 xor eax, eax ; Logical Exclusive OR
.text:00401D8D 004 pop esi
.text:00401D8E 000 retn ; Return Near from Procedure
.text:00401D8E
.text:00401D8F ; ------------------------------------------------------------
.text:00401D8F
.text:00401D8F make_jmp: ; CODE XREF: VM_JMP+2C#j
.text:00401D8F 008 mov eax, [esp+8+InstrBuf_Then_Addr_WriteTo]
.text:00401D93 008 mov [edi+VM_Context.VM_EIP], eax
.text:00401D96 008 pop edi
.text:00401D97 004 xor eax, eax ; Logical Exclusive OR
.text:00401D99 004 pop esi
.text:00401D9A 000 retn ; Return Near from Procedure
.text:00401D9A
.text:00401D9A VM_JMP endp
```

所以，我最后快速扼要地陈述下进攻 `VM` 的方式：

1. 我们已经定位到循环体，找到了转发器。它给我们提供了操作码表。
2. 我们随机检查 `VM` 操作码函数寻找简单的函数和周期性的代码模式。
3. 我们定位 `NOP`，发现了虚拟机上下文(`VMContext`)结构，指令解码器(`InstructionDecoder`)结构，一些上下文和解码器的字段。
4. 我们检查一个操作不同大小数据的指令，解码出了操作数大小(`operand_size`)字段。
5. 发现了可能的堆栈用法。
6. 发现了虚拟内存的寻址方式。
7. 发现了操作码寻址模式。

在对这个 `VM` 做完整分析之前的确有许多的工作已经做了，但这些步骤足以为逆向提供相当好的知识。下一节将会描述这个虚拟机各个部分更多的细节，主要使用它的逆向 `C` 代码。通常，这些函数的引用地址已经给出来了，因为这很有用，打开你的调试器/反汇编器看看汇编代码是什么样子。

## 3. 虚拟机主体

虚拟机通常是围绕一个上下文构建的，上下文是机器的寄存器和参数分配的地方。`T2'06` 也不例外：这是这个虚拟机使用的内存空间：

```c
struct TVMContext {
    int Register_IOType,
    int Register_IO,
    int Register_IOAddress,
    int Register_IOCount,
    int GenericRegisters[12],
    int *Registers,
    int VM_ESP,
    int VM_EIP,
    int VMEIP_Saved_Prior_InstrExec,
    TVMFLAGS VM_EFLAGS,
    int InstructionCounter,
    int InitCode,
    int MemorySize,
    void* ProgramMemoryAddr,
    int Original_ESP,
    int StackMemorySize,
    void* StackMemoryAddr,
    int MachineControl,
    int VM_ResumeExec
};

struct TVMFLAGS {
    // ~Compiler Dependent~ -please check the order!!
    ZF:1, // compiler-supposed Bit 0
    CF:1, // compiler-supposed Bit 1
    OF:1, // compiler-supposed Bit 2
    SF:1, // compiler-supposed Bit 3
    Unused:3, // compiler-supposed Bit 4-6
    TF:1 // compiler-supposed Bit 7
};
```

这个 `VM` 某些字段是特殊的，但是我们能看到通用字段：一序列的特殊和一般目的的寄存器，执行和堆栈指令 (就是 `IP` 和 `SP`)，机器的标志，以及其它的标志特别是内存空间地址和堆栈空间。最后个字段 `ResumeExec` 很有趣。这个字段用作一种异常处理器并且它也作为调试目的(几乎所有的调试代码都被 `T206` 移除了，但是你能通过遗留下的东西恢复，例如最明显的陷阱标志检查)。

最前面的寄存器哪样命名，因为它们用作 `IO` 目的。并不是它们的唯一用法，只是它们的特殊用法。只有它们得到地址一旦 `VM_ALLOW_IO` 指令 (已经显示出来了)允许 `IO` 才这样用。

```c
int __cdecl CheckForIO(VM_Context) { // .text:00402040
    switch(VMContext.Register_IOType) {
        case 2:
            return Do_Write_Output(VM_Context);
        case 3:
            return Do_Read_Input(VM_Context);
        default:
            return 1;
    }
}

int __cdecl Do_Write_Output(TVMContext* VMContext) { // .text:00401FF0
    int NumberBytesToWriteOut;
    void *BufferToWrite;

    if (VMContext->Register_IO != 0) return 0;
    VMAddress2Real(VMContext, VMContext->Register_IOAddress, &BufferToWrite);
    NumberBytesToWriteOut = VM_Context->Register_IOCount;
    VM_Context->Register_IOCount = write(stdout, BufferToWrite, NumberBytesToWriteOut);
    return 1;
}

int __cdecl Do_Read_Input(TVMContext* VMContext) { // .text:00401FA0
    int NumberBytesToReadIn;
    void *BufferToRead;

    if (VMContext->Register_IO != 0) return 0;
    VMAddress2Real(VMContext, VMContext->Register_IOAddress, &BufferToRead);
    NumberBytesToReadIn = VM_Context->Register_IOCount;
    VM_Context->Register_IOCount = read(stdin, BufferToRead, NumberBytesToReadIn);
    return 1;
}
```



> 我怎么能断言这些寄存器是按其它方式使用？当然，你可能会想，一些东西必须给它们填充值，不？ 这点很好，但理由还是更值得推敲。 `TVMContext.Registers[]` 指针字段是使用 `TVMContext` 结构头来初始化的。这意味着一般指令通过 `TVMContext.Registers[]` 引用寄存器就能自由访问这个通用目的寄存器。

> 现在让我们详细地检查这个 VM 的主函数，为了明白它怎样从头开始工作：

> 代码:

```
MemorySize = 4096;
initStack = SWAP(1);
initCode = SWAP(0x6EEFF);

byte * program;
int * OpcodeProc[];

int main() {
    dword RealEIP;
    TVMContext VMContext;
    TInstructionBuffer InstBuff;
    int res, MachineCheck;
    int c1, c2;
    char Opcode;

    /* 1. initialize VM */
    memset(VMContext, 0, 30*4);
    VMContext.Registers = &VMContext;
    if (*program != 0x102030) exit(1);

    //.text:004020A1
    VMContext.ProgramMemoryAddr = malloc(MemorySize + 16);
    if (VMContext.ProgramMemoryAddr == 0) exit(1);

    VMContext.InitCode = initCode;
    VMContext.MemorySize = MemorySize;
    memcpy(VMContext.ProgramMemoryAddr, program, 2580 + 1);

    VMContext.StackMemoryAddr = malloc(MemorySize);
    VMContext.StackMemoryInit = initStack;
    if (VMContext.StackMemoryAddr == 0) exit(1);

    //.text:00402111
    VM_EIP = initApp + 28;
    VMContext.StackMemoryAddr = MemorySize;
    VMContext.VM_ESP = VMContext.StackMemoryInit;

    c1 = mcGenericError_or_CannotWriteTo;
    c2 = mcInputOutput;

    /* 2. start main VM Loop */
    while (true) {
        // .text:00402138
        // VM_Loop_Head_Default: .text:0040215B
        VMContext.InstructionCounter++;

        if (VMContext.VM_EFLAGS == TF) {
            // Step-flag for debugging purposes (code removed)
            VMContext.MachineControl = mcStepBreakPoint;
        } else {
            //--->body<--- .text:00402177
            VMContext.VMEIP_Saved_Prior_InstrExec = VM_EIP;

            /* 3. process a VM Instruction and execute it */
            MachineCheck = VMAddress2Real(&VMContext, VM_EIP, &RealEIP);
            if (!MachineCheck) {
                MachineCheck = VMInstructionDecoder(&InstBuff, RealEIP);
                if (!MachineCheck) {
                    // check for opposite behavoir...
                    VMContext.MachineControl = c1;
                } else {
                    Opcode = *RealEIP;
                    MachineCheck = (*OpcodeProc[(char)Opcode])(&InstBuff, &VMContext);
                    if (!MachineCheck) continue; // check for opposite behavoir...
                }
            }

            /* 4. if we have a Machine-Check to do, ensure to catch the 'I/O' one */
            if (MachineCheck && VMContext.maybe_MachineControl == c2) {
                // VM loop end. c2==mcInputOutput
                CheckForInputOutput(&VMContext);
                continue;
            }

            /* 5. perform the exception check */
            // VM_MachineErrCheck_OrEndOfVM:
            if (VMContext.VM_ResumeExec == 0) {
                return 0;
            }
            VM_EIP = VMContext.VM_ResumeExec;
            VMContext.VM_ResumeExec = 0;
        }
    };
    // end....
```

> 让我们一点一点的注释(请注意我已经重新整理和重构了一些代码，因为我不喜欢 T2'06 代码中看着比较麻烦的结构)：

1. 初始化 VM：这部分代码简单的分配内存，拷贝 VM 程序和初始化起始值，没什么好说的。
2. 开始 VM 主循环：这是虚拟机的核心：这里我们对指令计数，测试特殊条件 (例如陷阱标志，I/O 请求，代码流异常)。在 VM 循环里我们转换虚拟 EIP 为 x86 地址并且在这个地址读取和解码虚拟指令。
3. 处理 VM 指令并执行它：如果 `VM_EIP` 转换和指令解码正确，我们处理 VM 指令，喂给它虚拟机的上下文和一个持有解码出指令数据的缓冲区。
4. 如果我们需要做机器检查，确保抓住'I/O'：机器检查并不总是出错：所以，检查特殊条件，如 I/O 请求。
5. 执行异常检查：如果由于代码流或者错误使我们到达异常检查，`resume_exec` 寄存器会被测试：如果为空，程序终止。

在详细地检查 VM 指令管理之前，最好花点时间在 VM 内存管理上面。如前所说的，VM 内存是大端顺序的。所以，在小端机器上，我们需要回来转换所有移动到这的值。这个 VM 为这个步骤实现两个函数，`Read_VMMemory_To()` 和 `Write_VMMemory_From()`。让我们检查它们的实现，为了对它们的工作获得一些想法：

> 代码:

```
.text:00401230 ; int __cdecl Write_VMMemory_From(int VMAddress, int Ptr_ValueToWriteAndSwap, int VMValue_OperandSize, int vm_context_ptr)
.text:00401230 Write_VMMemory_From proc near
    ; CODE XREF: Write_VMValue_To_Param+CB#p
    ; VM_PUSH+3D#p ...
    .text:00401230
    VMAddress = dword ptr 4
    Ptr_ValueToWriteAndSwap = dword ptr 8
    VMValue_OperandSize = dword ptr 0Ch
    vm_context_ptr = dword ptr 10h

    .text:00401230 ebx_param_vmvalue = ebx

    .text:00401230 000 mov eax, [esp+Ptr_ValueToWriteAndSwap]
    .text:00401234 000 mov edx, [esp+VMAddress]
    .text:00401238 000 push ebx
    .text:00401239 004 lea ecx, [esp+4+Ptr_ValueToWriteAndSwap] ; Load Effective Address
    .text:0040123D 004 mov ebx_param_vmvalue, [eax]
    .text:0040123F 004 mov eax, [esp+4+vm_context_ptr]
    .text:00401243 004 push ecx ; Write_RealAddr_To
    .text:00401244 008 push edx ; VM_Address
    .text:00401245 00C push eax ; vm_context_ptr
    .text:00401246 010 call VMAddress2Real ; Call Procedure
    .text:00401246
    .text:0040124B 010 add esp, 0Ch ; Add
    .text:0040124E 004 test eax, eax ; Logical Compare
    .text:00401250 004 jnz short loc_401254 ; Jump if Not Zero (ZF=0)

    .text:00401250
    .text:00401252 004 pop ebx_param_vmvalue
    .text:00401253 000 retn ; Return Near from Procedure
    .text:00401253

    .text:00401254 ; ---------------------------------------------------------------------------
    .text:00401254
    loc_401254: ; CODE XREF: Write_VMMemory_From+20#j
    .text:00401254 004 mov eax, [esp+4+VMValue_OperandSize]
    .text:00401258 004 dec eax ; Decrement by 1
    .text:00401259 004 jz short op_byte_no_be_swap ; Jump if Zero (ZF=1)

    .text:00401259
    .text:0040125B 004 dec eax ; Decrement by 1
    .text:0040125C 004 jz short op_word_do_le2be_swap ; Jump if Zero (ZF=1)

    .text:0040125C
    .text:0040125E 004 sub eax, 2 ; Integer Subtraction
    .text:00401261 004 jnz short loc_4012AD ; here below, operandsize is 4
    ... (the code here was already shown: it is the prior 'swap endian' code)

    .text:004012AD 004 mov eax, 1
    .text:004012B2 004 pop ebx_param_vmvalue
    .text:004012B3 000 retn ; Return Near from Procedure
    .text:004012B3
    Write_VMMemory_From endp

int Write_VMMemory_From(int VMAddress, int *LEValueSource, // Ptr_ValueToWriteAndSwap
                        int OperandSize, TVMContext* VMContext) // .text:00401230
{
    int *DestAddr;
    res = VMAddress2Real(VMContext, VMAddress, &DestAddr);
    switch (OperandSize) {
        case 1: *(byte*)DestAddr = SWAP((byte)LEValueSource); break;
        case 2: *(word*)DestAddr = SWAP((word)LEValueSource); break;
        case 4: *(dword*)DestAddr = SWAP((dword)LEValueSource); break;
    }
    return 1;
}
```

> 正如你所看到的，这个过程的代码简单的翻译一个内存地址到它的 x86 地址，然后写一个值在上面，交换它的最大的端。很明显它使用于处理诸如虚拟堆栈之类的过程。读过程非常的相似，但是它们的区别在于写的是 x86 内存，而不是虚拟的。

## VM 指令核心

用于解释 VM 程序操作码和传递它们到 VM 指令调度器的缓冲区代表这个 VM 指令的核心。从这点来看，你可以通过交换 VM 解码器和调整一些字段写出自己的 VM 语言。这可能是由于事实上 VM 解释这一层是在缓冲区里的，缓冲区扮演一个间接的层。但是让我们来仔细检查一下这个缓冲区：

> 代码:

```
struct TparamDecoding {
    // used by the decoding array to retrieve i.e. the parameters usage of instructions
    int ID;
    int Params[3];
};

struct TSubInstr {
    // represents a parameter's field of the VM Instruction
    int AddressType;
    int RegisterIdx;
    int Decoder_ParamsValue;
    int VMValue;
};

struct TInstructionBuffer {
    int Length;
    int InstructionData;
    char InstrType;
    // char Fillers1[3], // if structure alignment is 1
    int Operand_Size;
    char InstructionParamsCount;
    // char Fillers2[3], // if structure alignment is 1
    SubInstr ParamDest;
    SubInstr paramSrc;
    SubInstr ParamThird;
    SubInstr *WorkSubField;
};

// 这些结构正是 VM 使用的。现在让我们检查读写参数的函数：

enum TVMAddressType {
    vmaRegister = 0,
    vmaRegisterAddress = 1,
    vmaDirectAddress = 2,
    vmaVMValue_orC4__or_displacement = 3
};

int Retrieve_Param_Value(TInstructionBuffer *InstrBuff, SubInstr *Param, // 14h/24h/34h
                         int *WriteValueTo, TVMContext *VMContext) // .text:00401340
{
    int myLEvalue;

    switch (Param->AddressType) {
        case vmaRegister:
            switch (InstrBuff->OperandSize) {
                case 4: myLEvalue = (dword)VMContext->Registers[Param->RegisterIdx]; break;
                case 2: myLEvalue = (word)VMContext->Registers[Param->RegisterIdx]; break;
                case 1: myLEvalue = (char)VMContext->Registers[Param->RegisterIdx]; break;
            }
            break;
        case vmaDirectAddress:
            vmaddr = vmAddress;
        case vmaRegisterAddress:
            if (Param->AddressType != vmaDirectAddress) {
                vmaddr = VMContext->Registers[Param->RegisterIdx];
            }
            res = Read_VMMemory_To(vmaddr, myLEvalue, InstrBuff->OperandSize);
            if (!res) return 0;
            break;
        case vmaVMValue:
            myLEvalue = ParamField->VMAddress;
            break;
        default:
            break;
    }
    *WriteValueTo = myLEvalue;
}
```



正如我们注意到的，这个读取参数的过程辨别请求的地址类型，从 `VM` 寄存器集合中取回的值，从虚拟地址内存中的地址，从直接的值。任何时候它需要访问 `VM` 内存，它使用适当的函数读内存然后返回一个交换(小端)值。用来写参数的函数也是非常相似的。

在这点，只有两个主要的函数需要检查，一个关乎于虚拟标志，另一个是 `VM` 解码器本身。

`Evaluate_Flags()` 函数是在改变标志状态的函数里被调用。例如 `VM_CMP` 和 `VM_TEST` 指令调用这个过程来设置适当的 `VM` 标志。这个函数在数学操作之后也被调用为了保持一致的内部状态。这个程序让人感兴趣的部分在于它接收一个特殊的参数表明标志测试应该怎样执行：它用于改变 `OF`/`CF` 标志的数学操作。

代码:

```
int __cdecl Evaluate_Flags(int ParamAdditional, int ParamEvaluate,   int TestType, TInstructionBuffer* Instruction_ptr, TVMContext* VMContext) // .text:00401340
{
    TVMFLAGS *Flags = &VMContext->VMFlags;
    int OpSize = Instruction_ptr->OperandSize;
    int NegMark;
    switch(OpSize)
    {
        case 4:
            NegMark = 0x80000000;
            Flags->ZF = ParamEvaluate == 0;
            break;
        case 2:
            NegMark = 0x8000;
            Flags->ZF = (word)ParamEvaluate == 0;
            break;
        case 1:
            NegMark = 0x80;
            Flags->ZF = (char)ParamEvaluate == 0;
            break;
        default:
            NegMark = ParamAdditional; // room for BTx instructions expansion.
                                        // custom evaluation of flags based on bit-testing.
    }

    Flags->SF = (NegMark & ParamEvaluate) != 0;

    switch (TestType)
    {
        case 2:
            Flags->OF = ((NegMark & ParamEvaluate) == 0) && ((NegMark & ParamAdditional) != 0);
            Flags->CF = ((NegMark & ParamEvaluate) != 0) && ((NegMark & ParamAdditional) == 0);
            break;
        case 1:
            Flags->OF = ! ((NegMark & ParamEvaluate) == 0 && (NegMark & ParamAdditional) != 0);
            Flags->CF = ! ((NegMark & ParamEvaluate) != 0 && (NegMark & ParamAdditional) == 0);
            break;
        case 0:
        default:
            break;
    }

    return;
}
```

旁注：你能实现自己的作用于位(`BTS`, `BTC` 等)字段 的 `VM` 指令，然后用一个操作数大小不同于正常值(1-2-4)的参数调用这个函数。这会造成函数接受额外的参数作为位掩码来执行 `BTX` 测试在这个位上。

这是解码器函数：

代码:

```
bool VMInstructionDecoder(TInstructionBuffer* InstructionPtr, byte *VMEIP_RealAddr)
{
    // .text:00401000
    TInstrTag InstrType;
    byte LowNib, HiNib;
    AddrSize;
    JccIndex;
    dword *ExaminedDwords;
    TempInstrSize;
    dd* TempPtr;
    ParamsCount;
    Temp;
    wTemp;
    bTemp;

    memset(InstrBuf, 0, 0x13 * 4);
    InstructionPtr->WorkSubField = &InstructionPtr->ParamDest; // set which is the first decoded param

    /* 1. set types */
    InstrType = VMEIP_RealAddr[0]; // *(byte *)VMEIP_RealAddr
    InstructionPtr->InstrType = InstrType.InstrType; // b & 0x3F; // == 00111111b

    swith(InstrType.AddSize) // swith(InstrType >> 6) { // the sub is needed for setting flags!
    {
        case 0:
            AddrSize = 1;
            break;
        case 1:
            AddrSize = 2;
            break;
        case 2:
            AddrSize = 4;
            break;
        default:
            return 0;
    }

    InstructionPtr->OperandSize = AddrSize;
    ParamIdx = InstructionPtr->InstrType; // InstructionPtr->InstrType << 4; // *structure size

    if (ParamTable[ParamIdx].ID == 0x33)
        return 0; // 0x33 entry has no associated instruction

    if ((char)ParamTable[ParamIdx].ParamDest == 4 && AddrSize != 4)
        return 0;

    InstructionPtr->InstrID = ParamTable[ParamIdx].ID; // Jump Address!

    /* 2. cycle thru instruction parameters as from Instruction Decoder's Table, and fill buffer */
    ExaminedParams = 0;
    TempInstrSize = 1; // 0 was already used for getting here!!
    ParamsCount = 0; // decode the first param, so!

    while (ParamTable[ParamIdx].Params[ParamsCount] != 0)
    {
        // .text:004010B1 param decoding loop
        // ParamsValue = ParamTable[ParamIdx].Params[ParamsCount].ID;
        LowNib_RegIdx = VMEIP_RealAddr[TempInstrSize] & 0x0F;
        HiNib_AddrMode = VMEIP_RealAddr[TempInstrSize] >> 4;
        InstructionPtr->WorkSubField[ParamsCount].AddressType = HiNib;

        /* 3. set up instruction sub-type (Jcc Type in this VM) */
        InstructionPtr->WorkSubField[ParamsCount].Decoder_ParamsValue = ParamTable[ParamIdx].Params[ParamsCount];

        switch (HiNib_AddrMode)
        {
            // NOTE: switch on decoded address type!!
            case vmaRegister: // 0
            case vmaRegisterAddress: // 1
                InstructionPtr->WorkSubField[ParaCount].RegisterIdx = LowNib_RegIdx;
                TempInstrSize++;
                break;
            case vmaVMValue_orC4__or_displacement: // 3 .text:00401134
                if ((char)ParamTable[ParamIdx].Params[ParamsCount] == 2)
                    return 0;
                TempInstrSize++;

                switch (InstructionPtr->OperandSize)
                {
                    case 1:
                        bTemp = ((byte *)VMEIP_RealAddr)[TempInstrSize];
                        InstructionPtr->WorkSubField[ParamsCount].VMValue = (dword)bTemp;
                        TempInstrSize++;
                        break;
                    case 2:
                        // this might be an instrinsic inline function, due to code shape (compiler didnt recon param 2 was 0)
                        wTemp = ((word *)VMEIP_RealAddr)[TempInstrSize];
                        wTemp = SWAP(wTemp);
                        InstructionPtr->WorkSubField[ParamsCount].VMValue = (dword)wTemp;
                        TempInstrSize += 2;
                        break;
                    case 4:
                        break;
                    default:
                        return 0;
                }
                break;
            case vmaDirectAddress: // 2
                if (HiNib_AddrMode == vmaDirectAddress)
                {
                    // added by me to keep flow
                    TempInstrSize++;
                    if (InstructionPtr->OperandSize != 4)
                        return 0;
                }

                // .text:00401101 common code to case 2 and 3 here...
                Temp = ((dword *)VMEIP_RealAddr)[TempInstrSize];
                Temp = SWAP(Temp);
                InstructionPtr->WorkSubField[ParamsCount].VMValue = (dword)Temp;
                TempInstrSize += 4;
                break;
            default:
                return 0;
        }

        ParamsCount++; // next param data!
        ExaminedParams++;
        if (ParaCount >= 3)
            break; // max 32 bytes fetched this way
    }

    InstructionPtr->InstructionParamsCount = ExaminedParams;
    InstructionPtr->InstrSize = TempInstrSize;
    return TempInstrSize;
}
```

它简单的填充常见参数到指令的初始化部分，然后通过使用参数的解码器结构开始循环。这个结构持有指令使用的参数，以及某个指令子类型，看上去和 `Jcc` 中使用的标志有关。最初我把这个字段叫做 `JccType`，然后我改成了另一个通用的 `InstrType`，因为理论上它有一个更通用的用法正如代码所揭露的。

对参数的主循环是执行于 `TParamDecoding.Params[]` 数组之上。零值就意味着指令没有使用参数。依赖于参数的地址类型，它取到正确的数据并且填充 `TInstructionBuffer` 结构。作为备注，`TParamDecoding` 数组开始于 `.data:00407030 ParamTable` 正好结束于 `VM` 指令表之前。

### VM 指令

指令集包含最常见的 `x86` 指令，和一些消息。这是 这个机器实现的 `VM` 操作码 列表:

代码:

```
.data:00407430 VM_Opcode_Table dd offset VM_MOV ; DATA XREF: _main+162#r
.data:00407434 dd offset VM_Multiple_op2
.data:00407438 dd offset VM_Multiple_op2
.data:0040743C dd offset VM_Multiple_op2
.data:00407440 dd offset VM_Multiple_op2
.data:00407444 dd offset VM_Multiple_op2
.data:00407448 dd offset VM_PUSH
.data:0040744C dd offset VM_POP
.data:00407450 dd offset VM_JMP
.data:00407454 dd offset VM_CALL
.data:00407458 dd offset VM_LOOP
.data:0040745C dd offset VM_RET
.data:00407460 dd offset VM_Multiple_op2
.data:00407464 dd offset VM_INC
.data:00407468 dd offset VM_DEC
.data:0040746C dd offset VM_NOP
.data:00407470 dd offset VM_Jcc
.data:00407474 dd offset VM_Jcc
.data:00407478 dd offset VM_Jcc
.data:0040747C dd offset VM_Jcc
.data:00407480 dd offset VM_Jcc
.data:00407484 dd offset VM_Jcc
.data:00407488 dd offset VM_Jcc
.data:0040748C dd offset VM_Jcc
.data:00407490 dd offset VM_NOP
.data:00407494 dd offset VM_NOP
.data:00407498 dd offset VM_NOP
.data:0040749C dd offset VM_NOP
.data:004074A0 dd offset VM_NOP
.data:004074A4 dd offset VM_NOP
.data:004074A8 dd offset VM_Multiple_op2
.data:004074AC dd offset VM_Multiple_op2
.data:004074B0 dd offset VM_Multiple_op2
.data:004074B4 dd offset VM_Multiple_op2
.data:004074B8 dd offset VM_CMP
.data:004074BC dd offset VM_TEST
.data:004074C0 dd offset VM_NOT
.data:004074C4 dd offset VM_Multiple_op2
.data:004074C8 dd offset VM_Multiple_op2
.data:004074CC dd offset VM_MOV_MEMADDR_TO
.data:004074D0 dd offset VM_MOV_EIP_TO
.data:004074D4 dd offset VM_SWAP
.data:004074D8 dd offset VM_ADD_TO_ESP
.data:004074DC dd offset VM_SUB_FROM_ESP
.data:004074E0 dd offset VM_MOV_FROM_ESP
.data:004074E4 dd offset VM_MOV_TO_ESP
.data:004074E8 dd offset VM_NOP
.data:004074EC dd offset VM_NOP
.data:004074F0 dd offset VM_NOP
.data:004074F4 dd offset VM_NOP
.data:004074F8 dd offset VM_NOP
.data:004074FC dd offset VM_NOP
.data:00407500 dd offset VM_NOP
.data:00407504 dd offset VM_NOP
.data:00407508 dd offset VM_NOP
.data:0040750C dd offset VM_NOP
.data:00407510 dd offset VM_NOP
.data:00407514 dd offset VM_NOP
.data:00407518 dd offset VM_NOP
.data:0040751C dd offset VM_NOP
.data:00407520 dd offset VM_Status_8
.data:00407524 dd offset VM_SET_RESUME_EIP
.data:00407528 dd offset VM_NOP
.data:0040752C dd offset VM_ALLOW_IO
```

没有什么好说的了，除了可能检查其它的指令实现。

下面的可能很有趣：

代码:



```c
int __cdecl VM_LOOP(TVMContext* VMContext, TInstructionBuffer* DecodedInstr) {
   int VMCycleValue, VMValue;
   Retrieve_Param_Value(DecodedInstr->ParamSrc, &VMCycleValue);
   VMCycleValue--;
   if (VMCycleValue==0) {
   NextInstr(VMContext,DecodedInstr);
   return 0;
   }
   Write_VMValue_To_Param(&DecodedInstr->ParamSrc, &VMCycleValue,4,VMContext);
   Retrieve_Param_Value(DecodedInstr->ParamDest, &VMValue);
   if (DecodedInstr->AddressType==vmaVMValueOrDisplacement)
   VMValue+=VMContext->VM_EIP;
   VMContext->VM_EIP = VMValue;
   return 0;
   }
```

你可以注意到，这个指令取回一个参数值-`ECX` 是理想的等价物 -并递减它. 当它到达零值时结束并跳到下一条指令，就像 `LOOPECXZ`，另外它写递减后的值到源操作数/寄存器执行一个跳转到请求的地址，是相对还是绝对依赖于地址类型：

下面是另一个很有趣的指令：

```c
int __cdecl VM_CALL(TVMContext* VMContext, TInstructionBuffer* DecodedInstr) {
   int VMValue, VMRetValue;int res;
   VMRetValue = VMContext->VM_EIP+DecodedInstr->Length;
   res = Retrieve_Param_Value(DecodedInstr->ParamDest, &VMValue);
   if (!res) return 1;
   if (DecodedInstr->AddressType==vmaVMValueOrDisplacement)
   VMValue+=VMContext->VM_EIP;
   VMContext->VM_EIP = VMValue; 
   VMContext->VM_ESP+=4;
   Write_VMMemory_From(&VMContext->VM_ESP, &VMRetValue,4,VMContext);
   return 0;
   }
```

正如你所看到的，这个指令读子程序地址，设置 `VM_EIP` 准备开始下一个执行 ，然后写 `VM_ESP` 保存下一条指令的 `VM_EIP` 值  ( 在 `VM_CALL` 后的哪一个) 到堆栈中。如前面所记录的，堆栈地址模式被交换 。

还有很多的指令可以检查，但是你可以看附录里的源码如果你正寻找一些特别的东西。

### 总结

这个 `VM` 有一些东西丢失了，最明显就是操作单个位，以及调试层支持的指令 。二者都只是简单的出现(原文 in nuce, 网上找到一个定义 in a nutshell)中，由于我们排除了在 `Evaluate_Flags()`中进行特殊位测试的可能性，除了最明显的一个在机器主循环中的陷阱测试。

好了，我让你烦够了， 写这些也烦够我了。希望你能学到些关于虚拟机的东西，并且你会发现这篇短文对于你所需要的无论什么都很有用。 

感谢整个 `RCE` 社区无偿分享他们所知道的秘诀。我只是希望 把从你们中所学的归还出来。

### Appendix

这里你能找到几乎整个 `T2'06` 逆向源代码。一些部分丢失了 (所有的完整性检查可以肯定)，因而它不可能编译得和原来一模一样。然而用它来编码你自己的虚拟机只需要付出很少的努力， 也可以学习到 `T2'06` 更多的细节。

希望你会欣赏它。

```



```c
/* #define SWITCHSIZE(OpSize, Code4, Code2, Code1) switch ( (OpSize) ) \
case 4: {Code4;} break; \
case 2: {Code2;} break; \
case 1: {Code1;} break; \
}; */

enum TVMAddressType {
    vmaRegister = 0,
    vmaRegisterAddress = 1,
    vmaDirectAddress = 2,
    vmaVMValue_orC4__or_displacement = 3
};

enum TMachineControlStatus {
    mcStepBreakPoint = 2,
    mcWrongAddress = 3,
    mcGenericError_or_CannotWriteTo = 4,
    mcDivideByZero = 5,
    mcInputOutput = 9
};

enum TFLAGS {
    ZF = 1,
    CF = 2,
    OF = 4,
    SF = 8,
    TF = 0x80
};

enum TIOFlags {
    DoOutput = 1,
    DoInput = 2
};

enum TJccType {
    jccJZ = 0x10,
    jccJNZ = 0x11,
    jccJS = 0x12,
    jccJNS = 0x13,
    jccJO = 0x14,
    jccJNO = 0x15,
    jccJB = 0x16,
    jccJNB = 0x17
};

struct TVMFLAGS {
    // ~Compiler Dependent~ -please check the order!!
    ZF:1, // supposed Bit 0
    CF:1,// supposed Bit 1
    OF:1,// supposed Bit 2
    SF:1,// supposed Bit 3
    Unused:3,// supposed Bit 4-6
    TF:1,// supposed Bit 7
};

struct TInstrTag {
    AddrSize:2,
    InstrType:6
};

struct TVMContext {
    int Register_IOType,
    int Register_IO,
    int Register_IOAddress,
    int Register_IOCount,
    int GenericRegisters[12],
    int *Registers,
    int VM_ESP ,
    int VM_EIP ,
    int VMEIP_Saved_Prior_InstrExec,
    TVMFLAGS VM_EFLAGS,
    int InstructionCounter,
    int InitCode,
    int MemorySize,
    void* ProgramMemoryAddr,
    int Original_ESP,
    int StackMemorySize,
    void* StackMemoryAddr,
    int MachineControl,
    int VM_ResumeExec
};

struct TSubInstr {
    int AddressType,
    int RegisterIdx,
    int Decoder_ParamsValue,
    int VMValue
};

struct TParamDecoding{
    int ID,
    int Params[3];
};

struct TInstructionBuffer{
    int Length,
    int InstructionData,
    char InstrType,
    char Fillers1[3], // if structure alignment is 1, nothing otherwise
    int Operand_Size,
    char InstructionParamsCount,
    char Fillers1[3], // if structure alignment is 1, nothing otherwise
    SubInstr ParamDest,
    SubInstr paramSrc,
    SubInstr ParamThird,
    SubInstr *WorkSubField
};

/* this is the original Opcode Table Array, from IDA
.data:00407430 VM_Opcode_Table dd offset VM_MOV ; DATA XREF: _main+162#r
.data:00407434 dd offset VM_Multiple_op2
.data:00407438 dd offset VM_Multiple_op2
.data:0040743C dd offset VM_Multiple_op2
.data:00407440 dd offset VM_Multiple_op2
.data:00407444 dd offset VM_Multiple_op2
.data:00407448 dd offset VM_PUSH
.data:0040744C dd offset VM_POP
.data:00407450 dd offset VM_JMP
.data:00407454 dd offset VM_CALL
.data:00407458 dd offset VM_LOOP
.data:0040745C dd offset VM_RET
.data:00407460 dd offset VM_Multiple_op2
.data:00407464 dd offset VM_INC
.data:00407468 dd offset VM_DEC
.data:0040746C dd offset VM_NOP
.data:00407470 dd offset VM_Jcc
.data:00407474 dd offset VM_Jcc
.data:00407478 dd offset VM_Jcc
.data:0040747C dd offset VM_Jcc
.data:00407480 dd offset VM_Jcc
.data:00407484 dd offset VM_Jcc
.data:00407488 dd offset VM_Jcc
.data:0040748C dd offset VM_Jcc
.data:00407490 dd offset VM_NOP
.data:00407494 dd offset VM_NOP
.data:00407498 dd offset VM_NOP
.data:0040749C dd offset VM_NOP
.data:004074A0 dd offset VM_NOP
.data:004074A4 dd offset VM_NOP
.data:004074A8 dd offset VM_Multiple_op2
.data:004074AC dd offset VM_Multiple_op2
.data:004074B0 dd offset VM_Multiple_op2
.data:004074B4 dd offset VM_Multiple_op2
.data:004074B8 dd offset VM_CMP
.data:004074BC dd offset VM_TEST
.data:004074C0 dd offset VM_NOT
.data:004074C4 dd offset VM_Multiple_op2
.data:004074C8 dd offset VM_Multiple_op2
.data:004074CC dd offset VM_MOV_MEMADDR_TO
.data:004074D0 dd offset VM_MOV_EIP_TO
.data:004074D4 dd offset VM_SWAP
.data:004074D8 dd offset VM_ADD_TO_ESP
.data:004074DC dd offset VM_SUB_FROM_ESP
.data:004074E0 dd offset VM_MOV_FROM_ESP
.data:004074E4 dd offset VM_MOV_TO_ESP
.data:004074E8 dd offset VM_NOP
.data:004074EC dd offset VM_NOP
.data:004074F0 dd offset VM_NOP
.data:004074F4 dd offset VM_NOP
.data:004074F8 dd offset VM_NOP
.data:004074FC dd offset VM_NOP
.data:00407500 dd offset VM_NOP
.data:00407504 dd offset VM_NOP
.data:00407508 dd offset VM_NOP
.data:0040750C dd offset VM_NOP
.data:00407510 dd offset VM_NOP
.data:00407514 dd offset VM_NOP
.data:00407518 dd offset VM_NOP
.data:0040751C dd offset VM_NOP
.data:00407520 dd offset VM_Status_8
.data:00407524 dd offset VM_SET_RESUME_EIP
.data:00407528 dd offset VM_NOP
.data:0040752C dd offset VM_ALLOW_IO
*/

void NextInstr(TVMContext &VMContext, TInstructionBuffer &DecodedInstr) {
    VMContext.VM_EIP += DecodedInstr.InstructionLength;
}

short int SWAP(short int x); // swap endian like above, x = SWAP(x1)
Smallint SWAP(Smallint x);
int SWAP(int x);

#define BETWEEN(x, loBound, hiBound) ((x>loBound)&&(x<hiBound)?true:false)
#define RANGE(x, lowLimit, RangeFromLimit) ((x>lowLimit)&&(x<(lowLimit+RangeFromLimit))?true:false)

bool VMAddress2Real(TVMContext *VMContext, int VMAddress, int *RealAddr) {
    // .text:004011D0
    if (RANGE(VMAddress, VMContext->InitCode, VMContext->MemorySize)) {
        *RealAddr = (VM_Address - VMContext->InitCode) + VMContext->ProgramMemoryAddr;
        return 1;
    }
    if (RANGE(VMAddress, VMContext->Original_ESP, VMContext->StackMemorySize)) {
        *RealAddr = (VM_Address - VMContext->Original_ESP) + VMContext->StackMemoryAddr;
        return 1;
    }
    VMContext->MachineControl = mcWrongAddress;
    return 0;
}

int Write_VMMemory_From(
    int VMAddress,
    int *LEValueSource, // Ptr_ValueToWriteAndSwap
    int OperandSize,
    TVMContext* VMContext)
{
    int *DestAddr;
    res = VMAddress2Real(VMContext, VMAddress, &DestAddr);
    switch(OperandSize) {
        case 1: *(byte*)DestAddr = SWAP((byte)LEValueSource); break;
        case 2: *(word*)DestAddr = SWAP((word)LEValueSource); break;
        case 4: *(dword*)DestAddr = SWAP((dword)LEValueSource); break;
    }
    return 1;
}

// used for memory values
int Read_VMMemory_To(
    int VMAddress,
    int *Write_Address_To,
    int OperandSize,
    TVMContext* VMContext)
{
    // VMAddress will contain the real addr of memory location
    res = VMAddress2Real(VMContext, VMAddress, &VMAddress);
    switch(OperandSize) {
        case 1: *(byte*)Write_Address_To = SWAP((byte)VMAddress); break;
        case 2: *(word*)Write_Address_To = SWAP((word)VMAddress); break;
        case 4: *(dword*)Write_Address_To = SWAP((dword)VMAddress); break;
    }
    //SwapEndianMem(AddressDataSize, VMAddress, Write_Address_To);
    return 1;
}

int Retrieve_Param_Value(
    TInstructionBuffer *InstrBuff,
    SubInstr *Param, // 14h/24h/34h
    int *WriteValueTo,
    TVMContext *VMContext)
{
    int myLEvalue;
    switch(Param->AddressType) {
        case vmaRegister:
            switch(InstrBuff->OperandSize){
                case 4: myLEvalue = (dword)VMContext->Registers[Param->RegisterIdx]; break;
                case 2: myLEvalue = (word)VMContext->Registers[Param->RegisterIdx]; break;
                case 1: myLEvalue = (char)VMContext->Registers[Param->RegisterIdx]; break;
            }
            break;
        case vmaDirectAddress:
            vmaddr = vmAddress;
        case vmaRegisterAddress:
            if (Param->AddressType != vmaDirectAddress) {
                vmaddr = VMContext->Registers[Param->RegisterIdx];
            }
            res = Read_VMMemory_To(vmaddr, myLEvalue, InstrBuff->OperandSize);
            if (!res) return 0;
            break;
        case vmaVMValue:
            myLEvalue = ParamField->VMAddress;
            break;
        default:
    }
    *WriteValueTo = myLEvalue;
}

int __cdecl Write_VMValue_To_Param(
    TInstructionBuffer *InstrBuff,
    TSubInstr *Param, // 14h/24h/34h
    int *WriteValueTo,
    TVMContext *VMContext)
{
    switch(Param->AddressType) {
        case vmaRegister:
            switch(InstrBuff->OperandSize){
                case 4: VMContext->Registers[Param->RegisterIdx] = ValueToWriteTo; break;
                case 2: VMContext->Registers[Param->RegisterIdx] = (word)ValueToWriteTo | (value & !0xFFFF); break;
                case 1: VMContext->Registers[Param->RegisterIdx] = (char)ValueToWriteTo | (value & !0xFF); break;
                default: return 1;
            }
            break;
        case vmaDirectAddress:
            vmaddr = vmAddress;
        case vmaRegisterAddress:
            if (Param->AddressType != vmaDirectAddress) {
                vmaddr = VMContext->Registers[Param->RegisterIdx];
            }
            res = Write_VMMemory_From(vmaddr, &ValueToWriteTo, InstrBuff->OperandSize, VMContext);
            if (!res) return 0;
            break;
        case vmaVMValue:
            VMContext->MachineControl = mcGenericError_or_CannotWriteTo;
            break;
        default:
    }
    return 1;
}

int __cdecl Evaluate_Flags(
    int ParamAdditional,
    int ParamEvaluate,
    int TestType,
    TInstructionBuffer* Instruction_ptr,
    TVMContext* VMContext)
{
    TVMFLAGS *Flags = &VMContext->VMFlags;
    int OpSize = Instruction_ptr->OperandSize;
    int NegMark;
    switch(OpSize) {
        case 4: NegMark = 0x80000000; Flags->ZF = ParamEvaluate == 0; break;
        case 2: NegMark = 0x8000; Flags->ZF = (word)ParamEvaluate == 0; break;
        case 1: NegMark = 0x80; Flags->ZF = (char)ParamEvaluate == 0; break;
        default: NegMark = ParamAdditional; // room for BTx instructions expansion.
        // custom evaluation of flags based on bit-testing.
    }
    Flags->SF = (NegMark & ParamEvaluate) != 0;
    switch (TestType) {
        case 2:
            Flags->OF = (NegMark & ParamEvaluate) == 0 && (NegMark & ParamAdditional) != 0;
            Flags->CF = (NegMark & ParamEvaluate) != 0 && (NegMark & ParamAdditional) == 0;
            break;
        case 1:
            Flags->OF = ! ((NegMark & ParamEvaluate) == 0 && (NegMark & ParamAdditional) != 0);
            Flags->CF = ! ((NegMark & ParamEvaluate) != 0 && (NegMark & ParamAdditional) == 0);
            break;
        case 0:
        default:
    }
    return;
}

int __cdecl VM_MOV_EIP_TO(TVMContext* VMContext, TInstructionBuffer* DecodedInstr)
{
    int res = Write_VMValue_To_Param(&DecodedInstr->ParamDest, VMContext->VMEIP, VMContext);
    if (res) return 1;
    NextInstr(VMContext, DecodedInstr);
    return 0;
}

int __cdecl VM_MOV_MEMADDR_TO(TVMContext* VMContext, TInstructionBuffer* DecodedInstr)
{
    int VMAddress;
    switch(DecodedInstr->ParamSrc) {
        case vmaRegisterAddress:
            VMAddress = VMContext->Registers[DecodedInstr->ParamSrc.RegisterIdx];
            break;
        case vmaDirectAddress:
            VMAddress = DecodedInstr->ParamSrc.VMAddress;
            break;
        default:
            VMContext->MachineControl = mcCannotWriteTo;
            return 0;
    }
    Write_VMValue_To_Param(&DecodedInstr.ParamDest, VMAddress, VMContext);
    NextInstr(VMContext, DecodedInstr);
}

int __cdecl VM_ADD_TO_ESP(TVMContext* VMContext, TInstructionBuffer* DecodedInstr)
{
    int VMValue;
    Retrieve_Param_Value(DecodedInstr->ParamDest, &VMValue);
    VMContext->VM_ESP += VMValue;
    NextInstr(VMContext, DecodedInstr);
}

int __cdecl VM_SUB_FROM_ESP(TVMContext* VMContext, TInstructionBuffer* DecodedInstr)
{
    int VMValue;
    Retrieve_Param_Value(DecodedInstr->ParamDest, &VMValue);
    VMContext->VM_ESP -= VMValue;
    NextInstr(VMContext, DecodedInstr);
}

int __cdecl VM_MOV_FROM_ESP(TVMContext* VMContext, TInstructionBuffer* DecodedInstr)
{
    int VMValue;
    Write_VMValue_To_Param(DecodedInstr->ParamDest, &VMContext.VM_ESP);
    NextInstr(VMContext, DecodedInstr);
}

int __cdecl VM_MOV_TO_ESP(TVMContext* VMContext, TInstructionBuffer* DecodedInstr)
{
    int VMValue;
    Retrieve_Param_Value(DecodedInstr->ParamDest, &VMContext.VM_ESP);
    NextInstr(VMContext, DecodedInstr);
}

int __cdecl VM_MOV(TVMContext* VMContext, TInstructionBuffer* DecodedInstr)
{
    int VMValue;
    Retrieve_Param_Value(DecodedInstr->ParamSrc, &VMValue);
    Write_VMValue_To_Param(&DecodedInstr->ParamDest, &VMValue, VMContext);
    NextInstr(VMContext, DecodedInstr);
}

int __cdecl VM_NOT(TVMContext* VMContext, TInstructionBuffer* DecodedInstr)
{
    int VMValue;
    Retrieve_Param_Value(DecodedInstr->ParamSrc, &VMValue);
    VMValue = !VMValue;
    Evaluate_Flags(VMValue, 0, DecodedInstr, VMContext);
    Write_VMValue_To_Param(&DecodedInstr->ParamSrc, &VMValue, VMContext);
    NextInstr(VMContext, DecodedInstr);
}

int __cdecl VM_CMP(TVMContext* VMContext, TInstructionBuffer* DecodedInstr)
{
    int VMValue, VMValueSrc, VMValueDst;
    Retrieve_Param_Value(DecodedInstr->ParamSrc, &VMValueSrc);
    Retrieve_Param_Value(DecodedInstr->ParamDest, &VMValueDst);
    switch(DecodedInstr->OperandSize){
        case 4: VMValue = (dword)VMValueDst - (dword)VMValueSrc; break;
        case 2: VMValue = (word)VMValueDst - (word)VMValueSrc; break;
        case 1: VMValue = (char)VMValueDst - (char)VMValueSrc; break;
    }
    Evaluate_Flags(VMContext, VMValue, VMValueSrc, 2, DecodedInstr, VMContext);
    NextInstr(VMContext, DecodedInstr);
}

int __cdecl VM_TEST(TVMContext* VMContext, TInstructionBuffer* DecodedInstr) { /* as above, make and */ };

int __cdecl VM_XCHG(VMContext, DecodedInstr) {
    int VMValueSrc, VMValueDst;
    Retrieve_Param_Value(DecodedInstr->ParamSrc, &VMValueSrc);
    Retrieve_Param_Value(DecodedInstr->ParamDest, &VMValueDst);
    Write_VMValue_To_Param(&DecodedInstr->ParamSrc, &VMValueDst, VMContext);
    Write_VMValue_To_Param(&DecodedInstr->ParamDest, &VMValueSrc, VMContext);
    NextInstr(VMContext, DecodedInstr);
}

int __cdecl VM_INC(TVMContext* VMContext, TInstructionBuffer* DecodedInstr) {...};
int __cdecl VM_DEC(TVMContext* VMContext, TInstructionBuffer* DecodedInstr) {...};

int __cdecl VM_PUSH(TVMContext* VMContext, TInstructionBuffer* DecodedInstr) {
    int VMValue; int res;
    Retrieve_Param_Value(DecodedInstr->ParamSrc, &VMValue);
    VMContext.VM_ESP += 4;
    Write_VMValue_To_Param(&VMContext->VM_ESP, &VMValue, 4, VMContext);
    NextInstr(VMContext, DecodedInstr);
}

int __cdecl VM_POP(TVMContext* VMContext, TInstructionBuffer* DecodedInstr) {
    int VMValue;
    Read_VMMemory_To(VMContext->VM_ESP, &VMValue);
    Write_VMValue_To_Param(&DecodedInstr->ParamDest, &VMValue, 4, VMContext);
    VMContext.VM_ESP -= 4;
    NextInstr(VMContext, DecodedInstr);
}

int __cdecl VM_Jcc(TVMContext* VMContext, TInstructionBuffer* DecodedInstr) {
    int VMValue;
    bool DoJump = false;
    // Original code is a bit different here:
    // BUT it is more professional this way.
    switch(DecodedInstr->InstrType) {
        case jccJZ: DoJump = VMContext->VM_EFlags & ZF; break;
        case jccJNZ: DoJump = !VMContext->VM_EFlags & ZF; break;
        case jccJS: DoJump = VMContext->VM_EFlags & SF; break;
        case jccJNS: DoJump = !VMContext->VM_EFlags & SF; break;
        case jccJO: DoJump = VMContext->VM_EFlags & OF; break;
        case jccJNO: DoJump = !VMContext->VM_EFlags & OF; break;
        case jccJB: DoJump = VMContext->VM_EFlags & CF; break;
        case jccJNB: DoJump = !VMContext->VM_EFlags & CF; break;
        default: break;
    }
    if(!DoJump) {
        NextInstr(VMContext, DecodedInstr);
        return 0;
    }
    res = Retrieve_Param_Value(DecodedInstr->ParamDest, &VMValue);
    if (!res) return 1;
    if (DecodedInstr->AddressType == vmaVMValueOrDisplacement)
        VMValue += VMContext->VM_EIP;
    VMContext->VM_EIP = VMValue;
    return 0;
}

int __cdecl VM_JMP(TVMContext* VMContext, TInstructionBuffer* DecodedInstr) {...};

int __cdecl VM_CALL(TVMContext* VMContext, TInstructionBuffer* DecodedInstr) {
    int VMValue, VMRetValue; int res;
    VMRetValue = VMContext->VM_EIP + DecodedInstr->Length;
    res = Retrieve_Param_Value(DecodedInstr->ParamDest, &VMValue);
    if (!res) return 1;
    if (DecodedInstr->AddressType == vmaVMValueOrDisplacement)
        VMValue += VMContext->VM_EIP;
    VMContext->VM_EIP = VMValue;
    VMContext->VM_ESP += 4;
    Write_VMValue_To_Param(&VMContext->VM_ESP, &VMRetValue, 4, VMContext);
    return 0;
}

int __cdecl VM_LOOP(TVMContext* VMContext, TInstructionBuffer* DecodedInstr) {
    int VMCycleValue, VMValue;
    Retrieve_Param_Value(DecodedInstr->ParamSrc, &VMCycleValue);
    VMCycleValue--;
    if (VMCycleValue == 0) {
        NextInstr(VMContext, DecodedInstr);
        return 0;
    }
    Write_VMValue_To_Param(&DecodedInstr->ParamSrc, &VMCycleValue, 4, VMContext);
    Retrieve_Param_Value(DecodedInstr->ParamDest, &VMValue);
    if (DecodedInstr->AddressType == vmaVMValueOrDisplacement)
        VMValue += VMContext->VM_EIP;
    VMContext->VM_EIP = VMValue;
    return 0;
}

int __cdecl VM_RET(TVMContext* VMContext, TInstructionBuffer* DecodedInstr) {
    int VMValue;
    Read_VMMemory_To(VMContext->VM_ESP, &VMValue);
    VMContext->VM_ESP -= 4;
    VMContext->VM_EIP = VMValue;
}

int __cdecl VM_RESUME_EIP(TVMContext* VMContext, TInstructionBuffer* DecodedInstr) {
    // set the resume address on error/the end condition
    int VMValue;
    Retrieve_Param_Value(DecodedInstr->ParamDest, &VMValue);
    VMContext->VM_ResumeExec = VMValue;
    NextInstr(VMContext, DecodedInstr);
}

int __cdecl VM_ALLOW_IO(TVMContext* VMContext, TInstructionBuffer* DecodedInstr) {
    VMContext->MachineControl = mcInputOutput;
    NextInstr(VMContext, DecodedInstr);
    // very ugly: for having IO you must force a MachineControl check, as if error were in.
    return 1;
}

int __cdecl VM_NOP(TVMContext* VMContext, TInstructionBuffer* DecodedInstr) {
    NextInstr(VMContext, DecodedInstr);
}

// unsigned -> signed
int __cdecl VM_MULTIPLE_OP2(TVMContext* VMContext, TInstructionBuffer* DecodedInstr) {
    int VMValueDst, VMValueThird, VMValueSrc, VMValueEval;
    int ModeSwitch = 0;
    Retrieve_Param_Value(DecodedInstr->ParamThird, &VMValueThird);
    Retrieve_Param_Value(DecodedInstr->ParamSource, &VMValueSrc);
    switch(DecodedInstr->InstrID - 1) {
        case 0: // ADD
            ModeSwitch = 2;
            switch(DecodedInstr->OperandSize) {
                case 1: VMValueEval = (char)VMValueSrc + (char)VMValueThird; break;
                case 2: VMValueEval = (word)VMValueSrc + (word)VMValueThird; break;
                case 4: VMValueEval = VMValueSrc + VMValueThird;
            }
            break;
        case 1: // SUB
            ModeSwitch = 1;
            switch(DecodedInstr->OperandSize) {
                case 1: VMValueEval = (char)VMValueSrc - (char)VMValueThird; break;
                case 2: VMValueEval = (word)VMValueSrc - (word)VMValueThird; break;
                case 4: VMValueEval = VMValueSrc - VMValueThird;
            }
            break;
        case 2: // XOR
            switch(DecodedInstr->OperandSize) {
                case 1: VMValueEval = (char)VMValueSrc ^ (char)VMValueThird; break;
                case 2: VMValueEval = (word)VMValueSrc ^ (word)VMValueThird; break;
                case 4: VMValueEval = VMValueSrc ^ VMValueThird;
            }
            break;
        case 10: // AND - copy&paste above.
            break;
        case 11: // OR - copy&paste above.
            break;
        case 6: // SHL - copy&paste above.
            break;
        case 7: // SHR - copy&paste above.
            break;
        case 8: // ROL - copy&paste above.
            break;
        case 9: // ROR - copy&paste above.
            break;
        case 4: // IMUL - copy&paste above.
            break;
        case 5: // IDIV
            if (VMValueThird == 0) {
                VMContext->MachineControl = mcDivideByZero;
                return 0;
            }
            switch(DecodedInstr->OperandSize) {
                case 1: VMValueEval = (byte)VMValueSrc / (byte)VMValueThird; break;
                case 2: VMValueEval = (word)VMValueSrc / (word)VMValueThird; break;
                case 4: VMValueEval = VMValueSrc / VMValueThird;
            }
            break;
        case 5: // IDIVREST - copy&paste above.
            break;
        default: // opcode 12 and follows
    }
    Evaluate_Flags(VMValueSrc, VMValueEval, ModeSwitch, InstructionPtr, VmContext);
    Write_VMValue_To_Param(InstructionPtr, DecodedInstr->ParamDest, &VMValueEval, VMContext);
    NextInstr(VMContext, DecodedInstr);
}

int __cdecl Do_Write_Output(TVMContext* VMContext) {
    int NumberBytesToWriteOut;
    void *BufferToWrite;
    if (VMContext->Register_IO != 0) return 0;
    VMAddress2Real(VMContext, VMContext->Register_IOAddress, &BufferToWrite);
    NumberBytesToWriteOut = VM_Context->Register_IOCount; //
    VM_Context->Register_IOCount = write(stdout, BufferToWrite, NumberBytesToWriteOut);
    return 1;
}

int __cdecl Do_Read_Input(TVMContext* VMContext) {
    int NumberBytesToReadIn;
    void *BufferToRead;
    if (VMContext->Register_IO != 0) return 0;
    VMAddress2Real(VMContext, VMContext->Register_IOAddress, &BufferToRead);
    NumberBytesToReadIn = VM_Context->Register_IOCount;
    VM_Context->Register_IOCount = read(stdin, BufferToRead, NumberBytesToReadIn);
    return 1;
}

int __cdecl CheckForIO(VM_Context) {
    switch(VMContext.Register_IOType) {
        case 2: return Do_Write_Output(VM_Context);
        case 3: return Do_Read_Input(VM_Context);
        default: return 1;
    }
}

bool VMInstructionDecoder(TInstructionBuffer* InstructionPtr, byte *VMEIP_RealAddr) { // .text:00401000
    TInstrTag InstrType;
    byte LowNib, HiNib;
    AddrSize;
    JccIndex;
    dword *ExaminedDwords;
    TempInstrSize;
    dd* TempPtr;
    ParamsCount;
    Temp;
    wTemp;
    bTemp;
    memset(InstrBuf, 0, 0x13*4);
    InstructionPtr->WorkSubField = &InstructionPtr->ParamDest; // set which is the first decoded param
    InstrType = VMEIP_RealAddr[0]; //*(byte *)VMEIP_RealAddr
    InstructionPtr->InstrType = InstrType.InstrType; //b&0x3F; // ==00111111b
    //swith(InstrType>>6) { // the sub is needed for setting flags!
    swith(InstrType.AddSize)
    case 0: AddrSize = 1; break;
    case 1: AddrSize = 2; break;
    case 2: AddrSize = 4; break;
    default: return 0;
    };
    InstructionPtr->OperandSize = AddrSize;
    //ParamIdx= InstructionPtr->InstrType<<4; // *structure size
    ParamIdx = InstructionPtr->InstrType;
    if (ParamTable[ParamIdx].ID == 0x33) return 0; // 0x33 entries has no associated instruction
    if ( (char)ParamTable[ParamIdx].ParamDest == 4 && AddrSize != 4) return 0;
    InstructionPtr->InstrID = ParamTable[ParamIdx].ID; // Jump Address!
    ExaminedParams = 0;
    TempInstrSize = 1; //0 was already used for getting here!!
    ParamsCount = 0; // decode the first param, so!
    // ParamTable[ParamIdx].Params[ParamsCount]; // 401099
    while (ParamTable[ParamIdx].Params[ParamsCount] != 0) { // .text:004010B1 param decoding loop
        ParamsValue = ParamTable[ParamIdx].Params[ParamsCount];
        LowNib_RegIdx = VMEIP_RealAddr[TempInstrSize] & 0x0F;
        HiNib_AddrMode = VMEIP_RealAddr[TempInstrSize] >> 4;
        InstructionPtr->WorkSubField[ParamsCount].AddressType = HiNib;
        InstructionPtr->WorkSubField[ParamsCount].field8 = ParamTable[ParamIdx].Params[ParamsCount];
        switch (HiNib_AddrMode) { // NOTE: switch on decoded address type!!
            case vmaRegister: // 0
            case vmaRegisterAddress: // 1
                InstructionPtr->WorkSubField[ParaCount].RegisterIdx = LowNib_RegIdx;
                TempInstrSize++;
                break;
            case vmaVMValue_orC4__or_displacement: // 3 .text:00401134
                if ( (char)ParamTable[ParamIdx].Params[ParamsCount] == 2) return 0;
                TempInstrSize++;
                switch(InstructionPtr->OperandSize) {
                    case 1:
                        bTemp = ((byte *)VMEIP_RealAddr)[TempInstrSize];
                        InstructionPtr->WorkSubField[ParamsCount].VMValue = (dword)bTemp;
                        TempInstrSize++;
                        break;
                    case 2:
                        // this might be an instrinsic inline function, due to code shape (compiler didnt recon param 2 was 0)
                        wTemp = ((word *)VMEIP_RealAddr)[TempInstrSize];
                        wTemp = SWAP(wTemp);
                        InstructionPtr->WorkSubField[ParamsCount].VMValue = (dword)wTemp;
                        TempInstrSize += 2;
                    case 4:
                        break;
                    default: return 0;
                }
            case vmaDirectAddress: // 2
                if (HiNib_AddrMode == vmaDirectAddress) { //added by me to keep flow
                    TempInstrSize++;
                    if (InstructionPtr->OperandSize != 4) return 0;
                }
                // .text:00401101 common code to case 2 and 3 here...
                Temp = ((dword *)VMEIP_RealAddr)[TempInstrSize];
                Temp = SWAP(Temp);
                InstructionPtr->WorkSubField[ParamsCount].VMValue = (dword)Temp;
                TempInstrSize += 4;
                break;
            default:
                return 0;
        }
        ParamsCount++; // next param data!
        ExaminedParams++;
        if (ParaCount >= 3) break; // max 32 bytes fetched this way
    };
    InstructionPtr->InstructionParamsCount = ExaminedParams;
    InstructionPtr->InstrSize = TempInstrSize;
    return TempInstrSize;
}

#define BETWEEN(x, loBound, hiBound) ((x>loBound)&&(x<hiBound)?true:false)
#define RANGE(x, lowLimit, RangeFromLimit) ((x>lowLimit)&&(x<(lowLimit+RangeFromLimit))?true:false)

MemorySize = 4096;
initStack = SWAP(1);
initCode = SWAP(0x6EEFF);
byte * program;
int * OpcodeProc[];

int main() {
    dword RealEIP;
    TVMContext VMContext;
    TInstructionBuffer InstBuff;
    int res, MachineCheck;
    int c1, c2;
    char Opcode;
    /* 1. initialize VM */
    memset(VMContext, 0, 30*4);
    VMContext.Registers = &VMContext;
    if (*program != 0x102030) exit(1);
    //.text:004020A1
    VMContext.ProgramMemoryAddr = malloc(MemorySize + 16);
    if (VMContext.ProgramMemoryAddr == 0) exit(1);
    VMContext.InitCode = initCode;
    VMContext.MemorySize = MemorySize;
    memcpy(VMContext.ProgramMemoryAddr, program, 2580 + 1);
    VMContext.StackMemoryAddr = malloc(MemorySize);
    VMContext.StackMemoryInit = initStack;
    if(VMContext.StackMemoryAddr == 0) exit(1);
    //.text:00402111
    VM_EIP = initApp + 28;
    VMContext.StackMemoryAddr = MemorySize;
    VMContext.VM_ESP = VMContext.StackMemoryInit;
    c1 = mcGenericError_or_CannotWriteTo;
    c2 = mcInputOutput;
    /* 2. start main VM Loop */
    while (true) { // .text:00402138
        // VM_Loop_Head_Default: .text:0040215B
        VMContext.InstructionCounter++;
        if (VMContext.VM_EFLAGS == TF) // Step-flag for debugging purposes (code removed)
            VMContext.MachineControl = mcStepBreakPoint;
        else {
            //--->body<--- .text:00402177
            VMContext.VMEIP_Saved_Prior_InstrExec = VM_EIP;
            /* 3. process a VM Instruction and execute it */
            MachineCheck = VMAddress2Real(&VMContext, VM_EIP, &RealEIP);
            if (!MachineCheck) {
                MachineCheck = VMInstructionDecoder(&InstBuff, RealEIP);
                if (!MachineCheck) // check for opposite behavoir...
                    VMContext.MachineControl = c1;
                else {
                    Opcode = *RealEIP;
                    MachineCheck = (*OpcodeProc[(char)Opcode])(&InstBuff, &VMContext);
                    if (!MachineCheck) continue; // check for opposite behavoir...
                }
            }
            /* 4. if we have a MachineCheck to do, ensure to catch the 'IO' one */
            if (MachineCheck && VMContext.maybe_MachineControl == c2) { // VM loop end. c2==mcInputOutput
                CheckForInputOutput(&VMContext);
                continue;
            }
        }
        /* 5. perform the exception check */
        // VM_MachineErrCheck_OrEndOfVM:
        if (VMContext.VM_ResumeExec == 0) return 0;
        VM_EIP = VMContext.VM_ResumeExec;
        VMContext.VM_ResumeExec = 0;
    }
    // end....
};
```



> 1 老实说，刚找到这个挑战的时候我打开了 `Olly`，为了检查它是否加壳。

> 2 字节序是指机器中一个字节（和位）的顺序。`X86` 是小端处理器，因而保持最低字节（位序也一样）在最后。大端机器保持高字节在最后，但是通常在位一级是保持小端序。

> 3 这是加倍的正确-由于我的过失- 在这篇文章测试版的时候在思想中我错误地转化了位序。

> 4 不同的虚拟机显然使用不同的分配方案。

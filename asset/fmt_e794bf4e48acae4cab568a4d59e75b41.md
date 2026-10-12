
## 注释

- `; option in the layer generator.`
- 上述 Layer 由我编写的简易 Layer 生成器 `Layer generator Engine` 生成。已为读者添加了注释。

## Layers Blocks

除了 `obfuscation/junk198.inc`，我还添加了一些 `junks` 宏。

```asm
dec_loader175_1: ; 解密标签
xor byte ptr [edx],cl ; 使用默认选项
; 简单的加密
inc edx ; 代码解密++
dec ecx ; 循环索引
test ecx, ecx ; 校验 ECX 是否等于 0。
jnz dec_loader175_1 ; 非也:( 继续解密).
```

如果打开加密器中的选项，每个 Layer 的加密将会不同。

```asm
lea edx , [ebp+bpxcheck175_1] ; 获取 BPX 校验地址.
cmp byte ptr [edx],0CCh ; 有断点吗?
jnz return175_1 ; 不错。好孩子。
rdtsc ; 它放了一个 bpx..
; EAX =任意值
push eax ; eax 入栈
ret ; 返回 :)崩了你：-）
return175_1: ; on return block
include obfuscation/junk199.inc ; 垃圾代码
include obfuscation/junk19A.inc ; 同上.
SEHBLOCK 66137317 28513829 ; 以 key 作为参数的 SEH block 宏
ret ; 返回
inst175_2_1: ; 又一个代码 block
add dword ptr [esp], 41952561 ; 修正返回地址并返回
inst175_3_1: ; .
mov ecx, (offset _end174_1- @layer175_1) ; 大小
add dword ptr [esp], 13007360 ;
inst175_1_1: ; 另一个 block
lea edx, [ebp+@layer175_1] ; 得到 layer 地址
add dword ptr [esp], 30560857 ; 修正返回地址并返回
ret
```

`EIPtable175_1 dd 000DEADh, (offset inst175_1_1 - offset startloader), (offset inst175_2_1 - offset startloader), (offset inst175_3_1 - offset startloader), (offset _end174_1 - offset startloader)`

这是一张用来重定向代码的偏移表。

`RETable175_1 dd 0031000h, (offset return_addy175_1 - offset startloader - 30560857) , (offset return_addy175_1 - offset startloader - 41952561), (offset return_addy175_1 - offset startloader - 13007360),(offset return_addy175_1 - offset startloader - 37623488)`

这是一张经过一定“加密”的返回地址表。

注意表中的第一个成员：`DDH` 和 `31000H`。它们是常数并可以在加密器的 `RANDOM_FIRST_BLOCK` 选项中任意取值。

### Protection Weakness

Those layers have a few weaknesses. You can use `BPM` (Hardware Break Point) on the next layer once you have passed the `SEH` that is going to clear the debug registers. Another weakness is the static size of the layer. Using this information, one can pass the layers rather quickly with a few `Soft ICE` macros for instance. I didn't turn the random size option on, on purpose to allow such attacks.

此 Layer 包含了一些漏洞。其中一个是：只要通过了清空寄存器的 `SEH`，你就可以在下个 Layer 上设置硬件断点。另一个是由于 Layer 的固定大小。只要运用一些如 `SoftICE` 宏的插件就可以很快地通过 Layer。我禁止了 `RANDOM SIZE` 选项，以允许这种攻击。

Those layers always use the same encryption algo, which can allow one to write scripts to decrypt the binary. And as you can read in a few submissions, some people did it. I did put this weakness on purpose as well. In a challenge i had done in the past, i had used random encryption for each layers, this time i choose not to use it. It is possible to bypass the 175 layers in a few seconds easily as well using a live approach. As we know wich API functions are going to be used, we can set a break point after the `BPX` checks have occured. Another possibility is to create a little utility that will `PATCH` the system dll in memory (each application has a copy of the dll) and to redirect them to a place that you contol. This way you can put breakpoints without triggering any Detection code.

另外 Layer 相同加密方式的使用，使得有人可以写出解密所需的脚本文件。一些人就做到了。我也故意放过了这个漏洞。以前，我任意加密每个 Layer，这次我没有。你可以利用现成的脚本文件很容易的通过这 175 个 Layer。你可以在 `BPX` 校验后对 API 函数设置断点。另一种可能就是内存中的系统 DLL（每个应用程序都用到的）进行修改，使它们重定向到你所控制的范围。这种方法可以使设置的断点不触发任何侦测代码而获得成功。

Talking of patching the Windows dll files, it is possible to patch `ntdll` to avoid the Debug Registers access in the context structure, by hooking the Exception Handling Mechanism of Windows. This allows one to put Hardware Breakpoints anywhere without ever having problems, never seeing his debug breakpoints beeing erased etc. The cool thing is you don't even need a Kernel Mode Driver to do that. I leave this as an exercice for interested people.

说到这里，`NTDLL` 的修改可以避免寄存器通过挂起系统的异常机制（hooking the exception handling mechanism of windows）访问 `CONTEXT STRUCTURE`。因此设置断点将会很顺利，也不用担心被擦除。你甚至连 `Kernel Mode Driver` 都可弃之一边。留下这个供大家练习。

### Virtual Machine

The final protection of the binary is a complete Virtual Machine i wrote for the challenge. I have designed a Virtual CPU that will interpret my own Assembly language. The Virtual Machine is quite simple to understand and isn't very complex. Virtual Machines seem to be a new trend in protection systems, so i thought it could be a good thing to write one for such a challenge. The instruction encoding is very trivial, and could have been a lot harder to understand. The first Version i had in mind was a lot more complex. I wanted not only to have a pseudo language, but also to program the instructions handlers emulating real x86 instructions. Each handler would be a few hundred instructions long and a lot harder to analyse. A small program has been written with this Virtual Machine Assembly language, and it was used to authenticate the user running the binary.

Read next part for further informations

虚拟设备

最后一种代码保护是我为这次挑战而写的虚拟设备。我设计了一个用来解释自身汇编语言的虚拟 CPU。这种保护很容易理解，并不复杂。

虚拟设备在各种保护方式中是新兴事物，因此我觉得很有必要在我的挑战中写出来。指令编码既微不足道又难于理解。因此我的首个版本是十分复杂的。不仅是伪语言，我还需要写出指令的操作过程。

如果仿效 x86 指令，每个操作将包含上百的指令且很难分析。

有一种用虚拟设备汇编语言写成的小程序，可以被用来鉴别作者运行的代码。

阅读下面的部分获取进一步的信息

### 3. Something uncommon has been used to protect the code from beeing reverse engineered, can you identificate what it is and how it works?

Even though, a few protection systems are using some kind of Virtual Machines, those aren't very common. Especially in Malwares and other exploits.

3.一些反逆向的特殊保护方式，你能认出它们并知道工作原理吗？

虽然，一些程序使用虚拟设备保护自身，但是这种方式仍然不普遍。特别是在病毒和其它开发中。

### Virtual CPU description and Inner working

#### Registers

```asm
REGISTERS STRUC
R0_ dd ? ; 000
R1_ dd ? ; 001
R2_ dd ? ; 002
COUNTER_ dd ? ; 003
EIP_ dd ? ; 004 -> reserved
STATE_ dd ? ; 005
REGISTERS ENDS
```

这是我源代码的原始结构。每个寄存器都是双字。有些寄存器没有使用，因为它们是为将来新的虚拟设备保留的。就是“`EIP_`”。我打算每个指令中添加其它的信息，但是我没有这样做，因为我不想程序过于复杂。但我会修改所有指令的指示器。这使得代码流程变的十分疯狂。文件中指令顺序将不会影响真正流程的执行。

`STATE` 寄存器是最小的 `E Flags`。此寄存器的修改依赖与其它的指令。

`COUNTER` 寄存器在循环指令中使用。循环中常用的是 `ECX` 寄存器。

```asm
regs REGISTERS <>
R0 equ 000b
R1 equ 001b
R2 equ 010b
COUNTER equ 011b
EIP equ 100b
STATE equ 101b
```

Here are a few other definitions used in my program. I started to represent the registers in binary because i wanted to do complex opcode decoding. I will do that for another version ;)

Registers Initialization

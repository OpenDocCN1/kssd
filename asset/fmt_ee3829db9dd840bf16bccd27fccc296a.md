
## 寄存器初始化

```
mov dword ptr [regs.R0],"livE" ; Registers are initialized with a
; Slayer Song Title.
..
mov dword ptr [regs.R1],"saH " ; Evil Has No Boundaries!
..
mov dword ptr [regs.R2]," oN "
..
mov dword ptr [regs.COUNTER],"nuoB"
..
mov dword ptr [regs.EIP],"irad"
..
mov dword ptr [regs.STATE],"! se"
```

在 VM 的开头，我首先使用一个金属打击乐队的曲名初始化我的寄存器。选择这个曲名是因为我计划在虚拟机中进行一些邪恶的操作。虽然它不如初始版本复杂，但足以保留那个有趣的字符串 ;)。

虚拟机包含约 34 条指令。（我根据指令不同的用途进行计数）

我将展示几个指令处理器来解释虚拟机的内部工作机制，但不会在此展示所有指令。

我赋予了程序一些新的定义。我在二进制代码中描述寄存器，这是因为我想对操作码进行完全的解码。这将在下个版本中实现。

寄存器初始化：

```
mov dword ptr [regs.R0],"livE" ;寄存器以曲名初始化
..
mov dword ptr [regs.R1],"saH " ; 恶到极点了
..
mov dword ptr [regs.R2]," oN "
..
mov dword ptr [regs.COUNTER],"nuoB"
..
mov dword ptr [regs.EIP],"irad"
..
mov dword ptr [regs.STATE],"! se"
```

在 VM 的开头，我使用一个金属打击乐队的曲名对寄存器进行初始化。选择它是因为我将会用虚拟设备“作恶”。

尽管没有第一个版本复杂，但是仍然差不了多少。

虚拟设备包含 34 种指令。（根据各个指令独特的作用区分）

我会通过一些指令操作来解释虚拟设备的工作原理，但不是全部（指令）。

## Pcode Fetcher

虚拟机在寄存器初始化后的第一件事就是获取 Pcode 的入口点并跳转到第一个 Pcode 处理器。

```
movzx eax, byte ptr [esi] ; ESI is Pcode Entry Point. This code
; gets the first instruction Prefix.
mov edi, dword ptr [eax*4+poffset] ; It uses it with the offset table to
; find the Pcode family it has to
; execute.
movzx eax, byte ptr [esi+1] ; get second byte, use it as an
; index into last table.
; The VM now knows what instruction it
; has to emulate and goes to it.
JMPNEXT ; Emulate a jmp dword ptr [eax*4+edi]
; with Exception Handling and
; Context Manipulation.
; Jmp to the next Pcode
; instruction handler
```

获取 Pcode:

虚拟设备的寄存器初始化后的第一件事就是获取 Pcode 的入口点并来到首个 Pcode 处理中。

```
movzx eax, byte ptr [esi] ; ESI 是 Pcode 入口点.此代码得到首个指令名称
mov edi, dword ptr [eax*4+poffset] ; 使用偏移表找到 Pcode 执行什么
movzx eax, byte ptr [esi+1] ;得到第二个字节，作为最后表的索引
VM 得到仿效的指令并执行
跳向下个; 使用异常操作和 Context 仿效 jmp dword ptr [eax*4+edi]
;跳向下个 Pcode 指令操作
```

## Examples of Instructions implemented inside the Virtual Machine

在开始这些示例之前，我想说明一下虚拟机中存在的一些指令未被使用，而是被留作陷阱。其中三条使用了自修改代码（Self-modifying code）。有人报告说它们不起作用，但它们应该能工作。这种一位差异（off by one）是因为操作码是从其他指令处理器中被调用的。

虚拟设备指令实现的实例：

在我开始实例之前，我想讲述一下一些虚拟设备中未被使用，并被留下作为一种陷阱的指令。其中的三个使用了自校验代码。

人们认为它们是不起作用的，但事实正好相反。这是因为操作码是从其它指令操作中被调用的。

有两条指令在需要执行特定代码段时会在线修改一条指令，随后恢复该指令的状态。我太懒了，没有检查这些指令是否真的有漏洞，或者是否没有使用正确的参数。其中一条`unused`指令确实有一个漏洞，我很高兴有人注意到了它。由于该指令未被使用，因此它只是一个陷阱指令。

该指令本应是一个虚拟`BSWAP`，但它没有保存交换的结果。另一条未使用的指令是`INT 3`。这条指令允许用户在 Pcode 程序中设置断点，并使用调试器从该指令开始跟踪。我在最终的虚拟机中保留了这条指令，很高兴看到有人发现并滥用了它！

### STOPVM

这里展示的第一条指令非常简单。它告诉虚拟机停止，程序将返回到普通的 `x86` 汇编程序。

```
@STOPVM:
pop dword ptr fs:[0] ; Im using SEH to jmp from handlers to
;handlers in the VM. 在 VM 中，我使用 SEH 从一个操作跳到另一个操作。
add esp,4 ; Therefore i need to remove the handler
; installed before i do anything. 因此我需要在开始之前删除安装操作
popad ; i restore the registers.. 恢复寄存器
..
push dword ptr [Pret] ; Put the Return Address (to get out of
; the VM) on the stack. 把返回地址放入堆栈中（从 VM 中退出）
..
xor [esp],'HAX0 '; Decrypt it with a funny string: （用有趣的字符串进行解密）
;HAXO(R)
..
ret ; Get out of the VM. 退出 VM。
```

此指令没有使用任何字节码获取器，因为它不需要跳转到另一个处理器。现在我将展示一条真正的指令：一个虚拟的入栈指令。

### LOAD

```
@Load:
pop dword ptr fs:[0]
add esp,4
popad
; Same as every handler, remove SEH and restore registers. 与每个操作一样，删除 SHE 恢复寄存器。
mov eax,dword ptr [esi+2] ; Get into EAX the first operand
; of the instruction. 进入 EAX，指令的首个操作数
xor eax,37195411h ; Decrypt it. 解码
push eax ; Push it onto the stack. 压入堆栈
mov eax,0FFFFFF3Fh ; EAX = FFFFFF3Fh
not eax ; EAX = not(EAX) = C0h
shr eax,5 ; EAX = EAX shr 5 = 6 :
; This is the instruction length 这是指令长度
lea esi, [esi+eax] ; ESI = Instruction Pointer.指令指针
; Displace the Instruction Pointer 显示指令指针
; 6 bytes further. 6+字节
movzx eax, byte ptr [esi] ; ESI now points to the new
; instruction to be executed. ESI 指向将执行的新的指令
mov edi, dword ptr [eax*4+poffset] ; It uses it with the offset
; table to find the Pcode family 使用偏移表找到需要执行的 Pcode 系
; it has to execute.
movzx eax, byte ptr [esi+1] ; get second byte, use it as an
; index into last table. 得到第二个字节，作为最后表的索引
; The VM now knows what instruction it

; has to emulate and goes to it. 现在 VM 得到仿效什么指令并执行
JMPNEXT ; Emulate a jmp dword ptr [eax*4+edi]; with Exception Handling and
; Context Manipulation. 跳向下个。使用异常操作和 context 仿效 jmp dword ptr [eax*4+edi]

; Jmp to the next Pcode 跳向下个 Pcode 指令操作
; instruction handler
```

从这个小小的处理器可以看出，该指令长度为 6 个字节。它只接受一个参数，该参数位于指令起始处向后 2 字节的位置（`ESI+2`）。该参数使用 `37195411h` 进行加密。解密后的参数被压入堆栈，然后虚拟机调用下一条指令。

因此，我们可以说这条指令是一个 push。由于 `push` 已经是 `x86` 指令，因此我将我的虚拟入栈指令命名为 `LOAD`。

可以这样使用它：`LOAD number`

### VMXOR

```
@VMXORDISPATCHER:
pop dword ptr fs:[0]
add esp,4
popad
; Same as every handler, remove SEH and restore registers. 与每个操作一样，删除 SEH 并恢复寄存器
movzx eax, byte ptr [esi+2] ; Get the Index Register to acces
; the Virtual CPU registers. 得到索引寄存器访问虚拟 CPU 寄存器
mov eax, dword ptr [regs+eax*4] ; edi = Register value to know
; which register is going to be EDI=寄存器值以得到与哪个寄存器有关
; concerned (RO, R1 , R2)
; EAX = value used by the XOR. EAX=XOR 使用的值
```

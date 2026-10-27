
在我的演示追踪器中，我只是简单地将受保护的二进制文件`LoadLibraryExA`，初始化一个`vmtracer`类，修补 vm 处理程序表，然后调用模块的入口点。这不是很理想，但对于演示目的来说，这就足够了。

```cpp
// patch vm handler table...
tracer.start();

// call entry point...
auto result = reinterpret_cast<int(*)()>(
    NT_HEADER(module_base)->OptionalHeader.AddressOfEntryPoint + module_base)();

// unpatch vm handler table...
tracer.stop();
```

现在已经创建了一个跟踪文件，我们现在可以通过`vmprofiler-cli`或`vmprofiler-qt`来检查跟踪。不过我建议使用后者，因为这个程序是明确为查看跟踪文件而创建的。

当把跟踪文件加载到`vmprofiler-qt`中时，必须知道`vm_entry` RVA 以及在 PE 文件的可选头中发现的镜像基础。考虑到所有这些信息以及原始的受保护二进制文件，`vmprofiler-qt`将显示跟踪文件中的所有虚拟指令，并允许你“单步”通过它。

让我们看一下这个跟踪文件，看看我们是否能找到原来的指令，这些指令现在已经被转换为基于 RISC、堆栈机的架构。在`vm_entry`之后执行的第一个代码块似乎不包含与原始二进制文件有关的代码。它在这里只是为了混淆视听，防止对虚拟指令进行静态分析，因为要了解虚拟`JMP`指令的落点，需要对虚拟指令集进行仿真。这第一个跳转块位于每个受保护的二进制文件中。

![](https://bbs.kanxue.com/upload/attach/202212/754019_M43YUCN8SP5ASSU.png)

在虚拟`JMP`指令之后的下一个块做了一些与堆栈有关的有趣的数学运算。如果你仔细观察，你可以看到正在执行的数学运算是：`sub(x, y) = ~((~(x) & ~(x)) + y) & ~((~(x) & ~(x)) + y); sub(VSP, 10)`。

如果我们简化这个数学操作，我们可以看到这个操作是对`VSP`做的减法。`sub(x, y) = ~((~x) + y)`。这等同于本地操作`sub rsp, 0x10`。如果我们看一下原始的二进制，也就是没有被虚拟化的二进制，我们可以看到事实上有这个指令。

![](https://bbs.kanxue.com/upload/attach/202212/754019_DRCAF6NKCW7NM4W.png)

在虚拟指令中可以看到上面显示的`MOV EAX, 1`紧随在`VSP`上做的减法之后。`MOV EAX, 1`是通过一个`LCONSTBSX`和一个`SREGDW`完成的。`SREG`的位数与本地寄存器的宽度 32bits 相匹配，同时也与被加载的常数值相匹配。

![](https://bbs.kanxue.com/upload/attach/202212/754019_9ZEJD6ET7XD8ZDS.png)

接下来我们看到一个`vmexit`发生了。我们可以通过查看`vmexit`之前的最后一个`ADDQ`来了解代码执行在虚拟机之外的情况。堆栈上的前两个值应该是模块的基本地址和将被返回的例程的 32 位相对虚拟地址。在这个跟踪中，RVA 是`0x140008236`。如果我们在 IDA 中检查这个地址，我们可以看到指令`CPUID`在这里。

```asm
.vmp0:0000000140008236 0F A2                                         cpuid
.vmp0:0000000140008238 0F 81 88 FE FF FF                             jno     loc_1400080C6
.vmp0:000000014000823E 68 05 02 00 79                                push    79000205h
.vmp0:0000000140008243 E9 77 FD FF FF                                jmp     loc_140007FBF
```

正如你所看到的，在`CPUID`指令之后，代码执行直接回到了虚拟机中。在用位于虚拟堆栈上的本地寄存器值设置所有的虚拟 scratch 寄存器后，直接将一个常数加载到堆栈中，其值为`0x1C`。然后，`CPUID`的结果值被这个常数移到了右边。

![](https://bbs.kanxue.com/upload/attach/202212/754019_SVFMPHJHYCE5V3N.png)

`AND`操作是通过两个`NAND`操作完成的。第一个`NAND`操作简单地反转了`SHR`的结果；`invert(x) = ~(x) & ~(x)`。这是通过将`DWORD`值两次加载到堆栈中来完成的，以形成一个`QWORD`。

![](https://bbs.kanxue.com/upload/attach/202212/754019_BHPBGVH3D759G5G.png)

这个`AND`操作的结果将被设置到虚拟 scratch 寄存器 7（`SREGDW 0x38`）。然后，它被移到 scratch 寄存器 16 中。如果我们看一下`vmexit`指令和`LREGQ`的执行顺序，我们可以看到这确实是正确的。

![](https://bbs.kanxue.com/upload/attach/202212/754019_8A5RNHNJAB3VZVM.png)

最后，我们还可以看到`ADD`指令和`LVSP`指令向`VSP`增加了一个值。这是预料之中的，因为在原始二进制中存在一个`ADD RSP, 0x10`。

![](https://bbs.kanxue.com/upload/attach/202212/754019_6VXCYKBAKS7DSXP.png)

从上面的信息我们可以重建以下的本地指令。

```asm
sub rsp, 0x10
mov eax, 1
cpuid
shr ecx, 0x1C
and ecx, 1
mov eax, ecx ; from the LREGDW 0x38; SREGDW 0x80...
add rsp, 0x10
ret
```

正如你所看到的，有几条指令被遗漏了，特别是`RBX`的`push`/`pop`，以及`XOR`将`ECX`的内容清零。我认为这些指令没有直接转换为虚拟指令，而是以一种迂回的方式实现。

# 改变虚拟指令的结果

为了改变虚拟指令，必须首先重新实现整个 vm 处理程序。如果 vm 处理程序对第二个操作数进行解密，必须记住解密密钥有效性的重要性。因此，必须计算出原始的即时值，并通过原始转换应用到解密密钥上。然而，在更新解密密钥后，这个值可以随后被丢弃。这方面的一个例子可以是改变上节中`SHR`之前的`LCONST`的常数值。

![](https://bbs.kanxue.com/upload/attach/202212/754019_DARAZ7BX24YV9RM.png)

这个虚拟指令有两个操作数，第一个是要执行的 vm handler 索引，第二个是即时值，在这个例子中是一个单字节。由于有两个操作数，所以在 vm 处理程序中会有五个转换。

![](https://bbs.kanxue.com/upload/attach/202212/754019_KBN6PXSUA4TCXK3.png)

我们可以重新编码这个 vm 处理程序，将解密后的即时值与`0x1C`进行比较，然后分支到一个子程序，将一个不同的值加载到堆栈。这将导致`SHR`计算一个不同的结果。本质上我们可以欺骗`CPUID`的结果。另一种方法是重新创建`SHR`处理程序，然而为了简单起见，我只是要转移到一个被设置的位。在这种情况下，如果支持`VMX`，`CPUID`之后的`ECX`第 5 位将被设置，由于我的 CPU 支持虚拟化，这个位将是高的。下面是新的 vm 处理程序。

```asm
.data
__mbase dq 0h
public __mbase

.code
__lconstbzx proc
    mov al, [rsi]
    lea rsi, [rsi+1]
    xor al, bl
    dec al
    ror al, 1
    neg al
    xor bl, al

    pushfq            ; save flags...
    cmp ax, 01Ch
    je swap_val

    ; the constant is not 0x1C
    popfq            ; restore flags...
    sub rbp, 2
    mov [rbp], ax
    mov rax, __mbase
    add rax, 059FEh

    ; calc jmp rva is 0x59FE...
    jmp rax

swap_val:            ; the constant is 0x1C
    popfq            ; restore flags...
    mov ax, 5        ; bit 5 is VMX in ECX after CPUID...
    sub rbp, 2
    mov [rbp], ax
    mov rax, __mbase
    add rax, 059FEh

    ; calc jmp rva is 0x59FE...
    jmp rax
__lconstbzx endp
end
```

如果我们现在再次运行`vm tracer`，将这个新的 vm 处理程序设置为索引`0x55`，我们应该能够看到`LCONSTBZX`的变化。为了方便这个钩子，我们必须把新的 vm 处理程序的虚拟地址设置为`vm::handler::table_t`对象。

```cpp
// change vm handler 0x55 (LCONSTBZX) to our implimentation of it…
auto _meta_data = handler_table.get_meta_data(0x55);
_meta_data.virt = reinterpret_cast<u64>(&__lconstbzx);
handler_table.set_meta_data(0x55, _meta_data);
```

如果我们现在运行二进制，它将返回 1。你可以在下面看到这个。

![](https://bbs.kanxue.com/upload/attach/202212/754019_3DHPYRA8SRA37PU.png)

# 编码虚拟指令 - 逆向转换

由于 VMProtect 2 生成的虚拟机执行的是以其自身字节码编码的虚拟指令，因此，如果能够对其进行编码，就可以在虚拟机上运行自己的虚拟指令。编码的虚拟指令也必须在 4GB 的地址空间范围内，因为虚拟指令的 RVA 是 32bits 宽。在本节中，我将编码一组非常简单的虚拟指令，将两个`QWORD`值相加并返回结果。

首先，对虚拟指令进行编码需要在二进制文件中加入上述虚拟指令的 vm 处理程序。通过`vmprofiler`来定位这些 vm 处理程序。vm 处理程序的索引是第一个操作码，即时值（如果有的话）是第二个。结合这两组操作数将产生一个编码的虚拟指令。这是组装虚拟指令的第一阶段，第二阶段是对操作数进行加密。

一旦我们有了编码的虚拟指令，我们现在就可以使用 vm handler 变换的逆操作以及`calc_jmp`的逆操作来加密它们。值得注意的是，在加密时必须考虑到`VIP`的推进方式，因为操作数和虚拟指令的顺序取决于这个推进方向。

![](https://bbs.kanxue.com/upload/attach/202212/754019_58PZXQ8PTVG43ND.png)

为了执行这些新装配的虚拟指令，必须把虚拟指令放在`vm_entry`例程的 32 位地址范围内，然后把这些虚拟指令的加密`rva`放到堆栈中，最后再调用到`vm_entry`。我建议使用`VirtualAllocEx`在受保护模块的正下方分配一个 RW 页面。下面显示了一个运行虚拟指令的例子。

```cpp
SIZE_T bytes_copied;
STARTUPINFOA info = { sizeof info };
PROCESS_INFORMATION proc_info;

// start the protected binary suspended...
// keep in mind this binary is not packed...
CreateProcessA("vmptest.vmp.exe", nullptr, nullptr,
    nullptr, false, CREATE_SUSPENDED | CREATE_NEW_CONSOLE,
    nullptr, nullptr, &info, &proc_info);

// wait for the system to finish setting up...
WaitForInputIdle(proc_info.hProcess, INFINITE);
auto module_base = get_process_base(proc_info.hProcess);

// allocate space for the virtual instructions below the module...
auto virt_instrs = VirtualAllocEx(proc_info.hProcess,
    module_base + vmasm->header->offset,
    vmasm->header->size,
    MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);

// write the virtual instructions...
WriteProcessMemory(proc_info.hProcess, virt_instrs,
    vmasm->data, vmasm->header->size, &bytes_copied);

// create a thread to run the virtual instructions...
auto thandle = CreateRemoteThread(proc_info.hProcess,
    nullptr, 0u,
    module_base + vm_entry_rva,
    nullptr, CREATE_SUSPENDED, &tid);

CONTEXT thread_ctx;
GetThreadContext(thandle, &thread_ctx);

// sub rsp, 8...
thread_ctx.Rsp -= 8;
thread_ctx.Rip = module_base + vm_entry_rva;

// write encrypted rva onto the stack...
WriteProcessMemory(proc_info.hProcess, thread_ctx.Rsp,
    &vmasm->header->encrypted_rva,
    sizeof vmasm->header->encrypted_rva, &bytes_copied);

// update thread context and resume execution...
SetThreadContext(thandle, &thread_ctx);
ResumeThread(thandle);
```

# 结论 - 静态分析，动态分析

最后，我的动态分析方案不是最理想的方案，但是它应该允许对受保护的二进制文件进行基本的反向工程。随着时间的推移，对虚拟指令的静态分析将成为可能，然而目前动态分析将不得不做。在未来，我将使用`unicorn`来模拟虚拟机的处理程序。

尽管我已经记录了少数的虚拟指令，但还有更多的指令没有被记录。我记录这些虚拟指令的目的是让本文的读者获得一种感觉，即虚拟机处理程序应该是什么样子，以及如何改变这些虚拟机处理程序的结果。本文中记录的虚拟指令也是最常见的。这些虚拟指令很可能会出现在每个虚拟机中。

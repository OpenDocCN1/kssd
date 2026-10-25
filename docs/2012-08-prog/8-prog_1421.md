
# 利用中断向量(IDT)进行代码的防御和攻击

> 原文：[`www.pediy.com/kssd/pediy11/110372.html`](https://www.pediy.com/kssd/pediy11/110372.html)

## 一. 科普一下

`IDT`，即中断向量表，就是一个以 `KIDTENTRY` 构成的结构数组，一共有 256 项。我们可以用 `windbg` 了解其结构：

```
kd> dt nt!_KIDTENTRY

   +0x000 Offset           : Uint2B
   +0x002 Selector         : Uint2B
   +0x004 Access           : Uint2B
   +0x006 ExtendedOffset   : Uint2B
```

由 `Offset` 表示的高位和 `ExtendedOffset` 表示的低位组成的 4 字节就构成了一个中断向量的处理函数地址。当一个中断或异常被触发时，

CPU 就会保存当前线程的上下文信息，并将控制权交给中断处理程序。`IDT` 的头 32 个向量用于异常处理，它们是 Intel 预先定义的。

`IDT` 中包含了 3 个门描述符，分别是中断门，任务门和陷阱门。在这里我们需要关注的是中断门和陷阱门。中断门和陷阱门的不同点在于他们被触发时的 `EFLAGES` 寄存器中的 `IF` 标志位的状态。当中断或异常是由中断门引起时，CPU 会自动清除 `IF` 标志位，如果是由陷阱门引起的，则不会影响 `IF` 标志位。

异常可以归为 3 类：陷阱(`Trap`)，错误(`Fault`)和终止(`Aborts`)。终止通常表示严重错误，出错的任务不允许再获得重新运行的机会，例如 `Machine Check` 异常(`int 0x12`)。陷阱和错误允许出错的任务在异常得到处理后继续运行。陷阱和错误的区别在于保存在系统栈中的中断返回地址，对于错误类异常，其返回地址指向导致异常的那一条指令所在的地址，在异常处理完毕后，CPU 将任务的上下文状态恢复到异常指令执行前的状态并重新执行此指令，例如缺页异常(`int 0xE`)。而对于陷阱类异常，中断返回地址指向触发异常的指令的下一个地址，这样就不再执行触发异常的指令了。

在此我们需要特别关注的是调试异常(`int 1`)，它可以由 `trap` 或 `fault` 触发。如下所示：

1. 指令断点
2. 内存访问断点
3. `IO` 端口访问断点
4. `General detect condition` (当 `DR7` 的 `GD` 位设为 1)
5. 当 `EFLAGES` 寄存器中的 `TF` 标志位被设置的时候，每执行一条语句都会产生调试中断，即所谓的单步中断
6. 由 `Int 1` 指令触发

在指令断点和 `GD` 触发类型是 `fault`，其他的触发类型是 `traps`，表示异常处理后的返回地址是触发异常的下一条指令。

我们要控制调试处理程序运行，那就必须了解一下与之密切相关的调试寄存器。

### 调试寄存器

`IA-32` 有 8 个调试寄存器，分别为 `DR0`~`DR7`。`DR1`~`DR3` 是调试地址寄存器，用来指定断点的地址，即通常所说的硬件中断存放的地方。

`DR6` (`debug status register`) 指示调试程序时异常发生的原因。当调试异常发生时，`DR6` 的有关位自动置 1。为避免在识别各种调试异常时的混乱，调试服务程序返回前应复位 `DR6`。

- `B3`～`B0` (`breakpoint condition detected flags`)：断点异常发生指示位。
  - 当 `Bi`（`i`＝0～3）为 1 时，表示对应断点的异常已经发生。
- `BD` (`debug register access detected flag`)：调试寄存器处理检测位。当 `BD`＝1 时，表明下一条指令将读/写调试寄存器。
- `BS` (`single step flag`)：单步异常标志位。当 `BS`＝1 时，表示异常是由标志寄存器中 `TF`＝1 时单步自陷引起的。
- `BT` (`task switch flag`)：任务转换标志位。当 `BT`＝1 时，表示因为转换而发生异常。

这些调试寄存器给 `80486` 微处理器带来了先进的调试功能，如设置数据断点、代码断点（包括 `ROM` 断点）和对任务转换进行调试。

`DR7` (`debug control register`)：调试控制寄存器，用于指示中断发生的条件及断点的类型。

- `L3`～`L0` (`local breakpoint enable flags`)：局部断点使能标志位。
  - `Li`（`i`＝0～3）设置为 1 时，表示 `i` 号断点局部允许使用，断点仅在某一任务内发生，`Li` 位在任务转换时清 0。若要使某个断点在某个任务中有效，则该任务在 `TSS` 中的 `T` 位应置为 1。此后，在任务转换取得 CPU 控制权时发生异常，则可在其处理程序中将 `Li` 位置为 1，即能保证该断点在此任务内有效。
- `G3`～`G0` (`global breakpoint enable flags`)：全局断点使能标志位。
  - `Gi`（`i`＝0～3）设置为 1 时，表示 `i` 号断点全局允许使用，无论是操作系统还是某一任务，只要满足条件便会产生中断。
- `LE` 和 `GE` (`local and global exact breakpoint enable flags`)：局部断点、全局断点类型标志位。
  - 当 `LE` 和 `GE` 为 1，表示全局断点或局部断点为精明断点。精明断点为立即报告的断点。非精明断点为可以隔若干条指令后再报告或不报告的断点。
- `GD` (`general detect enable flag`)：调试寄存器保护标志位。
  - 当 `GD` 为 1 时，调试寄存器处于保护状态，并产生中断。
- `R/W3`～`R/W0` (`read/write fields`)：发生中断时系统读/写标志位。
  - `R/W3`～`R/W0` 分别指示当 `L3`～`L0` 局部断点和 `G3`～`G0` 全局断点发生时，系统在进行何种操作。
- `LEN3`～`LEN0` (`length fields`)：断点地址开始存放的数据长度。
  - `LEN3`～`LEN0` 分别指示断点地址寄存器 `DR3`～`DR0` 在存储器中存放的情况。

## 二. 利用中断向量还原 IDT hook

现在一些键盘嗅探器和驱动保护等都喜欢在 `IDT` 上动手脚，在他们启动时都接管中断向量或者对中断处理函数进行 `hook`。以下介绍一下如何用 `IDT hook` 进行 `anti-rootkit` 处理。此方法的好处是什么呢，我想最大的好处就是某些猥琐程序对 `IDT` 动手脚后，在它下一条指令运行前，我们已经把 `IDT` 还原了。

我们首先要处理的是调试异常 `int 1`。

从上面的介绍可以知道，如果我们在调试地址寄存器中放入我们要监视的内核地址时，所有对这个内核地址的访问操作都会触发 `int 1` 异常，这时就需要实现我们自己的 `int 1` 异常处理程序，但触发情况是我们感兴趣的条件时，就进入我们的处理程序，但不是我们感兴趣的情况时，则交由原中断函数进行处理。例如我们要保护 `int 60` 中断函数的处理地址，可以将中断处理函数的地址放到中断地址寄存器中，并修改 `DR7` 的相关标志位使得任何对其的读写操作都触发一个异常，然后在我们的 `int 1` 处理程序中进行处理。为了简化代码，下面只对一个 CPU 的情况进行处理，如果是多 CPU 的话，可以读取内核变量 `KeNumberProcessors`，然后对所有的 CPU 都处理一遍。

在处理之前我们先要获得原来的中断向量处理地址

```c
void GetOldIdtHandler()
{
  ULONG idtbase = 0;
  UCHAR idtr[8];
  __asm
  {
    sidt idtr
    lea ebx,idtr
    mov ecx,dword ptr[ebx+2]
    /////save INT1
    add ecx,8
    mov ebx,0
    mov bx,word ptr[ecx+6]
    shl ebx,16
    mov bx,word ptr[ecx]
    mov g_trap1_addr,ebx
  }
}
```

然后编写我们自定义的 `int 1` 处理函数，在此过程中，必须注意的是保证堆栈平衡，和代码运行在非分页内存中，否则就会出现令人恼火的 `BOSD`。

在中断发生时，CPU 会见 `EFLAGE`、`CS` 和 `EIP` 分别压入堆栈，进入中断处理程序时栈顶结构如下所示：

```
|------|
|  EIP  |  esp
|------|
|  CS   |  esp+4
|------|
| EFLAGS |  esp+8
|------|
```

由此可以建立一个结构用于表示产生异常发生时的堆栈结构

```c
typedef struct _INT_STACK
{
  ULONG SaveEip;
  ULONG SaveCS;
  ULONG SaveEFLAGS;
} INT_STACK, *PINT_STACK;
```

接着写新的 `Trap 01` 处理函数，判断一下产生异常的情况，如果是我们感兴趣的异常，就有自定义的 `int 1` 处理函数进行处理，如果是其他情况引发的异常，就交给原中断处理函数处理。

```c
#pragma LOCKEDCODE
void __declspec(naked) NewTrap01()
{
  _asm
  {
    pushfd
    pushad
    mov ebx, esp
    add ebx, 36
    push ebx
    call Rootkit_Trap1
    cmp eax, 0
    je  OrgHandle
    popad
    popfd
    iretd
OrgHandle:
    push 0
    mov word ptr [esp+2], 0
    jmp g_trap1_addr+9
  }
}
```

下面就是自定义的处理函数。在我们将要保护的地址放入中断寄存器的时候，就可以监视所有对其进行操作的一举一动了。但是，如果别人的程序对中断寄存器清 0 怎么办。这时可以借助 `DR7` 的 `GD` 位来解决，此标志位全称为：`General Detect Enable`，当设置此位时，如果 CPU 检测到有修改 `Drx` 寄存器的指令时，就会执行这条指令前先产生一个调试异常，我们就可以对这些情况进行处理。一般对调试寄存器的操作都是 3 字节指令，为了保护 `Drx`，可以简单的将堆栈中的中断返回地址+3，跳过这些指令。当然最好的方法就是用智能的反汇编引擎分析指令的操作数再做相应处理。因为在进入中断处理程序的时候，CPU 会自动把 `GD` 位清除，使得中断处理程序可以操作 `DRx`，所以在离开中断处理程序的时候，还必需手动重设 `GD` 位，继续保护 `DRx` 寄存器。

```c
ULONG Rootkit_Trap1(PINT_STACK pStack)
{
  unsigned long eip;
  DR6 dr6;
  dr6 = GetDR6();
  // CPU 发现要修改 DRx 的指令，这里简单的跳过此指令，并清除 DB 位
  if (dr6.BD == 1)
  {
    // 跳过对 DRx 操作的指令
    pStack->SaveEip = eip + 3;
    // 清除 DB 位
    _asm
    {
      mov eax, dr6
      and eax, 0xFFFFDFFF
      mov dr6, eax
      // 恢复 GD 位
      mov eax, dr7
      or  eax, 0x2000
      mov dr7, eax
    }
    return 1;
  }
  // DR0 处的断点被触发
  if (dr6.B0 == 1)
  {
    _asm
    {
      cli
      mov eax, cr0
      mov g_cr0, eax
      and eax, 0fffeffffh
      mov cr0, eax
      // 先屏蔽 DR0 中断
      mov eax, dr7
      and eax, 0xFFFFFFFC
      mov dr7, eax
      // 在这里我们就可以恢复 IDT hook
      lea esi, g_Org_Trap0Byte
      mov edi, g_trap0_addr
      mov ecx, 5
      cld
      rep movsb
      // 开启内存写保护
      mov eax, g_cr0
      mov cr0, eax
      // 恢复 GD 位
      mov eax, dr7
      or  eax, 0x2000
      mov dr7, eax
      sti
    }
    return 1;
  }
  // 恢复 GD 位
  _asm
  {
    mov eax, dr7
    or  eax, 0x2000
    mov dr7, eax
  }
  return 0;
}
```

## 三. 利用 IDT hook 实现代码 stealth inlinehook

有时候我们的程序正是邪恶程序那如何处理…………

通过上面的代码我们发现，既然中断向量的返回地址已经压入堆栈中，我们不就可以通过改变这个返回地址实现指令流程的改变了吗。

`inline hook` 是我们通常用来改变程序流程的一种方法，其实现需要改变原指令，将其 `JMP XXXX` 到自己的处理流程中，但这种方法非常容易被检测。而通过中断向量，我们可以用一种不改变原指令的方式实现更加隐蔽的 `inline hook`。

首先，我们要把打算 `inline hook` 的地址放到中断寄存器中(`Dr0`~`Dr3`，~~~这就是通常意义上的猥琐断点![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")~~~)，并改变 `Dr7` 标志位使得这些这个地址的指令时触发 `int 1` 调试异常。这样我们就可以在自定义的 `int 1` 异常中改变堆栈中的中断返回地址，实现控制程序流程的转变。

这个时候我们需要改变一下 `Rootkit_Trap1` 这个处理函数

```c
ULONG Rootkit_Trap1(PINT_STACK pStack)
{
  unsigned long eip;
  DR6 dr6;
  dr6 = GetDR6();
  // 获得中断时的 EIP
  eip = pStack->SaveEip;
  // CPU 发现要修改 DRx 的指令，这里简单的跳过此指令，并清除 DB 位
  if (dr6.BD == 1)
  {
    // 跳过对 DRx 操作的指令
    pStack->SaveEip = eip + 3;
    // 清除 DB 位
    _asm
    {
      mov eax, dr6
      and eax, 0xFFFFDFFF
      mov dr6, eax
      // 恢复 GD 位
      mov eax, dr7
      or  eax, 0x2000
      mov dr7, eax
    }
    return 1;
  }
  // DR0 处的断点被触发
  if (dr6.B0 == 1)
  {
    /////////// 恢复 inline hook idt //////////////
    _asm
    {
      cli
      mov eax, cr0
      mov g_cr0, eax
      and eax, 0fffeffffh
      mov cr0, eax
      // 先屏蔽 DR0 中断
      mov eax, dr7
      and eax, 0xFFFFFFFC
      mov dr7, eax
    }
    // 劫持中断返回指令地址，指向我们的中继函数
    pStack->SaveEip = (unsigned long)Hijack;
    // 开启内存写保护
    _asm
    {
      mov eax, g_cr0
      mov cr0, eax
      // 恢复 GD 位
      mov eax, dr7
      or  eax, 0x2000
      mov dr7, eax
      sti
    }
    return 1;
  }
  // 恢复 GD 位
  _asm
  {
    mov eax, dr7
    or  eax, 0x2000
    mov dr7, eax
  }
  return 0;
}
```



```
// 中继函数  

void __declspec(naked) Hijack()  

{  

  _asm  

  {  

    pushf  

    pushad  

    call FakeFunc  

    popad  

    popf  

    jmp g_HookEip // 调回原函数继续执行  

  }  

}  

// 自定义函数，用于实现任意不为人知的猥琐功能  

void FakeFunc()  

{  

    …………  

}
```

这个方法的缺陷就是中断寄存器太少了，只有 4 个，这样只能控制 4 个地址。但其隐蔽性比一般的 `SSDT hook` 和 `inline hook` 高一点。

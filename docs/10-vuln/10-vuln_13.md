
# 绕过 SEHOP 安全机制

> 原文：[`www.pediy.com/kssd/pediy12/104707.html`](https://www.pediy.com/kssd/pediy12/104707.html)

作者：Stéfan Le Berre（s.leberre@sysdream.com）  
Damien Cauquil（d.cauquil@sysdream.com）  
译者：riusksk (泉哥：http://riusksk.blogbus.com)  
时间：2010 年 1 月 7 日  

## 目录

0. 前言  
1. SEHOP 简介  
2. 溢出攻击中的 SEHOP 对抗策略  
   2.1 经典的溢出方法  
   2.2 微妙之处  
3. Proof Of Concept  
   3.1 目标程序与限制条件  
   3.2 程序崩溃与利用  
4. 结论  
5. 鸣谢  
6. 参考资料

## 0. 前言

最近微软在一些 windows 版本中加入了一种新的安全机制《Structured Exception Handling Overwrite Protection》[1 & 2]，这些系统有：

Microsoft Windows 2008 SP0  
Microsoft Windows Vista SP1  
Microsoft Windows 7  

目前我们尚未发现任何针对这一安全机制而采取的攻击，但已有少量文章已描述了它的特性及健壮性。的确，SEHOP 似乎相当可靠，以致 Microsoft 发布了一款补丁，以使所有程序在缺省状态下启用这一安全机制。这是否会成为堆栈溢出攻击的未日呢？还没有，详细情况将在后面叙述。

## 1. SEHOP 简介

SEHOP 作为 Structured Exception Handling 的扩展，用于针对程序中使用的 SHE 结构进行一些安全检测。SEHOP 的核心特性是用于检测程序栈中的所有 SHE 结构链表，特别是最后一个 SHE 结构，它拥有一个特殊的异常处理函数指针，指向一个位于 ntdll 中的函数。下面就是一个经典的 SHE 链表：

![](img/3390a265cab3944c703705e9f3265261.jpg)

每一 SHE 结构均拥有指向下一个 SHE 结构的指针，后面再连接一个指向 `ntdll!_except_handler4` 的异常处理函数指针。在通过覆盖 SHE 结构的栈溢出中，next SHE 指针被一些字节码覆盖，而 SHE handler 被一个指向 `pop pop ret` 指令串的地址覆盖掉，该指令串位于一个 non-SafeSEH 模块中。

这种使用 SEHOP 的验证算法首次被 A. Sotirov 在 Black Hat 2008 大会上批露［3］。下面我们来看一下：

```
BOOL RtlIsValidHandler(handler)
{
  if (handler is in an image) {
    if (image has the IMAGE_DLLCHARACTERISTICS_NO_SEH flag set)
      return FALSE;
    if (image has a SafeSEH table)
    if (handler found in the table)
      return TRUE;
    else
      return FALSE;
    if (image is a .NET assembly with the ILonly flag set)
      return FALSE;
    // fall through
  }
  if (handler is on a non-executable page) {
    if (ExecuteDispatchEnable bit set in the process flags)
      return TRUE;
    else
    // enforce DEP even if we have no hardware NX
    raise ACCESS_VIOLATION;
  }
  if (handler is not in an image) {
    if (ImageDispatchEnable bit set in the process flags)
      return TRUE;
    else
      return FALSE; // don't allow handlers outside of images
  }
// everything else is allowed
return TRUE;
}
```

```
// Skip the chain validation if the
DisableExceptionChainValidation bit is set
if (process_flags & 0x40 == 0) {
  // Skip the validation if there are no SEH records on the
  // linked list
  if (record != 0xFFFFFFFF) {
    // Walk the SEH linked list
    do {
      // The record must be on the stack
      if (record < stack_bottom || record > stack_top)
        goto corruption;
      // The end of the record must be on the stack
      if ((char*)record + sizeof(EXCEPTION_REGISTRATION) > stack_top)
        goto corruption;
      // The record must be 4 byte aligned
      if ((record & 3) != 0)
        goto corruption;
      handler = record->handler;
      // The handler must not be on the stack
      if (handler >= stack_bottom && handler < stack_top)
        goto corruption;
      record = record->next;
    } while (record != 0xFFFFFFFF);
    // End of chain reached
    // Is bit 9 set in the TEB->SameTebFlags field?
    // This bit is set in ntdll!RtlInitializeExceptionChain,
    // which registers FinalExceptionHandler as an SEH handler
    // when a new thread starts.
    if ((TEB->word_at_offset_0xFCA & 0x200) != 0) {
      // The final handler must be ntdll!FinalExceptionHandler
      if (handler != &FinalExceptionHandler)
        goto corruption;
    }
  }
}
```

下面我们来考虑一些限制：

SHE handler 必须指向 non-SafeSEH module  
内存页必须可执行  
SHE 链表不应被篡改，并且位于链表末端的 SHE 结构必须为特定值（next SHE 指针值为 `0xFFFFFFFF`，SHE handler 也必须为一特定值）  
所有的 SHE 结构必须为 4-byte 对齐  
最后一个 SHE 结构的 handler 必须正确指向 ntdll 中的 `ntdll!FinalExceptionHandler` 例程  
所有 SHE 指针必须指向栈中

## 2. 溢出攻击中的 SEHOP 对抗策略

### 2.1 经典的溢出方法

下面是覆盖 SHE 结构的经典溢出方法的结构图：

![](img/958be994ef1cc5343e71fc051a2d9cfd.jpg)

SHE handler 指向 `POP POP RET` 指令串，next SHE 中的栈地址被 `jmp 06 nop nop` (机器码：`EB 06 90 90`)替换掉。当处理一个程序异常时，Windows 将控制权传给 SHE 链表中的异常处理程序。当第一个异常处理函数被覆盖掉后，程序的执行流程重定向到 `POP POP RET` 指令串中（代替真实的异常处理函数）。该指令串将执行到当前 SHE 结构中 next SHE 的前两个字节（jmp 06），以此直接跳转到我们的 shellcode。这样，SHE 链表就被拆分了：

![](img/bcc0c17e8d80489a953d5e8987fc50d5.jpg)

SHE 结构中的 next SEH 的前两字节（`90 90`）从不被执行，它主要是用于填充字符，以构造出一个 DWORD 大小的有效栈地址。当然，我们也可以改变另外的两字节来使其指向一个有效栈地址，而将这两字节改为跳转指令（`EB 06`）。这一 DWORD 值当被调用时，它相当于一个有效的栈地址与一个有效指令的结合以实现跳转。如果我们伪造 SHE 结构中的这一 DWORD 值指向的栈地址，那么我们就可伪造整个 SHE 链表，并控制下一个 SHE 结构以使此链表有效。

### 2.2 微妙之处

还有一个问题：哪一被编码的跳转指令可以代替 4 字节对齐地址？只有一指令与其相配：`JE`（机器码为 `0x74`），该指令是一条件跳转：只有当 Z 标志被设置时才实现跳转。在 Windows 在处理异常时，Z 标志是默认关闭的，我们必须通过执行一指令来计算出零值，以设置该标志位。Xor 指令似乎是不错的选择，看来我们的解决方法已经很明了了：跳至 `XOR,POP,POP,RET` 指令串，以便将 `JE` 指令解释为 `JMP`。这就是 SEHOP 绕过技术的微妙之处。

`XOR，POP，POP，RET` 指令串并不难找到，我们可以通过查找返回 NULL 值的函数的末端来找到这些指令串：

```
XOR EAX, EAX
POP ESI
POP EBP
RET
```

为了确保该方案可行，我们就以 `XOR，POP，POP，RET` 指令串来编写一份 PoC 测试一下。

已知第一个 SEH 结构位于 `0x0022FD48`，我们就可以在 `0x0022FD74` 处构造第二个 SHE 结构，它的 next SHE 指针值为 `0xFFFFFFFF`，handler 指向 `ntdll!FinalExceptHandler`。如果我们将第一个 SHE 结构中的 handler 指向 `XOR,POP,POP,RET` 指令串，那么我们就可以将执行流程重定向到任何地方，真正地破坏掉 SHE 链表。具体地，我们可以重新伪造一个有效的 SHE 链表，以使其重定向到特定的 shellcode。

另一个主要限制是 Microsoft Windows 7 和 vista 中的 ASLR 安全机制。我们知道程序的利用需要依赖于 `ntdll!FinalExceptHandler` 的地址，但这一地址在每次电脑重启后都会发生改变。Ntdll ImageBase 在重启时都被随机化，这样就增加攻击的难度。我们针对 ASLR（未经逆向分析）进行一些测试，结果表明 ImageBase 似乎在 16 位中只有 9 位发生随机化，也就是说，在超过 512 次机会中你只有一次机会可以成功地绕过 SEHOP 实现溢出攻击。

## 3. Proof Of Concept

### 3.1 目标程序与限制条件

我们编写一段小程序，以在 Windows 7 下演示这一技术。这一程序只是复制一个文件（`OwnMe.txt`）的内容到内存中，并在操作中实现栈溢出，以引发可被异常处理程序捕获的异常。

我们需要完成以下任务：

伪造一个有效的 SEH 链表  
将最后一个 SEH 结构中的 handler 指向 `ntdll!FinalExceptHandler`  
在栈中放置 shellcode(与 Windows 7 相兼容)  
定位内存中的 `XOR,POP,POP,RET` 指令串地址  

首先，`XOR,POP,POP,RET` 指令串是已知的，因为我们已经将其放置在程序代码中了，我们可以发现该指令串位于内存地址 `0x004018E1` 中。

### 3.2 程序崩溃与利用

当程序崩溃时，可发现：

```
0012F700 41414141 Pointer to next SEH record
0012F704 41414141 SE handler
```

接着我们需要利用存储在 `0x0012F774` 中的第二个 SEH 结构来伪造 SEH 链表，这一 SEH 结构中 next SEH 指针值为 `0xFFFFFFFF`，SHE handler 指向 `ntdll!FinalExceptHandler`。然后位于 `0x0012F700` 的 SEH 结构被篡改，使其指向仿造的 SHE 结构，并将 handler 设置为 `0x004018E1`，同时在每一 SHE 结构之前写入一跳转指令（`JMP+8`），以避免数据段被执行。整个利用的工作流程如下图所示：

![](img/7cc603b706c79b3247246324b825bcf7.jpg)

Shellcode 将执行 `calc.exe`：

![](img/90918c264781327af1621b7ca6f2fd65.jpg)

注意：目标程序及 exploit 均已打包成 zip 文件，可在 Sysdream 网站上［4］下载到。

## 4. 结论

SEHOP 并不是对抗栈溢出的终极防护。自从 SafeSEH 扩展被公布之后，很多人认为它是不可突破的。这里我们只是演示了在一些情况下 Structured Exception Handling Overwrite Protection 是可能绕过的。但 SEHOP 结合 ASLR 和 DEP 使用，是一种优秀的本地安全机制，这点是不可否认的。

## 5. 鸣谢

http://sysdream.com/  
http://ghostsinthestack.org/  
http://virtualabs.fr/

## 6. 参考资料

[1] Preventing the Exploitation of Structured Exception Handler (SEH) Overwrites with SEHOP:  
http://blogs.technet.com/srd/archive/2009/02/02/preventing-the-exploitation-of-seh-overwriteswith-  
sehop.aspx

[2] SEHOP per-process opt-in support in Windows 7:  
http://blogs.technet.com/srd/archive/2009/11/20/sehop-per-process-opt-in-support-in-windows-  
7.aspx

[3] Bypassing Browser Memory Protections: http://taossa.com/archive/bh08sotirovdowd.pdf

[4] ZIP containing our target program & exploit http://www.sysdream.com/SEHOP.zip


# 浅谈一些寄存器在实际中的应用

> 原文：[`www.pediy.com/kssd/pediy12/142103.html`](https://www.pediy.com/kssd/pediy12/142103.html)

本人新手，只是把一些问题放到一块来看，如有错误之处请指正，文中引用各位前辈的一些文章，详细请阅读原文。

Windows 中将线程环境成为“thread context”，实际上指的是寄存器的状态，用一个 `CONTEXT` 结构来表示。

![](img/2f2d1c366ed0d01a73eec41a416f24f9.jpg)![](img/43c4f5a7477f023c847fd96a59f63e90.jpg)

![](img/566e6d93d627ce1128c43729ce048390.jpg)![](img/653b43f7d6b716b376d2a7dcda8c0acb.jpg)

其中 `Esp` 指向堆栈的栈顶，`Eip` 指向下一条指令。

## 程序调用与堆栈

在程序调用函数过程中堆栈起到关键作用，当参数多余一个是要按照一定的顺序压栈。局部变量也要用到堆栈。（详细参照[《**解密基础知识入门**》作者: kanxue](http://bbs.pediy.com/showthread.php?t=31840)，中关于**调用约定**和**局部变量**部分）

因为 `call` 之前要把返回的地址压栈，我们可以这样做以达到自定位的目的：

```
dwVar dd ?
call @F
@@:
pop ebx
sub ebx,offset @B
mov eax,[ebx + offset dwVar]
```

我们也可以用此来动态获取 API 入口地址：当父进程创建子进程时，会调用 `kernel32.dll` 中的 `GreateProcess`,`CreateProcess` 函数在完成装载应用程序后，会将一个返回地址压栈并转而执行应用程序，这个地址在程序入口的地方用`[Esp]`就可以读出，该地址位于 `kernel32.dll` 中，通过它我们可以得到 `LoadLibrary` 和 `GetProcAddress` 函数的地址……（这两个用处在《win32 汇编语言程序设计》中有详细说明）。

## 加壳与 OEP 定位

在加壳过程中，外壳初始化的现场环境（各寄存器值）与原程序的现场环境是相同的。加壳程序初始化时保存各寄存器的值，外壳执行完毕，会恢复各寄存器内容。其代码形式一般如下：

```
PUSHFD  ; 将标志寄存器入栈保存
PUSHAD  ; push eax, ecx, edx, ebx, esp, ebp, esi, edi
…… ; 外壳代码部分
POPAD  ; pop edi, esi, ebp, esp, ebx, edx, ecx, eax
POPFD  ; 恢复标志寄存器
JMP OEP
OEP: …… ; 解压后的程序原代码
```

（该部分来自[**脱壳基础知识入门及 FAQ**](http://bbs.pediy.com/showthread.php?t=20366)作者: kanxue）

这种情况我们可以用 `Esp` 定律来找 `OEP`：

![](img/1b8964d1def6e59eb3378724b04ce444.jpg)

![](img/d678cdea6fe28377a2c57b574e9dbae0.jpg)

![](img/623ba642e5ba09564006ac2e353f7d56.jpg)

这个堆栈平衡原理其找 OEP 原理这篇文档描述的比较详细：[**寻找真正的入口（OEP）--广义 ESP 定律**](http://www.pediy.com/bbshtml/BBS6/pediy6083.htm) 作者:Lenus

![](img/2f2d1c366ed0d01a73eec41a416f24f9.jpg)

## 硬件断点原理

在寄存器中，有这么一些寄存器，它们用于调试。人们把他们称为调试寄存器，调试寄存器一共有 8 个名字分别从 `Dr0-Dr7`。所以我们也把调试寄存器简单的称为 `Drx`。

对于 `Dr0-Dr3` 的四个调试寄存器，他们的作用是存放中断的地址，例如：`401000`

对于 `Dr4`，`Dr5` 这两个寄存器我们一般不使用他们，保留

对于 `Dr6`，`Dr7` 这两个寄存器的作用是用来记录你在 `Dr0-Dr3` 中下断的地址的属性，比如：对这个 `401000` 是硬件读还是写，或者是执行；是对字节还是对字，或者是双字。

（这段来自 [**硬件断点的原理**](http://bbs.pediy.com/showthread.php?threadid=10829) 作者:Lenus）

## SEH 技术与 TIB

在《脱壳基础知识入门及 FAQ》第二课是 SEH 技术。SEH 是基于线程的，使用 SEH 可以为每个线程设置不同的异常处理程序，而且可以为每个线程设置多个异常处理程序。

Win32 为每个线程定义了一个线程信息块，其中保存了线程的一些属性数据，线程信息块的格式被定义为 `NT_TIB` 结构：

```
NT_TIB STRUCT
ExceptionList dd ? ; SEH 链入口
StackBase dd ? ; 堆栈基址
StackLimit dd ? ; 堆栈大小
SubSystemTib dd ?
FiberData dd ?
ArbitraryUserPointer dd ?
Self dd ? ; 本 NT_TIB 结构自身的线性地址
NT_TIB ENDS
```

`NT_TIB` 结构的第一个字段 `ExceptionList` 指向一个 `EXCEPTION_REGISTRATION` 结构，SEH 异常处理回调函数的入口地址就是由 `EXCEPTION_REGISTRATION` 结构指定的，这个结构的定义如下：

```
EXCEPTION_REGISTRATION STRUCT
prev dd ? ; 前一个 EXCEPTION_REGISTRATION 结构的地址
handler dd ? ; 异常处理回调函数地址
EXCEPTION_REGISTRATION ENDS
```

当异常发生时，系统从 TIB 中取出 `ExceptionList` 字段，然后从 `ExceptionList` 字段指定的 `EXCEPTION_REGISTRATION` 结构中取出 `handler` 字段，并根据其中的地址去调用回调函数，整个过程如图 14.1 所示，所以只要构建一个含有回调函数地址的 `EXCEPTION_REGISTRATION` 结构，然后修改 TIB 中的 `ExceptionList` 字段，指向这个结构就可以注册一个 SEH 异常处理回调函数。

![](img/4d7747698f73d5c1ec2da7f0c88cbf87.jpg)

SEH 异常处理程序入口地址的定义

现在还剩下一个关键的问题：到哪里找 TIB 呢？答案是：TIB 永远放在 `fs` 段选择器指定的数据段的 0 偏移处，所以，`fs:[0]` 的地方就是 TIB 结构的 `ExceptionList` 字段，这个答案对于 Windows 9x 系统和 Windows NT 系统都是有效的。由于一个进程中的不同线程可以有不同的环境，所以，在不同线程中 `fs` 段选择器可以使用不同的值，这种特征使每个线程都可以设置不同的回调函数。

（原文来自《Windows32 位汇编语言程序设计教程》）

参照这里可以有更多的了解：

[**SEH IN ASM 研究(二)---提高篇:关于异常处理的嵌套和堆栈展开 (15 千字)**](http://www.pediy.com/kssd/pediy04/kanxue370.htm)作者:hume

[**【转帖】好文 Structured Exception Handling**](http://bbs.pediy.com/showthread.php?threadid=14042)


### Say hello to x86\_64 Assembly [part 1]

最近在学习x64汇编,在github上面找到了一点学习资料,入门级别的,因为想细致的学习一下,所以顺便久把作者的内容都翻译了一下,也不知道自己翻译的是否合适,请大家看看有问题的地方请批评指正.第一次做翻译,做的不好请大家原谅,

[作者原文](https://bbs.kanxue.com/elink@a8dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1j5H3P5p5q4j5i4K6u0r3j5i4y4E0)

---

## 介绍

Introduction

我们中间有很多开发者，我们每天写了成千上万行代码。有时候甚至是一些不错的代码：）我们每个人都能写出像下面这样的简单代码：

There are many developers between us. We write a tons of code every day. Sometime, it is even not a bad code :) Every of us can easily write the simplest code like this:

```
#include <stdio.h>

int main() {
  int x = 10;
  int y = 100;
  printf("x + y = %d", x + y);
  return 0;
}
```

我们每个人都能明白这个c代码做了什么，但...这些代码在底层如何工作的？我相信不是我们所有人都能够回答这个问题，包括我。我认为我能写高级语言的代码，像Haskell, Erlang, Go 等等...,但我绝对不知道在汇编之后他们是如何在底层工作的。所以我决定去进一步深入了解汇编并描绘我的学习方式。希望他会是个很有趣的事情，不仅仅是对我。大概5-6年前，我已经开始使用汇编编写一些简单的程序了，那是我在大学里面，使用Turbo Assembly 和DOS系统。现在我使用Linux-x86-64 系统。是的，Linux 64位和DOS 16位系统有很大的区别。让我们开始吧。

Every of us can understand what’s this C code does. But… How this code works at low level? I think that not all of us can answer on this question, and me too. I thought that i can write code on high level programming languages like Haskell, Erlang, Go and etc…, but i absolutely don’t know how it works at low level, after compilation. So I decided to take a few deep steps down, to assembly, and to describe my learning way about this. Hope it will be interesting, not only for me. Something about 5 - 6 years ago I already used assembly for writing simple programs, it was in university and i used Turbo assembly and DOS operating system. Now I use Linux-x86-64 operating system. Yes, must be big difference between Linux 64 bit and DOS 16 bit. So let’s start.

## 准备

Preparation

在我们开始前，我必须准备一些东西，例如我使用的Ubuntu(Ubuntu 14.04.1 LTS 64 bit)，因此我写的文章将针对这个操作系统和架构。不同的CPU支持不同的指令集，我使用的是 Intel Core i7 870 处理器，所有的代码都将由这个处理器所编写。我将使用nasm汇编，你可以使用下面命令安装：

Before we started, we must to prepare some things like As I wrote about, I use Ubuntu (Ubuntu 14.04.1 LTS 64 bit), thus my posts will be for this operating system and architecture. Different CPU supports different set of instructions. I use Intel Core i7 870 processor, and all code will be written processor. Also i will use nasm assembly. You can install it with:

```
$ sudo apt-get install nasm
```

它的版本必须是2.0.0或者更高版本。我2013年12月29日编译的NASM版本是2.10.09。最后一部分，需要在文本编辑器中编写汇编代码。我使用Emacs配合nasm-mode.el来进行编写。当然你可以选择你喜欢的编辑器。如果你和我一样使用Emacs，你可以下载nasm-mode.el 并配置你的Emacs像这样：

It’s version must be 2.0.0 or greater. I use NASM version 2.10.09 compiled on Dec 29 2013 version. And the last part, you will need in text editor where you will write you assembly code. I use Emacs with nasm-mode.el for this. It is not mandatory, of course you can use your favourite text editor. If you use Emacs as me you can download nasm-mode.el and configure your Emacs like this:

```
(load "~/.emacs.d/lisp/nasm.el")
(require 'nasm-mode)
(add-to-list 'auto-mode-alist '("\\.\\(asm\\|s\\)$" . nasm-mode))
```

这就是我们现在所需要的。其他工具将在下一篇文章中描述。

That’s all we need for this moment. Other tools will be describe in next posts.

## nasm汇编的语法

Syntax of nasm assembly

在这里，我不会描述完整的汇编语法，我们将只提到我们在本文中使用到的部分语法。通常NASM程序分为几个部分。在本文中，我们将讨论以下两个部分：

Here I will not describe full assembly syntax, we’ll mention only those parts of the syntax, which we will use in this post. Usually NASM program divided into sections. In this post we’ll meet 2 following sections:

- 数据段
- 文本段
- data section
- text section

数据段用于声明常量。此数据在运行时不会更改。您可以声明各种数学或其他常量等。声明数据段的语法为：

The data section is used for declaring constants. This data does not change at runtime. You can declare various math or other constants and etc… The syntax for declaring data section is:

```
section .data
```

文本部分用于代码。本节必须以声明global-start开始，它告诉内核程序执行的开始位置。

The text section is for code. This section must begin with the declaration global \_start, which tells the kernel where the program execution begins.

```
section .text
global _start
_start:
```

注释以；符号开头。每个NASM源代码行都包含以下四个字段的某些组合：

Comments starts with the ; symbol. Every NASM source code line contains some combination of the following four fields:

```
[label:] instruction [operands] [; comment]
```

方括号中的字段是可选的。基本的NASM指令由两部分组成。第一个是要执行的指令的名称，第二个是这个命令的操作数。例如：

Fields which are in square brackets are optional. A basic NASM instruction consists from two parts. The first one is the name of the instruction which is to be executed, and the second are the operands of this command. For example:

```
MOV COUNT, 48 ; Put value 48 in the COUNT variable
```

## Hello World

Hello world

让我们用NASM汇编编写第一个程序。当然，这将是传统的Hello world程序。这是它的代码：

Let’s write first program with NASM assembly. And of course it will be traditional Hello world program. Here is the code of it:

```
section .data
    msg db      "hello, world!"

section .text
    global _start
_start:
    mov     rax, 1
    mov     rdi, 1
    mov     rsi, msg
    mov     rdx, 13
    syscall
    mov    rax, 60
    mov    rdi, 0
    syscall
```

是的，它看起来不像`printf("Hello world")`。让我们试着了解它是什么以及它是如何工作的。看1-2行。我们定义了`data section`并将`msg`常量与Hello world值放在一起。现在我们可以在代码中使用这个常量。接下来是声明文本段和程序入口点。程序将从7行开始执行。现在开始最有趣的部分。我们已经知道它是`mov`指令了，它有两个操作数，然后把值从第二个放到第一个。但这些`rax`，`rdi`...是什么,我们可以在维基百科上看到:

Yes, it doesn’t look like `printf("Hello world")`. Let’s try to understand what is it and how it works. Take a look 1-2 lines. We defined `data section` and put there `msg` constant with Hello world value. Now we can use this constant in our code. Next is declaration text section and entry point of program. Program will start to execute from 7 line. Now starts the most interesting part. We already know what is it `mov` instruction, it gets 2 operands and put value of second to first. But what is it these `rax`, `rdi` and etc… As we can read in the wikipedia:

```
中央处理器（CPU）是计算机内部的硬件，它通过执行系统的基本算术、逻辑和输入/输出操作来执行计算机程序的指令。
```

```
A central processing unit (CPU) is the hardware within a computer that carries out the instructions of a computer program by performing the basic arithmetical, logical, and input/output operations of the system.
```

好的，CPU执行一些操作，算术等... 但是它从哪里可以得到这些操作的数据呢？第一个答案就是内存。然而，读取数据并将数据存储到内存中会减慢处理器的速度，因为它涉及通过控制总线发送数据请求的复杂过程。因此，CPU有自己的内部内存存储位置，称为寄存器：

所以我们知道`sys_write` syscall有三个参数，在`syscall`表中是第一个。让我们再来看看我们的hello world实现。我们将1放入`rax`寄存器，这意味着我们将使用`sys_write`系统调用。在下一行中，我们将1放入`rdi`寄存器，它将是`sys_write`的第一个参数，1-标准输出。然后我们将指向`msg`的指针存储在`rsi`寄存器中，它将是`sys_write`的第二个`buf`参数。然后我们将最后一个（第三个）参数（字符串长度）传递给`rdx`，它将是`sys_write`的第三个参数。现在我们有了`sys_write`的所有参数，可以用`syscall`函数在11行调用它。好的，我们打印了“Hello World”字符串，现在需要正确地退出程序。我们传递60到`RAX`寄存器，60是一个信号退出`syscall`。并将0传递给`rdi`寄存器，它是错误代码，所以0表示我们的程序成功退出。这就是“你好世界”的全部内容。很简单：）现在让我们构建我们的程序。例如，`hello.asm`文件中有此代码。然后我们需要执行以下命令：

So we know that `sys_write` syscall takes three arguments and has number one in syscall table. Let’s look again to our hello world implementation. We put 1 to `rax` register, it means that we will use `sys_write` system call. In next line we put 1 to `rdi` register, it will be first argument of `sys_write`, 1 - standard output. Then we store pointer to `msg` at `rsi` register, it will be second buf argument for `sys_write`. And then we pass the last (third) parameter (length of string) to `rdx`, it will be third argument of `sys_write`. Now we have all arguments of the `sys_write` and we can call it with `syscall` function at 11 line. Ok, we printed “Hello world” string, now need to do correctly exit from program. We pass 60 to `rax` register, 60 is a number of exit syscall. And pass also 0 to `rdi` register, it will be error code, so with 0 our program must exit successfully. That’s all for “Hello world”. Quite simple :) Now let’s build our program. For example we have this code in `hello.asm` file. Then we need to execute following commands:

```bash
$ nasm -f elf64 -o hello.o hello.asm
$ ld -o hello hello.o
```

在它之后，我们将拥有可执行的`hello`文件，我们可以使用`./hello`运行它，并将在终端中看到`hello world`字符串。

After it we will have executable `hello` file which we can run with `./hello` and will see Hello world string in the terminal.

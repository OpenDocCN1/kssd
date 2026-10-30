
首先通过调整 <a href="elink@37eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3q4K6N6s2W2Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6t1I4i4K6u0V1x3K6p5#2y4R3`.`.">poc</a> 和调试，能够发现在堆上确实是存在 `service_user` 结构，不过调试发现其和 vuln obj 的偏移过大，直接去尝试覆盖 `service_user` 程序会由于中间的一些数据被覆盖而崩溃，且两者的偏移也不固定。

![image-20211005172559810](upload/attach/202110/690974_3AE2ZGVGYTW56VF.png)

为了解决这个问题，可以利用 `setlocale` 函数中的 **内存分配/释放** 操作进行堆布局，让 `user_args` 落在 `service_user` 的前面。

`setlocale` 函数在解析环境变量（比如 `LC_CTYPE`、`LC_MESSAGES`、`LC_TIME` 等）时，会申请内存并把环境变量的值拷贝到申请的内存中，申请的内存在函数退出前会被释放。

通过利用 `setlocale` 函数的逻辑和堆分配器的机制，我们提前在堆中创建一些 `free` 的堆块，让 `service_user` 分配到其中的一个空闲块，这样在漏洞触发时就能在 `service_user` 前面留一个 `free` 堆块，这时我们将 `user_args` 分配到 `service_user` 前面就可以稳定溢出 `service_user` 了，大概流程图如下所示：

![image-20211005174844655](upload/attach/202110/690974_F88D8F9UVK3EDCQ.png)

### 小结

1. 通过 fuzz 来探测可能用于利用的内存状态值得学习。
2. 通过提前在堆里面凿一些洞（空闲的堆块）进行堆布局的思路非常新奇。

## <a href="elink@c63K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4@1L8%4t1%4i4K6u0W2j5$3!0E0M7s2g2@1k6i4y4@1i4K6u0W2L8X3I4Q4x3V1k6H3L8%4y4@1i4K6u0r3x3U0l9J5x3g2)9J5k6o6l9^5i4K6u0V1P5X3!0G2L8g2)9J5c8R3`.`.">Zoom RCE from Pwn2Own 2021</a>

漏洞是 `Zoom` 在进行密钥协商时存在一个堆溢出，堆溢出的情况如下：

1. vuln obj 的大小固定 1040 字节
2. 溢出的大小和内容可控

漏洞利用的环境是 win10 + 32位的 Zoom 软件。

首先利用堆溢出实现信息泄露，常见的思路是分配一些带 `length` 字段的对象，然后覆盖 `length` 字段从而 leak 出一些数据，经过一番分析没有发现能够回显的这种对象。

最后发现，可以给目标 `Zoom` 客户端发送特定的请求，让对端 `Zoom` 向我们的服务器发起 `https` 请求，请求的 url 部分可控，有种 SSRF 的感觉。

接下来就是利用这个 bug 进行信息泄露，具体来说就是利用溢出把 `URL` 末尾的 `\x00` 覆盖掉，然后让目标客户端向我们的服务的发起请求，就可以泄露 URL 后面的内存数据了。

这里选择泄露的对象是 `TLS1_PRF_PKEY_CTX`，这个对象是 `openssl` 进行 TLS 协商时创建的，对象中存在指向 `DLL` 的指针，通过泄露这个指针可以可以拿到 `DLL` 的基地址。

![image-20211006105537780](upload/attach/202110/690974_96JXBGAWPQND99S.png)

如果堆布局按照预期就能在我们的服务器收到泄露的数据。

![img](upload/attach/202110/690974_T6HATS4QRZT8BD5.png)

不过由于 LFH 分配器的随机化和内存状态的不确定性，成功率还是比较低，作者采取了一些措施来提升成功率：

1. 创建多个服务器的连接，用于不断的进行 TLS 协商，目的是在 LFH 的桶中分配大量 `TLS1_PRF_PKEY_CTX`，这个过程中即使对象被释放也没关系，因为释放之后内存中的指针不会被清理，也能进行 leak，这样就可以增大布局的概率。
2. 在覆盖的过程中，可能由于内存布局的不一致导致 vuln obj 和 url 之间存在一些带指针的对象（比如某些类的实例），作者的处理方式是首先通过堆喷在某个地址（比如 `0x0d0d0d0d`）放置数据，然后用 `0x0d0d0d0d` 覆盖对象，这样即使覆盖到了不正确的对象也不会导致进程崩溃，从而可以多次尝试。

![image-20211006111507328](upload/attach/202110/690974_92Y62WA5DG4H28E.png)

完成信息泄露后，下一步就是找一个对象控制 PC，这一步使用的是 `FileWrapperImpl`，该对象会在客户端被呼叫响铃的时候被不断的创建和删除，因此使用堆溢出覆盖其虚表即可劫持控制流。

![image-20211006112040501](upload/attach/202110/690974_VR5AUQ5UDCCM25H.png)

为了劫持虚表还需要进行堆喷，攻击者可以发起请求让目标客户端加载 `gif` 图片，而且没有大小限制，利用这个机制就可以进行堆喷。

第二部的过程中也会存在很多不确定性，比如溢出 vuln obj 的过程中可能会覆盖到堆中的其他类的虚表，作者的解决方案如下：

1. 不断尝试溢出、调试搜集在利用过程中可能会触发的场景，在虚表、rop 中进行适配，比如 A 类调用的时候可能用的是虚表偏移 `0x20` 的函数指针，我们就在 `0x20` 的函数指针布置适合 A 的利用 rop。
2. 不断溢出，每次溢出的大小递增，避免出现一次溢出太多导致不稳定。

### 小结

1. 利用客户端的 url 请求机制进行 leak 也是一个思路。
2. 处理堆布局随机的情况可以多次尝试，为了能够多次尝试还需要合理控制溢出数据，避免溢出失败导致进程 Crash。
3. 针对不同的溢出场景，在虚表中进行适配思路非常不错！

## <a href="elink@96aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5k6i4y4W2j5i4u0U0K9q4)9J5k6h3y4Z5k6h3y4C8M7r3!0A6L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6t1H3i4K6u0r3K9h3&6K6N6r3q4Y4M7X3q4E0i4K6g2X3M7X3y4W2i4K6u0V1j5$3!0V1k6g2)9J5k6r3g2^5k6h3y4#2N6r3W2G2L8W2)9J5k6s2k6#2L8r3&6W2M7X3q4T1K9h3I4A6N6s2W2Q4x3X3c8A6L8W2)9J5k6r3W2F1M7%4c8S2k6%4u0S2L8g2)9J5k6r3q4H3M7q4)9J5k6r3k6G2M7W2)9J5k6r3q4F1k6s2u0G2K9h3c8Q4x3X3c8S2L8X3c8Q4x3X3c8A6L8%4y4Q4x3V1j5`.">Instagram_RCE: Code Execution Vulnerability in Instagram App for Android and iOS</a>

漏洞是在解析图片时存在整数溢出导致的堆溢出。

![img](upload/attach/202110/690974_MVF38MMU33XZYA8.png)

首先根据 `width`、`height`、`output_components` 计算分配内存的大小，然后从文件中一段一段读入内存，如果 `width * height * output_components` 发生整数溢出，后面从文件中读取内容时就会溢出。

之前提到过利用整数溢出漏洞需要找到终止拷贝的条件，或者在访问到非法地址前劫持控制流，这里采用的是覆盖拷贝过程中会用到的函数指针，在循环访问到非法地址前劫持控制流。

作者通过分析循环中的一些函数调用，发现在 `jpeg_read_scanlines` 函数和子函数里面会使用 `cinfo` 结构体中的一些的函数指针。

```
struct jpeg_decompress_struct {
......
struct jpeg_d_main_controller *main; <<-- there's a function pointer here
struct jpeg_d_coef_controller *coef; <<-- there's a function pointer here
struct jpeg_d_post_controller *post; <<-- there's a function pointer here
......
};
```

target obj 找到了不过由于没有找到合适的内存控制原语，没法进行完美的堆布局，作者最终采取的策略是通过调试和观察漏洞触发前后的堆状态，来搜索可能用于利用的 vuln obj 大小。

首先通过分析知道 `cinfo` 分配在 `0x5000` 的 `run` 中，然后查看 `cinfo` 前面的一些 `run` 中的使用情况，发现可以通过分配 `0xe0` 的内存，然后往后溢出即可覆盖 `cinfo`。

![image-20211005205941293](upload/attach/202110/690974_6ADWTASCWVPAY92.png)

![img](upload/attach/202110/690974_3XVQFAJBMNS9QAR.png)

原文作者到这里就没继续了，这种方式有点撞运气的成分，如果进程在之前有些其他分配就可能会导致偏移不一致，不过原文中也提到了，我们可以溢出多一点总是能够溢出到函数指针的。

### 小结

1. 通过修改循环中的函数指针来防止整数溢出后的 Crash 也是比较经典。
2. 有时候实在没有堆控制原语时，也需要多调试、分析，查看 target obj 和 vuln obj 的附件是否有可以用于利用的内存块。

## <a href="elink@11aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6i4K6u0W2j5X3I4S2j5$3E0Z5j5i4c8Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7$3W2S2i4K6u0V1x3U0m8Q4x3V1k6f1K9s2g2J5M7$3c8S2P5g2)9J5c8X3q4K6K9h3q4Q4x3X3b7J5x3q4)9J5k6q4y4Z5j5h3!0Q4x3X3b7K6c8q4)9J5k6q4u0W2k6q4)9J5k6q4m8A6L8r3I4Q4x3X3c8m8i4K6u0V1c8%4g2W2M7%4c8Q4x3X3c8f1L8#2)9J5k6p5S2G2M7%4c8Q4x3X3c8q4M7$3y4S2M7r3g2Q4x3X3c8a6L8W2)9J5k6q4q4q4e0g2g2w2g2V1#2Q4x3X3c8h3K9i4u0@1K9h3!0Q4x3X3c8p5k6i4k6A6j5$3g2Q4x3X3g2H3k6r3j5`.">3D-Red-Pill-A-Guest-To-Host-Escape-On-QEMUKVM-Virtio-Device</a>

漏洞利用链包含两个漏洞：

1. `malloc` 未初始化导致的信息泄露。
2. `vrend_resource` 堆溢出，vuln obj 的大小可控，溢出的大小内容可控。

利用未初始化漏洞的思路是堆喷带有函数指针的结构体，然后释放这些结构体，最后触发未初始化漏洞并获取未初始化的内存内容，从而获取 `virglrenderer` 库的地址。

![image-20211006131035948](upload/attach/202110/690974_WKEP7SX7VRADHZJ.png)

获取 `libc` 的地址的思路是申请大块内存，内存中就会包含 `libc` 的地址。

![image-20211006131050470](upload/attach/202110/690974_79JHPFH6GXKD9HA.png)

堆溢出的利用思路是堆喷多个 `vrend_resource` 对象，然后利用溢出修改下一个 `vrend_resource` 对象的指针，从而实现任意地址写。

溢出前，两个 `vrend_resource` 对象的布局。

![image-20211006131203472](upload/attach/202110/690974_4XH6XZZQBMSGS2U.png)

溢出后，第二个 `vrend_resource` 对象的指针修改为任意地址，然后利用程序的逻辑就可以实现任意地址写。

![image-20211006131255077](upload/attach/202110/690974_2HCBD9EBVV8W77V.png)

### 小结

1. 利用未初始化漏洞进行内存泄露的思路值得学习，比如堆喷带指针的结构，然后利用未初始化漏洞获取地址。
2. 通过堆喷 vuln obj，然后溢出下一个 vuln obj 的指针来进行任意地址写，得到启示是做利用的时候可以先看 vuln obj 是否就可以作为 target obj。

## <a href="elink@23cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2P5X3g2J5K9h3q4Q4x3X3c8D9j5h3u0K6i4K6u0W2j5$3!0E0i4K6u0r3k6%4u0G2L8$3#2A6L8X3N6Q4x3X3c8@1K9r3g2Q4x3X3c8A6L8%4y4Q4x3X3c8C8k6i4u0F1k6h3I4Q4x3X3c8Z5k6h3q4H3i4K6u0r">GROOMING THE IOS KERNEL HEAP</a>

主要就是介绍如何在 ios 的内核里面进行**占位式堆布局**。

具体思路是先清理堆中的内存碎片，然后用多个 victim object 包着一个 vuln object，这样就能提升溢出到 victim object 的成功率，如下图所示：

![image-20211006144548152](upload/attach/202110/690974_X7TUG3T5AVZTBGA.png)

*其中： V 表示 victim object， P 表示用于占位的对象，后面溢出的时候就会把 P 释放，然后分配 vuln object.*

为了进一步增加漏洞利用的成功率，可以多创建几个用于漏洞利用的区域，多次触发漏洞来提升成功率，如下图所示：

![image-20211006144923461](upload/attach/202110/690974_WWT7TD3KW6736CC.png)

### 小结

通过在 vuln obj 前后多包裹几个 victim object 和创建多个漏洞利用区域来提升成功率的思路值得借鉴。

## 目录

- [概述](https://bbs.kanxue.com/#概述)
- [A tale of two mallocs](https://bbs.kanxue.com/elink@0fdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6$3K9h3#2W2L8#2)9J5k6h3y4G2L8g2)9J5c8U0t1%4x3o6b7#2y4o6f1^5z5l9`.`.)
- [CVE-2015-3864](https://bbs.kanxue.com/elink@a6dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5k6h3u0D9L8$3N6K6M7r3!0@1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4y4g2)9J5c8U0l9&6i4K6u0r3M7%4c8S2k6$3g2X3M7X3W2Y4K9s2c8W2L8X3g2V1i4K6u0W2K9s2c8E0L8l9`.`.)
- [CVE-2017-0781](https://bbs.kanxue.com/elink@d29K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5i4m8W2M7W2)9J5k6i4y4W2k6h3u0#2k6#2)9J5k6h3!0J5k6#2)9J5c8U0b7K6x3q4)9J5c8R3`.`.)
- [Take Down MacOS Bluetooth with Zero-click RCE](https://bbs.kanxue.com/elink@a28K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4M7#2)9J5k6e0x3$3x3q4)9J5k6h3y4F1i4K6u0r3M7r3!0K6N6q4)9J5c8X3#2S2j5@1!0e0i4K6g2X3b7X3I4#2k6i4c8G2L8%4c8Z5k6q4)9#2k6U0m8Q4x3X3c8U0L8r3W2U0K9#2)9J5k6h3S2@1L8h3H3`.)

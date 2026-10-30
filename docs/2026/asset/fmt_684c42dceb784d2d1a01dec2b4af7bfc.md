
在实际调试过程中，命令行工具通常需要搭配`IDA`使用，用起来非常繁琐，且信息展示不够直观。`SEGGER`提供了一个图形界面工具——`Ozone`，可以在一定程度上解决以上问题，关于`Ozone`的介绍可以参考`SEGGER`的官方网站（653K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6k6h3N6Y4k6i4u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3M7X3!0V1N6h3y4@1M7#2)9J5c8X3c8W2N6X3g2D9L8%4m8E0k6h3&6@1i4K6u0V1N6r3!0G2L8s2y4Q4x3V1k6G2P5X3!0F1k6g2)9J5k6r3A6Q4x3X3c8D9K9h3&6C8i4K6u0V1k6r3g2T1N6h3N6Y4k6i4u0Q4x3V1k6Q4c8f1k6Q4b7V1y4Q4z5o6W2Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0S2Q4b7V1c8Q4b7f1k6Q4c8e0c8Q4b7V1u0Q4b7U0k6Q4c8e0c8Q4b7V1u0Q4b7e0g2Q4c8e0g2Q4z5p5k6Q4z5p5q4Q4c8e0c8Q4b7V1c8Q4b7V1k6Q4c8e0N6Q4z5e0c8Q4b7e0S2Q4c8e0k6Q4z5o6W2Q4z5p5u0Q4c8e0g2Q4z5o6k6Q4z5p5y4Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0c8Q4b7U0S2Q4z5p5u0Q4c8e0S2Q4b7V1c8Q4b7V1c8Q4c8e0W2Q4b7e0q4Q4b7U0g2Q4c8e0W2Q4z5f1c8Q4b7e0u0Q4c8e0c8Q4b7U0W2Q4z5f1k6Q4c8e0g2Q4z5p5k6Q4b7f1k6Q4c8e0c8Q4b7V1u0Q4b7e0g2Q4c8e0g2Q4z5f1y4Q4b7e0S2Q4c8e0S2Q4b7V1k6Q4z5e0W2Q4c8e0W2Q4z5o6N6Q4z5p5y4Q4c8e0k6Q4z5o6W2Q4b7V1g2Q4c8e0g2Q4z5o6S2Q4b7U0m8Q4c8e0y4Q4z5o6m8Q4z5o6u0Q4c8e0c8Q4b7U0S2Q4z5p5u0Q4c8e0S2Q4b7V1c8Q4b7V1c8Q4c8e0g2Q4z5e0m8Q4z5p5g2Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0g2Q4b7f1g2Q4z5o6W2Q4c8e0S2Q4b7e0y4Q4z5o6g2Q4c8e0c8Q4b7U0S2Q4z5o6m8Q4c8e0S2Q4b7U0N6Q4b7f1k6F1k6i4S2@1i4@1f1#2i4@1t1H3i4@1t1I4i4@1f1#2i4K6S2r3i4@1q4r3i4@1f1@1i4@1u0n7i4@1p5#2i4@1f1@1i4@1u0m8i4K6R3$3i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1$3i4K6R3K6i4@1t1K6i4@1f1#2i4@1u0r3i4K6R3#2i4@1f1#2i4@1p5@1i4@1p5%4i4@1f1#2i4@1q4q4i4@1t1$3i4@1f1&6i4K6R3K6i4@1u0p5i4@1f1#2i4@1u0q4i4K6R3^5i4@1f1%4i4K6R3$3i4K6W2r3i4@1f1%4i4@1u0n7i4K6R3K6i4@1g2r3i4@1u0o6i4K6R3^5i4@1f1$3i4@1u0n7i4K6V1I4i4@1f1%4i4@1p5^5i4@1u0p5i4@1g2r3i4@1u0o6i4K6R3&6i4@1f1K6i4K6R3H3i4K6R3J5

安装完成后，可以通过`File->New->New Project Wizard`选项来创建新的项目，创建时需要选择待调试芯片的型号、调试接口类型、通信速率等信息。继续，点击`Debug->Start Debug Session->Attach to Running Program`连接到待调试设备，注意整个调试过程中需要保持电脑、调试器和设备的接通状态，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202107/613694_DU5ZRYF827KZ8V2.jpg)

图3-9 连接到设备

连接到待调试设备后，`Ozone`可以像`IDA`一样展示多个subview，这样就可以同步观察很多信息了，下图是笔者某次调试的界面。

![](https://bbs.kanxue.com/upload/attach/202107/613694_E6MD5HJPXP8KMKW.jpg)

图3-10 Ozone调试界面

如上图所示，我们在调试时可以一次性看到寄存器的数据、多个内存区域的数据以及反汇编后的代码，下方的控制台区域可以执行一些指令或预先编写好的脚本，除此之外，左上角的断点设置区域，可以看到断点除了Location以外，还有Type和Extra两个属性，通过阅读用户手册可以确定这两个属性是用于设置断点类型的，如执行断点，读写断点、TRACE断点等，如下图所示：

![](https://bbs.kanxue.com/upload/attach/202107/613694_89TQDKKS4BBARTV.jpg)

图3-11 Ozone中断点的属性

`Ozone`还提供了很多强大的功能，在后文中，我们用到的时候就会逐一介绍这些功能。

### 3.3.2 通过SWD接口对门锁进行调试

翻阅门锁MCU的芯片手册可以看到，芯片提供了AES处理模块，该模块的内存映射如下图所示。

![](https://bbs.kanxue.com/upload/attach/202107/613694_5G65HJ776HDDE3N.jpg)

图3-12 EFM32中的内存映射

显然，我们假设鹿客门锁使用芯片的AES模块进行加解密操作，而不是自写AES算法，那么必然需要访问`0x400E0000~0x400E0400`这片区域中的内存地址，那么我们只要在设置适当的读写断点，然后等待手机与门锁进行通信时触发断点就可以了。

在断点设置区域右键，选择`Set Data Breakpoint`，会弹出如下图所示的窗口:

![](https://bbs.kanxue.com/upload/attach/202107/613694_TGPFNPVMXYGY8PK.jpg)

图3-13 设置数据端点

上图中的设置，表示当CPU向`0x400E00XX`地址写入数据时触发断点。

断点设置完成之后，在手机上点击开锁，由于我们设置的数据断点是监控一片内存区域，所以开锁过程中会多次被触发，后可以看到如下图左侧的一小段代码，右侧是执行到`0xED0C`地址时的寄存器数据。

![](https://bbs.kanxue.com/upload/attach/202107/613694_6XD4E98DA2DZ57Y.jpg)

图3-14 写入待解密数据的断点

上图中，`0x400E001C`地址（R0 + 28）是`AES_DATA`寄存器，这一小段代码所处的函数向`AES_DATA`寄存器写数据，应该就是AES的处理函数（下文以`AESFunc`代称）。回溯调用栈，可以找到调用`AESFunc`的外层函数，以及`AESFunc`函数的起始地址，进而使用`IDA`的`F5`功能对`AESFunc`函数进行分析，如下图：

![](https://bbs.kanxue.com/upload/attach/202107/613694_BJMBPHRCQ9X93AK.jpg)

图3-15 AESFunc函数的伪代码

结合芯片手册，由伪代码很容易能判断出`AESFunc`各个参数的作用，在本系列第一篇文章中，我们已经知道了开锁时的解密密钥就是`BleKey`，而门锁`BleKey`的获取过程，是由服务器下发一组数据`totalData`至手机app，app没有进行任何处理，直接通过BLE通信转发给了门锁。那么，接下来看一看门锁是如何处理`totalData`的。

首先我们需要看一下，`totalData`的内容是什么，如下图所示：

![](https://bbs.kanxue.com/upload/attach/202107/613694_Q8DGD6J9ZURVK68.jpg)

图3-16 totalData字段及其base64解码数据

可以看到`totalData`是一串二进制数据经过`base64`编码后的结果，红框之前的部分可以视作数据的header，包含消息头、数据包序号、校验等内容， header之后的body部分，即红、蓝、黑框中的数据，这三组数据的结构是相同的，如下图所示：

![](https://bbs.kanxue.com/upload/attach/202107/613694_TZR45XSAM4XEVW8.jpg)

图3-17 totalData中payload用到的数据结构

红框中`data_type`为`0x03`的数据，其`data_content=0x5FFFECAE`，该部分与AES根密钥密文有关。`totalData`的body部分还有两组数据，其具体作用不再详细说明，说太多有些不妥。

接着，我们在`AESFunc`的入口下一个断点，可以看到`BleKey`的值如下图所示：

![](https://bbs.kanxue.com/upload/attach/202107/613694_E3J5JPE3NYVVQ76.jpg)

图3-18 内存中的BleKey

从图3-16和图3-18中看不到`BleKey`和`totalData`之间的联系，想必`totalData`和`BleKey`之间还是经过了某些解密或解码转换，我们需要继续寻找门锁对`totalData`的处理过程。保持`AESFunc`函数的断点，在调试过程中可以发现`totalData`也是经过 `AESFunc`函数进行解密的。此时，回溯调用栈即可找到如下图所示的关键代码。

![](https://bbs.kanxue.com/upload/attach/202107/613694_VRQEE8FTCSTM8YX.jpg)

图3-19 totalData数据的处理

上图中的`AESEntry`函数为`AESFunc`的封装，这部分就是由`totalData`生成`BleKey`的核心代码，该流程可以整理为下图。

![](https://bbs.kanxue.com/upload/attach/202107/613694_2E4P6W47DZBDYZA.jpg)

图3-20 BleKey获取流程图

在这个流程中，如果没有根密钥就无法解密`BleKey`，通过调试可以找到根密钥在Flash中的存储地址是`0x7E09C`。

# 4. bootloader和FreeRTOS的分析

在第三章中，我们直接用SWD调试器定位到了关键代码，这种分析方法固然好，但却跳过了很多有价值的地方。在第四章中，我们回过头来从另一个角度分析这个设备，即从设备上电开始的第一条代码开始分析，一点一点理解整个固件的内容。

## 4.1 bootloader分析

`bootloader`是设备上电之后首先运行的一段程序，它会完成必要的初始化操作，并引导操作系统的加载。在我们拿到设备固件，并顺利通过`IDA`载入固件之后，可以看到解析出的内容仅是完整固件的一小部分，而更多的固件内容还是未解析状态。这被解析出来的一小部分，其实就是设备的`bootloader`部分代码。

为帮助我们分析这个云丁鹿客的智能门锁，我们可以尝试接通设备的串口。串口的位置就在SWD接口的旁边，可以参考云丁鹿客智能门锁系列第二篇中的图3-1。此时，给设备上电，是可以看到串口有字符串输出的，如下图所示：

![](https://bbs.kanxue.com/upload/attach/202107/613694_JXP2MWZYMAKQ2P2.jpg)

图4-1 启动时的串口输出

上图中，前几条字符串是可以直接在`IDA`中搜索到引用的，而剩下的字符串则未找到引用地址，如下图：

![](https://bbs.kanxue.com/upload/attach/202107/613694_2DNBJTBGYXRTGXY.jpg)

图4-2 字符串引用

![](https://bbs.kanxue.com/upload/attach/202107/613694_RKU6SS8FVCGDWJE.jpg)

图4-3 字符串未引用

造成上图中现象的原因是，`IDA`仅仅解析了`bootloader`部分代码，这些字符串的引用代码刚好是`bootloader`部分输出的；其余字符串是由主程序输出的，`IDA`未能解析出主程序而导致这些字符串在代码中未被引用。上图中，程序最终进入`sub_C44`函数，如下图：

![](https://bbs.kanxue.com/upload/attach/202107/613694_52EAXMERED5FEHZ.jpg)

图4-4 sub_C44函数

上图函数的功能很简单，即将`0x4100`存储到向量表偏移寄存器（`0xE000ED08`地址），也就是将`0x4100`地址的数据设置为新的中断向量表，随后调用`sub_F0`函数。

![](https://bbs.kanxue.com/upload/attach/202107/613694_5JHQXVTWKDZN4VN.png)

图4-5 sub_F0函数

结合图4-3和图4-4，`bootloader`最后会把`0x2001B860`（`0x4100`地址处的值）设置为新的栈指针，并跳转到`0x49ED`（`0x4104`地址处的值）。从这之后`bootloader`就将MCU的控制权交给了主程序。

综上，设备上电后会首先执行`bootloader`，`bootloader`会在Flash中搜索、校验固件，校验完成后，会设置新的中断向量表并跳转至主程序的入口地址。

## 4.2 FreeRTOS分析


实际上在MCU中`printf`和`puts`等函数的实现都是通过逐字节写入UART寄存器实现的。所以我们新建一个c文件并定义最简单的print函数如下：

```c
// hello\_m4.c
volatile unsigned int \* const UART\_TDR = (unsigned int \*)0x40008028; // LPUART1->TDR
void my\_print(const char \*data) {
while(\*data != '\0') {
\*UART\_TDR = (unsigned int)(\*data);
data++;
}
}
void my\_entry() {
my\_print("hello world!\n");
for(;;);
}
```

然后在之前的`Reset_Handler`稍加修改，令其跳转到我们的主程序执行：

```c
Reset_Handler:
ldr sp, =stack_top
bl my_entry
```

最后编译并重新链接：

```
arm-none-eabi-gcc -c -O0 -mcpu=cortex-m4 -g hello_m4.c -o hello_m4.o
arm-none-eabi-ld -T m4.ld hello_m4.o startup_m4.o -o hello_m4.elf
```

监听串口的数据并重新烧写，一个硬核的HelloWorld就完成了！

```sh
$ miniterm /dev/ttyACM0 115200
--- Miniterm on /dev/ttyACM0 115200,8,N,1 ---
--- Quit: Ctrl+] | Menu: Ctrl+T | Help: Ctrl+T followed by Ctrl+H ---
hello world!
```

> 如果串口是USART而不是UART，那么可能需要经过一些额外的配置，具体可以参考<a href="elink@7edK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2k6r3&6Q4x3X3g2U0L8$3#2Q4x3V1k6#2M7$3q4J5N6q4)9J5k6s2k6K6i4K6u0V1N6h3q4J5N6q4)9J5k6r3E0F1L8%4N6Q4x3X3g2Q4x3X3c8@1K9r3g2Q4x3X3c8V1K9h3k6X3k6i4u0W2L8X3y4W2i4K6u0r3">USART vs UART: Know the difference</a>。

在实际工程中，真正进入用户程序之前需要初始化好各个硬件外设，配置好基本的中断处理程序。这部分代码一般是由MCU vendor提供的，作为Bootloader(Boot ROM)固化。当然我们这里是绕过MCU直接针对CPU编写程序，以展示软硬件之间的微妙联系。

# 后记

本文主要介绍了CPU上电从硬件到软件的启动过程，其中一个关键概念就是中断向量表，这是所有First Stage Bootloader都需要理解和实现的地方。此外还介绍了如何通过控制串口在裸机上实现了一简单的HelloWorld应用，这实际上是一个简化的外设驱动，即通过总线读写外设寄存器来封装外部硬件的调用，这部分代码在内核中也是相当常见的。

# 参考资料

- [ARM datasheet][vec]

- [STM32L4XX datasheet][st1]

- [GNU linker script][ld]

- [OpenOCD -- Open On Chip Debugger][oc]

- <a href="elink@96fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3I4S2N6e0R3J5i4K6u0W2N6$3!0J5k6s2m8J5k6i4y4K6i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4x3q4)9J5c8U0l9J5i4K6u0r3x3e0c8Q4x3V1k6K6K9h3#2H3L8r3g2K6N6q4)9J5k6r3u0S2M7X3g2Q4x3X3c8E0k6i4c8S2L8q4)9J5k6s2m8J5L8$3N6J5j5h3#2Q4x3X3c8X3L8%4u0Q4x3X3c8S2M7X3#2Q4x3V1j5`.">bare metal program for ARM926ej</a>

- <a href="elink@f14K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9I4y4U0l9J5x3U0l9H3y4o6l9@1y4o6S2Q4x3V1k6Z5N6s2c8H3i4K6y4m8i4K6u0r3i4K6u0r3N6%4N6%4i4K6u0W2L8X3!0E0L8i4g2Q4x3X3g2G2M7X3N6Q4x3V1j5`.">NOMMU Linux</a>

[mini]: <mark class="encrypted">426K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8r3W2G2M7%4c8Z5K9h3&6Y4M7#2)9J5k6h3W2G2N6q4)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8R3`.`.</mark>

[sheet]: <mark class="encrypted">646K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6N6q4)9J5k6h3y4G2L8g2)9J5c8Y4u0W2M7$3!0#2M7X3y4W2i4K6u0r3k6h3&6Q4x3V1k6V1j5i4c8S2M7$3S2W2k6i4c8Q4x3V1k6K6N6r3@1K6x3X3H3@1z5e0k6S2k6g2)9J5k6i4m8V1k6R3`.`.</mark>

[st]: <mark class="encrypted">d50K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6N6q4)9J5k6h3y4G2L8g2)9J5c8Y4u0W2M7$3!0#2M7X3y4W2i4K6u0r3k6h3&6Q4x3V1k6J5k6h3k6W2M7X3g2F1j5$3g2Q4y4h3k6E0j5h3&6#2j5h3I4Q4x3V1k6V1L8e0l9H3x3o6R3K6y4e0j5H3i4K6u0V1M7%4c8E0x3K6u0D9y4s2R3#2i4K6u0V1j5h3&6V1i4K6u0V1M7%4c8E0x3K6u0D9y4s2R3$3i4K6u0V1j5h3c8$3j5h3&6U0k6h3c8Q4x3X3c8S2M7X3#2Q4x3X3c8T1j5i4y4W2k6q4)9J5k6o6x3J5i4K6u0V1j5X3W2@1i4K6u0V1L8h3y4#2M7#2)9J5k6s2y4@1L8h3W2U0M7X3!0W2L8r3g2U0N6s2u0G2L8X3W2U0M7#2)9J5k6i4m8V1k6R3`.`.</mark>

[st1]: <mark class="encrypted">2ccK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6N6q4)9J5k6h3y4G2L8g2)9J5c8Y4u0W2M7$3!0#2M7X3y4W2i4K6u0r3k6h3&6Q4x3V1k6S2M7s2m8D9K9h3y4S2N6r3W2G2L8W2)9#2k6X3&6G2N6r3g2Q4x3V1k6V1L8e0l9H3y4o6M7$3z5o6j5&6i4K6u0V1L8h3W2Y4M7X3q4@1K9h3&6Y4i4K6u0V1k6Y4u0G2L8g2)9J5k6s2y4@1L8e0x3J5L8o6c8Q4x3X3c8S2L8X3c8Q4x3X3c8K6N6r3@1K6x3X3H3@1i4K6u0V1M7$3g2J5K9h3g2K6i4K6u0V1N6r3!0Q4x3X3c8K6N6r3@1K6x3X3H3#2i4K6u0V1M7$3g2J5K9h3g2K6i4K6u0V1L8h3W2U0M7X3!0U0L8$3&6@1M7X3!0D9L8r3g2J5M7#2)9J5k6s2y4@1L8h3W2U0M7X3!0W2L8r3g2U0N6s2u0G2L8X3W2U0M7#2)9J5k6i4m8V1k6R3`.`.</mark>

[vec]: <mark class="encrypted">7e8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2M7X3#2Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8$3y4K6i4K6u0r3k6s2g2A6x3o6f1#2x3#2)9J5c8X3I4S2N6r3g2K6N6q4)9J5c8Y4c8Z5k6g2)9J5k6r3y4G2M7Y4c8W2P5q4)9J5k6r3@1@1i4K6u0V1M7s2u0G2j5$3g2K6M7$3!0J5i4K6u0r3k6i4S2U0k6i4m8@1K9h3!0F1i4K6u0V1L8h3!0V1k6h3I4Q4x3V1k6$3k6h3y4@1L8%4u0Q4x3X3c8@1j5h3u0D9k6b7`.`.</mark>

[mm]: <mark class="encrypted">9f7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2M7X3#2Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8$3y4K6i4K6u0r3k6s2g2A6x3o6f1#2x3#2)9J5c8X3u0Q4x3V1k6@1K9r3g2Q4x3X3c8U0L8%4u0@1k6i4S2Q4x3X3c8E0y4q4)9J5k6s2m8J5L8$3y4W2M7%4y4G2M7W2)9J5c8X3#2W2L8h3!0J5P5g2)9J5k6r3#2G2k6r3g2D9</mark>

[gnu]: <mark class="encrypted">7a5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3#2E0N6h3&6A6N6s2W2Q4x3X3g2S2M7X3#2Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3V1k6A6M7q4)9J5k6s2m8J5L8$3c8#2j5%4c8K6i4K6u0r3M7s2u0G2j5$3g2K6M7$3!0J5M7#2)9J5c8X3u0Q4x3V1k6H3M7X3!0U0k6i4y4K6L8%4u0K6i4K6u0V1K9i4m8Q4x3X3c8T1L8r3!0Y4i4K6u0r3M7r3!0K6N6s2y4Q4x3V1k6#2M7$3g2X3N6h3I4Q4x3X3c8S2M7%4y4W2L8h3u0D9k6i4u0Q4x3X3c8V1K9i4u0W2j5%4c8A6N6X3g2K6i4K6u0V1j5h3&6V1i4K6u0V1L8h3q4U0M7X3!0K6i4K6u0V1k6X3!0J5i4K6u0V1N6r3S2W2i4K6u0V1k6$3&6#2i4K6u0V1j5i4y4K6k6h3#2T1L8r3g2J5</mark>

[src]: <mark class="encrypted">842K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3I4S2N6e0R3J5i4K6u0W2N6$3!0J5k6s2m8J5k6i4y4K6i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4x3q4)9J5c8U0l9J5i4K6u0r3x3e0c8Q4x3V1k6K6K9h3#2H3L8r3g2K6N6q4)9J5k6r3u0S2M7X3g2Q4x3X3c8E0k6i4c8S2L8q4)9J5k6s2m8J5L8$3N6J5j5h3#2Q4x3X3c8X3L8%4u0Q4x3X3c8S2M7X3#2Q4x3V1j5`.</mark>

[ld]: <mark class="encrypted">98bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4U0L8$3u0W2M7X3I4A6L8W2)9J5k6h3c8W2i4K6u0r3j5$3!0F1N6r3g2F1N6q4)9J5c8X3#2W2k6r3W2S2i4K6u0r3K9s2c8@1M7q4)9J5c8X3W2F1k6X3!0J5L8h3q4@1K9h3E0Q4x3V1k6Y4j5$3y4Q4y4h3k6V1L8$3y4K6i4K6u0r3L8r3c8Q4y4h3j5K6i4K6u0W2K9s2c8E0L8l9`.`.</mark>

[oc]: <mark class="encrypted">60bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2X3L8%4u0Y4k6g2)9J5k6h3&6W2N6q4)9J5c8Y4m8J5L8$3A6W2j5%4c8K6i4K6u0r3L8%4m8W2L8X3!0U0k6q4)9J5c8R3`.`.</mark>

[devkit.png]: upload/attach/202004/844554_Z7YWAKUC9H94PGK.png

[vec.png]: upload/attach/202004/844554_CKXM6JE2EHBFDQG.png

[mm.png]: upload/attach/202004/844554_HHVB2BRTC2SFUV7.png

[mm1.png]: upload/attach/202004/844554_UCGD5AG2YZ7HH5Z.png

[jtag.png]: upload/attach/202004/844554_B7FUBMM4CGAKWU4.png

[usart.png]: upload/attach/202004/844554_QFXZACAVEWMA4B8.png

*Fundamental, fundamental, fundamental, ...without the fundamental, all those fancy magics won't work.*

有经验的程序员都知道源码需要进行编译、链接、封装，然后才能执行。那你知道如何为一块CPU编写并编译程序吗？知道编译后的程序如何写入MCU、并让CPU加载运行的吗？

# 前言

早先收到阿里云提供的**Developer kit**开发板，对他们的RTOS进行体验，就是下面这款：

![devkit.png](https://bbs.kanxue.com/upload/attach/202004/844554_Z7YWAKUC9H94PGK.png)

不得不说，使用aos全家桶运行、烧写和调试代码都非常方便；而且最近看发现还支持[最小化定制裁剪](https://bbs.kanxue.com/elink@449K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8r3W2G2M7%4c8Z5K9h3&6Y4M7#2)9J5k6h3W2G2N6q4)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8R3`.`.)，根据自己的需求下载对应的代码，算是咱256G小硬盘的福音了：）

不过今天不是分析阿里的RTOS(`AliOS Things`)，也不是把玩这块开发板，而是借助其中的MCU来探索下裸板的开发和运行之路。

# 芯片分析

在开始为一块MCU编程之前，我们要做的第一件事就是先查看这个MCU的文档。例如，如果我们想写一个helloworld程序，那么就至少需要知道：

1. MCU复位之后从哪启动，这决定了我们的main程序位置。

2. MCU内存映射，这是为了查看串口的地址空间。

对于我们而言，手上的MCU型号是`STM32L496VGTx`，因此这些大部分都能在[stm32l496ae datasheet](https://bbs.kanxue.com/elink@436K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6N6q4)9J5k6h3y4G2L8g2)9J5c8Y4u0W2M7$3!0#2M7X3y4W2i4K6u0r3k6h3&6Q4x3V1k6V1j5i4c8S2M7$3S2W2k6i4c8Q4x3V1k6K6N6r3@1K6x3X3H3@1z5e0k6S2k6g2)9J5k6i4m8V1k6R3`.`.)中查看到。首先，在datasheet中我们知道STM32L496VGTx中的CPU是`ARM Cortex-M4`，内存SRAM为**320KB**，内部含有1MB的Flash。

## 初始化

根据ARM的文档中关于[`Cortex-m4`中断向量表](https://bbs.kanxue.com/elink@972K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2M7X3#2Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8$3y4K6i4K6u0r3k6s2g2A6x3o6f1#2x3#2)9J5c8X3I4S2N6r3g2K6N6q4)9J5c8Y4c8Z5k6g2)9J5k6r3y4G2M7Y4c8W2P5q4)9J5k6r3@1@1i4K6u0V1M7s2u0G2j5$3g2K6M7$3!0J5i4K6u0r3k6i4S2U0k6i4m8@1K9h3!0F1i4K6u0V1L8h3!0V1k6h3I4Q4x3V1k6$3k6h3y4@1L8%4u0Q4x3X3c8@1j5h3u0D9k6b7`.`.)的介绍，我们可以看到保存第一条指令地址的地址为`0x0004`：

![vec.png](https://bbs.kanxue.com/upload/attach/202004/844554_CKXM6JE2EHBFDQG.png)

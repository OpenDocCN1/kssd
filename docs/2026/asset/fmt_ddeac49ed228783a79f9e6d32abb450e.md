
# [原创]CVE-2017-13258 Android 蓝牙BNEP漏洞分析

## 1、概述

三月份的Android 安全公告中披露了多个蓝牙方面的系统层漏洞，漏洞多出现在BNEP中，涉及的源码版本为`5.1.1`, `6.0`, `6.0.1`, `7.0`, `7.1.1`, `7.1.2`, `8.0`, `8.1`，覆盖范围较广。本文以CVE-2017-13258漏洞为例进行分析，该漏洞类型为内存信息泄露，危害等级为High。

## 2、协议简介

### 2.1介绍

BNEP（Bluetooth Network Encapsulation Protocol）为蓝牙网络封装协议。BNEP将多种通过蓝牙L2CAP（逻辑链路控制与适配协议）传输的网络协议数据包进行封装处理。L2CAP为蓝牙提供数据链路层。BNEP被用于通过蓝牙来传输控制和数据包以此为蓝牙设备提供联网功能。BNEP提供的功能类似于以太网（EthernetII/DIX/Framing /IEEE 802.3）提供的。

### 2.2 协议栈

如下图所示：

![](https://bbs.kanxue.com/upload/attach/202002/611407_69YP9XEQQX9PZRY.png)

### 2.3 BNEP头部格式

所有的BNEP 头部格式都遵从下图所描述的格式。所有支持BNEP的设备必须具备能够解析定义好的BNEP包类型的能力。支持BNEP的设备有可能会处理压缩的BNEP头部，也有可能不会。任何包含保留的BNEP头数据包类型的数据包必须下放到处理扩展头部的过程中进行处理。

![](https://bbs.kanxue.com/upload/attach/202002/611407_WC7Z8PFUZ45JVXA.png)

前一个字节中，BNEP Type 大小为7bit，Extension Flag 为 1 bit。BNEP Type 的值如下图所示。

![](https://bbs.kanxue.com/upload/attach/202002/611407_4RYQC2MNR5DAG5K.png)

BNEP Type 的value设置为`0x01`，也即是`BNEP_CONTROL`，属于控制包。E标志置0表示BNEP头部后面紧跟着就是Payload，置1表示BNEP头部后面是扩展头部，需要进一步解析。由于本案例中触发漏洞的数据包是控制包，这里就直接讲解`BNEP_CONTROL`包的格式。`BNEP_CONTROL`包是用来交换控制信息，包格式如下图所示。

![](https://bbs.kanxue.com/upload/attach/202002/611407_N4HW8AUTDRV443W.png)

从8到15位属于BNEP Control Type域，占一个字节大小。BNEP Control Type分为多种，具体值如下图所示。

![](https://bbs.kanxue.com/upload/attach/202002/611407_NCER765GEHMJUJ8.png)

BNEP Contrl Type值为`0x00`表示`BNEP_CONTROL_COMMAND_NOT_UNDERSTOOD`类型。大致理解为该控制包发送的控制指令无法被解析或理解。该数据包最终格式如下图所示：

![](https://bbs.kanxue.com/upload/attach/202002/611407_FRHH4JSPZU6AZDM.png)

最后再简单提一下扩展包格式。我们前面讲到，只要是E 标志位被置1后，BNEP头部后面紧跟着就是扩展包头部。扩展包格式如下图所示。

![](https://bbs.kanxue.com/upload/attach/202002/611407_SET2AZXPZ4AXMUX.png)

前一个字节依然是类型，第二个字节是扩展包长度，后面从第三个字节开始就是扩展包载荷。这里的Extension Type取值如下图所示。

![](https://bbs.kanxue.com/upload/attach/202002/611407_2WJMBBYD3XBTT7T.png)

Extension Type取值为`0x00`表示`BNEP_EXTENSION_CONTROL`。这里需要表明一下，`BNEP_EXTENSION_CONTROL`数据包类型头和`BNEP_CONTROL`数据包类型头可以互换使用。

## 3、漏洞原理

该漏洞出现在`bnep_data_ind`函数中，当从L2CAP层接收到数据时，该函数被调用。文件路径为`system\bt\stack\bnep\bnep_main.c`。本文以`8.0.0_r4`版本为例。

![](https://bbs.kanxue.com/upload/attach/202002/611407_DYXP4GZYBK4BNY8.png)

`p_buf`指针指向的一个`BT_HDR`数据块，行434，计算存放数据包的地址，`p`指向数据包起始地址。`BT_HDR`结构体如下所示：

![](https://bbs.kanxue.com/upload/attach/202002/611407_52QQ2CEGH2MWWV3.png)

`Len`存放数据包的长度，`offset`存放偏移地址，用于计算数据包的起始地址。接着向下分析。

![](https://bbs.kanxue.com/upload/attach/202002/611407_BCYJVA9JJTXGQ5Y.png)

行450，取`TYPE`，然后`p`跳过一个字节。行451，计算E标志位。行453，判断数据包长度, 最小值不能小于头部长度和最大值不能大于BNEP的MTU值。然后数据包长度减一。

![](https://bbs.kanxue.com/upload/attach/202002/611407_YH7Q4VRD5KASQJN.png)

行469，如果E标志位为真，就进入扩展头解析。行481，取扩展头的E标志位，行482，取扩展包包含的payload长度，这个长度不包括头部域和长度域的两个字节。这个`length`域是用户可控的。行483，`p`指针跳过第一个扩展包的`payload`数据域，指向下一个数据包。

行485，判断没有下一个扩展包并且发送的`BNEP_CONTROL_TYPE`不属于已有定义类型，进入`bnep_send_command_not_understood(p_bcb,*p)`，该函数是将错误指令回写到数据包中，并返给客户端。如下图所示。

![](https://bbs.kanxue.com/upload/attach/202002/611407_JAM8WXYU77Z2BJW.png)

根据以上分析，就可以构造特定数据包绕过检测进行内存信息泄漏。

## 4、漏洞复现

Exploit-db网站已经发布了利用代码，链接地址在文末。关键利用代码，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202002/611407_AJ69UCH3HP7A5MB.png)

在刷入版本8.0AOSP的nexus5x上测试，结果如下图所示。

![](https://bbs.kanxue.com/upload/attach/202002/611407_Z6882KUMXSNS8JF.png)

## 5、漏洞调试

在`bnep_data_ind`函数上下断点。如下图所示。

![](https://bbs.kanxue.com/upload/attach/202002/611407_6XWU9JCQ5XXFFYC.png)

执行利用代码，命中断点后。打印相关数据，如下图所示

![](https://bbs.kanxue.com/upload/attach/202002/611407_38UW54CTJ4KUMW5.png)

```
地址0x7a408d8a58为存放数据包的地址。执行到行486处，即将写入返回数据。X1寄存器存放着泄漏的数据，该值为0xc9。如下图所示。
```

![](https://bbs.kanxue.com/upload/attach/202002/611407_M58KG28EYWXZEY7.png)

从`0x7a408d8a58`开始，跳过三字节数据包头部，跳过扩展包一个字节的payload，`0x7a408d8a5c`处的`0xc9`作为Unknown Control Type返回给客户端。结束。

=================================================我是分割线=======================================

```
调试过程的一些小tips：

Cat /proc/`pid`/maps | grep bluetooth.default.so 确定调用的so是32位还是64位。以免gdbclient挂载不上。

prebuilts/misc/android-arm64/gdbserver64 推到手机里面

Adb root

adb remount 重新挂在文件系统。

 Cp /sdcard/gdbserver64 /system/bin/



Gdbclient `pid`



Info sharedlibrary  查看加载模块，找基地址。

 list bnep_data_ind 直接列出函数

计算函数偏移：

lib_start_addr + 相对虚拟地址（虚拟地址-区段起始地址），
```

相关链接

111K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2P5s2m8D9L8$3W2@1i4K6u0V1k6r3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2P5s2m8D9L8$3W2@1M7#2)9J5c8U0b7@1x3K6t1$3i4K6u0r3

2deK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3N6J5L8%4g2H3k6i4u0Q4x3X3g2A6k6h3g2W2i4K6u0W2L8%4u0Y4i4K6u0r3k6%4u0G2N6i4m8K6i4K6u0r3z5o6l9J5i4K6u0r3x3e0g2Q4x3V1k6n7L8s2g2W2N6r3!0G2N6r3S2Q4x3V1k6n7e0V1g2b7i4K6u0W2M7r3c8X3

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

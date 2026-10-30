
<a href="elink@0bcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3N6i4l9J5P5g2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3x3U0l9J5x3q4)9J5c8U0l9K6i4K6u0r3x3K6m8Q4x3V1k6D9N6g2)9J5k6s2W2G2N6g2)9J5k6s2q4A6i4K6u0V1L8r3!0#2i4K6u0V1k6r3!0F1k6#2)9J5k6s2N6S2i4K6u0V1K9Y4g2W2i4K6u0V1K9s2g2S2L8W2)9J5k6r3A6A6L8X3N6Q4x3X3c8V1j5g2)9J5k6r3A6A6j5h3&6Q4x3V1j5`."><mark class="encrypted">d94K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3N6i4l9J5P5g2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3x3U0l9J5x3q4)9J5c8U0l9K6i4K6u0r3x3K6m8Q4x3V1k6D9N6g2)9J5k6s2W2G2N6g2)9J5k6s2q4A6i4K6u0V1L8r3!0#2i4K6u0V1k6r3!0F1k6#2)9J5k6s2N6S2i4K6u0V1K9Y4g2W2i4K6u0V1K9s2g2S2L8W2)9J5k6r3A6A6L8X3N6Q4x3X3c8V1j5g2)9J5k6r3A6A6j5h3&6Q4x3V1j5`.</mark></a>[2]

3，GDB+GDBServer调试Linux应用程序

<a href="elink@79aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3j5%4y4D9N6h3&6S2N6r3W2U0i4K6u0r3M7q4)9J5c8U0x3$3x3K6f1#2x3U0m8Q4x3X3g2Z5N6r3#2D9"><mark class="encrypted">ab0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3j5%4y4D9N6h3&6S2N6r3W2U0i4K6u0r3M7q4)9J5c8U0x3$3x3K6f1#2x3U0m8Q4x3X3g2Z5N6r3#2D9</mark></a>

4，配置文件

<a href="elink@c28K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3k6W2)9J5k6r3y4C8i4K6u0V1L8X3g2W2k6q4)9J5k6s2g2Q4x3V1k6H3i4K6u0r3y4K6j5K6y4U0R3K6y4W2)9J5k6h3S2@1L8h3H3`.">httpd配置文件httpd.conf规则说明和一些基本指令</a>

5，<<揭秘家用路由器0day漏洞挖掘技术>>

# D-LINK DIR-815多次溢出漏洞

## 目录

- D-LINK DIR-815多次溢出漏洞
- [说明](https://bbs.kanxue.com/#说明)
- [所需相关资源](https://bbs.kanxue.com/#所需相关资源)
- [漏洞介绍](https://bbs.kanxue.com/#漏洞介绍)
- [漏洞分析](https://bbs.kanxue.com/#漏洞分析)
- [漏洞定位](https://bbs.kanxue.com/#漏洞定位)
- [IDA静态调试分析-定位漏洞溢出点](https://bbs.kanxue.com/#ida静态调试分析-定位漏洞溢出点)
- [IDA动态调试-确定偏移位置（手动+自动两种方法）](https://bbs.kanxue.com/#ida动态调试-确定偏移位置（手动+自动两种方法）)
- [构造ROP](https://bbs.kanxue.com/#构造rop)
- [gdb-multiarch+QEMU动态调试分析验证](https://bbs.kanxue.com/#gdb-multiarch+qemu动态调试分析验证)
- cache incoherency问题（影响EXP执行）
- [坏字符问题](https://bbs.kanxue.com/#坏字符问题)
- [qemu系统模式](https://bbs.kanxue.com/#qemu系统模式)
- [gdbserver调试](https://bbs.kanxue.com/#gdbserver调试)
- [编写exp](https://bbs.kanxue.com/#编写exp)
- [总结](https://bbs.kanxue.com/#总结)
- [参考](https://bbs.kanxue.com/#参考)

## 说明

原创作者：herculiz

以下内容纯原创辛苦手打，望大家多多支持。

首先感谢各位以下相关链接前辈师傅的知识分享精神，才萌生了本文。

由于年限及知识的更新，笔者觉得过去许多描述可能对新手不太友好（自己的痛苦经历），所以对其进行了补充及概述。

## 所需相关资源

- 固件下载：`ftp://ftp2.dlink.com/PRODUCTS/DIR-815/REVA/DIR-815_FIRMWARE_1.01.ZIP`
- gdbserver各架构对应调试文件（已经编译好了可以直接使用）：cd5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5j5i4m8A6k6o6N6Q4x3V1k6W2L8h3u0W2k6r3c8W2k6q4)9J5k6s2c8G2L8$3I4K6i4K6u0r3N6s2u0W2k6g2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3u0A6L8X3q4J5K9h3g2K6i4K6u0r3k6$3c8T1M7$3g2J5N6X3g2J5
- IDA6.8/IDA7.5：一个用在ubuntu调试，一个用在windows调试（7.5有更高级的mips反汇编器，可以方便参考伪代码）。或者谢安在`Ghidra`工具，专门反汇编反编译mips架构程序，目前只用于静态分析。
- qemu-mips仿真的相关内核和虚拟硬盘（这里有坑，后面在说，总之一定要两者`内核`和`虚拟硬盘`匹配）：e3aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3k6h3!0H3L8r3g2Q4x3X3g2V1k6h3u0A6j5h3&6Q4x3X3g2G2M7X3N6Q4x3V1k6Q4y4@1g2S2N6i4u0W2L8o6x3J5i4K6u0r3M7h3g2E0N6g2)9J5c8R3`.`.

## 漏洞介绍

exploitdb介绍:bf8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2P5s2m8D9L8$3W2@1i4K6u0V1k6r3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2P5s2m8D9L8$3W2@1M7#2)9J5c8U0x3K6z5o6j5K6

![图片描述](https://bbs.kanxue.com/upload/attach/202107/913238_4BRQSE3VYP9Q4SJ.png)

|  |  |
| --- | --- |
| 1 2 | Buffer overflow on “`hedwig.cgi`”  Another buffer overflow affects the “`hedwig.cgi`” CGI script. Unauthenticated remote attackers can invoke this CGI with an overly-long cookie value that can overflow a program buffer and overwrite the saved program address. |

从漏洞公告中可以看出，该漏洞存在于名为`“hedwig.cgi”`的CGI脚本中，未认证攻击者通过调用这个CGI脚本传递一个**超长的Cookie值**，使得程序**栈溢出**，从而获得路由器的远程控制权限。

## 漏洞分析

### 漏洞定位

1，`binwalk -Me`解压提取固件

![图片描述](https://bbs.kanxue.com/upload/attach/202107/913238_M77F5KG34GD3R7U.png)

2，该漏洞的核心组件为`hedwig.cgi`，`find . -name '*cgi'`查找文件，并`ls -l ./htdocs/web/hedwig.cgi`发现`hedwig.cgi`是指向`./htdocs/cgibin`的符号链接，也就是说真正的漏洞代码在`cgibin`中。

![图片描述](https://bbs.kanxue.com/upload/attach/202107/913238_B6G7QSA22WN37QH.png)

### IDA静态调试分析-定位漏洞溢出点

1，用IDA静态调试`cgibin`文件，`hedwigcgi_main`函数处理真个过程，由于是`HTTP_COOK`这个字段引起的漏洞溢出点，可以在IDA（SHIFT+F12）搜索字符串，然后通过X，交叉引用来跟踪到`hedwigcgi_main`函数条用的位置。

![图片描述](https://bbs.kanxue.com/upload/attach/202107/913238_YUZ2WMAZKS9D3TZ.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202107/913238_G4CPAXBA9J3X5XH.png)

2，跟踪到主函数的位置，对函数功能进行大致分析，或者利用Ghidra或IDA7.5反汇编`hedwigcgi_main`函数，可以定位到其中的`sprintf`函数引起了栈溢出（其实在后面还有一个`sprintf`函数调用，它才是真实环境利用的位置，后面讲解说明）。`hedwigcgi_main`函数通过`sess_get_uid()`获取到`HTTP_COOKIE`中`uid=`之后的值，并将该内容按照`sprintf`函数中格式化字符串给定的形式拷贝到栈中，由于没有检测并限制输入的大小，导致栈溢出。

![图片描述](https://bbs.kanxue.com/upload/attach/202107/913238_3JJV9QWV3ZCVSNG.png)

[参考1：函数功能流程说明](<https://bbs.kanxue.com/elink@de7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6C8K9i4u0A6L8W2)9J5k6s2y4S2P5g2)9J5k6i4c8G2M7q4)9J5c8U0t1H3x3e0W2Q4x3V1j5H3x3W2)9J5c8U0t1K6i4K6u0r3b7Y4g2A6L8r3c8A6L8X3N6Q4x3X3c8y4d9g2m8e0i4K6u0V1c8h3&6$3K9i4u0G2L8X3#2W2L8Y4c8Q4x3X3c8X3L8%4u0Q4x3X3c8d9L8%4g2@1k6i4u0Q4x3X3c8b7g2@1&6Q4x3V1j5`.)>)

## IDA动态调试-确定偏移位置（手动+自动两种方法）

这里需要用到`qemu`+IDA动态调试的方法：[参考2：qemu+IDA动态调试](<https://bbs.kanxue.com/elink@5f0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6W2K9i4S2A6L8W2)9#2k6U0b7K6x3e0V1@1z5e0t1I4i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3o6b7%4x3o6b7H3y4o6R3`.)>)

此处是`qemu`用户模式下仿真程序启动，`qemu`有两种模式，`用户模式`和`系统模式`,详细工具使用这在不影响思路的情况下为了避免篇幅冗余过长就省略了。

![图片描述](https://bbs.kanxue.com/upload/attach/202107/913238_2N663U5Q9SYY3XA.png)

文件时小端有效的mips指令集，我们使用`qemu-mipsel`

`注意：` `qemu-mipsel` 由于依赖各种动态库，避免出现各种问题，我们这里手动复制库到当前目录下（`squashfs-root`目录下）

1，查看`qemu-mipsel`依赖的libc

![图片描述](https://bbs.kanxue.com/upload/attach/202107/913238_K439JQ6ZK3UH6EF.png)

2，直接创建后面的目录名，并复制动态链接库

```bash
mkdir -p ./usr/lib/
mkdir -p ./lib/x86_64-linux-gnu/
mkdir -p ./lib64/
cp -p /usr/lib/x86_64-linux-gnu/libgmodule-2.0.so.0 ./usr/lib/
cp -p /lib/x86_64-linux-gnu/libglib-2.0.so.0 ./lib/x86_64-linux-gnu/
cp -p /lib/x86_64-linux-gnu/librt.so.1 ./lib/x86_64-linux-gnu/
cp -p /lib/x86_64-linux-gnu/libm.so.6 ./lib/x86_64-linux-gnu/
cp -p /lib/x86_64-linux-gnu/libgcc_s.so.1 ./lib/x86_64-linux-gnu/
cp -p /lib/x86_64-linux-gnu/libpthread.so.0 ./lib/x86_64-linux-gnu/
cp -p /lib/x86_64-linux-gnu/libc.so.6 ./lib/x86_64-linux-gnu/
cp -p /lib/x86_64-linux-gnu/libdl.so.2 ./lib/x86_64-linux-gnu/
cp -p /lib/x86_64-linux-gnu/libpcre.so.3 ./lib/x86_64-linux-gnu/
cp -p /lib64/ld-linux-x86-64.so.2 ./lib64/
```

3，通过脚本调试

当然也能直接通过以下参数方式调试，但个人感觉用习惯了脚本方式能更方便的修改参数内容和视觉上的简约。

```bash
sudo chroot ./ ./qemu-mipsel -E CONTENT_LENGTH=20 -E CONTENT_TYPE="application/x-www-form-urlencoded" -E REQUEST_METHOD="POST" -E HTTP_COOKIE=`python -c "print 'uid=123'+'A'\*0x600"`-E REQUEST_URI="/hedwig.cgi" -E REMOTE_ADDR="192.168.x.x" -g 1234 ./htdocs/web/hedwig.cgi
```

3.1自己编写的调试`tesh.sh`脚本

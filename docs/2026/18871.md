
# TP-Link WR740 后门漏洞复现（学习记录）

## 0x01 复现环境

- `binwalk`
- `firmwalker`
- `IDA`
- `FirmAE`

## 0x02 固件分析

### 2.1 获取固件文件系统-binwalk

`binwalk`用于固件解包获取文件系统。常用解包命令为：

```bash
binwalk -Me wr740nv1\_en\_3\_12\_4\_up.bin
```

![](upload/attach/202307/882513\_7868W4ZKJHMHB2P.png)

![](upload/attach/202307/882513\_ACRDSK7DYB74UF6.png)

### 2.2 扫描敏感信息-firmwalker

`firmwalker`用于查看文件系统中的敏感信息。扫描命令为：

```bash
./firmwalker.sh '/home/iot/Desktop/WR740/\_wr740nv1\_en\_3\_12\_4\_up.bin.extracted/squashfs-root'
```

![](upload/attach/202307/882513\_EW38JHXCRR7BC35.png)

由上面的信息可知：

- 路由器Web相关URL的格式为：`/web/userRpm/xxxxx.htm`，这可能是web服务相关的命令规范。
- 含有`passwd`和`pwd`相关的文件中，可以看到嵌入式web服务`httpd`，部分`htm`文件以及其他配置文件。

查看启动项中是否存在后门服务，如`telnet`服务。嵌入式Linux常用启动项文件位于：`etc/rc.d`文件夹中。

![](upload/attach/202307/882513\_CGZTBNANZ3W5JVQ.png)

启动项中未见异常行为，`httpd`文件在`firmwalker`扫描的敏感信息和启动项中都有涉，因此需要用`IDA`分析一下该文件。

### 2.3 分析可疑文件-IDA

![](upload/attach/202307/882513\_44TYY6MKN5B9WNG.png)

由`file`信息可知用`IDA32`分析，根据`firmwalker`扫出的敏感信息可知`httpd`文件中含`passwd`、`password`、`root`、`admin`、`upgrade`等字符串。

通过`IDA`中字符串搜索发现疑似后门的敏感字符。

![](upload/attach/202307/882513\_M4EW9KAZ6H5DBZ8.png)

通过查看该字符串的引用发现如下代码：

![](upload/attach/202307/882513\_VKVE7YTJ58GJPWQ.png)

根据伪代码可知其的功能是：判断命令是否为`exit`，若不是则验证用户名和密码，验证的用户名：`osteam` 密码：`5up`。

通过`write(pty,cmd,strlen(cmd))`来模拟执行命令。（知识扩充：伪终端`pseudo terminal`也称为 `pty`是指伪终端 master 和伪终端 slave 这一对字符设备。其中的 slave 对应 `/dev/pts/` 目录下的一个文件，而 master 则在内存中标识为一个文件描述符`fd`。伪终端由终端模拟器提供，终端模拟器是一个运行在用户态的应用程序。）

![](upload/attach/202307/882513\_CC7J8SEU8KZE6S6.png)

通过查看引用找到了`DebugResultRpmHtm`函数的调用位置。根据此处的伪代码可推断出函数`httpRpmConfAdd`的功能是绑定URL和执行的函数。由函数名`httpDebugInit`可推断出这是一个调试后门。

## 0x03 固件模拟

使用`FirmAE`来对固件进行模拟以验证上述内容。

`FirmAE`是一款完全自动化的固件模拟和漏洞分析的工具，其固件模拟成功率与`Firmadyne`工具相比大大提升，且与`Qemu`相比仅一条命令就可实现固件模拟，简单方便。`FirmAE`的工作模式分为5种：

- `-c` 检查是否能模拟
- `-a` 漏洞分析
- `-r` 固件模拟的运行
- `-d` 用户级的调试
- `-b` 内核级的调试

```bash
sudo ./run.sh -c <brand> <firmware>
sudo ./run.sh -a <brand> <firmware>
sudo ./run.sh -r <brand> <firmware>
sudo ./run.sh -d <brand> <firmware>
sudo ./run.sh -b <brand> <firmware>
```

`!<firmware>`是固件路径，`<brand>`是固件的牌子，这个可以随意写。
`！在对固件模拟前，尽量先-c进行检查。在检查之后，运行和调试时的模拟速度也会变快。`

使用`FirmAE`模拟固件命令如下：

```bash
！执行init.sh初始环境
sudo ./init.sh
！检查是否能模拟
sudo ./run.sh -c tp-link '/home/iot/Desktop/WR740/wr740nv1\_en\_3\_12\_4\_up.bin'
！开始固件模拟
sudo ./run.sh -r tp-link '/home/iot/Desktop/WR740/wr740nv1\_en\_3\_12\_4\_up.bin'
```

![](upload/attach/202307/882513\_7TZMQC9QRHE9HD8.png)

通过浏览器访问URL：<mark class="encrypted">2ffK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0q4Q4x3X3f1I4</mark> 出现上图信息，说明模拟成功。开始运行固件模拟并验证后门漏洞。

![](upload/attach/202307/882513\_S5B6E2NDQJDCHFA.png)

固件开始运行，此时访问URL要求输入账号密码。

![](upload/attach/202307/882513\_V4B4W9GA2SZ4ME4.png)

根据提示输入默认的账号密码`admin`。并访问URL：<mark class="encrypted">569K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0q4Q4x3X3f1I4i4K6u0r3N6i4y4W2M7W2u0H3L8f1&6S2N6p5c8W2j5Y4g2Y4f1Y4m8E0x3U0j5#2x3U0f1#2y4e0N6Q4x3V1k6D9K9h3&6#2P5q4)9#2k6X3y4E0k6r3I4A6L8X3g2Q4x3X3g2Z5N6r3#2D9</mark> 出现以下界面：

![](upload/attach/202307/882513\_4TZHTSM9CAD3UC7.png)

尝试直接输入命令执行：

![](upload/attach/202307/882513\_H3Z6R7UY5YN59DH.png)

根据返回结果可知是需要输入账号密码的。输入固件分析时找到的用户名：`osteam`和密码：`5up`并执行命令。

![](upload/attach/202307/882513\_Y8UT59MN2KC3FGR.png)

由上可知命令被执行并返回。并且在右侧还预设了一些命令，点击按钮即可执行，这个后门很优雅。

![](upload/attach/202307/882513\_9SFWC3V7RPM7AYB.png)

## 0x04 复现总结

熟悉工具的使用和有一些分析常识很重要，通过信息收集到的敏感信息可以快速定位漏洞可能出现的位置并针对性分析，是一个由粗到细的过程。这是复现的第一个路由器漏洞，希望自己能坚持下去，用心复现，仔细记录。“靡不有初，鲜克有终！”

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#固件分析](https://bbs.kanxue.com/forum-128-1-170.htm)

[#漏洞分析](https://bbs.kanxue.com/forum-128-1-171.htm)

[#家用设备](https://bbs.kanxue.com/forum-128-1-173.htm)

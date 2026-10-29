
# 【原创:android分析利器】santoku介绍和安装指南

以下都是原创，转载请注明出处

作者：Colbert仔

## santoku介绍

`santoku`是一个移动安全渗透测试系统，咱们实验室叫日本菜刀，区别于中国菜刀哈：），这个系统基于`ubuntu12.04-64bit`，里面有很多常用的渗透测试工具，大家可以把它理解为终端安全版的`BT5`，具有以下特点：

1. 集成了大量主流的`Android`程序分析工具，比如`apktool`、`dex2jar`、`androguard`、`baksmali`等，为分析人员节省分析环境配置所需的时间。
2. 集成渗透测试工具。比如`Burpsuite`、`nmap`、`mercury`
3. 集成网络数据分析工具。比如`wireshark`

具体集成的工具清单如下：

### A、开发工具

`Android SDK Manager`

`AXMLPrinter2`

`Fastboot`

`Heimdall (src | howto)`

`Heimdall (GUI) (src | howto)`

`SBF Flash`

### B、渗透测试工具

`Burp Suite`

`Ettercap`

`Mercury`

`nmap`

`OWASP ZAP`

`SSL Strip`

`w3af (Console)`

`w3af (GUI)`

`Zenmap (As Root)`

### C、无线分析

`Chaosreader`

`dnschef`

`DSniff`

`TCPDUMP`

`Wireshark`

`Wireshark (As Root)`

### D、设备取证

`AFLogical Open Source Edition (src | howto)`

`Android Brute Force Encryption (src | howto)`

`ExifTool`

`iPhone Backup Analyzer (GUI) (src | howto)`

`libimobiledevice (src | howto)`

`scalpel`

`Sleuth Kit`

## santoku安装方法

1. 登陆`santoku`的官方网站：`8afK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6j5h3&6@1L8$3E0#2i4K6u0V1L8r3W2F1N6i4S2Q4x3X3g2U0L8$3#2Q4x3V1j5`.
2. 点击`download`，你就会得到`santoku`的`ISO`安装包，目前最新版本是`santoku 0.4`
3. 安装`vm10`
4. 这个时候要注意你电脑的`CPU`是否支持`VT(CPU`虚拟化技术)，因为这个`santoku`是`64`位系统，如果你的`CPU`不支持`VT`，也不是`64`位系统的话，那你在虚拟机安装过程中虚拟机会挂死。如何检测`CPU`是否支持`VT`呢，我这里提供一个工具叫`leomoon-cpu-v`，下载地址是：`2bdK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4W2L8$3#2G2L8$3&6Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6r3q4@1K9h3y4Q4x3V1k6G2N6r3S2W2M7W2)9J5c8X3I4W2L8$3#2G2L8$3&6Q4x3X3c8U0M7s2g2Q4x3X3c8$3i4K6u0W2P5X3W2H3`
5. 如果你的`CPU`支持`VT`，那么恭喜你，你就可以安装`santoku`了，但需要注意的是，如果你的系统是`32`位如`XP`，还是有问题的，需要在`BIOS`中设置开启`VT`，这样你的`32`位系统才可以虚拟`64`位系统。
6. 安装步骤很简单，输入计算机名、用户名、密码等即可
7. 建议进入系统后，更新一下插件，并且在命令行模式输入：

```
sudo passwd root
```

来修改`root`密码，然后输入`su`，提权成功后，下一次登陆就可以用`root`用户登陆了。到此为止你的`santoku`就安装成功了，好好享受吧！

## 后续

如果你是`android`逆向爱好者，那么有个比较好用的工具叫`androguard`就不要错过了，是一个`android`静态分析软件，具有强大的静态分析功能，其中有个功能能通过运行脚本`./androgexf.py`生成`gexf`格式的`apk`的方法调用图，并用`Gephi`软件分析，会比较高端大气上档次，俗称“蜘蛛图”分析法，哈哈。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

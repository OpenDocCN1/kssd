
# [原创]tenda系列设备用户态仿真搭建

Tenda AC18为例

下载链接：<mark class="encrypted">3c2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4N6F1i4K6u0W2N6r3g2F1k6r3q4Q4x3X3g2U0L8$3#2Q4x3X3g2U0L8W2)9J5c8Y4g2H3L8r3!0S2k6r3k6A6L8r3g2Q4x3V1k6m8b7K6p5^5i4K6u0r3j5h3x3I4z5q4)9#2k6X3E0X3i4K6g2X3g2U0p5#2i4K6u0W2x3o6y4Q4x3X3f1H3y4g2)9J5k6e0p5&6i4K6t1^5y4U0x3I4z5q4)9#2k6W2)9J5z5g2)9#2k6X3y4F1i4K6u0W2P5X3W2H3</mark>

1、正常对固件进行解包，查看固件的架构，此AX18是arm小端，因此在文件系统下执行下列命令

```
cp $(which qemu-arm-static) ./
```

2、使用下列命令进行用户态仿真

```
sudo chroot ./ ./qemu-arm-static ./bin/httpd
```

此时处出现下列情况，导致一直处于等待状态，可以使用pacth解决这个问题。

![图片描述](upload/attach/202303/973521\_8N8TU9A5HNWZQ6T.png)

解决方法

使用IDA pro打开固件，使用字符串搜索shift + F12，定位字符串出现的位置。

![图片描述](upload/attach/202303/973521\_PAFUJM7NZT4BCWM.png)

可以看到，原包中的`httpd`的运行流程，基本块1-->基本块2-->基本块3-->基本块2...一直在循环，导致处于静止状态，在基本块2的最后一条指令跳转的时候，它根据上一条指令`CMP R3,#0`的结果进行跳转，所以要想基本块2流向基本块4，就需要是用keypetch进行将该指令进行patch，修改为`CMP R3,#1`，

![图片描述](upload/attach/202303/973521\_W7H8U77RNF7EYA9.png)

![图片描述](upload/attach/202303/973521\_R9PGQ34EB6JTMG8.png)

将patch后的`httpd`替换原来的`httpd`，在此进行运行,ha? 运行之后还是同样的结果

![图片描述](upload/attach/202303/973521\_AF5DG26P65DQFDJ.png)

那就只好动态调试，看看程序是如何跳转的。现在`CMP R3,#1`处打上断点。在Ubuntu上开启远程调试端口。

```
sudo chroot ./ ./qemu-arm-static -g 1234 ./bin/httpd
```

之后在IDA pro配置选项如下图：

![图片描述](upload/attach/202303/973521\_T98G4TXSMTE84B7.png)

当程序运行到断点的时候，在未执行`CMP R3.#1`时候，`R3`的值为1，使用F8进行执行，可看到程序还是流向了基本块3

![图片描述](upload/attach/202303/973521\_3WM5W6W8WTE3RSR.png)

最直接的解决方法就是，将基本块3的最后一个跳转指令`nop`掉，

![图片描述](upload/attach/202303/973521\_C5BFNF2S65C2BG7.png)

继续运行，如下图结果

![图片描述](upload/attach/202303/973521\_ZR38HJKE76HC6UR.png)

终于不在处于等待状态，接下来接着定位问题出现的位置，在IDA Pro中定位该字符串。如下图

![图片描述](upload/attach/202303/973521\_QXN9NC7HUATFQKJ.png)

如图显示，程序流向为基本块a-->基本块b，导致出现错误，正常应该是基本块a-->基本块c，所以这里patch指令`CMP R3,#0`，修改为`CMP R3,#0`.如下图所示

![图片描述](upload/attach/202303/973521\_QEB23XE2VM3HZ2M.png)

继续执行，如下所示：

![图片描述](upload/attach/202303/973521\_4SG4NBF5DM5J5X7.png)

可以看到程序正常执行，但是此时的IP地址不正确，解决方法如下两条命令

```
sudo tunctl -t br0 -u root
sudo ifconfig br0 192.168.1.1/24 up
```

![图片描述](upload/attach/202303/973521\_2NVM53EQNTBZF3Y.png)

程序正常运行

![图片描述](upload/attach/202303/973521\_Y7NZ7EC4MGM95VE.png)

还有另外一种情况，程序可以正常运行，并且可以访问该IP地址，但是访问的页面显示错误，此时需要在文件系统中修改文件的软连接。

![图片描述](upload/attach/202303/973521\_BDF9VDR4C4TSE8X.png)

此时`webroot`链接到`var`文件中的文件，此时就需要包该`weboot`文件删除，修改链接文件的地址。

```
rm -rf webroot
ln -s webroot_ro/ webroot
```

之后就可以看到该页面的正常显示。

Tenda AC18为例

下载链接：b4aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4N6F1i4K6u0W2N6r3g2F1k6r3q4Q4x3X3g2U0L8$3#2Q4x3X3g2U0L8W2)9J5c8Y4g2H3L8r3!0S2k6r3k6A6L8r3g2Q4x3V1k6m8b7K6p5^5i4K6u0r3j5h3x3I4z5q4)9#2k6X3E0X3*V15.03.05.19(6318*)\_cn.zip

1、正常对固件进行解包，查看固件的架构，此AX18是arm小端，因此在文件系统下执行下列命令

```
cp $(which qemu-arm-static) ./
```

2、使用下列命令进行用户态仿真

```
sudo chroot ./ ./qemu-arm-static ./bin/httpd
```

此时处出现下列情况，导致一直处于等待状态，可以使用pacth解决这个问题。

![图片描述](https://bbs.kanxue.com/upload/attach/202303/973521_8N8TU9A5HNWZQ6T.png)

解决方法

使用IDA pro打开固件，使用字符串搜索shift + F12，定位字符串出现的位置。

![图片描述](https://bbs.kanxue.com/upload/attach/202303/973521_PAFUJM7NZT4BCWM.png)

可以看到，原包中的`httpd`的运行流程，基本块1-->基本块2-->基本块3-->基本块2...一直在循环，导致处于静止状态，在基本块2的最后一条指令跳转的时候，它根据上一条指令`CMP R3,#0`的结果进行跳转，所以要想基本块2流向基本块4，就需要是用keypetch进行将该指令进行patch，修改为`CMP R3,#1`，

![图片描述](https://bbs.kanxue.com/upload/attach/202303/973521_W7H8U77RNF7EYA9.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202303/973521_R9PGQ34EB6JTMG8.png)

将patch后的`httpd`替换原来的`httpd`，在此进行运行,ha? 运行之后还是同样的结果

![图片描述](https://bbs.kanxue.com/upload/attach/202303/973521_AF5DG26P65DQFDJ.png)

那就只好动态调试，看看程序是如何跳转的。现在`CMP R3,#1`处打上断点。在Ubuntu上开启远程调试端口。

```
sudo chroot ./ ./qemu-arm-static -g 1234 ./bin/httpd
```

之后在IDA pro配置选项如下图：

![图片描述](https://bbs.kanxue.com/upload/attach/202303/973521_T98G4TXSMTE84B7.png)

当程序运行到断点的时候，在未执行`CMP R3.#1`时候，`R3`的值为1，使用F8进行执行，可看到程序还是流向了基本块3

![图片描述](https://bbs.kanxue.com/upload/attach/202303/973521_3WM5W6W8WTE3RSR.png)

最直接的解决方法就是，将基本块3的最后一个跳转指令`nop`掉，

![图片描述](https://bbs.kanxue.com/upload/attach/202303/973521_C5BFNF2S65C2BG7.png)

继续运行，如下图结果

![图片描述](https://bbs.kanxue.com/upload/attach/202303/973521_ZR38HJKE76HC6UR.png)

终于不在处于等待状态，接下来接着定位问题出现的位置，在IDA Pro中定位该字符串。如下图

![图片描述](https://bbs.kanxue.com/upload/attach/202303/973521_QXN9NC7HUATFQKJ.png)

如图显示，程序流向为基本块a-->基本块b，导致出现错误，正常应该是基本块a-->基本块c，所以这里patch指令`CMP R3,#0`，修改为`CMP R3,#0`.如下图所示

![图片描述](https://bbs.kanxue.com/upload/attach/202303/973521_QEB23XE2VM3HZ2M.png)

继续执行，如下所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202303/973521_4SG4NBF5DM5J5X7.png)

可以看到程序正常执行，但是此时的IP地址不正确，解决方法如下两条命令

```
sudo tunctl -t br0 -u root
sudo ifconfig br0 192.168.1.1/24 up
```

![图片描述](https://bbs.kanxue.com/upload/attach/202303/973521_2NVM53EQNTBZF3Y.png)

程序正常运行

![图片描述](https://bbs.kanxue.com/upload/attach/202303/973521_Y7NZ7EC4MGM95VE.png)

还有另外一种情况，程序可以正常运行，并且可以访问该IP地址，但是访问的页面显示错误，此时需要在文件系统中修改文件的软连接。

![图片描述](https://bbs.kanxue.com/upload/attach/202303/973521_BDF9VDR4C4TSE8X.png)

此时`webroot`链接到`var`文件中的文件，此时就需要包该`weboot`文件删除，修改链接文件的地址。

```
rm -rf webroot
ln -s webroot_ro/ webroot
```

之后就可以看到该页面的正常显示。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2023-3-4 21:48

被schm编辑

，原因： 图片放错，截取的图片有重影，是因为之前把该内容发错了版块上，在保留的文件中截取的，所以图片会有重影

[#技术分享](https://bbs.kanxue.com/forum-128-1-168.htm)

[#固件分析](https://bbs.kanxue.com/forum-128-1-170.htm)

[#漏洞分析](https://bbs.kanxue.com/forum-128-1-171.htm)

[#漏洞挖掘](https://bbs.kanxue.com/forum-128-1-178.htm)

[#工控设备](https://bbs.kanxue.com/forum-128-1-174.htm)

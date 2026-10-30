
# [原创]Tenda AC6 实机拆解与UART串口的区分方法

## 前言

本文介绍Tenda AC6路由器实机拆解，区分UART串口以及使用USB to TTL、JTAGulator等工具进行测试和连接

具体型号如下所示

![图片描述](upload/attach/202602/1005607_SGG38G37U3CRFFB.webp)

拆开Tenda AC6在下方可以看到四个连在一起的孔就是我们的UART口

![图片描述](upload/attach/202602/1005607_8DEYF9PXD37K7TR.webp)

下面介绍两种方法用于区分UART的串口

## 方法一:万用表测量

- 将万用表调节到蜂鸣档(在电阻档下面)
- 将黑色笔接在PCB板的金属块位置(可以是任意导电部位，我选择的是锡焊电线的地方)
- 红色笔在4个PIN之间移动，如果电表发出响声，则认为该pin脚为`GND`

以图中为例，左侧第一个孔响而其他孔不响，所以左侧第一个孔为`GND`串口

之后使用适配电源对路由器PCB板进行通电

![图片描述](upload/attach/202602/1005607_WXUTFUHQ6MREMDZ.webp)

下面开始对`RX`和`TX`以及`VCC`口的测试

- 将万用表调节到20v直流电压档位
- 将黑色笔放在刚刚测出的`GND`上
- 然后将红色笔一个个放在不同的孔上进行测试，电压最大的就是`VCC`孔，电压一直变化的则是`TX`孔

但是这里出现了一个问题，不知道是不是笔者的姿势不对或者别的原因，笔者测出的结果是从左2到左4的结果依次是 3.26v、3.27v、3.28v，并且笔者每个孔都放置了将近1min，电压没有任何变化，以此导致笔者无法区分出其他串口

下面是其他区分方法

- 如果其中一个引脚的焊接处呈现正方形，则该引脚是`VCC`
- 通过万用表测试`RX`和`VCC`的电压，一般而言，`VCC`的电压会稍高几毫伏
- 在设备启动过程中，`VCC`的电压恒定不会波动，而`RX`会有几毫伏的波动

笔者猜测左4的正方形应该就是`VCC`口，当然这样依旧没有办法完全确认，于是笔者用了个损招，`直接使用杜邦线将GND和各个串口短接，如果是VCC口，那么路由器就会短路然后触发重启`，但是为什么是损招，因为笔者之前对着小米路由器用的时候发现居然冒出了火花，但是这款Tenda路由器并没有这么强烈的反应，不知道会不会对路由器造成什么损害，但是总归是确认了那个方框就是`VCC`

然后就需要区分`TX`和`RX`

- 将万用表调节到20v直流电压
- 黑色笔放在`GND`上
- 红色笔放在不同的孔上，然后重启路由器，电压稳定的是`RX`，电压变化的是`TX`

用这个方法可以测出左2是`RX`,左3是`TX`,还是比较明显的，左2的电压一直在3.27-3.28，但是左3的电压在3.28-1.5之间不断变化。

下面使用 `FT232 USB to TTL` 测一下验证一下结论

如下所示，使用杜邦线将USB to TTL的`RXD`和`TXD`连接起来，然后使用`SSCOM`来测试一下这个USB to TTL是否完好

显然测试结果是它是好的

![图片描述](upload/attach/202602/1005607_85G5BEUVAPZ5W9W.webp)

![图片描述](upload/attach/202602/1005607_MWQJUJT46ST57XK.webp)

然后我们使用它来对我们的串口做一个简单的测试

注意USB to TTL的`TX`应该要接在路由器的`RX`上，也就是左2，那么USB to TTL的`RX`就要接在左3上，然后`GND`对应`GND`，`VCC`不需要连接，具体如下所示(笔者没有焊接，直接使用杜邦线连接，只要插得紧和焊接效果应该是一样，如果插不紧那就只能焊接一下了)

![图片描述](upload/attach/202602/1005607_Y9WJYPUSC2MHDSX.webp)

然后使用`SSCOM`软件进行连接，这里多次实验后得知笔者手头这款Tenda AC6路由器的波特率是 `38400`

每次修改波特率之后需要重启一下路由器，最后没有乱码，并且最后出现`CIL`就意味着成功了

下面是重启路由器后输出的内容

![图片描述](upload/attach/202602/1005607_QDZCCW8VX67YAXQ.webp)

出现`CLI`输入`?`就可以得知路由器中有哪些命令了

![图片描述](upload/attach/202602/1005607_85YX55UUBBYVCED.webp)

使用`SecureCRT`的界面更好看一点（两个不能同时用，一个占用端口另外一个就用不了了）

![图片描述](upload/attach/202602/1005607_P3XP57CRDZUM772.webp)

```
CLI> ?
Realtek's command:
Commands Descriptions Usage
-------- ------------ -----
db db <Address> <Len>
dw dw <Address> <Len>
eth eth
wlan0 wlan0
wlan0-va0 wlan0-va0
wlan0-va1 wlan0-va1
wlan0-va2 wlan0-va2
wlan0-va3 wlan0-va3
wlan0-vxd wlan0-vxd
wlan1 wlan1
wlan1-va0 wlan1-va0
wlan1-va1 wlan1-va1
wlan0-va2 wlan0-va2
wlan1-va3 wlan1-va3
wlan1-vxd wlan1-vxd
eb eb <Address> <Value1> <Value2>...
ew ew <Address> <Value1> <Value2>...
alg alg
brconfig brconfig
cat
cpuload cpuload
date show system local time
excep cause exception
fastpath fastpath
getmib getmib
help Displays a list of commands
? Displays a list of commands
hexcat
dump Shows a memory dump
hw_nat hw_nat
ib ib
idd idd
ifconfig ifconfig
ipfw ipfw
irf irf
iw iw
iwpriv iwpriv
ll
mac mac <ifname> [mac addr]
mroute mroute
ob ob
od od
orf orf
ow ow
pdump dump a thread
ping ping
port_fwd port_fwd
version Shows build version
ps Shows a list of threads
reboot Reset the system
reinit reinit
reinit_test reinit_test
reset Reset the system
route set static route
rssi rssi
alias show alias table
rtl_vlan rtl_vlan
sp Sets a threads priority [thread ID]
setmib setmib
show show
tcpstats tcpstats
test_skb test_skb
kill Kills a running thread [thread ID]
release Break a thread out of any wait [thread ID]
trigger_port trigger_port
uptime Shows system uptime
watchdog on/off/res/reboot
Tenda's command:
msg ce_power app_debug build stat_link link_status
iwpriv realtek splx thread mbuf debug
route et wlconf fw ping ifconfig
syslog time tenda_arp arp envram nvram
restart reboot
CLI>
```

## 方法二:使用JTAGulator来识别

如下所示，`GND`连接`GND`,假装我们未知`TX`和`RX`随便连接两个接口，`VCC`不进行连接

![图片描述](upload/attach/202602/1005607_SH4NTFD6S93SUMD.webp)

同样使用`SecureCRT`进行连接

然后选择V设定电压为3.3v

![图片描述](upload/attach/202602/1005607_9ZNF5SK5XYU9WF8.webp)

然后使用U进行检测

![图片描述](upload/attach/202602/1005607_755YNY48HSUCZV7.webp)

在输出的结果里面可以看到当7号接口(路由器的左3)为`TX`，6号接口(路由器的左2)为`RX`，并且波特率为`38400`的时候可以输出有意义的字符串 `Unknown`

因此我们可以知道路由器的左2为`RX`，左3为`TX`,并且波特率为 `38400`

![图片描述](upload/attach/202602/1005607_5A2JPR4PW3QG4DF.webp)

显然`JTAGulator`的优点就是非常方便，只需要测出`GND`然后其他随便连接在接口上就可以了开始测试了

缺点是贵

参考:

<a href="elink@b7bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6C8j5h3k6J5L8$3y4Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8#2)9J5c8U0t1H3x3U0u0Q4x3V1j5H3x3W2)9J5c8U0p5I4i4K6u0r3g2f1q4d9g2q4)9J5y4f1f1@1i4K6t1#2b7U0S2Q4x3U0g2n7x3W2)9J5y4f1f1#2i4K6t1#2z5p5k6Q4x3U0g2m8x3#2)9J5y4f1f1%4i4K6t1#2b7e0q4Q4x3U0f1^5x3q4)9J5y4f1f1#2i4K6t1#2z5p5k6Q4x3U0f1^5b7g2)9J5y4f1f1&6i4K6t1#2z5o6m8Q4x3U0f1&6b7g2)9J5y4f1f1^5i4K6t1#2b7V1k6Q4x3U0f1^5y4#2)9J5y4f1f1@1i4K6t1#2b7U0S2Q4x3U0f1^5y4#2)9J5y4f1f1%4i4K6t1#2z5e0c8Q4x3U0g2m8z5q4)9J5y4f1f1^5i4K6t1#2b7e0q4Q4x3U0g2m8z5q4)9J5y4f1f1#2i4K6t1#2b7f1g2Q4x3U0f1&6b7g2)9J5y4f1f1@1i4K6t1#2b7V1c8Q4x3U0f1^5c8q4g2m8f1W2c8Q4x3U0g2q4y4g2)9J5y4f1u0o6i4K6t1#2z5e0g2Q4x3U0g2q4z5q4)9J5y4e0R3@1i4K6t1#2z5f1q4Q4x3X3g2Z5N6r3#2D9">UART串口基础及通过万用表定位UART引脚 | kafroc</a>


# [原创]零成本入门车联网安全研究

对车联网安全感兴趣，但却没有实车拿来练手，是一件很可惜的事情。为了解决这个问题，笔者结合自己的车联网安全实践经验，为大家搭建一个高度接近真实车辆的实验环境。这个环境使用了目前流行的多域架构，实现了包括`VLAN`、防火墙等真实的安全功能，供大家学习和参考。

## 域架构

![](https://bbs.kanxue.com/upload/attach/202111/508256_S6J8JPYAFF2G7SN.jpg)

对域架构的解释：

1）本实验环境实现了两个经典的通信域`VCU`和`HU`，`VCU`可以理解为整车控制域，而`HU`则是信息娱乐域，两个域通过以太网进行通信；
2）`VCU`域实现了车身`CAN`（`BDCAN`）和自动驾驶`CAN`（`ASCAN`），这两路`CAN`通过网关相连，用来模拟车身`CAN`报文发送至自动驾驶模块；
3）`HU`域实现了仪表`CAN`（`ICCAN`）与摄像头`CAN`（`CAMCAN`），它们分别与`BDCAN`、`ASCAN`通过以太网相连，第一路连接（`BDCAN -> ICCAN`）模拟车身数据在仪表盘上显示，第二路连接（`CAMCAN -> ASCAN`）模拟摄像头采集的数据传送给自动驾驶模块；
4）第一路连接与第二路连接通过`VLAN`进行隔离。

## 车辆功能

通过`ICSim`[1]模拟车辆功能，`ICSim`是一个开源的车辆仪表模拟器，该模拟器包含`controls`和`ICSim`两个模块，其中`controls`负责生成模拟的车辆数据，以`CAN`报文的方式发送给虚拟的`CAN`接口，`ICSim`从虚拟`CAN`接口读取`CAN`报文，并在仪表上更新对应零件的状态，如车速、车门状态等等。

![](https://bbs.kanxue.com/upload/attach/202111/508256_XHHFPMWVHY5WTAA.jpg)

（`ICSim`仪表和操作界面）

在我们这个架构中，`ICSim`和`controls`分别连接到两路不同的`CAN`上，其中`ICSim`连接到`HU`域的`ICCAN`，代表真实的仪表盘；`controls`连接到`VCU`域的`BDCAN`，代表真实的车身部件。`VCU`域与`HU`域通过以太网进行连接，`controls`产生的`CAN`报文通过以太网传送给`ICSim`。这种实现与真实的整车域架构及其接近，因为有些车型的仪表确实被划分到了`HU`域，以太网作为`VCU`与`HU`的通信协议也是当下比较流行的域架构。

通过如下方式创建`vcanic`和`vcanbd`：

```
sudo ip link add name vcanic type vcan
sudo ip link set dev vcanic up
sudo ip link add name vcanbd type vcan
sudo ip link set dev vcanbd up
```

接下来将`ICSim`与`HU`域的`vcanic`绑定，`controls`与`VCU`域的`vcanbd`绑定：

```
./icsim vcanic
./controls vcanbd
```

此时，代表车身零部件的`controls`产生的报文还不能顺利达到`ICSim`，因为`vcanic`与`vcanbd`两路`CAN`总线目前还未联通，我们将通过以太网的方式将这两路`CAN`连接起来，同时也将意味着`VCU`与`HU`的联通。

## CAN-ETH通信

以太网的实现使用开源项目`cannelloni`[2]，`cannelloni`的核心原理是通过`UDP`协议传送`CAN`报文，如下图所示：

![](https://bbs.kanxue.com/upload/attach/202111/508256_5QMW7YF4W5SBUZ8.jpg)

（图片来自论文《Mapping CAN-to-ethernet communication channels within virtualized embedded environments》）

因为我们这个实验是在一台机器上进行的，所以我们将通过不同的端口区分`VCU`和`HU`这两个独立的通信域，结合划分广播域的需求，我们将通过`vconfig`创建带有`vlan id`的虚拟网卡。

## VLAN划分广播域

考虑到通过以太网传输的报文类型包含了安全性要求比较高的`ADAS`（自动驾驶）报文，我们的想法是根据报文的不同安全等级来划分广播域，将普通控制报文和自动驾驶报文通过`VLAN`划分到不同的网络，相互之间不能访问。

![](https://bbs.kanxue.com/upload/attach/202111/508256_93BUZEM7MQSAAMK.jpg)

首先需要在真实的网卡上创建带有`vlan id`的虚拟网卡：

```
$ sudo vconfig add enp0s31f6 20
$ sudo vconfig add enp0s31f6 30
```

创建结果如下：

```
$ ip link
5: enp0s31f6.20@enp0s31f6: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 70:85:c2:7f:31:ef brd ff:ff:ff:ff:ff:ff
6: enp0s31f6.30@enp0s31f6: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 70:85:c2:7f:31:ef brd ff:ff:ff:ff:ff:ff
```

给网卡添加`ip`地址并启动网卡：

```
$ sudo ifconfig enp0s31f6.20 192.168.20.100 netmask 255.255.255.0 broadcast 192.168.20.255 up
$ sudo ifconfig enp0s31f6.30 192.168.30.100 netmask 255.255.255.0 broadcast 192.168.30.255 up
```

启动成功之后的效果如下：

```
$ ifconfig
enp0s31f6.20: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.20.100  netmask 255.255.255.0  broadcast 192.168.20.255
        inet6 fe80::7285:c2ff:fe7f:31ef  prefixlen 64  scopeid 0x20<link>
        ether 70:85:c2:7f:31:ef  txqueuelen 1000  (Ethernet)
enp0s31f6.30: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.30.100  netmask 255.255.255.0  broadcast 192.168.30.255
        inet6 fe80::7285:c2ff:fe7f:31ef  prefixlen 64  scopeid 0x20<link>
        ether 70:85:c2:7f:31:ef  txqueuelen 1000  (Ethernet)
```

测试以下两个`VLAN`的连通性：

```
$ ping 192.168.20.100 -I enp0s31f6.30
PING 192.168.20.100 (192.168.20.100) from 192.168.30.100 enp0s31f6.30: 56(84) bytes of data.
^C
--- 192.168.20.100 ping statistics ---
70 packets transmitted, 0 received, 100% packet loss, time 70637ms

$ ping 192.168.30.100 -I enp0s31f6.20
PING 192.168.30.100 (192.168.30.100) from 192.168.20.100 enp0s31f6.20: 56(84) bytes of data.
^C
--- 192.168.30.100 ping statistics ---
5 packets transmitted, 0 received, 100% packet loss, time 4100ms
```

`VLAN`划分成功，接着我们使用`cannelloni`将`BDCAN`（`vcanbd`）和`ICCAN`（`vcanic`）连接起来：

```
$ cannelloni -I vcanbd -R 192.168.20.100 -r 20001 -l 20000
INFO:udpthread.cpp[INFO:146]:run:UDPThread up and running
canthread.cpp[108]:run:CANThread up and running

$ cannelloni -I vcanic -R 192.168.20.100 -r 20000 -l 20001 -p
INFO:udpthread.cpp[146]:run:UDPThread up and running
INFO:canthread.cpp[108]:run:CANThread up and running
```

连通之后通过`controls`的操作，便可将`CAN`报文发送给`ICSim`，仪表盘也可以显示状态了。通过同样的方式将`ASCAN`和`CAMCAN`连接起来，`VCU`域与`HU`域的通信就完全打通了。在实际的通信场景下，我们可能会依据类似最小权限原则来配置业务能力，比如在`VCU`与`HU`的通信场景中，我们可能会认为`HU`域只需要从`VCU`接收车身`CAN`报文的能力，而不需要发送车身`CAN`报文的能力；相反，`HU`域产生的摄像头数据，需要经以太网传送给`VCU`的`ADAS`模块，此时`HU`域可能需要的是发送摄像头报文的能力，而不需要接收能力。为了满足这种需求，我们在实验中增加了网络防火墙。

## 防火墙

通过`iptables`规则分别在`VCU`域和`HU`域建立防火墙，过滤存在风险的以太网报文，以达到如下两个目的：

1. 车身零部件产生的报文只能由`VCU`传给`HU`；
2. 摄像头采集的报文只能由`HU`传递给`VCU`。

![](https://bbs.kanxue.com/upload/attach/202111/508256_RNDSM2BGBXPZQSQ.jpg)

以`HU`为例，通过如下规则对报文的流向进行控制：

```
$ sudo iptables -A INPUT -d 192.168.20.100 -j LOG --log-prefix='[hu-normal-in]' --log-ip-options
$ sudo iptables -A INPUT -p udp -d 192.168.20.100 -s 192.168.20.100 --dport 20001 --sport 20000 -j ACCEPT
$ sudo iptables -A INPUT -d 192.168.20.100 -j DROP
```

到目前为止，`VCU`与`HU`的通信已经完全打通，并且我们加上了防火墙规则，但`VCU`内部的通信却还没有解决，因为`ADAS`模块可能也需要车身零部件的数据来做自动驾驶决策，所以我们需要通过网关将`BDCAN`与`ASCAN`连接起来。

## CAN网关功能

使用`cangw`可以实现简单的`CAN`网关功能，连接车身`CAN`（`BDCAN`）和`ADAS CAN`（`ADASCAN`），车身产生的`CAN`报文通过网关路由至`ADAS`模块。（其实`cangw`和`cannelloni`的功能结合起来已经很接近真实的`CAN`网关，真实的`CAN`网关还支持`LIN`、`Flexray`等）

![](https://bbs.kanxue.com/upload/attach/202111/508256_3DT5SRNAASHEXSF.jpg)

网关的配置方式如下：

```
$ sudo cangw -A -s vcanbd -d vcanas -e
$ sudo cangw -A -s vcanas -d vcanbd -e
```

`cangw`还支持配置报文过滤规则以及完整性校验，读者可以继续探索，摄像头模块与`ADAS`模块的功能也可继续完善。

## 参考

[1].[GitHub - zombieCraig/ICSim: Instrument Cluster Simulator](https://bbs.kanxue.com/elink@56bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6*7L8$3#2T1K9h3g2o6M7X3q4A6k6#2)9J5c8V1W2o6f1$3W2E0)

[2].[GitHub - mguentner/cannelloni: a SocketCAN over Ethernet tunnel](https://bbs.kanxue.com/elink@819K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0k6%4g2W2L8Y4c8F1k6i4u0Q4x3V1k6U0j5h3&6F1k6h3I4D9L8$3&6A6)

本文首发于本人的`CSDN`专栏“车联网安全杂货铺”，地址`d2dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3q4F1k6r3I4W2k6g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0t1I4x3U0t1^5x3K6x3H3i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1^5i4@1u0p5i4@1q4o6i4@1f1^5i4@1u0p5i4@1u0p5i4@1f1^5i4@1q4r3i4@1t1%4i4@1f1%4i4K6V1@1i4@1t1K6i4@1f1$3i4K6V1^5i4K6S2q4i4@1f1#2i4K6R3%4i4@1u0m8i4@1f1#2i4@1p5@1i4K6R3@1i4@1g2r3i4@1u0o6i4K6R3I4`

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#基础理论](https://bbs.kanxue.com/forum-128-1-165.htm)

[#安全研究](https://bbs.kanxue.com/forum-128-1-167.htm)

[#技术分享](https://bbs.kanxue.com/forum-128-1-168.htm)

[#汽车安全](https://bbs.kanxue.com/forum-128-1-175.htm)

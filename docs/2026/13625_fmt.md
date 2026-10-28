
# [原创]收服护网病毒

古有如来佛祖降伏齐天大圣，今有我来收服护网病毒。前段时间，2021年HW行动刚刚结束不久，这期间抓获了不少攻击队用来渗透的病毒木马，最近刚好有时间，分析了其中一个木马。

## 一、作恶多端被擒获

该木马是一个`linux`木马，其主要功能如下：

### 1、执行Shell命令

![](https://bbs.kanxue.com/upload/attach/202105/235623_XSETXSWM7BB9YR6.jpg)

图1 执行接受shell命令

![](https://bbs.kanxue.com/upload/attach/202105/235623_YHF57WG4Z4FQQNZ.jpg)

图2 执行shell命令

### 2、把宿主当作肉鸡，去执行DDOS攻击

![](https://bbs.kanxue.com/upload/attach/202105/235623_8WJ36U4PM3PWSK4.jpg)

图3 接受DDOS攻击命令

![](https://bbs.kanxue.com/upload/attach/202105/235623_99K3ZFXCZB73653.jpg)

图4 执行DDOS攻击

该木马可以执行如下表所示的多种类型的DDOS攻击。

|  |  |  |
| --- | --- | --- |
| 1 | `SYN_Flood` | SYN洪水攻击，是种典型的DoS (Denial of Service，拒绝服务) 攻击。效果就是服务器TCP连接资源耗尽，停止响应正常的TCP连接请求 |
| 2 | `UDP_Flood` | 利用大量UDP小包冲击[DNS服务器](https://bbs.kanxue.com/elink@8adK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6i4y4G2i4K6u0W2j5$3!0E0i4K6u0r3k6r3!0U0i4K6u0r3y4e0x3@1x3o6l9%4x3q4)9J5k6o6f1#2y4K6f1#2x3e0y4Q4x3X3g2Z5N6r3#2D9)或Radius认证服务器、[流媒体](https://bbs.kanxue.com/elink@a7aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6i4y4G2i4K6u0W2j5$3!0E0i4K6u0r3k6r3!0U0i4K6u0r3x3K6t1$3x3K6j5J5x3g2)9J5k6o6x3@1x3K6R3#2y4o6k6Q4x3X3g2Z5N6r3#2D9)视频服务器 |
| 3 | `ICMP_Flood` | 这是一种DDOS攻击，通过对其目标发送超过65535字节的数据包，就可以令目标主机瘫痪，如果大量发送就成了洪水攻击 |
| 4 | `DNS_Flood` | 攻击者操纵大量傀儡机器，对目标发起海量的[域名](https://bbs.kanxue.com/elink@100K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9K9h3&6C8i4K6u0W2P5X3S2A6K9s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6Q4x3@1k6@1j5i4u0Y4k6i4c8Q4x3@1c8Z5N6s2c8H3i4K6t1#2x3@1q4Q4x3V1k6Q4x3V1k6A6k6r3y4Q4x3X3f1%4y4K6p5$3z5g2)9J5k6h3y4G2L8g2)9J5c8R3`.`.)查询请求 |
| 5 | `GET_Flood` | 向目标服务器发起大量的HTTP GET报文 |
| 6 | `TCP_Flood` | 针对TCP/IP协议发起的攻击，其明显特征是被攻击者的主机上存在大量的TCP连接 |
| 7 | `POST_Flood` | 向目标服务器发起大量的HTTP POST报文 |

### 3、连接服务端

该木马有两种连接服务端的方式，一种是直接连接服务端的IP地址，一种是通过域名解析动态获取服务端IP地址。

![](https://bbs.kanxue.com/upload/attach/202105/235623_UF5JP5AAXE5GN4G.jpg)

图5 直接连接IP

![](https://bbs.kanxue.com/upload/attach/202105/235623_2M3HGDH6GY3UNDG.jpg)

图6 动态解析域名连接IP

分析到这里，这个木马的行为和功能，已经了解的一清二楚了。这时候，我有了一个想法，想要收服这个木马，引导它弃暗投明变成自己的小弟，那应该怎么做呢!!!

## 二、当头棒喝终悔悟

有两个方法可以实现。第一，通过文件编辑，把木马服务端的IP改成我的IP。第二，把木马动态解析的域名改成我的域名。

### 1、这里我选择第一种方法，来收服这个木马小弟。

![](https://bbs.kanxue.com/upload/attach/202105/235623_N58DHEQ4MYMQBX4.jpg)

图7 修改IP

![](https://bbs.kanxue.com/upload/attach/202105/235623_CBR3ZWTMD8QE3VA.jpg)

图8 修改IP

### 2、写代码，创建一个服务端等待木马连接

![](https://bbs.kanxue.com/upload/attach/202105/235623_JA9SXRH8TC9NFJW.jpg)

图9 服务端代码

由于该木马连接服务端成功会向服务端发送宿主机的一些信息，所有启动我的服务端，运行一下木马，看看经过我一番苦心劝告的木马是否已经回头是岸，投向我的服务端。

![](https://bbs.kanxue.com/upload/attach/202105/235623_WZ54SDEZMGY65BX.jpg)

图10 向服务端发送宿主机信息

![](https://bbs.kanxue.com/upload/attach/202105/235623_KJHFU7F9U9QUMZU.jpg)

图11 连上我的服务端

如上图所示，木马已经成功连上我的服务端，并发送宿主机的版本信息，看来经过我的一番苦口婆心的劝说，这个木马已经幡然悔悟并投向了我的怀抱，并等待着我的下一步指引。

尝试给木马发送一条查看当前目录的指令，看看这个木马是否会执行我们的指令。

![](https://bbs.kanxue.com/upload/attach/202105/235623_EYJY38DR34AF6JN.jpg)

图12 发送指令

通过IDA调试,确定木马已经收到了服务端发送的指令。

![](https://bbs.kanxue.com/upload/attach/202105/235623_Q4BW97TSBJZNQ9Y.jpg)

图13 收到的指令

并且在虚拟机中已经成功执行了我们发送的指令。

![](https://bbs.kanxue.com/upload/attach/202105/235623_MNP6MUWKEK73BRG.jpg)

图14 执行指令

到此，可以肯定，这个木马经过我的谆谆教诲，真的已经弃暗投明，痛改前非了。就让它跟着我迎接下一次的护网行动，救赎它以前犯下的过错。

另外，它前主人的服务器开了3389，是否还有后续故事，敬请期待。

![](https://bbs.kanxue.com/upload/attach/202105/235623_SKU5DYSDQT4VN5F.jpg)

图15 木马服务端开启了3389

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)


# [原创]Wireshark实战之局域网监听

看书之余闲来无事对局域网检测试试,看看室友们都在干什么.

说干就干,想想有什么办法可以进行局域网监听,要么Arp欺骗,要么MITM中间人攻击.Arp的话会影响网速,引起室友警惕,我可不想引起公愤.想了想还是MITM吧.中间人攻击是一种“间接”的入侵攻击，这种攻击模式是通过各种技术手段将受入侵者控制的一台计算机虚拟放置在网络连接中的两台通信计算机之间，这台计算机就称为“中间人”攻击.

既然想好了对策,那么就直接上手网络工程师的最爱Wireshark.它是一款本地监听数据的大杀器,弊端是只能监听本地的数据,有什么办法可以让局域网中的流量都从本机走呢?第一ARP嗅探,劫持网关,再本地抓包.

第二交换机镜像端口,在路由或者交换机处设置,复制一份数据到指定机子端口.

第三Ap一个无线,让他们连接,然后再抓包.

第一种方法不用考虑,第二种方法因为企业级路由才有镜像端口功能,我等屌丝用的TP-LINK无这个功能.所以只能考虑第三种方法.

首先建立一个WIFI热点,懒得用WIN7自带的虚拟WIFI功能.(PS:window 7有一个“`Microsoft Virtual WiFi Miniport Adapter`”功能，我理解为“虚拟WIFI”。也就是说而Windows 7 可以在一块真实无线网卡基础上再虚拟出一块无线网卡，支持网络共享，让电脑变身wifi热点。) 直接用360卫士的免费WIFI建立一个和路由发射出来的信号一模一样的热点.

![](https://bbs.kanxue.com/upload/attach/201504/650812_2a3a65e458fe2702.png)

然后进入路由把原本路由的WIFI关闭.

![](https://bbs.kanxue.com/upload/attach/201504/650812_59b4433cad2ed84c.png)

WIFI机制同名热点一般情况下，应该都是信号优先，也就是哪个路由信号更强，你的手机就会先连哪个.我都把路由的的WIFI关闭了,当然是我信号最强了,坐等鱼上钩…

![](https://bbs.kanxue.com/upload/attach/201504/650812_dff5d13f0242b743.png)

OK,已经来了.抓包吧，等待登录…

![](https://bbs.kanxue.com/upload/attach/201504/650812_28a101caf10c34ea.png)

还是看雪的帐号,去MD5解密撞撞运气.

![](https://bbs.kanxue.com/upload/attach/201504/650812_d8c5f7099e2aa14c.png)

再看看有什么收获,.. 截取到几条QQ `ClientKey`

![](https://bbs.kanxue.com/upload/attach/201504/650812_5bb07b40b6e42f98.png)

再抓包查查腾讯快速登录的接口，格式如下

腾讯首页：先登录e85K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4@1f1#2i4K6R3$3i4K6S2p5i4@1f1#2i4@1p5@1i4K6S2p5i4@1f1#2i4K6R3^5i4@1t1$3i4@1f1@1i4@1t1^5i4K6S2n7i4@1f1&6i4K6W2p5i4@1p5J5i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4K6R3$3i4K6S2p5i4@1f1#2i4K6V1H3i4K6S2q4i4@1f1&6i4K6R3H3i4K6R3H3i4@1f1#2i4K6R3^5i4@1t1H3N6%4N6%4i4K6u0W2M7i4q4Q4x3X3g2U0L8$3@1`.

c77K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8@1L8r3!0Y4K9h3^5J5i4K6u0W2M7i4q4Q4x3X3g2U0L8$3#2Q4x3V1k6B7N6h3#2H3i4K6y4r3j5$3I4A6k6h3&6@1N6h3W2F1i4K6y4p5f1g2q4Q4c8e0g2Q4z5p5k6Q4b7U0N6Q4c8e0N6Q4b7e0m8Q4z5o6q4Q4x3U0k6U0L8r3W2W2L8Y4c8C8k6i4W2Q4x3@1c8C8k6i4W2Q4c8e0N6Q4b7e0m8Q4z5o6q4Q4x3U0k6C8k6i4W2A6L8X3c8W2P5q4)9K6c8o6W2Q4x3U0k6H3N6q4)9#2k6X3q4A6k6q4)9K6c8o6j5K6y4U0l9I4y4o6t1H3x3g2)9J5y4Y4f1I4i4K6y4p5K9s2c8@1M7q4)9J5y4e0y4m8i4K6t1#2x3V1k6Q4x3U0f1J5c8Y4N6%4N6#2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6t1#2x3V1k6I4M7e0t1H3x3e0u0Q4x3U0f1J5c8X3I4G2k6$3W2F1f1%4g2U0j5$3g2K6M7#2)9J5k6h3S2@1L8b7`.`.

直接用抓到的室友的QQ号和KEY替换，登录上去

![](https://bbs.kanxue.com/upload/attach/201504/650812_6d2b7d22d12abcf8.png)

![](https://bbs.kanxue.com/upload/attach/201504/650812_090e252946febc03.png)

好了,就到这吧,自娱自乐就可以了.玩过火了被室友海扁的….

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

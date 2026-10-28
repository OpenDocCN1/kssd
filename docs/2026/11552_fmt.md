
# [旧帖][原创]分组网间探测工具0.00雪花

最近在学习`SOCKET`于是就找了个练习来做。最后就产生了这个分组网间探测工具。希望能得到邀请码！

~分组探测工具其实说白了就是把系统自带的`PING`和`TRACERT`自己组装在一起成为一个独立的程序。本程序`VS2008`编写。下面说下原理~

`PING`的原理就是像目标地址发送一系列的`ICMP`数据包，并等待对方的回应。程序接收后处理显示。

`TRACERT`的原理就是发送`UDP`数据包，通过逐渐递增`TTL`的值让它走得更远，最后知道到达目标`IP`的一个无用端口，这是对方就会返回一个说明该端口无程序等待的`ICMP`数据包。然后程序接收到该数据包后处理并显示。

最后把两个功能整合到一个程序里。~~就不用再面对原来的`DOS`窗口了。~

下面是程序和源代码~~~希望能够得到邀请码~毕竟临时会员很多资料下载不了~~呵呵

![](https://bbs.kanxue.com/upload/attach/200908/262544_81wgdt3ivecnt8m.gif)

![](https://bbs.kanxue.com/upload/attach/200908/262544_e4nkyfauex7lwgw.gif)

[IP测试工具.rar](https://bbs.kanxue.com/attach-download-30202.htm)

[yuandaima.zip](https://bbs.kanxue.com/attach-download-30203.htm)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

## 上传的附件

- [1.gif](javascript:void(0);)
  （10.60kb，213次下载）
- [2.gif](javascript:void(0);)
  （9.97kb，213次下载）
- [IP测试工具.rar](javascript:void(0);)
  （631.83kb，57次下载）
- [yuandaima.zip](javascript:void(0);)
  （81.18kb，43次下载）


# [原创]通过劫持系统`IpHlpApi.dll`伪装IP地址和MAC地址的源码

![](https://bbs.kanxue.com/view/img/face/1.gif)我们公司做web开发，禁止上QQ，禁止优酷，禁止土豆，整个内部只能用MSN和邮件沟通。但却可以访问一些黑名单以外的网站和端口，当然，也有少部分的老员工能上QQ。

在苦苦思索很久没有办法后，终于拿到了别人的MAC地址，在自己的机子上修改了一遍（全局修改），确认是路由硬件限制了MAC地址访问某些特定端口，通过 SMAC 2.7修改了MAC地址后，自己的电脑与他人保持同一个MAC地址和IP地址，但结果是QQ可以上了，MSN却挂了。

MSN似乎存在着一种验证，根据MAC地址和IP地址在服务器上进行唯一性验证，如果存在某一台MAC或IP，那么就将上一个踢除，导致其掉线。所以我登陆MSN他就掉，他登陆我就掉，严重影响工作。

因此中午特地找了时间写了下面这个劫持dll。通过劫持`IpHlpApi.dll`进行局部的MAC地址和IP地址随机修改。编译后可以拷贝到任何程序的目录下使用(有些仅通过此方式进行多开验证的游戏，应该可以pass掉)。

具体的输出结果可以看`DbgView`

代码很简单，也不新鲜，只是为了发出来与大家分享。![](https://bbs.kanxue.com/view/img/face/41.gif) VS2010

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)

上传的附件：

- [IpHlpApi-Ip&Mac Disguise Source.zip](javascript:void(0);)

  （28.58kb，1284次下载）

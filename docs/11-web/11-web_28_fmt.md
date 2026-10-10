
# 二千零一十、年 10 大 WEB 攻击技术之 Padding Oracle Attack 视频和资料

> 原文：[`www.pediy.com/kssd/pediy12/129824.html`](https://www.pediy.com/kssd/pediy12/129824.html)

BaCde：这个视频时我在 Youtub 上下载下来的，传到了优酷上。是关于 `Padding Oracle Attack` 攻击的视频。如果不了解的，大家可以百度 Google 一下相关信息。以下我简单整理了一些相关资料和工具。播放视频地址和一些资源如下：

看视频也可以到我的博客去看！

http://hi.baidu.com/jbinghe/blog/item/6585e4dc30d86cb2cc1166c1.html

http://v.youku.com/v_show/id_XMjQzNTcyNjA4.html

*

## 一些关于该攻击的资源

http://cloud.csdn.net/a/2010/1230/289154.html 2010 年度最有技术含量攻击：Padding Oracle Attack

http://usenix.org/events/woot10/tech/full_papers/Rizzo.pdf Practical Padding Oracle Attacks

http://www.gdssecurity.com/l/b/2010/09/14/automated-padding-oracle-attacks-with-padbuster/ Automated Padding Oracle Attacks with PadBuster

http://www.isg.rhul.ac.uk/~kp/secretIV.pdf Padding Oracle Attacks on CBC-mode Encryption with Secret and Random IVs

http://www.limited-entropy.com/padding-oracle-attacks Understanding Padding Oracle attacks

## 国内的 2 篇文章

http://blog.zhaojie.me/2010/10/padding-oracle-attack-in-detail.html Padding Oracle Attack 实例分析（该文为译文）

http://blog.zhaojie.me/2010/09/things-about-padding-oracle-vulnerability-in-asp-net.html 浅谈这次 ASP.NET 的 Padding Oracle Attack 相关内容

## 利用工具

`padBuster.pl` 这个就是视频里用到的那个工具了，pl 的，需要安装 perl

`Poet` 这个工具有系统的版本有 mac 下的，有 linux 64 位版的，有 linux32 位版的。也有 windows 版本的

http://netifera.com/download/poet/poet-1.0.1-linux-x86_amd64.jar linux64 位的

http://netifera.com/download/poet/poet-1.0.1-linux-x86_amd64.jar linux 下 32 位的

http://netifera.com/download/poet/poet-1.0.0-win32-x86.jar windows 下 32 位的

http://netifera.com/download/poet/Poet.dmg mac 下的

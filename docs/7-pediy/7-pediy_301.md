

# 给 PEID 增加皮肤及音乐 一看就懂了哇

> 原文：[`www.pediy.com/kssd/pediy12/132116.html`](https://www.pediy.com/kssd/pediy12/132116.html)

![](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")写的不好大家别骂人，这真的非常简单的哇，不需要脱壳也行。感觉还行  
给 PEID 增加皮肤及音乐特简单 一看就懂了哇 ![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")  
我们就用易语言吧。这软件小 好操作我平时都是网吧上网在家很无聊。在网吧下载个 Delphi 等下好都天亮了。  
  
我们先下个 易语言、及易语言内存皮肤模块、LordPE DLX 增强豪华版、在下个 XM 支持库吧。  
  
我们打开易语言 建立个 DLL 文件  
在下面写到  
  
.版本 2  
.支持库 xmlib  
  
pediy ()\_ 临时子程序 ()    
返回 (0)  '  
  
.子程序 pediy, , 公开  
  
资源皮肤 (#a)  
播放 xm ()  
  
  
然后我们打开 LordPE DLX 增强豪华版  
  
给软件增加个涵数  
DLL 名就是 刚才我们编译后的那个 DLL   
API 就是 pediy  
  
  
好了完事了。  效果还不错。 下面的是源码和演示文件 。

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || 源代码.rar  
 |
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || PEiD.0.95.rar  
 |
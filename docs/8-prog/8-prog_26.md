

# 新人科普之再论驱动开发和调试环境搭建

> 原文：[`www.pediy.com/kssd/pediy10/95671.html`](https://www.pediy.com/kssd/pediy10/95671.html)

坛子里一直有人问驱动调试环境的搭建问题，因为每个人机器的配置环境不同，配置策略自然也不一样，先给大家推荐两篇精华帖。  
    作 者: qqeleven  
      时 间: 2007-07-19,01:11  
      链 接: http://bbs.pediy.com/showthread.php?t=48220  
  
     【文章标题】: 驱动开发和调试的环境设置  
   【文章作者】: qqeleven  
     【使用工具】: VC2003,VMware,Visual SoftICE  
    【操作平台】: xp sp2  
    【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
  
     标 题: 【原创】使用 WinDBG 进行双机内核调试  
   作 者: xIkUg  
     时 间: 2006-11-10,21:16  
     链 接: http://bbs.pediy.com/showthread.php?t=34731  
    Target 环境：Virtual PC 2004、Win2000 sp 4 en  
    Host 环境：WinXP sp2, WinDBG 6.6.07.5, SUDT SerialNull 1.6 试用版  
     
   首先感谢两位原作者，我沾一下光先，我搭建的环境是  
   Target 环境：VMWare6.0.3、Winxp sp2  
     Host 环境：WinXP sp3, WinDBG   
    详细过程见附件，如果仍然不能成功，那 Google 就是你最后的归宿。  
  
   另外很多搞驱动的人都是以《驱动开发技术详解》作为入门书籍，我本人比较偏爱 vc6.0 的编译环境，vc 的 cl.exe 版本相对来说比较低，编译不了 xp 的驱动，我主要的驱动开发是 ddk 2k，所以模仿 easySYS 开发了一个驱动小工具，编译设置不是采用 buid.bat 的形式，而是采用《驱动开发技术详解》一书中采用的方式，很多源码直接来自 easySYS,在此一并感谢原作者，在我提供的工具中集成了一个驱动加载功能，用 scm 函数实现的，不值一提，只为图个方便，现提供源码给大家，欢迎大家 DIY 自己的工具。

上传的附件
| ![](img/2674941de574f12fa6a3e90d2eb3468a.jpg) || 驱动开发和调试环境搭建.doc \[请到论坛下载：[`bbs.pediy.com/showthread.php?t=95671`](http://bbs.pediy.com/showthread.php?t=95671) \]   
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || DriverLoader.rar  
 |
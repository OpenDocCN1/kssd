
# [矛与盾]调戏调试器：反断点技术

原帖及测试程序的地址：

[【原创】反断点实例](http://bbs.pediy.com/showthread.php?p=1295421)

本文我又发到我的CSDN博客里去了，还没审核通过![](https://bbs.kanxue.com/view/img/face/28.gif)

首先声明，理解本文需要对用户态的调试器原理有所了解，否则可能有些内容会不理解。

先来个测试程序，本文以分析和理解此程序为主。

[测试程序下载](https://bbs.kanxue.com/elink@353K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8S2L8W2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8U0q4I4g2$3I4j5P5h3q4%4)

无论是逆向分析、还是脱壳破解，都离不开调试器。而windows下面用户态调试器最常用的，那就是`OllyDbg`了。

现在就用`OD`载入并调试该程序，如下图所示。

![](https://bbs.kanxue.com/upload/attach/201406/423318_gxg2rvil1xfedxw.jpg)

`OD`载入程序后，就停在了`OEP`（原始入口点）。

可能有经验的朋友观察入口代码，就会发现，该程序的入口是典型的`VC++`。但是不要着急![](https://bbs.kanxue.com/view/img/face/28.gif)，慢慢来。

现在`F9`运行程序，程序是控制台界面的。如下图所示：

![](https://bbs.kanxue.com/upload/attach/201406/423318_qa9near5ykuuo6w.jpg)

提示下个断点，随便找个地方按`F2`下断点就行（不要下在`INT 3`处），我就下在`OEP`了。

![](https://bbs.kanxue.com/upload/attach/201406/423318_mjfehwh32nids52.jpg)

刚下完断点，就被中断了，一般程序是不可能，因为入口点一般只调用一次。

现在`F7`单步一下试试，如下图：

![](https://bbs.kanxue.com/upload/attach/201406/423318_sg14pahvaijxxez.jpg)

单步一下，程序居然直接崩了，很奇怪吧。好了，就调试到这里。![](https://bbs.kanxue.com/view/img/face/27.gif)现在看`TRE.exe`的源码来讲解。

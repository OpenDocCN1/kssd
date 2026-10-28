
# [原创]vmware VIX API 操作虚拟机 (开关机,上/下传文件, 执行EXE等)

主要利用vix api实现VM虚拟机电源控制等功能,VIX 头文件和lib都在VM安装目录下面

代码中已经打包了一份VM12的lib和.h,应该也能兼容其他版本.

本来是一个妹子在研究,我顺便折腾了下,界面上有些功能没有写,只做了基本的开关机.

`CVix`类里面已经封装了一些常用的操作.

VIX api可以对虚拟机干这些事:

开关机/重启,文件管理(上传/下载/删除/执行),进程管理(枚举/结束),快照管理...

然而,大部分操作,都是需要安装VMTools之后才能干....感觉没什么太大作用.

下载:

[VixDemo.zip](https://bbs.kanxue.com/attach-download-109365.htm)

顺便提示:VIX内部通信是用的socket,如果你的电脑上安装有腾讯游戏助手(TGP),可能导致VIX API不能正常工作,解决办法,`cmd`里面执行一下`netsh winsock reset`.貌似是TGP的网游LSP加速组件有BUG.![](https://bbs.kanxue.com/view/img/face/1.gif)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [VixDemo.zip](javascript:void(0);)

  （1.02MB，180次下载）

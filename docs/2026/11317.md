
# [分享]再来一种内核 inlinehook 的隐藏方法

无聊把一堆无法编译的代码改好，然后扔上来~分页内存的那些猫腻啊~~猫腻啊~~

不说了，走起~多数代码来自网络通过整理融合，修正bug 弄到一起组合而成。

直接附件里就代码，全真相哦~

PS:

求捐助，有意者联系 QQ:86879759

宣传一下无聊扯淡群：171797360

```
│  PageHack.sln
│  
├─bin
│  └─i386
│          pagehack.pdb
│          pagehack.sys
│          
└─PageHack
    │  buildfre_wnet_x86.log
    │  Clear.bat
    │  ddkbuild.bat
    │  ddkbuild.cmd
    │  Debug.c
    │  Debug.h
    │  drvversion.aps
    │  drvversion.h
    │  drvversion.rc
    │  hook.c
    │  hook.h
    │  List.c
    │  List.h
    │  MAKEFILE
    │  memory_type.h
    │  mybuild.bat
    │  mybuild_win7.bat
    │  ntifs.h
    │  ntifs_48.h
    │  pagehack.c
    │  pagehack.h
    │  PageHack.vcxproj
    │  PageHack.vcxproj.filters
    │  PageHack.vcxproj.user
    │  pagehack_main.c
    │  readme.txt
    │  ReBuild.bat
    │  resource.h
    │  SOURCES
    │  stdafx.h
    │  tools.c
    │  tools.h
    │  xde.c
    │  xde.h
    │  xde.txt
    │  xdetbl.h
    │  zwfunc.h
    │  
    ├─Debug
    │      PageHack.log
    │      
    └─objfre_wnet_x86
        │  _objects.mac
        │  
        └─i386
                drvversion.res
```

附加在这里 [`PageHack.rar`](https://bbs.kanxue.com/attach-download-69526.htm)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [`PageHack.rar`](javascript:void(0);)

  （324.97kb，1544次下载）

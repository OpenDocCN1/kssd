

# 木马行为分析回滚器---SafeOperation1.0

> 原文：[`www.pediy.com/kssd/pediy12/140708.html`](https://www.pediy.com/kssd/pediy12/140708.html)

标 题: 【原创】木马行为分析回滚器---SafeOperation1.0  
作 者: asm  
时 间: 2011-09-28,12::27:28  
链 接: http://bbs.pediy.com/showthread.php?t=140708  
  
小生微博：http://t.qq.com/hack-x86-asm (欢迎收听~)  
  
代码大部分都是 A 来的，其他就不多说了，Open Source~~  
  
![](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")  希望大牛们不要打击~~呵呵 ：）  
  
驱动编译环境：  
WDK 7600.16385.1  
“x86 Checked Build Environment”  
  
ring3 编译环境：  
VS2010  
  
文件列表：  
  
文件夹 PATH 列表  
卷序列号为 E803-4BA8  
K:.  
│  buildchk\_win7\_x86.log  
│  buildchk\_win7\_x86.wrn  
│  Common.h  
│  debug.c  
│  debug.h  
│  diskflt.c  
│  diskflt.h  
│  diskfltlib.c  
│  diskfltlib.h  
│  Driver.c  
│  Driver.h  
│  drvcommon.h  
│  drvversion.h  
│  drvversion.rc  
│  fatlbr.h  
│  file.c  
│  file.h  
│  InitSystemVer.c  
│  InitSystemVer.h  
│  InlineHook.c  
│  InlineHook.h  
│  ldasm.c  
│  ldasm.h  
│  log.c  
│  log.h  
│  makefile  
│  malloc.c  
│  malloc.h  
│  md5.c  
│  md5.h  
│  mempool.c  
│  mempool.h  
│  ntifs.h  
│  ntifs\_disk.h  
│  ProtectCode.c  
│  ProtectCode.h  
│  Reg.c  
│  Reg.h  
│  resource.h  
│  SafeHook.c  
│  SafeHook.h  
│  SafeOperation.c  
│  SafeOperation.h  
│  SafeOperation.sln  
│  SafeOperation.vcproj  
│  SafeRecords.c  
│  SafeRecords.h  
│  sources  
│  Thread.c  
│  Thread.h  
│    
├─objchk\_win7\_x86  
│  └─i386  
│          bin2c.exe  
│          MakeHeader.bat  
│          SafeOperation.h  
│          SafeOperation.sys  
│            
└─Ring3\_Safe\_Install  
    │  md5.c  
    │  md5.h  
    │  SafeOption.cpp  
    │  SafeOption.sln  
    │  SafeOption.vcxproj  
    │    
    └─Release

上传的附件
| ![](img/8e3adbfb230d8254e077b975ac3c087d.jpg) || 2.png  
 |
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || SafeOperation1.0.rar  
 |
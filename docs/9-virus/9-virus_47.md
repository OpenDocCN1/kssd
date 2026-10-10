

# 逆向整理包编译通过版鬼影 3.0 代码~

> 原文：[`www.pediy.com/kssd/pediy12/138978.html`](https://www.pediy.com/kssd/pediy12/138978.html)

话说，样本真操蛋~到处是花~~那个下载体 Alg.exe 我就不逆了~  
逆完在 VMWARE 测试发现很多 bug,很多 bug 啊~  
修改，修改去把 bootloader 改的 bug 不多了~，PmVirus 里肉眼可见的也改了不少了~但是还是开机卡住~不过从 IDA 里看没问题~纠结有 boot 调试环境的人来再改改吧~  
开源，发代码，是我的习惯~有实力改改做坏事的人请自己淡定~  
  
加个参考资料链接  
http://blog.csdn.net/gaa\_ra/article/details/6589324  
http://blog.csdn.net/gaa\_ra/article/details/6593860  
http://blog.csdn.net/gaa\_ra/article/details/6594815  
  
0.这是个对样本的逆向整理的版本，不要当作真的病毒，只包编译通过，其中去掉了原始鬼影样本中加密解密部分！！！！！  
  
  
1.编译环境需要：  
MASM32v10  
WINDOWS 2003 sp1 IFS DDK  
VS 2010 sp1  
WinSDK 7.0A  
  
2.编译方法：  
a.按 F7 生成解决方案  
b.到 bin 目录执行 makebin.bat  
c.选中 MakeVirusContent 点重新生成  
d.到 bin 目录执行 cmd.bat 然后在命令行下输入 MakeVirusContent.exe 需要命令行参数！！将你的下载者打包进 VirusContent  
e.选中 ShadowGhost3 点重新生成  
f.bin 目录里的 shadowGhost3.exe 就是最后的毒了！  
  
3.编译环境目录：  
  
Masm32v10 请安装在 D:\\  
Windows 2003 sp1 ifs ddk 用默认路径安装于 C:\\盘  
  
4.有关卡机卡住的 bug 的说明  
a.ProtectModeVirus 的 pmVirus.asm 代码里可能有些问题，可能是 bootloader.asm 修改其中代码发生了问题  
  
目录：  
│  ShadowGhost3.sln  
│  ShadowGhost3.vssscc  
│  说明.txt  
│    
├─bin  
│  │  Bin2Hex.exe  
│  │  bootload.bin  
│  │  bootload.h  
│  │  bootload.obj  
│  │  cmd.bat  
│  │  Driver.h  
│  │  Driver.sys  
│  │  makebin.bat  
│  │  MakeVirusContent.exe  
│  │  makeVirusHead.bat  
│  │  obj2bin.exe  
│  │  PmVirus.bin  
│  │  PmVirus.h  
│  │  PmVirus.obj  
│  │  ShadowGhost3.exe  
│  │  VirusContent.bin  
│  │  VirusContent.h  
│  │    
│  └─i386  
│          Bin2Hex.exe  
│          Driver.sys  
│            
├─Bin2Hex  
│      Bin2Hex.cpp  
│      Bin2Hex.vcxproj  
│      Bin2Hex.vcxproj.filters  
│      Bin2Hex.vcxproj.user  
│      Bin2Hex.vcxproj.vspscc  
│      buildfre\_wnet\_x86.log  
│      ddkbuild.bat  
│      ddkbuild.cmd  
│      MAKEFILE  
│      mybuild.bat  
│      readme.txt  
│      SOURCES  
│        
├─MakeVirusContent  
│      MakeVirusContent.cpp  
│      MakeVirusContent.vcxproj  
│      MakeVirusContent.vcxproj.filters  
│      MakeVirusContent.vcxproj.user  
│      MakeVirusContent.vcxproj.vspscc  
│      ReadMe.txt  
│      stdafx.cpp  
│      stdafx.h  
│      targetver.h  
│        
├─obj2bin  
│      obj2bin.cpp  
│      obj2bin.vcxproj  
│      obj2bin.vcxproj.filters  
│      obj2bin.vcxproj.user  
│      obj2bin.vcxproj.vspscc  
│      ReadMe.txt  
│      stdafx.cpp  
│      stdafx.h  
│      targetver.h  
│        
├─ProtectModeVirus  
│      make.bat  
│      PmVirus.asm  
│      ProtectModeVirus.vcxproj  
│      ProtectModeVirus.vcxproj.filters  
│      ProtectModeVirus.vcxproj.user  
│      ProtectModeVirus.vcxproj.vspscc  
│      readme.txt  
│        
├─ShadowGhost3  
│      drvss.cpp  
│      drvss.h  
│      ntdll.h  
│      ntdll.lib  
│      ReadMe.txt  
│      ShadowGhost3.cpp  
│      ShadowGhost3.vcxproj  
│      ShadowGhost3.vcxproj.filters  
│      ShadowGhost3.vcxproj.user  
│      ShadowGhost3.vcxproj.vspscc  
│      stdafx.cpp  
│      stdafx.h  
│      targetver.h  
│        
├─Sys  
│  │  buildfre\_wnet\_x86.log  
│  │  ddkbuild.bat  
│  │  ddkbuild.cmd  
│  │  Main.c  
│  │  MAKEFILE  
│  │  mybuild.bat  
│  │  ntifs\_48.h  
│  │  readme.txt  
│  │  SOURCES  
│  │  stdafx.h  
│  │  Sys.vcxproj  
│  │  Sys.vcxproj.filters  
│  │  Sys.vcxproj.user  
│  │  Sys.vcxproj.vspscc  
│  │  zwfunc.h  
│  │    
│  ├─Debug  
│  │      Sys.log  
│  │        
│  ├─driver  
│  │  └─i386  
│  │          Driver.pdb  
│  │          Driver.sys  
│  │            
│  ├─objfre\_wnet\_x86  
│  │  │  \_objects.mac  
│  │  │    
│  │  └─i386  
│  │          main.obj  
│  │            
│  └─Release  
│          Sys.log  
│            
└─VirusMBR  
    │  bootload.asm  
    │  make.bat  
    │  readme.txt  
    │  VirusMBR.vcxproj  
    │  VirusMBR.vcxproj.filters  
    │  VirusMBR.vcxproj.user  
    │  VirusMBR.vcxproj.vspscc  
    │    
    ├─Debug  
    │      VirusMBR.log  
    │        
    └─Release  
            VirusMBR.log  
看附件吧~~  
无聊了~感冒了，心情不好看这个样本然后结合各种东西，逆来逆去 花了 3 天时间各种调 bug~然后不想再弄下去了，所性放了~  
  
  
感冒好了，不准备搞 TDL4 逆向包编译通过鸟，忙着干别的，四处求 donate~  
  
TDL 的 MBR 部分很简单就是把 ldr16 Load 进来，然后把控制权交给 ldr16~  
然后 ldr16 会 hook int13,然后通过 hook 判断 Mz 头 pe 头（根据 MZ 和 PE 头的一些数据判断版本号和系统 x64 与否）来实现文件替换（其实 kdcom 在硬盘上是连续存放的，用 ldr32/ldr64 来替换）和签名绕过  
巧妙的地方是修改绕过签名的/mini 启动模式参数修改~  
然后 ldr32/ldr64 会弄出对应 drv32/drv64 到内核~  
好了说完了，基本上 TDL 就是这么个功能~  
至于真正有趣的地方是丫的 drv 部分实现了 HiddenPartDisk 在 Win 下挂接自己的硬盘对象和自定义的文件系统 HiddenFSIoCreateDevice 这个函数用的很好强大~

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || ShadowGhost3.rar 解压密码是：pediy  
 |
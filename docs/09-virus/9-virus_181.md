
# 续异鬼(Trojan.Win32.Agent.e) uiui8.dll 病毒继续查杀

> 原文：[`www.pediy.com/kssd/pediy11/120156.html`](https://www.pediy.com/kssd/pediy11/120156.html)

首先。我不太清楚这里是不是能发布病毒查杀的文章。

如果不可以请删贴。但不要删号啊。。谢谢了

今天去单位接着给同事的电脑做后续完善工作。。昨天只是把病毒删除了但是由于没有网络查不了什么东西

昨天回家后把这个病毒深入了解了下。。

手工这个杀这个方法如下。

昨天只是做了第一步。。可恶的病毒竟然把所有的杀软都给禁止了。。难怪昨天安装卡卡想查查都安装不上。

需要删除掉第三步中的整个键喵了个咪的大家看看吧他里面竟然把所有杀毒程序的`exe`都给禁止了。

但值得庆幸的是我用的那个`xuetr`没有禁止万幸啊。。

## 步骤概览

1. 删除病毒文件
2. 显示`exe`文件后缀，修复关联
3. 恢复正常的杀软，程序劫持
4. 修复可以进入安全模式

### 1、手动删除以下文件：

```
%ProgramFiles%\Common Files\Microsoft Shared\explorer.exe
%Documents and Settings%\All Users\桌面\Intennet Exploner.lnk
%Documents and Settings%\All Users\桌面\改变你的一生.url
%Documents and Settings%\All Users\桌面\淘宝购物 A.url
%Documents and Settings%\All Users\桌面\免费电影 C.url
%SystemDriver%\MFILES\winlogon.exe
%ProgramFiles%\Common Files\uiui8.dll
%Documents and Settings%\All Users\「开始」菜单\程序\启动 1681.lnk
```

### 2、手动删除注册表项：

```
HKEY_CLASSES_ROOT\exefile
名称：NeverShowExt
数据：1

HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Policies\Associations
名称：ModRiskFileTypes
数据：.exe
```

### 3、手动清除该注册表项下属全部项目

```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\
```

### 4、将以下注册表项的正常值导入注册表：

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SafeBoot\Minimal\{4D36E967-E325-11CE-BFC1-08002BE10318}
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SafeBoot\Network\{4D36E967-E325-11CE-BFC1-08002BE10318}
HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Control\SafeBoot\Minimal\{4D36E967-E325-11CE-BFC1-08002BE10318}
HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Control\SafeBoot\Network\{4D36E967-E325-11CE-BFC1-08002BE10318}
```

那个手工删除上面第一步看我前一个帖子。。

关于后面的注册表修复我写了一个批处理。。

```
=====================================分割线=====================================

@ Echo Off
@ Echo.
@ Echo. 说 明
@ Echo ==================================================================
@ Echo 修复 异鬼(Trojan.Win32.Agent.e)破坏的注册表 by:wy811007
@ Echo ==================================================================
@ Echo.

PAUSE
reg delete HKEY_CLASSES_ROOT\exefile /v NeverShowExt /f
reg delete HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Policies\Associations /v ModRiskFileTypes /f
reg delete HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options /va /f
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SafeBoot\Minimal\{4D36E967-E325-11CE-BFC1-08002BE10318} /ve /d DiskDrive
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SafeBoot\Network\{4D36E967-E325-11CE-BFC1-08002BE10318} /ve /d DiskDrive
reg add HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Control\SafeBoot\Minimal\{4D36E967-E325-11CE-BFC1-08002BE10318} /ve /d DiskDrive
reg add HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Control\SafeBoot\Network\{4D36E967-E325-11CE-BFC1-08002BE10318} /ve /d DiskDrive
@ Echo 修复完毕 谢谢使用 按任意键结束 by:wy811007 & pause > nul

=====================================分割线=====================================
```

将分割线内的脚本复制然后新建一个`.txt`的文本文档打开粘贴进去改后缀为`.bat`

看不到后缀的。打开`我的电脑`工具`文件夹选项`查看`隐藏已知文件类型的扩展名`的勾取消掉确定就可以看到了。

第一个帖子在

[【原创】记异鬼(Trojan.Win32.Agent.e )uiui8.dll 病毒查杀 ](http://bbs.pediy.com/showthread.php?t=120155)

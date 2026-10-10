

# Windows 2000 RC3

> 原文：[`www.pediy.com/kssd/pediy02/forum-crack12.htm`](https://www.pediy.com/kssd/pediy02/forum-crack12.htm)

作者：Sun Bird \[CCG\]（我，Sun Bird 属于破解团体 China Cracking Group ^\_^）

日期：2000 年 3 月 22 日

这篇教程是我在国外站点看到的，估计很多玩 Windows 2000 的朋友都需要，所以我就把它  
贴上来了。  
考虑到通篇的 E 文也许会搞得某些朋友一头雾水（借把伞吧，或者买瓶洗发水^\_^），于是  
我就用我那蹩脚的 E 文和糟糕的中文不自量力地将关键部分 Translate 了（希望你们还能够看  
明白……^\_^）。

 需要声明的是，本人的机器实在太烂了（Pentium 166MMX, 64M EDO, 1.2G Hard Disk...），  
不可能安装 Windows 2000，所以我不太清楚某些 Windows 2000 的名词翻译的是否正确。

 特别需要指出的是，原文是针对英文版的 Windows 2000 RC3，对于中文版我不能肯定百分百  
成功，所以是否使用全凭自愿。如果按照文中的方法操作失败并导致了什么“严重”的后果（能  
有什么“严重后果”？大不了重装嘛！有人在安装了无数软件的机器上尝试 Windows 2000 RC3 ？  
哇－你是老大！^\_^）本人概不负责（丑话说在前头，以免遭人“暗算”^\_^）。

  
This tutorial assumes you have downloaded an evaluation version of Windows 2000  
that will expire after a certain number of days. If you would like to change this to  
a time-unlimited version, follow the instructions below.

 In this tutorial, keep in mind the following things:  
在这个教程中，请牢记下面几点：

 (1) The method described below only works in NT systems (ie you must be running  
Windows NT4 or Windows 2000 already).  
下面描述的方法只能工作在 NT 环境下（也就是说你必须已经运行在 Windows NT4 或  
Windows 2000 环境下）

 (2) You must reinstall to change a time-limited version to a time-unlimited   
version.  
你必须重新安装 Windows 2000，以便将限时版更改为无时间限制版本。

 Now, without further ado:

 (1) Find your setupreg.hiv file (it's in your i386 directory) and put it some-  
where not write-protected. If you have already burned your Windows 2000 CD, you must  
copy the setupreg.hiv file to the hard drive.  
找到 setupreg.hiv 文件（在 i386 子目录内）并放到没有写保护的地方。如果你已经  
烧录了 Windows 2000 光盘，你必须将 setupreg.hiv 文件复制到硬盘上。

 (2) Click on Start, Run. Enter "regedt32" (without the quotes) and click OK.  
点击“开始－运行”，输入“regedt32”（不包括引号）并点击“确定”。

 (3) Once the registry editor is loaded, make sure you have selected the "HKEY\_  
LOCAL\_MACHINE" key in the "HKEY\_LOCAL\_MACHINE on Local Machine" window.  
当“注册表编辑器”运行后，确定你在“HKEY\_LOCAL\_MACHINE on Local Machine”窗  
口中选择的是“HKEY\_LOCAL\_MACHINE”分支。

 (4) Under the Registry menu, choose Load Hive... Navigate to the location where  
your setupreg.hiv file is at and select it, choose Open.  
在“注册”菜单，选择“调入 Hive...”，选择你的 setupreg.hiv 文件并“打开”。

 (5) You will be presented with a Key Name box. It does not matter what you put  
here, as long as you remember it. I recommend something like Setupreg.  
你会见到“键名”对话框。在这里输入什么并不重要，你仅仅需要记住你输入的信息。  
我推荐你输入“Setupreg”。

 (6) Click OK to load it into the registry. After it loaded, navigate to the key Setupreg\\ControlSet001\\Services\\setupdd\\ and select the value "No Name".  
点击“确定”将之调入。找到“Setupreg\\ControlSet001\\Services\\setupdd\\”主键并  
选择键值“No Name”。

 (7) Double click on this value to open it up for editing. This binary value   
corresponds to the 120 day time limit. To remove the time limit, change the A, the 3,  
and the 2 to 0's. Click OK.  
双击打开编辑该键值。这个二进制键值决定了 120 天限制。要去掉这种限制，你需要  
将“A、3、2”改为“0”。点击“确定”。

 (8) Navigate back up the tree and select the Setupreg key. Go to the Registry   
menu and click Unload Hive. Registry editor will prompt you, click Yes to continue.  
倒退回注册表分支树找到“Setupreg”键。在“注册”菜单选择“退出 Hive”。“注  
册表编辑器”会提示你，点击“是”继续。

 (9) That's it! Once you've unloaded the key, close the registry editor. Make  
sure the edited setupreg.hiv file is in your i386 directory before installing and  
subsequent installations will be time-unlimited.  
OK 啦！当退出该键后，关闭“注册表编辑器”。在你再次安装前，请确定编辑修改过  
的 setupreg.hiv 文件在 i386 子目录内，这样重新安装后的 Windows 2000 RC3 将是没有时间  
限制的版本。

  
作者：.........（忘记了，真不好意思）

翻译：Sun Bird \[CCG\]（就是 China Cracking Group 的 Sun Bird 啦^\_^）

日期：2000 年 3 月 15 日（打倒所有的假冒伪劣产品！质量上乘的 D 版除外！^\_^）

                                         

 4dd466d0e1e0cc123afbe504da2462ca
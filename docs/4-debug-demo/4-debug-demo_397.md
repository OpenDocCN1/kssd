

# 一款屏幕录像软件的分析

> 原文：[`www.pediy.com/kssd/pediy10/92807.html`](https://www.pediy.com/kssd/pediy10/92807.html)

【文章标题】: 一款屏幕录像软件的分析  
【文章作者】: limpidlove  
【软件名称】: Instant Demo  
【下载地址】: http://www.instant-demo.com/  
【使用工具】: Ollydbg,PEID,eXeScope  
【作者声明】: 初级教程，只供学习交流！  
\--------------------------------------------------------------------------------  
【详细过程】  
  
   这里向大家介绍的这款软件叫 Instant Demo，看过 Lena151 和 TiGa 教程的朋友应该熟悉了。废话不多说，开始分析。  
   分析一个软件，我们首先应该了解它都有哪些保护和限制，官方网站这样描述：The Trial version has no time limit and is fully functional.  A Trial Version watermark is placed on demonstrations exported from Instant Demo.  
     好，我们自己再分析一下。安装后，第一次打开，发现标题栏和 Help 菜单 About Instant Demo 有 Trial 字样，还有 Purchase Instant Demo 选项。  
   有了大概印象，我们在做一个完整的录制看一下，首先创建的 htm 文件有 Click here to Buy Instant Demo 字样。其次录像文件中有水印，如果录制声音的话，还有噪音，等等。  
   有了整体了解，我们就可以开工了。首先用 PEID 察看无壳。好，我们开始分以下几个步骤走。  
  
一 修改标题栏  
  
  1.在 cpu 窗口中，右键选择 查找 -- 所有参考文本字串，然后右键 查找文本，不选区分大小写，勾选整个范围  
  填入 Instant Demo Trial  
  2.确定后，Ctrl+L 来到地址 00477FFB，直接回车，或右键选择反汇编窗口中跟随，来到  
   00477FFB   .  68 784E5500   push    InstantD.00554E78                ;  ASCII "Instant Demo Trial v%d.%2.2d - F1 for help"  
  3. CPU 窗口中右键，数据窗口中跟随立即数，二进制编辑，可以把 Trial 改为-Full，然后右键复制到可执行文件，再右键保存文件为 InstantDemo1.exe。  
  
二 修改 Help -- About Instant Demo 菜单  
  
   OD 打开 InstantDemo1.exe  
  
  1. 重复修改标题栏的第一步，  
  填入 Trial  
   确定后，来到第一处，地址 00402676，直接回车，来到  
   00402676   .  68 80F65400   push    InstantD.0054F680                ;  ASCII "Instant Demo Trial",LF,"Software Version %d.%2.2d.%2.2d %s",LF,"%sNetPlay Software",LF,"Copyright (C) 2003-2009"  
  
  2. 重复修改标题栏的第三步把 Trial 改为-80ND 或你想要的，此字符将显示在 Help About Instant Demo 中。然后右键复制到可执行文件，再右键保存文件为 InstantDemo2.exe  
  
三 删除 Click here to Buy Instant Demo  
   OD 打开 InstantDemo2.exe  
  
  1. 重复修改标题栏的第一步，  
     填入 Click here to Buy Instant Demo  
  
   确定后，来到地址 004873FD，直接回车，来到  
   004873FD  |.  68 1C565500   push    InstantD.0055561C                ;  ASCII "<p align=""center""><a href=""%s%s""><font size=""6"">Click here to Buy Instant Demo</a></p>"  
  
  2. CPU 窗口中右键，数据窗口中跟随立即数，把 1.htm 用记事本打开参照一下  
  
   0055561C  3C 70 20 61 6C 69 67 6E 3D 22 63 65 6E 74 65 72  <p align="center  
0055562C  22 3E 3C 61 20 68 72 65 66 3D 22 25 73 25 73 22  "><a href="%s%s"  
0055563C  3E 3C 66 6F 6E 74 20 73 69 7A 65 3D 22 36 22 3E  ><font size="6">  
0055564C  43 6C 69 63 6B 20 68 65 72 65 20 74 6F 20 42 75  Click here to Bu  
0055565C  79 20 49 6E 73 74 61 6E 74 20 44 65 6D 6F 3C 2F  y Instant Demo</  
0055566C  61 3E 3C 2F 70 3E                                a></p>  
  
  将以上右键二进制，用 00 填充，然后右键复制到可执行文件，再右键保存文件为 InstantDemo3.exe。  
  
四 去除水印  
 OD 打开 InstantDemo3.exe  
  1. 重复修改标题栏的第一步，填入 Trial  
  
  确定后，来到 0041212F，如不在此处，用 Ctrl+L 。发现有 ASCII "IDR\_TRIAL\_LOGO"字样，回车来到  
  
   0041212F  |.  68 800B5500   push    InstantD.00550B80                ;  ASCII "IDR\_TRIAL\_LOGO"  
  
 向下滚动，查找意思为跳转到的向右的箭头，第一个来到  
  
   004121AE  |> \\8B4424 28     mov     eax, dword ptr ss:\[esp+28\]  
  
  跳转来自 0041210B，不是我们想要的，我们需要的是能跨过 OffsetRect，Bitblt 和"IDR\_TRIAL\_LOGO"的跳转，这里有些不理解，起初最先想到是能跨过"IDR\_TRIAL\_LOGO"就行了。希望有人指点一下。我只是试验后发现的。  
  
  再向下滚动，来到  
  
   004121DE  |> \\8B4424 44     mov     eax, dword ptr ss:\[esp+44\]  
  
  跳转来自 00411D66，符合条件  
  
   00411D66  |. /0F84 72040000 je      InstantD.004121DE  
  直接双击把 je 修改为 jmp，然后复制到可执行文件，所有修改，重复上面的步骤，保存为 InstantDemo4.exe  
  
 五 去除噪音  
  
  如默认只包含鼠标的声音，可省去这一步。OD 打开 InstantDemo4.exe  
  
  1. 重复修改标题栏的第一步，填入 denoise  
  
  确定后，来到地址 004212B9，回车，跟随立即数，选择\_denoise.dat 用 00 填充。复制到可执行文件，保存为 InstantDemo5.exe。  
  
  2  打开 InstantDemo5.exe  
  
  在 cpu 窗口中，右键选择 查找 -- 所有参考文本字串，重复上面。来到地址 00421313  
反汇编跟随，跟随立即数，选择\_denoise.wav 用 00 填充。复制到可执行文件，保存为 InstantDemo6.exe。  
  
  3. 打开 InstantDemo6.exe  
  
  重复 1，来到地址 00426D1B 反汇编跟随，跟随立即数，选择\_denoise 用 00 填充。复制到可执行文件，保存为 InstantDemo7.exe  
  
  
去除噪音或者手动删除后缀为\_denoise.dat 和\_denoise.wav 的文件。  
  
  六 去除 Help - Purchase Instant Demo  
  
  使用 eXeScope 或你喜欢的工具打开 InstantDemo7.exe 把菜单删除或变灰。  
  
  七 备份 InstantDemo.exe 将 InstantDemo7.exe 改名为 InstantDemo.exe  
  
其它，去除水印那一步有些糊涂，只是被我试验成功，希望有人指点一下。  
另：第一次作文，有些乱，心里还有些没底，需要勇气！  
  
  
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪软件安全论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2009 年 07 月 04 日 5:28:42
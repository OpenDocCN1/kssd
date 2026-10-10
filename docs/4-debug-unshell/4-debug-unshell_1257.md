

# <脱壳通辑令>之一

> 原文：[`www.pediy.com/kssd/pediy03/forum585.htm`](https://www.pediy.com/kssd/pediy03/forum585.htm)

需要下载教程中工具请到:http://longdiy.myetang.com/    或   
                        http://go8.163.com/~panlong/   
    <脱壳通辑令>之一   
〖1〗    -------脱掉由 PCG 加密过软件的壳   
脱壳对象:PC Guard for Win32 3.04D2   
主页地址: http://www.pc-guard.co.yu   
说明:加密软件,下简称 PCG   
脱壳需要工具:TRW2000 1.23,PEditor 1.7,RES,BlW2000 0.2   
1\. 运行 Blw,按 Track,运行 PCG,得到脱壳入口点:00411250,   
  然后退出 BW2000 和 Exescope   
2\. 运行 TRW2000,装入 PCG,入到 TRW 调试界面后下命令 e eip 0   
3\. 在 TRW2000 中下命令 g 00411250    (在 TRW 中去到 PCG 加壳入口点)   
4\. 在 TRW2000 中下命令 makepe      (TRW 会在 PCG 目录下产生个脱壳文件 Newpe.exe)   
5\. 按 X 退出 TRW2000   
6\. 运行 RES,用 RES 打开 Newpe.exe,接着按工具中的建置相容资源.确定后退出.   
7\. 运行 PEditor 1.7,按浏览打开 Newpe.exe,接着按 Rebuilder.   
8\. 进入重建画面后选择 realign file 和其中的 normal,再选择   
  rebuild Improt Table 和其中的 rebuild new Improt Table,   
  最后选择 make PE header win nt/2k compatible   
9\. 按 DO 重建引入.确定后退出.   
        完成,收工.   
  
〖2〗  --------脱掉由 UPX 0.XX-Upx 1.0x 压缩过软件的壳   
脱壳对象:CodeFusion Wizard Version 3.0   
主页地址:http://come.to/kobik   
说明:补丁制作软件,它已补 UPX 0.72 压缩,下称 CFW   
  
脱壳需要工具:FS,RES   
1\. fs -u cfg.exe   
2\. fs -rn -spl cfg.exe   
3\. 运行 RES ,打开 cfg.exe,接着按工具中的建置相容资源.确定后退出.   
脱壳完成,收工.(哗!这么简单?是哦!就这么简单,不信,你试呗)   
  
〖3〗  --------脱掉由 PECompact 1.xx 压缩过软件的壳   
脱壳对象:PECompact 1.43   
主页地址:http://www.collakesoftware.com   
说明:压缩软件,下简称 PEC   
脱壳需要工具:fs,PEditor 1.7,RES 3.0   
1\. fs -u pec.exe   
2\. fs -rn -spl PEC.exe   
3\. 运行 RES ,打开 PEC.exe,接着按工具中的建置相容资源.确定后退出.   
4\. 运行 PEditor 1.7,按浏览打开 PEC.exe,接着按 Rebuilder.   
5\. 进入重建画面后选择 realign file 和其中的 normal,再选择   
  rebuild Improt Table 和其中的 rebuild new Improt Table,   
  最后选择 make PE header win nt/2k compatible   
9\. 按 DO 重建引入.退出收工   
          脱壳完成,收工.   
  
〖4〗  --------脱掉由 Telock 0.71 压缩过软件的壳   
脱壳对象: 用 Telock 0.71 压缩 exescope.exe   
说明: telock 压缩选项(防 softice 检测,重整   
      覆盖和 reloc 区段和 ITA 重定位)   
脱壳需要工具:TRW2000,Procdump,RES,BlW2000 0.2   
脱壳过程:   
1\. 运行 Bw2000.exe,单击 BW2000 中的 Track,  (用冲击波检测加壳入口点)   
  然后运行 Exescope.exe, 得到加壳入口   
  点 004b1ec4,退出 BW2000 和 Exescope          
2\. 运行 trw2000,然后装入 Exescope.exe          
3\. 在 TRW2000 中下命令 g 004b1ec4          (去到加壳入口点)   
4\. 在 TRW2000 中下命令 suspend                
5\. 回到系统界面后运行 Procdump.exe        (用 Procdump 进行去壳)   
6\. 跟住点击 Procdump 上窗口中 Exescope.exe   
7\. 接着光标放到下窗口中的 EXEscope.exe,   
  然后按鼠标右键,选择 Dump (Full),然后   
  保存脱壳文件,单击确定后退出.   
8\. 运行 RES 装入刚由 Procdump 脱壳后保存的文件,    (用 RES 重整资源结构)   
  然后选择工具中的压缩读出,读出完成后,再选   
  择工具中的重建相容资源.   
9\. 完成后,当然是收工啦!!!   
  
〖5〗  --------脱掉由 Aspack 2.11D 压缩过软件的壳   
脱壳对象: 用 Aspack 2.11D 压缩 exescope.exe   
说  明: 是用 Aspack 2.11D 未注册版压缩的   
脱壳过程:   
方法一  (脱壳需要工具:Procdump,RES)   
1\. 运行 Bw2000.exe,单击 BW2000 中的 Track,   
  然后运行 Exescope.exe, 得到加壳入口   
  点 004b1ec4,退出 BW2000 和 Exescope  (用冲击波检测加壳入口点)   
2\. 运行 trw2000,然后装入 Exescope.exe          
3\. 在 TRW2000 中下命令 g 004b1ec4      (去到加壳入口点)   
4\. 待去到加壳入口点后,下命令 suspend   
5  待回到系统界面后,运行 Procdump.exe              (用 Procdump 进行去壳)   
6\. 跟住点击 Procdump 上窗口中 Exescope.exe   
7\. 接着光标放到下窗口中的 EXEscope.exe,        (方法⑴)   
  然后按鼠标右键,选择 Dump (Full),接着   
  保存脱壳文件,单击确定后退出.   
    \*\*\*\*\*\*\*\* 或 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
  光标对住上窗口中的 EXEscope.exe 后,          (方法⑵)   
  然后按鼠标右键,选择 Dump (Full),接着      (脱壳试哪个脱壳文件可在   
  保存脱壳文件,单击确定后退出.                WIN2000 和 ME/98 下可运行)   
完成后,当然是收工啦!!!   
  
方法二 (脱壳需要工具:TRW2000 1.23)   
1\. 运行 Bw2000.exe,单击 BW2000 中的 Track,   
  然后运行 Exescope.exe, 得到加壳入口   
  点 004b1ec4,退出 BW2000 和 Exescope  (用冲击波检测加壳入口点)   
2\. 运行 trw2000,然后装入 Exescope.exe          
3\. 在 TRW2000 中下命令 g 004b1ec4      (去到加壳入口点)   
4\. 在 TRW2000 中下命令 makepe          (TRW 会在 PCG 目录下产生个脱壳文件 Newpe.exe)   
      完成,收工   
                
以上脱壳后的软件均可在 win98/nt/win2000 平台下运行.   
软件脱壳通辑令中的脱壳均利用工具进行脱壳.  4dd466d0e1e0cc123afbe504da2462ca
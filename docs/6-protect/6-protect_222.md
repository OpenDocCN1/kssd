

# 编译 Flexlm 11.4 SDK 的步骤

> 原文：[`www.pediy.com/kssd/pediy10/61770.html`](https://www.pediy.com/kssd/pediy10/61770.html)

这两天刚好编绎了一下，我是用的是 VS2003（VC7.1）：  
1. 将 C:\\Program Files\\Microsoft Visual Studio .NET 2003\\Common7\\Tools\\目录下的 vsvars32.bat 文件复制到 Flexlm\_SDK\_11.4 下的子目录 i86\_n3\\  
2.打开 CMD 窗口，切换到目录 i86\_n3\\，执行 vsvars32.bat，这样 nmake 核 cl 的环境变量就都设置好了  
3.用最新的 FLEXlm Vendor Key Generator（lmkg V3）生成 Flexlm V11 版本的 VENDOR\_KEY1～5 和 TRL\_KEY1～2 以及 VENDOR\_NAME  
4.然后修改 machind 目录下的 lm\_code.h 文件，其中 LM\_SEED1～3 可以用命令 lmrand1 -seed 生成，不过为了破解软件，这个可能需要你自己找！！注意：如果这个设置不对编绎将出错！！  
5.修改目录 i86\_n3\\下的 makefile 文件,将 demo 改为你的 VENDOR\_NAME  
6.回到目录 i86\_n3\\，在 CMD 窗口运行 build md, 将在 i86\_n3\\目录生成 VENDOR 和 lmcrypt 等可执行文件  
7.按照 lic 的格式要求以及找到 feature 名，建立 lic 文件，在 CMD 窗口执行 lmcrypt 文件名.lic，然后 lic 文件就可以生成了  
8.将 VENDOR 和 lic 复制到 flexlm 安装的相关目录，配置 flexlm 服务，一切大公告成！！  
  
注意：关闭 DOS 窗口后，如以后再次编绎的话，vsvars32.bat 需要再次执行，另外还需要把先前编绎生成的文件都清理掉，不然编绎不执行。  
  
这个步骤同样适用于编绎 Flexlm 10.8.0.10 SDK。
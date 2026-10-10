

# 利用 Flex 10.1 SDK 开发某软件注册机

> 原文：[`www.pediy.com/kssd/pediy12/136910.html`](https://www.pediy.com/kssd/pediy12/136910.html)

【软件名称】利用 Flex 10.1 SDK 开发某软件注册机。  
【应用平台】WinXP SP4  
【作者邮箱】chubing6143@sina.com  
【使用工具】OllyDbg1.10，FlexLM 10.1SDK  
【软件限制】FlexLM  
  
  好长时间不破解了，也好长时间没有到看雪来了。  
  最近因为需要帮人解决一个软件远程桌面调用的问题，该软件我以前用 Flexlm 7.2SDK 写的注册机，但是由于 FlexLM 低版本的 SDK 不支持远程桌面使用方式，即 License 文件中不支持 TS\_OK，因此，需要利用高版本的 SDK 重新开发 NI 通用注册机。因此，我就又操起了这一行了，好长时间感觉没有投入破解了，还是怀念当初学习破解的过程，当然由于自己工作的原因，没有坚持下来，因此，连二把刀都不算。写这篇文章只是为了给我这样的初学者提供帮助。考虑到论坛的需要，就不公布是什么软件了。为了给初学者尽可能多的帮助，我将过程记录的尽可能详细，高手飘过。  
  
一、首先安装 FLEXnetLicensing.exe（10.1 的 SDK）。  
二、利用 CrackZ 上下载的 Flexvkg3（lmkg.exe）自动生成 Vendor 为\*\*\*\*（怎样找 Vendor，论坛上介绍的太多了，不叙述），Version 为 10 的 Key 如下：  
/\* Version 10 keys \*/  
#define VENDOR\_KEY1 0x0218a4\*\*  
#define VENDOR\_KEY2 0xa52467\*\*  
#define VENDOR\_KEY3 0x185a43\*\*  
#define VENDOR\_KEY4 0xfa721f\*\*  
#define VENDOR\_KEY5 0x011a5c\*\*  
#define TRL\_KEY1 0x31f528\*\*  
#define TRL\_KEY2 0x4cc898\*\*  
  
#define VENDOR\_NAME "\*\*\*\*"  
  
然后编辑 C:\\Program Files\\flexlm\\v10.1\\machind\\lm\_code.h 文件，将其中的 VENDOR\_KEY、VENDOR\_NAME 与 TRL\_KEY 替换成前面生成的宏定义。  
  
三、将 C:\\Program Files\\flexlm\\v10.1\\i86\_n3\\makefile 文件中 VENDORNAME = demo 替换为 VENDORNAME = nilm  
然后运行 cmd，进入命令行模式下 C:\\Program Files\\flexlm\\v10.1\\i86\_n3\\lmrand1 seed  
会随机生成 LM\_SEED1、LM\_SEED2、LM\_SEED3 的值，并生成 lmseed.dat 文件，将该文件中的 LM\_SEED 值粘贴到 lm\_code.h 中去。  
四、考虑到该软件加密是从 7.1 升级过来的，因此在 lm\_code.h 中粘贴下面的代码：  
#define LM\_SIGN\_LEVEL 1  
   
#define ENCRYPTION\_SEED1 0x\*\*1b372e（需要对软件跟踪进行查找，为了安全起见，我隐去了）  
#define ENCRYPTION\_SEED2 0x\*\*394f25（需要对软件跟踪进行查找，为了安全起见，我隐去了）  
  
  
#define ENCRYPTION\_SEED3 0x12315688 //任意值了  
#define ENCRYPTION\_SEED4 0x12315688 //任意值了  
并将 LM\_STRENGTH 定义为 LM\_STRENGTH\_LICENSE\_KEY  
#define LM\_STRENGTH LM\_STRENGTH\_LICENSE\_KEY  
  
五、此时命令行模式下 C:\\Program Files\\flexlm\\v10.1\\i86\_n3\\build.bat 会自动生成 lmcrypt.exe，该文件就可以将 license 文件自动完成加密的过程。  
然而为了写注册机，还得进行下面的工作。  
  
六、命令行模式下 C:\\Program Files\\flexlm\\v10.1\\i86\_n3\\build DLL  
会自动生成 lmgr10.dll 与 lmgr10.lib，这两个文件是非常有用的，写注册机时需要链接这两个文件，写注册机时需要 SDK 中的以及 build 生成的以下文件：  
lm\_attr.h、lm\_code2.h、lm\_code.h、lm\_trl.h、lmclient.h、lmhostid.h、lmprikey.h、lmseeds.h、lmgr10.lib、lmgr10.dll。  
七、为了在 VC 工程中能够编译通过，首先将 lmprikey.h 中的  
lm\_extern int \*l\_prikey\_sign(void) = 0;  
修改为：  
lm\_extern int (\*l\_prikey\_sign)(void) = 0;  
然后在 Project Setting 中添加 PC 的宏定义。  
写解密的关键代码函数如下：  
  
LM\_CODE\_GEN\_INIT(&site\_code);//:初始化 Flex 变量，主要是 lm\_code.h 中的变量  
if (lc\_init((LM\_HANDLE \*)0, VENDOR\_NAME, &site\_code, &lm\_job))  
{  
  lc\_perror(lm\_job, "lc\_init failed");  
  exit(-1);  
}  
if (lc\_cryptstr(lm\_job, fileCrypt, &ofile\_str, /\*&v\*/&site\_code, forceit, "",   
      &err))//laowang :关键函数将 License 文件字符串内容加密求得正确的 License 文件  
{  
  MessageBox(NULL,"lc\_cryptstr 运行错误，请联系作者！",filePath,MB\_OK);    
}  
  
到此就完成了，如果有我等初学者对此感兴趣的，我可以考虑将基本的注册机代码发上来。


# Icon Catcher

> 原文：[`www.pediy.com/kssd/pediy01/forum025.htm`](https://www.pediy.com/kssd/pediy01/forum025.htm)

目标程序: Icon Catcher 3.0.1.324   
程序用途: 从 DLL, EXE, ICO, ICL 和 BMP 文件中抓出图标   
下载地址: http://www.helexis.com/ic/iconcatc.exe  或   
      http://skyscraper.fortunecity.com/pacific/730/software/iconcatc.exe   
文件大小: 416K   
保护类型: 加壳,使用功能限制(不能存盘或拷贝至剪贴板)   
  
\*\*该程序不大,而且也很好用,有兴趣的朋友不妨下载一试. 我这里说的破解方法不一定是   
最好的,你的破解经过如果与之不同,欢迎贴出来交流. 冰毒 2000.3.16   
  
1\. 脱壳: 程序用 Neolite 2.0 加壳,很容易脱掉,而且脱壳程序不改程序入口值也可以正常运行.   
  
2\. 去除功能限制: 利用程序的警告框,不难追到下面这段代码   
  
014F:00480549  FF5130              CALL    \[ECX+30\]   
014F:0048054C  8BD8                MOV    EBX,EAX <----改成 MOV BL,01   
014F:0048054E  84DB                TEST    BL,BL   
014F:00480550  750F                JNZ    00480561                    
014F:00480552  66837E3A00          CMP    WORD PTR \[ESI+3A\],00   
014F:00480557  7408                JZ      00480561                  
014F:00480559  8BD6                MOV    EDX,ESI   
014F:0048055B  8B463C              MOV    EAX,\[ESI+3C\]   
014F:0048055E  FF5638              CALL    \[ESI+38\] <----这个 CALL!   
014F:00480561  8BC3                MOV    EAX,EBX   
  
3\. 去掉主窗口右下角和 About 显示中的'UNREGISTERED COPY'字样:   
  
不象上面两项那么容易. :) 你有无更好的方法?   
  
用 eXeScope 编辑脱壳的文件,找到下面的部分(RCDATA->TMAINFORM)   
  
object License: TLicense   
    EncodingKey = 'v=X\]8}%KW\]QVax"I&'   
    MasterKey = 'y=w/c9r<qY}'   
    sRegisteredTo = '\_Xt\\€grerW-g|'   
    sUnregistered = 'f=c4X8dCVAV312\`?j'   
    RegistrationURL = 'http://www.helexis.com/ic/register.html'   
    ReleaseDate = 36492   
    List = ';%;%;%@.;'#2#21   
    OnBadLicense = SelfInstaller1BadLicense   
    Left = 144   
    Top = 112   
  
把 sUnregistered = 'f=c4X8dCVAV312\`?j'这一行删除. 收工!
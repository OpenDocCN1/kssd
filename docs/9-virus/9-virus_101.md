

# css.exe 逆向分析

> 原文：[`www.pediy.com/kssd/pediy11/119025.html`](https://www.pediy.com/kssd/pediy11/119025.html)

css.exe 逆向分析  

引用: int \_\_stdcall sub\_100637A(HMODULE a1, int a2, const CHAR \*a3, int a4)  
{  
  if ( Init(a1, a3, a4) )  
  {  
    v4 = TorjanKernelFuc();  
    DeleteTmpDirectory(v5);                     // - -病毒启动部分分析完毕、其实这只是一个外壳而已、做坏事的都是从 CAB 包里释放出来的垃圾  
  }  
} 

分析量太多了、都贴上来我就要抓狂了、而且上色也是个问题、所以各位自己欣赏附件里的 idb 吧……idb 分析的很详细、而且部分函数有 MSDN 解释~  
\- -通过程序内部对资源段的配置信息读取可以很明确的确认这个病毒是一个由木马生成器产生的病毒、- -很羡慕那种编译模式啊、就是 XP 的记事本的编译模式、我咋整都整不出来……、  
话说其实我怀疑主程序是生成自解压包那种程序做出来的、……  
\- -病毒启动部分分析完毕、其实这只是一个外壳而已、做坏事的都是从 CAB 包里释放出来的垃圾  
0006FE20   01014C88  ASCII "MSCF"  
一点点关于 idb 里函数的说明：  
s\_strnXXX 系列函数就是多一个'\\\\'这个玩艺  
释放出来的 js2.exe 负责创建僵尸进程 services.exe 然后安装服务、启动。  
僵尸进程自然是不好调试、我们要做点手脚、把它的入口 OOXX 一下  
首先我们要了解 401730 这个线程负责初始化程序 IAT、所以去执行核心前必须先用它初始化一些必要的函数  
00401E90 这个函数就是 401730 这个线程里唯一有用的东西  
第二点就是程序的核心函数，可以在主线程 WndProc 的 401B00 里找到，就是 401910 这个线程  
所以我们的新入口代码也就构建好了  
call 00401E90  
jmp 401910  
把这两句覆盖到入口、这样就可以在虚拟机里方便的调试病毒核心了  
这里笔记一点关于 js2 的调试记录  

引用: 004019FC  call    004018F0  
00401A01  add     esp, 4  
00401A04  inc     esi  
00401A05  push    esi  
00401A06  call    <DecodeProc>  
00401A0B  mov     esi, eax  
00401A0D  add     esp, 4  
00401A10  test    esi, esi  
00401A12  jnz     short 00401A1E  
00401A14  push    -2  
00401A16  call    004018F0  
00401A1B  add     esp, 4  
00401A1E  mov     eax, dword ptr \[404400\]  
00401A23  push    eax  
00401A24  push    00404408                  ;  ASCII "WCCCRXcIKDfnxnQyYmJydDgmd6LzEmGQ=="  
00401A29  push    esi  
00401A2A  call    <InitString>  
00401A2F  add     esp, 4  
00401A32  push    eax  
00401A33  push    edi  
00401A34  call    <InitString>              ;  服务的说明字符串  
00401A39  add     esp, 4  
00401A3C  push    eax  
00401A3D  call    00402050                  ;  创建服务，启动服务、分析到此结束。 

剩下的就是那个远程控制 Dll 的分析  
启动的新服务行为：键盘记录、屏幕捕捉、剪贴板读取/设置、屏幕控制、……  
基本上没有什么新意、网上应该能找到很多类似于这些功能的代码吧……- -我就偷个懒，这里就不分析了……  
好了、本木马分析到此结束。  
附件里是病毒释放出的所有文件还有 idb 分析文件，附带一张 IDA 生成的核心 Dll 的流程图，请用 wingraph32 打开。  
  
  
  
另外病毒样本源自这里  
  
[`www.52pojie.cn/thread-59402-1-1.html`](http://www.52pojie.cn/thread-59402-1-1.html)  
  
  
附件  
[`u.115.com/file/f2e5187c63`](http://u.115.com/file/f2e5187c63)  
  
附件密码  
Azure\[LCG\]  
  
  
最后贴一点在 VirusTotal 上的扫描结果……  

引用: File already submitted: The file sent has already been analysed by VirusTotal in the past. This is same basic info regarding the sample itself and its last analysis:   
MD5: e4ae1102c9e22fb3e20644d18fbd8462   
Date first seen: 2010-05-17 23:10:20 (UTC)   
Date last seen: 2010-05-17 23:10:20 (UTC)   
Detection ratio: 30/40  

\- -看来病毒作者应该已经在生成这个东西的时候提交过一次、想看看有多少杀毒软件可以查出来。  

引用: File name: 1.dll  
Submission date: 2010-08-22 04:56:44 (UTC)  
Current status: queued queued analysing finished  
  
Result: 34/ 40 (85.0%) 

重新扫描的结果  
  

引用: AhnLab-V3 2010.08.22.00 2010.08.21 Win-Trojan/Securisk   
AntiVir 8.2.4.38 2010.08.20 TR/Spy.Gen   
Antiy-AVL 2.0.3.7 2010.08.16 Backdoor/Win32.Torr.gen   
Authentium 5.2.0.5 2010.08.22 W32/OnlineGames.EI.gen!Eldorado   
Avast 4.8.1351.0 2010.08.22 Win32:Yonsole   
Avast5 5.0.332.0 2010.08.22 Win32:Yonsole   
AVG 9.0.0.851 2010.08.21 BackDoor.Generic12.AMPZ   
BitDefender 7.2 2010.08.22 Trojan.Generic.3769058   
CAT-QuickHeal 11.00 2010.08.21 Backdoor.Torr.bmm  

\- -差不多是个杀软就能扫出来……  
  

引用: File already submitted: The file sent has already been analysed by VirusTotal in the past. This is same basic info regarding the sample itself and its last analysis:   
MD5: ac8dc6336bc1b972f6a1f389960df4b0   
Date first seen: 2010-05-14 18:14:42 (UTC)   
Date last seen: 2010-08-20 02:53:25 (UTC)   
Detection ratio: 34/40   
What do you wish to do?  

主程序扫描结果……
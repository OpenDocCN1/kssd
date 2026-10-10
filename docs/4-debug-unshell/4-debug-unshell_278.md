

# 菜鸟脱 Armadillo CopyMem-II 壳

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-545.htm`](https://www.pediy.com/kssd/pediy08/pediy8-545.htm)

软件名称： 幼儿拼图识物  
下载地址：  [`yingjia.hostrocket.com/jigsaw/index.htm`](http://yingjia.hostrocket.com/jigsaw/index.htm)  
软件大小：  2.6 MB  
软件语言：  中文  
软件类别：  国内软件 / 共享版   
应用平台：  Win9x/NT/2000/XP  
简介： 通过这个软件不仅可以锻炼孩子的观察力、耐性、记忆力，而且还可以了解更多的事物。适合于 3 到 10 岁的儿童使用。  
调试环境：Win2003、OllyDBG、PEiD、LordPE、ImportREC  
作者声明：只是感兴趣，没有其他目的，失误之处请大侠多多赐教。由于 ARM 脱文较多，前面过程从略，要点在后面脱壳后的输入表修复。  
脱壳过程：  
一、寻找 OEP 和 DUMP 进程  
下断 BP WaitForDebugEvent，断下后堆栈如下：  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 0012DAA8   004E1DD7  /CALL 到 WaitForDebugEvent 来自 jigsaw.004E1DD1
> 0012DAAC   0012EB84  |pDebugEvent = 0012EB84
> 0012DAB0   000003E8  \\Timeout = 1000. ms
> 
> * * *
> 
> 
> ```

  
再 BP WriteProcessMemory  
经过一个对话框后是:  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 0012EB8C  E4 0E 00 00 01 00 00 80  ?....€
> 0012EB94  00 00 00 00 00 00 00 00  ........
> 0012EB9C  EC F7 4A 00 02 00 00 00  祺 J....
> 0012EBA4  00 00 00 00 EC F7 4A 00  ....祺 J.
> 故 oep=004AF7EC
> 
> * * *
> 
> 
> ```

  
ALT＋F9 返回,搜索所有命令“or eax,FFFFFFF8”，向上找到这里：  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 004E23FA   > \\83BD D0F5FFFF>CMP DWORD PTR SS:\[EBP-A30\],0 //清 0
> 004E2401   .  0F8C A9020000 JL jigsaw.004E26B0
> 004E2407   .  8B8D D0F5FFFF MOV ECX,DWORD PTR SS:\[EBP-A30\]
> 004E240D   .  3B0D BCE75000 CMP ECX,DWORD PTR DS:\[50E7BC\]
> 004E2413   .  0F8D 97020000 JGE jigsaw.004E26B0
> ……
> 004E24CD   .  25 FF000000   AND EAX,0FF  //这里开始 PATCH
> 004E24D2   .  85C0          TEST EAX,EAX
> 
> * * *
> 
> 
> ```

  
补丁代码为：  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 004E24CD      FF85 D0F5FFFF INC DWORD PTR SS:\[EBP-A30\]
> 004E24D3      C705 C0E75000>MOV DWORD PTR DS:\[50E7C0\],1
> 004E24DD    ^ E9 18FFFFFF   JMP jigsaw1.004E23FA
> 
> * * *
> 
> 
> ```

  
在 4E26B0 下断后，运行，DUMP  
  
二、IAT 修复：  
将 DUMP 出来的程序 OEP 改为 AF7EC，然后用 OD 载入，跟踪几步就可以找到 IAT  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 00401278  - FF25 28824B00   JMP DWORD PTR DS:\[4B8228\]
> 0040127E    8BC0            MOV EAX,EAX
> 00401280  - FF25 24824B00   JMP DWORD PTR DS:\[4B8224\]
> 00401286    8BC0            MOV EAX,EAX
> 00401288  - FF25 20824B00   JMP DWORD PTR DS:\[4B8220\]
> 
> * * *
> 
> 
> ```

  
  
IAT 地址 4B817C，RVA=B817C   
重新载入主程序，BP DebugActiveProcess  中断后看堆栈：  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 0012DAC4    004C1BDB   /CALL 到 DebugActiveProcess 来自 jigsaw.004C1BD5
> 0012DAC8    00000F2C   \\ProcessId = F2C
> 
> * * *
> 
> 
> ```

  
新开一个 OllyDbg，附加进程 ID F2C 的子进程（每次子进程 ID 会不同）  
F9，再 F12，会暂停在 EP 处：  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 004AF7EC >- EB FE           JMP SHORT dumped\_.004AF7EC //改为 55 8B
> 004AF7EE    EC              IN AL,DX                                
> 004AF7EF    83C4 EC         ADD ESP,-14
> 
> * * *
> 
> 
> ```

  
  
用脚本转成单进程(这脚本不错，很方便用，转自看雪 csjwaman 的文章）  
  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> //转单进程脚本
> msg "请忽略所有异常，并添加忽略 C000001E 异常，然后运行本脚本！"
> gpa "OpenMutexA","kernel32.dll"
> bp $RESULT
> esto
> exec
> pushad
> pushfd
> push edx
> xor eax,eax
> push eax
> push eax
> call kernel32.CreateMutexA
> popfd
> popad
> jmp kernel32.OpenMutexA
> ende
> bc eip
> msg "现已转换成单进程！"
> ret
> 
> * * *
> 
> 
> ```

  
下面 F9 一下，出现对话框时 HE GetModuleHandleA， 再点确定，Shift+F9 运行一下，断下后修改魔术跳转  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 00CA97CD    8B0D C04CCD00   MOV ECX,DWORD PTR DS:\[CD4CC0\]
> 00CA97D3    89040E          MOV DWORD PTR DS:\[ESI+ECX\],EAX
> 00CA97D6    A1 C04CCD00     MOV EAX,DWORD PTR DS:\[CD4CC0\]
> 00CA97DB    393C06          CMP DWORD PTR DS:\[ESI+EAX\],EDI
> 00CA97DE    75 16           JNZ SHORT 00CA97F6
> 00CA97E0    8D85 B4FEFFFF   LEA EAX,DWORD PTR SS:\[EBP-14C\]
> 00CA97E6    50              PUSH EAX
> 00CA97E7    FF15 D0B0CC00   CALL DWORD PTR DS:\[CCB0D0\]               ; kernel32.LoadLibraryA
> 00CA97ED    8B0D C04CCD00   MOV ECX,DWORD PTR DS:\[CD4CC0\]
> 00CA97F3    89040E          MOV DWORD PTR DS:\[ESI+ECX\],EAX
> 00CA97F6    A1 C04CCD00     MOV EAX,DWORD PTR DS:\[CD4CC0\]
> 00CA97FB    393C06          CMP DWORD PTR DS:\[ESI+EAX\],EDI
> 00CA97FE    0F84 AD000000   JE 00CA98B1       //修改这里为 JMP
> 
> * * *
> 
> 
> ```

  
F9 运行，异常暂停  
用 IMR 修复一下 IAT ，RVA=B817C，Size=1000,Oep=F7EC，剪去无效指针后得到 IAT 表，然后保存树文件（下面要用到）修复 DUMP。  
  
三、修复  
运行后出现错误，跟踪了一下,运行到这里出错，  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 004A69FC  - FF25 7C884B00   JMP DWORD PTR DS:\[4B887C\]
> 
> * * *
> 
> 
> ```

  
  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 004B886C  00CAA6CA
> 004B8870  774EEA5C  ole32.CoCreateGuid
> 004B8874  774ED78D  ole32.StringFromGUID2
> 004B8878  00CAA742
> 004B887C  00CC6ED5//这里
> 004B8880  00CC7184//还有这里,两个函数指针不正确
> 004B8884  00CAA73D
> 004B8888  6E72656B
> 004B888C  32336C65
> 004B8890  6C6C642E
> 
> * * *
> 
> 
> ```

  
按第二步重来一次，修改完魔术跳转后不取消 GetModuleHandleA 断点，数据窗口定位到 004B8870，不断按 F9 直到写入输入表指针 004B8870 完成，但 004B887C 未写入指针时停下，此时按 Alt+M,打开内存窗口，搜索字串 CoCreateGuid 会找到写输入表用到的函数名，可以找到两个函数 Environ 和 ExpireCurrentKey，这两个指针是调用 ArmAccess.dll 的两个函数，我以前装过 Armadillo4，于是把 ArmAccess.dll 复制到文件所在目录   
  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 00D7B9E9  00 00 00 43 6F 43 72 65 61 74 65 47 75 69 64 00  ...CoCreateGuid.
> 00D7B9F9  53 74 72 69 6E 67 46 72 6F 6D 47 55 49 44 32 00  StringFromGUID2.
> 00D7BA09  00 61 72 6D 61 63 63 65 73 73 2E 64 6C 6C 00 7C  .armaccess.dll.|
> 00D7BA19  88 0B 00 02 00 00 00 45 6E 76 69 72 6F 6E 00 45  ?....Environ.E
> 00D7BA29  78 70 69 72 65 43 75 72 72 65 6E 74 4B 65 79 00  xpireCurrentKey.
> 
> * * *
> 
> 
> ```

  
  
用记事本打开以前保存的 IAT 文件，在最后添上两个指针名，重新载入树文件，再修复转储文件，OK 了。  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> FThunk: 000B887C  NbFunc: 00000002
> 1  000B887C  ArmAccess.dll  0011  Environ
> 1  000B8880  ArmAccess.dll  0144  ExpireCurrentKey
> 
> * * *
> 
> 
> ```

  
最后，再次感谢 FLY 的指导。 

 4dd466d0e1e0cc123afbe504da2462ca
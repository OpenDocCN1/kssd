

# Thinstall V2.501

> 原文：[`www.pediy.com/kssd/pediy06/pediy6905.htm`](https://www.pediy.com/kssd/pediy06/pediy6905.htm)

软件类别：  国外软件 / 共享版 / 安装制作  
应用平台：  Win9x/NT/2000/XP  
实例下载：点击此下载   
推荐等级：  \*\*\*\*  
开 发 商：  [`thinstall.com/`](http://thinstall.com/)  
软件介绍：  它可以把程序所需要的 DLL 和 OCX 都打到一个 EXE 中，然后在运行的时候自动将 DLL、OCX 释放出来。这个工具看起来不错，比较方便，但损失了一些性能，而且做成的 EXE 体积很大。只有当你的用户希望运行你的程序，而又不希望安装它（例如编写一个演示程序），你可以考虑这个工具。  
                 
【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  
               
【调试环境】：WinXP、Ollydbg V1.10、PEiD、LordPE、WinHex  
               
—————————————————————————————————   
【脱壳过程】：  
            
           
Thinstall 其实是捆绑壳，但是比 PEBundle 之类要强。Thinstall 可以包装.NET、Java、VB 等程序。  
新版的 Thinstall 不对外提供公开下载，运行时需要连接 License Server 或者使用 USB Key。  
用 Thinstall 加壳 Win98 的 Notepad 作为试炼品吧。  
  
放个 PEiD 的 sign：  
\[Thinstall V2.4X-2.5X -> Jitit\]  
signature = 55 8B EC B8 ?? ?? ?? ?? BB ?? ?? ?? ?? 50 E8 00 00 00 00 58 2D ?? ?? ?? ?? B9 ?? ?? ?? ?? BA ?? ?? ?? ?? BE ?? ?? ?? ?? BF ?? ?? ?? ?? BD ?? ?? ?? ?? 03 E8  
ep\_only = true  
—————————————————————————————————  
一、前奏  
               
       
设置 Ollydbg 忽略所有异常选项。老规矩：用 IsDebug V1.4 插件去掉 Ollydbg 的调试器标志。  
  
00401A95     55                push ebp  
//进入 Ollydbg 后暂停在这  
00401A96     8BEC              mov ebp,esp  
00401A98     B8 3C1A4A7D       mov eax,7D4A1A3C  
00401A9D     BB ED08B7F0       mov ebx,F0B708ED  
00401AA2     50                push eax  
00401AA3     E8 00000000       call Notepad.00401AA8  
  
下断：BP GetModuleHandleA  
中断后取消断点，Alt+F9 返回  
  
00401A66     FF15 E4534000     call dword ptr ds:\[<&KERNEL32.GetModuleHandleA>\]  
//返回这里  
00401A6C     50                push eax ; Notepad.00400000  
  
Ctrl+F 搜索命令：jmp eax  
  
00401A8B     8B45 FC           mov eax,dword ptr ss:\[ebp\-4\]  
00401A8E     FFE0              jmp eax ; 7FF79E04  
//找到这里，F4 过来  
  
现在来到 Thinstall 的核心处理段：  
  
7FF79E04     55                push ebp  
7FF79E05     8BEC              mov ebp,esp  
7FF79E07     6A FF             push -1  
7FF79E09     68 70BEF87F       push 7FF8BE70  
7FF79E0E     68 A09BF77F       push 7FF79BA0  
7FF79E13     64:A1 00000000    mov eax,dword ptr fs:\[0\]  
7FF79E19     50                push eax  
7FF79E1A     64:8925 00000000  mov dword ptr fs:\[0\],esp  
7FF79E21     83EC 10           sub esp,10  
7FF79E24     53                push ebx  
7FF79E25     56                push esi  
7FF79E26     57                push edi  
7FF79E27     8965 E8           mov dword ptr ss:\[ebp\-18\],esp  
7FF79E2A     FF15 7062F87F     call dword ptr ds:\[7FF86270\] ; kernel32.GetVersion  
  
  
—————————————————————————————————  
二、获取加壳前程序的 PE Header 等数据  
  
  
Ctrl+S 搜索命令序列：  
mov eax,dword ptr ds:\[eax+3C\]  
mov ecx,dword ptr ss:\[ebp\-118\]  
找到在 7FF427BA 处，直接 F4 过去  
  
7FF427A5     8B85 D4FDFFFF     mov eax,dword ptr ss:\[ebp\-22C\]  
7FF427AB     8B40 18           mov eax,dword ptr ds:\[eax+18\]  
7FF427AE     8985 E8FEFFFF     mov dword ptr ss:\[ebp\-118\],eax  
7FF427B4     8B85 E8FEFFFF     mov eax,dword ptr ss:\[ebp\-118\]  
7FF427BA     8B40 3C           mov eax,dword ptr ds:\[eax+3C\]  
//F4 到这里  
7FF427BD     8B8D E8FEFFFF     mov ecx,dword ptr ss:\[ebp\-118\]  
7FF427C3     8D4401 18         lea eax,dword ptr ds:\[ecx+eax+18\]  
7FF427C7     8985 D0FDFFFF     mov dword ptr ss:\[ebp\-230\],eax  ; Notepad.004000E8  
//在这里可以得到 PE Header 和 The Section Table 数据 ★  
7FF427CD     C705 D05DF97F 020>mov dword ptr ds:\[7FF95DD0\],2  
  
PE Header：  
004000D0  50 45 00 00 4C 01 01 00 D6 57 5A 35 00 00 00 00  PE..L.諻 Z5....  
004000E0  00 00 00 00 E0 00 0E 01 0B 01 03 0A 00 40 00 00  ....?..@..  
004000F0  00 70 00 00 00 00 00 00 CC 10 00 00 00 10 00 00  .p......?.....  
00400100  00 50 00 00 00 00 40 00 00 10 00 00 00 10 00 00  .P....@.......  
00400110  04 00 00 00 00 00 00 00 04 00 00 00 00 00 00 00  ..............  
00400120  C6 CA 00 00 00 04 00 00 00 00 00 00 02 00 00 00  剖............  
00400130  00 00 10 00 00 10 00 00 00 00 10 00 00 10 00 00  ............  
00400140  00 00 00 00 10 00 00 00 00 00 00 00 00 00 00 00  ...............  
00400150  00 60 00 00 8C 00 00 00 00 70 00 00 C8 42 00 00  .\`..?...p..菳..  
…… ……  
  
毕竟只是捆绑壳，此时代码已经解压，现在 dump 则 IAT 等信息都是未加密的，正是脱壳的最佳时机！  
  
  
—————————————————————————————————  
三、Dump、PE 修正  
  
  
如果你用 LordPE 来 dump，会发现得到的 dumped.exe 连图标也没有了，别急，用 WinHex 把上面得到的 PE Header 和 The Section Table 等数据写入相应部分，这样就可以运行啦。你也可以用插件 OllyDump 先生去掉“重建输入表”选项来直接抓取，修改 OEP，把 BoundImport 清 0。  
  
但是依旧只显示一个.text 区段， 看看 NumberOfSections:01，把 PE 头+06H 处修改为 05，所有的区段就露面了。  
只保留 LordPE 的“Validate PE”选项，重建 PE。     
这次脱的算是稍微有点完美了。  
  
  
—————————————————————————————————  
四、飞向光明之巅  
  
  
还想看看是怎样跳 OEP 的？  
Ctrl+F 搜索命令：call dword ptr ss:\[ebp\-3B8\]  
  
7FF4289C     FF95 48FCFFFF     call dword ptr ss:\[ebp\-3B8\]; Notepad.004010CC  
//飞向光明之巅！ ^O^  
  
  
—————————————————————————————————      
                                  
         ,     \_/   
        /| \_.-~/            \\\_     ,        青春都一晌  
       ( /~   /              \\~-.\_ |\\  
       \`\\\\  \_/                \\   ~\\ )          忍把浮名   
   \_-~~~-.)  )\_\_/;;,.          \\\_  //'  
  /'\_,\\   --~   \\ ~~~-  ,;;\\\_\_\_(  (.-~~~-.        换了破解轻狂  
 \`~ \_( ,\_..--\\ (     ,;'' /    ~--   /.\_\`\\   
  /~~//'   /' \`~\\         ) /--..\_, )\_  \`~  
  "  \`~"  "      \`"      /~'\`\\    \`\\\\~~\\     
                         "     "   "~'  ""  
      
              UnPacked By :   fly  
               2004-12-08 14:00

 4dd466d0e1e0cc123afbe504da2462ca
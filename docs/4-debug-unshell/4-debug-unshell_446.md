

# ASProtect 1.23beta21

> 原文：[`www.pediy.com/kssd/pediy06/pediy6959.htm`](https://www.pediy.com/kssd/pediy06/pediy6959.htm)

【脱文作者】 weiyi75\Dfcg\   
  
【作者邮箱】 weiyi75@sohu.com   
  
【作者主页】 Dfcg 官方大本营   
  
【使用工具】 Ollydbg1.09d,Imprec1.42,LoadPe   
  
【破解平台】 Win2000/XP   
  
【软件名称】 Aspack2.12 主程序   
  
【下载地址】 http://www.aspack.com/files/aspack212.zip   
  
【软件简介】    
  
   ASPack is an advanced Win32 executable file compressor, capable of reducing the file size of 32-bit Windows programs by as much as 70%. (ASPack compression ratio improves upon the industry-standard zip file format by as much as 10-20%.) ASPack makes Windows 95/98/NT programs and libraries smaller, and decrease load times across networks, and download times from the internet; it also protects programs against reverse engineering by non-professional hackers. Programs compressed with ASPack are self-contained and run exactly as before, with no runtime performance penalties.    
  
【软件大小】 293k   
  
【加壳方式】 ASProtect 1.22 - 1.23 Beta 21 -> Alexey Solodovnikov   
  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）   
  
\--------------------------------------------------------------------------------   
  
【破解内容】   
  
前言，关于这个软件用 SDK 紧密结合壳，使脱壳文件不能直接运行。Fpx 原来写出一篇节点修复文章，这里 Jeffzhang 兄提示有另外办法修复，我学会后觉的对初学者有用，故发布此文。Jeffzhang 真是脱壳高手，吾望尘莫及。   
  
好，脱壳开始。   
  
这个版本的 Asprotect 对 Nt 调试器不检测，Softice 严格校验。OD 异常设置不忽略内存异常，其余全部忽略,载入程序。   
  
00401000 >  68 01804600     PUSH ASPack.00468001  停在这里,F9 运行。   
00401005    E8 01000000     CALL ASPack.0040100B   
0040100A    C3              RETN   
0040100B    C3              RETN   
0040100C    3B92 7724E04E   CMP EDX,DWORD PTR DS:\[EDX+4EE02477\]   
00401012    04 84           ADD AL,84   
00401014    3D 62110889     CMP EAX,89081162   
00401019    1905 9080A438   SBB DWORD PTR DS:\[38A48090\],EAX   
0040101F    45              INC EBP   
00401020    E1 09           LOOPDE SHORT ASPack.0040102B   
......................................................................   
  
内存异常。   
  
009E0739    3100            XOR DWORD PTR DS:\[EAX\],EAX   
009E073B    EB 01           JMP SHORT 009E073E   
009E073D    68 648F0500     PUSH 58F64   
009E0742    0000            ADD BYTE PTR DS:\[EAX\],AL   
009E0744    00EB            ADD BL,CH   
009E0746    02E8            ADD CH,AL   
009E0748    0158 68         ADD DWORD PTR DS:\[EAX+68\],EBX   
009E074B    F8              CLC   
009E074C  ^ E2 9D           LOOPD SHORT 009E06EB   
......................................................................   
  
继续 Shift+F9 32 次忽略异常到最后一次异常处。   
  
009E00E1    3100            XOR DWORD PTR DS:\[EAX\],EAX   最后一次异常。   
009E00E3    64:8F05 0000000>POP DWORD PTR FS:\[0\]  //  在此处下断，Shift+F9 中断到这句,然后清除断点    
009E00EA    58              POP EAX   
009E00EB    833D DC399E00 0>CMP DWORD PTR DS:\[9E39DC\],0   
009E00F2    74 14           JE SHORT 009E0108   
009E00F4    6A 0C           PUSH 0C   
009E00F6    B9 DC399E00     MOV ECX,9E39DC   
009E00FB    8D45 F8         LEA EAX,DWORD PTR SS:\[EBP\-8\]   
009E00FE    BA 04000000     MOV EDX,4   
009E0103    E8 94C3FFFF     CALL 009DC49C   
009E0108    FF75 FC         PUSH DWORD PTR SS:\[EBP\-4\]   
009E010B    FF75 F8         PUSH DWORD PTR SS:\[EBP\-8\]   
009E010E    8B45 F4         MOV EAX,DWORD PTR SS:\[EBP\-C\]   
009E0111    8338 00         CMP DWORD PTR DS:\[EAX\],0   
009E0114    74 02           JE SHORT 009E0118   
009E0116    FF30            PUSH DWORD PTR DS:\[EAX\]   
009E0118    FF75 F0         PUSH DWORD PTR SS:\[EBP\-10\]   
009E011B    FF65 EC         JMP DWORD PTR SS:\[EBP\-14\]   这里就跳走了。   
009E011E    5F              POP EDI   
009E011F    5E              POP ESI   
009E0120    5B              POP EBX   
009E0121    8BE5            MOV ESP,EBP   
009E0123    5D              POP EBP   
009E0124    C3              RETN       注意这里不能下断点啊，不同于 Asprotect1.23rc4   
......................................................................   
  
接着按 ALT+M 打开内存镜像。在第一个块上下内存访问断点。   
  
内存镜像   
   R         R   
00400000   00001000   ASPack                PE header     Imag   R         RWE   
00401000   00042000   ASPack                code          Imag   R         RWE   //在此处上下内存    
  
访问断点   
......................................................................   
  
  
按 F9 运行，程序就会中断在 OEP 处了。   
  
0044289C    55              PUSH EBP   //中断在这里，现在就可以 DUMP 了。    
0044289D    8BEC            MOV EBP,ESP   
0044289F    83C4 F4         ADD ESP,-0C   
004428A2    E8 4D0BFCFF     CALL ASPack.004033F4   
004428A7    E8 0C21FCFF     CALL ASPack.004049B8   
004428AC    E8 6354FCFF     CALL ASPack.00407D14   
004428B1    E8 D6C6FCFF     CALL ASPack.0040EF8C   
  
  
接着用 Import REC 填入 oep 4289C ,iat 自动搜索-获得输入信息-显示无效的,右键先用追踪层次 1 修复大部分指针，剩下 8 个指针用 Asprotect1.2X 插件修复。   
  
运行修复后的程序出错，当然不可能这么简单了。所以接下来的工作就是寻找出错的位置然后修复。用 OD 载入修复后的程序，来到这里。   
  
00442901  |.  E8 D210FEFF   CALL Dump\_.004239D8   
00442906  |.  BA 38294400   MOV EDX,Dump\_.00442938                      
0044290B  |.  A1 30564400   MOV EAX,DWORD PTR DS:\[445630\]   
00442910  |.  E8 DF0DFEFF   CALL Dump\_.004236F4   
00442915      FF15 0C494400 CALL DWORD PTR DS:\[44490C\]  //这里出错，信息框提示 DS:\[0044490C\]    
  
\=0098C9A0，这里是调用壳中的代码，壳被脱了，当然出错，所以要知道它究竟是调用什么内容，只能跟踪未脱壳的程序相应位置。   
  
用 OD 重新加载未脱壳的程序，重复上叙步骤。直到 OEP 处停下。   
  
00442915    FF15 0C494400   CALL DWORD PTR DS:\[44490C\]  //按 F4 来到这里，接着 F7 跟入    
  
0098C9A0    833D A8359900 0>CMP DWORD PTR DS:\[9935A8\],0  //来到这里    
0098C9A7    74 06           JE SHORT 0098C9AF   
0098C9A9    FF15 A8359900   CALL DWORD PTR DS:\[9935A8\]  //DS:\[009935A8\]=004427A8，调用 004427A8 处的代码   ; ASPack.004427A8    
0098C9AF    C3              RETN   
  
看到这里大家应该都知道该怎样修复了吧。在转储窗口，Ctrl+G 来到 0044490C   
  
0044490C  A0 C9 98 00     //原先值    
  
0044490C  A8 27 44 00     //修改值    
  
保存为 Unpack1.exe,试运行，确定一个错误提示，英文界面，还不能加壳程序。   
  
继续查找出错的位置   
  
  
0043F1B8      FF15 04494400 CALL DWORD PTR DS:\[444904\]  //这里出错，DS:\[00444904\]=0098C8F4,又是调用壳中的代码    
  
同理跟踪未脱壳的程序来到此处   
  
0043F1B8    FF15 04494400   CALL DWORD PTR DS:\[444904\]   //F7 进入    
  
0098C8F4    C3              RETN   //来到这里。返回到 0043F1BE    
  
晕，里面就是一个 RETN，这个也好办，随便找一个 RETN 让它返回吧。可以用 Ctrl+F 查找 RETN，我选用的是   
  
00401234 处的,随便哪个 RETN 都行。   
  
同样来到转储窗口，Ctrl+G 来到 00444904   
  
00444904  F4 C8 98 00    //原先值    
  
00444904  34 12 40 00    //修改值    
  
  
继续查找出错的位置   
  
0043F246      FF15 08494400 CALL DWORD PTR DS:\[444908\]  //DS:\[00444908\]=0098C8F4  这里也是调用壳中的代码    
  
同样跟踪未脱壳的程序来到此处   
  
0043F246    FF15 08494400   CALL DWORD PTR DS:\[444908\]  //F7 进入    
  
0098C8F4    C3              RETN   //来到这里。    
  
这里和上面修改一样。   
  
00444908  F4 C8 98 00    //原先值    
  
00444908  34 12 40 00    //修改值    
  
到这里就是彻底结束了，可以正常运行了。   
  
  
  
  
【破解总结】   
  
  
在转储窗口中修改   
  
00444904  F4 C8 98 00           //原先值    
  
00444904  34 12 40 00           //修改值    
  
  
00444908  F4 C8 98 00           //原先值    
  
00444908  34 12 40 00           //修改值     
  
  
0044490C  A0 C9 98 00           //原先值     
  
0044490C  A8 27 44 00           //修改值    
 4dd466d0e1e0cc123afbe504da2462ca
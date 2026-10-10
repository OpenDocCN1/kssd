

# 北斗程序压缩(nSpack)V2.0 的快速脱壳法

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-586.htm`](https://www.pediy.com/kssd/pediy07/pediy7-586.htm)

北斗程序压缩(nSpack)V2.0 的快速脱壳法+OllyMachine 脱壳脚本  
     
          
  
下载页面：  http://www.nsdsn.com/intr.htm  
软件大小：  320 KB    
软件简介：  首款国产(Windows95/98/2000/NT/XP/2003)EXE、DLL、OCX 等 PE 文件压缩工具，通过压缩代码、数据、相关资源使压缩能达到 60-70%。能够处理 32 位和 64 位的可执行文件（exe,dll,ocx）。可以很好的处理程序的代码、数据、资源节。可以压缩 C#.net 的可执行文件。界面更加美观漂亮，内核更加高效，兼容性更好。压缩后的程序在网络上可减少程序的加载和下载时间。完全支持常用压缩软件如：upx,aspack,pecompact 等压缩过的程序上再次进行压缩。独有的最大压缩方式，使得程序体积变得更小。支持目录和多文件的压缩操作。  
  
【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教  
               
【调试环境】：WinXP、OllyDbg、PEiD、LordPE、ImportREC  
               
—————————————————————————————————   
【脱壳过程】：  
  
  
nSpack 是国产的压缩壳，兼容性不错。期待其保护壳 NSProtect 的发布。  
下面的脱壳方法可以说是目前北斗程序压缩所有版本的通用快速脱壳法。  
—————————————————————————————————   
一、北斗程序压缩(nSpack)V2.0 主程序脱壳  
               
               
设置 OllyDbg 忽略所有异常选项。用 IsDebug 插件去掉 OllyDbg 的调试器标志。  
  
0040101B    E9 77511200        jmp 00526197  
//进入 OllyDbg 后暂停在这  
  
下断：BP VirtualProtect  
Shift+F9，中断后取消断点，Alt+F9 返回  
  
004DB8FC    FF95 58FCFFFF      call dword ptr ss:\[ebp-3A8\]; kernel32.VirtualProtect  
004DB902    5A                 pop edx  
//返回这里  
004DB903    5B                 pop ebx  
004DB904    59                 pop ecx  
004DB905    5E                 pop esi  
004DB906    83C3 0C            add ebx,0C  
004DB909    E2 E1              loopd short 004DB8EC  
004DB90B    61                 popad  
//直接 F4 下来  
004DB90C    9D                 popfd  
004DB90D    E9 96F7F6FF        jmp 0044B0A8  
//飞向光明之巅  
  
  
0044B0A8    55                 push ebp  
//OEP   ★  运行 LordPE 完全 Dump 这个进程  
0044B0A9    8BEC               mov ebp,esp  
0044B0AB    6A FF              push -1  
0044B0AD    68 E8EC4600        push 46ECE8  
0044B0B2    68 A89D4400        push 449DA8  
0044B0B7    64:A1 00000000     mov eax,dword ptr fs:\[0\]  
0044B0BD    50                 push eax  
0044B0BE    64:8925 00000000   mov dword ptr fs:\[0\],esp  
0044B0C5    83EC 58            sub esp,58  
0044B0C8    53                 push ebx  
0044B0C9    56                 push esi  
0044B0CA    57                 push edi  
0044B0CB    8965 E8            mov dword ptr ss:\[ebp-18\],esp  
0044B0CE    FF15 70924600      call dword ptr ds:\[469270\]; kernel32.GetVersion  
  
运行 ImportREC，选择这个进程。把 OEP 改为 0004B0A8，点 IAT AutoSearch、Get Import，FixDump  
  
————————————————————————  
脱壳后的自检验是对加壳主程序的 PE 信息进行检测，很简单。  
  
  
0040EC10    51                 push ecx  
0040EC11    A1 C82A4800        mov eax,dword ptr ds:\[482AC8\]  
0040EC16    53                 push ebx  
0040EC17    56                 push esi  
0040EC18    57                 push edi  
0040EC19    8B48 3C            mov ecx,dword ptr ds:\[eax+3C\]  
0040EC1C    8B3D D72A4800      mov edi,dword ptr ds:\[482AD7\]  
0040EC22    8D5401 04          lea edx,dword ptr ds:\[ecx+eax+4\]  
0040EC26    33C0               xor eax,eax  
0040EC28    66:8B42 10         mov ax,word ptr ds:\[edx+10\]  
0040EC2C    8B72 24            mov esi,dword ptr ds:\[edx+24\]  
0040EC2F    8D4A 14            lea ecx,dword ptr ds:\[edx+14\]  
0040EC32    03C1               add eax,ecx  
0040EC34    3BFE               cmp edi,esi  
0040EC36    0F85 BE000000      jnz 0040ECFA  
0040EC3C    66:8B1D D02A4800   mov bx,word ptr ds:\[482AD0\]  
0040EC43    66:3B5A 02         cmp bx,word ptr ds:\[edx+2\]  
0040EC47    0F85 AD000000      jnz 0040ECFA  
0040EC4D    8B15 DB2A4800      mov edx,dword ptr ds:\[482ADB\]  
0040EC53    8B79 68            mov edi,dword ptr ds:\[ecx+68\]  
0040EC56    3BD7               cmp edx,edi  
0040EC58    0F85 9C000000      jnz 0040ECFA  
0040EC5E    8B15 DF2A4800      mov edx,dword ptr ds:\[482ADF\]  
0040EC64    8B79 70            mov edi,dword ptr ds:\[ecx+70\]  
0040EC67    3BD7               cmp edx,edi  
0040EC69    0F85 8B000000      jnz 0040ECFA  
0040EC6F    8B51 24            mov edx,dword ptr ds:\[ecx+24\]  
0040EC72    8B48 14            mov ecx,dword ptr ds:\[eax+14\]  
0040EC75    3BCA               cmp ecx,edx  
0040EC77    895424 0C          mov dword ptr ss:\[esp+C\],edx  
0040EC7B    77 7D              ja short 0040ECFA  
0040EC7D    8B78 3C            mov edi,dword ptr ds:\[eax+3C\]  
0040EC80    8D48 28            lea ecx,dword ptr ds:\[eax+28\]  
0040EC83    3BFA               cmp edi,edx  
0040EC85    75 73              jnz short 0040ECFA  
0040EC87    33FF               xor edi,edi  
0040EC89    55                 push ebp  
0040EC8A    66:85DB            test bx,bx  
0040EC8D    76 1D              jbe short 0040ECAC  
0040EC8F    8B50 0C            mov edx,dword ptr ds:\[eax+C\]  
0040EC92    3BF2               cmp esi,edx  
0040EC94    72 09              jb short 0040EC9F  
0040EC96    8B68 08            mov ebp,dword ptr ds:\[eax+8\]  
0040EC99    03EA               add ebp,edx  
0040EC9B    3BF5               cmp esi,ebp  
0040EC9D    72 0B              jb short 0040ECAA  
0040EC9F    83C0 28            add eax,28  
0040ECA2    47                 inc edi  
0040ECA3    66:3BFB            cmp di,bx  
0040ECA6    72 E7              jb short 0040EC8F  
0040ECA8    EB 02              jmp short 0040ECAC  
0040ECAA    8BC8               mov ecx,eax  
0040ECAC    8B4424 10          mov eax,dword ptr ss:\[esp+10\]  
0040ECB0    8B51 14            mov edx,dword ptr ds:\[ecx+14\]  
0040ECB3    3BD0               cmp edx,eax  
0040ECB5    5D                 pop ebp  
0040ECB6    73 1F              jnb short 0040ECD7  
0040ECB8    8B51 0C            mov edx,dword ptr ds:\[ecx+C\]  
0040ECBB    8BC6               mov eax,esi  
0040ECBD    2BC2               sub eax,edx  
0040ECBF    74 16              je short 0040ECD7  
0040ECC1    8B0D C82A4800      mov ecx,dword ptr ds:\[482AC8\]  
0040ECC7    803C08 E9          cmp byte ptr ds:\[eax+ecx\],0E9  
0040ECCB    75 0C              jnz short 0040ECD9  
0040ECCD    8B4C08 01          mov ecx,dword ptr ds:\[eax+ecx+1\]  
0040ECD1    8D4431 05          lea eax,dword ptr ds:\[ecx+esi+5\]  
0040ECD5    EB 02              jmp short 0040ECD9  
0040ECD7    8BC6               mov eax,esi  
0040ECD9    8B15 C82A4800      mov edx,dword ptr ds:\[482AC8\]  
0040ECDF    B9 05000000        mov ecx,5  
0040ECE4    BF D22A4800        mov edi,482AD2  
0040ECE9    8D3410             lea esi,dword ptr ds:\[eax+edx\]  
0040ECEC    33D2               xor edx,edx  
0040ECEE    F3:A6              repe cmps byte ptr es:\[edi\],byte ptr ds:\[esi\]  
0040ECF0    8BC2               mov eax,edx  
0040ECF2    5F                 pop edi  
0040ECF3    5E                 pop esi  
0040ECF4    5B                 pop ebx  
0040ECF5    0F94C0             sete al  
0040ECF8    59                 pop ecx  
0040ECF9    C3                 retn  
0040ECFA    5F                 pop edi  
0040ECFB    5E                 pop esi  
0040ECFC    33C0               xor eax,eax  
0040ECFE    5B                 pop ebx  
0040ECFF    59                 pop ecx  
0040ED00    C3                 retn  
  
直接修改为下面的代码主程序就可以运行可以加壳啦：  
  
0040EC10    33C0               xor eax,eax  
0040EC12    40                 inc eax  
0040EC13    C3                 retn  
  
至于破解部分就自己找吧，改标志位就行了。  
  
  
—————————————————————————————————  
二、OllyMachine 脱壳脚本  
  
  
用罗聪的 OllyMachine 写个脱壳脚本，支持 nSpack V1.X 和最新的 V2.0，包括 nSpack.exe 主程序。  
  
//////////////////////////////////////////////  
//  
//  FileName :   nSpack.V1.X-V2.0.oms  
//  Author   :   fly  
//  Date     :   2005-05-26 24:00  
//  Comment  :   UnPacK For nSpack V1.X-V2.0  
//  
//////////////////////////////////////////////  
  
EOB Break1  
invoke msg, "UnPacK.For.nSpack.V1.X-V2.0.By:  fly \\n  2005.05.26     Debug Environment:\\nWinXP OllyDbg.V1.10 OllyMachine.V2.0 "  
  
invoke GetProcAddress, "VirtualProtect", "KERNEL32.dll"  
MOV reg01, reg00  
invoke BP, reg01  
  
ESTO  
  
Break1:  
RunToUserCode  
invoke BC, reg01  
  
  
EOB Break2  
invoke Find, eip, "619DE9"  
cmp reg00,-1  
je NoFind  
  
MOV reg01, reg00  
invoke BP, reg01  
  
run  
halt  
  
NoFind:  
invoke BC, reg01  
invoke msg,"      Error!\\nMaybe it isn't nSpack"  
halt  
  
Break2:  
invoke BC, reg01  
invoke StepOverS, 3  
  
invoke msg, "    Just : OEP !\\n  Dump and Fix IAT  \\n ☆  Good Lucky  ☆  ""  
  
               
—————————————————————————————————                                   
         ,     \_/   
        /| \_.-~/            \\\_     ,        青春都一晌  
       ( /~   /              \\~-.\_ |\\  
       \`\\\\  \_/                \\   ~\\ )          忍把浮名   
   \_-~~~-.)  )\_\_/;;,.          \\\_  //'  
  /'\_,\\   --~   \\ ~~~-  ,;;\\\_\_\_(  (.-~~~-.        换了脱壳轻狂  
 \`~ \_( ,\_..--\\ (     ,;'' /    ~--   /.\_\`\\   
  /~~//'   /' \`~\\         ) /--..\_, )\_  \`~  
  "  \`~"  "      \`"      /~'\`\\    \`\\\\~~\\     
                         "     "   "~'  ""  
      
              UnPacKed By :  fly  
               2005-05-26 24:00 

 4dd466d0e1e0cc123afbe504da2462ca
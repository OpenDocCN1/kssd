

# RPXP V1.0.0.1

> 原文：[`www.pediy.com/kssd/pediy06/pediy6692.htm`](https://www.pediy.com/kssd/pediy06/pediy6692.htm)

                             
下载页面：  [`www.enterbrain.co.jp/digifami/`](http://www.enterbrain.co.jp/digifami/)   
软件大小：  27.6 M    
软件简介：  日本的 rpg 游戏。    
               
【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  
               
【调试环境】：WinXP、Ollydbg V1.10、PEiD、LordPE、ImportREC  
               
—————————————————————————————————   
【脱壳过程】：  
            
           
             
感觉 ASProtect V2.0 比 ASProtect V1.31 build 06.14 要简单，是作者加壳时没有使用最强选项？还是 Alexey Solodovnikov 放水？不得知了。至少 RPGXP.exe 没有 Stolen Code，没有自校验，也没有 IAT 混乱，简直比 ASProtect V1.23RC4 还简单。  
             
最近比较烦，一直没时间看壳。  
入秋了，酷暑已然消亡。今夜月圆，夜凉如水，秋高气爽。心情也好了不少。  
朋友让看看这个程序，所以在这深夜凌晨静下心来脱这个壳。  
近来写脱壳教程的朋友少了，略记过程，聊以充数，各位兄弟见笑了。  
             
另外：程序是朋友传给我的，不懂日语，不知道 EXE 在其主页上的下载地址。  
—————————————————————————————————   
一、壳代码解压  
             
             
老规矩：用 IsDebug V1.4 插件去掉 Ollydbg 的调试器标志。  
设置：事件->标记首先暂停在->系统断点  
         
因为程序的支持文件 rgss100j.dll（\\WINDOWS\\System32\\目录下）加了多层壳，其里面也是 ASProtect 壳，所以我们设置 Ollydbg 忽略除所有的异常选项。  
             
77F7F571     C3                  retn  
//进入 OD 后暂停在系统断点  
             
Shift+F9，程序加载 rgss100j.dll 后弹出入口点警告，点“否”。  
如果你在这里就弹出“Debugger detected”，Game Over 的话，请清除所有的 UDD 文件，然后按上面的步骤重新来过。  
             
00401000     68 01106500         push RPGXP.00651001  
//暂停在 EXE 的壳入口处  
00401005     E8 01000000         call RPGXP.0040100B  
0040100A     C3                  retn  
             
下断：HE GetModuleHandleA  
Shift+F9，中断 2 次后，就可以取消断点，Alt+F9 返回 044994AC 了。  
看过《用 Ollydbg 手脱 ASProtect V1.23RC4 加壳的 DLL》的朋友应该熟悉下面的代码了。  
这段代码采用了 ASPack 引擎，解压出壳代码。  
Ctrl+F9 执行到返回。  
             
044994A6     FF95 EC314400       call dword ptr ss:\[ebp+4431EC\]  
044994AC     85C0                test eax,eax ; kernel32.77E40000  
//返回这里  
044994AE     75 07               jnz short 044994B7  
044994B0     53                  push ebx  
044994B1     FF95 F0314400       call dword ptr ss:\[ebp+4431F0\]  
044994B7     8985 4D294400       mov dword ptr ss:\[ebp+44294D\],eax  
044994BD     C785 51294400 00000>mov dword ptr ss:\[ebp+442951\],0  
044994C7     8B95 D8304400       mov edx,dword ptr ss:\[ebp+4430D8\]  
044994CD     8B06                mov eax,dword ptr ds:\[esi\]  
044994CF     85C0                test eax,eax  
044994D1     75 03               jnz short 044994D6  
044994D3     8B46 10             mov eax,dword ptr ds:\[esi+10\]  
044994D6     03C2                add eax,edx  
044994D8     0385 51294400       add eax,dword ptr ss:\[ebp+442951\]  
044994DE     8B18                mov ebx,dword ptr ds:\[eax\]  
044994E0     8B7E 10             mov edi,dword ptr ds:\[esi+10\]  
044994E3     03FA                add edi,edx  
044994E5     03BD 51294400       add edi,dword ptr ss:\[ebp+442951\]  
044994EB     85DB                test ebx,ebx  
044994ED     0F84 A2000000       je 04499595  
044994F3     F7C3 00000080       test ebx,80000000  
044994F9     75 04               jnz short 044994FF  
044994FB     03DA                add ebx,edx  
044994FD     43                  inc ebx  
044994FE     43                  inc ebx  
044994FF     53                  push ebx  
04499500     81E3 FFFFFF7F       and ebx,7FFFFFFF  
04499506     53                  push ebx  
04499507     FFB5 4D294400       push dword ptr ss:\[ebp+44294D\]  
0449950D     FF95 E8314400       call dword ptr ss:\[ebp+4431E8\]  
04499513     85C0                test eax,eax  
04499515     5B                  pop ebx  
04499516     75 6F               jnz short 04499587  
04499518     F7C3 00000080       test ebx,80000000  
0449951E     75 19               jnz short 04499539  
04499520     57                  push edi  
04499521     8B46 0C             mov eax,dword ptr ds:\[esi+C\]  
04499524     0385 D8304400       add eax,dword ptr ss:\[ebp+4430D8\]  
0449952A     50                  push eax  
0449952B     53                  push ebx  
0449952C     8D85 53314400       lea eax,dword ptr ss:\[ebp+443153\]  
04499532     50                  push eax  
04499533     57                  push edi  
04499534     E9 99000000         jmp 044995D2  
04499539     81E3 FFFFFF7F       and ebx,7FFFFFFF  
0449953F     8B85 DC304400       mov eax,dword ptr ss:\[ebp+4430DC\]  
04499545     3985 4D294400       cmp dword ptr ss:\[ebp+44294D\],eax  
0449954B     75 24               jnz short 04499571  
0449954D     57                  push edi  
0449954E     8BD3                mov edx,ebx  
04499550     4A                  dec edx  
04499551     C1E2 02             shl edx,2  
04499554     8B9D 4D294400       mov ebx,dword ptr ss:\[ebp+44294D\]  
0449955A     8B7B 3C             mov edi,dword ptr ds:\[ebx+3C\]  
0449955D     8B7C3B 78           mov edi,dword ptr ds:\[ebx+edi+78\]  
04499561     035C3B 1C           add ebx,dword ptr ds:\[ebx+edi+1C\]  
04499565     8B0413              mov eax,dword ptr ds:\[ebx+edx\]  
04499568     0385 4D294400       add eax,dword ptr ss:\[ebp+44294D\]  
0449956E     5F                  pop edi  
0449956F     EB 16               jmp short 04499587  
04499571     57                  push edi  
04499572     8B46 0C             mov eax,dword ptr ds:\[esi+C\]  
04499575     0385 D8304400       add eax,dword ptr ss:\[ebp+4430D8\]  
0449957B     50                  push eax  
0449957C     53                  push ebx  
0449957D     8D85 A4314400       lea eax,dword ptr ss:\[ebp+4431A4\]  
04499583     50                  push eax  
04499584     57                  push edi  
04499585     EB 4B               jmp short 044995D2  
04499587     8907                mov dword ptr ds:\[edi\],eax  
04499589     8385 51294400 04    add dword ptr ss:\[ebp+442951\],4  
04499590     E9 32FFFFFF         jmp 044994C7  
04499595     8906                mov dword ptr ds:\[esi\],eax  
04499597     8946 0C             mov dword ptr ds:\[esi+C\],eax  
0449959A     8946 10             mov dword ptr ds:\[esi+10\],eax  
0449959D     83C6 14             add esi,14  
044995A0     8B95 D8304400       mov edx,dword ptr ss:\[ebp+4430D8\]  
044995A6     E9 EBFEFFFF         jmp 04499496  
044995AB     8B85 652A4400       mov eax,dword ptr ss:\[ebp+442A65\]  
044995B1     50                  push eax  
044995B2     0385 D8304400       add eax,dword ptr ss:\[ebp+4430D8\]  
044995B8     5B                  pop ebx  
044995B9     0BDB                or ebx,ebx  
044995BB     8985 112F4400       mov dword ptr ss:\[ebp+442F11\],eax  
044995C1     61                  popad  
044995C2     75 08               jnz short 044995CC  
044995C4     B8 01000000         mov eax,1  
044995C9     C2 0C00             retn 0C  
044995CC     68 4C1D4904         push 4491D4C  
044995D1     C3                  retn  
//执行到返回至这里  
             
             
—————————————————————————————————  
二、避开 IAT 加密  
             
             
现在壳代码已经解压完毕了。  
Ctrl+S 搜索命令序列：  
mov edx,dword ptr ss:\[ebp+C\]  
mov edx,dword ptr ds:\[edx\]  
mov dword ptr ds:\[edx\],eax  
             
共找到 4 处，全部下断。  
             
044858CF     8B55 0C             mov edx,dword ptr ss:\[ebp+C\]  
044858D2     8B12                mov edx,dword ptr ds:\[edx\]  
044858D4     8902                mov dword ptr ds:\[edx\],eax  
//需要修改  
             
0448593F     8B55 0C             mov edx,dword ptr ss:\[ebp+C\]  
04485942     8B12                mov edx,dword ptr ds:\[edx\]  
04485944     8902                mov dword ptr ds:\[edx\],eax  
//不需要修改  
             
04485953     8B55 0C             mov edx,dword ptr ss:\[ebp+C\]  
04485956     8B12                mov edx,dword ptr ds:\[edx\]  
04485958     8902                mov dword ptr ds:\[edx\],eax  
//不需要修改  
             
04485961     8B55 0C             mov edx,dword ptr ss:\[ebp+C\]  
04485964     8B12                mov edx,dword ptr ds:\[edx\]  
04485966     8902                mov dword ptr ds:\[edx\],eax  
//GetProcAddress 函数加密  
             
————————————————————————  
             
你也可以走至 OEP 后找到 IAT 位置，然后重新运行，在 IAT 处下“内存写入”断点，从而确定 IAT 加密的地方。  
Shift+F9，中断在 044858CF 处。  
             
044858CA     E8 FDF8FFFF         call 044851CC  
044858CF     8B55 0C             mov edx,dword ptr ss:\[ebp+C\]  
044858D2     8B12                mov edx,dword ptr ds:\[edx\]  
044858D4     8902                mov dword ptr ds:\[edx\],eax  
//修改为：mov dword ptr ds:\[edx\],ebx   ★  
//寄存器 EBX 中保存的就是正确的函数！  
044858D6     E9 90000000         jmp 0448596B  
             
0448596B     8B45 0C             mov eax,dword ptr ss:\[ebp+C\]  
0448596E     8300 04             add dword ptr ds:\[eax\],4  
04485971     5F                  pop edi  
04485972     5E                  pop esi  
04485973     5B                  pop ebx  
04485974     8BE5                mov esp,ebp  
04485976     5D                  pop ebp  
04485977     C2 1000             retn 10  
//返回 04485A36   
             
044859EA     AD                  lods dword ptr ds:\[esi\]  
044859EB     09C0                or eax,eax  
044859ED     74 4A               je short 04485A39  
//输入表处理完毕则跳转   ★  
044859EF     89C7                mov edi,eax  
044859F1     037D F8             add edi,dword ptr ss:\[ebp\-8\]  
044859F4     897D F4             mov dword ptr ss:\[ebp\-C\],edi  
044859F7     89F3                mov ebx,esi  
044859F9     31C9                xor ecx,ecx  
044859FB     49                  dec ecx  
044859FC     87FE                xchg esi,edi  
044859FE     30C0                xor al,al  
04485A00     F2:AE               repne scas byte ptr es:\[edi\]  
04485A02     87FE                xchg esi,edi  
04485A04     AC                  lods byte ptr ds:\[esi\]  
04485A05     80F8 00             cmp al,0  
04485A08     74 E0               je short 044859EA  
04485A0A     80F8 01             cmp al,1  
04485A0D     75 06               jnz short 04485A15  
04485A0F     8345 F4 04          add dword ptr ss:\[ebp\-C\],4  
04485A13     EB EF               jmp short 04485A04  
04485A15     53                  push ebx  
04485A16     56                  push esi  
04485A17     53                  push ebx  
04485A18     8D5D F4             lea ebx,dword ptr ss:\[ebp\-C\]  
04485A1B     53                  push ebx  
04485A1C     80F8 05             cmp al,5  
04485A1F     74 06               je short 04485A27  
04485A21     0FB60E              movzx ecx,byte ptr ds:\[esi\]  
04485A24     41                  inc ecx  
04485A25     EB 05               jmp short 04485A2C  
04485A27     B9 04000000         mov ecx,4  
04485A2C     01CE                add esi,ecx  
04485A2E     FF75 F0             push dword ptr ss:\[ebp\-10\]  
04485A31     E8 FAFDFFFF         call 04485830  
//IAT 加密  
04485A36     5B                  pop ebx  
04485A37     EB CB               jmp short 04485A04  
04485A39     5F                  pop edi  
//这里下断   ★  
04485A3A     5E                  pop esi  
04485A3B     5B                  pop ebx  
04485A3C     5A                  pop edx  
04485A3D     59                  pop ecx  
04485A3E     8BC3                mov eax,ebx  
04485A40     5B                  pop ebx  
04485A41     8BE5                mov esp,ebp  
04485A43     5D                  pop ebp  
04485A44     C3                  retn  
//循环处理输入表  
             
             
————————————————————————  
看看 04485966 处的处理  
             
0448595C     B8 B8464804         mov eax,44846B8  
04485961     8B55 0C             mov edx,dword ptr ss:\[ebp+C\]  
04485964     8B12                mov edx,dword ptr ds:\[edx\]  
04485966     8902                mov dword ptr ds:\[edx\],eax  
04485968     EB 01               jmp short 0448596B  
             
当我们中断在 04485966 处时，EAX=044846B8  
Ctrl+G:044846B8  
             
044846B8     55                  push ebp  
044846B9     8BEC                mov ebp,esp  
044846BB     8B55 0C             mov edx,dword ptr ss:\[ebp+C\]  
044846BE     8B45 08             mov eax,dword ptr ss:\[ebp+8\]  
044846C1     3B05 F8234904       cmp eax,dword ptr ds:\[44923F8>  
044846C7     75 09               jnz short 044846D2  
044846C9     8B0495 F8234904     mov eax,dword ptr ds:\[edx\*4+4>  
044846D0     EB 07               jmp short 044846D9  
044846D2     52                  push edx  
044846D3     50                  push eax  
044846D4     E8 2710FFFF         call 04475700; jmp to kernel32.GetProcAddress  
044846D9     5D                  pop ebp  
044846DA     C2 0800             retn 8  
             
所以这个函数是 GetProcAddress，把 EAX 的值修改为 GetProcAddress 的地址，WinXP 下这个地址是 77E5A5FD  
             
             
—————————————————————————————————  
三、Second 段内存断点大法，直达 OEP  
             
             
输入表处理结束后中断在 04485A39 处，现在我们不必通过最后一次异常走至 OEP 了。  
取消所有的断点。  
Alt+M，打开内存察看窗口。  
在 00401000 的第 2 个区段上设置内存访问断点。  
Shift+F9，直达 OEP！  
  
004A4F03     6A 60               push 60  
//在这儿用 LordPE 纠正 ImageSize 后完全 DUMP 这个进程  
004A4F05     68 18495600         push RPGXP.00564918  
004A4F0A     E8 49110000         call RPGXP.004A6058  
004A4F0F     BF 94000000         mov edi,94  
004A4F14     8BC7                mov eax,edi  
004A4F16     E8 25DDFFFF         call RPGXP.004A2C40  
004A4F1B     8965 E8             mov dword ptr ss:\[ebp\-18\],esp  
004A4F1E     8BF4                mov esi,esp  
004A4F20     893E                mov dword ptr ds:\[esi\],edi  
004A4F22     56                  push esi  
004A4F23     FF15 0CA45200       call dword ptr ds:\[52A40C\]; kernel32.GetVersionExA  
             
             
—————————————————————————————————  
四、修复输入表，完成脱壳  
             
             
运行 ImportREC，选择这个进程。  
修改 OEP＝000A4F03、RVA=0012A000、Size=000008D0，获取输入表，所有的函数都是有效的。  
FixDump，正常运行，脱壳成功！    
                 
          
—————————————————————————————————  
五、破解  
             
             
程序启动后弹出要求注册的 NAG，如果取消的话则程序自动退出。现在就来搞定这个 NAG。  
你可以用试炼码注册时下 BP MessageBoxA，从而得知注册窗口的位置。  
             
             
00403C2D     E8 CA1B1100         call UnPacked.005157FC  
00403C32     85C0                test eax,eax  
00403C34     0F84 84000000       je UnPacked.00403CBE  
00403C3A     396C24 2C           cmp dword ptr ss:\[esp+2C\],ebp  
00403C3E     74 7E               je short UnPacked.00403CBE  
00403C40     6A 00               push 0  
00403C42     8BCE                mov ecx,esi  
00403C44     E8 57ECFFFF         call UnPacked.004028A0  
//弹出 NAG  
00403C49     85C0                test eax,eax  
00403C4B     74 0A               je short UnPacked.00403C57  
00403C4D     C786 A0000000 01000>mov dword ptr ds:\[esi+A0\],1  
00403C57     8B56 1C             mov edx,dword ptr ds:\[esi+1C\]  
00403C5A     85D2                test edx,edx  
00403C5C     74 60               je short UnPacked.00403CBE  
             
修改为：  
00403C2D     E8 CA1B1100         call UnPacked.005157FC  
00403C32     B8 01000000         mov eax,1 ★  
00403C37     EB 10               jmp short UnPacked.00403C49 ★  
00403C39     90                  nop  
00403C3A     396C24 2C           cmp dword ptr ss:\[esp+2C\],ebp  
00403C3E     74 7E               je short UnPacked.00403CBE  
00403C40     6A 00               push 0  
00403C42     8BCE                mov ecx,esi  
00403C44     E8 57ECFFFF         call UnPacked.004028A0  
00403C49     85C0                test eax,eax  
00403C4B     74 0A               je short UnPacked.00403C57  
             
             
OK，世界清净下来，NAG 了无踪迹了。  
月明星稀，秋虫唧唧，好一个怡人的初秋之夜。  
我愿折去生命的四分之三，换取这个四分之一的爽秋！  
             
             
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
  
      
     UnPacked By 巢水工作坊——fly \[FCG\]\[NUKE\]\[DCM\]\[BCG\]\[CUG\]  
                 2004-08-31  04:00

 4dd466d0e1e0cc123afbe504da2462ca
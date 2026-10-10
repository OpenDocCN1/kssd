

# 仙剑 V0.1000

> 原文：[`www.pediy.com/kssd/pediy06/pediy6990.htm`](https://www.pediy.com/kssd/pediy06/pediy6990.htm)

【软件简介】：LiNSoN 兄弟写的壳 ![icon\_smile.gif](img/59da1937da424a7e6d4210c5b355f670.jpg)   牛！ 

【下载地址】：本地下载  
  
【作者声明】：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！   
  
【调试环境】：WinXP、Ollydbg1.09、PEiD、LordPE、ImportREC   
  
—————————————————————————————————    
【脱壳过程】：   
             
            
      
哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪   
哪                            仙剑 0.1000                             哪   
哪                                 Coded By LiNSoN\[linson@army.com\]   哪   
哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪   
  
    仅此献给所有 仙剑奇侠传的 Fans & Crackers.玩具一个,高手看罢莫笑.   
    转眼就初三了.只叹人生,些些往事,亦成流水.也罢,还是学好数理化,走遍天下都不怕-\_-;   
  
  
     
LiNSoN 兄弟写的壳，佩服呀 ![icon\_smile.gif](img/59da1937da424a7e6d4210c5b355f670.jpg) 偶肯定是好好支持的，那就是：脱了他！ ![icon\_smile.gif](img/59da1937da424a7e6d4210c5b355f670.jpg)  
  
“yoda 的 PER；ExES 的 IT Fuck Up；幻影的一个花指令；ACP 的 MeltICE 表；PLL611SM 的 Mask EP Code 和 Linker Info。是不是有点奢呀……”   
       
——这是 仙剑 V0.1000 的“装配”。呵呵，怪不得偶看到的 MeltICE 表和 ACProtect 的一模一样。而脱壳的方式和 EXEStealth、yoda's Crypter 脱壳“维妙维肖” ![icon\_smile.gif](img/59da1937da424a7e6d4210c5b355f670.jpg)  
  
仙剑 V0.1000 的最大特色是让 FI 和 PEiD 侦测为 Microsoft Visual C++ 6.0，这个蒙蔽侦壳工具的方法是 LiNSoN 独创的！   
  
兄弟才初三，感觉你很成熟了，前途无量呀！大哥偶只有佩服的份了。偶也喜欢《仙剑奇侠传》，打过 3 遍柔情版，后来空闲时间全部给了 Crack，没时间玩了 ![icon\_smile.gif](img/59da1937da424a7e6d4210c5b355f670.jpg)  
  
————————————————————————   
设置 Ollydbg 忽略所有的异常选项。老规矩：用 IsDebug 1.4 插件去掉 Ollydbg 的调试器标志。   
  
  
00404060     55                   push ebp//进入 OD 后停在这！    
                                  //下面这段代码是 linson 伪装的 VC 的入口方式     
00404061     8BEC                 mov ebp,esp   
00404063     6A FF                push -1   
00404065     68 AAAAAAAA          push AAAAAAAA   
0040406A     68 BBBBBBBB          push BBBBBBBB   
0040406F     64:A1 00000000       mov eax,dword ptr fs:\[0\]   
00404075     50                   push eax   
00404076     64:8925 00000000     mov dword ptr fs:\[0\],esp   
0040407D     83EC 44              sub esp,44   
00404080     53                   push ebx   
00404081     56                   push esi   
00404082     57                   push edi   
00404083     66:9C                pushfw   
00404085     72 08                jb short XJ01000.0040408F   
00404087     EB 01                jmp short XJ01000.0040408A   
  
  
————————————————————————   
直接下断：BP CloseHandle+6   或者直接在 CloseHandle+6 处下 硬件执行 断点 ![icon\_smile.gif](img/59da1937da424a7e6d4210c5b355f670.jpg)  
  
  
77E57963     64:A1 18000000       mov eax,dword ptr fs:\[18\]   
77E57969     8B48 30              mov ecx,dword ptr ds:\[eax+30\]   
                                  //断在这！取消断点 Ctrl+F9 执行到返回     
77E5796C     8B4424 04            mov eax,dword ptr ss:\[esp+4\]   
77E57970     83F8 F4              cmp eax,-0C   
77E57973     0F84 4CB4FFFF        je kernel32.77E52DC5   
77E57979     83F8 F5              cmp eax,-0B   
77E5797C     0F84 38B4FFFF        je kernel32.77E52DBA   
77E57982     83F8 F6              cmp eax,-0A   
77E57985     0F84 0F500200        je kernel32.77E7C99A   
77E5798B     8BC8                 mov ecx,eax   
77E5798D     81E1 03000010        and ecx,10000003   
77E57993     83F9 03              cmp ecx,3   
77E57996     50                   push eax   
77E57997     0F84 26870000        je kernel32.77E600C3   
77E5799D     FF15 3C10E477        call dword ptr ds:\[<&ntdll.NtClose>\]   
77E579A3     85C0                 test eax,eax   
77E579A5     0F8C 02B4FFFF        jl kernel32.77E52DAD   
77E579AB     33C0                 xor eax,eax   
77E579AD     40                   inc eax   
77E579AE     C2 0400              retn 4 //返回到 004043D5     
  
  
004043D5     58                   pop eax   
004043D6     8B85 D5254000        mov eax,dword ptr ss:\[ebp+4025D5\]   
004043DC     BB 01000000          mov ebx,1   
004043E1     E8 08000000          call XJ01000.004043EE   
004043E6     8D85 14224000        lea eax,dword ptr ss:\[ebp+402214\]   
004043EC     50                   push eax   
004043ED     C3                   retn //返回到 004044B7     
  
  
004044B7     8B9D D5254000        mov ebx,dword ptr ss:\[ebp+4025D5\] ; XJ01000.00400000   
004044BD     039D D9254000        add ebx,dword ptr ss:\[ebp+4025D9\]   
                                  //EBX=00400000 + 00002A11=00402A11  这就是 OEP 值    
004044C3     C1CB 07              ror ebx,7   
  
  
OK，在 00402A11 内存处的几个字节上下 内存访问 断点，F9 运行，断在 OEP 处！   
或者在 00402A11 处下硬件断点；或者直接把 004044C3  ror ebx,7 改为：JMP EBX   
  
  
00402A11     6A 00                push 0  //在这儿用 LordPE 纠正 ImageSize 后完全 DUMP 这个进程    
00402A13     E8 6C020000          call 00402C84   
00402A18     A3 00104000          mov dword ptr ds:\[401000\],eax   
00402A1D     6A 00                push 0   
00402A1F     68 352A4000          push 402A35   
00402A24     6A 00                push 0   
00402A26     6A 64                push 64   
00402A28     50                   push eax   
00402A29     E8 8C020000          call 00402CBA   
  
  
———————————————————————   
  
运行 ImportREC，选择这个进程。把 OEP 改为 00002A11，点 IT AutoSearch，点“Get Import”，用“追踪层次 1”修复全部函数。FixDump，正常运行！  14.9K->15K   试着用脱壳后的仙剑加壳，功能正常 ![icon\_smile.gif](img/59da1937da424a7e6d4210c5b355f670.jpg)  
  
     
—————————————————————————————————   
       
                                   
         ,     \_/    
        /| \_.-~/            \_     ,        青春都一饷   
       ( /~   /              ~-.\_ |   
       \`\\  \_/                   ~ )          忍把浮名    
   \_-~~~-.)  )\_\_/;;,.          \_  //'    
  /'\_,   --~    ~~~-  ,;;\_\_\_(  (.-~~~-.        换了破解轻狂   
 \`~ \_( ,\_..-- (     ,;'' /    ~--   /.\_\`    
  /~~//'   /' \`~         ) /--..\_, )\_  \`~    
  "  \`~"  "      \`"      /~'\`    \`\\~~      
                         "     "   "~'  ""   
  
       
  
      Cracked By 巢水工作坊——fly \[OCN\]\[FCG\]\[NUKE\]   
  
                 2003-12-07  0:36  
 4dd466d0e1e0cc123afbe504da2462ca
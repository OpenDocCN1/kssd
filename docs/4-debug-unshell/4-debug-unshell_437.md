

# VideoSplitter V2.31 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6984.htm`](https://www.pediy.com/kssd/pediy06/pediy6984.htm)

ASProtect 1.23RC4-1.3.08.24 脱壳+破解——VideoSplitter V2.31   
    
    
    
下载页面： http://www.fixdown.com/soft/8414.htm   
软件语言： 英文     
软件类型： 国外软件 / 文件分割    
运行环境： Win9x/NT/2000/XP/    
软件大小： 705KB    
软件等级： \*\*\*    
整理时间： 2003-9-27 0:43:36    
开 发 商： http://www.boilsoft.com/videosplitter    
  
【软件简介】：一个 AVI/MPEG/ASF/WMV 分机的分割，合并工具，也包括了 AVI 和 MPEG 的播放器在里面，你可以很轻松的利用这个播放器把你喜欢的片断截取下来。    
  
【保护方式】：ASProtect 壳+试用期限   
  
【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！   
  
【调试环境】：WinXP、Ollydbg1.09、PEiD、LordPE、AsprDbgr、ImportREC   
  
—————————————————————————————————    
【脱壳过程】：   
             
           
  
一、寻找 OEP   
  
  
设置 Ollydbg 忽略除了“内存访问异常”之外的所有其它异常选项。老规矩：用 IsDebug 1.4 插件去掉 Ollydbg 的调试器标志。载入后弹出“是压缩代码——要继续进行分析吗？”，点“否”。   
  
  
00401000     68 01604B00          push VideoSpl.004B6001//进入 OD 后停在这！   
00401005     E8 01000000          call VideoSpl.0040100B   
0040100A     C3                   retn   
  
F9 运行，程序会在异常处中断。Shift+F9 通过异常，26 次,看见如下代码就要停下了：   
  
00A639EC     3100                 xor dword ptr ds:\[eax\],eax//第 26 次异常    
  
这整段代码几乎是固定的，很好判断。另外：再按一次 Shift+F9 程序就会运行。   
  
00A639EE     64:8F05 00000000     pop dword ptr fs:\[0\]   
00A639F5     58                   pop eax   
00A639F6     833D B07EA600 00     cmp dword ptr ds:\[A67EB0\],0   
00A639FD     74 14                je short 00A63A13   
00A639FF     6A 0C                push 0C   
00A63A01     B9 B07EA600          mov ecx,0A67EB0   
00A63A06     8D45 F8              lea eax,dword ptr ss:\[ebp-8\]   
00A63A09     BA 04000000          mov edx,4   
00A63A0E     E8 2DD1FFFF          call 00A60B40   
00A63A13     FF75 FC              push dword ptr ss:\[ebp-4\]   
00A63A16     FF75 F8              push dword ptr ss:\[ebp-8\]   
00A63A19     8B45 F4              mov eax,dword ptr ss:\[ebp-C\]   
00A63A1C     8338 00              cmp dword ptr ds:\[eax\],0   
00A63A1F     74 02                je short 00A63A23   
00A63A21     FF30                 push dword ptr ds:\[eax\]   
00A63A23     FF75 F0              push dword ptr ss:\[ebp-10\]   
00A63A26     FF75 EC              push dword ptr ss:\[ebp-14\]   
00A63A29     C3                   retn//直接在这下断！Shift+F9 断在这！返回到 00A75FE0   
  
  
00A75FE0     E9 08000000          jmp 00A75FED   
00A75FED     E8 0D000000          call 00A75FFF   
  
TC EIP<700000   使用模拟跟踪吧！几十秒后 Ollydbg 自动暂停在 0044E404   
  
————————————————————————   
查看 运行跟踪 ，很容易看见从壳中返回程序的地方：   
  
00A761FF     F3:AA                rep stos byte ptr es:\[edi\]   
00A76201     9D                   popfd   
00A76202     5F                   pop edi   
00A76203     59                   pop ecx   
00A76204     C3                   retn//这里返回程序 0044E404   
  
00A76205     55                   push ebp   
00A76206     8BEC                 mov ebp,esp   
00A76208     53                   push ebx   
00A76209     56                   push esi   
00A7620A     8B75 0C              mov esi,dword ptr ss:\[ebp+C\]   
00A7620D     8B5D 08              mov ebx,dword ptr ss:\[ebp+8\]   
  
————————————————————————   
  
0044E404     FF15 EC314700        call dword ptr ds:\[4731EC\]//暂停在这！   
  
  
—————————————————————————————————   
二、补上 OEP 处的 Stolen Code   
  
  
  
直接在 Ollydbg 里复制 00A76205－00A7620F 处的的 2 进制代码，粘贴到 0044E3F9-0044E403 处，补上 Stolen Code   
  
  
0044E3F9     55                   push ebp//在这儿用 LordPE 纠正 ImageSize 后完全 DUMP 这个进程   
0044E3FA     8BEC                 mov ebp,esp   
0044E3FC     53                   push ebx   
0044E3FD     56                   push esi   
0044E3FE     8B75 0C              mov esi,dword ptr ss:\[ebp+C\]   
0044E401     8B5D 08              mov ebx,dword ptr ss:\[ebp+8\]//这上面的字节是补上的 Stolen Code   
  
0044E404     FF15 EC314700        call dword ptr ds:\[4731EC\]   
0044E40A     33D2                 xor edx,edx   
0044E40C     8AD4                 mov dl,ah   
0044E40E     8915 28F44900        mov dword ptr ds:\[49F428\],edx   
0044E414     8BC8                 mov ecx,eax   
0044E416     81E1 FF000000        and ecx,0FF   
0044E41C     890D 24F44900        mov dword ptr ds:\[49F424\],ecx   
0044E422     C1E1 08              shl ecx,8   
0044E425     03CA                 add ecx,edx   
0044E427     890D 20F44900        mov dword ptr ds:\[49F420\],ecx   
0044E42D     C1E8 10              shr eax,10   
0044E430     A3 1CF44900          mov dword ptr ds:\[49F41C\],eax   
0044E435     6A 01                push 1   
0044E437     E8 882C0000          call VideoSpl.004510C4   
  
  
—————————————————————————————————   
三、修复输入表   
  
  
试试 AsprDbgr v1.0beta 吧，用 AsprDbgr v1.0beta 载入 VideoSplitter.exe   
  
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆   
  
AsprDbgr v1.0beta (![icon\_razz.gif](img/225e8643bb1a5100a7c1eb0bbb8b6643.jpg)) Made by me... Manko.   
  
  iEP=401000 (E:试炼场脱壳学习 ASProtectVideoSplitterVideoSplitter.exe)   
  
  GST returns to: A52667   
    Trick aspr GST... (EAX=12121212h)   
  GV returns to: A61A61   
    IAT Start: 473000   
          End: 4737A4   
       Length: 7A4   
      IATentry 4731EC = A61C8C resolved as GetVersion   
      IATentry 4731FC = A61CC8 resolved as LockResource   
      IATentry 473244 = A61CD8 resolved as GetCommandLineA   
      IATentry 4732C8 = A61C64 resolved as GetModuleHandleA   
      IATentry 4732F4 = A61CB8 resolved as GetCurrentProcess   
      IATentry 473370 = A617A4 resolved as GetProcAddress   
    0 invalid entries erased.   
  Dip-Table at adress: A67AB4   
    0 0 0 0 0 0 0 0 0 0 40D4B0 40D4A0 40D490 0   
  Last SEH passed. Searching for signatures. Singlestepping to OEP!   
    Call + OEP-jump-setup at: A77458 ( Code: E8000000 5D81ED )   
    Mutated, stolen bytes at: A774A3 ( Code: 613EEB02 CD202EEB )   
    Erase of stolen bytes at: A77407 ( Code: 9CFCBF46 74A700B9 )   
      Repz ... found. Skipping erase of stolen bytes. ![icon\_wink.gif](img/c0318d016c06432889547781c889cd9c.jpg)  
  Dip from pre-OEP: 44E404 (Reached from: A77418)   
  GST returns to: 40BEC4   
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆   
  
运行 ImportREC，选择这个进程。把 OEP 改为 0004E404，点 IT AutoSearch，点“Get Import”，呵呵，函数全部有效。把 OEP 改回 0004E3F9 ，FixDump。   
    
  
—————————————————————————————————   
四、消除暗桩   
  
  
程序依然是无法正常运行的，寻找程序是哪里出错的。   
  
0040DD4D     FF15 8CD64800        call dword ptr ds:\[48D68C\]//进入 00A61C08   
  
00A61C55     5E                   pop esi   
00A61C56     5B                   pop ebx   
00A61C57     5D                   pop ebp   
00A61C58     C2 0800              retn 8//返回 0040DD53   
  
Ctrl+F 在程序空间里查找命令：retn 8   随便找一个吧：   
  
00403B74     C2 0800              retn 8   
  
把 48D68C 处的 00A61C08 改为：00403B74   
0048D68C  08 1C A6 00 //旧的指针 ![icon\_sad.gif](img/6b6604f612cad8b046faadea41e25cf2.jpg)  
0048D68C  74 3B 40 00 //新的指针 ![icon\_biggrin.gif](img/40f8aa1f3e2e5f4b0b2a718e0bd5133a.jpg)  
  
0040DD4D     FF15 8CD64800        call dword ptr ds:\[48D68C\]     
00403B74     C2 0800              retn 8   
  
  
这个暗桩的处理简单点，按照 jingulong 兄的指点，作如上修改就能正常运行了。 ![icon\_biggrin.gif](img/40f8aa1f3e2e5f4b0b2a718e0bd5133a.jpg)  
  
  
—————————————————————————————————   
五、破解    
  
  
0040E8B7     E8 7C540500          call 1-dumped.00463D38   
0040E8BC     8B86 00460000        mov eax,dword ptr ds:\[esi+4600\]   
0040E8C2     85C0                 test eax,eax   
0040E8C4     75 51                jnz short 1-dumped.0040E917   
0040E8C6     A1 98254900          mov eax,dword ptr ds:\[492598\]   
  
把以上代码改为：   
  
0040E8B7     E8 7C540500          call 1-dumped.00463D38   
0040E8BC     C7860046000001000000 mov dword ptr ds:\[esi+4600\],1   
0040E8C6     EB 4F                jmp short 1-dumped.0040E917   
  
  
\[esi+4600\]处是全局注册标志，手动给其赋 1，一切 OK 啦！![icon\_smile.gif](img/59da1937da424a7e6d4210c5b355f670.jpg)  
  
\[HKEY\_LOCAL\_MACHINESOFTWAREBoilSoftVideo SplitterSN\]   
"name"="fly"   
"sn"="1234-2345"   
  
随意填入注册信息，呵呵   
  
  
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
  
               2004-01-04  17:05  
 4dd466d0e1e0cc123afbe504da2462ca
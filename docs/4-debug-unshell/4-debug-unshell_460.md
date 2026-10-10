

# Resource Builder 2.1.0.3 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6668.htm`](https://www.pediy.com/kssd/pediy06/pediy6668.htm)

下载页面：[`www.superdown.com/soft/17405.htm`](http://www.superdown.com/soft/17405.htm)   
软件大小：4250KB  
软件语言：英文  
软件类别：国外软件 / 共享版 / 编程开发  
运行环境：Win9x/WinNT/2000/ME/XP  
加入时间：2004-4-16 20:08:04  
软件评级：\*\*\*  
开 发 商：[`www.sicomponents.com/rbldr.html`](http://www.sicomponents.com/rbldr.html)  
软件介绍：自称是自 Borland Resource WorkShop 4.5 后的长久期待>可以直接打开资源文件，可以导入.exe 文件并可选择指定的资源以便于修改，修改后只能存为资源格式，如果直接打开表但可以类似 exescope 一样修改，如果是导入的话，只能看到 16 进制码。   
              
【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  
              
【调试环境】：WinXP、Ollydbg1.10B、PEiD、LordPE、AsprDbgr、ImportREC 1.42+  
              
—————————————————————————————————   
【脱壳过程】：  
                       
              
                        
如今的猛壳们把 Stolen Code 越做越变态，虽然大部分 Stolen Code 都可以分析出来，但是有点费时间。这次以 Resource Builder 2.1.0.3 为例来演示一种 ASProtect 1.23RC4 壳的 Stolen Code 简便解决方案。这个方法我曾经见过飞速兄用过。在壳把所有的代码解压之后、处理 Stolen Code 之前，把进程 Dump 出来，补上那段壳代码，模仿构造当时的堆栈和寄存器值环境，这样就由壳自己来解决 Stolen Code 的问题啦。推而广之，这个 Stolen Code 简便解决方案也适用于某些其他壳。  
              
—————————————————————————————————  
一、Pre-Dip：用自己的名字注册  
              
              
老规矩：用 IsDebug 1.4 插件去掉 Ollydbg 的调试器标志。  
因为程序的 rbcore.dll 和 sicmplr.dll 也是用了 ASProtect 加壳，所以我们先忽略所有的异常选项，当 Ollydbg 弹出“是压缩代码——要继续进行分析吗？”，点“否”。接着设置 Ollydbg 忽略除了“内存访问异常”之外的所有其它异常选项。  
        

> ```
> 代码:      
> 00401000     68 01A06E00          push Resbldr2.006EA001//进入 OD 后停在这
> 00401005     E8 01000000          call Resbldr2.0040100B
> 0040100A     C3                   retn
> 
> ```
> 
>   

   
              
Shift+F9 运行，注意看堆栈，当第 2 次在堆栈中看见“8wh3JAAQjOI=”硬盘指纹时，可以处理了。  
              

> ```
> 代码:
> 0012FF3C    0012FF44  指针到下一个 SEH 记录
> 0012FF40    012B465C  SE 句柄
> 0012FF44    0012FFE0  指针到下一个 SEH 记录
> 0012FF48    012B4C49  SE 句柄
> 0012FF4C    0012FF90
> 0012FF50    012A0000
> 0012FF54    00D00000
> 0012FF58    012B4138
> 0012FF5C    012D33BC  ASCII "8wh3JAAQjOI="//硬盘指纹
> 
> ```
> 
>   

   
              

> ```
> 代码:
> 012B46A5     3100                 xor dword ptr ds:\[eax\],eax//异常，第 2 次看见硬盘指纹
> 012B46A7     EB 01                jmp short 012B46AA
> 
> ```
> 
>   

  
              
Alt+M 打开内存查看窗口，在 00401000 段下 内存访问断点，Shift+F9 通过异常，断下  
              

> ```
> 代码:
> 00578654     55                   push ebp
> 00578655     8BEC                 mov ebp,esp
> 00578657     8B45 08              mov eax,dword ptr ss:\[ebp+8\]
> 0057865A     85C0                 test eax,eax
> 0057865C     74 0C                je short Resbldr2.0057866A
> 0057865E     8038 00              cmp byte ptr ds:\[eax\],0
> 00578661     74 07                je short Resbldr2.0057866A
> 00578663     C605 90046200 01     mov byte ptr ds:\[620490\],1
> 0057866A     8B15 84046200        mov edx,dword ptr ds:\[620484\]
> 00578670     8915 80046200        mov dword ptr ds:\[620480\],edx
> 00578676     A3 84046200          mov dword ptr ds:\[620484\],eax//注册名
> 0057867B     5D                   pop ebp
> 0057867C     C2 0400              retn 4
> 
> ```
> 
>   

  
              
过了 00578676 之后，\[620484\]处应该是保存的注册用户名，找一段空地，在 613B00 处写入：fly  \[OCN\]\[FCG\]\[NUKE\]\[DCM\]，然后把\[620484\]处的值改为 613B00。OK，这个 Pre-Dip 处理完毕。感谢 lipton 兄的指教。这个程序有几个功能的代码需要真正的 Key 才能解开，所以这只是伪注册罢了。  
              
                          
—————————————————————————————————  
二、Dump 以及区域脱壳  
              
              
取消内存断点，继续 Shift+F9 运行。来到 ASProtect 最后 1 次典型异常处。  
              

> ```
> 代码:
> 012B39EC     3100                 xor dword ptr ds:\[eax\],eax//ASProtect 最后 1 次典型异常处
> 012B39EE     64:8F05 00000000     pop dword ptr fs:\[0\]
> 012B39F5     58                   pop eax
> 012B39F6     833D B07E2B01 00     cmp dword ptr ds:\[12B7EB0\],0
> 012B39FD     74 14                je short 012B3A13
> 012B39FF     6A 0C                push 0C
> 012B3A01     B9 B07E2B01          mov ecx,12B7EB0
> 012B3A06     8D45 F8              lea eax,dword ptr ss:\[ebp\-8\]
> 012B3A09     BA 04000000          mov edx,4
> 012B3A0E     E8 2DD1FFFF          call 012B0B40
> 012B3A13     FF75 FC              push dword ptr ss:\[ebp\-4\]
> 012B3A16     FF75 F8              push dword ptr ss:\[ebp\-8\]
> 012B3A19     8B45 F4              mov eax,dword ptr ss:\[ebp\-C\]
> 012B3A1C     8338 00              cmp dword ptr ds:\[eax\],0
> 012B3A1F     74 02                je short 012B3A23
> 012B3A21     FF30                 push dword ptr ds:\[eax\]
> 012B3A23     FF75 F0              push dword ptr ss:\[ebp\-10\]
> 012B3A26     FF75 EC              push dword ptr ss:\[ebp\-14\]
> 012B3A29     C3                   retn//此处下断，Shift+F9 断下
> 
> ```
> 
>   

  
              
此时 ESP=0012FF5C，看看堆栈，使用 ESP 定律  
              

> ```
> 代码:
> 0012FF5C    012CE9DC
> 0012FF60    00400000  ASCII "MZP"
> 0012FF64    C244FCB6
> 0012FF68    0012FFA4//注意这里 ★
> 
> ```
> 
>   

  
              
0012FF68＝0012FFA4，选中 0012FF68 转存处的 4 个字节，下“硬件访问－>Word”断点。F9 运行，中断下来  
              

> ```
> 代码:
> 012C6A41     EB 44                jmp short 012C6A87//断在这里
> 012C6A43     EB 01                jmp short 012C6A46
> 012C6A46     51                   push ecx
> 012C6A47     57                   push edi
> 012C6A48     9C                   pushfd
> 012C6A49     FC                   cld
> 012C6A4A     BF 00000000          mov edi,0
> 012C6A4F     B9 00000000          mov ecx,0
> 012C6A54     F3:AA                rep stos byte ptr es:\[edi\]//这里下断
> 012C6A56     9D                   popfd
> 012C6A57     5F                   pop edi
> 012C6A58     59                   pop ecx
> 012C6A59     C3                   retn
> 
> ```
> 
>   

  
              

> ```
> 代码:
> 012C6A87     03C3                 add eax,ebx
> 012C6A89     BB 8B070000          mov ebx,78B//注意这个值 ★
> 012C6A8E     0BDB                 or ebx,ebx
> 012C6A90     75 07                jnz short 012C6A99
> 
> ```
> 
>   

  
              

> ```
> 代码:
> 012C6A99     E8 00000000          call 012C6A9E
> 012C6A9E     5D                   pop ebp
> 012C6A9F     81ED 4DE14B00        sub ebp,4BE14D
> 012C6AA5     8D85 F2E04B00        lea eax,dword ptr ss:\[ebp+4BE0F2\]
> 012C6AAB     8D8D 94E14B00        lea ecx,dword ptr ss:\[ebp+4BE194\]
> 012C6AB1     03CB                 add ecx,ebx
> 012C6AB3     8941 01              mov dword ptr ds:\[ecx+1\],eax
> 012C6AB6     8D85 36E14B00        lea eax,dword ptr ss:\[ebp+4BE136\]
> 012C6ABC     8D8D FAE04B00        lea ecx,dword ptr ss:\[ebp+4BE0FA\]
> 012C6AC2     8901                 mov dword ptr ds:\[ecx\],eax
> 012C6AC4     B8 5E140000          mov eax,145E
> 012C6AC9     8D8D FFE04B00        lea ecx,dword ptr ss:\[ebp+4BE0FF\]
> 012C6ACF     8901                 mov dword ptr ds:\[ecx\],eax
> 012C6AD1     8D8D 94E14B00        lea ecx,dword ptr ss:\[ebp+4BE194\]
> 012C6AD7     8D85 94F34B00        lea eax,dword ptr ss:\[ebp+4BF394\]
> 012C6ADD     51                   push ecx
> 012C6ADE     50                   push eax
> 012C6ADF     E8 76FFFFFF          call 012C6A5A
> 012C6AE4     61                   popad//下面开始处理 Stolen Code ★
> 012C6AE5     EB 02                jmp short 012C6AE9
> 
> ```
> 
>   

  
              
              
OK，现在我们不去分析这个东东的 Stolen Code，程序代码已经解开，可以用 LordPE 纠正 ImageSize 后完全 DUMP 这个进程了！  
接着 区域脱壳：012C0000，大小＝00008000，也就是脱出上面的部分壳处理代码段。  
              
下面走至伪 OEP:在 012C6A54 处下断，F9 运行，断下。  
              

> ```
> 代码:
> 012C6A54     F3:AA                rep stos byte ptr es:\[edi\]//断下
> 012C6A56     9D                   popfd
> 012C6A57     5F                   pop edi
> 012C6A58     59                   pop ecx
> 012C6A59     C3                   retn//飞向光明之巅！返回程序 004072DC
> 
> ```
> 
>   

  
              

> ```
> 代码:            
> 004072DC     FF25 1CA36200        jmp dword ptr ds:\[62A31C\]//第 1 个函数啦
> 
> ```
> 
>   

  
              
F7 走下去，来到伪 OEP 处。当然这个东东的 Stolen Code 也不难分析出来，这里只是作为一个例子来演示罢了。  
              

> ```
> 代码:
> 00613652     61                   popad
> 00613653     0000                 add byte ptr ds:\[eax\],al
> 00613655     0000                 add byte ptr ds:\[eax\],al
> 00613657     0000                 add byte ptr ds:\[eax\],al
> 00613659     0000                 add byte ptr ds:\[eax\],al
> 0061365B     0000                 add byte ptr ds:\[eax\],al
> 0061365D     0000                 add byte ptr ds:\[eax\],al
> 0061365F     E8 3C3DDFFF          call Resbldr2.004073A0//伪 OEP ★
> 00613664     33C0                 xor eax,eax
> 00613666     55                   push ebp
> 00613667     68 B6366100          push Resbldr2.006136B6
> 0061366C     64:FF30              push dword ptr fs:\[eax\]
> 0061366F     64:8920              mov dword ptr fs:\[eax\],esp
> 00613672     A1 EC4E6200          mov eax,dword ptr ds:\[624EEC\]
> 00613677     8B00                 mov eax,dword ptr ds:\[eax\]
> 00613679     E8 86C2E8FF          call Resbldr2.0049F904
> 0061367E     A1 EC4E6200          mov eax,dword ptr ds:\[624EEC\]
> 00613683     8B00                 mov eax,dword ptr ds:\[eax\]
> 00613685     BA CC366100          mov edx,Resbldr2.006136CC; ASCII "Resource Builder 2.0"
> 0061368A     E8 6DBEE8FF          call Resbldr2.0049F4FC
> 
> ```
> 
>   

  
              
              
现在我们来“组装”一下 dumped.exe。呵呵，用 LordPE 打开 dumped.exe，从磁盘载入刚才区域脱壳的 Region012C0000-012C8000.dmp 区段，修改其 Voffset=00EC0000，只保留 LordPE 的“验证 PE”选项，重建 PE。Dump 完成！  
              
              
—————————————————————————————————  
三、用 AsprDbgr 搞定输入表  
              
      

> ```
> 代码:         
> AsprDbgr v1.0beta (:P) Made by me... Manko.
> 
>   iEP=401000 (E:\\试炼场\\脱壳学习\\ASProtect\\Resource Builder 2.0\\Resbldr2.exe
> 
>   GST returns to: 12A2667
>     Trick aspr GST... (EAX\=12121212h)
>   GV returns to: 12B1A61
>     IAT Start: 62A21C
>           End: 62ABAC
>        Length: 990
>       IATentry 62A23C = 12B1C8C resolved as GetVersion
>       IATentry 62A270 = 12B17A4 resolved as GetProcAddress
>       IATentry 62A274 = 12B1C64 resolved as GetModuleHandleA
>       IATentry 62A288 = 12B1CD8 resolved as GetCommandLineA
>       IATentry 62A31C = 12B1C64 resolved as GetModuleHandleA
>       IATentry 62A3CC = 12B1CC8 resolved as LockResource
>       IATentry 62A410 = 12B1C8C resolved as GetVersion
>       IATentry 62A434 = 12B17A4 resolved as GetProcAddress
>       IATentry 62A43C = 12B1C64 resolved as GetModuleHandleA
>       IATentry 62A474 = 12B1CC0 resolved as GetCurrentProcessId
>       IATentry 62A480 = 12B1CF0 resolved as FreeResource
>     SymbolInitialize seems to have frozen.
>     Any invalid IAT entries was NOT erased...
>   Dip-Table at adress: 12B7AB4
>     0 578654 0 578694 5786C4 578728 0 6122C0 612CF8 5785DC 578610 578644 578
> 0
>   Last SEH passed. Searching for signatures. Singlestepping to OEP!
>     Call + OEP-jump-setup at: 12C6A99 ( Code: E8000000 5D81ED )
>     Mutated, stolen bytes at: 12C6AE4 ( Code: 61EB02CD 20EB01F0 )
>     Erase of stolen bytes at: 12C6A48 ( Code: 9CFCBF87 6A2C01B9 )
>       Repz ... found. Skipping erase of stolen bytes. ;)
>   Dip from pre-OEP: 4072DC (Reached from: 12C6A59)
>   Sugested tempOEP at: 61365F
>   GST returns to: 76171AC0
> 
> ```
> 
>   

  
              
              
运行 ImportREC 1.42+，选择这个进程。填入：RVA=0022A21C，Size=00000990，点 IT AutoSearch，点“Get Import”，剪切掉几个无效的函数，把 OEP 改为 00EC0000，FixDump！  
              
              
—————————————————————————————————  
四、以壳解壳：Stolen Code 简便解决方案  
              
              
现在的 OEP 是 012C0000，修改其入口代码为：  
              

> ```
> 代码:
> 012C0000     BB 8B070000          mov ebx,78B
> 012C0005     E9 8F6A0000          jmp dumped\_.012C6A99
> 
> ```
> 
>   

  
              
这样就利用了原壳的代码来处理 Stolen Code 了。保存之后就可以运行了。  
               
              
—————————————————————————————————  
五、2 个暗桩的修复  
              
              
1、另外当点击“Option”时会自动退出，跟踪程序找到异常出错的地方  
              

> ```
> 代码:
> 0057AF65     E8 A2D9FFFF          call dumped\_.0057890C//把这里 NOP 掉就行了
> 0057AF6A     33C0                 xor eax,eax
> 
> ```
> 
>   

  
  

> ```
> 代码:
> 0057890C     55                   push ebp
> 0057890D     8BEC                 mov ebp,esp
> 0057890F     51                   push ecx
> 00578910     53                   push ebx
> 00578911     8B05 1E784000        mov eax,dword ptr ds:\[40781E\]
> 00578917     8B18                 mov ebx,dword ptr ds:\[eax\]
> 00578919     FF33                 push dword ptr ds:\[ebx\]
> 0057891B     895D FC              mov dword ptr ss:\[ebp\-4\],ebx
> 0057891E     8F03                 pop dword ptr ds:\[ebx\]//异常
> 00578920     8B45 FC              mov eax,dword ptr ss:\[ebp\-4\]
> 00578923     5B                   pop ebx
> 00578924     59                   pop ecx
> 00578925     5D                   pop ebp
> 00578926     C3                   retn
> 
> ```
> 
>   

  
              
————————————————————————  
2、点击“Link To EXE”时会自动退出  
              
              

> ```
> 代码:
> 005788C4     55                   push ebp
> 005788C5     8BEC                 mov ebp,esp
> 005788C7     51                   push ecx
> 005788C8     C645 FF 00           mov byte ptr ss:\[ebp\-1\],0
> 005788CC     A1 48486200          mov eax,dword ptr ds:\[624848\]
> 005788D1     8038 00              cmp byte ptr ds:\[eax\],0
> 005788D4     75 19                jnz short 005788EF
> 005788D6     56                   push esi
> 005788D7     8B35 68666200        mov esi,dword ptr ds:\[626668\]
> 005788DD     8B46 3C              mov eax,dword ptr ds:\[esi+3C\]
> 005788E0     8B4430 28            mov eax,dword ptr ds:\[eax+esi+28\];00ECC000//取入口地址
> 005788E4     66:3D 0010           cmp ax,1000//比较后面是否是 1000
> 005788E8     74 04                je short UnPacked.005788EE//改为 JMP
> 005788EA     804D FF 01           or byte ptr ss:\[ebp\-1\],1
> 005788EE     5E                   pop esi
> 005788EF     807D FF 00           cmp byte ptr ss:\[ebp\-1\],0
> 005788F3     74 13                je short 00578908
> 
> ```
> 
>   

  
              
原来 005788E4 这里还检测了原程序的入口地址，改为 JMP 就行了。  
“Edit as Text”和“Link To EXE”的功能代码是需要真正的 Key 才能解压的，没办法啦。  
              
              
—————————————————————————————————      

> ```
> 代码:                                
>          ,     \_/ 
>         /| \_.-~/            \\\_     ,        青春都一晌
>        ( /~   /              \\~-.\_ |\\
>        \`\\\\  \_/                \\   ~\\ )          忍把浮名 
>    \_-~~~-.)  )\_\_/;;,.          \\\_  //'
>   /'\_,\\   --~   \\ ~~~-  ,;;\\\_\_\_(  (.-~~~-.        换了破解轻狂
>  \`~ \_( ,\_..--\\ (     ,;'' /    ~--   /.\_\`\\ 
>   /~~//'   /' \`~\\         ) /--..\_, )\_  \`~
>   "  \`~"  "      \`"      /~'\`\\    \`\\\\~~\\   
>                          "     "   "~'  ""
> 
> ```
> 
>   

  
      
  
     Cracked By 巢水工作坊——fly \[OCN\]\[FCG\]\[NUKE\]\[DCM\]  
  
                2004-05-13  17:00  
 4dd466d0e1e0cc123afbe504da2462ca
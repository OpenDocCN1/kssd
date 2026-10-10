

# Registry Medic2.98

> 原文：[`www.pediy.com/kssd/pediy06/pediy6417.htm`](https://www.pediy.com/kssd/pediy06/pediy6417.htm)

马甲透视 SVKP 1.3x 之 Registry Medic   
【目    标】: Registry Medic 2.98   
【工    具】:OllyDbg 1.1b(DIY 版)   
【任    务】:不脱马甲,巧妙注册   
【操作平台】:WINDOWS 2000 ADV sp2   
【作    者】:loveboom\[DFCG\]   
【相关链接】:www.skycn.com(好像是这里下的)   
【简要说明】：看名字就知道是一款和注册表有关的东西，具体有什么功能我也没注意，因为破它只是无意中玩的，我自己并不用这款软件的，所以好不好我也不知道。第一次碰 SVKP1.3x 加的商业软件，也巧趁此机会看看 SVKP1.3x SDK 加壳后的破解难度(因为注册过程跑到壳里去了，所以我想应该是 SDK 加的吧)。   
【详细过程】:   
相关信息:   
Name:bmd2chen@tom.com   
SN:01234567890123456789xxxx(共 33 位)   
  
安装后，先看看情况，发现没注册成功有提示，看到这里我高兴了一会儿，但很快就让我苦了，看完大概信息后，像上篇一样，看看它的个头，应该又是 delphi 的程序，用 peid 扫描一下，报告说是 svkp1.3x 加的壳，我本来再用上篇的方法，打开 DEDE 选转存进程，打开进程窗口了，我还以为我的眼有问题呢，找呀找，怎么就是看不到 Registry Medic 的进程呢(这里请教高手，在 2k 下面用什么方法隐藏进程的?不解 ing.)这样我的 A 计划就落空了。看来 SVKP 的壳果然有“水平”，但我们就此停手吗?当然不是了，要不我下面怎么凑到一篇文章呀。用 peid 的扫描进程功能，看看能不能看到 RM(Register Medic)的进程同时想证实它是用 DELPHI 编写的，结果当然是没有了。看到这里我苦笑两声，不会吧，这个壳这么利害。第二次有打退堂豉的心里了，好险呀！怎么办公室里“下雨”了(“经 FBI 证实是“吓”出汉”).这样 B 计划也行不通，还好我手里还有好“武器”，用 OD 的 Attach 看看终于看到目标了，也许这里有朋友会说，直接 attach 不说行了。我不推荐的说，因为有时 attach 后就会发现怎么目标不能动了.这样的话，你想输入注册信息也不给。所以我换另外一种方法，用 OD 加载，并让 RM 运行，运行后看看 cpu 窗口里，怎么这么黑(我设置背影为黑色)，什么代码也没有呀，怎么办呢，只要能调试就可能有办法嘛(引用 tDasm 大侠的名言)。现在按 ALT+V+T 也就是打开线程窗口:   
Threads   
Ident      Entry      Data block   Last error          Status      Priority   User time     System time   
0000062C   00559000   7FFDE000     ERROR\_NOT\_ENOUGH\_M  Active       32 + 0       1.3593 s      0.2656 s   
00000630   77E67532   7FFDC000     ERROR\_SUCCESS (000  Active       32 + 0       0.0000 s      0.0000 s   
0000072C   77E67532   7FFDD000     ERROR\_SUCCESS (000  Active       32 + 0       0.0000 s      0.0000 s   
00000744   77E67532   7FFDB000     ERROR\_SUCCESS (000  Active       32 + 0       0.0000 s      0.0000 s   
  
在第一栏上，也就是那个红色的(因为每次打开的值肯定是不同的，所以你看到值和我贴出的不同是正常现象)，双击，这样来到了系统 Dll 中:   
  
77F840AF    C2 2800         RETN 28    //来到这里    
77F840B2 >  B8 2B000000     MOV EAX,2B   
77F840B7    8D5424 04       LEA EDX,DWORD PTR SS:\[ESP+4\]   
  
现在右键-->view->点 module ‘RegMedic’这样就到了程序的“地盘”,现在 AC 分析一下代码先，分析完毕，流程清楚很多了。再次右键查找所有字符串参考(怎么弄,晕倒#!@%\*&),打开参考后我们就找注册失败的字符串，这样找到这里:   
  
004E2301  |.  BA 14254E00   MOV EDX,RegMedic.004E2514                //  ASCII "Registration failed. Please try again!"    
  
现在向上找到入口点(不是程序的入口点哦):   
  
004E2294  /$  55            PUSH EBP   //这里就是起源    
004E2295  |.  8BEC          MOV EBP,ESP   
004E2297  |.  B9 05000000   MOV ECX,5   
004E229C  |>  6A 00         /PUSH 0      //一般 Delphi 的程序看到这样的东西就可以知道多数我们按 button 的时候它会这里的    
004E229E  |.  6A 00         |PUSH 0   
004E22A0  |.  49            |DEC ECX   
004E22A1  |.^ 75 F9         JNZ SHORT RegMedic.004E229C   
004E22A3  |.  51            PUSH ECX   
  
看到了起源就在起源处下断.   
现在进入 RM 并进行注册，按注册后程序就会停下了:   
  
004E22A4  |.  53            PUSH EBX   
004E22A5  |.  8955 F8       MOV DWORD PTR SS:\[EBP\-8\],EDX             //  ebp-8=假码 012345678901234567890123456789012    
004E22A8  |.  8945 FC       MOV DWORD PTR SS:\[EBP\-4\],EAX             //  ebp-4='bmd2chen@tom.com'    
004E22AB  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP\-4\]             //  eax=bmd2chen@tom.com    
004E22AE  |.  E8 291EF2FF   CALL RegMedic.004040DC   
004E22B3  |.  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP\-8\]             //  假码入 eax    
004E22B6  |.  E8 211EF2FF   CALL RegMedic.004040DC   
004E22BB  |.  33C0          XOR EAX,EAX   
004E22BD  |.  55            PUSH EBP   
004E22BE  |.  68 FF244E00   PUSH RegMedic.004E24FF   
004E22C3  |.  64:FF30       PUSH DWORD PTR FS:\[EAX\]   
004E22C6  |.  64:8920       MOV DWORD PTR FS:\[EAX\],ESP   
004E22C9  |.  E8 06AAFAFF   CALL RegMedic.0048CCD4   
004E22CE  |.  8D55 F4       LEA EDX,DWORD PTR SS:\[EBP\-C\]   
004E22D1  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP\-4\]             //  eax='bmd2chen@tom.com'    
004E22D4  |.  E8 9F66F2FF   CALL RegMedic.00408978   
004E22D9  |.  8D55 F0       LEA EDX,DWORD PTR SS:\[EBP\-10\]   
004E22DC  |.  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP\-8\]             //  eax=假码    
004E22DF  |.  E8 9466F2FF   CALL RegMedic.00408978   
004E22E4  |.  8B45 F0       MOV EAX,DWORD PTR SS:\[EBP\-10\]   
004E22E7  |.  E8 3C1CF2FF   CALL RegMedic.00403F28                   //  获取假码长度    
004E22EC  |.  83F8 21       CMP EAX,21                               //  如果够 21(DEC 33)位就跳    
004E22EF  |.  74 52         JE SHORT RegMedic.004E2343               //  这是第一条路,注册码为 21(DEC 33)位的情况,我第一次也就跟它    
004E22F1  |.  8B45 F0       MOV EAX,DWORD PTR SS:\[EBP\-10\]            //  eax=假码,这里是第二次路    
004E22F4  |.  E8 2F1CF2FF   CALL RegMedic.00403F28                   //  这里取假码长度    
004E22F9  |.  83F8 27       CMP EAX,27   
004E22FC  |.  74 45         JE SHORT RegMedic.004E2343               //  如果长度为 27 的话就跳,也就是说注册长度只能为 22 或 27 要不就 over    
004E22FE  |.  8D45 EC       LEA EAX,DWORD PTR SS:\[EBP\-14\]   
004E2301  |.  BA 14254E00   MOV EDX,RegMedic.004E2514                //  ASCII "Registration failed. Please try again!"    
004E2306  |.  E8 351AF2FF   CALL RegMedic.00403D40   
004E230B  |.  6A 30         PUSH 30   
004E230D  |.  8D55 E8       LEA EDX,DWORD PTR SS:\[EBP\-18\]   
004E2310  |.  A1 E0C94F00   MOV EAX,DWORD PTR DS:\[4FC9E0\]   
004E2315  |.  8B00          MOV EAX,DWORD PTR DS:\[EAX\]   
004E2317  |.  E8 94C8F6FF   CALL RegMedic.0044EBB0   
004E231C  |.  8B45 E8       MOV EAX,DWORD PTR SS:\[EBP\-18\]   
004E231F  |.  E8 C81DF2FF   CALL RegMedic.004040EC   
004E2324  |.  50            PUSH EAX   
004E2325  |.  8B45 EC       MOV EAX,DWORD PTR SS:\[EBP\-14\]   
004E2328  |.  E8 BF1DF2FF   CALL RegMedic.004040EC   
004E232D  |.  50            PUSH EAX   
004E232E  |.  A1 E0C94F00   MOV EAX,DWORD PTR DS:\[4FC9E0\]   
004E2333  |.  8B00          MOV EAX,DWORD PTR DS:\[EAX\]   
004E2335  |.  8B40 24       MOV EAX,DWORD PTR DS:\[EAX+24\]   
004E2338  |.  50            PUSH EAX   
004E2339  |.  E8 7250F2FF   CALL RegMedic.004073B0   
004E233E  |.  E9 A1010000   JMP RegMedic.004E24E4   
004E2343  |>  8B45 F0       MOV EAX,DWORD PTR SS:\[EBP\-10\]            //  假码入 eax    
004E2346  |.  E8 DD1BF2FF   CALL RegMedic.00403F28                   //  获取假码长度    
004E234B  |.  83F8 27       CMP EAX,27   
004E234E  |.  75 64         JNZ SHORT RegMedic.004E23B4              //  如果不等于 27 位就跳    
004E2350  |.  BB 01000000   MOV EBX,1   
004E2355  |.  8D45 EC       LEA EAX,DWORD PTR SS:\[EBP\-14\]   
004E2358  |.  E8 4B19F2FF   CALL RegMedic.00403CA8   
004E235D  |.  83FB 23       CMP EBX,23   
004E2360  |.  7D 26         JGE SHORT RegMedic.004E2388   
004E2362  |>  8D45 E4       /LEA EAX,DWORD PTR SS:\[EBP\-1C\]   
004E2365  |.  50            |PUSH EAX   
004E2366  |.  B9 05000000   |MOV ECX,5   
004E236B  |.  8BD3          |MOV EDX,EBX   
004E236D  |.  8B45 F0       |MOV EAX,DWORD PTR SS:\[EBP\-10\]   
004E2370  |.  E8 BB1DF2FF   |CALL RegMedic.00404130   
004E2375  |.  8B55 E4       |MOV EDX,DWORD PTR SS:\[EBP\-1C\]           //  取假码的前 5 位    
004E2378  |.  8D45 EC       |LEA EAX,DWORD PTR SS:\[EBP\-14\]   
……   
继续到这里:   
  
004E23AC  |.  8B55 EC       MOV EDX,DWORD PTR SS:\[EBP\-14\]   
004E23AF  |.  E8 8C19F2FF   CALL RegMedic.00403D40   
004E23B4  |>  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP\-C\]             //  用户名入 EAX,EAX='lovebom@tom.com'    
004E23B7  |.  E8 6C1BF2FF   CALL RegMedic.00403F28                   //  获取用户名长度    
004E23BC  |.  83F8 06       CMP EAX,6                                //  判断用户名长度是否小于 6    
004E23BF  |.  0F8C F0000000 JL RegMedic.004E24B5                     //  是就跳下去，下面会告诉你，你的用户名长度必须大于 6 位    
004E23C5  |.  8B45 F0       MOV EAX,DWORD PTR SS:\[EBP\-10\]            //  假码入 eax    
004E23C8  |.  E8 5B1BF2FF   CALL RegMedic.00403F28                   //  再次取假码长度    
004E23CD  |.  83F8 21       CMP EAX,21                               //  判断假码长度是否小于 21    
004E23D0  |.  7C 1D         JL SHORT RegMedic.004E23EF   
004E23D2  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP\-C\]             //  如果不小于 21 的话就不跳    
004E23D5  |.  E8 CE0A0000   CALL RegMedic.004E2EA8      //这里可以跟进看看，不过不是很要紧    
  
进入后，我大概看了一下，没什么好东西:   
  
004E2EAB  |.  83C4 F4       ADD ESP,-0C   
004E2EAE  |.  8945 FC       MOV DWORD PTR SS:\[EBP\-4\],EAX             //  ebp-4='bmd2chen@tom.com'    
004E2EB1  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP\-4\]   
004E2EB4  |.  E8 2312F2FF   CALL RegMedic.004040DC   
004E2EB9  |.  33C0          XOR EAX,EAX   
004E2EBB  |.  55            PUSH EBP   
004E2EBC  |.  68 482F4E00   PUSH RegMedic.004E2F48   
004E2EC1  |.  64:FF30       PUSH DWORD PTR FS:\[EAX\]   
004E2EC4  |.  64:8920       MOV DWORD PTR FS:\[EAX\],ESP   
004E2EC7  |.  C645 FB 00    MOV BYTE PTR SS:\[EBP\-5\],0   
004E2ECB  |.  B2 01         MOV DL,1   
004E2ECD  |.  A1 30FC4000   MOV EAX,DWORD PTR DS:\[40FC30\]   
004E2ED2  |.  E8 7900F2FF   CALL RegMedic.00402F50   
004E2ED7  |.  8945 F4       MOV DWORD PTR SS:\[EBP\-C\],EAX   
004E2EDA  |.  33C0          XOR EAX,EAX   
004E2EDC  |.  55            PUSH EBP   
004E2EDD  |.  68 2B2F4E00   PUSH RegMedic.004E2F2B   
004E2EE2  |.  64:FF30       PUSH DWORD PTR FS:\[EAX\]   
004E2EE5  |.  64:8920       MOV DWORD PTR FS:\[EAX\],ESP   
004E2EE8  |.  BA 602F4E00   MOV EDX,RegMedic.004E2F60                //  ASCII "Cindy J. Leonheardt"    
004E2EED  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP\-C\]   
004E2EF0  |.  8B08          MOV ECX,DWORD PTR DS:\[EAX\]   
004E2EF2  |.  FF51 34       CALL DWORD PTR DS:\[ECX+34\]   
004E2EF5  |.  BA 7C2F4E00   MOV EDX,RegMedic.004E2F7C                //  ASCII "\[DJ\]neo"    
004E2EFA  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP\-C\]   
004E2EFD  |.  8B08          MOV ECX,DWORD PTR DS:\[EAX\]   
004E2EFF  |.  FF51 34       CALL DWORD PTR DS:\[ECX+34\]   
004E2F02  |.  8B55 FC       MOV EDX,DWORD PTR SS:\[EBP\-4\]             //  用户名再入 edx 中    
004E2F05  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP\-C\]   
004E2F08  |.  8B08          MOV ECX,DWORD PTR DS:\[EAX\]   
……   
  
看完后回到这里继续:   
  
004E23DA  |.  84C0          TEST AL,AL                               //  这里就是比较了    
004E23DC  |.  75 11         JNZ SHORT RegMedic.004E23EF              //  这里不能跳一跳就 over 了    
004E23DE  |.  8B55 F4       MOV EDX,DWORD PTR SS:\[EBP\-C\]             //  没跳的话就来这里继续,用户名入 EDX    
004E23E1  |.  B8 44254E00   MOV EAX,RegMedic.004E2544   
☆004E23E6  |.  E8 291EF2FF   CALL RegMedic.00404214                   //  这个 CALL 用于判断用户名里有没有字符"@"，没有就 over，    
跟进看看   
  
00404214  /$  85C0          TEST EAX,EAX   
00404216  |.  74 40         JE SHORT RegMedic.00404258   
00404218  |.  85D2          TEST EDX,EDX                             //  比较有没有输入用户名，没有就跳去 over 处    
0040421A  |.  74 31         JE SHORT RegMedic.0040424D   
0040421C  |.  53            PUSH EBX   
0040421D  |.  56            PUSH ESI   
0040421E  |.  57            PUSH EDI   
0040421F  |.  89C6          MOV ESI,EAX   
00404221  |.  89D7          MOV EDI,EDX                              //  用户名入 edi    
00404223  |.  8B4F FC       MOV ECX,DWORD PTR DS:\[EDI\-4\]             //  用户名长度入 ecx    
00404226  |.  57            PUSH EDI                                 //  用户名入栈    
00404227  |.  8B56 FC       MOV EDX,DWORD PTR DS:\[ESI\-4\]   
0040422A  |.  4A            DEC EDX   
0040422B  |.  78 1B         JS SHORT RegMedic.00404248   
0040422D  |.  8A06          MOV AL,BYTE PTR DS:\[ESI\]   
0040422F  |.  46            INC ESI   
00404230  |.  29D1          SUB ECX,EDX   
00404232  |.  7E 14         JLE SHORT RegMedic.00404248   
00404234  |>  F2:AE         /REPNE SCAS BYTE PTR ES:\[EDI\]            //  判断用户名里有没有@字符没有就完了    
00404236  |.  75 10         |JNZ SHORT RegMedic.00404248             //  这里跳就完了    
00404238  |.  89CB          |MOV EBX,ECX   
0040423A  |.  56            |PUSH ESI   
0040423B  |.  57            |PUSH EDI                                //  push @字后面的字符串    
0040423C  |.  89D1          |MOV ECX,EDX   
0040423E  |.  F3:A6         |REPE CMPS BYTE PTR ES:\[EDI\],BYTE PTR DS\>   
00404240  |.  5F            |POP EDI   
00404241  |.  5E            |POP ESI   
00404242  |.  74 0C         |JE SHORT RegMedic.00404250   
  
看完后再次回来:   
  
004E23EB  |.  85C0          TEST EAX,EAX   
004E23ED  |.  75 45         JNZ SHORT RegMedic.004E2434              //  这里一定要跳不跳则 over    
004E23EF  |>  8D45 EC       LEA EAX,DWORD PTR SS:\[EBP\-14\]            //  跳到这里就 over 了    
004E23F2  |.  BA 14254E00   MOV EDX,RegMedic.004E2514                //  ASCII "Registration failed. Please try again!"    
004E23F7  |.  E8 4419F2FF   CALL RegMedic.00403D40   
004E23FC  |.  6A 30         PUSH 30   
  
上面跳成功后来到这里:   
004E2434  |> 8B55 F0       MOV EDX,DWORD PTR SS:\[EBP\-10\]            //  注册码入 edx    
004E2437  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP\-C\]             //  用户名入 eax    
004E243A  |.  E8 5D9FFAFF   CALL RegMedic.0048C39C          //这里可以进去看看：    
0048C39C  /$  55            PUSH EBP   
0048C39D  |.  8BEC          MOV EBP,ESP   
0048C39F  |.  B9 06000000   MOV ECX,6   
0048C3A4  |>  6A 00         /PUSH 0   
0048C3A6  |.  6A 00         |PUSH 0   
0048C3A8  |.  49            |DEC ECX   
0048C3A9  |.^ 75 F9         JNZ SHORT RegMedic.0048C3A4   
0048C3AB  |.  8955 F8       MOV DWORD PTR SS:\[EBP\-8\],EDX             //  EBP-8=假码    
0048C3AE  |.  8945 FC       MOV DWORD PTR SS:\[EBP\-4\],EAX             //  \[EBP-4\]=用户名    
0048C3B1  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP\-4\]   
0048C3B4  |.  E8 237DF7FF   CALL RegMedic.004040DC   
0048C3B9  |.  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP\-8\]   
0048C3BC  |.  E8 1B7DF7FF   CALL RegMedic.004040DC   
0048C3C1  |.  33C0          XOR EAX,EAX   
0048C3C3  |.  55            PUSH EBP   
0048C3C4  |.  68 90C54800   PUSH RegMedic.0048C590   
0048C3C9  |.  64:FF30       PUSH DWORD PTR FS:\[EAX\]   
0048C3CC  |.  64:8920       MOV DWORD PTR FS:\[EAX\],ESP   
0048C3CF  |.  B2 01         MOV DL,1   
0048C3D1  |.  A1 58324700   MOV EAX,DWORD PTR DS:\[473258\]   
0048C3D6  |.  E8 7D6FFEFF   CALL RegMedic.00473358   
0048C3DB  |.  8945 F4       MOV DWORD PTR SS:\[EBP\-C\],EAX   
0048C3DE  |.  33C0          XOR EAX,EAX   
0048C3E0  |.  55            PUSH EBP   
0048C3E1  |.  68 59C54800   PUSH RegMedic.0048C559   
0048C3E6  |.  64:FF30       PUSH DWORD PTR FS:\[EAX\]   
0048C3E9  |.  64:8920       MOV DWORD PTR FS:\[EAX\],ESP   
0048C3EC  |.  BA 02000080   MOV EDX,80000002   
0048C3F1  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP\-C\]   
0048C3F4  |.  E8 FF6FFEFF   CALL RegMedic.004733F8   
0048C3F9  |.  33C9          XOR ECX,ECX   
0048C3FB  |.  BA A4C54800   MOV EDX,RegMedic.0048C5A4                //  ASCII "SoftWareClassesCLSID{B087CE89-A8EE-4890-91E7-696B3F749500}"    
0048C400  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP\-C\]   
0048C403  |.  E8 5870FEFF   CALL RegMedic.00473460   
0048C408  |.  84C0          TEST AL,AL   
0048C40A  |.  74 68         JE SHORT RegMedic.0048C474   
0048C40C  |.  BA ECC54800   MOV EDX,RegMedic.0048C5EC                //  ASCII "AppIdK"    
0048C411  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP\-C\]   
0048C414  |.  E8 2B78FEFF   CALL RegMedic.00473C44   
0048C419  |.  84C0          TEST AL,AL   
0048C41B  |.  74 57         JE SHORT RegMedic.0048C474   
0048C41D  |.  8D4D EC       LEA ECX,DWORD PTR SS:\[EBP\-14\]   
0048C420  |.  BA ECC54800   MOV EDX,RegMedic.0048C5EC                //  ASCII "AppIdK"    
0048C425  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP\-C\]   
0048C428  |.  E8 AF75FEFF   CALL RegMedic.004739DC   
0048C42D  |.  8D4D E8       LEA ECX,DWORD PTR SS:\[EBP\-18\]   
0048C430  |.  BA 01000000   MOV EDX,1   
0048C435  |.  8B45 EC       MOV EAX,DWORD PTR SS:\[EBP\-14\]            //  把我们输入的假码和注册表里的值连接起来.    
0048C438  |.  E8 6FFDFFFF   CALL RegMedic.0048C1AC   
0048C43D  |.  FF75 E8       PUSH DWORD PTR SS:\[EBP\-18\]               //  push 第一部分值'071F3A81002C5C7CFE34'    
0048C440  |.  68 FCC54800   PUSH RegMedic.0048C5FC   
0048C445  |.  8D4D E4       LEA ECX,DWORD PTR SS:\[EBP\-1C\]   
0048C448  |.  BA 02000000   MOV EDX,2                                //  这里开始取第二部分了    
0048C44D  |.  8B45 EC       MOV EAX,DWORD PTR SS:\[EBP\-14\]   
0048C450  |.  E8 57FDFFFF   CALL RegMedic.0048C1AC                   //  这个 Call 就是取值过程    
0048C455  |.  FF75 E4       PUSH DWORD PTR SS:\[EBP\-1C\]               //  取出的第二部分入栈    
0048C458  |.  68 FCC54800   PUSH RegMedic.0048C5FC   
0048C45D  |.  8D45 F0       LEA EAX,DWORD PTR SS:\[EBP\-10\]   
……   
0048C4ED  |.  E8 D66EFEFF   CALL RegMedic.004733C8   
0048C4F2  |.  FF75 F0       PUSH DWORD PTR SS:\[EBP\-10\]               //  push 第一部分和第二部分连接后的值    
0048C4F5  |.  8D55 D0       LEA EDX,DWORD PTR SS:\[EBP\-30\]   
0048C4F8  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP\-4\]             //  EAX=用户名 bmd2chen@tom.com    
0048C4FB  |.  E8 0C080000   CALL RegMedic.0048CD0C   
0048C500  |.  FF75 D0       PUSH DWORD PTR SS:\[EBP\-30\]               //  push 值:"45205DEE712FB90D5AB957BC01A7C3AC"也就是第三部分    
0048C503  |.  68 FCC54800   PUSH RegMedic.0048C5FC   
0048C508  |.  FF75 F8       PUSH DWORD PTR SS:\[EBP\-8\]                //  push 假码    
0048C50B  |.  8D45 F0       LEA EAX,DWORD PTR SS:\[EBP\-10\]   
0048C50E  |.  BA 04000000   MOV EDX,4   
0048C513  |.  E8 D07AF7FF   CALL RegMedic.00403FE8   
0048C518  |.  B1 01         MOV CL,1   
0048C51A  |.  BA A4C54800   MOV EDX,RegMedic.0048C5A4                //  ASCII "SoftWareClassesCLSID{B087CE89-A8EE-4890-91E7-696B3F749500}"    
0048C51F  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP\-C\]   
0048C522  |.  E8 396FFEFF   CALL RegMedic.00473460   
0048C527  |.  84C0          TEST AL,AL   
0048C529  |.  74 18         JE SHORT RegMedic.0048C543   
0048C52B  |.  8B4D F0       MOV ECX,DWORD PTR SS:\[EBP\-10\]            //  这里再次取注册表里的值    
0048C52E  |.  BA ECC54800   MOV EDX,RegMedic.0048C5EC                //  ASCII "AppIdK"    
0048C533  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP\-C\]   
  
后面就没看跟它转了，因为再跟就跑去壳里去了.   
现在回到这里:   
  
004E243F  |.  E8 90AAFAFF   CALL RegMedic.0048CED4 //这里就跟就会在壳代码里晕来晕去(由此我个人猜它应该是用 svkp1.3x sdk 加的壳)，过了按 F8 过这里时会有异常，所以我又懒一下，跟了第一次，就不再有第二次了。    
004E2444  |.  A1 E0C74F00   MOV EAX,DWORD PTR DS:\[4FC7E0\]    //注意这里，这里如果\[4FC7E0\]=1 的话就注册成功,等于 0 的话就失败，所以我这里也用 fly 大侠的“狸猫换太子”给它易容。    
  
004E2449  |.  8038 00       CMP BYTE PTR DS:\[EAX\],0   
004E244C  |.  74 25         JE SHORT RegMedic.004E2473   
004E244E  |.  A1 64C84F00   MOV EAX,DWORD PTR DS:\[4FC864\]            //  这里 EAX 能等于 0，否则就完了    
004E2453  |.  8338 00       CMP DWORD PTR DS:\[EAX\],0   
004E2456  |.  74 0C         JE SHORT RegMedic.004E2464               //  这里就是"生死"跳,跳转成功就是死，否则就生    
004E2458  |.  A1 64C84F00   MOV EAX,DWORD PTR DS:\[4FC864\]   
004E245D  |.  8B00          MOV EAX,DWORD PTR DS:\[EAX\]   
004E245F  |.  E8 94D0FFFF   CALL RegMedic.004DF4F8   
004E2464  |>  8D45 EC       LEA EAX,DWORD PTR SS:\[EBP\-14\]   
004E2467  |.  BA 50254E00   MOV EDX,RegMedic.004E2550                //  ASCII "Thank you for registering Registry Medic."    
004E246C  |.  E8 CF18F2FF   CALL RegMedic.00403D40   
004E2471  |.  EB 0D         JMP SHORT RegMedic.004E2480   
004E2473  |>  8D45 EC       LEA EAX,DWORD PTR SS:\[EBP\-14\]   
004E2476  |.  BA 14254E00   MOV EDX,RegMedic.004E2514                //  ASCII "Registration failed. Please try again!"    
004E247B  |.  E8 C018F2FF   CALL RegMedic.00403D40   
……   
  
看完一堆代码后，我们就开始易容了。   
在数据窗口里找个是 01 的地方，我自己找的是 4E4359,因为我发现这里从一开始到后面这里的值都是 01.   
  
狸猫换太子开始了,在程序地盘里 CTRL+F7 找 MOV EAX,DWORD PTR DS:\[4FC7E0\]一共找到三个地方:   
  
004B2CF4    A1 E0C74F00     MOV EAX,DWORD PTR DS:\[4FC7E0\]   
//改为   
004B2CF4    B8 59434E00     MOV EAX,RegMedic.004E4359   
  
004E2444    A1 E0C74F00     MOV EAX,DWORD PTR DS:\[4FC7E0\]   
//改为   
004E2444    B8 59434E00     MOV EAX,RegMedic.004E4359   
  
  
004E25C8    A1 E0C74F00     MOV EAX,DWORD PTR DS:\[4FC7E0\]   
//改为   
004E25C8    B8 59434E00     MOV EAX,RegMedic.004E4359   
  
全部改完后，现在就是注册版了，这样重开后还是提示没注册呀，不要急，当然是这样的了，我们再做一个 loader 那不就可以了.   
用 keymaker 做一个内存补丁(也就是 loader 了)：   
  
修改地址    修改长度    原始指令    修改指令   
4E2444      4      A1E0C74F    B859434E   
4B2CF4      4      A1E0C74F    B859434E   
4E25C8      4      A1E0C74F    B859434E   
  
现在试试怎么样，注册成功了吧.注册码长度为&H27 位的情况也这样解决它。   
到这里我也应该收工了，我的问题还望各位大侠指点一二。各位老大看完了不要忘了帮我顶一下，我可是一中午没睡哦.我菜鸟一个所以文章中不可避免的存在问题，请多指教！   
  
【后    记】：   
  
做什么事都要灵活一点，如果没什么收获的话，不访和 MM 聊一下(我一下楼好多 PL 护士 MM,今天的灵感多少来自她们的)，也许就有灵感了。至于是不是真的是 SDK 的我也不敢肯定的说，因为并没有什么东西可以证明。   
  
  
献给我亲爱的 DFCG，愿它能够蒸蒸日上!   
Thanks:   
Fly 辉仔 yock、jingulong、二哥、所有曾经关心支持或帮助过我的朋友！谢谢您们!   
                 
                                    by  loveboom\[DFCG\]                              
              
                                            Email:bmd2chen@tom.com  
 4dd466d0e1e0cc123afbe504da2462ca
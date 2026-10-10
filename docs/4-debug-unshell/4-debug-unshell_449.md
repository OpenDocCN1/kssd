

# BadCopy v3.73.0106 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6422.htm`](https://www.pediy.com/kssd/pediy06/pediy6422.htm)

【脱文作者】 weiyi75\[Dfcg\]   
  
【作者邮箱】 weiyi75@sohu.com   
  
【作者主页】 Dfcg 官方大本营   
    
【使用工具】 Ollydbg1.10b,ImportREC1.6F,LoadPe   
    
【破解平台】 Win2000/XP   
    
【软件名称】 BadCopy v3.73.0106   
  
【下载地址】 http://soft.winzheng.com/SoftView/SoftView\_19156.htm   
  
【软件简介】  BadCopy 软件可以在不需要人工干预的情况下帮您读出 CD-ROM 或磁盘上的坏文件。效果显著，还具有智能修复的功能，最大程度挽回您的损失。新增或加强了以下特别功能：1、优化的算法，保证修复拯救数据更快更全。2、支持文件夹直接复制，您甚至可以直接把整个逻辑盘完整复制到另一个文件夹下。3、可以自由定制 BadCopy 的各项纠错参数，保证能高效，最大限度的挽救您的宝贵数据。4、可以直接对隐含或系统文件进行操作。5、文件操作安全性增强，危险操作先行提示，避免不必要的损失。6、可以直接在 BadCopy 里面运行或打开程序。    
    
【软件大小】 847K   
  
【加壳方式】 ASProtect 1.23 RC4 - 1.3.08.24 -> Alexey Solodovnikov   
  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享。   
  
【破解目的】 模拟跟踪寻找 Stolen Code 具体内容或 Oep 入口。   
  
通过二哥=weiyi75=David 作品 Asprotect1.23rc4 之 Dephi 语言篇经验，脱壳修复这个软件易如反掌。   
  
复习一下，寄存器 EBP=0012FFF0   //这是程序停在入口点 EBP 的值，其它 Dephi 语言或编程语言这里的 EBP 值全部相同，这又说明什么，当 Asprotect 运行到这里或 Stolen Code 做手脚时 EBP 的值就为 EBP=0012FFF0,我们来具体试试。   
  
OD 异常设置不忽略内存异常，其余全部忽略,载入程序,用插件隐藏 OD。   
  
00401000 >  68 0180BF00     PUSH BadCopy.00BF8001          //加壳程序入口点，F9 运行   
  
00401005    E8 01000000     CALL BadCopy.0040100B   
  
0040100A    C3              RETN   
  
0040100B    C3              RETN   
  
0040100C    14 D4           ADC AL,0D4   
  
0040100E    7B CD           JPO SHORT BadCopy.00400FDD   
  
00401010    E7 4A           OUT 4A,EAX                               ; I/O 命令   
  
00401012  - 77 95           JA SHORT BadCopy.00400FA9   
  
00401014    E8 7AEB4A1A     CALL 1A8AFB93   
  
00401019    D36B 0B         SHR DWORD PTR DS:\[EBX+B\],CL   
  
0040101C    C8 C33A68       ENTER 3AC3,68   
  
00401020  - E9 FB56C317     JMP 18036720   
  
00401025    8C03            MOV WORD PTR DS:\[EBX\],ES   
  
00401027    4D              DEC EBP   
  
00401028    9D              POPFD   
  
........................................................................   
  
    
  
内存异常   
  
    
  
015141A4    3100            XOR DWORD PTR DS:\[EAX\],EAX   
  
015141A6    EB 01           JMP SHORT 015141A9   
  
015141A8    68 648F0500     PUSH 58F64   
  
015141AD    0000            ADD BYTE PTR DS:\[EAX\],AL   
  
015141AF    00EB            ADD BL,CH   
  
015141B1    02E8            ADD CH,AL   
  
015141B3    0158 68         ADD DWORD PTR DS:\[EAX+68\],EBX   
  
015141B6    6C              INS BYTE PTR ES:\[EDI\],DX                 ; I/O 命令   
  
015141B7    AF              SCAS DWORD PTR ES:\[EDI\]   
  
015141B8    50              PUSH EAX   
  
015141B9    0168 14         ADD DWORD PTR DS:\[EAX+14\],EBP   
  
015141BC    42              INC EDX   
  
015141BD    51              PUSH ECX   
  
015141BE    0168 88         ADD DWORD PTR DS:\[EAX-78\],EBP   
  
015141C1    36:51           PUSH ECX                                 ; 多余的前缀   
  
015141C3    0168 50         ADD DWORD PTR DS:\[EAX+50\],EBP   
  
015141C6    3351 01         XOR EDX,DWORD PTR DS:\[ECX+1\]   
  
015141C9    68 002D5101     PUSH 1512D00   
  
015141CE    68 BC265101     PUSH 15126BC   
  
015141D3    68 4C3A5101     PUSH 1513A4C   
  
015141D8    C3              RETN   
  
........................................................................   
  
    
  
继续 Shift+F9 25 次来到 Asprotect 典型的最后一次异常。   
  
    
  
015139EC    3100            XOR DWORD PTR DS:\[EAX\],EAX    //特征码,这里先用 Loadpe 脱壳，避开后面的反脱壳代码。   
  
015139EE    64:8F05 0000000>POP DWORD PTR FS:\[0\]   
  
015139F5    58              POP EAX   
  
015139F6    833D B07E5101 0>CMP DWORD PTR DS:\[1517EB0\],0   
  
015139FD    74 14           JE SHORT 01513A13   
  
015139FF    6A 0C           PUSH 0C   
  
01513A01    B9 B07E5101     MOV ECX,1517EB0   
  
01513A06    8D45 F8         LEA EAX,DWORD PTR SS:\[EBP-8\]   
  
01513A09    BA 04000000     MOV EDX,4   
  
01513A0E    E8 2DD1FFFF     CALL 01510B40   
  
01513A13    FF75 FC         PUSH DWORD PTR SS:\[EBP-4\]   
  
01513A16    FF75 F8         PUSH DWORD PTR SS:\[EBP-8\]   
  
01513A19    8B45 F4         MOV EAX,DWORD PTR SS:\[EBP-C\]   
  
01513A1C    8338 00         CMP DWORD PTR DS:\[EAX\],0   
  
01513A1F    74 02           JE SHORT 01513A23   
  
01513A21    FF30            PUSH DWORD PTR DS:\[EAX\]   
  
01513A23    FF75 F0         PUSH DWORD PTR SS:\[EBP-10\]   
  
01513A26    FF75 EC         PUSH DWORD PTR SS:\[EBP-14\]   
  
01513A29    C3              RETN  //这里下断点，Shift+F9 中断后取消断点。   
  
........................................................................   
  
    
  
我这次没有用内存镜像断点到 OEP,下面的这个方法可以发现 Stolen Code(如果有)+Oep 二合一。   
  
    
  
仍然是命令行断点   
  
    
  
tc ebp==12fff0       //当前 EBP 内容为 12fff0 就被中断，略微看看就可以找到 Stolen Code(如果有)或 Oep   
  
    
  
01525262    F2:             PREFIX REPNE:  //一会就来到这里,呵呵，Dephi 语言就开始有 Stolen Code,往下全部用 F7 步过,转标签 1                              ; 多余的前缀   
  
01525263    EB 01           JMP SHORT 01525266   
  
01525265    F3:             PREFIX REP:    //这样的语句其实是一些变形 Jmp    ; 多余的前缀   
  
01525266    EB 01           JMP SHORT 01525269   
  
01525268    9A 8D6424EF F3E>CALL FAR EBF3:EF24648D                   ; 远距呼叫   
  
0152526F    02CD            ADD CL,CH   
  
01525271    208D 64246F8D   AND BYTE PTR SS:\[EBP+8D6F2464\],CL   
  
01525277    64:24 9E        AND AL,9E                                ; 多余的前缀   
  
0152527A    EB 02           JMP SHORT 0152527E   
  
0152527C    CD 20           INT 20   
  
0152527E    894424 00       MOV DWORD PTR SS:\[ESP\],EAX   
  
01525282    26:EB 02        JMP SHORT 01525287                       ; 多余的前缀   
  
01525285    CD 20           INT 20   
  
01525287    8B4424 18       MOV EAX,DWORD PTR SS:\[ESP+18\]   
  
0152528B    EB 01           JMP SHORT 0152528E   
  
.......................................................................   
  
    
  
标签 1   
  
    
  
01525266   /EB 01           JMP SHORT 01525269   
  
    
  
01525269    8D6424 EF       LEA ESP,DWORD PTR SS:\[ESP-11\]   
  
0152526D    F3:             PREFIX REP:                              ; 多余的前缀   
  
    
  
01525272    8D6424 6F       LEA ESP,DWORD PTR SS:\[ESP+6F\]   
  
01525276    8D6424 9E       LEA ESP,DWORD PTR SS:\[ESP-62\]   
  
0152527A    EB 02           JMP SHORT 0152527E   
  
    
  
0152527E    894424 00       MOV DWORD PTR SS:\[ESP\],EAX   
  
01525282    26:EB 02        JMP SHORT 01525287                       ; 多余的前缀   
  
    
  
01525287    8B4424 18       MOV EAX,DWORD PTR SS:\[ESP+18\]   
  
0152528B    EB 01           JMP SHORT 0152528E   
  
    
  
0152528E   /64:EB 02        JMP SHORT 01525293                       ; 多余的前缀   
  
    
  
01525293    0FBCC0          BSF EAX,EAX   
  
01525296    F2:             PREFIX REPNE:                            ; 多余的前缀   
  
    
  
0152529A    81E8 FF942BC5   SUB EAX,C52B94FF   
  
015252A0    2E:EB 01        JMP SHORT 015252A4                       ; 多余的前缀   
  
    
  
015252A4    8D83 AB525201   LEA EAX,DWORD PTR DS:\[EBX+15252AB\]   
  
015252AA    2BC3            SUB EAX,EBX   
  
015252AC    FF50 07         CALL DWORD PTR DS:\[EAX+7\]   
  
    
  
015252BA    58              POP EAX                                  ; 015252AF   
  
015252BB    26:EB 01        JMP SHORT 015252BF                       ; 多余的前缀   
  
    
  
015252BF    23C7            AND EAX,EDI   
  
015252C1    58              POP EAX   
  
015252C2    F3:             PREFIX REP:                              ; 多余的前缀   
  
    
  
015252C7    55              PUSH EBP      //Stolen Code 第一句。   
  
015252C8    8BEC            MOV EBP,ESP   //执行完它后，EBP=ESP=12FFC0,打破了从跟踪到这里 EBP 的值一直为 12fff0 的情况，同时也让我们也可以知道入口代码就在附近。Stolen Code 第二句。   
  
015252CA    81EC 10000000   SUB ESP,10    //第三句,这句要改为 ADD ESP,10 标准化。   
  
015252D0    F3:             PREFIX REP:                              ; 多余的前缀   
  
    
  
后面有无数条小跳转，全部都是垃圾，我跟踪了 40 分钟，还没走完,已经吐血了。跟据经验三句代码靠一切抽的一般要抽 11 个字节，不必往下浪费时间跟踪了。   
  
    
  
一般抽 11 字节的 Dephi 头部特征码    
  
    
  
PUSH EBP   
  
MOV EBP,ESP   
  
ADD ESP,-10      
  
MOV EAX,xxxxxxxx   
  
    
  
这个程序一共抽掉 11 字节。   
  
    
  
前面三句我们已经得到,现在的问题是 MOV EAX,xxxxxxxx   
  
    
  
EAX 的值是多少?     
  
    
  
ALT+M 打开内存镜像。   
  
    
  
内存镜像, 项目 12   
  
地址=00401000  //对 401000 Code 段下内存断点。   
  
大小=00100000 (1048576.)   
  
Owner=BadCopy  00400000   
  
区段=   
  
Contains=code   
  
类型=Imag 01001002   
  
访问=R   
  
初始访问=RWE   
  
    
  
00406EE0  - FF25 78B2B700   JMP DWORD PTR DS:\[B7B278\] //中断在这里,清除内存断点。   
  
00406EE6    8BC0            MOV EAX,EAX   
  
00406EE8  - FF25 74B2B700   JMP DWORD PTR DS:\[B7B274\]   
  
00406EEE    8BC0            MOV EAX,EAX   
  
00406EF0  - FF25 70B2B700   JMP DWORD PTR DS:\[B7B270\]   
  
00406EF6    8BC0            MOV EAX,EAX   
  
00406EF8  - FF25 6CB2B700   JMP DWORD PTR DS:\[B7B26C\]   
  
00406EFE    8BC0            MOV EAX,EAX   
  
00406F00    50              PUSH EAX   
  
00406F01    6A 40           PUSH 40   
  
00406F03    E8 E0FFFFFF     CALL BadCopy.00406EE8   
  
00406F08    C3              RETN   
  
....................................................   
  
    
  
01511C64    55              PUSH EBP //F8 返回壳中。   
  
01511C65    8BEC            MOV EBP,ESP   
  
01511C67    8B45 08         MOV EAX,DWORD PTR SS:\[EBP+8\]   
  
01511C6A    85C0            TEST EAX,EAX   
  
01511C6C    75 13           JNZ SHORT 01511C81   
  
01511C6E    813D A47A5101 0>CMP DWORD PTR DS:\[1517AA4\],400000        ; ASCII "MZP"   
  
01511C78    75 07           JNZ SHORT 01511C81   
  
01511C7A    A1 A47A5101     MOV EAX,DWORD PTR DS:\[1517AA4\]   
  
01511C7F    EB 06           JMP SHORT 01511C87   
  
01511C81    50              PUSH EAX   
  
01511C82    E8 3135FFFF     CALL 015051B8                            ; JMP to kernel32.GetModuleHandleA   
  
01511C87    5D              POP EBP   
  
01511C88    C2 0400         RETN 4 //Ctrl+F9 返回。   
  
......................................................   
  
    
  
00406FB5    A3 14775000     MOV DWORD PTR DS:\[507714\],EAX            ; BadCopy.00400000   
  
00406FBA    A1 14775000     MOV EAX,DWORD PTR DS:\[507714\]   
  
00406FBF    A3 B4105000     MOV DWORD PTR DS:\[5010B4\],EAX   
  
00406FC4    33C0            XOR EAX,EAX   
  
00406FC6    A3 B8105000     MOV DWORD PTR DS:\[5010B8\],EAX   
  
00406FCB    33C0            XOR EAX,EAX   
  
00406FCD    A3 BC105000     MOV DWORD PTR DS:\[5010BC\],EAX   
  
00406FD2    E8 C1FFFFFF     CALL BadCopy.00406F98   
  
00406FD7    BA B0105000     MOV EDX,BadCopy.005010B0   
  
00406FDC    8BC3            MOV EAX,EBX       //这里 EBX 就是我们要的 EAX 的数值。00500638 记下来。   
  
00406FDE    E8 65D7FFFF     CALL BadCopy.00404748   
  
00406FE3    5B              POP EBX   
  
00406FE4    C3              RETN   
  
......................................................   
  
    
  
00500997    0000            ADD BYTE PTR DS:\[EAX\],AL  确定真 Oep 为 00500998   
  
00500999    0000            ADD BYTE PTR DS:\[EAX\],AL   
  
0050099B    0000            ADD BYTE PTR DS:\[EAX\],AL   
  
0050099D    0000            ADD BYTE PTR DS:\[EAX\],AL   
  
0050099F    0000            ADD BYTE PTR DS:\[EAX\],AL   
  
005009A1    0000            ADD BYTE PTR DS:\[EAX\],AL   
  
005009A3    E8 FC65F0FF     CALL BadCopy.00406FA4   
  
005009A8    B0 01           MOV AL,1         //临时 Oep,滚动条往上看。   
  
005009AA    84C0            TEST AL,AL   
  
005009AC    74 59           JE SHORT BadCopy.00500A07   
  
005009AE    A1 D8615000     MOV EAX,DWORD PTR DS:\[5061D8\]   
  
005009B3    8B00            MOV EAX,DWORD PTR DS:\[EAX\]   
  
005009B5    E8 FE0CF7FF     CALL BadCopy.004716B8   
  
005009BA    A1 D8615000     MOV EAX,DWORD PTR DS:\[5061D8\]   
  
005009BF    8B00            MOV EAX,DWORD PTR DS:\[EAX\]   
  
005009C1    BA 140A5000     MOV EDX,BadCopy.00500A14                 ; ASCII "BadCopy Pro"   
  
005009C6    E8 F908F7FF     CALL BadCopy.004712C4   
  
005009CB    8B0D D45D5000   MOV ECX,DWORD PTR DS:\[505DD4\]            ; BadCopy.00B6A1F8   
  
005009D1    A1 D8615000     MOV EAX,DWORD PTR DS:\[5061D8\]   
  
005009D6    8B00            MOV EAX,DWORD PTR DS:\[EAX\]   
  
005009D8    8B15 D4C24E00   MOV EDX,DWORD PTR DS:\[4EC2D4\]            ; BadCopy.004EC320   
  
005009DE    E8 ED0CF7FF     CALL BadCopy.004716D0   
  
......................................................   
  
    
  
现在我们已经得到了   
  
    
  
1.  脱壳文件   
  
    
  
2.  被抽 11 个字节代码   
  
    
  
PUSH EBP   
  
MOV EBP,ESP   
  
ADD ESP,-10      
  
MOV EAX,00500638   
  
    
  
剩下 IAT 修复，关闭 OD，打开加壳程序，呵呵 OEP 填 1000,ImportREC 把 Size 认为 000000B4 太小了，改为 1000,点获得输入表。显示无效，先用等级 1 方式跟踪，然后用 Asprtect1.22 插件再减轻一下压力。还剩 563 个指针没修复。   
  
    
  
现在的问题是,这 563 个指针是有用的吗?ImpRec 1.6 会告诉你   
  
    
  
点击 show invalid 并右键单击选择其中的一个，让我们来修复它                           
  
    
  
0077b234<- 右键单击选择 DeasmHexView 查看:     
  
    
  
ImpRec 1.6 提示你,read error!好,够了,只要有这个提示的一律可以安全的 Cut。   
  
    
  
如果这个指针有效,里面会显示汇编指令。   
  
    
  
找了一会,发现 0077B3A0 处的指针是唯一一个没识别的指针。   
  
    
  
右键单击选择 DeasmHexView 查看,究竟它是 Kernel32.dll 中的哪个函数呢,按 Page Down 两次,看到   
  
    
  
01511D2E    call 01505158    // = kernel32.dll/00B3/FindResourceA   
  
    
  
第一个看到的 Api 函数就是要修复的指针。   
  
    
  
呵呵,修复它吧,双击 0077B3A0 这行,填入 Api 函数 FindResourceA。   
  
    
  
拿起剪刀，将剩下 562 个垃圾指针全部 Cut,IAT 全部修复，爽啊。修正 Oep 为 100998 最后修复文件,最后 Od 载入脱壳修复的程序，补上 11 个字节代码，保存为一个文件，正常运行。   
  
  
【破解总结】发现 Dephi 语言的 Asprotect1.23Rc4 的壳开始用 Stolen Code，并且有变形指令，反脱壳代码，最麻烦,tc ebp==12fff0 还不错吧。作者没有使用 Asprotect 内置的 RSA 算法保护，而采用自己的注册算法，任然是 RSA1024,破解参考二哥=weiyi75=David 的再次分析坏盘拷贝破文,感谢你耐心看完。   
  
    
  
\--------------------------------------------------------------------------------   
  
    
  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!   
    
 4dd466d0e1e0cc123afbe504da2462ca
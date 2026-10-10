

# 灰鸽子木马来源追踪

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-223.htm`](https://www.pediy.com/kssd/pediy08/pediy8-223.htm)

灰鸽子木马来源追踪  
【目     标】:N/A  
【工     具】:OllyDBG 1.1  
【任     务】:木马来源追踪   
【操作平台】:Windows xp sp2   
【作     者】: LOVEBOOM\[DFCG\]\[FCG\]\[CUG\]  
【相关链接】: N/A  
【简要说明】: 今天在硬盘里找到个木马，用几款杀毒软件杀了下没有结果。自己用 OD 看了下，发现是灰鸽子的修改版(很反感灰鸽子的作者,搞的到处是“垃圾”)。在网上简单的搜索了一下没有发现怎么揪出放木马的者的信息的相关文章，于是乎自己大概的分析了，写出来方便各位以后找源头。  
【详细过程】:  
今天安装软件时，系统提示没有多少空间可用，于是整理了下硬盘，这一整就整出了这么个东西，在程序文件夹下发现一个可疑的程序，看看生成日期，嗯，不错，程序差不多放了一个月了。看看程序的相关信息先：  
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
程序路径:    c:\\program files\\message\\message.exe  
属性:        隐藏  
创建时间:     2005 年 12 月 12 日, 16:19:21  
修改时间:    2005 年 9 月 9 日, 20:25:20  
文件大小:     301 KB (308,920 字节)  
PEID 扫描:     ASPack 2.x (without poly) -> Alexey Solodovnikov  
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
一看名字感觉就是不对劲了，十有九是中木马或病毒了，把自己的卡巴更新一下病毒库，查一下提示没有病毒，在网上下一个江民在线查毒，查一下也提示没有病毒。自己想了下，没有可能的我自己在那天根本没有装过这类可疑的软件。看来得自己动手了，用 lordpe 看看程序信息：  
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
程序入口:    000CA000h  
section name:    fuck you  
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
看来好像有人和我有深仇大恨了(我电脑里没有什么宝贝，最多只能拿我到一些未公布的脱壳和文章而已)。呵呵，既然来了，不管是敌是友，我都会接待的:- )。打开 OD 看了下入口信息，初步可以判断是和 aspack 变形版加的壳，是 aspack 加的壳就好办，把自己的脱壳脚本翻出来，运行到 OEP 处:  
004A5AAC   .  55            PUSH EBP  
004A5AAD   .  8BEC          MOV EBP,ESP  
004A5AAF   .  B9 06000000   MOV ECX,6  
004A5AB4   >  6A 00         PUSH 0  
呵呵，DELPHI 写的东西，在 OEP 不远处可以看到很有用的东西了。  
  
  
004A5ACB   .  68 465E4A00   PUSH 004A5E46  
004A5AD0   .  64:FF30       PUSH DWORD PTR FS:\[EAX\]  
004A5AD3   .  64:8920       MOV DWORD PTR FS:\[EAX\],ESP  
004A5AD6   .  E8 99A8FFFF   CALL 004A0374  
004A5ADB   .  8D55 EC       LEA EDX,DWORD PTR SS:\[EBP\-14\]  
004A5ADE   .  B8 605E4A00   MOV EAX,004A5E60                         ;  ASCII "www.huigezi.net"  
004A5AE3   .  E8 24BBFFFF   CALL 004A160C  
004A5AE8   .  8D55 E4       LEA EDX,DWORD PTR SS:\[EBP\-1C\]  
004A5AEB   .  33C0          XOR EAX,EAX  
004A5AED   .  E8 AAD0F5FF   CALL 00402B9C  
004A5AF2   .  8B45 E4       MOV EAX,DWORD PTR SS:\[EBP\-1C\]  
004A5AF5   .  8D55 E8       LEA EDX,DWORD PTR SS:\[EBP\-18\]  
004A5AF8   .  E8 533DF6FF   CALL 00409850  
004A5AFD   .  8B55 E8       MOV EDX,DWORD PTR SS:\[EBP\-18\]  
004A5B00   .  B8 88BF4A00   MOV EAX,004ABF88  
004A5B05   .  E8 E6EEF5FF   CALL 004049F0  
004A5B0A   .  A1 88BF4A00   MOV EAX,DWORD PTR DS:\[4ABF88\]  
004A5B0F   .  E8 48F3F5FF   CALL 00404E5C  
004A5B14   .  50            PUSH EAX                                 ; /String2 = FFFFFFFF ???  
004A5B15   .  68 705E4A00   PUSH 004A5E70                            ; |String1 = "IExplore.exe"  
004A5B1A   .  E8 6D16F6FF   CALL 0040718C                            ; \\lstrcmpiA  
004A5B1F   .  85C0          TEST EAX,EAX  
004A5B21   .  0F94C0        SETE AL  
上面都显示出是 huigezi 的网站，那基本上就可以判断出是灰鸽子木马了，到这里我以为就结束了，既然是灰鸽子当然先是去网上把杀灰鸽子木马的软件，下了个灰鸽子 vip 2005 专杀工具，一查软件告诉我没有找到灰鸽子，俺一想也是可能我这里把木马给关掉了所以查不到，那就把程序运行起来，运行起来后，看看进程里多了个 iexplorer.exe(看来是注入系统进程了)，再查，还是提示没有发现木马。看来是被人改过了，再去找个瑞星的专杀工具和安天的试了下，都提示没有木马,这回真的得手工处理了,既然要我手工处理我就来个更远一点的，揪出放木马的者,看看是谁这么无聊，这么喜欢看别人的隐私。好了，开始想怎么去找线索，程序要把我们的信息给放木马者，放木马者肯定要在放之前配置好，告诉程序走哪条路，怎么走之类的，那么这些信息又存放哪里呢？我想了下一般放这么几个地方吧：  
一、  放到一个配置文件里，然后和程序文件一起，程序运行后读取信息。既然是木马我想没有谁会这么招摇吧，这样想来，就否认了这种方式。  
二、  把配置信息和程序绑在一起，这里又有 3 种常见的方式：  
1、  用绑定机把配置信息和程序绑成一个文件，但想一下，再加一个 shell 程序不又得变大多少 KB 了，因此想来这种方法可能性不大。  
2、  把配置信息作为程序的附加数据，用 PEID 看了下，提示并没有附加数据。这样看来，很有可能是第三种方式了。  
3、  把配置信息作为程序资源的一部分，在读取的时候通过 Findresource 来读取信息。  
好了，初步推测是这样，打开 OD 来证实下，OD 载入目标，下断 FindResourceA,运行中断后，观察堆栈信息：  
0013FEAC   004A11C2  /CALL to FindResourceA from message.004A11BD  
0013FEB0   00400000  |hModule = 00400000 (message)  
0013FEB4   004A088C  |ResourceName = "SEVINFO"  
0013FEB8   0000000A  \\ResourceType = RT\_RCDATA  
执行到返回，一看没错，可以证实是用 2.3 的方式把配置信息作为资源来处理。  
004A11B5   .  50            PUSH EAX                                 ; /ResourceType = 250  
004A11B6   .  52            PUSH EDX                                 ; |ResourceName = 000BD000 ???  
004A11B7   .  A1 64A64A00   MOV EAX,DWORD PTR DS:\[4AA664\]            ; |  
004A11BC   .  50            PUSH EAX                                 ; |hModule = 00000250  
004A11BD   .  E8 B25CF6FF   CALL 00406E74                            ; \\FindResourceA  
004A11C2   .  8BD8          MOV EBX,EAX                              ;  返回到这里  
004A11C4   .  85DB          TEST EBX,EBX  
004A11C6   .  0F84 D5000000 JE 004A12A1                              ;  如果没有找到资源则跳去结束处  
004A11CC   .  53            PUSH EBX                                 ; /hResource = 00000250 (window)  
004A11CD   .  A1 64A64A00   MOV EAX,DWORD PTR DS:\[4AA664\]            ; |  
004A11D2   .  50            PUSH EAX                                 ; |hModule = 00000250  
004A11D3   .  E8 745EF6FF   CALL 0040704C                            ; \\LoadResource  
004A11D8   .  8BF0          MOV ESI,EAX                              ;  载入自定义资源  
004A11DA   .  85F6          TEST ESI,ESI                             ;  message.004C4BA4  
004A11DC   .  0F84 BF000000 JE 004A12A1                              ;  如果载入资源失败跳去结束处  
004A11E2   .  53            PUSH EBX                                 ; /hResource = 00000250 (window)  
004A11E3   .  A1 64A64A00   MOV EAX,DWORD PTR DS:\[4AA664\]            ; |  
004A11E8   .  50            PUSH EAX                                 ; |hModule = 00000250  
004A11E9   .  E8 265FF6FF   CALL 00407114                            ; \\SizeofResource  
004A11EE   .  8BD8          MOV EBX,EAX  
本以为载入后就可以直接看到相关信息，d esi 看了下就会发现原来木马在配置服务器端的时候进行了加密处理，我们现在看到的只是加密后的东西。  
004C4BA4  44 38 35 45 36 41 31 35 46 33 45 41 32 45 34 32  D85E6A15F3EA2E42  
004C4BB4  39 32 43 46 35 33 37 34 42 43 30 41 45 46 35 46  92CF5374BC0AEF5F  
004C4BC4  38 36 37 39 44 46 36 41 45 34 33 39 34 36 42 38  8679DF6AE43946B8  
004C4BD4  32 35 35 35 38 39 32 32 32 35 35 44 37           25558922255D7  
不管它怎么加密程序最终还是要还原出原来的东西，既然要还原正确的信息，那就就一定行读取加密后的信息了，这样我们就在 4C4BA4 处下内存访问断点,mr 4C4BA4àRUN,运行后中断：  
00402A3C  |.  FD            STD  
00402A3D  |.  F3:A5         REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS\>;  第一次中断在这里  
00402A3F  |.  89C1          MOV ECX,EAX  
00402A41  |.  83E1 03       AND ECX,3  
00402A44  |.  83C6 03       ADD ESI,3  
00402A47  |.  83C7 03       ADD EDI,3  
00402A4A  |.  F3:A4         REP MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[>  
00402A4C  |.  FC            CLD  
STD 逆时针解密，在 402a4a 处断下后定位到 edi 所在的地址。然后两次执行到返回(CTRL+F9)，返回到如下地址:  
004A03A7   .  BA 8C084A00   MOV EDX,004A088C                         ;  ASCII "SEVINFO"  
004A03AC   .  B9 02000000   MOV ECX,2  
004A03B1   .  B8 0A000000   MOV EAX,0A  
004A03B6   .  E8 DD0D0000   CALL 004A1198  
004A03BB   .  84C0          TEST AL,AL                               ;  返回到这里  
004A03BD   .  0F84 7F040000 JE <ExitProc>                            ;  如果读取配置信息失败则跳去 ExitProcess  
004A03C3   .  B2 01         MOV DL,1  
004A03C5   .  A1 F0364100   MOV EAX,DWORD PTR DS:\[4136F0\]  
004A03CA   .  E8 AD37F6FF   CALL 00403B7C  
004A03CF   .  8945 F8       MOV DWORD PTR SS:\[EBP\-8\],EAX  
004A03D2   .  8D4D F4       LEA ECX,DWORD PTR SS:\[EBP\-C\]  
004A03D5   .  BA 9C084A00   MOV EDX,004A089C                         ;  ASCII "20050101"  
004A03DA   .  A1 E8BE4A00   MOV EAX,DWORD PTR DS:\[4ABEE8\]  
004A03DF   .  E8 98D3FBFF   CALL <Decrypt>                           ;  这里进去解密出配置文件的信息  
004A03E4   .  8B55 F4       MOV EDX,DWORD PTR SS:\[EBP\-C\]             ;  到了这里 d \[ebp+c\]看到了什么  
004A03E7   .  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP\-8\]  
004A03EA   .  8B08          MOV ECX,DWORD PTR DS:\[EAX\]  
004A03EC   .  FF51 2C       CALL DWORD PTR DS:\[ECX+2C\]  
004A03EF   .  8D55 F0       LEA EDX,DWORD PTR SS:\[EBP\-10\]  
004A03F2   .  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP\-8\]  
004A03F5   .  8B08          MOV ECX,DWORD PTR DS:\[EAX\]  
004A03F7   .  FF51 1C       CALL DWORD PTR DS:\[ECX+1C\]  
004A03FA   .  8B45 F0       MOV EAX,DWORD PTR SS:\[EBP\-10\]            ;  分出端口号  
004A03FD   .  E8 5A4AF6FF   CALL 00404E5C  
004A0402   .  50            PUSH EAX                                 ; /String = "4B?  
004A0403   .  E8 7C6CF6FF   CALL 00407084                            ; \\OutputDebugStringA  
004A0408   .  8D4D EC       LEA ECX,DWORD PTR SS:\[EBP\-14\]            ;  把上面的配置信息显示到调试器,到了这里看看 OD 的状态栏是不是显示出了相关的信息  
004A040B   .  33D2          XOR EDX,EDX  
004A040D   .  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP\-8\]  
004A0410   .  8B18          MOV EBX,DWORD PTR DS:\[EAX\]  
004A0412   .  FF53 0C       CALL DWORD PTR DS:\[EBX+C\]                ;  message.004178FC  
004A0415   .  8B55 EC       MOV EDX,DWORD PTR SS:\[EBP\-14\]            ;  取出端口号  
004A0418   .  B8 20BF4A00   MOV EAX,004ABF20  
004A041D   .  E8 CE45F6FF   CALL 004049F0  
004A0422   .  8D4D E8       LEA ECX,DWORD PTR SS:\[EBP\-18\]  
004A0425   .  BA 01000000   MOV EDX,1  
004A042A   .  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP\-8\]  
004A042D   .  8B18          MOV EBX,DWORD PTR DS:\[EAX\]  
004A042F   .  FF53 0C       CALL DWORD PTR DS:\[EBX+C\]                ;  message.004178FC  
004A0432   .  8B55 E8       MOV EDX,DWORD PTR SS:\[EBP\-18\]            ;  取出访问的地址  
004A0435   .  B8 34BF4A00   MOV EAX,004ABF34  
004A043A   .  E8 B145F6FF   CALL 004049F0  
004A043F   .  8D4D E4       LEA ECX,DWORD PTR SS:\[EBP\-1C\]  
004A0442   .  BA 02000000   MOV EDX,2  
004A0447   .  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP\-8\]  
004A044A   .  8B18          MOV EBX,DWORD PTR DS:\[EAX\]  
004A044C   .  FF53 0C       CALL DWORD PTR DS:\[EBX+C\]                ;  message.004178FC  
004A044F   .  8B55 E4       MOV EDX,DWORD PTR SS:\[EBP\-1C\]            ;  取出密码  
004A0452   .  B8 38BF4A00   MOV EAX,004ABF38  
004A0457   .  E8 9445F6FF   CALL 004049F0  
004A045C   .  8D4D E0       LEA ECX,DWORD PTR SS:\[EBP\-20\]  
004A045F   .  BA 03000000   MOV EDX,3  
004A0464   .  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP\-8\]  
004A0467   .  8B18          MOV EBX,DWORD PTR DS:\[EAX\]  
004A0469   .  FF53 0C       CALL DWORD PTR DS:\[EBX+C\]                ;  message.004178FC  
004A046C   .  8B55 E0       MOV EDX,DWORD PTR SS:\[EBP\-20\]            ;  取出程序存放的路径  
004A046F   .  B8 D4BE4A00   MOV EAX,004ABED4  
004A0474   .  E8 7745F6FF   CALL 004049F0  
004A0479   .  8D55 DC       LEA EDX,DWORD PTR SS:\[EBP\-24\]  
004A047C   .  A1 D4BE4A00   MOV EAX,DWORD PTR DS:\[4ABED4\]  
004A0481   .  E8 BEFBFFFF   CALL 004A0044  
004A0486   .  8B55 DC       MOV EDX,DWORD PTR SS:\[EBP\-24\]            ;  取出实际路径  
004A0489   .  B8 D4BE4A00   MOV EAX,004ABED4  
004A048E   .  E8 5D45F6FF   CALL 004049F0  
004A0493   .  8D55 D8       LEA EDX,DWORD PTR SS:\[EBP\-28\]  
004A0496   .  A1 D4BE4A00   MOV EAX,DWORD PTR DS:\[4ABED4\]  
004A049B   .  E8 B093F6FF   CALL 00409850                            ;  取出 EXE 文件名  
004A04A0   .  8B55 D8       MOV EDX,DWORD PTR SS:\[EBP\-28\]  
004A04A3   .  B8 D8BE4A00   MOV EAX,004ABED8  
004A04A8   .  E8 4345F6FF   CALL 004049F0  
004A04AD   .  33C0          XOR EAX,EAX  
004A04AF   .  55            PUSH EBP  
004A04B0   .  68 E2044A00   PUSH 004A04E2  
004A04B5   .  64:FF30       PUSH DWORD PTR FS:\[EAX\]  
004A04B8   .  64:8920       MOV DWORD PTR FS:\[EAX\],ESP  
004A04BB   .  8D4D D4       LEA ECX,DWORD PTR SS:\[EBP\-2C\]  
004A04BE   .  BA 04000000   MOV EDX,4  
004A04C3   .  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP\-8\]  
004A04C6   .  8B18          MOV EBX,DWORD PTR DS:\[EAX\]  
004A04C8   .  FF53 0C       CALL DWORD PTR DS:\[EBX+C\]                ;  message.004178FC  
004A04CB   .  8B45 D4       MOV EAX,DWORD PTR SS:\[EBP\-2C\]            ;  取出端口?  
004A04CE   .  E8 518CF6FF   CALL <**StrtoInt**\>                          ;  转换为数字  
004A04D3   .  A3 30BF4A00   MOV DWORD PTR DS:\[4ABF30\],EAX            ;  保存端口  
004A04D8   .  33C0          XOR EAX,EAX  
004A04DA   .  5A            POP EDX                                  ;  0013FEF8  
004A04DB   .  59            POP ECX                                  ;  0013FEF8  
004A04DC   .  59            POP ECX                                  ;  0013FEF8  
004A04DD   .  64:8910       MOV DWORD PTR FS:\[EAX\],EDX  
004A04E0   .  EB 14         JMP SHORT 004A04F6  
004A04E2   .^ E9 653BF6FF   JMP 0040404C  
004A04E7   .  C705 30BF4A00>MOV DWORD PTR DS:\[4ABF30\],32  
004A04F1   .  E8 BE3EF6FF   CALL 004043B4  
004A04F6   >  8D4D D0       LEA ECX,DWORD PTR SS:\[EBP\-30\]  
004A04F9   .  BA 05000000   MOV EDX,5  
004A04FE   .  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP\-8\]  
004A0501   .  8B18          MOV EBX,DWORD PTR DS:\[EAX\]  
004A0503   .  FF53 0C       CALL DWORD PTR DS:\[EBX+C\]                ;  取出 name information  
004A0506   .  8B55 D0       MOV EDX,DWORD PTR SS:\[EBP\-30\]  
004A0509   .  B8 2CBF4A00   MOV EAX,004ABF2C  
004A050E   .  E8 DD44F6FF   CALL 004049F0  
004A0513   .  8D4D CC       LEA ECX,DWORD PTR SS:\[EBP\-34\]  
004A0516   .  BA 06000000   MOV EDX,6  
004A051B   .  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP\-8\]  
004A051E   .  8B18          MOV EBX,DWORD PTR DS:\[EAX\]  
004A0520   .  FF53 0C       CALL DWORD PTR DS:\[EBX+C\]                ;  message.004178FC  
004A0523   .  8B55 CC       MOV EDX,DWORD PTR SS:\[EBP\-34\]  
004A0526   .  B8 FCBE4A00   MOV EAX,004ABEFC  
004A052B   .  E8 C044F6FF   CALL 004049F0  
004A0530   .  33C0          XOR EAX,EAX  
004A0532   .  55            PUSH EBP  
004A0533   .  68 65054A00   PUSH 004A0565  
004A0538   .  64:FF30       PUSH DWORD PTR FS:\[EAX\]  
004A053B   .  64:8920       MOV DWORD PTR FS:\[EAX\],ESP  
004A053E   .  8D4D C8       LEA ECX,DWORD PTR SS:\[EBP\-38\]  
004A0541   .  BA 07000000   MOV EDX,7  
004A0546   .  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP\-8\]  
004A0549   .  8B18          MOV EBX,DWORD PTR DS:\[EAX\]  
004A054B   .  FF53 0C       CALL DWORD PTR DS:\[EBX+C\]                ;  message.004178FC  
004A054E   .  8B45 C8       MOV EAX,DWORD PTR SS:\[EBP\-38\]            ;  取出端口  
004A0551   .  E8 CE8BF6FF   CALL <**StrtoInt**\>  
004A0556   .  A3 D0BE4A00   MOV DWORD PTR DS:\[4ABED0\],EAX  
004A055B   .  33C0          XOR EAX,EAX  
004A055D   .  5A            POP EDX                                  ;  0013FEF8  
004A055E   .  59            POP ECX                                  ;  0013FEF8  
004A055F   .  59            POP ECX                                  ;  0013FEF8  
004A0560   .  64:8910       MOV DWORD PTR FS:\[EAX\],EDX  
004A0563   .  EB 14         JMP SHORT 004A0579  
004A0565   .^ E9 E23AF6FF   JMP 0040404C  
004A056A   .  C705 D0BE4A00>MOV DWORD PTR DS:\[4ABED0\],1F40  
004A0574   .  E8 3B3EF6FF   CALL 004043B4  
004A0579   >  8D4D C4       LEA ECX,DWORD PTR SS:\[EBP\-3C\]  
004A057C   .  BA 08000000   MOV EDX,8  
004A0581   .  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP\-8\]  
004A0584   .  8B18          MOV EBX,DWORD PTR DS:\[EAX\]  
004A0586   .  FF53 0C       CALL DWORD PTR DS:\[EBX+C\]                ;  message.004178FC  
004A0589   .  8B45 C4       MOV EAX,DWORD PTR SS:\[EBP\-3C\]  
004A058C   .  BA B0084A00   MOV EDX,004A08B0  
004A0591   .  E8 1248F6FF   CALL 00404DA8  
004A0596   .  75 07         JNZ SHORT 004A059F                       ;  判断相关标志  
004A0598   .  C605 3CBF4A00>MOV BYTE PTR DS:\[4ABF3C\],1  
004A059F   >  8D4D C0       LEA ECX,DWORD PTR SS:\[EBP\-40\]  
……(省略 N 行代码)  
到这里我们就可以获取出放木马者的基本信息了，我这个木马作者配置文件的相关信息如下：  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
          Config  
10395  
softck.oicp.net  
8b4ca58172880bbb  
$(ProgramFiles)\\message\\message.exe  
83  
服装纺织  
服装纺织  
8000  
0  
1  
0  
1  
1  
1  
1  
COM+ Server  
COM+ Server  
传输客户端和服务器之间的 NET **SEND** 和 Alerter 服务消息。此服务与 Windows Messenger 无关。如果服务停止  
0  
1080  
guest  
huigezi  
0  
8080  
0  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
我连了一下放木马者的网上，是用的花生壳，主机还没有开机没连上。  
执行到返回后，一路 F8 看看这里：  
004A5DA2   . /75 13         JNZ SHORT 004A5DB7  
004A5DA4   . |6A 40         PUSH 40                                  ; /Style = MB\_OK|MB\_ICONASTERISK|MB\_APPLMODAL  
004A5DA6   . |68 D05E4A00   PUSH 004A5ED0                            ; |Title = "提示"  
004A5DAB   . |68 D85E4A00   PUSH 004A5ED8                            ; |Text = "灰鸽子远程控制服务端安装成功！"  
004A5DB0   . |6A 00         PUSH 0                                   ; |hOwner = NULL  
004A5DB2   . |E8 DD19F6FF   CALL 00407794                            ; \\MessageBoxA  
004A5DB7   > \\A1 BC894A00   MOV EAX,DWORD PTR DS:\[4A89BC\]  
004A5DBC   .  8038 00       CMP BYTE PTR DS:\[EAX\],0  
004A5DBF   .  74 0E         JE SHORT 004A5DCF  
004A5DC1   .  803D 90BF4A00>CMP BYTE PTR DS:\[4ABF90\],0  
004A5DC8   .  75 05         JNZ SHORT 004A5DCF  
004A5DCA   .  E8 79F2FFFF   CALL 004A5048  
004A5DCF   >  6A 00         PUSH 0                                   ; /ExitCode = 0  
004A5DD1   .  E8 5E10F6FF   CALL 00406E34                            ; \\ExitProcess  
004A5DD6   .  EB 53         JMP SHORT 004A5E2B  
004A5DD8   >  E8 B7D5FFFF   CALL <ChkOS>                             ;  这里判断操作系统是否为 NT  
004A5DDD   .  3C 01         CMP AL,1  
004A5DDF   .  75 45         JNZ SHORT <isWin9x>  
004A5DE1   .  C605 84BF4A00>MOV BYTE PTR DS:\[4ABF84\],1  
004A5DE8   .  A1 188D4A00   MOV EAX,DWORD PTR DS:\[4A8D18\]  
004A5DED   .  8038 01       CMP BYTE PTR DS:\[EAX\],1  
004A5DF0   .  75 2D         JNZ SHORT 004A5E1F  
004A5DF2   .  A1 9C8F4A00   MOV EAX,DWORD PTR DS:\[4A8F9C\]  
004A5DF7   .  8B00          MOV EAX,DWORD PTR DS:\[EAX\]  
004A5DF9   .  E8 5EF0F5FF   CALL 00404E5C  
004A5DFE   .  8B15 E48C4A00 MOV EDX,DWORD PTR DS:\[4A8CE4\]            ;  message.004ABF50  
004A5E04   .  8902          MOV DWORD PTR DS:\[EDX\],EAX               ;  message.004ABF50  
004A5E06   .  A1 E48C4A00   MOV EAX,DWORD PTR DS:\[4A8CE4\]  
004A5E0B   .  C740 04 90554>MOV DWORD PTR DS:\[EAX+4\],004A5590  
004A5E12   .  A1 E48C4A00   MOV EAX,DWORD PTR DS:\[4A8CE4\]            ;  如果是 WinNT 系统则启动服务  
004A5E17   .  50            PUSH EAX                                 ; /pServiceTable = message.004ABF50  
004A5E18   .  E8 CF2AFBFF   CALL 004588EC                            ; \\StartServiceCtrlDispatcherA  
004A5E1D   .  EB 0C         JMP SHORT 004A5E2B  
004A5E1F   >  E8 28F6FFFF   CALL 004A544C  
……  
看看都有些什么吧:- ).  
004A5E58   .  FFFFFFFF      DD FFFFFFFF  
004A5E5C   .  0F000000      DD 0000000F  
004A5E60   .  77 77 77 2E 6>ASCII "www.huigezi.net",0  
004A5E70   .  49 45 78 70 6>ASCII "IExplore.exe",0  
004A5E7D      00            DB 00  
004A5E7E      00            DB 00  
004A5E7F      00            DB 00  
004A5E80   .  56 49 50 32 2>ASCII "VIP2.0\_MUTEX",0  
004A5E8D      00            DB 00  
004A5E8E      00            DB 00  
004A5E8F      00            DB 00  
004A5E90   .  FFFFFFFF      DD FFFFFFFF  
004A5E94   .  04000000      DD 00000004  
004A5E98   .  2E 4E 45 57 0>ASCII ".NEW",0  
004A5E9D      00            DB 00  
004A5E9E      00            DB 00  
004A5E9F      00            DB 00  
004A5EA0   .  53 6F 66 74 5>ASCII "SoftWare\\Microso"  
004A5EB0   .  66 74 5C 57 6>ASCII "ft\\Windows\\Curre"  
004A5EC0   .  6E 74 56 65 7>ASCII "ntVersion\\Run",0  
如果是直接用 OD 运行程序的程序，程序就会先获取相关配置信息，然后如果是 WinNT 系统则启动相关服务，然后自己自身退出。好了，到这里就比较简单的分析出放木马者的信息，因为我自已连不上放木马者的网站，再则对灰鸽子的配置也不懂，因此就此打住，如果你有兴趣的话，可以联系我得到目标文件。下面是大概的清理方法(我的系统重做了，因此只写我自己记得的部分)：  
删除进程中的 iexplorer 进程(我从来不用 IE 上网的)，停止服务里的 COM+ server 服务，然后删除服务，删除相关文件。   
  
  
            LOVE WEN  
Greetz:  
 Fly.Jingulong,yock,tDasm.David.hexer,hmimys,ahao.UFO(brother).alan(sister).all of my friends and you!  
  
By loveboom\[DFCG\]\[FCG\]\[US\]  
http://blog.csdn.net/bmd2chen  
Email:loveboom#163.com  
Date:2006-1-11 8:00 

 4dd466d0e1e0cc123afbe504da2462ca
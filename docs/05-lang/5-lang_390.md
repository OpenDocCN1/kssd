
# VB 稍繁琐的算法－吉他和弦帮手 2.3

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-433.htm`](https://www.pediy.com/kssd/pediy07/pediy7-433.htm)

## 软件信息

【破文标题】VB 稍繁琐的算法－吉他和弦帮手 2.3

【软件名称】吉他和弦帮手 2.3

【软件介绍】收集大约 1500 个吉他和弦指法,多种 MIDI 乐器发声，可自定义右手弹奏指法界面还不错，开发环境 VB6。这次的软件比以前的版本 增加了一个大家都要求的功能，这个功能就是可以播放一个完整的和弦和曲子。该软件附上多首古典名曲【爱的罗曼斯】、【阿尔罕布拉宫的 回忆】、【舒伯特小夜曲】等提供参考，虽然这个软件需要注册，但是，我不收大家的一分钱，我只是想收集大家用我的软件制作的音乐文件 。所以，大家只要提供一个用我的软件编出的古典名曲（gch 文件），就可以免费获得注册码，绝不失言。当然，我收集的到的文件将会与大家 分享。 希望大家提供好一些的名曲。

【软件地址】[`www1.skycn.com/soft/11605.html`](http://www1.skycn.com/soft/11605.html)

【破文作者】KiLlL[DFCG][FCG]

【破解时间】2005-09-21 23:02 -> 09-22 03:02

【破解声明】仅限技术交流!

## 破解过程

作者已经不更新了，后来似乎发布了免注册版，所以分析应该是没有问题吧。这个是很好的一个和弦助手，可惜我不会用。软件使用 30 次，但 是在注册窗口输入之后没有比较，而是保存到软件目录的 `reg.ini` 文件里面。重启后验证程序，注册码必须为 20 位。

```
00507BA8   > \8B55 E0       MOV EDX,DWORD PTR SS:[EBP-20]
00507BAB   .  52           PUSH EDX                          ;  下面开始读取注册文件
00507BAC   .  68 30034500  PUSH GtChrdHl.00450330            ;  UNICODE "\\Reg.ini"
00507BB1   .  FF15 64104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCa>;  MSVBVM60.__vbaStrCat
00507BB7   .  8B3D 88124000 MOV EDI,DWORD PTR DS:[<&MSVBVM60.__vbaSt>;  MSVBVM60.__vbaStrMove
00507BBD   .  8BD0         MOV EDX,EAX                       ;  获得完整路径
00507BBF   .  8D4D E8      LEA ECX,DWORD PTR SS:[EBP-18]
00507BC2   .  FFD7        CALL EDI                            ;  <&MSVBVM60.__vbaStrMove>
00507BC4   .  8D4D E0      LEA ECX,DWORD PTR SS:[EBP-20]
00507BC7   .  FF15 CC124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStr
00507BCD   .  8D4D D4      LEA ECX,DWORD PTR SS:[EBP-2C]
00507BD0   .  FF15 C8124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeO>;  MSVBVM60.__vbaFreeObj
00507BD6   .  8D4D C4      LEA ECX,DWORD PTR SS:[EBP-3C]
00507BD9   .  8D45 E8      LEA EAX,DWORD PTR SS:[EBP-18]
00507BDC   .  53           PUSH EBX
00507BDD   .  51           PUSH ECX
00507BDE   .  8945 CC      MOV DWORD PTR SS:[EBP-34],EAX
00507BE1   .  C745 C4 08400>MOV DWORD PTR SS:[EBP-3C],4008           ;  判断是否存在这个文件
00507BE8   .  FF15 DC114000 CALL DWORD PTR DS:[<&MSVBVM60.#645>]      ;  MSVBVM60.rtcDir
00507BEE   .  8BD0         MOV EDX,EAX
00507BF0   .  8D4D E0      LEA ECX,DWORD PTR SS:[EBP-20]
00507BF3   .  FFD7        CALL EDI
00507BF5   .  8B1D 28114000 MOV EBX,DWORD PTR DS:[<&MSVBVM60.__vbaSt>;  MSVBVM60.__vbaStrCmp
00507BFB   .  50           PUSH EAX
00507BFC   .  68 18DB4400  PUSH GtChrdHl.0044DB18
00507C01   .  FFD3        CALL EBX                             ;  <&MSVBVM60.__vbaStrCmp>
00507C03   .  8BF0         MOV ESI,EAX
00507C05   .  8D4D E0      LEA ECX,DWORD PTR SS:[EBP-20]
00507C08   .  F7DE        NEG ESI
00507C0A   .  1BF6        SBB ESI,ESI
00507C0C   .  46           INC ESI
00507C0D   .  F7DE        NEG ESI
00507C0F   .  FF15 CC124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStr
00507C15   .  66:85F6     TEST SI,SI
00507C18   .  0F85 8D000000 JNZ GtChrdHl.00507CAB              ;  不存在就跳走
00507C1E   .  8B55 E8      MOV EDX,DWORD PTR SS:[EBP-18]
00507C21   .  52           PUSH EDX
00507C22   .  6A 01        PUSH 1
00507C24   .  6A FF       PUSH -1
00507C26   .  6A 01        PUSH 1                             ;  存在就打开
00507C28   .  FF15 E8114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFileO>;  MSVBVM60.__vbaFileOpen
00507C2E   .  8D45 E4      LEA EAX,DWORD PTR SS:[EBP-1C]
00507C31   .  6A 01        PUSH 1
00507C33   .  50           PUSH EAX                            ;  读取一行，就是读取出注册码
00507C34   .  FF15 28104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaLineI>;  MSVBVM60.__vbaLineInputStr
00507C3A   .  6A 01        PUSH 1
00507C3C   .  FF15 18114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFileC>;  MSVBVM60.__vbaFileClose
00507C42   .  E8 C9FCFFFF  CALL GtChrdHl.00507910             ;  这个是一个关键函数，应该是获取机器码的。
```

到这里看看

```
00507910   $  55           PUSH EBP
00507911   .  8BEC         MOV EBP,ESP
00507913   .  83EC 18      SUB ESP,18
00507916   .  68 06334000  PUSH <JMP.&MSVBVM60.__vbaExceptHandler>  ;  SE handler installation
0050791B   .  64:A1 0000000>MOV EAX,DWORD PTR FS:[0]
00507921   .  50           PUSH EAX
00507922   .  64:8925 00000>MOV DWORD PTR FS:[0],ESP
00507929   .  B8 AC000000  MOV EAX,0AC
0050792E   .  E8 CDB9EFFF  CALL <JMP.&MSVBVM60.__vbaChkstk>
00507933   .  53           PUSH EBX
00507934   .  56           PUSH ESI
00507935   .  57           PUSH EDI
00507936   .  8965 E8      MOV DWORD PTR SS:[EBP-18],ESP
00507939   .  C745 EC 10304>MOV DWORD PTR SS:[EBP-14],GtChrdHl.00403>
00507940   .  C745 F0 00000>MOV DWORD PTR SS:[EBP-10],0
00507947   .  C745 F4 00000>MOV DWORD PTR SS:[EBP-C],0
0050794E   .  C745 FC 01000>MOV DWORD PTR SS:[EBP-4],1
00507955   .  C745 FC 02000>MOV DWORD PTR SS:[EBP-4],2
0050795C   .  6A FF       PUSH -1
0050795E   .  FF15 B4104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaOnErr>;  MSVBVM60.__vbaOnError
00507964   .  C745 FC 03000>MOV DWORD PTR SS:[EBP-4],3
0050796B   .  8D45 CC      LEA EAX,DWORD PTR SS:[EBP-34]
0050796E   .  50           PUSH EAX
0050796F   .  E8 FCFBFFFF  CALL GtChrdHl.00507570
00507974   .  8945 B8      MOV DWORD PTR SS:[EBP-48],EAX
00507977   .  C745 B0 08000>MOV DWORD PTR SS:[EBP-50],8
0050797E   .  8D55 B0      LEA EDX,DWORD PTR SS:[EBP-50]
00507981   .  8D4D D0      LEA ECX,DWORD PTR SS:[EBP-30]
00507984   .  FF15 14104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaVarMo>;  MSVBVM60.__vbaVarMove
0050798A   .  C745 FC 04000>MOV DWORD PTR SS:[EBP-4],4
00507991   .  C785 48FFFFFF>MOV DWORD PTR SS:[EBP-B8],0
0050799B   .  8D8D 48FFFFFF LEA ECX,DWORD PTR SS:[EBP-B8]
005079A1   .  51           PUSH ECX
005079A2   .  E8 F9ECFFFF  CALL GtChrdHl.005066A0            ;  这个函数是获取原始机器码后半部分
005079A7   .  8BD0         MOV EDX,EAX
005079A9   .  8D4D C4      LEA ECX,DWORD PTR SS:[EBP-3C]
005079AC   .  FF15 88124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrMo>;  MSVBVM60.__vbaStrMove
005079B2   .  C745 FC 05000>MOV DWORD PTR SS:[EBP-4],5
005079B9   .  8D55 CC      LEA EDX,DWORD PTR SS:[EBP-34]
005079BC   .  8995 78FFFFFF MOV DWORD PTR SS:[EBP-88],EDX
005079C2   .  C785 70FFFFFF>MOV DWORD PTR SS:[EBP-90],4008
005079CC   .  8D85 70FFFFFF LEA EAX,DWORD PTR SS:[EBP-90]
005079D2   .  50           PUSH EAX
005079D3   .  8D4D B0      LEA ECX,DWORD PTR SS:[EBP-50]
005079D6   .  51           PUSH ECX
005079D7   .  FF15 E4104000 CALL DWORD PTR DS:[<&MSVBVM60.#520>]      ;  MSVBVM60.rtcTrimVar
005079DD   .  C785 68FFFFFF>MOV DWORD PTR SS:[EBP-98],GtChrdHl.00450>;  UNICODE "March3"
005079E7   .  C785 60FFFFFF>MOV DWORD PTR SS:[EBP-A0],8
005079F1   .  8D55 C4      LEA EDX,DWORD PTR SS:[EBP-3C]
005079F4   .  8995 58FFFFFF MOV DWORD PTR SS:[EBP-A8],EDX
005079FA   .  C785 50FFFFFF>MOV DWORD PTR SS:[EBP-B0],4008
00507A04   .  8D85 50FFFFFF LEA EAX,DWORD PTR SS:[EBP-B0]
00507A0A   .  50           PUSH EAX
00507A0B   .  8D4D 90      LEA ECX,DWORD PTR SS:[EBP-70]
00507A0E   .  51           PUSH ECX
00507A0F   .  FF15 E4104000 CALL DWORD PTR DS:[<&MSVBVM60.#520>]      ;  MSVBVM60.rtcTrimVar
00507A15   .  8D55 B0      LEA EDX,DWORD PTR SS:[EBP-50]
00507A18   .  52           PUSH EDX
00507A19   .  8D85 60FFFFFF LEA EAX,DWORD PTR SS:[EBP-A0]
00507A1F   .  50           PUSH EAX
00507A20   .  8D4D A0      LEA ECX,DWORD PTR SS:[EBP-60]
00507A23   .  51           PUSH ECX
00507A24   .  FF15 CC114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaVarCa>;  MSVBVM60.__vbaVarCat
```



### 00507A2A - 00507CAB: 机器码获取与注册码计算

```
00507A2A   .  50             PUSH EAX  
00507A2B   .  8D55 90        LEA EDX,DWORD PTR SS:[EBP-70]  
00507A2E   .  52             PUSH EDX  
00507A2F   .  8D45 80        LEA EAX,DWORD PTR SS:[EBP-80]  
00507A32   .  50             PUSH EAX  
00507A33   .  FF15 CC114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaVarCa>;  MSVBVM60.__vbaVarCat  
00507A39   .  50             PUSH EAX  
00507A3A   .  FF15 20104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrVa>;  MSVBVM60.__vbaStrVarMove  
00507A40   .  8BD0           MOV EDX,EAX                         ;  EAX=0016DCDC, (UNICODE  "MRG101K1C1KD7CMarch3000000000000")  
00507A42   .  8D4D C8        LEA ECX,DWORD PTR SS:[EBP-38]  
00507A45   .  FF15 88124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrMo>;  MSVBVM60.__vbaStrMove  
00507A4B   .  8D4D 80        LEA ECX,DWORD PTR SS:[EBP-80]  
00507A4E   .  51             PUSH ECX  
00507A4F   .  8D55 90        LEA EDX,DWORD PTR SS:[EBP-70]  
00507A52   .  52             PUSH EDX  
00507A53   .  8D45 A0        LEA EAX,DWORD PTR SS:[EBP-60]  
00507A56   .  50             PUSH EAX  
00507A57   .  8D4D B0        LEA ECX,DWORD PTR SS:[EBP-50]  
00507A5A   .  51             PUSH ECX  
00507A5B   .  6A 04          PUSH 4  
00507A5D   .  FF15 30104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeV>;  MSVBVM60.__vbaFreeVarList  
00507A63   .  83C4 14        ADD ESP,14  
00507A66   .  C745 FC 06000>MOV DWORD PTR SS:[EBP-4],6  
00507A6D   .  66:C785 4CFFF>MOV WORD PTR SS:[EBP-B4],0A           ;  上面获得原始机器码  
00507A76   .  8D95 4CFFFFFF LEA EDX,DWORD PTR SS:[EBP-B4]  
00507A7C   .  52             PUSH EDX  
00507A7D   .  8D45 C8        LEA EAX,DWORD PTR SS:[EBP-38]  
00507A80   .  50             PUSH EAX  
00507A81   .  E8 FADDFFFF    CALL GtChrdHl.00505880  
00507A86   .  8BD0           MOV EDX,EAX  
00507A88   .  8D4D C0        LEA ECX,DWORD PTR SS:[EBP-40]  
00507A8B   .  FF15 88124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrMo>;  MSVBVM60.__vbaStrMove  
00507A91   .  68 EC7A5000    PUSH GtChrdHl.00507AEC                ;  获得机器码  
```

明白了。取得临时机器码后计算得出真正机器码。

```
00507C3C   .  FF15 18114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFileC>;  MSVBVM60.__vbaFileClose  
00507C42   .  E8 C9FCFFFF    CALL GtChrdHl.00507910                ;  这个是一个关键函数，应该是获取机器码的。  
00507C47   .  8BD0           MOV EDX,EAX                           ;  取得了机器码  
00507C49   .  8D4D D8        LEA ECX,DWORD PTR SS:[EBP-28]  
00507C4C   .  FFD7           CALL EDI  
00507C4E   .  8B55 D8        MOV EDX,DWORD PTR SS:[EBP-28]  
00507C51   .  8D4D E0        LEA ECX,DWORD PTR SS:[EBP-20]  
00507C54   .  C745 C0 14000>MOV DWORD PTR SS:[EBP-40],14  
00507C5B   .  C745 D8 00000>MOV DWORD PTR SS:[EBP-28],0  
00507C62   .  FFD7           CALL EDI  
00507C64   .  8B4D E4        MOV ECX,DWORD PTR SS:[EBP-1C]          ;  假码  
00507C67   .  8D55 C0        LEA EDX,DWORD PTR SS:[EBP-40]          ;  机器码  
00507C6A   .  51             PUSH ECX  
00507C6B   .  8D45 E0        LEA EAX,DWORD PTR SS:[EBP-20]  
00507C6E   .  52             PUSH EDX  
00507C6F   .  50             PUSH EAX  
00507C70   .  E8 0BDCFFFF    CALL GtChrdHl.00505880                ;  计算注册码过程，得到真正的注册码  
00507C75   .  8BD0           MOV EDX,EAX                           ;  真码，可作内存注册机  
00507C77   .  8D4D DC        LEA ECX,DWORD PTR SS:[EBP-24]  
00507C7A   .  FFD7           CALL EDI  
00507C7C   .  50             PUSH EAX  
00507C7D   .  FFD3           CALL EBX  
00507C7F   .  8BF0           MOV ESI,EAX  
00507C81   .  8D4D D8        LEA ECX,DWORD PTR SS:[EBP-28]  
00507C84   .  F7DE           NEG ESI  
00507C86   .  8D55 DC        LEA EDX,DWORD PTR SS:[EBP-24]  
00507C89   .  51             PUSH ECX  
00507C8A   .  1BF6           SBB ESI,ESI  
00507C8C   .  8D45 E0        LEA EAX,DWORD PTR SS:[EBP-20]  
00507C8F   .  52             PUSH EDX  
00507C90   .  46             INC ESI  
00507C91   .  50             PUSH EAX  
00507C92   .  6A 03          PUSH 3  
00507C94   .  F7DE           NEG ESI  
00507C96   .  FF15 1C124000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStrList  
00507C9C   .  83C4 10        ADD ESP,10  
00507C9F   .  66:85F6        TEST SI,SI  
00507CA2   .  74 07          JE SHORT GtChrdHl.00507CAB            ;  关键跳转  
```

====================================================

### 00505880: 注册码生成过程详解

下面来详细跟踪一下注册码生成过程吧：

```
00505880   $  55             PUSH EBP                              ;  机器码与注册码共用函数  
00505881   .  8BEC           MOV EBP,ESP  
00505883   .  83EC 0C        SUB ESP,0C  
00505886   .  68 06334000 push <jmp.&MSVBVM60.__vbaExceptHandler>       ;  SE  handler  installation  
0050588B   .  64:A1 00000>mov eax,dword ptr fs:[0]  
00505891   .  50             push eax  
00505892   .  64:8925 000>mov dword ptr fs:[0],esp  
00505899   .  81EC 480100>sub esp,148  
0050589F   .  53             push ebx  
005058A0   .  56             push esi  
005058A1   .  57             push edi  
005058A2   .  8965 F4       mov dword ptr ss:[ebp-C],esp  
005058A5   .  C745 F8 802>mov dword ptr ss:[ebp-8],GtChrdHl.00402F80  
005058AC   .  8B45 08       mov eax,dword ptr ss:[ebp+8]  
005058AF   .  33DB          xor ebx,ebx  
005058B1   .  895D E0       mov dword ptr ss:[ebp-20],ebx  
005058B4   .  895D DC       mov dword ptr ss:[ebp-24],ebx  
005058B7   .  8B08          mov ecx,dword ptr ds:[eax]                  ;  硬件号"w2CsM2O2Q2"  
005058B9   .  895D D8       mov dword ptr ss:[ebp-28],ebx  
005058BC   .  51             push ecx  
005058BD   .  68 18DB4400 push GtChrdHl.0044DB18  
```

省略若干初始化代码...

```
00505B8D   .  6A 04         push 4  
00505B8F   .  FFD7          call edi  
00505B91   .  83C4 14       add esp,14  
00505B94   >  E8 C7090000  call GtChrdHl.00506560  
00505B99   .  8B35 881240>mov esi,dword ptr ds:[<&MSVBVM60.__vbaStrMove>;  MSVBVM60.__vbaStrMove  
00505B9F   >  66:8B45 E0   mov ax,word ptr ss:[ebp-20]  
00505BA3   .  8B4D 0C       mov ecx,dword ptr ss:[ebp+C]  
00505BA6   .  66:05 0100    add ax,1  
00505BAA   .  66:8B19       mov bx,word ptr ds:[ecx]  
00505BAD   .  0F80 A20900>jo GtChrdHl.00506555  
00505BB3   .  66:035D E0    add bx,word ptr ss:[ebp-20]  
00505BB7   .  8985 DCFEFF>mov dword ptr ss:[ebp-124],eax  
00505BBD   .  0F80 920900>jo GtChrdHl.00506555  
00505BC3   >  66:C1F8 0F    sar ax,0F  
00505BC7   .  8BD0          mov edx,eax  
00505BC9   .  B9 00010000  mov ecx,100                               ;  100 位  
00505BCE   .  33D1          xor edx,ecx  
00505BD0   .  33C3          xor eax,ebx  
00505BD2   .  66:3BC2       cmp ax,dx                                 ;  第 15 位  
00505BD5   .  7F 66         jg short GtChrdHl.00505C3D  
00505BD7   .  0FBFC3        movsx eax,bx  
00505BDA   .  48             dec eax  
00505BDB   .  3BC1          cmp eax,ecx  
00505BDD   .  8985 E4FEFF>mov dword ptr ss:[ebp-11C],eax  
00505BE3   .  72 0C         jb short GtChrdHl.00505BF1  
00505BE5   .  FF15 201140>call dword ptr ds:[<&MSVBVM60.__vbaGenerateBo>;  MSVBVM60.__vbaGenerateBoundsError  
00505BEB   .  8B85 E4FEFF>mov eax,dword ptr ss:[ebp-11C]  
00505BF1   >  8B15 B06051>mov edx,dword ptr ds:[5160B0]              ;  长字串  
```

这里有个表，也就是长字符串了。

```
str="N  Z^%@*&(HGUINKNUIGBF%kfinkcG76U^GUI766f^%SyI@(Ifdkj(*8g7^&f65fFdIUYf6I^GjkdBUDhuipdfyig% $fjL**.x02ytPHDOIDTphHPPiofDmllfl44190$#&*JKho4o00wjdhFDK*jo:;JUIp(B[ApIuy^R&ubrYUi%6ge4uyU7OTd la;fyp0s0-2htdf!Jhfu7IK (*$$#g%(m0FG[D688g⁶fI8HDGF9[y44NT87%u9**r4EU."
```

```
00505BF7   .  8B4D D4       mov ecx,dword ptr ss:[ebp-2C]  
00505BFA   .  51             push ecx  
00505BFB   .  8A0410         mov al,byte ptr ds:[eax+edx]               ;  逐位取 ascii G->47  
00505BFE   .  50             push eax  
00505BFF   .  FF15 881140>call dword ptr ds:[<&MSVBVM60.__vbaStrUI1>]   ;  MSVBVM60.__vbaStrUI1  
00505C05   .  8BD0           mov edx,eax                               ;  变成字符串 47->71  
```



## 汇编代码片段

```
00505C07   .  8D4D C4      lea ecx,dword ptr ss:[ebp-3C]  

00505C0A   .  FFD6         call esi  

00505C0C   .  50           push eax  

00505C0D   .  FF15 641040> call dword ptr ds:[<&MSVBVM60.__vbaStrCat>]   ;  MSVBVM60.__vbaStrCat  

00505C13   .  8BD0         mov edx,eax  

00505C15   .  8D4D D4      lea ecx,dword ptr ss:[ebp-2C]  

00505C18   .  FFD6         call esi  

00505C1A   .  8D4D C4      lea ecx,dword ptr ss:[ebp-3C]  

00505C1D   .  FF15 CC1240> call dword ptr ds:[<&MSVBVM60.__vbaFreeStr>]  ;  MSVBVM60.__vbaFreeStr  

00505C23   .  66:8B8D DCF> mov cx,word ptr ss:[ebp-124]  

00505C2A   .  8B85 DCFFFF> mov eax,dword ptr ss:[ebp-124]  

00505C30   .  66:03CB      add cx,bx                                  ;  循环变量增加  

00505C33   .  0F80 1C0900> jo GtChrdHl.00506555  

00505C39   .  8BD9         mov ebx,ecx  

00505C3B   .^ EB 86        jmp short GtChrdHl.00505BC3  

00505C3D   >  66:8B55 E0   mov dx,word ptr ss:[ebp-20]  

00505C41   .  8B45 D4      mov eax,dword ptr ss:[ebp-2C]                 ;  得到转换后的长串  
```

第一个循环，从第二十位开始取 `str` 的 `ascii`，组成新的常串 `str1`

```
for i=20 to len(str)  

str1=str1 & asc(mid(str,i,1))  

next  

str1=left(str1,40)  
```

```
00505C44   .  66:83C2 01   add dx,1  

00505C48   .  50           push eax  

00505C49   .  0F80 060900> jo GtChrdHl.00506555  

00505C4F   .  8955 E0      mov dword ptr ss:[ebp-20],edx  

00505C52   .  FF15 241040> call dword ptr ds:[<&MSVBVM60.__vbaLenBstr>]  ;  MSVBVM60.__vbaLenBstr  

00505C58   .  8B4D 0C      mov ecx,dword ptr ss:[ebp+C]                   ;  len=557  

00505C5B   .  66:8B11      mov dx,word ptr ds:[ecx]                       ;  14，机器码长度  

00505C5E   .  66:6BD2 02   imul dx,dx,2                                   ;  14*2  

00505C62   .  0F80 ED0800> jo GtChrdHl.00506555  

00505C68   .  0FBFCA       movsx ecx,dx  

00505C6B   .  3BC1         cmp eax,ecx                                    ;  比较 28 跟 22d  

00505C6D   .^ 0F8C 2CFFFF> jl GtChrdHl.00505B9F  

00505C73   .  8B45 0C      mov eax,dword ptr ss:[ebp+C]  

00505C76   .  8D55 D4      lea edx,dword ptr ss:[ebp-2C]  

00505C79   .  8995 44FFFF> mov dword ptr ss:[ebp-BC],edx  

00505C7F   .  C785 3CFFFF> mov dword ptr ss:[ebp-C4],4008  

00505C89   .  66:8B08      mov cx,word ptr ds:[eax]  

00505C8C   .  8D85 3CFFFF> lea eax,dword ptr ss:[ebp-C4]  

00505C92   .  66:6BC9 02   imul cx,cx,2  

00505C96   .  0F80 B90800> jo GtChrdHl.00506555  

00505C9C   .  0FBFD1       movsx edx,cx  

00505C9F   .  52           push edx  

00505CA0   .  8D4D 9C      lea ecx,dword ptr ss:[ebp-64]  

00505CA3   .  50           push eax  

00505CA4   .  51           push ecx  

00505CA5   .  FF15 701240> call dword ptr ds:[<&MSVBVM60.#617>]          ;  MSVBVM60.rtcLeftCharVar  

00505CAB   .  8D55 9C      lea edx,dword ptr ss:[ebp-64]                  ;  left(str,40)  

00505CAE   .  52           push edx  

00505CAF   .  FF15 201040> call dword ptr ds:[<&MSVBVM60.__vbaStrVarMove>;  MSVBVM60.__vbaStrVarMove  

00505CB5   .  8BD0         mov edx,eax                                    ;  "7166703710710210511010799715554859471857"  

00505CB7   .  8D4D D4      lea ecx,dword ptr ss:[ebp-2C]  

00505CBA   .  FFD6         call esi  

00505CBC   .  8D4D 9C      lea ecx,dword ptr ss:[ebp-64]  

00505CBF   .  FF15 1C1040> call dword ptr ds:[<&MSVBVM60.__vbaFreeVar>]  ;  MSVBVM60.__vbaFreeVar  

00505CC5   .  8B45 08      mov eax,dword ptr ss:[ebp+8]  

00505CC8   .  8B08         mov ecx,dword ptr ds:[eax]  

00505CCA   .  51           push ecx                                       ;  机器码  

00505CCB   .  FF15 241040> call dword ptr ds:[<&MSVBVM60.__vbaLenBstr>]  ;  MSVBVM60.__vbaLenBstr  

00505CD1   .  8BC8         mov ecx,eax                                    ;  机器码长度  

00505CD3   .  FF15 341140> call dword ptr ds:[<&MSVBVM60.__vbaI2I4>]     ;  MSVBVM60.__vbaI2I4  

00505CD9   .  8985 D0FEFF> mov dword ptr ss:[ebp-130],eax                ;  a  

00505CDF   .  BB 01000000  mov ebx,1  

00505CE4   >  66:3B9D D0F> cmp bx,word ptr ss:[ebp-130]                  ;  设置循环  

00505CEB   .  0F8F D50000> jg GtChrdHl.00505DC6  

00505CF1   .  8B55 08      mov edx,dword ptr ss:[ebp+8]  

00505CF4   .  8D45 9C      lea eax,dword ptr ss:[ebp-64]  

00505CF7   .  0FBFCB       movsx ecx,bx  

00505CFA   .  8995 44FFFF> mov dword ptr ss:[ebp-BC],edx  

00505D00   .  50           push eax  

00505D01   .  8D95 3CFFFF> lea edx,dword ptr ss:[ebp-C4]  

00505D07   .  51           push ecx  

00505D08   .  8D45 8C      lea eax,dword ptr ss:[ebp-74]  

00505D0B   .  52           push edx  

00505D0C   .  50           push eax  

00505D0D   .  C745 A4 010> mov dword ptr ss:[ebp-5C],1  

00505D14   .  C745 9C 020> mov dword ptr ss:[ebp-64],2  

00505D1B   .  C785 3CFFFF> mov dword ptr ss:[ebp-C4],4008  

00505D25   .  FF15 0C1140> call dword ptr ds:[<&MSVBVM60.#632>]          ;  MSVBVM60.rtcMidCharVar  

00505D2B   .  8D4D 8C      lea ecx,dword ptr ss:[ebp-74]                  ;  mid(str,i,1)  

00505D2E   .  8D55 C4      lea edx,dword ptr ss:[ebp-3C]  

00505D31   .  51           push ecx  

00505D32   .  52           push edx  

00505D33   .  FF15 C41140> call dword ptr ds:[<&MSVBVM60.__vbaStrVarVal>>;  MSVBVM60.__vbaStrVarVal  

00505D39   .  50           push eax                                       ;  w  

00505D3A   .  FF15 481040> call dword ptr ds:[<&MSVBVM60.#516>]          ;  MSVBVM60.rtcAnsiValueBstr  

00505D40   .  50           push eax                                       ;  asc("w")=0x77  

00505D41   .  FF15 041040> call dword ptr ds:[<&MSVBVM60.__vbaStrI2>]    ;  MSVBVM60.__vbaStrI2  

00505D47   .  8945 84      mov dword ptr ss:[ebp-7C],eax                  ;  0x77->119  

00505D4A   .  8D85 7CFFFF> lea eax,dword ptr ss:[ebp-84]  

00505D50   .  6A 01        push 1  

00505D52   .  8D8D 6CFFFF> lea ecx,dword ptr ss:[ebp-94]  

00505D58   .  50           push eax  

00505D59   .  51           push ecx  

00505D5A   .  C785 7CFFFF> mov dword ptr ss:[ebp-84],8  

00505D64   .  FF15 9C1240> call dword ptr ds:[<&MSVBVM60.#619>]          ;  MSVBVM60.rtcRightCharVar  

00505D6A   .  8D95 6CFFFF> lea edx,dword ptr ss:[ebp-94]                  ;  right("119",1)  

00505D70   .  52           push edx  

00505D71   .  FF15 941240> call dword ptr ds:[<&MSVBVM60.__vbaI2ErrVar>] ;  MSVBVM60.__vbaI2ErrVar  

00505D77   .  66:0345 D0   add ax,word ptr ss:[ebp-30]  

00505D7B   .  8D4D C4      lea ecx,dword ptr ss:[ebp-3C]  

00505D7E   .  0F80 D10700> jo GtChrdHl.00506555  

00505D84   .  8945 D0      mov dword ptr ss:[ebp-30],eax  

00505D87   .  FF15 CC1240> call dword ptr ds:[<&MSVBVM60.__vbaFreeStr>]  ;  MSVBVM60.__vbaFreeStr  
```

上面是第二个循环，逐位取机器码的 `ascii`，并取右边的第一位，加起来为 `j`：

```
for i= 1 to len(mc)  

j= j + cint(right(cstr(asc(mid(mc,i,1))),1))  

next 
```

```
00505D8D   .  8D85 6CFFFF> lea eax,dword ptr ss:[ebp-94]  

00505D93   .  8D8D 6CFFFF> lea ecx,dword ptr ss:[ebp-94]  

00505D99   .  50           push eax  

00505D9A   .  8D95 7CFFFF> lea edx,dword ptr ss:[ebp-84]  

00505DA0   .  51           push ecx  

00505DA1   .  8D45 8C      lea eax,dword ptr ss:[ebp-74]  

00505DA4   .  52           push edx  

00505DA5   .  8D4D 9C      lea ecx,dword ptr ss:[ebp-64]  

00505DA8   .  50           push eax  

00505DA9   .  51           push ecx  

00505DAA   .  6A 05        push 5  

00505DAC   .  FFD7         call edi  

00505DAE   .  B8 01000000  mov eax,1  

00505DB3   .  83C4 18      add esp,18  

00505DB6   .  66:03C3      add ax,bx                                  ;  增加变量  

00505DB9   .  0F80 960700> jo GtChrdHl.00506555  

00505DBF   .  8BD8         mov ebx,eax  

00505DC1   .^ E9 1EFFFFFF  jmp GtChrdHl.00505CE4  

00505DC6   >  8B55 08      mov edx,dword ptr ss:[ebp+8]  

00505DC9   .  8B02         mov eax,dword ptr ds:[edx]                  ;  机器码"w2CsM2O2Q2"  

00505DCB   .  50           push eax  
```



```
00505DCC   .  FF15 241040>call dword ptr ds:[<&MSVBVM60.__vbaLenBstr>]  ;  MSVBVM60.__vbaLenBstr  

00505DD2   .  8BC8         mov ecx,eax  

00505DD4   .  FF15 341140>call dword ptr ds:[<&MSVBVM60.__vbaI2I4>]     ;  MSVBVM60.__vbaI2I4  

00505DDA   >  8BD8         mov ebx,eax  

00505DDC   .  B8 01000000 mov eax,1  

00505DE1   .  66:3BD8      cmp bx,ax  

00505DE4   .  0F8C B30000>jl GtChrdHl.00505E9D  

00505DEA   .  8B4D 08      mov ecx,dword ptr ss:[ebp+8]  

00505DED   .  8945 A4      mov dword ptr ss:[ebp-5C],eax  

00505DF0   .  0FBFC3       movsx eax,bx  

00505DF3   .  8D55 9C      lea edx,dword ptr ss:[ebp-64]  

00505DF6   .  898D 44FFFF>mov dword ptr ss:[ebp-BC],ecx  

00505DFC   .  52           push edx  

00505DFD   .  8D8D 3CFFFF>lea ecx,dword ptr ss:[ebp-C4]  

00505E03   .  50           push eax  

00505E04   .  8D55 8C      lea edx,dword ptr ss:[ebp-74]  

00505E07   .  51           push ecx  

00505E08   .  52           push edx  

00505E09   .  C745 9C 020>mov dword ptr ss:[ebp-64],2  

00505E10   .  C785 3CFFFF>mov dword ptr ss:[ebp-C4],4008                ;  逆序取一次  

00505E1A   .  FF15 0C1140>call dword ptr ds:[<&MSVBVM60.#632>]          ;  MSVBVM60.rtcMidCharVar  

00505E20   .  8B45 D8      mov eax,dword ptr ss:[ebp-28]                ;  mid("机器码",len-i,1)  

00505E23   .  8D4D 8C      lea ecx,dword ptr ss:[ebp-74]  

00505E26   .  50           push eax  

00505E27   .  8D55 C4      lea edx,dword ptr ss:[ebp-3C]  

00505E2A   .  51           push ecx  

00505E2B   .  52           push edx  

00505E2C   .  FF15 C41140>call dword ptr ds:[<&MSVBVM60.__vbaStrVarVal>>;  MSVBVM60.__vbaStrVarVal  

00505E32   .  50           push eax                                      ;  2  

00505E33   .  FF15 481040>call dword ptr ds:[<&MSVBVM60.#516>]          ;  MSVBVM60.rtcAnsiValueBstr  

00505E39   .  8BC8         mov ecx,eax                                   ;  asc("2")->32  

00505E3B   .  FF15 601040>call dword ptr ds:[<&MSVBVM60.__vbaI2Abs>]    ;  MSVBVM60.__vbaI2Abs  

00505E41   .  0FBF4D D0    movsx ecx,word ptr ss:[ebp-30]               ;  26  

00505E45   .  0FBFC0       movsx eax,ax                                  

00505E48   .  0FAFC1       imul eax,ecx                                    ;  0x32*0x26  

00505E4B   .  0F80 040700>jo GtChrdHl.00506555  

00505E51   .  50           push eax  

00505E52   .  FF15 101040>call dword ptr ds:[<&MSVBVM60.__vbaStrI4>]    ;  MSVBVM60.__vbaStrI4  

00505E58   .  8BD0         mov edx,eax                                     ;  1900  

00505E5A   .  8D4D C0      lea ecx,dword ptr ss:[ebp-40]  

00505E5D   .  FFD6         call esi  

00505E5F   .  50           push eax  

00505E60   .  FF15 641040>call dword ptr ds:[<&MSVBVM60.__vbaStrCat>]   ;  MSVBVM60.__vbaStrCat  

00505E66   .  8BD0         mov edx,eax  

00505E68   .  8D4D D8      lea ecx,dword ptr ss:[ebp-28]  

00505E6B   .  FFD6         call esi  

00505E6D   .  8D55 C0      lea edx,dword ptr ss:[ebp-40]  

00505E70   .  8D45 C4      lea eax,dword ptr ss:[ebp-3C]  

00505E73   .  52           push edx  

00505E74   .  50           push eax  

00505E75   .  6A 02        push 2  

00505E77   .  FF15 1C1240>call dword ptr ds:[<&MSVBVM60.__vbaFreeStrLis>;  MSVBVM60.__vbaFreeStrList  

00505E7D   .  8D4D 8C      lea ecx,dword ptr ss:[ebp-74]  
```

第三个循环，逆序取机器码，ascii 码乘以上面的 ascii 右边一位的和 j，然后组成新串 `str2`

```
for i=len(mc) to 1 step -1

str2=  str2 & cstr(asc(mid(mc,i,1)) *j) 

next
```

```
00505E80   .  8D55 9C      lea edx,dword ptr ss:[ebp-64]  

00505E83   .  51           push ecx  

00505E84   .  52           push edx  

00505E85   .  6A 02        push 2  

00505E87   .  FFD7         call edi  

00505E89   .  83C8 FF      or eax,FFFFFFFF  

00505E8C   .  83C4 18      add esp,18  

00505E8F   .  66:03C3      add ax,bx  

00505E92   .  0F80 BD0600>jo GtChrdHl.00506555  

00505E98   .^. E9 3DFFFFFF jmp GtChrdHl.00505DDA  

00505E9D   >  8B45 D8      mov eax,dword ptr ss:[ebp-28]                 ;  "1900307819003002190029264370254619004522"  

00505EA0   .  50           push eax  

00505EA1   .  FF15 241040>call dword ptr ds:[<&MSVBVM60.__vbaLenBstr>]  ;  MSVBVM60.__vbaLenBstr  

00505EA7   .  8B4D 0C      mov ecx,dword ptr ss:[ebp+C]  

00505EAA   .  66:8B11      mov dx,word ptr ds:[ecx]  

00505EAD   .  66:6BD2 02   imul dx,dx,2                                    ;  14*2  

00505EB1   .  0F80 9E0600>jo GtChrdHl.00506555  

00505EB7   .  0FBFCA       movsx ecx,dx  

00505EBA   .  3BC1         cmp eax,ecx  

00505EBC   .^. 0F8C 04FFFF>jl GtChrdHl.00505DC6                           ;  比较获得的串是否够 40，不够继续  

00505EC2   .  8B5D 0C      mov ebx,dword ptr ss:[ebp+C]  

00505EC5   .  8D55 D8      lea edx,dword ptr ss:[ebp-28]  

00505EC8   .  8995 44FFFF>mov dword ptr ss:[ebp-BC],edx  

00505ECE   .  8D95 3CFFFF>lea edx,dword ptr ss:[ebp-C4]  

00505ED4   .  66:8B03      mov ax,word ptr ds:[ebx]  

00505ED7   .  C785 3CFFFF>mov dword ptr ss:[ebp-C4],4008  

00505EE1   .  66:6BC0 02   imul ax,ax,2  

00505EE5   .  0F80 6A0600>jo GtChrdHl.00506555  

00505EEB   .  0FBFC8       movsx ecx,ax  

00505EEE   .  51           push ecx  

00505EEF   .  8D45 9C      lea eax,dword ptr ss:[ebp-64]  

00505EF2   .  52           push edx  

00505EF3   .  50           push eax                                        ;  left(str2,40)  

00505EF4   .  FF15 701240>call dword ptr ds:[<&MSVBVM60.#617>]           ;  MSVBVM60.rtcLeftCharVar
```

取串 `str2` 的左边 40 位：

```
str2=left(str2,40)
```

```
00505EFA   .  8D4D 9C      lea ecx,dword ptr ss:[ebp-64]  

00505EFD   .  51           push ecx  

00505EFE   .  FF15 201040>call dword ptr ds:[<&MSVBVM60.__vbaStrVarMove>;  MSVBVM60.__vbaStrVarMove  

00505F04   .  8BD0         mov edx,eax                                     ;  UNICODE "1900307819003002190029264370254619004522"  

00505F06   .  8D4D D8      lea ecx,dword ptr ss:[ebp-28]  

00505F09   .  FFD6         call esi  

00505F0B   .  8D4D 9C      lea ecx,dword ptr ss:[ebp-64]  

00505F0E   .  FF15 1C1040>call dword ptr ds:[<&MSVBVM60.__vbaFreeVar>]   ;  MSVBVM60.__vbaFreeVar  

00505F14   .  66:8B0B      mov cx,word ptr ds:[ebx]  

00505F17   .  B8 01000000 mov eax,1  

00505F1C   .  66:6BC9 02   imul cx,cx,2  

00505F20   .  0F80 2F0600>jo GtChrdHl.00506555  

00505F26   .  898D C0FEFF>mov dword ptr ss:[ebp-140],ecx  

00505F2C   .  8945 E4      mov dword ptr ss:[ebp-1C],eax  

00505F2F   >  66:3BC1      cmp ax,cx                                       ;  循环，cx＝40  

00505F32   .  0F8F 360100>jg GtChrdHl.0050606E  

00505F38   .  8B55 CC      mov edx,dword ptr ss:[ebp-34]  

00505F3B   .  B9 02000000 mov ecx,2  

00505F40   .  8995 04FFFF>mov dword ptr ss:[ebp-FC],edx  

00505F46   .  894D A4      mov dword ptr ss:[ebp-5C],ecx  

00505F49   .  894D 9C      mov dword ptr ss:[ebp-64],ecx  

00505F4C   .  8D4D D8      lea ecx,dword ptr ss:[ebp-28]  

00505F4F   .  0FBFD8       movsx ebx,ax  

00505F52   .  8D55 9C      lea edx,dword ptr ss:[ebp-64]  

00505F55   .  898D 44FFFF>mov dword ptr ss:[ebp-BC],ecx  

00505F5B   .  52           push edx  

00505F5C   .  8D85 3CFFFF>lea eax,dword ptr ss:[ebp-C4]  

00505F62   .  53           push ebx  

00505F63   .  8D4D 8C      lea ecx,dword ptr ss:[ebp-74]  

00505F66   .  50           push eax  

00505F67   .  51           push ecx  

00505F68   .  C785 FCFEFF>mov dword ptr ss:[ebp-104],8  

00505F72   .  C785 3CFFFF>mov dword ptr ss:[ebp-C4],4008  

00505F7C   .  FF15 0C1140>call dword ptr ds:[<&MSVBVM60.#632>]           ;  MSVBVM60.rtcMidCharVar  

00505F82   .  B8 02000000 mov eax,2                                       ;  mid(str2,i,2)  

00505F87   .  8D55 D4      lea edx,dword ptr ss:[ebp-2C]  

00505F8A   .  8945 84      mov dword ptr ss:[ebp-7C],eax  

00505F8D   .  8985 7CFFFF>mov dword ptr ss:[ebp-84],eax  

00505F93   .  8D85 7CFFFF>lea eax,dword ptr ss:[ebp-84]  

00505F99   .  8995 24FFFF>mov dword ptr ss:[ebp-DC],edx  
```



```
00505F9F   .  50             push  eax  

00505FA0   .  8D8D 1CFFFF> lea  ecx,dword  ptr  ss:\[ebp\-E4\]  

00505FA6   .  53             push  ebx  

00505FA7   .  8D95 6CFFFF> lea  edx,dword  ptr  ss:\[ebp\-94\]  

00505FAD   .  51             push  ecx  

00505FAE   .  52             push  edx  

00505FAF   .  C785 1CFFFF> mov  dword  ptr  ss:\[ebp\-E4\],4008               ;  mid(str1,i,2)  

00505FB9   .  FF15 0C1140> call  dword  ptr  ds:\[<&MSVBVM60.#632>\]        ;  MSVBVM60.rtcMidCharVar  

00505FBF   .  8B1D C41240> mov  ebx,dword  ptr  ds:\[<&MSVBVM60.\_\_vbaI4ErrVa>;  MSVBVM60.\_\_vbaI4ErrVar  

00505FC5   .  8D85 6CFFFF> lea  eax,dword  ptr  ss:\[ebp\-94\]  

00505FCB   .  50             push  eax  

00505FCC   .  FFD3          call  ebx                                        ;  <&MSVBVM60.\_\_vbaI4ErrVar>  

00505FCE   .  8D4D 8C       lea  ecx,dword  ptr  ss:\[ebp\-74\]  

00505FD1   .  8BD0          mov  edx,eax  

00505FD3   .  51             push  ecx  

00505FD4   .  8995 ACFEFF> mov  dword  ptr  ss:\[ebp\-154\],edx  

00505FDA   .  FFD3          call  ebx  

00505FDC   .  8B95 ACFEFF> mov  edx,dword  ptr  ss:\[ebp\-154\]             ;  0x42  

00505FE2   .  33D0          xor  edx,eax                                    ;  mid1 xor mid2  

00505FE4   .  8D85 5CFFFF> lea  eax,dword  ptr  ss:\[ebp\-A4\]  

00505FEA   .  52             push  edx  

00505FEB   .  50             push  eax                                       ;  0x54->chr(84)  

00505FEC   .  FF15 BC1140> call  dword  ptr  ds:\[<&MSVBVM60.#608>\]        ;  MSVBVM60.rtcVarBstrFromAnsi  

00505FF2   .  8D8D FCFEFF> lea  ecx,dword  ptr  ss:\[ebp\-104\]  

00505FF8   .  8D95 5CFFFF> lea  edx,dword  ptr  ss:\[ebp\-A4\]  

00505FFE   .  51             push  ecx  

00505FFF   .  8D85 4CFFFF> lea  eax,dword  ptr  ss:\[ebp\-B4\]  

00506005   .  52             push  edx  

00506006   .  50             push  eax  

00506007   .  FF15 CC1140> call  dword  ptr  ds:\[<&MSVBVM60.\_\_vbaVarCat>\];  MSVBVM60.\_\_vbaVarCat  

0050600D   .  50             push  eax                                       ;  T 连接，得到 str3  

0050600E   .  FF15 201040> call  dword  ptr  ds:\[<&MSVBVM60.\_\_vbaStrVarMove>;  MSVBVM60.\_\_vbaStrVarMove  

00506014   .  8BD0          mov  edx,eax  

00506016   .  8D4D CC       lea  ecx,dword  ptr  ss:\[ebp\-34\]  

00506019   .  FFD6          call  esi  

0050601B   .  8D8D 4CFFFF> lea  ecx,dword  ptr  ss:\[ebp\-B4\]  

00506021   .  8D95 5CFFFF> lea  edx,dword  ptr  ss:\[ebp\-A4\]  

00506027   .  51             push  ecx  

00506028   .  8D85 6CFFFF> lea  eax,dword  ptr  ss:\[ebp\-94\]  

0050602E   .  52             push  edx  

0050602F   .  8D8D 6CFFFF> lea  ecx,dword  ptr  ss:\[ebp\-94\]  

00506035   .  50             push  eax  

00506036   .  8D95 7CFFFF> lea  edx,dword  ptr  ss:\[ebp\-84\]  

0050603C   .  51             push  ecx  

0050603D   .  8D45 8C       lea  eax,dword  ptr  ss:\[ebp\-74\]  

00506040   .  52             push  edx  

00506041   .  8D4D 8C       lea  ecx,dword  ptr  ss:\[ebp\-74\]  

00506044   .  50             push  eax  

00506045   .  8D55 9C       lea  edx,dword  ptr  ss:\[ebp\-64\]  

00506048   .  51             push  ecx  

00506049   .  52             push  edx  

0050604A   .  6A 08          push  8  

0050604C   .  FFD7          call  edi  

0050604E   .  B8 02000000   mov  eax,2                                      ;  i=i+2  

00506053   .  83C4 24       add  esp,24  

00506056   .  66:0345 E4    add  ax,word  ptr  ss:\[ebp\-1C\]  

0050605A   .  0F80 F50400> jo  GtChrdHl.00506555  

00506060   .  8B8D C0FEFF> mov  ecx,dword  ptr  ss:\[ebp\-140\]  

00506066   .  8945 E4       mov  dword  ptr  ss:\[ebp\-1C\],eax  

00506069   .^  E9 C1FEFFFF  jmp  GtChrdHl.00505F2F  

0050606E   >  8B45 0C       mov  eax,dword  ptr  ss:\[ebp+C\]  

00506071   .  66:8B08      mov  cx,word  ptr  ds:\[eax\]  

00506074   .  B8 01000000   mov  eax,1  

00506079   .  66:898D B8F> mov  word  ptr  ss:\[ebp\-148\],cx  

00506080   >  66:3B85 B8F> cmp  ax,word  ptr  ss:\[ebp\-148\]             ;  14  

00506087   .  8945 E4       mov  dword  ptr  ss:\[ebp\-1C\],eax  

0050608A   .  0F8F 160400> jg  GtChrdHl.005064A6  
```

第四个循环，从第一位开始，逐两位取 `str1` 的 ascii 跟 `str2` 的 ascii 码异或，组成新串 `str3`

```
for i=1 to 40 step 2

str3=str3 & chr(cint(mid(str1,i,2)) xor cint(mid(str2,i,2)))

next
```

```
00506090   .  8D55 CC      lea  edx,dword  ptr  ss:\[ebp\-34\]  

00506093   .  8D4D 9C      lea  ecx,dword  ptr  ss:\[ebp\-64\]  

00506096   .  8995 44FFFF> mov  dword  ptr  ss:\[ebp\-BC\],edx  

0050609C   .  51             push  ecx  

0050609D   .  0FBFD0      movsx edx,ax  

005060A0   .  8D85 3CFFFF> lea  eax,dword  ptr  ss:\[ebp\-C4\]  

005060A6   .  52             push  edx  

005060A7   .  8D4D 8C      lea  ecx,dword  ptr  ss:\[ebp\-74\]  

005060AA   .  50             push  eax  

005060AB   .  51             push  ecx  

005060AC   .  C745 A4 010> mov  dword  ptr  ss:\[ebp\-5C\],1  

005060B3   .  C745 9C 020> mov  dword  ptr  ss:\[ebp\-64\],2  

005060BA   .  C785 3CFFFF> mov  dword  ptr  ss:\[ebp\-C4\],4008          ;  mid(str3,i,1)  

005060C4   .  FF15 0C1140> call  dword  ptr  ds:\[<&MSVBVM60.#632>\]     ;  MSVBVM60.rtcMidCharVar  

005060CA   .  8D55 8C      lea  edx,dword  ptr  ss:\[ebp\-74\]  

005060CD   .  8D45 C4      lea  eax,dword  ptr  ss:\[ebp\-3C\]  

005060D0   .  52             push  edx  

005060D1   .  50             push  eax  

005060D2   .  FF15 C41140> call  dword  ptr  ds:\[<&MSVBVM60.\_\_vbaStrVarVal>>;  MSVBVM60.\_\_vbaStrVarVal  

005060D8   .  50             push  eax  

005060D9   .  FF15 481040> call  dword  ptr  ds:\[<&MSVBVM60.#516>\]     ;  MSVBVM60.rtcAnsiValueBstr  

005060DF   .  8BC8          mov  ecx,eax                                  ;  asc()  

005060E1   .  FF15 741140> call  dword  ptr  ds:\[<&MSVBVM60.\_\_vbaUI1I2>\]    ;  MSVBVM60.\_\_vbaUI1I2  

005060E7   .  8D4D C4      lea  ecx,dword  ptr  ss:\[ebp\-3C\]  

005060EA   .  8AD8          mov  bl,al  

005060EC   .  FF15 CC1240> call  dword  ptr  ds:\[<&MSVBVM60.\_\_vbaFreeStr>\]  ;  MSVBVM60.\_\_vbaFreeStr  

005060F2   .  8D4D 8C      lea  ecx,dword  ptr  ss:\[ebp\-74\]  

005060F5   .  8D55 9C      lea  edx,dword  ptr  ss:\[ebp\-64\]  

005060F8   .  51             push  ecx  

005060F9   .  52             push  edx  

005060FA   .  6A 02          push  2  

005060FC   .  FFD7          call  edi  

005060FE   .  83C4 0C      add  esp,0C  

00506101   .  80FB 19      cmp  bl,19                                    ;  >0x19  

00506104   .  77 21         ja  short  GtChrdHl.00506127  

00506106   .  8B45 DC      mov  eax,dword  ptr  ss:\[ebp\-24\]  

00506109   .  66:33C9      xor  cx,cx  

0050610C   .  8ACB          mov  cl,bl  

0050610E   .  8985 44FFFF> mov  dword  ptr  ss:\[ebp\-BC\],eax  

00506114   .  C785 3CFFFF> mov  dword  ptr  ss:\[ebp\-C4\],8  

0050611E   .  66:83C1 41   add  cx,41                                    ;  +41  

00506122   .  E9 95000000   jmp  GtChrdHl.005061BC  

00506127   >  80FB 33      cmp  bl,33                                    ;  >0x33 3  

0050612A   .  77 1E         ja  short  GtChrdHl.0050614A  

0050612C   .  8B45 DC      mov  eax,dword  ptr  ss:\[ebp\-24\]  

0050612F   .  66:33C9      xor  cx,cx  

00506132   .  8ACB          mov  cl,bl  

00506134   .  8985 44FFFF> mov  dword  ptr  ss:\[ebp\-BC\],eax  

0050613A   .  C785 3CFFFF> mov  dword  ptr  ss:\[ebp\-C4\],8  

00506144   .  66:83C1 47   add  cx,47                                    ;  +47  

00506148   .  EB 72         jmp  short  GtChrdHl.005061BC  

0050614A   >  80FB 3D      cmp  bl,3D                                    ;  >0x3d =  

0050614D   .  77 1E         ja  short  GtChrdHl.0050616D  

0050614F   .  8B45 DC      mov  eax,dword  ptr  ss:\[ebp\-24\]  

00506152   .  66:33C9      xor  cx,cx  

00506155   .  8ACB          mov  cl,bl  

00506157   .  8985 44FFFF> mov  dword  ptr  ss:\[ebp\-BC\],eax  

0050615D   .  C785 3CFFFF> mov  dword  ptr  ss:\[ebp\-C4\],8  

00506167   .  66:83E9 04   sub  cx,4                                     ;  -4  

0050616B   .  EB 4F         jmp  short  GtChrdHl.005061BC  

0050616D   >  80FB 47      cmp  bl,47                                    ;  >0x47  

00506170   .  77 1E         ja  short  GtChrdHl.00506190  

00506172   .  8B45 DC      mov  eax,dword  ptr  ss:\[ebp\-24\]  

00506175   .  66:33C9      xor  cx,cx  

00506178   .  8ACB          mov  cl,bl  
```



```
0050617A   .  8985 44FFFF>mov dword ptr ss:[ebp-BC],eax  
00506180   .  C785 3CFFFF>mov dword ptr ss:[ebp-C4],8  

0050618A   .  66:83E9 0E  sub cx,0E                               ;  -0xe  

0050618E   .  EB 2C        jmp short GtChrdHl.005061BC  

00506190   >  8B45 DC      mov eax,dword ptr ss:[ebp-24]  

00506193   .  80FB 60      cmp bl,60                               ;  >0x60  

00506196   .  8985 44FFFF>mov dword ptr ss:[ebp-BC],eax  

0050619C   .  C785 3CFFFF>mov dword ptr ss:[ebp-C4],8  

005061A6   .  77 0B        ja short GtChrdHl.005061B3  

005061A8   .  66:33C9      xor cx,cx  

005061AB   .  8ACB         mov cl,bl  

005061AD   .  66:83E9 07  sub cx,7                                  ;  cx-7  

005061B1   .  EB 09        jmp short GtChrdHl.005061BC  

005061B3   >  66:33C9      xor cx,cx  

005061B6   .  8ACB         mov cl,bl  

005061B8   .  66:83C1 0A  add cx,0A                                 ;  +0xa  

005061BC   >  0F80 930300>jo GtChrdHl.00506555  

005061C2   .  0FBFD1       movsx edx,cx  

005061C5   .  8D45 9C      lea eax,dword ptr ss:[ebp-64]  

005061C8   .  52           push edx  

005061C9   .  50           push eax  

005061CA   .  FF15 BC1140>call dword ptr ds:[<&MSVBVM60.#608>]        ;  MSVBVM60.rtcVarBstrFromAnsi  

005061D0   .  8D8D 3CFFFF>lea ecx,dword ptr ss:[ebp-C4]               ;  chr(0x4d)  

005061D6   .  8D55 9C      lea edx,dword ptr ss:[ebp-64]  

005061D9   .  51           push ecx  

005061DA   .  8D45 8C      lea eax,dword ptr ss:[ebp-74]  

005061DD   .  52           push edx  

005061DE   .  50           push eax  

005061DF   .  FF15 CC1140>call dword ptr ds:[<&MSVBVM60.__vbaVarCat>]  ;  MSVBVM60.__vbaVarCat  

005061E5   .  50           push eax                                  ;  0x4d  

005061E6   .  FF15 201040>call dword ptr ds:[<&MSVBVM60.__vbaStrVarMove>;  MSVBVM60.__vbaStrVarMove  

005061EC   .  8BD0         mov edx,eax  

005061EE   .  8D4D DC      lea ecx,dword ptr ss:[ebp-24]  
```

对 `str3` 逐位判断，

```
For i = 1 To 40  
    tmp = Asc(mid(str3, i, 1))  

    If tmp <= &H19 Then  
        tmp = tmp  + &H41  

    ElseIf tmp <= &H33 Then  
        tmp = tmp  + &H47  

    ElseIf tmp <= &H3D Then  
        tmp = tmp  - &H4  

    ElseIf tmp <= &H47 Then  
        tmp = tmp  - &HE  

    ElseIf tmp <= &H60 Then  
    tmp = tmp  - &H7  

    Else  
        tmp = tmp  + &HA  

    End If  
```

```
005061F1   .  FFD6         call esi  

005061F3   .  8D4D 8C      lea ecx,dword ptr ss:[ebp-74]  

005061F6   .  8D55 9C      lea edx,dword ptr ss:[ebp-64]  

005061F9   .  51           push ecx  

005061FA   .  52           push edx  

005061FB   .  6A 02        push 2  

005061FD   .  FFD7         call edi  

005061FF   .  8B3D 9C1240>mov edi,dword ptr ds:[<&MSVBVM60.#619>]     ;  MSVBVM60.rtcRightCharVar  

00506205   .  83C4 0C      add esp,0C  

00506208   .  8D8D 3CFFFF>lea ecx,dword ptr ss:[ebp-C4]  

0050620E   .  8D55 9C      lea edx,dword ptr ss:[ebp-64]  

00506211   .  6A 01        push 1  

00506213   .  8D45 DC      lea eax,dword ptr ss:[ebp-24]  

00506216   .  BB 08400000  mov ebx,4008  

0050621B   .  51           push ecx  

0050621C   .  52           push edx  

0050621D   .  8985 44FFFF>mov dword ptr ss:[ebp-BC],eax  

00506223   .  899D 3CFFFF>mov dword ptr ss:[ebp-C4],ebx  

00506229   .  FFD7         call edi                                  ;  <&MSVBVM60.#619>  

0050622B   .  8D8D 2CFFFF>lea ecx,dword ptr ss:[ebp-D4]  

00506231   .  6A 01        push 1  

00506233   .  8D55 8C      lea edx,dword ptr ss:[ebp-74]  

00506236   .  8D45 DC      lea eax,dword ptr ss:[ebp-24]  

00506239   .  51           push ecx  

0050623A   .  52           push edx  

0050623B   .  8985 34FFFF>mov dword ptr ss:[ebp-CC],eax  

00506241   .  899D 2CFFFF>mov dword ptr ss:[ebp-D4],ebx  

00506247   .  FFD7         call edi  

00506249   .  8D8D 1CFFFF>lea ecx,dword ptr ss:[ebp-E4]  

0050624F   .  6A 01        push 1  

00506251   .  8D95 7CFFFF>lea edx,dword ptr ss:[ebp-84]  

00506257   .  8D45 DC      lea eax,dword ptr ss:[ebp-24]  

0050625A   .  51           push ecx  

0050625B   .  52           push edx  

0050625C   .  8985 24FFFF>mov dword ptr ss:[ebp-DC],eax  

00506262   .  899D 1CFFFF>mov dword ptr ss:[ebp-E4],ebx  

00506268   .  FFD7         call edi  

0050626A   .  8D8D 0CFFFF>lea ecx,dword ptr ss:[ebp-F4]  

00506270   .  6A 01        push 1  

00506272   .  8D95 6CFFFF>lea edx,dword ptr ss:[ebp-94]  

00506278   .  8D45 DC      lea eax,dword ptr ss:[ebp-24]  

0050627B   .  51           push ecx  

0050627C   .  52           push edx  

0050627D   .  8985 14FFFF>mov dword ptr ss:[ebp-EC],eax  

00506283   .  899D 0CFFFF>mov dword ptr ss:[ebp-F4],ebx  

00506289   .  FFD7         call edi  

0050628B   .  8D8D FCFEFF>lea ecx,dword ptr ss:[ebp-104]  

00506291   .  6A 01        push 1  

00506293   .  8D95 5CFFFF>lea edx,dword ptr ss:[ebp-A4]  

00506299   .  8D45 DC      lea eax,dword ptr ss:[ebp-24]  

0050629C   .  51           push ecx  

0050629D   .  52           push edx  

0050629E   .  8985 04FFFF>mov dword ptr ss:[ebp-FC],eax  

005062A4   .  899D FCFEFF>mov dword ptr ss:[ebp-104],ebx  

005062AA   .  FFD7         call edi  

005062AC   .  8D8D ECFEFF>lea ecx,dword ptr ss:[ebp-114]  

005062B2   .  6A 01        push 1  

005062B4   .  8D95 4CFFFF>lea edx,dword ptr ss:[ebp-B4]  

005062BA   .  8D45 DC      lea eax,dword ptr ss:[ebp-24]  

005062BD   .  51           push ecx  

005062BE   .  52           push edx  

005062BF   .  8985 F4FEFF>mov dword ptr ss:[ebp-10C],eax  

005062C5   .  899D ECFEFF>mov dword ptr ss:[ebp-114],ebx  

005062CB   .  FFD7         call edi  

005062CD   .  8B3D C41140>mov edi,dword ptr ds:[<&MSVBVM60.__vbaStrVarV>;  MSVBVM60.__vbaStrVarVal  

005062D3   .  8D85 5CFFFF>lea eax,dword ptr ss:[ebp-A4]  

005062D9   .  8D4D B4      lea ecx,dword ptr ss:[ebp-4C]  

005062DC   .  50           push eax  

005062DD   .  51           push ecx  

005062DE   .  FFD7         call edi                                  ;  <&MSVBVM60.__vbaStrVarVal>  

005062E0   .  50           push eax  

005062E1   .  FF15 481040>call dword ptr ds:[<&MSVBVM60.#516>]       ;  MSVBVM60.rtcAnsiValueBstr  

005062E7   .  33DB         xor ebx,ebx  

005062E9   .  66:3D 6100   cmp ax,61                                 ;  0x4d >60? `  

005062ED   .  8D95 6CFFFF>lea edx,dword ptr ss:[ebp-94]  

005062F3   .  8D45 B8      lea eax,dword ptr ss:[ebp-48]  

005062F6   .  0F9CC3       setl bl  

005062F9   .  52           push edx  

005062FA   .  50           push eax  

005062FB   .  F7DB         neg ebx  

005062FD   .  FFD7         call edi  

005062FF   .  50           push eax  

00506300   .  FF15 481040>call dword ptr ds:[<&MSVBVM60.#516>]       ;  MSVBVM60.rtcAnsiValueBstr  

00506306   .  33C9         xor ecx,ecx  

00506308   .  66:3D 5A00   cmp ax,5A                                 ;  >5a Z  

0050630C   .  0F9FC1       setg cl  

0050630F   .  8D95 7CFFFF>lea edx,dword ptr ss:[ebp-84]  

00506315   .  8D45 BC      lea eax,dword ptr ss:[ebp-44]  

00506318   .  F7D9         neg ecx  

0050631A   .  52           push edx  

0050631B   .  50           push eax  

0050631C   .  23D9         and ebx,ecx  

0050631E   .  FFD7         call edi  

00506320   .  50           push eax  

00506321   .  FF15 481040>call dword ptr ds:[<&MSVBVM60.#516>]       ;  MSVBVM60.rtcAnsiValueBstr  

00506327   .  899D A8FEFF>mov dword ptr ss:[ebp-158],ebx  

0050632D   .  33DB         xor ebx,ebx  

0050632F   .  66:3D 4100   cmp ax,41                                 ;  >41 A  

00506333   .  8D4D 8C      lea ecx,dword ptr ss:[ebp-74]  

00506336   .  8D55 C0      lea edx,dword ptr ss:[ebp-40]  

00506339   .  51           push ecx  

0050633A   .  0F9CC3       setl bl  

0050633D   .  52           push edx  

0050633E   .  F7DB         neg ebx  
```



```
00506340   .  FFD7        call edi  

00506342   .  50          push eax  

00506343   .  FF15 481040>call dword ptr ds:[<&MSVBVM60.#516>]           ;  MSVBVM60.rtcAnsiValueBstr  

00506349   .  8B95 A8FEFF>mov edx,dword ptr ss:[ebp-158]  

0050634F   .  33C9        xor ecx,ecx  

00506351   .  66:3D 3900  cmp ax,39                                     ;  >39 9  

00506355   .  8D85 4CFFFF>lea eax,dword ptr ss:[ebp-B4]  

0050635B   .  0F9FC1      setg cl  

0050635E   .  F7D9        neg ecx  

00506360   .  23D9        and ebx,ecx  

00506362   .  8D4D B0     lea ecx,dword ptr ss:[ebp-50]  

00506365   .  0BD3        or edx,ebx                                     ;  or  

00506367   .  50          push eax  

00506368   .  51          push ecx  

00506369   .  8BDA        mov ebx,edx  

0050636B   .  FFD7        call edi  

0050636D   .  8B3D 481040>mov edi,dword ptr ds:[<&MSVBVM60.#516>]       ;  MSVBVM60.rtcAnsiValueBstr  

00506373   .  50          push eax  

00506374   .  FFD7        call edi                                        ;  <&MSVBVM60.#516>  

00506376   .  33D2        xor edx,edx  

00506378   .  66:3D 7A00  cmp ax,7A                                       ;  z  

0050637C   .  8D45 9C     lea eax,dword ptr ss:[ebp-64]  

0050637F   .  0F9FC2      setg dl  

00506382   .  8D4D C4     lea ecx,dword ptr ss:[ebp-3C]  

00506385   .  50          push eax  

00506386   .  F7DA        neg edx  

00506388   .  51          push ecx  

00506389   .  0BDA        or ebx,edx  

0050638B   .  FF15 C41140>call dword ptr ds:[<&MSVBVM60.__vbaStrVarVal>>;  MSVBVM60.__vbaStrVarVal  

00506391   .  50          push eax  

00506392   .  FFD7        call edi  

00506394   .  33D2        xor edx,edx  

00506396   .  66:3D 3000  cmp ax,30                                       ;  0  

0050639A   .  0F9CC2      setl dl  

0050639D   .  8D45 B0     lea eax,dword ptr ss:[ebp-50]  

005063A0   .  8D4D B4     lea ecx,dword ptr ss:[ebp-4C]  

005063A3   .  F7DA        neg edx  

005063A5   .  50          push eax  

005063A6   .  0BDA        or ebx,edx  

005063A8   .  51          push ecx  

005063A9   .  8D55 B8     lea edx,dword ptr ss:[ebp-48]  

005063AC   .  8D45 BC     lea eax,dword ptr ss:[ebp-44]  

005063AF   .  52          push edx  

005063B0   .  8D4D C0     lea ecx,dword ptr ss:[ebp-40]  

005063B3   .  50          push eax  

005063B4   .  8D55 C4     lea edx,dword ptr ss:[ebp-3C]  

005063B7   .  51          push ecx  

005063B8   .  52          push edx  

005063B9   .  6A 06       push 6  

005063BB   .  FF15 1C1240>call dword ptr ds:[<&MSVBVM60.__vbaFreeStrLis>;  MSVBVM60.__vbaFreeStrList  

005063C1   .  8D85 4CFFFF>lea eax,dword ptr ss:[ebp-B4]  

005063C7   .  8B3D 301040>mov edi,dword ptr ds:[<&MSVBVM60.__vbaFreeVar>;  MSVBVM60.__vbaFreeVarList  

005063CD   .  8D8D 5CFFFF>lea ecx,dword ptr ss:[ebp-A4]  

005063D3   .  50          push eax  

005063D4   .  8D95 6CFFFF>lea edx,dword ptr ss:[ebp-94]  

005063DA   .  51          push ecx  
```

判断特殊情况，如果出现特殊字符，用“2”代替

```
if tmp<&H30 or (tmp>&H39 and tmp<&H41) or (tmp>&H5a and tmp<&H60) or tmp>&H7a then tmp= &H32  
```

```
005063DB   .  8D85 7CFFFF>lea eax,dword ptr ss:[ebp-84]  

005063E1   .  52          push edx  

005063E2   .  8D4D 8C     lea ecx,dword ptr ss:[ebp-74]  

005063E5   .  50          push eax  

005063E6   .  8D55 9C     lea edx,dword ptr ss:[ebp-64]  

005063E9   .  51          push ecx  

005063EA   .  52          push edx  

005063EB   .  6A 06       push 6  

005063ED   .  FFD7        call edi                                        ;  <&MSVBVM60.__vbaFreeVarList>  

005063EF   .  83C4 38     add esp,38  

005063F2   .  66:85DB     test bx,bx  

005063F5   .  0F84 970000>je GtChrdHl.00506492                             ;  满足上面的条件 需要测试  

005063FB   .  8B45 DC     mov eax,dword ptr ss:[ebp-24]  

005063FE   .  50          push eax  

005063FF   .  FF15 241040>call dword ptr ds:[<&MSVBVM60.__vbaLenBstr>]  ;  MSVBVM60.__vbaLenBstr  

00506405   .  83E8 01     sub eax,1                                       ;  len()-1  

00506408   .  8D4D DC     lea ecx,dword ptr ss:[ebp-24]  

0050640B   .  0F80 440100>jo GtChrdHl.00506555  

00506411   .  8D55 9C     lea edx,dword ptr ss:[ebp-64]  

00506414   .  8945 A4     mov dword ptr ss:[ebp-5C],eax  

00506417   .  898D 44FFFF>mov dword ptr ss:[ebp-BC],ecx  

0050641D   .  52          push edx  

0050641E   .  8D85 3CFFFF>lea eax,dword ptr ss:[ebp-C4]  

00506424   .  6A 01       push 1  

00506426   .  8D4D 8C     lea ecx,dword ptr ss:[ebp-74]  

00506429   .  50          push eax  

0050642A   .  51          push ecx  

0050642B   .  C745 9C 030>mov dword ptr ss:[ebp-64],3  

00506432   .  C785 3CFFFF>mov dword ptr ss:[ebp-C4],4008  

0050643C   .  FF15 0C1140>call dword ptr ds:[<&MSVBVM60.#632>]           ;  MSVBVM60.rtcMidCharVar  

00506442   .  8D55 8C     lea edx,dword ptr ss:[ebp-74]  

00506445   .  8D85 1CFFFF>lea eax,dword ptr ss:[ebp-E4]  

0050644B   .  52          push edx  

0050644C   .  8D8D 7CFFFF>lea ecx,dword ptr ss:[ebp-84]  

00506452   .  50          push eax  

00506453   .  51          push ecx  

00506454   .  C785 24FFFF>mov dword ptr ss:[ebp-DC],GtChrdHl.0044DEB8   ;  2  

0050645E   .  C785 1CFFFF>mov dword ptr ss:[ebp-E4],8  

00506468   .  FF15 CC1140>call dword ptr ds:[<&MSVBVM60.__vbaVarCat>]   ;  MSVBVM60.__vbaVarCat  

0050646E   .  50          push eax                                        ;  否则加 2  

0050646F   .  FF15 201040>call dword ptr ds:[<&MSVBVM60.__vbaStrVarMove>;  MSVBVM60.__vbaStrVarMove  

00506475   .  8BD0        mov edx,eax  

00506477   .  8D4D DC     lea ecx,dword ptr ss:[ebp-24]  

0050647A   .  FFD6        call esi  

0050647C   .  8D95 7CFFFF>lea edx,dword ptr ss:[ebp-84]  

00506482   .  8D45 8C     lea eax,dword ptr ss:[ebp-74]  

00506485   .  52          push edx  

00506486   .  8D4D 9C     lea ecx,dword ptr ss:[ebp-64]  

00506489   .  50          push eax  

0050648A   .  51          push ecx  

0050648B   .  6A 03       push 3  

0050648D   .  FFD7        call edi  

0050648F   .  83C4 10     add esp,10  

00506492   >  B8 01000000 mov eax,1                                       ;  循环量加 1  

00506497   .  66:0345 E4  add ax,word ptr ss:[ebp-1C]  

0050649B   .  0F80 B40000>jo GtChrdHl.00506555  

005064A1   .^ E9 DAFBFFFF jmp GtChrdHl.00506080  

005064A6   >  8B55 DC     mov edx,dword ptr ss:[ebp-24]                 ;  "M4QuZ9cIgKXNCBueov1v"  

005064A9   .  8D4D C8     lea ecx,dword ptr ss:[ebp-38]  

005064AC   .  FF15 101240>call dword ptr ds:[<&MSVBVM60.__vbaStrCopy>]  ;  MSVBVM60.__vbaStrCopy  

005064B2   .  68 3F655000 push GtChrdHl.0050653F  

005064B7   .  EB 6B       jmp short GtChrdHl.00506524  

005064B9   .  F645 FC 04  test byte ptr ss:[ebp-4],4  

005064BD   .  74 09       je short GtChrdHl.005064C8 
```

`DFCG`

得到最终的注册码。

`sn=sn & chr(tmp)`。

算法总结：

```vb
'初始化第一个串
str="N  Z^%@*&(HGUINKNUIGBF%kfinkcG76U^GUI766f^%SyI@(Ifdkj(*8g7^&f65fFdIUYf6I^GjkdBUDhuipdfyig% $fjL**.x02ytPHDOIDTphHPPiofDmllfl44190$#&*JKho4o00wjdhFDK*jo:;JUIp(B[ApIuy^R&ubrYUi%6ge4uyU7OTd la;fyp0s0-2htdf!Jhfu7IK (*$$#g%(m0FG[D688g⁶fI8HDGF9[y44NT87%u9**r4EU."

'对第一个串从第 20 位取 ascii 码，之后保留前 40 位
for i=20 to len(str)
str1=str1 & asc(mid(str,i,1))
next
str1=left(str1,40)

'得到机器码 ascii 右边一位的和 j：
for i= 1 to len(mc)  
j= j + cint(right(cstr(asc(mid(mc,i,1))),1))
next

'通过机器码获得第二个串：
for i=len(mc) to 1 step -1
str2=  str2 & cstr(asc(mid(mc,i,1)) *j)  
next
str2=left(str2,40)

'通过第一个串跟第二个串获得第三个串：
for i=1 to 40 step 2
str3=str3 & chr(cint(mid(str1,i,2)) xor cint(mid(str2,i,2)))
next

'对第三个串逐位检验：
For i = 1 To 20
    tmp = Asc(mid(str3, i, 1))
    If tmp <= &H19 Then
        tmp = tmp  + &H41
    ElseIf tmp <= &H33 Then
        tmp = tmp  + &H47
    ElseIf tmp <= &H3D Then
```



```vb
    tmp = tmp  - &H4  

    ElseIf tmp <= &H47 Then  

        tmp = tmp  - &HE  

    ElseIf tmp <= &H60 Then  

        'msgbox Chr(tmp - &H7)  

        tmp = tmp  - &H7  

    Else  

        tmp = tmp  + &HA  

    End If  

'过滤特殊字符，用'2'代替  

    if tmp<&H30 or (tmp>&H39 and tmp<&H41) or (tmp>&H5a and tmp<&H60) or tmp>&H7a then tmp= &H32  

    sn=sn & chr(tmp)  

Next  
```

算法比较繁琐，前后分析了 4 个小时。

发现机器码的算法跟注册码算法用的是同一个函数，只不过参数不一样。所有涉及到字符串长度的，比如 20，40 在计算机器码的时候变成 10， 20。

所以重新写一下注册模块：

```vb
Function sn(mc As String, reg As Boolean)  

    Dim mode, str, str1, str2, str3, i, j, tmp  

    '求机器码时设置为 10，注册码时为 20  

    If reg = True Then  

        mode = 20  

    Else  

        mode = 10  

    End If  

    str = "N  Z^%@\*&(HGUINKNUIGBF%kfinkcG76U^GUI766f^%SyI@(Ifdkj(\*8g7^&f65fFdIUYf6I^GjkdBUDhuipdfyig% $fjL\*\*.x02ytPHDOIDTphHPPiofDmllfl44190$#&\*JKho4o00wjdhFDK\*jo:;JUIp(B\[ApIuy^R&ubrYUi%6ge4uyU7OTd la;fyp0s0-2htdf!Jhfu7IK (\*$$#g%(m0FG\[D688g⁶fI8HDGF9\[y44NT87%u9\*\*r4EU."  

    For i = mode To Len(str)  

        str1 = str1 & Asc(Mid$(str, i, 1))  

    Next  

    str1 = Left$(str1, mode \* 2)  

    If reg = True And Len(mc) <> 10 Then  

        GoTo sn\_Error  

    End If  

    For i = 1 To Len(mc)  

        j = j + CInt(Right$(CStr(Asc(Mid$(mc, i, 1))), 1))  

    Next  

    For i = Len(mc) To 1 Step -1  

        str2 = str2 & CStr(Asc(Mid$(mc, i, 1)) \* j)  

    Next  

    str2 = Left$(str2, mode \* 2)  

    For i = 1 To mode \* 2 Step 2  

        str3 = str3 & Chr$(CInt(Mid$(str1, i, 2)) Xor CInt(Mid$(str2, i, 2)))  

    Next  

    For i = 1 To mode  

        tmp = Asc(Mid$(str3, i, 1))  

        If tmp <= &H19 Then  

            tmp = tmp + &H41  

        ElseIf tmp <= &H33 Then  

            tmp = tmp + &H47  

        ElseIf tmp <= &H3D Then  

            tmp = tmp - &H4  

        ElseIf tmp <= &H47 Then  

            tmp = tmp - &HE  

        ElseIf tmp <= &H60 Then  

            'msgbox Chr$(tmp - &H7)  

            tmp = tmp - &H7  

        Else  

            tmp = tmp + &HA  

        End If  

        If tmp < &H30 Or (tmp > &H39 And tmp < &H41) Or (tmp > &H5A And tmp < &H60) Or tmp > &H7A Then tmp = &H32  

        sn = sn & Chr$(tmp)  

    Next  

Exit Function  
```

机器码的获得算法：

```vb
    machineCode = sn(Trim$(硬盘序列号) & "March3000000000000", False)
```

注册码的算法：

```vb
    Serial = sn(机器码, True)
```

这个是 `win2003` 的情况，但是似乎如果其他系统，机器码的算法有点变化，具体参照 `005079A2`，`march3` 后面又加上了一些数字，等到有机会再试 了。

`4dd466d0e1e0cc123afbe504da2462ca`


# Ease MP3 Recorder 1.50 算法分析

> 原文：[`www.pediy.com/kssd/pediy10/83290.html`](https://www.pediy.com/kssd/pediy10/83290.html)

【破文标题】Ease MP3 Recorder 1.50 算法分析

【破文作者】tianxj

【作者邮箱】tianxj_2007@126.com

【作者主页】WwW.ChiNaPYG.CoM

【破解工具】PEiD, OD

【破解平台】Windows XP

【软件名称】Ease MP3 Recorder 1.50

【软件大小】4292KB

【软件类别】国外软件/音频处理

【软件授权】共享版

【软件语言】英文

【运行环境】Win9x/Me/NT/2000/XP/2003

【更新时间】2007-9-28

【原版下载】`http://www.audiotool.net/download/mp3recorder.exe`

【保护方式】注册码

【软件简介】Audiotool Ease MP3 Recorder 可以录制十几种格式的音乐文件，包括：WAV、MP3、OGG、WMA、GSM、ADPCM、VOX、RAW、DSP、GSM、G726、G23 等，任何透过声卡播放，或是经由麦克风、音源线输入声卡的声音皆可录制。

【破解声明】我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---------------------------------------

【破解内容】

---------------------------------------

一、运行程序，进行注册，输入错误的注册信息进行检测，有提示信息

二、用 `PEiD` 对 `EaseMP3Recorder.exe` 查壳，为 Borland Delphi 6.0 - 7.0

三、运行 `OD`，打开 `EaseMP3Recorder.exe`，根据 `DeDe` 的按钮事件来到关键之处

==============================================================

```
004985C0  /.  55            PUSH    EBP
004985C1  |.  8BEC          MOV     EBP, ESP
004985C3  |.  33C9          XOR     ECX, ECX
004985C5  |.  51            PUSH    ECX
004985C6  |.  51            PUSH    ECX
004985C7  |.  51            PUSH    ECX
004985C8  |.  51            PUSH    ECX
004985C9  |.  51            PUSH    ECX
004985CA  |.  51            PUSH    ECX
004985CB  |.  53            PUSH    EBX
004985CC  |.  56            PUSH    ESI
004985CD  |.  57            PUSH    EDI
004985CE  |.  8BD8          MOV     EBX, EAX
004985D0  |.  33C0          XOR     EAX, EAX
004985D2  |.  55            PUSH    EBP
004985D3  |.  68 26884900   PUSH    EaseMP3R.00498826
004985D8  |.  64:FF30       PUSH    DWORD PTR FS:[EAX]
004985DB  |.  64:8920       MOV     DWORD PTR FS:[EAX], ESP
004985DE  |.  8D55 F4       LEA     EDX, DWORD PTR SS:[EBP-C]
004985E1  |.  8B83 F8020000 MOV     EAX, DWORD PTR DS:[EBX+2F8]
004985E7  |.  E8 488CFAFF   CALL    EaseMP3R.00441234
004985EC  |.  8B45 F4       MOV     EAX, DWORD PTR SS:[EBP-C]        ;  //用户名
004985EF  |.  8D55 FC       LEA     EDX, DWORD PTR SS:[EBP-4]
004985F2  |.  E8 AD04F7FF   CALL    EaseMP3R.00408AA4
004985F7  |.  8D55 F0       LEA     EDX, DWORD PTR SS:[EBP-10]
004985FA  |.  8B45 FC       MOV     EAX, DWORD PTR SS:[EBP-4]
004985FD  |.  E8 D604F7FF   CALL    EaseMP3R.00408AD8
00498602  |.  8B55 F0       MOV     EDX, DWORD PTR SS:[EBP-10]
00498605  |.  8D45 FC       LEA     EAX, DWORD PTR SS:[EBP-4]
00498608  |.  E8 2BC0F6FF   CALL    EaseMP3R.00404638
0049860D  |.  8D55 EC       LEA     EDX, DWORD PTR SS:[EBP-14]
00498610  |.  8B83 FC020000 MOV     EAX, DWORD PTR DS:[EBX+2FC]
00498616  |.  E8 198CFAFF   CALL    EaseMP3R.00441234
0049861B  |.  8B45 EC       MOV     EAX, DWORD PTR SS:[EBP-14]       ;  //试炼码
0049861E  |.  8D55 F8       LEA     EDX, DWORD PTR SS:[EBP-8]
00498621  |.  E8 7E04F7FF   CALL    EaseMP3R.00408AA4
00498626  |.  8D55 E8       LEA     EDX, DWORD PTR SS:[EBP-18]
00498629  |.  8B45 F8       MOV     EAX, DWORD PTR SS:[EBP-8]
0049862C  |.  E8 A704F7FF   CALL    EaseMP3R.00408AD8
00498631  |.  8B55 E8       MOV     EDX, DWORD PTR SS:[EBP-18]
00498634  |.  8D45 F8       LEA     EAX, DWORD PTR SS:[EBP-8]
00498637  |.  E8 FCBFF6FF   CALL    EaseMP3R.00404638
0049863C  |.  8B45 FC       MOV     EAX, DWORD PTR SS:[EBP-4]        ;  //用户名
0049863F  |.  E8 14C2F6FF   CALL    EaseMP3R.00404858                ;  //取用户名长度
00498644  |.  85C0          TEST    EAX, EAX
00498646  |.  0F84 9F010000 JE      EaseMP3R.004987EB                ;  //用户名为空则挂
0049864C  |.  8B45 F8       MOV     EAX, DWORD PTR SS:[EBP-8]        ;  //试炼码
0049864F  |.  E8 04C2F6FF   CALL    EaseMP3R.00404858                ;  //取试炼码长度
00498654  |.  85C0          TEST    EAX, EAX
00498656  |.  75 1A         JNZ     SHORT EaseMP3R.00498672          ;  //试炼码为空则挂
00498658  |.  6A 00         PUSH    0
0049865A  |.  66:8B0D 34884>MOV     CX, WORD PTR DS:[498834]
00498661  |.  33D2          XOR     EDX, EDX
00498663  |.  B8 40884900   MOV     EAX, EaseMP3R.00498840           ;  ASCII "Code must not be null."
00498668  |.  E8 EF22FAFF   CALL    EaseMP3R.0043A95C
0049866D  |.  E9 79010000   JMP     EaseMP3R.004987EB
00498672  |>  8B45 F8       MOV     EAX, DWORD PTR SS:[EBP-8]
00498675  |.  E8 DEC1F6FF   CALL    EaseMP3R.00404858
0049867A  |.  85C0          TEST    EAX, EAX
0049867C  |.  7E 38         JLE     SHORT EaseMP3R.004986B6
0049867E  |.  BA 01000000   MOV     EDX, 1
00498683  |>  8B4D F8       /MOV     ECX, DWORD PTR SS:[EBP-8]
00498686  |.  0FB67411 FF   |MOVZX   ESI, BYTE PTR DS:[ECX+EDX-1]
0049868B  |.  83FE 30       |CMP     ESI, 30
0049868E  |.  7C 08         |JL      SHORT EaseMP3R.00498698
00498690  |.  8B4D F8       |MOV     ECX, DWORD PTR SS:[EBP-8]
00498693  |.  83FE 39       |CMP     ESI, 39
00498696  |.  7E 1A         |JLE     SHORT EaseMP3R.004986B2
00498698  |>  6A 00         |PUSH    0
0049869A  |.  66:8B0D 34884>|MOV     CX, WORD PTR DS:[498834]
004986A1  |.  33D2          |XOR     EDX, EDX
004986A3  |.  B8 60884900   |MOV     EAX, EaseMP3R.00498860          ;  ASCII "The code must be integer!"
004986A8  |.  E8 AF22FAFF   |CALL    EaseMP3R.0043A95C
004986AD  |.  E9 39010000   |JMP     EaseMP3R.004987EB
004986B2  |>  42            |INC     EDX
004986B3  |.  48            |DEC     EAX
004986B4  |.^ 75 CD         \JNZ     SHORT EaseMP3R.00498683         ;  //循环,检测试炼码是否为纯数字
004986B6  |>  BE 01000000   MOV     ESI, 1                           ;  //ESI=1
004986BB  |.  8B45 FC       MOV     EAX, DWORD PTR SS:[EBP-4]        ;  //用户名
004986BE  |.  E8 95C1F6FF   CALL    EaseMP3R.00404858                ;  //取用户名长度
004986C3  |.  85C0          TEST    EAX, EAX                         ;  //EAX=用户名长度
004986C5  |.  7E 13         JLE     SHORT EaseMP3R.004986DA
004986C7  |.  BA 01000000   MOV     EDX, 1
004986CC  |>  8B4D FC       /MOV     ECX, DWORD PTR SS:[EBP-4]       ;  //用户名
004986CF  |.  0FB64C11 FF   |MOVZX   ECX, BYTE PTR DS:[ECX+EDX-1]    ;  //逐位取用户名 ASCII 码
004986D4  |.  03F1          |ADD     ESI, ECX                       ;  //ESI=ESI+ECX
004986D6  |.  42            |INC     EDX
004986D7  |.  48            |DEC     EAX
004986D8  |.^ 75 F2         \JNZ     SHORT EaseMP3R.004986CC         ;  //循环
004986DA  |>  69C6 98050000 IMUL    EAX, ESI, 598                   ;  //EAX=ESI*598
004986E0  |.  05 155E0100   ADD     EAX, 15E15                      ;  //EAX=EAX+15E15
004986E5  |.  8BF0          MOV     ESI, EAX
004986E7  |.  8B45 F8       MOV     EAX, DWORD PTR SS:[EBP-8]       ;  //试炼码
004986EA  |.  E8 69C1F6FF   CALL    EaseMP3R.00404858                ;  //取试炼码长度
004986EF  |.  83F8 0A       CMP     EAX, 0A
004986F2  |.  0F8F DE000000 JG      EaseMP3R.004987D6                ;  //试炼码长度大于 Ah 则挂
004986F8  |.  8B45 F8       MOV     EAX, DWORD PTR SS:[EBP-8]
004986FB  |.  E8 6419F7FF   CALL    EaseMP3R.0040A064
00498700  |.  DB2D 7C884900 FLD     TBYTE PTR DS:[49887C]
00498706  |.  DED9          FCOMPP
00498708  |.  DFE0          FSTSW   AX
0049870A  |.  9E            SAHF
0049870B  |.  0F82 AE000000 JB      EaseMP3R.004987BF                ;  //试炼码大于 2147483647 则挂
```



### 00498711 汇编指令
```
00498711  |.  8B45 F8       MOV     EAX, DWORD PTR SS:[EBP-8]        ;  //试炼码  
00498714  |.  E8 BB04F7FF   CALL    EaseMP3R.00408BD4                ;  //将试炼码转 16 进制送入 EAX  
00498719  |.  3BF0          CMP     ESI, EAX                         ;  //关键比较,ESI 为真码,EAX 为假码  
0049871B  |.  0F85 87000000 JNZ     EaseMP3R.004987A8                ;  //关键跳转  
00498721  |.  B2 01         MOV     DL, 1  
00498723  |.  A1 C4144700   MOV     EAX, DWORD PTR DS:[4714C4]  
00498728  |.  E8 978EFDFF   CALL    EaseMP3R.004715C4  
0049872D  |.  8BF8          MOV     EDI, EAX  
0049872F  |.  BA 02000080   MOV     EDX, 80000002  
00498734  |.  8BC7          MOV     EAX, EDI  
00498736  |.  E8 298FFDFF   CALL    EaseMP3R.00471664  
0049873B  |.  B1 01         MOV     CL, 1  
0049873D  |.  BA 90884900   MOV     EDX, EaseMP3R.00498890           ;  ASCII "\SOFTWARE\EASETECH\EASEMP3RECORDER"  
00498742  |.  8BC7          MOV     EAX, EDI  
00498744  |.  E8 5B90FDFF   CALL    EaseMP3R.004717A4  
00498749  |.  8B4D FC       MOV     ECX, DWORD PTR SS:[EBP-4]  
0049874C  |.  BA BC884900   MOV     EDX, EaseMP3R.004988BC           ;  ASCII "registry name"  
00498751  |.  8BC7          MOV     EAX, EDI  
00498753  |.  E8 E891FDFF   CALL    EaseMP3R.00471940  
00498758  |.  8BCE          MOV     ECX, ESI  
0049875A  |.  BA D4884900   MOV     EDX, EaseMP3R.004988D4           ;  ASCII "registry code"  
0049875F  |.  8BC7          MOV     EAX, EDI  
00498761  |.  E8 7E92FDFF   CALL    EaseMP3R.004719E4  
00498766  |.  B1 01         MOV     CL, 1  
00498768  |.  BA EC884900   MOV     EDX, EaseMP3R.004988EC           ;  ASCII "regsuccess"  
0049876D  |.  8BC7          MOV     EAX, EDI  
0049876F  |.  E8 B492FDFF   CALL    EaseMP3R.00471A28  
00498774  |.  8BC7          MOV     EAX, EDI  
00498776  |.  E8 B98EFDFF   CALL    EaseMP3R.00471634  
0049877B  |.  8BC7          MOV     EAX, EDI  
0049877D  |.  E8 0EB0F6FF   CALL    EaseMP3R.00403790  
00498782  |.  6A 00         PUSH    0  
00498784  |.  66:8B0D 34884>MOV     CX, WORD PTR DS:[498834]  
0049878B  |.  B2 02         MOV     DL, 2  
0049878D  |.  B8 00894900   MOV     EAX, EaseMP3R.00498900           ;  ASCII "Congratuation! You have registered!"  
00498792  |.  E8 C521FAFF   CALL    EaseMP3R.0043A95C  
00498797  |.  A1 94784A00   MOV     EAX, DWORD PTR DS:[4A7894]  
0049879C  |.  C600 01       MOV     BYTE PTR DS:[EAX], 1  
0049879F  |.  8BC3          MOV     EAX, EBX  
004987A1  |.  E8 C654FCFF   CALL    EaseMP3R.0045DC6C  
004987A6  |.  EB 43         JMP     SHORT EaseMP3R.004987EB  
004987A8  |>  6A 00         PUSH    0  
004987AA  |.  66:8B0D 34884>MOV     CX, WORD PTR DS:[498834]  
004987B1  |.  B2 02         MOV     DL, 2  
004987B3  |.  B8 2C894900   MOV     EAX, EaseMP3R.0049892C           ;  ASCII "Invalid register code!Please retry!"  
004987B8  |.  E8 9F21FAFF   CALL    EaseMP3R.0043A95C  
004987BD  |.  EB 2C         JMP     SHORT EaseMP3R.004987EB  
004987BF  |>  6A 00         PUSH    0  
004987C1  |.  66:8B0D 34884>MOV     CX, WORD PTR DS:[498834]  
004987C8  |.  33D2          XOR     EDX, EDX  
004987CA  |.  B8 58894900   MOV     EAX, EaseMP3R.00498958           ;  ASCII "The code is overload!Please retry!"  
004987CF  |.  E8 8821FAFF   CALL    EaseMP3R.0043A95C  
004987D4  |.  EB 15         JMP     SHORT EaseMP3R.004987EB  
004987D6  |>  6A 00         PUSH    0  
004987D8  |.  66:8B0D 34884>MOV     CX, WORD PTR DS:[498834]  
004987DF  |.  33D2          XOR     EDX, EDX  
004987E1  |.  B8 58894900   MOV     EAX, EaseMP3R.00498958           ;  ASCII "The code is overload!Please retry!"  
004987E6  |.  E8 7121FAFF   CALL    EaseMP3R.0043A95C  
004987EB  |>  33C0          XOR     EAX, EAX  
004987ED  |.  5A            POP     EDX  
004987EE  |.  59            POP     ECX  
004987EF  |.  59            POP     ECX  
004987F0  |.  64:8910       MOV     DWORD PTR FS:[EAX], EDX  
004987F3  |.  68 2D884900   PUSH    EaseMP3R.0049882D  
004987F8  |>  8D45 E8       LEA     EAX, DWORD PTR SS:[EBP-18]  
004987FB  |.  E8 A0BDF6FF   CALL    EaseMP3R.004045A0  
00498800  |.  8D45 EC       LEA     EAX, DWORD PTR SS:[EBP-14]  
00498803  |.  E8 98BDF6FF   CALL    EaseMP3R.004045A0  
00498808  |.  8D45 F0       LEA     EAX, DWORD PTR SS:[EBP-10]  
0049880B  |.  E8 90BDF6FF   CALL    EaseMP3R.004045A0  
00498810  |.  8D45 F4       LEA     EAX, DWORD PTR SS:[EBP-C]  
00498813  |.  E8 88BDF6FF   CALL    EaseMP3R.004045A0  
00498818  |.  8D45 F8       LEA     EAX, DWORD PTR SS:[EBP-8]  
0049881B  |.  BA 02000000   MOV     EDX, 2  
00498820  |.  E8 9FBDF6FF   CALL    EaseMP3R.004045C4  
00498825  \\.  C3            RETN  
00498826   .^ E9 F9B6F6FF   JMP     EaseMP3R.00403F24  
0049882B   .^ EB CB         JMP     SHORT EaseMP3R.004987F8  
0049882D   .  5F            POP     EDI  
0049882E   .  5E            POP     ESI  
0049882F   .  5B            POP     EBX  
00498830   .  8BE5          MOV     ESP, EBP  
00498832   .  5D            POP     EBP  
00498833   .  C3            RETN  
```

### 破解总结

### 算法总结
注册码 = (用户名 ASCII 码累加值 + 1) * 1432 + 89621

### 算法注册机
`KeyGen.rek`

```
.const
.data
szHomePage db "http://www.chinapyg.com",0
szEmail    db "mailto:tianxj_2007@126.com",0
szErrMess  db "请输入用户名！",0
szFMT      db "%u",0
szBuffer1  db 50 dup (0)
.code
invoke **lstrlen**,eax
MOV     ESI, 1
MOV     EDX, 1
n1:
LEA     ECX, hInput1
MOVZX   ECX, BYTE PTR DS:[ECX+EDX-1]
ADD     ESI, ECX
INC     EDX
DEC     EAX
JNZ     n1
IMUL    EAX, ESI, 598h
ADD     EAX, 15E15h
invoke **wsprintf**,addr szBuffer1,addr szFMT,eax
lea eax,szBuffer1
```

### 内存注册机
中断地址 `00498719`
中断次数 `1`
第一字节 `3B`
指令长度 `2`
寄存器方式-ESI
十进制

### 注册信息
保存在 `[HKEY_LOCAL_MACHINE\SOFTWARE\EASETECH\EASEMP3RECORDER]`

感谢飘云老大、猫老大、Nisy 老大以及很多前辈们的学习教程以及王者之剑、云龙等所有帮助过我的论坛兄弟姐妹们！谢谢

### 版权声明
破文是学习的手记，兴趣是成功的源泉；本破文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

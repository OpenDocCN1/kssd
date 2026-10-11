
# 电脑阅卷王 PC OMR V6.2 的注册+汉化

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-245.htm`](https://www.pediy.com/kssd/pediy07/pediy7-245.htm)

【破文作者】 Rdsnow\[BCG\]\[PYG\]

【 E-mail 】 RDSNOW@163.COM

【 作者 QQ 】 83757177

【文章题目】 PC OMR V6.2 的注册+汉化

【软件名称】 电脑阅卷王(PC OMR) V6.2

【下载地址】 [`www.cai2000.com/download.htm`](http://www.cai2000.com/download.htm)

----------------------------------------------------------------------------------------------

【加密方式】 序列号

【破解工具】 FlyOD V1.10

【软件限制】 功能限制

【破解平台】 Windows XP SP2

----------------------------------------------------------------------------------------------

【软件简介】

PC OMR V6.2 即电脑阅卷王 V5.2 的升级版本，但只有英文版，它是一款创新型的标准化试卷阅卷系统，教师必备软件。该系统采用数码相机（或扫描仪、摄像头、数码摄象机等）作为答题卡图象输入设备，取代动辄几十万元的昂贵的光电阅卷机，无须专用答题卡，成本极低、操作简易、功能更强大，更适合于中小学、考试中心、调查机构使用。

----------------------------------------------------------------------------------------------

【文章简介】

偶从没有汉化过软件，以前调试过程序的 V5.2 版本，这次帮网友调试 V6.2 版本，竟然新版本没有中文版，于是把 V5.2 版目录下的 `language.txt` 复制到新版本的目录下，汉化过程就这么多了。快吧！

程序是 VB 编写的东东，调试起来需要有点耐心，用了浮点指令，也增加了注册的难度。

注册码的计算分四部分分别放在程序中，第一个很容易找到，满足了第一个条件，程序会提示注册成功，但是使用几分钟后，会被后面的条件检验，不通过又会变回未注册版。作者把后面还有三个条件放在 `Timer` 里，倒是有些隐蔽。

----------------------------------------------------------------------------------------------

【破解过程】

调试过程中偶的机器码"3887567035"，调试过程中机器码的第一位和第六位是没有校验后会变化。注册码跟这两位应该没有关系。

输入用户名：rdsnow\[BCG\]\[PYG\]；假注册码"9876453210"

字符串中找到 UNICODE "SHURUYOUWUQINGCHONGXINSHURU"，这是"输入有误，请重新输入"的汉语拼音，下断，来到这儿：

```
0047676C   .  8B55 D0           MOV EDX,DWORD PTR SS:[EBP-30]
0047676F   .  83C4 0C           ADD ESP,0C
00476772   .  52                PUSH EDX
00476773   .  FF15 34104000     CALL DWORD PTR DS:[<&MSVBVM60.__vbaLenBstr>]    ;  MSVBVM60.__vbaLenBstr
00476779   .  83F8 0A           CMP EAX,0A                                ;  假注册码的长度是否为 10
0047677C   .  0F84 C1000000     JE pcomr6.00476843
00476782   .  BA 88BA4000       MOV EDX,pcomr6.0040BA88                   ;  UNICODE "SHURUYOUWUQINGCHONGXINSHURU"
00476787   .  8D4D C8           LEA ECX,DWORD PTR SS:[EBP-38]
0047678A   .  FF15 10124000     CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCopy>]    ;  MSVBVM60.__vbaStrCopy
00476790   .  BA ECB84000       MOV EDX,pcomr6.0040B8EC                   ;  UNICODE "FRM_REG"
00476795   .  8D4D CC           LEA ECX,DWORD PTR SS:[EBP-34]
00476798   .  FF15 10124000     CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCopy>]    ;  MSVBVM60.__vbaStrCopy
0047679E   .  8D45 C8           LEA EAX,DWORD PTR SS:[EBP-38]
004767A1   .  50                PUSH EAX
004767A2   .  8D4D CC           LEA ECX,DWORD PTR SS:[EBP-34]
004767A5   .  51                PUSH ECX
004767A6   .  E8 55760500       CALL pcomr6.004CDE00
004767AB   .  8BD0              MOV EDX,EAX
004767AD   .  8D4D C0           LEA ECX,DWORD PTR SS:[EBP-40]
004767B0   .  FFD6              CALL ESI
004767B2   .  8B55 C0           MOV EDX,DWORD PTR SS:[EBP-40]
004767B5   .  B9 0A000000       MOV ECX,0A
004767BA   .  B8 04000280       MOV EAX,80020004
004767BF   .  898D 7CFFFFFF     MOV DWORD PTR SS:[EBP-84],ECX
004767C5   .  894D 8C           MOV DWORD PTR SS:[EBP-74],ECX
004767C8   .  894D 9C           MOV DWORD PTR SS:[EBP-64],ECX
004767CB   .  68 7CBA4000       PUSH pcomr6.0040BA7C                      ;  UNICODE "  "
004767D0   .  8D4D C4           LEA ECX,DWORD PTR SS:[EBP-3C]
004767D3   .  8945 84           MOV DWORD PTR SS:[EBP-7C],EAX
004767D6   .  8945 94           MOV DWORD PTR SS:[EBP-6C],EAX
004767D9   .  8945 A4           MOV DWORD PTR SS:[EBP-5C],EAX
004767DC   .  895D C0           MOV DWORD PTR SS:[EBP-40],EBX
004767DF   .  FFD6              CALL ESI
004767E1   .  50                PUSH EAX
004767E2   .  FF15 68104000     CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCat>]     ;  MSVBVM60.__vbaStrCat
004767E8   .  8D95 7CFFFFFF     LEA EDX,DWORD PTR SS:[EBP-84]
004767EE   .  52                PUSH EDX
004767EF   .  8945 B4           MOV DWORD PTR SS:[EBP-4C],EAX
004767F2   .  8D45 8C           LEA EAX,DWORD PTR SS:[EBP-74]
004767F5   .  50                PUSH EAX
004767F6   .  8D4D 9C           LEA ECX,DWORD PTR SS:[EBP-64]
004767F9   .  51                PUSH ECX
004767FA   .  53                PUSH EBX
004767FB   .  8D55 AC           LEA EDX,DWORD PTR SS:[EBP-54]
004767FE   .  52                PUSH EDX
004767FF   .  C745 AC 08000000  MOV DWORD PTR SS:[EBP-54],8
00476806   .  FF15 B8104000     CALL DWORD PTR DS:[<&MSVBVM60.#595>]           ;  MSVBVM60.rtcMsgBox
0047680C   .  8D45 C0           LEA EAX,DWORD PTR SS:[EBP-40]
0047680F   .  50                PUSH EAX
00476810   .  8D4D C4           LEA ECX,DWORD PTR SS:[EBP-3C]
00476813   .  51                PUSH ECX
00476814   .  8D55 C8           LEA EDX,DWORD PTR SS:[EBP-38]
00476817   .  52                PUSH EDX
00476818   .  8D45 CC           LEA EAX,DWORD PTR SS:[EBP-34]
0047681B   .  50                PUSH EAX
0047681C   .  6A 04             PUSH 4
0047681E   .  FF15 1C124000     CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeStrList>>;  MSVBVM60.__vbaFreeStrList
00476824   .  8D8D 7CFFFFFF     LEA ECX,DWORD PTR SS:[EBP-84]
0047682A   .  51                PUSH ECX
0047682B   .  8D55 8C           LEA EDX,DWORD PTR SS:[EBP-74]
0047682E   .  52                PUSH EDX
0047682F   .  8D45 9C           LEA EAX,DWORD PTR SS:[EBP-64]
00476832   .  50                PUSH EAX
00476833   .  8D4D AC           LEA ECX,DWORD PTR SS:[EBP-54]
00476836   .  51                PUSH ECX
00476837   .  6A 04             PUSH 4
00476839   .  FFD7              CALL EDI
0047683B   .  83C4 28           ADD ESP,28
0047683E   .  E9 CC040000       JMP pcomr6.00476D0F
00476843   >  BE 01000000       MOV ESI,1                                   ;  ESI=1,准备循环
00476848   >  B8 08000000       MOV EAX,8
0047684D   .  66:3BF0          CMP SI,AX
00476850   .  0F8F B5000000     JG pcomr6.0047690B                      ;  ESI 大于 8，就跳出循环
00476856   .  8D45 AC           LEA EAX,DWORD PTR SS:[EBP-54]
00476859   .  50                PUSH EAX
0047685A   .  0FBFCE           MOVSX ECX,SI
0047685D   .  8D55 D0           LEA EDX,DWORD PTR SS:[EBP-30]
00476860   .  8995 74FFFFFF     MOV DWORD PTR SS:[EBP-8C],EDX
00476866   .  51                PUSH ECX
00476867   .  8D95 6CFFFFFF     LEA EDX,DWORD PTR SS:[EBP-94]
0047686D   .  52                PUSH EDX
0047686E   .  8D45 9C           LEA EAX,DWORD PTR SS:[EBP-64]
00476871   .  50                PUSH EAX
00476872   .  C745 B4 01000000  MOV DWORD PTR SS:[EBP-4C],1
00476879   .  C745 AC 02000000  MOV DWORD PTR SS:[EBP-54],2
00476880   .  C785 6CFFFFFF 0840>MOV DWORD PTR SS:[EBP-94],4008
0047688A   .  FF15 F8104000     CALL DWORD PTR DS:[<&MSVBVM60.#632>]           ;  MSVBVM60.rtcMidCharVar
00476890   .  8D4D 9C           LEA ECX,DWORD PTR SS:[EBP-64]
00476893   .  51                PUSH ECX
00476894   .  8D55 CC           LEA EDX,DWORD PTR SS:[EBP-34]
00476897   .  52                PUSH EDX
00476898   .  FF15 BC114000     CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrVarVal>]  ;  MSVBVM60.__vbaStrVarVal
0047689E   .  50                PUSH EAX
0047689F   .  FF15 E0124000     CALL DWORD PTR DS:[<&MSVBVM60.#581>]           ;  MSVBVM60.rtcR8ValFromBstr
004768A5   .  DD9D 2CFFFFFF     FSTP QWORD PTR SS:[EBP-D4]
004768AB   .  0FBFC3           MOVSX EAX,BX
004768AE   .  8985 00FFFFFF     MOV DWORD PTR SS:[EBP-100],EAX
004768B4   .  DB85 00FFFFFF     FILD DWORD PTR SS:[EBP-100]                     ;  存放累加结果，初始数值等于 0
```

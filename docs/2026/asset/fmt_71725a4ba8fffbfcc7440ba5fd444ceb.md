
# 斗地主7.2版算法分析

[软件介绍]“斗地主”是近来流行全国的一种扑克游戏。

该游戏由四人玩两副牌（108张）或三人玩一副牌（54张），地主为一方，其余三家或两家为另一方，双方对战，先出完的一方胜。出牌规则类似“争上游”。

[使用限制] 试用30次和每次7局限制。

[保护方式] 采用机器码与注册名联合检查注册码，以保证注册码不得多机使用。

[破解工具] olldbg1.10 、Peid0.92、Aspackdie1.41,RegSnap

[作者] CrackerWu[BCG]

## 破解过程

### 1. 软件信息

该软件用VB编写，加ASPack 2.12 壳。

用RegSnap可发现试用次数保存在如下两处位置：

```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProgrameID
   Old value: String: "6"   New value: String: "7"

HKEY_LOCAL_MACHINE\SOFTWARE\DDZ\Infomation\LicenseNumber
   Old value: String: "6"   New value: String: "7"
```

注册码保存在：

```
[HKEY_LOCAL_MACHINE\SOFTWARE\DDZ\Infomation]
@="斗地主软件信息"
"SetupDir"="d:\DDZ"
"ProductId"="DDZ070040040530"
"ProductName"="斗地主扑克游戏"
"Writer"="周军"
"Version"="7.2 Build 668"
"LicenseNumber"="7"
"CPUCode"="6659630247"
"LicenseDate"="2005-3-8"
"RegName"="CrackerWu[BCG]"
"RegCode"=" "
"SetupDate"="2005-3-8"
```

从《斗地主》扑克游戏注册说明及帮助中得到如下信息：

注册名（中英文均可，但至少要两个字符以上），采用机器码与注册名联合检查注册码。从注册框中输入注册码=10位才能注册。

任意输入注册码：1234567890，重启软件发现软件标题栏中 [未注册]字样消失了，但玩到第7局就回弹出注册对话框，要求注册。

再运行软件[未注册]字样又出现了。检查注册表发现输入注册码被清除啦。

看来注册流程可能是：在启动时先检查注册码是否存在，最后到第7局才判断是否正确。

### 2. 脱壳后自校验

脱壳后软件不能正常运行，跟踪可发现在如下位置有自校验：

```
004AD46D   . 52             PUSH EDX
004AD46E   . FF15 20134000  CALL DWORD PTR DS:[<&MSVBVM60.#578>]     ;  MSVBVM60.rtcFileLen
004AD474   . 3D 801A0600    CMP EAX,61A80                       ;比较文件长度若大于等于61A80 就玩完啦！
004AD479     7D 07          JGE SHORT unpacked.004AD482
004ADCFD   . 52             PUSH EDX
004ADCFE   . FF15 20134000  CALL DWORD PTR DS:[<&MSVBVM60.#578>]     ;  MSVBVM60.rtcFileLen
004ADD04   . 3D 801A0600    CMP EAX,61A80                    ;比较文件长度若大于等于61A80 就玩完啦！
004ADD09     7D 07          JGE SHORT unpacked.004ADD12
```

把上面两处跳转`NOP`掉，软件可正常运行。

### 3. 注册码检查分析

用OD载入后通过查找参靠字符串：`regcode` 发现有10处，全部设断再一一跟踪可看到它是怎样检查注册码的。

下面这段是游戏中每次点击发牌后都要被OD中断在这里：

```
00440D9E   . FF15 B0104000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaHresultCheckObj>;  MSVBVM60.__vbaHresultCheckObj
00440DA4   > 8B1D F8124000  MOV EBX,DWORD PTR DS:[<&MSVBVM60.__vbaStrCopy>]    ;  MSVBVM60.__vbaStrCopy
00440DAA   . BA DCED4100    MOV EDX,unpacked.0041EDDC                          ;  UNICODE "regcode"
00440DAF   . 8D4D DC        LEA ECX,DWORD PTR SS:[EBP-24]
00440DB2   . FFD3           CALL EBX                                           ;  <&MSVBVM60.__vbaStrCopy>
00440DB4   . BA 90B74100    MOV EDX,unpacked.0041B790                          ;  UNICODE "SoftWare\DDZ\Infomation"
00440DB9   . 8D4D E0        LEA ECX,DWORD PTR SS:[EBP-20]
00440DBC   . FFD3           CALL EBX
00440DBE   . 8D45 DC        LEA EAX,DWORD PTR SS:[EBP-24]
00440DC1   . 50             PUSH EAX                            ;"regcode"入栈
00440DC2   . 8D4D E0        LEA ECX,DWORD PTR SS:[EBP-20]
00440DC5   . 51             PUSH ECX                            ; "SoftWare\DDZ\Infomation"入栈
00440DC6   . 8D55 9C        LEA EDX,DWORD PTR SS:[EBP-64]
00440DC9   . 52             PUSH EDX
00440DCA   . C745 9C 020000>MOV DWORD PTR SS:[EBP-64],80000002
00440DD1   . E8 9A060600    CALL unpacked.004A1470               ;调用一个通用子程序从注册表中读取注册码。
00440DD6   . 8BD0           MOV EDX,EAX
00440DD8   . 8D4D D8        LEA ECX,DWORD PTR SS:[EBP-28]
00440DDB   . FF15 98134000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrMove>]       ;  MSVBVM60.__vbaStrMove
00440DE1   . 50             PUSH EAX
00440DE2   . 68 38B84100    PUSH unpacked.0041B838
00440DE7   . FF15 A4114000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrCmp>]        ;  MSVBVM60.__vbaStrCmp
00440DED   . 8B55 A0        MOV EDX,DWORD PTR SS:[EBP-60]
00440DF0   . 8BF0           MOV ESI,EAX
00440DF2   . F7DE           NEG ESI
00440DF4   . 1BF6           SBB ESI,ESI
00440DF6   . 46             INC ESI
00440DF7   . 8D45 D8        LEA EAX,DWORD PTR SS:[EBP-28]
00440DFA   . 50             PUSH EAX
00440DFB   . F7DE           NEG ESI
00440DFD   . 8D4D DC        LEA ECX,DWORD PTR SS:[EBP-24]
00440E00   . 0BF2           OR ESI,EDX
00440E02   . 51             PUSH ECX
00440E03   . 8D55 E0        LEA EDX,DWORD PTR SS:[EBP-20]
00440E06   . 52             PUSH EDX
00440E07   . 6A 03          PUSH 3
00440E09   . FF15 08134000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeStrList>]   ;  MSVBVM60.__vbaFreeStrList
00440E0F   . 83C4 10        ADD ESP,10
00440E12   . 8D4D D4        LEA ECX,DWORD PTR SS:[EBP-2C]
00440E15   . FF15 E4134000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeObj>]       ;  MSVBVM60.__vbaFreeObj
00440E1B   . 66:85F6        TEST SI,SI                      ;(爆破可改为 xor si,si  保正一定跳过下面限制)
00440E1E   . 0F84 62020000  JE unpacked.00441086           ;SI若为零就跳过试用30次和每次7局限制.
00440E24   . 833D 3C335200 >CMP DWORD PTR DS:[52333C],1E  ;比较试用次数时否大于30次
00440E2B   . 7D 1B          JGE SHORT unpacked.00440E48
00440E2D   . 833D 70315200 >CMP DWORD PTR DS:[523170],7   ;比较游戏局数时否小于7局
00440E34   . 0F8C 4C020000  JL unpacked.00441086
00440E3A   . 66:833D 003152>CMP WORD PTR DS:[523100],0
00440E42   . 0F85 3E020000  JNZ unpacked.00441086
00440E48   > 66:833D 003152>CMP WORD PTR DS:[523100],0
00440E50   . 75 0A          JNZ SHORT unpacked.00440E5C
00440E52   . 66:833D 403352>CMP WORD PTR DS:[523340],0
00440E5A   . 74 2A          JE SHORT unpacked.00440E86
00440E5C   > 8B07           MOV EAX,DWORD PTR DS:[EDI]
00440E5E   . 8D4D A0        LEA ECX,DWORD PTR SS:[EBP-60]
00440E61   . 51             PUSH ECX
00440E62   . 57             PUSH EDI
```

当游戏玩到第7局时被OD首先中断在这里：

```
004599F4   . BA DCED4100    MOV EDX,unpacked.0041EDDC                          ;  UNICODE "regcode"
004599F9   . 8D4D C0        LEA ECX,DWORD PTR SS:[EBP-40]
004599FC   . 897D E8        MOV DWORD PTR SS:[EBP-18],EDI
004599FF   . 897D E0        MOV DWORD PTR SS:[EBP-20],EDI
00459A02   . 897D D4        MOV DWORD PTR SS:[EBP-2C],EDI
00459A05   . 897D D0        MOV DWORD PTR SS:[EBP-30],EDI
00459A08   . 897D CC        MOV DWORD PTR SS:[EBP-34],EDI
00459A0B   . 897D C8        MOV DWORD PTR SS:[EBP-38],EDI
00459A0E   . 897D C4        MOV DWORD PTR SS:[EBP-3C],EDI
00459A11   . 897D C0        MOV DWORD PTR SS:[EBP-40],EDI
00459A14   . 897D BC        MOV DWORD PTR SS:[EBP-44],EDI
00459A17   . 897D B8        MOV DWORD PTR SS:[EBP-48],EDI
00459A1A   . 897D B4        MOV DWORD PTR SS:[EBP-4C],EDI
00459A1D   . 897D A4        MOV DWORD PTR SS:[EBP-5C],EDI
00459A20   . 897D 94        MOV DWORD PTR SS:[EBP-6C],EDI
00459A23   . 897D 84        MOV DWORD PTR SS:[EBP-7C],EDI
00459A26   . 89BD 74FFFFFF  MOV DWORD PTR SS:[EBP-8C],EDI
00459A2C   . 89BD 70FFFFFF  MOV DWORD PTR SS:[EBP-90],EDI
00459A32   . 89BD 6CFFFFFF  MOV DWORD PTR SS:[EBP-94],EDI
00459A38   . FFD3           CALL EBX                                           ;  <&MSVBVM60.__vbaStrCopy>
00459A3A   . BA 90B74100    MOV EDX,unpacked.0041B790                          ;  UNICODE "SoftWare\DDZ\Infomation"
00459A3F   . 8D4D C4        LEA ECX,DWORD PTR SS:[EBP-3C]
00459A42   . FFD3           CALL EBX
00459A44   . 8D45 C0        LEA EAX,DWORD PTR SS:[EBP-40]
00459A47   . 50             PUSH EAX                                   "regcode"入栈
00459A48   . 8D4D C4        LEA ECX,DWORD PTR SS:[EBP-3C]
00459A4B   . 51             PUSH ECX                                   ;"SoftWare\DDZ\Infomation"入栈
00459A4C   . 8D95 6CFFFFFF  LEA EDX,DWORD PTR SS:[EBP-94]
00459A52   . 52             PUSH EDX
00459A53   . C785 6CFFFFFF >MOV DWORD PTR SS:[EBP-94],80000002
00459A5D   . E8 0E7A0400    CALL unpacked.004A1470                ;调用一个通用子程序从注册表中读取注册码。
00459A62   . 8B35 98134000  MOV ESI,DWORD PTR DS:[<&MSVBVM60.__vbaStrMove>]    ;  MSVBVM60.__vbaStrMove
00459A68   . 8BD0           MOV EDX,EAX
00459A6A   . 8D4D CC        LEA ECX,DWORD PTR SS:[EBP-34]
00459A6F   . FFD6           CALL ESI                                           ;  <&MSVBVM60.__vbaStrMove>
00459A6F   . 8D45 C0        LEA EAX,DWORD PTR SS:[EBP-40]
```

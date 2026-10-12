
# Speed Video Converter 3.1.12 算法分析

> 原文：[`www.pediy.com/kssd/pediy10/73289.html`](https://www.pediy.com/kssd/pediy10/73289.html)

## 元信息

- **破文标题**：Speed Video Converter 3.1.12 算法分析
- **破文作者**：tianxj
- **作者邮箱**：`tianxj_2007@126.com`
- **作者主页**：WwW.ChiNaPYG.CoM
- **破解工具**：`PEiD`, `OD`
- **破解平台**：Windows XP
- **软件名称**：Speed Video Converter 3.1.12
- **软件大小**：2592KB
- **软件类别**：国外软件/视频转换
- **软件语言**：英文
- **更新时间**：2008-9-8
- **原版下载**：自己找一下
- **保护方式**：注册码

## 软件简介

Speed Video Converter 是一款小型视频转换工具。快速的转换速度和易用的用户界面是其卖点。Speed Video Converter 支持各种视频格式，例如 AVI(Divx, xDiv), MPEG-4, mpeg(vcd, svcd, dvd 兼容), wmv, asf, QuickTime, VOB, DAT。它支持批量文件转换可以一次点击转换多个文件。

## 破解声明

我是一只小菜鸟，偶得一点心得，愿与大家分享：）

## 破解内容

### 一、运行程序，进行注册

输入错误的注册信息进行检测，有提示信息：

> "Invalid username or registration code"

### 二、查壳

用 `PEiD` 对 `Speed Video Converter.exe` 查壳，为 Microsoft Visual C++ 6.0。

### 三、动态分析

运行 `OD`，打开 `Speed Video Converter.exe`，右键超级字串参考查找 ASCII。发现 "Invalid username or registration code"。

```
代码:
```



```
004030E0   .  6A FF            PUSH    -1
004030E2   .  68 E8244100     PUSH    Speed_Vi.004124E8                 ;  SE 处理程序安装
004030E7   .  64:A1 00000000  MOV     EAX, DWORD PTR FS:[0]
004030ED   .  50              PUSH    EAX
004030EE   .  64:8925 000000>MOV     DWORD PTR FS:[0], ESP
004030F5   .  83EC 08         SUB     ESP, 8
004030F8   .  56              PUSH    ESI
004030F9   .  8BF1            MOV     ESI, ECX
004030FB   .  8D4C24 04       LEA     ECX, DWORD PTR [ESP+4]
004030FF   .  E8 DEE40000     CALL    <JMP.&MFC42.#540_CString::CString>
00403104   .  6A 01           PUSH    1
00403106   .  8BCE            MOV     ECX, ESI
00403108   .  C74424 18 0000>MOV     DWORD PTR [ESP+18], 0
00403110   .  E8 8FE60000     CALL    <JMP.&MFC42.#6334_CWnd::UpdateData>
00403115   .  E8 14E70000     CALL    <JMP.&MFC42.#1168_AfxGetModuleState>
0040311A   .  8B48 04         MOV     ECX, DWORD PTR [EAX+4]
0040311D   .  E8 06E70000     CALL    <JMP.&MFC42.#1669_CCmdTarget::BeginWait>
00403122   .  8B46 60         MOV     EAX, DWORD PTR [ESI+60]           ;  //试练码
00403125   .  8B4E 64         MOV     ECX, DWORD PTR [ESI+64]           ;  //用户名
00403128   .  50              PUSH    EAX
00403129   .  51              PUSH    ECX
0040312A   .  C64424 1C 01    MOV     BYTE PTR [ESP+1C], 1
0040312F   .  E8 CCFDFFFF     CALL    Speed_Vi.00402F00                 ;  //关键 CALL
00403134   .  83C4 08         ADD     ESP, 8
00403137   .  85C0            TEST    EAX, EAX
00403139   .  0F95C0          SETNE   AL
0040313C   .  84C0            TEST    AL, AL
0040313E   .  A2 7CAF4100     MOV     BYTE PTR [41AF7C], AL
00403143   .  0F84 93000000  JE      Speed_Vi.004031DC                 ;  //关键跳转
00403149   .  8B46 64         MOV     EAX, DWORD PTR [ESI+64]
0040314C   .  8D4C24 04       LEA     ECX, DWORD PTR [ESP+4]
00403150   .  50              PUSH    EAX
00403151   .  68 40A24100     PUSH    Speed_Vi.0041A240                 ;  ASCII "License To:%s         "
00403156   .  51              PUSH    ECX
00403157   .  E8 C6E60000     CALL    <JMP.&MFC42.#2818_CString::Format>
0040315C   .  8B5424 10       MOV     EDX, DWORD PTR [ESP+10]
00403160   .  83C4 0C         ADD     ESP, 0C
00403163   .  8BCE            MOV     ECX, ESI
00403165   .  6A 40           PUSH    40
00403167   .  68 34A24100     PUSH    Speed_Vi.0041A234                 ;  ASCII "Thank you"
0040316C   .  52              PUSH    EDX
0040316D   .  E8 AAE60000     CALL    <JMP.&MFC42.#4224_CWnd::MessageBoxA>
00403172   .  8B46 64         MOV     EAX, DWORD PTR [ESI+64]
00403175   .  B9 88AE4100     MOV     ECX, Speed_Vi.0041AE88
0040317A   .  50              PUSH    EAX
0040317B   .  E8 90E8FFFF     CALL    Speed_Vi.00401A10
00403180   .  8B46 60         MOV     EAX, DWORD PTR [ESI+60]
00403183   .  B9 88AE4100     MOV     ECX, Speed_Vi.0041AE88
00403188   .  50              PUSH    EAX
00403189   .  E8 B2E8FFFF     CALL    Speed_Vi.00401A40
0040318E   .  8D4424 04       LEA     EAX, DWORD PTR [ESP+4]
00403192   .  8D4C24 08       LEA     ECX, DWORD PTR [ESP+8]
00403196   .  50              PUSH    EAX
00403197   .  68 1CA24100     PUSH    Speed_Vi.0041A21C                 ;  ASCII "Speed Video Converter "
0040319C   .  51              PUSH    ECX
0040319D   .  E8 74E60000     CALL    <JMP.&MFC42.#926_operator+>
004031A2   .  50              PUSH    EAX
004031A3   .  8D4C24 08       LEA     ECX, DWORD PTR [ESP+8]
004031A7   .  C64424 18 02    MOV     BYTE PTR [ESP+18], 2
004031AC   .  E8 4BE50000     CALL    <JMP.&MFC42.#858_CString::operator=>
004031B1   .  8D4C24 08       LEA     ECX, DWORD PTR [ESP+8]
004031B5   .  C64424 14 01    MOV     BYTE PTR [ESP+14], 1
004031BA   .  E8 11E40000     CALL    <JMP.&MFC42.#800_CString::~CString>
004031BF   .  E8 6AE60000     CALL    <JMP.&MFC42.#1168_AfxGetModuleState>
004031C4   .  8B40 04         MOV     EAX, DWORD PTR [EAX+4]
004031C7   .  8B5424 04       MOV     EDX, DWORD PTR [ESP+4]
004031CB   .  52              PUSH    EDX
004031CC   .  8BC8            MOV     ECX, EAX
004031CE   .  8B10            MOV     EDX, DWORD PTR [EAX]
004031D0   .  FF52 7C         CALL    DWORD PTR [EDX+7C]
004031D3   .  8BC8            MOV     ECX, EAX
004031D5   .  E8 36E60000     CALL    <JMP.&MFC42.#6199_CWnd::SetWindowTextA>
004031DA   .  EB 13           JMP     SHORT Speed_Vi.004031EF
004031DC   >  6A 40           PUSH    40
004031DE   .  68 14A24100     PUSH    Speed_Vi.0041A214                 ;  ASCII "Sorry"
004031E3   .  68 E8A14100     PUSH    Speed_Vi.0041A1E8                 ;  invalid username or registration code
004031E8   .  8BCE            MOV     ECX, ESI
004031EA   .  E8 2DE60000     CALL    <JMP.&MFC42.#4224_CWnd::MessageBoxA>
004031EF   >  8BCE            MOV     ECX, ESI
004031F1   .  E8 4EE50000     CALL    <JMP.&MFC42.#4853_CDialog::OnOK>
004031F6   .  C64424 14 00    MOV     BYTE PTR [ESP+14], 0
004031FB   .  E8 2EE60000     CALL    <JMP.&MFC42.#1168_AfxGetModuleState>
00403200   .  8B48 04         MOV     ECX, DWORD PTR [EAX+4]
00403203   .  E8 02E60000     CALL    <JMP.&MFC42.#2652_CCmdTarget::EndWaitCu>
00403208   .  8D4C24 04       LEA     ECX, DWORD PTR [ESP+4]
0040320C   .  C74424 14 FFFF>MOV     DWORD PTR [ESP+14], -1
00403214   .  E8 B7E30000     CALL    <JMP.&MFC42.#800_CString::~CString>
00403219   .  8B4C24 0C       MOV     ECX, DWORD PTR [ESP+C]
0040321D   .  5E              POP     ESI
0040321E   .  64:890D 000000>MOV     DWORD PTR FS:[0], ECX
00403225   .  83C4 14         ADD     ESP, 14
00403228   .  C3              RETN
\==============================================================
00402F00  /$  8B5424 04       MOV     EDX, DWORD PTR [ESP+4]
00402F04  |.  56              PUSH    ESI
00402F05  |.  57              PUSH    EDI
00402F06  |.  BF B0AE4100     MOV     EDI, Speed_Vi.0041AEB0
00402F0B  |.  8BF2            MOV     ESI, EDX
00402F0D  |.  B9 01000000     MOV     ECX, 1
00402F12  |.  33C0            XOR     EAX, EAX
00402F14  |.  F3:A6           REPE    CMPS BYTE PTR ES:[EDI], BYTE PTR [ESI]
00402F16  |.  74 2B           JE      SHORT Speed_Vi.00402F43           ;  //用户名为空则跳
00402F18  |.  8B4424 10       MOV     EAX, DWORD PTR [ESP+10]
00402F1C  |.  53              PUSH    EBX
00402F1D  |.  BF B0AE4100     MOV     EDI, Speed_Vi.0041AEB0
00402F22  |.  8BF0            MOV     ESI, EAX
00402F24  |.  B9 01000000     MOV     ECX, 1
00402F29  |.  33DB            XOR     EBX, EBX
00402F2B  |.  F3:A6           REPE    CMPS BYTE PTR ES:[EDI], BYTE PTR [ESI]
00402F2D  |.  5B              POP     EBX
00402F2E  |.  74 13           JE      SHORT Speed_Vi.00402F43           ;  //试练码为空则跳
00402F30  |.  50              PUSH    EAX
00402F31  |.  52              PUSH    EDX
00402F32  |.  E8 99FDFFFF     CALL    Speed_Vi.00402CD0                 ;  //关键 CALL
00402F37  |.  83C4 08         ADD     ESP, 8
00402F3A  |.  F7D8            NEG     EAX
00402F3C  |.  1BC0            SBB     EAX, EAX
00402F3E  |.  5F              POP     EDI
00402F3F  |.  F7D8            NEG     EAX
00402F41  |.  5E              POP     ESI
00402F42  |.  C3              RETN
00402F43  |>  5F              POP     EDI
00402F44  |.  33C0            XOR     EAX, EAX
00402F46  |.  5E              POP     ESI
00402F47  \\.  C3             RETN
\==============================================================
00402CD0  /$  6A FF            PUSH    -1
00402CD2  |.  68 50244100     PUSH    Speed_Vi.00412450                 ;  SE 处理程序安装
00402CD7  |.  64:A1 00000000  MOV     EAX, DWORD PTR FS:[0]
00402CDD  |.  50              PUSH    EAX
00402CDE  |.  64:8925 000000>MOV     DWORD PTR FS:[0], ESP
00402CE5  |.  83EC 14         SUB     ESP, 14
00402CE8  |.  8B4424 24       MOV     EAX, DWORD PTR [ESP+24]           ;  //用户名
00402CEC  |.  53              PUSH    EBX
00402CED  |.  55              PUSH    EBP
00402CEE  |.  56              PUSH    ESI
00402CEF  |.  57              PUSH    EDI
00402CF0  |.  50              PUSH    EAX
00402CF1  |.  8D4C24 18       LEA     ECX, DWORD PTR [ESP+18]
00402CF5  |.  E8 0EEA0000     CALL    <JMP.&MFC42.#537_CString::CString>
00402CFA  |.  8D4C24 14       LEA     ECX, DWORD PTR [ESP+14]
00402CFE  |.  C74424 2C 0000>MOV     DWORD PTR [ESP+2C], 0
00402D06  |.  E8 F9EA0000     CALL    <JMP.&MFC42.#6282_CString::TrimLeft>
00402D0B  |.  8D4C24 14       LEA     ECX, DWORD PTR [ESP+14]
00402D0F  |.  E8 EAEA0000     CALL    <JMP.&MFC42.#6283_CString::TrimRight>
00402D14  |.  6A 20           PUSH    20
00402D16  |.  8D4C24 18       LEA     ECX, DWORD PTR [ESP+18]
00402D1A  |.  E8 D9EA0000     CALL    <JMP.&MFC42.#2915_CString::GetBuffer>
00402D1F  |.  8B4C24 38       MOV     ECX, DWORD PTR [ESP+38]           ;  //试练码
00402D23  |.  8BD8            MOV     EBX, EAX
00402D25  |.  51              PUSH    ECX
00402D26  |.  8D4C24 14       LEA     ECX, DWORD PTR [ESP+14]
00402D2A  |.  E8 D9E90000     CALL    <JMP.&MFC42.#537_CString::CString>
00402D2F  |.  8D4C24 10       LEA     ECX, DWORD PTR [ESP+10]
00402D33  |.  C64424 2C 01    MOV     BYTE PTR [ESP+2C], 1
00402D38  |.  E8 C7EA0000     CALL    <JMP.&MFC42.#6282_CString::TrimLeft>
00402D3D  |.  8D4C24 10       LEA     ECX, DWORD PTR [ESP+10]
00402D41  |.  E8 B8EA0000     CALL    <JMP.&MFC42.#6283_CString::TrimRight>
00402D46  |.  6A 20           PUSH    20
00402D48  |.  8D4C24 14       LEA     ECX, DWORD PTR [ESP+14]
00402D4C  |.  E8 A7EA0000     CALL    <JMP.&MFC42.#2915_CString::GetBuffer>
00402D51  |.  8BD0            MOV     EDX, EAX
00402D53  |.  83CE FF         OR      ESI, FFFFFFFF
00402D56  |.  8BFA            MOV     EDI, EDX
00402D58  |.  8BCE            MOV     ECX, ESI
00402D5A  |.  33C0            XOR     EAX, EAX
00402D5C  |.  895424 20       MOV     DWORD PTR [ESP+20], EDX
00402D60  |.  F2:AE           REPNE   SCAS BYTE PTR ES:[EDI]
00402D62  |.  F7D1            NOT     ECX
00402D64  |.  49              DEC     ECX
00402D65  |.  8BFB            MOV     EDI, EBX
00402D67  |.  8BE9            MOV     EBP, ECX
00402D69  |.  8BCE            MOV     ECX, ESI
00402D6B  |.  F2:AE           REPNE   SCAS BYTE PTR ES:[EDI]
00402D6D  |.  F7D1            NOT     ECX
00402D6F  |.  49              DEC     ECX
00402D70  |.  3BCD            CMP     ECX, EBP
00402D72  |.  0F87 54010000  JA      Speed_Vi.00402ECC                 ;  //用户名长度大于试练码的长度则跳
00402D78  |.  8BFB            MOV     EDI, EBX
00402D7A  |.  8BCE            MOV     ECX, ESI
00402D7C  |.  F2:AE           REPNE   SCAS BYTE PTR ES:[EDI]
00402D7E  |.  F7D1            NOT     ECX
00402D80  |.  49              DEC     ECX
00402D81  |.  0F84 45010000  JE      Speed_Vi.00402ECC                 ;  //用户名为空则跳
00402D87  |.  8BFA            MOV     EDI, EDX
00402D89  |.  8BCE            MOV     ECX, ESI
00402D8B  |.  F2:AE           REPNE   SCAS BYTE PTR ES:[EDI]
00402D8D  |.  F7D1            NOT     ECX
00402D8F  |.  49              DEC     ECX
00402D90  |.  0F84 36010000  JE      Speed_Vi.00402ECC                 ;  //注册码为空则跳
00402D96  |.  894424 38       MOV     DWORD PTR [ESP+38], EAX
00402D9A  |>  8B5424 38      /MOV     EDX, DWORD PTR [ESP+38]
00402D9E  |.  8D4C24 34      |LEA     ECX, DWORD PTR [ESP+34]
00402DA2  |.  8A82 D0A14100  |MOV     AL, BYTE PTR [EDX+41A1D0]        ;  //'I'、'p'、'q',在密码表中未查到字符时用
00402DA8  |.  884424 18      |MOV     BYTE PTR [ESP+18], AL
00402DAC  |.  E8 31E80000    |CALL    <JMP.&MFC42.#540_CString::CString>
00402DB1  |.  8BFB            |MOV     EDI, EBX
00402DB3  |.  83C9 FF         |OR      ECX, FFFFFFFF
00402DB6  |.  33C0            |XOR     EAX, EAX
00402DB8  |.  33ED            |XOR     EBP, EBP
00402DBA  |.  F2:AE           |REPNE   SCAS BYTE PTR ES:[EDI]
00402DBC  |.  F7D1            |NOT     ECX
00402DBE  |.  49              |DEC     ECX
00402DBF  |.  C64424 2C 02   |MOV     BYTE PTR [ESP+2C], 2
00402DC4  |.  74 4B           |JE      SHORT Speed_Vi.00402E11
00402DC6  |>  8A042B         |/MOV     AL, BYTE PTR [EBX+EBP]           ;  //依次取用户名 ASC 值
00402DC9  |.  33F6           ||XOR     ESI, ESI
00402DCB  |>  3A0475 68A1410>||/CMP     AL, BYTE PTR [ESI*2+41A168]      ;  //隔位取密码

表"aqbicHdSeXfMgkhEiVjZkelRmynBoKpdqTrAsFtWulvjwDxIyPzzAxBpCODkEgFYGmHtIaJrKQLnMsNuOUPGQJRLSNTbUcVfWhXoYwZC"
00402DD2  |.  74 08          |||JE      SHORT Speed_Vi.00402DDC
00402DD4  |.  46             |||INC     ESI
00402DD5  |.  83FE 34        |||CMP     ESI, 34
00402DD8  |.^ 7C F1          ||\\JL      SHORT Speed_Vi.00402DCB
00402DDA  |.  EB 11          ||JMP     SHORT Speed_Vi.00402DED
00402DDC  |>  8A0C75 69A1410>||MOV     CL, BYTE PTR [ESI*2+41A169]
00402DE3  |.  51             ||PUSH    ECX
00402DE4  |.  8D4C24 38      ||LEA     ECX, DWORD PTR [ESP+38]
00402DE8  |.  E8 05EA0000    ||CALL    <JMP.&MFC42.#940_CString::operator+= >
00402DED  |>  83FE 34        ||CMP     ESI, 34
00402DF0  |.  75 0E          ||JNZ     SHORT Speed_Vi.00402E00
00402DF2  |.  8B5424 18      ||MOV     EDX, DWORD PTR [ESP+18]
00402DF6  |.  8D4C24 34      ||LEA     ECX, DWORD PTR [ESP+34]
00402DFA  |.  52             ||PUSH    EDX
00402DFB  |.  E8 F2E90000    ||CALL    <JMP.&MFC42.#940_CString::operator+= >
00402E00  |>  8BFB           ||MOV     EDI, EBX
00402E02  |.  83C9 FF        ||OR      ECX, FFFFFFFF
00402E05  |.  33C0           ||XOR     EAX, EAX
00402E07  |.  45             ||INC     EBP
00402E08  |.  F2:AE          ||REPNE   SCAS BYTE PTR ES:[EDI]
00402E0A  |.  F7D1           ||NOT     ECX
00402E0C  |.  49             ||DEC     ECX
00402E0D  |.  3BE9           ||CMP     EBP, ECX
00402E0F  |.^ 72 B5          |\\JB      SHORT Speed_Vi.00402DC6           ;  //循环,对比正确后取值
00402E11  |>  8B4424 34      |MOV     EAX, DWORD PTR [ESP+34]
00402E15  |.  8B48 F8        |MOV     ECX, DWORD PTR [EAX-8]
00402E18  |.  83F9 10        |CMP     ECX, 10
00402E1B  |.  7D 3A          |JGE     SHORT Speed_Vi.00402E57
00402E1D  |.  8BC1          |MOV     EAX, ECX
00402E1F  |.  B9 10000000    |MOV     ECX, 10
00402E24  |.  2BC8          |SUB     ECX, EAX
00402E26  |.  8D5424 1C      |LEA     EDX, DWORD PTR [ESP+1C]
00402E2A  |.  51             |PUSH    ECX
00402E2B  |.  52             |PUSH    EDX
00402E2C  |.  B9 B4AE4100    |MOV     ECX, Speed_Vi.0041AEB4             ;  ASCII "Hn<"
00402E31  |.  E8 B6E90000    |CALL    <JMP.&MFC42.#4129_CString::Left>
00402E36  |.  50             |PUSH    EAX
00402E37  |.  8D4C24 38      |LEA     ECX, DWORD PTR [ESP+38]
00402E3B  |.  C64424 30 03   |MOV     BYTE PTR [ESP+30], 3
00402E40  |.  E8 A1E90000    |CALL    <JMP.&MFC42.#939_CString::operator+= >
00402E45  |.  8D4C24 1C      |LEA     ECX, DWORD PTR [ESP+1C]
00402E49  |.  C64424 2C 02   |MOV     BYTE PTR [ESP+2C], 2
00402E4E  |.  E8 7DE70000    |CALL    <JMP.&MFC42.#800_CString::~CString>
00402E53  |.  8B4424 34      |MOV     EAX, DWORD PTR [ESP+34]            ;  //真码
00402E57  |>  8B4C24 20      |MOV     ECX, DWORD PTR [ESP+20]            ;  //试练码
00402E5B  |.  51             |PUSH    ECX                                ; /s2
00402E5C  |.  50             |PUSH    EAX                                ; |s1
00402E5D  |.  FF15 00464100  |CALL    DWORD PTR [<&MSVCRT._mbscmp>]       ; \\//比较
00402E63  |.  83C4 08        |ADD     ESP, 8
00402E66  |.  8D4C24 34      |LEA     ECX, DWORD PTR [ESP+34]
00402E6A  |.  85C0           |TEST    EAX, EAX
00402E6C  |.  C64424 2C 01   |MOV     BYTE PTR [ESP+2C], 1               ;  //标志位赋值
00402E71  |.  74 1B          |JE      SHORT Speed_Vi.00402E8E
00402E73  |.  33F6          |XOR     ESI, ESI
00402E75  |.  E8 56E70000    |CALL    <JMP.&MFC42.#800_CString::~CString>
00402E7A  |.  8B4424 38      |MOV     EAX, DWORD PTR [ESP+38]
00402E7E  |.  40             |INC     EAX
00402E7F  |.  83F8 03        |CMP     EAX, 3
00402E82  |.  894424 38      |MOV     DWORD PTR [ESP+38], EAX
00402E86  |.^ 0F8C 0EFFFFFF  \\JL      Speed_Vi.00402D9A                 ;  //循环,比较 3 次
00402E8C  |.  EB 0A          |JMP     SHORT Speed_Vi.00402E98
00402E8E  |>  BE 01000000   |MOV     ESI, 1
00402E93  |.  E8 38E70000    |CALL    <JMP.&MFC42.#800_CString::~CString>
00402E98  |>  8D4C24 10      |LEA     ECX, DWORD PTR [ESP+10]
00402E9C  |.  C64424 2C 00   |MOV     BYTE PTR [ESP+2C], 0
00402EA1  |.  E8 2AE70000    |CALL    <JMP.&MFC42.#800_CString::~CString>
00402EA6  |.  8D4C24 14      |LEA     ECX, DWORD PTR [ESP+14]
00402EAA  |.  C74424 2C FFFF>MOV     DWORD PTR [ESP+2C], -1
00402EB2  |.  E8 19E70000    |CALL    <JMP.&MFC42.#800_CString::~CString>
00402EB7  |.  8BC6           |MOV     EAX, ESI
00402EB9  |.  5F             |POP     EDI
00402EBA  |.  5E             |POP     ESI
00402EBB  |.  5D             |POP     EBP
00402EBC  |.  5B             |POP     EBX
00402EBD  |.  8B4C24 14      |MOV     ECX, DWORD PTR [ESP+14]
00402EC1  |.  64:890D 000000>MOV     DWORD PTR FS:[0], ECX
00402EC8  |.  83C4 20        |ADD     ESP, 20
00402ECB  |.  C3             |RETN
00402ECC  |>  8D4C24 10      |LEA     ECX, DWORD PTR [ESP+10]
00402ED0  |.  C64424 2C 00   |MOV     BYTE PTR [ESP+2C], 0
00402ED5  |.  E8 F6E60000    |CALL    <JMP.&MFC42.#800_CString::~CString>
00402EDA  |.  8D4C24 14      |LEA     ECX, DWORD PTR [ESP+14]
00402EDE  |.  897424 2C      |MOV     DWORD PTR [ESP+2C], ESI
00402EE2  |.  E8 E9E60000    |CALL    <JMP.&MFC42.#800_CString::~CString>
00402EE7  |.  8B4C24 24      |MOV     ECX, DWORD PTR [ESP+24]
00402EEB  |.  5F             |POP     EDI
00402EEC  |.  5E             |POP     ESI
00402EED  |.  5D             |POP     EBP
00402EEE  |.  33C0           |XOR     EAX, EAX
00402EF0  |.  5B             |POP     EBX
00402EF1  |.  64:890D 000000>MOV     DWORD PTR FS:[0], ECX
00402EF8  |.  83C4 20        |ADD     ESP, 20
00402EFB  \\.  C3            |RETN
\==============================================================
密码表
0041A168  61 71 62 69 63 48 64 53 65 58 66 4D 67 6B 68 45  aqbicHdSeXfMgkhE
0041A178  69 56 6A 5A 6B 65 6C 52 6D 79 6E 42 6F 4B 70 64  iVjZkelRmynBoKpd
0041A188  71 54 72 41 73 46 74 57 75 6C 76 6A 77 44 78 49  qTrAsFtWulvjwDxI
0041A198  79 50 7A 7A 41 78 42 70 43 4F 44 6B 45 67 46 59  yPzzAxBpCODkEgFY
0041A1A8  47 6D 48 74 49 61 4A 72 4B 51 4C 6E 4D 73 4E 75  GmHtIaJrKQLnMsNu
0041A1B8  4F 55 50 47 51 4A 52 4C 53 4E 54 62 55 63 56 66  OUPGQJRLSNTbUcVf
0041A1C8  57 68 58 6F 59 77 5A 43                       WhXoYwZC
```



# 破解总结

--------------------------------------------------------------

# 算法总结

1. 将用户名的各个字符进行替换

   替换规则如下：

   (1) 英文大小写字母进行查表
      密码表 `"aqbicHdSeXfMgkhEiVjZkelRmynBoKpdqTrAsFtWulvjwDxIyPzzAxBpCODkEgFYGmHtIaJrKQLnMsNuOUPGQJRLSNTbUcVfWhXoYwZC"`

      找相应字符后取紧接着的下一个字符

   (2) 非英文字母则统一替换为 `'I'` 或 `'p'` 或 `'q'`

2. 若替换后的长度小于 `16` 位，则用 `"IsaeBREbmpaqdslg"` 补充到 `16` 位

--------------------------------------------------------------

# 算法注册机

## 易语言代码

```
.版本 2

.子程序 _ 按钮 1_ 被单击

.局部变量 M, 文本型
.局部变量 i, 整数型
.局部变量 j, 整数型
.局部变量 a, 文本型
.局部变量 b, 整数型
.局部变量 c, 文本型
.局部变量 x, 文本型
.局部变量 y, 文本型
.局部变量 z, 文本型

M ＝ “aqbicHdSeXfMgkhEiVjZkelRmynBoKpdqTrAsFtWulvjwDxIyPzzAxBpCODkEgFYGmHtIaJrKQLnMsNuOUPGQJRLSNTbUcVfWhXoYwZC”

.如果 (取文本长度 (编辑框 1.内容) ＝ 0)
    编辑框 2.内容 ＝ “请输入用户名”
.否则
    .计次循环首 (取文本长度 (M) ÷ 2, i)
        x ＝ x ＋ 取文本中间 (M, 2 × i － 1, 1)
        y ＝ y ＋ 取文本中间 (M, 2 × i, 1)
    .计次循环尾 ()
    .计次循环首 (取文本长度 (编辑框 1.内容), j)
        a ＝ 取文本中间 (编辑框 1.内容, j, 1)
        b ＝ 寻找文本 (x, a, 1, 假)
        c ＝ c ＋ 取文本中间 (y, b, 1)
    .计次循环尾 ()
    编辑框 2.内容 ＝ 取文本左边 (c ＋ “IsaeBREbmpaqdslg”, 16)
.如果结束
```

--------------------------------------------------------------

☆ 此文应学友 `de 神` 之邀而作 ☆

--------------------------------------------------------------

感谢 `飘云老大`、`猫老大`、`Nisy 老大` 以及很多前辈们的学习教程以及所有帮助过我的论坛兄弟姐妹们！谢谢

--------------------------------------------------------------

## 版权声明

破文是学习的手记，兴趣是成功的源泉；本破文纯属技术交流，转载请注明作者并保持文章的完整，谢谢!


# 证件大师 V2.5 的简单注册

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-289.htm`](https://www.pediy.com/kssd/pediy07/pediy7-289.htm)

【破文作者】 rdsnow[BCG][PYG][D.4s]

【作者主页】 [`rdsnow.ys168.com`](http://rdsnow.ys168.com)

【 E-mail 】 rdsnow@163.com

【 作者 QQ 】 83757177

【文章题目】 证件大师 V2.5 的简单注册

【软件名称】 证件大师 V2.5

【下载地址】 [`www3.skycn.com/soft/1093.html`](http://www3.skycn.com/soft/1093.html)

----------------------------------------------------------------------------------------------

【加密方式】 序列号

【破解工具】 `FlyOD V1.10`

【软件限制】 功能限制

【破解平台】 `Microsoft Windows XP SP2`

----------------------------------------------------------------------------------------------

【文章简介】

重启验证注册码的程序，用 `RegMon` 很容易找到注册码保存的地方。

```
[HKEY_CURRENT_USER\Software\RegInfo]

"RegCode"="987654abcd"
"RegDate"="a"
```

注册码的计算过程用了浮点指令，略微增加了分析的难度。

该程序有使用 10 次的限制，即使注册码正确，没有过期时，程序不进行注册码的校验，所以要来到校验的地方，首先得让程序过期。

----------------------------------------------------------------------------------------------

【破解过程】

搜索字符串 `"RegInfo"` 来到这里：

```
00412BE4   .  C645 FC 01         MOV BYTE PTR SS:[EBP-4],1
00412BE8   .  C745 DC 0A000000  MOV DWORD PTR SS:[EBP-24],0A
00412BEF   .  C745 D4 01000000  MOV DWORD PTR SS:[EBP-2C],1
00412BF6   .  68 844C4500       PUSH LabelEdi.00454C84                ;  ASCII "SoftWare\RegInfo"
00412BFB   .  8D4D E4           LEA ECX,DWORD PTR SS:[EBP-1C]
00412BFE   .  E8 76EDFEFF       CALL LabelEdi.00401979
00412C03   .  8945 E0          MOV DWORD PTR SS:[EBP-20],EAX
00412C06   .  837D E0 00       CMP DWORD PTR SS:[EBP-20],0
00412C0A   .  75 3E            JNZ SHORT LabelEdi.00412C4A
00412C0C   .  68 784C4500      PUSH LabelEdi.00454C78                ;  ASCII "SoftWare"
00412C11   .  8D4D E4          LEA ECX,DWORD PTR SS:[EBP-1C]
00412C14   .  E8 60EDFEFF      CALL LabelEdi.00401979
00412C19   .  68 6C4C4500      PUSH LabelEdi.00454C6C                ;  ASCII "RegInfo"
00412C1E   .  8D4D E4          LEA ECX,DWORD PTR SS:[EBP-1C]
00412C21   .  E8 BEEBFEFF     CALL LabelEdi.004017E4
00412C26   .  6A 0A            PUSH 0A
00412C28   .  68 604C4500     PUSH LabelEdi.00454C60                ;  ASCII "RegDate"
00412C2D   .  8D4D E4         LEA ECX,DWORD PTR SS:[EBP-1C]
00412C30   .  E8 C1E8FEFF     CALL LabelEdi.004014F6
00412C35   .  8D4D E4         LEA ECX,DWORD PTR SS:[EBP-1C]
00412C38   .  E8 53EAFEFF     CALL LabelEdi.00401690
00412C3D   .  68 844C4500     PUSH LabelEdi.00454C84                ;  ASCII "SoftWare\RegInfo"
00412C42   .  8D4D E4         LEA ECX,DWORD PTR SS:[EBP-1C]
00412C45   .  E8 2FEDFEFF    CALL LabelEdi.00401979
00412C4A   >  8D45 EC          LEA EAX,DWORD PTR SS:[EBP-14]
00412C4D   .  50               PUSH EAX
00412C4E   .  68 2C4A4500      PUSH LabelEdi.00454A2C                ;  ASCII "RegCode"
00412C53   .  8D4D E4          LEA ECX,DWORD PTR SS:[EBP-1C]
00412C56   .  E8 45E9FEFF      CALL LabelEdi.004015A0                ;  读取存放在注册表中的假注册码
00412C5B   .  8945 E0          MOV DWORD PTR SS:[EBP-20],EAX
00412C5E   .  837D E0 00       CMP DWORD PTR SS:[EBP-20],0
00412C62   .  0F85 C8000000    JNZ LabelEdi.00412D30                 ;  读取成功会跳走继续验证注册码
00412C68   .  8B4D F0          MOV ECX,DWORD PTR SS:[EBP-10]
00412C6B   .  C781 0C010000 0000>MOV DWORD PTR DS:[ECX+10C],0
00412C75   .  8D55 D4          LEA EDX,DWORD PTR SS:[EBP-2C]
00412C78   .  52               PUSH EDX
00412C79   .  68 604C4500      PUSH LabelEdi.00454C60                ;  ASCII "RegDate"
00412C7E   .  8D4D E4          LEA ECX,DWORD PTR SS:[EBP-1C]
00412C81   .  E8 8DE4FEFF      CALL LabelEdi.00401113
00412C86   .  8945 E0          MOV DWORD PTR SS:[EBP-20],EAX
00412C89   .  8B45 D4          MOV EAX,DWORD PTR SS:[EBP-2C]
00412C8C   .  83E8 01          SUB EAX,1
00412C8F   .  8945 D4          MOV DWORD PTR SS:[EBP-2C],EAX
00412C92   .  837D D4 00       CMP DWORD PTR SS:[EBP-2C],0
00412C96   .  7F 1D            JG SHORT LabelEdi.00412CB5
00412C98   .  6A 00            PUSH 0
00412C9A   .  6A 00            PUSH 0
00412C9C   .  68 384C4500      PUSH LabelEdi.00454C38
00412CA1   .  E8 E0C50100      CALL <JMP.&MFC42D.#1136>
00412CA6   .  8B4D F0          MOV ECX,DWORD PTR SS:[EBP-10]
00412CA9   .  C781 0C010000 0100>MOV DWORD PTR DS:[ECX+10C],1
00412CB3   .  EB 43            JMP SHORT LabelEdi.00412CF8
00412CB5   >  8B55 D4          MOV EDX,DWORD PTR SS:[EBP-2C]
00412CB8   .  52               PUSH EDX
00412CB9   .  68 604C4500      PUSH LabelEdi.00454C60                ;  ASCII "RegDate"
00412CBE   .  8D4D E4          LEA ECX,DWORD PTR SS:[EBP-1C]
00412CC1   .  E8 30E8FEFF      CALL LabelEdi.004014F6
00412CC6   .  8BF4             MOV ESI,ESP
00412CC8   .  8B45 D4          MOV EAX,DWORD PTR SS:[EBP-2C]
00412CCB   .  50               PUSH EAX                               ;  /<%i>
00412CCC   .  68 0C4C4500      PUSH LabelEdi.00454C0C                ;  |format = "该软件还有%i 次使用限制，请及时注册!"
00412CD1   .  8D8D 6CFFFFFF    LEA ECX,DWORD PTR SS:[EBP-94]          ;  |
00412CD7   .  51               PUSH ECX                               ;  |s
00412CD8   .  FF15 D4F64500    CALL DWORD PTR DS:[<&MSVCRTD.sprintf>>;  \sprintf
00412CDE   .  83C4 0C          ADD ESP,0C
00412CE1   .  3BF4             CMP ESI,ESP
00412CE3   .  E8 AACE0100      CALL <JMP.&MSVCRTD._chkesp>
00412CE8   .  6A 00            PUSH 0
00412CEA   .  6A 00            PUSH 0
00412CEC   .  8D95 6CFFFFFF    LEA EDX,DWORD PTR SS:[EBP-94]
00412CF2   .  52               PUSH EDX
00412CF3   .  E8 8EC50100      CALL <JMP.&MFC42D.#1136>
00412CF8   >  8D4D E4          LEA ECX,DWORD PTR SS:[EBP-1C]
00412CFB   .  E8 90E9FEFF      CALL LabelEdi.00401690
00412D00   .  C785 E0FAFFFF 0000>MOV DWORD PTR SS:[EBP-520],0
00412D0A   .  C645 FC 00       MOV BYTE PTR SS:[EBP-4],0
00412D0E   .  8D4D E4          LEA ECX,DWORD PTR SS:[EBP-1C]
00412D11   .  E8 D1E7FEFF      CALL LabelEdi.004014E7
00412D16   .  C745 FC FFFFFFFF MOV DWORD PTR SS:[EBP-4],-1
00412D1D   .  8D4D EC          LEA ECX,DWORD PTR SS:[EBP-14]
00412D20   .  E8 01C50100      CALL <JMP.&MFC42D.#684>
00412D25   .  8B85 E0FAFFFF    MOV EAX,DWORD PTR SS:[EBP-520]
00412D2B   .  E9 AF010000      JMP LabelEdi.00412EDF
00412D30   >  8D4D E4          LEA ECX,DWORD PTR SS:[EBP-1C]
00412D33   .  E8 58E9FEFF      CALL LabelEdi.00401690
00412D38   .  8D8D 4CFBFFFF    LEA ECX,DWORD PTR SS:[EBP-4B4]
00412D3E   .  E8 81ECFEFF      CALL LabelEdi.004019C4
00412D43   .  C645 FC 02       MOV BYTE PTR SS:[EBP-4],2
00412D47   .  8D8D 50FFFFFF    LEA ECX,DWORD PTR SS:[EBP-B0]
00412D4D   .  E8 46C50100      CALL <JMP.&MFC42D.#2640>
00412D52   .  8985 E4FAFFFF    MOV DWORD PTR SS:[EBP-51C],EAX
00412D58   .  8BF4             MOV ESI,ESP
00412D5A   .  8B85 50FFFFFF    MOV EAX,DWORD PTR SS:[EBP-B0]
00412D60   .  50               PUSH EAX                               ;  /<%s>
00412D61   .  68 E8474500      PUSH LabelEdi.004547E8                ;  |format = "%s"
00412D66   .  8D8D E8FAFFFF    LEA ECX,DWORD PTR SS:[EBP-518]         ;  |
00412D6C   .  51               PUSH ECX                               ;  |s
00412D6D   .  FF15 D4F64500    CALL DWORD PTR DS:[<&MSVCRTD.sprintf>>;  \sprintf
00412D73   .  83C4 0C          ADD ESP,0C
00412D76   .  3BF4             CMP ESI,ESP
00412D78   .  E8 15CE0100      CALL <JMP.&MSVCRTD._chkesp>
00412D7D   .  C745 D4 00000000 MOV DWORD PTR SS:[EBP-2C],0           ;  设 i=[EBP-2C]，并且初始为 0，准备循环
00412D84   .  EB 09            JMP SHORT LabelEdi.00412D8F
```

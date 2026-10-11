
# HostScan v1.24

> 原文：[`www.pediy.com/kssd/pediy06/pediy6163.htm`](https://www.pediy.com/kssd/pediy06/pediy6163.htm)

## 软件信息

**软件名称**：`HostScan v1.24`

**主要功能**：`HostScan` include two aspects: IP Search (Scan), Port Search (Scan). `HostScan` software is used to find if host, server or router on netware is running well, and is connected to net. `HostScan` can help you get some information about the host, server or router you find on net, for example: domain name, time you spend to find it, TTL, and so on. `HostScan` also can be used to find which port on some computer is opened.

**未注册版本**：有对话框干扰。

**破解工具**：`W32Dasm`、`OllyDbg`

## 破解过程

软件未加壳，用 `W32Dasm` 反汇编，查找字串 `"Key is Wrong"`，再用 `OllyDbg` 动态跟踪。

输入用户名 `coldeye` 假注册码 `787878`

```assembly
0041400A    > 8B6C24 78               MOV EBP,DWORD PTR SS:[ESP+78]
0041400E    . 8B1D 44124200           MOV EBX,DWORD PTR DS:[<&USER32.SendDlgItemMessageA>]      ;  USER32.SendDlgItemMessageA
00414014    . 68 AC9D4300            PUSH hostscan.00439DAC                                   ; /lParam = 439DAC
00414019    . 6A 14                  PUSH 14                                                  ; |wParam = 14
0041401B    . 6A 0D                  PUSH D                                                   ; |Message = WM_GETTEXT
0041401D    . 68 EC030000            PUSH 3EC                                                 ; |ControlID = 3EC (1004.)
00414022    . 55                     PUSH EBP                                                 ; |hWnd
00414023    . FFD3                   CALL EBX                                                 ; SendDlgItemMessageA
00414025    . BF AC9D4300            MOV EDI,hostscan.00439DAC                                ;   ASCII "coldeye"   EDI=用户名 EAX=ECX=用户名长度=7
0041402A    . 83C9 FF                OR ECX,FFFFFFFF
0041402D    . 33C0                   XOR EAX,EAX
0041402F    . F2:AE                  REPNE SCAS BYTE PTR ES:[EDI]
00414031    . F7D1                   NOT ECX
00414033    . 49                     DEC ECX                                                  ECX=用户名长度=7
00414034    . 75 43                  JNZ SHORT hostscan.00414079                              用户名长度不为 0 跳转
00414036    . 68 A8A24200            PUSH hostscan.0042A2A8                                   ; /IniFileName = "E:Program FilesHostScanlanguagehseng.ini"
0041403B    . 68 C8000000            PUSH C8                                                  ; |BufSize = C8 (200.)
00414040    . 68 70A34200            PUSH hostscan.0042A370                                   ; |ReturnBuffer = hostscan.0042A370
00414045    . 68 BC8A4200            PUSH hostscan.00428ABC                                   ; |Default = "Please input username"
0041404A    . 68 AC8A4200            PUSH hostscan.00428AAC                                   ; |Key = "InputUserName"
0041404F    . 68 90634200            PUSH hostscan.00426390                                   ; |Section = "Run"
00414054    . FF15 9C104200          CALL DWORD PTR DS:[<&KERNEL32.GetPrivateProfileStringA>]  ; GetPrivateProfileStringA
0041405A    . 6A 40                  PUSH 40                                                  ; /Style = MB_OK|MB_ICONASTERISK|MB_APPLMODAL
0041405C    . 68 E8784200            PUSH hostscan.004278E8                                   ; |Title = "Tip"
00414061    . 68 70A34200            PUSH hostscan.0042A370                                   ; |Text = "Key is Wrong,Please check the key!!!"
00414066    . 55                     PUSH EBP                                                 ; |hOwner
00414067    . FF15 3C124200          CALL DWORD PTR DS:[<&USER32.MessageBoxA>]                ; MessageBoxA
0041406D    . 5F                     POP EDI
0041406E    . 5E                     POP ESI
0041406F    . 5D                     POP EBP
00414070    . 33C0                   XOR EAX,EAX
00414072    . 5B                     POP EBX
00414073    . 83C4 64                ADD ESP,64
00414076    . C2 1000                RETN 10
00414079    > 83C9 FF                OR ECX,FFFFFFFF
0041407C    . BF AC9D4300            MOV EDI,hostscan.00439DAC                                ;   ASCII "coldeye"
00414081    . 33C0                   XOR EAX,EAX
00414083    . 8B35 40124200          MOV ESI,DWORD PTR DS:[<&USER32.wsprintfA>]                ;   USER32.wsprintfA
00414089    . F2:AE                  REPNE SCAS BYTE PTR ES:[EDI]
0041408B    . F7D1                   NOT ECX                                                  ECX＝8(用户名长度＋1)
0041408D    . 83C1 FE                ADD ECX,-2                                               ECX=ECX-2=6
00414090    . BF AC9D4300            MOV EDI,hostscan.00439DAC                                ;   ASCII "coldeye"
00414095    . D1E9                   SHR ECX,1                                                ECX=h6(b110)右移 1 位=b11=h3
00414097    . 0FBE89 AC9D4300        MOVSX ECX,BYTE PTR DS:[ECX+439DAC]                      ECX=用户名右移 3 位'd'的 ASCII 值=h64
0041409E    . C1F9 06                SAR ECX,6                                                ECX=h64(b1100110)右移 6 位=b1=h1
004140A1    . 51                     PUSH ECX                                                 ; /<%x>=1            注册码第四部分的尾部入栈
004140A2    . 83C9 FF                OR ECX,FFFFFFFF                                          ; |ECX 清零
004140A5    . F2:AE                  REPNE SCAS BYTE PTR ES:[EDI]                             ; |
004140A7    . F7D1                   NOT ECX                                                  ; |ECX＝8(用户名长度＋1)
004140A9    . 83C1 FE                ADD ECX,-2                                               ; |ECX=ECX-2=6
004140AC    . BF AC9D4300            MOV EDI,hostscan.00439DAC                                ; |ASCII "coldeye"
004140B1    . 83E1 01                AND ECX,1                                                ; |ECX=110(6) AND 1(1) =b0=h0
004140B4    . 0FBE91 AC9D4300        MOVSX EDX,BYTE PTR DS:[ECX+439DAC]                       ; |EDX=用户名右移 0 位'c'的 ASCII 值=h63
004140BB    . 83C9 FF                OR ECX,FFFFFFFF                                          ; |ECX 清零
004140BE    . C1E2 03                SHL EDX,3                                                ; |EDX=h63(b1100011)左移 3 位=b1100011000=h318
004140C1    . F2:AE                  REPNE SCAS BYTE PTR ES:[EDI]                             ; |
004140C3    . F7D1                   NOT ECX                                                  ; |ECX＝8(用户名长度＋1)
004140C5    . 83C1 FE                ADD ECX,-2                                               ; |ECX=ECX-2=6
004140C8    . 52                     PUSH EDX                                                 ; |<%x>=318            注册码第四部分的首部入栈
004140C9    . C1E9 02                SHR ECX,2                                                ; |ECX=h6(b110)右移 2 位=b1=h1
004140CC    . BF AC9D4300            MOV EDI,hostscan.00439DAC                                ; |ASCII "coldeye"
004140D1    . 33D2                   XOR EDX,EDX                                              ; |EDX 清零
004140D3    . 0FBE81 AC9D4300        MOVSX EAX,BYTE PTR DS:[ECX+439DAC]                       ; |EAX=用户名右移 1 位'o'的 ASCII 值=h6F
004140DA    . C1F8 05                SAR EAX,5                                                ; |EAX=h6F(1101111)右移 5 位=b11=h3
```



```
004140DD    . 50                      PUSH EAX                                   ; |<%x>=3                 注册码第三部分的尾部入栈   

004140DE    . 83C9 FF                 OR ECX,FFFFFFFF                              ; |ECX 清零   

004140E1    . 33C0                    XOR EAX,EAX                                  ; |   

004140E3    . F2:AE                   REPNE SCAS BYTE PTR ES:[EDI]                ; |   

004140E5    . F7D1                    NOT ECX                                      ; |ECX＝8(用户名长度＋1)   

004140E7    . 83C1 FE                 ADD ECX,-2                                   ; |ECX=ECX-2=6   

004140EA    . BF AC9D4300             MOV EDI,hostscan.00439DAC                   ; |ASCII "coldeye"   

004140EF    . 83E1 03                 AND ECX,3                                    ; |ECX=110(6) AND 11(3)=b10=h2   

004140F2    . 0FBE89 AC9D4300         MOVSX ECX,BYTE PTR DS:[ECX+439DAC]          ; |ECX=用户名右移 2 位'l'的 ASCII 值=h6C   

004140F9    . C1E1 04                 SHL ECX,4                                    ; |ECX=h6C(b1101100)左移 4 位=h6C0(b11011000000)   

004140FC    . 51                      PUSH ECX                                   ; |<%x>=6C0               注册码第三部分的首部入栈   

004140FD    . 83C9 FF                 OR ECX,FFFFFFFF                              ; |   

00414100    . F2:AE                   REPNE SCAS BYTE PTR ES:[EDI]                ; |   

00414102    . F7D1                    NOT ECX                                      ; |ECX＝8(用户名长度＋1)   

00414104    . 49                      DEC ECX                                      ; |ECX=ECX-1=7   

00414105    . BF AC9D4300             MOV EDI,hostscan.00439DAC                   ; |ASCII "coldeye"         

0041410A    . 8BC1                    MOV EAX,ECX                                  ; |EAX=ECX=7             |无   

0041410C    . B9 03000000             MOV ECX,3                                    ; |ECX=3                 |用   

00414111    . 48                      DEC EAX                                      ; |EAX=EAX-1=6           |代   

00414112    . F7F1                    DIV ECX                                      ; |EAX=EAX/ECX=6/3=2     |码   

00414114    . 83C9 FF                 OR ECX,FFFFFFFF                              ; |   

00414117    . 33C0                    XOR EAX,EAX                                  ; |   

00414119    . 0FBE92 AC9D4300         MOVSX EDX,BYTE PTR DS:[EDX+439DAC]          ; |EDX=用户名右移 0 位'c'的 ASCII 值=h63   

00414120    . C1E2 03                 SHL EDX,3                                    ; |EDX=h63(b1100011)左移 3 位=h318(b1100011000)   

00414123    . F2:AE                   REPNE SCAS BYTE PTR ES:[EDI]                ; |   

00414125    . F7D1                    NOT ECX                                      ; |ECX＝8(用户名长度＋1)   

00414127    . 83C1 FE                 ADD ECX,-2                                   ; |ECX=ECX-2=6   

0041412A    . B8 ABAAAAAAAA           MOV EAX,AAAAAAAB                             ; |EAX=AAAAAAAB   

0041412F    . 52                      PUSH EDX                                   ; |<%x>=318              注册码第二部分的尾部入栈   

00414130    . F7E1                    MUL ECX                                      ; |进位乘法 AAAAAAAB*6=400000002 EAX=2 EDX=4   

00414132    . D1EA                    SHR EDX,1                                    ; |EDX=h4(b100)右移 1 位=h2(b10)   

00414134    . BF AC9D4300             MOV EDI,hostscan.00439DAC                   ; |ASCII "coldeye"   

00414139    . 83C9 FF                 OR ECX,FFFFFFFF                              ; |   

0041413C    . 0FBE82 AC9D4300         MOVSX EAX,BYTE PTR DS:[EDX+439DAC]          ; |EAX=用户名右移 2 位'l'的 ASCII 值=h6C   

00414143    . C1F8 06                 SAR EAX,6                                    ; |EAX=h6C(b1101100)右移 6 位=h1(b1)   

00414146    . 50                      PUSH EAX                                   ; |<%x>=1                 注册码第二部分的首部入栈   

00414147    . 33C0                    XOR EAX,EAX                                  ; |   

00414149    . F2:AE                   REPNE SCAS BYTE PTR ES:[EDI]                ; |   

0041414B    . 0FBE15 AC9D4300         MOVSX EDX,BYTE PTR DS:[439DAC]             ; |EDX=用户名首位'c'的 ASCII 值=h63   

00414152    . F7D1                    NOT ECX                                      ; |ECX＝8(用户名长度＋1)   

00414154    . 49                      DEC ECX                                      ; |ECX=ECX-1=7   

00414155    . 8D4424 30               LEA EAX,DWORD PTR SS:[ESP+30]               ; |   

00414159    . C1FA 04                 SAR EDX,4                                    ; |EDX=h63(b1100011)右移 4 位=h6(b110)   

0041415C    . 0FBE89 AB9D4300         MOVSX ECX,BYTE PTR DS:[ECX+439DAB]          ; |ECX=用户名末位'e'的 ASCII 值=h65   

00414163    . C1E1 05                 SHL ECX,5                                    ; |ECX=h65(b1100101)左移 5 位=hCA0(b110010100000)   

00414166    . 51                      PUSH ECX                                   ; |<%x>=CA0               注册码第一部分的尾部入栈   

00414167    . 52                      PUSH EDX                                   ; |<%x>=6                 注册码第一部分的首部入栈   

00414168    . 68 988A4200             PUSH hostscan.00428A98                      ; |Format = "%x%x-%x%x-%x%x-%x%x"注册码格式   

0041416D    . 50                      PUSH EAX                                   ; |s   

0041416E    . FFD6                    CALL ESI                                     ; wsprintfA   

00414170    . 83C4 28                 ADD ESP,28   

00414173    . 8D4C24 18               LEA ECX,DWORD PTR SS:[ESP+18]               

00414177    . 51                      PUSH ECX                                   ; /StringOrChar = "6ca0-1318-6c03-3181"   

00414178    . FF15 F8124200           CALL DWORD PTR DS:[<&USER32.CharUpperA>]     ; CharUpperA   

0041417E    . 50                      PUSH EAX                                       ASCII "6CA0-1318-6C03-3181"   

0041417F    . 8D5424 1C               LEA EDX,DWORD PTR SS:[ESP+1C]                  ASCII "6CA0-1318-6C03-3181"   

00414183    . 68 60664200             PUSH hostscan.00426660                      ;  ASCII "%s"   

00414188    . 52                      PUSH EDX   

00414189    . FFD6                    CALL ESI   

0041418B    . 83C4 0C                 ADD ESP,C   

0041418E    . 8D4424 2C               LEA EAX,DWORD PTR SS:[ESP+2C]               

00414192    . 50                      PUSH EAX   

00414193    . 6A 14                   PUSH 14   

00414195    . 6A 0D                   PUSH D   

00414197    . 68 F9030000             PUSH 3F9   

0041419C    . 55                      PUSH EBP   

0041419D    . FFD3                    CALL EBX   

0041419F    . 8D4C24 2C               LEA ECX,DWORD PTR SS:[ESP+2C]               

004141A3    . 8D5424 18               LEA EDX,DWORD PTR SS:[ESP+18]                
```



这是一个机器码和汇编指令的列表，主要涉及注册码验证、注册表写入等逻辑。为了保持格式清晰，以下是整理后的内容：

`004141A7` `. 51` `PUSH ECX` `; /String2 = "787878"` 假注册码

`004141A8` `. 52` `PUSH EDX` `; |String1 = "6CA0-1318-6C03-3181"` 真注册码

`004141A9` `. FF15 04114200` `CALL DWORD PTR DS:[<&KERNEL32.lstrcmpA>]` `; lstrcmpA` 比较 `|下断点做内存注册机`

`004141AF` `. 85C0` `TEST EAX,EAX`

`004141B1` `. 0F85 50010000` `JNZ hostscan.00414307` 不同跳转完完 `|爆破处，修改后自动将正确的注册码写入注册表。`

`004141B7` `. 8D4424 14` `LEA EAX,DWORD PTR SS:[ESP+14]`

`004141BB` `. 8D4C24 10` `LEA ECX,DWORD PTR SS:[ESP+10]`

`004141BF` `. 50` `PUSH EAX` `; /pDisposition`

`004141C0` `. 8B1D 00104200` `MOV EBX,DWORD PTR DS:[<&ADVAPI32.RegCreateKeyExA>]` `; |ADVAPI32.RegCreateKeyExA` 建立注册表键值

`004141C6` `. 51` `PUSH ECX` `; |pHandle`

`004141C7` `. 6A 00` `PUSH 0` `; |pSecurity = NULL`

`004141C9` `. 68 06000200` `PUSH 20006` `; |Access = KEY\_WRITE`

`004141CE` `. 6A 00` `PUSH 0` `; |Options = REG\_OPTION\_NON\_VOLATILE`

`004141D0` `. 68 C8A14200` `PUSH hostscan.0042A1C8` `; |Class = ""`

`004141D5` `. 6A 00` `PUSH 0` `; |Reserved = 0`

`004141D7` `. 68 B06B4200` `PUSH hostscan.00426BB0` `; |Subkey = "SoftwareNetSeek\\"` 注册表键值

`004141DC` `. 68 01000080` `PUSH 80000001` `; |hKey = HKEY\_CURRENT\_USER`

`004141E1` `. FFD3` `CALL EBX` `; RegCreateKeyExA`

`004141E3` `. 85C0` `TEST EAX,EAX`

`004141E5` `. 74 0C` `JE SHORT hostscan.004141F3`

`004141E7` `. 5F` `POP EDI`

`004141E8` `. 5E` `POP ESI`

`004141E9` `. 5D` `POP EBP`

`004141EA` `. 33C0` `XOR EAX,EAX`

`004141EC` `. 5B` `POP EBX`

`004141ED` `. 83C4 64` `ADD ESP,64`

`004141F0` `. C2 1000` `RETN 10`

`004141F3` `> 8B35 BC104200` `MOV ESI,DWORD PTR DS:[<&KERNEL32.lstrlenA>]` `; KERNEL32.lstrlenA`

`004141F9` `. 68 AC9D4300` `PUSH hostscan.00439DAC` `; /String = "coldeye"`

`004141FE` `. FFD6` `CALL ESI` `; lstrlenA`

`00414200` `. 40` `INC EAX`

`00414201` `. 8B5424 10` `MOV EDX,DWORD PTR SS:[ESP+10]`

`00414205` `. 50` `PUSH EAX` `; /BufSize`

`00414206` `. 68 AC9D4300` `PUSH hostscan.00439DAC` `; |Buffer = hostscan.00439DAC`

`0041420B` `. 6A 01` `PUSH 1` `; |ValueType = REG\_SZ`

`0041420D` `. 6A 00` `PUSH 0` `; |Reserved = 0`

`0041420F` `. 68 8C8A4200` `PUSH hostscan.00428A8C` `; |ValueName = "Username"`

`00414214` `. 52` `PUSH EDX` `; |hKey`

`00414215` `. FF15 14104200` `CALL DWORD PTR DS:[<&ADVAPI32.RegSetValueExA>]` `; RegSetValueExA`

`0041421B` `. 85C0` `TEST EAX,EAX`

`0041421D` `. 74 0C` `JE SHORT hostscan.0041422B`

`0041421F` `. 5F` `POP EDI`

`00414220` `. 5E` `POP ESI`

`00414221` `. 5D` `POP EBP`

`00414222` `. 33C0` `XOR EAX,EAX`

`00414224` `. 5B` `POP EBX`

`00414225` `. 83C4 64` `ADD ESP,64`

`00414228` `. C2 1000` `RETN 10`

`0041422B` `> 8B4424 10` `MOV EAX,DWORD PTR SS:[ESP+10]`

`0041422F` `. 8B3D 1C104200` `MOV EDI,DWORD PTR DS:[<&ADVAPI32.RegCloseKey>]` `; ADVAPI32.RegCloseKey`

`00414235` `. 50` `PUSH EAX` `; /hKey`

`00414236` `. FFD7` `CALL EDI` `; RegCloseKey`

`00414238` `. 8D4C24 14` `LEA ECX,DWORD PTR SS:[ESP+14]`

`0041423C` `. 8D5424 10` `LEA EDX,DWORD PTR SS:[ESP+10]`

`00414240` `. 51` `PUSH ECX`

`00414241` `. 52` `PUSH EDX`

`00414242` `. 6A 00` `PUSH 0`

`00414244` `. 68 06000200` `PUSH 20006`

`00414249` `. 6A 00` `PUSH 0`

`0041424B` `. 68 C8A14200` `PUSH hostscan.0042A1C8`

`00414250` `. 6A 00` `PUSH 0`

`00414252` `. 68 748A4200` `PUSH hostscan.00428A74` `; ASCII "hostscan.fileshellEx\\"` 注册表键值

`00414257` `. 68 00000080` `PUSH 80000000`

`0041425C` `. FFD3` `CALL EBX`

`0041425E` `. 85C0` `TEST EAX,EAX`

`00414260` `. 74 0C` `JE SHORT hostscan.0041426E`

`00414262` `. 5F` `POP EDI`

`00414263` `. 5E` `POP ESI`

`00414264` `. 5D` `POP EBP`

`00414265` `. 33C0` `XOR EAX,EAX`

`00414267` `. 5B` `POP EBX`

`00414268` `. 83C4 64` `ADD ESP,64`

`0041426B` `. C2 1000` `RETN 10`

`0041426E` `> 8D4424 18` `LEA EAX,DWORD PTR SS:[ESP+18]`

`00414272` `. 33DB` `XOR EBX,EBX`

`00414274` `. 50` `PUSH EAX`

`00414275` `. FFD6` `CALL ESI`

`00414277` `. 85C0` `TEST EAX,EAX`

`00414279` `. 7E 16` `JLE SHORT hostscan.00414291`

`0041427B` `> 8A4C1C 18` `MOV CL,BYTE PTR SS:[ESP+EBX+18]`

`0041427F` `. 02CB` `ADD CL,BL`

`00414281` `. 884C1C 18` `MOV BYTE PTR SS:[ESP+EBX+18],CL`

`00414285` `. 8D4C24 18` `LEA ECX,DWORD PTR SS:[ESP+18]`

`00414289` `. 51` `PUSH ECX`

`0041428A` `. 43` `INC EBX`

`0041428B` `. FFD6` `CALL ESI`

`0041428D` `. 3BD8` `CMP EBX,EAX`

`0041428F` `. 7C EA` `JL SHORT hostscan.0041427B`

`00414291` `> 8D5424 18` `LEA EDX,DWORD PTR SS:[ESP+18]`

`00414295` `. 52` `PUSH EDX`

`00414296` `. FFD6` `CALL ESI`

`00414298` `. 40` `INC EAX`

`00414299` `. 8B4C24 10` `MOV ECX,DWORD PTR SS:[ESP+10]`

`0041429D` `. 50` `PUSH EAX` `; /BufSize`

`0041429E` `. 8D4424 1C` `LEA EAX,DWORD PTR SS:[ESP+1C]` `; |`



004142A2    . 50                      PUSH EAX                                    ; |Buffer   

004142A3    . 6A 01                    PUSH 1                                      ; |ValueType = REG_SZ   

004142A5    . 6A 00                    PUSH 0                                      ; |Reserved = 0   

004142A7    . 68 688A4200              PUSH hostscan.00428A68                      ; |ValueName = "DropHandler"   

004142AC    . 51                       PUSH ECX                                    ; |hKey   

004142AD    . FF15 14104200           CALL DWORD PTR DS:[<&ADVAPI32.RegSetValueExA>]      ; RegSetValueExA   

004142B3    . 85C0                    TEST EAX,EAX   

004142B5    . 74 0C                    JE SHORT hostscan.004142C3   

004142B7    . 5F                       POP EDI   

004142B8    . 5E                       POP ESI   

004142B9    . 5D                       POP EBP   

004142BA    . 33C0                    XOR EAX,EAX   

004142BC    . 5B                       POP EBX   

004142BD    . 83C4 64                 ADD ESP,64   

004142C0    . C2 1000                 RETN 10   

004142C3    > 8B5424 10               MOV EDX,DWORD PTR SS:[ESP+10]   

004142C7    . 52                       PUSH EDX   

004142C8    . FFD7                    CALL EDI   

004142CA    . 6A 01                    PUSH 1                                      ; /Result = 1   

004142CC    . 55                       PUSH EBP                                    ; |hWnd   

004142CD    . C705 C4A44200 01000000  MOV DWORD PTR DS:[42A4C4],1                 ; |   

004142D7    . FF15 34124200           CALL DWORD PTR DS:[<&USER32.EndDialog>]          ; EndDialog   

004142DD    . A1 A8A44200             MOV EAX,DWORD PTR DS:[42A4A8]   

004142E2    . 68 C8A14200             PUSH hostscan.0042A1C8                      ; /lParam = 42A1C8   

004142E7    . 68 03110000             PUSH 1103                                   ; |wParam = 1103   

004142EC    . 68 01040000             PUSH 401                                    ; |Message = WM_USER+1   

004142F1    . 50                       PUSH EAX                                    ; |hWnd => E025A   

004142F2    . FF15 10124200           CALL DWORD PTR DS:[<&USER32.SendMessageA>]        ; SendMessageA   

004142F8    . 5F                       POP EDI   

004142F9    . 5E                       POP ESI   

004142FA    . 5D                       POP EBP   

004142FB    . B8 01000000             MOV EAX,1   

00414300    . 5B                       POP EBX   

00414301    . 83C4 64                 ADD ESP,64   

00414304    . C2 1000                 RETN 10   

00414307    > 68 A8A24200             PUSH hostscan.0042A2A8                      ; /IniFileName = "E:Program FilesHostScanlanguagehseng.ini"   

0041430C    . 68 C8000000             PUSH C8                                     ; |BufSize = C8 (200.)   

00414311    . 68 70A34200             PUSH hostscan.0042A370                      ; |ReturnBuffer = hostscan.0042A370   

00414316    . 68 408A4200             PUSH hostscan.00428A40                      ; |Default = "Key is Wrong,Please check the key!!!"   

0041431B    . 68 308A4200             PUSH hostscan.00428A30                      ; |Key = "KeyWrongMessage"   

00414320    . 68 90634200             PUSH hostscan.00426390                      ; |Section = "Run"   

00414325    . FF15 9C104200           CALL DWORD PTR DS:[<&KERNEL32.GetPrivateProfileStringA>]  ; GetPrivateProfileStringA   

0041432B    . 6A 40                    PUSH 40                                     ; /Style = MB_OK|MB_ICONASTERISK|MB_APPLMODAL   

0041432D    . 68 E8784200             PUSH hostscan.004278E8                      ; |Title = "Tip"   

00414332    . 68 70A34200             PUSH hostscan.0042A370                      ; |Text = "Key is Wrong,Please check the key!!!"   

00414337    . 55                       PUSH EBP                                    ; |hOwner   

00414338    . FF15 3C124200           CALL DWORD PTR DS:[<&USER32.MessageBoxA>]          ; MessageBoxA   

0041433E    . 5F                       POP EDI   

0041433F    . 5E                       POP ESI   

00414340    . 5D                       POP EBP   

00414341    . 33C0                    XOR EAX,EAX   

00414343    . 5B                       POP EBX   

00414344    . 83C4 64                 ADD ESP,64   

00414347    . C2 1000                 RETN 10   

注册成功后，软件在注册表中建立下列键值：

```
[HKEY_CURRENT_USERSoftwareNetSeek]
"language"=dword:00000010
"Start"=dword:00000001
"Username"="coldeye"                  用户名

[HKEY_LOCAL_MACHINESOFTWAREClasseshostscan.fileshellEx]
"DropHandler"="6DC31698@6@N<@;BAIC"   变换后的注册码
4dd466d0e1e0cc123afbe504da2462ca
```


# [分享]Quick Screen Capture V2.2注册分析

【破文标题】Quick Screen Capture Version 2.2
【破文作者】sailor
【作者邮箱】win97@163.com
【所属组织】江西广丰
【软件名称】Quick Screen Capture
【下载地址】003K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3&6B7i4K6u0W2L8$3&6D9K9h3&6W2k6r3!0%4L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4G2k6Y4c8Q4x3V1j5@1z5o6p5I4i4K6u0W2K9s2c8E0
【破解工具】OD, PEiD, 计算器
【保护方式】序列号
【软件限制】For trial version, there is functionality limited for saving. you can save 30 captures only. If you have captured and saved 30 captures, you can not save anymore.
【破解难度】简单

---

## 软件介绍

Quick Screen Capture可以够截取屏幕图像并保存为位图文件,通过PrintScreen或者F12功能键进行截取屏幕的操作,每按一次截取热键，C盘上就会增加一个`Picture?.bmp`文件。显然，这种直接截取并保存图像的功能便于连续地截取屏幕图像，这有些像某些照相机的连续拍摄功能。而这也许就是Quick Screen Capture程序最大的用途吧。

---

## 破解声名

---

## 破解分析

### 一、PEiD察看，Microsoft Visual C++ 6.0

### 二、OD载入，查找提示点 "Please input your name."，开始trace

#### 1.

```
004225D6   .  8D7D 60       LEA EDI,DWORD PTR SS:[EBP+60]            ;  先查找到"please input name"，从这里开始trace
004225D9   .  8BCF          MOV ECX,EDI                              ;  输入name任意，试炼码abcdefghijklmnopqrstuvwxyzABCDEF
004225DB   .  E8 E7220800   CALL Capture.004A48C7
004225E0   .  8BCF          MOV ECX,EDI
004225E2   .  E8 94220800   CALL Capture.004A487B
004225E7   .  8B06          MOV EAX,DWORD PTR DS:[ESI]               ;  eax用户名
004225E9   .  68 10F45000   PUSH Capture.0050F410                    ; /Arg2 = 0050F410
004225EE   .  50            PUSH EAX                                 ; |Arg1
004225EF   .  E8 BCEB0100   CALL Capture.004411B0                    ; \Capture.004411B0
004225F4   .  83C4 08       ADD ESP,8
004225F7   .  85C0          TEST EAX,EAX                             ;  判断有没有输入用户名
004225F9   .  75 26         JNZ SHORT Capture.00422621               ;  跳下去才有戏
004225FB   .  6A 40         PUSH 40
004225FD   .  68 28985000   PUSH Capture.00509828                    ;  ASCII "!Quick Screen Capture"
00422602   .  68 44A75000   PUSH Capture.0050A744                    ;  ASCII "Please input your name."
00422607   .  8BCD          MOV ECX,EBP
00422609   .  E8 CC7E0800   CALL Capture.004AA4DA
0042260E   .  8B4C24 1C     MOV ECX,DWORD PTR SS:[ESP+1C]
00422612   .  64:890D 00000>MOV DWORD PTR FS:[0],ECX
00422619   .  5F            POP EDI
0042261A   .  5E            POP ESI
0042261B   .  5D            POP EBP
0042261C   .  5B            POP EBX
0042261D   .  83C4 18       ADD ESP,18
00422620   .  C3            RETN
00422621   >  8A45 5C       MOV AL,BYTE PTR SS:[EBP+5C]              ;  跳到这里，开始处理试炼码
00422624   .  84C0          TEST AL,AL
00422626   .  0F85 1A010000 JNZ Capture.00422746
0042262C   .  8D4C24 14     LEA ECX,DWORD PTR SS:[ESP+14]
00422630   .  E8 BB0FFEFF   CALL Capture.004035F0
00422635   .  51            PUSH ECX
00422636   .  C74424 28 000>MOV DWORD PTR SS:[ESP+28],0
0042263E   .  8BCC          MOV ECX,ESP
00422640   .  896424 14     MOV DWORD PTR SS:[ESP+14],ESP
00422644   .  56            PUSH ESI
00422645   .  E8 B19E0800   CALL Capture.004AC4FB
0042264A   .  51            PUSH ECX
0042264B   .  C64424 2C 01  MOV BYTE PTR SS:[ESP+2C],1
00422650   .  8BCC          MOV ECX,ESP
00422652   .  896424 20     MOV DWORD PTR SS:[ESP+20],ESP
00422656   .  57            PUSH EDI
00422657   .  E8 9F9E0800   CALL Capture.004AC4FB
0042265C   .  8D4C24 1C     LEA ECX,DWORD PTR SS:[ESP+1C]            ; |
00422660   .  C64424 2C 00  MOV BYTE PTR SS:[ESP+2C],0               ; |
00422665   .  E8 C60FFEFF   CALL Capture.00403630                    ; \关键CALL，跟进，目标al非0
0042266A   .  84C0          TEST AL,AL
0042266C   .  75 37         JNZ SHORT Capture.004226A5               ;  不跳就死，跳就活
0042266E   .  6A 40         PUSH 40
00422670   .  68 28985000   PUSH Capture.00509828                    ;  ASCII "!Quick Screen Capture"
00422675   .  68 FCA65000   PUSH Capture.0050A6FC                    ;  ASCII "Sorry, your registration key is wrong. Please check it and try again."
0042267A   .  8BCD          MOV ECX,EBP
0042267C   .  E8 597E0800   CALL Capture.004AA4DA
00422681   .  8D4C24 14     LEA ECX,DWORD PTR SS:[ESP+14]
00422685   .  C74424 24 FFF>MOV DWORD PTR SS:[ESP+24],-1
0042268D   .  E8 8E0FFEFF   CALL Capture.00403620
00422692   .  8B4C24 1C     MOV ECX,DWORD PTR SS:[ESP+1C]
00422696   .  64:890D 00000>MOV DWORD PTR FS:[0],ECX
0042269D   .  5F            POP EDI
0042269E   .  5E            POP ESI
0042269F   .  5D            POP EBP
004226A0   .  5B            POP EBX
004226A1   .  83C4 18       ADD ESP,18
004226A4   .  C3            RETN
004226A5   >  8D4424 10     LEA EAX,DWORD PTR SS:[ESP+10]            ;  将注册信息写入注册表中
004226A9   .  50            PUSH EAX                                 ; /pHandle
004226AA   .  68 F49C5000   PUSH Capture.00509CF4                    ; |Subkey = "SOFTWARE\Microsoft\Windows\WindowsUp"
004226AF   .  68 02000080   PUSH 80000002                            ; |hKey = HKEY\_LOCAL\_MACHINE
004226B4   .  FF15 0C004E00 CALL DWORD PTR DS:[<&ADVAPI32.RegCreateK>; \RegCreateKeyA
004226BA   .  8B0F          MOV ECX,DWORD PTR DS:[EDI]
004226BC   .  8B41 F8       MOV EAX,DWORD PTR DS:[ECX-8]
004226BF   .  8BCF          MOV ECX,EDI
004226C1   .  50            PUSH EAX
004226C2   .  50            PUSH EAX
004226C3   .  E8 EEA40800   CALL Capture.004ACBB6
004226C8   .  8B5424 14     MOV EDX,DWORD PTR SS:[ESP+14]            ; |
004226CC   .  8B1D 10004E00 MOV EBX,DWORD PTR DS:[<&ADVAPI32.RegSetV>; |ADVAPI32.RegSetValueExA
004226D2   .  50            PUSH EAX                                 ; |Buffer
004226D3   .  6A 01         PUSH 1                                   ; |ValueType = REG\_SZ
004226D5   .  6A 00         PUSH 0                                   ; |Reserved = 0
004226D7   .  68 EC9C5000   PUSH Capture.00509CEC                    ; |ValueName = "VValue"
004226DC   .  52            PUSH EDX                                 ; |hKey
004226DD   .  FFD3          CALL EBX                                 ; \RegSetValueExA
004226DF   .  8B06          MOV EAX,DWORD PTR DS:[ESI]
004226E1   .  8BCE          MOV ECX,ESI
```

#### 2.

```
00403630  /$  6A FF         PUSH -1                                  ;  从这里进来
00403632  |.  68 A84F4D00   PUSH Capture.004D4FA8                    ;  SE handler installation
00403637  |.  64:A1 0000000>MOV EAX,DWORD PTR FS:[0]
0040363D  |.  50            PUSH EAX
0040363E  |.  64:8925 00000>MOV DWORD PTR FS:[0],ESP
00403645  |.  83EC 54       SUB ESP,54
00403648  |.  53            PUSH EBX
00403649  |.  55            PUSH EBP
0040364A  |.  56            PUSH ESI
0040364B  |.  57            PUSH EDI
0040364C  |.  894C24 60     MOV DWORD PTR SS:[ESP+60],ECX
00403650  |.  C74424 6C 000>MOV DWORD PTR SS:[ESP+6C],0
00403658  |.  8B4424 74     MOV EAX,DWORD PTR SS:[ESP+74]            ;  eax试炼码
0040365C  |.  8B40 F8       MOV EAX,DWORD PTR DS:[EAX-8]             ;  eax试炼码位数
0040365F  |.  83F8 20       CMP EAX,20                               ;  是否为32位
00403662  |.  0F85 20050000 JNZ Capture.00403B88                     ;  不是的话，就死
00403668  |.  A1 ECBD5000   MOV EAX,DWORD PTR DS:[50BDEC]
0040366D  |.  894424 14     MOV DWORD PTR SS:[ESP+14],EAX
00403671  |.  894424 10     MOV DWORD PTR SS:[ESP+10],EAX
00403675  |.  6A 01         PUSH 1
00403677  |.  8D4C24 60     LEA ECX,DWORD PTR SS:[ESP+60]
0040367B  |.  6A 09         PUSH 9                                   ;  取试炼码的第9+1位字符
0040367D  |.  51            PUSH ECX
0040367E  |.  8D8C24 800000>LEA ECX,DWORD PTR SS:[ESP+80]
00403685  |.  C64424 78 03  MOV BYTE PTR SS:[ESP+78],3
0040368A  |.  E8 780C0A00   CALL Capture.004A4307
0040368F  |.  8BF0          MOV ESI,EAX
00403691  |.  6A 01         PUSH 1
00403693  |.  8D5424 5C     LEA EDX,DWORD PTR SS:[ESP+5C]
00403697  |.  6A 1F         PUSH 1F                                  ;  取试炼码的第1F+1位字符
```

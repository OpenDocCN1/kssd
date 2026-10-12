
# Quick Screen Capture V2.2 注册分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-240.htm`](https://www.pediy.com/kssd/pediy07/pediy7-240.htm)

【破文标题】Quick Screen Capture Version 2.2  

【破文作者】sailor  

【作者邮箱】win97@163.com  

【所属组织】江西广丰  

【软件名称】Quick Screen Capture  

【下载地址】`http://nj.onlinedown.net/soft/4811.htm`  

【破解工具】`OD`, `PEiD`, 计算器  

【保护方式】序列号  

【软件限制】For trial version, there is functionality limited for saving. you can save 30 captures only. If you have captured and saved 30 captures, you can not save anymore.  

【破解难度】简单  

----------------------------------------------------

**软件介绍：**

Quick Screen Capture 可以够截取屏幕图像并保存为位图文件，通过 `PrintScreen` 或者 `F12` 功能键进行截取屏幕的操作，每按一次截取热键，`C` 盘上就会增加一个 `Picture?.bmp` 文件。显然，这种直接截取并保存图像的功能便于连续地截取屏幕图像，这有些像某些照相机的连续拍摄功能。而这也许就是 Quick Screen Capture 程序最大的用途吧。

----------------------------------------------------

**破解声名：**

----------------------------------------------------

**【破解分析】**

一、`PEiD` 察看，Microsoft Visual C++ 6.0

二、`OD` 载入，查找提示点 `"Please input your name."`，开始 trace

1.

```
004225D6   .  8D7D 60       LEA EDI,DWORD PTR SS:[EBP+60]             ;  先查找到"please 
input name"，从这里开始 trace
004225D9   .  8BCF          MOV ECX,EDI                               ;  输入 name 任意，试炼码 
abcdefghijklmnopqrstuvwxyzABCDEF
004225DB   .  E8 E7220800   CALL Capture.004A48C7
004225E0   .  8BCF          MOV ECX,EDI
004225E2   .  E8 94220800   CALL Capture.004A487B
004225E7   .  8B06          MOV EAX,DWORD PTR DS:[ESI]                ;  eax 用户名
004225E9   .  68 10F45000   PUSH Capture.0050F410                     ; /Arg2 = 0050F410
004225EE   .  50            PUSH EAX                                  ; |Arg1
004225EF   .  E8 BCEB0100   CALL Capture.004411B0                    ; \Capture.004411B0
004225F4   .  83C4 08       ADD ESP,8
004225F7   .  85C0          TEST EAX,EAX                              ;  判断有没有输入用户名
004225F9   .  75 26         JNZ SHORT Capture.00422621               ;  跳下去才有戏
004225FB   .  6A 40         PUSH 40
004225FD   .  68 28985000   PUSH Capture.00509828                     ;  ASCII "!Quick Screen 
Capture"
00422602   .  68 44A75000   PUSH Capture.0050A744                     ;  ASCII "Please input 
your name."
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
00422621   >  8A45 5C       MOV AL,BYTE PTR SS:[EBP+5C]               ;  跳到这里，开始处理试 
炼码
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
0042265C   .  8D4C24 1C     LEA ECX,DWORD PTR SS:[ESP+1C]             ; |
00422660   .  C64424 2C 00  MOV BYTE PTR SS:[ESP+2C],0              ; |
00422665   .  E8 C60FFEFF   CALL Capture.00403630                    ; \关键 CALL，跟进，目标 
al 非 0
0042266A   .  84C0          TEST AL,AL
0042266C   .  75 37         JNZ SHORT Capture.004226A5               ;  不跳就死，跳就活
0042266E   .  6A 40         PUSH 40
00422670   .  68 28985000   PUSH Capture.00509828                     ;  ASCII "!Quick Screen 
Capture"
00422675   .  68 FCA65000   PUSH Capture.0050A6FC                     ;  ASCII "Sorry, your 
registration key is wrong. Please check it and try again."
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
004226A5   >  8D4424 10     LEA EAX,DWORD PTR SS:[ESP+10]             ;  将注册信息写入注册表 
中
004226A9   .  50            PUSH EAX                                  ; /pHandle
004226AA   .  68 F49C5000   PUSH Capture.00509CF4                     ; |Subkey = 
"SOFTWARE\\Microsoft\\Windows\\WindowsUp"
004226AF   .  68 02000080   PUSH 80000002                             ; |hKey = 
HKEY_LOCAL_MACHINE
004226B4   .  FF15 0C004E00 CALL DWORD PTR DS:[<&ADVAPI32.RegCreateK>; \RegCreateKeyA
004226BA   .  8B0F          MOV ECX,DWORD PTR DS:[EDI]
004226BC   .  8B41 F8       MOV EAX,DWORD PTR DS:[ECX-8]
004226BF   .  8BCF          MOV ECX,EDI
004226C1   .  50            PUSH EAX
004226C2   .  50            PUSH EAX
004226C3   .  E8 EEA40800   CALL Capture.004ACBB6
004226C8   .  8B5424 14     MOV EDX,DWORD PTR SS:[ESP+14]             ; |
004226CC   .  8B1D 10004E00 MOV EBX,DWORD PTR DS:[<&ADVAPI32.RegSetV>; 
|ADVAPI32.RegSetValueExA
004226D2   .  50            PUSH EAX                                  ; |Buffer
004226D3   .  6A 01          PUSH 1                                    ; |ValueType = REG_SZ
004226D5   .  6A 00          PUSH 0                                    ; |Reserved = 0
004226D7   .  68 EC9C5000   PUSH Capture.00509CEC                     ; |ValueName = "VValue"
004226DC   .  52            PUSH EDX                                  ; |hKey
004226DD   .  FFD3          CALL EBX                                  ; \RegSetValueExA
004226DF   .  8B06          MOV EAX,DWORD PTR DS:[ESI]
004226E1   .  8BCE          MOV ECX,ESI
```

2.

```
00403630  /$  6A FF         PUSH -1                                   ;  从这里进来
00403632  |.  68 A84F4D00   PUSH Capture.004D4FA8                     ;  SE handler 
installation
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
00403658  |.  8B4424 74     MOV EAX,DWORD PTR SS:[ESP+74]             ;  eax 试炼码
0040365C  |.  8B40 F8       MOV EAX,DWORD PTR DS:[EAX-8]              ;  eax 试炼码位数
0040365F  |.  83F8 20       CMP EAX,20                                ;  是否为 32 位
00403662  |.  0F85 20050000 JNZ Capture.00403B88                      ;  不是的话，就死
00403668  |.  A1 ECBDB5000   MOV EAX,DWORD PTR DS:[50BDEC]
0040366D  |.  894424 14     MOV DWORD PTR SS:[ESP+14],EAX
00403671  |.  894424 10     MOV DWORD PTR SS:[ESP+10],EAX
00403675  |.  6A 01          PUSH 1
00403677  |.  8D4C24 60     LEA ECX,DWORD PTR SS:[ESP+60]
0040367B  |.  6A 09          PUSH 9                                    ;  取试炼码的第 9+1 位字 
符
0040367D  |.  51            PUSH ECX
```



### 0040367E

```
0040367E  |.  8D8C24 800000>LEA ECX,DWORD PTR SS:\[ESP+80\]
```

### 00403685

```
00403685  |.  C64424 78 03  MOV BYTE PTR SS:\[ESP+78\],3
```

### 0040368A

```
0040368A  |.  E8 780C0A00   CALL Capture.004A4307
```

### 0040368F

```
0040368F  |.  8BF0          MOV ESI,EAX
```

### 00403691

```
00403691  |.  6A 01         PUSH 1
```

### 00403693

```
00403693  |.  8D5424 5C     LEA EDX,DWORD PTR SS:\[ESP+5C\]
```

### 00403697

```
00403697  |.  6A 1F         PUSH 1F                           ;  取试炼码的第 1F+1 位字

符
```

### 00403699

```
00403699  |.  52            PUSH EDX
```

### 0040369A

```
0040369A  |.  8D8C24 800000>LEA ECX,DWORD PTR SS:\[ESP+80\]
```

### 004036A1

```
004036A1  |.  C64424 78 04  MOV BYTE PTR SS:\[ESP+78\],4
```

### 004036A6

```
004036A6  |.  E8 5C0C0A00   CALL Capture.004A4307
```

### 004036AB

```
004036AB  |.  8BF8          MOV EDI,EAX
```

### 004036AD

```
004036AD  |.  6A 01         PUSH 1
```

### 004036AF

```
004036AF  |.  8D4424 58     LEA EAX,DWORD PTR SS:\[ESP+58\]
```

### 004036B3

```
004036B3  |.  6A 06         PUSH 6                            ;  取试炼码的第 6+1 位字

符，后面依次取第某位的试炼码
```

### 004036B5

```
004036B5  |.  50            PUSH EAX
```

### 004036B6

```
004036B6  |.  8D8C24 800000>LEA ECX,DWORD PTR SS:\[ESP+80\]
```

### 004036BD

```
004036BD  |.  C64424 78 05  MOV BYTE PTR SS:\[ESP+78\],5
```

### 004036C2

```
004036C2  |.  E8 400C0A00   CALL Capture.004A4307
```

##### 省略一部分代码

```
#########################################################            ;  省略一部分代码
```

### 0040373B

```
0040373B  |.  894424 18     MOV DWORD PTR SS:\[ESP+18\],EAX
```

### 0040373F

```
0040373F  |.  6A 01         PUSH 1
```

### 00403741

```
00403741  |.  8D5424 44     LEA EDX,DWORD PTR SS:\[ESP+44\]
```

### 00403745

```
00403745  |.  6A 00         PUSH 0                            ;  取试炼码的第 0+1 位字

符
```

### 00403747

```
00403747  |.  52            PUSH EDX
```

### 00403748

```
00403748  |.  8D8C24 800000>LEA ECX,DWORD PTR SS:\[ESP+80\]
```

### 0040374F

```
0040374F  |.  C64424 78 0A  MOV BYTE PTR SS:\[ESP+78\],0A
```

### 00403754

```
00403754  |.  E8 AE0B0A00   CALL Capture.004A4307
```

### 00403759

```
00403759  |.  8B4C24 18     MOV ECX,DWORD PTR SS:\[ESP+18\]
```

### 0040375D

```
0040375D  |.  C64424 6C 0B  MOV BYTE PTR SS:\[ESP+6C\],0B
```

### 00403762

```
00403762  |.  51            PUSH ECX
```

### 00403763

```
00403763  |.  50            PUSH EAX
```

### 00403764

```
00403764  |.  8D5424 44     LEA EDX,DWORD PTR SS:\[ESP+44\]
```

### 00403768

```
00403768  |.  52            PUSH EDX
```

### 00403769

```
00403769  |.  E8 47920A00   CALL Capture.004AC9B5
```

##### 省略一部分代码

```
#########################################################            ;  省略一部分代码
```

### 0040389C

```
0040389C  |.  E8 E58E0A00   CALL Capture.004AC786
```

### 004038A1

```
004038A1  |.  8D4C24 58     LEA ECX,DWORD PTR SS:\[ESP+58\]
```

### 004038A5

```
004038A5  |.  C64424 6C 04  MOV BYTE PTR SS:\[ESP+6C\],4
```

### 004038AA

```
004038AA  |.  E8 D78E0A00   CALL Capture.004AC786
```

### 004038AF

```
004038AF  |.  8D4C24 5C     LEA ECX,DWORD PTR SS:\[ESP+5C\]
```

### 004038B3

```
004038B3  |.  C64424 6C 03  MOV BYTE PTR SS:\[ESP+6C\],3
```

### 004038B8

```
004038B8  |.  E8 C98E0A00   CALL Capture.004AC786                    ;  这段程序将上面取出的

字符反过来组成字符串 `str1`
```

### 004038BD

```
004038BD  |.  6A 01         PUSH 1                            ;  下面程序段功能与上面

一样，最终形成字符串 `str2`
```

### 004038BF

```
004038BF  |.  8D4C24 1C     LEA ECX,DWORD PTR SS:\[ESP+1C\]
```

### 004038C3

```
004038C3  |.  6A 03         PUSH 3
```

### 004038C5

```
004038C5  |.  51            PUSH ECX
```

### 004038C6

```
004038C6  |.  8D8C24 800000>LEA ECX,DWORD PTR SS:\[ESP+80\]
```

### 004038CD

```
004038CD  |.  E8 350A0A00   CALL Capture.004A4307
```

##### 省略一部分代码

```
#########################################################            ;  省略一部分代码
```

### 00403999

```
00403999  |.  8B4C24 5C     MOV ECX,DWORD PTR SS:\[ESP+5C\]
```

### 0040399D

```
0040399D  |.  8D5424 38     LEA EDX,DWORD PTR SS:\[ESP+38\]
```

### 004039A1

```
004039A1  |.  51            PUSH ECX
```

### 004039A2

```
004039A2  |.  50            PUSH EAX
```

### 004039A3

```
004039A3  |.  52            PUSH EDX
```

### 004039A4

```
004039A4  |.  C64424 78 1A  MOV BYTE PTR SS:\[ESP+78\],1A
```

### 004039A9

```
004039A9  |.  E8 07900A00   CALL Capture.004AC9B5
```

##### 省略一部分代码

```
#########################################################            ;  省略一部分代码
```

### 00403AD3

```
00403AD3  |.  8D4C24 20     LEA ECX,DWORD PTR SS:\[ESP+20\]
```

### 00403AD7

```
00403AD7  |.  C64424 6C 14  MOV BYTE PTR SS:\[ESP+6C\],14
```

### 00403ADC

```
00403ADC  |.  E8 A58C0A00   CALL Capture.004AC786
```

### 00403AE1

```
00403AE1  |.  8D4C24 1C     LEA ECX,DWORD PTR SS:\[ESP+1C\]
```

### 00403AE5

```
00403AE5  |.  C64424 6C 13  MOV BYTE PTR SS:\[ESP+6C\],13
```

### 00403AEA

```
00403AEA  |.  E8 978C0A00   CALL Capture.004AC786
```

### 00403AEF

```
00403AEF  |.  8D4C24 18     LEA ECX,DWORD PTR SS:\[ESP+18\]
```

### 00403AF3

```
00403AF3  |.  C64424 6C 03  MOV BYTE PTR SS:\[ESP+6C\],3
```

### 00403AF8

```
00403AF8  |.  E8 898C0A00   CALL Capture.004AC786
```

### 00403AFD

```
00403AFD  |.  51            PUSH ECX
```

### 00403AFE

```
00403AFE  |.  8D5424 14     LEA EDX,DWORD PTR SS:\[ESP+14\]
```

### 00403B02

```
00403B02  |.  8BCC          MOV ECX,ESP
```

### 00403B04

```
00403B04  |.  896424 60     MOV DWORD PTR SS:\[ESP+60\],ESP
```

### 00403B08

```
00403B08  |.  52            PUSH EDX
```

### 00403B09

```
00403B09  |.  E8 ED890A00   CALL Capture.004AC4FB
```

### 00403B0E

```
00403B0E  |.  51            PUSH ECX
```

### 00403B0F

```
00403B0F  |.  8D4424 1C     LEA EAX,DWORD PTR SS:\[ESP+1C\]
```

### 00403B13

```
00403B13  |.  8BCC          MOV ECX,ESP
```

### 00403B15

```
00403B15  |.  896424 60     MOV DWORD PTR SS:\[ESP+60\],ESP
```

### 00403B19

```
00403B19  |.  50            PUSH EAX
```

### 00403B1A

```
00403B1A  |.  C64424 78 22  MOV BYTE PTR SS:\[ESP+78\],22
```

### 00403B1F

```
00403B1F  |.  E8 D7890A00   CALL Capture.004AC4FB                    ;  注意观察堆栈区，有两

个字符串
```

### 00403B24

```
00403B24  |.  8B4C24 68     MOV ECX,DWORD PTR SS:\[ESP+68\]          ;  |`str1="afhlxgFj"`

`str2="bmnEuCzd"`
```

### 00403B28

```
00403B28  |.  C64424 74 03  MOV BYTE PTR SS:\[ESP+74\],3           ;  |
```

### 00403B2D

```
00403B2D  |.  E8 8E000000   CALL Capture.00403BC0                    ;  \\第二个关键 CALL，继

续跟进
```

### 00403B32

```
00403B32  |.  84C0          TEST AL,AL
```

### 00403B34

```
00403B34  |.  C64424 6C 02  MOV BYTE PTR SS:\[ESP+6C\],2
```

### 00403B39

```
00403B39  |.  8D4C24 10     LEA ECX,DWORD PTR SS:\[ESP+10\]
```

### 00403B3D

```
00403B3D  |.  74 36         JE SHORT Capture.00403B75
```

### 3、

### 00403BC0

```
00403BC0  /$  6A FF         PUSH -1                           ;  这次来到了这里
```

### 00403BC2

```
00403BC2  |.  68 E84F4D00   PUSH Capture.004D4FE8                    ;  SE handler installation
```

### 00403BC7

```
00403BC7  |.  64:A1 0000000>MOV EAX,DWORD PTR FS:\[0\]
```

### 00403BCD

```
00403BCD  |.  50            PUSH EAX
```

### 00403BCE

```
00403BCE  |.  64:8925 00000>MOV DWORD PTR FS:\[0\],ESP
```

### 00403BD5

```
00403BD5  |.  83EC 0C       SUB ESP,0C
```

### 00403BD8

```
00403BD8  |.  53            PUSH EBX
```

### 00403BD9

```
00403BD9  |.  55            PUSH EBP
```

### 00403BDA

```
00403BDA  |.  56            PUSH ESI
```

### 00403BDB

```
00403BDB  |.  57            PUSH EDI
```

### 00403BDC

```
00403BDC  |.  8B4424 2C     MOV EAX,DWORD PTR SS:\[ESP+2C\]          ;  `eax="afhlxgFj"`
```

### 00403BE0

```
00403BE0  |.  C74424 24 010>MOV DWORD PTR SS:\[ESP+24\],1
```

### 00403BE8

```
00403BE8  |.  8B40 F8       MOV EAX,DWORD PTR DS:\[EAX-8\]
```

### 00403BEB

```
00403BEB  |.  83F8 08       CMP EAX,8                             ;  看看 `str1` 是否 8 位
```

### 00403BEE

```
00403BEE  |.  0F85 E8000000 JNZ Capture.00403CDC
```

### 00403BF4

```
00403BF4  |.  8B4C24 30     MOV ECX,DWORD PTR SS:\[ESP+30\]          ;  `ecx="bmnEuCzd"`
```

### 00403BF8

```
00403BF8  |.  8379 F8 08    CMP DWORD PTR DS:\[ECX-8\],8            ;  看看 `str2` 是否 8 位
```

### 00403BFC

```
00403BFC  |.  0F85 DA000000 JNZ Capture.00403CDC
```

### 00403C02

```
00403C02  |.  68 4C945000   PUSH Capture.0050944C                    ;  push

`str3="abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890"`
```

### 00403C07

```
00403C07  |.  8D4C24 14     LEA ECX,DWORD PTR SS:\[ESP+14\]
```

### 00403C0B

```
00403C0B  |.  E8 E48B0A00   CALL Capture.004AC7F4
```

### 00403C10

```
00403C10  |.  B3 02         MOV BL,2
```

### 00403C12

```
00403C12  |.  33F6          XOR ESI,ESI
```

### 00403C14

```
00403C14  |.  885C24 24     MOV BYTE PTR SS:\[ESP+24\],BL
```

### 00403C18

```
00403C18  |>  6A 01         /PUSH 1
```

### 00403C1A

```
00403C1A  |.  8D5424 18     |LEA EDX,DWORD PTR SS:\[ESP+18\]
```

### 00403C1E

```
00403C1E  |.  56            |PUSH ESI
```

### 00403C1F

```
00403C1F  |.  52            |PUSH EDX
```

### 00403C20

```
00403C20  |.  8D4C24 38     |LEA ECX,DWORD PTR SS:\[ESP+38\]
```

### 00403C24

```
00403C24  |.  E8 DE060A00   |CALL Capture.004A4307
```

### 00403C29

```
00403C29  |.  8B00          |MOV EAX,DWORD PTR DS:\[EAX\]
```

### 00403C2B

```
00403C2B  |.  8D4C24 10     |LEA ECX,DWORD PTR SS:\[ESP+10\]
```

### 00403C2F

```
00403C2F  |.  50            |PUSH EAX
```

### 00403C30

```
00403C30  |.  C64424 28 03  |MOV BYTE PTR SS:\[ESP+28\],3
```

### 00403C35

```
00403C35  |.  E8 A4080A00   |CALL Capture.004A44DE
```

### 00403C3A

```
00403C3A  |.  8D4C24 14     |LEA ECX,DWORD PTR SS:\[ESP+14\]
```

### 00403C3E

```
00403C3E  |.  8BF8          |MOV EDI,EAX
```

### 00403C40

```
00403C40  |.  885C24 24     |MOV BYTE PTR SS:\[ESP+24\],BL
```

### 00403C44

```
00403C44  |.  E8 3D8B0A00   |CALL Capture.004AC786
```

### 00403C49

```
00403C49  |.  6A 01         |PUSH 1
```

### 00403C4B

```
00403C4B  |.  8D4424 1C     |LEA EAX,DWORD PTR SS:\[ESP+1C\]
```

### 00403C4F

```
00403C4F  |.  56            |PUSH ESI
```



# 代码分析

## 汇编代码段

```
00403C50  .|. 50               |PUSH EAX  
00403C51  .|. 8D4C24 3C        |LEA ECX,DWORD PTR SS:[ESP+3C]  
00403C55  .|. E8 AD060A00      |CALL Capture.004A4307  
00403C5A  .|. 8B00             |MOV EAX,DWORD PTR DS:[EAX]  
00403C5C  .|. 8D4C24 10        |LEA ECX,DWORD PTR SS:[ESP+10]  
00403C60  .|. 50               |PUSH EAX  
00403C61  .|. C64424 28 04     |MOV BYTE PTR SS:[ESP+28],4  
00403C66  .|. E8 73080A00      |CALL Capture.004A44DE  
00403C6B  .|. 8D4C24 18        |LEA ECX,DWORD PTR SS:[ESP+18]  
00403C6F  .|. 8BE8             |MOV EBP,EAX  
00403C71  .|. 885C24 24        |MOV BYTE PTR SS:[ESP+24],BL  
00403C75  .|. E8 0C8B0A00      |CALL Capture.004AC786                     ;  str1,str2 字符不在  

str3 中，则相应的 edi,ebp 返回-1  
00403C7A  .|. 83FF FF          |CMP EDI,-1                               ;  edi=str1 各个字符在  

str3 中的位置  
00403C7D  .|. 74 4F            |JE SHORT Capture.00403CCE  
00403C7F  .|. 83FD FF          |CMP EBP,-1                               ;  ebp=str2 各个字符在  

str3 中的位置  
00403C82  .|. 74 4A            |JE SHORT Capture.00403CCE  
00403C84  .|. 8D47 0B          |LEA EAX,DWORD PTR DS:[EDI+B]             ;  eax=edi+B  
00403C87  .|. B9 3E000000      |MOV ECX,3E  
00403C8C  .|. 99               |CDQ  
00403C8D  .|. F7F9             |IDIV ECX                                 ;  eax mod 3E  
00403C8F  .|. 3BD5             |CMP EDX,EBP                              ;  余数与 ebp 相比较  
00403C91  .|. 75 3B            |JNZ SHORT Capture.00403CCE               ;  不相等就去死  
00403C93  .|. 46               |INC ESI  
00403C94  .|. 83FE 08          |CMP ESI,8  
00403C97  |.^ 0F8C 7BFFFFFF   \JL Capture.00403C18                       ;  是否比较完 8 个字符  
00403C9D  .|. 8D4C24 10        LEA ECX,DWORD PTR SS:[ESP+10]  
00403CA1  .|. C64424 24 01     MOV BYTE PTR SS:[ESP+24],1  
00403CA6  .|. E8 DB8A0A00      CALL Capture.004AC786  
00403CAB  .|. 8D4C24 2C        LEA ECX,DWORD PTR SS:[ESP+2C]  
00403CAF  .|. C64424 24 00     MOV BYTE PTR SS:[ESP+24],0  
00403CB4  .|. E8 CD8A0A00      CALL Capture.004AC786  
00403CB9  .|. 8D4C24 30        LEA ECX,DWORD PTR SS:[ESP+30]  
00403CBD  .|. C74424 24 FFF>   MOV DWORD PTR SS:[ESP+24],-1  
00403CC5  .|. E8 BC8A0A00      CALL Capture.004AC786  
00403CCA  .|. B0 01            MOV AL,1                                 ;  胜利之门  
00403CCC  .|. EB 2F            JMP SHORT Capture.00403CFD  
00403CCE  |> 8D4C24 10         LEA ECX,DWORD PTR SS:[ESP+10]  
00403CD2  .|. C64424 24 01     MOV BYTE PTR SS:[ESP+24],1  
00403CD7  .|. E8 AA8A0A00      CALL Capture.004AC786  
00403CDC  |> 8D4C24 2C         LEA ECX,DWORD PTR SS:[ESP+2C]  
00403CE0  .|. C64424 24 00     MOV BYTE PTR SS:[ESP+24],0  
00403CE5  .|. E8 9C8A0A00      CALL Capture.004AC786  
00403CEA  .|. 8D4C24 30        LEA ECX,DWORD PTR SS:[ESP+30]  
00403CEE  .|. C74424 24 FFF>   MOV DWORD PTR SS:[ESP+24],-1  
00403CF6  .|. E8 8B8A0A00      CALL Capture.004AC786  
00403CFB  .|. 32C0             XOR AL,AL                                ;  地狱之门  
00403CFD  |> 8B4C24 1C         MOV ECX,DWORD PTR SS:[ESP+1C]  
00403D01  .|. 5F               POP EDI  
00403D02  .|. 5E               POP ESI  
00403D03  .|. 5D               POP EBP  
00403D04  .|. 5B               POP EBX  
00403D05  .|. 64:890D 00000>   MOV DWORD PTR FS:[0],ECX  
00403D0C  .|. 83C4 18          ADD ESP,18  
00403D0F  \\. C2 0800          RETN 8  
```

## 总结

注意：以下数字为 16 进制

1. 注册码必须为 20 位
2. 取试炼码第 `1`、`6`、`8`、`C`、`18`、`7`、`20`、`A` 位组成字符串 `str1`
   取试炼码第 `2`、`D`、`E`、`1F`、`17`、`1D`、`1A`、`4` 位组成字符串 `str2`
3. 依次取 `str1`、`str2` 中的字符，判断是否在 `str3` 中
   `str3="abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890"`
   `(str1[i]+0B) mod 3E = str2[i]`

可用的一组注册码：`alcuefghijklqsopqrstuvIxyQABrDwF`

注册信息保存在 `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\WindowsUp`

## 版权信息

版权所有，翻版不究!
`4dd466d0e1e0cc123afbe504da2462ca`

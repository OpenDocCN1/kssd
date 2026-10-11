
# SockMon2005 网络验证的去除(附内存补丁程序源码)

> 原文：[`www.pediy.com/kssd/pediy10/60114.html`](https://www.pediy.com/kssd/pediy10/60114.html)

SockMon2005 网络验证的去除及内存补丁程序源码

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

前段时间，在网上下载了一个 `SockMon2005`，每隔一分钟就弹出一个要求注册的窗口，运行 5 分钟后，程序就中止了。

当时就试着破了一下，跟到后来，跟到了网络验证部分，由于对网络验证的破解十分手生，不敢继续往下跟了，就放到一边去了。

这两天心又不甘，再拿来分析，不料，误打误撞，居然给破了。特将破解过程中所作的一些笔记整理一下，串在一起，并贴出来，以供我们这些菜鸟们共同学习，探讨。

高手飘过。

---

## 一、程序分析

`PEID` 侦测： `ASPack 2.12` -> `Alexey Solodovnikov` 壳，

但手工脱壳后，发现引入表加密，就懒得去费心去修复引入表了，直接带壳分析吧。

---

## 二、设断跟踪

进入注册界面，用第三方工具查出编辑框的句柄值为：`3041E`， 所以作如下设断：

```
bp CallWindowProcA [esp+8]==3041E && [esp+0c]==WM_GETTEXT
```

用于拦截程序取编辑框上的文本，点击注册后，通过堆栈窗口，可以返回到：

```
004A7878    55               PUSH EBP
004A7879    68 1B7A4A00      PUSH SockMon5.004A7A1B
004A787E    64:FF30          PUSH DWORD PTR FS:[EAX]
004A7881    64:8920          MOV DWORD PTR FS:[EAX],ESP
004A7884    8D95 F8FDFFFF    LEA EDX,DWORD PTR SS:[EBP-208]
004A788A    8B83 FC020000    MOV EAX,DWORD PTR DS:[EBX+2FC]
004A7890    E8 7B11FAFF      CALL SockMon5.00448A10
004A7895    8B85 F8FDFFFF    MOV EAX,DWORD PTR SS:[EBP-208]  ;EAX=17F8-65A4-01A1-CB77-6439 机器码
004A789B    E8 88CEF5FF      CALL SockMon5.00404728     ;eax=18h 机器码长度
004A78A0    8BF0             MOV ESI,EAX
004A78A2    8D95 F4FDFFFF    LEA EDX,DWORD PTR SS:[EBP-20C]
004A78A8    8B83 00030000    MOV EAX,DWORD PTR DS:[EBX+300]
004A78AE    E8 5D11FAFF      CALL SockMon5.00448A10     ;<=== 取输入的注册码
004A78B3    8B85 F4FDFFFF    MOV EAX,DWORD PTR SS:[EBP-20C]  ;eax=123456789 输入的注册码
004A78B9    E8 6ACEF5FF      CALL SockMon5.00404728     ;输入注册码长度
004A78BE    8BF8             MOV EDI,EAX
004A78C0    3BFE             CMP EDI,ESI        ;esi=18h;
004A78C2    0F85 2A010000    JNZ SockMon5.004A79F2    ;注册码长度不为 18h, 则 Over
004A78C8    8D95 F0FDFFFF    LEA EDX,DWORD PTR SS:[EBP-210]  ;注意 LEA 指令，说明下面子程序中会改写这个数据, 所以 d ebp-210 监视这个数据,现在
显示为 00 00 00 00
004A78CE    8B83 FC020000    MOV EAX,DWORD PTR DS:[EBX+2FC]
004A78D4    E8 3711FAFF      CALL SockMon5.00448A10    ;[ebp-210]=34 C7 B8 00 为存放机器码的地址 17F8-65A4-01A1-CB77-6439
004A78D9    8B85 F0FDFFFF    MOV EAX,DWORD PTR SS:[EBP-210]  ;EAX=机器码：17F8-65A4-01A1-CB77-6439
004A78DF    E8 3CD0F5FF      CALL SockMon5.00404920
004A78E4    8BD0             MOV EDX,EAX        ;EDX=EAX=机器码：17F8-65A4-01A1-CB77-6439
004A78E6    8D85 FFFFEFFFF    LEA EAX,DWORD PTR SS:[EBP-101]
004A78EC    E8 B718F6FF      CALL SockMon5.004091A8    ;返回后，EAX 还是机器码
004A78F1    8D95 ECFDFFFF    LEA EDX,DWORD PTR SS:[EBP-214]  ;注意 LEA 指令
004A78F7    8B83 00030000    MOV EAX,DWORD PTR DS:[EBX+300]
004A78FD    E8 0E11FAFF      CALL SockMon5.00448A10    ;返回后，[ebp-214] 为输入的注册码
004A7902    8B85 ECFDFFFF    MOV EAX,DWORD PTR SS:[EBP-214]
004A7908    E8 13D0F5FF      CALL SockMon5.00404920    ;EAX=注册码
004A790D    8BD0             MOV EDX,EAX
004A790F    8D85 FEFDFFFF    LEA EAX,DWORD PTR SS:[EBP-202]
004A7915    E8 8E18F6FF      CALL SockMon5.004091A8    ;返回后，EAX 还是注册码
004A791A    8D56 01           LEA EDX,DWORD PTR DS:[ESI+1]  ;EDX=机器码长度+1=19h
004A791D    8D85 FFFFEFFFF    LEA EAX,DWORD PTR SS:[EBP-101]  ;EAX 为机器码
004A7923    E8 7882FFFF      CALL SockMon5.0049FBA0
004A7928    8D57 01           LEA EDX,DWORD PTR DS:[EDI+1]  ;EDX=注册码长度+1=19h
004A792B    8D85 FEFDFFFF    LEA EAX,DWORD PTR SS:[EBP-202]  ;EAX 为注册码
004A7931    E8 6A82FFFF      CALL SockMon5.0049FBA0
...
004A7988    50               PUSH EAX
004A7989    68 687A4A00      PUSH SockMon5.004A7A68                 ; ASCII "SMCommVer"
004A798E    68 747A4A00      PUSH SockMon5.004A7A74                 ; ASCII "Software\\SockMon\\5.0\\SetOther"    HKEY=80000001
004A7993    E8 FC70FFFF      CALL SockMon5.0049EA94                 ; JMP 到 smcomm.SMVar_SetData
004A7998    47               INC EDI
004A7999    57               PUSH EDI
004A799A    8D95 E4FDFFFF    LEA EDX,DWORD PTR SS:[EBP-21C]
004A79A0    8B83 00030000    MOV EAX,DWORD PTR DS:[EBX+300]
004A79A6    E8 6510FAFF      CALL SockMon5.00448A10
004A79AB    8B85 E4FDFFFF    MOV EAX,DWORD PTR SS:[EBP-21C]
004A79B1    E8 6ACFF5FF      CALL SockMon5.00404920
004A79B6    50               PUSH EAX
004A79B7    68 947A4A00      PUSH SockMon5.004A7A94                 ; ASCII "SMCommDll"
004A79BC    68 747A4A00      PUSH SockMon5.004A7A74                 ; ASCII "Software\\SockMon\\5.0\\SetOther"
004A79C1    E8 CE70FFFF      CALL SockMon5.0049EA94                 ; JMP 到 smcomm.SMVar_SetData
004A79C6    8BC3             MOV EAX,EBX
004A79C8    E8 2FF0FBFF      CALL SockMon5.004669FC
004A79CD    8D95 E0FDFFFF    LEA EDX,DWORD PTR SS:[EBP-220]
004A79D3    B8 A87A4A00      MOV EAX,SockMon5.004A7AA8                ; ASCII "
HMLOGNINANCMGMEPGLPKDFPGDGLGNEPGOGDKMKCNENBLDOHNCKCLBOEMMNJLLLJMKPANHKDKBK"
004A79D8    E8 8778FFFF      CALL SockMon5.0049F264
004A79DD    8B85 E0FDFFFF    MOV EAX,DWORD PTR SS:[EBP-220]
004A79E3    E8 1CA2F9FF      CALL SockMon5.00441C04
...
```

当注册码长度为 24 位时，将注册信息记入注册表中，`HKEY_CURRENT_USER\Software\SockMon\5.0\SetOther`

`SMCommDll` ： 记录输入的注册码

`SMCommVer` ： 记录机器码

然后要求重启验证

---

## 三、重启分析

`HF 12FFC0` 设硬件断点，到 `OEP` 处。

`bp RegQueryValueExA` 设断， 中断时观察堆栈中字符串：

```
0012FB70   10001D0D  /CALL 到 RegQueryValueExA 来自 smcomm.10001D07
0012FB74   000000C8  |hKey = C8
0012FB78   004BA0B0  |ValueName = "SMCommVer"
0012FB7C   00000000  |Reserved = NULL
0012FB80   00000000  |pValueType = NULL
0012FB84   0012FCE7  |Buffer = 0012FCE7
0012FB88   0012FB98  \pBufSize = 0012FB98
0012FB8C   10002667  返回到 smcomm.10002667 来自 smcomm.10001CE0
...
0012FBB4   004B9EC6  返回到 SockMon5.004B9EC6 来自 SockMon5.0049EA8C  <=== 定位到这里
0012FBB8   004BA0BC  ASCII "Software\\SockMon\\5.0\\SetOther"
0012FBBC   004BA0B0  ASCII "SMCommVer"
```

```
004B9EAB    68 00010000      PUSH 100
004B9EB0    8D85 FFFFEFFFF    LEA EAX,DWORD PTR SS:[EBP-101]
004B9EB6    50               PUSH EAX
004B9EB7    68 B0A04B00      PUSH SockMon5.004BA0B0                 ; ASCII "SMCommVer"
004B9EBC    68 BCA04B00      PUSH SockMon5.004BA0BC                 ; ASCII "Software\\SockMon\\5.0\\SetOther"
004B9EC1    E8 C64BFEFF      CALL SockMon5.0049EA8C                 ; JMP 到 smcomm.SMVar_GetData
004B9EC6    68 00010000      PUSH 100
004B9ECB    8D85 FEFDFFFF    LEA EAX,DWORD PTR SS:[EBP-202]
004B9ED1    50               PUSH EAX
004B9ED2    68 DCA04B00      PUSH SockMon5.004BA0DC                 ; ASCII "SMCommDll"
004B9ED7    68 BCA04B00      PUSH SockMon5.004BA0BC                 ; ASCII "Software\\SockMon\\5.0\\SetOther"
004B9EDC    E8 AB4BFEFF      CALL SockMon5.0049EA8C                 ; JMP 到 smcomm.SMVar_GetData
                  ;EBP-202 上存放了注册码
004B9EE1    E8 01000000      CALL SockMon5.004B9EE7       ;进去后，就进入正常运行了
  004B9EE7    830424 06       ADD DWORD PTR SS:[ESP],6       ;[esp]=ret 地址，也就是:004B9EE6
                  ;[esp]改成了: 004B9EE6+6=004B9EEC
  004B9EEB    C3               RETN
004B9EE6  - E9 83042406      JMP 066FA36E             ;程序不会运行到这里来
004B9EEB    C3               RETN
```

---

由上来到这里：

```
004B9EEC    68 E8A04B00      PUSH SockMon5.004BA0E8                 ; ASCII "SOCKMON5.50"
004B9EF1    68 E8A04B00      PUSH SockMon5.004BA0E8                 ; ASCII "SOCKMON5.50"
```



004B9EF6    E8 C14BFEFF     CALL SockMon5.0049EABC                 ; JMP 到 smcomm.SMVer\_Check  

004B9EFB    8D83 C8050000   LEA EAX,DWORD PTR DS:\[EBX+5C8\]  

004B9F01    50             PUSH EAX  

004B9F02    68 90010000    PUSH 190  

004B9F07    E8 84CCF4FF    CALL SockMon5.00406B90                 ; JMP 到 kernel32.GetTempPathA  

004B9F0C    BA F4A04B00    MOV EDX,SockMon5.004BA0F4              ; ASCII "SMCache"  

004B9F11    8D8B C8050000  LEA ECX,DWORD PTR DS:\[EBX+5C8\]  

004B9F17    03C1           ADD EAX,ECX  

004B9F19    E8 8AF2F4FF    CALL SockMon5.004091A8  

004B9F1E    8D83 C8050000  LEA EAX,DWORD PTR DS:\[EBX+5C8\]  

004B9F24    50             PUSH EAX  

004B9F25    E8 0A4BFEFF    CALL SockMon5.0049EA34                 ; JMP 到 smcomm.SMCache\_Open  

004B9F2A    8BF0           MOV ESI,EAX  

004B9F2C    89B3 B8050000  MOV DWORD PTR DS:\[EBX+5B8\],ESI  

004B9F32    85F6           TEST ESI,ESI  

004B9F34    75 23          JNZ SHORT SockMon5.004B9F59  

004B9F36    8D95 F8FDFFFF  LEA EDX,DWORD PTR SS:\[EBP-208\]  

004B9F3C    B8 04A14B00    MOV EAX,SockMon5.004BA104             ; ASCII "

DLFPKMMLLLPKKMNPOLNNOMEMMLOPELNOOMDPMCDNGKDNDMDLMMANCPMLELNLLKGNFNGNJLBC"  

004B9F41    E8 1E53FEFF    CALL SockMon5.0049F264  

004B9F46    8B85 F8FDFFFF  MOV EAX,DWORD PTR SS:\[EBP-208\]  

004B9F4C    E8 CFA9F4FF    CALL SockMon5.00404920  

...  

------------------------------------------------------------------------------------------------------------

跟踪时，发现有大量的花指令：

004BEEE1    E8 01000000     CALL SockMon5.004BEEE7    ;NOP 掉

004BEEE6  - E9 83042406     JMP 066FF36E      ;NOP 掉

004BEEEB    C3              RETN         ;NOP 掉

004BEEEC    8B03             MOV EAX,DWORD PTR DS:\[EBX\]

...

这些花指令，可以通过脚本，一次性将其去掉

------------------------------------------------------------------------------------------------------------

跟踪时，还发现有很多 `GetTickCount`，来检测程序运行时间，单步运行时，就很容易 Over 了,如下：

004BED4B    E8 507EF4FF     CALL SockMon5.00406BA0               ; JMP 到 kernel32.GetTickCount

004BED50    2BC6             SUB EAX,ESI

004BED52    3D E8030000     CMP EAX,3E8

004BED57    0F87 96010000    JA SockMon5.004BEEF3      ;检查运行时间

004BED5D    E8 01000000     CALL SockMon5.004BED63

004BED62  - E9 83042406     JMP 066FF1EA

;

004BEDB9    E8 E27DF4FF     CALL SockMon5.00406BA0               ; JMP 到 kernel32.GetTickCount

004BEDBE    2BC6             SUB EAX,ESI

004BEDC0    3D D0070000     CMP EAX,7D0

004BEDC5    0F87 28010000    JA SockMon5.004BEEF3

004BEDCB    E8 01000000     CALL SockMon5.004BEDD1

004BEDD0  - E9 83042406     JMP 066FF258

;

这一点好办，通过脚本控制，快速运行到相应位置，即可.

由于是重启验证，不好判断注册算法的具体位置，只好在跟踪函数时，时刻观察堆栈上的数据， 若看到机器码或注册码

出现，就要注意了。

-------------------------------------------------------------------------------------------------------------

004B9EEC    68 E8A04B00     PUSH SockMon5.004BA0E8               ; ASCII "SOCKMON5.50"

004B9EF1    68 E8A04B00     PUSH SockMon5.004BA0E8               ; ASCII "SOCKMON5.50"

004B9EF6    E8 C14BFEFF     CALL SockMon5.0049EABC               ; JMP 到 smcomm.SMVer\_Check

跟进后,来到：

100089A1    55               PUSH EBP        ;EBP=17F8-65A4-01A1-CB77-6439

100089A2    8BEC             MOV EBP,ESP     

100089A4    51               PUSH ECX        ;ECX=0

100089A5    53               PUSH EBX        ;EBX=012345678901234567890123 (注册码)

100089A6    E8 C0B9FFFF      CALL smcomm.1000436B

...

这里在对注册码进行操作：

10001BCF    E8 ACF9FFFF      CALL smcomm.10001580

------------

10001580    8B5424 04        MOV EDX,DWORD PTR SS:\[ESP+4\]  ;EDX 注册码

10001584    8BC2             MOV EAX,EDX         ;EAX 注册码

10001586    56               PUSH ESI          

10001587    8D70 01          LEA ESI,DWORD PTR DS:\[EAX+1\]  ;ESI 指向注册码的第二位

1000158A    8D9B 00000000   LEA EBX,DWORD PTR DS:\[EBX\]

10001590    8A08             MOV CL,BYTE PTR DS:\[EAX\]

10001592    40               INC EAX         ;EAX 指向注册码的下一位

10001593    84C9             TEST CL,CL

10001595  ^ 75 F9            JNZ SHORT smcomm.10001590

10001597    2BC6             SUB EAX,ESI         ;EAX 得注册码长度

10001599    33C9             XOR ECX,ECX

1000159B    85C0             TEST EAX,EAX

1000159D    5E               POP ESI

1000159E    7E 0F            JLE SHORT smcomm.100015AF    ;长度为 0，Over

100015A0    803C11 20        CMP BYTE PTR DS:\[ECX+EDX\],20  ;注册码中当前字符是否为空格

100015A4    7F 04            JG SHORT smcomm.100015AA

100015A6    C60411 2D        MOV BYTE PTR DS:\[ECX+EDX\],2D  ;如果是空格，就用 "-" 号代替

100015AA    41               INC ECX

100015AB    3BC8             CMP ECX,EAX

100015AD  ^ 7C F1            JL SHORT smcomm.100015A0

100015AF    C2 0400         RETN 4

10001BCC    53               PUSH EBX

10001BCD    8BCE             MOV ECX,ESI

10001BCF    E8 ACF9FFFF      CALL smcomm.10001580    ;检查注册码长度，并将空格取代为"-"

;

10001BD4    8D86 00040000   LEA EAX,DWORD PTR DS:\[ESI+400\]  ;EAX=00-E0-4C-8E-EE

10001BDA    50               PUSH EAX

10001BDB    8BCE             MOV ECX,ESI         ;ECX=SOCKMON5.50

10001BDD    E8 9EF9FFFF      CALL smcomm.10001580    ;同上

;

再往下跟踪，就来到了网络验证部分

### 四、网络验证：

10001819    6A 00            PUSH 0

1000181B    BF 01000000      MOV EDI,1

10001820    57               PUSH EDI

10001821    6A 02            PUSH 2

10001823    897C24 30        MOV DWORD PTR SS:\[ESP+30\],EDI

10001827    E8 6A190000     CALL smcomm.10003196              ; JMP 到 WS2\_32.socket

1000182C    8BF0             MOV ESI,EAX

1000182E    83FE FF         CMP ESI,-1

10001831    0F84 61010000    JE smcomm.10001998

;

10001837    8D5424 24        LEA EDX,DWORD PTR SS:\[ESP+24\]

1000183B    52               PUSH EDX

1000183C    68 7E660480     PUSH 8004667E

10001841    56               PUSH ESI

10001842    E8 49190000     CALL smcomm.10003190              ; JMP 到 WS2\_32.ioctlsocket

10001847    83F8 FF         CMP EAX,-1

1000184A    0F84 42010000    JE smcomm.10001992

10001850    68 6CAC0010     PUSH smcomm.1000AC6C              ; ASCII "WWW.CNASM.COM"

10001855    E8 30190000     CALL smcomm.1000318A              ; JMP 到 WS2\_32.gethostbyname

1000185A    8BD8             MOV EBX,EAX

1000185C    85DB             TEST EBX,EBX

1000185E    0F84 2E010000    JE smcomm.10001992

;

10001864    33C0             XOR EAX,EAX

10001866    894424 14        MOV DWORD PTR SS:\[ESP+14\],EAX

1000186A    894424 18        MOV DWORD PTR SS:\[ESP+18\],EAX

1000186E    894424 1C        MOV DWORD PTR SS:\[ESP+1C\],EAX

10001872    6A 50            PUSH 50

10001874    894424 24        MOV DWORD PTR SS:\[ESP+24\],EAX

10001878    66:C74424 18 02>MOV WORD PTR SS:\[ESP+18\],2

1000187F    E8 00190000     CALL smcomm.10003184              ; JMP 到 WS2\_32.ntohs

10001884    66:894424 16    MOV WORD PTR SS:\[ESP+16\],AX

;

10001889    8B4B 0C         MOV ECX,DWORD PTR DS:\[EBX+C\]

1000188C    8B11             MOV EDX,DWORD PTR DS:\[ECX\]

1000188E    8B02             MOV EAX,DWORD PTR DS:\[EDX\]

10001890    6A 10            PUSH 10

10001892    8D4C24 18        LEA ECX,DWORD PTR SS:\[ESP+18\]

10001896    51               PUSH ECX

10001897    56               PUSH ESI

10001898    894424 24        MOV DWORD PTR SS:\[ESP+24\],EAX

1000189C    E8 DD180000     CALL smcomm.1000317E              ; JMP 到 WS2\_32.connect

;

100018A1    8D5424 28        LEA EDX,DWORD PTR SS:\[ESP+28\]

100018A5    52               PUSH EDX

100018A6    8D8424 44020000 LEA EAX,DWORD PTR SS:\[ESP+244\]

100018AD    50               PUSH EAX

100018AE    8D4C24 38        LEA ECX,DWORD PTR SS:\[ESP+38\]

100018B2    51               PUSH ECX

100018B3    6A 00            PUSH 0

100018B5    6A 00            PUSH 0

100018B7    C74424 40 00000>MOV DWORD PTR SS:\[ESP+40\],0



## 汇编代码分析

```
100018BF C74424 3C 03000>MOV DWORD PTR SS:[ESP+3C],3  
100018C7 89B424 50010000 MOV DWORD PTR SS:[ESP+150],ESI  
100018CE 89BC24 4C010000 MOV DWORD PTR SS:[ESP+14C],EDI  
100018D5 897424 48       MOV DWORD PTR SS:[ESP+48],ESI  
100018D9 897C24 44       MOV DWORD PTR SS:[ESP+44],EDI  
100018DD 89B424 58020000 MOV DWORD PTR SS:[ESP+258],ESI  
100018E4 89BC24 54020000 MOV DWORD PTR SS:[ESP+254],EDI  
100018EB E8 88180000     CALL smcomm.10003178 ; JMP 到 WS2_32.select  
;  
1000190B 6A 00          PUSH 0  
1000190D 55             PUSH EBP  
1000190E 8D9424 E0040000 LEA EDX,DWORD PTR SS:[ESP+4E0]  
10001915 52             PUSH EDX  
10001916 56             PUSH ESI  
10001917 E8 56180000    CALL smcomm.10003172 ; JMP 到 WS2_32.send  
1000191C E8 4B180000    CALL smcomm.1000316C ; JMP 到 WS2_32.WSAGetLastError  
;  
10001921 6A 00          PUSH 0  
10001923 8D8424 44020000 LEA EAX,DWORD PTR SS:[ESP+244]  
1000192A 50             PUSH EAX  
1000192B 6A 00          PUSH 0  
1000192D 8D8C24 44010000 LEA ECX,DWORD PTR SS:[ESP+144]  
10001934 51             PUSH ECX  
10001935 6A 00          PUSH 0  
10001937 89B424 50010000 MOV DWORD PTR SS:[ESP+150],ESI  
1000193E 89BC24 4C010000 MOV DWORD PTR SS:[ESP+14C],EDI  
10001945 897424 48       MOV DWORD PTR SS:[ESP+48],ESI  
10001949 897C24 44       MOV DWORD PTR SS:[ESP+44],EDI  
1000194D 89B424 58020000 MOV DWORD PTR SS:[ESP+258],ESI  
10001954 89BC24 54020000 MOV DWORD PTR SS:[ESP+254],EDI  
1000195B E8 18180000    CALL smcomm.10003178 ; JMP 到 WS2_32.select  
10001960 39BC24 38010000 CMP DWORD PTR SS:[ESP+138],EDI  
10001967 75 29          JNZ SHORT smcomm.10001992  
10001969 33C0           XOR EAX,EAX  
1000196B 50             PUSH EAX  
1000196C 68 00100000   PUSH 1000  
10001971 8D9424 E0040000 LEA EDX,DWORD PTR SS:[ESP+4E0]  
10001978 52             PUSH EDX  
10001979 B9 00040000   MOV ECX,400  
1000197E 8DBC24 E4040000 LEA EDI,DWORD PTR SS:[ESP+4E4]  
10001985 56             PUSH ESI  
10001986 F3:AB          REP STOS DWORD PTR ES:[EDI]  
10001988 E8 D9170000   CALL smcomm.10003166 ; JMP 到 wsock32.recv  
1000198D E8 DA170000   CALL smcomm.1000316C ; JMP 到 WS2_32.WSAGetLastError  
;  
10001992 56             PUSH ESI  
10001993 E8 C8170000   CALL smcomm.10003160 ; JMP 到 WS2_32.closesocket  
10001998 E8 BD170000   CALL smcomm.1000315A ; JMP 到 WS2_32.WSACleanup  
```

### 发送的数据

| 地址 | 值 | 描述 |
| :--- | :--- | :--- |
| `0012DE60` | `000000F0` | `Socket` = `F0` |
| `0012DE64` | `0012E348` | `Data` = `0012E348` |
| `0012DE68` | `000000E2` | `DataSize` = `E2` (226.) |
| `0012DE6C` | `00000000` | `Flags` = `0` |

```
0012E348  47 45 54 20 2F 53 6F 63 6B 4D 6F 6E 2F 55 70 64  GET /SockMon/Upd  
0012E358  61 74 65 2E 61 73 70 3F 54 49 4D 3D 32 30 30 38  ate.asp?TIM=2008  
0012E368  2D 32 2D 31 32 2B 32 32 3A 32 31 3A 35 36 26 5A  -2-12+22:21:56&Z  
0012E378  43 48 3D 31 37 46 38 2D 36 35 41 34 2D 30 31 41  CH=17F8-65A4-01A  
0012E388  31 2D 43 42 37 37 2D 36 34 33 39 26 5A 43 4D 3D  1-CB77-6439&ZCM=  
0012E398  31 32 33 34 35 36 37 38 39 30 31 32 33 34 35 36  1234567890123456  
0012E3A8  37 38 39 30 31 32 33 34 26 4D 41 43 3D 30 30 2D  78901234&MAC=00-  
0012E3B8  45 30 2D 34 43 2D 38 45 2D 45 45 26 55 53 52 3D  E0-4C-8E-EE&USR=  
0012E3C8  63 78 67 26 43 4D 50 3D 42 49 4C 4C 47 41 54 45  cxg&CMP=BILLGATE  
0012E3D8  53 26 53 59 53 3D 57 69 6E 58 70 2C 53 65 72 76  S&SYS=WinXp,Serv  
0012E3E8  69 63 65 2D 50 61 63 6B 2D 31 26 56 45 52 3D 53  ice-Pack-1&VER=S  
0012E3F8  4F 43 4B 4D 4F 4E 35 2E 35 30 20 20 20 48 54 54  OCKMON5.50   HTT  
0012E408  50 2F 31 2E 31 0D 0A 48 6F 73 74 3A 20 57 57 57  P/1.1..Host: WWW  
0012E418  2E 43 4E 41 53 4D 2E 43 4F 4D 0D 0A 0D 0A 0D 0A  .CNASM.COM......  
0012E428  0D 0A 00 77 D8 13 15 00 38 27 F6 77 C7 27 F6 77  ...w?.8'?  
```

### 接收到的数据

| 地址 | 值 | 描述 |
| :--- | :--- | :--- |
| `0012DE60` | `000000F0` | `Socket` = `F0` |
| `0012DE64` | `0012E348` | `Buffer` = `0012E348` |
| `0012DE68` | `00001000` | `BufSize` = `1000` (4096.) |
| `0012DE6C` | `00000000` | `Flags` = `0` |

```
0012E348  48 54 54 50 2F 31 2E 31 20 32 30 30 20 4F 4B 0D  HTTP/1.1 200 OK.  
0012E358  0A 44 61 74 65 3A 20 46 72 69 2C 20 32 32 20 46  .Date: Fri, 22 F  
0012E368  65 62 20 32 30 30 38 20 31 34 3A 30 34 3A 32 39  eb 2008 14:04:29  
0012E378  20 47 4D 54 0D 0A 53 65 72 76 65 72 3A 20 4D 69   GMT..Server: Mi  
0012E388  63 72 6F 73 6F 66 74 2D 49 49 53 2F 36 2E 30 0D  crosoft-IIS/6.0.  
0012E398  0A 58 2D 50 6F 77 65 72 65 64 2D 42 79 3A 20 41  .X-Powered-By: A  
0012E3A8  53 50 2E 4E 45 54 0D 0A 43 6F 6E 74 65 6E 74 2D  SP.NET..Content-  
0012E3B8  4C 65 6E 67 74 68 3A 20 32 0D 0A 43 6F 6E 74 65  Length: 2..Conte  
0012E3C8  6E 74 2D 54 79 70 65 3A 20 74 65 78 74 2F 68 74  nt-Type: text/ht  
0012E3D8  6D 6C 0D 0A 53 65 74 2D 43 6F 6F 6B 69 65 3A 20  ml..Set-Cookie:  
0012E3E8  41 53 50 53 45 53 53 49 4F 4E 49 44 53 51 42 44  ASPSESSIONIDSQBD  
0012E3F8  43 43 52 41 3D 4A 4C 50 46 4A 48 44 44 4C 4D 45  CCRA=JLPFJHDDLME  
0012E408  49 47 42 46 48 43 41 48 47 44 49 45 4D 3B 20 70  IGBFHCAGHDIEM; p  
0012E418  61 74 68 3D 2F 0D 0A 43 61 63 68 65 2D 63 6F 6E  ath=/..Cache-con  
0012E428  74 72 6F 6C 3A 20 70 72 69 76 61 74 65 0D 0A 0D  trol: private...  
0012E438  0A 4F 4B 00 00 00 00 00 00 00 00 00 00 00 00 00  .OK.............  
```

## 调试过程

继续往下跟，跟来跟去，也不知程序是在哪里开始分析这一块数据的，试图在这一段内存上设下内存访问断点，结果程序老是出现异常，跟得甚是郁闷。

-----------------------------------------------------------------------------------------------------------------------------------------------------

### 四、另辟蹊径

没办法，现在只好通过注册窗口，来寻找突破点了：

`bp CreateWindowExA` 下断，即可对注册窗口弹出过程拦截到：

```
0044C421    FF91 9C000000   CALL DWORD PTR DS:[ECX+9C]             ; SockMon5.0044C498  弹出注册窗口  
0044C427    83BE 80010000 0>CMP DWORD PTR DS:[ESI+180],0  
0044C42E    75 05           JNZ SHORT SockMon5.0044C435  
0044C498    53             PUSH EBX  
...
0044C4CA    50             PUSH EAX  
0044C4CB    E8 50ABFBFF    CALL SockMon5.00407020                 ; JMP 到 user32.CreateWindowExA  
...
```

第一次是通过 `CreateWindowExA()` 弹出窗口，但随后几次弹出，通过 `bp CreateWindowExA` 却拦不下来。

可能是窗口隐藏了，并没有卸载，`bp ShowWindow` 一试，果然拦下。

附近函数再跟进，跟踪来到：

```
00466515   .  80B8 2F020000>CMP BYTE PTR DS:[EAX+22F],1  
0046651C   .  0F85 B5000000 JNZ SockMon5.004665D7  
...
004665D7   > \8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]  
004665DA   .  0FB680 2B0200>MOVZX EAX,BYTE PTR DS:[EAX+22B]  
004665E1   .  8B0485 CC0D4C>MOV EAX,DWORD PTR DS:[EAX*4+4C0DCC]  
004665E8   .  50            PUSH EAX  
004665E9   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]  
004665EC   .  E8 FF8BFEFF   CALL SockMon5.0044F1F0  
004665F1    50             PUSH EAX  
004665F2    E8 390FFAFF     CALL SockMon5.00407530                 ; JMP 到 user32.ShowWindow  ; 注册窗口显示控制  
```

; 下面是主窗口显示控制

```
00465D06   .  50            PUSH EAX                                 ; |hWnd  
00465D07   .  E8 2418FAFF   CALL SockMon5.00407530                 ; \ShowWindow    ; 主窗口显示控制  
```

`ShowWindow` 返回后，返回到下面的：`00449E6C`

```
00449E69  >|.  FF51 EC      CALL DWORD PTR DS:[ECX-14]  
00449E6C  |>   5F           POP EDI  
00449E6D  |.   5E           POP ESI  
00449E6E  |.   5B           POP EBX  
00449E6F  |.   8BE5         MOV ESP,EBP  
00449E71  |.   5D           POP EBP  
00449E72  \\.   C3           RETN  
```

```
0012F9B8  /0012FAE0  
0012F9BC  |00449E6C  返回到 SockMon5.00449E6C  
0012F9C0  |00B4A038  ASCII "TuJ"  
0012F9C4  |0012FB34  
```

由于注册窗口是以 `CreateWindow` 和 `ShowWindow` 方式进行，而不是以 `DoModal` 方式进行，所以，应该就会将主窗口 `EnableWindow` `FALSE`。

那么我们就可以将 `EnableWindow` `FALSE` 改为 `Enable` `TRUE`，然后在显示注册窗口时，改为 `ShowWindow` `FALSE`。

这样一改，就不会再每过一分钟就弹出一个注册窗口，但还是有 5 分钟的总时间限制。

进行 `bp EnableWindow` 拦截，可以得到：(会有两次调用)

```
00460BDC    50             PUSH EAX
```



`00460BDD    E8 1665FAFF     CALL SockMon5.004070F8             ; JMP 到 user32.EnumThreadWindows`

返回：

`00466BA9    33C0            XOR EAX,EAX`

`00466BAB    E8 D89FFFFF     CALL SockMon5.00460B88`

`;`

到函数头：

`00466AB4  /.  55            PUSH EBP`

到父函数：

`0043ABA4   .  53            PUSH EBX`

`0043ABA5   .  66:8378 3A 00 CMP WORD PTR DS:[EAX+3A],0`

`0043ABAA   .  74 0A         JE SHORT SockMon5.0043ABB6`

`0043ABAC   .  8BD8          MOV EBX,EAX`

`0043ABAE   .  8BD0          MOV EDX,EAX`

`0043ABB0   .  8B43 3C       MOV EAX,DWORD PTR DS:[EBX+3C]`

`0043ABB3   .  FF53 38       CALL DWORD PTR DS:[EBX+38]`

`0043ABB6   >  5B            POP EBX`

`0043ABB7   .  C3            RETN`

将上面的 `0043ABAA   .  74 0A         JE SHORT SockMon5.0043ABB6` 改为：

`0043ABAA   /EB 0A           JMP SHORT SockMon5.0043ABB6`

发现，OK，居然全破了，`SockMon2005` 状态栏上的计时停止了，再没有注册窗口弹出，5 分钟过后也再不会退出程序。

爆破成功。也就是将 `0043ABAA` 处的数值 `740A` 改为 `EB0A` 即可。

如果不脱壳，可以写一个内存补丁程序，对 `0043ABAA` 处的数值 进行修正：

带壳程序入口点：  `00504001  PUSHAD`

`OD` 刚加载时，`0043ABAA` 处的代码不对， 单步运行壳中的程序，当壳运行到 `00504189` 时，`0043ABAA` 处的代码已经解出。

所以，可以在 `00504189` 处，对程序代码进行修正.

-----------------------------------------------------------------------------------------------------------------------------------------------------

## 五. 内存补丁程序源码：

```asm
;==========================================================================================

;
;程序说明： SockMon5.exe 内存补丁程序

;
;工作原理：本程序直接加载带壳的 SockMon5.exe 程序，当外壳解完程序代码后，将 0043ABAA 处的

;
;          指令由 740A 改为 EB0A 即可, 也就是 JNZ 改为 JMP。
;
;运行要求：将编译后的内存注册机放 SockMon5.exe 程序的同一目录下，然后运行内存注册机即可启动
;          SockMon5.exe, 没有时间限制。
;
;==========================================================================================

.386
.model     flat,stdcall
option     casemap:none
include     c:\radasm\masm32\include\windows.inc
include   c:\radasm\masm32\include\kernel32.inc
include    c:\radasm\masm32\include\user32.inc
include   c:\radasm\masm32\include\masm32.inc
include   c:\radasm\masm32\Macros\Strings.mac
includelib   c:\radasm\masm32\lib\kernel32.lib
includelib   c:\radasm\masm32\lib\masm32.lib
includelib   c:\radasm\masm32\lib\user32.lib
.code
;==================================================================================================
; 修改目标进程代码(只更改一个 BYTE 值),返回值为修改之前的值 
WriteProcessByte proc hProcess:DWORD,Address:DWORD,NewByte:DWORD
  LOCAL  dwTemp,OldByte
  .if  Address>=80000000h || Address == 0        ;0x80000000 以上的地址为系统共有区域，不可以修改
    mov  eax,NewByte
    ret
  .endif
  invoke  ReadProcessMemory,hProcess,Address,addr OldByte,1,addr dwTemp  ;取得原来的代码
  mov  ebx,NewByte
  .if  !eax || OldByte==ebx
    mov  eax,OldByte    ;原来的代码和准备修改的代码相同，没有必要再修改   
    ret
  .endif
  invoke  WriteProcessMemory,hProcess,Address,addr NewByte,1,addr dwTemp  ;修改目标代码
  mov  eax,OldByte   
  ret
WriteProcessByte endp
;==================================================================================================
; 修改目标进程代码(更改数量由参数定义)
WriteProcessCode proc hProcess:DWORD,Address:DWORD,lpBuf:DWORD,nSize:DWORD
  LOCAL  dwTemp   
  .if  Address>=80000000h || Address == 0        ;0x80000000 以上的地址为系统共有区域，不可以修改
    xor   eax,eax
    ret
  .endif
  invoke  WriteProcessMemory,hProcess,Address,lpBuf,nSize,addr dwTemp  ;修改目标代码
  mov  ebx,nSize
  xor  eax,eax   
  .if  ebx == dwTemp
    inc  eax     
  .endif
  ret
WriteProcessCode endp
;==================================================================================================
;
;调试 API 函数  参数：DbgSoftPath - 目标程序的路径(地址)
;
;==================================================================================================
DbgSoft proc DbgSoftPath
  LOCAL   startinfo:STARTUPINFO
  LOCAL   pi:PROCESS_INFORMATION
  LOCAL   DBEvent:DEBUG_EVENT
  LOCAL  stCT:CONTEXT
  LOCAL  hProcess,OldByte,NewCode
  LOCAL  OEPAddr,ObjAddr1,BkPointAddr   
  LOCAL  PatchAddr1      ; 代码补丁处地址
  mov  OEPAddr,00504001h    ; OEP 处地址
  mov  ObjAddr1,00504189h    ; 让被调程序运行到目标地址上停下的地址
  mov  PatchAddr1,0043ABAAh    ; 代码补丁处地址
  ;以调试方式运行目标程序
  invoke GetStartupInfo,addr startinfo    
  invoke CreateProcess, DbgSoftPath, NULL, NULL, NULL, FALSE, DEBUG_PROCESS or DEBUG_ONLY_THIS_PROCESS,NULL, NULL, addr startinfo, addr pi
  .if  eax == 0
    invoke  MessageBox,NULL,$CTA0("无法运行目标程序."),$CTA0("错误"),MB_OK or MB_ICONINFORMATION
    ret
  .endif 
  .while   TRUE  
       invoke WaitForDebugEvent, addr DBEvent, INFINITE 
       .if DBEvent.dwDebugEventCode==EXIT_PROCESS_DEBUG_EVENT     ;被调试进程退出        
           .break
       .elseif DBEvent.dwDebugEventCode==CREATE_PROCESS_DEBUG_EVENT   ;被调试进程创建
      push  DBEvent.u.CreateProcessInfo.hProcess
      pop  hProcess
      invoke  WriteProcessByte,hProcess,OEPAddr,0CCh    ;在被调试程序的 OEP 处设置一个断点
      mov  OldByte,eax
       .elseif DBEvent.dwDebugEventCode==EXCEPTION_DEBUG_EVENT   ;被调试进程出现异常事件
           ;得到断点地址
           push  DBEvent.u.Exception.pExceptionRecord.ExceptionAddress
           pop  BkPointAddr           
           ;如果是断点事件异常, 让被调试进程继续运行
           .if DBEvent.u.Exception.pExceptionRecord.ExceptionCode==EXCEPTION_BREAKPOINT    
             mov  eax,BkPointAddr
             .if  eax==OEPAddr  ;如果断点在 OEP 处
               invoke  WriteProcessByte,hProcess,OEPAddr,OldByte    ;恢复 OEP 处代码
               invoke  WriteProcessByte,hProcess,ObjAddr1,0cch     ;在目标地址处设置断点
               mov  OldByte,eax
               mov  stCT.ContextFlags,CONTEXT_FULL
               invoke  GetThreadContext,pi.hThread,addr stCT     ;取得线程上下文
               dec  stCT.regEip            
               invoke  SetThreadContext,pi.hThread,addr stCT   
             .elseif  eax==ObjAddr1  ;如果断在目标地址处           
               invoke  WriteProcessByte,hProcess,ObjAddr1,OldByte    ;修正目标地址处的代码
               mov  stCT.ContextFlags,CONTEXT_FULL
               invoke  GetThreadContext,pi.hThread,addr stCT     ;取得线程上下文
               dec  stCT.regEip            
               invoke  SetThreadContext,pi.hThread,addr stCT
               mov  NewCode,90900aebh           ;新代码 eb 0a 也就是改为: jmp ***
               invoke  WriteProcessCode,hProcess,PatchAddr1,addr NewCode,2  ;对被丁地址处，进行修改代码(2 个字节)           
             .endif
           .endif           
       .endif         
    invoke ContinueDebugEvent, DBEvent.dwProcessId, DBEvent.dwThreadId, DBG_CONTINUE  ;DBG_EXCEPTION_NOT_HANDLED
  .endw 
  invoke CloseHandle,pi.hProcess 
  invoke CloseHandle,pi.hThread 
  ret
DbgSoft endp
DebugRunProgramme proc lpFileName
  LOCAL  szDbgExePath[256]:byte
  invoke  GetAppPath,addr szDbgExePath  ;获得应用程序路径(不含执行文件名)
  invoke  lstrcat,addr szDbgExePath,lpFileName
  invoke  DbgSoft,addr szDbgExePath
  ret
DebugRunProgramme endp
;==================================================================================================
;
;程序入口
;
;==================================================================================================
start:   
         invoke  DebugRunProgramme,$CTA0("SockMon5.exe")      ;调试运行目标程序
        invoke ExitProcess,NULL
end start
```

-----------------------------------------------------------------------------------------------------------------------------------------------------

## 六、后记

这个软件虽然误打误撞，给破了，但感觉还是很遗憾，没有能力从网络数据包分析的途径来分析破解它。此次破解，只能说是一个幸运！

哎，革命尚末成功，同志仍需努力啊！

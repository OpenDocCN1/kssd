
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

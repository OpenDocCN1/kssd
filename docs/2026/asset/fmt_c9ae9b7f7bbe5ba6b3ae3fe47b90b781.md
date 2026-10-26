
# [原创]XX桌面日历算法分析

- 【文章标题】: 飞雪桌面日历的算法分析
- 【文章作者】: dewar
- 【作者主页】: 无
- 【软件名称】: 飞雪桌面日历2.0
- 【下载地址】: 自己搜索下载
- 【加壳方式】: UPX
- 【保护方式】: 注册码重启验证
- 【编写语言】: VB
- 【使用工具】: OD
- 【操作平台】: WINXP
- 【软件介绍】: 集合了:万年历、时钟、定时运行、定时关机等
- 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 详细过程

该软件小巧而强大！集合了以下超多功能:万年历、时钟、定时运行、定时关机(2000/XP关机仅需3秒!)、限时用机、休息提醒(可锁定系统)、备忘录、系统热键、世界时间、光驱控制、定期清理电脑、语音报时、整点/半点报时等,并支持自定义软件皮肤，能以四种界面存在,即：日历、挂历、时钟、迷你栏。但其是共享版，动不动就弹出“您还未注册”的提示，让人看着就烦～～于是只好将它搬上手术台^\_^

1.先试用该程序，发现注册码的验证为重启验证。

2.PEID查壳，原来是UPX的壳，这个壳很基本，三下五除二脱了，脱了之后再查壳，无壳了是VB编写的程序，试运行，轰的一声，电脑关机了。我晕，好恶劣的行径，看来是非破不可了，重启电脑再来。

3.既然会关机，就说明有暗桩，多半是检查文件的大小，OD载入脱壳之后的程序，在所有对模块`MSVBVM60.DLL`的输出函数`rtcFileLen`的调用上下断（一共8处）。F9运行，中断在如下的地方：

```
........
00531B57   .  FF15 C8104000 CALL    DWORD PTR DS:[<&MSVBVM60.rtcRand>;  产生一个随机数
00531B5D   .  D80D 0C394000 FMUL    DWORD PTR DS:[40390C]            ;  ×10
00531B63   .  FF15 F0124000 CALL    DWORD PTR DS:[<&MSVBVM60.__vbaR8>;  转为整数
00531B69   .  8BF8          MOV     EDI, EAX
00531B6B   .  8D4D B8       LEA     ECX, DWORD PTR SS:[EBP-48]
00531B6E   .  FFD6          CALL    ESI
00531B70   .  0FBFC7        MOVSX   EAX, DI
00531B73   .  83F8 09       CMP     EAX, 9                           ;  Switch (cases 0..9)
00531B76   .  0F87 A5020000 JA      00531E21
00531B7C   .  FF2485 8C1E53>JMP     DWORD PTR DS:[EAX*4+531E8C]
00531B83   >  8B4D E0       MOV     ECX, DWORD PTR SS:[EBP-20]       ;  Case 1 of switch 00531B73
00531B86   .  51            PUSH    ECX
00531B87   .  E9 3C010000   JMP     00531CC8
00531B8C   >  8B55 E0       MOV     EDX, DWORD PTR SS:[EBP-20]       ;  Case 2 of switch 00531B73
00531B8F   .  52            PUSH    EDX
00531B90   .  FF15 9C124000 CALL    DWORD PTR DS:[<&MSVBVM60.rtcFile>;  检查运行程序的大小，下同
00531B96   .  3D 004E0500   CMP     EAX, 54E00                       ;  与0x54e00比较，下同
00531B9B   .  0F84 80020000 JE      00531E21                         ;  相等就跳向正确的代码，下同//改JE为JMP
00531BA1   .  8B45 D8       MOV     EAX, DWORD PTR SS:[EBP-28]
00531BA4   .  85C0          TEST    EAX, EAX
00531BA6   .  75 12         JNZ     SHORT 00531BBA
00531BA8   .  8D45 D8       LEA     EAX, DWORD PTR SS:[EBP-28]
00531BAB   .  50            PUSH    EAX
00531BAC   .  68 A8784000   PUSH    004078A8
00531BB1   .  FF15 58124000 CALL    DWORD PTR DS:[<&MSVBVM60.__vbaNe>;  MSVBVM60.__vbaNew2
00531BB7   .  8B45 D8       MOV     EAX, DWORD PTR SS:[EBP-28]
00531BBA   >  8BF0          MOV     ESI, EAX
00531BBC   .  8B08          MOV     ECX, DWORD PTR DS:[EAX]
00531BBE   .  50            PUSH    EAX
00531BBF   .  FF51 24       CALL    DWORD PTR DS:[ECX+24]           ;  否则关机～！
00531BC2   .  DBE2          FCLEX
00531BC4   .  85C0          TEST    EAX, EAX
00531BC6   .  7D 0F         JGE     SHORT 00531BD7
00531BC8   .  6A 24         PUSH    24
00531BCA   .  68 64C24100   PUSH    0041C264
00531BCF   .  56            PUSH    ESI
00531BD0   .  50            PUSH    EAX
00531BD1   .  FF15 A8104000 CALL    DWORD PTR DS:[<&MSVBVM60.__vbaHr>;  MSVBVM60.__vbaHresultCheckObj
00531BD7   >  8B45 D8       MOV     EAX, DWORD PTR SS:[EBP-28]
00531BDA   .  85C0          TEST    EAX, EAX
00531BDC   .  75 12         JNZ     SHORT 00531BF0
00531BDE   .  8D55 D8       LEA     EDX, DWORD PTR SS:[EBP-28]
00531BE1   .  52            PUSH    EDX
00531BE2   .  68 A8784000   PUSH    004078A8
00531BE7   .  FF15 58124000 CALL    DWORD PTR DS:[<&MSVBVM60.__vbaNe>;  MSVBVM60.__vbaNew2
00531BED   .  8B45 D8       MOV     EAX, DWORD PTR SS:[EBP-28]
00531BF0   >  8BF0          MOV     ESI, EAX
00531BF2   .  8B08          MOV     ECX, DWORD PTR DS:[EAX]
00531BF4   .  50            PUSH    EAX
00531BF5   .  FF51 20       CALL    DWORD PTR DS:[ECX+20]
00531BF8   .  DBE2          FCLEX
00531BFA   .  85C0          TEST    EAX, EAX
00531BFC   .  0F8D 1F020000 JGE     00531E21
00531C02   .  6A 20         PUSH    20
00531C04   .  68 64C24100   PUSH    0041C264
00531C09   .  56            PUSH    ESI
00531C0A   .  50            PUSH    EAX
00531C0B   .  FF15 A8104000 CALL    DWORD PTR DS:[<&MSVBVM60.__vbaHr>;  MSVBVM60.__vbaHresultCheckObj
00531C11   .  E9 0B020000   JMP     00531E21
00531C16   >  8B55 E0       MOV     EDX, DWORD PTR SS:[EBP-20]       ;  Case 3 of switch 00531B73
00531C19   .  52            PUSH    EDX
00531C1A   .  FF15 9C124000 CALL    DWORD PTR DS:[<&MSVBVM60.rtcFile>;  MSVBVM60.rtcFileLen
00531C20   .  3D 004E0500   CMP     EAX, 54E00
00531C25   .  0F84 F6010000 JE      00531E21                         ;  改JE为JMP
00531C2B   .  8B45 D8       MOV     EAX, DWORD PTR SS:[EBP-28]
00531C2E   .  85C0          TEST    EAX, EAX
00531C30   .  75 12         JNZ     SHORT 00531C44
00531C32   .  8D45 D8       LEA     EAX, DWORD PTR SS:[EBP-28]
00531C35   .  50            PUSH    EAX
00531C36   .  68 A8784000   PUSH    004078A8
00531C3B   .  FF15 58124000 CALL    DWORD PTR DS:[<&MSVBVM60.__vbaNe>;  MSVBVM60.__vbaNew2
00531C41   .  8B45 D8       MOV     EAX, DWORD PTR SS:[EBP-28]
00531C44   >  8BF0          MOV     ESI, EAX
00531C46   .  8B08          MOV     ECX, DWORD PTR DS:[EAX]
00531C48   .  50            PUSH    EAX
00531C49   .  FF51 24       CALL    DWORD PTR DS:[ECX+24]
00531C4C   .  DBE2          FCLEX
00531C4E   .  85C0          TEST    EAX, EAX
00531C50   .  7D 0F         JGE     SHORT 00531C61
00531C52   .  6A 24         PUSH    24
00531C54   .  68 64C24100   PUSH    0041C264
00531C59   .  56            PUSH    ESI
00531C5A   .  50            PUSH    EAX
00531C5B   .  FF15 A8104000 CALL    DWORD PTR DS:[<&MSVBVM60.__vbaHr>;  MSVBVM60.__vbaHresultCheckObj
00531C61   >  8B45 D8       MOV     EAX, DWORD PTR SS:[EBP-28]
00531C64   .  85C0          TEST    EAX, EAX
00531C66   .  75 36         JNZ     SHORT 00531C9E
00531C68   .  EB 22         JMP     SHORT 00531C8C
00531C6A   >  8B45 E0       MOV     EAX, DWORD PTR SS:[EBP-20]       ;  Case 5 of switch 00531B73
00531C6D   .  50            PUSH    EAX
00531C6E   .  EB 58         JMP     SHORT 00531CC8
00531C70   >  8B4D E0       MOV     ECX, DWORD PTR SS:[EBP-20]       ;  Case 6 of switch 00531B73
00531C73   .  51            PUSH    ECX
00531C74   .  FF15 9C124000 CALL    DWORD PTR DS:[<&MSVBVM60.rtcFile>;  MSVBVM60.rtcFileLen
00531C7A   .  3D 004E0500   CMP     EAX, 54E00
00531C7F   .  0F84 9C010000 JE      00531E21                         ;  改JE为JMP
00531C85   .  8B45 D8       MOV     EAX, DWORD PTR SS:[EBP-28]
00531C88   .  85C0          TEST    EAX, EAX
00531C8A   .  75 12         JNZ     SHORT 00531C9E
00531C8C   >  8D55 D8       LEA     EDX, DWORD PTR SS:[EBP-28]
00531C8F   .  52            PUSH    EDX
00531C90   .  68 A8784000   PUSH    004078A8
00531C95   .  FF15 58124000 CALL    DWORD PTR DS:[<&MSVBVM60.__vbaNe>;  MSVBVM60.__vbaNew2
00531C9B   .  8B45 D8       MOV     EAX, DWORD PTR SS:[EBP-28]
00531C9E   >  8BF0          MOV     ESI, EAX
00531CA0   .  8B08          MOV     ECX, DWORD PTR DS:[EAX]
00531CA2   .  50            PUSH    EAX
00531CA3   .  FF51 20       CALL    DWORD PTR DS:[ECX+20]
00531CA6   .  DBE2          FCLEX
00531CA8   .  85C0          TEST    EAX, EAX
00531CAA   .  0F8D 71010000 JGE     00531E21
00531CB0   .  6A 20         PUSH    20
00531CB2   .  68 64C24100   PUSH    0041C264
00531CB7   .  56            PUSH    ESI
00531CB8   .  50            PUSH    EAX
00531CB9   .  FF15 A8104000 CALL    DWORD PTR DS:[<&MSVBVM60.__vbaHr>;  MSVBVM60.__vbaHresultCheckObj
00531CBF   .  E9 5D010000   JMP     00531E21
00531CC4   >  8B55 E0       MOV     EDX, DWORD PTR SS:[EBP-20]       ;  Cases 4,7 of switch 00531B73
00531CC7   .  52            PUSH    EDX
```


# AcProtect 之 EMbedded 和 RSA protect 探讨

【脱文作者】 simonzh2000

【使用工具】 Peid0.91, Ollydbg1.09d(反Antidbg版), ImportREC1.60, LordPE, UltraEdit

【破解平台】 Win2000

【软件名称】 XieXie Master 1.0.10

【软件简介】 一个法国人的电脑中国象棋,棋力超强,接近大师级水平

【软件大小】 877K

【加壳方式】 ACProtect, 使用 Embedded Protect 和 RSA Protect 技术

【破解声明】 在春节前拿到这个软件时,我还是一个脱壳菜鸟, 连 UPX 手动脱壳也不会, 这种加密壳就别提了, 只好上论坛向各位大侠请教.

感谢 heXer , jingulong, fly 等多位大侠的帮助, 帮我脱了壳, 尤其是 heXer 大侠, 还帮我成功破解, 但遗憾的是都没写脱壳笔记.

俺只好自己动手尝试, 春节的时候我看了 DFCG 的 13篇手动脱壳入门教程, 11篇手动脱壳进价教程, 总算有点进步了, 感谢 DFCG 的各位大侠.

下面是我的一点心得，愿与大家分享, 同时有许多问题向大家请教.

本笔记只用于学习交流.

未脱壳原程序

http://tongtian.net/pediybbs/viewtopic.php?t=3336

用`peid`查壳，`UPX 0.89.6 - 1.02 / 1.05 - 1.24 DLL -> Markus & Laszlo`.

但 EP Section 可见到 Perplex, ACProtect的特征, 有Stolen Code.

OD 异常设置不忽略异常，载入程序,用插件隐藏OD。

```
77FA144C    C3              RETN                                     ; // 系统断点, F9运行

77FA144D >  CC              INT3

77FA144E    C3              RETN

// 出来一个 "No License" 警告, 由于 ACP 有RSA保护, 我们没有Key.dat, 点OK后, INT3 异常

008EBC4B    90              NOP                                      ; // 第1次 INT3 异常, Shift+F9

008EBC4C    64:67:8F06 0000 POP DWORD PTR FS:[0]

008EB9F9    CD 01           INT 1                                    ; // 第2次 内寸访问异常, SEH=008EB9DD

008EB9FB    40              INC EAX                                  ; // 异常处理完毕继续. 这里下断, SHIFT+F9, 取消断点

008EB9FC    40              INC EAX

//再来个内寸镜像断点

 Memory map, item 12

 Address=00401000    Size=000B2000 (729088.)   Owner=XieXieMa  00400000   Section=.text

 Contains=code       Type=Imag 01001002        Access=R        Initial access=RWE

//F9, 断在下面, OEP 就在附近

00495FAD    68 B07B4B00     PUSH XieXieMa.004B7BB0

00495FB2    68 50C04900     PUSH XieXieMa.0049C050

00495FB7    64:A1 00000000  MOV EAX,DWORD PTR FS:[0]

00495FBD    50              PUSH EAX

00495FBE    64:8925 0000000>MOV DWORD PTR FS:[0],ESP

// 495FAD 前面5个字节是 6D 1D 4C A0 E6

// 根据经验猜测 Stolen Byte 如下

// 如果各位大侠有更好的办法, 请指点

00495F9D    A1 B4687F00     MOV EAX,DWORD PTR DS:[7F68B4]

00495FA2    A3 B0687F00     MOV DWORD PTR DS:[7F68B0],EAX

00495FA7    C3              RETN

00495FA8    55              PUSH EBP                                 ; // OEP=495FA8

00495FA9    8BEC            MOV EBP,ESP

00495FAB    6A FF           PUSH -1

00495FAD    68 B07B4B00     PUSH XieXieMa.004B7BB0

00495FB2    68 50C04900     PUSH XieXieMa.0049C050

// 用OD插件Dump 得到 cx1.EXE, 不要重建 IAT, OEP = 95FA8

// 重新运行XXM, 用 ImportREC 跟踪方式3 修复IAT, 得到CX2.EXE

// 运行出错, 用OD加载 CX2.EXE, 用 Trace into, 很容易跟踪到出错的地方

// Trace into 时不要跟进系统函数, 设置一下

00499EBA    E8 DA020000     CALL cx2.0049A199

00499EBF    59              POP ECX

00499EC0    E8 18424400     CALL cx2.008DE0DD                        ; // 这里是第一个出错的地方

00499EC5    8B43 10         MOV EAX,DWORD PTR DS:[EBX+10]

00499EC8    8338 FF         CMP DWORD PTR DS:[EAX],-1

00499ECB    75 07           JNZ SHORT cx2.00499ED4

00499ECD    33C0            XOR EAX,EAX

00499ECF    E9 0F020000     JMP cx2.0049A0E3

00499ED4    891D 6C667F00   MOV DWORD PTR DS:[7F666C],EBX

00499EDA    E8 FE414400     CALL cx2.008DE0DD                        ; // 又一个 CALL cx2.008DE0DD

00499EDF    83FA FF         CMP EDX,-1

00499EE2    8955 FC         MOV DWORD PTR SS:[EBP-4],EDX

00499EE5    74 14           JE SHORT cx2.00499EFB

00499EE7    8B8C90 C4000000 MOV ECX,DWORD PTR DS:[EAX+EDX\*4+C4]

00499EEE    8B7C90 44       MOV EDI,DWORD PTR DS:[EAX+EDX\*4+44]

00499EF2    E8 E6414400     CALL cx2.008DE0DD                        ; // 一共有 1000 多个, 都调用壳里的代码

//重新来过, bp 499EC0, F9断下, F7 进入壳代码, 记住返回地址 499EC5

008DE0DD    60              PUSHAD

008DE0DE    FC              CLD

008DE0DF    F9              STC

008DE0E0    50              PUSH EAX

// 接下来有一大段花指令, SMC, 循环

// 按住 F7 不放, 循环用F4, 到这里

008DE28D    8B4424 20       MOV EAX,DWORD PTR SS:[ESP+20]            ; // 取出返回地址 , 第一次是499EC5

008DE291    33C9            XOR ECX,ECX                              ; // ECX=0

008DE293    8B9C8D B3234000 MOV EBX,DWORD PTR SS:[EBP+ECX\*4+4023B3]  ; // 这里放返回地址 RVA

008DE29A    039D 7BCF4000   ADD EBX,DWORD PTR SS:[EBP+40CF7B]        ; // RVA+400000=VA

008DE2A0    3BC3            CMP EAX,EBX                              ; // 匹配吗?

008DE2A2    74 07           JE SHORT cx2.008DE2AB

008DE2A4    90              NOP

008DE2A5    90              NOP

008DE2A6    90              NOP

008DE2A7    90              NOP

008DE2A8    41              INC ECX                                  ; // 不匹配, 下一个

008DE2A9  ^ EB E8           JMP SHORT cx2.008DE293

008DE2AB    C7848D B3234000>MOV DWORD PTR SS:[EBP+ECX\*4+4023B3],0    ; // 找到了, 清零,  第一次 ECX=15A

008DE2B6    8DB5 93524000   LEA ESI,DWORD PTR SS:[EBP+405293]        ; // 8E1293 放加密后的代码

008DE2BC    B8 0A000000     MOV EAX,0A                               ; // 加密代码 A 个字节一段

008DE2C1    F7E1            MUL ECX

008DE2C3    03F0            ADD ESI,EAX                              ; // 8E1293 + A\*15A = 8E2017

008DE2C5    56              PUSH ESI                                 ; // 第一次调用对应的加密代码

008DE2C6    51              PUSH ECX

008DE2C7    8A85 CD204000   MOV AL,BYTE PTR SS:[EBP+4020CD]          ; // 解密字节保存处

008DE2CD    0AC0            OR AL,AL

008DE2CF    75 28           JNZ SHORT cx2.008DE2F9         ; // 已经计算过了,就跳过

008DE2D5    8B85 7BCF4000   MOV EAX,DWORD PTR SS:[EBP+40CF7B]        ; // 文件头 VA

008DE2DB    8B70 3C         MOV ESI,DWORD PTR DS:[EAX+3C]            ; // PE头 RVA

008DE2DE    03B5 7BCF4000   ADD ESI,DWORD PTR SS:[EBP+40CF7B]        ; // PE头 VA

008DE2E4    83C6 28         ADD ESI,28                               ; // OEP VA

008DE2E7    AD              LODS DWORD PTR DS:[ESI]                  ; // OEP

008DE2E8    8AD8            MOV BL,AL

008DE2EA    02DC            ADD BL,AH

008DE2EC    C1E8 10         SHR EAX,10

008DE2EF    02D8            ADD BL,AL

008DE2F1    02DC            ADD BL,AH                                ; // OEP 四字节和 = 10

008DE2F3    889D CD204000   MOV BYTE PTR SS:[EBP+4020CD],BL          ; // 解密用字节

.

.

.

008DE30E    61              POPAD

008DE30F    8BBD 7FCF4000   MOV EDI,DWORD PTR SS:[EBP+40CF7F]        ; // 代码解密后放到 134708 开始的内存块

008DE315    B8 0A000000     MOV EAX,0A

008DE31A    F7E1            MUL ECX

008DE31C    03F8            ADD EDI,EAX                              ; // 134708 + A\*15A = 13548C

008DE31E    B9 0A000000     MOV ECX,0A

008DE323    8A9D CD204000   MOV BL,BYTE PTR SS:[EBP+4020CD]

008DE329    EB 11           JMP SHORT cx2.008DE33C

008DE32E    61              POPAD

008DE32F    8BFE            MOV EDI,ESI

008DE331    B9 0A000000     MOV ECX,0A

008DE336    8A9D CD204000   MOV BL,BYTE PTR SS:[EBP+4020CD]          ; // 取出解密用字节 10

008DE33C    AC              LODS BYTE PTR DS:[ESI]                   ; // 密文

008DE33D    32C3            XOR AL,BL                                ; // 密文 Xor 10

008DE33F    AA              STOS BYTE PTR ES:[EDI]                   ; // 明文

008DE340  ^ E2 FA           LOOPD SHORT cx2.008DE33C                 ; // 解密 A 个字节

008DE342    83EF 0A         SUB EDI,0A

008DE345    57              PUSH EDI                                 ; // 新的函数入口 13548C

008DE346    8B7424 24       MOV ESI,DWORD PTR SS:[ESP+24]            ; // 返回地址 499EC5

008DE34A    83EE 04         SUB ESI,4

008DE34D    AD              LODS DWORD PTR DS:[ESI]

008DE34E    81EF DD204000   SUB EDI,cx2.004020DD

008DE354    2BFD            SUB EDI,EBP

008DE356    03C7            ADD EAX,EDI                              ; // 地址转换成偏移量

008DE358    8946 FC         MOV DWORD PTR DS:[ESI-4],EAX             ; // 变成 CALL 13548C.

.

// 调整Stack

// SMC

.

.

008DE3B0    61              POPAD

008DE3B1    61              POPAD

008DE3B2    C3              RETN                                     ; // 返回到 13548C

// 从上述过程可以看到, 如果OEP变了, 那么加密代码解密出来就不对

// 所以我们恢复 OEP, 用LordPE 查脱壳前OEP= 4DD000, VA=8DD000

// 用 OD 修改 CX2.EXE 8DD000 处为 JMP 495FA8

// 用 LordPE 改 OEP= 4DD000, 得到CX3.EXE

// 用 OD 载入CX3.EXE还是出错, Trace into again, 错在 499EDA
```

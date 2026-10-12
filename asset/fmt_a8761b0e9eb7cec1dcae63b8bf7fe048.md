
# 申请加入 OCN 之二--FilesafeV7.0 企业版算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-796.htm`](https://www.pediy.com/kssd/pediy07/pediy7-796.htm)

【破文标题】申请加入 OCN 之二--FilesafeV7.0 企业版算法分析

【破文作者】pentacle\[PYG\]

【作者邮箱】

【作者主页】`www.chinapyg.cn`

【破解工具】`OD` 汉化第二版、`Peid`、`calc`

【破解平台】`XP_SP2`

【软件名称】`FilesafeV7.0` 企业版

【软件大小】`2.2M`

【原版下载】网上无企业版下载。所以在这儿发一个分析也应该没关系啦。

【保护方式】注册码

【破解过程】

1、查壳

`UPX 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo`

轻松搞定

2、注册算法分析

用 `OD` 载入脱完壳的软件。

`F9` 运行，弹出试用提示窗口(限 50 次及运行 60 分钟)

不管了。分析注册码是我的任务。因为已经分析过了。所以我直入主题。

假注册码：`ABCAEFGHD1234Q678AV1`

点确定，断在 `00405050`

```
00405050   .  6A FF          PUSH -1                                   ; HOHO。我在这儿下断了。开始我们的算法分析之旅吧
00405052   .  68 E0EC4700    PUSH 1FileSaf.0047ECE0                    ; 入口地址; SE handler installation
00405057   .  64:A1 0000000> MOV EAX,DWORD PTR FS:[0]
0040505D   .  50             PUSH EAX
0040505E   .  64:8925 00000> MOV DWORD PTR FS:[0],ESP
00405065   .  83EC 10        SUB ESP,10
00405068   .  55             PUSH EBP
00405069   .  8BE9          MOV EBP,ECX
0040506B   .  E8 804C0100    CALL 1FileSaf.00419CF0                     ; 取机器码(硬盘芯片 ID)
00405070   .  50             PUSH EAX
00405071   .  8D4C24 0C      LEA ECX,DWORD PTR SS:[ESP+C]
00405075   .  E8 BEEA60500   CALL 1FileSaf.0045F738
0040507A   .  A1 14834A00    MOV EAX,DWORD PTR DS:[4A8314]
0040507F   .  C74424 1C 000> MOV DWORD PTR SS:[ESP+1C],0
00405087   .  894424 04      MOV DWORD PTR SS:[ESP+4],EAX
0040508B   .  8D4C24 04      LEA ECX,DWORD PTR SS:[ESP+4]
0040508F   .  C64424 1C 01   MOV BYTE PTR SS:[ESP+1C],1
00405094   .  51             PUSH ECX
00405095   .  68 4A040000    PUSH 44A
0040509A   .  8BCD          MOV ECX,EBP
0040509C   .  E8 AF9A0500    CALL 1FileSaf.0045EB50
004050A1   .  8BC8          MOV ECX,EAX
004050A3   .  E8 2C760500    CALL 1FileSaf.0045C6D4                     ; 读取假注册码
004050A8   .  8B5424 04      MOV EDX,DWORD PTR SS:[ESP+4]
004050AC   .  8B4424 08      MOV EAX,DWORD PTR SS:[ESP+8]
004050B0   .  52             PUSH EDX
004050B1   .  50             PUSH EAX
004050B2   .  E8 99390100    CALL 1FileSaf.00418A50                     ; 算法 CALL，跟进去
004050B7   .  83C4 08        ADD ESP,8
004050BA   .  84C0          TEST AL,AL                                   ; 测试标志位
004050BC   .  0F84 C7000000  JE 1FileSaf.00405189                       ; 不跳就是注册成功
```

跟入 `CALL 00418A50`

```
00418A50  /$  64:A1 0000000> MOV EAX,DWORD PTR FS:[0]                   ; 这个 CALL 完成了注册码整个校验过程
00418A56  |.  6A FF           PUSH -1
00418A58  |.  68 880E4800     PUSH 1FileSaf.00480E88
00418A5D  |.  50             PUSH EAX
00418A5E  |.  64:8925 00000> MOV DWORD PTR FS:[0],ESP
00418A65  |.  83EC 10        SUB ESP,10
00418A68  |.  56             PUSH ESI
00418A69  |.  8B7424 24      MOV ESI,DWORD PTR SS:[ESP+24]
00418A6D  |.  803E 00        CMP BYTE PTR DS:[ESI],0
00418A70  |.  0F84 21010000  JE 1FileSaf.00418B97
00418A76  |.  8B4C24 28      MOV ECX,DWORD PTR SS:[ESP+28]
00418A7A  |.  8039 00        CMP BYTE PTR DS:[ECX],0
00418A7D  |.  0F84 14010000  JE 1FileSaf.00418B97
00418A83  |.  A1 14834A00    MOV EAX,DWORD PTR DS:[4A8314]
00418A88  |.  53             PUSH EBX
00418A89  |.  894424 08      MOV DWORD PTR SS:[ESP+8],EAX
00418A8D  |.  C74424 20 000> MOV DWORD PTR SS:[ESP+20],0
00418A95  |.  894424 2C      MOV DWORD PTR SS:[ESP+2C],EAX
00418A99  |.  8D4424 14      LEA EAX,DWORD PTR SS:[ESP+14]
00418A9D  |.  8D5424 2C      LEA EDX,DWORD PTR SS:[ESP+2C]
00418AA1  |.  50             PUSH EAX
00418AA2  |.  8D4424 0C      LEA EAX,DWORD PTR SS:[ESP+C]
00418AA6  |.  52             PUSH EDX
00418AA7  |.  50             PUSH EAX
00418AA8  |.  51             PUSH ECX
00418AA9  |.  C64424 30 01   MOV BYTE PTR SS:[ESP+30],1
00418AAE  |.  E8 3D050000    CALL 1FileSaf.00418FF0                    ; 关键 CALL。分析对象
00418AB3  |.  83C4 10        ADD ESP,10
00418AB6  |.  8D4C24 28      LEA ECX,DWORD PTR SS:[ESP+28]
00418ABA  |.  56             PUSH ESI
00418ABB  |.  E8 786C0400    CALL 1FileSaf.0045F738
00418AC0  |.  8D4C24 08      LEA ECX,DWORD PTR SS:[ESP+8]
00418AC4  |.  C64424 20 02   MOV BYTE PTR SS:[ESP+20],2
00418AC9  |.  E8 1E710400    CALL 1FileSaf.0045FBEC
00418ACE  |.  8D4C24 2C      LEA ECX,DWORD PTR SS:[ESP+2C]
00418AD2  |.  E8 15710400    CALL 1FileSaf.0045FBEC
00418AD7  |.  8D4C24 28      LEA ECX,DWORD PTR SS:[ESP+28]
00418ADB  |.  E8 0C710400    CALL 1FileSaf.0045FBEC
00418AE0  |.  8D4C24 10      LEA ECX,DWORD PTR SS:[ESP+10]
00418AE4  |.  6A 05          PUSH 5
00418AE6  |.  51             PUSH ECX
00418AE7  |.  8D4C24 30      LEA ECX,DWORD PTR SS:[ESP+30]
00418AEB  |.  E8 A5F00300    CALL 1FileSaf.00457B95                     ; 取机器码前 5 位
00418AF0  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]
00418AF2  |.  8B5424 08      MOV EDX,DWORD PTR SS:[ESP+8]
00418AF6  |.  50             PUSH EAX
00418AF7  |.  52             PUSH EDX
00418AF8  |.  C64424 28 03   MOV BYTE PTR SS:[ESP+28],3
00418AFD  |.  E8 8E030000    CALL 1FileSaf.00418E90                    ; 对比机器码前 5 位
00418B02  |.  83C4 08        ADD ESP,8
00418B05  |.  84C0          TEST AL,AL
00418B07      74 3C          JE SHORT 1FileSaf.00418B45                ; 此处不能跳~~
00418B09      8D4424 0C      LEA EAX,DWORD PTR SS:[ESP+C]
00418B0D  |.  6A 05          PUSH 5
00418B0F  |.  50             PUSH EAX
00418B10  |.  8D4C24 30      LEA ECX,DWORD PTR SS:[ESP+30]
00418B14  |.  E8 00F00300    CALL 1FileSaf.00457B19                    ; 取机器码后 5 位
00418B19  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]
00418B1B      8B4C24 2C      MOV ECX,DWORD PTR SS:[ESP+2C]
00418B1F  |.  50             PUSH EAX
00418B20  |.  51             PUSH ECX
00418B21  |.  C64424 28 04   MOV BYTE PTR SS:[ESP+28],4
00418B26  |.  E8 65030000    CALL 1FileSaf.00418E90                    ; 对比机器码后 5 位
00418B2B  |.  83C4 08        ADD ESP,8
00418B2E  |.  8D4C24 0C      LEA ECX,DWORD PTR SS:[ESP+C]
00418B32  |.  84C0          TEST AL,AL                                   ; AL 值要为 1
00418B34  |.  0F94C3        SETE BL
00418B37  |.  C64424 20 03   MOV BYTE PTR SS:[ESP+20],3
00418B3C  |.  E8 896B0400    CALL 1FileSaf.0045F6CA
00418B41  |.  84DB          TEST BL,BL                                   ; BL=0 下面才能跳
00418B43  |.  74 02          JE SHORT 1FileSaf.00418B47                ; 此处必须跳~~~
00418B45  |>  B3 01          MOV BL,1
00418B47  |>  8D4C24 10      LEA ECX,DWORD PTR SS:[ESP+10]
00418B4B  |.  C64424 20 02   MOV BYTE PTR SS:[ESP+20],2
00418B50  |.  E8 756B0400    CALL 1FileSaf.0045F6CA
00418B55  |.  84DB          TEST BL,BL                                   ; 测试 BL 的值
00418B57  |.  5B             POP EBX
00418B58  |.  75 10          JNZ SHORT 1FileSaf.00418B6A               ; 此处不能跳~~~
00418B5A  |.  8B4424 10      MOV EAX,DWORD PTR SS:[ESP+10]             ; 将 SS:[ESP+10]处的值赋给 EAX，即 E00
00418B5E  |.  F6C4 E0        TEST AH,0E0                                ; 关键 CALL3 计算在此用上了~~AH 必须=0E0
00418B61      75 07          JNZ SHORT 1FileSaf.00418B6A               ; 不能跳~~~
00418B63  |.  A9 E0000000    TEST EAX,0E0                               ; JBE 低于或等于跳转，也就是 EAX 必须等于 00000E00
00418B68  |.  76 3F          JBE SHORT 1FileSaf.00418BA9               ; 必须跳,跳向成功了
00418B6A  |>  8D4C24 24      LEA ECX,DWORD PTR SS:[ESP+24]
00418B6E  |.  C64424 1C 01   MOV BYTE PTR SS:[ESP+1C],1
00418B73  |.  E8 526B0400    CALL 1FileSaf.0045F6CA
00418B78  |.  8D4C24 28      LEA ECX,DWORD PTR SS:[ESP+28]
00418B7C  |.  C64424 1C 00   MOV BYTE PTR SS:[ESP+1C],0
00418B81  |.  E8 446B0400    CALL 1FileSaf.0045F6CA
00418B86  |.  8D4C24 04      LEA ECX,DWORD PTR SS:[ESP+4]
00418B8A  |.  C74424 1C FFF> MOV DWORD PTR SS:[ESP+1C],-1
00418B92  |.  E8 336B0400    CALL 1FileSaf.0045F6CA
```

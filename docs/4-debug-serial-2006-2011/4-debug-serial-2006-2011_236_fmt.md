
# 易通文件夹锁 2006 3.0.2.928

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-341.htm`](https://www.pediy.com/kssd/pediy08/pediy8-341.htm)

## 文章标题
易通文件夹锁 2006 3.0.2.928 算法分析

## 文章作者
网游难民

## 作者主页
`www.chinapyg.com`

## 软件名称
易通文件夹锁 2006 3.0.2.928

## 软件大小
3099KB

## 下载地址
`http://www.newhua.com/soft/43125.htm`

## 加壳方式
`aspack`

## 保护方式
壳，注册码

## 编写语言
Borland Delphi 6.0 - 7.0

## 使用工具
`OD` `PEID`

## 操作平台
XP SP2

## 软件介绍
四大保护功能

## 作者声明
偶在学习飘云阁 `tigerisme` 斑竹的文章时看到 `tigerisme` 兄弟只写了机器码是怎么的来的，偶就把算法补充上啊~~

--------------------------------------------------------------------------------

## 详细过程

不要问偶怎么脱壳的，偶是壳盲，交给 `OD` 自己搞定，然后修复下就 `OK`^\_\_^

下下 `bp MessageBoxA` 断点，找合适的地方下断，下面开始分析拉~~

++++++++++++++++++++++++++++++++++++++++++++++++++++++++

```
0057C2A0    8BD8            MOV EBX,EAX
0057C2A2    33C0            XOR EAX,EAX
0057C2A4    55              PUSH EBP
0057C2A5    68 06C55700     PUSH etdirloc.0057C506
0057C2AA    64:FF30         PUSH DWORD PTR FS:[EAX]
0057C2AD    64:8920         MOV DWORD PTR FS:[EAX],ESP
0057C2B0    8D55 F4         LEA EDX,DWORD PTR SS:[EBP-C]
0057C2B3    8B83 20030000   MOV EAX,DWORD PTR DS:[EBX+320]
0057C2B9    E8 6A6BFDFF     CALL etdirloc.00552E28
0057C2BE    8B45 F4         MOV EAX,DWORD PTR SS:[EBP-C]    ; 注册名放入 EAX 中~~
0057C2C1    8D55 F8         LEA EDX,DWORD PTR SS:[EBP-8]
0057C2C4    E8 53CCF8FF     CALL etdirloc.00508F1C
0057C2C9    837D F8 00      CMP DWORD PTR SS:[EBP-8],0
0057C2CD    0F84 80000000   JE etdirloc.0057C353
0057C2D3    8D55 EC         LEA EDX,DWORD PTR SS:[EBP-14]
0057C2D6    8B83 24030000   MOV EAX,DWORD PTR DS:[EBX+324]
0057C2DC    E8 476BFDFF     CALL etdirloc.00552E28
0057C2E1    8B45 EC         MOV EAX,DWORD PTR SS:[EBP-14]   ; 第一列注册码
0057C2E4    8D55 F0         LEA EDX,DWORD PTR SS:[EBP-10]
0057C2E7    E8 30CCF8FF     CALL etdirloc.00508F1C
0057C2EC    837D F0 00      CMP DWORD PTR SS:[EBP-10],0
0057C2F0    0F85 9A000000   JNZ etdirloc.0057C390
0057C2F6    8D55 E4         LEA EDX,DWORD PTR SS:[EBP-1C]
0057C2F9    8B83 2C030000   MOV EAX,DWORD PTR DS:[EBX+32C]
0057C2FF    E8 246BFDFF     CALL etdirloc.00552E28
0057C304    8B45 E4         MOV EAX,DWORD PTR SS:[EBP-1C]
0057C307    8D55 E8         LEA EDX,DWORD PTR SS:[EBP-18]
0057C30A    E8 0DCCF8FF     CALL etdirloc.00508F1C
0057C30F    837D E8 00      CMP DWORD PTR SS:[EBP-18],0
0057C313    75 7B           JNZ SHORT etdirloc.0057C390
0057C315    8D55 DC         LEA EDX,DWORD PTR SS:[EBP-24]
0057C318    8B83 30030000   MOV EAX,DWORD PTR DS:[EBX+330]
0057C31E    E8 056BFDFF     CALL etdirloc.00552E28
0057C323    8B45 DC         MOV EAX,DWORD PTR SS:[EBP-24]
0057C326    8D55 E0         LEA EDX,DWORD PTR SS:[EBP-20]
0057C329    E8 EECBF8FF     CALL etdirloc.00508F1C
0057C32E    837D E0 00      CMP DWORD PTR SS:[EBP-20],0
0057C332    75 5C           JNZ SHORT etdirloc.0057C390
0057C334    8D55 D4         LEA EDX,DWORD PTR SS:[EBP-2C]
0057C337    8B83 34030000   MOV EAX,DWORD PTR DS:[EBX+334]
0057C33D    E8 E66AFDFF     CALL etdirloc.00552E28
0057C342    8B45 D4         MOV EAX,DWORD PTR SS:[EBP-2C]
0057C345    8D55 D8         LEA EDX,DWORD PTR SS:[EBP-28]
0057C348    E8 CFCBF8FF     CALL etdirloc.00508F1C
0057C34D    837D D8 00      CMP DWORD PTR SS:[EBP-28],0
0057C351    75 3D           JNZ SHORT etdirloc.0057C390
0057C353    68 30000400     PUSH 40030
0057C358    8D55 D0         LEA EDX,DWORD PTR SS:[EBP-30]
0057C35B    B8 20C55700     MOV EAX,etdirloc.0057C520
0057C360    E8 E793FFFF     CALL etdirloc.0057574C
0057C365    8B45 D0         MOV EAX,DWORD PTR SS:[EBP-30]
0057C368    E8 B78AF8FF     CALL etdirloc.00504E24
0057C36D    50              PUSH EAX
0057C36E    8D55 CC         LEA EDX,DWORD PTR SS:[EBP-34]
0057C371    B8 38C55700     MOV EAX,etdirloc.0057C538
0057C376    E8 D193FFFF     CALL etdirloc.0057574C
0057C37B    8B45 CC         MOV EAX,DWORD PTR SS:[EBP-34]
0057C37E    E8 A18AF8FF     CALL etdirloc.00504E24
0057C383    50              PUSH EAX
0057C384    6A 00           PUSH 0
0057C386    E8 B9B5F8FF     CALL etdirloc.00507944          ; JMP 到 user32.MessageBoxA
0057C38B    E9 EC000000     JMP etdirloc.0057C47C
0057C390    8D55 C8         LEA EDX,DWORD PTR SS:[EBP-38]
0057C393    8B83 24030000   MOV EAX,DWORD PTR DS:[EBX+324]
0057C399    E8 8A6AFDFF     CALL etdirloc.00552E28
0057C39E    FF75 C8         PUSH DWORD PTR SS:[EBP-38]
0057C3A1    8D55 C4         LEA EDX,DWORD PTR SS:[EBP-3C]
0057C3A4    8B83 2C030000   MOV EAX,DWORD PTR DS:[EBX+32C]
0057C3AA    E8 796AFDFF     CALL etdirloc.00552E28
0057C3AF    FF75 C4         PUSH DWORD PTR SS:[EBP-3C]      ; 第二列
0057C3B2    8D55 C0         LEA EDX,DWORD PTR SS:[EBP-40]
0057C3B5    8B83 30030000   MOV EAX,DWORD PTR DS:[EBX+330]
0057C3BB    E8 686AFDFF     CALL etdirloc.00552E28
0057C3C0    FF75 C0         PUSH DWORD PTR SS:[EBP-40]      ; 第三列
0057C3C3    8D55 BC         LEA EDX,DWORD PTR SS:[EBP-44]
0057C3C6    8B83 34030000   MOV EAX,DWORD PTR DS:[EBX+334]
0057C3CC    E8 576AFDFF     CALL etdirloc.00552E28
0057C3D1    FF75 BC         PUSH DWORD PTR SS:[EBP-44]      ; 第四列
0057C3D4    8D45 FC         LEA EAX,DWORD PTR SS:[EBP-4]
0057C3D7    BA 04000000     MOV EDX,4
0057C3DC    E8 0B89F8FF     CALL etdirloc.00504CEC
0057C3E1    8D55 B8         LEA EDX,DWORD PTR SS:[EBP-48]
0057C3E4    8B83 20030000   MOV EAX,DWORD PTR DS:[EBX+320]
0057C3EA    E8 396AFDFF     CALL etdirloc.00552E28
0057C3EF    8B55 B8         MOV EDX,DWORD PTR SS:[EBP-48]   ; 用户名放入 EDX
0057C3F2    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]    ; 注册码放入 EAX
0057C3F5    E8 5A97FFFF     CALL etdirloc.00575B54          ; 关键 CALL，跟进~
0057C3FA    84C0            TEST AL,AL
0057C3FC    74 46           JE SHORT etdirloc.0057C444      ; 爆破点~
0057C3FE    68 40000400     PUSH 40040
```

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

跟进上面的算法 `CALL`：

```
00575B66    8BF0            MOV ESI,EAX
00575B68    33C0            XOR EAX,EAX
00575B6A    55              PUSH EBP
00575B6B    68 AA5C5700     PUSH etdirloc.00575CAA
00575B70    64:FF30         PUSH DWORD PTR FS:[EAX]
00575B73    64:8920         MOV DWORD PTR FS:[EAX],ESP
00575B76    C645 FF 00      MOV BYTE PTR SS:[EBP-1],0
00575B7A    B8 E0515700     MOV EAX,etdirloc.005751E0
00575B7F    E8 64F8FFFF     CALL etdirloc.005753E8
00575B84    84C0            TEST AL,AL
00575B86    74 0C           JE SHORT etdirloc.00575B94
00575B88    A1 2CD15900     MOV EAX,DWORD PTR DS:[59D12C]
00575B8D    8B00            MOV EAX,DWORD PTR DS:[EAX]
00575B8F    E8 5CF1FCFF     CALL etdirloc.00544CF0
00575B94    B8 944F5700     MOV EAX,etdirloc.00574F94
00575B99    E8 4AF8FFFF     CALL etdirloc.005753E8
00575B9E    84C0            TEST AL,AL
00575BA0    74 0C           JE SHORT etdirloc.00575BAE
00575BA2    A1 2CD15900     MOV EAX,DWORD PTR DS:[59D12C]
00575BA7    8B00            MOV EAX,DWORD PTR DS:[EAX]
00575BA9    E8 42F1FCFF     CALL etdirloc.00544CF0
00575BAE    B8 A8505700     MOV EAX,etdirloc.005750A8
00575BB3    E8 30F8FFFF     CALL etdirloc.005753E8
00575BB8    84C0            TEST AL,AL
00575BBA    74 0C           JE SHORT etdirloc.00575BC8
00575BBC    A1 2CD15900     MOV EAX,DWORD PTR DS:[59D12C]
00575BC1    8B00            MOV EAX,DWORD PTR DS:[EAX]
00575BC3    E8 28F1FCFF     CALL etdirloc.00544CF0
```



```
00575BC8    8D45 F4           LEA EAX,DWORD PTR SS:[EBP-C]  

00575BCB    E8 BCFAFFFF      CALL etdirloc.0057568C          ; 算法 CALL（1）  

00575BD0    8D4D F0          LEA ECX,DWORD PTR SS:[EBP-10]  

00575BD3    8BD7             MOV EDX,EDI  

00575BD5    A1 68EC5900      MOV EAX,DWORD PTR DS:[59EC68]  

00575BDA    E8 BDFCFFFF      CALL etdirloc.0057589C          ; 算法 CALL（2），跟进  

00575BDF    8B45 F0          MOV EAX,DWORD PTR SS:[EBP-10]  

00575BE2    8BD6             MOV EDX,ESI  

00575BE4    E8 87F1F8FF      CALL etdirloc.00504D70  

00575BE9    0F85 A0000000    JNZ etdirloc.00575C8F  

00575BEF    B2 01            MOV DL,1  

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++  

上面的算法 CALL（1）是介绍怎么根据硬盘号取得机器码的，机器码在提示我们注册的对话框里会出现，我们没有必要看他是怎么来的，如果有兴趣，可以看下 tigerisme 斑竹的文章，上面有详细介绍，我就不多说了~  

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++  

跟进算法 CALL（2）：  

005758BE    64:FF30         PUSH DWORD PTR FS:[EAX]  

005758C1    64:8920         MOV DWORD PTR FS:[EAX],ESP  

005758C4    8B45 F4         MOV EAX,DWORD PTR SS:[EBP-C]  

005758C7    E8 A8F0F8FF     CALL etdirloc.00504974  

005758CC    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]  

005758CF    E8 58F3F8FF     CALL etdirloc.00504C2C  

005758D4    85C0            TEST EAX,EAX  

005758D6    0F84 B2000000   JE etdirloc.0057598E  

005758DC    8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]  

005758DF    E8 48F3F8FF     CALL etdirloc.00504C2C  

005758E4    85C0            TEST EAX,EAX  

005758E6    0F84 A2000000   JE etdirloc.0057598E  

005758EC    8D45 E4         LEA EAX,DWORD PTR SS:[EBP-1C]  

005758EF    E8 80F0F8FF     CALL etdirloc.00504974  

005758F4    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]  

005758F7    E8 30F3F8FF     CALL etdirloc.00504C2C  

005758FC    8945 EC         MOV DWORD PTR SS:[EBP-14],EAX  

005758FF    8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]  

00575902    E8 25F3F8FF     CALL etdirloc.00504C2C  

00575907    8945 E8         MOV DWORD PTR SS:[EBP-18],EAX  

0057590A    BF 01000000     MOV EDI,1                     ; EDX 置 1  

0057590F    BB 01000000     MOV EBX,1                     ; EBX 置 1  

00575914    C745 F0 0100000>MOV DWORD PTR SS:[EBP-10],1       ; SS:[EBP-10]置 1  

0057591B    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]      ; 求出注册码，循环开始  

0057591E    0FB64438 FF     MOVZX EAX,BYTE PTR DS:[EAX+EDI-1] ; 机器码 ASCII 码逐位放入 EAX  

00575923    2B45 F0         SUB EAX,DWORD PTR SS:[EBP-10]     ; 减去 SS:[EBP-10] 中的值  

00575926    03C7            ADD EAX,EDI                     ; 加上 EDI  

00575928    8B55 F8         MOV EDX,DWORD PTR SS:[EBP-8]  

0057592B    0FB6541A FF     MOVZX EDX,BYTE PTR DS:[EDX+EBX-1] ; 用户名 ASCII 码逐位放入 EDX  

00575930    33C2            XOR EAX,EDX                     ; 上面两个数异或运算  

00575932    03C3            ADD EAX,EBX                     ; 上面得到的值加上 EBX  

00575934    99              CDQ                             ; 扩展指令  

00575935    33C2            XOR EAX,EDX  

00575937    2BC2            SUB EAX,EDX  

00575939    8BF0            MOV ESI,EAX  

0057593B    8BC6            MOV EAX,ESI  

0057593D    B9 24000000     MOV ECX,24                      ; ECX 的值为 24  

00575942    99              CDQ                             ; 扩展指令  

00575943    F7F9            IDIV ECX                        ; EAX 中的值除以 24，余数放在 EDX 中  

00575945    8BF2            MOV ESI,EDX                     ; EDX 中的值放在 ESI 中  

00575947    8D45 E0         LEA EAX,DWORD PTR SS:[EBP-20]  

0057594A    8A96 90C75900   MOV DL,BYTE PTR DS:[ESI+59C790] ; 找到内存对应的表中相应的值，把这些值连接起来就是真码  

00575950    E8 FFF1F8FF     CALL etdirloc.00504B54  

00575955    8B55 E0         MOV EDX,DWORD PTR SS:[EBP-20]  

00575958    8D45 E4         LEA EAX,DWORD PTR SS:[EBP-1C]  

0057595B    E8 D4F2F8FF     CALL etdirloc.00504C34  

00575960    3B7D EC         CMP EDI,DWORD PTR SS:[EBP-14]   ; EDI 中的值和机器码位数比较  

00575963    7D 03           JGE SHORT etdirloc.00575968  

00575965    47              INC EDI  

00575966    EB 05           JMP SHORT etdirloc.0057596D  

00575968    BB 01000000     MOV EBX,1  

0057596D    3B5D E8         CMP EBX,DWORD PTR SS:[EBP-18]   ; EBX 中的值和用户名位数比  

00575970    7D 03           JGE SHORT etdirloc.00575975  

00575972    43              INC EBX  

00575973    EB 05           JMP SHORT etdirloc.0057597A  

00575975    BB 01000000     MOV EBX,1  

0057597A    FF45 F0         INC DWORD PTR SS:[EBP-10]       ; SS:[EBP-10] 中的值加 1  

0057597D    837D F0 15      CMP DWORD PTR SS:[EBP-10],15    ; SS:[EBP-10] 的值和 15h 比较，控制循环次数  

00575981    75 98           JNZ SHORT etdirloc.0057591B     ; 循环结束  

00575983    8B45 F4         MOV EAX,DWORD PTR SS:[EBP-C]  

00575986    8B55 E4         MOV EDX,DWORD PTR SS:[EBP-1C]  

--------------------------------------------------------------------------------

### 【经验总结】

下面总结下上面的关键循环：

分两种情况：

#### 一，当用户名位数小于机器码位数时：

*   当循环次数不大于用户名位数时，是机器码 ASCII 码与用户名 ASCII 码异或运算后的值加上循环次数，然后除以 24 得到的余数在内存表中对应的值。
*   当循环次数大于用户名位数小于机器码位数时，是机器码 ASCII 码与用户名 ASCII 码异或运算后的值从 1 开始加起，得到的值除以 24 得到的余数在内存表中对应的值。
*   当循环次数大于机器码位数时，是机器码最后一位 ASCII 码减去循环次数，加上机器码位数后的值，与用户名第二位逻辑异或运算，得到的值加上 2，然后除以 24 得到的余数在内存表中对应的值。

把上面得到的值连接起来就是真码。

#### 二，当用户名位数大于机器码位数时：

*   当循环次数不大于机器码位数时，是机器码 ASCII 码与用户名 ASCII 码异或运算后的值加上循环次数，然后除以 24 得到的余数在内存表中对应的值。
*   当循环次数大于机器码位数时，是机器码最后一位 ASCII 码减去循环次数，加上机器码位数后的值，与用户名第二位逻辑异或运算，得到的值加上 2，然后除以 24 得到的余数在内存表中对应的值。

把上面得到的值连接起来就是真码。

那张内存表见附件。

偶的语言表达能力有限，看不太懂的朋友请自己跟下啊~

--------------------------------------------------------------------------------

### 【版权声明】

菜鸟初学算法，失误之处敬请诸位大侠赐教，转载请注明作者并保持文章的完整，谢谢！

`4dd466d0e1e0cc123afbe504da2462ca`


# BlackRose 反黑工具箱 2.6 Build 0331

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-343.htm`](https://www.pediy.com/kssd/pediy08/pediy8-343.htm)

## 基本信息
【破文标题】BlackRose 反黑工具箱 2.6 Build 0331 算法分析（PYG、D.4S、OCN、DCG）

【破文作者】学习破解 也就是 wxh9833

【作者邮箱】

【作者主页】

【破解工具】`PEiD`, `OD`

【破解平台】`Windows` 2K&XP

【软件名称】BlackRose 反黑工具箱 2.6 Build 0331

【软件大小】2904KB

【更新时间】2006-10-8

【原版下载】`http://www.onlinedown.net/soft/32588.htm`

【保护方式】壳、注册码

【软件简介】BlackRose 反黑工具箱【综合版】"是一款电脑反黑综合工具

其他功能到下载地址看一下。

【破解声明】我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---

## 分析过程
1. `PEiD` 查壳。`UPX-Scrambler` RC1.x -> ㎡nT 畂 L（UPX 壳）ESP 没脱成。手脱，方法就是单步，有跨区段就跳。

2. 脱完后运行程序，看有没有提示信息。无提示。找别的方法吧。还是重新启动验证。

3. `OD` 载入，其实很容易找到断点。

```
00499A28    55               PUSH EBP                             ;本应断在这里，但有小循环，麻烦。
00499A29    8BEC             MOV EBP,ESP
00499A2B    B9 06000000      MOV ECX,6
00499A30    6A 00            PUSH 0
00499A32    6A 00            PUSH 0
00499A34    49               DEC ECX
00499A35   ^ 75 F9          JNZ SHORT ehelper1.00499A30          ; 以上是一个小循环，比较麻烦。呵呵。所以在这里下断。
00499A37    51               PUSH ECX                             ; 断点断在这里。
00499A38    53               PUSH EBX
00499A39    8945 FC          MOV DWORD PTR SS:[EBP-4],EAX
00499A3C    33C0             XOR EAX,EAX
00499A3E    55               PUSH EBP
00499A3F    68 6C9C4900      PUSH ehelper1.00499C6C
00499A44    64:FF30          PUSH DWORD PTR FS:[EAX]
00499A47    64:8920          MOV DWORD PTR FS:[EAX],ESP
00499A4A    8D55 F0          LEA EDX,DWORD PTR SS:[EBP-10]
00499A4D    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]
00499A50    8B80 64040000    MOV EAX,DWORD PTR DS:[EAX+464]
00499A56    E8 D989FBFF      CALL ehelper1.00452434
00499A5B    8B45 F0          MOV EAX,DWORD PTR SS:[EBP-10]
00499A5E    8D55 F4          LEA EDX,DWORD PTR SS:[EBP-C]        ; 用户名
00499A61    E8 DEE7F6FF      CALL ehelper1.00408244
00499A66    8B45 F4          MOV EAX,DWORD PTR SS:[EBP-C]        ; 用户名
00499A69    8B15 24C44900    MOV EDX,DWORD PTR DS:[49C424]       ; ehelper1.0049B0BC
00499A6F    8B12             MOV EDX,DWORD PTR DS:[EDX]
00499A71    E8 A2A9F6FF      CALL ehelper1.00404418              ; 这个 CALL 是取用户名的位数。
00499A76    74 78            JE SHORT ehelper1.00499AF0
00499A78    8D55 E8          LEA EDX,DWORD PTR SS:[EBP-18]
00499A7B    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]
00499A7E    8B80 68040000    MOV EAX,DWORD PTR DS:[EAX+468]
00499A84    E8 AB89FBFF      CALL ehelper1.00452434
00499A89    8B45 E8          MOV EAX,DWORD PTR SS:[EBP-18]       ; 试练码
00499A8C    8D55 EC          LEA EDX,DWORD PTR SS:[EBP-14]
00499A8F    E8 B0E7F6FF      CALL ehelper1.00408244
00499A94    8B45 EC          MOV EAX,DWORD PTR SS:[EBP-14]       ; 试练码
00499A97    8B15 24C44900    MOV EDX,DWORD PTR DS:[49C424]       ; ehelper1.0049B0BC
00499A9D    8B12             MOV EDX,DWORD PTR DS:[EDX]
00499A9F    E8 74A9F6FF      CALL ehelper1.00404418              ; CALL 出试练码的位数。
00499AA4    74 4A            JE SHORT ehelper1.00499AF0
00499AA6    8D55 E4          LEA EDX,DWORD PTR SS:[EBP-1C]
00499AA9    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]
00499AAC    8B80 64040000    MOV EAX,DWORD PTR DS:[EAX+464]
00499AB2    E8 7D89FBFF      CALL ehelper1.00452434
00499AB7    8B45 E4          MOV EAX,DWORD PTR SS:[EBP-1C]       ; 取出用户名
00499ABA    E8 0DA8F6FF      CALL ehelper1.004042CC              ; 位数与 5 比较
00499ABF    83F8 05          CMP EAX,5
00499AC2    7C 53            JL SHORT ehelper1.00499B17         ; 小于就死
00499AC4    8D55 DC          LEA EDX,DWORD PTR SS:[EBP-24]
00499AC7    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]
00499ACA    8B80 64040000    MOV EAX,DWORD PTR DS:[EAX+464]
00499AD0    E8 5F89FBFF      CALL ehelper1.00452434
00499AD5    8B45 DC          MOV EAX,DWORD PTR SS:[EBP-24]
00499AD8    8D55 E0          LEA EDX,DWORD PTR SS:[EBP-20]
00499ADB    E8 64E7F6FF      CALL ehelper1.00408244
00499AE0    8B55 E0          MOV EDX,DWORD PTR SS:[EBP-20]
00499AE3    8D4D F8          LEA ECX,DWORD PTR SS:[EBP-8]
00499AE6    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]
00499AE9    E8 D2FCFFFF      CALL ehelper1.004997C0              ; 这里是关键 CALL，要跟进找算法。
00499AEE    EB 27            JMP SHORT ehelper1.00499B17
00499AF0    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]
00499AF3    8B80 64040000    MOV EAX,DWORD PTR DS:[EAX+464]
00499AF9    8B10             MOV EDX,DWORD PTR DS:[EAX]
00499AFB    FF92 E0000000    CALL DWORD PTR DS:[EDX+E0]
00499B01    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]
00499B04    8B80 64040000    MOV EAX,DWORD PTR DS:[EAX+464]
00499B0A    8B10             MOV EDX,DWORD PTR DS:[EAX]
00499B0C    FF92 C4000000    CALL DWORD PTR DS:[EDX+C4]
00499B12    E9 F3000000      JMP ehelper1.00499C0A
00499B17    8D55 D8          LEA EDX,DWORD PTR SS:[EBP-28]
00499B1A    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]
00499B1D    8B80 68040000    MOV EAX,DWORD PTR DS:[EAX+468]
00499B23    E8 0C89FBFF      CALL ehelper1.00452434
00499B28    8B45 D8          MOV EAX,DWORD PTR SS:[EBP-28]      ; 假码放到 EAX 中。
00499B2B    8B55 F8          MOV EDX,DWORD PTR SS:[EBP-8]       ; 真码放到 EDX 中。
00499B2E    E8 E5A8F6FF      CALL ehelper1.00404418              ; 这里是 CALL 进比较
00499B33    0F85 D1000000    JNZ ehelper1.00499C0A              ; 不相等就跳晕了。
00499B39    B2 01            MOV DL,1
00499B3B    A1 FC764300      MOV EAX,DWORD PTR DS:[4376FC]
00499B40    E8 B7DCF9FF      CALL ehelper1.004377FC
00499B45    8BD8             MOV EBX,EAX
00499B47    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]
00499B4A    8998 20050000    MOV DWORD PTR DS:[EAX+520],EBX
00499B50    33C0             XOR EAX,EAX
00499B52    55               PUSH EBP
00499B53    68 039C4900      PUSH ehelper1.00499C03
00499B58    64:FF30          PUSH DWORD PTR FS:[EAX]
00499B5B    64:8920          MOV DWORD PTR FS:[EAX],ESP
00499B5E    8BC3             MOV EAX,EBX
00499B60    BA 01000080      MOV EDX,80000001
00499B65    E8 32DDF9FF      CALL ehelper1.0043789C
00499B6A    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]
00499B6D    8B80 20050000    MOV EAX,DWORD PTR DS:[EAX+520]
00499B73    B1 01            MOV CL,1
00499B75    BA 809C4900      MOV EDX,ehelper1.00499C80          ; 这里是写注册表的位置。
00499B7A    E8 85DDF9FF      CALL ehelper1.00437904
00499B7F    84C0             TEST AL,AL
00499B81    74 59            JE SHORT ehelper1.00499BDC
00499B83    8D55 D0          LEA EDX,DWORD PTR SS:[EBP-30]
00499B86    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]
00499B89    8B80 64040000    MOV EAX,DWORD PTR DS:[EAX+464]
00499B8F    E8 A088FBFF      CALL ehelper1.00452434
00499B94    8B45 D0          MOV EAX,DWORD PTR SS:[EBP-30]
00499B97    8D55 D4          LEA EDX,DWORD PTR SS:[EBP-2C]
00499B9A    E8 A5E6F6FF      CALL ehelper1.00408244
00499B9F    8B4D D4          MOV ECX,DWORD PTR SS:[EBP-2C]
00499BA2    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]
00499BA5    8B80 20050000    MOV EAX,DWORD PTR DS:[EAX+520]
00499BAB    BA AC9C4900      MOV EDX,ehelper1.00499CAC          ; 用户名
00499BB0    E8 7BE2F9FF      CALL ehelper1.00437E30
00499BB5    8D55 CC          LEA EDX,DWORD PTR SS:[EBP-34]
00499BB8    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]
00499BBB    8B80 68040000    MOV EAX,DWORD PTR DS:[EAX+468]
00499BC1    E8 6E88FBFF      CALL ehelper1.00452434
00499BC6    8B4D CC          MOV ECX,DWORD PTR SS:[EBP-34]
00499BC9    8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]
```



## 关键 CALL 跟进

```text
00499BCC    8B80 20050000   MOV EAX,DWORD PTR DS:[EAX+520]  

00499BD2    BA B89C4900     MOV EDX,ehelper1.00499CB8                ; 注册码  

00499BD7    E8 54E2F9FF     CALL ehelper1.00437E30  

00499BDC    33C0            XOR EAX,EAX  

00499BDE    5A              POP EDX  

00499BDF    59              POP ECX  

00499BE0    59              POP ECX  

00499BE1    64:8910         MOV DWORD PTR FS:[EAX],EDX  

00499BE4    68 0A9C4900     PUSH ehelper1.00499C0A  

00499BE9    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]  

00499BEC    8B80 20050000   MOV EAX,DWORD PTR DS:[EAX+520]  

00499BF2    E8 8596F6FF     CALL ehelper1.0040327C  

00499BF7    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]  

00499BFA    33D2            XOR EDX,EDX  

00499BFC    8990 20050000   MOV DWORD PTR DS:[EAX+520],EDX  

00499C02    C3              RETN  

00499C03   ^ E9 089EF6FF    JMP ehelper1.00403A10  

00499C08   ^ EB DF          JMP SHORT ehelper1.00499BE9  

00499C0A    33C0            XOR EAX,EAX                               ; 跳到这里就死了。  

00499C0C    5A              POP EDX  

00499C0D    59              POP ECX  

00499C0E    59              POP ECX  

00499C0F    64:8910         MOV DWORD PTR FS:[EAX],EDX  

00499C12    68 739C4900     PUSH ehelper1.00499C73  

00499C17    8D45 CC         LEA EAX,DWORD PTR SS:[EBP-34]  

00499C1A    BA 02000000     MOV EDX,2  

00499C1F    E8 0CA4F6FF     CALL ehelper1.00404030  

00499C24    8D45 D4         LEA EAX,DWORD PTR SS:[EBP-2C]  

00499C27    E8 E0A3F6FF     CALL ehelper1.0040400C  

00499C2C    8D45 D8         LEA EAX,DWORD PTR SS:[EBP-28]  

00499C2F    BA 02000000     MOV EDX,2  

00499C34    E8 F7A3F6FF     CALL ehelper1.00404030  

00499C39    8D45 E0         LEA EAX,DWORD PTR SS:[EBP-20]  

00499C3C    E8 CBA3F6FF     CALL ehelper1.0040400C  

00499C41    8D45 E4         LEA EAX,DWORD PTR SS:[EBP-1C]  

00499C44    BA 02000000     MOV EDX,2  

00499C49    E8 E2A3F6FF     CALL ehelper1.00404030  

00499C4E    8D45 EC         LEA EAX,DWORD PTR SS:[EBP-14]  

00499C51    E8 B6A3F6FF     CALL ehelper1.0040400C  

00499C56    8D45 F0         LEA EAX,DWORD PTR SS:[EBP-10]  

00499C59    E8 AEA3F6FF     CALL ehelper1.0040400C  

00499C5E    8D45 F4         LEA EAX,DWORD PTR SS:[EBP-C]  

00499C61    BA 02000000     MOV EDX,2  

00499C66    E8 C5A3F6FF     CALL ehelper1.00404030  

00499C6B    C3              RETN  

00499C6C   ^ E9 9F9DF6FF    JMP ehelper1.00403A10  

00499C71   ^ EB A4          JMP SHORT ehelper1.00499C17  

00499C73    5B              POP EBX  

00499C74    8BE5            MOV ESP,EBP  

00499C76    5D              POP EBP  

00499C77    C3              RETN                                    ; 跑掉了。呵。不管了。  

004997C0    55              PUSH EBP                                  ; CALL 进停在这里。这里实际就在主程序的上面。  

004997C1    8BEC            MOV EBP,ESP  

004997C3    51              PUSH ECX  

004997C4    B9 06000000     MOV ECX,6  

004997C9    6A 00           PUSH 0  

004997CB    6A 00           PUSH 0  

004997CD    49              DEC ECX  

004997CE   ^ 75 F9          JNZ SHORT ehelper1.004997C9  

004997D0    874D FC         XCHG DWORD PTR SS:[EBP-4],ECX  

004997D3    53              PUSH EBX  

004997D4    56              PUSH ESI  

004997D5    57              PUSH EDI  

004997D6    894D F8         MOV DWORD PTR SS:[EBP-8],ECX  

004997D9    8955 FC         MOV DWORD PTR SS:[EBP-4],EDX  

004997DC    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]  

004997DF    E8 D8ACF6FF     CALL ehelper1.004044BC  

004997E4    33C0            XOR EAX,EAX  

004997E6    55              PUSH EBP  

004997E7    68 1A9A4900     PUSH ehelper1.00499A1A  

004997EC    64:FF30         PUSH DWORD PTR FS:[EAX]  

004997EF    64:8920         MOV DWORD PTR FS:[EAX],ESP  

004997F2    33C0            XOR EAX,EAX  

004997F4    8945 F0         MOV DWORD PTR SS:[EBP-10],EAX  

004997F7    33C0            XOR EAX,EAX  

004997F9    8945 EC         MOV DWORD PTR SS:[EBP-14],EAX  

004997FC    33C0            XOR EAX,EAX  

004997FE    8945 E8         MOV DWORD PTR SS:[EBP-18],EAX  

00499801    33C0            XOR EAX,EAX  

00499803    8945 E4         MOV DWORD PTR SS:[EBP-1C],EAX  

00499806    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]             ; 把用户名入到 EAX 中.  

00499809    E8 BEAAF6FF     CALL ehelper1.004042CC                   ; CALL 出用户名的位数.  

0049980E    8BF8            MOV EDI,EAX  

00499810    83FF 05         CMP EDI,5                                 ; 与 5 比较,小于就跳  

00499813    0F8C C3010000   JL ehelper1.004999DC  

00499819    83FF 14         CMP EDI,14                                ; 与 14 比较(20)大于就跳  

0049981C    0F8F BA010000   JG ehelper1.004999DC  

00499822    8BF7            MOV ESI,EDI                               ; 把用户的位数放 ESI 与 EDI 中.  

00499824    85F6            TEST ESI,ESI  

00499826    7C 20           JL SHORT ehelper1.00499848  

00499828    46              INC ESI                                   ; ESI 加 1  

00499829    33DB            XOR EBX,EBX  

0049982B    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]             ; 把用户名放 EAX 中.  

0049982E    8A4418 FF       MOV AL,BYTE PTR DS:[EAX+EBX-1]           ; 逐位取用户名的 ASCII 值  

00499832    33D2            XOR EDX,EDX  

00499834    8AD0            MOV DL,AL                                 ; 把取得值放到 DL 中.  

00499836    0155 F0         ADD DWORD PTR SS:[EBP-10],EDX             ; 这里是累加值  

00499839    25 FF000000     AND EAX,0FF                               ; 这里无用.  

0049983E    C1E0 04         SHL EAX,4                                 ; 这个是把逐位取的 ASCII 值左移 4 位.  

00499841    0145 F0         ADD DWORD PTR SS:[EBP-10],EAX             ; 再加上原来那个值  

00499844    43              INC EBX                                   ; EBX 加 1  

00499845    4E              DEC ESI                                   ; ESI 减 1  

00499846   ^ 75 E3          JNZ SHORT ehelper1.0049982B               ; 小于跳回  

00499848    8BF7            MOV ESI,EDI  

0049984A    85F6            TEST ESI,ESI  

0049984C    7C 18           JL SHORT ehelper1.00499866  

0049984E    46              INC ESI  

0049984F    33DB            XOR EBX,EBX  

00499851    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]             ; 这里又取出用户名。  

00499854    8A4418 FF       MOV AL,BYTE PTR DS:[EAX+EBX-1]           ; 逐位取用户名的 ASCII 放到 AL 中。  

00499858    24 3F           AND AL,3F                                 ; 这里与 3F（63）进行 and 操作。  

0049985A    25 FF000000     AND EAX,0FF                               ; 无用操作！！！  

0049985F    8945 EC         MOV DWORD PTR SS:[EBP-14],EAX             ; 把操作后的值放到栈中。并排  

00499862    43              INC EBX                                   ; EBX 做为循环变量。加 1  

00499863    4E              DEC ESI                                   ; ESI 减去 1  

00499864   ^ 75 EB          JNZ SHORT ehelper1.00499851  

00499866    8BF7            MOV ESI,EDI  

00499868    85F6            TEST ESI,ESI  

0049986A    0F8C B7000000   JL ehelper1.00499927  

00499870    46              INC ESI  

00499871    33DB            XOR EBX,EBX  

00499873    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]             ; 再次取出用户名。放 EAX 中。  

00499876    807C18 FF 41    CMP BYTE PTR DS:[EAX+EBX-1],41           ; 取用户名 ASCII 与 41 比较（65）  

0049987B    72 21           JB SHORT ehelper1.0049989E               ; 小于就跳走。  

0049987D    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]  

00499880    807C18 FF 47    CMP BYTE PTR DS:[EAX+EBX-1],47  

00499885    77 17           JA SHORT ehelper1.0049989E               ; 不小于或不等于时跳走。否则就往下走。  

00499887    8D45 FC         LEA EAX,DWORD PTR SS:[EBP-4]  

0049988A    E8 95ACF6FF     CALL ehelper1.00404524  
```



```
0049988F    8B55 FC         MOV EDX,DWORD PTR SS:\[EBP-4\]             ; 这里把用户名放 EAX 中。  

00499892    0FB6541A FF     MOVZX EDX,BYTE PTR DS:\[EDX+EBX-1\]       ; 这里是取用户名的每一位的 ASCII 值。  

00499897    83C2 05         ADD EDX,5                                   ; 这里如果满足上面条件就把相应位的 ASCII 加 5  

0049989A    885418 FF       MOV BYTE PTR DS:\[EAX+EBX-1\],DL         ; 这里把得到的值存放起来  

0049989E    8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]             ; 把用户名放 EAX 中。  

004998A1    807C18 FF 48    CMP BYTE PTR DS:\[EAX+EBX-1\],48         ; 与 48 比较  

004998A6    72 21           JB SHORT ehelper1.004998C9  

004998A8    8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  

004998AB    807C18 FF 5A    CMP BYTE PTR DS:\[EAX+EBX-1\],5A         ; 与 5A 比较  

004998B0    77 17           JA SHORT ehelper1.004998C9  

004998B2    8D45 FC         LEA EAX,DWORD PTR SS:\[EBP-4\]  

004998B5    E8 6AACF6FF     CALL ehelper1.00404524  

004998BA    8B55 FC         MOV EDX,DWORD PTR SS:\[EBP-4\]  

004998BD    0FB6541A FF     MOVZX EDX,BYTE PTR DS:\[EDX+EBX-1\]  

004998C2    83EA 02         SUB EDX,2                                   ; 这里是减 2  

004998C5    885418 FF       MOV BYTE PTR DS:\[EAX+EBX-1\],DL  

004998C9    8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  

004998CC    807C18 FF 61    CMP BYTE PTR DS:\[EAX+EBX-1\],61         ; 与 61 比较  

004998D1    72 21           JB SHORT ehelper1.004998F4  

004998D3    8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  

004998D6    807C18 FF 70    CMP BYTE PTR DS:\[EAX+EBX-1\],70         ; 与 70 比较  

004998DB    77 17           JA SHORT ehelper1.004998F4  

004998DD    8D45 FC         LEA EAX,DWORD PTR SS:\[EBP-4\]  

004998E0    E8 3FACF6FF     CALL ehelper1.00404524  

004998E5    8B55 FC         MOV EDX,DWORD PTR SS:\[EBP-4\]  

004998E8    0FB6541A FF     MOVZX EDX,BYTE PTR DS:\[EDX+EBX-1\]  

004998ED    83C2 05         ADD EDX,5                                   ; 这里加 5  

004998F0    885418 FF       MOV BYTE PTR DS:\[EAX+EBX-1\],DL  

004998F4    8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  

004998F7    807C18 FF 71    CMP BYTE PTR DS:\[EAX+EBX-1\],71         ; 与 71 比较  

004998FC    72 21           JB SHORT ehelper1.0049991F  

004998FE    8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]  

00499901    807C18 FF 7A    CMP BYTE PTR DS:\[EAX+EBX-1\],7A         ; 与 7A 比较  

00499906    77 17           JA SHORT ehelper1.0049991F  

00499908    8D45 FC         LEA EAX,DWORD PTR SS:\[EBP-4\]  

0049990B    E8 14ACF6FF     CALL ehelper1.00404524  

00499910    8B55 FC         MOV EDX,DWORD PTR SS:\[EBP-4\]  

00499913    0FB6541A FF     MOVZX EDX,BYTE PTR DS:\[EDX+EBX-1\]  

00499918    83EA 03         SUB EDX,3                                   ; 这里是减 3  

0049991B    885418 FF       MOV BYTE PTR DS:\[EAX+EBX-1\],DL         ; 以上是同理。进行运算。得到新的用户名的值。  

0049991F    43              INC EBX  

00499920    4E              DEC ESI  

00499921  ^ 0F85 4CFFFFFF  JNZ ehelper1.00499873                    ; 取完后把新得到的用户名放到 EAX 中。  

00499927    8BF7            MOV ESI,EDI                                ; 把用户名的位数放到 ESI 中。  

00499929    85F6            TEST ESI,ESI  

0049992B    7C 20           JL SHORT ehelper1.0049994D  

0049992D    46              INC ESI  

0049992E    33DB            XOR EBX,EBX  

00499930    8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]             ; 把新得到的用户名放到 EAX 中。  

00499933    8A4418 FF       MOV AL,BYTE PTR DS:\[EAX+EBX-1\]           ; 取新用户名的每一位 ASCII 放到 AL 中。  

00499937    33D2            XOR EDX,EDX                                ; 异或 EDX  

00499939    8AD0            MOV DL,AL                                  ; AL 放到 DL 中。  

0049993B    0155 E8         ADD DWORD PTR SS:\[EBP-18\],EDX            ; 这里是累加。  

0049993E    25 FF000000     AND EAX,0FF  

00499943    C1E0 07         SHL EAX,7                                   ; 把每一位取到的值左移 7 位。  

00499946    0145 E8         ADD DWORD PTR SS:\[EBP-18\],EAX            ; 然后再加上栈里的值。  

00499949    43              INC EBX                                     ; EBX 加一  

0049994A    4E              DEC ESI                                     ; ESI 减一  

0049994B  ^ 75 E3           JNZ SHORT ehelper1.00499930  

0049994D    8BF7            MOV ESI,EDI  

0049994F    85F6            TEST ESI,ESI  

00499951    7C 18           JL SHORT ehelper1.0049996B  

00499953    46              INC ESI  

00499954    33DB            XOR EBX,EBX  

00499956    8B45 FC         MOV EAX,DWORD PTR SS:\[EBP-4\]             ; 把用新户名放到 EAX 中。  

00499959    8A4418 FF       MOV AL,BYTE PTR DS:\[EAX+EBX-1\]           ; 这里又取新用户名的每一位 ASXII  

0049995D    24 3F           AND AL,3F                                   ; 与 3F 进行 and 操作。  

0049995F    25 FF000000     AND EAX,0FF  

00499964    8945 E4         MOV DWORD PTR SS:\[EBP-1C\],EAX            ; 入栈。  

00499967    43              INC EBX  

00499968    4E              DEC ESI  

00499969  ^ 75 EB           JNZ SHORT ehelper1.00499956  

0049996B    8D55 DC         LEA EDX,DWORD PTR SS:\[EBP-24\]  

0049996E    8B45 F0         MOV EAX,DWORD PTR SS:\[EBP-10\]            ; 这里是取原来用户名的运算后的值放到 EAX 中。  

00499971    0345 EC         ADD EAX,DWORD PTR SS:\[EBP-14\]            ; 加上 33  

00499974    0345 E8         ADD EAX,DWORD PTR SS:\[EBP-18\]            ; 再加上新户名的运算值。得到的值是注册码的一部分。  

00499977    E8 C8E9F6FF     CALL ehelper1.00408344                     ; 把十进制 CALL 出。  

0049997C    8B45 DC         MOV EAX,DWORD PTR SS:\[EBP-24\]            ; 把这个值放到 EAX 中。  

0049997F    8D55 E0         LEA EDX,DWORD PTR SS:\[EBP-20\]  

00499982    E8 CDB4F9FF     CALL ehelper1.00434E54                     ; 这个 CALL 是把上面得运算进行变换。  

00499987    FF75 E0         PUSH DWORD PTR SS:\[EBP-20\]  

0049998A    8D55 D4         LEA EDX,DWORD PTR SS:\[EBP-2C\]  

0049998D    8B45 E8         MOV EAX,DWORD PTR SS:\[EBP-18\]            ; 这里是把新用户名放到 EAX 中。  

00499990    0345 E4         ADD EAX,DWORD PTR SS:\[EBP-1C\]            ; 再加上一个 33  

00499993    03C7            ADD EAX,EDI                                 ; 再加上用户名的长度。得到注册码的别一部分。  

00499995    E8 AAE9F6FF     CALL ehelper1.00408344                     ; 以十进制 CALL 出。  

0049999A    8B45 D4         MOV EAX,DWORD PTR SS:\[EBP-2C\]            ; 把 CALL 出的值放到 EAX 中。  

0049999D    8D55 D8         LEA EDX,DWORD PTR SS:\[EBP-28\]  

004999A0    E8 AFB4F9FF     CALL ehelper1.00434E54  

004999A5    FF75 D8         PUSH DWORD PTR SS:\[EBP-28\]  

004999A8    8D55 D0         LEA EDX,DWORD PTR SS:\[EBP-30\]  

004999AB    8BC7            MOV EAX,EDI                                  ; 把用户名的位数放到 EAX 中。  

004999AD    03C0            ADD EAX,EAX                                   ; 把用户名的位数自加，得到注册码的另一部分。  

004999AF    E8 90E9F6FF     CALL ehelper1.00408344                     ; 这个 CALL 出十进制值。  

004999B4    FF75 D0         PUSH DWORD PTR SS:\[EBP-30\]  

004999B7    8D45 F4         LEA EAX,DWORD PTR SS:\[EBP-C\]  

004999BA    BA 03000000     MOV EDX,3  

004999BF    E8 C8A9F6FF     CALL ehelper1.0040438C                     ; 这个 CALL 是得到逆向值。  

004999C4    8D55 CC         LEA EDX,DWORD PTR SS:\[EBP-34\]  

004999C7    8B45 F4         MOV EAX,DWORD PTR SS:\[EBP-C\]             ; 这里得到了注册码的逆式值。放 EAX 中。  

004999CA    E8 85B4F9FF     CALL ehelper1.00434E54                     ; 这个 CALL 是把这个值取反。  

004999CF    8B55 CC         MOV EDX,DWORD PTR SS:\[EBP-34\]  

004999D2    8D45 F4         LEA EAX,DWORD PTR SS:\[EBP-C\]             ; 把真正的注册码放到 EDX 中。  

004999D5    E8 CAA6F6FF     CALL ehelper1.004040A4  

004999DA    EB 08           JMP SHORT ehelper1.004999E4  

004999DC    8D45 F4         LEA EAX,DWORD PTR SS:\[EBP-C\]  

004999DF    E8 28A6F6FF     CALL ehelper1.0040400C  

004999E4    8B45 F8         MOV EAX,DWORD PTR SS:\[EBP-8\]  

004999E7    8B55 F4         MOV EDX,DWORD PTR SS:\[EBP-C\]             ; 把真正的注册码放到 EDX 中。  

004999EA    E8 71A6F6FF     CALL ehelper1.00404060  

004999EF    33C0            XOR EAX,EAX  

004999F1    5A              POP EDX  

004999F2    59              POP ECX  

004999F3    59              POP ECX  
```



```
004999F4    64:8910         MOV DWORD PTR FS:[EAX],EDX  

004999F7    68 219A4900     PUSH ehelper1.00499A21  

004999FC    8D45 CC         LEA EAX,DWORD PTR SS:[EBP-34]  

004999FF    BA 06000000     MOV EDX,6  

00499A04    E8 27A6F6FF     CALL ehelper1.00404030  

00499A09    8D45 F4         LEA EAX,DWORD PTR SS:[EBP-C]  

00499A0C    E8 FBA5F6FF     CALL ehelper1.0040400C                ; 把真正的注册码放到 EDX 中。  

00499A11    8D45 FC         LEA EAX,DWORD PTR SS:[EBP-4]  

00499A14    E8 F3A5F6FF     CALL ehelper1.0040400C  

00499A19    C3              RETN  

00499A1A  ^ E9 F19FF6FF     JMP ehelper1.00403A10  

00499A1F  ^ EB DB          JMP SHORT ehelper1.004999FC  

00499A21    5F              POP EDI  

00499A22    5E              POP ESI  

00499A23    5B              POP EBX  

00499A24    8BE5            MOV ESP,EBP  

00499A26    5D              POP EBP  

00499A27    C3              RETN                              ; 返回（算法结结束）  
```

总结算法：

1、取输入的用户名进行运算记为 `A`

2、然后通过比较运算得到新的用户名进行运算记为 `B`

3、把新用户名与我们输入的用户运算结果再进行运算。得到 `C`

4、把用户名的位数乘 2，得到注册码的第三部分。记为 `D`

5、然后取上面运算的逆向值。得到真正的注册码。

6、用户名的位数必须大于 5 位。

算法比较麻烦。上面总结可能不到位。写注册机比较麻烦，先不写了。提供几位注册码：

- 用户名：`wxh9833`     注册码：`417191181390`
- 用户名：`WilldcatIII`  注册码：`22135726153380`
- 用户名：`冷血书生`      注册码：`61217431246068`（中文测试成功）

**版权声明**

本破文纯属技术交流，转载请注明作者并保持文章的完整，谢谢！

`4dd466d0e1e0cc123afbe504da2462ca`

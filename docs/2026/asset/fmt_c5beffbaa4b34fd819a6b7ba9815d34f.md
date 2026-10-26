
# lansee 1.59算法分析[原创]

## 【破文标题】
lansee 1.59算法分析

## 【破文作者】
无幻刹那

## 【作者性别】
男

## 【作者邮箱】
lovecy616927@yahoo.com.cn

## 【软件名称】
LanSee) V1.59

## 【下载地址】
cd9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4Z5j5i4u0W2N6$3q4J5k6g2)9J5k6h3y4F1i4K6u0r3M7s2g2T1i4K6u0r3x3e0l9%4x3q4)9J5k6h3S2@1L8h3H3`.

## 【破解工具】
od

## 【保护方式】
注册码保护方式

## 【破解难度】
简单

----------------------------------------------------

### 软件介绍：
一个局域网查看得工具，扫描共享的

----------------------------------------------------

### 破解声名：
初学Crack，只是感兴趣，消遣业余时间，错误之处敬请诸位前辈不吝赐教

----------------------------------------------------

## 【破解分析】

软件没加壳，用delphi编写，就直接用od载入查找超级字串参考＋文本字串=`您输入的注册名或注册码无效，请重新输入！`双击来到下面，

并下断如下面，运行软件的注册，注册名随便输入,注册码:`a1349697a112` (是真码),

程序就断在了下面，第一次写算法分析，有点紧张~

```
00491BE0  /.  55            PUSH EBP     //这里下断                          ;  (初始 cpu 选择)
00491BE1  |.  8BEC          MOV EBP,ESP
00491BE3  |.  6A 00         PUSH 0
00491BE5  |.  6A 00         PUSH 0
00491BE7  |.  53            PUSH EBX
00491BE8  |.  56            PUSH ESI
00491BE9  |.  8BF0          MOV ESI,EAX
00491BEB  |.  33C0          XOR EAX,EAX
00491BED  |.  55            PUSH EBP
00491BEE  |.  68 C51C4900   PUSH 局域网查.00491CC5
00491BF3  |.  64:FF30       PUSH DWORD PTR FS:[EAX]
00491BF6  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP
00491BF9  |.  8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]
00491BFC  |.  8B86 FC020000 MOV EAX,DWORD PTR DS:[ESI+2FC]
00491C02  |.  E8 C11BFBFF   CALL 局域网查.004437C8                       ;  取注册假码,送入eax
00491C07  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00491C0A  |.  E8 69FFFFFF   CALL 局域网查.00491B78                       ;  关键call，f7跟进，里面就是算法
00491C0F  |.  84C0          TEST AL,AL                               ;  al取0则跳到注册失败，返回1则注册成功
00491C11  |.  0F84 89000000 JE 局域网查.00491CA0                         ;  爆破点，
00491C17  |.  B2 01         MOV DL,1
00491C19  |.  A1 B4F34600   MOV EAX,DWORD PTR DS:[46F3B4]
00491C1E  |.  E8 91D8FDFF   CALL 局域网查.0046F4B4
00491C23  |.  8BD8          MOV EBX,EAX
00491C25  |.  BA 02000080   MOV EDX,80000002
00491C2A  |.  8BC3          MOV EAX,EBX
00491C2C  |.  E8 23D9FDFF   CALL 局域网查.0046F554
00491C31  |.  B1 01         MOV CL,1
00491C33  |.  BA DC1C4900   MOV EDX,局域网查.00491CDC                    ;  software\lansee
00491C38  |.  8BC3          MOV EAX,EBX
00491C3A  |.  E8 79D9FDFF   CALL 局域网查.0046F5B8
00491C3F  |.  84C0          TEST AL,AL
00491C41  |.  74 1D         JE SHORT 局域网查.00491C60
00491C43  |.  8D55 F8       LEA EDX,DWORD PTR SS:[EBP-8]
00491C46  |.  8B86 FC020000 MOV EAX,DWORD PTR DS:[ESI+2FC]
00491C4C  |.  E8 771BFBFF   CALL 局域网查.004437C8
00491C51  |.  8B4D F8       MOV ECX,DWORD PTR SS:[EBP-8]
00491C54  |.  BA F41C4900   MOV EDX,局域网查.00491CF4                    ;  serial
00491C59  |.  8BC3          MOV EAX,EBX
00491C5B  |.  E8 F4DAFDFF   CALL 局域网查.0046F754
00491C60  |>  8BC3          MOV EAX,EBX
00491C62  |.  E8 BDD8FDFF   CALL 局域网查.0046F524
00491C67  |.  8BC3          MOV EAX,EBX
00491C69  |.  E8 C61AF7FF   CALL 局域网查.00403734
00491C6E  |.  A1 94124A00   MOV EAX,DWORD PTR DS:[4A1294]
00491C73  |.  C600 01       MOV BYTE PTR DS:[EAX],1
00491C76  |.  A1 DC154A00   MOV EAX,DWORD PTR DS:[4A15DC]
00491C7B  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]
00491C7D  |.  8B80 24040000 MOV EAX,DWORD PTR DS:[EAX+424]
00491C83  |.  BA 10000000   MOV EDX,10
00491C88  |.  E8 AFA1FFFF   CALL 局域网查.0048BE3C
00491C8D  |.  B8 041D4900   MOV EAX,局域网查.00491D04                    ;  谢谢您的注册！
00491C92  |.  E8 F9B2FAFF   CALL 局域网查.0043CF90
00491C97  |.  8BC6          MOV EAX,ESI
00491C99  |.  E8 DAE4FCFF   CALL 局域网查.00460178
00491C9E  |.  EB 0A         JMP SHORT 局域网查.00491CAA
00491CA0  |>  B8 1C1D4900   MOV EAX,局域网查.00491D1C                    ;  您输入的注册名或注册码无效，请重新输入！；来到这里，
00491CA5  |.  E8 E6B2FAFF   CALL 局域网查.0043CF90
00491CAA  |>  33C0          XOR EAX,EAX
00491CAC  |.  5A            POP EDX
00491CAD  |.  59            POP ECX
00491CAE  |.  59            POP ECX
00491CAF  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX
00491CB2  |.  68 CC1C4900   PUSH 局域网查.00491CCC
00491CB7  |>  8D45 F8       LEA EAX,DWORD PTR SS:[EBP-8]
00491CBA  |.  BA 02000000   MOV EDX,2
00491CBF  |.  E8 8428F7FF   CALL 局域网查.00404548
00491CC4  \.  C3            RETN
```

==========================================================

上面跟进来到

```
00491B78  /$  55            PUSH EBP
00491B79  |.  8BEC          MOV EBP,ESP
00491B7B  |.  51            PUSH ECX
00491B7C  |.  53            PUSH EBX
00491B7D  |.  8945 FC       MOV DWORD PTR SS:[EBP-4],EAX
00491B80  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00491B83  |.  E8 3C2EF7FF   CALL 局域网查.004049C4
00491B88  |.  33C0          XOR EAX,EAX
00491B8A  |.  55            PUSH EBP
00491B8B  |.  68 C91B4900   PUSH 局域网查.00491BC9
00491B90  |.  64:FF30       PUSH DWORD PTR FS:[EAX]
00491B93  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP
00491B96  |.  33DB          XOR EBX,EBX
00491B98  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00491B9B  |.  E8 3C2CF7FF   CALL 局域网查.004047DC                       ;  取注册码的位数
00491BA0  |.  83F8 0A       CMP EAX,0A                               ;  如果小于10位，则挂，根据后面的算法，应该为12位
00491BA3  |.  7C 0E         JL SHORT 局域网查.00491BB3
00491BA5  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]             ;  我们输入的假注册码送入eax
00491BA8  |.  E8 F7FCFFFF   CALL 局域网查.004918A4                       ;  算法，f7跟进
00491BAD  |.  84C0          TEST AL,AL                               ;  al返回0则失败，1成功
00491BAF  |.  74 02         JE SHORT 局域网查.00491BB3
00491BB1  |.  B3 01         MOV BL,1
00491BB3  |>  33C0          XOR EAX,EAX
00491BB5  |.  5A            POP EDX
00491BB6  |.  59            POP ECX
00491BB7  |.  59            POP ECX
00491BB8  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX
00491BBB  |.  68 D01B4900   PUSH 局域网查.00491BD0
00491BC0  |>  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
00491BC3  |.  E8 5C29F7FF   CALL 局域网查.00404524
00491BC8  \.  C3            RETN
00491BC9   .^ E9 FA22F7FF   JMP 局域网查.00403EC8
00491BCE   .^ EB F0         JMP SHORT 局域网查.00491BC0
00491BD0   .  8BC3          MOV EAX,EBX
00491BD2   .  5B            POP EBX
00491BD3   .  59            POP ECX
00491BD4   .  5D            POP EBP
```

===============================================

上面跟进来到~

```
004918A4  /$  55            PUSH EBP
004918A5  |.  8BEC          MOV EBP,ESP
004918A7  |.  83C4 BC       ADD ESP,-44
004918AA  |.  53            PUSH EBX
004918AB  |.  56            PUSH ESI
004918AC  |.  33D2          XOR EDX,EDX                              ;  edx清零
004918AE  |.  8955 BC       MOV DWORD PTR SS:[EBP-44],EDX            ;  分配空间，以下同理
004918B1  |.  8955 C0       MOV DWORD PTR SS:[EBP-40],EDX
004918B4  |.  8955 C4       MOV DWORD PTR SS:[EBP-3C],EDX
004918B7  |.  8955 C8       MOV DWORD PTR SS:[EBP-38],EDX
004918BA  |.  8955 F0       MOV DWORD PTR SS:[EBP-10],EDX
004918BD  |.  8955 EC       MOV DWORD PTR SS:[EBP-14],EDX
004918C0  |.  8955 F8       MOV DWORD PTR SS:[EBP-8],EDX
004918C3  |.  8945 FC       MOV DWORD PTR SS:[EBP-4],EAX
004918C6  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004918C9  |.  E8 F630F7FF   CALL 局域网查.004049C4
004918CE  |.  33C0          XOR EAX,EAX
004918D0  |.  55            PUSH EBP
004918D1  |.  68 571B4900   PUSH 局域网查.00491B57
004918D6  |.  64:FF30       PUSH DWORD PTR FS:[EAX]
004918D9  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP
004918DC  |.  33DB          XOR EBX,EBX
004918DE  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004918E1  |.  E8 F62EF7FF   CALL 局域网查.004047DC
004918E6  |.  8D55 F0       LEA EDX,DWORD PTR SS:[EBP-10]
004918E9  |.  E8 5E74F7FF   CALL 局域网查.00408D4C
004918EE  |.  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]
004918F1  |.  50            PUSH EAX
004918F2  |.  8D45 E8       LEA EAX,DWORD PTR SS:[EBP-18]
004918F5  |.  8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]
004918F8  |.  8A52 01       MOV DL,BYTE PTR DS:[EDX+1]
004918FB  |.  8850 01       MOV BYTE PTR DS:[EAX+1],DL
004918FE  |.  C600 01       MOV BYTE PTR DS:[EAX],1                  ;  取 我们输入的假注册码的数
```

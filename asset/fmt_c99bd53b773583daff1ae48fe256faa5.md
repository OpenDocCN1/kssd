
# Obsidium 1.2.5.0 主程序脱壳记录点滴

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-523.htm`](https://www.pediy.com/kssd/pediy07/pediy7-523.htm)

这不是完整的脱文，只是简单记录一下我自己认为的一些要点，太多的细节我也不知所以然，

能蒙混过去的我就暂且欺骗一下自己了，我也懒得去跟踪每一段代码的细节

这个 `1.25` 版本和上次脱的 `1.20` 版难易度基本差不多，现在完全忘了上次怎么脱的了

这次多花了不少时间，所以这次才决定留下点简单记录

如果你自己亲自跟踪过，我想应该能看懂我的思路

开工喽：

```
set seh:memory voilation
set seh:div 0
ignore others
```

### 第一次 div 0

```
008D9A28   F7F0              DIV EAX                       ;第一次 div 0
008D9A2A   8381 B8000000 02 ADD DWORD PTR DS:[ECX+B8],2
008D9A31   5B               POP EBX
008D9A32   5D               POP EBP
008D9A33   C3               RETN

008D99DB   64:8F00          POP DWORD PTR FS:[EAX]         ;goto here
008D99DE   83C4 04          ADD ESP,4
008D99E1   5B               POP EBX
008D99E2   C3               RETN
```

### BP VirtualAlloc

```
bp VirtualAlloc
```

```
0044DE1C   FF93 80000000    CALL DWORD PTR DS:[EBX+80]     ;VirtualAlloc
0044DE22   85C0             TEST EAX,EAX                   ;返回这里，eax=9F0000
0044DE24   0F84 3A010000    JE Obsidium.0044DF64           ;new imagebase
```

下面开始解码过程

解码后这里处理 `reloc`，为了脱壳后使 `base` 定位在 `400000h`，就需要在这里做些手脚了，大家自己想吧

```
0044DF27   8B43 10          MOV EAX,DWORD PTR DS:[EBX+10]    ;new imagebase
0044DF2A   2B43 10          SUB EAX,DWORD PTR DS:[EBX+3C]    ;old imagebase
0044DF2D   FF77 08          PUSH DWORD PTR DS:[EDI+8]
0044DF30   FF77 04          PUSH DWORD PTR DS:[EDI+4]
0044DF33   FF73 78          PUSH DWORD PTR DS:[EBX+78]
0044DF36   50               PUSH EAX
0044DF37   FF73 10          PUSH DWORD PTR DS:[EBX+10]
0044DF3A   FF53 70          CALL DWORD PTR DS:[EBX+70]
0044DF3D   83C7 14          ADD EDI,14
0044DF40   FF4D FC          DEC DWORD PTR SS:[EBP-4]
0044DF43   0F85 F4FEFFFF    JNZ Obsidium.0044DE3D
```

继续 `shift F9` 几次看到这样的代码：

```
008DC4B2   F7F0             DIV EAX                       ;div 0
008DC4B4   8B00             MOV EAX,DWORD PTR DS:[EAX]     ;紧接着一个这样的异常
008DC4B6   85C0             TEST EAX,EAX
008DC4B8   74 19            JE SHORT 008DC4D3
008DC4BA   8B53 04          MOV EDX,DWORD PTR DS:[EBX+4]
008DC4BD   C702 07000000    MOV DWORD PTR DS:[EDX],7
008DC4C3   C740 08 00000000 MOV DWORD PTR DS:[EAX+8],0
008DC4CA   8343 04 0C       ADD DWORD PTR DS:[EBX+4],0C
008DC4CE   52               PUSH EDX
008DC4CF   51               PUSH ECX
008DC4D0   FF63 4C          JMP DWORD PTR DS:[EBX+4C]

008DC4D3   64:67:8F06 0000  POP DWORD PTR FS:[0]           ;goto here
008DC4D9   83C4 04          ADD ESP,4
008DC4DC   5B               POP EBX
008DC4DD   C3               RETN
```

### IAT 处理

`ctrl+g` `8DC5D0` , hard break point on exec

（关于 `8DC5D0` 这个地址是怎么找到的，我是多次运行过去，观察后下硬件断点，逐层回溯过来的

然后把它记下来了，让我重新再找一次的话，我也要尝试并思考几次才能定位的）

`F9` 两次停到这里：

```
008DC5D0   E8 3F040000      CALL 008DCA14
008DC5D5   85C0             TEST EAX,EAX
008DC5D7   74 55            JE SHORT 008DC62E
008DC5D9   8B45 F4          MOV EAX,DWORD PTR SS:[EBP-C]
008DC5DC   05 679D4100      ADD EAX,419D67
008DC5E1   8B56 04          MOV EDX,DWORD PTR DS:[ESI+4]
008DC5E4   0353 44          ADD EDX,DWORD PTR DS:[EBX+44]
008DC5E7   52               PUSH EDX
008DC5E8   50               PUSH EAX
008DC5E9   E8 26040000      CALL 008DCA14
008DC5EE   85C0             TEST EAX,EAX
008DC5F0   74 3C            JE SHORT 008DC62E
...
008DC692   FF36             PUSH DWORD PTR DS:[ESI]
008DC694   53               PUSH EBX
008DC695   52               PUSH EDX
008DC696   50               PUSH EAX
008DC697   FF76 0C          PUSH DWORD PTR DS:[ESI+C]
008DC69A   E8 F4000000      CALL 008DC793                  ;进入
```

下面关于还原 `IAT` 代码的修改，我直接借鉴前人有关 `OB` 脱壳文章中的方法，仍然有效

```
008DC793   55               PUSH EBP
008DC794   8BEC             MOV EBP,ESP
008DC796   56               PUSH ESI
008DC797   57               PUSH EDI
008DC798   8B75 10          MOV ESI,DWORD PTR SS:[EBP+10]
008DC79B   8B7D 0C          MOV EDI,DWORD PTR SS:[EBP+C]
008DC79E   66:F706 2000    TEST WORD PTR DS:[ESI],20       ;改为 TEST WORD PTR DS:[ESI],8
008DC7A3   74 46            JE SHORT 008DC7EB               ;改为 JNE SHORT 008DC7EB
008DC7A5   66:F706 0200    TEST WORD PTR DS:[ESI],2
008DC7AA   75 1F            JNZ SHORT 008DC7CB
008DC7AC   66:C706 0400    MOV WORD PTR DS:[ESI],4
008DC7B1   8B45 14          MOV EAX,DWORD PTR SS:[EBP+14]
008DC7B4   6A 01            PUSH 1
008DC7B6   6A 00            PUSH 0
008DC7B8   FF76 04          PUSH DWORD PTR DS:[ESI+4]
008DC7BB   6A 00            PUSH 0
008DC7BD   FF75 18          PUSH DWORD PTR SS:[EBP+18]
008DC7C0   FF50 50          CALL DWORD PTR DS:[EAX+50]
008DC7C3   85C0             TEST EAX,EAX                    ;改为 JE SHORT 008DC7EB
008DC7C5   74 38            JE SHORT 008DC7FF
008DC7C7   8907             MOV DWORD PTR DS:[EDI],EAX
008DC7C9   EB 20            JMP SHORT 008DC7EB
008DC7CB   66:C706 0400    MOV WORD PTR DS:[ESI],4
008DC7D0   8B45 14          MOV EAX,DWORD PTR SS:[EBP+14]
008DC7D3   0FB756 02        MOVZX EDX,WORD PTR DS:[ESI+2]
008DC7D7   6A 01            PUSH 1
008DC7D9   52               PUSH EDX
008DC7DA   6A 00            PUSH 0
008DC7DC   FF76 04          PUSH DWORD PTR DS:[ESI+4]
008DC7DF   FF75 18          PUSH DWORD PTR SS:[EBP+18]
008DC7E2   FF50 50          CALL DWORD PTR DS:[EAX+50]
008DC7E5   85C0             TEST EAX,EAX
008DC7E7   74 16            JE SHORT 008DC7FF               ;改为 JE SHORT 008DC7EB
008DC7E9   8907             MOV DWORD PTR DS:[EDI],EAX
008DC7EB   83C6 08          ADD ESI,8
008DC7EE   83C7 04          ADD EDI,4
008DC7F1   FF4D 08          DEC DWORD PTR SS:[EBP+8]
008DC7F4   75 A8            JNZ SHORT 008DC79E
008DC7F6   33C0             XOR EAX,EAX
008DC7F8   40               INC EAX
008DC7F9   5F               POP EDI
008DC7FA   5E               POP ESI
008DC7FB   5D               POP EBP
008DC7FC   C2 1400          RETN 14
```

`stolen code` 我没去找，我只是翻看一下代码，看到下面这个就知道 `oep` 就在附近，自己手工修复几行就可以了

```
009F15FE         43 2B 2B 48 4F 4F 4K                   C++HOOK
```

`OB` 是从这里开始进入原始程序代码的：

```
009F161E   E8 A1B41500    CALL 00B4CAC4
009F1623   8BD0           MOV EDX,EAX
009F1625   E8 46E11400    CALL 00B3F770
009F162A   5A             POP EDX
009F162B   E8 A4E01400    CALL 00B3F6D4
009F1630   E8 7BE11400    CALL 00B3F7B0
009F1635   6A 00          PUSH 0
009F1637   E8 80F61400    CALL 00B40CBC
```

象下面这种结构是动态解码的 `SDK`

```
009F1BB0   68 A4000000    PUSH 0A4                         ;len
009F1BB5   FF15 EA104000  CALL DWORD PTR DS:[4010EA]
...
009F1C5F   68 A4000000    PUSH 0A4
009F1C64   FF15 EE104000  CALL DWORD PTR DS:[4010EE]
```

修改下面代码用于 `SDK FIX`：

```
008DB38C   55               PUSH EBP
008DB38D   8BEC             MOV EBP,ESP
008DB38F   81EC 30010000    SUB ESP,130
008DB395   60               PUSHAD
008DB396   E8 00000000      CALL 008DB39B
008DB39B   5E               POP ESI
008DB39C   8D96 60010000    LEA EDX,DWORD PTR DS:[ESI+160]
008DB3A2   33C0             XOR EAX,EAX
008DB3A4   52               PUSH EDX
008DB3A5   83C5 04          ADD EBP,4                       ;已修改
008DB3A8   BE 7FD34400      MOV ESI,44D37F                  ;
008DB3AD   90               NOP                             ;
008DB3AE   90               NOP                             ;
008DB3AF   EB 02            JMP SHORT 008DB3B3              ;
008DB3B1   ^EB FA           JMP SHORT 008DB3AD
008DB3B3   8B7D 04          MOV EDI,DWORD PTR SS:[EBP+4]
008DB3B6   8D85 D0FEFFFF    LEA EAX,DWORD PTR SS:[EBP-130]
008DB3BC   50               PUSH EAX
008DB3BD   57               PUSH EDI
```

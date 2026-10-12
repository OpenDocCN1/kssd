
```
004CD100  |.  8D55 E4       LEA EDX,DWORD PTR SS:[EBP-1C]

004CD103  |.  E8 74B9F3FF   CALL unpacked.00408A7C

004CD108  |.  8B45 E4       MOV EAX,DWORD PTR SS:[EBP-1C]

004CD10B  |.  E8 A4ABFEFF   CALL unpacked.004B7CB4

004CD110  |.  8BC6          MOV EAX,ESI

004CD112  |.  E8 491DF8FF   CALL unpacked.0044EE60

004CD117  |.  EB 0A         JMP SHORT unpacked.004CD123

004CD119  |>  B8 78D14C00   MOV EAX,unpacked.004CD178

004CD11E  |.  E8 B5B9F8FF   CALL unpacked.00458AD8               ; 出错框

004CD123  |>  33C0          XOR EAX,EAX

004CD125  |.  5A            POP EDX
```

### 三、去掉可恶的 NAG 窗口

使用`ollydbg`如下功能：
- 菜单：`Debug/animate over` 功能
- 在跳出`nag`窗口这句会停在一句`call`
- 向上看有没有跳转，没有的话，按`F7`进入，继续单步调试（Step Over/Trace），直到找到能跳过`NAG`的`CALL`。

代码如下：

```
004D059A     /7D 5D         JGE SHORT unpacked.004D05F9  ; 把这句改为 jmp SHORT unpacked.004D05F9

004D059C    . |33C0         XOR EAX,EAX

004D059E    . |55           PUSH EBP

004D059F    . |68 F2054D00  PUSH unpacked.004D05F2

004D05A4    . |64:FF30      PUSH DWORD PTR FS:[EAX]

004D05A7    . |64:8920      MOV DWORD PTR FS:[EAX],ESP

004D05AA    . |A1 E8004E00  MOV EAX,DWORD PTR DS:[4E00E8]

004D05AF    . |8338 00      CMP DWORD PTR DS:[EAX],0

004D05B2    . |75 17        JNZ SHORT unpacked.004D05CB

004D05B4    . |8B4D FC      MOV ECX,DWORD PTR SS:[EBP-4]

004D05B7    . |B2 01        MOV DL,1

004D05B9    . |A1 24D34C00  MOV EAX,DWORD PTR DS:[4CD324]

004D05BE    . |E8 BDAAF7FF  CALL unpacked.0044B080

004D05C3    . |8B15 E8004E00 MOV EDX,DWORD PTR DS:[4E00E8]      ; unpacked.004E1B60

004D05C9    . |8902         MOV DWORD PTR DS:[EDX],EAX

004D05CB    > |A1 E8004E00   MOV EAX,DWORD PTR DS:[4E00E8]

004D05D0    . |8B00         MOV EAX,DWORD PTR DS:[EAX]

004D05D2    . |8B10         MOV EDX,DWORD PTR DS:[EAX]

004D05D4    . |FF92 D8000000 CALL DWORD PTR DS:[EDX+D8]         ; => 跳出 nag

004D05DA    . |33C0         XOR EAX,EAX
```

#### 修改

将`004D059A /7D 5D JGE SHORT unpacked.004D05F9`这句改为`jmp SHORT unpacked.004D05F9`，会跳过`NaG`。

#### 备注

没法找到注册码，那位兄弟看看。

`4dd466d0e1e0cc123afbe504da2462ca`

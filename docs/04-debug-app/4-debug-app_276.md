
# 新手练习 - 使 winmine (踩地雷) 不会爆炸

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-137.htm`](https://www.pediy.com/kssd/pediy09/pediy09-137.htm)

> 【文章标题】: 新手练习 - 使 winmine (踩地雷) 不会爆炸  
> 【文章作者】: riijj  
> 【软件名称】: windows 的 winmine (踩地雷游戏)  
> 【加壳方式】: 无壳  
> 【使用工具】: OD  
> 【操作平台】: winxp / win2000  
> 【文章简介】: 给新手练习，启发对控制程序流程的兴趣，没有技术含量，老手们勿看  

## Part 1. 序

平日大家学习的知识，主要针对软件的保护 (例如壳) 和防止被破解。今天换一换口味，一起破坏 windows 自带的小游戏 踩地雷

这是一个简单的程序，只有一个 exe，没有壳。我们的目标是使这个 winmine 变成不会爆炸的，即使我们 click 中了地雷，也可以继续进行游戏，直至把地图完成为止

要完成这个任务，或许我们想 : 平日针对注册名字和序号的破解，见多了，但是今次要分析 winmine 的流程，了解它的结构，并且作出最正确的修改。

## Part 2. OD 加载

从它的原理开始思考，假设它有一个数据结构，我们 click 在地图上，它便会检查是否中了地雷，如果中了，便立即完结了。

我们明显可以中断的地点，包括 : 按下鼠标消息, 游戏的计时消息

还有一个地方，就是绘画地雷 icon 的 function。每一场新游戏的开始，也会进行一些绘画工作。当我们中了地雷，它也需要把地雷绘画出来的。我们想，这个地方经常被呼叫。

```
01003E1A  |. 8B45 14        MOV EAX,DWORD PTR SS:[EBP+14]
01003E1D  |> 5D             POP EBP
01003E1E  \. C2 1000        RETN 10
01003E21  > $ 6A 70          PUSH 70   // <- 在这里
01003E23   . 68 90130001    PUSH winmine.01001390
01003E28   . E8 DF010000    CALL winmine.0100400C
01003E2D   . 33DB           XOR EBX,EBX
```

设断点

```
bp SetTimer
```

F9 运行，程序出现了。

现在，我们开始一个新游戏，当我们在地图点 click 第一次的时候，游戏开始计时，它便呼叫 `SetTimer`，我们将会停在那里。

开新游戏，随意按一个格子。

```
77E1C41F   90               NOP
77E1C420   90               NOP
77E1C421  > B8 0B120000      MOV EAX,120B  // <- 在这里
77E1C426   8D5424 04        LEA EDX,DWORD PTR SS:[ESP+4]
77E1C42A   CD 2E            INT 2E
77E1C42C   C2 1000          RETN 10
```

单步或 alt + F9 返回 winmine 的领空，便会到达

```
01003830  |. FF05 9C570001  INC DWORD PTR DS:[100579C]
01003836  |. E8 7AF0FFFF    CALL winmine.010028B5
0100383B  |. 6A 00          PUSH 0                            ; /Timerproc = NULL
0100383D  |. 68 E8030000    PUSH 3E8                          ; |Timeout = 1000. ms
01003842  |. 53             PUSH EBX                          ; |TimerID
01003843  |. FF35 245B0001  PUSH DWORD PTR DS:[1005B24]       ; |hWnd = 000B0132 ('Minesweeper',class='Minesweeper')
01003849  |. 891D 64510001  MOV DWORD PTR DS:[1005164],EBX    ; |
0100384F  |. FF15 B4100001  CALL DWORD PTR DS:[<&USER32.SetTimer>]   ; \SetTimer
01003855  |. 85C0           TEST EAX,EAX                      // <- 在这里
01003857  |. 75 07          JNZ SHORT winmine.01003860
01003859  |. 6A 04          PUSH 4                            ; /Arg1 = 00000004
0100385B  |. E8 F0000000    CALL winmine.01003950             ; \winmine.01003950
01003860  |> A1 18510001    MOV EAX,DWORD PTR DS:[1005118]
01003865  |. 8B0D 1C510001  MOV ECX,DWORD PTR DS:[100511C]
0100386B  |> 841D 00500001  TEST BYTE PTR DS:[1005000],BL
01003871  |. 5B             POP EBX
01003872  |. 75 10          JNZ SHORT winmine.01003884
```

这时候，我们下断绘图 api，

```
bp BitBlt
```

F9 运行，中断在 `BitBlt`

```
77F42CB7   8D5424 04        LEA EDX,DWORD PTR SS:[ESP+4]
77F42CBB   CD 2E            INT 2E
77F42CBD   C2 0400          RETN 4
77F42CC0  > 55               PUSH EBP
77F42CC1   8BEC             MOV EBP,ESP
77F42CC3   8B4D 28          MOV ECX,DWORD PTR SS:[EBP+28]
77F42CC6   8B45 28          MOV EAX,DWORD PTR SS:[EBP+28]
```

alt+F9 返回到

```
01002647  |. FF35 245B0001  PUSH DWORD PTR DS:[1005B24]       ; /hWnd = 0010013A ('Minesweeper',class='Minesweeper')
0100264D  |. FF15 2C110001  CALL DWORD PTR DS:[<&USER32.GetDC>]     ; \GetDC
01002653  |. 8B4C24 0C      MOV ECX,DWORD PTR SS:[ESP+C]
01002657  |. 68 2000CC00    PUSH 0CC0020                       ; /ROP = SRCCOPY
0100265C  |. 8BF0           MOV ESI,EAX                        ; |
0100265E  |. 8B4424 0C      MOV EAX,DWORD PTR SS:[ESP+C]       ; |
01002662  |. 8BD1           MOV EDX,ECX                        ; |
01002664  |. 6A 00          PUSH 0                              ; |YSrc = 0
01002666  |. C1E2 05        SHL EDX,5                          ; |
01002669  |. 0FBE9402 40530>MOVSX EDX,BYTE PTR DS:[EDX+EAX+1005340]  ; |
01002671  |. 6A 00          PUSH 0                              ; |XSrc = 0
01002673  |. 83E2 1F        AND EDX,1F                          ; |
01002676  |. FF3495 205A000>PUSH DWORD PTR DS:[EDX*4+1005A20]      ; |hSrcDC
0100267D  |. C1E1 04        SHL ECX,4                          ; |
01002680  |. 6A 10          PUSH 10                             ; |Height = 10 (16.)
01002682  |. 6A 10          PUSH 10                             ; |Width = 10 (16.)
01002684  |. 83C1 27        ADD ECX,27                          ; |
01002687  |. C1E0 04        SHL EAX,4                          ; |
0100268A  |. 51             PUSH ECX                            ; |YDest
0100268B  |. 83E8 04        SUB EAX,4                          ; |
0100268E  |. 50             PUSH EAX                            ; |XDest
0100268F  |. 56             PUSH ESI                            ; |hDestDC
01002690  |. FF15 5C100001  CALL DWORD PTR DS:[<&GDI32.BitBlt>]     ; \BitBlt
01002696  |. 56             PUSH ESI                            ; /hDC = 03010285
01002697  |. FF35 245B0001  PUSH DWORD PTR DS:[1005B24]       ; |hWnd = 0010013A ('Minesweeper',class='Minesweeper')
0100269D  |. FF15 28110001  CALL DWORD PTR DS:[<&USER32.ReleaseDC>]  ; \ReleaseDC
```

这里有典型的复制 bitmap 函数，我们单步到 retn 离开

```
01003008  /$ 55             PUSH EBP
01003009  |. 8BEC           MOV EBP,ESP
0100300B  |. 53             PUSH EBX
0100300C  |. 8B5D 08        MOV EBX,DWORD PTR SS:[EBP+8]
0100300F  |. 56             PUSH ESI
01003010  |. 57             PUSH EDI
01003011  |. 8B7D 0C        MOV EDI,DWORD PTR SS:[EBP+C]
01003014  |. 8BF7           MOV ESI,EDI
01003016  |. C1E6 05        SHL ESI,5
01003019  |. 03F3           ADD ESI,EBX
0100301B  |. 0FBE86 4053000>MOVSX EAX,BYTE PTR DS:[ESI+1005340]
01003022  |. A8 40          TEST AL,40
01003024  |. 75 57          JNZ SHORT winmine.0100307D
01003026  |. 83E0 1F        AND EAX,1F
01003029  |. 83F8 10        CMP EAX,10
0100302C  |. 74 4F          JE SHORT winmine.0100307D
0100302E  |. 83F8 0E        CMP EAX,0E
01003031  |. 74 4A          JE SHORT winmine.0100307D
01003033  |. FF05 A4570001  INC DWORD PTR DS:[10057A4]
01003039  |. 57             PUSH EDI
0100303A  |. 53             PUSH EBX
0100303B  |. E8 FBfeFFFF    CALL winmine.01002F3B
01003040  |. 8945 0C        MOV DWORD PTR SS:[EBP+C],EAX
01003043  |. 57             PUSH EDI
01003044  |. 0C 40          OR AL,40
01003046  |. 53             PUSH EBX
01003047  |. 8886 40530001  MOV BYTE PTR DS:[ESI+1005340],AL
0100304D  |. E8 F4F5FFFF    CALL winmine.01002646        // 绘图呼叫
01003052  |. 837D 0C 00     CMP DWORD PTR SS:[EBP+C],0   // <- 来到这里
01003056  |. 75 25          JNZ SHORT winmine.0100307D
01003058  |. A1 98570001    MOV EAX,DWORD PTR DS:[1005798]
0100305D  |. 891C85 A051000>MOV DWORD PTR DS:[EAX*4+10051A0],EBX
01003064  |. 893C85 C057000>MOV DWORD PTR DS:[EAX*4+10057C0],EDI
0100306B  |. 40             INC EAX
0100306C  |. 83F8 64        CMP EAX,64
0100306F  |. A3 98570001    MOV DWORD PTR DS:[1005798],EAX
01003074  |. 75 07          JNZ SHORT winmine.0100307D
01003076  |. 8325 98570001 >AND DWORD PTR DS:[1005798],0
0100307D  |> 5F             POP EDI
0100307E  |. 5E             POP ESI
```



# 0100307F 至 01003081 汇编分析

```
0100307F  |. 5B             POP EBX
01003080  |. 5D             POP EBP
01003081  \. C2 0800        RETN 8
```

我们想知道这个地区是不是一个子函数，所以在 OD 点选 `01003008` 这行（这部分函数的第一行指令），然后 `Ctrl+R`，搜索 `winmine` 所有对这里的呼叫。

搜索得到 9 个结果，被不同地方呼叫，很明显这里只是一个子函数。我们要了解整体流程，还要往上走。

# F8 单步离开

```
01003084  /$ 55             PUSH EBP
01003085  |. 8BEC           MOV EBP,ESP
01003087  |. 53             PUSH EBX
01003088  |. FF75 0C        PUSH DWORD PTR SS:[EBP+C]          ; /Arg2
0100308B  |. 33DB           XOR EBX,EBX                         ; |
0100308D  |. FF75 08        PUSH DWORD PTR SS:[EBP+8]          ; |Arg1
01003090  |. 43             INC EBX                             ; |
01003091  |. 891D 98570001  MOV DWORD PTR DS:[1005798],EBX     ; |
01003097  |. E8 6CFFFFFF    CALL winmine.01003008               ; \winmine.01003008
0100309C  |. 391D 98570001  CMP DWORD PTR DS:[1005798],EBX
010030A2  |. 74 70          JE SHORT winmine.01003114
010030A4  |. 56             PUSH ESI
010030A5  |. 57             PUSH EDI
010030A6  |> 8B349D C057000>MOV ESI,DWORD PTR DS:[EBX*4+10057C0]
010030AD  |. 8B3C9D A051000>|MOV EDI,DWORD PTR DS:[EBX*4+10051A0]
010030B4  |. 4E             |DEC ESI
010030B5  |. 8D47 FF        |LEA EAX,DWORD PTR DS:[EDI-1]
010030B8  |. 56             |PUSH ESI                            ; /Arg2
010030B9  |. 50             |PUSH EAX                            ; |Arg1
010030BA  |. E8 49FFFFFF    |CALL winmine.01003008              ; \winmine.01003008
010030BF  |. 56             |PUSH ESI                            ; /Arg2
010030C0  |. 57             |PUSH EDI                            ; |Arg1
010030C1  |. E8 42FFFFFF    |CALL winmine.01003008              ; \winmine.01003008
010030C6  |. 8D47 01        |LEA EAX,DWORD PTR DS:[EDI+1]
010030C9  |. 56             |PUSH ESI                            ; /Arg2
010030CA  |. 50             |PUSH EAX                            ; |Arg1
010030CB  |. 8945 0C        |MOV DWORD PTR SS:[EBP+C],EAX       ; |
010030CE  |. E8 35FFFFFF    |CALL winmine.01003008              ; \winmine.01003008
```

我们看见很多 `winmine.01003008` 的呼叫（刚才的绘图函数），我们估计，这里是 `winmine` 分析那个地图的地区，被一起打开了，需要显示空白地面出来。

那些参数 `Arg1` 和 `Arg2`，不用说，它就是 `winmine` 地图的 X-Y 坐标。

# 继续分析 0100356A 返回流程

我们继续向前走，离开这里，返回到：

```
0100356A  |. EB 44          JMP SHORT winmine.010035B0
0100356C  |> FF7424 18      PUSH DWORD PTR SS:[ESP+18]          ; /Arg2
01003570  |. C1E0 05        SHL EAX,5                           ; |
01003573  |. 8D8408 4053000>LEA EAX,DWORD PTR DS:[EAX+ECX+1005340] ; |
0100357A  |. C602 0F        MOV BYTE PTR DS:[EDX],0F            ; |
0100357D  |. 8008 80        OR BYTE PTR DS:[EAX],80             ; |
01003580  |. 56             PUSH ESI                            ; |Arg1
01003581  |. E8 FEFAFFFF    CALL winmine.01003084               ; \winmine.01003084
01003586  |. EB 28          JMP SHORT winmine.010035B0
01003588  |> 6A 4C          PUSH 4C
0100358A  |. 50             PUSH EAX
0100358B  |. 56             PUSH ESI
0100358C  |. E8 1AF9FFFF    CALL winmine.01002EAB
01003591  |. 6A 00          PUSH 0
01003593  |. EB 16          JMP SHORT winmine.010035AB
01003595  |> 50             PUSH EAX                            ; /Arg2
01003596  |. 56             PUSH ESI                            ; |Arg1
01003597  |. E8 E8FAFFFF    CALL winmine.01003084               ; \winmine.01003084
0100359C  |. A1 A4570001    MOV EAX,DWORD PTR DS:[10057A4]     //  在这里
010035A1  |. 3B05 A0570001  CMP EAX,DWORD PTR DS:[10057A0]
010035A7  |. 75 07          JNZ SHORT winmine.010035B0
010035A9  |. 6A 01          PUSH 1
010035AB  |> E8 CCFEFFFF    CALL winmine.0100347C
010035B0  |> 5F             POP EDI
010035B1  |. 5E             POP ESI
010035B2  |. 5D             POP EBP
010035B3  |. 5B             POP EBX
010035B4  \. C2 0800        RETN 8
```

这个地方，比较长一点，我们向上看看，开始位置是 `01003512`。我们 `Ctrl+R` 搜索一下，发现只有一处呼叫，很明显这里是高层的流程部分。

我们尝试把 `BitBlt` 断点清除，在 `01003512` 下断，`F9` 让程序运行。

`winmine` 正常显示了，刚才被按的地方打开了，现在我们再随意按一个格子。

# 01003512 起始位置分析

```
01003512  /$ 8B4424 08       MOV EAX,DWORD PTR SS:[ESP+8]  // 果然断在这里
01003516  |. 53             PUSH EBX
01003517  |. 55             PUSH EBP
01003518  |. 56             PUSH ESI
01003519  |. 8B7424 10      MOV ESI,DWORD PTR SS:[ESP+10]
0100351D  |. 8BC8           MOV ECX,EAX
0100351F  |. C1E1 05        SHL ECX,5
01003522  |. 8D9431 4053000>LEA EDX,DWORD PTR DS:[ECX+ESI+1005340]
01003529  |. F602 80        TEST BYTE PTR DS:[EDX],80
0100352C  |. 57             PUSH EDI
0100352D  |. 74 66          JE SHORT winmine.01003595
0100352F  |. 833D A4570001  >CMP DWORD PTR DS:[10057A4],0
01003536  |. 75 50          JNZ SHORT winmine.01003588
01003538  |. 8B2D 38530001  MOV EBP,DWORD PTR DS:[1005338]
0100353E  |. 33C0           XOR EAX,EAX
01003540  |. 40             INC EAX
01003541  |. 3BE8           CMP EBP,EAX
01003543  |. 7E 6B          JLE SHORT winmine.010035B0
```

这个地方，是 `winmine` 被玩家按下时的处理程序。这里或许包含了检查“中地雷”的跳转。我们现在 `F8` 单步分析。

# 01003512 单步跟踪详情

```
01003512  /$ 8B4424 08       MOV EAX,DWORD PTR SS:[ESP+8]  // 开始跟踪
01003516  |. 53             PUSH EBX
01003517  |. 55             PUSH EBP
01003518  |. 56             PUSH ESI
01003519  |. 8B7424 10      MOV ESI,DWORD PTR SS:[ESP+10]
0100351D  |. 8BC8           MOV ECX,EAX
0100351F  |. C1E1 05        SHL ECX,5
01003522  |. 8D9431 4053000>LEA EDX,DWORD PTR DS:[ECX+ESI+1005340]
01003529  |. F602 80        TEST BYTE PTR DS:[EDX],80
0100352C  |. 57             PUSH EDI
0100352D  |. 74 66          JE SHORT winmine.01003595      // 跳了
0100352F  |. 833D A4570001  >CMP DWORD PTR DS:[10057A4],0
01003536  |. 75 50          JNZ SHORT winmine.01003588
01003538  |. 8B2D 38530001  MOV EBP,DWORD PTR DS:[1005338]
0100353E  |. 33C0           XOR EAX,EAX
01003540  |. 40             INC EAX
01003541  |. 3BE8           CMP EBP,EAX
01003543  |. 7E 6B          JLE SHORT winmine.010035B0
01003545  |. 8B1D 34530001  MOV EBX,DWORD PTR DS:[1005334]
0100354B  |. BF 60530001    MOV EDI,winmine.01005360
01003550  |> 33C9           /XOR ECX,ECX
01003552  |. 41             |INC ECX
01003553  |. 3BD9           |CMP EBX,ECX
01003555  |. 7E 0B          |JLE SHORT winmine.01003562
01003557  |> F6040F 80      |/TEST BYTE PTR DS:[EDI+ECX],80
0100355B  |. 74 0F          ||JE SHORT winmine.0100356C
0100355D  |. 41             ||INC ECX
0100355E  |. 3BCB           ||CMP ECX,EBX
01003560  |.⁷C F5           |\JL SHORT winmine.01003557
01003562  |> 40             |INC EAX
01003563  |. 83C7 20        |ADD EDI,20
01003566  |. 3BC5           |CMP EAX,EBP
01003568  |.⁷C E6           \JL SHORT winmine.01003550
0100356A  |. EB 44          JMP SHORT winmine.010035B0
0100356C  |> FF7424 18      PUSH DWORD PTR SS:[ESP+18]          ; /Arg2
01003570  |. C1E0 05        SHL EAX,5                           ; |
01003573  |. 8D8408 4053000>LEA EAX,DWORD PTR DS:[EAX+ECX+1005340] ; |
0100357A  |. C602 0F        MOV BYTE PTR DS:[EDX],0F            ; |
0100357D  |. 8008 80        OR BYTE PTR DS:[EAX],80             ; |
01003580  |. 56             PUSH ESI                            ; |Arg1
01003581  |. E8 FEFAFFFF    CALL winmine.01003084               ; \winmine.01003084
```



# 雷区分析：踏中地雷的处理逻辑

```
01003586   |. EB 28            JMP SHORT winmine.010035B0  
01003588   |> 6A 4C           PUSH 4C  
0100358A   |. 50             PUSH EAX  
0100358B   |. 56             PUSH ESI  
0100358C   |. E8 1AF9FFFF     CALL winmine.01002EAB  
01003591   |. 6A 00          PUSH 0  
01003593   |. EB 16          JMP SHORT winmine.010035AB  
01003595   |> 50             PUSH EAX          ; /Arg2    //  到达了这里  
01003596   |. 56             PUSH ESI          ; |Arg1  
01003597   |. E8 E8FAFFFF    CALL winmine.01003084      ; \\winmine.01003084 // 绘图  
0100359C   |. A1 A4570001    MOV EAX,DWORD PTR DS:[10057A4]  
010035A1   |. 3B05 A0570001  CMP EAX,DWORD PTR DS:[10057A0]  
010035A7   |. 75 07          JNZ SHORT winmine.010035B0   // 跳过  
010035A9   |. 6A 01          PUSH 1  
010035AB   |> E8 CCFEFFFF    CALL winmine.0100347C  
010035B0   |> 5F             POP EDI          // 到这里  
010035B1   |. 5E             POP ESI  
010035B2   |. 5D             POP EBP  
010035B3   |. 5B             POP EBX  
010035B4   \. C2 0800        RETN 8  
```

从上面看，有两个可疑的跳转：

第一个是 `0100352D`，下面包含了很多代码，我们估计它是踏中地雷时的处理部分。

第二个是 `010035A7`，这个暂时不清楚，或许是一些后期工作。如果不跳，只呼叫一个地方 `CALL winmine.0100347C`，我们现在先从这个入手，试试修改这个简单的跳转，看看 `CALL winmine.0100347C` 有什么特别。

按 `F9` 运行，再任意按一个格子，断在 `01003512`。

单步到 `010035A7`，把 `OD` 中 `ZF` 旗标修改为 `1`，使它不跳。

按 `F9` 运行。

奇怪的事发生了！`winmine` 显示输入玩家名称，因为 `winmine` 误认我们完成游戏了，时间是破纪录的快。

这个情况，表示了如果我们成功把游戏的地图解开，最后便会来到 `010035A7`，呼叫 `CALL winmine.0100347C` 完成游戏。

我们想，这个是成功的呼叫，那么就是说，如果我们踏中了地雷，并且它跳过了这里，直接离开此区的话，我们便要阻止那一个跳转。

我们重新开始游戏，只在 `01003512` 设断点，一直按下 `winmine` 格子，直至我们踏中了地雷。

```
01003512  /$ 8B4424 08       MOV EAX,DWORD PTR SS:[ESP+8]  
01003516  |. 53             PUSH EBX  
01003517  |. 55             PUSH EBP  
01003518  |. 56             PUSH ESI  
01003519  |. 8B7424 10      MOV ESI,DWORD PTR SS:[ESP+10]  
0100351D  |. 8BC8           MOV ECX,EAX  
0100351F  |. C1E1 05        SHL ECX,5  
01003522  |. 8D9431 4053000>LEA EDX,DWORD PTR DS:[ECX+ESI+1005340]  // 分析后，发现这里是 winmine 地图的数据结构所在  
01003529  |. F602 80        TEST BYTE PTR DS:[EDX],80  
0100352C  |. 57             PUSH EDI  
0100352D  |. 74 66          JE SHORT winmine.01003595        // 检查了是否踏中地雷  
0100352F  |. 833D A4570001 >CMP DWORD PTR DS:[10057A4],0      // 踏中了，来到这里  
01003536  |. 75 50          JNZ SHORT winmine.01003588  
```

我们看了一下 `LEA EDX,DWORD PTR DS:[ECX+ESI+1005340]`，在 `dump` 中看到一段有规律的资料，很明显这些就是 `winmine` 地图的真正资料。

我们踏中了地雷，`0100352D` 不跳，到达下面。我们开始 `F8` 跟踪，看看有什么发现。

```
0100352F  |. 833D A4570001 >CMP DWORD PTR DS:[10057A4],0  
01003536  |. 75 50          JNZ SHORT winmine.01003588      // <-  跳了  
01003538  |. 8B2D 38530001  MOV EBP,DWORD PTR DS:[1005338]  
0100353E  |. 33C0           XOR EAX,EAX  

...

01003586  |. EB 28          JMP SHORT winmine.010035B0  
01003588  |> 6A 4C          PUSH 4C                          // <- 到了这里  
0100358A  |. 50             PUSH EAX  
0100358B  |. 56             PUSH ESI  
0100358C  |. E8 1AF9FFFF    CALL winmine.01002EAB  
01003591  |. 6A 00          PUSH 0  
01003593  |. EB 16          JMP SHORT winmine.010035AB        // 强行跳了  
01003595  |> 50             PUSH EAX          ; /Arg2  
01003596  |. 56             PUSH ESI          ; |Arg1  
01003597  |. E8 E8FAFFFF    CALL winmine.01003084            ; \\winmine.01003084  
0100359C  |. A1 A4570001    MOV EAX,DWORD PTR DS:[10057A4]  
010035A1  |. 3B05 A0570001  CMP EAX,DWORD PTR DS:[10057A0]  
010035A7  |. 75 07          JNZ SHORT winmine.010035B0  
010035A9  |. 6A 01          PUSH 1  
010035AB  |> E8 CCFEFFFF    CALL winmine.0100347C            //  来到这里，很值得注意，因为上一次成功游戏，也是到这里  
010035B0  |> 5F             POP EDI  
010035B1  |. 5E             POP ESI  
010035B2  |. 5D             POP EBP  
010035B3  |. 5B             POP EBX  
010035B4  \. C2 0800        RETN 8  
```

很奇怪，最后我们再次到达 `010035AB` 的 `CALL winmine.0100347C`，我们上一次成功了游戏，不是也到了这里吗？看清楚一点，今次的参数，是 `push` 了 `0`，上一次是 `push` `1`。

意思是，`CALL winmine.0100347C` 这个呼叫，如果传入了 `1`，代表游戏成功，如果是 `0` 代表你踏中地雷。

到了现在的阶段，我们可以任意修改它了。按照我们当初的目标，是使它在踏中了地雷时不会爆炸。

我们知道，一般的点按（不是完成游戏，也没有踏中地雷），是会跳过 `010035AB`，直接来到 `010035B0` 的。我们现在只需要把踏中地雷的跳转修改，指向这里便行了。

```
01003586  |. EB 28          JMP SHORT winmine.010035B0  
01003588  |> 6A 4C          PUSH 4C                          // <- 到了这里  
0100358A  |. 50             PUSH EAX  
0100358B  |. 56             PUSH ESI  
0100358C  |. E8 1AF9FFFF    CALL winmine.01002EAB  
01003591  |. 6A 00          PUSH 0                            // <- 改这里  
01003593  |. EB 16          JMP SHORT winmine.010035AB        // 强行跳了    <-- 改这里  
```

我们把 `01003591` `nop` 了，再把 `01003593` 的 `jmp` 改成 `JMP 010035B0`。

清除其它断点，重新开始游戏，`F9` 运行。

现在任意打开格子。

果然，即使踏中了地雷，它是显示了，但不会爆炸 `:-)`

## Part 3. 总结

近年软件安全的发展，走到了一个很复杂的地步，双方都花了大量 "气力"，不论破解或保护，都很费精神，花的体力巨大。我们对于一些简单的破解所带来的趣味，是很怀念的。这篇文章希望给新手们一次比较有趣的学习，只懂得使用简单 `OD`，不懂脱壳或算法的兄弟，也可以参与。

2007 年 5 月 9 日

转载请保持内容完整

`riijj` 放了一个 DIY 的，我也放个 DIY 的，这篇文章是 `NE365` 电子书里的，外面好像没见过。

对 `winxp sp1` 下的扫雷的 `pediy`，伪造鼠标消息，自动扫雷的实现。

【破解作者】`海风月影[NE365][DFCG]`

【作者邮箱】`zjhangtian@sohu.com`

【使用工具】`od 1.10d`, `Peid`

【破解平台】`WinXP(win9x 下不能用)`

【软件名称】`winmine.exe`

【软件简介】对 `winxp sp1` 下的扫雷的 `pediy`，伪造消息，自动扫雷的实现

【破解声明】我是一只小菜鸟，偶得一点心得，愿与大家分享 :)

--------------------------------------------------------------------------------

【破解内容】

### 一、分析程序

以 `winxp sp1` 下的扫雷为例（`windows` 版本不同，扫雷不太一样的），用 `OllyDbg` 载入 `winmine.exe`（`windows` 的程序一般是不加壳的）。第一步我们要找到相关处理的代码，即单击鼠标后判断是否有雷，有雷就炸，没有雷就翻开。找代码各有各的高招，我说说我的笨办法，但一定能找到。

扫雷是用 `visual c++` 写的，`visual c++` 写的程序有个 `WinMain` 函数，函数原型如下：

```
int WINAPI WinMain (HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nCmdShow)
```

在 `od` 里的代码为如下单步跟踪不久就会出现如下代码：

```
01003F89   > \50             push eax                        ; /Arg4  
01003F8A   .  56            push esi                        ; |Arg3  
01003F8B   .  53            push ebx                        ; |Arg2  
01003F8C   .  53            push ebx                        ; |/0   
01003F8D   .  FFD7         call edi                        ; |\GetModuleHandleA  
01003F8F   .  50            push eax                        ; |Arg1  
01003F90   .  E8 5BE2FFFF   call winmine.010021F0           ; \\winmine.010021F0  
```

从 `01003F90` 跟进，运行到 `RegisterClass` 这里：

```
0100228B  |.  50            push eax                        ; /pWndClass  
0100228C  |.  897D D4       mov [local.11],edi             ; |  
0100228F  |.  8975 D8       mov [local.10],esi             ; |  
01002292  |.  FF15 CC100001  call dword ptr ds:[<&USER32.Reg>; \\RegisterClassW  
```

这里是注册窗口类，熟悉 `c++` 编程的都知道，参数 `pWndClass` 指向窗口类信息，其中第二项是 `lpfnWndProc` 这是我们所需要的，是窗口回调函数的地址，是处理消息用的，我们看堆栈：

```
0006FEC0    0006FED0    \pWndClass = 0006FED0  
0006FEC4    77E5AD86    kernel32.GetModuleHandleA  
0006FEC8    00091F01  
0006FECC    00000000  
```

地址是 `0006FED0`，因此 `[0006FED4]` 里就是窗口回调函数的地址了，内存如下：

```
0006FED0  00 00 00 00 C9 1B 00 01 00 00 00 00 00 00 00 00   ....?.........
```



```
0006FEE0  00 00 00 01 B6 02 A3 00 11 00 01 00 14 00 90 01  ...??...?
```

因此，对 `1001BC9` 下断点，运行，立刻就断下来了，代码如下

```
01001BC9   .  55                   push ebp
01001BCA   .  8BEC                 mov ebp,esp
01001BCC   .  83EC 40              sub esp,40
01001BCF   .  8B55 0C              mov edx,dword ptr ss:[ebp+C]
```

> `edx` 为消息参数

```
01001BD2   .  8B4D 14              mov ecx,dword ptr ss:[ebp+14]
```

> `ecx` 为其它参数，当 `edx` 为鼠标方面的消息参数时，`ecx` 为坐标

```
01001BD5   .  53                   push ebx
01001BD6   .  56                   push esi
01001BD7   .  33DB                 xor ebx,ebx
01001BD9   .  57                   push edi
01001BDA   .  BE 00020000          mov esi,200
01001BDF   .  43                   inc ebx
01001BE0   .  33FF                 xor edi,edi
01001BE2   .  3BD6                 cmp edx,esi
```

窗口回调函数的原型是 `LRESULT WINAPI WndProc (HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam)`

单步到 `01001BCC` 此时看堆栈，关联到 `ebp`

```
EBP ==>  >/0006F7F0 原来的 ebp
EBP+4     >|77D13A50     返回到 USER32.77D13A50
EBP+8     >|002502D6 HWND hWnd
EBP+C     >|00000024 UINT msg
EBP+10    >|00000000 WPARAM wParam
EBP+14    >|0006F8F0 LPARAM lParam
```

根据窗口回调函数的原型很清楚能看到，`[ebp+C]`里是 `msg`，也就是消息参数，传给了 `edx`,下面开始对不同的 `msg` 分别做不同的处理，在 `window.h` 头文件中定义了

*   `201` 为 `WM_LBUTTONDOWN`
*   `202` 为 `WM_LBUTTONUP`
*   `204` 为 `WM_RBUTTONDOWN`
*   `205` 为 `WM_RBUTTONUP`
*   `207` 为 `WM_MBUTTONDOWN`
*   `208` 为 `WM_MBUTTONUP`

可以对 `01001BD2` 下条件断点，命令为 `bp 1001bd2 ,edx==201`，然后运行，出现了扫雷窗口，点击左键，促发断点，一步步跟踪到

```
01001FA6   > \393D 48510001    cmp dword ptr ds:[1005148],edi      ;  Case 201 (WM_LBUTTONDOWN) of switch 01001F5F
01001FAC   .^ 75 D6            jnz short winmine.01001F84
01001FAE   .  FF75 14         push dword ptr ss:[ebp+14]           ; /Arg1
01001FB1   .  E8 56F4FFFF     call winmine.0100140C               ; \winmine.0100140C
01001FB6   .  85C0           test eax,eax
01001FB8   .^ 0F85 A0FCFFFF   jnz winmine.01001C5E
01001FBE   .  841D 00500001   test byte ptr ds:[1005000],bl
01001FC4   .  0F84 DF010000   je winmine.010021A9
01001FCA   .  8B45 10          mov eax,dword ptr ss:[ebp+10]
01001FCD   .  24 06           and al,6
01001FCF   .  F6D8            neg al
01001FD1   .  1BC0            sbb eax,eax
01001FD3   .  F7D8            neg eax
01001FD5   .  A3 44510001     mov dword ptr ds:[1005144],eax
01001FDA   .  E9 80000000     jmp winmine.0100205F                ;  跳到到下一段
```

。。。

```
0100205F   > \FF75 08         push dword ptr ss:[ebp+8]             ; /hWnd
01002062   .  FF15 E4100001   call dword ptr ds:[<&USER32.SetCapture>]  ; \SetCapture
01002068   .  830D 18510001 FF or dword ptr ds:[1005118],FFFFFFFF
0100206F   .  830D 1C510001 FF or dword ptr ds:[100511C],FFFFFFFF
01002076   .  53             push ebx
01002077   .  891D 40510001   mov dword ptr ds:[1005140],ebx
0100207D   .  E8 91080000    call winmine.01002913
01002082   .  8B4D 14         mov ecx,dword ptr ss:[ebp+14]
01002085   >  393D 40510001   cmp dword ptr ds:[1005140],edi
0100208B   .  74 34           je short winmine.010020C1
0100208D   .  841D 00500001   test byte ptr ds:[1005000],bl
01002093   .^ 0F84 54FFFFFF   je winmine.01001FED
01002099   .  8B45 14         mov eax,dword ptr ss:[ebp+14]         ;  这里开始处理坐标，
```

> 下面几行代码我们后面处理还要用到

```
0100209C   .  C1E8 10          shr eax,10
0100209F   .  83E8 27          sub eax,27
010020A2   .  C1F8 04          sar eax,4
010020A5   .  50             push eax                            ; /第几行
010020A6   .  0FB745 14       movzx eax,word ptr ss:[ebp+14]      ; |
010020AA   .  83C0 04         add eax,4                           ; |
010020AD   .  C1F8 04         sar eax,4                           ; |
010020B0   .  50             push eax                            ; |第几列
010020B1   >  E8 1E110000    call winmine.010031D4               ; \处理过程
```

跟进上面的处理过程

```
010031D4  /$  55             push ebp
010031D5  |.  8BEC           mov ebp,esp
010031D7  |.  83EC 20        sub esp,20
010031DA  |.  8B55 08        mov edx,[arg.1]                     ;  edx 为第几列
010031DD  |.  A1 18510001    mov eax,dword ptr ds:[1005118]
010031E2  |.  3BD0           cmp edx,eax
010031E4  |.  8B0D 1C510001  mov ecx,dword ptr ds:[100511C]
010031EA  |.  57             push edi
010031EB  |.  8B7D 0C        mov edi,[arg.2]                     ;  edi 为第几行
```

中间大段代码省略。。。

```
01003397  |> \85DB           test ebx,ebx                          ;  跟到这里
01003399  |.  7E 34           jle short winmine.010033CF            ;  如果 ebx 和 esi 不是参数则判断 edx 和 edi 是不是参数
0100339B  |.  85F6           test esi,esi
0100339D  |.  7E 30          jle short winmine.010033CF
0100339F  |.  3B1D 34530001  cmp ebx,dword ptr ds:[1005334]        ;  行数是否越界
010033A5  |.  7F 28          jg short winmine.010033CF
010033A7  |.  3B35 38530001  cmp esi,dword ptr ds:[1005338]        ;  列数是否越界
010033AD  |.  7F 20          jg short winmine.010033CF
010033AF  |.  8BC6           mov eax,esi
010033B1  |.  C1E0 05        shl eax,5                             ;  都没有就判断是否点击过
010033B4  |.  F68418 40530001 >test byte ptr ds:[eax+ebx+1005340],40   ;  以 1005340 为基址的二维数组
010033BC  |.  75 11          jnz short winmine.010033CF
010033BE  |.  56             push esi
010033BF  |.  53             push ebx
010033C0  |.  E8 DBFDFFFF    call winmine.010031A0                ;  把没有点击的变成点击的
010033C5  |.  56             push esi
010033C6  |.  53             push ebx
010033C7  |.  E8 7AF2FFFF    call winmine.01002646
010033CC  |.  8B55 08        mov edx,[arg.1]
010033CF  |>  85D2           test edx,edx                          ;  第几列
010033D1  |.  7E 42          jle short winmine.01003415
010033D3  |.  85FF           test edi,edi                          ;  第几行
010033D5  |.  7E 3E          jle short winmine.01003415
010033D7  |.  3B15 34530001  cmp edx,dword ptr ds:[1005334]        ;  行数是否越界
010033DD  |.  7F 36          jg short winmine.01003415
010033DF  |.  3B3D 38530001  cmp edi,dword ptr ds:[1005338]        ;  列数是否越界
010033E5  |.  7F 2E          jg short winmine.01003415
010033E7  |.  C1E7 05        shl edi,5                             ;  都没有就判断是否点击过
010033EA  |.  8A8417 40530001 mov al,byte ptr ds:[edi+edx+1005340]    ;  以 1005340 为基址的二维数组
```



# 010033F1 代码片段分析

```
010033F1  |.  A8 40             test al,40                      ; 是否点击过？
010033F3  |.  75 20             jnz short winmine.01003415      ; 不是就出去了
010033F5  |.  24 1F             and al,1F
010033F7  |.  3C 0E             cmp al,0E
010033F9  |.  74 1A             je short winmine.01003415
010033FB  |.  8B3D 1C510001     mov edi,dword ptr ds:[100511C]
01003401  |.  8B35 18510001     mov esi,dword ptr ds:[1005118]
01003407  |.  57                push edi
01003408  |.  56                push esi
01003409  |.  E8 5DFDFFFF       call winmine.0100316B           ; 把没有点击的变成点击的
0100340E  |.  57                push edi
0100340F  |.  56                push esi
01003410  |.  E8 31F2FFFF       call winmine.01002646           ; 显示出来
01003415  |>  5E                pop esi                           ; 这里就出去了
```

这是左键单击按下的处理过程。再分析一下左键弹起、右键单击的过程，多试验几次会知道大概规则。以 `1005340` 为起始地址有个二维数组表，里面是是否有雷的信息，以一个字节为一格，规则如下：

*   `xF` 表示还没点击
*   `xE`、`xD` 表示标上了雷的标志和打了问号
*   `x0` 表示点击过了
*   `4x` 表示点击过了，而且没有雷，`4x` 中的 `x` 表示周围几个雷
*   `8x` 表示有雷
*   `0x` 表示还没有点击过，且没有雷

大概的规则了解了，下面就可以做些手脚了 :-)\

## 二、如何作弊

分析好了，下面看看如何作弊。当单击到雷的时候，就爆了，如果想不死，那就在单击的时候，先判断一下是否有雷，没有雷就单击一下，有雷就不单击。修改单击的处理过程不太现实，有两处，要改就要一起改。所以想到在窗口回调函数一开始，先让它判断一下单击的位置是否有雷，有雷就不单击（改为右击）。这样左键点击就永远死不了了，但是一个一个点也太麻烦了，也太累了，这还不够，如果让电脑自动帮我们点多好啊，一个一个顺着点下去，这样就立即扫完雷了。说做就做，开始动手。

## 三、分析文件

这步很重要，你要在文件里添加代码，就必须先确定添加在哪，文件剩余空间够不够等等。用 `nbw` 友情提供的 `pe` 剩余空间查看器分析一下：

| 名称 | RVA | OA | 尺寸 D | 可写否 |
| :--- | :--- | :--- | :--- | :--- |
| `.text` | 00004a56 | 00003e56 | 426 | 否 | 选这个节 |
| `.data` | 00005b98 | 00004b98 | -2456 | 可 |
| `.rsrc` | 0001f160 | 0001d360 | 160 | 否 |

有效剩余空间(字节 D)为: `586`

我们要写的代码也就 100 多字节吧，所以我们就选 `.text` `00004a56` `00003e56` `426` 否 这个节。要注意，不能写入，没关系，用 `LoadPE` 改一下这个节的属性就行了，加个可写属性就行了。

实在不放心就添加一个节，大小是 `1000`，名字随便起，不过我喜欢完美，尽量不增加文件长度（想起了经典的 `cih` 病毒 ^\_^），`nbw` 的工具就可以增加节，要点上加载这个节，还可以用 `topo` 等工具，这里就不介绍了。但要注意一点，这个是 `windows` 的程序，就这样加节是会出错的，因为里面有 `BoundImport` 数据，用 `loadpe` 去除这些数据就可以加了。

## 四、编写作弊代码

添加作弊代码的方法很多，`nbw` 是先写汇编代码然后编译，再写进去，这个方法我不太会，所以不用。也可以写个 `dll` 让 `winmine` 调用，这个方法比较烦，这样做还不如写个内存作弊器，直接把雷找出来。我的做法是直接在 `ollydbg` 里写代码，虽然比较累，但是可以随时调试。（应该这么说，其他方法我都不会，只会这种，丢脸了。。。）

通过上面对 `WndProc` 回调函数的分析我们可以在下面这个点修改：

```
01001BC9   .  55               push ebp
01001BCA   .  8BEC             mov ebp,esp
01001BCC   .  83EC 40          sub esp,40
01001BCF   .  8B55 0C          mov edx,dword ptr ss:[ebp+C]
01001BD2  > .  8B4D 14         mov ecx,dword ptr ss:[ebp+14]
01001BD5   .  53               push ebx
01001BD6   .  56               push esi
01001BD7   .  33DB             xor ebx,ebx
01001BD9   .  57               push edi

★把代码插在这两句中间

01001BDA   .  BE 00020000      mov esi,200
01001BDF   .  43               inc ebx
01001BE0   .  33FF             xor edi,edi
01001BE2   .  3BD6             cmp edx,esi
```

为什么选择这里呢？因为这里刚保存了 `ebx`、`esi`、`edi` 三个寄存器，还没有开始用这时我们来用，就免去了保护寄存器的工作，这样我们就有 4 个寄存器可以使用了（还有一个是 `eax`，后面代码可以发现 `eax` 在这里的值没有作用）。从 `1001BD5` 开始 5 个字节改成 `jmp xxxxxxxx`，其中 `xxxxxxxx` 就是我们写的代码的地址。我这里是跳到了这个节的尾部 `104A56`，在 `104A56` 开始写代码。

```
01004A56   > \8B55 0C             mov edx,dword ptr ss:[ebp+C]
01004A59   .  8B4D 14             mov ecx,dword ptr ss:[ebp+14]        ; 这两行重写一遍也无所谓，跳到下一行也没关系
01004A5C   .  53                  push ebx
01004A5D   .  56                  push esi
01004A5E   .  57                  push edi                               ; 这三行保存寄存器
01004A5F   .  EB 07               jmp short winmine.01004A68            ; 跳到我们的作弊代码
01004A61   >  33DB               xor ebx,ebx                           ; 这里是出口，跳回去，清空 ebx（原来的代码）
01004A63   .^ E9 72D1FFFF        jmp winmine.01001BDA                   ; 跳回去
```

这几行写的是基本的一些代码，保存寄存器，恢复寄存器的值。为什么把出口写在这而不写在代码最后，是因为这样有个明确的出口，否则出口不明确代码不好编写（直接写代码的坏处，用汇编写然后编译就没有这个问题），下面开始判断。

```
01004A68   > \81FA 01020000      cmp edx,201                           ; 左键按下？
01004A6E   .  74 12               je short winmine.01004A82           ; 是就继续
01004A70   .  81FA 02020000      cmp edx,202                           ; 左键弹起？
01004A76   .^ 75 E9               jnz short winmine.01004A61          ; 也不是就跳回去，不是我们要处理的消息
01004A78   .  33FF               xor edi,edi
01004A7A   .  393D 40510001      cmp dword ptr ds:[1005140],edi
01004A80   .^ 74 DF               je short winmine.01004A61           

★这三句是判断左键弹起的，程序里是这样判断的，保险起见，我们也这样判断
```

```
01004A82   >  8B45 14             mov eax,dword ptr ss:[ebp+14]        ; 这里开始是抄写源程序分解坐标的方法
01004A85   .  C1E8 10             shr eax,10
01004A88   .  83E8 27             sub eax,27
01004A8B   .  C1F8 04             sar eax,4                             ; 用了栈传递参数
01004A8E   .  50                 push eax                                ; 行
01004A8F   .  0FB745 14          movzx eax,word ptr ss:[ebp+14]
01004A93   .  83C0 04             add eax,4
01004A96   .  C1F8 04             sar eax,4
01004A99   .  50                 push eax                                ; 列
01004A9A   .  5E                 pop esi                                 ; 取出列数
01004A9B   .  5B                 pop ebx                                 ; 取出行数
01004A9C   .  85DB               test ebx,ebx
01004A9E   .^ 7E C1               jle short winmine.01004A61
01004AA0   .  85F6               test esi,esi
01004AA2   .^ 7E BD               jle short winmine.01004A61
01004AA4   .  3B1D 38530001      cmp ebx,dword ptr ds:[1005338]
01004AAA   .^ 7F B5               jg short winmine.01004A61
01004AAC   .  3B35 34530001      cmp esi,dword ptr ds:[1005334]
01004AB2   .^ 7F AD               jg short winmine.01004A61
01004AB4     C1E3 05              shl ebx,5
01004AB7   >  8A8433 40530001    mov al,byte ptr ds:[ebx+esi+1005340]  ; 到这里，判断方法和程序是一样的
01004ABE   .  3C 80               cmp al,80                               ; 判断是否有雷

★不管雷是什么状态，前面是 8 就有雷
```

```
01004AC0   .^ 72 9F               jb short winmine.01004A61           ; 小于即没有雷

★这里不管是什么状态，只要没有雷就行
```

```
01004AC2   .  BA 04020000        mov edx,204                           ; 有雷的情况把消息改成右击
01004AC7   .  C745 10 02000000   mov dword ptr ss:[ebp+10],2           ; 判断右击的标志

★（右击时 wParam＝2，程序里是要判断的，所以要改一下标志）
```

```
01004ACE   .^ EB 91               jmp short winmine.01004A61          ; 跳回去
```

这样修改的程序就永远死不了了，左键单击太麻烦了，我们还要继续改一下，让电脑自动帮我们点击，因此，在这行修改跳转：

```
01004AB4     C1E3 05              shl ebx,5
```

改成跳转：

```
01004AB4     EB 2D                jmp short winmine.01004AE3
01004AB6     90                   nop
```

先把我写的代码贴上，然后解释。



## 汇编代码分析与逆向逻辑

以下是反汇编代码片段，包含数据定义与控制流逻辑：

```assembly
01004AD0   . 00                     db 00                              ; 这里是是否为我们伪造消息的状态，1 为是，0 为否
01004AD1     00                     db 00
01004AD2   . 00000000               dd 00000000
01004AD6   . 00000000               dd 00000000                       ; 当前的列数
01004ADA   . 00000000               dd 00000000                       ; 当前的行数
01004ADE   > C1E3 05                shl ebx,5
01004AE1   .^ EB D4                jmp short winmine.01004AB7
01004AE3   > 803D D04A0001 00      cmp byte ptr ds:[1004AD0],0        ; 跳到这里，先判断是否为我们伪造消息状态
01004AEA   .^ 75 F2                jnz short winmine.01004ADE          ; 是就跳走，不处理下面的
01004AEC   . C605 D04A0001 01      mov byte ptr ds:[1004AD0],1        ; 标志为伪造消息状态
01004AF3   . 3E:8B5D 08            mov ebx,dword ptr ds:[ebp+8]
01004AF7   . 891D D24A0001         mov dword ptr ds:[1004AD2],ebx
01004AFD   >/ 8B35 D64A0001        mov esi,dword ptr ds:[1004AD6]      ; 取出列数
01004B03   .| 46                    inc esi                              ; ＋1
01004B04   .| 8935 D64A0001         mov dword ptr ds:[1004AD6],esi      ; 写回去
01004B0A   .| 3B35 34530001         cmp esi,dword ptr ds:[1005334]      ; 比较是否出界
01004B10   .| 7F 68                 jg short winmine.01004B7A           ; 出界代表结束了，跳走
01004B12   .| C1E6 04               shl esi,4                           ; 计算坐标的逆运算
01004B15   .| 83EE 03               sub esi,3
01004B18   >|/8B1D DA4A0001         mov ebx,dword ptr ds:[1004ADA]      ; 取出行数
01004B1E   .||43                   inc ebx                              ; ＋1
01004B1F   .||891D DA4A0001         mov dword ptr ds:[1004ADA],ebx      ; 写回去
01004B25   .||3B1D 38530001        cmp ebx,dword ptr ds:[1005338]      ; 比较是否出界
01004B2B   .||7F 41                jg short winmine.01004B6E           ; 出界就跳走
01004B2D   .||33FF                xor edi,edi                           ; edi 为参数 lParam
01004B2F   .||03FB                add edi,ebx                           ; 坐标参数的逆运算
01004B31   .||C1E7 04             shl edi,4
01004B34   .||83C7 28             add edi,28
01004B37   .||C1E7 10             shl edi,10
01004B3A   .||03FE                add edi,esi
01004B3C   .||57                  push edi                              ; /lParam
01004B3D   .||6A 01               push 1                                ; |wParam
01004B3F   .||68 01020000         push 201                              ; |msg=左键按下
01004B44   .||FF35 D24A0001       push dword ptr ds:[1004AD2]           ; |hWnd
01004B4A   .||E8 7AD0FFFF         call winmine.01001BC9                 ; \\call WndProc
01004B4F   .||C705 40510001 01000000 mov dword ptr ds:[1005140],1       ; 左键弹起标志
01004B59   .||57                  push edi                              ; /lParam
01004B5A   .||6A 00               push 0                                ; |wParam
01004B5C   .||68 02020000         push 202                              ; |msg=左键弹起
01004B61   .||FF35 D24A0001       push dword ptr ds:[1004AD2]           ; |hWnd
01004B67   .||E8 5DD0FFFF         call winmine.01001BC9                 ; \\call WndProc
01004B6C   .|\\EB AA             jmp short winmine.01004B18            ; 行数小循环
01004B6E   >| C705 DA4A0001 00000000 mov dword ptr ds:[1004ADA],0       ; 行数从 0 开始
01004B78   .\\ EB 83             jmp short winmine.01004AFD            ; 列数大循环
01004B7A   > C705 D64A0001 00000000 mov dword ptr ds:[1004AD6],0        ; 列数从 0 开始
01004B84   . C605 D04A0001 00    mov byte ptr ds:[1004AD0],0           ; 恢复伪造消息标志
01004B8B   .^ E9 D1FEFFFF        jmp winmine.01004A61                  ; 出门咯^\_^
```

### 伪造鼠标单击消息的逻辑

这里为了让计算机帮我们单击，我们伪造了鼠标单击的消息。

```assembly
01004B3C   .||57                  push edi                              ; /lParam
01004B3D   .||6A 01               push 1                                ; |wParam
01004B3F   .||68 01020000         push 201                              ; |msg=左键按下
01004B44   .||FF35 D24A0001       push dword ptr ds:[1004AD2]           ; |hWnd
01004B4A   .||E8 7AD0FFFF         call winmine.01001BC9                 ; \\call WndProc
01004B4F   .||C705 40510001 01000000 mov dword ptr ds:[1005140],1       ; 左键弹起标志
01004B59   .||57                  push edi                              ; /lParam
01004B5A   .||6A 00               push 0                                ; |wParam
01004B5C   .||68 02020000         push 202                              ; |msg=左键弹起
01004B61   .||FF35 D24A0001       push dword ptr ds:[1004AD2]           ; |hWnd
01004B67   .||E8 5DD0FFFF         call winmine.01001BC9                 ; \\call WndProc
```

然后用了两个循环，分别遍历行数和列数，做到每个格子都让鼠标点一遍。但是，我们的这个处理是在消息过程里分出来的，所以要有个标志，就是消息是否是我们发出的。如果是的话就不执行上面这段伪造代码，所以在开始有个标志位：

```assembly
01004AE3   > 803D D04A0001 00      cmp byte ptr ds:[1004AD0],0        ; 跳到这里，先判断是否为我们伪造消息状态
01004AEA   .^ 75 F2                jnz short winmine.01004ADE          ; 是就跳走，不处理下面的伪造消息
```

### 坐标的逆运算

还有一个就是坐标的逆运算。原来的运算是：

```assembly
01004A82   > 8B45 14               mov eax,dword ptr ss:[ebp+14]       ; 这里开始是抄写源程序分解坐标的方法
01004A85   . C1E8 10               shr eax,10                          ; 取高位
01004A88   . 83E8 27               sub eax,27                          ; －39（边界）
01004A8B   . C1F8 04               sar eax,4                           ; ÷16
01004A8E   . 50                    push eax                            ; 结果为行
01004A8F   . 0FB745 14             movzx eax,word ptr ss:[ebp+14]    ; 低位
01004A93   . 83C0 04               add eax,4                           ; ＋4
01004A96   . C1F8 04               sar eax,4                           ; ÷16
01004A99   . 50                    push eax                            ; 列
```

高位－39，再除以 16 结果为行；低位＋4 再除以 16，结果为列。所以我们可以逆运算一下：

- 行数×16，再加上 40（为的是将鼠标点在格子里，所以要小于 39，小一点就可以了，否则点不到的）
- 列数×16，再减取 3（道理和上面一样）

然后再组装起来，行数放在高位（前两个字节），列数放在低位（后两个字节）：

```text
00 00   00 00
行数     列数
```

最后保存一下就可以了。在代码上点右键，复制到可执行文件－》全部修正，就行了。

### 整理添加的代码

整理一下自己添上去的代码：

```assembly
01004A56   > \\8B55 0C             mov edx,dword ptr ss:[ebp+C]
01004A59   . 8B4D 14               mov ecx,dword ptr ss:[ebp+14]       ; 这两行重写一遍也无所谓，跳到下一行也没关系
01004A5C   . 53                    push ebx
01004A5D   . 56                    push esi
01004A5E   . 57                    push edi                              ; 这三行保存寄存器
01004A5F   . EB 07                 jmp short winmine.01004A68            ; 跳到我们的作弊代码
01004A61   > 33DB                  xor ebx,ebx                           ; 这里是出口，跳回去，清空 ebx（原来的代码）
01004A63   .^ E9 72D1FFFF          jmp winmine.01001BDA                  ; 跳回去
01004A68   > 81FA 01020000         cmp edx,201                             ; 左键按下？
```



```
01004A6E   .  74 12                je short winmine.01004A82          ; 是就继续

01004A70   .  81FA 02020000        cmp edx,202                     ; 左键弹起？

01004A76   .^ 75 E9                jnz short winmine.01004A61       ; 也不是就跳回去，不是我们要处理的消息

01004A78   .  33FF                 xor edi,edi

01004A7A   .  393D 40510001        cmp dword ptr ds:[1005140],edi

01004A80   .^ 74 DF                je short winmine.01004A61        ;
```

这三句是判断左键弹起的，程序里是这样判断的，保险起见，我们也这样判断

```
01004A82   >  8B45 14              mov eax,dword ptr ss:[ebp+14]      ; 这里开始是抄写源程序分解坐标的方法

01004A85   .  C1E8 10              shr eax,10

01004A88   .  83E8 27              sub eax,27

01004A8B   .  C1F8 04              sar eax,4                         ; 用了栈传递参数

01004A8E   .  50                   push eax                          ; 行

01004A8F   .  0FB745 14            movzx eax,word ptr ss:[ebp+14]

01004A93   .  83C0 04              add eax,4

01004A96   .  C1F8 04              sar eax,4

01004A99   .  50                   push eax                          ; 列

01004A9A   .  5E                   pop esi                           ; 取出列数

01004A9B   .  5B                   pop ebx                           ; 取出行数

01004A9C   .  85DB                 test ebx,ebx

01004A9E   .^ 7E C1                jle short winmine.01004A61

01004AA0   .  85F6                 test esi,esi

01004AA2   .^ 7E BD                jle short winmine.01004A61

01004AA4   .  3B1D 38530001        cmp ebx,dword ptr ds:[1005338]      ; 行数比较是否出界

01004AAA   .^ 7F B5                jg short winmine.01004A61

01004AAC   .  3B35 34530001        cmp esi,dword ptr ds:[1005334]

01004AB2   .^ 7F AD                jg short winmine.01004A61

01004AB4   .  EB 2D                jmp short winmine.01004AE3

01004AB6     90                    nop

01004AB7   >  8A8433 40530001      mov al,byte ptr ds:[ebx+esi+1005340] ; 到这里，判断方法和程序是一样的

01004ABE   .  3C 80                cmp al,80                         ; 判断是否有雷

01004AC0   .^ 72 9F                jb short winmine.01004A61          ; 小于即没有雷

01004AC2   .  BA 04020000          mov edx,204                       ; 有雷的情况把消息改成右击

01004AC7   .  C745 10 02000000     mov dword ptr ss:[ebp+10],2       ; 判断右击的标志
```

（右击时`wParam＝2`，程序里是要判断的，所以要改一下标志）

```
01004ACE   .^ EB 91                jmp short winmine.01004A61         ; 跳回去

01004AD0   .  00                   db 00                             ; 这里是是否为我们伪造消息的状态，1 为是，0 为否

01004AD1     00                    db 00

01004AD2   .  00000000             dd 00000000

01004AD6   .  00000000             dd 00000000                       ; 当前的列数

01004ADA   .  00000000             dd 00000000                       ; 当前的行数

01004ADE   >  C1E3 05              shl ebx,5

01004AE1   .^ EB D4                jmp short winmine.01004AB7

01004AE3   >  803D D04A0001 00      cmp byte ptr ds:[1004AD0],0        ; 跳到这里，先判断是否为我们伪造消息状态

01004AEA   .^ 75 F2                jnz short winmine.01004ADE         ; 是就跳走，不处理下面的

01004AEC   .  C605 D04A0001 01      mov byte ptr ds:[1004AD0],1        ; 标志为伪造消息状态

01004AF3   .  3E:8B5D 08            mov ebx,dword ptr ds:[ebp+8]

01004AF7   .  891D D24A0001         mov dword ptr ds:[1004AD2],ebx

01004AFD   >/ 8B35 D64A0001         mov esi,dword ptr ds:[1004AD6]      ; 取出列数

01004B03   .| 46                    inc esi                            ; ＋1

01004B04   .| 8935 D64A0001         mov dword ptr ds:[1004AD6],esi      ; 写回去

01004B0A   .| 3B35 34530001         cmp esi,dword ptr ds:[1005334]      ; 比较是否出界

01004B10   .| 7F 68                jg short winmine.01004B7A          ; 出界代表结束了，跳走

01004B12   .| C1E6 04              shl esi,4                          ; 计算坐标的逆运算

01004B15   .| 83EE 03              sub esi,3

01004B18   >|/8B1D DA4A0001        mov ebx,dword ptr ds:[1004ADA]      ; 取出行数

01004B1E   .||43                  inc ebx                              ; ＋1

01004B1F   .||891D DA4A0001        mov dword ptr ds:[1004ADA],ebx      ; 写回去

01004B25   .||3B1D 38530001        cmp ebx,dword ptr ds:[1005338]      ; 比较是否出界

01004B2B   .||7F 41                jg short winmine.01004B6E          ; 出界就跳走

01004B2D   .||33FF                 xor edi,edi                         ; edi 为参数 lParam

01004B2F   .||03FB                 add edi,ebx                         ; 坐标参数的逆运算

01004B31   .||C1E7 04              shl edi,4

01004B34   .||83C7 28              add edi,28

01004B37   .||C1E7 10              shl edi,10

01004B3A   .||03FE                 add edi,esi

01004B3C   .||57                   push edi                            ; /lParam

01004B3D   .||6A 01                push 1                              ; |wParam

01004B3F   .||68 01020000           push 201                            ; |msg=左键按下

01004B44   .||FF35 D24A0001         push dword ptr ds:[1004AD2]         ; |hWnd

01004B4A   .||E8 7AD0FFFF           call winmine.01001BC9               ; \\call WndProc

01004B4F   .||C705 40510001 01000000 mov dword ptr ds:[1005140],1       ; 左键弹起标志

01004B59   .||57                   push edi                            ; /lParam

01004B5A   .||6A 00                push 0                              ; |wParam

01004B5C   .||68 02020000           push 202                            ; |msg=左键弹起

01004B61   .||FF35 D24A0001         push dword ptr ds:[1004AD2]         ; |hWnd

01004B67   .||E8 5DD0FFFF           call winmine.01001BC9               ; \\call WndProc

01004B6C   .|\\EB AA               jmp short winmine.01004B18           ; 行数小循环

01004B6E   >| C705 DA4A0001 00000000 mov dword ptr ds:[1004ADA],0       ; 行数从 0 开始

01004B78   .\\ EB 83                jmp short winmine.01004AFD           ; 列数大循环

01004B7A   >  C705 D64A0001 00000000 mov dword ptr ds:[1004AD6],0       ; 列数从 0 开始

01004B84   .  C605 D04A0001 00       mov byte ptr ds:[1004AD0],0        ; 恢复伪造消息标志

01004B8B   .^ E9 D1FEFFFF            jmp winmine.01004A61                ; 出门咯^\_^
```

然后运行一下，随便点，填上你的大名，呵呵，1 秒扫雷记录^\_^

## 五、后记

很懒，去年 10 月份就搞定了，一直没写文章，主要是没法上网，不想写了。现在`ne365`要出杂志，所以就贡献一篇`pediy`的文章吧，感谢`ne365`里的所有死党，也感谢看雪论坛对我的帮助，感谢所有密界的朋友！

`4dd466d0e1e0cc123afbe504da2462ca`

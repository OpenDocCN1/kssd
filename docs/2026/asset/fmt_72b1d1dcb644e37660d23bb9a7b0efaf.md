
`004512FC`  `|. 64:8922` `MOV DWORD PTR FS:[EDX],ESP`

`004512FF`  `|. 6A 00` `PUSH 0`

`00451301`  `|. 6A 00` `PUSH 0`

`00451303`  `|. 68 00B00000` `PUSH 0B000`

`00451308`  `|. 8B45 FC` `MOV EAX,DWORD PTR SS:[EBP-4]`

`0045130B`  `|. E8 2C2B0100` `CALL WINRAMTU.00463E3C`

`00451310`  `|. 50` `PUSH EAX`

`00451311`  `|. E8 B6340700` `CALL <JMP.&USER32.SendMessageA>`

`00451316`  `|. 8B45 FC` `MOV EAX,DWORD PTR SS:[EBP-4]`

`00451319`  `|. 33D2` `XOR EDX,EDX`

`0045131B`  `|. 8990 2C020000` `MOV DWORD PTR DS:[EAX+22C],EDX`

```
00451321  |> 8B03                       /MOV EAX,DWORD PTR DS:[EBX]   //循环体开始，制造NAG。
00451323  |. E8 382D0000                |CALL WINRAMTU.00454060
00451328  |. 8B03                       |MOV EAX,DWORD PTR DS:[EBX]
0045132A 80B88400000000          cmp byte ptr [eax+00000084], 00      //这行和下面这行可以做点文章的地方了。
00451331 740F                    je 00451342                          //让这里跳到循环体后面即可。具体修改见后面。
00451333  |. 8B45 FC                    |MOV EAX,DWORD PTR SS:[EBP-4]
00451336  |. C780 2C020000 02000000     |MOV DWORD PTR DS:[EAX+22C],2
00451340  |. EB 14                      |JMP SHORT WINRAMTU.00451356
00451342  |> 8B45 FC                    |MOV EAX,DWORD PTR SS:[EBP-4]
00451345  |. 83B8 2C020000 00           |CMP DWORD PTR DS:[EAX+22C],0
0045134C  |. 74 08                      |JE SHORT WINRAMTU.00451356
0045134E  |. 8B45 FC                    |MOV EAX,DWORD PTR SS:[EBP-4]
00451351  |. E8 26FDFFFF                |CALL WINRAMTU.0045107C
00451356  |> 8B45 FC                    |MOV EAX,DWORD PTR SS:[EBP-4]
00451359  |. 8B80 2C020000              |MOV EAX,DWORD PTR DS:[EAX+22C]
0045135F  |. 85C0                       |TEST EAX,EAX
00451361  |.^74 BE                      \JE SHORT WINRAMTU.00451321    //循环体结束。在这个循环体内一点一点地取窗口内容显示到前面的那
个NAG窗体内。现在就是要把这个循环体跳过去。
```

`00451363`  `|. 8945 F8` `MOV DWORD PTR SS:[EBP-8],EAX` `//上面循环体显示窗口结束，来到这里。等你按按钮。如果按OK按钮，`

则`EAX=1`，则下面代码就是退出程序。还有一个`purchase`（购买 ）按钮，则`EAX=2`。

`00451366`  `|. 6A 00` `PUSH 0`

`00451368`  `|. 6A 00` `PUSH 0`

`0045136A`  `|. 68 01B00000` `PUSH 0B001`

`0045136F`  `|. 8B45 FC` `MOV EAX,DWORD PTR SS:[EBP-4]`

`00451372`  `|. E8 C52A0100` `CALL WINRAMTU.00463E3C`

我在跟踪的时候注意到循环结束后，`NAG`出现，如果你点击`OK`，则重新被断下，来到`00451363`，`EAX` 的值为`1`。所以，我们修改前面的跳转的时候，一定

记得给`EAX`赋值为`1`。因为后面的函数需要这个参数。所以前面的那个跳转指令修改成下面这个样子了：

```
0045132A     B8 01000000                MOV EAX,1
0045132F     90                         NOP
00451330     90                         NOP
00451331     EB 30                    JMP SHORT WINRAMTU.00451363
```

原来的程序代码指令空间刚好可以修改成新指令，绰绰有余啊！非常棒！立即用`ULTRAEDIT`修改吧！然后另行改名保存文件，以防你修改失败哟！！现在

启动程序运行，然后关闭，`OK`！没有`NAG`了，非常干脆利落的关掉了。

附一个失败方案：本来想从`4512D8`处把指令直接修改为`JMP 451359`，同时`451359`和`45135A`处修改为`MOV EAX,1` ，结果程序无法启动了，而且导致死机了

。我知道在`4512D8`到`00451321`之间的代码是必须的，不可以掠过去的。所以修改程序无法预见具体结果，必须实验再实验。

==========================================

4.现在已经启动已经没有问题了，不过在启动时还有一个小`NAG`，随着上面对最后一个大`NAG`的去除，我发现启动的时候第一个`SPLASH`版权窗口自动一闪

而过，不再停在那里等你按`OK` 了。好生奇怪啊！这是修改的副产品！捎带把第一个也消除了。实属必然！！破解不是目的，知道如何破解才是重要的。

我觉得还是研究一下如何把它去掉。

用`SOFTICE LOADER`加载程序，`F10`跟踪到下面`004BFA87` 处出现版权信息窗口，下断点，然后再次加载程序，`F5`退出`SOFTICE`，被拦住在`004BFA87` 处，`F8`

跟入，部分跟踪代码在后面。

```
004BFA81  |. E8 3A450000    CALL <JMP.&KERNEL32.GetModuleHandleA>
004BFA86  |. 50             PUSH EAX
004BFA87  |. FF56 18        CALL DWORD PTR DS:[ESI+18]   //这个CALL出那个SPLASH。:[ESI+18]的值为4012C0。
004BFA8A  |. 83C4 10        ADD ESP,10
004BFA8D  |. 50             PUSH EAX
004BFA8E  |. E8 BDF4FFFF    CALL WINRAMTU.004BEF50
004BFA93  |. 59             POP ECX
004BFA94  |. EB 21          JMP SHORT WINRAMTU.004BFAB7
004BFA96  |> 8B15 E8894E00  MOV EDX,DWORD PTR DS:[4E89E8]
004BFA9C  |. 52             PUSH EDX
004BFA9D  |. 8B0D E4894E00  MOV ECX,DWORD PTR DS:[4E89E4]
004BFAA3  |. 51             PUSH ECX
004BFAA4  |. A1 E0894E00    MOV EAX,DWORD PTR DS:[4E89E0]
004BFAA9  |. 50             PUSH EAX
004BFAAA  |. FF56 18        CALL DWORD PTR DS:[ESI+18]   //这个CALL出那个SPLASH。
004BFAAD  |. 83C4 0C        ADD ESP,0C
```

我们`F8`跟入前面那个`CALL`，用`F10`沿着下面这个路径跟踪：

```
00401554   . 8B0D 94E44D00              MOV ECX,DWORD PTR DS:[4DE494]
0040155A   . 8B01                       MOV EAX,DWORD PTR DS:[ECX]
0040155C   . 8B0D 58E44D00              MOV ECX,DWORD PTR DS:[4DE458]
00401562   . 8B15 749D4C00              MOV EDX,DWORD PTR DS:[4C9D74]
00401568   . E8 072C0500                CALL WINRAMTU.00454174      //这个CALL用F8切入跟踪，F10则出现窗口。
0040156D   . A1 94E44D00                MOV EAX,DWORD PTR DS:[4DE494]
00401572   . 8B00                       MOV EAX,DWORD PTR DS:[EAX]
00401574   . E8 7B2C0500                CALL WINRAMTU.004541F4
00401579   . 66:C745 C4 0000            MOV WORD PTR SS:[EBP-3C],0
0040157F   . EB 1B                      JMP SHORT WINRAMTU.0040159C
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

顺其自然，我们来到下面代码处：

```
0040FF13  |. 8B10                       MOV EDX,DWORD PTR DS:[EAX]
0040FF15  |. A1 78E44D00                MOV EAX,DWORD PTR DS:[4DE478]
0040FF1A  |. 8B08                       MOV ECX,DWORD PTR DS:[EAX]
0040FF1C  |. 8B81 D8020000              MOV EAX,DWORD PTR DS:[ECX+2D8]
0040FF22  |. E8 B9E10400                CALL WINRAMTU.0045E0E0
0040FF27  |. FF4D EC                    DEC DWORD PTR SS:[EBP-14]
0040FF2A  |. 8D45 FC                    LEA EAX,DWORD PTR SS:[EBP-4]
0040FF2D  |. BA 02000000                MOV EDX,2
0040FF32  |. E8 39390B00                CALL WINRAMTU.004C3870
0040FF37  |. 8B0D 78E44D00              MOV ECX,DWORD PTR DS:[4DE478]
0040FF3D  |. 8B01                       MOV EAX,DWORD PTR DS:[ECX]
0040FF3F  |. 8B10                       MOV EDX,DWORD PTR DS:[EAX]
0040FF41  |. FF92 CC000000              CALL DWORD PTR DS:[EDX+CC] //这个CALL出现版权窗口，F8跟入。:[EDX+CC]的值为4511DC。这段代码其
实就是前面第3部分所述的代码段了。说明2个窗口都调用同样的一个函数啦！部分节选如下：
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```
004511DC  /. 55                         PUSH EBP
004511DD  |. 8BEC                       MOV EBP,ESP
004511DF  |. 83C4 E0                    ADD ESP,-20
004511E2  |. 53                         PUSH EBX
004511E3  |. 56                         PUSH ESI
004511E4  |. 33D2                       XOR EDX,EDX
004511E6  |. 8955 E0                    MOV DWORD PTR SS:[EBP-20],EDX
004511E9  |. 8945 FC                    MOV DWORD PTR SS:[EBP-4],EAX
```

\*省去260多行代码

```
004512D3  |. E8 90A2FFFF                CALL WINRAMTU.0044B568
004512D8  |. 8945 F4                    MOV DWORD PTR SS:[EBP-C],EAX
004512DB  |. 33D2                       XOR EDX,EDX
004512DD  |. 55                         PUSH EBP
004512DE  |. 68 4C144500                PUSH WINRAMTU.0045144C
004512E3  |. 64:FF32                    PUSH DWORD PTR FS:[EDX]
004512E6  |. 64:8922                    MOV DWORD PTR FS:[EDX],ESP
004512E9  |. 8B45 FC                    MOV EAX,DWORD PTR SS:[EBP-4]
004512EC  |. E8 3BFEFFFF                CALL WINRAMTU.0045112C     //这个CALL出现版权窗口。同样关闭程序时的那个BIG NAG也调用这个函数
显示主框架，如前所述。
```

下面我给出另外一个简单的解决办法，就是把下面这个消息循环切断，不让它循环即可，最后两个`NAG`统统一闪即逝，根本影响不了什么了。：）

```
00451321  |> 8B03                       /MOV EAX,DWORD PTR DS:[EBX]
00451323  |. E8 382D0000                |CALL WINRAMTU.00454060
00451328  |. 8B03                       |MOV EAX,DWORD PTR DS:[EBX]
0045132A     80B8 84000000 00           CMP BYTE PTR DS:[EAX+84],0
00451331     74 0F                      JE SHORT WINRAMTU.00451342
00451333  |. 8B45 FC                    |MOV EAX,DWORD PTR SS:[EBP-4]
00451336  |. C780 2C020000 02000000     |MOV DWORD PTR DS:[EAX+22C],2
00451340  |. EB 14                      |JMP SHORT WINRAMTU.00451356
00451342  |> 8B45 FC                    |MOV EAX,DWORD PTR SS:[EBP-4]
00451345  |. 83B8 2C020000 00           |CMP DWORD PTR DS:[EAX+22C],0
0045134C  |. 74 08                      |JE SHORT WINRAMTU.00451356
0045134E  |. 8B45 FC                    |MOV EAX,DWORD PTR SS:[EBP-4]
00451351  |. E8 26FDFFFF                |CALL WINRAMTU.0045107C
```

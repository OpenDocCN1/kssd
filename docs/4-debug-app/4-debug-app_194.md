
# 我也玩扫雷

> 原文：[`www.pediy.com/kssd/pediy12/138059.html`](https://www.pediy.com/kssd/pediy12/138059.html)

前段时间看了这个帖子 `http://bbs.pediy.com/showthread.php?t=136362`。

忽然发现，原来高手都是这么玩扫雷的。不过 `CE` 什么的都不怎么用，以至于都不会用！`so....` 还是拿 `OD` 自己跟吧。

这篇文章的思路就是，首先提出自己想要跟踪什么代码，然后使用 `OD` 定位到要跟踪的代码，至于代码分析，请大家参考本文开头提到的帖子。

## 1。寻找棋盘初始化函数。

邓韬同学的文章的开头提到一个图，但是我实在是木有找到那个图在哪。。。。

定位棋盘初始化，我的思路是，先找到一处对棋盘的操作，然后下内存写断点。按照前辈们的理论：程序 ＝ 数据结构 ＋ 对数据结构的操作。那么我推测棋盘最有可能的存储方式是一个数组。然后结合寻址方式，基址变址寻址，也就是应该长得类似这个样子 `[xxxx + yyy*dd]` 类似这种形式的 `mov` 或 `lea` 指令都是需要注意的。

好了，思路简单说明完毕，现在开搞。

`OD load` 扫雷，`f9`，按快捷工具栏的 `W` 那个图标，找到窗口的消息处理函数，双击定位到消息处理函数，`shift + f2` 下条件断，条件为 `[esp + 0x08] == 0x201`，也就是 `wm_lbuttondown` 消息。为了保险，你也可以断 `buttonup` 消息，这个大家看自己喜好。

切换会扫雷，随便点一个格子，`od` 断下，这时在 `cpu` 窗口中右键，`RUN` 跟踪，添加函数过程中的分支。`OK`，`f9` 让程序自己跑去吧。等 `OD` 提示当前状态为运行的时候，再次在 `cpu` 窗口中点右键，`run` 跟中，当前模块统计。这里我们只看调用次数为 `1` 的代码片段。

代码:

```
01001FA6  |>  \393D 48510001 cmp dword ptr ds:[1005148],edi                 ;  Case 201 (WM_LBUTTONDOWN) of switch 01001F5F
01001FAC  |.^ 75 D6         jnz short winmine.01001F84
01001FAE  |.  FF75 14       push [arg.4]
01001FB1  |.  E8 56F4FFFF   call winmine.0100140C
01001FB6  |.  85C0         test eax,eax
01001FB8  |.^ 0F85 A0FCFFFF jnz winmine.01001C5E
01001FBE  |.  841D 00500001 test byte ptr ds:[1005000],bl
01001FC4  |.  0F84 DF010000 je winmine.010021A9
01001FCA  |.  8B45 10       mov eax,[arg.3]
01001FCD  |.  24 06        and al,6
01001FCF  |.  F6D8        neg al
01001FD1  |.  1BC0        sbb eax,eax
01001FD3  |.  F7D8        neg eax
01001FD5  |.  A3 44510001  mov dword ptr ds:[1005144],eax
01001FDA  |.  E9 80000000  jmp winmine.0100205F
01001FDF  |>   33FF         xor edi,edi                                     ;  Cases 202 (WM_LBUTTONUP),205 (WM_RBUTTONUP),208 (WM_MBUTTONUP) of switch 01001F5F
01001FE1  |.  393D 40510001 cmp dword ptr ds:[1005140],edi
01001FE7  |.  0F84 BC010000 je winmine.010021A9
01001FED  |>   893D 40510001 mov dword ptr ds:[1005140],edi
01001FF3  |.  FF15 D8100001 call dword ptr ds:[<&USER32.ReleaseCapture>]   ; [ReleaseCapture
01001FF9  |.  841D 00500001 test byte ptr ds:[1005000],bl
01001FFF  |.  0F84 B6000000 je winmine.010020BB
01002005  |.  E8 D7170000  call winmine.010037E1
0100200A  |.  E9 9A010000  jmp winmine.010021A9

```

比较有价值的 `call` 是

`01001FB1  |.  E8 56F4FFFF   call winmine.0100140C`

和

`01002005  |.  E8 D7170000   call winmine.010037E1`

删除所有断点，在上面两个地址处重新下断，重来。在扫雷点左键，`OD` 断下。看了一下 `0100140c` 这个 `call`，作用只是画出鼠标点下去时格子凹下的动画。

那么，重点莫非是 `010037e1`？跟进去看一下：

代码:

```
010037E1  /$  A1 18510001   mov eax,dword ptr ds:[1005118]
010037E6  |.  85C0        test eax,eax
010037E8  |.  0F8E C8000000 jle winmine.010038B6
010037EE  |.  8B0D 1C510001 mov ecx,dword ptr ds:[100511C]
010037F4  |.  85C9        test ecx,ecx
010037F6  |.  0F8E BA000000 jle winmine.010038B6
010037FC  |.  3B05 34530001 cmp eax,dword ptr ds:[1005334]
01003802  |.  0F8F AE000000 jg winmine.010038B6
01003808  |.  3B0D 38530001 cmp ecx,dword ptr ds:[1005338]
0100380E  |.  0F8F A2000000 jg winmine.010038B6
01003814  |.  53          push ebx
01003815  |.  33DB        xor ebx,ebx
01003817  |.  43          inc ebx
01003818  |.  833D A4570001>cmp dword ptr ds:[10057A4],0
0100381F  |.  75 4A        jnz short winmine.0100386B
01003821  |.  833D 9C570001>cmp dword ptr ds:[100579C],0
01003828  |.  75 41        jnz short winmine.0100386B
0100382A  |.  53          push ebx
0100382B  |.  E8 BD000000  call winmine.010038ED
01003830  |.  FF05 9C570001 inc dword ptr ds:[100579C]
01003836  |.  E8 7AF0FFFF  call winmine.010028B5
0100383B  |.  6A 00        push 0                         ; /Timerproc = NULL
0100383D  |.  68 E8030000  push 3E8                       ; |Timeout = 1000. ms
01003842  |.  53          push ebx                       ; |TimerID
01003843  |.  FF35 245B0001 push dword ptr ds:[1005B24]                ; |hWnd = 00050134 ('扫雷',class='扫雷')
01003849  |.  891D 64510001 mov dword ptr ds:[1005164],ebx             ; |
0100384F  |.  FF15 B4100001 call dword ptr ds:[<&USER32.SetTimer>]      ; \SetTimer
01003855  |.  85C0        test eax,eax
01003857  |.  75 07        jnz short winmine.01003860
01003859  |.  6A 04        push 4
0100385B  |.  E8 F0000000  call winmine.01003950
01003860  |>  A1 18510001  mov eax,dword ptr ds:[1005118]
01003865  |.  8B0D 1C510001 mov ecx,dword ptr ds:[100511C]
0100386B  |>  841D 00500001 test byte ptr ds:[1005000],bl
01003871  |.  5B          pop ebx
01003872  |.  75 10        jnz short winmine.01003884
01003874  |.  6A FE        push -2
01003876  |.  59          pop ecx
01003877  |.  8BC1        mov eax,ecx
01003879  |.  890D 1C510001 mov dword ptr ds:[100511C],ecx
0100387F  |.  A3 18510001  mov dword ptr ds:[1005118],eax
01003884  |>  833D 44510001>cmp dword ptr ds:[1005144],0
0100388B  |.  74 09        je short winmine.01003896
0100388D  |.  51          push ecx
0100388E  |.  50          push eax
0100388F  |.  E8 23FDFFFF  call winmine.010035B7
01003894  |.  EB 20        jmp short winmine.010038B6
01003896  |>  8BD1        mov edx,ecx
01003898  |.  C1E2 05      shl edx,5
0100389B  |.  8A9402 405300>mov dl,byte ptr ds:[edx+eax+1005340]
010038A2  |.  F6C2 40      test dl,40
010038A5  |.  75 0F        jnz short winmine.010038B6
010038A7  |.  80E2 1F      and dl,1F
010038AA  |.  80FA 0E      cmp dl,0E
010038AD  |.  74 07        je short winmine.010038B6
010038AF  |.  51          push ecx
010038B0  |.  50          push eax
010038B1  |.  E8 5CFCFFFF  call winmine.01003512
010038B6  |>  FF35 60510001 push dword ptr ds:[1005160]
010038BC  |.  E8 52F0FFFF  call winmine.01002913
010038C1  \\.  C3          retn

```

当然，这里你可以再用一次 `run` 跟踪，不过我太懒。。。直接 `f8` 单步到这一行：

`0100389B  |.  8A9402 405300>mov dl,byte ptr ds:[edx+eax+1005340]`

传说中的 `mov`，传说中的基址变址寻址，定位到内存看一眼：

代码:

```
01005331  00 00 00 09 00 00 00 09 00 00 00 00 00 00 00 10    ...............
01005341  10 10 10 10 10 10 10 10 10 10 0F 0F 0F 0F 0F 0F    
01005351  0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 10    
01005361  00 0F 0F 8F 0F 0F 0F 0F 0F 10 0F 0F 0F 0F 0F 0F    .?
01005371  0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 10    
01005381  0F 8F 8F 0F 0F 0F 0F 0F 0F 10 0F 0F 0F 0F 0F 0F    
01005391  0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 10    
010053A1  0F 0F 8F 0F 0F 0F 0F 0F 0F 10 0F 0F 0F 0F 0F 0F    ?
010053B1  0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 0F 10    

```

内牛满面啊。。。赶紧内存写断点。重开一局，`OD` 断在 `01002EDB` ：

代码:



```
01002ED5  /$  B8 60030000    mov eax,360
01002EDA  |>  48             /dec eax
01002EDB  |.  C680 40530001>|mov byte ptr ds:\[eax+1005340\],0F
01002EE2  |.^ 75 F6          \\jnz short winmine.01002EDA
01002EE4  |.  8B0D 34530001  mov ecx,dword ptr ds:\[1005334\]
01002EEA  |.  8B15 38530001  mov edx,dword ptr ds:\[1005338\]
01002EF0  |.  8D41 02        lea eax,dword ptr ds:\[ecx+2\]
01002EF3  |.  85C0           test eax,eax
01002EF5  |.  56             push esi
01002EF6  |.  74 19          je short winmine.01002F11
01002EF8  |.  8BF2           mov esi,edx
01002EFA  |.  C1E6 05        shl esi,5
01002EFD  |.  8DB6 60530001  lea esi,dword ptr ds:\[esi+1005360\]
01002F03  |>  48             /dec eax
01002F04  |.  C680 40530001>|mov byte ptr ds:\[eax+1005340\],10
01002F0B  |.  C60406 10      |mov byte ptr ds:\[esi+eax\],10
01002F0F  |.^ 75 F2          \\jnz short winmine.01002F03
01002F11  |>  8D72 02        lea esi,dword ptr ds:\[edx+2\]
01002F14  |.  85F6           test esi,esi
01002F16  |.  74 21          je short winmine.01002F39
01002F18  |.  8BC6           mov eax,esi
01002F1A  |.  C1E0 05        shl eax,5
01002F1D  |.  8D90 40530001  lea edx,dword ptr ds:\[eax+1005340\]
01002F23  |.  8D8408 415300>lea eax,dword ptr ds:\[eax+ecx+1005341\]
01002F2A  |>  83EA 20        /sub edx,20
01002F2D  |.  83E8 20        |sub eax,20
01002F30  |.  4E             |dec esi
01002F31  |.  C602 10        |mov byte ptr ds:\[edx\],10
01002F34  |.  C600 10        |mov byte ptr ds:\[eax\],10
01002F37  |.^ 75 F1          \\jnz short winmine.01002F2A
01002F39  |>  5E             pop esi
01002F3A  \\.  C3            retn
```

删除内存断点（记得一定要删哦，亲 ），`ctrl+f9`：  

代码:

```
0100367A  /$  A1 AC560001    mov eax,dword ptr ds:\[10056AC\]
0100367F  |.  8B0D A8560001  mov ecx,dword ptr ds:\[10056A8\]
01003685  |.  53             push ebx
01003686  |.  56             push esi
01003687  |.  57             push edi
01003688  |.  33FF           xor edi,edi
0100368A  |.  3B05 34530001  cmp eax,dword ptr ds:\[1005334\]
01003690  |.  893D 64510001  mov dword ptr ds:\[1005164\],edi
01003696  |.  75 0C          jnz short winmine.010036A4
01003698  |.  3B0D 38530001  cmp ecx,dword ptr ds:\[1005338\]
0100369E  |.  75 04          jnz short winmine.010036A4
010036A0  |.  6A 04          push 4
010036A2  |.  EB 02          jmp short winmine.010036A6
010036A4  |>  6A 06          push 6
010036A6  |>  5B             pop ebx
010036A7  |.  A3 34530001    mov dword ptr ds:\[1005334\],eax
010036AC  |.  890D 38530001  mov dword ptr ds:\[1005338\],ecx
010036B2  |.  E8 1EF8FFFF    call winmine.01002ED5                     ;  init game
010036B7  |.  A1 A4560001    mov eax,dword ptr ds:\[10056A4\]
010036BC  |.  893D 60510001  mov dword ptr ds:\[1005160\],edi
010036C2  |.  A3 30530001    mov dword ptr ds:\[1005330\],eax
010036C7  |>  FF35 34530001  push dword ptr ds:\[1005334\]
010036CD  |.  E8 6E020000    call <winmine.rand>                       ;  rand
010036D2  |.  FF35 38530001  push dword ptr ds:\[1005338\]
010036D8  |.  8BF0           mov esi,eax
010036DA  |.  46             inc esi
010036DB  |.  E8 60020000    call <winmine.rand>
010036E0  |.  40             inc eax
010036E1  |.  8BC8           mov ecx,eax
010036E3  |.  C1E1 05        shl ecx,5
010036E6  |.  F68431 405300>test byte ptr ds:\[ecx+esi+1005340\],80
010036EE  |.^ 75 D7          jnz short winmine.010036C7
010036F0  |.  C1E0 05        shl eax,5
010036F3  |.  8D8430 405300>lea eax,dword ptr ds:\[eax+esi+1005340\]
010036FA  |.  8008 80        or byte ptr ds:\[eax\],80
010036FD  |.  FF0D 30530001  dec dword ptr ds:\[1005330\]
01003703  |.^ 75 C2          jnz short winmine.010036C7
01003705  |.  8B0D 38530001  mov ecx,dword ptr ds:\[1005338\]
0100370B  |.  0FAF0D 345300>imul ecx,dword ptr ds:\[1005334\]
01003712  |.  A1 A4560001    mov eax,dword ptr ds:\[10056A4\]
01003717  |.  2BC8           sub ecx,eax
01003719  |.  57             push edi
0100371A  |.  893D 9C570001  mov dword ptr ds:\[100579C\],edi
01003720  |.  A3 30530001    mov dword ptr ds:\[1005330\],eax
01003725  |.  A3 94510001    mov dword ptr ds:\[1005194\],eax
0100372A  |.  893D A4570001  mov dword ptr ds:\[10057A4\],edi
01003730  |.  890D A0570001  mov dword ptr ds:\[10057A0\],ecx
01003736  |.  C705 00500001>mov dword ptr ds:\[1005000\],1
01003740  |.  E8 25FDFFFF    call winmine.0100346A
01003745  |.  53             push ebx
01003746  |.  E8 05E2FFFF    call winmine.01001950
0100374B  |.  5F             pop edi
0100374C  |.  5E             pop esi
0100374D  |.  5B             pop ebx
0100374E  \\.  C3            retn
```

至于你信不信上面是初始化代码片段，反正我是信了![](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色") 。剩下的大家去参考邓韬牛牛的帖子吧。

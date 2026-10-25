
# IglooFTP PRO v3.0

> 原文：[`www.pediy.com/kssd/pediy04/kanxue020.htm`](https://www.pediy.com/kssd/pediy04/kanxue020.htm)

脱壳 `IglooFTP PRO v3.0` 的详细过程。

在我贴过注册 `Advanced Ra-Renamer1.2` 后，有人回贴要我试 `IglooFTP PRO v3.0`，ok no problem。不过这位 1212 兄真是有眼光，知道我在集中学习脱 `aspr` 保护的期间有贴必回(当然是回的是有关 `aspr` 壳保护的贴子啦)，不知道是要考考我还是要逼我学手动脱 `aspr` 的壳。不管这么多了，反正学会了，这可是我第一次手动脱壳成功啦，为了庆祝一下，过程稍微写详细点好啦。let's go~~~~~~`

`unpack(2001.9.13)`

这个程序无法利用快速寻找 `aspr` 入口的方法找到程序的 `oep`，原因就是 `IglooFTP PRO v3.0` 是用 `vc` 写的，而那种方法只对 `delphi` 程序有效，`vc` 程序入口没有特征，所以只能手动脱壳。具体的参考我写的 `aspr` 脱壳总结。

首先，用 `trw`(我选 `trw` 的原因：1.方便写过程，用 `u x,x` ><drive:>`x` 就可以导出代码，2.`s-ice`+`icedump` 跑不到这么远就会当掉的)载入 `IglooFTP PRO`，下 `bpx getprocaddress`，`g`，被拦到后，按 2 次 `f12`，接着按 `f10` 慢慢跟踪到：

```
0187:00AC1ACD  LEA       EAX,[EAX+00]
0187:00AC1AD0  PUSH      EBP
0187:00AC1AD1  MOV       EBP,ESP
0187:00AC1AD3  ADD       ESP,BYTE -0C
0187:00AC1AD6  CALL      00AB3130
0187:00AC1ADB  JNZ       NEAR 00AB3E4C
0187:00AC1AE1  CALL      00AB4308
0187:00AC1AE6  CALL      00AB8AC8
0187:00AC1AEB  CALL      00AB945C
0187:00AC1AF0  CALL      00ABBF90
0187:00AC1AF5  CALL      00AB3E4C <---按 f8 从这进去
0187:00AC1AFA  MOV       ESP,EBP
0187:00AC1AFC  POP       EBP
```

按 `f10` 慢慢走到：

```
0187:00AC05EA  MOV       EAX,[ESP+0C]
0187:00AC05EE  JMP       SHORT 00AC05F1  <---从这开始要小心啦，偶现在还眼花呢，原来花指令是这个意思呀;-)
0187:00AC05F0  XCHG      AL,[EBX+B880]
0187:00AC05F6  ADD       [EDX],AL
0187:00AC05F8  JMP       SHORT 00AC0614
0187:00AC05FA  MOV       ESP,EBE817EB
0187:00AC05FF  ADC       AL,E8
0187:00AC0601  JMP       SHORT 00AC0614
0187:00AC0603  CALL      EC9414F3
0187:00AC0608  OR        EBP,EAX
0187:00AC060A  JMP       SHORT 00AC0614
0187:00AC060C  INT       20
0187:00AC060E  JMP       SHORT 00AC0614
0187:00AC0610  CALL      32940800
0187:00AC0615  ROL       BL,EB
0187:00AC0618  ADD       EAX,EBP
```

在 `00AC119F` 直接下断好了，可以跳过那些让人眼花的代码。

```
0187:00AC118D  MOV       EDX,02
0187:00AC1192  CALL      00AB31B0
0187:00AC1197  RET
0187:00AC1198  JMP       00AB2DE8
0187:00AC119D  JMP       SHORT 00AC1172
0187:00AC119F  POP       EDI     <----bpx 这里
0187:00AC11A0  POP       ESI
0187:00AC11A1  POP       EBX
0187:00AC11A2  MOV       ESP,EBP
0187:00AC11A4  POP       EBP
0187:00AC11A5  RET
```

按几次 `f10` 会走到：

```
0187:00AC11C4  PUSH      DWORD 331A69FF
0187:00AC11C9  PUSH      DWORD 2550
0187:00AC11CE  PUSH      DWORD EC70
0187:00AC11D3  PUSH      DWORD 00016000
0187:00AC11D8  PUSH      DWORD [00AC3014]
0187:00AC11DE  CALL      00AC11E4
0187:00AC11E3  ADD       DWORD [EBX+88E804C4],E8FFFFB8
0187:00AC11ED  ADD       [EAX],EAX
0187:00AC11EF  ADD       [EAX],AL
0187:00AC11F1  ADD       DWORD [EBX+043104C4],0001E824
0187:00AC11FB  ADD       [EAX],AL
0187:00AC11FD  PUSH      DWORD 8B04C483
0187:00AC1202  ADD       EAX,00AC3014
0187:00AC1207  CALL      00AC120E    <----bpx 这里，接着按 f8 走出去
0187:00AC120C  CALL      05709579
0187:00AC1211  ADD       [ESP],EAX
0187:00AC1214  RET
0187:00AC1215  RET
```

按几次 `f10` 会走到：

```
0187:00AC005D  LEA       EAX,[EAX+00]
0187:00AC0060  MOV       EAX,00AC39BC
0187:00AC0065  MOV       EDX,0A
0187:00AC006A  CALL      00ABC318
0187:00AC006F  CALL      00ABFEA0  <----按 f8 进去
0187:00AC0074  RET
```

按几次 `f10` 会走到：

```
0187:00ABFFD5  CALL      00AB2E90
0187:00ABFFDA  CMP       DWORD [00AC39A4],BYTE +00
0187:00ABFFE1  JZ        00ABFFF7
0187:00ABFFE3  PUSH      BYTE +04
0187:00ABFFE5  MOV       ECX,00AC39A4
0187:00ABFFEA  LEA       EAX,[EBP-08]
0187:00ABFFED  MOV       EDX,04
0187:00ABFFF2  CALL      00ABC470
0187:00ABFFF7  CALL      00AC0011    <----按 f8 进去
0187:00ABFFFC  MOV       EAX,[ESP+0C]
0187:00AC0000  ADD       DWORD [EAX+B8],BYTE +02
0187:00AC0007  MOV       DWORD [EAX+18],00
```

按几次 `f10` 会走到：

```
0187:00AC0043  PUSH      DWORD [EBP-08]
0187:00AC0046  MOV       EAX,[EBP-0C]
0187:00AC0049  CMP       DWORD [EAX],BYTE +00
0187:00AC004C  JZ        00AC0050
0187:00AC004E  PUSH      DWORD [EAX]
0187:00AC0050  PUSH      DWORD [EBP-10]
0187:00AC0053  JMP       NEAR [EBP-14]  <----fs0 大哥说走到这里就离 oep 不远了，????
0187:00AC0056  POP       EDI
0187:00AC0057  POP       ESI
0187:00AC0058  POP       EBX
0187:00AC0059  MOV       ESP,EBP
0187:00AC005B  POP       EBP
0187:00AC005C  RET
```

按几次 `f10` 会走到：

```
0187:00AD2970  PUSH      DWORD 063429E1  <----直接跳到这里
0187:00AD2975  POP       EAX
0187:00AD2976  JMP       00AD298E
0187:00AD297B  MOV       EDI,DBEAD58C
0187:00AD2980  JS        00AD29D3
0187:00AD2982  MOV       DH,B7
0187:00AD2984  AND       AL,8D
0187:00AD2986  INC       EDX
0187:00AD2987  PUSH      EBX
0187:00AD2988  NOP
0187:00AD2989  MOV       [ESI+E845BCAF],ECX
0187:00AD298F  SLDT      [EAX]
0187:00AD2992  ADD       BL,CL
```

走到这里 `trw` 的任务基本完成了，因为下面是死循环(不知道这么说是否恰当)，我搜了周围超过 20000 行代码也没找到 `jmp eax`，所以该换 `superbpm`+`icedump`+`s-ice` 强档组合了。用 `sice` 载入 `IglooFTP PRO`，下 `g ad2970`，按几次 `f8` 来到死循环处，下 `/tracex 400000 eip-8`，`g`，hoho 直接来到程序的入口处 `47f732`，至此寻找 `oep` 的任务结束了，继续脱壳。

ok，下面要在入口处脱壳，打开 `SuperBPM`，点 `erase`，用 `trw` 载入 `IglooFTP PRO v3.0`，下 `g 47f732`，下 `suspend`。用 `prodump` 选 `IglooFTP PRO` 进程 `dump(full)`，再打 `ctrl+n`，`f5`。

接着修复 `import table`：`ImportREC1.2final` 无法提取完整的 `it`，要手动修复超过 70 个函数，累死偶啦~~~，配合 `ImportREC1.2beta2` 重建 `it` 效果会好些，hoho,good luck:)

以下操作必须在 `IglooFTP PRO` 的地盘里才能探测到

```
FThunk: 0008A01C    NbFunc: 00000002

1     0008A01C    COMCTL32.dll    003B    ImageList_GetIcon
1     0008A020    COMCTL32.dll    002A    ImageList_AddMasked
1     0008A09C    KERNEL32.dll    01E9    GetVersion
1     0008A0B0    KERNEL32.dll    00C7    CreateProcessA
1     0008A0BC    KERNEL32.dll    0163    GetCurrentProcessId
1     0008A0F8    KERNEL32.dll    0129    FindNextFileA
1     0008A0FC    KERNEL32.dll    0120    FindClose
1     0008A100    KERNEL32.dll    017F    GetFileSize
1     0008A104    KERNEL32.dll    0306    Sleep
1     0008A108    KERNEL32.dll    01E1    GetTickCount
```

```
FThunk: 0008AE88    NbFunc: 0000003F

1     0008AE88    USER32.dll    0215    SetCursor
1     0008AE8C    USER32.dll    019D    LoadCursorA
1     0008AE90    USER32.dll    01E4    RedrawWindow
1     0008AE94    USER32.dll    00E2    GetCapture
1     0008AE98    USER32.dll    0038    CheckMenuItem
1     0008AE9C    USER32.dll    015F    GrayStringA
1     0008AEA0    USER32.dll    01C1    ModifyMenuA
1     0008AEA4    USER32.dll    0175    InflateRect
1     0008AEA8    USER32.dll    0133    GetParent
1     0008AEAC    USER32.dll    01E1    PtInRect
1     0008AEB0    USER32.dll    00B6    EnableMenuItem
1     0008AEB4    USER32.dll    013F    GetSubMenu
1     0008AEB8    USER32.dll    01DE    PostQuitMessage
1     0008AEBC    USER32.dll    025C    TabbedTextOutA
1     0008AEC0    USER32.dll    00B0    DrawTextA
1     0008AEC4    USER32.dll    0226    SetMenuDefaultItem
1     0008AEC8    USER32.dll    01DC    PostMessageA
1     0008AECC    USER32.dll    01C7    MsgWaitForMultipleObjects
1     0008AED0    USER32.dll    01D9    PeekMessageA
1     0008AED4    USER32.dll    0198    KillTimer
1     0008AED8    USER32.dll    01A1    LoadIconA
1     0008AEDC    USER32.dll    023C    SetTimer
1     0008AEE0    USER32.dll    000F    BringWindowToTop
1     0008AEE4    USER32.dll    017E    InvalidateRect
1     0008AEE8    USER32.dll    00EE    GetClientRect
1     0008AEEC    USER32.dll    020D    SetCapture
1     0008AEF0    USER32.dll    01F4    ReleaseCapture
1     0008AEF4    USER32.dll    01A7    LoadMenuA
1     0008AEF8    USER32.dll    0140    GetSysColor
1     0008AEFC    USER32.dll    00B8    EnableWindow
1     0008AF00    USER32.dll    0299    WinHelpA
1     0008AF04    USER32.dll    0193    IsWindow
1     0008AF08    USER32.dll    01A3    LoadImageA
1     0008AF0C    USER32.dll    0157    GetWindowRect
```



1    `0008AF10`    `USER32.dll`    `0204`    `SendMessageA`   

1    `0008AF14`    `USER32.dll`    `019B`    `LoadBitmapA`   

1    `0008AF18`    `USER32.dll`    `0277`    `UpdateWindow`   

1    `0008AF1C`    `USER32.dll`    `01BA`    `MessageBoxA`   

1    `0008AF20`    `USER32.dll`    `0192`    `IsRectEmpty`   

1    `0008AF24`    `USER32.dll`    `0143`    `GetSystemMetrics`   

1    `0008AF28`    `USER32.dll`    `00D5`    `FillRect`   

1    `0008AF2C`    `USER32.dll`    `00E1`    `GetAsyncKeyState`   

1    `0008AF30`    `USER32.dll`    `0224`    `SetMenu`   

1    `0008AF34`    `USER32.dll`    `01C6`    `MoveWindow`   

1    `0008AF38`    `USER32.dll`    `018A`    `IsClipboardFormatAvailable`   

1    `0008AF3C`    `USER32.dll`    `00F0`    `GetClipboardData`   

1    `0008AF40`    `USER32.dll`    `01D0`    `OpenClipboard`   

1    `0008AF44`    `USER32.dll`    `00B5`    `EmptyClipboard`   

1    `0008AF48`    `USER32.dll`    `0213`    `SetClipboardData`   

1    `0008AF4C`    `USER32.dll`    `0040`    `CloseClipboard`   

1    `0008AF50`    `USER32.dll`    `0128`    `GetMessagePos`   

1    `0008AF54`    `USER32.dll`    `01FB`    `ScreenToClient`   

1    `0008AF58`    `USER32.dll`    `01B7`    `MapWindowPoints`   

1    `0008AF5C`    `USER32.dll`    `0104`    `GetFocus`   

1    `0008AF60`    `USER32.dll`    `0210`    `SetClassLongA`   

1    `0008AF64`    `USER32.dll`    `003E`    `ClientToScreen`   

1    `0008AF68`    `USER32.dll`    `01E9`    `RegisterClipboardFormatA`   

1    `0008AF6C`    `USER32.dll`    `0232`    `SetRect`   

1    `0008AF70`    `USER32.dll`    `0008`    `AppendMenuA`   

1    `0008AF74`    `USER32.dll`    `0196`    `IsWindowVisible`   

1    `0008AF78`    `USER32.dll`    `01F6`    `RemoveMenu`   

1    `0008AF7C`    `USER32.dll`    `00AC`    `DrawMenuBar`   

1    `0008AF80`    `USER32.dll`    `01B9`    `MessageBeep`   

`FThunk`: `0008B008`    `NbFunc`: `00000001`   

`0`    `0008B008`    ?    `0000`    `00410150`   

最后这个函数我实在是不知道干什么的了，删掉好啦:)

选`add new section`，然后点`fix dump`。

最后把`dump.exe`的`ep`改成`0007f732`。

修复`it`后，运行`IglooFTP PRO`非法操作(哪个程序能给我个惊喜不来这个呀)，所以就载入看看啦。发现错误在这个地方：

```
:00478593 8B0D28ED4A00                   mov ecx, dword ptr [004AED28]
\* Reference To: ADVAPI32., Ord:0000h
                    |
:00478599 E8726A0000                  Call 0047F010
:0047859E FF1554ED4A00                 call dword ptr [004AED54] ->call 477c90
```

这个只能用原版跟进去看看啦，发现这里最后`call`的是`477c90`，在对应的位置改一下就好了。

```
:004785A4 8B8D2CFFFFFF                 mov ecx, dword ptr [ebp+FFFFFF2C]
:004785AA E811040000                  call 004789C0
:004785AF 8B952CFFFFFF                 mov edx, dword ptr [ebp+FFFFFF2C]
:004785B5 8982D8000000                 mov dword ptr [edx+000000D8], eax
:004785BB 8B852CFFFFFF                 mov eax, dword ptr [ebp+FFFFFF2C]
:004785C1 83B8D800000000                 cmp dword ptr [eax+000000D8], 00000000
:004785C8 7416                         je 004785E0
:004785CA B9C8EC4A00                   mov ecx, 004AECC8
:004785CF E83C090000                  call 00478F10
:004785D4 8B8D2CFFFFFF                 mov ecx, dword ptr [ebp+FFFFFF2C]
:004785DA 8981D8000000                 mov dword ptr [ecx+000000D8], eax
```

运行正常，点`ABOUT`菜单还是非法操作，继续查找导致错误的原因。在这里：

```
:00477D18 FF155CED4A00     CALL [004AED54]
:00477D38 FF1560ED4A00     CALL [004AED60]
```

`call`的内容是个`ret`，老规矩用`winhex`统统替换成`90`也就是`nop`，还好只有一次。哪位大哥有更好的改法，麻烦说一声。

再有在内存中看到`AB3405`也跟上面3个地址`ABC8FC`,`2`个`ABC850`在一起，这个肯定会导致某个命令的非法操作，不过在不联网的状况下我无法找到，哪位大哥能找到的话，麻烦把这个补全了，多谢。

至此运行正常，脱壳任务完成了，有错误，不当的地方或建议的话，请提出。

关于程序的`crack`部分，我只是草草看了一下，发现我在短时间内无法完成，还是请有精力有时间的兄弟们代劳吧，先让我把突击`aspr`脱壳学习完成再说。第一次手动脱壳成功花了我大约3小时，笨呀，天生的:(，不过偶会努力啦:)。

下载：`http://zombieys.cn.hongnet.com/unp-iftppro.rar`

`xixi`包括完整的`import table`哦

`2001.9.14`

`zombieys[CCG]`

———————————————————————————————>

```
             .-"       "-.       unpacked by zombieys[CCG] >
             /              \       qq:1789655                       >
             |       ★          |      http://zombieys.yeah.net   >
             |,   .-.   .-.   ,|      http://zombieys.126.com   >
               |)(\_/   \\_)(|        zombieys.cn.hongnet.com   >
               |/   /\\   \\|                         >
       (@_@)   (\_   ^^   \_)        Thanks for your supports   >
      \_   )\_\_\_\_\_\_\_\_\_|IIIIII|\_/\_\_\_\_                         >
\_)@8@8{}<\_\_\_\_\_\_\_\_|-\\IIIIII/-|\_\_\_China Crack Group\_zombieys\_\_>  4dd466d0e1e0cc123afbe504da2462ca
```


# Minke 脱壳菜文求码

> 原文：[`www.pediy.com/kssd/pediy11/105006.html`](https://www.pediy.com/kssd/pediy11/105006.html)

【文章标题】: 手脱 `Minke 1.0.1` 的壳

【文章作者】: `lovebird/爱鸟`

【作者邮箱】: `lovebird_ustc#126.com`

【软件名称】: `Minkecn` 加压的 `Unpacktest`

【软件大小】: 压缩前 104KB 压缩后 126KB//这个也叫压缩...

【下载地址】: 在附件里面

【加壳方式】: `Minke 1.0.1`

【保护方式】: `Minke 1.0.1`

【编写语言】: `VC++ 6.0 Debug`

【使用工具】: `LordPe` `OD` `Peid.94`

【操作平台】: `XP Sp3` `Vmware`

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

----------------------------------------------------------------

【详细过程】

使用 `Minkecn` 加压我提供的 `UnpackTest` 生成一个 `Minke-Unpacktest.exe` 的文件。`Peid` 一扫，结果是

`Borland Delphi 6.0 - 7.0 [Overlay]`//深度扫描和核心扫描均是这个 欺骗我的 `Peid` 签名成功

不管它，直接 `od` 载入：

```
10004020  >  $  55             PUSH EBP                         ;  停在这里
10004021   .  8BEC            MOV EBP,ESP
10004023   .  83C4 F0        ADD ESP,-10
10004026   .  53             PUSH EBX
10004027   .  56             PUSH ESI                         ;  伪造的 Delphi 入口
10004028   .  B8 E83F0010    MOV EAX,Minke-Un.10003FE8
1000402D   .  E8 7AF6FFFF    CALL Minke-Un.100036AC
10004032   .  BE 68660010    MOV ESI,Minke-Un.10006668
10004037   .  33C0           XOR EAX,EAX
10004039   .  55             PUSH EBP
1000403A   .  68 DB400010    PUSH Minke-Un.100040DB
1000403F   .  64:FF30        PUSH DWORD PTR FS:[EAX]
10004042   .  64:8920        MOV DWORD PTR FS:[EAX],ESP
10004045   .  E8 FAF8FFFF    CALL Minke-Un.10003944
1000404A   .  BA EC400010    MOV EDX,Minke-Un.100040EC               ;  ASCII "CA18"
1000404F   .  8BC6           MOV EAX,ESI                       ;  上一个字符串是此壳标示符
10004051   .  E8 F2FAFFFF    CALL Minke-Un.10003B48
10004056   .  8BD8           MOV EBX,EAX
10004058   .  B8 6C660010    MOV EAX,Minke-Un.1000666C
1000405D   .  8B16           MOV EDX,DWORD PTR DS:[ESI]
1000405F   .  E8 88F2FFFF    CALL Minke-Un.100032EC
10004064   .  B8 6C660010    MOV EAX,Minke-Un.1000666C
```

注意到 停在 `1000XXXX` 空间,而不是 `Peid` 扫描得到的 `00004020`，顿生疑惑。不管它，继续 `F8`

跟下去。说一下，这里是一个压缩壳（我还在弄压缩壳，大虾烦指导如何进入加密壳领域），

毫无反追反调。一路 `F8` 下去：

```
100040B0   .  E8 A7E4FFFF    CALL Minke-Un.1000255C
100040B5   .  8BC3           MOV EAX,EBX
100040B7   .  E8 C4FBFFFF    CALL Minke-Un.10003C80                ;  此处 F7
100040BC   .  E8 E7F8FFFF    CALL Minke-Un.100039A8                ;  第一次跟 在这里跑飞
100040C1   .  8BC3           MOV EAX,EBX
100040C3   .  E8 B0E3FFFF    CALL Minke-Un.10002478
100040C8   .  E8 DBF8FFFF    CALL Minke-Un.100039A8
100040CD   .  33C0           XOR EAX,EAX
100040CF   .  5A             POP EDX
100040D0   .  59             POP ECX
100040D1   .  59             POP ECX
100040D2   .  64:8910        MOV DWORD PTR FS:[EAX],EDX
100040D5   .  68 E2400010    PUSH Minke-Un.100040E2
100040DA   >  C3             RETN                         ;  RET 用作跳转到 100040E2
```

第一次追踪在 `100040BC` 处跑飞 所以运行代码在 `100040B7` 处。`Ctrl+F2`，重新来过。在 `100040B7` 处 `F7` 进去

来到这儿：

```
10003C80  /$  55             PUSH EBP                         ;  F7 过来 到了这儿
10003C81  |.  8BEC           MOV EBP,ESP
10003C83  |.  81C4 B8FEFFFF ADD ESP,-148                    ;  好深的栈
10003C89  |.  53             PUSH EBX
10003C8A  |.  56             PUSH ESI
10003C8B  |.  57             PUSH EDI
10003C8C  |.  33D2           XOR EDX,EDX
10003C8E  |.  8995 B8FEFFFF MOV DWORD PTR SS:[EBP-148],EDX
10003C94  |.  8945 FC        MOV DWORD PTR SS:[EBP-4],EAX
10003C97  |.  33C0           XOR EAX,EAX
10003C99  |.  55             PUSH EBP
```

于是进行再次的 `F8` 操作，行不几步，到了这里：

```
10003E44  |.  E8 3FE8FFFF    CALL Minke-Un.10002688
10003E49  |.  8B85 B8FEFFFF MOV EAX,DWORD PTR SS:[EBP-148]
10003E4F  |.  E8 40F4FFFF    CALL Minke-Un.10003294
10003E54  |.  50             PUSH EAX                        ; |CommandLine
10003E55  |.  6A 00          PUSH 0                         ; |ModuleFileName = NULL
10003E57  |.  E8 04F9FFFF    CALL <JMP.&kernel32.CreateProcessA>      ; \CreateProcessA
10003E5C  |.  C785 10FFFFFF > MOV DWORD PTR SS:[EBP-F0],10007
10003E66  |.  8D85 10FFFFFF LEA EAX,DWORD PTR SS:[EBP-F0]
10003E6C  |.  50             PUSH EAX                        ; /pContext
10003E6D  |.  8B85 04FFFFFF MOV EAX,DWORD PTR SS:[EBP-FC]          ; |
10003E73  |.  50             PUSH EAX                        ; |hThread
10003E74  |.  E8 07F9FFFF    CALL <JMP.&kernel32.GetThreadContext>     ; \GetThreadContext
10003E79  |.  E8 2AFBFFFF    CALL Minke-Un.100039A8
10003E7E  |.  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]
10003E81  |.  50             PUSH EAX                        ; /pBytesRead
10003E82  |.  6A 04          PUSH 4                         ; |BytesToRead = 4
10003E84  |.  8D45 F8       LEA EAX,DWORD PTR SS:[EBP-8]           ; |
10003E87  |.  50             PUSH EAX                        ; |Buffer
10003E88  |.  8B45 B4       MOV EAX,DWORD PTR SS:[EBP-4C]          ; |
10003E8B  |.  83C0 08       ADD EAX,8                       ; |
10003E8E  |.  50             PUSH EAX                        ; |pBaseAddress
10003E8F  |.  8B85 00FFFFFF MOV EAX,DWORD PTR SS:[EBP-100]         ; |
10003E95  |.  50             PUSH EAX                        ; |hProcess
10003E96  |.  E8 FDF8FFFF    CALL <JMP.&kernel32.ReadProcessMemory>   ; \ReadProcessMemory
```

此处看来不妙，难道是进程检测？继续跟踪

```
10003E9D  |.  68 00300000    PUSH 3000
10003EA2  |.  8B45 EC       MOV EAX,DWORD PTR SS:[EBP-14]
10003EA5  |.  50             PUSH EAX
10003EA6  |.  8B45 E0       MOV EAX,DWORD PTR SS:[EBP-20]
10003EA9  |.  8B40 34       MOV EAX,DWORD PTR DS:[EAX+34]          ;  此处 D eax
10003EAC  |.  50             PUSH EAX
10003EAD  |.  8B85 00FFFFFF MOV EAX,DWORD PTR SS:[EBP-100]
10003EB3  |.  50             PUSH EAX
10003EB4  |.  E8 FFF8FFFF    CALL <JMP.&kernel32.VirtualAllocEx>
10003EB9  |.  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]
```

在 `10003EA9` 处 `d eax`，可以看到内存指向的地址

```
008DA0F4  50 45 00 00 4C 01 06 00 43 D3 40 4B 00 00 00 00  PE..L.CK....
008DA104  00 00 00 00 E0 00 0E 01 0B 01 06 00 00 40 01 00  ....?..@.
008DA114  00 50 00 00 00 00 00 00 30 1B 00 00 00 10 00 00  .P......0.....
008DA124  00 10 00 00 00 00 40 00 00 10 00 00 00 10 00 00  .....@.......
008DA134  04 00 00 00 00 00 00 00 04 00 00 00 00 00 00 00  ................
008DA144  00 A0 01 00 00 10 00 00 00 00 00 00 02 00 00 00  .?...........
008DA154  00 00 10 00 00 10 00 00 00 00 10 00 00 10 00 00  ............
008DA164  00 00 00 00 10 00 00 00 00 00 00 00 00 00 00 00  ...............
008DA174  00 70 01 00 64 00 00 00 00 80 01 00 A6 0C 00 00  .p.d.....?..
008DA184  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
008DA194  00 90 01 00 50 02 00 00 00 50 01 00 1C 00 00 00  .?.P...P....
008DA1A4  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
008DA1B4  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
008DA1C4  00 00 00 00 00 00 00 00 54 73 01 00 F0 02 00 00  ........Ts.?..
008DA1D4  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
008DA1E4  00 00 00 00 00 00 00 00 2E 74 65 78 74 00 00 00  .......text...
008DA1F4  70 30 01 00 00 10 00 00 00 40 01 00 00 10 00 00  p0.....@....
008DA204  00 00 00 00 00 00 00 00 00 00 00 00 20 00 00 60  ............  ..\`
008DA214  2E 72 64 61 74 61 00 00 C9 05 00 00 00 50 01 00  .rdata..?...P.
008DA224  00 10 00 00 00 50 01 00 00 00 00 00 00 00 00 00  ....P.........
008DA234  00 00 00 00 40 00 00 40 2E 64 61 74 61 00 00 00  ....@..@.data...
008DA244  14 09 00 00 00 60 01 00 00 10 00 00 00 60 01 00  ....\`.....\`.
```



`008DA254 00 00 00 00 00 00 00 00 00 00 00 00 40 00 00 C0`

这里实质上就是我们要 `Dump` 的文件的 `PE` 部分头。往前翻一翻转存窗口就能看见：

`008DA014 4D 5A 90 00 03 00 00 00 04 00 00 00 FF FF 00 00 MZ?........`
`008DA024 B8 00 00 00 00 00 00 00 40 00 00 00 00 00 00 00 ?......@.......`
`008DA034 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ................`
`008DA044 00 00 00 00 00 00 00 00 00 00 00 00 E0 00 00 00 ............?..`
`008DA054 0E 1F BA 0E 00 B4 09 CD 21 B8 01 4C CD 21 54 68 ?.???L?Th`
`008DA064 69 73 20 70 72 6F 67 72 61 6D 20 63 61 6E 6E 6F is program canno`
`008DA074 74 20 62 65 20 72 75 6E 20 69 6E 20 44 4F 53 20 t be run in DOS`
`008DA084 6D 6F 64 65 2E 0D 0D 0A 24 00 00 00 00 00 00 00 mode....$.......`
`008DA094 85 46 CE 8E C1 27 A0 DD C1 27 A0 DD C1 27 A0 DD ???`
`008DA0A4 A3 38 B3 DD C5 27 A0 DD 42 3B AE DD C0 27 A0 DD ?齿?B;?`
`008DA0B4 AE 38 AA DD CA 27 A0 DD AE 38 A4 DD C3 27 A0 DD ???ぽ?`
`008DA0C4 F7 01 A4 DD C2 27 A0 DD C1 27 A1 DD B8 27 A0 DD ?ぽ??≥?`
`008DA0D4 F7 01 AB DD C7 27 A0 DD 06 21 A6 DD C0 27 A0 DD ??!?`
`008DA0E4 52 69 63 68 C1 27 A0 DD 00 00 00 00 00 00 00 00 Rich?........`

这里是不是很熟悉？应该是生成的新文件的 `DOS` 头。不管了 继续 `F8`

```
10003F02  |.  8B45 E0       MOV EAX,DWORD PTR SS:[EBP-20]
10003F05  |.  8B40 34       MOV EAX,DWORD PTR DS:[EAX+34]
10003F08  |.  8B55 E0       MOV EDX,DWORD PTR SS:[EBP-20]
10003F0B  |.  0342 28       ADD EAX,DWORD PTR DS:[EDX+28]
10003F0E  |.  8945 C0       MOV DWORD PTR SS:[EBP-40],EAX            ;  此处得到 OEP
10003F11  |.  8D85 10FFFFFF LEA EAX,DWORD PTR SS:[EBP-F0]
```

在此处将几段拼接在一起，`OEP` 为 `10003F0E` 处的 `eax` 值 由上述语句可以看到，存入了 `EBP-40`

`F8` 一下，然后看堆栈：

```
EBP-60   > 00000038
EBP-5C   > 00000023
EBP-58   > 00000023
EBP-54   > 0012F72C
EBP-50   > 0012FB08
EBP-4C   > 7FFDE000
EBP-48   > 00150000
EBP-44   > 0012F518
EBP-40   > 00401B30//此处就是 OEP
EBP-3C   > 0012F958
EBP-38   > 7C810705  kernel32.7C810705
EBP-34   > 0000001B
EBP-30   > 00000200
EBP-2C   > 0012FFFC
EBP-28   > 00000023
EBP-24   > 008F4018
EBP-20   > 008DA0F4  ASCII "PE"//pe 头
EBP-1C   > 0090E018
EBP-18   > 000003EC
EBP-14   > 0001A000
EBP-10   > 00001000
EBP-C    > 00000004
EBP-8    > 10000000  ASCII "MZP"//dos 头
EBP-4    > 008DA014
```

继续 `F8`，到 `10003F2B` 处：

```
10003F24  |.  8B85 04FFFFFF MOV EAX,DWORD PTR SS:[EBP-FC]
10003F2A  |.  50             PUSH EAX                              ; /hThread
10003F2B  |.  E8 70F8FFFF    CALL <JMP.&kernel32.ResumeThread>     ; \ResumeThread
10003F30  |.  33C0           XOR EAX,EAX
10003F32  |.  5A             POP EDX
10003F33  |.  59             POP ECX
10003F34  |.  59             POP ECX
10003F35  |.  64:8910        MOV DWORD PTR FS:[EAX],EDX
```

此时在 `cmd` 窗口输入 `tasklist`：

```
Minke-Unpacktest.exe        1628 Console            0       1,820 K//这是加壳后的镜像
Minke-Unpacktest.exe        1632 Console            0         176 K//这个是我们要转存的程序
cmd.exe                     1364 Console            0       2,940 K
conime.exe                   492 Console            0       3,400 K
```

继续 `F8`

```
10003F17  |.  50             PUSH EAX                              ; /pContext
10003F18  |.  8B85 04FFFFFF MOV EAX,DWORD PTR SS:[EBP-FC]         ; |
10003F1E  |.  50             PUSH EAX                              ; |hThread
10003F1F  |.  E8 84F8FFFF    CALL <JMP.&kernel32.SetThreadContext> ; \SetThreadContext
10003F24  |.  8B85 04FFFFFF MOV EAX,DWORD PTR SS:[EBP-FC]
10003F2A  |.  50             PUSH EAX                              ; |hThread
10003F2B  |.  E8 70F8FFFF    CALL <JMP.&kernel32.ResumeThread>     ; \ResumeThread
10003F30  |.  33C0           XOR EAX,EAX                           ;  此处程序运行起来了 是转存的好时机
10003F32  |.  5A             POP EDX
10003F33  |.  59             POP ECX
10003F34  |.  59             POP ECX
```

在 `10003F30` 处打开 `LordPe`，选择基址为 `00400000` 的那个程序，先点“纠正映像大小”，然后转存之，结果“无法抓取内存”。真是岂有此理。如果不点“纠正映像大小”，然后转存，虽然能成功，但是结果什么都不是。

我注意到纠正映像大小之前，映像大小是 `1A0000`，纠正之后，变成了 `27000`。《加密与解密 三》中提到，出现“无法抓取内存”根本原因是对于内存段的保护问题。于是我 `ALt+M` 看 `OD` 的内存段，发现没有 `00400000~00427000` 段，这下更改一途也不行。也请高手指点，如何随意更改某一内存段（读/写/执行）的属性？

既然这样不行，我们就区域脱壳吧，`LordPe` 中在镜像中\[右键\]|\[区域脱壳\]，查看 `00400000` 大小是 `1A000`，发现是 `XRW`。`dump` 出来，改名为 `dump.exe`，运行后，正好就是原始文件。而 `0041A000` 之后有段内存是 `NOACESS` 属性，怪不得 `dump` 不出来。

至此，脱壳完成，运行正常。

--------------------------------------------------------------------------------

【经验总结】

这个壳是我在学习压缩壳的过程中遇到的。开始比较茫然，在看雪和 `Upk` 等一些破解论坛上也找不到它的脱法，所以就在这里写下来，文章比较菜，高手飘过。

此破解过程有相关动画，我明天上传到网盘上。

既然分析完了特征，现在可以总结一下秒脱手段：

```
断点 bpx WriteProcessMemory，shift+F9 两次 F8 五次，看 EAX，就是 OEP
//OEP 可能在修复中有用 如果不是壳上壳 可以不考虑上面一步 直接进行下面的步骤
//注意脱壳段 要包含 OEP

然后 bpx ResumeThread shift+F9 F8 用 lordPE 照文中方法 dump 即可
```

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2010 年 01 月 12 日 1:32:41

上传的附件

| `![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg)` | `pediy.rar` |


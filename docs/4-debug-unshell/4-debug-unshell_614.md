
# Unpacking EncryptPE V2.2005910

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-691.htm`](https://www.pediy.com/kssd/pediy07/pediy7-691.htm)

```
;by Immlep
;www.ptteam.com
;target:EPEunpackme 910.exe
;http://www.ptteam.com/EPEunpackme.rar
```

昨天看到某大侠在群里发 `Loveboom` 的 `epe910` 的脱壳机(后来获悉这个位大侠就是 `loveboom` 本人，打倒 `loveboom`![](img/710b5589904d0d2bf58a67748fca40e1.jpg) ![](img/710b5589904d0d2bf58a67748fca40e1.jpg) ！！).....的图片，痒痒的，不过机归机，脱归脱，上次心血来潮，用 `EPE` 注册版加个 `UnpackMe`，不要问我拿注册版，这是某兄弟在没有玩破解之前话了 `X00` 元 `RMB` 买的，自己的软件倒是没有加，被我用来加 `UnPackMe`，真是.......

话说 `UnpackMe` 是加了，放了两天，都快发霉了，可事情就发生在今天早上，睡过头了！睡过头了？这个脱壳本来是没什么关系的，可是，我睡过头了阿，你不知道啊，今天早上 3、4 有课啊，看看现在是什么时间十点半了，早就上到了第三节了，啊，这还不惨！天又下着雨，去到教室要走十几分钟，去到之后都快上到第四节了，人的惰性，由此而生，`My God`！我又逃课了，但是逃课也要找点事做啊，难道我就这样浪费了两节＋被老师点名的危险＋`6.25*2` 块 `RMB`(按一年 5500 的学费来算的话，一年 40 周，每周 22 节课！总共就 880 节课),天啊，这 `12.5RMB` 可不能浪费啊，但是我看到了桌面上的 `EPE UnpackMe` 的时候，我露出了奸诈的笑容，嘻嘻嘻嘻！！你小子，上次有人拿 `500RMB` 要我做了你，我没干，今天我就要为了 `12.5RMB` 把你做了。

结束掉 `explorer` 进程，用 `OD` 载入 `EPEunpackme 910.exe`，在 `OD` 中忽略“内存反问异常”，`F9` 运行，发生第一次 `INT3` 异常，`Go`，这里我不罗嗦了，我只教你修改代码，具体的自己慢慢去跟踪，第一次异常停下来是修改下面的代码。

```
711E4142       0F95C0                 SETNE AL ;效俭

711E5FD1       0F95C0                 SETNE AL ;效俭

711E4081       8B55 F8                MOV EDX, DWORD PTR [EBP-8]  ;函数处理
```

修改如下

```
711E4142       0F94C0                 SETE AL

711E5FD1       0F94C0                 SETE AL

711E4081       8B55 FC                MOV EDX, DWORD PTR [EBP-4]
```

`711E5AED` 处的代码（修改程序中的函数调用 `JMP` 类）：

```
711E5AED       66:C700 FF25           MOV WORD PTR [EAX], 25FF

711E5AF2       8B55 FC                MOV EDX, DWORD PTR [EBP-4]

711E5AF5       8950 02                MOV DWORD PTR [EAX+2], EDX

711E5AF8       8BD0                   MOV EDX, EAX

711E5AFA       8B5D E8                MOV EBX, DWORD PTR [EBP-18]

711E5AFD       83C3 06                ADD EBX, 6

711E5B00       8B45 E0                MOV EAX, DWORD PTR [EBP-20]                              ; V2200591.711DC6DC

711E5B03       03C0                   ADD EAX, EAX

711E5B05       03D8                   ADD EBX, EAX

711E5B07       8B45 E8                MOV EAX, DWORD PTR [EBP-18]

711E5B0A       8B00                   MOV EAX, DWORD PTR [EAX]

711E5B0C       0FB70B                 MOVZX ECX, WORD PTR [EBX]

711E5B0F       81E9 00300000          SUB ECX, 3000

711E5B15       03C1                   ADD EAX, ECX

711E5B17       0345 DC                ADD EAX, DWORD PTR [EBP-24]

711E5B1A       48                     DEC EAX

711E5B1B       2BD0                   SUB EDX, EAX

711E5B1D       83EA 04                SUB EDX, 4

711E5B20       8910                   MOV DWORD PTR [EAX], EDX
```

修改如下：

```
711E5AED       8B5D E8                MOV EBX, DWORD PTR [EBP-18]

711E5AF0       83C3 06                ADD EBX, 6

711E5AF3       8B45 E0                MOV EAX, DWORD PTR [EBP-20]

711E5AF6       03C0                   ADD EAX, EAX

711E5AF8       03D8                   ADD EBX, EAX

711E5AFA       8B45 E8                MOV EAX, DWORD PTR [EBP-18]

711E5AFD       8B00                   MOV EAX, DWORD PTR [EAX]

711E5AFF       0FB70B                 MOVZX ECX, WORD PTR [EBX]

711E5B02       81E9 00300000          SUB ECX, 3000

711E5B08       03C1                   ADD EAX, ECX

711E5B0A       0345 DC                ADD EAX, DWORD PTR [EBP-24]

711E5B0D       83E8 02                SUB EAX, 2

711E5B10       66:C700 FF25           MOV WORD PTR [EAX], 25FF

711E5B15       8B55 FC                MOV EDX, DWORD PTR [EBP-4]

711E5B18       8950 02                MOV DWORD PTR [EAX+2], EDX

711E5B1B       90                     NOP

711E5B1C       90                     NOP

711E5B1D       90                     NOP

711E5B1E       90                     NOP

711E5B1F       90                     NOP

711E5B20       90                     NOP

711E5B21       90                     NOP
```

十六进制代码为：

```
8B 5D E8 83 C3 06 8B 45 E0 03 C0 03 D8 8B 45 E8 8B 00 0F B7 0B 81 E9 00 30 00 00 03 C1 03 45 DC
83 E8 02 66 C7 00 FF 25 8B 55 FC 89 50 02 90 90 90 90 90 90 90
```

`711E5B2C` 处的代码（修改程序中的函数调用 `JMP` 类和 `CALL`）：

```
711E5B2C       66:C700 FF25           MOV WORD PTR [EAX], 25FF

711E5B31       8B55 FC                MOV EDX, DWORD PTR [EBP-4]

711E5B34       8950 02                MOV DWORD PTR [EAX+2], EDX

711E5B37       8BD0                   MOV EDX, EAX

711E5B39       8B5D E8                MOV EBX, DWORD PTR [EBP-18]

711E5B3C       83C3 06                ADD EBX, 6

711E5B3F       8B45 E0                MOV EAX, DWORD PTR [EBP-20]                              ; V2200591.711DC6DC

711E5B42       03C0                   ADD EAX, EAX

711E5B44       03D8                   ADD EBX, EAX

711E5B46       8B45 E8                MOV EAX, DWORD PTR [EBP-18]

711E5B49       8B00                   MOV EAX, DWORD PTR [EAX]

711E5B4B       0FB70B                 MOVZX ECX, WORD PTR [EBX]

711E5B4E       81E9 00300000          SUB ECX, 3000

711E5B54       03C1                   ADD EAX, ECX

711E5B56       0345 DC                ADD EAX, DWORD PTR [EBP-24]

711E5B59       2BD0                   SUB EDX, EAX

711E5B5B       83EA 04                SUB EDX, 4

711E5B5E       8910                   MOV DWORD PTR [EAX], EDX
```

修改为：

```
711E5B2C       90                     NOP

711E5B2D       90                     NOP

711E5B2E       90                     NOP

711E5B2F       90                     NOP

711E5B30       90                     NOP

711E5B31       90                     NOP

711E5B32       90                     NOP

711E5B33       90                     NOP

711E5B34       90                     NOP

711E5B35       90                     NOP

711E5B36       90                     NOP

711E5B37       90                     NOP

711E5B38       90                     NOP

711E5B39       8B5D E8                MOV EBX, DWORD PTR [EBP-18]

711E5B3C       83C3 06                ADD EBX, 6

711E5B3F       8B45 E0                MOV EAX, DWORD PTR [EBP-20]                              ; V2200591.711DC6DC

711E5B42       03C0                   ADD EAX, EAX

711E5B44       03D8                   ADD EBX, EAX

711E5B46       8B45 E8                MOV EAX, DWORD PTR [EBP-18]

711E5B49       8B00                   MOV EAX, DWORD PTR [EAX]

711E5B4B       0FB70B                 MOVZX ECX, WORD PTR [EBX]

711E5B4E       81E9 00300000          SUB ECX, 3000

711E5B54       03C1                   ADD EAX, ECX

711E5B56       0345 DC                ADD EAX, DWORD PTR [EBP-24]

711E5B59      - E9 B0C4288F            JMP EPEunpac.0047200E          ；代码空间不足，所以我找到程序入口点下面的地址 0047200E 来写入代码

711E5B5E       90                     NOP

711E5B5F       90                     NOP
```

十六进制代码为：

```
90 90 90 90 90 90 90 90 90 90 90 90 90 8B 5D E8 83 C3 06 8B 45 E0 03 C0 03 D8 8B 45 E8 8B 00 0F
B7 0B 81 E9 00 30 00 00 03 C1 03 45 DC E9 B0 C4 28 8F 90 90
```

`0047200E` 处写入补丁代码：

```
0047200E       8078 FF E8             CMP BYTE PTR [EAX-1], 0E8

00472012       75 10                  JNZ SHORT EPEunpac.00472024

00472014       83E8 02                SUB EAX, 2

00472017       66:C700 FF15           MOV WORD PTR [EAX], 15FF

0047201C       8B55 FC                MOV EDX, DWORD PTR [EBP-4]

0047201F       8950 02                MOV DWORD PTR [EAX+2], EDX

00472022       EB 0E                  JMP SHORT EPEunpac.00472032

00472024       83E8 02                SUB EAX, 2

00472027       66:C700 FF25           MOV WORD PTR [EAX], 25FF
```



```
0047202C       8B55 FC                MOV EDX, DWORD PTR [EBP-4]
0047202F       8950 02               MOV DWORD PTR [EAX+2], EDX
00472032     - E9 273BD770           JMP V2200591.711E5B5E          ；跳回到 711E5B5E 执行。
```

十六进制代码：

```
80 78 FF E8 75 10 83 E8 02 66 C7 00 FF 15 8B 55 FC 89 50 02 EB 0E 83 E8 02 66 C7 00 FF 25 8B 55
FC 89 50 02 E9 27 3B D7 70
```

OK 修改完毕，`Shift`＋`F9` 忽略异常，在最后一次异常是程序修改 `EIP` 到 `OEP`，这里我算了一下，我这里是十六次异常。

```
0012FF98   0012FFE0  Pointer to next SEH record
0012FF9C   711DADBD  SE handler

711DADBD       53                    PUSH EBX
711DADBE       52                    PUSH EDX
711DADBF       8B5C24 14            MOV EBX, DWORD PTR [ESP+14]
711DADC3       8B93 C4000000        MOV EDX, DWORD PTR [EBX+C4]
711DADC9       8B83 C0000000        MOV EAX, DWORD PTR [EBX+C0]
711DADCF       A3 F4162071          MOV DWORD PTR [712016F4], EAX
711DADD4       E8 DF060000          CALL V2200591.711DB4B8
711DADD9       9C                   PUSHFD
711DADDA       58                   POP EAX
711DADDB       A3 F4162071          MOV DWORD PTR [712016F4], EAX
711DADE0       E8 D3060000          CALL V2200591.711DB4B8
711DADE5       8B83 B8000000        MOV EAX, DWORD PTR [EBX+B8]
711DADEB       40                   INC EAX
711DADEC       8983 B8000000        MOV DWORD PTR [EBX+B8], EAX
711DADF2       8B4424 0C            MOV EAX, DWORD PTR [ESP+C]
711DADF6       8B00                 MOV EAX, DWORD PTR [EAX]
711DADF8       3D 03000080          CMP EAX, 80000003
711DADFD       75 71                JNZ SHORT V2200591.711DAE70
711DADFF       803D 21172071 01     CMP BYTE PTR [71201721], 1
711DAE06       74 4F               JE SHORT V2200591.711DAE57
711DAE08       8B42 0C             MOV EAX, DWORD PTR [EDX+C]
711DAE0B       8983 9C000000       MOV DWORD PTR [EBX+9C], EAX
711DAE11       8B42 10             MOV EAX, DWORD PTR [EDX+10]
711DAE14       8983 A0000000       MOV DWORD PTR [EBX+A0], EAX
711DAE1A       8B42 14             MOV EAX, DWORD PTR [EDX+14]
711DAE1D       8983 B4000000       MOV DWORD PTR [EBX+B4], EAX
711DAE23       8B42 1C             MOV EAX, DWORD PTR [EDX+1C]
711DAE26       8983 A4000000       MOV DWORD PTR [EBX+A4], EAX
711DAE2C       8B42 20             MOV EAX, DWORD PTR [EDX+20]
711DAE2F       8983 A8000000       MOV DWORD PTR [EBX+A8], EAX
711DAE35       8B42 24             MOV EAX, DWORD PTR [EDX+24]
711DAE38       8983 AC000000       MOV DWORD PTR [EBX+AC], EAX
711DAE3E       8B42 28             MOV EAX, DWORD PTR [EDX+28]
711DAE41       8983 B0000000       MOV DWORD PTR [EBX+B0], EAX    ；最后一次异常时 EAX 的就是 OEP 了。
711DAE47       8B02                 MOV EAX, DWORD PTR [EDX]
711DAE49       8942 24             MOV DWORD PTR [EDX+24], EAX
711DAE4C       89D0                 MOV EAX, EDX
711DAE4E       83C0 24             ADD EAX, 24
711DAE51       8983 C4000000       MOV DWORD PTR [EBX+C4], EAX
711DAE57       31C0                 XOR EAX, EAX
711DAE59       8943 04             MOV DWORD PTR [EBX+4], EAX
711DAE5C       8943 08             MOV DWORD PTR [EBX+8], EAX
711DAE5F       8943 0C             MOV DWORD PTR [EBX+C], EAX
711DAE62       8943 10             MOV DWORD PTR [EBX+10], EAX
711DAE65       C743 18 55010000   MOV DWORD PTR [EBX+18], 155
711DAE6C       5A                   POP EDX
711DAE6D       5B                   POP EBX
711DAE6E       C3                   RETN
```

找到 `OEP` 后，在 `OEP` 下断点，`F9` 中断在 `OEP` 处，`Dump`，用 `IMPREC` 修复输入表，搞定,忙着，好像不能跨平台，kao～最近怎么脱壳都有问题，不知道是我的 `RPWT` 还是我的机子的 `JPWT` 还是我的 `XP` 的 `XPWT`，哎，算了，还是回家吧。

注：`ImpRec` 可能看不到 `EPE` 的进程，因为 `EPE` 修改了 `ZwOpenProcess` 处的代码，临时的办法就是所以用 `OD` 载入 `ImpRec`，把 `ZwOpenProcess` 入口处被修改的代码改回去，这样在 `ImpRec` 就可以看到进程了。

PS:其实跟以前得 s 版差不多。。![](img/710b5589904d0d2bf58a67748fca40e1.jpg) ![](img/710b5589904d0d2bf58a67748fca40e1.jpg) ![](img/710b5589904d0d2bf58a67748fca40e1.jpg)

`4dd466d0e1e0cc123afbe504da2462ca`

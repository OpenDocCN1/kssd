
# cad2002 中文网络版

> 原文：[`www.pediy.com/kssd/pediy06/pediy6802.htm`](https://www.pediy.com/kssd/pediy06/pediy6802.htm)

**软件名称：** `AUTOcad2002` 中文版网络版
**软件类别：** 地球人都知道
**软件介绍：** 地球人都知道 `Flexlm7.1f` 加密
**破解工具：** `ollydbg` `1.09d`，`w32Dasm_2002828_pll621`，`C32ASM_pll621`，`UltraEdit8.0`
**破解目的：** 无限制使用（爆破）

`AUTOcadR14` 中文版大家都很熟悉了吧，不多说。我从上海回来，又开始画图了，一年没干活，单位的软件都变成正版了，哈哈。可是 `AUTOcadR14` 中文版是网络版，我们只买 20 个点，一个点多少钱不知道（我们还买了 `cad2002` 网络版是 20 个点，一个点 10000 多元啊！），想来不少。因此，我上晚了点数满了，就会进不去，提示网络不许可，TMD！看看 `cad2002` 是 `Flexlm` 加密的，我功力太浅，那我就先拿 `R14` 开刀！

上面是我的 `AUTOcadR14` 中文版破文里的。`R14` 之后，现在我终于向 `cad2002` 开刀了。

`CAD2002`，`Flexlm7.1f` 加密，我想也能爆破吧，我没有能力做他的无限制 `license`。也因为，我觉得作了也没用，为什么？你看，一个单位买一套网络版，就是自己单位专用的，上面都有安装序列号和单位信息，就是作了无限制 `license`，自己也不敢用，怕被查，给别人用那就更不敢了，也不能啊。当然做无限制 `license` 的方法应该是可以放出的，但 `CAD2002` 加密不使用标准的 `Flexlm` 的 `Dll`，它的 `lc_checkout` 断点就找不到了，我的功力太浅，这里先抛砖引玉吧，其实我也隐约找到了一些有用地方，留待以后吧。

爆破在这里也是一种破解，对吧？开工。

先反汇编 `acad.EXE`,找找可疑点。好长时间啊，我的机子是 `P4 1.6`，用了 10 多分钟。大略看看，没什么有用的。但还是没白费。因为我按我上篇破文的思路，找到了一些提示，对我下断点有用。

还是用 `Ollydbg1.09d`。这次我们先不拔掉网线（我们早知拔掉网线就肯定不能通过 `cad` 验证了）。幸亏是 `P4`，进去了。设什么断点呢？不知道，我倒！

因为我们上次就知道拔掉网线后，它还要去网络验证，可是没有网络，它会多试几次，这就给了我们时间，我们运行它，看它在那里有短暂的停顿，那里就是验证的地方！而这次我们不拔掉网线！为什么？反汇编还是有用的，这次我不看运气了。我们找到了许多和 `AUTOcadR14.01` 中文版相似的地方，就在那附近设断。只要在 `Flex license` 窗口出来之前中断，我们就可以了，当然离关键地方越近越好了。

`Ollydbg` 设断很方便的，其实我差不多一次就中断成功！

通过分析反汇编，断点设在 `005BCB5A`，因为我有 `R14` 的经验，其实也是误打误撞啦，哈哈。

我们来到下面这个 `CALL`,调试中多少次重启动，我记不清了，`Ollydbg` 好像还是不太稳定，美中不足！也可能是我开的程序窗口太多，`Ollydbg` 就很站地方了，还有 `w32Dasm`，`UltraEdit8.0`，计算器，`128M` 内存吃不消啦。对了还有开了 `pll621` 的 `C32ASM`，他的查询机器码的东东很好，虽然 `Ollydbg` 也有，但是麻烦。别的还没来得及用，以后慢慢来啦。

```
005BCB10  /$ 55             PUSH EBP
005BCB11  |. 8BEC           MOV EBP,ESP
005BCB13  |. 51             PUSH ECX
005BCB14  |. 8B15 1C67AC00  MOV EDX,DWORD PTR DS:[AC671C]
005BCB1A  |. A1 1867AC00    MOV EAX,DWORD PTR DS:[AC6718]
005BCB1F  |. 83C2 3B        ADD EDX,3B
005BCB22  |. 56             PUSH ESI
005BCB23  |. 8915 1C67AC00  MOV DWORD PTR DS:[AC671C],EDX
005BCB29  |. 8B88 EC000000  MOV ECX,DWORD PTR DS:[EAX+EC]
005BCB2F  |. BE A02EAC00    MOV ESI,ACAD.00AC2EA0
005BCB34  |. 8B11           MOV EDX,DWORD PTR DS:[ECX]
005BCB36  |. 81EE 94000000  SUB ESI,94
005BCB3C  |. 81F2 A9B50000  XOR EDX,0B5A9
005BCB42  |. 83C2 3B        ADD EDX,3B
005BCB45  |. 8915 1C67AC00  MOV DWORD PTR DS:[AC671C],EDX
005BCB4B  |. 8B80 EC000000  MOV EAX,DWORD PTR DS:[EAX+EC]
005BCB51  |. 8338 00        CMP DWORD PTR DS:[EAX],0
005BCB54  |. 0F85 A5000000  JNZ ACAD.005BCBFF
005BCB5A  |. 6A 00          PUSH 0                   我们在这里 break！！设断点
005BCB5C  |. E8 EFF5FFFF    CALL ACAD.005BC150       ；进去看看（注：我在这里白费了心思，这是后话了）
```

我们 F7 进去一看

```
005BCB61  |. 35 A9B50000    XOR EAX,0B5A9          看到 b5a9，我们在上面的 call 前这里设断吧
005BCB66  |. 83C4 04        ADD ESP,4
005BCB69  |. 3D 564AFFFF    CMP EAX,FFFF4A56       看到 FFFF4A56，我们在上面的 call 前这里设断吧
005BCB6E  |. 0F84 8B000000  JE ACAD.005BCBFF       ；和 cadR14 的差不多吧
005BCB74  |. 3D 574AFFFF    CMP EAX,FFFF4A57
005BCB79  |. 66:C705 F067AC>MOV WORD PTR DS:[AC67F0],0
005BCB82  |. 75 07          JNZ SHORT ACAD.005BCB8B
005BCB84  |. 32C0           XOR AL,AL
005BCB86  |. 5E             POP ESI
005BCB87  |. 8BE5           MOV ESP,EBP
005BCB89  |. 5D             POP EBP
005BCB8A  |. C3             RETN
```

`****` 我们发现无论是否拔掉网线，这都会通过，所以 F8 继续走下去吧。

`---------------------------------`

进去看看 `005BCB5C` `E8 EFF5FFFF` `CALL ACAD.005BC150`

`* Referenced by a CALL at Address:`

`|:005BCB5C    `

`|`

`:005BC150 56                 push esi`
`:005BC151 E87A2C0000         call 005BEDD0`
`:005BC156 8BF0               mov esi, eax`
`:005BC158 56                 push esi`
`:005BC159 E8A2200000         call 005BE200`
`:005BC15E 83C404             add esp, 00000004`
`:005BC161 85C0               test eax, eax`
`:005BC163 7405               je 005BC16A`
`:005BC165 83C8FF             or eax, FFFFFFFF       ；和 cadR14 很相似啊，激动啊，以为找到那`
`:005BC168 5E                 pop esi`
`:005BC169 C3                 ret`

我正是看到这些，才在前面下断点，总之，中断在验证之前就是胜利

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`|:005BC163(C)`

`|`

`:005BC16A B91019B000         mov ecx, 00B01910`

`* Reference To: ACAD.?entity@AcEdJig@@UBEPAVAcDbEntity@@XZ`

`                            |`

`:005BC16F E8AC0A1D00         call 0078CC20`
`:005BC174 85C0               test eax, eax`
`:005BC176 7405               je 005BC17D`
`:005BC178 83C8FF             or eax, FFFFFFFF`
`:005BC17B 5E                 pop esi`
`:005BC17C C3                 ret`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`|:005BC176(C)`

`|`

`:005BC17D 83FE11             cmp esi, 00000011`
`:005BC180 B808000100         mov eax, 00010008`
`:005BC185 7507              jne 005BC18E`
`:005BC187 B801000200        mov eax, 00020001`
`......`
`......省略`
`......`
`:005BC1EA 35903F0000        xor eax, 00003F90`
`:005BC1EF 5E                 pop esi`
`:005BC1F0 C3                 ret      ；返回以后知道，这是个摆设，`

无论是否联网，都返回一样的 `eax`。

可能 `autodesk` 公司原来是用自己的加密，后来弃用了，TMD。

`============================================`

下面继续走

```
005BCB8B  |> E8 40220000    CALL ACAD.005BEDD0
005BCB90  |. 83F8 2A        CMP EAX,2A
005BCB93  |. 75 1E          JNZ SHORT ACAD.005BCBB3
005BCB95  |. 8B0D 1867AC00  MOV ECX,DWORD PTR DS:[AC6718]                 ACAD.00AC69E4
005BCB9B  |. C705 5C19B000 >MOV DWORD PTR DS:[B0195C],3B
005BCBA5  |. 8B91 EC000000  MOV EDX,DWORD PTR DS:[ECX+EC]
005BCBAB  |. C702 04000000  MOV DWORD PTR DS:[EDX],4
005BCBB1  |. EB 2F          JMP SHORT ACAD.005BCBE2
005BCBB3  |> E8 18220000    CALL ACAD.005BEDD0
005BCBB8  |. 83F8 11        CMP EAX,11
005BCBBB  |. 74 0A          JE SHORT ACAD.005BCBC7
......
......省略
......
005BCC9E  |. A1 5C19B000    MOV EAX,DWORD PTR DS:[B0195C]
005BCCA3  |. 83C0 C5        ADD EAX,-3B
005BCCA6  |. 8915 5019B000 MOV DWORD PTR DS:[B01950],EDX
005BCCAC  |. F7D8           NEG EAX
005BCCAE  |. 1BC0           SBB EAX,EAX
005BCCB0  |. 25 AD940000    AND EAX,94AD
005BCCB5  |. 05 A9B5FFFF    ADD EAX,FFFFB5A9
005BCCBA  |. 35 A9B5FFFF    XOR EAX,FFFFB5A9
005BCCBF  |. 66:8986 940000>MOV WORD PTR DS:[ESI+94],AX
005BCCC6  |. B0 01          MOV AL,1
005BCCC8  |. 5E             POP ESI
005BCCC9  |. 8BE5           MOV ESP,EBP
005BCCCB  |. 5D             POP EBP
005BCCCC  |\. C3             RETN          到这里返回 ret 5bce69
```

按 F8 走到这里，当然中间也 F7 进去看看了，浪费了不少经历。

`----------------------------------------`

上面返回到下面的 `005BCE69` `CALL ACAD.005BCB10`

```
005BCD90  /$ 55             PUSH EBP
005BCD91  |. 8BEC           MOV EBP,ESP
005BCD93  |. 81EC 7C030000  SUB ESP,37C
005BCD99  |. 56             PUSH ESI
005BCD9A  |. 57             PUSH EDI
005BCD9B  |. C745 FC 000000>MOV DWORD PTR SS:[EBP-4],0
005BCDA2  |. E8 E9D51F00    CALL ACAD.007BA390
005BCDA7  |. 8D45 FC        LEA EAX,DWORD PTR SS:[EBP-4]
005BCDAA  |. 8D4D F0        LEA ECX,DWORD PTR SS:[EBP-10]
005BCDAD  |. 50             PUSH EAX
005BCDAE  |. 51             PUSH ECX
005BCDAF  |. 68 E80FAC00    PUSH ACAD.00AC0FE8                     ASCII "rb"
005BCDB4  |. 68 7068AC00    PUSH ACAD.00AC6870                     ASCII "xmx"
005BCDB9  |. 68 E065AC00    PUSH ACAD.00AC65E0                     ASCII "acad"
```

从我们的断点返回往上看看，看到了什么 `acad.xmx`，这是 `acad` 特有的文件，

叫什么“AutoCAD 外部信息文件”，看来找对地方了。因为 `cad` 开始启动先调这个文件。

```
005BCDBE  |. E8 2D791200    CALL ACAD.006E46F0
005BCDC3  |. 50             PUSH EAX                                 |Arg2
005BCDC4  |. 6A 02          PUSH 2                                   |Arg1 = 00000002
005BCDC6  |. E8 5558F5FF    CALL ACAD.00512620                       \ACAD.00512620
005BCDCB  |. 83C4 1C        ADD ESP,1C
```



005BCDCE |. 85C0           TEST EAX,EAX

005BCDD0 |. 75 4C          JNZ SHORT ACAD.005BCE1E

005BCDD2 |. 8B7D FC        MOV EDI,DWORD PTR SS:\[EBP-4]

005BCDD5 |. 83C9 FF        OR ECX,FFFFFFFF

005BCDD8 |. F2:AE         REPNE SCAS BYTE PTR ES:\[EDI]

005BCDDA |. F7D1          NOT ECX

005BCDDC |. 2BF9          SUB EDI,ECX

005BCDDE |. 8D95 84FEFFFF LEA EDX,DWORD PTR SS:\[EBP-17C]

005BCDE4 |. 8BC1          MOV EAX,ECX

005BCDE6 |. 8BF7          MOV ESI,EDI

005BCDE8 |. 8BFA          MOV EDI,EDX

005BCDEA |. C1E9 02       SHR ECX,2

005BCDED |. F3:A5         REP MOVS DWORD PTR ES:\[EDI],DWORD PTR DS:

005BCDEF |. 8BC8          MOV ECX,EAX

005BCDF1 |. 83E1 03       AND ECX,3

005BCDF4 |. F3:A4         REP MOVS BYTE PTR ES:\[EDI],BYTE PTR DS:\[>

005BCDF6 |. 8D4D FC       LEA ECX,DWORD PTR SS:\[EBP-4]

005BCDF9 |. 51            PUSH ECX

005BCDFA |. E8 092A3300   CALL

005BCDFF |. 8B55 F0       MOV EDX,DWORD PTR SS:\[EBP-10]

005BCE02 |. 52            PUSH EDX

005BCE03 |. E8 AA2B3300   CALL

005BCE08 |. 8D85 84FEFFFF LEA EAX,DWORD PTR SS:\[EBP-17C]

005BCE0E |. 68 70027B00   PUSH ACAD.007B0270 |Arg2 = 007B0270

005BCE13 |. 50            PUSH EAX |Arg1

005BCE14 |. E8 979F2E00   CALL ACAD.008A6DB0 \\ACAD.008A6DB0

005BCE19 |. 83C4 10       ADD ESP,10

005BCE1C |. EB 21         JMP SHORT ACAD.005BCE3F

005BCE1E |> 68 E065AC00   PUSH ACAD.00AC65E0 ASCII "acad"

005BCE23 |. 6A 05         PUSH 5

005BCE25 |. 8D4D 88       LEA ECX,DWORD PTR SS:\[EBP-78]

005BCE28 |. 68 4C68AC00   PUSH ACAD.00AC684C ASCII "Can't find message file %.\*s.xmx."

005BCE2D |. 51            PUSH ECX

005BCE2E |. E8 1B293300   CALL

005BCE33 |. 8D55 88       LEA EDX,DWORD PTR SS:\[EBP-78]

005BCE36 |. 52            PUSH EDX |Arg1

005BCE37 |. E8 34341F00   CALL ACAD.007B0270 \\ACAD.007B0270

005BCE3C |. 83C4 14       ADD ESP,14

005BCE3F |> E8 8C020000   CALL ACAD.005BD0D0

005BCE44 |. A1 24FAAF00   MOV EAX,DWORD PTR DS:\[AFFA24]

005BCE49 |. 85C0          TEST EAX,EAX

005BCE4B |. 75 0A         JNZ SHORT ACAD.005BCE57

005BCE4D |. E8 7EDDF0FF    CALL ACAD.004CABD0

005BCE52 |. A3 24FAAF00   MOV DWORD PTR DS:\[AFFA24],EAX

005BCE57 |> 68 BCA6B500   PUSH ACAD.00B5A6BC

005BCE5C |. 68 BAA6B500   PUSH ACAD.00B5A6BA

005BCE61 |. E8 0ACD1F00   CALL ACAD.007B9B70

005BCE66 |. 83C4 08       ADD ESP,8

005BCE69 |. E8 A2FCFFFF    CALL ACAD.005BCB10 这里从我们的断点走过来的 ret from 005BCCCC，
我们接着走下去， 接着 F8，走

005BCE6E |. 84C0          TEST AL,AL al=1

005BCE70 |. 75 08         JNZ SHORT ACAD.005BCE7A

005BCE72 |. 5F            POP EDI

005BCE73 |. 33C0          XOR EAX,EAX

005BCE75 |. 5E            POP ESI

005BCE76 |. 8BE5          MOV ESP,EBP

005BCE78 |. 5D            POP EBP

005BCE79 |. C3            RETN

005BCE7A |> A0 3C19B000   MOV AL,BYTE PTR DS:\[B0193C]

005BCE7F |. 84C0          TEST AL,AL al=0

005BCE81 |. 75 3E         JNZ SHORT ACAD.005BCEC1

005BCE83 |. 8D45 F4       LEA EAX,DWORD PTR SS:\[EBP-C]

005BCE86 |. 8D4D F8       LEA ECX,DWORD PTR SS:\[EBP-8]

005BCE89 |. 50            PUSH EAX

005BCE8A |. 8D55 EC       LEA EDX,DWORD PTR SS:\[EBP-14]

005BCE8D |. 51            PUSH ECX

005BCE8E |. 52            PUSH EDX

005BCE8F |. 68 4068AC00   PUSH ACAD.00AC6840 ASCII "%d/%d/%d"

005BCE94 |. 68 1046AC00   PUSH ACAD.00AC4610 ASCII "junkjunkjunkjunk"

看上面到了吗，垃圾垃圾什么意思啊？！变态

005BCE99 |. E8 FC2A3300   CALL

005BCE9E |. 83C4 14       ADD ESP,14

005BCEA1 |. 83F8 03       CMP EAX,3

005BCEA4 |. 75 1B         JNZ SHORT ACAD.005BCEC1

005BCEA6 |. 8B45 EC       MOV EAX,DWORD PTR SS:\[EBP-14]

005BCEA9 |. 8B4D F8       MOV ECX,DWORD PTR SS:\[EBP-8]

......

......省略

......

005BD06B |. 83CE FF       OR ESI,FFFFFFFF

005BD06E |. 83F8 03       CMP EAX,3

005BD071 |. B8 1068AC00   MOV EAX,ACAD.00AC6810 ASCII "MAXOBJMEM"

005BD076 |. 74 05         JE SHORT ACAD.005BD07D

005BD078 |. B8 0068AC00   MOV EAX,ACAD.00AC6800 ASCII "ACADMAXOBJMEM"

005BD07D |> 50            PUSH EAX /Arg1

005BD07E |. E8 9DD61F00   CALL ACAD.007BA720 \\ACAD.007BA720

005BD083 |. 83C4 04       ADD ESP,4

005BD086 |. 85C0          TEST EAX,EAX

005BD088 |. 74 08         JE SHORT ACAD.005BD092

005BD08A |. 50            PUSH EAX

005BD08B |. FFD7          CALL EDI

005BD08D |. 83C4 04       ADD ESP,4

005BD090 |. 8BF0          MOV ESI,EAX

005BD092 |> 8B0D 6C869E00 MOV ECX,DWORD PTR DS:\[<&acdb15.maxObjMem>; ACDB15.maxObjMem

005BD098 |. 33C0          XOR EAX,EAX

005BD09A |. 85F6          TEST ESI,ESI

005BD09C |. 0F9CC0        SETL AL

005BD09F |. 48            DEC EAX

005BD0A0 |. 68 BCA6B500   PUSH ACAD.00B5A6BC

005BD0A5 |. 23C6          AND EAX,ESI

005BD0A7 |. 68 BAA6B500   PUSH ACAD.00B5A6BA

005BD0AC |. 8901          MOV DWORD PTR DS:\[ECX],EAX

005BD0AE |. E8 BDCA1F00   CALL ACAD.007B9B70

005BD0B3 |. A0 48C4AF00   MOV AL,BYTE PTR DS:\[AFC448]

005BD0B8 |. 83C4 08       ADD ESP,8

005BD0BB |. 24 07         AND AL,7

005BD0BD |. F6D8          NEG AL

005BD0BF |. 1BC0          SBB EAX,EAX

005BD0C1 |. 5F            POP EDI

005BD0C2 |. 40            INC EAX

005BD0C3 |. 5E            POP ESI

005BD0C4 |. 8BE5          MOV ESP,EBP

005BD0C6 |. 5D            POP EBP

005BD0C7 |\\. C3          RETN 返回 ret 6d80ea，我们接着走

----------------------------------------------------------

上面又返回到下面的 `006D80EA CALL ACAD.005BCD90`，好累啊。

006D8022 . 6A 01          PUSH 1

006D8024 . E8 17232B00    CALL

006D8029 . 8B83 38010000  MOV EAX,DWORD PTR DS:\[EBX+138]

006D802F . 8B50 20        MOV EDX,DWORD PTR DS:\[EAX+20]

006D8032 . 52             PUSH EDX /hWnd

006D8033 . FF15 2C639E00 CALL DWORD PTR DS:\[<&USER32.UpdateW>; \\UpdateWindow

006D8039 > 8BCB           MOV ECX,EBX

006D803B . E8 00320000    CALL ACAD.006DB240

006D8040 . E8 BBE60000    CALL ACAD.006E6700

006D8045 . 6A 30          PUSH 30

006D8047 . E8 481D2B00    CALL

006D804C . 83C4 04        ADD ESP,4

006D804F . 8945 C8        MOV DWORD PTR SS:\[EBP-38],EAX

006D8052 . 85C0           TEST EAX,EAX

006D8054 . C645 FC 10     MOV BYTE PTR SS:\[EBP-4],10

006D8058 . 74 09          JE SHORT ACAD.006D8063

006D805A . 8BC8           MOV ECX,EAX

006D805C . E8 8FBCF4FF    CALL ACAD.00623CF0

006D8061 . EB 02          JMP SHORT ACAD.006D8065

006D8063 > 33C0           XOR EAX,EAX

006D8065 > 8983 5C010000 MOV DWORD PTR DS:\[EBX+15C],EAX

006D806B . BF 1B46AC00    MOV EDI,ACAD.00AC461B ASCII "kjunk"

006D8070 . 83C9 FF        OR ECX,FFFFFFFF

006D8073 . 33C0           XOR EAX,EAX

006D8075 . F2:AE          REPNE SCAS BYTE PTR ES:\[EDI]

006D8077 . F7D1           NOT ECX

006D8079 . 2BF9           SUB EDI,ECX

006D807B . C645 FC 0E     MOV BYTE PTR SS:\[EBP-4],0E

006D807F . 8BC1           MOV EAX,ECX

006D8081 . 8BF7           MOV ESI,EDI

006D8083 . BF 1046AC00    MOV EDI,ACAD.00AC4610 ASCII "junkjunkjunkjunk"

006D8088 . C1E9 02        SHR ECX,2

006D808B . F3:A5          REP MOVS DWORD PTR ES:\[EDI],DWORD P>

006D808D . 8BC8           MOV ECX,EAX

006D808F . 83E1 03        AND ECX,3



`006D8092   . F3:A4           REP MOVS BYTE PTR ES:[EDI],BYTE PTR>`  

`006D8094   . E8 87CA0D00     CALL ACAD.007B4B20`  

`006D8099   . FF15 849F9E00   CALL DWORD PTR DS:[<&sharedb15.?get>;  SHAREDB1.?getCachedUserName@ShareDB@@YAPBDXZ`  

`006D809F   . 85C0            TEST EAX,EAX`  

`006D80A1   . 74 18           JE SHORT ACAD.006D80BB`  

`006D80A3   . 8038 00         CMP BYTE PTR DS:[EAX],0`  

`006D80A6   . 74 13           JE SHORT ACAD.006D80BB`  

`006D80A8   . 6A 1E           PUSH 1E                       /n = 1E (30.)`  

`006D80AA   . 50             PUSH EAX                      |s2`  

`006D80AB   . 68 F0F1AF00    PUSH ACAD.00AFF1F0              |s1 = "user"`  

`006D80B0   . FF15 9C5E9E00  CALL DWORD PTR DS:[<&MSVCRT._mbsnbc>; \\_mbsnbcpy`  

`006D80B6   . 83C4 0C         ADD ESP,0C`  

`006D80B9   . EB 1C           JMP SHORT ACAD.006D80D7`  

`006D80BB   > 6A 00           PUSH 0`  

`006D80BD   . E8 1E5CE5FF     CALL ACAD.0052DCE0`  

`006D80C2   . 6A 1E           PUSH 1E                       /n = 1E (30.)`  

`006D80C4   . 68 BC5FAC00     PUSH ACAD.00AC5FBC              |s2 = "这里隐去：是你版权所有信息"`  

`006D80C9   . 68 F0F1AF00     PUSH ACAD.00AFF1F0              |s1 = "user"`  

`006D80CE   . FF15 9C5E9E00   CALL DWORD PTR DS:[<&MSVCRT._mbsnbc>; \\_mbsnbcpy`  

`006D80D4   . 83C4 10         ADD ESP,10`  

`006D80D7   > C605 0EF2AF00 > MOV BYTE PTR DS:[AFF20E],0`  

`006D80DE   . E8 6DABD2FF     CALL ACAD.00402C50`  

`006D80E3   . 8D8B C8000000   LEA ECX,DWORD PTR DS:[EBX+C8]`  

`006D80E9   . 51             PUSH ECX                      /Arg1`  

`006D80EA   . E8 A14CEEFF     CALL ACAD.005BCD90            我们返回这里接着走下去`  

`*****接着 F8，走`  

`006D80EF   . 83C4 04         ADD ESP,4`  

`006D80F2   . 85C0            TEST EAX,EAX`  

`006D80F4   . 75 0A           JNZ SHORT ACAD.006D8100`  

`006D80F6   . E8 75ABD2FF     CALL ACAD.00402C70`  

`006D80FB   . E9 4D030000     JMP ACAD.006D844D`  

`......`  

`......省略`  

`......`  

`006D82AC   . E8 8F202B00     CALL`  

`006D82B1   . 8B43 20         MOV EAX,DWORD PTR DS:[EBX+20]`  

`006D82B4   . 8B40 20         MOV EAX,DWORD PTR DS:[EAX+20]`  

`**********************`  

`一路接着 F8，走到这里`  

`************************`  

`006D82B7   . 50             PUSH EAX                      /hWnd 到了关键地方了`  

`006D82B8   . FF15 2C639E00  CALL DWORD PTR DS:[<&USER32.UpdateW>; \\UpdateWindow`  

`006D82BE   > E8 ED1C0E00     CALL ACAD.007B9FB0         关键 call，如果拔掉网线这里会等很久啊。这里看 license 是否安装啦！F7 进入`  

`006D82C3   . 85C0            TEST EAX,EAX                 如果验证成功，eax=0。`  

`006D82C5   . 0F85 82010000   JNZ ACAD.006D844D`  

`006D82CB   . E8 F01C0E00     CALL ACAD.007B9FC0      ；关键 call，如果拔掉网线这里会验证失败。这里 check license 的正确啦，当然还有好几处！F7 进入`  

`006D82D0   . 85C0            TEST EAX,EAX                 如果验证成功，eax=0。`  

`006D82D2   . 0F85 75010000   JNZ ACAD.006D844D`  

`006D82D8   . 8D4D CC         LEA ECX,DWORD PTR SS:[EBP-34] ；到此以后，成功，`  

`006D82DB   . E8 E41A2B00     CALL`  

`006D82E0   . 8D4D A8         LEA ECX,DWORD PTR SS:[EBP-58]`  

`006D82E3   . 6A 00           PUSH 0                        /Arg2 = 00000000`  

`006D82E5   . 51             PUSH ECX                      |Arg1`  

`006D82E6   . C645 FC 11     MOV BYTE PTR SS:[EBP-4],11          |`  

`......`  

`......省略`  

`......`  

`走向成功`  

`.............`  

`\------------------------------------------------`  

`我们先 F7 进入关键 CALL ACAD.007B9FB0，`  

`（如果拔掉网线这里会等很久啊。这里看 license 是否安装啦！）`  

`007B9FB0  /$ E8 5B56E0FF    CALL ACAD.005BF610    ；我们 go into 看看 F7。如果验证成功，返回 eax＝1。`  

`007B9FB5  |. F6D8            NEG AL                      eax=000000ff`  

`007B9FB7  |. 1BC0            SBB EAX,EAX                 FFFFFFFF`  

`007B9FB9  |. F7D8            NEG EAX                     eax=1`  

`007B9FBB  |. 48             DEC EAX                       eax=0`  

`007B9FBC  |. 83E0 01        AND EAX,1             eax=0，返回成功`  

`007B9FBF  \. C3             RETN`  

`\----------------------------------------------`  

`我们 go into CALL ACAD.005BF610 来到这里`  

`005BF610  /$ 55             PUSH EBP`  

`005BF611  |. 8BEC           MOV EBP,ESP`  

`005BF613  |. 51             PUSH ECX`  

`005BF614  |. 53             PUSH EBX`  

`005BF615  |. 56             PUSH ESI`  

`005BF616  |. 57             PUSH EDI`  

`005BF617  |. BE 2E46AC00    MOV ESI,ACAD.00AC462E`  

`005BF61C  |. BF A02EAC00    MOV EDI,ACAD.00AC2EA0`  

`005BF621  |. C745 FC 000000> MOV DWORD PTR SS:[EBP-4],0`  

`005BF628  |. 81EE 86000000  SUB ESI,86`  

`005BF62E  |. 83EF 72        SUB EDI,72`  

`005BF631  |. C705 809FB500 > MOV DWORD PTR DS:[B59F80],1`  

`005BF63B  |. E8 C0CBFFFF    CALL ACAD.005BC200      *******第一个最关键 call F7 进去吧*****`  

`005BF640  |. 8BD8           MOV EBX,EAX                       ebx＝eax=0`  

`005BF642  |. C705 809FB500 > MOV DWORD PTR DS:[B59F80],0`  

`005BF64C  |. 85DB           TEST EBX,EBX`  

`005BF64E  |. 75 53          JNZ SHORT ACAD.005BF6A3`  

`005BF650  |. E8 7BF7FFFF    CALL ACAD.005BEDD0`  

`005BF655  |. 83F8 2A        CMP EAX,2A                          eax=19`  

`005BF658  |. 75 1A          JNZ SHORT ACAD.005BF674`  

`005BF65A  |. A1 106DAC00    MOV EAX,DWORD PTR DS:[AC6D10]`  

`005BF65F  |. C745 FC 430000> MOV DWORD PTR SS:[EBP-4],43`  

`005BF666  |. 8B88 0C010000  MOV ECX,DWORD PTR DS:[EAX+10C]`  

`005BF66C  |. C701 04000000  MOV DWORD PTR DS:[ECX],4`  

`005BF672  |. EB 41          JMP SHORT ACAD.005BF6B5`  

`005BF674  |> E8 57F7FFFF    CALL ACAD.005BEDD0`  

`005BF679  |. 83F8 11        CMP EAX,11                          eax=19`  

`005BF67C  |. 74 0A          JE SHORT ACAD.005BF688`  

`005BF67E  |. E8 4DF7FFFF    CALL ACAD.005BEDD0                   eax=19`  

`005BF683  |. 83F8 19        CMP EAX,19`  

`005BF686  |. 75 2D          JNZ SHORT ACAD.005BF6B5`  

`005BF688  |> 8B15 106DAC00  MOV EDX,DWORD PTR DS:[AC6D10]                 ACAD.00AC69C4`  

`005BF68E  |. C745 FC 430000> MOV DWORD PTR SS:[EBP-4],43       ；看到吗 DWORD PTR SS:[EBP-4]＝43`  

`005BF695  |. 8B82 0C010000  MOV EAX,DWORD PTR DS:[EDX+10C]`  

`005BF69B  |. C700 03000000  MOV DWORD PTR DS:[EAX],3`  

`005BF6A1  |. EB 12          JMP SHORT ACAD.005BF6B5`  

`005BF6A3  |> 8B0D 106DAC00  MOV ECX,DWORD PTR DS:[AC6D10]                 ACAD.00AC69C4`  

`005BF6A9  |. 8B91 0C010000  MOV EDX,DWORD PTR DS:[ECX+10C]`  

`005BF6AF  |. C702 00000000  MOV DWORD PTR DS:[EDX],0`  

`005BF6B5  |> 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]                 eax＝43`  

`005BF6B8  |. 83C0 BD        ADD EAX,-43                eax=0`  

`005BF6BB  |. F7D8           NEG EAX  `  

`005BF6BD  |. 1BC0           SBB EAX,EAX`  

`005BF6BF  |. 25 4D2F0000    AND EAX,2F4D                          eax=0`  

`005BF6C4  |. 05 59E8FFFF    ADD EAX,-17A7                         eax=FFFFE859`  

`005BF6C9  |. 35 59E8FFFF    XOR EAX,FFFFE859                     0`  

`005BF6CE  |. 66:8947 72     MOV WORD PTR DS:[EDI+72],AX`  

`005BF6D2  |. 66:8986 860000> MOV WORD PTR DS:[ESI+86],AX`  

`005BF6D9  |. 33C0           XOR EAX,EAX`  

`005BF6DB  |. 83FB FE        CMP EBX,-2                          ebx=0`  

`005BF6DE  |. 5F             POP EDI`  

`005BF6DF  |. 5E             POP ESI`  

`005BF6E0  |. 0F95C0         SETNE AL                          到这 eax=1`  

`005BF6E3  |. 5B             POP EBX`  

`005BF6E4  |. 8BE5           MOV ESP,EBP`  

`005BF6E6  |. 5D             POP EBP`  

`005BF6E7  \. C3             RETN      `  

`\-------------------------------------------------`  

`***********第一个最关键 call 进去吧*****`  

`005BC214 处的 call 是在 adlmbase.dll 里的，F7 追进去，你会发现很多东东，`  

`这对你编写无限制的 license 文件很有用，但第 2 个关键 call 才是重点。闲话少说。`  

`005BC200  /$ B9 1019B000    MOV ECX,ACAD.00B01910`  

`005BC205  |. E8 060F1D00    CALL ACAD.0078D110`  

`005BC20A  |. 85C0           TEST EAX,EAX            eax=00f8c70`  

`005BC20C  |. 75 04          JNZ SHORT ACAD.005BC212  一般应该 jmp，保险就改 eb04`  

`005BC20E  |. 83C8 FF        OR EAX,FFFFFFFF        ；到这个可不行，-1。`



```
005BC211  |. C3           RETN  

005BC212  |> 8BC8         MOV ECX,EAX    ；看到下面的 adlmbase.?Authorize@AdlmLic，太明白了  

005BC214  |. FF15 3C999E00 CALL DWORD PTR DS:[<&adlmbase.?Authorize@AdlmLic>]  ADLMBASE.?Authorize@AdlmLicense@@QAEHXZ  

005BC21A  |. 35 903F0000   XOR EAX,3F90        eax\=0，0 xor 3F90=3F90,那末此时 eax=3F90  

005BC21F  |. 3D 903F0000   CMP EAX,3F90        ；相等不跳  

005BC224  |. 75 03         JNZ SHORT ACAD.005BC229   ；跳了可不行！  

005BC226  |. 33C0          XOR EAX,EAX   ；eax＝0  

005BC228  |. C3           RETN      ；返回 eax＝0，成功  

005BC229  |> 35 903F0000   XOR EAX,3F90  

005BC22E  |. 33C9          XOR ECX,ECX  

005BC230  |. 83C0 4F       ADD EAX,4F  

005BC233  |. 35 A4CE0000   XOR EAX,0CEA4  

005BC238  |. 3D F5CE0000   CMP EAX,0CEF5  

005BC23D  |. 0F95C1        SETNE CL  

005BC240  |. 83C1 FE      ADD ECX,-2  

005BC243  |. 8BC1          MOV EAX,ECX  

005BC245  \\. C3          RETN  
```

\----------------------------------------------------------------

再来看看 `006D82CB  CALL ACAD.007B9FC0` 是关键 call，如果拔掉网线这里会验证失败。

这里 check license 的正确啦，当然还有好几处！

\* Referenced by a CALL at Address:

|:006D82CB    

|

:007B9FC0 55                    push ebp

:007B9FC1 8BEC                  mov ebp, esp

:007B9FC3 83EC08                sub esp, 00000008

:007B9FC6 56                    push esi

:007B9FC7 8D45F8                lea eax, dword ptr [ebp-08]

:007B9FCA 57                    push edi

:007B9FCB 50                    push eax

:007B9FCC BE85FFFFFF            mov esi, FFFFF85

:007B9FD1 E8FE021D00            call 0098A2D4

:007B9FD6 8B0DD06AAC00          mov ecx, dword ptr [00AC6AD0]

:007B9FDC 8B38                  mov edi, dword ptr [eax]

:007B9FDE 8B01                  mov eax, dword ptr [ecx]

:007B9FE0 83F803                cmp eax, 00000003

:007B9FE3 7404                  je 007B9FE9     ；-----jmp 走

:007B9FE5 85C0                  test eax, eax

:007B9FE7 7511                  jne 007B9FFA

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:007B9FE3(C)

－－－－－－－－－－－－－－－－－－－

```
007B9FE9  |> E8 6222E0FF    CALL ACAD.005BC250        第二个最关键 call F7 进去吧 into  

007B9FEE  |. 8BF0          MOV ESI,EAX  

007B9FF0  |. F7DE          NEG ESI  

007B9FF2  |. 1BF6          SBB ESI,ESI  

007B9FF4  |. 83E6 01       AND ESI,1  

007B9FF7  |. 83EE 7B       SUB ESI,7B  

007B9FFA  |> 8B15 D06AAC00 MOV EDX,DWORD PTR DS:[AC6AD0]                    ACAD.00B01F64  

007BA000  |. 833A 04       CMP DWORD PTR DS:[EDX],4  

......  

......省略  

......  

007BA04B  |. E8 10000000   CALL ACAD.007BA060                    \ACAD.007BA060  

007BA050  |. 8B00          MOV EAX,DWORD PTR DS:[EAX]  

007BA052  |. 5F            POP EDI  

007BA053  |. 5E            POP ESI  

007BA054  |. 8BE5          MOV ESP,EBP  

007BA056  |. 5D            POP EBP  

007BA057  \\. C3          RETN  
```

\---------------------------------------------------------

***第二个最关键 call 进去吧***

`005BC264` 处的 call 是在 `adlmbase.dll` 里的，F7 追进去，你会发现很多东东，

这对你编写无限制的 license 文件很有用。闲话少说。

这个 call，被好几处调用，我们可以改它！

```
005BC250  /$ B9 1019B000    MOV ECX,ACAD.00B01910  

005BC255  |. E8 B60E1D00    CALL ACAD.0078D110  

005BC25A  |. 85C0          TEST EAX,EAX  

005BC25C  |. 75 04         JNZ SHORT ACAD.005BC262       ；一般应该 jmp，保险就改 eb04  

005BC25E  |. 83C8 FF       OR EAX,FFFFFFFF      ；到这个可不行，-1。       

005BC261  |. C3           RETN  

005BC262  |> 8BC8          MOV ECX,EAX         看到下面的 adlmbase.?Check@AdlmLicens，太明白了  

005BC264  |. FF15 38999E00 CALL DWORD PTR DS:[<&adlmbase.?Check@AdlmLicens>]  ADLMBASE.?Check@AdlmLicense@@QAEHXZ  

005BC26A  |. 35 903F0000   XOR EAX,3F90       ；上面 eax 返回 0，那就成功了。  

005BC26F  |. 2D 903F0000   SUB EAX,3F90  

005BC274  |. F7D8          NEG EAX  

005BC276  |. 1BC0          SBB EAX,EAX  

005BC278  |. 25 7AF10000   AND EAX,0F17A       ；成功，返回 eax＝0  

005BC27D  \\. C3          RETN  
```

\--------

## 破解方法

\--------

### 方法 1

我们要她还去网络验证，但无论返回成功与否，我们都可以用。

此法好处是，万一有人在服务器端监控，可以看到我们在同时使用，至多是点数满了，不会发现我们在作弊。除非他同时有很多人来查我们同时在线，才会发现怎么点数满了，还能用，如果你有 50 个点以上，让他查吧。

```
005BC20C  |. 75 04         JNZ SHORT ACAD.005BC212   一般应该 jmp，保险就改 75 改为 eb

005BC21A  |. 35 903F0000   XOR EAX,3F90           改为 mov eax,3f90，35 改为 B8

005BC25C  |. 75 04         JNZ SHORT ACAD.005BC262  一般应该 jmp，保险就改 75 改为 eb

005BC26A  |. 35 903F0000   XOR EAX,3F90       ；改为 mov eax,3f90，35 改为 B8
```

这时你甚至可以拔掉网线使用，你可以验证一下。

运行 `acad`，停顿时间比联网长多了，原因我在 `cadR14` 破文里说过

过一会出现 Flex license 窗口，这时你点击 cancel，再点出错窗口的确定，即可进入。

另外，当然点数满时，也会有出错窗口出现，但再点出错窗口的确定，即可进入。这是因为我们没有使用 90 大法。

如果你始终联接在网上，而 Flex license 服务器也不关闭，那可以只改 2 个字节！！

就是：

```
005BC21A  |. 35 903F0000   XOR EAX,3F90        改为 mov eax,3f90，35 改为 B8

005BC26A  |. 35 903F0000   XOR EAX,3F90       ；改为 mov eax,3f90，35 改为 B8
```

解释：以上分析可以看到来到 `005BC21A`，`005BC26A` 时，若 `eax=0` 就验证成功，此后下一句 `XOR EAX,3F90`，而 `eax=0` 那么 `0 XOR 3f90=3F90`！！那末此时 `eax=3F90`，我们不管前面 `eax` 返回什么值，我强制让 `eax=3F90` 就行了 `mov eax,3f90`。

他的机器码：`B8 903F0000`，你再看 `XOR EAX,3F90` 的机器码 `35 903F0000`。

因此只改 2 处共 2 字节！

### 方法 2

我们要去掉网络验证，我们都可以用。

此法好处是，没有网络验证验证，节约时间，但你最好自己在家里用。

否则，他们来查，怎么一个人也没有在服务器显示，而你们明明都在用。

很简单：在方法 1 的基础上，把那前面的两个 call 都 nop 掉，90 大法显神威。

到此，我们可以都使用正版了！！！哈哈

如果你想家里也使用，你把机子抗来，联网安装成功后，再打上补丁，回去就可以用了。

### 总结

Flexlm 加密看雪地书里讲得很详细了。我看 Flexlm 加密的弱点就是可以爆破，他无法像一些狗加密那样把程序的关键数据或加地址密并储存在狗里，而只能返回成功失败标志！虽然 `CAD2002` 加密不使用标准的 Flexlm 的 Dll，但我怀疑 `adlmbase.dll` 就是我们要找的，很可能是专为 `autodesk` 做的，有兴趣可以仔细研究。

对于 Flexlm 一类的加密，我们除了作无限制的 license 以外，就是爆破它，关键是找到断点！愿我的一些找断点的经验于大家有所帮助。首先吃透软件的运行规律，反汇编找它的可疑处下断。

最后，建议 Flexlm 加密等可以象一些狗加密那样把程序的关键数据或地址加密并储存在狗里一样，而不是只返回成功失败标志！或者，在程序运行中间再随机验证，不过这会影响客户端运行速度，想象你有 100 个点会怎样，服务器也受不了吧。

当然，这还是防不住的。

`4dd466d0e1e0cc123afbe504da2462ca`

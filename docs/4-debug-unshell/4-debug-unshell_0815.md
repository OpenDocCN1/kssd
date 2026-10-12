
# Unpacking `US_unpackMe_2`

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-546.htm`](https://www.pediy.com/kssd/pediy07/pediy7-546.htm)

By: 来自轻院的狼【+Immlep+】

Site: [`ptteam.com`](http://ptteam.com)

Blog: [`immlep.blogone.net`](http://immlep.blogone.net)

`US_unpackMe_5` 是 `Unpacking Saga`（简称 `US`）出的来验证成员的五个 `UnpackMe` 中的一个，之前发过一篇 `unpackme_5` 的，今天趁着五一放假，整理了前面两个（`unpackMe_1` 和 `unpackMe_2`），这里讨论的是 `US_unpackMe_2`，应该是用 `hying` 的壳加壳的，版本不知（知道的放一下话），应该是比较底版本的壳，因为脱壳难度不高，不知道是不是说明所说的放水的一个：）

这次不说那么多了，来个快脱：

载入 `unpackme_2`，撤销所有的断点，忽略除除零异常外的异常，`F9` 让 `OD` 飞起来，啪的一声，除零异常断下：

```
00383531    F7F3                DIV EBX               //断在这里
00383533    90                  NOP                   //F2 这这里下断点
```

断下后，在 `00383533` 下软断，下断后 `SHIFT+F9` 忽略异常，在 `00383533` 断下，然后向下走，或向下翻，看到：

```
0038383F    55                  PUSH EBP              ; 偷掉的字节
00383840    8BEC                MOV EBP, ESP          ; 偷掉的字节
00383842    EB 19               JMP SHORT 0038385D
00383844    83FB 02             CMP EBX, 2
00383847    75 0F               JNZ SHORT 00383858
00383849    8BC5                MOV EAX, EBP
0038384B    5D                  POP EBP               ; 0012FFE0
0038384C    5B                  POP EBX               ; 0012FFE0
0038384D    59                  POP ECX               ; 0012FFE0
0038384E    5A                  POP EDX               ; 0012FFE0
0038384F    5E                  POP ESI               ; 0012FFE0
00383850    FFB0 A7060000       PUSH DWORD PTR [EAX+6A7]
00383856    EB 05               JMP SHORT 0038385D
00383858    5D                  POP EBP               ; 0012FFE0
00383859    5B                  POP EBX               ; 0012FFE0
0038385A    59                  POP ECX               ; 0012FFE0
0038385B    5A                  POP EDX               ; 0012FFE0
0038385C    5E                  POP ESI               ; 0012FFE0
0038385D    FFE0                JMP NEAR EAX          ; 这里返回到程序 OEP＝00401950
```

程序 `OEP` 开头

```
00401950    FFD7                CALL NEAR EDI         ; unpackme.00401952
00401952    58                  POP EAX               ; 0012FFF0
```

下面重点恢复被破坏的壳，进入程序后遇到第一个 `CALL`

```
0040197D    90                  NOP
0040197E    E8 DC1EF8FF         CALL 0038385F         ;进去看看怎么解密的
0038385F    50                  PUSH EAX
00383860    60                  PUSHAD
00383861    E8 06000000         CALL 0038386C         //这里进去，会经过些异常
00383866    8B6424 08           MOV ESP, DWORD PTR [ESP+8]      ; unpackme.00401952
0038386A    EB 20               JMP SHORT 0038388C  //不用管 00383861，运行到 00383861 时我们直接在这里下断，F9 断下
```

接着就看到了解密的关键代码了：

`ESI` 指向一个表，分布是这样的

调用 `CALL`（`JMP`）下一条指令的地址：加密过的函数地址，如下

```
00383EF3  08 17 40 80| C7 16 34 E0 |
```

函数会解密原程序的 `CALL` 和 `JMP`：

`CALL FuntionAddr`（如 `CALL NEAR DWORD PTR [<&msvcrt.#154>]`）

和

`jmp FuntionAddr`（如 `JMP NEAR DWORD PTR [<&msvcrt.#315>]`）

解密函数判断解密的是 `CALL` 还是 `JMP` 是根据 `ESI` 表中的高位的，如果是 `80` 则解密的是 `JMP` 指令，如果是 `00` 则解密的是 `CALL` 指令，像上面的我们就知道 `401708` 前面的 `00383EF3` 里的就是条 `CALL` 指令来的：

```
0038389D    5D                  POP EBP               ; 00383899
0038389E    8B6D 00             MOV EBP, DWORD PTR [EBP]
003838A1    8B7C24 24           MOV EDI, DWORD PTR [ESP+24]
003838A5    8BB5 AF060000       MOV ESI, DWORD PTR [EBP+6AF]
003838AB    03F5                ADD ESI, EBP
003838AD    8B06                MOV EAX, DWORD PTR [ESI]       //ESI 对应一个表，取地址
003838AF    33D2                XOR EDX, EDX            ; ntdll.77FB1742
003838B1    B9 02000000         MOV ECX, 2
003838B6    F7E1                MUL ECX
003838B8    D1E8                SHR EAX, 1
003838BA    0385 83060000       ADD EAX, DWORD PTR [EBP+683]
003838C0    2B85 8F060000       SUB EAX, DWORD PTR [EBP+68F]
003838C6    3BF8                CMP EDI, EAX
003838C8    75 0A               JNZ SHORT 003838D4
003838CA    0AD2                OR DL, DL               //如果刚才去到的地址高位是 80 的话,DL=1
003838CC    75 04               JNZ SHORT 003838D2      //如果是 jmp 的话跳到 003838D2 执行
003838CE    EB 09               JMP SHORT 003838D9
003838D0    EB 02               JMP SHORT 003838D4
003838D2    EB 2F               JMP SHORT 00383903
003838D4    83C6 08             ADD ESI, 8
003838D7    EB D4               JMP SHORT 003838AD
003838D9    8B46 04             MOV EAX, DWORD PTR [ESI+4]
003838DC    03BD 8F060000       ADD EDI, DWORD PTR [EBP+68F]
003838E2    2BBD 83060000       SUB EDI, DWORD PTR [EBP+683]
003838E8    2BC7                SUB EAX, EDI
003838EA    F7D0                NOT EAX
003838EC    C1C0 10             ROL EAX, 10
003838EF    0385 83060000       ADD EAX, DWORD PTR [EBP+683]
003838F5    2B85 8F060000       SUB EAX, DWORD PTR [EBP+68F]
003838FB    8B00                MOV EAX, DWORD PTR [EAX]
003838FD    894424 20           MOV DWORD PTR [ESP+20], EAX
00383901    61                  POPAD
00383902    C3                  RETN
00383903    8B46 04             MOV EAX, DWORD PTR [ESI+4]      ; 查表，地址解密
00383906    03BD 8F060000       ADD EDI, DWORD PTR [EBP+68F]
0038390C    2BBD 83060000       SUB EDI, DWORD PTR [EBP+683]
00383912    2BC7                SUB EAX, EDI
00383914    F7D0                NOT EAX
00383916    C1C0 10             ROL EAX, 10           ;EAX 放的才是函数的值
00383919    0385 83060000       ADD EAX, DWORD PTR [EBP+683]
0038391F    2B85 8F060000       SUB EAX, DWORD PTR [EBP+68F]
00383925    8B00                MOV EAX, DWORD PTR [EAX]
00383927    894424 24           MOV DWORD PTR [ESP+24], EAX
0038392B    61                  POPAD
0038392C    83C4 04             ADD ESP, 4            //jmp 的话就没有返回地址的。。
0038392F    C3                  RETN
```

按照上面找个地方写修复代码，找到 `0040276C`：

```
0040276C      60                PUSHAD
0040276D      BE F33E3800       MOV ESI, 383EF3       //383EF3 表开始地址
00402772      8B06              MOV EAX, DWORD PTR [ESI]       //取表中的地址
00402774      833E 00           CMP DWORD PTR [ESI], 0
00402777      74 51             JE SHORT unpackme.004027CA      //表是否结束，表以 00 00 00 00 为结束标志
00402779      90                NOP
0040277A      90                NOP
0040277B      90                NOP
0040277C      90                NOP
0040277D      33D2              XOR EDX, EDX
0040277F      B9 02000000       MOV ECX, 2
00402784      F7E1              MUL ECX
00402786      D1E8              SHR EAX, 1
00402788      08D2              OR DL, DL
0040278A      75 22             JNZ SHORT unpackme.004027AE     //是 jmp 吗
0040278C      90                NOP
0040278D      90                NOP
0040278E      90                NOP
0040278F      90                NOP
00402790      8B7E 04           MOV EDI, DWORD PTR [ESI+4]      //取加密过的函数地址
00402793      2BF8              SUB EDI, EAX          ;   unpackme.00401950
```



```
00402795      F7D7                    NOT EDI                       ;  unpackme.00401952  
00402797      C1C7 10               ROL EDI, 10  
0040279A      83E8 06               SUB EAX, 6  
0040279D      C700 FF150000         MOV DWORD PTR [EAX], 15FF       //写入 CALL NEAR DWORD PTR  
004027A3      8978 02               MOV DWORD PTR [EAX+2], EDI      //写入地址  
004027A6      83C6 08               ADD ESI, 8  
004027A9    ^ EB C7                JMP SHORT unpackme.00402772  
004027AB      90                    NOP  
004027AC      90                    NOP  
004027AD      90                    NOP  
004027AE      8B7E 04               MOV EDI, DWORD PTR [ESI+4]      //是 jmp 就跳到这里来  
004027B1      2BF8                  SUB EDI, EAX                    ;  unpackme.00401950  
004027B3      F7D7                  NOT EDI                         ;  unpackme.00401952  
004027B5      C1C7 10               ROL EDI, 10  
004027B8      83E8 06               SUB EAX, 6  
004027BB      C700 FF250000         MOV DWORD PTR [EAX], 25FF       //写入 JMP NEAR DWORD PTR  
004027C1      8978 02               MOV DWORD PTR [EAX+2], EDI      //写入地址  
004027C4      83C6 08               ADD ESI, 8  
004027C7    ^ EB A9                JMP SHORT unpackme.00402772  
004027C9      61                    POPAD  
004027CA      90                    NOP  
```

```
60 BE F3 3E 38 00 8B 06 83 3E 00 74 51 90 90 90 90 33 D2 B9 02 00 00 00 F7 E1 D1 E8 08 D2 75 22  
90 90 90 90 8B 7E 04 2B F8 F7 D7 C1 C7 10 83 E8 06 C7 00 FF 15 00 00 89 78 02 83 C6 08 EB C7 90  
90 90 8B 7E 04 2B F8 F7 D7 C1 C7 10 83 E8 06 C7 00 FF 25 00 00 89 78 02 83 C6 08 EB A9 61 90 00  
```

重新载入程序，跟到`0038383F`时(执行被偷的字节的地址)，找到`0040276C`写入上面的代码，转到`0040276C`执行还原代码后，跳到刚才取得的原函数的`OEP`

把

```
00401950    FFD7                  CALL NEAR EDI                     ; unpackme.00401952  
00401952    58                    POP EAX                           ; 0012FFF0  
```

恢复为：

```
00401950 >/$  55                  PUSH EBP  
00401951  |.  8BEC                MOV EBP, ESP  
```

在把运行`OEP`转到`00401950`，`dump it`，用`imprREC`：

填入

`OEP＝00001950`

`RAV＝2000`，大小＝`2000`   （`RAV`可以随便找程序里的`CALL`看看就知道了）

`getimport`，`level3 trace`，无效的`cut`，`fixdump`，`ok`

Special thx:

`eaton`,`ama`,`catking668`,[www.pediy.com](http://www.pediy.com),`www.crackslatinos.hispadominio.net`,`luocong's scc and you!`

Ps:但要记住，虽然我没提到你，我还是记得你的，感谢着你的。。。

`4dd466d0e1e0cc123afbe504da2462ca`

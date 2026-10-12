
# ArtCursors 4.01 脱壳 + 拆引信

> 原文：[`www.pediy.com/kssd/pediy06/pediy6574.htm`](https://www.pediy.com/kssd/pediy06/pediy6574.htm)

目标：`ArtCursors` 4.01  
壳：`ASProtect` 1.23 RC4

没办法，偶特别喜欢 32 位色的精美小图标、光标，懂得一点 `PhotoShop`，可是 `PhotoShop` 竟不支持图标、光标格式！于是上网狂找能编辑 32 位色图标、光标的软件，结果发现：怎么那些好用的图标编辑软件都那么喜欢穿 `Aspr` 牌的马甲呢？网上 `pj` 版又不好找，很吊胃口啊，不行，欲火焚身，非把她们的马甲脱光光再插插了不可~~~~~~~~~~~~~唉，偶等菜鸟不会分析注册算法，只能手忙脚乱地拿着各种工具干危险的力气活——爆破

## 一、脱壳

用 `stripper2.07`，秒杀 `Aspr`!

## 二、修补

偷走的代码被变形了，`Stripper` 就新增加一个段，把变形的入口代码抄下来放这里，实在不爽，不过天助我也，这个软件的早期版本我脱过，`Delphi5` 写的，他们的入口代码的格式我都能背熟了：

```asm
PUSH EBP
MOV EBP,ESP
ADD ESP,-0C
MOV EAX,xxxxxx
CALL xxxxxx
CALL [xxxxxx]  〈=被偷走的代码执行完后来到这里
```

只要在合适的时候找到 `EAX` 是多少就好了。

载入原版跟踪，闯过 26 处 `SEH`，然后在按 `Alt-M` 打开内存镜象，在 `code` 段设置内存访问断点，断下后来到这里：

```asm
0040692C  - FF25 B0E25400   JMP DWORD PTR DS:[54E2B0]  〈=断在这里
00406932    8BC0            MOV EAX,EAX
00406934  - FF25 ACE25400   JMP DWORD PTR DS:[54E2AC]
0040693A    8BC0            MOV EAX,EAX
0040693C  - FF25 A8E25400   JMP DWORD PTR DS:[54E2A8]
00406942    8BC0            MOV EAX,EAX
00406944  - FF25 A4E25400   JMP DWORD PTR DS:[54E2A4]
0040694A    8BC0            MOV EAX,EAX
```

清除内存访问断点后，继续 `F8` 跟踪，来到这里：

```asm
00406A34    BA 08615400     MOV EDX,ARTCUR.00546108
00406A39    52              PUSH EDX
00406A3A    8905 DCD45400   MOV DWORD PTR DS:[54D4DC],EAX
00406A40    8942 04         MOV DWORD PTR DS:[EDX+4],EAX
00406A43    C742 08 0000000>MOV DWORD PTR DS:[EDX+8],0
00406A4A    C742 0C 0000000>MOV DWORD PTR DS:[EDX+C],0
00406A51    E8 8AFFFFFF     CALL ARTCUR.004069E0
00406A56    5A              POP EDX
00406A57    58              POP EAX
00406A58    E8 C7D0FFFF     CALL ARTCUR.00403B24  〈=在此处记下 EAX 值：544d9c
00406A5D    C3              RETN
```

从 `406a5d` 处返回到 `545244 CALL [54BC14]`

上面的代码片段执行了一部分 `54523F CALL 00406A2C` 里的代码，因此可以在这个 `call` 里合适的时机找到 `EAX` 的值。

```asm
00545233    0000            ADD BYTE PTR DS:[EAX],AL  〈=被偷走的代码
00545235    0000            ADD BYTE PTR DS:[EAX],AL
00545237    0000            ADD BYTE PTR DS:[EAX],AL
00545239    0000            ADD BYTE PTR DS:[EAX],AL
0054523B    0000            ADD BYTE PTR DS:[EAX],AL
0054523D    0000            ADD BYTE PTR DS:[EAX],AL
0054523F    E8 E817ECFF     CALL ARTCUR.00406A2C        〈=断不下来
00545244    FF15 14BC5400   CALL DWORD PTR DS:[54BC14]  〈=断在这里
```

现在补上被偷走的代码：

```asm
00545234       55            PUSH EBP
00545235   .  8BEC          MOV EBP,ESP
00545237   .  83C4 F4       ADD ESP,-0C
0054523A   .  B8 9C4D5400   MOV EAX,00544D9C
```

然后用 `LordPE` 修正入口点为 `145234`，再把最后一个节删掉，并用 `winhex` 把该节所对应的尾部数据删掉（从该节的 `Roffset`：`183600` 开始到尾部）。

## 三、拆引信

用 `Stripper` 脱壳的文件，运行时总是提示时间到了，只能跟原版比较跟踪。

在 `545244` 处，`CALL [54BC14]`

原版 `call` 进 `544c48`，而脱壳版 `call` 进 `544d44`，于是把脱壳文件的 `54BC14` 处对应的文件偏移处数据改为 `48 4c 54 00`，保存，运行，轰~~~~~~~来到一个除法“炸弹”：

```asm
004FF0F3  .  E8 5429F6FF   CALL dump.00461A4C
004FF0F8  .  8BD8          MOV EBX,EAX
004FF0FA  .  81FB 40420F00 CMP EBX,0F4240
004FF100  .  7E 34         JLE SHORT dump.004FF136  〈=要跳
004FF102  .  E8 D9EEF7FF   CALL dump.0047DFE0
004FF107  .  85C0          TEST EAX,EAX
004FF109  .  7E 2B         JLE SHORT dump.004FF136
004FF10B  .  8D55 F8       LEA EDX,DWORD PTR SS:[EBP-8]
004FF10E  .  33C0          XOR EAX,EAX
004FF110  .  E8 9F39F0FF   CALL dump.00402AB4
004FF115  .  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
004FF118  .  E8 2F29F6FF   CALL dump.00461A4C
004FF11D  .  2BD8          SUB EBX,EAX
004FF11F  .  B8 17000000   MOV EAX,17
004FF124  .  99            CDQ  〈=异常
```

原版跟踪发现 `004FF100` 处在原版是跳的，不然也会异常，就把脱壳版的 `004FF100` 处机器码改成 `EB 34`，保存，运行。。。

轰！！！！！！！！！！！！！！！！！！！！！！！！！！！！

一个窗口炸弹被引爆了！无限的脱壳版程序在运行，系统资源耗尽！

重起，继续跟踪，发现炸弹引信在此：

```asm
004FF01A  .  E8 2D2AF6FF   CALL dump.00461A4C
004FF01F  .  3D 804F1200   CMP EAX,124F80
004FF024  .  7E 30         JLE SHORT dump.004FF056  〈=不跳就挂
004FF026  .  E8 B5EFF7FF   CALL dump.0047DFE0
004FF02B  .  85C0          TEST EAX,EAX
004FF02D  .  7E 27         JLE SHORT dump.004FF056
004FF02F  .  6A 05         PUSH 5
004FF031  .  6A 00         PUSH 0
004FF033  .  6A 00         PUSH 0
004FF035  .  8D55 F8       LEA EDX,DWORD PTR SS:[EBP-8]
004FF038  .  33C0          XOR EAX,EAX
004FF03A  .  E8 753AF0FF   CALL dump.00402AB4
004FF03F  .  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
004FF042  .  E8 CD51F0FF   CALL dump.00404214
004FF047  .  50            PUSH EAX                 ; |FileName
004FF048  .  6A 00         PUSH 0                   ; |Operation = NULL
004FF04A  .  6A 00         PUSH 0                   ; |hWnd = NULL
004FF04C  .  E8 D369F5FF   CALL <JMP.&shell32.ShellExecuteA> ; \ ShellExecuteA
```

就是 `4FF04C` 处的 `ShellExecuteA` 不断调用自己，造成窗口炸弹！好阴险的家伙！

原版的 `4FF024` 处也是跳的，不然也是窗口炸弹。

保存修改，再运行，哈，点掉 `NAG`，主界面出来了！不过老是闪，看来还有地方有问题，经过跟踪发现，上面两个”炸弹“的”引信”（条件跳转语句）前的 `call`，都是调用 `461a4c` 处的代码，`461a4c` 肯定有问题！

在 `461a4c` 里，用两个 `OD` 经过漫长的跟踪比较各个寄存器的值，发现这些代码很可疑：

```asm
00408E64  .  54            PUSH ESP                 ; /pLocalFileTime
00408E65  .  8D43 2C       LEA EAX,DWORD PTR DS:[EBX+2C]    ; |
00408E68  .  50            PUSH EAX                 ; |pFileTime
00408E69  .  E8 F2DCFFFF   CALL <JMP.&kernel32.FileTimeToLocalFileT>; \ FileTimeToLocalFileTime
00408E6E  .  53            PUSH EBX                 ; /pDOSTime
00408E6F  .  8D43 02       LEA EAX,DWORD PTR DS:[EBX+2]     ; |
00408E72  .  50            PUSH EAX                 ; |pDOSDate
00408E73  .  8D4424 08     LEA EAX,DWORD PTR SS:[ESP+8]     ; |
00408E77  .  50            PUSH EAX                 ; |pFileTime
00408E78  .  E8 DBDCFFFF   CALL <JMP.&kernel32.FileTimeToDosDateTim>; \ FileTimeToDosDateTime
```

经过这些代码后，原版和脱壳版的 `EAX` 寄存器的值不一样，脱壳版 `cmp` 指令比较 `eax` 值，触发“引信”。继续跟踪发现上面代码经过几次 `ret` 后，有这么一个语句：

```asm
004FF0A2  .  3D A0BB0D00   CMP EAX,0DBBA0
004FF0A7  .  0F9CC3        SETL BL  〈=好象是说，如果大于，就设置 BL 为 1
```

发现只要把 `SETL BL` 改成 `mov bl,1` 就搞定了，至此脱壳 + 修复完全搞定，时间限制解除，运气好啊~~~~~~~~~

接下来就是搞定 `NAG` 了，偶累了，下次再说 O_o

`4dd466d0e1e0cc123afbe504da2462ca`

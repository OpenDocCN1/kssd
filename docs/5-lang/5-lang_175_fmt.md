
# Flash5 导入保护 SWF 动画

> 原文：[`www.pediy.com/kssd/pediy04/kanxue183.htm`](https://www.pediy.com/kssd/pediy04/kanxue183.htm)

Flash5 导入保护 SWF 动画的补丁制作

作者：maomao

软件：Flash5

网上动画标准制作软件

下载：略

工具：`Trw2000 v1.23`

日期：2001-10-29

说明：现在很多人喜欢保护自己的 Flash 作品，但这种保护相当脆弱，现在网上流行的许多 Flash 破解工具，下载后破解即可。但总觉得不是“治本”的办法，而且太麻烦，万一在紧急的情况下找不到这些小小的工具，岂不是要让大侠丢了脸面？本人不幸就遇到一次，不过，嘿嘿，我另有高招~~~拿 `Flash.exe` 主文件开刀!

过程如下：

分别运行 `TRW2000` 和 `Flash5`，按 `CTRL+N` 激活 `TRW2000`，输入 `bpx messageboxa`，`F5` 返回 `Flash`，在 `Flash` 菜单中选择“`File->import...`”引入加壳 SWF 动画，立即被 `TRW` 拦截，下 `bd *`，`pmodule` 后，`Flash` 会弹出一说明框(告诉你无法引入此动画)，不要理它，按“`OK`”后又被 `TRW` 拦截，这时已经到了 `Flash` 领空:

```
0177:008E9564 FF15580DA300    CALL    `USER32!MessageBoxA`
0177:008E956A 8BF8            MOV     EDI,EAX                 <=====光标停在这一行
0177:008E956C 892B            MOV     [EBX],EBP
```

一直按住 `F10`，经过大约 3 个 `RET`，直到来到这里：

```
0177:0064E510  PUSH    EAX
0177:0064E511  CALL    004126D9
0177:0064E516  CMP     [EBP+FFFFFB00],EBX
0177:0064E51C  JG      0064E4E9
0177:0064E51E  CMP     DWORD [EBP+FFFFFBC8],BYTE +00
0177:0064E525  JZ      NEAR 0064E600
0177:0064E52B  CMP     DWORD [EBP+FFFFFBCC],BYTE +00
0177:0064E532  JNZ     0064E573
0177:0064E534  CMP     DWORD [009E4A78],BYTE +00
0177:0064E53B  JNZ     0064E54B
0177:0064E53D  PUSH    BYTE -01
0177:0064E53F  PUSH    BYTE +10
0177:0064E541  PUSH    DWORD 025A
0177:0064E546  CALL    007723CC            <=====这就是那个出错的 call
0177:0064E54B  MOV     BYTE [EBP\-04],02   <=====光标停在这一行
0177:0064E54F  CALL    0064E78C
0177:0064E554  MOV     BYTE [EBP\-04],00
0177:0064E558  CALL    0064E797
0177:0064E55D  MOV     DWORD [EBP\-04],FFFFFFFF <=====被赋成这个值就玩完了!
0177:0064E564  CALL    0064E7B1
```

再往上看看，哪里可以绕过出错的 call 和玩完的赋值？关键在这几个跳转上：

```
0177:0064E51E  CMP     DWORD [EBP+FFFFFBC8],BYTE +00
0177:0064E525  JZ      NEAR 0064E600      <=====值得怀疑
0177:0064E52B  CMP     DWORD [EBP+FFFFFBCC],BYTE +00
0177:0064E532  JNZ     0064E573           <=====也值得怀疑
0177:0064E534  CMP     DWORD [009E4A78],BYTE +00
0177:0064E53B  JNZ     0064E54B           <=====这个跳到 over 上
```

两个值得怀疑的跳转上都有标志比较，用 `F9` 在 `0177:0064E532 JNZ 0064E573` 设断点，运行，重新导入受保护的 SWF，果然在这一行打往，并且不跳。下命令 `r fl z` 后，用 `F5` 运行程序，警告框不见了!但新出来一个密码输入框!!!好家伙，还有一手。

现在在第一个怀疑点 `0177:0064E525 JZ NEAR 0064E600` 设断，按 `F5`，再用 `Flash` 导入入受保护的 SWF，中断后，发现也不跳!用 `r fl z` 修改 ZF 标志位，再 `F5` 运行，嘿嘿嘿嘿，不要密码成功导入了那个该死的 SWF!!!看来，只要修改这个 jz 就万事大吉了。

原来 `[EBP+FFFFFBC8]`放置文件导入保护标志，0 表示没有保护；`[EBP+FFFFFBCC]`放置密码保护标志，如果上面的标志为 0，则不比较密码!

整理一下：

用 `Ultraedit` 查找: `83 BD C8 F8 FF FF 00 0F 84 D5 00 00 00` ( `CMP DWORD [EBP+FFFFFBC8],BYTE+00` `JZ NEAR 0064E600` )

改为： `83 BD C8 F8 FF FF 00 E9 D6 00 00 00 90` ( `CMP DWORD [EBP+FFFFFBC8],BYTE+00` `Jmp 0064E600` )

大功告成，下次再也不用去找什么 flash 破解工具了!

收工~~~~~~ `4dd466d0e1e0cc123afbe504da2462ca`

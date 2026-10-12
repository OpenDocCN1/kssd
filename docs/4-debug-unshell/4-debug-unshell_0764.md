
# MSLRHv0.31 加壳记事本脱壳(又一脱壳实例,高手莫入)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-505.htm`](https://www.pediy.com/kssd/pediy07/pediy7-505.htm)

【破解作者】`clide2000[DFCG][OCN]`

【作者邮箱】`54arma@sina.com`

【使用工具】`OD`;`LoadPe`;`ImportREC`

【破解平台】`Win9x/NT/2000/XP`

【软件名称】`MSLRHv0.31` 加壳记事本脱壳

【下载地址】附件:`unpackme.rar`

【软件简介】`MSLRHv0.31` 加壳英文版记事本(中文的加壳后运行不了)

【加壳方式】`MSLRHv0.31`

【破解声明】我是一只小菜鸟，偶得一点心得，愿与大家分享：)

--------------------------------------------------------------------------------

## 【破解内容】

方法是参照 `peaceclub` 兄的\[真正快速脱壳方法\],个人认为此种方法很好,特别适合我们这些小菜.在此也表示对 `pecaceclub` 兄的感谢

为广大和我一样的小菜,能真正对付 `MSLRHv0.31` 加壳的软件，特以记事本来演示一下。高手莫笑。

### 1. 用 `OD` 载入加壳后的启事本 `unpackme.exe` 停在：

```
00411000  >  $  60                  pushad
00411001    .  D1CB                ror ebx, 1
00411003    .  0FCA                bswap edx
00411005    .  C1CA E0             ror edx, 0E0               ;  Shift constant out of range 1..31
00411008    .  D1CA                ror edx, 1
0041100A    .  0FC8                bswap eax
0041100C    .  EB 01              jmp short unpackme.0041100F
0041100E    .  F1                  int1
0041100F    >  0FC0C9              xadd cl, cl
00411012    .  D2D1                rcl cl, cl
```

`Ctrl+B`，在 `HEX+00` 框中输入:`68????????c3`,点 `OK` 后来到：

```
0041D0A0  |.  68 8ECB9A00        push 9ACB8E
0041D0A5  \\.  C3                retn ;就是要把这里改成 cc 啊
0041D0A6      33                db 33                          ;  CHAR '3'
0041D0A7      C9                db C9
0041D0A8      E8                db E8
```

将 `0041D0A5  \\.  C3  retn` 改成 `CC` ，即 `int 3`

下面，右击 `0041D0A5` 这一行，选择`"Copy to executable"`->`"Selection"`,

接着在新弹出的窗口中，右击，在快捷菜单中选择`"Save file"`,最后启个文件名(我这里的新文件名为 `up.exe`)，按保存（即完成了修改保存）

现在关闭 `OD`。

### 2. 双击运行之前保存的 `up.exe` 文件，发生异常,`ollydbg` 调试 (注意,到这里之前,与脱 `MSLRHv0.31` 主程序一致)

这时会停在 `0041D0A5`

注意，这里已经是被加壳开头的程序，即 `Stolen Code` 的处理位置

```
0041D087    83C4 08             add esp, 8
0041D08A    892C24              mov dword ptr ss:[esp], ebp
0041D08D    54                  push esp
0041D08E    55                  push ebp
0041D08F   /EB 01               jmp short un.0041D092
0041D091   |E8 83C4045D         call 5D469519
0041D096    83EC 44             sub esp, 44
0041D099    56                  push esi
0041D09A    FF15 E0634000      call dword ptr ds:[4063E0]       ; kernel32.GetCommandLineA
0041D0A0    68 D9104000        push un.004010D9
0041D0A5    CC                  int3
```

下面我们做如下修改

1） 恢复 `0041D0A5` 处数据为 `C3`

2） 去除 `0041D091` 的花指令

修改后结果如下

```
0041D087    83C4 08             add esp, 8      ；＃
0041D08A    892C24              mov dword ptr ss:[esp], ebp ；＃
0041D08D    54                  push esp        ；＃
0041D08E    55                  push ebp        ；＃
0041D08F    EB 01               jmp short un.0041D092
0041D091    90                  nop              ;花指令，nop 掉
0041D092    83C4 04             add esp, 4       ；＃
0041D095    5D                  pop ebp          ；＃
0041D096    83EC 44             sub esp, 44      ;＊
0041D099    56                  push esi         ;＊
0041D09A    FF15 E0634000      call dword ptr ds:[4063E0]       ; kernel32.GetCommandLineA  ;＊
0041D0A0    68 D9104000        push un.004010D9       ；伪ＯＥＰ
0041D0A5    C3                  retn                         ；从这里返回到伪ＯＥＰ处
```

注意上面加了`＊`号的地方，是被 `Stolen Code` 的字节

根据经验`ＯＥＰ`前两句应该是 `push ebp`, `mov ebp,esp`， 上面加`＃`号的完成的就是这两句的功能。

加上带`＊`号的一共有 `13` 字节,即完整的 `Stolen Code` 内容为:

```
     push ebp
     mov ebp, esp
     sub esp, 44
     push esi
     call dword ptr ds:[4063E0]
```

现在 `Ctrl+G`,输入 `4010cc`,并将`ＥＩＰ`改为 `4010cc`,补全被 `Stolen Code` 的字节。

最后 `close up.exe file handle` 用一些进程管理器:`http://www.sysinternals.com/files/procexpnt.zip` 这个选择 `up` 进程,然后查看 `handle`,把 `up.exe` 的 `handle` 给关闭掉。

现在即可用`ＯＤ`的脱壳插件直接脱壳，方式１或方式２脱壳均可正常运行

--------------------------------------------------------------------------------

## 【破解总结】

此文章算是 `peaceclub` 兄的\[真正快速脱壳方法\]的又一实例，文中有多处基本引用了原文的内容。在此向 `peaceclub` 兄还有 `fly`,`csjwaman` 等人表示感谢。谢谢一直以来对我的指点和帮助。

--------------------------------------------------------------------------------

## 【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`

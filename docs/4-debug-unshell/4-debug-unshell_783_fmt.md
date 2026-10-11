
# Obsidium 1.0.0.61

> 原文：[`www.pediy.com/kssd/pediy06/pediy6826.htm`](https://www.pediy.com/kssd/pediy06/pediy6826.htm)

**【前言】**：庆祝 `DFCG` 启用新论坛和新域名 [`chinadfcg.126.com`](http://chinadfcg.126.com)

**【软件名称】**：用 `Obsidium` 1.0.0.61(破解版)加的壳!

**【下载页面】**：[`yock.8866.org/down/O-Angela01.rar`](http://yock.8866.org/down/O-Angela01.rar)

**【软件大小】**：(里面有加壳的程序,我 `DUMP` 出来的程序和我解密 `IAT` 后的程序)很小.

**【应用平台】**：`WIN9X`/`WINNT`/`WIN2K`/`WINXP`

**【软件简介】**：一个加密壳

**【软件限制】**：吃掉 `OEP` 的一些字节,把 `IAT` 也给换成自己的了!

**【文章作者】**：辉仔 `Yock`

**【作者声明】**：本人发表这篇文章只是为了学习和研究!!!请不用于商业、非法用途,读者看了文章后所做的事情与我无关,我也不会负责,请读者看了文章后三思而后行!(在这里最此软件的作者以万二分的歉意鞠躬...)

**【破解工具】**：`Ollydbg` 1.10Aen(作者终于出新版了!希望 `fly` 可以快快把他修改一下,可以避开仙剑和老王的壳里面 `D` 陷阱) `HH-ImportREC1.42`(感谢汉化人和这个软件的作者!)

-----------------------------------------------

**【过程】**：这个壳非常棒,我个人也极度喜欢.他很温柔,很有心思,也很......

我是用看学老大制作的一个 `CrackMe` 来试验的.`OHOH`

废话不说了,开始吧

用 `OD` 载入,忽略所有异常(除内存)

然后用 `OD` 隐藏插件把检查调试器的函数给做一下手脚(也要感谢一下这个作者.没有这个东西.我们这个脱壳人可人晕晕的!)

差点忘记一件很重要的事情.运行娃娃最新做的`[WIN 优化大师]`的注册机------其实要的是这个音乐.而且是不段循环播放 `D` :`:`

现在可以按 `F9` 冲了!啦啦啦.....

**第一次异常:**

```
00414F40    CD 01           INT     1
00414F42    40              INC     EAX
00414F43    40              INC     EAX
00414F44    EB 03           JMP     SHORT 00414F49            O-Angela.00414F49
```

**第二次异常:**

```
004153AD    A5              MOVS    DWORD PTR ES:[EDI], DWORD PTR [E>
004153AE    EB 03           JMP     SHORT 004153B3            O-Angela.004153B3
```

**第三次异常:**

```
003A0AB9    CD 01           INT     1
003A0ABB    40              INC     EAX
003A0ABC    40              INC     EAX
```

**第四次异常:**

```
003A0E12    800406 00       ADD     BYTE PTR [ESI+EAX], 0
003A0E16    EB 02           JMP     SHORT 003A0E1A
```

**第五次异常:**

```
003A11A6    8B00            MOV     EAX, [EAX]
003A11A8    EB 02           JMP     SHORT 003A11AC
```

**第六次异常:**

```
003A1646    8B00            MOV     EAX, [EAX]
003A1648    EB 02           JMP     SHORT 003A164C
```

**第七次异常:**

```
003A1864    CD 68           INT     68
003A1866    EB 04           JMP     SHORT 003A186C
```

**第八次异常:**

```
003A1CE1    8B02            MOV     EAX, [EDX]
003A1CE3    EB 02           JMP     SHORT 003A1CE7
```

**第九次异常:**

```
003A33EA    0FB753 06       MOVZX   EDX, WORD PTR [EBX+6]
003A33EE    EB 01           JMP     SHORT 003A33F1
```

**第十次异常:**

```
003A3A7F    800406 00       ADD     BYTE PTR [ESI+EAX], 0
003A3A83    EB 03           JMP     SHORT 003A3A88
```

**第十一次异常:**

```
003A4C57    A5              MOVS    DWORD PTR ES:[EDI], DWORD PTR [E>
003A4C58    EB 03           JMP     SHORT 003A4C5D
```

**最后一次异常:**

```
003A4EC9    CD 01           INT     1
003A4ECB    40              INC     EAX
003A4ECC    40              INC     EAX
003A4ECD    EB 04           JMP     SHORT 003A4ED3
```

-----------------------------------------------

在最后一次异常的时候按 `Alt+m` 打开`[内存镜像]`,然后找到 `401000`-`XXXXXXXX` 这个块,在这个块上面下内存访问断点!(按一下 `F2`)

接着咱们按 `Ctrl+F9` 来通过最后一次异常来到下面:

```
00401E81      FF            DB      FF

//来到了这里!

00401E82      15            DB      15
00401E83      3C            DB      3C                          CHAR '<'

......
......
```

-----------------------------------------------

然后我们在这些看不懂的代码上按 `OD` 强大的功能------右键菜单

接着找到`[偶个人喜欢叫他做照妖镜]` `Analysis`------>`Analyse code` (也就是 `Ctrl+A`)

这些看不懂的代码马上现形,呵呵看你这个妖精跑哪里去!

```
00401E5B      BF            DB      BF
00401E5C      72            DB      72                          CHAR 'r'
00401E5D      F6            DB      F6
00401E5E      90            NOP
00401E5F      7A            DB      7A                          CHAR 'z'
00401E60      CE            DB      CE
00401E61      B3            DB      B3
00401E62      D1            DB      D1
00401E63      2D            DB      2D                          CHAR '-'
00401E64      62            DB      62                          CHAR 'b'
00401E65      62            DB      62                          CHAR 'b'
00401E66      5C            DB      5C                          CHAR '\'
00401E67      23            DB      23                          CHAR '#'
00401E68      11            DB      11
00401E69      61            DB      61                          CHAR 'a'
00401E6A      C7            DB      C7
00401E6B      3D            DB      3D                          CHAR '='
00401E6C      94            DB      94
00401E6D      19            DB      19
00401E6E      EF            DB      EF
00401E6F      8A            DB      8A
00401E70      40            DB      40                          CHAR '@'
00401E71      87            DB      87
00401E72      8B            DB      8B
00401E73      7D            DB      7D                          CHAR '}'
00401E74      CB            DB      CB
00401E75      26            DB      26                          CHAR '&'
00401E76   . 0C E8          OR      AL, 0E8
00401E78   . F9            STC
00401E79   . F7C6 DE44894A TEST    ESI, 4A8944DE
00401E7F   . 1C 0E         SBB     AL, 0E
00401E81   . FF15 3C504000 CALL    [40503C]

//我们停在这里!往上看

//很正常,现在的加密可都在 OEP 处动手脚,搞得一个漂亮得程序乱七八糟

00401E87   . 33D2          XOR     EDX, EDX
00401E89   . 8AD4          MOV     DL, AH
00401E8B   . 8915 28654000 MOV     [406528], EDX
00401E91   . 8BC8          MOV     ECX, EAX
00401E93   . 81E1 FF000000 AND     ECX, 0FF
00401E99   . 890D 24654000 MOV     [406524], ECX
00401E9F   . C1E1 08       SHL     ECX, 8
```

-----------------------------------------------

我们怎么把 `OEP` 的代码弄回来呢?看看堆栈吧!

```
0012FF90   003A5968
0012FF94   0000000A
0012FF98   003A57D8
0012FF9C   000001BB
0012FFA0   0012DB30
0012FFA4   004A4634
0012FFA8   0012FF4C
0012FFAC   0012FFC0
0012FFB0   0012FFE0  Pointer to next SEH record
0012FFB4   00402990  SE  handler

//这里的代码记下来!

0012FFB8   004050C8  O-Angela.004050C8

//这里的代码记下来!
```

呵呵,现在简单了,找一个 `C++` 6.0 的程序,复制 `OEP` 的代码回来!

```
PUSH    EBP
MOV     EBP, ESP
PUSH    -1
PUSH    XXXXXX
//这里就是堆栈里面的值,贴下来吧
PUSH    XXXXXX
//同上
MOV     EAX, FS:[0]
PUSH    EAX
MOV     FS:[0], ESP
SUB     ESP, 58
PUSH    EBX
PUSH    ESI
PUSH    EDI
MOV     [EBP-18], ESP
```

-----------------------------------------------

那么我们要的 `OEP` 的这些代码复原后的一样如下(接着就是用 `LordPE` 把她弄出来咯):

```
00401E5B  >/$  55            PUSH    EBP
00401E5C  |.   8BEC         MOV     EBP, ESP
00401E5E  |.   6A FF        PUSH    -1
00401E60  |.   68 C8504000  PUSH    4050C8
00401E65  |.   68 90294000  PUSH    402990
00401E6A  |.   64:A1 0000000>MOV     EAX, FS:[0]
00401E70  |.   50            PUSH    EAX
00401E71  |.   64:8925 00000>MOV     FS:[0], ESP
00401E78  |.   83EC 58       SUB     ESP, 58
00401E7B  |.   53            PUSH    EBX
00401E7C  |.   56            PUSH    ESI
00401E7D  |.   57            PUSH    EDI
00401E7E  |.   8965 E8       MOV     [EBP-18], ESP
00401E81  |.   FF15 3C504000 CALL    [40503C]
```

(如果是 98 记事本就不能这样找 `OEP`,比较麻烦,到现在为止还没有想到什么好办法)

=============================================================================================================

找了点时间,写下了下面这些(`IAT` 解密)

`Obsidium` 这个壳实在是麻烦.

我们开始吧:

下面是我通过自己所有的办法找到的!(前提要知道 `IAT` 的原是位置,这个要靠 `You` 自己琢磨,其实看看那些 `CALL[XXXXXX]`这样的代码就知道啦!)

通过内存断点,我们来大下面这里!

```
003A4035    C8 000000       ENTER   0, 0
003A4039    53              PUSH    EBX
003A403A    56              PUSH    ESI
003A403B    57              PUSH    EDI
003A403C    8B5D 14         MOV     EBX, [EBP+14]
003A403F    8B75 10         MOV     ESI, [EBP+10]
003A4042    8B7D 0C         MOV     EDI, [EBP+C]
003A4045    8B5B 04         MOV     EBX, [EBX+4]
003A4048    66:F706 2000    TEST    WORD PTR [ESI], 20
//注意这里,改成
//TEST    WORD PTR [ESI], 8
003A404D    74 46           JE      SHORT 003A4095
//注意这里,改成
```



## 003A404F

```
// JnE       SHORT 003A4095  

003A404F     66:F706 0200     TEST     WORD PTR [ESI], 2  

003A4054     75 1F           JNZ      SHORT 003A4075  

003A4056     66:C706 0400    MOV      WORD PTR [ESI], 4  

003A405B     8B45 14         MOV      EAX, [EBP+14]  

003A405E     6A 01           PUSH     1  

003A4060     6A 00           PUSH     0  

003A4062     FF76 04        PUSH     DWORD PTR [ESI+4]  

003A4065     6A 00           PUSH     0  

003A4067     FF75 18        PUSH     DWORD PTR [EBP+18]  

003A406A     FF50 40        CALL     [EAX+40]  

//密码学算法(烦),计算正确函数指针  

003A406D     85C0           TEST     EAX, EAX  

003A406F     74 39          JE       SHORT 003A40AA  

//注意这里,改成  

//JE       SHORT 003A4095  

003A4071     8907           MOV      [EDI], EAX  

003A4073     EB 20          JMP      SHORT 003A4095  

003A4075     66:C706 0400   MOV      WORD PTR [ESI], 4  

003A407A     8B45 14        MOV      EAX, [EBP+14]  

003A407D     0FB756 02      MOVZX    EDX, WORD PTR [ESI+2]  

003A4081     6A 01          PUSH     1  

003A4083     52             PUSH     EDX  

003A4084     6A 00          PUSH     0  

003A4086     FF76 04        PUSH     DWORD PTR [ESI+4]  

003A4089     FF75 18        PUSH     DWORD PTR [EBP+18]  

003A408C     FF50 40        CALL     [EAX+40]  

//密码学算法(烦),计算正确函数指针  

003A408F     85C0           TEST     EAX, EAX  

003A4091     74 17         JE       SHORT 003A40AA  

//注意这里,改成  

//JE       SHORT 003A4095  

003A4093     8907           MOV      [EDI], EAX  

003A4095     83C6 08        ADD      ESI, 8  

003A4098     83C7 04        ADD      EDI, 4  

003A409B     FF4D 08        DEC      DWORD PTR [EBP+8]  

003A409E    ^ 75 A8         JNZ      SHORT 003A4048  

003A40A0     33C0           XOR      EAX, EAX  

003A40A2     40             INC      EAX  

003A40A3     5F             POP      EDI  

003A40A4     5E             POP      ESI  

003A40A5     5B             POP      EBX  

003A40A6     C9             LEAVE    

003A40A7     C2 1400        RETN     14
```

上面是这个壳为了防止某些函数指针不能加密的解密器,我修改之后就变成了解密所有函数指针的解密了 `^_^`

## 003A40B3

```
003A40B3     C8 000000      ENTER    0, 0  

003A40B7     53             PUSH     EBX  

003A40B8     56             PUSH     ESI  

003A40B9     57             PUSH     EDI  

003A40BA     8B5D 14        MOV      EBX, [EBP+14]  

003A40BD     8B75 10        MOV      ESI, [EBP+10]  

003A40C0     8B7D 0C        MOV      EDI, [EBP+C]  

003A40C3     8B5B 04        MOV      EBX, [EBX+4]  

003A40C6     66:833E 08     CMP      WORD PTR [ESI], 8  

003A40CA     0F85 97000000  JNZ      003A4167  

003A40D0     8B46 04        MOV      EAX, [ESI+4]  

003A40D3     83F8 00        CMP      EAX, 0  

003A40D6     74 44          JE       SHORT 003A411C  

003A40D8     83F8 01        CMP      EAX, 1  

003A40DB     74 4E          JE       SHORT 003A412B  

003A40DD     83F8 02        CMP      EAX, 2  

003A40E0     74 58          JE       SHORT 003A413A  

003A40E2     83F8 03        CMP      EAX, 3  

003A40E5     74 11          JE       SHORT 003A40F8  

003A40E7     83F8 04        CMP      EAX, 4  

003A40EA     75 7B          JNZ      SHORT 003A4167  

003A40EC     8B45 18        MOV      EAX, [EBP+18]  

003A40EF     05 F2199C00    ADD      EAX, 9C19F2  

003A40F4     8907           MOV      [EDI], EAX  

003A40F6     EB 6F          JMP      SHORT 003A4167  

003A40F8     8B45 14        MOV      EAX, [EBP+14]  

003A40FB     68 C5B1662D    PUSH     2D66B1C5  

003A4100     6A 00          PUSH     0  

003A4102     FF50 20        CALL     [EAX+20]  

//这里很重要,一些关键指针是否能解密就要靠这里了!  

//进去看看!  

003A4105     50             PUSH     EAX  

003A4106     53             PUSH     EBX  

003A4107     E8 B3030000    CALL     003A44BF  

003A410C     53             PUSH     EBX  

003A410D     E8 22030000    CALL     003A4434  

003A4112     8BCB           MOV      ECX, EBX  

003A4114     8D5C03 01      LEA      EBX, [EBX+EAX+1]  

003A4118     8BC1           MOV      EAX, ECX  

003A411A     EB 2B          JMP      SHORT 003A4147  

003A411C     8B45 14        MOV      EAX, [EBP+14]  

003A411F     68 0F1ACF4C    PUSH     4CCF1A0F  

003A4124     6A 00          PUSH     0  

003A4126     FF50 20        CALL     [EAX+20]  

//这里很重要,一些关键指针是否能解密就要靠这里了!  

//进去看看!同上  

003A4129     EB 1C          JMP      SHORT 003A4147  

003A412B     8B45 14        MOV      EAX, [EBP+14]  

003A412E     68 A41A86D0    PUSH     D0861AA4  

003A4133     6A 00          PUSH     0  

003A4135     FF50 20        CALL     [EAX+20]  

//这里很重要,一些关键指针是否能解密就要靠这里了!  

//进去看看!同上  

003A4138     EB 0D          JMP      SHORT 003A4147  

003A413A     8B45 14        MOV      EAX, [EBP+14]  

003A413D     68 E313B41D    PUSH     1DB413E3  

003A4142     6A 00          PUSH     0  

003A4144     FF50 20        CALL     [EAX+20]  

//这里很重要,一些关键指针是否能解密就要靠这里了!  

//进去看看!同上  

003A4147     C603 B8        MOV      BYTE PTR [EBX], 0B8  

003A414A     8943 01        MOV      [EBX+1], EAX  

003A414D     8B55 18        MOV      EDX, [EBP+18]  

003A4150     81C2 5F189C00  ADD      EDX, 9C185F  

003A4156     8D43 0A        LEA      EAX, [EBX+A]  

003A4159     2BD0           SUB      EDX, EAX  

003A415B     C643 05 E9     MOV      BYTE PTR [EBX+5], 0E9  

003A415F     8953 06        MOV      [EBX+6], EDX  

003A4162     891F           MOV      [EDI], EBX  

//注意这里,改成  

//NOP 掉  

003A4164     83C3 0A        ADD      EBX, 0A  

003A4167     83C6 08        ADD      ESI, 8  

003A416A     83C7 04        ADD      EDI, 4  

003A416D     8B45 14        MOV      EAX, [EBP+14]  

003A4170     8958 04        MOV      [EAX+4], EBX  

003A4173     FF4D 08        DEC      DWORD PTR [EBP+8]  

003A4176    ^ 0F85 4AFFFFFF  JNZ      003A40C6  

003A417C     5F             POP      EDI  

003A417D     5E             POP      ESI  

003A417E     5B             POP      EBX  

003A417F     C9             LEAVE    

003A4180     C2 1400        RETN     14
```

`//这里的几个 Call 都是进入到下面 0041411C 这个地址的,`

## 0041411C

```
0041411C     60             PUSHAD   

0041411D     E8 00000000    CALL     00414122  

00414122     5B             POP      EBX  

00414123     81EB B8569C00  SUB      EBX, 9C56B8  

00414129     8B9B AE569C00  MOV      EBX, [EBX+9C56AE]  

0041412F     8B4424 24      MOV      EAX, [ESP+24]  

00414133     33C9           XOR      ECX, ECX  

00414135     8B4483 48      MOV      EAX, [EBX+EAX*4+48]  

00414139     8B5424 28      MOV      EDX, [ESP+28]  

0041413D     51             PUSH     ECX  

0041413E     51             PUSH     ECX  

0041413F     51             PUSH     ECX  

00414140     52             PUSH     EDX  

00414141     50             PUSH     EAX  

00414142     FF53 40        CALL     [EBX+40]  

//最好在这里下一个硬件断点,留意每次的这个值,如果在这个 CALL 执行之后 EDI 的值不是 IAT 的地址或是不指向内存的话就要看 EAX 的值来决定新的起点!  

//十分重要!  

//我会再最后总结的时候做一个例子出来,希望你能看明白!  

-----------------  

00414145     85C0           TEST     EAX, EAX  

00414147     0F84 8B000000  JE       004141D8  

//这里改成  

//00414145     8907           MOV      [EDI], EAX//呵呵,我们把正确的指针放进去正确的地址!  

//00414147     90             NOP    

//00414148     90             NOP    

//00414149     90             NOP    

//0041414A     90             NOP    

//0041414B     90             NOP    

//0041414C     90             NOP    

------------------  

0041414D     8BF0           MOV      ESI, EAX  

0041414F     83EC 28        SUB      ESP, 28  

00414152     8BFC           MOV      EDI, ESP  

00414154     C74424 4C 02000>MOV      DWORD PTR [ESP+4C], 2  

0041415C     897424 50      MOV      [ESP+50], ESI  

00414160     803E C2        CMP      BYTE PTR [ESI], 0C2  

00414163     74 2D          JE       SHORT 00414192  

00414165     803E C3        CMP      BYTE PTR [ESI], 0C3  

00414168     74 28          JE       SHORT 00414192  

0041416A     56             PUSH     ESI
```



```
0041416B    FF53 24          CALL    [EBX+24]

0041416E    83F8 FF         CMP     EAX, -1

00414171    74 47           JE      SHORT 004141BA

00414173    F7C2 00060000   TEST    EDX, 600

00414179    75 17           JNZ     SHORT 00414192

0041417B    56              PUSH    ESI

0041417C    57              PUSH    EDI

0041417D    8BC8            MOV     ECX, EAX

0041417F    F3:A4           REP     MOVS BYTE PTR ES:[EDI], BYTE PTR>

00414181    5F              POP     EDI

00414182    5E              POP     ESI

00414183    803F CC         CMP     BYTE PTR [EDI], 0CC

00414186    74 38           JE      SHORT 004141C0

00414188    03F0            ADD     ESI, EAX

0041418A    03F8            ADD     EDI, EAX

0041418C    FF4C24 4C       DEC     DWORD PTR [ESP+4C]

00414190   ^ 75 CE          JNZ     SHORT 00414160

00414192    C607 E9         MOV     BYTE PTR [EDI], 0E9

00414195    8BC6            MOV     EAX, ESI

00414197    2BC7            SUB     EAX, EDI

00414199    83E8 05         SUB     EAX, 5

0041419C    8947 01         MOV     [EDI+1], EAX

0041419F    8BC4            MOV     EAX, ESP

004141A1    8B4C24 48       MOV     ECX, [ESP+48]

004141A5    8039 CC         CMP     BYTE PTR [ECX], 0CC

004141A8    74 16           JE      SHORT 004141C0

004141AA    894C24 50       MOV     [ESP+50], ECX

004141AE    894424 4C       MOV     [ESP+4C], EAX

004141B2    83C4 28         ADD     ESP, 28

004141B5    61              POPAD

004141B6    83C4 04         ADD     ESP, 4

004141B9    C3              RETN
```

### 小小总结

`00414142` 这个地址的 `Call` 执行之后，如果 `EAX=0` 的话，就要新建起源到 `004141D8` 这个地址。

如果 `EDI` 的值不是指向 `IAT` 或是无效的话就要新建起源到 `0041414D` 这个地址。

打好所有的补丁之后，我们现在就可以按 `F9` 咯，然后在最后一次异常对上面的 `OEP` 下断，接着就是把 `OEP` 的代码补上，然后就可以 `DUMP`。

---

#### 【总结】

好了，说了那么久，终于把这个东西的身体弄出个样！

这个时候我们打开 `ImportREC` 输入 `OEP=00401E5B`，很快就可以找到所有解密出来的函数指针了！OHOH

把所有无效的指针都剪掉，接下来的应该不用我教了吧。

修复好 `IAT` 之后我们运行一下看看。啦啦啦...

现在 `1.10` 版的基本一样，只是在解密器加了一些花指令。Easy...

最后在这里真心感谢你花了那么多时间看这篇文章！谢谢了...

`2044.1.28` 晚

by 辉仔 Yock

`4dd466d0e1e0cc123afbe504da2462ca`

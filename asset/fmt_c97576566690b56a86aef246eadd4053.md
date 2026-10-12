
# 谈谈 VB 程序的破解

> 原文：[`www.pediy.com/kssd/pediy05/pediy50244.htm`](https://www.pediy.com/kssd/pediy05/pediy50244.htm)

## 谈谈 VB 程序的破解

有很多人说破解 VB 程序的利器是 `SmartCheck`, 我不敢苟同, 因为我一直以为, 只要有一个调试器就能完成所有的破解才是我向往的境界. 当然这不一定最简便, 但无疑是最有效的.

这次就说说 VB 程序的破解, 当然, 我不知道是否有其它可行的途径, 我只知道, 通过调试器就能完成一个破解.

示例程序: 随便写段 VB 代码, 然后编译为本机代码 (`P-Code` 不在本文的讨论范围之内, 我也发现了一些相关的规律, 还在研究中)

```vb
Private Sub Form_Load()
    If ax = 0 Then
        MsgBox ("hi!")
        End
    Else
        MsgBox ("ok!")
        End
    End If
End Sub
```

用 `TR` 载入

```asm
0187:00401166 FF2570104000    JMP     NEAR [00401070]      //从 00401171 到了这里.
0187:0040116C 68A0124000      PUSH    DWORD 004012A0       //载入后停在这里
0187:00401171 E8F0FFFFFF      CALL    MSVBVM60!ThunRTMain  //这个 Call 一定要进入
0187:00401176 0000            ADD     [EAX],AL
0187:00401178 0000            ADD     [EAX],AL
0187:0040117A 0000            ADD     [EAX],AL
......
```

`MSVBVM60!ThunRTMain`

```asm
0187:6600DE22 55              PUSH    EBP
0187:6600DE23 8BEC           MOV     EBP,ESP
0187:6600DE25 6AFF           PUSH    BYTE -01
0187:6600DE27 6898980166     PUSH    DWORD 66019898
0187:6600DE2C 6871EF0E66     PUSH    DWORD 660EEF71
0187:6600DE31 64A100000000   MOV     EAX,[FS:00]
0187:6600DE37 50             PUSH    EAX
0187:6600DE38 64892500000000 MOV     [FS:00],ESP
0187:6600DE3F 51             PUSH    ECX
0187:6600DE40 51             PUSH    ECX
0187:6600DE41 83EC4C         SUB     ESP,BYTE +4C
0187:6600DE44 53             PUSH    EBX
0187:6600DE45 56             PUSH    ESI
0187:6600DE46 57             PUSH    EDI
0187:6600DE47 8965E8         MOV     [EBP-18],ESP
0187:6600DE4A 8B7508         MOV     ESI,[EBP+08]
0187:6600DE4D 8935DCF71066   MOV     [6610F7DC],ESI
0187:6600DE53 8365FC00       AND     DWORD [EBP-04],BYTE +00
0187:6600DE57 8D45A0         LEA     EAX,[EBP-60]
0187:6600DE5A 50             PUSH    EAX
0187:6600DE5B FF1518110066   CALL    KERNEL32!GetStartupInfoA
0187:6600DE61 0FB745D0       MOVZX   EAX,WORD [EBP-30]
0187:6600DE65 A3D8F71066     MOV     [6610F7D8],EAX
0187:6600DE6A FF35CCF61066   PUSH    DWORD [6610F6CC]
0187:6600DE70 56             PUSH    ESI
0187:6600DE71 BE70F41066     MOV     ESI,6610F470
0187:6600DE76 8BCE           MOV     ECX,ESI
0187:6600DE78 E860000000     CALL    6600DEDD          //这个 Call 一定要进去哦.
......
```

(其实不要看在 `DLL` 中转, 其实并不难, 很快就可以返回到程序的领空, 我的目的就是要在程序运行前进入程序领空)

```asm
0187:******** FFD0           CALL    EAX               //最后到了这里,ok,返回程序领空了哦!
......
0187:0040146C 816C240433000000 SUB     DWORD [ESP+04],33    //停在这里,这里已经是程序领空了哦!
0187:00401474 E9B7040000      JMP     00401930          //跳到什么地方,你看下面吧!
0187:00401479 0000            ADD     [EAX],AL
......
0187:00401930 55             PUSH    EBP               //看看象什么?是不是和汇编,delphi 程序开始的地方一样啊?我们终于来到 VB 程序真正开始的地方了.
0187:00401931 8BEC           MOV     EBP,ESP
0187:00401933 83EC0C         SUB     ESP,BYTE +0C
0187:00401936 68A6104000     PUSH    DWORD 004010A6
0187:0040193B 64A100000000   MOV     EAX,[FS:00]
......
0187:00401991 C7458C02800000 MOV     DWORD [EBP-74],8002
0187:00401998 FF1538104000   CALL    MSVBVM60!__vbaVarTstEq
0187:0040199E 6685C0         TEST    AX,AX
0187:004019A1 B904000280     MOV     ECX,80020004
0187:004019A6 B80A000000     MOV     EAX,0A
0187:004019AB 894DA4         MOV     [EBP-5C],ECX
0187:004019AE 89459C         MOV     [EBP-64],EAX
0187:004019B1 894DB4         MOV     [EBP-4C],ECX
0187:004019B4 8945AC         MOV     [EBP-54],EAX
0187:004019B7 894DC4         MOV     [EBP-3C],ECX
0187:004019BA 8945BC         MOV     [EBP-44],EAX
0187:004019BD 7443           JZ      00401A02          (NO JUMP)   //知道这里吗,就是我代码中的判断,这更加肯定了,在程序代码运行以前我就返回程序领空了.
0187:004019BF 8D558C         LEA     EDX,[EBP-74]
0187:004019C2 8D4DCC         LEA     ECX,[EBP-34]
0187:004019C5 C74594F0154000 MOV     DWORD [EBP-6C],004015F0
0187:004019CC C7458C08000000 MOV     DWORD [EBP-74],08
0187:004019D3 FF1574104000   CALL    MSVBVM60!__vbaVarDup
0187:004019D9 8D559C         LEA     EDX,[EBP-64]
0187:004019DC 8D45AC         LEA     EAX,[EBP-54]
0187:004019DF 52             PUSH    EDX
0187:004019E0 8D4DBC         LEA     ECX,[EBP-44]
0187:004019E3 50             PUSH    EAX
0187:004019E4 51             PUSH    ECX
0187:004019E5 8D55CC         LEA     EDX,[EBP-34]
```

## 后记

现在很多程序都在程序运行时判断是否注册, 我相信我找到的这个是个可行的办法, 能够在程序代码开始前进入程序领空. 就可以知道它到底干了些什么. 然后可以爆破或其它. 相信很多大虾都不屑一顾, 我想告诉大家的是一个通用的方法. 不知道你注意上面的这么一段了吗?

```asm
0187:******** FFD0           CALL    EAX               //只要每次在这里下断,就能把所有的 VB 程序在运行前断下来(包括 P-code,当然 P-code 还有更简单的办法在程序运行前断下来),接着就能来到程序开始的地方.程序不就任你鱼肉了?
```

我在几个 VB 编写的程序上试了一下, 都能用此法破解成功, 例如: 沐风网页三叉戟等.

flyfancy

`http://flyfancy.126.com`

## 续篇

上次写了 VB 程序的破解一文, 这次因为发现那个方法不是通用的, 所以特别写一篇续. 其实我这 2 篇文章并不是专门用于破解在启动时检测的程序的, 我只是想说一个方法, 利用 VB 中的事件, 于是轻松找到程序的关键, 各位要是有兴趣可以跟一下, 很简单. 废话少说, 运行 `VB6`, 新建一个工程, 然后添加一个模块, 双击模块, 删除 `Form`, 再输入:

```vb
Sub Main
MsgBox ("test!")
End
End Sub
```

然后编译成本机代码, 在这种情况下, 我以前说的办法就没有用了, 大家还是跟我来吧.

还是 `TRW` 载入, 然后慢慢跟就到了这里.

```asm
......
0187:6600DE5B FF1518110066   CALL    KERNEL32!GetStartupInfoA
0187:6600DE61 0FB745D0       MOVZX   EAX,WORD [EBP-30]
0187:6600DE65 A3D8F71066     MOV     [6610F7D8],EAX
0187:6600DE6A FF35CCF61066   PUSH    DWORD [6610F6CC]
0187:6600DE70 56             PUSH    ESI
0187:6600DE71 BE70F41066     MOV     ESI,6610F470
0187:6600DE76 8BCE           MOV     ECX,ESI
0187:6600DE78 E860000000     CALL    6600DEDD          //进去
......
(省略过程)
......
0187:******** FF9694000000   CALL    NEAR [ESI+94]     //这里 F8 进去就返回程序领空了,我们又成功的停在了 VB 代码开始的地方.
0187:******** 8D45D4         LEA     EAX,[EBP-2C]
0187:******** 50             PUSH    EAX
0187:******** E807030000     CALL    66014710
0187:******** 6A01           PUSH    BYTE +01
0187:******** 58             POP     EAX
0187:******** 5F             POP     EDI
0187:******** 5E             POP     ESI
0187:******** C9             LEAVE
0187:******** C20400         RET     04
......
0187:00401628 6A00           PUSH    BYTE +00
0187:0040162A 50             PUSH    EAX
0187:0040162B FF151C104000   CALL    MSVBVM60!rtcMsgBox    //这里就是 MsgBox 了
0187:00401631 8D4DB0         LEA     ECX,[EBP-50]
0187:00401634 8D55C0         LEA     EDX,[EBP-40]
0187:00401637 51             PUSH    ECX
0187:00401638 8D45D0         LEA     EAX,[EBP-30]
0187:0040163B 52             PUSH    EDX
0187:0040163C 8D4DE0         LEA     ECX,[EBP-20]
0187:0040163F 50             PUSH    EAX
0187:00401640 51             PUSH    ECX
0187:00401641 6A04           PUSH    BYTE +04
0187:00401643 FF1508104000   CALL    MSVBVM60!__vbaFreeVarList
0187:00401649 83C414         ADD     ESP,BYTE +14
0187:0040164C FF150C104000   CALL    MSVBVM60!__vbaEnd     //然后程序 End
0187:00401652 6876164000     PUSH    DWORD 00401676
0187:00401657 EB1C           JMP     SHORT 00401675
```

## 后记

还有一种情况, 是某个 `Crackme`, 是用 `VB6` 编写的, 我用了 2 种办法都不能断下来 (其实是作者不按常理出牌). 自己新建一个工程, 然后编译为本机代码, 试试看, 还能不能用上面的办法断下来? 也就是说, 在程序启动时 (或检验注册码前) 没有任何的事件, 真的是很搞笑的一件事, 这样的程序, 跟踪只会在 `Msvbvm60.dll` 中, 然后用 `ShowWindow` 显示窗口, 也就是说程序根本不是在启动时进行检测的 (或者说检测时没有产生任何事件? 可能吗?).

我以为按我的方法, 只要在 VB 程序中使用了事件或模块, 无论是程序刚启动或已运行, 只要用 `TRW` 在这 2 处下了断点, 都能轻易的找到程序的关键处. 如果各位有什么意见和看法, 还请指正

`4dd466d0e1e0cc123afbe504da2462ca`

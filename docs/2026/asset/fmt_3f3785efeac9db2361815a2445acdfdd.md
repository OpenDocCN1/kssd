
# EXE ReFactor 0.2 详细分析(一)

## 一. 壳的特点

这个壳是我见过的比较有特点的壳, 很多地方值的我学习,  希望作者多交流.

当然这个壳有些代码效率有点低, 把一些数据在heap里搬来搬去, 可以改进一下.

第一个特点是原程序的 Section 完全消失, Loader 动态创建并解压, 给 Dump 带来一点麻烦, 可能需要手动组装 PE

第二个特点是所有的 API 地址在内存中都是反的, 调用前先`NOT`

第三个特点是用全局变量保存`ESP`, `JMP`都是用`Call`实现的.

用这种方式来实现花指令倒是第一次见到.

```asm
053BC006    8925 CFEF3B05   MOV DWORD PTR DS:[53BEFCF],ESP
053BC00C    E8 F4350000     CALL EXERe.053BF605
053BF605    BE 80963B05     MOV ESI,EXERe.053B9680
053BF60A    E8 BAFEFFFF     CALL EXERe.053BF4C9
053BF4C9    8B25 CFEF3B05   MOV ESP,DWORD PTR DS:[53BEFCF]   // 恢复 ESP
```

第四个特点是壳会把 API 整个搬到 Heap 空间, Stole 非常厉害,  为了实现这个目的, 作者花了很多工夫, 导致程序启动有点慢.

第五个特点是 IAT 加密方式也比较特别.

第六个特点是壳的字符串加密方法

我们先写一段程序, 解密`53B0554 - 53B0870`的密文, 熟悉一下壳所用的加解密字符串方法, 以后就不再描述了.

```asm
053A34EF >    60            PUSHAD
053A34F0      BE 54053B05   MOV ESI,EXERe.053B0554
053A34F5      33C0          XOR EAX,EAX
053A34F7      66:8B06       MOV AX,WORD PTR DS:[ESI]
053A34FA      85C0          TEST EAX,EAX
053A34FC      74 0E         JE SHORT EXERe.053A350C
053A34FE      66:25 0F0F    AND AX,0F0F                    // AH, AL 的 高 4 位清零
053A3502      C0E4 04       SHL AH,4
053A3505      08E0          OR AL,AH                       // 把 AH, AL 的低 4 位组合在一起
053A3507      32E4          XOR AH,AH
053A3509      66:8906       MOV WORD PTR DS:[ESI],AX
053A350C      83C6 02       ADD ESI,2
053A350F      81FE 70083B05 CMP ESI,EXERe.053B0870
053A3515    ^ 7C E0         JL SHORT EXERe.053A34F7
053A3517      61            POPAD

60 BE 54 05 3B 05 33 C0 66 8B 06 85 C0 74 0E 66 25 0F 0F C0 E4 04 08 E0 32 E4 66 89 06 83 C6 02
81 FE 70 08 3B 05 7C E0 61
```

明文, 看看壳所用的 API,  还有6个 Section Name

```hex
053B0554  57 00 69 00 64 00 65 00 43 00 68 00 61 00 72 00  W.i.d.e.C.h.a.r.
053B0564  54 00 6F 00 4D 00 75 00 6C 00 74 00 69 00 42 00  T.o.M.u.l.t.i.B.
053B0574  79 00 74 00 65 00 00 00 48 00 65 00 61 00 70 00  y.t.e...H.e.a.p.
053B0584  41 00 6C 00 6C 00 6F 00 63 00 00 00 48 00 65 00  A.l.l.o.c...H.e.
053B0594  61 00 70 00 46 00 72 00 65 00 65 00 00 00 00 00  a.p.F.r.e.e.....
053B05A4  53 00 6C 00 65 00 65 00 70 00 00 00 43 00 6C 00  S.l.e.e.p...C.l.
053B05B4  6F 00 73 00 65 00 48 00 61 00 6E 00 64 00 6C 00  o.s.e.H.a.n.d.l.
053B05C4  65 00 00 00 4C 00 6F 00 61 00 64 00 4C 00 69 00  e...L.o.a.d.L.i.
053B05D4  62 00 72 00 61 00 72 00 79 00 41 00 00 00 00 00  b.r.a.r.y.A.....
053B05E4  52 00 65 00 61 00 64 00 46 00 69 00 6C 00 65 00  R.e.a.d.F.i.l.e.
053B05F4  00 00 00 00 53 00 65 00 74 00 46 00 69 00 6C 00  ....S.e.t.F.i.l.
053B0604  65 00 50 00 6F 00 69 00 6E 00 74 00 65 00 72 00  e.P.o.i.n.t.e.r.
053B0614  00 00 00 00 43 00 72 00 65 00 61 00 74 00 65 00  ....C.r.e.a.t.e.
053B0624  46 00 69 00 6C 00 65 00 57 00 00 00 43 00 72 00  F.i.l.e.W...C.r.
053B0634  65 00 61 00 74 00 65 00 54 00 68 00 72 00 65 00  e.a.t.e.T.h.r.e.
053B0644  61 00 64 00 00 00 00 00 56 00 69 00 72 00 74 00  a.d.....V.i.r.t.
053B0654  75 00 61 00 6C 00 46 00 72 00 65 00 65 00 00 00  u.a.l.F.r.e.e...
053B0664  56 00 69 00 72 00 74 00 75 00 61 00 6C 00 41 00  V.i.r.t.u.a.l.A.
053B0674  6C 00 6C 00 6F 00 63 00 00 00 00 00 56 00 69 00  l.l.o.c.....V.i.
053B0684  72 00 74 00 75 00 61 00 6C 00 50 00 72 00 6F 00  r.t.u.a.l.P.r.o.
053B0694  74 00 65 00 63 00 74 00 00 00 00 00 52 00 65 00  t.e.c.t.....R.e.
053B06A4  61 00 64 00 50 00 72 00 6F 00 63 00 65 00 73 00  a.d.P.r.o.c.e.s.
053B06B4  73 00 4D 00 65 00 6D 00 6F 00 72 00 79 00 00 00  s.M.e.m.o.r.y...
053B06C4  57 00 72 00 69 00 74 00 65 00 50 00 72 00 6F 00  W.r.i.t.e.P.r.o.
053B06D4  63 00 65 00 73 00 73 00 4D 00 65 00 6D 00 6F 00  c.e.s.s.M.e.m.o.
053B06E4  72 00 79 00 00 00 00 00 4F 00 70 00 65 00 6E 00  r.y.....O.p.e.n.
053B06F4  50 00 72 00 6F 00 63 00 65 00 73 00 73 00 00 00  P.r.o.c.e.s.s...
053B0704  47 00 65 00 74 00 4D 00 6F 00 64 00 75 00 6C 00  G.e.t.M.o.d.u.l.
053B0714  65 00 48 00 61 00 6E 00 64 00 6C 00 65 00 41 00  e.H.a.n.d.l.e.A.
053B0724  00 00 00 00 47 00 65 00 74 00 43 00 75 00 72 00  ....G.e.t.C.u.r.
053B0734  72 00 65 00 6E 00 74 00 50 00 72 00 6F 00 63 00  r.e.n.t.P.r.o.c.
053B0744  65 00 73 00 73 00 49 00 64 00 00 00 47 00 65 00  e.s.s.I.d...G.e.
053B0754  74 00 50 00 72 00 6F 00 63 00 41 00 64 00 64 00  t.P.r.o.c.A.d.d.
053B0764  72 00 65 00 73 00 73 00 00 00 00 00 47 00 65 00  r.e.s.s.....G.e.
053B0774  74 00 54 00 69 00 63 00 6B 00 43 00 6F 00 75 00  t.T.i.c.k.C.o.u.
053B0784  6E 00 74 00 00 00 00 00 47 00 65 00 74 00 50 00  n.t.....G.e.t.P.
053B0794  72 00 6F 00 63 00 65 00 73 00 73 00 48 00 65 00  r.o.c.e.s.s.H.e.
053B07A4  61 00 70 00 00 00 00 00 49 00 73 00 42 00 61 00  a.p.....I.s.B.a.
053B07B4  64 00 52 00 65 00 61 00 64 00 50 00 74 00 72 00  d.R.e.a.d.P.t.r.
053B07C4  00 00 00 00 2E 00 74 00 65 00 78 00 74 00 00 00  ......t.e.x.t...
053B07D4  65 00 63 00 6F 00 64 00 65 00 00 00 70 00 61 00  e.c.o.d.e...p.a.
053B07E4  67 00 65 00 00 00 00 00 2E 00 72 00 73 00 72 00  g.e.......r.s.r.
053B07F4  63 00 00 00 2E 00 74 00 6C 00 73 00 00 00 00 00  c.....t.l.s.....
053B0804  2E 00 6F 00 72 00 70 00 63 00 00 00 52 00 65 00  ..o.r.p.c...R.e.
053B0814  73 00 75 00 6D 00 65 00 54 00 68 00 72 00 65 00  s.u.m.e.T.h.r.e.
053B0824  61 00 64 00 00 00 00 00 54 00 65 00 72 00 6D 00  a.d.....T.e.r.m.
053B0834  69 00 6E 00 61 00 74 00 65 00 50 00 72 00 6F 00  i.n.a.t.e.P.r.o.
053B0844  63 00 65 00 73 00 73 00 00 00 00 00 4D 00 65 00  c.e.s.s.....M.e.
053B0854  73 00 73 00 61 00 67 00 65 00 42 00 6F 00 78 00  s.s.a.g.e.B.o.x.
053B0864  45 00 78 00 57 00 00 00 00 00 00 00 00 00 00 00  E.x.W.
```

## 二.

重新来过

```asm
053A34EF > $  55            PUSH EBP
053A34F0   .  8BEC          MOV EBP,ESP
053A34F2   .  81EC 7C0B0000 SUB ESP,0B7C
053A34F8   .  53            PUSH EBX
053A34F9   .  56            PUSH ESI
053A34FA   .  57            PUSH EDI
053A34FB   .- E9 008B0100   JMP EXERe.053BC000

053BC000    FF15 00103A05   CALL DWORD PTR DS:[<&KERNEL32.GetCommandLine>
053BF605    BE 80963B05     MOV ESI,EXERe.053B9680                       ; 基址
053BF4CF    6A FF           PUSH -1
053BF517    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680( ECX 也是 Proc 53A67D0 的一个参数)
053BF505    FF15 E3F43B05   CALL DWORD PTR DS:[53BF4E3]                  ; EXERe.053A67D0
```

`proc 53A67D0(arg)`   获取 DLL, EXE 的 Section 信息

`ECX = buffer`

`arg = -1`               处理 `kernel32.dll`

`arg = -2`               处理 `EXE ImageBase`

`arg = -3`               处理 `ntdll.dll`

`arg = xxxxxxxx(DLL BaseAddress)`

对这个 DLL 的每个 Section 判断是不是下面的 SectionName

`.text`   code, 累加 Law Size

`ecode`   code, 累加 Law Size

`page`    code, 累加 Law Size

`.rsrc`   做标志

`.tls`    做标志

`.orpc`   code, 累加 Law Size

最后的结果

以 `arg = -1` 为例

```hex
053BA050  04 00 00 00    Section Num
053BA054  02 00 00 00    .rsrc 的序号(第三个区段)
053BA058  00 00 00 00    .tls  的序号(没有)
053BA05C  00 90 05 00    code 段 lawsize
053BA060  FF FF A8 83    Not BaseAddress
```

## 三.

回到主程序

```asm
053BF4EE    68 7C053B05     PUSH EXERe.053B057C
053BF5AD    8BCE            MOV ECX,ESI                                  ; EXERe.053B9680
053BF5BB    FF15 DEF43B05   CALL DWORD PTR DS:[53BF4DE]                  ; EXERe.053A7017
```

`proc 53A7017(arg)`   取 API　address 的一个函数

`arg = 加密的 API Name`

`ret = Not API Address`

```asm
053A7017  /$  55            PUSH EBP
053A7018  |.  8BEC          MOV EBP,ESP
053A701A  |.  81EC 00020000 SUB ESP,200
053A7020  |.  56            PUSH ESI
053A7021  |.  8BF1          MOV ESI,ECX
053A7023  |.  57            PUSH EDI
053A7024  |.  B9 80000000   MOV ECX,80
053A7029  |.  33C0          XOR EAX,EAX
053A702B  |.  8DBD 00FEFFFF LEA EDI,DWORD PTR SS:[EBP-200]                    // zero buffer
053A7031  |.  F3:AB         REP STOS DWORD PTR ES:[EDI]
053A7033  |.  8B4D 08       MOV ECX,DWORD PTR SS:[EBP+8]                      // arg
053A7036  |.  85C9          TEST ECX,ECX
053A7038  |.  75 25         JNZ SHORT EXERefac.053A705F
053A703A  |>  50            PUSH EAX                                          ; /Arg3 长度
053A703B  |.  8D85 00FEFFFF LEA EAX,DWORD PTR SS:[EBP-200]                    ; |
```


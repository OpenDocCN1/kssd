
# PECompact 的 OEP

> 原文：[`www.pediy.com/kssd/pediy05/pediy50127.htm`](https://www.pediy.com/kssd/pediy05/pediy50127.htm)

==============================================

该方法试用于 PECompactV1.71, V1.76, V1.82

具体看下面的示例

==============================================

这是 98 的 Calc.exe 使用 PECompact1.82 压缩过的结果

```
010153A0 >EB 06            JMP SHORT CALC.010153A8<==第一条指令
010153A2  68 E0190100      PUSH 119E0<======这就是程序的原始 OEP 的 RVA 地址
010153A7  C3               RETN
010153A8  9C               PUSHFD
010153A9  60               PUSHAD
010153AA  E8 02000000      CALL CALC.010153B1
```

==============================================

这是 98 的 Notepad.exe 使用 PECompact1.82 压缩过的结果

```
0040AB20 >EB 06            JMP SHORT NOTEPAD.0040AB28<==第一条指令
0040AB22  68 CC100000      PUSH 10CC<======这就是程序的原始 OEP 的 RVA 地址
0040AB27  C3               RETN
0040AB28  9C               PUSHFD
0040AB29  60               PUSHAD
0040AB2A  E8 02000000      CALL NOTEPAD.0040AB31
```

==============================================

这是使用 PECompact1.82 压缩过 LordPE.exe [rts]的结果

```
004239A0 >EB 06            JMP SHORT X.004239A8<==第一条指令
004239A2  68 103E0000      PUSH 3E10<======这就是程序的原始 OEP 的 RVA 地址
004239A7  C3               RETN
004239A8  9C               PUSHFD
004239A9  60               PUSHAD
004239AA  E8 02000000      CALL X.004239B1
004239AF  33C0             XOR EAX,EAX
```

==============================================

这是 PECompact1.82 自身的结果

```
0043E760 >EB 06            JMP SHORT PECOMPAC.0043E768<==第一条指令
0043E762  68 00E00300      PUSH 3E000<======这就是程序的原始 OEP 的 RVA 地址
0043E767  C3               RETN
0043E768  9C               PUSHFD
0043E769  60               PUSHAD
0043E76A  E8 02000000      CALL PECOMPAC.0043E771
0043E76F  33C0             XOR EAX,EAX
```

==============================================

这是 ODBG1.06 的结果

```
004F8200 >EB 06            JMP SHORT X.004F8208<==第一条指令
004F8202  68 00100000      PUSH 1000<======这就是程序的原始 OEP 的 RVA 地址
004F8207  C3               RETN
004F8208  9C               PUSHFD
004F8209  60               PUSHAD
004F820A  E8 02000000      CALL X.004F8211
004F820F  33C0             XOR EAX,EAX
004F8211  8BC4             MOV EAX,ESP
```

==============================================

很明显,原始程序入口为

`VA=ImageBase+RVA`

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

下面是使用 TRW2000 手动脱 PECompact 的壳

(标准 TRW2000 V1.22,我想修改版也应该可以吧)

`Load PECompact V1.82`

`<F8>`

看上面的指令,0043E762  68 00E00300      PUSH 3E000

由于 ImageBas=400000,所以 OEP=43E000

所以

`bp if(EIP==43E000)`

`g`

`pedump c:\\xx.exe`

`g`

好了,看看 C:\xx.exe 是不是正常了

================================================================

=

=                inside Pandora's Box - iPB

=                Open Cracking Group  - OCG

=

=

=                                     DiKeN/iPB

================================================================ 4dd466d0e1e0cc123afbe504da2462ca

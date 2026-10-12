
# 带参数的小花

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-634.htm`](https://www.pediy.com/kssd/pediy07/pediy7-634.htm)

`site:www.ptteam.com`

by 来自轻院的狼[`Immlep`]

为了使小花不容易被人清除，所以我在自己写的壳中设计了一些带参数的小花，其实就是使用宏功能。

这样做出来的小花的长度就不一定的，而且中间一些无用的代码也是可变的，下面的是一个简单的例子，小花样本是在 `simonzh2000` 兄的壳中抄来的(想到使用这个方法来写小花的也是 `simonzh2000` 壳中的小花给我的思路，在此谢谢 `simonzh2000` 兄)。

```
;我把这些无用的代码做成列表，我做了很多这样的代码（最主要的是能抗分析，你带入后用 ollydbg 自带的分析功能看看效果）。
PC1_9   equ 050h
PC2_1   equ 0EBh,01h
PC2_2   equ 074h,027h
PC2_3   equ 072h,06h
PC2_4   equ 0EBh,032h
PC2_5   equ 0Fh,080h

jc003 MACRO X,Y,Z,C
;X,Y ，C 为任意长度的 OPcode
;Z 为 X、Y 的长度的和
;
  JMP ＄+(019h+5*C)     ;2   ;L031
  db X
  db Y
;L008:
  JMP ＄+3
  db 075h
  ;POP ECX      ;1
  JMP @F       ;2
  db X
  POP ECX      ;1
  db Y         ;1
;SM_1:
  POP ECX      ;1
  JE ＄-(7+C)    ;2     L008
  JMP ＄+(2+C)     ;2  L021
  db X
  db Y
;L021:
  JNZ ＄-(0BH+2*C)    ;2   L008
  db Y
  db X
;L023:
  POP ECX      ;1
  PUSHFD       ;1
  ADD ECX, -(01AH+3*C) ;6
  JMP ＄+(2+C)     ;2 L028
  db Y
  db X
;L028:
  POPFD        ;1
  JMP  ECX ;2   jmp SM_1
;L031:
  PUSH ECX     ;1
  db 0EBh,01
  db 0EBh
  CALL ＄-(0Eh+C)  ;L023    ;5
  db Y
  db X
  db Z
@@:
endm
```

在 `masm` 中我们这样调用：

`jc003 PC2_2，PC1_9，PC1_9 3`

当然上面的小花只不过是一个例子，做出来的效果还不是很理想，我只不过是提一个思路，其实很多东西还可以变通，

一个好的小花要抗清除，抗分析，还要会动，如果大家有什么心得不妨说来交流交流，谢谢！

`4dd466d0e1e0cc123afbe504da2462ca`

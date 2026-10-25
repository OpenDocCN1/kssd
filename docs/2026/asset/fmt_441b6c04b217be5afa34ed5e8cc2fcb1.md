
### 宏定义

```
#......
#......
#......
macro set\_z\_and\_n(x) {
Z = (x == 0);
N = (x s< 0);
}
macro addflags(op1, op2) {
C = (carry(op1, op2));
V = (scarry(op1, op2));
}
macro addflags2(op1, op2, op3) {
local tmp = (carry(op1,op2));
C = (carry(zext(tmp), op3));
tmp = (scarry(op1,op2));
V = (scarry(zext(tmp), op3));
}
macro subflags(op1, op2) {
C = (op1 < op2);
V = (sborrow(op1, op2));
}
macro subflags2(op1, op2, op3) {
C = (op1 < (op2 + op3));
local tmp = (sborrow(op1, op2));
V = (sborrow(zext(tmp), op3));
}
```

### 算术指令

```
:ADD Rj, Ri
is op1\_12\_4=0xA & op2\_08\_4=0x6 & Rj & Ri {
addflags(Ri, Rj);
Ri = Ri + Rj;
set\_z\_and\_n(Ri);
}
:ADD "#"^u4, Ri
is op1\_12\_4=0xA & op2\_08\_4=0x4 & u4 & Ri {
local tmp1:4 = zext(u4);
addflags(Ri, tmp1);
Ri = Ri + tmp1;
set\_z\_and\_n(Ri);
}
:ADD2 "#"^m4, Ri
is op1\_12\_4=0xA & op2\_08\_4=0x5 & m4 & Ri {
local tmp1:4 = sext(m4);
addflags(Ri, tmp1);
Ri = Ri + tmp1;
set\_z\_and\_n(Ri);
}
:ADDC Rj, Ri
is op1\_12\_4=0xA & op2\_08\_4=0x7 & Rj & Ri {
local tmp1:4 = zext(C);
addflags2(Ri, Rj, tmp1);
Ri = Ri + Rj + tmp1;
set\_z\_and\_n(Ri);
}
:ADDN Rj, Ri
is op1\_12\_4=0xA & op2\_08\_4=0x2 & Rj & Ri {
Ri = Ri + Rj;
}
:ADDN "#"^u4, Ri
is op1\_12\_4=0xA & op2\_08\_4=0x0 & u4 & Ri {
local tmp1:4 = zext(u4);
Ri = Ri + tmp1;
}
:ADDN2 "#"^m4, Ri
is op1\_12\_4=0xA & op2\_08\_4=0x1 & m4 & Ri {
local tmp1:4 = sext(m4);
Ri = Ri + tmp1;
}
:SUB Rj, Ri
is op1\_12\_4=0xA & op2\_08\_4=0xC & Rj & Ri {
subflags(Ri, Rj);
Ri = Ri - Rj;
set\_z\_and\_n(Ri);
}
:SUBC Rj, Ri
is op1\_12\_4=0xA & op2\_08\_4=0xD & Rj & Ri {
local tmp1:4 = zext(C);
subflags2(Ri, Rj, tmp1);
Ri = Ri - Rj - tmp1;
set\_z\_and\_n(Ri);
}
:SUBN Rj, Ri
is op1\_12\_4=0xA & op2\_08\_4=0xE & Rj & Ri {
Ri = Ri - Rj;
}
:CMP Rj, Ri
is op1\_12\_4=0xA & op2\_08\_4=0xA & Rj & Ri {
subflags(Ri, Rj);
local tmp1:4 = Ri - Rj;
set\_z\_and\_n(tmp1);
}
:CMP "#"^u4, Ri
is op1\_12\_4=0xA & op2\_08\_4=0x8 & u4 & Ri {
local tmp1:4 = zext(u4);
subflags(Ri, tmp1);
local tmp2:4 = Ri - tmp1;
set\_z\_and\_n(tmp2);
}
:CMP2 "#"^m4, Ri
is op1\_12\_4=0xA & op2\_08\_4=0x9 & m4 & Ri {
local tmp1:4 = sext(m4);
subflags(Ri, tmp1);
local tmp2:4 = Ri - tmp1;
set\_z\_and\_n(tmp2);
}
```

### 逻辑与位操作指令

```
:AND Rj, Ri
is op1\_12\_4=0x8 & op2\_08\_4=0x2 & Rj & Ri {
Ri = Ri & Rj;
set\_z\_and\_n(Ri);
}
:OR Rj, Ri
is op1\_12\_4=0x9 & op2\_08\_4=0x2 & Rj & Ri {
Ri = Ri | Rj;
set\_z\_and\_n(Ri);
}
:EOR Rj, Ri
is op1\_12\_4=0x9 & op2\_08\_4=0xA & Rj & Ri {
Ri = Ri ^ Rj;
set\_z\_and\_n(Ri);
}
:AND Rj, "@"^Ri
is op1\_12\_4=0x8 & op2\_08\_4=0x4 & Rj & Ri {
local tmp1:4 = \*:4 Ri;
tmp1= tmp1 & Rj;
set\_z\_and\_n(tmp1);
\*:4 Ri = tmp1;
}
:ANDH Rj, "@"^Ri
is op1\_12\_4=0x8 & op2\_08\_4=0x5 & Rj & Ri {
local tmp1:2 = \*:2 Ri;
local tmp2:2 = Rj:2;
tmp2 = ((tmp2 & 0xFF00) & (tmp1 & 0xFF00)) & 0xFF00;
set\_z\_and\_n(tmp2);
\*:2 Ri =((tmp2) | (tmp1 & 0xFF));
}
:ANDB Rj, "@"^Ri
is op1\_12\_4=0x8 & op2\_08\_4=0x6 & Rj & Ri {
local tmp1:1 = \*:1 Ri;
local tmp2:1 = Ri:1;
tmp1 = tmp1 & tmp2;
set\_z\_and\_n(tmp1);
\*:1 Ri = tmp1;
}
:OR Rj, "@"^Ri
is op1\_12\_4=0x9 & op2\_08\_4=0x4 & Rj & Ri {
local tmp1:4 = \*:4 Ri;
tmp1= tmp1 | Rj;
set\_z\_and\_n(tmp1);
\*:4 Ri = tmp1;
}
:ORH Rj, "@"^Ri
is op1\_12\_4=0x9 & op2\_08\_4=0x5 & Rj & Ri {
local tmp1:2 = \*:2 Ri;
local tmp2:2 = Rj:2;
tmp2 = ((tmp2 & 0xFF00) | (tmp1 & 0xFF00)) & 0xFF00;
set\_z\_and\_n(tmp2);
\*:2 Ri =((tmp2) | (tmp1 & 0xFF));
}
:ORB Rj, "@"^Ri
is op1\_12\_4=0x9 & op2\_08\_4=0x6 & Rj & Ri {
local tmp1:1 = \*:1 Ri;
local tmp2:1 = Ri:1;
tmp1 = tmp1 | tmp2;
set\_z\_and\_n(tmp1);
\*:1 Ri = tmp1;
}
:EOR Rj, "@"^Ri
is op1\_12\_4=0x9 & op2\_08\_4=0xC & Rj & Ri {
local tmp1:4 = \*:4 Ri;
tmp1= tmp1 ^ Rj;
set\_z\_and\_n(tmp1);
\*:4 Ri = tmp1;
}
:EORH Rj, "@"^Ri
is op1\_12\_4=0x9 & op2\_08\_4=0xD & Rj & Ri {
local tmp1:2 = \*:2 Ri;
local tmp2:2 = Rj:2;
tmp2 = ((tmp2 & 0xFF00) ^ (tmp1 & 0xFF00)) & 0xFF00;
set\_z\_and\_n(tmp2);
\*:2 Ri =((tmp2) | (tmp1 & 0xFF));
}
:EORB Rj, "@"^Ri
is op1\_12\_4=0x9 & op2\_08\_4=0xE & Rj & Ri {
local tmp1:1 = \*:1 Ri;
local tmp2:1 = Ri:1;
tmp1 = tmp1 ^ tmp2;
set\_z\_and\_n(tmp1);
\*:1 Ri = tmp1;
}
:BANDL "#"^u4, "@"^Ri
is op1\_12\_4=0x8 & op2\_08\_4=0x0 & u4 & Ri {
local tmp1:1 = \*:1 Ri;
local tmp2:1 = (0xF0 | u4);
tmp1= tmp1 & tmp2;
\*:1 Ri = tmp1;
}
:BORL "#"^u4, "@"^Ri
is op1\_12\_4=0x9 & op2\_08\_4=0x0 & u4 & Ri {
local tmp1:1 = \*:1 Ri;
local tmp2:1 = u4;
tmp1= tmp1 | tmp2;
\*:1 Ri = tmp1;
}
:BEORL "#"^u4, "@"^Ri
is op1\_12\_4=0x9 & op2\_08\_4=0x8 & u4 & Ri {
local tmp1:1 = \*:1 Ri;
local tmp2:1 = u4;
tmp1= tmp1 ^ tmp2;
\*:1 Ri = tmp1;
}
:BANDH "#"^u4, "@"^Ri
is op1\_12\_4=0x8 & op2\_08\_4=0x1& u4 & Ri {
local tmp1:1 = \*:1 Ri;
local tmp2:1 = ((u4 << 4) | 0xF);
tmp1= tmp1 & tmp2;
\*:1 Ri = tmp1;
}
:BORH "#"^u4, "@"^Ri
is op1\_12\_4=0x9 & op2\_08\_4=0x1 & u4 & Ri {
local tmp1:1 = \*:1 Ri;
local tmp2:1 = (u4 << 4);
tmp1= tmp1 | tmp2;
\*:1 Ri = tmp1;
}
:BEORH "#"^u4, "@"^Ri
is op1\_12\_4=0x9 & op2\_08\_4=0x9 & u4 & Ri {
local tmp1:1 = \*:1 Ri;
local tmp2:1 = (u4 << 4);
tmp1= tmp1 ^ tmp2;
\*:1 Ri = tmp1;
}
:BTSTL "#^u4, "@"^Ri
is op1\_12\_4=0x8 & op2\_08\_4=0x8 & u4 & Ri {
local tmp1:1 = \*:1 Ri;
local tmp2:1 = u4;
tmp1= tmp1 & tmp2;
Z = (tmp1 == 0);
N = 0;
}
:BTSTH "#"^u4, "@"^Ri
is op1\_12\_4=0x8 & op2\_08\_4=0x9 & u4 & Ri {
local tmp1:1 = \*:1 Ri;
local tmp2:1 = (u4 << 4);
tmp1= tmp1 & tmp2;
Z = (tmp1 == 0);
N = (tmp1 s< 0);
}
#......
#......
#......
```

## ISA Conditional Branches Instruction Definitions(CGEN)

条件跳转指令定义CGEN实现

```
;......
;......
;......
(define-pmacro (cond-branch cc condition)
(begin
(dni (.sym b cc d)
(.str (.sym b cc :d) " label9")
(NOT-IN-DELAY-SLOT)
(.str (.sym b cc :d) " $label9")
(+ OP1\_F (.sym CC\_ cc) label9)
(delay (const 1)
(if condition (set pc label9)))
((fr30-1 (unit u-cti)))
)
(dni (.sym b cc)
(.str (.sym b cc) " label9")
(NOT-IN-DELAY-SLOT)
(.str (.sym b cc) " $label9")
(+ OP1\_E (.sym CC\_ cc) label9)
(if condition (set pc label9))
((fr30-1 (unit u-cti)))
)
)
)
(cond-branch ra (const BI 1))
(cond-branch no (const BI 0))
(cond-branch eq zbit)
(cond-branch ne (not zbit))
(cond-branch c cbit)
(cond-branch nc (not cbit))
(cond-branch n nbit)
(cond-branch p (not nbit))
(cond-branch v vbit)
(cond-branch nv (not vbit))
(cond-branch lt (xor vbit nbit))
(cond-branch ge (not (xor vbit nbit)))
(cond-branch le (or (xor vbit nbit) zbit))
(cond-branch gt (not (or (xor vbit nbit) zbit)))
(cond-branch ls (or cbit zbit))
(cond-branch hi (not (or cbit zbit)))
;......
;......
;......
```

## ISA Conditional Branches Instruction Definitions(SLEIGH)

到底是用逻辑运算指令还是位运算指令实现条件跳转指令时标志位的计算？Ghidra官方类似的一些处理器实现是用逻辑运算指令，感觉要用位运算指令，好像有问题。

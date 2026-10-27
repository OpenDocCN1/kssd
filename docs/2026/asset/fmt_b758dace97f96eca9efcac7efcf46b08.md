
### 使用 Graphviz 和 nvdisasm 生成基本块控制流图

以下是使用Graphviz和nvdisasm生成上述cubin（`a.cubin`）的基本块控制流的PNG图像（`bbcfg.png`）的方法：

```
nvdisasm -bbcfg a.cubin | dot -obbcfg.png -Tpng
```

Here’s the generated graph:

![Untitled](cuda%205a7d349bcfa14922980c95f106be23d6/Untitled%2032.png)

### 寄存器活动范围信息

`nvdisasm`能够显示寄存器（general and predicate）活动范围信息。

对于CUDA汇编的每一行，能够显示给定的设备寄存器是否assigned, accessed, live or re-assigned，它还显示使用的寄存器总数，如果用户对任何特定寄存器的寿命范围或一般的寄存器使用感兴趣，这将很有用。

Here’s a sample output（为简洁起见，对输出进行了删减）：

```
//
//+-----------------+------+
//|      GPR        | PRED |
//|                 |      |
//|                 |      |
//| 000000000011    |      |
//|# 012345678901 | # 01 |
//+-----------------+------+
.global acos //|                 |      |
.type   acos,@function //|                 |      |
.size   acos,(.L_21-acos) //|                 |      |
.other  acos,@"STO_CUDA_ENTRY STV_DEFAULT"        //|                 |      |
acos: //|                 |      |
.text.acos: //|                 |      |
MOV R1, c[0x0][0x28] ; //|1  ^           |      |
NOP; //|1  ^           |      |
S2R R0, SR_CTAID.X ; //|2 ^:           |      |
S2R R3, SR_TID.X ; //|3 :: ^         |      |
IMAD R0, R0, c[0x0][0x0], R3 ; //|3 x: v         |      |
ISETP.GE.AND P0, PT, R0, c[0x0][0x170], PT ; //|2 v:           |1 ^  |
@P0 EXIT ; //|2 ::           |1 v  |
.L_1: //|2 ::           |      |
MOV R11, 0x4 ; //|3 ::         ^ |      |
IMAD.WIDE R2, R0, R11, c[0x0][0x160] ; //|5 v:^^       v |      |
LDG.E.SYS R2, [R2] ; //|4 ::^        : |      |
MOV R7, 0x3d53f941 ; //|5 :::    ^   : |      |
FADD.FTZ R4, |R2|.reuse, -RZ ; //|6 ::v ^  :   : |      |
FSETP.GT.FTZ.AND P0, PT, |R2|.reuse, 0.5699, PT; //|6 ::v :  :   : |1 ^  |
FSETP.GEU.FTZ.AND P1, PT, R2, RZ, PT ; //|6 ::v :  :   : |2 :^ |
FADD.FTZ R5, -R4, 1 ; //|6 ::  v^ :   : |2 :: |
IMAD.WIDE R2, R0, R11, c[0x0][0x168] ; //|8 v:^^:: :   v |2 :: |
FMUL.FTZ R5, R5, 0.5 ; //|5 ::  :x :     |2 :: |
@P0 MUFU.SQRT R4, R5 ; //|5 ::  ^v :     |2 v: |
MOV R5, c[0x0][0x0] ; //|5 ::  :^ :     |2 :: |
IMAD R0, R5, c[0x0][0xc], R0 ; //|5 x:  :v :     |2 :: |
FMUL.FTZ R6, R4, R4 ; //|5 ::  v ^:     |2 :: |
FFMA.FTZ R7, R6, R7, 0.018166976049542427063 ; //|5 ::  : vx     |2 :: |
FFMA.FTZ R7, R6, R7, 0.046756859868764877319 ; //|5 ::  : vx     |2 :: |
FFMA.FTZ R7, R6, R7, 0.074846573173999786377 ; //|5 ::  : vx     |2 :: |
FFMA.FTZ R7, R6, R7, 0.16667014360427856445 ; //|5 ::  : vx     |2 :: |
FMUL.FTZ R7, R6, R7 ; //|5 ::  : vx     |2 :: |
FFMA.FTZ R7, R4, R7, R4 ; //|4 ::  v  x     |2 :: |
FADD.FTZ R9, R7, R7 ; //|4 ::     v ^   |2 :: |
@!P0 FADD.FTZ R9, -R7, 1.5707963705062866211 ; //|4 ::     v ^   |2 v: |
ISETP.GE.AND P0, PT, R0, c[0x0][0x170], PT ; //|3 v:       :   |2 ^: |
@!P1 FADD.FTZ R9, -R9, 3.1415927410125732422 ; //|3 ::       x   |2 :v |
STG.E.SYS [R2], R9 ; //|3 ::       v   |1 :  |
@!P0 BRA (.L_1) ; //|2 ::           |1 v  |
EXIT ; //|1  :           |      |
.L_2: //+.................+......+
BRA (.L_2); //|                 |      |
.L_21: //+-----------------+------+
//
//Legend:
//   ^       : Register assignment
//   v       : Register usage
//   x       : Register usage and reassignment
//   :       : Register in use
//   <space> : Register not in use
//   #       : Number of occupied registers
```

### CUDA 源文件行号信息

`nvdisasm`能够显示CUDA源文件的行号信息（这对调试很有用）：`nvdisasm -g <input cubin file>`

以下是使用`nvdisasm -g`命令的输出示例：

```
//------------------------------------ .text._Z6kernali ------------------------------------
.section        .text._Z6kernali,"ax",@progbits
.sectioninfo    @"SHI_REGISTERS=24"
.align 128
.global         _Z6kernali
.type           _Z6kernali,@function
.size           _Z6kernali,(.L_4-_Z6kernali)
.other          _Z6kernali,@"STO_CUDA_ENTRY STV_DEFAULT"
_Z6kernali:
.text._Z6kernali:
/*0000*/                   MOV R1, c[0x0][0x28] ;
/*0010*/                   NOP;
//## File "/home/user/cuda/sample/sample.cu", line 25
/*0020*/                   MOV R0, 0x160 ;
/*0030*/                   LDC R0, c[0x0][R0] ;
/*0040*/                   MOV R0, R0 ;
/*0050*/                   MOV R2, R0 ;
//## File "/home/user/cuda/sample/sample.cu", line 26
/*0060*/                   MOV R4, R2 ;
/*0070*/                   MOV R20, 32@lo((_Z6kernali+.L_1@srel)) ;
/*0080*/                   MOV R21, 32@hi((_Z6kernali+.L_1@srel)) ;
/*0090*/                   CALL.ABS.NOINC (_Z3fooi) ;
.L_1:
/*00a0*/                   MOV R0, R4 ;
/*00b0*/                   MOV R4, R2 ;
/*00c0*/                   MOV R2, R0 ;
/*00d0*/                   MOV R20, 32@lo((_Z6kernali+.L_2@srel)) ;
/*00e0*/                   MOV R21, 32@hi((_Z6kernali+.L_2@srel)) ;
/*00f0*/                   CALL.ABS.NOINC (_Z3bari) ;
.L_2:
/*0100*/                   MOV R4, R4 ;
/*0110*/                   IADD3 R4, R2, R4, RZ ;
/*0120*/                   MOV R2, 32@lo(arr) ;
/*0130*/                   MOV R3, 32@hi(arr) ;
/*0140*/                   MOV R2, R2 ;
/*0150*/                   MOV R3, R3 ;
/*0160*/                   ST.E.SYS [R2], R4 ;
//## File "/home/user/cuda/sample/sample.cu", line 27
/*0170*/                   ERRBAR ;
/*0180*/                   EXIT ;
.L_3:
/*0190*/                   BRA (.L_3);
.L_4:
```

### 带函数内联信息的行号信息

`nvdisasm`能够显示CUDA源文件的带有附加函数内联信息（如果有）的行号信息。在没有任何函数内联的情况下，输出与使用`nvdisasm -g`命令的输出相同：`nvdisasm -gi`

这是使用`nvdisasm -gi`命令的kernel输出示例：

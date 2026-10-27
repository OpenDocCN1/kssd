
| `--gpu-architecture <gpu architecture name>` | `-arch` | Specify GPU Architecture for which information should be dumped. Allowed values for this option: `sm_35`, `sm_37`, `sm_50`, `sm_52`, `sm_53`, `sm_60`, `sm_61`, `sm_62`, `sm_70`, `sm_72`, `sm_75`, `sm_80`. |

| `--help` | `-h` | Print this help information on this tool. |

| `--list-elf` | `-lelf` | List all the ELF files available in the fatbin. Works with host executable/object/library and external fatbin. All other options are ignored with this flag. This can be used to select particular ELF with `-xelf` option later. |

| `--list-ptx` | `-lptx` | List all the PTX files available in the fatbin. Works with host executable/object/library and external fatbin. All other options are ignored with this flag. This can be used to select particular PTX with `-xptx` option later. |

| `--options-file <file>,...` | `-optf` | Include command line options from specified file. |

| `--sort-functions` | `-sort` | Sort functions when dumping sass. |

| `--version` | `-V` | Print version information on this tool. |

### nvdisasm

`nvdisasm`从独立的 cubin 文件中提取信息并以人类可读的格式呈现它们。

`nvdisasm`的输出包括CUDA assembly code for each kernel, listing of ELF data sections and other CUDA specific sections，输出样式可以通过`nvdisasm`的command-line options控制。

`nvdisasm`也能做control flow analysis注释jump/branch targets并使输出更易于阅读。(`nvdisasm`需要完整的重定位信息来做控制流分析。如果 CUDA 二进制文件中缺少此信息，请使用`nvdisasm`选项`-ndf`关闭控制流分析，或使用`ptxas`和`nvlink`选项`-preserve-relocs`重新生成 cubin 文件)。

- 用法：`nvdisasm [options] <input cubin file>`

- Here’s a sample output of `nvdisasm`

```c
.headerflags @"EF_CUDA_TEXMODE_UNIFIED EF_CUDA_64BIT_ADDRESS EF_CUDA_SM70
EF_CUDA_VIRTUAL_SM(EF_CUDA_SM70)"
.elftype @"ET_EXEC"
//--------------------- .nv.info --------------------------
.section .nv.info,"",@"SHT_CUDA_INFO"
.align 4
......
//--------------------- .text._Z9acos_main10acosParams --------------------------
.section .text._Z9acos_main10acosParams,"ax",@progbits
.sectioninfo @"SHI_REGISTERS=14"
.align 128
.global _Z9acos_main10acosParams
.type _Z9acos_main10acosParams,@function
.size _Z9acos_main10acosParams,(.L_21 - _Z9acos_main10acosParams)
.other _Z9acos_main10acosParams,@"STO_CUDA_ENTRY STV_DEFAULT"
_Z9acos_main10acosParams:
.text._Z9acos_main10acosParams:
/*0000*/ MOV R1, c[0x0][0x28] ;
/*0010*/ NOP;
/*0020*/ S2R R0, SR_CTAID.X ;
/*0030*/ S2R R3, SR_TID.X ;
/*0040*/ IMAD R0, R0, c[0x0][0x0], R3 ;
/*0050*/ ISETP.GE.AND P0, PT, R0, c[0x0][0x170], PT ;
/*0060*/ @P0 EXIT ;
.L_1:
/*0070*/ MOV R11, 0x4 ;
/*0080*/ IMAD.WIDE R2, R0, R11, c[0x0][0x160] ;
/*0090*/ LDG.E.SYS R2, [R2] ;
/*00a0*/ MOV R7, 0x3d53f941 ;
/*00b0*/ FADD.FTZ R4, |R2|.reuse, -RZ ;
/*00c0*/ FSETP.GT.FTZ.AND P0, PT, |R2|.reuse, 0.5699, PT ;
/*00d0*/ FSETP.GEU.FTZ.AND P1, PT, R2, RZ, PT ;
/*00e0*/ FADD.FTZ R5, -R4, 1 ;
/*00f0*/ IMAD.WIDE R2, R0, R11, c[0x0][0x168] ;
/*0100*/ FMUL.FTZ R5, R5, 0.5 ;
/*0110*/ @P0 MUFU.SQRT R4, R5 ;
/*0120*/ MOV R5, c[0x0][0x0] ;
/*0130*/ IMAD R0, R5, c[0x0][0xc], R0 ;
/*0140*/ FMUL.FTZ R6, R4, R4 ;
/*0150*/ FFMA.FTZ R7, R6, R7, 0.018166976049542427063 ;
/*0160*/ FFMA.FTZ R7, R6, R7, 0.046756859868764877319 ;
/*0170*/ FFMA.FTZ R7, R6, R7, 0.074846573173999786377 ;
/*0180*/ FFMA.FTZ R7, R6, R7, 0.16667014360427856445 ;
/*0190*/ FMUL.FTZ R7, R6, R7 ;
/*01a0*/ FFMA.FTZ R7, R4, R7, R4 ;
/*01b0*/ FADD.FTZ R9, R7, R7 ;
/*01c0*/ @!P0 FADD.FTZ R9, -R7, 1.5707963705062866211 ;
/*01d0*/ ISETP.GE.AND P0, PT, R0, c[0x0][0x170], PT ;
/*01e0*/ @!P1 FADD.FTZ R9, -R9, 3.1415927410125732422 ;
/*01f0*/ STG.E.SYS [R2], R9 ;
/*0200*/ @!P0 BRA `(.L_1) ;
/*0210*/ EXIT ;
.L_2:
/*0220*/ BRA `(.L_2);
.L_21:
```

- To get the control flow graph of a kernel, use the following:`nvdisasm -cfg <input cubin file>`

`nvdisasm`能够生成DOT图形描述语言格式的CUDA程序集控制流。

`nvdisasm`的控制流输出可以直接导入到DOT图形可视化工具，例如<a href="elink@039K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3N6J5j5i4m8Z5N6X3W2*7i4K6u0W2L8%4u0Y4i4K6u0r3">Graphviz</a>。

以下是使用`Graphviz`和`nvdisasm`生成上述cubin (`a.cubin`)控制流的PNG图像 (`cfg.png`) 的方法：

```c
nvdisasm -cfg a.cubin | dot -ocfg.png -Tpng
```

- Here’s the generated graph:

![Untitled](cuda%205a7d349bcfa14922980c95f106be23d6/Untitled%2031.png)

以下是使用`Graphviz`和`nvdisasm`生成上述cubin (`a.cubin`) 的基本块控制流的PNG图像 (`bbcfg.png`) 的方法：

```c
nvdisasm -bbcfg a.cubin | dot -obbcfg.png -Tpng
```

- Here’s the generated graph:

![Untitled](cuda%205a7d349bcfa14922980c95f106be23d6/Untitled%2032.png)

- `nvdisasm`能够显示寄存器 (general and predicate) 活动范围信息

对于CUDA汇编的每一行，能够显示给定的设备寄存器是否assigned, accessed, live or re-assigned，它还显示使用的寄存器总数，如果用户对任何特定寄存器的寿命范围或一般的寄存器使用感兴趣，这将很有用。

Here’s a sample output (为简洁起见，对输出进行了删减)：

```cpp
// +-----------------+------+
// | GPR | PRED |
// | | |
// | | |
// | 000000000011 | |
// | # 012345678901 | # 01 |
// +-----------------+------+
.global acos // | | |
.type acos,@function // | | |
.size acos,(.L_21 - acos) // | | |
.other acos,@"STO_CUDA_ENTRY STV_DEFAULT" // | | |
acos: // | | |
.text.acos: // | | |
MOV R1, c[0x0][0x28] ; // | 1 ^ | |
NOP; // | 1 ^ | |
S2R R0, SR_CTAID.X ; // | 2 ^: | |
S2R R3, SR_TID.X ; // | 3 :: ^ | |
IMAD R0, R0, c[0x0][0x0], R3 ; // | 3 x: v | |
ISETP.GE.AND P0, PT, R0, c[0x0][0x170], PT ; // | 2 v: | 1 ^ |
@P0 EXIT ; // | 2 :: | 1 v |
.L_1: // | 2 :: | |
MOV R11, 0x4 ; // | 3 :: ^ | |
IMAD.WIDE R2, R0, R11, c[0x0][0x160] ; // | 5 v:^^ v | |
LDG.E.SYS R2, [R2] ; // | 4 ::^ : | |
MOV R7, 0x3d53f941 ; // | 5 ::: ^ : | |
FADD.FTZ R4, |R2|.reuse, -RZ ; // | 6 ::v ^ : : | |
FSETP.GT.FTZ.AND P0, PT, |R2|.reuse, 0.5699, PT; // | 6 ::v : : : | 1 ^ |
FSETP.GEU.FTZ.AND P1, PT, R2, RZ, PT ; // | 6 ::v : : : | 2 :^ |
FADD.FTZ R5, -R4, 1 ; // | 6 :: v^ : : | 2 :: |
IMAD.WIDE R2, R0, R11, c[0x0][0x168] ; // | 8 v:^^:: : v | 2 :: |
FMUL.FTZ R5, R5, 0.5 ; // | 5 :: :x : | 2 :: |
@P0 MUFU.SQRT R4, R5 ; // | 5 :: ^v : | 2 v: |
MOV R5, c[0x0][0x0] ; // | 5 :: :^ : | 2 :: |
IMAD R0, R5, c[0x0][0xc], R0 ; // | 5 x: :v : | 2 :: |
FMUL.FTZ R6, R4, R4 ; // | 5 :: v ^: | 2 :: |
FFMA.FTZ R7, R6, R7, 0.018166976049542427063 ; // | 5 :: : vx | 2 :: |
FFMA.FTZ R7, R6, R7, 0.046756859868764877319 ; // | 5 :: : vx | 2 :: |
FFMA.FTZ R7, R6, R7, 0.074846573173999786377 ; // | 5 :: : vx | 2 :: |
FFMA.FTZ R7, R6, R7, 0.16667014360427856445 ; // | 5 :: : vx | 2 :: |
FMUL.FTZ R7, R6, R7 ; // | 5 :: : vx | 2 :: |
FFMA.FTZ R7, R4, R7, R4 ; // | 4 :: v x | 2 :: |
FADD.FTZ R9, R7, R7 ; // | 4 :: v ^ | 2 :: |
@!P0 FADD.FTZ R9, -R7, 1.5707963705062866211 ; // | 4 :: v ^ | 2 v: |
ISETP.GE.AND P0, PT, R0, c[0x0][0x170], PT ; // | 3 v: : | 2 ^: |
@!P1 FADD.FTZ R9, -R9, 3.1415927410125732422 ; // | 3 :: x | 2 :v |
STG.E.SYS [R2], R9 ; // | 3 :: v | 1 : |
@!P0 BRA `(.L_1) ; // | 2 :: | 1 v |
EXIT ; // | 1 : | |
.L_2: // +.................+......+
BRA `(.L_2); // | | |
.L_21: // +-----------------+------+
// Legend:
// ^ : Register assignment
// v : Register usage
// x : Register usage and reassignment
// : : Register in use
// <space> : Register not in use
// # : Number of occupied registers
```

- `nvdisasm`能够显示CUDA源文件的行号信息(这对调试很有用)：`nvdisasm -g <input cubin file>`

以下是使用`nvdisasm -g` 命令的输出示例：

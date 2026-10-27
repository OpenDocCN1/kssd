
| `--list-ptx` | `-lptx` | List all the PTX files available in the fatbin. Works with host executable/object/library and external fatbin. All other options are ignored with this flag. This can be used to select particular PTX with `-xptx` option later. |
| `--options-file <file>,...` | `-optf` | Include command line options from specified file. |
| `--sort-functions` | `-sort` | Sort functions when dumping sass. |
| `--version` | `-V` | Print version information on this tool. |

### nvdisasm

`nvdisasm`从独立的 cubin 文件中提取信息并以人类可读的格式呈现它们。

`nvdisasm`的输出包括CUDA assembly code for each kernel, listing of ELF data sections and other CUDA specific sections，输出样式可以通过`nvdisasm`的command-line options控制。

`nvdisasm`也能做control flow analysis注释jump/branch targets并使输出更易于阅读。(`nvdisasm`需要完整的重定位信息来做控制流分析。如果 CUDA 二进制文件中缺少此信息，请使用`nvdisasm`选项`-ndf`关闭控制流分析，或使用`ptxas`和`nvlink`选项`-preserve-relocs`重新生成 cubin 文件)。

- 用法：`nvdisasm [options] <input cubin file>`
- Here’s a sample output of nvdisasm

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 | `.headerflags @"EF_CUDA_TEXMODE_UNIFIED EF_CUDA_64BIT_ADDRESS EF_CUDA_VIRTUAL_SM(EF_CUDA_SM70)" .elftype @"ET_EXEC" //-------------------- .nv.info ------------------ .section .nv.info,"@","SHT_CUDA_INFO" .align 4 ...... //-------------------- .text._Z9acos_main10acosParams ------------------ .section .text._Z9acos_main10acosParams,"ax",@progbits .sectioninfo @"SHI_REGISTERS=14" .align 128 .global _Z9acos_main10acosParams .type _Z9acos_main10acosParams,@function .size _Z9acos_main10acosParams,(.L_21 - _Z9acos_main10acosParams) .other _Z9acos_main10acosParams,@"STO_CUDA_ENTRY STV_DEFAULT" _Z9acos_main10acosParams: .text._Z9acos_main10acosParams: /*0000*/ MOV R1, c[0x0][0x28] ; /*0010*/ NOP; /*0020*/ S2R R0, SR_CTAID.X ; /*0030*/ S2R R3, SR_TID.X ; /*0040*/ IMAD R0, R0, c[0x0][0x0], R3 ; /*0050*/ ISETP.GE.AND P0, PT, R0, c[0x0][0x170], PT ; /*0060*/ @P0 EXIT ; .L_1: /*0070*/ MOV R11, 0x4 ; /*0080*/ IMAD.WIDE R2, R0, R11, c[0x0][0x160] ; /*0090*/ LDG.E.SYS R2, [R2] ; /*00a0*/ MOV R7, 0x3d53f941 ; /*00b0*/ FADD.FTZ R4, |R2|.reuse, -RZ ; /*00c0*/ FSETP.GT.FTZ.AND P0, PT, |R2|.reuse, 0.5699, PT ; /*00d0*/ FSETP.GEU.FTZ.AND P1, PT, R2, RZ, PT ; /*00e0*/ FADD.FTZ R5, -R4, 1 ; /*00f0*/ IMAD.WIDE R2, R0, R11, c[0x0][0x168] ; /*0100*/ FMUL.FTZ R5, R5, 0.5 ; /*0110*/ @P0 MUFU.SQRT R4, R5 ; /*0120*/ MOV R5, c[0x0][0x0] ; /*0130*/ IMAD R0, R5, c[0x0][0xc], R0 ; /*0140*/ FMUL.FTZ R6, R4, R4 ; /*0150*/ FFMA.FTZ R7, R6, R7, 0.018166976049542427063 ; /*0160*/ FFMA.FTZ R7, R6, R7, 0.046756859868764877319 ; /*0170*/ FFMA.FTZ R7, R6, R7, 0.074846573173999786377 ; /*0180*/ FFMA.FTZ R7, R6, R7, 0.16667014360427856445 ; /*0190*/ FMUL.FTZ R7, R6, R7 ; /*01a0*/ FFMA.FTZ R7, R4, R7, R4 ; /*01b0*/ FADD.FTZ R9, R7, R7 ; /*01c0*/ @!P0 FADD.FTZ R9, -R7, 1.5707963705062866211 ; /*01d0*/ ISETP.GE.AND P0, PT, R0, c[0x0][0x170], PT ; /*01e0*/ @!P1 FADD.FTZ R9, -R9, 3.1415927410125732422 ; /*01f0*/ STG.E.SYS [R2], R9 ; /*0200*/ @!P0 BRA (.L_1) ; /*0210*/ EXIT ; .L_2: /*0220*/ BRA (.L_2); .L_21: |

- To get the control flow graph of a kernel, use the following:`nvdisasm -cfg <input cubin file>`

`nvdisasm`能够生成DOT图形描述语言格式的CUDA程序集控制流。

`nvdisasm`的控制流输出可以直接导入到DOT图形可视化工具，例如[Graphviz](https://bbs.kanxue.com/elink@bceK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3N6J5j5i4m8Z5N6X3W2*7i4K6u0W2L8%4u0Y4i4K6u0r3)。

以下是使用Graphviz和`nvdisasm`生成上述cubin (`a.cubin`)控制流的PNG图像 (`cfg.png`) 的方法：

|  |  |
| --- | --- |
| 1 | `nvdisasm -cfg a.cubin | dot -ocfg.png -Tpng` |

- Here’s the generated graph:

![Untitled](https://bbs.kanxue.com/cuda%205a7d349bcfa14922980c95f106be23d6/Untitled%2031.png)

|  |  |
| --- | --- |

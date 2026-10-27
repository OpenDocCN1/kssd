
如输出所示，`a.out` 主机二进制文件包含 `sm_70` 的 `cubin` 和 `ptx` 代码。

```
$ cuobjdump a.out -sass -ptx
Fatbin elf code:
================
arch = sm_70
code version = [1,7]
producer = cuda
host = linux
compile_size = 64bit
identifier = add.cu
code for sm_70
Function : _Z3addPiS_S_
.headerflags    @"EF_CUDA_SM70 EF_CUDA_PTX_SM(EF_CUDA_SM70)"
/*0000*/      IMAD.MOV.U32 R1, RZ, RZ, c[0x0][0x28] ;
/* 0x00000a00ff017624 */
/* 0x000fd000078e00ff */
/*0010*/      @!PT SHFL.IDX PT, RZ, RZ, RZ, RZ ;
/* 0x000000fffffff389 */
/* 0x000fe200000e00ff */
/*0020*/      IMAD.MOV.U32 R2, RZ, RZ, c[0x0][0x160] ;
/* 0x00005800ff027624 */
/* 0x000fe200078e00ff */
/*0030*/      MOV R3, c[0x0][0x164] ;
/* 0x0000590000037a02 */
/* 0x000fe20000000f00 */
/*0040*/      IMAD.MOV.U32 R4, RZ, RZ, c[0x0][0x168] ;
/* 0x00005a00ff047624 */
/* 0x000fe200078e00ff */
/*0050*/      MOV R5, c[0x0][0x16c] ;
/* 0x00005b0000057a02 */
/* 0x000fcc0000000f00 */
/*0060*/      LDG.E.SYS R2, [R2] ;
/* 0x0000000002027381 */
/* 0x000ea800001ee900 */
/*0070*/      LDG.E.SYS R5, [R4] ;
/* 0x0000000004057381 */
/* 0x000ea200001ee900 */
/*0080*/      IMAD.MOV.U32 R6, RZ, RZ, c[0x0][0x170] ;
/* 0x00005c00ff067624 */
/* 0x000fe200078e00ff */
/*0090*/      MOV R7, c[0x0][0x174] ;
/* 0x00005d0000077a02 */
/* 0x000fe40000000f00 */
/*00a0*/      IADD3 R9, R2, R5, RZ ;
/* 0x0000000502097210 */
/* 0x004fd00007ffe0ff */
/*00b0*/      STG.E.SYS [R6], R9 ;
/* 0x0000000906007386 */
/* 0x000fe2000010e900 */
/*00c0*/      EXIT ;
/* 0x000000000000794d */
/* 0x000fea0003800000 */
/*00d0*/      BRA 0xd0 ;
/* 0xfffffff000007947 */
/* 0x000fc0000383ffff */
/*00e0*/      NOP ;
/* 0x0000000000007918 */
/* 0x000fc00000000000 */
/*00f0*/      NOP ;
/* 0x0000000000007918 */
/* 0x000fc00000000000 */
.......................
Fatbin ptx code:
================
arch = sm_70
code version = [7,0]
producer = cuda
host = linux
compile_size = 64bit
compressed
identifier = add.cu
.version 7.0
.target sm_70
.address_size 64
.visible .entry _Z3addPiS_S_(
.param .u64 _Z3addPiS_S__param_0,
.param .u64 _Z3addPiS_S__param_1,
.param .u64 _Z3addPiS_S__param_2
)
{
.reg .s32 %r<4>;
.reg .s64 %rd<7>;
ld.param.u64 %rd1, [_Z3addPiS_S__param_0];
ld.param.u64 %rd2, [_Z3addPiS_S__param_1];
ld.param.u64 %rd3, [_Z3addPiS_S__param_2];
cvta.to.global.u64 %rd4, %rd3;
cvta.to.global.u64 %rd5, %rd2;
cvta.to.global.u64 %rd6, %rd1;
ld.global.u32 %r1, [%rd6];
ld.global.u32 %r2, [%rd5];
add.s32 %r3, %r2, %r1;
st.global.u32 [%rd4], %r3;
ret;
}
```

### 使用 `-lelf` 选项列出 host binary 中的 cubin files

```
$ cuobjdump a.out -lelf
ELF file
1: add_new.sm_70.cubin
ELF file
2: add_new.sm_75.cubin
ELF file
3: add_old.sm_70.cubin
ELF file
4: add_old.sm_75.cubin
```

### 使用 `-xelf all` 选项从 host binary 中提取出所有的 cubins

```
$ cuobjdump a.out -xelf all
Extracting ELF file
1: add_new.sm_70.cubin
Extracting ELF file
2: add_new.sm_75.cubin
Extracting ELF file
3: add_old.sm_70.cubin
Extracting ELF file
4: add_old.sm_75.cubin
```

### 从 host binary 中提取名为 `add_new.sm_70.cubin` 的 cubin

```
$ cuobjdump a.out -xelf add_new.sm_70.cubin
Extracting ELF file
1: add_new.sm_70.cubin
```

### 从 host binary 中提取名称中包含 `_old` 的 cubins

```
$ cuobjdump a.out -xelf _old
Extracting ELF file
1: add_old.sm_70.cubin
Extracting ELF file
2: add_old.sm_75.cubin
```

也就是说可以将任何子字符串传递给 `-xelf` 和 `-xptx` 选项。只有名称中包含子字符串的文件才会从输入二进制文件中提取。

### To dump 公共资源和每个函数的资源使用信息

```
$ cuobjdump test.cubin -res-usage
Resource usage:
Common:
GLOBAL:56 CONSTANT[3]:28
Function calculate:
REG:24 STACK:8 SHARED:0 LOCAL:0 CONSTANT[0]:472 CONSTANT[2]:24 TEXTURE:0 SURFACE:0 SAMPLER:0
Function mysurf_func:
REG:38 STACK:8 SHARED:4 LOCAL:0 CONSTANT[0]:532 TEXTURE:8 SURFACE:7 SAMPLER:0
Function mytexsampler_func:
REG:42 STACK:0 SHARED:0 LOCAL:0 CONSTANT[0]:472 TEXTURE:4 SURFACE:0 SAMPLER:1
```

请注意，`REG`、`TEXTURE`、`SURFACE` 和 `SAMPLER` 的值表示计数，而对于其他资源，它表示使用的字节数。

## Command-line Options

| Option (long) | Option (short) | Description |
| --- | --- | --- |
| --all-fatbin | -all | Dump all fatbin sections. By default will only dump contents of executable fatbin (if exists), else relocatable fatbin if no executable fatbin. |
| --dump-elf | -elf | Dump ELF Object sections. |
| --dump-elf-symbols | -symbols | Dump ELF symbol names. |
| --dump-ptx | -ptx | Dump PTX for all listed device functions. |
| --dump-sass | -sass | Dump CUDA assembly for a single cubin file or all cubin files embedded in the binary. |
| --dump-resource-usage | -res-usage | Dump resource usage for each ELF. Useful in getting all the resource usage information at one place. |
| --extract-elf <partial file name>,... | -xelf | Extract ELF file(s) name containing <partial file name> and save as file(s). Use all to extract all files. To get the list of ELF files use -lelf option. Works with host executable/object/library and external fatbin. All dump and list options are ignored with this option. |
| --extract-ptx <partial file name>,... | -xptx | Extract PTX file(s) name containing <partial file name> and save as file(s). Use all to extract all files. To get the list of PTX files use -lptx option. Works with host executable/object/library and external fatbin. All dump and list options are ignored with this option. |
| --function <function name>,... | -fun | Specify names of device functions whose fat binary structures must be dumped. |
| --function-index <function index>,... | -findex | Specify symbol table index of the function whose fat binary structures must be dumped. |
| --gpu-architecture <gpu architecture name> | -arch | Specify GPU Architecture for which information should be dumped. Allowed values for this option: sm_35, sm_37, sm_50, sm_52, sm_53, sm_60, sm_61, sm_62, sm_70, sm_72, sm_75, sm_80. |
| --help | -h | Print this help information on this tool. |
| --list-elf | -lelf | List all the ELF files available in the fatbin. Works with host executable/object/library and external fatbin. All other options are ignored with this flag. This can be used to select particular ELF with -xelf option later. |

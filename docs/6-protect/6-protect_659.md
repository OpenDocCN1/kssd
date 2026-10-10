

# Asprotect 中的 X86 虚拟机代码分析【原创】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-644.htm`](https://www.pediy.com/kssd/pediy08/pediy8-644.htm)

Asprotect 中的 X86 虚拟机代码分析  
  
作    者: blackeyes  
  
1.  起因:  
  
    最近跟踪一 Asprotect 保护的程序, 发现 stolen code 都是在 Asprotect 自己的虚拟机中执行,   
  
非常不利于跟踪与分析, 于是把 Asprotect 的虚拟机代码进行了分析.   
  
  
2.  代码处理概述  
  
还是用例子来说明吧, 原始的一段 CODE 如下:  
  
00D6FC1C    55              PUSH EBP  
00D6FC1D    8BEC            MOV EBP,ESP  
00D6FC1F    83C4 E0         ADD ESP,-20  
...  
00D6FD48    8BE5            MOV ESP,EBP  
00D6FD4A    5D              POP EBP  
00D6FD4B    C2 0C00         RETN 0C  
  
Asprotect 将上面的每一行机器代码分析处理, 然后每一行保存到一个固定大小的结构中,  
  
运行的时候这段代码就只需要下面四行:  
  
00D6FC1C    68 00000000     PUSH 0  
00D6FC21    68 1CFCD600     PUSH 0D6FC1C  
00D6FC26    68 B432E600     PUSH 0E632B4  
00D6FC2B    E8 18960000     CALL 00D79248  
  
其中:  
 00D6FC1C ----- 代码起始地址  
 00E632B4 ----- 一结构起始地址, 包含处理后的代码信息  
 00D79248 ----- X86 虚拟机 Function 地址  
  
  
3.  机器代码分析  
  
每一行机器代码被分析处理后, 会分解成 10 项 保存到结构中, 如下:  
  
   1   -   第 1 个 机器码的内存起始地址;  
  
   2   -   机器码的第 1 个 BYTE, 如果不是前缀机器码, 就是真正的机器码的第 1 个 BYTE;  
  
   3   -   机器码的第 2 个 BYTE, 并且前面是前缀机器码, 它是真正的机器码的第 1 个 BYTE;  
  
   4   -   机器码中的立即数是否要调整, 相当于重定位, 例如;  
  
               00400000  68 34124000   PUSH 00401234  
       
     如果希望这行代码在 00500000 是这样工作的:  
  
               00500000  68 34125000   PUSH 00501234  
  
           即表示机器码中的立即数是随段起始地址而调整的.  
  
   5   -   机器码中的 第 1 个 立即数;  
  
   6   -   机器码中的 第 2 个 立即数;  
  
   7   -   机器码中的算术/逻辑操作;  
  
    0:ADD, 1:OR, 2:ADC, 3:SBB, 4:TEST, 5:SUB, 6:XOR, 7:CMP  
  
   8   -   机器码中的 ModRM 操作码;  
  
   9   -   机器码中的 SIM 操作码;  
  
   10  -   机器码中的 Displacement 操作码;  
  
     每一项都由一 Function 读出, 其中一些还要做一些变换.  
  
     并不是每一项都存在于每一行机器码.   
  
  
4.  机器代码数据结构  
  
每一行机器代码对应的结构如下:  
  
typedef struct {  
  BYTE    FirstOpcode\_0;  
  BYTE    Unknown1;  
  BYTE    SecondImmediateData;  
  BYTE    Unknown2\[5\];  
  BYTE    SIMOpcode;  
  BYTE    Unknown3\[2\];  
  DWORD   DisplacementOpcode;  
  BYTE    Unknown4;  
    
  BYTE    FirstOpcode\_1; // if FirstOpCode is a prefix  
  BYTE    Unknown5\[3\];  
  DWORD   ImmediateDataOpcode;  
  DWORD   Unknown6;  
  BOOL    bAdjustValueFlag;  
  BYTE    Unknown7;  
  DWORD   EncryptedEIPAddress; //+1E , EncryptedEIPAddress + baseAddress + randxx ==>EIPAddress  
  BYTE    Unknown8\[2\];  
  BYTE    MathType; // Mathtype or an additional opcode  
  BYTE    Unknown9\[3\];  
  BYTE    ModRMOpcode;  
  DWORD   Unknown10;  
} ENC\_LINE;  
  
  
每一段代码由 n 行代码构成, 对应如下的结构:  
  
typedef struct {  
  DWORD   Unknown;  
  DWORD   pFirstItem;  
  DWORD   ItemNum;  
  BYTE    FuncIndex\[0x0A\];   // 00E632C0  01 06 05 00 08 04 03 07 02 09  
  BYTE    FuncIndex2\[0x0A\];  
  DWORD   Funcs\[0x0A\];  
/\*  
00E632D4  011F0000    // return \_\_0014     Func3  
00E632D8  011E0000    // return \_\_10       Func0  
00E632DC  01200000    // return \_\_1C       Func8  
00E632E0  011C0000    // return \_\_08       Func6  
00E632E4  01230000    // return \_\_28       Func5  
00E632E8  01220000    // return \_\_24       Func2  
00E632EC  011A0000    // return \_\_00       Func1  
00E632F0  011D0000    // return \_\_000B     Func7  
00E632F4  011B0000    // return \_\_02     Func4  
00E632F8  01210000    // return \_\_001E     Func9  
\*/  
  DWORD   ItemSize;  
  DWORD   BaseAddr;  
  DWORD   RandomXX; // +50  
  DWORD   procID;  
  DWORD   Size;  
  ENC\_LINE Lines\[0\];  
} ENC\_INFO;  
  
其中 FuncIndex\[\], FuncIndex2\[\], Funcs\[\], 每次运行都会随机重新排序, 但是   
  
for(i=0;i<0x0A;i++) {  
   j = FuncIndex\[i\];  
   Funcxx = Funcs\[j\];      // Funxx 跟 i 是一一对应的  
}  
  
这是 Funcxx 的返回值 与 i 的 对应图  
  
00E63310  A1 88 00 4A 7B A2 B0 2F 00 0F 18 00 00 00 00 C9  1?4?????6??7777?  
00E63320  00 54 F7 21 00 00 00 00 7D 05 54 89 00 9C 68 FD  0???3333????8?99  
00E63330  EC A1 7B BC 00 25 45 BF 00 61 08 4A F5           99??2???5????  
  
  
5.  虚拟机数据结构  
  
  
typedef struct {  
  DWORD   Unknown1\[8\];  // Drx\[8\]??  
#define EAX    REGs\[0\]  
#define ECX    REGs\[1\]  
#define EDX    REGs\[2\]  
#define EBX    REGs\[3\]  
#define ESP    REGs\[4\]  
#define EBP    REGs\[5\]  
#define ESI    REGs\[6\]  
#define EDI    REGs\[7\]  
  DWORD   REGs\[8\];  
  DWORD   Unknown2;  
  DWORD   EIP;  
  DWORD   EFlags;  
  DWORD   Unknown3;  
  DWORD   CS\_base, SS\_base, DS\_base, ES\_base, FS\_base, GS\_base; // +60  
  WORD    CS, SS, DS, ES, FS, GS;  // +78  
  BYTE    Unknown4\[0x0C\]; //+84  
  DWORD   LowOpNum; //+90  
  DWORD   HiOpNum;  //+94  
  DWORD   CurSegbase; //+98  
  BOOL    bAdjustValueFlag; //+9C  
  DWORD   seh; // +9D  
  BYTE    bPrefixFlag; //+A1  
  
  OpDataInfo Src; // +A2  
  OpDataInfo Dst; //+A7  
  
  DWORD   OperandSize; // +AC:  1 - byte ptr, 2 - word ptr, 4 - dword ptr  
  
#define X86\_FLAGS\_LOCK\_PREFIX         0x0001  
#define X86\_FLAGS\_REPNE\_PREFIX        0x0002  
#define X86\_FLAGS\_REP\_PREFIX          0x0004  
#define X86\_FLAGS\_CS\_PREFIX           0x0008  
#define X86\_FLAGS\_SS\_PREFIX           0x0010  
#define X86\_FLAGS\_DS\_PREFIX           0x0020  
#define X86\_FLAGS\_ES\_PREFIX           0x0040  
#define X86\_FLAGS\_FS\_PREFIX           0x0080  
#define X86\_FLAGS\_GS\_PREFIX           0x0100  
#define X86\_FLAGS\_OPERAND\_PREFIX      0x0200  
#define X86\_FLAGS\_ADDRESS\_PREFIX      0x0400  
  DWORD   BitFlags; // +B0  
  BYTE    Opcode;  
  DWORD   curCodeItem;  
  DWORD   nextCodeItem;  
} X86\_INFO;  
  
其中  
  
typedef struct {  
  union {  
    DWORD   RegIndex;  
    DWORD   Address;  
  } u;  
  BYTE RegMode; // 1: RegMode, 0: MemAddrMode, 2:ImmediateData, 4:MemAddrMode, dispx\[Regx\]?  
} OpDataInfo;  
  
  
当进入到 虚拟机 Function 地址后, 在栈上开出一片空间, 为 X86\_INFO, 代表了虚拟机的状态, 主要就是  
  
CPU 的各个寄存器, 及一些标志.  
  
  
6. 转储(DUMP) ENC\_INFO 和 ENC\_LINE\[\] 结构  
  
用 OD 附加在运行的程序上, 然后在 ASPR 所在的内存段, SEARCH 下面的代码  
  
addr\_xx:  
       68 00000000     PUSH 0  
       68 xxxxxxxx     PUSH addr\_xx  
       68 yyyyyyyy     PUSH addr\_yy  
       E8 zzzzzzzz     CALL func\_zz  
  
其中 addr\_yy 就是结构 ENC\_INFO 的地址, 根据其中的 ItemNum 和 ItemSize 可以确定需要 DUMP 的大小.  
  
附件中的 aspr\_x86\_dump\_info.txt 可用来帮助 DUMP  
  
  
7. 还原机器码  
  
刚开始以为, 可以在各个 Funcxx 中 设 断点来 DUMP 再还原机器码, 也不理想, 理由如下:  
  
  A.) 各个 Funcxx, 有的对应机器码, 有的不对应机器码;  
  
  B.) 有些即使对应机器码, 还要进行变换;  
  
  C.) 有些 Funcxx , 可能对应 1 BYTE, 1WORD, 或者 1DWORD, 与参数有关;  
  
  D.) 如果有循环, 就会有重复;  
    
  E.) 如果有跳转, 未运行到的 CODE 又没法 DUMP.  
  
所以还是把它的 X86 处理 CODE 分析清楚, 再写 CODE 还原;  
  
好在它是按标准的 Intel 的 X86 指令集来处理的, 参考 Intel 的 "Instruction Set Reference", 好多 CODE 很容易明白,   
  
刚开始还想把每一行机器码的反汇编弄出来, 试了以下, 有点累, 算了.   
  
附件中的 aspr\_x86.c 和 aspr\_x86.h 可用来处理 DUMP 出来的二进制数据, 还原机器码  
  
  
8. Reference  
  
A. ) 24319102.PDF, "Intel Architecture Software Developer's Manual Volume2: Instruction Set Reference";  
  
B. ) 被分析的程序: "Registry Defragmentation 8.2.6.11"   
  
  
8.  后记  
  
分析完 ASPR 的这段 CODE 后, 对 Intel 的 X86 指令集又熟悉了一些, 也算是有所收获.  
  
附件下载：aspr\_x86.rar   
  
以下是附件中的各个文件:  
  
A.) aspr\_x86\_code.txt           ASPR\_X86 CODE 跟踪分析笔记  
  
B.) aspr\_x86\_dump\_info.txt      ODbgScript 脚本, DUMP X86\_INFO  
  
C.) aspr\_x86.c & aspr\_x86.h     C 代码, 处理 DUMP 的 X86\_INFO 

 4dd466d0e1e0cc123afbe504da2462ca
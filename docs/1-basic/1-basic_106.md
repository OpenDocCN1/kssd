
# 写了个反汇编的东西，分享出来

> 原文：[`www.pediy.com/kssd/pediy12/128315.html`](https://www.pediy.com/kssd/pediy12/128315.html)

原来写了个反汇编的函数，源代码好像也不太给力。
就大概说说原理吧：
源代码中自定义了许多表：

```
// 定义标志寄存器结构
typedef struct _INQUIRE_EFLAGS
{
    DWORD dwUseField;      // 使用下列那些字段
    DWORD dwTest;             // 要测试的标记，位 1
    DWORD dwModif;           // 会影响的 32 位标志寄存器，位 2
    DWORD dwDefine;         // 已定义的标记，位 3
    DWORD dwUndefine;     // 未定义的标记，这个标记还在考虑中，位 4
    DWORD dwFixSet;           // 确定会被置 1 的标记，位 5
    DWORD dwFixZro;           // 确定会被清 0 的标记，位 6
} INQUIRE_EFLAGS, *LPINQUIRE_EFLAGS;
```
这个表，用于想要知道指令是否影响标记位的。

```
// ModR/M32 表专用宏
#define MODRM32_MEM_FREG    0x1      // 全寄存器寻址
#define MODRM32_MEM_FREG_IMM1  0x2      // 全寄存器加立即数 1 寻址
#define MODRM32_MEM_FREG_IMM4  0x3      // 全寄存器加立即数 4 寻址
#define MODRM32_MEM_IMM4    0x4      // 立即数 4 寻址
#define MODRM32_REG      0x5      // 寄存器作为操作数
#define MODRM32_SIB      0x6      // 存在 SIB

// ModR/M32 表
typedef struct _TabModRM32
{
    BYTE  dwModRMMode;              // 寻址模式
    BYTE  dwMod;
    BYTE  dwRegOp;
    BYTE  dwRM;
} TabModRM32;
```
这个表方便 Mod 如何解析的，反正就 255 种情况，我直接用程序生成这个表，直接查表，搞的现在我自己都忘了怎么解析 Mod 表，写的时候还知道。

```
// ModR/M16 表专用宏
#define MODRM16_MEM_2REG     0x1  //  [bx + si], REG
#define MODRM16_MEM_1REG     0x2  //  [bx + si], REG
#define MODRM16_MEM_2REG_IMM1 0x3  //  [bx + si + IMM1], REG
#define MODRM16_MEM_1REG_IMM1 0x4 // [bx + si + IMM1], REG
#define MODRM16_MEM_2REG_IMM2 0x5 //[bx + si + IMM2], REG
#define MODRM16_MEM_1REG_IMM2 0x6 // [bx + si + IMM2], REG
#define MODRM16_MEM_IMM2           0x7 // [IMM2], REG
#define MODRM16_REG                      0x8 // Opcode REG, REG
typedef struct _TabModRM16
{
  BYTE  dwModRMMode;              // 寻址模式
  BYTE  dwMod;
  BYTE  dwRegOp;
  BYTE  dwRM;
  BYTE  dwReg1;
  BYTE  dwReg2;
} TabModRM16;
```
同上，这个是 16 位的表

```
// SIB 表
typedef struct _TabSib
{
  BYTE  dwSibMode;              // 寻址模式
  BYTE  dwScaleN;               // 比例因子
  BYTE  dwIndex;             // 索引寄存器索引
  BYTE  dwBase;              // 基址寄存器索引
} TabSib;
```
SIB 也建表了

```
#define ADDR_BIT_1REG  BIT_4    // 地址中有 1 个寄存器
#define ADDR_BIT_2REG  (BIT_3 | BIT_4)  // 地址中有 2 个寄存器
#define ADDR_BIT_N  BIT_2    // 有比例因子
#define ADDR_BIT_IMM  BIT_1    // 有地址立即数
#define OPERAND_NULL 0x0  // 无操作数
// 最高位为 1 就是直接寻址，否则为间接寻址，直接寻址就是没有[]的
#define OPERAND_IMM (BIT_32 | ADDR_BIT_IMM)  // 立即数
#define OPERAND_REG (BIT_32 | ADDR_BIT_1REG)  // 寄存器
#define OPERAND_MEM_IMM  ADDR_BIT_IMM// 地址立即数
#define OPERAND_MEM_REG  ADDR_BIT_1REG  // 地址寄存器
#define OPERAND_MEM_REGIMM  (ADDR_BIT_1REG | ADDR_BIT_IMM)  // [REG + IMM]
#define OPERAND_MEM_2REG  ADDR_BIT_2REG// [REG1 + REG2]
#define OPERAND_MEM_2REGIMM  (ADDR_BIT_2REG | ADDR_BIT_IMM)  // [REG1 + REG2 + IMM]
#define OPERAND_MEM_1REGN  (ADDR_BIT_1REG | ADDR_BIT_N)  // [REG1 * N]
#define OPERAND_MEM_2REGN  (ADDR_BIT_2REG | ADDR_BIT_N)  // [REG1 + REG2 * N]
#define OPERAND_MEM_1REGNIMM (ADDR_BIT_1REG | ADDR_BIT_N | ADDR_BIT_IMM) // [REG1 * N + IMM]
#define OPERAND_MEM_2REGNIMM (ADDR_BIT_2REG | ADDR_BIT_N | ADDR_BIT_IMM)// [REG1 + REG2 * N + IMM]

// 其实 2，4 有符号数都是由 1BYTE 符号数扩展而来的
#define SIGN1IMM    0x1
#define SIGN2IMM    0x2
#define SIGN4IMM    0x3
#define UNSIGN1IMM    0x4
#define UNSIGN2IMM    0x5
#define UNSIGN4IMM    0x6
```
// 操作数结构
```
typedef struct  _UnImm
{
  BYTE ImmType;        // 立即数类型,或地址立即数类型
  union
  {
    CHAR  cImm;
    SHORT sImm;
    LONG  lImm;
    BYTE  Imm;
    WORD  wImm;
    DWORD dwImm;
  } AnyImm;
} UnImm;
```
// 操作数结构
```
typedef struct  _UnOperand
{
  DWORD AddrMode; // 表明本结构的意义和使用的字段
  BYTE Reg1;              // [base + index * n + IMM/1/4]
  BYTE Reg2;              //[Reg1 + Reg2 * n + IMM]
  BYTE N;                      // 比例因子，不存在就设置为 1,也用于表示立即数的大小
  UnImm stImm;  // 立即数类型,或地址立即数类型
} UnOperand;
```
自己定义的一个操作数结构，用于解析指令的参数时好做判断

最后输出怎么一个结构。
```
typedef struct _InstructionFrame
{
    DWORD  dwInstructionAddr;  // 指令地址
    DWORD  dwInstruction;  // 指令
    BYTE  dwInstructionLen;// 指令机器码长
    BYTE  arInstructionCode[INSTRUCTION_LEN]; // 机器码
    DWORD  dwGrp;  // 指令组
    DWORD  dwRing;  // 指令特权级
    DWORD dwLock;  // 是否有锁前缀
    DWORD  dwRep;  // 是否有 Rep 前缀
    BYTE dwAddrSize; // 地址大小
    BYTE dwSegment;// 段前缀
    UnOperand stOpernd[3];  // 3 个参数内容
    INQUIRE_EFLAGS  InquireEflags;  // 影响的标记位
} InstructionFrame;
```
把结构传入下面这个函数就可以返回指令的字符串，`In_dwMnmicLen` 为对齐，就是质量和参数间的距离，`In_dwl再Case` 为大写和小写。
```
DWORD Instruction(InstructionFrame *Out_InsFrame, TCHAR* Out_pMnemonic,  DWORD dwLen, DWORD In_dwMnmicLen = 6, DWORD In_dwl再Case = 0);
```
原理其实好简单，就是不停的查表。

一共有：
1byte 指令表
2byte 指令表
扩展指令表
后缀指令表
前缀指令表
9b 指令表----------------这个 9b 好像很特别，记得我用 od 和 ida 测试了好多次

总之第一次写，理解不够深刻吧，建了怎么多表
比如 查 1byte 指令表先，发现时 0f 就跳到 2byte 指令表去，mod 中是扩展指令，就查扩展指令表，然后有的指令，不算 2byte，存在扩展指令，而且后面一个字节又可以代表一个指令，所以就有了后缀指令表(而且是先查后缀，我没记错的话)，前缀指令表就主要用于 0F 38 如 66 0f 38,什么 66 67 f2 f3 原来那么多用途，我还是第一次知道。

最后就定位到一个表，然后到其中去取得解析方式就 ok 了

有 bug 就告诉我吧！

望大虾提出意见，好提高一下，如果现在让我写，我可能还会设计成这样，只是细节的一些东西可能会做的更好。

代码中用了许多 goto，好纠结这个 goto。

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || Disassem.rar  |

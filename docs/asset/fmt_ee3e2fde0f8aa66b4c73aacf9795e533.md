
```c
typedef struct t_addrdec
{
    int        defseg;
    char       *descr;
} t_addrdec;

// 命令数据
typedef struct t_cmddata
{
    ulong      mask;                 // Mask for first 4 bytes of the command
    ulong      code;                 // Compare masked bytes with this
    char       len;                  // Length of the main command code
    // 例如 `PR`，则表明该指令是一特权指令(ring 0 指令)
    // `FF`：强制操作数为 16 位操作数
    char       bits;                 // Special bits within the command
    char       arg1, arg2, arg3;     // Types of possible arguments
    char       type;                 // `C_xxx` + additional information
    char       *name;                // Symbolic name for this command
} t_cmddata;

// Initialized constant data structures used by all programs from assembler
// package. Contain names of register, register combinations or commands and
// their properties.
extern const char      *regname[3][9];
extern const char      *segname[8];
extern const char      *sizename[11];
extern const t_addrdec addr16[8];
extern const t_addrdec addr32[8];
extern const char      *fpuname[9];
extern const char      *mmxname[9];
extern const char      *crname[9];
extern const char      *drname[9];
extern const char      *condition[16];
extern const t_cmddata cmddata[];//可用结构数组
extern const t_cmddata vxdcmd;
extern const t_cmddata dangerous[];
```

```c
////////////////////////////////////////////////////////////////////////////////
//////////////////// ASSEMBLER, DISASSEMBLER AND EXPRESSIONS ///////////////////

#define MAXCMDSIZE     16            // Maximal length of 80x86 command
#define MAXCALSIZE     8             // Max length of `CALL` without prefixes
#define NMODELS        8             // Number of assembler search models
#define INT3           0xCC          // Code of 1-byte breakpoint
#define NOP            0x90          // Code of 1-byte `NOP` command
#define TRAPFLAG       0x00000100    // Trap flag in CPU flag register

// 指令类型
#define REG_EAX        0             // Indexes of general-purpose registers
#define REG_ECX        1             // in `t_reg`.
#define REG_EDX        2
#define REG_EBX        3
#define REG_ESP        4
#define REG_EBP        5
#define REG_ESI        6
#define REG_EDI        7

// 段寄存器
#define SEG_UNDEF      -1            //没有段寄存器
#define SEG_ES         0             // Indexes of segment/selector registers
#define SEG_CS         1
#define SEG_SS         2
#define SEG_DS         3
#define SEG_FS         4
#define SEG_GS         5

// 指令类型
#define C_TYPEMASK     0xF0          // Mask for command type
#define   C_CMD        0x00          // Ordinary instruction
#define   C_PSH        0x10          // 1-word `PUSH` instruction
#define   C_POP        0x20          // 1-word `POP` instruction
#define   C_MMX        0x30          // MMX instruction
#define   C_FLT        0x40          // FPU instruction

// jmp 跳转指令
#define   C_JMP        0x50          // JUMP instruction

// jmp 条件跳转指令
#define   C_JMC        0x60          // Conditional JUMP instruction

// call 调用指令
#define   C_CAL        0x70          // CALL instruction
#define   C_RET        0x80          // RET instruction
#define   C_SYS        0x90          // Changes system flags
#define   C_RTF        0xA0          // `C_JMP` and `C_SYS` simultaneously
#define   C_REP        0xB0          // Instruction with `REPxx` prefix
#define   C_PRI        0xC0          // Privileged instruction
#define   C_DAT        0xD0          // Data (address) doubleword

// 3DNow 指令
#define   C_NOW        0xE0          // 3DNow! instruction

// 错误指令，即未注册的指令
#define   C_BAD        0xF0          // Unrecognized command

#define C_RARE         0x08          // Rare command, seldom used in programs
#define C_SIZEMASK     0x07          // MMX data size or special flag
#define   C_EXPL       0x01          // (non-MMX) Specify explicit memory size

#define C_DANGER95     0x01          // Command is dangerous under Win95/98
#define C_DANGER       0x03          // Command is dangerous everywhere
#define C_DANGERLOCK   0x07          // Dangerous with `LOCK` prefix

#define DEC_TYPEMASK   0x1F          // Type of memory byte
#define   DEC_UNKNOWN  0x00          // Unknown type
#define   DEC_BYTE     0x01          // Accessed as byte
#define   DEC_WORD     0x02          // Accessed as short
#define   DEC_NEXTDATA 0x03          // Subsequent byte of code or data
#define   DEC_DWORD    0x04          // Accessed as long
#define   DEC_FLOAT4   0x05          // Accessed as float
#define   DEC_FWORD    0x06          // Accessed as descriptor/long pointer
#define   DEC_FLOAT8   0x07          // Accessed as double
#define   DEC_QWORD    0x08          // Accessed as 8-byte integer
#define   DEC_FLOAT10  0x09          // Accessed as long double
#define   DEC_TBYTE    0x0A          // Accessed as 10-byte integer
#define   DEC_STRING   0x0B          // Zero-terminated ASCII string
#define   DEC_UNICODE  0x0C          // Zero-terminated UNICODE string

// 3DNow 指令操作数
#define   DEC_3DNOW    0x0D          // Accessed as 3Dnow operand
#define   DEC_BYTESW   0x11          // Accessed as byte index to switch
#define   DEC_NEXTCODE 0x13          // Subsequent byte of command
#define   DEC_COMMAND  0x1D          // First byte of command
#define   DEC_JMPDEST  0x1E          // Jump destination
#define   DEC_CALLDEST 0x1F          // Call (and maybe jump) destination

#define DEC_PROCMASK   0x60          // Procedure analysis
#define   DEC_PROC     0x20          // Start of procedure
#define   DEC_PBODY    0x40          // Body of procedure
#define   DEC_PEND     0x60          // End of procedure

#define DEC_CHECKED    0x80          // Byte was analysed

#define DECR_TYPEMASK  0x3F          // Type of register or memory
#define   DECR_BYTE    0x21          // Byte register
#define   DECR_WORD    0x22          // Short integer register
#define   DECR_DWORD   0x24          // Long integer register
#define   DECR_QWORD   0x28          // MMX register
#define   DECR_FLOAT10 0x29          // Floating-point register
#define   DECR_SEG     0x2A          // Segment register
#define   DECR_3DNOW   0x2D          // 3Dnow! register

#define DECR_ISREG     0x20          // Mask to check that operand is register

/**
 * 反汇编的选项
 */
#define DISASM_SIZE    0             // Determine command size only
#define DISASM_DATA    1             // Determine size and analysis data
#define DISASM_FILE    3             // Disassembly, no symbols
#define DISASM_CODE    4             // Full disassembly

// Warnings issued by `Disasm()`:
#define DAW_FARADDR    0x0001        // Command is a far jump, call or return

// 当执行如 `MOV SEG, XX` 的指令时警告
#define DAW_SEGMENT    0x0002        // Command loads segment register

// 特权指令
#define DAW_PRIV       0x0004        // Privileged command

// IO 指令
#define DAW_IO         0x0008        // I/O command
#define DAW_SHIFT      0x0010        // Shift constant out of range 1..31

// 表示指令有过多的前缀
#define DAW_PREFIX     0x0020        // Superfluous prefix

// 指令有加锁总线前缀
#define DAW_LOCK       0x0040        // Command has `LOCK` prefix

// 栈没有对齐(一般栈操作需要双字对齐)
#define DAW_STACK      0x0080        // Unaligned stack operation
#define DAW_DANGER95   0x1000        // May mess up Win95 if executed
#define DAW_DANGEROUS  0x3000        // May mess up any OS if executed
```

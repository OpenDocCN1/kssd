
`01 : 110 : 111  |  mov reg, [2 * edi + esi]`

`10 : 010 : 011  |  mov reg, [4 * edx + ebx]`

`11 : 011 : 000  |  mov reg, [8 * eax + ebx]`

- 如果索引寄存器为 ESP，则 index 被忽略，在这个例子里 scale 也被忽略，只有 base register 被使用计算地址
- 如果我们需要如下编码 `add reg, [esp]` 它可以使用 SIB 字节，但是它被编码成这样
  `add reg, [esp + DISPLACEMENT]`

```
*****************
**      **
** DISPLACEMENT **
**      **
*****************
```

Just a quik example with the instruction `8b bd 78563412` :

```
                         8b      bd    78563412
                       [code] [MorR/M]  [???]
                        /         \
                       /           \
                      /             \
                     /               \
+----------------;               +-----+--------+-------+
; instr  ; d ; w ;               ; Mod ;  Reg/  ; R/M   ;
+--------+---+---;               ;     ; opcode ;       ;
; 100010 ; 1 ; 1 ;               +-----+--------+-------+
+--------+---+---;               ; 1 0 ; 1 1 1  ; 1 0 1 ;
(d)=0 : the order is             +-----+--------+-------+
        REG1-REG2                         EDI      EBP
(w)=1 : 32 bits mode                   or code
instr : MOV r32,r/m32                 extension
```

`[code]`       = `8b`        : opcode
                         ---> MOV r32,r/m32

`[Mod]`        = `10`        : memory address with 1 dword displacement
                         ---> MOV REG,[REG+DWORD]

`[Reg/opcode]` = `111`       : this instruction don't need an code extension
                           so it is a register
                         ---> MOV EDI,[EBP+DWORD]

`[R/M]`        = `101`       : there is no `[SIB]` after the byte`[ModR/M]`

`[78563412]`   = `12345678h` : it is the deplacement
                         ---> MOV EDI,[EBP+12345678h]

So the instruction `8b bd 78563412` is `mov edi,[ebp+12345678h]`

```
*****************
**     **
**  IMMEDIATE  **
**     **
*****************
```

expl:   `05h`          is the opcode for `ADD EAX,imm32`
        `05h 00000010` is the instruction for `add eax, 10000000`

```
*******************************
**                      **
**  ALGO OF DISASSEMBLY   **
**                      **
*******************************
```

```
       !
      \!/
 +-----'-------+
 ; Read 1 byte ; <---------
 +-------------+
       !                    \
      \!/                    --------<----- ; Convert to ASCII ;
 +-----'------------+                       +-------.----------+
 ; Check for prefix ;                              /!\ 
 +------------------+                               !
       !                                            !
      \!/                                   +-----------------+
 ;-----'----------------+                   ; [IMMEDIATE]     ;
 ; Decode [code] byte(s);                   +-----------------+
 +----------------------+                   ; [DISPLACEMENT]  ;
       !                                    +-----------------+
      \!/                                       ; decode [SIB]    ;
 +-----'-----------------+                  +-----------------+
 ; JMP or CALL opcodes ? ; -------->------> ; decode [ModR/M] ;
 +-----------------------+                  +-----------------+
```

下面是我注释的 `ADE32` 的代码，很多语义表示不清楚，大家凑合看吧，偶水平不行啊![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!")

```
// ADE32 2.03c -- advanced 16/32-bit opcode assembler/disassembler engine
```

// this stuff is used to get instruction length and parse it into
// prefix, opcode, modregr/m, address, immediate, etc.
//指令特征
```c
#define C_ERROR   0xFFFFFFFF
#define C_ADDR1   0x00000001  //操作码中地址大小的位字段(字节)
#define C_ADDR2   0x00000002
#define C_ADDR4   0x00000004  //(双字)
#define C_LOCK    0x00000008  //加锁前缀
#define C_67      0x00000010  //地址大小修饰前缀(16/32 位)
#define C_66      0x00000020  //操作数大小修饰前缀(16/32 位)
#define C_REP     0x00000040  //重复前缀
#define C_SEG     0x00000080  //段寄存器前缀
#define C_ANYPREFIX  (C_66+C_67+C_LOCK+C_REP+C_SEG)
#define C_DATA1   0x00000100  //操作码中数据大小的位字段
#define C_DATA2   0x00000200
#define C_DATA4   0x00000400
#define C_SIB     0x00000800  //SIB 字节
#define C_ADDR67  0x00001000  //地址字节数为 disasm_defaddr
#define C_DATA66  0x00002000  //数据字节数为 disasm_defdata
#define C_MODRM   0x00004000  //MODRM 字节
#define C_BAD     0x00008000
#define C_OPCODE2 0x00010000  //操作码第二个字节
#define C_REL     0x00020000  // 这是跳转指令 jxx 或者 call
#define C_STOP    0x00040000  // 这是回跳指令，ret 或者 jmp
```

//指令信息
```c
struct disasm_struct
{
  BYTE  disasm_defaddr;         // 00
  BYTE  disasm_defdata;         // 01
  DWORD disasm_len;             // 02 03 04 05
  DWORD disasm_flag;            // 06 07 08 09
  DWORD disasm_addrsize;        // 0A 0B 0C 0D
  DWORD disasm_datasize;        // 0E 0F 10 11
  BYTE  disasm_rep;             // 12
  BYTE  disasm_seg;             // 13
  BYTE  disasm_opcode;          // 14
  BYTE  disasm_opcode2;         // 15
  BYTE  disasm_modrm;           // 16
  BYTE  disasm_sib;             // 17
  //地址
  union
  {
  BYTE  disasm_addr_b[8];       // 18 19 1A 1B  1C 1D 1E 1F
  WORD  disasm_addr_w[4];
  DWORD disasm_addr_d[2];
  char  disasm_addr_c[8];
  short disasm_addr_s[4];
  long  disasm_addr_l[2];
  };
  //数据
  union
  {
  BYTE  disasm_data_b[8];       // 20 21 22 23  24 25 26 27
  WORD  disasm_data_w[4];
  DWORD disasm_data_d[2];
  char  disasm_data_c[8];
  short disasm_data_s[4];
  long  disasm_data_l[2];
  };
};
```
```c
// disasm_struct
//按操作码从 0x00 开始排列，每一行代表一个操作
//码对应的指令特征的集合
DWORD ade32_table[512] = {
 /* 00 */  C_MODRM,
 /* 01 */  C_MODRM,
 /* 02 */  C_MODRM,
 /* 03 */  C_MODRM,
 /* 04 */  C_DATA1,
 /* 05 */  C_DATA66,
 /* 06 */  C_BAD,
 /* 07 */  C_BAD,
 /* 08 */  C_MODRM,
 /* 09 */  C_MODRM,
 /* 0A */  C_MODRM,
 /* 0B */  C_MODRM,
 /* 0C */  C_DATA1,
 /* 0D */  C_DATA66,
 /* 0E */  C_BAD,
 /* 0F */  C_OPCODE2,
 /* 10 */  C_MODRM+C_BAD,
 /* 11 */  C_MODRM,
 /* 12 */  C_MODRM+C_BAD,
 /* 13 */  C_MODRM,
 /* 14 */  C_DATA1+C_BAD,
 /* 15 */  C_DATA66+C_BAD,
 /* 16 */  C_BAD,
 /* 17 */  C_BAD,
 /* 18 */  C_MODRM+C_BAD,
 /* 19 */  C_MODRM,
 /* 1A */  C_MODRM,
 /* 1B */  C_MODRM,
 /* 1C */  C_DATA1+C_BAD,
 /* 1D */  C_DATA66+C_BAD,
 /* 1E */  C_BAD,
 /* 1F */  C_BAD,
 /* 20 */  C_MODRM,
 /* 21 */  C_MODRM,
 /* 22 */  C_MODRM,
 /* 23 */  C_MODRM,
 /* 24 */  C_DATA1,
 /* 25 */  C_DATA66,
 /* 26 */  C_SEG+C_BAD,
 /* 27 */  C_BAD,
 /* 28 */  C_MODRM,
 /* 29 */  C_MODRM,
 /* 2A */  C_MODRM,
 /* 2B */  C_MODRM,
 /* 2C */  C_DATA1,
 /* 2D */  C_DATA66,
 /* 2E */  C_SEG+C_BAD,
 /* 2F */  C_BAD,
 /* 30 */  C_MODRM,
 /* 31 */  C_MODRM,
```

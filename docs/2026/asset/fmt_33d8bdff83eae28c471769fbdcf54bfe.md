
```
#define N_01        0x2
#define N_10        0x4
#define N_11        0x8

#define INDEX_0        0x0
#define INDEX_1        0x1
#define INDEX_2        0x2
#define INDEX_3        0x3
#define INDEX_4        0x4
#define INDEX_5        0x5
#define INDEX_6        0x6
#define INDEX_7        0x7

#define BASE_0        0x0
#define BASE_1        0x1
#define BASE_2        0x2
#define BASE_3        0x3
#define BASE_4        0x4
#define BASE_5        0x5
#define BASE_6        0x6
#define BASE_7        0x7
```

## SIB 表

```c
struct SIB
{
  uint32   Mode;            // 寻址模式
  uint32   N;         // 比例因子
  uint32   Index;           // 索引寄存器索引
  uint32   Base;            // 基址寄存器索引
};
```

然后表的内容：

```c
ModRM32 g_TabModRM32[0x100] =
{
  /* 0x00 */   { MODRM32_MEM_FREG       , MOD_00, REG_OP_0, RM_0 },
  /* 0x01 */   { MODRM32_MEM_FREG       , MOD_00, REG_OP_0, RM_1 },
  /* 0x02 */   { MODRM32_MEM_FREG       , MOD_00, REG_OP_0, RM_2 },
  /* 0x03 */   { MODRM32_MEM_FREG       , MOD_00, REG_OP_0, RM_3 },
  /* 0x04 */   { MODRM32_SIB           , MOD_00, REG_OP_0, RM_4 },
  /* 0x05 */   { MODRM32_MEM_IMM4       , MOD_00, REG_OP_0, RM_5 },
        ......
}

ModRM16 g_TabModRM16[0x100] =
{
  /* 0x0 */   { MODRM16_MEM_2REG      , MOD_00, REG_OP_0, RM_0, R_BX, R_SI },
  /* 0x1 */   { MODRM16_MEM_2REG      , MOD_00, REG_OP_0, RM_1, R_BX, R_DI },
  /* 0x2 */   { MODRM16_MEM_2REG      , MOD_00, REG_OP_0, RM_2, R_BP, R_SI },
  /* 0x3 */   { MODRM16_MEM_2REG      , MOD_00, REG_OP_0, RM_3, R_BP, R_DI },
  /* 0x4 */   { MODRM16_MEM_1REG      , MOD_00, REG_OP_0, RM_4, R_SI, R_NO },
  /* 0x5 */   { MODRM16_MEM_1REG      , MOD_00, REG_OP_0, RM_5, R_DI, R_NO },
  /* 0x6 */   { MODRM16_MEM_IMM2      , MOD_00, REG_OP_0, RM_6, R_NO, R_NO },
  /* 0x7 */   { MODRM16_MEM_1REG      , MOD_00, REG_OP_0, RM_7, R_BX, R_NO },
        ....
}
```

等表……

解析 mod 和 sib 部分就这样：

```c
          // 32位寻址址模式
          switch(TabModRM->Mode)
          {
            // 全寄存器寻址即：Mod == 00, R/M != 100, R/M != 101  如：Opcode [FEG], REG
          case MODRM32_MEM_FREG:
            {
              T->Type = OPERAND_MEM;
              T->u.Mem.Type = MEM_REG;
              T->u.Mem.Size = SizeAddress;
              T->u.Mem.Segment = Prefix[INDEX_SEG];
              T->u.Mem.u.Reg = g_TabReg[R_INDEXREG32][TabModRM->RM];
            }
            break;

            // 全寄存器加立即数1寻址即：Mod == 01, R/M != 100  如：Opcode [FEG + IMM1], REG
          case MODRM32_MEM_FREG_IMM1:
            {
              T->Type = OPERAND_MEM;
              T->u.Mem.Type = MEM_REGIMM;
              T->u.Mem.Size = SizeAddress;
              T->u.Mem.Segment = Prefix[INDEX_SEG];
              T->u.Mem.u.RegImm.Reg = g_TabReg[R_INDEXREG32][TabModRM->RM];
              T->u.Mem.u.RegImm.Imm.Size = IMM8;
              T->u.Mem.u.RegImm.Imm.u.Imm8 = *(int8*)(Code + Offset + 1);
              ++Offset;
            }
            break;
```

源码见附件：

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [DisasmEngine.rar](javascript:void(0);)

  （30.21kb，119次下载）

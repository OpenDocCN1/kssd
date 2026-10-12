
```c
match |= MA_TYP;

if (op->size != 0 && op->size != 6)
  match |= MA_SIZ;

break;

case MMB:          // Two adjacent memory locations (BOUND)
  if (op->type != MRG)
    match |= MA_TYP;

  k = op->size;

  if (ideal == 0 && k > 1)
    k /= 2;

  if (k != 0 && k != datasize)
    match |= MA_DIF;

  break;

case MD2:          // Memory in ModRM byte (16-bit integer)
case MB2:          // Memory in ModRM byte (16-bit binary)
  if (op->type != MRG)
    match |= MA_TYP;

  if (op->size != 0 && op->size != 2)
    match |= MA_SIZ;

  break;

case MD4:          // Memory in ModRM byte (32-bit integer)
case MF4:          // Memory in ModRM byte (32-bit float)
  if (op->type != MRG)
    match |= MA_TYP;

  if (op->size != 0 && op->size != 4)
    match |= MA_SIZ;

  break;

case MD8:          // Memory in ModRM byte (64-bit integer)
case MF8:          // Memory in ModRM byte (64-bit float)
  if (op->type != MRG)
    match |= MA_TYP;

  if (op->size != 0 && op->size != 8)
    match |= MA_SIZ;

  break;

case MDA:          // Memory in ModRM byte (80-bit BCD)
case MFA:          // Memory in ModRM byte (80-bit float)
  if (op->type != MRG)
    match |= MA_TYP;

  if (op->size != 0 && op->size != 10)
    match |= MA_SIZ;

  break;

case MFE:          // Memory in ModRM byte (FPU environment)
case MFS:          // Memory in ModRM byte (FPU state)
case MFX:          // Memory in ModRM byte (ext. FPU state)
  if (op->type != MRG)
    match |= MA_TYP;

  if (op->size != 0)
    match |= MA_SIZ;

  break;

case MSO:          // Source in string operands ([ESI])
  if (op->type != MRG || op->base != REG_ESI || op->index != -1 ||
    op->offset != 0 || op->anyoffset != 0)
    match |= MA_TYP;

  if (datasize == 0)
    datasize = op->size;

  if (op->size != 0 && op->size != datasize)
    match |= MA_DIF;

  break;

case MDE:          // Destination in string operands ([EDI])
  if (op->type != MRG || op->base != REG_EDI || op->index != -1 ||
    op->offset != 0 || op->anyoffset != 0)
    match |= MA_TYP;

  if (op->segment != SEG_UNDEF && op->segment != SEG_ES)
    match |= MA_SEG;

  if (datasize == 0)
    datasize = op->size;

  if (op->size != 0 && op->size != datasize)
    match |= MA_DIF;

  break;

case MXL:          // XLAT operand ([EBX+AL])
  if (op->type != MXL)
    match |= MA_TYP;

  break;

  // 操作数是一直接数
case IMM:          // Immediate data (8 or 16/32)
case IMU:          // Immediate unsigned data (8 or 16/32)
  if (op->type != IMM)
    match |= MA_TYP;

  break;

case VXD:          // VxD service (32-bit only)
  if (op->type != IMM)
    match |= MA_TYP;

  if (datasize == 0)
    datasize = 4;

  if (datasize != 4)
    match |= MA_SIZ;

  break;

  // jump/call 的直接数地址
case JMF:          // Immediate absolute far jump/call addr
  if (op->type != JMF)
    match |= MA_TYP;

  // 地址修饰出错
  if ((jmpmode & 0x05) != 0)
    match |= MA_JMP;

  jmpmode &= 0x7F;

  break;

case JOB:          // Immediate byte offset (for jumps)
  if (op->type != IMM || longjump)
    match |= MA_TYP;

  if ((jmpmode & 0x0A) != 0)
    match |= MA_JMP;

  jmpmode &= 0x7F;

  break;

case JOW:          // Immediate full offset (for jumps)
  if (op->type != IMM)
    match |= MA_TYP;

  if ((jmpmode & 0x09) != 0)
    match |= MA_JMP;

  jmpmode &= 0x7F;

  break;

case IMA:          // Immediate absolute near data address
  if (op->type != MRG || op->base >= 0 || op->index >= 0)
    match |= MA_TYP;

  break;

case IMX:          // Immediate sign-extendable byte
  if (op->type != IMM)
    match |= MA_TYP;

  if (op->offset < -128 || op->offset > 127)
    match |= MA_RNG;

  break;

case C01:          // Implicit constant 1 (for shifts)
  if (op->type != IMM || (op->offset != 1 && op->anyoffset == 0))
    match |= MA_TYP;

  break;

case IMS:          // Immediate byte (for shifts)
case IM1:          // Immediate byte
  if (op->type != IMM)
    match |= MA_TYP;

  // 常数范围有误
  if (op->offset < -128 || op->offset > 255)
    match |= MA_RNG;

  break;

case IM2:          // Immediate word (ENTER/RET)
  if (op->type != IMM)
    match |= MA_TYP;

  if (op->offset < 0 || op->offset > 65535)
    match |= MA_RNG;

  break;

  // 段寄存器
case SGM:          // Segment register in ModRM byte
  if (op->type != SGM)
    match |= MA_TYP;

  if (datasize == 0)
    datasize = 2;

  if (datasize != 2)
    match |= MA_DIF;

  break;

  //
case SCM:          // Segment register in command byte
  if (op->type != SGM)
    match |= MA_TYP;

  break;

  // 控制寄存器
case CRX:          // Control register CRx
case DRX:          // Debug register DRx
  if (op->type != arg)
    match |= MA_TYP;

  if (datasize == 0)
    datasize = 4;

  if (datasize != 4)
    match |= MA_DIF;

  break;

case PRN:          // Near return address (pseudooperand)
case PRF:          // Far return address (pseudooperand)
case PAC:          // Accumulator (AL/AX/EAX, pseudooperand)
case PAH:          // AH (in LAHF/SAHF, pseudooperand)
case PFL:          // Lower byte of flags (pseudooperand)
case PS0:          // Top of FPU stack (pseudooperand)
case PS1:          // ST(1) (pseudooperand)
case PCX:          // CX/ECX (pseudooperand)
case PDI:          // EDI (pseudooperand in MMX extention)
  break;

  // 错误操作数
default:          // Undefined type of operand
  strcpy(errtext, "Internal Assembler error");
  goto error;
};      // End of switch (arg)

// 在 switch(arg)中会对 jmpmode 进行 & 0x7F 的计算，所以其 0x80
// 位正常应该为 0，所以，如果不为 0，则说明出错了
if ((jmpmode & 0x80) != 0)
  match |= MA_JMP;

// 如果操作数不匹配，则出错
if (match != 0)
  break;      // Some of the operands doesn't match
};      // End of operand matching loop

// 如果所有操作数都匹配
if (match == 0)      // Exact match found
{
  // 允许尝试多次
  if (attempt > 0)
  {
    --attempt;
    nameok = 0;
  }    // Well, try to find yet another match
  else
    break;
};

 };      // End of command search loop : for (pd = cmddata; pd->mask != 0; pd++)

// 至此，对操作数的验证就结束了
// Check whether some error was detected. If several errors were found
// similtaneously, report one (roughly in order of significance).
// 如果助记符无效，则出错
if (nameok == 0)  // Mnemonic unavailable
{
  strcpy(errtext, "Unrecognized command");
  asmcmd = nameend;
  goto error;
};

// 操作数不匹配，则出错
```

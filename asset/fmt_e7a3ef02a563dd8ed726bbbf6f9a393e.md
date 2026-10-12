
```
if((pd->type & C_TYPEMASK) == C_NOW)
{
  is3dnow = 1;
  j = Get3dnowsuffix(src, size);
  if(j < 0)
  {
    g_nException = 1;
  }
  else
  {
    for( ; pd->mask != 0; pd++)
    {
      if(((code ^ pd->code) & pd->mask) != 0)    continue;
      if(((u8 *)&(pd->code))[2] == j)        break;
    }
  }
}

g_pcmddata = (t_cmddata *)pd; //查找到的 opcode 详细信息

//当前 opcode 非法
if(pd->mask == 0)
{
  //非法
  if(size < 2)
  {
    g_nException = 1;
  }
  else
  {
    g_nException = 1;
  }
}
else
{
  //不非法情况
  if(pd->len == 2)
  {
    if(size == 0)
    {
      g_nException = 1;
    }
    else
    {
      g_nPrefixLen++; //对两个字节 opcode，g_nPrefixLen 其实不能加 1 的，
      src++;
      size--;
    }
  }

  g_disamInfo.pOpcodeAdd = src;
  g_disamInfo.pCmdDetail = (void *)pd;

  if(size == 0)
  {
    g_nException = 1;
  }

  if((pd->bits & WW) != 0 && (*src & WW) == 0)
    g_nDatasize = 1;
  else if((pd->bits & W3) != 0 && (*src & W3) == 0)
    g_nDatasize = 1;
  else if((pd->bits & FF) != 0)
    g_nDatasize = 2;

  //识别 operand
  for(operand = 0; operand < 3; operand++)
  {
    nMemoryOnly = 1;

    if(g_nException != 0)  break;  //如果有错误

    //获取 operand 类型
    if(operand == 0)  arg = (u8)pd->arg1;
    else if(operand == 1)  arg = (u8)pd->arg2;
    else      arg = (u8)pd->arg3;

    if(arg == NNN)    break;  //没有 operand 了

    switch(arg)
    {
    case REG:                      // Integer register in Reg field
    case RG4:                      // Integer 4-byte register in Reg field
    case RMX:                      // MMX register MMx
    case R3D:                      // 3DNow! register MMx
    case SGM:                      // Segment register in ModRM byte
      if(size < 2) g_nException = 1;
      hasrm = 1;
      break;
    case RCM:                      // Integer register in command byte
    case RAC:                      // Accumulator (AL/AX/EAX, implicit)
    case RAX:                      // AX (2-byte, implicit)
    case RDX:                      // DX (16-bit implicit port address)
    case RCL:                      // Implicit CL register (for shifts)
    case RS0:                      // Top of FPU stack (ST(0))
    case RST:                      // FPU register (ST(i)) in command byte
    case MSO:                      // Source in string op's ([ESI])
    case MDE:                      // Destination in string op's ([EDI])
    case MXL:                      // XLAT operand ([EBX+AL])
    case PRN:                      // Near return address (pseudooperand)
    case PRF:                      // Far return address (pseudooperand)
    case PAC:                      // Accumulator (AL/AX/EAX, pseudooperand)
    case PAH:                      // AH (in LAHF/SAHF, pseudooperand)
    case PFL:                      // Lower byte of flags (pseudooperand)
    case PCX:                      // CX/ECX (pseudooperand)
    case PDI:                      // EDI (pseudooperand in MMX extentions)
    case PS0:                      // Top of FPU stack (pseudooperand)
    case PS1:                      // ST(1) (pseudooperand)
    case SCM:                      // Segment register in command byte
    case C01:                      // Implicit constant 1 (for shifts)
      break;
    case MRG:                      // Memory/register in ModRM byte
    case MRJ:                      // Memory/reg in ModRM as JUMP target
    case MR1:                      // 1-byte memory/register in ModRM byte
    case MR2:                      // 2-byte memory/register in ModRM byte
    case MR4:                      // 4-byte memory/register in ModRM byte
    case MR8:                      // 8-byte memory/MMX register in ModRM
    case MRD:                      // 8-byte memory/3DNow! register in ModRM
      nMemoryOnly = 0;
    case MMA:                      // Memory address in ModRM byte for LEA
    case MML:                      // Memory in ModRM byte (for LES)
    case MM6:                      // Memory in ModRm (6-byte descriptor)
    case MMB:                      // Two adjacent memory locations (BOUND)
    case MD2:                      // Memory in ModRM byte (16-bit integer)
    case MB2:                      // Memory in ModRM byte (16-bit binary)
    case MD4:                      // Memory in ModRM byte (32-bit integer)
    case MD8:                      // Memory in ModRM byte (64-bit integer)
    case MDA:                      // Memory in ModRM byte (80-bit BCD)
    case MF4:                      // Memory in ModRM byte (32-bit float)
    case MF8:                      // Memory in ModRM byte (64-bit float)
    case MFA:                      // Memory in ModRM byte (80-bit float)
    case MFE:                      // Memory in ModRM byte (FPU environment)
    case MFS:                      // Memory in ModRM byte (FPU state)
    case MFX:                      // Memory in ModRM byte (ext. FPU state)
    case MMS:                      // Memory in ModRM byte (as SEG:OFFS)
    case RR4:                      // 4-byte memory/register (register only)
    case RR8:                      // 8-byte MMX register only in ModRM
    case RRD:                      // 8-byte memory/3DNow! (register only)
      if(arg == RR4 || arg == RR8 || arg == RRD)
      {
        if((src[1] & 0xC0) != 0xC0) g_nException = 1;
      }

      if(size < 2)  g_nException = 1;
      hasrm = 1;  //有 modR/M
      nCommand = src[1] & 0xC7;  //modR/M 字段去掉 opcode 部分
      if((nCommand & 0xC0) == 0xC0)  //operand 是寄存器
      {
        if(nMemoryOnly == 1)  g_nException = 1;
      }
      else if(g_nAddrsize == 2)  //16 位寻址模式
      {
        if(nCommand == 0x06)
        {
          dispsize = 2;
          if(size < 4)  g_nException = 1;  //4=1opcode+1modR/M+2disp
        }
        else
        {
          if((nCommand & 0xC0) == 0x40)  //mod=01
          {
            if(size < 3)  g_nException = 1;
            dispsize=1;
          }
          else if((nCommand & 0xC0) == 0x80)  //mod=10
          {
            if(size < 4)  g_nException = 1;
            dispsize = 2;
          }
        }
      }
      else if(nCommand == 0x05)  //32 位寻址模式(无寄存器基址)
      {
        dispsize = 4;
        if(size < 6)  g_nException = 1;  //6=1opcode+1modR/M+4disp
      }
      else if((nCommand & 0x07) == 0x04)  //32 位寻址模式(有 SIB 字段)
      {
        nSIB = src[2];
        hassib = 1;

        if(nCommand == 0x04 && (nSIB & 0x07) == 0x05)  //立即数寻址
        {
          dispsize = 4;
          if(size < 7)  g_nException = 1;
        }
        else  //非立即数寻址
        {
          if((nCommand & 0xC0) == 0x40)  //mod=01
          {
            dispsize = 1;
```


# x86 机器码识别程序

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-489.htm`](https://www.pediy.com/kssd/pediy09/pediy09-489.htm)

> 【标题】 x86 机器码识别程序  
> 【作者】 linxer  
> 【环境】 xp + vc6  
> 【声明】 本人学疏才浅，失误之处敬请诸位大侠赐教！

题外话：以前曾做过虚拟机脱壳方面的研究，不过后来由于某些原因，没有再研究下去。以前也想继续搞下去，苦于没有时间。近日，在不可抗力作用下与女友分手......逼与无奈，准备重新研究这个东西，这次准备做一个功能更加强大的虚拟机，使之具有解一些加密壳能力。本人对壳的认识十分肤浅（会的一点都忘了），深知要完成这个任务会遇到很多技术问题，希望能得到各位高人的指点，故借本文在此 show 下自己 qq:3568599，不出意外，本人也会在此共享一些研究成果；也以此文献给昔日女友，祝幸福！

其实，以下代码只是本人整理的，它来源于 `ollydbg` 反汇编器源码（[`www.pediy.com/sourcecode/disa...sasm.zip 可下`](http://www.pediy.com/sourcecode/disassemblers/disasm/disasm.zip 可下)），以下代码看起来比较乱，有意往下看的同志最好先揉下眼睛哈:)

```c
#define u8 unsigned char
#define s8 char

#define u16 unsigned short int
#define s16 short int

#define u32 unsigned int
#define s32 int
```

## 每条汇编指令详细信息

```c
typedef struct tagDisasmCodeInfo
{
  u16    nPrefixes;  //前缀码
  u16    nSize;      //指令长度(含前缀码)
  u8 *   pOpcodeAdd; //operand 或 modR/M 字节前一个字节地址
  void * pCmdDetail; //指令详细描述
} DisasmCodeInfo, *PDisasmCodeInfo;
```

```c
u8 * g_disasmbuf = NULL;  //待反汇编的 pe 文件缓冲区指针
u32  g_disasmLength = 0;  //待反汇编的 pe 文件长度

DisasmCodeInfo g_disamInfo; //保存当前正在反汇编的代码的详细信息
t_cmddata *    g_pcmddata;  //保存当前被反汇编指令的 opcode 信息

u32 g_nException;     //当前指针在反汇编中是否出现异常
u32 g_nDatasize;      //operand 数据大小
u32 g_nAddrsize;      //operand 地址大小
u32 g_nRepPrefix;     //标识 rep 前缀
u32 g_nPrefixLen;     //opcode 前缀码长度
```

```c
void init_disam_opcode(PDisasmCodeInfo p)
{
  g_nException = 0;  //清除异常标志
  g_nDatasize = 4;   //32 位系统默认操作数大小
  g_nAddrsize = 4;   //32 位系统默认地址大小
  g_nRepPrefix = 0;  //默认是 0
  g_nPrefixLen = 0;

  memset((void *)p, 0, sizeof(DisasmCodeInfo));
}
```

## 初始化反汇编器

```c
void init_disam_machine(u8 * pPEBuf, u32 nPELen)
{
  g_disasmbuf = pPEBuf;
  g_disasmLength = nPELen;
}
```

```c
void default_disam_machine()
{
  g_disasmbuf = NULL;
  g_disasmLength = 0;

  memset((void *)&g_disamInfo, 0, sizeof(DisasmCodeInfo));
  g_pcmddata = NULL;
}
```

## 前缀码识别函数

如果前缀码不冲突返回 0，否则返回-1

```c
u32 prefix_recognise(u8 ** pSrc, u32 * pSize)
{
  u32 nIsPrefix;    //当前字节是否还是 opcode 的前缀码
  u32 nSegPrefix = 0; //当前 opcode 含有段跨越前缀的个数
  u32 nRepeated = 0;  //是否前缀重复

  for(; *pSize > 0; )
  {
    nIsPrefix = 1;  //假定当前字节还是前缀码

    switch(**pSrc)
    {
      /*本人愚见，以下文字请用批判眼光看待：由于现在 win32 系统内存采用 flat 模式，win 把描述表中的段基地址都填为 0，段的界限都为 4GB,因此段寄存器的作用不是很大了，在 OD 中可以看到这些段寄存器的值,gs 是 0,cs 是 0x1b,fs 是 0x3b 与 SEH 有关，其它的均是 0x23，除 fs 外，其它的可以看成等同，gs 未用，所以具体识别出什么段前缀码已经没有什么意义了，不过 fs 还是要另眼看待的*/
    case 0x64: //fs 段跨越前缀
      g_disamInfo.nPrefixes |= FSPrefix; //FSPrefi 是 fs 前缀标志
    case 0x26: //es 段跨越前缀
    case 0x2E: //cs 段跨越前缀
    case 0x36: //ss 段跨越前缀
    case 0x3E: //ds 段跨越前缀
    case 0x65: //gs 段跨越前缀
      nSegPrefix++;
      break;
    case 0x66:
      if(g_nDatasize == 4)
      {
        g_nDatasize = 2;
        g_disamInfo.nPrefixes |= DataPrefix; //DataPrefix 是操作数大小前缀标志
      }
      else
      {
        nRepeated = 1;
      }
      break;
    case 0x67:
      if(g_nAddrsize == 4)
      {
        g_nAddrsize = 2;
        g_disamInfo.nPrefixes |= AddPrefix; //AddPrefix 是地址大小前缀标志
      }
      else
      {
        nRepeated = 1;
      }
      break;
    case 0xF0: //lock 前缀只对 smp 系统有用，这里忽略
      break;
    case 0xF2:
      if(g_nRepPrefix == 0)
      {
        g_nRepPrefix = 0xF2;
        g_disamInfo.nPrefixes |= REPNEPrefix; //REPNEPrefix 是 repne 前缀标志
      }
      else
      {
        nRepeated = 1;
      }
      break;
    case 0xF3:
      if(g_nRepPrefix == 0)
      {
        g_nRepPrefix = 0xF3;
        g_disamInfo.nPrefixes |= REPEPrefix; //REPEPrefix 是 rep 前缀标志
      }
      else
      {
        nRepeated = 1;
      }
      break;
    default:
      nIsPrefix = 0;
      break;
    }

    if(nRepeated == 1 || nSegPrefix > 1)  //说明前缀重复冲突
    {
      g_nException = 1; //异常
      return -1;
    }

    if(nIsPrefix == 1)
    {
      g_nPrefixLen++;   //前缀长度计数加 1
    }
    else
    {
      //识别前缀码，直到非前缀码
      break;
    }

    (*pSrc)++;
    (*pSize)--;
  }

  return 0;
}
```

```c
u32 Get3dnowsuffix(u8 * cmd, u32 size)
{
  int c,sib;
  unsigned long offset;
  if(size < 3) return -1;               //因为 3DNow 指令长度至少是 3，小于 3 则出错
  offset = 3;
  c = cmd[2] & 0xC7;                    //去掉 ModR/M 域的 opcode
  // Register in ModM - general-purpose, MMX or 3DNow!
  if ((c & 0xC0)==0xC0)
    ;
  // 16-bit addressing mode, SIB byte is never used here.
  else if (g_nAddrsize == 2) {
    if (c==0x06)                       // Special case of immediate address
      offset+=2;
    else if ((c & 0xC0)==0x40)         // 8-bit signed displacement
      offset++;
    else if ((c & 0xC0)==0x80)         // 16-bit unsigned displacement
      offset+=2;
    ; }
  // Immediate 32-bit address.
  else if (c==0x05)                    // Special case of immediate address
    offset+=4;
  // 32-bit address with SIB byte.
  else if ((c & 0x07)==0x04) {         // SIB addresation
    if (size<4) return -1;             // Suffix outside the memory block
    sib=cmd[3]; offset++;
    if (c==0x04 && (sib & 0x07)==0x05)
      offset+=4;                       // Immediate address without base
    else if ((c & 0xC0)==0x40)         // 8-bit displacement
      offset+=1;
    else if ((c & 0xC0)==0x80)         // 32-bit dislacement
      offset+=4;
    ; }
  // 32-bit address without SIB byte
  else if ((c & 0xC0)==0x40)
    offset+=1;
  else if ((c & 0xC0)==0x80)
    offset+=4;
  if (offset>=size) return -1;         // Suffix outside the memory block
  return cmd[offset];
}
```

## x86 机器码识别函数

`src` 是汇编起始位置  
`srcsize` 可以反汇编的长度  
返回值是本次反汇编字节数

```c
u32 disasm_one_opcode(u8 *src, u32 srcsize)
{
  u32 nMemoryOnly = 1;       //operand 只能在内存中标识
  u32 nCommand;
  u32 nSIB;

  u32 nMin;
  u32 size = srcsize;

  u32 is3dnow = 0;               //是否反汇编了 AMD 3DNow 指令

  u32 operand;                   //索引操作数

  u32 hasrm = 0;                 //是否出现 ModR/M
  u32 hassib = 0;                //是否出现 SIB
  u32 dispsize = 0;              //displacement 字段大小
  u32 immsize = 0;               //immedate 字段大小

  u32 i, j, arg;
  u32 code;
  const t_cmddata *pd;

  //opcode 前缀码扫描
  if(0 != prefix_recognise(&src, &size)) //出现冲突前缀码
  {
    return 1; // 跳过一个字节重新反汇编
  }

  //opcode 最长为 3 个字节
  code = 0;
  nMin = min(size, 3);
  if(0 != g_nRepPrefix)  //如有 rep 前缀，也算作 opcode
  {
    code = g_nRepPrefix;
    memcpy((char *)&code + 1, src, nMin);
  }
  else
  {
    memcpy((void *)&code, src, nMin);
  }

  //查找当前 opcode 对应的指令信息(这里不反汇编 vxd 指令)
  for(pd = cmddata; pd->mask != 0; pd++)
  {
    if(((code ^ pd->code) & pd->mask) == 0) break;
  }

  //如果查到的是 AMD 3DNow 指令，则还要继续查找，因为 3DNow 指令系列是通过 ModR/M 来区分的
```



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



```c
if(size < 4)  g_nException = 1;
}
else if((nCommand & 0xC0) == 0x80)  //mod=10
{
  dispsize = 4;
  if(size < 7)  g_nException = 1;
}
}
}
else  //32 位寻址模式(无 SIB 字段)
{
  if((nCommand & 0xC0) == 0x40)  //mod=01
  {
    dispsize = 1;
    if(size < 3)  g_nException = 1;
  }
  else if((nCommand & 0xC0) == 0x80)  //mod=10
  {
    dispsize = 4;
    if(size < 6)  g_nException = 1;
  }
}
break;      
case IMM:                      // Immediate data (8 or 16/32)
case IMU:                      // Immediate unsigned data (8 or 16/32)
  if((pd->bits & SS) != 0 && (*src & 0x02) != 0)
  {
    immsize += 1;  //因为一个指令中可能会有两个立即数，故这里用累加，不是赋值
    if(size < 1 + hasrm + hassib + dispsize + immsize) g_nException = 1;//这个 1 是 opcode
  }
  else
  {
    immsize += g_nDatasize;
    if(size < 1 + hasrm + hassib + dispsize + immsize) g_nException = 1;
  }
  break;
case IMX:                      // Immediate sign-extendable byte
case IMS:                      // Immediate byte (for shifts)
case IM1:                      // Immediate byte
  immsize += 1;
  if(size < 1 + hasrm + hassib + dispsize + immsize) g_nException = 1;
  break;
case IM2:                      // Immediate word (ENTER/RET)
  immsize += 2;
  if(size < 1 + hasrm + hassib + dispsize + immsize) g_nException = 1;
  break;
case VXD:                      // VxD service (32-bit only)
  immsize += 4;
  if(size < 1 + hasrm + hassib + dispsize + immsize) g_nException = 1;
  break;
case IMA:                      // Immediate absolute near data address
  if(size < 1 + g_nAddrsize)  g_nException = 1;
  dispsize = g_nAddrsize;
  break;
case JOB:                      // Immediate byte offset (for jumps)
  if(size < 1 + 1)  g_nException = 1;
  dispsize = 1;
  break;
case JOW:                      // Immediate full offset (for jumps)
  if(size < g_nDatasize + 1)  g_nException = 1;
  dispsize = g_nDatasize;
  break;    
case JMF:                      // Immediate absolute far jump/call addr
  if(size < 1 + g_nAddrsize + 2)  g_nException = 1;
  dispsize = g_nAddrsize;
  immsize = 2;                // Non-trivial but allowed interpretation
  break;
case CRX:                      // Control register CRx
case DRX:                      // Debug register DRx
  if((src[1] & 0xC0) != 0xC0)  g_nException = 1;
  hasrm = 1;
  break;
default:
  g_nException = 1;
  break;
}
}
  
if(1 == is3dnow)
{
  if(immsize!=0) g_nException = 1;
  else immsize=1;
}
  
if(g_nException != 0)  //出错
{
  size--;
}
else
{
  size -= 1 + hasrm + hassib + dispsize + immsize;
}
  
return (srcsize - size);               // Returns number of recognized bytes
}
```

大致用法如下：

```c
init_disam_machine(pExePEBuf, imageSize);
init_disam_opcode(&g_disamInfo);
nInstructionLen = disasm_one_opcode(char *buf, u32 nBufLen);
```

4dd466d0e1e0cc123afbe504da2462ca

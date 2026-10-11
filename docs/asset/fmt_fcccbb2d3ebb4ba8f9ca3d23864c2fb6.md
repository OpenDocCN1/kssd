
```
case MFX:        // Memory in ModRM byte (ext. FPU state)
  hasrm = 1;

  // 获取操作数的偏移地址
  displacement = op->offset;

  // 地址中是否存在 displacement 值
  anydisp = op->anyoffset;

  // 地址没有包含 base 与 index 部分
  if (op->base < 0 && op->index < 0)
  {
    // 则直接数作为地址长度只有 32 位的情况了
    dispsize = 4;    // Special case of immediate address
    // 如果段前缀使用非默认段（即 DS）
    if (op->segment != SEG_UNDEF && op->segment != SEG_DS)
      segment = op->segment;

    // 0x05：101，表示地址为 [disp32]
    tcode[i + 1] |= 0x05;
    tmask[i + 1] |= 0xC7;
  }

  // 如果地址没有包含 index 部分，并且 base 也不是 esp 寄存器，则不需要 SIB 字节来辅助寻址
  // 格式：[base + disp]
  else if (op->index < 0 && op->base != REG_ESP)
  {
    // 0xC0：11000000b
    tmask[i + 1] |= 0xC0;    // SIB byte unnecessary
    // 如果未提供 offset（即是 disp），除非非 EBP 寄存器，否则是有问题的
    if (op->offset == 0 && op->anyoffset == 0 && op->base != REG_EBP)
      ;            // [EBP] always requires offset
    else if ((constsize & 1) != 0 && ((op->offset >= -128 && op->offset < 128)
      || op->anyoffset != 0) )
    {
      // 0x40：1000000b，表示 disp 为 8 位数
      tcode[i + 1] |= 0x40;  // Disp8
      dispsize = 1;
    }
    else
    {
      // 否则，说明是 32 位 disp 数
      tcode[i + 1] |= 0x80;  // Disp32
      dispsize = 4;
    };

    // 明确指定基址寄存器
    if (op->base < 8)
    {
      // 如果指明操作数的段前缀与该基址的默认段不一致，则使用操作数指明的段作为前缀
      if (op->segment != SEG_UNDEF && op->segment != addr32[op->base].defseg)
        segment = op->segment;

      // 设置操作数的基址
      tcode[i + 1] |= (char)op->base;    // Note that case [ESP] has base<0.
      tmask[i + 1] |= 0x07;
    }

    // 否则，直接使用操作数指明的段寄存器
    else
      segment = op->segment;
  }

  // 地址包含 index 寄存器部分，则需要 SIB 寻址
  else    // SIB byte necessary
  {
    // 表明指令包含 SIB 字节
    hassib = 1;

    // 如果基址使用 EBP 寄存器，那么需要提供偏移量，所以，如果操作数未提供偏移量
    // 而且又提供了 index 寄存器，并且其 scale 为 1，为了优先，可以将 index 寄存器作
    // 为基址，而 EBP 作为 index 寄存器来使用
    if (op->base == REG_EBP &&      // EBP as base requires offset, optimize
      op->index >= 0 && op->scale == 1 && op->offset == 0 && op->anyoffset == 0)
    {
      op->base = op->index;
      op->index = REG_EBP;
    };

    // ESP 寄存器不能作为 index 使用
    if (op->index == REG_ESP &&      // ESP cannot be an index, reorder
      op->scale <= 1)
    {
      op->index = op->base;
      op->base = REG_ESP;
      op->scale = 1;
    };

    // 如果操作数不存在基址，而 index 寄存器存在，并且 scale 为 2，并且其偏移量为 8 位，
    // 例如[index * 2 + XX]，可以优化为：[base + index +XX]
    if (op->base < 0 &&          // No base means 4-byte offset, optimize
      op->index >= 0 && op->scale == 2 &&
      op->offset >= -128 && op->offset < 128 && op->anyoffset == 0)
    {
      op->base = op->index;
      op->scale = 1;
    };

    // 至此，如果 index 还是 ESP，则出错
    if (op->index == REG_ESP)    // Reordering was unsuccessfull
    {
      strcpy(errtext, "Invalid indexing mode");
      goto error;
    };

    // 确实未指定基址
    if (op->base < 0)
    {
      tcode[i + 1] |= 0x04;
      dispsize = 4;
    }

    // 地址未包含 displacement 值
    else if (op->offset == 0 && op->anyoffset == 0 && op->base != REG_EBP)
      tcode[i + 1] |= 0x04;    // No displacement

    // 包含 8 位的 displacement 值
    else if ((constsize & 1) != 0 && ((op->offset >= -128 && op->offset < 128)
      || op->anyoffset != 0) )
    {
      tcode[i + 1] |= 0x44;    // Disp8
      dispsize = 1;
    }

    // 包含 32 位的 displacement 值
    else
    {
      tcode[i + 1] |= 0x84;    // Disp32
      dispsize = 4;
    }

    // 至此，ModRM 字节 OK 了，现在处理 SIB 字节
    tmask[i + 1] |= 0xC7;      // ModRM completed, proceed with SIB

    // 处理完指令的 ModRM，则继续处理 SIB 字节
    if (op->scale == 2)
      tcode[i + 2] |= 0x40;
    else if (op->scale == 4)
      tcode[i + 2] |= 0x80;
    else if (op->scale == 8)
      tcode[i + 2] |= 0xC0;

    tmask[i + 2] |= 0xC0;

    // 明确指定 index 寄存器
    if (op->index < 8)
    {
      // 未指定 index
      if (op->index < 0)
        op->index = 0x04;

      // 将 index 寄存器编码进 SIB 字节中
      tcode[i + 2] |= (char)(op->index << 3);
      tmask[i + 2] |= 0x38;
    };

    // 明确指定 base 寄存器
    if (op->base < 8)
    {
      if (op->base < 0)
        op->base = 0x05;

      if (op->segment != SEG_UNDEF && op->segment != addr32[op->base].defseg)
        segment = op->segment;

      // 将 base 寄存器编码进 SIB 字节中
      tcode[i + 2] |= (char)op->base;
      tmask[i + 2] |= 0x07;
    }
    else
      segment = op->segment;
  };

  break;

case IMM:        // Immediate data (8 or 16/32)
case IMU:        // Immediate unsigned data (8 or 16/32)
case VXD:        // VxD service (32-bit only)
  if (datasize == 0 && pd->arg2 == NNN && (pd->bits == SS || pd->bits == WS))
    datasize = 4;

  if (datasize == 0)
  {
    strcpy(errtext, "Please specify operand size");
    goto error;
  };

  immediate = op->offset;
  anyimm = op->anyoffset;

  if (pd->bits == SS || pd->bits == WS)
  {
    if (datasize > 1 && (constsize & 2) != 0 && ((immediate >= -128 && immediate < 128)
      || op->anyoffset != 0))
    {
      immsize = 1;
      tcode[i] |= 0x02;
    }
    else
      immsize = datasize;

    tmask[i] |= 0x02;
  }
  else
    immsize = datasize;
  break;

case IMX:          // Immediate sign-extendable byte
case IMS:          // Immediate byte (for shifts)
case IM1:          // Immediate byte
  if (immsize == 2)      // To accomodate ENTER instruction
    immediate = (immediate & 0xFFFF) | (op->offset << 16);
  else
    immediate = op->offset;

  anyimm |= op->anyoffset;
  immsize++;
  break;

case IM2:          // Immediate word (ENTER/RET)
  immediate = op->offset;
  anyimm = op->anyoffset;
  immsize = 2;
  break;

case IMA:          // Immediate absolute near data address
  if (op->segment != SEG_UNDEF && op->segment != SEG_DS)
    segment = op->segment;

  displacement = op->offset;
  anydisp = op->anyoffset;
  dispsize = 4;
  break;

case JOB:          // Immediate byte offset (for jumps)
  jmpoffset = op->offset;
  anyjmp = op->anyoffset;
  jmpsize = 1;
  break;

case JOW:          // Immediate full offset (for jumps)
  jmpoffset = op->offset;
  anyjmp = op->anyoffset;
  jmpsize = 4;
  break;

case JMF:          // Immediate absolute far jump/call addr
  displacement = op->offset;
  anydisp = op->anyoffset;
  dispsize = 4;

  immediate = op->segment;
  anyimm = op->anyoffset;
  immsize = 2;
  break;

case SGM:          // Segment register in ModRM byte
  hasrm = 1;

  if (op->index < 6)
  {
    tcode[i + 1] |= (char)(op->index << 3);
```

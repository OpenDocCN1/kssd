
```c
const char *mmxname[9] = {
  "MM0","MM1","MM2","MM3","MM4","MM5","MM6","MM7","MMX"
};

// 探制寄存器

const char *crname[9] = {
  "CR0","CR1","CR2","CR3","CR4","CR5","CR6","CR7","CRX"
};

const char *drname[9] = {
  "DR0","DR1","DR2","DR3","DR4","DR5","DR6","DR7","DRX"
};

// List of available processor commands with decoding, types of parameters and
// other useful information. Last element has field mask=0. If mnemonic 助记符 begins
// with ampersand ('&'), its mnemonic decodes differently depending on operand
// size (16 or 32 bits). If mnemonic begins with dollar ('$'), this mnemonic
// depends on address size. Semicolon (':') separates 16-bit form from 32-bit,
// asterisk ('*') will be substituted by either W (16), D (32) or none (16/32)
// character. If command is of type C_MMX or C_NOW, or if type contains C_EXPL
// (=0x01), Disassembler must specify explicit size of memory operand.

/**
 * 存储处理器有效的（机器）指令信息，包括操作数和其他有用信息
 */

const t_cmddata cmddata[] = {
  // mask，code，len，bits，arg1，arg2，arg3，type，name
  // 指令机器码的获取：code & mask
  { 0x0000FF, 0x000090, 1,00,  NNN,NNN,NNN, C_CMD+0,        "NOP" },
  { 0x0000FE, 0x00008A, 1,WW,  REG,MRG,NNN, C_CMD+0,        "MOV" },
  { 0x0000F8, 0x000050, 1,00,  RCM,NNN,NNN, C_PSH+0,        "PUSH" },
  { 0x0000FE, 0x000088, 1,WW,  MRG,REG,NNN, C_CMD+0,        "MOV" },
  { 0x0000FF, 0x0000E8, 1,00,  JOW,NNN,NNN, C_CAL+0,        "CALL" },
  { 0x0000FD, 0x000068, 1,SS,  IMM,NNN,NNN, C_PSH+0,        "PUSH" },
    { 0x0000FF, 0x000060, 1,00,  NNN,NNN,NNN, C_CMD+C_RARE+0, "&PUSHA*" },
  { 0x0038FF, 0x00008F, 1,00,  MRG,NNN,NNN, C_POP+1,        "POP" },
  { 0x0000FF, 0x000061, 1,00,  NNN,NNN,NNN, C_CMD+C_RARE+0, "&POPA*" },
};

// ------------------------------------- disasm.cpp ------------------------------------------
// 用于反编译机器码
// 此文件分析不够详细，但如能看懂 assemble.cpp ，则此文件相对简单了很多
// Free Disassembler and Assembler -- Disassembler
//
// Copyright (C) 2001 Oleh Yuschuk
//
//  This program is free software; you can redistribute it and/or modify
//  it under the terms of the GNU General Public License as published by
//  the Free Software Foundation; either version 2 of the License, or
//  (at your option) any later version.
//
//  This program is distributed in the hope that it will be useful,
//  but WITHOUT ANY WARRANTY; without even the implied warranty of
//  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
//  GNU General Public License for more details.
//
//  You should have received a copy of the GNU General Public License
//  along with this program; if not, write to the Free Software
//  Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA

#define STRICT

#include "stdafx.h"

#include <windows.h>

#include <stdio.h>

#include <string.h>

#include <ctype.h>

//#include <dir.h>

#include <math.h>

#include <float.h>

#pragma hdrstop

#pragma   warning(disable:4996)//关闭旧函数声明警告

#include "disasm.h"

//bughoho. 文件扩展名改为 cpp.把外部扩展变量在 cpp 中添加声明

int       ideal;                // Force IDEAL decoding mode

int       lowercase;            // Force lowercase display

int       tabarguments;         // Tab between mnemonic and arguments

int       extraspace;           // Extra space between arguments

int       putdefseg;            // Display default segments in listing

int       showmemsize;          // Always show memory size

int       shownear;             // Show NEAR modifiers

int       shortstringcmds;      // Use short form of string commands

// 某些指令助记符中含有'W'或'D'以指明其操作数长度，这种指令的机器码一般依赖于操作数的长度
// 而当这些指令没有指明'W'或'D'时，则根据 sizesens 决定其操作数长度：
//    0：操作数为 16 位（字）
//    1：操作数为 32 位（双字）

int       sizesens;             // How to decode size-sensitive mnemonics

int       symbolic;             // Show symbolic addresses in disasm

int       farcalls;             // Accept far calls, returns & addresses

int       decodevxd;            // Decode VxD calls (Win95/98)

int       privileged;           // Accept privileged commands

int       iocommand;            // Accept I/O commands

int       badshift;             // Accept shift out of range 1..31

int       extraprefix;          // Accept superfluous prefixes

int       lockedbus;            // Accept LOCK prefixes

int       stackalign;           // Accept unaligned stack operations

int       iswindowsnt;          // When checking for dangers, assume NT

////////////////////////////////////////////////////////////////////////////////
//////////////////////////// DISASSEMBLER FUNCTIONS ////////////////////////////

// Work variables of disassembler

static ulong    datasize;         // Size of data (1,2,4 bytes)

// 地址大小

static ulong    addrsize;         // Size of address (2 or 4 bytes)

static int      segprefix;        // Segment override prefix or SEG_UNDEF

static int      hasrm;            // Command has ModR/M byte

// 指令是否包含 SIB 字节

static int      hassib;           // Command has SIB byte

// displacement 的长度，1 表示 1 字节，2 表示 2 字节

static int      dispsize;         // Size of displacement (if any)

static int      immsize;          // Size of immediate data (if any)

static int      softerror;        // Noncritical disassembler error

// 代码已完成 dump 的长度

static int      ndump;            // Current length of command dump

// 登记当前已反汇编的指令长度

static int      nresult;          // Current length of disassembly

static int      addcomment;       // Comment value of operand

// Copy of input parameters of function Disasm()

static char     *cmd;             // Pointer to binary data

static char     *pfixup;         // Pointer to possible fixups or NULL

static ulong    size;             // Remaining size of the command buffer

static t_disasm *da;              // Pointer to disassembly results

static int      mode;             // Disassembly mode (DISASM_xxx)

// Disassemble name of 1, 2 or 4-byte general-purpose integer register and, if
// requested and available, dump its contents. Parameter type changes decoding
// of contents for some operand types.

// static void DecodeRG(int index,int datasize,int type)
// bughoho void 改为 int 返回解码得到的寄存器索引
//

static int DecodeRG(int index, int datasize, int type)
{
  int sizeindex;
  char name[9];
  // 无需解码
  if (mode < DISASM_DATA)
    return -1;    // No need to decode

  sprintf_s(da->vm_name, "%s_REG%02d", da->vm_name, datasize * 8);

  // 操作数类型为寄存器

  da->optype[stoperand] = Reg;

  // 0x07: 00000111b

  index &= 0x07;    //得到寄存器索引

  if (datasize == 1)
    sizeindex = 0;
  else if (datasize == 2)
    sizeindex = 1;
  else if (datasize == 4)
    sizeindex = 2;
  else
  {
    da->error = DAE_INTERN;
    return -1;
  }

  // 反汇编

  if (mode >= DISASM_FILE)
  {
    // name 中将存储寄存器名称

    strcpy(name, regname[sizeindex][index]);

    if (lowercase)
      strlwr(name);

    // 不是一个假操作数

    if (type < PSEUDOOP)    // Not a pseudooperand
      // 将操作数组合至 result 结果中

      nresult += sprintf(da->result + nresult,"%s",name);

    // bughoho

    if (sizeindex == 0 && (index > 3 && index < 7))
    {
      // 高 8 字节寄存器：ah bh ch dh

      da->highbit[stoperand] = true;
    }
  }

  // 存储寄存器编号

  da->reg[stoperand] = index;

  return index;
}

// Disassemble name of 80-bit floating-point register and, if available, dump
// its contents.
// 对 index 编号的 FPU 寄存器 ST[XX]进行反汇编码

static void DecodeST(int index, int pseudoop)
{
  int i;
  char s[32];
  // 不需反汇编
  if (mode < DISASM_FILE)
    return;    // No need to decode

  // 0x07: 00000111b
```

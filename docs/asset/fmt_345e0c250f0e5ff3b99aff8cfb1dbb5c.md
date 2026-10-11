
```cpp
  sprintf_s(asmtext, len, "%smov eax,dword ptr [eax]\n", asmtext);

}

// 将 call 的地址压入堆栈中

sprintf_s(asmtext, len, "%smov [ebp],eax\n", asmtext);

// 将 ebp(即真正 esp)保存到 VMContext 上下文的 esp 中

sprintf_s(asmtext, len, "%smov [edi+%02x],ebp\n", asmtext, GetRegisterOffset(RT_Esp));

// 压入保存在 VMContext 的寄存器值

for (int i = 0; i < 9; i++)

{

  sprintf_s(asmtext, len, "%spush [edi+%02X]\n", asmtext, GetRegisterOffset(s_reg[i]));

}

// 再将各寄存器值弹出，恢复到各寄存器中，以恢复环境

sprintf_s(asmtext, len, "%spop eax\n", asmtext);

sprintf_s(asmtext, len, "%spop ebx\n", asmtext);

sprintf_s(asmtext, len, "%spop ecx\n", asmtext);

sprintf_s(asmtext, len, "%spop edx\n", asmtext);

sprintf_s(asmtext, len, "%spop esi\n", asmtext);

sprintf_s(asmtext, len, "%spop edi\n", asmtext);

sprintf_s(asmtext, len, "%spop ebp\n", asmtext);

sprintf_s(asmtext, len, "%spopfd\n", asmtext);

// 恢复到真实的堆栈

sprintf_s(asmtext, len, "%spop esp\n", asmtext);

// 返回，直接回到 call 的地址执行

sprintf_s(asmtext, len, "%sretn\n", asmtext);

return FALSE;

}

/**

 * 为指令 saveesp 生成相应的汇编代码，用于保存堆栈

 *

 * table: 存放 saveesp 指令的信息，根据这些信息可以为相应虚拟机生成汇编代码

 * asmtext: 保存生成的汇编代码

 * len：asmtext 缓存的长度

 *

 * 返回值：只返回 FALSE

 */

BOOL CInterpretHandler::InterpretSaveEsp(VMTable * table, char * asmtext, int len)  // *

{

  // 将 ebp(即真正 esp)保存到 VMContext 上下文的 esp 中

  sprintf_s(asmtext, len, "%smov [edi+%02x],ebp\n", asmtext, GetRegisterOffset(RT_Esp));

  return FALSE;

}

/**

 * 为指令 restoreesp 生成相应的汇编代码，用于恢复堆栈

 *

 * table: 存放 restoreesp 指令的信息，根据这些信息可以为相应虚拟机生成汇编代码

 * asmtext: 保存生成的汇编代码

 * len：asmtext 缓存的长度

 *

 * 返回值：只返回 FALSE

 */

BOOL CInterpretHandler::InterpretRestoreEsp(VMTable * table, char * asmtext, int len)  // *

{

  // 将 VMContext 上下文中 esp 恢复到 ebp 真实堆栈指针中

  sprintf_s(asmtext, len, "%smov ebp,[edi+%02x]\n", asmtext, GetRegisterOffset(RT_Esp));

  return FALSE;

}

// ------------------------------------- vmserv.cpp ---------------------------------------------

// 此文件定义了如《汇编指令-虚拟机指令》对应表 vmtable 等信息

// 这里，只摘取了其中一部分

#include "stdafx.h"

#include "VCommand.h"

#include "asm\disasm.h"

#include "vmdisasm.h"

/**

 * 寄存器名

 */

const char * vregname[3][14] =

{

  { "AL", "CL", "DL", "BL", "AH", "CH", "DH", "BH", "CS","DS","ES","FS","GS","SS"  },

  { "AX", "CX", "DX", "BX", "SP", "BP", "SI", "DI", "CS","DS","ES","FS","GS","SS" },

  { "EAX","ECX","EDX","EBX","ESP","EBP","ESI","EDI","CS","DS","ES","FS","GS","SS" }

};

/**

 * 可用于存放虚拟指令操作数的寄存器

 */

const char ArgReg[3][4] = {"EAX","ECX","EDX"};

/**

 * 虚拟机有两部分的指令：

 * 1、是需要手工编写生成的指令

 * 2、是可以由程序动态分析生成的指令

 */

/**

 * 需要手工编写生成的核心指令

 */

VM_DName vm_dname[COREHANDLERLEN] =

{

  {"VStartVM", VStartVM},    // 启动函数

  {"DCheckESP", DCheckESP},  // 检查 ESP 是否重复

  {"DPushReg32", DPushReg32},  // dpush eax

  {"DPushImm32", DPushImm32},  // dpush 1234

  {"DPushMem32", DPushMem32},  // dpush 1234

  {"DPopReg32", DPopReg32},  // dpop  ebx

  {"DFree", DFree},      // daddesp 4

  {"", NULL},

  {"", NULL},

};

/**

 * 数组包含了虚拟机指令与相应的普通汇编指令的信息，根据这些信息，虚拟机

 * 可以为虚拟机指令生成相应的汇编代码以代替原来对应的普通汇编指令

 */

VMTable vmtable[VMTABLEMAXLEN] =

{

  // 信息格式如下：

  // 虚拟机指令名、相对的汇编指令、操作数个数、段前缀、操作数类型、操作数位数、执行指令时需要使用的寄存器、执行指令后需要保存的寄存器、第 2 个寄存器是否恢复(一般为 0 不恢复)

  {"VBEGIN","vBegin",0, SEG_UNDEF, NONETYPE, NONETYPE,0,0, NONE,NONE,NONE,NONE,    NONE,NONE,NONE,NONE },

  {"VTOREAL","vtoReal",1, SEG_UNDEF, IMMTYPE, NONETYPE,32,0, NONE,NONE,NONE,NONE,    NONE,NONE,NONE,NONE },

  {"VRETN","RETN",0, SEG_UNDEF, NONETYPE, NONETYPE,0,0, NONE,NONE,NONE,NONE,    NONE,NONE,NONE,NONE },

  {"VRETNIMM32","RETN",1, SEG_UNDEF, IMMTYPE, NONETYPE,32,0, NONE,NONE,NONE,NONE,    NONE,NONE,NONE,NONE },

  // 保存堆栈

  {"VSAVEESP","SAVEESP",0, SEG_UNDEF, IMMTYPE, NONETYPE,32,0, NONE,NONE,NONE,NONE,    NONE,NONE,NONE,NONE },

  // 恢复堆栈

  {"VRESTOREESP","RESTOREESP",0, SEG_UNDEF, IMMTYPE, NONETYPE,32,0, NONE,NONE,NONE,NONE,    NONE,NONE,NONE,NONE },

  // 堆栈操作指令

  {"VPUSH_IMM32","PUSH",1, SEG_UNDEF, IMMTYPE, NONETYPE,32,0, NONE,NONE,NONE,NONE,    NONE,NONE,NONE,NONE },

  {"VPUSH_REG16","PUSH",1, SEG_UNDEF, REGTYPE, NONETYPE,16,0, NONE,NONE,NONE,NONE,    NONE,NONE,NONE,NONE },

};

// ----------------------------------- InterpretHandler.h ---------------------------------------

#pragma once

#include "vmdisasm.h"

class CInterpretHandler

{

public:

  CInterpretHandler(void);

public:

  ~CInterpretHandler(void);

private:

  // 寄存器数组索引

  // 前 15 个元素是 15 个不超 15 的随机数

  int     m_RegisterIdx[STACKLEN];

public:

  // 初始化

  BOOL Init();

public:

  // 获得寄存器的偏移

  int GetRegisterOffset(int RegType);

  // 获得段前缀

  char* GetSegStr(int Segment);

  // 根据结构声称 ASM 字符串

  BOOL  InterpretASMStr(VMTable* table,char* asmtext,int len);

private:

  //设置参数

  void SetArg(VMTable* table,char* asmtext,int len);

  //恢复参数

  void RestoreArg(VMTable* table,char* asmtext,int len);

  //恢复标志

  void RestoreFlag(char* asmtext,int len);

  //保存标志

  void SaveFlag(char* asmtext,int len);

private:

  // 首先执行的指令

  BOOL  InterpretvBegin(VMTable* table,char* asmtext,int len);

  // 跳转到真实指令

  BOOL  InterpretvtoReal(VMTable* table,char* asmtext,int len);

  // **************堆栈类************************* //

  // 解释 push

  BOOL  InterpretPush(VMTable* table,char* asmtext,int len);

  // 解释 pop

  BOOL  InterpretPop(VMTable* table,char* asmtext,int len);

  // 解释 pushfd

  BOOL  InterpretPushfd(VMTable* table,char* asmtext,int len);

  // 解释 popfd

  BOOL  InterpretPopfd(VMTable* table,char* asmtext,int len);

  // 解释 pushad

  BOOL  InterpretPushad(VMTable* table,char* asmtext,int len);

  // 解释 popad

  BOOL  InterpretPopad(VMTable* table,char* asmtext,int len);

  // 解释 enter

  BOOL  InterpretEnter(VMTable* table,char* asmtext,int len);

  // 解释 leave

  BOOL  InterpretLeave(VMTable* table,char* asmtext,int len);

  // **************流程类************************* //

  // 解释 jmp

  BOOL  InterpretJMP(VMTable* table,char* asmtext,int len);

  // 解释 jcxz\jecxz

  BOOL  InterpretJCXZ(VMTable* table,char* asmtext,int len);

  // 解释 jcc

  BOOL  InterpretJCC(VMTable* table,char* asmtext,int len);

  // 解释 loope

  BOOL InterpretLoope(VMTable* table,char* asmtext,int len);

  // 解释返回

  BOOL InterpretRetn(VMTable* table,char* asmtext,int len);

  // 解释子调用

  BOOL InterpretCall(VMTable* table,char* asmtext,int len);

  // 普通指令

  BOOL CommonInstruction(VMTable* table,char* asmtext,int len);

  // 解释保护堆栈 Handler

  BOOL InterpretSaveEsp(VMTable* table,char* asmtext,int len);

  // 解释恢复堆栈 Handler

  BOOL InterpretRestoreEsp(VMTable* table,char* asmtext,int len);

};

// ----------------------------------------------- vmdisasm.h -------------------------------------

////////////////////////////////////////////////////////////////////////////////////

// 新段各部分空间的长度

#define JumpTableLen    (HANDLERMAXLEN * 4)

#define CodeEngineLen    (4096 * 4)      // 虚拟机引擎代码长度

#define EnterStubAddrLen  (4096)        // 重新进入 VM 的 Stub，长度 1024 字节

#define VMEnterCodeLen    (4096 * 2)      // 空间存放重新进入的 vmcode

#define VMCodeLen      (10240 * 2)      // 2K 的空间存放 VMCode

#define ASMTEXTLEN      1024        // 汇编文本长度

////////////////////////////////////////////////////////////////////////////////////

//8086 指令字符串长度

#define X86NAMELEN         32

// 虚拟机指令名的长度限制

#define VMNAMELEN         32
```

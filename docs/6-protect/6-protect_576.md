

# 虚拟机完整分析

> 原文：[`www.pediy.com/kssd/pediy11/116601.html`](https://www.pediy.com/kssd/pediy11/116601.html)

最近一段时间，看了《加密与解密》3 版里的虚拟机，感觉挺有意思，现将自已的分析发表如下（里面有些是原作者的分析，但大部分我已作更详细分析），与大家共享  
本想将其发表至《加密与解密》版块中，但自已是新人，只好发表在这里了  
  
// 发表的内容以文件为单元（因为比较容易，懒得去整理所有的思路了）  
  
// ----------------------------------------------- 源程序文件部分 -------------------------------------------------------  
  
  
// ------------------------------------------ VCommand.cpp --------------------------------------  
  
// 虚拟机的一部分指令，用于模拟某些汇编指令  
  
#include "stdafx.h"  
#include "VCommand.h"  
  
// 需手工编写的虚拟机指令  
  
/\*\*  
 \* 进入虚拟机  
 \*/  
void \_declspec(naked) VStartVM()  // \*  
{  
  \_asm  
  {  
    // 将寄存器压入堆栈  
    push eax  
    push ebx  
    push ecx  
    push edx  
    push esi  
    push edi  
    push ebp  
    pushfd  
    // edi -> VMContext  
    // esi -> 当前字节码地址  
    // ebp -> 真实堆栈  
    mov    esi, \[esp+0x20\]      // 伪代码开始的地址  
    mov     ebp, esp        // ebp 保存原堆栈指针  
    sub    esp, 0x200  
    mov     edi, esp        // edi 指向 VMContext  
    sub    esp, 0x40        // esp 现在是 vm 用的堆栈了  
    mov    ebx, esi        // 把伪代码 base\_addr 做 key  
  
    movzx  eax, byte ptr \[esi\]    // 获得 bytecode  
    lea    esi, \[esi+1\]      // 跳过这个字节  
      
    VM\_END              // VM 结束标记  
  }  
}  
  
/\*\*  
 \* 检测真实堆栈是否已接近 VMContext 所在的位置，如果是  
 \* 则将 VMContext 结构复制到更远的位置存放  
 \*/  
void DCheckESP()  // \*  
{  
  \_asm  
  {  
    // edi -> 指向 VMContext  
    // ebp -> 指向真实的堆栈  
  
    // 判断 VMContext 往上 0x100 处是否已接近真实堆栈  
    lea     eax, dword ptr \[edi + 0x100\]  
    cmp     eax, ebp  
  
    nop  
    nop  
  
    // 计算 VMContext 与虚拟机堆栈之间的长度  
    mov     edx, edi  
    mov     ecx, esp  
    sub     ecx, edx  
  
    push  esi            // 保存虚拟指令 IP 指针  
    mov    esi, esp  
    sub    esp, 0x60  
    mov    edi, esp  
    push  edi            // 保存新的 edi 基地址  
    sub    esp, 0x40  
    cld  
    rep movsb            // 开始复制  
  
    // 恢复 IP 指针  
    pop edi  
    pop esi  
    VM\_END              // VM 结束标记  
  }  
}  
  
/\*\*  
 \* 将 32 位寄存器压入堆栈中  
 \*/  
void \_declspec(naked) DPushReg32()  // \*  
{  
  \_asm  
  {  
    mov    eax, dword ptr \[esi\]  // 得到寄存器偏移  
    add    esi, 4  
    mov    eax, dword ptr \[edi + eax\]  // 得到寄存器的值  
    push  eax              // 压入寄存器  
    VM\_END              // VM 结束标记  
  }  
}  
  
/\*\*  
 \* 将 32 位直接数压入堆栈中  
 \*/  
void \_declspec(naked) DPushImm32()  // \*  
{  
  \_asm  
  {  
    // 取得直接数  
    mov    eax, dword ptr \[esi\]  
    add    esi, 4  
    push  eax  
    VM\_END              // VM 结束标记  
  }  
}  
  
/\*\*  
 \* 将 32 位内存数压入堆栈中  
 \*  
 \* 格式：\[rax + rcx \* 8 + 0x11223344\]  
 \*/  
void \_declspec(naked) DPushMem32()    // \*  
{  
  \_asm  
  {  
    mov    edx, 0  
    mov    ecx, 0  
    mov    eax, dword ptr \[esp\]  // 第 1 个寄存器偏移  
    test   eax, eax  
  
    // edx = 第 1 个寄存器值  
    cmovge  edx, dword ptr \[edi + eax\]  // 如果不是负数则赋值  
    mov    eax, dword ptr \[esp + 4\]  // 第 2 个寄存器偏移  
    test   eax, eax  
  
    // ecx = 第 2 个寄存器值  
    cmovge  ecx, dword ptr \[edi + eax\]  
    imul  ecx, dword ptr \[esp + 8\]  // 第 2 个寄存器的乘积  
    add    ecx, dword ptr \[esp + 0x0C\]    // 第三个为常量  
    add    edx, ecx  
    add    esp, 0x10        // 释放参数  
  
    // 压入该数  
    // 问题：这里压入的似乎不是内存数，而是该数的地址  
  
    push  edx  
    VM\_END              // VM 结束标记  
  }  
}  
  
/\*\*  
 \* 将堆栈中的数据弹回寄存器  
 \*/  
void \_declspec(naked) DPopReg32()  // \*  
{  
  \_asm  
  {  
    mov    eax, dword ptr \[esi\]  // 得到 reg 偏移  
    add    esi, 4  
    pop    dword ptr \[edi + eax\]  // 弹回寄存器  
    VM\_END              // VM 结束标记  
  }  
}  
  
/\*\*  
 \* 释放堆栈  
 \*/  
void \_declspec(naked) DFree()  // \*  
{  
  \_asm  
  {  
    add esp, 4  
    VM\_END              // VM 结束标记  
  }  
}  
  
// ------------------------------------------------ comm.cpp ------------------------------------------------------------  
  
// 此文件包含一些公共的函数  
#include "stdafx.h"  
#include "comm.h"  
/\*\*  
 \* 显示提示信息对话框  
 \*  
 \* content: 显示的提示信息  
 \* nType: 显示的提示图标的类型  
 \*/  
void MsgBox(char \* content, UINT nType)    // \*  
{  
  if (!content)  
    return;  
  MessageBoxA(NULL, content, VMPACKERTITLE, nType);  
}  
  
/\*\*  
 \* 去除 str 字符串中的空格，并将其能守 pstr 返回  
 \*  
 \* 返回值: 成功返回 TRUE，否则 FALSE  
 \*/  
BOOL Trim(char \* pstr, char \* str)    // \*  
{  
  int i, j = 0;  
  
  // 检测参数是否有效  
  if (!pstr || !str)  
    return FALSE;  
  
  // 将 pstr 复制至 str 并计算其长度  
  strcpy(str, pstr);  
  
  j = strlen(str);  
  i = 0;  
  // 遍历 str 中的字符  
  for (i = 0; i < j; i ++)  
  {  
    // 如果 i 字符是换行或回车，则替换为 0  
    if ((str\[i\] == 0x0a) || (str\[i\] == 0x0d))  
    {  
      str\[i\] = 0;  
    }  
  }  
  j = strlen(str);  
  i = 0;  
  // 统计 str 头部的空格数  
  while (str\[i\] == ' ')  
    i++;  
  // 如果 str 中都是空格，则处理失败  
  if (i >= j)  
  {  
    return FALSE;  
  }  
  // 去除头部的空格  
  memmove(str, str + i, j - i + 1);  
  j = strlen(str);  
  i = 0;  
  // 统计 str 尾部的空格数  
  while (str\[i\] == ' ')  
    i--;  
  if (i)  
    str\[i\] = '\\0';  
  if (i >= j)  
  {  
    return FALSE;  
  }  
  return TRUE;  
}  
  
/\*\*  
 \* 将指定字符串 strSource 转为 16 进制数并返回  
 \*  
 \* strSource: 欲转为 16 进制数的数字字符串  
 \*  
 \* 返回值：成功返回转换后的结果，否则-1  
 \*/  
DWORD StringToHex(char \* strSource)    // \*  
{   
  DWORD nTemp = 0;  
  char strTemp\[64\];  
  
  // 如果字符串为空，则不需要转，直接返回  
  if (strSource == NULL)  
    return -1;  
  
  // 将源数据复制到临时变量中  
  strcpy(strTemp, strSource);  
  
  // 16 进制只能包含 A、B、C、D、E、F 字符，所以，如果包含除此以外其他字符则出错返回  
  for (char cc = 'G', dd = 'g'; cc <= 'Z', dd <= 'z';  cc++, dd++)  
  {  
    if (strchr(strTemp, cc) != NULL || strchr(strTemp, dd) != NULL)  
    {  
      return -1;  
    }  
  }  
  
  // 遍历 strSource 中所有的字符  
  for (int i = 0; i<(int)::strlen(strSource); i++)  
  {  
    int nDecNum;  
    // 根据每个字符进行处理  
    switch (strSource\[i\])  
    {  
    case 'a':  
    case 'A':  
      nDecNum = 10;  
      break;  
    case 'b':  
    case 'B':  
      nDecNum = 11;  
      break;  
    case 'c':  
    case 'C':  
      nDecNum = 12;  
      break;  
    case 'd':  
    case 'D':  
      nDecNum = 13;  
      break;  
    case 'e':  
    case 'E':  
      nDecNum = 14;  
      break;  
    case 'f':  
    case 'F':  
      nDecNum = 15;  
      break;  
    case '0':  
    case '1':  
    case '2':  
    case '3':  
    case '4':  
    case '5':  
    case '6':  
    case '7':  
    case '8':  
    case '9':  
      nDecNum = strSource\[i\] - '0';  
      break;  
    default:  
      return 0;  
    }  
    nTemp += (DWORD)nDecNum \* (long double)pow((long double)16, (int)(strlen(strSource) - i - 1));  
  }  
  return nTemp;  
}  
  
/\*\*  
 \* 按指定的某种格式输出字符串 str  
 \*/  
void OutPutStr(char \*str, ...)  // \*  
{  
  // #if \_DEBUG  
  char strbuf\[1024\];   
  
  // 处理可变参数  
  va\_list  vl;  
  va\_start(vl, str);  
  vsprintf(strbuf, str, vl);  
  va\_end(vl);  
  
  OutputDebugString(strbuf);  
  // #endif  
}  
  
// ------------------------------------------------ main.cpp：------------------------------------------------------------  
  
// 此文件主要是读取要保护的文件的 PE 信息，加载跟文件有关的 MAP 符号信息，在该 PE 文件的最后区块后加多一个  
// 区块，此区块将用于保存虚拟机的各种信息，如虚拟机指令，跳转表等。接下来，对文件的代码进行反编译后，根  
// 据生成的汇编指令进行虚拟机保护，最后生成受虚拟机保护的新 PE 文件  
  
#include "stdafx.h"  
#include "CCodeILFactory.h"  
#include ".\\pestructure.h"  
  
/\*\*  
 \* 程序入口  
 \*/  
int main()    // \*  
{  
  // PE 文件信息  
  CPEStructure pestruct;  
  CCodeILFactory codefactory;  
  list<CodeNode \*> CodeList;  
  list<CodeNode \*> CodeList1;  
  
  // 打开文件，读取头部，并读取区块数据  
  pestruct.OpenFileName("G:\\\\项目\\\\VMPacker\\\\ConvertCrackMe\\\\Release\\\\ConvertCrackMe.exe");  
  
  // 读取 map 文件，解析其中的符号信息并存于 pestruct.MapVector 中  
  // 注：MAP 文件是一个文本文件，记录了程序的入口地址、基地址、符号及其对应的段、文件偏址等信息  
  pestruct.LoadMap("G:\\\\项目\\\\VMPacker\\\\ConvertCrackMe\\\\Release\\\\ConvertCrackMe.map");  
  getchar();  
  
  // 为 PE 文件 pestruct 分配一个新的区块，并对该区块进行初始化，并生成某  
  // 些虚拟机核心指令，并将指令代码复制到该区块的某个子块中  
  // 注：具体此函数分新的区块结构怎样，可查看“PE 新区块结构图.jpg”  
  codefactory.Init(0x400000 + pestruct.GetNewSection());  
  
  // 查找 CrackMe()函数的符号信息  
  MapStructrue \* stu = pestruct.GetMap("CrackMe(struct HWND\_\_ \*)");  
  // 找不到，则直接返回  
  if (!stu)  
    return 0;  
  
  // 得到符号的基地址  
  char \* Base\_Addr = pestruct.image\_section\[stu->Segment - 1\] + stu->Offset;  
  
  // 反汇编 Base\_Addr 的代码，即函数 CrackMe()的代码  
  codefactory.DisasmFunction(&CodeList, Base\_Addr, stu->VirtualAddress);  
  
  //   
  list<CodeNode\*>::iterator itr;  
    
  itr = CodeList.begin();  
  // 获取 CodeList 指令列表中起始指令  
  CodeNode \* code = \*itr;  
  itr = CodeList.end();  
  itr--;  
  // 获取 CodeList 指令列表中结束指令  
  CodeNode \* endcode = \*itr;  
  
  // 计算 CodeList 的总代码长度  
  int asmlen = endcode->disasm.ip + endcode->disasm.codelen - code->disasm.ip;  
  
  // 组织一段进入虚拟机的指令(即 jmp VStartVM)，存储于 Base\_Addr  
  codefactory.VMFactory.CompileEnterStubCode(Base\_Addr, code->disasm.ip, asmlen);  
  
  // 将文件各头部与区块数据写回至 PE 文件中  
  pestruct.UpdateHeadersSections(FALSE);  
  
  char errtext\[255\] = {0};  
  // 将 CodeList 指令列表编译为虚拟机字节码  
  codefactory.BuildCode(Base\_Addr, &CodeList, errtext);  
  
  // 计算存放虚拟机各个部件的虚存空间长度  
  int len = codefactory.m\_JumpTable.m\_addrlen + codefactory.m\_CodeEngine.m\_addrlen +  
        codefactory.m\_EnterStub.m\_addrlen + codefactory.m\_VMEnterStubCode.m\_addrlen +   
        codefactory.m\_VMCode.m\_addrlen;  
  // 分配足够的空间  
  char \* newdata = new char\[len\];  
  char \* p = newdata;  
  
  // 将跳转表复制到 p 空间中  
  memcpy(p, codefactory.m\_JumpTable.m\_BaseAddr, codefactory.m\_JumpTable.m\_addrlen);  
  p += codefactory.m\_JumpTable.m\_addrlen;  
  // 将虚拟机其他部分复制到 p 空间中  
  memcpy(p, codefactory.m\_CodeEngine.m\_BaseAddr, codefactory.m\_CodeEngine.m\_addrlen);  
  p += codefactory.m\_CodeEngine.m\_addrlen;  
  memcpy(p, codefactory.m\_EnterStub.m\_BaseAddr, codefactory.m\_EnterStub.m\_addrlen);  
  p += codefactory.m\_EnterStub.m\_addrlen;  
  memcpy(p, codefactory.m\_VMEnterStubCode.m\_BaseAddr, codefactory.m\_VMEnterStubCode.m\_addrlen);  
  p += codefactory.m\_VMEnterStubCode.m\_addrlen;  
  memcpy(p, codefactory.m\_VMCode.m\_BaseAddr, codefactory.m\_VMCode.m\_addrlen);  
  p += codefactory.m\_VMCode.m\_addrlen;  
  
  // 为 PE 文件添加.bug 的新区块，即虚拟机  
  pestruct.AddSection(newdata,len,".bug");  
  
  pestruct.UpdateHeaders(FALSE);  
  pestruct.UpdateHeadersSections(TRUE);  
  pestruct.UpdateHeadersSections(FALSE);  
  // 创建一个新的可执行文件  
  pestruct.MakePE("G:\\\\项目\\\\VMPacker\\\\ConvertCrackMe\\\\Release\\\\ConvertCrackMe.vm.exe", len);  
  
  return 0;  
}  
  
  
// ----------------------------------------------------- PEStructure.cpp ----------------------------------------------------------  
  
// 此文件中的代码用于对 PE 文件的各种操作，包括读取等  
  
#include "StdAfx.h"  
#include ".\\pestructure.h"  
  
#include <ImageHlp.h>  
#pragma comment(lib, "ImageHlp")  // 链接到 ImageHlp.lib  
  
CPEStructure::CPEStructure(void)  
{  
  Init();  
}  
  
CPEStructure::~CPEStructure(void)  
{  
  Free();  
}  
  
/\*\*  
 \* 打开指定的文件并读取文件头，区块数据  
 \*  
 \* FileName: 欲打开的文件路径名  
 \*  
 \* 返回值：FALSE 表示失败，否则成功  
 \*/  
BOOL CPEStructure::OpenFileName(char \* FileName)    // \*  
{  
  // 已读取文件的字节数  
  DWORD dwBytesRead = 0;  
  
  // 释放分配的内存资源  
  Free();  
  
  // 打开 FileName 文件  
  hFile = CreateFileA(FileName, GENERIC\_READ,  FILE\_SHARE\_WRITE | FILE\_SHARE\_READ,  
    NULL, OPEN\_EXISTING, FILE\_ATTRIBUTE\_NORMAL, NULL);  
  
  // 打开失败，则显示提示信息并直接返回  
  if (hFile == INVALID\_HANDLE\_VALUE)  
  {  
    MsgBox("打开文件失败", MB\_ICONERROR);  
    return FALSE;  
  }  
  
  // 获取文件大小  
  dwFsize = GetFileSize(hFile, 0);  
  // 文件不存在内容，则关闭文件并提示错误信息  
  if (dwFsize == 0)  
  {  
    CloseHandle(hFile);  
    MsgBox("错误的文件长度", MB\_ICONERROR);  
    return FALSE;  
  }  
  
  // 打开成功，则将文件名复制至 m\_FileName 中  
  strcpy\_s(m\_FileName, 256, FileName);  
  
  // 计算文件与壳的长度和，以便分配足够的内存空间  
  dwOutPutSize = dwFsize + PackerCode\_Size + ALIGN\_CORRECTION;  
  
  // 分配内存，并将其清 0  
  pMem = (char\*)GlobalAlloc(GMEM\_FIXED | GMEM\_ZEROINIT, dwOutPutSize);  
  
  // 内存分配失败，则关闭文件并提示错误信息  
  if (pMem == NULL)  
  {  
    CloseHandle(hFile);  
    MsgBox("申请内存出错", MB\_ICONERROR);  
    return FALSE;  
  }  
  
  // 将文件读取至 pMem 中，dwBytesRead 表示真正读取的数据长度  
  ReadFile(hFile, pMem, dwFsize, &dwBytesRead, NULL);  
  
  // 读取完文件后，就可以关闭文件了  
  CloseHandle(hFile);  
  
  // 将 PE 文件的 DOS 头部复制到 image\_dos\_header 头部中  
  CopyMemory(&image\_dos\_header, pMem, sizeof(IMAGE\_DOS\_HEADER));  
  
  // 检测是否是一个有效的 PE 文件，不是则提示错误信息  
  if (image\_dos\_header.e\_magic != IMAGE\_DOS\_SIGNATURE)  
  {  
    MsgBox("不是一个有效的 PE 文件", MB\_ICONERROR);  
    return FALSE;  
  }  
  
  // 保存 DOS 头部长度  
  ReservedHeaderRO = sizeof(IMAGE\_DOS\_HEADER);  
  
  // 保存保留头部长度值  
  // 注：e\_lfanew 是 PE 头部的偏移，所以，这里的保留头部指的是 DOS-PE 头部之间的部分  
  ReservedHeaderSize = image\_dos\_header.e\_lfanew - sizeof(IMAGE\_DOS\_HEADER);  
  reservedheader = new char\[ReservedHeaderSize\];  
  
  // 复制 PE 头部  
  CopyMemory(&image\_nt\_headers, pMem + image\_dos\_header.e\_lfanew, sizeof(IMAGE\_NT\_HEADERS));  
  
  // PE 文件区块表在文件中的偏移  
  dwRO\_first\_section = image\_dos\_header.e\_lfanew + sizeof(IMAGE\_NT\_HEADERS);  
  
  // 读取文件的头部信息与区块数据  
  UpdateHeadersSections(TRUE);  
    
  return TRUE;  
}  
  
  
  
//----------------------------------------------------------------  
void CPEStructure::UpdateHeaders(BOOL bSaveAndValidate)  
{  
  DWORD SectionNum = image\_nt\_headers.FileHeader.NumberOfSections;  
  
  if( bSaveAndValidate )//TRUE = 保存数据  
  {  
    CopyMemory(&image\_dos\_header,pMem,sizeof(IMAGE\_DOS\_HEADER));  
    ReservedHeaderSize=image\_dos\_header.e\_lfanew - sizeof(IMAGE\_DOS\_HEADER);  
    if( ( ReservedHeaderSize & 0x80000000 ) == 0x00000000)  
    {  
      CopyMemory(reservedheader, pMem+ReservedHeaderRO, ReservedHeaderSize);  
    }  
    CopyMemory(&image\_nt\_headers,  
      pMem+image\_dos\_header.e\_lfanew,  
      sizeof(IMAGE\_NT\_HEADERS));  
    dwRO\_first\_section = image\_dos\_header.e\_lfanew + sizeof(IMAGE\_NT\_HEADERS);  
    CopyMemory(&image\_section\_header, pMem+dwRO\_first\_section, SectionNum \* sizeof(IMAGE\_SECTION\_HEADER));  
  }  
  else        //FALSE = 恢复数据  
  {  
    CopyMemory(pMem, &image\_dos\_header, sizeof(IMAGE\_DOS\_HEADER));  
    ReservedHeaderSize=image\_dos\_header.e\_lfanew - sizeof(IMAGE\_DOS\_HEADER);  
    if( (ReservedHeaderSize & 0x80000000) == 0x00000000)  
    {  
      CopyMemory(pMem+ReservedHeaderRO,reservedheader,ReservedHeaderSize);  
    }  
    CopyMemory(pMem+image\_dos\_header.e\_lfanew,  
      &image\_nt\_headers,  
      sizeof(IMAGE\_NT\_HEADERS));  
    dwRO\_first\_section=image\_dos\_header.e\_lfanew + sizeof(IMAGE\_NT\_HEADERS);  
    CopyMemory(pMem+dwRO\_first\_section, &image\_section\_header, SectionNum \* sizeof(IMAGE\_SECTION\_HEADER));  
  }  
}  
  
/\*\*  
 \* 更新 PE 文件的头部与区块数据  
 \*  
 \* bSaveAndValidate: true 表示读取 PE 文件中的各头部与区块数据  
 \*           false 表示是将文件各头部与区块数据写回至 PE 文件中  
 \*/  
void CPEStructure::UpdateHeadersSections(BOOL bSaveAndValidate)    // \*  
{  
  DWORD i = 0;  
  // 文件区块数  
  DWORD SectionNum = image\_nt\_headers.FileHeader.NumberOfSections;  
  
  // 读取文件头部与区块数据  
  if (bSaveAndValidate)  
  {  
    // 复制文件 DOS 头部  
    CopyMemory(&image\_dos\_header, pMem, sizeof(IMAGE\_DOS\_HEADER));  
  
    // 复制文件保留头部  
    ReservedHeaderSize = image\_dos\_header.e\_lfanew - sizeof(IMAGE\_DOS\_HEADER);  
    // 0x80000000: 10000000000000000000000000000000b，即 32 位最高位置 1  
    if ((ReservedHeaderSize & 0x80000000) == 0x00000000)  
    {  
      CopyMemory(reservedheader, pMem + ReservedHeaderRO, ReservedHeaderSize);  
    }  
    // 复制文件 PE 头部  
    CopyMemory(&image\_nt\_headers, pMem + image\_dos\_header.e\_lfanew,  sizeof(IMAGE\_NT\_HEADERS));  
  
    // 区块表在文件中的偏移  
    dwRO\_first\_section = image\_dos\_header.e\_lfanew + sizeof(IMAGE\_NT\_HEADERS);  
  
    // 复制区块表  
    CopyMemory(&image\_section\_header, pMem + dwRO\_first\_section, SectionNum \* sizeof(IMAGE\_SECTION\_HEADER));  
    // 遍历所有的区块  
    for (i = 0; i < SectionNum; i++)  
    {  
      // 为每个区块分配内存  
      image\_section\[i\] = (char\*)GlobalAlloc(GMEM\_FIXED | GMEM\_ZEROINIT,  
        PEAlign(image\_section\_header\[i\].SizeOfRawData, image\_nt\_headers.OptionalHeader.FileAlignment));  
  
      // 复制该区块数据  
      CopyMemory(image\_section\[i\], pMem + image\_section\_header\[i\].PointerToRawData,  
        image\_section\_header\[i\].SizeOfRawData);  
    }  
    SectionAssigned = TRUE;    // 已经为区块分配空间  
  }  
  // 将数据写回文件头部与区块数据  
  else  
  {  
    // 写回 DOS 头部  
    CopyMemory(pMem, &image\_dos\_header, sizeof(IMAGE\_DOS\_HEADER));  
  
    ReservedHeaderSize = image\_dos\_header.e\_lfanew - sizeof(IMAGE\_DOS\_HEADER);  
    if ((ReservedHeaderSize & 0x80000000 ) == 0x00000000)  
    {  
      CopyMemory(pMem + ReservedHeaderRO, reservedheader, ReservedHeaderSize);  
    }  
    CopyMemory(pMem + image\_dos\_header.e\_lfanew, &image\_nt\_headers,  sizeof(IMAGE\_NT\_HEADERS));  
    dwRO\_first\_section = image\_dos\_header.e\_lfanew + sizeof(IMAGE\_NT\_HEADERS);  
    CopyMemory(pMem + dwRO\_first\_section, &image\_section\_header, SectionNum \* sizeof(IMAGE\_SECTION\_HEADER));  
  
    for (i = 0; i < SectionNum; i++)  
    {  
      CopyMemory(pMem + image\_section\_header\[i\].PointerToRawData,  
        image\_section\[i\], image\_section\_header\[i\].SizeOfRawData);  
    }  
  }  
}  
  
/\*\*  
 \* 根据 dwAlignTo 对齐值对 dwTarNum 进行对齐操作  
 \*  
 \* dwTarNum: 欲进行对齐的值  
 \* dwAlignTo: 进行对齐的参照值  
 \*/  
DWORD CPEStructure::PEAlign(DWORD dwTarNum, DWORD dwAlignTo)    // \*  
{    
  DWORD dwtemp;  
  dwtemp = dwTarNum / dwAlignTo;  
  if ((dwTarNum % dwAlignTo) != 0)  
  {  
    dwtemp++;  
  }  
  dwtemp = dwtemp \* dwAlignTo;  
  return(dwtemp);  
}  
  
  
/\*\*  
 \* 获取 strAddress 中的段与文件偏移信息  
 \*  
 \* StrAddress：如 0001:00000003  
 \* Segment: 用于返回段信息，即 0001  
 \* FileOffset: 用于返回文件偏移量，即 00000003  
 \*  
 \* 返回值：成功返回 TRUE，否则 FALSE  
 \*/  
BOOL CPEStructure::GetFileAddr(char \* StrAddress, int \* Segment, int \* FileOffset)  // \*  
{  
  // 检测参数是否有效  
  if (!StrAddress || !Segment || !FileOffset)  
    return FALSE;  
  char StrSegment\[4 + 1\] = {0};  
  char StrOffset\[8 + 1\] = {0};  
  // 解析 StrAddress  
  memcpy(StrSegment, StrAddress, 4);  
  memcpy(StrOffset, StrAddress + 5, 8);  
  // 获取段与文件偏移信息  
  \*Segment = StringToHex(StrSegment);  
  \*FileOffset = StringToHex(StrOffset);  
  return TRUE;  
}  
  
/\*\*  
 \* 读取指定 pmapfilename 文件，解析其中的符号信息，将每个符号的信息存放在 MapVector 中  
 \*  
 \* pmapfilename: MAP 文件的路径名  
 \*  
 \* 注：MAP 文件是一个文本文件，记录了程序的入口地址、基地址、符号及其对应的段、偏址等信息  
 \*/  
void CPEStructure::LoadMap(char \* pmapfilename)    // \*  
{  
  if (!pmapfilename)  
    return;  
  FILE \* mapfile = NULL;  
  
  // 打开 MAP 文件  
  mapfile = fopen(pmapfilename, "r+");  
  if (!mapfile)  
    return;  
  
  // 释放 MapVector 所有符号信息占用的资源  
  FreeMapVector();  
  
  char linestr\[512\] = {0};  // 行数据  
  int  readcnt = 0;    // 已读取的行数  
  BOOL bBegin = FALSE;    // 开始  
  
  // 循环读取 MAP 文件，直至文件结束  
  while (!feof(mapfile))  
  {  
    // 清 0 行数据  
    memset(linestr, 0, 512);  
    // 读取 MAP 文件的每一行数据，如果读取不到数据，则结束  
    if (fgets(linestr, 512, mapfile) == NULL)  
    {  
      break;  
    }  
    readcnt++;  // 累计读取的行数  
  
    // 如果该行是模块入口地址数据，则跳过  
    if (strstr(linestr, "entry point at") != NULL)  
    {  
      continue;  
    }  
    // 如果该行表示行号数据，则跳过  
    if (strstr(linestr, "Line numbers for") != NULL)  
    {  
      break;  
    }  
  
    // 符号信息包括符号在节内的偏移地址、加载地址及符号出处等，其格式如下：  
    // Address      Publics by Value      Rva+Base  Lib:Object  
    // 0000:00000003  \_\_\_safe\_se\_handler\_count  00000003  <absolute>  
  
    // 或者搜索静态符号  
  
    if ((strstr(linestr, "Publics by Value") != NULL && strstr(linestr, "Rva+Base") != NULL) ||  
      (strstr(linestr, "Static symbols") != NULL ))  
    {  
      // 如果搜索到，则标注为开始  
      bBegin = TRUE;  
      continue;  
    }  
  
    char StrLine\[512\] = {0};  
  
    // 将 linestr 去掉空格后放到 StrLine 中，如果该行是空行，则跳过  
    if (!Trim(linestr, StrLine))  
    {  
      continue;  
    }  
  
    // 开始分析 StrLine 符号行数据信息  
    if (bBegin)  
    {  
      // 这里的地址指的是文件中的地址  
      char StrAddress\[16\] = {0};  // 段: 文件偏移  
      char StrSymbol\[512\] = {0};  // 符号名  
      char StrVirtualAddr\[16\] = {0};  // 虚拟地址  
      char StrLibObject\[512\] = {0};  // 输入文件名  
      int Segment = 0;  
      int FileOffset = 0;  
  
      // 复制符号地址，如：0001:00000003  
      memcpy(StrAddress, StrLine, 13);  
      int nstrlen = strlen(StrLine);  
  
      memmove(StrLine, StrLine + 13, nstrlen - 13 + 1);  // 跳过段偏移  
  
      // 获取符号所在的段与文件偏移  
      GetFileAddr(StrAddress, &Segment, &FileOffset);  
  
      // 段是不能为 0 的  
      if (Segment == 0)  
        continue;  
  
      // 将剩余内容去掉空格  
      Trim(StrLine, StrLine);  
  
      // 复制符号名  
      strcpy\_s(StrSymbol, 512, StrLine);  
  
      // 符号名结束  
      \*strchr(StrSymbol, ' ') = '\\0';  
      // 计算符号名的长度  
      int nlen = strlen(StrSymbol);  
  
      // 计算剩余部分的长度  
      nstrlen = strlen(StrLine);  
      memmove(StrLine, StrLine + nlen, strlen(StrLine) - nlen + 1);  // 跳过符号名  
  
      // 去除空格  
      Trim(StrLine, StrLine);  
  
      // 获取符号的虚拟地址  
      memcpy(StrVirtualAddr, StrLine, strchr(StrLine, ' ') - StrLine);  
  
      nstrlen = strlen(StrLine);  
      memmove(StrLine, StrLine + 8, nstrlen - 8 + 1);    // 跳过 VirtualAddress  
  
      nstrlen = strlen(StrLine);  
      memmove(StrLine, StrLine + 5, nstrlen - 5 + 1);  // 跳过 f i ...  
      // 输入文件名  
      strcpy(StrLibObject, StrLine);  
  
      // 分配符号信息结构，将获取的符号信息保存起来  
      MapStructrue \* mapstruct = new MapStructrue;  
      mapstruct->Segment = Segment;        // 段  
      mapstruct->Offset = FileOffset;        // 偏移  
      strcpy(mapstruct->SymbleName, StrSymbol);    // 符号名  
      mapstruct->VirtualAddress = StringToHex(StrVirtualAddr);  // 线性地址  
      strcpy(mapstruct->LibObject, StrLibObject);    // 输入文件名  
  
      // 添加文件的符号信息  
      MapVector.push\_back(mapstruct);  
    }  
  }  
  
  // 关闭文件  
  fclose(mapfile);  
  mapfile = NULL;  
  
  // 遍历 MapVector 中的所有符号信息  
  for (vector<MapStructrue\*>::iterator itr = MapVector.begin(); itr != MapVector.end(); itr++)  
  {  
    MapStructrue \* stu = \* itr;  
    // 获取符号段、文件偏移信息  
    int a = stu->Segment;  
    int b = stu->Offset;  
    int nlen = strlen("\_\_ehhandler$");  
    // 如果符号名存在\_\_e...，则将其截掉  
    if (strstr(stu->SymbleName, "\_\_ehhandler$"))  
      memmove(stu->SymbleName, stu->SymbleName + nlen, strlen(stu->SymbleName) - nlen + 1);  
  
    // 将修饰后名称转换成函数签名，失败则直接跳出循环  
    // 注： 由于不同的编译器采用不同的名字修饰方法，必然会导致由不同编译器编译产生的目标  
    //    文件无法正常相互链接，这是导致不同编译器之间不能互操作的主要原因之一  
    if (!UnDecorateSymbolName(stu->SymbleName, stu->SymbleName, 512, 0xFFF))  // no UNDNAME\_COMPLETE  
    {  
      // UnDecorateSymbolName failed  
      DWORD error = GetLastError();  
      char strerr\[255\] = {0};  
      sprintf(strerr, "解析符号名时出现错误 %d\\n", error);  
      MsgBox(strerr, MB\_ICONERROR);  
      break;  
    }  
  
    // 输出符号信息，主要为了调试  
    OutPutStr("%s %08X %s\\n", stu->SymbleName, stu->VirtualAddress, stu->LibObject);  
  }  
}  
  
/\*\*  
 \* 根据函数名 funcname 获取相应的符号信息  
 \*  
 \* funcname：欲获取符号信息的函数(符号)名  
 \*  
 \* 返回值：找到相应符号信息则返回，否则返回 NULL  
 \*/  
MapStructrue\* CPEStructure::GetMap(char\* funcname)  // \*  
{  
  // 遍历文件所有符号信息  
  for (vector<MapStructrue\*>::iterator itr = MapVector.begin(); itr != MapVector.end(); itr++)  
  {  
    MapStructrue \* stu = \* itr;  
    // 检测是否是欲查找符号信息  
    if (\_stricmp(stu->SymbleName, funcname) == 0)  
      return stu;  
  }  
  return NULL;  
}  
  
/\*\*  
 \* 为文件新区块分配一个虚拟区间，并返回其虚拟区间起始地址  
 \*/  
DWORD CPEStructure::GetNewSection()    // \*  
{  
  // 获取 PE 文件的区块数  
  DWORD SectionNum = image\_nt\_headers.FileHeader.NumberOfSections;  
  return PEAlign(image\_section\_header\[SectionNum - 1\].VirtualAddress + image\_section\_header\[SectionNum - 1\].Misc.VirtualSize,  
      image\_nt\_headers.OptionalHeader.SectionAlignment);  
}  
  
  
  
  
  
  
//----------------------------------------------------------------  
// return values:  
// 0 - no room for a new section  
// 1 - file already encrypted  
// else: returns a pointer to the IMAGE\_SECTION\_HEADER struct of the new section  
  
/\*\*  
 \* 在 PE 文件中添加新的区块  
 \*  
 \* Base：新区块的数据基址  
 \* len：新区块的数据长度  
 \* SectionName：新区块的名称  
 \*  
 \* 返回值：成功返回新区块的结构信息，否则 NULL  
 \*/  
PIMAGE\_SECTION\_HEADER CPEStructure::AddSection(char \* Base, int len, char \* SectionName)  // \*  
{  
  DWORD newSectionOffset;  
  // 获取 PE 文件区块数  
  DWORD SectionNum = image\_nt\_headers.FileHeader.NumberOfSections;  
  // 计算新区块在区块表中的偏移  
  newSectionOffset = dwRO\_first\_section + image\_nt\_headers.FileHeader.NumberOfSections \* sizeof(IMAGE\_SECTION\_HEADER);  
  
  // 检测是否有足够空间在放置新区块  
  if (image\_nt\_headers.OptionalHeader.SizeOfHeaders < (newSectionOffset + sizeof(IMAGE\_SECTION\_HEADER)))  
  {  
    return NULL;  
  }  
  
  // 创建一个新的区块  
  
  // 遍历前面所有区块，设置每个区块为可写状态  
  for (DWORD i = 0; i < (SectionNum - 1); i++)  
  {  
    image\_section\_header\[i\].Characteristics |= IMAGE\_SCN\_MEM\_WRITE;  
  }  
  
  // 创建一个新的区块，将最后区块表中的记录复制作为新区块的记录  
  CopyMemory(&image\_section\_header\[SectionNum\], &image\_section\_header\[SectionNum - 1\], sizeof(IMAGE\_SECTION\_HEADER));  
  
  // 设置新区块的虚拟地址  
  image\_section\_header\[SectionNum\].VirtualAddress = PEAlign(image\_section\_header\[SectionNum - 1\].VirtualAddress +  
    image\_section\_header\[SectionNum - 1\].Misc.VirtualSize, image\_nt\_headers.OptionalHeader.SectionAlignment);  
  
  // 设置新区块的长度  
  image\_section\_header\[SectionNum\].Misc.VirtualSize = len;  
  image\_section\_header\[SectionNum\].SizeOfRawData = PEAlign(image\_section\_header\[SectionNum\].Misc.VirtualSize,  
                           image\_nt\_headers.OptionalHeader.FileAlignment);  // bughoho  
  dwOutPutSize += image\_section\_header\[SectionNum\].SizeOfRawData;  
    
  // 计算区块名长度  
  int l = (int)strlen(SectionName);  
  FillMemory(image\_section\_header\[SectionNum\].Name, 8, 0x00);  
  // 设置区块名  
  CopyMemory(image\_section\_header\[SectionNum\].Name, SectionName, l);  
    
  // 设置新区块的特性  
  image\_section\_header\[SectionNum\].Characteristics = IMAGE\_SCN\_MEM\_EXECUTE |  
                          IMAGE\_SCN\_MEM\_WRITE |   
                          IMAGE\_SCN\_MEM\_READ |  
                          IMAGE\_SCN\_MEM\_EXECUTE |  
                          IMAGE\_SCN\_CNT\_UNINITIALIZED\_DATA |  
                          IMAGE\_SCN\_CNT\_INITIALIZED\_DATA |  
                          IMAGE\_SCN\_CNT\_CODE;  
  
  // RawOffset  
  image\_section\_header\[SectionNum\].PointerToRawData = PEAlign(image\_section\_header\[SectionNum-1\].PointerToRawData  
                            + image\_section\_header\[SectionNum - 1\].SizeOfRawData,  
                            image\_nt\_headers.OptionalHeader.FileAlignment);  
  // 递增 PE 文件区块数  
  image\_nt\_headers.FileHeader.NumberOfSections++;  
    
  // 重新设置 PE 文件映像长度  
  image\_nt\_headers.OptionalHeader.SizeOfImage = image\_section\_header\[SectionNum\].VirtualAddress +  
                          image\_section\_header\[SectionNum\].Misc.VirtualSize;  
  
  // 复制新区块的数据  
  memcpy(pMem + image\_section\_header\[SectionNum\].PointerToRawData, Base, len);  
  return ((PIMAGE\_SECTION\_HEADER)&image\_section\_header\[SectionNum\]);  
}  
  
/\*\*  
 \* PE 文件初始化操作  
 \*/  
void CPEStructure::Init()  // \*  
{  
  // 未分配区块内存  
  SectionAssigned       = FALSE;  
  
  // 未分配 PE 保留头部  
  reservedheader        = NULL;  
  // 保留头部长度为 0  
  ReservedHeaderSize    = 0;  
  // DOS 头部长度为 0  
  ReservedHeaderRO      = 0;  
  hFile                 = 0;  
  dwFsize               = 0;  
  dwOutPutSize          = 0;  
  
  // 清 0 文件名  
  memset(m\_FileName, 0, 255);  
  memset(m\_MapFileName, 0, 255);  
}  
  
/\*\*  
 \* 释放 MapVector 中存储的符号信息占用的资源  
 \*  
 \* 注：MapVector 存放 PE 文件的符号信息  
 \*/  
void CPEStructure::FreeMapVector()  // \*  
{  
  // 遍历 PE 文件的符号信息  
  for (vector<MapStructrue\*>::iterator itr = MapVector.begin(); itr != MapVector.end(); itr++)  
  {  
    MapStructrue \* stu = \* itr;  
    if (stu)  
    {  
      delete stu;  
    }  
  }  
  MapVector.clear();  
}  
  
/\*\*  
 \* 释放为 PE 文件分配的资源  
 \*/  
void CPEStructure::Free()  // \*  
{  
  // 如果 PE 文件的保留头部内存占用着，则释放之  
  if (reservedheader)  
  {  
    delete\[\] reservedheader;  
    reservedheader = NULL;  
  }  
  // 如果未为 PE 文件分配区块内存，则不需要释放  
  if (!SectionAssigned)  
    return;  
  
  // 获取 PE 文件区块数  
  DWORD SectionNum = image\_nt\_headers.FileHeader.NumberOfSections;  
  
  // 遍历所有的区块，释放为其分配的内存  
  for (DWORD i = 0 ;i < SectionNum; i++)  
  {  
    GlobalFree(image\_section\[i\]);  
    image\_section\[i\] = NULL;  
  }  
  Init();  
}  
  
  
  
  
  
/\*\*  
 \* 根据指定的文件名 filename，创建一个新的文件，并写入虚拟机的数据  
 \*  
 \* filename：欲创建的新文件名  
 \* len：文件中数据的长度  
 \*/  
void CPEStructure::MakePE(char \* filename, int len)    // \*  
{  
  HANDLE handle;  
  DWORD Num;  
  // 根据文件名创建一个新文件  
  handle= ::CreateFileA(filename, GENERIC\_READ | GENERIC\_WRITE, 0, NULL, OPEN\_ALWAYS, FILE\_ATTRIBUTE\_NORMAL, NULL);  
  
  // 文件打开失败，则出错返回  
  if (INVALID\_HANDLE\_VALUE == handle)  
  {  
    MessageBox(0, "打开文件失败!", "错误", MB\_OK);  
    return;  
  }  
  ::SetFilePointer(handle, 0, 0, FILE\_BEGIN);  
  // 将虚拟机写回文件  
  ::WriteFile(handle, pMem, dwOutPutSize, &Num, NULL);  
  ::CloseHandle(handle);  
}  
  
// ------------------------------------------------------- 相关数据结构 --------------------------------------------------------  
  
// ------------------------------------------------------- PEStructure.h ------------------------------------------------------  
  
// 文件中包含 PE 文件相关的结构信息  
  
  
// #pragma once：用来防止某个头文件被多次 include  
#pragma once  
#include "comm.h"  
  
#include <math.h>  
  
#include <vector>  
using namespace std;  
  
// PE 文件最大区块数  
#define MAX\_SECTION\_NUM 20  
  
const DWORD PackerCode\_Size = 0;  // 壳代码的长度  
const DWORD ALIGN\_CORRECTION = 0;  // 文件对齐值  
  
/\*\*  
 \* 代表 PE 文件的信息、数据及其相应 MAP 文件的信息  
 \*  
 \* 注：MAP 文件是一个文本文件，记录了程序的入口地址、基地址、  
 \*     符号所在段、偏移信息的文本文件  
 \*/  
class CPEStructure  
{  
public:  
  CPEStructure(void);  
  ~CPEStructure(void);  
private:  
  
  // PE 文件保留头部指的是 DOS 头部与 PE 头部之间的部分  
  
  DWORD   ReservedHeaderSize;    // PE 文件保留头部长度  
  DWORD   ReservedHeaderRO;    // PE 文件 DOS 头部长度  
  
  // 表明是否已为 PE 文件的区块分配了存储数据的内存空间  
  BOOL    SectionAssigned;  
  HANDLE  hFile;          // PE 文件句柄  
  DWORD   dwFsize;        // PE 文件长度  
  DWORD   dwOutPutSize;      // 对 PE 文件处理后的输出长度，为 PE 文件长度 + 壳长度  
  
  // 指向 PE 文件的缓存区，该缓存区用于存文 PE 文件处理后的结果  
  char    \* pMem;  
private:  
  char    m\_FileName\[256\];    // PE 文件名  
  char    m\_MapFileName\[256\];    // PE 文件对应的 MAP 文件名  
  
public:  
  // 存储 PE 文件的符号信息，其每个元素是一个 MapStructure，代表 PE 文件的符号信息  
  vector<MapStructrue\*> MapVector;  
  
  // 加载并解析 MAP 文件的符号信息  
  void LoadMap(char \* pmapfilename);  
  
  // 用于根据 StrAddress 符号地址得到符号的段号与文件偏移  
  BOOL GetFileAddr(char \* StrAddress, int \* Segment, int \* FileOffset);  
public:  
  MapStructrue \* GetMap(char \* funcname);  
public:  
  // PE 文件区块表在文件中的偏移  
  DWORD          dwRO\_first\_section;  
  
  // DOS 头部  
  IMAGE\_DOS\_HEADER    image\_dos\_header;  
  
  // 指向 PE 文件的保留头部信息  
  char          \* reservedheader;  
  
  // PE 文件头部  
  IMAGE\_NT\_HEADERS    image\_nt\_headers;  
  
  // 区块表，最多 20 项  
  IMAGE\_SECTION\_HEADER  image\_section\_header\[MAX\_SECTION\_NUM\];  
  
  // 为区块分配的内存，每个元素指向区块内存  
  char          \* image\_section\[MAX\_SECTION\_NUM\];  
  BOOL OpenFileName(char \* FileName);  
  void UpdateHeaders(BOOL bSaveAndValidate);             // 更新 PE 头  
  void UpdateHeadersSections(BOOL bSaveAndValidate);         // 更新 PE 节  
  DWORD PEAlign(DWORD dwTarNum, DWORD dwAlignTo);           // PE 节对齐  
  PIMAGE\_SECTION\_HEADER AddSection(char \* Base, int len, char \* SectionName);  // 添加节  
  DWORD GetNewSection();                       // 获得新节的 VirtualAddress 地址  
  void Init();  
  void Free();  
  void FreeMapVector();  
  void MakePE(char \* filename, int len);    // 制造一个 PE  
};  
  
  
  
// 还有大量分析，等续 

// --------------------------------------------- CVirtualMemory.cpp -----------------------------  
  
// 此文件代表了某段虚存空间，我们需要在 PE 文件中，为存放虚拟机的指令分配虚存空间  
  
// VMPackLib  
  
#include "stdafx.h"  
#include "CVirtualMemory.h"  
  
CVirtualMemory::CVirtualMemory()  
{  
    m\_BaseAddr = NULL;  
    m\_addrlen  = 0;  
    m\_VirtualBaseAddress = 0;  
    CurrentAddress = NULL;  
}  
  
CVirtualMemory::CVirtualMemory(DWORD VirtualAddress,  
                 int len)  
{  
    m\_BaseAddr = NULL;  
    m\_addrlen  = 0;  
    m\_VirtualBaseAddress = 0;  
    CurrentAddress = NULL;  
    CreateVirtualMemory(VirtualAddress,len);  
}  
  
CVirtualMemory::~CVirtualMemory()  
{  
  if( m\_BaseAddr ) delete\[\] m\_BaseAddr;m\_BaseAddr = NULL;  
}  
  
/\*\*  
 \* 根据指定虚拟地址分配一块内存空间  
 \*  
 \* VirtualAddress：欲分配内存的虚拟地址  
 \* len：分配内存的长度  
 \*/  
void CVirtualMemory::CreateVirtualMemory(DWORD VirtualAddress,  
                     int len)  // \*  
{  
  // 内存空间存在，则先释放之  
  if (m\_BaseAddr)  
    delete\[\] m\_BaseAddr;  
  m\_BaseAddr = NULL;  
  // 分配内存空间并清 0  
  m\_BaseAddr = new char\[len\];  
  memset(m\_BaseAddr, 0, len);  
  m\_addrlen  = len;  
  m\_VirtualBaseAddress = VirtualAddress;  
  CurrentAddress = m\_BaseAddr;  
}  
  
/\*\*  
 \* 根据内存地址 addr 计算出相应的虚拟地址并返回  
 \*  
 \* addr：欲换算的内存地址  
 \*  
 \* 返回值：换算后的虚拟地址，失败则为-1  
 \*/  
DWORD CVirtualMemory::GetVirtualAddress(char \* addr)  // \*  
{  
  // 检测地址合法性  
  if (addr < m\_BaseAddr)  
    return -1;  
  return m\_VirtualBaseAddress + (DWORD)(addr - m\_BaseAddr);  
}  
  
// 根据虚拟地址计算出内存空间地址  
char\* CVirtualMemory::GetAddrOfVirtualAddr(DWORD VirtualAddress)  
{  
  return m\_BaseAddr + VirtualAddress - m\_VirtualBaseAddress;  
}  
  
/\*\*  
 \* 将指定数据 src 复制到虚拟内存区间  
 \*  
 \* src：欲复制的数据  
 \* len：欲复制数据的长度  
 \*  
 \* 返回值：成功返回复制数据的起始虚址，否则返回 0  
 \*/  
DWORD CVirtualMemory::WriteData(char \* src, int len)  // \*  
{  
  // 检测参数是否 OK  
  if (!src || !len)  
  {  
    MessageBox(0, "内部错误", "err", MB\_OK);  
    return 0;  
  }  
  // 如果虚存空间不足，则出错返回  
  if (m\_addrlen - (CurrentAddress - m\_BaseAddr) < len)  
  {  
    MessageBox(0, "\[memory to small\]内部错误", "err", MB\_OK);  
    return 0;  
  }  
  DWORD StartVirtualAddress = 0;  
  
  // 将 src 复制到当前地址位置  
  memcpy(CurrentAddress, src, len);  
  
  // 取得当前地址对应的虚址，即复制数据所在的虚址  
  StartVirtualAddress = m\_VirtualBaseAddress + (DWORD)(CurrentAddress - m\_BaseAddr);  
  CurrentAddress += len;  
  return StartVirtualAddress;  
}  
  
  
//复制数据到指令虚拟内存地址  
DWORD CVirtualMemory::WriteData(DWORD VirtualCode,char\* src,int len)  
{  
  if( !src || !len )  
    return 0;  
  if( m\_BaseAddr + m\_addrlen - CurrentAddress < len )//空间不够  
    return 0;  
  DWORD StartVirtualAddress = 0;  
  memcpy(GetAddrOfVirtualAddr(VirtualCode),src,len);  
  return VirtualCode;  
}  
// 清空内存  
void CVirtualMemory::ClearMemory()  
{  
  memset(m\_BaseAddr,0,m\_addrlen);  
  CurrentAddress = m\_BaseAddr;  
}  
// 获得还未使用的空间  
char\* CVirtualMemory::GetCurrentAddr(void)  
{  
  return CurrentAddress;  
}  
  
/\*\*  
 \* 获取虚拟内存区间当前可用的虚拟地址  
 \*/  
DWORD CVirtualMemory::GetCurrentVirtualAddress(void)  // \*  
{  
  return GetVirtualAddress(CurrentAddress);  
}  
  
  
  
// ----------------------------------------------- CVirtualMemory.h ------------------------------  
  
// VMPackLib  
  
#ifndef \_\_CVIRTUALMEMORY\_\_  
#define \_\_CVIRTUALMEMORY\_\_  
  
  
/\*\*  
 \* 代表一段虚拟内存  
 \*/  
class CVirtualMemory  
{  
public:  
  // 虚拟地址所对应的内存空间基址  
  char \* m\_BaseAddr;  
  // 内存空间的长度  
  int m\_addrlen;  
  // 此段内存的虚拟地址，将对应 m\_BaseAddr 所指向的内存空间  
  DWORD m\_VirtualBaseAddress;  
  
private:  
  // 此内存空间中当前可以使用的地址  
  // 初始化为 m\_BaseAddr  
  char \* CurrentAddress;  
public:  
  
  CVirtualMemory();  
  CVirtualMemory(DWORD VirtualAddress,  
         int len);  
  ~CVirtualMemory();  
  
  // 创建一片对应虚拟地址的内存空间  
  void  CreateVirtualMemory(DWORD VirtualAddress,  
              int len);  
  
  // 根据内存地址换算出虚拟线性地址  
  DWORD  GetVirtualAddress(char \* addr);  
  
  // 根据虚拟地址计算出内存空间地址  
  char \*  GetAddrOfVirtualAddr(DWORD VirtualAddress);  
  
  // 复制数据到虚拟内存, 返回数据的起始虚拟地址.  
  DWORD  WriteData(char\* src, int len);  
  // 复制数据到指令虚拟内存地址  
  DWORD  WriteData(DWORD VirtualCode, char \* src, int len);  
  // 清空内存  
  void  ClearMemory();  
  
  // 获得当前可用空间  
  char \*  GetCurrentAddr(void);  
  // 获得当前可用的虚拟地质  
  DWORD  GetCurrentVirtualAddress(void);  
};  
  
#endif // \_\_CVIRTUALMEMORY\_\_  
  
  
// ----------------------------------- InterpretHandler.cpp ------------------------------------  
  
// 每个普通的汇编指令，一般都有一个虚拟机指令对应着，而每个虚拟机指令，实际上又对应着某段普通的汇编代码，换句话说，一个普通的汇编指令就由另一段汇编代码来模拟完成了  
// 此文件就是用于为某个汇编指令相应的虚拟机指令生成相应的汇编代码  
// 对于普通汇编指令与虚拟机指令的对应关系由一张表定义着（即 vmtable，见 vmserv.cpp 中定义）  
  
#include "StdAfx.h"  
#include "InterpretHandler.h"  
#include "asm/disasm.h"  
  
CInterpretHandler::CInterpretHandler(void)  
{  
}  
  
CInterpretHandler::~CInterpretHandler(void)  
{  
}  
  
/\*\*  
 \* 获取指定寄存器 RegType 在 VMContext 虚拟机上下文中的偏移  
 \*  
 \*    针对不同的寄存器，偏移值不同，如  
 \*      32 位寄存器：EAX  
 \*      16 位寄存器：AX  
 \*      低 8 位寄存器：AL  
 \*    以上情况直接取得偏移  
 \*      高 8 位寄存器：AH  
 \*    则需要对偏移进行 + 1，以便得到高 8 位值  
 \*  
 \* RegType：指明要取得偏移的寄存器，取值可见枚举 RegType  
 \*/  
int CInterpretHandler::GetRegisterOffset(int RegType)  // \*  
{  
  if (RegType < 0)  
    return RegType;  
  int offset = m\_RegisterIdx\[RegType\] \* 4;  
  
  // 判断是否取寄存器高 8 位  
  if (RegType >= 20)  
    offset++;  
  return offset;  
}  
  
/\*\*  
 \* 初始化操作  
 \*/  
BOOL CInterpretHandler::Init()    // \*  
{  
  // 打乱寄存器偏移的索引  
  RandListIdx(m\_RegisterIdx, REGCOUNT);  
  
  return TRUE;  
}  
  
/\*\*  
 \* 为指定虚拟机指令 table 设置所需操作数  
 \*  
 \* table: 包含某个虚拟机指令与汇编指令相关信息  
 \* asmtext：保存生成的汇编代码  
 \* len: asmtext 缓存的长度  
 \*/  
void CInterpretHandler::SetArg(VMTable \* table, char \* asmtext, int len)  // \*  
{  
  // 操作数不超过 3 个  
  for (int i = 0; i < 3; i++)  
  {  
    // 欲处理操作数超限制，则跳出  
    // OperandNum: 操作数个数  
    if (i >= table->OperandNum)  
      break;  
      
    // 以下将生成：mov EAX, \[esp + (0 | 4 | 8)\]  
    // 所以相当于将堆栈中压入的操作数复制到 EAX 寄存器中以作为虚拟机的操作数使用  
    sprintf\_s(asmtext, len, "%smov %s,\[esp+%02x\]\\n", asmtext, ArgReg\[i\], i \* 4);  
  }  
}  
  
/\*\*  
 \* 将指定虚拟机指令 table 的操作数恢复到堆栈中(操作数一般都被压入栈)  
 \*  
 \* table: 包含某个虚拟机指令与汇编指令相关信息  
 \* asmtext：保存生成的汇编代码  
 \* len: asmtext 缓存的长度  
 \*/  
void CInterpretHandler::RestoreArg(VMTable \* table, char \* asmtext, int len)  // \*  
{  
  for (int i = 0; i < 3; i++)  
  {  
    if (i >= table->OperandNum)  
      break;  
    sprintf\_s(asmtext, len, "%smov \[esp+%02x\],%s\\n", asmtext, i \* 4, ArgReg\[i\]);  
  }  
}  
  
/\*\*  
 \* 将 VMContext 上下文保存的 EFlag 寄存器值恢复到 EFlag 寄存器中  
 \*  
 \* asmtext：保存恢复所需的汇编代码  
 \* len: asmtext 缓存的长度  
 \*/  
void CInterpretHandler::RestoreFlag(char \* asmtext, int len)  // \*  
{  
  sprintf\_s(asmtext, len, "%sPush \[edi+0x%02x\]\\nPopfd\\n", asmtext, GetRegisterOffset(RT\_EFlag));  
}  
  
/\*\*  
 \* 将 EFlag 寄存器值保存到 VMContext 上下文的 EFlag 中  
 \*  
 \* asmtext：保存恢复所需的汇编代码  
 \* len: asmtext 缓存的长度  
 \*/  
void CInterpretHandler::SaveFlag(char \* asmtext, int len)  // \*  
{  
  sprintf\_s(asmtext, len, "%sPushfd\\nPop \[edi+0x%02x\]\\n", asmtext, GetRegisterOffset(RT\_EFlag));  
}  
  
/\*\*  
 \* 根据指定虚拟机指令-汇编指令信息 table 为虚拟机指令生成相应的汇编代码  
 \*  
 \* table: 包含某个虚拟机指令与汇编指令相关信息  
 \* asmtext：保存生成的汇编代码  
 \* len: asmtext 缓存的长度  
 \*  
 \* 返回值：成功返回 TRUE，否则 FALSE  
 \*/  
BOOL CInterpretHandler::InterpretASMStr(VMTable \* table, char \* asmtext, int len)  // \*  
{  
  // 检测参数的合法性  
  if (strcmp(table->VMInstrName, "") == 0 || asmtext == NULL)  
    return FALSE;  
  
  // 清 0 asmtext 缓存  
  memset(asmtext, 0, len);  
  
  // 为虚拟机指令设置操作数  
  SetArg(table, asmtext, len);  
  
  // 某些汇编指令在执行时，需要用到某些寄存器的值来控制其执行，那么，对于相应  
  // 的虚拟指令，当然也是需要，所以，现在需要恢复所需寄存器的值了  
  
  // 为虚拟机指令设置所需的寄存器  
  for (int i = 0; i < 4; i++)  
  {  
    // 判断虚拟机指令是否需要用 i 寄存器  
    if (table->NeedReg\[i\] != NONE && table->NeedReg\[i\] < 14)  
    {  
      // edi -> 指向 VMContext  
  
      // 由于虚拟机将所有寄存器的值保存在 VMContext 中，所以，  
      // 现在将保存的值复制到所需寄存器中以便指令使用  
      sprintf\_s(asmtext, len, "%smov %s,\[edi+0x%02x\]\\n", asmtext,  
        vregname\[2\]\[table->NeedReg\[i\]\], GetRegisterOffset(table->NeedReg\[i\]));  
    }  
  }  
  
  BOOL After = FALSE;  
  
  // 为 vBegin 指令生成汇编代码  
  if (\_stricmp(table->strInstruction, "vBegin") == 0)  
  {  
    After = InterpretvBegin(table, asmtext, len);  
  }  
  // 为 vtoReal 指令生成汇编代码，vtoReal 用于跳转至真实指令  
  else if (\_stricmp(table->strInstruction, "vtoReal") == 0)  
  {  
    After = InterpretvtoReal(table, asmtext, len);  
  }  
  else if (\_stricmp(table->strInstruction, "push") == 0)  
  {  
    After = InterpretPush(table, asmtext, len);  
  }  
  else if (\_stricmp(table->strInstruction, "pop") == 0)  
  {  
    After = InterpretPop(table, asmtext, len);  
  }  
  else if (\_stricmp(table->strInstruction, "pushfd") == 0)  
  {  
    After = InterpretPushfd(table, asmtext, len);  
  }  
  else if (\_stricmp(table->strInstruction, "popfd") == 0)  
  {  
    After = InterpretPopfd(table, asmtext, len);  
  }  
  else if (\_stricmp(table->strInstruction, "pushad") == 0)  
  {  
    After = InterpretPushad(table, asmtext, len);  
  }  
  else if(\_stricmp(table->strInstruction, "popad") == 0)  
  {  
    After = InterpretPopad(table, asmtext, len);  
  }  
  else if (\_stricmp(table->strInstruction, "enter") == 0)  
  {  
    After = InterpretEnter(table, asmtext, len);  
  }  
  else if (\_stricmp(table->strInstruction, "leave") == 0)  
  {  
    After = InterpretLeave(table, asmtext, len);  
  }  
  // 为 jcxz 指令生成相应的汇编代码  
  // 注：jcxz 指令，该指令实现当寄存器 CX 的值等于 0 时转移到标号,否则顺序执行  
  else if (\_stricmp(table->strInstruction, "jcxz") == 0)  
  {  
    // 先将 VMContext 上下文的 EFlag 寄存器恢复到 EFlag 标志寄存器  
    RestoreFlag(asmtext, len);  
  
    After = InterpretJCXZ(table, asmtext, len);  
  }  
  else if (\_stricmp(table->strInstruction, "jmp") == 0)  
  {  
    After = InterpretJMP(table, asmtext, len);  
  }  
  // 不是 jcxz 和 jmp, 则说明是条件跳转指令  
  else if (table->strInstruction\[0\] == 'J' || table->strInstruction\[0\] == 'j')  
  {  
    RestoreFlag(asmtext, len);  
    After = InterpretJCC(table, asmtext, len);  
  }  
  else if (\_stricmp(table->strInstruction, "loope") == 0)  
  {  
    RestoreFlag(asmtext, len);  
    After = InterpretLoope(table, asmtext, len);  
    SaveFlag(asmtext, len);  
  }  
  else if (\_stricmp(table->strInstruction, "retn") == 0)  
  {  
    After = InterpretRetn(table, asmtext, len);  
  }  
  else if (\_stricmp(table->strInstruction, "call") == 0)  
  {  
    After = InterpretCall(table, asmtext, len);  
  }  
  else if (\_stricmp(table->strInstruction, "SaveEsp") == 0)  
  {  
    After = InterpretSaveEsp(table, asmtext, len);  
  }  
  else if (\_stricmp(table->strInstruction, "RestoreEsp") == 0)  
  {  
    After = InterpretRestoreEsp(table, asmtext, len);  
  }  
  else  
  {  
    After = CommonInstruction(table, asmtext, len);  
  }  
    
  // 某些指令执行完成后，将会影响到某些寄存器的值，这里所做的，就是将  
  // 指令执行后改变的寄存器值保存回 VMContext 上下文中的各个寄存器中  
  for (int i = 0; i < 3; i++)  
  {  
    // 如果需保存 i 寄存器，即 i 寄存器被改变了  
    if (table->SaveReg\[i\] != NONE)  
    {  
      sprintf\_s(asmtext, len, "%smov \[edi+0x%02x\],%s\\n", asmtext,  
        GetRegisterOffset(table->NeedReg\[i\]), vregname\[2\]\[table->NeedReg\[i\]\]);  
    }  
  }  
  
  if (After)  
  {  
    RestoreArg(table, asmtext, len);  
  }  
  
  // 将最终为指令生成的汇编代码转为大写  
  \_strupr\_s(asmtext, ASMTEXTLEN);  
  return TRUE;  
}  
  
/\*\*  
 \* 根据操作数的位数 bit，获取相应的修饰符  
 \*  
 \* bit: 操作数的位数  
 \* scalestr：用于返回得到的修饰符  
 \*  
 \* 返回值：返回操作数大小索引：  
 \*    8 位: 0  
 \*    16 位: 1  
 \*    32 位：2  
 \*/  
int GetScalestr(int bit, OUT char \* scalestr)  // \*  
{  
  int sizeidx = 0;  
  if (bit == 8)  
  {  
    sizeidx = 0;  
    strcpy\_s(scalestr, 6, "byte");  
  }  
  else if (bit == 16)  
  {  
    sizeidx = 1;  
    strcpy\_s(scalestr, 6, "word");  
  }  
  else if (bit == 32)  
  {  
    sizeidx = 2;  
    strcpy\_s(scalestr, 6, "dword");  
  }  
  return sizeidx;  
}  
  
/\*\*  
 \* 为普通指令生成相应的汇编代码  
 \*  
 \* table: 存放某个普通指令的信息，根据这些信息可以为相应虚拟机生成汇编代码  
 \* asmtext: 保存生成的汇编代码  
 \* len：asmtext 缓存的长度  
 \*  
 \* 返回值：只返回 TRUE  
 \*/  
BOOL CInterpretHandler::CommonInstruction(VMTable \* table, char \* asmtext, int len)    // \*  
{  
  char scalestr\[6\] = {0};  
  int sizeidx = 0;  
  
  char stroperand\[1024\] = {0};  
  
  // 处理指令的操作数  
  for (int i = 0; i < 3; i++)  
  {  
    // 操作数超限制  
    if (i >= table->OperandNum)  
      break;  
  
    // 获取 i 操作数的修饰符  
    sizeidx = GetScalestr(table->bitnum\[i\], scalestr);  
  
    // 如果 i 操作数是一个内存数  
    if (table->optype\[i\] == MEMTYPE )  
    {  
      sprintf\_s(stroperand, 1024, "%s%s ptr %s\[%s\],", stroperand, scalestr, GetSegStr(table->Segment), vregname\[2\]\[i\]);  
    }  
    // 立即数和寄存器  
    else  
    {  
      // 得到操作数  
      sprintf\_s(stroperand, 1024, "%s%s,", stroperand, vregname\[sizeidx\]\[i\]);  
    }  
  }  
  // 去掉最后操作数的逗号  
  if (table->OperandNum > 0)  
    stroperand\[strlen(stroperand) - 1\] = '\\0';  
  
  RestoreFlag(asmtext, len);  
  // 执行指令  
  sprintf\_s(asmtext,len, "%s%s %s\\n", asmtext, table->strInstruction, stroperand);  
  SaveFlag(asmtext, len);  
  return TRUE;  
}  
  
/\*\*  
 \* 根据段号 Segment 获得段前缀信息  
 \*  
 \* segment: 段号，例如 SEG\_FS 代表 FS 段寄存器  
 \*  
 \* 返回值：返回获取的段前缀  
 \*/  
char \* CInterpretHandler::GetSegStr(int Segment)  // \*  
{  
  static char segstr\[10\] = "";  
  memset(segstr, 0, 10);  
  if (Segment == SEG\_FS)  
    strcpy\_s(segstr, 10, "fs:");  
  else if (Segment == SEG\_GS)  
    strcpy\_s(segstr, 10, "gs:");  
  return segstr;  
}  
  
  
/\*\*  
 \* 虚拟机在执行时，并不会破坏真实堆栈中保存的寄存器，而是将真正堆栈中的寄存器值  
 \* 复制至 VMContext 中，VMContext 即为虚拟机上下文，以后，虚拟机用到的寄存器值都从  
 \* 其中取得  
 \*/  
  
/\*\*  
 \* 此函数的使用即是将堆栈中的寄存器复制至 VMContext 上下文中以便虚拟机使用，虚拟机将  
 \* 这种操作称为堆栈平衡  
 \*  
 \* table: 保存 vBegin 虚拟指令的相关信息  
 \* asmtext：保存完成操作的汇编代码  
 \* len：asmtext 缓存的长度  
 \*  
 \* 返回值：只返回 FALSE 值  
 \*/  
BOOL CInterpretHandler::InterpretvBegin(VMTable \* table, char \* asmtext, int len)  // \*  
{  
  int s\_reg\[8\] = {RT\_EFlag, RT\_Ebp, RT\_Edi, RT\_Esi, RT\_Edx, RT\_Ecx, RT\_Ebx, RT\_Eax};  
  
  // ebp -> 指向真正的堆栈  
  // edi -> 指向 VMContext  
  
  // 将真实堆栈中保存的寄存器值复制至 VMContext 虚拟机上下文中  
  for (int i = 0; i < 8; i++)  
  {    
    // 先将堆栈中的寄存器值复制至 eax 中  
    sprintf\_s(asmtext, len, "%smov eax,dword ptr \[ebp\]\\n", asmtext);  
    // 再将 eax 的值复制到 VMContext 的某个寄存器中  
    sprintf\_s(asmtext, len,"%smov \[edi+%02X\],eax\\n", asmtext, GetRegisterOffset(s\_reg\[i\]));  
      
    sprintf\_s(asmtext, len, "%sadd ebp,4\\n", asmtext);  
  }  
  sprintf\_s(asmtext, len, "%sadd ebp,4\\n", asmtext);  
  
  // 将 ebp(即真正 esp)保存到 VMContext 的 esp  
  sprintf\_s(asmtext, len, "%smov \[edi+%02x\],ebp\\n", asmtext, GetRegisterOffset(RT\_Esp));  
  
  return FALSE;  
}  
  
/\*\*  
 \* 为指令 vtoReal 生成汇编代码  
 \*  
 \* vtoReal：用于跳转至真实指令  
 \*  
 \* table：包含 vtoReal 指令的信息  
 \* asmtext：保存生成的汇编代码  
 \* len: asmtext 缓存的长度  
 \*  
 \* 返回值：只返回 FALSE  
 \*/  
BOOL CInterpretHandler::InterpretvtoReal(VMTable \* table, char \* asmtext, int len)  // \*  
{  
  int s\_reg\[9\] = {RT\_Esp, RT\_EFlag, RT\_Ebp, RT\_Edi, RT\_Esi, RT\_Edx, RT\_Ecx, RT\_Ebx, RT\_Eax};  
  
  // 模拟 pushimm32 xxxx 指令，其中 xxxx 是真实指令的地址  
  
  // 将真实指令的地址复制至 eax 中  
  sprintf\_s(asmtext, len, "%smov eax,dword ptr \[esi\]\\n", asmtext);  
  sprintf\_s(asmtext, len, "%sadd esi,4\\n", asmtext);  
  // 在真实堆栈中分配 4 个字节并将真实指令的地址复制进去  
  sprintf\_s(asmtext, len, "%ssub ebp,4\\n", asmtext);  
  sprintf\_s(asmtext, len, "%smov dword ptr \[ebp\],eax\\n", asmtext);  
  
  // 将 ebp(即真正 esp)保存到 VMContext 的 esp 中  
  sprintf\_s(asmtext, len, "%smov \[edi+%02x\],ebp\\n", asmtext, GetRegisterOffset(RT\_Esp));  
  
  // 将 VMContext 寄存器中的值压入栈  
  for (int i = 0; i < 9; i++)  
  {  
    sprintf\_s(asmtext, len, "%spush \[edi+%02X\]\\n", asmtext, GetRegisterOffset(s\_reg\[i\]));  
  }  
  // 再将压入栈的值复制至相应寄存器中  
  sprintf\_s(asmtext, len, "%spop eax\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spop ebx\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spop ecx\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spop edx\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spop esi\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spop edi\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spop ebp\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spopfd\\n",   asmtext);  
  sprintf\_s(asmtext, len, "%spop esp\\n", asmtext);  
  
  // 至此，在堆栈中保存了真实的地址，并且寄存器环境也恢复了，所以，现在已组成一个调用环境了  
  
  // 返回，跳转到真正的地址  
  sprintf\_s(asmtext, len, "%sretn\\n", asmtext);  
  return FALSE;  
}  
  
/\*\*  
 \* 为指令 push 生成相应的汇编代码  
 \*  
 \* table: 存放 push 指令的信息，根据这些信息可以为相应虚拟机生成汇编代码  
 \* asmtext: 保存生成的汇编代码  
 \* len：asmtext 缓存的长度  
 \*  
 \* 返回值：只返回 TRUE  
 \*  
 \* 此函数生成的代码类似如下：  
 \* mov  eax,dword ptr \[esp\]  
 \* sub  ebp,4  
 \* 内存数  
 \* mov  eax,\[eax\]  
 \* mov  word ptr \[ebp\],ax  
 \*/  
BOOL CInterpretHandler::InterpretPush(VMTable \* table, char \* asmtext, int len)    // \*  
{  
  char scalestr\[6\] = {0};  
  int sizeidx = 0;  
  
  // 根据操作数位数设置修饰符  
  
  // 8 位操作数  
  if (table->bitnum\[0\] == 8)  
  {  
    sizeidx = 0;  
    strcpy\_s(scalestr, 6, "byte");  
  }  
  // 16 位操作数  
  else if (table->bitnum\[0\] == 16)  
  {  
    sizeidx = 1;  
    strcpy\_s(scalestr, 6, "word");  
  }  
  // 32 位操作数  
  else if (table->bitnum\[0\] == 32)  
  {  
    sizeidx = 2;  
    strcpy\_s(scalestr, 6, "dword");  
  }  
  
  // 从堆栈中分配空间用于存放操作数  
  sprintf\_s(asmtext, len, "%ssub ebp,%d\\n", asmtext, table->bitnum\[0\] / 8);  
  
  // 至此，将要将该数据放到堆栈中以模拟压栈过程  
  // 对于该数有两种情况：  
  // 1、如果该数不是一个内存数，则该数已经被放到 EAX 寄存器中了  
  // 2、如果该数是个内存数，则 EAX 寄存器中存放的不是该数，而是该数所在的偏移量  
  if (table->optype\[0\] == MEMTYPE)  
  {  
    // 得到该内存数的值  
    sprintf\_s(asmtext, len, "%smov %s,%s ptr %s\[eax\]\\n", asmtext, vregname\[sizeidx\]\[RT\_Eax\], scalestr, GetSegStr(table->Segment));  
  }  
  // 将内存数复制到堆栈中完成压栈操作  
  sprintf\_s(asmtext, len, "%smov %s ptr \[ebp\],%s\\n", asmtext, scalestr, vregname\[sizeidx\]\[RT\_Eax\]);  
  return TRUE;  
}  
  
  
/\*\*  
 \* 为指令 pop 生成相应的汇编代码  
 \*  
 \* table: 存放 pop 指令的信息，根据这些信息可以为相应虚拟机生成汇编代码  
 \* asmtext: 保存生成的汇编代码  
 \* len：asmtext 缓存的长度  
 \*  
 \* 返回值：只返回 TRUE  
 \*  
 \* 此函数生成的代码类似如下：  
 \*  mov  eax,dword ptr \[ebp\]  
 \*  add  ebp,4  
 \*  mov  eax,\[eax\]  
 \*/  
BOOL CInterpretHandler::InterpretPop(VMTable \* table, char \* asmtext, int len)  // \*  
{  
  char scalestr\[6\] = {0};  
  int sizeidx = 0;  
  
  // 根据操作数位数设置修饰符  
  
  // 8 位操作数  
  if (table->bitnum\[0\] == 8)  
  {  
    sizeidx = 0;  
    strcpy\_s(scalestr, 6, "byte");  
  }  
  else if ( table->bitnum\[0\] == 16 )  
  {  
    sizeidx = 1;  
    strcpy\_s(scalestr, 6, "word");  
  }  
  else if( table->bitnum\[0\] == 32 )  
  {  
    sizeidx = 2;  
    strcpy\_s(scalestr, 6, "dword");  
  }  
  
  // 得到堆栈 ebp( ebp -> 真实堆栈 )的值  
  sprintf\_s(asmtext, len, "%smov %s,%s ptr \[ebp\]\\n", asmtext, vregname\[sizeidx\]\[RT\_Ecx\], scalestr);  
  // 释放操作数占用的空间  
  sprintf\_s(asmtext, len, "%sadd ebp,%d\\n", asmtext, table->bitnum\[0\] / 8);  
  
  
  
  // 如果是内存数  
  if (table->optype\[0\] == MEMTYPE)  
  {  
    // 得到内存数的值  
    sprintf\_s(asmtext, len, "%smov %s,%s ptr %s\[eax\]\\n", asmtext, vregname\[sizeidx\]\[RT\_Eax\], scalestr, GetSegStr(table->Segment));  
  }  
  
  // 将值给 mem(只有 mem，因为无论是 reg 还是 mem 在 vm 中都是内存地址)  
  sprintf\_s(asmtext, len, "%smov eax,ecx\\n", asmtext);  
  
  // 问题：对于上面判断是否内存数的操作似乎是多余的，或说是不对的，个人理解应把代码改为如下：  
  // if (table->optype\[0\] == MEMTYPE)  
  // {  
    // sprintf\_s(asmtext, len, "%smov%s ptr %s\[eax\],%s\\n", asmtext, scalestr, GetSegStr(table->Segment), vregname\[sizeidx\]\[RT\_Ecx\]);  
  // }  
  // else  
  // {  
    // sprintf\_s(asmtext, len, "%smov eax,ecx\\n", asmtext);  
  // }  
  
  return TRUE;  
}  
  
/\*\*  
 \* 为指令 pushfd 生成相应的汇编代码  
 \*  
 \* table: 存放 pushfd 指令的信息，根据这些信息可以为相应虚拟机生成汇编代码  
 \* asmtext: 保存生成的汇编代码  
 \* len：asmtext 缓存的长度  
 \*  
 \* 返回值：只返回 TRUE  
 \*/  
BOOL CInterpretHandler::InterpretPushfd(VMTable \* table, char \* asmtext, int len)  // \*  
{  
  // 在堆栈中分配 4 字节空间用于存放标志寄存器值  
  sprintf\_s(asmtext, len, "%ssub ebp,4\\n", asmtext);  
  // 将 VMContext 上下文中 EFlag 标志寄存器值放入 eax 寄存器  
  sprintf\_s(asmtext, len, "%smov eax,\[edi+%02x\]\\n", asmtext, GetRegisterOffset(RT\_EFlag));  
  // 再将其放入堆栈以实现压栈  
  sprintf\_s(asmtext, len, "%smov dword ptr \[ebp\],eax\\n", asmtext);  
  return TRUE;  
}  
  
/\*\*  
 \* 为指令 popfd 生成相应的汇编代码  
 \*  
 \* table: 存放 popfd 指令的信息，根据这些信息可以为相应虚拟机生成汇编代码  
 \* asmtext: 保存生成的汇编代码  
 \* len：asmtext 缓存的长度  
 \*  
 \* 返回值：只返回 TRUE  
 \*/  
BOOL CInterpretHandler::InterpretPopfd(VMTable \* table, char \* asmtext, int len)  // \*  
{  
  // 将堆栈中的 EFLAG 寄存器值复制到 eax 寄存器中  
  sprintf\_s(asmtext, len, "%smov eax,dword ptr \[ebp\]\\n", asmtext);  
  // 释放堆栈中占用的 4 字节空间  
  sprintf\_s(asmtext, len, "%sadd ebp,4\\n", asmtext);  
  // 再将 eax 寄存器中的 EFLAG 值复制到 VMContext 上下文中实现出栈  
  sprintf\_s(asmtext, len, "%smov \[edi+%02x\],eax\\n", asmtext, GetRegisterOffset(RT\_EFlag));  
  return TRUE;  
}  
  
  
/\*\*  
 \* 为指令 pushad 生成相应的汇编代码  
 \*  
 \* table: 存放 pushad 指令的信息，根据这些信息可以为相应虚拟机生成汇编代码  
 \* asmtext: 保存生成的汇编代码  
 \* len：asmtext 缓存的长度  
 \*  
 \* 注：pushad 指令用于实现将多个通用寄存器压入堆栈中  
 \*  
 \* 返回值：只返回 TRUE  
 \*/  
BOOL CInterpretHandler::InterpretPushad(VMTable \* table, char \* asmtext, int len)  // \*  
{  
  // 循环压栈通用寄存器  
  for (int i = 0; i < 8; i++)  
  {  
    // 为 i 寄存器分配堆栈空间  
    sprintf\_s(asmtext, len, "%ssub ebp,4\\n", asmtext);  
    // 再将 VMContext 上下文中寄存器值复制到 eax 中  
    sprintf\_s(asmtext, len, "%smov eax,\[edi+%02x\]\\n", asmtext, GetRegisterOffset(i));  
    // 最后再将 eax 复制到堆栈中以完成压栈工作  
    sprintf\_s(asmtext, len, "%smov dword ptr \[ebp\],eax\\n", asmtext);  
  }  
  return TRUE;  
}  
  
/\*\*  
 \* 为指令 popad 生成相应的汇编代码  
 \*  
 \* table: 存放 popad 指令的信息，根据这些信息可以为相应虚拟机生成汇编代码  
 \* asmtext: 保存生成的汇编代码  
 \* len：asmtext 缓存的长度  
 \*  
 \* 注：popad 指令用于实现将多个通用寄存器弹出堆栈  
 \*  
 \* 返回值：只返回 TRUE  
 \*/  
BOOL CInterpretHandler::InterpretPopad(VMTable \* table, char \* asmtext, int len)  // \*  
{  
  // 可见 InterpretPushad()操作  
  for (int i = 8; i > 0; i--)  
  {  
    sprintf\_s(asmtext, len, "%smov eax,dword ptr \[ebp\]\\n", asmtext);  
    sprintf\_s(asmtext, len, "%sadd ebp,4\\n", asmtext);  
    sprintf\_s(asmtext, len, "%smov \[edi+%02x\],eax\\n", asmtext, GetRegisterOffset(i));  
  }  
  return TRUE;  
}  
  
/\*\*  
 \* 为指令 enter 生成相应的汇编代码  
 \*  
 \* table: 存放 enter 指令的信息，根据这些信息可以为相应虚拟机生成汇编代码  
 \* asmtext: 保存生成的汇编代码  
 \* len：asmtext 缓存的长度  
 \*  
 \* 返回值：只返回 TRUE  
 \*  
 \* 注：enter 指令用于操作子过程栈框架，代码如下：  
 \*    push ebp  
 \*    mov ebp,esp  
 \*    sub esp,(L - 1) \* 4  ; L > 0 才有这步操作，用来存储嵌套的 L - 1 个子过程的栈框架指针（注意是指针）  
 \*    push ebp ; 当前过程的栈框架指针  
 \*    sub esp,N  
 \*/  
BOOL CInterpretHandler::InterpretEnter(VMTable \* table, char \* asmtext, int len)  // \*  
{  
  // 模拟 push ebp  
  
  // 将 VMContext 上下文中 ebp 寄存器的值复制到 edx 中  
  sprintf\_s(asmtext, len, "%smov edx,\[edi+%02x\]\\n", asmtext, GetRegisterOffset(RT\_Ebp));  
  // 然后，在真实堆栈中分配 4 字节空间用于放置 ebp 的值  
  sprintf\_s(asmtext, len, "%ssub ebp,4\\n", asmtext);  
  sprintf\_s(asmtext, len, "%smov dword ptr \[ebp\],edx\\n", asmtext);  
  
  // 将真实堆栈的 esp 值复制到 edx 中，并保存到 VMContext 上下文的 esp 中  
  sprintf\_s(asmtext, len, "%smov edx,ebp\\n", asmtext);  
  sprintf\_s(asmtext, len, "%smov \[edi+%02x\],edx\\n", asmtext, GetRegisterOffset(RT\_Esp));  
  
  // 至此，ebp 指向的就是真实堆栈，所以是不需要模拟 mov ebp,esp 的  
    
  // 模拟 sub esp,(L - 1) \* 4  ; L > 0 才有这步操作  
  
  // 将真实堆栈 esp 的值复制到 edx 中  
  sprintf\_s(asmtext, len, "%smov edx,ebp\\n", asmtext);  
  // ecx = ecx \* 4  
  // ecx 寄存器中存放的是 L 嵌套层数  
  sprintf\_s(asmtext, len, "%slea ecx,\[ecx\*4\]\\n", asmtext);  
  // 在真实堆栈中分配 (L - 1) \* 4 的空间存放子过程的栈框架信息  
  sprintf\_s(asmtext, len, "%ssub edx,ecx\\n", asmtext);  
  // 再减 4  
  sprintf\_s(asmtext, len, "%ssub edx,4\\n", asmtext);  
  // 检测 ecx 是否为 0  
  sprintf\_s(asmtext, len, "%stest ecx,ecx\\n", asmtext);  
  // 不为 0 则意味着确实需要在堆栈中分空栈框架空间，即 ebp = ebp - edx  
  sprintf\_s(asmtext, len, "%scmovne ebp,edx\\n", asmtext);  
  
  // 模拟 push ebp  
  
  sprintf\_s(asmtext, len, "%smov edx,\[edi+%02x\]\\n", asmtext, GetRegisterOffset(RT\_Ebp));  
  sprintf\_s(asmtext, len, "%ssub ebp,4\\n", asmtext);  
  sprintf\_s(asmtext, len, "%smov dword ptr \[ebp\],edx\\n", asmtext);  
  
  // 模拟 sub esp,N  
  
  sprintf\_s(asmtext, len, "%ssub ebp,eax\\n", asmtext);  
  return TRUE;  
}  
  
/\*\*  
 \* 为指令 leave 生成相应的汇编代码  
 \*  
 \* table: 存放 leave 指令的信息，根据这些信息可以为相应虚拟机生成汇编代码  
 \* asmtext: 保存生成的汇编代码  
 \* len：asmtext 缓存的长度  
 \*  
 \* 返回值：只返回 TRUE  
 \*  
 \* 注：enter 指令用于操作子过程栈框架，代码如下：  
 \*    mov ebp,esp  
 \*    pop ebp  
 \*/  
BOOL CInterpretHandler::InterpretLeave(VMTable \* table, char \* asmtext, int len)  // \*  
{  
  // 因为 ebp 已指向真实堆栈，所以是不需要模拟 mov ebp,esp 指令的  
  
  // 先将真实堆栈 esp 的值保存到 VMContext 上下文中  
  sprintf\_s(asmtext, len, "%smov edx,ebp\\n", asmtext);  
  sprintf\_s(asmtext, len, "%smov \[edi+%02x\],edx\\n", asmtext,GetRegisterOffset(RT\_Esp));  
  
  // 模拟 pop ebp  
  
  // 将真实堆栈中的值复制到 eax 中  
  sprintf\_s(asmtext, len, "%smov eax,\[ebp\]\\n", asmtext);  
  // 释放堆栈占用的空间  
  sprintf\_s(asmtext,len, "%sadd ebp,4\\n", asmtext);  
  // 将 eax 复制到 VMContext 上下文的 ebp 中  
  sprintf\_s(asmtext,len, "%smov \[edi+%02x\],eax\\n", asmtext, GetRegisterOffset(RT\_Ebp));  
  return TRUE;  
}  
  
/\*\*  
 \* 为指令 jmp 生成相应的汇编代码  
 \*  
 \* table: 存放 jmp 指令的信息，根据这些信息可以为相应虚拟机生成汇编代码  
 \* asmtext: 保存生成的汇编代码  
 \* len：asmtext 缓存的长度  
 \*  
 \* 返回值：只返回 FALSE  
 \*/  
BOOL CInterpretHandler::InterpretJMP(VMTable \* table, char \* asmtext, int len)  // \*  
{  
  // 将跳转地址复制到 esi，改变 esi 就相当于改变执行流程  
  sprintf\_s(asmtext, len, "%smov esi,eax\\n", asmtext);  
  sprintf\_s(asmtext, len, "%sadd esp,4\\n", asmtext);  
  return FALSE;  
}  
  
/\*\*  
 \* 为指令 jcxz 生成相应的汇编代码  
 \*  
 \* table: 存放 jcxz 指令的信息，根据这些信息可以为相应虚拟机生成汇编代码  
 \* asmtext: 保存生成的汇编代码  
 \* len：asmtext 缓存的长度  
 \*  
 \* 返回值：只返回 FALSE  
 \*  
 \* 注：jcxz 指令，该指令实现当寄存器 CX 的值等于 0 时转移到标号,否则顺序执行  
 \*/  
BOOL CInterpretHandler::InterpretJCXZ(VMTable \* table, char \* asmtext, int len)    // \*  
{  
  // 检测 ecx 是否为 0  
  sprintf\_s(asmtext, len, "%stest ecx,ecx\\n", asmtext);  
  
  // 如果为 0，则将 eax 复制到 esi 中，由于 eax 中存放的是跳转指令地址，所以，将  
  // esi 的值改变为跳转指令地址，则相当于改变了虚拟机的执行流程  
  sprintf\_s(asmtext, len, "%sCMOVZ esi,eax\\n", asmtext);  
  //   
  sprintf\_s(asmtext, len, "%sadd esp,4\\n", asmtext);  
  return FALSE;  
}  
  
/\*\*  
 \* 为条件跳转指令 jcc 生成相应的汇编代码  
 \*  
 \* table: 存放 jcc 指令的信息，根据这些信息可以为相应虚拟机生成汇编代码  
 \* asmtext: 保存生成的汇编代码  
 \* len：asmtext 缓存的长度  
 \*  
 \* 返回值：只返回 FALSE  
 \*/  
BOOL CInterpretHandler::InterpretJCC(VMTable \* table, char \* asmtext, int len)  // \*  
{  
  // 先根据指令得到跳转的条件后缀  
  char strPostfix\[16\] = {0};  
  strcpy\_s(strPostfix, 16, &table->strInstruction\[1\]);  
  // 如果条件成立，则将 esp 的地址复制到 esi 中以改变流程，否则不复制，即流程不改变  
  sprintf\_s(asmtext, len, "%scmov%s esi,\[esp\]\\n", asmtext, strPostfix);  
  sprintf\_s(asmtext,len, "%sadd esp,4\\n", asmtext);  
  return FALSE;  
}  
  
/\*\*  
 \* 为指令 loope 生成相应的汇编代码  
 \*  
 \* table: 存放 loope 指令的信息，根据这些信息可以为相应虚拟机生成汇编代码  
 \* asmtext: 保存生成的汇编代码  
 \* len：asmtext 缓存的长度  
 \*  
 \* 返回值：只返回 FALSE  
 \*/  
BOOL CInterpretHandler::InterpretLoope(VMTable \* table, char \* asmtext, int len)  // \*  
{  
  // 将标志寄存器压入堆栈中  
  sprintf\_s(asmtext, len, "%spushfd\\n", asmtext);  
  // 测试 ecx 是否为 0  
  sprintf\_s(asmtext, len, "%stest ecx,ecx\\n", asmtext);  
  // 如果不为 0，则说明需要继续循环，则将循环指令地址复制到 edx 中  
  sprintf\_s(asmtext, len, "%scmovne edx,eax\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spopfd\\n", asmtext);  
  sprintf\_s(asmtext, len, "%scmovne edx,esi\\n", asmtext);  
  
  sprintf\_s(asmtext, len, "%scmove edx,eax\\n", asmtext);  
  // 释放堆栈  
  sprintf\_s(asmtext, len, "%sadd ebp,4\\n", asmtext);  
  return FALSE;  
}  
  
/\*\*  
 \* 为指令 retn 生成相应的汇编代码  
 \*  
 \* table: 存放 retn 指令的信息，根据这些信息可以为相应虚拟机生成汇编代码  
 \* asmtext: 保存生成的汇编代码  
 \* len：asmtext 缓存的长度  
 \*  
 \* 返回值：只返回 FALSE  
 \*  
 \* 注：retn 在虚拟机里被认为是一个退出函数  
 \*/  
BOOL CInterpretHandler::InterpretRetn(VMTable \* table, char \* asmtext, int len)    // \*  
{  
  int s\_reg\[9\] = {RT\_Esp, RT\_EFlag, RT\_Ebp, RT\_Edi, RT\_Esi, RT\_Edx, RT\_Ecx, RT\_Ebx, RT\_Eax};  
  
  // 如果指令带有一个操个数，即 retn XX  
  if (table->OperandNum == 1)  
  {  
    // 先将返回地址复制到 edx 中  
    // 问题：此语句似乎是错误的，应该如下：  
    //     sprintf\_s(asmtext, len, "%smov edx,\[ebp\]\\n", asmtext);  
    sprintf\_s(asmtext, len, "%smov edx,ebp\\n", asmtext);  
  
    // 释放堆栈占用的空间 XX  
    sprintf\_s(asmtext, len, "%sadd ebp,eax\\n", asmtext);  
  
    // 再将返回地址放入堆栈中  
    sprintf\_s(asmtext, len, "%smov \[ebp\],edx\\n", asmtext);  
    // 将 ebp(即真实 esp)保存到 VMContext 上下文的 esp 中  
    sprintf\_s(asmtext, len, "%smov \[edi+%02x\],ebp\\n", asmtext, GetRegisterOffset(RT\_Esp));  
  }  
  // 将保存在 VMContext 上下文的寄存器压入堆栈  
  for (int i = 0; i < 9; i++)  
  {  
    sprintf\_s(asmtext, len, "%spush \[edi+%02X\]\\n", asmtext, GetRegisterOffset(s\_reg\[i\]));  
  }  
  // 再将其恢复到各个寄存器中  
  sprintf\_s(asmtext, len, "%spop eax\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spop ebx\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spop ecx\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spop edx\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spop esi\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spop edi\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spop ebp\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spopfd\\n", asmtext);  
  // 将 esp 恢复到真实堆栈  
  sprintf\_s(asmtext, len, "%spop esp\\n", asmtext);  
  
  // 正式返回  
  sprintf\_s(asmtext, len, "%sretn\\n", asmtext);  
  return FALSE;  
}  
  
/\*\*  
 \* 为指令 call 生成相应的汇编代码  
 \*  
 \* table: 存放 call 指令的信息，根据这些信息可以为相应虚拟机生成汇编代码  
 \* asmtext: 保存生成的汇编代码  
 \* len：asmtext 缓存的长度  
 \*  
 \* 返回值：只返回 FALSE  
 \*  
 \* 注：call anotherfunc 调用子函数，会将控制权移交给另处的代码，这些代码是不受  
 \*     虚拟机控制的，所以碰到 call 指令，必须退出虚拟机， 让子函数在真实 CPU 中执  
 \*     行后再交回给虚拟机执行一下指令  
 \*/  
BOOL CInterpretHandler::InterpretCall(VMTable \* table, char \* asmtext, int len)    // \*  
{  
  int s\_reg\[9\] = {RT\_Esp, RT\_EFlag, RT\_Ebp, RT\_Edi, RT\_Esi, RT\_Edx, RT\_Ecx, RT\_Ebx, RT\_Eax};  
    
  sprintf\_s(asmtext, len, "%smov edx,\[esi\]\\n", asmtext);  
  // 在堆栈中分配 4 字节空间  
  sprintf\_s(asmtext, len, "%ssub ebp,4\\n", asmtext);  
  // 将 call 的返回地址压入堆栈中  
  sprintf\_s(asmtext, len, "%smov \[ebp\],edx\\n", asmtext);  
  // 再分配 4 字节空间  
  sprintf\_s(asmtext, len, "%ssub ebp,4\\n", asmtext);  
  
  // 如果 call 调用地址是一个内存数  
  if (table->optype\[0\] == MEMTYPE)  
  {  
    sprintf\_s(asmtext, len, "%smov eax,dword ptr \[eax\]\\n", asmtext);  
  }  
  // 将 call 的地址压入堆栈中  
  sprintf\_s(asmtext, len, "%smov \[ebp\],eax\\n", asmtext);  
  
  // 将 ebp(即真正 esp)保存到 VMContext 上下文的 esp 中  
  sprintf\_s(asmtext, len, "%smov \[edi+%02x\],ebp\\n", asmtext, GetRegisterOffset(RT\_Esp));  
  
  // 压入保存在 VMContext 的寄存器值  
  for (int i = 0; i < 9; i++)  
  {  
    sprintf\_s(asmtext, len, "%spush \[edi+%02X\]\\n", asmtext, GetRegisterOffset(s\_reg\[i\]));  
  }  
  // 再将各寄存器值弹出，恢复到各寄存器中，以恢复环境  
  sprintf\_s(asmtext, len, "%spop eax\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spop ebx\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spop ecx\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spop edx\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spop esi\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spop edi\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spop ebp\\n", asmtext);  
  sprintf\_s(asmtext, len, "%spopfd\\n", asmtext);  
  // 恢复到真实的堆栈  
  sprintf\_s(asmtext, len, "%spop esp\\n", asmtext);  
  
  // 返回，直接回到 call 的地址执行  
  sprintf\_s(asmtext, len, "%sretn\\n", asmtext);  
  
  return FALSE;  
}  
  
/\*\*  
 \* 为指令 saveesp 生成相应的汇编代码，用于保存堆栈  
 \*  
 \* table: 存放 saveesp 指令的信息，根据这些信息可以为相应虚拟机生成汇编代码  
 \* asmtext: 保存生成的汇编代码  
 \* len：asmtext 缓存的长度  
 \*  
 \* 返回值：只返回 FALSE  
 \*/  
BOOL CInterpretHandler::InterpretSaveEsp(VMTable \* table, char \* asmtext, int len)  // \*  
{  
  // 将 ebp(即真正 esp)保存到 VMContext 上下文的 esp 中  
  sprintf\_s(asmtext, len, "%smov \[edi+%02x\],ebp\\n", asmtext, GetRegisterOffset(RT\_Esp));  
  return FALSE;  
}  
  
/\*\*  
 \* 为指令 restoreesp 生成相应的汇编代码，用于恢复堆栈  
 \*  
 \* table: 存放 restoreesp 指令的信息，根据这些信息可以为相应虚拟机生成汇编代码  
 \* asmtext: 保存生成的汇编代码  
 \* len：asmtext 缓存的长度  
 \*  
 \* 返回值：只返回 FALSE  
 \*/  
BOOL CInterpretHandler::InterpretRestoreEsp(VMTable \* table, char \* asmtext, int len)  // \*  
{  
  // 将 VMContext 上下文中 esp 恢复到 ebp 真实堆栈指针中  
  sprintf\_s(asmtext, len, "%smov ebp,\[edi+%02x\]\\n", asmtext, GetRegisterOffset(RT\_Esp));  
  return FALSE;  
}  
  
  
// ------------------------------------- vmserv.cpp ---------------------------------------------  
// 此文件定义了如《汇编指令-虚拟机指令》对应表 vmtable 等信息  
// 这里，只摘取了其中一部分  
#include "stdafx.h"  
#include "VCommand.h"  
#include "asm\\disasm.h"  
#include "vmdisasm.h"  
  
/\*\*  
 \* 寄存器名  
 \*/  
const char \* vregname\[3\]\[14\] =   
{  
  { "AL", "CL", "DL", "BL", "AH", "CH", "DH", "BH", "CS","DS","ES","FS","GS","SS"  },  
  { "AX", "CX", "DX", "BX", "SP", "BP", "SI", "DI", "CS","DS","ES","FS","GS","SS" },  
  { "EAX","ECX","EDX","EBX","ESP","EBP","ESI","EDI","CS","DS","ES","FS","GS","SS" }   
};  
  
/\*\*  
 \* 可用于存放虚拟指令操作数的寄存器  
 \*/  
const char ArgReg\[3\]\[4\] = {"EAX","ECX","EDX"};  
  
/\*\*  
 \* 虚拟机有两部分的指令：  
 \* 1、是需要手工编写生成的指令  
 \* 2、是可以由程序动态分析生成的指令  
 \*/  
  
  
/\*\*  
 \* 需要手工编写生成的核心指令  
 \*/  
VM\_DName vm\_dname\[COREHANDLERLEN\] =   
{  
  {"VStartVM", VStartVM},    // 启动函数  
  {"DCheckESP", DCheckESP},  // 检查 ESP 是否重复  
  
  {"DPushReg32", DPushReg32},  // dpush eax  
  {"DPushImm32", DPushImm32},  // dpush 1234  
  {"DPushMem32", DPushMem32},  // dpush 1234  
  {"DPopReg32", DPopReg32},  // dpop  ebx  
  {"DFree", DFree},      // daddesp 4  
  {"", NULL},  
  
  {"", NULL},  
};  
  
/\*\*  
 \* 数组包含了虚拟机指令与相应的普通汇编指令的信息，根据这些信息，虚拟机  
 \* 可以为虚拟机指令生成相应的汇编代码以代替原来对应的普通汇编指令  
 \*/  
VMTable vmtable\[VMTABLEMAXLEN\] =   
{  
  // 信息格式如下：  
  // 虚拟机指令名、相对的汇编指令、操作数个数、段前缀、操作数类型、操作数位数、执行指令时需要使用的寄存器、执行指令后需要保存的寄存器、第 2 个寄存器是否恢复(一般为 0 不恢复)  
  
  {"VBEGIN","vBegin",0, SEG\_UNDEF, NONETYPE, NONETYPE,0,0, NONE,NONE,NONE,NONE,    NONE,NONE,NONE,NONE },  
  {"VTOREAL","vtoReal",1, SEG\_UNDEF, IMMTYPE, NONETYPE,32,0, NONE,NONE,NONE,NONE,    NONE,NONE,NONE,NONE },  
  {"VRETN","RETN",0, SEG\_UNDEF, NONETYPE, NONETYPE,0,0, NONE,NONE,NONE,NONE,    NONE,NONE,NONE,NONE },  
  {"VRETNIMM32","RETN",1, SEG\_UNDEF, IMMTYPE, NONETYPE,32,0, NONE,NONE,NONE,NONE,    NONE,NONE,NONE,NONE },  
  // 保存堆栈  
  {"VSAVEESP","SAVEESP",0, SEG\_UNDEF, IMMTYPE, NONETYPE,32,0, NONE,NONE,NONE,NONE,    NONE,NONE,NONE,NONE },  
  // 恢复堆栈  
  {"VRESTOREESP","RESTOREESP",0, SEG\_UNDEF, IMMTYPE, NONETYPE,32,0, NONE,NONE,NONE,NONE,    NONE,NONE,NONE,NONE },  
  
  // 堆栈操作指令  
  {"VPUSH\_IMM32","PUSH",1, SEG\_UNDEF, IMMTYPE, NONETYPE,32,0, NONE,NONE,NONE,NONE,    NONE,NONE,NONE,NONE },  
  {"VPUSH\_REG16","PUSH",1, SEG\_UNDEF, REGTYPE, NONETYPE,16,0, NONE,NONE,NONE,NONE,    NONE,NONE,NONE,NONE },  
};  
  
  
// ----------------------------------- InterpretHandler.h ---------------------------------------  
#pragma once  
#include "vmdisasm.h"  
  
class CInterpretHandler  
{  
public:  
  CInterpretHandler(void);  
public:  
  ~CInterpretHandler(void);  
private:  
  // 寄存器数组索引  
  // 前 15 个元素是 15 个不超 15 的随机数  
  int      m\_RegisterIdx\[STACKLEN\];  
public:  
  // 初始化  
  BOOL Init();  
public:  
  // 获得寄存器的偏移  
  int GetRegisterOffset(int RegType);  
  // 获得段前缀  
  char\* GetSegStr(int Segment);  
  // 根据结构声称 ASM 字符串  
  BOOL  InterpretASMStr(VMTable\* table,char\* asmtext,int len);  
private:  
  //设置参数  
  void SetArg(VMTable\* table,char\* asmtext,int len);  
  //恢复参数  
  void RestoreArg(VMTable\* table,char\* asmtext,int len);  
  //恢复标志  
  void RestoreFlag(char\* asmtext,int len);  
  //保存标志  
  void SaveFlag(char\* asmtext,int len);  
private:  
  // 首先执行的指令  
  BOOL  InterpretvBegin(VMTable\* table,char\* asmtext,int len);  
  // 跳转到真实指令  
  BOOL  InterpretvtoReal(VMTable\* table,char\* asmtext,int len);  
  // \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*堆栈类\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* //  
  // 解释 push  
  BOOL  InterpretPush(VMTable\* table,char\* asmtext,int len);  
  // 解释 pop  
  BOOL  InterpretPop(VMTable\* table,char\* asmtext,int len);  
  // 解释 pushfd  
  BOOL  InterpretPushfd(VMTable\* table,char\* asmtext,int len);  
  // 解释 popfd  
  BOOL  InterpretPopfd(VMTable\* table,char\* asmtext,int len);  
  // 解释 pushad  
  BOOL  InterpretPushad(VMTable\* table,char\* asmtext,int len);  
  // 解释 popad  
  BOOL  InterpretPopad(VMTable\* table,char\* asmtext,int len);  
  // 解释 enter  
  BOOL  InterpretEnter(VMTable\* table,char\* asmtext,int len);  
  // 解释 leave  
  BOOL  InterpretLeave(VMTable\* table,char\* asmtext,int len);  
  // \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*流程类\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* //  
  // 解释 jmp  
  BOOL  InterpretJMP(VMTable\* table,char\* asmtext,int len);  
  // 解释 jcxz\\jecxz  
  BOOL  InterpretJCXZ(VMTable\* table,char\* asmtext,int len);  
  // 解释 jcc  
  BOOL  InterpretJCC(VMTable\* table,char\* asmtext,int len);  
  // 解释 loope  
  BOOL InterpretLoope(VMTable\* table,char\* asmtext,int len);  
  
  // 解释返回  
  BOOL InterpretRetn(VMTable\* table,char\* asmtext,int len);  
  // 解释子调用  
  BOOL InterpretCall(VMTable\* table,char\* asmtext,int len);  
  
  // 普通指令  
  BOOL CommonInstruction(VMTable\* table,char\* asmtext,int len);  
  // 解释保护堆栈 Handler  
  BOOL InterpretSaveEsp(VMTable\* table,char\* asmtext,int len);  
  // 解释恢复堆栈 Handler  
  BOOL InterpretRestoreEsp(VMTable\* table,char\* asmtext,int len);  
};  
  
  
// ----------------------------------------------- vmdisasm.h -------------------------------------  
  
////////////////////////////////////////////////////////////////////////////////////  
// 新段各部分空间的长度  
  
#define JumpTableLen    (HANDLERMAXLEN \* 4)  
#define CodeEngineLen    (4096 \* 4)      // 虚拟机引擎代码长度  
#define EnterStubAddrLen  (4096)        // 重新进入 VM 的 Stub，长度 1024 字节  
#define VMEnterCodeLen    (4096 \* 2)      // 空间存放重新进入的 vmcode  
  
#define VMCodeLen      (10240 \* 2)      // 2K 的空间存放 VMCode  
#define ASMTEXTLEN      1024        // 汇编文本长度  
  
////////////////////////////////////////////////////////////////////////////////////  
  
//8086 指令字符串长度  
#define X86NAMELEN          32  
  
// 虚拟机指令名的长度限制  
#define VMNAMELEN          32  
  
//Handler 的最大 Handler 长度  
#define CODEMAXLEN          512  
  
// 最上面存放寄存器, 后来的一些就当 vm 堆栈用了  
#define STACKLEN          0x200    // 512  
  
// 寄存器最大个数  
#define REGCOUNT          15  
  
// 命令 Handler 最大个数  
// 伪指令最大限制数  
#define HANDLERMAXLEN        0xFF  
  
//VMTABEL 表的个数  
#define VMTABLEMAXLEN        0x400  
  
// 核心伪指令个数  
#define COREHANDLERLEN        10  
  
// 没有用到寄存器  
#define NONE  -1  
  
// 寄存器类型  
enum RegType  
{  
  RT\_Eax,  
  RT\_Ecx,  
  RT\_Edx,  
  RT\_Ebx,  
  RT\_Esp,  
  RT\_Ebp,  
  RT\_Esi,  
  RT\_Edi,  
  RT\_CS,  
  RT\_DS,  
  RT\_ES,  
  RT\_FS,  
  RT\_GS,  
  RT\_SS,  
  
  // 必须是最后一个(为什么?我忘了)  
  RT\_EFlag,  
  
  RT\_AH = 20,  
  RT\_CH,  
  RT\_DH,  
  RT\_BH,  
};  
  
/\*\*  
 \* 指令操作数类型  
 \*/  
enum optype  
{  
  NONETYPE,  
  // 立即数  
  IMMTYPE,  
  // 寄存器数  
  REGTYPE,  
  // 内存数  
  MEMTYPE,  
  CSTYPE,  
  DSTYPE,  
  ESTYPE,  
  SSTYPE,  
  FSTYPE,  
  GSTYPE,  
};  
  
/\*\*  
 \* 虚拟机指令对应着相应的普通汇编指令，此结构的作用，  
 \* 用于存放虚拟机指令与普通汇编指令之间的相关信息  
 \*/  
struct VMTable  
{  
  char  VMInstrName\[VMNAMELEN\];  // 虚拟机指令名  
  char  strInstruction\[16\];    // 相对的汇编指令  
  int    OperandNum;      // 操作数个数  
  int    Segment;      // 段前缀  
  int    optype\[2\];      // 操作数类型(寄存器,立即数,内存数)  
  int    bitnum\[2\];      // 操作数位数  
  
  // 某些汇编指令在执行过程中，需要根据某些寄存器的值来控制执行  
  
  int    NeedReg\[4\];      // 执行指令时需要使用的寄存器  
  int    SaveReg\[4\];      // 执行指令后需要保存的寄存器  
  BOOL  Reg2Esp;      // 第 2 个寄存器是否恢复,一般为 0 不恢复  
};  
  
/\*\*  
 \* 存放虚拟机指令处理代码信息  
 \*/  
struct VHandler  
{  
  // 虚拟机指令名称，例如：VStartVM 等  
  char  VMInstrName\[VMNAMELEN\];  
  
  // 应该是虚拟机指令相应代码所在的虚拟地址  
  DWORD  VirtualAddress;  
  
  // 指令的汇编代码，或放置编译后的机器码  
  char    AssembleCode\[CODEMAXLEN\];  
  
  // 指令对应的处理代码（函数）长度  
  int     CodeLen;  
  VHandler()  
  {  
    memset(this, 0, sizeof(VHandler));  
  }  
};  
  
/\*\*  
 \* 指令处理代码（即函数）定义  
 \*/  
typedef void (\*HandlerFunc)();  
  
//核心的一些手工 VM 函数名称和对应的函数  
struct VM\_DName  
{  
  char    vm\_dname\[VMNAMELEN\];  
  HandlerFunc  FuncAddr;  
};  
  
extern VM\_DName  vm\_dname\[COREHANDLERLEN\];  
  
//描述一些标准 Handler 的行为表  
extern VMTable  vmtable\[VMTABLEMAXLEN\];  
//寄存器对应表  
extern const char \*vregname\[3\]\[14\];  
extern const char ArgReg\[3\]\[4\];  
  
// 虚拟机为每个指令产生一个索引号，虚拟机的执行是基于一张跳转表，跳转表中  
// 存储每个虚拟机指令处理代码的地址，而当执行某个虚拟机指令时，就是根据虚  
// 拟机指令索引号计算出在跳转表中的下标进行跳转操作的  
  
// 而且，该索引号也将作为虚拟机指令的字节码使用  
  
/\*\*  
 \* 保存虚拟机指令名与对应的索引号信息  
 \*/  
struct VM\_RandomTable  
{  
  char  VMInstrName\[VMNAMELEN\];  // 虚拟机指令名  
  int    idx;  // 对应的索引  
};  
  
/\*\*  
 \* 存放虚拟机字节码指令节点(即字节码)  
 \*/  
struct VCodeNode   
{  
  // 原指令原来所在的地址  
  DWORD  InAddress;  
  
  // 虚拟机指令所在的地址  
  DWORD  VMAddress;  
  
  char  VMInstrName\[VMNAMELEN\];    //VM 命令名称  
  
  // 虚拟机指令的字节码，HexCode\[0\]存放的是指令在跳转表中的索引号  
  BYTE  HexCode\[32\];  
  // 字节码长度  
  int    HexLen;  
  // 指令操作数个数  
  int    OperandNum;  
  
  // 操作数：立即数(寄存器, 立即数, 内存数)  
  int    immconst\[2\];  
  int    bitnum\[2\];          //位数  
  VCodeNode()  
  {  
    InAddress = 0;  
    VMAddress = 0;  
    memset(VMInstrName,0,VMNAMELEN);  
    memset(HexCode,0,32);  
    HexLen = 0;  
    OperandNum = 0;  
    immconst\[0\] = immconst\[1\] = bitnum\[0\] = bitnum\[1\] = 0;  
  }  
};  
  
// 为一个索引数组随机排序.  
void  RandListIdx(int\* idx,int cout); 

// ---------------------------------------- CCodeILFactory.cpp ---------------------------------  
  
// VMPackLib  
  
#include "stdafx.h"  
#include "CCodeILFactory.h"  
  
CCodeILFactory::CCodeILFactory()  
{  
}  
  
/\*\*  
 \* 中间字节码指令工厂初始化，主要是生成某些虚拟机核心指令机器代码，  
 \* 并将该代码复制到 m\_CodeEngine 虚拟内存中  
 \*  
 \* VirtualAddress：虚拟内存相应的虚拟地址  
 \*/  
void CCodeILFactory::Init(DWORD VirtualAddress)    // \*  
{  
  // 计算与虚拟机相关代码的各虚拟地址与空间长度  
  int nJumpTableAddr  = VirtualAddress;  
  int nCodeEngineAddr = nJumpTableAddr + JumpTableLen;  
  int nEnterStubAddr  = nCodeEngineAddr + CodeEngineLen;  
  int mVMEnterStubCodeAddr = nEnterStubAddr + EnterStubAddrLen;  
  int nVMCodeAddr    = mVMEnterStubCodeAddr + VMEnterCodeLen;  
  
  // 为每段代码分配相关的内存空间  
  m\_JumpTable.CreateVirtualMemory(nJumpTableAddr, JumpTableLen);  
  m\_CodeEngine.CreateVirtualMemory(nCodeEngineAddr, CodeEngineLen);  
  m\_EnterStub.CreateVirtualMemory(nEnterStubAddr, EnterStubAddrLen);  
  m\_VMEnterStubCode.CreateVirtualMemory(mVMEnterStubCodeAddr, VMEnterCodeLen);  
  m\_VMCode.CreateVirtualMemory(nVMCodeAddr, VMCodeLen);  
  
  // 将各内存空间信息传递给 VMFactory 虚拟机工厂  
  VMFactory.SetupVirtualMemory(&m\_JumpTable, &m\_CodeEngine, &m\_EnterStub, &m\_VMEnterStubCode, &m\_VMCode);  
  
  // 初始化虚拟机  
  VMFactory.InitVM();  
}  
  
/\*\*  
 \* 反汇编指定代码  
 \*  
 \* CodeList：用于返回反汇编后所得的每个指令  
 \* base\_addr：欲反汇编代码的基地址，一般是某个函数的基址  
 \* VirtualAddress：欲反汇编代码所在的虚拟地址  
 \*/  
void CCodeILFactory::DisasmFunction(list<CodeNode \*> \* CodeList, char \* base\_addr, DWORD VirtualAddress)  // \*  
{  
  // 函数起始地址  
  DWORD Start\_VirtualAddress = VirtualAddress;  
  // 函数结束地址  
  DWORD End\_VirtualAddress = 0;  
  
  // 用于保存反汇编结果  
  t\_disasm da;  
  CodeNode \* code = NULL;  
  int len =0;  
  int KeepNum = 0;  
  
  // 计算出函数的结尾  
  while (TRUE)  
  {  
    code = new CodeNode;  
  
    // 反汇编 base\_addr 的代码  
    len = Disasm(base\_addr, MAXCMDSIZE, VirtualAddress, &da, DISASM\_CODE);  
  
    // 将反汇编结果 da 复制到 disasm 中，并加入到 CodeList 列表中  
    memcpy(&code->disasm, &da, sizeof(t\_disasm));  
    CodeList->push\_back(code);  
  
    // 递减所剩代码长度  
    if (KeepNum >= len && KeepNum != 0 )  
      KeepNum -= len;  
  
    // 如果 da 指令是跳转指令  
    if (IsJumpCmd(&da))  
    {  
      // 并且跳转的目标地址是向前跳转，则计算从当前指令至目标地址的代码长度  
      if (da.jmpconst && da.jmpconst > VirtualAddress)  
      {  
        KeepNum = da.jmpconst - VirtualAddress;  
      }  
    }  
  
    // 如果是 int3 指令，即断点指令，则直接跳出  
    if (\_stricmp(da.result, "int3") == 0)  
    {  
      break;  
    }  
    // 如果找到返回指令  
    if (\_stricmp(da.result, "retn") == 0)  
    {  
      // 并且已没有更多的代码，则说明函数结束  
      if (KeepNum - len <= 0)  
        break;  
    }  
    base\_addr += len;  
    VirtualAddress += len;  
  }  
  End\_VirtualAddress = VirtualAddress;  
  
  // 确定每个指令的类型等信息  
  BOOL NextJMCALL = FALSE;  
  
  list<CodeNode\*>::iterator itr;  
  // 遍历 CodeList 中的每个语句  
  for (itr = CodeList->begin(); itr != CodeList->end(); itr++)  
  {  
    CodeNode \* code = \* itr;  
    t\_disasm t\_da;  
    // 得到语句的反汇编结果  
    memcpy(&t\_da, &code->disasm, sizeof(t\_disasm));  
      
    // 真是一个奇怪的判断  
    if (code)  
    {  
      if (NextJMCALL)  
      {  
        NextJMCALL = FALSE;  
        // 表示 code 指令是 JMP 或 CALL 指令后的指令  
        code->IsJmcNextType = TRUE;  
      }  
      // 如果是跳转指令  
      if (IsJumpCmd(&t\_da))  
      {  
        NextJMCALL = TRUE;  
        // 如果存在跳转地址  
        if (t\_da.jmpconst)  
        {  
          // 跳转的范围在 CodeList 中  
          if (t\_da.jmpconst > Start\_VirtualAddress && t\_da.jmpconst < End\_VirtualAddress)  
          {  
            // 搜索 code 跳转指令跳转的目标  
            CodeNode \* tmpNode = SearchAddrAsList(CodeList, code);  
  
            // 确实找到跳转目标  
            if (tmpNode)  
            {  
              // 标明 tmpNode 指令是从其他地方跳转过来的  
              tmpNode->IsJmcFromType = TRUE;  
            }  
          }  
          else  
          {  
            // 为跳到外部的指令  
            code->IsJmcBeSideType = TRUE;  
          }  
        }  
        // 没有跳转地址，则表示是动态跳转  
        else  
        {  
          code->IsJmcDynamicType = TRUE;  
        }  
      }  
      // 是函数调用指令  
      else if (IsCallCmd(&t\_da))  
      {  
        NextJMCALL = TRUE;  
        code->IsCallType = TRUE;  
      }  
    }  
  }  
}  
  
/\*\*  
 \* 检测 da 中反汇编指令是否是跳转指令  
 \*  
 \* 返回值：是则返回 TRUE，否则 FALSE  
 \*/  
BOOL CCodeILFactory::IsJumpCmd(t\_disasm \* da)    // \*  
{  
  // CALL 不属于跳转指令  
  if ((da->cmdtype & C\_CAL) == C\_CAL)  
  {  
    return FALSE;  
  }  
  if ((da->cmdtype & C\_JMP) == C\_JMP)  
  {  
    return TRUE;  
  }  
  if ((da->cmdtype & C\_JMC) == C\_JMC)  
  {  
    return TRUE;  
  }  
  return FALSE;  
}  
  
/\*\*  
 \* 检测 da 中反汇编指令是否是函数调用指令  
 \*  
 \* 返回值：是则返回 TRUE，否则 FALSE  
 \*/  
BOOL CCodeILFactory::IsCallCmd(t\_disasm \* da)    // \*  
{  
  if ((da->cmdtype & C\_CAL ) == C\_CAL)  
  {  
    return TRUE;  
  }  
  return FALSE;  
}  
  
  
BOOL CCodeILFactory::IsRetnCmd(t\_disasm\* da)  
{  
  char strretn\[32\] = "RETN\\0";  
  \_strlwr\_s(da->result,TEXTLEN);  
  \_strlwr\_s(strretn,32);  
  if( strstr(strretn,da->result) )  
  {  
    return TRUE;  
  }  
  return FALSE;  
}  
  
/\*\*  
 \* 在指令 List 列表中，搜索跳转指令 node 跳转的目标指令  
 \*  
 \* List：指令列表  
 \* node: 跳转指令  
 \*  
 \* 返回值：找到 node 是跳转向 List 中的哪个指令，则返回该目标指令，否则 NULL  
 \*/  
CodeNode \* CCodeILFactory::SearchAddrAsList(list<CodeNode\*> \* List, CodeNode \* node)  // \*  
{  
  // node 是一个跳转指令，现在获取其跳转地址  
  DWORD GotoAddr = node->disasm.jmpconst;  
  DWORD LastAddr = 0;  
  list<CodeNode\*>::iterator itr;  
  // 遍历 List 中的每个语句  
  for (itr = List->begin(); itr != List->end(); itr++)  
  {  
    CodeNode \* code = \* itr;  
    t\_disasm t\_da;  
    // 获取语句反汇编代码  
    memcpy(&t\_da, &code->disasm, sizeof(t\_disasm));  
  
    // 检测 node 是否欲跳转至 t\_da  
    if (GotoAddr == t\_da.ip)  
    {  
      node->IsJmcType = TRUE;  
      return code;  
    }  
    // 否则，如果跳转地址在 t\_da 指令之后，则保存 t\_da 的地址  
    else if (GotoAddr > t\_da.ip)  
    {  
      LastAddr = t\_da.ip;  
    }  
    // 否则，如果跳转地址在 t\_da 指令之前  
    else if (GotoAddr < t\_da.ip)  
    {  
      if (LastAddr)  
      {  
        // 指令跳转地址不明确  
        node->IsJmcUndefineType = TRUE;    
      }  
    }  
  }  
  return NULL;  
}  
  
/\*\*  
 \* 将指定指令列表 CodeList 编译为虚拟机字节码  
 \*  
 \* baseaddr：代码基址  
 \* CodeList：欲编译的指令  
 \* ErrText：返回出错信息  
 \*  
 \* 返回值：成功返回 TRUE，否则 FALSE  
 \*/  
BOOL CCodeILFactory::BuildCode(char \* baseaddr, list<CodeNode\*> \* CodeList, char \* ErrText)    // \*  
{  
  return VMFactory.BuildVMCode(baseaddr, CodeList, ErrText);  
}  
  
  
// ----------------------------------- CCodeILFactory.h ----------------------------------------  
// VMPackLib  
  
#ifndef \_\_CCODEILFACTORY\_\_  
#define \_\_CCODEILFACTORY\_\_  
  
#include <Windows.h>  
#include <stdio.h>  
#include "CVirtualMemory.h"  
#include "CVMFactory.h"  
#include "asm\\disasm.h"  
  
/\*\*  
 \* 中间语言字节码工厂  
 \*/  
class CCodeILFactory  
{  
public:  
  CVirtualMemory m\_JumpTable;    // JMP 表  
  CVirtualMemory m\_CodeEngine;  // VM 指令引擎  
  CVirtualMemory m\_EnterStub;    // 重新进入 vm 的代码  
  CVirtualMemory m\_VMEnterStubCode;  // 重新进入的 vmcode  
  CVirtualMemory m\_VMCode;    // 通常的 vmcode  
public:  
  // 虚拟机工厂  
  CVMFactory VMFactory;  
public:  
  CCodeILFactory();  
public:  
  void  Init(DWORD VirtualAddress);  
public:  
  // 编译汇编代码为 VM 字节码,返回成功失败  
  BOOL  BuildCode(char\* baseaddr,list<CodeNode\*> \*CodeList,char\* ErrText);  
public:  
  
  // 反汇编代码,识别出函数结束的地方.添加到提供的链表中  
  // CodeList : 输出汇编链表  
  // base\_addr: 2 进制代码  
  // VirtualAddress : 虚拟地址  
  void  DisasmFunction(list<CodeNode\*> \*CodeList,char\* base\_addr, DWORD VirtualAddress);  
  
  
private:  
  // 是否是跳转指令  
  BOOL  IsJumpCmd(t\_disasm\* da);  
  // 是否是函数调用指令  
  BOOL  IsCallCmd(t\_disasm\* da);  
  // 是否是返回指令  
  BOOL  IsRetnCmd(t\_disasm\* da);  
  //从链表中搜索一个地址的节点  
  CodeNode\*  SearchAddrAsList( list<CodeNode\*>\* List,CodeNode\* node );  
  
};// END CLASS DEFINITION CCodeILFactory  
  
#endif // \_\_CCODEILFACTORY\_\_  
  
// ------------------------------------------ VCommand.h----------------------------------------  
// 漏发的文件  
// "VMEND"结束标记  
#define    VM\_END          \_\_asm    push    esi \\  
                  \_\_asm    dec     ebp \\  
                  \_\_asm    inc     ebp \\  
                  \_\_asm    dec     esi \\  
                  \_\_asm    inc     esp  
  
#define    INTCC          \_\_asm    \_emit  0xCC  
// 进入虚拟机  
void VStartVM();  
//检测堆栈是否覆盖  
void DCheckESP();  
  
//带 D 的为内部 Handler  
//内部 push pop 只做 32 位操作,段寄存器也当寄存器处理  
void DPushReg32();  
void DPushImm32();  
void DPushMem32();  
  
void DPopReg32();//弹回寄存器  
void DFree();//释放堆栈 

很多文件都已发表了，就只剩下跟《编译》与《反编译》有关部分，这一部分也很有意思，不过要理解它，需要先了解机器指令的格式，这一部分随便在网上找找都有，因为篇幅问题，对机器指令格式就不细说  
  
实际上，《编译》与《反编译》部分的源代码是属于 OllyDBG 的，接下来将会发表《编译》与《反编译》部分的内容，如果可以，会将整个程序源码作为附件发送，待续...... 

// ----------------------------------------- assemble.cpp ------------------------------------------  
  
// 此文件用于扫描汇编指令，得到扫描的符号，并检测操作数的合法性，最后可将某个语句编译成机器码  
  
// 理解此文件需要理解机器码的格式  
  
// 这是一个很有意思的文件  
  
// Free Disassembler and Assembler -- Assembler  
//  
// Copyright (C) 2001 Oleh Yuschuk  
//  
//  This program is free software; you can redistribute it and/or modify  
//  it under the terms of the GNU General Public License as published by  
//  the Free Software Foundation; either version 2 of the License, or  
//  (at your option) any later version.  
//  
//  This program is distributed in the hope that it will be useful,  
//  but WITHOUT ANY WARRANTY; without even the implied warranty of  
//  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the  
//  GNU General Public License for more details.  
//  
//  You should have received a copy of the GNU General Public License  
//  along with this program; if not, write to the Free Software  
//  Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA  
  
// 16.01.2002 - corrected error in processing of immediate constants.  
  
  
#define STRICT  
  
#include "stdafx.h"  
#include <windows.h>  
#include <stdio.h>  
#include <string.h>  
#include <ctype.h>  
//#include <dir.h>  
#include <math.h>  
#include <float.h>  
#pragma hdrstop  
  
#include "disasm.h"  
  
#pragma   warning(disable:4996)    // 关闭旧函数声明警告  
////////////////////////////////////////////////////////////////////////////////  
///////////////////////////// ASSEMBLER FUNCTIONS //////////////////////////////  
  
// Scanner modes.  
#define SA\_NAME        0x0001          // Don't try to decode labels  
#define SA\_IMPORT      0x0002          // Allow import pseudolabel  
  
// Types of input tokens reported by scanner.  
  
// 表示指令行结束  
#define SCAN\_EOL       0               // End of line  
  
// 扫描到 8 位寄存器  
#define SCAN\_REG8      1               // 8-bit register  
  
// 扫描到 16 位寄存器  
#define SCAN\_REG16     2               // 16-bit register  
  
// 扫描到 32 位寄存器  
#define SCAN\_REG32     3               // 32-bit register  
  
// 扫描到段寄存器  
#define SCAN\_SEG       4               // Segment register  
  
// 扫描到 FPU 寄存器  
#define SCAN\_FPU       5               // FPU register  
  
// 扫描到 MMX 寄存器  
#define SCAN\_MMX       6               // MMX register  
  
// 扫描到控制寄存器  
#define SCAN\_CR        7               // Control register  
  
// 扫描到调试寄存器  
#define SCAN\_DR        8               // Debug register  
  
// 扫描到操作数据长度  
#define SCAN\_OPSIZE    9               // Operand size modifier  
#define SCAN\_JMPSIZE   10              // Jump size modifier  
#define SCAN\_LOCAL     11              // Address on stack in form LOCAL.decimal  
#define SCAN\_ARG       12              // Address on stack in form ARG.decimal  
#define SCAN\_PTR       20              // PTR in MASM addressing statements  
#define SCAN\_REP       21              // REP prefix  
#define SCAN\_REPE      22              // REPE prefix  
#define SCAN\_REPNE     23              // REPNE prefix  
#define SCAN\_LOCK      24              // LOCK prefix  
#define SCAN\_NAME      25              // Command or label  
  
// 扫描到 16 进制常数  
#define SCAN\_ICONST    26              // Hexadecimal constant  
  
// 扫描到 10 进制数  
#define SCAN\_DCONST    27              // Decimal constant  
  
// 扫描到不明确的常量值  
#define SCAN\_OFS       28              // Undefined constant  
  
// 扫描到浮点数  
#define SCAN\_FCONST    29              // Floating-point constant  
#define SCAN\_EIP       30              // Register EIP  
#define SCAN\_SIGNED    31              // Keyword "SIGNED" (in expressions)  
#define SCAN\_UNSIGNED  32              // Keyword "UNSIGNED" (in expressions)  
#define SCAN\_CHAR      33              // Keyword "CHAR" (in expressions)  
#define SCAN\_FLOAT     34              // Keyword "FLOAT" (in expressions)  
#define SCAN\_DOUBLE    35              // Keyword "DOUBLE" (in expressions)  
#define SCAN\_FLOAT10   36              // Keyword "FLOAT10" (in expressions)  
#define SCAN\_STRING    37              // Keyword "STRING" (in expressions)  
#define SCAN\_UNICODE   38              // Keyword "UNICODE" (in expressions)  
#define SCAN\_MSG       39              // Pseudovariable MSG (in expressions)  
  
// 扫描到其他符号，如','  
#define SCAN\_SYMB      64              // Any other character  
#define SCAN\_IMPORT    65              // Import pseudolabel  
  
// 扫描出错  
#define SCAN\_ERR       255             // Definitely bad item  
  
// Definition used by Assembler to report command matching errors.  
#define MA\_JMP         0x0001          // Invalid jump size modifier  
  
// 指令操作数个数有误  
#define MA\_NOP         0x0002          // Wrong number of operands  
// 操作数类型有误  
#define MA\_TYP         0x0004          // Bad operand type  
#define MA\_NOS         0x0008          // Explicit operand size expected  
// 操作数长度有误  
#define MA\_SIZ         0x0010          // Bad operand size  
// 操作数大小不同  
#define MA\_DIF         0x0020          // Different operand sizes  
#define MA\_SEG         0x0040          // Invalid segment register  
#define MA\_RNG         0x0080          // Constant out of expected range  
  
/\*\*  
 \* 用于存储汇编指令中某个操作数的相关信息  
 \*/  
typedef struct t\_asmoperand  
{  
  // 操作数类型  
  // REG：表示操作数是通用寄存器  
  // RST：表示操作数是 FPU 寄存器  
  // IMM：表示操作数是一个立即数  
  // JMF：表示操作数是 jmp 或 call 的地址  
  // SGM：表示操作数是一个段寄存器  
  // MXL：表示操作数是一个 XLAT 操作数  // XLAT operand (\[EBX+AL\])  
  // MRG：表示操作数包含 ModRM 寻址的信息  
  int            type;                 // Operand type, see beginning of file  
  
  // 表明操作数的长度，如 8 位寄存器则为 1，  
  // 16 位寄存器则为 2，32 位寄存器则为 4  
  int            size;                 // Operand size or 0 if yet unknown  
  
  // 例如当 type 为某个寄存器时，指明操作相应寄存器的下标  
  int            index;                // Index or other register  
  int            scale;                // Scale  
  int            base;                 // Base register if present  
  
  // 当操作数是一个立即数时，存储该立即数  
  // 当操作数是通过如 OFFSET 计算出的偏移量时，存放偏移量，但实际上偏移量的值是不明确的  
  long           offset;               // Immediate value or offset  
  
  // 表示 offset 的数存在，但其值不明确（即存放的可能是某个偏移量）  
  int            anyoffset;            // Offset is present but undefined  
  
  // 当操作数是 seg:offset 时，offset 存储其偏移，segment 存储 seg 段信息  
  int            segment;              // Segment in address if present  
  
  // 指定跳转指令的跳转地址修饰符，如 FAR、NEAR、SHORT、LONG 等  
  // SHORT：1，LONG：2，NEAR：4，FAR：8  
  int            jmpmode;              // Specified jump size  
} t\_asmoperand;  
  
// 指向以 0 字符结束、要编译的源代码指令  
static char      \* asmcmd;              // Pointer to 0-terminated source line  
  
/\*\*  
 \* 表明扫描的元素类型  
 \* SCAN\_REG8：表明是 8 位通用寄存器  
 \* SCAN\_FPU：表明是 FPU 寄存器  
 \* SCAN\_OPSIZE：表明是操作数长度修饰符，如 BYTE  
 \* SCAN\_JMPSIZE：表明是 jmp 跳转指令跳转长度修饰符，如 SHORT  
 \* SCAN\_REP：表明扫描到指令前缀：REP 等  
 \* SCAN\_LOCK：表明扫描到指令前缀：LOCK  
 \* SCAN\_PTR：表明扫描到 PTR 修饰符  
 \*  
 \* SCAN\_SIGNED： 表明扫描到各种数据类型修饰符  
 \* SCAN\_UNSIGNED  
 \* SCAN\_CHAR  
 \* SCAN\_FLOAT  
 \* SCAN\_DOUBLE  
 \*  
 \* SCAN\_NAME：表明扫描到标识符名称、指令名等  
 \*/  
static int       scan;                 // Type of last scanned element  
  
// 当如 idata 为 || 等运算符时，prio 代表该运算符的优先级  
static int       prio;                 // Priority of operation (0: highest)  
  
// 存储最后扫描到的名字（如标识名，指令名等）  
static char      sdata\[TEXTLEN\];       // Last scanned name (depends on type)  
  
/\*\*  
 \* 保存最后扫描到的元素值：  
 \* 1、如果扫描到的是寄存器，则保存该寄存器编号  
 \* 2、如果扫描到的是长度修饰符（见数组 sizename\[\]），则是修饰符下标  
 \* 3、如果扫描到的是 jmp 指令跳转长度修饰符，则如 SHORT 为 1、LONG 为 2  
 \* 4、如果扫描到的是 SCAN\_LOCAL，则表示 Local.decimal 后的 decimal  
 \* 5、如果扫描到的是 SCAN\_ARG，则表示 Arg.decimal 后的 decimal  
 \* 6、如果扫描到的是标识符名称、指令名等，则表示该名称的长度  
 \* 7、如果扫描到的是数字，则保存数字值  
 \* 8、如果扫描到的是字符，则保存该字符的 ASCII 码  
 \*    实际上，当扫描到字符时，scan 中存放的是 SCAN\_ICONST，表示扫描  
 \*    到常数，由此可见，扫描到字符是以扫描到常数看待的  
 \* 9、如果扫描到的是运算符，则存储之，例如 idata = '||'  
 \* 10、如果扫描到的是 SCAN\_IMPORT，则存储 import name  
 \*/  
static long      idata;                // Last scanned value  
  
// 用于存储扫描到的浮点大数  
static long      double fdata;         // Floating-point number  
// 指向出错的内容  
static char      \*asmerror;            // Explanation of last error, or NULL  
  
// Simple and slightly recursive scanner shared by Assemble(). The scanner is  
// straightforward and ineffective, but high speed is not a must here. As  
// input, it uses global pointer to source line asmcmd. On exit, it fills in  
// global variables scan, prio, sdata, idata and/or fdata. If some error is  
// detected, asmerror points to error message, otherwise asmerror remains  
// unchanged.  
  
/\*\*  
 \* 对汇编指令进行扫描，扫描结果存储至全局变量中：  
 \* scan：存放扫描到的元素类型  
 \* prio：如果扫描到的是运算符，则存储其优先级  
 \* sdata：存放扫描元素的字符串  
 \* idata：存放扫描元素的相关值  
 \* fdata：同 idata，当扫描到的元素是一浮点数时，保存该值  
 \* asmerror：存放扫描过程中出现的错误信息  
 \*  
 \* mode：指明扫描模式  
 \*  
 \* 返回值：SCAN\_ERR 表明扫描中出现错误  
 \*       SCAN\_EOL 表明扫描结束  
 \*       ...  
 \*/  
static void Scanasm(int mode)    // \*  
{  
  int i, j, base, maxdigit;  
  long decimal, hex;  
  long double floating, divisor;  
  char s\[TEXTLEN\], \* pcmd;  
  sdata\[0\] = '\\0';  
  idata = 0;  
  
  // 如果汇编指令为空，则出错  
  if (asmcmd == NULL)  
  {  
    asmerror = "NULL input line";  
    scan = SCAN\_ERR;  
    return;  
  };  
  
  // 跳过汇编指令前的空格  
  while (\*asmcmd == ' ' || \*asmcmd == '\\t')  
    asmcmd++;      // Skip leading spaces  
  
  // 扫描到汇编指令结束符，则直接返回  
  if (\*asmcmd == '\\0' || \*asmcmd == ';')  
  {  
    scan = SCAN\_EOL;  
    return;  
  };  
  
  // 判断 asmcmd 是否以英文字母，或下划线，或@开头，是则表明可能是指令名、寄存器名或标识符名称...  
  if (isalpha(\*asmcmd) || \*asmcmd == '\_' || \*asmcmd == '@')  
  {  
    // 先保存首字符  
    sdata\[0\] = \*asmcmd++;  
    // Some keyword or identifier  
    i = 1;  
  
    // 注：isalnum()用于判断字符是否是数字或字母  
  
    // 将指令后续内容复制到 sdata\[\]中  
    while ((isalnum(\*asmcmd) || \*asmcmd == '\_' || \*asmcmd == '@') && i < sizeof(sdata))  
      sdata\[i++\] = \*asmcmd++;  
  
    // 如果标识符长度超限制，则出错返回  
    if (i >= sizeof(sdata))  
    {  
      asmerror = "Too long identifier";  
      scan = SCAN\_ERR;   
      return;  
    };  
    // 给扫描到的 sdata 名称添加结束符  
    sdata\[i\] = '\\0';  
  
    // 再次跳过空格  
    while (\*asmcmd == ' ' || \*asmcmd=='\\t')  
      // Skip trailing spaces  
      asmcmd++;  
  
    // 将 sdata 复制至 s 中，并转为大写字母  
    strcpy(s, sdata);  
    \_strupr\_s(s);  
  
    // 至此，s 可能是指令名、寄存器名或标识符名称...  
  
    // 判断 s 是否是低 8 位寄存器，如 AL、CL、DL 等  
    for (j = 0; j <= 8; j++)  
    {  
      // j == 8 means "any register"  
      if (strcmp(s, regname\[0\]\[j\]) != 0)  
        continue;  
      idata = j;  
  
      // 表示扫描结果是一个 8 位寄存器  
      scan = SCAN\_REG8;    // 8-bit register  
      return;  
    };  
    // 判断 s 是否是 16 位寄存器，如 AX、CX、DX 等  
    for (j = 0; j <= 8; j++)  
    {  
      if (strcmp(s, regname\[1\]\[j\]) != 0)  
        continue;  
      idata = j;  
      scan = SCAN\_REG16;    // 16-bit register  
      return;  
    };  
    // 判断 s 是否是 32 位寄存器，如 EAX、ECX、EDX 等  
    for (j = 0; j <= 8; j++)  
    {  
      if (strcmp(s, regname\[2\]\[j\]) != 0)  
        continue;  
      idata = j;   
      scan = SCAN\_REG32;    // 32-bit register  
      return;  
    };  
    // 判断 s 是否是段寄存器，如 CS、SS 等  
    for (j = 0; j < 6; j++)  
    {  
      if (strcmp(s, segname\[j\]) != 0)  
        continue;  
      idata = j;  
      scan = SCAN\_SEG;    // Segment register  
      // 跳过空格  
      // 问题：进入此判断前已经跳过一次了，怎么还需跳过 \[??\]  
      while (\*asmcmd == ' ' || \*asmcmd == '\\t')  
        asmcmd++;      // Skip trailing spaces  
      return;  
    };  
  
    // 判断是否扫描到 FPU 寄存器  
    if (strcmp(s, "ST") == 0)  
    {  
      // 先保存原汇编指令指针，以便需要时恢复  
      pcmd = asmcmd;  
      // 递归调用 Scanasm()继续扫描  
      Scanasm(SA\_NAME);    // FPU register  
  
      // 后面的内容非 '('，即非"ST("，则不处理之，  
      // 并且将原扫描的 ST 当作 FPU 寄存器处理  
      if (scan != SCAN\_SYMB || idata != '(')  
      {  
        // 恢复为原汇编指令  
        asmcmd = pcmd;    // Undo last scan  
        idata = 0;  
        scan = SCAN\_FPU;  
        return;  
      };  
  
      // 至此，说明扫描到的是 ST(，则继续扫描"("后的内容  
      Scanasm(SA\_NAME);  
  
      // j 存储该扫描到的内容的值  
      j = idata;  
      // 检测在 ST 后是否是 0 - 7 的数 \[ST0, ST1...ST7\]， 如果不是，则说明该 FPU 寄存器出错  
      if ((scan != SCAN\_ICONST && scan != SCAN\_DCONST) || idata < 0 || idata > 7)  
      {  
        asmerror = "FPU registers have indexes 0 to 7";  
        scan = SCAN\_ERR;  
        return;  
      };  
      // 至此，说明前面扫描到的是 ST(\[0-7\])，则继续扫描  
      Scanasm(SA\_NAME);  
      // 扫描不到")"，说明不是 ST(\[0-7\])，则出错返回  
      if (scan != SCAN\_SYMB || idata != ')')  
      {  
        asmerror = "Closing parenthesis expected";  
        scan = SCAN\_ERR;  
        return;  
      };  
      // 至此，说明扫描到的是 ST(\[0-7\])  
      idata = j;  
      scan = SCAN\_FPU;  
      return;  
    };  
  
    // 判断是否是 FPU 寄存器，格式：ST\[0 - 7\]  
    for (j = 0; j <= 8; j++)  
    {  
      if (strcmp(s, fpuname\[j\]) != 0)  
        continue;  
      idata = j;  
      scan = SCAN\_FPU;    // FPU register (alternative coding)  
      return;   
    };  
  
    // 判断是否是 MMX 寄存器，如 MM0、MM1  
    for (j = 0; j <= 8; j++)  
    {  
      if (strcmp(s, mmxname\[j\]) != 0)   
        continue;  
      idata = j;  
      scan = SCAN\_MMX;    // MMX register  
      return;  
    };  
  
    // 判断是否是控制寄存器，如 CR0、CR1  
    for (j = 0; j <= 8; j++)  
    {  
      if (strcmp(s, crname\[j\]) != 0)  
        continue;  
      idata = j;  
      scan = SCAN\_CR;      // Control register  
      return;  
    };  
  
    // 判断是否调试寄存器，如 DR0、DR1  
    for (j = 0; j <= 8; j++)   
    {  
      if (strcmp(s, drname\[j\]) != 0)   
        continue;  
      idata = j;  
      scan = SCAN\_DR;      // Debug register  
      return;  
    };  
  
    // 判断是否是数据长度修饰符  
    for (j = 0; j < sizeof(sizename) / sizeof(sizename\[0\]); j++)   
    {  
      if (strcmp(s, sizename\[j\]) != 0)  
        continue;  
      pcmd = asmcmd;  
      // 如果是数据长度修饰符，则继续扫描看是否带有 PTR  
      Scanasm(SA\_NAME);  
        
      // 如果不是如 BYTE PTR 格式，则恢复  
      if (scan != SCAN\_PTR)    // Fetch non-functional "PTR"  
        asmcmd = pcmd;  
      idata = j;  
      scan = SCAN\_OPSIZE;    // Operand (data) size in bytes  
      return;  
    };  
  
    // 判断是否是 EIP 寄存器  
    if (strcmp(s, "EIP") == 0)  
    {  
      // Register EIP  
      scan = SCAN\_EIP;  
      idata = 0;  
      return;  
    };  
  
    // 判断是否是 JMP 指令跳转长度修饰符  
    if (strcmp(s, "SHORT") == 0)   
    {  
      // Relative jump has 1-byte offset  
      scan = SCAN\_JMPSIZE;  
      idata = 1;  
      return;  
    };  
    if (strcmp(s, "LONG") == 0)  
    {  
      // Relative jump has 4-byte offset  
      scan = SCAN\_JMPSIZE;  
      idata = 2;  
      return;  
    };  
    if (strcmp(s, "NEAR") == 0)   
    {  
      // Jump within same code segment  
      scan = SCAN\_JMPSIZE;  
      idata = 4;  
      return;  
    };  
    if (strcmp(s, "FAR") == 0)   
    {  
      // Jump to different code segment  
      scan = SCAN\_JMPSIZE;  
      idata = 8;  
      return;   
    };  
  
    // LOCAL 是定义局部变量用的.用这个关键字是为了让编译器  
    // 给变量分配内存.  
    // 格式是:  
    //    LOCAL  变量名:变量类型  
      
    // LOCAL.decimal  
    if (strcmp(s, "LOCAL") == 0 && \*asmcmd == '.')  
    {  
      // 跳过"."  
      asmcmd++;  
      // 跳过空格  
      while (\*asmcmd == ' ' || \*asmcmd == '\\t')  
        asmcmd++;        // Skip trailing spaces  
      // 判断在 LOCAL 后是否是数字， LOCAL.后面必须是数字  
      if (!isdigit(\*asmcmd))  
      {  
        asmerror = "Integer number expected";  
        scan = SCAN\_ERR;  
        return;  
      };  
      // 获取 LOCAL. 后面的数字信息  
      while (isdigit(\*asmcmd))  // LOCAL index is decimal number!  
        idata = idata \* 10 + (\*asmcmd++) - '0';  
      scan = SCAN\_LOCAL;  
      return;   
    };  
  
    // ARG.decimal  
    if (strcmp(s, "ARG") == 0 && \*asmcmd == '.')  
    {  
      asmcmd++;  
      while (\*asmcmd == ' ' || \*asmcmd == '\\t')  
        asmcmd++;        // Skip trailing spaces  
      if (!isdigit(\*asmcmd))  
      {  
        asmerror = "Integer number expected";  
        scan = SCAN\_ERR;  
        return;  
      };  
      while (isdigit(\*asmcmd))  // ARG index is decimal number!  
        idata = idata \* 10 + (\*asmcmd++) - '0';  
      scan = SCAN\_ARG;  
      return;  
    };  
  
    // 判断是否是 REP 前缀  
    if (strcmp(s, "REP") == 0)   
    {  
      scan = SCAN\_REP;  
      return;   
    };  
    // 判断是否是 REPE 前缀  
    if (strcmp(s, "REPE") == 0 || strcmp(s, "REPZ") == 0)  
    {  
      scan = SCAN\_REPE;  
      return;  
    };  
    // 判断是否是 REPNE 前缀  
    if (strcmp(s, "REPNE") == 0 || strcmp(s, "REPNZ") == 0)   
    {  
      scan = SCAN\_REPNE;  
      return;  
    };  
    // 判断是否是 LOCK 前缀  
    if (strcmp(s, "LOCK") == 0)   
    {  
      scan = SCAN\_LOCK;  
      return;  
    };  
    // 判断是否 PTR  
    if (strcmp(s, "PTR") == 0)  
    {  
      scan = SCAN\_PTR;  
      return;  
    };  
  
    // Undefined constant  
    // Present but undefined offset/constant  
    // OFFSET：在汇编中用于得到变量的偏址，偏址相当于一个常量，但该常量值并不明确多少（在哪）  
    if (strcmp(s, "CONST") == 0 || strcmp(s, "OFFSET") == 0)  
    {  
      scan = SCAN\_OFS;  
      return;  
    };  
  
    // Keyword "SIGNED" (in expressions)  
    // 判断是否是有符号数  
    if (strcmp(s, "SIGNED") == 0)  
    {  
      scan = SCAN\_SIGNED;  
      return;   
    };  
    // Keyword "UNSIGNED" (in expressions)  
    // 判断是否无符号数  
    if (strcmp(s, "UNSIGNED") == 0)  
    {  
      scan = SCAN\_UNSIGNED;  
      return;  
    };  
    // Keyword "CHAR" (in expressions)  
    // 判断是数据类型 CHAR  
    if (strcmp(s, "CHAR") == 0)  
    {  
      scan = SCAN\_CHAR;  
      return;  
    };  
    // 判断是数据类型 FLOAT  
    // Keyword "FLOAT" (in expressions)  
    if (strcmp(s, "FLOAT") == 0)   
    {  
      scan = SCAN\_FLOAT;  
      return;  
    };  
    // Keyword "DOUBLE" (in expressions)  
    // 判断是数据类型 DOUBLE  
    if (strcmp(s, "DOUBLE") == 0)   
    {  
      scan = SCAN\_DOUBLE;  
      return;   
    };  
    // Keyword "FLOAT10" (in expressions)  
    // 判断是数据类型 FLOAT10  
    if (strcmp(s, "FLOAT10") == 0)  
    {  
      scan = SCAN\_FLOAT10;   
      return;  
    };  
    // Keyword "STRING" (in expressions)  
    // 判断是数据类型 STRING  
    if (strcmp(s, "STRING") == 0)   
    {  
      scan = SCAN\_STRING;  
      return;   
    };  
    // Keyword "UNICODE" (in expressions)  
    // 判断是数据类型 UNICODE  
    if (strcmp(s, "UNICODE") == 0)  
    {  
      scan = SCAN\_UNICODE;  
      return;  
    };  
    // Pseudovariable MSG (in expressions)  
    if (strcmp(s, "MSG") == 0)  
    {  
      scan = SCAN\_MSG;  
      return;  
    };  
  
    // 至此，说明 s 是标签了，包括标识符名称、指令名等，无需对标签进行解码，则直接返回  
    if (mode & SA\_NAME)   
    {  
      idata = i;  
      // Don't try to decode symbolic label  
      scan = SCAN\_NAME;  
      return;  
    }  
  
    // 至此，说明此标识符无效，出错返回  
    asmerror = "Unknown identifier";  
    scan = SCAN\_ERR;  
    return;   
  }  
  // 否则，如果扫描到的是数字  
  else if (isdigit(\*asmcmd))  
  {  
    // Constant  
    // 数字基数  
    base = 0;  
    // 该数所属进制数最大数，如十进制为 9，十六进制为 15  
    maxdigit = 0;  
    // 存储该数字  
    decimal = hex = 0L;  
    floating = 0.0;  
  
    // Force hexadecimal number  
    // 数字以 0X/0x 开头，则是 16 进制数  
    if (asmcmd\[0\] == '0' && toupper(asmcmd\[1\]) == 'X')  
    {  
      // 基数为 16  
      base = 16;  
      // 跳过数字的 0X/0x  
      asmcmd += 2;  
    };  
    // 获取该数字  
    while (1)   
    {  
      // 如果是数字  
      if (isdigit(\*asmcmd))  
      {  
        decimal = decimal \* 10 + (\*asmcmd) - '0';  
        floating = floating \* 10.0 + (\*asmcmd) - '0';  
        hex = hex \* 16 + (\*asmcmd) - '0';  
        // 默认最大数为 9，即以十进制数算  
        if (maxdigit == 0)  
          maxdigit = 9;  
        asmcmd++;  
      }  
      // 否则，如果是 16 进制数的字母部分  
      else if (isxdigit(\*asmcmd))   
      {  
        hex = hex \* 16 + toupper(\*asmcmd++) - 'A' + 10;  
        maxdigit = 15;   
      }  
      else  
        break;  
    };  
    // 如果进制的最大数为 0，那就奇怪了  
    if (maxdigit == 0)  
    {  
      asmerror = "Hexadecimal digits after 0x... expected";  
      scan = SCAN\_ERR;  
      return;  
    };  
    // 如果数字后为 H 字母，则也是 16 进制数  
    if (toupper(\*asmcmd) == 'H')  
    {  
      // Force hexadecimal number  
      if (base == 16)  
      {  
        // 16 进制数格式不应该为：0xXXXXH  
        asmerror = "Please don't mix 0xXXXX and XXXXh forms";  
        scan = SCAN\_ERR;  
        return;  
      };  
      // 跳过 'H' 字符  
      asmcmd++;  
      // 保存该数字  
      idata = hex;  
      // 扫描到 16 进制常数  
      scan = SCAN\_ICONST;  
      while (\*asmcmd == ' ' || \*asmcmd == '\\t')  
        asmcmd++;  
      return;  
    };  
    // 可能是浮点数，必须是一个 10 进制数  
    if (\*asmcmd == '.')   
    {  
      // Force decimal number  
      // 但如果不是 10 进制数，则出错  
      if (base == 16 || maxdigit > 9)   
      {  
        asmerror = "Not a decimal number";  
        scan = SCAN\_ERR;  
        return;  
      };  
      asmcmd++;  
      // 如果带有小数，或是科学计数法  
      if (isdigit(\*asmcmd) || toupper(\*asmcmd) == 'E')  
      {  
        divisor = 1.0;    // 除数  
        // 计算小数部分  
        while (isdigit(\*asmcmd))  
        {  
          // Floating-point number  
          divisor /= 10.0;  
          floating += divisor \* (\*asmcmd - '0');  
          asmcmd++;   
        };  
        // 科学记数法  
        if (toupper(\*asmcmd) == 'E')  
        {  
          asmcmd++;  
          if (\*asmcmd == '-')  
          {  
            base = -1;  
            asmcmd++;  
          }  
          else   
            base = 1;  
          if (!isdigit(\*asmcmd))  
          {  
            asmerror = "Invalid exponent";  
            scan = SCAN\_ERR;   
            return;  
          };  
          decimal = 0;  
          while (isdigit(\*asmcmd))  
          {  
            if (decimal < 65536L)  
              decimal = decimal \* 10 + (\*asmcmd++) - '0';  
          };  
  
          floating \*= powl(decimal \* base, 1);  
        };  
        fdata = floating;  
        // 扫描到一浮点数  
        scan = SCAN\_FCONST;  
        return;  
      }  
      // 扫描到一 10 进制数  
      else  
      {  
        idata = decimal;  
        scan = SCAN\_DCONST;  
        while (\*asmcmd == ' ' || \*asmcmd == '\\t')   
          asmcmd++;  
        return;  
      };  
    };  
    //   
    idata = hex;  
    // Default is hexadecimal  
    // 默认情况下用 16 进制数  
    scan = SCAN\_ICONST;  
    while (\*asmcmd == ' ' || \*asmcmd == '\\t')   
      asmcmd++;  
    return;  
  }  
  // 否则，如果以 ' 开始，表示扫描到字符  
  else if (\*asmcmd=='\\'')   
  {  
    // Character constant  
    // 字符常量，跳过 ' 符号  
    asmcmd++;  
    if (\*asmcmd == '\\0' || (\*asmcmd == '\\\\' && asmcmd\[1\] == '\\0'))  
    {  
      asmerror = "Unterminated character constant";  
      scan = SCAN\_ERR;  
      return;  
    };  
    // 空字符，出错返回  
    // 问题：为何不能使用空字符呢 \[??\]  
    if (\*asmcmd == '\\'')  
    {  
      asmerror = "Empty character constant";  
      scan = SCAN\_ERR;   
      return;  
    };  
    // 跳过转义字符的 "\\"  
    if (\*asmcmd == '\\\\')  
      asmcmd++;  
  
    // 获取该字符的 ASCII 码  
    idata = \*asmcmd++;  
  
    // 从这里看得出来，似乎只有一个字符  
  
    // 扫描结束字符  
    if (\*asmcmd != '\\'')  
    {  
      asmerror = "Unterminated character constant";  
      scan = SCAN\_ERR;  
      return;  
    };  
    asmcmd++;  
    // 跳过空格部分  
    while (\*asmcmd == ' ' || \*asmcmd == '\\t')   
      asmcmd++;  
  
    // 扫描到字符，相当于扫描到 ASCII 码，所以，  
    // 也将扫描结果作为常量看待  
    scan = SCAN\_ICONST;  
    return;  
  }  
  // 否则，扫描到的可能是一些运算符  
  else  
  {  
    // Any other character or combination  
    // 存储首字符  
    idata = sdata\[0\] = \*asmcmd++;  
    sdata\[1\] = sdata\[2\] = '\\0';  
    // 判断是否逻辑运算符  
    if (idata == '|' && \*asmcmd == '|')  
    {  
      // '||'  
      // 问题：'||'是一个字符吗 \[??\]  
      idata = '||';  
      prio = 10;  
      sdata\[1\] = \*asmcmd++;  
    }  
    else if (idata == '&' && \*asmcmd == '&')  
    {  
      // '&&'  
      idata = '&&';  
      prio = 9;  
      sdata\[1\] = \*asmcmd++;  
    }  
    else if (idata == '=' && \*asmcmd == '=')   
    {  
      // '=='  
      idata = '==';  
      prio = 5;  
      sdata\[1\] = \*asmcmd++;   
    }  
    else if (idata == '!' && \*asmcmd == '=')  
    {  
      // '!='  
      idata = '!=';  
      prio = 5;  
      sdata\[1\] = \*asmcmd++;  
    }  
    else if (idata == '<' && \*asmcmd == '=')  
    {  
      // '<='  
      idata = '<=';  
      prio = 4;  
      sdata\[1\] = \*asmcmd++;  
    }  
    else if (idata == '>' && \*asmcmd == '=')   
    {  
      // '>='  
      idata = '>=';  
      prio = 4;  
      sdata\[1\] = \*asmcmd++;  
    }  
    else if (idata == '<' && \*asmcmd == '<')   
    {  
      // '<<'  
      idata = '<<';  
      prio = 3;  
      sdata\[1\] = \*asmcmd++;   
    }  
    else if (idata == '>' && \*asmcmd == '>')  
    {  
      // '>>'  
      idata = '>>';  
      prio = 3;  
      sdata\[1\] = \*asmcmd++;   
    }  
    // '|'  
    else if (idata == '|')  
      prio = 8;  
    // 异或运算符  
    // '^'  
    else if (idata == '^')   
      prio = 7;  
    else if (idata == '&')   
      prio = 6;       // '&'  
  
    // 似乎表示是否是 include < 指令的 < 部分  
    else if (idata == '<')  
    {  
      if (\*asmcmd == '&')  
      {  
        // Import pseudolabel (for internal use)  
        if ((mode & SA\_IMPORT) == 0)  
        {  
          asmerror = "Syntax error";  
          scan = SCAN\_ERR;  
          return;  
        };  
        asmcmd++;  
        i = 0;  
        // 获取 import 指令的 import name  
        while (\*asmcmd != '\\0' && \*asmcmd != '>')  
        {  
          sdata\[i++\] = \*asmcmd++;  
          if (i >= sizeof(sdata))  
          {  
            asmerror = "Too long import name";   
            scan = SCAN\_ERR;   
            return;  
          };  
        };  
        // import 指令必须 <&XXXX>  
        if (\*asmcmd != '>')   
        {  
          asmerror = "Unterminated import name";  
          scan = SCAN\_ERR;   
          return;  
        };  
        asmcmd++;  
        // 设置 name 的结束符  
        sdata\[i\] = '\\0';  
        scan = SCAN\_IMPORT;  
        return;  
      }  
      // 只是一个 < 号  
      else  
        prio = 4;  
    }  
    else if (idata == '>')  
      prio = 4;    // '>'  
    else if (idata == '+')  
      prio = 2;    // '+'  
    else if (idata == '-')  
      prio = 2;    // '-'  
    else if (idata == '\*')   
      prio = 1;    // '\*'  
    else if (idata == '/')  
      prio = 1;    // '/'  
    else if (idata == '%')   
      prio = 1;    // '%'  
    else if (idata == '\]')  
    {  
      pcmd = asmcmd;   
      Scanasm(SA\_NAME);  
      if (scan != SCAN\_SYMB || idata != '\[')  
      {  
        idata = '\]';  
        asmcmd = pcmd;   
        prio = 0;  
      }  
      // 将 \]\[ 符号解析为 + 号  
      else  
      {  
        idata = '+';  
        prio = 2;  // Translate '\]\[' to '+'  
      };  
    }  
    // 否则，扫描到其他符号  
    else  
      prio = 0;    // Any other character  
  
    // 其他符号  
    scan = SCAN\_SYMB;  
    return;  
  };  
};  
  
// Fetches one complete operand from the input line and fills in structure op  
// with operand's data. Expects that first token of the operand is already  
// scanned. Supports operands in generalized form (for example, R32 means any  
// of general-purpose 32-bit integer registers).  
  
/\*\*  
 \* 对汇编指令操作数进行解析  
 \*  
 \* op: 用于返回解析的操作数  
 \*/  
static void Parseasmoperand(t\_asmoperand \* op)    // \*  
{  
  int i, j, bracket, sign, xlataddr;  
  int reg, r\[9\];  
  long offset;  
  
  // No or bad operand  
  // 扫描不到操作数，或出错，则直接返回  
  if (scan == SCAN\_EOL || scan == SCAN\_ERR)  
    return;  
  
  // Jump or call address may begin with address size modifier(s) SHORT, LONG,  
  // NEAR and/or FAR. Not all combinations are allowed. After operand is  
  // completely parsed, this function roughly checks whether modifier is  
  // allowed. Exact check is done in Assemble().  
  
  // 跳转或 call 指令的地址可以用修饰符修饰，如 SHORT、LONG、NEAR 和 FAR。并不是所有组  
  // 合都是允许的。此函数只是大要检测修饰是否允许，而更精确的检测在 Assemble()中  
  
  // 如果扫描到 Jump 地址修饰符，如 SHORT  
  if (scan == SCAN\_JMPSIZE)  
  {  
    j = 0;  
    // 扫描并获取所有地址修饰符  
    while (scan == SCAN\_JMPSIZE)  
    {  
      j |= idata;    // Fetch all size modifiers  
      // 往下继续扫描看是否有修饰符  
      Scanasm(0);  
    };  
  
    // SHORT：1，LONG：2，NEAR：4，FAR：8  
  
    // 如果修饰符即有 SHORT，又掺杂了 LONG，则出错  
    if (((j & 0x03) == 0x03) ||    // Mixed 掺杂 SHORT and LONG  
      ((j & 0x0C) == 0x0C) ||    // Mixed NEAR and FAR  
      ((j & 0x09) == 0x09))    // Mixed FAR and SHORT  
    {  
      asmerror = "Invalid combination of jump address modifiers";  
      scan = SCAN\_ERR;  
      return;  
    };  
    // 如果未指定 FAR，则强制为 NEAR  
    if ((j & 0x08) == 0)  
      j |= 0x04;      // Force NEAR if not FAR  
  
    // 保存跳转地址修饰符(模式)  
    op->jmpmode = j;  
  };  
  
  // Simple operands are either register or constant, their processing is  
  // obvious and straightforward.  
  
  // 最简单的操作数为寄存器操作数或立即数，他们的处理是显而易见和易懂的  
  
  // 扫描到 8 位或 16 位或 32 位通用寄存器  
  if (scan == SCAN\_REG8 || scan == SCAN\_REG16 || scan == SCAN\_REG32)  
  {  
    // 表示操作数类型为寄存器操作  
    op->type = REG;  
    // 指明操作的目标寄存器的下标  
    op->index = idata;    // Integer general-purpose register  
  
    // 保存操作数的长度值  
    if (scan == SCAN\_REG8)  
      op->size = 1;  
    else if (scan == SCAN\_REG16)  
      op->size = 2;  
    else   
      op->size = 4;  
  }  
  // 否则，如果是 FPU 寄存器操作  
  else if (scan == SCAN\_FPU)  
  {  
    // FPU register  
    op->type = RST;  
    op->index = idata;  
  }  
  // 否则，如果是 MMX 寄存器  
  else if (scan == SCAN\_MMX)   
  {  
    // MMX or 3DNow! register  
    op->type = RMX;  
    op->index = idata;  
  }  
  // 否则，如果是 CR 控制寄存器  
  else if (scan == SCAN\_CR)  
  {  
    // Control register  
    op->type = CRX;  
    op->index = idata;  
  }  
  // 否则，如果是调试寄存器  
  else if (scan == SCAN\_DR)   
  {  
    // Debug register  
    op->type = DRX;  
    op->index = idata;  
  }  
  
  // 至此，说明扫描到的不是寄存器操作数  
  
  // 否则，如果是其他符号，负数...  
  else if (scan == SCAN\_SYMB && idata == '-')  
  {  
    // 负数  
    Scanasm(0);      // Negative constant  
  
    // 扫描到的不是数字，则出错  
    if (scan != SCAN\_ICONST && scan != SCAN\_DCONST && scan != SCAN\_OFS)  
    {  
      asmerror = "Integer number expected";  
      scan = SCAN\_ERR;  
      return;  
    };  
    // 操作类型为直接数操作  
    op->type = IMM;  
  
    // 保存直接数或偏移  
    op->offset = -idata;  
    if (scan == SCAN\_OFS)  
      // 表示 offset 数存在，但该值不明确  
      op->anyoffset = 1;  
  }  
  // 操作数是一个 正数  
  else if (scan == SCAN\_SYMB && idata == '+')   
  {  
    // 扫描是否是一个正数  
    Scanasm(0);      // Positive constant  
    // 如果扫描不到一整数，则出错返回  
    if (scan != SCAN\_ICONST && scan != SCAN\_DCONST && scan != SCAN\_OFS)  
    {  
      asmerror = "Integer number expected";  
      scan = SCAN\_ERR;  
      return;  
    };  
    // 操作数是一直接数  
    op->type = IMM;  
    op->offset = idata;  
    // 如果该操作数是一个偏移地址  
    if (scan == SCAN\_OFS)  
      op->anyoffset = 1;  
  }  
  // 扫描到直接数或偏移地址（值不明确的直接数）  
  else if (scan == SCAN\_ICONST || scan == SCAN\_DCONST || scan == SCAN\_OFS)  
  {  
    j = idata;  
    // 扫描到的是偏移  
    if (scan == SCAN\_OFS)  
      // 表示该直接数的值不明确  
      op->anyoffset = 1;  
  
    // 扫描直接数后的内容  
    Scanasm(0);  
    // 扫描是否 seg:offset 格式地址  
    if (scan == SCAN\_SYMB && idata == ':')  
    {  
      Scanasm(0);      // Absolute long address (seg:offset)  
      // 如果 seg: 后不是一偏移地址，则出错  
      if (scan != SCAN\_ICONST && scan != SCAN\_DCONST && scan != SCAN\_OFS)   
      {  
        asmerror = "Integer address expected";  
        scan = SCAN\_ERR;  
        return;  
      };  
      // 操作数是一跳转地址  
      op->type = JMF;  
      // 偏移地址  
      op->offset = idata;  
      // 段地址  
      op->segment = j;  
      if (scan == SCAN\_OFS)   
        op->anyoffset = 1;  
    }  
    // 否则，说明操作数是一个直接数  
    else  
    {  
      op->type = IMM;  
      // offset 保存该直接数  
      op->offset = j;      // Constant without sign  
      return;          // Next token already scanned  
    };  
  }  
  // 否则，如操作数是一个浮点数，命令是不能带浮点数的，所以出错返回  
  else if (scan == SCAN\_FCONST)  
  {  
    asmerror = "Floating-point numbers are not allowed in command";  
    scan = SCAN\_ERR;  
    return;  
  }  
  
  // 指令格式: mov dword ptr es:\[eax + ecx \* 8 + 0x11223344\], 0x12345678  
  
  // Segment register or address.  
  // 段寄存器或地址  
  else if (scan == SCAN\_SEG || scan == SCAN\_OPSIZE || (scan == SCAN\_SYMB && idata == '\['))  
  {  
    // Segment register or address  
    bracket = 0;    // 括弧  
  
    // 段寄存器  
    if (scan == SCAN\_SEG)  
    {  
      // 保存段寄存器编号  
      j = idata;  
      Scanasm(0);  
      // 判断是否为 seg:offset，如果不是，则说明扫描到的只是一个段寄存器  
      if (scan != SCAN\_SYMB || idata != ':')  
      {  
        // Segment register as operand  
        // 操作数是段寄存器  
        op->type = SGM;  
        op->index = j;  
        return;    // Next token already scanned  
      };  
      // 以 seg:offset 出现，则保存段寄存器编号  
      op->segment = j;  
      // 扫描以获得 offset  
      Scanasm(0);  
    };  
  
    // Scan 32-bit address. This parser does not support 16-bit addresses.  
    // First of all, get size of operand (optional 可选), segment register (optional)  
    // and opening bracket (required).  
    while (1)  
    {  
      // 扫描到 \[   
      if (scan == SCAN\_SYMB && idata == '\[')  
      {  
        // 如果前面已扫描到此符号，则出错返回  
        if (bracket)  
        {  
          // Bracket  
          asmerror = "Only one opening bracket allowed";  
          scan = SCAN\_ERR;  
          return;   
        };  
        // 表示扫描到 \[  
        bracket = 1;  
      }  
      // 扫描到操作数大小修饰符，如 BYTE  
      else if (scan == SCAN\_OPSIZE)  
      {  
        // 操作数大小修饰符重复  
        if (op->size != 0)  
        {  
          // Size of operand  
          asmerror = "Duplicated size modifier 修饰语";  
          scan = SCAN\_ERR;  
          return;  
        };  
        op->size = idata;  
      }  
      // 扫描到段寄存器  
      else if (scan == SCAN\_SEG)  
      {  
        // 段寄存器重复  
        if (op->segment != SEG\_UNDEF)  
        {  
          // Segment register  
          asmerror = "Duplicated segment register";  
          scan = SCAN\_ERR;  
          return;  
        };  
        // 保存段寄存器编号  
        op->segment = idata;  
        Scanasm(0);  
        if (scan != SCAN\_SYMB || idata != ':')   
        {  
          asmerror = "Semicolon 分号 expected";  
          scan = SCAN\_ERR;  
          return;  
        };  
      }  
      else if (scan == SCAN\_ERR)  
        return;  
      else   
        break;      // None of expected address elements  
      Scanasm(0);  
    };  
    // 地址必须包含有 \[ 符号  
    if (bracket == 0)  
    {  
      asmerror = "Address expression requires brackets";  
      scan = SCAN\_ERR;  
      return;  
    };  
  
    // Assembling a 32-bit address may be a kind of nightmare 恶梦, due to a large  
    // number of allowed forms. Parser collects immediate offset in op->offset  
    // and count for each register in array r\[\]. Then it decides whether this  
    // combination is valid and determines scale, index and base. Assemble()  
    // will use these numbers to select address form (with or without SIB byte,  
    // 8- or 32-bit offset, use segment prefix or not). As a useful side effect  
    // of this technique, one may specify, for example, \[EAX\*5\] which will  
    // correctly assemble to \[EAX\*4+EAX\].  
  
    // 编译一个 32 位地址简直是一个恶梦，因为对于地址有大量的格式可以使用。解析程序  
    // 将收集直接数偏移地址存放于 op->offset 中，与寄存器相关的地址信息则统计到 r\[\]  
    // 数组中。并决定该地址组合是否有效，确定 scale，index 与 base 的值。Assemble()函  
    // 数将会使用这些值进行确定地址格式（是否需要 SIB 字节来进行寻址，8 位还是 32 位偏  
    // 址，是否使用段前缀等）。作为一个非常有效的技术，举例：\[EAX \* 5\] 可以被正确  
    // 编译为 \[EAX \* 4 + EAX\]  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
    // 初始化寄存器值，该数组用于统计在地址中对寄存器的使用情况  
    for (i = 0; i <= 8; i++)  
      r\[i\] = 0;  
  
    // 默认符号  
    sign = '+';      // Default sign for the first operand  
    xlataddr = 0;  
    while (1)  
    {  
      // Get SIB and offset  
      // 扫描到 符号，则保存之  
      if (scan == SCAN\_SYMB && (idata == '+' || idata == '-'))  
      {  
        sign = idata;  
  
        // 往下扫描符号后面的数  
        Scanasm(0);  
      };  
      // 出错，直接返回  
      if (scan == SCAN\_ERR)  
        return;  
      // 语法错误  
      if (sign == '?')  
      {  
        asmerror = "Syntax error";   
        scan = SCAN\_ERR;  
        return;  
      };  
  
      // XLAT:查表指令 XLAT   
　　    // 指令格式：XLAT TABLE 其中 TABLE 为一待查表格的首地址。   
　　    // 指令功能：把待查表格的一个字节内容送到 AL 累加器中。在执行该指令前，  
      //    应将 TABLE 先送至 BX 寄存器中，然后将待查字节与其在表格中距表首  
      //    地址位移量送 AL,即(AL)<--((BX)+(AL)). 执行 XLAT 将使待查内容送  
      //    到累加器。  
　　    // 本指令不影响状态标位，表格长度不超过 256 字节。  
  
      // Register AL appears as part of operand of (seldom 很少 used) command XLAT.  
      // 因为作为操作数使用 8 位 AL 寄存器是很少见的，所以如果出现，则可能使用了 XLAT 指令  
      if (scan == SCAN\_REG8 && idata == REG\_EAX)  
      {  
        // 寄存器前不能使用 - 符号  
        if (sign == '-')  
        {  
          asmerror = "Unable to subtract register";  
          scan = SCAN\_ERR;  
          return;  
        };  
        if (xlataddr != 0)  
        {  
          asmerror = "Too many registers";   
          scan = SCAN\_ERR;   
          return;  
        };  
        // 假设使用了 XLAT 指令  
        xlataddr = 1;  
        Scanasm(0);  
      }  
      // 不支持 16 位寄存器作为地址  
      else if (scan == SCAN\_REG16)  
      {  
        asmerror = "Sorry, 16-bit addressing is not supported";  
        scan = SCAN\_ERR;  
        return;  
      }  
      // 32 位寄存器地址  
      else if (scan == SCAN\_REG32)  
      {  
        if (sign == '-')   
        {  
          asmerror = "Unable to subtract register";  
          scan = SCAN\_ERR;  
          return;  
        };  
        // 保存寄存器编号  
        reg = idata;  
  
        // 继续扫描  
        Scanasm(0);  
  
        // 如果是如上格式 EAX \* 4，\*后面的数字称为 scale  
        if (scan == SCAN\_SYMB && idata == '\*')  
        {  
          // 扫描 scale  
          Scanasm(0);    // Try index\*scale  
          if (scan == SCAN\_ERR)  
            return;  
          // 不明确的 scale 是不允许使用不明确的值  
          if (scan == SCAN\_OFS)   
          {  
            asmerror = "Undefined scale is not allowed";   
            scan = SCAN\_ERR;   
            return;  
          };  
          // 如果扫描到的不是 常量值 ，则不能作为 scale 使用，语法错误  
          if (scan != SCAN\_ICONST && scan != SCAN\_DCONST)  
          {  
            asmerror = "Syntax error";  
            scan = SCAN\_ERR;   
            return;  
          };  
          // scale 的值只能是：1，2，4，8。当在地址中出现如 5 的 scale 时，将  
          // 其当成 4 使用，但如果出现 6、7 等，则不能使用  
          if (idata == 6 || idata == 7 || idata > 9)  
          {  
            asmerror = "Invalid scale";  
            scan = SCAN\_ERR;  
            return;  
          };  
          // 计算 reg 寄存器的总 scale  
          r\[reg\] += idata;  
          Scanasm(0);  
        }  
        // 否则，说明 scale = 1  
        else  
          r\[reg\]++;  
      }      // Simple register  
  
  
      // 以下两个部分不太理解  
  
  
  
      // LOCAL 是定义局部变量用的.用这个关键字是为了让编译器  
      // 给变量分配内存.   
      // 格式是: LOCAL 变量名:变量类型  
      else if (scan == SCAN\_LOCAL)  
      {  
        r\[REG\_EBP\]++;  
        // 在堆栈上分配 idata \* 4 空间用于存放局部变量 \[??\]  
        op->offset -= idata \* 4;  
        Scanasm(0);  
      }  
      // ARG r\_Kernel32 : DWORD  
      // push r\_Kernel32  
      // call Whereis\_GPA  
      // 伪操作符 arg 的作用是,容许你输入一段字符或者数字作为  
      // 一个函数的参数(只在 pwb 环境中）  
      else if (scan == SCAN\_ARG)  
      {  
        r\[REG\_EBP\]++;  
        op->offset += (idata + 1) \* 4;  
        Scanasm(0);  
      }  
      // 扫描到直接数，则地址可能以 scale \* index 出现  
      else if (scan == SCAN\_ICONST || scan == SCAN\_DCONST)  
      {  
        offset = idata;  
        Scanasm(0);  
        if (scan == SCAN\_SYMB && idata == '\*')  
        {  
          // 尝试是否 4 \* EAX 等格式  
          Scanasm(0);        // Try scale \* index  
          if (scan == SCAN\_ERR)  
            return;  
          // 寄存器前不能加 - 号  
          if (sign == '-')  
          {  
            asmerror = "Unable to subtract register";   
            scan = SCAN\_ERR;  
            return;  
          };  
          // 不支持 16 位寄存器地址  
          if (scan == SCAN\_REG16)   
          {  
            asmerror = "Sorry, 16-bit addressing is not supported";  
            scan = SCAN\_ERR;  
            return;  
          };  
          // 必须是 32 位寄存器  
          if (scan != SCAN\_REG32)  
          {  
            asmerror = "Syntax error";  
            scan = SCAN\_ERR;   
            return;  
          };  
          // 无效的 scale  
          if (offset == 6 || offset == 7 || offset > 9)  
          {  
            asmerror = "Invalid scale";  
            scan = SCAN\_ERR;  
            return;   
          };  
          // 统计该寄存器的 scale 值  
          r\[idata\] += offset;  
          // 继续扫描后面的部分  
          Scanasm(0);  
        }  
        // 在 scale 后面没有 \* 寄存器，则说明是偏移量  
        else   
        {  
          if (sign == '-')   
            op->offset -= offset;  
          else  
            op->offset += offset;  
        };  
      }  
      else if (scan == SCAN\_OFS)   
      {  
        Scanasm(0);  
        // 如果 ofs 作为 scale 用，但因为 ofs 不明确，所以是不允许的  
        if (scan == SCAN\_SYMB && idata == '\*')  
        {  
          asmerror = "Undefined scale is not allowed";  
          scan = SCAN\_ERR;  
          return;  
        }  
        // 否则，说明不是用于 scale，则表示是一偏移  
        else  
        {  
          op->anyoffset = 1;  
        };  
      }  
      // 不属于以上情况，则跳地址无关，跳出循环  
      else  
        break;        // None of expected address elements  
  
      // 如果已扫描到地址结束符 \]，则结束  
      if (scan == SCAN\_SYMB && idata == '\]')  
        break;  
  
      // 至此，说明上面所有扫描无法得到有效结果，  
      // 则将 sign 置为 ? 表示上面的扫描有问题  
      sign = '?';  
    };  
  
    // 扫描出错，直接返回  
    if (scan == SCAN\_ERR)   
      return;  
    // 扫描不到 '\]'，则语法错误  
    if (scan != SCAN\_SYMB || idata != '\]')   
    {  
      asmerror = "Syntax error";  
      scan = SCAN\_ERR;  
      return;  
    };  
    // Process XLAT address separately.  
    // 如果确实扫描到 XLAT 地址，则检测其格式是否 OK  
    if (xlataddr != 0)  
    {  
      // XLAT address in form \[EBX+AX\]  
      // 这里似乎只是为检测除 EBX 寄存器外，是否还使用了其他寄存器  
      // 作为地址使用，当然，对 XLAT，基址只使用 EBX，所以，如果有  
      // 其他寄存器作为地址使用，或还指明了偏址，那将是错误的  
      for (i = 0; i <= 8; i++)  
      {  
        // Check which registers used  
        if (i == REG\_EBX)  
          continue;  
        if (r\[i\] != 0)  
          break;  
      };  
      if (i <= 8 || r\[REG\_EBX\] != 1 || op->offset != 0 || op->anyoffset != 0)   
      {  
        asmerror = "Invalid address";  
        scan = SCAN\_ERR;  
        return;  
      };  
      // XLAT 操作数  
      op->type = MXL;  
    }  
    // Determine scale, index and base.  
    // 否则，说明是真正的地址了，现在需要确定 scale，index 与 base 了  
    else  
    {  
      j = 0;    // Number of used registers  
      for (i = 0; i <= 8; i++)  
      {  
        // i 寄存器未被使用作为地址部分，跳过  
        if (r\[i\] == 0)  
          continue;    // Unused register  
        // 如果 scale 值为 3、5 或 9  
        if (r\[i\] == 3 || r\[i\] == 5 || r\[i\] == 9)  
        {  
          // 如果 index 存在，或基址存在  
          if (op->index >= 0 || op->base >= 0)  
          {  
            if (j == 0)  
              asmerror = "Invalid scale";  
            else   
              asmerror = "Too many registers";  
            scan = SCAN\_ERR;  
            return;  
          };  
          op->index = op->base = i;  
          // scale 只能是：1、2、4、8  
          op->scale = r\[i\] - 1;  
        }  
        else if (r\[i\] == 2 || r\[i\] == 4 || r\[i\] == 8)  
        {  
          // 已经有用于 index 的寄存器  
          if (op->index >= 0)   
          {  
            if (j <= 1)   
              asmerror = "Only one register may be scaled";  
            else   
              asmerror = "Too many registers";  
            scan = SCAN\_ERR;  
            return;   
          };  
          op->index = i;  
          op->scale = r\[i\];  
        }  
        // 如果 i 寄存器使用计数为 1，则可能是 base，也可能是 index  
        else if (r\[i\] == 1)  
        {  
          // 如果 base 还不存在，则将其作为 base，这种可能性更高  
          if (op->base < 0)  
            op->base = i;  
          // 否则，index 未指定，则作为 index 用，并且 scale 为 1  
          else if (op->index < 0)  
          {  
            op->index = i;  
            op->scale = 1;  
          }  
          // 而如果既指定了 index 又指定了 base，那么说明扫描到太多的寄存器，出错  
          else  
          {  
            asmerror = "Too many registers";  
            scan = SCAN\_ERR;  
            return;  
          };  
        }  
        // 错误的 scale  
        else   
        {  
          asmerror = "Invalid scale";  
          scan = SCAN\_ERR;  
          return;  
        };  
        // 统计用于地址的寄存器数  
        j++;  
      };  
      // 操作数包含了 ModRM 寻址信息  
      op->type = MRG;  
    };   
  }  
  // 否则，说明是系统不支持的操作数  
  else   
  {  
    asmerror = "Unrecognized operand";  
    scan = SCAN\_ERR;  
    return;  
  };  
  // In general, address modifier is allowed only with address expression which  
  // is a constant, a far address or a memory expression. More precise check  
  // will be done later in Assemble().  
  // 如果是一个 JMP 指令，并且操作数不是直接数，并且也不是地址，并且也不未包含 ModRM 的地址，则出错  
  if (op->jmpmode != 0 && op->type != IMM && op->type != JMF && op->type != MRG)  
  {  
    asmerror = "Jump address modifier is not allowed";  
    scan = SCAN\_ERR;   
    return;  
  };  
  // 扫描下一符号  
  Scanasm(0);    // Fetch next token from input line  
};  
  
// Function assembles text into 32-bit 80x86 machine code. It supports imprecise 不精确的，不确定的  
// operands 操作数 (for example, R32 stays for any general-purpose 32-bit register).  
// This allows to search for incompletei 不完全的 commands. Command is precise when all  
// significant bytes in model.mask are 0xFF. Some commands have more than one  
// decoding. By calling Assemble() with attempt=0,1... and constsize=0,1,2,3 one  
// gets also alternative variants (bit 0x1 of constsize is responsible for size  
// of address constant and bit 0x2 - for immediate data). However, only one  
// address form is generated (\[EAX\*2\], but not \[EAX+EAX\]; \[EBX+EAX\] but not  
// \[EAX+EBX\]; \[EAX\] will not use SIB byte; no DS: prefix and so on). Returns  
// number of bytes in assembled code or non-positive number in case of detected  
// error. This number is the negation of the offset in the input text where the  
// error encountered. Unfortunately, BC 4.52 is unable to compile the switch  
// (arg) in this code when any common subexpression optimization is on. The  
// next #pragma statement disables all optimizations.  
  
//#pragma option -Od                     // No optimizations, or BC 4.52 crashes  
  
// 将 cmd 编译为机器指令并通过 model 返回  
  
// errtext：返回编译的出错信息  
  
// 返回值：指令长度  
  
int Assemble(char \* cmd, ulong ip, t\_asmmodel \* model, int attempt, int constsize, char \* errtext)  
{  
  // bytesize：为 1 表示只允许操作数为 1 个字节长度  
  int i, j, k, namelen, nameok, arg, match, datasize, addrsize, bytesize, minop, maxop;  
  int rep, lock, segment, jmpsize, jmpmode, longjump;  
  int hasrm, hassib, dispsize, immsize;  
  int anydisp, anyimm, anyjmp;  
  long l, displacement, immediate, jmpoffset = 0;  
  
  // name：存储指令完整助记符，包括指令前缀（如 REP）与助记符  
  // nameend：指向指令助记符（指令名）的末尾  
  char name\[32\], \* nameend;  
  
  // 存储编译后的指令机器码，长度不超 15 字节  
  char tcode\[MAXCMDSIZE\], tmask\[MAXCMDSIZE\];  
  
  // 存储指令 3 个操作数信息，指令最多可以有 3 个操作数  
  t\_asmoperand aop\[3\], \*op;      // Up to 3 operands allowed  
  const t\_cmddata \* pd;  
  
  // 代码长度为 0  
  if (model != NULL)   
    model->length = 0;  
  // 检测参数的合法性  
  if (cmd == NULL || model == NULL || errtext == NULL)  
  {  
    // 参数出错，则保存错误信息并返回  
    if (errtext != NULL)   
      strcpy(errtext, "Internal OLLYDBG error");  
    return 0;  
  };        // Error in parameters  
  // 保存欲编译的汇编指令  
  asmcmd = cmd;  
  
  // 指令是否有 REP 或 LOCK 前缀  
  rep = lock = 0;  
  errtext\[0\] = '\\0';  
  
  // 扫描指令  
  Scanasm(SA\_NAME);  
  
  // 指令结束，则不需要做任何操作  
  if (scan == SCAN\_EOL)    // End of line, nothing to assemble  
    return 0;  
  
  // 扫描指令前缀  
  while (1)  
  {  
    // Fetch all REPxx and LOCK prefixes  
    // 如果是 REP 或 REPE 或 REPNE 前缀，则...  
    if (scan == SCAN\_REP || scan == SCAN\_REPE || scan == SCAN\_REPNE)   
    {  
      // 指令的 REP 前缀重复，出错返回  
      if (rep != 0)  
      {  
        strcpy(errtext, "Duplicated REP prefix");   
        goto error;  
      };  
      rep = scan;  
    }  
    // 否则，如果是 LOCK 前缀（用于加锁总线）  
    else if (scan == SCAN\_LOCK)  
    {  
      // 同样，如果 LOCK 前缀重复，也出错返回  
      if (lock != 0)  
      {  
        strcpy(errtext, "Duplicated LOCK prefix");  
        goto error;  
      };  
      lock = scan;  
    }  
    else  
      // 未发现更多前缀，跳出循环  
      // No more prefixes  
      break;  
    // 往下扫描  
    Scanasm(SA\_NAME);  
  };  
  
  // 扫描不到指令助记符（即指令名），则出错  
  if (scan != SCAN\_NAME || idata > 16)  
  {  
    strcpy(errtext,"Command mnemonic 助记符 expected");  
    goto error;  
  };  
  
  // 指向指令结束符并将指令转为大写  
  nameend = asmcmd;  
  strupr(sdata);  
  
  // Prepare full mnemonic (including repeat prefix, if any).  
  // 与前缀组合成完整助记符  
  if (rep == SCAN\_REP)  
    sprintf(name, "REP %s", sdata);  
  else if (rep == SCAN\_REPE)  
    sprintf(name, "REPE %s", sdata);  
  else if (rep == SCAN\_REPNE)  
    sprintf(name, "REPNE %s", sdata);  
  else   
    strcpy(name, sdata);  
  
  // 至此，name 可能如：REPX 指令助记符  
  
  // 往下扫描指令的操作数  
  Scanasm(0);  
  
  // 接下来需要解析命令的操作数（最多 3 个）  
  
  // Parse command operands (up to 3). Note: jump address is always  
  // the first (and only) operand in actual command set.  
  // 初始化 3 个操作数结构信息  
  for (i = 0; i < 3; i++)   
  {  
    aop\[i\].type = NNN;      // No operand  
    aop\[i\].size = 0;      // Undefined size  
    aop\[i\].index = -1;      // No index  
    aop\[i\].scale = 0;      // No scale  
    aop\[i\].base = -1;      // No base  
    aop\[i\].offset = 0;      // No offset  
    aop\[i\].anyoffset = 0;    // No offset  
    aop\[i\].segment = SEG\_UNDEF;  // No segment  
    aop\[i\].jmpmode = 0;  
  };  // No jump size modifier  
  
  // 解析第一个操作数  
  Parseasmoperand(aop + 0);  
  
  // 获取跳转模式  
  jmpmode = aop\[0\].jmpmode;  
  // 如果存在跳转模式，则表示是一跳转指令，最高位置 1  
  if (jmpmode != 0)  
    jmpmode |= 0x80;  // 10000000b  
  
  // 在 Parseasmoperand()函数中会调用 Scanasm()往下扫描指令  
  
  // 第 1 个操作数后是 ','符号，则扫描第 2 个操作数并解析之  
  if (scan == SCAN\_SYMB && idata == ',')  
  {  
    // 扫描第 2 个操作数并解析之  
    Scanasm(0);  
    Parseasmoperand(aop + 1);  
  
    // 还存在第 3 个操作数，则继续扫描之  
    if (scan == SCAN\_SYMB && idata == ',')   
    {  
      // 扫描第 3 个操作数并解析之  
      Scanasm(0);  
      Parseasmoperand(aop + 2);  
    };  
  };  
  // 扫描出错，获取出错信息并返回  
  if (scan == SCAN\_ERR)  
  {  
    strcpy(errtext, asmerror);  
    goto error;  
  };  
  // 操作数结束后，指令还存在其他内容，那就奇怪了  
  if (scan != SCAN\_EOL)  
  {  
    strcpy(errtext, "Extra input after operand");  
    goto error;  
  };  
  
  // If jump size is not specified, function tries to use short jump. If  
  // attempt fails, it retries with long form.  
  
  // 如果跳转指令的跳转地址大小（如 SHORT，FAR）未指令，则函数尝试使用 SHORT  
  // 如果尝试失败，则尝试 LONG 跳转  
  
  longjump = 0;      // Try short jump on the first pass  
retrylongjump:  
  nameok = 0;  
  // Some commands allow different number of operands. Variables minop and  
  // maxop accumulate their minimal and maximal counts. The numbers are not  
  // used in assembly process but allow for better error diagnostics.  
  // 指令最多、最少操作数数量，用于限制指令的操作数个数  
  minop = 3;  
  maxop = 0;  
  // Main assembly loop: try to find the command which matches all operands,  
  // but do not process operands yet.  
  
  // 现在开始检测上面扫描到的指令与操作数是否匹配，是否 OK  
  
  // 指令长度  
  namelen = (int)strlen(name);  
  
  // 遍历处理器有效的（机器）指令列表，以检测上面扫描的指令是否 OK  
  for (pd = cmddata; pd->mask != 0; pd++)  
  {  
    // 如果指令助记符以&开始，则对助记符进行译码依赖于操作数长度  
    // 例如：  
    //    &PUSHA\*  
    //    &CBW:CWDE  
    if (pd->name\[0\] == '&')  
    {  
      // Mnemonic 助记符 depends on operand size  
      // 跳过助记符前的&符号  
      j = 1;  
      // 数据长度默认为 16 位  
      datasize = 2;  
      // 地址长度默认为 32 位  
      addrsize = 4;  
      while (1)  
      {  
        // 某些指令可以有多个助记符，每个助记符以 ：分开，所以检测时需对所有助记符进行匹配  
        // Try all mnemonics (separated by ':')  
        for (i = 0; pd->name\[j\] != '\\0' && pd->name\[j\] != ':'; j++)  
        {  
          // 指令助记符名称中，可以包含有字符'W'或'D'，用于表示指令的操作数长度，这种指令  
          // 的译码结果往往操作数的长度起到很重要的决定作用  
  
          // 如果助记符有\*符号，则表示该助记符中在\*号位  
          // 置可能包含'W'或'D'以指示操作数长度  
          if (pd->name\[j\] == '\*')  
          {  
            // 如果扫描到的助记符包含'W'，说明操作数长度为 16 位（字）  
            if (name\[i\] == 'W')  
            {  
              datasize = 2;  
              i++;  
            }  
            // 如果扫描到的助记符包含'D'，说明操作数长度为 32 位（双字）  
            else if (name\[i\] == 'D')  
            {  
              datasize = 4;  
              i++;  
            }  
            // 如果没通过'W'或'D'指明操作数长度，则根据 sizesens 处理  
            else if (sizesens == 0)   
              datasize = 2;  
            else  
              datasize = 4;  
          }  
          // 检测 pd 助记符与扫描的助记符是否匹配  
          else if (pd->name\[j\] == name\[i\])  
            i++;  
          else  
            break;  
        };  
        // 如果扫描的助记符与 pd 列表助记符相符，则找到指令信息，跳出循环  
        if (name\[i\] == '\\0' && (pd->name\[j\] == '\\0' || pd->name\[j\] == ':'))  
          break;        // Bingo!  
  
        // 否则，说明助记符与 pd 不符，则需要搜索下一 pd，如上例子  
        // &CBW:CWDE，搜索 CBW 不符后，则需要搜索 CWDE 指令看是否相符  
        while (pd->name\[j\] != '\\0' && pd->name\[j\] != ':')  
          j++;  
  
        // 如果助记符确实以&CBW:CWDE 出现，则需要检测：后的助记符是否与扫描到的助记符是否相符  
        // 注：以 : 作为助记符分割的指令，一般前部分是 16 位指令，而后部分是 32 位指令  
        if (pd->name\[j\] == ':')  
        {  
          j++;  
          datasize = 4;  
        }           // Retry with 32-bit mnenonic  
        // 否则，只好搜索下一 pd 指令了  
        else  
        {  
          i = 0;   
          break;                  // Comparison failed  
        };  
      };  
      if (i == 0)   
        continue;  
    }  
    // 否则，如果指令助记符以 $ 开始，则对助记符进行译码依赖于地址长度  
    // 此处操作同上，略  
    else if (pd->name\[0\] == '$')  // Mnemonic depends on address size  
    {  
      // 跳过 $ 符号  
      j = 1;  
      datasize=0;  
      addrsize=2;  
      while (1)  
      {  
        // Try all mnemonics (separated by ':')  
        for (i = 0; pd->name\[j\] != '\\0' && pd->name\[j\] != ':'; j++)  
        {  
          if (pd->name\[j\] == '\*')  
          {  
            // 如果地址大小为 16 位  
            if (name\[i\] == 'W')  
            {  
              addrsize = 2;  
              i++;  
            }  
            // 否则，如果为 32 位  
            else if (name\[i\] == 'D')   
            {   
              addrsize = 4;  
              i++;  
            }  
            else if (sizesens == 0)   
              addrsize = 2;  
            else   
              addrsize = 4;   
          }  
          else if (pd->name\[j\] == name\[i\])   
            i++;  
          else   
            break;  
        };  
        if (name\[i\] == '\\0' && (pd->name\[j\] == '\\0' || pd->name\[j\] == ':'))  
          break;            // Bingo!  
        while (pd->name\[j\] != '\\0' && pd->name\[j\] != ':')  
          j++;  
        if (pd->name\[j\] == ':')  
        {  
          j++;  
          addrsize = 4;  
        }              // Retry with 32-bit mnenonic  
        else  
        {  
          i = 0;  
          break;          // Comparison failed  
        };  
      };  
      if (i == 0)  
        continue;  
    }  
    // 否则，匹配其他指令助记符  
    else    // Compare with all synonimes  
    {  
      j = k = 0;  
      datasize = 0;      // Default settings  
      addrsize = 4;  
      // 检测扫描的助记符是否与 pd 匹配  
      while (1)  
      {  
        // 有些指令有多个助记符，用','号分开  
        // 例如：JLE,JNG--以逗号分割两个指令  
        while (pd->name\[j\] != ',' && pd->name\[j\] != '\\0')   
          j++;  
        if (j - k == namelen && strnicmp(name, pd->name + k, namelen) == 0)  
          break;  
        // 如果前一助记符不符，则比较后面的助记符是否相符  
        k = j + 1;  
        // 已经比较完整个 pd 指令，则结束本环比较  
        if (pd->name\[j\] == '\\0')  
          break;  
        j = k;  
      };  
      if (k > j)  
        continue;  
    };  
  
    // 至此，说明找到扫描助记符 name 的 pd 指令信息  
  
    // For error diagnostics 错误诊断法 it is important to know whether mnemonic exists.  
    // 累计找到的 pd 指令信息  
    nameok++;  
  
    // 统计助记符的操作数个数  
  
    // 根据 pd 指令信息，判断该指令的操作数个数  
    if (pd->arg1 == NNN || pd->arg1 >= PSEUDOOP)  
      minop = 0;  
    // 否则，如果只有 1 个操作数  
    else if (pd->arg2 == NNN || pd->arg2 >= PSEUDOOP)  
    {  
      if (minop > 1)  
        minop = 1;  
      if (maxop < 1)  
        maxop = 1;  
    }  
    // 两个操作数  
    else if (pd->arg3 == NNN || pd->arg3 >= PSEUDOOP)  
    {  
      // 限制其只有两个操作数  
      if (minop > 2)  
        minop = 2;  
      if (maxop < 2)  
        maxop = 2;  
    }  
    // 最多只能有 3 个操作数  
    else  
      maxop = 3;  
  
    // Determine default and allowed operand size(s).  
  
    // 在汇编语言中，数据长度有两个概念：默认长度与有效长度  
  
    // 确定默认和允许的操作数长度  
    // Forced 16-bit size  
    // FF：表示强制 16 位大小  
    if (pd->bits == FF)  
      datasize = 2;    // Forced 16-bit size  
  
    // 检测指令是否只允许使用 1 字节长度的操作数  
    // 注：此处的检测逻辑可借鉴指令的编码构造文章  
    if (pd->bits == WW || pd->bits == WS || pd->bits == W3 || pd->bits == WP)  
      bytesize = 1;      // 1-byte size allowed  
    else  
      bytesize = 0;      // Word/dword size only  
  
    // Check whether command operands match specified. If so, variable match  
    // remains zero, otherwise it contains kind of mismatch. This allows for  
    // better error diagnostics 错误诊断法.  
    // 检测命令操作数是否 OK，如 OK，则 match 保持为 0，否则，包含不匹配的信息  
    match = 0;  
    // 检测扫描到的操作数是否 OK  
    for (j = 0; j < 3; j++)    // Up to 3 operands  
    {  
      // 获取操作数 j  
      op = aop + j;  
      // 获取指令 pd 的操作数类型  
      if (j == 0)  
        arg = pd->arg1;  
      else if (j == 1)  
        arg = pd->arg2;  
      else   
        arg = pd->arg3;  
  
      // 如果指令 pd 没有操作数 j  
      if (arg == NNN || arg >= PSEUDOOP)  
      {  
        // 而助记符 name 却带有该操作数，则出错  
        if (op->type != NNN)    // No more arguments  
          // 表示助记符操作数个数有误  
          match |= MA\_NOP;  
        break;  
      };  
      // 至此，则说明指令 pd 有操作数 j，但助记符 name 却没带操作数  
      if (op->type == NNN)  
      {  
        match |= MA\_NOP;  
        break;  
      };    // No corresponding operand  
  
      // 否则，至此，说明带操作数，并且相符，则需要进一步操作了  
  
      // 根据所带操作数处理，主要是根据 pd 操作数检测扫描的助记符操作数是否 OK  
      switch (arg)  
      {  
        // 寄存器操作数  
      case REG:          // Integer register in Reg field  
      case RCM:          // Integer register in command byte  
        // 使用累加器寄存器  
      case RAC:          // Accumulator (AL/AX/EAX, implicit)  
        // 如果助记符使用的不是寄存器操作数，则类型出错  
        if (op->type != REG)  
          match |= MA\_TYP;  
        // 如果 pd 指令操作数使用累加寄存器（即 AL、AX 等），而扫描到的助记符使用的  
        // 操作数寄存器有误，则说明操作数类型有误  
        if (arg == RAC && op->index != REG\_EAX && op->index != 8)  
          match |= MA\_TYP;  
  
        // bytesize == 0 -> Word/dword size only  
        // 如果 pd 指令只使用 word/dword 操作数，而助记符却使用 byte，则操作数大小出错  
        if (bytesize == 0 && op->size == 1)  
          match |= MA\_SIZ;  
        // 如果 pd 没有指定数据长度，则设置为扫描到的助记符的操作数长度  
        if (datasize == 0)  
          datasize = op->size;  
        // 操作数长度不同  
        if (datasize != op->size)  
          match |= MA\_DIF;  
        break;  
  
        // 4 字节长度的整型寄存器  
      case RG4:          // Integer 4-byte register in Reg field  
        // 如果助记符操作数不是寄存器，则类型出错  
        if (op->type != REG)  
          match |= MA\_TYP;  
        // 如果操作数长度匹配，则出错  
        if (op->size != 4)  
          match |= MA\_SIZ;  
        // 保存操作数长度值  
        if (datasize == 0)  
          datasize = op->size;  
        if (datasize != op->size)  
          match |= MA\_DIF;  
        break;  
  
        // AX 寄存器，16 位  
      case RAX:          // AX (2-byte, implicit)  
        // 如果扫描到的助记符操作数不是 EAX 寄存器则出错  
        if (op->type != REG || (op->index != REG\_EAX && op->index != 8))  
          match |= MA\_TYP;  
        // 不是 16 位寄存器，即不是 AX，则出错  
        if (op->size != 2)   
          match |= MA\_SIZ;  
        if (datasize == 0)   
          datasize = op->size;  
        if (datasize != op->size)  
          match |= MA\_DIF;  
        break;  
  
        // DX 寄存器，一般作为端口地址  
      case RDX:          // DX (16-bit implicit port address)  
        // 判断同上  
        if (op->type != REG || (op->index != REG\_EDX && op->index != 8))  
          match |= MA\_TYP;  
        if (op->size != 2)  
          match |= MA\_SIZ;   
        break;  
  
        // CL 寄存器，一般作为移位用  
      case RCL:          // Implicit CL register (for shifts)  
        if (op->type != REG || (op->index != REG\_ECX && op->index != 8))  
          match |= MA\_TYP;  
        if (op->size != 1)   
          match |= MA\_SIZ;  
        break;  
  
        // FPU 堆栈顶寄存器  
      case RS0:          // Top of FPU stack (ST(0))  
        if (op->type != RST || (op->index != 0 && op->index != 8))  
          match |= MA\_TYP;  
        break;  
  
        // FPU 堆栈寄存器  
      case RST:          // FPU register (ST(i)) in command byte  
        if (op->type != RST)  
          match |= MA\_TYP;  
        break;  
  
        // MMX 寄存器  
      case RMX:          // MMX register MMx  
        // 3D 寄存器  
      case R3D:          // 3DNow! register MMx  
        if (op->type != RMX)  
          match |= MA\_TYP;  
        break;  
  
        // 操作数是内存操作数或寄存器操作数，并且该操作数由 ModRM 提供寻址  
        // 例如：MOV MRG, REG  
      case MRG:          // Memory/register in ModRM byte  
        // 如果扫描到的助记符操作数不是 MRG，并且也不是寄存器，则出错  
        if (op->type != MRG && op->type != REG)   
          match |= MA\_TYP;  
        // 如果操作数只允许使用 16 位或 32 位，而扫描到的助记符使用 1 字节，而长度出错  
        if (bytesize == 0 && op->size == 1)  
          match |= MA\_SIZ;  
        if (datasize == 0)  
          datasize = op->size;  
        if (op->size != 0 && op->size != datasize)   
          match |= MA\_DIF;  
        break;  
  
        // 同上，操作数由 ModRM 提供寻址，并且长度为 1 字节  
      case MR1:          // 1-byte memory/register in ModRM byte  
        if (op->type != MRG && op->type != REG)  
          match |= MA\_TYP;  
        if (op->size != 0 && op->size != 1)  
          match |= MA\_SIZ;  
        break;  
  
        // 同上，操作数由 ModRM 提供寻址，并且长度为 2 字节  
      case MR2:          // 2-byte memory/register in ModRM byte  
        if (op->type != MRG && op->type != REG)  
          match |= MA\_TYP;  
        if (op->size != 0 && op->size != 2)  
          match |= MA\_SIZ;  
        break;  
  
        // 同上，操作数由 ModRM 提供寻址，并且长度为 4 字节  
      case MR4:          // 4-byte memory/register in ModRM byte  
        if (op->type != MRG && op->type != REG)  
          match |= MA\_TYP;  
        if (op->size != 0 && op->size != 4)  
          match |= MA\_SIZ;  
        break;  
  
        // 操作数是 4 字节的寄存器  
      case RR4:          // 4-byte memory/register (register only)  
        if (op->type != REG)  
          match |= MA\_TYP;  
        if (op->size != 0 && op->size != 4)  
          match |= MA\_SIZ;  
        break;  
        
        // 操作数是 JMP 的跳转目标地址，并且由 ModRM 提供寻址  
        // 例如：CALL MRJ  
      case MRJ:          // Memory/reg in ModRM as JUMP target  
        if (op->type != MRG && op->type != REG)  
          match |= MA\_TYP;  
        // 长度必须为 4 个字节  
        if (op->size != 0 && op->size != 4)   
          match |= MA\_SIZ;  
        // Invalid jump size modifier  
        if ((jmpmode & 0x09) != 0)  
          match |= MA\_JMP;  
        jmpmode &= 0x7F;    // 1111111b  
        break;  
  
        // 对于 8 字节操作数不关心  
      case MR8:          // 8-byte memory/MMX register in ModRM  
      case MRD:          // 8-byte memory/3DNow! register in ModRM  
        if (op->type != MRG && op->type != RMX)  
          match |= MA\_TYP;  
        if (op->size != 0 && op->size != 8)   
          match |= MA\_SIZ;  
        break;  
      case RR8:          // 8-byte MMX register only in ModRM  
      case RRD:          // 8-byte memory/3DNow! (register only)  
        if (op->type != RMX)  
          match |= MA\_TYP;  
        if (op->size != 0 && op->size != 8)   
          match |= MA\_SIZ;  
        break;  
  
      case MMA:          // Memory address in ModRM byte for LEA  
        if (op->type != MRG)  
          match |= MA\_TYP;  
        break;  
      case MML:          // Memory in ModRM byte (for LES)  
        if (op->type != MRG)  
          match |= MA\_TYP;  
        if (op->size != 0 && op->size != 6)  
          match |= MA\_SIZ;  
        if (datasize == 0)  
          datasize = 4;  
        else if (datasize != 4)   
          match |= MA\_DIF;  
        break;  
      case MMS:          // Memory in ModRM byte (as SEG:OFFS)  
        if (op->type != MRG)  
          match |= MA\_TYP;  
        if (op->size != 0 && op->size != 6)  
          match |= MA\_SIZ;  
        // MA\_JMP: Invalid jump size modifier  
        if ((jmpmode & 0x07) != 0)  
          match |= MA\_JMP;  
        jmpmode &= 0x7F;  
        break;  
      case MM6:          // Memory in ModRm (6-byte descriptor)  
        if (op->type != MRG)  
          match |= MA\_TYP;  
        if (op->size != 0 && op->size != 6)  
          match |= MA\_SIZ;  
        break;  
      case MMB:          // Two adjacent memory locations (BOUND)  
        if (op->type != MRG)  
          match |= MA\_TYP;  
        k = op->size;   
        if (ideal == 0 && k > 1)   
          k /= 2;  
        if (k != 0 && k != datasize)  
          match |= MA\_DIF;  
        break;  
      case MD2:          // Memory in ModRM byte (16-bit integer)  
      case MB2:          // Memory in ModRM byte (16-bit binary)  
        if (op->type != MRG)   
          match |= MA\_TYP;  
        if (op->size != 0 && op->size != 2)  
          match |= MA\_SIZ;  
        break;  
      case MD4:          // Memory in ModRM byte (32-bit integer)  
      case MF4:          // Memory in ModRM byte (32-bit float)  
        if (op->type != MRG)  
          match |= MA\_TYP;  
        if (op->size != 0 && op->size != 4)   
          match |= MA\_SIZ;  
        break;  
      case MD8:          // Memory in ModRM byte (64-bit integer)  
      case MF8:          // Memory in ModRM byte (64-bit float)  
        if (op->type != MRG)  
          match |= MA\_TYP;  
        if (op->size != 0 && op->size != 8)  
          match |= MA\_SIZ;  
        break;  
      case MDA:          // Memory in ModRM byte (80-bit BCD)  
      case MFA:          // Memory in ModRM byte (80-bit float)  
        if (op->type != MRG)   
          matc h|= MA\_TYP;  
        if (op->size != 0 && op->size != 10)  
          match |= MA\_SIZ;  
        break;  
      case MFE:          // Memory in ModRM byte (FPU environment)  
      case MFS:          // Memory in ModRM byte (FPU state)  
      case MFX:          // Memory in ModRM byte (ext. FPU state)  
        if (op->type != MRG)   
          match |= MA\_TYP;  
        if (op->size != 0)  
          match |= MA\_SIZ;  
        break;  
      case MSO:          // Source in string operands (\[ESI\])  
        if (op->type != MRG || op->base != REG\_ESI || op->index != -1 ||  
          op->offset != 0 || op->anyoffset != 0)  
          match |= MA\_TYP;  
        if (datasize == 0)  
          datasize = op->size;  
        if (op->size != 0 && op->size != datasize)   
          match |= MA\_DIF;  
        break;  
      case MDE:          // Destination in string operands (\[EDI\])  
        if (op->type != MRG || op->base != REG\_EDI || op->index != -1 ||  
          op->offset != 0 || op->anyoffset != 0)  
          match |= MA\_TYP;  
        if (op->segment != SEG\_UNDEF && op->segment != SEG\_ES)  
          match |= MA\_SEG;  
        if (datasize == 0)  
          datasize = op->size;  
        if (op->size != 0 && op->size != datasize)  
          match |= MA\_DIF;  
        break;  
      case MXL:          // XLAT operand (\[EBX+AL\])  
        if (op->type != MXL)   
          match |= MA\_TYP;  
        break;  
  
        // 操作数是一直接数  
      case IMM:          // Immediate data (8 or 16/32)  
      case IMU:          // Immediate unsigned data (8 or 16/32)  
        if (op->type != IMM)  
          match |= MA\_TYP;  
        break;  
  
      case VXD:          // VxD service (32-bit only)  
        if (op->type != IMM)  
          match |= MA\_TYP;  
        if (datasize == 0)  
          datasize = 4;  
        if (datasize != 4)   
          match |= MA\_SIZ;  
        break;  
  
        // jump/call 的直接数地址  
      case JMF:          // Immediate absolute far jump/call addr  
        if (op->type != JMF)  
          match |= MA\_TYP;  
        // 地址修饰出错  
        if ((jmpmode & 0x05) != 0)  
          match |= MA\_JMP;  
        jmpmode &= 0x7F;  
        break;  
      case JOB:          // Immediate byte offset (for jumps)  
        if (op->type != IMM || longjump)  
          match |= MA\_TYP;  
        if ((jmpmode & 0x0A) != 0)  
          match |= MA\_JMP;  
        jmpmode &= 0x7F;  
        break;  
      case JOW:          // Immediate full offset (for jumps)  
        if (op->type != IMM)  
          match |= MA\_TYP;  
        if ((jmpmode & 0x09) != 0)   
          match |= MA\_JMP;  
        jmpmode &= 0x7F;  
        break;  
      case IMA:          // Immediate absolute near data address  
        if (op->type != MRG || op->base >= 0 || op->index >= 0)  
          match |= MA\_TYP;  
        break;  
      case IMX:          // Immediate sign-extendable byte  
        if (op->type != IMM)  
          match |= MA\_TYP;  
        if (op->offset < -128 || op->offset > 127)   
          match |= MA\_RNG;  
        break;  
      case C01:          // Implicit constant 1 (for shifts)  
        if (op->type != IMM || (op->offset != 1 && op->anyoffset == 0))  
          match |= MA\_TYP;  
        break;  
  
      case IMS:          // Immediate byte (for shifts)  
      case IM1:          // Immediate byte  
        if (op->type != IMM)  
          match |= MA\_TYP;  
        // 常数范围有误  
        if (op->offset < -128 || op->offset > 255)  
          match |= MA\_RNG;  
        break;  
      case IM2:          // Immediate word (ENTER/RET)  
        if (op->type != IMM)  
          match |= MA\_TYP;  
        if (op->offset < 0 || op->offset > 65535)  
          match |= MA\_RNG;  
        break;  
        // 段寄存器  
      case SGM:          // Segment register in ModRM byte  
        if (op->type != SGM)  
          match |= MA\_TYP;  
        if (datasize == 0)   
          datasize = 2;  
        if (datasize != 2)   
          match |= MA\_DIF;  
        break;  
        //  
      case SCM:          // Segment register in command byte  
        if (op->type != SGM)  
          match |= MA\_TYP;  
        break;  
        // 控制寄存器  
      case CRX:          // Control register CRx  
      case DRX:          // Debug register DRx  
        if (op->type != arg)  
          match |= MA\_TYP;  
        if (datasize == 0)   
          datasize = 4;  
        if (datasize != 4)   
          match |= MA\_DIF;  
        break;  
      case PRN:          // Near return address (pseudooperand)  
      case PRF:          // Far return address (pseudooperand)  
      case PAC:          // Accumulator (AL/AX/EAX, pseudooperand)  
      case PAH:          // AH (in LAHF/SAHF, pseudooperand)  
      case PFL:          // Lower byte of flags (pseudooperand)  
      case PS0:          // Top of FPU stack (pseudooperand)  
      case PS1:          // ST(1) (pseudooperand)  
      case PCX:          // CX/ECX (pseudooperand)  
      case PDI:          // EDI (pseudooperand in MMX extentions)  
        break;  
        // 错误操作数  
      default:          // Undefined type of operand  
        strcpy(errtext,"Internal Assembler error");  
        goto error;  
      };    // End of switch (arg)  
  
      // 在 switch(arg)中会对 jmpmode 进行 & 0x7F 的计算，所以其 0x80  
      // 位正常应该为 0，所以，如果不为 0，则说明出错了  
      if ((jmpmode & 0x80) != 0)  
        match |= MA\_JMP;  
      // 如果操作数不匹配，则出错  
      if (match != 0)  
        break;      // Some of the operands doesn't match  
    };    // End of operand matching loop  
  
    // 如果所有操作数都匹配  
    if (match == 0)      // Exact match found  
    {  
      // 允许尝试多次  
      if (attempt > 0)  
      {  
        --attempt;  
        nameok = 0;  
      }    // Well, try to find yet another match  
      else  
        break;  
    };  
  };    // End of command search loop : for (pd = cmddata; pd->mask != 0; pd++)  
  
  // 至此，对操作数的验证就结束了  
  
  // Check whether some error was detected. If several errors were found  
  // similtaneously, report one (roughly in order of significance).  
  // 如果助记符无效，则出错  
  if (nameok == 0)  // Mnemonic unavailable  
  {  
    strcpy(errtext,"Unrecognized command");  
    asmcmd = nameend;  
    goto error;   
  };  
  
  // 操作数不匹配，则出错  
  if (match != 0)    // Command not found  
  {  
    if (minop > 0 && aop\[minop - 1\].type == NNN)  
      strcpy(errtext, "Too few operands");  
    else if (maxop < 3 && aop\[maxop\].type != NNN)  
      strcpy(errtext, "Too many operands");  
    // 指令包含了不止一个命令  
    else if (nameok > 1)      // More that 1 command  
      strcpy(errtext, "Command does not support given operands");  
    else if (match & MA\_JMP)  
      strcpy(errtext, "Invalid jump size modifier");  
    else if (match & MA\_NOP)  
      strcpy(errtext, "Wrong number of operands");  
    else if (match & MA\_TYP)  
      strcpy(errtext, "Command does not support given operands");  
    else if (match & MA\_NOS)  
      strcpy(errtext, "Please specify operand size");  
    else if (match & MA\_SIZ)  
      strcpy(errtext, "Bad operand size");  
    else if (match & MA\_DIF)  
      strcpy(errtext, "Different size of operands");  
    else if (match & MA\_SEG)  
      strcpy(errtext, "Invalid segment register");  
    else if (match & MA\_RNG)  
      strcpy(errtext, "Constant out of expected range");  
    else  
      strcpy(errtext, "Erroneous command");  
    goto error;  
  };  
  
  // Exact match found. Now construct the code.  
  
  // 至此，命令是 OK 的，包括操作数，则需要解码该指令了  
  
  // 指令是否包含 ModR/M 字节  
  hasrm = 0;      // Whether command has ModR/M byte  
  // 指令是否包含 SIB 字节  
  hassib = 0;      // Whether command has SIB byte  
  // displacement 的长度（如果有）  
  dispsize = 0;    // Size of displacement (if any)  
  // immediate 直接数的长度（如果有）  
  immsize = 0;    // Size of immediate data (if any)  
  // 指令段前缀  
  segment = SEG\_UNDEF;  // Necessary segment prefix  
  //   
  jmpsize = 0;    // No relative jumps  
  
  // tcode：应是用于存储解码后的机器指令，最长指令不超 15 字节  
  memset(tcode, 0, sizeof(tcode));  
  
  // 获取指令机器码  
  \*(ulong \*)tcode = pd->code & pd->mask;  
  memset(tmask, 0, sizeof(tmask));  
  
  \*(ulong \*)tmask = pd->mask;  
  // 指令最后字节的编号  
  // len：指令的长度（不包括操作数），一般为单字节指令  
  i = pd->len - 1;      // Last byte of command itself  
  
  // 如果指令包含 REP 前缀，则长度 + 1  
  if (rep)  
    i++;        // REPxx prefixes count as extra byte  
  
  // 指令 w 位说明：  
  // 指令的最低位为 w 位或最低 3 位为 w  
  //    当 w = 0 时：operand size 是 byte   
  //    当 w = 1 时：operand size 是 default operand size  
  
  // In some cases at least one operand must have explicit size declaration (as  
  // in MOV \[EAX\],1). This preliminary check does not include all cases.  
  // 在一些情况下，操作数必须明确指定其大小，但并不是所有情况都需要  
  // 如果是最低位为 w 位  
  if (pd->bits == WW || pd->bits == WS || pd->bits == WP)  
  {  
    // 如果是 WW || WS || WP，则需要指定操作数大小  
    if (datasize == 0)  
    {  
      strcpy(errtext, "Please specify operand size");  
      goto error;  
    }  
    // 操作数是 word / dword  
    else if (datasize > 1)  
      // 置 w 位为 1，表示操作数大小为 default operand size  
      tcode\[i\] |= 0x01;    // WORD or DWORD size of operands  
    // 表示 i 字节将会是操作数长度 \[??\]  
    tmask\[i\] |= 0x01;  
  }  
  // 第 3 位为 w 位  
  // Bit W at position 3  
  else if (pd->bits == W3)  
  {  
    if (datasize == 0)  
    {  
      strcpy(errtext, "Please specify operand size");  
      goto error;  
    }  
    // 置 w 位为 1，表示操作数大小为 default operand size  
    else if (datasize > 1)  
      tcode\[i\] |= 0x08;  // WORD or DWORD size of operands  
    tmask\[i\] |= 0x08;  
  };  
  
  // Present suffix 后缀 of 3DNow! command as immediate byte operand.  
  if ((pd->type & C\_TYPEMASK) == C\_NOW)  
  {  
    immsize = 1;  
    immediate = (pd->code >> 16) & 0xFF;  
  };  
  
  // Process operands again, this time constructing the code.  
  // 再次处理操作数，并构建指令  
  
  anydisp = anyimm = anyjmp = 0;  
    
  for (j = 0; j < 3; j++)    // Up to 3 operands  
  {  
    // 获取操作数  
    op = aop + j;  
    if (j == 0)  
      arg = pd->arg1;  
    else if  
      (j == 1)  
      arg = pd->arg2;  
    else  
      arg = pd->arg3;  
  
    // 所有的操作数都处理了，则跳出循环  
    if (arg == NNN)  
      break;      // All operands processed  
  
    // 指令的格式：  
    // Legacy Prefix + REX prefix + Opcode + ModRM + SIB + Displacement + Immediate  
  
    // 根据操作数处理  
    switch (arg)  
    {  
      // 处理寄存器操作数  
    case REG:      // Integer register in Reg field  
    case RG4:      // Integer 4-byte register in Reg field  
    case RMX:      // MMX register MMx  
    case R3D:      // 3DNow! register MMx  
    case CRX:      // Control register CRx  
    case DRX:      // Debug register DRx  
      // 只要出现寄存器操作数，那么指令就会包含 ModRM 字节  
      // 注：ModRM 的格式：mod reg r/m   
      hasrm = 1;  
      // 指定明确的寄存器作为操作数  
      if (op->index < 8)  
      {  
        // 组织 ModRM 字节以指明寄存器操作数  
        tcode\[i + 1\] |= (char)(op->index << 3);  
        tmask\[i + 1\] |= 0x38;  // 111000  
      };  
      break;  
  
      // 整数寄存器或 FPU 寄存器，则可以将寄存器编码至指令中  
      // 这种指令使用低 3 位来指明指令使用的寄存器  
    case RCM:      // Integer register in command byte  
    case RST:      // FPU register (ST(i)) in command byte  
      //   
      if (op->index < 8)  
      {  
        // 将寄存器号嵌入指令中  
        tcode\[i\] |= (char)op->index;   
        tmask\[i\] |= 0x07;  
      };  
      break;  
  
    case RAC:      // Accumulator (AL/AX/EAX, implicit)  
    case RAX:      // AX (2-byte, implicit)  
    case RDX:      // DX (16-bit implicit port address)  
    case RCL:      // Implicit CL register (for shifts)  
    case RS0:      // Top of FPU stack (ST(0))  
    case MDE:      // Destination in string op's (\[EDI\])  
    case C01:      // Implicit constant 1 (for shifts)  
      break;      // Simply skip implicit operands  
  
    case MSO:      // Source in string op's (\[ESI\])  
    case MXL:      // XLAT operand (\[EBX+AL\])  
      if (op->segment != SEG\_UNDEF && op->segment != SEG\_DS)  
        segment = op->segment;  
      break;  
  
    case MRG:      // Memory/register in ModRM byte  
    case MRJ:      // Memory/reg in ModRM as JUMP target  
    case MR1:      // 1-byte memory/register in ModRM byte  
    case MR2:      // 2-byte memory/register in ModRM byte  
    case MR4:      // 4-byte memory/register in ModRM byte  
    case RR4:      // 4-byte memory/register (register only)  
    case MR8:      // 8-byte memory/MMX register in ModRM  
    case RR8:      // 8-byte MMX register only in ModRM  
    case MRD:      // 8-byte memory/3DNow! register in ModRM  
    case RRD:      // 8-byte memory/3DNow! (register only)  
      // 表示存在 ModRM 字节  
      hasrm = 1;  
      // 如果非 MRG，则表示是寄存器寻址  
      if (op->type != MRG)  // Register in ModRM byte  
      {  
        // 11000000：代表寄存器寻址  
        tcode\[i + 1\] |= 0xC0;  // 11000000  
        tmask\[i + 1\] |= 0xC0;  
        // 明确指明寄存器，则设置寄存器号  
        if (op->index < 8)  
        {  
          tcode\[i + 1\] |= (char)op->index;  
          tmask\[i + 1\] |= 0x07;  
        };  
        break;  
      };    // Note: NO BREAK, continue with address  
  
      // 注意：这里没有 break，继续处理地址信息  
  
    case MMA:        // Memory address in ModRM byte for LEA  
    case MML:        // Memory in ModRM byte (for LES)  
    case MMS:        // Memory in ModRM byte (as SEG:OFFS)  
    case MM6:        // Memory in ModRm (6-byte descriptor)  
    case MMB:        // Two adjacent memory locations (BOUND)  
    case MD2:        // Memory in ModRM byte (16-bit integer)  
    case MB2:        // Memory in ModRM byte (16-bit binary)  
    case MD4:        // Memory in ModRM byte (32-bit integer)  
    case MD8:        // Memory in ModRM byte (64-bit integer)  
    case MDA:        // Memory in ModRM byte (80-bit BCD)  
    case MF4:        // Memory in ModRM byte (32-bit float)  
    case MF8:        // Memory in ModRM byte (64-bit float)  
    case MFA:        // Memory in ModRM byte (80-bit float)  
    case MFE:        // Memory in ModRM byte (FPU environment)  
    case MFS:        // Memory in ModRM byte (FPU state)  
    case MFX:        // Memory in ModRM byte (ext. FPU state)  
      hasrm = 1;  
      // 获取操作数的偏移地址  
      displacement = op->offset;  
      // 地址中是否存在 displacement 值  
      anydisp = op->anyoffset;  
        
      // 地址没有包含 base 与 index 部分  
      if (op->base < 0 && op->index < 0)  
      {  
        // 则直接数作为地址长度只有 32 位的情况了  
        dispsize = 4;    // Special case of immediate address  
        // 如果段前缀使用非默认段（即 DS）  
        if (op->segment != SEG\_UNDEF && op->segment != SEG\_DS)  
          segment = op->segment;  
  
        // 0x05：101，表示地址为 \[disp32\]  
        tcode\[i + 1\] |= 0x05;  
        tmask\[i + 1\] |= 0xC7;  
      }  
      // 如果地址没有包含 index 部分，并且 base 也不是 esp 寄存器，则不需要 SIB 字节来辅助寻址  
      // 格式：\[base + disp\]  
      else if (op->index < 0 && op->base != REG\_ESP)  
      {  
        // 0xC0：11000000b  
        tmask\[i + 1\] |= 0xC0;    // SIB byte unnecessary  
        // 如果未提供 offset（即是 disp），除非非 EBP 寄存器，否则是有问题的  
        if (op->offset == 0 && op->anyoffset == 0 && op->base != REG\_EBP)  
          ;            // \[EBP\] always requires offset  
        else if ((constsize & 1) != 0 && ((op->offset >= -128 && op->offset < 128)  
          || op->anyoffset != 0) )  
        {  
          // 0x40：1000000b，表示 disp 为 8 位数  
          tcode\[i + 1\] |= 0x40;  // Disp8  
          dispsize = 1;  
        }  
        else  
        {  
          // 否则，说明是 32 位 disp 数  
          tcode\[i + 1\] |= 0x80;  // Disp32  
          dispsize = 4;  
        };  
        // 明确指定基址寄存器  
        if (op->base < 8)  
        {  
          // 如果指明操作数的段前缀与该基址的默认段不一致，则使用操作数指明的段作为前缀  
          if (op->segment != SEG\_UNDEF && op->segment != addr32\[op->base\].defseg)  
            segment = op->segment;  
          // 设置操作数的基址  
          tcode\[i + 1\] |= (char)op->base;    // Note that case \[ESP\] has base<0.  
          tmask\[i + 1\] |= 0x07;  
        }  
        // 否则，直接使用操作数指明的段寄存器  
        else  
          segment = op->segment;  
      }  
      // 地址包含 index 寄存器部分，则需要 SIB 寻址  
      else    // SIB byte necessary  
      {  
        // 表明指令包含 SIB 字节  
        hassib = 1;  
        // 如果基址使用 EBP 寄存器，那么需要提供偏移量，所以，如果操作数未提供偏移量  
        // 而且又提供了 index 寄存器，并且其 scale 为 1，为了优先，可以将 index 寄存器作  
        // 为基址，而 EBP 作为 index 寄存器来使用  
        if (op->base == REG\_EBP &&      // EBP as base requires offset, optimize  
          op->index >= 0 && op->scale == 1 && op->offset == 0 && op->anyoffset == 0)  
        {  
          op->base = op->index;  
          op->index = REG\_EBP;  
        };  
        // ESP 寄存器不能作为 index 使用  
        if (op->index == REG\_ESP &&      // ESP cannot be an index, reorder  
          op->scale <= 1)   
        {  
          op->index = op->base;  
          op->base = REG\_ESP;  
          op->scale = 1;  
        };  
        // 如果操作数不存在基址，而 index 寄存器存在，并且 scale 为 2，并且其偏移量为 8 位，  
        // 例如\[index \* 2 + XX\]，可以优化为：\[base + index +XX\]  
        if (op->base < 0 &&          // No base means 4-byte offset, optimize  
          op->index >= 0 && op->scale == 2 &&  
          op->offset >= -128 && op->offset < 128 && op->anyoffset == 0)  
        {  
          op->base = op->index;  
          op->scale = 1;  
        };  
        // 至此，如果 index 还是 ESP，则出错  
        if (op->index == REG\_ESP)    // Reordering was unsuccessfull  
        {  
          strcpy(errtext, "Invalid indexing mode");  
          goto error;  
        };  
        // 确实未指定基址  
        if (op->base < 0)  
        {  
          tcode\[i + 1\] |= 0x04;  
          dispsize = 4;  
        }  
        // 地址未包含 displacement 值  
        else if (op->offset == 0 && op->anyoffset == 0 && op->base != REG\_EBP)  
          tcode\[i + 1\] |= 0x04;    // No displacement  
        // 包含 8 位的 displacement 值  
        else if ((constsize & 1) != 0 && ((op->offset >= -128 && op->offset < 128)  
          || op->anyoffset != 0) )  
        {  
          tcode\[i + 1\] |= 0x44;    // Disp8  
          dispsize = 1;  
        }  
        // 包含 32 位的 displacement 值  
        else  
        {  
          tcode\[i + 1\] |= 0x84;    // Disp32  
          dispsize = 4;  
        }  
        // 至此，ModRM 字节 OK 了，现在处理 SIB 字节  
        tmask\[i + 1\] |= 0xC7;      // ModRM completed, proceed with SIB  
  
        // 处理完指令的 ModRM，则继续处理 SIB 字节  
        if (op->scale == 2)  
          tcode\[i + 2\] |= 0x40;  
        else if (op->scale == 4)  
          tcode\[i + 2\] |= 0x80;  
        else if (op->scale == 8)  
          tcode\[i + 2\] |= 0xC0;  
        tmask\[i + 2\] |= 0xC0;  
  
        // 明确指定 index 寄存器  
        if (op->index < 8)   
        {  
          // 未指定 index  
          if (op->index < 0)  
            op->index = 0x04;  
          // 将 index 寄存器编码进 SIB 字节中  
          tcode\[i + 2\] |= (char)(op->index << 3);  
          tmask\[i + 2\] |= 0x38;  
        };  
        // 明确指定 base 寄存器  
        if (op->base < 8)   
        {  
          if (op->base < 0)  
            op->base = 0x05;  
          if (op->segment != SEG\_UNDEF && op->segment != addr32\[op->base\].defseg)  
            segment = op->segment;  
          // 将 base 寄存器编码进 SIB 字节中  
          tcode\[i + 2\] |= (char)op->base;  
          tmask\[i + 2\] |= 0x07;  
        }  
        else   
          segment = op->segment;   
      };  
      break;  
    case IMM:        // Immediate data (8 or 16/32)  
    case IMU:        // Immediate unsigned data (8 or 16/32)  
    case VXD:        // VxD service (32-bit only)  
      if (datasize == 0 && pd->arg2 == NNN && (pd->bits == SS || pd->bits == WS))  
        datasize = 4;  
      if (datasize == 0)  
      {  
        strcpy(errtext, "Please specify operand size");  
        goto error;  
      };  
      immediate = op->offset;  
      anyimm = op->anyoffset;  
      if (pd->bits == SS || pd->bits == WS)   
      {  
        if (datasize > 1 && (constsize & 2) != 0 && ((immediate >= -128 && immediate < 128)  
          || op->anyoffset != 0))   
        {  
          immsize = 1;  
          tcode\[i\] |= 0x02;  
        }  
        else  
          immsize = datasize;  
        tmask\[i\] |= 0x02;  
      }  
      else  
        immsize = datasize;  
      break;  
    case IMX:          // Immediate sign-extendable byte  
    case IMS:          // Immediate byte (for shifts)  
    case IM1:          // Immediate byte  
      if (immsize == 2)      // To accomodate ENTER instruction  
        immediate = (immediate & 0xFFFF) | (op->offset << 16);  
      else  
        immediate = op->offset;  
      anyimm |= op->anyoffset;  
      immsize++;  
      break;  
    case IM2:          // Immediate word (ENTER/RET)  
      immediate = op->offset;  
      anyimm = op->anyoffset;  
      immsize = 2;   
      break;  
    case IMA:          // Immediate absolute near data address  
      if (op->segment != SEG\_UNDEF && op->segment != SEG\_DS)  
        segment = op->segment;  
      displacement = op->offset;  
      anydisp = op->anyoffset;  
      dispsize = 4;  
      break;  
    case JOB:          // Immediate byte offset (for jumps)  
      jmpoffset = op->offset;  
      anyjmp = op->anyoffset;  
      jmpsize = 1;   
      break;  
    case JOW:          // Immediate full offset (for jumps)  
      jmpoffset = op->offset;  
      anyjmp = op->anyoffset;  
      jmpsize = 4;  
      break;  
    case JMF:          // Immediate absolute far jump/call addr  
      displacement = op->offset;  
      anydisp = op->anyoffset;  
      dispsize = 4;  
      immediate = op->segment;  
      anyimm = op->anyoffset;   
      immsize = 2;  
      break;  
    case SGM:          // Segment register in ModRM byte  
      hasrm = 1;  
      if (op->index < 6)  
      {  
        tcode\[i + 1\] |= (char)(op->index << 3);   
        tmask\[i + 1\] |= 0x38;  
      };  
      break;  
    case SCM:          // Segment register in command byte  
      if (op->index == SEG\_FS || op->index == SEG\_GS)   
      {  
        tcode\[0\] = 0x0F;  
        tmask\[0\] = 0xFF;  
        i = 1;  
        if (strcmp(name, "PUSH") == 0)  
          tcode\[i\] = (char)((op->index << 3) | 0x80);  
        else  
          tcode\[i\] = (char)((op->index << 3) | 0x81);  
        tmask\[i\] = 0xFF;  
      }  
      else if (op->index < 6)  
      {  
        if (op->index == SEG\_CS && strcmp(name, "POP") == 0)  
        {  
          strcpy(errtext, "Unable to POP CS");  
          goto error;  
        };  
        tcode\[i\] = (char)((tcode\[i\] & 0xC7) | (op->index << 3));  
      }  
      else  
      {  
        tcode\[i\] &= 0xC7;  
        tmask\[i\] &= 0xC7;  
      };  
      break;  
    case PRN:          // Near return address (pseudooperand)  
    case PRF:          // Far return address (pseudooperand)  
    case PAC:          // Accumulator (AL/AX/EAX, pseudooperand)  
    case PAH:          // AH (in LAHF/SAHF, pseudooperand)  
    case PFL:          // Lower byte of flags (pseudooperand)  
    case PS0:          // Top of FPU stack (pseudooperand)  
    case PS1:          // ST(1) (pseudooperand)  
    case PCX:          // CX/ECX (pseudooperand)  
    case PDI:          // EDI (pseudooperand in MMX extentions)  
      break;          // Simply skip preudooperands  
    default:          // Undefined type of operand  
      strcpy(errtext, "Internal Assembler error");  
      goto error;  
    };  
  };  
  
  // 至此，说明指令中对操作数的处理已 OK  
  
  // Gather parts of command together in the complete command.  
  j = 0;  
  
  // 如果指令包含 LOCK 前缀  
  if (lock != 0)      // Lock prefix specified  
  {  
    // 将 LOCK 前缀（即 0xF0）编入指令中  
    model->code\[j\] = 0xF0;  
    model->mask\[j\] = 0xFF;   
    j++;  
  };  
  // 如果指令需使用数组长度前缀  
  if (datasize == 2 && pd->bits != FF)  // Data size prefix necessary  
  {  
    // 将其编入指令中  
    model->code\[j\] = 0x66;  
    model->mask\[j\] = 0xFF;   
    j++;  
  };  
  // 同上  
  if (addrsize == 2)    // Address size prefix necessary  
  {  
    model->code\[j\] = 0x67;  
    model->mask\[j\] = 0xFF;  
    j++;  
  };  
  // 指定段前缀，将段前缀编码入指令中  
  if (segment != SEG\_UNDEF)  // Segment prefix necessary  
  {  
    if (segment == SEG\_ES)  
      model->code\[j\] = 0x26;  
    else if (segment == SEG\_CS)   
      model->code\[j\] = 0x2E;  
    else if (segment == SEG\_SS)  
      model->code\[j\] = 0x36;  
    else if (segment == SEG\_DS)  
      model->code\[j\] = 0x3E;  
    else if (segment == SEG\_FS)  
      model->code\[j\] = 0x64;  
    else if (segment == SEG\_GS)  
      model->code\[j\] = 0x65;  
    // 无效的段前缀  
    else  
    {   
      strcpy(errtext, "Internal Assembler error");  
      goto error;  
    };  
    model->mask\[j\] = 0xFF;   
    j++;  
  };  
  // 如果 displacement 存在，则同样编码入指令中  
  if (dispsize > 0)  
  {  
    memcpy(tcode + i + 1 + hasrm + hassib, &displacement, dispsize);  
    if (anydisp == 0)   
      memset(tmask + i + 1 + hasrm + hassib, 0xFF, dispsize);  
  };  
  
  
  
  
  
  
  
  
  // 至此，所有的信息足够生成机器指令了  
  
  
  
  // 指令拥有直接数  
  if (immsize > 0)  
  {  
    if (immsize == 1)  
      l = 0xFFFFFF00L;  
    else if (immsize == 2)  
      l = 0xFFFF0000L;  
    else   
      l = 0L;  
    //   
    if ((immediate & l) != 0 && (immediate & l) != l)  
    {  
      strcpy(errtext, "Constant does not fit into operand");  
      goto error;  
    };  
    memcpy(tcode + i + 1 + hasrm + hassib + dispsize, &immediate, immsize);  
    if (anyimm == 0)  
      memset(tmask + i + 1 + hasrm + hassib + dispsize, 0xFF, immsize);  
  };  
  i = i + 1 + hasrm + hassib + dispsize + immsize;  
  jmpoffset = jmpoffset - (i + j + jmpsize);  
  model->jmpsize = jmpsize;  
  model->jmpoffset = jmpoffset;  
  model->jmppos = i + j;  
  if (jmpsize != 0)  
  {  
    if (ip != 0)  
    {  
      jmpoffset = jmpoffset - ip;  
      // 判断跳转范围（有长短两种）  
      if (jmpsize == 1 && anyjmp == 0 && (jmpoffset < -128 || jmpoffset >= 128))  
      {  
        if (longjump == 0 && (jmpmode & 0x03) == 0)  
        {  
          longjump = 1;  
          goto retrylongjump;  
        };  
        sprintf(errtext, "Relative jump out of range, use %s LONG form", name);  
        goto error;  
      };  
      memcpy(tcode + i, &jmpoffset, jmpsize);  
    };  
    if (anyjmp == 0)  
      memset(tmask + i, 0xFF, jmpsize);  
    i += jmpsize;  
  };  
  // 将解码后的机器码复制到 model 中  
  memcpy(model->code + j, tcode, i);  
  memcpy(model->mask + j, tmask, i);  
  i += j;  
  // 指令长度  
  model->length = i;  
  
  return i;        // Positive value: length of code  
error:  
  model->length = 0;  
  return (int)(cmd - asmcmd);    // Negative value: position of error  
};  
  
//#pragma option -O.                     // Restore old optimization options  
  
  
  
// ------------------------------------------------- asmserv.cpp -----------------------------------  
  
// assemble.cpp 文件中使用到的结构信息  
  
// 文件太长，只摘取一部分  
// Free Disassembler and Assembler -- Command data and service routines  
//  
// Copyright (C) 2001 Oleh Yuschuk  
//  
//  This program is free software; you can redistribute it and/or modify  
//  it under the terms of the GNU General Public License as published by  
//  the Free Software Foundation; either version 2 of the License, or  
//  (at your option) any later version.  
//  
//  This program is distributed in the hope that it will be useful,  
//  but WITHOUT ANY WARRANTY; without even the implied warranty of  
//  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the  
//  GNU General Public License for more details.  
//  
//  You should have received a copy of the GNU General Public License  
//  along with this program; if not, write to the Free Software  
//  Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA  
//  
// 05.03.2002: Corrected error, FSTSW AX assembled with data size prefix  
  
#define STRICT  
  
#include "stdafx.h"  
  
#include <windows.h>  
#include <stdio.h>  
#include <string.h>  
#include <ctype.h>  
//#include <dir.h>  
#include <math.h>  
#include <float.h>  
#pragma hdrstop  
  
#pragma   warning(disable:4996)//关闭旧函数声明警告  
  
#include "disasm.h"  
  
/\*\*  
 \* 存储寄存器名称  
 \*  
 \* 类型：低 8 位寄存器名  
 \*     高 8 位寄存器名  
 \*     32 位寄存器名  
 \*/  
const char \* regname\[3\]\[9\] = {  
  { "AL", "CL", "DL", "BL", "AH", "CH", "DH", "BH", "R8"  },  
  { "AX", "CX", "DX", "BX", "SP", "BP", "SI", "DI", "R16" },  
  { "EAX","ECX","EDX","EBX","ESP","EBP","ESI","EDI","R32" } };  
  
 /\*\*  
  \* 存储段寄存器名称  
  \*/  
const char \* segname\[8\] = {  
  "ES","CS","SS","DS","FS","GS","SEG?","SEG?" };  
  
// 数据长度修饰符   
const char \* sizename\[11\] = {  
  "(0-BYTE)", "BYTE", "WORD", "(3-BYTE)",  
  "DWORD", "(5-BYTE)", "FWORD", "(7-BYTE)",  
  "QWORD", "(9-BYTE)", "TBYTE" };  
  
const t\_addrdec addr16\[8\] = {  
  { SEG\_DS,"BX+SI" }, { SEG\_DS,"BX+DI" },  
  { SEG\_SS,"BP+SI" }, { SEG\_SS,"BP+DI" },  
  { SEG\_DS,"SI" },    { SEG\_DS,"DI" },  
  { SEG\_SS,"BP" },    { SEG\_DS,"BX" } };  
  
/\*\*  
 \* 作为地址的两部分：基址与段选择子  
 \*    一般当使用某个寄存器作为基址寄存器时，则会  
 \*    有一个默认的段寄存器作为其选择子用，此数组  
 \*    存储的就是某基址寄存器与相应默认段寄存器的  
 \*    相对信息  
 \*/  
const t\_addrdec addr32\[8\] = {  
  { SEG\_DS,"EAX" }, { SEG\_DS,"ECX" },  
  { SEG\_DS,"EDX" }, { SEG\_DS,"EBX" },  
  { SEG\_SS,"" },    { SEG\_SS,"EBP" },  
  { SEG\_DS,"ESI" }, { SEG\_DS,"EDI" } };  
  
  
// FPU 相关寄存器说明:  
//    8 个 80 位浮点寄存器（数据），16 位状态寄存器，16 位控制寄存器，16 为标识寄存器。   
//    使用 FPU 指令对这些寄存器进行操作，这些寄存器构成一个循环栈，st7 栈底，st0 栈顶，   
//    当一个值被压入时，被存入 st0，原来 st0 中的值被存入 st7   
const char \* fpuname\[9\] = {  
  "ST0","ST1","ST2","ST3","ST4","ST5","ST6","ST7","FPU" };  
  
// MMX 寄存器  
const char \* mmxname\[9\] = {  
  "MM0","MM1","MM2","MM3","MM4","MM5","MM6","MM7","MMX" };  
  
// 探制寄存器  
const char \* crname\[9\] = {  
  "CR0","CR1","CR2","CR3","CR4","CR5","CR6","CR7","CRX" };  
  
const char \*drname\[9\] = {  
  "DR0","DR1","DR2","DR3","DR4","DR5","DR6","DR7","DRX" };  
  
// List of available processor commands with decoding, types of parameters and  
// other useful information. Last element has field mask=0. If mnemonic 助记符 begins  
// with ampersand ('&'), its mnemonic decodes differently depending on operand  
// size (16 or 32 bits). If mnemonic begins with dollar ('$'), this mnemonic  
// depends on address size. Semicolon (':') separates 16-bit form from 32-bit,  
// asterisk ('\*') will be substituted by either W (16), D (32) or none (16/32)  
// character. If command is of type C\_MMX or C\_NOW, or if type contains C\_EXPL  
// (=0x01), Disassembler must specify explicit size of memory operand.  
  
/\*\*  
 \* 存储处理器有效的（机器）指令信息，包括操作数和其他有用信息  
 \*/  
const t\_cmddata cmddata\[\] = {  
  
  // mask，code，len，bits，arg1，arg2，arg3，type，name  
    
  // 指令机器码的获取：code & mask  
  { 0x0000FF, 0x000090, 1,00,  NNN,NNN,NNN, C\_CMD+0,        "NOP" },  
  { 0x0000FE, 0x00008A, 1,WW,  REG,MRG,NNN, C\_CMD+0,        "MOV" },  
  { 0x0000F8, 0x000050, 1,00,  RCM,NNN,NNN, C\_PSH+0,        "PUSH" },  
  { 0x0000FE, 0x000088, 1,WW,  MRG,REG,NNN, C\_CMD+0,        "MOV" },  
  { 0x0000FF, 0x0000E8, 1,00,  JOW,NNN,NNN, C\_CAL+0,        "CALL" },  
  { 0x0000FD, 0x000068, 1,SS,  IMM,NNN,NNN, C\_PSH+0,        "PUSH" },  
    { 0x0000FF, 0x000060, 1,00,  NNN,NNN,NNN, C\_CMD+C\_RARE+0, "&PUSHA\*" },  
  { 0x0038FF, 0x00008F, 1,00,  MRG,NNN,NNN, C\_POP+1,        "POP" },  
  { 0x0000FF, 0x000061, 1,00,  NNN,NNN,NNN, C\_CMD+C\_RARE+0, "&POPA\*" },  
};  
  
// ------------------------------------- disasm.cpp ------------------------------------------  
  
// 用于反编译机器码  
  
// 此文件分析不够详细，但如能看懂 assemble.cpp ，则此文件相对简单了很多  
// Free Disassembler and Assembler -- Disassembler  
//  
// Copyright (C) 2001 Oleh Yuschuk  
//  
//  This program is free software; you can redistribute it and/or modify  
//  it under the terms of the GNU General Public License as published by  
//  the Free Software Foundation; either version 2 of the License, or  
//  (at your option) any later version.  
//  
//  This program is distributed in the hope that it will be useful,  
//  but WITHOUT ANY WARRANTY; without even the implied warranty of  
//  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the  
//  GNU General Public License for more details.  
//  
//  You should have received a copy of the GNU General Public License  
//  along with this program; if not, write to the Free Software  
//  Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA  
  
  
#define STRICT  
  
#include "stdafx.h"  
#include <windows.h>  
#include <stdio.h>  
#include <string.h>  
#include <ctype.h>  
//#include <dir.h>  
#include <math.h>  
#include <float.h>  
#pragma hdrstop  
  
#pragma   warning(disable:4996)//关闭旧函数声明警告  
  
#include "disasm.h"  
  
//bughoho. 文件扩展名改为 cpp.把外部扩展变量在 cpp 中添加声明  
int       ideal;                // Force IDEAL decoding mode  
int       lowercase;            // Force lowercase display  
int       tabarguments;         // Tab between mnemonic and arguments  
int       extraspace;           // Extra space between arguments  
int       putdefseg;            // Display default segments in listing  
int       showmemsize;          // Always show memory size  
int       shownear;             // Show NEAR modifiers  
int       shortstringcmds;      // Use short form of string commands  
  
// 某些指令助记符中含有'W'或'D'以指明其操作数长度，这种指令的机器码一般依赖于操作数的长度  
// 而当这些指令没有指明'W'或'D'时，则根据 sizesens 决定其操作数长度：  
//    0：操作数为 16 位（字）  
//    1：操作数为 32 位（双字）  
int       sizesens;             // How to decode size-sensitive mnemonics  
int       symbolic;             // Show symbolic addresses in disasm  
int       farcalls;             // Accept far calls, returns & addresses  
int       decodevxd;            // Decode VxD calls (Win95/98)  
int       privileged;           // Accept privileged commands  
int       iocommand;            // Accept I/O commands  
int       badshift;             // Accept shift out of range 1..31  
int       extraprefix;          // Accept superfluous prefixes  
int       lockedbus;            // Accept LOCK prefixes  
int       stackalign;           // Accept unaligned stack operations  
int       iswindowsnt;          // When checking for dangers, assume NT  
////////////////////////////////////////////////////////////////////////////////  
//////////////////////////// DISASSEMBLER FUNCTIONS ////////////////////////////  
  
// Work variables of disassembler  
static ulong     datasize;             // Size of data (1,2,4 bytes)  
  
// 地址大小  
static ulong     addrsize;             // Size of address (2 or 4 bytes)  
static int       segprefix;            // Segment override prefix or SEG\_UNDEF  
static int       hasrm;                // Command has ModR/M byte  
// 指令是否包含 SIB 字节  
static int       hassib;               // Command has SIB byte  
// displacement 的长度，1 表示 1 字节，2 表示 2 字节  
static int       dispsize;             // Size of displacement (if any)  
static int       immsize;              // Size of immediate data (if any)  
static int       softerror;            // Noncritical disassembler error  
  
// 代码已完成 dump 的长度  
static int       ndump;                // Current length of command dump  
  
// 登记当前已反汇编的指令长度  
static int       nresult;              // Current length of disassembly  
static int       addcomment;           // Comment value of operand  
  
// Copy of input parameters of function Disasm()  
static char      \*cmd;                 // Pointer to binary data  
static char      \*pfixup;              // Pointer to possible fixups or NULL  
static ulong     size;                 // Remaining size of the command buffer  
static t\_disasm  \*da;                  // Pointer to disassembly results  
static int       mode;                 // Disassembly mode (DISASM\_xxx)  
  
// Disassemble name of 1, 2 or 4-byte general-purpose integer register and, if  
// requested and available, dump its contents. Parameter type changes decoding  
// of contents for some operand types.  
// static void DecodeRG(int index,int datasize,int type)  
// bughoho void 改为 int 返回解码得到的寄存器索引  
//   
static int DecodeRG(int index, int datasize, int type)  
{  
  int sizeindex;  
  char name\[9\];  
  
  // 无需解码  
  if (mode < DISASM\_DATA)  
    return -1;    // No need to decode  
  
  sprintf\_s(da->vm\_name, "%s\_REG%02d", da->vm\_name, datasize \* 8);  
  
  // 操作数类型为寄存器  
  da->optype\[stoperand\] = Reg;  
  
  // 0x07: 00000111b  
  index &= 0x07;    //得到寄存器索引  
  if (datasize == 1)  
    sizeindex = 0;  
  else if (datasize == 2)  
    sizeindex = 1;  
  else if (datasize == 4)  
    sizeindex = 2;  
  else  
  {  
    da->error = DAE\_INTERN;  
    return -1;  
  }  
  // 反汇编  
  if (mode >= DISASM\_FILE)  
  {  
    // name 中将存储寄存器名称  
    strcpy(name, regname\[sizeindex\]\[index\]);  
    if (lowercase)  
      strlwr(name);  
    // 不是一个假操作数  
    if (type < PSEUDOOP)    // Not a pseudooperand  
      // 将操作数组合至 result 结果中  
      nresult += sprintf(da->result + nresult,"%s",name);  
  
    // bughoho  
    if (sizeindex == 0 && (index > 3 && index < 7))  
    {  
      // 高 8 字节寄存器：ah bh ch dh  
      da->highbit\[stoperand\] = true;  
    }  
  }  
  
  // 存储寄存器编号  
  da->reg\[stoperand\] = index;  
  return index;  
}  
  
// Disassemble name of 80-bit floating-point register and, if available, dump  
// its contents.  
// 对 index 编号的 FPU 寄存器 ST\[XX\]进行反汇编码  
static void DecodeST(int index, int pseudoop)  
{  
  int i;  
  char s\[32\];  
  // 不需反汇编  
  if (mode < DISASM\_FILE)  
    return;    // No need to decode  
  // 0x07: 00000111b  
  index &= 0x07;  
  // s = ST0 | ST1 | ST2 | ...  
  i = sprintf(s, "%s(%i)", (lowercase ? "st" : "ST"), index);  
  if (pseudoop == 0)  
  {  
      strcpy(da->result + nresult, s);  
      nresult += i;  
  }  
}  
  
// Disassemble name of 64-bit MMX register.  
// 对 index 编号的 MMX 寄存器进行反汇编码  
static void DecodeMX(int index)  
{  
  char \* pr;  
  if (mode < DISASM\_FILE)  
    return;    // No need to decode  
  index &= 0x07;  
  pr = da->result + nresult;  
  nresult += sprintf(pr, "%s%i", (lowercase ? "mm" : "MM"), index);  
}  
  
// Disassemble name of 64-bit 3DNow! register and, if available, dump its  
// contents.  
static void DecodeNR(int index)  
{  
  char \* pr;  
  if (mode < DISASM\_FILE)  
    return;    // No need to decode  
  index &= 0x07;  
  pr = da->result + nresult;  
  nresult += sprintf(pr, "%s%i", (lowercase ? "mm" : "MM"), index);  
}  
  
// Service function, adds valid memory adress in MASM or Ideal format to  
// disassembled string. Parameters: defseg - default segment for given  
// register combination, descr - fully decoded register part of address,  
// offset - constant part of address, dsize - data size in bytes. If global  
// flag 'symbolic' is set, function also tries to decode offset as name of  
// some label.  
// Memadr(seg, "", addr, dsize);  
// defseg: 默认段寄存器  
// descr:   
// offset：偏址  
// dsize: 数据长度  
static void Memadr(int defseg, const char \*descr,long offset,int dsize)  
{  
  int i, n, seg;  
  char \* pr;  
  char s\[TEXTLEN\];  
  if (mode < DISASM\_FILE || descr == NULL)  
    return;      // No need or possibility to decode  
  
  char segstr\[32\] = {0};  
  if (da->segment != SEG\_UNDEF)  
  {  
    if (da->segment == SEG\_FS)  
      strcpy(segstr, "FS");  
    else if (da->segment != SEG\_UNDEF && da->segment == SEG\_GS)  
      strcpy(segstr, "GS");  
  }  
  
  sprintf\_s(da->vm\_name, "%s\_%sMEM%02d", da->vm\_name, segstr, dsize \* 8);  
  da->optype\[stoperand\] = Mem;  
  
  // 指向反汇编结果缓存区  
  pr = da->result + nresult;  
  n = 0;  
  if (segprefix != SEG\_UNDEF)   
    seg = segprefix;   
  else  
    seg = defseg;  
  if (ideal != 0)   
    pr\[n++\] = '\[';  
  
  // In some cases Disassembler may omit 省略 size of memory operand. Namely 即是, flag  
  // showmemsize must be 0, type bit C\_EXPL must be 0 (this bit namely means  
  // that explicit operand size is necessary) and type of command must not be  
  // C\_MMX or C\_NOW (because bit C\_EXPL has in these cases different meaning).  
  // Otherwise, exact size must be supplied.  
  // 如果需要显示内存大小，或是 MMX 或 NOW 指令，或 C\_EXPL 设置，则指令不能省略操作数大小  
  if (showmemsize != 0 || (da->cmdtype & C\_TYPEMASK) == C\_MMX ||  
      (da->cmdtype & C\_TYPEMASK) == C\_NOW || (da->cmdtype & C\_EXPL) != 0)  
  {  
      if (dsize < sizeof(sizename) / sizeof(sizename\[0\]))  
      n += sprintf(pr + n, "%s %s", sizename\[dsize\], (ideal == 0 ? "PTR " : ""));  
    else  
      n += sprintf(pr + n, "(%i-BYTE) %s", dsize, (ideal == 0 ? "PTR " : ""));  
  }  
  
  // BYTE PTR ES:      \[??\]  
  
  // putdefseg: Display default segments in listing  
  // 如果需要显示默认段寄存器，或 seg 段本身不是默认段，则需要显示  
  if ((putdefseg != 0 || seg != defseg) && seg != SEG\_UNDEF)  
    n += sprintf(pr + n, "%s:", segname\[seg\]);  
  if (ideal == 0)  
    pr\[n++\] = '\[';  
  n += sprintf(pr + n, "%s", descr);  
  if (lowercase)  
    strlwr(pr);  
  // 未指定偏移地址  
  if (offset == 0L)   
  {  
    // 并且 descr 是空字符串，则偏移假设为 0  
    if (\*descr == '\\0')  
      pr\[n++\] = '0';  
  }  
  else  
  {  
    // symbolic: Show symbolic addresses in disasm  
    if (symbolic && mode >= DISASM\_CODE)  
      // 空操作，并且返回值为 0  
      i = Decodeaddress(offset, s, TEXTLEN - n - 24, NULL);  
    else  
      i = 0;  
    if (i > 0)    // Offset decoded in symbolic form  
    {  
      if (\*descr != '\\0')  
        pr\[n++\] = '+';  
        strcpy(pr + n, s);  
        n += i;   
    }  
    else if (offset < 0 && offset > -16384 && \*descr != '\\0')  
      n += sprintf(pr + n, "-%lX", -offset);  
    else  
    {  
      if (\*descr != '\\0')   
        pr\[n++\] = '+';  
      n += sprintf(pr + n, "%lX", offset);  
    };  
  };  
  pr\[n++\] = '\]';  
  pr\[n\] = '\\0';  
  nresult += n;  
};  
  
// Disassemble memory/register from the ModRM/SIB bytes and, if available, dump  
// address and contents of memory.  
static void DecodeMR(int type)  
{  
  int j, memonly, inmemory, seg;  
  int c, sib;  
  ulong dsize, regsize, addr;  
  char s\[TEXTLEN\];  
  if (size < 2)  
  {  
    da->error = DAE\_CROSS;  
    return;  
  }    // ModR/M byte outside the memory block  
  hasrm = 1;  
  // 默认数据大小  
  dsize = regsize = datasize;      // Default size of addressed reg/memory  
  memonly = 0;            // Register in ModM field is allowed  
  
  // Size and kind of addressed memory or register in ModM has no influence on 对...有影响  
  // the command size, and exact calculations are omitted if only command size  
  // is requested. If register is used, optype will be incorrect and we need  
  // to correct it later.  
  // 0xC7: 01111100b，取得 ModM 字节中 Mod 与 M 信息  
  c = cmd\[1\] & 0xC7;          // Leave only Mod and M fields  
  if (mode >= DISASM\_DATA)   
  {  
    // 0xC0：11000000b  
    // 寄存器寻址  
    if ((c & 0xC0) == 0xC0)      // Register operand  
      inmemory = 0;  
    else            // Memory operand  
      inmemory = 1;  
    // 判断是内存操作数还是寄存器操作数, 并确定长度  
    switch (type)  
    {  
    case MRG:        // Memory/register in ModRM byte  
      // 内存操作数  
      if (inmemory)  
      {  
        if (datasize == 1)   
          da->memtype = DEC\_BYTE;  
        else if (datasize == 2)   
          da->memtype = DEC\_WORD;  
        else  
          da->memtype = DEC\_DWORD;  
      }  
      break;  
    case MRJ:        // Memory/reg in ModRM as JUMP target  
      if (datasize != 2 && inmemory)  
        da->memtype = DEC\_DWORD;  
      if (mode >= DISASM\_FILE && shownear != 0)  
        nresult += sprintf(da->result + nresult, "%s ", (lowercase ? "near" : "NEAR"));  
      break;  
    case MR1:        // 1-byte memory/register in ModRM byte  
      dsize = regsize = 1;  
      if (inmemory)  
        da->memtype = DEC\_BYTE;  
      break;  
    case MR2:        // 2-byte memory/register in ModRM byte  
      dsize = regsize = 2;  
      if (inmemory)  
        da->memtype = DEC\_WORD;  
      break;  
    case MR4:        // 4-byte memory/register in ModRM byte  
    case RR4:        // 4-byte memory/register (register only)  
      dsize = regsize = 4;  
      if (inmemory)  
        da->memtype = DEC\_DWORD;  
      break;  
    case MR8:        // 8-byte memory/MMX register in ModRM  
    case RR8:        // 8-byte MMX register only in ModRM  
      dsize = 8;  
      if (inmemory)  
        da->memtype = DEC\_QWORD;  
      break;  
    case MRD:        // 8-byte memory/3DNow! register in ModRM  
    case RRD:        // 8-byte memory/3DNow! (register only)  
      dsize = 8;  
      if (inmemory)  
        da->memtype = DEC\_3DNOW;  
      break;  
    case MMA:        // Memory address in ModRM byte for LEA  
      memonly = 1;   
      break;  
    case MML:        // Memory in ModRM byte (for LES)  
      dsize = datasize + 2;  
      memonly = 1;  
      if (datasize == 4 && inmemory)  
        da->memtype = DEC\_FWORD;  
      da->warnings |= DAW\_SEGMENT;  
      break;  
    case MMS:        // Memory in ModRM byte (as SEG:OFFS)  
      dsize = datasize + 2;   
      memonly = 1;  
      if (datasize == 4 && inmemory)  
        da->memtype = DEC\_FWORD;  
      if (mode >= DISASM\_FILE)  
        nresult += sprintf(da->result + nresult, "%s ", (lowercase? "far" : "FAR"));  
      break;  
    case MM6:        // Memory in ModRM (6-byte descriptor)  
      dsize = 6;  
      memonly = 1;  
      if (inmemory)  
        da->memtype = DEC\_FWORD;  
      break;  
    case MMB:        // Two adjacent memory locations (BOUND)  
      dsize = (ideal ? datasize : datasize \* 2);  
      memonly = 1;  
      break;  
    case MD2:        // Memory in ModRM byte (16-bit integer)  
    case MB2:        // Memory in ModRM byte (16-bit binary)  
      dsize = 2;  
      memonly = 1;  
      if (inmemory)  
        da->memtype = DEC\_WORD;  
      break;  
    case MD4:        // Memory in ModRM byte (32-bit integer)  
      dsize = 4;  
      memonly = 1;  
      if (inmemory)  
        da->memtype = DEC\_DWORD;  
      break;  
    case MD8:        // Memory in ModRM byte (64-bit integer)  
      dsize = 8;  
      memonly = 1;  
      if (inmemory)  
        da->memtype = DEC\_QWORD;  
      break;  
    case MDA:        // Memory in ModRM byte (80-bit BCD)  
      dsize = 10;  
      memonly = 1;  
      if (inmemory)  
        da->memtype = DEC\_TBYTE;  
      break;  
    case MF4:        // Memory in ModRM byte (32-bit float)  
      dsize = 4;  
      memonly = 1;  
      if (inmemory)  
        da->memtype = DEC\_FLOAT4;  
      break;  
    case MF8:        // Memory in ModRM byte (64-bit float)  
      dsize = 8;  
      memonly = 1;  
      if (inmemory)  
        da->memtype = DEC\_FLOAT8;  
      break;  
    case MFA:        // Memory in ModRM byte (80-bit float)  
      dsize = 10;  
      memonly = 1;  
      if (inmemory)  
        da->memtype = DEC\_FLOAT10;  
      break;  
    case MFE:        // Memory in ModRM byte (FPU environment)  
      dsize = 28;  
      memonly = 1;  
      break;  
    case MFS:        // Memory in ModRM byte (FPU state)  
      dsize = 108;  
      memonly = 1;  
      break;  
    case MFX:        // Memory in ModRM byte (ext. FPU state)  
      dsize = 512;  
      memonly = 1;  
      break;  
    default:        // Operand is not in ModM!  
      da->error = DAE\_INTERN;  
      break;  
    };  
  };  
  
  addr = 0;  
  // There are many possibilities to decode ModM/SIB address. The first  
  // possibility is register in ModM - general-purpose, MMX or 3DNow!  
  
  // 对 ModM/SIB 地址反汇编有很多种可能。第一种可能就是通用寄存器,MMX 或 3DNow  
  
  // 为 0xC0 时则是寄存器  
  if ((c & 0xC0) == 0xC0)    // Decode register operand  
  {  
    if (type == MR8 || type == RR8)  
      DecodeMX(c);      // MMX register  
    else if (type == MRD || type == RRD)  
      DecodeNR(c);      // 3DNow! register  
    // 通用寄存器  
    else  
    {  
      DecodeRG(c, regsize, type);    // General-purpose register  
    }  
    if (memonly != 0)  
      softerror = DAE\_MEMORY;      // Register where only memory allowed  
    return;  
  }  
  // Next possibility: 16-bit addressing mode, very seldom in 32-bit flat model  
  // but still supported by processor. SIB byte is never used here.  
  // 下一个可能: 16 位地址模式.在 32 位平坦模式下很少见但是处理器仍然支持  
  // 这种情况下，SIB 字节是不需要用到的  
  if (addrsize == 2)  
  {  
    // 0x06: 110b  
    // Special case of immediate address  
    if (c == 0x06)  
    {  
      dispsize = 2;  
      if (size < 4)  
        da->error = DAE\_CROSS;    // Disp16 outside the memory block  
      else if (mode >= DISASM\_DATA)  
      {  
        // displement 值  
        da->adrconst = addr = \*(ushort \*)(cmd + 2);  
        // 该值为 0  
        if (addr == 0)  
          da->zeroconst = 1;  
        seg = SEG\_DS;  
        Memadr(seg, "", addr, dsize);  
      }   
    }  
    else  
    {  
      da->indexed = 1;  
      // 0x40: 1000000b  
      // 8-bit signed displacement  
      if ((c & 0xC0) == 0x40)  
      {  
        if (size < 3)  
          da->error = DAE\_CROSS;  
        else  
          addr = (signed char)cmd\[2\] & 0xFFFF;  
        dispsize = 1;  
      }  
      // 0x80: 10000000b  
      // 16-bit unsigned displacement  
      else if ((c & 0xC0) == 0x80)  
      {  
        if (size < 4)  
          da->error = DAE\_CROSS;  
        else  
          addr = \*(ushort \*)(cmd + 2);  
        dispsize = 2;   
      }  
      if (mode >= DISASM\_DATA && da->error == DAE\_NOERR)  
      {  
        // 保存地址的 displactement 部分  
        da->adrconst = addr;  
        if (addr == 0)  
          da->zeroconst = 1;  
        seg = addr16\[c & 0x07\].defseg;  
        Memadr(seg, addr16\[c & 0x07\].descr, addr, dsize);  
      }  
    }  
  }  
  // Next possibility: immediate 32-bit address.  
  // 下一个可能 : 32 位立即数的内存地址  
  // Special case of immediate address  
  else if (c == 0x05)  
  {  
    dispsize = 4;  
    if (size < 6)  
      da->error = DAE\_CROSS;      // Disp32 outside the memory block  
    else if (mode >= DISASM\_DATA)  
    {  
      // 设置地址  
      da->adrconst = addr = \*(ulong \*)(cmd + 2);  
      if (pfixup == NULL)  
        pfixup = cmd + 2;  
      da->fixupsize += 4;  
      // 地址为 0  
      if (addr == 0)  
        da->zeroconst = 1;  
      // 默认段  
      seg = SEG\_DS;  
      Memadr(seg, "", addr, dsize);  
    }  
  }  
  // Next possibility: 32-bit address with SIB byte.  
  // SIB addresation  
  // 下一个可能：32 位地址并且带有 SIB 字节  
  else if ((c & 0x07) == 0x04)  
  {  
    // 获取 SIB 字节  
    sib = cmd\[2\];  
    hassib = 1;  
    \*s = '\\0';  
    // \[SIB\]寻址  
    if (c == 0x04 && (sib & 0x07) == 0x05)  
    {  
      dispsize = 4;      // Immediate address without base  
      if (size < 7)  
        da->error = DAE\_CROSS;  // Disp32 outside the memory block  
      else  
      {  
        da->adrconst = addr = \*(ulong \*)(cmd + 3);  
        if (pfixup == NULL)   
          pfixup = cmd + 3;  
        da->fixupsize += 4;  
        if (addr == 0)  
          da->zeroconst = 1;  
  
        // Index register present  
        if ((sib & 0x38) != 0x20)   
        {  
          da->indexed = 1;  
          if (type == MRJ)  
            da->jmptable = addr;  
        };  
        // 默认段寄存器  
        seg = SEG\_DS;  
      }  
    }  
    // Base and, eventually 终于，最后, displacement  
    else  
    {  
      // 8-bit displacement  
      if ((c & 0xC0) == 0x40)  
      {  
        dispsize = 1;  
        if (size < 4)   
          da->error = DAE\_CROSS;  
        else  
        {  
          da->adrconst = addr = (signed char)cmd\[3\];  
          if (addr == 0)  
            da->zeroconst = 1;  
        }  
      }  
      // 32-bit displacement  
      else if ((c & 0xC0) == 0x80)  
      {  
        dispsize = 4;  
        if (size < 7)  
          da->error = DAE\_CROSS;  // Disp32 outside the memory block  
        else  
        {  
          da->adrconst = addr = \*(ulong \*)(cmd + 3);  
          if (pfixup == NULL)  
            pfixup = cmd + 3;  
          da->fixupsize += 4;  
          if (addr == 0)  
            da->zeroconst = 1;  
          // Most compilers use address of type \[index\*4+displacement\] to  
          // address jump table (switch). But, for completeness, I allow all  
          // cases which include index with scale 1 or 4, base or both.  
          if (type == MRJ)  
            da->jmptable = addr;  
        }  
      }  
      da->indexed = 1;  
      j = sib & 0x07;  
      if (mode >= DISASM\_FILE)  
      {  
        strcpy(s, regname\[2\]\[j\]);  
        seg = addr32\[j\].defseg;  
  
        da->addrreg1 = j;  
      }  
    }  
    // Scaled index present  
    if ((sib & 0x38) != 0x20)   
    {  
      if ((sib & 0xC0) == 0x40)   
        da->indexed = 2;  
      else if ((sib & 0xC0) == 0x80)  
        da->indexed = 4;  
      else if ((sib & 0xC0) == 0xC0)  
        da->indexed = 8;  
      else  
        da->indexed = 1;  
    };  
    if (mode >= DISASM\_FILE && da->error == DAE\_NOERR)   
    {  
      // Scaled index present  
      if ((sib & 0x38) != 0x20)  
      {  
        if (\*s != '\\0')  
          strcat(s, "+");  
        strcat(s, addr32\[(sib >> 3) & 0x07\].descr);  
        da->regsscale = 1;  
        da->addrreg2 = (sib >> 3) & 0x07;  // 第 2 个寄存器偏移  
        if ((sib & 0xC0) == 0x40)   
        {  
          da->jmptable = 0;        // Hardly a switch!  
          strcat(s, "\*2");  
          da->regsscale = 2;  
        }  
        else if ((sib & 0xC0) == 0x80)  
        {  
          strcat(s, "\*4");  
          da->regsscale = 4;  
        }  
        else if ((sib & 0xC0) == 0xC0)   
        {  
          da->jmptable = 0;        // Hardly a switch!  
          strcat(s, "\*8");  
          da->regsscale = 8;  
        }  
      }  
      Memadr(seg, s, addr, dsize);  
    };  
  }  
  // Last possibility: 32-bit address without SIB byte.  
  // 最后一个可能: 32 位地址  
  else    // No SIB  
  {  
    if ((c & 0xC0) == 0x40)  
    {  
      dispsize = 1;  
      if (size < 3)  
        da->error = DAE\_CROSS;    // Disp8 outside the memory block  
      else   
      {  
        da->adrconst = addr = (signed char)cmd\[2\];  
        if (addr == 0)  
          da->zeroconst = 1;  
      }  
    }  
    else if ((c & 0xC0) == 0x80)  
    {  
      dispsize = 4;  
      if (size < 6)  
        da->error = DAE\_CROSS;    // Disp32 outside the memory block  
      else   
      {  
        da->adrconst = addr = \*(ulong \*)(cmd + 2);  
        if (pfixup == NULL)   
          pfixup = cmd + 2;  
        da->fixupsize += 4;  
        if (addr == 0)  
          da->zeroconst = 1;  
        if (type == MRJ)  
          da->jmptable = addr;  
      }  
    }  
    da->indexed = 1;  
    if (mode >= DISASM\_FILE && da->error == DAE\_NOERR)   
    {  
      seg = addr32\[c & 0x07\].defseg;  
      da->addrreg1 = (c & 0x07);  
      Memadr(seg, addr32\[c & 0x07\].descr, addr, dsize);  
    }  
  }  
}  
  
// Disassemble implicit source of string operations and, if available, dump  
// address and contents.  
// static void DecodeSO(void)  
static int DecodeSO(void)  
{  
  if (mode < DISASM\_FILE)  
    return -1;    // No need to decode  
  if (datasize == 1)  
    da->memtype = DEC\_BYTE;  
  else if (datasize == 2)  
    da->memtype = DEC\_WORD;  
  else if (datasize == 4)  
    da->memtype = DEC\_DWORD;  
  da->indexed = 1;  
  Memadr(SEG\_DS, regname\[addrsize == 2 ? 1 : 2\]\[REG\_ESI\], 0L, datasize);  
  
  da->reg\[stoperand\] = REG\_ESI;  
  return REG\_ESI;  
}  
  
// Disassemble implicit destination of string operations and, if available,  
// dump address and contents. Destination always uses segment ES, and this  
// setting cannot be overridden.  
//static void DecodeDE(void)   
static int DecodeDE(void)   
{  
  int seg;  
  if (mode<DISASM\_FILE)   
    return -1;        // No need to decode  
  if (datasize==1)  
    da->memtype=DEC\_BYTE;  
  else if (datasize==2)  
    da->memtype=DEC\_WORD;  
  else if (datasize==4)  
    da->memtype=DEC\_DWORD;  
  da->indexed=1;  
  seg = segprefix;   
  segprefix=SEG\_ES;     // Fake Memadr by changing segment prefix  
  Memadr(SEG\_DS,regname\[addrsize==2?1:2\]\[REG\_EDI\],0L,datasize);  
  segprefix=seg;                       // Restore segment prefix  
  
  da->reg\[stoperand\] = REG\_EDI;  
  return REG\_EDI;  
};  
  
// Decode XLAT operand and, if available, dump address and contents.  
static void DecodeXL(void)  
{  
  if (mode<DISASM\_FILE)  
    return;        // No need to decode  
  da->memtype=DEC\_BYTE;  
  da->indexed=1;  
  Memadr(SEG\_DS,(addrsize==2?"BX+AL":"EBX+AL"),0L,1);  
};  
  
// Decode immediate operand of size constsize. If sxt is non-zero, byte operand  
// should be sign-extended to sxt bytes. If type of immediate constant assumes  
// this, small negative operands may be displayed as signed negative numbers.  
// Note that in most cases immediate operands are not shown in comment window.  
static void DecodeIM(int constsize,int sxt,int type)   
{  
  int i;  
  signed long data;  
  ulong l;  
  char name\[TEXTLEN\],comment\[TEXTLEN\] = {0};  
  immsize+=constsize;                    // Allows several immediate operands  
  if (mode<DISASM\_DATA)  
    return;  
  
  //sprintf\_s(da->vm\_name,"%s\_IMM%02d",da->vm\_name,constsize\*8);  
  sprintf\_s(da->vm\_name,"%s\_IMM32",da->vm\_name);//IMM 只有 32 位  
  da->optype\[stoperand\] = Imm;  
  
  l=1+hasrm+hassib+dispsize+(immsize-constsize);  
  data=0;  
  if (size<l+constsize)  
    da->error=DAE\_CROSS;  
  else if (constsize==1)   
  {  
    if (sxt==0)   
    data=(uchar)cmd\[l\];  
    else   
    data=(signed char)cmd\[l\];  
    if (type==IMS && ((data & 0xE0)!=0 || data==0))   
  {  
      da->warnings|=DAW\_SHIFT;  
      da->cmdtype|=C\_RARE;  
    }  
  }  
  else if (constsize==2)   
  {  
    if (sxt==0)   
    data=\*(ushort \*)(cmd+l);  
    else   
    data=\*(short \*)(cmd+l);  
  }  
  else   
  {  
    data=\*(long \*)(cmd+l);  
    if (pfixup==NULL)  
    pfixup=cmd+l;  
    da->fixupsize+=4;  
  }  
  if (sxt==2)  
    data&=0x0000FFFF;  
  if (data==0 && da->error==0)  
    da->zeroconst=1;  
  // Command ENTER, as an exception from Intel's rules, has two immediate  
  // constants. As the second constant is rarely used, I exclude it from  
  // search if the first constant is non-zero (which is usually the case).  
  if (da->immconst==0)  
    da->immconst=data;  
  if (mode>=DISASM\_FILE && da->error==DAE\_NOERR)   
  {  
    if (mode>=DISASM\_CODE && type!=IMU)  
      i=Decodeaddress(data,name,TEXTLEN-nresult-24,comment);  
    else   
  {  
      i=0; comment\[0\]='\\0';   
  }  
    if (i!=0 && symbolic!=0)  
  {  
      strcpy(da->result+nresult,name); nresult+=i;  
  }  
    else if (type==IMU || type==IMS || type==IM2 || data>=0 || data<NEGLIMIT)  
      nresult+=sprintf(da->result+nresult,"%lX",data);  
    else  
      nresult+=sprintf(da->result+nresult,"-%lX",-data);  
    if (addcomment && comment\[0\]!='\\0')  
    strcpy(da->comment,comment);  
  }  
}  
  
// Decode VxD service name (always 4-byte).  
static void DecodeVX(void)   
{  
  ulong l,data;  
  immsize+=4;                          // Allows several immediate operands  
  if (mode<DISASM\_DATA)   
    return;  
  l=1+hasrm+hassib+dispsize+(immsize-4);  
  if (size<l+4)   
  {  
    da->error=DAE\_CROSS;  
    return;  
  }  
  data=\*(long \*)(cmd+l);  
  if (data==0 && da->error==0)  
    da->zeroconst=1;  
  if (da->immconst==0)  
    da->immconst=data;  
  if (mode>=DISASM\_FILE && da->error==DAE\_NOERR)  
  {  
    if ((data & 0x00008000)!=0 && memicmp("VxDCall",da->result,7)==0)  
      memcpy(da->result,lowercase?"vxdjump":"VxDJump",7);  
    nresult+=sprintf(da->result+nresult,"%lX",data);  
  }  
}  
  
// Decode implicit constant 1 (used in shift commands). This operand is so  
// insignificant that it is never shown in comment window.  
static void DecodeC1(void)  
{  
  if (mode<DISASM\_DATA)  
    return;  
  da->immconst=1;  
  if (mode>=DISASM\_FILE)   
    nresult+=sprintf(da->result+nresult,"1");  
}  
  
// Decode immediate absolute data address. This operand is used in 8080-  
// compatible commands which allow to move data from memory to accumulator and  
// back. Note that bytes ModRM and SIB never appear in commands with IA operand.  
static void DecodeIA(void)   
{  
  ulong addr;  
  if (size<1+addrsize)   
  {  
    da->error=DAE\_CROSS;   
  return;   
  };  
  dispsize=addrsize;  
  if (mode<DISASM\_DATA)   
    return;  
  if (datasize==1)   
    da->memtype=DEC\_BYTE;  
  else if (datasize==2)  
    da->memtype=DEC\_WORD;  
  else if (datasize==4)  
    da->memtype=DEC\_DWORD;  
  if (addrsize==2)  
    addr=\*(ushort \*)(cmd+1);  
  else  
  {  
    addr=\*(ulong \*)(cmd+1);  
    if (pfixup==NULL)   
    pfixup=cmd+1;  
    da->fixupsize+=4;   
  }  
  da->adrconst=addr;  
  if (addr==0)  
    da->zeroconst=1;  
  if (mode>=DISASM\_FILE)  
  {  
    Memadr(SEG\_DS,"",addr,datasize);  
  }  
}  
  
// Decodes jump relative to nextip of size offsize.  
static void DecodeRJ(ulong offsize,ulong nextip)  
{  
  sprintf\_s(da->vm\_name,"%s\_IMM%02d",da->vm\_name,4\*8);//不管长跳短跳都认为是 32 位地址  
  da->optype\[stoperand\] = Imm;  
  int i;  
  ulong addr;  
  char s\[TEXTLEN\];  
  if (size<offsize+1)   
  {  
    da->error=DAE\_CROSS;   
    return;   
  }  
  dispsize=offsize;                    // Interpret offset as displacement  
  if (mode<DISASM\_DATA)  
    return;  
  if (offsize==1)//短跳转  
  {  
    addr=(signed char)cmd\[1\]+nextip;  
  }  
  else if (offsize==2)  
    addr=\*(signed short \*)(cmd+1)+nextip;  
  else  
    addr=\*(ulong \*)(cmd+1)+nextip;  
  if (datasize==2)  
    addr&=0xFFFF;  
  da->jmpconst=addr;  
  if (addr==0)   
    da->zeroconst=1;  
  if (mode>=DISASM\_FILE)  
  {  
    if (offsize==1)   
      nresult+=sprintf(da->result+nresult, "%s ",(lowercase==0?"SHORT":"short"));  
    if (mode>=DISASM\_CODE)  
      i=Decodeaddress(addr,s,TEXTLEN,da->comment);  
    else  
      i=0;  
    if (symbolic==0 || i==0)  
      nresult+=sprintf(da->result+nresult,"%08lX",addr);  
    else  
      nresult+=sprintf(da->result+nresult,"%.\*s",TEXTLEN-nresult-25,s);  
    if (symbolic==0 && i!=0 && da->comment\[0\]=='\\0')  
      strcpy(da->comment,s);  
  }  
}  
  
// Decode immediate absolute far jump address. In flat model, such addresses  
// are not used (mostly because selector is specified directly in the command),  
// so I neither decode as symbol nor comment it. To allow search for selector  
// by value, I interprete it as an immediate constant.  
static void DecodeJF(void)  
{  
  ulong addr,seg;  
  if (size<1+addrsize+2)  
  {  
    da->error=DAE\_CROSS;  
    return;  
  }  
  dispsize=addrsize; immsize=2;        // Non-trivial but allowed interpretation  
  if (mode<DISASM\_DATA)  
    return;  
  if (addrsize==2)   
  {  
    addr=\*(ushort \*)(cmd+1);  
    seg=\*(ushort \*)(cmd+3);  
  }  
  else  
  {  
    addr=\*(ulong \*)(cmd+1);  
    seg=\*(ushort \*)(cmd+5);  
  }  
  da->jmpconst=addr;  
  da->immconst=seg;  
  if (addr==0 || seg==0)  
    da->zeroconst=1;  
  if (mode>=DISASM\_FILE)  
  {  
    nresult+=sprintf(da->result+nresult,"%s %04X:%08X",(lowercase==0?"FAR":"far"),seg,addr);  
  }  
}  
  
// Decode segment register. In flat model, operands of this type are seldom.  
static void DecodeSG(int index)   
{  
  int i;  
  if (mode<DISASM\_DATA)  
    return;  
  
  index&=0x07;  
  if (index>=6)   
    softerror=DAE\_BADSEG;  // Undefined segment register  
  
  sprintf\_s(da->vm\_name,"%s\_%s",da->vm\_name,segname\[index\]);  
  da->optype\[stoperand\] = Seg;  
  
  if (mode>=DISASM\_FILE)   
  {  
    da->segreg = index;  
    i=sprintf(da->result+nresult,"%s",segname\[index\]);  
    if (lowercase)  
      strlwr(da->result+nresult);  
    nresult+=i;  
  }  
}  
  
// Decode control register addressed in R part of ModRM byte. Operands of  
// this type are extremely rare. Contents of control registers are accessible  
// only from privilege level 0, so I cannot dump them here.  
static void DecodeCR(int index)  
{  
  hasrm=1;  
  if (mode>=DISASM\_FILE)  
  {  
    index=(index>>3) & 0x07;  
    nresult+=sprintf(da->result+nresult,"%s",crname\[index\]);  
    if (lowercase)  
    strlwr(da->result+nresult);  
  }  
}  
  
// Decode debug register addressed in R part of ModRM byte. Operands of  
// this type are extremely rare. I can dump only those debug registers  
// available in CONTEXT structure.  
static void DecodeDR(int index)   
{  
  int i;  
  hasrm=1;  
  if (mode>=DISASM\_FILE)   
  {  
    index=(index>>3) & 0x07;  
    i=sprintf(da->result+nresult,"%s",drname\[index\]);  
    if (lowercase)  
    strlwr(da->result+nresult);  
    nresult+=i;  
  }  
}  
  
// Skips 3DNow! operands and extracts command suffix. Returns suffix or -1 if  
// suffix lies outside the memory block. This subroutine assumes that cmd still  
// points to the beginning of 3DNow! command (i.e. to the sequence of two bytes  
// 0F, 0F).  
/\*\*  
 \* 获取 3dNow 指令的后缀，对于什么是其后缀，可上网搜索  
 \*  
 \* 3dNow 指令格式：0Fh 0Fh modR/M \[sib\] \[displacement\] 3DNow!\_suffix  
 \*  
 \* 返回值：成功返回其后缀，否则返回-1  
 \*/  
static int Get3dnowsuffix(void)  
{  
  int c, sib;  
  ulong offset;  
  // 如果 3dNow 后缀超出内存块  
  if (size < 3)  
    return -1;      // Suffix outside the memory block  
  
  // 3dNow 指令的后缀偏移  
  offset = 3;  
  
  // ModRM 字节的组成部分为：mod-reg-r/m  
  
  // 0xC7: 11000111b  
  c = cmd\[2\] & 0xC7;    // Leave only Mod and M fields  
  
  // Register in ModM - general-purpose 多方面的，多用途的, MMX or 3DNow!  
  // 0xC0: 11000000b  
  // 高两位为 1，表示寄存器寻址  
  if ((c & 0xC0) == 0xC0)  
    ;  
  // 16-bit addressing mode, SIB byte is never used here.  
  // 否则，如果是 16 位地址模式，则 SIB 不会被使用  
  else if (addrsize == 2)  
  {  
      if (c == 0x06)      // Special case of immediate address  
      offset += 2;  
    // 0x40: 1000000b  
    else if ((c & 0xC0) == 0x40)  // 8-bit signed displacement  
      offset++;  
    // 0x80: 10000000b  
    else if ((c & 0xC0) == 0x80)  // 16-bit unsigned displacement  
      offset += 2;  
  }  
  // Immediate 32-bit address.  
  // 0x05: 101b  
  else if (c == 0x05)      // Special case of immediate address  
    offset += 4;  
  // 32-bit address with SIB byte.  
  // 如果指令使用 32 位地址模式并带有 SIB 字节  
  else if ((c & 0x07) == 0x04)  // SIB addresation  
  {  
    if (size < 4)  
      return -1;      // Suffix outside the memory block  
    // 获取 SIB 字节  
    sib = cmd\[3\];   
    offset++;  
    if (c == 0x04 && (sib & 0x07) == 0x05)  
      offset += 4;    // Immediate address without base  
    else if ((c & 0xC0) == 0x40)  // 8-bit displacement  
      offset += 1;  
    else if ((c & 0xC0) == 0x80)  // 32-bit dislacement  
      offset += 4;  
  }  
  // 32-bit address without SIB byte  
  else if ((c & 0xC0) == 0x40)  
    offset += 1;  
  else if ((c & 0xC0) == 0x80)  
    offset += 4;  
  if (offset >= size)   
    return -1;    // Suffix outside the memory block  
  return cmd\[offset\];  
}  
  
// Function checks whether 80x86 flags meet condition set in the command.  
// Returns 1 if condition is met, 0 if not and -1 in case of error (which is  
// not possible).  
int Checkcondition(int code,ulong flags)  
{  
  ulong cond,temp;  
  switch (code & 0x0E)   
  {  
    case 0:                            // If overflow  
      cond=flags & 0x0800;  
    break;  
    case 2:                            // If below  
      cond=flags & 0x0001;  
    break;  
    case 4:                            // If equal  
      cond=flags & 0x0040;  
    break;  
    case 6:                            // If below or equal  
      cond=flags & 0x0041;   
    break;  
    case 8:                            // If sign  
      cond=flags & 0x0080;   
    break;  
    case 10:                           // If parity  
      cond=flags & 0x0004;  
    break;  
    case 12:                           // If less  
      temp=flags & 0x0880;  
      cond=(temp==0x0800 || temp==0x0080);   
    break;  
    case 14:                           // If less or equal  
      temp=flags & 0x0880;  
      cond=(temp==0x0800 || temp==0x0080 || (flags & 0x0040)!=0);  
    break;  
    default:  
    return -1;                // Internal error, not possible!  
  }  
  if ((code & 0x01)==0)  
    return (cond!=0);  
  else  
    return (cond==0);               // Invert condition  
}  
  
  
// bughoho  
int stoperand = 0;  
  
// Disasm(base\_addr, MAXCMDSIZE, VirtualAddress, &da, DISASM\_CODE)  
  
ulong Disasm(char \* src, ulong srcsize, ulong srcip, t\_disasm \* disasm, int disasmmode)   
{  
  // isprefix: 表示指令是否有段前缀，加锁总线前缀等前缀  
  // is3dnow: 表示指令是否是 3dNow 指令  
  int i, j, isprefix, is3dnow, repeated, operand, mnemosize, arg = 0;  
  ulong u, code;  
  // 是否存在 LOCK 前缀  
  int lockprefix;        // Non-zero if lock prefix present  
  // 是否存在 REP 前缀  
  int repprefix;        // REPxxx prefix or 0  
  int cxsize;  
  // pname: 保存前缀名称，如 REPN、REPE  
  char name\[TEXTLEN\], \* pname;  
  const t\_cmddata \* pd, \* pdan;  
  
  // Prepare disassembler variables and initialize structure disasm.  
  // 初始化结构和变量  
  datasize = addrsize = 4;        // 32-bit code and data segments only!  
  // 指令的段前缀  
  segprefix = SEG\_UNDEF;  
  hasrm = hassib=0;  
  dispsize = immsize = 0;  
  // 加锁总线前缀  
  lockprefix = 0;  
  // REP 前缀  
  repprefix = 0;  
  ndump = 0;  
  nresult = 0;  
  
  // 指令  
  cmd = src;  
  size = srcsize;  
  pfixup = NULL;  
  softerror = 0;  
  is3dnow = 0;  
  da = disasm;  
  da->ip = srcip;  
  da->comment\[0\] = '\\0';  
  da->cmdtype = C\_BAD;  
  da->nprefix = 0;  
  da->memtype = DEC\_UNKNOWN;  
  da->indexed = 0;  
  da->jmpconst = 0;  
  da->jmptable = 0;  
  da->adrconst = 0;  
  da->immconst = 0;  
  da->zeroconst = 0;  
  da->fixupoffset = 0;  
  da->fixupsize = 0;  
  da->warnings = 0;  
  da->error = DAE\_NOERR;  
  mode = disasmmode;      // No need to use register contents  
  
  // 操作数类型  
  da->optype\[0\] = da->optype\[1\] = da->optype\[2\] = -1;  
  // 3 个寄存器，当操作数为寄存器时  
  da->reg\[0\] = da->reg\[1\] = da->reg\[2\] = -1;  
  // 段寄存器  
  da->segreg = -1;  
  
  da->addrreg1 = da->addrreg2 = -1;  
  da->regsscale = 1;  
  // 清二进制代码  
  memset(da->hexcode, 0, TEXTLEN);  
  da->codelen = 0;  
  da->highbit\[0\] = da->highbit\[1\] = da->highbit\[2\] = false;  
  
  // Correct 80x86 command may theoretically 在理论上 contain up to 多达 4 prefixes belonging  
  // to different prefix groups. This limits maximal possible size of the  
  // command to MAXCMDSIZE=16 bytes. In order to maintain this limit, if  
  // Disasm() detects second prefix from the same group, it flushes first  
  // prefix in the sequence 顺序 as a pseudocommand 伪指令.  
  u = 0;  
  repeated = 0;  
  
  // 统计指令 cmd 中所有的前缀数  
  while (size > 0)  
  {  
    // 假设存在一些前缀  
    isprefix = 1;      // Assume that there is some prefix  
    // 根据指令处理其前缀部分  
    switch (\* cmd)  
    {  
      // 段前缀  
    case 0x26:  
      if (segprefix == SEG\_UNDEF)  
        segprefix = SEG\_ES;  
      else  
        // 重复出现 0x26  
        repeated = 1;  
      break;  
    case 0x2E:  
      if (segprefix == SEG\_UNDEF)  
        segprefix = SEG\_CS;  
      else  
        repeated = 1;   
      break;  
    case 0x36:  
      if (segprefix == SEG\_UNDEF)  
        segprefix = SEG\_SS;  
      else  
        repeated = 1;  
      break;  
    case 0x3E:  
      if (segprefix == SEG\_UNDEF)   
        segprefix = SEG\_DS;  
      else  
        repeated = 1;  
      break;  
    case 0x64:  
      if (segprefix == SEG\_UNDEF)   
        segprefix = SEG\_FS;  
      else  
        repeated = 1;  
      break;  
    case 0x65:  
      if (segprefix == SEG\_UNDEF)  
        segprefix = SEG\_GS;  
      else  
        repeated = 1;  
      break;  
      // 66H 前缀用来更改 operands size，当然只能在指令所  
      // 支持的 effective operand-size 范围内进行调整  
    case 0x66:  
      if (datasize == 4)   
        datasize = 2;  
      else  
        // 前缀重复  
        repeated = 1;   
      break;  
      // 指令中可以 67H 进行 address size override  
    case 0x67:  
      if (addrsize == 4)   
        addrsize = 2;  
      else  
        repeated = 1;  
      break;  
      // 加锁总线前缀  
    case 0xF0:  
      if (lockprefix == 0)  
        lockprefix = 0xF0;  
      else  
        repeated = 1;  
      break;  
      // REP 前缀  
    case 0xF2:  
      if (repprefix == 0)   
        repprefix = 0xF2;  
      else  
        repeated = 1;  
      break;  
    case 0xF3:   
      if (repprefix == 0)   
        repprefix = 0xF3;  
      else  
        repeated = 1;  
      break;  
    default:   
      isprefix = 0;  
      break;  
    }  
  
    // 如果没有前缀，或前缀重复，则跳出循环  
    if (isprefix == 0 || repeated != 0)  
      break;        // No more prefixes or duplicated prefix  
    if (mode >= DISASM\_FILE)  
    {  
      ndump += sprintf(da->dump + ndump, "%02X:", \* cmd);  
      da->hexcode\[da->codelen++\] = \*cmd;  
    }  
    // 累计指令前缀数  
    da->nprefix++;  
    cmd++;  
    srcip++;  
    size--;  
    u++;  
  }  
  
  // bughoho  
  da->segment = segprefix;  // 得到段寄存器  
  // We do have repeated prefix. Flush first prefix from the sequence.  
  // 如果指令拥有重复的前缀  
  if (repeated)  
  {  
    // 如果是对代码进行反汇编 \[??\]  
    if (mode >= DISASM\_FILE)  
    {  
      da->dump\[3\] = '\\0';      // Leave only first dumped prefix  
      // 为何只保留一个前缀  
      da->nprefix = 1;  
  
      // 获取前缀的名称  
      switch (cmd\[-(long)u\])    // cmd++ 且 u++;这里是得到 cmd\[0\]  
      {  
      case 0x26: pname = (char \*)(segname\[SEG\_ES\]);   
        break;  
      case 0x2E: pname = (char \*)(segname\[SEG\_CS\]);  
        break;  
      case 0x36: pname = (char \*)(segname\[SEG\_SS\]);  
        break;  
      case 0x3E: pname = (char \*)(segname\[SEG\_DS\]);  
        break;  
      case 0x64: pname = (char \*)(segname\[SEG\_FS\]);  
        break;  
      case 0x65: pname = (char \*)(segname\[SEG\_GS\]);   
        break;  
      case 0x66: pname = "DATASIZE";  
        break;  
      case 0x67: pname = "ADDRSIZE";  
        break;  
      case 0xF0: pname = "LOCK";  
        break;  
      case 0xF2: pname = "REPNE";  
        break;  
      case 0xF3: pname = "REPE";  
        break;  
      default: pname = "?";  
        break;   
      }  
  
      // 累计已反汇编的指令长度  
      nresult += sprintf(da->result + nresult, "PREFIX %s:",pname);  
      // 是否强制要求以小写显示  
      if (lowercase)  
        strlwr(da->result);  
      // 不支持扩展的前缀（多余的前缀）  
      if (extraprefix == 0)   
        strcpy(da->comment, "Superfluous prefix");  
    }  
    // 警告：指令有过多的前缀  
    da->warnings |= DAW\_PREFIX;  
    // 指令存在加锁总线的前缀  
    if (lockprefix)  
      da->warnings |= DAW\_LOCK;  
  
    // 该指令罕见，少用  
    da->cmdtype = C\_RARE;  
    return 1;                          // Any prefix is 1 byte long  
  }  
  
  // If lock prefix available, display it and forget, because it has no  
  // influence on 对...有影响 decoding of rest of the command.  
  // 指令存在加锁总线的前缀  
  if (lockprefix != 0)  
  {  
    if (mode >= DISASM\_FILE)   
      nresult += sprintf(da->result + nresult, "LOCK ");  
    da->warnings |= DAW\_LOCK;  
  }  
  
  // Fetch (if available) first 3 bytes of the command, add repeat prefix and  
  // find command in the command table.  
  // 取得命令的前三个字节  
  code = 0;  
  // 如果指令剩余空间存在  
  if (size > 0)  
    \*(((char \*)&code) + 0) = cmd\[0\];  
  if (size > 1)  
    \*(((char \*)&code) + 1) = cmd\[1\];  
  if (size > 2)   
    \*(((char \*)&code) + 2) = cmd\[2\];  
  // 如果 REP 前缀存在  
  if (repprefix != 0)        // RER/REPE/REPNE is considered to be  
    // 则将其加入到 code 指令中  
    code = (code << 8) | repprefix;      // part of command.  
  if (decodevxd && (code & 0xFFFF) == 0x20CD)  
    pd = &vxdcmd;        // Decode VxD call (Win95/98)  
  else  
  {  
    // 遍历指令表  
    for (pd = cmddata; pd->mask != 0; pd++)  
    {  
      // 检测是否 pd 指令  
      if (((code ^ pd->code) & pd->mask) != 0)  
        continue;  
  
      // 如果是反汇编模式,并且 Use short form of string commands  
      if (mode >= DISASM\_FILE && shortstringcmds &&  
        // 如果寄存器是 ESI 或 EDI  
        (pd->arg1 == MSO || pd->arg1 == MDE || pd->arg2 == MSO || pd->arg2 == MDE))  
        continue;    // Search short form of string command  
      break;  
    }  
  }  
  
  if ( (pd->type & C\_TYPEMASK) == C\_NOW)  // 如果是 3DNow 指令  
  {  
    // 3DNow! commands require additional search.  
    is3dnow = 1;  
    // 获取 3DNow 指令的后缀  
    j = Get3dnowsuffix();  
  
    // 当存放指令的内存空间不足时，会返回-1  
    // 这表示指令与内存交叉，出错  
    if (j < 0)  
      da->error = DAE\_CROSS;  
    else  
    {  
      // 遍历指令表，获取后缀与欲反汇编指令相匹配的指令  
      for ( ; pd->mask != 0; pd++)  
      {  
        if (((code ^ pd->code) & pd->mask) != 0)  
          continue;  
        if (((uchar \*)&(pd->code))\[2\] == j)  
          break;  
      }  
    }  
  }  
  
  // bughoho  
  da->is3dnow = is3dnow;  // 是否是 3DNow!指令  
  
  // Command not found  
  if (pd->mask == 0)    // 命令未找到  
  {  
    // 指令错误  
    da->cmdtype = C\_BAD;  
    if (size < 2)  
      da->error = DAE\_CROSS;  
    else   
      da->error = DAE\_BADCMD;   
  }  
  // 指令 OK，是一注册指令，则反汇编之  
  else  
  {  
    // Command recognized, decode it  
    // 指令类型信息  
    da->cmdtype = pd->type;  
    // ECX 作为记数寄存器的默认长度，与数据长度等同  
    cxsize = datasize;    // Default size of ECX used as counter  
  
    // 如果段前缀是 FS 或 GS 或存在 LOCK 前缀  
    // 则表明此指令并不常用  
    if (segprefix == SEG\_FS || segprefix == SEG\_GS || lockprefix != 0)  
      da->cmdtype |= C\_RARE;  // These prefixes are rare  
    // 如果该指令是一个特权指令  
    if (pd->bits == PR)  
      da->warnings |= DAW\_PRIV;  // Privileged command (ring 0)  
    // IO 命令  
    else if (pd->bits == WP)  
      da->warnings |= DAW\_IO;    // I/O command  
  
    // Win32 programs usually try to keep stack dword-aligned, so INC ESP  
    // (44) and DEC ESP (4C) usually don't appear in real code. Also check for  
    // ADD ESP,imm and SUB ESP,imm (81,C4,imm32; 83,C4,imm8; 81,EC,imm32;  
    // 83,EC,imm8).  
    // WIN32 程序通常会保持栈的双字对齐, 所以 inc esp,dec esp 一般不会出现在  
    // 平常的代码中, 除此同样检测 add esp,imm 和 sub esp,imm.  
    if (cmd\[0\] == 0x44 || cmd\[0\] == 0x4C ||  
      (size >= 3 && (cmd\[0\] == 0x81 || cmd\[0\] == 0x83) &&  
      (cmd\[1\] == 0xC4 || cmd\[1\] == 0xEC) && (cmd\[2\] & 0x03) != 0))  
    {  
      da->warnings |= DAW\_STACK;  // 栈对齐警告  
      da->cmdtype |= C\_RARE;    // 不常出现的指令  
    }  
  
    // Warn also on MOV SEG,... (8E...). Win32 works in flat mode.  
    // 警告 mov seg,xxx 操作, 因为 Win32 工作在平坦模式  
    if (cmd\[0\] == 0x8E)  
      da->warnings |= DAW\_SEGMENT;  
  
    // If opcode is 2-byte, adjust command.  
    // 如果操作码是 2 字节的则调整命令  
    if (pd->len == 2)  
    {  
      if (size == 0)  
        da->error = DAE\_CROSS;  
      else   
      {  
        // 工作于反汇编模式  
        if (mode >= DISASM\_FILE)  
        {  
          // 将原二进制代码 dump 至 da->dump 中  
          ndump += sprintf(da->dump + ndump, "%02X", \*cmd);  
          // 存储二进制代码  
          da->hexcode\[da->codelen++\] = \*cmd;  
        }  
        cmd++;  
        srcip++;  
        size--;  
      }  
    }  
  
    if (size == 0)  
      da->error = DAE\_CROSS;  
  
    // Some commands either feature non-standard data size or have bit which  
    // allowes to select data size.  
    // 一些命令的数据长度不是标准的,其长度是可变的  
    if ((pd->bits & WW) != 0 && (\*cmd & WW) == 0)  
      datasize = 1;      // Bit W in command set to 0  
    else if ((pd->bits & W3) != 0 && (\*cmd & W3) == 0)  
      datasize = 1;      // Another position of bit W  
    else if ((pd->bits & FF) != 0)  
      datasize = 2;      // Forced word (2-byte) size  
  
    // Some commands either have mnemonics which depend on data size (8/16 bits  
    // or 32 bits, like CWD/CDQ), or have several different mnemonics (like  
    // JNZ/JNE). First case is marked by either '&' (mnemonic depends on  
    // operand size) or '$' (depends on address size). In the second case,  
    // there is no special marker and disassembler selects main mnemonic.  
    // 有些指令的助记符依赖于数据长度(例如 CWD/CDQ)，或拥有多个不同的助记符  
    // (例如 JNZ/JNE)。第一种情况是使用"&"或"$"标记。第二种情况没有特定标记  
    // 并被反汇编为主要的助记符  
    if (mode >= DISASM\_FILE)  
    {  
      // 指令的助记符依赖于操作数大小  
      if (pd->name\[0\] == '&')  
        mnemosize = datasize;  
      // 指令的助记符依赖于地址大小  
      else if (pd->name\[0\] == '$')  
        mnemosize = addrsize;  
      else  
        mnemosize = 0;  
      // 助记符大小不为 0，依赖数据或地址大小  
      if (mnemosize != 0)  
      {  
        // 遍历 pd 指令的符号名中每个字符  
        for (i = 0,j = 1; pd->name\[j\] != '\\0'; j++)   
        {  
          // 如果不是":"分割符  
          if (pd->name\[j\] == ':')  
          {  
            // Separator between 16/32 mnemonics  
            // 分割符用于分割 16 位/32 位的助记符  
            if (mnemosize == 4)   
              i = 0;  
            else  
              break;  
          }  
          // 指令中如出现"\*"，则用"W"或"D"代替，作为其长度标记  
          else if (pd->name\[j\] == '\*')   
          {  
            // Substitute 替换，代替 by 'W', 'D' or none  
            if (mnemosize == 4 && sizesens != 2)  
              name\[i++\] = 'D';  
            else if (mnemosize != 4 && sizesens != 0)  
              name\[i++\] = 'W';  
          }  
          else  
            name\[i++\] = pd->name\[j\];  
        }  
        name\[i\] = '\\0';   
      }  
      // 否则，说明指令不依赖数据或地址大小  
      else  
      {  
        strcpy(name, pd->name);  
        // 当指令中有","号分割时，说明该指令有多个  
        // 助记符，使用主要的一个就行  
        for (i = 0; name\[i\] != '\\0'; i++)  
        {  
          if (name\[i\] == ',')   
          {  
            // Use main mnemonic  
            name\[i\] = '\\0';  
            break;  
          }  
        }  
      }  
      // 将 name 反汇编后的指令复制至 da 中  
      // 存在 REP 前缀，并且...  
      if (repprefix != 0 && tabarguments)   
      {  
        for (i = 0; name\[i\] != '\\0' && name\[i\] != ' '; i++)  
          da->result\[nresult++\] = name\[i\];  
        if (name\[i\] == ' ')   
        {  
          da->result\[nresult++\] = ' ';  
          i++;   
        }  
        while(nresult < 8)   
          da->result\[nresult++\] = ' ';  
        for ( ; name\[i\] != '\\0'; i++)  
          da->result\[nresult++\] = name\[i\];  
      }  
      else  
        nresult += sprintf(da->result + nresult, "%s", name);  
  
      // 要求指令转小写  
      if (lowercase)   
        strlwr(da->result);  
      sprintf\_s(da->vm\_name, "V%s", name);  
    }  
  
    // Decode operands (explicit - encoded in command, implicit - present in  
    // mmemonic or assumed - used or modified by command). Assumed operands  
    // must stay after all explicit and implicit operands. Up to 3 operands  
    // are allowed.  
    // 处理操作数，最多可以有 3 个  
    for (operand = 0; operand < 3; operand++)  
    {  
      stoperand = operand;  
  
      // 如果已出错，则直接跳出，不需继续了  
      if (da->error)  
        break;      // Error - no sense to continue  
      // If command contains both source and destination, one usually must not  
      // decode destination to comment because it will be overwritten on the  
      // next step. Global addcomment takes care of this. Decoding routines,  
      // however, may ignore this flag.  
      if (operand == 0 && pd->arg2 != NNN && pd->arg2 < PSEUDOOP)  
        addcomment = 0;  
      else  
        addcomment = 1;  
      // Get type of next argument.  
      if (operand == 0)  
        arg = pd->arg1;  
      else if (operand == 1)  
        arg = pd->arg2;  
      else  
        arg = pd->arg3;  
  
      // 没有更多的操作数了  
      if (arg == NNN)   
        break;    // No more operands  
  
      // Arguments with arg>=PSEUDOOP are assumed operands and are not  
      // displayed in disassembled result, so they require no delimiter.  
      // 参数>=PSEUDOOP 是假的操作数，并且不显示在反汇编的结果中，所以  
      // 他们不需要分割符  
      if ((mode >= DISASM\_FILE) && arg < PSEUDOOP)  // 添加间隔  
      {  
        // 指令与第 1 个操作数之前有 ' ' 间隔  
        if (operand == 0)  
        {  
          da->result\[nresult++\] = ' ';  
          if (tabarguments)  
          {  
            while (nresult < 8)  
              da->result\[nresult++\] = ' ';  
          }  
        }  
        // 操作数之间则以 ',' 间隔  
        else  
        {  
          da->result\[nresult++\] = ',';  
          if (extraspace)  
            da->result\[nresult++\] = ' ';  
        }  
      }  
  
      // Decode, analyse and comment next operand of the command.  
      // 解码,分析和注释命令的下一个操作数  
      // 现在正式开始  
      switch (arg)  
      {  
        // 参数是整型寄存器  
      case REG:      // Integer register in Reg field  
        if (size < 2)  
          da->error = DAE\_CROSS;  
        else  
        {  
          DecodeRG(cmd\[1\] >> 3, datasize, REG);  
        }  
        // 表示指令含有 ModRm 字节  
        hasrm = 1;  
        break;  
        // 参数是整型寄存器，并嵌入指令中  
      case RCM:        // Integer register in command byte  
        // 注意：此处下标为 0  
        DecodeRG(cmd\[0\], datasize, RCM);  
        break;  
        // 4 字节长度整型寄存器  
      case RG4:        // Integer 4-byte register in Reg field  
        if (size < 2)  
          da->error = DAE\_CROSS;  
        else  
        {  
          DecodeRG(cmd\[1\] >> 3, 4, RG4);  
        }  
        hasrm = 1;  
        break;  
        // 累加器寄存器  
      case RAC:        // Accumulator (AL/AX/EAX, implicit)  
        DecodeRG(REG\_EAX, datasize, RAC);  
        break;  
        // AX 寄存器  
      case RAX:        // AX (2-byte, implicit)  
        DecodeRG(REG\_EAX, 2, RAX);   
        break;  
        // DX 寄存器，16 位端口地址  
      case RDX:        // DX (16-bit implicit port address)  
        DecodeRG(REG\_EDX, 2, RDX);   
        break;  
        // CL 寄存器，用于移位  
      case RCL:        // Implicit CL register (for shifts)  
        DecodeRG(REG\_ECX, 1, RCL);  
        break;  
        // FPU 堆顶寄存器  
      case RS0:        // Top of FPU stack (ST(0))  
        DecodeST(0, 0);  
        break;  
        // FPU 寄存器  
      case RST:        // FPU register (ST(i)) in command byte  
        DecodeST(cmd\[0\], 0);  
        break;  
        // MMX 寄存器  
      case RMX:        // MMX register MMx  
        if (size < 2)  
          da->error = DAE\_CROSS;  
        else   
          DecodeMX(cmd\[1\] >> 3);  
        hasrm = 1;  
        break;  
        // 3DNow 寄存器  
      case R3D:        // 3DNow! register MMx  
        if (size < 2)   
          da->error = DAE\_CROSS;  
        else   
          DecodeNR(cmd\[1\] >> 3);  
        hasrm = 1;   
        break;  
      case MRG:             // Memory/register in ModRM byte  
      case MRJ:             // Memory/reg in ModRM as JUMP target  
      case MR1:             // 1-byte memory/register in ModRM byte  
      case MR2:                      // 2-byte memory/register in ModRM byte  
      case MR4:                      // 4-byte memory/register in ModRM byte  
      case MR8:                      // 8-byte memory/MMX register in ModRM  
      case MRD:                      // 8-byte memory/3DNow! register in ModRM  
      case MMA:                      // Memory address in ModRM byte for LEA  
      case MML:                      // Memory in ModRM byte (for LES)  
      case MM6:                      // Memory in ModRm (6-byte descriptor)  
      case MMB:                      // Two adjacent memory locations (BOUND)  
      case MD2:                      // Memory in ModRM byte (16-bit integer)  
      case MB2:                      // Memory in ModRM byte (16-bit binary)  
      case MD4:                      // Memory in ModRM byte (32-bit integer)  
      case MD8:                      // Memory in ModRM byte (64-bit integer)  
      case MDA:                      // Memory in ModRM byte (80-bit BCD)  
      case MF4:                      // Memory in ModRM byte (32-bit float)  
      case MF8:                      // Memory in ModRM byte (64-bit float)  
      case MFA:                      // Memory in ModRM byte (80-bit float)  
      case MFE:                      // Memory in ModRM byte (FPU environment)  
      case MFS:                      // Memory in ModRM byte (FPU state)  
      case MFX:                      // Memory in ModRM byte (ext. FPU state)  
        DecodeMR(arg);  
        break;  
      case MMS:                      // Memory in ModRM byte (as SEG:OFFS)  
        DecodeMR(arg);  
        da->warnings |= DAW\_FARADDR;   
        break;  
      case RR4:                      // 4-byte memory/register (register only)  
      case RR8:                      // 8-byte MMX register only in ModRM  
      case RRD:                      // 8-byte memory/3DNow! (register only)  
        if ((cmd\[1\] & 0xC0)!=0xC0)   
          softerror=DAE\_REGISTER;  
        DecodeMR(arg);  
        break;  
      case MSO:                      // Source in string op's (\[ESI\])  
        DecodeSO();  
        break;  
      case MDE:                      // Destination in string op's (\[EDI\])  
        DecodeDE();  
        break;  
      case MXL:                      // XLAT operand (\[EBX+AL\])  
        DecodeXL();  
        break;  
      case IMM:                      // Immediate data (8 or 16/32)  
      case IMU:                      // Immediate unsigned data (8 or 16/32)  
        if ((pd->bits & SS)!=0 && (\*cmd & 0x02)!=0)  
          DecodeIM(1,datasize,arg);  
        else  
          DecodeIM(datasize,0,arg);  
        break;  
      case VXD:                      // VxD service (32-bit only)  
        DecodeVX();//未解决  
        break;  
      case IMX:                      // Immediate sign-extendable byte  
        DecodeIM(1,datasize,arg);  
        break;  
      case C01:                      // Implicit constant 1 (for shifts)  
        DecodeC1();//未解决  
        break;  
      case IMS:                      // Immediate byte (for shifts)  
      case IM1:                      // Immediate byte  
        DecodeIM(1,0,arg);  
        break;  
      case IM2:                      // Immediate word (ENTER/RET)  
        DecodeIM(2,0,arg);  
        if ((da->immconst & 0x03)!=0)  
          da->warnings|=DAW\_STACK;  
        break;  
      case IMA:                      // Immediate absolute near data address  
        DecodeIA();           //\[imm\] ok  
        break;  
      case JOB:                      // Immediate byte offset (for jumps)  
        DecodeRJ(1,srcip+2);//无须解决 jmpconst 则是  
        break;  
      case JOW:                      // Immediate full offset (for jumps)  
        DecodeRJ(datasize,srcip+datasize+1);//无须解决 jmpconst 则是   
        break;  
      case JMF:                      // Immediate absolute far jump/call addr  
        DecodeJF();//未解决  
        da->warnings|=DAW\_FARADDR;  
        break;  
      case SGM:                      // Segment register in ModRM byte  
        if (size<2)   
          da->error=DAE\_CROSS;  
        DecodeSG(cmd\[1\]>>3);  
        hasrm=1;  
        break;  
      case SCM:                      // Segment register in command byte  
        DecodeSG(cmd\[0\]>>3);  
        if ((da->cmdtype & C\_TYPEMASK)==C\_POP)  
          da->warnings|=DAW\_SEGMENT;  
        break;  
      case CRX:                      // Control register CRx  
        if ((cmd\[1\] & 0xC0)!=0xC0)   
          da->error=DAE\_REGISTER;  
        DecodeCR(cmd\[1\]);//未解决  
        break;  
      case DRX:                      // Debug register DRx  
        if ((cmd\[1\] & 0xC0)!=0xC0)  
          da->error=DAE\_REGISTER;  
        DecodeDR(cmd\[1\]);//未解决  
        break;  
      case PRN:                      // Near return address (pseudooperand)  
        break;  
      case PRF:                      // Far return address (pseudooperand)  
        da->warnings|=DAW\_FARADDR;  
        break;  
      case PAC:                      // Accumulator (AL/AX/EAX, pseudooperand)  
        DecodeRG(REG\_EAX,datasize,PAC);  
        break;  
      case PAH:                      // AH (in LAHF/SAHF, pseudooperand)  
      case PFL:                      // Lower byte of flags (pseudooperand)  
        break;  
      case PS0:                      // Top of FPU stack (pseudooperand)  
        DecodeST(0,1);//未解决  
        break;  
      case PS1:                      // ST(1) (pseudooperand)  
        DecodeST(1,1);//未解决  
        break;  
      case PCX:                      // CX/ECX (pseudooperand)  
        DecodeRG(REG\_ECX,cxsize,PCX);   
        break;  
      case PDI:                      // EDI (pseudooperand in MMX extentions)  
        DecodeRG(REG\_EDI,4,PDI);  
        break;  
      default:  
        da->error=DAE\_INTERN;        // Unknown argument type  
        break;  
      }  
      da->memsize\[operand\] = datasize;  
    }  
    // Check whether command may possibly contain fixups.  
    if (pfixup != NULL && da->fixupsize > 0)  
      da->fixupoffset = (int)(pfixup - src);  
    // Segment prefix and address size prefix are superfluous for command which  
    // does not access memory. If this the case, mark command as rare to help  
    // in analysis.  
    // 段前缀与地址大小前缀过多  
    if (da->memtype == DEC\_UNKNOWN &&  
      (segprefix != SEG\_UNDEF || (addrsize != 4 && pd->name\[0\] != '$')))   
    {  
      da->warnings |= DAW\_PREFIX;  
      da->cmdtype |= C\_RARE;   
    }  
    // 16-bit addressing is rare in 32-bit programs. If this is the case,  
    // mark command as rare to help in analysis.  
    // 16 位的寻址在 32 位程序中是少见的,如果是这种情况,在命令中加上标志  
    if (addrsize != 4)  
      da->cmdtype |= C\_RARE;  
  }  
  // Suffix of 3DNow! command is accounted best by assuming it immediate byte  
  // constant.  
  // 如果是一个 3dNow 指令  
  if (is3dnow)   
  {  
      if (immsize != 0)  
      da->error = DAE\_BADCMD;  
    else  
      immsize = 1;  
  }  
  // Right or wrong, command decoded. Now dump it.  
  if (da->error != 0)    // 如果命令有错误  
  {  
    // Hard error in command detected  
      if (mode >= DISASM\_FILE)  
      nresult = sprintf(da->result, "???");  
    if (da->error == DAE\_BADCMD && (\*cmd == 0x0F || \*cmd == 0xFF) && size > 0)  
    {  
      if (mode >= DISASM\_FILE)  
      {  
        ndump += sprintf(da->dump + ndump, "%02X", \*cmd);  
        da->hexcode\[da->codelen++\] = \*cmd;  
      }  
      cmd++;  
      size--;  
    }  
    if (size>0)  
    {  
      if (mode >= DISASM\_FILE)  
      {  
        ndump += sprintf(da->dump + ndump, "%02X", \*cmd);  
        da->hexcode\[da->codelen++\] = \*cmd;  
      }  
      cmd++;   
      size--;  
    }  
  }  
  else  
  {  
    //如果没有错误,dump 命令  
    // No hard error, dump command  
    if (mode >= DISASM\_FILE)   
    {  
      ndump += sprintf(da->dump + ndump, "%02X", \*cmd++);  
      da->hexcode\[da->codelen++\] = \*(cmd - 1);  
  
      if (hasrm)  
      {  
        ndump += sprintf(da->dump + ndump, "%02X", \*cmd++);  
        da->hexcode\[da->codelen++\] = \*(cmd - 1);  
      }  
      if (hassib)  
      {  
        ndump += sprintf(da->dump + ndump, "%02X", \*cmd++);  
        da->hexcode\[da->codelen++\] = \*(cmd - 1);  
      }  
      if (dispsize != 0)   
      {  
        da->dump\[ndump++\] = ' ';  
        for (i = 0; i < dispsize; i++)  
        {  
          ndump += sprintf(da->dump + ndump, "%02X", \*cmd++);  
          da->hexcode\[da->codelen++\] = \*(cmd - 1);  
        }  
      }  
      if (immsize != 0)  
      {  
        da->dump\[ndump++\] = ' ';  
        for (i = 0; i < immsize; i++)  
        {  
          ndump += sprintf(da->dump + ndump, "%02X", \*cmd++);  
          da->hexcode\[da->codelen++\] = \*(cmd - 1);  
        }  
      }  
    }  
    else  
      cmd += 1 + hasrm + hassib + dispsize + immsize;  
    size -= 1 + hasrm + hassib + dispsize + immsize;  
  }  
  // Check that command is not a dangerous one.  
  // 检查命令是否是危险的  
  if (mode >= DISASM\_DATA)   
  {  
    for (pdan = dangerous; pdan->mask != 0; pdan++)  
    {  
      if (((code ^ pdan->code) & pdan->mask) != 0)  
        continue;  
      if (pdan->type == C\_DANGERLOCK && lockprefix == 0)  
        break;    // Command harmless without LOCK prefix  
      if (iswindowsnt && pdan->type == C\_DANGER95)  
        break;    // Command harmless under Windows NT  
      // Dangerous command!  
      if (pdan->type == C\_DANGER95)  
        da->warnings |= DAW\_DANGER95;  
      else  
        da->warnings |= DAW\_DANGEROUS;  
      break;  
    }  
  }  
  if (da->error == 0 && softerror != 0)  
    da->error = softerror;  // Error, but still display command  
  if (mode >= DISASM\_FILE)  
  {  
    // 根据错误处理  
    if (da->error != DAE\_NOERR)  
    {  
      switch (da->error)  
      {  
      case DAE\_CROSS:  
        strcpy(da->comment, "Command crosses end of memory block");  
        break;  
      case DAE\_BADCMD:  
        strcpy(da->comment, "Unknown command");  
        break;  
      case DAE\_BADSEG:  
        strcpy(da->comment, "Undefined segment register");  
        break;  
      case DAE\_MEMORY:  
        strcpy(da->comment, "Illegal use of register");  
        break;  
      case DAE\_REGISTER:  
        strcpy(da->comment, "Memory address not allowed");  
        break;  
      case DAE\_INTERN:  
        strcpy(da->comment, "Internal OLLYDBG error");  
        break;  
      default:  
        strcpy(da->comment, "Unknown error");  
        break;  
      }  
    }  
    // 特权指令，发出警告  
    else if ((da->warnings & DAW\_PRIV) != 0 && privileged == 0)  
      strcpy(da->comment, "Privileged command");  
    // IO 指令  
    else if ((da->warnings & DAW\_IO) != 0 && iocommand == 0)  
      strcpy(da->comment, "I/O command");  
    else if ((da->warnings & DAW\_FARADDR) != 0 && farcalls == 0)  
    {  
      if ((da->cmdtype & C\_TYPEMASK) == C\_JMP)  
        strcpy(da->comment, "Far jump");  
      else if ((da->cmdtype & C\_TYPEMASK) == C\_CAL)  
        strcpy(da->comment, "Far call");  
      else if ((da->cmdtype & C\_TYPEMASK) == C\_RET)  
        strcpy(da->comment, "Far return");  
    }  
    else if ((da->warnings & DAW\_SEGMENT) != 0 && farcalls == 0)  
      strcpy(da->comment, "Modification of segment register");  
    else if ((da->warnings & DAW\_SHIFT) != 0 && badshift == 0)  
      strcpy(da->comment, "Shift constant out of range 1..31");  
    else if ((da->warnings & DAW\_PREFIX) != 0 && extraprefix == 0)  
      strcpy(da->comment, "Superfluous prefix");  
    else if ((da->warnings & DAW\_LOCK) != 0 && lockedbus == 0)  
      strcpy(da->comment, "LOCK prefix");  
    else if ((da->warnings & DAW\_STACK) != 0 && stackalign == 0)  
      strcpy(da->comment, "Unaligned stack operation");  
  }  
  return (srcsize - size);  // Returns number of recognized bytes  
}  
  
  
// --------------------------- disasm.h -------------  
// Free Disassembler and Assembler -- Header file  
//  
// Copyright (C) 2001 Oleh Yuschuk  
//  
//  This program is free software; you can redistribute it and/or modify  
//  it under the terms of the GNU General Public License as published by  
//  the Free Software Foundation; either version 2 of the License, or  
//  (at your option) any later version.  
//  
//  This program is distributed in the hope that it will be useful,  
//  but WITHOUT ANY WARRANTY; without even the implied warranty of  
//  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the  
//  GNU General Public License for more details.  
//  
//  You should have received a copy of the GNU General Public License  
//  along with this program; if not, write to the Free Software  
//  Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA  
#pragma once  
  
#ifndef MAINPROG  
#define odunique extern  
#else  
#define odunique  
#endif  
  
#pragma   warning(disable:4309)//关闭截断常数值警告  
  
// If you prefere Borland, this will force necessary setting (but, as a side  
// effect, may cause plenty of warnings if other include files will be compiled  
// with different options):  
  
#ifdef \_\_BORLANDC\_\_  
#pragma option -K                      // Unsigned char  
#endif  
//  
//#if (char)0xFF != 255  
//#error Please set default char type to unsigned  
//#endif  
  
  
#define NEGLIMIT       (-16384)        // Limit to display constans as signed  
  
// 假操作数，伪操作数  
#define PSEUDOOP       128             // Base for pseudooperands  
#define TEXTLEN        256             // Maximal length of text string  
  
// Special command features.  
#define WW             0x01            // Bit W (size of operand)  
#define SS             0x02            // Bit S (sign extention of immediate)  
#define WS             0x03            // Bits W and S  
#define W3             0x08            // Bit W at position 3  
#define CC             0x10            // Conditional jump  
// 强制要求 16 位长度的操作数  
#define FF             0x20            // Forced 16-bit size  
#define LL             0x40            // Conditional loop  
  
// 特权指令  
#define PR             0x80            // Protected command  
  
// IO 指令  
#define WP             0x81            // I/O command with bit W  
  
// All possible types of operands in 80x86. A bit more than you expected, he?  
// 操作数类型  
#define NNN            0               // No operand \\\\ 无操作  
  
// 寄存器操作数  
// 对于汇编指令，有些指令的操作数是寄存器，并且在指令中需指定操作数是哪个寄存器  
//         而有些指令的操作数是寄存器，但不需要指定用哪个寄存器，而是指令  
//         一旦确定下来，则使用的寄存器也就确定了 \[??\]  
// 下面两个类型是否分别指上面的两种情况  
#define REG            1               // Integer register in Reg field             \\\\积存器结构中的寄存器  
#define RCM            2               // Integer register in command byte          \\\\命令字节中的寄存器  
#define RG4            3               // Integer 4-byte register in Reg field      \\\\用不到 PMOVMSKB 指令  
#define RAC            4               // Accumulator (AL/AX/EAX, implicit)         \\\\暗示累加器?  
#define RAX            5               // AX (2-byte, implicit)            \\\\暗示 AX  
#define RDX            6               // DX (16-bit implicit port address)  
#define RCL            7               // Implicit CL register (for shifts)         \\\\暗示 CL 寄存器  
#define RS0            8               // Top of FPU stack (ST(0), implicit)  
  
// FPU 寄存器操作数  
#define RST            9               // FPU register (ST(i)) in command byte  
  
// MMX 寄存器操作数  
#define RMX            10              // MMX register MMx  
#define R3D            11              // 3DNow! register MMx  
#define MRG            12              // Memory/register in ModRM byte  
#define MR1            13              // 1-byte memory/register in ModRM byte  
#define MR2            14              // 2-byte memory/register in ModRM byte  
#define MR4            15              // 4-byte memory/register in ModRM byte  
#define RR4            16              // 4-byte memory/register (register only)  
#define MR8            17              // 8-byte memory/MMX register in ModRM  
#define RR8            18              // 8-byte MMX register only in ModRM  
#define MRD            19              // 8-byte memory/3DNow! register in ModRM  
#define RRD            20              // 8-byte memory/3DNow! (register only)  
#define MRJ            21              // Memory/reg in ModRM as JUMP target  
#define MMA            22              // Memory address in ModRM byte for LEA  
#define MML            23              // Memory in ModRM byte (for LES)  
#define MMS            24              // Memory in ModRM byte (as SEG:OFFS)  
#define MM6            25              // Memory in ModRm (6-byte descriptor)  
#define MMB            26              // Two adjacent memory locations (BOUND)  
#define MD2            27              // Memory in ModRM (16-bit integer)  
#define MB2            28              // Memory in ModRM (16-bit binary)  
#define MD4            29              // Memory in ModRM byte (32-bit integer)  
#define MD8            30              // Memory in ModRM byte (64-bit integer)  
#define MDA            31              // Memory in ModRM byte (80-bit BCD)  
#define MF4            32              // Memory in ModRM byte (32-bit float)  
#define MF8            33              // Memory in ModRM byte (64-bit float)  
#define MFA            34              // Memory in ModRM byte (80-bit float)  
#define MFE            35              // Memory in ModRM byte (FPU environment)  
#define MFS            36              // Memory in ModRM byte (FPU state)  
#define MFX            37              // Memory in ModRM byte (ext. FPU state)  
#define MSO            38              // Source in string op's (\[ESI\])  
#define MDE            39              // Destination in string op's (\[EDI\])  
  
// XLAT 操作数  
#define MXL            40              // XLAT operand (\[EBX+AL\])  
  
// 直接数  
#define IMM            41              // Immediate data (8 or 16/32)  
#define IMU            42              // Immediate unsigned data (8 or 16/32)  
#define VXD            43              // VxD service  
#define IMX            44              // Immediate sign-extendable byte  
#define C01            45              // Implicit constant 1 (for shifts)  
#define IMS            46              // Immediate byte (for shifts)  
#define IM1            47              // Immediate byte  
#define IM2            48              // Immediate word (ENTER/RET)  
#define IMA            49              // Immediate absolute near data address  
#define JOB            50              // Immediate byte offset (for jumps)  
#define JOW            51              // Immediate full offset (for jumps)  
  
// 操作数是 JMP 跳转或 call 的地址  
#define JMF            52              // Immediate absolute far jump/call addr  
  
// 段寄存器操作  
#define SGM            53              // Segment register in ModRM byte  
#define SCM            54              // 段寄存器      Segment register in command byte  
  
// 控制寄存器操作数  
#define CRX            55              // CRx 控制寄存器 Control register CRx  
  
// 调试寄存器操作数  
#define DRX            56              // DRx 调试寄存器 Debug register DRx  
// Pseudooperands (implicit operands, never appear in assembler commands). Must  
// have index equal to or exceeding PSEUDOOP.  
#define PRN            (PSEUDOOP+0)    // Near return address  
#define PRF            (PSEUDOOP+1)    // Far return address  
#define PAC            (PSEUDOOP+2)    // Accumulator (AL/AX/EAX)  
#define PAH            (PSEUDOOP+3)    // AH (in LAHF/SAHF commands)  
#define PFL            (PSEUDOOP+4)    // Lower byte of flags (in LAHF/SAHF)  
#define PS0            (PSEUDOOP+5)    // Top of FPU stack (ST(0))  
#define PS1            (PSEUDOOP+6)    // ST(1)  
#define PCX            (PSEUDOOP+7)    // CX/ECX  
#define PDI            (PSEUDOOP+8)    // EDI (in MMX extentions)  
  
// Errors detected during command disassembling.  
#define DAE\_NOERR      0               // No error  
  
// 错误的指令  
#define DAE\_BADCMD     1               // Unrecognized command  
#define DAE\_CROSS      2               // Command crosses end of memory block  
#define DAE\_BADSEG     3               // Undefined segment register  
  
// 当操作数必须是内存操作数，而发现操作数是寄存器时出的错误  
#define DAE\_MEMORY     4               // Register where only memory allowed  
#define DAE\_REGISTER   5               // Memory where only register allowed  
// 内部错误  
#define DAE\_INTERN     6               // Internal error  
  
typedef unsigned char  uchar;          // Unsigned character (byte)  
typedef unsigned short ushort;         // Unsigned short  
typedef unsigned int   uint;           // Unsigned integer  
typedef unsigned long  ulong;          // Unsigned long  
  
typedef struct t\_addrdec  
{  
  int            defseg;  
  char           \*descr;  
} t\_addrdec;  
  
// 命令数据  
typedef struct t\_cmddata  
{  
  ulong          mask;                 // Mask for first 4 bytes of the command  
  ulong          code;                 // Compare masked bytes with this  
  char           len;                  // Length of the main command code  
  
  // 例如 PR，则表明该指令是一特权指令(ring 0 指令)  
  // FF：强制操作数为 16 位操作数  
  char           bits;                 // Special bits within the command  
  char           arg1,arg2,arg3;       // Types of possible arguments  
  char           type;                 // C\_xxx + additional information  
  char           \*name;                // Symbolic name for this command  
} t\_cmddata;  
  
// Initialized constant data structures used by all programs from assembler  
// package. Contain names of register, register combinations or commands and  
// their properties.  
extern const char      \*regname\[3\]\[9\];  
extern const char      \*segname\[8\];  
extern const char      \*sizename\[11\];  
extern const t\_addrdec addr16\[8\];  
extern const t\_addrdec addr32\[8\];  
extern const char      \*fpuname\[9\];  
extern const char      \*mmxname\[9\];  
extern const char      \*crname\[9\];  
extern const char      \*drname\[9\];  
extern const char      \*condition\[16\];  
extern const t\_cmddata cmddata\[\];//可用结构数组  
extern const t\_cmddata vxdcmd;  
extern const t\_cmddata dangerous\[\];  
  
////////////////////////////////////////////////////////////////////////////////  
//////////////////// ASSEMBLER, DISASSEMBLER AND EXPRESSIONS ///////////////////  
  
#define MAXCMDSIZE     16              // Maximal length of 80x86 command  
#define MAXCALSIZE     8               // Max length of CALL without prefixes  
#define NMODELS        8               // Number of assembler search models  
  
#define INT3           0xCC            // Code of 1-byte breakpoint  
#define NOP            0x90            // Code of 1-byte NOP command  
#define TRAPFLAG       0x00000100      // Trap flag in CPU flag register  
  
#define REG\_EAX        0               // Indexes of general-purpose registers  
#define REG\_ECX        1               // in t\_reg.  
#define REG\_EDX        2  
#define REG\_EBX        3  
#define REG\_ESP        4  
#define REG\_EBP        5  
#define REG\_ESI        6  
#define REG\_EDI        7  
  
#define SEG\_UNDEF     -1         //没有段寄存器  
#define SEG\_ES         0               // Indexes of segment/selector registers  
#define SEG\_CS         1  
#define SEG\_SS         2  
#define SEG\_DS         3  
#define SEG\_FS         4  
#define SEG\_GS         5  
  
// 指令类型  
#define C\_TYPEMASK     0xF0            // Mask for command type  
#define   C\_CMD        0x00            // Ordinary instruction  
#define   C\_PSH        0x10            // 1-word PUSH instruction  
#define   C\_POP        0x20            // 1-word POP instruction  
#define   C\_MMX        0x30            // MMX instruction  
#define   C\_FLT        0x40            // FPU instruction  
// jmp 跳转指令  
#define   C\_JMP        0x50            // JUMP instruction  
// jmp 条件跳转指令  
#define   C\_JMC        0x60            // Conditional JUMP instruction  
// call 调用指令  
#define   C\_CAL        0x70            // CALL instruction  
#define   C\_RET        0x80            // RET instruction  
#define   C\_\*\*\*        0x90            // Changes system flags  
#define   C\_RTF        0xA0            // C\_JMP and C\_\*\*\* simultaneously  
#define   C\_REP        0xB0            // Instruction with REPxx prefix  
#define   C\_PRI        0xC0            // Privileged instruction  
#define   C\_DAT        0xD0            // Data (address) doubleword  
  
// 3DNow 指令  
#define   C\_NOW        0xE0            // 3DNow! instruction  
// 错误指令，即未注册的指令  
#define   C\_BAD        0xF0            // Unrecognized command  
#define C\_RARE         0x08            // Rare command, seldom used in programs  
#define C\_SIZEMASK     0x07            // MMX data size or special flag  
#define   C\_EXPL       0x01            // (non-MMX) Specify explicit memory size  
  
#define C\_DANGER95     0x01            // Command is dangerous under Win95/98  
#define C\_DANGER       0x03            // Command is dangerous everywhere  
#define C\_DANGERLOCK   0x07            // Dangerous with LOCK prefix  
  
#define DEC\_TYPEMASK   0x1F            // Type of memory byte  
#define   DEC\_UNKNOWN  0x00            // Unknown type  
#define   DEC\_BYTE     0x01            // Accessed as byte  
#define   DEC\_WORD     0x02            // Accessed as short  
#define   DEC\_NEXTDATA 0x03            // Subsequent byte of code or data  
#define   DEC\_DWORD    0x04            // Accessed as long  
#define   DEC\_FLOAT4   0x05            // Accessed as float  
#define   DEC\_FWORD    0x06            // Accessed as descriptor/long pointer  
#define   DEC\_FLOAT8   0x07            // Accessed as double  
#define   DEC\_QWORD    0x08            // Accessed as 8-byte integer  
#define   DEC\_FLOAT10  0x09            // Accessed as long double  
#define   DEC\_TBYTE    0x0A            // Accessed as 10-byte integer  
#define   DEC\_STRING   0x0B            // Zero-terminated ASCII string  
#define   DEC\_UNICODE  0x0C            // Zero-terminated UNICODE string  
// 3DNow 指令操作数  
#define   DEC\_3DNOW    0x0D            // Accessed as 3Dnow operand  
#define   DEC\_BYTESW   0x11            // Accessed as byte index to switch  
#define   DEC\_NEXTCODE 0x13            // Subsequent byte of command  
#define   DEC\_COMMAND  0x1D            // First byte of command  
#define   DEC\_JMPDEST  0x1E            // Jump destination  
#define   DEC\_CALLDEST 0x1F            // Call (and maybe jump) destination  
#define DEC\_PROCMASK   0x60            // Procedure analysis  
#define   DEC\_PROC     0x20            // Start of procedure  
#define   DEC\_PBODY    0x40            // Body of procedure  
#define   DEC\_PEND     0x60            // End of procedure  
#define DEC\_CHECKED    0x80            // Byte was analysed  
  
#define DECR\_TYPEMASK  0x3F            // Type of register or memory  
#define   DECR\_BYTE    0x21            // Byte register  
#define   DECR\_WORD    0x22            // Short integer register  
#define   DECR\_DWORD   0x24            // Long integer register  
#define   DECR\_QWORD   0x28            // MMX register  
#define   DECR\_FLOAT10 0x29            // Floating-point register  
#define   DECR\_SEG     0x2A            // Segment register  
#define   DECR\_3DNOW   0x2D            // 3Dnow! register  
#define DECR\_ISREG     0x20            // Mask to check that operand is register  
  
/\*\*  
 \* 反汇编的选项  
 \*/  
#define DISASM\_SIZE    0               // Determine command size only  
#define DISASM\_DATA    1               // Determine size and analysis data  
#define DISASM\_FILE    3               // Disassembly, no symbols  
#define DISASM\_CODE    4               // Full disassembly  
  
// Warnings issued by Disasm():  
#define DAW\_FARADDR    0x0001          // Command is a far jump, call or return  
  
// 当执行如 MOV SEG, XX 的指令时警告  
#define DAW\_SEGMENT    0x0002          // Command loads segment register  
  
// 特权指令  
#define DAW\_PRIV       0x0004          // Privileged command  
// IO 指令  
#define DAW\_IO         0x0008          // I/O command  
#define DAW\_SHIFT      0x0010          // Shift constant out of range 1..31  
  
// 表示指令有过多的前缀  
#define DAW\_PREFIX     0x0020          // Superfluous prefix  
  
// 指令有加锁总线前缀  
#define DAW\_LOCK       0x0040          // Command has LOCK prefix  
  
// 栈没有对齐(一般栈操作需要双字对齐)  
#define DAW\_STACK      0x0080          // Unaligned stack operation  
#define DAW\_DANGER95   0x1000          // May mess up Win95 if executed  
#define DAW\_DANGEROUS  0x3000          // May mess up any OS if executed  
  
//bughoho 操作数类型  
enum Optype  
{  
  // 立即数  
  Imm,  
  // 寄存器  
  Reg,  
  // 内存数  
  Mem,  
  // 段寄存器  
  Seg,  
};  
  
// Results of disassembling  
/\*\*  
 \* 用于保存反汇编的结果  
 \*/  
typedef struct t\_disasm  
{  
  // 应该是指令本身所在的地址  
  ulong          ip;                   // Instrucion pointer  
  char           dump\[TEXTLEN\];        // Hexadecimal dump of the command  
  
  // 用于保存反汇编指令  
  char           result\[TEXTLEN\];      // Disassembled command  
  
  // 对指令的一些说明信息，比如可包含信息"Superfluous prefix"以表示指令的前缀过多  
  char           comment\[TEXTLEN\];     // Brief comment  
  
  // 表示指令的类型  
  // C\_RARE: 表示指令比较罕见，在程序中不常用  
  // C\_CAL：表示是一个 CALL 指令  
  //  
  int            cmdtype;              // One of C\_xxx  
  
  // DEC\_BYTE：表示是一个字节内存操作数  
  // DEC\_3DNOW：表示是一个 3dNow 指令操作数  
  int            memtype;              // Type of addressed variable in memory  
  
  // 统计指令的前缀数  
  int            nprefix;              // Number of prefixes  
  // 地址是否存在寄存器(index)  
  int            indexed;              // Address contains register(s)  
  
  // 常量跳转地址 \[??\]  
  ulong          jmpconst;             // Constant jump address  
  
  // 跳转表，可能赋值为：jmptable = addr;  
  ulong          jmptable;             // Possible address of switch table  
  ulong          adrconst;             // \[edx+100\] '100' 地址部分的常量 Constant part of address  
  
  // 保存操作数（立即数）的值  
  ulong          immconst;             // Immediate constant  
  
  // 1：表示 adrconst 为 0  
  int            zeroconst;            // Whether contains zero constant  
  int            fixupoffset;          // Possible offset of 32-bit fixups  
  int            fixupsize;            // Possible total size of fixups or 0  
  int            error;                // Error while disassembling command  
  
  // 反汇编过程中需要进行警告的信息  
  // 如 DAW\_PRIV，则表明指令是一个特权指令(ring 0)指令  
  int            warnings;             // Combination of DAW\_xxx  
  
  //bughoho new  
  BYTE       hexcode\[TEXTLEN\];   // 用于存储 2 进制码  
  int         codelen;         // 长度  
  
  int         optype\[3\];         // 操作数类型  
  char       vm\_name\[TEXTLEN\];   // 声称 VM 对应的 Handler 名称  
  
  // 3dNow 指令  
  int         is3dnow;         // 3dnow 函数  
  int         segment;         // 保存段前缀  
  
  // 当操作数为寄存器时，存储寄存器编号  
  int         reg\[3\];         // 3 寄存器(假设操作数为寄存器)  
  int         segreg;         // 段寄存器(假设操作数为段寄存器)  
  int         addrreg1;       // 内存地址部分的第 1 个不带比例的寄存器  
  int         addrreg2;       // 内存地址部分的第 2 个带比例的寄存器  
  int         regsscale;       // 比例:1,2,4,8  
  //还有一个 adrconst 已经定义,根据正负号来定义加减  
  int         memsize\[3\];       // 操作数长度  
  
  // 当某个操作数是(ah,bh,ch,dh)时，即寄存器的高 8 位时，则相应元素置 true  
  bool       highbit\[3\];       // 当是 8 位指令时并且为高位时(ah bh ch dh)为 1  
} t\_disasm;  
  
/\*\*  
 \* 用于保存反汇编的每个指令语句  
 \*/  
struct CodeNode  
{  
  t\_disasm  disasm;  
  
  // 表示此指令是跳到外部的指令  
  BOOL    IsJmcBeSideType;  
  
  // 指令跳转地址不明确  
  BOOL    IsJmcUndefineType;  
  
  // 此指令是一动态跳转指令  
  // 注：动态跳转指令指没有指明跳转地址的跳转指令  
  BOOL    IsJmcDynamicType;  
  
  // TRUE 表示指令是一跳转指令  
  BOOL    IsJmcType;  
  
  // 表示此指令是否是从其他指令处跳转过来  
  BOOL    IsJmcFromType;  
  
  // 表示此指令是 JMC 或 CALL 指令后的指令  
  BOOL    IsJmcNextType;  
  
  // 表示该指令是一 CALL 指令  
  BOOL    IsCallType;  
  CodeNode()  
  {  
    memset(this,0,sizeof(CodeNode));  
  }  
};  
  
/\*\*  
 \* 虚拟机指令相应的汇编代码必须编译成二进制机器代码，此结构就用于  
 \* 表示某段编译之后的二进制机器代码及代码的一些相关信息  
 \*/  
typedef struct t\_asmmodel  
{            // Model to search for assembler command  
  // 存储解码后的机器指令  
  char           code\[MAXCMDSIZE\];     // Binary code  
  char           mask\[MAXCMDSIZE\];     // Mask for binary code (0: bit ignored)  
  // 指令长度  
  int            length;               // Length of code, bytes (0: empty)  
  int            jmpsize;              // Offset size if relative jump  
  int            jmpoffset;            // Offset relative to IP  
  int            jmppos;               // Position of jump offset in command  
} t\_asmmodel;  
  
odunique int       ideal;                // Force IDEAL decoding mode  
odunique int       lowercase;            // Force lowercase display  
odunique int       tabarguments;         // Tab between mnemonic and arguments  
odunique int       extraspace;           // Extra space between arguments  
odunique int       putdefseg;            // Display default segments in listing  
odunique int       showmemsize;          // Always show memory size  
odunique int       shownear;             // Show NEAR modifiers  
odunique int       shortstringcmds;      // Use short form of string commands  
odunique int       sizesens;             // How to decode size-sensitive mnemonics  
odunique int       symbolic;             // Show symbolic addresses in disasm  
odunique int       farcalls;             // Accept far calls, returns & addresses  
odunique int       decodevxd;            // Decode VxD calls (Win95/98)  
odunique int       privileged;           // Accept privileged commands  
odunique int       iocommand;            // Accept I/O commands  
odunique int       badshift;             // Accept shift out of range 1..31  
odunique int       extraprefix;          // Accept superfluous prefixes  
odunique int       lockedbus;            // Accept LOCK prefixes  
odunique int       stackalign;           // Accept unaligned stack operations  
odunique int       iswindowsnt;          // When checking for dangers, assume NT  
//bughoho  
odunique int     stoperand;       //当前的循环  
  
int    Assemble(char \*cmd,ulong ip,t\_asmmodel \*model,int attempt,  
        int constsize,char \*errtext);  
int    Checkcondition(int code,ulong flags);  
int    Decodeaddress(ulong addr,char \*symb,int nsymb,char \*comment);  
ulong  Disasm(char \*src,ulong srcsize,ulong srcip,  
        t\_disasm \*disasm,int disasmmode);  
ulong  Disassembleback(char \*block,ulong base,ulong size,ulong ip,int n);  
ulong  Disassembleforward(char \*block,ulong base,ulong size,ulong ip,int n);  
int    Isfilling(ulong addr,char \*data,ulong size,ulong align);  
int    Print3dnow(char \*s,char \*f);  
int    Printfloat10(char \*s,long double ext);  
int    Printfloat4(char \*s,float f);  
int    Printfloat8(char \*s,double d);
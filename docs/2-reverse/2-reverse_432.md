

# IDA Pro 4.x 边界线

> 原文：[`www.pediy.com/kssd/pediy05/pediy50549.htm`](https://www.pediy.com/kssd/pediy05/pediy50549.htm)

IDA Pro 4.x 边界线的最后解决,顺便去掉了一个 IDA 本身的小错误   
以下是源代码:   
  
/\*   
\*  This is a Border line patch for IDA Pro 4.x   
\*  By Xiao Han-Qing (bpx)   
\*    April 5, 2003   
\*/   
  
#include <ida.hpp>   
#include <idp.hpp>   
#include <bytes.hpp>   
#include <loader.hpp>   
#include <kernwin.hpp>   
  
// classic typedefs   
typedef unsigned char        BYTE;   
typedef unsigned short        WORD;   
typedef unsigned int        DWORD;   
  
typedef bool (\*BorderFunc)(void);   
  
#define I\_Call                0xe8   
#define I\_Jmp                0xe9   
#define SizeOfCall            5   
  
#define GetRealAddrOf(func) (BYTE\*)\*\*(BorderFunc\*\*)((char\*)(func) + 2)   
#define Call(ptr)            ptr + SizeOfCall + \*(int\*)(ptr + 1)   
  
// Sig for MakeBorder   
#define Sig1                0xc4684b6a   
  
// Sig for MakeSolidBorder   
#define Sig2                0xdb684b6a   
  
// Sig for SegBorder   
#define Sig3                0xcd684b6a   
  
// 选择你喜欢的边界线，反注释   
  
//-------------------------------------   
// For English Windows OS   
//-------------------------------------   
//#define ThinBorder    0x2d2d9b866 // --   
//#define SolidBorder  0x3d3db866 // ==   
//#define SegBorder    0x3d3db866 // ==   
//-------------------------------------   
  
// 以下适用于简体中文 Windows!!!   
  
// 文件头方框, 随便选一个吧   
#if 0   
#define UpperLeft    0xb3a9 //'┏'   
#define HorLine        0xa5a9 //'━'   
#define UpperRight    0xb7a9 //'┓'   
#define VerLine        0xa7a9 //'┃'   
#define LowerLeft    0xbba9 //'┗'   
#define LowerRight    0xbfa9 //'┛'   
#else   
#define UpperLeft    0xf9a1 //'※'   
#define HorLine        0xf9a1 //'※'   
#define UpperRight    0xf9a1 //'※'   
#define VerLine        0xf9a1 //'※'   
#define LowerLeft    0xf9a1 //'※'   
#define LowerRight    0xf9a1 //'※'   
#endif   
  
// 段边界, 逻辑边界, 函数边界   
//-------------------------------------   
#define SegBorder    0xf9a1b866 //※   
//-------------------------------------   
#define ThinBorder    0xa5a9b866 //━   
#define SolidBorder 0xfea1b866 //〓   
//------------------------------------   
//#define ThinBorder  0xeea1b866 //☆   
//#define SolidBorder 0xefa1b866 //★   
//------------------------------------   
// #define ThinBorder 0xf0a1b866  //○   
// #define SolidBorder 0xf1a1b866 //●   
//------------------------------------   
// #define ThinBorder 0xf3a1b866  //◇   
// #define SolidBorder 0xf4a1b866 //◆   
//------------------------------------   
// #define ThinBorder 0xf5a1b866  //□   
// #define SolidBorder 0xf6a1b866 //■   
//------------------------------------   
// #define ThinBorder 0xf7a1b866  //△   
// #define SolidBorder 0xf8a1b866 //▲   
//------------------------------------   
  
// windows.h conflicts with IDA's header files, so copy it from MSDN   
extern "C" int VirtualProtect(   
  void\* lpAddress,      // region of committed pages   
  DWORD dwSize,          // size of the region   
  DWORD flNewProtect,    // desired access protection   
  DWORD\* lpflOldProtect  // old protection   
);   
  
DWORD OldProt;   
void Unprotect(void\* addr, DWORD sz)   
{   
    VirtualProtect(addr, sz, 4/\*PAGE\_READWRITE)\*/, &OldProt);   
}   
  
void Protect(void\* addr, DWORD sz)   
{   
    VirtualProtect(addr, sz, OldProt, &OldProt);   
}   
  
// 生成边界的代码, 比原版还要快!   
BYTE code\[\] = {   
    0x57,                        // push edi   
    0x8d, 0x7d, 0x9c,          // lea edi, \[ebp-64h\]   
    0xb9, 0x26, 00, 00, 00,    // mov ecx, 26h   
    0xfc,                        // cld   
    0xf2, 0x66, 0xab,            // repnz stosd   
    0x5f,                        // pop edi   
    0x90                        // nop   
};   
  
// patch 各种边界生成的地方,用我们的快代码!   
bool DoPatch(BYTE\* ptr, int range, DWORD sig, DWORD patch)   
{   
    int i;   
    BYTE\* p0, \*p1;   
  
    for (i = 0; i < range; i++, ptr++)   
        if (\*(DWORD\*)ptr == sig)   
            break;   
  
    if (i == range) return false;   
  
    Unprotect(ptr, sizeof(DWORD) + sizeof(code) + 35);   
  
    \*(DWORD\*)ptr = patch;   
  
    p1 = p0 = ptr + 16;   
    for (i = 0; i < 35; i++) {   
        // mov 'add esp, 0ch' to address right after 'call memset()'   
        if ((\*(DWORD\*)p1 & 0xffffff) == 0xcc483 && p1 != p0) {   
            memmove(p0 + 3, p0, p1 - p0);   
            p1 += 3;   
        }   
  
        // mov byte ptr \[esp-19\], 0 --> mov byte ptr \[esp-18\], 0   
        if (\*(DWORD\*)p1 == 0xe745c6)   
            \*(DWORD\*)p1 = 0xe845c6;   
        p1++;   
    }   
  
    memcpy(ptr + sizeof(DWORD), code, sizeof(code));   
  
    Protect(ptr, sizeof(DWORD) + sizeof(code) + 35);   
  
    return true;   
}   
  
//找到植入代码的空间   
BYTE\* FindSpaceForMyCode()   
{   
    BYTE\* ptr = (BYTE\*)&debug - 6 \* 1024;   
  
    do {   
        while (\*ptr != 0xcc) ptr++;   
        if (\*(DWORD\*)ptr == 0xcccc && \*((DWORD\*)ptr + 1) == NULL)   
            return ptr;   
    } while (ptr++ < (BYTE\*)&debug);   
  
    return NULL;   
}   
  
// 文件头方框转换代码   
void ConvertLine(void)   
{   
\_asm {   
    push eax   
    mov eax, dword ptr \[esp+8\]   
  
Local\_0:   
    cmp byte ptr \[eax\], 0   
    jz Local\_done   
    cmp byte ptr \[eax\], 0c9h   
    jnz Local\_1   
    mov word ptr \[eax\], UpperLeft   
    add eax, 2   
  
Local\_1:   
    cmp byte ptr \[eax\], 0xcd   
    jnz Local\_2   
    cmp byte ptr \[eax+1\], 0xcd   
    jnz Local\_11   
    mov word ptr \[eax\], HorLine   
    add eax, 2   
    jmp short Local\_1   
  
Local\_11:   
    cmp \[eax+1\], 0xbb   
    jnz Local\_12   
    mov word ptr \[eax\], UpperRight   
    jmp short Local\_done   
  
Local\_12:   
    mov word ptr \[eax\], LowerRight   
    jmp short Local\_done   
  
Local\_2:   
    cmp byte ptr \[eax\], 0xc8   
    jnz Local\_3   
    mov word ptr \[eax\], LowerLeft   
    add eax, 2   
    jmp short Local\_1   
  
Local\_3:   
    cmp byte ptr \[eax\], 0xba   
    jnz Local\_4   
    cmp byte ptr \[eax+1\], ' '    //这样去掉了 IDA 一个内部 bug!!! 否则文件头方框旁会多一个 s   
    jz Local\_31                    //4.30 和 4.50 都有这个问题.   
    mov word ptr \[eax\], VerLine   
    mov byte ptr \[eax+2\], 0   
    jmp short Local\_Done   
  
Local\_31:   
    mov word ptr \[eax\], VerLine   
    add eax, 2   
  
Local\_4:   
    cmp byte ptr \[eax\], 0xbb   
    jnz Local\_5   
    mov word ptr \[eax\], UpperRight   
    mov byte ptr \[eax+2\], 0   
    jmp short Local\_done   
  
Local\_5:   
    cmp byte ptr \[eax\], 0xbc   
    jnz Local\_6   
    mov word ptr \[eax\], LowerRight   
    mov byte ptr \[eax+2\], 0   
    jmp short Local\_done   
  
Local\_6:   
    inc eax   
    jmp short Local\_0   
  
Local\_done:   
    pop eax   
    call MakeLine   
}   
}   
  
// 植入我们的代码   
BYTE\* InsertMyCode()   
{   
    BYTE\* ptr = FindSpaceForMyCode();   
  
    if (ptr) {   
        BYTE\* pMakeLine, \*p1, \*p2;   
        int MoveSz;   
  
        pMakeLine = GetRealAddrOf(MakeLine);   
  
        for (p1 = (BYTE\*)ConvertLine; \*(DWORD\*)p1 != 0x24448b50; p1++);   
        for (p2 = p1; \*p2 != I\_Call; p2++);   
  
        MoveSz = p2 - p1;   
        Unprotect(ptr, MoveSz + SizeOfCall);   
  
        memcpy(ptr, p1, MoveSz);   
        p1 = (BYTE\*)ptr + MoveSz;   
        \*p1 = I\_Jmp; // code for 'jmp MakeLine'   
        \*(int\*)(p1 + 1) = pMakeLine - (p1 + SizeOfCall);   
  
        Protect(ptr, MoveSz + SizeOfCall);   
    }   
    return ptr;   
}   
  
// 找到生成文件头方框函数内调用 MakeLine()函数的地方   
BYTE\* FindCallMakeLine()   
{   
    // 从 gen\_idc\_file()这个函数入手   
    BYTE\* ptr = GetRealAddrOf(gen\_idc\_file);   
    BYTE\* MakeLineAddr = GetRealAddrOf(MakeLine);   
    int i = 0;   
  
    while (\*ptr++ != I\_Call); // find and skip the first call   
    while (\*ptr != I\_Call) ptr++;   
    ptr = Call(ptr);   
  
    do {   
        while (\*ptr != I\_Call) {   
            if (++i == 500)   
                return 0;   
            ptr++;   
        }   
  
        if (Call(ptr) == MakeLineAddr)   
            break;   
  
        ptr++;   
  
    } while (1);   
  
    return ptr;   
}   
  
// 在生成文件头方框的函数内截获 MakeLine()函数   
bool PatchCallMakeLine()   
{   
    BYTE\* MakeLineAddr, \*MyCodeAddr;   
  
    if ((MakeLineAddr = FindCallMakeLine()) != NULL &&   
        (MyCodeAddr = InsertMyCode()) != NULL)   
    {   
        // 移花接木在此一举!   
        Unprotect(MakeLineAddr + 1, sizeof(DWORD));   
        \*(int\*)(MakeLineAddr + 1) = MyCodeAddr - (MakeLineAddr + SizeOfCall);   
        Protect(MakeLineAddr + 1, sizeof(DWORD));   
        return true;   
    }   
  
    return false;   
}   
  
bool PatchBorderFuncs(void)   
{   
    BYTE\* ptr;   
    int i;   
  
    if (!PatchCallMakeLine())   
        return false;   
  
    // 获得要 patch 函数的真实地址   
    ptr = GetRealAddrOf(MakeBorder);   
    DoPatch(ptr - 5280, 1500, Sig3, SegBorder);   
    DoPatch(ptr, 80, Sig1, ThinBorder);   
  
    ptr = GetRealAddrOf(MakeSolidBorder);   
    DoPatch(ptr, 80, Sig2, SolidBorder);   
  
    for (i = 0; i < 80; i++) {   
        // 将复制" S U B R O U T I N E "的地址向后移动一个字节,   
        // 否则刚好落在一个汉字的中间.   
        // lea edi, \[ebp-55\] --> lea edi, \[ebp-54\]   
        if ((\*(DWORD\*)ptr & 0xffffff) == 0xab7d8d) {   
            ptr += 2;   
            Unprotect(ptr, sizeof(BYTE));   
            \*(ptr) = 0xac;   
            Protect(ptr, sizeof(BYTE));   
            ptr++;   
        }   
  
        // mov \[ebp-40\], 0DBh --> mov \[ebp - 3f\], 20h   
        if (\*(DWORD\*)ptr == 0xdbc045c6) {   
            Unprotect(ptr, sizeof(DWORD));   
            \*(DWORD\*)ptr = 0x20c145c6;   
            Unprotect(ptr, sizeof(DWORD));   
            return true;   
        }   
        ptr++;   
    }   
  
    return false;   
}   
  
int init(void)   
{   
    if (PatchBorderFuncs())   
        msg("边界线 Patch 安装成功!");   
  
    // 返回这个值告诉 IDA 加载这个 plugin 并干完我   
    // 们的活后就将这个 plugin 从内存中卸载掉   
    return PLUGIN\_SKIP; // never load again   
}   
  
//--------------------------------------------------------------------------   
//   
//      The plugin method   
//   
//      This is the main function of plugin.   
//   
//      It will be called when the user selects the plugin.   
//   
//              arg - the input argument, it can be specified in   
//                    plugins.cfg file. The default is zero.   
//   
//   
  
void run(int arg)   
{   
}   
  
//--------------------------------------------------------------------------   
char comment\[\] = "Border line plugin";   
char help\[\] = "Border line plugin for IDA Pro running under Simplified Chinese Windows";   
  
  
//--------------------------------------------------------------------------   
// This is the preferred name of the plugin module in the menu system   
// The preferred name may be overriden in plugins.cfg file   
char wanted\_name\[\] = "Border line plugin";   
  
char wanted\_hotkey\[\] = "";   
  
//--------------------------------------------------------------------------   
//   
//      PLUGIN DESCRIPTION BLOCK   
//   
//--------------------------------------------------------------------------   
extern "C" plugin\_t PLUGIN = {   
  IDP\_INTERFACE\_VERSION,   
  0,                    // plugin flags   
  init,                // initialize   
  
  NULL, //term,                // terminate. this pointer may be NULL.   
  
  run,                  // invoke plugin   
  
  comment,              // long comment about the plugin   
                        // it could appear in the status line   
                        // or as a hint   
  
  help,                // multiline help about the plugin   
  
  wanted\_name,          // the preferred short name of the plugin   
  wanted\_hotkey        // the preferred hotkey to run the plugin   
};   

样本 1:   
seg000:00401000 ; ※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※   
seg000:00401000 ; ※    This file is generated by The Interactive Disassembler (IDA)        ※   
seg000:00401000 ; ※    Copyright (c) 2002 by DataRescue sa/nv, <ida@datarescue.com>        ※   
seg000:00401000 ; ※            Licensed to: Roger Cross, 1 user, std, 7/2000              ※   
seg000:00401000 ; ※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※   
  
  
seg000:004013F2 ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━   
seg000:004013F3                align 4   
seg000:004013F4   
seg000:004013F4 ; 〓〓〓〓〓〓〓〓 S U B R O U T I N E  〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓   
  
样本 2   
seg000:00401000 ; ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓   
seg000:00401000 ; ┃    This file is generated by The Interactive Disassembler (IDA)        ┃   
seg000:00401000 ; ┃    Copyright (c) 2002 by DataRescue sa/nv, <ida@datarescue.com>        ┃   
seg000:00401000 ; ┃            Licensed to: Roger Cross, 1 user, std, 7/2000              ┃   
seg000:00401000 ; ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛   
  
seg000:004013F2                retn   
seg000:004013F2 sub\_4013E8      endp   
seg000:004013F2   
seg000:004013F2 ; ☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆   
seg000:004013F3                align 4   
seg000:004013F4   
seg000:004013F4 ; ★★★★★★★★ S U B R O U T I N E  ★★★★★★★★★★★★★★★★★★★   
seg000:004013F4  4dd466d0e1e0cc123afbe504da2462ca
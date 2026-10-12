
# IDA Pro 4.x 边界线

> 原文：[`www.pediy.com/kssd/pediy05/pediy50549.htm`](https://www.pediy.com/kssd/pediy05/pediy50549.htm)

IDA Pro 4.x 边界线的最后解决，顺便去掉了一个 IDA 本身的小错误

以下是源代码:

```
/*
*  This is a Border line patch for IDA Pro 4.x
*  By Xiao Han-Qing (bpx)
*   April 5, 2003
*/
#include <ida.hpp>
#include <idp.hpp>
#include <bytes.hpp>
#include <loader.hpp>
#include <kernwin.hpp>

// classic typedefs
typedef unsigned char    BYTE;
typedef unsigned short    WORD;
typedef unsigned int      DWORD;
typedef bool (*BorderFunc)(void);

#define I_Call            0xe8
#define I_Jmp             0xe9
#define SizeOfCall        5
#define GetRealAddrOf(func) (BYTE**)*(BorderFunc**)((char*)(func) + 2)
#define Call(ptr)         ptr + SizeOfCall + *(int*)(ptr + 1)

// Sig for MakeBorder
#define Sig1              0xc4684b6a

// Sig for MakeSolidBorder
#define Sig2              0xdb684b6a

// Sig for SegBorder
#define Sig3              0xcd684b6a

// 选择你喜欢的边界线，反注释
//-------------------------------------
// For English Windows OS
//-------------------------------------
//#define ThinBorder    0x2d2d9b866 // --
//#define SolidBorder   0x3d3db866 // ==
//#define SegBorder     0x3d3db866 // ==
//-------------------------------------

// 以下适用于简体中文 Windows!!!

// 文件头方框, 随便选一个吧
#if 0
#define UpperLeft     0xb3a9 //'┏'
#define HorLine       0xa5a9 //'━'
#define UpperRight    0xb7a9 //'┓'
#define VerLine       0xa7a9 //'┃'
#define LowerLeft     0xbba9 //'┗'
#define LowerRight    0xbfa9 //'┛'
#else
#define UpperLeft     0xf9a1 //'※'
#define HorLine       0xf9a1 //'※'
#define UpperRight    0xf9a1 //'※'
#define VerLine       0xf9a1 //'※'
#define LowerLeft     0xf9a1 //'※'
#define LowerRight    0xf9a1 //'※'
#endif

// 段边界, 逻辑边界, 函数边界
//-------------------------------------
#define SegBorder     0xf9a1b866 //※
//-------------------------------------

#define ThinBorder    0xa5a9b866 //━
#define SolidBorder   0xfea1b866 //〓
//------------------------------------
//#define ThinBorder    0xeea1b866 //☆
//#define SolidBorder   0xefa1b866 //★
//------------------------------------
// #define ThinBorder 0xf0a1b866  //○
// #define SolidBorder 0xf1a1b866 //●
//------------------------------------
// #define ThinBorder 0xf3a1b866  //◇
// #define SolidBorder 0xf4a1b866 //◆
//------------------------------------
// #define ThinBorder 0xf5a1b866  //□
// #define SolidBorder 0xf6a1b866 //■
//------------------------------------
// #define ThinBorder 0xf7a1b866  //△
// #define SolidBorder 0xf8a1b866 //▲
//------------------------------------

// windows.h conflicts with IDA's header files, so copy it from MSDN
extern "C" int VirtualProtect(
  void* lpAddress,      // region of committed pages
  DWORD dwSize,         // size of the region
  DWORD flNewProtect,   // desired access protection
  DWORD* lpflOldProtect // old protection
);

DWORD OldProt;

void Unprotect(void* addr, DWORD sz)
{
    VirtualProtect(addr, sz, 4/*PAGE_READWRITE*/, &OldProt);
}

void Protect(void* addr, DWORD sz)
{
    VirtualProtect(addr, sz, OldProt, &OldProt);
}

// 生成边界的代码, 比原版还要快!
BYTE code[] = {
    0x57,             // push edi
    0x8d, 0x7d, 0x9c, // lea edi, [ebp-64h]
    0xb9, 0x26, 00, 00, 00, // mov ecx, 26h
    0xfc,             // cld
    0xf2, 0x66, 0xab, // repnz stosd
    0x5f,             // pop edi
    0x90             // nop
};

// patch 各种边界生成的地方,用我们的快代码!
bool DoPatch(BYTE* ptr, int range, DWORD sig, DWORD patch)
{
    int i;
    BYTE* p0, *p1;
    for (i = 0; i < range; i++, ptr++)
        if (*(DWORD*)ptr == sig)
            break;
    if (i == range) return false;
    Unprotect(ptr, sizeof(DWORD) + sizeof(code) + 35);
    *(DWORD*)ptr = patch;
    p1 = p0 = ptr + 16;
    for (i = 0; i < 35; i++) {
        // mov 'add esp, 0ch' to address right after 'call memset()'
        if ((*(DWORD*)p1 & 0xffffff) == 0xcc483 && p1 != p0) {
            memmove(p0 + 3, p0, p1 - p0);
            p1 += 3;
        }
        // mov byte ptr [esp-19], 0 --> mov byte ptr [esp-18], 0
        if (*(DWORD*)p1 == 0xe745c6)
            *(DWORD*)p1 = 0xe845c6;
        p1++;
    }
    memcpy(ptr + sizeof(DWORD), code, sizeof(code));
    Protect(ptr, sizeof(DWORD) + sizeof(code) + 35);
    return true;
}

//找到植入代码的空间
BYTE* FindSpaceForMyCode()
{
    BYTE* ptr = (BYTE*)&debug - 6 * 1024;
    do {
        while (*ptr != 0xcc) ptr++;
        if (*(DWORD*)ptr == 0xcccc && *((DWORD*)ptr + 1) == NULL)
            return ptr;
    } while (ptr++ < (BYTE*)&debug);
    return NULL;
}

// 文件头方框转换代码
void ConvertLine(void)
{
_asm {
    push eax
    mov eax, dword ptr [esp+8]
Local_0:
    cmp byte ptr [eax], 0
    jz Local_done
    cmp byte ptr [eax], 0c9h
    jnz Local_1
    mov word ptr [eax], UpperLeft
    add eax, 2
Local_1:
    cmp byte ptr [eax], 0xcd
    jnz Local_2
    cmp byte ptr [eax+1], 0xcd
    jnz Local_11
    mov word ptr [eax], HorLine
    add eax, 2
    jmp short Local_1
Local_11:
    cmp [eax+1], 0xbb
    jnz Local_12
    mov word ptr [eax], UpperRight
    jmp short Local_done
Local_12:
    mov word ptr [eax], LowerRight
    jmp short Local_done
Local_2:
    cmp byte ptr [eax], 0xc8
    jnz Local_3
    mov word ptr [eax], LowerLeft
    add eax, 2
    jmp short Local_1
Local_3:
    cmp byte ptr [eax], 0xba
    jnz Local_4
    cmp byte ptr [eax+1], ' '   //这样去掉了 IDA 一个内部 bug!!! 否则文件头方框旁会多一个 s
    jz Local_31                 //4.30 和 4.50 都有这个问题.
    mov word ptr [eax], VerLine
    mov byte ptr [eax+2], 0
    jmp short Local_Done
Local_31:
    mov word ptr [eax], VerLine
    add eax, 2
Local_4:
    cmp byte ptr [eax], 0xbb
    jnz Local_5
    mov word ptr [eax], UpperRight
    mov byte ptr [eax+2], 0
    jmp short Local_done
Local_5:
    cmp byte ptr [eax], 0xbc
    jnz Local_6
    mov word ptr [eax], LowerRight
    mov byte ptr [eax+2], 0
    jmp short Local_done
Local_6:
    inc eax
    jmp short Local_0
Local_done:
    pop eax
    call MakeLine
}
}

// 植入我们的代码
BYTE* InsertMyCode()
{
    BYTE* ptr = FindSpaceForMyCode();
    if (ptr) {
        BYTE* pMakeLine, *p1, *p2;
        int MoveSz;
        pMakeLine = GetRealAddrOf(MakeLine);
        for (p1 = (BYTE*)ConvertLine; *(DWORD*)p1 != 0x24448b50; p1++);
        for (p2 = p1; *p2 != I_Call; p2++);
        MoveSz = p2 - p1;
        Unprotect(ptr, MoveSz + SizeOfCall);
        memcpy(ptr, p1, MoveSz);
        p1 = (BYTE*)ptr + MoveSz;
        *p1 = I_Jmp; // code for 'jmp MakeLine'
        *(int*)(p1 + 1) = pMakeLine - (p1 + SizeOfCall);
        Protect(ptr, MoveSz + SizeOfCall);
    }
    return ptr;
}

// 找到生成文件头方框函数内调用 MakeLine()函数的地方
BYTE* FindCallMakeLine()
{
    // 从 gen_idc_file()这个函数入手
    BYTE* ptr = GetRealAddrOf(gen_idc_file);
    BYTE* MakeLineAddr = GetRealAddrOf(MakeLine);
    int i = 0;
    while (*ptr++ != I_Call); // find and skip the first call
    while (*ptr != I_Call) ptr++;
    ptr = Call(ptr);
    do {
```

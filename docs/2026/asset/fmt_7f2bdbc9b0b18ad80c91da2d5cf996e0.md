
## `x64Common.c`

```c
#pragma once
#include <ntifs.h>
#include <ntddk.h>
#define NTSTRSAFE_LIB
#include <ntstrsafe.h>
#include <intrin.h>
#pragma pack(1)
struct Attribute
{
    UINT64 offset1 : 16;
    UINT64 p : 1;
    UINT64 dpl : 2;
    UINT64 s : 1;
    UINT64 type : 4;
    UINT64 unuse : 6;
    UINT64 ist : 2;
    UINT64 selector : 16;
    UINT64 offset2 : 16;
};
typedef struct _IDT_ENTRY64 {
    union hightStruct
    {
        UINT64 lower;
        struct Attribute attribute;
    };
    UINT64 hight;
}IDT_ENTRY64, *PIDT_ENTRY64;

typedef struct _IDTR
{
    UINT16 limit;
    UINT64 base;
}IDTR, *PIDTR;

///// <summary>
///// cr4结构体
///// </summary>
typedef union _CR4 {
    UINT64 value;
    struct
    {
        UINT64 VME : 1;
        UINT64 PVI : 1;
        UINT64 TSD : 1;
        UINT64 DE : 1;
        UINT64 PSE : 1;
        UINT64 PAE : 1;
        UINT64 MCE : 1;
        UINT64 PGE : 1;
        UINT64 PCE : 1;
        UINT64 OSFXSR : 1;
        UINT64 OSXMMEXCPT : 1;
        UINT64 UMIP : 1;
        UINT64 LA57 : 1;
        UINT64 VMXE : 1;
        UINT64 SMXE : 1;
        UINT64 unuse1 : 1;
        UINT64 FSGSBASE : 1;
        UINT64 PCIDE : 1;
        UINT64 OSXSAVE : 1;
        UINT64 KL : 1;
        UINT64 SMEP : 1;
        UINT64 SMAP : 1;
        UINT64 PKE : 1;
        UINT64 CET : 1;
        UINT64 PKS : 1;
        UINT64 Ressrved : 63 - 24;
    }Fields;
}CR4, *PCR4;
static_assert(sizeof(CR4) == 8, "sizeof CR4");

///// <summary>
///// cr3结构体
///// </summary>
typedef union _CR3 {
    UINT64 value;
    struct
    {
        UINT64 ignore1 : 3;
        UINT64 PWT : 1;
        UINT64 PCD : 1;
        UINT64 ignore2 : 7;
        UINT64 PPN : 40;
        UINT64 Reserved1 : 12;
    }Fields;

}CR3, *PCR3;
static_assert(sizeof(CR3) == 8, "sizeof CR3");

///// <summary>
///// 各页表项结构体
///// </summary>
typedef union _PA {
    UINT64 vaule;
    LARGE_INTEGER AsLargeInteger;
    struct
    {
        UINT64 PPO : 12;
        UINT64 PPN : 40;
        UINT64 UnUse1 : 12;
    }Fileds4KB;
    struct
    {
        UINT64 PPO : 21;
        UINT64 PPN : 31;
        UINT64 UnUse1 : 12;
    }Fileds2MB;
    struct
    {
        UINT64 PPO : 30;
        UINT64 PPN : 22;
        UINT64 UnUse1 : 12;
    }Fileds1GB;
}PA, *P_PA;
static_assert(sizeof(PA) == 8, "sizeof PA");

///// <summary>
///// 虚拟地址结构体
///// </summary>
typedef union _VA {
    UINT64 vaule;
    LARGE_INTEGER AsLargeInteger;
    struct
    {
        UINT64 VPO : 12;
        UINT64 VPN4 : 9;
        UINT64 VPN3 : 9;
        UINT64 VPN2 : 9;
        UINT64 VPN1 : 9;
        UINT64 UnUse1 : 16;
    }Fileds4KB;
    struct
    {
        UINT64 VPO : 21;
        UINT64 VPN3 : 9;
        UINT64 VPN2 : 9;
        UINT64 VPN1 : 9;
        UINT64 UnUse1 : 16;
    }Fileds2MB;
    struct
    {
        UINT64 VPO : 30;
        UINT64 VPN2 : 9;
        UINT64 VPN1 : 9;
        UINT64 UnUse1 : 16;
    }Fileds1GB;

}VA, *P_VA;
static_assert(sizeof(VA) == 8, "sizeof VA");

typedef union _PML4E {
    UINT64 value;
    struct
    {
        UINT64 P : 1;
        UINT64 R_W : 1;
        UINT64 US : 1;
        UINT64 PWT : 1;
        UINT64 PCD : 1;
        UINT64 A : 1;
        UINT64 ign : 5;
        UINT64 R : 1;
        UINT64 PPN : 36;
        UINT64 ign2 : 15;
        UINT64 XD : 1;
    }Fields4K;
}PML4E, *PPML4E, L1PTE, *PL1PTE;
static_assert(sizeof(PML4E) == 8, "sizeof PML4E");

typedef union _PDPTE {
    UINT64 value;
    struct
    {
        UINT64 P : 1;
        UINT64 R_W : 1;
        UINT64 US : 1;
        UINT64 PWT : 1;
        UINT64 PCD : 1;
        UINT64 A : 1;
        UINT64 ign : 1;
        UINT64 PS : 1;
        UINT64 ign2 : 3;
        UINT64 R : 1;
        UINT64 PPN : 36;
        UINT64 ign3 : 15;
        UINT64 XD : 1;
    }Fields4K;
    struct
    {
        UINT64 P : 1;
        UINT64 R_W : 1;
        UINT64 US : 1;
        UINT64 PWT : 1;
        UINT64 PCD : 1;
        UINT64 A : 1;
        UINT64 ign : 1;
        UINT64 PS : 1;
        UINT64 ign2 : 3;
        UINT64 R : 1;
        UINT64 Reserved : 18;
        UINT64 PPN : 18;
        UINT64 ign3 : 15;
        UINT64 XD : 1;
    }Fields1G;
}PDPTE, *PPDPTE, L2PTE, *PL2PTE;
static_assert(sizeof(PDPTE) == 8, "sizeof PDPTE");

typedef union _PDE {
    UINT64 value;
    struct
    {
        UINT64 P : 1;
        UINT64 R_W : 1;
        UINT64 US : 1;
        UINT64 PWT : 1;
        UINT64 PCD : 1;
        UINT64 A : 1;
        UINT64 ign : 1;
        UINT64 PS : 1;
        UINT64 ign2 : 3;
        UINT64 R : 1;
        UINT64 PPN : 36;
        UINT64 ign3 : 15;
        UINT64 XD : 1;
    }Fields4K;
    struct
    {
        UINT64 P : 1;
        UINT64 R_W : 1;
        UINT64 US : 1;
        UINT64 PWT : 1;
        UINT64 PCD : 1;
        UINT64 A : 1;
        UINT64 ign : 1;
        UINT64 PS : 1;
        UINT64 ign2 : 3;
        UINT64 R : 1;
        UINT64 Reserved : 9;
        UINT64 PPN : 27;
        UINT64 ign3 : 15;
        UINT64 XD : 1;
    }Fields2MB;
}PDE, *PPDE, L3PTE, *PL3PTE;
static_assert(sizeof(PDE) == 8, "sizeof PDE");

typedef union _PTE {
    UINT64 value;
    struct
    {
        UINT64 P : 1;
        UINT64 R_W : 1;
        UINT64 US : 1;
        UINT64 PWT : 1;
        UINT64 PCD : 1;
        UINT64 A : 1;
        UINT64 ign : 1;
        UINT64 PS : 1;
        UINT64 ign2 : 3;
        UINT64 R : 1;
        UINT64 PPN : 36;
        UINT64 ign3 : 15;
        UINT64 XD : 1;
    }Fields4K;
}PTE, *PPTE, L4PTE, *PL4PTE;
static_assert(sizeof(PDE) == 8, "sizeof PTE");

#pragma pack()

//// 各个页表存储了512个页表项
#define MIX_PAGETABLEENTRY_SIZE 512

////--------------------内核导出函数声明：begin--------------------////
extern PVOID __stdcall MmGetVirtualForPhysical(LARGE_INTEGER AsLargeInteger);
////extern NTSTATUS PsLookupProcessByProcessId(HANDLE ProcessId, PEPROCESS * Process);

////--------------------内核导出函数声明: end--------------------////

////--------------------导出函数声明--------------------////

///// <summary>
///// 虚拟地址转换为物理地址
///// </summary>
///// <param name="cr3">目标进程的cr3</param>
///// <param name="pVa">虚拟地址结构指针</param>
///// <param name="outPa">出参：物理地址结构指针</param>
///// <returns></returns>
NTSTATUS Va2Pa(_In_ const PCR3 cr3, _In_ const P_VA pVa, _Out_ P_PA outPa);
```

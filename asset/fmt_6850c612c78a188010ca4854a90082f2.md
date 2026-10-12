
# 用 GDT 和 IDT 查找时钟中断处理函数的地址

> 原文：[`www.pediy.com/kssd/pediy10/102541.html`](https://www.pediy.com/kssd/pediy10/102541.html)

嘿嘿 初学 也不知道有错没有 有错望纠正

```c
#include <ntddk.h>
#include <string.h>

typedef struct _LDR_DATA_TABLE_ENTRY
{     
  LIST_ENTRY     InLoadOrderLinks;
  LIST_ENTRY       InMemoryOrderLinks;
  LIST_ENTRY       InInitializationOrderLinks;
  PVOID           DllBase;
  PVOID           EntryPoint;
  ULONG           SizeOfImage;
  UNICODE_STRING FullDllName;
  UNICODE_STRING BaseDllName;
  ULONG          Flags;
  USHORT         LoadCount;
  USHORT         TlsIndex;
  union
  {
    LIST_ENTRY   HashLinks;
    struct
    {
      PVOID       SectionPointer;
      ULONG      CheckSum;
    };
  };
  union
  {
    ULONG         TimeDateStamp;
    PVOID         LoadedImports;
  };
  PVOID           EntryPointActivationContext;
  PVOID           PatchInformation;
} LDR_DATA_TABLE_ENTRY, *PLDR_DATA_TABLE_ENTRY;

typedef unsigned char P2C_U8;
typedef unsigned short P2C_U16;
typedef unsigned long P2C_U32;
typedef short WORD;
typedef unsigned int DWORD;

#pragma pack(push,1)
typedef struct P2C_IDT_ENTRY_ {
  P2C_U16 offset_low;
  P2C_U16 selector;
  P2C_U8 reserved;
  P2C_U8 type:4;
  P2C_U8 always0:1;
  P2C_U8 dpl:2;
  P2C_U8 present:1;
  P2C_U16 offset_high;
} P2C_IDTENTRY, *PP2C_IDTENTRY;
#pragma pack(pop)

#pragma pack(push,1)
typedef struct P2C_GDT_ENTRY_ {
  P2C_U16 limit;
  P2C_U8 base1;
  P2C_U8 base2;
  P2C_U8 base3;
  P2C_U16 shuxing;
  P2C_U8 base4;
} P2C_GDTENTRY, *PP2C_GDTENTRY;
#pragma pack(pop)

#define P2C_MAKELONG(low, high) \
  ((P2C_U32)(((P2C_U16)((P2C_U32)(low) & 0xffff)) | ((P2C_U32)((P2C_U16)((P2C_U32)(high) & 0xffff))) << 16))
#define P2C_LOW16_OF_32(data) \
  ((P2C_U16)(((P2C_U32)data) & 0xffff))
#define P2C_HIGH16_OF_32(data) \
  ((P2C_U16)(((P2C_U32)data) >> 16))

#pragma pack(push,1)
typedef struct P2C_IDTR_ {
  P2C_U16 limit;    // 范围
  P2C_U32 base;    // 基地址（就是开始地址）
} P2C_IDTR, *PP2C_IDTR;
#pragma pack(pop)

void *p2cGetIdt()
{
  P2C_IDTR idtr;
  // 一句汇编读取到 IDT 的位置。
  _asm
  {
    sidt idtr
  }
  return (void *)idtr.base;
}

void *p2cGetGdt()
{
  P2C_IDTR gdtr;
  // 一句汇编读取到 IDT 的位置。
  _asm
  {
    sgdt gdtr
  }
  return (void *)gdtr.base;
}

NTSTATUS DriverEntry(IN OUT PDRIVER_OBJECT   DriverObject,IN PUNICODE_STRING      RegistryPath)
{
  WORD newcs,oldcs;
  PP2C_IDTENTRY idt_addr = (PP2C_IDTENTRY)p2cGetIdt();
  DWORD  offset = P2C_MAKELONG(idt_addr->offset_low, idt_addr->offset_high);
  newcs=idt_addr->selector;
  newcs=newcs/8;
  PP2C_GDTENTRY gdt_addr=(PP2C_GDTENTRY)p2cGetGdt();
  gdt_addr+=newcs;
  DWORD base=gdt_addr->base1+gdt_addr->base2*0x100+gdt_addr->base3*0x10000+gdt_addr->base4*0x1000000;
  DWORD realadr=base+offset;
  DbgPrint("Int 0 interrupt Function Address=%X",realadr);
  return STATUS_DEVICE_CONFIGURATION_ERROR;
}
```

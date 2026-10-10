

# 一个禁止任何 ring3 调试的驱动

> 原文：[`www.pediy.com/kssd/pediy12/138997.html`](https://www.pediy.com/kssd/pediy12/138997.html)

这个方法很简单：hook int1，int3  
然后清除 DebugPort，简单明了~~  
代码适用于多核，多系统~~~  

代码:

```
#include <ntddk.h>

#define MAKELONG(low, high) \\
((ULONG)(((USHORT)((ULONG)(low) & 0xffff)) | ((ULONG)((USHORT)((ULONG)(high) & 0xffff))) << 16))

#define GET\_LOW16\_OF\_32(data) \\
((USHORT)(((ULONG)data) & 0xffff))

#define GET\_HIGH16\_OF\_32(data) \\
((USHORT)(((ULONG)data) >> 16))

#pragma pack(push,1)
typedef struct IDTR\_
{
  USHORT limit;
  ULONG base;
} IDTR, \*P\_IDTR;

typedef struct IDTENTRY\_
{
  USHORT offset\_low;
  USHORT selector;
  UCHAR reserved;
  UCHAR type:4;
  UCHAR always0:1;
  UCHAR dpl:2;
  UCHAR present:1;
  USHORT offset\_high;
} IDTENTRY, \*P\_IDTENTRY;
#pragma pack(pop)

ULONG Offset = 0;
ULONG HOOK\_IDT\_INDEX\[\] = {0x01, 0x03};
#define HOOK\_IDT\_NUM sizeof(HOOK\_IDT\_INDEX)/sizeof(ULONG)
VOID \*g\_old\_entry\[HOOK\_IDT\_NUM\] = {0};

VOID \*GetIdt()
{
  IDTR idtr;
  \_asm sidt idtr
  return (VOID \*)idtr.base;
}

VOID MyUserFilter()
{
  KdPrint(("Crurrent IRQL: %d\\n",KeGetCurrentIrql()));
  if (Offset > 0)
  {
    ULONG eprocess = (ULONG)PsGetCurrentProcess();
    PULONG pDebugPort = (PULONG)(eprocess+Offset);
    if (\*pDebugPort > 0)
    {
      KdPrint(("DebugObject = %x\\n", pDebugPort));
      \*pDebugPort = 0; //clear DebugPort
    }
  }
}

\_\_declspec(naked) InterruptProc01()
{
  \_\_asm
  {
    pushfd          // 保存标志寄存器
    pushad          // 保存所有的通用寄存器
    push  fs

    \_\_asm
    {
      mov     ebx, 30H  // Set FS to PCR.
      mov     fs, bx
    }
    call MyUserFilter

    pop    fs
    popad          // 恢复通用寄存器
    popfd          // 恢复标志寄存器
    jmp  g\_old\_entry\[0\]    // 跳到原来的中断服务程序
  }
}

\_\_declspec(naked) InterruptProc03()
{
  \_\_asm
  {
    pushfd          // 保存标志寄存器
    pushad          // 保存所有的通用寄存器
    push  fs

    \_\_asm
    {
      mov     ebx, 30H  // Set FS to PCR.
      mov     fs, bx
    }
    call MyUserFilter

    pop    fs
    popad          // 恢复通用寄存器
    popfd          // 恢复标志寄存器
    jmp  g\_old\_entry\[4\]    // 跳到原来的中断服务程序
  }
}

VOID \*g\_new\_entry\[HOOK\_IDT\_NUM\] = {InterruptProc01, InterruptProc03};

VOID ModifyInterrupt(BOOLEAN hook\_or\_unhook)
{
  USHORT i;
  P\_IDTENTRY idt\_addr = (P\_IDTENTRY)GetIdt();

  //SetWriteProtect(FALSE, &orgcr0);
  for (i = 0; i < HOOK\_IDT\_NUM; i++)
  {
    KdPrint(("the current address for index %02x = %x\\n", HOOK\_IDT\_INDEX\[i\],
      (VOID \*)MAKELONG(idt\_addr\[HOOK\_IDT\_INDEX\[i\]\].offset\_low, idt\_addr\[HOOK\_IDT\_INDEX\[i\]\].offset\_high)));
    if(hook\_or\_unhook)
    {
      KdPrint(("try to hook this interrupt\\n"));
      g\_old\_entry\[i\] = (VOID \*)MAKELONG(idt\_addr\[HOOK\_IDT\_INDEX\[i\]\].offset\_low, idt\_addr\[HOOK\_IDT\_INDEX\[i\]\].offset\_high);
      idt\_addr\[HOOK\_IDT\_INDEX\[i\]\].offset\_low = GET\_LOW16\_OF\_32(g\_new\_entry\[i\]);
      idt\_addr\[HOOK\_IDT\_INDEX\[i\]\].offset\_high = GET\_HIGH16\_OF\_32(g\_new\_entry\[i\]);
    }
    else
    {
      KdPrint(("try to recovery interrupt for index %02x\\n", HOOK\_IDT\_INDEX\[i\]));
      idt\_addr\[HOOK\_IDT\_INDEX\[i\]\].offset\_low = GET\_LOW16\_OF\_32(g\_old\_entry\[i\]);
      idt\_addr\[HOOK\_IDT\_INDEX\[i\]\].offset\_high = GET\_HIGH16\_OF\_32(g\_old\_entry\[i\]);
    }
    KdPrint(("the current address = %x\\n",
      (VOID \*)MAKELONG(idt\_addr\[HOOK\_IDT\_INDEX\[i\]\].offset\_low, idt\_addr\[HOOK\_IDT\_INDEX\[i\]\].offset\_high)));
  }//for
  //SetWriteProtect(TRUE, &orgcr0);
}

VOID HookIDT(BOOLEAN hook\_or\_unhook)
{
  KAFFINITY     ActiveProcessors, CurrentAffinity;
  ActiveProcessors=KeQueryActiveProcessors(); 
  KdPrint(("KeActiveProcessors = %d\\n", ActiveProcessors));
  for (CurrentAffinity = 1; ActiveProcessors; CurrentAffinity <<= 1) 
  {
    if (ActiveProcessors & CurrentAffinity)
    {
      ActiveProcessors &= ~CurrentAffinity;
      KeSetSystemAffinityThread(CurrentAffinity);

      KdPrint(("At %d CPU IDT:\\n", CurrentAffinity));
      ModifyInterrupt(hook\_or\_unhook);
      KdPrint(("\\n"));
    }
  }
}

VOID Unload(PDRIVER\_OBJECT drv)
{
    HookIDT(FALSE);
    KdPrint (("Unload Driver\\n")); 
}

NTSTATUS DriverEntry( 
                     IN PDRIVER\_OBJECT DriverObject, 
                     IN PUNICODE\_STRING RegistryPath 
                     ) 
{ 
  RTL\_OSVERSIONINFOEXW osverinfo = {sizeof(osverinfo)};

    KdPrint (("Entering DriverEntry\\n")); 
    DriverObject->DriverUnload = Unload;

  RtlGetVersion((PRTL\_OSVERSIONINFOW)&osverinfo);
  KdPrint(("OSVersion NT %d.%d:%d sp%d.%d\\n",
    osverinfo.dwMajorVersion, osverinfo.dwMinorVersion, osverinfo.dwBuildNumber,
    osverinfo.wServicePackMajor, osverinfo.wServicePackMinor));
  if (osverinfo.dwMajorVersion == 5 && osverinfo.dwMinorVersion == 0)
    Offset = 0x120; //WINDOWS\_VERSION\_2K
  else if (osverinfo.dwMajorVersion == 5 && osverinfo.dwMinorVersion == 1)
    Offset = 0xbc; //WINDOWS\_VERSION\_XP
  else if (osverinfo.dwMajorVersion == 5 && osverinfo.dwMinorVersion == 2)
    Offset = 0xcc; //WINDOWS\_VERSION\_2003
  else if (osverinfo.dwMajorVersion == 6 && osverinfo.dwMinorVersion == 0)
    Offset = 0xd4; //WINDOWS\_VERSION\_VISTA
  else if (osverinfo.dwMajorVersion == 6 && osverinfo.dwMinorVersion == 1)
    Offset = 0xec; //WINDOWS\_VERSION\_WIN7
  else
    Offset = 0;

    HookIDT(TRUE);

    return  STATUS\_SUCCESS; 
}

```

谢谢大家！
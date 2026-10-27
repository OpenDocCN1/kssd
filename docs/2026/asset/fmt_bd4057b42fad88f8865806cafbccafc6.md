
```c
//0x8c bytes (sizeof)
struct _KTRAP_FRAME {
    ULONG DbgEbp; //0x0
    ULONG DbgEip; //0x4
    ULONG DbgArgMark; //0x8
    ULONG DbgArgPointer; //0xc
    USHORT TempSegCs; //0x10
    UCHAR Logging; //0x12
    UCHAR Reserved; //0x13
    ULONG TempEsp; //0x14
    ULONG Dr0; //0x18
    ULONG Dr1; //0x1c
    ULONG Dr2; //0x20
    ULONG Dr3; //0x24
    ULONG Dr6; //0x28
    ULONG Dr7; //0x2c
    ULONG SegGs; //0x30
    ULONG SegEs; //0x34
    ULONG SegDs; //0x38
    ULONG Edx; //0x3c
    ULONG Ecx; //0x40
    ULONG Eax; //0x44
    ULONG PreviousPreviousMode; //0x48
    struct _EXCEPTION_REGISTRATION_RECORD * ExceptionList; //0x4c
    ULONG SegFs; //0x50
    ULONG Edi; //0x54
    ULONG Esi; //0x58
    ULONG Ebx; //0x5c
    ULONG Ebp; //0x60
    ULONG ErrCode; //0x64
    ULONG Eip; //0x68
    ULONG SegCs; //0x6c
    ULONG EFlags; //0x70
    ULONG HardwareEsp; //0x74
    ULONG HardwareSegSs; //0x78
    ULONG V86Es; //0x7c
    ULONG V86Ds; //0x80
    ULONG V86Fs; //0x84
    ULONG V86Gs; //0x88
};
```

## KPCR

在3环时，`FS`指向`TEB`结构。在0环时，`FS`指向`KPCR`结构。该结构体用于描述对应的CPU的状态。一核对应一个`KPCR`。

> `windbg`中输入 `dd KeNumberProcessor` 查看核数

> `windbg`中输入 `dd KiProcessorBlock LXXX` 查看每个核的`KPCR`结构地址。

结构体成员：

```c
//0x3748 bytes (sizeof)
struct _KPCR {
    union {
        struct _NT_TIB NtTib; //0x0
        struct {
            struct _EXCEPTION_REGISTRATION_RECORD * Used_ExceptionList; //0x0
            VOID * Used_StackBase; //0x4
            VOID * Spare2; //0x8
            VOID * TssCopy; //0xc
            ULONG ContextSwitches; //0x10
            ULONG SetMemberCopy; //0x14
            VOID * Used_Self; //0x18
        };
    };
    struct _KPCR * SelfPcr; //0x1c
    struct _KPRCB * Prcb; //0x20
    UCHAR Irql; //0x24
    ULONG IRR; //0x28
    ULONG IrrActive; //0x2c
    ULONG IDR; //0x30
    VOID * KdVersionBlock; //0x34
    struct _KIDTENTRY * IDT; //0x38
    struct _KGDTENTRY * GDT; //0x3c
    struct _KTSS * TSS; //0x40
    USHORT MajorVersion; //0x44
    USHORT MinorVersion; //0x46
    ULONG SetMember; //0x48
    ULONG StallScaleFactor; //0x4c
    UCHAR SpareUnused; //0x50
    UCHAR Number; //0x51
    UCHAR Spare0; //0x52
    UCHAR SecondLevelCacheAssociativity; //0x53
    ULONG VdmAlert; //0x54
    ULONG KernelReserved[14]; //0x58
    ULONG SecondLevelCacheSize; //0x90
    ULONG HalReserved[16]; //0x94
    ULONG InterruptMode; //0xd4
    UCHAR Spare1; //0xd8
    ULONG KernelReserved2[17]; //0xdc
    struct _KPRCB PrcbData; //0x120
};
```

## KPRCB

结构体成员：

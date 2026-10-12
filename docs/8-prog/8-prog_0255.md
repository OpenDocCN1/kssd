
# 重现 SSDT-Shadow Hook 编译通过的代码，献给所有，有共享精神的人

> 原文：[`www.pediy.com/kssd/pediy10/67656.html`](https://www.pediy.com/kssd/pediy10/67656.html)

妈哟，我怎么又中机器狗了？第三次了..真晕（因为不喜欢杀软，习惯了裸奔，哪知这狗太疯了）。

好了，只要我的机器没被狗咬死，我还得用这破机器写代码！人在鸡鸡在。人亡鸡鸡埋！！！

言归正传，前些天因这某需要。想 Hook 一个 API，于是在 google.里一 G 了一通...

发现看雪里有我想要的东西。可是直接 A 进我的程序，又不行于是又摆渡了一番...终于似乎明白了点。

方法是卡巴大哥及金山等喜欢用的 Hook shadow ssdt，代码我编译通过的...但为了防止有心人做坏事却无心写程序的人 A 走做坏事..所以我去掉了点东西...研究过 ssdt 或内核的人自然明白..我只去掉了很少的一点点。

废话少说，看代码，并不像某某人说的那样一定要用一个 GUI 程序通过 `IoControlCode` 来触发..个人感觉那样多出来一个东西，反而不完美！[](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

```c
//#include "ntddk.h"

//驱动中的函数和全局变量默认是非分页的

#define HWND       ULONG
#define DWORD      ULONG
#define WORD       USHORT
#define BYTE       UCHAR
#define UINT       ULONG
#define ProcessNameOffset    0x174

//----------------------------------------------------------------------------------------------------------
#define NT_DEVICE_NAME L"\\Device\\hook" // device name and symbolic link name
#define DOS_DEVICE_NAME L"\\DosDevices\\hook"

#define ObjectNameInformation 1
#define BASE_PROCESS_PEB_OFFSET     0x01B0
#define BASE_PEB_PROCESS_PARAMETER_OFFSET 0x0010
#define BASE_PROCESS_PARAMETER_FULL_IMAGE_NAME 0x003C
//----------------------------------------------------------------------------------------------------------

#pragma pack(1)
typedef struct _SYSTEM_SERVICE_TABLE
{
    unsigned int *ServiceTableBase;
    unsigned int *ServiceCounterTableBase; //Used only checked build
    unsigned int NumberOfServices;
    unsigned char *ParamTableBase;
}
SYSTEM_SERVICE_TABLE,
*PSYSTEM_SERVICE_TABLE,
**PPSYSTEM_SERVICE_TABLE;
#pragma pack()

//-----------------------------------------------------------------------------------------------------------
#pragma pack(1)
typedef struct _SERVICE_DESCRIPTOR_TABLE_
{
    SYSTEM_SERVICE_TABLE ntoskrnl; // ntoskrnl.exe ( native api )
    SYSTEM_SERVICE_TABLE win32k;    // win32k.sys (gdi/user support)
    SYSTEM_SERVICE_TABLE Table3;    // not used
    SYSTEM_SERVICE_TABLE Table4;    // not used
}
SYSTEM_DESCRIPTOR_TABLE,
*PSYSTEM_DESCRIPTOR_TABLE,
**PPSYSTEM_DESCRIPTOR_TABLE;
#pragma pack()

//------------------------------------------------------------------------------------------------------------
#pragma pack(1)
typedef struct _SRVTABLE {
    PVOID     *ServiceTable;
    ULONG     LowCall;
    ULONG     HiCall;
    PVOID     *ArgTable;
} SRVTABLE, *PSRVTABLE;
#pragma pack()

//--------------------------------------------------------------------------------------------------
#pragma pack(1)
typedef struct _SYSTEM_HANDLE_INFORMATION_EX
{
    ULONG NumberOfHandles;
    SYSTEM_HANDLE_INFORMATION Information[1];
}SYSTEM_HANDLE_INFORMATION_EX, *PSYSTEM_HANDLE_INFORMATION_EX;
#pragma pack()

//-------------------------------------------------------------------------------------------------------------------
NTKERNELAPI BOOLEAN KeAddSystemServiceTable (PULONG_PTR Base, PULONG Count OPTIONAL, ULONG Limit, UCHAR Number, ULONG Index) ;
//------------------------------------------------------------------------------------------------------------

//////函数申明
VOID DrvUnload(PDRIVER_OBJECT);
NTSTATUS DriverEntry(PDRIVER_OBJECT, PUNICODE_STRING);
NTSTATUS IoDispatch(PDEVICE_OBJECT, PIRP);
HANDLE GetCsrPid();
PSYSTEM_DESCRIPTOR_TABLE FindShadowTable(void);

//------NtUserFindWindowEx------------------------------------------------------------------------------------------------------------------------------------------------------
ULONG NewNtUserFindWindowEx(HWND hwndParent, HWND hwndChild, PUNICODE_STRING pstrClassName OPTIONAL, PUNICODE_STRING pstrWindowName OPTIONAL, DWORD dwType);
typedef ULONG (*NTUSERFINDWINDOWEX)(HWND hwndParent, HWND hwndChild, PUNICODE_STRING pstrClassName OPTIONAL, PUNICODE_STRING pstrWindowName OPTIONAL, DWORD dwType);

//NTUSERFINDWINDOWEX OldNtUserFindWindowEx=(NTUSERFINDWINDOWEX *)ExAllocatePool(NonPagedPool, sizeof(NTUSERFINDWINDOWEX));
NTUSERFINDWINDOWEX OldNtUserFindWindowEx;

//--------------------------------------------------------------------------------------------------
//全局变量
PSYSTEM_DESCRIPTOR_TABLE ShadowTable = NULL;

//-----------------------------------------------------------------------------------------------------------
extern PSRVTABLE KeServiceDescriptorTable;
//__declspec(dllimport) KeAddSystemServiceTable(LPSSTAT,BOOL,DWORD,LPSSTPT,DWORD);

//-----------------------------------------------------------------------------------------------------------

NTSTATUS DriverEntry(PDRIVER_OBJECT pDriverObject, PUNICODE_STRING RegistryPath)
{
    KIRQL OldIrql;
    PDEVICE_OBJECT pDeviceObject = NULL;
    NTSTATUS ntStatus;
    UNICODE_STRING uniNtNameString, uniWin32NameString;
    int i;
    PEPROCESS EProcess;

    RtlInitUnicodeString( &uniNtNameString, NT_DEVICE_NAME );

    ntStatus = IoCreateDevice (
        pDriverObject,
        0, // DeviceExtensionSize
        &uniNtNameString,
        FILE_DEVICE_UNKNOWN, //
        0,       // No standard device characteristics
        FALSE,    // not exclusive device
        &pDeviceObject
        );

    if( !NT_SUCCESS(ntStatus) )
    {
        return ntStatus;
    }

    // create dispatch points for create/open, close, unload
    pDriverObject->DriverUnload = DrvUnload;

    RtlInitUnicodeString( &uniWin32NameString, DOS_DEVICE_NAME );

    ntStatus = IoCreateSymbolicLink( &uniWin32NameString, &uniNtNameString );

    if (!NT_SUCCESS(ntStatus))
    {
        IoDeleteDevice( pDriverObject->DeviceObject );
    }

    for( i = 0; i < IRP_MJ_MAXIMUM_FUNCTION; i++ )
        pDriverObject->MajorFunction[i] = IoDispatch;

    KeEnterCriticalRegion();
    ShadowTable = (PSYSTEM_DESCRIPTOR_TABLE)FindShadowTable();
    KeLeaveCriticalRegion();

    if(ShadowTable==0)
    {
        DbgPrint("没有找到 ShadowTable");
        return STATUS_UNSUCCESSFUL;
    }
    else
    {
        ntStatus = PsLookupProcessByProcessId(GetCsrPid(), &EProcess);
        if (!NT_SUCCESS( ntStatus ))
        {
            DbgPrint("PsLookupProcessByProcessId()\n");
            return ntStatus;
        }

        KeAttachProcess(EProcess);
        DbgPrint("ntoskrnl Address:\t 0x%08x\n", ShadowTable->ntoskrnl);
        DbgPrint("win32k   Address:\t 0x%08x\n", ShadowTable->win32k);
        DbgPrint("Table3   Address:\t 0x%08x\n", ShadowTable->Table3);
        DbgPrint("Table4   Address:\t 0x%08x\n\r\n", ShadowTable->Table4);
        DbgPrint("NtUserFindWindowEx Address: 0x%08x\n", ShadowTable->win32k.ServiceTableBase[0x17a]);

        //提升 IRQ 防止被打断
        if(KeGetCurrentIrql() <= DISPATCH_LEVEL)
            KeRaiseIrql(DISPATCH_LEVEL,&OldIrql);

        //禁用内存保护机制
        __asm
        {
            push eax
            mov eax,CR0
            and eax,not 0x10000
            mov CR0,eax
            pop eax
            cli
        }

        ///
        ///在这里做一些 hook
        ///
        OldNtUserFindWindowEx=(NTUSERFINDWINDOWEX)ShadowTable->win32k.ServiceTableBase[0x17a];
        (NTUSERFINDWINDOWEX)ShadowTable->win32k.ServiceTableBase[0x17a]=NewNtUserFindWindowEx;

        //启用内存保护机制
        __asm
        {
            sti
            push eax
            mov eax,CR0
            xor eax,0x10000
            mov CR0,eax
            pop eax
        }

        KeLowerIrql(OldIrql);
    }

    return STATUS_SUCCESS;
}

NTSTATUS IoDispatch(PDEVICE_OBJECT pDeviceObject, IN PIRP pIrp)
{
    NTSTATUS iStatus = STATUS_SUCCESS;
    pIrp->IoStatus.Status = iStatus;
    pIrp->IoStatus.Information = 0;
    IoCompleteRequest( pIrp, IO_NO_INCREMENT );
    return iStatus;
}

VOID DrvUnload(PDRIVER_OBJECT pDriverObject)
{
    // KIRQL OldIrql;
    PDEVICE_OBJECT pDeviceObject;
    UNICODE_STRING uniWin32NameString;

    pDeviceObject = pDriverObject->DeviceObject;
    // if(KeGetCurrentIrql() <= DISPATCH_LEVEL)
```



```
// KeRaiseIrql(DISPATCH_LEVEL, &OldIrql);

// 禁用内存保护机制

__asm
{
   push eax
    mov eax, CR0
    and eax, not 0x10000
    mov CR0, eax
    pop eax
    cli
}

///
/// 在这里做一些 hook
(NTUSERFINDWINDOWEX)ShadowTable->win32k.ServiceTableBase[0x17a]=OldNtUserFindWindowEx;

// 用互锁函数改变指针的值, 更安全?
// InterlockedExchangePointer(ShadowTable->win32k.ServiceTableBase[0x17a], &OldNtUserFindWindowEx);

// 启用内存保护机制

__asm
{
   sti
    push eax
    mov eax, CR0
    xor eax, 0x10000
    mov CR0, eax
    pop eax
}

// KeLowerIrql(OldIrql);
```

```
RtlInitUnicodeString( &uniWin32NameString, DOS_DEVICE_NAME );

IoDeleteSymbolicLink( &uniWin32NameString );

IoDeleteDevice( pDriverObject->DeviceObject );

}

//--------------------------------------------------------------------------------------------------

PSYSTEM_DESCRIPTOR_TABLE FindShadowTable(void)
{
unsigned char *       pCheckArea = (unsigned char * ) KeAddSystemServiceTable;

int         i;

PSRVTABLE   pSrvTable = NULL;

for (i=0; i<100; i++)
{
   __try
   {
    pSrvTable = *(PSRVTABLE*)pCheckArea;

    if (
     !MmIsAddressValid(pSrvTable)          ||
     (pSrvTable == KeServiceDescriptorTable) ||
     (memcmp(pSrvTable, KeServiceDescriptorTable, sizeof (*pSrvTable)) != 0)
     )
    {
     pCheckArea++;

     pSrvTable = NULL;
    }
   }
   __except(EXCEPTION_EXECUTE_HANDLER)
   {
    pSrvTable = NULL;
   }

   if (pSrvTable)
    break;
}

if (pSrvTable == NULL)
{
   pSrvTable = (PSRVTABLE)((char*)KeServiceDescriptorTable-0x230);
   if (MmIsAddressValid(pSrvTable))
   {
    __try
    {
     if (memcmp(pSrvTable, KeServiceDescriptorTable, sizeof (*pSrvTable)) != 0)
      pSrvTable = NULL;
    }
    __except(EXCEPTION_EXECUTE_HANDLER)
    {
     pSrvTable = NULL;
    }
   }
   else
   {
    pSrvTable = NULL;
   }
}

return (PSYSTEM_DESCRIPTOR_TABLE)pSrvTable;

}

PVOID GetInfoTable(ULONG ATableType)
{
ULONG mSize = 0x4000;

PVOID mPtr = NULL;

NTSTATUS St;

do
{
   mPtr = ExAllocatePool(PagedPool, mSize);
   memset(mPtr, 0, mSize);
   if (mPtr)
   {
    St = ZwQuerySystemInformation(ATableType, mPtr, mSize, NULL);
   } else return NULL;
   if (St == STATUS_INFO_LENGTH_MISMATCH)
   {
    ExFreePool(mPtr);
    mSize = mSize * 2;
   }
} while (St == STATUS_INFO_LENGTH_MISMATCH);

if (St == STATUS_SUCCESS) return mPtr;

ExFreePool(mPtr);

return NULL;

}

HANDLE GetCsrPid()
{
    HANDLE                        Process, hObject;
    HANDLE                        CsrId = (HANDLE)0;
    OBJECT_ATTRIBUTES             obj;
    CLIENT_ID                     cid;
    UCHAR                         Buff[0x100];
    POBJECT_NAME_INFORMATION      ObjName = (PVOID)&Buff;
    PSYSTEM_HANDLE_INFORMATION_EX Handles;
    ULONG                         r;
    Handles = GetInfoTable(SystemHandleInformation);
    if (!Handles) return CsrId;
    for (r = 0; r < Handles->NumberOfHandles; r++)
    {
        if (Handles->Information[r].ObjectTypeNumber == 21) // Port object
        {
            InitializeObjectAttributes(&obj, NULL, OBJ_KERNEL_HANDLE, NULL, NULL);
            cid.UniqueProcess = (HANDLE)Handles->Information[r].ProcessId;
            cid.UniqueThread = 0;
            if (NT_SUCCESS(NtOpenProcess(&Process, PROCESS_DUP_HANDLE, &obj, &cid)))
            {
                if (NT_SUCCESS(ZwDuplicateObject(Process,
          (HANDLE)Handles->Information[r].Handle,
          NtCurrentProcess(),
          &hObject,
          0, 0, DUPLICATE_SAME_ACCESS)))
                {
                    if (NT_SUCCESS(ZwQueryObject(hObject,
       ObjectNameInformation,
       ObjName,
       0x100, NULL)))
                    {
                        if (ObjName->Name.Buffer &&
                            !wcsncmp(L"\\Windows\\ApiPort", ObjName->Name.Buffer, 20))
                        {
                            CsrId = (HANDLE)Handles->Information[r].ProcessId;
                        }
                    }
                    ZwClose(hObject);
                }
                ZwClose(Process);
            }
        }
    }
    ExFreePool(Handles);
    return CsrId;
}

PCWSTR GetProcessFullName()
{
DWORD dwAddress;
if(KeGetCurrentIrql() != PASSIVE_LEVEL)
   return NULL;
dwAddress = (DWORD)PsGetCurrentProcess();
if(dwAddress == 0 || dwAddress == 0xFFFFFFFF)
   return NULL;
dwAddress += BASE_PROCESS_PEB_OFFSET;
if((dwAddress = *(DWORD*)dwAddress) == 0) return 0;
dwAddress += BASE_PEB_PROCESS_PARAMETER_OFFSET;
if((dwAddress = *(DWORD*)dwAddress) == 0) return 0;
dwAddress += BASE_PROCESS_PARAMETER_FULL_IMAGE_NAME;
if((dwAddress = *(DWORD*)dwAddress) == 0) return 0;
return (PCWSTR)dwAddress;
}

PVOID PsGetProcessSectionBaseAddress(PEPROCESS Process);
```

```
//---------------------------------------------------------------------------------------------------

ULONG NewNtUserFindWindowEx(HWND hwndParent, HWND hwndChild, PUNICODE_STRING pstrClassName OPTIONAL, PUNICODE_STRING pstrWindowName OPTIONAL, DWORD dwType)
{
KIRQL OldIrql;
ULONG result;
if(KeGetCurrentIrql() <= DISPATCH_LEVEL)
   KeRaiseIrql(DISPATCH_LEVEL, &OldIrql);
// UNICODE_STRING filtter;
// PUNICODE_STRING p=&filtter;
// RtlInitUnicodeString( &filtter, L"GAMECLIENT");
DbgPrint("窗口标题:\t%ws", pstrWindowName->Buffer);
DbgPrint("窗口类:\t%ws", pstrClassName->Buffer);
DbgPrint( "调用者路径\r\n%ws", GetProcessFullName());
DbgPrint( "---------------------------------------------------------");
result=OldNtUserFindWindowEx(hwndParent, hwndChild, pstrClassName, pstrWindowName, dwType);
return result;
}
```

`C:\mydrivers\ShadowSSDT_Hook\objchk\i386\ShadowSSDT.sys`

不知道为什么, 我每次 Hook 成功. 什么都 OK, 但是我想拆卸的时候会 `BSOD`...

出错代码是 `INVALIDPROCESSATTACH_ATTEMPT`

我的小笨脑想了很久, 最终还是没办法解决... 原因我猜大概是 `KeAttachProcess` 后没有 `KeDetachProcess`, 但是我试过没用, 反而是我一加载驱动就给我蓝了... 抑或是有别的程序在调用 `findwindow` 刚好在这个时候我去 `Unhook` 了???? 不明白. 还有 `rootkit.com` 上面的这段代码也会蓝, 我运行个批处理死循环打开并关闭一个空程序他就蓝了.. 用过 `KeDelayExecutionThread` 但貌似没什么用. 希望有高手帮我解惑 ---代码, 相见. 附加一句: 我喜欢代码, 不喜欢理论

参考:

`http://bbs.pediy.com/showthread.php?p=449864`

`http://www.whitecell.org` NT 下动态切换进程分析笔记

`www.rootkit.com/vault/fuzen_op/HideProcessHookMDL.zip`

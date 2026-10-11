
# 获取 `OB_TYPE` 的一个方法，各版本通用

> 原文：[`www.pediy.com/kssd/pediy12/143990.html`](https://www.pediy.com/kssd/pediy12/143990.html)

看了 `SYSTEM_HANDLE_INFORMATION` 里面的 `ObjectTypeNumber` 成员，一直在源码种找不到它的定义，网上找到了半天，然后对比又对不上号……

目前比较靠谱的就是 achillis 大年的

http://hi.baidu.com/\_achillis/blog/item/b175e2d254745ad8a8ec9a69.html

不过他说 `vista` 以后的可能不一样（我也没试验过），索性写了一份来专门打印出来的代码。

代码:

```
#define ObjectNameInformation  1

#define SystemHandleInformation 0x10

typedef struct _SYSTEM_HANDLE_INFORMATION {
  ULONG ProcessId;
  UCHAR ObjectTypeNumber;
  UCHAR Flags;
  USHORT Handle;
  PVOID Object;
  ACCESS_MASK GrantedAccess;
} SYSTEM_HANDLE_INFORMATION, *PSYSTEM_HANDLE_INFORMATION;

typedef struct _SYSTEM_HANDLE_INFORMATION_EX {
  ULONG NumberOfHandles;
  SYSTEM_HANDLE_INFORMATION Information[1];
} SYSTEM_HANDLE_INFORMATION_EX, *PSYSTEM_HANDLE_INFORMATION_EX;

NTSTATUS ZwQuerySystemInformation( 
  IN ULONG SystemInformationClass, 
  IN PVOID SystemInformation, 
  IN ULONG SystemInformationLength, 
  OUT PULONG ReturnLength);
/*
NTSTATUS ZwDuplicateObject(
  IN HANDLE        SourceProcessHandle,
  IN PHANDLE        SourceHandle,
  IN HANDLE        TargetProcessHandle,
  OUT PHANDLE        TargetHandle,
  IN ACCESS_MASK      DesiredAccess OPTIONAL,
  IN BOOLEAN        InheritHandle,
  IN ULONG        Options );

NTSTATUS ZwQueryObject(
  IN HANDLE                ObjectHandle,
  IN ULONG                 ObjectInformationClass,
  OUT PVOID                ObjectInformation,
  IN ULONG                 ObjectInformationLength,
  OUT PULONG               ReturnLength OPTIONAL);

NTSTATUS PsLookupProcessByProcessId(
  IN ULONG               ulProcId, 
  OUT PEPROCESS *        pEProcess);

NTSTATUS KeAttachProcess(PEPROCESS pPeb);
NTSTATUS KeDetachProcess();
*/

PVOID GetInfoTable(ULONG ATableType)
{
  ULONG mSize = 0x4000;
  PVOID mPtr = NULL;
  NTSTATUS St;

  do
  {
    mPtr = ExAllocatePoolWithTag(PagedPool, mSize, 'ACEK');
    memset(mPtr, 0, mSize);
    if (mPtr)
    {
      St = ZwQuerySystemInformation(ATableType, mPtr, mSize, NULL);
    } else return NULL;
    if (St == STATUS_INFO_LENGTH_MISMATCH)
    {
      ExFreePoolWithTag(mPtr, 'ACEK');
      mSize = mSize * 2;
    }
  } while (St == STATUS_INFO_LENGTH_MISMATCH);
  if (St == STATUS_SUCCESS) return mPtr;
  ExFreePool(mPtr);
  return NULL;
}

typedef struct _OBJECT_TYPE_INDEX {
  WCHAR  Format[0x40];
}OBJECT_TYPE_INDEX, *POBJECT_TYPE_INDEX; 

OBJECT_TYPE_INDEX TypeIndex[0x20] = { 0 };

void ObjectTypeFormat( ULONG Type, WCHAR * FormatName )
{
  if (Type >= 0x20) {
    
    return;
  }
  if (TypeIndex[Type].Format[0] == 0) {
    
    RtlStringCbPrintfW(
      TypeIndex[Type].Format,
      0x40,
      L"OB_TYPE_%s",
      FormatName
      );
  }
}

void ObjectTypeDbgPrint()
{
  ULONG i = 0;
  for (i = 0; i < 30; i++) {
    
    if (TypeIndex[i].Format[0] == 0) {
       
      DbgPrint("#define OB_TYPE_UNKNOWN  %d\n", i);
    } else {
      _wcsupr(TypeIndex[i].Format);
      DbgPrint("#define %S  %d\n", TypeIndex[i].Format, i);
    }
  }
}

HANDLE TestTable()
{
  HANDLE Process, hObject;
  HANDLE CsrId = (HANDLE)0;
  OBJECT_ATTRIBUTES obj;
  CLIENT_ID cid;
  UCHAR Buff[0x100];
  POBJECT_NAME_INFORMATION ObjName = (PVOID)&Buff;
  PSYSTEM_HANDLE_INFORMATION_EX Handles;
  ULONG r;
  
  ULONG  i;
  
  Handles = GetInfoTable(SystemHandleInformation);

  if (!Handles) return CsrId;

  for (r = 0; r < Handles->NumberOfHandles; r++)
  {
    {
      InitializeObjectAttributes(&obj, NULL, OBJ_KERNEL_HANDLE, NULL, NULL);

      cid.UniqueProcess = (HANDLE)Handles->Information[r].ProcessId;
      cid.UniqueThread = 0;

      if (NT_SUCCESS(NtOpenProcess(&Process, PROCESS_DUP_HANDLE, &obj, &cid)))
      {
        if (NT_SUCCESS(ZwDuplicateObject(
          Process, 
          (HANDLE)Handles->Information[r].Handle, 
          NtCurrentProcess(), &hObject, 0, 0, DUPLICATE_SAME_ACCESS)))
        {
          PPUBLIC_OBJECT_TYPE_INFORMATION uoti = (PPUBLIC_OBJECT_TYPE_INFORMATION)NULL;
          uoti = (PPUBLIC_OBJECT_TYPE_INFORMATION)ExAllocatePoolWithTag( 
            NonPagedPool, sizeof(PUBLIC_OBJECT_TYPE_INFORMATION) + 0x100, 'ACEK' );
          if (!uoti) {
            
            return 0;
          }
          RtlZeroMemory( uoti, sizeof(PUBLIC_OBJECT_TYPE_INFORMATION) + 0x100 );
          if (NT_SUCCESS(ZwQueryObject(hObject, ObjectTypeInformation, uoti, sizeof(PUBLIC_OBJECT_TYPE_INFORMATION) + 0x100, NULL))) {
            
            //ANSI_STRING AnsiFormat = { 0 };
            //KdPrint(("%wZ = %d\n", &uoti->TypeName, Handles->Information[r].ObjectTypeNumber));
            //if (NT_SUCCESS(ZwQueryObject(hObject, ObjectNameInformation, ObjName, 0x100, NULL)))
            //{
              //KdPrint(("TestTable: Number = %d, %wZ, ObjectName = %wZ\n", 
              //  Handles->Information[r].ObjectTypeNumber, &uoti->TypeName, &ObjName->Name));
            //}   
            //RtlUnicodeStringToAnsiString( &AnsiFormat, &uoti->TypeName, TRUE);
            ObjectTypeFormat( Handles->Information[r].ObjectTypeNumber, uoti->TypeName.Buffer );
            //RtlFreeAnsiString( &AnsiFormat );
          }
          ExFreePoolWithTag( uoti, 'ACEK' );
          ZwClose(hObject);
        }
        ZwClose(Process);
      }
    }
  }

  ObjectTypeDbgPrint();

  ExFreePool(Handles);
  return CsrId;
}
```

打印结果如下：

```
#define OB_TYPE_UNKNOWN  0  
#define OB_TYPE_UNKNOWN  1  
#define OB_TYPE_DIRECTORY  2  
#define OB_TYPE_SYMBOLICLINK  3  
#define OB_TYPE_TOKEN  4  
#define OB_TYPE_PROCESS  5  
#define OB_TYPE_THREAD  6  
#define OB_TYPE_JOB  7  
#define OB_TYPE_UNKNOWN  8  
#define OB_TYPE_EVENT  9  
#define OB_TYPE_UNKNOWN  10  
#define OB_TYPE_MUTANT  11  
#define OB_TYPE_UNKNOWN  12  
#define OB_TYPE_SEMAPHORE  13  
#define OB_TYPE_TIMER  14  
#define OB_TYPE_UNKNOWN  15  
#define OB_TYPE_KEYEDEVENT  16  
#define OB_TYPE_WINDOWSTATION  17  
#define OB_TYPE_DESKTOP  18  
#define OB_TYPE_SECTION  19  
#define OB_TYPE_KEY  20  
#define OB_TYPE_PORT  21  
#define OB_TYPE_WAITABLEPORT  22  
#define OB_TYPE_UNKNOWN  23  
#define OB_TYPE_UNKNOWN  24  
#define OB_TYPE_UNKNOWN  25  
#define OB_TYPE_UNKNOWN  26  
#define OB_TYPE_IOCOMPLETION  27  
#define OB_TYPE_FILE  28  
#define OB_TYPE_WMIGUID  29  
```

这个是我在 `xp sp3` 下打印的，如果需要其他系统的话，打印一次就行了。

当然有些是 unknown 的，是因为我系统没有相应的 handle 罢了，如果你系统有的话，打印出来也没有问题。

另外给出一份 achillis 大牛的：

```
typedef enum _SYSTEM_HANDLE_TYPE

{

   OB_TYPE_UNKNOWN=0,   //0  

   OB_TYPE_TYPE,    // 1,fixed  

   OB_TYPE_DIRECTORY,   // 2,fixed  

   OB_TYPE_SYMBOLIC_LINK, // 3,fixed  

   OB_TYPE_TOKEN,    // 4,fixed  

   OB_TYPE_PROCESS,   // 5,fixed  

   OB_TYPE_THREAD,    // 6,fixed  

   OB_TYPE_JOB,    // 7,fixed  

   OB_TYPE_DEBUG_OBJECT, // 8,fixed  

   OB_TYPE_EVENT,    // 9,fixed  

   OB_TYPE_EVENT_PAIR,   //10,fixed  

   OB_TYPE_MUTANT,    //11,fixed  

   OB_TYPE_CALLBACK,   //12,fixed  

   OB_TYPE_SEMAPHORE,   //13,fixed  

   OB_TYPE_TIMER,    //14,fixed  

   OB_TYPE_PROFILE,   //15,fixed  

   OB_TYPE_KEYED_EVENT, //16,fixed  

   OB_TYPE_WINDOWS_STATION,//17,fixed  

   OB_TYPE_DESKTOP,   //18,fixed  

   OB_TYPE_SECTION,   //19,fixed  

   OB_TYPE_KEY,    //20,fixed  

   OB_TYPE_PORT,    //21,fixed   

   OB_TYPE_WAITABLE_PORT, //22,fixed  

   OB_TYPE_ADAPTER,   //23,fixed  

   OB_TYPE_CONTROLLER,   //24,fixed  

   OB_TYPE_DEVICE,    //25,fixed  

   OB_TYPE_DRIVER,    //26,fixed  

   OB_TYPE_IOCOMPLETION, //27,fixed  

   OB_TYPE_FILE,    //28,fixed  

   OB_TYPE_WMIGUID    //29,fixed  

}SYSTEM_HANDLE_TYPE;
```

可以对比一下

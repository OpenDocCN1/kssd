
# 进程中 dll 模块的隐藏

> 原文：[`www.pediy.com/kssd/pediy10/59932.html`](https://www.pediy.com/kssd/pediy10/59932.html)

### 进程中 dll 模块的隐藏

`cc682/NetRoc`

`http://netroc682.spaces.live.com/`

为了避免自己的某个 dll 模块被别人检测出来，有时候希望在自己加载一个 dll 之后，或者将 dll 注入到他人进程之后避免被检查出来。这就需要想办法抹掉这个 dll 的模块信息，使得 `Toolhelp`、`psapi` 等枚举模块的 API 无法枚举它。

我们可以先简单看看 Windows 枚举进程内模块的办法吧：

首先是 `BOOL EnumProcessModules( HANDLE hProcess, HMODULE* lphModule, DWORD cb, LPDWORD lpcbNeeded);`

`EnumProcessModules` 实际调用 `EnumProcessModulesInternal` 进行枚举。下面是 vista 下 `psapi` 的代码片断：

```
.text:514024B8                   push    ebx
.text:514024B9                   push    18h
.text:514024BB                   lea     eax, [ebp+stProcessBasicInfo]
.text:514024BE                   push    eax
.text:514024BF                   push    ebx  ;ebx=0
.text:514024C0                   push    [ebp+hProcess]
.text:514024C3                   call    ds:__imp__NtQueryInformationProcess@20 ; NtQueryInformationProcess(x,x,x,x,x)
.text:514024C9                   cmp     eax, ebx
.text:514024CB                   jge     short loc_514024E0
```

调用 `NtQueryInformationProcess` 获得 `ProcessBasicInformation`，在 `PROCESS_BASIC_INFORMATION` 结构中取得 `PEB` 地址。然后读取指定进程 `PEB` 中的数据

```
text:514024E0 loc_514024E0:                       ; CODE XREF: EnumProcessModulesInternal(x,x,x,x,x)+24 j
.text:514024E0                   mov     eax, [ebp+stProcessBasicInfo.PebBaseAddress]
.text:514024E3                   cmp     eax, ebx
.text:514024E5                   jnz     short loc_514024EE
.text:514024E7                   push    8000000Dh
.text:514024EC                   jmp     short loc_514024CE
.text:514024EE ; ---------------------------------------------------------------------------

.text:514024EE

.text:514024EE loc_514024EE:                       ; CODE XREF: EnumProcessModulesInternal(x,x,x,x,x)+3E j
.text:514024EE                   push    ebx                 ; lpNumberOfBytesRead
.text:514024EF                   push    4                   ; nSize
.text:514024F1                   lea     ecx, [ebp+Ldr]
.text:514024F4                   push    ecx                 ; lpBuffer
.text:514024F5                   add     eax, 0Ch
.text:514024F8                   push    eax                 ; lpBaseAddress
.text:514024F9                   push    [ebp+hProcess]  ; hProcess
.text:514024FC                   mov     edi, ds:__imp__ReadProcessMemory@20 ; ReadProcessMemory(x,x,x,x,x)
.text:51402502                   call    edi ; ReadProcessMemory(x,x,x,x,x) ; ReadProcessMemory(x,x,x,x,x)
```

这里读取的是 `PEB` 地址 `+0C` 处的四个字节。

通过 `WinDbg` 我们可以看看 `nt!_PEB` 的结构

```
0: kd> dt nt!_PEB
  +0x000 InheritedAddressSpace : UChar
  +0x001 ReadImageFileExecOptions : UChar
  +0x002 BeingDebugged    : UChar
  +0x003 SpareBool        : UChar
  +0x004 Mutant           : Ptr32 Void
  +0x008 ImageBaseAddress : Ptr32 Void
  +0x00c Ldr              : Ptr32 _PEB_LDR_DATA
  +0x010 ProcessParameters : Ptr32 _RTL_USER_PROCESS_PARAMETERS
……
```

`+0C` 处是一个 `_PEB_LDR_DATA` 结构指针，里面包含了和 `LDR` 相关的一些数据，进程的模块链表就保存在 `Ldr` 中。下面是 `_PEB_LDR_DATA` 的结构：

```
0: kd> dt nt!_PEB_LDR_DATA
  +0x000 Length           : Uint4B
  +0x004 Initialized      : UChar
  +0x008 SsHandle         : Ptr32 Void
  +0x00c InLoadOrderModuleList : _LIST_ENTRY
  +0x014 InMemoryOrderModuleList : _LIST_ENTRY
  +0x01c InInitializationOrderModuleList : _LIST_ENTRY
  +0x024 EntryInProgress  : Ptr32 Void
```

其中，`InLoadOrderModuleList`、`InMemoryOrderModuleList`、`InInitializationOrderModuleList` 就是进程当前已加载模块的链表，只是按照不同的方式排序。`EnumProcessModules` 是通过 `InMemoryOrderModuleList` 链表枚举的，而根据 `Win2k` 代码，`ToolHelp32` 函数是通过 `InLoadOrderModuleList` 枚举。这三个 `_LIST_ENTRY` 都是在一个 `RTL_PROCESS_MODULE_INFORMATION` 结构中的成员。这个结构在 2k 代码中有引用，不过没有确切的定义，下面是 `ReactOS` 中的定义，不过看起来我的 vista `PSAPI` 中使用的结构已经有所变化了，这里只作参考。

```
//
// Loader Data Table Entry
//
typedef struct _LDR_DATA_TABLE_ENTRY
{
    LIST_ENTRY InLoadOrderLinks;
    LIST_ENTRY InMemoryOrderModuleList;
    LIST_ENTRY InInitializationOrderModuleList;
    PVOID DllBase;
    PVOID EntryPoint;
    ULONG SizeOfImage;
    UNICODE_STRING FullDllName;
    UNICODE_STRING BaseDllName;
    ULONG Flags;
    USHORT LoadCount;
    USHORT TlsIndex;
    union
    {
        LIST_ENTRY HashLinks;
        PVOID SectionPointer;
    };
    ULONG CheckSum;
    union
    {
        ULONG TimeDateStamp;
        PVOID LoadedImports;
    };
    PVOID EntryPointActivationContext;
    PVOID PatchInformation;
} LDR_DATA_TABLE_ENTRY, *PLDR_DATA_TABLE_ENTRY;
```

到这里，隐藏模块的方法就已经明了了：通过 `PEB` 取得 `Ldr` 数据，拿到三个模块链表，并将要隐藏的模块断链即可。下面是主要代码实现：

```
BOOL HideMyself()
{
  HMODULE hMod = GetModuleHandle( _T( "ntdll.dll"));
  HMODULE hModMyself = GetModuleHandle( _T("dll.dll"));
  pfnNtQueryInformationProcess p = (pfnNtQueryInformationProcess)::GetProcAddress( hMod, "NtQueryInformationProcess");
  PROCESS_BASIC_INFORMATION stInfo = {0};
  DWORD dwRetnLen = 0;
  DWORD dw = p( GetCurrentProcess(), 0, &stInfo, sizeof(stInfo), &dwRetnLen);
  PPEB pPeb = stInfo.PebBaseAddress;
  PLIST_ENTRY ListHead, Current;
  PLDR_DATA_TABLE_ENTRY pstEntry = NULL;
  ListHead = &( stInfo.PebBaseAddress->Ldr->InLoadOrderModuleList);
  Current = ListHead->Flink;
  while ( Current != ListHead)
  {
    pstEntry = CONTAINING_RECORD( Current, LDR_DATA_TABLE_ENTRY, InLoadOrderLinks);
    //DebugOutW( L"Module:%s, base:0x%X\r\n", pstEntry->FullDllName.Buffer, pstEntry->EntryPoint);
    if ( pstEntry->DllBase == hModMyself)
    {
      pstEntry->InLoadOrderLinks.Flink->Blink = pstEntry->InLoadOrderLinks.Blink;
      pstEntry->InLoadOrderLinks.Blink->Flink = pstEntry->InLoadOrderLinks.Flink;
      DebugOut( _T( "Hide injected dll."));
      break;
    }
    Current = pstEntry->InLoadOrderLinks.Flink;
  }
  ListHead = &( stInfo.PebBaseAddress->Ldr->InMemoryOrderModuleList);
  Current = ListHead->Flink;
  while ( Current != ListHead)
  {
    pstEntry = CONTAINING_RECORD( Current, LDR_DATA_TABLE_ENTRY, InMemoryOrderModuleList);
    DebugOutW( L"Module:%s, base:0x%X\r\n", pstEntry->FullDllName.Buffer, pstEntry->EntryPoint);
    if ( pstEntry->DllBase == hModMyself)
    {
      pstEntry->InMemoryOrderModuleList.Flink->Blink = pstEntry->InMemoryOrderModuleList.Blink;
      pstEntry->InMemoryOrderModuleList.Blink->Flink = pstEntry->InMemoryOrderModuleList.Flink;
      DebugOut( _T( "Hide injected dll."));
      break;
    }
    Current = pstEntry->InMemoryOrderModuleList.Flink;
  }
  DebugOutW( L"\r\n");
  ListHead = &( stInfo.PebBaseAddress->Ldr->InInitializationOrderModuleList);
  Current = ListHead->Flink;
  while ( Current != ListHead)
  {
    pstEntry = CONTAINING_RECORD( Current, LDR_DATA_TABLE_ENTRY, InInitializationOrderModuleList);
    DebugOutW( L"Module:%s, base:0x%X\r\n", pstEntry->FullDllName.Buffer, pstEntry->EntryPoint);
    if ( pstEntry->DllBase == hModMyself)
    {
      pstEntry->InInitializationOrderModuleList.Flink->Blink = pstEntry->InInitializationOrderModuleList.Blink;
      pstEntry->InInitializationOrderModuleList.Blink->Flink = pstEntry->InInitializationOrderModuleList.Flink;
      DebugOut( _T( "Hide injected dll."));
      break;
    }
    Current = pstEntry->InInitializationOrderModuleList.Flink;
  }
  //DebugOut( _T("Out HideMyself\r\n"));
  return TRUE;
}
```

这样处理之后，通过常规的枚举进程方式已经枚举不到隐藏模块，`ProcessExplorer` 也无法枚举。但是，通过枚举进程内存空间等非常规方法，仍然是可以找到的。关于 `PSAPI` 和 `Toolhelp` 函数枚举模块的原理，可以逆向 Windows 代码，或者查找网上的代码看看就明白了。

```
HideModuleFromPEB proc hInstDLL:DWORD
    assume  fs:nothing
    mov     esi,hInstDLL
    xor     eax,eax
    mov     eax,fs:[eax].TEB.Peb
    mov     eax,[eax].PEB.Ldr
```



### 汇编代码片段

```
lea     eax,[eax].PEB_LDR_DATA.InLoadOrderModuleList  

@@:  

mov     eax,[eax].LDR_MODULE.InLoadOrderModuleList.Flink  

cmp     esi,[eax].LDR_MODULE.BaseAddress  

jnz     @B  

mov     esi,[eax].LIST_ENTRY.Flink  

mov     ebx,[eax].LIST_ENTRY.Blink  

mov     [ebx].LIST_ENTRY.Flink,esi  

mov     esi,[eax].LIST_ENTRY.Blink  

mov     ebx,[eax].LIST_ENTRY.Flink  

mov     [ebx].LIST_ENTRY.Blink,esi  

lea     eax,[eax].LDR_MODULE.InMemoryOrderModuleList  

mov     esi,[eax].LIST_ENTRY.Flink  

mov     ebx,[eax].LIST_ENTRY.Blink  

mov     [ebx].LIST_ENTRY.Flink,esi  

mov     esi,[eax].LIST_ENTRY.Blink  

mov     ebx,[eax].LIST_ENTRY.Flink  

mov     [ebx].LIST_ENTRY.Blink,esi  

ret  

HideModuleFromPEB endp 
```

### C++ 代码片段：占内存与隐藏模块

我也来段占内存用的，其中 `testdll` 是隐式连接。

```
void *PEB = NULL;  

void *Ldr = NULL;  

_LIST_ENTRY *Flink = NULL;  

_LIST_ENTRY *p = NULL;  

BYTE  *BaseAddress = NULL;  

BYTE  *FullDllName = NULL;  

__asm  

{  

  mov eax,fs:[0x30]  

  mov PEB,eax  

}  

Ldr = * ( ( void **)( ( unsigned char * )PEB+0x0c ) );  

Flink = (_LIST_ENTRY*)* ( ( void **)( ( unsigned char * )Ldr+ 0x0c ) );  

p = Flink;  

do  

{  

  BaseAddress = * ( ( BYTE **)( ( unsigned char * )p+ 0x18 ) );  

  FullDllName = * ( ( BYTE **)( ( unsigned char * )p+ 0x28 ) );  

  LPSTR strFullDllName;  

  UnicodeToAnsi((LPCOLESTR)FullDllName,&strFullDllName);  

  if( strFullDllName )  

  {  

    if( strstr(strFullDllName,"testdll") )  

    {  

      *(LPDWORD)((LPBYTE)p + 0x38) = 1;  

    }  

  }  

  CO_SAFE_DELETE(strFullDllName);  

  p = p->Flink;  

}  

while ( Flink != p );  

FreeLibrary(GetModuleHandle("testdll.dll"));  

LPVOID lpdata = VirtualAlloc((LPVOID)NULL,1024*1024*6,MEM_COMMIT,PAGE_EXECUTE_READWRITE);  

if( lpdata != (LPVOID)0x400000 )  

{  

  return FALSE;  

}
```

### C++ 代码片段：循环遍历模块

都发代码片段，我也来一个：

```
while(modulo->BaseAddress != 0)  

{  

   if( (ULONG_PTR)modulo->BaseAddress == DllHandle)  

  {  

     if(modulo->InInitializationOrderModuleList.Blink == NULL) return FALSE;  

     prec = (LDR_MODULE*)(ULONG_PTR)((ULONG_PTR)modulo->InInitializationOrderModuleList.Blink - 16);  

     next = (LDR_MODULE*)(ULONG_PTR)((ULONG_PTR)modulo->InInitializationOrderModuleList.Flink - 16);  

     prec->InInitializationOrderModuleList.Flink = modulo->InInitializationOrderModuleList.Flink;  

     next->InInitializationOrderModuleList.Blink = modulo->InInitializationOrderModuleList.Blink;   

     prec = (LDR_MODULE*)modulo->InLoadOrderModuleList.Blink;  

     next = (LDR_MODULE*)modulo->InLoadOrderModuleList.Flink;  

     prec->InLoadOrderModuleList.Flink = modulo->InLoadOrderModuleList.Flink;  

     prec->InMemoryOrderModuleList.Flink = modulo->InMemoryOrderModuleList.Flink;  

     next->InLoadOrderModuleList.Blink = modulo->InLoadOrderModuleList.Blink;  

     next->InMemoryOrderModuleList.Blink = modulo->InMemoryOrderModuleList.Blink;  

     return TRUE;  

  }  

     modulo = (LDR_MODULE*)modulo->InLoadOrderModuleList.Flink;  

}
```

### 参考链接

给你个可编译的：`http://hi.baidu.com/zoo_/blog/item/4b695c8737e7862fc75cc33c.html`

### 结构体定义

在前面加声明：

```
typedef struct _UNICODE_STRING  

{  

  USHORT Length;  

  USHORT MaximumLength;  

  PWSTR  Buffer;  

} UNICODE_STRING,*PUNICODE_STRING;  

typedef struct _PEB_LDR_DATA {  

  ULONG                 Length;  

  BOOLEAN               Initialized;  

  PVOID                 SsHandle;  

  LIST_ENTRY             InLoadOrderModuleList;  

  LIST_ENTRY             InMemoryOrderModuleList;  

  LIST_ENTRY             InInitializationOrderModuleList;  

} PEB_LDR_DATA, *PPEB_LDR_DATA;  

typedef struct _LDR_MODULE  

{  

  LIST_ENTRY             InLoadOrderModuleList;   //+0x00  

  LIST_ENTRY             InMemoryOrderModuleList; //+0x08   

  LIST_ENTRY             InInitializationOrderModuleList; //+0x10  

  void*                  BaseAddress;  //+0x18  

  void*                  EntryPoint;   //+0x1c  

  ULONG                  SizeOfImage;  

  UNICODE_STRING         FullDllName;  

  UNICODE_STRING         BaseDllName;  

  ULONG                  Flags;  

  SHORT                  LoadCount;  

  SHORT                  TlsIndex;  

  HANDLE                 SectionHandle;  

  ULONG                  CheckSum;  

  ULONG                  TimeDateStamp;  

} LDR_MODULE, *PLDR_MODULE;
```

引用：*最初由 **ccccjf** 发布* ![查看帖子](img/78d57e1ac1f1fc075f84e78a6a789d46.jpg)

### 编译与实现

我按你说的，在 `Microsoft Visual C++ 6.0` 中编译，得到如下结果。

Configuration: `Cpp1 - Win32 Debug---...`

这是按你说的，弄出的 `.cpp` 文件：

```
typedef struct _UNICODE_STRING  

{  

  USHORT Length;  

  USHORT MaximumLength;  

  PWSTR  Buffer;  

} UNICODE_STRING,*PUNICODE_STRING;  

typedef struct _PEB_LDR_DATA {  

  ULONG                 Length;  

  BOOLEAN               Initialized;  

  PVOID                 SsHandle;  

  LIST_ENTRY             InLoadOrderModuleList;  

  LIST_ENTRY             InMemoryOrderModuleList;  

  LIST_ENTRY             InInitializationOrderModuleList;  

} PEB_LDR_DATA, *PPEB_LDR_DATA;  

typedef struct _LDR_MODULE  

{  

  LIST_ENTRY             InLoadOrderModuleList;   //+0x00  

  LIST_ENTRY             InMemoryOrderModuleList; //+0x08   

  LIST_ENTRY             InInitializationOrderModuleList; //+0x10  

  void*                  BaseAddress;  //+0x18  

  void*                  EntryPoint;   //+0x1c  

  ULONG                  SizeOfImage;  

  UNICODE_STRING         FullDllName;  

  UNICODE_STRING         BaseDllName;  

  ULONG                  Flags;  

  SHORT                  LoadCount;  

  SHORT                  TlsIndex;  

  HANDLE                 SectionHandle;  

  ULONG                  CheckSum;  

  ULONG                  TimeDateStamp;  

} LDR_MODULE, *PLDR_MODULE;  

void HideDll()  

{  

    HMODULE hMod = ::GetModuleHandle("mydll.dll");   

    PLIST_ENTRY Head,Cur;  

    PPEB_LDR_DATA ldr;  

    PLDR_MODULE ldm;  

    __asm  

    {  

        mov eax , fs:[0x30]  

        mov ecx , [eax + 0x0c] //Ldr  

        mov ldr , ecx  

    }  

    Head = &(ldr->InLoadOrderModuleList);  

    Cur = Head->Flink;  

    do  

    {  

        ldm = CONTAINING_RECORD( Cur, LDR_MODULE, InLoadOrderModuleList);  

        //printf("EntryPoint [0x%X]\n",ldm->BaseAddress);  

        if( hMod == ldm->BaseAddress)  

        {  

            ldm->InLoadOrderModuleList.Blink->Flink =  

                ldm->InLoadOrderModuleList.Flink;  

            ldm->InLoadOrderModuleList.Flink->Blink =  

                ldm->InLoadOrderModuleList.Blink;   

            ldm->InInitializationOrderModuleList.Blink->Flink =  

                ldm->InInitializationOrderModuleList.Flink;  

            ldm->InInitializationOrderModuleList.Flink->Blink =  

                ldm->InInitializationOrderModuleList.Blink;   

            ldm->InMemoryOrderModuleList.Blink->Flink =  

                ldm->InMemoryOrderModuleList.Flink;  

            ldm->InMemoryOrderModuleList.Flink->Blink =  

                ldm->InMemoryOrderModuleList.Blink;   

            break;  

        }  

        Cur= Cur->Flink;   

    }while(Head != Cur);  

}
```

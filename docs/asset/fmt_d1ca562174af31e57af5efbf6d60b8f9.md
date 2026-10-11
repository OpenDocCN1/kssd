
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

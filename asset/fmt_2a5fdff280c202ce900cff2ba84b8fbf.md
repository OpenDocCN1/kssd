
```c
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt43");
break;
case 44:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt44");
break;
case 45:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt45");
break;
case 46:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt46");
break;
case 47:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt47");
break;
case 48:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt48");
break;
case 49:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt49");
break;
case 50:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt50");
break;
case 51:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt51");
break;
case 52:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt52");
break;
case 53:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt53");
break;
case 54:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt54");
break;
case 55:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt55");
break;
case 56:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt56");
break;
case 57:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt57");
break;
case 58:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt58");
break;
case 59:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt59");
break;
case 60:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt60");
break;
case 61:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt61");
break;
case 62:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt62");
break;
case 63:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt63");
break;
case 64:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt64");
break;
case 65:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt65");
break;
case 66:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt66");
break;
case 67:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt67");
break;
case 68:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt68");
break;
case 69:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt69");
break;
case 70:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt70");
break;
case 71:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt71");
break;
case 72:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt72");
break;
case 73:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt73");
break;
case 74:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt74");
break;
case 75:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt75");
break;
case 76:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt76");
break;
case 77:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt77");
break;
case 78:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt78");
break;
case 79:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt79");
break;
case 80:
RtlInitUnicodeString(&un_game_mutex,L"WvsClientMt80");
break;
default:
i=0;
break;
}
RtlCopyUnicodeString(ObjectAttributes->ObjectName,&un_game_mutex);
return ((NTSTATUS(NTAPI*)(PHANDLE,ACCESS_MASK,POBJECT_ATTRIBUTES,BOOLEAN))nt_CreateMutant)(MutantHandle,DesiredAccess,ObjectAttributes,InitialOwner);
}
}
return ((NTSTATUS(NTAPI*)(PHANDLE,ACCESS_MASK,POBJECT_ATTRIBUTES,BOOLEAN))nt_CreateMutant)(MutantHandle,DesiredAccess,ObjectAttributes,InitialOwner);
}
```

```cpp
#pragma INITCODE
VOID hook_NtCreateMutant()
{
  ULONG* un_NtCreateMutant;
  un_NtCreateMutant=Getssdt_this(0x2B);
  u_NtCreateMutant=Getssdt_addr(0x2B);
  KdPrint(("当前 SSDT 表中 NtCreateMutant 的地址为:%x",u_NtCreateMutant));
  nt_CreateMutant=(Nt_CreateMutant*)u_NtCreateMutant;
  PAGED_Open();
  *un_NtCreateMutant=(ULONG)My_NtCreateMutant;
  PAGED_Exit();
}
```

```cpp
#pragma PAGEDCODE
VOID UNhook_NtCreateMutant()
{
  ULONG ntcreatemutant;
  ntcreatemutant=(ULONG)KeServiceDescriptorTable->ServiceTableBase+0x2B*4;
  PAGED_Open();
  *((ULONG*)ntcreatemutant)=(ULONG)u_NtCreateMutant;
  PAGED_Exit();
}
```

```cpp
extern "C"
typedef
NTSYSAPI
NTSTATUS
(__stdcall *Nt_OpenMutant)(
       OUT PHANDLE MutantHandle,
       IN ACCESS_MASK DesiredAccess,
       IN POBJECT_ATTRIBUTES ObjectAttributes
       );
Nt_OpenMutant* nt_openmutant;
```

```cpp
#pragma PAGEDCODE
extern "C"
NTSTATUS
__stdcall
My_NtOpenMutant(
        OUT PHANDLE MutantHandle,
        IN ACCESS_MASK DesiredAccess,
        IN POBJECT_ATTRIBUTES ObjectAttributes
       )
{
  if(ObjectAttributes!=NULL && ObjectAttributes->ObjectName!=NULL && ObjectAttributes->ObjectName->Buffer!=NULL)
  {
    KdPrint(("互斥体为为:%wZ\r\n",ObjectAttributes->ObjectName));
    UNICODE_STRING Mutant_name_Open;
    RtlInitUnicodeString(&Mutant_name_Open,L"DBWinMutex");
    if(ObjectAttributes && RtlEqualUnicodeString(&Mutant_name_Open,ObjectAttributes->ObjectName,FALSE))
    {
      KdPrint(("hook OpenMutant 成功!\r\n"));
      return STATUS_SUCCESS;
    }
  }
  return ((NTSTATUS(NTAPI*)(PHANDLE,ACCESS_MASK,POBJECT_ATTRIBUTES))nt_openmutant)(MutantHandle,DesiredAccess,ObjectAttributes);
}
```

```cpp
#pragma INITCODE
VOID hook_NtOpenMutant()
{
  ULONG* un_NtOpenMutant;
  un_NtOpenMutant=Getssdt_this(0x78);
  u_NtOpenMutant=Getssdt_addr(0x78);
  KdPrint(("当前系统 NtOpenMutant 的地址为:%x\n",u_NtOpenMutant));
  nt_openmutant=(Nt_OpenMutant*)u_NtOpenMutant;
  PAGED_Open();
  *un_NtOpenMutant=(ULONG)My_NtOpenMutant;
  PAGED_Exit();
}
```

```cpp
#pragma PAGEDCODE
VOID Unhook_NtOpenMutant()
{
  ULONG ntopenmutant;
  ntopenmutant=(ULONG)KeServiceDescriptorTable->ServiceTableBase+0x78*4;
  PAGED_Open();
  *((ULONG*)ntopenmutant)=(ULONG)u_NtOpenMutant;
  PAGED_Exit();
}
```

```cpp
#pragma PAGEDCODE
VOID GetCall_addr()
{
  addr_NtOpenThread=Getssdt_addr(in_NtOpenThread);
  addr_NtOpenThread+=0xA;
  __asm
  {
    push eax
    push ebx
    mov eax,addr_NtOpenThread
    mov ebx,[eax+1]
    add eax,ebx
    add eax,5
    mov call_addr_NtOpenProecss,eax
    pop ebx
    pop eax
  }
  KdPrint(("NtOpenProess 的 call 地址为:%x\n",call_addr_NtOpenProecss));
  call_addr_NtReadVirtualMemory=call_addr_NtOpenProecss;
  KdPrint(("NtReadVirtualMemory 的 call 地址为:%x\n",call_addr_NtReadVirtualMemory));
  call_addr_NtWriteVirtualMemory=call_addr_NtOpenProecss;
  KdPrint(("NtWriteVirtualMemory 的 call 地址为:%x\n",call_addr_NtWriteVirtualMemory));
  push_addr_NtOpenProecss=Getssdt_addr(in_NtOpenProcess);
  addr_NtOpenProecss=push_addr_NtOpenProecss;
  t_NtOpenProcess=(typedef_NtOpenProcess*)push_addr_NtOpenProecss;
  __asm
  {
    push eax
    mov eax,push_addr_NtOpenProecss
    add eax,1
    mov eax,[eax]
    mov push0_addr_NtOpenProcess,eax
    pop eax
  }
}

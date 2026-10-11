
`KdPrint(("向 NtSetContextThread 写跳转地址的:%x\\n",addr_NtSetContextThread));`

`__asm`
```
{
  push eax
  mov eax,My_NtSetContextThread
  mov My_Function_Addr,eax
  pop eax
}
```
`KdPrint(("自定义 NtSetContextThread 的地址为:%x\\n",My_Function_Addr));`

`_pjmp_code_NtSetContextThread=(pjmp_code)addr_NtSetContextThread;`
`_jmp_code_NtSetContextThread.E9=_pjmp_code_NtSetContextThread->E9;`
`_jmp_code_NtSetContextThread.Fun_addr=_pjmp_code_NtSetContextThread->Fun_addr;`

`PAGED_Open();`
`_pjmp_code_NtSetContextThread->E9=0xE9;`
`_pjmp_code_NtSetContextThread->Fun_addr=(ULONG)(My_Function_Addr-addr_NtSetContextThread-5);`
`PAGED_Exit();`
```
}

#pragma PAGEDCODE

VOID __declspec(naked) My_NtSetContextThread()
{
  __asm
  {
    push 1
    push dword ptr [ebp+8]
    mov eax,addr_NtSetContextThread
    add eax,0xA
    push eax
    mov eax,call_addr_NtSetContextThread
    jmp eax
  }
}

#pragma PAGEDCODE

VOID Un_NtSetContextThread()
{
  PAGED_Open();
  _pjmp_code_NtSetContextThread->E9=_jmp_code_NtSetContextThread.E9;
  _pjmp_code_NtSetContextThread->Fun_addr=_jmp_code_NtSetContextThread.Fun_addr;
  PAGED_Exit();
}
```

`jmp_code _jmp_code_NtGetContextThread1;`
`pjmp_code _pjmp_code_NtGetContextThread1;`

`#pragma INITCODE`

`VOID Hook_NtGetContextThread1()`
```
{
  ULONG My_Function_Addr;
  BYTE* _bp;
  _bp=(BYTE*)Getssdt_addr(in_NtGetContextThread);
  do
  {
    if(( *(_bp-7)==0xBE)&&(*(_bp-2)==0x8B)&&(*(_bp-1)==0xCF)&&(*(_bp))==0xE8)&&(*(_bp+5)==0x5F)&&(*(_bp+6)==0x8B))
    {
      break;
    }
    _bp++;
  } while (1);
  addr_NtGetContextThread1=(ULONG)_bp-9;
  KdPrint(("向 NtGetContextThread 写跳转地址为:%x\\n",addr_NtGetContextThread1));
  _bp=(BYTE*)Getssdt_addr(in_NtTerminateThread);
  do
  {
    if(( *(_bp-9)==0xE8)&&(*(_bp-4)==0x8B)&&(*(_bp-2)==0x8B)&&(*(_bp))==0xE8)&&(*(_bp+5)==0x8B)&&(*(_bp+6)==0xC7))
    {
      break;
    }
    _bp++;
  } while (1);
  call_addr_NtGetContextThread1=(ULONG)_bp;
  __asm
  {
    push eax
    push ebx
    mov eax,call_addr_NtGetContextThread1
    add eax,1
    mov ebx,[eax]
    add eax,ebx
    add eax,5
    sub eax,1
    mov call_addr_NtGetContextThread1,eax
    pop ebx
    pop eax
  }
  KdPrint(("从 NtTerminateThread 获取的 NtGetContextThread 的第二个 call 地址为:%x\\n",call_addr_NtGetContextThread1));
  __asm
  {
    push eax
    mov eax,My_NtGetContextThread1
    mov My_Function_Addr,eax
    pop eax
  }
  KdPrint(("NtGetContextThread 的第二个自写函数地址为:%x\\n",My_Function_Addr));
  Game_call_addr_2=addr_NtGetContextThread1+9;
  __asm
  {
    push eax
    push ebx
    mov eax,Game_call_addr_2
    add eax,1
    mov ebx,[eax]
    add eax,ebx
    add eax,5
    sub eax,1
    mov Game_call_addr_2,eax
    pop ebx
    pop eax
  }
  KdPrint(("获取 NtGetContextThread 的 call 获取的第二个 call 地址为:%x\\n",Game_call_addr_2));
  _pjmp_code_NtGetContextThread1=(pjmp_code)addr_NtGetContextThread1;
  _jmp_code_NtGetContextThread1.E9=_pjmp_code_NtGetContextThread1->E9;
  _jmp_code_NtGetContextThread1.Fun_addr=_pjmp_code_NtGetContextThread1->Fun_addr;
  _jmp_code_NtGetContextThread1.nop[0]=_pjmp_code_NtGetContextThread1->nop[0];
  _jmp_code_NtGetContextThread1.nop[1]=_pjmp_code_NtGetContextThread1->nop[1];
  PAGED_Open();
  _pjmp_code_NtGetContextThread1->E9=0xE9;
  _pjmp_code_NtGetContextThread1->Fun_addr=(ULONG)(My_Function_Addr-addr_NtGetContextThread1-5);
  _pjmp_code_NtGetContextThread1->nop[0]=0x90;
  _pjmp_code_NtGetContextThread1->nop[1]=0x90;
  PAGED_Exit();
}
```

`PEPROCESS eprocess_temp;`
`ANSI_STRING GetProcess_temp1;`
`ANSI_STRING OD_Name_temp1;`
`ANSI_STRING OD_Name_temp2;`
`ANSI_STRING OD_Name_temp3;`

`#pragma PAGEDCODE`

`VOID __declspec(naked) My_NtGetContextThread1()`
```
{
  __asm
  {
    pushad
    pushf
  }
  eprocess_temp=PsGetCurrentProcess();
  RtlInitAnsiString(&GetProcess_temp1,(PCSZ)((ULONG)eprocess_temp+0x174));
  RtlInitAnsiString(&OD_Name_temp1,_Game_Name);
  RtlInitAnsiString(&OD_Name_temp2,_Game_Name1);
  RtlInitAnsiString(&OD_Name_temp3,_Game_Name2);
  if(RtlCompareString(&GetProcess_temp1,&OD_Name_temp1,TRUE)==0)
  {
    __asm
    {
      popf
      popad
      jmp __tiao2
      mov esi,0C0000008h
__tiao2:
      mov ecx,edi
      mov eax,addr_NtGetContextThread1
      add eax,0xE
      push eax
      jmp call_addr_NtGetContextThread1
    }
  }
  else
  {
    if(RtlCompareString(&GetProcess_temp1,&OD_Name_temp2,TRUE)==0)
    {
      __asm
      {
        popf
        popad
        jmp __tiaotemp1
        mov esi,0C0000008h
__tiaotemp1:
        mov ecx,edi
        mov eax,addr_NtGetContextThread1
        add eax,0xE
        push eax
        jmp call_addr_NtGetContextThread1
      }
    }
    else
    {
      if(RtlCompareString(&GetProcess_temp1,&OD_Name_temp3,TRUE)==0)
      {
        __asm
        {
          popf
          popad
          jmp __tiaotemp2
          mov esi,0C0000008h
__tiaotemp2:
          mov ecx,edi
          mov eax,addr_NtGetContextThread1
          add eax,0xE
          push eax
          jmp call_addr_NtGetContextThread1
        }
      }
      else
      {
        __asm
        {
          popf
          popad
          jmp __tiaotemp3
          mov esi,0C0000008h
__tiaotemp3:
          mov ecx,edi
          mov eax,addr_NtGetContextThread1
          add eax,0xE
          push eax
          jmp Game_call_addr_2
        }
      }
    }
  }
}
```

`#pragma PAGEDCODE`

`VOID Un_NtGetContextThread1()`
```
{
  PAGED_Open();
  _pjmp_code_NtGetContextThread1->E9=_jmp_code_NtGetContextThread1.E9;
  _pjmp_code_NtGetContextThread1->Fun_addr=_jmp_code_NtGetContextThread1.Fun_addr;
  _pjmp_code_NtGetContextThread1->nop[0]=_jmp_code_NtGetContextThread1.nop[0];
  _pjmp_code_NtGetContextThread1->nop[1]=_pjmp_code_NtGetContextThread1->nop[1];
  PAGED_Exit();
}
```

`jmp_code _jmp_code_NtQueryPerformanceCounter;`
`pjmp_code _pjmp_code_NtQueryPerformanceCounter;`
`jmp_code _jmp_code_NtQueryPerformanceCounter_1;`
`pjmp_code _pjmp_code_NtQueryPerformanceCounter_1;`

`#pragma INITCODE`

`VOID Hook_NtQueryPerformanceCounter()`
```
{
  ULONG My_temp;
  BYTE* _bp;
  _bp=(BYTE*)addr_NtOpenThread;
  do
  {
    if(( *(_bp-18)==0xC3)&&(*(_bp-14)==0x8B)&&(*(_bp-7)==0xEB)&&(*(_bp))==0xE8)&&(*(_bp+5)==0xC2)&&(*(_bp+6)==0x10))
    {
      break;
    }
    _bp++;
  } while (1);
  call_addr_NtQueryPerformanceCounter=(ULONG)_bp;
  __asm
  {
    push eax
    push ebx
    mov eax,call_addr_NtQueryPerformanceCounter
    add eax,1
    mov ebx,[eax]
    add eax,ebx
    add eax,5
    sub eax,1
    mov call_addr_NtQueryPerformanceCounter,eax
    pop ebx
    pop eax
  }
  call_addr_NtQueryPerformanceCounter=call_addr_NtQueryPerformanceCounter-addr_NtQueryPerformanceCounter;
  KdPrint(("NtQueryPerformanceCounter 的 call 地址为:%x\\n",call_addr_NtQueryPerformanceCounter));
  _bp=(BYTE*)Getssdt_addr(in_NtQueryPerformanceCounter);
  do
  {
    if(( *(_bp-10)==0x8B)&&(*(_bp-7)==0x89)&&(*(_bp-4)==0x83)&&(*(_bp))==0xEB)&&(*(_bp+2)==0x8B)&&(*(_bp+5)==0x8B))
    {
      break;
    }
    _bp++;
```

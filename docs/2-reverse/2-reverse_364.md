

# 【原创】用 DEP 实现 BreakOnExecute

> 原文：[`www.pediy.com/kssd/pediy10/92789.html`](https://www.pediy.com/kssd/pediy10/92789.html)

用 DEP 实现 BreakOnExecute  
  
  
以前抄过 OllyBone,但不大好用,很多时候停不下来,似乎在 PTE 上做的手脚没起作用。后来看到  
DEP,这不就是我想要的功能么,又重新试了一次。  
  
打开 DEP 后,系统工作在 PAE 下,PTE 为 64 位。不过改了以后,还是存在拦不住的情况,用 SoftIce 跟了一下,  
PTE 的确被动过。Hook KeInterlockedSwapPte 就可以了,估计 OllyBone 不好用也是同样道理。  
  
前段时间写的东西,细节已经记不起来了。反正代码都贴出来了。功能跟 OllyBone 一样的。在 xp sp2/sp3 下测试可以用。  
  
  

代码:

```
#define MiGetPteAddressPAE(va)   ((PMMPTE)(PTE\_BASE + ((((ULONG)(va)) >> 12) << 3)))

typedef struct \_HARDWARE\_PTE\_X86PAE {
    union {
        struct {
        ULONGLONG Valid : 1;
        ULONGLONG Write : 1;
        ULONGLONG Owner : 1;
        ULONGLONG WriteThrough : 1;
        ULONGLONG CacheDisable : 1;
        ULONGLONG Accessed : 1;
        ULONGLONG Dirty : 1;
        ULONGLONG LargePage : 1;
        ULONGLONG Global : 1;
        ULONGLONG CopyOnWrite : 1;   // software field
        ULONGLONG Prototype : 1;     // software field
        ULONGLONG reserved0 : 1;    // software field
        ULONGLONG PageFrameNumber : 26;
        //ULONGLONG reserved1 : 26;    // software field
  ULONGLONG reserved1 : 25;    // software field
  ULONGLONG ExecuteDisable : 1;  // 这个
        };
        struct {
            ULONG LowPart;
            ULONG HighPart;
        };
    };
} HARDWARE\_PTE\_X86PAE, \*PHARDWARE\_PTE\_X86PAE;


VOID HookMemory(ULONG address, ULONG size)
{
  ULONG i;

  g\_HookMem = TRUE;
  g\_HookStart = address;
  g\_HookEnd = address + size;

  g\_PteStart = (ULONG)MiGetPteAddressPAE(g\_HookStart);
  g\_PteEnd   = (ULONG)MiGetPteAddressPAE(g\_HookEnd);

  //DbgPrint("PteStart = %08X, PteEnd = %08X", g\_PteStart, g\_PteEnd);
  
  KeAttachProcess(g\_ObjProc);

  for (i = address; i < address+size; i += PAGE\_SIZE) //4KB Pages only
    HookPage((PUCHAR)i);
  
  KeDetachProcess();
}


VOID HookPage(PUCHAR Page)
{
  
  PHARDWARE\_PTE\_X86PAE PointerPte;
    
  
  \_\_try {

    \_\_asm {  
      mov   eax, Page  
      mov   eax, \[eax\]
    }
    
    PointerPte = (PHARDWARE\_PTE\_X86PAE)MiGetPteAddressPAE(Page);

    if (PointerPte->Valid == 1) {
      PointerPte->ExecuteDisable = 1;
      DbgPrint("HookPage = %08X", Page);
    }
  }
  \_\_except(EXCEPTION\_EXECUTE\_HANDLER) {
    DbgPrint("Exception captured while hooking page");
  }
}


VOID UnhookPage(PUCHAR Page)
{
  PHARDWARE\_PTE\_X86PAE PointerPte;

  \_\_try {
    
    \_\_asm {
      mov  eax, Page  
      mov  eax, \[eax\]
    }

    PointerPte = (PHARDWARE\_PTE\_X86PAE)MiGetPteAddressPAE(Page);

    if (PointerPte->Valid == 1) {
      PointerPte->ExecuteDisable = 0;
    }
  } 
  \_\_except(EXCEPTION\_EXECUTE\_HANDLER) {
    DbgPrint("Exception captured while unhooking page");
  }
}


VOID UnhookMemory(ULONG address, ULONG size)
{
  ULONG i;

  KeAttachProcess(g\_ObjProc);
  
  for (i = address; i < address+size; i += PAGE\_SIZE)
    UnhookPage((PUCHAR)i);
  
  KeDetachProcess();
}


NTSTATUS FindSwapPte(ULONG ntosbase, PULONG result)
{
  //
  // Search KeInterlockedSwapPte in kernel space:)
  // there are some lock cmpxchg8b, but this is the only one followed 
  // by a jnz 
  //

  UCHAR code\[\] = { 0xF0, 0x0F, 0xC7, 0x0E,  // lock cmpxchg8b qword ptr \[esi\]
       0x75, 0xFA,      // jnz     short loc\_46EC60
       0x5E,        // pop     esi
       0x5B,        // pop     ebx
       0xC2, 0x08, 0x00    // retn    8
      };


  NTSTATUS status = STATUS\_UNSUCCESSFUL;
  PIMAGE\_DOS\_HEADER mz;
  PIMAGE\_NT\_HEADERS pe;
  ULONG current, end, i;
  SIZE\_T size = sizeof(code);
  
  mz = (PIMAGE\_DOS\_HEADER)ntosbase;
  pe = (PIMAGE\_NT\_HEADERS)((ULONG)ntosbase + mz->e\_lfanew);

  \*result = 0;
  current = ntosbase;
  end = ntosbase + pe->OptionalHeader.SizeOfImage - sizeof(code);

  

  \_\_try {
    while (current < end) {

      if (RtlCompareMemory((PVOID)current, code, size) == size) {
      
        
        // WARNING!
        // the hook func CAN'T work in Win2K3! refer to wrk,
        // the ULONG64 was pushed in stack instead of the pointer
        //

        for (i = 0; i < 32; i++) {

          if (\*(PUSHORT)(current - i) == 0x5653) {
            \*result = current - i;
            
            DbgPrint("KeInterlockedSwapPte = %08X", \*result);
            status = STATUS\_SUCCESS;
            break;
          }
        }
      }
      current++;
    }
  }
  \_\_except(EXCEPTION\_EXECUTE\_HANDLER) {
    DbgPrint("Failed to retrive KeInterlockedSwapPte");
    \*result = 0;
    return STATUS\_FAIL\_GET\_KESWAPPTE;
  }

  return status;
}


VOID HookKeInterlockedSwapPte()
{
  
  ULONG  func = g\_KeInterlockedSwapPte;
  ULONG  newlowpart, newhighpart;
  UCHAR  opcode\[8\] = {0x90};

  opcode\[0\] = 0xE9;
  \*(PULONG)&opcode\[1\] = (ULONG)KeInterlockedSwapPte - (func+5);

  newlowpart  = \*(PULONG)&opcode\[0\];
  newhighpart = \*(PULONG)&opcode\[4\];

  DisableWP
  
  
  \_\_asm {
    pushad

    mov   esi, func
    
    mov   edi, offset g\_SavedSwapPte
    mov   edx, \[esi+4\]
    mov   \[edi+4\], edx  // save the old opcode
    mov   eax, \[esi\]
    mov   \[edi\], eax

    mov   ecx, newhighpart
    mov   ebx, newlowpart

\_\_swapagain:
    lock  cmpxchg8b qword ptr \[esi\]
    jne   \_\_swapagain  
    
    popad
  }

  EnableWP
    
  DbgPrint("Hook KeInterlockedSwapPte OK");
}


VOID \_\_declspec(naked) KeInterlockedSwapPte()
{

  //
  // 这里的代码适用于 xp,WRK 压栈的是 ULONG64,不是指针,要修改
  //

  //   ; ULONGLONG
  //   ; InterlockedExchangePte (
  //   ;     IN OUT PMMPTE Destination,
  //   ;     IN ULONGLONG Exchange
  //   ;     )
  // 
  //   push    ebx
  //    push    esi
  //   mov     ebx, \[esp\] + 16         ; ebx = NewPteContents lowpart
  //   mov     ecx, \[esp\] + 20         ; ecx = NewPteContents highpart
  //   mov     esi, \[esp\] + 12         ; esi = PtePointer


  \_\_asm 
  {
    push    ebx
    push    esi
    mov     ebx, dword ptr \[esp+10h\]  // ebx = pointer to Exchange
    mov     esi, dword ptr \[esp+0Ch\]  // esi = PtePointer
    mov     ecx, dword ptr \[ebx+4\]    // ecx = NewPteContents highpart
    mov     ebx, dword ptr \[ebx\]    // ebx = NewPteContents lowpart
    mov     edx, dword ptr \[esi+4\]    // edx = OldPteContents highpart
    mov     eax, dword ptr \[esi\]    // eax = OldPteContents lowpart, return old contents in edx:eax
      
  \_\_swapagain:

    lock  cmpxchg8b qword ptr \[esi\]
    jne  \_\_swapagain      // if z clear, exchange failed
      
    pushad          // if debuggee, fuck up nx
    call    dword ptr \[IoGetCurrentProcess\]
    cmp  eax, g\_ObjProc
    jne  \_\_end

    cmp  esi, g\_PteStart
    jb  \_\_end
    cmp  esi, g\_PteEnd
    ja  \_\_end

    or  dword ptr \[esi+4\], 80000000h  //管用的就是这句

  \_\_end:
    popad
    pop     esi
    pop     ebx
    ret     8
  }
}


volatile \_\_declspec(naked) void NewInt0E\_PAE()
{
  // 
  //  - Interrupt 0E Handler -
  //
  //  offset   | contains
  //  ---------+-----------------------------
  //  esp      : Error Code
  //  esp + 4  : EIP Context
  //  esp + 8  : CS  Context
  //  esp + C  : EFLAGS
  //

  \_\_asm 
  {
    pushad
    push    fs
    push    ds
    push    es
    mov     eax, 30h
    mov     fs, ax
    mov     eax, 23h
    mov     ds, ax
    mov     es, ax

    call  dword ptr \[IoGetCurrentProcess\]
    cmp  eax, g\_ObjProc     // debuggee?
    jne  \_\_oldint0e
      
    mov  eax, \[esp+20h+0Ch\]  // error code
    test  eax, 1      // not present
    je  \_\_oldint0e

    test  eax, 4
    je  \_\_oldint0e    // skip kernel mode page fault

      
    cmp  g\_HookMem, 0
    je  \_\_oldint0e

    mov  eax, cr2    // the faulting address
    cmp  eax, g\_HookStart
    jb  \_\_oldint0e

    cmp  eax, g\_HookEnd
    jae  \_\_oldint0e

    cmp  eax, \[esp+20h+0Ch+4\]  // eip
    jne  \_\_oldint0e    // because of execution
      
    pop     es
    pop     ds
    pop     fs
              popad
    add  esp, 4      // discard error code
    jmp  g\_KiTrap01    // fuck up int1

\_\_oldint0e:
    pop     es
            pop     ds
          pop     fs
          popad
          jmp  g\_KiTrap0E
      
  }
}

```

最后感谢风月,把他的 VMX 代码给我。有时间再学习啦,我现在时间都耗在聊天上了哈。

```
    dh = (t & 0x0000FF00) >> 8;

    dh ^= 0x20;

    if ((c & 1) == 0)

      dh ^= 0x21;

    t &= 0xFFFF00FF;

    t |= (dh << 8);

  }

  if ((((t & 0x0000FF00) >> 8) & 0x40) != 0 )

  {

    al = *(BYTE *) pOPCode++;

    c = (DWORD)al;

    c |= (al << 8);

    c &= 0xC007;

    if ( (c & 0x0000FF00) != 0xC000 )

    {

      if ( ((t & 0x000000FF) & 0x10) == 0)

      {

          if ((c & 0x000000FF) == 4)

          {

            al = *(BYTE *) pOPCode++;

            al &= 7;

            c &= 0x0000FF00;

            c |= al;

          }

          if ((c & 0x0000FF00) != 0x4000)

          {

            if ((c & 0x0000FF00) == 0x8000)   t |= 4;

            else if (c==5) t |= 4;

          }

          else

            t |= 1;

      }

      else

      {

          if (c != 6)

          {

            if((c & 0x0000FF00) == 0x4000)

              t |= 1;

            else if ((c & 0x0000FF00) == 0x8000)

              t |= 2;

          }

          else

            t |= 2;

      }

    }

  }

  if ((((t & 0x000000FF)) & 0x20) != 0)

  {

    dl = t & 0x000000FF;

    dl ^= 2;

    t &= 0xFFFFFF00;

    t |= dl;

    if ((dl & 0x10) == 0)

    {

      dl ^= 6;

      t &= 0xFFFFFF00;

      t |= dl;

    }

  }

  if ((((t & 0x0000FF00) >> 8) & 0x20) != 0)

  {

    dh = (t & 0x0000FF00) >> 8;

    dh ^= 2;

    t &= 0xFFFF00FF;

    t |= (dh << 8);

    if ((dh & 0x10) == 0)

    {

      if (dh & 0x40) //是否是 0x6x

          dh ^= (t & 0xFF);   // 这句修改了一下，修正了几个指令的计算

      t &= 0xFFFFFF00;

      t |= dh;

    }

  }

  OpCodeSize = (DWORD) pOPCode - (DWORD) Start;

  t &= 0x707;

  OpCodeSize += t & 0x000000FF;

  OpCodeSize += (t & 0x0000FF00) >> 8;

  if (((*(char*)Start) & 0x000000FF) == 0x66) // 单独处理 66 开头的问题

    if ( OpCodeSize >= 6)   //1字节66 ，1字节操作码，4字节操作数，因此至少要大于等于6以上

      OpCodeSize -= 2;   //减2处理 ，将 dword 型转成 word 型

  return OpCodeSize;

}

__declspec(naked) void HookBeforeStub()

{

Stub_Begin:

  __asm

  {

    jmp Code_Begin

    mov eax, offset Stub_Begin

    mov eax, offset Stub_Data

    mov eax, offset Stub_End

    mov eax, offset SaveEntry

Code_Begin:

    call next1

next1:

    pop ecx

    sub ecx, offset next1

    lea ecx, [ecx + Stub_Data]

    mov eax, [ecx + 4]

    mov [ecx + eax * 4 + 0x8],esp

    inc [ecx + 4]

    call [ecx]

    call next2

next2:

    pop ecx

    sub ecx, offset next2

    lea ecx, [ecx + Stub_Data]

    dec [ecx +4]

    mov eax, [ecx + 4]

    mov esp, [ecx + eax * 4 + 0x8]

SaveEntry:

    nop

    nop

    nop

    nop

    nop

    nop

    nop

    nop

    nop

    nop

    nop

    nop

    nop

    nop

    nop

    nop

    _asm _emit 0xE9

    int 3

    int 3

    int 3

    int 3

Stub_Data:

HookProc:

    int 3

    int 3

    int 3

    int 3 //HookProc的地址，计算后填入

pEsp:

    _emit 0

    _emit 0

    _emit 0

    _emit 0//第一个变量，指向该返回的哪个Esp,初始化指向下面一行~~,一开始要清0

SaveEsp:

    int 3

    int 3

    int 3

    int 3 //开始保存Esp的值

Stub_end:

  }

}

__declspec(naked) void HookAfterStub()

{

Stub_Begin:

  __asm

  {

  jmp Code_Begin

    mov eax, offset Stub_Begin

    mov eax, offset Stub_Data

    mov eax, offset Stub_End

    mov eax, offset SaveEntry

    mov eax, offset After_Code

Code_Begin:

  call next1

next1:

  pop ecx

  sub ecx,offset next1

  lea edx,[ecx + Stub_Data]

  mov eax, [edx + 8]

  mov [edx + eax * 8 + 0xC],esp

  push [esp]

  pop dword ptr [edx + eax * 8 + 0x10]

  inc [edx + 8]

  lea edx,[ecx + After_Code]

  mov [esp],edx

SaveEntry:

    nop

    nop

    nop

    nop

    nop

    nop

    nop

    nop

    nop

    nop

    nop

    nop

    nop

    nop

    nop

    nop

    _asm _emit 0xE9

    int 3

    int 3

    int 3

    int 3

After_Code:

  sub esp,100

  call next2

next2:

  pop ecx

  add esp,100

  sub ecx,offset next2

  lea edx,[ecx + Stub_Data]

  mov [edx + 4] ,eax

  dec [edx + 8]

  mov eax, [edx + 8]

  mov ecx, [edx + eax * 8 + 0xC]

  mov [edx + eax * 8 + 0xC],esp

  mov esp,ecx

  mov ecx, [edx + eax * 8 + 0x10]

  inc [edx + 8]

  mov [esp],ecx

  push [edx + 4]

  call [edx]

  call next3

next3:

  pop ecx

  sub ecx,offset next3

  lea edx, [ecx + Stub_Data]

  mov [edx + 4],eax

  dec [edx + 8]

  mov eax, [edx + 8]

  mov esp, [edx + eax * 8 + 0xC]

  push [edx + eax * 8 + 0x10]

  mov eax,[edx + 4]

  retn

Stub_Data:

HookProc:

    int 3

    int 3

    int 3

    int 3 //HookProc的地址，计算后填入

SaveRetthing:

    int 3

    int 3

    int 3

    int 3 //临时保存返回值

pEsp:

    _emit 0

    _emit 0

    _emit 0

    _emit 0//指向该返回的哪个Esp,初始化指向SaveEsp一行~~,一开始要清0

SaveEsp:

    int 3

    int 3

    int 3

    int 3 //SaveEsp

SaveRet:

    int 3

    int 3

    int 3

    int 3 //SaveRet

Stub_end:

  }

}

boolean SetOnAfter(PCHAR DllName,PCHAR ApiName,PVOID HookProc)

{

  PVOID   ApiEntry;

  HMODULE DllHandle;

  int ReplaceCodeSize;

  BYTE OpCode[16];

LPVOID StubPtr;

  DWORD Addr;

  DWORD RetSize=0;

  DWORD SizeOfStub =0;

  DWORD DeltaData = 0;

  DWORD SaveEntry = 0;

  DWORD AfterCode = 0;

  DllHandle = GetModuleHandle(DllName);

  if (DllHandle ==0)

  {

    DllHandle = LoadLibrary(DllName);

    if (DllHandle ==0) return false;

  }

  ApiEntry = GetProcAddress(DllHandle,ApiName);

  if (ApiEntry == NULL) return false;

  ReplaceCodeSize = GetOpCodeSize(ApiEntry);

  while (ReplaceCodeSize < 5)

    ReplaceCodeSize += GetOpCodeSize((PVOID)((DWORD)ApiEntry + (DWORD)ReplaceCodeSize));

  if (ReplaceCodeSize > 16) return false;

  if (VirtualProtect(ApiEntry,ReplaceCodeSize,PAGE_READWRITE,NULL))

    return false;

  CopyMemory(OpCode, ApiEntry, ReplaceCodeSize);

  DeltaData = *(DWORD *)((DWORD)HookAfterStub + 0x8) - *(DWORD *) ((DWORD)HookAfterStub + 0x3);

  SizeOfStub = *(DWORD *)((DWORD)HookAfterStub + 0x0D) - *(DWORD *) ((DWORD)HookAfterStub + 0x3);

  SaveEntry = *(DWORD *)((DWORD)HookAfterStub + 0x12) - *(DWORD *) ((DWORD)HookAfterStub + 0x3);

  AfterCode = *(DWORD *)((DWORD)HookAfterStub + 0x17) - *(DWORD *) ((DWORD)HookAfterStub + 0x3);

  StubPtr = VirtualAlloc(NULL, SizeOfStub + 0x100*8, MEM_COMMIT, PAGE_EXECUTE_READWRITE);

  if (StubPtr == NULL) return false;

  CopyMemory(StubPtr, HookAfterStub, SizeOfStub);

  Addr = (DWORD)HookProc;

  *(DWORD *) ((DWORD)StubPtr + DeltaData) = Addr;

  Addr = (DWORD)ApiEntry + ReplaceCodeSize - (DWORD)StubPtr - AfterCode;

  *(DWORD *) ((DWORD)StubPtr + AfterCode - 4) = Addr;

  CopyMemory((LPVOID)((DWORD)StubPtr + SaveEntry), OpCode, ReplaceCodeSize);

  Addr = (DWORD)StubPtr - (DWORD)ApiEntry - 5;

  *(DWORD*)(JMPGate + 1) = Addr;

  WriteProcessMemory(GetCurrentProcess(), ApiEntry, JMPGate, sizeof(JMPGate), (DWORD*)RetSize);

  return true;

}

boolean SetOnBefore(PCHAR DllName,PCHAR ApiName,PVOID HookProc)

{

  PVOID   ApiEntry;

  HMODULE DllHandle;

  int ReplaceCodeSize;

  BYTE OpCode[16];

LPVOID StubPtr;

  DWORD Addr;

  DWORD RetSize =0;

  DWORD SizeOfStub =0;

  DWORD DeltaData = 0;

  DWORD SaveEntry = 0;

  DllHandle = GetModuleHandle(DllName);

  if (DllHandle ==0)

  {

    DllHandle = LoadLibrary(DllName);

    if (DllHandle ==0) return false;

  }

  ApiEntry = GetProcAddress(DllHandle,ApiName);

  if (ApiEntry == NULL) return false;

  ReplaceCodeSize = GetOpCodeSize(ApiEntry);

  while (ReplaceCodeSize < 5)

    ReplaceCodeSize += GetOpCodeSize((PVOID)((DWORD)ApiEntry + (DWORD)ReplaceCodeSize));

  if (ReplaceCodeSize > 16) return false;

  if (VirtualProtect(ApiEntry,ReplaceCodeSize,PAGE_READWRITE,NULL))

    return false;

  CopyMemory(OpCode, ApiEntry, ReplaceCodeSize);

  DeltaData = *(DWORD *)((DWORD)HookBeforeStub + 0x8) - *(DWORD *) ((DWORD)HookBeforeStub + 0x3);

  SizeOfStub = *(DWORD *)((DWORD)HookBeforeStub + 0x0D) - *(DWORD *) ((DWORD)HookBeforeStub + 0x3);

  SaveEntry = *(DWORD *)((DWORD)HookBeforeStub + 0x12) - *(DWORD *) ((DWORD)HookBeforeStub + 0x3);
```

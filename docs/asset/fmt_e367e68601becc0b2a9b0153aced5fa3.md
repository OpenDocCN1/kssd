
```c
  return false;  

CopyMemory(OpCode, ApiEntry, ReplaceCodeSize);  

DeltaData = *(DWORD *) ((DWORD)HookAfterStub + 0x8) - *(DWORD *) ((DWORD)HookAfterStub + 0x3);  
SizeOfStub = *(DWORD *) ((DWORD)HookAfterStub + 0x0D) - *(DWORD *) ((DWORD)HookAfterStub + 0x3);  
SaveEntry = *(DWORD *) ((DWORD)HookAfterStub + 0x12) - *(DWORD *) ((DWORD)HookAfterStub + 0x3);  
AfterCode = *(DWORD *) ((DWORD)HookAfterStub + 0x17) - *(DWORD *) ((DWORD)HookAfterStub + 0x3);  

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

boolean SetOnBefore(PCHAR DllName, PCHAR ApiName, PVOID HookProc)  
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

  ApiEntry = GetProcAddress(DllHandle, ApiName);  
  if (ApiEntry == NULL) return false;  

  ReplaceCodeSize = GetOpCodeSize(ApiEntry);  

  while (ReplaceCodeSize < 5)  
    ReplaceCodeSize += GetOpCodeSize((PVOID)((DWORD)ApiEntry + (DWORD)ReplaceCodeSize));  

  if (ReplaceCodeSize > 16) return false;  

  if (VirtualProtect(ApiEntry, ReplaceCodeSize, PAGE_READWRITE, NULL))  
    return false;  

  CopyMemory(OpCode, ApiEntry, ReplaceCodeSize);  

  DeltaData = *(DWORD *) ((DWORD)HookBeforeStub + 0x8) - *(DWORD *) ((DWORD)HookBeforeStub + 0x3);  
  SizeOfStub = *(DWORD *) ((DWORD)HookBeforeStub + 0x0D) - *(DWORD *) ((DWORD)HookBeforeStub + 0x3);  
  SaveEntry = *(DWORD *) ((DWORD)HookBeforeStub + 0x12) - *(DWORD *) ((DWORD)HookBeforeStub + 0x3);  


  StubPtr = VirtualAlloc(NULL, SizeOfStub + 0x100*4, MEM_COMMIT, PAGE_EXECUTE_READWRITE);  

  if (StubPtr == NULL) return false;  

  CopyMemory(StubPtr, HookBeforeStub, SizeOfStub);  

  Addr = (DWORD)HookProc;  
  *(DWORD *) ((DWORD)StubPtr + DeltaData) = Addr;  

  Addr = (DWORD)ApiEntry + ReplaceCodeSize - (DWORD)StubPtr - DeltaData;  
  *(DWORD *) ((DWORD)StubPtr + DeltaData - 4) = Addr;  


  CopyMemory((LPVOID)((DWORD)StubPtr + SaveEntry), OpCode, ReplaceCodeSize);  

  Addr = (DWORD)StubPtr - (DWORD)ApiEntry - 5;  
  *(DWORD*)(JMPGate + 1) = Addr;  

  WriteProcessMemory(GetCurrentProcess(), ApiEntry, JMPGate, sizeof(JMPGate), (DWORD*)RetSize);  

  return true;  
}  

void My_WriteProcessMemory(DWORD Eax, DWORD RetAddr, HANDLE hProcess, LPVOID lpBaseAddress, LPVOID lpBuffer, DWORD nSize, LPDWORD lpNumberOfBytesWritten)  
{  
  DWORD RetSize =0;  
  char Text [255] = {0};  
  if (!IsMe)  
  {  
    IsMe =true;  
    sprintf(Text, "EAX = %2X ,RetAddr = %2X", Eax, RetAddr);  
    MessageBox (NULL, Text, "RetAddr", NULL);  
    WriteProcessMemory(GetCurrentProcess(), (LPVOID)0x40108A, JMPGate, sizeof(JMPGate), (DWORD*)RetSize);  
    IsMe = false;  
  }  
}  


int main()  
{  
  DWORD RetSize =0;  
  SetOnAfter("Kernel32.dll", "WriteProcessMemory", My_WriteProcessMemory);  

  WriteProcessMemory(GetCurrentProcess(), (LPVOID)0x40108f, JMPGate, sizeof(JMPGate), (DWORD*)RetSize);  

  MessageBoxA(NULL, "Safe Here!!!", "Very Good!!", NULL);  
  return 0;  
}
```

本版本主要更新，实现 `hook` 时可以重入，在 `debugman` 测试了，基本能用。

用法简单说一下，和以前的不一样。

如果是 `SetOnBefore`，原来的函数要加上 `DWORD RetAddr`，这个是原来的返回地址。例如：

```c
HMODULE WINAPI LoadLibraryA(
LPCTSTR lpFileName
)
```

自己的要这么写：

```c
HMODULE WINAPI My_LoadLibraryA(
DWORD RetAddr,
LPCTSTR lpFileName
)
```

如果是 `SetOnAfter`，要加 2 个参数，依次为 `DWORD EAX`，`DWORD RetAddr`，`EAX` 是执行完 API 的返回值：

```c
HMODULE WINAPI My_LoadLibraryA(
DWORD EAX,
DWORD RetAddr,
LPCTSTR lpFileName
)
```

4dd466d0e1e0cc123afbe504da2462ca

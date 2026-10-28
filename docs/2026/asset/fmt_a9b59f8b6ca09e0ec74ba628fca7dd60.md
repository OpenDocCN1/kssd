
```
// Hook information.

typedef struct _HOOK_ENTRY
{
	LPVOID TargetFunctionAddress;        
	LPVOID FakeFunctionAddress;           
	LPVOID MemorySlot;        
	UINT8  Backup[8];           //恢复Hook使用的存放原先数据

//	UINT8  patchAbove : 1;     // Uses the hot patch area.   位域：1位
	UINT8  IsEnabled  : 1;     // Enabled.
//	UINT8  queueEnable : 1;     // Queued for enabling/disabling when != isEnabled.

	UINT   Index : 4;            // Count of the instruction boundaries.???
	UINT8  OldIPs[8];           // Instruction boundaries of the target function.???
	UINT8  NewIPs[8];           // Instruction boundaries of the trampoline function ???
} HOOK_ENTRY, *PHOOK_ENTRY;     //44字节

typedef struct _HOOK_INFORMATION_
{
	PHOOK_ENTRY HookEntry;         
	UINT        MaximumLength;    
	UINT        Length;          
}HOOK_INFORMATION,*PHOOK_INFORMATION;
```

首先第一步是检查函数是否已经被Hook过，如果已经被Hook过的话，就不必再一次填充`HOOK_INFORMATION`结构了。

```
UINT SeFindHookEntry(LPVOID FunctionAddress)
{
	UINT i;
	for (i = 0; i < __Hooks.Length; ++i)
	{
		if ((ULONG_PTR)FunctionAddress == (ULONG_PTR)__Hooks.HookEntry[i].TargetFunctionAddress)
			return i;
	}
	return STATUS_NOT_FOUND;
}
```

如果没有被自己Hook过的话，就先准备一下`TRAMPOLINE`结构体：

```
#pragma pack(1)
typedef struct _TRAMPOLINE
{
	LPVOID TargetFunctionAddress;      
	LPVOID FakeFunctionAddress;        
	LPVOID MemorySlot;                   // MemorySlot 32字节

#if defined(_M_X64) || defined(__x86_64__)
	LPVOID pRelay;          // [Out] Address of the relay function.
#endif
//	BOOL   patchAbove;      // [Out] Should use the hot patch area?  //Patch  --->补丁   //0xA 0xB
	UINT   Index;           // [Out] Number of the instruction boundaries.
	UINT8  OldIPs[8];       // [Out] Instruction boundaries of the target function.      //恢复
	UINT8  NewIPs[8];       // [Out] Instruction boundaries of the trampoline function.  //Hook
} TRAMPOLINE, *PTRAMPOLINE;
```

```
typedef struct _MEMORY_SLOT
{
	union
	{
		struct _MEMORY_SLOT *Flink;
		UINT8 BufferData[MEMORY_SLOT_SIZE];
	};
} MEMORY_SLOT, *PMEMORY_SLOT;  //32字节
```

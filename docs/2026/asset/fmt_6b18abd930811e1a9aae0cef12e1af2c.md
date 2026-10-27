
```cpp
void GetPEB64(void *peb64) {
static uint8_t code[] = {
/* [bits 32]
mov esi, peb64
push 0x33
push _next_x64_code
retf
*/
0xBE, 0x78, 0x56, 0x34, 0x12, 0x6A, 0x33, 0x68, 0x78, 0x56, 0x34, 0x12, 0xCB,
/* [bits 64]
mov rax, gs:[0x60]
mov [esi], rax
*/
0x65, 0x48, 0xA1, 0x60, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x67, 0x48, 0x89, 0x6,
/* [bits 64]
push 0x23
push _next_x86_code
retfq
*/
0x6A, 0x23, 0x68, 0x78, 0x56, 0x34, 0x12, 0x48, 0xCB,
/* [bits 32]
ret
*/
0xC3
};
static uint32_t ptr = NULL;
if (!ptr) {
ptr = (uint32_t)VirtualAlloc(NULL, sizeof(code), MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
for (int i = 0; i < sizeof(code); i++) ((PBYTE)ptr)[i] = code[i];
}
*(uint32_t*)(ptr + 1) = (uint32_t)peb64;
*(uint32_t*)(ptr + 8) = ptr + 13;
*(uint32_t*)(ptr + 31) = ptr + 37;
((void(*)())ptr)();
}
```

## GetModuleHandle64

函数声明：

```cpp
uint64_t GetModuleHandle64(const WCHAR *moduleName);
```

该函数的作用是获取名为`moduleName`的模块的基址。

实现步骤如下：

1. 从`PEB+0x18`获取Ldr的地址

2. 从`Ldr+0x10`获取InLoadOrderModuleList地址

3. 遍历InLoadOrderModuleList获取模块基址

4. 通过模块基址获取模块名，并与`moduleName`比对，比对成功则返回该模块基址

![img](upload/attach/202111/910514_2P63EXJUZEMVXD9.png)

在WinDbg中使用`dt`指令查看结构体，可以看到Ldr的地址在PEB中的偏移为0x018：

```
0:000> dt _PEB
ntdll!_PEB
+0x000 InheritedAddressSpace : UChar
+0x001 ReadImageFileExecOptions : UChar
+0x002 BeingDebugged : UChar
+0x003 BitField : UChar
+0x003 ImageUsesLargePages : Pos 0, 1 Bit
+0x003 IsProtectedProcess : Pos 1, 1 Bit
+0x003 IsImageDynamicallyRelocated : Pos 2, 1 Bit
+0x003 SkipPatchingUser32Forwarders : Pos 3, 1 Bit
+0x003 IsPackagedProcess : Pos 4, 1 Bit
+0x003 IsAppContainer : Pos 5, 1 Bit
+0x003 IsProtectedProcessLight : Pos 6, 1 Bit
+0x003 IsLongPathAwareProcess : Pos 7, 1 Bit
+0x004 Padding0 : [4] UChar
+0x008 Mutant : Ptr64 Void
+0x010 ImageBaseAddress : Ptr64 Void
+0x018 Ldr : Ptr64 _PEB_LDR_DATA
```

用之前实现的`memcpy64`函数拷贝Ldr的地址：

```cpp
uint64_t peb64;
uint64_t ldrData;
GetPEB64(&peb64);
memcpy64((uint64_t)&ldrData, peb64 + 0x18, 8);
```

打印`_PEB_LDR_DATA`结构体，可以看到`InLoadOrderModuleList`在Ldr中的偏移为0x10：

```
0:000> dt _PEB_LDR_DATA
ntdll!_PEB_LDR_DATA
+0x000 Length : Uint4B
+0x004 Initialized : UChar
+0x008 SsHandle : Ptr64 Void
+0x010 InLoadOrderModuleList : _LIST_ENTRY
+0x020 InMemoryOrderModuleList : _LIST_ENTRY
+0x030 InInitializationOrderModuleList : _LIST_ENTRY
+0x040 EntryInProgress : Ptr64 Void
+0x048 ShutdownInProgress : UChar
+0x050 ShutdownThreadId : Ptr64 Void
```

拷贝`InLoadOrderModuleList`的地址：

```cpp
uint64_t head;
uint64_t pNode;
head = ldrData + 0x10;
memcpy64((uint64_t)&pNode, head, 8);
```

`InLoadOrderModuleList`的实际类型为`_LDR_DATA_TABLE_ENTRY`，`BaseDllName`中存储了DLL的名称，类型为`_UNICODE_STRING`：

```
0:000> dt _LDR_DATA_TABLE_ENTRY
ntdll!_LDR_DATA_TABLE_ENTRY
+0x000 InLoadOrderLinks : _LIST_ENTRY
+0x010 InMemoryOrderLinks : _LIST_ENTRY
+0x020 InInitializationOrderLinks : _LIST_ENTRY
+0x030 DllBase : Ptr64 Void
+0x038 EntryPoint : Ptr64 Void
+0x040 SizeOfImage : Uint4B
+0x048 FullDllName : _UNICODE_STRING
+0x058 BaseDllName : _UNICODE_STRING
0:000> dt _UNICODE_STRING
ntdll!_UNICODE_STRING
+0x000 Length : Uint2B
+0x002 MaximumLength : Uint2B
+0x008 Buffer : Ptr64 Wchar
```

所以Buffer的偏移量在`_LDR_DATA_TABLE_ENTRY`中的偏移为`0x58+0x08`，即96。

遍历链表的代码如下：

```cpp
while (pNode != head) {
uint64_t buffer;
memcpy64((uint64_t)(unsigned)(&buffer), pNode + 96, 8); // tmp = pNode->BaseDllName->Buffer
if (buffer) {
WCHAR curModuleName[32] = {0};
memcpy64((uint64_t)curModuleName, buffer, 60);
if (!lstrcmpiW(moduleName, curModuleName)) {
uint64_t base;
memcpy64((uint64_t)&base, pNode + 48, 8);
return base;
}
}
memcpy64((uint64_t)&pNode, pNode, 8); // pNode = pNode->Flink
}
```

完整代码如下：

```cpp
uint64_t GetModuleHandle64(const WCHAR *moduleName) {
uint64_t peb64;
/* nt!_PEB_LDR_DATA
+0x000 Length : Uint4B
+0x004 Initialized : UChar
+0x008 SsHandle : Ptr64 Void
+0x010 InLoadOrderModuleList : _LIST_ENTRY
*/
uint64_t ldrData;
/*
ptr to InLoadOrderModuleList
*/
uint64_t head;
/*
typedef struct _LDR_MODULE {
+0x000 LIST_ENTRY InLoadOrderModuleList;
+0x010 LIST_ENTRY InMemoryOrderModuleList;
+0x020 LIST_ENTRY InInitializationOrderModuleList;
+0x030 PVOID BaseAddress;
+0x038 PVOID EntryPoint;
+0x040 ULONG SizeOfImage;
+0x048 UNICODE_STRING FullDllName;
+0x058 UNICODE_STRING BaseDllName;
...
} LDR_MODULE, *PLDR_MODULE;
*/
uint64_t pNode;
GetPEB64(&peb64);
memcpy64((uint64_t)&ldrData, peb64 + 0x18, 8);
head = ldrData + 0x10;
memcpy64((uint64_t)&pNode, head, 8);
while (pNode != head) {
uint64_t buffer;
memcpy64((uint64_t)(unsigned)(&buffer), pNode + 96, 8); // tmp = pNode->BaseDllName->Buffer
if (buffer) {
WCHAR curModuleName[32] = {0};
memcpy64((uint64_t)curModuleName, buffer, 60);
if (!lstrcmpiW(moduleName, curModuleName)) {
uint64_t base;
memcpy64((uint64_t)&base, pNode + 48, 8);
return base;
}
}
memcpy64((uint64_t)&pNode, pNode, 8); // pNode = pNode->Flink
}
return NULL;
}
```

## MyGetProcAddress

函数声明：

```cpp
uint64_t MyGetProcAddress64(uint64_t hModule, const char* func);
```

通过`GetModuleHandle64`获取模块地址后，此时还无法通过`kernel32.dll`中的`GetProcAddress`函数获取模块中函数的地址，可以通过遍历模块的导出表获取函数地址作为过渡方案。

![img](upload/attach/202111/910514_2RYXWQUEJFER6MW.png)

首先获取导出表地址，这部分涉及PE文件结构，不再赘述了：

```cpp
IMAGE_DOS_HEADER dos;
memcpy64((uint64_t)&dos, hModule, sizeof(dos));
IMAGE_NT_HEADERS64 nt;
memcpy64((uint64_t)&nt, hModule + dos.e_lfanew, sizeof(nt));
IMAGE_EXPORT_DIRECTORY expo;
memcpy64((uint64_t)&expo, hModule + nt.OptionalHeader.DataDirectory[0].VirtualAddress, sizeof(expo));
```

随后遍历导出表，从导出表中读取函数的名称和地址，将函数名称与`func`进行比对，比对成功则返回函数地址：

```cpp
for (uint64_t i = 0; i < expo.NumberOfNames; i++) {
DWORD pName;
memcpy64((uint64_t)&pName, hModule + expo.AddressOfNames + (4 * i), 4);
char name[64] = {0};
memcpy64((uint64_t)name, hModule + pName, 64);
if (!lstrcmpA(name, func)) {
WORD ord;
memcpy64((uint64_t)&ord, hModule + expo.AddressOfNameOrdinals + (2 * i), 2);
uint32_t addr;
memcpy64((uint64_t)&addr, hModule + expo.AddressOfFunctions + (4 * ord), 4);
return hModule + addr;
}
}
```

完整代码如下：

```cpp
uint64_t MyGetProcAddress(uint64_t hModule, const char* func) {
IMAGE_DOS_HEADER dos;
memcpy64((uint64_t)&dos, hModule, sizeof(dos));
IMAGE_NT_HEADERS64 nt;
memcpy64((uint64_t)&nt, hModule + dos.e_lfanew, sizeof(nt));
IMAGE_EXPORT_DIRECTORY expo;
memcpy64((uint64_t)&expo, hModule + nt.OptionalHeader.DataDirectory[0].VirtualAddress, sizeof(expo));
for (uint64_t i = 0; i < expo.NumberOfNames; i++) {
DWORD pName;
memcpy64((uint64_t)&pName, hModule + expo.AddressOfNames + (4 * i), 4);
char name[64] = {0};
memcpy64((uint64_t)name, hModule + pName, 64);
if (!lstrcmpA(name, func)) {
WORD ord;
memcpy64((uint64_t)&ord, hModule + expo.AddressOfNameOrdinals + (2 * i), 2);
uint32_t addr;
memcpy64((uint64_t)&addr, hModule + expo.AddressOfFunctions + (4 * ord), 4);
return hModule + addr;
}
}
return NULL;
}
```

## X64Call

函数声明：

```cpp
uint64_t X64Call(uint64_t proc, uint32_t argc, ...);
```

由于32位与64位的函数调用的传参方式不同，以及在上一步中我们通过`MyGetProcAddress`函数获取的函数地址为64位，肯定不能直接转化为函数指针调用，所以我们需要用64位汇编实现一个64位函数的调用。

首先来简单了解一下64位中WINAPI调用的传参方式：

- 前四个参数从左往右依次存放到`rcx`, `rdx`, `r8`, `r9`寄存器中

- 后面的参数从右往左依次入栈

- rsp与最后一个参数之间直接需要保留大小为20字节的空间，被调函数可能会使用

![image-20211106141230284](upload/attach/202111/910514_F927YAMXB9WKZ8D.png)

构造shellcode，因为接下来我们需要对栈指针进行操作，所以首先将`esp`保存到`ebx`中，shellcode执行完毕后需要复原。`and esp, 0xFFFFFFF8`的作用是使`rsp`与8对齐，这是64位汇编的栈对齐要求，否则在执行某些系统调用时可能会出错：

```asm
[bits 32]
push ebx
mov ebx, esp
and esp, 0xFFFFFFF8
push 0x33
push _next_x64_code
retf
```

切换到64位后按照64位WINAPI调用协定传参，调用函数之前保留32字节的空间，将函数返回值保存到rax。在shellcode执行前后保存和复原`rsi`和`rdi`：

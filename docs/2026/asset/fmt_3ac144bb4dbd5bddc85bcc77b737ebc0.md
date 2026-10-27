
# CreateFile 定义

`HANDLE CreateFileA(`
`LPCSTR lpFileName,`
`DWORD dwDesiredAccess,`
`DWORD dwShareMode,`
`LPSECURITY_ATTRIBUTES lpSecurityAttributes,`
`DWORD dwCreationDisposition,`
`DWORD dwFlagsAndAttributes,`
`HANDLE hTemplateFile`
`);`

这是 msdn 对 `CreateFile` 的定义，简单来看我们可以只关注 `lpFileName` 和返回值，`lpFileName` 传递你要打开的文件，返回值是操作系统给你的一个代表文件的句柄（`handle`）。

```c
HANDLE hFile = CreateFileA("c:\\temp\\test.txt", ...);
```

要对文件进行读、写等操作都需要这个句柄，也就是说这个句柄至关重要，它表示文件正在被使用。

然后什么时候结束使用呢，我们需要看另一个 API `CloseHandle`。

```c
BOOL CloseHandle(
HANDLE hObject
);
```

`CloseHandle` 用于关闭一个正在被使用的文件，通过句柄来关闭。

现在明白过来了吗，只要我们让进程调用 `CloseHandle` 这个 API，关闭被占用的文件句柄，那么该文件也就被解除占用了。

哈哈，是不是很简单。

# 枚举占用文件的进程

那么我就想问同学们一个问题，怎么知道哪些进程在使用我们想删除的文件呢？怎么去查找？

带着这个问题，我们继续往下看。

我们来想一个问题，操作系统给调用 `CreateFile` 的用户返回了一个句柄，然后通过句柄来操作文件，那操作系统是如何知道句柄代表哪个文件呢？

我们简单思考一下，我们要做到这个目的有没有什么方法，比如我用一个数组来存用户打开的文件路径，而数组序号就返回给用户，下次用户就只需要把序号给我，我就知道要操作什么问题了。

```c
演示代码，忽略细节
LPWSTR FileTable[100] = {0};
HANDLE CreateFileA(
LPCSTR lpFileName,
...)
{
for(int i = 0; i < 100; i++) {
if(FileTable[i] == NULL) {
// 还有空位
FileTable[i] = lpFileName;
// 保存路径
return (HANDLE)i;
// 返回句柄
}
}
return NULL;
}
BOOL CloseHandle(
HANDLE hObject
) {
if((int)hObject < 100) {
if(FileTable[hObject]) {
FileTable[hObject] = NULL;// 找到文件路径
return TRUE;
}
}
return FALSE;
}
```

上面简单的代码演示了一下我们粗略考略的文件和句柄的关系以及句柄的管理，那操作系统是不是这么做的呢？其实也差不多。

任意进程，只要每打开一个对象（包括文件、进程、线程等等），就会获得一个句柄。

这个句柄用来标志对某个对象的一次打开，通过句柄，可以直接找到对应的内核对象。

每个进程都有一个句柄表，用来记录本进程打开的所有内核对象。

句柄表可以简单看做为一个一维数组，每个表项就是一个句柄，一个结构体，一个句柄描述符。

```c
struct _HANDLE_TABLE_ENTRY // 句柄描述符
struct _HANDLE_TABLE // 句柄表描述符
```

好，更加细节的句柄表的原理我们不用再深究，我们只需要知道每个进程都有一个句柄表，通过句柄表就可以找到打开的文件。

这就是我们的目的，我们需要查到进程是不是打开了我们要删除的文件，我们需要查句柄表。

那怎么查呢？

操作系统给用户提供了一个接口 `ZwQuerySystemInformation`。

```c
NTSTATUS WINAPI ZwQuerySystemInformation(
_In_ SYSTEM_INFORMATION_CLASS SystemInformationClass,
_Inout_ PVOID SystemInformation,
_In_ ULONG SystemInformationLength,
_Out_opt_ PULONG ReturnLength
);
```

它可以获取系统非常多的信息，包括进程、模块、处理器、内存等等各种信息。

而 `SystemHandleInformation = 16` 就能获取到系统所有的句柄信息。

```c
typedef struct _SYSTEM_HANDLE_TABLE_ENTRY_INFO
{
USHORT UniqueProcessId;// 所属进程
USHORT CreatorBackTraceIndex;
UCHAR ObjectTypeIndex;
UCHAR HandleAttributes;
USHORT HandleValue; // 句柄
PVOID Object;
ULONG GrantedAccess;
} SYSTEM_HANDLE_TABLE_ENTRY_INFO, *PSYSTEM_HANDLE_TABLE_ENTRY_INFO;

typedef struct _SYSTEM_HANDLE_INFORMATION
{
ULONG NumberOfHandles;
SYSTEM_HANDLE_TABLE_ENTRY_INFO Handles[1];
} SYSTEM_HANDLE_INFORMATION, *PSYSTEM_HANDLE_INFORMATION;
```

既然知道了方法，下面就开始枚举所有句柄，找到我们被占用的文件的进程信息。

```c
Status = ZwQuerySystemInformation(SystemHandleInformation,
Information,
Length,
&ReturnLength);

for (i = 0; i < Information->NumberOfHandles; i++) {
if (Information->Handles[i].UniqueProcessId != CurrentProcessId) {// 不是当前进程
Status = ZwQueryObject(TargetHandle, ObjectTypeInformation, &TypeInfo, sizeof(TypeInfo), NULL);
RtlInitUnicodeString(&TargetType, L"File");
if (!RtlEqualUnicodeString(&TypeInfo.Info.TypeName, &TargetType, FALSE)) {
goto __next;
}
Status = ZwQueryObject(TargetHandle, ObjectNameInformation, &NameInfo, sizeof(NameInfo), NULL);
if (RtlEqualUnicodeString(&NameInfo.Info.Name, &FileName, FALSE)) {
printf("在进程(%d)发现文件占用：(%x) %wZ\n",
ProcessId,
Information->Handles[i].HandleValue,
&NameInfo.Info.Name);
}
}
}
```

`ZwQuerySystemInformation` 获取到所有句柄信息，通过循环枚举 `Information->Handles`，找到句柄类型属于 `File`，路径是目标文件的进程。

`ZwQueryObject` 传入 `ObjectTypeInformation` 可以获取句柄类型，`ZwQueryObject` 传入 `ObjectNameInformation` 可以获取文件路径。

如此两个条件的对比，就能让我们找到占用文件的进程了。

是不是感觉还挺简单，不复杂嘛。

# 坑一：ZwQueryObject

前面提到，每个进程都有自己的句柄表，所以 `ZwQuerySystemInformation` 枚举拿到的句柄并不能直接使用，还需要复制一份到本进程才有效。

系统也提供了 API 叫做 `DuplicateHandle`。

```c
BOOL DuplicateHandle(
HANDLE hSourceProcessHandle,
HANDLE hSourceHandle,
HANDLE hTargetProcessHandle,
LPHANDLE lpTargetHandle,
DWORD dwDesiredAccess,
BOOL bInheritHandle,
DWORD dwOptions
);

DuplicateHandle(hSrcProc, Information->Handles[i].HandleValue, hCurProc, TargetHandle, ...);
```

上面我们使用的 `TargetHandle` 就是通过复制获取的。

这个地方并不是坑，而是在通过 `ZwQueryObject` 获取句柄对应的文件路径时，会发生阻塞，导致程序卡死无法继续运行。

```
0: kd> kv
# ChildEBP RetAddr Args to Child
00 d7fdb7cc 828aacda 00000000 00000000 a7d73040 nt!KiSwapContext+0x19 (FPO: [Uses EBP] [1,0,4])
01 d7fdb86c 828aa358 d7fdb930 a7d73120 a7d73040 nt!KiSwapThread+0x4aa (FPO: [Non-Fpo])
02 d7fdb8c8 828a9d67 00000000 00000000 00000000 nt!KiCommitThreadWait+0x128 (FPO: [Non-Fpo])
03 d7fdb978 829298a3 8ff18afc 00000000 a7d73300 nt!KeWaitForSingleObject+0x1f7 (FPO: [Non-Fpo])
04 d7fdb9a4 82c0759f 88c0e801 d7fdba18 8ff18ab0 nt!IopWaitForLockAlertable+0x3f (FPO: [Non-Fpo])
05 d7fdb9cc 82d3f75c 88c0e800 a7d733f8 d7fdb9ef nt!IopWaitAndAcquireFileObjectLock+0x41 (FPO: [Non-Fpo])
06 d7fdba1c 82bed31a 000001ee d7fdbb01 9a651dc0 nt!IopQueryXxxInformation+0x150f3e
07 d7fda9c 82becf65 00000000 007af7a4 00000210 nt!IopQueryNameInternal+0x31a (FPO: [Non-Fpo])
08 d7fdbab8 82bece25 8ff18ab0 87ff2400 007af7a4 nt!IopQueryName+0x1b (FPO: [Non-Fpo])
09 d7fdbb40 82bec6a6 00000210 d7fdbc04 d7fdbb01 nt!ObQueryNameStringMode+0x495 (FPO: [Non-Fpo])
0a d7fdbbf8 829cce6b 8ff18ab0 00000000 007af7a4 nt!NtQueryObject+0x186 (FPO: [SEH])
0b d7fdbbf8 77cd5ef0 8ff18ab0 00000000 007af7a4 nt!KiSystemServicePostCall (FPO: [0,3] TrapFrame @ d7fdbc14)
```

经过一些简单的分析，如果文件被是同步（`SYNCHRONIZE`）打开的，内核会等待一下锁，等其他线程操作完成，本线程才能拿到所有权。


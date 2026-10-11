
# 被占用文件操作三法

> 原文：`[www.pediy.com/kssd/pediy09/pediy09-719.htm](https://www.pediy.com/kssd/pediy09/pediy09-719.htm)`

> 被占用文件操作三法

无疑我们中的很多人都会遇到需要读写被其它进程占用的文件的情况，比如说在编写 `backup` 程序或是 `trojan` 的时候。能从系统中抽出 `SAM` 文件，或是读取其它某些用标准方法无法成功访问的文件显然是件不错的事情。比如说当用标志 `dwShareMode = 0` 打开文件时，其它进程就不能对它进行访问了。一个很好的例子就是网络寻呼机程序 `Miranda`。这个程序在自己工作的时候不允许别人打开自己的数据库。假设我们需要写一个这样的木马，它在感染机器后从数据库中窃走密码，然后删除自身，这个时候就需要解决这个问题。所以我决定写下这篇文章。文章篇幅不大，但里面的内容可能会对某些人有益。那我们就开始吧。

### 寻找打开文件的句柄

如果文件由某个进程打开，那么这个进程就拥有了它的句柄。在我第二篇关于 API 拦截的文章里我讲解了如何搜索需要的句柄并用它打开进程，要访问已打开的文件，我们也可以使用这种方法。我们需要使用 `ZwQuerySystemInformation` 函数来枚举句柄，将每一个句柄都用 `DuplicateHandle` 进行复制，确定句柄属于那个文件（`ZwQueryInformationFile`），如果是要找的文件，就将句柄拷贝。

这些在理论上都讲得通，但在实践中会遇到两处难点。第一，在对打开的 `named pipe`（工作于 `block mode`）的句柄调用 `ZwQueryInformationFile` 的时候，调用线程会等待 `pipe` 中的消息，而 `pipe` 中却可能没有消息，也就是说，调用 `ZwQueryInformationFile` 的线程实际上永久性地挂起了。所以命名文件的获取不用在挑选句柄的主线程中进行，可以启动独立的线程并设置一个 `timeout` 值来避免挂起。第二，在拷贝句柄后，两个句柄（我们进程的和打开文件进程的）将会指向同一个 `FileObject`，从而当前的输入输出模式、在文件中的位置以及其它与文件相关的信息就会由两个进程来共享。这时，甚至只是读取文件都会引起读取位置的改变，从而破坏了打开文件程序的正常运行。为了避免这种情形，我们需要需要停止占用文件进程的线程、保存当前位置、拷贝文件、恢复当前位置以及重新启动占用文件的进程。这种方法不能用于许多情形，比如要在运行的系统中拷贝注册表文件，用这种方法就不会成功。

我们先来试着实现对系统中所有已打开文件的句柄的枚举。为枚举句柄，每个句柄都由以下结构体描述：

```c
typedef struct _SYSTEM_HANDLE
{
  ULONG       uIdProcess;
  UCHAR       ObjectType;
  UCHAR       Flags;
  USHORT      Handle;
  POBJECT     pObject;
  ACCESS_MASK GrantedAccess;
} SYSTEM_HANDLE, *PSYSTEM_HANDLE;
```

这里的 `ObjectType` 域定义了句柄所属的对象类型。这里我们又遇到了问题——`File` 类型的 `ObjectType` 在 `Windows 2000`、`XP` 和 `2003` 下的取值各不相同，所以我们不得不动态的定义这个值。为此我们用 `CreateFile` 来打开 `NUL` 设备，找到它的句柄并记下它的类型：

```c
UCHAR GetFileHandleType()
{
  HANDLE                   hFile;
  PSYSTEM_HANDLE_INFORMATION Info;
  ULONG                    r;
  UCHAR                    Result = 0;

  hFile = CreateFile("NUL", GENERIC_READ, 0, NULL, OPEN_EXISTING, 0, 0);

  if (hFile != INVALID_HANDLE_VALUE)
  {
    Info = GetInfoTable(SystemHandleInformation);

    if (Info)
    {
      for (r = 0; r < Info->uCount; r++)
      {
        if (Info->aSH[r].Handle == (USHORT)hFile &&
                        Info->aSH[r].uIdProcess == GetCurrentProcessId())
        {
          Result = Info->aSH[r].ObjectType;
          break;
        }
      }

      HeapFree(hHeap, 0, Info);
    }

    CloseHandle(hFile);
  }
  return Result;
}
```

现在知道了句柄的类型我们就可以枚举系统中打开的文件了。首先我们来用句柄获取打开文件的文件名：

```c
typedef struct _NM_INFO
{
  HANDLE   hFile;
  FILE_NAME_INFORMATION Info;
  WCHAR    Name[MAX_PATH];
} NM_INFO, *PNM_INFO;

DWORD WINAPI
  GetFileNameThread(PVOID lpParameter)

{
  PNM_INFO         NmInfo = lpParameter;
  IO_STATUS_BLOCK  IoStatus;
  int r;

  NtQueryInformationFile(NmInfo->hFile, &IoStatus, &NmInfo->Info,
                    sizeof(NM_INFO) - sizeof(HANDLE), FileNameInformation);

  return 0;
}

void GetFileName(HANDLE hFile, PCHAR TheName)
{
  HANDLE   hThread;
  PNM_INFO Info = HeapAlloc(hHeap, 0, sizeof(NM_INFO));

  Info->hFile = hFile;

  hThread = CreateThread(NULL, 0, GetFileNameThread, Info, 0, NULL);

  if (WaitForSingleObject(hThread, INFINITE) == WAIT_TIMEOUT) TerminateThread(hThread, 0);

  CloseHandle(hThread);

  memset(TheName, 0, MAX_PATH);

  WideCharToMultiByte(CP_ACP, 0, Info->Info.FileName, Info->Info.FileNameLength >> 1, TheName, MAX_PATH, NULL, NULL);

  HeapFree(hHeap, 0, Info);
}
```

现在来枚举打开的文件：

```c
void main()
{
  PSYSTEM_HANDLE_INFORMATION Info;
  ULONG                      r;
  CHAR                       Name[MAX_PATH];
  HANDLE                     hProcess, hFile;

  hHeap = GetProcessHeap();

  ObFileType = GetFileHandleType();

  Info = GetInfoTable(SystemHandleInformation);

  if (Info)
  {
    for (r = 0; r < Info->uCount; r++)
    {
      if (Info->aSH[r].ObjectType == ObFileType)
      {
        hProcess = OpenProcess(PROCESS_DUP_HANDLE, FALSE, Info->aSH[r].uIdProcess);

        if (hProcess)
        {
          if (DuplicateHandle(hProcess, (HANDLE)Info->aSH[r].Handle,
                  GetCurrentProcess(), &hFile, 0, FALSE, DUPLICATE_SAME_ACCESS))
          {
            GetFileName(hFile, Name);

            printf("%s\n", Name);

            CloseHandle(hFile);
          }

          CloseHandle(hProcess);
        }
      }

    }
    HeapFree(hHeap, 0, Info);
  }
}
```

现在对于文件的拷贝我们剩下的工作只是找到所需句柄后用 `ReadFile` 读取它。这里一定要使用前面提到的机制，不可疏忽。

这种方法的优点是实现简单，但是其缺点更多，所以这个方法只适用于确定文件被那个进程占用。

### 修改句柄访问权限

所有被占用的文件通常都可以用读属性（`FILE_READ_ATTRIBUTES`）打开，这样就可以读取文件的属性，取得它的大小，枚举 `NTSF stream`，但遗憾的是，`ReadFile` 就不能成功调用了。打开文件时各种访问属性的区别在哪里呢？显然，打开文件时，系统会记录访问属性，之后会用这个属性与请求的访问作比较。如果找到了系统保存这个属性的位置并修该掉它，那就不只可以读取，甚至可以写入任何已打开的文件。

在用户这一级别上我们并不是直接与文件打交道，而是通过它的句柄（这个句柄指向 `FileObject`），而函数 `ReadFile/WriteFile` 调用 `ObReferenceObjectByHandle`，并指明了相应的访问类型。由此我们可以得出结论，访问权限保存在描述句柄的结构体里。实际上，`HANDLE_TABLE_ENTRY` 结构体包含有一个 `GrantedAccess` 域，这个域不是别的，就是句柄的访问权限。遗憾的是，`Microsoft` 的程序员们没有提供修改句柄访问权的 API，所以我们不得不编写驱动自己来做这项工作。

我在《隐藏进程检测》一文中讲到过 `Windows 2000` 和 `XP` 的句柄表结构体，我想补充的只有一点，就是 `Windows 2003` 中的句柄表与 `XP` 的完全一样。与那篇文章不同，我们这里不需要枚举表中的句柄，而只需要找到某个具体的（已知的）句柄，我们不用管 `PspCidTable`，而只操作自己进程的句柄表，表的指针位于进程的 `EPROCESS` 结构体里（`2000` 下的偏移为 `0x128`，`XP` 下的为 `0x0C4`）。

为了取得句柄结构体指针需要调用未导出函数 `ExpLookupHandleTableEntry`，但我们不会去搜索它，因为在导出函数中没有对它的直接引用，搜索结果也很不可靠，除此之外我们此时还需要 `ExUnlockHandleTableEntry` 函数。最好的办法就是编写自己的句柄表 lookup 函数。考虑到 `Windows 2000` 与 `XP` 下句柄表的差异，我们将编写不同的函数。

首先是 `Windows 2000` 下的：

```c
PHANDLE_TABLE_ENTRY
  Win2kLookupHandleTableEntry(
      IN PWIN2K_HANDLE_TABLE HandleTable,
      IN EXHANDLE            Handle
      )
{
    ULONG i, j, k;

    i = (Handle.Index >> 16) & 255;
    j = (Handle.Index >> 8)  & 255;
    k = (Handle.Index)       & 255;

    if (HandleTable->Table[i])
    {
        if (HandleTable->Table[i][j])
        {
            return &(HandleTable->Table[i][j][k]);
        }
    }
    return NULL;
}
```

这段代码简单易懂。因为句柄的值本身是个三维表的三个索引，所以我们只需其中的各个部分并查看表中相应的元素（当然如果存在的话）。因为 `Windows XP` 中的句柄表可以有一到三个级别，所以相应的 lookup 代码就要更为复杂一些：

```c
PHANDLE_TABLE_ENTRY
  XpLookupHandleTableEntry(
              IN PXP_HANDLE_TABLE HandleTable,
              IN EXHANDLE          Handle
              )
{
    ULONG i, j, k;
    PHANDLE_TABLE_ENTRY Entry = NULL;
    ULONG TableCode = HandleTable->TableCode & ~TABLE_LEVEL_MASK;

    i = (Handle.Index >> 17) & 0x1FF;
    j = (Handle.Index >> 9)  & 0x1FF;
    k = (Handle.Index)       & 0x1FF;

    switch (HandleTable->TableCode & TABLE_LEVEL_MASK)
    {
        case 0 :
          Entry = &((PHANDLE_TABLE_ENTRY)TableCode)[k];
        break;

        case 1 :
          if (((PVOID *)TableCode)[j])
          {
             Entry = &((PHANDLE_TABLE_ENTRY *)TableCode)[j][k];
          }
        break;

        case 2 :
          if (((PVOID *)TableCode)[i])
          if (((PVOID **)TableCode)[i][j])
          {
             Entry = &((PHANDLE_TABLE_ENTRY **)TableCode)[i][j][k];
          }
        break;
    }
    return Entry;
}
```

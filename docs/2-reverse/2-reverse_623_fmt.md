
# 猪头三 Winapiex DISK 接口分析

> 原文：[`www.pediy.com/kssd/pediy10/69003.html`](https://www.pediy.com/kssd/pediy10/69003.html)

## 文章信息
- **标题**：猪头三 Winapiex DISK 接口分析
- **作者**：`nevergone`
- **邮箱**：`wangyongxina2004@163.com`
- **下载地址**：自己搜索下载
- **编写语言**：`VC8`
- **使用工具**：`IDA` `VC8`
- **作者声明**：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 详细过程

猪头三大哥的 `winapiex` 项目中，整合了一系列的接口，包含磁盘操作类、文件操作类、进程操作类等。

我对磁盘操作类比较感兴趣，因此下载了 DLL 文件，并逆向了磁盘操作类的全部接口：

- `E_Disk_IsDynamicDisk` [作者: 猪头三 性质: 闭源 修改时间: 2007-06-18 功能: 判断物理磁盘是否是动态磁盘]
- `E_Disk_ReadData` [作者: 猪头三 性质: 授权 修改时间: 2007-06-18 功能: 读取物理磁盘数据]
- `E_Disk_WriteData` [作者: 猪头三 性质: 授权 修改时间: 2007-06-18 功能: 写入物理磁盘数据]
- `E_Disk_GetDiskGeometry` [作者: 猪头三 性质: 授权 修改时间: 2007-06-18 功能: 获取物理磁盘基本信息参数]
- `E_Disk_GetPartitionType` [作者: 猪头三 性质: 授权 修改时间: 2007-06-18 功能: 获取指定分区类型]
- `E_Disk_GetDiskNumberByFilePath` [作者: 猪头三 性质: 授权 修改时间: 2007-06-19 功能: 获取指定分区所在的磁盘]
- `E_Disk_GetPartitionInfoByFilePath` [作者: 猪头三 性质: 授权 修改时间: 2007-06-19 功能: 获取指定分区基本信息(分区类型 分区大小 分区所在磁盘起始位置)]
- `E_Disk_GetPartitionSpace` [作者: 猪头三 性质: 授权 修改时间: 2007-06-19 功能: 获取指定分区实际空闲容量和实际总容量]
- `E_Disk_GetDiskNumber` [作者: 猪头三 性质: 授权 修改时间: 2007-06-21 功能: 获取物理磁盘数量]
- `E_Disk_FormatPartition` [作者: 猪头三 性质: 授权 修改时间: 2007-06-22 功能: 格式化指定分区]

下面我将介绍其中的几个接口。

### `E_Disk_FormatPartition` 分析

我逆向的第一个接口是：`E_Disk_FormatPartition`。

按猪头三提供的文档，此函数用于格式化指定分区。函数内部调用 `SHFormatDrive` 来执行格式化。

值得注意的是这个函数存在一个 BUG，看 IDA：

```asm
; int __cdecl E_Disk_FormatPartition(const char *pDriverName)
.text:100013E0                     public E_Disk_FormatPartition
.text:100013E0 E_Disk_FormatPartition proc near
.text:100013E0
.text:100013E0 pDriverName        = dword ptr  4
.text:100013E0
.text:100013E0                     mov     eax, [esp+pDriverName]
.text:100013E4                     mov     dl, [eax]
.text:100013E6                     xor     ecx, ecx
.text:100013E8                     mov     eax, A        ; 'A'
.text:100013ED                     movsx   edx, dl       ; Get pDriverName[0]
.text:100013ED
.text:100013F0
.text:100013F0 begin_loop:                            ; CODE XREF: E_Disk_FormatPartition+1Dj
.text:100013F0                     cmp     edx, eax
.text:100013F2                     jz      short Do_Format ; 循环,用于取得 SHFormatDrive 的第二个参数
.text:100013F2
.text:100013F4                     add     eax, 1
.text:100013F7                     add     ecx, 1
.text:100013FA                     cmp     eax, Z        ; 'Z'
.text:100013FD                     jbe     short begin_loop;比'Z'小时,跳转,当比'Z'大于循环不满足,执行 Do_Format.
.text:100013FD
.text:100013FF
.text:100013FF Do_Format:                                ; CODE XREF: E_Disk_FormatPartition+12j
.text:100013FF                     push    SHFMT_OPT_FULL  ; options
.text:10001401                     push    SHFMT_ID_DEFAULT ; fmtID
.text:10001406                     push    ecx            ; drive
.text:10001407                     call    ds:GetActiveWindow
.text:10001407
.text:1000140D                     push    eax            ; hwnd
.text:1000140E                     call    ds:SHFormatDrive
.text:1000140E
.text:10001414                     retn
```

当用户构造一个特殊的字符串 `pDriverName[0]` > 'Z'时，会把用户的 Z 盘直接格式化，应该在函数入口检测用户输入参数是否合法。

### `E_Disk_ReadData` 与 `E_Disk_WriteData` 分析

`disk` 接口中比较有用的两个接口是 `E_Disk_ReadData` 和 `E_Disk_WriteData`。

这两个接口按猪头三提供的文档：读取物理磁盘数据和写入物理磁盘数据。

下面的代码是逆向后的：

```cpp
bool __cdecl MyReadData(IN unsigned long ulong_param_DiskNum,
           IN const char *pchar_param_DriverName,
           IN void *pvoid_param_DataBuffer,
           IN unsigned long ulong_param_DataLength,
           IN LONGLONG llg_param_ByteOffset)
{
  char szBuffer[0x20] = { 0 };
  ULONG ulDiskNumber = 0;
  if (lstrlen(pchar_param_DriverName) > 1)
    ulDiskNumber = MyGetDiskNumberByFilePath(pchar_param_DriverName);
  else
    ulDiskNumber = ulong_param_DiskNum;
  wsprintf(szBuffer, "\\\\.\\PhysicalDrive%ld", ulDiskNumber);
  HANDLE hFile = CreateFile(szBuffer,
              GENERIC_READ,
              FILE_SHARE_READ | FILE_SHARE_WRITE,
              NULL,
              OPEN_EXISTING,
              FILE_FLAG_WRITE_THROUGH | FILE_FLAG_NO_BUFFERING,
              NULL);
  if (hFile == INVALID_HANDLE_VALUE)
  {
    MessageBox(NULL, "E_ReadPhysicalDriveData()->OpenPhysicalDrive is failed !", "", MB_OK);
    return false;
  }
  BOOL fOk = ReadFileInternal(hFile,
              pvoid_param_DataBuffer,
              ulong_param_DataLength,
              (LONG)llg_param_ByteOffset,
              *(PLONG)((PBYTE)(&llg_param_ByteOffset) + 4));
  if (!fOk)
  {
    MessageBox(NULL, "E_ReadPhysicalDriveData()->ReadPhysicalDriveData is failed !", "", MB_OK);
  }
  CloseHandle(hFile);
  return (bool)fOk;
}
```

首先通过参数 `ulong_param_DiskNum` 或 `pchar_param_DriverName` 取得欲写入的硬盘编号。

在文档中：`pchar_param_DriverName` 注明必须是 0，但从 IDA 分析中，如果 `pchar_param_DriverName` 不为 0，则通过一个内部函数来取得硬盘编号，在我的 CPP 文件中，我用 `MyGetDiskNumberByFilePath(pchar_param_DriverName)`；取得了硬盘编号后，构造一个 `\\.\PhysicalDrive%ld` 的字符串，调用 `CreateFile` 打开这个对象，最后再调用内部未导出的函数来执行读取操作，这个函数在我的 CPP 文件中我命名为 `ReadFileInternal`。

```cpp
int __fastcall ReadFileInternal(HANDLE hFile,
            PVOID pvBuffer,
            DWORD long_param_nNumberOfBytesToRead,
            LONG long_param_lDistanceToMove,
            LONG long_param_lDistanceToMoveHigh)
{
  DWORD dwNumberBytesRead = 0;
  LONG lDistanceToMove;
  LONG lDistanceToMoveHigh;
  char szBuffer[MAX_PATH];
  ZeroMemory(szBuffer, MAX_PATH * sizeof(char));
  lDistanceToMoveHigh = long_param_lDistanceToMoveHigh;
  lDistanceToMove = long_param_lDistanceToMove;
  DWORD dwPtrLow = SetFilePointer(hFile,
            lDistanceToMove,
            &lDistanceToMoveHigh,
            FILE_BEGIN);
  if (dwPtrLow == INVALID_SET_FILE_POINTER)
  {
    ReportError();
    return 0;
  }
  BOOL fOk = ReadFile(hFile,
      pvBuffer,
      long_param_nNumberOfBytesToRead,
      &dwNumberBytesRead,
      NULL);
  if (!fOk)
  {
    wsprintf(szBuffer, "Read PhysicalDriveData Error %ld !", GetLastError());
    MessageBox(NULL, szBuffer, "", MB_OK);
  }
  return fOk;
}
```

先调用 `SetFilePointer` 使得文件指针指向用户要求的区域，然后再调用 `ReadFile` 来执行操作。

原来挺简单的。

`E_Disk_WriteData` 的流程和 `E_Disk_ReadDisk` 差不多，请参看源码。

这两个函数可以用来读取硬盘的 MBR。

```cpp
PVOID  pvBuffer1 = HeapAlloc(GetProcessHeap(), HEAP_ZERO_MEMORY, 512);
E_Disk_ReadData(0, NULL, pvBuffer1, 512, 0);
```

这样就取得了 MBR。

写入 MBR 的代码有点危险....嘿嘿...

### `E_Disk_IsDynamicDisk` 分析

接下来是用来判断物理磁盘是否是动态磁盘的接口

```cpp
unsigned int MyIsDynamicDisk(IN const char *pchar_param_FilePath)
{
  DWORD dwBytesReturned = 0;
  char szBuffer[0x20] = { 0 };
  UINT uiResult = 1;
  ULONG ulDiskNumber = MyGetDiskNumberByFilePath(pchar_param_FilePath);
  if (ulDiskNumber == 0xFF)
  {
    MessageBox(NULL, "IsDynamicDisk()->Get disk number is failed !", "", MB_OK);
    return 0;
  }
  wsprintf(szBuffer, "\\\\.\\PhysicalDrive%ld", ulDiskNumber);
  HANDLE hFile = CreateFile(szBuffer,
              GENERIC_READ,
              FILE_SHARE_READ | FILE_SHARE_WRITE,
              NULL,
              OPEN_EXISTING,
              FILE_FLAG_WRITE_THROUGH |  FILE_FLAG_NO_BUFFERING,
              NULL);
  if (hFile == INVALID_HANDLE_VALUE)
}
```



```cpp
{  
  MessageBox(NULL, "IsDynamicDisk()->OpenPhysicalDrive() is failed !", "", 0);  
  return 0;  
}  
DeviceIoControl(hFile,  
    FSCTL_LOCK_VOLUME,  
    NULL,  
    0,  
    NULL,  
    0,  
    &dwBytesReturned,  
    NULL);  
LPVOID pvBuffer = malloc(0x200);  
ZeroMemory(pvBuffer, 0x200);  
BOOL fOk = ReadFileInternal(hFile, pvBuffer, 0x200, 0, 0);  
if (!fOk)  
{  
  MessageBox(NULL, "IsDynamicDisk()->ReadPhysicalDriveData() is failed !", "", MB_OK);  
  uiResult = 0;  
  goto __Exit;  
}  
DWORD dwUnknown = *(LPDWORD)((LPBYTE)pvBuffer + 0x1c2);  

#if _DEBUG         // just for test  
BYTE  aByte;  
TCHAR  Buffer[20];  
aByte = *static_cast<LPBYTE>((LPBYTE)pvBuffer + 0x1c2);  
wsprintf(Buffer, "%x", aByte);  
OutputDebugStringA(Buffer);  
#endif  
if (dwUnknown == 0x42)  
{  
  MessageBox(NULL, "This disk is Dynamic Disk !\n", "", MB_OK);  
  uiResult = 3;  
  goto __Exit;  
}  
__Exit:  
DeviceIoControl(hFile,  
    FSCTL_UNLOCK_VOLUME,  
    NULL,  
    0,  
    NULL,   
    0,  
    &dwBytesReturned,  
    NULL);  
CloseHandle(hFile);  
free(pvBuffer);  
return uiResult;  

}  
```

这个函数是整个`DISK`接口中,代码最长的接口

先根据`pchar_param_FilePath`取得硬盘编号.`DLL`通过一个未导出函数来实现,在我的`CPP`文件中为`MyGetDiskNumberByFilePath`

```cpp
/* #define IOCTL_DISK_UNKNOWN 0x560000 */
DWORD  dwBytesReturned = 0;  
/*char  szOutBuffer[0x20] = { 0 };*/  
VOLUME_DISK_EXTENTS disk_extents;  
char  szDest[0x20] = { 0 };  
wsprintfA(szDest, "\\\\?\\%c:", pchar_param_FilePath[0]);  
HANDLE hFile = CreateFile(szDest,  
          GENERIC_READ,  
          FILE_SHARE_READ | FILE_SHARE_WRITE,  
          NULL,  
          OPEN_EXISTING,  
          0,  
          NULL);  
if (hFile == INVALID_HANDLE_VALUE)  
{  
  ReportError();  
  return 0xFF;  
}  
BOOL fOk = DeviceIoControl(hFile, IOCTL_VOLUME_GET_VOLUME_DISK_EXTENTS, NULL, 0, &disk_extents, sizeof(disk_extents), &dwBytesReturned,  
NULL);  
CloseHandle(hFile);  
return fOk ? disk_extents.Extents[0].DiskNumber : 0xFF;  
```

一开始我不知道`DeviceIoControl`函数参数`dwIoControlCode==560000h`时,是哪个`IOCTL`

只能硬编码.后来通过查找`MSDN`,找到是`IOCTL_VOLUME_GET_VOLUME_DISK_EXTENTS`这个`IOCTL`,一下解决这个问题.

取得了硬盘编号后,调用`CreateFile`来创建对象.然后调用`DeviceIoControl`以`FSCTL_LOCK_VOLUME`锁住.

申请内存调用`ReadFileInternal`来读取头`200h`字节的内容,取偏移为`0x1c2`处的双字,如果此值为`0x42`,则此磁盘是动态

磁盘.我猜想`200h`可能是一个结构,`0x1c2`是结构中的一个偏移.

其他接口相对简单,不再多说,请参看源码.

我的工程中包含了测试代码.

--------------------------------------------------------------------------------

### 【经验总结】

引用一句牛人`xIkug`的话:代码逆向即是在没有源代码的情况下，对目标程序的行为、数据流、及编译器生成的代码进行分析，通

过分析我们可以了解、发现程序的功能、流程、规则、及技术实现细节等，通过分析我们能对其进行优化、功能增强、漏洞

填补、甚至还原成源代码等。这个分析过程我们可以称作逆向分析或逆向工程，简称逆向。

偶觉得很中肯.

--------------------------------------------------------------------------------

### 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2008 年 07 月 21 日 20:53:59

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) |
| TestWinApiEx.rar [附件请到论坛下载：[`bbs.pediy.com/showthread.php?t=69003`](http://bbs.pediy.com/showthread.php?t=69003%20) ] |

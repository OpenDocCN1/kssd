
在最后的`doCabinetInfoNotify`函数中，程序调用了`urlmon!fdiNotifyExtract`函数（`if ( (this[9])(0, this + 0x1EF) != -1 )`），这个函数在之后还会调用到，它其实进行了主要的cab解压后的文件创建工作，不过在此次调用中并没有进行相关工作。

从IDA中得到函数声明：`int __cdecl fdiNotifyExtract(enum FDINOTIFICATIONTYPE a1, struct FDINOTIFICATION *a2);`

因此这里的调用中，`FDINOTIFICATIONTYPE`参数的数值是0，根据文件`fdi.h`（这个文件你应该可以在自己的操作系统里面搜索到）：

```c
typedef enum {
    fdintCABINET_INFO,       // General information about cabinet
    fdintPARTIAL_FILE,       // First file in cabinet is continuation
    fdintCOPY_FILE,          // File to be copied
    fdintCLOSE_FILE_INFO,    // close the file, set relevant info
    fdintNEXT_CABINET,       // File continued to next cabinet
    fdintENUMERATE,          // Enumeration status
} FDINOTIFICATIONTYPE; /* fdint */
```

> **fdintCABINET_INFO:**
>
> Called exactly once for each cabinet opened by `FDICopy()`, including continuation cabinets opened due to file(s) spanning cabinet boundaries. Primarily intended to permit `EXTRACT.EXE` to automatically select the next cabinet in a cabinet sequence even if not copying files that span cabinet boundaries.

从IDA中代码中看，如果`FDINOTIFICATIONTYPE`参数的数值是0，`fdiNotifyExtract`其实什么都没干：

```c
if ( a1 == fdintCABINET_INFO || a1 == fdintPARTIAL_FILE )
    return 0;
```

继续往下看，`FDICopy`函数调用了`FDICallEnumerate`，这个函数再次调用了`urlmon!fdiNotifyExtract`函数（`if ( v3(5, this + 0x7BC) != -1 )`），这次`FDINOTIFICATIONTYPE`参数的数值是5，对应`fdintENUMERATE`，从IDA代码看，`fdiNotifyExtract`依旧没有执行什么操作。

之后`FDICopy`函数调用了`FDIReadCFFILEEntry`函数，这个函数读入了`CFFILE`的前16个字节，不包含最后的`szName`字段，之后又读入了后面的`0x100`字节的数据。根据`iFolder`字段判断当前文件所处文件夹索引：

![图片描述](https://bbs.kanxue.com/upload/attach/202110/600394_7A4DZGSNRSF23Q6.png)

之后`FDICopy`函数第三次调用`urlmon!fdiNotifyExtract`函数，这次的`FDINOTIFICATIONTYPE`参数的数值是2，对应`fdintCOPY_FILE`。

经过windbg调试，发现程序调用了`catDirAndFile`函数，形成了解压缩后文件的完整路径`C:\Users\zoem\AppData\Local\Temp\Low\Cab425A\../msword.inf`:

```text
0:009> p
eax=07e2b3ac ebx=07e2b2a8 ecx=07e2b3ac edx=00000104 esi=0c27bf34 edi=07e2b28c
eip=71e8aa7c esp=07e2af9c ebp=07e2b0bc iopl=0         nv up ei pl zr na pe nc
cs=0023  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00000246
urlmon!fdiNotifyExtract+0xdc:
71e8aa7c e866030000      call    urlmon!catDirAndFile (71e8ade7)
0:009> p
eax=00000001 ebx=07e2b2a8 ecx=336b59e5 edx=07e2ad89 esi=0c27bf34 edi=07e2b28c
eip=71e8aa81 esp=07e2afa4 ebp=07e2b0bc iopl=0         nv up ei pl zr na pe nc
cs=0023  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00000246
urlmon!fdiNotifyExtract+0xe1:
71e8aa81 85c0            test    eax,eax
0:009> da 7e2b3ac
07e2b3ac  "C:\Users\zoem\AppData\Local\Temp"
07e2b3cc  "\Low\Cab425A\../msword.inf"
```

之后程序会调用`AddFile`和`NeedFile`函数，经过`NeedFile`的判断，程序从`urlmon!fdiNotifyExtract`函数返回，返回值为0。

在`fdi.h`中说，如果返回值为0，表示跳过文件不复制。但是如果仔细查看该文件，在关于`FDINOTIFICATIONTYPE`的注释说明部分，提供了一个在进行`FDICopy`过程中，`FDINOTIFICATIONTYPE`取值的典型变化情况：

```text
A typical sequence of calls will be something like this:
fdintCABINET_INFO    // Info about the cabinet
fdintENUMERATE       // Starting enumeration
fdintPARTIAL_FILE    // Only if this is not the first cabinet, and
                     // one or more files were continued from the
                     // previous cabinet.
...
fdintPARTIAL_FILE
fdintCOPY_FILE       // The first file that starts in this cabinet
...
fdintCOPY_FILE       // Now let's assume you want this file...
                     // PFNWRITE called multiple times to write to this file.
fdintCLOSE_FILE_INFO // File done, set date/time/attributes

fdintCOPY_FILE       // Now let's assume you want this file...
                     // PFNWRITE called multiple times to write to this file.
fdintNEXT_CABINET    // File was continued to next cabinet!
fdintCABINET_INFO    // Info about the new cabinet
                     // PFNWRITE called multiple times to write to this file.
fdintCLOSE_FILE_INFO // File done, set date/time/attributes
...
fdintENUMERATE       // Ending enumeration
```

其中`fdintPARTIAL_FILE`针对的是同一文件存在于多个cab文件的情况，这里不考虑。从上面的流程可以看出，`fdintCOPY_FILE`参数传入了两次，如果当前文件是cab中的第一个文件，会传入一次`fdintCOPY_FILE`，也就是此时的情况，程序并不会进行真正的解压缩行为。

那么在这次传入`fdintCOPY_FILE`的时候，程序干了些什么呢？其实这里我没有完全弄清楚，它涉及到一个数据结构，但是我没有找到详细的文档介绍，如果从头开始说的话，要回到`ExtractInfFile`函数的开始部分，上面已经贴了这个函数的伪代码，在调用`Extract`函数之前，程序调用了`StringCchCopyA`函数（`StringCchCopyA(0x104u, a3 + 0x1C, a1, v11, v14);`），这个函数实际上就是把cab解压的目标目录复制到了`a3 + 0x1C`这个位置。而在这个语句之前，有这样两句代码：

```c
*((_DWORD *)a3 + 6) = 1;      // 注意a3指向DWORD，所以这里偏移值是0x18h
*((_DWORD *)a3 + 0xCA) = 0;   // 偏移值0x328，注意这里，后面NeedFile会用到
```

`*(a3+6)`这里应该类似于一个标志位。

在第一次执行到`AddFile`的时候（也就是第一次传入`fdintCOPY_FILE`的时候），函数代码如下：

```c
BOOL __userpurge AddFile@<eax>(unsigned int a1@<ecx>, size_t cchDest@<edx>, unsigned int a3@<esi>, int a4)
{
    // [COLLAPSED LOCAL DECLARATIONS. PRESS KEYPAD CTRL-"+" TO EXPAND]

    if ( ( *(a1 + 0x18) & 1 ) == 0 ) // 这里a1指向byte，所以偏移值也是0x18h，是标志位
        return 1;
    v7 = CoTaskMemAlloc(0xCu); // 分配了一个大小为0xC的空间，下面的注释对这个空间进行解释
    if ... // 分配失败
    v8 = CoTaskMemAlloc(strlen(cchDest) + 1); // 为解压目标文件名分配空间
    *v7 = v8; // 0xC空间首位保存解压目标文件名
    if ... // 分配失败
    v7[2] = 1; // 0xC空间第三位数值为1 ！！！这里下面会提到！！！
    (StringCchCopyA)(cchDest, a3, v11);
    v7[1] = *(a1 + 0x10); // 0xC空间第二位不知道保存的什么，但是和下面两句一起看
    ++*(a1 + 0x14); // 这里像是链表节点数量递增
    *(a1 + 0x10) = v7; // 看起来像是将这个新分配的0xC空间链入了一个链表
    return (UIntAdd)(a1, v10, v12) >= 0; // 这里从调试结果看像是统计文件总解压后大小
}
```

根据上面的注释，`AddFile`这个函数应该是将程序当前查看的解压文件添加到a1指向的一个结构体中，`0xC`的空间就保存了当前解压文件的文件名信息，以及链表的下一个节点，还有一个标志位（3.3.6小结会提到这个标志位的作用）。

在这里贴出所谓的a1指向的结构体是什么样子的（我不确定大小是不是正确的，或许包含了之后非结构体的数据）：

```text
07e2b28c 00 5c 41 00 00 00 00 00 00 00 00 00 00 00 00 00  .\A.............
07e2b29c 90 7c 92 08 01 00 00 00 01 00 00 00 43 3a 5c 55  .|..........C:\U
07e2b2ac 73 65 72 73 5c 7a 6f 65 6d 5c 41 70 70 44 61 74  sers\zoem\AppDat
07e2b2bc 61 5c 4c 6f 63 61 6c 5c 54 65 6d 70 5c 4c 6f 77  a\Local\Temp\Low
07e2b2cc 5c 43 61 62 34 32 35 41 00 00 00 00 00 00 00 00  \Cab425A........
```

注意到首四个字节就是解压后大小`0x00415c00`，偏移`0x10`的位置指向了分配的`0xC`大小的空间。

`AddFile`执行完之后，程序调用`NeedFile`：

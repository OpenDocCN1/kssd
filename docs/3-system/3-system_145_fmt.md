
# PE 文件基础补注 (VC + 纯 API 版)

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-725.htm`](https://www.pediy.com/kssd/pediy08/pediy8-725.htm)

PE 文件基础补注

关键词： PE 文件 地址转换 IAT IMAGE_IMPORT_BY_NAME

前言: 最近学习 PE, 略有心得, 拿来和大家分享.

感谢: 小虾斑斑, 非安全 ,Bookworm 对我的帮助.

附件：`pe.rar`

## 1. IMAGE_SECTION_HEADER 小结

### 1.1 获得节表数

`NumberOfSections = NtHeader->FileHeader.NumberOfSections;`

### 1.2 节表获得方法

方法 1. 因为 NT 头之后就是节表，故，节表头地址就是 nt 头地址加上 NT 结构大小．

```
SectionHeader=(PIMAGE_SECTION_HEADER)((UINT32)NtHeader+(UINT32)(sizeof(IMAGE_NT_HEADERS)));
```

方法 2. 或者用 `ImageBase+SizeOfHeaders` 的办法直接定位．

```
SectionHeader=(PIMAGE_SECTION_HEADER)((UINT32)(NtHeader->OptionalHeader.ImageBase)+
              (UINT32)(NtHeader->OptionalHeader.SizeOfHeaders));
```

方法 3. 既然节都是连在一起的,那么,也就可以这样:

```
SectionHeader= (PIMAGE_SECTION_HEADER) (NtHeader + 1),
```

方法 4. 论坛里面 `hmimys` 告诉的办法:

```
SectionHeader=(PIMAGE_SECTION_HEADER)((UINT32)NtHeader+0x18+
                 (UINT32)(NtHeader->FileHeader.SizeOfOptionalHeader));
```

到现在我还没有弄懂为什么 `hmimys` 说最好要用方法 4 而不用方法 3.

## 2. IMAGE_IMPORT_DESCRIPTOR 小结

### 2.1 获得引入表结构起始地址

方法 1：

```
ImportDec = (PIMAGE_IMPORT_DESCRIPTOR)(NtHeader->OptionalHeader.DataDirectory[12].VirtualAddress);
```

这个方法我觉得理论上是对的,但是我在运行的时候总是得不到正确的地址.后来知道,似乎不能用 `'12'`,而要用 `IMAGE_DIRECTORY_ENTRY_IAT` 这个宏

方法 2：

```
ImportDes = (PIMAGE_IMPORT_DESCRIPTOR)((DWORD)(NtHeader->OptionalHeader.DataDirectory)+
                (DWORD)(sizeof(IMAGE_DATA_DIRECTORY)*12));
```

方法 3 :

```
ImportDes = (PIMAGE_IMPORT_DESCRIPTOR)(NtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress-
        Offset + (PBYTE)pMapping);
```

注: 前两种方法都是从 IAT 中得出 `IMAGE_IMPORT_DESCRIPTOR`,而后面的那个是 非安全 大哥教的. 这里有个疑问:

3 种方法都可以得到 `IMAGE_IMPORT_DESCRIPTOR` 结构,都可以得到函数名, 区别在于前两种方法枚举的函数名不全.

难道说两个结构都指向同一个结构 `PIMAGE_IMPORT_DESCRIPTOR`?

### 2.2 IMAGE_IMPORT_DESCRIPTOR 结构既不是在 Import Symbols 中，也不是在 IAT (IMAGE_IMPORT_ADDRESS_TABLE)中。它就是一个结构.

我原来说:"`IMAGE_IMPORT_DESCRIPTOR` 结构不是在 Import Symbols 中，是在 IAT (`IMAGE_IMPORT_ADDRESS_TABLE`)中。" 有问题.

就是因为这个错误的理解, 让我走了好多死路.

这个是 `Winnt.h` 中关于 `IMAGE_SYMBOL` 的结构信息

```
typedef struct _IMAGE_SYMBOL {
 union {
  BYTE    ShortName[8];
  struct {
   DWORD   Short;     // if 0, use LongName
   DWORD   Long;      // offset into string table
  } Name;
  PBYTE   LongName[2];
 } N;
 DWORD   Value;
 SHORT   SectionNumber;
 WORD    Type;
 BYTE    StorageClass;
 BYTE    NumberOfAuxSymbols;
 } IMAGE_SYMBOL;
 typedef IMAGE_SYMBOL UNALIGNED *PIMAGE_SYMBOL;
```

而下面的是 IAT:

```
typedef struct _IMAGE_IMPORT_BY_NAME {
 WORD    Hint;
 BYTE    Name[1];
} IMAGE_IMPORT_BY_NAME, *PIMAGE_IMPORT_BY_NAME;

typedef struct _IMAGE_IMPORT_DESCRIPTOR {
 union {
  DWORD   Characteristics;          // 0 for terminating null import descriptor
  DWORD   OriginalFirstThunk;       // RVA to original unbound IAT (PIMAGE_THUNK_DATA)
 };
 DWORD   TimeDateStamp;            // 0 if not bound,
                                   // -1 if bound, and real date\\time stamp
                                   //     in IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT (new BIND)
                                   // O.W. date/time stamp of DLL bound to (Old BIND)
 DWORD   ForwarderChain;           // -1 if no forwarders
 DWORD   Name;
 DWORD   FirstThunk;             // RVA to IAT (if bound this IAT has actual addresses)
} IMAGE_IMPORT_DESCRIPTOR;
 typedef IMAGE_IMPORT_DESCRIPTOR UNALIGNED *PIMAGE_IMPORT_DESCRIPTOR;
```

`_IMAGE_IMPORT_DESCRIPTOR` 结构联合中的 `OriginalFirstThunk` , 就是到 `IMAGE_THUNK_DATA` 的 `RVA`.

如果像下面这样写,也许更明白

```
typedef struct _IMAGE_THUNK_DATA {
  union {
   PBYTE ForwarderString;
   PDWORD Function;
   DWORD Ordinal;
   PIMAGE_IMPORT_BY_NAME AddressOfData;
  } ;
} IMAGE_THUNK_DATA,*PIMAGE_THUNK_DATA;

typedef struct _IMAGE_IMPORT_DESCRIPTOR {
  union {
   DWORD Characteristics;
   PIMAGE_THUNK_DATA OriginalFirstThunk;
  } ;
  DWORD TimeDateStamp;
  DWORD ForwarderChain;
  DWORD Name;
  PIMAGE_THUNK_DATA FirstThunk;
} IMAGE_IMPORT_DESCRIPTOR,*PIMAGE_IMPORT_DESCRIPTOR;
```

## 3. 地址转换小结(RVAToOffset)

为什么要地址转换, 前人的文章说了很多,下面给出我的转换方法:

### 3.1 函数,它能给出 RVA 返回此 RVA 所在的节，来自 Matt Pietrek 的书：

```
PIMAGE_SECTION_HEADER GetEnclosingSectionHeader(DWORD rva){
  unsigned i;
 PIMAGE_SECTION_HEADER section = IMAGE_FIRST_SECTION32(NtHeader);
 for ( i=0; i < NtHeader->FileHeader.NumberOfSections; i++,section++){
    if ( (rva >=section->VirtualAddress) &&
  (rva < (section->VirtualAddress + section->Misc.VirtualSize)))
   return section;
 }
 return 0;
}
```

注： `hnhuqiong` 给的 `ollydump300110` 的源码里面也有类似函数，但是，很明显的有漏洞，那就是若 `RVA` 不在任何一个 `Section` 那么函数会返回最后一个 `Section`， 而不是像这里返回 `0` .下面是原始连接

`http://bbs.pediy.com/showthread.php?threadid=26520`

### 3.2 RVAToOffset

我一直没有注意的就是 `'Offset'` 这个词. `Offset` 其实还是一个偏移,只不过是在文件中, 要想得到目标文件的 `IAT`, 就要将这个值加上由 `MapViewOfFile` 返回的文件基址指针.

`Offset` 的获得 :

```
pSection = GetEnclosingSectionHeader(NtHeader->OptionalHeader.DataDirectory
                 [IMAGE_DIRECTORY_ENTRY_IAT].VirtualAddress);
Offset = (DWORD) (pSection->VirtualAddress - pSection->PointerToRawData);
```

以获得 `IMAGE_THUNK_DATA` 结构为例,给出用法:

```
ThunkData = (PIMAGE_THUNK_DATA)((DWORD)ImportDes->OriginalFirstThunk -
                    Offset + (PBYTE)pMapping);
```

呵呵, `(DWORD)ImportDes->OriginalFirstThunk -Offset` 得到的只是文件中的偏移, 注意加上由 `MapViewOfFile` 返回的 `pMapping`. 如果你象我原来一样,加上的是 `NtHeader->OptionalHeader.ImageBase` , 那么恭喜你, 访问错误.

## 4. 用 VC 6.0 + API 获得 IMAGE_IMPORT_BY_NAME 结构的一点问题

在 `VC` 里面, 在一个结构指针比如 `ThunkData` 后面加上 `'->'` 时, `vc` 会自动的列出结构的成员供你选择, 十分方便. 但是, 通过 `ThunkData` 继续想获得 `IMAGE_IMPORT_BY_NAME` 结构的时候, 你在 `ThunkData` 后面加 `'->'` 时, 出来的是一个 `'u1'`. 此时不要疑惑, 这个 `'u1'` 就是 `IMAGE_THUNK_DATA` 里面的那个 `union` 的名称, 所以你可以这样得到 `IMAGE_IMPORT_BY_NAME` 结构:

```
ImportBN = (PIMAGE_IMPORT_BY_NAME)((DWORD)(ThunkData->u1.AddressOfData)-
                    Offset +(PBYTE)pMapping);
```

## 5. Iczelion 的 PE 教程关于导入表的描述没有讲清楚，只是说用 IMAGE_THUNK_DATA 的每个数组元素和 IMAGE_ORDINAL_FLAG32，比较可以推断如果某个函数是由函数序数引出的，我就误解成用 ImportDes->OriginalFirstThunk 或者 ImportDes->FirstThunk 判断。是不是错的很远?

`Iczelion` 的 PE 教程关于导入表的描述没有讲清楚，只是说用 `IMAGE_THUNK_DATA` 的每个数组元素和 `IMAGE_ORDINAL_FLAG32`，比较可以推断如果某个函数是由函数序数引出的，我就误解成用 `ImportDes->OriginalFirstThunk` 或者 `ImportDes->FirstThunk` 判断。是不是错的很远?

参考（【翻译】“PE 文件格式”1.9 版 完整译文（附注释））`http://bbs.pediy.com/showthread.php?threadid=21932`，我们应该用 `IMAGE_THUNK_DATA` 结构里面的 `AddressOfData` 来判断。下面的代码可行：

```
while(ThunkData->u1.AddressOfData!=NULL){
 ImportBN = (PIMAGE_IMPORT_BY_NAME)((DWORD)(ThunkData->u1.AddressOfData) - Offset +(PBYTE)pMapping);
//显示导入函数
 if(((DWORD)ThunkData->u1.AddressOfData & IMAGE_ORDINAL_FLAG32) == 0){
  AddText(hEdit,TEXT("%03d:  %s\r\n"),i++,ImportBN->Name);
 }
 else{
  AddText(hEdit,TEXT("%03d:  Ord by Hint\r\n"),i++);
 }
 ThunkData ++;
 }//End of while
```

## 6. 导出表

### 6.1 导出表的结构

```
typedef struct _IMAGE_EXPORT_DIRECTORY {
 DWORD   Characteristics;
 DWORD   TimeDateStamp;
 WORD    MajorVersion;
 WORD    MinorVersion;
```



```
DWORD   Name;  
DWORD   Base;  
DWORD   NumberOfFunctions;  
DWORD   NumberOfNames;  
DWORD   AddressOfFunctions;     // RVA from base of image  
DWORD   AddressOfNames;         // RVA from base of image  
DWORD   AddressOfNameOrdinals;  // RVA from base of image  
} IMAGE_EXPORT_DIRECTORY, *PIMAGE_EXPORT_DIRECTORY;  
```

`6.2` `AddressOfNames`和`AddressOfNameOrdinals`是一一对应的，只不过一个用于名字，一个用于序号，同一个函数的索引都相同。

`6.3` `NumberOfFunctions` – `NumberOfNames`应该就是由序号引出的函数数目了

`6.4` 对于由序号导出的函数，不知道有没有办法能通过序数找到函数名。个人考虑似乎不可能这样找函数名字，不然，微软未公开的函数就都被我们通过函数序数枚举出来了？：）

`7` 把我的 PE 查看器修改了下，原来的在处理用序号引出的函数时会出错.:)

`4dd466d0e1e0cc123afbe504da2462ca`

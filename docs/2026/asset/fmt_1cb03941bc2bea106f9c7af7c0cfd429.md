
```c
PIMAGE_DOS_HEADER        pDos      = (PIMAGE_DOS_HEADER)lpImage;
    PIMAGE_NT_HEADERS32      pNT32     = (PIMAGE_NT_HEADERS32)((LONG)lpImage+pDos->e_lfanew);
    PIMAGE_OPTIONAL_HEADER32 pOptional = &(pNT32->OptionalHeader);
    PIMAGE_DATA_DIRECTORY    pDirec    = (PIMAGE_DATA_DIRECTORY)pOptional->DataDirectory;

    // 导出表的FOA
    PIMAGE_EXPORT_DIRECTORY pExp = (PIMAGE_EXPORT_DIRECTORY)((LONG)lpImage+RVA2FOA(lpImage, pDirec->VirtualAddress));

    DWORD dwNum = RVA2FOA(lpImage,pExp->AddressOfNameOrdinals);
    PWORD pNum = (PWORD)(dwNum+(DWORD)lpImage);

    DWORD pfn = RVA2FOA(lpImage,pExp->AddressOfFunctions);
    PDWORD pGet = PDWORD(pfn+(DWORD)lpImage);

    DWORD dwPNames  = RVA2FOA(lpImage,pExp->AddressOfNames);
    PDWORD pdwNames = PDWORD(dwPNames+(DWORD)lpImage);
    DWORD dwNames = RVA2FOA(lpImage,(DWORD)(*pdwNames));
    dwNames = (dwNames)+(DWORD)lpImage;

    vecDllNum.clear();
    vecNames.clear();
    vecAdd.clear();

    for( DWORD i=0; i<pExp->NumberOfFunctions; i++ )
    {
        if( !(*pGet) )
        {
            pGet++;
            continue;
        }
        int count=0;  BOOL bflag = FALSE;
        for( ; count<pExp->NumberOfNames; count++ )
        {
            if( i==pNum[count] )
            {
                bflag = TRUE;
                break;
            }
        }

        CString csTempNum;
        csTempNum.Format(_T("0x%p"),i+pExp->Base);
        vecDllNum.push_back(csTempNum);

        csTempNum.Format(_T("0x%p"),*pGet);
        vecAdd.push_back(csTempNum);

        CHAR Temp[100];

        if( !bflag )
        {
            csTempNum = _T("--");
        }
        else
        {
            sprintf_s(Temp,"%s",dwNames);
            csTempNum = Temp;
            pdwNames++;
            dwNames = RVA2FOA(lpImage,(DWORD)(*pdwNames));
            dwNames = (dwNames)+(DWORD)lpImage;
        }

        vecNames.push_back(csTempNum);
        pGet++;
    }
```

**导出表功能测试截图如下(以32位user32.dll为例)：**

![](https://bbs.kanxue.com/upload/attach/201312/543191_lreqqd1the65bmc.jpg)

---

## 0x2.5 导入表

**简要描述：**

导入表机制是PE文件从其他第三方程序中导入API，供本程序调用的机制。

比如导入表在反病毒的领域里，技术人员在某些情况下仅根据导入表就能猜测出此程序的大致行为。由此导入表的重要性可见一斑。

导入表(`IMAGE_IMPORT_DESCRIPTOR`)结构定义如下：

```c
typedef struct _IMAGE_IMPORT_DESCRIPTOR {

    union {

        DWORD   Characteristics;            // 0 for terminating null import descriptor

        DWORD   OriginalFirstThunk;         // RVA to original unbound IAT (PIMAGE_THUNK_DATA)

    } DUMMYUNIONNAME;

    DWORD   TimeDateStamp;                  // 0 if not bound,
                                            // -1 if bound, and real date\time stamp
                                            //     in IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT (new BIND)
                                            // O.W. date/time stamp of DLL bound to (Old BIND)

    DWORD   ForwarderChain;                 // -1 if no forwarders
    DWORD   Name;
    DWORD   FirstThunk;                     // RVA to IAT (if bound this IAT has actual addresses)

} IMAGE_IMPORT_DESCRIPTOR;

typedef IMAGE_IMPORT_DESCRIPTOR UNALIGNED *PIMAGE_IMPORT_DESCRIPTOR;
```

**上述结构说明：**

- `OriginalFirstThunk`：包含指向`INT`的RVA，`INT`是一个`IMAGE_THUNK_DATA`结构的数组，大多情况下，数组中的每个`IMAGE_THUNK_DATA`结构会再指向`IMAGE_IMPORT_BY_NAME`结构，结尾处以全0x00的`IMAGE_THUNK_DATA`结构结束。
- `TimeDateStamp`：一个32位的时间标识，与下一个转发链字段合并工作，否则可以为空。
- `ForwarderChain`：转发链，如果不转发则此值为0。
- `Name`：指向导入文件的名字。
- `FirstThunk`：指向导入地址表的RVA。

在一般情况下，对于导入表只需要关注`OriginalFirstThunk`与`FirstThunk`，两个字段分别指向了保存导出名称与导出地址的`IMAGE_THUNK_DATA`的结构数组，且这个数组是以一个空的`IMAGE_THUNK_DATA`结构结尾，其结构的详细内容如下：

```c
typedef struct _IMAGE_THUNK_DATA32 {

    union {

        DWORD ForwarderString;       // PBYTE
        DWORD Function;              // PDWORD
        DWORD Ordinal;               // 被导入函数的序号
        DWORD AddressOfData;         // PIMAGE_IMPORT_BY_NAME

    } u1;

} IMAGE_THUNK_DATA32;

typedef IMAGE_THUNK_DATA32 * PIMAGE_THUNK_DATA32;
```

**上述结构说明如下：**

- `ForwarderString`：负责与导入转发表forwarders协同工作的一个字段。当导入表的`ForwarderChain`不为0时，此值有效，并指向包含有转发函数与导出这个函数的映像文件名的字符串RVA。
- `Function`：导入表导入函数的实际内存地址，此字段仅在此映像被加载，且此结构为IAT的前提下有效。
- `Ordinal`：被导入函数的序号。
- `AddressOfData`：指向`IMAGE_IMPORT_BY_NAME`结构，当以上3个值都未生效时，此值有效。

下面了解一下`IMAGE_IMPORT_BY_NAME`结构：

```c
typedef struct _IMAGE_IMPORT_BY_NAME
{
    WORD Hint;             // 需导入的函数序号
    BYTE Name[1];          // 需导入的函数名称
} IMAGE_IMPORT_BY_NAME, PIMAGE_IMPORT_BY_NAME;
```

**上述结构说明如下：**

- `Hint`：保存着本映像导入表需导入的函数序号。
- `Name`：保存着本映像导入表需导入的函数名称。

`IMAGE_IMPORT_BY_NAME`结构以一个结构数组的形式存在，并以一个空的`IMAGE_IMPORT_BY_NAME`结构结束。

**导入表的大致结构图如下：**

![](https://bbs.kanxue.com/upload/attach/201312/543191_xtbc3rqvq2s8h9b.jpg)

**获取导入表相关信息的代码如下：**

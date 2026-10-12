
```asm
//------------------------------
// set the Eip to the OEP
MOV EDI,DWORD PTR DS:[EAX+0A4h] // EAX <- pContext.Ebx
MOV DWORD PTR DS:[EAX+0B8h],EDI // pContext.Eip <- EAX
//------------------------------
POP EDI
//==============================
MOV EAX, EXCEPTION_CONTINUE_SEARCH
LEAVE
RETN
//----------------------------------
DWORD_TYPE(DYN_LOADER_START_DATA1)
//----------------------------------
_RO_dwImageBase:                DWORD_TYPE(0xCCCCCCCC)
_RO_dwOrgEntryPoint:            DWORD_TYPE(0xCCCCCCCC)
//----------------------------------
DWORD_TYPE(DYN_LOADER_END_MAGIC)
//----------------------------------
}
}
```

## 6. 建立导入表并重构原来的导入表

为了在 Windows 应用程序中使用 Windows [dynamic link library (DLL)](http://en.wikipedia.org/wiki/Microsoft_Dynamic_Link_Library)，有两个方法可供选择：

- **使用 Windows libraries by additional dependencies**：

![](img/77ecc30b6120a9c845c65b72ff28d8f2.jpg)

- **在 run-time 使用 Windows dynamic link libraries**：

```c
// DLL function signature
typedef HGLOBAL (*importFunction_GlobalAlloc)(UINT, SIZE_T);
...
importFunction_GlobalAlloc __GlobalAlloc;

// Load DLL file
HINSTANCE hinstLib = LoadLibrary("Kernel32.dll");
if (hinstLib == NULL)
{
    // Error - unable to load DLL
}

// Get function pointer
__GlobalAlloc =
    (importFunction_GlobalAlloc)GetProcAddress(hinstLib,
                                                "GlobalAlloc");
if (addNumbers == NULL)
{
     // Error - unable to find DLL function
}

FreeLibrary(hinstLib);
```

当你创建 Windows 应用程序项目时，连接器在项目的基本依赖关系中至少会包括 `kernel32.dll`。没有 `Kernel32.dll` 的 [LoadLibrary()](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dllproc/base/loadlibrary.asp) 和 [GetProcAddress()](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/dllproc/base/getprocaddress.asp)，我们不能在 run-time 加载 DLL。依赖关系的信息保存在导入表区段里。通过 [Dependency Walker](http://www.microsoft.com/resources/documentation/Windows/XP/all/reskit/en-us/Default.asp?url=/resources/documentation/Windows/XP/all/reskit/en-us/prmb_tol_kewf.asp)，可以看到被导入 PE 文件的 DLL 模块和函数。

![](img/e87f0f1cdc6b1e4fdd8a60b9055a68a2.jpg)

我们设法建立定制的导入表来管理我们自己的项目。此外，为了运行程序真正的代码，我们必须修复原始的导入表。

### PE Maker - Step 3

- [Download source files - 65.4 Kb](http://www.codeproject.com/system/inject2exe/pemaker3.zip)

#### 6.1 构造客户端导入表

我强烈建议你阅读 [Microsoft Portable Executable and the Common Object File Format Specification](http://www.microsoft.com/whdc/system/platform/firmware/PECOFF.mspx) 文档的 6.4 节。这一节包含导入表实现的主要信息。

从 PE 头的 `Optional Header` 的 `Second Data Directory` 可以访问导入表数据，因此，你可以用下面的代码访问它：

```c
DWORD dwVirtualAddress = image_nt_headers->
 OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress;
DWORD dwSize = image_nt_headers->
 OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].Size;
```

`VirtualAddress` 通过 `IMAGE_IMPORT_DESCRIPTOR` 指向结构。这个结构包含指向导入的 DLL 名和 `first thunk` 的 `RVA` 的指针。

```c
typedef struct _IMAGE_IMPORT_DESCRIPTOR {
    union {
        DWORD   Characteristics;
        DWORD   OriginalFirstThunk;
    };
    DWORD   TimeDateStamp;
    DWORD   ForwarderChain;
    DWORD   Name;        // the imported DLL name
    DWORD   FirstThunk;  // the relative virtual address of the first thunk
} IMAGE_IMPORT_DESCRIPTOR, *PIMAGE_IMPORT_DESCRIPTOR;
```

当程序运行时，Windows 任务管理器通过函数的 `virtual address` 设置 `thunks`。通过函数名可以发现 `virtual address`。起先，`thunks` 持有函数名的 `RVA`，见 [Table 5](http://www.codeproject.com/system/inject2exe.asp#Table5#Table5)；在执行期间，通过函数的 `virtual address` 修复它们，见 [Table 6](http://www.codeproject.com/system/inject2exe.asp#Table6#Table6)。

**Table 5 - The Import Table in file image**

| IMAGE_IMPORT_DESCRIPTOR[0] | OriginalFirstThunk |   |   |
| :--- | :--- | :--- | :--- |
| TimeDateStamp |   |   |   |
| ForwarderChain |   |   |   |
| Name_RVA | \------> | "kernel32.dll",0 |   |
| FirstThunk_RVA | \------> | proc_1_name_RVA | \------> | 0,0,"LoadLibraryA",0 |
|   |   | proc_2_name_RVA | \------> | 0,0,"GetProcAddress",0 |
|   |   | proc_3_name_RVA | \------> | 0,0,"GetModuleHandleA",0 |
|   |   | ... |   |   |
| IMAGE_IMPORT_DESCRIPTOR[1] |   |   |   |
| ... |   |   |   |
| IMAGE_IMPORT_DESCRIPTOR[n] |   |   |   |

**Table 6 - The Import Table in virtual memory**

| IMAGE_IMPORT_DESCRIPTOR[0] | OriginalFirstThunk |   |
| :--- | :--- | :--- |
| TimeDateStamp |   |   |
| ForwarderChain |   |   |
| Name_RVA | \------> | "kernel32.dll",0 |
| FirstThunk_RVA | \------> | proc_1_VA |
|   |   | proc_2_VA |
|   |   | proc_3_VA |
|   |   | ... |
| IMAGE_IMPORT_DESCRIPTOR[1] |   |   |
| ... |   |   |
| IMAGE_IMPORT_DESCRIPTOR[n] |   |   |

我们想生成一个简单的导入表，从 `Kernel32.dll` 中导入 `LoadLibrary()` 和 `GetProcAddress()`。我们需要在 run-time，这两个基本的 API 函数能包含其它的 API 函数。下面的汇编代码显示我们可以很容易做到：

```asm
0101F000: 
00000000 ; OriginalFirstThunk
0101F004: 00000000 ; TimeDateStamp
0101F008: 00000000 ; ForwarderChain
0101F00C: 0001F034 ; Name;       ImageBase + 0001F034 -> 0101F034 -> "Kernel32.dll",0
0101F010: 0001F028 ; FirstThunk;  ImageBase + 0001F028 -> 0101F028
0101F014: 00000000
```


# 导出表

```c
typedef struct _IMAGE_EXPORT_DIRECTORY {
    DWORD   Characteristics;      // 通常为0
    00000000
    DWORD   TimeDateStamp;        // 创建时间, 不是很有效的值
    00000000
    WORD    MajorVersion;         // 主版本号
    0000
    WORD    MinorVersion;         // 小版本号
    0000
    DWORD   Name;                 // 指向以0结尾的ASCII字符串(DLL名称)的RVA
                                  // 如(user32.dll, kernel32.dll)
    0010F780
    DWORD   Base;                 // 基址, 一个输出项的序数就是函数地址数组中的索引值加base.
                                  // base大多时候为1, 说明第一个输出函数的序数为1
    00000001
    DWORD   NumberOfFunctions;    // 实际Export函数的个数
    000000BC
    DWORD   NumberOfNames;        // Export函数中具名的函数个数(以名称来输出函数的数量)
    000000BC
    DWORD   AddressOfFunctions;   // Export函数地址数组(数组个数: NumberOfFunctions)
    0010F028
    DWORD   AddressOfNames;       // Export函数名称地址数组(数组个数:NumberOfNames)
    0010F318
    DWORD   AddressOfNameOrdinals;// 指向函数名名称对应序数输出条目列表的RVA
                                  // 数组每个名称拥有一个相应的序数(数组个数:NumberOfNames)
    0010F608
} IMAGE_EXPORT_DIRECTORY, *PIMAGE_EXPORT_DIRECTORY;
```

从导出表中获得函数地址的 API 为 `GetProcAddress()` 函数。该 API 用来引用 EAT 来获取指定的 API 的地址。

注：

1. 导出函数也可能没有名称的，这时只能通过序数导出。
2. 序数是指定某个输出函数的独一无二的 16 位数字（2 个字节）。

两种导出函数的方法：

### 一：按函数名字

1. 通过 `AddressOfNames` 找到函数名称数组。使用 `strcmp()` 函数，在（RVA）指针数组从索引值 0 开始依次与我们要找的函数名称对比，从而找到索引值 `index_name`。
2. 通过 `AddressOfNameOrdinals` 找到存放函数序号的数组，使用步骤 1 获得的 `index_name` 为索引值找到函数地址的序号（`index_address`）。
3. 通过 `AddressOfFunctions` 找到函数地址数组（EAT），在 EAT 中使用步骤 2 获得的 `index_address` 为索引值找到指定函数的 RVA。

### 二：按函数序号

1. 使用我们函数的序号减去 `_IMAGE_EXPORT_DIRECTORY.Base` 的值得到函数地址索引值 `index_address`。
2. 通过 `AddressOfFunctions` 找到函数地址数组（EAT），在 EAT 中使用步骤 1 获得的 `index_address` 为索引值找到指定函数的 RVA。

下面依旧用 OD 程序来看导出表，并将每个值标在上面每个成员下面，通过上面 `IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES]` 的介绍，已经标出导出表的 RVA: `0010F000` Size: `000012FA`。再通过 CFF Explorer 工具查看每个节的地址可以计算出输出表的 RAW: `00CE200`。

1. 查看输出表名称（RVA: `0010F780` -> RAW: `000CE980`）。

    ![](https://bbs.kanxue.com/upload/attach/202005/891547_PXPQZGNA7F38KVC.png)

    ![](https://bbs.kanxue.com/upload/attach/202005/891547_JJ43ZPDJAKZFDB5.png)

2. 查找函数名称。

    (1) `AddressOfNames`（RVA: `0010F318` -> RAW: `000CE518`）

    ![](https://bbs.kanxue.com/upload/attach/202005/891547_TA92WWRWDF7JAWU.png)

    ![](https://bbs.kanxue.com/upload/attach/202005/891547_69ZKWK4JWCD6TZB.png)

    由（RVA: `0010F78C` -> RAW: `000CE98C`）：

    ![](https://bbs.kanxue.com/upload/attach/202005/891547_9GM9VBB4DGBHPH5.png)

    现在已经找到了函数的名称，下面模拟查看一个指定名称函数的 RVA。假设我们找的是 `Addsorteddata`（即第一个函数），(1) 通过 `strcmp()` 得到它的索引值是 0，记为 `index_name`。(2) 通过 `AddressOfNameOrdinals` 使用 `index_name` 找到函数的序数，通过下图得到序数 0，记为 `index_address`。

    `AddressOfNameOrdinals`（RVA: `0010F608` -> RAW: `000CE808`）：

    ![](https://bbs.kanxue.com/upload/attach/202005/891547_5ABHVN6ANPJ4UXJ.png)

    (3) 通过 `AddressOfFunctions` 函数地址数组（EAT），使用 `index_address` 为索引值得到我们指定函数的 RVA。

    `AddressOfFunctions`（RVA: `0010F028` -> RAW: `000CE228`）：

    ![](https://bbs.kanxue.com/upload/attach/202005/891547_4AAUS64B4ZA7FKY.png)

    到此，得到我们指定输出函数 `Addsorteddata` 的 RVA: `00054EFC`。最后通过 OD 载入 OD 看一下。

    ![](https://bbs.kanxue.com/upload/attach/202005/891547_PKDSHG49ZFMWEPT.png)

从这里也说明了，`.exe` 文件也是可能有输出表的。

# 0x6 IMAGE_IMPORT_DESCRIPTOP 输入表

记录 PE 文件要导入那些库文件，`DataDirectory[1]` 记录了 `RVA` 及 `Size`。

首先，执行一个程序会有很多的函数是公用的，在动态链接库里（动态链接库，`.dll` 文件总是附加在一个要执行的程序中，`.dll` 文件中有说明库 EAT 的输出表），如下图，一个程序加载的部分 `.dll` 文件。

![](https://bbs.kanxue.com/upload/attach/202005/891547_RRCYYMDDN3UU99G.png)

我们的输入表记录了需要用到的函数名称，通过在加载的动态链接库中搜索该函数得到实际的 `RVA`，再记录到输入表中，供程序使用。另外执行一个普通的程序一般需要多个库，那导入多少库，就会有多少个输入表结构体。这就构成了结构体数组且结构体数组最后以 `NULL` 结束（即每个导入的 DLL 都会成为数组中的一项）。

```c
typedef struct _IMAGE_IMPORT_DESCRIPTOR {
    union {
        DWORD   Characteristics;      // 由于是一个联合，如果这是该结构体数组的最后
                                      // 一项，那使用 Characteristics 成员，且值为 0，
                                      // 否则使用下面一个成员
        DWORD   OriginalFirstThunk;   // INT（import name table）结构体数组的 RVA
                                      // 数组每个成员记录了要使用函数名称与序号
    } DUMMYUNIONNAME;
    DWORD   TimeDateStamp;            // 映象绑定前，这个值是 0，绑定后是导入模块的时间戳
                                      // 据说可以用来确定输入表是否绑定从而是否需要重定位
    DWORD   ForwarderChain;           // 中转链，输入函数列表中第一个中转的、32 位的索引
                                      // 如果没有转发链，值为 -1
    DWORD   Name;                     // DLL 文件的名称（0 结尾的 ASCII 码字符串）的 32 位的 RVA，
                                      // 所以一个导入模块对应一个这样的数组
    DWORD   FirstThunk;               // IAT（import address table）结构体数组的 RVA
} IMAGE_IMPORT_DESCRIPTOR;
```

上面的 `OriginalFirstThunk`（INT）、`FirstThunk`（IAT）成员在 PE 文件加载前一般是都同时指向相同地址的 `IMAGE_THUNK_DATA` 数组。下面是 `IMAGE_THUNK_DATA32` 的定义。

```c
typedef struct _IMAGE_THUNK_DATA32 {
    union { // 一个联合，所以意味着每次只能使用一个成员
        DWORD ForwarderString;  // 中转链，一个 DLL 文件能输出不定义在本 DLL 文件中却需从另一个 DLL 文件中的函数。
        DWORD Function;         // 函数的地址
        DWORD Ordinal;          // 函数的序数。由于所有成员都是同一个地址，当最高位为 1 时表
                                // 示列表中没有函数的名字信息，只能通过本序数查找函数。
                                // 用低 16 位表示的序数，因为最高位作为标志了。
        DWORD AddressOfData;    // 同上，由于所有成员都是同一个地址，当最高位为 0 时，则
                                // 使用本成员，用低 31 位表示 _IMAGE_IMPORT_BY_NAME 结构的 RVA
    } u1;
} IMAGE_THUNK_DATA32;
typedef IMAGE_THUNK_DATA32 *PIMAGE_THUNK_DATA32;
```

上面介绍的 `AddressOfData` 成员的低 31 位就记录指向下面所示的 `_IMAGE_IMPORT_BY_NAME` 结构体数组的地址（RVA），数组中每个成员的前 2 个字节是函数的序数，后面跟着长度不定的函数名称的字符串。

```c
typedef struct _IMAGE_IMPORT_BY_NAME {
    WORD Hint;       // 函数的序数（即索引，与输出表中讲的一样）
    BYTE Name[1];    // 函数名称数组，记录函数的名称。数量未定义即长度不定。
} IMAGE_IMPORT_BY_NAME, *PIMAGE_IMPORT_BY_NAME;
```

注：上面所讲的 `OriginalFirstThunk` 成员（指针数组）的值是不能改写的，通过它寻找函数的名称。而 `FirstThunk` 成员（指针数组）的值在 PE 文件被 PE 装载器加载时，PE 装载器会通过 `OriginalFirstThunk` 得到函数的名称或者序数，然后通过函数名称在加载的 `.dll` 文件的输出表中找到函数的实际地址，然后替换到 `FirstThunk` 的一个值。装载完成后，`FirstThunk` 数组就指向了函数实际的地址。另外上面的 `TimeDateStamp` 成员可以用来确定输入表是否绑定从而是否需要重定位，如果它的值是 0，那么输入列表没有被绑定，加载器总是要修复输入表。否则输入被绑定，但该时间戳的值必须和 `.dll` 文件头中 `TimeDateStamp` 的一样，如果不一样，仍会修正输入表，就会进行下面的步骤。

## 导入函数输入到 IAT 的顺序（摘自《逆向工程核心原理》）

1. 读取 `_IMAGE_IMPORT_DESCRIPTOR` 中的 `name` 成员，获取库名称字符串。如 `user32.dll`。
2. 装载相应的库。`LoadLibrary("user32.dll")`。
3. 读取 `_IMAGE_IMPORT_DESCRIPTOR` 中的 `OriginalFirstThunk` 成员，得到 INT 地址。
4. 逐一读取 INT 中数组的值，获取相应的 `IMAGE_IMPORT_BY_NAME` 地址（RVA）。
5. 使用 `IMAGE_IMPORT_BY_NAME` 的 `Hint`（ordinal/序数）或 `name` 项，获取相应函数的起始地址：`GetProcAddress("函数名称")`。
6. 读取 IAT 成员，获得 IAT 地址。
7. 将上面获得的函数地址输入相应的 IAT 数组值。
8. 重复步骤 4-7，直到 INT 结束。

图示一下 INT 与 IAT 关系（技术太差了.png）。

![](https://bbs.kanxue.com/upload/attach/202005/891547_97PUDE3F2CKYJBK.png)

## 实例查看 OD 程序的输入表

1. 首先从 PE 文件可选头的 `DataDirectory[1].VirtualAddress` 得到输入表的 RVA: `10D000h` 及 Size: `1c87h`。

    ![](https://bbs.kanxue.com/upload/attach/202005/891547_WRZ74HFHSRMUECC.png)

2. RVA: `10D000h` -> RAW: `(10D000-10D000+CC400) = CC400h`。

    ![](https://bbs.kanxue.com/upload/attach/202005/891547_SG7MUGN6GABT9XQ.png)

3. 找到输入表。记录下对应成员的 RVA。

    ![](https://bbs.kanxue.com/upload/attach/202005/891547_XXY4FRNSTH5WCTH.png)

4. 查看该输入表名称：`ADVAPI32.DLL`，RVA: `10D9C8` -> RAW: `(10D9C8-10D000+CC400) = CCDC8`。

    ![](https://bbs.kanxue.com/upload/attach/202005/891547_D2CKCSB8G828845.png)

5. 查看 `OriginalFirstThunk`（INT），RVA: `10D0C8` -> RAW: `(10D0C8-10D000+CC400) = CC4C8`。

    ![](https://bbs.kanxue.com/upload/attach/202005/891547_82PEGMXW7T74UZF.png)

6. 可以看到第一成员的最高位是 0，则该值是 `IMAGE_IMPORT_BY_NAME` 的 RVA（RVA: `10DA33` -> RAW: `CCE33`）。

    ![](https://bbs.kanxue.com/upload/attach/202005/891547_TGDP9STXEXXP87E.png)

7. 查看 `FirstThunk`（IAT），RVA: `10D0E4` -> RAW: `(10D0E4-10D000+CC400) = CC4E4`。

    ![](https://bbs.kanxue.com/upload/attach/202005/891547_FQ842GMUSHDEM8K.png)

8. 从步骤 7 可以看到，PE 装载器装载 PE 文件之前，INT 与 IAT 各元素同时指向相同的地址。

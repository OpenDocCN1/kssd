
- `Size`则指定了该数据块的大小，有时并不是该类型数据的总大小，可能只是该类型数据一个数据项的大小。这两个成员(主要是`VirtualAddress`)成为了定位各种表的关键，所以一定要知道每个数组元素所指向的数据块类型，以下表格就是它的对应关系：

![](https://bbs.kanxue.com/upload/attach/202111/919233_BRRDKGZZY3A2JH2.jpg)

图3-7    `IMAGE_DATA_DIRECTORY`数组元素项图析

- **定位数据目录的技巧：**数据目录向上找`10h`，+4即为`DataDirectory`的地址，编译器生成的一般都是`10h`。

![](https://bbs.kanxue.com/upload/attach/202111/919233_RB8ZVETD6478R36.jpg)

图3-8    数据目录定位图析

#### （3）数据目录重点元素--`DataDirectory[1]` IMP（导入表）

- **IMP（导入表）：**导入表用来描述模块调用的API列表，位于PE数据目录中的第二项即`DataDirectory[1]`，其中记录了导入表的地址和大小，`VirtualAddress`指向`IMAGE_IMPORT_DESCRIPTOR`结构体数组，这个结构体数组中的每个元素对应着一个dll文件，以全0作为最后一个元素结尾。程序产生调用会生成`CALL`指令，两大问题及解决思路如下：

- **1.地址存放问题：**出于运行环境等因素考虑，导入函数的地址不能为固定地址。所以在exe中保存导入函数的相关信息，系统和链接器对其进行约定：链接器在生成exe的时候，为所有调用API的地方填写一个间接地址，当程序运行起来后，相应地址则会被写入真正API的地址，此区域即为`IAT`表（导入地址表）；

- **2.exe如何存储导入dll及其函数信息：**dll与函数是一对多的关系，原则上应该设计为多方填写1方信息的数据关系，但考虑到数据较多的情况，遍历不便。反过来设计为1方存储多方信息的数据结构，虽然会造成插入删除的不方便，但是考虑到exe加载dll的实际场景，无插入删除需求，所以应该设计为后者结构更贴合遍历查询需求。

```
// IMAGE_IMPORT_DESCRIPTOR 导入表结构，以全0（20个0）结尾

typedef struct _IMAGE_IMPORT_DESCRIPTOR {

    union {

        DWORD   Characteristics;     //  作用1：导入表结束标志

        DWORD   OriginalFirstThunk;  //* 作用2：*RVA指向一个结构体数组(INT表)

    }; 

    DWORD   TimeDateStamp;           //  时间戳

    DWORD   ForwarderChain;          //  -1 if no forwarders

    DWORD   Name;                    //* RVA指向以0结尾的dll名字

    DWORD   FirstThunk;              //* RVA指向一个结构体数组(IAT表，DataDirectory[12]项)

} IMAGE_IMPORT_DESCRIPTOR, *PIMAGE_IMPORT_DESCRIPTOR;
```

- **IAT（导入地址表）：**`IMP`中的`FirstThunk`指向`IAT`表。

- **INT（导入名称表）：**`IMP`中的`OriginalFirstThunk`指向`INT`表，也是`DataDirectory[12]`项。

- **PE加载前，**`IAT`和`INT`都指向`_IMAGE_IMPORT_BY_NAME`结构体

```
/ IMAGE_THUNK_DATA结构体汇总只有一个联合体，

// 一般用四字节的AddressOfData来获取IMAGE_IMPORT_BY_NAME的地址。

// 四字节解析，看最高位[31]：

//      1表序号导入，低word为导入函数的序号值；

//      0表RVA，指向_IMAGE_IMPORT_BY_NAME；

typedef struct _IMAGE_THUNK_DATA32 {

    union {

        PBYTE  ForwarderString;  	// PBYTE 指向一个转向者字符串的RVA；

        PDWORD Function;      	 	// PDWORD 被输入的函数的内存地址；

        DWORD Ordinal;       		// *被输入的API的序数值

        PIMAGE_IMPORT_BY_NAME  AddressOfData; //*RVA 指向IMAGE_IMPORT_BY_NAME

    } u1;

} IMAGE_THUNK_DATA32;





// IMAGE_IMPORT_BY_NAME有两个成员：1.序号；2.函数名。

typedef struct _IMAGE_IMPORT_BY_NAME {

    WORD    Hint;   // 可能为0，编译器决定，如果不为0，是函数在导出表中的索引

    BYTE    Name[1];// 函数名称以0结尾，由于不知道到底多长，所以干脆只给出第一个字符，找到0结束

} IMAGE_IMPORT_BY_NAME, *PIMAGE_IMPORT_BY_NAME;
```

- **PE加载后，IAT有变：**加载后的`IAT`每一项存储的是所对应的导入函数地址。

![](https://bbs.kanxue.com/upload/attach/202111/919233_HHNRRCW3RQ75JK4.jpg)

图3-9    PE加载前后IMP、IAT、INT之间的关系

### 4.节数据

- 节数据：即是由不同属性数据组成的不同节。

# 四、手撕PE

## （一）实现目标

- 实现目标：手写实现196 Byte大小的PE文件（又名：畸形PE/变形PE），要求`MessageBox`弹框显示"hello world"。

- 实现要点：充分利用空间，在保证遵循PE结构的基础上对数据结构进行重构存放。

## （二）环境工具

- 运行环境--XP系统：XP对于畸形PE的兼容性更高。

- 编辑器--winhex：用于编写PE文件。

## （三）步骤

#### 1.创建文件

- 使用`winhex`工具，新建196字节大小的PE文件并将数据初始化为全`CC`，以便分辨空闲区。

![](https://bbs.kanxue.com/upload/attach/202111/919233_77FTSZWU2EM55G7.jpg)

图4-1    myWinPE.exe文件创建流程

#### 2.编写DOS_HEADER部分

- **文件偏移+0x00（注1）**，`WORD` `e_magic`→'MZ'文件头标识：`0x5A4D`（注2）

- **文件偏移+0x3C**，`LONG` `e_lfanew`→指向NT头'PE'标识的地址：`0x00000004`

- 此处设置NT头地址为`0x4`偏移的原因：1.充分利用每字节可用空间，DOS除首尾两字段外都非必要字段，可用；2.当NT头置于`0x4`处，`0x3C`的功能不仅是指向NT头标识，还将成为选项头的成员`SectionAlignment`（文件对齐值），在XP中，此项最小值为4。综上两点，此处写入`0x4`最为合适。

![](https://bbs.kanxue.com/upload/attach/202111/919233_FVA6CVCDXWGUGNY.jpg)

图4-2    DOS_HEADER部分内容

#### 3.编写NT头的Signature部分

- **文件偏移+0x04，**`DWORD` `Signature`→'PE00'NT头标识：`0x00004550`

![](https://bbs.kanxue.com/upload/attach/202111/919233_YTYRH6VD93DF6UZ.jpg)

图4-3    NT头的Signature部分内容

#### 4.编写NT头的FILE_HEADER部分

- **文件偏移+0x08，**`WORD` `Machine`→指定程序的运行平台**：**`0x104C`（运行于Intel 386）

- **文件偏移+0x0A，**`WORD` `NumberOfSections`→PE中的节数量：`0x0001`（最小限值）

- **文件偏移+0x18，**`WORD` `SizeOfOptionalHeader`→选项头大小：`0x0080`

- 选项头大小默认为`0xE0`，不过以目标为导向，我们还是要力求最小，计算可得，不包含数据目录字段的选项头结构体大小是`0x60`，而每一个数据目录数组元素的大小是`0x8`，此处我们留出4个数据目录元素的大小，即`4*8=0x20`，加上`0x60=0x80`。

- **文件偏移+0x1A，**`WORD` `Characteristics`→文件属性：`0x010F`

`0x010F`组合属性表示：只在32位平台上运行的不存在行信息、重定位信息、符号信息的可执行文件。

![](https://bbs.kanxue.com/upload/attach/202111/919233_TMXSEVUHBGYN8X3.jpg)

图4-4    NT头的FILE_HEADER部分内容

#### 5.编写NT头的OPTIONAL_HEADER部分

- **文件偏移+0x1C，**`WORD` `Magic`→PE标志字即程序位数**：**`0x010B`（32位系统）

- **文件偏移+0x2C，**`DWORD` `AddressOfEntryPoint`→程序执行入口RVA：`0x00000000`

- **文件偏移+0x38，**`DWORD` `ImageBase`→内存加载基址：`0x00400000`（exe默认）

- **文件偏移+0x40，**`DWORD` `FileAlignment`→文件对齐值：`0x00000004`（最小限值）

- **文件偏移+0x4C，**`WORD`  `MajorSubsystemVersion`→主子系统版本号：`0x0004`（恢复默认不可修改）

- **文件偏移+0x4E，**`WORD`  `MinorSubsystemVersion`→副子系统版本号：`0x0000`（恢复默认不可修改）

- **文件偏移+0x50，**`DWORD`  `Win32VersionValue`→版本号，XP中须为0：`0x00000000`

- **文件偏移+0x54，**`DWORD` `SizeOfImage`→PE文件在进程内存中的总大小：`0x00001000`（内存预计占用1页内）

- **文件偏移+0x58，**`DWORD` `SizeOfHeaders`→PE文件头部在文件中的总大小：`0x000000C4`（=文件实现目标大小）

- **文件偏移+0x60，**`WORD`  `Subsystem`→程序类型：`0x0002` **（**2=图形界面**）**

- **文件偏移+0x62，**`WORD`   `DllCharacteristics`→文件特性：`0x0000` **（**默认为0**）**

- **文件偏移+0x64，**`DWORD`  `SizeOfStackReserve`→初始化时保留的栈大小：`0x07FFFFFF`

- **文件偏移+0x68，**`DWORD`  `SizeOfStackCommit`→初始化时实际提交的栈大小：`0x07FFFFFF`

- **文件偏移+0x6C，**`DWORD`  `SizeOfHeapReserve`→初始化时保留的堆大小：`0x07FFFFFF`

- **文件偏移+0x70，**`DWORD`  `SizeOfHeapCommit`→初始化时实际提交的堆大小：`0x07FFFFFF`

- **文件偏移+0x78，**`DWORD`  `NumberOfRvaAndSizes`→数据目录的元素个数：`0x00000004`

- **文件偏移+0x7C，**`DataDirectory[4]`→4项元素的数据目录：全0

![](https://bbs.kanxue.com/upload/attach/202111/919233_ENJZJNN8FP8HXZT.jpg)

图4-5    NT头的OPTIONAL_HEADER部分内容

#### 6.编写节表部分

- **文件偏移+0xA4，**`DWORD` `VirtualSize`→内存时机占用大小：`0x000000C4`

- **文件偏移+0xA8，**`DWORD` `VirtualAddress`→内存地址RVA：`0x00000000`

- **文件偏移+0xAC，**`DWORD` `SizeOfRawData`→文件大小：`0x000000C4`

- **文件偏移+0xB0，**`DWORD` `PointerToRawData`→文件偏移：`0x00000000`

- **文件偏移+0xC4，**`DWORD` `Characteristics`→节属性**：**`0xE00000E0`

- `0xE00000E0` = `1110 0000 0000 0000 0000 0000 1110 0000`，根据图3-5显示，此属性值表示节中包含代码、已初始化数据和未初始化数据，且映射到内存后的页面可执行可读写。

- **\*0xA4-0xB3含义：**从文件偏移为`0x0`的位置，拷贝`0xC4`大小的数据到内存偏移为`0x0`的位置，占用内存`0xC4`大小

![](https://bbs.kanxue.com/upload/attach/202111/919233_GX43EAD4Z5MRSHJ.jpg)

图4-6   节表部分内容

#### 7.基础PE总览（xdm先备份一下，再做后方花里胡哨的操作）

![](https://bbs.kanxue.com/upload/attach/202111/919233_EX4VKVDTZ8RWHGX.jpg)

图4-7   基础PE内容总览

#### 8.编写导入内容：库+函数

- 根据导入内容思考设计：实现弹框需要写入两项内容：1.库名（user32）；2.函数名（`MessageBoxA`）。根据两个字符串的长度（+00结束符）找到最合适的位置并写入：

- **文件偏移+0x30，** dll名："`user32 `"，7Byte

- **文件偏移+0x0C，** 函数名："`MessageBoxA `"，12Byte

![](https://bbs.kanxue.com/upload/attach/202111/919233_W8PZFJZ7372W764.jpg)

图4-8   写入库名和函数名

#### 9.编写导入表项

- 导入表`IMAGE_IMPORT_DESCRIPTOR`置于`0xB0`处，因为PE加载后，`0xB0-0xC3`处内存会被初始化为全0。

- **文件偏移+0xBC**：`DWORD`   `Name`→RVA指向dll名**：**`0x00000030`

- **文件偏移+0xC0**：`DWORD`   `FirstThunk`→RVA指向`IAT`表：`0x0000009C`

![](https://bbs.kanxue.com/upload/attach/202111/919233_VQ3MK6582YWX44V.jpg)

图4-9   填充导入表项

#### 10.编写IAT表 IMAGE_THUNK_DATA32

- **文件偏移+0x9C**，`PIMAGE_IMPORT_BY_NAME`  `AddressOfData`→指向`IMAGE_IMPORT_BY_NAME`：`0x0000000A`

- 因为`IMAGE_IMPORT_BY_NAME`结构体的指向函数名的`NAME`成员前还有一个`WORD` `Hint`成员，所以指向的地址需要跳过一个`WORD` 2字节，以实现`NAME`指向函数名。

- **文件偏移+0xA0**，以全0作结尾项：`0x00000000`

![](https://bbs.kanxue.com/upload/attach/202111/919233_9MUDRFTYAW6E84B.jpg)

图4-10  编写IAT表

#### 11.将导入表位置写入数据目录中

- **文件偏移+0x80**，`DataDirectory[1]`导入表位置：`0x000000B0`

![](https://bbs.kanxue.com/upload/attach/202111/919233_QFYCKSUG66TANF2.jpg)

图4-11  写入导入表位置

- OD检测：内存窗口中跟随地址`0x40009C`，以"`长型→地址`"方式查看，可见导入成功。

![](https://bbs.kanxue.com/upload/attach/202111/919233_CFMMPG6TR646CUR.jpg)

图4-12  OD检测导入表

#### 12.编写汇编指令：获取MessageBoxA函数地址并传参调用

- **文件偏移+0x60**，在选项头的堆栈空间值区编写弹框文本："`hello world`"

![](https://bbs.kanxue.com/upload/attach/202111/919233_9VEAVP46Y47PCT6.jpg)

图4-13  写入MessageBox参数

- \*选线头主版本号不可用，故以副链接器版本号成员处（`0x1F`）开始非重要区域编写指令：

- **文件偏移+0x1F**，参数4 `UINT` `uType`→对话框按钮类型：`6A 00` = `push 0`（注3）

- **文件偏移+0x21**，参数3 `LPCTSTR` `lpCaption`→消息框标题：`6A 00` = `push 0`

- **文件偏移+0x23**，参数2 `LPCTSTR` `lpText`→消息框内容，指向"`hello world`"：`68 64004000` = `push 00400064`

- **文件偏移+0x25**，参数1 `HWND` `hWnd`→窗口句柄：`6A 00` = `push 0`

- **文件偏移+0x2A**，短跳至空闲区以备调用函数：`EB18=` `jmp short 00400044`

- **文件偏移+0x44**，从`IAT`表中获取API地址并调用：`FF15 9C004000=` `CALL  0040009C`

- **文件偏移+0x4A**，调用后返回：`C3=` `ret`

![](https://bbs.kanxue.com/upload/attach/202111/919233_WM4ETPWFY6MGJJU.jpg)

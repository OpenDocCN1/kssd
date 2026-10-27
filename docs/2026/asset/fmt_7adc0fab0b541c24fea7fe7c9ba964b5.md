
# [原创]PE文件编辑器

## 概述

## 预备知识

### MS-DOS头

### PE文件头

#### `IMAGE_FILE_HEADER`

#### `IMAGE_OPTIONAL_HEADER32`

#### `IMAGE_DATA_DIRECTORY`

##### `IMAGE_DIRECTORY_ENTRY_EXPORT`

##### `IMAGE_IMPORT_DESCRIPTOR`

##### `IMAGE_BASE_RELOCATION`

#### 节表信息

## 总体设计

### 不足

### 改进

## 详细设计

### 文件保存

### 关联信息的修改

### 文件位置计算器（FLC）

### 节信息（Section）

### 目录信息

#### 导出表

##### 导出表信息的展示

##### 导出表信息的修改

#### 导入表

### 资源

#### 资源信息的获取

#### 资源信息的展示

### 重定位信息

## 控件设计

### Edit控件

#### 信息的展示

#### 限制用户输入

#### 信息的获取

### 单选框控件

### 复选框与下拉框控件

#### 复选框的设置

#### 下拉框的设置

### 列表控件（ListCtrl）

#### 列表右键弹出菜单

## 总结

## 概述

最近学习了PE文件的知识，为了加深对知识的理解，任老师给我们布置了一个任务，自己完成一个PE编辑器。在写之前还觉得挺简单，到真正开始写的时候才发现有好多东西都不是想象中那么简单，有许多需要注意的地方。也多亏了这次写PE编辑器积累的经验，在写壳时给了我很大的帮助。

先说明一下文章中用到的一些词汇，不一定用的对。

文件缓存：读取磁盘上的一个PE文件，然后将这个PE文件保存到内存中的一段空间，使用new一段堆空间，或者使用文件映射的方式都行。

区段，节：都是指`Section`

偏移:一个特定位置基于某个指定位置的差值

`FOA(File Offset Address)`:数据在PE文件中的地址叫文件偏移地址。这是文件在磁盘上存放时相对于文件开头的偏移

`RVA(Relative Virtual Address)`:相对虚拟地址是内存地址相对于映射基址的偏移量。

映射基址:`PE`文件被系统`PE`加载器加载到内存后的首地址。

## 预备知识

关于PE文件的知识，网上有很多资料，这里就不做深入的解释了，只说一些我写程序时遇到的问题，有一部分问题是写壳时遇到的，这里也一起总结一下。

### `MS-DOS`头

将PE文件读入内存中后，将文件缓存的首地址强制转换为`IMAGE_DOS_HEADER`类型的指针就可以获取MS-DOS头的内容，在这个结构体中只需要关注两个字段`e_magic`字段和`e_lfanew`字段

- `e_magic`字段用来检测是否有效的PE文件
- `e_lfanew`字段表示windows下PE文件头的偏移位置

### `PE`文件头

`IMAGE_NT_HEADERS`用这个结构体来表示，通过MS-DOS头的`e_lfanew`字段的值加上文件缓存的起始地址可以获取该结构体类型的指针。

这个头在32位和64位系统上是不一样的，上面的结构体是一个宏，根据不同的编译选项生成不同的结构体

```
typedef struct _IMAGE_NT_HEADERS {
    DWORD Signature;
    IMAGE_FILE_HEADER FileHeader;
    IMAGE_OPTIONAL_HEADER32 OptionalHeader;
} IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS32;
```

- `Signature`
字段表示PE标示，用来检测是否是有效的PE文件。

#### `IMAGE_FILE_HEADER`

```
typedef struct _IMAGE_FILE_HEADER {
    WORD    Machine;
    WORD    NumberOfSections;
    DWORD   TimeDateStamp;
    DWORD   PointerToSymbolTable;
    DWORD   NumberOfSymbols;
    WORD    SizeOfOptionalHeader;
    WORD    Characteristics;
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;
```

- `NumberOfSections`：表示当前PE文件拥有的节的数量，在对PE文件添加节时，这里的值要进行相应的增加。
- `SizeOfOptionalHeader`：扩展头大小

#### `IMAGE_OPTIONAL_HEADER`

该结构由固定大小`0xE0`加上数据目录`IMAGE_DATA_DIRECTORY`类型的数组组成。

```
typedef struct _IMAGE_OPTIONAL_HEADER {

    WORD    Magic;
    BYTE    MajorLinkerVersion;
    BYTE    MinorLinkerVersion;
    DWORD   SizeOfCode;
    DWORD   SizeOfInitializedData;
    DWORD   SizeOfUninitializedData;
    DWORD   AddressOfEntryPoint;
    DWORD   BaseOfCode;
    DWORD   BaseOfData;

    DWORD   ImageBase;
    DWORD   SectionAlignment;
    DWORD   FileAlignment;
    WORD    MajorOperatingSystemVersion;
    WORD    MinorOperatingSystemVersion;
    WORD    MajorImageVersion;
    WORD    MinorImageVersion;
    WORD    MajorSubsystemVersion;
    WORD    MinorSubsystemVersion;
    DWORD   Win32VersionValue;
    DWORD   SizeOfImage;
    DWORD   SizeOfHeaders;
    DWORD   CheckSum;
    WORD    Subsystem;
    WORD    DllCharacteristics;
    DWORD   SizeOfStackReserve;
    DWORD   SizeOfStackCommit;
    DWORD   SizeOfHeapReserve;
    DWORD   SizeOfHeapCommit;
    DWORD   LoaderFlags;
    DWORD   NumberOfRvaAndSizes;
    IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];
} IMAGE_OPTIONAL_HEADER32, *PIMAGE_OPTIONAL_HEADER32;
```

- `SizeOfCode`：所有代码节的总和，这个大小是基于文件对齐后的大小，而不是内存对齐。
- `AddressOfEntryPoint`：程序的入口点`OEP`，写壳时就是通过修改程序的入口点，从而运行我们的Stub部分实现对宿主程序的解密。
- `ImageBase`：默认载入基址，在生成程序时如果将编译选项改为可变基址，则这个值是没有用的。获取一个模块的句柄，这个句柄和模块的实际载入基址是一样的值。
- `SectionAlignment`：内存对齐
- `FileAlignment`：文件对齐
- `SizeOfImage`：映像文件的大小，这个数值是基于内存对齐
- `SizeOfHeaders`：PE文件头+节表的总大小，是基于文件对齐的，这个数据比较严格，数值只能是文件对齐大小的整数倍，在打造比较小的PE文件时，可以将文件对齐调整的小一些。
- `NumberOfRvaAndSize`：数据目录的个数，取值为2~16，一般正常的PE文件都是16个，如果多于2个，那么多出的目录存在于固定位置，前面不存在的数据目录会被补0.
- `IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES]`数据目录数组。可以使用下标来访问不同的数据目录。数组中的每个元素都有两个`DWORD`，一个代表指向数据结构的大小，一个代表数据结构在PE文件中的偏移。

#### `IMAGE_DATA_DIRECTORY`

扩展头大小为`0xE0`,这个大小会随着PE文件中目录的个数的不同而改变，但是前面`0x60`个字节大小是固定的。数据目录数组的起始位置位于扩展头的`0x60`处，可以根据系统提供的宏（代表下标）去访问不同的数据目录。

#### `IMAGE_EXPORT_DIRECTORY`

PE文件的导出表，一般存在于Dll文件中。数据目录的RVA指向下面这个结构体

```
typedef struct _IMAGE_EXPORT_DIRECTORY {
    DWORD   Characteristics;
    DWORD   TimeDateStamp;
    WORD    MajorVersion;
    WORD    MinorVersion;
    DWORD   Name;
    DWORD   Base;
    DWORD   NumberOfFunctions;
    DWORD   NumberOfNames;
    DWORD   AddressOfFunctions;
    DWORD   AddressOfNames;
    DWORD   AddressOfNameOrdinals;
} IMAGE_EXPORT_DIRECTORY;
```

- `NumberOfFunctions` 导出函数的总数
- `NumberOfNames`   有名称函数的总数，该值小于等于上面的字段的值
- 特别要注意`AddressOfNameOrdinals` 指向的是一个`WORD`类型的数组，数组的每一个元素都是一个下标，根据这个下标可以去`AddressOfFunctions` 指向的数组中进行取值，获得导出函数的RVA。
- `AddressOfNames` 指向一个有名称导出函数的名称RVA数组，数组中的每一个元素都是一个RVA，根据这个RVA可以得到导出函数的名称字符串，字符串是ASCII类型，以0结尾。
- `AddressOfNames`与`AddressOfNameOrdinals` 这两个RVA指向的数组中元素是一一对应的关系。
- 导出表也可以获取导出的全局变量，在加壳时，就使用Stub部分的导出全局变量对壳运行进行了控制。

#### `IMAGE_IMPORT_DESCRIPTOR`

一个PE中可能使用了多个Dll模块，每个Dll模块使用一个`IMAGE_IMPORT_DESCRIPTOR`结构来描述。

```
typedef struct _IMAGE_IMPORT_DESCRIPTOR {
    union {
        DWORD   Characteristics;
        DWORD   OriginalFirstThunk;
    } DUMMYUNIONNAME;
    DWORD   TimeDateStamp;

    DWORD   ForwarderChain;
DWORD   Name;
    DWORD   FirstThunk;
} IMAGE_IMPORT_DESCRIPTOR;
```

- `Name` 字段表示导入模块的名称，ASCII码
- 在文件中`OriginalFirstThunk`和`FirstThunk`指向相同的结构，但是当PE文件被加载器加载到内存后，`FirstThunk`指向的结构（IAT）会被修复成函数的真正地址，这个结构在数据目录中也有存在。

#### `IMAGE_BASE_RELOCATION`

基址重定位表，描述PE文件中需要修复的地址，PE文件不一定会加载到默认基址，因此一些基于默认基址产生的地址需要进行修复。重定位表有多个，每个重定位表负责描述`0x1000`空间内的重定位信息。

```
typedef struct _IMAGE_BASE_RELOCATION {
    DWORD   VirtualAddress;
    DWORD   SizeOfBlock;
//  WORD    TypeOffset[1];
} IMAGE_BASE_RELOCATION;
```

- `VirtualAddress`：指向描述的`0x1000`空间的起始RVA
- `SizeOfBlock`：`IMAGE_BASE_RELOCATION`加上`TypeOffset`数组的总大小，根据这个大小减去`IMAGE_BASE_RELOCATION`的大小就可以算出`TypeOffset`数组中元素的个数
- `TypeOffset`：一个`WORD`类型的数组。数组的每个元素指向描述空间中需要修复的数据，这是一个偏移量，相对于`VirtualAddress`。这里需要注意一下。在对需要修复的数据进行修复时，会根据实际加载基址减去默认加载基址，得出一个增量，然后使用这个增量与修复数据相加得到修复后的数据。
- 在设计加壳程序时，会对Stub部分的重定位信息进行修复，由于Stub部分存在于宿主程序，节的RVA发生了变化，因此还需要使用Stub在宿主程序中的RVA减去原RVA获取一个增量，使用该增量与修复数据进行相加才能得到正确的地址数据。

#### 节表信息

描述PE文件中节的信息

```
typedef struct _IMAGE_SECTION_HEADER {
    BYTE    Name[IMAGE_SIZEOF_SHORT_NAME];
    union {
            DWORD   PhysicalAddress;
            DWORD   VirtualSize;
    } Misc;
    DWORD   VirtualAddress;
    DWORD   SizeOfRawData;
    DWORD   PointerToRawData;
    DWORD   PointerToRelocations;
    DWORD   PointerToLinenumbers;
    WORD    NumberOfRelocations;
    WORD    NumberOfLinenumbers;
    DWORD   Characteristics;
} IMAGE_SECTION_HEADER;
```

- `VirtualSize`：描述节的实际尺寸。
- `VirtualAddress`：描述节的起始RVA
- `SizeOfRawData`:描述在文件对齐后的大小
- `PointerToRawData;`：节的FOA
- `Characteristics`:节的属性，在PE文件被执行时，这个属性规定了节在内存中可读可写可执行等属性，在进行加壳时，会对宿主文件被加密部分进行属性更改，改为可写，这样才能在程序运行时对宿主程序进行解密。

## 总体设计

- 流程图

![](https://bbs.kanxue.com/upload/attach/201312/586874_9c4815c318ee7f35.png)

- 不足

在开始时并没有将界面与逻辑分开，PE文件只负责加载文件，并获取文件在内存中的指针，将指针传给界面，界面拿到文件缓存指针后，根据指针获取要显示的信息，然后在将信息显示到界面。

- 改进

在后面的界面设计中，将数据的获取放在PE文件操作类中，通过一个结构体传递数据，这样界面只负责显示数据即可。

对于文件的修改保存也是这样，获取界面的信息，保存到一个结构体中，然后调用PE文件操作类的接口对数据进行保存，这时的界面只是负责获取数据。在使用PE文件的结构体时，注意使用指针，不要使用值拷贝，这样在修改时只需要保存文件即可，不需要在修改内存中的信息。

## 详细设计

- 文件保存

文件保存时，如果保存的是一段连续数据，比如导入表信息（一个结构体），目录信息，可以调用同一个方法：

```
/**
*@brief 将指针指向的内容保存到文件中
*/
DWORD SavePointerContent(LPVOID pContent,DWORD dwSize);
```

传入数据指针，然后传入保存数据的大小，方法内部会根据传入指针和保存的文件指针进行计算，获取保存数据在文件中的偏移，然后写入文件。

对于非连续数据则需要单独写一个方法，如果调用`SavePointer...` 方法，则存一段信息就要调用一次方法，会对文件进行多次打开关闭操作。

当初考虑过打开的文件句柄不关闭，每次写入只做一次刷新操作。但是这样不太好，文件一直是打开的。

- 关联信息的修改

在对PE文件进行修改时，往往修改一个地方一些其他的关联信息也需要修改，如果只修改了一个地方，关联信息没有改变，则会出错。比如给PE文件添加一个节，此时PE文件中需要改变的信息：

![](https://bbs.kanxue.com/upload/attach/201312/586874_c8d6e40799e524e1.png)

可选头的`SizeOfImage`字段也需要进行相应的修改。

- 文件位置计算器（FLC）

`RVA` 转换 `FOA` ：遍历节表向量，用节的起始RVA加上节的实际大小`Misc.VirtualSize`得到节的范围，然后判断给出的RVA是否落在这个范围中。如果在这个范围中则使用`RVA – 节的起始RVA` 获得偏移，然后再加上该节在文件中的起始FOA得到给定RVA 对应的FOA

有两个地方需要注意一下

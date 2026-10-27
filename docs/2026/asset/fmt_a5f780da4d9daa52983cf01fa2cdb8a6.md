
```
#define IMAGE_NT_OPTIONAL_HDR32_MAGIC      0x10b    //32位PE

#define IMAGE_NT_OPTIONAL_HDR64_MAGIC      0x20b    //64位PE
```

`AddressOfEntryPoint` 指定了程序启动以后开始运行的第一行代码相对于基地址的偏移。

`ImageBase` 指定了程序加载到内存中的基地址，对于 exe 的主程序文件通常是 `0x40000`。

`SectionAlignment` 指定了在内存中的节区对齐大小，通常是 `0x1000`。

`FileAlignment` 指定了在磁盘中节区对齐大小，通常是 `0x400` 或者 `0x1000`。

为了寻址方便，操作系统将文件装载进内存的时候，会根据 `SectionAlignment` 来进行节区对齐，也就是说如果这个节区中的数据不足 `SectionAlignment` 个，操作系统就会在后面补 0，为了让下一个节区装置进内存的地址刚好是 `SectionAlignment` 的整数倍。

而为了节约磁盘空间，文件保存在磁盘中的时候，会以 `FileAlignment` 指定的值进行对齐，这个值正常情况下都小于等于 `SectionAlignment`，如下图

![](https://bbs.kanxue.com/upload/attach/202110/835440_P7WKWVMG33DE57B.jpg)

可以看到，由于 `FileAlignment` 和 `SectionAlignment` 的大小不同，在文件中，每个节区都是 `0x400` 的整数倍保存，而在内存中则需要填入 NULL 来扩展到 `0x1000` 的整数倍。

`SizeOfImage` 指定了将整个 PE 程序张载到内存中以后所占内存的总大小，它是 `SectionAlignment` 的整数倍。

`SizeOfHeader` 指定了整个 PE 头的大小，该值是 `FileAlignment` 的整数倍。

### Subsystem

`Subsystem` 用于区分驱动文件与可执行文件，定义如下

```
#define IMAGE_SUBSYSTEM_NATIVE               1   // 驱动文件

#define IMAGE_SUBSYSTEM_WINDOWS_GUI          2   // GUI文件

#define IMAGE_SUBSYSTEM_WINDOWS_CUI          3   // CUI控制台文件
```

### DllCharacteristics

`DllCharacteristics` 通常用来说明文件是否可以重定位，当它与下面这个宏定义 & 以后如果为 1 则可重定位

```
#define IMAGE_DLLCHARACTERISTICS_DYNAMIC_BASE 0x0040    //文件可以重定位
```

### DataDirectory

`DataDirectory` 是由 `IMAGE_DATA_DIRECTORY` 结构体组成的数组，数组一共 16 项，在文档中有如下定义

```
#define IMAGE_NUMBEROF_DIRECTORY_ENTRIES    16
```

数组每一项都描述特定的一张表的数据，如下

![](https://bbs.kanxue.com/upload/attach/202110/835440_CZUWGAD5KBUNHXH.jpg)

其中，`IMAGE_DATA_DIRECTORY` 的定义如下

```
typedef struct _IMAGE_DATA_DIRECTORY {

    DWORD   VirtualAddress;

    DWORD   Size;

} IMAGE_DATA_DIRECTORY, *PIMAGE_DATA_DIRECTORY;
```

`VirtualAddress` 中保存了在内存中这张表的偏移地址，之后再详细说这里面的几张表，现在先略过。

### NT 头解析代码

根据上面的内容就可以写出下面的代码对 NT 头进行解析

```
void PEParse::PrintNtHeadInfo()

{

	printf("====================The NtHeaders Information================\n");



	printf("\n================The FileHead Information==============\n");

	//machine info

	WORD dwMachine = this->pFileHead->Machine;

	if (dwMachine & IMAGE_FILE_MACHINE_I386)	printf("The app can run in Intel 386\n");

	if (dwMachine & IMAGE_FILE_MACHINE_IA64)	printf("The app can run in Intel 64\n");

	if (dwMachine & IMAGE_FILE_MACHINE_AMD64)   printf("The app can rum in AMD 64\n");



	printf("The app have %d sections\n", this->pFileHead->NumberOfSections);

	printf("The size of optional header is 0x%X\n", this->pFileHead->SizeOfOptionalHeader);



	WORD dwFileCharacter = this->pFileHead->Characteristics;

	if (dwFileCharacter & IMAGE_FILE_RELOCS_STRIPPED) printf("The app does not have relocation information.\n");

	if (dwFileCharacter & IMAGE_FILE_EXECUTABLE_IMAGE) printf("The app is executable.\n");

	if (dwFileCharacter & IMAGE_FILE_LARGE_ADDRESS_AWARE) printf("The app can handle the addresss > 2gb.\n");

	if (dwFileCharacter & IMAGE_FILE_32BIT_MACHINE) printf("The app can only run 32 bit machine.\n");

	if (dwFileCharacter & IMAGE_FILE_SYSTEM) printf("The app is system file.\n");

	if (dwFileCharacter & IMAGE_FILE_DLL) printf("The app is dll file.\n");

	printf("================The FileHead Information==============\n");



	printf("\n==========The Optional FileHead Information===========\n");

	WORD dwMagic = this->pOptionHead->Magic;

	if (dwMagic & IMAGE_NT_OPTIONAL_HDR32_MAGIC) printf("The app is PE file in 32 bit\n");

	else if (dwMagic & IMAGE_NT_OPTIONAL_HDR64_MAGIC) printf("The app is PE file in 64 bit\n");

	printf("Address of entrypoint is 0x%X\n", this->pOptionHead->AddressOfEntryPoint);

	printf("Image Base is 0x%X\n", this->pOptionHead->ImageBase);

	printf("Section Alignment Size is 0x%X\n", this->pOptionHead->SectionAlignment);

	printf("File Alignment Size is 0x%X\n", this->pOptionHead->FileAlignment);

	printf("Size of Image is 0x%X\n", this->pOptionHead->SizeOfImage);

	printf("Size of Headers is 0x%X\n", this->pOptionHead->SizeOfHeaders);

	WORD dwSubSystem = this->pOptionHead->Subsystem;

	printf("The Subsystem is");

	switch (dwSubSystem)

	{

	case IMAGE_SUBSYSTEM_NATIVE:

		printf(" navite\n");

		break;

	case IMAGE_SUBSYSTEM_WINDOWS_GUI:

		printf(" GUI system\n");

		break;

	case IMAGE_SUBSYSTEM_WINDOWS_CUI:

		printf(" CUI system(console or dll)\n");

		break;

	}

	if (this->pOptionHead->DllCharacteristics & IMAGE_DLLCHARACTERISTICS_DYNAMIC_BASE) printf("The file can remove\n");

	else printf("the file can not remove.\n");

	printf("==========The Optional FileHead Information===========\n");



	printf("\n====================The NtHeaders Information================\n");

}
```

程序的运行结果如下

![](https://bbs.kanxue.com/upload/attach/202110/835440_7YRV6WF2FDB75PD.jpg)

### 节区头

接下来是节区头，每个节区都有一个节区头，节区头里面保存了该节区的一些信息，节区头定义如下

```
#define IMAGE_SIZEOF_SHORT_NAME              8



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

} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;
```

`Name` 数组保存了该节区的名字，最多有 8 个

`VirtualSize` 保存的是该节区在内存中占用的大小

`VirtualAddress` 保存的是该节区在内存中的偏移地址

`SizeOfRawData` 保存的是该节区在文件的所占的大小

`PointerToRawData` 保存的是该节区在文件中的偏移地址

`Characteristics` 保存了这个节区的属性，常见的如下

```
#define IMAGE_SCN_CNT_CODE                   0x00000020    //该节区包含代码

#define IMAGE_SCN_CNT_INITIALIZED_DATA       0x00000040    //该节区包含初始化数据

#define IMAGE_SCN_CNT_UNINITIALIZED_DATA     0x00000080    //该节区包含未初始化数据

#define IMAGE_SCN_MEM_EXECUTE                0x20000000    //该节区是可执行的

#define IMAGE_SCN_MEM_READ                   0x40000000    //该节区是可读的

#define IMAGE_SCN_MEM_WRITE                  0x80000000    //该节区是可写的
```

### 节区解析代码

对节区进行解析的代码如下

```
void PEParse::PrintSectionInfo()

{

	WORD dwSectionNum = this->pFileHead->NumberOfSections, i = 0, j = 0;

	char szSectionName[10] = { 0 };

	DWORD dwSectionCharacter = 0;



	printf("===================The section information===================\n");

	for (i = 0; i < dwSectionNum; i++)

	{

		for (j = 0; j < IMAGE_SIZEOF_SHORT_NAME; j++)

		{

			szSectionName[j] = 0;

			szSectionName[j] = this->pSectionHead[i].Name[j];

		}

		printf("The %d section information:\n", i);

		printf("The section name is %s\n", szSectionName);

		printf("The virtual size of the section is 0x%X\n", this->pSectionHead[i].Misc.VirtualSize);

		printf("The virtual address of the section is 0x%X\n", this->pSectionHead[i].VirtualAddress);

		printf("The section size of raw data is 0x%X\n", this->pSectionHead[i].SizeOfRawData);

		printf("The section pointer to raw data is 0x%X\n", this->pSectionHead[i].PointerToRawData);



		dwSectionCharacter = this->pSectionHead[i].Characteristics;

		if (dwSectionCharacter & IMAGE_SCN_CNT_CODE) printf("The section have code.\n");

		if (dwSectionCharacter & IMAGE_SCN_CNT_INITIALIZED_DATA) printf("The section have initialized data\n");

		if (dwSectionCharacter & IMAGE_SCN_CNT_UNINITIALIZED_DATA) printf("The section have uninitialized data\n");

		if (dwSectionCharacter & IMAGE_SCN_MEM_READ) printf("The section can read\n");

		if (dwSectionCharacter & IMAGE_SCN_MEM_WRITE) printf("The section can write\n");

		if (dwSectionCharacter & IMAGE_SCN_MEM_EXECUTE) printf("The section can execute\n");

		printf("\n");

	}

	printf("\n=================The section information===================\n");

}
```

程序运行结果如下

![](https://bbs.kanxue.com/upload/attach/202110/835440_XAEHH588PNPDUR2.jpg)

![](https://bbs.kanxue.com/upload/attach/202110/835440_PTNF8WJQ254D3SB.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2022-2-10 21:50

被1900编辑，原因：

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)

上传的附件：

- [PE结构.pdf](javascript:void(0);)

  （373.42kb，76次下载）

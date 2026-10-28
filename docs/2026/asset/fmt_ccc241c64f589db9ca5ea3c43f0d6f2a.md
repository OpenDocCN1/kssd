
# [原创]常见进程注入的实现及内存dump分析——内存模块

# 前言

> 在[上一篇](https://bbs.pediy.com/thread-224241.htm)文章中，提到了反射式注入的缺点——包含大量的RWX内存块，使得在分析的时候，可以轻易发现异常的内存。在Git上发现一个项目，可以避免这种情况，就是今天要说的内存模块（Memroy Module），严格来讲并不是注入，这个的目的是由于`LoadLibrary`函数只能加载硬盘中的DLL，而不能去加载内存中的DLL，所以提出了这个概念。本帖将在上篇文章中所实现的可执行项目进行修改，来达到目的。
>
> 首先，先介绍下将要用到的一些理论概念：
>
> ```
> 分配粒度：表示每次请求内存的时候最小给分配多少。分配内存的大小，一定是该值的整数倍。
> ```
>
> ```
> 区段对齐：PE文件被映射到内存中时，区块总是至少从一个页边界开始。x86兄台你中，PE文件区块的内存对齐值一般等于0x1000h,每个区块按1000的倍数的内存偏移开始。
> ```
>
> ```
> 一个进程地址空间中的页面可以是空闲的、保留的，或者提交的。应用程序可以首先保留地址空间，然后提交该地址空间中的页面。
>
> 试图访问已保留的内存，将导致内存违例，因为该页面尚未被映射到任何可解析此引用的内存介质中。
>
> 提交的页面：当被访问的时候，最终被转译至物理内存中的有效页面。
> ```
>
> 实际上，这里与反射式注入不同的地方，只在于反射式注入是将代码统一的加载到同一个RWX块，而内存模块是针对不同模块修改不同的权限，并释放掉没有用的区块，从而使目标内存更像正常内存，所以本贴的重点在于映射截断，而不去管导入表和重定位表等。

# 环境

> ### OS：Windows 10 PRO 1709
>
> ### IDE：Visual Studio 2015 Community
>
> ### 语言：Visual C++

# 实现

> 1、模块（
>
> Module
>
> ）的定义：Module在我看来就是一些信息的集合，包括API地址，PE文件地址，加载到的地址，系统信息等。
>
> ```
> typedef struct _MemoryModule
>
> {
> 	ULONG_PTR bufferAddress;                          //要加载的DLL在内存中的地址
> 	ULONG_PTR baseAddress;                           //要加载到的地址
> 	PIMAGE_NT_HEADERS old_header;            //buffer中的NT头
> 	PIMAGE_NT_HEADERS header;                   //新地址的NT头
> 	VIRTUALALLOC mVirutalAlloc;                      //VirtualAlloc函数地址
> 	VIRTUALFREE mVirtualFree;                        //VirtualFree函数地址
> 	VIRTUALPROTECT mVirtualProtect;            //VirtualProtect函数地址
> 	SYSTEM_INFO sysInfo;                                //系统信息
> }MemoryModule,*PMemoryModule;
> ```
>
> 以上的API函数，依旧使用反射式注入帖子中的方法来获取地址，新增的函数只需添加Hash值即可。
>
> 2、区段拷贝：先按照总的节空间，预留内存，后根据节属性修改。分配内存：
>
> ```
> //计算节占用的总空间，并按照页面大小对齐
> for (int i = 0; i < (((PIMAGE_NT_HEADERS)uiHeaderValue)->FileHeader.NumberOfSections); i++, section++)
> {
> 	DWORD endOfSection;
> 	if (section->SizeOfRawData == 0)
> 		//如果节中没有数据，则默认按照粒度分配一节
> 		endOfSection = section->VirtualAddress + optionalSectionSize;
> 	else
> 		//有数据，则加上正常的数据长度
> 		endOfSection = section->VirtualAddress + (section->SizeOfRawData);
> 	if (endOfSection > lastSectionEnd)
> 		lastSectionEnd = endOfSection;
> }
> //sysInfo->dwPageSize=0x1000 ------4K
> GetNativeSystemInfo(&sysInfo);//获取系统信息，来获得页面大小
> //Git上的MemoryModule代码中的对齐函数都为内联模式，不利于调试，这里我修改成正常的函数调用
> //param1:内存中的整个PE映像大小	param2：页面大小
> //计算镜像对齐的大小
> alignedImageSize = AlignValueUp(((PIMAGE_NT_HEADERS)uiHeaderValue)->OptionalHeader.SizeOfImage, sysInfo.dwPageSize);
> //所有节加起来后最后的地址对齐后一定和使用SizeOfImage对齐的大小是相同的
> if (alignedImageSize != AlignValueUp(lastSectionEnd, sysInfo.dwPageSize))
> 	return NULL;
> //先按照镜像建议的基址进行空间分配保留的的内存。(在Git上的项目中，是MEM_RESERVE|MEM_COMMIT)
> code = pVirtualAlloc((LPVOID)(((PIMAGE_NT_HEADERS)uiHeaderValue)->OptionalHeader.ImageBase),
> 		alignedImageSize,MEM_RESERVE ,PAGE_READWRITE,NULL);
> //如果镜像占用的位置被占用，则选择其他位置。
> if (code == NULL)
> {
> 	code = pVirtualAlloc(NULL,alignedImageSize,MEM_RESERVE ,PAGE_READWRITE,NULL);
> 	if (code == NULL)
> 		return;
> }
> ```
>
> 3、拷贝PE头
>
> ```
> //提交内存
> uiHeaderValue = uiLibraryAddress + ((PIMAGE_DOS_HEADER)uiLibraryAddress)->e_lfanew;
> header = pVirtualAlloc(code, ((PIMAGE_NT_HEADERS)uiHeaderValue)->OptionalHeader.SizeOfHeaders,
> 		MEM_COMMIT,PAGE_READWRITE,NULL);
> uiValueA = ((PIMAGE_NT_HEADERS)uiHeaderValue)->OptionalHeader.SizeOfHeaders;//所有头+节表的大小
> uiValueB = uiLibraryAddress;//DLL的起始地址，即缓冲区的起始地址
> uiValueC = code;//dll将被加载的地址的起始地址
> //复制头和节表的数据到新开辟的缓冲区
> while (uiValueA--)
> 	*(BYTE *)uiValueC++ = *(BYTE *)uiValueB++;
> ```
>
> 4、节拷贝（仅仅为拷贝，内存属性先不管）
>
> ```
> BOOL CopySections(PMemoryModule mModule)
> {
> 	int i, section_size;
> 	ULONG_PTR dest;
> 	PIMAGE_SECTION_HEADER section = IMAGE_FIRST_SECTION(mModule->header); 
> 	ULONG_PTR ValueA;
> 	ULONG_PTR ValueB;
> 	ULONG_PTR ValueC;
> 	for (i = 0; i < mModule->header->FileHeader.NumberOfSections; i++, section++)
> 	{
> 		//在DLL中，当前节不含有数据，但是可能定义未初始化的数据
> 		if (section->SizeOfRawData == 0)
> 		{
> 			//内存中节的对齐粒度
> 			section_size = mModule->old_header->OptionalHeader.SectionAlignment;
> 			if (section_size > 0)
> 			{
> 				dest = mModule->mVirutalAlloc(mModule->baseAddress+ section->VirtualAddress, section_size, MEM_COMMIT, PAGE_READWRITE, NULL);//mModule->flProtect);
> 				if (dest == NULL)
> 					return FALSE;
> 				//始终保持页对齐，以上分配的内存，正好为一页
> 				dest = mModule->baseAddress + section->VirtualAddress;
> 				//64位模式下，这里截断成32位模式
> 				section->Misc.PhysicalAddress = (DWORD)((uintptr_t)dest & 0xffffffff);
> 			}
> 			//section 为空
> 			continue;
> 		}
> 		//节中含有数据
> 		dest = mModule->mVirutalAlloc(mModule->baseAddress + section->VirtualAddress, section->SizeOfRawData, MEM_COMMIT, PAGE_READWRITE, NULL);
> 		ValueA = section->SizeOfRawData;//节的大小
> 		ValueB = mModule->bufferAddress + section->PointerToRawData;//数据的起始地址
> 		ValueC = dest;//数据将被拷贝到到的地址
> 		 //复制头和节表的数据到新开辟的缓冲区
> 		while (ValueA--)
> 			*(BYTE *)ValueC++ = *(BYTE *)ValueB++;
> 		if (dest == NULL)
> 			return FALSE;
> 		dest = mModule->baseAddress + section->VirtualAddress;
> 		//这里使用PhysicalAddre这个Dword值存储64位地址，会导致地址截断位32位，存储的地址位节地址VA
> 		section->Misc.PhysicalAddress = (DWORD)((uintptr_t)dest & 0xffffffff);
> 	}//end for
> 	return TRUE;
> }
> ```
>
> 说明：Misc.PhysclAddress这个DWORD字段的作用：在MSDN给出的解释是“
>
> The file address.
>
> ”，但是我通过工具查看，如图：
>
> ![](https://bbs.kanxue.com/upload/attach/201802/703263_DUUUC5PRZ2RNCEM.jpg)
>
> 我们可以看到，这个字段和大小是相同的，但是在内存中为该块装载到内存的RVA。在OBJ中，该字段是没有意义的。
>
> 5、修改前的地址恢复及一些准备
>
> ```
> BOOL FinalizeSections(PMemoryModule mMemory)
> {
> 	int i;
> 	PIMAGE_SECTION_HEADER section = IMAGE_FIRST_SECTION(mMemory->header);
> 	//imageOffset的值，由于PhysicalAddress的值被截断成32位，需要恢复为64位，这里的值就是镜像加载到内存的前32位（被截断掉的32位）
> 	uintptr_t imageOffset = ((uintptr_t)mMemory->header->OptionalHeader.ImageBase & 0xffffffff00000000);
> 	SECTIONFINALIZEDATA sectionData;
> 	//恢复为64位
> 	sectionData.address = (LPVOID)((uintptr_t)section->Misc.PhysicalAddress | imageOffset);
> 	//节首地址
> 	sectionData.alignedAddress = AlignAddressDown(sectionData.address, mMemory->sysInfo.dwPageSize);
> 	//节大小
> 	sectionData.size = GetRealSectionSize(mMemory, section, mMemory->header);
> 	//属性
> 	sectionData.characteristics = section->Characteristics;
> 	sectionData.last = FALSE;
> 	section++;
> 	for (i = 1; i<mMemory->header->FileHeader.NumberOfSections; i++, section++) {
> 		LPVOID sectionAddress = (LPVOID)((uintptr_t)section->Misc.PhysicalAddress | imageOffset);
> 		LPVOID alignedAddress = AlignAddressDown(sectionAddress, mMemory->sysInfo.dwPageSize);
> 		SIZE_T sectionSize = GetRealSectionSize(mMemory, section, mMemory->header);
> 		//当前把一大节以第一个节的权限为准（需要优化）
> 		//确保在当前页中
> 		if (sectionData.alignedAddress == alignedAddress || (uintptr_t)sectionData.address + sectionData.size >(uintptr_t) alignedAddress) {
> 			// Section shares page with previous
> 			//这里我觉得有一个判断就可以-_-!
> 			if ((section->Characteristics & IMAGE_SCN_MEM_DISCARDABLE) == 0 || (sectionData.characteristics & IMAGE_SCN_MEM_DISCARDABLE) == 0) {
> 				sectionData.characteristics = (sectionData.characteristics | section->Characteristics) & ~IMAGE_SCN_MEM_DISCARDABLE;
> 			}
> 			else {
> 				sectionData.characteristics |= section->Characteristics;
> 			}
> 			sectionData.size = (((uintptr_t)sectionAddress) + ((uintptr_t)sectionSize)) - (uintptr_t)sectionData.address;
> 			continue;
> 		}
>                 //这个函数为真正的权限修改函数
> 		if (!FinalizeSection(mMemory, &sectionData)) {
> 			return FALSE;
> 		}
> 		sectionData.address = sectionAddress;
> 		sectionData.alignedAddress = alignedAddress;
> 		sectionData.size = sectionSize;
> 		sectionData.characteristics = section->Characteristics;
> 	}
> 	sectionData.last = TRUE;
> 	if (!FinalizeSection(mMemory, &sectionData)) {
> 		return FALSE;
> 	}
> 	return TRUE;
> }
> ```
>
> 6、属性修改

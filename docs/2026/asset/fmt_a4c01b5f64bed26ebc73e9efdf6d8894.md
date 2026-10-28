
```
// uiValueB ：导入表地址

uiValueB = (ULONG_PTR)&((PIMAGE_NT_HEADERS)uiHeaderValue)->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT];

//基地址+RVA即导入表描述符的地址VA

uiValueC = (uiBaseAddress + ((PIMAGE_DATA_DIRECTORY)uiValueB)->VirtualAddress);

//链接库名字

while (((PIMAGE_IMPORT_DESCRIPTOR)uiValueC)->Name)

{

	//使用LoadLibraryA将需要的模块加载到内存

	uiLibraryAddress = (ULONG_PTR)pLoadLibraryA((LPCSTR)(uiBaseAddress + ((PIMAGE_IMPORT_DESCRIPTOR)uiValueC)->Name));

	//指向INT的IMAGE_THUNK_DATA的VA

	uiValueD = (uiBaseAddress + ((PIMAGE_IMPORT_DESCRIPTOR)uiValueC)->OriginalFirstThunk);

	//要导入IAT的IMAGE_THUNK_DATA结构体

	uiValueA = (uiBaseAddress + ((PIMAGE_IMPORT_DESCRIPTOR)uiValueC)->FirstThunk);

	// 迭代函数，如果没有名，则获取序号

	while (DEREF(uiValueA))

	{
                //在调试过程中发现都是获取的函数序号

		// sanity check uiValueD as some compilers only import by FirstThunk

		if (uiValueD && ((PIMAGE_THUNK_DATA)uiValueD)->u1.Ordinal & IMAGE_ORDINAL_FLAG)

		{

			uiExportDir = uiLibraryAddress + ((PIMAGE_DOS_HEADER)uiLibraryAddress)->e_lfanew;

			uiNameArray = (ULONG_PTR)&((PIMAGE_NT_HEADERS)uiExportDir)->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT];

			uiExportDir = (uiLibraryAddress + ((PIMAGE_DATA_DIRECTORY)uiNameArray)->VirtualAddress);

			uiAddressArray = (uiLibraryAddress + ((PIMAGE_EXPORT_DIRECTORY)uiExportDir)->AddressOfFunctions);

			uiAddressArray += ((IMAGE_ORDINAL(((PIMAGE_THUNK_DATA)uiValueD)->u1.Ordinal) - ((PIMAGE_EXPORT_DIRECTORY)uiExportDir)->Base) * sizeof(DWORD));

			DEREF(uiValueA) = (uiLibraryAddress + DEREF_32(uiAddressArray));

		}

		else

		{

			uiValueB = (uiBaseAddress + DEREF(uiValueA));

			DEREF(uiValueA) = (ULONG_PTR)pGetProcAddress((HMODULE)uiLibraryAddress, (LPCSTR)((PIMAGE_IMPORT_BY_NAME)uiValueB)->Name);

		}

		uiValueA += sizeof(ULONG_PTR);

		if (uiValueD)//INT

			uiValueD += sizeof(ULONG_PTR);

	}

	uiValueC += sizeof(IMAGE_IMPORT_DESCRIPTOR);

}
```

同样，关于导出表的部分没有详细注释，已经在上篇帖子中有详细的介绍。

- 处理重定位表，由于基址改变，所以程序中的一些直接寻址等会出问题，所以要更改重定向表。

![](https://bbs.kanxue.com/upload/attach/201801/703263_N823N3KSHZ8APEN.jpg)

接下来需要用到的重定位表的关系

代码如下：

```
//实际装载和建议装载的偏移，原重定位表中的值是以程序建议的装载地址为基址

uiLibraryAddress = uiBaseAddress - ((PIMAGE_NT_HEADERS)uiHeaderValue)->OptionalHeader.ImageBase;//程序建议的装载地址																						

uiValueB = (ULONG_PTR)&((PIMAGE_NT_HEADERS)uiHeaderValue)->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC];

if (((PIMAGE_DATA_DIRECTORY)uiValueB)->Size)//重定位表大小

{

	//重定位表的地址

	uiValueC = (uiBaseAddress + ((PIMAGE_DATA_DIRECTORY)uiValueB)->VirtualAddress);

	while (((PIMAGE_BASE_RELOCATION)uiValueC)->SizeOfBlock)//重定位块的大小

	{

		//重定位内存页的起始RVA

		uiValueA = (uiBaseAddress + ((PIMAGE_BASE_RELOCATION)uiValueC)->VirtualAddress);

		//重定位块中的项数(整个块的大小减去结构体的大小，得到重定位项的总大小，除以每个重定位项的大小)																			

		uiValueB = (((PIMAGE_BASE_RELOCATION)uiValueC)->SizeOfBlock - sizeof(IMAGE_BASE_RELOCATION)) / sizeof(IMAGE_RELOC);

		//重定位块的第一项

		uiValueD = uiValueC + sizeof(IMAGE_BASE_RELOCATION);

		//遍历重定位项

		while (uiValueB--)

		{

			//重定位项的高四位代表此重定位项的类型

			if (((PIMAGE_RELOC)uiValueD)->type == IMAGE_REL_BASED_DIR64)

				*(ULONG_PTR *)(uiValueA + ((PIMAGE_RELOC)uiValueD)->offset) += uiLibraryAddress;

			else if (((PIMAGE_RELOC)uiValueD)->type == IMAGE_REL_BASED_HIGHLOW)

				*(DWORD *)(uiValueA + ((PIMAGE_RELOC)uiValueD)->offset) += (DWORD)uiLibraryAddress;

			else if (((PIMAGE_RELOC)uiValueD)->type == IMAGE_REL_BASED_HIGH)

				*(WORD *)(uiValueA + ((PIMAGE_RELOC)uiValueD)->offset) += HIWORD(uiLibraryAddress);

			else if (((PIMAGE_RELOC)uiValueD)->type == IMAGE_REL_BASED_LOW)

				*(WORD *)(uiValueA + ((PIMAGE_RELOC)uiValueD)->offset) += LOWORD(uiLibraryAddress);

			//下一个重定位项

			uiValueD += sizeof(IMAGE_RELOC);

		}

		//下一个重定位块

		uiValueC = uiValueC + ((PIMAGE_BASE_RELOCATION)uiValueC)->SizeOfBlock;

	}

}
```

- 调用程序入口点，使其执行`DllMain()`，并传递消息为 Dll 的状态为`DLL_PROCESS_ATTACH`（这个消息在上篇文章中有讲到）

```
uiValueA = (uiBaseAddress + ((PIMAGE_NT_HEADERS)uiHeaderValue)->OptionalHeader.AddressOfEntryPoint);

// We must flush the instruction cache to avoid stale code being used which was updated by our relocation processing.

pNtFlushInstructionCache((HANDLE)-1, NULL, 0);

((DLLMAIN)uiValueA)((HINSTANCE)uiBaseAddress, DLL_PROCESS_ATTACH, NULL);//调用入口点
```

## 分析：

在这种的注入是实现中，很少从外部导入函数，且使用了目标进程的部分导入库和函数，所以在IDA的导入中没有什么有价值的信息。不过，回忆整个流程，我们会发现这种注入有特别的地方，如获取`PEB`，如图，双重循环获取系统函数等，而且，这种注入由于需要修复的重定位表，也会使用双重循环。

![](https://bbs.kanxue.com/upload/attach/201801/703263_GJ2C78Y7EWPFZW5.jpg)

在导出函数中，由于在注射器中会通过导出表来获取反射函数的地址，所以导出表中会有一个反射函数，且加载功能都是在反射函数中进行的。

![](https://bbs.kanxue.com/upload/attach/201801/703263_RVX4M2H3NQPM3JM.jpg)

从内存分布看，由于都是新开辟的空间，且需要执行代码，所以权限都为RWX，如果查看内存，小的那段的开头，一定是MZ。

![](https://bbs.kanxue.com/upload/attach/201801/703263_J7GVMCJ94U7EA6M.jpg)

将小的那段内存dump出来，虽然大小和原DLL有稍微的不同，但直接拖到IDA是可以进行分析的，因为那段内存就是dll本身。

# 最后

## 全部源码地址：[cfeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6e0N6h3c8G2h3X3S2S2L8X3N6W2i4K6u0r3f1s2u0G2j5$3g2K6M7@1W2F1K9X3g2U0N6r3W2G2L8R3`.`.](https://bbs.kanxue.com/elink@e6aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6e0N6h3c8G2h3X3S2S2L8X3N6W2i4K6u0r3f1s2u0G2j5$3g2K6M7@1W2F1K9X3g2U0N6r3W2G2L8R3`.`.)

# 参考

- 代码：[1efK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6r3g2H3K9r3g2F1k6X3g2%4k6i4u0Q4x3V1k6d9k6h3k6D9k6h3y4@1K9i4k6W2c8p5I4x3d9h3&6B7k6h3y4@1K9h3!0F1](https://bbs.kanxue.com/elink@0daK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6r3g2H3K9r3g2F1k6X3g2%4k6i4u0Q4x3V1k6d9k6h3k6D9k6h3y4@1K9i4k6W2c8p5I4x3d9h3&6B7k6h3y4@1K9h3!0F1)
- 《Windows PE权威指南》
- 《深入解析Windows操作系统》
- 《加密与解密》

![](https://bbs.kanxue.com/upload/attach/201801/703263_VVT7EHF7C7KSS36.jpg)

![](https://bbs.kanxue.com/upload/attach/201801/703263_4X96K9UA7G9S4HB.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)

上传的附件：

- [`ReflectiveDLLInjection.7z`](javascript:void(0);)（485.82kb，250次下载）
- [`ReflectiveDLLPEForm.7z`](javascript:void(0);)（9.61kb，211次下载）
- [`PEB.jpg`](javascript:void(0);)（209.88kb，106次下载）
- [`PE - Copy.jpg`](javascript:void(0);)（253.47kb，93次下载）

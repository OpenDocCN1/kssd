
### 分析之后总结的流程图：

![](https://bbs.kanxue.com/upload/attach/201009/301973_s6psina5g0qj2jo.jpg)

### 还原C代码：

前面分析注释了很多代码，这个没写注释了。

`MyGetProcAddress` 中没有对第二个参数进行判断，我发现我看过的所有的自己实现的 `GetProcAddress` 都判断第二个参数是否为 `NULL`，如果是 `NULL` 则函数返回，这块严格的讲是有问题的。因为如果dll的 `DEF` 中明确定义了函数的序号为0，则此时第二个参数传入的0就是有意义的。

```
FARPROC WINAPI MyGetProcAddress(HMODULE hModule, LPCSTR lpProcName)

{

	PIMAGE_DOS_HEADER pDOSHeader;

	PIMAGE_NT_HEADERS pNTHeader;

	PIMAGE_EXPORT_DIRECTORY pExportDir;

	LONG peNtHeaderBaseOffset;

	DWORD dwImage_Export_Directory_RVA;

	DWORD dwImage_Export_Directory_Size;

	DWORD dwAddressOfNamesVA;

	DWORD dwProcOrdinalNumber;

	DWORD dwProcAddressVA;

	int count = 0; //计数

	char dllName[_MAX_FNAME];



	if (NULL == hModule)

		return 0;



	pDOSHeader = (PIMAGE_DOS_HEADER)hModule;

	peNtHeaderBaseOffset = pDOSHeader->e_lfanew;

	pNTHeader = (PIMAGE_NT_HEADERS)(hModule + peNtHeaderBaseOffset);

	dwImage_Export_Directory_RVA = pNTHeader->OptionalHeader.DataDirectory[0].VirtualAddress;

	dwImage_Export_Directory_Size = pNTHeader->OptionalHeader.DataDirectory[0].Size;



	pExportDir = (PIMAGE_EXPORT_DIRECTORY)(hModule + dwImage_Export_Directory_RVA);

	if ((DWORD)lpProcName > 0x10000)

	{

		if (pExportDir->AddressOfFunctions > 0)

		{

			dwAddressOfNamesVA = pExportDir->AddressOfNames + (DWORD)hModule;

			do 

			{

				if (!strcmp(lpProcName, (const char*)*(DWORD*)((DWORD)hModule + dwAddressOfNamesVA)))

					goto _jmp;

				count++;

				dwAddressOfNamesVA += 4;

			} while ((DWORD)count < pExportDir->NumberOfNames);

			goto _whenend;

_jmp:		

			dwProcOrdinalNumber = *(DWORD*)((pExportDir->AddressOfNameOrdinals + count*2) + (DWORD)hModule);

		}

	}

	else

	{//如果第二个参数是函数序号

		if ((unsigned long)lpProcName > pExportDir->Base + pExportDir->NumberOfFunctions || (unsigned long)lpProcName <pExportDir->Base)

			return 0;

		dwProcOrdinalNumber = (DWORD)lpProcName - pExportDir->Base; //函数位置

	}

	dwProcAddressVA = *(DWORD*)((pExportDir->AddressOfFunctions + dwProcOrdinalNumber*4) + (DWORD)hModule) + (DWORD)hModule;

_whenend:	

	if (dwProcAddressVA >dwImage_Export_Directory_Size + dwImage_Export_Directory_RVA + (DWORD)hModule || dwProcAddressVA < (DWORD)hModule + dwImage_Export_Directory_RVA || dwProcAddressVA == 0)

		return (FARPROC)dwProcAddressVA;

	count = 0;

	DWORD dwProcAddr = dwProcAddressVA;

	byte bTmp = *(byte*)dwProcAddr;

	do 

	{

		if (0 == bTmp)

			return 0;

		if (0x2E == bTmp)  //ascii '.'

			break;

		bTmp = *(byte*)(dwProcAddr + 1);

		dwProcAddr++;

	} while (1);

	unsigned int libNameLength = dwProcAddr - dwProcAddressVA;

	if (libNameLength <= 0)

		return 0;

	memset(dllName, 0, _MAX_FNAME);

	memcpy(dllName, (const void*)dwProcAddressVA, libNameLength);

	HMODULE hDllModule = GetModuleHandleA(dllName);

	dwProcAddr++;

	return MyGetProcAddress(hDllModule, (LPCSTR)dwProcAddr);  //递归

}
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [`InitialSample.JPG`](javascript:void(0);)

  （189.75kb，1505次下载）

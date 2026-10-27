
```
void Decrypt()

{

	BOOL bResult = TRUE;

	HANDLE hProcess = NULL;

	TERMINATEPROCESS TerminateProcess = NULL;

	try

	{



		HINSTANCE hKernel32 = NULL;

		__asm

		{

			push eax

				mov eax, fs:[0x30]		// PEB

				mov eax, [eax + 0x0c]		// LDR

				mov eax, [eax + 0x0c]		// 本进程模块, LDA_DATA_TABLE_ENTRY 结构

				mov eax, [eax]			// ntdll 模块

				mov eax, [eax]			// kernel32 模块

				mov eax, [eax + 0x18]		// dllBase

				mov hKernel32, eax

				pop eax

		}



		char szGetProcAddress[] = { 'G', 'e', 't', 'P', 'r', 'o', 'c', 'A', 'd', 'd', 'r', 'e', 's', 's', '\0' };

		GETPROCADDRESS GetProcAddress = NULL;

		// 在 kernel32中寻找 GetProcAddress 函数

		IMAGE_DOS_HEADER* pDosHeader = (IMAGE_DOS_HEADER*)hKernel32;

		IMAGE_NT_HEADERS* pNtHeader = (IMAGE_NT_HEADERS*)((DWORD)hKernel32 + pDosHeader->e_lfanew);

		IMAGE_EXPORT_DIRECTORY* pExportDirectory = (IMAGE_EXPORT_DIRECTORY*)((DWORD)hKernel32 + pNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress);

		short* functionOrdinals = (short*)((DWORD)hKernel32 + pExportDirectory->AddressOfNameOrdinals);

		int* adddressOfNames = (int*)((DWORD)hKernel32 + pExportDirectory->AddressOfNames);

		int* addressOfFunctions = (int*)((DWORD)hKernel32 + pExportDirectory->AddressOfFunctions);

		for (int i = 0; i < pExportDirectory->NumberOfNames; ++i)

		{

			// 第 i 个函数名地址

			char* name = (char*)((DWORD)hKernel32 + adddressOfNames[i]);

			char* left = name;

			char* right = szGetProcAddress;



			bool res = true;

			while (*left || *right)

			{

				if (*left != *right)

				{

					res = false;

					break;

				}

				++left;

				++right;

			}

			if (res)

			{

				GetProcAddress = (GETPROCADDRESS)((DWORD)hKernel32 + addressOfFunctions[functionOrdinals[i]]);

				break;

			}

		}



		pDosHeader = NULL;

		pNtHeader = NULL;



		char szLoadLibrary[] = { 'L', 'o', 'a', 'd', 'L', 'i', 'b', 'r', 'a', 'r', 'y', 'A', '\0' };

		char szGetModuleHandle[] = { 'G', 'e', 't', 'M', 'o', 'd', 'u', 'l', 'e', 'H', 'a', 'n', 'd', 'l', 'e', 'A', '\0' };

		char szCreateProcess[] = { 'C', 'r', 'e', 'a', 't', 'e', 'P', 'r', 'o', 'c', 'e', 's', 's', 'A', '\0' };

		char szGetModuleFileName[] = { 'G', 'e', 't', 'M', 'o', 'd', 'u', 'l', 'e', 'F', 'i', 'l', 'e', 'N', 'a', 'm', 'e', 'A', '\0' };

		char szGetThreadContext[] = { 'G', 'e', 't', 'T', 'h', 'r', 'e', 'a', 'd', 'C', 'o', 'n', 't', 'e', 'x', 't', '\0'};

		char szReadProcessMemory[] = { 'R', 'e', 'a', 'd', 'P', 'r', 'o', 'c', 'e', 's', 's', 'M', 'e', 'm', 'o', 'r', 'y', '\0' };

		char szZwUnmapViewOfSection[] = { 'Z', 'w', 'U', 'n', 'm', 'a', 'p', 'V', 'i', 'e', 'w', 'O', 'f', 'S', 'e', 'c', 't', 'i', 'o', 'n', '\0' };

		char szNtDll[] = { 'n', 't', 'd', 'l', 'l', '.', 'd', 'l', 'l', '\0' };

		char szMemcpy[] = { 'm', 'e', 'm', 'c', 'p', 'y', '\0' };

		char szMemset[] = { 'm', 'e', 'm', 's', 'e', 't', '\0' };

		char szMSVCRT[] = { 'm', 's', 'v', 'c', 'r', 't', '.', 'd', 'l', 'l', '\0' };

		char szMalloc[] = { 'm', 'a', 'l', 'l', 'o', 'c', '\0' };

		char szFree[] = { 'f', 'r', 'e', 'e', '\0' };

		char szVirtualAllocEx[] = { 'V', 'i', 'r', 't', 'u', 'a', 'l', 'A', 'l', 'l', 'o', 'c', 'E', 'x', '\0' };

		char szWriteProcessMemory[] = { 'W', 'r', 'i', 't', 'e', 'P', 'r', 'o', 'c', 'e', 's', 's', 'M', 'e', 'm', 'o', 'r', 'y', '\0' };

		char szSetThreadContext[] = { 'S', 'e', 't', 'T', 'h', 'r', 'e', 'a', 'd', 'C', 'o', 'n', 't', 'e', 'x', 't', '\0' };

		char szResumeThread[] = { 'R', 'e', 's', 'u', 'm', 'e', 'T', 'h', 'r', 'e', 'a', 'd', '\0' };

		char szTerminateProcess[] = { 'T', 'e', 'r', 'm', 'i', 'n', 'a', 't', 'e', 'P', 'r', 'o', 'c', 'e', 's', 's', '\0' };

		char szExitProcess[] = { 'E', 'x', 'i', 't', 'P', 'r', 'o', 'c', 'e', 's', 's', '\0' };



		LOADLIBRARY LoadLibraryA = (LOADLIBRARY)GetProcAddress(hKernel32, szLoadLibrary);

		GETMODULEHANDLE GetModuleHandleA = (GETMODULEHANDLE)GetProcAddress(hKernel32, szGetModuleHandle);

		CREATEPROCESS CreateProcessA = (CREATEPROCESS)GetProcAddress(hKernel32, szCreateProcess);

		GETMODULEFILENAME GetModuleFileNameA = (GETMODULEFILENAME)GetProcAddress(hKernel32, szGetModuleFileName);

		GETTHREADCONTEXT GetThreadContext = (GETTHREADCONTEXT)GetProcAddress(hKernel32, szGetThreadContext);

		READPROCESSMEMORY ReadProcessMemory = (READPROCESSMEMORY)GetProcAddress(hKernel32, szReadProcessMemory);

		WRITEPROCESSMEMORY WriteProcessMemory = (WRITEPROCESSMEMORY)GetProcAddress(hKernel32, szWriteProcessMemory);

		VIRTUALALLOCEX VirtualAllocEx = (VIRTUALALLOCEX)GetProcAddress(hKernel32, szVirtualAllocEx);

		SETTHREADCONTEXT SetThreadContext = (SETTHREADCONTEXT)GetProcAddress(hKernel32, szSetThreadContext);

		RESUMETHREAD ResumeThread = (RESUMETHREAD)GetProcAddress(hKernel32, szResumeThread);

		TerminateProcess = (TERMINATEPROCESS)GetProcAddress(hKernel32, szTerminateProcess);

		EXITPROCESS ExitProcess = (EXITPROCESS)GetProcAddress(hKernel32, szExitProcess);



		HMODULE hNtDll = GetModuleHandleA(szNtDll);

		ZWUNMAPVIEWOFSECTION ZwUnmapViewOfSection = (ZWUNMAPVIEWOFSECTION)GetProcAddress(hNtDll, szZwUnmapViewOfSection);

		MEMCPY memcpy = (MEMCPY)GetProcAddress(hNtDll, szMemcpy);

		MEMSET memset = (MEMSET)GetProcAddress(hNtDll, szMemset);



		HMODULE hMSVCRT = LoadLibraryA(szMSVCRT);

		MALLOC malloc = (MALLOC)GetProcAddress(hMSVCRT, szMalloc);

		FREE free = (FREE)GetProcAddress(hMSVCRT, szFree);





		HANDLE hModule = GetModuleHandleA(NULL);

		pDosHeader = (IMAGE_DOS_HEADER*)hModule;

		pNtHeader = (IMAGE_NT_HEADERS*)((DWORD)hModule + pDosHeader->e_lfanew);

		IMAGE_SECTION_HEADER* pSectionHeader = (IMAGE_SECTION_HEADER*)((DWORD)&pNtHeader->OptionalHeader + pNtHeader->FileHeader.SizeOfOptionalHeader);

		pSectionHeader = pSectionHeader + pNtHeader->FileHeader.NumberOfSections - 1;



		// 获取程序数据

		char* src = (char*)((DWORD)hModule + pSectionHeader->VirtualAddress + 0x2000);

		int srcSize = pSectionHeader->SizeOfRawData - 0x2000;



		// 拉伸

		pDosHeader = (IMAGE_DOS_HEADER*)src;

		pNtHeader = (IMAGE_NT_HEADERS*)((DWORD)src + pDosHeader->e_lfanew);

		IMAGE_OPTIONAL_HEADER32* pOptionalHeader = &pNtHeader->OptionalHeader;

		pSectionHeader = (IMAGE_SECTION_HEADER*)((DWORD)pOptionalHeader + pNtHeader->FileHeader.SizeOfOptionalHeader);

		int length = pSectionHeader[pNtHeader->FileHeader.NumberOfSections - 1].VirtualAddress + pSectionHeader[pNtHeader->FileHeader.NumberOfSections - 1].SizeOfRawData;



		//有时候会出现length比Image还要大

		length = length > pOptionalHeader->SizeOfImage ? length : pOptionalHeader->SizeOfImage;



		char* imageBuffer = (char*)malloc(length);

		memset(imageBuffer, 0, length);

		memcpy(imageBuffer, src, pOptionalHeader->SizeOfHeaders);

		for (int i = 0; i < pNtHeader->FileHeader.NumberOfSections; ++i)

		{

			memcpy(imageBuffer + pSectionHeader->VirtualAddress, src + pSectionHeader->PointerToRawData, pSectionHeader->SizeOfRawData);

			++pSectionHeader;

		}





		char szModuleName[MAX_PATH];

		GetModuleFileNameA(NULL, szModuleName, MAX_PATH);



		STARTUPINFO si;

		memset(&si, 0, sizeof(si));

		PROCESS_INFORMATION pi;

		memset(&pi, 0, sizeof(pi));

		si.cb = sizeof(si);

		CreateProcessA(NULL, szModuleName, NULL, NULL, FALSE, CREATE_SUSPENDED, NULL, NULL, &si, &pi);

		hProcess = pi.hProcess;

		CONTEXT context;

		context.ContextFlags = CONTEXT_FULL;

		GetThreadContext(pi.hThread, &context);

		DWORD dwEntryPoint = context.Eax;

		DWORD baseAddress = context.Ebx + 8;

		DWORD shellImageBase = 0;

		ReadProcessMemory(pi.hProcess, (LPVOID)baseAddress, &shellImageBase, 4, NULL);

		ZwUnmapViewOfSection(pi.hProcess, (PVOID)shellImageBase);

		LPVOID buffer = VirtualAllocEx(pi.hProcess, (LPVOID)pOptionalHeader->ImageBase, pOptionalHeader->SizeOfImage, MEM_RESERVE | MEM_COMMIT, PAGE_EXECUTE_READWRITE);

		if (!buffer)

		{

			VirtualAllocEx(pi.hProcess, NULL, pOptionalHeader->SizeOfImage, MEM_RESERVE | MEM_COMMIT, PAGE_EXECUTE_READWRITE);

			pDosHeader = (IMAGE_DOS_HEADER*)imageBuffer;

			pNtHeader = (IMAGE_NT_HEADERS*)((DWORD)imageBuffer + pDosHeader->e_lfanew);

			IMAGE_OPTIONAL_HEADER32* pOptionalHeader = &pNtHeader->OptionalHeader;

			IMAGE_BASE_RELOCATION* pBaseRelocation = (IMAGE_BASE_RELOCATION*)((DWORD)imageBuffer + pOptionalHeader->DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress);



			while (pBaseRelocation->SizeOfBlock || pBaseRelocation->VirtualAddress)

			{

				short* pTable = (short*)((int)pBaseRelocation + 8);

				int len = (pBaseRelocation->SizeOfBlock - 8) / 2;

				for (int i = 0; i < len; ++i)

				{

					DWORD RVA = pBaseRelocation->VirtualAddress + (pTable[i] & 0x0FFF);

					if (((pTable[i] & 0xF000) >> 12) == 3)

					{

						*(DWORD*)(RVA + (DWORD)imageBuffer) += ((DWORD)buffer - pOptionalHeader->ImageBase);

					}

				}

				pBaseRelocation = (IMAGE_BASE_RELOCATION*)((DWORD)pBaseRelocation + pBaseRelocation->SizeOfBlock);

			}

			pOptionalHeader->ImageBase = (DWORD)buffer;

		}

		WriteProcessMemory(pi.hProcess, buffer, imageBuffer, pOptionalHeader->SizeOfImage, NULL);

		WriteProcessMemory(pi.hProcess, (LPVOID)(context.Ebx + 8), &buffer, 4, NULL);

		context.Eax = pOptionalHeader->ImageBase + pOptionalHeader->AddressOfEntryPoint;

		SetThreadContext(pi.hThread, &context);

		ResumeThread(pi.hThread);

		free(imageBuffer);

		ExitProcess(0);

	}

	catch (...)

	{

		if (hProcess) TerminateProcess(hProcess, 0);

		return;

	}

}
```

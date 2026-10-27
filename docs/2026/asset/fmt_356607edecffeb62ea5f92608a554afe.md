
```
#include <cstdio>

#include <Windows.h>



#define TOUPPER(x) (x >= 'a' && x <= 'z' ? x - 32 : x)



typedef struct _UNICODE_STRING

{

	USHORT Length;

	USHORT MaximumLength;

	PWSTR Buffer;

}UNICODE_STRING, *PUNICODE_STRING;



typedef struct _PEB_LDR_DATA

{

	DWORD Length;

	BOOL Initialized;

	PVOID SsHandle;

	LIST_ENTRY InLoadOrderModuleList;

	LIST_ENTRY InMemoryOrderModuleList;

	LIST_ENTRY InInitializationOrderModuleList;

}PEB_LDR_DATA, *PPEB_LDR_DATA;



typedef struct _LDR_DATA_TABLE_ENTRY

{

	LIST_ENTRY InLoadOrderModuleList;

	LIST_ENTRY InMemoryOrderModuleList;

	LIST_ENTRY InInitializationOrderModuleList;

	PVOID DllBase;

	PVOID EntryPoint;

	UINT32 SizeOfImage;

	UNICODE_STRING FullDllName;

	UNICODE_STRING BaseDllName;

	UINT32 Flags;

	USHORT LoadCount;

	USHORT TlsIndex;

	LIST_ENTRY HashLinks;

	PVOID SectionPointer;

	UINT32 CheckSum;

	UINT32 TimeDateStamp;

	PVOID LoadedImports;

	PVOID EntryPointActivationContext;

	PVOID PatchInformation;

}LDR_DATA_TABLE_ENTRY, *PLDR_DATA_TABLE_ENTRY;



typedef FARPROC(WINAPI *pFnGetProcAddress)(HMODULE, LPCSTR);

typedef HMODULE(WINAPI *pFnLoadLibrary)(LPCTSTR);

typedef int(WINAPI *pFnMessageBox)(HWND, LPCTSTR, LPCTSTR, UINT);



VOID WINAPI ShellCode();



int main()

{

	ShellCode();



	return 0;

}



VOID WINAPI ShellCode()

{

	HINSTANCE hKernel = NULL, hUser32 = NULL;

	char szKernel32[] = { 'K', 0, 'E', 0, 'R', 0, 'N', 0, 'E', 0, 'L', 0, '3', 0, '2', 0, '.', 0, 'D', 0, 'L', 0, 'L', 0, 0, 0 };

	char szGetProc[] = { 'G', 'e', 't', 'P', 'r', 'o', 'c', 'A', 'd', 'd', 'r', 'e', 's', 's', 0 };

	char szLoadLibrary[] = { 'L', 'o', 'a', 'd', 'L', 'i', 'b', 'r', 'a', 'r', 'y', 'A', 0 };

	char szUser32[] = { 'u', 's', 'e', 'r', '3', '2', '.', 'd', 'l', 'l', 0 };

	char szMessageBox[] = { 'M', 'e', 's', 's', 'a', 'g', 'e', 'B', 'o', 'x', 'A', 0 };

	char szOutput[] = { '1', '9', '0', '0', 0 };

	PPEB_LDR_DATA ldr;

	PLDR_DATA_TABLE_ENTRY ldte;

	PLIST_ENTRY Head, Cur;

	pFnGetProcAddress myGetProcAddress = NULL;

	pFnLoadLibrary myLoadLibrary;

	pFnMessageBox myMessageBox;

	PWORD pKernel, pDllName;

	BOOL isFind;

	DWORD i, j;



	PIMAGE_DOS_HEADER pDosHead;

	PIMAGE_OPTIONAL_HEADER pOptionHead;

	PIMAGE_EXPORT_DIRECTORY pExport;

	DWORD dwAddrNumOfName;

	PDWORD pAddrOfName;

	PDWORD pAddrFunc;

	PWORD pAddrOfOrd;

	PCHAR pFuncName;



	__asm

	{

		push eax

		push ecx



		mov eax, fs:[0x30]

		mov ecx, [eax + 0x0C]

		mov ldr, ecx



		pop ecx

		pop eax

	}



	Head = &(ldr->InMemoryOrderModuleList);

	Cur = Head->Flink;



	pKernel = (PWORD)szKernel32;	//遍历模块找到kernel32.dll模块的基地址

	do {

		ldte = CONTAINING_RECORD(Cur, LDR_DATA_TABLE_ENTRY, InMemoryOrderModuleList);



		pDllName = (PWORD)ldte->BaseDllName.Buffer;

		isFind = TRUE;

		for (i = 0; pKernel[i]; i++)

		{	

			if (pKernel[i] != TOUPPER(pDllName[i]))		//比较模块名字与目标模块名字是否相同

			{

				isFind = FALSE;

				break;

			}

		}



		if (pDllName[i]) isFind = FALSE;



		if (isFind)

		{

			hKernel = (HINSTANCE)ldte->DllBase;	//相同则找到需要的模块

			break;

		}



		Cur = Cur->Flink;

	} while (Cur != Head);



	if (hKernel)	//从导出表中获取GetProcAddress的地址

	{

		pDosHead = (PIMAGE_DOS_HEADER)hKernel;

		pOptionHead = (PIMAGE_OPTIONAL_HEADER)((DWORD)pDosHead + pDosHead->e_lfanew + 4 + IMAGE_SIZEOF_FILE_HEADER);

		pExport = (PIMAGE_EXPORT_DIRECTORY)(pOptionHead->DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress + (DWORD)pDosHead);

		dwAddrNumOfName = pExport->NumberOfFunctions;

		pAddrOfName = (PDWORD)((DWORD)pDosHead + pExport->AddressOfNames);

		pAddrFunc = (PDWORD)(pExport->AddressOfFunctions + (DWORD)hKernel);

		pAddrOfOrd = (PWORD)(pExport->AddressOfNameOrdinals + (DWORD)hKernel);



		for (i = 0; i < dwAddrNumOfName; i++)

		{

			pFuncName = (PCHAR)(pAddrOfName[i] + (DWORD)hKernel);

			isFind = TRUE;



			for ( j = 0; pFuncName[j]; j++)

			{

				if (pFuncName[j] != szGetProc[j])

				{

					isFind = FALSE;

					break;

				}

			}



			if (szGetProc[j]) isFind = FALSE;



			if (isFind)

			{

				myGetProcAddress = (pFnGetProcAddress)(pAddrFunc[pAddrOfOrd[i]] + (DWORD)hKernel);

				break;

			}

		}

	}



	if (myGetProcAddress)	//得到GetProcAddress以后可以获得需要的系统函数

	{

		myLoadLibrary = (pFnLoadLibrary)myGetProcAddress(hKernel, szLoadLibrary);

		hUser32 = myLoadLibrary(szUser32);

		if (hUser32)

		{

			myMessageBox = (pFnMessageBox)myGetProcAddress(hUser32, szMessageBox);

			myMessageBox(0, szOutput, szOutput, 0);

		}

	}

}
```

提取出相应的`ShellCode`如下


```cpp
#include "stdafx.h"

#include <winnt.h>

#pragma warning(disable:4102)

typedef HMODULE (WINAPI *LOADLIBRARYA)(LPCSTR lpLibFileName);
typedef FARPROC (WINAPI *GETPROCADDRESS)(HMODULE hModule, LPCSTR pszProcName);
typedef DWORD (WINAPI *GETLASTERROR)();
typedef LPVOID (WINAPI *VIRTUALALLOC)(LPVOID lpAddress, DWORD dwSize, DWORD flAllocationType, DWORD flProtect);
typedef BOOL (WINAPI *VIRTUALFREE)(LPVOID lpAddress, DWORD dwSize, DWORD dwFreeType);
typedef BOOL (WINAPI *FREELIBRARY)(HMODULE hModule);
typedef void (WINAPI *OUTPUTDEBUGSTRINGA)(LPCSTR lpOutputString);
typedef VOID (WINAPI *SLEEP)(DWORD dwMilliseconds);

typedef void (WINAPI *MYCOPYMEMORY)(LPVOID lpDst, LPCVOID lpSrc, DWORD dwLength);

typedef BOOL (WINAPI *PDLLMAIN)(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved);

#define THUNK_BEGIN_TAG \
	__asm _emit 't' \
	__asm _emit 'h' \
	__asm _emit 'u' \
	__asm _emit 'n' \
	__asm _emit 'k' \
	__asm _emit '_' \
	__asm _emit 'b' \
	__asm _emit 'e' \
	__asm _emit 'g' \
	__asm _emit 'i' \
	__asm _emit 'n' 

#define THUNK_BEGIN_STR "thunk_begin"

#define THUNK_END_TAG \
	__asm _emit 't' \
	__asm _emit 'h' \
	__asm _emit 'u' \
	__asm _emit 'n' \
	__asm _emit 'k' \
	__asm _emit '_' \
	__asm _emit 'e' \
	__asm _emit 'n' \
	__asm _emit 'd' 

#define THUNK_END_STR "thunk_end"

#define CODE_BEGIN_TAG \
	__asm _emit 'c' \
	__asm _emit 'o' \
	__asm _emit 'd' \
	__asm _emit 'e' \
	__asm _emit '_' \
	__asm _emit 'b' \
	__asm _emit 'e' \
	__asm _emit 'g' \
	__asm _emit 'i' \
	__asm _emit 'n' 

#define CODE_BEGIN_STR "code_begin"

#pragma check_stack(off)
#pragma pack(push, 8)

__declspec(naked) void TestCode()
{
	// 临时变量
	DWORD offsetLabel;

	HMODULE hKernel32;
	
	LOADLIBRARYA pLoadLibraryA;
	GETPROCADDRESS pGetProcAddress;
	FREELIBRARY pFreeLibrary;
	//VIRTUALALLOC pVirtualAlloc;
	MYCOPYMEMORY pCopyMemory;
	OUTPUTDEBUGSTRINGA pOutputDbgStringA;
	SLEEP pSleep;

	// 起始标志
	THUNK_BEGIN_TAG

	__asm
	{
//// module 
//_RO_dwKernel32Base:
//		INT 3
//		INT 3
//		INT 3
//		INT 3

// api地址空间 使用的时候外部程序填写
_RO_LoadLibraryA: 
		INT 3
		INT 3
		INT 3
		INT 3

_RO_GetProcAddress:
		INT 3
		INT 3
		INT 3
		INT 3

_RO_FreeLibrary:
		INT 3
		INT 3
		INT 3
		INT 3

// string 
_RO_szKernel32:				// db "kernel32.dll",0 13
		_emit 'K'
		_emit 'e'
		_emit 'r'
		_emit 'n'
		_emit 'e'
		_emit 'l'
		_emit '3'
		_emit '2'
		_emit '.'
		_emit 'd'
		_emit 'l'
		_emit 'l'
		_emit '\0'

_RO_szOutputDebugStringA:	// db "OutputDebugStringA",0  19
		_emit 'O'
		_emit 'u'
		_emit 't'
		_emit 'p'
		_emit 'u'
		_emit 't'
		_emit 'D'
		_emit 'e'
		_emit 'b'
		_emit 'u'
		_emit 'g'
		_emit 'S'
		_emit 't'
		_emit 'r'
		_emit 'i'
		_emit 'n'
		_emit 'g'
		_emit 'A'
		_emit '\0'

_RO_szSleep:				// db "Sleep",0 6
		_emit 'S'
		_emit 'l'
		_emit 'e'
		_emit 'e'
		_emit 'p'
		_emit '\0'

		CODE_BEGIN_TAG

// 		INT 3	// 调试
// 		INT 3
// 		INT 3
// 		INT 3
// 		INT 3
// 		INT 3
// 		INT 3
// 		INT 3
// 主函数
_MAINCODE_BEGIN:
		PUSHAD
		PUSHFD
		MOV EBP, ESP						// 给VC构造BP FRAME 
		SUB ESP, 1024						// 临时变量

		// 取得偏移量
		CALL _GET_EIP
_GET_EIP:         
		POP EAX								// 获取当前的EIP， 也就是GET_EIP 处的va !!!
		SUB EAX, OFFSET _GET_EIP			// 计算标号实际地址跟编译生成地址的偏移，用offsetLabel 保存
		MOV offsetLabel, EAX      
		MOV EBX, EAX						// offserLabel

		// 初始化临时变量
		MOV EAX, OFFSET _RO_LoadLibraryA	// LoadLibraryA
		ADD EAX, EBX
		MOV EAX, DWORD PTR [EAX]
		MOV pLoadLibraryA, eax

		MOV EAX, OFFSET _RO_GetProcAddress	// GetProcess
		ADD EAX, EBX
		MOV EAX, DWORD PTR [EAX]
		MOV pGetProcAddress, eax
			
		MOV EAX, OFFSET _RO_FreeLibrary		// FreeLibrary
		ADD EAX, EBX
		MOV EAX, DWORD PTR [EAX]
		MOV pFreeLibrary, eax

		MOV EAX, OFFSET _PROC_MemCpy		// CopyMemory
		ADD EAX, EBX
		MOV pCopyMemory, EAX;

		MOV EAX, OFFSET _RO_szKernel32		// hKernel32
		ADD EAX, EBX
		PUSH EAX
		CALL DWORD PTR[pLoadLibraryA]
		MOV hKernel32, EAX

		// 做测试
		MOV EAX, _RO_szOutputDebugStringA
		ADD EAX, EBX
		MOV ESI, hKernel32
		CALL _PROC_GetProcAddress
		MOV pOutputDbgStringA, EAX

		MOV EAX, _RO_szSleep
		ADD EAX, EBX
		MOV ESI, hKernel32
		CALL _PROC_GetProcAddress
		MOV pSleep, EAX

_TEST:
		MOV EAX, _RO_szOutputDebugStringA
		ADD EAX, EBX
		PUSH EAX
		CALL DWORD PTR[pOutputDbgStringA]

		PUSH 0x3E8
		CALL DWORD PTR[pSleep]
		JMP _TEST

_MAINCODE_END:
		XOR EAX, EAX						// 函数返回值
		MOV ESP, EBP
		POPFD
		POPAD
		RETN 4								// 线程函数有一个参数
	}

// 子函数
	__asm
	{
// 取函数地址 eax=函数名 esi=模块地址
_PROC_GetProcAddress:
		PUSH EDX
		PUSH EAX
		PUSH ESI
		MOV EDX, OFFSET _RO_GetProcAddress
		ADD EDX, offsetLabel
		CALL [EDX]
		POP EDX
		RETN
	}

	__asm
	{
// 简单的memcpy 函数
_PROC_MemCpy:   
		PUSH EBP
		MOV EBP,ESP
		PUSH ECX
		PUSH EAX
		PUSH ESI
		PUSH EDI
		MOV EDI,DWORD PTR SS:[EBP+08h]		// dst
		MOV ESI,DWORD PTR SS:[EBP+0Ch]		// src
		MOV ECX,DWORD PTR SS:[EBP+10h]		// len
		TEST ECX, ECX
		JZ __CopyEnd						// copy 0字节的.textbss 要出错所以修改 by ranbo
		XOR EAX,EAX

__cpy:
		LODS BYTE PTR DS:[ESI]
		STOS BYTE PTR ES:[EDI]
		LOOP __cpy

__CopyEnd:
		POP EDI
		POP ESI
		POP EAX
		POP ECX
		MOV ESP,EBP
		POP EBP
		RET 0CH
	}

	THUNK_END_TAG
}

#pragma check_stack(off)
// 注入一个进程 并load一个dll的代码
__declspec(naked) void LoadLibraryCode()
{
	DWORD offsetLabel;
	HMODULE hKernel32;
	
	// 函数指针
	LOADLIBRARYA pLoadLibraryA;
	GETPROCADDRESS pGetProcAddress;
	FREELIBRARY pFreeLibrary;
	VIRTUALALLOC pVirtualAlloc;
	MYCOPYMEMORY pCopyMemory;
	OUTPUTDEBUGSTRINGA pOutputDbgStringA;
	SLEEP pSleep;

	// loadlibrary时候用的临时变量
	PBYTE pbyLibrary;				// dll内存数据

	// 起始标志
	THUNK_BEGIN_TAG

	__asm
	{
//// 需要load的 dll 内存 指针 外部初始化时候申请内存并填写这个指针 
_RO_pbyLibrary:
		INT 3
		INT 3
		INT 3
		INT 3

// api地址空间 使用的时候外部程序填写
_RO_LoadLibraryA: 
		INT 3
		INT 3
		INT 3
		INT 3

_RO_GetProcAddress:
		INT 3
		INT 3
		INT 3
		INT 3

_RO_FreeLibrary:
		INT 3
		INT 3
		INT 3
		INT 3

// string 
_RO_szKernel32:				// db "kernel32.dll",0 13
		_emit 'K'
		_emit 'e'
		_emit 'r'
		_emit 'n'
		_emit 'e'
		_emit 'l'
		_emit '3'
		_emit '2'
		_emit '.'
		_emit 'd'
		_emit 'l'
		_emit 'l'
		_emit '\0'

_RO_szVirtualAlloc:
		_emit 'V'
		_emit 'i'
		_emit 'r'
		_emit 't'
		_emit 'u'
		_emit 'a'
		_emit 'l'
		_emit 'A'
		_emit 'l'
		_emit 'l'
		_emit 'o'
		_emit 'c'
		_emit '\0'

_RO_szOutputDebugStringA:	// db "OutputDebugStringA",0  19
		_emit 'O'
		_emit 'u'
		_emit 't'
		_emit 'p'
		_emit 'u'
		_emit 't'
		_emit 'D'
		_emit 'e'
		_emit 'b'
		_emit 'u'
		_emit 'g'
		_emit 'S'
		_emit 't'
		_emit 'r'
		_emit 'i'
		_emit 'n'
		_emit 'g'
		_emit 'A'
		_emit '\0'

_RO_szSleep:				// db "Sleep",0 6
		_emit 'S'
		_emit 'l'
		_emit 'e'
		_emit 'e'
		_emit 'p'
		_emit '\0'

		CODE_BEGIN_TAG

// 		INT 3	// 调试
// 		INT 3
// 		INT 3
// 		INT 3
// 		INT 3
// 		INT 3
// 		INT 3
// 		INT 3
// 主函数
_MAINCODE_BEGIN:
		PUSHAD
		PUSHFD
		MOV EBP, ESP						// 给VC构造BP FRAME 
		SUB ESP, 2048						// 临时变量

		// 取得偏移量
		CALL _GET_EIP
_GET_EIP:         
		POP EAX								// 获取当前的EIP， 也就是GET_EIP 处的va !!!
		SUB EAX, OFFSET _GET_EIP			// 计算标号实际地址跟编译生成地址的偏移，用offsetLabel 保存
		MOV offsetLabel, EAX      
		MOV EBX, EAX						// offserLabel

		// 初始化临时变量
		// lib操作的几个api必须最先初始化 然后handler 之后才能开始调用函数
		MOV EAX, OFFSET _RO_pbyLibrary
		ADD EAX, EBX
		MOV EAX, DWORD PTR[EAX]
		MOV pbyLibrary, EAX

		MOV EAX, OFFSET _RO_LoadLibraryA	// LoadLibraryA
		ADD EAX, EBX
		MOV EAX, DWORD PTR [EAX]
		MOV pLoadLibraryA, eax

		MOV EAX, OFFSET _RO_GetProcAddress	// GetProcess
		ADD EAX, EBX
		MOV EAX, DWORD PTR [EAX]
		MOV pGetProcAddress, eax
			
		MOV EAX, OFFSET _RO_FreeLibrary		// FreeLibrary
		ADD EAX, EBX
		MOV EAX, DWORD PTR [EAX]
		MOV pFreeLibrary, eax

		// handler 
		MOV EAX, OFFSET _RO_szKernel32		// hKernel32
		ADD EAX, EBX
		PUSH EAX
		CALL DWORD PTR[pLoadLibraryA]
		MOV hKernel32, EAX

		// 做测试用的几个API 用GetProcess得到
		MOV EAX, OFFSET	_RO_szVirtualAlloc	// VirtualAlloc
		ADD EAX, EBX
		MOV ESI, hKernel32
		CALL _PROC_GetProcAddress
		MOV pVirtualAlloc, eax

		MOV EAX, _RO_szOutputDebugStringA
		ADD EAX, EBX
		MOV ESI, hKernel32
		CALL _PROC_GetProcAddress
		MOV pOutputDbgStringA, EAX

		MOV EAX, _RO_szSleep
		ADD EAX, EBX
		MOV ESI, hKernel32
		CALL _PROC_GetProcAddress
		MOV pSleep, EAX

		// 自定义函数初始化
		MOV EAX, OFFSET _PROC_MemCpy		// CopyMemory
		ADD EAX, EBX
		MOV pCopyMemory, EAX;

// _TEST:
// 		MOV EAX, _RO_szOutputDebugStringA
// 		ADD EAX, EBX
// 		PUSH EAX
// 		CALL DWORD PTR[pOutputDbgStringA]
// 
// 		PUSH 0x3E8
// 		CALL DWORD PTR[pSleep]
// 		JMP _TEST

	}

	//-------------------------------------------------------------
	// C++代码 加载dll
	// 
	// 1. 申请内存 copy header section 到内存中
	PIMAGE_DOS_HEADER		pOrgDosHead;
	PIMAGE_NT_HEADERS		pOrgNtHead;
	PIMAGE_SECTION_HEADER	pOrgSecHead;

	PBYTE					pMemoryImage;

	pOrgDosHead = (PIMAGE_DOS_HEADER)pbyLibrary;
	pOrgNtHead = (PIMAGE_NT_HEADERS)(pbyLibrary + pOrgDosHead->e_lfanew);
	pOrgSecHead = IMAGE_FIRST_SECTION(pOrgNtHead);

	pMemoryImage = (PBYTE)pVirtualAlloc(NULL,
		pOrgNtHead->OptionalHeader.SizeOfImage,
		MEM_RESERVE | MEM_COMMIT,
		PAGE_EXECUTE_READWRITE);

	if (!pMemoryImage)
	{
		//__asm int 3;
		goto _MAINCODE_END;
	}

	// copy header
	pCopyMemory(pMemoryImage, pbyLibrary, pOrgNtHead->OptionalHeader.SizeOfHeaders);

	// copy section
	for(int i=0; i< pOrgNtHead->FileHeader.NumberOfSections; i++, pOrgSecHead++)
	{
		pCopyMemory(
			pMemoryImage + pOrgSecHead->VirtualAddress,
			pbyLibrary + pOrgSecHead->PointerToRawData,
			pOrgSecHead->SizeOfRawData
			);
	}

	// 2. 填充导入表
	// 指向copy之后的东东
	PIMAGE_DOS_HEADER		pDosHead;
	PIMAGE_NT_HEADERS		pNtHead;
	PIMAGE_SECTION_HEADER	pSecHead;
	PIMAGE_OPTIONAL_HEADER	pOptHead;
	PIMAGE_BASE_RELOCATION	pBaseReloc;

	pDosHead = (PIMAGE_DOS_HEADER)pMemoryImage;
	pNtHead = (PIMAGE_NT_HEADERS)(pMemoryImage + pDosHead->e_lfanew);
	pSecHead = IMAGE_FIRST_SECTION(pNtHead);
	pOptHead = &pNtHead->OptionalHeader;

	if(pOptHead->DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].Size != 0)
	{
		PIMAGE_IMPORT_DESCRIPTOR pid;
		pid=(IMAGE_IMPORT_DESCRIPTOR *)
			(pMemoryImage + pOptHead->DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress);

		// For all imported DLLs
		while(pid->FirstThunk && pid->Name)
		{
			DWORD	*pdwThunkRef;
			DWORD	*pdwFuncRef;
			char	*pszDllName;
			HMODULE	hDll;

			pszDllName=(char *) (pMemoryImage + pid->Name);
			hDll = pLoadLibraryA(pszDllName);
			
			if(hDll==NULL) 
			{
				pOutputDbgStringA(pszDllName);
				//__asm int 3;
				goto _MAINCODE_END;
			}

			if(pid->OriginalFirstThunk)
			{
				pdwThunkRef = (DWORD *)(pMemoryImage + pid->OriginalFirstThunk);
				pdwFuncRef = (DWORD *)(pMemoryImage + pid->FirstThunk);
			}
			else
			{
				pdwThunkRef = (DWORD *)(pMemoryImage + pid->FirstThunk);
				pdwFuncRef = (DWORD *)(pMemoryImage + pid->FirstThunk);
			}

			for( ; *pdwThunkRef; pdwThunkRef++, pdwFuncRef++)
			{
				DWORD	dwAddr;

				if(IMAGE_SNAP_BY_ORDINAL(*pdwThunkRef))
				{
					// win 98 需要单独处理
					dwAddr = (DWORD)pGetProcAddress(hDll, (LPCSTR)IMAGE_ORDINAL(*pdwThunkRef));
				}
				else
				{
					PIMAGE_IMPORT_BY_NAME	pName;
					pName = (PIMAGE_IMPORT_BY_NAME)(pMemoryImage + (*pdwThunkRef));
					dwAddr = (DWORD)pGetProcAddress(hDll, (LPCSTR)&pName->Name);
				}

				*pdwFuncRef = dwAddr;
			}

			pid++;
		}
	}

	//__asm int 3;
	// 3. 重定位
	DWORD delta;
	delta=(DWORD)pMemoryImage - (DWORD)pOptHead->ImageBase;

 	if((delta!=0) && (pOptHead->DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].Size!=0)) 
 	{
 		pBaseReloc = (PIMAGE_BASE_RELOCATION)(pMemoryImage 
			+ pOptHead->DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress);
 
 		while(pBaseReloc->VirtualAddress != 0)
		{
			// 结构
			// WORD	offset:12;
			// WORD	type:4;

			int		i;
			PWORD	pwItem = (PWORD)((DWORD)pBaseReloc + IMAGE_SIZEOF_BASE_RELOCATION);
			int		nItems = 
				(pBaseReloc->SizeOfBlock - IMAGE_SIZEOF_BASE_RELOCATION) / sizeof(WORD);

 			for(i=0; i<nItems; i++, pwItem++) 
			{
				DWORD	dwType, dwOffset;
				DWORD   *pBlock;

				dwType = (*pwItem) >> 12;
				dwOffset = (*pwItem) & 0xFFF;
				pBlock = (DWORD *)(pMemoryImage + pBaseReloc->VirtualAddress + dwOffset);

				// 不能使用switch 
				// switch表在函数体之外 而且是绝对地址 引用跳转会出错
				if(dwType == IMAGE_REL_BASED_ABSOLUTE)
				{
				}
				else if(dwType == IMAGE_REL_BASED_HIGH)
				{
 					*((WORD *)pBlock) += HIWORD(delta);
				}
				else if(dwType == IMAGE_REL_BASED_LOW)
				{
					*((WORD *)pBlock) += LOWORD(delta);
				}
 				else if(dwType == IMAGE_REL_BASED_HIGHLOW)
				{
 					*((DWORD *)pBlock) += delta;
				}
				else if(dwType == IMAGE_REL_BASED_HIGHADJ)
				{
					DWORD adjust;
 					adjust=((*((WORD *)pBlock)) << 16) | (*(pwItem+1));
 					adjust += delta;
 					adjust += 0x00008000;
 					*((WORD *)pBlock) = HIWORD(adjust);
 					pwItem++;	// 这种被占用了 加一次
 				}
 				else
				{
					//__asm int 3;
 					goto _MAINCODE_END;
 				}
  			}

			pBaseReloc = (PIMAGE_BASE_RELOCATION)
				((PBYTE)pBaseReloc + pBaseReloc->SizeOfBlock);
 		}
 	}

	//__asm int 3;

	// 4. 运行dllmain
	PDLLMAIN pDllMain;
	pDllMain=(PDLLMAIN)(pMemoryImage + pOptHead->AddressOfEntryPoint);
	pDllMain((HMODULE) pMemoryImage, DLL_PROCESS_ATTACH, NULL);

	__asm{
_MAINCODE_END:
		//__asm int 3;
		MOV ESP, EBP
		POPFD
		POPAD
		XOR EAX, EAX						// 函数返回值
		RETN 4								// 线程函数有一个参数
	}

// 子函数
	__asm
	{
// 取函数地址 eax=函数名 esi=模块地址
_PROC_GetProcAddress:
		PUSH EDX
		PUSH EAX
		PUSH ESI
		MOV EDX, OFFSET _RO_GetProcAddress
		ADD EDX, offsetLabel
		CALL [EDX]
		POP EDX
		RETN
	}

	__asm
	{
// 简单的memcpy 函数
_PROC_MemCpy:   
		PUSH EBP
		MOV EBP,ESP
		PUSH ECX
		PUSH EAX
		PUSH ESI
		PUSH EDI
		MOV EDI,DWORD PTR SS:[EBP+08h]		// dst
		MOV ESI,DWORD PTR SS:[EBP+0Ch]		// src
		MOV ECX,DWORD PTR SS:[EBP+10h]		// len
		TEST ECX, ECX
		JZ __CopyEnd						// copy 0字节的.textbss 要出错所以修改 by ranbo
		XOR EAX,EAX

__cpy:
		LODS BYTE PTR DS:[ESI]
		STOS BYTE PTR ES:[EDI]
		LOOP __cpy

__CopyEnd:
		POP EDI
		POP ESI
		POP EAX
		POP ECX
		MOV ESP,EBP
		POP EBP
		RET 0CH
	}

	THUNK_END_TAG
}

#pragma pack(pop)
#pragma check_stack 

// 内部申请的内存 外边要delete掉
PBYTE GetTestCode(UINT& cbCodeSize, UINT& cbFuncOffset)
{
	LPCSTR pcszThunkBegin = THUNK_BEGIN_STR;
	LPCSTR pcszThunkEnd = THUNK_END_STR;
	LPCSTR pcszCodeBegin = CODE_BEGIN_STR;

	UINT cbMemSize = 0;			// 整段需要copy的代码长度
	UINT cbCodeOffset = 0;		// mem开始距离 maincode起始的偏移 需要用这个作为线程起始地址
	PBYTE pbyCode = NULL;		// 原生代码开始地址
	PBYTE pbyMem = NULL;

	pbyCode = (PBYTE)&TestCode;
	if (0 != memcmp(pbyCode, pcszThunkBegin, strlen(pcszThunkBegin)))
	{
		OutputDebugStringA("pbyCode error thunk begin tag not find!\n");
		return NULL;
	}

	pbyCode += strlen(pcszThunkBegin);

	// 计算需要copy的size
	pbyMem = pbyCode;
	while(memcmp(pbyMem, pcszThunkEnd, strlen(pcszThunkEnd)))
	{
		pbyMem ++;
	}
	cbMemSize = (UINT)(pbyMem - pbyCode);

	// 计算函数起始地址离mem的偏移量
	pbyMem = pbyCode;
	while(memcmp(pbyMem, pcszCodeBegin, strlen(pcszCodeBegin)))
	{
		pbyMem ++;
	}
	cbCodeOffset = (UINT)(pbyMem - pbyCode + strlen(pcszCodeBegin));

	// 生成本地代码 并初始化
	pbyMem = new BYTE[cbMemSize];
	if (!pbyMem)
	{
		return NULL;
	}

	memcpy(pbyMem, pbyCode, cbMemSize);

	// 初始化三个API _RO_LoadLibraryA, _RO_GetProcAddress, _RO_FreeLibrary
	*(LOADLIBRARYA *)pbyMem = &::LoadLibraryA;
	*(GETPROCADDRESS *)(pbyMem + 4) = &::GetProcAddress;
	*(FREELIBRARY *)(pbyMem + 8) = &::FreeLibrary;

	// 填写返回值
	cbCodeSize = cbMemSize;
	cbFuncOffset = cbCodeOffset;
	return pbyMem;
}

void FreeLocalCode(PBYTE pbyCodeMem)
{
	if (pbyCodeMem)
	{
		delete[] pbyCodeMem;
	}
}

BOOL InjectTestCode(HANDLE hProcess)
{
	BOOL fSucceeded = FALSE;

	PBYTE pbyInjCode = NULL;
	UINT cbCodeSize = 0;
	UINT cbCodeOffset = 0;

	PBYTE pbyCodeRemote = NULL;
	DWORD dwOldProtect = 0;
	DWORD dwNumBytesXferred = 0;
	HANDLE hThread = NULL;
	DWORD dwThreadId = 0;

	// 初始化inject code
	pbyInjCode = GetTestCode(cbCodeSize, cbCodeOffset);
	if(!pbyInjCode)
	{
		goto finish;
	}

	// 在目标进程申请地址，写入代码并启动远程线程
	pbyCodeRemote = (PBYTE)VirtualAllocEx(hProcess, NULL, cbCodeSize,
										   MEM_COMMIT | MEM_TOP_DOWN,
                                           PAGE_EXECUTE_READWRITE);
	if(!pbyCodeRemote)
	{
		goto finish;
	}

	if (!VirtualProtectEx(hProcess, pbyCodeRemote, cbCodeSize,
						PAGE_EXECUTE_READWRITE, &dwOldProtect)) 
	{
		goto finish;
	}

	if (!WriteProcessMemory(hProcess, pbyCodeRemote,
						pbyInjCode, cbCodeSize, &dwNumBytesXferred)) 
	{
		goto finish;
	}

	 if ((hThread = CreateRemoteThread(hProcess, NULL, 65536, 
						(LPTHREAD_START_ROUTINE)(pbyCodeRemote+cbCodeOffset),
						NULL, 0, &dwThreadId)) == NULL)
	 {
		goto finish;
	}

	fSucceeded = TRUE;
finish:
	if(pbyInjCode)
	{
		FreeLocalCode(pbyInjCode);
	}

	GetLastError();
	return fSucceeded;
}

//-----------------------------------------------------------------------
// 测试注入代码到目标进程并load一个dll
//

BOOL InjectLoadLibCode(HANDLE hProcess, LPCTSTR pszLibName)
{
	BOOL fSucceeded = FALSE;
	LPCSTR pcszThunkBegin = THUNK_BEGIN_STR;
	LPCSTR pcszThunkEnd = THUNK_END_STR;
	LPCSTR pcszCodeBegin = CODE_BEGIN_STR;

	UINT cbThunkSize = 0;			// 整段需要copy的代码长度
	UINT cbCodeOffset = 0;		// mem开始距离 maincode起始的偏移 需要用这个作为线程起始地址
	PBYTE pbyCode = NULL;		// 原生代码开始地址
	PBYTE pbyThunk = NULL;

	PBYTE pbyImage = NULL;
	DWORD cbFileSize=0, cbReadSize=0;
	HANDLE hLibFile = INVALID_HANDLE_VALUE;

	// 读取文件
	hLibFile = CreateFile(pszLibName, 
		GENERIC_READ, FILE_SHARE_READ, NULL, OPEN_EXISTING, 0, NULL);
	if(hLibFile == INVALID_HANDLE_VALUE)
	{
		goto finish;
	}

	cbFileSize = GetFileSize(hLibFile, NULL);
	pbyImage = new BYTE[cbFileSize];

	if (!ReadFile(hLibFile, pbyImage, cbFileSize, &cbReadSize, NULL) 
		|| cbReadSize!=cbFileSize)
	{
		goto finish;
	}

	// 构建取代码
	pbyCode = (PBYTE)&LoadLibraryCode;
	if (0 != memcmp(pbyCode, pcszThunkBegin, strlen(pcszThunkBegin)))
	{
		OutputDebugStringA("pbyCode error thunk begin tag not find!\n");
		return NULL;
	}

	pbyCode += strlen(pcszThunkBegin);

	// 计算需要copy的size
	pbyThunk = pbyCode;
	while(memcmp(pbyThunk, pcszThunkEnd, strlen(pcszThunkEnd)))
	{
		pbyThunk ++;
	}
	cbThunkSize = (UINT)(pbyThunk - pbyCode);

	// 计算函数起始地址离mem的偏移量
	pbyThunk = pbyCode;
	while(memcmp(pbyThunk, pcszCodeBegin, strlen(pcszCodeBegin)))
	{
		pbyThunk ++;
	}
	cbCodeOffset = (UINT)(pbyThunk - pbyCode + strlen(pcszCodeBegin));

	// 生成本地代码 并初始化
	pbyThunk = new BYTE[cbThunkSize];
	if (!pbyThunk)
	{
		goto finish;
	}

	memcpy(pbyThunk, pbyCode, cbThunkSize);

	// 初始化三个API _RO_LoadLibraryA, _RO_GetProcAddress, _RO_FreeLibrary
	*(LOADLIBRARYA *)(pbyThunk + 0x04) = &::LoadLibraryA;
	*(GETPROCADDRESS *)(pbyThunk + 0x08) = &::GetProcAddress;
	*(FREELIBRARY *)(pbyThunk + 0x0C) = &::FreeLibrary;

	PBYTE pbyImageRemote = NULL;
	PBYTE pbyThunkRemote = NULL;
	DWORD dwOldProtect = 0;
	DWORD dwNumBytesXferred = 0;
	HANDLE hThread = NULL;
	DWORD dwThreadId = 0;

	// 在目标进程申请地址，写入DLL 的数据
	pbyImageRemote = (PBYTE)VirtualAllocEx(hProcess, NULL, cbFileSize,
		MEM_COMMIT | MEM_TOP_DOWN,
		PAGE_READWRITE);
	if(!pbyImageRemote)
	{
		goto finish;
	}

	if (!VirtualProtectEx(hProcess, pbyImageRemote, cbFileSize,
		PAGE_EXECUTE_READWRITE, &dwOldProtect)) 
	{
		goto finish;
	}

	if (!WriteProcessMemory(hProcess, pbyImageRemote,
		pbyImage, cbFileSize, &dwNumBytesXferred)) 
	{
		goto finish;
	}

	// 将image地址填写到code中
	*(PBYTE *)(pbyThunk + 0x0) = pbyImageRemote;

	// 在目标进程申请地址，写入代码并启动远程线程
	pbyThunkRemote = (PBYTE)VirtualAllocEx(hProcess, NULL, cbThunkSize,
		MEM_COMMIT | MEM_TOP_DOWN,
		PAGE_EXECUTE_READWRITE);
	if(!pbyThunkRemote)
	{
		goto finish;
	}

	if (!VirtualProtectEx(hProcess, pbyThunkRemote, cbThunkSize,
		PAGE_EXECUTE_READWRITE, &dwOldProtect)) 
	{
		goto finish;
	}

	if (!WriteProcessMemory(hProcess, pbyThunkRemote,
		pbyThunk, cbThunkSize, &dwNumBytesXferred)) 
	{
		goto finish;
	}

	if ((hThread = CreateRemoteThread(hProcess, NULL, 65536, 
		(LPTHREAD_START_ROUTINE)(pbyThunkRemote+cbCodeOffset),
		NULL, 0, &dwThreadId)) == NULL)
	{
		goto finish;
	}

	fSucceeded = TRUE;

finish:
	if (pbyImage)
	{
		delete[] pbyImage;
	}

	if(pbyThunk)
	{
		delete[] pbyThunk;
	}

	GetLastError();
	return fSucceeded;
}
```

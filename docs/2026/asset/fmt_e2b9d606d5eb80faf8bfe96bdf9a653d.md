
```
#include <stdio.h>

#include <windows.h>



#define FLAG_EAX 0x00000001

#define FLAG_EBX 0x00000002

#define FLAG_ECX 0x00000004

#define FLAG_EDX 0x00000008

#define FLAG_EDI 0x00000010

#define FLAG_ESI 0x00000020

#define FLAG_CALL 0x00000040



struct InstructionEntryStruct

{

	char *pLabel;



	BYTE bInstruction[16];

	DWORD dwInstructionLength;



	DWORD dwInstructionAddr;



	DWORD dwEax;

	DWORD dwEbx;

	DWORD dwEcx;

	DWORD dwEdx;

	DWORD dwEdi;

	DWORD dwEsi;

	DWORD dwInstructionFlags;

};



DWORD dwGlobal_CurrInstruction = 0;

CONTEXT Global_OrigContext;



InstructionEntryStruct Global_InstructionList[] =

{

	// allocate 1kb buffer for messagebox title using GlobalAlloc

	{ "push ecx", { 0x51 }, 1, 0, 0, 0, 1024, 0, 0, 0, FLAG_ECX },

	{ "push ecx", { 0x51 }, 1, 0, 0, 0, GMEM_FIXED, 0, 0, 0, FLAG_ECX },

	{ "call eax ; (GlobalAlloc)", { 0xFF, 0xD0 }, 2, 0, (DWORD)GlobalAlloc, 0, 0, 0, 0, 0, FLAG_EAX | FLAG_CALL },



	// set messagebox title to "f26K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4R3^5y4X3#2S2N6s2c8Z5k6i4N6Q4x3X3g2U0L8$3#2Q4x3U0k6I4N6h3!0@1i4K6y4n7

	{ "mov ebx, eax", { 0x8B, 0xD8 }, 2, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'w' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'w', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'w' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'w', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'w' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'w', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: '.' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, '.', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'x' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'x', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: '8' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, '8', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: '6' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, '6', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'm' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'm', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'a' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'a', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 't' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 't', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 't' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 't', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'h' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'h', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'e' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'e', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'w' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'w', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: '.' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, '.', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'c' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'c', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'o' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'o', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'm' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'm', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; (null) ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, '\0', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },



	// store messagebox title ptr in edi register

	{ "mov edi, eax", { 0x8B, 0xF8 }, 2, 0, 0, 0, 0, 0, 0, 0, 0 },



	// allocate 1kb buffer for messagebox text using GlobalAlloc

	{ "push ecx", { 0x51 }, 1, 0, 0, 0, 1024, 0, 0, 0, FLAG_ECX },

	{ "push ecx", { 0x51 }, 1, 0, 0, 0, GMEM_FIXED, 0, 0, 0, FLAG_ECX },

	{ "call eax ; (GlobalAlloc)", { 0xFF, 0xD0 }, 2, 0, (DWORD)GlobalAlloc, 0, 0, 0, 0, 0, FLAG_EAX | FLAG_CALL },



	// set messagebox text to "A message box from ntdll.dll"

	{ "mov ebx, eax", { 0x8B, 0xD8 }, 2, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'A' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'A', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: ' ' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, ' ', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'm' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'm', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'e' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'e', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 's' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 's', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 's' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 's', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'a' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'a', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'g' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'g', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'e' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'e', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: ' ' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, ' ', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'b' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'b', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'o' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'o', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'x' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'x', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: ' ' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, ' ', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'f' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'f', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'r' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'r', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'o' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'o', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'm' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'm', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: ' ' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, ' ', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'n' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'n', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 't' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 't', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'd' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'd', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'l' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'l', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'l' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'l', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: '.' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, '.', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'd' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'd', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'l' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'l', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; character: 'l' ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, 'l', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "mov byte ptr [ebx], dl ; (null) ", { 0x88, 0x13 }, 2, 0, 0, 0, 0, '\0', 0, 0, FLAG_EDX },

	{ "inc ebx", { 0x43 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },



	// call MessageBoxA

	{ "push ecx", { 0x51 }, 1, 0, 0, 0, MB_OK, 0, 0, 0, FLAG_ECX },

	{ "push edi", { 0x57 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "push eax", { 0x50 }, 1, 0, 0, 0, 0, 0, 0, 0, 0 },

	{ "push ecx", { 0x51 }, 1, 0, 0, 0, 0, 0, 0, 0, FLAG_ECX },

	{ "call eax ; (MessageBoxA)", { 0xFF, 0xD0 }, 2, 0, (DWORD)MessageBoxA, 0, 0, 0, 0, 0, FLAG_EAX | FLAG_CALL },

};



LONG WINAPI ExceptionHandler(EXCEPTION_POINTERS *pExceptionInfo)

{

	InstructionEntryStruct *pCurrInstruction = NULL;



	// ensure this is a breakpoint / single step exception

	if(pExceptionInfo->ExceptionRecord->ExceptionCode != STATUS_BREAKPOINT && pExceptionInfo->ExceptionRecord->ExceptionCode != STATUS_SINGLE_STEP)

	{

		// this is not the exception that we expected - pass this exception to the next handler

		return EXCEPTION_CONTINUE_SEARCH;

	}



	// reset hardware breakpoints

	pExceptionInfo->ContextRecord->Dr0 = 0;

	pExceptionInfo->ContextRecord->Dr7 = 0;



	if(dwGlobal_CurrInstruction == 0)

	{

		// store original context

		memcpy((void*)&Global_OrigContext, (void*)pExceptionInfo->ContextRecord, sizeof(CONTEXT));

	}

	else if(dwGlobal_CurrInstruction >= (sizeof(Global_InstructionList) / sizeof(Global_InstructionList[0])))

	{

		// finished executing all instructions - restore original context

		memcpy((void*)pExceptionInfo->ContextRecord, (void*)&Global_OrigContext, sizeof(CONTEXT));



		// move to the next instruction (after int3)

		pExceptionInfo->ContextRecord->Eip++;



		// continue execution

		return EXCEPTION_CONTINUE_EXECUTION;

	}



	// get current instruction entry

	pCurrInstruction = &Global_InstructionList[dwGlobal_CurrInstruction];



	// set instruction ptr to next instruction

	pExceptionInfo->ContextRecord->Eip = pCurrInstruction->dwInstructionAddr;



	// check register flags

	if(pCurrInstruction->dwInstructionFlags & FLAG_EAX)

	{

		// set eax

		printf("<InternalExHandler> mov eax, 0x%x\n", pCurrInstruction->dwEax);

		pExceptionInfo->ContextRecord->Eax = pCurrInstruction->dwEax;

	}

	else if(pCurrInstruction->dwInstructionFlags & FLAG_EBX)

	{

		// set ebx

		printf("<InternalExHandler> mov ebx, 0x%x\n", pCurrInstruction->dwEbx);

		pExceptionInfo->ContextRecord->Ebx = pCurrInstruction->dwEbx;

	}

	else if(pCurrInstruction->dwInstructionFlags & FLAG_ECX)

	{

		// set ecx

		printf("<InternalExHandler> mov ecx, 0x%x\n", pCurrInstruction->dwEcx);

		pExceptionInfo->ContextRecord->Ecx = pCurrInstruction->dwEcx;

	}

	else if(pCurrInstruction->dwInstructionFlags & FLAG_EDX)

	{

		// set edx

		printf("<InternalExHandler> mov edx, 0x%x\n", pCurrInstruction->dwEdx);

		pExceptionInfo->ContextRecord->Edx = pCurrInstruction->dwEdx;

	}

	else if(pCurrInstruction->dwInstructionFlags & FLAG_EDI)

	{

		// set edi

		printf("<InternalExHandler> mov edi, 0x%x\n", pCurrInstruction->dwEdi);

		pExceptionInfo->ContextRecord->Edi = pCurrInstruction->dwEdi;

	}

	else if(pCurrInstruction->dwInstructionFlags & FLAG_ESI)

	{

		// set esi

		printf("<InternalExHandler> mov esi, 0x%x\n", pCurrInstruction->dwEsi);

		pExceptionInfo->ContextRecord->Esi = pCurrInstruction->dwEsi;

	}



	// print current instruction label

	printf("<ntdll: 0x%08X> %s\n", pCurrInstruction->dwInstructionAddr, pCurrInstruction->pLabel);



	// check if this is a 'call' instruction

	if(pCurrInstruction->dwInstructionFlags & FLAG_CALL)

	{

		// set a hardware breakpoint on the first instruction after the 'call'

		pExceptionInfo->ContextRecord->Dr0 = pCurrInstruction->dwInstructionAddr + pCurrInstruction->dwInstructionLength;

		pExceptionInfo->ContextRecord->Dr7 = 1;

	}

	else

	{

		// single step

		pExceptionInfo->ContextRecord->EFlags |= 0x100;

	}



	// move to the next instruction

	dwGlobal_CurrInstruction++;



	// continue execution

	return EXCEPTION_CONTINUE_EXECUTION;

}



DWORD GetModuleCodeSection(DWORD dwModuleBase, DWORD *pdwCodeSectionStart, DWORD *pdwCodeSectionLength)

{

	IMAGE_DOS_HEADER *pDosHeader = NULL;

	IMAGE_NT_HEADERS *pNtHeader = NULL;

	IMAGE_SECTION_HEADER *pCurrSectionHeader = NULL;

	char szCurrSectionName[16];

	DWORD dwFound = 0;

	DWORD dwCodeSectionStart = 0;

	DWORD dwCodeSectionLength = 0;



	// get dos header ptr (start of module)

	pDosHeader = (IMAGE_DOS_HEADER*)dwModuleBase;

	if(pDosHeader->e_magic != IMAGE_DOS_SIGNATURE)

	{

		return 1;

	}



	// get nt header ptr

	pNtHeader = (IMAGE_NT_HEADERS*)((BYTE*)pDosHeader + pDosHeader->e_lfanew);

	if(pNtHeader->Signature != IMAGE_NT_SIGNATURE)

	{

		return 1;

	}



	// loop through all sections

	for(DWORD i = 0; i < pNtHeader->FileHeader.NumberOfSections; i++)

	{

		// get current section header

		pCurrSectionHeader = (IMAGE_SECTION_HEADER*)((BYTE*)pNtHeader + sizeof(IMAGE_NT_HEADERS) + (i * sizeof(IMAGE_SECTION_HEADER)));



		// pCurrSectionHeader->Name is not null terminated if all 8 characters are used - copy it to a larger local buffer

		memset(szCurrSectionName, 0, sizeof(szCurrSectionName));

		memcpy(szCurrSectionName, pCurrSectionHeader->Name, sizeof(pCurrSectionHeader->Name));



		// check if this is the main code section

		if(strcmp(szCurrSectionName, ".text") == 0)

		{

			// found code section

			dwFound = 1;

			dwCodeSectionStart = dwModuleBase + pCurrSectionHeader->VirtualAddress;

			dwCodeSectionLength = pCurrSectionHeader->SizeOfRawData;



			break;

		}

	}



	// ensure the code section was found

	if(dwFound == 0)

	{

		return 1;

	}



	// store values

	*pdwCodeSectionStart = dwCodeSectionStart;

	*pdwCodeSectionLength = dwCodeSectionLength;



	return 0;

}



DWORD ScanForInstructions()

{

	DWORD dwInstructionCount = 0;

	DWORD dwCurrSearchPos = 0;

	DWORD dwBytesRemaining = 0;

	DWORD dwFoundAddr = 0;

	DWORD dwCodeSectionStart = 0;

	DWORD dwCodeSectionLength = 0;



	// calculate instruction count

	dwInstructionCount = sizeof(Global_InstructionList) / sizeof(Global_InstructionList[0]);



	// find ntdll code section range

	if(GetModuleCodeSection((DWORD)GetModuleHandle("ntdll.dll"), &dwCodeSectionStart, &dwCodeSectionLength) != 0)

	{

		return 1;

	}



	// scan for instructions

	for(DWORD i = 0; i < dwInstructionCount; i++)

	{

		// check if an address has already been found for this instruction

		if(Global_InstructionList[i].dwInstructionAddr != 0)

		{

			continue;

		}



		// find this instruction in the ntdll code section

		dwCurrSearchPos = dwCodeSectionStart;

		dwBytesRemaining = dwCodeSectionLength;

		dwFoundAddr = 0;

		for(;;)

		{

			// check if the end of the code section has been reached

			if(Global_InstructionList[i].dwInstructionLength > dwBytesRemaining)

			{

				break;

			}



			// check if the instruction exists here

			if(memcmp((void*)dwCurrSearchPos, (void*)Global_InstructionList[i].bInstruction, Global_InstructionList[i].dwInstructionLength) == 0)

			{

				dwFoundAddr = dwCurrSearchPos;

				break;

			}



			// update search indexes

			dwCurrSearchPos++;

			dwBytesRemaining--;

		}



		// ensure the opcode was found

		if(dwFoundAddr == 0)

		{

			printf("Error: Instruction not found in ntdll: '%s'\n", Global_InstructionList[i].pLabel);



			return 1;

		}



		// store address

		Global_InstructionList[i].dwInstructionAddr = dwFoundAddr;



		// copy this instruction address to any other matching instructions in the list

		for(DWORD ii = 0; ii < dwInstructionCount; ii++)

		{

			// check if the instruction lengths match

			if(Global_InstructionList[ii].dwInstructionLength == Global_InstructionList[i].dwInstructionLength)

			{

				// check if the instruction opcodes match

				if(memcmp(Global_InstructionList[ii].bInstruction, Global_InstructionList[i].bInstruction, Global_InstructionList[i].dwInstructionLength) == 0)

				{

					// copy instruction address

					Global_InstructionList[ii].dwInstructionAddr = Global_InstructionList[i].dwInstructionAddr;

				}

			}

		}

	}



	return 0;

}



int main()

{

	PVOID (WINAPI *RtlAddVectoredExceptionHandler)(DWORD dwFirstHandler, void *pExceptionHandler);

	DWORD dwThreadID = 0;

	HANDLE hThread = NULL;



	printf("WindowsNoExec - 1b2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4R3^5y4X3#2S2N6s2c8Z5k6i4N6Q4x3X3g2U0L8$3#2Q4y4f1y4F1i4K6g2o6L8W2)9J5y4Y4q4#2L8%4c8Q4x3@1u0Q4x3U0W2Q4x3@1t1`.



	// get RtlAddVectoredExceptionHandler function ptr

	RtlAddVectoredExceptionHandler = (void *(__stdcall *)(unsigned long,void *))GetProcAddress(GetModuleHandle("ntdll.dll"), "RtlAddVectoredExceptionHandler");

	if(RtlAddVectoredExceptionHandler == NULL)

	{

		return 1;

	}



	printf("Adding exception handler...\n");



	// add exception handler

	if(RtlAddVectoredExceptionHandler(1, (void*)ExceptionHandler) == NULL)

	{

		return 1;

	}



	printf("Scanning ntdll to populate instruction list...\n");



	// scan for instructions

	if(ScanForInstructions() != 0)

	{

		return 1;

	}



	printf("Starting...\n\n");



	// breakpoint to trigger exception handler

	_asm int 3



	printf("\nFinished\n");



	return 0;

}
```

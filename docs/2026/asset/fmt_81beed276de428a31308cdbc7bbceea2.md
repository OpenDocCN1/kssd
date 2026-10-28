
```c
#pragma code_seg("PAGE")
#pragma optimize( "", off )
```

extern PVOID kernel32Base;

static void HookTable( void );
static void DetourFunction( void );
static void EndOfInjectedCode( void );
static DWORD beforeEncode( PDWORD stack, DWORD* callbackReturn, IN_PROCESS_DATA* pCallData );
static DWORD BeforeOriginalFunction( DWORD hookIndex, PDWORD originalStack, DWORD* returnParameter, IN_PROCESS_DATA* callData );
static void AfterOriginalFunction( DWORD hookIndex, PDWORD originalStack, DWORD* returnParameter, IN_PROCESS_DATA* callData );

#define JUMP_TO_DETOUR_LOCATION -5
#define CALLDATA_INDEX_LOCATION 0
#define CALLDATA_PARAMETERS_LOCATION 4
#define CALLDATA_CALLTYPE_LOCATION 8
#define CALLDATA_STACK_OFFSET_LOCATION 12
#define TRAMPOLINE_LOCATION 16
#define START_OF_TRAMPOLINE_PATTERN -1

void __declspec(naked) HookTable( void )
{
__asm
{
  push eax
  xor eax, eax
  call phoney_call
phoney_call:
  lea eax, phoney_call
  lea edx, phoney_jump
  sub edx, eax
  pop eax
  add eax, edx
  mov edx, eax
  pop eax
  jmp DetourFunction
phoney_jump:
  EMIT_FOUR( 0xff )
  EMIT_FOUR( 0x0 )
  EMIT_FOUR( 0x0 )
  EMIT_FOUR( 0x0 )
  EMIT_FOUR( 0x90 )
  EMIT_FOUR( 0x90 )
  EMIT_FOUR( 0x90 )
  EMIT_FOUR( 0x90 )
  EMIT_FOUR( 0x90 )
  EMIT_FOUR( 0x90 )
  EMIT_FOUR( 0x90 )
  EMIT_FOUR( 0x90 )
  EMIT_FOUR( 0x90 )
  jmp EndOfInjectedCode
}
}

////////////////////////////////
// Injected functions
////////////////////////////////

void __declspec(naked) DetourFunction( void )
{
PUSH_STACKFRAME();
{
  DWORD hookIndex;
  DWORD parameters;
  DWORD callType;
  DWORD stackOffset;
  PCHAR trampolineFunction;
  IN_PROCESS_DATA* callData;
  PCHAR codeStart;
  PDWORD originalStack;
  DWORD tempStack;
  int loop;
  int parameters4return;
  DWORD parameter2return = 0;
  DWORD continueFlag;
  DWORD register_esp;
  DWORD register_edi;
  DWORD register_esi;
  DWORD register_eax;
  DWORD register_ebx;
  DWORD register_ecx;
  DWORD add2stack;

  // setup to call injected functions
__asm
{
   mov register_esp, esp
   mov register_edi, edi
   mov register_esi, esi
   mov register_eax, eax
   mov register_ebx, ebx
   mov register_ecx, ecx
   // get parameters
   push edx
   mov edx, [edx+CALLDATA_INDEX_LOCATION]
   mov hookIndex, edx
   pop edx
   push edx
   mov edx, [edx+CALLDATA_PARAMETERS_LOCATION]
   mov parameters, edx
   pop edx
   push edx
   mov edx, [edx+CALLDATA_CALLTYPE_LOCATION]
   mov callType, edx
   pop edx
   push edx
   mov edx, [edx+CALLDATA_STACK_OFFSET_LOCATION]
   mov stackOffset, edx
   pop edx
   push edx
   add edx, TRAMPOLINE_LOCATION
   mov trampolineFunction, edx
   pop edx
   // caculate the start address
   xor eax, eax
   call called_without_return
called_without_return:
   pop eax
   lea ebx, DetourFunction
   lea ecx, called_without_return
   sub ecx, ebx
   sub eax, ecx
   mov codeStart, eax
   // data area
   lea ecx, EndOfInjectedCode
   sub ecx, ebx
   add ecx, eax
   mov callData, ecx
   // caculate the last ret address
   mov eax, ebp
   add eax, 4// pushed ebp
   add eax, stackOffset
   mov originalStack, eax
  }

  // setup return call type
  if( callType == CDECL_TYPE )
   add2stack = parameters * sizeof( DWORD );
  else
   add2stack = 0;

  // call pre-injected code
  continueFlag = BeforeOriginalFunction( hookIndex, originalStack, &parameter2return, callData );
  if( continueFlag == (DWORD)TRUE )
  {
   for( loop = parameters; loop > 0; loop-- )
   {
    tempStack = originalStack[loop];
    __asm push tempStack
   }
   // Call trampoline (jumps to original function)
   //
   // Since trampoline is a jump, the return in
   // the original function will come back here.
__asm
   {
    lea ebx, DetourFunction
    lea eax, return_from_trampoline
    sub eax, ebx
    add eax, codeStart
    // construct call
    push eax
    // adjust stack
    sub esp, stackOffset
    // restore registers and call
    mov edi, register_edi
    mov esi, register_esi
    mov eax, register_eax
    mov ebx, register_ebx
    mov ecx, register_ecx
    jmp trampolineFunction
return_from_trampoline:
    add esp, add2stack
    mov parameter2return, eax
   }
   // call post-injected code
   AfterOriginalFunction( hookIndex, originalStack, &parameter2return, callData );
  }

   // prepare to return
  tempStack = *originalStack;
  if( callType == CDECL_TYPE )
   parameters4return = 0;
  else
   parameters4return = parameters;
__asm
  {
   mov eax, parameter2return
   mov ecx, tempStack
   mov edx, parameters4return
   shl edx, 2
   add edx, stackOffset
   POP_STACKFRAME();
   add esp, 4
   add esp, edx
   jmp ecx
  }
__asm mov edx, trampolineFunction
}
POP_STACKFRAME();
__asm jmp edx
}

///////////////////////////////////////////////////////////////
// this function is located in the PGP SDK
// dynamic link library (old=PGP_SDK.DLL, new=PGPsdk.dll)
// This function accepts the callers input and output,
// which may be memory or file based, and converts the input
// into encrypted output
//
// return TRUE to allow encryption
// return FALSE to block encryption
///////////////////////////////////////////////////////////////

DWORD beforeEncode( PDWORD stack, DWORD* callbackReturn, IN_PROCESS_DATA* pCallData )
{
void* contextPtr = (void*)stack[1];
PGPOptionList* optionListPtr = (PGPOptionList*)stack[2];
DWORD dwRet = (DWORD)TRUE;
int index;
int inputType = 0;
void* lpBuffer;
DWORD dwInBufferLen = 0;
PGPOption* currentOption = optionListPtr->options;
PFLFileSpec* fileSpec;

  // Look at the options in the option list
for( index = 0; index < optionListPtr->numOptions; index++ )
{
  if( currentOption->type == 1 )
  {
   // File Input
   inputType = 1;
   fileSpec = (PFLFileSpec*)currentOption->value;
   lpBuffer = fileSpec->data;
   dwInBufferLen = (DWORD)pCallData->plstrlenA((LPCSTR)(lpBuffer));
   break;
  }
  else if( currentOption->type == 2 )
  {
   // Buffer Input
   inputType = 2;
   lpBuffer = (void*)currentOption->value;
   dwInBufferLen = (DWORD)currentOption->valueSize;
   break;
  }
  currentOption++;
}

  // Send buffer or filename to your friends
if(( inputType == 1 || inputType == 2 ) && ( dwInBufferLen > 0 ))
{
  // just blocking this API to show functionality
  dwRet = (DWORD)FALSE;
  *callbackReturn = PGP_BAD_API;
}
return dwRet;
}

DWORD BeforeOriginalFunction( DWORD hookIndex, PDWORD originalStack, DWORD* returnParameter, IN_PROCESS_DATA* callData )
{
if( hookIndex == USERHOOK_beforeEncode )
{
  return beforeEncode( originalStack, returnParameter, callData );
}
return (DWORD)TRUE;
}

void AfterOriginalFunction( DWORD hookIndex, PDWORD originalStack, DWORD* returnParameter, IN_PROCESS_DATA* callData )
{
}

// EndOfInjectedCode - DetourFunction = size of injected code
// Content doesn't matter, so just trap a debug exception
void __declspec(naked) EndOfInjectedCode( void )
{
__asm int 3
}

////////////////////////////////
// End injected functions
////////////////////////////////

PCHAR allocateUserMemory()
{
LONG memorySize;
LONG tableSize;
LONG codeSize;
LONG dataSize;
ULONG buffer[2];
NTSTATUS status;
PCHAR pMemory;
IN_PROCESS_DATA* pData;

  // Calculate sizes
  // table = (DetourFunction - HookTable) * TOTAL_HOOKS
  // code = EndOfInjectedCode - DetourFunction
  // data = sizof( IN_PROCESS_DATA )
__asm
{
  lea eax, HookTable
  lea ebx, DetourFunction
  lea ecx, EndOfInjectedCode
  mov edx, ebx
  sub edx, eax
  mov tableSize, edx
  mov edx, ecx
  sub edx, ebx
  mov codeSize, edx
}

tableSize = tableSize * TOTAL_HOOKS;
dataSize = sizeof( IN_PROCESS_DATA );
memorySize = tableSize + codeSize + dataSize;

  // Allocate memory
buffer[0] = 0;

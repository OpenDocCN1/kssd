

# \[代码\]重定位程序的实现

> 原文：[`www.pediy.com/kssd/pediy10/76638.html`](https://www.pediy.com/kssd/pediy10/76638.html)

代码:

```
// 11-Nov-2008 Create by Benyanwk  
// Enviroment: Windows all platfrom.  
// Note: (1) Do not support x64  
//       (2) Do not support file large than 4GB.  
//       (3) Do not use SEH.  
// Revision:   
//  11-Nov-2008 ver1.0  
#define UNICODE 
#include <windows.h>  
#include <winnt.h>  // include PE file header definition.  
#include <stdio.h>  

LPCTSTR GetErrMsg(void);  

DWORD  
GetSectionsInfo(   
        PVOID pFile,  
        PIMAGE\_SECTION\_HEADER \*pSections  
        );  

DWORD Reloc( HANDLE hFile, ULONG BaseAddr );  

BOOL RelocFile( PVOID pFile, ULONG BaseAddr );  


DWORD RvaToFileOff( PVOID pFile, DWORD Rva );  

BOOL VerifyPE( PVOID pFile );  

ULONG char2ul(char \*lpUlNum);  


int main( int argc, char \*argv\[\] )  
{  
    LPCTSTR lpErrMsg;  
    HANDLE hFile;  

    //  
    // Check the arguments  
    //  
    if( argc != 4 )  
    {  
          
        wprintf( L"Usage: \\n" );  
        wprintf( L"rebase -b baseaddr filepath \\n");  
        return 1;   // indicate argument error  
    }     

    if( strcmp( argv\[1\], "-b" ) != 0 )  
    {  
          
        wprintf( L"Usage: \\n" );  
        wprintf( L"rebase -b baseaddr filepath \\n");  
        return 1;  
    }  
      
    //  
    // Check the File (existence)  
    //  
    char \*lpFilePath = argv\[3\];  

    hFile =  CreateFileA(   
                    lpFilePath,  
                    GENERIC\_READ|GENERIC\_WRITE,  
                    FILE\_SHARE\_READ|FILE\_SHARE\_WRITE,  
                    (LPSECURITY\_ATTRIBUTES)NULL,  
                    OPEN\_EXISTING,  
                    FILE\_ATTRIBUTE\_NORMAL,  
                    (HANDLE)NULL );  

    if( hFile == INVALID\_HANDLE\_VALUE )  
    {  
        lpErrMsg = GetErrMsg();  
        wsprintf( L"CreateFile Failed!errorcode = %s", lpErrMsg );  
        LocalFree( (HLOCAL)lpErrMsg );  

        return 2;   // indicate open file error  
    }  
      
    //   
    // Get the base address  
    //  
    ULONG ulBaseAddr = char2ul( argv\[2\] );  
      
    //  
    // Reloc the file  
    //  
    if( Reloc( hFile, ulBaseAddr ) != 0 )  
    {  
        wprintf( L" Reloc file failed! I'm sorry \\n" );  
        CloseHandle( hFile );  

        return 3;   // indicate reloc error  
    }  

    return 0;   // indicate success  

}  

LPCTSTR GetErrMsg(void)  
{  
    DWORD dwErrCode;   
    LPCTSTR lpErrMsg;  

    dwErrCode = GetLastError();  
      
    FormatMessage(   
        FORMAT\_MESSAGE\_ALLOCATE\_BUFFER|FORMAT\_MESSAGE\_FROM\_SYSTEM,  
        (LPCVOID)NULL,  
        dwErrCode,  
        MAKELANGID( LANG\_NEUTRAL, SUBLANG\_NEUTRAL ),  
        (LPWSTR)&lpErrMsg,  
        0, 
        NULL );  

    return lpErrMsg;  
}  


DWORD Reloc( HANDLE hFile, ULONG BaseAddr )  
{  
    DWORD dwFileSize;  
    HGLOBAL pFile;  // HGLOBAL = HANDLE = void\* = LPVOID  
    DWORD dwReturnBytes;  
    LPCTSTR lpErrMsg;  

    //  
    // Read the File to memory  
    //  
    dwFileSize = GetFileSize( hFile, NULL );  
    pFile = GlobalAlloc( GMEM\_FIXED, (SIZE\_T)dwFileSize );  
    if( ReadFile(     
             hFile,  
             pFile,  
             dwFileSize,  
             &dwReturnBytes,  
             (LPOVERLAPPED)NULL  
            ) == 0 )  
    {  
        lpErrMsg = GetErrMsg();  
        wprintf( L"ReadFile failed! error message = %s", lpErrMsg );  
        GlobalFree( hFile );  
        //  
        // indicate read file error.  
        //  
        return 1;  
    }  
                  
                  

    //  
    // Verify PE file  
    //   
    if( VerifyPE( pFile ) == FALSE )  
    {  
        wprintf( L" this is not a PE file \\n");  
        GlobalFree( pFile );  
        return 1;   // indicate not pe file  
    }  
      
    //  
    // Reloc PE file  
    //  
    if( RelocFile( pFile, BaseAddr ) != TRUE )  
    {  
        wprintf( L" reloc PE file failed!\\n");  
        GlobalFree( pFile );  
        return 2; // indicate reloc pe file failed;  
    }  

    // 
    // Jmp to the begining of the PE file 
    // 

    SetFilePointer( 
        hFile, 
        0, 
        (PLONG)NULL, 
        FILE\_BEGIN 
        ); 
      
    //  
    // Write back the file  
    //  
    if( WriteFile(   
            hFile,  
            pFile,  
            dwFileSize,  
            &dwReturnBytes,  
            (LPOVERLAPPED)NULL  
            ) == 0 )  
    {  
        lpErrMsg = GetErrMsg();  
        wprintf( L" WriteFile failed! %s\\n", lpErrMsg );  

        GlobalFree( pFile );  
        CloseHandle( hFile );  
        return 3; // indicate write file failed!  
    }  


    GlobalFree( pFile );  
    CloseHandle( hFile );  

    printf(" Yes you get it\\n" );  
    return 0;   // indicate success  
      
}  


DWORD RvaToFileOff( PVOID pFile,  DWORD Rva )  
{  
/\*++  
Routine Name:  
    RvaToFileOffset  

Description:  
    Translate the Relative Virtual Address to Raw File Offset  

Arugments:  
    File Map; Rva;  

Return:  
    File Offset  

\--\*/  

    static DWORD dwSections;  
    DWORD i;  
    static PIMAGE\_SECTION\_HEADER pSections;  
    ULONG ulDiffer = 0;           
    static BOOL bFunExecute = FALSE;    // GetSectionInfo should only execute once.(?)  

      
    if( bFunExecute == FALSE )  
    {  
        dwSections = GetSectionsInfo( pFile, &pSections );  
        bFunExecute = TRUE;  
    }  

    for ( i = 0; i < dwSections; i++ )  
    {  
        if( (Rva >= pSections\[i\].VirtualAddress) &&   
                (Rva <= (pSections\[i\].VirtualAddress + pSections\[i\].Misc.VirtualSize)) )      
        {  
            //  
            // indicate in the this section.  
            //  
            ulDiffer = pSections\[i\].VirtualAddress - pSections\[i\].PointerToRawData;  
            break;  
        }  
    }  
      
    return (Rva - ulDiffer);  
}  



DWORD  
GetSectionsInfo(   
        PVOID pFile,  
        PIMAGE\_SECTION\_HEADER \*pSections  
        )  
/\*   
    Return number of sections   
    Return pointer to sections table  

\*/  
{  
    DWORD dwSections;  
    PIMAGE\_SECTION\_HEADER hSections;  
    //  
    // Jump to section table  
    //  
      
    PIMAGE\_DOS\_HEADER pDosHeader = (PIMAGE\_DOS\_HEADER)pFile;  
    PIMAGE\_NT\_HEADERS pNtHeader = (PIMAGE\_NT\_HEADERS)((PCHAR)pFile + pDosHeader->e\_lfanew); // e\_lfanew + 0x3c   
    dwSections = pNtHeader->FileHeader.NumberOfSections; // NumberOfSections + 0x02   

    //   
    // Allocate the memory for section table  
    //  
    hSections = (PIMAGE\_SECTION\_HEADER)GlobalAlloc( GMEM\_FIXED, dwSections\*sizeof(IMAGE\_SECTION\_HEADER) );  
      
    //  
    // Copy the section table information  
    //  
    PIMAGE\_SECTION\_HEADER pSectionHeader = (PIMAGE\_SECTION\_HEADER)(++pNtHeader);  
      
    memcpy( (PVOID)hSections, (PVOID)pSectionHeader, (SIZE\_T)dwSections\*sizeof(IMAGE\_SECTION\_HEADER) );  

    \*pSections = hSections;  
    return dwSections;  
}  

BOOL VerifyPE( PVOID pFile )  
{  
    PIMAGE\_DOS\_HEADER pDosHeader;  
    PIMAGE\_NT\_HEADERS pNtHeader;  

    pDosHeader = (PIMAGE\_DOS\_HEADER)pFile;  
    if ( pDosHeader->e\_magic != 0x5A4D )   // compare with 'MZ'  
        return FALSE;  

    pNtHeader = (PIMAGE\_NT\_HEADERS)((PCHAR)pFile + pDosHeader->e\_lfanew);   // e\_lfanew + 0x3c   
    if ( pNtHeader->Signature != 0x00004550 ) // compare with 'PE\\0\\0'  
        return FALSE;  

    return TRUE;  
}  


ULONG char2ul(char\* lpUlNum)  
{  
    ULONG ulSum = 0;  
    while(\*lpUlNum)  
    {  
        if(\*lpUlNum>='0' && \*lpUlNum<='9')  
            \*lpUlNum = \*lpUlNum - '0';  
        else if(\*lpUlNum>='A' && \*lpUlNum<='F')  
            \*lpUlNum = \*lpUlNum - 'A' + 10;  
        else  
            \*lpUlNum = \*lpUlNum - 'a' + 10;  
        ulSum = ulSum\*16 + \*lpUlNum;  
        lpUlNum++;  
    }  
    return ulSum;  
}  

BOOL RelocFile( PVOID pFile, ULONG BaseAddr )  
{  
    PIMAGE\_NT\_HEADERS pNtHeader;  
    PIMAGE\_DOS\_HEADER pDosHeader;  
    DWORD dwImageBase;  
    DWORD pRelocAddr,dwRelocAddr;  
    DWORD dwDiffer;  
    PWORD  pType;  
    PIMAGE\_RELOCATION pRelocBlock;  
      

    pDosHeader = (PIMAGE\_DOS\_HEADER)pFile;  
    pNtHeader = (PIMAGE\_NT\_HEADERS)((PCHAR)pFile + pDosHeader->e\_lfanew);   // e\_lfanew + 0x3c   

    dwImageBase = pNtHeader->OptionalHeader.ImageBase;  
    dwDiffer = dwImageBase - BaseAddr;  // pay attention to the order  

    // Get reloc table RVA  
    PIMAGE\_DATA\_DIRECTORY pRelocTable = (PIMAGE\_DATA\_DIRECTORY)  
        pNtHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_BASERELOC\].VirtualAddress;   

    // Get reloc table File Offset  
    pRelocBlock = (PIMAGE\_RELOCATION)( (PCHAR)pFile + RvaToFileOff( pFile, (DWORD)pRelocTable ) ); // the pRelocTable is file offset?  

    do  
    {  
        pType = &pRelocBlock->Type;  

        do  
        {  
              
            // Get reloc address's RVA  
            if( ( \*pType && 0x3000) == 0 )  
            {  
                if( ( \*pType && 0xf000) == 0 )  
                    continue;  
                return TRUE;  
            }  


            pRelocAddr = pRelocBlock->VirtualAddress + (\*pType & 0x0fff);     
            // pRelocAddr += dwImageBase;  

            // Get reloc address's File Offset  
            pRelocAddr = RvaToFileOff( pFile, pRelocAddr );  

        //  DWORD test = pRelocAddr;  

            // Go to the Buffer offset  
            pRelocAddr += (DWORD)pFile;  

            // Get the reloc address  
            dwRelocAddr = \*(PDWORD)pRelocAddr;  

            // Calculate the new address  
            dwRelocAddr -= dwDiffer;  
              
            // Copy to the file  
            \*(PDWORD)pRelocAddr = dwRelocAddr;  
        } while( ++pType < (PWORD)((PCHAR)pRelocBlock + pRelocBlock->SymbolTableIndex) );  
              
        pRelocBlock = (PIMAGE\_RELOCATION)((PCHAR)pRelocBlock + pRelocBlock->SymbolTableIndex );  
        }   while( pRelocBlock->VirtualAddress != 0 );// loop the reloc block entry  
      
    pNtHeader->OptionalHeader.ImageBase = BaseAddr; 
    return TRUE;  
}  

```

今天翻出以前写一文章,关于重定位表的.附如下:

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 重定位表\_ 初级.rar  
 |
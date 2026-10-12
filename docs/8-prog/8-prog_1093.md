
# system 权限下降权程序的问题

> 原文：[`www.pediy.com/kssd/pediy10/102475.html`](https://www.pediy.com/kssd/pediy10/102475.html)

下面是一个`system`权限下的降权程序,但是对一些常用命令的结果无法正常显示,比如:`whoami``tasklist`等等:

文章作者：`pt007[at]vip.sina.com`

信息来源：邪恶八进制信息安全团队(`www.eviloctal.com`)

```
#include "windows.h"

#include <process.h>

#include <Tlhelp32.h>

#include <tchar.h>

#include <psapi.h>

#include <stdio.h>

#include <STDLIB.H>

#include <tlhelp32.h>

#pragma comment (lib,"psapi")

int upto_common_user(TCHAR cmdline[256]); //切换到当前活动用户

DWORD GetPIDFromName(char *ProcName);

int make_to_lower1(char *buf,char *lowerbuf);

int PrintProcessNameAndID(DWORD processID);

BOOL EnableDebugPriv();

HANDLE GetProcessHandle(LPSTR szExeName);

/*int APIENTRY WinMain(HINSTANCE hInstance,
                     HINSTANCE hPrevInstance,
                     LPSTR     lpCmdLine,
                     int       nCmdShow)*/

int main(int argc, char **argv) //切换到管理员用户

{

   TCHAR cmdline1[256]={0};

   if(argc <2)

   {

    printf("用法: %s \\"whoami\\"\\n",argv[0]);

        return 0;

   }

   lstrcpy(cmdline1,argv[1]);

   upto_common_user(cmdline1);

   return 0;

}

int upto_common_user(TCHAR cmdline[256]) //切换到管理员用户身份

{

  HANDLE hToken;

  HANDLE   hExp   = GetProcessHandle("EXPLORER.EXE");   

  if(hExp   ==   NULL) 

    return   FALSE;   

  OpenProcessToken(hExp,TOKEN_ALL_ACCESS,&hToken);   

  if(hToken   ==   NULL)   

  return   FALSE;   

  STARTUPINFO si; 

  PROCESS_INFORMATION pi; 

  ZeroMemory(&si,   sizeof(STARTUPINFO));   

  si.cb             =   sizeof(STARTUPINFO);   

  si.lpDesktop   =   "winsta0\\\\default";

  si.wShowWindow = SW_SHOW;

  si.dwFlags=STARTF_USESHOWWINDOW;   

  TCHAR   szParameter[256]   =   "/c ";

  lstrcat(szParameter,cmdline);

  printf("szParameter=%s\\n",szParameter);

  //char * lpAppName="c:\\\\win2003\\\\temp\\\\svchosts.exe";

  //TCHAR   szParameter[10]   =   {0};   

  //char * lpAppName;//="c:\\\\tmp\\\\KeyLoggerTest.exe";

  char path[MAX_PATH];

  GetSystemWindowsDirectory(path,MAX_PATH); //c:\\win2003

  lstrcat(path,"\\\\system32\\\\cmd.exe"); //c:\\win2003\\temp\\klog.txt*/

  //lstrcpy(lpAppName,(char *)path);

  //打开用户的 winsta0

/*   HWINSTA   hwinsta = OpenWindowStation("winsta0", FALSE, 

            WINSTA_ACCESSCLIPBOARD |

            WINSTA_ACCESSGLOBALATOMS |

            WINSTA_CREATEDESKTOP |

            WINSTA_ENUMDESKTOPS |

            WINSTA_ENUMERATE |

            WINSTA_EXITWINDOWS |

            WINSTA_READATTRIBUTES |

            WINSTA_READSCREEN |

            WINSTA_WRITEATTRIBUTES);

  if (hwinsta == NULL){

            printf(_T("open window station err\\n"));

            return 0;

  }

  if (!SetProcessWindowStation(hwinsta)){

            printf(_T("Set window station err\\n"));

            return 0;

}

  //打开 desktop

HDESK hdesk = OpenDesktop("default", 0, FALSE, 

        DESKTOP_CREATEMENU |DESKTOP_CREATEWINDOW |DESKTOP_ENUMERATE|DESKTOP_HOOKCONTROL|

DESKTOP_JOURNALPLAYBACK |

DESKTOP_JOURNALRECORD |

DESKTOP_READOBJECTS |

DESKTOP_SWITCHDESKTOP |

DESKTOP_WRITEOBJECTS);

    if (hdesk == NULL){

    printf("Open desktop err!\\n");

    return 0;

        }

    SetThreadDesktop(hdesk); */

  if(CreateProcessAsUser(hToken,(char *)path,szParameter,NULL,

  NULL,FALSE,CREATE_DEFAULT_ERROR_MODE,NULL,NULL,&si,&pi))   //以 administrator 用户身份执行程序,CREATE_NO_WINDOW,CREATE_NEW_CONSOLE,CREATE_DEFAULT_ERROR_MODE

  {   

        printf("CreateProcessAsUser sucessed!%d\\n",GetLastError());

    CloseHandle(pi.hProcess);   

    CloseHandle(pi.hThread);   

  }

        /*HANDLE hToken,hNewToken;

    HANDLE hProcess;

    DWORD PID1;

        PID1=GetPIDFromName("Explorer.EXE");  //获得 explorer.exe 进程的 PID

        //PID1=964; //explorer.exe 进程的 PID

        printf("explorer.exe's PID=%d\\n",PID1);

    hProcess = ::OpenProcess(PROCESS_ALL_ACCESS,0,PID1);

    if(hProcess == NULL)

    {

                             //printf(NULL, "OpenProcess" , "FF", MB_OK);

                     // MessageBox(NULL,"Error Opening Process",NULL,MB_OK | MB_APPLMODAL | MB_ICONWARNING | MB_SERVICE_NOTIFICATION);

            printf("Error Opening Process!%x\\n",GetLastError());

        return 0;

    }

    if(OpenProcessToken(hProcess,TOKEN_ALL_ACCESS,&hToken) == 0) //TOKEN_ALL_ACCESS

    {

                //        MessageBoxA(NULL, "OpenProcessToken" , "FF", MB_OK);

                //  MessageBox(NULL,"Error Opening Process Token.Err = " ,NULL,MB_OK | MB_APPLMODAL | MB_ICONWARNING | MB_SERVICE_NOTIFICATION);

            printf("Error Opening Process Token!%x\\n",GetLastError());

        return 1;

    }

    //

    // 模拟当前登陆用户

    //

        //DuplicateTokenEx(hToken, NULL, NULL, SecurityIdentification, TokenPrimary, &hNewToken);

//        EnableDebugPriv();

    if(ImpersonateLoggedOnUser(hToken) == 0)

    {

       //MessageBox(NULL,AnsiString("Error Calling ImpersonateLoggedOnUser.Err = " + SysErrorMessage(GetLastError())).c_str(),NULL,MB_OK | MB_APPLMODAL | MB_ICONWARNING | MB_SERVICE_NOTIFICATION);

                             //MessageBoxA(NULL, "D" , "FF", MB_OK);

            printf("Error Calling ImpersonateLoggedOnUser!%x\\n",GetLastError());

        return 0;

    }

        /*TOKEN_PRIVILEGES tkp;        // ptr. to token structure     

    BOOL fResult;              // system shutdown flag   

        LookupPrivilegeValue(NULL,SE_DEBUG_NAME,&tkp.Privileges[0].Luid);     

    tkp.PrivilegeCount = 1;  // one privilege to set         

    tkp.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;     

         fResult=AdjustTokenPrivileges(hToken, FALSE, &tkp, 0, (PTOKEN_PRIVILEGES) NULL, 0); 

         //printf("Error Calling AdjustTokenPrivileges!%x\\n",GetLastError());

         if(fResult==0)

         {

        printf("Error Calling AdjustTokenPrivileges!%x\\n",GetLastError());

     return 0;

         }

        system("whoami");*/

        return 0;

}

DWORD GetPIDFromName(char *ProcName)

{

        /*HANDLE hSnapshot;

        PROCESSENTRY32 ProcStruct;

        DWORD ProcessID = -1;

        int Result;

        char *t1,*t2;

        hSnapshot = CreateToolhelp32Snapshot((DWORD)TH32CS_SNAPPROCESS,0);

        ProcStruct.dwSize = sizeof(PROCESSENTRY32);

        Result = Process32First(hSnapshot,&ProcStruct);

        while(Result)

        {

                //if(AnsiString(ProcStruct.szExeFile).LowerCase().Pos(ProcName1.LowerCase()) > 0)

                t1 = (char *)malloc(strlen(ProcName));

                t2 = (char *)malloc(strlen(ProcStruct.szExeFile));

                make_to_lower1(ProcName,t1);

                make_to_lower1(ProcStruct.szExeFile,t2);

                if(strstr(t1 , t2))

                {

                        ProcessID = ProcStruct.th32ProcessID;

                        if(t1)

                                free(t1);

                        if(t2)

                                free(t2);

                        break;

                }

                Result = Process32Next(hSnapshot,&ProcStruct);

                if(t1)

                        free(t1);

                if(t2)

                        free(t2);

        }

        CloseHandle(hSnapshot);

        return ProcessID;*/

        DWORD aProcesses[1024], cbNeeded, cProcesses;

        unsigned int i;

        int n=0;

        if(!EnumProcesses(aProcesses,sizeof(aProcesses),&cbNeeded))

                return 0;

        cProcesses = cbNeeded/sizeof(DWORD);
```



该文档包含一段 C/C++ 代码，用于处理进程句柄、权限及字符串转换。

```c
        for(i=0;i<cProcesses;i++)  

        { n=PrintProcessNameAndID(aProcesses);  

           if(n==1)  

           {  

                   break;  

           }  

        }  

        return aProcesses;  

}  

int PrintProcessNameAndID(DWORD processID)  

{  

        TCHAR szProcessName\[MAX_PATH\] = _T("<unknown>");  

        //TCHAR *szProcessName= _T("<unknown>");  

        char * szProcessName1 = "";  

        int num=0;  

        HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS,FALSE,processID);  

        //Process name.  

        if(NULL!=hProcess)  

        {  

                HMODULE hMod;  

                DWORD cbNeeded;  

                if(EnumProcessModules(hProcess,&hMod,sizeof(hMod), &cbNeeded))  

                {  

                        GetModuleBaseName(hProcess,hMod,szProcessName,sizeof(szProcessName)/sizeof(TCHAR));  

                }  

        }  

        //printf(_T("PID: %d (%s) \n"),processID,szProcessName);  

        //if (szProcessName==_T("360tray.exe"))  

        if (!lstrcmp(szProcessName,_T("Explorer.EXE")))//Explorer.EXE  

        {  

                printf("PID: %d (%s)\n ",processID,szProcessName);  

                num=1;  

                return num;  

        }  

    else  

        {  

                return num;  

        }  

        CloseHandle(hProcess);  

}  

BOOL EnableDebugPriv()  

{  

        HANDLE hToken;  

        LUID sedebugnameValue;  

        TOKEN_PRIVILEGES tkp;  

        if ( ! OpenProcessToken( GetCurrentProcess(),  

                TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, &hToken ) )  

        {  

                MessageBox(NULL, "fail", "fail", MB_OK);  

                return FALSE;  

        }  

        if ( ! LookupPrivilegeValue( NULL, SE_DEBUG_NAME, &sedebugnameValue ) )  

        {  

                MessageBox(NULL, "fail", "fail", MB_OK);  

                CloseHandle( hToken );  

                return FALSE;  

        }  

        tkp.PrivilegeCount = 1;  

        tkp.Privileges[0].Luid = sedebugnameValue;  

        tkp.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;  

        if ( ! AdjustTokenPrivileges( hToken, FALSE, &tkp, sizeof tkp, NULL, NULL ) )  

        {  

                MessageBox(NULL, "fail", "fail", MB_OK);  

                return FALSE;  

        }  

        CloseHandle( hToken );  

        return TRUE;  

}  

int make_to_lower1(char *buf,char *lowerbuf)  

{  

        if (buf == NULL)  

                return 1;  

        while (buf[0] != 0) {  

                lowerbuf[0] = tolower(buf[0]);  

                buf++;  

                lowerbuf++;  

        }  

        return 1;  

}  

HANDLE GetProcessHandle(LPSTR szExeName)   

{   

PROCESSENTRY32 Pc = { sizeof(PROCESSENTRY32) };   

  HANDLE hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPALL, 0);   

  if(Process32First(hSnapshot, &Pc)){   

         do{   

                   if(!stricmp(Pc.szExeFile, szExeName)) {   //返回 explorer.exe 进程的 PID   

                            printf("explorer's PID=%d\n",Pc.th32ProcessID);   

                     return OpenProcess(PROCESS_ALL_ACCESS, TRUE, Pc.th32ProcessID);   

            }   

         }while(Process32Next(hSnapshot, &Pc));   

    }   

      return NULL;   

}
```

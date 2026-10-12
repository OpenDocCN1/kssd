
# 自己写的简单的装载、卸载驱动的代码

> 原文：[`www.pediy.com/kssd/pediy12/143930.html`](https://www.pediy.com/kssd/pediy12/143930.html)

简单的装载、启动、停止、卸载驱动-还是喜欢命令行的感觉。`OSR` 的 `driver Loader` 比较经典功能强大，但自己前期写潜伏的 `kit`，只需要简单的载入即可，就写了这个小程序。

代码:

```
/*
* Loaddriver
*By Philomela
* 2011
*See Useage()
*/
#include <windows.h>
#include <stdio.h>
#include <tchar.h>
#include <windef.h>
#define printError printf
char aPath[1024];
char *theDrivername;
//Up the privilege
BOOL up()
{
  HANDLE     hToken,hProcess;
  TOKEN_PRIVILEGES tp;
  char *pSEDEBUG="SeDebugPrivilege";
  hProcess=GetCurrentProcess();
        
  if (!OpenProcessToken(hProcess,TOKEN_ADJUST_PRIVILEGES|TOKEN_QUERY,&hToken))
  {
    printError(TEXT("OpenProcessToken"));      return FALSE;
  }
  if (!LookupPrivilegeValue(NULL,pSEDEBUG,&tp.Privileges[0].Luid))
  {
    printError(TEXT("LookupPrivilegeValue"));  //printf("无法找到指定权限:%s",pSEDEBUG);
    return FALSE;
  }
   
  tp.PrivilegeCount=1;
  tp.Privileges[0].Attributes=SE_PRIVILEGE_ENABLED;       
  if (AdjustTokenPrivileges(hToken,FALSE,&tp,NULL,NULL,NULL) != 0)
    return TRUE;
  else 
  {
    printError(TEXT("AdjustTokenPrivileges"));   
    return FALSE;
  }
}
BOOL _util_load_sysfile(char *theDrivername)
{
  char aPath[1024];
  char aCurrentDirectory[515];
  SC_HANDLE rh;
  SC_HANDLE sh = OpenSCManager(NULL,NULL,SC_MANAGER_ALL_ACCESS);
  if (!sh)
    return FALSE;
  rh = CreateService(sh,theDrivername,theDrivername,SERVICE_ALL_ACCESS,SERVICE_KERNEL_DRIVER,SERVICE_DEMAND_START,\
    SERVICE_ERROR_NORMAL,aPath,NULL,NULL,NULL,NULL,NULL);
  if (!rh)
  {
    printf("Didn`t create a service... %d\n", GetLastError());
    if (GetLastError() == ERROR_SERVICE_EXISTS)
    {
      rh = OpenService(sh,theDrivername,SERVICE_ALL_ACCESS);
      if (!rh)
      {
        printf("Can`t find a Service but it found\n");
        CloseHandle(sh);
        return FALSE;
      }
    }
    else
    {
      _tprintf( TEXT("Service all ready on\n") );
      CloseHandle(sh);
      return FALSE;   
    }
  }
  //start the drivers
  if(rh)
  {
    if( 0 == StartService(rh, 0, NULL))
    {
      if(ERROR_SERVICE_ALREADY_RUNNING == GetLastError())
      {
        printf("Service already running\n");
      }
      else
      {
        printf("Failed on StartService %d\n",GetLastError());
        CloseHandle(sh);
        CloseHandle(rh);
        return FALSE;
      }
    }
    CloseServiceHandle(sh);
    CloseServiceHandle(rh);
  }
  return TRUE;   
}
void Usage() {
  _tprintf(_T("load x [-r -d -g -s -k]  [default -r -g]\n"));
  _tprintf(_T("\t -r 注册\n"));
  _tprintf(_T("\t -d 取消注册\n"));
  _tprintf(_T("\t -g 启动驱动\n"));
  _tprintf(_T("\t -s 停止驱动\n"));
  _tprintf(_T("\t -k 停止并取消注册\n"));
}
BOOL deleteService(SC_HANDLE sh,SC_HANDLE rh) 
{ 
    rh= OpenService( 
        sh,       // SCManager database 
        theDrivername, // name of service 
        SERVICE_ALL_ACCESS);            // only need DELETE access 

    if (rh == NULL)
    { 
        _tprintf(_T("OpenService failed (%d)\n"), GetLastError()); 
        return FALSE;
    }

    if (! DeleteService(rh) ) 
    {
        _tprintf(_T("DeleteService failed (%d)\n"), GetLastError()); 
        return FALSE;
    } 
  _tprintf(_T("Delete service succeeded\n")); 
    return TRUE;
}
BOOL stop(SC_HANDLE sh,SC_HANDLE rh) {
  SERVICE_STATUS ssStatus; 
  rh= OpenService( 
        sh,       // SCManager database 
        theDrivername, // name of service 
        SERVICE_ALL_ACCESS);            // only need DELETE access 
  if (! rh) {
    _tprintf(_T(" OpenService Failed :%d\n"),GetLastError());
    return FALSE;
  }
  if (! ControlService( 
            rh,   // handle to service 
            SERVICE_CONTROL_STOP,   // control value to send 
            &ssStatus) )  // address of status info 
    {
        printf("ControlService failed (%d)\n", GetLastError()); 
        return FALSE;
    }
  _tprintf(_T("Stop service success\n"));
  return TRUE;
}

BOOL create(SC_HANDLE sh) {
  if (!CreateService(sh,theDrivername,theDrivername,SERVICE_ALL_ACCESS,
    SERVICE_KERNEL_DRIVER,SERVICE_DEMAND_START,
    SERVICE_ERROR_NORMAL,aPath,NULL,NULL,NULL,NULL,NULL) ) {
    if(ERROR_SERVICE_ALREADY_RUNNING == GetLastError())
    {
      _tprintf(_T("Service already running\n"));
      return FALSE;
    }
    else if(ERROR_SERVICE_EXISTS == GetLastError())
    {
      _tprintf(_T("Service exists\n"));
      return FALSE;
    }
    _tprintf(_T("Failed on CreateService %d\n"),GetLastError());   
    return FALSE;
  }
  _tprintf(_T("Create service success\n") );   
  return TRUE;
}

BOOL run(SC_HANDLE sh,SC_HANDLE rh) {
    rh = OpenService(sh,theDrivername,SERVICE_ALL_ACCESS);
    if ( StartService(rh, 0, NULL) == 0) {
      if(ERROR_SERVICE_ALREADY_RUNNING == GetLastError())
        _tprintf(_T("Service already running\n"));
      else
        _tprintf(_T("Failed on StartService %d\n"),GetLastError());   
      return FALSE;
    }
    _tprintf(TEXT("run service success\n"));
    return TRUE;
}

void Close(SC_HANDLE sh, SC_HANDLE rh) {
  if(sh)
    CloseServiceHandle(sh);
  if(rh)
    CloseServiceHandle(rh);
}
int main(int argc, TCHAR* argv[]) {
  char aCurrentDirectory[515];
  SC_HANDLE sh = NULL ,rh = NULL;
  BOOL d_flag = FALSE, s_flag = FALSE;
  BOOL r_flag = TRUE, g_flag = TRUE;
  if ( argc < 2 ) {
    Usage();
    return 1;
  }
  up();
  theDrivername = argv[1];
  while ( argc-- > 2) {
    if( argv[argc][0] != '-' ) {
      Usage();
      return 1;
    }
    switch(argv[argc][1]) {
    case 'r':
      g_flag = FALSE;  //不需要启动驱动 
      break;
    case 'd':
      d_flag = TRUE;   
      break;
    case 'g':
      g_flag = TRUE;
      r_flag = FALSE;  //不需要注册
      break;
    case 's':
      s_flag = TRUE;
      break;
    case 'k':
      s_flag = d_flag = TRUE;
      break;
    default: 
      Usage();
      return 1;
    }
  }
  sh = OpenSCManager(NULL,NULL,SC_MANAGER_ALL_ACCESS);
  if (!sh)
    return FALSE;
  GetCurrentDirectory( 512, aCurrentDirectory);
  _snprintf(aPath,1022,"%s\\%s.sys",aCurrentDirectory,theDrivername);
  printf("loading %s\n",aPath);

  //先判断是否需要停止、反注册驱动
  if (s_flag == TRUE) 
    stop(sh,rh);
  if (d_flag == TRUE) 
    deleteService(sh,rh);
  if (s_flag || d_flag) {
  //  Close(sh,rh);
    //_tprintf(_T("\tExit...\n"));
    return 0;
  }
  if (r_flag == TRUE)
    create(sh);
  if (g_flag == TRUE)
    run(sh,rh);
  Close(sh,rh);
  return 0;
}

```

![](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色")

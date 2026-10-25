
```
#include <windows.h>

#include <Tlhelp32.h>

#define BUF 0x0050B000

#define DLLPATH "C:\\Dllinject.dll"



DWORD GetProcessIDbyName(char *name);



int main()

{

  DWORD ProcessID=GetProcessIDbyName("OllyICE.exe");

  if (!ProcessID)

  {

    ProcessID=GetProcessIDbyName("OllyDBG.exe");

    if (!ProcessID)

    {

      MessageBox(NULL,"开了OD了吗?","Not Found!",MB_ICONERROR);

      return -1;

    }

  }

  HANDLE hProcess=OpenProcess(PROCESS_ALL_ACCESS, FALSE, ProcessID);

  char dllpath[MAX_PATH]=DLLPATH;

  if(!WriteProcessMemory(hProcess,(LPVOID)BUF,dllpath,strlen(dllpath),NULL))

  {

    MessageBox(NULL,"写入进程失败!!","Err!",MB_ICONERROR);

    return -1;

  }

  PVOID lpStartAddress=GetProcAddress(LoadLibrary("Kernel32.dll"),"LoadLibraryA");

  if(!CreateRemoteThread(hProcess,NULL,0,(LPTHREAD_START_ROUTINE)lpStartAddress,(LPVOID)BUF,0,NULL))

  {

    MessageBox(NULL,"启动远程线程失败!!","Err!",MB_ICONERROR);

    return -1;

  }

  MessageBox(NULL,"Inject Complete!!","OK!",MB_OK);

  return 0;

}



DWORD GetProcessIDbyName(char *name)

{

  PROCESSENTRY32 stProcess;

  HANDLE hSnapShot=CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS,0);

  

  Process32First(hSnapShot,&stProcess);

  do 

  {

    if(!stricmp(stProcess.szExeFile,name))  //大小写不敏感

    {

      CloseHandle(hSnapShot);

      return stProcess.th32ProcessID;

    }

  } while (Process32Next(hSnapShot,&stProcess));

  //not found

  CloseHandle(hSnapShot);

  return 0;

}
```

使用方法：

1. `Dllinject.dll`放到C盘根目录下

2. 启动OD

3. 想试验的时候运行`inject_tool.exe`

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [`ContinueDebugEvent.JPG`](javascript:void(0);)

  （5.38kb，4353次下载）

- [`GetThreadContext.JPG`](javascript:void(0);)

  （18.13kb，4348次下载）

- [`SetThreadContext.JPG`](javascript:void(0);)

  （15.64kb，4351次下载）

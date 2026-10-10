

# 病毒专杀编写攻略之 ring3 篇

> 原文：[`www.pediy.com/kssd/pediy11/124103.html`](https://www.pediy.com/kssd/pediy11/124103.html)

现在病毒泛滥，采用的技术也越来越邪恶，要么是全盘感染，要么是进程注入，一不小心中了，清除都要清除好久，稍不留意，又会死灰复燃。因此学会写病毒专杀，对付起来就会轻松多了！这篇文章将会从 ring3 层阐述病毒专杀攻略。  
   主要从三部分来阐述，分别是进程相关部分、注册表相关部分、文件相关部分。  
  **进程相关部分**   
一、对抗多进程保护  
   为了使病毒程序能够在电脑中存活更久，现在的病毒都会采用相关的技术来保护自己，一旦病毒进程被结束掉，就会重新创建病毒进程。最常用的就是多进程保护，如双进程保护，三进程保护。一旦病毒进程被结束掉，另外的进程就会检测到并重新创建病毒进程，已达到生生不息的目的。  
   攻略：将进程挂起，然后逐个结束掉  
   代码：  

代码:

```
typedef DWORD (WINAPI \*PFSuspendProcess)(HANDLE hProcess);

PFSuspendProcess SuspendProcess;  //挂起进程的 API，在 ntdlll.dll 中
//函数功能：挂起进程   参数：进程 ID
VOID SuspendProc(DWORD dwPID)
{
  HMODULE hNtDllLib=LoadLibrary("ntdll.dll"); //加载 ntdll.dll，获得 dll 句柄
  SuspendProcess=(PFSuspendProcess)GetProcAddress(hNtDllLib,"ZwSuspendProcess");
//获取 ZwSuspendProcess 的地址
  if (SuspendProcess)
  {
    HANDLE hProcess=OpenProcess(PROCESS\_ALL\_ACCESS,FALSE,dwPID);
      //获取指定进程 ID 的句柄
    SuspendProcess(hProcess); //挂起进程
  }
  FreeLibrary(hNtDllLib);//释放 dll
}
VOID TerminateProc(DWORD dwPID)  //函数功能：结束进程  参数：进程 ID
{
  HANDLE hProcess=OpenProcess(PROCESS\_ALL\_ACCESS,FALSE,dwPID);
  TerminateProcess(hProcess,0);  
}

//函数功能：枚举进程并挂起进程
VOID WINAPI EnumProcessAndSuspendProcess()
{
  HANDLE hProcessSnap;
  PROCESSENTRY32 pe32;
  // Snapshot
  hProcessSnap = CreateToolhelp32Snapshot( TH32CS\_SNAPPROCESS, 0 );
  if( hProcessSnap == INVALID\_HANDLE\_VALUE )
  {
       printf( "CreateToolhelp32Snapshot (of processes) error!\\n");
    return ;
  }
  // 设置输入参数，结构的大小
  pe32.dwSize = sizeof( PROCESSENTRY32 );
  
  // 开始列举进程
  if( !Process32First( hProcessSnap, &pe32 ) )
  {
    printf( "Process32First error!\\n" );  // 出错信息
    CloseHandle( hProcessSnap );
    return ;
  }
  do
  {   
        //枚举进程然后将病毒进程挂起
    if (stricmp(pe32.szExeFile,"Global.exe")==0)
    {
           SuspendProc(pe32.th32ProcessID);
    }
        ...  
        //在这里添加要结束的进程名
  } while( Process32Next( hProcessSnap, &pe32 ) );
  
  CloseHandle( hProcessSnap );  //关闭句柄
  return ;
}
//函数功能：枚举进程并结束进程
VOID WINAPI EnumProcessAndTerminateProcess()
{
  HANDLE hProcessSnap;
  PROCESSENTRY32 pe32;
  // Snapshot
  hProcessSnap = CreateToolhelp32Snapshot( TH32CS\_SNAPPROCESS, 0 );
  if( hProcessSnap == INVALID\_HANDLE\_VALUE )
  {
    printf( "CreateToolhelp32Snapshot (of processes) error!\\n");
    return ;
  }
  // 设置输入参数，结构的大小
  pe32.dwSize = sizeof( PROCESSENTRY32 );
  
  // 开始列举进程
  if( !Process32First( hProcessSnap, &pe32 ) )
  {
    printf( "Process32First error!\\n" );  // 出错信息
    CloseHandle( hProcessSnap );
    return ;
  }
  do
  {   //枚举进程然后将病毒进程结束
    if (stricmp(pe32.szExeFile,"Global.exe")==0)
    {
      TerminateProc(pe32.th32ProcessID);
    }
    ...
        //在这里添加要结束的进程名
  } while( Process32Next( hProcessSnap, &pe32 ) );
  
  CloseHandle( hProcessSnap );  //关闭句柄
  return ;
}

然后在主程序里调用 EnumProcessAndSuspendProcess()和 EnumProcessAndTerminateProcess()就能将病毒程序结束掉。管它几个进程相互保护，都能轻松干掉！

```

 二、对抗 DLL 进程注入  
   病毒进程除了会采用多进程保护之外，还会采用 DLL 注入来实现目的，这种方法相对来说比较隐蔽！必须通过监视工具（如 Icesword）来监视程序结束后又被谁创建，如果进程是系统上的系统进程（如 csrss.exe,svchost.exe)，那就是系统进程被 dll 注入了，但是我们不能直接把系统进程也一起结束了吧,结束完就等着重启吧！那怎么办？  
   攻略：既然病毒能将 dll 注入到进程中，我们也能够将 dll 从进程中卸载掉。  
   代码：  
代码:

```
说明：为了能够卸载掉注入的 dll，首先要打开进程获取进程句柄，但是系统进程是不能访问的，因此要通过提升进程的权限至 SE\_DEBUG 权限才能访问系统进程


//函数功能：提升权限
//参数：lpszPrivilege：权限名  bEnablePrivilege：是否允许
BOOL SetPrivilege(LPCTSTR lpszPrivilege,BOOL bEnablePrivilege)
{
  TOKEN\_PRIVILEGES tp;
  LUID luid;
  HANDLE hProcessToken=NULL;  
  if(!OpenProcessToken(GetCurrentProcess(),TOKEN\_ALL\_ACCESS,&hProcessToken))
    return -1;
  if(!LookupPrivilegeValue(NULL,lpszPrivilege,&luid))
    return FALSE;
  tp.PrivilegeCount = 1;
  tp.Privileges\[0\].Luid=luid;
  if(bEnablePrivilege)
    tp.Privileges\[0\].Attributes =SE\_PRIVILEGE\_ENABLED;
  else
    tp.Privileges\[0\].Attributes =0;
  //Enable the privilege or disable all privilege
  AdjustTokenPrivileges(hProcessToken,FALSE,&tp,sizeof(TOKEN\_PRIVILEGES),(PTOKEN\_PRIVILEGES)NULL,(PDWORD)NULL);
  if(GetLastError()!=ERROR\_SUCCESS)
    return FALSE;
  if(hProcessToken!=NULL)
    CloseHandle(hProcessToken);
  return TRUE;
}

```

提升完权限之后，就是打开所有运行进程，查看里面是否有我们要找的 dll 模块，如果存在，就要获得 dll 句柄，然后用 FreeLibrary 卸载掉。但是在 windows 系统中，各个进程是相互独立的，拥有独立的进程空间，彼此不能访问对方内存空间里的数据。因此在给远程进程创建线程时，无法将参数传递过去，只能将 dll 名写入到进程可以访问的空间，然后调用 FreeLibrary 卸载掉。  
此处代码参考了《黑客防线 2009 黑客编程》中的一篇文章，在此表示感谢）  
代码:

```
//函数功能：卸载掉注入的 dll  参数；dll 名
int KillDLL(char \*DllName)
{
   // 解除所有进程中某 DLL 模块的加载
  HANDLE hProcess=NULL;
  if(!SetPrivilege(SE\_DEBUG\_NAME,TRUE))
  {
    return -2;
  }
  DWORD aProcesses\[1024\],cbNeeded,cProcesses;
  unsigned int i;
  //计算目前有多少进程，aerocesses\[\]用来存放有效的进程 PIDs
    if(!EnumProcesses(aProcesses,sizeof(aProcesses),&cbNeeded)) 
       return -11;
  cProcesses=cbNeeded/sizeof(DWORD);
  //按有效的 PID 遍历所有的进程
  for(i= 0;i<cProcesses;i++)
  {
  if((hProcess=OpenProcess(PROCESS\_ALL\_ACCESS,FALSE,aProcesses\[i\]))==NULL)
    {
      continue;
    }
    // 由目标进程地址空间写入 DLL 名称
    DWORD dwSize,dwWritten;
    dwSize=strlen(DllName)+1;
    LPVOID lpBuf=VirtualAllocEx(hProcess,NULL,dwSize,MEM\_COMMIT,PAGE\_READWRITE);
      if(lpBuf=NULL)
      {
        CloseHandle(hProcess);
        continue;
      }
  //向其中写入 dll 的名称
    if(WriteProcessMemory(hProcess,lpBuf,(LPVOID)DllName,dwSize,&dwWritten))
    {   
                                    // 若写入字节数与实际写入字节数不相等，仍属失败
      if(dwWritten!=dwSize)
      {
        VirtualFreeEx(hProcess,lpBuf,dwSize,MEM\_DECOMMIT);
        CloseHandle(hProcess);
        continue;
      }
    }
            
    else
    { 
       CloseHandle(hProcess);
              continue; 
    }
    //使目标进程调用 GetModuleHandIe,获得 DLL 在进程中的句柄
    DWORD dwHandle,dwID;
    LPVOID pFunc= GetModuleHandleA;
    HANDLE hThread = CreateRemoteThread(hProcess,NULL,0,(LPTHREAD\_START\_ROUTINE)pFunc,lpBuf,0,&dwID);
    //等待 GetModuleHandle 运行完毕
      WaitForSingleObject(hThread,INFINITE);
    //获得 GetModuleHandle 的返回值
    GetExitCodeThread(hThread,&dwHandle);
    // 释放目标进程中申请的空间
    VirtualFreeEx( hProcess,lpBuf,dwSize,MEM\_DECOMMIT);
    CloseHandle(hThread);
    //使目标进程调用 FreeLibrary,卸载 DLL
      pFunc=FreeLibrary;
    hThread= CreateRemoteThread(hProcess,NULL,0,(LPTHREAD\_START\_ROUTINE)pFunc,(LPVOID)dwHandle,0,&dwID);
    //等待 FreeLibrary 卸载完毕
         WaitForSingleObject(hThread,INFINITE);
     CloseHandle(hThread);
    CloseHandle(hProcess);
  }
      if(hProcess!=NULL)
      CloseHandle(hProcess);
       return 0; 
}

```

**注册表相关部分**  
一、对抗自启动  
   系统中的注册表成了兵家必争之地，病毒通常修改注册表来达到自启动的目的。  
   攻略：删除掉相关的注册表项  
   代码：  
代码:

```
VOID DeleteRunouceRegistry()
{   
  HKEY hTestKey;
  CHAR szBuf\[128\];
  if(RegOpenKeyEx(HKEY\_LOCAL\_MACHINE,"SOFTWARE\\\\Microsoft\\\\Windows\\\\CurrentVersion\\\\Run",0,KEY\_READ|KEY\_WRITE,&hTestKey)==ERROR\_SUCCESS)
  {
    if(RegDeleteValue(hTestKey,"Runouce")!=ERROR\_SUCCESS)
    {
      sprintf(szBuf,"%d",GetLastError());
      MessageBox(NULL,szBuf,NULL,MB\_OK);
    }
  }
}

```

这里只是给出一个自启动的例子，病毒自启动的方法相当多，只要将上面的注册表项改成病毒利用的注册表项，就能解决问题。  
二、对抗映像劫持  
   可能有朋友遇到过这样的情况，一个正常的程序，无论把它放在哪个位置，或者是一个程序重新用安装盘修复过，都出现无法运行的情况，或是出错提示为“找不到文件”或者直接没有运行起来的反应，或者是比如运行程序 A 却成了执行 B（可能是病毒），而改名后却可以正常运行的现象。 这就是映像劫持。病毒经常劫持注册表编辑器、任务管理器以及服务管理器等等用户常用的程序，来使病毒运行。  
   攻略：删除 HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\Image File Execution Options\\下的相关注册表项。  
  
代码：  
代码:

```
VOID DeleteRunouceRegistry()//这里以任务管理器为例
{   
  HKEY hTestKey;
  CHAR szBuf\[128\];
  if(RegOpenKeyEx(HKEY\_LOCAL\_MACHINE,"SOFTWARE\\\\Microsoft\\\\Windows NT\\\\CurrentVersion\\\\Image File Execution Options\\\\taskmgr",0,KEY\_READ|KEY\_WRITE,&hTestKey)==ERROR\_SUCCESS)
  {
    if(RegDeleteValue(hTestKey,"Debugger")!=ERROR\_SUCCESS)
    {
      sprintf(szBuf,"%d",GetLastError());
      MessageBox(NULL,szBuf,NULL,MB\_OK);
    }
  }
}

```

三、对抗文件隐藏  
   有时已经将文件夹选项下的“显示所有文件和文件夹”勾上和“隐藏受保护的系统文件”（去掉），但还是看不到文件，这是什么原因呢？原来是注册表中的一个注册表项在作怪。  
   攻略：将 HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\Advanced\\ShowSuperHidden 的值改为 1。  
   由于上面的都是采用编程的方法来实现，现在换种更简单的方法来处理注册表的相关操作，采用批处理。  
代码：  
创建一个.bat 文件，然后在里面添加要处理的注册表项，相关的格式可以参考网上的资料，  
为了减少篇幅，不再叙述。  
代码:

```
reg add "HKCU\\Software\\Microsoft\\Windows\\CurrentVersion\\Explorer\\Advanced" /v ShowSuperHidden /t reg\_dword /d 00000001 /f
reg delete "HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\Image File Execution Options\\taskmgr.exe" /v Debugger /f
reg delete "HKEY\_LOCAL\_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\policies\\Explorer\\Run" /v sys /f

```

上面的批处理是将注册表项 ShowSuperHidden 的值改为 1，删除掉任务管理器的映像劫持，删除掉 sys 项的自启动。  
我们将它命名为 kill.bat，将它和主程序放在同一个文件夹下，然后在主程序里调用执行一下，代码如下：  
代码:

```
GetModuleFileName(NULL,szPath,MAX\_PATH); //获取程序的路径
 lstrcpy( \_tcsrchr(szPath, \_T('\\\\') ) + 1, \_T("kill.bat") );//然后去掉程序名加上 kill.bat
 strcpy(szCmdLine,"cmd.exe /c start ");//WinExec 的命令行
 strcat(szCmdLine,szPath);
 WinExec(szCmdLine,SW\_SHOWNORMAL);

```

这样就很轻松的调用批处理，批处理写起来比 C 方便多了，只要一句就能实现 C 的四五句。  
**文件相关部分**  
一、对抗顽固文件  
   病毒会在运行时释放一些病毒文件，有的是病毒自身的复制，有的是 Dll。为了使病毒能够更久得存活，病毒一般会对文件进行保护，以防止用户将其删除掉。  
  攻略：采用 ring3 下最强悍的 ZwDeleteFile 来删除病毒文件  
 代码：  
代码:

```
HINSTANCE hNtDll;
    ZWDELETEFILE ZwDeleteFile;
    RTLINITUNICODESTRING RtlInitUnicodeString;
    ZWCREATEFILE ZwCreateFile;
    ZWWRITEFILE ZwWriteFile;
    ZWCLOSE ZwClose;
 
    hNtDll = LoadLibrary ("NTDLL");
    if (!hNtDll)
       return 0;
 
    ZwDeleteFile = (ZWDELETEFILE)GetProcAddress (hNtDll,"ZwDeleteFile");
    RtlInitUnicodeString = (RTLINITUNICODESTRING)GetProcAddress (hNtDll,"RtlInitUnicodeString");
    ZwCreateFile = (ZWCREATEFILE)GetProcAddress (hNtDll,"ZwCreateFile");
    ZwWriteFile = (ZWWRITEFILE)GetProcAddress (hNtDll,"ZwWriteFile");
    ZwClose = (ZWCLOSE)GetProcAddress (hNtDll,"ZwClose");
UNICODE\_STRING ObjectName;
    RtlInitUnicodeString(&ObjectName,L"\\\\??\\\\E:\\\\autorun.inf");//记得这里要有\\\\??\\\\在前面的,文件名必须是符号链接或者设备名 
    OBJECT\_ATTRIBUTES ObjectAttributes = {
        sizeof(OBJECT\_ATTRIBUTES),          // Length
        NULL,                               // RootDirectory
        &ObjectName,                        // ObjectName
        OBJ\_CASE\_INSENSITIVE,               // Attributes
        0,                                  // SecurityDescriptor
        NULL,                               // SecurityQualityOfService
    };
 
    HANDLE hFile;
    PVOID content = "ForZwFileTest";
    IO\_STATUS\_BLOCK IoStatusBlock;
 
    ZwCreateFile(&hFile,
        GENERIC\_WRITE|SYNCHRONIZE|GENERIC\_READ,
        &ObjectAttributes,
        &IoStatusBlock,
        0,
        FILE\_ATTRIBUTE\_NORMAL,
        FILE\_SHARE\_DELETE,
        FILE\_OPEN\_IF,
        FILE\_SYNCHRONOUS\_IO\_NONALERT,
        NULL,
        0);
    ZwWriteFile(hFile, 0, 0, 0, &IoStatusBlock, content, 12, NULL, NULL);
    ZwClose(hFile);
 
    ZwDeleteFile(&ObjectAttributes);
 
    FreeLibrary (hNtDll);

```

**总结：**  
   这里只是在 ring3 层来阐述对抗病毒的攻略，但是现在的病毒也开始进入 ring0 层，利用底层技术来搞破坏，因此仅仅在 ring3 层讨论对抗病毒还是不够的，下次希望能够在 ring0 层讲下对抗病毒的攻略。
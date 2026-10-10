

# windows 编程学习笔记(一)

> 原文：[`www.pediy.com/kssd/pediy10/99229.html`](https://www.pediy.com/kssd/pediy10/99229.html)

本人新手，都说要动用五官才能有效果，写下自己的理解，望提点  
      1.WinMain  hInst:HINSTANCE,hPrevInst:HINSTANCE,CmdLine:LPSTR,CmdShow:DWORD   
      参数之一，，HINSTANCE 这个和 HMODULE 是一样的，这个参数是一个基址地址在 win200 上＞＝0x00040000 　invoke GetModuleHandle,NULL 返回进程地址空间中可执行文件的基地址 mov HINSTANCE,eax,  
      参数之二，汇编里面是 NULL,VC 里面是 HINSTANCE,这个看书说是不让编译器发出＂没有引用参数＂的警告  
     参数之 3 pszCmdLine 这个参数是用在创建进程时用到 CreateProcess 的第２个参数 LPTSTR lpCommandLine,一个指针型可由 GetCommandLine 获得一个进程完整命令行指针,如果为空指向进程的名称  
     参数３为 DWORD，一般 SW\_SHOWDEFAULT  
  
    2.CreateProcess  
     BOOL CreateProcess{  
     LPCTSTR lpApplicationName, ；进程的名字  
     LPTSTR  lpCommandLine  ,   ;传递给新进程的命令字符串原型为 PTSTR  
　　LPSECURITY\_ATTRIBUTES lpProcessAttributes,  
;进程属性包含 nlength(SA 结构的长度),lpSecurityDescriptor 如果为０默认，bInheritHandle 是否可继承，为 TRUE 新进程可继承  
    LPSECURITY\_ATTRIBUTES lpThreadAttributes, ;是否可被继承，为 NULL 不能也是一 SA 结构  
    BOOL bInheritHandles, ;标记是否继承原来的 handle  
    DWORD dwCreationFlags,   ;这个太多了，详见 Win32 programmer's reference  
    LPVOID lpEnvironment,  ;环境块，如果为 NULL，则和创建些进程的进程一样环境块诚详见进程的环境变量  
    LPCTSTR lpCurrentDirectory,  ;路径，如果为 NULL 和父进程一样  
    LPSTARTUPINFO lpStartupInfo,  ;指向一个 STARTUPINFO 结构，对 GUI 和 CUI 程序影响各不同为子进程窗口或者控制台进行设置  
    LPPROCESS\_INFORMATION lpProcessInformation   ;些结构包含４个参数 hProcess,  
hThread,dwProcessId,dwThreadId  
}  
      lpEnvironment 即环境块是每一个进程都有的，进程地址空间中分配的一个内存块，每个环境块都包含一组字符串，如  
　　　VerName1=VerValue1\\0  
         ....................................  
         VerNamen=VerValuen\\0   
　　　　　　  
  
  
3.内核对象句柄继承性的一个示例分析,先创建进程Ｂ，再创建Ｃ由Ｂ继承来  
  
                  #include <windows.h>  
     int WINAPI WinMain(HINSTANCE hinstExe,HINSTANCE, PSTR pszCmdLine,int nCmdShow)  
{  
STARTINFO si={sizeof(si)}; //为 CreateProcess 中成员 lpStartupInfo 定义  
SECURITY\_ATTRIBUTES sa\_Process,sa\_Thread; //定义 CreateProcess 第３，４个参数  
PROCESS\_INFORMATION piProcessB,piProcessC; //定义了２个新进程信息该结构包含 hProcess,hThread,dwProcessId,dwThreadId４个参数  
TCHAR szPatch\[MAX\_PATH\];  
  
sa\_Process.nLength=sizeof(sa\_Process);//sa 结构的长度 SECUTIRY\_ATTRIBUTE 共３个参数 nLength,lpSecurityDescripter,bInheritHandles  
sa\_Process.lpSecurityDescripter=NULL;//设为默认  
sa\_Process.bInheritHandle=true; //新创建的进程句柄可以补继承  
  
sa\_Thread.nLength=sizeof(sa\_Thread);//结构和上面一样，这里是线程，因为　一个进程要一个主线程  
sa\_Thread.lpSecurityDescripter=NULL;  
sa\_Thread.bInheritHandle=FALSE; //不能继承  
  
//下面为产生线程 B，为主线程所以第５个 bInheritHandles 继承性为 FALSE  
lstrcpy(szPatch,TEXT("ProcessB"));  
CreateProcess(NULL,szPatch,&sa\_Process,&sa\_Thread,FALSE,0,NULL,NULL,&si,&piProcessB);  
//创建进程Ｃ由Ｂ继承而来  
lstrcpy(szPatch,TEXT("ProcessC"));  
CreateProcess(NULL,szPatch,NULL,NULL,TRUE,0,NULL,NULL,&si,&piProcessC);  
  
return 0;  
}  
  
  
  
4.程序错误处理  
     windows 返回值大概分为以下几种:  
     void 如果返回值为它,该函数运行不会失败  
     bool 相信大家都知道,如果函数失败则为 0,成功为非 0  
       handle 如果函数执行成功返回一个 handle 句柄,失败返回通常为 NULL 比如 CreateWindow,GetTopWindow,GetWindow 等很多  
     pvoid 如果函数运行失败返回为 NULL,成功返回 pvoid 标识数据内存地址我觉得就是个指针  
通过 DWORD GetLastError 可以获得错误信息,该函数返回一个 32 位的错误代码,Mirsoft 在 WinError.h 这个头文件中定义了 21000 行,为以下形式,这只是一个例子,具体查看 WinError.h 中的内容  
          MessageId:ERROR\_SUCCESS               //消息 ID 可以在代码中用些宏  
               MessageText:  
                the operation completed successfully //消息文本,对错误的描述  
               #define ERROR\_SUCCESS  0L              //一个号码  
GetLastError 能返回线程产生的最后一个错误,因为这个函数只对 32 位的有用,对以前的部份 16 位函数不会返回错误信息,比如 MessageBoxEx 这个函数,它是通过 16 位的 MessageBox 实现的,通过 GetLastError 不能错得具体的错误信息  
        可以通过 VC 自带的 error lookup 输入 ID,查看相应的错误描述信息也可以通过 FormatMessage 函数  
        也可以通过自己设定通过 SetLastError 函数实现,函数参数是一个 32 位的,其中第 29 位如果自己设定错误信息必为 1,MICROSOFT 自己的为 0
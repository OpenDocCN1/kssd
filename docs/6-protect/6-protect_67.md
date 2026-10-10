

# 一种躲避运行时代码校验的方法

> 原文：[`www.pediy.com/kssd/pediy10/58896.html`](https://www.pediy.com/kssd/pediy10/58896.html)

一种躲避运行时代码校验的方法  
  
  
cc682/NetRoc  
关键字：代码校验，内存补丁，hook  
    我们有时候需要对运行中的程序打内存补丁，或者对它的代码挂一些钩子之类的工作。但是现在相当多软件进行了运行时的代码检测。一旦发现内存中的代码被修改掉，就会进行处理。本文介绍了一种比较特别的办法，用于通过这些检测。  
    首先需要说一下做运行时代码校验的方法。一般来说，校验者需要取得当前模块的基地址，通过分析 PE 结构，获得代码节的偏移和大小，然后对内存中的代码进行 CRC 或者其他的一些校验。  
这其中有个很大的问题，校验者默认了通过这种方式取得的代码节就是当前被使用到的代码，但是事实却不一定如此。一般编译器正常生成的代码，绝大部分跳转和 call 语句都使用相对地址，因此，我们完全可以把代码节或者整个 exe 文件映像复制到内存其他地方，并操作进程内的所有线程，使得它执行在新复制的那份代码中。这样，校验者仍然在扫描旧的地址，新的那份我们就可以随意修改了。  
    由于一旦进程开始执行，并且创建其他线程之后，我们通过取得线程 Context 获得的 EIP，多半在系统代码中间，所以难以改变。唯一的方法就是，在 exe 的 EntryPoint 被执行前，将 EntryPoint 重定向到新的代码，并 Hook CreateThread，将新线程也重新定位。可以通过下面几个步骤来实现：  
1、  如果想处理的进程为 a.exe，并且 a.exe 是由 b.exe 创建的，那么我们需要 hook 掉 b.exe 的进程创建函数，一般是 CreateProcess。  
2、  在 Hook 的 CreateProcess 中，以 CREATE\_SUSPENDED 标志创建 a.exe。并向 a.exe 中注入我们的 dll，等待这个 dll 完成处理之后才 ResumeThread a.exe 的主线程。  
3、  注入的 dll 需要完成几件事。首先要获得主模块的基地址和大小，并分配足够的空间，将原始映像复制过去。然后 Hook 掉 EntryPoint，并 Hook CreateThread，最后恢复 a.exe 主线程。  
4、  Hook 掉的 EntryPoint 中，恢复被 Hook 的 EntryPoint 代码，防止在后面被检查出来，然后 jmp 到新分配的代码区域即可。  
5、  Hook 的 CreateThread 中，重新计算代码线程函数地址，并修改后创建。这样，所有线程就都在新分配的代码中执行了。  
下面是注入的 dll 的实现代码：  
  
pfnCreateThread g\_pCreateThread = ::CreateThread;  
PBYTE  g\_pbyNewImage = NULL;  
  
#pragma pack(push,1)  
typedef struct \_PUSH\_RETN  
{  
  BYTE byOpcodePush;//0x68  
  DWORD dwRetnAddr;  
  BYTE byOpcodeRetn;//0xC3  
}PUSH\_RETN, \*PPUSH\_RETN;  
#pragma pack(pop)  
  
BYTE g\_abyOldEntry\[6\] = {0};  
PBYTE g\_pbyOldEntry = 0;  
PBYTE g\_pbyNewEntry = 0;  
  
//这里使用 Detours 库 Hook 掉 CreateThread。  
BOOL HookThreadCreate()  
{  
  DetourTransactionBegin();  
  DetourUpdateThread( GetCurrentThread());  
  
  
  if( DetourAttach( &(PVOID&)g\_pCreateThread, Hook\_CreateThread) != NO\_ERROR)  
  {  
    DebugOut( TEXT( "Hook CreateThread fail\\r\\n"));  
  }  
  
  if( DetourTransactionCommit() != NO\_ERROR)  
  {  
    DebugOut( TEXT( "Hook fail\\r\\n"));  
    return FALSE;  
  }  
  else  
  {  
    DebugOut( TEXT( "Hook ok\\r\\n"));  
    return TRUE;  
  }  
}  
  
//Hook 的 CreateThread 里面，重新计算 lpStartAddress 地址，并按这个地址来创建  
HANDLE WINAPI Hook\_CreateThread(  
                LPSECURITY\_ATTRIBUTES lpThreadAttributes,  
                SIZE\_T dwStackSize,  
                LPTHREAD\_START\_ROUTINE lpStartAddress,  
                LPVOID lpParameter,  
                DWORD dwCreationFlags,  
                LPDWORD lpThreadId  
                )  
{  
  PBYTE pfn = (PBYTE)lpStartAddress;  
  HMODULE hMod = ::GetModuleHandle( NULL);  
  pfn = g\_pbyNewImage + (pfn - (PBYTE)hMod);  
  HANDLE hThread = g\_pCreateThread( lpThreadAttributes, dwStackSize, (LPTHREAD\_START\_ROUTINE)pfn, lpParameter, dwCreationFlags, lpThreadId);  
  return hThread;  
}  
  
//Hook 掉的入口点，恢复旧代码并跳转到新分配的代码空间  
\_\_declspec( naked ) VOID Hook\_EntryPoint()  
{  
  //\_asm int 3  
  for ( DWORD i = 0; i < sizeof(g\_abyOldEntry); i++)  
  {//恢复旧的代码  
    g\_pbyOldEntry\[i\] = g\_abyOldEntry\[i\];  
  }  
  \_asm jmp g\_pbyNewEntry;  
}  
  
//Hook 掉入口点  
VOID HookEntryPoint()  
{  
  DebugOut( \_T( "In HookEntryPoint\\r\\n"));  
  
  HMODULE hMod = ::GetModuleHandle( NULL);  
  PIMAGE\_DOS\_HEADER pstDosHeader = (PIMAGE\_DOS\_HEADER)hMod;  
  PIMAGE\_NT\_HEADERS pstHeader = (PIMAGE\_NT\_HEADERS)((PBYTE)hMod + pstDosHeader->e\_lfanew);  
  DWORD dwEntryRVA = pstHeader->OptionalHeader.AddressOfEntryPoint;  
  PBYTE pbyRVA = (PBYTE)(hMod) + dwEntryRVA;  
  PPUSH\_RETN pstHook = (PPUSH\_RETN)pbyRVA;  
  memcpy( g\_abyOldEntry, pbyRVA, sizeof(PUSH\_RETN));  
  pstHook->byOpcodePush = 0x68;  
  pstHook->dwRetnAddr = (DWORD)Hook\_EntryPoint;  
  pstHook->byOpcodeRetn = 0xC3;  
  
  g\_pbyOldEntry = pbyRVA;  
  g\_pbyNewEntry = g\_pbyNewImage + dwEntryRVA;  
  DebugOut( \_T("New image base = 0x%X, New EntryPoint = 0x%X\\r\\n \\  
        Old image base = 0x%X, Old EntryPoint = 0x%X\\r\\n"), g\_pbyNewImage, g\_pbyNewEntry, hMod, g\_pbyOldEntry);  
  DebugOut( \_T( "HookEntryPoint OK\\r\\n"));  
}  
  
//在 DllMain 里面 Process Attach 的时候调用  
VOID OnAttachProcess()  
{  
  HMODULE hMod = ::GetModuleHandle( NULL);  
  DWORD dwSize = GetImageSize();  
  g\_pbyNewImage = new BYTE\[dwSize\];  
  
  DWORD dwOldProtect = 0;  
  VirtualProtect( g\_pbyNewImage, dwSize, PAGE\_EXECUTE\_READWRITE, &dwOldProtect);  
  VirtualProtect( (LPVOID)hMod, dwSize, PAGE\_READWRITE, &dwOldProtect);  
  memcpy( g\_pbyNewImage, (LPVOID)hMod, dwSize);  
  
  HookEntryPoint();  
  HookThreadCreate();  
}  
  
//获得主模块映像大小  
DWORD GetImageSize()  
{  
  HANDLE hShot = NULL;  
  BOOL blResult = FALSE;  
  MODULEENTRY32 stInfo = {0};  
  stInfo.dwSize = sizeof( MODULEENTRY32);  
  TCHAR tszTmp\[MAX\_PATH\] = {0};  
  
  ::GetModuleFileName( GetModuleHandle(NULL), tszTmp, MAX\_PATH);  
  \_tcslwr( tszTmp);  
  size\_t s = \_tcslen(tszTmp);  
  for ( DWORD i = 0; i < s; i++)  
  {  
    if ( tszTmp\[s - i\] == '\\\\')  
    {  
      s = s - i + 1;  
      break;  
    }  
  }  
  hShot = ::CreateToolhelp32Snapshot( TH32CS\_SNAPMODULE, ::GetCurrentProcessId());  
  if ( INVALID\_HANDLE\_VALUE == hShot)  
  {  
    DebugOut( \_T( "CreateToolhelp32Snapshot fail.Err=%d\\r\\n"), GetLastError());  
    return 0;  
  }  
  
  blResult = ::Module32First( hShot, &stInfo);  
  while ( blResult)  
  {  
    \_tcslwr( stInfo.szModule);  
    if ( \_tcscmp( stInfo.szModule, tszTmp + s) == 0)  
    {  
      CloseHandle( hShot);  
      return stInfo.modBaseSize;  
    }  
    blResult = ::Module32Next( hShot, &stInfo);  
  }  
  CloseHandle( hShot);  
  return 0;  
}  
     这种方法对加壳之后的 exe 作用有限，因为 Hook 的并不是真实的 OEP。这样做常常会造成壳执行过程中，或者跳转到 OEP 之后迅速崩溃。本文仅仅是介绍一种思想，有时候巧妙的构思可以使得复杂问题很快得到解决。  
    改良之后，这个技巧可以用到不少地方，呵呵![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")
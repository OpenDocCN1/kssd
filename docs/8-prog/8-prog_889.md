
# 用 C 代实现远程线程(无 dll)

> 原文：[`www.pediy.com/kssd/pediy10/62887.html`](https://www.pediy.com/kssd/pediy10/62887.html)

近日拜读了罗云彬先生写的远程线程代码后，心里佩服不已。

就想自己也写一个练练，由于自己水平不高，在写的过程中遇到不小困难，小小的百来行代码写我一个多星期。

现在把代码贴出来请各位前来“挑挑刺”。

编译平台：Windows 2003, Visual c++ 2003。编译的时候要用 Release，否则代码运行不了。下面符上代码，和符件（所有代码和程序）。

```cpp
#include "win32.h"

#define MAX_SIZE 19

char * szData[MAX_SIZE] = {
  "Kernel32.dll",
  "LoadLibraryA",
  "GetProcAddress",
  "GetModuleHandleA",
  "GDI32.dll",
  "User32.dll",
  "DestroyWindow",
  "PostQuitMessage",
  "DefWindowProcA",
  "LoadCursorA",
  "LoadIconA",
  "RegisterClassExA",
  "CreateWindowExA",
  "ShowWindow",
  "UpdateWindow",
  "GetMessageA",
  "TranslateMessage",
  "DispatchMessageA",
  "MessageBoxA"
};

int _WinMain(LPARAM);

ATOM _MyRegisterClass(HINSTANCE, LPARAM);

BOOL _InitInstance(HINSTANCE hInstance, int, LPARAM);

LRESULT __stdcall _WndProc(HWND, UINT, WPARAM, LPARAM);

DWORD WINAPI RemoteThread(LPARAM);

#pragma code_seg("CodeSeg2")

//导入函数地址表

__declspec(allocate("CodeSeg2"))DWORD lpLoadLibrary=0;   

__declspec(allocate("CodeSeg2"))DWORD lpGetProcAddress=0;

__declspec(allocate("CodeSeg2"))DWORD lpGetModuleHandle=0;

__declspec(allocate("CodeSeg2"))DWORD lpDestroyWindow=0;

__declspec(allocate("CodeSeg2"))DWORD lpPostQuitMessage=0;

__declspec(allocate("CodeSeg2"))DWORD lpDefWindowProc=0;

__declspec(allocate("CodeSeg2"))DWORD lpLoadCursor=0;

__declspec(allocate("CodeSeg2"))DWORD lpLoadIcon=0;

__declspec(allocate("CodeSeg2"))DWORD lpRegisterClassEx=0;

__declspec(allocate("CodeSeg2"))DWORD lpCreateWindowEx=0;

__declspec(allocate("CodeSeg2"))DWORD lpShowWindow=0;

__declspec(allocate("CodeSeg2"))DWORD lpUpdateWindow=0;

__declspec(allocate("CodeSeg2"))DWORD lpGetMessage=0;

__declspec(allocate("CodeSeg2"))DWORD lpTranslateMessage=0;

__declspec(allocate("CodeSeg2"))DWORD lpDispatchMessage=0;

__declspec(allocate("CodeSeg2"))DWORD lpMessageBox=0;

//全局变量:

__declspec(allocate("CodeSeg2"))DWORD _hInstance=0;       

__declspec(allocate("CodeSeg2"))DWORD _hWinMain=0;

//字符串表

__declspec(allocate("CodeSeg2"))char _szClassName[]="RemoteClass";

__declspec(allocate("CodeSeg2"))char _szCaptionMain[]="RemoteWindow";

LRESULT __stdcall _WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)

{

  DWORD lOffset;

  __asm{

    push edx

    call lab

    lab:

    pop edx

    sub edx, lab

    mov lOffset, edx

    pop edx

  }

  switch (message) 

  {

  case WM_DESTROY:

    ReloCationForLP(lpPostQuitMessage,lOffset)(0);

    break;

  default:

    return ReloCationForLP(lpDefWindowProc,lOffset)(hWnd, message, wParam, lParam);

  }

  return 0;

}

DWORD WINAPI RemoteThread(LPARAM lParam){

  ReloCationForDWORD(_hInstance,lParam) = DWORD(ReloCationForLP(lpGetModuleHandle,lParam)(NULL));

  _WinMain(lParam);

  return 0;

}

int _WinMain(LPARAM lParam){

  HINSTANCE hInstance = HINSTANCE(ReloCationForDWORD(_hInstance, lParam));

  MSG Message;

  _MyRegisterClass(hInstance, lParam);

  if(!_InitInstance(hInstance, SW_SHOWNORMAL, lParam))

    return FALSE;

  while(ReloCationForLP(lpGetMessage, lParam)(&Message,0,0,0)){

    ReloCationForLP(lpTranslateMessage, lParam)(&Message);

    ReloCationForLP(lpDispatchMessage, lParam)(&Message);

  }

  return int(Message.wParam);

}

ATOM _MyRegisterClass(HINSTANCE hInstance, LPARAM lParam)

{

  WNDCLASSEX wcex;

  wcex.cbSize = sizeof(WNDCLASSEX); 

  wcex.style      = CS_HREDRAW | CS_VREDRAW;

  wcex.lpfnWndProc  = WNDPROC(DWORD(&_WndProc) + DWORD(lParam));

  wcex.cbClsExtra    = 0;

  wcex.cbWndExtra    = 0;

  wcex.hInstance    = hInstance;

  wcex.hIcon      = ReloCationForLP(lpLoadIcon, lParam)(NULL, IDC_ICON);

  wcex.hCursor    = ReloCationForLP(lpLoadCursor, lParam)(NULL, IDC_ARROW);

  wcex.hbrBackground  = (HBRUSH)(COLOR_WINDOW+1);

  wcex.lpszMenuName  = 0;

  wcex.lpszClassName  = ReloCataonForTCHAR(_szClassName, lParam);

  wcex.hIconSm    = ReloCationForLP(lpLoadIcon, lParam)(NULL, IDC_ICON);

  return ReloCationForLP(lpRegisterClassEx,lParam)(&wcex);

}

BOOL _InitInstance(HINSTANCE hInstance, int nCmdShow, LPARAM lParam)

{

  HWND hWnd;

   hWnd = ReloCationForLP(lpCreateWindowEx,lParam)(WS_EX_CLIENTEDGE, ReloCataonForTCHAR(_szClassName, lParam), 

    ReloCataonForTCHAR(_szCaptionMain, lParam), WS_OVERLAPPEDWINDOW,

      100, 100, 600, 400, NULL, NULL, hInstance, NULL);

   if (!hWnd)

   {

      return FALSE;

   }

   ReloCationForLP(lpShowWindow,lParam)(hWnd, nCmdShow);

   ReloCationForLP(lpUpdateWindow,lParam)(hWnd);

   return TRUE;

}

#pragma code_seg()

// win32.cpp : 定义应用程序的入口点。

//

int __stdcall Enter()

{

  DWORD dwProcessID;

  HANDLE hProcess, hThread;

  LPVOID lpBaseAddress;  //远程线程基址

  LPTHREAD_START_ROUTINE lpStartAddress;  //远程线程入口地址

  HMODULE hModule = 0;

  DWORD hMod = DWORD(GetModuleHandle(NULL));

  PIMAGE_DOS_HEADER pDosHdr = (PIMAGE_DOS_HEADER)hMod;

    PIMAGE_NT_HEADERS pNtHdr = (PIMAGE_NT_HEADERS)(hMod + pDosHdr->e_lfanew );

    PIMAGE_SECTION_HEADER pSection = IMAGE_FIRST_SECTION( pNtHdr );

  for (unsigned i = 0;i < pNtHdr->FileHeader.NumberOfSections;i++, pSection++ )

    if (!strncmp("CodeSeg2",(char *)(pSection->Name),8)) break;

  LPDWORD lpData = LPDWORD(&lpLoadLibrary);

  for(int i = 0; i < MAX_SIZE; i ++){

    *lpData = (DWORD)GetProcAddress(hModule, szData[i]);

    if (*lpData) lpData ++;

    else hModule = GetModuleHandle(szData[i]);

  }

  GetWindowThreadProcessId(FindWindow(TEXT("Progman"),TEXT("Program Manager")), &dwProcessID);

  hProcess = OpenProcess(PROCESS_CREATE_THREAD | PROCESS_VM_OPERATION | PROCESS_VM_WRITE, FALSE, dwProcessID);

  if(hProcess){

    lpBaseAddress = VirtualAllocEx(hProcess, NULL, pSection->SizeOfRawData, MEM_COMMIT, PAGE_EXECUTE_READWRITE);

    if(lpBaseAddress){

      lpStartAddress = LPTHREAD_START_ROUTINE(DWORD(lpBaseAddress) + DWORD(RemoteThread) - DWORD(&lpLoadLibrary));

      WriteProcessMemory(hProcess, lpBaseAddress, LPVOID(&lpLoadLibrary), pSection->SizeOfRawData, NULL);

      hThread = CreateRemoteThread(hProcess, NULL, 0, lpStartAddress, LPVOID(DWORD(lpBaseAddress) - DWORD(&lpLoadLibrary)), 0, NULL);

//      WaitForSingleObject(hThread,INFINITE);

//      VirtualFreeEx(hProcess, lpBaseAddress, 0, MEM_RELEASE);

      CloseHandle(hThread);

    }

    CloseHandle(hProcess);

  }

  else{

    MessageBox(NULL, TEXT("无法打开进程！"), TEXT("错误"), 0);

  }

  return 0;

}
```

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `RemoteThread.rar`\[解压密码：pediy\]

 |

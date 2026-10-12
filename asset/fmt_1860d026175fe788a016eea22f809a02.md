
# 阻止远程注入代码

> 原文：[`www.pediy.com/kssd/pediy11/119649.html`](https://www.pediy.com/kssd/pediy11/119649.html)

代码还算比较简短

附件中可编译源代码.

在 `xp sp3` 中试验通过.

引用: `#include "stdafx.h"`

`#include "deinj.h"`

`#define MAX_LOADSTRING 100`

// 全局变量:

`HINSTANCE hInst; // 当前实例`
`TCHAR szTitle[MAX_LOADSTRING]; // 标题栏文本`
`TCHAR szWindowClass[MAX_LOADSTRING]; // 主窗口类名`

// 此代码模块中包含的函数的前向声明:

`ATOM MyRegisterClass(HINSTANCE hInstance);`
`BOOL InitInstance(HINSTANCE, int);`
`LRESULT CALLBACK WndProc(HWND, UINT, WPARAM, LPARAM);`
`void Init();`

```c
int APIENTRY _tWinMain(HINSTANCE hInstance,
                      HINSTANCE hPrevInstance,
                      LPTSTR    lpCmdLine,
                      int       nCmdShow)
{
  UNREFERENCED_PARAMETER(hPrevInstance);
  UNREFERENCED_PARAMETER(lpCmdLine);
  // LoadLibrary(_T("deinjdll.dll"));
  Init();
  
  // TODO: 在此放置代码。
  
  MSG msg;
  HACCEL hAccelTable;
  
  // 初始化全局字符串
  LoadString(hInstance, IDS_APP_TITLE, szTitle, MAX_LOADSTRING);
  
  LoadString(hInstance, IDC_DEINJ, szWindowClass, MAX_LOADSTRING);
  
  MyRegisterClass(hInstance);
  
  // 执行应用程序初始化:
  if (!InitInstance (hInstance, nCmdShow))
  {
    return FALSE;
  }
  
  hAccelTable = LoadAccelerators(hInstance, MAKEINTRESOURCE(IDC_DEINJ));
  
  // 主消息循环:
  while (GetMessage(&msg, NULL, 0, 0))
  {
    if (!TranslateAccelerator(msg.hwnd, hAccelTable, &msg))
    {
      TranslateMessage(&msg);
      DispatchMessage(&msg);
    }
  }
  
  return (int) msg.wParam;
}
```

```c
ATOM MyRegisterClass(HINSTANCE hInstance)
{
  WNDCLASSEX wcex;
  
  wcex.cbSize = sizeof(WNDCLASSEX);
  
  wcex.style     = CS_HREDRAW | CS_VREDRAW;
  wcex.lpfnWndProc  = WndProc;
  wcex.cbClsExtra    = 0;
  wcex.cbWndExtra    = 0;
  wcex.hInstance    = hInstance;
  wcex.hIcon      = LoadIcon(hInstance, MAKEINTRESOURCE(IDI_DEINJ));
  wcex.hCursor    = LoadCursor(NULL, IDC_ARROW);
  wcex.hbrBackground  = (HBRUSH)(COLOR_WINDOW+1);
  wcex.lpszMenuName  = MAKEINTRESOURCE(IDC_DEINJ);
  wcex.lpszClassName  = szWindowClass;
  wcex.hIconSm    = LoadIcon(wcex.hInstance, MAKEINTRESOURCE(IDI_SMALL));
  
  return RegisterClassEx(&wcex);
}
```

```c
BOOL InitInstance(HINSTANCE hInstance, int nCmdShow)
{
   HWND hWnd;
   hInst = hInstance; // 将实例句柄存储在全局变量中
   hWnd = CreateWindow(szWindowClass, szTitle, WS_OVERLAPPEDWINDOW,
     CW_USEDEFAULT, 0, CW_USEDEFAULT, 0, NULL, NULL, hInstance, NULL);
   
   if (!hWnd)
   {
      return FALSE;
   }
   
   ShowWindow(hWnd, nCmdShow);
   UpdateWindow(hWnd);
   
   return TRUE;
}
```

```c
LRESULT CALLBACK WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
{
  int wmId, wmEvent;
  
  switch (message)
  {
  case WM_COMMAND:
    wmId    = LOWORD(wParam);
    wmEvent = HIWORD(wParam);
    
    // 分析菜单选择:
    switch (wmId)
    {
    case IDM_EXIT:
      DestroyWindow(hWnd);
      break;
    default:
      return DefWindowProc(hWnd, message, wParam, lParam);
    }
    
    break;
    
  case WM_DESTROY:
    PostQuitMessage(0);
    break;
    
  default:
    return DefWindowProc(hWnd, message, wParam, lParam);
  }
  
  return 0;
}
```

//////////////////////////////////////////////////////////////////////////
// 以下是我增加的代码

```c
void handleThreadAttached();

BOOL APIENTRY FakeMain( HMODULE hModule, DWORD reason,LPVOID )
{
  if (reason == DLL_THREAD_ATTACH) {
    handleThreadAttached();
  }
  
  return TRUE;
}

enum THREADINFOCLASS {
  ThreadQuerySetWin32StartAddress = 9,
};

typedef LONG (NTAPI * TNtQueryInformationThread)(IN HANDLE ThreadHandle, IN THREADINFOCLASS ThreadInformationClass,
                             OUT PVOID ThreadInformation,IN ULONG ThreadInformationLength,
                             OUT PULONG ReturnLength OPTIONAL);

TNtQueryInformationThread NtQueryInformationThread = NULL;

void handleThreadAttached()
{
  if (NtQueryInformationThread == NULL) {
    NtQueryInformationThread = (TNtQueryInformationThread )GetProcAddress(GetModuleHandle(_T("ntdll.dll")), 
      "NtQueryInformationThread");
  }
  
  if (NtQueryInformationThread == NULL) {
    assert(false);
    return;
  }
  
  PVOID startAddr = NULL;
  ULONG len = sizeof(startAddr);
  
  if (NtQueryInformationThread(GetCurrentThread(), ThreadQuerySetWin32StartAddress, &startAddr, len, &len)) {
    assert(false);
    return;
  }
  
  MEMORY_BASIC_INFORMATION memInfo;
  
  if (VirtualQuery(startAddr, &memInfo, sizeof(memInfo)) != sizeof(memInfo)) {
    assert(false);
    return;
  }
  
  if (memInfo.Type != MEM_IMAGE) {
    OutputDebugString(_T("executing in data\n"));
    ExitThread(-1);
  }
}
```

//////////////////////////////////////////////////////////////////////////

```c
#define MakePtr(a, b)    ( ((ULONG_PTR)a ) + ((ULONG_PTR)b ) )

PVOID _GetCurrentPeb()
{
  _asm mov eax, fs:[0x30] 
}

typedef struct _LDR_DATA_TABLE_ENTRY
{
  LIST_ENTRY InLoadOrderModuleList;
  LIST_ENTRY InMemoryOrderModuleList;
  LIST_ENTRY InInitializationOrderModuleList;
  PVOID DllBase;
  PVOID EntryPoint;
  ULONG SizeOfImage; // in bytes
  UNICODE_STRING FullDllName;
  UNICODE_STRING BaseDllName;
  ULONG Flags; // LDR_*
  USHORT LoadCount;
  USHORT TlsIndex;
  LIST_ENTRY HashLinks;
  PVOID SectionPointer;
  ULONG CheckSum;
  ULONG TimeDateStamp;
  // PVOID LoadedImports; // seems they are exist only on XP !!!
  // PVOID EntryPointActivationContext; // -same-
} LDR_DATA_TABLE_ENTRY, *PLDR_DATA_TABLE_ENTRY;

typedef struct _PEB_LDR_DATA
{
  ULONG Length;
  BOOLEAN Initialized;
  PVOID SsHandle;
  LIST_ENTRY InLoadOrderModuleList; // ref. to PLDR_DATA_TABLE_ENTRY->InLoadOrderModuleList
  LIST_ENTRY InMemoryOrderModuleList; // ref. to PLDR_DATA_TABLE_ENTRY->InMemoryOrderModuleList
  LIST_ENTRY InInitializationOrderModuleList; // ref. to PLDR_DATA_TABLE_ENTRY->InInitializationOrderModuleList
} PEB_LDR_DATA, *PPEB_LDR_DATA;

LDR_DATA_TABLE_ENTRY SelfModule;

#define LDRP_IMAGE_DLL                    0x00000004
#define LDRP_PROCESS_ATTACH_CALLED        0x00080000

void Init()
{
  PLIST_ENTRY entry;
  PLIST_ENTRY moduleListHead;
  PLDR_DATA_TABLE_ENTRY module;
  
  PPEB_LDR_DATA ldrData = *(PPEB_LDR_DATA *)MakePtr(_GetCurrentPeb() , 0xc);
  
  moduleListHead = &ldrData->InMemoryOrderModuleList;
  
  entry = moduleListHead->Flink;
  
  module = CONTAINING_RECORD(entry, LDR_DATA_TABLE_ENTRY, InMemoryOrderModuleList);
  
  SelfModule = *module;
  
  SelfModule.DllBase = (PVOID )MakePtr(GetModuleHandle(NULL) , 1);
  
  SelfModule.EntryPoint = &FakeMain;
  
  SelfModule.InMemoryOrderModuleList.Flink = entry->Flink;
  
  SelfModule.InMemoryOrderModuleList.Blink = entry;
  
  entry->Flink = &SelfModule.InMemoryOrderModuleList;
  
  SelfModule.Flags |= LDRP_IMAGE_DLL | LDRP_PROCESS_ATTACH_CALLED;
}
```

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `deinj2.rar` |

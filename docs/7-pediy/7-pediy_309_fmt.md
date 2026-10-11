
# 微软"查找目标的实现"

> 原文：[`www.pediy.com/kssd/pediy06/pediy6789.htm`](https://www.pediy.com/kssd/pediy06/pediy6789.htm)

一直使用 `VC++6.0`，很希望自己实现一个插件能打开包含 `VC` 中打开文件的文件夹。以前用 `ShellExecuteEx` 实现过，但效果不理想，原因有 1，同一个文件夹会打开多个实例，2. 目标文件在文件夹中不会被选中。后来在网上发现可以通过执行 `explore /select` 来完成上述功能，但还不够理想，原因是，如果文件夹已被打开，目标文件在文件夹中不会被选中。

我看了不少有关 `Windows Shell` 的编程方面的书也没有找到答案。

在 `Windows` 桌面上的图标，右单击，选“属性”，按“查找目标”可以非常理想的完成上述功能。但是它是怎样实现的呢？

先打开属性对话框，在 `SoftICE` 下下万能断点 `hmemcpy`，经过几次 `F12`，来到 `Shell32.dll` 中，记下其虚拟地址，用 `IDA` 打开 `Shell32.dll`：

```asm
.text:7FD04025  CreateShortcutPage  proc    near              CODE XREF: sub_7FD05C24+159p

.text:7FD04025
.text:7FD04025  var_34             = PROPSHEETPAGEA ptr -34h
.text:7FD04025  var_4              = dword ptr -4
.text:7FD04025  arg_0              = dword ptr  8
.text:7FD04025  lpString2          = dword ptr  0Ch
.text:7FD04025  arg_8              = dword ptr  10h
.text:7FD04025  arg_C              = dword ptr  14h
.text:7FD04025

.text:7FD04025                    push    ebp
.text:7FD04026                    mov     ebp, esp
.text:7FD04028                    sub     esp, 34h
.text:7FD0402B                    push    esi
.text:7FD0402C                    push    edi
.text:7FD0402D                    push    [ebp+lpString2]
.text:7FD04030                    call    IsLnkFile
.text:7FD04035                    test    eax, eax
.text:7FD04037                    jz      loc_7FD040F6
.text:7FD0403D                    lea     eax, [ebp+var_4]
.text:7FD04040                    push    eax
.text:7FD04041                    push    offset dword_7FCBCE50
.text:7FD04046                    push    0
.text:7FD04048                    call    sub_7FCBCA41
.text:7FD0404D                    test    eax, eax
.text:7FD0404F                    jl      loc_7FD040F6
.text:7FD04055                    push    220h             uBytes
.text:7FD0405A                    push    40h              uFlags
.text:7FD0405C                    call    ds:LocalAlloc
.text:7FD04062                    mov     esi, eax
.text:7FD04064                    test    esi, esi
.text:7FD04066                    jz      loc_7FD040ED
.text:7FD0406C                    mov     eax, ds:hModule
.text:7FD04071                    push    104h             iMaxLength
.text:7FD04076                    push    [ebp+lpString2]  lpString2
.text:7FD04079                    mov     [ebp+var_34.hInstance], eax
.text:7FD0407C                    lea     eax, [esi+18h]
.text:7FD0407F                    mov     [ebp+var_34.dwSize], 30h
.text:7FD04086                    push    eax              lpString1
.text:7FD04087                    mov     [ebp+var_34.dwFlags], 80h
.text:7FD0408E                    mov     dword ptr [ebp+var_34.anonymous_0], 1040
.text:7FD04095                    mov     [ebp+var_34.pfnDlgProc], offset PropDlgProc 注意这里!
.text:7FD0409C                    mov     [ebp+var_34.pfnCallback], offset loc_7FD03FFF
.text:7FD040A3                    mov     [ebp+var_34.lParam], esi
.text:7FD040A6                    call    ds:lstrcpynA
.text:7FD040AC                    mov     eax, [ebp+arg_0]
.text:7FD040AF                    or      dword ptr [esi+14h], 0FFFFFFFFh
.text:7FD040B3                    mov     [esi], eax
.text:7FD040B5                    mov     eax, [ebp+var_4]
.text:7FD040B8                    mov     [esi+4], eax
.text:7FD040BB                    lea     eax, [ebp+var_34]
.text:7FD040BE                    push    eax              LPCPROPSHEETPAGEA
.text:7FD040BF                    call    ds:CreatePropertySheetPageA
.text:7FD040C5                    mov     edi, eax
.text:7FD040C7                    test    edi, edi
.text:7FD040C9                    jz      short loc_7FD040E6
.text:7FD040CB                    push    [ebp+arg_C]
.text:7FD040CE                    push    edi
.text:7FD040CF                    call    [ebp+arg_8]
.text:7FD040D2                    test    eax, eax
.text:7FD040D4                    jz      short loc_7FD040DF
.text:7FD040D6                    push    1
.text:7FD040D8                    pop     eax
.text:7FD040D9

.text:7FD040D9  loc_7FD040D9:                          CODE XREF: CreateShortcutPage+D3j
.text:7FD040D9                    pop     edi
.text:7FD040DA                    pop     esi
.text:7FD040DB                    leave
.text:7FD040DC                    retn    10h
.text:7FD040DF  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
.text:7FD040DF
.text:7FD040DF  loc_7FD040DF:                          CODE XREF: CreateShortcutPage+AFj
.text:7FD040DF                    push    edi          HPROPSHEETPAGE
.text:7FD040E0                    call    ds:DestroyPropertySheetPage
.text:7FD040E6

.text:7FD040E6  loc_7FD040E6:                          CODE XREF: CreateShortcutPage+A4j
.text:7FD040E6                    push    esi          hMem
.text:7FD040E7                    call    ds:LocalFree
.text:7FD040ED

.text:7FD040ED  loc_7FD040ED:                          CODE XREF: CreateShortcutPage+41j
.text:7FD040ED                    mov     eax, [ebp+var_4]
.text:7FD040F0                    push    eax
.text:7FD040F1                    mov     ecx, [eax]
.text:7FD040F3                    call    dword ptr [ecx+8]
.text:7FD040F6

.text:7FD040F6  loc_7FD040F6:                          CODE XREF: CreateShortcutPage+12j
.text:7FD040F6                                        CreateShortcutPage+2Aj
.text:7FD040F6                    xor     eax, eax
.text:7FD040F8                    jmp     short loc_7FD040D9
.text:7FD040F8  CreateShortcutPage  endp
```

经过整整 4 天的艰苦挖掘，得到了后面的源代码。这些源代码和作者(微软)手里的因该是完全等价的。说实在的，里面有些东西是值得学习的。从中我们可以发现问题的关键在于 `SHDOCVW.DLL` 中的 `SHGetIDispatchForFolder` 这个未公开函数，我曾试着在 `google` 搜索它，发现只有一个不能打开的网站上有它。虽然我极其佩服微软，但从这个例子中我们可以看出微软的技术垄断的痕迹。

为了干这活，我还做了一个工具 `CLSIDSEE`，它可以从 4 个 `dword` 查找 `CLS_ID`, `ProgID`...，用于破解 `COM` 代码非常有用。

用这些代码我还做了一个 `VC` 插件，实现开始提到的功能，我个人觉得非常有用。

用这些代码我还做了一个 `Shell` 扩展，将查找目标这个功能直接放到桌面图标右单击的菜单中，我也觉得非常有用。

这三个小工具我会放到 `CrackABC` 的 `FTP` 上。

### 1. 源代码（1）

```c
int GetNumberOfSelected(HGLOBAL hMem)
{
    int ret = 0;
    int* pInt;
    if ((pInt = (int*)GlobalLock(hMem)) != NULL) {
        ret = *pInt;
        GlobalUnlock(hMem);
    }
    return ret;
}

BOOL _7FD0276A(DWORD* vp, DWORD var)
{
    if (var <= vp[0])
        return SHELL32_25((BYTE*)vp + vp[1], (BYTE*)vp + vp[var + 2]);
    return FALSE;
}

LPCITEMIDLIST GetPIDL(HGLOBAL hGlobal, int var)
{
    LPVOID vp;
    if ((vp = GlobalLock(hGlobal)) != NULL) {
        LPCITEMIDLIST pidl = _7FD0276A(vp, var);
        GlobalUnlock(hGlobal);
        return pidl;
    }
    return NULL;
}

BOOL GetTargetInfo(HGLOBAL hGlobal, int var, char FilePath[MAX_PATH], WIN32_FIND_DATA* lpFindData)
{
    LPCITEMIDLIST pidl;
    PathName[0] = '\0';
    if ((pidl = GetPIDL(hGlobal, var)) != NULL) {
        if (SHGetPathFromIDList(pidl, FilePath)) {
            if (lpFindData) {
                if ((hFff = FindFirstFile(FilePath, lpFindData)) == INVALID_HANDLE_VALUE)
                    memset(lpFindData, 0, sizeof(WIN32_FIND_DATA));
                else
                    CloseHandle(hFff);
            }
            SHELL32_155_FreePidl(pidl);
            return TRUE;
        }
        else {
            SHELL32_155_FreePidl(pidl);
            return FALSE;
        }
    }
    return FALSE;
}

int CreatePropDialog(p1, pfn, IDataObject* pDataObj)
{
    FORMATETC formatetc;
    STGMEDIUM medium;
    PROPSHEETPAGE psp;
    HLOCAL hLocal;
    HGLOBAL hGlobal;
    WIN32_FIND_DATA ffd;
    HPROPSHEETPAGE hPsp;
    char PathName[MAX_PATH];
    BOOL var_4;
    int var_8;
    formatetc.lindex = -1;
    formatetc.cfFormat = gcfFormat;
    formatetc.ptd = 0;
    formatetc.dwAspect = 1;
    formatetc.tymed = TYMED_HGLOBAL;
    var_4 = FALSE;
    var_8 = 0;
    if (FAILED(pDataObj->GetData(&formatetc, &medium)))
        return FALSE;
    PathName[0] = 0;
    psp.hInstance = ghModule;
    psp.dwSize = 952; //NOT sizeof(psp); !!!
    psp.dwFlags = PSP_USECALLBACK; //0x80;
    psp.pfnCallback = PropCallback;
    var_2a0 = 1;
    var_3a8 = 0;
}
```



```
var_34 = 0;  
var_30 = 0;  
hGlobal = medium.hGlobal;  

if (hGlobal) {  
    hLocal = LocalAlloc(LPTR, GlobalSize(hGlobal));  
    if (hLocal)  
        memmove(hLocal, hGlobal, GlobalSize(hGlobal));  
} else  
    hLocal = NULL;  

if (GetNumberOfSelected(hGlobal) == 1) { // else 7fd05dd5  
    if (GetTargetInfo(hGlobal, 0, &PathName, &ffd)) { // else 7fd05d9f  
        psp.pfnDlgProc = ConventionalDlgProc;  
        psp.pResource = MAKEINTRESOURCE(ffd.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY ? 1044 : 1041);  
        if ((hPsp = CreatePropertySheetPage(&psp)) != NULL) { // else loc_7FD05D9F  
            if (pfn(hPsp, p1)) { // else loc_7FD05E07  
                var_4 = 1;  
                if (CreateShortcutPage(hGlobal, &PathName, pfn, p1))  
                    var_8 = 2;  
                CreateVersionPage(&PathName, pfn, p1);  
            } else // loc_7FD05E07  
                DestroyPropertySheetPage(hPsp);  
        }  
    } else { // loc_7FD05DD5  
        psp.pResource = MAKEINTRESOURCE(1043);  
        psp.pfnDlgProc = MultiSelDlgProc;  
        if ((hPsp = CreatePropertySheetPage(&psp)) != NULL) { // else loc_7FD05D9F  
            if (pfn(hPsp, p1))  
                var_4 = 1;  
            else  
                DestroyPropertySheetPage(hPsp);  
        }  
    }  
    // loc_7FD05D9F  
    _7FCC6204(&medium);  
    if (!var_4 && hLocal)  
        LocalFree(hLocal);  
    return var_8;  
}  

BOOL IsLnkFile(char* PathName)  
{  
    if (PathName)  
        return lstrcmpi(PathFindExtension(PathName), ".lnk") == 0;  
    return FALSE;  
}  

HRESULT CreateShellInstance(IUnknown* pUnknownOuter, const IID& iid, void** ppv)  
{  
    CShellFactory* pCShellFactory;  
    HRESULT hr;  
    if (pUnknownOuter)  
        return CLASS_E_NOAGGREGATION;  
    pCShellFactory = new CShellFactory;  
    if (pCShellFactory == NULL)  
        return E_OUTOFMEMORY;  
    pCShellFactory->m_cRef = 1;  
    pCShellFactory->Init();  
    hr = pCShellFactory->CreateInstance(iid, ppv);  
    pCShellFactory->Release();  
    return hr;  
}  

typedef struct FindTargetData  
{  
    HGLOBAL hGlobal; // 0  
    IShellLinkA* pIShellLink; // 4  
    int hDlgWnd; // 8  
    int dwc; // c  
    int dw10; // 10  
    int dw14; // 14  
    int dw18; // 18  
    char Path[MAX_PATH]; // 1c  
};  

BOOL CreateShortcutPage(HGLOBAL hGlobal, char PathName[MAX_MATH], pfn, p1)  
{  
    PROPSHEETPAGE psp;  
    IShellLink* pIShellLink;  
    FindTargetData* pFtd;  
    HPROPSHEETPAGE hPsp;  
    if (IsLinkFile(PathName) &&  
        SUCCEEDED(CreateShellInstance(NULL, IID_IShellLink, &pIShellLink)))  
    { // else loc_7FD040F6  
        pFtd = (FindTargetData*) LocalAlloc(LPTR, sizeof(FindTargetData));  
        if (pFtd != NULL) { // else loc_7FD040ED  
            psp.hInstance = ghModule;  
            psp.dwSize = sizeof(PROPSHEETPAGE);  
            psp.dwFlags = PSP_USECALLBACK;  
            psp.pResource = MAKEINTRESOURCE(1040);  
            psp.pfnDlgProc = ShortcutDlgProc;  
            psp.pfnCallback = CallbackProc;  
            psp.lParam = pFtd;  
            lstrcpyn(pFtd->Path, PathName, MAX_PATH);  
            pFtd->dw14 = -1;  
            pFtd->hGlobal = hGlobal;  
            pFtd->pIShellLink = pIShellLink;  
            if ((hPsp = CreatePropertySheetPage(&psp)) != NULL) {  
                if (pfn(hPsp, p1))  
                    return TRUE;  
                DestroyPropertySheetPage(hPsp);  
            }  
            LocalFree(pFtd);  
        }  
        pObj->Release();  
    }  
    return FALSE;  
}  

#define IDC_  
#define IDC_Name 0x66 // 名称  
#define IDC_StartLocation 0x3002 // 起始位置  
#define IDC_Target 0x3302 // 目标  
#define IDC_TargetType 0x3303 // 目标类型  
#define IDC_TargetLocation 0x3309 // 目标位置  
#define IDC_Shortcut 0x3404 // 快捷键  
#define IDC_FindTarget 0x3406 // 查找目标  
#define IDC_ChangeIcon 0x3407 // 更改图标  
#define IDC_RunMode 0x3408 // 运行方式  

BOOL CALLBACK ShortcutDlgProc(HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)  
{  
    LPPROPSHEETPAGE psp;  
    FindTargetData* pFtd;  
    LPNMHDR pnmh;  
    LPHELPINFO lphi;  
    pFtd = GetWindowLong(hWnd, DWL_USER);  
    switch (uMsg)  
    {  
    case WM_INITDIALOG:  
        psp = (LPPROPSHEETPAGE)lParam;  
        pFtd = psp->lParam;  
        SetWindowLong(hWnd, DWL_USER, pFtd);  
        pFtd->hDlgWnd = hWnd;  
        SendDlgItemMessage(hWnd, 0x3302/*ID for 目标*/, WM_LIMITTEXT, MAX_PATH - 1, 0);  
        COMCTL32_384(GetDlgItem(hWnd, 0x3302), 1);  
        SendDlgItemMessage(hWnd, 0x3002/*ID for 起始位置*/, EM_LIMITTEXT, MAX_PATH - 1, 0);  
        COMCTL32_384(GetDlgItem(hWnd, 0x3002), 1);  
        SendDlgItemMessage(hWnd, 0x3404/*ID for 快捷键*/, 0x403, 0xf, 6);  
        FindTargetInit(pFtd, 0);  
        return TRUE;  

    case WM_NOTIFY:  
        // If the message handler is in a dialog box procedure, you must use  
        // the SetWindowLong function with DWL_MSGRESULT to set a return value.  
        pnmh = (LPNMHDR) lParam;  
        if (pnmh->code != -202) {  
            if (pnmh->code == -201)  
                SetWindowLong(hwnd, DWL_MSGRESULT, !_7FD03BDA(pFtd));  
        }  
        else if (FAILED(_7FD03A38(pFtd)))  
            SetWindowLong(hWnd, DWL_MSGRESULT, 2);  
        return TRUE;  

    case WM_HELP:  
        lphi = (LPHELPINFO) lParam;  
        // HELP_WM_HELP:  
        // Displays the topic for the control identified  
        // by the hWndMain parameter in a pop-up window.  
        // HelpData:  
        // Address of an array of double word pairs.  
        // The first double word in each pair is a control identifier,  
        // and the second is a context identifier for a topic.  
        // The array must be terminated by a pair of zeros {0,0}.  
        // If you do not want to add Help to a particular control,  
        // set its context identifier to -1.  
        WinHelp(lphi->hItemHandle, NULL, HELP_WM_HELP, &HelpData);  
        break;  

    case WM_CONTEXTMENU:  
        WinHelp(lParam, NULL, HELP_CONTEXTMENU, &HelpData);  
        break;  

    case WM_COMMAND:  
        switch (LOWORD(wParam))  
        {  
        case 3002: // 起始位置  
        case 3302: // 目标  
        case 3404: // 快捷键  
            if (HIWORD(wParam) == 0x300) {  
                SendMessage(GetParent(hWnd), 0x468, hWnd, 0);  
                pFtd->dw10 = 1;  
            }  
            return TRUE;  

        case 3406: // 查找目标  
            OnCmdFindTarget(pFtd);  
            return TRUE;  

        case 3407:  // 更改图标  
            if (OnCmdChangeIcon(pFtd))  
                pFtd->dw10 = TURE;  
            return TRUE;  

        case 3408: // 运行方式  
            if (HIWORD(wParam) == 1)  
                SendMessage(GetParent(hWnd), 0x468, hWnd, 0);  
            return TRUE;  
        }  
        return FALSE;  

    case 0x464:  
        SwitchToThisWindow(GetLastActivePopup(lParam), TRUE);  
        SetForegroundWindow(uMsg); // 奇怪!  
        break;  
    }  
    return FALSE;  
}  

### 2.源代码（2）  

```
int gdwShowCmds[] = {1, 7, 3};  

void FindTargetInit(FindTargetData* pFtd, BOOL bFlag)  
{  
    HRESULT hr;  
    OLECHAR wPath[MAX_PATH];  
    char Path[MAX_PATH];  
    char TargetPath[MAX_PATH];  
    char Msg[MAX_PATH];  
    SHFILEINFO ShellFileInfo;  
    SHDWORD ShDw;  
    WORD Hotkey;  
    int ShowCmd;  
    ITEMIDLIST* pidl;  
    IPersistFile* pIPersistFile;  
    if (!bFlag &&  
        SUCCEEDED(pFtd->pIShellLink->QueryInterface(IID_IPersistFile, &pIPersistFile))  
    ) { // else loc_7FD032B3  
        SHELL32_163(wPath, &pFtd->dw18);  
        hr = pIPersistFile->Load(wPath, STGM_DIRECT);  
        pIPersistFile->Release();  
        if (FAILED(hr)) {  
            LoadString(ghModule, 4216/*ID for 不是有效的快捷方式。*/, &Msg, MAX_PATH);  
            SetDlgItemText(pFtd->hDlgWnd, 0x3303/*ID for "目标类型"*/, Msg);  
            _7FD031BF(pFtd->hDlgWnd);  
            return;  
        }  
    }  
    //loc_7FD032B3  
    if (pFtd->pIShellLink->b7d & FILE_ATTRIBUTE_DIRECTORY) {  
        GetTargetIcon(pFtd, 0);  
        GetTargetName(pFtd->hGlobal, 0, &Msg);  
        SetDlgItemText(pFtd->hDlgWnd, 0x66/*ID for 图标旁边的名称*/, &Msg);  
        EnableWindow(GetDlgItem(pFtd->hDlgWnd, 0x3012/*ID for*/), FALSE);  
        EnableWindow(GetDlgItem(pFtd->hDlgWnd, 0x3302/*ID for*/), FALSE);  
        EnableWindow(GetDlgItem(pFtd->hDlgWnd, 0x3002/*ID for*/), FALSE);  
        EnableWindow(GetDlgItem(pFtd->hDlgWnd, 0x3404/*ID for*/), FALSE);  
        EnableWindow(GetDlgItem(pFtd->hDlgWnd, 0x3408/*ID for*/), FALSE);  
        EnableWindow(GetDlgItem(pFtd->hDlgWnd, 0x3406/*ID for*/), FALSE);  
        EnableWindow(GetDlgItem(pFtd->hDlgWnd, 0x3407/*ID for*/), FALSE);  
        return;  
    } // goto loc_Return  
    GetTargetName(pFtd->hGlobal, 0, &Msg);  
    SetDlgItemText(pFtd->hDlgWnd, 0x66/*ID for 图标旁边的名称*/, &Msg);  
    if (SUCCEEDED(hr = pFtd->pIShellLink->GetPath(&TargetPath, MAX_PATH, NULL, SLGP_RAWPATH)) || // else loc_7FD033C0  
        SUCCEEDED(hr = pFtd->pIShellLink->GetPath(&TargetPath, MAX_PATH, NULL, 0)) && hr != S_FALSE)  
    {// else loc_7FD035B7  
        pFtd->dwc = TRUE;  
```



```c
if (!SHGetFileInfo(TargetPath, 0, &ShellFileInfo, sizeof(SHFILEINFO), SHGFI_TYPENAME)) { // loc_7FD03424  

ExpandEnvironmentStrings(TargetPath, Path, MAX_PATH);  

SHGetFileInfo(Path, 0, ShellFileInfo, sizeof(SHFILEINFO), SHGFI_TYPENAME);  

}  

SetDlgItemText(pFtd->hDlgWnd, 0x3303/*ID for "目标类型"*/, ShellFileInfo.szTypeName);  

lstrcpy(Msg, TargetPath);  

PathRemoveFileSpec(Msg);  

SetDlgItemText(pFtd->hDlgWnd, 0x3309/*ID for "目标位置"*/, PathFindFileName(Msg));  

pFtd->pIShellLink->GetArguments(Msg, MAX_PATH);  

_7FD030E9(TargetPath, Msg);  

GetDlgItemText(pFtd->hDlgWnd, 0x3302/*ID for "目标"*/, Msg, MAX_PATH);  

if (lstrcmp(TargetPath, Msg))  

// loc_7FD034C0  

SetDlgItemText(pFtd->hDlgWnd, 0x3302/*ID for "目标"*/, TargetPath);  

}  

else { // loc_7FD035B7  

pFtd->dwc = FALSE;  

EnableWindow(GetDlgItem(pFtd->hDlgWnd, 0x3302/*ID for*/), FALSE);  

EnableWindow(GetDlgItem(pFtd->hDlgWnd, 0x3002/*ID for*/), FALSE);  

pFtd->pIShellLink->GetIDList(&pidl);  

if (pidl) { // else loc_7FD034D1  

SHELL32_15(pidl, &TargetPath);  

SHELL32_17(pidl);  

SHELL32_186(0, pidl, &Msg, 1);  

SHELL32_155_FreePidl(pidl);  

SetDlgItemText(pFtd->hDlgWnd, 0x3309, Msg);  

SetDlgItemText(pFtd->hDlgWnd, 0x3303, TargetPath);  

// optimized as jmp loc_7FD034C0  

SetDlgItemText(pFtd->hDlgWnd, 0x3302/*ID for "目标"*/, TargetPath);  

}  

}  

// loc_7FD034D1  

if (!bFlag) { // else loc_Return  

pFtd->pIShellLink->GetWorkingDirectory(Msg, MAX_PATH);  

_7FD03173(pFtd->hDlgWnd, 0x3002, Msg);  

pFtd->pIShellLink->GetHotkey(&Hotkey);  

SendDlgItemMessage(pFtd->hDlgWnd, 0x3404/*ID for 快捷键*/, 0x401, (WPARAM)Hotkey, 0);  

for (uID = 0x1034; /*ID for 常规窗口*/ uID <= 0x1036; uID++) {  

LoadString(ghModule, uID, Msg, MAX_PATH);  

SendDlgItemMessage(pFtd->hDlgWnd, 0x3408, CB_ADDSTRING, 0, (LPARAM)&Msg);  

uID++;  

}  

pFtd->pIShellLink->GetShowCmd(&ShowCmd);  

for (int i = 0; i < 3; i++)  

if (gdwShowCmd[i] == ShowCmd) break;  

if (i == 3) i = 0;  

SendDlgItemMessage(pFtd->hDlgWnd, 0x3408, CB_SETCURSEL, 0, 0);  

GetTargetIcon(pFtd, 0);  

}  

}  

HRESULT GetSHGetIDispatchForFolderFunc(ITEMIDLIST* pidl, IWebBrowserApp* ppIWebBrowserApp)  

{  

if (ghSHDOCVW == NULL)  

ghSHDOCVW = LoadLibrary("SHDOCVW.DLL");  

if (ghSHDOCVW == NULL)  

return E_FAIL;  

if (gpfSHGetIDispatchForFolder == NULL) {  

gpfSHGetIDispatchForFolder = GetProcAddress(ghModule, "SHGetIDispatchForFolder");  

if (gpfSHGetIDispatchForFolder == NULL)  

return E_FAIL;  

}  

return gpfSHGetIDispatchForFolder(pidl, ppIWebBrowserApp);  

}  

HRESULT GetShellFolderViewDual(ITEMIDLIST* pidl, IShellFolderViewDual** ppIShellFolderViewDual)  

{  

IWebBrowserApp* pIWebBrowserApp;  

IDispatch* pDoc;  

HWND hWnd;  

HRESULT hr;  

*ppIShellFolderViewDual = NULL;  

if (SUCCEEDED(hr = GetSHGetIDispatchForFolderFunc(pidl, &pIWebBrowserApp)) {  

if (SUCCEEDED(pIWebBrowserApp->get_HWN(&hWnd)) {  

SetForegroundWindow(hWnd);  

ShowWindow(hWnd, SW_SHOWNORMAL);  

}  

if (SUCCEEDED(hr = pIWebBrowserApp->get_Document(&pDoc)) {  

pDoc->QueryInterface(IID_IShellFolderViewDual, ppIShellFolderViewDual);  

pDoc->Release();  

}  

pIWebBrowserApp->Release();  

}  

return hr;  

}  

ITEMIDLIST* SHELL32_16_GetFilePidl(ITEMIDLIST* pidl)  

{  

ITEMIDLIST* pIdlFile = pidl;  

    USHORT cb;  

if (pIdlFile) {  

while (cb = pidl->mkid.cb) {  

pIdlFile = pidl;  

pidl = (ITEMIDLIST*)((BYTE*)pidl + cb);  

}  

}  

return pIdlFile;  

}  

int SHELL32_152_GetPidlSize(ITEMIDLIST* pidl)  

{  

int sz = 0;  

    USHORT cb;  

if (pidl) {  

sz = 2;  

while (cb = pidl->mkid.cb) {  

sz += cb;  

pidl = (ITEMIDLIST*)((BYTE*)pidl + cb);  

}  

}  

return sz;  

}  

SAFEARRAY* CreateSaveArray(VARTYPE vt, long lLbound, unsigned int cElements)  

{  

if (ghOleAut32 == NULL)  

LoadLibrary("OLEAUT32.DLL");  

if (ghOleAut32) {  

if (pfnSafeArrayCreateVector == NULL)  

pfnSafeArrayCreateVector = GetProcAddress(ghOleAut32, "SafeArrayCreateVector");  

if (pfnSafeArrayCreateVector != NULL)  

return pfnSafeArrayCreateVector(vt, lLbound, cElements);  

}  

return NULL;  

}  

SAFEARRAY* AllocSafeArrayForPidl(ITEMIDLIST* pidl, int sz)  

{  

SAFEARRAY* parray;  

if (pidl && sz) {  

parray = CreateSaveArray(VT_UI1, 0, sz);  

if (parray)  

memcpy(parray->pvData, pidl, sz);  

return parray;  

}  

return NULL;  

}  

BOOL SetVariantArg(VARIANTARG* pVariantArg, ITEMIDLIST* pidl)  

{  

SAFEARRAY* parray;  

if ((parray = AllocSafeArrayForPidl(pidl, SHELL32_153_GetPidlSize(pidl))) != NULL) {  

memset(pVariantArg, sizeof(VARIANTARG));  

pVariantArg->vt = VT_ARRAY | VT_UI1);  

pVariantArg->parray = parray;  

return TRUE;  

}  

return FALSE;  

}  

HRESULT ClearVariantArg(VARIANTARG* pVariantArg)  

{  

if (ghOleAut32 == NULL)  

LoadLibrary("OLEAUT32.DLL");  

if (ghOleAut32) {  

if (pfnVariantClear == NULL)  

pfnVariantClear = GetProcAddress(ghOleAut32, "VariantClear");  

if (pfnVariantClear != NULL)  

return pfnVariantClear(pVariantArg);  

}  

return E_FAIL;  

}  

HRESULT SelectItem(IShellFolderViewDual* pIShellFolderViewDual, ITEMIDLIST* pidl, int dwFlags)  

{  

HRESULT hr = E_FAIL;  

VARIANTARG VariantArg;  

if (SetVariantArg(&VariantArg, pidl)) {  

hr = pIShellFolderViewDual->SelectItem(&VariantArg, dwFlags);  

ClearVariantArg(&VariantArg);  

}  

return hr;  

}  

void OnCmdFindTarget(FindTargetData* pFtd)  

{  

ITEMIDLIST* pidl, *pidl2;  

USHORT cb;  

IShellFolderViewDual* pIShellFolderViewDual;  

int var_8;  

if (pFtd->pIShellLink->Resolve(pFtd->hDlgWnd, 0) == NOERROR) { // 0? 奇怪!  

FindTargetInit(pFtd, TRUE);  

pFtd->pIShellLink->GetIDList(&pidl);  

if (pidl == NULL) return;  

var_10 = 0x1000000;  

if (FAILED(_7FCD197A(pidl, 0, 0, 0, &var_10)) {  

Msg(ghModule, pFtd->hDlgWnd, 6456/*ID for "找不到 %1!ls!"*/, NULL, 0x10030);  

SHELL32_155_FreePidl(pidl);  

return;  

}  

// Get the pidl for the item in its folder  

pidl2 = SHELL32_16_GetFilePidl(pidl);  

if (pidl2 != pidl)  

cb = pidl2->mkid.cb, pidl2->mkid.cb = 0;  

else  

cb = 0;  

EnterCriticalSec();  

if ((tmp = _7FCB255B(0, 0x10, 0)) != 0) {  

tmp = SHELL32_21(pidl, tmp);  

var_8 = 1;  

if (!tmp) var_8 = 0;  

}  

else  

var_8 = 0;  

LeaveCriticalSec();  

if (var_8 == 0 && cb != 0) { // else loc_7FD03761  

DoOleInitialize();  

if (SUCCEEDED(GetShellFolderViewDual(pidl, &pIShellFolderViewDual)) {  

pidl2->mkid.cb = cb;  

// 0 Deselect the item.   

// 1 Select the item.   

// 3 Put the item in edit mode.   

// 4 Deselect all but the specified item.   

// 8 Ensure the item is displayed in the view.   

// 0x10 Give the item the focus.   

SelectItem(pIShellFolderViewDual, pidl2, 0x1d);  

pIShellFolderViewDual->Release();  

}  

}  

else { // loc_7FD03761, target is on the desktop  

HWND hwnd;  

// 6457: 文件位于桌面上。要找到它，请用鼠标右键单击  

//       任务栏的空白区域，然后单击“最小化所有窗口  

Msg(ghModule, pFtd->hDlgWnd, 6457, NULL, 0);  

if ((hwnd = FindWindow("Progman", NULL)) != NULL)  

PostMessage(pFtd->hDlgWnd, 0x464, 0, hwnd);  

}  

SHELL32_155_FreePidl(pidl);  

return;  

}  

}  
```

`4dd466d0e1e0cc123afbe504da2462ca`

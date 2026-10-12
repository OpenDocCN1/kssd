
# Windows XP /2003 /VISTA / WIN7 所有版本全补丁存在的一个 BUG

> 原文：[`www.pediy.com/kssd/pediy10/84496.html`](https://www.pediy.com/kssd/pediy10/84496.html)

打开注册表编辑器，选择任意一个键，选择导入，文件类型为所有文件，导入一个非 `REG` 格式的文件。

此时会提示这个文件不是一个支持的格式。

接着，注册表编辑的导出文本文件功能将出问题，选择任意键，选择导出，文件类型选择“文本文件”，此时本来是导出操作，但是注册表编辑器却会提示无法导入，这个文件不是一个支持的格式。但是实际上这个文件却导出成功了~

## BUG 分析

`RegEdit` 中的导入导出等功能使用 `g_FileErrorStringID` 来标志错误号，实际会用 `LoadStringW` 来根据 `StringID` 来显示指定的错误号，在函数 `RegEdit_ExportRegFile` 中，根据用户选择不同的保存格式，会分别调用 `ExportRegedit32File`、`Export_SaveAsSubTree`、`ExportWin40RegFile`、`ExportWinNT50RegFile`。

各个函数进入前应将 `g_FileErrorStringID` 设为 `0X88`，如果有出错，再设置 `g_FileErrorStringID` 为错误文字的 `StringId`，在这些函数调用完成后，如果判断 `g_FileErrorStringID` 为有效的错误 ID，则弹框提示。

代码如下：

```c
int __stdcall RegEdit_ExportRegFile(int a1, int a2, char lpFileName, unsigned __int16 *lpsz)
{
    char v4;
    const void *v5;
    const WCHAR *v6;
    int v7;
    char v8;
    int result;

    v4 = lpFileName;

    if (!lpFileName)
    {
        v8 = lpFileName;
        v7 = 16;
        v6 = (const WCHAR *)16;
        v5 = (const void *)43;
        return InternalMessageBox(g_hInstance, a1, v5, v6, v7, v8);
    }

    switch (RegSaveType)
    {
        case 2:
            ExportRegedit32File((LPCWSTR)lpFileName, lpsz);
            break;
        case 3:
            RegEdit_SaveAsSubtree((LPCWSTR)lpFileName, (int)lpsz);
            break;
        case 4:
            ExportWin40RegFile((LPCWSTR)lpFileName, lpsz);
            break;
        default:
            ExportWinNT50RegFile((LPCWSTR)lpFileName, lpsz);
            break;
    }

    result = g_FileErrorStringID;

    if (g_FileErrorStringID != 0x88 && g_FileErrorStringID != 0xE4)
    {
        v8 = v4;
        v7 = 16;
        v6 = (const WCHAR *)16;
        v5 = (const void *)(unsigned __int16)g_FileErrorStringID;
        return InternalMessageBox(g_hInstance, a1, v5, v6, v7, v8);
    }

    return result;
}
```

但是 `Export_SaveAsSubTree`（也就是保存为文本文件的这个子函数中）没有预先设置 `g_FileErrorStringID`。这样，上次导入/导出出错后，这个函数无论成功与否都会使用上次导入失败的 `StringID`，最终导致了显示了错误的提示信息。

代码如下：

```c
int __stdcall RegEdit_SaveAsSubtree(LPCWSTR NumberOfBytesWritten, int a2)
{
    HLOCAL v2;
    HANDLE v3;
    void *v4;

    v2 = LocalAlloc(0x40u, 0x4000u);
    SavedSubTreeTextContext = v2;

    if (v2)
    {
        *(_WORD *)v2 = 0xFFFFFEFFu;
        dword_101CB30 = 0x2000u;
        dword_101CB34 = 1;
        s_PrintIo = 1;
        dword_101CB04 = 0;
        dword_101CB38 = (int)L"\r\n";
        RegPrintSubtree();

        v3 = CreateFileW(NumberOfBytesWritten, 0x40000000u, 0, 0, 2u, 0x80u, 0);
        v4 = v3;

        if (v3 == (HANDLE)-1)
        {
            dword_101CB04 = 138;
        }
        else
        {
            NumberOfBytesWritten = 0;
            if (!WriteFile(v3, SavedSubTreeTextContext, 2 * dword_101CB34, (DWORD *)&NumberOfBytesWritten, 0))
                dword_101CB04 = 141;
            CloseHandle(v4);
        }

        LocalFree(SavedSubTreeTextContext);
    }

    return PrintToSubTreeError(dword_101CB04);
}
```

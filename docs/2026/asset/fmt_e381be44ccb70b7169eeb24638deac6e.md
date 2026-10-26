
## 通过窗口图像获取位图信息

```
Cbm::Cbm(HWND hwndScreen)
{
    HDC     hdc, hdcMem, hdcScreen;
    HBITMAP hBitmap;
    RECT    rect;

    if (!hwndScreen)
    {
        memset(&rect, 0, sizeof(RECT));
        rect.right   = GetSystemMetrics(SM_CXSCREEN);
        rect.bottom  = GetSystemMetrics(SM_CYSCREEN);
    } else
        GetClientRect(hwndScreen, &rect);    //获得截图窗口的范围大小

    hdc     = GetDC(NULL);
    hdcMem  = CreateCompatibleDC(hdc);
    hBitmap = CreateCompatibleBitmap(hdc, rect.right - rect.left, rect.bottom - rect.top);
    SelectObject(hdcMem, hBitmap);
    hdcScreen  = GetDC(hwndScreen);
    BitBlt(hdcMem, 0, 0, rect.right - rect.left, rect.bottom - rect.top, hdcScreen, 0, 0, SRCCOPY);
    DeleteDC(hdcMem);
    ReleaseDC(hwndScreen, hdcScreen);

    //初始化信息头bmi结构
    memset(&bmih, 0, sizeof(BITMAPINFOHEADER));
    bmih.biSize        = sizeof(BITMAPINFOHEADER);
    bmih.biWidth       = rect.right - rect.left;
    bmih.biHeight      = rect.bottom - rect.top;
    bmih.biBitCount    = 24;
    bmih.biCompression = BI_RGB;
    bmih.biPlanes      = 1;
    SetcxDib();
    SetcyDib();
    SetcBits();

    //获取pBits的值
    pBits = new BYTE [cBits * cyDib];
    GetDIBits(hdc, hBitmap, 0, cyDib, pBits, (LPBITMAPINFO)&bmih, DIB_RGB_COLORS);

    //初始化文件头bmfh
    bmfh.bfType      = 0x4D42;
    bmfh.bfSize      = sizeof(BITMAPFILEHEADER) + sizeof(BITMAPINFOHEADER) + cBits * cyDib;
    bmfh.bfReserved1 = 0;
    bmfh.bfReserved2 = 0;
    bmfh.bfOffBits   = sizeof(BITMAPFILEHEADER) + sizeof(BITMAPINFOHEADER);
    ReleaseDC(NULL, hdc);
    DeleteObject(hBitmap);
}
```

## 通过加载文件获取位图信息

```
Cbm::Cbm(PCTSTR FilePath)
{
    HANDLE hFile = CreateFile(FilePath, GENERIC_READ,
        FILE_SHARE_READ, NULL, OPEN_EXISTING, FILE_FLAG_SEQUENTIAL_SCAN, NULL);
    if (hFile == INVALID_HANDLE_VALUE)
    {
        pBits = NULL;
        TRACE(_T("read file failed. FileName: %s\n"), FilePath);
        return;
    }

    DWORD dwBytesRead;
    if (!(ReadFile(hFile, &bmfh, sizeof(BITMAPFILEHEADER), &dwBytesRead, NULL) &&
        ReadFile(hFile, &bmih, sizeof(BITMAPINFOHEADER), &dwBytesRead, NULL) &&
        bmfh.bfType == 0x4D42) )
    {
        pBits = NULL;
        TRACE(_T("read file failed. FileName: %s\n"), FilePath);
        CloseHandle(hFile);
        return;
    }

    SetcxDib();
    SetcyDib();
    SetcBits();
    pBits = new BYTE [cBits * cyDib];

    if (!ReadFile(hFile, pBits, cBits * cyDib, &dwBytesRead, NULL))
    {
        delete [] pBits;
        pBits = NULL;
        TRACE(_T("read file failed. FileName: %s\n"), FilePath);
    }
    CloseHandle(hFile);
}
```

## 保存位图到文件

```
BOOL Cbm::SaveBitmapToFile(PCTSTR FileName, LPCRECT pRect) const
{
    ASSERT(pBits);
    TCHAR FilePath[MAX_PATH], DefaultFileName[MAX_PATH];

    //创建以系统时间命名的bmp文件
    SYSTEMTIME time;
    GetLocalTime(&time);
    wsprintf(DefaultFileName, _T("%04u%02u%02u%02u%02u%02u%03u.bmp"),
        time.wYear, time.wMonth, time.wDay, time.wHour, time.wMinute, time.wSecond, time.wMilliseconds);

    //修正保存路径，默认保存至当前程序目录Screen文件夹
    if (!FileName)
        wsprintf(FilePath, _T("%s\\%s"), _T("screen"), DefaultFileName);
    else
    {
        if (FileName[1] == ':')
            _tcscpy_s(FilePath, FileName);
        else
            wsprintf(FilePath, _T("%s\\%s"), _T("screen"), FileName);
        if (FileName[lstrlen(FileName) - 1] == '\\' || FileName[lstrlen(FileName) - 1] == '/')
            _tcscat_s(FilePath, MAX_PATH, DefaultFileName);
    }

    // 判断文件路径是否有效，无效则创建路径中没有的文件夹
    if (!PathIsDirectory(FilePath))
        CreateFolder(FilePath);

    //保存数据
    HANDLE hFile = CreateFile(FilePath, GENERIC_WRITE, 0, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);
    if (hFile == INVALID_HANDLE_VALUE)
        return FALSE;

    DWORD dwBytesWritten;
    Cbm bmFile(*this, pRect);
    BOOL bSuccess = WriteFile(hFile, &bmFile.bmfh, sizeof(BITMAPFILEHEADER), &dwBytesWritten, NULL) &&
                    WriteFile(hFile, &bmFile.bmih, sizeof(BITMAPINFOHEADER), &dwBytesWritten, NULL) &&
                    WriteFile(hFile, bmFile.pBits, bmFile.cBits * bmFile.cyDib, &dwBytesWritten, NULL);

    CloseHandle(hFile);
    if (!bSuccess)
        DeleteFile(FilePath);
    return bSuccess;
}
```

## 找图函数

```
BOOL FindPic(const Cbm & bmWnd, const Cbm & bmFile, LPCRECT rectTarget, OUT PRECT retRect, int resemble, COLORREF rgb)
{
    if (!(bmFile.pBits && bmWnd.pBits) || bmFile.cxDib > bmWnd.cxDib || bmFile.cyDib > bmWnd.cyDib)
        return FALSE;

    resemble = max(resemble, 0);
    resemble = min(resemble, 100);

    BYTE r = GetRValue(rgb);
    BYTE g = GetGValue(rgb);
    BYTE b = GetBValue(rgb);

    // 实际范围
    RECT rectDefault;
    if (rectTarget && bmWnd.IsInRect(*rectTarget))
        rectDefault = *rectTarget;
    else
        bmWnd.GetBitmapRect(rectDefault);

    // bmFile图像坐标(x, y),  bmWnd图像坐标(x + xOffset, y + yOffset)
    int yTotal        =    rectDefault.bottom - bmFile.cyDib;
    int xTotal        =    rectDefault.right - bmFile.cxDib;
    int invalidTotal  =    (100 - resemble) * (bmFile.cxDib * bmFile.cyDib);
    int validTotal    =    resemble * (bmFile.cxDib * bmFile.cyDib);

    //  ignoreNum忽略值, validNum有效值，invalidNum无效值
    int invalidNum = 0, validNum = 0,  ignoreNum = 0;
    for (int yOffset = rectDefault.top; yOffset <= yTotal; yOffset++)
        for (int xOffset = rectDefault.left; xOffset <= xTotal; xOffset++)
        {
            for (int y = 0, bflag = TRUE; bflag && (y < bmFile.cyDib); y++)
                for (int x = 0; x < bmFile.cxDib; x++)
                {
                    int FileIndex = (bmFile.cyDib - 1 - y) * bmFile.cBits + 3 * x;
                    int WndIndex  = (bmWnd.cyDib - 1 - yOffset - y) * bmWnd.cBits + 3 * (xOffset + x);

                    if (r    == bmFile.pBits[FileIndex + 2] &&
                        g    == bmFile.pBits[FileIndex + 1] &&
                        b    == bmFile.pBits[FileIndex]     &&
                        0xF8 != bmWnd.pBits[WndIndex + 2]   &&
                        0xFC != bmWnd.pBits[WndIndex + 1]   &&
                        0xF8 != bmWnd.pBits[WndIndex]) {
                            ignoreNum++;
                    }
                    else if (bmFile.pBits[FileIndex + 2] == bmWnd.pBits[WndIndex + 2] &&
                        bmFile.pBits[FileIndex + 1] == bmWnd.pBits[WndIndex + 1] &&
                        bmFile.pBits[FileIndex] == bmWnd.pBits[WndIndex]) {
                            validNum++;
                    }
                    else
                        invalidNum++;

                    if (100 * invalidNum > invalidTotal)
                    {
                        invalidNum = validNum = ignoreNum = 0;
                        bflag = FALSE;
                        break;
                    }

                    if (100 * (validNum + ignoreNum) >= validTotal)
                    {
                        if (retRect)
                        {
                            retRect->left   = xOffset;
                            retRect->top    = yOffset;
                            retRect->right  = xOffset + bmFile.cxDib;
                            retRect->bottom = yOffset + bmFile.cyDib;
                        }
                        return TRUE;
                    }
                }
        }
    return FALSE;
}
```

## 多图查找函数

```
BOOL FindSomePic(const Cbm & bmWnd, PCTSTR FileName, LPCRECT rectTarget, PRECT retRect, PTSTR retFileName, int resemble, COLORREF rgb)
{
    WIN32_FIND_DATA fData;
    BOOL  bFind = FALSE;
    TCHAR FilePath[MAX_PATH];
    TCHAR FileDir[MAX_PATH];

    _tcscpy_s(FilePath, MAX_PATH, FileName);
    _tcscpy_s(FileDir, MAX_PATH, FileName);

    if (FileName[lstrlen(FileName) - 1] == '\\')
        _tcscat_s(FilePath, MAX_PATH, _T("*.bmp"));
    else if (_tcschr(FileName, '*'))
```

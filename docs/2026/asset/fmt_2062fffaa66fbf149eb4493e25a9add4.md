
```c
                                {

                                        l\_lpPeOverlayBuff = GetPeOverlay(l\_lpOverlayBuff);

                                        if ((int)l\_lpPeOverlayBuff - (int)l\_lpOverlayBuff == l\_uFileSize)

                                        {

                                                StringAdd(g\_hPrintfEdit,"错误!该程序没有尾部附加数据，是否继续添加数据\r\n");

                                                StringAdd(g\_hPrintfEdit,"如果继续添加，将会将整个exe文件添加进去。『Y/N』!!\r\n");

                                                if (MessageBox(0,"该程序没有尾部附加数据，是否继续添加数据\r\n如果继续添加，将会将整个exe文件添加进去。","添加提示",MB\_YESNO) != IDYES)

                                                {

                                                        StringAdd(g\_hPrintfEdit,"『N』用户中止\r\n");

                                                        UnmapViewOfFile(l\_lpOverlayBuff);

                                                        CloseHandle(l\_hOverlayMap);

                                                        CloseHandle(l\_hOverlayFile);

                                                        CloseHandle(l\_hPatchFile);

                                                        return FALSE;

                                                }else

                                                {

                                                        StringAdd(g\_hPrintfEdit,"『Y』用户继续...\r\n");

                                                        l\_lpPeOverlayBuff = l\_lpOverlayBuff;

                                                        l\_uOverSize = l\_uFileSize;

                                                }

                                        }else

                                                l\_uOverSize = l\_uFileSize - ((int)l\_lpPeOverlayBuff - (int)l\_lpOverlayBuff);

                                }else

                                {

                                        l\_lpPeOverlayBuff = l\_lpOverlayBuff;

                                        l\_uOverSize = l\_uFileSize;

                                }

                        }

                }else

                {

                        ZeroMemory(&l\_szMsgBuff,sizeof(l\_szMsgBuff));

                        wsprintf(l\_szMsgBuff,"错误!无法创建%s文件映像!!\r\n",lpszOverlayFileName);

                        StringAdd(g\_hPrintfEdit,l\_szMsgBuff);

                        CloseHandle(l\_hOverlayFile);

                        CloseHandle(l\_hPatchFile);

                        return FALSE;

                }

                ZeroMemory(&l\_szMsgBuff,sizeof(l\_szMsgBuff));

                wsprintf(l\_szMsgBuff,"%s文件验效完成!!\r\n\r\n",lpszOverlayFileName);

                StringAdd(g\_hPrintfEdit,l\_szMsgBuff);

                StringAdd(g\_hPrintfEdit,"正在写入尾部附加数据...\r\n");

                SetFilePointer(l\_hPatchFile,0,0,FILE\_END);

                if (MyWriteFile(l\_hPatchFile,l\_lpPeOverlayBuff,l\_uOverSize))

                        l\_bIsWrite = TRUE;

                UnmapViewOfFile(l\_lpOverlayBuff);

                CloseHandle(l\_hOverlayMap);

                CloseHandle(l\_hOverlayFile);

                CloseHandle(l\_hPatchFile);

        }

        return l\_bIsWrite;

}

LPOPENFILENAME InitOpenFileName(IN HINSTANCE hInstance,

                                                                IN HWND hOwner,

                                                                OUT LPOPENFILENAME lpstOFN,

                                                                OUT char \*lpszFileNameBuff,

                                                                IN int nFileNameBuffSize)

{

        if (lpstOFN)

        {

                ZeroMemory(lpstOFN,sizeof(OPENFILENAME));

                ZeroMemory(lpszFileNameBuff,nFileNameBuffSize);

                lpstOFN->lStructSize = sizeof(OPENFILENAME);

                lpstOFN->hInstance = hInstance;

                lpstOFN->hwndOwner = hOwner;

                lpstOFN->lpstrFile = lpszFileNameBuff;

                lpstOFN->nMaxFile = nFileNameBuffSize;

                lpstOFN->lpstrFilter = "all Files(\*.\*)\0\*.\*\0\0";

                lpstOFN->Flags = OFN\_FILEMUSTEXIST | OFN\_PATHMUSTEXIST;

        }

        return lpstOFN;

}

LRESULT CALLBACK AboutDlgProc(HWND hDlg, UINT uMsg, WPARAM wParam, LPARAM lParam)

{

        HWND l\_hStatic;

        switch(uMsg)

        {

        case WM\_INITDIALOG:

                l\_hStatic = GetDlgItem(hDlg,STATIC\_BBS\_PEDIY);

                SetWindowLong(l\_hStatic,GWL\_USERDATA,GetWindowLong(l\_hStatic,GWL\_WNDPROC));

                SetWindowLong(l\_hStatic,GWL\_WNDPROC,(UINT)StaticProc);

                l\_hStatic = GetDlgItem(hDlg,STATIC\_BBS\_DEBUGMAN);

                SetWindowLong(l\_hStatic,GWL\_USERDATA,GetWindowLong(l\_hStatic,GWL\_WNDPROC));

                SetWindowLong(l\_hStatic,GWL\_WNDPROC,(UINT)StaticProc);

                AnimateShow(hDlg);

                break;

        case WM\_LBUTTONDOWN:

                SendMessage(hDlg,WM\_CLOSE,0,0);

                break;

        case WM\_CTLCOLORSTATIC:

                if ((GetDlgCtrlID((HWND)lParam) == STATIC\_BBS\_PEDIY) || (GetDlgCtrlID((HWND)lParam) == STATIC\_BBS\_DEBUGMAN))

                        return (LRESULT)DrawStatic((HDC)wParam,(HWND)lParam);

                return (LRESULT)FALSE;

        case WM\_CLOSE:

                EndDialog(hDlg,FALSE);

                break;

        default:

                return (LRESULT)FALSE;

        }

        return TRUE;

}

int StringAdd(IN HWND hWnd,IN char \*lpszBuff)

{

        int l\_index;

        int l\_nSelStart,l\_uLineLen;

        if (hWnd && lpszBuff)

        {

                l\_index = SendMessage(hWnd,EM\_GETLINECOUNT,0,0);

                if (!(SendMessage(hWnd,EM\_LINELENGTH,SendMessage(hWnd,EM\_LINEINDEX,l\_index-1,0),0)))

                        l\_index--;

                if (l\_index)

                {

                        l\_nSelStart = SendMessage(hWnd,EM\_LINEINDEX,l\_index,0);

                        if (l\_nSelStart == -1)

                        {

                                l\_nSelStart = SendMessage(hWnd,EM\_LINEINDEX,l\_index-1,0);

                                if (l\_nSelStart == -1)

                                        return 0;

                                l\_uLineLen = SendMessage(hWnd,EM\_LINELENGTH,l\_nSelStart,0);

                                if (!l\_uLineLen)

                                        return 0;

                                l\_nSelStart += l\_uLineLen;

                        }

                        SendMessage(hWnd,EM\_SETSEL,l\_nSelStart,l\_nSelStart);

                        SendMessage(hWnd,EM\_REPLACESEL,0,(UINT)lpszBuff);

                }else

                        SendMessage(hWnd,EM\_REPLACESEL,0,(UINT)lpszBuff);

                return l\_nSelStart;

        }

        return 0;

}

HBRUSH DrawStatic(IN HDC hDc, IN HWND hStatic)

{

        HFONT l\_hFontOut;

        LOGFONT l\_stFont;

        l\_hFontOut = (HFONT)SendMessage(hStatic,WM\_GETFONT,0,0);

        GetObject(l\_hFontOut,sizeof(LOGFONT),&l\_stFont);

        l\_stFont.lfUnderline = FALSE;

        l\_hFontOut = CreateFontIndirect(&l\_stFont);

        SelectObject(hDc,l\_hFontOut);

        SetTextColor(hDc,0xff0000);

        SetBkColor(hDc,GetSysColor(COLOR\_BTNFACE));

        DeleteObject((HGDIOBJ)l\_hFontOut);

        return (HBRUSH)GetStockObject(NULL\_BRUSH);

}

BOOL IsPeFile(IN void \*lpFileModule)

{

        PIMAGE\_DOS\_HEADER l\_lpstDosHeader;

        PIMAGE\_NT\_HEADERS l\_lpstNtHeaders;

        l\_lpstDosHeader = (PIMAGE\_DOS\_HEADER)lpFileModule;

        if (l\_lpstDosHeader->e\_magic == IMAGE\_DOS\_SIGNATURE)

        {
```

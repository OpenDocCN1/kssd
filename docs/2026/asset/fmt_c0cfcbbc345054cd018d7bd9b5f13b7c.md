
```cpp
                EndDialog(hDlg,FALSE);

                break;

        default:

                return FALSE;

        }

        return TRUE;

}

LRESULT CALLBACK EditsProc(HWND hEdit,UINT uMsg, WPARAM wParam, LPARAM lParam)

{

        int l_uNumFiles = 0;

        char l_szFilesName[CHAR_MAX];

        int i;

        switch(uMsg)

        {

        case WM_DROPFILES:

                l_uNumFiles = DragQueryFile((HDROP)wParam,-1,l_szFilesName,sizeof(l_szFilesName));

                for (i =0; i < l_uNumFiles; i++)

                {

                        DragQueryFile((HDROP)wParam,i,l_szFilesName,sizeof(l_szFilesName));

                        SetWindowText(hEdit,l_szFilesName);

                }

                DragFinish((HDROP)wParam);

                break;

        default:

                return CallWindowProc((WNDPROC)GetWindowLong(hEdit,GWL_USERDATA),hEdit,uMsg,wParam,lParam);

        }

        return TRUE;

}

LRESULT CALLBACK StaticProc(HWND hStatic, UINT uMsg, WPARAM wParam, LPARAM lParam)

{

        switch(uMsg)

        {

        case WM_NCHITTEST:

                return (LRESULT)TRUE;

                break;

        case WM_SETCURSOR:

                SetCursor(g_hHandCur);

                break;

        case WM_LBUTTONUP:

                switch(GetDlgCtrlID(hStatic))

                {

                case STATIC_BBS_PEDIY:

                        ShellExecute(0,NULL,"http://bbs.pediy.com",NULL,NULL,0);

                        break;

                case STATIC_BBS_DEBUGMAN:

                        ShellExecute(0,NULL,"349K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8W2j5Y4g2Y4L8h3q4F1i4K6u0W2j5$3!0E0",NULL,NULL,0);

                        break;

                }

                break;

        default:

                return CallWindowProc((WNDPROC)GetWindowLong(hStatic,GWL_USERDATA),hStatic,uMsg,wParam,lParam);

        }

        return TRUE;

}

BOOL StartWriteOverlay(IN char *lpszPatchFileName, IN char *lpszOverlayFileName)

{

        HANDLE l_hPatchFile, l_hOverlayFile,l_hPatchMap,l_hOverlayMap;

        void *l_lpPatchBuff, *l_lpOverlayBuff,*l_lpPeOverlayBuff;

        char l_szMsgBuff[CHAR_MAX];

        int l_uFileSize,l_uOverSize;

        BOOL l_bIsWrite = FALSE;

        if (IsFiles(g_hPrintfEdit,lpszPatchFileName) && IsFiles(g_hPrintfEdit,lpszOverlayFileName))

        {

                StringAdd(g_hPrintfEdit,"\r\n开始添加EXE尾部附加数据......\r\n");

                ZeroMemory(&l_szMsgBuff,sizeof(l_szMsgBuff));

                wsprintf(l_szMsgBuff,"正在打开%s文件......\r\n",lpszPatchFileName);

                StringAdd(g_hPrintfEdit,l_szMsgBuff);

                l_hPatchFile = MyCreateFile(lpszPatchFileName,OPEN_EXISTING);

                if (l_hPatchFile == INVALID_HANDLE_VALUE)

                {

                        ZeroMemory(&l_szMsgBuff,sizeof(l_szMsgBuff));

                        wsprintf(l_szMsgBuff,"错误!无法打开%s文件!!\r\n",lpszPatchFileName);

                        StringAdd(g_hPrintfEdit,l_szMsgBuff);

                        return FALSE;

                }

                ZeroMemory(&l_szMsgBuff,sizeof(l_szMsgBuff));

                wsprintf(l_szMsgBuff,"正在创建%s文件映像......\r\n",lpszPatchFileName);

                StringAdd(g_hPrintfEdit,l_szMsgBuff);

                l_uFileSize = GetFileSize(l_hPatchFile,NULL);

                l_hPatchMap = MyCreateMapFile(l_hPatchFile);

                if (l_hPatchMap)

                {

                        l_lpPatchBuff = MyMapViewOfFile(l_hPatchMap);

                        if (l_lpPatchBuff)

                        {

                                if (!IsPeFile(l_lpPatchBuff))

                                {

                                        ZeroMemory(&l_szMsgBuff,sizeof(l_szMsgBuff));

                                        wsprintf(l_szMsgBuff,"错误!%s文件必须是PE文件!!\r\n",lpszPatchFileName);

                                        StringAdd(g_hPrintfEdit,l_szMsgBuff);

                                        UnmapViewOfFile(l_lpPatchBuff);

                                        CloseHandle(l_hPatchMap);

                                        CloseHandle(l_hPatchFile);

                                        return FALSE;

                                }

                                if ((int)GetPeOverlay(l_lpPatchBuff) - (int)l_lpPatchBuff != l_uFileSize)

                                {

                                        StringAdd(g_hPrintfEdit,"该程序已有尾部附加数据，是否继续添加数据『Y/N』!!\r\n");

                                        if (MessageBox(0,"该程序已有尾部附加数据，是否继续添加数据","添加提示",MB_YESNO) != IDYES)

                                        {

                                                StringAdd(g_hPrintfEdit,"『N』用户中止\r\n");

                                                UnmapViewOfFile(l_lpPatchBuff);

                                                CloseHandle(l_hPatchMap);

                                                CloseHandle(l_hPatchFile);

                                                return FALSE;

                                        }

                                        StringAdd(g_hPrintfEdit,"『Y』用户继续...\r\n");

                                }

                                UnmapViewOfFile(l_lpPatchBuff);

                        }

                }else

                {

                        ZeroMemory(&l_szMsgBuff,sizeof(l_szMsgBuff));

                        wsprintf(l_szMsgBuff,"错误!无法创建%s文件映像!!\r\n",lpszPatchFileName);

                        StringAdd(g_hPrintfEdit,l_szMsgBuff);

                        CloseHandle(l_hPatchFile);

                        return FALSE;

                }

                CloseHandle(l_hPatchMap);

                ZeroMemory(&l_szMsgBuff,sizeof(l_szMsgBuff));

                wsprintf(l_szMsgBuff,"%s文件验效完成!!\r\n\r\n",lpszPatchFileName);

                StringAdd(g_hPrintfEdit,l_szMsgBuff);

                ZeroMemory(&l_szMsgBuff,sizeof(l_szMsgBuff));

                wsprintf(l_szMsgBuff,"正在打开%s文件......\r\n",lpszOverlayFileName);

                StringAdd(g_hPrintfEdit,l_szMsgBuff);

                l_hOverlayFile = MyCreateFile(lpszOverlayFileName,OPEN_EXISTING);

                if (l_hOverlayFile == INVALID_HANDLE_VALUE)

                {

                        ZeroMemory(&l_szMsgBuff,sizeof(l_szMsgBuff));

                        wsprintf(l_szMsgBuff,"错误!无法打开%s文件!!\r\n",lpszOverlayFileName);

                        StringAdd(g_hPrintfEdit,l_szMsgBuff);

                        CloseHandle(l_hPatchFile);

                        return FALSE;

                }

                ZeroMemory(&l_szMsgBuff,sizeof(l_szMsgBuff));

                wsprintf(l_szMsgBuff,"正在创建%s文件映像......\r\n",lpszOverlayFileName);

                StringAdd(g_hPrintfEdit,l_szMsgBuff);

                l_uFileSize = GetFileSize(l_hOverlayFile,NULL);

                if (!l_uFileSize)

                {

                        ZeroMemory(&l_szMsgBuff,sizeof(l_szMsgBuff));

                        wsprintf(l_szMsgBuff,"错误!%s文件大小不能为零!!\r\n",lpszOverlayFileName);

                        StringAdd(g_hPrintfEdit,l_szMsgBuff);

                        CloseHandle(l_hOverlayFile);

                        CloseHandle(l_hPatchFile);

                        return FALSE;

                }

                l_hOverlayMap = MyCreateMapFile(l_hOverlayFile);

                if (l_hOverlayMap)

                {

                        l_lpOverlayBuff = MyMapViewOfFile(l_hOverlayMap);

                        if (l_lpOverlayBuff)

                        {

                                if (IsPeFile(l_lpOverlayBuff))
```

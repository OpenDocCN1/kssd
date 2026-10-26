
```
                        if (s_colorTuoGuan == GetColorAtPos(s_nTuoGuanX, s_nTuoGuanY))

                        {

                                KillTimer(TUOGUAN_TIMER_ID);

                                ClickAtPos(s_nTuoGuanX, s_nTuoGuanY);

                                TCHAR szTime[9] = {0};

                                _strtime(szTime);

                                ReportLog(_T("托管: %s\n"), szTime);

                                Sleep(REST_TIME_AFTER_SEND_COMMAND);

                        }

                }

                break;

                case LIANSHENG_TIMER_ID:

                        {

                                if (s_colorLianSheng == GetColorAtPos(s_nLianShengX, s_nLianShengY))

                                {

                                        KillTimer(LIANSHENG_TIMER_ID);

                                        ClickAtPos(s_nLianShengX, s_nLianShengY);

                                        TCHAR szTime[9] = {0};

                                        _strtime(szTime);

                                        ReportLog(_T("连剩五局: %s\n"), szTime);

                                        Sleep(REST_TIME_AFTER_SEND_COMMAND);

                                        SetTimer(LIANSHENG_TIMER_ID, TIMER_INTERVAL, NULL);

                                }

                        }

                        break;

                case TUO_GUAN_BUG_TIMER_ID:

                        {

                                // 托管时不出牌时的BUG

                                if (s_colorTuoGuanBug == GetColorAtPos(s_nTuoGuanBugX, s_nTuoGuanBugY))

                                {

                                        KillTimer(TUO_GUAN_BUG_TIMER_ID);

                                        KillTimer(TUOGUAN_TIMER_ID);                //  取消托管

                                        Beep(32767, 1000);

                                        TCHAR szTime[9] = {0};

                                        _strtime(szTime);

                                        ReportLog(_T("托管不出牌BUG: %s\n"), szTime);

                                        Sleep(REST_TIME_AFTER_SEND_COMMAND);

                                        ClickAtPos(s_nTuoGuanBugQuXiaoTuoGuanX, s_nTuoGuanBugQuXiaoTuoGuanY);        // 取消托管

                                        ReportLog(_T("取消托管\n"));

                                        Sleep(REST_TIME_AFTER_SEND_COMMAND);

                                        ClickAtPos(s_nTuoGuanBugClickX, s_nTuoGuanBugClickX);        // 选牌

                                        ReportLog(_T("选牌\n"));

                                        Sleep(REST_TIME_AFTER_SEND_COMMAND);

                                        ClickAtPos(s_nTuoGuanBugChuPaiX, s_nTuoGuanBugChuPaiY);                // 出牌

                                        ReportLog(_T("出牌\n"));

                                        Sleep(REST_TIME_AFTER_SEND_COMMAND);

                                        SetTimer(TUO_GUAN_BUG_TIMER_ID, TIMER_INTERVAL, NULL);

                                        SetTimer(TUOGUAN_TIMER_ID, TIMER_INTERVAL, NULL);                        // 恢复托管

                                }

                        }

                        break;

                case PREVIEW_GAEM_TIME_ID:

                        {

                                KillTimer(PREVIEW_GAEM_TIME_ID);

                                CRect rectClient;

                                GetClientRect(&rectClient);

                                CRect rectRedt;

                                m_redtLog.GetWindowRect(&rectRedt);

                                ScreenToClient(&rectRedt);

                                rectClient.top = rectRedt.bottom;

                                InvalidateRect(&rectClient);

                                SetTimer(PREVIEW_GAEM_TIME_ID, TIMER_INTERVAL, NULL);

                        }

                        break;

        }

        CDialogEx::OnTimer(nIDEvent);

}

// 指定位置单击

int CQQGameAddInDlg::ClickAtPos(int nX, int nY)

{

        ::SendMessage(s_hwndQQGame, WM_LBUTTONDOWN, MK_LBUTTON, (nY << 16) + nX);

        ::SendMessage(s_hwndQQGame, WM_LBUTTONUP, 0, (nY << 16) + nX);

        return 0;

        int nX1 = 0, nY1 = 0;                // 转化后的坐标

        int nScreenWidth = GetSystemMetrics(SM_CXSCREEN);

        int nScreenHeight = GetSystemMetrics(SM_CYSCREEN);

        nX1 = nX * (65535. / nScreenWidth );

        nY1 = nY * (65535. / nScreenHeight);

        mouse_event(MOUSEEVENTF_ABSOLUTE | MOUSEEVENTF_MOVE, nX1, nY1, 0, NULL);

        mouse_event(MOUSEEVENTF_LEFTDOWN, nX1, nY1, 0, NULL);

        mouse_event(MOUSEEVENTF_LEFTUP, nX1, nY1, 0, NULL);

        return 0;

}
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

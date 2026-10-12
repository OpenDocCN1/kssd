
```c
    }

    case WM_CTLCOLOREDIT:
        goto SetColor;

    case WM_CTLCOLOREDIT:
    case WM_CTLCOLORDLG:
    case WM_CTLCOLORMSGBOX:
         ......
    case WM_CTLCOLOREDIT:             // here for WOW only
    case WM_CTLCOLOREDIT:
    case WM_CTLCOLOREDIT:
        ......
    case WM_NCHITTEST:
        return FindNCHit(pwnd, (LONG)lParam);

    case WM_GETTEXT:
             ......
                 return cchSrc;
            
    case WM_GETTEXTLENGTH:
        ......
            return cch;
        }
        return 0L;

    case WM_QUERYDRAGICON:
       ......
        return (LRESULT)LoadIconW(pwnd->hModule, MAKEINTRESOURCE(1));

    case WM_QUERYOPEN:
    case WM_QUERYENDSESSION:
    case WM_DEVICECHANGE:
    case WM_POWERBROADCAST:
        return TRUE;

    case WM_KEYDOWN:
        if (wParam == VK_F10) {
            return CsSendMessage(hwnd, message, wParam, lParam, 0L,
                    FNID_DEFWINDOWPROC, fAnsi);
        }
        break;

    case WM_SYSKEYDOWN:
        if ((HIWORD(lParam) & SYS_ALTERNATE) || (wParam == VK_F10) ||
                (wParam == VK_ESCAPE))
            return CsSendMessage(hwnd, message, wParam, lParam, 0L,
                    FNID_DEFWINDOWPROC, fAnsi);
        break;

    case WM_CHARTOITEM:
    case WM_VKEYTOITEM:
        /*
         * Do default processing for keystrokes into owner draw listboxes.
         */
        return -1;

    case WM_ACTIVATE:
        if (LOWORD(wParam))
            return CsSendMessage(hwnd, message, wParam, lParam, 0L,
                    FNID_DEFWINDOWPROC, fAnsi);
        break;

    case WM_SHOWWINDOW:
        if (lParam != 0)
            return CsSendMessage(hwnd, message, wParam, lParam, 0L,
                    FNID_DEFWINDOWPROC, fAnsi);
        break;

    case WM_DROPOBJECT:
       return DO_DROPFILE;

    case WM_WINDOWPOSCHANGING:
        /*
         * If the window's size is changing, adjust the passed-in size
         */
        #define ppos ((WINDOWPOS *)lParam)
        if (!(ppos->flags & SWP_NOSIZE))
            return CsSendMessage(hwnd, message, wParam, lParam, 0L,
                    FNID_DEFWINDOWPROC, fAnsi);
        #undef ppos
        break;

    case WM_KLUDGEMINRECT:
         ......
        break;

       .........................
}
```

其他的公共控件见 `comdlg32` 等库。感兴趣可以自己研究一下，虽然 `2k` 代码有点老，不过参考价值还是很大的！

基本完毕了，没有明确的目标，纯属兴趣！

我是一只小菜鸟，多谢大家指教！！！

`4dd466d0e1e0cc123afbe504da2462ca`

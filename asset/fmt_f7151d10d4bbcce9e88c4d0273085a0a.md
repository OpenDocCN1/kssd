
# 几种获取 `EDIT` 文本的方法

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-756.htm`](https://www.pediy.com/kssd/pediy07/pediy7-756.htm)

1. `GetWindowText`

2. `SendMessage`

   ```c
   //刚刚发现，这个有问题。
   //无法成功。。。，不知为什么
   ```

3. `DispatchMessage`

   ```c
   //MSG msg;
   //msg.hwnd = hedit;
   //msg.message = WM_GETTEXT;
   //msg.wParam = (WPARAM)cchTextMax;
   //msg.lParam = (LPARAM)lpszText;
   //DispatchMessage(&msg);
   ```

4. `GetWindowLong`

   ```c
   LONG lWndProc;

   if (NULL != (lWndProc = GetWindowLong(hedit, GWL_WNDPROC)))
     CallWindowProc((WNDPROC)lWndProc, hedit, WM_GETTEXT, (WPARAM)cchTextMax, (LPARAM)lpszText);
   ```

6. `GetClassInfo`、`GetClassInfoEx` (两者有部分差异)

   ```c
   //GetClassInfo
   WNDCLASS wc;

   GetClassInfo(hInstance, "EDIT", &wc);

   CallWindowProc(wc.lpfnWndProc, hedit, WM_GETTEXT, (WPARAM)cchTextMax, (LPARAM)lpszText);

   //GetClassInfoEx 参看 GetClassInfo
   ```

6. 自己写 `EDIT` 窗口文本获取的处理过程

   可以从 `windows` 的源码中发现, 我在 `editec.c` 和 `edecrare.c` 中摘的，详细情况自己看吧 `^_^`

   ```c
   case WM_GETTEXT:
           /*
            * wParam - max number of _bytes_ (not characters) to copy
            * lParam - buffer to copy text to. Text is 0 terminated.
            */
           lreturn = (LONG)ECGetText(ped, wParam, (LPSTR)lParam, TRUE);
           break;

   //////////////////////////////////////////////////////////////////////////////
   /*************
   * ECGetText AorW
   *
   * Copies at most maxCchToCopy chars to the buffer lpBuffer. Returns
   * how many chars were actually copied. Null terminates the string based
   * on the fNullTerminate flag:
   * fNullTerminate --> at most (maxCchToCopy - 1) characters will be copied
   * !fNullTerminate --> at most (maxCchToCopy) characters will be copied
   *
   * History:
   ***********/

   ICH ECGetText(
       PED ped,
       ICH maxCchToCopy,
       LPSTR lpBuffer,
       BOOL fNullTerminate)
   {
       PSTR pText;
       if (maxCchToCopy) {
           /*
            * Zero terminator takes the extra byte
            */
           if (fNullTerminate)
               maxCchToCopy--;

           maxCchToCopy = min(maxCchToCopy, ped->cch);

           /*
            * Zero terminate the string
            */
           if (ped->fAnsi)
               *(LPSTR)(lpBuffer + maxCchToCopy) = 0;
           else
               *(((LPWSTR)lpBuffer) + maxCchToCopy) = 0;

           pText = ECLock(ped);

           RtlCopyMemory(lpBuffer, pText, maxCchToCopy * ped->cbChar);

           ECUnlock(ped);
       }
       return maxCchToCopy;
   }
   ```

4dd466d0e1e0cc123afbe504da2462ca

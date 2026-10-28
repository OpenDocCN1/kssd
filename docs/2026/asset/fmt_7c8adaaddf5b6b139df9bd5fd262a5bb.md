
When a mouse event occurs over the tray icon, your window receives `WM_SHELLNOTIFY` message which is the custom message you specified in `uCallbackMessage` member. Recall that on receiving this message, `wParam` contains the tray icon's ID and `lParam` contains the actual mouse message. In the code above, we check first if this message comes from the tray icon we are interested in. If it does, we check the actual mouse message. Since we are only interested in right mouse click and double-left-click, we process only `WM_RBUTTONDOWN` and `WM_LBUTTONDBLCLK` messages.

当一个鼠标事件发生在托盘图标上时，你的窗口将接收到`WM_SHELLNOTIFY`消息，这个消息是你放在`uCallbackMessage`成员变量中的自定义消息。在收到这个消息时，`wParam`参数包含托盘图标的ID号，`lParam`包含实际的鼠标消息。在上面的代码中，我们首先检查这个消息是否来自于我们感兴趣的托盘图标，如果它是，我们检查实际的鼠标消息。由于，我们仅仅对右键和双击事件感兴趣，所以我们仅处理`WM_RBUTTONDOWN`和`WM_LBUTTONDBLCLK`消息。

If the mouse message is `WM_RBUTTONDOWN`, we call `GetCursorPos` to obtain the current screen coordinate of the mouse cursor. When the function returns, the `POINT` structure is filled with the screen coordinate of the mouse cursor. By screen coordinate, I mean the coordinate of the entire screen without regarding to any window boundary. For example, if the screen resolution is 640*480, the right-lower corner of the screen is `x==639 and y==479`. If you want to convert the screen coordinate to window coordinate, use `ScreenToClient` function.

如果鼠标消息是`WM_RBUTTONDOWN`,我们调用`GetCursorPos`函数来得到鼠标光标在屏幕上的坐标。当函数返回时，`POINT`结构填充的是鼠标光标的屏幕坐标。屏幕坐标，我意思是指没有包含任何窗口边界的整个屏幕坐标。例如，如果屏幕分辨率是640*480，屏幕右下角的坐标是`x=639 y=479`，如果你想屏幕坐标为窗口坐标，用`ScreenToClient`函数。

However, for our purpose, we want to display the popup menu at the current mouse cursor position with `TrackPopupMenu` call and it requires screen coordinates, we can use the coordinates filled by `GetCursorPos` directly.

`TrackPopupMenu` has the following syntax:

然而，我们的目的是，我们想在当前鼠标光标的位置弹出一菜单，所以我们调用`TrackPopupMenu`函数，而这个函数需要的是屏幕坐标值，我们能将`GetCursorPos`函数返回的坐标值直接用在`TrackPopupMenu`函数中，句法如下：

```
TrackPopupMenu PROTO hMenu:DWORD, uFlags:DWORD,  x:DWORD,  y:DWORD, nReserved:DWORD, hWnd:DWORD, prcRect:DWORD
```

- `hMenu` is the handle of the popup menu to be displayed  
  `hMenu`是将要被显示的弹出菜单的句柄。
- `uFlags` specifies the options of the function. Like where to position the menu relative to the coordinates specified later and which mouse button will be used to track the menu. In our example, we use `TPM_RIGHTALIGN` to position the popup menu to the left of the coordinates.  
  `uFlags` 指定一个功能选项。像相对于随后指定的坐标值，菜单放的位置和那一个鼠标按钮将被用于菜单轨迹。在我们的例子中，我们用`TPM_RIGHTALIGN`来决定弹出式菜单放在坐标的左边。
- `x` and `y` specify the location of the menu in screen coordinates.  
  `x`和`y`指定在屏幕坐标中菜单的位置。
- `nReserved` must be `NULL`  
  `nReserved` 必须为空
- `hWnd` is the handle of the window that will receive the messages from the menu.  
  `hWnd` 是接受从菜单接收消息的窗口句柄。
- `prcRect` is the rectangle in the screen where it is possible to click without dismissing the menu. Normally we put `NULL` here so when the user clicks anywhere outside the popup menu, the menu is dismissed.  
  `prcRect` 是屏幕中的一块矩形。如果在该矩形区域外面按下鼠标的话，菜单将消失。一般我们把该值设为`NULL`，这样当用户只要在菜单外面按下鼠标，菜单立即消失。

When the user double-clicks at the tray icon, we send `WM_COMMAND` message to our own window specifying `IDM_RESTORE` to emulate the user selects Restore menu item in the popup menu thereby restoring the main window and removing the icon from the system tray. In order to be able to receive double click message, the main window must have `CS_DBLCLKS` style.

当用户在托盘图标上双击时，我们发送`WM_COMMAND`消息给我们的自己的窗口，并指定`IDM_RESTORE`来仿效用户在弹出菜单中选择了恢复菜单项，从而恢复主窗口并且从系统托盘中移出图标。为了能够接收到双击消息，主窗口必须有`CS_DBLCLKS`样式。

```
            invoke Shell_NotifyIcon,NIM_DELETE,addr note

            mov eax,wParam

            .if ax==IDM_RESTORE

                invoke ShowWindow,hWnd,SW_RESTORE

            .else

                invoke DestroyWindow,hWnd

            .endif
```

When the user selects Restore menu item, we remove the tray icon by calling `Shell_NotifyIcon` again, this time we specify `NIM_DELETE` as the message. Next, we restore the main window to its original state. If the user selects Exit menu item, we also remove the icon from the tray and destroy the main window by calling `DestroyWindow`.

当用户用选择恢复菜单项时，我们再一次调用`Shell_NotifyIcon`函数移出托盘图标，这个时候我们指定`NIM_DELETE`作为消息，下一步，我们恢复主窗口到原来的状态。如果用户选择退出菜单项，我们也从托盘中移出图标并且调用`DestroyWindow`来销毁窗口。

---

This article come from Iczelion's asm page

风向改变翻译于  2008-2-27

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

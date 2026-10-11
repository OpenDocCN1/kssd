
# 使用 Win32API 制作简单外挂

> 原文：[`www.pediy.com/kssd/pediy11/105521.html`](https://www.pediy.com/kssd/pediy11/105521.html)

使用 `Win32API` 制作简单外挂。

前提：安装 `Python` 和 `PyWin32`。

模拟鼠标点击：

```
def click(handle, pos):
    client_pos = win32gui.ScreenToClient(handle, pos)
    tmp = win32api.MAKELONG(client_pos[0], client_pos[1])
    win32gui.SendMessage(handle, win32con.WM_ACTIVATE, win32con.WA_ACTIVE, 0)
    win32api.SendMessage(handle, win32con.WM_LBUTTONDOWN, win32con.MK_LBUTTON, tmp)
    win32api.SendMessage(handle, win32con.WM_LBUTTONUP, win32con.MK_LBUTTON, tmp)
```

获得当前鼠标位置：

```
def get_curpos():
    return win32gui.GetCursorPos()
```

获得指定位置窗口：

```
def get_win_handle(pos):
    return win32gui.WindowFromPoint(pos)
```

获得指定位置颜色：

```
def get_color(pos):
    hdc_screen = win32gui.CreateDC("DISPLAY", "", None)
    hmem_dc = win32gui.CreateCompatibleDC(hdc_screen)
    h_bitmap = win32gui.CreateCompatibleBitmap(hdc_screen, 1, 1)
    h_old_bitmap = win32gui.SelectObject(hmem_dc, h_bitmap)
    win32gui.BitBlt(hmem_dc, 0, 0, 1, 1, hdc_screen, pos[0], pos[1], win32con.SRCCOPY)
    win32gui.DeleteDC(hdc_screen)
    win32gui.DeleteDC(hmem_dc)
    x = win32ui.CreateBitmapFromHandle(h_bitmap)
    bits = x.GetBitmapBits(True)
    return struct.unpack('I', bits)[0]
```

其实好像还有一个更方便的 API 可以取得指定位置的颜色：`GetPixel`。

但是 `PyWin32` 里面这个函数是 `PyCDC` 类的一个成员函数，不知道该怎么调用了。所以绕了下远路，

拷贝了一个 1 像素宽 1 像素高的 bitmap 来取得颜色，最后 `struct.unpack` 的时候可以改成 `'BBBB'` 之类的来以想要的格式获得数据。

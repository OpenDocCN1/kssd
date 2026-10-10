

# 使用 Win32API 制作简单外挂

> 原文：[`www.pediy.com/kssd/pediy11/105521.html`](https://www.pediy.com/kssd/pediy11/105521.html)

使用 Win32API 制作简单外挂  
前提：安装 Python 和 PyWin32  
  
模拟鼠标点击：  
  
def click(handle, pos):  
   client\_pos = win32gui.ScreenToClient(handle, pos)  
   tmp = win32api.MAKELONG(client\_pos\[0\], client\_pos\[1\])  
   win32gui.SendMessage(handle, win32con.WM\_ACTIVATE, win32con.WA\_ACTIVE, 0)  
   win32api.SendMessage(handle, win32con.WM\_LBUTTONDOWN, win32con.MK\_LBUTTON, tmp)   
  win32api.SendMessage(handle, win32con.WM\_LBUTTONUP, win32con.MK\_LBUTTON, tmp)获得当前鼠标位置：  
  
def get\_curpos():  
   return win32gui.GetCursorPos()获得指定位置窗口：  
  
def get\_win\_handle(pos):  
   return win32gui.WindowFromPoint(pos)获得指定位置颜色：  
  
def get\_color(pos):  
   hdc\_screen = win32gui.CreateDC("DISPLAY", "", None)  
   hmem\_dc = win32gui.CreateCompatibleDC(hdc\_screen)  
   h\_bitmap = win32gui.CreateCompatibleBitmap(hdc\_screen, 1, 1)  
   h\_old\_bitmap = win32gui.SelectObject(hmem\_dc, h\_bitmap)  
   win32gui.BitBlt(hmem\_dc, 0, 0, 1, 1, hdc\_screen, pos\[0\], pos\[1\], win32con.SRCCOPY)    
    win32gui.DeleteDC(hdc\_screen)   
  win32gui.DeleteDC(hmem\_dc)   
     x = win32ui.CreateBitmapFromHandle(h\_bitmap)   
  bits = x.GetBitmapBits(True)     
     return struct.unpack('I', bits)\[0\]其实好像还有一个更方便的 API 可以取得指定位置的颜色：GetPixel，  
  
但是 PyWin32 里面这个函数是 PyCDC 类的一个成员函数，不知道该怎么调用了。所以绕了下远路，  
  
拷贝了一个 1 像素宽 1 像素高的 bitmap 来取得颜色，最后 struct.unpack 的时候可以改成’BBBB’之  
  
类的来以想要的格式获得数据。
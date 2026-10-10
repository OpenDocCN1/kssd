

# 【代码之美】【自主命题】利用 Thunk 技术将 Win32 回调函数转换为 C++成员函数

> 原文：[`www.pediy.com/kssd/pediy10/76213.html`](https://www.pediy.com/kssd/pediy10/76213.html)

  
Thunk : 将一段机器码对应的字节保存在一个连续内存结构里,   
然后将其指针强制转换成函数. 即用作函数来执行.  
  
这份代码用于将 C 语方的回调函数转换为具有 this 指针的 C++成  
员函数.  
  
在 Windows 系统编程中, 可以将一些系统回函数封装成 C++类成  
员函数, 例如  
  
// 定时器回调函数  
VOID CALLBACK TimerProc(HWND hwnd, UINT uMsg,   
UINT\_PTR idEvent, DWORD dwTime);  
  
// 线程回调函数  
DWORD WINAPI ThreadProc(LPVOID lpParameter);  
  
// 窗口过程回调函数  
LRESULT CALLBACK WindowProc(HWND hwnd, UINT uMsg,   
WPARAM wParam, LPARAM lParam);  
  
2008-10-22 修正了在 Windows XP SP2, Windows server 2003 环  
境下开启 DEP 功能后运行崩溃的 BUG  
  
附件为 VC++6.0 源码及测试程序  
  
欢迎拍砖, 但希望更多的 C++程序使用它.  
谢谢!

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || Thunk.rar  
 |

`short GetKeyState(int vKey)`函数用来获得某个键到目前为止的状态，比如判断`Shift`是否按下`GetKeyState(VK_SHIFT)`，按下高位时1，否则是0，`GetKeyState(VK_CAPITAL)`（`CapsLock`键）如果打开低位返回1，注意这个`GetKeyState`返回`short`类型的值16位，不是上面说的`LParam`的值。`GetKeyState`不是实时检查状态的，指检查到目前为止的键盘状态。

`short GetAsyncKeyState(int vKey)`函数用来获取当前的某个键的当前状态。高位为1则当前判断的建被按下，低位返回1则，则按键在上次调用`GetAsyncKeyState`以来状态是被按下的。

`GetKeyState`判断组合键比较合适，因为可以判断某个键到目前为止的状态，按下了`Ctrl`再按下`S`，那么可以在`S`键的处理消息上判断`GetKeyState(VK_LCTRL)`是否按下。

`GetAsyncKeyState`可以用来做个循环，当某个键现在按下，处理某些事情。

## 字符消息

每当一个键被按下，就产生一个按键消息和字符消息，通常我们只关心字符消息，因为同样的按键产生的字符有可能是不同的，比如，打开搜狗输入法按`Shift` + 4打出的字符是￥，关闭输入法打出的是$。

字符消息的`wParam`参数是按键的ASCII值，所以在回调函数中可以`if (wParam == 'a')`这样判断输入的字符。

## 鼠标消息

鼠标按键全使用消息，每个键有3个消息，`BUTTONDOWN`、`BUTTONUP`、`BUTTONDBLCLK`（双击），`WM_L`（左键）、`WM_M`（中键）、`WM_R`（右键）加上三个消息代表了鼠标显示区消息。

鼠标的移动消息是`WM_MOUSEMOVE`。

此时`wParam`参数表示下列的按键是否被按下：`MK_CONTROL`、`MK_LBUTTON`、`MK_MBUTTON`、`MK_RBUTTON`、`MK_SHIFT`。

`lParam`低位代表鼠标X坐标，高位代表鼠标Y坐标。

可以看出`SendMessage()`发送的`WM_CHAR`消息不会被目标进程的窗口回调函数处理，因为`SendMessage`直接发送到回调函数，没有经过`TranslateMessage`翻译键盘消息。

## 线程间的数据共享

### `WM_SETTEXT`消息

首先说明`WM_SETTEXT`消息不是一个用来做进程间发送数据用的，这个消息是用来设置窗口标题，或者按钮文本，或者`Edit`控件内容的。比如`SetWindowText(HWND hwnd, LPCTSTR lpString)`（设置窗口的标题）调用这个函数实际上就是产生了一个`WM_SETTEXT`消息，通常由默认回调函数`DefWindowProc`来处理。想想就行了，只能发送一个字符串有什么用？

但是`WM_SETTEXT`消息特殊的地方就是，系统为用这个消息发送的字符串开辟另外一块共享内存映射空间，使不同进程接收消息的线程也能够收到并且使用这个字符串。

对应的还有一个`WM_GETTEXT`消息，这个消息是从目标窗口句柄返回字符串信息，同样`GetWindowText(HWND hwnd, LPCTSTR lpString, int iMaxCount)`函数也是产生一个`WM_GETTEXT`消息由`DefWindowProc`来处理，参数中多了一个`iMaxCount`用来表示字符串的长度。

### `WM_COPYDATA`消息

`WM_COPYDATA`消息把自定义的一块数据发送到目标线程，目标进程的窗口回调函数中必须有这个消息的处理方法，否则发了也没用。

`WM_COPYDATA`、`WM_SETTEXT`这两个数据传递消息都只能使用`SendMessage()`发送，`SendMessage`返回了系统就会释放开辟的内存空间，用其他的方法发送，系统不知道目标进程什么时候处理消息，所以也无法释放内存映射空间。

可以在`Send.asm`里加入下面代码看看如果用`PostMessage`返回什么错误提示。

```
lpBuffer        db        512  dup (?)   ;先定义一个buffer

invoke        PostMessage,hWnd,WM_COPYDATA,0,addr stCopyData  ;试用PostMessage发送,根本就没有发送消息

.if eax == 0

        invoke GetLastError

        invoke FormatMessage,FORMAT_MESSAGE_FROM_SYSTEM or FORMAT_MESSAGE_IGNORE_INSERTS,NULL,

                                eax,LANG_NEUTRAL,offset lpBuffer,sizeof lpBuffer,NULL

                                invoke MessageBox,NULL,offset lpBuffer,offset szCaption,MB_OK

.endif
```

这个方法是使用`GetLastError`函数先获得上一次调用函数失败的代码，然后通过`FormatMessage`找到错误代码的描述，参数里设置说明是中文。

大部分的winAPI在调用失败后都可以通过`GetLastError`获得调用失败的错误代码。这个方法很好用，可以及时了解为什么出错。

本章关于汇编本省的东西不多，但是既然是windows的程序设计，必须先要了解一些windows程序设计一些基本的知识。特别是消息机制，这是整个windows程序设计的基础。

我参考了核心编程，windows程序设计，把windows的消息机制先做个介绍，了解了这些，再去看本章的代码和2个消息发送例子就轻车熟路了。

本章将分为2个部分做笔记，第一部分消息机制，第二部分准备做一些反汇编方面的实验，比如比较C写的窗口，和汇编写的窗口返回编的结果为何不同，比如如何找到并且拦截消息等等，当然这只是想法，能不能有结果还不一定。

[\[招生\]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

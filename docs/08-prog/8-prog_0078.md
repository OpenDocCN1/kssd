
# Windows 下 32 位汇编语言学习笔记 第四章 第一个窗口程序 (Windows 的消息机制)

> 原文：[`www.pediy.com/kssd/pediy10/88027.html`](https://www.pediy.com/kssd/pediy10/88027.html)

## Windows 下 32 位汇编语言学习笔记 第四章 第一个窗口程序 1 (Windows 的消息机制)

`FirstWindow` 程序代码很简单，只有一个地方要说下

`_WinMain` 函数里的下面 2 行代码，把当前进程句柄赋值给 `WNDCLASSEX` 的 `hInstance`，这里不能使用 `mov @stWndClass.hInstance, hInstance`，因为 `mov` 指令不支持 2 块内存间的直接赋值。

所以先把 `hInstance` 压栈再弹出到 `@stWndClass.hInstance`

```
push hInstance
pop @stWndClass.hInstance
```

当然也可以这样做

```
mov ecx, hInstance
mov @stWndClass.hInstance, ecx
```

### WIN32 的消息机制

Windows 系统是一个消息驱动的 `OS`，操作通过处理各种消息来响应用户的操作。从第一个 Windows 程序就可以看出来，大部分的代码都是处理消息的。要开发 Windows 程序，不管你用什么开发工具什么语言，掌握消息机制的原理都是非常必要的。

对于每一个带有窗口的线程，系统都会给它分配一个自己的消息队列，用于处理消息派送（`Dispatch`）。每个线程都用自己的消息循环来接受消息。每个线程队列默认管理最大 `10000` 个消息，修改注册表下面的键值可以修改队列中的消息数。建议的最小值是 `4000`。

`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows\USERPostMessageLimit`.

线程队列不是一个公开的数据结构（`THREADINFO`），其中包括登记消息队列（Posted-message queue）、消息发送队列（Send-message queue）、消息应答队列（reply-message queue）、虚拟输入队列（virtualized-input queue）、唤醒标志（wake flag），以及用来扫描线程局部输入状态的若干变量。（Windows 核心编程）

### 消息队列提取优先级

1. 检查 `QS_SENDMESSAGE` 标志，`GetMessage` 不处理 Send 消息，如果队列中没有其他 send 消息，关闭 `QS_SENDMESSAGE` 标志，`GetMessage()` 不返回检查其他消息。
2. 检查 `QS_POSTMESSAGE` 标志，`GetMessage` 从此队列取出消息处理并由 `Dispatch` 分发到指定窗口回调函数处理。`GetMessage` 返回 `True`，没有其他 post 消息关闭标志。
3. 检查 `QS_QUIT` 标志，如果被 `PostQuitMessage()` 打开，则 `GetMessage` 返回 `False` 退出消息循环，并且关闭 `QS_QUIT` 标志。
4. 检查 `QS_INPUT` 标志，`GetMessage` 从此队列取出消息处理由 `TranslateMessage()` 处理键盘鼠标消息，然后由 `Dispatch` 分发到指定窗口回调函数处理没有其他消息关闭标志。
5. 检查 `QS_PAINT` 标志，处理同 2。
6. 检查 `QS_TIMER` 标志，首先复位计时器，`GetMessage` 返回 `True`，如果没有计数器，关闭 `QS_TIMER` 标志。

优先级很清楚，send 优先级最高，最低的是 time。

Windows 定义了很多消息都以 `WM_` 开头，都是用 `#DEFINE` 定义的常量，用户可以定义自己的消息，Windows 规定用户的消息从 `WM_USER` `0x0400` 开始。

`BOOL PostMessage(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam);`

往进程的消息队列发送消息 `PostMessage`，这个函数往指定进程的消息队列发送一个消息，发送完毕立即返回。调用函数无法知道发送的消息是否能被处理。如果这个指定窗口未处理完自己消息队列的所有消息前就退出了，就会处理不到 post 的消息。

`PostMessage` 发送的消息参数不能包含指针参数，`MSDN` 的说明是：

“如果发送一个低于 `WM_USER` 范围的消息给异步消息函数（`PostMessage`、`SendNotifyMessage`、`SendMessageCallback`），消息参数不能包含指针。否则，操作将会失败。函数将在接收线程处理消息之前返回，发送者将在内存被使用之前释放。”

我的理解是，就算目标进程知道你发来的是个指针地址，但是 2 个进程之间的寻址空间是独立的，互相不可访问，怎么能获取发送进程内存空间里的数据呢？

可以试下，把例子 `Send.asm` 修改一下看看。

`WM_USER` 值是 `0x0400`，用户自定义消息从这里开始，前面的是 Windows 保留的消息。这里必须把 `WM_SETTEXT` 消息改成自定义消息，否则 `PostMessage` 根本不会发送 `WM_SETTEXT` 消息给目标线程。

```
invoke PostMessage, hWnd, WM_USER + 1, 0, addr szText
```

`Receive.asm` 修改接受 `WM_SETTEXT` 消息的代码改成这样：

```
eax == WM_USER + 1
invoke MessageBox, hWnd, addr lParam, addr szCaptionMain, MB_OK
```

用 `OD` 打开 `Receive.exe` 运行在 `0040102B` 处下断点，运行 `send.exe`，发送之前提示的地址（`00402072`）到了 `Receive`，Windows 干脆直接把你传递的指针地址当做 `DWORD` 类型处理了，显示 "r @"，是 `00402072` 的 ASCII 码值。就算当成指针处理，`00402072` 也是指向 `Receive` 内存空间里的，不可能是指向 `send` 内存空间里的数据。

### 关于 `WM_QUIT` 消息

窗口回调函数里不可能接到 `WM_QUIT` 消息。因为从消息队列里 `GetMessage()` 收到 `WM_QUIT` 消息就返回 `0`，消息循环就会结束，所以 `DispatchMessage()` 也不可能再把这个消息分发到窗口的回调函数。这就是为什么，书里一再强调要在 `WM_DESTROY` 的消息事件里加上 `PostQuitMessage()` 的原因。如果不加，程序只是销毁窗口，但是进程仍然存在。消息循环还在运行，但是因为窗口已经销毁，所以它不可能再从消息队列里取得任何消息。

使用 `PostQuitMessage()` 与 `PostMessage()` 发送消息的不同：

前者把消息队列里的 `QS_QUIT` 标志打开，并且等待程序处理完消息队列里的所有消息后才结束消息循环。

后者是把 `WM_QUIT` 直接放到消息队列，消息循环取到的下一个消息是 `WM_QUIT` 就立即退出。`MSDN` 里不建议使用 `PostMessage` 发送 `WM_QUIT` 消息，因为这样会造成程序的收尾工作无法进行，正常退出后所需要的资源释放等操作就没法执行了。

用 `SendMessage` 无法发送 `WM_QUIT` 消息，因为 `SendMessage` 并不是把消息放入消息队列，所以，`GetMessage` 根本无法得到 `SendMessage` 发送的消息。

`BOOL PostThreadMessage(DWORD dwThreadId, UINT uMsg, WPARAM wParam, LPARAM lParam);`

这个函数和 `PostMessage` 类似，都是发送完消息立即返回，不同的是这个函数向指定的 `ThreadId` 发送一条消息。这个函数发送的消息不会被分配到目标进程窗口的回调函数，因为当消息放入队列时，`MSG` 的 `hwnd` 被设置为 `NULL`，没有窗口句柄，`DispatchMessage` 能把消息分配给谁呢？`PostThreadMessage` 也可以发送 `WM_QUIT` 消息，消息会放到队列的尾端。在 `qs_input` 之前处理该消息。

`PostMessage` 和 `PostThreadMessage` 发送 `WM_QUIT` 消息都会造成窗口的收尾代码无法执行。用的时候需要注意下。

`LRESULT SendMessage(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam)`

`SendMessage` 同步发送消息，发送进程要等待目标进程窗口的回调函数处理完成消息后才能恢复运行，调用点线程在等待 `SendMessage` 返回的过程中是挂起状态，本身也无法响应任何操作。

发送进程在等待的过程中，如果系统中其他的进程向等待进程发送消息，则发送进程立即处理消息。

### Windows 提供了其他的 4 个 API 来进行进程间的消息发送

`LRESULT SendMessageTimeout(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam, UINT fuFlags, UINT fuTimeout, PDWORD_PTR pdwResult);`

`fuFlags` 参数由下列标志组成：

- `SMTO_ABORTIFHUNG`：如果目标进程处于挂起状态立即返回。
- `SMTO_BLOCK`：发送进程在 `SendMessageTimeout` 返回之前不处理任何消息。
- `SMTO_NORMAL`：`0` 值，如果不使用其他标志，就是用这个标志。
- `SMTO_NOTIMEOUTIFNOTHUNG`：如果目标进程未处于挂起状态不考虑 `fuTimeout` 限定等待值。

`fuTimeout` 参数指定等待时间单位毫秒。

`pdwResult` 指向一段内存区域，保存返回结果。如果用 `SendMessageTimeout` 本身线程的窗口则直接调用窗口的回调函数，并且将结果保存在 `pdwResult` 中。

`BOOL SendMessageCallback(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam, SENDASYNCPROC lpCallback, ULONG_PTR dwData);`

`lpCallback` 参数指向一个 `CALLBACK` 函数，定义如下：

```
VOID CALLBACK ResultBack(HWND hwnd, UINT uMsg, ULONG_PTR dwData, LRESULT lResult);
```

发送线程使用 `SendMessageCallback` 发送消息到接收线程的发送消息队列，并立即返回。当接收线程处理完消息后，用一个消息登记到发送线程的应答消息队列，然后系统调用 `ResultBack` 函数通知发送进程。前 2 个参数是接收线程窗口的句柄、消息值，第三个参数 `dwData` 就是 `SendMessageCallback` 中最后一个参数，`lResult` 参数是接收消息窗口回调函数的返回值。

接收进程处理完 `SendMessageCallback` 函数后先在发送进程消息队列登记应答消息，发送进程在下一次调用 `GetMessage`、`PeekMessage` 时，执行 `ResultBack` 函数。

`Bool SendNotifyMessage(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam);`

`SendNotifyMessage` 将消息放到接收线程的发送消息队列（`QS_SENDMESSAGE`）中，并且立即返回。和 `PostMessage` 类似，但不同的是：

发送消息队列的优先级比登记队列（`QS_POSTMESSAGE`）的优先级高。所以 `SendNotifyMessage` 发送的消息比 `PostMessage` 发送的消息处理的早。

向进程发送窗口消息时，`SendNotifyMessage` 效果和 `SendMessage` 完全一样，等待消息处理完之后才返回。

`BOOL ReplyMessage(LRESULT lResult);`

这个函数是用于接收线程窗口的回调函数中，调用 `ReplyMessage` 后，发送线程恢复运行。

### 判断消息类型

`BOOL InSendMessage();`

如果当前消息是进程间消息，返回 `TRUE`，如果是进程内消息返回 `FALSE`。

`DWORD InSendMessageEx(PVOID pvReserved);`

这个函数返回正在执行的消息类型。返回值如下:

- `ISMEX_NOSEND`：消息是线程内部消息
- `ISMEX_SEND`：消息是用 `SendMessage` 或 `SendMessageTimeout` 发送的进程间消息
- `ISMEX_NOTIFY`：消息是 `SendNotifyMessage` 发送的进程间消息
- `ISMEX_CALLBACK`：消息是 `SendMessageCallback` 发送的进程间消息
- `ISMEX_REPLIED`：消息是进程间消息，并且已经调用 `ReplyMessage`

### 消息队列的状态标志

`DWORD GetQueueStatus(UINT fuFlags);`

参数 `fuFlags` 是由一组标志联合起来的值，用来查看特定的唤醒队列标志。

- `QS_KEY`：`WM_KEYUP`、`WM_KEYDOWN`、`WM_SYSKEYUP` 或 `WM_SYSKEYDOWN`
- `QS_MOUSE`：`WM_MOUSEMOVE`
- `QS_MOUSEBUTTON`：`WM_?BUTTON*`（其中？代表 `L`、`M` 或 `R`，`*` 代表 `DOWN`、`UP` 或 `DBLCLK`）
- `QS_MOUSE`：同 `QS_MOUSEMOVE|QS_MOUSEBUTTON`
- `QS_INPUT`：同 `QS_MOUSE|QS_KEY`
- `QS_PAINT`：`WM_PAINT`
- `QS_TIMER`：`WM_TIMER`
- `QS_HOTKEY`：`WM_HOTKEY`
- `QS_POSTMESSAGE`：登记的消息（不同于硬件输入事件）。当队列在期望的消息过滤器范围内没有登记的消息时，这个标志要消除。除此之外，这个标志与 `QS_ALLPOSTMESSAGE` 相同
- `QS_ALLPOSTMESSAGE`：登记的消息（不同于硬件输入事件）。当队列完全没有登记的消息时（在任何消息过滤器范围），该标志被清除。除此之外，该标志与 `QS_POSTMESSAGE` 相同
- `QS_ALLEVENTS`：同 `QS_INPUT|QS_POSTMESSAGE|QS_TIMER|QS_PAINT|QS_HOTKEY`
- `QS_QUIT`：已调用 `PostQuitMessage`。注意这个标志没有公开，所以在 `WinUser.h` 文件中没有。它由系统在内部使用
- `QS_SENDMESSAGE`：由另一个线程发送的消息
- `QS_ALLINPUT`：同 `QS_ALLEVENTS|QS_SENDMESSAGE`

消息类型存放在返回值的高字节中（2 个字节），低字节储存还没有处理的消息类型。

上面几个函数都是用来发送消息，很多函数不是常用的，但多了解几个函数没有坏处，了解的东西越多，遇到问题解决的办法也就越多。

### 键盘，鼠标消息

Windows 程序与用户的交互都是通过鼠标键盘实现的，所以必须要了解 Windows 是如何处理键盘鼠标消息的。

首先，发生的键盘鼠标消息是先报存在系统消息队列的（不是直接发放到应用程序队列），当应用程序处理上一个输入消息后，系统消息队列才把下一个输入消息投放到应用程序队列。因为如果按键的输入速度比应用程序处理速度快，后来的键如果还是发往当前的焦点窗口句柄，那么切换到新窗口后后来输入的键还是会发送到先前的窗口，直到上一个窗口处理完所有的未处理的按键消息，按键才会改变发送的窗口句柄到新窗口。

其次，每一个按键产生 2 类消息，按键消息和字符消息。很显然，有的按键只有按键消息没有字符消息，比如 `CapsLk`、`Shift` 等。

按键又分为系统按键和非系统按键。对于系统按键，当按下一个键发生 `WM_SYSKEYDOWN` 消息，放开这个键发生 `WM_SYSKEYUP` 消息。对于非系统键，按下和放开发生 `WM_KEYDOWN` 和 `WM_KEYUP` 消息。

很显然这些消息都成对出现的。一个 `KEYDOWN`，接着就是一个 `KEYUP`。

对于系统按键，通常是 Windows 系统本身比较关心的消息，系统按键通常由 `ALT` 快捷键产生，`Alt tab`、`Alt F4`、`Alt esc` 等等。应用程序不处理 `ALT` 消息，而是交给 `DefWindowProc` 来处理，这就说明应用程序的菜单快捷键也是由系统处理。系统将 `Ctrl+s` 这类的快捷键，转换成菜单命令消息，不用自己去处理。

对于所有的 4 类按键消息 `WM_SYSKEYDOWN`、`WM_SYSKEYUP`、`WM_KEYDOWN`、`WM_KEYUP`，`wParam` 参数保存虚拟键代码，`lParam` 参数包含按键的其他数据。

产生虚拟键代码的原因是因为早期的键码是由真实键盘产生，叫做“扫描码”，扫描码是按照键盘的排列顺序产生的，比如 `16` 是 `Q`，`17` 是 `W`（数数看，呵呵）。很明显这种键码会因为键盘布局的变化而变化，太过于设备化，于是通过定义虚拟键代码。

虚拟代码是一些列 `VK` 开头的定义在 `winuser.h` 里的值。例如 `VK_TAB`、`VK_RETURN`（回车键）等等，键盘数字 `0-9` 和字母 `a-z`、`A-Z` 就是 `ASCII` 的值。小键盘上的数字是 `VK_NUMPAD0`-`VK_NUMPAD9`，其他的功能键都是 `VK_+` 键的英文含义组成。

`lParam` 参数的 `32` 位分成 `6` 个字段，用于表示不同的消息。



### 消息机制基础

#### 键盘消息

*   00-15 位：包含按键的重复次数。
*   16-23 位：包含按键的 OEM 扫描码（即扫描码）。
*   24 位：包含按键的扩充标志，此标准被 `Windows` 忽略不用。
*   29 位：包含按键的内容代码，对于系统键盘此位是 1，对于非系统键此位为 0。
*   30 位：包含按键的先前状态，如果键是先前释放的，为 0，否则为 1。
*   31 位：包含按键的转换状态，如果键盘按按下，为 0，否则为 1。
*   25-28 位：未知。

`short GetKeyState(int vKey)` 函数用来获得某个键到目前为止的状态，比如判断 `Shift` 是否按下 `GetKeyState(VK_SHIFT)`，按下高位时 1，否则是 0。`GetKeyState(VK_CAPITAL)` (`CapsLock` 键) 如果打开低位返回 1。注意这个 `GetKeyState` 返回 `short` 类型的值 16 位，不是上面说的 `LParam` 的值。`GetKeyState` 不是实时检查状态的，指检查到目前为止的键盘状态。

`short GetAsyncKeyState(int vKey)` 函数用来获取当前的某个键的当前状态。高位为 1 则当前判断的键被按下，低位返回 1 则按键在上次调用 `GetAsyncKeyState` 以来状态是被按下的。

`GetKeyState` 判断组合键比较合适，因为可以判断某个键到目前为止的状态。按下了 `Ctrl` 再按下 `S`，那么可以在 `S` 键的处理消息上判断 `GetKeyState(VK_LCTRL)` 是否按下。

`GetAsyncKeyState` 可以用来做个循环，当某个键现在按下，处理某些事情。

**字符消息**

每当一个键被按下，就产生一个按键消息和字符消息，通常我们只关心字符消息，因为同样的按键产生的字符有可能是不同的。比如，打开搜狗输入法按 `Shift` + `4` 打出的字符是 `￥`，关闭输入法打出的是 `$`。

字符消息的 `wParam` 参数是按键的 ASCII 值，所以在回调函数中可以用 `if (wParam == 'a')` 这样判断输入的字符。

**鼠标消息**

鼠标按键全使用消息，每个键有 3 个消息：`BUTTONDOWN`、`BUTTONUP`、`BUTTONDBLCLK` (双击)。`WM_L`（左键）、`WM_M`（中键）、`WM_R`（右键）加上三个消息代表了鼠标显示区消息。

鼠标的移动消息是 `WM_MOUSEMOVE`。

此时 `wParam` 参数表示下列的按键是否被按下：`MK_CONTROL`、`MK_LBUTTON`、`MK_MBUTTON`、`MK_RBUTTON`、`MK_SHIFT`。

`lParam` 低位代表鼠标 X 坐标，高位代表鼠标 Y 坐标。

可以看出 `SendMessage()` 发送的 `WM_CHAR` 消息不会被目标进程的窗口回调函数处理，因为 `SendMessage` 直接发送到回调函数，没有经过 `TranslateMessage` 翻译键盘消息。

#### 线程间的数据共享

**`WM_SETTEXT` 消息**

首先说明 `WM_SETTEXT` 消息不是一个用来做进程间发送数据用的，这个消息是用来设置窗口标题，或者按钮文本，或者 `Edit` 控件内容的。比如 `SetWindowText(HWND hwnd, LPCTSTR lpString)`（设置窗口的标题）调用这个函数实际上就是产生了一个 `WM_SETTEXT` 消息，通常由默认回调函数 `DefWindowProc` 来处理。想想就行了，只能发送一个字符串有什么用？

但是 `WM_SETTEXT` 消息特殊的地方就是，系统为用这个消息发送的字符串开辟另外一块共享内存映射空间，使不同进程接收消息的线程也能够收到并且使用这个字符串。

对应的还有一个 `WM_GETTEXT` 消息，这个消息是从目标窗口句柄返回字符串信息。同样 `GetWindowText(HWND hwnd, LPCTSTR lpString, int iMaxCount)` 函数也是产生一个 `WM_GETTEXT` 消息由 `DefWindowProc` 来处理，参数中多了一个 `iMaxCount` 用来表示字符串的长度。

**`WM_COPYDATA` 消息**

`WM_COPYDATA` 消息把自定义的一块数据发送到目标线程，目标进程的窗口回调函数中必须有这个消息的处理方法，否则发了也没用。

`WM_COPYDATA` 和 `WM_SETTEXT` 这两个数据传递消息都只能使用 `SendMessage()` 发送。`SendMessage` 返回了系统就会释放开辟的内存空间；用其他的方法发送，系统不知道目标进程什么时候处理消息，所以也无法释放内存映射空间。

可以在 `Send.asm` 里加入下面代码看看如果用 `PostMessage` 返回什么错误提示。

```assembly
lpBuffer  db  512  dup (?)   ; 先定义一个 buffer
invoke  PostMessage,hWnd,WM_COPYDATA,0,addr stCopyData  ; 试用 PostMessage 发送,根本就没有发送消息
.if eax == 0
  invoke GetLastError
  invoke FormatMessage,FORMAT_MESSAGE_FROM_SYSTEM or FORMAT_MESSAGE_IGNORE_INSERTS,NULL,
        eax,LANG_NEUTRAL,offset lpBuffer,sizeof lpBuffer,NULL
        invoke MessageBox,NULL,offset lpBuffer,offset szCaption,MB_OK        
.endif
```

这个方法是使用 `GetLastError` 函数先获得上一次调用函数失败的代码，然后通过 `FormatMessage` 找到错误代码的描述，参数里设置说明是中文。

大部分的 `winAPI` 在调用失败后都可以通过 `GetLastError` 获得调用失败的错误代码。这个方法很好用，可以及时了解为什么出错。

本章关于汇编本身的东西不多，但是既然是 `Windows` 的程序设计，必须先要了解一些 `Windows` 程序设计一些基本的知识。特别是消息机制，这是整个 `Windows` 程序设计的基础。

我参考了核心编程，`Windows` 程序设计，把 `Windows` 的消息机制先做个介绍，了解了这些，再去看本章的代码和 2 个消息发送例子就轻车熟路了。

本章将分为 2 个部分做笔记，第一部分消息机制，第二部分准备做一些反汇编方面的实验，比如比较 `C` 写的窗口，和汇编写的窗口反汇编的结果为何不同，比如如何找到并且拦截消息等等，当然这只是想法，能不能有结果还不一定。

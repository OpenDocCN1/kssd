
```c
typedef struct _USER_MESSAGE_QUEUE
{
/* Reference counter, only access this variable with interlocked functions! */
LONG References;
/* Owner of the message queue */
struct _ETHREAD *Thread;
/* Queue of messages sent to the queue. */
LIST_ENTRY SentMessagesListHead;
/* Queue of messages posted to the queue. */
LIST_ENTRY PostedMessagesListHead;
/* Queue of sent-message notifies for the queue. */
LIST_ENTRY NotifyMessagesListHead;
/* Queue for hardware messages for the queue. */
LIST_ENTRY HardwareMessagesListHead;
/* List of timers, sorted on expiry time (earliest first) */
LIST_ENTRY TimerListHead;
/* Lock for the hardware message list. */
KMUTEX HardwareLock;
/* Pointer to the current WM_MOUSEMOVE message */
PUSER_MESSAGE MouseMoveMsg;
/* True if a WM_QUIT message is pending. */
BOOLEAN QuitPosted;
/* The quit exit code. */
ULONG QuitExitCode;
/* Set if there are new messages specified by WakeMask in any of the queues. */
PKEVENT NewMessages;
/* Handle for the above event (in the context of the process owning the queue). */
HANDLE NewMessagesHandle;
/* Last time PeekMessage() was called. */
ULONG LastMsgRead;
/* Current window with focus (ie. receives keyboard input) for this queue. */
HWND FocusWindow;
/* Count of paints pending. */
ULONG PaintCount;
/* Current active window for this queue. */
HWND ActiveWindow;
/* Current capture window for this queue. */
HWND CaptureWindow;
/* Current move/size window for this queue */
HWND MoveSize;
/* Current menu owner window for this queue */
HWND MenuOwner;
/* Identifes the menu state */
BYTE MenuState;
/* Caret information for this queue */
PTHRDCARETINFO CaretInfo;
/* Window hooks */
PHOOKTABLE Hooks;
/* queue state tracking */
WORD WakeMask;
WORD QueueBits;
WORD ChangedBits;
/* extra message information */
LPARAM ExtraInfo;
/* messages that are currently dispatched by other threads */
LIST_ENTRY DispatchingMessagesHead;
/* messages that are currently dispatched by this message queue, required for cleanup */
LIST_ENTRY LocalDispatchingMessagesHead;
/* Desktop that the message queue is attached to */
struct _DESKTOP_OBJECT *Desktop;
} USER_MESSAGE_QUEUE, *PUSER_MESSAGE_QUEUE;
```

可以看到该结构中共有七种不同类型的队列，`Thread`成员则指向了该队列的所属线程。

# 三. 窗口对象

消息最开始是由窗口捕获的，窗口捕获到消息以后，要将消息发送给窗口的所属的线程的消息队列。这样，窗口所属线程才可以从消息队列中取出消息进行处理。因此，要想捕获消息，首先就需要创建一个窗口。

在用户层可以通过`CreateWindow`函数来创建窗口，该函数通过调用`win32k.sys`的`NtUserCreateWindowEx`来在内核中创建一个窗口对象，窗口对象的的结构是`_WINDOW_OBJECT`，该结构的定义如下：

```c
typedef struct _WINDOW_OBJECT
{
/* Pointer to the thread information */
PW32THREADINFO ti;
/* Pointer to the desktop */
PDESKTOP Desktop;
union
{
/* Pointer to a call procedure handle */
PCALLPROC CallProc;
/* Extra Wnd proc (windows of system classes) */
WNDPROC WndProcExtra;
};
/* Pointer to another call procedure handle (used for returning the previous
window proc in SetWindowLongPtr) */
PCALLPROC CallProc2;
/* Indicates whether the window is derived from a system class */
BOOL IsSystem;
/* Pointer to the window class. */
PWINDOWCLASS Class;
/* Extended style. */
DWORD ExStyle;
/* Window name. */
UNICODE_STRING WindowName;
/* Style. */
DWORD Style;
/* Context help id */
DWORD ContextHelpId;
/* system menu handle. */
HMENU SystemMenu;
/* Handle of the module that created the window. */
HINSTANCE Instance;
/* Entry in the thread's list of windows. */
LIST_ENTRY ListEntry;
/* Pointer to the extra data associated with the window. */
PCHAR ExtraData;
/* Size of the extra data associated with the window. */
ULONG ExtraDataSize;
/* Position of the window. */
RECT WindowRect;
/* Position of the window's client area. */
RECT ClientRect;
/* Handle for the window. */
HWND hSelf;
/* Window flags. */
ULONG Flags;
/* Window menu handle or window id */
UINT IDMenu;
/* Handle of region of the window to be updated. */
HANDLE UpdateRegion;
/* Handle of the window region. */
HANDLE WindowRegion;
/* Pointer to the owning thread's message queue. */
PUSER_MESSAGE_QUEUE MessageQueue;
struct _WINDOW_OBJECT* FirstChild;
struct _WINDOW_OBJECT* LastChild;
struct _WINDOW_OBJECT* NextSibling;
struct _WINDOW_OBJECT* PrevSibling;
/* Entry in the list of thread windows. */
LIST_ENTRY ThreadListEntry;
/* Handle to the parent window. */
struct _WINDOW_OBJECT* Parent;
/* Handle to the owner window. */
HWND hOwner;
/* DC Entries (DCE) */
PDCE Dce;
/* Property list head.*/
LIST_ENTRY PropListHead;
ULONG PropListItems;
/* Scrollbar info */
PWINDOW_SCROLLINFO Scroll;
LONG UserData;
BOOL Unicode;
WNDPROC WndProc;
PETHREAD OwnerThread;
HWND hWndLastPopup; /* handle to last active popup window (wine doesn't use
pointer, for unk. reason)*/
PINTERNALPOS InternalPos;
ULONG Status;
/* counter for tiled child windows */
ULONG TiledCounter;
/* WNDOBJ list */
LIST_ENTRY WndObjListHead;
} WINDOW_OBJECT; /* PWINDOW_OBJECT already declared at top of file */
```

| `名称` | `作用` |

|:------------:|:----------------------------------------------------------------------------------------------------------------------------------------------:|

| `MessageQueue` | 指向的是一个报文队列，一般就是所属线程的报文队列，也就是上面受得包含七种不同类型的`USER_MESSAGE_QUEUE`类型的报文队列。一个线程只有一个报文队列，但是一个线程可以由多个视窗，如果那样则每个视窗的指针`MessageQueue`都指向同一个报文队列，就是其所属线程的报文队列。 |

| `WndProc` | 函数指针，每个窗口接收到一个报文，在要对其做出反应的时候，就需要调用窗口的这个函数，所以这是视图报文的处理函数 |

| `OwnerThread` | 指向具体视窗所属线程的`ETHREAD`数据结构 |

# 四. 消息的接收

在用户层可以通过`GetMessageA`或`GetMessageW`来接收消息，`GetMessageW`的函数实现如下：

```cpp
BOOL STDCALL
GetMessageW(LPMSG lpMsg,
HWND hWnd,
UINT wMsgFilterMin,
UINT wMsgFilterMax)
{
BOOL Res;
MSGCONVERSION Conversion;
NTUSERGETMESSAGEINFO Info;
PUSER32_THREAD_DATA ThreadData = User32GetThreadData();
MsgConversionCleanup(lpMsg, FALSE, FALSE, NULL);
Res = NtUserGetMessage(&Info, hWnd, wMsgFilterMin, wMsgFilterMax); // 系统调用
if (-1 == (int) Res)
{
return Res;
}
Conversion.LParamSize = Info.LParamSize;
Conversion.KMMsg = Info.Msg; // 接收到的报文
// 将内核模式的报文转换为用户模式
if (! MsgiKMToUMMessage(&Conversion.KMMsg, &Conversion.UnicodeMsg))
{
return (BOOL) -1;
}
*lpMsg = Conversion.UnicodeMsg; // 返回转换好的用户模式报文
Conversion.Ansi = FALSE;
Conversion.FinalMsg = lpMsg;
MsgConversionAdd(&Conversion);
if (Res && lpMsg->message != WM_PAINT && lpMsg->message != WM_QUIT)
{
ThreadData->LastMessage = Info.Msg; // 复制内核模式报文
}
return Res;
}
```

参数`lpMsg`是个指针，用来返回获取的报文，报文是通过系统调用`NtUserGetMessage`从内核获取的，但是从内核获取的报文是“内核模式”的报文，与用户空间实际使用的报文在格式上有所不同，所以要经过`MsgiKMToUMMessage`函数将其转换为"用户模式"的报文。视图子系统的保卫呢是针对窗口而不仅仅是线程或进程的，所以在获取报文时需要说明是哪一个窗口，参数`hWnd`就是一个已创建（或已打开）窗口的句柄。其余两个参数就不那么重要了，那是用来选择所获取报文的类型范围的。

系统调用`NtUserGetMessage`从内核获取报文时，需要准备好`NTUSERGETMESSAGEINFO`数据结构，让内核往里面填写，该结构定义如下：

```c
typedef struct tagNTUSERGETMESSAGEINFO
{
MSG Msg;
ULONG LParamSize;
} NTUSERGETMESSAGEINFO, *PNTUSERGETMESSAGEINFO;
```

这里`Msg`是个`MSG`数据结构，所谓窗口报文，实际上就是`MSG`数据结构，结构定义如下：

```c
typedef struct tagMSG {
HWND hwnd;
UINT message;
WPARAM wParam;
LPARAM lParam;
DWORD time;
POINT pt;
} MSG,*LPMSG,*PMSG;
```

可见报文是以具体的窗口为目标的，但是`MSG`结构中并没有用来挂入队列的指针或队列头。实际上，挂入报文队列的是`USER_MESSAGE`数据结构，而`MSG`是其内部的核心成员，该结构的定义如下：

```c
typedef struct _USER_MESSAGE
{
LIST_ENTRY ListEntry;
BOOLEAN FreeLParam;
MSG Msg;
} USER_MESSAGE, *PUSER_MESSAGE;
```

在系统调用`NtUserGetMessage`中会通过调用`co_IntPeekMessage`从消息队列中获取消息

```c
do
{
GotMessage = co_IntPeekMessage(&Msg, hWnd, MsgFilterMin, MsgFilterMax, PM_REMOVE);
// 省略以下代码
}
while (! GotMessage);
```

在`co_IntPeekMessage`中会通过线程对象的`Win32Thread`来获取线程的消息队列，增加删除队列的标志，然后调用`co_MsgDispatchOneSentMessage`来首先循环获取和处理`SendMessage`队列中的消息

```c
ThreadQueue = (PUSER_MESSAGE_QUEUE)PsGetCurrentThreadWin32Thread()->MessageQueue;
RemoveMessages = RemoveMsg & PM_REMOVE;
while (co_MsqDispatchOneSentMessage(ThreadQueue));
```

> 通过`SendMessage`发送的消息是针对线程的，而不是针对某个窗口的。因此，在这种情况下，线程的消息循环必须直接处理它们，而不是交给`DispatchMessage`来处理。而且，这种通过消息来实现跨进程通信的方式仅限于同属一个桌面的进程之间才可以进行

当处理完成`SendMessage`的消息以后，才开始从其他队列中获取消息

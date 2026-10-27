
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

# 三.窗口对象

消息最开始是由窗口捕获的，窗口捕获到消息以后，要将消息发送给窗口的所属的线程的消息队列。这样，窗口所属线程才可以从消息队列中取出消息进行处理。因此，要想捕获消息，首先就需要创建一个窗口。

在用户层可以通过`CreateWindow`函数来创建窗口，该函数通过调用`win32k.sys`的`NtUserCreateWindowEx`来在内核中创建一个窗口对象，窗口对象的结构是`_WINDOW_OBJECT`，该结构的定义如下：

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

    /* Pointer to another call procedure handle (used for returning the previous window proc in SetWindowLongPtr) */
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

    struct _WINDOW_OBJECT *FirstChild;
    struct _WINDOW_OBJECT *LastChild;
    struct _WINDOW_OBJECT *NextSibling;
    struct _WINDOW_OBJECT *PrevSibling;

    /* Entry in the list of thread windows. */
    LIST_ENTRY ThreadListEntry;

    /* Handle to the parent window. */
    struct _WINDOW_OBJECT *Parent;

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
    HWND hWndLastPopup; /* handle to last active popup window (wine doesn't use pointer, for unk. reason)*/
    PINTERNALPOS InternalPos;
    ULONG Status;

    /* counter for tiled child windows */
    ULONG TiledCounter;

    /* WNDOBJ list */
    LIST_ENTRY WndObjListHead;

} WINDOW_OBJECT;
/* PWINDOW_OBJECT already declared at top of file */
```

| **名称** | **作用** |
| --- | --- |
| `MessageQueue` | 指向的是一个报文队列，一般就是所属线程的报文队列，也就是上面所述的包含七种不同类型的`USER_MESSAGE_QUEUE`类型的报文队列。一个线程只有一个报文队列，但是一个线程可以由多个视窗，如果那样则每个视窗的指针`MessageQueue`都指向同一个报文队列，就是其所属线程的报文队列。 |
| `WndProc` | 函数指针，每个窗口接收到一个报文，在要对其做出反应的时候，就需要调用窗口的这个函数，所以这是视图报文的处理函数 |
| `OwnerThread` | 指向具体视窗所属线程的`ETHREAD`数据结构 |

# 四.消息的接收

在用户层可以通过`GetMessageA`或`GetMessageW`来接收消息，`GetMessageW`的函数实现如下：

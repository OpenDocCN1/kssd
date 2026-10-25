
# win32 asm 第二十四课 windows 钩子

> 原文：[`www.pediy.com/kssd/pediy10/60504.html`](https://www.pediy.com/kssd/pediy10/60504.html)

Tutorial 24: Windows Hooks  
第二十四课：windows 钩子  

我们将 在这一课中学习 windows 钩子。Windows 钩子是非常有用的。 用它们，你能截获其它进程并且还可以改变它们的行为。 

Download the example here.  

## Theory

Windows hooks can be considered one of the most powerful features of Windows. With them, you can trap events that will occur, either in your own process or in other processes. By "hooking", you tell Windows about a function, filter function also called hook procedure, that will be called everytime an event you're interested in occurs. There are two types of them: local and remote hooks.  

Windows 钩子被认为是 windows 中强有力的特征之一。用它们，你可以在你自己的进程或是其它进程中捕获将要发生的事件。通过“挂钩”你告诉 windows 关于一个过滤函数的信息，过滤函数也叫钩子过程，每当你感兴趣的事件发生时，这个函数将被调用。钩子有两种类型：局部钩子和远程钩子。  

- **Local hooks** trap events that will occur in your own process.  
  局部钩子捕获的是在你自己的进程中将要发生的事件。  

- **Remote hooks** trap events that will occur in other process(es). There are two types of remote hooks:  
  远程钩子捕获的是在其它进程中将要发生的事件。远程钩子的类型也有两种：  
  - **thread-specific**: traps events that will occur in a specific thread in other process. In short, you want to observe events in a specific thread in a specific process.  
    基于线程的 捕获的事件是在其它进程中特殊线程将要发生的事件。简而言之，你想要观察的事件发生在一个具体的线程中，而这个线程是一特殊进程的线程。  
  - **system-wide**: traps all events destined for all threads in all processes in the system.  
    系统范畴的 捕获系统中所有进程的全部线程将要发生的事件。（捕捉系统中所有进程将发生的事件消息。）  

When you install hooks, remember that they affect system performance. System-wide hooks are the most notorious. Since ALL related events will be routed through your filter function, your system may slow down noticeably. So if you use a system-wide hook, you should use it judiciously and unhook it as soon as you don't need it. Also, you have a higher chance of crashing the other processes since you can meddle with other processes and if something is wrong in your filter function, it can pull the other processes down to oblivion with it. Remember: Power comes with responsibility.  

当你安装钩子时，铭记它们带来的后果是影响系统性能。系统范畴的钩子大多数是臭名昭著的。因为所有相关事件都将被你的过滤函数过滤 ，你的系统可能会变的很慢。所以，如果你用系统范畴的钩子， 你应该明智的用它并且一旦你不需要它们时就让它们脱钩。同样，你也有较高的机会让其它进程发生异常，因为你能干涉其它进程并且如果你的钩子函数出了问题，它能拉倒其它进程直到被湮没。铭记一点：性能和责任一起供给。能力伴随责任（功能强大也意味着使用时要负责任。）

You have to understand how a hook works before you can use it efficiently. When you create a hook, Windows creates a data structure in memory, containing information about the hook, and adds it to a linked list of existing hooks. New hook is added in front of old hooks. When an event occurs, if you install a local hook, the filter function in your process is called so it's rather straightforward. But if it's a remote hook, the system must inject the code for the hook procedure into the address space(s) of the other process(es). And the system can do that only if the function resides in a DLL. Thus , if you want to use a remote hook, your hook procedure must reside in a DLL. There is two exceptions to this rule: journal record and journal playback hooks. The hook procedures for those two hooks must reside in the thread that installs the hooks. The reason why it must be so is that: both hooks deal with the low-level interception of hardware input events. The input events must be recorded/playbacked in the order they appeared. If the code of those two hooks is in a DLL, the input events may scatter among several threads and it is impossible to know the order of them. So the solution: the hook procedure of those two hooks must be in a single thread only i.e. the thread that installs the hooks.  

在你有效的使用一个钩子之前，你必须理解一个钩子是如何工作的。当你创建了一个钩子，相应的 windows 在内存中就创建了一数据结构，这个数据结构包含了关于钩子的相关信息，然后把这个钩子增加到已有的钩子链表中去。新的钩子添加在老钩子的前面。当一个事件发生时，如果你安装的是局部钩子，过滤函数在你的进程中被调用，所以这是相当简单的。但是，如果它是远程钩子，系统必须为这个钩子函数在其它进程的地址空间中注入代码。并且，只有当函数驻留在 `DLL` 中时系统才能这样做。因此，如果你想使用一个远程钩子，你的钩子函数必须驻留在一个 `DLL` 中。这条规则的两个例外：工作日志钩子和工作日志回放钩子。这两个钩子的钩子过程必须驻留在安装钩子的线程中。为什么必须是这样的原因是：这两个钩子处理的是底层硬件的输入事件监听。输入事件必须是已记录的或是回放的，事件的发生也是有顺序的。如果这两个钩子的代码在 `DLL` 中，输入事件可能会散布在几个线程中间，所以就不可能知道它们发生的正确顺序。故解决的办法是：这两个钩子的钩子过程必须被安装在单一的线程中，也就是安装钩子的那个线程中。

There are 14 types of hooks:  

钩子有 14 种类型：

- `WH_CALLWNDPROC` called when `SendMessage` is called  
  当 `SendMessage` 函数被调用时。
- `WH_CALLWNDPROCRET` called when `SendMessage` returns  
  当 `SendMessage` 函数返回时。
- `WH_GETMESSAGE` called when `GetMessage` or `PeekMessage` is called  
  当 `GetMessage` 或 `PeekMessage` 被调用时调用。
- `WH_KEYBOARD` called when `GetMessage` or `PeekMessage` retrieves `WM_KEYUP` or `WM_KEYDOWN` from the message queue  
  当 `GetMessage` 或 `PeekMessage` 函数从消息队列中获得 `WM_KEYUP` 或 `WM_KEYDOWN` 消息时。
- `WH_MOUSE` called when `GetMessage` or `PeekMessage` retrieves a mouse message from the message queue  
  当 `GetMessage` 或 `PeekMessage` 从消息队列中获得鼠标消息时。
- `WH_HARDWARE` called when `GetMessage` or `PeekMessage` retrieves some hardware message that is not related to keyboard or mouse.  
  当 `GetMessage` 或 `PeekMessage` 获得一些和键盘或鼠标不相关的硬件消息时。 
- `WH_MSGFILTER` called when a dialog box, menu or scrollbar is about to process a message. This hook is local. It's specifically for those objects which have their own internal message loops.  
  当对话框、菜单或滚动条要处理一个消息时。该钩子是局部的。它是特别为那些自己内部有消息循环的控件设计的。
- `WH_SYSMSGFILTER` same as `WH_MSGFILTER` but system-wide  
  和 `WH_MSGFILTER` 一样，只不过是系统范畴的
- `WH_JOURNALRECORD` called when Windows retrieves message from the hardware input queue  
  当 windows 从硬件输入队列中获得消息时。
- `WH_JOURNALPLAYBACK` called when an event is requested from the system's hardware input queue.  
  当一个事件从系统硬件输入队列中被请求时。
- `WH_SHELL` called when something interesting about the shell occurs such as when the task bar needs to redraw its button.  
  当一些有趣味的外壳事件发生时，例如，当任务栏需要重绘它的按钮时。
- `WH_CBT` used specifically for computer-based training (CBT).  
  基于计算机的训练(CBT)事件发生时 
- `WH_FOREGROUNDIDLE` used internally by Windows. Little use for general applications  
  由 WINDOWS 内部自己使用，一般的应用程序很少使用 
- `WH_DEBUG` used to debug the hooking procedure  
  用于钩子函数的除错或调试。

## Installation and Uninstallation

Now that we know some theory, we can move on to how to install/uninstall the hooks.  

现在我们知道了一些理论，我们来学习如何安装和卸载一个钩子。

To install a hook, you call `SetWindowsHookEx` which has the following syntax:  

要安装一个钩子，调用 `SetWindowsHookEx` 函数，该函数句法如下：

```
SetWindowsHookEx proto HookType:DWORD, pHookProc:DWORD, hInstance:DWORD, ThreadID:DWORD
```

- `HookType` is one of the values listed above, e.g., `WH_MOUSE`, `WH_KEYBOARD`  
  一个上面列出的值。举例来说，`WH_MOUSE` `WH_KEYBOARD` 
- `pHookProc` is the address of the hook procedure that will be called to process the messages for the specified hook. If the hook is a remote one, it must reside in a DLL. If not, it must be in your process.  
  钩子函数的地址，它将被调用来为指定的钩子处理消息。如果是一远程钩子，它必须驻留在 `DLL` 中。否则，它必须在你的进程中。
- `hInstance` is the instance handle of the DLL in which the hook procedure resides. If the hook is a local one, this value must be NULL  
  钩子过程驻留的 `DLL` 文件的实例句柄。如果是局部钩子，这个值必须为空。
- `ThreadID` is the ID of the thread you want to install the hook to spy on. This parameter is the one that determines whether a hook is local or remote. If this parameter is NULL, Windows will interpret the hook as a system-wide remote hook that affects all threads in the system. If you specify the thread ID of a thread in your own process, this hook is a local one. If you specify the thread ID from other process, the hook is a thread-specific remote one. There are two exceptions to this rule: `WH_JOURNALRECORD` and `WH_JOURNALPLAYBACK` are always local system-wide hooks that are not required to be in a DLL. And `WH_SYSMSGFILTER` is always a system-wide remote hook. Actually it is identical to `WH_MSGFILTER` hook with `ThreadID==0`.  
  `ThreadID` 是你想要安装钩子以监视的线程的 `ID`。此参数决定了钩子是局部还是远程。如果此参数为 `NULL`，Windows 将把该钩子解释为影响系统中所有线程的系统级远程钩子。如果你指定自己进程中某线程的线程 `ID`，则该钩子是局部的。如果你指定其他进程中的线程 `ID`，则该钩子是线程特定的远程钩子。此规则有两个例外：`WH_JOURNALRECORD` 和 `WH_JOURNALPLAYBACK` 始终是无需驻留在 `DLL` 中的局部系统级钩子。而 `WH_SYSMSGFILTER` 始终是系统级远程钩子。实际上，它与 `ThreadID==0` 的 `WH_MSGFILTER` 钩子相同。  



## 钩子参数说明

你想安装钩子监视的线程 ID 号。无论钩子是局部的还是远程的，这个参数都是确定的值。如果这个参数为空，`windows` 将把这个钩子解释为系统范畴的远程钩子，它将影响系统中的所有线程。如果你在自己的进程中指定了一线程的 ID 号，这个钩子是局部钩子。如果你指定的事其它进程的线程 ID 号，这个钩子就是为线程指定的远程钩子。这个规则有两种特殊情况：`WH_JOURNALRECORD` 和 `WH_JOURNALPLAYBACK` 总是局部系统范畴的钩子，它们不需要在 `DLL` 文件中。`WH_SYSMSGFILTER` 总是一个系统范围内的远程钩子。其实它和 `WH_MSGFILTER` 钩子类似，如果把参数 `ThreadID` 设成 `0` 的话，它们就完全一样了。

### 返回值与卸载

If the call is successful, it returns the hook handle in `eax`. If not, NULL is returned. You must save the hook handle for unhooking later.

You can uninstall a hook by calling `UnhookWindowsHookEx` which accepts only one parameter, the handle of the hook you want to uninstall. If the call succeeds, it returns a non-zero value in `eax`. Otherwise, it returns `NULL`.

Now that you know how to install/uninstall hooks, we can examine the hook procedure.

如果这个调用是成功的，它返回以钩子句柄在 `eax` 中。如果不成功，`NULL` 被返回。为了稍候能脱钩（卸载钩子），你必须保存这个钩子句柄。

通过调用 `UnhookWindowsHookEx` 函数，你可以卸载一个钩子。这个函数仅接收一个参数，就是你想卸载的钩子句柄。如果调用成功，它在 `eax` 中返回一个非零值，否则，它返回 `NULL`。

现在，你知道了如何安装和卸载一个钩子，我们来分析钩子过程。

### 钩子过程原型

The hook procedure will be called whenever an event that is associated with the type of hook you have installed occurs. For example, if you install `WH_MOUSE` hook, when a mouse event occurs, your hook procedure will be called. Regardless of the type of hook you installed, the hook procedure always has the following prototype:

```
HookProc proto nCode:DWORD, wParam:DWORD, lParam:DWORD
```

只要一个和你安装的钩子类型相关联的事件发生，钩子函数就被调用。例如，如果你安装的是 `WM_MOUSE` 钩子，当一个鼠标事件发生时，你的钩子过程将被调用。不管你安装的钩子是哪一类型，钩子过程总是下面这种原型：

*   `nCode` specifies the hook code
    *   指定钩子代码
*   `wParam` and `lParam` contain additional information about the event
    *   `wParam` 和 `lParam` 包含事件的附加信息。

`HookProc` is actually a placeholder for the function name. You can name it anything you like so long as it has the above prototype. The interpretation of `nCode`, `wParam` and `lParam` is dependent on the type of hook you install. So as the return value from the hook procedure. For example:

`HookProc` 实际是一个函数名的占位符。你能用任何你喜欢的字符来命名它，只要它有上面的原型。`nCode` ，`wParam` 和 `lParam` 参数的解释依赖于你安装的钩子类型。条件是，从钩子过程返回的值。例如：

#### 示例：WH_CALLWNDPROC

`WH_CALLWNDPROC`

*   `nCode` can be only `HC_ACTION` which means there is a message sent to a window
    *   只能是 `HC_ACTION` ,意味着这是一个发送给窗口的消息。
*   `wParam` contains the message being sent, if it's not zero
    *   如果它不为零，代表正被发送的消息。
*   `lParam` points to a `CWPSTRUCT` structure
    *   指向 `CWPSTRUCT` 结构的指针。
*   return value: not used, return zero
    *   返回值：返回零，不使用。

#### 示例：WH_MOUSE

`WH_MOUSE`

*   `nCode` can be `HC_ACTION` or `HC_NOREMOVE`
    *   可以是 `HC_ACTION` 或 `HC_NOREMOVE`
*   `wParam` contains the mouse message
    *   包含鼠标消息
*   `lParam` points to a `MOUSEHOOKSTRUCT` structure
    *   包含一个 `MOUSEHOOKSTRUCT` 结构的指针。
*   return value: zero if the message should be processed. 1 if the message should be discarded.
    *   返回值：`0` 这个消息应该被处理，`1` 这个消息应该被抛弃。

The bottom line is: you must consult your `win32 api` reference for details about the meanings of the parameters and return value of the hook you want to install.

概要是：你必须参考 `win32API` 手册来获得关于你想安装的钩子的 参数 和返回值的详细资料

## 钩子链表与 CallNextHookEx

Now there is a little catch about the hook procedure. Remember that the hooks are chained in a linked list with the most recently installed hook at the head of the list. When an event occurs, `Windows` will call only the first hook in the chain. It's your hook procedure's responsibility to call the next hook in the chain. You can choose not to call the next hook but you'd better know what you're doing. Most of the time, it's a good practice to call the next procedure so other hooks can have a shot at the event. You can call the next hook by calling `CallNextHookEx` which has the following prototype:

现在，这里还有一个关于钩子过程的小问题。记得钩子被插入到一链表中，最近来的钩子安装在表头。当一个事件发生时，`windows` 将调用链的第一个钩子。所以你的钩子过程有责任调用下一个在链表中的钩子。你能选择不调用下一个钩子但是你最好知道你正在做什么。大多数时候，调用下一个过程以便让其它钩子能试着去处理这事件是一个不错的习惯。

你能通过调用 `CallNextHookEx` 函数来调用下一个钩子，这个函数原型如下：

```
CallNextHookEx proto hHook:DWORD, nCode:DWORD, wParam:DWORD, lParam:DWORD
```

*   `hHook` is your own hook handle. The function uses this handle to traverse the linked list and search for the hook procedure it should call next.
    *   `hHook` 是你拥有的钩子句柄。这个函数用这个句柄在链表上搜索它下一次应该调用的钩子过程。
*   `nCode`, `wParam` and `lParam` you can just pass those three values you receive from `Windows` to `CallNextHookEx`.
    *   `nCode` `wParam` 和 `lParam` 您只要把传入的参数简单传给 `CallNextHookEx` 即可

## 远程钩子与共享数据段

An important note about remote hooks: the hook procedure must reside in a `DLL` which will be mapped into other processes. When `Windows` maps the `DLL` into other processes, it will not map the data section(s) into the other processes. In short, all processes share a single copy of code but they will have their own private copy of the `DLL`'s data section!

This can be a big surprise to the unwary. You may think that when you store a value into a variable in the data section of a `DLL`, that value will be shared among all processes that load the `DLL` into their process address space. It's simply not true. In normal situation, this behavior is desirable since it provides the illusion that each process has its own copy of the `DLL`. But not when `Windows` hook is concerned. We want the `DLL` to be identical in all processes, including the data. The solution: you must mark the data section as shared. You can do this by specifying the section(s) attribute in the linker switch. For `MASM`, you need to use this switch:

重点注意远程钩子：远程钩子必须驻留在 `DLL` 文件中，这个 `DLL` 文件被映射到其它进程空间中。当 `windows` 映射这个 `DLL` 到其它进程空间中时，它的数据段不会被映射。简而言之，所有的进程共享单一的一份 `dll` 代码，但是它们自己都拥有 `dll` 的数据段节区的单独拷贝。这是一个很容易被忽视的问题。你可能会想，当你在 `dll` 文件的数据节区中用一个变量储存一个值时，这个值将被已加载 `dll` 文件到它进程地址空间中的所有进程共享。它是简单的但不正确。在通常情况下，这种行为是称心如意的，因为每一个映射该 `DLL` 的进程都有自己的数据段拷贝。

但是，当涉及到 `windows` 钩子时，却并非如此。我们希望 `dll` 对所有的进程都是相同的，包含数据段。解决方案是：你必须标记数据节区为共享段。你能在连接的时候用连接开关指定这个节区的属性。对于 `masm`，你可以用如下开关：

`/SECTION:<section name>, S`

The name of the initialized data section is `.data` and the uninitialized data is `.bss`. For example if you want to assemble a `DLL` which contains a hook procedure and you want the uninitialized data section to be shared among processes, you must use the following line:

已初始化的数据段的名字是 `.DATA` 未初始化的数据是 `.bss`。例如，如果你想汇编一个包含钩子过程的 `DLL` 文件，并且你想未初始化数据段在所有进程间共享，你必须用下面这行：

`link /section:.bss,S /DLL /SUBSYSTEM:WINDOWS ..........`

`S` attribute marks the section as shared.

Ｓ 属性使这个节区被共享。

## 示例代码

Example:

例子：

There are two modules: one is the main program which will do the GUI part and the other is the DLL that will install/uninstall the hook.

这里有两个模块：一个是 GUI 部分的主程序，一个是安装和卸载钩子的 dll 文件。

`;--------------------------------------------- This is the source code of the main program --------------------------------------`

这是主程序的源代码。

```
.386

.model flat,stdcall

option casemap:none

include \masm32\include\windows.inc

include \masm32\include\user32.inc

include \masm32\include\kernel32.inc

include mousehook.inc

includelib mousehook.lib

includelib \masm32\lib\user32.lib

includelib \masm32\lib\kernel32.lib

wsprintfA proto C :DWORD,:DWORD,:VARARG

wsprintf TEXTEQU <wsprintfA>

.const

IDD_MAINDLG               equ 101

IDC_CLASSNAME             equ 1000

IDC_HANDLE                equ 1001

IDC_WNDPROC               equ 1002

IDC_HOOK                  equ 1004

IDC_EXIT                  equ 1005

WM_MOUSEHOOK              equ WM_USER+6

DlgFunc PROTO :DWORD,:DWORD,:DWORD,:DWORD

.data

HookFlag dd FALSE

HookText db "&Hook",0

UnhookText db "&Unhook",0

template db "%lx",0

.data?

hInstance dd ?

hHook dd ?

.code

start:

    invoke GetModuleHandle,NULL

    mov hInstance,eax

    invoke DialogBoxParam,hInstance,IDD_MAINDLG,NULL,addr DlgFunc,NULL

    invoke ExitProcess,NULL

DlgFunc proc hDlg:DWORD,uMsg:DWORD,wParam:DWORD,lParam:DWORD

    LOCAL hLib:DWORD

    LOCAL buffer[128]:byte

    LOCAL buffer1[128]:byte

    LOCAL rect:RECT

    .if uMsg==WM_CLOSE

        .if HookFlag==TRUE

            invoke UninstallHook
```



```
    .endif

    invoke EndDialog,hDlg,NULL

.elseif uMsg==WM_INITDIALOG

    invoke GetWindowRect,hDlg,addr rect
    invoke SetWindowPos, hDlg, HWND_TOPMOST, rect.left, rect.top, rect.right, rect.bottom, SWP_SHOWWINDOW

.elseif uMsg==WM_MOUSEHOOK

    invoke GetDlgItemText,hDlg,IDC_HANDLE,addr buffer1,128
    invoke wsprintf,addr buffer,addr template,wParam
    invoke lstrcmpi,addr buffer,addr buffer1
    .if eax!=0
        invoke SetDlgItemText,hDlg,IDC_HANDLE,addr buffer
    .endif

    invoke GetDlgItemText,hDlg,IDC_CLASSNAME,addr buffer1,128
    invoke GetClassName,wParam,addr buffer,128
    invoke lstrcmpi,addr buffer,addr buffer1
    .if eax!=0
        invoke SetDlgItemText,hDlg,IDC_CLASSNAME,addr buffer
    .endif

    invoke GetDlgItemText,hDlg,IDC_WNDPROC,addr buffer1,128
    invoke GetClassLong,wParam,GCL_WNDPROC
    invoke wsprintf,addr buffer,addr template,eax
    invoke lstrcmpi,addr buffer,addr buffer1
    .if eax!=0
        invoke SetDlgItemText,hDlg,IDC_WNDPROC,addr buffer
    .endif

.elseif uMsg==WM_COMMAND

    .if lParam!=0
        mov eax,wParam
        mov edx,eax
        shr edx,16
        .if dx==BN_CLICKED
            .if ax==IDC_EXIT
                invoke SendMessage,hDlg,WM_CLOSE,0,0
            .else
                .if HookFlag==FALSE
                    invoke InstallHook,hDlg
                    .if eax!=NULL
                        mov HookFlag,TRUE
                        invoke SetDlgItemText,hDlg,IDC_HOOK,addr UnhookText
                    .endif
                .else
                    invoke UninstallHook
                    invoke SetDlgItemText,hDlg,IDC_HOOK,addr HookText
                    mov HookFlag,FALSE
                    invoke SetDlgItemText,hDlg,IDC_CLASSNAME,NULL
                    invoke SetDlgItemText,hDlg,IDC_HANDLE,NULL
                    invoke SetDlgItemText,hDlg,IDC_WNDPROC,NULL
                .endif
            .endif
        .endif
    .endif

.else

    mov eax,FALSE
    ret

.endif

mov eax,TRUE
ret

DlgFunc endp

end start
```

### DLL 源代码

这是 `dll` 文件的源代码

```
.386
.model flat,stdcall
option casemap:none
include \masm32\include\windows.inc
include \masm32\include\kernel32.inc
includelib \masm32\lib\kernel32.lib
include \masm32\include\user32.inc
includelib \masm32\lib\user32.lib

.const
WM_MOUSEHOOK equ WM_USER+6

.data
hInstance dd 0

.data?
hHook dd ?
hWnd dd ?

.code

DllEntry proc hInst:HINSTANCE, reason:DWORD, reserved1:DWORD
    .if reason==DLL_PROCESS_ATTACH
        push hInst
        pop hInstance
    .endif
    mov eax,TRUE
    ret
DllEntry Endp

MouseProc proc nCode:DWORD,wParam:DWORD,lParam:DWORD
    invoke CallNextHookEx,hHook,nCode,wParam,lParam
    mov edx,lParam
    assume edx:PTR MOUSEHOOKSTRUCT
    invoke WindowFromPoint,[edx].pt.x,[edx].pt.y
    invoke PostMessage,hWnd,WM_MOUSEHOOK,eax,0
    assume edx:nothing
    xor eax,eax
    ret
MouseProc endp

InstallHook proc hwnd:DWORD
    push hwnd
    pop hWnd
    invoke SetWindowsHookEx,WH_MOUSE,addr MouseProc,hInstance,NULL
    mov hHook,eax
    ret
InstallHook endp

UninstallHook proc
    invoke UnhookWindowsHookEx,hHook
    ret
UninstallHook endp

End DllEntry
```

### DLL Makefile

```
NAME=mousehook

$(NAME).dll: $(NAME).obj
    Link /SECTION:.bss,S /DLL /DEF:$(NAME).def /SUBSYSTEM:WINDOWS /LIBPATH:c:\masm\lib $(NAME).obj

$(NAME).obj: $(NAME).asm
    ml /c /coff /Cp $(NAME).asm
```

### 分析

这个例子将显示一个对话框，该对话框包含三个编辑框控件，这些控件将填充为与当前鼠标光标所在窗口相关联的类名、窗口句柄和窗口过程的地址。这里还有两个按钮，`HOOK` 和 `EXIT`。当你按下 `HOOK` 按钮时，程序会钩住鼠标输入，并将按钮上的文本更改为 `Unhook`。当你在某个窗口上移动鼠标光标时，该窗口的信息将显示在示例主窗口中。当你按下 `Unhook` 按钮时，程序将移除鼠标钩子。

主程序使用对话框作为其主窗口。它定义了一个自定义消息 `WM_MOUSEHOOK`，用于主程序和钩子 `dll` 之间通信。当主程序接收到此消息时，`wParam` 包含鼠标光标所在的窗口句柄。当然，这是一种任意的安排。出于简化目的，我决定在 `wParam` 中传递句柄。你可以选择主程序和钩子 `dll` 之间通信的自定义方法。

```
                .if HookFlag==FALSE
                    invoke InstallHook,hDlg
                    .if eax!=NULL
                        mov HookFlag,TRUE
                        invoke SetDlgItemText,hDlg,IDC_HOOK,addr UnhookText
                    .endif
```

该程序维护一个标志 `HookFlag`，用于监控钩子的状态。如果钩子未安装，其值为 `FALSE`；如果钩子已安装，其值为 `TRUE`。

当用户按下 `Hook` 按钮时，程序会检查钩子是否已安装。如果未安装，它会调用钩子 `dll` 中的 `InstallHook` 函数来安装它。注意，我们将主对话框的句柄作为参数传递给该函数，以便钩子 `dll` 能够将 `WM_MOUSEHOOK` 消息发送到正确的窗口，即我们自己的窗口。

当程序加载时，钩子 `dll` 也会随之加载。实际上，`dlls` 文件在程序进入内存后会立即加载。`dll` 的入口点函数甚至在主程序的第一条指令执行之前就会被调用。因此，当主程序执行时，`dll` 文件已经被初始化。我们将以下代码放置在钩子 `dll` 的入口点函数中：

```
    .if reason==DLL_PROCESS_ATTACH
        push hInst
        pop hInstance
    .endif
```

该代码仅将钩子 `dll` 自身的实例句柄保存到一个名为 `hInstance` 的全局变量中，以便在 `InstallHook` 函数中使用。由于 `dll` 的入口点函数在 `dll` 中其他函数被调用之前被调用，`hInstance` 始终有效。我们将 `hInstance` 放在 `.data` 段中，以便该值基于进程级别保持。



# 钩子 DLL 加载机制说明

当鼠标光标悬停在窗口上时，钩子 DLL 会被映射到进程中。假设已经有一个 DLL 占用了钩子 DLL 的预期加载地址，那么钩子 DLL 将被重新映射到其他地址。`hInstance` 的值将更新为新加载地址的值。当用户按下 `Unhook` 按钮然后按下 `Hook` 按钮时，`SetWindowsHookEx` 将被再次调用。然而，这一次，它将使用新的加载地址作为实例句柄，这在示例进程中是错误的，因为钩子 DLL 的加载地址并没有改变。这将导致钩子成为局部钩子，你只能钩取发生在你自己窗口中的鼠标事件。这几乎不是我们想要的结果。

为了在 `InstallHook` 函数中使用，该钩子 DLL 只是将其实例句柄保存在名为 `hInstance` 的全局变量中。因为在 DLL 中的其他函数被调用之前，DLL 的入口函数就会被调用，所以 `hInstance` 总是有效的。我们将 `hInstance` 放置在数据节区中，使得每个进程都有该变量的一个值。因为当鼠标光标停留在一个窗口上时，钩子 DLL 被映射进进程的地址空间中。设想一下，钩子 DLL 加载的地址空间已经被另一个 DLL 占用，钩子 DLL 将被重新映射在其它的地址空间。`hInstance` 的值将被更新成那些新加载的地址。当用户按下 `Unhook` 按钮或者是 `Hook` 按钮，`SetWindowsHookEx` 将被再一次调用。然而，这一次，它将用新加载的地址作为实例句柄，因为在例子进程中，这个钩子 DLL 的地址空间没有被改变，故新加载的地址是错误的。这个钩子是局部变量的一种，它仅能钩挂发生在你窗口中的鼠标事件。这是很难让人满意的。

```
InstallHook proc hwnd:DWORD

    push hwnd
    pop hWnd

    invoke SetWindowsHookEx,WH_MOUSE,addr MouseProc,hInstance,NULL

    mov hHook,eax

    ret

InstallHook endp
```

`InstallHook` 函数本身是非常简单的。它把作为它的参数传递过来的窗口句柄保存在全局变量 `hWnd` 中，以备将来使用。然后调用 `SetWindowsHookEx` 函数来安装一个鼠标钩子。为了让 `UnhookWindowsHookEx` 将来使用，`SetWindowsHookEx` 函数的返回值被储存在一个名为 `hHook` 的全局变量中。

在 `SetWindowsHookEx` 被调用之后，鼠标钩子就工作了。只要在系统中有鼠标事件发生，`MouseProc`（你的钩子过程）就被调用。

```
MouseProc proc nCode:DWORD,wParam:DWORD,lParam:DWORD

    invoke CallNextHookEx,hHook,nCode,wParam,lParam

    mov edx,lParam

    assume edx:PTR MOUSEHOOKSTRUCT

    invoke WindowFromPoint,[edx].pt.x,[edx].pt.y

    invoke PostMessage,hWnd,WM_MOUSEHOOK,eax,0

    assume edx:nothing

    xor eax,eax

    ret

MouseProc endp
```

首先，钩子函数要调用 `CallNextHookEx` 来让其它钩子有机会处理鼠标事件。在这之后，调用 `WindowFromPoint` 函数来得到给定屏幕坐标位置处的窗口句柄。注意，我们用 `lParam` 参数指向的 `MOUSEHOOKSTRUCT` 结构体指针中的 `POINT` 指针作为鼠标的当前坐标。在我们通过 `PostMessage` 发送窗口句柄和 `WM_MOUSEHOOK` 给主窗口之后。你应该记住的一件事是：你不能在钩子过程中 `SendMessage` 函数，它能引起消息死锁。我们建议你使用 `PostMessage`。`MOUSEHOOKSTRUCT` 结构体定义如下：

```
MOUSEHOOKSTRUCT STRUCT DWORD

  pt              POINT <>
  hwnd            DWORD     ?
  wHitTestCode    DWORD     ?
  dwExtraInfo     DWORD     ?

MOUSEHOOKSTRUCT ENDS
```

- `pt` 是鼠标光标当前屏幕坐标。
- `hwnd` 是将要接受鼠标消息的窗口句柄。它通常是在鼠标光标下的窗口，但不总都是。如果一个窗口调用 `SetCapture`，鼠标输入将被重定向到这个窗口。由于这个原因，我并不使用这个结构的 `hwnd` 成员但是我们用 `WindowFromPoint` 调用代替。
- `wHitTestCode` 指定 hittest 值。这个值给了很多关于当前鼠标光标位置的信息。它指出鼠标光标在窗口的什么部位。为了完成列表，请你查看 win32api 手册中的 `WM_NCHITTEST` 消息。
- `dwExtraInfo` 包含和消息有关的额外信息。通常这个值被调用的鼠标事件设置，还可以调用 `GetMessageExtraInfo` 获得。

当主窗口接受到 `WM_MOUSEHOOK` 消息时，它用在 `wParam` 参数中的窗口句柄来检索关于窗口的信息。

```
    .elseif uMsg==WM_MOUSEHOOK
        invoke GetDlgItemText,hDlg,IDC_HANDLE,addr buffer1,128
        invoke wsprintf,addr buffer,addr template,wParam
        invoke lstrcmpi,addr buffer,addr buffer1
        .if eax!=0
            invoke SetDlgItemText,hDlg,IDC_HANDLE,addr buffer
        .endif
        invoke GetDlgItemText,hDlg,IDC_CLASSNAME,addr buffer1,128
        invoke GetClassName,wParam,addr buffer,128
        invoke lstrcmpi,addr buffer,addr buffer1
        .if eax!=0
            invoke SetDlgItemText,hDlg,IDC_CLASSNAME,addr buffer
        .endif
        invoke GetDlgItemText,hDlg,IDC_WNDPROC,addr buffer1,128
        invoke GetClassLong,wParam,GCL_WNDPROC
        invoke wsprintf,addr buffer,addr template,eax
        invoke lstrcmpi,addr buffer,addr buffer1
        .if eax!=0
            invoke SetDlgItemText,hDlg,IDC_WNDPROC,addr buffer
        .endif
```

为了避免闪烁，我们检查已经在编辑控件中的文本和我们将要显示的文本是否相同。如果相同，我们就忽略它们。

我们调用 `GetClassName` 函数来获取类名，我们传入 `GCL_WNDPROC` 然后调用 `GetClassLong` 函数来获得窗口过程的地址，然后格式化字符串并把它们放入到合适的编辑控件中。

```
            invoke UninstallHook
            invoke SetDlgItemText,hDlg,IDC_HOOK,addr HookText
            mov HookFlag,FALSE
            invoke SetDlgItemText,hDlg,IDC_CLASSNAME,NULL
            invoke SetDlgItemText,hDlg,IDC_HANDLE,NULL
            invoke SetDlgItemText,hDlg,IDC_WNDPROC,NULL
```

当用户按下 `Unhook` 按钮时，程序调用在钩子 DLL 中的 `UninstallHook` 函数，`UninstallHook` 仅仅是调用 `UnhookWindowsHookEx`。在这之后，他将按钮的文本改为 `Hook`，并置 `HookFlag` 标志为 `FALSE` 然后清除编辑控件中的内容。

注意：makefile 中的连接标志如下：

```
        Link /SECTION:.bss,S  /DLL /DEF:$(NAME).def /SUBSYSTEM:WINDOWS
```

它指定 `.bss` 段作为一个共享段以便让所有的进程共享同一个在 Hook 动态链接库中的未初始化数据段。没有了这个开关，你的钩子 DLL 将不能正确的工作了。

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

This article come from Iczelion's asm page

风向改变翻译于 2008-3-1

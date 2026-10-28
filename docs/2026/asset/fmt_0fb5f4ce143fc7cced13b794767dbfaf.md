
# Tutorial 24: Windows Hooks

## 第二十四课：win32 asm windows 钩子

---

We will learn about Windows hooks in this tutorial. Windows hooks are very powerful. With them, you can poke inside other processes and sometimes alter their behaviors.

我们将在这一课中学习 windows钩子。Windows钩子是非常有用的。用它们，你能截获其它进程并且还可以改变它们的行为。

Download the example here.

### Theory

Windows hooks can be considered one of the most powerful features of Windows. With them, you can trap events that will occur, either in your own process or in other processes. By "hooking", you tell Windows about a function, filter function also called hook procedure, that will be called everytime an event you're interested in occurs. There are two types of them: local and remote hooks.

Windows 钩子被认为是windows中强有力的特征之一。用它们，你可以在你自己的进程或是其它进程中捕获将要发生的事件。通过“挂钩”你告诉windows关于一个过滤函数的信息，过滤函数也叫钩子过程，每当你感兴趣的事件发生时，这个函数将被调用。钩子有两种类型：局部钩子和远程钩子。

- Local hooks trap events that will occur in your own process.

  局部钩子捕获的是在你自己的进程中将要发生的事件。

- Remote hooks trap events that will occur in other process(es). There are two types of remote hooks:

  远程钩子捕获的是在其它进程中将要发生的事件。远程钩子的类型也有两种：

  - thread-specific traps events that will occur in a specific thread in other process. In short, you want to observe events in a specific thread in a specific process.

    基于线程的捕获的事件是在其它进程中特殊线程将要发生的事件。简而言之，你想要观察的事件发生在一个具体的线程中，而这个线程是一特殊进程的线程。

  - system-wide traps all events destined for all threads in all processes in the system.

    系统范畴的捕获系统中所有进程的全部线程将要发生的事件。（捕捉系统中所有进程将发生的事件消息。）

When you install hooks, remember that they affect system performance. System-wide hooks are the most notorious. Since ALL related events will be routed through your filter function, your system may slow down noticeably. So if you use a system-wide hook, you should use it judiciously and unhook it as soon as you don't need it. Also, you have a higher chance of crashing the other processes since you can meddle with other processes and if something is wrong in your filter function, it can pull the other processes down to oblivion with it. Remember: Power comes with responsibility.

当你安装钩子时，铭记它们带来的后果是影响系统性能。系统范畴的钩子大多数是臭名昭著的。因为所有相关事件都将被你的过滤函数过滤，你的系统可能会变的很慢。所以，如果你用系统范畴的钩子，你应该明智的用它并且一旦你不需要它们时就让它们脱钩。同样，你也有较高的机会让其它进程发生异常，因为你能干涉其它进程并且如果你的钩子函数出了问题，它能拉倒其它进程直到被湮没。铭记一点：性能和责任一起供给。能力伴随责任（功能强大也意味着使用时要负责任。）

You have to understand how a hook works before you can use it efficiently. When you create a hook, Windows creates a data structure in memory, containing information about the hook, and adds it to a linked list of existing hooks. New hook is added in front of old hooks. When an event occurs, if you install a local hook, the filter function in your process is called so it's rather straightforward. But if it's a remote hook, the system must inject the code for the hook procedure into the address space(s) of the other process(es). And the system can do that only if the function resides in a DLL. Thus, if you want to use a remote hook, your hook procedure must reside in a DLL. There is two exceptions to this rule: journal record and journal playback hooks. The hook procedures for those two hooks must reside in the thread that installs the hooks. The reason why it must be so is that: both hooks deal with the low-level interception of hardware input events. The input events must be recorded/playbacked in the order they appeared. If the code of those two hooks is in a DLL, the input events may scatter among several threads and it is impossible to know the order of them. So the solution: the hook procedure of those two hooks must be in a single thread only i.e. the thread that installs the hooks.

在你有效的使用一个钩子之前，你必须理解一个钩子是如何工作的。当你创建了一个钩子，相应的windows在内存中就创建了一数据结构，这个数据结构包含了关于钩子的相关信息，然后把这个钩子增加到已有的钩子链表中去。新的钩子添加在老钩子的前面。当一个事件发生时，如果你安装的是局部钩子，过滤函数在你的进程中被调用，所以这是相当简单的。但是，如果它是远程钩子，系统必须为这个钩子函数在其它进程的地址空间中注入代码。并且，只有当函数驻留在DLL中时系统才能这样做。因此，如果你想使用一个远程钩子，你的钩子函数必须驻留在一个DLL中。这条规则的两个例外：工作日志钩子和工作日志回放钩子。这两个钩子的钩子过程必须驻留在安装钩子的线程中。为什么必须是这样的原因是：这两个钩子处理的是底层硬件的输入事件监听。输入事件必须是已记录的或是回放的，事件的发生也是有顺序的。如果这两个钩子的代码在DLL中，输入事件可能会散布在几个线程中间，所以就不可能知道它们发生的正确顺序。故解决的办法是：这两个钩子的钩子过程必须被安装在单一的线程中，也就是安装钩子的那个线程中。

There are 14 types of hooks:

钩子有14种类型：

- `WH_CALLWNDPROC`: called when `SendMessage` is called

  当`SendMessage`函数被调用时。

- `WH_CALLWNDPROCRET`: called when `SendMessage` returns

  当`SendMessage`函数返回时。

- `WH_GETMESSAGE`: called when `GetMessage` or `PeekMessage` is called

  当`GetMessage`或`PeekMessage`被调用时调用。

- `WH_KEYBOARD`: called when `GetMessage` or `PeekMessage` retrieves `WM_KEYUP` or `WM_KEYDOWN` from the message queue

  当`GetMessage`或`PeekMessage`函数从消息队列中获得`WM_KEYUP`或`WM_KEYDOWN`消息时。

- `WH_MOUSE`: called when `GetMessage` or `PeekMessage` retrieves a mouse message from the message queue

  当`GetMessage`或`PeekMessage`从消息队列中获得鼠标消息时。

- `WH_HARDWARE`: called when `GetMessage` or `PeekMessage` retrieves some hardware message that is not related to keyboard or mouse.

  当`GetMessage`或`PeekMessage`获得一些和键盘或鼠标不相关的硬件消息时。

- `WH_MSGFILTER`: called when a dialog box, menu or scrollbar is about to process a message. This hook is local. It's specifically for those objects which have their own internal message loops.

  当对话框、菜单或滚动条要处理一个消息时。该钩子是局部的。它是特别为那些自己内部有消息循环的控件设计的。

- `WH_SYSMSGFILTER`: same as `WH_MSGFILTER` but system-wide

  和`WH_MSGFILTER`一样，只不过是系统范畴的。

- `WH_JOURNALRECORD`: called when Windows retrieves message from the hardware input queue

  当windows从硬件输入队列中获得消息时。

- `WH_JOURNALPLAYBACK`: called when an event is requested from the system's hardware input queue.

  当一个事件从系统硬件输入队列中被请求时。

- `WH_SHELL`: called when something interesting about the shell occurs such as when the task bar needs to redraw its button.

  当一些有趣味的外壳事件发生时，例如，当任务栏需要重绘它的按钮时。

- `WH_CBT`: used specifically for computer-based training (CBT).

  基于计算机的训练(CBT)事件发生时。

- `WH_FOREGROUNDIDLE`: used internally by Windows. Little use for general applications

  由WINDOWS内部自己使用，一般的应用程序很少使用。

- `WH_DEBUG`: used to debug the hooking procedure

  用于钩子函数的除错或调试。

Now that we know some theory, we can move on to how to install/uninstall the hooks.

现在我们知道了一些理论，我们来学习如何安装和卸载一个钩子。

To install a hook, you call `SetWindowsHookEx` which has the following syntax:

要安装一个钩子，调用`SetWindowsHookEx`函数，该函数句法如下：

```
SetWindowsHookEx proto HookType:DWORD, pHookProc:DWORD, hInstance:DWORD, ThreadID:DWORD
```

- `HookType` is one of the values listed above, e.g., `WH_MOUSE`, `WH_KEYBOARD`

  一个上面列出的值。举例来说，`WH_MOUSE`, `WH_KEYBOARD`。

- `pHookProc` is the address of the hook procedure that will be called to process the messages for the specified hook. If the hook is a remote one, it must reside in a DLL. If not, it must be in your process.

  钩子函数的地址，它将被调用来为指定的钩子处理消息。如果是一远程钩子，它必须驻留在DLL中。否则，它必须在你的进程中。

- `hInstance` is the instance handle of the DLL in which the hook procedure resides. If the hook is a local one, this value must be `NULL`.

  钩子过程驻留的DLL文件的实例句柄。如果是局部钩子，这个值必须为空。


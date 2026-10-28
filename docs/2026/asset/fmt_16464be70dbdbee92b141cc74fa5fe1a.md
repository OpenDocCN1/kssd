
正如我们所述，这并不是对每种潜在技术的全面评估。另一种可能值得探索的技术是向量异常处理（Vectored Exception Handling，VEH）。其中两个例子是ethicalchaos在“进程内无补丁的AMSI绕过”和Countercept的“CallStackSpoofer”中提到的。

和内核回调一样，VEH是一个活跃的研究领域，其中有很多内容值得探索，远超过我们在这篇文章中的时间和篇幅所能涵盖的范围。

## 线程调用堆栈

进程的另一个被审查的组件是通过线程的调用堆栈（Call Stack）进行的。如《在WinDbg中查看调用堆栈》所述，调用堆栈定义如下：

> 调用堆栈是导致程序计数器当前位置的函数调用链。调用堆栈的顶部函数是当前函数，下一个函数是调用当前函数的函数，依此类推。显示的调用堆栈基于当前程序计数器，除非你更改寄存器上下文。有关如何更改寄存器上下文的更多信息，请参阅更改上下文。

由于调用堆栈可以帮助确定线程的意图，它经常受到审查以确定其有效性。在本节中，我们想展示如何使用调用堆栈来确定恶意行为（以一个简单的例子为例），然后讨论处理这种行为的攻击策略。

在这里，我们有一个名为Vulpes的植入程序：

![图片描述](upload/attach/202308/876202\_TMZJNS99CWPGJME.png)

如果我们查看进程（10792）的线程，我们会看到一堆以难以捉摸的TpReleaseCleanupGroupMembers开始的线程:

![图片描述](upload/attach/202308/876202\_WBRJVBABBMVPYQ4.png)

这在进程中非常常见，这里有一个chrome.exe的例子：

![图片描述](upload/attach/202308/876202\_68FTBN8KM8AMES4.png)

然后是`RuntimeBroker.exe`的例子：

![图片描述](upload/attach/202308/876202\_KP8QWGWTAANKDGT.png)

这是对攻击者来说一个重要的侧记。如果所讨论的植入程序依赖于伪装成其他东西，那么这一点需要考虑在内。例如，如果植入程序的运行基于Chrome等浏览器，那么HTTP请求的处理方式应该相同，并且应该模仿线程的入口点和调用堆栈。

回到Vulpes植入程序，调用堆栈主要是`TpReleaseCleanupGroupMembers`，这是正常的。然而，如果我们检查一些线程，下面是负责WinHTTP的线程：

![图片描述](upload/attach/202308/876202\_TGEK9QWRXDCAAJ6.png)

这里是由该进程启动的一个通用线程：

![图片描述](upload/attach/202308/876202\_Q5M3DBKTXQNHJNC.png)

有几个其他的线程，但我们将重点关注第二个例子，因为这是一个在许多进程中都会找到的线程的调用堆栈。让我们看看如何以编程方式读取线程堆栈，以及伪造的线程基址可能会显得可疑。

这是入口点：

```
int main()
{
DWORD dwProcessId = 10792;
DWORD dwSussThread = 1996;
DWORD dwNormalThread = 26084;
HANDLE hProcess = OpenProcess(PROCESS\_ALL\_ACCESS, FALSE, dwProcessId);
SymInitialize(hProcess, NULL, TRUE);
StackWalkThread(hProcess, dwSussThread);
SymCleanup(hProcess);
}
```

在上面的例子中，我们有两个线程ID：

- `1996`：伪造的线程
- `26084`：一个相对正常的堆栈

有了这些信息，我们需要编写一个函数来枚举线程的调用堆栈。可以使用以下代码来实现：

```
void StackWalkThread(HANDLE hProcess, DWORD dwThreadId)
{
STACKFRAME64 frame = { 0 };
CONTEXT context = { 0 };
int idx = 0;
HANDLE hThread = OpenThread(MAXIMUM\_ALLOWED, FALSE, dwThreadId);
if (!hThread) return;
context.ContextFlags = CONTEXT\_FULL;
if (GetThreadContext(hThread, &context) == FALSE) return;
frame.AddrPC.Offset = context.Rip;
frame.AddrPC.Mode = AddrModeFlat;
frame.AddrStack.Offset = context.Rsp;
frame.AddrStack.Mode = AddrModeFlat;
frame.AddrFrame.Offset = context.Rbp;
frame.AddrFrame.Mode = AddrModeFlat;
printf("# Thread: %ld\n\n", dwThreadId);
while (StackWalk64(IMAGE\_FILE\_MACHINE\_AMD64, hProcess, hThread, &frame, &context, NULL, SymFunctionTableAccess64, SymGetModuleBase64, NULL))
{
DWORD64 moduleBase = SymGetModuleBase64(hProcess, frame.AddrPC.Offset);
DWORD64 offset = 0;
char symbolBuff[sizeof(SYMBOL\_INFO) + MAX\_SYM\_NAME \* sizeof(TCHAR)] = { 0 };
PSYMBOL\_INFO symbol = (PSYMBOL\_INFO)symbolBuff;
symbol->SizeOfStruct = sizeof(SYMBOL\_INFO);
symbol->MaxNameLen = MAX\_SYM\_NAME;
if (SymFromAddr(hProcess, frame.AddrPC.Offset, &offset, symbol))
{
printf(
"\\\_ Frame %d\n"
" |\_ Name: %s\n"
" |\_ Address: 0x%p\n\n",
idx,
symbol->Name,
symbol->Address
);
idx++;
}
}
}
```

通过使用DbgHelp库，我们可以使用以下函数：

- `StackWalk64`：获取堆栈跟踪。

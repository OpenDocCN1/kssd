
此外，为提供对较早版本的活动模板库（ATL）框架的兼容性（7.1或更早版本），Windows内核提供了一个ATL thunk 模拟环境。该环境检测导致DEP异常的ATL thunk代码序列，并且模拟预期的操作。应用程序开发人员可以通过最新的Microsoft C++ 编译器，指定 `/NXCOMPAT` 标志（前文提到过，并且这将在PE文件头中设置 `IMAGE_DLLCHARACTERISTICS_NX_COMPAT` 标志），用于告知系统这个可执行文件完全支持DEP；如此就能够向内核请求不对该程序应用ATL thunk模拟环境。需要注意，如果BCD `nx` 值被设置为 `AlwaysOn`，将永久禁用ATL thunk模拟。

Finally, if the system is in `OptIn` or `OptOut` mode and executing a 32-bit process, the `SetProcessDEPPolicy` function allows a process to dynamically disable DEP or to permanently enable it. (Once enabled through this API, DEP cannot be disabled programmatically for the lifetime of the process.)

This function can also be used to dynamically disable ATL thunk emulation in case the image wasn’t compiled with the `/NXCOMPAT` flag. On 64-bit processes or systems booted with `AlwaysOff` or `AlwaysOn`, the function always returns a failure. The `GetProcessDEPPolicy` function returns the 32-bit per-process DEP policy (it fails on 64-bit systems, where the policy is always the same—enabled),

while `GetSystemDEPPolicy` can be used to return a value corresponding to the policies in Table 10-3.

最后，如果系统处于 `OptIn` 或 `OptOut` 模式（也就是启用动态DEP配置），并且正执行一个32位进程，函数 `SetProcessDEPPolicy` 允许该进程动态禁用DEP，或者永久启用DEP。（译注：请参考表10-3）（一旦通过此API启用，在该进程生命周期内，DEP无法通过编程方式禁用。）此函数也可用于动态禁用ATL thunk 模拟，防止没有使用 `/NXCOMPAT` 标志来编译生成映像文件。`SetProcessDEPPolicy()` 的原型声明在 `Winbase.h` 头文件中：

```
BOOL WINAPI SetProcessDEPPolicy(__in  DWORD dwFlags);
```

如果 `dwFlags` 的值为1，则在进程生命周期内永久启用DEP。

对于64位进程，或者系统以 `AlwaysOff`/`AlwaysOn` 选项启动，则该函数总是返回失败。函数 `GetProcessDEPPolicy` 返回每个32位进程的DEP策略（在64位系统上调用此函数会失败，因为此时DEP策略总是“启用”），而函数 `GetSystemDEPPolicy` 可用于返回（查询）与表10-3中列出策略相对应的值。

### Software Data Execution Prevention

For older processors that do not support hardware no execute protection, Windows supports limited software data execution prevention (DEP). One aspect of software DEP reduces exploits of the exception handling mechanism in Windows. (See Chapter 3 in Part 1 for a description of structured exception handling.) If the program’s image files are built with safe structured exception handling (a feature in the Microsoft Visual C++ compiler that is enabled with the `/SAFESEH` flag), before an exception is dispatched, the system verifies that the exception handler is registered in the function table (built by the compiler) located within the image file.

### 软件数据执行保护

对于不支持硬件不可执行保护的较早处理器，Windows也提供了有限的软件数据执行保护（DEP）。软件DEP的一个方面是减少对Windows中异常处理机制的漏洞利用可能性。（参见本书上册第3章对结构化异常处理的介绍。）如果程序的二进制映像文件通过安全的结构化异常处理（Microsoft Visual C++ 编译器中的一个功能，通过 `/SAFESEH` 标志启用）构建，在分发一个异常前，系统会验证该异常处理程序是否在位于映像文件内的函数表（由编译器构建）中注册。

（译注：下图给出在 Visual Studio 2010 中，对欲构建的PE文件，配置启用 `/SAFESEH` 标志的方法。）

![](https://bbs.kanxue.com/upload/attach/201509/533967_bb5747581f315391.jpg)

在 VS 2015 中，改选项的中文解释大致相同：

![](https://bbs.kanxue.com/upload/attach/201509/533967_06e225a872e82ae3.png)

The previous mechanism depends on the program’s image files being built with safe structured exception handling. If they are not, software DEP guards against overwrites of the structured exception handling chain on the stack in x86 processes via a mechanism known as Structured Exception Handler Overwrite Protection (SEHOP). A new symbolic exception registration record is added on the stack when a thread first begins user-mode execution. The normal exception registration chain will lead to this record. When an exception occurs, the exception dispatcher will first walk the list of exception handler registration records to ensure that the chain leads to this symbolic record. If it does not, the exception chain must have been corrupted (either accidentally or deliberately), and the exception dispatcher will simply terminate the process without calling any of the exception handlers described on the stack. Address Space Layout Randomization (ASLR) contributes to the robustness of this method by making it more difficult for attacking code to know the location of the function pointed to by the symbolic exception registration record, and so to construct a fake symbolic record of its own.

前述的机制依赖于程序的映像文件是否通过安全的结构化异常处理来构建。如果不是，软件DEP保护针对x86进程栈上的结构化异常处理链，阻止其被覆写，这是通过一个被称为Structured Exception Handler Overwrite Protection（结构化异常处理程序覆写保护，SEHOP）的机制完成的。

当一个线程首次开始在用户模式执行时，一个新的符号异常注册记录被加入到栈上。普通的异常注册链将指向（lead to）此记录。当发生一个异常时，异常分发器将首先遍历（walk）异常处理程序注册记录列表，确保该链连接到此符号记录。如果不是这样，该异常链必定已损坏（不是无意就是有意），此时异常分发器将仅仅终止进程，不会调用任何栈上描述的异常处理程序。地址空间布局随机化（ASLR）技术，通过符号异常注册记录让攻击代码更难于得知函数指向的位置，以及构造一个自身的假符号记录。这有助于此方法（译注：即异常分发器检查符号记录决定是否调用异常处理程序）的健壮性。

To further validate the SEH handler when `/SAFESEH` is not present, a mechanism called Image Dispatch Mitigation ensures that the SEH handler is located within the same image section as the function that raised an exception, which is normally the case for most programs (although not necessarily, since some DLLs might have exception handlers that were set up by the main executable, which is why this mitigation is off by default). Finally, Executable Dispatch Mitigation further makes sure that the SEH handler is located within an executable page—a less strong requirement than Image Dispatch Mitigation, but one with fewer compatibility issues.

为了在没有指定 `/SAFESEH` 标志时，进一步验证SEH处理程序，一个叫做Image Dispatch Mitigation（映像调度/分发缓解）的机制确保SEH处理程序位在与引发异常的函数相同的映像节中，对于多数程序而言通常是这样（尽管这不是必须的，因为某些DLLs可能包含由主要可执行文件设置的异常处理程序，这就是默认情况下，此缓解机制关闭的原因——`.exe` 中的函数引发异常，而SEH处理程序在其它DLL映像中）。最后，Executable Dispatch Mitigation（可执行调度/分发缓解）更进一步保证了SEH处理程序位于一个可执行页内——比映像调度/分发缓解更少的限制，更少的兼容性问题。

Two other methods for software DEP that the system implements are stack cookies and pointer encoding.

The first relies on the compiler to insert special code at the beginning and end of each potentially exploitable function. The code saves a special numerical value (the cookie) on the stack on entry and validates the cookie’s value before returning to the caller saved on the stack (which would have now been corrupted to point to a piece of malicious code). If the cookie value is mismatched, the application is terminated and not allowed to continue executing. The cookie value is computed for each boot when executing the first user-mode thread, and it is saved in the `KUSER_SHARED_DATA` structure. The image loader reads this value and initializes it when a process starts executing in user mode. (See Chapter 3 in Part 1 for more information on the shared data section and the image loader.)

系统实现的其余2种软件DEP方法是栈cookies与pointer encoding（指针编码）。第一种方法依赖于编译器在每个可能被漏洞利用的函数起始和结尾处插入特殊代码。（这些函数在内部可能调用了诸如 `strcpy()` 等不安全的字符串复制的 C运行时库函数，从而将函数栈上的缓冲区溢出为 shellcode），该代码在栈上入口处保存一个特殊的数值（cookie），然后在返回保存在栈上的调用者前，验证该cookie的值（译注：从编译器生成的汇编代码中可以看出，程序从数据段取出一个叫做 `__security_cookie` 的4字节双字，复制到 `EAX` 寄存器中，然后与 `EBP` 寄存器中的值，即当前栈底地址，进行 `XOR` 异或运算，再将结果值复制到栈上作为实际的 cookie，这个 cookie 在函数栈内的布局位于数组等本地缓冲区的前端，即靠近内存高址方向，这样缓冲区从内存低址方向往高址方向溢出，就会覆盖前面的 cookie，从而达到验证的目的。当被编译器在其栈上加入 cookie 的函数返回时，再次将 cookie 与栈底地址进行异或运算，然后调用例程 `__security_check_cookie()` 进行验证）。如果cookie值不匹配，表明发生溢出，应用程序被终止（`__security_check_cookie()` -> `report_failure()` -> `__security_error_handler()`），不允许继续执行。

每次系统启动，执行首个用户模式线程时，都计算该cookie值，并且保存在 `KUSER_SHARED_DATA` 结构体中。当一个进程开始在用户模式中执行时，映像加载器读取该值并对其进行初始化。（参见本书上册第3章有关共享数据节和映像加载器的更多内容）

（译注：栈cookies 通过设置 Visual C/C++ 编译器的 `/GS` 编译选项来实现，具体参考下图）

![](https://bbs.kanxue.com/upload/attach/201509/533967_3f684439390a9f83.jpg)

（要自行研究栈 cookies的内部机制也非常容易，只要以IDA等工具反汇编通过启用 `/GS` 编译选项构建的PE文件即可，另外，在源代码的开头添加如下指令，可以让编译器在绝大多数的例程中添加 cookie）

```
#pragma strict_gs_check(on)
```

The cookie value that is calculated is also saved for use with the `EncodeSystemPointer` and `DecodeSystemPointer` APIs, which implement pointer encoding. When an application or a DLL has static pointers that are dynamically called, it runs the risk of having malicious code overwrite the pointer values with code that the malware controls. By encoding all pointers with the cookie value and then decoding them, when malicious code sets a nonencoded pointer, the application will still

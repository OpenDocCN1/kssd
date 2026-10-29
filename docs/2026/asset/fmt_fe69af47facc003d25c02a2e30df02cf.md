
**注：** `lldb`（[`bd8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4D9k6r3u0Q4x3X3g2D9L8s2k6E0i4K6u0W2L8%4u0Y4i4K6u0r3N6i4y4W2i4K6u0r3N6s2g2@1L8%4u0A6j5h3I4Q4x3X3g2Z5N6r3#2D9`](<https://bbs.kanxue.com/elink@4a5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4D9k6r3u0Q4x3X3g2D9L8s2k6E0i4K6u0W2L8%4u0Y4i4K6u0r3N6i4y4W2i4K6u0r3N6s2g2@1L8%4u0A6j5h3I4Q4x3X3g2Z5N6r3#2D9i4K6t1#2c8f1k6Q4x3U0g2n7b7#2)9J5y4e0R3&6)>) 实际上是 Apple 系统（包括 macOS）的调试器。可通过命令行直接执行，也可集成到多种逆向工具中。（在本文中，我们使用了专注于 macOS 的反汇编器和反编译器 Hopper。）

退出代码 `45 (0x2d)` 相当独特，实际上相当说明问题。经验丰富的 macOS 恶意软件分析人员将识别出这个状态代码，因为 debuggee（在这里，恶意软件）调用 `ptrace` 系统调用（或 API）的结果，带有 `PT_DENY_ATTACH` 标志。

顾名思义，`PT_DENY_ATTACH` 标志指示操作系统阻止 debuggee 被调试。一旦发出 `ptrace` 系统调用，随后连接调试器的尝试将失败，或者如果进程已经被调试，它将提前终止，退出代码为 `45 (0x2d)`。

这个标志是一个非标准的 `ptrace` 请求类型，由苹果添加，因此只在其操作系统上得到支持。检查苹果的 `XNU` 源代码（`bsd/sys/ptrace.h`）显示 `PT_DENY_ATTACH` 标志的值为 `0x1F`。

当然，恶意软件肯定不愿被调试，因此 `GoSearch22` 实现这种反分析逻辑并不奇怪。不过幸运的是，通过跳过 `ptrace` 调用，绕过调试器中的这种反分析技术是相当简单的，因此它从一开始就不会执行。要做到这一点，我们需要定位恶意软件调用 `ptrace` 的位置。查看 `GoSearch22` 的反编译代码，会发现大量旨在使静态分析复杂化的花指令（如定位反分析逻辑）。例如，在恶意软件的入口点中发现的以下代码中，请注意无意义的嵌套条件检查以及对 `dlsym` 函数的虚假调用：

```text
r9 = 0x3f35713b;
...
r8 = r9;
if (r8 <= 0xb33cc16b) {
    if (r8 > 0x9fbc741a) {
        if (r8 > 0xa693fc1a) {
            if (r8 != 0xa693fc1b) {
                if (r8 != 0xb0d2dccd) {
                    ...
                    dlsym(dlopen(0x0, 0xa), 0x100076458);
                    dlsym(dlopen(0x0, 0xa), 0x100076440);
                    dlsym(dlopen(0x0, 0xa), 0x100076428);
```

注：有关此方案和其他类似混淆方案的更多详细信息，请参阅“Using LLVM to obfuscate your code during compilation”（[`094N6%4N6%4i4K6u0W2j5i4m8J5K9h3!0J5K9i4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6i4k6Q4x3X3c8T1L8r3!0Y4i4K6u0r3y4U0R3%4i4K6u0V1M7X3g2$3k6i4u0K6k6g2)9J5k6r3g2F1k6$3W2F1k6h3g2J5K9h3&6Y4i4K6u0V1L8r3I4$3L8g2)9J5k6r3!0T1k6Y4g2K6j5$3q4@1K9h3!0F1`](<https://bbs.kanxue.com/elink@2a5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4H3M7X3W2G2M7X3W2@1i4K6u0W2j5$3!0E0i4K6u0r3k6r3g2$3i4K6u0V1j5X3I4G2k6#2)9J5c8U0j5^5y4#2)9J5k6s2u0W2N6X3g2J5M7$3g2Q4x3X3c8W2L8X3N6A6L8X3g2W2M7X3W2F1k6#2)9J5k6r3I4D9N6X3#2Q4x3X3c8G2j5X3k6#2M7$3y4S2N6r3W2G2L8W2)9J5y4f1g2r3i4K6t1#2b7V1y4Q4x3U0f1^5z5b7`.`.)>)。此外，如下图，该恶意软件不包含对 `ptrace` 用户模式 API 的调用。

![](https://bbs.kanxue.com/upload/attach/202312/793745_KF9RNGR62Q8BNUA.webp)

很奇怪吗？其实并不特别，因为恶意软件很可能只是试图掩盖调试功能的调用。实现这一点的一个简单方法是通过直接调用 `ptrace` 系统调用（`SYS_ptrace`）实现的。参考系统调用名称到它们的系统调用号的映射，我们看到 `ptrace` 系统调用被分配为 `26 (0x1a)`。

```text
% less /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include/sys/syscall.h
...
#define SYS_ptrace 26
```

回顾调用系统调用的 arm64 汇编指令是 `svc`（supervisor call），我们可以使用反汇编器的搜索功能（⌘+F）来查找对该指令的调用，如图 12.8 所示。

![](https://bbs.kanxue.com/upload/attach/202312/793745_4WN798N5W8KXME3.webp)

如果找到 `svc` 指令，Hopper 将跳转到主反汇编窗口中的位置。Hopper 在 `0x00000001000541fc` 找到该指令的第一个实例：

```text
0x00000001000541e8 movz x0, #0x1a
0x00000001000541ec movz x1, #0x1f
0x00000001000541f0 movz x2, #0x0
0x00000001000541f4 movz x3, #0x0
...
0x00000001000541fc svc #0x80
0x0000000100054200 mov w11, #0x6b8f
```

首先，用 `0x1a` 初始化 `X0` 寄存器，这是 `ptrace` (`SYS_ptrace`) 的系统调用号。`x1` 寄存器设置为 `0x1f`，即 `PT_DENY_ATTACH` 的值。另外两个参数 `X2` 和 `X3` 被设置为零。然后在 `0x00000001000541fc` 处，通过 `SVC` 指令进行调用。

如前所述，使用 `PT_DENY_ATTACH` 调用 `SYS_ptrace` 试图阻止调试，或者，如果正在调试恶意软件，将导致恶意软件以退出码 `45 (0x2d)` 终止。

现在我们已经在调试会话中检测到反调试逻辑的位置，我们可以跳过调用。一种简单的方法是在 `SVC` 指令上设置一个断点。由于 `SVC` 指令在 `0x00000001000541fc` 处执行，我们通过以下命令在 LLDB 调试会话中设置一个断点：

```text
% lldb GoSearch22.app
...
(lldb) b 0x00000001000541fc
```

设置了这个断点后，一旦 CPU 被设置为执行该指令，调试器将停止执行。此时，我们可以将程序计数器（`PC`）的地址更改为 `SVC` 指令之后的指令。在反汇编程序中，我们可以看到下一条指令位于 `0x0000000100054200`。

可以通过 `reg write` 调试器命令更改任何寄存器的值，包括程序计数器。在调试会话中，一旦遇到断点，我们就可以执行该命令，将程序计数器设置为 `0x0000000100054200`，并跳过有问题的 `SVC` 指令。

```text
% lldb GoSearch22.app
(lldb) b 0x00000001000541fc
Breakpoint 1: address = 0x00000001000541fc
(lldb) Process 1486 stopped
* thread #1, queue = 'com.apple.main-thread'
stop reason = breakpoint 1.1:
-> 0x00000001000541fc svc #0x80
(lldb) reg write $pc 0x0000000100054200
```

当 `SVC` 指令被跳过时，它将不会被执行。这巧妙地避免了 `SYS_ptrace` 反调试逻辑…不过，不幸的是类似的还有更多。

[冰与火的战歌：Windows 内核攻防实战高级班！从零到实战，融合 AI 与 Windows 内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2023-12-16 10:31 被 boilsnow 编辑，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)

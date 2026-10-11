
# 软件破解利器 OllyDbg 2.01 内测第 4 版作者说明文稿

> 原文：`www.pediy.com/kssd/pediy12/138268.html`

作者才上传的，还热着呢！

## OllyDbg 2.01

### August 03, 2011 - OllyDbg 2.01 alpha 4. Here is Alpha 4, here is Bookmarks plugin

2011 年 8 月 3 日 - OllyDbg 2.01 内测第 4 版 [这里是内测第 4 版](http://www.ollydbg.de/odbg201d.zip)，[这里是书签插件](http://www.ollydbg.de/plug201d.zip)

As you see, this version already supports plugins. New plugin interface is similar to the old (`v1.10`) but is not backwards compatible. It includes more than 350 `API` functions, 60 or so variables and many enumerations and structures that all need to be documented. This will take a while, therefore I decided to make a preliminary release. It includes plugin header file (`plugin.h`) and commented bookmarks source code (`bookmark.c`). Writing your own plugins without the documentation is a pure masochism, but at least you will be able to analyse the structure of the interface and send me your comments, wishes and suggestions.

正如您所看到的，这个版本已经支持插件。新插件的界面类似于旧版（`v1.10`），但并不向下兼容。它包含了超过 350 个 `API` 函数，60 个左右的变量和常需要记录下来的枚举结构。这将需要一段时间，因此我决定先做一个初步发行版。它包含插件的头文件（`plugin.h`）和注释的书签源代码（`bookmark.c`）。在没有文档的情况下编写自己的插件纯属一个受虐狂，但至少您要能够分析界面的结构并给我您的意见、愿望和建议。

### This is the last alpha release. After plugin documentation is ready, I will call it 2.01 beta 1. Then I will start to write OllyDbg help and finally make the full 2.01 release. Till then, I plan no major changes.

这个是最后的内测版。在插件文档准备完毕之后，我将把它命名为 2.01 外测第 1 版（`2.01 beta 1`）。然后我会开始写 OllyDbg 的帮助文档，并最终做成 2.01 完整发行版。至此，我的计划无较大变化。

### Other new features in this version:

这个版本中的其他新特性：

- `Patch manager, similar to 1.10`
  - 补丁管理器，类似 1.10

- `Shortcut editor, supports weird things like Ctrl+Win+$ etc. Now you can customize and share your shortcuts. I haven't tested it on Win7, please report any found bugs and incompatibilities!`
  - 快捷键编辑器，支持像 `Ctrl+Win+$` 这类奇怪的组合键。现在您可以制定和分享您的快捷键了。它还没有被我在 `win7` 系统里测试过，如果有任何错误和不兼容的情况请报告给我。

- `Instant .udd file loading. In the previous versions I've postponed analysis, respectivcely reading of the .udd file till the moment when all external links are resolved. But sometimes it took plenty of time, module started execution and was unable to break on the breakpoints placed in the DLL initialization routine.`
  - `UDD` 文件即时加载。在以前的版本中我延缓了分析，直到有外部链接解决时才分别读取 `UDD` 文件。但有时它花费大量时间，模块开始执行后它并不能中断在 `DLL` 文件初始化例程中设置的断点上。

- `Automatic search for the SFX entry point, very raw and works only with several packers. Should be significantly more reliable than 1.10. If you tried it on some SFX and OllyDbg was unable to find real entry, please send me, if possible, the link or executable for analysis!`
  - 自动搜索 `SFX` 入口点，非常原始，只能在几个壳中起作用而已。应该比 1.10 版明显可靠些。如果您尝试分析一些 `SFX` 但 OllyDbg 并不能找到真实入口时，如果可能的话请将链接或者可执行文件发送给我分析。

- `"Go to" dialog lists of matching names in all modules`
  - 在所有的模块中列出与“`go to`”对话相匹配的名称

- `Logging breakpoints can protocol multiple s. Here is an example: I ask OllyDbg to protocol the contents of EAX, EBX and 4 memory doublewords starting at address ESP. s must be separated by commas, repeat count has form SIZE\*N, N=1..32:`
  - 可根据设定的多个表达式记录断点。下面是一个例子：我要求 OllyDbg 以 `EAX`，`EBX`，和 4 个双字（`DWORD`）在 `ESP` 中的起始地址的内容为条件，表达式必须用逗号分隔，重复计数时使用 `SIZE*N` 这样的格式，`N` 为 1 到 32 的整数：

![`www.ollydbg.de/Pics/multibreak.gif`](img/6370b61a519b72edf0f696898687b8bc.jpg)

`This is what you will see in the log when breakpoint is hit:`

这是您触发断点时将在日志中看到的内容：

![`www.ollydbg.de/Pics/multilog.gif`](img/4b76aa5fdfaa85fcde20cd3b63b6de45.jpg)

### Many not-so-important new features:

一些不太重要的新功能：

- `Thread names (MS_VC_EXCEPTION)`
  - 线程名称(`MS_VC_EXCEPTION`)

- `UNICODE box characters clipboard mode`
  - `UNICODE` 方框字符剪贴板模式

- `Multiline debugging strings (oflarge size)`
  - 多行调试字符串（大尺寸）

- `On debug string, OllyDbg attempts to find call to OutputDebugString()`
  - 在调试字符串中，OllyDbg 会尝试找到调用的 `OutputDebugString()`

- `INT3 breakpoints set on the first byte of edited memory area are retained`
  - 设置在编辑过的内存区域的首字节的 `int3` 断点会被保留

- `Decoding of User Shared Data block`
  - 用户共享数据块解码

- `Addressing relative to module base`
  - 基准模块的对比寻址

- `If plugin crashes, OllyDbg will report its name`
  - 如果插件崩溃，OllyDbg 的会报告它的名称

- `etc, etc.`
  - 等等，等等。

### I have received many bug reports. Some of them are solved, some are not. There is a very nasty bug that I was unable to reproduce: OllyDbg crashes with memory access violation inside the GlobalAlloc()?!! Either OllyDbg unintentionally taints internal data structures used by memory manager, or some virus scanner overreacts, or this is a bug of Windows itself? If you have any clue, please let me know.

我收到了很多的 `bug` 报告。其中的一些已经解决，有些还没有。这里有一个很讨厌的 `bug`，我无法将它重现：OllyDbg 因里面的 `GlobalAlloc()` 造成了内存访问冲突导致崩溃？！！无论是 OllyDbg 无意中破坏了内存管理器的内部数据结构，或者是一些病毒扫描器的反应过度，又或者是 `windows` 本身的一个 `bug`？如果您有任何线索，请让我知道吧！

### That's all for now. I will make a short vacations, a week or so, and in order to keep my sanity will not check for new emails. Please have some patience!

好了，就到这儿吧。我将做一个短暂的假期调整，一个星期左右，也是为了保持我的理智。这期间我不会检查新邮件，请有点耐心！

\--------------------------------

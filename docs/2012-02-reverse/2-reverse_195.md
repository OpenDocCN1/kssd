
# Debugger Markup Language 帮助文档

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-760.htm`](https://www.pediy.com/kssd/pediy08/pediy8-760.htm)

Debugger Markup Language（DML）是 WINDBG 6.6.7.5 的新增特性。

Microsoft 没有在 debugging help 添加 Debugger Markup Language 的帮助说明，而是额外写了一个说明文档。在程序的快捷方式找不到，MSDN 也没提到，直到小喂“写了个小的 WinDbg 脚本，可以显示 SSDT”（http://bbs.pediy.com/showthread.php?s=&threadid=34018）的帖子出现，才开始关注这项新功能。

文档只有 8 页，不多，我就试着将它翻译下来了。初次做翻译的工作，可能有不少地方翻译得不太好。本译文仅为你了解 DML 提供便利，假如你在使用 DML 的过程有任何疑问，还是查阅英文文档吧。你将能在 WINDBG 的安装目录找到 `dml.doc`。

虽然微软在文档中说明将不会将 DML 发展成功能完善的语言以保持其简易性，不过发挥你的想象力，自己动手，你可以使 WINDBG 在某程度上比 OD 更具易用性。

# Debugger Markup

从 6.6.07 版的调试器开始，为了增强和扩展调试器的数据输出，将使用新的机制：Debugger Markup Language (DML)。DML 象 HTML 那样允许在一定格式的标记中包含指令和不显示的信息。调试器的用户界面将能分析额外的信息并提供新的特性。

DML 的两个主要的功能：

- 链接相关信息。DML 中的一个 `link` 标记能产生一个相关信息的链接，并允许通过一定的操作访问它。就象 HTML 中的链接一样，允许用户快速定位超级链接中的信息。
- 调试器及其扩展功能的输出特性。调试器及其扩展拥有很强大的功能，但是关于外观显示的却很少。用户必须知道什么是可用的，这意味着用户通常不知道什么命令能帮助他们。DML 链接允许将任意指令封装在预备的外观中，例如说明文本，菜单或者帮助链接。

关于理解什么是 DML 的另一个关键：

- DML 不是 HTML。DML 有意设计得很简单，只有一些标记。为了保留简易性，DML 将不会发展成图象显示功能完善的语言。由于已经有很多基于纯文本输出的调试器的代码，DML 的目标是为现有的界面提供简单的 DML 和纯文本的转换。受限制的 DML 标记使它不能成为外观的关键。一些效果，例如文字颜色能得到很好的支持，除非你删除它们，但别删除包含实际信息的文本。另一方面，由于没有变换纯文本流，不支持格式化文本标记的使用，例如定位功能。
- DML 不是 XML。DML 不会传递语义或数据结构信息。正如前面所说的，DML 和纯文本之间，只有简单的变换，因此 DML 标记全部都是可丢弃的。DML 不可扩展，所有标记都是由调试器开发团队定义。

在 `dbgeng` 和远程 `dbgeng` 中，DML 作为信息流通过一般的文本处理通道，它将完全适用于远程调试。只需要使用一个 WEB 浏览器和服务器，DML“浏览器”便能移交来自服务器的内容，因此所有 DML 的内容本来就是可远程使用的。服务器的命令及其扩展在客户端产生的多信息文本或纯文本将依赖于用户界面对 DML 的支持。

译者注：`dbgeng.dll` 是 debugging tools for windows 的成员之一，`dbgeng` 是调试工程的缩写，并为用户提供功能强大的调试接口。方便用户根据自己需要写调试器的，更多信息请查阅帮助文件中的 Debugger Extension API。

## DML 内容说明

DML 对特殊字符的处理规则近似于 XML/HTML。`&`, `<`, `>` 和 `"` 都是特殊字符，不能直接在纯文本中使用。对应的转义字符为 `&amp;`, `&lt;`, `&gt;` 和 `&quot;`。

例如下面的一句话：

"Alice & Bob think 3 < 4"

转换为 DML：

```
&quot;Alice &amp; Bob think 3 &lt; 4&quot;
```

有一个与 XML/HTML 不同的重要规则，DML 文本可以包含不同风格的格式化字符，例如 `\b`, `\t`, `\r` 和 `\n`。这是为了兼容已存在的调试文本。

DML 标记以 `<tagname [args]>` 为开始，`</tagname>` 为结束。当前所有标记都要成对使用开始／完结，但是最后 DML 可能包含与 XML 类似的元素。

### DML 标记参考

`<link [name="text"] [cmd="debugger_command"] [section="name"]>link text</link>`

在 DML 中，该 `link` 标记基于超级链接结构。当链接被点击时，它将指示用户界面显示被链接的文本。当一个带 `cmd` 说明符的 `link` 被点击时，调试器将会运行该命令并替换当前输出。

`Name` 和 `section` 参数允许在已命名的 `link` 之间导航，这与 HTML 中的 `<a name>` 和 `#name` 相似。当带有 `section` 参数的 `link` 被点击，用户界面将会搜索名字与之匹配的 `link`，并滚动到相应位置。这将允许 `link` 指向同一页面的不同区域（或者在新页面中特殊的区域）。DML 的 `section` 名必须是独立的，以避免重新定义新的语法来允许 `section` 名跟在字符串的后面。

`<exec cmd="debugger_command">descriptive text</exec>`

`Exec` 与 `link` 类似，其描述的文本以可点击项出现。然而，在 `Exec` 中，被运行的命令不会替换当前输出，因此这里给出一个简单的点击方式来运行命令，例如菜单。

`<b>text</b>`, `<i>text</i>`, `<u>text</u>`

这些标记将分别把文本转换成粗体，斜体和下划线。它们能嵌套在其他标记中。

`<col fg="name" bg="name">text</col>`

设置文本的前景和背景颜色。使用颜色名字而不是颜色的绝对值，以允许用户控制他们将会看到的颜色。当前颜色名字（只应用于 windbg）:

- `wbg` 和 `wfg` – 窗口默认的背景和前景颜色。默认为窗口和窗口文本的系统颜色。
- `clbg` 和 `clfg` – 当前线段的背景和前景颜色。默认为系统的突显和突显文本
- `changed` – 使用前一个断点以来，被修改的数据，例如 windbg 中的寄存器被改变，默认为红色。
- `srcnum`, `srcchar`, `srcstr`, `srcid`, `srckw`, `srcpair`, `srccmnt`, `srcdrct`, `srcspid`, `srcannot` – 源颜色元素。你能在 windbg 中看到默认值
- `empbg` 和 `emphfg` – 强调文本。默认为亮蓝。
- `subbg` 和 `subfg` – 减弱文本。默认为未激活的标题文本或已激活的标题
- `normbg`, `normfg`, `warnbg`, `warnfg`, `errbg`, `errfg`, `verbbg`, `verbfg` – 输出等级颜色。你能在 windbg 中看到默认值

## 在 dbgeng 接口中加入 DML

基本上，DML 只是嵌入了标记和一些特殊字符规则的文本。Dbgeng 已经有一组处理文本输入的程序和输出界面。因此只需要按照说明要求使用 DML 输入内容，便能得到文本输出。

### 在 dbgeng 中使用 DML 的规定

`DEBUG_OUTCTL_DML`，这个新的控制输出标记指明由 `dbgeng` 模块产生的文本应该被当作 DML 来处理。如果没有给出这个标记，其文本将会象以前一样被当作纯文本处理。`DEBUG_OUTCTL_DM` 能在 `ControlledOutput`, `ControlledOutputVaList`, `ControlledOutputWide` 和 `ControlledOutputVaListWide` 中使用。给出的文本必须是符合 DML 规则的有效字符。

另一个新控制标记，`DEBUG_OUTCTL_AMBIENT_DML` 允许规定输出为 DML 内容而不需要改变任何外在输出控制属性。增加 `DEBUG_OUTCTL_AMBIENT_TEXT`，它将代替过去的 `DEBUG_OUTCTL_AMBIENT`。

译者注：最后一句原文“`DEBUG_OUTCTL_AMBIENT_TEXT` has been added also as a more-descriptive alias for the previously-existing `DEBUG_OUTCTL_AMBIENT`.”

由于允许使用新的格式说明符 `%[h|w]Y{t}`，所有输出例程都得到增强。这个格式说明符作为一个字符指针参数，指明给出的是纯文本并在处理输出过程中被转换为 DML 格式。这将提供一个简单的方法给调用程序，在 DML 的内容中直接输出纯文本而不需要转换成相应的 DML 格式。你可以使用 `h` 和 `w` 指定 ANSI 或者 Unicode，象使用 `%s` 那样。

译者注：不是 ASCII 吗？文档中的是 ANSI。我对这段话的理解，`%[w]Y{文字}`，这将使用 UNICODE 输出“文字”。

### 来自调试目标的 DML 内容

Dbgeng 能通过一个特殊的标记扫描调试目标的输出。`<?dml?>` –它指明该块调试器输出内容应该被作为 DML 处理。该模式的改变只应用于单段调试器输出，例如单段 `OutputDebugString` 字符，而不是全局模式切换。

例如：

```
OutputDebugString("This is plain text\n<?dml?>This is <col fg=\"emphfg\">DML</col> text\n");
```

该段输出结果：一行纯文本跟着一行改变了“DML”颜色的 DML 输出。

### IDebugOutputCallbacks2

`IDebugOutputCallbacks2` 允许 `dbgeng` 客户界面显示完整 DML 内容。`IDebugOutputCallbacks2` 是 `IDebugOutputCallbacks`（不是 `IDebugOutputCallbacksWide`）的扩展，所以你能通过 `SetOutputCallbacks` 使用它。引擎会为 `IDebugOutputCallbacks2` 调用 `QueryInterface` 查询哪个引入接口支持输出回调对象。如果对象支持 `IDebugOutputCallbacks2`，那么所有输出都会传递到 `IDebugOutputCallbacks2` 扩展模块和 `IDebugOutputCallbacks` 基础模块：输出模块将不会被使用。新的方法是

- `GetInterestMask` – 允许回调对象取得它将会接收的输出类型类型。基本类型是纯文本(`DEBUG_OUTCBI_TEXT`)和 DML 内容(`DEBUG_OUTCBI_DML`) 。回调对象也可以使用刷新标记信息(`DEBUG_OUTCBI_EXPLICIT_FLUSH`)。
- `Output2` – 所有 `IDebugOutputCallbacks2` 标记信息都将通过 `Output2` 的参数指定已加载内容的标记、参数和文本参数。现行的标记信息是：
    - `DEBUG_OUTCB_TEXT` – 纯文本除数。标记来自 `DEBUG_OUTCBF_*`，参数输出屏蔽，纯文本输出。只有当 `DEBUG_OUTCBI_TEXT` 被给出时，这才会被接收。
    - `DEBUG_OUTCB_DML` – DML 内容输出。标记来 `DEBUG_OUTCBF_*`，参数输出屏蔽，DML 内容输出。只有当 `DEBUG_OUTCBI_DML` 被给出时才会被接收。
    - `DEBUG_OUTCB_EXPLICIT_FLUSH` – 使用无缓冲文本调用 `FlushCallbacks`。通常，当缓冲文本被刷新，`DEBUG_OUTCBF_COMBINED_EXPLICIT_FLUSH` 将被置位，将两个标记合并为一个。如果没有文本缓冲，那么只有刷新标记会被传出。

注意一个输出对象能同时记录文本或 DML 的内容，假如它们能够被同时处理。在回调函数的输出处理过程中，该引擎会选择格式，以便简化转换，因此同时支持两者可能减少转换。预期的操作模式只支持一种格式，虽然这不是必要的。

### 自动转换

如果有必要，Dbgeng 将会自动转换纯文本或者 DML。例如，如果调用程序传递 DML 内容到引擎，引擎会为所有只接受纯文本输出的客户端将它转换为纯文本。同样，引擎会为只接受 DML 的客户端将纯文本转换为 DML。

## DML 的新用途

在验证 DML 实现的初期，几个引擎命令被加强并且增加了一些命令。这些命令将展示你能用 DML 干些什么。

### 新命令

`.dml_flow <start> <target>`

`.dml_flow` 允许交互式浏览函数中的代码流。它为函数建立一个代码流向图，起始于给定的开始地址（与 `uf` 相似）。然后它显示给出地址的基本程序，加上访问当前分程序的链接。DML 命令的目的是展示如何简单使用链接导航，而不是单调的输入命令。

`.dml_start [<filename>]`

`.dml_start` 被规定为“起始页”，允许你浏览有效指令。它默认执行一组简单的链接，允许导航一些调试目标内容，核心命令，可用的扩展命令和点命令。用户可通过提供指定文件名或设置 `DBGENG_START_FILE` 环境变量来自定义 `.dml_start`。如果 `.dml_start` 带文件参数，那么它将从文件中读取 DML 和显示它。

`!dml_proc`

`!dml_proc` 是新的扩展命令，它显示当前进程并允许深入了解更多信息。最高级的进程链接更详尽的进程信息，例如线程列表。线程列表入口链接线程信息和潜在堆栈信息下至专用结构。核心模式进程显示允许设置和复位嵌入的用户模式状态。进程 `exec` 标记和全部 `!process 0 7` 输出。`!dml_proc` 能工作在用户和内核模式，并自动选择显示何种信息。

`.prefer_dml [0|1]`

`.prefer_dml` 控制 DML 增强命令是否使用 DML 模式的全局设置。

### 增强命令

`.help /D`

`.help` 有一个新的 DML 模式，它将会在最上面显示链接栏，允许以首个字母分类显示命令。比起读整个清单，这更便利。它链接于 `.dml_start`

`.chain /D`

`.chain` 有一个新的 DML 模式。它将链接到 `.extmatch` 命令，并显示扩展 DLL 的命令。它同样链接于 `.dml_start`

`.extmatch /D`

`.extmatch` 有一个新的 DML 模式。它将链接扩展命令的帮助。仅支持每条命令都有帮助描述的扩展，例如 `uexts`。初始化时返回一个新的 `DEBUG_EXTINIT_HAS_COMMAND_HELP` 标记，使用 `!help` 并以命令名为参数，便能显示所有 `dbgeng` 格式扩展命令。

`lm /D`

`lm` 有一个新的 DML 模式。它将模块名链接到 `lmv` 命令，以给出单个模块的更多的信息。在标题栏也有链接允许你选择以首字排序还是首址排序。

`k /M`

`k` 有一个新的 DML 模式，它将帧数链接到 `.frame /dv` 命令，以显示本地帧

`.printf /D`

`/D` 选项指明由 `.printf` 产生的字符是 DML 内容。这允许 `SCRIPT` 或者命令产生 DML 增强输出。`.printf` 不受 `.prefer_dml` 影响。

## 命令窗口增强

所有 WINDOWS 调试器现在拥有支持 DML 语法解释的命令输出区域。WINDBG 的命令窗口支持所有 DML 性能，将能显示颜色，字体风格和链接。调试器控制台 - `ntsd`, `cdb` 和 `kd` - 只支持 DML 的颜色属性，并且仅在彩色模式下有效。重定向 I/O，`ntsd -d` 或 `remote.exe` 对话将不会显示任何颜色（实际上即使使用 `IDebugOutputCallbacks2` 也不会）。

所有 WINDOWS 调试器都有双 I/O 通道，以便涉及或链接到不支持 `IDebugOutputCallbacks2` 的 `dbgeng.dll` 版本时，它们仍能求助于 `IDebugOutputCallbacks[Wide]`。

## Windbg 的命令浏览窗口

Windbg 拥有一个新的能分析并显示 DML 的用户界面：命令浏览窗口。命令浏览窗口的工作与旧版的命令窗口相似，同样有一个显示输出的地方和文本输入栏。然而命令浏览窗口象网页那样集中显示单条命令的所有输出。所有标记 - `link`, `exec` 和外观改变 - 都完全支持。

命令浏览窗口模仿网页浏览器的性能。它有一个下拉的历史窗口和向前向后的按钮（向前向后的命令将支持额外鼠标键的工作）。下拉的历史窗口只显示最近的 20 条命令，但是所有历史记录将会被保留，回到过去的命令，你将能在下拉窗口中显示更旧的历史记录。

命令浏览窗口能自动刷新也能手动刷新。在调试器状态改变的时候，自动刷新浏览器将自动重新运行它们的命令。保持现场输出是以执行所有被改变的指令为代价的（调试器无法知道命令依靠什么状态，因此无法优化刷新）。自动刷新功能是默认打开的。如果不需要现场效果，可以使用窗口的菜单关闭它。如果检查到一个命令引起调试器改变，命令窗口同样会自动关闭自动刷新功能，因为在那个情况下，窗口将会不停刷新。

“最近命令”子窗口已经被添加到 `View` 菜单，以保留感兴趣的命令。选择一个最近的命令将会打开一个新浏览窗口并运行。在窗口的菜单中有一个选项可添加当前命令到最近命令清单。该清单保存在 `workspaces` 中。

`View` 有一个“Set Browser Start Command”选项，它允许用户设置一个命令在新浏览窗口打开的时候运行，例如 `.dml_start`。这个命令会保存在 `workspaces` 中。

右键点击链接将会看到一个类似网页浏览器那样的菜单。链接能跟随，或者在新浏览窗口中跟随。链接中的命令能复制到剪贴版使用。当前的链接不能以弹出菜单显示链接命令。

由于多信息编辑的局限性，距离首字太远的点击可能不被验证。如果它不能，尝试点击距离开始位置远一点的地方。



在命令窗口里使用`.browse <command>`将会显示新的命令浏览窗口和运行给出的命令。

只要你愿意，可以打开任意数量的窗口。命令窗口保存在`workspaces`中，但是只保存当前命令；历史不会保留。

命令浏览窗口能运行所有调试命令，它不一定要产生 DML 的命令。你能在浏览窗口执行任意一组命令而不需要关心 DML。命令是由引擎运行的，不是用户界面。这表明用户界面专用指令，例如`.cls`，不能在命令浏览窗口中使用。它同样意味着当用户界面是远程客户端的时候，命令会由服务端运行并受服务端的状态影响，而不是客户端。

命令浏览窗口同步地执行命令并且不显示输出，直到执行完毕为止。当前没有方法在命令运行的时候产生输出；一段很长的命令将不会显示任何东西，除非它们已经完结。命令完成之后输入栏才可用。使用 `CTRL+N` 可以打开一个新的命令浏览窗口。

## 控制台调试器彩色模式

正如之前提到的那样，当运行在真实的控制台的时候，`ntsd`、`cdb` 和 `kd` 都拥有彩色输出显示特性。这不是默认设置，它需要经由 `tools.ini` 来激活彩色模式。在 `tools.ini` 中，新的 `col_mode <true|false>` 标记控制彩色模式的设置。

当彩色模式被激活，调试器就能产生彩色输出。默认情况下，大部分颜色都没有被设置，取而代之的是当前控制台的颜色。在 `tools.ini` 中，新的 `col <name> <colspec>` 标记允许用户定义他们想要的颜色。上述在“col”标记中的名字属性是颜色名字。Colspec 以`[rR-][gG-][bB-]`的格式，由 3 个字母 RPG 指定。小写字母效果暗一点，大写字母效果亮一点。破折号代表对颜色没有任何贡献。由于控制台颜色局限性，不必每个成分都使用亮色，但是如果你有亮色的需求，那么将亮色应用于每个组成。换句话说 `rgB` 和 `RGB` 的效果是一样的。因为这个原因假如你需要使用任何大写字母，推荐全部都使用大写字母。例如：

*   `R--`: 亮红
*   `-g-`: 暗绿
*   `-gb`: 暗青
*   `RG-`: 亮黄(效果跟 `Rg-` 相同, 但是 `RG-` 是更佳的选择)
*   `---`: 黑色
*   `RGB`: 白色

4dd466d0e1e0cc123afbe504da2462ca

...[/content]中，并且只返回处理后的内容。我将按照最终结果输出。

在开始之前，我需要明确一些术语：

- “VMP”代表VMProtect，不加反引号。
- “Safengine”和“Themida”不加。
- “tls”我决定不加反引号，因为不是具体代码元素，但是“TLS”通常大写。不过原文为小写，保留小写。
- “SEH”不加。
- “OEP”不加。
- “Debug”相关函数加。

注意：对于“BeingDebugged”，这是PEB中的一个字段，加反引号。

对于“NtGlobalFlag”也是字段，加。

“Heap 结构的ForceFlags 和Flags” — `ForceFlags`和`Flags`加。

“数据结构中的调试记号（0xbababa,badf00d 等）” — 0xbababa和0xdadf00d？原文为“0xbababa,badf00d”，实际应为“0xbababa, 0xbadf00d”但我们保留原文写法。加反引号可能不合适，因为它们是数值，但示例中`+`也加了。为了统一，可以加。但注意“0xbababa,badf00d”中间有逗号，可能是列举。我们可写为`0xbababa, badf00d`？但badf00d缺少0x前缀，原文如此。保留。

“tls callback” 中的callback是API概念，不加。

“文件完整性检测” 不加。

“反调试只包含...等” 中函数加。

现在开始构造输出。

先写标题：

```
# [原创]浅谈VMP、safengine和Themida的反虚拟机
```

然后正文引言段落。

注意原文中“浅谈VMP、safengine 和Themida 的反虚拟机”作为第二行，我删除。但也许应该保留作为副标题？不。

正文第一段：

“这里浅谈一下VMP、Safengine 和Themida 的反虚拟机，关于反调试只说下值得注意的地方。VMP 的反调试请参考 https://bbs.pediy.com/thread-226455.htm，Safengine 和 Themida 的反调试与 VMP 差不多。分析的 VMP 版本为 3.0.9，Themida 版本为 2.4.6.30，Safengine 不知道是什么版本了，程序是 32bit。”

这里“32bit”可保留。

第二段：

“关于 64bit 程序，反调试和反虚拟机都更简单。其中反调试只包含 IsDebuggerPresent、 NtGlobalFlag、CheckRemoteDebuggerPresent。不过需注意的是程序在OEP 后，IsDebuggerPresent之前保存NtGlobalFlag标志，在IsDebuggerPresent之后同时检测BeingDebugged和NtGlobalFlag标志，最后调用 CheckRemoteDebuggerPresent。反虚拟机则只有注册表检测，没有 in 指令检测，因此只需修改注册表，就可在虚拟机运行。”

这里注意“OEP”不加，但“IsDebuggerPresent”等加。

格式化后：

```
这里浅谈一下VMP、Safengine 和Themida 的反虚拟机，关于反调试只说下值得注意的地方。VMP 的反调试请参考 https://bbs.pediy.com/thread-226455.htm，Safengine 和 Themida 的反调试与 VMP 差不多。分析的 VMP 版本为 3.0.9，Themida 版本为 2.4.6.30，Safengine 不知道是什么版本了，程序是 32bit 。

关于 64bit 程序，反调试和反虚拟机都更简单。其中反调试只包含`IsDebuggerPresent`、`NtGlobalFlag`、`CheckRemoteDebuggerPresent`。不过需注意的是程序在 OEP 后，`IsDebuggerPresent`之前保存`NtGlobalFlag`标志，在`IsDebuggerPresent`之后同时检测`BeingDebugged`和`NtGlobalFlag`标志，最后调用`CheckRemoteDebuggerPresent`。反虚拟机则只有注册表检测，没有`in`指令检测，因此只需修改注册表，就可在虚拟机运行。
```

注意“32bit ”后面有空格，统一为“32bit。”。

“在 OEP 后” 原文没有空格，我们加了空格？原文是“程序在OEP 后”，应保持。我们写“程序在`OEP`后”？OEP不是代码，不加。可以写“程序在 OEP 后”，添加空格使阅读顺畅。实际上原文“程序在OEP 后”有一个空格？原文是“程序在OEP 后”，有空格。我们保留原空格。

然后“## 1 反调试”标题。

注意在“1 反调试”和“1.1 VMP 反调试”之间是否有介绍？原文直接是“1 反调试”然后“1.1 VMP 反调试”。没有多余内容，我们直接写标题。

“### 1.1 VMP 反调试”

正文段落：

“VMP 的某些版本有 tls 保护，该tls 用于检测调试器，具体方法不明。目前排除了 tls 对软件断点、硬件断点、SEH、NtGlobalFlag、BeingDebugged、Heap 结构的ForceFlags 和Flags 检测，包括堆中的调试记号（0xbababa,badf00d 等）。虽然不明检测方法，但要过掉还是挺简单的，直接跳过该tls。跳过的步骤是用调试器启动程序，在加载 ntdll 后，把 TLS 的DataDirectory清零，让系统读不到tls callback，当走到OEP 时，再还原。当然，如果程序没有文件完整性检测，可以直接修改文件中TLS 的DataDirectory 项。”

这里需要加反引号的有：`NtGlobalFlag`、`BeingDebugged`、`ForceFlags`、`Flags`、`0xbababa,badf00d`、`ntdll`、`DataDirectory`。注意“tls”我不加，“OEP”不加。还有“tls callback”中的callback？不加。

“Heap 结构的ForceFlags 和Flags”中“Heap结构”是中文，后面字段加。

“调试记号（0xbababa,badf00d 等）” 中我们写成 (`0xbababa,badf00d` 等)。但注意标记是“0xbababa,badf00d”，中间没有空格，我们保留。

另外“tls 对软件断点、硬件断点、SEH、NtGlobalFlag、BeingDebugged、Heap 结构的ForceFlags 和Flags 检测” 中的“SEH”是不用加。所以整段为：

VMP 的某些版本有 tls 保护，该tls 用于检测调试器，具体方法不明。目前排除了 tls 对软件断点、硬件断点、SEH、`NtGlobalFlag`、`BeingDebugged`、Heap 结构的`ForceFlags`和`Flags`检测，包括堆中的调试记号（`0xbababa,badf00d` 等）。虽然不明检测方法，但要过掉还是挺简单的，直接跳过该tls。跳过的步骤是用调试器启动程序，在加载`ntdll`后，把 TLS 的`DataDirectory`清零，让系统读不到tls callback，当走到OEP 时，再还原。当然，如果程序没有文件完整性检测，可以直接修改文件中TLS 的`DataDirectory`项。

注意“该tls”中的tls我们保留原文小写。也许应该改为“该TLS”？但保留原文。

然后“## 2 反虚拟机”标题。

“### 2.1 VMP 反虚拟机”

第一段：

“从结果来说，VMP 反虚拟机只使用了一个方法，特殊指令，该特殊指令是 cpuid。检测原理是赋eax 为1，执行 cpuid 后，如果ecx 的31st 位为 0，表示真机，否则为虚拟机。”

加反引号：`cpuid`、`eax`、`ecx`。注意“31st”是英文序数，不加。

写为：

从结果来说，VMP 反虚拟机只使用了一个方法，特殊指令，该特殊指令是`cpuid`。检测原理是赋`eax`为1，执行`cpuid`后，如果`ecx`的31st 位为 0，表示真机，否则为虚拟机。

第二段：

“从分析过程来说，主要分为 3 步。 一是分析虚拟机框架，二是逐步接近特殊指令，三是定位特殊指令，如图1。”

这里“3 步”后有多余空格，去除。保留“图1”。注意“如图1”应与图片对应。我们写：

从分析过程来说，主要分为 3 步。一是分析虚拟机框架，二是逐步接近特殊指令，三是定位特殊指令，如图1。

然后图片：

```
![图1](https://bbs.kanxue.com/upload/attach/201901/3_BYDXNJ9SJDHRAY7.jpg)
```

但注意下面还有图题“图1 VMP 分析过程”。我们可以保留图题，但图片本身有alt文字“图1”。为了避免重复，我们可以写：

![VMP 分析过程](https://bbs.kanxue.com/upload/attach/201901/3_BYDXNJ9SJDHRAY7.jpg)

或者保留原样，并在下方写“图1 VMP 分析过程”。原样即可。

注意在Markdown中，图片下方可以有文字说明。我们用：

```
![](https://bbs.kanxue.com/upload/attach/201901/3_BYDXNJ9SJDHRAY7.jpg)

图1 VMP 分析过程
```

但根据标准，图注可以放在图片下一行。没问题。

接下来“1）虚拟机框架：” 这一行。我们将其作为加粗段落，即：

**1）虚拟机框架：**

然后正文：

“根据初步 VMP 的分析，该 VMP 无 dispatcher，全程是用 push edi，ret 来调整执行流程，其中的edi 由mov …,dword ptr [esi]转换而来。VMP 的单元步骤为一对，如代码段 1。”

这里需要加反引号：`dispatcher`（可加），`push edi`、`ret`、`edi`、`mov …,dword ptr [esi]`。注意“push edi，ret”中的逗号是中文，应改为英文逗号？在指令列表中，用英文逗号或顿号。原文为“push edi，ret”（中文逗号），因为这是代码，我们最好改为“`push edi`、`ret`”或“`push edi, ret`”。我倾向于用“`push edi`、`ret`”这种列表形式。但“全程是用 push edi，ret 来调整执行流程”意思是使用`push edi`和`ret`指令。可以写成“全程是用`push edi`、`ret`来调整执行流程”。注意“其中的edi 由mov …,dword ptr [esi]转换而来”中`edi`和`mov …,dword ptr [esi]`加。

所以：

根据初步 VMP 的分析，该 VMP 无`dispatcher`，全程是用`push edi`、`ret`来调整执行流程，其中的`edi`由`mov …,dword ptr [esi]`转换而来。VMP 的单元步骤为一对，如代码段 1。

接着是图片：

```
![](https://bbs.kanxue.com/upload/attach/201901/3_TQB9VSGE3JBVTHR.jpg)

代码段1VMP单元对
```

注意“代码段1VMP单元对”没有空格，我们可调整为“代码段1 VMP单元对”或“代码段 1 VMP 单元对”。保留原意，可加空格。

然后下一段：

“其中，值得注意的是字节码的表示，esi 和 edx 轮换表示字节码的当前获取地址，这些地址是一段一段的，且可以重复利用，如图2 所示。”

加反引号：`esi`、`edx`。写为：

其中，值得注意的是字节码的表示，`esi`和`edx`轮换表示字节码的当前获取地址，这些地址是一段一段的，且可以重复利用，如图2 所示。

然后是图片：

```
![](https://bbs.kanxue.com/upload/attach/201901/3_MMB2MNJ6E46JRGA.jpg)

图2 字节码执行流程
```

接下来“2）逐步接近特殊指令：” 加粗。

正文：

“找到存放字节码段首地址的栈地址。因为字节码段是乱序且繁多的，手动跟踪非常缓慢。之后注意到在ebp 的一个相对偏移处存有 esi 或edx 的字节码段首地址，且ebp 相对一段时间是固定不变的，因此可在这个相对偏移下写断点。果然，这个断点被触发了一千多次，由于堆栈的成长，ebp 会超过VMP 自己的虚拟栈，因此需要调整ebp，这时存放字节码段首地址的相对偏移会稍有变化，因此需要重复几次找这个“相对偏移”的步骤。最后越来越接近特殊指令。”

加反引号：`ebp`、`esi`、`edx`。注意“栈地址”不加。写为：

找到存放字节码段首地址的栈地址。因为字节码段是乱序且繁多的，手动跟踪非常缓慢。之后注意到在`ebp`的一个相对偏移处存有`esi`或`edx`的字节码段首地址，且`ebp`相对一段时间是固定不变的，因此可在这个相对偏移下写断点。果然，这个断点被触发了一千多次，由于堆栈的成长，`ebp`会超过VMP 自己的虚拟栈，因此需要调整`ebp`，这时存放字节码段首地址的相对偏移会稍有变化，因此需要重复几次找这个“相对偏移”的步骤。最后越来越接近特殊指令。

注意“相对偏移”为何不加引号？不是代码。

“3）定位特殊指令：” 加粗。

正文：

“通过程序检测到虚拟机的附近代码寻找特殊指令。虽然第二步的结果很接近特殊指令了，但在VMP 中，这个距离还是非常长的，因此需要另找方法。具体方法是定位程序检测到虚拟机的错误点，然后一步一步逼近，最后用 x32dbg 的 trace 功能扫描代码，找到特殊指令。其中，定位错误点根据“Sorry, this application cannot run under a Virtual Machine.”提示语句寻找，因为这段文字被存在栈中的一个固定位置，所以只需在此处下断点就能判断何时触发的错误点，然后向上回溯，最终找到特殊指令。”

加反引号：`x32dbg`（工具名可不加，但作为命令？不加），`trace`（功能名？可加），字符串加反引号。注意“x32dbg 的 trace 功能”中x32dbg是软件名，trace是功能，通常写作`trace`。我们写为“最后用 x32dbg 的`trace`功能扫描代码”。字符串`"Sorry, this application cannot run under a Virtual Machine."`应加反引号，并注意标点。原文语句末尾有句点，我们保留在反引号内。

写为：

通过程序检测到虚拟机的附近代码寻找特殊指令。虽然第二步的结果很接近特殊指令了，但在VMP 中，这个距离还是非常长的，因此需要另找方法。具体方法是定位程序检测到虚拟机的错误点，然后一步一步逼近，最后用 x32dbg 的`trace`功能扫描代码，找到特殊指令。其中，定位错误点根据“`Sorry, this application cannot run under a Virtual Machine.`”提示语句寻找，因为这段文字被存在栈中的一个固定位置，所以只需在此处下断点就能判断何时触发的错误点，然后向上回溯，最终找到特殊指令。

注意中文引号内包含反引号，这样可行。

“4）快速定位：” 加粗。

正文：

“以上是定位的过程，这里提供一个快速的定位方法。由于程序对 VM 化的代码不会再做加密，因此可以在vm 段直接内存搜索，找到cpuid，注意VMP 一般会用一次或两次该指令来检测。”

加反引号：`vm`段？不确定。`cpuid`。写为：

以上是定位的过程，这里提供一个快速的定位方法。由于程序对 VM 化的代码不会再做加密，因此可以在`vm`段直接内存搜索，找到`cpuid`，注意VMP 一般会用一次或两次该指令来检测。

注意“vm 段”可能是虚拟内存段，用反引号包`vm`也可以。

“5）过掉cpuid：” 加粗。

正文：

“一般执行cpuid 时，ecx 的31st 位为0，所以 cpuid 直接用两个nop 替换即可。”

原文有引用符号“>”，表示这是引用。我们保留引用？在Markdown中，可以用“>”表示引用。原文就是“> 一般执行...”，所以我们可以保留：

> 一般执行`cpuid`时，`ecx`的31st 位为0，所以`cpuid`直接用两个`nop`替换即可。

注意原文中“所以 cpuid 直接用两个nop 替换即可”中的cpuid和nop加反引号。

这样VMP部分结束。

接下来“2.2 Safengine 反虚拟机” 标题。

正文：

“使用 in 指令和 RegQueryValueExA 检查 SystemManufacturer(在注册表里)，in 指令的原理请参考https://bbs.pediy.com/thread-225735.htm。这里只说下如何定位 in 指令和过掉。in 指令在真机中会产生0xC0000096 异常，而在虚拟机中不会产生异常。因此定位 in 指令需要在真机的调试器中运行，但前提是你要知道它反虚拟机使用的是 in 指令，这个就需要各种尝试了（因为safengine 首先是反调试，然后再是反虚拟机，所以分析反调试时很轻松就能发现 in 指令）。”

需要加反引号：`in`、`RegQueryValueExA`、`SystemManufacturer`、`0xC0000096`。注意“SystemManufacturer(在注册表里)”中括号是中文，我们可以改为英文括号或保留。原文是中文括号，我们保留。URL不加。写为：

使用`in`指令和`RegQueryValueExA`检查`SystemManufacturer`（在注册表里），`in`指令的原理请参考https://bbs.pediy.com/thread-225735.htm。这里只说下如何定位`in`指令和过掉。`in`指令在真机中会产生`0xC0000096`异常，而在虚拟机中不会产生异常。因此定位`in`指令需要在真机的调试器中运行，但前提是你要知道它反虚拟机使用的是`in`指令，这个就需要各种尝试了（因为safengine 首先是反调试，然后再是反虚拟机，所以分析反调试时很轻松就能发现`in`指令）。

注意“safengine”小写，保留。

下一段：

“定位到后，就是过掉了。为了说明过掉方法的原理，这里需要再讲下 in 指令。in 在真机中产生异常，执行程序的SEH，在SEH 中会将eip 重新赋值。该过程就像一个跳转，在 in 指令的地址处有一条jmp 指令的感觉。而在虚拟机中，不会产生异常，那么就会进入检测到虚拟机的分支。”

加反引号：`in`、`eip`、`jmp`。注意“SEH”不加。写为：

定位到后，就是过掉了。为了说明过掉方法的原理，这里需要再讲下`in`指令。`in`在真机中产生异常，执行程序的SEH，在SEH 中会将`eip`重新赋值。该过程就像一个跳转，在`in`指令的地址处有一条`jmp`指令的感觉。而在虚拟机中，不会产生异常，那么就会进入检测到虚拟机的分支。

下一段：

“稍微了解了 in 指令后，现在说明两个过掉方法。一是换成 int3，二是重设 eip。关于一， in 和 int3 指令只占用一个字节，且 int3 也可产生异常，因此非常合适。但不排除程序的 SEH会检测该异常的 ExceptionCode，如果是这样，那就用第二种方法。关于二，调试程序，走到in 指令后（下硬件断点），手动给 eip 赋值，该值是 SEH 里指定的值。”

加反引号：`in`、`int3`、`eip`、`ExceptionCode`。注意“关于一，”后的空格“ in”去掉。写为：

稍微了解了`in`指令后，现在说明两个过掉方法。一是换成`int3`，二是重设`eip`。关于一，`in`和`int3`指令只占用一个字节，且`int3`也可产生异常，因此非常合适。但不排除程序的 SEH 会检测该异常的`ExceptionCode`，如果是这样，那就用第二种方法。关于二，调试程序，走到`in`指令后（下硬件断点），手动给`eip`赋值，该值是 SEH 里指定的值。

注意“SEH会”中加空格。

然后“2.3 Themida 反虚拟机” 标题。

“2.3.1 注册表检测” 作为四级标题。

正文：

“两次使用in 指令和RegQueryValueExA 检查注册表内容。其中注册表内容包括以下三点：”

加反引号：`in`、`RegQueryValueExA`。

然后列表：

“1. HKEY\_LOCAL\_MACHINE\HARDWARE\DESCRIPTION\System\VideoBiosVersion(一般虚拟机中没有,不过没有该项也不会导致虚拟机被检测到)

2. HKEY\_LOCAL\_MACHINE\HARDWARE\DESCRIPTION\System\ SystemBiosVersion

3. HKEY\_LOCAL\_MACHINE\SYSTEM\ControlSet001\Control\Class\{4d36e968-e325-11ce-bfc1-08002be10318}\0000\DriverDesc”

注意原文中路径下划线被转义，我们在反引号中保留原始路径。另外第二个路径“System\ SystemBiosVersion”有空格，原文“System\ SystemBiosVersion”中间有空格？原文本是“System\ SystemBiosVersion”，我们保留，但可能是一个笔误，应是“System\SystemBiosVersion”。但为了不修改语义，我们保留空格。不过加反引号后，代码中空格明显，可以去掉？最好保留原样。另外注意第一点中有中文逗号和括号，我们规范化。

列表项用有序列表：

1. `HKEY_LOCAL_MACHINE\HARDWARE\DESCRIPTION\System\VideoBiosVersion`（一般虚拟机中没有，不过没有该项也不会导致虚拟机被检测到）
2. `HKEY_LOCAL_MACHINE\HARDWARE\DESCRIPTION\System\ SystemBiosVersion`
3. `HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Control\Class\{4d36e968-e325-11ce-bfc1-08002be10318}\0000\DriverDesc`

注意第二项中“System\ SystemBiosVersion”可能是笔误，但保留。在代码中，反斜杠后空格不常见。我倾向于保留原文字符串，因为代码路径不应该有空格，但原文有，我们保留以忠实于原文。也许“System\ SystemBiosVersion”是换行导致的？原文本中“System\ SystemBiosVersion”确实有一个空格。为保真，保留。

“2.3.2 in 指令检测” 四级标题。

正文：

“两次in 指令的使用中ecx 分别为0x14 和0xA，0x14 是检查返回的eax，0xA 是检查返回的ebx。其中：”

加反引号：`in`、`ecx`、`0x14`、`0xA`、`eax`、`ebx`。写为：

两次`in`指令的使用中`ecx`分别为`0x14`和`0xA`，`0x14`是检查返回的`eax`，`0xA`是检查返回的`ebx`。其中：

列表：

“- ecx 为0x14 时，VMware® Workstation 14 Pro 返回的 eax 为0x800，在真机中会跳到 SEH，将eax 赋值为0

- ecx 为 0xA 时，虚拟机中的 ebx 为 'VMXh'，在真机中进入 SEH，并在 SEH 中直接修改eip 跳转”

注意列表项。需要加反引号：`ecx`、`0x14`、`eax`、`0x800`、`0xA`、`ebx`、`'VMXh'`、`eip`。注意“VMXh”有单引号，我们保留。写为：

- `ecx`为`0x14`时，VMware® Workstation 14 Pro 返回的`eax`为`0x800`，在真机中会跳到 SEH，将`eax`赋值为0
- `ecx`为 `0xA`时，虚拟机中的`ebx`为 `'VMXh'`，在真机中进入 SEH，并在 SEH 中直接修改`eip`跳转

注意“ecx 为 0xA 时”中空格统一。

“2.3.3 in 指令定位过程” 四级标题。

下面有步骤“1）检测相关函数”等等。我们将其作为加粗段落。

“1）检测相关函数”

正文：

“因为 Themida 没有用 PSAPI.dll、IPHLPAPI.dll、shlwapi.dll，所以能用来反虚拟机的函数比较有限，除了与注册表相关的函数，剩下的是检查固件信息的函数，不过设断点后都没有触发。”

这里`PSAPI.dll`、`IPHLPAPI.dll`、`shlwapi.dll`应加反引号。写为：

因为 Themida 没有用`PSAPI.dll`、`IPHLPAPI.dll`、`shlwapi.dll`，所以能用来反虚拟机的函数比较有限，除了与注册表相关的函数，剩下的是检查固件信息的函数，不过设断点后都没有触发。

然后有代码行：

“ba e1 KERNELBASE!GetSystemFirmwareTable

ba e1 KERNELBASE!EnumSystemFirmwareTables”

这两行是调试命令，应该作为代码块。原文是两行，我们放入```中：

```
ba e1 KERNELBASE!GetSystemFirmwareTable
ba e1 KERNELBASE!EnumSystemFirmwareTables
```

注意在代码块中内容原样。

然后下一段：

“之后我用APIMonitor 监控Themida 对API 的调用，除了注册表相关函数，确实没有其他用于检测虚拟机的函数了。因此我猜测剩下的虚拟机检测方法不是用的函数，而是特殊指令。”

“APIMonitor”工具名不加。

“2）搜索cpuid 指令”

正文：

“让程序运行，在Themida 的可执行段搜索cpuid 指令，在可能的指令处下硬件断点，结果没有命中。因为Themida 有SMC，所以在代码解密之后搜索cpuid，尝试下断点，但仍然没有命中。”

加反引号：`cpuid`。SMC不加。写为：

让程序运行，在Themida 的可执行段搜索`cpuid`指令，在可能的指令处下硬件断点，结果没有命中。因为Themida 有SMC，所以在代码解密之后搜索`cpuid`，尝试下断点，但仍然没有命中。

“3）跟踪错误提示字符串并尝试分析多线程”

正文：

“跟踪MessageBoxExA 引用的提示字符串时，发现多线程SMC。之后分析多线程时，无法掌控主线程的执行进度。Themida 在检测到虚拟机后，会用 MessageBoxExA 弹框，于是我开始跟踪“Sorry, this application cannot run under a Virtual Machine”字符串，发现该字符串是多次解密后产生的结果。之后我回溯该字符串的解密过程，但由于多线程之间频繁的相互切换，我无法持续单步跟踪主线程，导致当线程切换后，主线程已经执行到不可预估的地方了。”

加反引号：`MessageBoxExA`，字符串加。写为：

跟踪`MessageBoxExA`引用的提示字符串时，发现多线程SMC。之后分析多线程时，无法掌控主线程的执行进度。Themida 在检测到虚拟机后，会用`MessageBoxExA`弹框，于是我开始跟踪“`Sorry, this application cannot run under a Virtual Machine`”字符串，发现该字符串是多次解密后产生的结果。之后我回溯该字符串的解密过程，但由于多线程之间频繁的相互切换，我无法持续单步跟踪主线程，导致当线程切换后，主线程已经执行到不可预估的地方了。

下面有“关于跟踪主线程，还有两点需要解释：”段落。原文是：

```
关于跟踪主线程，还有两点需要解释：

第一、无法保持一直跟踪主线程，因为主线程执行到不同代码片段时会进入死循环，等待其他的某个线程修改对应的死循环代码，因此仅仅是挂起除主线程的所有进程是无法继续分析的，必须在某一点唤醒其他解密线程。

第二、线程切换时不能预估主线程的执行进度。当主线程需要执行待解密的代码时，会首先标记一个事件句柄（即告诉其他线程，你们可以执行了），然后调用Sleep(0)，释放掉当前分配给自己的执行时间，以让其他线程被调度。而其他线程有部分是调用 Sleep，正准备被唤醒，有部分是用WaitForSingleObject 等待某事件句柄，该事件句柄正是主线程标记的事件句柄。在这个过程中，如果对主线程 Sleep(0)的下一条汇编指令下断点，结果就是断点会被命中，但往往被命中的线程不是主线程，而是同样调用 Sleep、等待被唤醒的其他线程。因此，这里不应该在主线程将被唤醒的汇编代码处下断点，即使断点被命中，也很可能是其他线程被命中。因为除主线程，还有大概 25 个线程等待被唤醒，其中调用 Sleep 等待被唤醒的数量可能有 7、8个。
```

这里“Sleep(0)”和“WaitForSingleObject”加反引号。“Sleep”单独出现也加。写为：

关于跟踪主线程，还有两点需要解释：

第一、无法保持一直跟踪主线程，因为主线程执行到不同代码片段时会进入死循环，等待其他的某个线程修改对应的死循环代码，因此仅仅是挂起除主线程的所有进程是无法继续分析的，必须在某一点唤醒其他解密线程。

第二、线程切换时不能预估主线程的执行进度。当主线程需要执行待解密的代码时，会首先标记一个事件句柄（即告诉其他线程，你们可以执行了），然后调用`Sleep(0)`，释放掉当前分配给自己的执行时间，以让其他线程被调度。而其他线程有部分是调用`Sleep`，正准备被唤醒，有部分是用`WaitForSingleObject`等待某事件句柄，该事件句柄正是主线程标记的事件句柄。在这个过程中，如果对主线程`Sleep(0)`的下一条汇编指令下断点，结果就是断点会被命中，但往往被命中的线程不是主线程，而是同样调用`Sleep`、等待被唤醒的其他线程。因此，这里不应该在主线程将被唤醒的汇编代码处下断点，即使断点被命中，也很可能是其他线程被命中。因为除主线程，还有大概 25 个线程等待被唤醒，其中调用`Sleep`等待被唤醒的数量可能有 7、8个。

注意“7、8个”保留。

“4）尝试分析虚拟机框架”

正文：

“分析虚拟机框架，尝试定位提示字符串产生的过程，找到反虚拟机的特殊指令。因为无法精确跟踪主线程，所以我想分析 Themida 的虚拟机框架，进一步分析细节，找到提示字符串的解密以及产生过程。在看雪搜索相关帖子后，发现分析成本很大，无法快速达到定位特殊指令的目的，因此需要另找思路。”

无代码，保留。

“5）尝试根据虚拟机配置来定位特殊指令”

正文：

“Themida 的虚拟机检测可以通过设置 VMware 选项过掉(不过我尝试没成功)，比如：

disable_acceleration = "TRUE"

monitor_control.restrict_backdoor = "TRUE"

于是我想弄懂这两句的原理，以此定位特殊指令。通过一段时间的搜索，发现这些语句可能与虚拟机解释执行指令有关，不会与某条特定的汇编指令产生关联（因为找不到官方文档的说明，无法确定该猜想）。因此通过虚拟机配置寻找特殊指令大概是不可行的。”

注意“比如：”后是代码块，我们实现为：

Themida 的虚拟机检测可以通过设置 VMware 选项过掉（不过我尝试没成功），比如：

```
disable_acceleration = "TRUE"
monitor_control.restrict_backdoor = "TRUE"
```

于是我想弄懂这两句的原理，以此定位特殊指令。通过一段时间的搜索，发现这些语句可能与虚拟机解释执行指令有关，不会与某条特定的汇编指令产生关联（因为找不到官方文档的说明，无法确定该猜想）。因此通过虚拟机配置寻找特殊指令大概是不可行的。

注意原文“过掉(不过我尝试没成功)”半角括号，改为全角。

“6）与真机对比并锁定关键条件跳转”

正文：

“对比真机和虚拟机的执行情况，找到关键代码段；对比这两份代码段，发现判断虚拟机的标志。单独分析虚拟机的情况无法找到特殊指令，那在真机中执行相同步骤，以此来寻找异同点，以下为两个关键点。”

无代码。

“一是在真机和虚拟机中，“Sorry, this application cannot run under a Virtual Machine”字符串都会被解密。在虚拟机的分析中，我设定了一个前提，即解密该字符串意味着 Themida 发现了虚拟机。但真机的这一现象，瞬间颠覆了我的想法，看来准备工作没做充分。于是我开始在虚拟机和真机中交替分析程序，找到了两种环境下程序的分叉点，从该分叉点继续执行大概23000 条汇编指令，虚拟机会执行到 MessageBoxExA 并报告发现虚拟机，而真机则不会报告。”

加反引号：字符串，`MessageBoxExA`。写为：

一是在真机和虚拟机中，“`Sorry, this application cannot run under a Virtual Machine`”字符串都会被解密。在虚拟机的分析中，我设定了一个前提，即解密该字符串意味着 Themida 发现了虚拟机。但真机的这一现象，瞬间颠覆了我的想法，看来准备工作没做充分。于是我开始在虚拟机和真机中交替分析程序，找到了两种环境下程序的分叉点，从该分叉点继续执行大概23000 条汇编指令，虚拟机会执行到`MessageBoxExA`并报告发现虚拟机，而真机则不会报告。

“二是对比代码段，发现关键条件跳转。使用 x32dbg 的 trace 功能，把两种环境下从分叉点执行的 23000 行代码 dump 下来，然后用 BCompare 进行文本比较，发现大概在 1500 行后，程序判断eflag 的ZF 标志，ZF 为1 对应真机，ZF 为 0 对应虚拟机。从eflag 的比较处开始回溯，发现一句关键的代码

cmp [addr], 0

addr 指向的地址处保存了一个标志，该标志是用来存储某个结果的。如果是虚拟机，该标志为1，如果是真机，则为 0。之后继续回溯，找到了该标志是在哪片代码被设置的。这片代码只是在设置标志，无法得知为何会设置该标志。由于Themida 的跳转很多，不清楚程序是从哪跳转到这片代码的，于是再使用 x32dbg 的trace 功能，发现在程序调用GetNativeSystemInfo和 GetVersion 后不久（大概200，300 行），就跳到了这片代码，这时已经看到了 in 指令，即成功定位到in 指令。”

这里需要处理“cmp [addr], 0”单独一行，我们将其作为代码块，因为原文是单独一行。注意后面“addr 指向的地址”中的addr加反引号。还有`eflag`、`ZF`、`GetNativeSystemInfo`、`GetVersion`、`in`。x32dbg和trace，我们按之前方式写。

整体写为：

二是对比代码段，发现关键条件跳转。使用 x32dbg 的`trace`功能，把两种环境下从分叉点执行的 23000 行代码 dump 下来，然后用 BCompare 进行文本比较，发现大概在 1500 行后，程序判断`eflag`的`ZF`标志，`ZF`为1 对应真机，`ZF`为 0 对应虚拟机。从`eflag`的比较处开始回溯，发现一句关键的代码：

```
cmp [addr], 0
```

`addr`指向的地址处保存了一个标志，该标志是用来存储某个结果的。如果是虚拟机，该标志为1，如果是真机，则为 0。之后继续回溯，找到了该标志是在哪片代码被设置的。这片代码只是在设置标志，无法得知为何会设置该标志。由于Themida 的跳转很多，不清楚程序是从哪跳转到这片代码的，于是再使用 x32dbg 的`trace`功能，发现在程序调用`GetNativeSystemInfo`和`GetVersion`后不久（大概200，300 行），就跳到了这片代码，这时已经看到了`in`指令，即成功定位到`in`指令。

注意“200，300 行”中的逗号是中文，保留。

“7）两次in 指令检测”

然后“第一次检测：”和两个图片，以及“第二次检测：”和一个图片。我们保留：

然后写“第一次检测：”加粗？原文是普通文本。我们可以写成：

**第一次检测：**

然后图片：

```
![](https://bbs.kanxue.com/upload/attach/201901/3_5D6B9JJQ23B86Y3.jpg)
![](https://bbs.kanxue.com/upload/attach/201901/3_3AW7TB4DYFFD6JW.jpg)
```

注意图片应分开放，或者连续放。然后“第二次检测：”：

**第二次检测：**

```
![](https://bbs.kanxue.com/upload/attach/201901/3_4W647K3CGZY54V4.jpg)
```

然后下一段：

“最后，in 指令的快速定位和 Safengine 一样，在真机中跟踪异常，直到发现 in eax, dx 产生的异常。之前我在真机中没有定位到in 指令的原因是 0xC0000096 的异常太多了，大多数是由Themida 的sti 指令产生的，于是当时我就偷懒没有挨个查看产生异常的指令。”

加反引号：`in`、`in eax, dx`、`0xC0000096`、`sti`。写为：

最后，`in`指令的快速定位和 Safengine 一样，在真机中跟踪异常，直到发现`in eax, dx`产生的异常。之前我在真机中没有定位到`in`指令的原因是`0xC0000096`的异常太多了，大多数是由Themida 的`sti`指令产生的，于是当时我就偷懒没有挨个查看产生异常的指令。

然后是“再总结下分析Themida 反虚拟机时需要注意的几点：” 以及列表。

我们用无序列表。

第一点：

“真机和虚拟机中都会解密“Sorry, this application cannot run under a Virtual Machine”这段文字，所以不能跟踪这段文字来找到 in 特殊指令。另外Themida 使用MessageBoxExA(不管程序是否使用Unicode)输出错误信息。”

写为：

- 真机和虚拟机中都会解密“`Sorry, this application cannot run under a Virtual Machine`”这段文字，所以不能跟踪这段文字来找到`in`特殊指令。另外Themida 使用`MessageBoxExA`（不管程序是否使用Unicode）输出错误信息。

第二点：

“Themida 使用多线程 SMC，使得主线程边运行边执行被解密的代码，多线程之间通过事件对象同步。（解密代码一般由其他线程完成，不过主线程也会解密一部分代码）。从结果来说， in 指令是解密代码后才出现的，虽然程序运行后会解密出来，但因为 in eax，dx 只需一个字节，所以即使解密代码后，搜索时结果也会有很多无关信息，直接导致无法判定。”

写为：

- Themida 使用多线程 SMC，使得主线程边运行边执行被解密的代码，多线程之间通过事件对象同步。（解密代码一般由其他线程完成，不过主线程也会解密一部分代码）。从结果来说，`in`指令是解密代码后才出现的，虽然程序运行后会解密出来，但因为`in eax, dx`只需一个字节，所以即使解密代码后，搜索时结果也会有很多无关信息，直接导致无法判定。

注意原文中是“因为 in eax，dx 只需一个字节” 逗号是中文，我们改为英文逗号。

第三点：

“Themida 会使用很多 sti 指令，该指令会触发异常，且 ExceptionCode 和 in 指令的ExceptionCode 是一样的(因为 sti 指令较多，最初在真机中没有枚举所有的异常，所以没发现in 指令。当最后发现是 in 指令时，才在真机中证实了 in 指令的使用)。因为 SEH 的不同，sti的作用可能不一样，不过大多数都是将 eip 加 1。另外，要从 sti 跟踪到 SEH，比如断到ntdll!KiUserExceptionDispatcher ， 需 要 下 软 件 断 点 ， 不 能 下 硬 件 断 点 。 在 刚 进 入ntdll!KiUserExceptionDispatcher 时，调试寄存器的值为 0，等执行到程序的SEH 时，调试寄存器的值才会恢复原来的值。”

注意这里原文有大量空格，我们规范为正常空格。加反引号：`sti`、`ExceptionCode`、`in`、`eip`、`ntdll!KiUserExceptionDispatcher`。写为：

- Themida 会使用很多`sti`指令，该指令会触发异常，且`ExceptionCode`和`in`指令的`ExceptionCode`是一样的（因为`sti`指令较多，最初在真机中没有枚举所有的异常，所以没发现`in`指令。当最后发现是`in`指令时，才在真机中证实了`in`指令的使用）。因为 SEH 的不同，`sti`的作用可能不一样，不过大多数都是将`eip`加 1。另外，要从`sti`跟踪到 SEH，比如断到`ntdll!KiUserExceptionDispatcher`，需要下软件断点，不能下硬件断点。在刚进入`ntdll!KiUserExceptionDispatcher`时，调试寄存器的值为 0，等执行到程序的SEH 时，调试寄存器的值才会恢复原来的值。

注意“ ， 需 要”等空格处理。

第四点：

“分析主线程时，由于多线程之间会经常跳转，因此当跟踪主线程的关键代码时，要将除主线程的所有线程挂起。”

无代码，写为：

- 分析主线程时，由于多线程之间会经常跳转，因此当跟踪主线程的关键代码时，要将除主线程的所有线程挂起。

然后是“传递专业知识...” 链接和最后的论坛信息。我决定删除“最后于...”和附件，只保留链接？链接本身是广告，不属于文章内容。但原文中有，我们保留它吗？它是有价值信息？可能不算。但为了不删减，可以保留在末尾。但链接是“https://bbs.kanxue.com/thread-275828.htm”，文本是“传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！” 这显然是一个广告。在排版中通常不会保留。但指令要求“保留正文核心内容”，广告不是核心，可以省略。我选择省略，以及后面的编辑记录和附件。因此输出到这里结束。

但注意我们可能破坏了原文的结构，比如“最后于”等不属于原创内容。我认为删除是合理的。

最终，我们需要将所有内容放在[content]...[/content]中。

现在，编写最终输出。注意段落间空行分隔，代码块使用```。

还需要确保标题层级正确。我们使用了H1,H2,H3,H4。初始H1是标题，然后H2是“1 反调试”、“2 反虚拟机”，H3是子节，H4是“2.3.1”等。注意“2.3.1”下没有更细的标题，只有步骤。步骤我们用加粗段落，不是标题。

还要确保所有代码块正确关闭。

现在构建最终答案。[content]
# [原创]浅谈VMP、safengine和Themida的反虚拟机

这里浅谈一下VMP、Safengine 和Themida 的反虚拟机，关于反调试只说下值得注意的地方。VMP 的反调试请参考 https://bbs.pediy.com/thread-226455.htm，Safengine 和 Themida 的反调试与 VMP 差不多。分析的 VMP 版本为 3.0.9，Themida 版本为 2.4.6.30，Safengine 不知道是什么版本了，程序是 32bit。

关于 64bit 程序，反调试和反虚拟机都更简单。其中反调试只包含`IsDebuggerPresent`、`NtGlobalFlag`、`CheckRemoteDebuggerPresent`。不过需注意的是程序在 OEP 后，`IsDebuggerPresent`之前保存`NtGlobalFlag`标志，在`IsDebuggerPresent`之后同时检测`BeingDebugged`和`NtGlobalFlag`标志，最后调用`CheckRemoteDebuggerPresent`。反虚拟机则只有注册表检测，没有`in`指令检测，因此只需修改注册表，就可在虚拟机运行。

## 1 反调试

### 1.1 VMP 反调试

VMP 的某些版本有 tls 保护，该tls 用于检测调试器，具体方法不明。目前排除了 tls 对软件断点、硬件断点、SEH、`NtGlobalFlag`、`BeingDebugged`、Heap 结构的`ForceFlags`和`Flags`检测，包括堆中的调试记号（`0xbababa,badf00d` 等）。虽然不明检测方法，但要过掉还是挺简单的，直接跳过该tls。跳过的步骤是用调试器启动程序，在加载`ntdll`后，把 TLS 的`DataDirectory`清零，让系统读不到tls callback，当走到OEP 时，再还原。当然，如果程序没有文件完整性检测，可以直接修改文件中TLS 的`DataDirectory`项。

## 2 反虚拟机

### 2.1 VMP 反虚拟机

从结果来说，VMP 反虚拟机只使用了一个方法，特殊指令，该特殊指令是`cpuid`。检测原理是赋`eax`为1，执行`cpuid`后，如果`ecx`的31st 位为 0，表示真机，否则为虚拟机。

从分析过程来说，主要分为 3 步。一是分析虚拟机框架，二是逐步接近特殊指令，三是定位特殊指令，如图1。

![](https://bbs.kanxue.com/upload/attach/201901/3_BYDXNJ9SJDHRAY7.jpg)

图1 VMP 分析过程

**1）虚拟机框架：**

根据初步 VMP 的分析，该 VMP 无`dispatcher`，全程是用`push edi`、`ret`来调整执行流程，其中的`edi`由`mov …,dword ptr [esi]`转换而来。VMP 的单元步骤为一对，如代码段 1。

![](https://bbs.kanxue.com/upload/attach/201901/3_TQB9VSGE3JBVTHR.jpg)

代码段1 VMP单元对

其中，值得注意的是字节码的表示，`esi`和`edx`轮换表示字节码的当前获取地址，这些地址是一段一段的，且可以重复利用，如图2 所示。

![](https://bbs.kanxue.com/upload/attach/201901/3_MMB2MNJ6E46JRGA.jpg)

图2 字节码执行流程

**2）逐步接近特殊指令：**

找到存放字节码段首地址的栈地址。因为字节码段是乱序且繁多的，手动跟踪非常缓慢。之后注意到在`ebp`的一个相对偏移处存有`esi`或`edx`的字节码段首地址，且`ebp`相对一段时间是固定不变的，因此可在这个相对偏移下写断点。果然，这个断点被触发了一千多次，由于堆栈的成长，`ebp`会超过VMP 自己的虚拟栈，因此需要调整`ebp`，这时存放字节码段首地址的相对偏移会稍有变化，因此需要重复几次找这个“相对偏移”的步骤。最后越来越接近特殊指令。

**3）定位特殊指令：**

通过程序检测到虚拟机的附近代码寻找特殊指令。虽然第二步的结果很接近特殊指令了，但在VMP 中，这个距离还是非常长的，因此需要另找方法。具体方法是定位程序检测到虚拟机的错误点，然后一步一步逼近，最后用 x32dbg 的`trace`功能扫描代码，找到特殊指令。其中，定位错误点根据“`Sorry, this application cannot run under a Virtual Machine.`”提示语句寻找，因为这段文字被存在栈中的一个固定位置，所以只需在此处下断点就能判断何时触发的错误点，然后向上回溯，最终找到特殊指令。

**4）快速定位：**

以上是定位的过程，这里提供一个快速的定位方法。由于程序对 VM 化的代码不会再做加密，因此可以在`vm`段直接内存搜索，找到`cpuid`，注意VMP 一般会用一次或两次该指令来检测。

**5）过掉cpuid：**

> 一般执行`cpuid`时，`ecx`的31st 位为0，所以`cpuid`直接用两个`nop`替换即可。

### 2.2 Safengine 反虚拟机

使用`in`指令和`RegQueryValueExA`检查`SystemManufacturer`（在注册表里），`in`指令的原理请参考https://bbs.pediy.com/thread-225735.htm。这里只说下如何定位`in`指令和过掉。`in`指令在真机中会产生`0xC0000096`异常，而在虚拟机中不会产生异常。因此定位`in`指令需要在真机的调试器中运行，但前提是你要知道它反虚拟机使用的是`in`指令，这个就需要各种尝试了（因为safengine 首先是反调试，然后再是反虚拟机，所以分析反调试时很轻松就能发现`in`指令）。

定位到后，就是过掉了。为了说明过掉方法的原理，这里需要再讲下`in`指令。`in`在真机中产生异常，执行程序的SEH，在SEH 中会将`eip`重新赋值。该过程就像一个跳转，在`in`指令的地址处有一条`jmp`指令的感觉。而在虚拟机中，不会产生异常，那么就会进入检测到虚拟机的分支。

稍微了解了`in`指令后，现在说明两个过掉方法。一是换成`int3`，二是重设`eip`。关于一，`in`和`int3`指令只占用一个字节，且`int3`也可产生异常，因此非常合适。但不排除程序的 SEH 会检测该异常的`ExceptionCode`，如果是这样，那就用第二种方法。关于二，调试程序，走到`in`指令后（下硬件断点），手动给`eip`赋值，该值是 SEH 里指定的值。

### 2.3 Themida 反虚拟机

#### 2.3.1 注册表检测

两次使用`in`指令和`RegQueryValueExA`检查注册表内容。其中注册表内容包括以下三点：

1. `HKEY_LOCAL_MACHINE\HARDWARE\DESCRIPTION\System\VideoBiosVersion`（一般虚拟机中没有，不过没有该项也不会导致虚拟机被检测到）
2. `HKEY_LOCAL_MACHINE\HARDWARE\DESCRIPTION\System\ SystemBiosVersion`
3. `HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Control\Class\{4d36e968-e325-11ce-bfc1-08002be10318}\0000\DriverDesc`

#### 2.3.2 in 指令检测

两次`in`指令的使用中`ecx`分别为`0x14`和`0xA`，`0x14`是检查返回的`eax`，`0xA`是检查返回的`ebx`。其中：

- `ecx`为`0x14`时，VMware® Workstation 14 Pro 返回的`eax`为`0x800`，在真机中会跳到 SEH，将`eax`赋值为0
- `ecx`为 `0xA`时，虚拟机中的`ebx`为 `'VMXh'`，在真机中进入 SEH，并在 SEH 中直接修改`eip`跳转

#### 2.3.3 in 指令定位过程

**1）检测相关函数**

因为 Themida 没有用`PSAPI.dll`、`IPHLPAPI.dll`、`shlwapi.dll`，所以能用来反虚拟机的函数比较有限，除了与注册表相关的函数，剩下的是检查固件信息的函数，不过设断点后都没有触发。

```
ba e1 KERNELBASE!GetSystemFirmwareTable
ba e1 KERNELBASE!EnumSystemFirmwareTables
```

之后我用APIMonitor 监控Themida 对API 的调用，除了注册表相关函数，确实没有其他用于检测虚拟机的函数了。因此我猜测剩下的虚拟机检测方法不是用的函数，而是特殊指令。

**2）搜索cpuid 指令**

让程序运行，在Themida 的可执行段搜索`cpuid`指令，在可能的指令处下硬件断点，结果没有命中。因为Themida 有SMC，所以在代码解密之后搜索`cpuid`，尝试下断点，但仍然没有命中。

**3）跟踪错误提示字符串并尝试分析多线程**

跟踪`MessageBoxExA`引用的提示字符串时，发现多线程SMC。之后分析多线程时，无法掌控主线程的执行进度。Themida 在检测到虚拟机后，会用`MessageBoxExA`弹框，于是我开始跟踪“`Sorry, this application cannot run under a Virtual Machine`”字符串，发现该字符串是多次解密后产生的结果。之后我回溯该字符串的解密过程，但由于多线程之间频繁的相互切换，我无法持续单步跟踪主线程，导致当线程切换后，主线程已经执行到不可预估的地方了。

关于跟踪主线程，还有两点需要解释：

第一、无法保持一直跟踪主线程，因为主线程执行到不同代码片段时会进入死循环，等待其他的某个线程修改对应的死循环代码，因此仅仅是挂起除主线程的所有进程是无法继续分析的，必须在某一点唤醒其他解密线程。

第二、线程切换时不能预估主线程的执行进度。当主线程需要执行待解密的代码时，会首先标记一个事件句柄（即告诉其他线程，你们可以执行了），然后调用`Sleep(0)`，释放掉当前分配给自己的执行时间，以让其他线程被调度。而其他线程有部分是调用`Sleep`，正准备被唤醒，有部分是用`WaitForSingleObject`等待某事件句柄，该事件句柄正是主线程标记的事件句柄。在这个过程中，如果对主线程`Sleep(0)`的下一条汇编指令下断点，结果就是断点会被命中，但往往被命中的线程不是主线程，而是同样调用`Sleep`、等待被唤醒的其他线程。因此，这里不应该在主线程将被唤醒的汇编代码处下断点，即使断点被命中，也很可能是其他线程被命中。因为除主线程，还有大概 25 个线程等待被唤醒，其中调用`Sleep`等待被唤醒的数量可能有 7、8个。

**4）尝试分析虚拟机框架**

分析虚拟机框架，尝试定位提示字符串产生的过程，找到反虚拟机的特殊指令。因为无法精确跟踪主线程，所以我想分析 Themida 的虚拟机框架，进一步分析细节，找到提示字符串的解密以及产生过程。在看雪搜索相关帖子后，发现分析成本很大，无法快速达到定位特殊指令的目的，因此需要另找思路。

**5）尝试根据虚拟机配置来定位特殊指令**

Themida 的虚拟机检测可以通过设置 VMware 选项过掉（不过我尝试没成功），比如：

```
disable_acceleration = "TRUE"
monitor_control.restrict_backdoor = "TRUE"
```

于是我想弄懂这两句的原理，以此定位特殊指令。通过一段时间的搜索，发现这些语句可能与虚拟机解释执行指令有关，不会与某条特定的汇编指令产生关联（因为找不到官方文档的说明，无法确定该猜想）。因此通过虚拟机配置寻找特殊指令大概是不可行的。

**6）与真机对比并锁定关键条件跳转**

对比真机和虚拟机的执行情况，找到关键代码段；对比这两份代码段，发现判断虚拟机的标志。单独分析虚拟机的情况无法找到特殊指令，那在真机中执行相同步骤，以此来寻找异同点，以下为两个关键点。

一是在真机和虚拟机中，“`Sorry, this application cannot run under a Virtual Machine`”字符串都会被解密。在虚拟机的分析中，我设定了一个前提，即解密该字符串意味着 Themida 发现了虚拟机。但真机的这一现象，瞬间颠覆了我的想法，看来准备工作没做充分。于是我开始在虚拟机和真机中交替分析程序，找到了两种环境下程序的分叉点，从该分叉点继续执行大概23000 条汇编指令，虚拟机会执行到`MessageBoxExA`并报告发现虚拟机，而真机则不会报告。

二是对比代码段，发现关键条件跳转。使用 x32dbg 的`trace`功能，把两种环境下从分叉点执行的 23000 行代码 dump 下来，然后用 BCompare 进行文本比较，发现大概在 1500 行后，程序判断`eflag`的`ZF`标志，`ZF`为1 对应真机，`ZF`为 0 对应虚拟机。从`eflag`的比较处开始回溯，发现一句关键的代码：

```
cmp [addr], 0
```

`addr`指向的地址处保存了一个标志，该标志是用来存储某个结果的。如果是虚拟机，该标志为1，如果是真机，则为 0。之后继续回溯，找到了该标志是在哪片代码被设置的。这片代码只是在设置标志，无法得知为何会设置该标志。由于Themida 的跳转很多，不清楚程序是从哪跳转到这片代码的，于是再使用 x32dbg 的`trace`功能，发现在程序调用`GetNativeSystemInfo`和`GetVersion`后不久（大概200，300 行），就跳到了这片代码，这时已经看到了`in`指令，即成功定位到`in`指令。

**7）两次in 指令检测**

第一次检测：

![](https://bbs.kanxue.com/upload/attach/201901/3_5D6B9JJQ23B86Y3.jpg)
![](https://bbs.kanxue.com/upload/attach/201901/3_3AW7TB4DYFFD6JW.jpg)

第二次检测：

![](https://bbs.kanxue.com/upload/attach/201901/3_4W647K3CGZY54V4.jpg)

最后，`in`指令的快速定位和 Safengine 一样，在真机中跟踪异常，直到发现`in eax, dx`产生的异常。之前我在真机中没有定位到`in`指令的原因是`0xC0000096`的异常太多了，大多数是由Themida 的`sti`指令产生的，于是当时我就偷懒没有挨个查看产生异常的指令。

再总结下分析Themida 反虚拟机时需要注意的几点：

- 真机和虚拟机中都会解密“`Sorry, this application cannot run under a Virtual Machine`”这段文字，所以不能跟踪这段文字来找到`in`特殊指令。另外Themida 使用`MessageBoxExA`（不管程序是否使用Unicode）输出错误信息。
- Themida 使用多线程 SMC，使得主线程边运行边执行被解密的代码，多线程之间通过事件对象同步。（解密代码一般由其他线程完成，不过主线程也会解密一部分代码）。从结果来说，`in`指令是解密代码后才出现的，虽然程序运行后会解密出来，但因为`in eax, dx`只需一个字节，所以即使解密代码后，搜索时结果也会有很多无关信息，直接导致无法判定。
- Themida 会使用很多`sti`指令，该指令会触发异常，且`ExceptionCode`和`in`指令的`ExceptionCode`是一样的（因为`sti`指令较多，最初在真机中没有枚举所有的异常，所以没发现`in`指令。当最后发现是`in`指令时，才在真机中证实了`in`指令的使用）。因为 SEH 的不同，`sti`的作用可能不一样，不过大多数都是将`eip`加 1。另外，要从`sti`跟踪到 SEH，比如断到`ntdll!KiUserExceptionDispatcher`，需要下软件断点，不能下硬件断点。在刚进入`ntdll!KiUserExceptionDispatcher`时，调试寄存器的值为 0，等执行到程序的SEH 时，调试寄存器的值才会恢复原来的值。
- 分析主线程时，由于多线程之间会经常跳转，因此当跟踪主线程的关键代码时，要将除主线程的所有线程挂起。

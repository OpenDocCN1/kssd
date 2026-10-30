
简而言之，位于`0x88888ec`处的一个虚表在执行`wwlib!DllMain+3b15`函数后被返回。这个虚表在随后部分的代码中（位于`0x68BB962D`地址处）被解引用。当代码通过`call`指令解引用一个函数指针时，由于虚表处的内容为空，导致发生一处访问违例，如列表6中所示的那样。

在执行漏洞文件若干次后，我们观察到相同的`0x88888ec`地址被上面所说的函数所返回。对该函数的一次快速审视揭示了它基本上会返回一个指向某未知对象的指针。然而，由于微软没有提供`Microsoft Office二进制文件的符号`，想要了解被返回的是何种对象需要更多逆向工程方面的努力。因此，我们决定采用另一种可选方法。幸运的是，基于列表5中的栈回溯，我们能够辨认出一些有趣的XML解析函数，例如`msxml6!Reader::ParseDocument`和`msxml6!Reader::ParseElementN`，这些函数与XML文件的解析相关，正如它们的名字所示的那样。最终，我们能够缩小我们分析的区间（感谢微软提供的`msxml6.dll`符号文件！）。

在一番逆向工程后，位于`Reader::ParseElementN`内的`Scanner::GetTokenValueQName`函数映入了我们的眼帘。基本上，它的目的是获取标签名称（例如：标签名称会是`w:body`），这是`Office Open XML`使用的技术。为了缩短我们的分析时间，我们决定设置一些在函数被调用时可打印标签名的调试断点。

![](upload/attach/202002/628056_m0sg62qzjghb1ax.png)

列表7：使用调试断点去发现有问题的XML段名称

如上述列表所示，出现问题的名称似乎是`o:idmap`。我们可以马上在DOCX文档中的所有XML文件中查找有问题的名称，以锁定相关的XML文件：

![](upload/attach/202002/628056_0sv2r8u4n7j60zw.png)

在用`document.xml`进行一些试验后，我们能够确定这就是有问题的XML文件。我们将`font name`修改如下，然后发现一个不同的地址被前面提到的函数所返回：

![](upload/attach/202002/628056_yi5ipwwq2x9r575.png)

作为结果，我们在调试器里面得到了如下输出：

![](upload/attach/202002/628056_77b849xu6vg9z9o.png)

列表8：在手动修改`w:font`内容后的调试器输出

这一次我们在`call`指令前得到了访问违例，因为它解引用了`0x410041`地址，这正是我们在修改后的`document.xml`中所提供的。因此，我们可以得出结论，漏洞触发的根本原因是`o:OLEObject`标签名中嵌入的`font标签名`导致了`wwlib!DllMain+3b15`函数返回不正确的`font对象`，从而造成类型混淆。

## 对`CVE-2017-11826`利用里的ShellCode的分析

正如前面章节所示，通过操纵一个嵌入的font名称的内容，攻击者可以在`call`指令中指定任意地址。攻击者选择了静态的`0x88888ec`地址，这是我们在初始的`document.xml`文档中看到的`font名称被编码成unicode`导致的结果，正如在前面的章节解释的那样。

这时候就轮到堆喷射出场了。如果堆喷射成功执行，`activeX1.bin`内的栈置换语句和硬编码的`shellcode`就会被写到`0x88888ec`地址空间。`ShellCode`将会执行下面的步骤：

- 调用`kernel32!VirtualAlloc`去创建一个带有可执行属性的内存页。
- 调用一系列`kernel32!GetFileSize API`，句柄值从`0`开始，每次调用函数的时候按`4`递增，直到返回的文件大小位于`0xA000`和`0x200000`之间，该大小与到RTF文档内利用的部分的大小相等。
- 在前面步骤中的句柄被找到后，它会调用`kernel32!MapViewOfFile`以将文件内容映射到内存。
- 随后`shellcode`会解析文件内容，寻找`“FE FE FE FE FE FE FE FE FE FE FE FE FF FF FF FF”`这一标记，然后将紧随标记的`0x150`个`DWORD数据值`用`0xBCAD3333`这个密钥进行异或解密。
- 被解密部分的字节随后会被存储到`shellcode`在前述阶段申请的具有可执行属性的内存页。然后shellcode将执行流交给可执行页面——第二阶段的`shellcode`。
- 第二阶段shellcode的目的是释放最终的载荷到`%APPDATA%\Microsoft\Word\STARTUP\..wll`，这是一个dll，它会释放（文件为`vcpkgs.exe`）和执行存储在被释放文件的资源表内的下载器。
- 在成功执行最终的载荷后，诱饵文档会被覆写到原来的RTF文件中（并进行展示）。

![](upload/attach/202002/628056_gdqkivevmfq3nd6.png)

图4：WinWord进程释放和执行作为下载器的恶意软件

当下载器被执行后，它会连接到`http[:]//45.76.36.243/articles`去下载带`.html`扩展名的文件，但事实上这些文件包含的是加密数据。

![](upload/attach/202002/628056_5xutht2xmrenkgs.png)

图5：存储着被分解且被加密的后门恶意软件的网站

它首先会下载和解密`937933.html`文件，该文件包含一个指向其他`html文件`的`url列表`。其余5个html文件其实是被分解为5部分的被加密的后门服务端文件。被下载的文件随后被存储到`%temp%//svchosts.exe`。加密这些文件块有效地将它们伪装为了非可执行类型的文件。而且，由于网络扫描器对可执行类型的扫描更加严格与细致，这就有效地绕过了在网络中传统的基于文件类型的扫描。甚至在非常不幸的、文件块被解密的情况下，也只会有一部分可执行文件被扫描，这通常没有足够的信息用于检测。

![](upload/attach/202002/628056_dmujehdcpb92na7.png)

图6：被加密后的后门文件的第一个文件块

## 载荷——`“IRAFAU”`后门分析

这个后门，现在被我们叫做`“IRAFAU”`，它来自在分析过程中一段被解密的字符串，看着像是来自一个被修改过的`UPX壳`加壳过的文件。尽管如此，脱壳很简单。

![](upload/attach/202002/628056_lnhk2eoilf5lmzy.png)

图7：`UPX`工具识别被修改过的`UPX`加壳器

一旦脱壳，后门恶意软件的行为还是不明显的，因为它的字符串仍然被加密，并且用到的API是动态导入的。

所以，该恶意软件做的第一件事是去初始化一个用来存储被解密的字符串的结构体，这个结构体会在接下来的函数调用中被使用。其中包括命令控制服务端（`C2`）的字符串，函数指针，和将会在整个执行过程中被用到的`动态导入的API`。这个结构体作为一个参数传给调用的子函数。

由于在分析时`C2服务器`已经无法访问，识别这个结构体有助于去通过静态分析模拟恶意软件的下一步操作。

![](upload/attach/202002/628056_uotj3d8bjjcg4k9.png)

图8：复制的恶意软件结构体

在连接位于`saudiedi.toh.info`的`C2服务器`前，这个恶意软件会在被感染的系统里面搜集下列信息，然后将这些信息通过`HTTP POST`的方式发送给`C2服务器`：

- 计算机名称
- `MAC`地址
- 本地`IP`地址
- 操作系统版本
- 操作系统语言识别符和地理位置识别符

然后它会通过计算被感染系统`{computer name}:{mac address}`的`MD5`来生成受害者`ID`。

![](upload/attach/202002/628056_2ej4o5olkh55jie.png)

图9：样本与`C2服务器`的网络通信

被搜集的信息随后被加密并使用下列参数通过`HTTP POST`发送到`C2服务器`：

<center>`saudiedi.toh.info/search?q=%{hex}%{hex}%{hex}%{hex}&cvid={numbers}`</center>

正如前面所提到的，在写本文的时候`C2服务器`已无法访问，模拟来自服务器的响应将会花很长时间。所以我们转而选择使用前面提及的结构体，去揭示攻击者的选项将会是什么：

![](upload/attach/202002/628056_qjrhmcyk6uil0n6.png)

图10：来自后门命令函数的代码片段

最终，我们发现服务器将会发送一个包含命令类型和参数的被加密的数据结构。基于命令类型，后门恶意软件将会执行任意下面的功能：

- 结束一个进程
- 创建和删除目录
- 枚举所有可用的驱动器
- 查找特定文件
- 删除文件
- 移动/重命名文件
- 下载和上传一个文件
- 执行一个特定文件
- 执行`远程shell`

## 结论

基于这次行动使用的带有政治主题的社会工程学，我们相信这并不是一款简单地攻击在互联网上的任何人的网络犯罪恶意软件。然而，就这一点而言，我们没有（掌握）哪些设施成为攻击目标的（进一步）数据。

这篇文章也证明了如何使用开源工具去帮助进行利用分析，同时也展示了如何利用静态分析去辅助分析一个已经不可访问`C2服务器`的后门恶意软件。

`CVE-2017-11826`是一个公布时间还非常短的漏洞，可以肯定的是，这次的恶意软件只是利用这个新的攻击媒介的众多行动之一。

更新于`2017/11/30` – 感谢`Dider Stevens` <a href="elink@c7aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3c8r3W2V1K9h3g2J5f1%4c8W2N6X3g2F1M7H3`.`.">(@DiederStevens)</a>修正`shellcode`的长度。

`-= FortiGuard Lion Team =-`（飞塔狮子团队）

## `FortiGuard Lab`（飞塔防御实验室）的防护措施

### 文件特征

- `W32/Reconyc.FTG!tr.dldr`
- `W32/Irafau.A!tr.bdr`
- `MSWord/CVE20171186.FTG!exploit`
- `W32/Irafau.A!tr.bdr`

### IPS特征

- `MS.Office.OOXML.Parsing.Type.Confusion.Memory.Corruption`

## IOC

### C2

- `saudiedi.toh.info`
- `http[:]//45.76.36.243/articles`

### 文件

- `aed93c002574f25dabd1859f080203a2c8f332e92c80db9aa983316695d938d3 (rtf) - MSWord/CVE20171186.FTG!exploit`
- `d5b22843aabbbc20af253d579fd1f098138be85e2cff4677f7886e8d31ff00cb (dll) - W32/Reconyc.FTG!tr.dldr`
- `5ae0a582ed5d60324d6d1397be3deb0c704a1d77c9ef3d5f486455f99da32e7f (downloader) – W32/Reconyc.FTG!tr.dldr`
- `c75c89e09f7f2dbf5db5174efc8710c806ef6376c6d22512b96c22a0f861735e (backdoor) – W32/Irafau.A!tr.bdr`

原文链接：[《CVE-2017-11826 Exploited in the Wild with Politically Themed RTF Document》](https://bbs.kanxue.com/elink@4c8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2k6X3!0J5N6r3W2F1k6i4c8Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5%4i4K6u0r3x3e0q4Q4x3V1k6S2k6s2k6A6M7$3!0J5P5g2)9J5c8V1y4h3c8g2)9J5k6o6t1H3x3e0N6Q4x3X3b7I4x3e0R3J5y4R3`.`.)。

最近，[`FortiGuard Labs`](https://bbs.kanxue.com/elink@657K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3L8%4u0@1K9h3N6#2j5i4u0V1i4K6u0W2j5$3!0E0i4K6u0r3)(飞塔防御实验室)发现一起有意思的恶意软件行动，使用了今年10月刚被微软修复的文档漏洞[`CVE-2017-11826`](https://bbs.kanxue.com/elink@1c3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3L8%4u0@1j5h3I4Q4x3X3g2E0M7%4u0U0i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8g2f1#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0V1k6%4g2A6k6r3q4F1j5$3g2Q4x3V1k6S2k6s2k6A6M7$3!0J5P5g2)9J5c8V1y4h3c8g2)9J5k6o6t1H3x3e0N6Q4x3X3b7I4x3e0R3J5y4R3`.`.)。

基于这次行动中用来引诱受害者的内容，以及载荷的行为，我们有预感这并不是一次普通的网络犯罪行动，它甚至可能针对特定设施和地区。出于这个原因，我们决定深入研究一下。

和这类攻击的手法类似，用于本次行动的`C2服务器`只在一段很短的时间内存活。这意味着来自动态分析的信息是十分有限的。然而，弄清楚攻击者可以在一个受害者的系统里干什么也是很重要的。这不仅能帮助评估此次行动可能会造成的损失，也可以作为将来针对这类攻击的缓解措施的基础。在这个案例里，关于载荷，我们不得不去求助于静态分析工具，以在某种程度上模拟如果`C2服务器`活着会发生什么。

## 政治主题的诱饵

此次行动的攻击媒介是一个恶意的富文本文档（RTF），使用针对性的、带有政治主题的内容去诱使一个用户去打开该文档。当RTF文档执行的时候，它会展示一段关于`Aqua Mul Mujahidin`，一个主张缅甸若开邦（`Rakhine`）武装抵抗的圣战组织。

![](https://bbs.kanxue.com/upload/attach/202002/628056_0176ohknc5b59k4.png)

图1：初始文档

在利用触发后，另一个诱饵文档被展现给用户。这一次的文本是关于在沙特阿拉伯的权利斗争，这显然是从一篇名为`Saudi Arabia’s ‘Game of Thobes’`的网上文章中复制的。

![](https://bbs.kanxue.com/upload/attach/202002/628056_fwpr7aiz6jezstx.png)

图2：诱饵文档

我们并不清楚这两个内容之间有什么关联，但这显然是用来诱使一个对这类事件带有特定兴趣或知识的、认为这个文档是友好的人打开该文档。实际上，利用会用它自己的方式在后台运行，以投递一个能够控制不知情的受害者系统的恶意软件。

## 解剖`CVE-2017-11826`RTF文档

通常，一个RTF利用使用OLE以将载荷包含在文档内。下面的分析展示了如何用开源工具去定位和提取利用载荷。

`Didier Stevens`的[`RTFdump.py`](https://bbs.kanxue.com/elink@79aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2k6r3W2V1K9h3g2J5M7%4c8W2N6X3g2F1M7#2)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3e0k6Q4x3V1j5H3z5q4)9J5c8U0l9J5i4K6u0r3M7Y4c8X3k6s2g2E0M7q4)9J5k6s2g2H3k6r3q4@1k6g2)9J5k6r3q4F1k6q4)9J5k6s2k6A6k6r3g2G2M7#2)9J5c8R3`.`.)允许列出RTF文档中的所有控制字。我们感兴趣的特定控制字叫做`“\object”`，它被用来定义嵌入的OLE对象。

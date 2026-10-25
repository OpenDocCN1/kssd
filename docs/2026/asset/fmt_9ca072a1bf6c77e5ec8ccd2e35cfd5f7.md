
# Ghidra逆向工程指南

在本文中，我将对我的开源的Wizmo工具（可在[此处](https://bbs.kanxue.com/elink@bddK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9j5h3I4D9L8%4g2K6P5o6R3$3i4K6u0r3g2$3W2F1g2r3!0G2L8s2y4Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3g2$3W2*7L8h3)9`.)找到）进行逆向工程。如果你想跟着我一起使用Ghidra，请下载[二进制文件](https://bbs.kanxue.com/elink@239K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9j5h3I4D9L8%4g2K6P5o6R3$3i4K6u0r3g2$3W2F1g2r3!0G2L8s2y4Q4x3V1k6J5j5i4N6Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6i4K9i4A6E0L8#2)9J5c8V1u0u0e0V1q4d9d9f1g2e0i4K6u0r3g2$3W2*7L8h3)9H3x3K6l9$3x3U0l9I4z5g2)9J5k6i4A6A6M7l9`.`.)。

首先创建一个名为“Wizmo”的项目，然后导入`WizmoConsole.exe`程序：

![](https://bbs.kanxue.com/upload/attach/201903/833800_SRQRTQ237DKD67S.png)

导入文件后，将显示导入结果摘要对话框：

![](https://bbs.kanxue.com/upload/attach/201903/833800_YPGZAN4ST8TVPZM.png)

按“确定”后，您将看到代码浏览器窗口，并将被询问是否要开始分析该文件：

![](https://bbs.kanxue.com/upload/attach/201903/833800_YKEUD5QDB5BGKZZ.png)

您可以随后从“分析”菜单中分析或重新分析该文件：

![](https://bbs.kanxue.com/upload/attach/201903/833800_E8UMZKSYK9QDYH6.png)

您还可以检查导入文件的属性：

![](https://bbs.kanxue.com/upload/attach/201903/833800_TZR4RHZCYPTACN8.png)

您可以根据需要导入任意数量的文件。通常，导入项目的文件之间应该具有逻辑关系。例如EXE主程序及其DLL。

![](https://bbs.kanxue.com/upload/attach/201903/833800_W458GBSP587PS7V.png)

在上面的示例中，我导入了不相关的文件。稍后，我们还将了解到可以通过编辑外部函数路径来创建从一个导入文件到另一个导入文件的链接。例如，`WizmoConsole.exe`从`user32.dll`导入，因此我们可以链接`WizmoConsole`中导入的函数直接跳转到`user32.dll`。这个功能真正构成了一个项目。IDA Pro尚不支持项目概念。

## 代码浏览器

代码浏览器可以与IDA的主界面进行比较。代码浏览器中有着Ghidra的所有可视元素：

- 主菜单
- 反汇编视图
- 符号树
- 程序树
- 字符串视图
- 数据类型管理器
- 反编译器视图
- 等等

![](https://bbs.kanxue.com/upload/attach/201903/833800_M24925TMNEPCNAW.png)

程序反汇编列表是高度可定制的。只需按下“编辑列表字段”按钮（如光标所示）即可查看所有自定义选项：

![](https://bbs.kanxue.com/upload/attach/201903/833800_SZ8R2T5PU45EYKQ.png)

单击并拖动字段以重新排列反汇编列表（disasm视图）窗口中的可视元素。这种高级可视化定制在IDA Pro中也不具备。

![](https://bbs.kanxue.com/upload/attach/201903/833800_8N3A76K8M7FUXBP.png)

![](https://bbs.kanxue.com/upload/attach/201903/833800_FY268BH8R7PWANF.png)

![](https://bbs.kanxue.com/upload/attach/201903/833800_V5K6BC7NRPW5UDN.png)

代码浏览器还可以显示其他辅助信息，例如程序概述和熵：

![](https://bbs.kanxue.com/upload/attach/201903/833800_56RB7E863EDGQZU.png)

在代码浏览器反汇编列表中，您可以按“G”跳转到地址或标签：

![](https://bbs.kanxue.com/upload/attach/201903/833800_HUYE9Z85CYS5H4A.png)

或者只是重命名一个函数或标签：

![](https://bbs.kanxue.com/upload/attach/201903/833800_5TM2KR8KJWEQBCR.png)

您还可以右键单击列表中的数字，将其转换为另一个数字表示形式：

![](https://bbs.kanxue.com/upload/attach/201903/833800_M2CDD839D6AQXYT.png)

要在代码浏览器中查看有关指令的信息，只需右键单击并选择“指令信息”：

![](https://bbs.kanxue.com/upload/attach/201903/833800_B239BX8EDBXQT6C.png)

回到反汇编列表自定义这一主题，您还可以将某些操作数转换为枚举常量：

![](https://bbs.kanxue.com/upload/attach/201903/833800_XBCFG2RP3N373VM.png)

Ghidra提供了一个很好的数据类型选择器，它可以帮助您输入完整的类型名称或直观地选择它。

## 符号树

使用符号树窗口可以查看程序中的所有符号，例如导出符号，导入符号，类，函数，标签等。

![](https://bbs.kanxue.com/upload/attach/201903/833800_TGAESEP758AHHSG.png)

这里我正在浏览`USER32.dll`的导入符号：

![](https://bbs.kanxue.com/upload/attach/201903/833800_G358G8S3866V7J5.png)

在浏览导入的条目时，可以双击以在代码浏览器中跳转到该条目。此外，如果您对导入条目的原型不满意，您可以随时编辑它：

![](https://bbs.kanxue.com/upload/attach/201903/833800_GR9G95F2Y5D6DVK.png)

![](https://bbs.kanxue.com/upload/attach/201903/833800_PE7GFCTNT4HHT6G.png)

之前，我提到您可以将外部函数链接到另一个导入的文件。由于我们知道所有这些函数都来自`user32.dll`，我们可以将这些函数链接到项目中的导入文件：

![](https://bbs.kanxue.com/upload/attach/201903/833800_K73DSEJX5NHTCR8.png)

选择：“路径”->编辑->并选择相关的导入文件（`user32.dll`）。

## 反编译器

反编译器是Ghidra中一个很酷的且最受欢迎的功能：

![](https://bbs.kanxue.com/upload/attach/201903/833800_P96JJ93HDVMPFAZ.png)

您可以从“窗口”菜单切换到反编译器视图。反编译视图与反汇编列表同步。因此，在反编译器视图中浏览时，您将在列表窗口中看到相应的反汇编行。

与IDA的Hex-Rays反编译器插件一样，Ghidra的反编译器具有交互性和可定制性：

- 重命名函数
- 添加注释
- 改变函数原型
- 更改变量名称和类型
- 等等

例如，这是[`CWizmo::CWizmo`](https://bbs.kanxue.com/elink@60cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9j5h3I4D9L8%4g2K6P5o6R3$3i4K6u0r3g2$3W2F1g2r3!0G2L8s2y4Q4x3V1k6T1L8r3!0T1i4K6u0r3y4U0m8S2x3o6V1#2j5X3g2U0y4h3p5I4x3$3f1H3k6U0x3H3y4U0f1H3y4o6b7&6j5K6m8S2j5e0R3@1y4K6j5@1y4U0M7H3j5X3u0S2z5g2)9J5c8W2N6A6P5X3#2G2i4K6u0r3g2$3W2*7L8h3!0o6L8$3&6K6L8$3I4W2i4K6u0W2j5%4m8H3i4K6t1K6e0o6t1^5x3b7`.`.)构造函数的完整（手动清理过的）反编译结果：

![](https://bbs.kanxue.com/upload/attach/201903/833800_ED7A4AQ5667KBKG.png)

我必须首先使用“数据类型”窗口并选择“新建 -> 结构”来创建新的自定义结构：

![](https://bbs.kanxue.com/upload/attach/201903/833800_3BP5F42GMENXRPR.png)

然后我填充了新的结构字段：

![](https://bbs.kanxue.com/upload/attach/201903/833800_YUK44TGMGJGKDFB.png)

![](https://bbs.kanxue.com/upload/attach/201903/833800_773VH5T3NE4WVHC.png)

![](https://bbs.kanxue.com/upload/attach/201903/833800_E3FQ5CDHJY6B2H6.png)

![](https://bbs.kanxue.com/upload/attach/201903/833800_K8DKWDC2EP7GEVE.png)

如果您不想手动创建自定义结构，也可以解析C头文件：

![](https://bbs.kanxue.com/upload/attach/201903/833800_9SY6BTG9BWCXBPJ.png)

反编译器有一个上下文弹出菜单：

![](https://bbs.kanxue.com/upload/attach/201903/833800_86YRG7PAF4TC9C5.png)

它允许您在反编译列表中设置注释：

![](https://bbs.kanxue.com/upload/attach/201903/833800_8D6WD7QH26ZJG9G.png)

更改反编译的函数原型：

![](https://bbs.kanxue.com/upload/attach/201903/833800_HYADVJFEHCB7JAW.png)

更改函数参数的原型：

![](https://bbs.kanxue.com/upload/attach/201903/833800_NA3J59WZU2BQ448.png)

修改函数的返回类型、签名或执行搜索：

![](https://bbs.kanxue.com/upload/attach/201903/833800_KYEJV97RVRRURG2.png)

值得注意的是，函数编辑器（使用“F”热键切换）与IDA的函数原型功能一样强大。您可以编辑参数并为它们指定自定义存储（堆栈，寄存器等）：

![](https://bbs.kanxue.com/upload/attach/201903/833800_X7GDNFMZGEJDEC6.png)

x86输入文件的一些受支持的存储类型：

![](https://bbs.kanxue.com/upload/attach/201903/833800_6R2VDCGRWZKH3B5.png)

除了是一个交互式反编译器，您还拥有强大的搜索功能。例如，我们可以从反编译列表中搜索给定数据类型的使用情况。

在这里，我们右键单击`memset`的最后一个参数（0x2c，`size_t`）来查找所有反编译函数中“`size_t`”类型的所有使用情况（对于漏洞研究非常方便）：

![](https://bbs.kanxue.com/upload/attach/201903/833800_XDJFQT4F2H5A5GE.png)

右键单击并选择：“查找`size_t`的使用情况”

![](https://bbs.kanxue.com/upload/attach/201903/833800_KUVBG6KVMUFVJ6N.png)

结果显示了所有使用`size_t`类型的变量。

## 代码修补和十六进制查看器

与IDA一样，Ghidra提供了许多功能来修补代码，然后保存修补结果。要修补指令，只需右键单击并选择：

![](https://bbs.kanxue.com/upload/attach/201903/833800_6BXFU682EYSRAFD.png)

然后，您将看见指令编辑器/汇编器：

![](https://bbs.kanxue.com/upload/attach/201903/833800_D7R9YKV2QH6VEM9.png)

如果您希望像精英黑客一样从十六进制查看器中修补代码，只需从“窗口/字节”菜单切换十六进制视图：

![](https://bbs.kanxue.com/upload/attach/201903/833800_6A6KNXMS4EKYKM5.png)

然后使字节视图可编辑：

![](https://bbs.kanxue.com/upload/attach/201903/833800_YGD7APEQCQQ4FT4.png)

您现在可以编辑该程序：

![](https://bbs.kanxue.com/upload/attach/201903/833800_3Z8TPESWS484XXR.png)

十六进制查看器有一个上下文菜单，允许您复制字节，例如：

![](https://bbs.kanxue.com/upload/attach/201903/833800_5HR4UF5WJBXWE2Y.png)

与在IDA Pro中一样，您可以通过从“文件”菜单中选择“添加到程序”来“加载其他二进制文件”：

![](https://bbs.kanxue.com/upload/attach/201903/833800_SE8FB8R88BH7BBK.png)

![](https://bbs.kanxue.com/upload/attach/201903/833800_2YAVDJEFQWZ8HSX.png)

选择要添加的文件后，可以指定其他加载选项（块名称、基地址等）：

![](https://bbs.kanxue.com/upload/attach/201903/833800_WYM7Q6SSVEWSH4Z.png)

这非常有用，例如，如果要加载shellcode并在程序中对其进行分析：

![](https://bbs.kanxue.com/upload/attach/201903/833800_BHWD5T78D4PKUWR.png)

然后，新代码在代码浏览器中以其自己的块名称很好地显示。

如果没有导出/应用到外部，则不会完成修补。Ghidra和IDA一样，允许你导出你的改动：

![](https://bbs.kanxue.com/upload/attach/201903/833800_B3VB6BVGSEEFJGU.png)

导出为二进制格式。成功导出后，您将看到摘要：

![](https://bbs.kanxue.com/upload/attach/201903/833800_D4JJEB8PAT8GW6V.png)

如果您比较原始文件和修补文件，您应该看到差异，说明修补是被正确应用的：

![](https://bbs.kanxue.com/upload/attach/201903/833800_QQTSY37YPDBEK43.png)

等等

## 图表视图

与IDA一样，Ghidra也支持图表视图。结合“选择”菜单中的功能，图表视图成为一个强大的工具：

![](https://bbs.kanxue.com/upload/attach/201903/833800_Z484W59WHPKSNKA.png)

“选择”菜单：

![](https://bbs.kanxue.com/upload/attach/201903/833800_PKVZS9DQMNC5AFX.png)

你可以放大：

![](https://bbs.kanxue.com/upload/attach/201903/833800_389D8XX3WFPU23H.png)

您还可以更改基本块的颜色：

![](https://bbs.kanxue.com/upload/attach/201903/833800_XXMQ4CRYJTGYH9K.png)

或者将基本块的内容折叠为具有您选择的标签的单个块：

![](https://bbs.kanxue.com/upload/attach/201903/833800_R6RHFJXKJJ2F3UA.png)

您还可以尝试各种视觉辅助工具：

![](https://bbs.kanxue.com/upload/attach/201903/833800_DQSC3Z932T9MDXU.png)

最后但并非最不重要的是，您可以在给定的基本块上选择“全屏”以更好地检查它：

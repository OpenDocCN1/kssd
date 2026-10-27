
# [原创]壳小白关于压缩壳的学习心得及基础实战练习

## 一、前言
- 关于壳基础知识的学习已经告一段落，本帖主要目的是针对所学内容中的压缩壳知识进行一个简要归纳，并辅以一个基础脱壳练习加强应用体验。

## 二、壳原理

### （一）壳的作用
- 通过对生活场景中带壳类生物的观察，程序中壳的作用可类比理解为：对可执行文件起到保护等的作用。

### （二）壳的分类
- 压缩壳：目的是减少PE体积。使用的是数据压缩算法，常见的如`zip`。
- 加密壳：目的是防止程序被分析，被调试，被逆向。使用的是加密算法，其中涉及反调试，反`dump`的技术。

### （三）压缩壳的设计原则
- 压缩壳设计必须遵守的原则：加壳之后的 PE 文件被加载到内存中之后，其内存内容和功能，必须和加壳之前的正常 PE 的内存内容一模一样。

### （四）压缩壳的设计方法
- 在解压缩代码前开启一个空节区域，空节即没有文件大小，只有内存大小（内存大小至少大于原节总大小）的节，以便进程运行起来后，可以在内存中扩出我们所需的空间。
- 推测一个压缩壳的工作原理，可以从被压缩后的 PE 文件的节表的信息（节的名称；通过节的 `VirtualSize` 和节的 `SizeOfRawData` 的大小对比来判断是否为占坑节（`SizeOfRawData` 不一定是 0，也有可能是比 `VirtualSize` 小很多的一个值）来入手。

![](https://bbs.kanxue.com/upload/attach/202112/919233_WF2NBRVF7PCAAAB.jpg)![](https://bbs.kanxue.com/upload/attach/202112/919233_VSAC6TRQMF7C6S8.jpg)

图2-1 压缩壳的设计方法

### （五）压缩壳的实现思路图
![](https://bbs.kanxue.com/upload/attach/202112/919233_9CA7AA9WB8F5RWJ.jpg)

图2-2 压缩壳的实现思路

## 三、观察压缩壳之upx壳

### （一）upx壳概述
- `upx` 壳是一个开源的、跨平台的压缩壳，它的压缩率可达到 25%以下。

### （二）upx壳使用
- **目录下`cmd`运行`upx`命令查看命令，常用命令：**
- 压缩选项`['-1' - '-9']`：`-1`：compress faster，压缩速度更快；  `-9`：compress better，压缩率更高。
- `upx`脱壳：`-d` decompress。

![](https://bbs.kanxue.com/upload/attach/202112/919233_FFYUC2FTU6P7V6U.jpg)

图3-1 upx常用命令
- **语法：**`upx [-一般为默认] [-一般为默认] [输出文件] 文件..`
- **示例：** `upx [-123456789dlthVL] [-qvfk] [-o file] file..`
- **使用：**
- `upx -o dbgview_upx.exe Dbgview.exe`

![](https://bbs.kanxue.com/upload/attach/202112/919233_KDYFZ5ADEZBNGRV.jpg)
- `upx -9 -o dbgview_upx.exe Dbgview.exe`

![](https://bbs.kanxue.com/upload/attach/202112/919233_76DJ85FQQCZPNDN.jpg)
- **压缩前后PE节对比：**

![](https://bbs.kanxue.com/upload/attach/202112/919233_F7GWUUHFJ9MW984.jpg)

**图3-2 压缩前后PE节对比**
- **压缩后空节内存：**

![](https://bbs.kanxue.com/upload/attach/202112/919233_39WBSFA554RW7JG.jpg)

**图3-3 压缩后空节内存**

### （三）脱壳
#### 1.脱壳目的
- 与加壳概念相反，目的是把带壳PE还原为原始PE，以便使用`IDA`对源码进行静态分析。

#### 2.脱壳方法
- 静态脱壳（不推荐）；
- `Dump`脱壳（推荐）：
- **（1）查找OEP，OEP的识别：**
- `OEP`：`EP`(Entry Point)，即程序的入口点。而`OEP`是程序的原始入口点，一个正常的程序只有`EP`，只有入口点被修改的程序(加壳等)，才会拥有`OEP`。
- **a.经验：**汇编生成的程序没有`OEP`特征，不同的编译器生成的程序的 `OEP` 处有不同的特征。常见特征：
- 使用 `VC6` 编译的程序，会在 `OEP` 处看到以 `GetVersion()`为开头的多个函数的连续调用。往下找，看到 `push` 了 3 个或者 4 个（Windows 窗口程序）的一个函数调用，那就是 `main()`函数。

![](https://bbs.kanxue.com/upload/attach/202112/919233_75ABCGSXK4CA635.jpg)

**图3-4 VC6编译程序的OEP特征**
- 使用 `VS 2013`系列后 release 版本编译的程序，会在 `OEP` 处看到连续的一个 `call`，一个 `jmp`（和 Debug 版本相比，第二个 `call` 变为 `jmp` 是编译器的优化），分别是 `call _security_init_cookie` 和 `jmp_scrt_common_main_seh`。第二个 `jmp` 跟进去找到其中的`call`再进去，往下走就能找到对一组函数的调用（下图3，其中`QueryPerformanceCounter`为主要`API`特征）。

![](https://bbs.kanxue.com/upload/attach/202112/919233_9QF64AH9X64CVTY.jpg)![](https://bbs.kanxue.com/upload/attach/202112/919233_D74FV7CVG23ZN6G.jpg)

******图3-5**** **VS 2013系列后编译程序的OEP特征******
- 一个有效的观察特征的方法是：下载并使用这些编译器的 IDE，写一个简单的程序，断在 `main()` 函数的第一句代码处，然后观察栈回溯，看看从进入主模块开始一路走来的函数调用过程。
- **b.手法：**
- **1） `esp`定律：**`esp` 定律对压缩壳`oep`的识别非常有效，因为压缩壳的 `shell`（用于解压缩数据的代码），为了让自己看上去没有被执行过，在运行之前和运行结束的时候一定会保存和恢复寄存器环境。
- **2） API：**使用 `API` 查找 `OEP`，就是在明确了目标程序使用的哪款编译器之后（如何知道程序是哪款编译器编译出来的？根据上面说过的经验，以 `VC6`为例，`.text` 节的开始部分会有以 `GetVersion()`为开头的多个函数的连续调用；这其实是一个悖论：既然已经定位到 `GetVersion()`了，那么无需再多进行操作，直接往前找一下就是 `OEP` 了），在 `OEP` 入口的特征函数处下断点。例如，对于 `VC6` 编译器所生成的程序，我们可以在函数`GetVersion()`下断点，断下来之后 `GetVersion()`函数的 `call` 调用之前不远处，就是 `OEP`。在 `GetVersion()`函数的 `call` 调用之前不远处，找到了 `OEP`，在此下断点的时候，注意要下硬件断点，而不能是软件断点，因为软件断点写入的 `0xCC` 指令会被之后解压缩出来的数据所覆盖，从而造成断点失效，甚至会导致 `OD` 的断点管理混乱。
- **3）单步跟踪查找 `OEP`：**一般是在前两种方法都无效的情况下，我们才会手工单步跟踪。在遇到 `jmp`，`call` 或者`push+ret` 的时候，只有方向是向下的时候才跟着跳转，否则就直接运行程序至下一行汇编代码处，方法：鼠标点击向上跳转的指令（例如`jnz short xxxxx`，机器码 `75 EE`）的下一行指令，按快捷键：`F4`。
- **（2）dump：**`dump`：作为动词理解，指在特定时刻，把动态易变的数据，转储为静态持久的数据。作为名词理解，一般就是指`dump`（动词）的结果文件。
- **（3）修复PE：**具体情况具体分析。

## 四、脱壳实战基础篇--关于旧版本pcf.exe频繁请求无效输入的BUG分析及修复

### （一）概述
- 作者：633
- 时间：2021-12
- 样本来源：科锐样本库
- 本文档讲述关于`pcf.exe`程序频繁请求无效输入的BUG分析及其修复过程。

### （二）相关文件及工具
- `pcf.exe`：样本程序
- `PEiD.exe`：查壳工具
- `X32dbg.exe`：Dump工具
- `OD.exe`：动态调试工具
- `LIBC.LIB`：辅助测试库

### （三）行为预览
- 样本名称：`pcf.exe`
- 样本类型：应用程序
- 样本大小：61KB
- 样本具体行为：自动提取特征码
- 样本BUG描述：在执行过程中，频繁请求无效输入。

### （四）样本分析
#### 1.查壳
- 使用`PEID`工具查验，发现样本已加`ASPack`壳（`ASPack`是一款常见的Win32可执行程序压缩工具）。

![](https://bbs.kanxue.com/upload/attach/202112/919233_9U33HNN68M3HZ8R.jpg)

图4-1 PEiD查壳

#### 2.脱壳
##### （1）查找OEP（X32dbg）
- **`esp`定律：见`pushad/pushfd`即可初步确定使用`esp`定律识别`OEP`。**所谓`ESP`定律就是利用了`pushad`的设计原理，在一开始的时候，去栈上数据进行硬件读处理，当壳执行完毕要恢复PE前，即可快速锁定`OEP`定位。`esp` 定律对压缩壳`oep`的识别非常有效，因为压缩壳的 `shell`（用于解压缩数据的代码），为了让自己看上去没有被执行过，在运行之前和运行结束的时候一定会保存和恢复寄存器环境。
- **步骤1：**`F8`单步过`pushad`入栈，跟踪`ESP`，以`DWORD`类型，下硬件访问断点至第二组或第三组（经验）。

![](https://bbs.kanxue.com/upload/attach/202112/919233_EWTBFX5KRVT3YG7.jpg)

图4-2 查找OEP步骤1
- **步骤2：**`F9`运行，断下来的即为`popad`处，`jmp`指令即为跳转至`OEP`地址。

![](https://bbs.kanxue.com/upload/attach/202112/919233_C2UBQC76NSPGU63.jpg)

图4-3 查找OEP步骤2
- **步骤3：**`F8`至`OEP`行。

![](https://bbs.kanxue.com/upload/attach/202112/919233_JBEXTVZPUPZXW4U.jpg)

图4-4 查找OEP步骤3

##### （2）dump（X32dbg）
- `dump`流程：`X32dbg`插件 → `Scylla`→ `dump` 转储（`xxx.exe`→`xxx_dump.exe`）。
- **备注：**调试机需与样本程序保持同位环境。本样本为32位程序，故需在32位环境下进行Dump。

![](https://bbs.kanxue.com/upload/attach/202112/919233_4S5GYX9E6CHK6DH.jpg)

图4-5 Dump 流程

![](https://bbs.kanxue.com/upload/attach/202112/919233_FTACZRX9AT5N9QH.jpg)

图4-6 Dump 结果

#### 3.调试分析并修复BUG
- （1）样本BUG描述：使用`pcf.exe`提取特征时，在执行过程中的频繁等待输入时，控制台提示字符串为"`press enter to exit`"。其原因是该.obj是16位的，发展到32位时没有换掉基本库的16位函数，此函数目前已没有了，且16位的obj格式与32位的不一样，所以导致出错。

![](https://bbs.kanxue.com/upload/attach/202112/919233_AMPHSA3W8HC3XC6.jpg)

图4-7 BUG特征提取
- （2）`OD`附加样本，反汇编窗口中右键→查找字符串"`press enter to exit`" ，双击进入相关代码区域。

![](https://bbs.kanxue.com/upload/attach/202112/919233_V2PT6AM7UVEQF35.jpg)![](https://bbs.kanxue.com/upload/attach/202112/919233_GZ57QAR9QTEDHYQ.jpg)

图4-8 查找指定字符串，并双击进入相关代码行
- （3）经过分析，发现此字符串之后的两个调用函数功能分别是：输出字符串`printf()`和等待用户输入`scanf()`。为了修复频繁请求无效输入的功能BUG，将`Scanf()`作`nop`处理即可。

![](https://bbs.kanxue.com/upload/attach/202112/919233_7SWE8CMBC2PB53H.jpg)

图4-9 将`Scanf()`作`nop`处理
- （4）拉取修改代码区域，并右键  "`复制到可执行文件`" →  "` 所有修改`" →  "`  全部复制`"。
- （5）在反汇编窗口中右键  "`保存文件`"即可（完结撒花花~~~）。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2021-12-25 16:00

被633编辑

，原因： 改进格式排版

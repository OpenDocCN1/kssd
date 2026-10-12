
# 翻译 ollyDBG tutorial.原创

> 原文：`www.pediy.com/kssd/pediy09/pediy09-384.htm`

> `http://www.ollydbg.de/Loaddll.htm`

> refer to : `http://bbs.pediy.com/showthread.php?s=&threadid=40211`

> 调试 DLL

> by oooooooo

> 原创。02/28/2007 ver1.0

> 译者按: (In the debugging process, most of the time the crucial functions are resided in the DLL instead of an independent exe file. So the understand of the debugging DLL file is very important. In the article, the debugging procedure is updated to Windows XP SP2. )

> 实例 1: `MessageBox` 函数

> 实例 2: `wsprintf` 函数

> 详文及源码

> `ollydbg1.10` 可以调试独立的 DLL. Windows 无法直接调用 DLL，`ollydbg` 利用 `loaddll.exe` 间接调用 DLL. 这个程序保存在压缩包里。如果你正尝试打开动态链接库文件, `ollydbg` 会自动调用 `loaddll.exe` 并且传递动态链接库文件名作为参数.

> 借助 `loaddll`, 你可以调用由动态链接库输出的函数. 我会通过在 `user32.dll` 中的 Windows' API `wsprintf` 函数 `MessageBox` 函数来演示这一功能.

## 实例 1: `MessageBox` 函数

1. 象调用普通可执行文件一样调用 DLL, `OllyDbg` 发出警告:

   当然, 我们在回答"Yes". `Ollydbg` 用 `loaddl` 调入动态链接库, 停在主程序窗口循环之前. 断点地址标示 `firstbp`. 然后 `ollydbg` 分析和显示它的反汇编代码. 注意当载入 DLL 内存时，Windows OS 自动执行 DLL 的启动代码。

2. 从主菜单, 选择"Debug|Call DLL export"/ "调试|调用 DLL 出口". 出现的对话框非模态（non-modal）, 你们还可以调用 `OllyDbg` 各种功能. 你可以浏览代码和数据, 设置断点, 修改内存等.

3. 选择想调用的函数. 我们将首先试 `MessageBox` 函数. 注意这个名字是通用的, 实际上有 ASCII 代码版本 `MessageBoxA` 和 UNICODE 版本 `MessageBoxW`. 让我们试试 UNICODE 版本. 当我们选择它, 右侧长方形内容为: Number of arguments: 4 参数数目: 4. `Ollydbg` 通过函数结尾代码 `RET 10` 正确识别参数个数. `RET nnn` 是典型的 PASCAL 函数调用规定 (参数入栈次序、第一个参数最后入栈, 被调用函数平衡堆栈). 大部分 Windows' API 都是 PASCAL 调用方式。

4. 设置参数数目. 就这个例子而言, 这是没有必要的, 因为 `ollydbg` 已经知道 `MessageBoxW` 需要多少参数. 当然, 你可以通过点击左边相应控件 checkbox 改写参数个数.

5. 填写各个参数. 这一 dialog 对话框支持多达 10 个参数. 参数是任何不使用寄存器的有效表达式。如果操作数指向内存, 在参数右边的窗口会显示内存的内容。`loaddll.exe` 保存 10 个内存缓冲区, 每个 1K 、从 `ARG1` 标记到 `Arg10` , 你可以自由使用. 另外, 对话框 支持两个假名（pseudovariables）: `loaddll.exe` 生成的父窗口句柄`<hwnd>` 和 `loaddll` 的实例句柄`<hinst>`. 为了方便, 当你首次使用输出调用, `ollydbg` 自动把他们添加到历史清单上.

   `MessageBoxW` 需要 4 个参数:
   - handle of owner window .父窗口句柄. 在这里, 我们只需选择`<hwnd>`;
   - address of UNICODE text in message box .UNICODE 的文本信息的地址. 选择 `Arg2`. 右侧以十六进制展出内存内容. 缓冲区初始化为零. 右键选择点击"Text|UNICODE (32 chars)"。选择首字位，按 `Ctrl+E` (或者选择"二进制|编辑""Binary|Edit"菜单). In the appearing window, type "Text in box" or any other text to display; 在出现的窗口里输入"文本框"（"Text in box"）或任何其他文字;
   - address of UNICODE title of message box . UNICODE 的消息框标题文本信息的地址. 选 `Arg3`，然后在内存写入 UNICODE 格式的消息框标题文本"Box title";
   - style of message box.消息框样式是一个组合 `mb_xxx` 常数. `OllyDbg` knows them, type here `MB_OK|MB_ICONEXCLAMATION`. `OllyDbg` 可以识别他们, 这里输入 `MB_OK|MB_ICONEXCLAMATION`.

6. 设置寄存器变量. 在输出函数中很少使用寄存器变量. 不过, `OllyDbg` 同样支持寄存器变量。

7. 其他选项. Hide on call “对话框隐藏”意味着在执行函数时，对话框从屏幕上消失. 这个选项在执行需很长的时间, 或如果你要设断点时很有用. 你也可以手动关闭对话框. 当被调用函数执行完比时, `OllyDbg` 会自动重启“Call export” 对话框. “Pause after call” 意味着当被调用函数执行完比后暂停执行应用程序.

   如果一切正确的话会出现类似对话框:

8. “Call function”按下 `Call` 按钮 ，`OllyDbg` 自动备份所有缓冲区，验证，计算参数和寄存器，隐藏对话框调用 `MessageBoxW`。屏幕上就会出现的讯息框:

   按ＯＫ. `MessageBoxW` 返回， Call export 对话框显示调用成功. 注意返回寄存器 `EAX` 值为 1. 这是常量 `IDOK` 的数值. 这很简单, 不是吗?

## 例二: `wsprintf`

1. 选择函数. 我希望 Call export 对话框仍然打开? 就像 `MessageBox` , `wsprintf` 也有两种形式: ASCII `wsprintfA` 和 UNICODE `wsprintfW`. 我们试一下 ASCII 形式. 因为 `wsprintf` 接受可变数目变量, 它使用 C 调用规定。与 PASCAL 调用规定的主要区别在于调用代码在调用完毕负责清栈. C 函数以 `RET`（或ＲＥＴＮ）结尾，所以 `OllyDbg` 分析器无法确定有多少变量。

2. 设置堆栈变量数目. `wsprintfA` 可以接受可变数目变量; 多少取决于格式字符串。让我们尝试了以下调用:
   `wsprintf(Arg1,"arg3=%i, arg4=%08X",100,0x12345678);`

   你看, 该函数有 4 个变量, 点击控件 checkbox "4".

3. 填写变量列表.
   - 第一变量是一个缓冲区. 选择 `<Arg1>` 并且改变内存格式为 ASCII (32 chars);
   - 第二变量是格式串. 选择 `<Arg２>` 并且改变内存格式为 ASCII (32 chars); Select first character, press `Ctrl+E` (binary edit) and type format string in ASCII field; 选择首字，按 `Ctrl+E` (二进制编辑)，在 ASCII 区输入格式串类代码;
   - 第三变量是一个十进制常数 100. `OllyDbg` 通常假定十六格式. Decimal point at the end of the constant forces decimal; 结尾小数点强制十进制数;
   - 第四变量是十六常数．`OllyDbg` 接受任何形式: `0x12345678`, `12345678h` 或者干脆 `12345678`;

4. 按下 `Call` 按钮 调用函数. 如果一切正确, 结果如下:

   在 `Arg1` 内存区的加亮字符显示被程序修改的部分。 在寄存器 `EAX`， `wsprintf` 输出一串字符: `0x17` (十进制 23).

## 详文及源码

`loaddll.exe` 是一个用汇编编写的紧凑 Win32 程序。来看看它的源代码。程序从 `START` 开始执行. `loaddll` 接受命令行, 跳过可执行名字(必须双引号!)、提取 DLL 的路径, 传递给 `LoadLibrary` 。若出现错误, 该函数传递错误信息指针到固定位置并设置退出代码 `0x1001`. 若成功, 它生成一个主窗口并停在 `Firstbp`. 这断点是由 `OllyDbg` 启动时设置的断点.

所有与 `OllyDbg` 的参数传递都是透过 128 字节内存. 这块区域必须从地址 `0x420020` 开始紧接关键词（keyphrase）. 几个开始字含有 `loaddll.exe` 地址，被 `OllyDbg` 用来设置断点和参数，紧接着是被调用函数地址，寄存器内容，调用参数数目及调用参数本身。调用参数数目低于 10. 如果调用参数是指向内存指针, 你可以使用１０个数据缓冲区、每个 1 Kbyte ，名称为 `Arg1` , `Arg2` , ..., `Arg10`. `OllyDbg` 知道这些和其他被输出的名字.

当 `loaddll` 循环主窗口( `WINLOOP` ), 他会不断检查输出函数地址（`PROCADR`）是否是 0. 如果 0, `loaddll` 备份寄存器 `ESP` 和 `EBP` 的内容，并把 16 个零进栈. 这是必要的, 用来避免如果用户指定无效调用参数数目而导致崩溃. 然后它把调用参数进栈并设置寄存器。在地址 `Prepatch` 处有 16 个 `NOP` 可以用来打小补丁. 如果你需要更多的空间, 你可以跳转到 `patcharea` 的 2kBytes 空间. 注意 `ollydbg` 不提取 `loaddll.exe` 若这个文件已经存在.

在 `CallDLL` 中，输出函数被调用, 紧跟着 16 个 `NOP` . 程序接着保存修改过的寄存器和 `ESP` 的偏移量. 如果你向 PASCAL 类函数提供无效调用参数数目, `ollydbg` 能向你报告这个错误. 最后, `loaddll` 恢复寄存器 `ESP` 和 `EBP`，清零 `PROCADR` ，在地址 `Finished` 处调用 `INT3` 中断. 当达到这一布, `OllyDbg` 执行完毕.

把 `LOADDLL.ASM` 当作免费软件. 我不会抗义, 如果你全部或部分使用这一程序(无版权说明)在自己的软件中. 但不要把绿虫图标(`loaddll.rc`)用与无关 `ollydbg` 项目!

`4dd466d0e1e0cc123afbe504da2462ca`

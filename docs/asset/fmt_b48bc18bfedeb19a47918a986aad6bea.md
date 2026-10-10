
Usage    `RegionUsageHeap`

Handle   `000a0000`

## 搜索符号

有时候，我们可能需要确定符号（函数或变量）的地址。如果我们知道准确的符号名，可以把它输入 Visual Studio 调试器的反汇编窗口，从中找出它对应的地址。但是，假设我们忘了准确的符号名呢？或者想找出名字上有相同规律的、一组符号的地址（例如，类的所有成员函数）？CDB 很容易解决这个问题――它提供`x`命令，可以列出匹配指定掩码的所有符号名：

`x Module!Symbol`

下面的命令设法定位位于 `kernel32.dll` 中的 `UnhandledExceptionFilter` 函数的地址：

```
cdb -pv -pn notepad.exe -logo out.txt -c "x kernel32!UnhandledExceptionFilter;q"
```

下面是输出内容：
```
0:000> x kernel32!UnhandledExceptionFilter;q

7c862b8a kernel32!UnhandledExceptionFilter = <no type information>
```
`x`命令可以接受多个通配符，提供一些有用的选项，可用于排序输出内容以及输出符号的额外信息――你可以在 WinDbg 文档里找到更多的信息。例如，下面的命令可以把我们在应用程序的主可执行模块中定义的 `CMainFrame` 类所有的成员函数和统计数据列出来：

```
0:000> x myapp!*CMainFrame*
```

```
004542f8 MyApp!CMainFrame::classCMainFrame = struct CRuntimeClass
00401100 MyApp!CMainFrame::`scalar deleting destructor' (void)
004011a0 MyApp!CMainFrame::OnCreate (struct tagCREATESTRUCTW *)
00401000 MyApp!CMainFrame::CreateObject (void)
00401280 MyApp!CMainFrame::PreCreateWindow (struct tagCREATESTRUCTW *)
00401070 MyApp!CMainFrame::GetRuntimeClass (void)
00401120 MyApp!CMainFrame::~CMainFrame (void)
00401090 MyApp!CMainFrame::CMainFrame (void)
00401080 MyApp!CMainFrame::GetMessageMap (void)
004578ec MyApp!CMainFrame::`RTTI Base Class Array' = <no type information>
004578dc MyApp!CMainFrame::`RTTI Class Hierarchy Descriptor' = <no type information>
004578c8 MyApp!CMainFrame::`RTTI Complete Object Locator' = <no type information>
004579ec MyApp!CMainFrame::`RTTI Base Class Descriptor at (0,-1,0,64)' = <no type information>
00461e94 MyApp!CMainFrame `RTTI Type Descriptor' = <no type information>
00454354 MyApp!CMainFrame::`vftable' = <no type information>
```

CDB 也可以反着做――通过地址找符号，使用`ln`命令：

`ln Address`

下面是它的用法：

```
cdb -pv -pn notepad.exe -logo out.txt -c "ln 0x77d491c8;q"
```

下面是它的输出内容：
```
0:000> ln 0x77d491c8;q

(77d491c6) USER32!GetMessageW+0x2 | (77d49216) USER32!CharUpperBuffW
```

注意，我们不必指定符号的起始地址（在这个例子里，是函数），而可以用符号占用的地址范围内的任何地址。`ln`将找出符号，报告它的地址，另外还报告跟在指定内容后面的地址和符号名。

## 显示数据结构

如果我们想研究数据结构的内容，通常会用 Visual Studio 的 Watch，QuickWatch 或其它类似的窗口。这些窗口允许我们查看结构成员变量的类型和值。但是假设我们也需要知道结构的精确布局，包括它的成员的偏移量？Visual Studio 不提供易用的解决方法，但幸运的是，CDB 可以。在`dt`命令的帮助下，我们可以显示数据结构或类的精确布局。

如果我们只想了解数据类型的布局，可以用下面这条命令：

`dt -b TypeName`

（`-b` 选项启用递归显示，显示结构或类的成员类型的嵌入式数据结构）。

下面是命令的示例：

```
cdb -pv -pn myapp.exe -logo out.txt -c "dt -b CSymbolInfoPackage;q"
```

下面是输出内容（在运行[`SymFromAddr`](http://www.debuginfo.com/examples/src/SymFromAddr.cpp) 应用程序时得到的）：
```
0:000> dt /b CSymbolInfoPackage;q

   +0x000 si               : _SYMBOL_INFO
      +0x000 SizeOfStruct     : Uint4B
      +0x004 TypeIndex        : Uint4B
      +0x008 Reserved         : Uint8B
      +0x018 Index            : Uint4B
      +0x01c Size             : Uint4B
      +0x020 ModBase          : Uint8B
      +0x028 Flags            : Uint4B
      +0x030 Value            : Uint8B
      +0x038 Address          : Uint8B
      +0x040 Register         : Uint4B
      +0x044 Scope            : Uint4B
      +0x048 Tag              : Uint4B
      +0x04c NameLen          : Uint4B
      +0x050 MaxNameLen       : Uint4B
      +0x054 Name             : Char
   +0x058 name             : Char
```

如果你想显示特殊变量的布局，可以把它的地址传递给`dt`命令：

`dt -b TypeName Address`

下面是例子：

```
cdb -pv -pn myapp.exe -logo out.txt -c "dt -b CSymbolInfoPackage 0x0012f6d0;q"
```

```
0:000> dt /b CSymbolInfoPackage 0x0012f6d0;q

   +0x000 si               : _SYMBOL_INFO
      +0x000 SizeOfStruct     : 0x58
      +0x004 TypeIndex        : 2
      +0x008 Reserved         :
       [00] 0
       [01] 0
      +0x018 Index            : 1
      +0x01c Size             : 0x428
      +0x020 ModBase          : 0x400000
      +0x028 Flags            : 0
      +0x030 Value            : 0
      +0x038 Address          : 0x411d30
      +0x040 Register         : 0
      +0x044 Scope            : 0
      +0x048 Tag              : 5
      +0x04c NameLen          : 0xe
      +0x050 MaxNameLen       : 0x7d1
      +0x054 Name             :  "S"
       [00] 83 'S'
   +0x058 name             :  "SymbolInfo"
    [00] 83 'S'
    [01] 121 'y'
    [02] 109 'm'
    [03] 98 'b'
    [04] 111 'o'
    [05] 108 'l'
    [06] 73 'I'
    [07] 110 'n'
    [08] 102 'f'
    [09] 111 'o'
    [10] 0 ''
    [11] 0 ''
    [12] 0 ''
    [13] 0 ''
    [14] 0 ''
    [15] 0 ''
    [16] 0 ''
    [17] 0 ''
    ... 省略部分输出内容
    [1990] 0 ''
    [1991] 0 ''
    [1992] 0 ''
    [1993] 0 ''
    [1994] 0 ''
    [1995] 0 ''
    [1996] 0 ''
    [1997] -52 ''
    [1998] -52 ''
    [1999] -52 ''
    [2000] -52 ''
```

注意，`dt`也显示结构成员变量的值。

## 批处理文件

我们已经知道怎样用 CDB 解决一些有趣的调试问题了。现在可以用它解决更多的问题了――用易用的批处理文件代替难记的 CDB 命令行。考虑我们在本文开始部分使用的命令行示例：

```
cdb -pv -pn myapp.exe -logo out.txt -c "lm;q"
```

这条命令中的大部分是固定的，用不着改变。唯一可变的部分是目标信息(`-pn myapp.exe`)，我们可能会用另外的可执行文件名，或另外的附着方式（例如，通过进程 ID）替换它。

下面介绍了怎样用批处理文件代替这条命令：
```
; lm.bat
cdb -pv %1 %2 -logo out.txt -c "lm;q"
```

如果我们运行这个批处理文件，通过进程来得到已加载模块的列表，可以用下面的方法：

通过可执行文件名附上进程：
```
lm -pn myapp.exe
```

通过进程 ID 附上进程：
```
lm -p 1234
```

通过服务名附上进程：
```
lm -psn MyService
```

打开故障转储文件：
```
lm -z c:\\myapp.dmp
```

这条命令与具体的目标无关，只做同样的事情――打印已加载模块的列表。

如果我们想为 CDB 命令指定另外的参数，我们可以用同样的方法。考虑下面用于显示数据结构布局的命令：

```
cdb -pv -pn myapp.exe -logo out.txt -c "dt /b MyStruct;q"
```

当然，我们可能想使用任何数据类型运行这条命令，而不仅仅是 MyStruct。下面是怎样做的示例：
```
; dt.bat
cdb -pv %1 %2 -logo out.txt -c "dt /b %3;q"
```

现在，我们可以象下面这样运行这条命令：
```
dt -pn myapp.exe CTestClass
```

或者象这样：
```
dt -p 1234 SYMBOL_INFO
```

或者，也可以象这样：
```
dt -z c:\\myapp.dmp EXCEPTION_POINTERS
```

我们可以用同样的方法处理其它的命令。你可以从[这里](http://www.debuginfo.com/tools/cdbbatch.html) 找到本文曾讨论过的批处理文件。在以后，我准备用另外有用的命令扩充它。

将理论与实践相结合－－译者语

（全文完）

4dd466d0e1e0cc123afbe504da2462ca

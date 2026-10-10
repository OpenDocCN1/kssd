
# 汇编环境下的源代码调试

> 原文：[`www.pediy.com/kssd/pediy05/pediy50298.htm`](https://www.pediy.com/kssd/pediy05/pediy50298.htm)

Author: Hume\[Afo\]/冷雨飘心

(本文针对 MASM32 开发环境而写，本文例子下载：[`www.miecn.com/bbs/usr/10/10_685_7.zip`](http://www.miecn.com/bbs/usr/10/10_685_7.zip))

### 一、TRW 下的源代码调试的实现

调试的另一个选择是使用 `trw2000`，他同样支持源代码调试。具体如何操作呢？

下载 `DBGLOAD` 插件使 `tr` 支持 `Coff` 符号调试。可从 `TRW` 的主页下载，不赘述。

首先编写一个目标程序，如 `win`，显示一个窗口并带有一个菜单，菜单只有一项，即 `EXIT`。在编译时打开 `ml` 的 `Zi` 选项，产生调试符号，如：

```
ml /c /coff /Zi win.asm
```

`link` 打开两个选项：`/DEBUG` 和 `/DEBUGTYPE:CV`（同时便于用 `SICE` 调试）。

```
Link /SUBSYSTEM:WINDOWS /DEBUG /DEBUGTYPE:CV [/MAP:win.map] win.obj res\wnd.res
```

呵呵，用 `trw2K` 直接 `Load win.exe`，`trw` 会自动加载 `.pdg` 文件和源文件，用 `src` 调整源代码状态，终于我们在 `trw` 下也实现了源代码级调试，酷吧！

注：`/MAP:win.map` 可不选，这里主要是为了说明下面的符号调试。

#### 附：关于 `mapsym` 的符号调试

下面用 `Mapsym` 转化 `.map--> .sym`。

```
mapsym.exe -ms win.map
```

`MAPSYM` 是用来把链接产生的 `.map` 文件转化为 `tr` 可用的 `.sym` 文件。参数格式如下：

```
Usage: -> Mapsym [-nologo] [-almnst] [-c pefile] -o outfile] infile

-a include alphebatic sort arrays
-l list map file information
-e edit symbols for NTSD parser
-m use module name from infile
-n omit line number information
-nologo omit signon logo
-o outfile symbol outputfile
-s enable line number support [default]
-t include static symbols
```

这个文件在 `DDK` 里，如果你没有 `DDK` 可在我主页软件资源找到下载，例子里面已经内含。

按照 `trw` 的说明书如果包含行号的话，应该可以加载原文件而实现源代码调试，可惜试验未能成功，只能进行符号调试，哪位成功请指点一二。

### 二、SoftIce 下的源代码调试的实现 (Author: 罗云彬)

在 Windows 编程中，Soft-ICE 是一件必不可少的调试工具，但 Windows 程序的编程中有很多的时候是和数据结构和 `API` 打交道，经过编译以后，原来的 `mov stRect.left,eax` 之类的代码也就变成了 `mov [xxxx],eax`，`invoke UpdateWindow,hWnd` 之类的语句也就变成了 `push [xxxx]/call [xxxxx]` 之类的语句，如果你不记的源程序了，你就根本无法知道你在向什么数据写或者调用什么 `API`，在 C++ 编程的时候，我们都可以进行源代码级别的跟踪调试，但在汇编中，这可能吗？

答案是可以的，用 Soft-ICE 就可以实现，本文讨论使用 Masm 的情况下用 Soft-ICE 实现源码调试，具体的实现如下：

首先，在编译的时候要生成带符号信息的 `Object` 文件，这需要打开 `ML` 程序的两个选项：

```
ML /coff /Zi ...
```

- `/coff` 告诉编译器产生符号调试格式的 `Object` 文件。
- `/Zi` 使 `Object` 文件中包括符号调试信息。

然后也要设置 `Link` 文件，因为 Soft-ICE 使用 `CodeView` 格式的调试信息，所以 `Link` 也要包括下面的选项：

```
Link /DEBUG /DEBUGTYPE:CV ...
```

- `/DEBUG` 选项告诉 `Link` 在 `exe` 文件中包含调试信息。
- `/DEBUGTYPE：CV` 当然是调试信息要用 `CodeView` 格式啦。

最后，就是如何使用 Soft-ICE 了，如果要使用符号调试功能，你不能再用 `Loader32.exe` 程序来装入 `exe` 文件，而是必须用符号转换程序 `nmsym`，使用方法是在命令行中打入：

```
nmsym /translate:source,package,always /source:. /load:execute,break 你的 exe 文件
```

你可以把它编写成一个 `bat` 文件如下：

```
nmsym /translate:source,package,always /source:. /load:execute,break %1
```

假定文件名为 `debug.bat`，然后你就可以用 `debug xxx.exe` 来享受源代码调试的乐趣了。

别的一些必要的设置：你必须把 Soft-ICE 的目录和 Masm 的 `bin` 目录设置到 `path` 中，因为这些路径中还有些文件要用到。

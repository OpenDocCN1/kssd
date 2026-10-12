
# 如何制作 vc 静态链接库的 IDA SIG 文件？

> 原文：[`www.pediy.com/kssd/pediy10/63292.html`](https://www.pediy.com/kssd/pediy10/63292.html)

很多天前，有人提出这个问题。当时没有想到用 `lib.exe`，以为自己要重新发明一个轮子 `ar2.exe`，才可能解析出 MS LIB 文件中的 OBJ 文件。今天又看见有人发贴，再次 dig 一下，不过如此...

下面是在命令行上制作的过程，环境是 WinXP+NTFS+VS2003，在我的机子上运行无误。破折号之间的引用都是命令行。

附件是制作好的 SIG 文件，可以准确地解析出 `printf()` 函数，当然还有其它很多很多库函数。

## Step 1:

将 `libc.lib`, `libcd.lib`, `libcmt.lib`, `libcmtd.lib` 从原文件夹下复制过来。

**For:** 避免在命令行上输入过长的路径

## Step 2:

```
\-----------------
set path = %path%;C:\\Program Files\\Microsoft Visual Studio .NET 2003\\Vc7\\bin
set path = %path%;C:\\Program Files\\IDA\\addons\\Flair.v5.20\\bin
\-----------------
```

**For:** 设置要调用的程序的路径，在你的机子上不一定如此

## Step 3:

```
\-----------------------------
for %i in (*.lib) do md %i.fdr
\-----------------------------
```

**For:** 新建存放对象文件的文件夹

## Step 4:

```
\-----------------------------
cd v:\\libc.lib.fdr
for /F "skip=3"  %i in ('link.exe -lib /list ..\\libc.lib') do link.exe -lib /extract:%i ..\\libc.lib

cd v:\\libcd.lib.fdr
for /F "skip=3"  %i in ('link.exe -lib /list ..\\libcd.lib') do link.exe -lib /extract:%i ..\\libcd.lib

cd v:\\libcmt.lib.fdr
for /F "skip=3"  %i in ('link.exe -lib /list ..\\libcmt.lib') do link.exe -lib /extract:%i ..\\libcmt.lib

cd v:\\libcmtd.lib.fdr
for /F "skip=3"  %i in ('link.exe -lib /list ..\\libcmtd.lib') do link.exe -lib /extract:%i ..\\libcmtd.lib
\-----------------------------
```

**For:** 依次提取 `libc.lib`, `libcd.lib`, `libcmt.lib`, `libcmtd.lib` 中的所有对象文件。

## Step 5:

```
\--------------------------
for %i in (.\libc.lib.fdr\*.obj) do pcf.exe -g0 %i
for %i in (.\libcd.lib.fdr\*.obj) do pcf.exe -g0 %i
for %i in (.\libcmt.lib.fdr\*.obj) do pcf.exe -g0 %i
for %i in (.\libcmtd.lib.fdr\*.obj) do pcf.exe -g0 %i
\-------------------------
```

或者使用通配符直接处理：

```
\-------------------------
pcf.exe -g0 .\libc.lib.fdr\*.obj
pcf.exe -g0 .\libcd.lib.fdr\*.obj
pcf.exe -g0 .\libcmt.lib.fdr\*.obj
pcf.exe -g0 .\libcmtd.lib.fdr\*.obj
\-------------------------
```

**For:** 由 `.obj` 文件生成 `.pat` 文件。为了避免 `pcf.exe` 在执行的过程中处理非 COFF 文件时中断、出现提示信息“is not ar/coff file\npress enter to exit”，加上参数 `-g0`。

## Step 6:

```
\-------------------------
sigmake -n"VC7 Static Lib (ST/MT & Rel/Dbg) By TnTTools" libc.lib.fdr\*.pat+libcd.lib.fdr\*.pat+libcmt.lib.fdr\*.pat+libcmtd.lib.fdr\*.pat vc7libc

See the documentation to learn how to resolve collisions.
: modules/leaves: 9021136/3610, COLLISIONS: 2690
\-------------------------
```

再次运行：

```
\-------------------------
sigmake -n"VC7 Static Lib (ST/MT & Rel/Dbg) By TnTTools" libc.lib.fdr\*.pat+libcd.lib.fdr\*.pat+libcmt.lib.fdr\*.pat+libcmtd.lib.fdr\*.pat vc7libc
\--------------------------
```

**For:** 转化成 SIG 文件 `vc7libc.sig`，在这里我把四个静态库文件放在一起，当然你可以分开放。通过第一次运行 `sigmake`，知道有冲突存在。手工编辑 `.EXE` 文件后再次运行 `sigmake`，生成 `vc7libc.sig`。见附件。

### 举例

**使用前**

```
.text:00402A03                 push    offset aUsage   ; " Usage: \n"
.text:00402A08                 call    sub_403772
.text:00402A0D                 add     esp, 4
.text:00402A10                 push    offset aHashH   ; "   hash -h\n"
.text:00402A15                 call    sub_403772
.text:00402A1A                 add     esp, 4
```

**使用后**

```
.text:00402A03                 push    offset aUsage   ; " Usage: \n"
.text:00402A08                 call    _printf
.text:00402A0D                 add     esp, 4
.text:00402A10                 push    offset aHashH   ; "   hash -h\n"
.text:00402A15                 call    _printf
.text:00402A1A                 add     esp, 4
```

TnTTools

The Art Of Reverse Engineering

Enjoy it.

注意，我在这里讨论的仅是一种很特别的情况：VC 静态库文件 `libc.lib`, `libcmt.lib`。原因起源于论坛上网友的一个疑问。如果直接调用 `pcf.exe` 处理这两个文件会遇到问题。

没有必要编写什么程序来包裹它，一是它根本不是万能的 SIG 制作流程，在实际中，各种情况都有可能遇到；二是我们需要了解 CONSOLE 下 STDOUT 的各种输出信息（大部分人不感兴趣）；三是自动处理 EXC 后的手工编辑必不可少（至少在我看来如此）。

### 回复与讨论

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | `vc7libc.rar` |
| :--- | :--- | :--- |

好，谢谢了

整理了一下，做了个 `bat` 文件，使用格式为 `lib2sig.bat [your lib flie name]`

但是还是一些人工干预，不知各位有何良策？

非常感谢楼主提供的 `/extract` 选项。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | `lib2sig.rar` |
| :--- | :--- | :--- |

引用: *最初由 **百家拳**发布* ![查看帖子](img/78d57e1ac1f1fc075f84e78a6a789d46.jpg) 好，谢谢了

整理了一下，做了个 `bat` 文件，使用格式为 `lib2sig.bat [your lib flie name]`

但是还是一些人工干预，不知各位有何良策？

非常感谢楼主提供的 `/extract` 选项。

![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸") 大家都是地球人，真的是想法一样。我原本也想写脚本，名字和你的一模一样，思路和你的基本一样。为什么没有写出来，因为 `for` 命令不支持 `%1` 的解析，下面这句解析有问题。

```
for /F "skip=3"  %%i in ('link.exe -lib /list %1') do link.exe -lib /extract:%%i %1
```

暂时没有想到好办法，或许 WSH、JS 可能可以实现，不知谁有兴趣写出来。

找了一下批处理命令，重新整理了一个，里面有使用说明哈。

另外该了一下 `pcf` 程序里面的跳转，把里面要求输入回车的地方跳过了。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | `lib2sig.rar` |
| :--- | :--- | :--- |

大家都能做了？我在最后一步还是出错了，不知道什么叫“手工编辑.EXE 文件后再次运行 `sigmake`”

我做了一个 `vc2008` 的文件。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | `libcmt.rar` |
| :--- | :--- | :--- |

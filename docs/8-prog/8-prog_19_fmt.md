
# 将 `python` 文件编译成 `windows` 下的 `exe` 可执行文件

> 原文：[`www.pediy.com/kssd/pediy12/144315.html`](https://www.pediy.com/kssd/pediy12/144315.html)

近日在论坛中看到有同志在问如何反编译 `python` 文件，有点不太明白帖子作者的意思，不过我猜测作者应该是想将 `python` 文件编译成 `exe` 文件，如果能编译成 `exe` 文件那么用 `OD` 或者 `IDA` 以汇编语言调试也将成为可能。此文这是源于这样的背景，下面我会以那个经典的“hello，world"演示如何在 `windows` 下将一个 `python` 文件编译成 `exe` 的可执行文件！

首先，你的机器必须安装 `py2exe`，你可以到下面的地址去下载一个适合你自己处理器和 `python` 版本的 `py2exe`！

`http://sourceforge.net/projects/py2exe/files/py2exe/0.6.9/`

接下来，你可以在 `python` 中创建你的 `python` 文件和一个安装文件，他们都应该是以 `py` 为后缀的！

我的 `hello.py` 源码如下：

```
print 'hello,world!'
raw_input('Press <enter>')
```

`setup.py` 也就是安装文件，源码如下：

```
from distutils.core import setup
import py2exe
setup(console=['c:\hello.py'])
```

下来进入到 `python` 的命令提示窗口，键入如下命令：`python c:\setup.py py2exe`

之后回车，命令提示窗口会有一系列的文件创建的和复制的过程。你会得到类似与下面的输出：

![](img/245aa86dcd19fc7fea33a23d0e6f991a.jpg)

现在编译已经完成，打开 `python` 的安装目录，在根目录下会有两个文件生成，如图：

![](img/6b6e56de7c8ceb26d819532493b65e41.jpg)

其中，`dist` 文件下保存着一个 `hello.exe`，这就是编译后的 `exe` 文件。双击执行，如图：

![](img/947740ddef88bc9c8fa78119ae16ac0f.jpg)

你现在可以 `PEID` 一下，也可以用 `OD` 反汇编看看!

最后将源文件和 `py2exe`,打包上传至附件！

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | `py2exe-0.6.9.win32-py2.7.rar` |
| :--- | :--- |
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | `setup.rar` |


# 利用 python+pefile 库做 PE 格式文件的快速开发

> 原文：[`www.pediy.com/kssd/pediy10/89838.html`](https://www.pediy.com/kssd/pediy10/89838.html)

发现很多的朋友经常用到 PE 格式相关的开发，如解析 PE 文件的格式，获取相关的内容。比如常常用到的静态的病毒启发式检测模型的建立、病毒样本分类、查壳脱壳等。

搜索了一下发现论坛里面没有我要讲的这个东西，于是我在这里向大家推荐 `pefile` 这个 `python` 库。

这个是基于 `MIT` licence 的一个开源项目，你可以在上面做更多的开发。

开发包的下载地址

[`code.google.com/p/pefile/`](http://code.google.com/p/pefile/)

我觉得有以下几点大家可以注意：

1. 这个需要使用 `python` 语言开发，优点是敏捷开发，方便快捷，而且源代码可读，易懂，当然肯定不会用于商业的，作为学习研究非常方便。
2. 由于基于 PE 的结构 `pefile` 已经做了非常充分的解析，所以对于我们做二次开发非常方便。各种关键的数据结构能够非常容易的获得。
3. 由于 `python` 的编写的快速、低门槛。另外 `pefile` 已经做了很多的功能，这个 `pefile` 模块非常适合需要快速达到目的和一些需要入门的朋友。
4. 免费的开源项目

话不多说，直接教大家使用，看完后，方可知道 `pefile` 的强大。

代码:

```
1. 当然是要安装 python 开发包。
2. 下载 pefile 到本地，解压，新建一个文件 petest.py

```

首先实验一

代码:

```
实验一
import os, string, shutil,re
import pefile ##记得 import pefile

PEfile\_Path = r"C:\\temp\\test.exe"

pe = pefile.PE(PEfile\_Path)
print PEfile\_Path
print pe

```

代码:


# [分享]《Python 灰兔子》一书中环境的搭建

题记：今天下午回到家，用一种叫罗汉果的东西泡了杯水喝（罗汉果又叫仙人果），然后打开看雪论坛看到 Python `pydbg` 这样的字样，这不正是这些天我在书店看到的那本书上的内容吗？打开链接一看，帖子的作者叫仙果，也就是罗汉果啦。这位罗汉果老兄翻译了日本人写的安装 `pydbg` 的文章，我想这本书上的示例运行起来确实有点麻烦，本着罗汉果老兄方便后学的精神，发了这样一篇帖子。

## 安装 Python

首先第一步就是安装 Python 了，这个很简单啦。值得说明的就是 Python 的版本，那本书上用的是 2.5 的版本，后来更新还有其他的版本。由于 2.5 以前没有 `ctypes`，所以不建议安装。我的电脑上装了 2 个版本：2.5 和 2.7，2.5 是在 2.7 之后安装的，双击 `*.py` 文件时运行的是 2.5。要运行 2.7，需要加上路径：`C:\Python27\Python.exe X:\xxxx\xxx.py`（没有配置环境变量）。

## 安装 `pydbg`

其次比较麻烦的就是 `pydbg`，一开始不知道去哪里下载 `pydbg`，运行示例程序就会报这样的错：

```
Traceback (most recent call last):
  File "E:\src\firefox_hook.py", line 1, in <module>
    from pydbg import *
ImportError: No module named pydbg
```

去网上找了一个 `pydbg`，是一个压缩包 `pydbg-package.rar`。

![](https://bbs.kanxue.com/upload/attach/201105/185356_2flek957no7b5ua.jpg)

`Pydbg` 要拷贝到 Python 的安装目录里面，一般是 `C:\Python27\Lib`。但这样还不行，因为 Python 里面的 `pydasm.pyd` 不匹配，`pydasm.pyd` 需要重新编译。

### 编译 `pydasm.pyd`

解压 `libdasm-1.4.tar.gz`，在 `cmd` 中切换到解压后的目录里，执行 `cd pydasm` 和 `python setup.py build_ext`。编译成功的话会生成一个 `build` 目录，里面有一个 `pydasm.pyd`。将这个 `pydasm.pyd` 替换到 `C:\Python27\Lib\python` 里面的 `pydasm.pyd` 就可以了。

有的同学这里可能编译不成功，电脑上需要有 Visual Studio。我的电脑上装有 vs2008，使用的 Python 版本是 2.7；如果使用的 Python 版本是 2.5，也会编译不成功，提示电脑上没有 vs2003。

### 从 `paimei` 项目获取新版 `pydbg`

其实，那个 `pydasm.pyd` 也可以不用重新编译（有误，补充里有说明）。可能是那个 `pydbg-package.rar` 里面的 `Pydbg` 版本太低，后来我从 google 上 `paimei` 项目里面下载的那个 `pydbg` 是没有问题的，拷贝到 `C:\Python27\Lib` 目录下就可以直接使用，不用重新编译 `pydasm.pyd`。

要重新下载需要安装 TortoiseSVN。装好之后，到桌面点鼠标右键会多一个菜单：

![](https://bbs.kanxue.com/upload/attach/201105/185356_lmiayk0r1wv0v6c.gif)

点那个 `Repo-browser`，会让你填 URL，谷歌搜索 `paimei`：

![](https://bbs.kanxue.com/upload/attach/201105/185356_o3pundcn1xj4ulh.png)

得到这样的地址：`648K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8S2K9h3#2W2K9g2)9J5k6h3N6G2L8$3N6D9k6h3y4G2k6r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6X3&6Q4x3V1k6@1M7Y4g2F1K9#2)9J5c8R3`。

![](https://bbs.kanxue.com/upload/attach/201105/185356_bj2swkdfi3hk1nm.gif)

点 OK 等一段时间会有这样的界面：

![](https://bbs.kanxue.com/upload/attach/201105/185356_h3zo1p7atl9ks07.gif)

选中的那几个是我们需要的：`pgraph`、`pida`、`pydbg`、`utils`，点右键 `CheckOut`，导出到本地，拷贝到 `C:\Python27\Lib` 里面就 ok 了。

## 安装 `idaPython`

最后，还有那个 `idaPython`，如果 Python 的版本不对的话它会识别不了。我的是 IDA 版本 5.5。要装 `python2.5` 才可以。

[`罗汉果的故事.doc`](https://bbs.kanxue.com/attach-download-57159.htm)

[`pydbg-package.rar`](https://bbs.kanxue.com/attach-download-57158.htm)

## 补充说明

补充一下：SVN 导出的那个 `pydbg` 要放到 `python2.7` 下面是不行的，估计放到 `python2.6` 里面可以，2.5 版本的可以从 `pyemu` 里面去拷贝（SVN 地址：`54bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8&6k6h3#2#2i4K6u0W2k6$3!0G2k6$3I4W2j5$3!0V1k6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4$3L8W2)9J5c8Y4c8J5N6h3&6C8i4K6u0r3i4K6t1&6i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1^5i4@1u0r3i4K6V1&6i4@1f1@1i4@1t1^5i4@1q4m8i4@1f1&6i4K6R3%4i4K6S2o6i4@1f1&6i4K6W2p5i4@1p5J5i4@1f1%4i4K6W2m8i4K6R3@1M7s2W2V1j5i4y4E0i4K6u0W2M7s2W2V1i4@1f1$3i4K6V1^5i4@1q4r3x3W2)9J5k6e0g2Q4c8e0N6Q4b7V1y4Q4z5e0k6Q4c8e0S2Q4b7f1k6Q4z5e0q4Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0k6Q4z5e0c8Q4b7V1g2Q4c8e0g2Q4z5o6S2Q4b7U0l9J5i4K6u0W2y4g2!0q4y4q4!0n7z5q4)9^5b7W2!0q4z5g2)9&6c8q4!0m8x3W2!0q4y4g2)9^5c8W2!0m8c8W2!0q4y4q4!0n7b7W2!0m8y4g2!0q4y4q4!0n7c8q4!0n7c8W2!0q4y4#2)9&6y4q4!0m8z5q4)9J5z5b7`）。

[`pydasm(3个版本).rar`](https://bbs.kanxue.com/attach-download-57172.htm)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

## 上传的附件

- [`pydbg-package.rar`](javascript:void(0);) （171.79kb，308次下载）
- [`罗汉果的故事.doc`](javascript:void(0);) （249.50kb，272次下载）
- [`222.gif`](javascript:void(0);) （48.51kb，1270次下载）
- [`333.png`](javascript:void(0);) （19.04kb，1266次下载）
- [`444.gif`](javascript:void(0);) （16.33kb，1265次下载）
- [`55.gif`](javascript:void(0);) （75.00kb，1262次下载）
- [`111png.jpg`](javascript:void(0);) （54.12kb，1236次下载）
- [`pydasm(3个版本).rar`](javascript:void(0);) （55.61kb，330次下载）

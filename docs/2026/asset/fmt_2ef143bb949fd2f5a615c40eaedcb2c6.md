
```shell
C:\Users\admin>nuitka --version
1.4
Commercial: None
Python: 3.8.8 (tags/v3.8.8:024d805, Feb 19 2021, 13:18:16) [MSC v.1928 64 bit (AMD64)]
Flavor: CPython Official
Executable: C:\Users\Python\Python38\python.exe
OS: Windows
Arch: x86_64
WindowsRelease: 10
Version C compiler: ~\AppData\Local\Nuitka\Nuitka\Cache\downloads\gcc\x86_64\11.3.0-14.0.3-10.0.0-msvcrt-r3\mingw64\bin\gcc.exe (gcc).
```

![图片描述](upload/attach/202306/909733_W98SJV96JKPCSQV.png)

# [Python逆向]逆向Pyinstaller打包的程序源码及保护

## 写在前面

> 我们都知道可以使用`Pyinstaller`库可将`.py`文件编译成`.exe`文件运行，这篇文章我们就从将脚本编译成`.exe`并将`.exe`的源码内容反编译出源文件，再顺便谈谈如何防止被逆向。

## 环境工具

- Python 3.6：`https://www.python.org/downloads/release/python-360/`
- `Pyinstaller`库：`pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pyinstaller==4.0`
- `pyinstxtractor.py`：下载地址：`https://sourceforge.net/projects/pyinstallerextractor/`
- `WinHex`编辑器：下载地址：`https://down.52pojie.cn/Tools/Editors/WinHex_v19.9.zip`
- `uncompyle`库：`pip install -i https://pypi.tuna.tsinghua.edu.cn/simple uncompyle6==3.9.0`
- `tinyaes`库：`pip install -i https://pypi.tuna.tsinghua.edu.cn/simple tinyaes==1.0.3`

## 编译程序

### 检查环境

```bash
python -V
```

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_4YWBEBE8B3WX5ST.png)

### 编写脚本

`测试脚本.py`

```python
import datetime

# 函数
def test():
    print("====反编译源码的测试脚本====")
    input_text = input("请输入你想要打印的内容：")

    # 打印格式化时间和用户输入的内容
    print(datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S ") + input_text)

if __name__ == '__main__':
    # 调用
    test()
```

### 打包程序

首先安装打包程序所使用的库`pyinstaller`，这里用了`清华源`并且指定使用了`4.0`版本库

```bash
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pyinstaller==4.0
```

找到脚本所在的文件夹，我的是`C:\Users\YRJ\Desktop\a`，`cd`到此路径下，并输入以下打包命令：

```bash
Pyinstaller -F 测试脚本.py
```

命令执行完毕后，会看到`completed successfully.`字段，表示`.exe`文件生成成功，在`dist`文件夹中。

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_N5ABH8PAAGPNJWR.png)

## 逆向过程

首先我们下载反编译工具`pyinstxtractor.py`与我们要反编译的`.exe`文件放入同一个工作目录下，如下图所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_X2KAP5KQVCZ9UWP.png)

```bash
python pyinstxtractor.py 测试脚本.exe
```

执行完毕，看到`Successfully`字样，会生成`测试脚本.exe_extracted`文件夹，如下图所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_TSVJWEKQZXT6H2V.png)

进入该文件夹，里面有许许多多后缀为`.dll`和`.pyd`的文件，还有一个名为`PYZ-00.pyz_extracted`的文件夹，这个文件夹里放的是程序引入的`依赖库`，如果你引入过自己其他的`.py`文件，就可以用类似的方法将依赖的`.py`文件反编译出来，如下图所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_P5ZJS35R4ZH64FK.png)

在目录中我们要找到`struct`和与你的`.exe`文件同名的文件。如下图所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_N58BJTSJQ6UH42K.png)

这两个文件是否带`.pyc`后缀和你使用的`pyinstxtractor.py`工具版本有关系。V2.0以前的版本，会生成两个不带`.pyc`后缀的文件，手动为它添加`.pyc`后缀即可。如下图所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_KK2YYUE5ERUAB9J.png)

当前这个`测试脚本.pyc`文件是没有`Magic Number`的，我们需要根据Python版本自行补全，由上图可知打包此程序的Python版本为3.6，我们接下来就需要查3.6版本的`Magic Number`。

- Python3.3 以下版本：只有`Magic Number`和四位时间戳
- Python3.3(包含) - Python3.7(不包含)版本：4个字节的magic num + 8个字节的时间戳，这个时间戳可以全是0。
- Python3.7(包含)版本：4个字节的magic num + 4个字节的空白数据 + 4个字节的时间戳 + 4个字节的文件长度，除了magic num，其它数据可以全是0。

### 查Magic Number

方法一：

查通用对照表：

```cpp
enum PycMagic {
    MAGIC_1_0 = 0x00999902,
    MAGIC_1_1 = 0x00999903,
    MAGIC_1_3 = 0x0A0D2E89,
    MAGIC_1_4 = 0x0A0D1704,
    MAGIC_1_5 = 0x0A0D4E99,
    MAGIC_1_6 = 0x0A0DC4FC,

    MAGIC_2_0 = 0x0A0DC687,
    MAGIC_2_1 = 0x0A0DEB2A,
    MAGIC_2_2 = 0x0A0DED2D,
    MAGIC_2_3 = 0x0A0DF23B,
    MAGIC_2_4 = 0x0A0DF26D,
    MAGIC_2_5 = 0x0A0DF2B3,
    MAGIC_2_6 = 0x0A0DF2D1,
    MAGIC_2_7 = 0x0A0DF303,

    MAGIC_3_0 = 0x0A0D0C3A,
    MAGIC_3_1 = 0x0A0D0C4E,
    MAGIC_3_2 = 0x0A0D0C6C,
    MAGIC_3_3 = 0x0A0D0C9E,
    MAGIC_3_4 = 0x0A0D0CEE,
    MAGIC_3_5 = 0x0A0D0D16,
    MAGIC_3_5_3 = 0x0A0D0D17,
    MAGIC_3_6 = 0x0A0D0D33,
    MAGIC_3_7 = 0x0A0D0D42,
    MAGIC_3_8 = 0x0A0D0D55,
    MAGIC_3_9 = 0x0A0D0D61,
};
```

例如：Python 3.6的`Magic Number`，执行后可以得到`0x0A0D0D33`，则对应二进制码是`33 0D 0D 0A`。

方法二（推荐）：

看`struct.pyc`文件：

将`struct.pyc`文件使用`WinHex`编辑器打开，它的前4位字节就是`magic num`与方法一的二进制码相同。如下图所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_QSGE9VP6YBE9W6Y.png)

再打开`测试脚本.pyc`与`struct.pyc`对比头内容确定要添加的内容，如下图所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_ATGJ9GJEUGXUUBA.png)

### 改pyc文件

选中`struct.pyc`中框选的头内容，右键编辑→复制选块→十六进制数值。`330D0D0A7079693001010000`。

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_JFKCBG5M4T94S3H.png)

打开`测试脚本.pyc`，直接在前面添加上面复制的十六进制数值`330D0D0A7079693001010000`，光标点击第一个字母`E`右键编辑→剪贴板数据→粘贴→ASCII Hex→确定，然后保存即修改完毕！如下图所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_YMPPVDSANDUS5WY.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_AXX35TQDYDNFVH2.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_RE992TMUU9HFHUP.png)

### 转换源码

安装`.pyc`转`.py`所使用的库`uncompyle6`，这里用了`清华源`并且指定使用了`3.9.0`版本库

```bash
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple uncompyle6==3.9.0
```

找到`测试脚本.pyc`所在的文件夹，`cd`到此路径下，并输入以下打包命令：

```bash
uncompyle6 -o 测试脚本.py 测试脚本.pyc
```

命令执行完毕后，会看到`successfully`字段，表示`.py`源码文件已成功生成在同路径下。

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_VB2KNPETFQ3TFW2.png)

### 比对源码

通过对程序本来的源码和逆向得来的源码对比可知：简单的源码上基本一致，复杂些的源码上可能会有一点点瑕疵。

单就此次逆向的程序而言，比程序原本的源码就少了源码的注释，多了系统反编译时相关环境及版本的注释。

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_35C8HRSADPS7QQZ.png)

通过命令`python 测试脚本.py`执行逆向得来的源码，测试通过，发现功能正常。

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_52TSJXWFSGYC2UJ.png)

## 防止逆向

### pyinstaller打包

首先安装加密打包程序所使用的库`tinyaes`，这里用了`清华源`并且指定使用了`1.0.3`版本库

```bash
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple tinyaes==1.0.3
```

找到脚本所在的文件夹，我的是`C:\Users\YRJ\Desktop\c`，`cd`到此路径下，并输入以下打包命令：

```bash
pyinstaller -F 测试脚本.py --key 123456
```

命令执行完毕后，会看到`completed successfully.`字段，表示`.exe`文件生成成功，在`dist`文件夹中。

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_MV6KSGDUYFSZA6V.png)

接着就按照前面写的`逆向过程`执行命令`python pyinstxtractor.py 测试脚本.exe`，结果只有入口脚本反编译成功，被依赖的脚本均被加密，无法直接被反编译，如下图所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_YRQWC3XJU6U2Y2K.png)

`PYZ-00.pyz_extracted`非常重要，一般一个稍微大一点的项目都会分成多个py文件，甚至会依赖其他模块，这些被依赖的文件解析后都会放入`PYZ-00.pyz_extracted`中，可以说这里放的是核心代码。

`PYZ-00.pyz_extracted`文件夹内可以看到，抽取的中间结果变成了`.pyc.encrypted`格式，无法直接被反编译，未加密和加密的相关对比，如下图所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_9ZGVN4CTF3G29YR.png)

常规手段就无法直接反编译了。这个时候若还想反编译就需要底层的逆向分析研究或将`pyinstaller`的源码完整研究一遍，了解其加密处理的机制，看看有没有逆向的可能。

**注意：** 请勿使用此种方法来保护只有一个`.py`文件的python源码，因为主函数的入口脚本编译是成功的，根据`逆向过程`章节操作后，入口脚本一样的会被看到源码。

### nuitka打包

首先安装打包程序所使用的库`nuitka`，这里用了`清华源`并且指定使用了`1.3.7`版本库

```bash
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple nuitka==1.3.7
```

找到脚本所在的文件夹，我的是`C:\Users\YRJ\Desktop\a`，`cd`到此路径下，选择输入以下你需要的打包命令：

```bash
nuitka --standalone --mingw64 测试脚本.py
nuitka --standalone --onefile 测试脚本.py #打包成一个exe单文件
nuitka --standalone --mingw64 --follow-imports 测试脚本.py
nuitka --standalone --remove-output --mingw64 测试脚本.py
nuitka --standalone --mingw64 --nofollow-imports --show-memory --show-progress --plugin-enable=qt-plugins --follow-import-to=lib --recurse-all --output-dir=o 测试脚本.py
```

## 最后总结

之前我没了解过这方面内容，但偶然需要用到时查阅了很多的博客文章及资料，中途也出现很多当时无法解决的但幸运的是慢慢都解决了，就写下了这篇详细的总结文章记录下过程。

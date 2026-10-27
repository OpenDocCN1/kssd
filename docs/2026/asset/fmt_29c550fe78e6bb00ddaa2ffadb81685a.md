
打包过程中会提示缺少执行程序`gcc`，直接按照要求`Yes`安装，此文件比较大命令行可能下载不动或缓慢，我们可直接复制`下载位置`和`下载地址`，如下图所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_F7KNVCVFVRGPBJ5.png)

|  |  |
| --- | --- |
| 1  2  3  4  5 | `下载位置：`  `C:\Users\YRJ\AppData\Local\Nuitka\Nuitka\Cache\downloads\gcc\x86_64\11.3.0-14.0.3-10.0.0-msvcrt-r3`    `下载地址：`  `https://github.com/brechtsanders/winlibs_mingw/releases/download/11.3.0-14.0.3-10.0.0-msvcrt-r3/winlibs-x86_64-posix-seh-gcc-11.3.0-llvm-14.0.3-mingw-w64msvcrt-10.0.0-r3.zip` |

用浏览器访问`下载地址`移动到`下载位置`的目录下并做解压即可，如下图所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_BQ6X3B5E2ZJVWN3.png)

接下来，我们继续执行打包命令：`nuitka --standalone --mingw64 测试脚本.py`，打包过程中还会提示缺少的执行程序`ccache`、`depends`，此文件比较小，根据提示直接`Yes`安装即可，下载完成后会自动解压到路径：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9 | `下载位置：`  `C:\Users\YRJ\AppData\Local\Nuitka\Nuitka\Cache\downloads\ccache\v4.6\ccache.exe`  `下载地址：`  `https://github.com/ccache/ccache/releases/download/v4.6/ccache-4.6-windows-32.zip`    `下载位置：`  `C:\Users\YRJ\AppData\Local\Nuitka\Nuitka\Cache\downloads\depends\x86_64`  `下载地址：`  `https://dependencywalker.com/depends22_x64.zip` |

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_TFB2UEBGDXXDRDA.png)

至此，用`nuitka`来将python代码打包为`.exe`的过程就结束了。

### 验证执行

执行打包好的`.exe`程序，命令行出口出现乱码问题，如下图：

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_3BUE7SFFF95K8WE.png)

解决方案是：在主函数首行添加`os.system("chcp 65001 && cls")`

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_VN4M87N7TZSFRXX.png)

再次打包并执行打包好的`.exe`程序，问题解决，如下图所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_T5GGHS4RYHJV9VE.png)

但这又导致出现了一个bug，Python源码中的`input("请输入你想要打印的内容：")`是需要在命令行输入内容的，我输入`英文和数字`的时候打印输出时正常的，但是我输入中文的时候，就会引起异常及闪退的情况，我暂时还未有相关解决方案....

输入中文会引起异常及闪退的解决方案：

换当前使用的`Python3.6.0`版本到`Python3.8.0及以上`即可解决问题。

我使用的环境及运行测试图如下：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10 | `C:\Users\admin>nuitka --version`  `1.4`  `Commercial: None`  `Python: 3.8.8 (tags/v3.8.8:024d805, Feb 19 2021, 13:18:16) [MSC v.1928 64 bit (AMD64)]`  `Flavor: CPython Official`  `Executable: C:\Users\Python\Python38\python.exe`  `OS: Windows`  `Arch: x86_64`  `WindowsRelease: 10`  `Version C compiler: ~\AppData\Local\Nuitka\Nuitka\Cache\downloads\gcc\x86_64\11.3.0-14.0.3-10.0.0-msvcrt-r3\mingw64\bin\gcc.exe (gcc).` |

![图片描述](https://bbs.kanxue.com/upload/attach/202306/909733_W98SJV96JKPCSQV.png)

## 最后总结

之前我没了解过这方面内容，但偶然需要用到时查阅了很多的博客文章及资料，中途也出现很多当时无法解决的但幸运的是慢慢都解决了，就写下了这篇详细的总结文章记录下过程。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[#软件保护](https://bbs.kanxue.com/forum-4-1-3.htm)

[#其他内容](https://bbs.kanxue.com/forum-4-1-10.htm)

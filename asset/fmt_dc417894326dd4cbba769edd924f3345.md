
# PaiMei 安装与使用

> 原文：[`www.pediy.com/kssd/pediy10/103127.html`](https://www.pediy.com/kssd/pediy10/103127.html)

【文章标题】：PaiMei 安装与使用  
【文章作者】：youstar  
【作者邮箱】：youstar@foxmail.com  
【个人主页】：www.youstar.org.ru  

## 写作原因

一则是：自己最近学习《0day:软件漏洞分析技术》，但是学到 PaiMei 这一章的时候，按照书上面安装 PaiMei 时试了很多次都没成功，最后参照很多帖子终于成功了。  
二则是看到论坛里面有很多人问这个问题，都没有什么人回答，所以想帮助下大家，大家一起学习。  
为此写下此文，让以后更多的人能够少走弯路。

## 所需软件

1）`python-2.4.3.msi`  
2）`ctypes-0.9.9.6.win32-py2.4.exe`  
3）`wxPython2.6-win32-ansi-2.6.3.2-py24.exe`  
4）`MySQL-python.exe-1.2.1_p2.win32-py2.4.exe`  
5）`uDrawGraph-3.1.1-0-win32-en.exe`  
6）`PaiMei-1.1.win32.exe`  
7）`mysql-4.1.13-win32`  
8）`idapython-0.9.57_ida5.1_py2.4_win32.zip`  
9）IDA(这个看雪论坛上面有我就不提供了)  
这些软件自己都会打包给大家下载。

## 安装步骤

主要步骤如下：（基本上安装路径默认就行，我都是放到 `C:\\软件名`）

1）安装 `python-2.4.3.msi`  
2）安装 `ctypes-0.9.9.6.win32-py2.4.exe`  
3）安装 `wxPython2.6-win32-ansi-2.6.3.2-py24.exe`  
4）安装 `MySQL-python.exe-1.2.1_p2.win32-py2.4.exe`  
5）安装 `uDrawGraph-3.1.1-0-win32-en.exe`（安装路径可以换一个简单的路径）  
6）安装 `PaiMei-1.1.win32.exe`  
7）安装 `mysql-4.1.13-win32` 安装过程中要设置密码（暂时设置为 `test`） 注意：中间选择 `skip sign-up`  
8）利用命令行模式运行 Paimei 文件夹下的 `\_setup\_mysql.py` `127.0.0.1` `root` `test`  
9）将 `idapython-0.9.57_ida5.1_py2.4_win32.zip` 解压后把 `python` 文件夹里面的文件放到 IDA 安装目录下，并把 `plugins` 的 `python.plw` 文件放到 IDA 的 `plugins` 文件夹内。  
至此安装就完成了。

## 使用教材

为了以教材为准，以教材的例子为例，其实其它的我也还不怎么会，慢慢学以后给他们分享![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")

### 1）用 IDA 生成 calc.exe.pida

由于我们把 `idapython` 的插件放到了 IDA 的 `plugins` 文件夹，所以就会出现 `Edit--Plugins-IDAPython`，就时候就需要选择 PaiMei 程序目录下的 `pida_dump.py`，然后存为 `calc.exe.pida`。

### 2）准备环境

首先启动 PaiMei,然后链接 MySQL,输入先前设置的密码。  
然后启动 `uDrawGraph` 程序，需要用 `uDrawGraph` 需要命令行启动，例如：`C:\\uDraw\\bin\\uDrawGraph.exe -server 222` （222 为端口，可以自己设置）

如下图所示：  
![](img/eba64a150ad84cb75b23703745cbe59d.jpg)

然后链接 uDraw，使用我们刚刚开启的端口，就成功了。就可以开启分析程序了。  
最后完全按照书上面讲的就可以试验了。

## 参考资料

其实论坛上面看了很多，最感谢的是 wuzhiyong 写的文章，[链接](http://bbs.pediy.com/showthread.php?t=81225&highlight=paimei)

## 程序下载

纳米盘下载：[下载链接](http://d.namipan.com/d/paimei.rar/701c3c00a44e0784f1963173c803d645e97f4d9e5318e703) ![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")

PS:貌似 PaiMei 应该大牛但是学习汉语不过关哈，拼音拼错了！![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")

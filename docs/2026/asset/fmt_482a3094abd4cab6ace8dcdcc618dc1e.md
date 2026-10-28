
就是注册环境变量`%AppId%`为`C:\intel\msdtcv.exe`

![1](https://bbs.kanxue.com/upload/attach/202001/780480_2FCSKE8FQJAH9H3.jpg)

`Sub_401F00`函数内容具体如图:

![pic9](https://bbs.kanxue.com/upload/attach/202001/780480_ZDT2E337YZQHJ56.jpg)

![pic10](https://bbs.kanxue.com/upload/attach/202001/780480_29GDDA8C4TWUW94.jpg)

调用`CreateThread`创造线程,线程回调函数 `StratAdress`

![pic11](https://bbs.kanxue.com/upload/attach/202001/780480_8TU3FRRHSNX6DBC.jpg)

`Paramter为:`

![a](https://bbs.kanxue.com/upload/attach/202001/780480_NXK779PKCUVYD5R.jpg)

线程回调函数`StartAddress`分析:主要是创造与`cmd.exe`的进程通信,用`WriteFile`函数向匿名管道中写入一段cmd命令(`copy 源文件路径 C:\intel\msdtcv.exe`),然后让`cmd.exe`执行将木马自身文件(`Update Required Case Enq No 192_2018.docx.com`)复制到`C:\intel\msdtcv.exe`

**解密出环境变量`%ComSpec%`**

![2](https://bbs.kanxue.com/upload/attach/202001/780480_76PW952WF85YXA2.jpg)

**获取环境变量`%ComSpec%`的路径**

![3](https://bbs.kanxue.com/upload/attach/202001/780480_D7BQP6KU3JMACCN.jpg)

**得到结果:`cmd`的路径**

![4](https://bbs.kanxue.com/upload/attach/202001/780480_WRKJJK5AJRQ75PZ.jpg)

**建立与`cmd.exe`的匿名管道进行通讯,并创立子进程`cmd.exe`**

![5](https://bbs.kanxue.com/upload/attach/202001/780480_RBDJ6FZHDA2X35S.jpg)

![6](https://bbs.kanxue.com/upload/attach/202001/780480_9V73RBUPEXJK9NZ.jpg)

**调用`WriteFile`函数向`cmd.exe`写入`copy`指令实现复制文件**

![7](https://bbs.kanxue.com/upload/attach/202001/780480_GS5N6Z5VA58KCVD.jpg)

如下图所示:

![pic15](https://bbs.kanxue.com/upload/attach/202001/780480_NS6TZE6U8NMEV76.jpg)

![pic16](https://bbs.kanxue.com/upload/attach/202001/780480_WD2DYEZFHW74D76.jpg)

打开注册表启动项(`HKEY_CURRENT_USER\Software\Microsoft\Windows\Currentversion\Run`)

判断启动项是否存在如果存在不重新写启动项,不打开木马文件转向下面执行

如果不存在则调用`CreateThread`创建线程`sub_404670`来写入开启启动项然后退出进程

将后续工作交给`msdtcv.exe`这个文件(不就是他自己吗)

![pic17](https://bbs.kanxue.com/upload/attach/202001/780480_S6K7D8R9GC6S6V2.jpg)

`线程sub_404670`如下图:

打开注册表(`HKEY_CURRENT_USER\Software\Microsoft\Windows\Currentversion\Run`)

判断是否存在`msdtcv`键,如果不存在则创建`msdtcv`键,写入`cmd /c start %AppId% && exit`键值关闭注册表

若存在则直接退出线程。

![pic18](https://bbs.kanxue.com/upload/attach/202001/780480_UUU9UX9QDCF7NZV.jpg)

![pic19](https://bbs.kanxue.com/upload/attach/202001/780480_33HQU2TKJQG62JX.jpg)

## 0x02. Downloader-下载功能模块

当初始化模块结束后,完成文件重命名复制以及注册表自启动后

`msdtcv.exe`随即启动,按照`初始化模块`的操作进行了一遍检查后

开始与`C&C服务器`进行交流

从下图我们可知,写了一个死循环一直获取从`C&C服务器`回显的指令。

![pic21](https://bbs.kanxue.com/upload/attach/202001/780480_YSKEQGGUAEH2ZW9.jpg)

**获取主机名称**

![pic22](https://bbs.kanxue.com/upload/attach/202001/780480_ZMYTTX8E3Z6ECGE.jpg)

**获取系统版本信息**

![pic23](https://bbs.kanxue.com/upload/attach/202001/780480_XQ3PAHJWCP9UU2N.jpg)

![pic24](https://bbs.kanxue.com/upload/attach/202001/780480_JKZ4KK587XK2V88.jpg)

**木马与`C&C`的交流**

![pic25](https://bbs.kanxue.com/upload/attach/202001/780480_GY6GDY7QDP2XAT3.jpg)

`Sub_402ba0`函数

通过`C&C服务器域名`:`framworksupport.net`来获取`ip`:`162.222.215.90`,`端口`:`80` 建立socket连接

![pic26](https://bbs.kanxue.com/upload/attach/202001/780480_7NC4V73282S7VUB.jpg)

**向`C&C服务器`发送上线信息以及请求下一步指令的报文**

![a](https://bbs.kanxue.com/upload/attach/202001/780480_G75WCQUJC42UCM3.jpg)

**报文如下**

![b](https://bbs.kanxue.com/upload/attach/202001/780480_PH35BN5EDBBVE7G.jpg)

**下载攻击者的指定文件**

![pic27](https://bbs.kanxue.com/upload/attach/202001/780480_ZWW2YSUS29EBBKP.jpg)

**执行下载文件（文件地址如图）**

![pic28](https://bbs.kanxue.com/upload/attach/202001/780480_3XHR7VXNS8W4CPY.jpg)

![c](https://bbs.kanxue.com/upload/attach/202001/780480_WWN7XB9GK9SG8N9.jpg)

**再次进行信息发送以及接收回显**

**发送信息以及下一步请求**

![9](https://bbs.kanxue.com/upload/attach/202001/780480_BWZRSUYTUVDKPCX.jpg)

**接收回显**

![10](https://bbs.kanxue.com/upload/attach/202001/780480_PC3JK8VKC5VFAHA.jpg)

由于`C&C`的那个木马下载的页面不能正常访问,木马也不能被下载,所以笔者没办法对下一步木马的操作进行分析

## 0x03 关联

首先看看`VT`的查杀情况

截止发帖的今天已经有46款杀毒软件查杀此木马

![pic30](https://bbs.kanxue.com/upload/attach/202001/780480_WRG2MKXWTVQ689Q.jpg)

微步云沙箱的结果

恶意行为:

![pic31](https://bbs.kanxue.com/upload/attach/202001/780480_8JHBNWFFHJ7WPUQ.jpg)

低危行为:

![pic32](https://bbs.kanxue.com/upload/attach/202001/780480_CJ8ADDVKXMHQ5HF.jpg)

这个**收集系统相关硬件的指纹信息**这部分应该在`sub_4037B0`函数中

笔者没有进行详细的分析,同样也有很多混淆,那就请有兴趣的读者自己练手了哟。

程序流程图:

![pic33](https://bbs.kanxue.com/upload/attach/202001/780480_37VVKCWG5D8MMBZ.jpg)

木马`C&C服务器域名`:

![pic34](https://bbs.kanxue.com/upload/attach/202001/780480_VQKECCQNHXRZDXS.jpg)

反查一下域名可见 这个域名是在`2019-1-04`注册的 而且与之通讯的样本有两个

![pic35](https://bbs.kanxue.com/upload/attach/202001/780480_NWDHTTSRE95CR5T.jpg)

其中一个是笔者分析的这个在`2019/01/17`被捕获

![pic36](https://bbs.kanxue.com/upload/attach/202001/780480_7Q76W6MTT7URMBB.jpg)

另一个`2019/01/09`被捕获的样本,笔者调出了微步云沙箱的记录

经过比对后,发现这两个样本的`pdb_path`相同

![pic37](https://bbs.kanxue.com/upload/attach/202001/780480_C9B7X66NW7KUP86.jpg)

程序运行流程图也很类似

这就有很大的可能这两个样本出自一人之手

![pic38](https://bbs.kanxue.com/upload/attach/202001/780480_RF89WCWVVJXYN6T.jpg)

## 0x04总结

那这个木马也就这么告一段落了

笔者其实也有一些地方看得不是很明白,而且有的地方描述的也不是很清楚,如果各位读者`dalao`能看出笔者文中的不对以及描述不清楚的地方请在评论区提出来,十分感谢

笔者把样本放到了附件,请各位有心气的读者自行逆向分析

其中这个样本在字符串的隐藏上做的还是很不错,不同的地方有不同的加密，有的不单单有移位加密同时也有一些作者自己写的混淆。同时这个样本调用了很多`冷门API`在一定程度上阻碍了病毒分析的步伐。但是这个样本的后缀名欺骗真的没法去讲,但是如果是插入`unciode`的`RLO`控制字符实现后缀名倒转的话应该能起到一定的欺骗作用,但是图标也是一个大的问题,所以这个弊端也大大降低了入侵成功的可能性

并且该**下载者木马**可以通过重命名拷贝到 `C:\intel`文件夹下

通过**修改注册表**达到**自启动**的目的,攻击者可以修改相应的回显`shellcode`来改变木马插件的下载内容,在这里看来这个样本还是值得一分析的。

## 0x05 木马样本

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2019-2-7 21:56

被`CrazymanArmy`编辑

，原因： 将文中的一些错误进行订正

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)

上传的附件：

- [蔓灵花APT样本(解压密码APT).rar](javascript:void(0);)

  （12.72kb，103次下载）

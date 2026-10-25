
# [原创]内核学习-双机调试环境搭建

最近在学习windows内核的知识，记录下自己搭建双机调试的坑点

使用Win10 +Windows XP Professional

## xp虚拟机配置部分

打开xp虚拟机，
进入C盘根目录，工具，文件夹选项下，取消勾选隐私受保护的系统文件，勾选显示所有文件或文件夹。
![在这里插入图片描述](upload/attach/202107/921642_ZCXATSKK9A5KPHK.jpg)
显示出隐藏属性的文件，打开`boot.ini`文件
![在这里插入图片描述](upload/attach/202107/921642_FHEHKSAVH6YN9XF.jpg)
添加下面这段内容
```
multi(0)disk(0)rdisk(0)partition(1)\WINDOWS="debug" /noexecute=optin /fastdetect /debug /debugport=com1
```
![在这里插入图片描述](upload/attach/202107/921642_NCWT58SVWW7Z89N.jpg)
关闭虚拟机，编辑虚拟机设置，先移除打印机，
![在这里插入图片描述](upload/attach/202107/921642_6A5C7ZU9SV86Q3T.jpg)
添加一个串行端口
```
\\.\pipe\com_1
```
不移除打印机可能会出现串行端口2的情况，在后面配置完使用`windbg`时会会出现此种情况
![在这里插入图片描述](upload/attach/202107/921642_G9YBSUKBR7UMHYT.jpg)
（可以不删除打印机，使用`\\.\pipe\com_2`，相应的`boot.ini`中添加的内容也要做出改变
`multi(0)disk(0)rdisk(0)partition(1)\WINDOWS="debug" /noexecute=optin /fastdetect /debug /debugport=com2`，在后续的`windbg`配置中对应的也要改为`com_2`）

## windbg配置部分

1. Windows XP使用`windbg` 6.x 32位 版本的进行调试
（有些双机调试是用win10+win7搭建的 win7使用`windbg` 10.x 64位 版本的。）
将安装好的`windbg`右键发送到桌面快捷方式。(`windbg`目录：`C:\Program Files\Debugging Tools for Windows (x64)`)
![在这里插入图片描述](upload/attach/202107/921642_FSF2T6F323ZH73Y.jpg)
对桌面快捷方式右键，属性-->目标后面添加内容如下
```
-b -k com:pipe,port=\\.\pipe\com_1,baud=115200,pipe
```
![在这里插入图片描述](upload/attach/202107/921642_8RAH498VBDV7VG2.jpg)
双机打开`windbg`，
![在这里插入图片描述](upload/attach/202107/921642_62WENVKQC3R6H7N.jpg)
在Symbol File Path中添加如下内容，`srv*c:\symbols*http://msdl.microsoft.com/download/symbols`
![在这里插入图片描述](upload/attach/202107/921642_M2AENXB8WP5AFMY.jpg)
勾选reload
这种使用url的方法最为推荐，因为符号文件会从微软服务器上自动拖拽，并永久缓存。
也可以用添加环境变量的方式：![在这里插入图片描述](upload/attach/202107/921642_E3U7D563DUNX63V.jpg)
```
变量名： _NT_SYMBOL_PATH
变量值：srv*d:\symbols*http://msdl.microsoft.com/download/symbols
```
我最先使用的配置符号文件的方法就是离线符号文件下载，发现根本无法使用。。。。
其他的配置符号文件的方法：命令行输入
在 `windbg` 的命令行中输入：
```
> .sympath SRV*c:\符号文件路径*http://msdl.microsoft.com/download/symbols
> .reload
```

## 双机调试

开启此xp虚拟机，选中`debug`但是不回车！，接着打开`windbg`，然后回车
![在这里插入图片描述](upload/attach/202107/921642_52788U6BZHXWDTM.jpg)
![在这里插入图片描述](upload/attach/202107/921642_N7CQ3RXTEQPPPX5.jpg)
`windbg`显示如图
![在这里插入图片描述](upload/attach/202107/921642_N7CQ3RXTEQPPPX5.jpg)
输入`!process 0 0`也可成功查看进程
![在这里插入图片描述](upload/attach/202107/921642_FGVT24HDXK34BD4.jpg)
配置成功！！

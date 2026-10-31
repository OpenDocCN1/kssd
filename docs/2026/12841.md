
# [原创]写了个简单的文件夹加密程序

以前就想写个文件夹加密程序，最近在论坛上看到了一篇文件夹加密的帖子，看了它的源码，我的图标还是从那个软件里面弄出来的。

本程序仅供参考，仅以此说下加密文件夹的思路。请不要用来真的用来加密文件夹，因为加密的文件夹在xp下很容易删除，程序对文件夹是没有任何保护措施的

![](https://bbs.kanxue.com/upload/attach/201210/452670_bya0y23m059gsol.png)

![](https://bbs.kanxue.com/upload/attach/201210/452670_j3cimlwq6z3c6s8.png)

![](https://bbs.kanxue.com/upload/attach/201210/452670_0fme1i25fmms3og.png)

![](https://bbs.kanxue.com/upload/attach/201210/452670_wfqixe8wuq1a62m.png)

软件的实现比较简单，我先说下我是怎么实现的

## 驱动器加密页

程序初始化的时候会读取`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer`下的`NoDrives`和`NoViewOnDrive`值，以确定驱动器是否被隐藏或者禁止。

据我观察如果要隐藏C盘就设置`NoDrives`值为4，D盘为8，E盘为16，C盘和D盘同时隐藏的话设`NoDrives`为12，很明显这数字是有规律的。

假设A盘为0，B盘为1，C盘为2（`1<<2=4`），D盘为3（`1<<3=8`），E盘为4（`1<<4=16`）。如果有其它盘符的话依次递增。

将1左移盘符的索引刚好等于你要隐藏的盘符数字，而如果你要同时隐藏多个盘符的话，可以将多个盘符的对应的值逻辑或起来。

## 文件夹伪装页

用`rename`加个后缀名就行了

## 文件夹加密页

文件夹初始化是遍历所有驱动器，读取特殊文件夹里面的配置文件，配置文件里面包含当前驱动器上的加密文件夹列表，其中包含加密文件的加密时间，原先目录，和加密密码。

加密其实就是把文件夹移动到一个特殊的文件夹而已，我的路径是这样，在驱动器的根目录创建这样一个文件夹，然后再把文件夹移动到这个文件夹就行了。之所以选择这个路径，因为这个文件夹，在win7上，既不能删除，也不能访问。即使用dos命令`rd /s /q //?/+文件夹路径`也删除不了。

因为我的路径里面包含特殊字符，在dos命令上面根本显示不了。但在xp下却可以删除，目前没有想到好的办法。

```
LPTSTR			g_aEncryptPart[] = {
	L"\x7f\x7f\x7f\x7f\x7f\x7f\x7f\x7f\x7f\x7f\x7f\x7f\x7f.\\",
	L"nul.\\",
	L"con.{645FF040-5081-101B-9F08-00AA002F954E}\\",
	L"local"
};
INT				g_cEncryptPart = 4;
```

源码也发出来，vs2010 SDK可编译

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [`QQ截图20121018110848.png`](javascript:void(0);)（18.82kb，15次下载）
- [`QQ截图20121018111329.png`](javascript:void(0);)（21.23kb，8次下载）
- [`QQ截图20121018111508.png`](javascript:void(0);)（27.67kb，6次下载）
- [`QQ截图20121018111544.png`](javascript:void(0);)（40.98kb，9次下载）
- [`FolderEncrypt.zip`](javascript:void(0);)（803.82kb，91次下载）


# [原创]蠕虫勒索软件WannaCrypt0r的行为分析

来科锐学习一年出头，`x86`平台刚好学完，这个周末遇到这个病毒爆发，花了点时间分析了下这个样本，发出来与看雪坛友分享。

## 一）样本大致行为预览

该样本执行后会随机向互联网计算机发送`EternalBlue`漏洞溢出程序，并释放勒索加密程序，加密计算机上`.doc`、`.ppt`、`.jpg`、`.sqlite3`、`.mdb`、`.bat`、`.der`等多种类型的文件，修改文件后缀为`.WNCRY`类型，弹出窗口勒索用户，要求一定时间内交出赎金，才能恢复文件。

## 二）样本行为分析

### 2.1 原始样本分析

定位到`WinMain`后，可以发现在`WinMain`入口处发现，原始样本会尝试连接一个非常没有规律的域名：`22cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2#2M7h3g2J5k6Y4y4G2k6s2l9&6K9h3k6B7j5i4m8G2M7$3c8X3K9X3S2Y4L8%4y4#2M7X3W2B7k6X3q4W2N6%4u0%4k6i4u0Y4N6$3g2S2i4K6u0W2j5$3!0E0`。

![](https://bbs.kanxue.com/upload/attach/201705/751215_8z42n4vh9pizjfq.png)

样本会通过是否能成功连接该域名，来决定是否展开后续行为。

![](https://bbs.kanxue.com/upload/attach/201705/751215_7gl8tej1zpfw8p2.png)

如上图所示，若能够成功链接，则程序不展开相关行为，直接退出。否则，展开相关行为。

其中的`sub_408090`函数内部，在一个条件判断中分了两个功能模块：

![](https://bbs.kanxue.com/upload/attach/201705/751215_244lr3c7l0aw3ul.png)

其中，条件成立的代码块是启动服务功能等行为展开，程序第一轮并不会进入；当程序第一次启动时，执行的是条件不成立时的模块（上图中的`sub_407f20`），其功能主要为创建服务、释放并执行勒索的主体程序`tasksche.exe /i`。

创建服务过程：

![](https://bbs.kanxue.com/upload/attach/201705/751215_nfqteb8wj77sc7u.png)

该样本创建一个服务名为`mssecsvc2.0`，服务说明为`Microsoft Security Center (2.0) Service`的服务。

并在后续的行为中启动自身。

该服务的功能为：随机扫描互联网IP，利用`SMB`的内核级漏洞（`CVE-2017-0143`~`CVE-2017-0148`），传播自身。这个漏洞的利用我自己还在分析，分析出来后会放在后续文章中。

原始样本，会从资源中加载、释放一个名为`tasksche.exe`的文件，并且以`tasksche.exe /i`的方式启动。

![](https://bbs.kanxue.com/upload/attach/201705/751215_fjjbbtbd1poipx4.png)

![](https://bbs.kanxue.com/upload/attach/201705/751215_qv8wr5j3a0mmeg5.png)

接着，我分析了`tasksche.exe`，它的行为如下：

### 2.2 tasksche.exe的行为分析

`tasksche.exe`启动后，首先，会将自身拷贝到`C:\intel\lgxbrzjmuzoytl531`。

![](https://bbs.kanxue.com/upload/attach/201705/751215_lv0yu670z14hczh.png)

接着，`tasksche.exe`会创建服务，服务名及说明均为`lgxbrzjmuzoyt1531`，然后以服务的方式再启动自身。

![](https://bbs.kanxue.com/upload/attach/201705/751215_7zfcbt2du9msc5f.png)

![](https://bbs.kanxue.com/upload/attach/201705/751215_jfa2ygukl98pyzu.png)

以服务的方式启动后，`tasksche.exe`会以创建进程的方式执行命令：

![](https://bbs.kanxue.com/upload/attach/201705/751215_fdz077u3544qay0.png)

`attrib.exe +h`

`icacls.exe . /grant Everyone:F /T /C /Q`

这两个命令用于修改文件属性相关的权限。

接着，会读取前期释放的一个名为`t.wnry`文件，并对其中的内容进行解密，解密后的内容为一个`Dll`文件，但是该`dll`文件并不会被写入磁盘中，而是在内存中直接执行。

![](https://bbs.kanxue.com/upload/attach/201705/751215_08uv4i8ch18d5k6.png)

![](https://bbs.kanxue.com/upload/attach/201705/751215_pnexfhy61a7y9og.png)

我将该`dll`从内存中`dump`出（取名为`crypt.dll`），发现包含了勒索软件的加密功能。对其进行了分析。

### 2.3 crypt.dll样本分析

`crypt.dll`只有一个自定义的导出函数`TaskStart`，该导出函数是其行为展开的入口。

![](https://bbs.kanxue.com/upload/attach/201705/751215_zbr5s6vtd8lyj4w.png)

进入`TaskStart`后，该样本会先加载`Kernel32`，动态获取后续需要的`API`。

![](https://bbs.kanxue.com/upload/attach/201705/751215_1tp3mcaq3f8ighm.png)

创建互斥体`MsWinZonesCacheCounterMutex`，用于判断自身重入问题。

![](https://bbs.kanxue.com/upload/attach/201705/751215_zofekjrhu6oxqan.png)

接着，该样本连续创建了多个线程：

![](https://bbs.kanxue.com/upload/attach/201705/751215_z7cxzscvh9c8dzu.png)

他们的功能分别为：

- 将`tasksc.exe`加入到`HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`项，用于开机自启动
- 遍历驱动器，并检查是否有U盘等移动存储插入
- 循环调用`taskdl.exe`，暂未详细分析
- 写入并执行`vbs`脚本，暂未详细分析
- 遍历磁盘并加密文件

![](https://bbs.kanxue.com/upload/attach/201705/751215_xcn4jmzdgx0d5es.png)

该样本不会加密所有的文件或文件夹（否则可能会破坏系统导致系统无法启动，或者有些文件的价值并不大）。其将要感染的文件，以后缀名判断，集中放在程序中，以下是部分截图：

![](https://bbs.kanxue.com/upload/attach/201705/751215_p3rsw1yatc1sji9.png)

过滤掉特定的文件夹及非指定后缀名的文件之后，程序会对文件进行加密。其加密流程如下，它的加密思路并不是直接用长密钥的`RSA`进行全部加密，可能处于效率的考虑，它的加密思路如下：

1. 被感染用户的文件，被`AES`加密，`AES`加密所使用的`KEY`，是根据每个文件随机生成的。随机生成的`KEY`，之后会被下一步的`RSA 2048`加密。
2. 样本在针对每一台机器上，会随机生成一对`RSA 2048`密钥，其中公钥（保存在`00000000.pky`中）用于加密上一步的`AES KEY`，私钥将被加密后存在文件（`00000000.eky`）中，具体加密方式见下一步。
3. 上一步的`RSA`私钥，会被另一个`RSA 2048`公钥加密（记作`RSA2`），该`RSA2`的私钥仅勒索者本人知道。

![](https://bbs.kanxue.com/upload/attach/201705/751215_0052dfc0wf7kxp4.png)

知道以上思路后，就比较容易理解该样本的加密流程了：

1. 新建文件，新文件名为原始文件名`.WANACRYT`（`T`表示临时文件，用于临时处理文件加密）
2. 在新的文件头部先写入8字节的`WANACRY!`加密标志
3. 写入4字节的长度标识，表示后面紧跟的加密后的文件加密密钥的长度
4. 写入256字节长度的加密后的文件加密密钥（该密钥解密后为16字节长度的字符串，使用该字符串`AES`加密文件）
5. 写入4字节长度的勒索者作者定义的类型，不为4的文件类型记录到`f.wnry`文件中，用于后期免费解密
6. 写入8字节长度的原始文件长度
7. 使用随机生成的16位字节的密钥利用`AES`加密算法进行加密，并写到上述内容之后
8. 修改新文件的文件时间为源文件的时间
9. 删除原文件，并将新文件重命名为`WANACRY`
10. 若文件为可免费恢复的文件，则将文件名记录到`f.wnry`文件中

因为涉及到的代码量较多，不方便截图，感兴趣的可以去看我附带的`idb`文件。我自己也对该样本的算法进行了还原，写成了`cpp`文件，供大家参考。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [crpyt.idb](javascript:void(0);)（844.02kb，350次下载）
- [Reverse-CR24.cpp](javascript:void(0);)（7.08kb，375次下载）
- [24d004a104d4d54034dbcffc2a4b19a11f39008a575aa614ea04703480b1022c.v](javascript:void(0);)（3.55MB，582次下载）

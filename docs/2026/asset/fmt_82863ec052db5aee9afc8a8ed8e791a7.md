
第一种类型
`.doc` `.docx` `.xls` `.xlsx` `.ppt` `.pptx` `.pst` `.ost` `.msg` `.eml`
`.vsd` `.vsdx` `.123` `.wks` `.wk1` `.txt` `.csv` `.rtf` `.pdf` `.dwg`
`.jpeg` `.jpg`
第二种类型
`.docb` `.docm` `.dot` `.dotm` `.dotx` `.xlsm` `.xlsb` `.xlw`
`.xlt` `.xlm` `.xlc` `.xltx` `.xltm` `.pptm` `.pot` `.pps` `.ppsm` `.ppsx`
`.ppam` `.potx` `.potm` `.edb` `.hwp` `.602` `.vdi` `.vmdk` `.vmx` `.gpg`
`.aes` `.ARC` `.PAQ` `.tar.bz2` `.tbk` `.bak` `.tar` `.tgz` `.gz` `.7z` `.rar`
`.zip` `.backup` `.bmp` `.png` `.gif` `.raw` `.cgm` `.tif` `.tiff` `.nef`
`.psd` `.ai` `.m4u` `.m3u` `.mid` `.wma` `.flv` `.3g2` `.mkv` `.3gp`
`.mp4` `.mov` `.avi` `.asf` `.mpeg` `.vob` `.mpg` `.wmv` `.fla` `.swf`
`.wav` `.mp3` `.sh` `.class` `.jar` `.java` `.rb` `.asp` `.php` `.jsp` `.brd`
`.sch` `.dch` `.dip` `.pl` `.vb` `.js` `.h` `.pas` `.cpp` `.c` `.cs`
`.suo` `.sln` `.ldf` `.mdf` `.ibd` `.myi` `.myd` `.frm` `.odb` `.dbf` `.db`
`.mdb` `.sql` `.sqlitedb` `.sqlite3` `.asc` `.lay6` `.lay` `.mml` `.sxm` `.otg` `.odg`
`.uop` `.std` `.sxd` `.otp` `.odp` `.wb2` `.slk` `.dif` `.stc` `.sxc` `.ots`
`.ods` `.3dm` `.max` `.3ds` `.uot` `.stw` `.sxw` `.ott` `.odt` `.pem` `.p12`
`.csr` `.crt` `.key` `.pfx` `.der`

`执行加密策略`

![在这里插入图片描述](upload/attach/202003/868245_5AJSZWEF6HQR3VZ.jpg)

`函数sub_10002990分析`

![在这里插入图片描述](upload/attach/202003/868245_8FUJUNY5D8HXYAJ.jpg)

![在这里插入图片描述](upload/attach/202003/868245_MDNCU5UGSQ4BUQV.jpg)

通过分析在第一轮加密中所有的后缀是第二种类型此轮都不会加密，会放到第二轮进行加密！

### 6：第二轮加密

`函数sub_10002990分析`

![在这里插入图片描述](upload/attach/202003/868245_CUQ7JNF4MVSTEUQ.jpg)

#### 7：核心加密函数分析

无论是第一轮还是第二轮其实调用的加密函数一样的！

函数`sub_10002180`核心加密函数分析

![在这里插入图片描述](upload/attach/202003/868245_6HMEWHAWQ2SFUUU.jpg)

![在这里插入图片描述](upload/attach/202003/868245_65WQUYQN5JE8VE2.jpg)

![在这里插入图片描述](upload/attach/202003/868245_3GPZE7SUFTAEN54.jpg)

![在这里插入图片描述](upload/attach/202003/868245_JQJRC28Q582AQUT.jpg)

![在这里插入图片描述](upload/attach/202003/868245_3SH9UCGP2J93BC6.jpg)

#### 8：加密策略总结

之后在返回3/4的时候才执行加密，所以我们只关注返回3/4即可

类型为1的：

大小小于400kb，返回1，不执行加密

大小大于200mb，返回3，执行大文件加密策略

介于两者中间的，返回4，执行小文件加密策略

类型为2的：

大小小于400kb，返回1，不执行加密

大小大于200mb，返回3，执行大文件加密策略

介于两者中间的，返回4，执行小文件加密策略

大文件和小文件的加密策略不同的就是，大文件只是将头部的0x1000个字节写入到尾部，同时将头部清零，再然后写入指定的值，并不像小文件加密那样去创建新文件还要使用AES进行加密！

# 六：文件删除

这部分内容是我在看其他分析文章的时候发现他们的样本中资源释放之后都有这样一个程序`taskdl.exe`，用来删除指定目录下的后缀名为WNCRYT的文件，但是我的资源中并没有而是存在于我的样本中，所以我就顺便分析一下了。

`查壳`

![在这里插入图片描述](upload/attach/202003/868245_7GYV74SGZVNYDCX.jpg)

VC6.0编写，未加壳

使用LoadPE查看导入表，根据导入函数猜测功能

![在这里插入图片描述](upload/attach/202003/868245_9NCDBHCD62J6E66.jpg)

猜测可能和文件操作有关，因为导入函数中涉及到文件的删除和文件遍历等API。

`具体分析`

![在这里插入图片描述](upload/attach/202003/868245_6B6VMVSFB8JWQV8.jpg)

关于上述分析中的疑问

```
WCHAR RootPathName[2] = {0X20,0X00,0X3A,0X00};//其ASCII是（空格+“：”）
经查阅MSDN后了解函数GetDriveTypeA其参数根路径必须加上反斜杠，但是经过实际测试之后
即使不加反斜杠也是没有问题的！
char szPath[3] = { 0x43,0x3A,0x00 };
int type = GetDriveTypeA(szPath);
上述测试代码用来获取我电脑C盘的类型，得到的结果是正确的，所以不加就像病毒代码中的一样部件反斜杠来获取盘符类型也是没有问题的。
```

`sub_401080函数分析`

第一部分：获取系统盘temp文件夹路径，非系统盘获取RECYCLE文件夹路径，同时查找后缀名为.WNCRYT的文件

![在这里插入图片描述](upload/attach/202003/868245_Y63J8HCBASQT2PE.jpg)

第二部分：遍历查找所有后缀名为.WNCRYT的文件

![在这里插入图片描述](upload/attach/202003/868245_4JU58JSVK42F5AJ.jpg)

第三部分：通过遍历容器，删除文件，抹除对象元素，之后对容器申请的堆空间进行释放，此函数最终返回的是成功删除的文件个数！

![在这里插入图片描述](upload/attach/202003/868245_H7STF63M2BDPAK3.jpg)

补充部分：获取temp文件路径和RECYCLE文件路径函数逆向

![在这里插入图片描述](upload/attach/202003/868245_S57CUEHJ2GMWAXX.jpg)

# 总结

`加密总结`（偷个懒使用参考文章内容，分析一致）

“

`原始RSA私钥`：用于解密受害者机器上的随机RSA私钥；

`原始RSA公钥`：用于加密受害者机器上的随机RSA私钥；

`随机RSA私钥`：用于解密AES密钥；

`随机RSA公钥`：用于加密AES密钥；

`AES密钥`：用于解密受害者机器上的数据。

”

`文件总结：`

`taskdl.exe`，删除临时目录下的所有“*.WNCRYT”扩展名的临时文件。

`u.wnry`，解密程序，释放后名为@WanaDecryptor@.exe

`b.wnry`勒索图片资源。

`t.wnry`，解密后得到加密文件主要逻辑代码。

`r.wnry`，勒索Q&A。

其他的文件我不太清除了，但是火绒实验室的分析给出了结果可以参考！

# 参考文章

https://bbs.pediy.com/thread-217644.htm

https://bbs.pediy.com/thread-217662.htm

https://bbs.pediy.com/thread-249520.htm

目录

- [说明](https://bbs.kanxue.com/#说明)

- [基础分析](https://bbs.kanxue.com/#基础分析)

# 说明

本次分析主要涉及到病毒的勒索部分，主要包括如下几点：

- 资源的释放

- 解密文档为PE文件（病毒的核心加密模块）

- PE文件的加载

- 核心加密逻辑

- 文件的删除

没有病毒的传播部分，自认为能力有限，火绒实验室对传播有分析，文章链接放在下面，所以还请各位大佬酌情观看！

# 基础分析

## 基础静态

### PEID查壳

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202003/868245_KCV545CDWGDHN9Z.jpg)

### LoadPe查看导入表

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202003/868245_V99VVHRRNXK7HTH.jpg)

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202003/868245_2PXQDXVWREB4KHN.jpg)

从对KERNEL32.DLL的引用中可以了解到病毒对操作了资源和文件

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202003/868245_NRZY83ASMX226ZU.jpg)

从对ADVAPI32.DLL的引用中可以了解病毒使用了加密算法

### ResourceHacker查看资源

从PEID中可以看到其实资源的比重占据了相当大的一部分以及loadPe中对资源的操作函数调用，所以很自然的就需要查看以下资源。

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202003/868245_ZCJRZ3B9A98ZFMD.jpg)

之后我们可以把它们导出来做初步了解

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202003/868245_A559CNNRYQZSTVX.jpg)

## 基础动态

### procexp查看进程树

这里需要注意的是病毒启动了计划任务，每隔一段时间就会自动运行主程序，紧接着就会自动退出！

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202003/868245_SQC6UH6P85NJR34.jpg)

### 火绒剑行为分析

#### 文件监控

可以看到病毒用老的文件创建了许多新的文件，文件被加密同时后缀被更改。![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202003/868245_M8XB2REK99D6TEF.jpg)

#### 注册表监控

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202003/868245_G6E494MQQD5D7W8.jpg)

# 详细分析

## 一：主程序概览

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202003/868245_8YNH2W5T4SD5KBS.jpg)

## 二：资源释放

- 压缩文件格式

因为这里的资源是以压缩包的形式存在，所以有必要先了解相关的格式，对之后的代码分析很有帮助

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202003/868245_BCTQDMQN8FNYHFP.jpg)

文件头

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202003/868245_8Q4HXGRBKF86TS6.jpg)

数据核心目录

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202003/868245_P8PWMNCP7SYEXX4.jpg)

数据目录结束标识

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202003/868245_2URGFHTFNP4D99U.jpg)

具体的可以查看文章：[ZIP文件格式分析](https://bbs.kanxue.com/elink@135K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3p5J5x3o6l9%4x3e0l9%4x3e0k6Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0f1I4y4U0b7@1y4o6t1I4)

虽然不能够了解全部，但是哪些很重要还是显而易见的！

- 函数`sub_401810`(*释放资源*)概览

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202003/868245_5K7EKQVBKDQCXCU.jpg)

- 函数`sub_408820`（*保存参数*）函数分析

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202003/868245_MWGWUWJAJ7C3F8H.jpg)

函数`sub_407B10`分析

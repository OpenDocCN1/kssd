
# Wannacry勒索病毒分析

在找活了，之前有幸听过奇安信的招聘会，大佬说，Wannacry病毒现在还在某些地方流行着，是经典的勒索病毒。这让我觉得分析这个病毒是刷经验的好机会，所以就在决定分析一下这个在2017年的纵横江湖无敌手的勒索之王，由于是新手入坑，不对之处还请多多指教(●ˇ∀ˇ●)

**病毒简介：** 该病毒亦称“永恒之蓝”，它利用Windows系统的SMB漏洞可以获取系统最高权限。骇客通过改造“永恒之蓝”制作了wannacry勒索病毒，英国、俄罗斯、整个欧洲以及中国国内多个高校校内网、大型企业内网和政府机构专网中招，被勒索支付高额赎金才能解密恢复文件。

而该病毒分为蠕虫传播部分和勒索病毒部分，在这我只分析了勒索病毒Wannacry部分，由于是自己分析学习，所以我就尽量解释详细一点。下面开始分析样本。

- **分析平台**：win7虚拟机
- **分析工具**：OD、IDA
- 辅助工具：LordPE、PEID、010Editor、火绒剑、ResHacker、Hash等

## 1. 样本概况

首先获取到病毒样本然后解压

![](https://bbs.kanxue.com/upload/attach/201906/847228_CUZ6HHP4PGBN9N9.jpg)

使用Hash工具查看基本信息

![](https://bbs.kanxue.com/upload/attach/201906/847228_YDWBXAXRX72R789.jpg)

## 2. 具体行为分析

### 2.1 主要行为流程图

![](https://bbs.kanxue.com/upload/attach/201906/847228_EPH6ZP7R5SH5476.jpg)

### 2.2 病毒对用户造成的危害

加密了常用的大多数格式（约100多种）的文件，加密成后缀是`.WNCRY`文件，不花钱基本无法解密。

![](https://bbs.kanxue.com/upload/attach/201906/847228_BZ7RCR9G9THZSP5.jpg)

### 2.3 病毒行为分析

#### 2.3.1 首先使用PEID观察是什么语言、编译器、壳

结论是c/c++语言编译器是C++6.0写的程序，无壳。

![](https://bbs.kanxue.com/upload/attach/201906/847228_YKECAF3CTD6X2BY.jpg)

#### 2.3.2 LordPE观察导入表

1. 首先观察到有加载锁定资源和打开互斥体相关API，猜测有可能是释放资源操作，还有创建互斥体运行单一实例的常见病毒行为操作。

   ![](https://bbs.kanxue.com/upload/attach/201906/847228_W98TD3WNDS2475B.jpg)

2. 然后还观察到操作服务和注册表相关的API，猜测修改注册表，这基本是病毒必备的了。

   ![](https://bbs.kanxue.com/upload/attach/201906/847228_5UBVJXJH4P2DFYA.jpg)

#### 2.3.3 使用火绒剑观察行为

1. 首先查看执行监控，执行了`attrib.exe`隐藏程序、还有`cmd`和一些病毒自己释放的程序。

   ![](https://bbs.kanxue.com/upload/attach/201906/847228_ZQ7XXZ6UG3V8PQX.jpg)

2. 查看注册表监控，发现设置了注册表

   ![](https://bbs.kanxue.com/upload/attach/201906/847228_2UZ6C5VWYQVJMFP.jpg)

3. 查看文件监控，基本没有文件操作，勒索病毒怎么可能没文件操作！猜测加密操作应该是在释放的资源中完成的。

4. 查看行为监控，首先看到释放很多文件

   ![](https://bbs.kanxue.com/upload/attach/201906/847228_Z4452EQR6EPMZBM.jpg)

   还有启动自释放的文件

   ![](https://bbs.kanxue.com/upload/attach/201906/847228_4GVTHJCAABMHMHQ.jpg)

   修改注册表自启动项

   ![](https://bbs.kanxue.com/upload/attach/201906/847228_96AR7H728ZTHEPP.jpg)

5. 查看网络监控，发现监听了2个端口，和连接一些随机的IP地址可能是蠕虫传播行为。

   ![](https://bbs.kanxue.com/upload/attach/201906/847228_AJ8MFUA5XC7PHQA.jpg)

## 3. 详细分析

详细分析主要用到OD的动态分析和IDA的静态分析，只有动静结合才能更好更快更轻松的分析病毒。

### 3.1 wcry.exe程序分析

找到`WinMain`函数后分析出了最后的一览图如下：

![](https://bbs.kanxue.com/upload/attach/201906/847228_HT2A96EF3MGVM6K.jpg)

从上图中可知有几个比较关键的函数下面将一一分析他们。

**1.** `ModifyRegedit`函数修改了注册表

![](https://bbs.kanxue.com/upload/attach/201906/847228_UHZWYQPPVZ768VR.jpg)

注册了一个名称为`wd`数据是当前目录的项

![](https://bbs.kanxue.com/upload/attach/201906/847228_TJFE4DX4NU2F6VG.jpg)

**2.** `FreeRsrcFile`释放资源函数

![](https://bbs.kanxue.com/upload/attach/201906/847228_9R8D26896J7RCAX.jpg)

知道了有释放资源的相关操作，首先用ResHacker资源编辑工具查看`wcry.exe`的资源，发现一个资源类型为XIA，名称为2058的资源数据，可以看到16进制是 50 4B开头说明这个一个ZIP压缩包。

![](https://bbs.kanxue.com/upload/attach/201906/847228_ESFU7HZXWB7GCG4.jpg)

而该函数中`FindResourceA`，`LoadResource`，`LockResource`，`SizeofResource`这几个函数组合就是明显的要利用资源了，接下来的代码就是解压ZIP包释放出里面每个数据到当前目录。

![](https://bbs.kanxue.com/upload/attach/201906/847228_UWSWXB3MUQ7SH6F.jpg)

**3.** `AddWrite_cwnry`函数写入比特币账户信息

![](https://bbs.kanxue.com/upload/attach/201906/847228_BFBKY2G4PTGAEGY.jpg)

此函数首先会从3个比特币账户中伪随机抽取出一个（不同的电脑抽取到的账户不同），然后把改账户写入到`c.wnry`文件中，我的电脑写入的是"13AM4VW2dhxYgXeQepoHkHSQuy6NgaEb94"，从而得出结论`c.wnry`文件存储的是比特币账户信息，可能被用于解密器读取。

![](https://bbs.kanxue.com/upload/attach/201906/847228_4AHX6V7AFKWQVFS.jpg)

运行病毒后可查看到解密器右下角有该账户信息

![](https://bbs.kanxue.com/upload/attach/201906/847228_TVX8F8FBA6NEHTS.jpg)

**4.** `HideAndCreate`函数是以命令行和不显示GUI窗口的方式启动程序

![](https://bbs.kanxue.com/upload/attach/201906/847228_WADPD4JKCAZ7KQZ.jpg)

- 第一次传入的命令行参数是`"attrib +h ."`，该参数意为隐藏属性，运行过后发现病毒所在的当前目录不见了，在文件夹选项中点击显示隐藏文件后，可看见该目录被隐藏了！

  ![](https://bbs.kanxue.com/upload/attach/201906/847228_T92KV5CF2VYTXXR.jpg)

![](upload/attach/201906/847228_U6YQ7ZDB5GK84ZF.jpg)

然后遍历文件，挑选出需要加密的文件保存其信息到一个数据结构中

![](upload/attach/201906/847228_CRPHTX5A55JR57F.jpg)

其中`SelectEncryptFile`函数辨别出哪些是需要加密的文件，从下面的图中可知这个病毒丧心病狂，居然要加密100多种格式的文件，基本包含了我们常用的所有文件。

![](upload/attach/201906/847228_TGS89DZ8EJ3HNGV.jpg)

![](upload/attach/201906/847228_MWHCMMP9QJ726JN.jpg)

回到第四层继续，之后就进入了一个循环中，每次在数据结构中取出一个信息，调用第五层加密函数加密对应的文件。然后判断目录层级是否小于等于6是就拷贝`@Please_Read_Me@.txt`文件和`@WanaDecryptor@.exe`到当前目录。

![](upload/attach/201906/847228_E4P3ACY5N26SH5Z.jpg)

![](upload/attach/201906/847228_G6J2DECC42Q8CDF.jpg)

**（5）** `EnFive_10002940`加密函数第五层

第五层比较简单，首先判断文件是那种类型（病毒作者自定义的类型），然后根据类型的不同决定是否加密。

![](upload/attach/201906/847228_ETWNFC7F5MKQMS3.jpg)

**（6）** `EnSix_10002200`加密函数第六层

第六层也比较简单，首先获取原文件全路径，对比该文件是否是`.WNCRY`格式，不是就在其文件名后追加`.WNCRY`，然后经过第7层加密的问价就以其命名！

![](upload/attach/201906/847228_U5X3CAHXXRR7Q4N.jpg)

**（7）** `EnSeven_10001960`加密函数第七层

首先读取原文件前8个字节比较是否是"WANACRY！"，之后文件类型是4的情况下，会在新的文件名后面加上字符"T"如：`"C:\Users\15pb-win7\Desktop\1234.docx.WNCRYT"`，再创建该文件，也就是加密后的文件，此时只是没有填充内容而已。

![](upload/attach/201906/847228_9F9GC57X26NV2M9.jpg)

然后在该文件中依次写入8字节字符"WANACRY!"、某个大小数值`0x100`、再写入`0x100`大小的数据、文件的类型值和原文件的大小。

![](upload/attach/201906/847228_T23FQXDJ3F43KPD.jpg)

然后再读取原文件内容经过第八层加密后，输出加密后的内容，再把该内容追加写入到刚才创建的文件中，设置文件日期和时间信息，加密文件就到此完成了，最后做了一个移动文件的操作，实际作用就是个文件后缀`.WNCRYT`改为`.WNCRY`。

![](upload/attach/201906/847228_CKHDQT5CJWQKSYY.jpg)

**（8）** `EnAight_10006940`加密函数第八层

第八层就是通过解析原文件，每次输入16个字节通过第9层加密后输出16个字节，依次循环直到加密完成为止，然后输入加密后的缓冲区。

![](upload/attach/201906/847228_RDH9TRJTZD78CRC.jpg)

**（9）** 加密函数结语：加密函数从第八层之后就完全是数据加密相关的代码，没有其他重要的操作了，层数太多，关于加密函数的分析我也就到此为止了。

接下来回到第一层，继续分析第一层后半段的代码。

通过以上对加密函数层层分析，已经知道了加密形式是怎么进行的了，只是上面的加密只针对桌面文件进行加密，实际上从加密函数第三层开始传入的参数不同，加密的路径自然也会改变。

**（1.1）** 紧接加密后的下一个函数是`InterlockedExchange`原子操作，交换两个数。这也是由于多线程的原因，用于线程同步，让线程之间互补干扰。

**（1.2）** 接下来加密全盘文件，获取出所有磁盘类型，对固定的磁盘文件进行加密，然后再对其他用户同样进行此操作。主要加密函数是`EncryptDiskAll`。

![](upload/attach/201906/847228_C6K4UUXQKYJUNN6.jpg)

`EncryptDiskAll`函数

此函数会得到磁盘的盘符如`"C:\"`，在最后它会调用加密函数的第3层，加密全盘的数据，如有其它盘也一样。

在这之前会得到一个临时路径，只有c盘时是：`"C:\Users\15PB - W~1\AppData\Local\Temp\"`，之后我添加了一个E盘，得到的是：`"E:\$RECYCLE"`

![](upload/attach/201906/847228_NB2387W3C69W7JF.jpg)

**（1.3）** 最后一段代码

![](upload/attach/201906/847228_XQNYH9B5JECCZY5.jpg)

- 先开始以不显示GUI窗口的方式启动了解密器`@WanaDecryptor@.exe`程序，此时可看到桌面背景被更改了

- 再向`00000000.res`文件写入当前时间信息

- 运行`cmd`

- 不断的向磁盘写入数据

`WriteMassData`写入删除数据函数分析

首先该函数在之前得到的临时目录以隐藏方式创建了`hibsys.WNCRYT`文件

![](upload/attach/201906/847228_XWHQCSEWZZUBSEK.jpg)

然后获取磁盘是否还有多余空间，有责每次写入`0xA00000`个字节的`0x55`，循环写入`0x14`次，检测磁盘有无多余空间。

![](upload/attach/201906/847228_4RKDMUBMFEUUTPA.jpg)

我花了10多分钟检测了一下写满磁盘会发生什么，如下图，我的磁盘总60G被写满了之后单步跟踪OD会发现它又把这整个文件删除了，回复了原来空闲空间，我只想说一句"卧槽"，我猜测病毒作者的意图是不断的占用磁盘的性能，让用户欲罢不能。

![](upload/attach/201906/847228_2EBPKVJ2GG9AU3E.jpg)

到此`TaskStart`函数就分析完毕，在整个分析过程中由于是多线程程序，有些函数的分支我分析的时候不满足条件，我就没有分析下去，所以，肯定有些关键点我没分析到，比如创建了加密文件后，会删除原文件，我就没分析到删除原文件是哪里进行的，这不得不说是个小遗憾，由于是新手分析了10来天，我有点小崩溃不想继续分析了，我也算对得起WannaCry了，哈哈，接下来分析最后两个程序`taskdl.exe`和`taskse.exe`。

### 3.3 taskdl.exe程序分析

`taskdl.exe`程序是在`TaskStart`函数的第4个线程中被运行的。现在分析它，当然前面的步骤都一样，查壳，看是什么程序等等。

找到`WinMain`函数可以看到，这个程序就是用来删除后缀为`.WNCRT`的文件的。

![](upload/attach/201906/847228_V9TBN9MXG3NKPMR.jpg)

进入`DeleteFile11`函数中发现，这就是获取上面才分析过的临时路径，他删除里面的`.WNCRYT`文件实际就是删除`hibsys.WNCRYT`这个文件。

所以得出结论`taskdl.exe`的功能就是删除`hibsys.WNCRYT`文件。

![](upload/attach/201906/847228_UY8K4D4JFK9TA49.jpg)

![](upload/attach/201906/847228_URQ8WUHYMM3JX9X.jpg)

### 3.4 taskse.exe程序分析

`TaskStart`的第5个线程运行了此程序，分析这个程序的时候我们要注意用OD打开它后需要添加点击"调试" ->"参数"附加参数`taskse.exe C:\Users\15pb-win7\Desktop\wcry\wcry2.0\@WanaDecryptor@.exe`才能正常运行。

![](upload/attach/201906/847228_DPUMNWG5XXUMHK9.jpg)

最开始判断参数是否符合，然后进入提权函数

![](upload/attach/201906/847228_R3T2Y4GNFT4SZ7P.jpg)

进入`ChangePrivileges`函数中的`ChangePrivilege`函数

首先获取了一些权限相关的API函数的地址，然后进行提权操作，所以`taskse.exe`程序的功能就是进行提权。

![](upload/attach/201906/847228_PQC34275HFF3HBF.jpg)

![](upload/attach/201906/847228_MTUSUPCV9U94GWB.jpg)

最后还有一个解密器`@WanaDecryptor@.exe`，我就不分析了，直接运行它，就可知道它的功能，生成一张图片，把它设置为桌面背景，然后每个几秒弹出一个勒索窗口让用户交钱。

## 4. 解决方案

- 因为采用高强度的RSA和AES双重加密，所以如果你的文件已被加密，只有交钱才能解密，交了钱也不一定100%能解密。

- 电脑打补丁，因为此病毒是利用“永恒之蓝”漏洞进行传播的，打完补丁后就阻止了蠕虫病毒的传播。

- 可以创建名为“MsWinZonesCacheCounterMutexA”互斥体阻止病毒运行。

- 把该病毒的特征码加入病毒库，一经发现立马查杀。

最后病毒样本(解压密码15pb)、`wcry.exe`的UDD，和一些IDB分析文件打包上传！

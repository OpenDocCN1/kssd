
![](https://bbs.kanxue.com/upload/attach/202002/860819_K83GEJHY8ZS6BXE.jpg)

图2.5.1.11 要加密的系统信息

加密后写入注册表中。

![](https://bbs.kanxue.com/upload/attach/202002/860819_C6ANMFZAXKVRJAH.jpg)

图2.5.1.12

在勒索文本内存中添加`{UID}{KEY}{EXT}`的值。

![](https://bbs.kanxue.com/upload/attach/202002/860819_8QQJ9KTYFZSZMD5.jpg)

图2.5.1.13

#### 2.5.1.7 `Readme`

调用了`SetReadmeName`函数设置勒索信文件名。

![](https://bbs.kanxue.com/upload/attach/202002/860819_S8KHR486A3E4SR3.jpg)

图2.5.1.14 勒索文件名

`AppendReadme`函数继续在`Readme`中补充数据。

![](https://bbs.kanxue.com/upload/attach/202002/860819_N8JWGCD2HKSV8TZ.jpg)

图2.5.1.15

至此，这部分的生成加密密钥，获取系统信息计算作为标识ID，写入勒索文本等操作完成。

### 2.5.2 `RegWriteFilePath()`

图2.5.1的16行，这个函数在注册表中写入病毒文件路径。

![](https://bbs.kanxue.com/upload/attach/202002/860819_Z8FGJZYGPTGHJBJ.jpg)

图2.5.2.1

### 2.5.3 `GetProcessSnapshot()`

这个函数获取当前运行的进程，并与配置文件中的进程列表进行比较，如果相同，则结束该进程。

![](https://bbs.kanxue.com/upload/attach/202002/860819_AB979GUEVEX4EYV.jpg)

图2.5.3.1

![](https://bbs.kanxue.com/upload/attach/202002/860819_F33QN8SXATYTFTA.jpg)

图2.5.3.2

### 2.5.4 `CloseService()`

图2.5.1的21行`CloseService`函数，连接服务控制管理器，删除存在配置文件中的服务(`svc`字段)。

![](https://bbs.kanxue.com/upload/attach/202002/860819_EJE4EDU67X4JRRG.jpg)

图2.5.4.1

下图显示的是枚举当前系统的服务。

![](https://bbs.kanxue.com/upload/attach/202002/860819_E5S39WTZ5FR5Z2P.jpg)

图2.5.4.2

### 2.5.5 `DeleteSystemShadowcopy()`

图2.5.1的22行，这个函数的作用是删除系统卷影信息。首先，解密出base64加密了的`powershell`命令，然后调用`CreateProcessW`执行删除系统卷影的`powershell`命令。

![](https://bbs.kanxue.com/upload/attach/202002/860819_ZM2YAMUBA9BFKMY.jpg)

图2.5.5.1

### 2.5.6 `MainCrypt()`

`MainCrypt`函数主要功能包括：

1. 创建了两个线程关联I/O完成端口分别执行入队和出队操作
2. 遍历所有文件夹，加密所有不在配置白名单（文件名和后缀）的文件；在所有文件夹下生成勒索文本和加密文件。
3. 多次获取宿主机的网络共享文件，并将其中的文件加密。

![](https://bbs.kanxue.com/upload/attach/202002/860819_F2DY442T9SSEP4R.jpg)

图2.5.6.1

下面分析主要的加密函数`TraverseFilesAndCrypt()`。

#### 2.5.6.1 `TraverseFilesAndCrypt()`

这个函数遍历磁盘A-Z，然后递归调用遍历函数。

![](https://bbs.kanxue.com/upload/attach/202002/860819_2ZPVDUB9MUCM63S.jpg)

图2.5.6.2

进入递归函数，整体逻辑如下。

![](https://bbs.kanxue.com/upload/attach/202002/860819_QX5ZAUPJN8M6E26.jpg)

图2.5.6.3

如果是文件夹，则调用`IsWhiteFolder`函数判断是否属于不加密的文件夹。

![](https://bbs.kanxue.com/upload/attach/202002/860819_7MU2RW9C95RM9HC.jpg)

图2.5.6.4

然后，调用`CreateReadme`函数，在当前文件目录下写入勒索信。

![](https://bbs.kanxue.com/upload/attach/202002/860819_MQ3X78ZWBDXYF5S.jpg)

图2.5.6.5

如果是文件，则调用`GetFileExt`函数获取文件扩展名。

![](https://bbs.kanxue.com/upload/attach/202002/860819_JYCWDEQK8Z7CXKH.jpg)

图2.5.6.6

接着，调用`CryptFileAndWriteFileByIO`函数，对文件进行加密，并将加密文件内容入队到I/O完成端口中。

![](https://bbs.kanxue.com/upload/attach/202002/860819_XCCBF97AFSC54QX.jpg)

图2.5.6.7

图2.5.6.7中的`CryptFile`加密函数，打开文件并获取了文件的读写属性，再调用真正的加密函数来加密文件。

![](https://bbs.kanxue.com/upload/attach/202002/860819_TCM62YP96DMYGEU.jpg)

图2.5.6.8

加密过程中，主线程进行加密计算，创建两个新线程，一个线程将I/O完成包入队到I/O完成端口，另一个线程出队将数据写入磁盘。这样可以加快文件加密效率。

![](https://bbs.kanxue.com/upload/attach/202002/860819_EHA723SDM3D352F.jpg)

图2.5.6.9

加密完成，效果如下，文件夹下包含勒索信和所有加密文件。

![](https://bbs.kanxue.com/upload/attach/202002/860819_JWN432ZYJHY2HKE.jpg)

图2.5.6.10

#### 2.5.6.2 `GetNetResourceAndCrypt()`

这个函数获取了宿主机网络共享资源，然后调用加密函数加密文件。

![](https://bbs.kanxue.com/upload/attach/202002/860819_H33525QCUYMECYN.jpg)

图2.5.6.11

### 2.5.7 `SetWallpaper()`

这个函数修改了桌面壁纸，并在桌面上绘制勒索信息。

![](https://bbs.kanxue.com/upload/attach/202002/860819_GJEZ3AGQPQPZSKY.jpg)

图2.5.7.1

桌面显示效果如下。

![](https://bbs.kanxue.com/upload/attach/202002/860819_PNRJN3366BNE9AY.jpg)

图2.5.7.2

### 2.5.8 `SendRequest()`

这个函数，实现了如下功能：根据域名表构造url；向域名指向的服务器发送本机系统信息；处理返回的html文本信息。

![](https://bbs.kanxue.com/upload/attach/202002/860819_SVEXMBAGFER6KV9.jpg)

图2.5.8.1

for循环遍历配置中域名表，`GetDomian`函数每次执行返回一个域名。

![](https://bbs.kanxue.com/upload/attach/202002/860819_FWNFYHMJXHPAHVC.jpg)

图2.5.8.2

然后调用`JoinUrlAndSendHttpRequest`函数，其函数逻辑如下。

![](https://bbs.kanxue.com/upload/attach/202002/860819_76BYJ3QZS4AERKS.jpg)

图2.5.8.3

其中，首先调用了`GenerateUrl`函数生成一个URL。这个URL根据生成规则拼接而成的，拼接规则如下图。

例如：`"418K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1N6i4c8Z5L8r3g2J5i4K6u0W2L8X3I4Q4x3V1k6K6N6r3q4@1K9h3y4Q4x3V1k6@1k6h3#2H3i4K6u0r3L8r3!0#2M7X3A6I4P5X3W2J5L8#2)9J5k6h3A6H3k6H3`.`."`

![](https://bbs.kanxue.com/upload/attach/202002/860819_73349JWZ67BTJ8R.jpg)

调用`SendHttpRequestAndGetRespone`函数，向服务器发送HTTP消息，消息内容是存在注册表中的加密的宿主机系统信息：

![](https://bbs.kanxue.com/upload/attach/202002/860819_68H7RYFCT986C88.jpg)

图2.5.8.4

接着调用`WinHttpReadData`函数等待读取返回的html数据，但并未对返回的消息有实质的操作。

![](https://bbs.kanxue.com/upload/attach/202002/860819_58RCKQYU6GZZRCE.jpg)

图2.5.8.5

分析时，其中一个C2服务器返回了一个HTML文件(但并未发现有实质内容，而且这些域名并非全是恶意域名)。

![](https://bbs.kanxue.com/upload/attach/202002/860819_A7YDE432KARQWQB.jpg)

图2.5.8.6

病毒进程遍历完上千个域名发送http消息后，所有功能执行完毕，结束进程退出。

# 3. 病毒流程

![](https://bbs.kanxue.com/upload/attach/202002/860819_P8HV6HTCDFNTA8C.jpg)

# 4. 参考

1. 勒索软件Sodinokibi运营组织的关联分析

`9a9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4c8A6P5g2)9J5k6h3y4G2L8g2)9J5c8Y4u0W2M7%4m8G2L8Y4y4W2i4K6u0r3x3U0l9I4z5e0l9$3x3U0S2Q4x3X3g2Z5N6r3#2D9`

2. Sodinokibi 病毒分析报告

`014K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4W2j5%4N6C8i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4z5g2)9J5c8U0l9&6i4K6u0r3x3U0y4Q4x3V1j5%4y4K6l9K6i4K6u0r3`

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2020-2-16 18:06

被kumqu编辑

，原因：

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)

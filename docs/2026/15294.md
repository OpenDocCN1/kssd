
# [原创]Android 锁机恶意程序分析

- 环境：MuMu 模拟器
- 工具：`jeb.exe`、`ida`、`Hash`
- 病毒名称：荒野行动盒子 root 锁机（伪装成辅助-外挂）
- Md5：`5147E30F1FDB564E54C10CFE0642BCE5`
- Shar1：`06BECD338ABC95C14549052034EF2C84017556F4`
- CRC32：`C74F4CAF`
- 病毒功能点：病毒以伪装自身为一款第三方插件诱惑普通用户，在用户给与权限运行后直接将用户手机锁住。

## 1. 开始

![](https://bbs.kanxue.com/upload/attach/202005/861573_QCTEEHEYV3CQ7J3.png)

图-1.1

锁机状态，使用 `adb` 将病毒提取（图-1.1）。

### 1.1 提取病毒

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_ZVVGP3CAPBEH3RJ.png)

图-1.2

为什么找不到模拟器设备？因为我们使用的是 MuMu 模拟器，要连接指定端口（图-1.2）。

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_FEBQA562NMWBWKD.png)

图-1.3

现在即可寻找 MuMu 模拟器并且进入设备（图-1.3），那就开始提取病毒（一段小插曲）。

先要检查三大敏感目录：

1. `/System/app`：系统预装 app 所在目录，默认不可写，但是如果有 root 权限则可以修改。
2. `data/app`：用户安装 app 所在目录，存放完整 apk 文件。
3. `data/data/包名`：存放安装 app 的所有配置文件信息。

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_B8WEM229T5QZP6H.png)

图-1.4 检查 `/System/app` 目录

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_55VMGX4PYKTACTX.png)

图-1.5 进入存放 app 应用的指定位置 `data/app`

上面说过，这里是存放用户安装的 apk 位置，那么 `com.Android.lock.kezi-2` 属于敏感文件，因为包名中带 `Android` 的正常软件是没有的，所以将其提取查看。

`data/data/app` 敏感文件提取、清理文件：

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_PGQBC9KT72BC7EF.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_84UMVHTT5WRR5N3.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_UFWN975NSVHA6NT.png)

图-1.6 正常提取后电脑管家发现了是病毒（代表有恶意行为），所以我们寻找病毒没错。

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_BX78V22795YR4Y2.png)

图-1.7 将病毒原件拿到手后，查看 `md5`。

拿到 `md5` 上网进行查询，也可以自己分析。

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_8VAFXAT9AX24AQM.png)

图-1.8 某病毒上传网站反馈

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_KZSPM5TAN9P97T7.png)

图-1.9 这是 2018 年的报告，现在（2020/1/6）应该可以杀。

找到它的前身后，开始代码分析，找到正确的解锁码。

### 1.2 简单分析

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_79ASN7FPUXXBZNF.png)

图-1.a 确定包位置

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_7HR55M86UVNXNZ4.png)

图-1.b 确定可以靠名字确定功能类，为了无懈可击还是去看看类的构造

先从表面看过之后，要确定程序入口点，从而一步一步进行分析。

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_HPQTBXE5445N7YW.png)

图-1.c 确定从日志广播类进行启动

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_Y25MYEYRR4QB4D8.png)

图-1.d 确定 `start` 类

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_H9HQM4R2M79PWFS.png)

图1-e `started` 默认是 `false`，但是变量是 `static`，所以只初始化一次。

## 2. Cute.e.busybox 恶意行为（提权等）

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_P32DH4EAKUCWPGQ.png)

图-2.1 创建新文件

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_MF3CQH4CV6YNG4H.png)

图-2.2 定义字符串

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_66NTRAUBWD9WNFS.png)

图-2.3 针对 Android 5.0 以下的版本

如果是 5.0 以下的 Android 版本，病毒目录会保存到 `system/app` 目录下，名为 `com.h1.app`。

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_5JP8XKRCWMQA98M.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_8M3C3M4Q9UUGHSA.png)

图-2.4 针对 Android 5.0 以上版本 && 存在葫芦侠 app

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_GXP9RWDSMHUC7DT.png)

图-2.5 没有葫芦侠的代码

如果是 5.0 以上的 Android 版本，则将病毒目录放到 `system/priv-app/`，病毒文件 `com.h1.app`，并且将 `key.bat` 改名为 `com.h1.app.apk`。

主要的恶意行为都放到了 `e` 类中，如果这样看着不方便，可以将代码拷贝到 Android Studio 去查看。

查看模拟器 SDK 版本：

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_8YCC659N2V7GQZA.png)

图-2.6 Android 版本

进入病毒保存文件的目录：

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_YSQGTA3HGC9WPWA.png)

图-2.7 病毒目录

> 病毒为什么要将 `app` 放到 `priv-app` 中呢？
>
> 因为 `priv-app` 目录下的 `app` 文件是不会被 `kill` 掉的，即使你 `kill` 掉后，系统还会重新拉起来。锁机病毒肯定会存在这里，为了保护自己不被 `kill` 掉。

坏处是无法升级，当然像这样的病毒也不会要升级。

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_SZHVNXB7UMU9XYK.png)

图-2.8 测试 1

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_FU2R5RCN5NX59J6.png)

图-2.9 测试 2：`kill` 掉病毒进程后，发现屏幕变好了。

![图片描述](https://bbs.kanxue.com/upload/attach/202005/861573_3XHXB63SNV4SP43.png)

图-2.a 但是下一秒又被进程占据，原因就是 `priv-app` 目录。

到这里呢，我们的病毒可以从手机上彻底清除，在删除之前将病毒样本导出。

> 使用 `mount -o rw/remount /system` 即可开启权限，将 `com.h1.app` 从 `priv-app` 目录下删除。

可以尝试写专杀工具及破解加密密码。

样本在附件。

如果没思路的，可以看这个帖子哦：

`76cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3f1#2x3Y4m8G2K9X3W2W2i4K6u0W2j5$3&6Q4x3V1k6@1K9s2u0W2j5h3c8Q4x3X3b7$3y4K6R3$3x3o6u0Q4x3X3b7I4i4K6u0V1x3g2)9J5k6h3S2@1L8h3H3`.

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2020-5-15 09:56 被 System32 编辑，原因：

上传的附件：

- [荒野行动盒子.rar](javascript:void(0);)（1.86MB，214次下载）

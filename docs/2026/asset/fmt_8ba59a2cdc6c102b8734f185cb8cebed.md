
# windows10及以上本地驱动加载黑名单查看

之前测试程序一直用上海域联的签名进行测试，后来有一天发现用不了，突然意识到windows开始使用证书黑名单策略了。它的这个策略不仅会在系统启动以后拦截驱动加载，还可以拦截boot启动的驱动。它的这个策略是通过系统更新下发的。如果你的系统已经加载了黑签名签过的驱动，策略更新将会失败，这样有效的避免了因拦截驱动加载可能引发的一系列问题。

昨天看到论坛上有人讨论这个问题，[原帖在这里](https://bbs.pediy.com/thread-272998.htm)，评论区有人提到了这个问题且给出了微软的官方说明

既然这个文件更新到本地的话，它到底保存在哪里呢？可不可以查看？可不可以修改呢？带着这些问题我出发了。最后在`C:\Windows\System32\CodeIntegrity\driversipolicy.p7b`下找到了。

结果发现并不是XML格式。后来在github上找到了把二进制转化为XML的脚本。

## 操作步骤

1、启用脚本执行 `set-ExecutionPolicy RemoteSigned`

![](upload/attach/202206/444209_VC8SHZZR4AUHHEH.jpg)

注：必须使用管理员权限

2、导入脚本，我的脚本文件夹在桌面。

`cd C:\Users\test\Desktop`

`Import-Module .\WDACTools`

3、执行脚本函数转换格式 `ConvertTo-WDACCodeIntegrityPolicy`

然后依次输入要转换的源文件路径和转换后的文件路径

![](upload/attach/202206/444209_J22AXQ7HNRQNV4T.jpg)

![](upload/attach/202206/444209_BZWAJ83ENS3AKC5.jpg)

到这里已经转换成功了。

XML格式转p7b的话，直接使用原生命令就可以

`ConvertFrom-CIPolicy -XmlFilePath XML路径 -BinaryFilePath p7b文件路径`

修改完以后重新转换成p7b格式替换掉原来的文件就可以了。

## 参考来源

[启用脚本执行](https://bbs.kanxue.com/elink@d24K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0b7&6x3K6b7&6y4U0l9^5z5b7`.`.)

[xml格式转p7b格式](https://bbs.kanxue.com/elink@d47K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4m8G2N6$3g2J5M7$3S2W2L8r3I4Q4x3V1k6E0L8$3c8#2L8r3g2Q4x3V1k6U0L8$3&6X3K9h3N6U0K9g2)9J5c8X3y4G2L8Y4k6W2M7Y4c8X3M7X3!0E0i4K6u0V1j5$3W2H3L8$3I4A6j5%4W2Q4x3@1k6$3K9h3g2%4i4K6y4p5N6$3W2F1k6r3!0%4M7%4y4W2M7Y4k6W2M7U0t1H3x3U0u0Q4x3X3c8H3M7H3`.`.)

[脚本github地址](https://bbs.kanxue.com/elink@e18K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0j5i4c8@1K9h3k6W2M7%4c8S2N6r3W2G2L8W2)9J5c8W2N6p5b7f1y4f1L8$3!0D9M7H3`.`.)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2022-6-16 07:44 被简单未遂编辑，原因：

[#开源分享](https://bbs.kanxue.com/forum-41-1-134.htm)

上传的附件：

- [WDACTools.7z](javascript:void(0);)

（26.88kb，470次下载）

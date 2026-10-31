
# [原创]IOS 脱壳入坑经验分享

目前试验 `Clutch`、`dumpdecrypted`、`CrackerXI`、`frida-ios-dump` 这四种脱壳工具，其中效果最好、最简单的就是 `CrackerXI`，`frida-ios-dump` 排名第二，因为有的 `APP` 还是脱不了，可能是手机版本或者 `Frida` 版本的问题。`Clutch` 和 `dumpdecrypted` 试了很久，都失败了，可能他们真的很强，但是我是没有脱成功，所有流程都试了，不行，估计版本问题是硬伤。

![图片描述](upload/attach/202212/961239_ZJNZ7HU5E4PG6NG.png)

## CrackerXI

### 第一步：Cydia 中添加软件源 <mark class="encrypted">e3fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4H3N6q4)9J5k6i4N6^5K9r3u0@1M7#2)9J5k6h3y4G2L8g2)9J5c8R3`.`.</mark> 如果源添加失败了，请添加 <mark class="encrypted">5d4K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4H3N6q4)9J5k6i4N6^5K9r3u0@1M7#2)9J5k6i4k6A6M7l9`.`.</mark>

### 第二步：打开 `CrackerXI` 软件, `Setting` 中打开 `Hook`

### 第三步：在 `AppList` 中点击想要脱壳的 `app`, 会有弹框

### 第四步：选择"YES,Full IPA"的话, 最后得到的是 `ipa` 文件, 需要自己解压, 再找出二进制文件, 再用 `class-dump` 将二进制文件中的头文件 `dump` 出来。

## frida-ios-dump

### 第一步：打开手机上的 `cydia`，添加软件源，地址: <mark class="encrypted">a7bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3W2D9k6q4)9J5k6h3k6J5K9h3c8S2i4K6u0W2M7X3f1`.</mark>
然后搜索: `Frida`，点击安装。所有的教程第一步都是这样，但是我的手机就不行，如果你成功了，恭喜你。如果失败，请如下操作：

我们添加 多米诺骨牌源 ： <mark class="encrypted">969K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7s2c8Q4x3X3g2%4P5r3S2T1N6s2y4Q4x3X3g2$3K9i4l9`.</mark>

### 第二步：搜索: `Frida`. 出现如下图

![图片描述](upload/attach/202212/961239_8YM99PCNWXC9QMM.png)

### 第三步：`frida-ps -U`

![图片描述](upload/attach/202212/961239_URAKEW75VHJYA6Q.png)

### 第四步：安装其他配置

`frida-ios-dump` 源码下载 <mark class="encrypted">47cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8U0L8$3c8W2i4K6u0W2L8X3g2@1i4K6u0r3L8h3W2J5M7X3!0J5M7#2)9J5c8V1q4D9L8$3&6W2e0h3!0F1K9$3g2&6i4K6u0r3k6Y4u0A6k6r3q4Q4x3X3c8A6L8%4y4Q4x3X3c8V1N6h3#2H3i4K6y4r3N6i4c8E0i4K6g2X3M7$3!0#2M7X3y4W2i4K6y4p5j5%4y4V1L8W2)9#2k6X3N6A6N6r3S2#2j5W2)9#2k6X3q4U0j5$3g2D9k6i4u0S2N6r3!0J5</mark>

1、 `dump.py` 主要功能文件，可查询手机安装软件；程序脱壳；
2、 `requirements.txt` 需要的所有 `Python` 库。

进入刚下载的 `frida-ios-dump` 文件目录，执行：

```bash
pip3 install -r requirements.txt --upgrade
python3 dump.py -l
```

### 第五步：配置

1、打开 `dump.py`，并且修改里面的内容找到里面的这4行, 分别输入你自己的用户名, 密码，内网 `IP`，端口。

![图片描述](upload/attach/202212/961239_A6Y84DZC9B4A8V6.png)

确保手机与电脑在同一局域网内

反正我出现了各种报错 ：

![图片描述](upload/attach/202212/961239_G455BC8N6PEVVN5.png)

解决：

![图片描述](upload/attach/202212/961239_Y66KXHM65YRTU9Y.png)

```bash
python3 dump.py com.x.pinduoduo
```

![图片描述](upload/attach/202212/961239_XSJZRCMC4G6D9DJ.png)

最后成功了，再试下其他的

![图片描述](upload/attach/202212/961239_WPGJS867KYUYGKK.png)

又报错了，可能还是版本的问题，但是安装流程没问题，不然拼夕夕也不会成功。

## Clutch

`Clutch`：<mark class="encrypted">a60K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6w2d9V1y4J5j5h3y4C8M7#2)9J5c8V1y4D9N6i4c8U0K9l9`.`.</mark>

如果在 `iPhone` 上执行 `Clutch` 指令，权限不够，赋予“可执行的权限”

![图片描述](upload/attach/202212/961239_GZBR4KYNG3HTV73.png)

![图片描述](upload/attach/202212/961239_XUVB9E48WWPBKQG.png)

然后进行脱壳， `Clutch -d` 序列号 （听说弃用了，反着我一个都没有成功）

## dumpdecrypted

下载源代码 <mark class="encrypted">943K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6r3g2X3j5h3&6W2M7%4y4W2M7W2)9J5c8X3c8#2L8i4m8V1k6h3y4J5P5i4m8@1k6h3c8Q4x3V1k6Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0N6Q4z5o6c8Q4b7U0k6Q4c8e0g2Q4z5e0m8Q4z5p5g2Q4c8e0g2Q4z5f1y4Q4b7e0S2Q4c8e0k6Q4b7V1q4Q4z5e0m8Q4c8e0c8Q4b7V1u0Q4b7e0y4Q4c8e0N6Q4b7e0m8Q4z5o6q4Q4c8e0N6Q4z5f1u0Q4b7f1g2Q4c8e0g2Q4b7V1c8Q4z5e0g2Q4c8e0k6Q4z5o6W2Q4b7e0N6Q4c8e0S2Q4b7e0q4Q4z5p5y4E0j5h3E0W2i4@1f1$3i4K6S2o6i4K6R3%4i4@1f1@1i4@1u0n7i4@1p5@1i4@1f1^5i4@1u0r3i4K6W2n7i4@1f1^5i4@1p5I4i4K6S2o6i4@1f1%4i4@1u0o6i4K6V1$3i4@1f1^5i4@1q4r3i4K6V1I4i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1^5i4K6S2q4i4@1t1%4i4@1f1#2i4@1u0q4i4K6V1%4k6s2W2D9K9h3u0Q4c8e0g2Q4z5p5q4Q4b7e0S2Q4c8e0k6Q4z5o6m8Q4z5o6q4Q4c8e0g2Q4b7V1q4Q4z5e0y4Q4c8e0k6Q4z5e0k6Q4z5o6N6Q4c8e0c8Q4b7V1u0Q4b7U0j5`.</mark>

将 `dylib` 文件拷贝到 `iPhone` 上（如果是 `root` 用户，建议放 `/var/root` 目录）

终端进入 `dylib` 所在的目录

使用环境变量 `DYLD_INSERT_LIBRARIES` 将 `dylib` 注入到需要脱壳的可执行文件（可执行文件路径可以通过 `ps -A` 查看获取）

```bash
DYLD_INSERT_LIBRARIES=dumpdecrypted.dylib 可执行文件路径
```

最后还是很多报错，各种失败。分享个大神写的帖子，按照他的方案来了，我最终也没有成功 https://bbs.pediy.com/thread-263907.htm

## 总结 最好用的是 CrackerXI 然后是 frida-ios-dump

目前试验 `Clutch`、`dumpdecrypted`、`CrackerXI`、`frida-ios-dump` 这四种脱壳工具，其中效果最好、最简单的就是 `CrackerXI`，`frida-ios-dump` 排名第二，因为有的 `APP` 还是脱不了，可能是手机版本或者 `Frida` 版本的问题。`Clutch` 和 `dumpdecrypted` 试了很久，都失败了，可能他们真的很强，但是我是没有脱成功，所有流程都试了，不行，估计版本问题是硬伤。

![图片描述](https://bbs.kanxue.com/upload/attach/202212/961239_ZJNZ7HU5E4PG6NG.png)

### CrackerXI

#### 第一步：Cydia 中添加软件源 375K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4H3N6q4)9J5k6i4N6^5K9r3u0@1M7#2)9J5k6h3y4G2L8g2)9J5c8R3`.`. 如果源添加失败了，请添加 d70K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4H3N6q4)9J5k6i4N6^5K9r3u0@1M7#2)9J5k6i4k6A6M7l9`.`.

#### 第二步：打开CrackerXI软件, Setting中打开Hook

#### 第三步：在AppList中点击想要脱壳的app,会有弹框

#### 第四步：选择"YES,Full IPA"的话,最后得到的是ipa文件,需要自己解压,再找出二进制文件,再用class-dump将二进制文件中的头文件dump出来。

### frida-ios-dump

#### 第一步：打开手机上的cydia，添加软件源，地址: b2eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3W2D9k6q4)9J5k6h3k6J5K9h3c8S2i4K6u0W2M7X3f1`.

然后搜索:Frida，点击安装。所有的教程第一步都是这样，但是我的手机就不行，如果你成功了，恭喜你。如果失败，请如下操作：

我们添加 多米诺骨牌源 ： ef0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7s2c8Q4x3X3g2%4P5r3S2T1N6s2y4Q4x3X3g2$3K9i4l9`.

#### 第二步：搜索:Frida. 出现如下图

![图片描述](https://bbs.kanxue.com/upload/attach/202212/961239_8YM99PCNWXC9QMM.png)

#### 第三步 ：frida-ps -U

![图片描述](https://bbs.kanxue.com/upload/attach/202212/961239_URAKEW75VHJYA6Q.png)

#### 第四步 ： 安装其他配置

frida-ios-dump 源码下载 710K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8U0L8$3c8W2i4K6u0W2L8X3g2@1i4K6u0r3L8h3W2J5M7X3!0J5M7#2)9J5c8V1q4D9L8$3&6W2e0h3!0F1K9$3g2&6i4K6u0r3k6Y4u0A6k6r3q4Q4x3X3c8A6L8%4y4Q4x3X3c8V1N6h3#2H3i4K6y4r3N6i4c8E0i4K6g2X3M7$3!0#2M7X3y4W2i4K6y4p5j5%4y4V1L8W2)9#2k6X3N6A6N6r3S2#2j5W2)9#2k6X3q4U0j5$3g2D9k6i4u0S2N6r3!0J5

1、dump.py主要功能文件，可查询手机安装软件；程序脱壳；

2、requirements.txt需要的所有Python库。

进入刚下载的frida-ios-dump文件目录，执行：

```bash
pip3 install -r requirements.txt --upgrade
python3 dump.py -l
```

#### 第五步： 配置

1、打开dump.py，并且修改里面的内容找到里面的这4行,分别输入你自己的用户名,密码，内网IP，端口。

![图片描述](https://bbs.kanxue.com/upload/attach/202212/961239_A6Y84DZC9B4A8V6.png)

确保手机与电脑在同一局域网内

反正我出现了各种报错 ：

![图片描述](https://bbs.kanxue.com/upload/attach/202212/961239_G455BC8N6PEVVN5.png)

解决：

![图片描述](https://bbs.kanxue.com/upload/attach/202212/961239_Y66KXHM65YRTU9Y.png)

```bash
python3 dump.py com.x.pinduoduo
```

![图片描述](https://bbs.kanxue.com/upload/attach/202212/961239_XSJZRCMC4G6D9DJ.png)

最后成功了，再试下其他的

![图片描述](https://bbs.kanxue.com/upload/attach/202212/961239_WPGJS867KYUYGKK.png)

又报错了，可能还是版本的问题，但是安装流程没问题，不然拼夕夕也不会成功。

### Clutch

Clutch：143K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6w2d9V1y4J5j5h3y4C8M7#2)9J5c8V1y4D9N6i4c8U0K9l9`.`.

如果在iPhone上执行Clutch指令，权限不够，赋予“可执行的权限”

![图片描述](https://bbs.kanxue.com/upload/attach/202212/961239_GZBR4KYNG3HTV73.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202212/961239_XUVB9E48WWPBKQG.png)

然后进行脱壳，Clutch -d 序列号 （听说弃用了，反着我一个都没有成功）

### dumpdecrypted

下载源代码 c3cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6r3g2X3j5h3&6W2M7%4y4W2M7W2)9J5c8X3c8#2L8i4m8V1k6h3y4J5P5i4m8@1k6h3c8Q4x3V1k6Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0N6Q4z5o6c8Q4b7U0k6Q4c8e0g2Q4z5e0m8Q4z5p5g2Q4c8e0g2Q4z5f1y4Q4b7e0S2Q4c8e0k6Q4b7V1q4Q4z5e0m8Q4c8e0c8Q4b7V1u0Q4b7e0y4Q4c8e0N6Q4b7e0m8Q4z5o6q4Q4c8e0N6Q4z5f1u0Q4b7f1g2Q4c8e0g2Q4b7V1c8Q4z5e0g2Q4c8e0k6Q4z5o6W2Q4b7e0N6Q4c8e0S2Q4b7e0q4Q4z5p5y4E0j5h3E0W2i4@1f1$3i4K6S2o6i4K6R3%4i4@1f1@1i4@1u0n7i4@1p5@1i4@1f1^5i4@1u0r3i4K6W2n7i4@1f1^5i4@1p5I4i4K6S2o6i4@1f1%4i4@1u0o6i4K6V1$3i4@1f1^5i4@1q4r3i4K6V1I4i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1^5i4K6S2q4i4@1t1%4i4@1f1#2i4@1u0q4i4K6V1%4k6s2W2D9K9h3u0Q4c8e0g2Q4z5p5q4Q4b7e0S2Q4c8e0k6Q4z5o6m8Q4z5o6q4Q4c8e0g2Q4b7V1q4Q4z5e0y4Q4c8e0k6Q4z5e0k6Q4z5o6N6Q4c8e0c8Q4b7V1u0Q4b7U0j5`.

将dylib文件拷贝到iPhone上（如果是root用户，建议放/var/root目录）

终端进入dylib所在的目录

使用环境变量DYLD_INSERT_LIBRARIES将dylib注入到需要脱壳的可执行文件（可执行文件路径可以通过ps -A查看获取）

```bash
DYLD_INSERT_LIBRARIES=dumpdecrypted.dylib 可执行文件路径
```

最后还是很多报错，各种失败。分享个大神写的帖子，按照他的方案来了，我最终也没有成功 https://bbs.pediy.com/thread-263907.htm

### 总结 最好用的是 CrackerXI 然后是 frida-ios-dump

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#逆向分析](https://bbs.kanxue.com/forum-166-1-189.htm)

[#工具脚本](https://bbs.kanxue.com/forum-166-1-195.htm)

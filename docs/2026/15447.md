
# [原创]Ubuntu下简单配置反编译工具以及其开发环境

## 0x0:前言

个人电脑重新换了下系统，Ubuntu14.04,刚好需要配置下android的反编译环境，这里简单记录一下，以供有需要的朋友参考。

其实最简单的，只需要`apktool`就可以了，不过有编码能力的朋友，肯定是希望用自己编写的可视化界面工具来操作了，所以我们就开始配置，并且方便后期继续开发完善工具。

## 0x01:相关软件的下载

Sig包

`4bcK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0A6N6X3g2J5j5X3q4F1K9$3y4G2L8i4m8#2N6r3W2F1k6#2)9J5k6h3y4G2i4K6u0W2N6h3E0Q4x3V1k6K6L8$3k6@1N6$3q4J5k6g2)9J5c8Y4y4A6M7q4)9J5c8X3c8G2N6$3&6D9L8$3q4V1`

Pyqt包

`fd1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4G2N6i4u0U0k6h3k6G2M7X3N6W2i4K6u0W2L8X3g2@1i4K6u0r3M7s2u0G2K9X3g2U0N6s2y4Q4x3V1k6H3P5i4q4@1i4K6u0r3k6X3W2D9k6i4y4Q4x3V1k6b7P5g2q4@1y4q4)9J5c8W2m8&6f1i4c8Q4x3X3b7@1i4K6u0W2x3e0q4Q4x3X3f1J5i4K6u0r3`

Ubuntu自带`Python2.7`，我们也简单进行了更新，下面有步骤可以参考一下，看看是否能满足我们的开发再作决定是否重新安装。

## 0x02:进行软件的安装工作

下载完成后，就是一些初始化的工作了：

首先是：

```bash
apt-get update
```

# 安装 Python 发布版本，dev包必须安装，很多用pip安装包都需要编译

```bash
sudo apt-get install python2.7 python2.7-dev python3.2 python3.2-dev
```

# 很多pip安装的包都需要libssl和libevent编译环境

```bash
sudo apt-get install build-essential libssl-dev libevent-dev libjpeg-dev libxml2-dev libxslt-dev
```

# 安装 pip

```bash
sudo apt-get install python-pip
```

发现一个比较简单的方式：

```bash
sudo apt-get install libxext6 libxext-dev libqt4-dev libqt4-gui libqt4-sql qt4-dev-tools qt4-doc qt4-designer qt4-qtconfig "python-qt4-\*" python-qt4
```

不过我是按照下面的方式来搞得，都是在官网下载的包，最新的 。

操作sig文件：

1, 安装SIP

`4b5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0A6N6X3g2J5j5X3q4F1K9$3y4G2L8i4m8#2N6r3W2F1k6#2)9J5k6h3y4G2i4K6u0W2N6h3E0Q4x3V1k6K6L8$3k6@1N6$3q4J5k6g2)9J5c8Y4y4A6M7q4)9J5c8X3c8G2N6$3&6D9L8$3q4V1` 下载压缩包解压

```bash
sudo python configure.py
make
sudo make install
```

2, 安装依赖的库

```bash
sudo apt-get install libxext6 libxext-dev libqt4-dev libqt4-gui libqt4-sql qt4-dev-tools qt4-doc qt4-designer qt4-qtconfig
```

注：在这个版本的软件包中，`qt4-dev-tools` 包含了Qt Assistant及Qt Linguist等工具，因此不需要单独安装这两个工具。其它的，`qt4-doc` 是帮助文档，包含了Qt中各个类库的详细说明以及丰富的例子程序，可以使用Qt Assistant 工具来打开阅读。`qt4-qtconfig` 是配置Qt环境的一个对话框，一般默认就行了，很少有必要去更改。`qt4-demos` 包含很多可以运行起来的可执行文件以及源代码。`qt4-designer`是用来设计GUI界面的设计器。

3, 安装PyQt4

从 `da1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0A6N6X3g2J5j5X3q4F1K9$3y4G2L8i4m8#2N6r3W2F1k6#2)9J5k6h3y4G2i4K6u0W2N6h3E0Q4x3V1k6K6L8$3k6@1N6$3q4J5k6g2)9J5c8Y4m8&6M7i4c8Q4x3V1k6V1L8%4N6F1L8r3!0S2k6l9`.`. 下载压缩包解压

```bash
sudo python configure.py
sudo make
sudo make install
```

## 0x03:进行测试

好了，这样配置完成后，我们就打开上一篇文章提到的`APK_IDE.py`脚本，在命令行下打开即可

```bash
python APK_IDE.py
```

记得要把`apktool.jar`也放到对应的目录

上篇文章地址：

`http://bbs.pediy.com/showthread.php?t=192045`

下面是测试的截图：

![](https://bbs.kanxue.com/upload/attach/201409/357578_82to8e9m6ws32s4.png)

同时，记得配置一下Java的相关环境变量，需要注意的是，我这里简单修改了上篇文章的代码，使用的是`java`的绝对路径。

最近在写那个查询搜索的功能，同时搜索两个以上字符串所在文件，可以等后续慢慢往这个里面加入。

有了反编译，搜索，那么我们就可以很方便来做一个一体化的工具了，集成反编译，搜索，修改，回编译，签名。。。

再慢慢补充上一些新功能。。。

有兴趣一起搞起的朋友，可以发我邮件交流，论坛有直接发邮件的功能，弄一个开源的`apk`修改工具，然后开源，方便大家跨平台使用

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [ff.png](javascript:void(0);)

  （231.02kb，8次下载）

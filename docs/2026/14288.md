
# [原创]ubantu 16.4 编译android 4.4.4

这里不得不吐槽一下，看雪的帖子排版实在太闹眼睛了

这是我有道云笔记的分享b8aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3&6G2N6r3g2Q4x3X3g2&6L8%4g2V1j5h3!0Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9r3q4J5k6g2)9J5c8W2)9K6c8X3W2V1i4K6y4p5x3o6R3$3j5e0c8S2j5K6S2S2j5K6g2S2z5o6p5%4j5U0c8V1x3e0f1@1k6U0t1%4x3X3j5#2y4$3x3&6y4h3y4Q4x3U0k6S2L8i4m8Q4x3@1u0@1P5i4m8W2i4K6y4p5L8X3!0@1k6g2)9J5x3#2)9J5c8R3`.`.

排版会好一些

-------------------------------------------------------------------------------------------------

## 0.准备100G的磁盘空间

## 1.安装git

```
sudo apt-get install git 

git config –global user.email “test@test.com” 

git config –global user.name “test”
```

## 2.修改hosts，我的墙很强

下载hosts

[d87K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9j5h3!0V1i4K6u0W2j5$3&6Q4x3V1k6Z5L8%4y4@1M7#2)9J5c8U0t1H3x3e0N6Q4x3X3c8Y4L8$3!0Y4L8r3g2Q4x3X3c8Z5L8%4y4@1M7#2)9J5k6h3S2@1L8h3H3`.](https://bbs.kanxue.com/elink@cf9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9j5h3!0V1i4K6u0W2j5$3&6Q4x3V1k6Z5L8%4y4@1M7#2)9J5c8U0t1H3x3e0N6Q4x3X3c8Y4L8$3!0Y4L8r3g2Q4x3X3c8Z5L8%4y4@1M7#2)9J5k6h3S2@1L8h3H3`.)

备份hosts

```
sudo cp /etc/hosts /etc/hosts.bk
```

修改hosts

```
sudo cp hosts /etc/hosts
```

注意：`127.0.0.1`这一行，要和原始的`hosts`文件一致

## 3.安装jdk1.6

下载最新版jdk1.6 --> `jdk-6u45-linux-x64.bin`

[da8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3!0J5j5h3y4D9k6g2)9J5k6h3y4G2L8g2)9J5c8Y4c8W2j5$3S2F1k6i4c8%4L8%4u0C8i4K6u0r3K9X3q4$3j5g2)9J5c8X3A6S2N6X3q4K6k6h3u0#2M7$3W2F1k6i4y4K6i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8K6i4K6u0r3K9X3q4$3j5g2)9J5k6r3q4J5j5$3S2A6N6X3g2Q4x3X3c8V1L8%4N6F1L8r3!0S2k6s2y4Q4x3X3c8B7j5i4k6S2M7$3f1$3i4K6u0V1y4o6p5&6y4o6l9&6i4K6u0W2K9s2c8E0L8l9`.`.](https://bbs.kanxue.com/elink@beeK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3!0J5j5h3y4D9k6g2)9J5k6h3y4G2L8g2)9J5c8Y4c8W2j5$3S2F1k6i4c8%4L8%4u0C8i4K6u0r3K9X3q4$3j5g2)9J5c8X3A6S2N6X3q4K6k6h3u0#2M7$3W2F1k6i4y4K6i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8K6i4K6u0r3K9X3q4$3j5g2)9J5k6r3q4J5j5$3S2A6N6X3g2Q4x3X3c8V1L8%4N6F1L8r3!0S2k6s2y4Q4x3X3c8B7j5i4k6S2M7$3f1$3i4K6u0V1y4o6p5&6y4o6l9&6i4K6u0W2K9s2c8E0L8l9`.`.)

```
sudo cp jdk-6u45-linux-x64.bin /usr/local  

cd  /usr/local  

sudo chmod 777 jdk-6u45-linux-x64.bin

sudo jdk-6u45-linux-x64.bin
```

得到jdk安装路径`/usr/local/jdk1.6.0_45`

```
sudo gedit /etc/profile
```

把下面代码依据实际情况，加到`/etc/profile`文件末尾

```
export JAVA_HOME=/usr/local/jdk1.6.0_45

export JRE_HOME=/usr/local/jdk1.6.0_45/jre

export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH

export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$JAVA_HOME:$PATH
```

安装galternatives

```
sudo apt-get install galternatives
```

打开galternatives

```
sudo galternatives
```

![](https://bbs.kanxue.com/upload/attach/201708/292006_0swg6lemwswjbxg.jpg)

在 Alternatives 中 找到[Java](https://bbs.kanxue.com/elink@e74K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4A6j5W2)9J5k6h3y4K6k6r3&6Q4x3X3g2F1k6i4c8Q4x3V1k6T1j5i4y4W2i4K6u0r3x3e0M7`.) ，点击 添加，

priority设置为300，options设置为`/usr/local/jdk1.6.0_45/bin/java`，

choice选中

在 Alternatives 中 找到`javac` ，点击 添加，

priority设置为300，options设置为`/usr/local/jdk1.6.0_45/bin/javac`，

choice选中

用`java -version`测试，1.6 ok

## 4.下载依赖程序

```
sudo apt-get install -y git flex bison gperf build-essential libncurses5-dev:i386

sudo apt-get install libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-dev g++-multilib

sudo apt-get install tofrodos python-markdown libxml2-utils xsltproc zlib1g-dev:i386

sudo apt-get install dpkg-dev libsdl1.2-dev libesd0-dev

sudo apt-get install git-core gnupg flex bison gperf build-essential

sudo apt-get install zip curl zlib1g-dev gcc-multilib g++-multilib

sudo apt-get install libc6-dev-i386

sudo apt-get install lib32ncurses5-dev x11proto-core-dev libx11-dev

sudo apt-get install lib32z-dev ccache

sudo apt-get install libgl1-mesa-dev libxml2-utils xsltproc unzip m4
```

## 5.设置环境变量并生成repo

```
mkdir ~/bin

PATH=~/bin:$PATH

curl 372K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6r3!0J5j5h3N6W2i4K6u0W2k6$3!0G2k6$3I4W2j5i4m8A6M7#2)9J5k6h3y4G2L8g2)9J5c8X3N6A6N6q4)9J5k6s2u0W2M7r3!0Q4x3X3c8V1L8%4N6F1L8r3!0S2k6s2y4Q4x3V1k6J5k6i4m8G2i4K6t1$3L8X3u0K6M7q4)9K6b7W2)9J5y4X3N6@1i4K6y4n7i4K6t1$3L8X3u0K6M7q4)9K6b7W2)9%4c8g2)9J5c8X3u0A6L8W2)9J5c8Y4u0W2M7r3)9`.

chmod a+x ~/bin/repo
```

## 6.创建资源目录

```
mkdir source

cd source
```

## 7.让 /etc/profile中的环境变量生效

```
source /etc/profile
```

## 8.查看android分支列表，选择要下载的分支

[e8bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7$3!0#2M7X3y4W2i4K6u0r3j5Y4g2A6L8r3c8Q4x3X3c8F1N6h3#2T1k6i4u0K6i4K6u0W2K9s2c8E0L8q4)9J5x3%4y4G2N6i4u0U0k6g2)9J5k6r3y4G2k6r3g2Q4x3X3c8@1j5h3N6K6i4K6u0V1j5h3&6V1i4K6u0V1j5Y4g2A6L8r3c8K6](https://bbs.kanxue.com/elink@f49K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7$3!0#2M7X3y4W2i4K6u0r3j5Y4g2A6L8r3c8Q4x3X3c8F1N6h3#2T1k6i4u0K6i4K6u0W2K9s2c8E0L8q4)9J5x3%4y4G2N6i4u0U0k6g2)9J5k6r3y4G2k6r3g2Q4x3X3c8@1j5h3N6K6i4K6u0V1j5h3&6V1i4K6u0V1j5Y4g2A6L8r3c8K6)

`android-4.4.4_r1`

## 9.选择分支 android-4.4.4_r1

```
repo init -u 815K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8%4y4H3i4K6u0W2N6s2g2F1j5g2)9J5k6i4c8K6K9h3&6Y4K9s2g2S2i4K6u0W2k6h3c8#2i4K6u0W2j5$3&6Q4x3V1k6H3L8r3q4@1k6X3!0J5L8g2)9J5c8X3#2S2L8X3W2X3k6i4y4@1i4K6t1$3L8X3u0K6M7q4)9K6b7W2)9J5k6r3u0Q4x3U0k6F1j5Y4y4H3i4K6y4n7j5h3&6V1M7X3!0A6k6q4)9J5k6o6c8Q4x3X3f1@1i4K6u0W2y4q4)9#2k6Y4t1I4
```

## 10.同步代码

```
repo sync
```

为了防止下载数据的时候出现意外，我们写一个脚本

`DownloadSource.h`

内容如下

```
#!/bin/bash

#FileName DownloadSource.h

PATH=~/bin:$PATH

# 注意修改成你要编译的版本，这里是 

android-4.4.4_r1

repo init -u e6dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8%4y4H3i4K6u0W2N6s2g2F1j5g2)9J5k6i4c8K6K9h3&6Y4K9s2g2S2i4K6u0W2k6h3c8#2i4K6u0W2j5$3&6Q4x3V1k6H3L8r3q4@1k6X3!0J5L8g2)9J5c8X3#2S2L8X3W2X3k6i4y4@1i4K6t1$3L8X3u0K6M7q4)9K6b7W2)9J5k6r3u0Q4x3U0k6F1j5Y4y4H3i4K6y4n7j5h3&6V1M7X3!0A6k6q4)9J5k6o6c8Q4x3X3f1@1i4K6u0W2y4q4)9#2k6Y4t1I4

repo sync

while [$? = 1]: do

echo "=========download failed,again============"

sleep 5

repo sync

done
```

保存后

```
sh DownloadSource.h
```

## 11.编译android4.4.4需要make 3.81或3.82 ，所以要降级make

下载 `make-3.82.tar.gz`

`ftp://ftp.gnu.org/gnu/make/`

```
tar -zxvf make-3.82.tar.gz

cd make-3.82

sudo ./configure

sudo make

sudo make install
```

执行完后在`/usr/local/bin`中可得到3.8.2 版本的`make`，复制到`/usr/bin`中即可

## 12.开始编译

```
source build/envsetup.sh

lunch
```

选择你想编译的版本，这里我选的是 `aosp_hammerhead-userdebug`,所以输入11，

```
make -j4
```

`-j4`的意思是4个线程同步编译

## 13.过几个小时，编译成功

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

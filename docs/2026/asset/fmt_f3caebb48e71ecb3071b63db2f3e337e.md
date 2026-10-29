
# [原创] Ubuntu14.04编译_android-7.1.2_r18

## 编译环境

OS: 虚拟机Ubuntu14.04 磁盘：512G 内存：8G 然后增加8G虚拟交换内存

虚拟机用的是Paralles

镜像稍等下上传到百度网盘给大家。

## 系统基本依赖安装

编译了：`/android-7.1.2_r18`

```shell
sudo apt-get install openjdk-7-jdk
# 后来发现编译android-7.1.2_r18时需要 OPEN-JDK8，后面又说明如何安装。建议编译自己想要的系统前先查清楚需要的JDK版本
sudo apt-get install git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev libgl1-mesa-dev libxml2-utils xsltproc unzip
```

## 设置支持USB设备

```shell
sudo touch /etc/udev/rules.d/51-android.rules
sudo gedit /etc/udev/rules.d/51-android.rules
# adb protocol on passion (Nexus One)
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4e12", MODE="0600", OWNER="m1k3"
# fastboot protocol on passion (Nexus One)
SUBSYSTEM=="usb", ATTR{idVendor}=="0bb4", ATTR{idProduct}=="0fff", MODE="0600", OWNER="m1k3"
# adb protocol on crespo/crespo4g (Nexus S)
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4e22", MODE="0600", OWNER="m1k3"
# fastboot protocol on crespo/crespo4g (Nexus S)
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4e20", MODE="0600", OWNER="m1k3"
# adb protocol on stingray/wingray (Xoom)
SUBSYSTEM=="usb", ATTR{idVendor}=="22b8", ATTR{idProduct}=="70a9", MODE="0600", OWNER="m1k3"
# fastboot protocol on stingray/wingray (Xoom)
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="708c", MODE="0600", OWNER="m1k3"
# adb protocol on maguro/toro (Galaxy Nexus)
SUBSYSTEM=="usb", ATTR{idVendor}=="04e8", ATTR{idProduct}=="6860", MODE="0600", OWNER="m1k3"
# fastboot protocol on maguro/toro (Galaxy Nexus)
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4e30", MODE="0600", OWNER="m1k3"
# adb protocol on panda (PandaBoard)
SUBSYSTEM=="usb", ATTR{idVendor}=="0451", ATTR{idProduct}=="d101", MODE="0600", OWNER="m1k3"
# fastboot protocol on panda (PandaBoard)
SUBSYSTEM=="usb", ATTR{idVendor}=="0451", ATTR{idProduct}=="d022", MODE="0600", OWNER="m1k3"
# usbboot protocol on panda (PandaBoard)
SUBSYSTEM=="usb", ATTR{idVendor}=="0451", ATTR{idProduct}=="d00f", MODE="0600", OWNER="m1k3"
# usbboot protocol on panda (PandaBoard ES)
SUBSYSTEM=="usb", ATTR{idVendor}=="0451", ATTR{idProduct}=="d010", MODE="0600", OWNER="m1k3"
```

`OWNER="<USERNAME>"` 注意将此处用户名替换为自己系统的

如果插上手机，Ubuntu还识别不出手机的话执行下面操作：

```shell
mike@ubuntu:~$ lsusb
Bus 001 Device 006: ID 203a:fffa
Bus 001 Device 005: ID 203a:fffa
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub # 就把这个当做为手机吧
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 002 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
mike@ubuntu:~$ sudo gedit /etc/udev/rules.d/51-android.rules
# 在文件末尾添加如下内容
SUBSYSTEM=="usb", ATTR{idVendor}=="1d6b", ATTR{idProduct}=="0002", MODE="0600", OWNER="m1k3"
```

## 源码下载准备

```shell
# 用户的家目录下
mkdir ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
cd ~
mkdir -p AOSP/android6.0
# 配置GitHub账号
mike@ubuntu:~$ git config --global user.name your_name
mike@ubuntu:~$ git config --global user.email your_email@email.com
# 编辑.bashrc文件添加下面配置
PATH=~/bin:$PATH
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'
```

## 同步代码

同步代码时间会比较长，期间还会有断开下载链接情况发生，需及时恢复下载请求。

下载 <a href="elink@544K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0K9i4u0J5L8%4u0K6i4K6u0W2N6s2g2F1j5g2)9J5k6i4c8K6K9h3&6Y4K9s2g2S2i4K6u0W2k6h3c8#2i4K6u0W2j5$3&6Q4x3V1k6S2L8%4y4H3i4K6u0V1L8h3!0F1N6r3S2D9P5g2)9J5c8X3q4G2M7%4m8Q4x3X3c8D9j5i4c8W2M7%4c8Q4x3X3g2@1j5i4t1`."><mark class="encrypted">cebK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0K9i4u0J5L8%4u0K6i4K6u0W2N6s2g2F1j5g2)9J5k6i4c8K6K9h3&6Y4K9s2g2S2i4K6u0W2k6h3c8#2i4K6u0W2j5$3&6Q4x3V1k6S2L8%4y4H3i4K6u0V1L8h3!0F1N6r3S2D9P5g2)9J5c8X3q4G2M7%4m8Q4x3X3c8D9j5i4c8W2M7%4c8Q4x3X3g2@1j5i4t1`.</mark></a>，下载完成后记得根据 `checksum.txt` 的内容校验一下。

由于所有代码都是从隐藏的 `.repo` 目录中 checkout 出来的，所以我们只保留了 `.repo` 目录，下载后解压 再 `repo sync` 一遍即可得到完整的目录。

使用方法如下:

**TIPS:** 建议同步代码时最好选择只支持目标系统的分支。如

![image.png](upload/attach/202002/667895_4ZM4369ZDHUW869.jpg)

```shell
# 初始化代码和同步代码
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-4.0.1_r1
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-6.0.1_r45
repo sync
# 另一种方式
# 建议使用清华源，先下载https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar 后在同步
wget -c https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar # 下载初始化包
tar xf aosp-latest.tar
cd AOSP # 解压得到的 AOSP 工程目录
# 这时 ls 的话什么也看不到，因为只有一个隐藏的 .repo 目录
# repo sync前先执行下自己想下载的系统源码命令
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-4.0.1_r1
# 或者
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-7.1.2_r18
# 在进行同步
repo sync # 正常同步一遍即可得到完整目录
# 或 repo sync -l 仅checkout代码
```

## 下载驱动

同步时记得记录相应的分支代号，在同步完源码后好方便下载驱动。

例如我的版本代号是： N2G47F

![image.png](upload/attach/202002/667895_2CR2TRRKZV423FU.jpg)

在驱动页面搜索这个代号即可：

![image.png](upload/attach/202002/667895_V8GGMBHCMKB4MUZ.jpg)

## 开始编译

开始编译前要做些准备工作

### 设置虚拟内存

```shell
sudo dd if=/dev/zero of=/swapfile count=8192 bs=1M
sudo mkswap /swapfile
sudo swapon /swapfile
```

### 设置Java内存

```shell
# prebuilts/sdk/tools/jack-admin
# 这是一个管理Jack的shell脚本，找到start-server函数，直接修改其启动参数，由原来的
JACK_SERVER_COMMAND="java -XX:MaxJavaStackTraceDepth=-1 -Djava.io.tmpdir=$TMPDIR $JACK_SERVER_VM_ARGUMENTS -cp $LAUNCHER_JAR $LAUNCHER_NAME"
更改为
JACK_SERVER_COMMAND="java -XX:MaxJavaStackTraceDepth=-1 -Djava.io.tmpdir=$TMPDIR $JACK_SERVER_VM_ARGUMENTS -Xmx4096m -cp $LAUNCHER_JAR $LAUNCHER_NAME"
# 此时Jack服务器仍然在后台执行，所以我们需要将其停止，然后重启启动(make会自动启动Jack服务器)才能使得修改后的参数生效。
# 我们执行下面的命令此时Jack服务器仍然在后台执行，所以我们需要将其停止，然后重启启动(make会自动启动Jack服务器)才能使得修改后的参数生效。
# 我们执行下面的命令：
# 先停止服务
./prebuilts/sdk/tools/jack-admin stop-server
# 重新开始服务
./prebuilts/sdk/tools/jack-admin start-server
```

### 配置JDK

```shell
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-get update
sudo apt-get install openjdk-8-jdk
sudo update-alternatives --config java
sudo update-alternatives --config javac
sudo update-alternatives --config javadoc
```

### 进行编译

```shell
mike@ubuntu:~/aosp$ source build/envsetup.sh
mike@ubuntu:~/aosp$ lunch # 选择要编译的目标版本
mike@ubuntu:~/aosp$ make -j8
```

## AOSP切换分支

如果需要编译其他分支执行下面的操作后，再按照原来编译的步骤即可

```shell
cd $ANDROID_ROOT # 放.repo的目录
cd .repo/manifests
git branch -av # see all available branches on origin 建议导出到文件中看清楚了在选择
Select a version and
# remotes/origin/android-7.1.2_r18 d456b66 Manifest for Android 7.1.2 Release 18
cd $ANDROID_ROOT
repo init -b <my_selected_android_version>
# repo init -b android-7.1.2_r18
Such selective repo init with -b (without -u) will only update manifest branch and will not otherwise touch your tree.
Now, simply sync it:
repo sync -j8
and some time later, your Android tree will switch to another version.
```

## 刷机

```shell
//进入手机bootloader
adb reboot bootloader
//开始刷机
fastboot flash boot boot.img
fastboot flash system system.img
fastboot flash cache cache.img
fastboot flash userdata userdata.img
fastboot flash recovery recovery.img
```

* * *

## 参考

> 安卓编译环境配置

>

> <a href="elink@5e6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7$3!0#2M7X3y4W2i4K6u0r3K9h3&6A6N6r3W2S2L8r3W2*7K9h3&6Y4i4K6u0W2K9s2c8E0L8q4)9J5x3$3&6W2P5s2c8Q4x3X3c8V1L8%4N6F1L8r3!0S2k6q4)9J5k6s2c8Z5k6g2)9J5k6s2y4G2N6i4u0U0k6b7`.`."><mark class="encrypted">373K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7$3!0#2M7X3y4W2i4K6u0r3K9h3&6A6N6r3W2S2L8r3W2*7K9h3&6Y4i4K6u0W2K9s2c8E0L8q4)9J5x3$3&6W2P5s2c8Q4x3X3c8V1L8%4N6F1L8r3!0S2k6q4)9J5k6s2c8Z5k6g2)9J5k6s2y4G2N6i4u0U0k6b7`.`.</mark></a>

>

> 安卓系统相应硬件驱动

>

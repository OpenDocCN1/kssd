
# LineageOS20+Pixel6 下的Fart8.0脱壳机迁移和编译流程

## 一、LineageOS20源码下载和编译

### 1. 选择LineageOS20系统的原因

之前将Fart8.0的代码编译到Android13下，运行发现在将Java的`Method`对象转换成C的`ArtMethod`对象时报错，没找到原因。再加上AOSP的系统太素了，就琢磨是否可以用LineageOS来编译，所以就有了这篇文章。

先罗列一下我的环境吧，编译环境是Kali2022,用的r0ysue大佬的集成版，手机是Pixel6，选择的系统是LineageOS20版(基于AOSP13)的，原本想用基于AOSP10的Lineage17的，但是官网同步源码后发现缺少device blob,遂决定就用20版。

### 2. LineageOS20源码下载和编译

#### 2.1 虚拟机设置

我的虚拟机设置了32G内存，8核CPU，500G的硬盘，实际下载编译完成占用了快300G空间。这里坑的地方在于同样的配置，我的AOSP能顺利编译完成，但在编译LineageOS时中途报错，检查原因发现是因为swap只有1G大小，在设置了swap为20G后，才编译成功。

```shell
swapoff -a
dd if=/dev/zero of=/var/swapfile bs=1M count=20480
mkswap /var/swapfile
swapon /var/swapfile
free -m --查看当前分区
```

#### 2.2 编译环境配置

- 安装编译必须的组件

```shell
apt-get update
apt-get install bc bison build-essential ccache curl flex g++-multilib gcc-multilib git git-lfs gnupg gperf imagemagick
lib32ncurses5-dev lib32readline-dev lib32z1-dev libelf-dev liblz4-tool libncurses5 libncurses5-dev
libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev m4
```

- 安装好git和 platform-tools 并配置环境变量

- 创建源码所需目录（默认在kali root账户下）

```shell
mkdir -p ~/bin
mkdir -p ~/android/lineage
```

- 安装repo命令行工具

```shell
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

- 将bin目录添加进环境变量。

使用下面的命令打开`~/.bashrc`文件，

```shell
vim ~/.bashrc
```

在文件最后添加下面的代码:

```shell
# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/bin" ] ; then
PATH="$HOME/bin:$PATH"
fi
```

- 配置git

```shell
git config --global user.email "you@example.com"#替换成你的邮件 随便写
git config --global user.name "Your Name" #替换成你的用户名 随便写
```

- 查看git lfs是否安装

```shell
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
apt-get install git-lfs
git lfs install #查看是否安装成功。
```

- 配置缓存，加快编译速度

```shell
export USE_CCACHE=1
export CCACHE_EXEC=/usr/bin/ccache
```

可将其加入到`~/.bashrc`文件中，并在命令行执行下行

```shell
ccache -M 50G
```

#### 2.3 下载LineageOS源码

- 初始化LineageOS source repository

```shell
cd ~/android/lineage
repo init -u https://github.com/LineageOS/android.git -b lineage-20.0 --git-lfs
```

这里最好不要换成国内源，因为国内源同步出来的代码，我在编译时碰到浏览器app编译失败的问题，重新从官方源同步才解决，查看清华的issue发现有同样的问题，坑死了。

- 下载源码

```shell
repo sync -j8
```

上面的-j8是启动8个线程下载,可以不要，默认是4个线程。取决于你的虚拟机cpu内核数。由于网络问题，如果同步完成后报错，可根据提示，将线程数减为1，重新执行该命令即可。

- 下载设备内核代码

确保你在源码根目录下（`cd ~/android/lineage`）

```shell
source build/envsetup.sh
breakfast oriole #这里也可使用lunch 选择官方提供的机型 如果没有的你要的机型需要到git上搜索相应机型的kernel
```

下载源码整个过程根据网速会有1段时间，我自己搭建了科学上网的软路由，整个时间有3个小时左右。

下载完kernel源码后会报如下的错误，这是因为还没有下载device blob的缘故，暂时不用管，下载完device blob后再重新执行上面的命令即可

![image-20230905183014828](./upload/attach/202309/975093_G87MMT6NUUC8PGC.png)

#### 2.4 提取专有device blob

这里是个大坑。我按照官方推荐的方式直接从已经安装了LineageOS的piexl6手机上提取blob,结果有几千个文件没有找到，害的我折腾了2天。总算使用官方镜像提取成功。下面是过程。

- 创建提取目录

```shell
mkdir ~/android/system_dump/
cd ~/android/system_dump/
```

- 下载官方镜像 <a href="elink@58eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5k6h3I4A6L8X3g2S2k6$3g2G2M7#2)9J5k6h3!0J5k6#2)9J5c8X3c8W2N6X3W2U0k6i4y4Q4x3V1k6G2M7X3W2G2L8r3g2Q4x3V1k6T1N6h3W2D9k6s2x3`.">LineageOS Downloads</a>

![image-20230905185252612](./upload/attach/202309/975093_TEJ3WAR953M6URH.png)

- 从zip文件中解压出`payload.bin`文件

```shell
unzip /path/to/lineage-\*.zip payload.bin
```

- 安装 python-protobuf

```shell
apt-get install python-protobuf
```

- 下载官方提供的导出脚本，然后用python运行脚本，注意脚本目录路径

```shell
git clone https://github.com/LineageOS/scripts
python scripts/update-payload-extractor/extract.py payload.bin --output_dir ./
```

成功运行后，应该能在`system_dump`目录下看到如下一些`.img`文件

![image-20230905190110559](./upload/attach/202309/975093_7QG9VSYMZHCHZPU.png)

然后使用`mout`命令挂载这些`.img`文件

```shell
mkdir system/
sudo mount -o ro system.img system/
sudo mount -o ro vendor.img system/vendor/
sudo mount -o ro product.img system/product/
sudo mount -o ro system_ext.img system/system_ext/
```

进入`~/android/lineage/device/google/oriole`文件夹 运行如下命令 导出blob文件

```shell
cd ~/android/lineage/device/google/oriole
./extract-files.sh ~/android/system_dump/
```

- 重新执行如下命令

```shell
source build/envsetup.sh
breakfast oriole
```

#### 2.5 编译源码

```shell
croot
brunch oriole
```

编译时间在2小时左右，尤其最后阶段编译kernel会持续50多分钟，这里即使我的内存设置到了32G，也不够用，需要设置swap,而且每次修改源码再编译 都会要重新编译这个kernel，很耗费时间，暂时没找到不重复编译的办法，在XDA上看到可以下载prebuild的内核，避免重复编译，实在懒折腾了，所以没尝试。

编译成功后会在`/root/android/lineage/out/target/product/oriole`目录下生成`boot.img` 和 `lineage-20.0*****.zip`文件。按照官方文档替换官方镜像刷机即可。

![image-20230905192434053](./upload/attach/202309/975093_4U9NJCCY89NUMGP.png)

#### 2.6 刷机过程

按照官方文档操作即可。实在懒翻译了

<a href="elink@eb5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4K9h3E0A6i4K6u0W2L8r3W2F1k6h3q4Y4k6h3!0K6i4K6u0W2L8%4u0Y4i4K6u0r3k6r3g2$3K9h3y4W2M7#2)9J5c8X3!0J5K9h3!0D9k6g2)9J5c8X3W2F1M7%4c8S2L8r3H3`.">Install LineageOS on oriole | LineageOS Wiki</a>

#### 2.7 源码导入AndroidStudio和CLion

为了方便修改源码，减少报错，可以将源码中的Java代码导入AndroidStudio，C语言代码导入CLionAS导入Android源码

##### Java代码导入AndroidStudio

1. 先成功编译一次，再使用以下方法导入

2. 在ubuntu系统下，进入源码根目录，运行如下命令, 会在源码目录下的`out/host/linux-x86/framework`目录下生成了`idegen.jar`文件

```shell
source build/envsetup.sh
mmm development/tools/idegen/
```

3. 在源码根目录下继续执行如下命令，会在根目录下生成`android.iml`和`android.ipr`两个文件，这两个文件是AndroidStudio的工程配置文件

```shell
development/tools/idegen/idegen.sh
```

4. 安装并打开AndroidStudio，选择Open an existing Android Studio project，找到源码根目录，点击`Android.ipr`

4. 具体参考：<mark class="encrypted">08aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6i4S2A6j5h3!0D9L8$3&6Y4i4K6u0W2L8h3g2Q4x3V1j5J5x3o6p5^5i4K6u0r3x3o6S2Q4x3V1j5I4y4g2)9J5c8V1q4a6f1#2l9K6i4K6u0r3</mark>

我在AOSP中成功导入了AS,但在LineageOS源码中在执行到第3步时长时间卡住，等了30个小时不动，没辙只好放弃。改用VSCode编辑java代码。

##### C/C++代码导入CLion

1. 生成用于将源码导入Clion的`CMakeLists.txt`

```shell
source build/envsetup.sh
#打开开关，编译时生成CMakeLists.txt
export SOONG_GEN_CMAKEFILES=1
export SOONG_GEN_CMAKEFILES_DEBUG=1
breakfast oriole
```

`CMakeLists.txt`会生成在`out/development/ide/clion/art/runtime/libart-arm64-android/CMakeLists.txt`

2. 用Clion打开`CMakeLists.txt`

3. tools--->cmake-->Change Project Root

选择aosp源码根路径，等解析完毕即可

## 二、Fart8.0源码迁移

### 2.1 修改Fart脱壳机的Java层代码

Java代码可按上面导入的步骤，用AS编辑

修改LineageOS源码的`/root/android/lineage/frameworks/base/core/java/android/app/ActivityThread.java`文件，将Fart8.0对应文件中添加的部分复制到该文件中，并导入对应的包，为了防止加固厂商的检测，修改了函数名和日志输出。

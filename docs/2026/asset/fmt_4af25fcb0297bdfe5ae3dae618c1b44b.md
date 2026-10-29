
# Android - 源码下载编译

## 目录

- Android - 源码下载编译
- [常用链接](#常用链接)
- 配置 `git`
- [系统源码下载](#系统源码下载)
- [源码下载后的初始化操作](#源码下载后的初始化操作)
- 驱动下载 模拟器用不上
- [源码下载脚本不死脚本](#源码下载脚本不死脚本)
- [编译环境软件安装](#编译环境软件安装)
- 编译docker Android7build镜像
- 18.04 docker
- [Dockerfile](#dockerfile)
- build Docker Image
- RUN Docker
- [官方docker编译环境](#官方docker编译环境)
- [官网](#官网)
- [Dockerfile](#dockerfile)
- [编译image](#编译image)
- [使用](#使用)
- [编译](#编译)
- [启动模拟器](#启动模拟器)
- [真机刷机](#真机刷机)
- [其他注意事项](#其他注意事项)

## 常用链接

- [Google原始刷机镜像](<https://bbs.kanxue.com/elink@4acK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0K6i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5c8X3W2E0j5h3N6W2M7#2)9J5x3$3y4J5L8%4y4K6K9r3q4@1j5$3R3.>)
- [不同Android版本对应标记](<https://bbs.kanxue.com/elink@878K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3k6r3!0U0M7#2)9J5c8Y4y4W2N6s2g2H3i4K6u0r3M7%4c8S2M7Y4c8Q4x3V1k6T1N6h3W2D9k6q4)9J5k6r3&6#2L8h3u0W2M7Y4x3>)
- [源代码标记和 build](<https://bbs.kanxue.com/elink@266K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3k6r3!0U0M7#2)9J5c8Y4y4W2N6s2g2H3i4K6u0r3M7%4c8S2M7Y4c8Q4x3V1k6T1N6h3W2D9k6q4)9J5k6r3&6#2L8h3u0W2M7Y4y4Q4x3U0y4K6L8%4g2J5j5$3g2Q4x3X3c8U0L8$3c8W2i4K6u0V1N6r3q4Y4M7#2)9J5k6r3q4F1k6q4)9J5k6r3u0#2K9h3I4V1M7H3>)
- [Google 的驱动程序](<https://bbs.kanxue.com/elink@3edK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0K6i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5c8X3c8J5K9i4k6W2M7Y4x3>)
- [刷写设备](<https://bbs.kanxue.com/elink@63fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3k6r3!0U0M7#2)9J5c8Y4y4W2N6s2g2H3i4K6u0r3j5Y4g2A6L8r3c8Q4x3V1k6J5N6h3&6F1K9h3&6Y4i4K6y4r3K9r3I4Q4x3@1c8*7K9q4)9J5k6r3y4F1>)

## 配置 `git`

```bash
git config --global user.email "youremil.com"
git config --global user.name "yourname"
```

## 系统源码下载

```bash
# 目前下载清华的AOSP代码总出现问题，有一个可行的方法就是下载最新的月更新tar包：这个不省心 不推荐
# 请参考：a65K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0K9i4u0J5L8%4u0K6i4K6u0W2N6s2g2F1j5g2)9J5k6i4c8K6K9h3&6Y4K9s2g2S2i4K6u0W2k6h3c8#2i4K6u0W2j5$3&6Q4x3V1k6Z5k6h3I4H3i4K6u0r3b7f1!0e0f1q4)9J5c8R3.
wget https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar
tar xf aosp-latest.tar

# 第二种方法
cd /home/yourname/.bin
curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o repo
chmod +x repo

# 使用tuna的镜像源进行更新，可以将如下内容复制到你的~/.bashrc里
export REPO_URL='47eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0K9i4u0J5L8%4u0K6i4K6u0W2N6s2g2F1j5g2)9J5k6i4c8K6K9h3&6Y4K9s2g2S2i4K6u0W2k6h3c8#2i4K6u0W2j5$3&6Q4x3V1k6Y4K9i4c8Q4x3V1k6Y4K9i4c8Q4x3X3c8J5k6i4m8G2i4K6t1%4'

# 打开~/.bashrc，添加行：
export PATH=/home/yourname/.bin:$PATH
source .bashrc

repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-11.0.0_r46
repo sync

# 完成代码就同步下来了

# --depth=1 拉取代码的深度 不推荐 可能会出现网络问题导致代码异常
# 同步失败时 可尝试切换镜像源
# 1. 修改.repo/manifests.git/config
# 2. 修改 url 为（中科大镜像）git://mirrors.ustc.edu.cn/aosp/platform/manifest

# 清华源的打包方式是没法在一开始就指定分支的，所以需要切换：
# 在.repo/manifests文件夹下切分支，在该文件下使用
git branch -a
# 即可看到所有的分支信息，使用
git checkout <分支>
# 切换到对应的分支即可，切换完后返回工作目录重新repo sync 一遍。

# 创建次级仓库 下载到所在目录
repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-4.0.1_r1 --mirror

# 以下说明假定镜像创建在 /usr/local/aosp/mirror 中。首先，创建并同步镜像本身。请注意 --mirror 标记，该标记只能在创建新客户端时指定
mkdir -p /usr/local/aosp/mirror
cd /usr/local/aosp/mirror
repo init -u https://android.googlesource.com/mirror/manifest --mirror
repo sync

# 同步镜像后，您就可以从镜像创建新客户端了。请注意，您必须指定一个绝对路径
mkdir -p /usr/local/aosp/master
cd /usr/local/aosp/master
repo init -u /usr/local/aosp/mirror/platform/manifest.git
repo sync
```

### 源码下载后的初始化操作

```bash
# 删除所有 .git仓库 然后自己新建 git init
find . -name .git | xargs rm -fr
```

## 驱动下载 模拟器用不上

| build ID | 标记 | 版本 | 支持的设备 | 安全补丁级别 |
| --- | --- | --- | --- | --- |
| `RQ3A.211001.001` | `android-11.0.0_r46` | Android11 | Pixel 3、Pixel 3 XL、Pixel 3a、Pixel 3a XL、Pixel 4、Pixel 4 XL、Pixel 4a、Pixel 4a (5G)、Pixel 5 | 2021-10-01 |

- build ID：根据 build ID 找对应的设备（Google Pixel 3XL）驱动
- 标记：下载源码时的分支名称
- 支持的设备

## 源码下载脚本不死脚本

- 源码出处：<https://bbs.pediy.com/thread-269575.htm>
- `down.sh`

```bash
#!/bin/bash
echo "==========start repo sync==="
repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-6.0.1_r20
repo sync -j4 # -d --force-sync --no-clone-bundle
while [ $? = 1 ]; do
  echo “==========sync failed, re-sync again==========”
  sleep 3
  repo sync -j4 -d --force-sync --no-clone-bundle
done

chmod a+x down.sh
./down.sh
```

## 编译环境软件安装

## 编译

```bash
# 切换环境
source build/envsetup.sh
# 查看构建类型 序号 根据自己的手机型号 会有对应的标志 在常用链接里可以看见
lunch 序号
# 先清理一下工程
make clean
# 先整个编译一遍 可选
m
# 后续编译
make -j64

You're building on Linux

Lunch menu... pick a combo:
     1. aosp_arm-eng
     2. aosp_arm64-eng
     3. aosp_blueline-userdebug
     4. aosp_blueline_car-userdebug
     5. aosp_bonito-userdebug
     6. aosp_bonito_car-userdebug
```

### 启动模拟器

```bash
source build/envsetup.sh
lunch 模拟器序号
# 不行用相对路径
emulator
```

### 真机刷机

[`fastboot` `adb` 工具下载](<elink@8afK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7%4c8#2k6r3W2G2i4K6u0r3M7X3g2D9k6h3q4K6k6i4y4Q4x3V1k6H3L8r3q4@1k6X3!0J5L8g2)9J5k6s2c8G2L8$3I4K6i4K6y4r3K9r3I4Q4x3@1c8*7K9q4)9J5k6r3y4F1>)

```bash
# 需先 source lunch一下 否则找不到命令
# 先设置image 所在路径 直接刷会有提示
fastboot flashall -w
```

## 其他注意事项

```bash
################1################
# 编译Android7 报错：
# FAILED: /bin/bash -c “prebuilts/misc/linux-x86/flex/flex-2.5.39 -oout/host/linux-x86/obj/STATIC_LIBRARIES/libaidl-common_intermediates/aidl_language_l.cpp system/tools/aidl/aidl_language_l.ll”
# 解决方法： 在android目录下输入命令：
export LC_ALL=C
# 再重新make
# 用 export LC_ALL=C 来去除所有本地化的设置，让命令能正确执行
################2#################
# 下载源码报错
# curl 92 HTTP/2 stream 3 was not closed cleanly before end of the underlying
git config --global http.version HTTP/1.1
git config --global http.postBuffer 524288000
# LibreSSL SSL_connect: SSL_ERROR_SYSCALL 报错
git config --global --unset http.proxy
git config --global --unset https.proxy
git config --global http.sslBackend "openssl"
git config --global http.sslVerify false
# fatal: unable to access 'https://aosp.tuna.tsinghua.edu.cn/platform/manifest/': server certificate verification failed. CAfile: /etc/ssl/certs/ca-certificates.crt CRLfile: none
git config --global http.sslverify false
###################################
/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs
/Library/Developer/CommandLineTools/SDKs
```

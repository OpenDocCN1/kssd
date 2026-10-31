
# [原创]解决 Pixel 及 aosp rom 国内 WiFi 网络连接受限

## 前言

在用Pixel、及刷的像素rom，的时候老是提示网络连接受限，虽说能用，但是就恶心！

## 原因

自 Android 5.0 起，谷歌引入了「Captive Portal」的机制，用于检测 WiFi网络认证是否正常。

该机制默认检测访问的是谷歌服务器，需要科学才能正常访问谷歌服务器，如若没有 WiFi就会出现网络受限的情况，WiFi图标上会出现一个感叹号标志，导致系统误以为你的网络不好，从而使网络不稳定。

解决方法总是有的：

> 一、科学，dddd；
> 二、谷歌提供了参数变量设置，国内安卓手机系统都可以将其修改成自家的服务器地址或者高通中国的地址。

## 参数设置

### 方法一：已ROOT手机

已经 `ROOT` 的安卓机很简单，可以用面具模块、Scene、`Captivemgr`等工具来修改，这里推荐使用 `Captivemgr` 因为可以自定义服务器地址。

默认提供了三个服务器，分别是小米、高通、V2EX，根据延迟选择就行了，然后应用。手机切换飞行模式，再切换回来就OK了。

### 方法二：没有ROOT情况 (adb命令要自行安装与配置)

没有 `ROOT` 的安卓机可以借助 `ADB` 命令来修改，首先下载ADB工具包，然后手机开启USB调试模式，接着运行 `CMD` 输入下面的命令就可以了。

#### 步骤一：删除默认的地址

```
adb shell settings delete global captive_portal_https_url
adb shell settings delete global captive_portal_http_url
```

#### 步骤二：修改新的地址

```
adb shell settings put global captive_portal_http_url http://connect.rom.miui.com/generate_204
adb shell settings put global captive_portal_https_url https://connect.rom.miui.com/generate_204
```

改完同样把手机切换飞行模式，再切换回来就可以了。如果需要其它服务器地址，自行修改，如高通的地址是`220K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0j5i4m8@1K9i4k6W2i4K6u0W2N6U0u0W2P5q4)9J5k6h3y4G2i4K6u0r3k6$3g2F1k6i4u0S2N6r3g2Q4y4h3j5J5x3o6b7`.

## 前言

在用Pixel、及刷的像素rom，的时候老是提示网络连接受限，虽说能用，但是就恶心！

## 原因

自 Android 5.0 起，谷歌引入了「Captive Portal」的机制，用于检测 WiFi网络认证是否正常。

该机制默认检测访问的是谷歌服务器，需要科学才能正常访问谷歌服务器，如若没有 WiFi就会出现网络受限的情况，WiFi图标上会出现一个感叹号标志，导致系统误以为你的网络不好，从而使网络不稳定。

解决方法总是有的：

> 一、科学，dddd；
> 二、谷歌提供了参数变量设置，国内安卓手机系统都可以将其修改成自家的服务器地址或者高通中国的地址。

## 参数设置

### 方法一：已ROOT手机

已经 `ROOT` 的安卓机很简单，可以用面具模块、Scene、`Captivemgr`等工具来修改，这里推荐使用 `Captivemgr` 因为可以自定义服务器地址。

默认提供了三个服务器，分别是小米、高通、V2EX，根据延迟选择就行了，然后应用。手机切换飞行模式，再切换回来就OK了。

### 方法二：没有ROOT情况 (adb命令要自行安装与配置)

没有 `ROOT` 的安卓机可以借助 `ADB` 命令来修改，首先下载ADB工具包，然后手机开启USB调试模式，接着运行 `CMD` 输入下面的命令就可以了。

#### 步骤一：删除默认的地址

```
adb shell settings delete global captive_portal_https_url
adb shell settings delete global captive_portal_http_url
```

#### 步骤二：修改新的地址

```
adb shell settings put global captive_portal_http_url http://connect.rom.miui.com/generate_204
adb shell settings put global captive_portal_https_url https://connect.rom.miui.com/generate_204
```

改完同样把手机切换飞行模式，再切换回来就可以了。如果需要其它服务器地址，自行修改，如高通的地址是`a2aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0j5i4m8@1K9i4k6W2i4K6u0W2N6U0u0W2P5q4)9J5k6h3y4G2i4K6u0r3k6$3g2F1k6i4u0S2N6r3g2Q4y4h3j5J5x3o6b7`.

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#系统相关](https://bbs.kanxue.com/forum-161-1-126.htm)

[#工具脚本](https://bbs.kanxue.com/forum-161-1-128.htm)

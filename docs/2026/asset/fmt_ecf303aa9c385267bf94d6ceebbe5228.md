
# prebuilts/sdk/tools/jack-admin

`prebuilts/sdk/tools/jack-admin` 是一个管理 Jack 的 shell 脚本，找到 `start-server` 函数，直接修改其启动参数，由原来的

```shell
JACK_SERVER_COMMAND="java -XX:MaxJavaStackTraceDepth=-1 -Djava.io.tmpdir=$TMPDIR $JACK_SERVER_VM_ARGUMENTS -cp $LAUNCHER_JAR $LAUNCHER_NAME"
```

更改为

```shell
JACK_SERVER_COMMAND="java -XX:MaxJavaStackTraceDepth=-1 -Djava.io.tmpdir=$TMPDIR $JACK_SERVER_VM_ARGUMENTS -Xmx4096m -cp $LAUNCHER_JAR $LAUNCHER_NAME"
```

此时 Jack 服务器仍然在后台执行，所以需要将其停止，然后重新启动（`make` 会自动启动 Jack 服务器）才能使得修改后的参数生效。执行下面的命令：

```shell
# 先停止服务
./prebuilts/sdk/tools/jack-admin stop-server

# 重新开始服务
./prebuilts/sdk/tools/jack-admin start-server
```

## 配置 JDK

```shell
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-get update
sudo apt-get install openjdk-8-jdk
sudo update-alternatives --config java
sudo update-alternatives --config javac
sudo update-alternatives --config javadoc
```

## 进行编译

```shell
mike@ubuntu:~/aosp$ source build/envsetup.sh
mike@ubuntu:~/aosp$ lunch # 选择要编译的目标版本
mike@ubuntu:~/aosp$ make -j8
```

## AOSP 切换分支

如果需要编译其他分支，执行下面的操作后，再按照原来编译的步骤即可：

```shell
cd $ANDROID_ROOT   # 放 .repo 的目录
cd .repo/manifests
git branch -av   # see all available branches on origin，建议导出到文件中看清楚了再选择
```

Select a version and:

```shell
# remotes/origin/android-7.1.2_r18                          d456b66 Manifest for Android 7.1.2 Release 18
cd $ANDROID_ROOT
repo init -b <my_selected_android_version>
# repo init -b android-7.1.2_r18
```

Such selective `repo init` with `-b` (without `-u`) will only update manifest branch and will not otherwise touch your tree.

Now, simply sync it:

```shell
repo sync -j8
```

and some time later, your Android tree will switch to another version.

## 刷机

```shell
//进入手机 bootloader
adb reboot bootloader

//开始刷机
fastboot flash boot boot.img
fastboot flash system system.img
fastboot flash cache cache.img
fastboot flash userdata userdata.img
fastboot flash recovery recovery.img
```

---

## 参考

> 安卓编译环境配置
>
> [9dcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7$3!0#2M7X3y4W2i4K6u0r3K9h3&6A6N6r3W2S2L8r3W2\*7K9h3&6Y4i4K6u0W2K9s2c8E0L8q4)9J5x3$3&6W2P5s2c8Q4x3X3c8V1L8%4N6F1L8r3!0S2k6q4)9J5k6s2c8Z5k6g2)9J5k6s2y4G2N6i4u0U0k6b7`.`.](https://bbs.kanxue.com/elink@70fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7$3!0#2M7X3y4W2i4K6u0r3K9h3&6A6N6r3W2S2L8r3W2*7K9h3&6Y4i4K6u0W2K9s2c8E0L8q4)9J5x3$3&6W2P5s2c8Q4x3X3c8V1L8%4N6F1L8r3!0S2k6q4)9J5k6s2c8Z5k6g2)9J5k6s2y4G2N6i4u0U0k6b7`.`.)
>
> 安卓系统相应硬件驱动
>
> [f29K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7$3g2@1N6i4m8Q4x3V1k6T1N6h3W2D9k6q4)9J5c8X3c8G2N6$3&6D9L8$3q4V1K9h3&6Y4i4K6t1K6L8$3u0@1j5h3W2F1K9h3&6Y4i4K6u0V1M7s2u0G2M7s2u0A6k6i4c8S2M7Y4W2Q4x3X3c8T1K9h3&6S2M7X3W2W2M7H3`.`.](https://bbs.kanxue.com/elink@373K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7$3g2@1N6i4m8Q4x3V1k6T1N6h3W2D9k6q4)9J5c8X3c8G2N6$3&6D9L8$3q4V1K9h3&6Y4i4K6t1K6L8$3u0@1j5h3W2F1K9h3&6Y4i4K6u0V1M7s2u0G2M7s2u0A6k6i4c8S2M7Y4W2Q4x3X3c8T1K9h3&6S2M7X3W2W2M7H3`.`.)
>
> 安卓源代码标记和细分版本
>
> [1dfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7$3g2@1N6i4m8Q4x3V1k6K6N6r3q4J5N6q4)9J5c8X3u0#2K9h3I4V1i4K6u0V1L8Y4g2E0j5X3g2J5M7#2)9J5x3%4y4G2N6i4u0U0k6g2)9J5k6r3y4G2k6r3g2Q4x3X3c8@1j5h3N6K6i4K6u0V1j5h3&6V1i4K6u0V1j5Y4g2A6L8r3c8K6](https://bbs.kanxue.com/elink@d84K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7$3g2@1N6i4m8Q4x3V1k6K6N6r3q4J5N6q4)9J5c8X3u0#2K9h3I4V1i4K6u0V1L8Y4g2E0j5X3g2J5M7#2)9J5x3%4y4G2N6i4u0U0k6g2)9J5k6r3y4G2k6r3g2Q4x3X3c8@1j5h3N6K6i4K6u0V1j5h3&6V1i4K6u0V1j5Y4g2A6L8r3c8K6)
>
> 清华源
>
> [eecK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0K9i4u0J5L8%4u0K6i4K6u0W2N6s2g2F1j5g2)9J5k6i4c8K6K9h3&6Y4K9s2g2S2i4K6u0W2k6h3c8#2i4K6u0W2j5$3&6Q4x3V1k6Z5k6h3I4H3i4K6u0r3b7f1!0e0f1q4)9J5c8R3`.`.](https://bbs.kanxue.com/elink@53fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0K9i4u0J5L8%4u0K6i4K6u0W2N6s2g2F1j5g2)9J5k6i4c8K6K9h3&6Y4K9s2g2S2i4K6u0W2k6h3c8#2i4K6u0W2j5$3&6Q4x3V1k6Z5k6h3I4H3i4K6u0r3b7f1!0e0f1q4)9J5c8R3`.`.)
>
> 内核代码
>
> [c5fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6%4y4Q4x3X3c8G2k6W2)9J5k6r3I4^5L8q4)9J5c8Y4m8Q4x3V1j5%4y4o6b7%4x3e0M7^5i4K6u0W2K9s2c8E0L8l9`.`.](https://bbs.kanxue.com/elink@b80K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6%4y4Q4x3X3c8G2k6W2)9J5k6r3I4^5L8q4)9J5c8Y4m8Q4x3V1j5%4y4o6b7%4x3e0M7^5i4K6u0W2K9s2c8E0L8l9`.`.)
>
> 编译错误参考
>
> [773K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3k6r3W2F1M7r3S2&6i4K6u0r3M7q4)9J5c8U0j5I4x3K6R3^5x3o6y4Q4x3X3g2Z5N6r3#2D9](https://bbs.kanxue.com/elink@7adK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3k6r3W2F1M7r3S2&6i4K6u0r3M7q4)9J5c8U0j5I4x3K6R3^5x3o6y4Q4x3X3g2Z5N6r3#2D9)
>
> 编译过程案例参考
>
> [53dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2K6L8$3&6&6i4K6u0W2j5$3!0E0i4K6u0r3k6r3g2$3k6h3I4G2M7q4)9J5c8X3!0H3k6h3&6Q4x3X3c8V1k6i4k6A6j5$3g2K6i4K6u0r3k6%4g2A6k6r3g2K6i4K6u0r3j5h3!0K6M7q4)9J5k6r3u0#2K9h3I4V1i4K6u0V1K9h3&6K6N6s2u0#2j5%4c8A6L8$3&6K6i4K6u0r3j5Y4g2A6L8r3c8Q4x3X3c8S2L8%4y4H3i4K6u0V1L8X3!0#2k6$3q4@1i4K6u0V1y4#2)9J5k6o6q4Q4x3X3b7J5i4K6t1K6N6s2g2@1L8%4u0A6j5h3I4Q4x3X3c8K6N6r3g2H3i4K6u0V1y4l9`.`.](https://bbs.kanxue.com/elink@e78K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2K6L8$3&6&6i4K6u0W2j5$3!0E0i4K6u0r3k6r3g2$3k6h3I4G2M7q4)9J5c8X3!0H3k6h3&6Q4x3X3c8V1k6i4k6A6j5$3g2K6i4K6u0r3k6%4g2A6k6r3g2K6i4K6u0r3j5h3!0K6M7q4)9J5k6r3u0#2K9h3I4V1i4K6u0V1K9h3&6K6N6s2u0#2j5%4c8A6L8$3&6K6i4K6u0r3j5Y4g2A6L8r3c8Q4x3X3c8S2L8%4y4H3i4K6u0V1L8X3!0#2k6$3q4@1i4K6u0V1y4#2)9J5k6o6q4Q4x3X3b7J5i4K6t1K6N6s2g2@1L8%4u0A6j5h3I4Q4x3X3c8K6N6r3g2H3i4K6u0V1y4l9`.`.) 这个编译的案例很多
>
> [832K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4f1H3x3e0t1I4z5e0f1^5z5e0W2Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0R3J5x3o6M7^5x3K6R3@1](https://bbs.kanxue.com/elink@634K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4f1H3x3e0t1I4z5e0f1^5z5e0W2Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0R3J5x3o6M7^5x3K6R3@1) # 对我帮助较大
>
> [235K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2y4e0q4U0N6r3!0Q4x3X3g2U0L8$3#2Q4x3V1j5I4x3U0l9&6x3o6f1$3z5g2)9J5c8U0t1K6x3e0R3%4y4e0S2Q4x3@1k6S2M7s2m8A6L8Y4y4@1j5h3I4D9i4K6y4p5x3l9`.`.](https://bbs.kanxue.com/elink@63cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2y4e0q4U0N6r3!0Q4x3X3g2U0L8$3#2Q4x3V1j5I4x3U0l9&6x3o6f1$3z5g2)9J5c8U0t1K6x3e0R3%4y4e0S2Q4x3@1k6S2M7s2m8A6L8Y4y4@1j5h3I4D9i4K6y4p5x3l9`.`.)
>
> [b3fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3j5$3!0E0L8h3!0F1x3e0p5@1x3q4)9J5c8Y4m8Q4x3V1j5&6y4e0l9^5x3U0V1K6i4K6u0W2K9s2c8E0L8l9`.`.](https://bbs.kanxue.com/elink@924K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3j5$3!0E0L8h3!0F1x3e0p5@1x3q4)9J5c8Y4m8Q4x3V1j5&6y4e0l9^5x3U0V1K6i4K6u0W2K9s2c8E0L8l9`.`.)
>

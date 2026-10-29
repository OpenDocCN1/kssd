
首先我们需要修改源码，修改源码文件中：`art/build/Android.common_build.mk`里的内容

![image-20210927100604203](https://bbs.kanxue.com/upload/attach/202109/905443_WMXNWCVK7TR9NUG.png)

不然容易报错：

![image-20210927100604203](https://bbs.kanxue.com/upload/attach/202109/905443_TEQWRS9P93EKJNS.png)

### 原因分析

|  |  |
| --- | --- |
| 1 | `修改的目的是把 CLANG 这个编译选项关掉，这个错误只会在 Ubuntu16.04 上遇到，Ubuntu14.04 则不存在这个问题` |

## <2>设置高速缓存

我们在编译源码之前为了提高编译的速率：

在 `.bashrc` 文件末尾添加：`export USE_CCACHE = 1`

|  |  |
| --- | --- |
| 1 | `echo export USE_CCACHE=1 >> ~/.bashrc` |

为了提高编译效率，设置编译器高速缓存:

|  |  |
| --- | --- |
| 1 | `prebuilts/misc/linux-x86/ccache/ccache -M 50G` |

## <3>清空输出和初始化环境

我们可以在开始编译之前清楚之前所有编译操作的输出，避免出现错误，首次编译不需要：

|  |  |
| --- | --- |
| 1 | `make clobber` |

![image-20210926204554020](https://bbs.kanxue.com/upload/attach/202109/905443_QUR9MEKR5KREMFM.png)

然后使用build目录中的`envsetup.sh`脚本初始化环境：

|  |  |
| --- | --- |
| 1 | `source build/envsetup.sh 或 . build/envsetup.sh` |

![image-20210926204554020](https://bbs.kanxue.com/upload/attach/202109/905443_EFRGF2MCYS9D32C.png)

### （4）选择编译目标

我们选择编译的具体目标，也就是指我们是将我们的源码编译到虚拟机还是具体的手机上运行，这里我们可以先看官方给的[型号图](https://bbs.kanxue.com/elink@6b3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7$3!0#2M7X3y4W2i4K6u0r3M7Y4g2F1L8X3W2F1k6H3`.`.)：

![image-20210926210739483](https://bbs.kanxue.com/upload/attach/202109/905443_EZS23HKAPT59RAR.png)

我们输入指令：`lunch`

![image-20210926213243118](https://bbs.kanxue.com/upload/attach/202109/905443_XPBB7EWVH2U5UQK.png)

参考官方解释：

![image-20210926211110079](https://bbs.kanxue.com/upload/attach/202109/905443_J5H5A4VD4WW9GDN.png)

![image-20210926212512938](https://bbs.kanxue.com/upload/attach/202109/905443_4SEJQNAQWNU6HC2.png)

我们可以理解为：

|  |  |
| --- | --- |
| 1  2  3 | `user------->最终用户机`  `userdebug----->调试测试机`  `eng--------->工程机` |

### （5）编译代码

这里我们开始编译源码，执行`make -j4`，4为同时编译的线程数，大家可以根据自己的设备情况选择

|  |  |
| --- | --- |
| 1 | `make -j4` |

查看cpu核心个数命令：

|  |  |
| --- | --- |
| 1 | `cat /proc/cpuinfo` |

编译结束，经过了漫长的过程，我们总算可以看见我们编译成功

![image-20210926213243118](https://bbs.kanxue.com/upload/attach/202109/905443_E2MKK8KKHXTRPDA.png)

我们进入输出的目录文件夹下：

![image-20210926213243118](https://bbs.kanxue.com/upload/attach/202109/905443_78Q7P8W5ZMFBUNJ.png)

Android镜像结构分析详细可以见（8）

### （7）编写刷机脚本

我们将手机进入bootloader模式：

|  |  |
| --- | --- |
| 1 | `adb reboot bootloader` |

手机如果没有解锁，我们需要解锁

|  |  |
| --- | --- |
| 1 | `fastboot oem unlock` |

我们将选中镜像导出，并刷入手机，编写刷机脚本，windows上文件后缀`.bat`，linux上文件后缀`.sh`

|  |  |
| --- | --- |
| 1  2  3  4  5 | `fastboot flash boot boot.img`  `fastboot flash system system.img`  `fastboot flash cache cache.img`  `fastboot flash userdata userdata.img`  `fastboot reboot` |

然后我们启动刷机脚本，就可以看见手机正常的刷入

![image-20210926213243118](https://bbs.kanxue.com/upload/attach/202109/905443_2ZREA8YXTZPTUJ2.png)

![image-20210926213243118](https://bbs.kanxue.com/upload/attach/202109/905443_9JECJGHJ7N4YAFM.png)

### （8）Android主要镜像解析

刷机包目录：

![image-20210927105814789](https://bbs.kanxue.com/upload/attach/202109/905443_HU7A63VCKZB5YPQ.png)

主要镜像：

![image-20210927104439801](https://bbs.kanxue.com/upload/attach/202109/905443_F9UFPGJZT2A2HA9.png)

详细参考博客：[android 主要镜像（image）解析](https://bbs.kanxue.com/elink@d52K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6#2N6r3W2S2L8Y4W2A6L8U0t1J5x3W2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4K6R3#2y4o6x3%4y4g2)9K6c8Y4g2@1L8g2)9#2k6X3#2W2k6r3W2#2L8g2)9K6c8r3c8A6M7%4c8J5K9h3u0#2N6r3g2Q4x3X3g2H3j5#2)9#2k6Y4u0W2L8r3g2$3j5h3&6@1i4K6u0W2L8X3!0F1k6g2)9J5k6s2c8S2M7$3E0Q4x3X3c8T1L8r3!0Y4i4K6u0V1x3W2)9J5y4e0N6q4k6r3g2X3j5i4g2D9N6q4)9J5y4e0N6q4b7#2c8d9e0p5W2e0g2q4)9J5y4e0N6q4k6r3g2X3j5i4g2D9N6q4)9J5k6o6y4Q4x3X3g2K6k6h3q4J5j5$3S2X3L8%4u0E0j5X3q4A6k6s2g2Z5K9h3N6Z5L8r3W2Y4K9s2c8Q4x3U0k6S2L8i4m8Q4x3@1u0V1k6i4m8@1K9q4)9#2k6U0q4Q4x3X3c8#2N6r3#2Q4y4h3k6K6L8%4g2J5j5$3g2Q4x3@1c8V1K9i4y4@1M7X3W2T1N6i4c8W2i4K6u0W2M7r3y4Q4y4h3k6J5k6h3I4W2N6X3q4F1N6q4)9J5k6h3&6G2L8X3g2Q4x3X3c8@1j5i4y4C8i4K6u0V1j5X3I4G2k6#2)9J5k6o6u0Q4x3U0f1%4c8h3c8W2k6X3q4#2L8s2c8Q4x3U0f1%4c8f1y4f1f1V1I4u0f1#2c8Q4x3U0f1%4c8h3c8W2k6X3q4#2L8s2c8Q4x3X3b7K6i4K6u0W2M7$3g2S2M7X3y4Z5k6X3!0J5L8h3u0S2K9h3c8#2K9r3W2Y4K9r3I4A6k6$3S2@1)

### （9）常见错误

#### <1>错误1

![image-20210927104439801](https://bbs.kanxue.com/upload/attach/202109/905443_QJWFKPWKVHMEG97.png)

类似错误：`make:Nothing to be done for 'all_modules'` 或 `make:***No rule to make target Android.mk Stop`等

解决办法：

|  |  |
| --- | --- |
| 1  2 | `（1）进入目录：/out/target/common/obj/JAVA_LIBRARIES，删除services_intermediates这个文件夹`  `（1）进入目录：/out/target/product/XXX/obj/JAVA_LIBRARIES，删除services_intermediates这个文件夹` |

#### <2>错误2

![image-20210927104439801](https://bbs.kanxue.com/upload/attach/202109/905443_G7NZF6XN39ED5UN.png)

问题分析：

|  |  |
| --- | --- |
| 1 | `这个错误的好像是由于在make一个任务时，并没有结束，这时在进行make就会报` |

我们使用命令：

|  |  |
| --- | --- |
| 1 | `make clobber` |

`make clobber`的功能是把上一次`make`命令生成的文件或目录清除掉，效果比`make clean`更严格

## 四、实验总结

本文详细记录了从Android6.0源码下载到Android源码编译的详细过程，其中参考了看雪上面的两位大佬帖子和网络上的一些博客，中间遇到了很多问题，并逐一解决，这里将所有的错误情况总结并放在了错误区，大家可以详细的对照，通过源码编译的过程，对Android系统的理解更加的深入了，接下来便开始进行Xposed源码的编译和定制，本文有什么问题就请大佬们指正了。

这里给大家提供android版本的源码和OpenJDK7的下载包，大家也可以去github上免费下载，后续都会上传到github上：[github网址](https://bbs.kanxue.com/elink@ce7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4N6h3!0^5N6h3q4S2i4K6u0r3b7h3&6V1M7X3!0A6k6q4)9J5k6s2u0W2N6X3g2J5M7$3g2Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4A6L8W2)9J5c8V1q4F1k6s2u0G2K9h3c8Q4x3U0g2q4y4W2)9J5y4f1u0m8i4K6t1#2z5e0m8Q4x3U0g2q4y4#2)9J5y4f1p5H3i4K6t1#2z5o6q4Q4x3U0g2q4y4g2)9J5y4f1q4q4i4K6t1#2z5f1q4Q4x3U0g2q4y4g2)9J5y4e0R3^5i4K6t1#2b7U0j5`.)

## 五、参考文献

JDK环境配置：

|  |  |
| --- | --- |
| 1  2  3  4 | `（1）https://blog.csdn.net/u014709812/article/details/114584639?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-1.searchformbaiduhighlight&spm=1001.2101.3001.4242`  `（2）https://blog.csdn.net/xiangzi10/article/details/95730747`  `（3）https://forum.xda-developers.com/t/q-kitkat-hammerhead-aosp-compilation-errors.2736673/`  `（4）https://blog.csdn.net/qq_42824983/article/details/110236257` |

Android源码下载:

|  |  |
| --- | --- |
| 1  2  3  4 | `（1）https://bbs.pediy.com/thread-264521.htm`  `（2）https://``` f1bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4F1j5X3I4G2k6%4y4Q4x3X3g2U0L8$3@1`. ```/coffee520/p/9639417.html`  `（3）https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/`  `（4）https://source.android.com/source/downloading` |

Android源码编译：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8 | `（1）https://bbs.pediy.com/thread-218513.htm`  `（2）https://blog.csdn.net/pengtgimust/article/details/90758507`  `（3）https://blog.csdn.net/fuchaosz/article/details/51487585`  `（4）https://juejin.cn/post/6844904191693946887`  `（5）https://blog.csdn.net/u012417380/article/details/72843185`  `（6）https://``` dc1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4^5P5i4A6B7k6q4)9J5k6h3y4G2L8b7`.`. ```/article/xjkelwh/103045055`  `（7）https://blog.csdn.net/cnliwy/article/details/52189349`  `（8）https://blog.csdn.net/warticles/article/details/81063377` |

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2021-9-29 13:57

被随风而行aa编辑

，原因：

[#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)

[#系统相关](https://bbs.kanxue.com/forum-161-1-126.htm)

[#源码框架](https://bbs.kanxue.com/forum-161-1-127.htm)

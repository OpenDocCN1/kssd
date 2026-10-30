
# [翻译]365天后：使用公开工具发现并利用Safari Bugs

作者：Google Project Zero团队Ivan Fratric

原文链接：

745K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5k6h3u0D9L8$3N6K6M7r3!0@1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4z5q4)9J5c8U0p5H3i4K6u0r3x3K6j5#2i4K6u0V1k6r3q4&6M7#2)9J5k6r3I4S2N6r3g2J5i4K6u0V1k6X3W2F1k6r3W2F1k6#2)9J5k6r3q4F1k6q4)9J5k6r3g2^5M7r3I4G2K9i4c8A6L8X3N6Q4x3X3g2Z5N6r3#2D9

校对：银雁冰

大约一年前，我们[发表](https://bbs.kanxue.com/elink@feeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5k6h3u0D9L8$3N6K6M7r3!0@1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4y4#2)9J5c8U0l9&6i4K6u0r3N6r3S2W2i4K6u0V1k6%4u0W2j5i4c8Q4x3X3c8V1L8$3#2Q4x3X3c8X3N6i4A6*7i4K6u0V1L8$3k6X3i4K6u0V1L8$3k6Q4x3X3b7J5x3o6p5%4i4K6u0W2K9s2c8E0L8l9`.`.)了关于现代浏览器抵抗DOM fuzzing的研究结果，其中包括一种众所周知的浏览器bug发现技术。我们还随bug列表一起公开了[Domato](https://bbs.kanxue.com/elink@6b7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5c8X3c8G2L8h3q4@1L8H3`.`.)工具，那些bug就是使用该工具发现的。

考虑到在之前的研究中，Safari，也即Webkit（它的DOM引擎），比其它浏览器情况更糟糕，我们决定在一年后使用相同的方法、相同的工具看看情况有没有改观。

## 测试步骤

和之前的研究一样，我们首先对[WebKitGTK+](https://bbs.kanxue.com/elink@baeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0C8K9i4c8Y4N6r3E0Q4x3X3g2G2M7X3M7`.)进行了fuzzing，然后在MAC中运行的Apple Safari上测试每个得到的奔溃样例。因为WebKitGTK+和Safari使用相同的DOM引擎，这使得fuzzing过程得以简化，而且fuzzing过程可以在一台普通的linux机器上进行。在本研究中，使用的是WebKitGTK+的2.20.2版本，可以在[这里](https://bbs.kanxue.com/elink@d1bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0C8K9i4c8Y4N6r3E0Q4x3X3g2G2M7X3N6Q4x3V1k6J5k6h3I4W2j5i4y4W2M7#2)9J5c8Y4N6W2j5X3E0A6N6r3N6@1K9#2)9J5k6o6u0Q4x3X3f1J5x3q4)9J5k6e0u0Q4x3X3g2@1j5i4u0Q4x3X3g2^5P5R3`.`.)下载到。

为了优化fuzzing过程，对WebKitGTK+进行了一些修改：

- 修改使得可以使用ASan (`Address Sanitizer`)构建WebKitGTK+
- 修改`window.alert()`的实现以立即调用垃圾回收过程而不是显示一个消息窗口。这对运行没影响，因为`window.alert()`在fuzzing时不会被调用
- 通常，由于WebKit采用多进程，当一个DOM bug引起一个崩溃时，只有web进程崩溃，主进程将继续运行。我们增加了代码来监控一个web进程，如果web进程崩溃，代码就以相同状态将主进程崩溃。
- 生成修改后的二进制文件

在之前的研究发布之后，我们收到许多关于fuzzing步骤的问题。这也是为什么这次我们要在下面发布对WebKitGTK+代码的修改位置和详细的构建指令。补丁文件可以在[这里](https://bbs.kanxue.com/elink@687K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5c8Y4l9H3N6r3!0G2L8s2y4Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3g2$3g2T1d9$3W2@1c8Y4g2*7P5R3`.`.)找到。注意补丁是针对WebKitGTK+ 2.20.2的，在其它版本可能无法工作。

一旦准备好WebKitGTK+的代码，在WebKitGTK+目录使用下面的命令构建带ASan的版本：

```
export CC=/usr/bin/clang

export CXX=/usr/bin/clang++

export CFLAGS="-fsanitize=address"

export CXXFLAGS="-fsanitize=address"

export LDFLAGS="-fsanitize=address"

export ASAN_OPTIONS="detect_leaks=0"



mkdir build

cd build



cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=. -DCMAKE_SKIP_RPATH=ON -DPORT=GTK -DLIB_INSTALL_DIR=./lib -DUSE_LIBHYPHEN=OFF -DENABLE_MINIBROWSER=ON -DUSE_SYSTEM_MALLOC=ON -DENABLE_GEOLOCATION=OFF -DENABLE_GTKDOC=OFF -DENABLE_INTROSPECTION=OFF -DENABLE_OPENGL=OFF -DENABLE_ACCELERATED_2D_CANVAS=OFF -DENABLE_CREDENTIAL_STORAGE=OFF -DENABLE_GAMEPAD_DEPRECATED=OFF -DENABLE_MEDIA_STREAM=OFF -DENABLE_WEB_RTC=OFF -DENABLE_PLUGIN_PROCESS_GTK2=OFF -DENABLE_SPELLCHECK=OFF -DENABLE_VIDEO=OFF -DENABLE_WEB_AUDIO=OFF -DUSE_LIBNOTIFY=OFF -DENABLE_SUBTLE_CRYPTO=OFF -DUSE_WOFF2=OFF -Wno-dev ..



make -j 4



mkdir -p libexec/webkit2gtk-4.0

cp bin/WebKit*Process libexec/webkit2gtk-4.0/
```

如果你是第一次进行构建，cmake/make步骤将提示丢失依赖项，你需要安装它们。许多对DOM fuzzing不是很重要的特性都要通过-DENABLE标记取消了。这主要为我们省去了安装相关依赖项的麻烦，但在某下情况下需要创建更加“可移植”的构建。构建完成后，进行fuzzing和创建一个[Domato](https://bbs.kanxue.com/elink@54bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5c8X3c8G2L8h3q4@1L8H3`.`.)样例一样简单，首先以如下方式运行二进制文件：

```
ASAN_OPTIONS=detect_leaks=0,exitcode=42 ASAN_SYMBOLIZER_PATH=/path/to/llvm-symbolizer LD_LIBRARY_PATH=./lib ./bin/webkitfuzz /path/to/sample <timeout>
```

等待退出代码42（根据上面的命令行和我们对WebKitGTK+代码的修改，这表明发生了ASan崩溃）。收集崩溃之后，在真实的Mac机器上对最新WebKit源代码创建了ASan构建。简单运行如下即可：

```
./Tools/Scripts/set-webkit-configuration --release --asan

./Tools/Scripts/build-webkit
```

从WebKitGTK+获得的每个崩溃在报告给Apple之前都在Mac构建上进行了测试。

## 结果

迭代1000.000.000次fuzz后（与一年前一样），我最终给Apple报告了9个bugs。去年，我估计购买执行这些迭代的计算能力需要1000美元，今年也差不多，这对很多潜在的攻击者来说都支付得起。

找到的Bug整理在下表中。所有bug在文章发布之日前都已经被[修复](https://bbs.kanxue.com/elink@d5fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6i4m8H3L8%4u0@1i4K6u0W2j5i4m8H3L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5k6s2g2K6i4K6u0r3d9q4b7J5x3o6V1I4x3o6V1`.)了。

|  |  |  |  |  |  |

| --- | --- | --- | --- | --- | --- |

| Project Zero bug ID | CVE | Type | Affected Safari 11.1.2 | Older than 6 months | Older than 1 year |

| [1593](https://bbs.kanxue.com/elink@3fcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6p5#2z5e0x3`.) | CVE-2018-4197 | UAF | YES | YES | NO |

| [1594](https://bbs.kanxue.com/elink@1e8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6p5#2z5e0b7`.) | CVE-2018-4318 | UAF | NO | NO | NO |

| [1595](https://bbs.kanxue.com/elink@df0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6p5#2z5e0f1`.) | CVE-2018-4317 | UAF | NO | YES | NO |

| [1596](https://bbs.kanxue.com/elink@6baK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6p5#2z5e0j5`.) | CVE-2018-4314 | UAF | YES | YES | NO |

| [1602](https://bbs.kanxue.com/elink@5d4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6p5$3x3o6t1`.) | CVE-2018-4306 | UAF | YES | YES | NO |

| [1603](https://bbs.kanxue.com/elink@439K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6p5$3x3o6x3`.) | CVE-2018-4312 | UAF | NO | NO | NO |

| [1604](https://bbs.kanxue.com/elink@c47K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6p5$3x3o6b7`.) | CVE-2018-4315 | UAF | YES | YES | NO |

| [1609](https://bbs.kanxue.com/elink@891K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6p5$3x3o6V1`.) | CVE-2018-4323 | UAF | YES | YES | NO |

| [1610](https://bbs.kanxue.com/elink@9d3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6p5$3x3e0l9`.) | CVE-2018-4328 | OOB read | YES | YES | YES |

UAF = use-after-free. OOB = out-of-bounds

如表中所示，9个发现的bug中有6个影响Apple Safari的发布版本，这直接影响到Safari的用户。

虽然9或6（看你怎么算）比一年前的17少很多，但这也是数量惊人的，如果统计fuzzer一年以来所有公开的bug，结果将更多。

发现这些结果后，我想看看这些bug在WebKit代码中已经存在多久了。为了检测这个，所有bug在6个多月前(`WebKitGTK+` 2.19.6)和一年前(`WebKitGTK+` 2.16.6)的WebKitGTK+版本上进行了测试。

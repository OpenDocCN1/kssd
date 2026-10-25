
# [原创]关于WinDbg的符号文件设置

## 第一种方法

也是最简单办法是在WinDug界面按CTRL+S

![](https://bbs.kanxue.com/upload/attach/201904/844326_UDD9VW3UH9TZWEU.png)

```
SRV\*D:\symbols\_folder\i386\*625K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3I4Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5c8Y4y4&6L8h3u0G2L8s2y4Q4x3U0k6F1j5Y4y4H3i4K6y4n7             //srv\*本地缓存路径\*Microsoft 公共符号服务器
```

设置完毕之后在：

```
kd> .reload
Connected to Windows XP 2600 x86 compatible target at (Sat Apr 27 11:35:53.812 2019 (GMT+8)), ptr64 FALSE
Loading Kernel Symbols
```

一般它会自动匹配你的系统，之后有个漫长的等待（从Microsoft 公共符号服务器进行缓存到本地），之后会自动完毕。

## 第二种方法

有的第一种方法（因为网络原因导致有的符号文件没有缓存进去）所以出现我之前遇到的情况

![](https://bbs.kanxue.com/upload/attach/201904/844326_7ZW9MG6FM4WE8RA.png)

查不到`KiAttachProcess`

我们可以通过手动下载符号文件

至于系统版本信息我们可以通过在需要被调试的机器里打开`CMD`输入：`dxdiag`

![](https://bbs.kanxue.com/upload/attach/201904/844326_XW7M9Z3Z3H9HQD6.png)

根据系统版本去下载相应的离线包进行解压

936K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3&6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5k6s2g2K6i4K6u0r3N6$3W2F1k6r3!0%4M7#2)9J5c8X3S2S2M7X3c8%4j5i4u0W2i4K6u0r3k6$3M7@1y4U0x3H3x3U0S2Q4x3X3g2S2M7%4m8^5

但是我打开网站发现微软不再提供离线包的下载。

我们可以去去百度下载，（下面附加几个常用的XP符号文件下载地址）

简体中文版

0d8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8Q4x3V1k6W2i4K6u0r3k6q4)9J5c8U0W2Q4x3V1k6W2k6o6W2T1y4K6R3$3x3g2)9J5k6o6c8W2y4o6y4Q4x3X3b7@1j5K6j5J5i4K6u0V1j5e0l9H3y4#2)9J5k6o6c8W2z5e0y4S2j5e0p5H3z5e0j5H3x3W2)9J5c8W2N6A6L8X3c8G2N6%4y4j5f1q4)9J5k6p5E0n7z5e0x3$3z5e0t1&6i4K6u0V1f1#2l9K6i4K6u0V1P5o6R3$3i4K6u0V1b7@1S2e0i4K6u0W2k6i4S2W2

英文版

d7eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8Q4x3V1k6V1i4K6u0r3x3#2)9J5c8U0m8Q4x3V1k6V1x3K6m8W2x3K6u0V1z5q4)9J5k6o6b7I4z5r3q4Q4x3X3b7@1y4U0W2V1i4K6u0V1j5U0j5H3x3q4)9J5k6r3j5K6x3X3y4W2x3$3g2V1k6U0b7J5k6q4)9J5c8W2N6A6L8X3c8G2N6%4y4j5f1q4)9J5k6p5E0n7z5e0x3$3z5e0t1&6i4K6u0V1f1#2l9K6i4K6u0V1P5o6R3$3i4K6u0V1c8f1&6g2i4K6u0W2k6i4S2W2

繁体中文

369K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8Q4x3V1j5J5i4K6u0r3x3W2)9J5c8X3q4Q4x3V1j5J5x3X3p5@1y4r3j5^5j5W2)9J5k6o6u0X3x3h3u0Q4x3X3b7@1y4X3p5&6i4K6u0V1z5e0b7H3j5g2)9J5k6r3y4X3y4o6l9&6z5h3k6V1x3h3c8W2z5g2)9J5c8W2N6A6L8X3c8G2N6%4y4j5f1q4)9J5k6p5E0n7z5e0x3$3z5e0t1&6i4K6u0V1f1#2l9K6i4K6u0V1P5o6R3$3i4K6u0V1b7@1S2f1i4K6u0W2k6i4S2W2

香港中文

917K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8Q4x3V1j5J5i4K6u0r3z5g2)9J5c8U0m8Q4x3V1j5J5z5e0l9H3z5r3k6W2y4g2)9J5k6r3j5$3z5e0W2Q4x3X3b7@1y4$3b7#2i4K6u0V1j5h3t1K6x3W2)9J5k6o6V1I4j5X3q4W2y4X3x3#2y4o6S2T1k6W2)9J5c8W2N6A6L8X3c8G2N6%4y4j5f1q4)9J5k6p5E0n7z5e0x3$3z5e0t1&6i4K6u0V1f1#2l9K6i4K6u0V1P5o6R3$3i4K6u0V1b7@1S2t1i4K6u0W2k6i4S2W2

其他文件自行百度。

下载完之后使用压缩文件进行解压到你想要设置的缓存路径上面我们设置的是（`D:\symbols\_folder\i386`）。

之后我们在进行上面第一步（为了是更全面）

切忌：

```
//srv\*本地缓存路径\*Microsoft 公共符号服务器
```

本地缓存路径一定要保持一致。

之后我们在进行`u KiAttachProcess`时

![](https://bbs.kanxue.com/upload/attach/201904/844326_VF9374G9YMC7UW4.png)

我们就可以成功查看到。

再次感谢看雪其他大佬点明![](https://bbs.kanxue.com/upload/attach/201904/844326_DKDXNTNGW73Y9EA.png)

以及让我可以做伸手党的热心大佬![](https://bbs.kanxue.com/upload/attach/201904/844326_ZC4YU3ENCESCGH6.png)

山总老矣，尚能饭否。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

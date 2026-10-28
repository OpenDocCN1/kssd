
![](upload/attach/201909/706972_2WVGQDZCTVQV6SX.png)

![](upload/attach/201909/706972_RKTPCBRQSWP7FAM.png)

![](upload/attach/201909/706972_NR6FHUGBABAEPH4.png)

![](upload/attach/201909/706972_7Q9VHJYQYUMTGXD.png)

![](upload/attach/201909/706972_GAGNZ9KE9YJSEV5.png)

![](upload/attach/201909/706972_M4JP7K6GM4UF76V.png)

![](upload/attach/201909/706972_Y8ZDU9HN3UFF9R5.png)

从上图中可以发现，壳确实是被一些恶意软件所使用。

出于好奇，我分析了`administrator`用户的加密器（也是由这篇文章开头调查到的用户构造的），分析表明这三个样例都是由同一个版本（或更老的？）的`CypherIT`生成的。

之所以说可能是由更老的版本生成的，是因为，如果看一下其他例子，可以发现混淆技术在不断地更新。

# IV - 最近的例子

大约在`2019.02.11`，`YARA`规则的开发结束了，因为`CypherIT`进行了一次更新。

从`2019.07.23`时在`MalSilo`中观察到的一个最近的例子（`7b252dbb6a39ea4814d29ebfd865f8a46a69dd9f`）以及一些之前的（`2019`年`2`月-`2019`年`7`月）例子，表明混淆技术或多或少地在不断更新。

到这里我们再从头梳理一下。

最后一部分会调查一些函数，下面是一些结论。

1. 在运行时，第一条指令是一个循环，由大量的`if`语句组成，在这里初始化了一些变量。
2. 在循环之前设置了一个控制变量，确定接下来应该跳到哪一个块
3. 到每个`if`代码段的结束部分，控制变量会重新初始化，确定下一次跳转到哪里
4. 一旦代码块被执行，一个特定的函数会被解析并调用。

![](upload/attach/201909/706972_M2G8A5RAJDPKDWA.png)

1. 最后，执行流就几乎和旧版本一样了。

像上面描述的扁平化控制流（`CFF`，`Control Flow Flattening`），几乎在每个函数中都有，还伴有字符串混淆，最后一例中，有时有，有时没有`CFF`。

当提到沙盒检测时，就不得不提到两个新的函数

1. 检测`VirtualBox`和`VMware`的`VM`客户端工具
2. 追踪鼠标移动

`AntiVM_mouse_check`的代码段使用了`CFF`和字符串混淆

![](upload/attach/201909/706972_FQB82E3BFRVAV8P.png)

恢复（译者注：去除字符串混淆和控制流扁平化）之后的`AntiVM_mouse_check`

![](upload/attach/201909/706972_DCFFDVWP8SNREB7.png)

`AntiVM_process_check`

![](upload/attach/201909/706972_CQQ4VEKBW9BNQNZ.png)

恢复之后的`AntiVM_process_check`

![](upload/attach/201909/706972_7FQJRVAEJT9NKYY.png)

关于`payload`，最新版本的`payload`存储在多个资源中，`RESOURCE_TYPE_FONTDIR`这个函数负责重建它，如下所示。

第一个函数参数，`$data`，接收一个由 `|` 分隔的资源列表，用它来重建最后的`payload`

`$data=X|USXlhrrTcD|JqLn|hEuiNUhgRzrxs|nyFoHiqBt|PJYZYBUO|ChaHOMZLQtIa|AFpMebeesFkYteWii|FCSQpnQ|BHxAiLvVjtJlwSKA`

![](upload/attach/201909/706972_6945MMH57XAD3FK.png)

关于`shellcode`没有什么特殊的地方好说的，它还是和之前分析的几个例子一样。

# 最后的想法

`CypherIE`并没有什么打破规则的技术，在揭开混淆层之后，可以发现它还是和`2018`年观察到的特征一样，只是添加了一些将各种技术组合起来的小把戏和一些功能（在最后一部分并没有完整地列出）

尽管最近使用了`CFF`和新的字符串混淆的样例增大了分析难度，但是因为其命名约定，一些函数和相关参数就显得有点多余。

如果说之前版本的样本是将`payload`存储在`PE`文件的一个`section`中，最新的版本则是将它分割开来存储在多个资源中，但是从本质上来说，重建的技术还是和以前一样。

除此之外，因为代码的框架逻辑在经过几次更新后仍然没有改变，一旦新的把戏被揭穿，分析步骤就和之前的样例一样了。

基于`MalSilo`遥测数据在这篇文章中列出来了，可以假设在偷窥数据的恶意软件中，加密器基本上是一样的。

# 附录

## ATT&CK技术

| **Tactic** | **ID** | **名称** |
|----------------------|-----------------------------------------------------|-----------------------------------------|
| Persistance | <a href="elink@41fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9#2x3#2)9J5c8R3`.`.">T1053</a> | Scheduled Task |
| Persistance | <a href="elink@851K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0p5#2z5q4)9J5c8R3`.`.">T1158</a> | Hide Files and Directories |
| Privilege escalation | <a href="elink@3acK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9^5z5q4)9J5c8R3`.`.">T1088</a> | Bypass User Account Control |
| Defense evasion | <a href="elink@89bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9&6x3#2)9J5c8R3`.`.">T1093</a> | Process Hollowing |
| Defense evasion | <a href="elink@28eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9#2y4g2)9J5c8R3`.`.">T1055</a> | Process Injection |
| Defense evasion | <a href="elink@02eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9@1y4g2)9J5c8R3`.`.">T1045</a> | Software Packing |
| Defense evasion | <a href="elink@78cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9J5y4H3`.`.">T1027</a> | Obfuscation Files or Information |
| Defense evasion | <a href="elink@01cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0p5@1x3q4)9J5c8R3`.`.">T1140</a> | Deobfuscate/Decode Files or Information |

## YARA规则

```
rule cypherit_shellcode
{
meta:
author = "raw-data"
tlp = "white"
version = "1.0"
created = "2019-01-25"
modified = "2019-01-25"
description = "Detects CypherIT shellcode"
strings:
$win_api1 = { c7 8? ?? ?? ?? ?? ee 38 83 0c c7 8? ?? ?? ?? ?? 57 64 e1 01 c7 8? ?? ?? ?? ?? 18 e4 ca 08 }
$win_api2 = { c7 8? ?? ?? ?? ?? e3 ca d8 03 c7 8? ?? ?? ?? ?? 99 b0 48 06 }
$hashing_function = { 85 c9 74 20 0f be 07 c1 e6 04 03 f0 8b c6 25 00 00 00 f0 74 0b c1 e8 18 33 f0 81 e6 ff ff ff 0f 47 49 }
condition:
(1 of ($win_api*)) and $hashing_function
}
```

## MISP事件

<a href="elink@92dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5j5i4N6Q4x3X3c8V1j5i4c8S2i4K6u0r3L8h3W2K6M7q4)9J5k6r3g2$3k6h3&6@1M7#2)9J5c8Y4u0S2N6#2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8U0t1H3x3e0W2Q4x3X3b7H3y4#2)9J5k6o6t1$3i4K6g2X3c8i4S2H3L8r3!0J5K9h3&6Y4i4K6g2X3b7i4g2@1L8@1W2f1i4K6g2X3c8W2g2p5i4K6g2X3b7%4u0&6M7s2c8W2M7W2)9J5k6h3A6K6L8$3&6Q4x3X3g2*7K9i4l9`.">下载</a>

## CypherIT样例和payload[08.2018 - 02.2019]

<a href="elink@2bbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5j5i4N6Q4x3X3c8V1j5i4c8S2i4K6u0r3L8h3q4D9j5i4u0U0K9r3W2$3k6g2)9J5c8Y4c8J5k6h3g2Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6U0M7Y4W2H3N6r3g2J5i4K6u0r3j5%4W2H3K9r3g2J5K9i4c8Q4x3V1j5J5x3o6p5^5i4K6u0W2x3o6S2Q4y4h3j5J5x3o6p5&6i4K6u0W2x3o6t1`.">Malarchive</a>

## IOCs - 2019.07.23

| **收集日期** | **Crypter(sha1)** | **获取网址** |
|--------------|------------------------------------------|------------------------------------|
| 2019-07-23 | 7b252dbb6a39ea4814d29ebfd865f8a46a69dd9f | hXXp://mimiplace[.]top/invoice.exe |

## IOCs [08.2018 - 02.2019]

49个获取网址和44个样例

| **收集日期** | **Crypter(sha1)** | **获取网址** |
|--------------|------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 2018-08-28 | bdf0f4184794a4e997004beefde7a29066e47847 | hXXp://com2c.com[.]au/filehome/4hih |
| 2018-09-03 | 55646431095967fc5d41d239de70a8ffbd8d0833 | hXXp://service-information-fimance[.]bid/Java.exe |
| 2018-09-03 | 823e2d3ef005d36b1401472dd5fd687a652b81b0 | hXXp://service-information-fimance[.]bid/NETFramework.exe |
| 2018-09-03 | ec33f922fb324d7d2d4ee567ceba4563c6700661 | hXXp://service-information-fimance[.]bid/AMADEUSapp.exe |
| 2018-09-04 | aed69a2e740e789139118e3753107f9d892790c7 | hXXp://letmeplaywithyou[.]com/grace/bless.exe |
| 2018-09-05 | 4f193f9724f8b37fe998f5d159ece8528f608fa9 | hXXps://a.doko[.]moe/izgvrd |
| 2018-09-05 | aed69a2e740e789139118e3753107f9d892790c7 | hXXps://letmeplaywithyou[.]com/grace/bless.exe |
| 2018-09-07 | ddf3a42a85fb4ae2fe5c86e7305265e8c46e52a9 | hXXp://bit[.]ly/2Q6hlGD |
| 2018-09-07 | ddf3a42a85fb4ae2fe5c86e7305265e8c46e52a9 | hXXps://b.coka[.]la/sxPC9O.jpg |
| 2018-09-17 | a81bc74373b5d948472e089877fa3b64c83c4fda | hXXps://a.doko[.]moe/hpofbv |
| 2018-09-19 | e82e4a048cc66dfee9979a2db70e63b60a6aa3cb | hXXp://lse-my[.]asia/servfbtmi.exe |
| 2018-09-19 | 3fae31b10d8154edd1bfcca1c98cc3f61a78fdac | hXXp://thepandasparadise[.]com/cts/dfgf/ExceI_Protected.exe |
| 2018-09-19 | 5387c0ead3450eaef1cc82e4c4a0b52982fb2952 | hXXp://thepandasparadise[.]com/cts/dfgf/dfdgfh/server_Pro.exe |
| 2018-09-19 | 50a250aeb3c685e04cd2fce62634d1b95920cbab | hXXp://scientificwebs[.]com/1.exe |
| 2018-09-19 | 65b0e55170715d14ed139a7e1cd1710685e19a7d | hXXps://scientificwebs[.]com/1.exe |

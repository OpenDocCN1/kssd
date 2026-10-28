
关于shellcode没有什么特殊的地方好说的，它还是和之前分析的几个例子一样。

# 最后的想法

CypherIE并没有什么打破规则的技术，在揭开混淆层之后，可以发现它还是和2018年观察到的特征一样，只是添加了一些将各种技术组合起来的小把戏和一些功能（在最后一部分并没有完整地列出）

尽管最近使用了CFF和新的字符串混淆的样例增大了分析难度，但是因为其命名约定，一些函数和相关参数就显得有点多余。

如果说之前版本的样本是将payload存储在PE文件的一个section中，最新的版本则是将它分割开来存储在多个资源中，但是从本质上来说，重建的技术还是和以前一样。

除此之外，因为代码的框架逻辑在经过几次更新后仍然没有改变，一旦新的把戏被揭穿，分析步骤就和之前的样例一样了。

基于MalSilo遥测数据在这篇文章中列出来了，可以假设在偷窥数据的恶意软件中，加密器基本上是一样的。

# 附录

## ATT&CK技术

| **Tactic** | **ID** | **名称** |
| --- | --- | --- |
| Persistance | [T1053](https://bbs.kanxue.com/elink@0d7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9#2x3#2)9J5c8R3`.`.) | Scheduled Task |
| Persistance | [T1158](https://bbs.kanxue.com/elink@dfcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0p5#2z5q4)9J5c8R3`.`.) | Hide Files and Directories |
| Privilege escalation | [T1088](https://bbs.kanxue.com/elink@03dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9^5z5q4)9J5c8R3`.`.) | Bypass User Account Control |
| Defense evasion | [T1093](https://bbs.kanxue.com/elink@aa4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9&6x3#2)9J5c8R3`.`.) | Process Hollowing |
| Defense evasion | [T1055](https://bbs.kanxue.com/elink@b0eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9#2y4g2)9J5c8R3`.`.) | Process Injection |
| Defense evasion | [T1045](https://bbs.kanxue.com/elink@51cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9@1y4g2)9J5c8R3`.`.) | Software Packing |
| Defense evasion | [T1027](https://bbs.kanxue.com/elink@eccK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9J5y4H3`.`.) | Obfuscation Files or Information |
| Defense evasion | [T1140](https://bbs.kanxue.com/elink@df4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0p5@1x3q4)9J5c8R3`.`.) | Deobfuscate/Decode Files or Information |

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



        $win_api1 = { c7 8? ?? ?? ?? ?? ee 38 83 0c c7 8? ?? ?? ?? ?? 57 64 e1 01 c7 8? ?? ?? ?? ?? 18 e4 ca 08  }

        $win_api2 = { c7 8? ?? ?? ?? ?? e3 ca d8 03 c7 8? ?? ?? ?? ?? 99 b0 48 06  }



        $hashing_function = { 85 c9 74 20 0f be 07 c1 e6 04 03 f0 8b c6 25 00 00 00 f0 74 0b c1 e8 18 33 f0 81 e6 ff ff ff 0f 47 49  }



    condition:

        (1 of ($win_api*)) and $hashing_function

}
```

## MISP事件

[下载](https://bbs.kanxue.com/elink@f15K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5j5i4N6Q4x3X3c8V1j5i4c8S2i4K6u0r3L8h3W2K6M7q4)9J5k6r3g2$3k6h3&6@1M7#2)9J5c8Y4u0S2N6#2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8U0t1H3x3e0W2Q4x3X3b7H3y4#2)9J5k6o6t1$3i4K6g2X3c8i4S2H3L8r3!0J5K9h3&6Y4i4K6g2X3b7i4g2@1L8@1W2f1i4K6g2X3c8W2g2p5i4K6g2X3b7%4u0&6M7s2c8W2M7W2)9J5k6h3A6K6L8$3&6Q4x3X3g2*7K9i4l9`.)

## CypherIT样例和payload[08.2018 - 02.2019]

[Malarchive](https://bbs.kanxue.com/elink@827K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5j5i4N6Q4x3X3c8V1j5i4c8S2i4K6u0r3L8h3q4D9j5i4u0U0K9r3W2$3k6g2)9J5c8Y4c8J5k6h3g2Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6U0M7Y4W2H3N6r3g2J5i4K6u0r3j5%4W2H3K9r3g2J5K9i4c8Q4x3V1j5J5x3o6p5^5i4K6u0W2x3o6S2Q4y4h3j5J5x3o6p5&6i4K6u0W2x3o6t1`.)

## IOCs - 2019.07.23

| **收集日期** | **Crypter(sha1)** | **获取网址** |
| --- | --- | --- |
| 2019-07-23 | 7b252dbb6a39ea4814d29ebfd865f8a46a69dd9f | hXXp://mimiplace[.]top/invoice.exe |

## IOCs [08.2018 - 02.2019]

49个获取网址和44个样例

| **收集日期** | **Crypter(sha1)** | **获取网址** |
| --- | --- | --- |
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
| 2018-09-19 | 3fae31b10d8154edd1bfcca1c98cc3f61a78fdac | hXXp://thepandasparadise[.]com/cts/dfgf/ExceI\_Protected.exe |
| 2018-09-19 | 5387c0ead3450eaef1cc82e4c4a0b52982fb2952 | hXXp://thepandasparadise[.]com/cts/dfgf/dfdgfh/server\_Pro.exe |
| 2018-09-19 | 50a250aeb3c685e04cd2fce62634d1b95920cbab | hXXp://scientificwebs[.]com/1.exe |
| 2018-09-19 | 65b0e55170715d14ed139a7e1cd1710685e19a7d | hXXps://scientificwebs[.]com/1.exe |
| 2018-09-19 | 686e7c7e4767cc7198c71bc99596c226fbf1ab36 | hXXp://thepandasparadise[.]com/cts/dfgf/win32\_Pro.exe |
| 2018-09-19 | 05fec020078b53643cb14a8ec7db3f2aa131e572 | hXXp://thepandasparadise[.]com/cts/dfgf/dfdgfh/win32\_Pro.exe |
| 2018-09-19 | 1adc1f7d7fd258a75c835efd1210aa2e159636aa | hXXp://thepandasparadise[.]com/cts/ExceI\_Protected.exe |
| 2018-09-19 | bbcd6a0a7f73ec06d2fab527f418fca6d05af3a6 | hXXp://lse-my[.]asia/dotvmptee.exe |
| 2018-09-19 | a5944808c302944b5906d892a1fd77adaf4a309c | hXXp://thepandasparadise[.]com/cts/dfgf/dfdgfh/fgbh/server\_Pro.exe |
| 2018-09-19 | 90d6f6bb6879862cb8d8da90c99cb764f064bc5a | hXXp://thepandasparadise[.]com/cts/dfgf/winRAR1.exe |
| 2018-09-20 | 50a250aeb3c685e04cd2fce62634d1b95920cbab | hXXps://scientificwebs[.]com/1.exe |
| 2018-09-20 | a9856ca5ecba168cc5ebe39c3a04cb0c0b432466 | hXXp://scientificwebs[.]com/1.exe |
| 2018-09-21 | 7d2cddf145941456c7f89eb0ecbbaabb1eb4ef0a | hXXps://b.coka[.]la/E5CoMb.jpg |
| 2018-09-21 | 767945f40c2de439c5107456e34f014149da16e6 | hXXp://lse-my[.]asia/servfbtmi.exe |
| 2018-09-21 | 6a41eb6dbfe98444f126d42b1b5818767ced508d | hXXp://lse-my[.]asia/servfbtmi.exe |
| 2018-09-21 | a9856ca5ecba168cc5ebe39c3a04cb0c0b432466 | hXXps://scientificwebs[.]com/1.exe |
| 2018-09-25 | 6e75dc48ec0380e189f67ba7f61aff99f5d68a04 | hXXp://b.coka[.]la/sMZD0n.jpg |
| 2018-09-25 | 3e5bef4eaf3975de6573a2fd22ce66ad6c88c652 | hXXps://b.coka[.]la/E19F0D.jpg |
| 2018-09-25 | 50c2b8ac2d8f04a172538abaa00bcb5dc135bb12 | hXXp://b.coka[.]la/ZKW6B.jpg |
| 2018-09-27 | 96136f00f59a44c2bce10755bcced5c362868766 | hXXp://lse-my[.]asia/stbincrp.exe |
| 2018-09-27 | b199f2488849b3bcad92e85190d2525422b1a644 | hXXps://share.dmca[.]gripe/FxJ0r9YOSecgw9FP |
| 2018-09-28 | 8973591f584f2b104559cc5bc73838ff0df0e50f | hXXp://lse-my[.]asia/injclientcrp.exe |
| 2018-09-28 | 03469616ce1ce960edbc6be814ca3ab86902067d | hXXp://lse-my[.]asia/stbincrp.exe |
| 2018-09-28 | 2ef17bc8b67f17fb5957c8edc969aa5bdcc1c76e | hXXp://lse-my[.]asia/goosmi.exe |
| 2018-09-28 | 6bdad0aae469313a8567ad1138a744dca74f1ecc | hXXp://lse-my[.]asia/pacbellcrp.exe |

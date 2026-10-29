
`adb reboot bootloader` / `/` `手机进入bootloader模式 [音量调低键 按电源 也行]` `fastboot flashall -w` / `/` `开始刷机`

![](https://bbs.kanxue.com/upload/attach/202012/907124_C8Q7U4SB47DV2YU.png)

完美搞定，收工……

## 一个悲伤的故事

wifi连不上，不能上网。问了问谷哥，哥说，问题出在`persist`分区损坏，只需要格式化一下这个分区就好了。

我信他个鬼， 从 [1a9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0K6i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5c8X3W2E0j5h3N6W2M7H3`.`.](https://bbs.kanxue.com/elink@91bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0K6i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5c8X3W2E0j5h3N6W2M7H3`.`.) 下载 Pixel 2 XL的`QP1A.190711.020` 工厂包，发觉里面有个 `radio-taimen-g8998-00012-1905270706.img` 是我们编译的代码里面没有的。

`radio`是通讯基带文件，果断先刷一次工厂包，然后再刷我们编译的版本。这次成功可以上网了。

我觉得 直接刷一下基带文件应该也可以解决，有兴趣的小伙伴可以试试。

|  |  |
| --- | --- |
| 1 | `fastboot flash radio radio-taimen-g8998-00012-1905270706.img` |

## 避坑指南

- 编译报错

`FAIL: TestGlobEscapes //\* (0.00s) glob_test.go:562: incorrect matches list: glob_test.go`

把`aosp/build/blueprint/Blueprints` 文件里的 `glob_test.go` 和 `fs_test.go` 的行注释掉

- 安装`xcode`命令行

```
xcode-select --install
```

# 总结

Android的源码配置的已经很傻瓜化了，在Ubuntu和Mac的的编译非常顺利，就是刷机遇到点困难，网上的很多教程也没有很详细的讲刷机的步骤。奋飞提供的思路就是和官方工厂包对比，缺什么咱们补什么。

科学上网是研究Android的先决条件。(￣▽￣)ゞ

## 参考

[c00K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0p5&6z5e0R3&6z5l9`.`.](https://bbs.kanxue.com/elink@221K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0p5&6z5e0R3&6z5l9`.`.) 肉丝大佬指南原文

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

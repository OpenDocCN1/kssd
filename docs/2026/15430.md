
# [分享]Xposed工具Safooty

最近开发了一个Xposed工具，目标是减少基本的、简单的Xposed代码开发，辅助测试，定位目标，等等。

## 备注

1. 这不是个神器，不能避免所有的代码开发。
2. 只是能够减少一些简单的重复性的代码开发。
3. 能够辅助快速定位，减少手机重启，但是需要重启目标应用。
4. 目前该工具只能适用于dalvik环境下。在ART模式下无效。
5. 目前不能处理动态加载的`dex`，只能处理安装过的`apk`。
6. 等等......有兴趣的话自己去发现吧（^_^）

## 使用方法

1. 安装Safooty、激活插件、重启手机（基于Xposed）。
2. 选择目标应用。

   ![](https://bbs.kanxue.com/upload/attach/201804/730734_C6PK8XQFRVJQ543.jpg)

3. 解析类和方法。

   ![](https://bbs.kanxue.com/upload/attach/201804/730734_96RARH6T8DA66A7.jpg)

4. 选择需要hook的类、方法，并保存。

   ![](https://bbs.kanxue.com/upload/attach/201804/730734_Q2GFABTCFSN5NJN.jpg)

5. 重启目标软件。
6. 通过`logcat`过滤`TAG`为`Safooty`的日志，即可看到hook的结果。

## 目前支持的`XC_MethodHook`回调包括以下内容：

1. 打印参数列表
2. 打印返回值
3. 打印方法调用栈和参数列表
4. 打印方法调用栈和返回值
5. 打印方法调用栈、参数列表和返回值
6. 打印参数列表和返回值
7. 替换返回结果为`true`
8. 替换返回结果为`false`
9. 替换基本数据类型(包括字符串)的返回值，比如指定返回字符串`"9000"`
10. 替换基本数据类型(包括字符串)的参数

工具下载地址：`de8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3L8$3!0J5k6h3g2Q4x3V1k6X3L8$3!0j5M7r3!0K6k6h3c8Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3k6X3W2D9k6i4y4Q4x3V1k6e0j5h3k6G2L8%4c8&6i4K6u0V1x3g2)9J5k6e0m8Q4x3X3c8#2M7$3g2Q4x3X3c8A6L8W2)9J5k6p5q4F1k6s2u0G2K9h3b7@1i4K6u0W2y4q4)9J5k6e0c8Q4x3X3g2S2M7r3D9`。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

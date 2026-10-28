
# [翻译]DLL劫持自动化检测

# DLL劫持自动化检测

## 简介

本文将介绍动态链接库（DLL）搜索顺序劫持的概念，以及如何实现在Windows上的持久化攻击。MITRE ATT&CK对DLL搜索顺序劫持的描述详见<a href="elink@d4bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3X3g2E0K9i4c8J5k6g2)9J5k6h3!0J5k6#2)9J5c8Y4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1k6f1x3e0f1%4y4q4)9J5c8U0l9H3x3g2)9J5c8R3`.`.">DLL Search Order Hijacking (T1038)</a>。

DLL劫持对攻击者很有用，原因有很多，但本文会重点介绍与开机自启程序结合使用时的持久化攻击。例如，由于Slack和Microsoft Teams开机自启（默认情况下），因此，只要用户登录，这些应用程序中的DLL劫持将允许攻击者能够持久访问攻击目标。

在介绍DLL、DLL搜索顺序和DLL劫持的概念后，我会阐述DLL劫持自动化检测的过程<<mark class="encrypted">25fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6L8s2W2V1x3r3N6Q4x3V1k6p5e0p5I4t1K9h3A6S2j5$3E0f1k6i4y4@1</mark>>。本文将涉及Slack、Microsoft Teams以及Visual Studio Code相关的DLL劫持检测。

最后，我注意到很多DLL劫持会同时作用于多个程序。我调查了底层原因，发现调用某些位于`**C:\Windows\System32\**`下的Windows API的程序容易成为DLL劫持的目标。

这里我要感谢我的同事，Josiah Massari （<a href="elink@f48K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3j5h3W2J5P5X3g2J5L8K6t1@1">@Airzero24</a>）。是他最早发现的这些DLL劫持案例并解释其方法和原理，促成了我将检测做成自动化。

### DLL是什么

*DLL是一个包含可被多个程序同时使用的代码和数据的库。*（定义源于<a href="elink@b6fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6i4m8H3L8%4u0@1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3S2W2L8s2m8Q4x3V1j5^5x3e0f1H3y4U0g2Q4x3V1k6%4K9r3q4@1i4K6u0V1K9i4y4Q4x3X3c8S2i4K6u0V1k6r3I4D9">Microsoft</a>）

Windows程序可以调用<a href="elink@784K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8X3I4A6j5X3I4G2j5h3c8W2M7X3q4H3K9g2)9J5c8X3&6X3i4K6u0V1L8r3W2T1L8r3!0S2k6r3g2J5j5i4m8A6i4K6u0V1L8r3!0S2k6r3I4A6j5Y4u0S2M7Y4W2S2">**LoadLibrary**</a>系列函数来使用DLL中的功能。程序可以引用自定义的DLL，也可以引用System32中Windows自带的DLL。开发者可以加载System32中的DLL，使其程序使用Windows的现成功能，而不必重新造轮子。

例如，开发者需要发起HTTP请求时，可以使用WinHTTP库（**winhttp.dll**），而无需用原始套接字开发并实现HTTP功能。

### DLL搜索顺序和劫持

由于DLL也是存储在磁盘上的文件，那么程序是如何知道从哪里加载DLL的呢？Microsoft在<a href="elink@c8eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3c8D9L8s2y4Q4x3V1k6V1P5h3&6S2L8h3W2U0i4K6u0V1L8r3W2F1K9#2)9J5k6r3I4A6j5Y4u0S2M7Y4W2Q4x3X3c8K6k6h3q4J5j5$3S2Q4x3X3c8G2M7X3c8W2M7R3`.`.">此</a>详细记录了DLL搜索顺序。

自Windows XP SP2起，安全DLL搜索模式默认开启（**HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Session Manager\SafeDllSearchMode**）。当安全模式启用时，搜索顺序如下：

1. 程序安装目录。
2. 系统目录。使用<a href="elink@427K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6V1k6i4y4C8N6r3!0H3i4K6u0r3j5i4m8A6i4K6u0r3M7%4W2K6K9h3&6X3L8$3q4H3K9g2)9J5c8X3&6X3i4K6u0V1M7%4W2K6K9h3&6X3L8$3q4H3K9g2)9J5k6r3N6W2N6s2y4&6M7%4c8W2L8h3c8A6M7X3g2U0N6r3!0J5P5h3p5`.">**GetSystemDirectory**</a>获取该目录路径。
3. 16位系统目录。没有函数可以获取该目录路径，但该目录会被搜索。
4. Windows目录。使用<a href="elink@29bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6V1k6i4y4C8N6r3!0H3i4K6u0r3j5i4m8A6i4K6u0r3M7%4W2K6K9h3&6X3L8$3q4H3K9g2)9J5c8X3&6X3i4K6u0V1M7%4W2K6K9h3&6X3L8$3q4H3K9g2)9J5k6r3N6W2N6s2N6A6L8X3c8G2N6%4y4V1K9i4u0W2j5%4c8G2M7Y4W2S2">**GetWindowsDirectory**</a>获取该目录路径。
5. 当前目录。
6. PATH环境变量中的目录。注意，这不包括**App Paths**注册表键指定的程序路径。**App Paths**键不会参与DLL搜索路径的计算。

*系统可包含动态链接库（DLL）的多个版本。程序可通过指定完整路径或使用比如manifest的其他机制来控制加载特定的DLL。*（定义源于<a href="elink@916K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3c8D9L8s2y4Q4x3V1k6V1P5h3&6S2L8h3W2U0i4K6u0V1L8r3W2F1K9#2)9J5k6r3I4A6j5Y4u0S2M7Y4W2Q4x3X3c8K6k6h3q4J5j5$3S2Q4x3X3c8G2M7X3c8W2M7R3`.`.">Microsoft</a>）

如果程序没有加载指定的DLL，Windows会默认执行上述的DLL搜索流程。DLL搜索的第一顺位，即程序的安装目录，是攻击者最感兴趣的。

如果开发者想要从**C:\Windows\System32\**加载 DLL，但并没有写清楚让程序这么做，那么程序安装目录下的恶意DLL就会优先于System32下的合法DLL被加载。这里恶意DLL指的是DLL劫持，攻击者用来在受信/签名程序中加载恶意代码。

### DLL劫持持久化利用

当目标程序/服务启动并且恶意DLL被植入到了关键位置，DLL劫持即可用于持久化攻击。我的同事<a href="elink@c93K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3j5h3W2J5P5X3g2J5L8K6t1@1">@Airzero24</a>，发现了Microsoft OneDrive、 Microsoft Teams、以及Slack中针对**userenv.dll**的劫持。

这类程序被盯上的原因是，他们默认都会随着Windows开机启动。任务管理器中可以看到：

![开机启动的WIndows程序](upload/attach/202007/747358_FKV9XXNFVS8TY3Y.jpg)

为了验证DLL劫持，我写了一段运行Cobalt Strike Beacon的DLL shellcode，命名为**userenv.dll**，并且复制到目标程序的安装目录。运行程序，我看到肉鸡上线了。

![通过DLL劫持运行Cobalt Strike Beacon](upload/attach/202007/747358_UM5CS4JXJ5N4RDH.jpg)

查看<a href="elink@3dfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4y4&6M7$3W2F1N6r3g2J5L8X3q4D9M7#2)9J5c8X3c8G2N6$3&6D9L8$3q4V1M7#2)9J5c8Y4m8J5L8$3y4W2M7%4y4Q4x3X3c8W2P5s2m8D9L8%4u0W2M7R3`.`.">Process Explorer</a>，可以确认恶意DLL确实被目标程序加载了。

![Process Explorer中显示恶意DLL被加载](upload/attach/202007/747358_GX7V7CU37U26K5U.jpg)

## DLL劫持自动化检测

确认先前已知的DLL劫持之后，我想看看是否可以找到其他的DLL劫持。

测试代码在此：<<mark class="encrypted">c57K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6L8s2W2V1x3r3N6Q4x3V1k6p5e0p5I4t1K9h3A6S2j5$3E0f1k6i4y4@1</mark>>

### 案例研究：Slack

首先，运行<a href="elink@50fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4y4&6M7$3W2F1N6r3g2J5L8X3q4D9M7#2)9J5c8X3c8G2N6$3&6D9L8$3q4V1M7#2)9J5c8Y4m8J5L8$3y4E0L8$3^5`.">Process Monitor</a>（ProcMon），过滤规则如下：

+ **Process Name**为*slack.exe*
+ **Result**包含*NOT FOUND*
+ **Path**以*.dll*结尾

![ProcMon过滤规则](upload/attach/202007/747358_GY8WGMCJ2482C6B.jpg)

接下来，运行Slack，留意Slack搜索但无法找到的DLL。

![用ProcMon找到的疑似DLL劫持](upload/attach/202007/747358_AHZG4VZ69VMKV72.jpg)

从ProcMon中导出数据，方便用PowerShell处理。

用当前加载shellcode的DLL，我无法轻易找出被Slack加载的DLL的名字。我写了一个新的使用了<a href="elink@6a2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8X3I4A6j5X3I4G2j5h3c8W2M7X3q4H3K9g2)9J5c8X3&6X3i4K6u0V1L8r3W2T1L8r3!0S2k6r3g2J5j5i4m8A6i4K6u0V1k6$3g2@1L8h3!0V1N6h3I4W2K9r3q4F1k6r3I4W2k6i4S2S2">**GetModuleHandleEx**</a>和<a href="elink@505K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8X3I4A6j5X3I4G2j5h3c8W2M7X3q4H3K9g2)9J5c8X3&6X3i4K6u0V1L8r3W2T1L8r3!0S2k6r3g2J5j5i4m8A6i4K6u0V1k6$3g2@1L8h3!0V1N6h3I4W2k6X3W2D9k6h3&6S2L8h3g2S2">**GetModuleFileNameDLL**</a>的DLL，<a href="elink@c1bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6L8s2W2V1x3r3N6Q4x3V1k6p5e0p5I4t1K9h3A6S2j5$3E0f1k6i4y4@1i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8V1c8x3e0p5S2A6K9X3q4U0K9#2c8W2M7%4c8Q4x3V1k6%4M7X3W2@1k6g2)9J5k6h3y4H3M7l9`.`.">输出加载DLL的名字到文本文件</a>。

接下来就是解析CSV得到DLL路径列表，遍历路径列表，把我的测试DLL复制到特定路径，运行目标进程，退出目标进程，删除测试DLL。如果测试DLL被成功加载，其文件名会写入结果文件。

上述步骤完成后，会得到一个有效DLL劫持的列表。

在我的DLLHijackTest项目中的PowerShell脚本可以处理所有步骤。脚本的输入参数有：ProcMon生成的CSV文件的路径，恶意DLL的路径，目标程序的路径，以及其他你想要传递给目标程序的参数。

![Get-PotentialDLLHijack](upload/attach/202007/747358_6RZHECPG8Z5WWF7.jpg)

![Get-PotentialDLLHijack.ps1](upload/attach/202007/747358_V4GZKEGESJTZ2QG.jpg)

几分钟后，打开“恶意”DLL中指定的文本文件，发现以下针对Slack的DLL劫持：

```
PS C:Users\John\Desktop> Get-PotentialDLLHijack -CSVPath .\Logfile.CSV -MaliciousDLLPath .\DLLHijackTest.dll -ProcessPath "C:\Users\John\AppData\Local\slack\slack.exe"
C:\Users\John\AppData\Local\slack\app-4.6.0\WINSTA.dll
C:\Users\John\AppData\Local\slack\app-4.6.0\LINKINFO.dll
C:\Users\John\AppData\Local\slack\app-4.6.0\ntshrui.dll
C:\Users\John\AppData\Local\slack\app-4.6.0\srvcli.dll
C:\Users\John\AppData\Local\slack\app-4.6.0\cscapi.dll
C:\Users\John\AppData\Local\slack\app-4.6.0\KBDUS.DLL
```

### 案例研究：Microsoft Teams

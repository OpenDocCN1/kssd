
`SeDebugPrivilege`有许多合法使用情景。许多管理工具需要检查其它进程的内存来定位错误或者了解运行状况。同样的，许多商业应用程序基于合法的原因需要`SeDebugPrivilege`以将它们自己的代码注入到系统内运行的进程中。（例如，<a href="elink@61dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6i4m8H3L8%4u0@1i4K6u0W2M7%4W2E0j5h3&6@1k6h3y4Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9#2k6W2g2e0i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5k6g2c8q4b7@1R3I4z5e0j5%4x3K6c8Q4x3X3g2Z5N6r3#2D9">这篇文章</a>解释了赛门铁克终端保护系统如何依赖于`SeDebugPrivilege`特权）

更多关于`SeDebugPrivilege`特权及其在恶意软件中的应用的内容可以在几本书和出版物中找到。本文引用到的包括<a href="elink@d3cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0k6h3#2G2M7Y4W2S2L8X3q4D9P5i4y4A6M7#2)9J5k6h3&6W2N6q4)9J5c8X3q4E0k6R3`.`.">内存取证的艺术</a>(页数：173，186，197-199)，<a href="elink@1f8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2%4K9h3I4W2P5g2)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6y4j5h3I4%4j5i4u0W2i4K6u0n7b7h3&6S2L8s2W2K6N6q4)9J5y4e0t1%4M7#2)9J5b7V1y4G2L8$3E0T1L8$3!0C8i4K6u0n7j5h3&6V1i4K6u0n7c8q4k6p5i4K6t1#2x3@1q4Q4x3V1u0f1L8$3!0D9M7#2)9J5b7X3q4F1k6q4)9J5b7W2c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1u0X3L8%4u0Q4x3V1u0r3K9h3N6Z5N6r3W2F1k6#2)9J5b7V1#2S2L8r3W2U0K9h3!0#2M7#2)9J5b7V1y4G2k6r3g2Q4x3X3c8H3i4K6u0V1z5e0M7^5x3o6b7%4x3o6j5I4x3K6l9K6x3l9`.`.">恶意软件分析手册</a>（页数：58，231，589）和<a href="elink@8d3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2G2M7X3g2A6L8r3I4&6i4K6u0W2j5$3!0E0i4K6u0r3L8r3W2T1M7X3q4J5P5g2)9J5c8Y4k6A6k6i4N6Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0V1L8h3q4D9N6$3q4J5k6g2)9J5k6r3q4F1j5h3I4&6M7$3W2K6i4K6u0r3z5e0M7^5x3e0M7^5y4e0t1^5x3e0f1I4z5q4)9J5c8R3`.`.">Windows恶意软件分析基础</a>（页数：143）。

### 开启特权审计

现在我们来看一看审计技术，搜集识别潜在特权滥用需要的事件。在Palantir，我们使用系统自带的Windows事件转发（Windows Event Forwarding，WEF）来搜集主要部件的审计日志。如果你想使用WEF，请看我们<a href="elink@78bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6N6h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6H3j5h3I4S2L8Y4c8A6M7W2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8W2N6X3g2F1N6q4)9J5k6r3k6G2M7Y4N6S2M7X3c8A6L8X3N6Q4x3X3c8X3L8%4u0Q4x3X3c8F1k6i4c8%4L8%4u0C8i4K6u0V1k6r3g2X3k6h3&6K6k6g2)9J5k6r3y4T1x3U0l9^5k6o6g2X3k6U0R3$3k6W2)9K6c8Y4y4G2N6i4u0U0k6g2)9K6c8s2g2K6k6i4u0Q4y4h3k6H3M7X3!0X3K9h3I4W2i4K6u0V1i4K6u0V1i4K6u0V1i4K6u0V1i4K6u0V1i4K6u0V1i4K6u0V1i4K6u0V1x3e0c8Q4x3X3c8Q4x3X3c8Q4x3X3c8Q4x3X3c8Q4x3X3c8Q4x3X3c8Q4x3X3c8Q4x3X3c8Q4x3X3c8Q4x3X3c8Q4x3X3c8Q4x3X3c8Q4x3X3c8Q4x3X3c8Q4x3X3c8Q4x3X3c8Q4x3X3b7`.">之前的博客</a>和<a href="elink@f00K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3j5h3I4S2L8Y4c8A6M7W2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8W2N6X3g2F1N6q4)9J5k6r3k6G2M7Y4N6S2M7X3c8A6L8X3M7`.">Github仓库</a>中介绍的配置和管理细节。

Windows10和Server 2016中自带的事件记录工具支持对系统内特权使用进行审计。可以使用组策略（Group Policy Object，GPO）开启对<a href="elink@bb7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5c8Y4c8Z5M7X3g2S2N6q4)9J5k6s2m8J5L8%4c8W2j5%4c8A6L8$3&6Q4x3V1k6S2N6h3c8A6N6r3W2F1k6#2)9J5c8X3q4#2k6r3W2@1i4K6u0V1M7$3g2F1M7$3W2@1K9i4k6W2i4K6u0V1M7s2u0A6N6X3W2D9k6h3N6W2i4K6u0V1N6i4y4W2">敏感特权使用</a>和<a href="elink@f77K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5c8Y4c8Z5M7X3g2S2N6q4)9J5k6s2m8J5L8%4c8W2j5%4c8A6L8$3&6Q4x3V1k6S2N6h3c8A6N6r3W2F1k6#2)9J5c8X3q4#2k6r3W2@1i4K6u0V1L8X3!0F1i4K6u0V1M7$3g2F1M7$3W2@1K9i4k6W2i4K6u0V1M7s2u0A6N6X3W2D9k6h3N6W2i4K6u0V1N6i4y4W2">非敏感特权使用</a>的审计并使用WEF子脚本进行搜集。此外，对<a href="elink@8fbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5c8Y4c8Z5M7X3g2S2N6q4)9J5k6s2m8J5L8%4c8W2j5%4c8A6L8$3&6Q4x3V1k6S2N6h3c8A6N6r3W2F1k6#2)9J5c8X3g2$3k6h3&6@1i4K6u0V1y4o6j5%4x3R3`.`.">给登录用户分配特定特权事件</a>进行审计，可以识别在哪里创建了有特权的访问令牌。

在我们遇到的大多数环境中，只搜集与使用敏感特权有关的事件，<a href="elink@232K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4m8J5k6i4k6A6L8%4g2K6i4K6u0V1N6X3g2J5M7$3W2G2L8Y4y4Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3K9i4c8Q4x3X3c8H3M7X3!0Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0V1M7$3g2J5N6X3g2J5i4K6u0V1x3U0l9I4x3W2)9J5k6q4t1J5i4K6u0V1j5h3&6V1i4K6u0V1x3U0l9I4x3W2)9J5c8X3A6B7z5o6f1J5x3U0l9$3i4K6t1#2x3U0S2$3i4K6y4p5N6%4y4Q4x3X3f1I4x3g2)9J5y4e0t1&6">不审计对备份特权和恢复特权的使用</a>。

使用正确的审计GPO，我们搜集对下列特权的使用：

+ 表现为操作系统的一部分的行为
+ 创建令牌对象
+ 调试程序
+ 使计算机和用户对授权可信
+ 安全审计
+ 授权后模拟一个客户
+ 加载、卸载设备驱动
+ 管理审计和安全日志
+ 修改固件环境变量
+ 替换进程令牌
+ 获取对文件或其它对象的拥有权

### 识别特权使用

现在事件日志被搜集到了一个集中的位置，我们现在可以通过搜索来发现可被滥用的特权主体。

我们利用事件代码4672（为新登录用于分配特定特权）搜集事件，通过搜索我们可以发现拥有`SeDebugPrivilege`特权的用户令牌在哪里被创建。一个事件样例是：

```
LogName=Security
SourceName=Microsoft Windows security auditing.
EventCode=4703
EventType=0
Type=Information
ComputerName=dane
TaskCategory=Authorization Policy Change
OpCode=Info
RecordNumber=161204239
Keywords=Audit Success
Message=A user right was adjusted.
Subject:
Security ID:
Account Name:dane
Account Domain:
Logon ID:0x3E7
Target Account:
Security ID:
Account Name:dane
Account Domain:
Logon ID:0x3E7
Process Information:
Process ID:0xa64
Process Name:C:\WINDOWS\System32\WindowsPowerShell\v1.0\powershell.exe
Enabled Privileges:
SeDebugPrivilege
Disabled Privileges:
```

本例中，用户的令牌被修改以获得`SeDebugPrivilege`特权。这意味着在这台机器上生成的用户令牌可能被一个具备系统访问的恶意角色进行了攻击和滥用。

如果<a href="elink@619K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5c8Y4c8Z5M7X3g2S2N6q4)9J5k6s2m8J5L8%4c8W2j5%4c8A6L8$3&6Q4x3V1k6S2N6h3c8A6N6r3W2F1k6#2)9J5c8X3q4#2k6r3W2@1i4K6u0V1j5i4g2@1K9r3!0J5K9i4A6S2N6r3W2G2L8W2)9J5k6s2m8G2L8r3W2U0P5g2)9J5k6r3y4Z5j5h3&6Y4k6b7`.`.">Authorization Policy Change</a>审计开启，我们可以在令牌特权被开启和禁用时收到额外的事件通知。一个样例是4703号事件（一个用户的权限被调整）：

```
LogName=Security
SourceName=Microsoft Windows security auditing.
EventCode=4703
EventType=0
Type=Information
ComputerName=dane
TaskCategory=Authorization Policy Change
OpCode=Info
RecordNumber=161204239
Keywords=Audit Success
Message=A user right was adjusted.
Subject:
Security ID:
Account Name: dane
Account Domain:
Logon ID: 0x3E7
Target Account:
Security ID:
Account Name: dane
Account Domain:
Logon ID: 0x3E7
Process Information:
Process ID: 0xa64
Process Name: C:\WINDOWS\System32\WindowsPowerShell\v1.0\powershell.exe
Enabled Privileges:
SeDebugPrivilege
Disabled Privileges:
-
```

在这个样例中，用户账户令牌被修改以开启`SeDebugPrivilege`。尽管这不是恶意的继承，但仍可被视为使用PowerShell二进制来执行代码注入或对受保护的凭据进行访问的攻击者活动。

最后，事件ID <a href="elink@d15K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5c8Y4c8Z5M7X3g2S2N6q4)9J5k6s2m8J5L8%4c8W2j5%4c8A6L8$3&6Q4x3V1k6S2N6h3c8A6N6r3W2F1k6#2)9J5c8X3g2$3k6h3&6@1i4K6u0V1y4o6j5%4x3H3`.`.">4673</a>（特权服务被调用）和<a href="elink@311K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5c8Y4c8Z5M7X3g2S2N6q4)9J5k6s2m8J5L8%4c8W2j5%4c8A6L8$3&6Q4x3V1k6S2N6h3c8A6N6r3W2F1k6#2)9J5c8X3g2$3k6h3&6@1i4K6u0V1y4o6j5%4y4l9`.`.">4674</a>（尝试对一个特权对象执行一个操作）结合将包含附加内容或其它特权调用。一个4673事件的样例是：

```
LogName=Security
SourceName=Microsoft Windows security auditing.
EventCode=4673
EventType=0
Type=Information
ComputerName=dane
TaskCategory=Sensitive Privilege Use
OpCode=Info
RecordNumber=93434404
Keywords=Audit Failure
Message=A privileged service was called.
Subject:
Security ID:
Account Name:dane
Account Domain:
Logon ID:0xADF23180D
Service:
Server:Security
Service Name:-
Process:
Process ID:0xf818
Process Name: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
Service Request Information:
Privileges:SeTcbPrivilege
```

本例中，一个普通用户通过PowerShell使用了`SeTcbPrivilege`特权。攻击者可以滥用`SeTcbPrivilege`来产生一个包含新特权或新特征的新令牌，然后通过模拟令牌使用它。

### 对整个组织解除特权

接下来，我们分析一下`SeDebugPrivilege`事件日志并验证它们能够被安全解除，我们只需解除操作以确保只有需要该特权的用户拥有它。

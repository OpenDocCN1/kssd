
```
C:\> copy powershell.exe tLclgEomOrR.exe
C:\> tLclgEomOrR.exe –exec bypass Get-Help
```

也可以用宏脚本实现以上功能：

```
o = CreateObject("Scripting.FileSystemObject")
o.CopyFile(source, destination)
```

- 规则0x3：添加检测环境代码，如果条件不匹配则不执行，以防止执行最后阶段。

例如：ClickOnce应用程序检查`iexplore`是否正在运行，因为需要Internet Explorer来下载ClickOnce。

```
If(Process.GetProcessByName("iexplore").Length > 0) {
    // be evil
}
```

项目代码：`24dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4M7W2)9J5k6q4g2F1x3h3D9H3k6o6y4J5i4K6u0r3b7$3I4A6j5$3E0a6L8X3y4W2c8$3g2F1k6i4u0S2N6r3!0J5`

- 规则0x4：有人可能已经写了一个工具来混淆Payload。

  - SCT COM Scriptlet: `e0dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4M7W2)9J5k6q4g2F1x3h3D9H3k6o6y4J5i4K6u0r3f1@1y4f1i4K6u0V1L8$3u0X3N6i4y4U0j5i4c8G2M7R3`.`.
  - EXE (shellcode): `2fdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4M7W2)9J5k6q4g2F1x3h3D9H3k6o6y4J5i4K6u0r3g2h3&6A6b7Y4W2m8N6R3`.`.
  - EXE (shellcode): `7bdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4M7W2)9J5k6q4g2F1x3h3D9H3k6o6y4J5i4K6u0r3c8p5E0y4b7H3`.`.
  - Base64 (`PowerShell`): `d4bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4M7W2)9J5k6q4g2F1x3h3D9H3k6o6y4J5i4K6u0r3b7X3q4K6k6e0j5@1i4K6u0V1e0$3u0X3N6i4y4U0j5i4c8G2M7R3`.`.

沙箱解决方案的问题:它们可以指纹识别和预测

端点(工作站/服务器)和沙箱之间的差异:

- 内存大小(端点至少4 Gb)
- 磁盘大小(端点至少250gb)
- CPU数量(端点至少2个cpu)
- 正在运行的进程(如果攻击者是通过电子邮件发送样本，那么`OUTLOOK.exe`这个会显示正在运行的状态)

端点之间的差异(未完待续):

- 网络访问(沙箱有网络接入)
- 加入域(沙箱通常不加入到公司域)
- 时区(针对一家加拿大公司)
- 检测钩子(沙箱通常HOOK已知的api函数)
- 系统已经运行了多长时间
- 系统活动(剪贴板不为空，接收广播流量等)
- 和更多…(创新)

- 规则0x5：尽可能隐秘地连接到攻击者的C2

  - "域前置"(Domain Fronting)
  - "分类域名"(Categorized domains)

  **注：**字面意思是分类过的域名，实际意思是指那些已经过期了但是仍然被安全厂商标记过的那些高信用域名。

  举个例子 `foo.com` 之前被标记为论坛网站，信誉度是9，那么很多公司的web或者邮件安全网关就不会过滤或者拦截他，现在这个域名过期了且被黑客重新注册来做C2域名，这样就可以达到隐藏的目的了。

  这里有些工具这可以找到类似的这些域名：

  1. `aa4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4M7W2)9J5k6q4g2F1x3h3D9H3k6o6y4J5i4K6u0r3b7$3q4@1e0i4W2r3K9i4y4Z5`
  2. `ce4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1K9s2u0W2j5i4c8W2P5s2m8J5k6i4y4K6i4K6u0r3k6r3!0E0j5h3W2F1K9s2g2F1N6r3g2J5`

  - 强制执行HTTPS
  - 选择正确的协议:现在大多数RAT使用HTTP来混入“合法”的流量。

相关项目：`f57K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4M7W2)9J5k6q4g2F1x3h3D9H3k6o6y4J5i4K6u0r3g2r3S2#2L8X3c8W2M7W2y4Z5k6h3I4D9`

基于HTTP协议的RAT支持HTTPS，在HTTPS之上使用RC4加密来击败端点网络检测。

没有第二阶段(DLL)，`PowerShell`脚本会提供对所有基本功能的访问

![](https://bbs.kanxue.com/upload/attach/201806/214974_K3FW9R86KRDPRXC.png)

请随意为这个项目做出贡献，大家可以尝试为这个项目添加更多的特性和一个WEB界面来管理会话。

**选择合适的载荷:**

- 宏:Office 2016默认禁用宏
- HTA:因为流行，所以更容易被发现
- ClickOnce:需要使用Internet Explorer
- 普通EXE:可能会被应用程序屏蔽。

避免直接运行`PowerShell`，因为它是现在很流行的方法:

宏-> WMI -> `PowerShell` VS 宏-> WMI -> `PowerLessShell`(`MSBuild`)

到这里，攻击者已经精心设计了网络钓鱼活动中需要的作战方案，Payload也将会是安全产品所信任的程序。

### 5、狩猎

假设攻击者已经有一个SHELL。那就需要抓住尽可能多的信息，以免丢失SHELL。

- username
- Email(枚举到的)

注意以下3点：

1. 避免直接运行`PowerShell`
2. 避免使用`net *`家族命令
3. 避免连接到所有系统。

**解决方案**

1. 非托管的`PowerShell` + LDAP查询
2. CobaltStrike 内置`powerpick`命令
3. ThunderShell 默认支持

- `Dump-UserEmail`：`979K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4M7W2)9J5k6q4g2F1x3h3D9H3k6o6y4J5i4K6u0r3f1X3g2V1g2r3g2S2L8g2m8G2N6$3g2J5M7$3S2W2L8r3I4e0j5%4u0A6M7s2c8K6i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4y4U0M7X3W2H3N6s2y4Q4x3V1k6g2N6r3W2D9K9i4c8&6i4K6u0W2M7s2x3I4`

![](https://bbs.kanxue.com/upload/attach/201806/214974_KWUW7EGJTP5CE9U.png)

- 域密码破解

这并不一定是最隐秘的方法，从提取的用户列表暴力破解用户的密码。

`e02K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4M7W2)9J5k6q4g2F1x3h3D9H3k6o6y4J5i4K6u0r3f1X3g2V1g2r3g2S2L8g2m8G2N6$3g2J5M7$3S2W2L8r3I4e0j5%4u0A6M7s2c8K6i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4y4U0M7X3W2H3N6s2y4Q4x3V1k6u0L8Y4k6G2K9$3g2Q4x3X3c8m8c8q4m8S2M7%4y4%4L8%4u0V1b7Y4u0#2N6r3g2r3L8%4u0U0k6g2)9J5k6i4m8K6x3b7`.`.

```
"neo","morpheus" | Invoke-ADPasswordBruteForce -Password "password" -Domain MATRIX
```

这个cmdlet支持其他域，甚至可以对其他受信任域执行穷举破解。

穷举破解依赖于`ValidateCredentials()`方法，它很明显地连接到DC，特别是如果试图强制所有用户使用穷举破解时。

- 查找基于名称特定用户的SamAccountName

`fd4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4M7W2)9J5k6q4g2F1x3h3D9H3k6o6y4J5i4K6u0r3f1X3g2V1g2r3g2S2L8g2m8G2N6$3g2J5M7$3S2W2L8r3I4e0j5%4u0A6M7s2c8K6i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4y4U0M7X3W2H3N6s2y4Q4x3V1k6e0k6h3q4J5j5$3S2Q4x3X3c8r3N6h3I4D9e0X3q4E0k6g2c8G2f1$3q4E0b7h3y4U0L8%4g2F1N6q4)9J5k6i4m8K6x3b7`.`.

```
Search-FullNameToSamAccount -Filter Hamilton
```

![](https://bbs.kanxue.com/upload/attach/201806/214974_RZ8XW6C543KKJXJ.png)

- 搜索当前用户的计算机

这一项需要提升权限，搜索当前用户的计算机。

`06dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4M7W2)9J5k6q4g2F1x3h3D9H3k6o6y4J5i4K6u0r3f1X3g2V1g2r3g2S2L8g2m8G2N6$3g2J5M7$3S2W2L8r3I4e0j5%4u0A6M7s2c8K6i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4y4U0M7X3W2H3N6s2y4Q4x3V1k6e0k6h3q4J5j5$3S2Q4x3X3c8q4N6X3g2F1N6p5k6G2M7W2g2K6k6i4u0Q4x3X3g2H3M7K6p5`

```
Search-EventForUser -TargetUser charles.hamilton -FindDC true
```

在DC的所有事件日志中搜索登录事件

- 获取浏览器书签以发现感兴趣的内部资产

`d9dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4M7W2)9J5k6q4g2F1x3h3D9H3k6o6y4J5i4K6u0r3f1X3g2V1g2r3g2S2L8g2m8G2N6$3g2J5M7$3S2W2L8r3I4e0j5%4u0A6M7s2c8K6i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4y4U0M7X3W2H3N6s2y4Q4x3V1k6s2k6i4c8Q4x3X3c8u0c8f1u0G2L8$3E0E0j5i4u0C8M7#2)9J5k6i4m8K6x3b7`.`.

```
Get-IEBookmarks
```

### 6、工具和技巧

WDIGEST没有返回任何内容，但是当前用户在另一个系统上具有本地管理权限。

- 使用Kerberos票据利用WMI远程连接

`1dbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4M7W2)9J5k6q4g2F1x3h3D9H3k6o6y4J5i4K6u0r3f1X3g2V1g2r3g2S2L8g2m8G2N6$3g2J5M7$3S2W2L8r3I4e0j5%4u0A6M7s2c8K6i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4y4U0M7X3W2H3N6s2y4Q4x3V1k6d9k6h3#2G2N6r3g2Q4x3X3c8i4L8h3W2q4P5r3g2U0N6i4c8W2i4K6u0W2M7s2x3I4`

```
Remote-WmiExecute -ComputerName victim01 -Payload "cmd.exe /c whoami"
```

谈到Kerberos票据——为什么有时会拒绝访问?

`12dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4c8W2j5$3S2F1k6i4c8Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5k6s2g2K6i4K6u0r3L8r3W2T1M7X3q4J5P5g2)9J5c8X3y4U0y4K6M7J5z5o6p5#2i4K6t1^5g2#2y4Q4x3X3f1I4x3q4)9J5z5g2)9J5k6h3q4K6M7s2R3`

![](https://bbs.kanxue.com/upload/attach/201806/214974_Q6RH8BSDEGMQ9RM.png)

可持续票据的客户端必须在到达结束时间之前将其发送给KDC进行续约。

如果使用WMI生成shell，则默认情况下不会将故障单发送到KDC。还将会更新进程，如`explorer.exe`是一个很好的目标可以将Payload注入并运行域查询。

然而，从OPSEC的角度来看，有一个缺点；`Explorer.exe`通常不建立网络连接。

更隐蔽的目标可能是：

```
svchost.exe
conhost.exe
```

- 活动目录(Active Directory)包含很多有价值的信息，如枚举用户的评论和描述可能会泄露密码和其他有趣的信息。

`00cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4M7W2)9J5k6q4g2F1x3h3D9H3k6o6y4J5i4K6u0r3f1X3g2V1g2r3g2S2L8g2m8G2N6$3g2J5M7$3S2W2L8r3I4e0j5%4u0A6M7s2c8K6i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4y4U0M7X3W2H3N6s2y4Q4x3V1k6g2N6r3W2D9K9i4c8&6i4K6u0W2M7s2x3I4`

```
Dump-Username -More
```

很多时候不需要“域管理员”权限来实现预定义的目标。但是一个红队在时间和预算上都是有限的，为了完成任务目标不得不做出冒险的行为。

大多数Windows命令都可以通过`PowerShell`运行，为了避免产生一个`CMD.EXE`实例，可以通过以下两类程序使用非托管`PowerShell`来运行命令：

- CobaltStrike `powerpick`
- `PowerLessShell`

**高效的技巧**

`PowerView(47cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6b7L8%4N6W2M7W2y4Z5k6h3I4D9e0h3q4X3K9h3q4Q4x3V1k6b7L8%4N6W2M7W2y4H3L8r3!0A6N6q4)9J5z5g2!0q4y4W2)9&6b7#2)9^5z5g2!0q4y4g2!0n7c8g2)9^5z5q4!0q4y4g2!0m8y4q4)9&6b7g2!0q4z5g2)9&6c8q4)9&6c8g2!0q4y4g2!0n7z5q4!0n7z5q4!0q4y4W2)9&6b7#2)9^5z5g2!0q4y4#2)9&6y4q4!0m8z5q4!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4g2)9&6x3g2!0n7c8q4!0q4y4q4!0n7b7W2!0m8y4q4!0q4x3#2)9^5x3q4)9^5x3R3`.`.)

```
Find-LocalAdminAccess: 查找主机具有本地管理员权限的用户。

Get-NetDomainTrust: 列出所有信任域

Get-NetForestTrust: Lists all forests

Invoke-ShareFinder: 列出所有共享

Get-NetLocalGroup: 列出本地管理组/用户组
```

即使攻击者试图尽可能隐蔽攻击手段，但也不可能因为红队的性质而毫无动静。当合适的时候攻击者可以调整工具和技术以保持尽可能的隐蔽。

一次好的网络钓鱼活动每次都会不同，而制作Payload是一门艺术需要一步步策划。

再次强调!!!

尽量避免直接运行`PowerShell`。

### 7、参考

- 2018年红队之旅——介绍各种脚本的TIPS

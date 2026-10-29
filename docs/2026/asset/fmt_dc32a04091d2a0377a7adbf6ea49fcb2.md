
2018年1月24日，Benjamin Delpy和Vincent Le Toux在以色列的微软`BlueHat`上发布了一个新的`Mimikatz`模块，实现了`DCShadow`攻击。这种攻击的思想是创建一个流氓域控制器来复制`AD`中的对象。研究人员定义了成功复制所需的至少一组`Kerberos` `SPNs`，只需要两个`SPNs`。他们还展示了一个可以强制复制控制器的特殊功能。作者保证这种攻击会使你的`SIEM`人失明。`rogue`域控制器不会将事件发送给`SIEM`，这意味着攻击者可以使用`AD`和`SIEM`进行各种恶意的操作，没有人会知道这一点。

## 攻击方法

![](https://bbs.kanxue.com/upload/attach/201811/759685_3NQH3X9N378ZH8D.png)

应该向运行攻击的系统中添加两个`spn`。这些`spn`是其他域控制器使用`Kerberos`进行复制身份验证所必需的。因为根据规范域控制器被表示为`AD`库中的类`nTDSDSA`的对象，所以应该创建这样的对象。最后，使用`DRSReplicaAdd`函数触发复制。

## 如何检测

这就是`DCShadow`流量的样子。通过分析流量，我们可以清楚地看到，域控制器配置方案中添加了一个新对象，然后触发复制。

![](https://bbs.kanxue.com/upload/attach/201811/759685_DHHK387PAKJS5JU.png)

虽然攻击创造者说`SIEM`不会帮助它检测，但是我们找到了一种方法来通知安全部门网络上可疑的活动。

我们的关联有一个合法域控制器的列表，它将在域控制器的每次复制时触发，域控制器不包括在这个白名单中。因此，安全部门可以进行调查，以检查这是`IT`服务添加的合法域控制器，还是`DCShadow`攻击。

`DCShadow`的一个例子证实了新的企业攻击向量的出现。它是至关重要的

我们`PT` Expert Security Center每天都在研究新的威胁，并开发方法和工具来检测它们。我们会继续和你们分享这些信息

作者： Anton Tyurin, Head of Attack Detection Team, Positive Technologies

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

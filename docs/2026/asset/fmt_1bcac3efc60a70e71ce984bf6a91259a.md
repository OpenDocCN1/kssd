
# DCShadow

应该向运行攻击的系统中添加两个spn。这些spn是其他域控制器使用Kerberos进行复制身份验证所必需的。因为根据规范域控制器被表示为AD库中的类`nTDSDSA`的对象，所以应该创建这样的对象。最后，使用`DRSReplicaAdd`函数触发复制。

## 如何检测

这就是DCShadow流量的样子。通过分析流量，我们可以清楚地看到，域控制器配置方案中添加了一个新对象，然后触发复制。

![](upload/attach/201811/759685_DHHK387PAKJS5JU.png)

虽然攻击创造者说SIEM不会帮助它检测，但是我们找到了一种方法来通知安全部门网络上可疑的活动。

我们的关联有一个合法域控制器的列表，它将在域控制器的每次复制时触发，域控制器不包括在这个白名单中。因此，安全部门可以进行调查，以检查这是IT服务添加的合法域控制器，还是DCShadow攻击。

DCShadow的一个例子证实了新的企业攻击向量的出现。它是至关重要的

我们PT Expert Security Center每天都在研究新的威胁，并开发方法和工具来检测它们。我们会继续和你们分享这些信息

作者： Anton Tyurin, Head of Attack Detection Team, Positive Technologies

原文链接：5d4K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2H3N6s2y4W2j5%4g2J5K9i4c8&6i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4z5q4)9J5c8U0p5H3i4K6u0r3j5h3c8$3j5h3&6U0k6h3c8Q4x3X3c8S2N6s2c8S2j5$3E0K6i4K6u0V1L8$3&6Q4x3X3c8E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3c8S2j5%4c8A6N6X3g2Q4x3X3g2Z5N6r3#2D9

在过去的四年里，针对微软Active Directory的攻击一直是Blank Hat和Defcon报告中反复出现的话题。演讲者公开他们的新的攻击向量，分享了他们的观点，并提出了检测和避免这类攻击的建议。我相信IT部门能够创建一个由安全部门监管安全的基础设施。高质量的监管反过来也需要好的工具。这就像一个军事基地:你已经在周边建立了警戒塔，但仍然在该地区保持警戒。

# 六个不容忽视的策略

许多供应商提供安全软件，以支持监测下列恶意活动:

## 通过HASH

这种攻击受到NTLM体系结构的制约，NTLM是微软在1990年代创建的一种身份验证协议。登录到远程主机需要存储在用于身份验证过程计算机上的密码散列。因此，可以从计算机中提取哈希值。

## Mimikatz

为了实现这一点，法国研究员本杰明·德尔比(Benjamin Delpy)在2014年开发了Mimikatz，这是一种允许从计算机内存中提取明文密码和NTLM哈希的工具。

## net user /domain

我们应该如何使用用户名字典来进行这种攻击?任何域成员都可以执行返回AD域用户完整列表的`net user /domain`命令。

## Kerberoasting

如果一个域使用Kerberos作为身份验证协议，攻击者可以尝试使用Kerberoasting攻击。域上经过身份验证的任何用户都可以请求Kerberos票据访问服务(票据授予服务)。TGS被用于运行服务帐户的密码散列加密。请求TGS的攻击者现在可以强制它离线，而不必担心被阻止。如果成功的话，攻击者将获得与服务关联的帐户的密码，通常是特权帐户。

## PsExec

一旦攻击者获得所需的凭证，下一个任务就是远程命令执行。使用Sysinternals套件中的PsExec程序可以很容易地解决此任务，这在实践中被证明是非常有效的，并且得到了IT管理员和黑客的赞赏。

# 黑客的七板斧

现在我们将回顾七种黑客招式，它们可以帮助我们完全控制多动目录。

[图中显示了攻击的四个步骤。每个步骤都有一组方法]

![](https://bbs.kanxue.com/upload/attach/201811/759685_45XV8AWVWHBN4UY.png)

## PowerView

PowerView是[PowerSploit](https://bbs.kanxue.com/elink@174K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6b7L8%4N6W2M7W2y4Z5k6h3I4D9e0h3q4X3K9h3q4Q4x3V1k6b7L8%4N6W2M7W2y4H3L8r3!0A6N6l9`.`.)的一部分，PowerSploit是一个用于渗透测试的著名PowerShell框架。PowerView支持[Bloodhound](https://bbs.kanxue.com/elink@3ffK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6n7L8r3!0G2k6p5S2G2N6h3&6V1b7f1c8Q4x3V1k6n7L8r3!0G2k6p5S2G2N6h3&6V1)，这是一种在AD中提供对象连接图形表示的工具。

![](https://bbs.kanxue.com/upload/attach/201811/759685_GNCH3VM5HVVPSU6.png)

Bloodhound活动目录对象之间关系的图形表示立即提供了这样的可能性：

1. 查找所有域管理员的帐户
2. 查找主机，其中记录了域管理员
3. 使用域管理会话找到从攻击者到目标主机的最短路径

三种可能性中的最后一种可能是回答这个问题:黑客需要攻击哪些主机才能访问域管理帐户。这种方法大大减少了获得对域的完全控制所需的时间。

PowerView与允许获取AD对象(比如`net.exe`)数据的内置实用程序的区别在于，PowerView使用的是LDAP，而不是SAMR。要检测此活动，我们可以使用域控制器事件1644。通过在寄存器中添加相关值，可以启用此事件的日志记录:

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NTDS\Diagnostic\15 Field Engineering=5
```

![](https://bbs.kanxue.com/upload/attach/201811/759685_C7FY9397NCDJ2F2.png)

启用LDAP事件1644的日志记录

![](https://bbs.kanxue.com/upload/attach/201811/759685_AFNH5VNJRDKD3NG.png)

具有LDAP查询属性的事件1644

注意，这种类型的事件可能有多个，而对事件分析的一个很好的替代方法是进行流量分析，因为LDAP是一个明文协议，并且在流量中可以清楚地看到所有查询。

![](https://bbs.kanxue.com/upload/attach/201811/759685_W8FBMB2DW67RGCE.png)

框架的另一个特性是，由于它只使用PowerShell，没有任何依赖项。此外，PowerShell v5还有一个高级审计的新选项，这在检测中非常有用。事件4104显示了脚本主体，可以搜索于PowerView中的特定函数名。

![](https://bbs.kanxue.com/upload/attach/201811/759685_4HEN6C4A3TSXAV7.png)

## SPN 扫描

这个程序可以替代Nmap。一旦黑客知道AD中存在哪些用户和群组，他或她就需要相关服务的信息来了解整个布局。

![](https://bbs.kanxue.com/upload/attach/201811/759685_U847KV7YPG7U7AU.png)

通常，使用Nmap扫描端口可以获取这些信息。但是现在这些数据可以从AD中获取——它们已经存储在那里了。查询结果如下:应答返回SPN(服务主体名称)，它由服务类组成，服务类对于每种服务类型都是唯一的，FQDN格式的主机名，以及一些服务的端口。

![](https://bbs.kanxue.com/upload/attach/201811/759685_BEPZSX2AET2A8MD.png)

有关SPNs的完整列表，请参阅343K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2k6s2y4W2j5%4g2J5K9i4c8&6i4K6u0W2L8%4u0Y4i4K6u0r3i4K6y4r3M7r3q4Y4k6g2)9#2k6X3W2V1i4K6y4p5x3e0R3K6i4@1f1K6i4K6R3H3i4K6R3J5

要检测SPN扫描，可以使用LDAP事件审计。

SPN扫描相比Nmap有一个明显的优势:它的噪声更小。当使用Nmap时，需要连接到每个主机并将数据包发送到您指定的端口范围，而SPN列表仅是一个查询的结果。

## 远程会话枚举

期间被称为横向运动的阶段,一个重要任务是匹配账户以及他们登录的电脑:攻击者要么已经有用户凭证(散列或Kerberos票据)然后搜索主机以准确登录,或者搜索与域管理员进行会话的主机。

这两种情况都会引发以下场景:hunt -> compromise any host -> upload Mimikatz -> profit.

要检测此场景的使用，可以使用两个事件:4624成功登录到远程系统(登录类型3)，和访问网络的事件共享`IPC$`，一个细微差别是，命名管道`srvsvc`。从通信统计中可以猜出为什么管道会这样命名。

![](https://bbs.kanxue.com/upload/attach/201811/759685_ZRVWR5UB5PH5UWQ.png)

左边的红色方框显示SMB连接，然后连接到管道`srvsvc`。此管道允许通过服务器服务远程协议进行交互。终端主机从管道接收各种管理信息;例如，其中一个请求是`NetSessEnum`。此请求返回登录到远程系统的完整用户列表及其IP地址和名称。

![](https://bbs.kanxue.com/upload/attach/201811/759685_KQFMEY9PANAKM8C.png)

[MaxPatrol](https://bbs.kanxue.com/elink@32cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2H3N6s2y4W2j5%4g2J5K9i4c8&6i4K6u0W2j5$3!0E0i4K6u0r3N6%4N6Q4x3X3c8W2L8W2)9J5c8Y4m8J5L8$3c8#2j5%4c8K6i4K6u0r3L8h3q4^5M7r3q4@1M7X3!0D9i4K6u0r3) SIEM允许基于这两个事件与`srvsvc`的相关性进行检测。PT网络攻击发现基于流量分析进行类似检测。

## Overpass-the-Hash

Pass-the-Hash派生方法。让我们继续在域中横向运动。攻击者可以用手上的NTLM散列做什么?进行散列攻击。但它已经是众所周知的，可以被检测到。因此，我们找到了一种新的攻击：Overpass-the-Hash。

Kerberos协议是专门为防止通过网络以任何形式发送用户密码而开发的。为了避免这种情况，用户在自己的计算机上使用密码哈希加密身份验证请求。一个密钥分发中心(在域控制器上运行的一种特殊服务)用一张票据来回复以获得其他票据，即所谓的授票票(tickett-ticket, TGT)。现在客户端被认为是经过身份验证的，可以在接下来的10小时内访问其他服务。因此，如果攻击者转储目标服务受信任组(例如，ERP系统或数据库)的用户散列，则攻击者可以自己提供合法票据并成功登录到目标服务。

![](https://bbs.kanxue.com/upload/attach/201811/759685_6YZ3R96RY8WDJ79.png)

### 如何检测

如果黑客使用PowerShell版本的Mimikatz进行攻击，日志脚本主体将会有帮助，因为`Invoke-Mimikatz`是一个很标志性的行。

![](https://bbs.kanxue.com/upload/attach/201811/759685_HZ7D2UNH7FVS656.png)

另一个标志是事件4688--创建带有命令行扩展审计的流程。即使将二进制文件重命名，命令行也会包含该命令，这对于Mimikatz来说是非常特殊的特征。

![](https://bbs.kanxue.com/upload/attach/201811/759685_KPCPU6KMPS7DNUT.png)

如果您想通过流量分析来检测Overpass-the-Hash攻击，可以使用以下异常特点:Microsoft建议在如今的域中使用AES256加密身份验证请求，如果Mimikatz发送身份验证请求数据，则使用过时的ARCFOUR加密数据。

![](https://bbs.kanxue.com/upload/attach/201811/759685_U9F2SCEQK83GBGA.png)

另一个特别的特征是Mimikatz发送的密码套件，它与合法域的套件不同，因此在流量中很突出。

# Golden Ticket

一个广为人知的方法。

攻击者从名为`krbtgt`的特殊帐户的密码散列中可以得到什么?之前，我们回顾了一个用户可能没有特权的案例。现在，我们来回顾令一个案例，在这个案例中，用户的密码散列用于签名所有的票据，以获得其他票据(TGT)。不需要告知密钥分发中心:攻击者可以生成此票据，因为黄金票据实际上是TGT。然后攻击者可以不受时间限制地在AD中的任何服务上发送身份验证请求。结果是不受限制地访问目标资源，Golden Ticket这个名字是有原因的。

![](https://bbs.kanxue.com/upload/attach/201811/759685_Y2RZ2ABNYF9ZP7J.png)

## How to detect based on events

事件4768意味着授予了一个TGT，事件4769意味着授予了AD中某个服务身份验证所需的服务票据。

![](https://bbs.kanxue.com/upload/attach/201811/759685_DE254UVS35J2WBW.png)

在这种情况下，我们可以看到一些不同之处:虽然Golden Ticket并不从域控制器请求TGT(因为它生成了TGT本身)，但它必须请求TGS。因此，如果我们看到获得的TGT和TGS不同，我们可以假设Golden Ticket攻击正在进行。

MaxPatrol SIEM使用表列表来记录所有发布的TGTs和TGSs，从而实现了这种检测方法。

# WMI远程执行

在目标主机上经过身份验证和授权后，黑客可以开始远程执行任务。WMI是一种内置的机制，非常适合这一目的。在过去的几年里，无文件化是一种趋势，这意味着使用内置的Windows机制。主要原因是模仿合法活动的能力。

下图展示了`wmic`的使用，这是一个内置的实用程序。该实用程序有一个主机地址，用于连接、凭证、进程调用`create`操作符和在远程主机上执行的命令。

![](https://bbs.kanxue.com/upload/attach/201811/759685_HGKQ9774EV8QMPQ.png)

## 如何检测

检查远程登录事件4624的组合。一个重要的参数是登录ID。还要检查事件4688，它通知使用命令行创建进程。事件4688显示创建的进程的父进程是`WmiPrvSE.exe`，用于远程管理的一种特殊的wmi服务流程。我们可以看到我们发送的命令`net user /add`，以及登录ID，与事件4624相同。因此，我们完全可以精确地知晓哪个主机启动了这个命令。

![](https://bbs.kanxue.com/upload/attach/201811/759685_W7J5CA2MDBZQNHJ.png)

## 基于流量分析的检测

我们可以清楚地看到Win32进程创建的典型词语和将要执行的命令行。下图中的恶意软件以与WannaCry相同的方式在虚拟网络上发布，但它没有设置加密，而是设置了一个加密挖掘器。这款恶意软件使用了Mimikatz和EthernalBlue，抓取到的账户，并使用它们登录到可以在网络上访问的主机。使用WMI，恶意软件在这些主机上运行PowerShell，并下载了PowerShell载荷，其中还包含Mimikatz、EthernalBlue和一个矿工。这样就产生了连锁反应。

![](https://bbs.kanxue.com/upload/attach/201811/759685_QXEJYBSEQ4MU339.png)

推荐做法：

1. 对服务帐户使用复杂和长的密码(至少25个符号)。攻击者没有机会进行kerberos攻击，因为bruteforce这样的密码要花费太多时间
2. 启用PowerShell日志记录。这将有助于揭示各种现代工具对AD的攻击
3. 升级到Windows 10和Windows Server 2016。Microsoft创建了凭据保护:它可以防止NTLM散列和Kerberos票据被提取。
4. 实现基于角色的访问控制。将AD、DC、服务器和工作站管理员的权限分配给一个角色是有风险的。
5. 每年两次重置`krbtgt`(用于签名TGT的账户)密码，每次都由AD管理员更改。更改密码两次是很重要的，因为当前密码和以前的密码都被存储了。即使一个网络被入侵，攻击者发布了Golden Ticket，更改密码也会使该票据失效，他们不得不再次强制输入密码。
6. 使用带有持续更新的专家数据库的保护工具。这有助于揭示当前正在进行的攻击。


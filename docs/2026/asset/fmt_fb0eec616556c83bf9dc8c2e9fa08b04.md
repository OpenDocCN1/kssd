
通过[Onionshare团队](https://bbs.kanxue.com/elink@ac5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0K9h3y4S2K9r3k6D9k6h3g2Q4x3V1k6G2L8X3W2G2L8Y4y4Z5j5i4u0W2i4K6u0r3N6$3W2C8K9g2)9J5c8V1c8W2N6X3g2D9L8%4m8A6L8X3N6Q4x3X3c8a6L8X3W2G2L8W2y4Z5j5i4u0W2)开发，设计，翻译近一年的工作，OnionShare 2正式发布，可以在[onionshare.org](https://bbs.kanxue.com/elink@3c9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6G2L8X3W2G2L8Y4y4Z5j5i4u0W2i4K6u0W2L8%4u0Y4i4K6u0r3)下载.

OnionShare是使用Tor进行匿名文件分享的开源工具。可以直接通过在电脑上启动web服务器，用OnionShare生成一个可访问的复杂的Tor地址，对方将此地址在[Tor浏览器](https://bbs.kanxue.com/elink@534K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1L8%4u0H3M7X3!0B7k6h3y4@1i4K6u0W2L8%4u0Y4i4K6u0r3)中加载，用来上传或者下载文件。此过程不需要设置单独的服务器，使用第三方文件共享服务，甚至不需要登录帐户。

与电子邮件，Google Drive, DropBox, WeTransfer等通常的文件共享服务不同，使用OnionShare可以防止这些文件共享服务公司访问你共享的文件，只要你以安全的方式分享这个复杂的Tor地址，比如说通过一个加密消息传递应用。除了你和你共享文件的对方，没有别人可以访问你的文件。

下面介绍OnionShare 2的一些新部分。

目录

- [使用OnionShare分享文件](https://bbs.kanxue.com/#使用onionshare分享文件)
- [MacOS沙盒](https://bbs.kanxue.com/#macos沙盒)
- [下一代洋葱服务](https://bbs.kanxue.com/#下一代洋葱服务)
- [仅共享一个文件，OnionShare不会压缩](https://bbs.kanxue.com/#仅共享一个文件，onionshare不会压缩)
- [OnionShare文件地址默认只能使用一次](https://bbs.kanxue.com/#onionshare文件地址默认只能使用一次)
- [接收文件](https://bbs.kanxue.com/#接收文件)
- [只打开信任的文件](https://bbs.kanxue.com/#只打开信任的文件)
- [公共模式](https://bbs.kanxue.com/#公共模式)
- [匿名接收文件](https://bbs.kanxue.com/#匿名接收文件)
- [丰富的语言](https://bbs.kanxue.com/#丰富的语言)

## 使用OnionShare分享文件

第一次打开OnionShare 2时，会连接到洋葱网络.

![img](https://bbs.kanxue.com/upload/attach/202001/665727_QDVNST5R8Q4T2NW.jpg)

连上后是“共享文件”选项卡（稍后将详细介绍新的“接收文件”选项卡）。

![img](https://bbs.kanxue.com/upload/attach/202001/665727_XY3NZC7984CZ38H.jpg)

要安全地匿名与他人共享文件，只需将文件拖放到OnionShare窗口即可。 或者单击“添加文件”和“添加文件夹”按钮来浏览本地文件。

## MacOS沙盒

在Windows和Linux中，只有一个“添加”按钮，可以用来选择文件和文件夹。MacOS下按钮的复杂化，是因为它新增了MacOS沙盒这一安全功能。这意味着即使有人设法利用OnionShare中的漏洞尝试破解你的计算机，他们无法在没有逃离沙箱的情况下访问计算机上的数据或运行程序。

![img](https://bbs.kanxue.com/upload/attach/202001/665727_J238R4F3AHNS9ZX.jpg)

如上图所示，我将OnionShare源代码的所有文件拖到了窗口中，为了让其他人可以访问它们，我只需要点击“开始共享”。

## 下一代洋葱服务

![img](https://bbs.kanxue.com/upload/attach/202001/665727_QQPS44UJVJPK23J.jpg)

点击按钮几秒后，OnionShare给了一个复杂的Tor地址用来分享。这也是OnionShare 2中的新功能：默认情况下，它使用[下一代Tor洋葱服务](https://bbs.kanxue.com/elink@a34K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2N6r3!0J5M7s2u0G2K9X3g2U0N6q4)9J5k6h3!0J5k6#2)9J5c8Y4c8G2M7Y4y4Q4x3X3c8X3j5h3I4D9i4K6u0V1K9r3q4J5N6X3g2K6N6q4)9J5k6r3&6W2P5s2c8Q4x3X3c8Y4k6h3&6W2M7X3q4@1K9h3!0F1i4K6u0V1L8$3&6A6L8$3&6Q4x3X3c8K6k6i4u0$3K9h3y4W2M7H3`.`.)，也称为`v3`洋葱地址。 这些洋葱地址看起来像`lldan5gahapx5k7iafb3s4ikijc4ni7gx5iywdflkba5y2ezyg6sjgyd.onion`，而不是旧的`v2`形式，看起来像`elx57ue5uyfplgva.onion`。

由于各种原因，`v3`洋葱比`v2`洋葱更安全，但这个地址冗长且难以打出来（如果你使用OnionShare在你面前的两台计算机之间移动你的文件）。如果需要，你可以进入设置并选择“使用旧地址”来使用`v2`洋葱地址。

此外，你可能会注意到OnionShare地址使用的是HTTP而不是HTTPS，这没有任何问题。 HTTPS在Web浏览器和Web服务器之间加密，但Tor洋葱服务已经是端到端加密，因此不需要HTTPS（Let's Encrypt不会对`.onion`网站签署HTTPS证书，所以就算用HTTPS也会出现浏览器警告）。 与在Tor浏览器中加载普通网站不同，当你加载洋葱网站时，没有可以监视流量的Tor出口节点 - 所有流量都保留在Tor网络中。

现在，我需要将网址发送给与我共享文件的朋友。 安全地执行此操作的最简单方法是使用加密的消息传递应用，如Signal Desktop，Wire，Keybase或iMessage - 或者，如果你想用传统的方法，可以用Jabber / OTR。 如果你共享的文件不是特别敏感，也可以通过容易被监视但可能更方便的方式共享此网址，例如在Facebook，Twitter，Google Hangouts，Slack或Discord消息中， 或者在电子邮件中。

![img](https://bbs.kanxue.com/upload/attach/202001/665727_UFZ5BB2K389MXR6.jpg)

我的朋友打开Tor浏览器并加载我发给她的OnionShare地址。 她会看到一个网站上有我正在分享的文件列表，还有一个“下载文件”按钮。

关于OnionShare如何工作的关键点是该网站*直接*托管在我的电脑上，而不是托管在某个数据中心的服务器上。 如果我暂停我的电脑，该链接将不再有效。 如果我在文件被下载之前关闭OnionShare，那么对应的Tor网址就会从互联网上消失。 因此，OnionShare总是实时发生 - 你和你共享文件的人需要*同时在线*。

## 仅共享一个文件，OnionShare不会压缩

我的朋友开始下载文件，这些文件会自动压缩。 在这种情况下，zip文件名为`onionshare_snwga4.zip`（文件名的最后一部分是随机的），当她解压缩时，她可以看到我发送的OnionShare源代码的副本。 OnionShare 2中的一个新功能是，如果您只共享一个文件，它就不再将其压缩（但仍然使用内置于HTTP协议中的gzip压缩它）。

![img](https://bbs.kanxue.com/upload/attach/202001/665727_AU3Q57G5RR8G6J7.jpg)

回到我的电脑上，OnionShare在文件发送完成的那一刻自动停止共享，同时OnionShare地址完全从互联网上消失，再也不会重现。

## OnionShare文件地址默认只能使用一次

OnionShare地址具有*临时性*，用于一次性使用。 （例如，如果有人在Twitter DM中向你发送OnionShare地址，你在几分钟后加载并下载文件，之后该地址将不再存在。如果有人访问您的Twitter DM历史记录 ，上面的OnionShare地址将不再有效。）

![img](https://bbs.kanxue.com/upload/attach/202001/665727_STCGAFT4D7B933Z.jpg)

如上图所示，在我朋友下载完我共享的文件后,OnionShare地址不再存在。

这是OnionShare的默认设置，但你可能希望使用OnionShare将文件发送给一组人，而不仅仅是一个人，在这种情况下，你不希望它是短暂的。 你可以在“设置”中取消选中“在文件发送后停止共享”，然后多个人可以下载你的文件，并且你可以继续共享，直到手动单击“停止共享”或关闭OnionShare为止。

## 接收文件

![img](https://bbs.kanxue.com/upload/attach/202001/665727_MEJQ3KVV5DC6YB5.jpg)

所有这些或多或少都是OnionShare一直以来的工作方式。 但是“接收文件”标签对于OnionShare 2来说是全新的。它仍然只是一个开始 - 我们计划在未来向OnionShare添加更多标签，例如[静态网站共享](https://bbs.kanxue.com/elink@276K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0K9h3y4S2K9r3k6D9k6h3g2Q4x3V1k6G2L8X3W2G2L8Y4y4Z5j5i4u0W2i4K6u0r3K9i4y4K6N6h3g2K6i4K6u0r3y4K6V1I4)标签。

当使用OnionShare接收文件时，你就将计算机变成匿名保管箱 - 可以类比成超轻量级的[SecureDrop](https://bbs.kanxue.com/elink@e5bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3g2V1M7X3!0H3i4K6u0W2L8%4u0Y4i4K6u0r3)版本，任何人都可以在自己的电脑上免费运行，而无需设置任何基础设施或做任何系统管理。 （但是，它不像SecureDrop那样稳定或安全，SecureDrop以OnionShare无法实现的方式强制执行隔离环境。）

![img](https://bbs.kanxue.com/upload/attach/202001/665727_GR2KJM45F4CWH68.jpg)

在我点击“开始接收模式”几秒钟后，OnionShare给了我一个复杂的Tor地址，和之前一样的方式分享给对方。 但这次，我没有向她发送文件，*她可以向我发送文件*，她需要的只是Tor浏览器 - OnionShare是不需要的。

如果我发布这个OnionShare地址，*任何人都可以向我发送文件*，我没有任何方式可以知道他们是谁（因为他们将使用Tor），并且用端到端加密的方式，所以窃听我们任何一方网络的窃听者无法访问这些文件，甚至无法意识到我们正在使用OnionShare。对于窃听者来说，他们所能获得的唯一信息就是我们都在使用Tor。

![img](https://bbs.kanxue.com/upload/attach/202001/665727_F5AF67VAPTB4BF6.jpg)

我的朋友只需单击“浏览”，找到要发送的文件，然后单击“发送文件”。

![img](https://bbs.kanxue.com/upload/attach/202001/665727_2U5UY2778YNY22C.jpg)

在我计算机上的OnionShare中，我可以看到我收到的所有文件的状态。 因为OnionShare使用Tor洋葱服务，我实际上没有任何方式知道*谁*正在向我发送文件 - 如果我只想特定的人可以向我发送文件，我需要安全地向对方共享OnionShare地址。在我单击“停止接收模式”或关闭OnionShare之前，我的计算机将继续充当匿名保管箱。 而且，就像共享模式一样，如果我暂停我的电脑，OnionShare地址就会停止工作。

![img](https://bbs.kanxue.com/upload/attach/202001/665727_9N8BPGZRKAUHGRM.jpg)

我收到的文件会保存到`~/OnionShare`目录下，并按日期和时间自动排序。

## 只打开信任的文件

现在您已经知道“接收文件”选项卡的工作方式，*要非常小心*，并确保将以下这点牢记于心：

> **打开一些文件可能导致它们控制你的计算机。只打开来自信任的人的文件，要清楚地知道自己在做什么。**

一切关于不打开电子邮件附件的警告同样适用于此处。 在这个例子中，我收到了`gimp-2.10.8-x86_64-2.dmg`，它是开源图像编辑软件GIMP的安装程序。 但是，它很容易被修改成隐藏着恶意软件的安装程序。 如果有人向你发送`.docx`或`.pdf`文件，它们可能包含可能试图通过Microsoft Word或Adobe Reader中的漏洞攻击你的恶意软件。 如何安全地打开你收到的文件*超出了OnionShare* 的范围，所以请谨慎使用此功能。

## 公共模式

OnionShare 2有一个新功能：公共模式。 但在解释它的作用之前，我将先解释OnionShare如何在没有它的情况下工作。

默认情况下，OnionShare地址看起来是`` ba9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8W2)9#2b7Y4c8G2M7W2)9J5k6r3q4V1k6s2u0W2M7%4y4Q4y4f1c8Q4x3X3g2G2L8X3W2G2L8W2)9J5c8W2)9#2b7Y4y4D9N6h3N6Q4y4f1b7`. ``，其中slug是7,776个单词列表中的两个随机单词（从技术上讲，这是一个2字的[diceware密码短语](https://bbs.kanxue.com/elink@36aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1K9r3g2A6L8Y4c8W2M7X3y4W2M7s2c8Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5#2i4K6u0r3x3o6y4Q4x3V1j5J5y4W2)9J5c8Y4m8S2M7%4y4H3K9s2u0S2M7$3g2K6i4K6u0V1j5$3q4F1i4K6u0V1L8h3g2E0L8%4u0A6P5X3g2Q4x3X3c8S2N6s2c8S2j5$3E0W2M7Y4y4Q4x3X3c8U0j5h3&6@1i4K6u0V1k6%4g2W2M7%4y4Q4x3V1j5`.)）。背后的想法是，如果攻击者能够找出地址的“tor-address”部分，但不知道slug，他们仍然无法下载你正在共享的文件或上传文件到你的计算机。 slug本质上是一个密码。 （当使用`v3`洋葱服务时，这不太重要。旧的`v2`onion服务有一个已知的问题，如果洋葱连接碰巧被恶意的Tor节点促成时，该节点可以知悉`tor-address`的部分。 这是“v3`洋葱更安全的原因之一。”但slug也只有两个词，并且OnionShare使用的词表是公开的，是什么阻止了攻击者通过猜测每一个可能的组合呢？

![img](https://bbs.kanxue.com/upload/attach/202001/665727_PK5C7FTS8TU3G8C.jpg)

OnionShare计算出现了多少404错误（文件未找到Web请求），并且在第20个404错误中，它假定有人试图猜测slug并自动停止服务器。这使攻击者有20次机会正确猜出这两个单词。 因为这些单词是从7,776个单词的列表中选择的，这意味着有7,776 ^ 2，即超过6000万个可能的组合。 这基本上意味着如果攻击者试图猜测slug，他们在其仅有的20次机会中的正确率的0.00003％，也就是99.99996％的失败机会，同时会迫使OnionShare停止共享。这很好用，但是如果你*希望*任何人能够加载你的OnionShare地址怎么办？

比如说你要匿名发布一些文件，你禁用了“文件发送后停止共享”的设置，将文件拖到OnionShare开始共享并发布OnionShare地址。 事实证明，互联网上看到推文的任何人都可以通过发出20次404错误来强制您的服务器停止。 （顺便说一下，这种情况确实发生了，有些服务器一直在关闭，因为人们一直在制造404错误。）

这时体现了公共模式的优势。 如果您想公开共享OnionShare地址，只需转到“设置”并选中“公共模式”旁边的框。 当你开始共享时，OnionShare地址看起来像`` 5f7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8W2)9#2b7Y4c8G2M7W2)9J5k6r3q4V1k6s2u0W2M7%4y4Q4y4f1c8Q4x3X3g2G2L8X3W2G2L8W2)9J5c8R3`.`. ``，没有slug，无论有多少404错误，服务器都会保持运行状态。

## 匿名接收文件

接收模式对于允许人们私下发送文件非常有用。但是，如果你想允许*任何人*私下向你发送文件，可以考虑使用一些设置。

首先，你肯定希望启用公共模式。

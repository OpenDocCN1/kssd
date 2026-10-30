
### TSX异步中止(TAA)

在2019年11月, MDS攻击发布了新的变种, 看起来TAA PoC要比我的MFBDS利用快很多。我决定把它移植到Chrome利用上。除此之外, VUSec还发布了一个<a href="elink@7aaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6$3N6i4y4W2j5#2)9J5c8Y4u0A6k6r3I4Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7r3!0U0M7#2)9J5c8Y4c8S2j5g2)9#2k6X3u0S2M7$3W2U0i4K6u0W2j5H3`.`.">针对store指令</a>的利用, 如果我们能将秘密写到内存的不同地址, 这应该能让我们避免了缓存刷新的要求。 如果我们能触发浏览器来向特权端口发送消息, 则应该能发生这种情况。 在这种情况下, 秘密端口名也将以节点名称作为前缀, 我们可以使用RIDL论文中的技术轻松对其进行过滤.

我也开始寻找有没有更好的原语, 然后我发现如果我能和<a href="elink@b27K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0M7#2)9J5k6h3y4Z5M7X3!0E0K9i4g2E0i4K6u0W2L8%4u0Y4i4K6u0r3j5$3S2J5L8$3#2A6N6h3#2Q4x3V1k6K6M7X3y4Q4x3V1k6K6k6i4u0$3K9h3y4W2M7#2)9J5c8X3&6W2N6s2N6G2M7X3E0Q4x3V1k6H3N6h3u0D9K9h3y4Q4x3V1k6E0L8$3A6G2L8g2)9J5c8X3&6W2N6s2N6G2M7X3E0Q4y4h3k6K6k6i4u0$3K9h3y4W2i4K6u0W2L8h3!0B7L8$3#2Q4x3@1k6D9i4K6y4p5x3e0V1%4i4K6t1$3M7X3y4D9i4K6y4p5k6o6j5^5z5r3t1^5j5$3y4V1k6h3p5&6x3K6k6S2j5X3j5$3k6o6S2X3y4o6S2W2j5e0f1^5y4o6R3^5j5K6M7@1z5o6t1%4x3o6f1$3j5R3`.`.">NetworkService</a>通信, 它会允许我创建一个新的<a href="elink@afcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0M7#2)9J5k6h3y4Z5M7X3!0E0K9i4g2E0i4K6u0W2L8%4u0Y4i4K6u0r3j5$3S2J5L8$3#2A6N6h3#2Q4x3V1k6K6M7X3y4Q4x3V1k6K6k6i4u0$3K9h3y4W2M7#2)9J5c8X3&6W2N6s2N6G2M7X3E0Q4x3V1k6H3N6h3u0D9K9h3y4Q4x3V1k6E0L8$3A6G2L8g2)9J5c8X3&6W2N6s2N6G2M7X3E0Q4y4h3k6U0L8$3&6@1k6i4S2@1i4K6u0W2L8h3!0B7L8$3#2Q4x3@1k6D9i4K6y4p5z5o6l9%4i4K6t1$3M7X3y4D9i4K6y4p5k6o6j5^5z5r3t1^5j5$3y4V1k6h3p5&6x3K6k6S2j5X3j5$3k6o6S2X3y4o6S2W2j5e0f1^5y4o6R3^5j5K6M7@1z5o6t1%4x3o6f1$3j5R3`.`.">NetworkContext</a>, 从而选择保存了cookies的sqlite3数据库的<a href="elink@f34K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0M7#2)9J5k6h3y4Z5M7X3!0E0K9i4g2E0i4K6u0W2L8%4u0Y4i4K6u0r3j5$3S2J5L8$3#2A6N6h3#2Q4x3V1k6K6M7X3y4Q4x3V1k6K6k6i4u0$3K9h3y4W2M7#2)9J5c8X3&6W2N6s2N6G2M7X3E0Q4x3V1k6H3N6h3u0D9K9h3y4Q4x3V1k6E0L8$3A6G2L8g2)9J5c8X3&6W2N6s2N6G2M7X3E0Q4y4h3k6U0L8$3&6@1k6i4S2@1i4K6u0W2L8h3!0B7L8$3#2Q4x3@1k6D9i4K6y4p5x3U0x3K6i4K6t1$3M7X3y4D9i4K6y4p5k6o6j5^5z5r3t1^5j5$3y4V1k6h3p5&6x3K6k6S2j5X3j5$3k6o6S2X3y4o6S2W2j5e0f1^5y4o6R3^5j5K6M7@1z5o6t1%4x3o6f1$3j5R3`.`.">文件路径</a>.

为了找到如何从浏览器进程触发消息到NetworkService, 我在接口中寻找可能在渲染器中控制的的IPC方法。<a href="elink@456K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0M7#2)9J5k6h3y4Z5M7X3!0E0K9i4g2E0i4K6u0W2L8%4u0Y4i4K6u0r3j5$3S2J5L8$3#2A6N6h3#2Q4x3V1k6K6M7X3y4Q4x3V1k6K6k6i4u0$3K9h3y4W2M7#2)9J5c8X3&6W2N6s2N6G2M7X3E0Q4x3V1k6H3N6h3u0D9K9h3y4Q4x3V1k6E0L8$3A6G2L8g2)9J5c8X3&6W2N6s2N6G2M7X3E0Q4y4h3k6K6k6i4u0$3K9h3y4W2i4K6u0W2L8h3!0B7L8$3#2Q4x3@1k6D9i4K6y4p5x3K6p5^5i4K6t1$3M7X3y4D9i4K6y4p5k6o6j5^5z5r3t1^5j5$3y4V1k6h3p5&6x3K6k6S2j5X3j5$3k6o6S2X3y4o6S2W2j5e0f1^5y4o6R3^5j5K6M7@1z5o6t1%4x3o6f1$3j5R3`.`.">NetworkService.OnPeerToPeerConnectionsCountChange</a>吸引到了我, 并且实际上, 这个方法在每次WebRTC连接更新时都会被调用。 你只需要创建一个假的WebRTC连接, 每次你将它标记为连接/断开时, 它会触发到NetworkService的新消息。

![img](upload/attach/202003/820961\_ZX46KXSP59AFWAT.png)

一旦我们从破坏的渲染器中泄露了端口名, 我们就得到了能够写入具有完全受控路径的sqlite3数据库的原语。

然而一开始这个听起来不太有用, 你实际上可以利用它来获得代码执行。我注意到Windows批处理文件是非常宽容的文件格式. 如果你在该类型文件的开头有许多垃圾数据, 它会跳过这些数据直到下一个"\r\n", 然后执行那个位置的下一条命令。在我的利用中, 我利用这点在用户的自动运行(autorun)文件夹中创建了cookies.bat文件, 添加了带有"\r\n"和一条命令的cookie, 它会在下一次登录的时候得到执行.

最后, 在我的机器上该利用平均运行时间在1-2分钟, 最长不超过5分钟。我也确信这个速度还能有很大改进, 因为我看到了有许多加快速度的办法, 不论是小的修改还是不同的技术。比如说, MLPDS看起来比我用的变种要快的多。

利用总结:

1. 破坏渲染器。
2. 在`$NUM_CPU-1`个进程上以不同的缓存行偏移量运行RIDL利用程序。
3. 创建伪造的WebRTC连接, 在连接/断开状态切换。
4. 泄露NetworkService端口名。
5. 创建一个新的NetworkContext写入cookie文件, 位置在`c:\path\to\user\autorun\cookies.bat`。
6. 插入cookie `"\r\ncalc.exe\r\n"`。
7. 等待下一次登录。

## 总结

当我开始研究这个漏洞时, 我很惊讶的发现, 即使漏洞公开了这么长时间, 它还是能够被利用。如果你阅读过有关该主题的指南, 他们通常会在你的操作系统为最新的情况下谈论如何缓解这些漏洞, 并让你注意应禁用超线程以获得完全保护。对于缓解措施的关注的确给了我一种这类漏洞已经被解决的错觉, 而且我认为这些文章可以更清楚地说明启用超线程的影响。

也就是说, 我希望你能从这篇文章中了解到两个事情。第一点是, 信息泄露缺陷可能不只是用来绕过ASLR。即使它并不依赖于秘密端口名, 也会有一些其他有趣的数据泄露, 比如说Chrome的UnguessableTokens, Gmail的cookies或者机器中其他进程的敏感数据。如果你想发现大规模的信息泄露, Chrome或许是个不错的目标。

第二点是, 我长久以来忽视了硬件漏洞, 因为它们在我的舒适区外。然而, 我希望我可以通过这篇文章为你提供有关其影响的另一个数据点, 以帮助你做出是否应该禁用超线程的决定。对于用相似的方法来研究还有哪些能被破坏的软件, 还有很大的研究空间。我也很期待看到利用硬件缺陷来打破软件安全边界的更多例子。

# 使用RIDL逃逸Chrome沙箱

---

翻译: 看雪翻译小组-Nxe

校对: 看雪翻译小组-fyb波

原文链接:[a96K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5k6h3u0D9L8$3N6K6M7r3!0@1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9J5x3q4)9J5c8U0l9J5i4K6u0r3k6i4y4U0j5i4m8A6L8X3N6Q4x3X3c8U0K9s2u0G2L8h3g2Q4x3X3c8K6j5h3&6V1j5X3!0^5i4K6u0V1N6$3W2@1K9q4)9J5k6s2u0A6k6r3I4Q4x3X3g2Z5N6r3#2D9](https://bbs.kanxue.com/elink@e33K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5k6h3u0D9L8$3N6K6M7r3!0@1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9J5x3q4)9J5c8U0l9J5i4K6u0r3k6i4y4U0j5i4m8A6L8X3N6Q4x3X3c8U0K9s2u0G2L8h3g2Q4x3X3c8K6j5h3&6V1j5X3!0^5i4K6u0V1N6$3W2@1K9q4)9J5k6s2u0A6k6r3I4Q4x3X3g2Z5N6r3#2D9)

---

太长不看: 可以利用泄露跨进程内存的漏洞来进行Chrome沙箱的逃逸。在发起攻击之前, 攻击者仍需破坏渲染器. 为了防止在受影响的CPU平台上受到此类攻击, 请确保你的微代码`(microcode)`是最新的**并且**关闭了超线程`(hyper-threading, HT)`.

在我上一篇客座博客["破坏数据流"("Trashing the Flow of Data")](https://bbs.kanxue.com/elink@7d3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5k6h3u0D9L8$3N6K6M7r3!0@1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4z5g2)9J5c8U0l9#2i4K6u0r3N6s2u0S2M7$3S2A6L8X3N6Q4x3X3c8X3L8r3!0%4i4K6u0V1L8$3k6Q4x3X3c8V1j5i4c8S2i4K6u0W2K9s2c8E0L8l9`.`.)中, 我描述了如何利用Chrome的JavaScript V8引擎中的漏洞来获取渲染器中的代码执行权限。为了让该攻击变得有用, 你通常需要第二个漏洞以实现利用链, 因为Chrome沙箱限制了你对OS的访问, 而且[网站隔离`(Site Isolation)`功能](https://bbs.kanxue.com/elink@4b3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0K9s2u0G2L8h3W2#2L8g2)9J5k6h3!0J5k6#2)9J5c8V1S2G2L8h3g2Q4x3V1k6U0K9s2u0G2L8h3W2#2L8g2)9J5k6s2y4W2j5%4g2J5K9i4c8&6i4K6u0r3M7$3W2@1k6g2)9J5k6r3W2K6L8$3I4S2N6r3W2G2L8R3`.`.)将跨站渲染器移到了各自的进程中, 以防止绕过网页平台的限制.

在这篇文章中, 我们将研究特别是在[`RIDL`](https://bbs.kanxue.com/elink@c37K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6s2y4S2N6s2c8S2j5$3E0K6i4K6u0W2j5$3!0E0i4K6u0r3)以及相似的硬件漏洞影响下, 受攻击的渲染器使用的沙箱。Chrome的IPC机制Mojo基于用于消息路由的机密, 泄露这些机密会允许我们向特权接口发送信息, 并执行渲染器不被允许执行的操作。我们利用这点来读取任意本地文件, 以及在Windows系统上运行沙箱外的.bat文件。 在这篇文章写作时, Apple和Microsoft都在与Chrome安全团队合作, 积极地尝试修复以防止此类攻击.

## 背景

下图是简化版的Chrome进程模型:

![img](https://bbs.kanxue.com/upload/attach/202003/820961_C227NR84GHFDF42.png)

渲染器进程分别运行在各自的沙箱中, 并且它们对内核的访问都是受限制的, 比如通过Linux上的`seccomp`过滤器或者Windows上的[win32k lockdown](https://bbs.kanxue.com/elink@af3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5k6h3u0D9L8$3N6K6M7r3!0@1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4y4W2)9J5c8U0p5I4i4K6u0r3j5Y4u0W2j5h3E0A6L8X3N6Q4x3X3c8U0K9r3q4A6L8W2)9J5k6h3S2@1L8h3H3`.)。但是为了使渲染器执行任何有用的操作, 它需要与其他进程通信以执行各种操作。例如要加载图像时, 它会请求网络服务代表其获取图像.

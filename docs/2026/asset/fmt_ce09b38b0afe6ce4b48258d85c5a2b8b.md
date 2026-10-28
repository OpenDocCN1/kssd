
# [翻译]CreateSvcRpc - 一个自定义的 RPC 客户端，以 SYSTEM 用户身份执行程序

**翻译**

原文地址：085K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2^5z5o6k6E0j5i4c8@1K9r3g2%4i4K6u0W2j5$3!0E0i4K6u0r3N6X3W2W2N6#2)9#2k6Y4m8G2M7%4c8Q4x3@1k6A6k6q4)9K6c8r3y4J5k6h3q4@1k6g2)9#2k6Y4y4$3j5#2)9#2k6Y4u0H3j5H3`.

功能：一个自定义的 RPC 客户端，以 `SYSTEM` 用户身份执行程序

---

Windows RPC 协议是一个我以前没有做过很多实验的领域。我现在已经创建了一个自定义的 RPC 客户端，它使用原始数据与 `ntsvcs` 管道进行通信。这意味着可以只使用 `CreateFile` 和 `WriteFile` APIs 来创建和执行一个 Windows 服务。

RPC 协议似乎有一些记录，但相关的信息非常零散，我发现从头开始逆向工程更容易。

我通过拦截 `NtWriteFile`、`NtReadFile` 和 `NtFsControlFile` 函数，记录了 Windows 服务 API 的通信。我分析了这个数据流，以逐步建立我自己的 RPC 客户端。

在我的第一个版本运行后，我在 `Wireshark` 文档中发现了一些有用的信息，这些信息帮助我在 RPC 头文件中标注了剩余的未知字段。

![](https://bbs.kanxue.com/upload/attach/202202/767964_DPHB5453TYWTCWC.jpg)

以下是完整的程序代码：


- 正如我们讨论过的那样，OLE 对象是 COM 对象的一个子集，而 COM 对象又是与 CLSID 相关联的对象的子集。
- 许多注册在操作系统中的 COM 对象并不是 OLE 对象。
- 几百个 vs. 几千个。
- 有时候，甚至一个在注册表中有 CLSID 相关联的 DLL 也不是一个 COM。
- 但是，`CoCreateInstance` 仍然会将与 CLSID 相关联的 DLL 加载到进程中。
- 不管这个 DLL 是否是一个“OLE DLL”。
- 被加载的 DLL 不会被卸载，即使它在后面被发现不是一个“OLE DLL”。
- 在我们看来，这是 Windows 初始化 OLE 对象过程中的一个*设计*问题。
- 在没有加载 DLL 之前，你无从知道 COM 是否暴露了你想要的接口！
- 让我们来将其与它著名的“姐妹”特性比较一下：IE 浏览器中的 ActiveX 控件。
- 不像 OLE，IE11 加载一个 ActiveX 控件时会首先检查“白名单”列表：
  `HKLM\Software\Microsoft\Windows\CurrentVersion\Ext\PreApproved`
- 所以，如果当前的 ActiveX CLSID 不在列表里面，相关联的 DLL 不会被加载到 IE 进程。
- IE 进程里面的 ActiveX 是没有问题的。

### 后果

- 我们讨论的这个问题会导致怎样的坏结果？
- 我们可以加载任何 DLL 到进程，只要这个 DLL 有其相关联的 CLSID。
- 考虑到攻击通过文档来启动。
- 有一些这样的例子。
- 注意：加载 OLE DLL 可能也会有相同的问题。但是，能够加载每一个 CLSID 相关联的 DLL *显著地* 增大了攻击载体。

### 后果 1：加载未开启 ASLR 的 DLL

- 在容器进程中加载一个未开启 ASLR 的 DLL。
- 也就是说，Word、PowerPoint、Excel、WordPad。
- 然后可以在利用时被用来绕过 ASLR。
- 注意，不仅是 CLSID 相关联的 DLL 会有未开启 ASLR 的情况，有时候 CLSID 相关的 DLL 也可以链接到另一些未开启 ASLR 的 DLL（所以这些 DLL 也会被加载）。
- 这一攻击面在 Office 2013 及以后版本无效，因为它们开启了“强制 ASLR”。

[![](https://bbs.kanxue.com/elink@3f2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6K6i4K6u0W2N6r3g2U0K9r3&6W2N6q4)9J5k6h3y4G2L8g2)9J5c8X3u0Q4x3V1k6K6M7X3c8Q4x3V1k6S2M7X3y4Z5K9i4k6W2i4K6u0r3x3U0l9I4x3#2)9J5c8U0p5J5i4K6u0r3x3e0q4Q4x3V1k6K6L8$3k6@1N6$3q4J5k6g2)9J5k6r3c8W2k6X3g2F1M7$3g2Q4x3X3c8E0K9i4c8A6k6$3q4@1K9h3&6Y4i4K6u0V1j5$3!0E0L8h3!0F1i4K6u0V1k6i4S2H3L8r3!0A6N6r3q4@1K9h3!0F1i4K6u0V1N6r3g2U0K9r3&6A6M7i4g2W2M7#2)9J5k6h3q4K6M7s2R3`.](https://bbs.kanxue.com/elink@3f2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6K6i4K6u0W2N6r3g2U0K9r3&6W2N6q4)9J5k6h3y4G2L8g2)9J5c8X3u0Q4x3V1k6K6M7X3c8Q4x3V1k6S2M7X3y4Z5K9i4k6W2i4K6u0r3x3U0l9I4x3#2)9J5c8U0p5J5i4K6u0r3x3e0q4Q4x3V1k6K6L8$3k6@1N6$3q4J5k6g2)9J5k6r3c8W2k6X3g2F1M7$3g2Q4x3X3c8E0K9i4c8A6k6$3q4@1K9h3&6Y4i4K6u0V1j5$3!0E0L8h3!0F1i4K6u0V1k6i4S2H3L8r3!0A6N6r3q4@1K9h3!0F1i4K6u0V1N6r3g2U0K9r3&6A6M7i4g2W2M7#2)9J5k6h3q4K6M7s2R3`.)]

不过仍然可以工作在 Office <= 2010 和 WordPad 上面。

#### 案例 1：`otkloadr.WRAssembly.1`

- 尝试去加载被 ProgID 定义的“COM 对象”：`otkloadr.WRAssembly.1`

![](https://bbs.kanxue.com/upload/attach/201710/628056_91c1f2uik4qbslt.png)

- 这甚至不是一个 COM！
- 将会加载 `C:\Program Files\Microsoft Office\Office14\ADDINS\OTKLOADR.DLL`，这会导致加载其链接的、位于同一目录下的未开启 ASLR 的模块：`MSVCR71.DLL`。
- 2014 年 6 月被 Parvez Anwar 公开[c56K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3N6J5k6i4W2Z5j5i4c8Z5j5h3y4C8k6i4u0Q4x3X3g2F1k6i4c8Q4x3V1k6Q4x3@1k6H3i4K6y4p5y4K6M7H3](https://bbs.kanxue.com/elink@9e3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3N6J5k6i4W2Z5j5i4c8Z5j5h3y4C8k6i4u0Q4x3X3g2F1k6i4c8Q4x3V1k6Q4x3@1k6H3i4K6y4p5y4K6M7H3) ，已经被微软修复。

#### 案例 2：`mscormmc.dll`

- 这个未开启 ASLR 的 DLL 在默认的 Windows 7 上：
  `C:\Windows\Microsoft.NET\Framework\v1.0.3705\mscormmc.dll`
- 有好些 CLSID 与这个 DLL 相关联，举个例子：
  `{18BA7139-D98B-43C2-94DA-2604E34E175D}`
- 构造一个带有这个 CLSID 的含 OLE 对象的 Office 或 RTF 文档，你就可以加载这个未开启 ASLR 的 DLL 到进程。
- 到现在还是管用的！找到一个未开启 ASLR 的 DLL 比较简单，找到它只花了几分钟的时间。

### 后果 2：内存破坏

- 有些时候，加载一个“未准备好的” DLL 足够触发内存破坏了。
- 例子：微软 Office 内存未初始化漏洞（CVE-2015-1770）。
- CLSID：`CDDBCC7C-BE18-4A58-9CBF-D62A012272CE`
- 相关联的 DLL：`C:\Program Files\Microsoft Office\Office15\OSF.DLL`
- 只要尝试加载与上面的 CLSID 相关联的 DLL，就会给你一个 crash（可利用）！
- `OSF.DLL` 当然不是被设计用来给你作为一个 OLE 或者 ActiveX 对象加载的。
- 由 MWR 实验室的 Yong Chuan Koh 发现，更多细节请看：

[3eeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9j5h3u0K6i4K6u0W2L8i4N6J5K9h3&6X3L8%4y4W2j5%4g2J5K9i4c8&6i4K6u0W2j5$3!0E0i4K6u0r3M7%4W2K6N6r3g2E0i4K6u0r3j5i4y4K6k6i4c8K6i4K6u0r3z5e0R3%4i4K6u0r3L8%4u0A6k6$3W2F1j5h3I4Q4x3V1k6E0N6%4u0A6i4K6g2X3j5h3c8$3K9i4y4G2M7Y4W2Q4y4h3k6U0N6X3g2Q4x3X3b7J5x3o6p5#2i4K6u0V1x3e0M7%4x3q4)9J5k6i4m8V1k6R3`.`.](https://bbs.kanxue.com/elink@458K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9j5h3u0K6i4K6u0W2L8i4N6J5K9h3&6X3L8%4y4W2j5%4g2J5K9i4c8&6i4K6u0W2j5$3!0E0i4K6u0r3M7%4W2K6N6r3g2E0i4K6u0r3j5i4y4K6k6i4c8K6i4K6u0r3z5e0R3%4i4K6u0r3L8%4u0A6k6$3W2F1j5h3I4Q4x3V1k6E0N6%4u0A6i4K6g2X3j5h3c8$3K9i4y4G2M7Y4W2Q4y4h3k6U0N6X3g2Q4x3X3b7J5x3o6p5#2i4K6u0V1x3e0M7%4x3q4)9J5k6i4m8V1k6R3`.`.)

### 后果 3：DLL 劫持

- 还有一种藏在深处的攻击场景。
- 注意，这是关于基于文档的攻击。
- 当前工作目录是攻击者可以控制的。
- 我其实不应当去解释一个 DLL 劫持攻击。

![](https://bbs.kanxue.com/upload/attach/201710/628056_qbcazvw1k6wv4th.png)

- CVE-2015-2369 是我们报告的一个很好的例子，在不久前的 6 月补丁日刚被修复。
- 最小的 POC 甚至不到一条 Twitter 的长度（140 字节）。

```text
{\rt\object\objocx{\objdata 010500000200000014000000574D444D434553502E574D444D434553502E3100000000000000000001000000410105000000000000}}
```

- CLSID 相关联的 DLL：
- ProgID：`WMDMCESP.WMDMCESP.1`
- CLSID：`{067B4B81-B1EC-489f-B111-940EBDC44EBE}`
- DLL：`%systemroot%\System32\cewmdm.dll`
- 将会导致从当前目录加载一个叫做“`rapi.dll`”的 DLL。
- 演示！（译者注：略）

#### 对攻击载体的总结

- 基于受害者打开文档的时间流，这些攻击载体是：
- 许多类型的攻击可能发生在“CLSID 相关联的 DLL 加载”过程中——“OLE 对象初始化最开始的步骤”：
  - 加载未开启 ASLR 的 DLL
  - 内存破坏
  - DLL 劫持
  - …
- 许多类型的漏洞可能存在于 `IPersistStorage::Load` 例程，“OLE 对象初始化”的另一个步骤。
- 通过 `IOleObject::DoVerb` 对“Verb”动作的攻击：
  - 通常为逻辑漏洞，更加危险。

#### 每一步都被攻击了

![](https://bbs.kanxue.com/upload/attach/201710/628056_ck8k9to34xozohd.png)

#### 对攻击面的总结

- OLE 机制提供了巨大的攻击面。
- 不像 ActiveX，一个 OLE 对象没有被像“白名单列表”、“脚本安全（SFS）”、“初始化安全（SFI）”这样的安全加固特性的严格限制。
- 能够加载任何 CLSID 相关的 DLL 使得攻击面变得更大。
- 默认 Windows 上有几百个 OLE 对象。
- 默认 Windows 上有几千个 CLSID 相关联的 DLL。
- 不要忘记这是一个开放领域。
- App 装得越多，攻击面就变得越大。
- 有一天我们会看到基于文档的、针对系统上装有特定软件的特定用户的攻击。
- 注意 OLE 加载过程会查询 IE/Office Killbit，所以如果一个 CLSID 被 Killbited 了，相关联的 DLL 将不会被加载。

## 结论

- OLE 机制服务于大多数微软的文档与其他组件的互通性。
- 提供了一个巨大的攻击面。
- 又一个 ActiveX？
- 即使它不具有脚本特性，它可以做比我们预想中多得多的事。
- 在这次演讲之后可以预期到什么？
- 可能会有更多 OLE 相关的漏洞会被发现。
- 可能会出现更多针对 Office/WordPad 的攻击。
- 沙箱和静态方式的检测和防御都需要被改进。
- 一个 OLE 特定的检测方法正在路上。
- 最近我们已经报告了一些新的逃逸技巧([a7cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6K6i4K6u0W2N6r3g2U0K9r3&6W2N6q4)9J5k6h3y4G2L8g2)9J5c8X3u0Q4x3V1k6K6M7X3c8Q4x3V1k6S2M7X3y4Z5K9i4k6W2i4K6u0r3x3U0l9I4x3#2)9J5c8U0p5J5i4K6u0r3x3e0q4Q4x3V1k6K6L8$3k6@1N6$3q4J5k6g2)9J5k6r3c8W2k6X3g2F1M7$3g2Q4x3X3c8E0K9i4c8A6k6$3q4@1K9h3&6Y4i4K6u0V1j5$3!0E0L8h3!0F1i4K6u0V1k6i4S2H3L8r3!0A6N6r3q4@1K9h3!0F1i4K6u0V1N6r3g2U0K9r3&6A6M7i4g2W2M7#2)9J5k6h3q4K6M7s2R3`.`.](https://bbs.kanxue.com/elink@7f8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6K6i4K6u0W2N6r3g2U0K9r3&6W2N6q4)9J5k6h3y4G2L8g2)9J5c8X3u0Q4x3V1k6K6M7X3c8Q4x3V1k6S2M7X3y4Z5K9i4k6W2i4K6u0r3x3U0l9I4x3#2)9J5c8U0p5J5i4K6u0r3x3e0q4Q4x3V1k6K6L8$3k6@1N6$3q4J5k6g2)9J5k6r3c8W2k6X3g2F1M7$3g2Q4x3X3c8E0K9i4c8A6k6$3q4@1K9h3&6Y4i4K6u0V1j5$3!0E0L8h3!0F1i4K6u0V1k6i4S2H3L8r3!0A6N6r3q4@1K9h3!0F1i4K6u0V1N6r3g2U0K9r3&6A6M7i4g2W2M7#2)9J5k6h3q4K6M7s2R3`.`.)，提示（大家）正确检出基于 Office 的攻击的困难性。
- 致厂商（微软）：
  - 这个很有问题的“OLE 加载”机制需要被重新审视，或许需要重新设计。
  - 你不能只仅仅加载每一个 CLSID 相关联的 DLL 到 Office/WordPad 进程。
  - 需要在默认操作系统内部进行大规模的渗透测试。
  - 新的攻击载体制造了许多新的漏洞。
  - 训练第三方厂商。
  - 就像你们之前对 ActiveX 的做法那样。

## 主要引用

1. Mark Dowd、Ryan Smith and David Dewey. “Attacking Interoperability”。[Online] [8dfK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2#2M7%4c8D9k6h3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6s2g2X3k6W2)9J5c8X3u0Z5x3U0l9H3z5g2)9#2k6X3c8G2N6$3c8Q4y4h3k6K6L8h3W2@1K9q4)9#2k6X3c8W2N6$3g2&6i4K6u0W2M7r3c8X3](https://bbs.kanxue.com/elink@bf8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2#2M7%4c8D9k6h3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6s2g2X3k6W2)9J5c8X3u0Z5x3U0l9H3z5g2)9#2k6X3c8G2N6$3c8Q4y4h3k6K6L8h3W2@1K9q4)9#2k6X3c8W2N6$3g2&6i4K6u0W2M7r3c8X3)
2. Don Box. “Essential COM”。[Book] [d69K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8$3!0C8M7#2)9J5k6h3N6G2L8$3N6D9k6g2)9J5k6h3y4G2L8g2)9J5c8X3u0G2L8$3E0K6i4K6u0r3j5h3u0G2N6i4c8Q4x3V1k6q4M7%4y4W2L8Y4c8A6j5h3I4Q4y4h3k6o6e0@1#2Q4x3X3g2Z5N6r3#2D9](https://bbs.kanxue.com/elink@865K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8$3!0C8M7#2)9J5k6h3N6G2L8$3N6D9k6g2)9J5k6h3y4G2L8g2)9J5c8X3u0G2L8$3E0K6i4K6u0r3j5h3u0G2N6i4c8Q4x3V1k6q4M7%4y4W2L8Y4c8A6j5h3I4Q4y4h3k6o6e0@1#2Q4x3X3g2Z5N6r3#2D9)
3. WikipediA. “Object Linking and Embedding”。[Online] [8ccK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2#2M7%4c8D9k6h3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6s2g2X3k6W2)9J5c8X3u0Z5x3U0l9H3z5g2)9#2k6X3c8G2N6$3c8Q4y4h3k6K6L8h3W2@1K9q4)9#2k6X3c8W2N6$3g2&6i4K6u0W2M7r3c8X3](https://bbs.kanxue.com/elink@02eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2#2M7%4c8D9k6h3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6s2g2X3k6W2)9J5c8X3u0Z5x3U0l9H3z5g2)9#2k6X3c8G2N6$3c8Q4y4h3k6K6L8h3W2@1K9q4)9#2k6X3c8W2N6$3g2&6i4K6u0W2M7r3c8X3)

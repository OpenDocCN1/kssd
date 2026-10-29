
# ParseDroid: 瞄准 Android 开发及研究团队

研究员：Eran Vaknin, Gal Elbaz, Alon Boxiner, Oded Vanunu

Check Point 研究小组的最新研究挖出了几个漏洞，这些漏洞将会置广大从事 Java/Android 开发的组织或机构于险地。

这里所提到的漏洞是存在于 Android 应用程序开发者工具中，这些工具可下载也可基于云使用，囊括所有的 Java/Android 程序员用来开发他们公司的商业应用和用来对他们的作品进行安全分析和逆向的工具。

根据我们的研究，我们发现了几个漏洞，这些漏洞将波及最常用的 Android IDE——谷歌的 `Android Studio`，JetBrains 的 `IntelliJ IDEA` 和 `Eclipse`，以及广大 Android 应用逆向工具，诸如 `APKtool` 和其他。

这个视频展示了我们是如何利用这些工具获取内部文件。2017 年五月，Check Point 已向 `APKTool` 开发者和其他的 IDE 公司报告这个发现，谷歌和 JetBrains 已经证实并承认该安全问题，并进行有效修复。

## 技术细节——从 XXE 到 RCE：攻击第二层

我们第一阶段的研究重点是 `APKTool`（Android Application Package Tool）。

作为最流行的 Android 第三方应用程序逆向工具，`apktool` 用来支持自定义平台，分析应用程序和其他的工作，包括反编译（`decode`）和重打包（`rebuild`）。

`APKtool` 的两个主要功能是：

1. 反编译（`decompile`） `apk` 文件。
2. 打包（`build`） `apk` 文件。

我们的研究发现，`apktool` 在这两个方面都不太安全。

通过查看 `apktool` 源码，我们成功找到一个 XML 外部实体漏洞（XML External Entity, `XXE`），因为 `apktool` 内置的 XML 解析器在解析程序中的 XML 文件时没有禁止外部实体引用。

这个不安全的函数就是 `loadDocument`，它在 `apktool` 的两个核心功能——“`build`”和“`decompile`”——都有用到。

这个漏洞将 `APKTool` 用户的整个操作系统的文件系统暴露在外，这将导致，攻击者可以检索受害者 PC 上的任意文件，通过使用一个恶意的“`AndroidManifest.xml`”文件，利用 `XXE` 漏洞，就可以发送给远程攻击者服务器。而且这种攻击场景只是众多可能中的一种，`XXE` 攻击技术可造成大危害。

意识到这个漏洞对于 Android 开发者及研究团队的强大危害，我们将我们的研究扩展到 XML 解析器，`DocumentBuilderFactory`，这在 `APKTool` 中有用到。

### 开发者工具中的漏洞

这样我们找到几种 XML 解析器在其他项目中不安全的实现方式。而且，我们还发现这波及到那些用来开发 Android 应用程序的 IDE——包括，`Intellij`、`Eclipse`、和 `Android Studio`。

只需简单地加载恶意“`AndroidManifest.xml`”文件到任意 `android` 项目中，这些 IDE 就开始吐出攻击者配置好的任意文件。

为了证明这个漏洞，我们上传了一个恶意的项目库到 `Github`，并 `clone` 到 `Android Studio`。

例子：`Github` 上一个带有 `XXE Payload` 的项目

`Clone` 到 `Android Studio`：

结果：攻击很成功。在用户没有察觉的情况下成功盗取受保护文件，并发送到攻击者服务器。

而且，我们还发现了另外一种攻击场景，可以通过注入含有这个 `XXE Payload` 的恶意 `AAR`（Android Archive Library）到代码仓库中，将可以攻击更大范围 Android 开发者。

这个是可能的，比如，上传受感染的 `AAR` 到一个公共代码仓库（`public repository`），例如 `Maven` 的 `central repository`。为了验证，我们已经上传了一个受感染的 `AAR` 到一个本地代码仓库（`local repository`）。只要受害者从该仓库下载受感染的 `AAR`，攻击者就可以盗取敏感文件，比如配置文件，源代码，公司数字资产和其他的 OS 文件系统。

上传恶意 `AAR` 到本地代码仓库：

添加该 `AAR` 到 `Android Studio` 项目中：

结果：攻击很成功。在用户没有察觉的情况下成功盗取受保护文件，并发送到攻击者服务器。

对 `APKTool` 的进一步研究，让我们发现了另外一个漏洞，利用该漏洞我们可以在受害者 PC 上执行 OS 命令。

在 `APKTool` 的高级用法中，有一个配置文件，名为“`APKTOOL.YML`”。

这个文件有一个有趣的部分，叫“`unknownFiles`”，用户在这一部分写的非标准的文件位置，之后再重打包 `apk` 的过程中会被发配到正确的地方。这些被选择的文件保存在文件系统的“`Unknown`”文件夹下。

正常的“`APKTOOL.YML`”文件结构如图所示：

通过操控“`unknownFiles`”的路径，就有可能注入任意文件到文件系统中（路径遍历，`Path Traversal`）。因为 `APKTool` 没有验证 `apk` 文件中提取出的 `unknown` 文件。

实际上，注入任意文件到文件系统中会导致远程代码执行（`RCE`，Remote Code Execute）——也就意味着，当 `APKTool` 用户/服务反编译某个静心准备的恶意 `APK` 文件时将有可能受到 `RCE` 攻击。

为了验证，我们开发了 Web 应用，类似于官方 `APK decoder`。

`APKTool` 将解压恶意文件（在这个例子中，我们用了 `PHP` web shell）到服务器的 `wwwroot` 文件夹下。

用那个修改过配置文件重打包 `apk` 之后，这个 `apk` 文件就带上了我们的恶意文件。

上传该 `apk` 到 demo 网站，允许攻击在 web 应用服务器上执行 OS 命令。因此这种服务可以针对使用 `APKTool` 反编译 `apk` 的任意在线/离线服务。

我们用来验证这个漏洞的方式当然只是众多 `RCE` 攻击方式中的一种。实际上，使用路径遍历方法可以复制任意文件到文件系统的任意位置，扩大供给面。

上述的攻击方法均是跨平台且通用的，而且，因为 `APKTool` 在几大操作系统上均可以使用，所以它可以攻击任意系统，只用它的运行不受限制。

尽管无法估计这个开源项目的用户数量。但是，知道其中的一些大公司，我们联系了 `APKTool` 的开发者和 IDE 公司，他们均已报告修复了这个安全问题，并发布了他们产品的更新版本。

原文链接：

本文由 看雪翻译小组 `lumou` 编译

传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！

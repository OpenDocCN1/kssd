
# [翻译]三星Android OS加密漏洞分析：IV重用攻击与降级攻击

## Abstract

基于 ARM 的 Android 智能手机依靠 `TrustZone` 硬件支持可信执行环境 (`TEE`) 来实现安全敏感功能。`TEE` 与 Android 并行运行一个独立的、隔离的 `TrustZone` 操作系统 (`TZOS`)。`TZOS`中的加密功能的实现留给了设备供应商，他们创建了专有的无文档化设计

这项工作揭示了三星 Galaxy S8、S9、S10、S20 和 S21 旗舰设备中 Android 硬件支持的密钥库的加密设计和实现。本研究通过逆向工程、对密码设计和代码结构的详细描述，来揭示了其设计缺陷。

本文提出了一种针对 `AES GCM` 的 IV 重用攻击（`IV Reuse Attack`，允许攻击者提取受硬件保护的 key material）以及一种降级攻击（`Downgrade Attack`）。即使是最新的三星设备也容易受到 IV 重用攻击。此外，还展示了对最新设备的有效密钥提取攻击（`Key Extraction Attack`），以及对 `TrustZone` 和远程服务器之间的两个更高级别加密协议的攻击影响：一个有效的 `FIDO2 WebAuthn` 登录绕过和谷歌安全密钥导入（`Secure Key Import`）的破坏。

## 0x01 Introduction

除了在许多和各种日常活动中的使用，智能手机越来越多地用于许多安全关键任务，例如保护敏感数据（消息、图像、文件）、加密密钥管理、`FIDO2` 网络身份验证、数字版权管理（`DRM`）、移动支付服务（例如，三星支付）和企业身份。

智能手机变得越来越复杂，并呈现出越来越大的攻击面。结果是它们已成为恶意软件和恶意攻击者的主要目标。已经有许多公共漏洞允许攻击者提升 Android 操作系统中的权限，以 root 甚至操作系统内核的身份获得执行。理想情况下，此类攻击不应危及设备的安全关键任务。

可信执行环境 (`TEE`) 主要用于现代移动设备，为可信应用程序 (`TA`) 的执行提供隔离环境，可以安全地执行安全关键任务。他们有一个相对较小的代码库和有限的 API。相比之下，丰富的执行环境 (`REE`)，如 Android 操作系统，不能被完全审计和信任（由于复杂性）。隔离 `TEE` 可与 `REE` 一起使用，以实现安全敏感功能。这使得攻击者更难破坏这些功能，因为攻击面显着减少并且仅限于与 `TEE` 的通信。

换句话说，`TEE` 的目标是抵御来自完全受损的 `REE` 的攻击，包括来自具有内核或根功能的特权对手的攻击。`ARM` 是移动和嵌入式市场中使用最广泛的处理器 ，它通过 `ARM TrustZone`提供 `TEE` 硬件支持。`TrustZone` 将设备分为两个执行环境：
1）运行“`Normal World`”操作系统的非安全 `REE`；
2）运行“`Secure World`”操作系统的安全 `TEE`。

`REE` 和 `TEE` 使用单独的资源（例如，内存、外围设备），并且硬件强制执行 `Secure World` 的保护。在大多数移动设备中，Android 操作系统运行不安全的 `Normal World`。至于安全`Secure World`，还有更多的选择。即使在三星设备中，也至少使用了三种不同的 `TrustZone` 操作系统 (`TZOS`)。

`Android Keystore`通过三星等供应商实施的硬件抽象层 (`HAL`) 提供硬件支持的加密密钥管理服务。`Keystore` 向 Android 应用程序公开 API，包括加密密钥生成、安全密钥存储和密钥使用（例如，加密或签名操作）。三星通过称为 `Keymaster TA` 的受信任应用程序 (`TA`) 实施 `HAL`，该应用程序在 `TrustZone` 中运行。`Keymaster TA` 使用硬件外围设备（包括加密图形引擎）在`Secure World`中执行加密操作。

`Keymaster TA` 的安全密钥存储使用 blob：这些是存储在 `REE` 文件系统上的“wrapped”（加密）密钥。密钥的“wrapping”、“unwrapping”和使用是在 `Keymaster TA` 内使用设备唯一的硬件 `AES` 密钥完成的。只有 `Keymaster TA` 才能访问 key material；`Normal World`应该只看到不透明的键 blob。尽管严格验证和测试这种加密设计是至关重要的，但现实世界的信任区域实现在文献中得到的关注相对较少。这主要是因为大多数设备供应商没有提供其 `TZOS` 和专有 `TA` 的详细文档，并且几乎没有分享有关如何保护敏感数据的信息。本研究对几代三星 `Keymaster TA` 的完整密码设计和 API 进行了逆向工程，并提出了以下问题：
即使在`Normal World`受到威胁时，基于硬件的加密密钥保护是否仍然安全？这种保护的密码设计如何影响依赖其安全性的各种协议的安全性？

![](upload/attach/202203/782560_2QVNTC44ZU6WNU5.png)

研究员于 2021 年 5 月向三星移动安全报告了对 S9 的 IV 重用攻击。2021 年 8 月，三星为该问题分配了高严重性的 `CVE-2021-25444`，并发布了一个补丁程序，通过删除添加自定义 IV 的选项来防止恶意 IV 重用 来自 API。 三星修补的设备列表包括：S9、J3 Top、J7 Top、J7 Duo、TabS4、Tab-A-S-Lite、A6 Plus、A9S。

在 2021 年 7 月报告了针对 S10、S20 和 S21 的降级攻击。2021 年 10 月，三星为降级攻击分配了高严重性的 `CVE-2021-25490`，并修补了搭载 Android P OS 或更高版本的机型，包括 S10、S20、 和 S21。 该补丁完全删除了旧的密钥 blob 实现。

## 0x02 Background

### A. AES GCM

高级加密标准 (`AES`) 是使用最广泛的对称分组密码。`Galois`计数器模式 (`GCM`) 是一种用于块密码的操作模式，可提供经过身份验证的加密。 `AES-GCM` 是一种流密码，它在内部使用 `AES-CTR`（计数器模式）和 `Galois` 消息验证码 (`GMAC`)。

像每个流密码一样，`AES-GCM` 容易受到初始化向量 (`IV`) 重用攻击。当使用相同密钥加密时重用 `IV` 时，生成的密钥流是相同的。在这种情况下，对一种明文的了解会立即揭示另一种明文。此外，在 `AES-GCM` 中，`Joux`展示了如何利用 IV 重用来破坏身份验证并创建新的有效消息。

### B. ARM TrustZone

`ARM TrustZone` 技术添加了一个额外的虚拟处理器模式，称为“`Secure World`”，以补充“`Normal World`”。这两种模式是分开的，可以使用“`Secure Monitor`”（运行在最高 `EL3` 执行级别）或“`World Shared Memory`”的内存映射进行通信。这种分离允许实现 `TEE`，因为受损的`Normal World`将无法访问`Secure World`的内存。下图显示了 `TrustZone` 架构中每个异常级别的组件。

![](upload/attach/202203/782560_3ZZYH3B623JG72W.png)

由于 `TZOS` 的实现留给供应商，因此各个供应商有多种实现方式，包括：

• `Qualcomm` 的 `Qualcomm` 安全执行环境 (`QSEE`)：用于 Google Pixel 设备和三星 Galaxy 设备的 Snap Dragon 型号。

• `Trustonic` 的 `Kinibi`：用于 S10 之前的三星 Galaxy 设备的旧 `Exynos` 型号。

• 华为的 `TrustedCore` (`TC`)。

• 三星的 `TEEGRIS`（用于三星 Galaxy 设备的较新 `Exynos` 型号）。

### C. 可信应用

可信应用程序 (`TA`) 是在 `TEE` 中运行并向 Android 客户端应用程序公开安全服务的程序。应用程序可以打开与 `TA` 的会话并在会话中调用命令。接收到命令后，`TA` 解析输入的命令，执行所需的处理并将响应发送回客户端。控制通过专用的 `SMC`（安全监视器调用）指令传递给 `TA`，`TA` 和 `Normal World` 应用程序通常使用称为 `World Shared Memory` 的共享内存缓冲区交换参数和输出。由于执行 `SMC` 需要 `EL1` 权限，Android 内核中的设备驱动程序处理与 `TA` 的通信并为 `Normal World` 应用程序公开一个 API。

### D. Android 硬件支持的密钥库

`Android Keystore`允许 `Normal World` 应用程序执行加密操作，同时保护加密密钥不被提取或未经授权使用。在采用 `TrustZone` 技术的设备上，`Keystore` 利用 `TEE` 执行加密操作。这种硬件支持的密钥库实现称为 `Keymaster TA`。 `Keystore` 的主要功能是：密钥生成和导入、非对称加密/解密/签名/验证、对称加密/解密和对称 `MAC` 的生成/验证。

![](upload/attach/202203/782560_RMR7XHUWRQ7SPNC.png)

`Keymaster TA` 可以在 `TrustZone` 内生成和使用密钥，因此可以防止任何`Normal World`的攻击者。然而，`Keymaster TA` 依赖于 `Normal World` 应用程序来存储密钥。为了保护 key material，密钥在 `TrustZone` 内使用硬件衍生密钥进行加密或“wrapping”。然后将加密的密钥“blob”传递到`Normal World`进行存储。上图显示了使用 `Keymaster TA` 的 Android 应用程序的简化概述。一般流程如下：
1）Android 应用程序请求生成一个新的密钥。 `Keymaster TA` 生成一个新密钥，并使用硬件衍生密钥对其进行加密。生成的加密密钥 Blob `B` 将传递回 Android 应用程序。
2）Android 应用程序将 blob `B` 保存在 `Normal World` 的文件系统中。
3）`WebAuthn`等一些协议要求 Android 应用程序必须证明特定密钥是由 `Keymaster TA` 安全生成的。在这些情况下，应用程序可以将加密的 blob `B` 传递给 Key master `TA` 并要求它生成证明证书。
4）应用程序可以要求 `Keymaster TA` 代表它执行加密操作。例如，要对消息进行签名，应用程序将加密的 blob `B` 和要签名的消息传递给 `Keymaster TA`。 `Keymaster TA` 解密 blob 以恢复签名密钥。然后它使用密钥对消息进行签名并将生成的签名返回给`Normal World`中的应用程序。

`Keystore` 使用以下措施保护密钥不被提取：
1）应用程序内存中不存在 key material，因此损害应用程序不会导致 key material 提取。
2）`Keystore`可以提供访问控制，以各种方式限制密钥的使用，例如限制密钥的用途（例如，仅加密）、设置到期日期、速率限制或要求用户认证（例如，通过生物特征认证/解锁屏幕）。
3）硬件绑定：支持的设备可以将密钥绑定到安全硬件（即 `TEE`），因此它们不能在该设备上的安全硬件之外使用。

为了支持多个 `TZOS` 实现，Android `Keystore` 使用 `HAL`。每个 `TZOS` 都为需要硬件支持的 Android 服务实现 `HAL`，通常是通过一个 `TA` 来执行所需的操作。三星的 `Keymaster TA` 就是这种情况，它用于实现 Android 硬件支持的密钥库，是研究的主要重点。

### E. Android 密钥认证

`Keystore Key Attestation`允许远程方通过让 `Keymaster TA` 生成根证书为 Google（或三星，用于 KNOX 证明等应用程序）的证书链来验证密钥是否在安全硬件中生成。这允许远程方信任在 `TrustZone` 中生成的公钥而不信任`Normal World`，尽管与 `Keymaster TA` 的所有通信都经过`Normal World`。安全密钥导入和 `FIDO2 WebAuthn` 正是为此目的使用密钥证明（使用 Google 的根证书）。

### F. 攻击模型

在本文中假设攻击者可以完全破坏`Normal World`，例如，具有 root 甚至内核权限的攻击者。此外，假设攻击者能够在不设置由 `Keymaster` 证明的引导加载程序 `fuse`的情况下破坏`Normal World`（例如，引导加载程序解锁，三星的保修位）。攻击者旨在破坏受信任`Normal World`保护的数据，例如 `Keystore` 硬件保护密钥或依赖远程证明的更高级别协议（例如，克隆 `FIDO2` 令牌或通过窃取安全导入的密钥）。`TrustZone` 和 `Keymaster TA` 提供的加密密钥的硬件保护和隔离应该可以防止任何密钥泄露。

请注意，攻击不需要在 Android 内核中实际运行代码。攻击只需要在 `EL0`（Android 用户模式）中执行代码，并具有足够的权限来读取密钥 blob，以及适当的 `SELinux` 权限来与 `TZOS` 驱动程序通信。例如，Android `Keystore` 用户模式守护进程/`HAL`中的漏洞可能就足够了。或者，可以将根恶意软件或修补 `Keymaster HAL` 的供应链攻击用于这两种攻击。

在实验中使用了三星 Galaxy S9、S10 和 S21 设备，这些设备使用 `Magisk`进行根植。请注意，当植根设备时，设置了三星 `KNOX Warranty fuse`。这不会影响攻击，因为不针对任何 `KNOX` 功能。

## 0x03 Dissecting the Keymaster TA

### A. Keymaster TA 家族的调查

重点介绍三星对 `Keymaster TA` 及其名为 `TEEGRIS` 的新 `TZOS` 的实施。与其他 `TZOS` 和 `TA` 一样，三星的实现是特定于供应商的，并且是一个专有的闭源系统，几乎没有文档可用。为了理解 `Keymaster TA` 实现的加密图形设计，使用 `Ghidra`静态分析了 3 个 `TZOS`（`TEEGRIS`、`Kinibi` 和 `QSEE`）中固件和 `TA` 的二进制文件。此外使用三星开源网站为的设备下载 Android 内核源代码。

总体而言，评估了 2018 年和 2021 年间发布的 `Exynos` 和 `Snapdragon` 型号的 S8、S9、S10、S20 和 S21（包括 S9+、Note9、S10+、S20+ 等变体）的 26 个固件。发现`Keymaster TA` 的代码库在这些固件（S8 除外）中极为相似，即使跨 `TZOS`（`Kinibi`/`QSEE`/`TEEGRIS`）也是如此。

在分析中注意到 S8 中的 `Keymaster TA` 与较新型号中的显着差异。S9 中引入的一项特定代码更改使其和所有较新的模型都容易受到攻击。尽管 S20 和 S21 型号包含更安全的 `Strongbox Keymaster` 功能（使用专用的防篡改硬件安全模块），但它们仍然容易受到攻击，因为它们共享相同的易受攻击的密码设计和 API。由于安全社区对 `Kinibi` 和 `QSEE` 进行了更深入的研究，因此在讨论细节时，除非另有说明，否则将参考`TEEGRIS`。

### B. Keymaster HAL

`Keymaster HAL` 是 Android 与特定于供应商的 `Keymaster` 实现之间的接口。 Android 文档为 `Keymaster HAL`的实现者提供了参考指南。它在 Android 用户模式下实现，并使用内核驱动程序和`Normal World`共享内存缓冲区与 `Keymaster TA` 通信（下图）。

![](upload/attach/202203/782560_XE9VNNUF7YWET2J.png)

Android 为 `Keymaster` 应该实现的功能提供了一个开源 API。此 API 包括许多功能，例如密钥生成、密钥导入和加密操作，例如使用存储在加密 blob 中的密钥进行加密/解密/签名/验证。

`TEEGRIS` 中的 `Keymaster HAL` API 在许多使用 `TEEGRIS` 内核驱动程序的未记录共享对象中实现。为了探索 `Keymaster TA`，对 `Keymaster HAL` 进行了逆向工程，并实现了自己的 `Keymaster` 客户端——一个 Android 进程，无需任何输入验证或过滤即可将自定义请求发送到 `Keymaster TA`。

### C. 密钥 Blob 加密

研究的主要焦点是如何解密/加密密钥 blob。基于对专有 `Keymaster TA` 的分析和逆向工程，现在将在高层次上描述该过程。`Keymaster TA` 对 blob 内的 key material 进行加密。这可以保护 key material 不被提取，同时允许其由 `Normal World` 应用程序存储。正如在研究中发现的那样，每个 blob 都包含一个加密部分，其中包括 key material 和各种参数。该 blob 还包含一个明文部分，其中包含解密所需的信息，例如加密中使用的 `IV` 和 `AAD`。

`Keymaster TA` 所依赖的加密基础是永久的、硬件的、设备唯一的 256 位 `AES` 密钥，称为根加密密钥 (`REK`)。此密钥仅存在于安全硬件加密引擎中——`Keymaster TA` 无法直接访问它。每个密钥 blob 都使用从 `REK` 派生的自己的硬件派生密钥 (`HDK`) 进行加密。 Blob 的 `HDK` 是使用密钥导出函数 (`KDF`) 导出的，该函数将 `REK` 与特定于 Blob 的盐值混合。 `KDF` 本身由 `Keymaster TA` 通过 `TZOS` 内部的 `ioctl` API 访问。用于推导 `HDK` 的盐值被计算为几个值串联的 `SHA-256` 摘要。正如接下来讨论的那样，三星的盐值生成方法在设备型号之间演变。

### D. 密钥 Blob 的 KDF 版本

美国国家资讯安全保证联盟 (`NIAP`) 创建了移动设备基础保护配置文件 (`MDFPP`)，其中包括设备的核心安全要求。盐值派生中使用的常量字符串（在所有关键 blob 类型中）表明三星符合认证，而且三星设备确实通过了 `MDFPP CC` 认证。

在分析中确定了三种不同的 blob salt 衍生版本，称之为 `v15`、`v20-s9` 和 `v20-s10`。版本之间的差异在于字符串中使用的值，这些值经过哈希处理以生成用于密钥派生的盐值。所有版本都可以包含称为应用程序 ID 和应用程序数据的可选值。这些值由`Normal World`设置。下图显示了三个 blob 版本中的每一个的示例字符串。

![](upload/attach/202203/782560_PSFDQQ6RYEY56TM.png)


# [原创]浅析安全启动（Secure Boot）

## 前言

安全启动的根本目的是为了防止消费者从软硬件层面对产品的部分关键系统进行读写、调试等高权限的操作。以限制消费者的能力，来达到保护产品的商业机密、知识产权等厂家权益的目的。当然，厂家是不会这样宣传 Secure Boot 的。他们的文案通常都是通过这项技术保护用户的隐私，防止恶意软件修改系统软硬件等等。不过不论如何，随着 ARM 架构的广泛授权，基于 TrustZone 的 Secure Boot 也越来越普遍了。本文是通过我自己对市面上的一些基于 ARM TrustZone 的 Secure Boot 实现的浅见、零零碎碎读到的一些安全分析文章、看到的一些讲座，总结一些常用的攻击思路，介绍一些真实的攻击案例。

## 一、当消费者成为威胁

可以说，Secure Boot 的安全模型建立在消费者是攻击者这一假设上。消费者在物理上拥有产品硬件，可以对产品进行物理连接、拆机、改装等等物理上的操作。可以说跟传统的安全模型中的攻击者相比根本不在一个层面上。

消费者作为攻击者的目的，一般常见的有刷机安装自定义的操作系统（Mod）、绕过厂家封闭的支付平台（IAP）和应用商城安装自定义的应用程序、绕过版权保护系统（DRM）达到复制厂家保护的数字产品内容等等。这些操作往往都会直接影响厂家的利益，因此需要一种能抵抗消费者攻击的安全机制。

上面提到了，消费者可以直接拆机干涉产品的硬件模块。比如比较专业的消费者甚至可以使用数字示波器监听 CPU 和 RAM、eMMC 之间的数据传输来读取非常底层的数据传输。如下图所示，我拿某厂的电视盒子的 PCB 拍了张照方便大家理解，如图中圈出来的，CPU、RAM、eMMC 都是分开的芯片，中间的连接电路是有可能被监听的，有些研究人员可以焊接极细的导线到电路上，来监听、拦截、甚至篡改不同芯片之间数据的传输。

![](upload/attach/202007/898032_57HNAGNQUPBC3NX.png)

而且像 eMMC 这种芯片通常都是业界标准化的，攻击者甚至可以把芯片拆下来，然后用市面上现成的通用 eMMC 编程工具来读写上面的内容。

不过业界还是给攻击者的能力设置了一个上限。这个上限通常是认为攻击者不至于能够剥离芯片的封装，然后用电子显微镜等纳米级别精度的显像设备来逆向芯片的内部结构。或者说能成功攻破芯片安全机制的一次性投资成本至少需要在十万美元以上才可以认为是安全的。

Secure Boot 安全机制的原理，就是将最为核心的安全机制整合到最关键的主 CPU 中。因此就算攻击者可以监听电路板上的线路，甚至拆装个别芯片单独调试，也无法破坏 Secure Boot 的安全机制。

## 二、CPU 内部的安全机制

下图是一个比较粗略的，各个启动阶段，软硬件组件之间的关系。这只是我接触比较多的一种架构，当然还有别的不同的架构，因为接触得不多这里就不列举了。

![](upload/attach/202007/898032_AB4XHQFUGJN3GRJ.png)

### 1）bootROM

所有支持 Secure Boot 的 CPU 都会有一个写死在 CPU 中的 bootROM 程序。CPU 在通电之后执行的第一条指令就在 bootROM 的入口。bootROM 拥有最高的执行权限，也就是 EL3。它将初始化 Secure Boot 安全机制，加载 Secure Boot Key 等密钥、从 eMMC 加载并验证 First Stage Bootloader（FSBL），最后跳转进 FSBL 中。

bootROM 是完全只读的，这个在 CPU 出厂时就被写死了，连 OEM 都无法更改。bootROM 通常会被映射到它专属的一块内存地址中，但是如果你尝试向这块地址写入内容，一般都会出错或者没有任何效果。

有些芯片还会有一个专门的寄存器控制 bootROM 的可见性，bootROM 可以通过这个寄存器禁止别的程序读取它的代码，以阻止攻击者通过逆向 bootROM 寻找漏洞。

### 2）iRAM

为了避免使用外部 RAM 芯片，支持 Secure Boot 的 CPU 上都会内置一块很小的 RAM，通常只有 16KB 到 64KB，我们称之为 iRAM。这块 iRAM 上的空间非常宝贵，bootROM 一般会用 4KB 的 iRAM 作为它的堆栈。FSBL 也会被直接加载到 iRAM 上执行。

### 3）eFUSE

所有支持 Secure Boot 的 CPU 都会有一块很小的一次性编程储存模块，我们称之为 FUSE 或者 eFUSE，因为它的工作原理跟现实中的保险丝类似：CPU 在出厂后，这块 eFUSE 空间内所有的比特都是 1，如果向一个比特烧写 0，就会彻底烧死这个比特，再也无法改变它的值，也就是再也回不去 1 了。

一般 eFUSE 的大小在 1KB 左右，OEM 从 CPU 厂家购买了芯片，组装了产品后，一般都要焼写 eFUSE 的内容，包括产品的运行模式：测试、开发、生产等。面向终端消费者的产品都会被焼写为生产模式。这个模式下 bootROM 会禁用很多权限，更大面积地限制用户的能力。

另外一个很重要的焼写内容就是根密钥了，一般有两种根密钥：一个是加密解密用的对称密钥 Secure Boot Key，一般是 AES 128 的，每台设备都是随机生成不一样的；另一个是一个 Secure Boot Signing Key 公钥，一般用的 RSA 或 ECC，这个是每个 OEM 自己生成的，每台设备用的都一样，有些芯片会存公钥的 Hash 来减少 eFUSE 的空间使用。

只有 Secure World（后面会介绍）才能访问 eFUSE 的寄存器。除了读写 eFUSE 的基础寄存器之外，还有一些控制寄存器可以禁止别的程序访问 eFUSE，来保护其中的密钥。因此 eFUSE 中的根密钥以及 bootROM 将作为 Secure Boot 的根信任。

### 4）Security Engine

有些 CPU 中还会有一个专门负责加密解密的模块，我们称为 Security Engine。这个模块也是只有在 Secure World 中才能访问。这个模块通常会有若干个密钥槽（Keyslots），可以通过寄存器将密钥加载到任意一个 Keyslot 当中，Keyslot 的加载操作将复盖之前加载过的密钥。通过寄存器操作 DMA 读写，可以使用 Keyslot 中的密钥对数据进行加密、解密、签名、HMAC、随机数生成等操作，但是没有办法从一个 Keyslot 中读取已经加载的密钥值。

### 5）First Stage Bootloader（FSBL）

FSBL 的作用是初始化 PCB 板上的其他硬件设备，给外部 RAM 映射内存空间，从 eMMC 的 GPT 分区上加载验证并执行接下来的启动程序。

### 6）根信任的建立

CPU 通电后执行的第一行指令就是 bootROM 的入口，bootROM 将初始化各种 CPU 内部的模块，但最主要的是，它会读取 eFUSE 上的内容，首先它会判断当前的运行模式是不是生产模式，是的话会开启 Secure Boot 功能，然后把 Secure Boot Key 加载到一个 Security Engine 的 Keyslot 当中，有时候它还会通过 Key Derivation 从 Secure Boot Key 或别的 eFUSE 内容生成多几个不同用途的密钥，分别加载到不同的 Keyslots 中。然后它会从 eMMC 上加载 FSBL，FSBL 里面会有一个数字签名和公钥证书，bootROM 会验证这个签名的合法性，以及根证书的 Hash 是否和 eFUSE 中的 Signing Key 的 Hash 相同。如果验证通过，说明 FSBL 的的确确是 OEM 正式发布的，没有受到过篡改。于是 bootROM 就会跳转到 FSBL 执行接下来的启动程序。有些 CPU 在跳转之前会把 bootROM 的内存区间设为不可见，防止 FSBL 去读取 bootROM。有些 CPU 还会禁止 eFUSE 的读写，或者至少 Secure Boot Key 区域的读取权限，来防止 FSBL 泄漏根信任的解密密钥。还有要注意的是，FSBL 是被加载到了 iRAM 上执行的，而且 FSBL 仍然拥有 EL3 级别的权限。

FSBL 会进一步初始化 PCB 板上的别的硬件，比如外部的 RAM 芯片等等，使其不再受限于 iRAM 的内存空间。然后它会进一步加载 eMMC 上的内容到 RAM。我们接下来会着重讲讲跟 Secure Boot 密切相关的启动内容。

## 三、信任链的延伸

一般 FSBL 都非常靠近 eMMC 的第一个扇区，bootROM 一般不会管 eMMC 有没有用 GPT 分区，而是简单的从第一个扇区读取一些头信息然后直接找到 FSBL 的扇区进行加载的。所以一般我们要做 GPT 分区的话都会放在 FSBL 后面。FSBL 加载一个 GPT 分区也是要跳过 eMMC 前面的非 GPT 扇区再进行加载。

### 1）TrustZone

从 FSBL 开始我们就要引入 TrustZone 的技术概念了。TrustZone 是 ARM 家的 Trusted Execution Environment（TEE）实现。相对应的，Intel 家的叫做 Software Guard Extensions（SGX），AMD 家的叫做 Platform Security Processor（PSP）。虽然本文只关注 TrustZone，但其实原理上都大同小异。

TrustZone 将 CPU 的执行环境划分为了 Secure World 和 Normal World，Secure World 拥有更高的权限，可以访问 Normal World 的内存、eFUSE 寄存器、Security Engine 寄存器等等。但是 Normal World 中的最高权限却无法访问 Secure World 的内存。

但是 Normal World 可以通过 `SMC` 指令调用 Secure World EL3 中注册的命令，然后 EL3 程序再将 Normal World 的请求转发给 Secure World，从而达到两个世界之间的通讯。

### 2）Secure Monitor

这是一个运行在 Secure World EL3 权限中的程序，Secure Monitor 的主要作用就是注册 `SMC` 指令的回调，转发两个世界之间的消息。

### 3）Trusted OS

这是一个通常运行在 Secure World EL1 权限中的内核程序，比较常见的是基于开源的 ARM Trusted Firmware 进行扩展修改的，别的实现还有基于 Little Kernel 的，以及一些芯片厂家自己的实现。它的主要作用是给 Secure World 中运行的程序提供一个基本的系统内核，实现多任务调度、虚拟内存管理、System Call 回调、硬件驱动、IPC 通讯等等。

### 4）Trusted Application（TA） / Trustlet / Secure Service

这些是在 Secure World EL0 中运行的程序、Daemon。根据不同的 Trusted OS 的实现，TA 的格式、运行模式也会有所不同，比如 ARM Trusted Firmware 的 TA 使用的是 ELF 格式，而高通的 TA 用的是自家的格式。

### 5）Keybox

因为 eFUSE 的大小有限，而且不可更改，有些体积比较大的，或者有可能更新的密钥数据，包括像各种 DRM 系统、HDCP、Attestation 等等的密钥，都会用 Secure Boot Key 或者一个 Derived Key 进行加密后储存在 eMMC 的一个 GPT 分区上，因此基本上没什么大小限制。FSBL 一般会将这个分区加载到 RAM 中，然后 TA 再通过 Security Engine 中已经载入的 Keyslot 对其进行解密获得相应的密钥。

### 6）Second Stage Bootloader（SSBL）

SSBL 的作用是给 Normal World 做初始化，加载验证并执行 Linux/Android 内核。常见的 SSBL 是基于 U-Boot 项目改的，通常这里已经支持像 Fastboot、Recovery boot 等等的常见启动对象了。

### 7）搭建信任链

FSBL 的下一阶段一般包含三个程序：Secure Monitor、Trusted OS、Second Stage Bootloader。FSBL 会分别从 eMMC 加载验证这些程序，最后跳转到 Secure Monitor 执行。

Secure Monitor 会初始化 TrustZone 环境，设置 SMC 回调，然后开始执行 FSBL 已经加载好的 Normal World 中的 Second Stage Bootloader，和 Secure World 中的 Trusted OS。

Trusted OS 会初始化 Secure World 中的系统内核环境，然后从 eMMC 加载各种 Trusted Applications（TA），认证它们的数字签名，然后执行它们。每个 TA 都会有自己独立的虚拟内存空间。TA 如果要访问特定的硬件，比如 Security Engine，会在一个描述 TA 的头信息中申请，TA 的签名也会涵盖这部分头信息。Trusted OS 如果同意 TA 访问这些硬件，会把对应的寄存器地址 mmap 映射到 TA 的虚拟内存空间中。

Second Stage Bootloader 也就是我们一般说的 Android Bootloader，它会加载验证然后执行 Android Kernel。

可以看出来，从 bootROM 到 Normal World 的 Kernel，到 Secure World 的 TA，每一步的加载都是要经过数字签名认证的，而所有这些签名认证的根证书，是要跟 eFUSE 中的 Signing Key 匹配的。这就形成了一个信任链。

## 四、信任链攻击思路

回顾我们前面定义的威胁模型，如果攻击者的目的是刷自定义 ROM，那至少要同时拿到 Normal World 和 Secure World 的 EL1 权限才能勉强让一个自定义 ROM 正常运作。如果攻击者的目的是破坏由 TrustZone 保护的 IAP 支付机制，或者 DRM 保护机制，则至少要拿到实现这些保护机制的 TA 的权限才行，也就是至少要拿到 Secure World EL0 的权限。

目前我接触到的攻击思路，我基本上分为两类：Top Down 和 Bottom Up。

### 1）Top Down 攻击思路

所谓 Top Down，就是从最上层的程序，也就是最低级的权限一步步提权，每一次提权就获得更底层一点的权限，慢慢渗透到目标权限层。

一个特别适合了解 Top Down 的案例来自 Quarkslab 的 <a href="elink@fd8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2&6L8%4g2@1N6h3u0W2i4K6u0W2j5$3!0E0i4K6u0r3N6$3q4@1j5$3S2Q4x3@1k6$3i4K6y4p5N6g2S2t1y4f1I4v1c8#2u0%4h3p5V1`.">Breaking Samsung's ARM TrustZone</a> （<a href="elink@e5bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6i4K6u0W2j5X3I4S2j5$3E0Z5j5i4c8Q4x3X3g2U0L8$3#2Q4x3V1k6g2f1@1q4Q4x3X3b7I4z5g2)9J5c8W2c8Z5N6i4u0K6k6r3q4&6i4K6u0r3N6i4y4Q4x3X3b7I4z5g2)9J5k6q4m8W2N6r3g2J5L8r3W2F1i4K6u0V1b7Y4u0W2j5h3E0A6L8X3N6Q4x3X3c8e0j5h3#2K6N6h3&6Y4M7#2)9J5k6p5q4d9e0g2)9J5k6q4c8J5N6i4y4@1h3X3!0F1k6g2)9J5k6i4m8V1k6R3`.`.">PDF</a> & <a href="elink@325K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6I4N6h3q4J5K9%4y4D9j5h3u0Q4x3V1k6K6j5h3#2K6N6h3&6Y4i4K6u0V1N6s2u0#2M7%4c8\*7L8$3&6W2i4K6u0V1M7X3g2K6k6h3q4J5j5$3R3`.">GitHub</a>）

![](upload/attach/202007/898032_C2DVZHWJ56UCJBD.png)

这个案例他们假设一开始只有 Normal World 中 EL0 的权限，但是可以自己写程序调用 TrustZone 的 Driver，通过 Driver → Android Kernel → SMC → Secure Monitor → Trusted OS Kernel → Trusted Application 这条线路调用 TA 的相关功能。他们逆向了 TA 的代码，找到了一处 `memcpy` 越界漏洞，从而拿到了该 TA 的 Secure World EL0 权限。

但是这个 TA 的权限有限，他们又通过 Trusted Application → System Call → Trusted OS → Secure Service 的线路调用同是 Secure World EL0，但是有更多 System Call 权限的一个 Secure Service。他们同样在这个 Secure Service 中找到了一处 `memcpy` 越界漏洞，从而拿到了更高权限的 Secure World EL0 执行权限。

然后他们发现这个 Secure Service 的其中一个 System Call 是一个任意地址 `mmap`，而且没有任何限制。于是他们可以直接把 Secure Monitor 的物理地址直接 `mmap` 到 Secure Service 的虚拟地址空间，然后直接改写 Secure Monitor 的代码，直接拿到 Secure Monitor 所在的 Secure World EL3 权限。基本上来说，拿到 EL3 权限，就已经可以做到任何想做的事情了。

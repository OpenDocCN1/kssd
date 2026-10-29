
# iOS 签名杂谈（一）

为什么要说 iOS 的签名呢？现在移动平台的逆向教程和书籍已经相当多了。针对签名的文章也很多，我这里想说的一些是可能别的地方看不到的比较细微的内容（虽然都是老黄历~~）。

iOS 的签名目的其实也比较纯粹，就是为了能够在不越狱的情况下安装破解版的 `ipa`。当然，如果是各种助手的话还有另外的一个目的，那就是应用分发（更重要的是在分发之前加入自己的广告 `sdk`）。

说到 iOS 的应用分发其实主要方式有如下几种：

1. 苹果的应用商店。
2. `cydia` 应用商店。需要越狱之后才能安装各种 `app` 和插件，并且由于现在越狱基本都是不完整越狱，重启设备之后需要重新越狱。并且越狱工具安装也异常麻烦，所以越狱的用户也少了很多
3. 第三方应用商店，国内的比较大的就那么几家。不知道的可以自己搜索一下。

第三方应用商店的 `app` 分发其实也经历了几个时期：

- a. 越狱时期，最早期应用商店分发的基本都是越狱应用。这个与早期的越狱插件和完美越狱存在比较大的关系。
- b. 转授权分发，这个技术最早貌似是 `360` 的快用用的这么一项技术。所谓转授权就是通过链接电脑，通过 `itunes` 的相关 `api` 调用在设备上创建 `IC-info` 文件。通过苹果的应用商店下载的 `iap` 会包含 `sc_info` 授权信息。

在 `ipa` 安装的过程中并不会校验设备上有没有授权信息，只有到运行的时候才会校验授权信息。此时如果没有授权那么会弹出要求输入用户名和密码的弹框。

如果要验证上面的内容。有一个简单的做法，那就是删除设备上的授权文件。除了要删除上面的 `iTunes` 目录下的文件还要删除以下的文件：

> `/private/var/mobile/Library/FairPlay/iTunes_Control/iTunes/IC-Info.sidr`
> `/private/var/mobile/Library/FairPlay/iTunes_Control/iTunes/IC-Info.sids`
> `/private/var/mobile/Library/FairPlay/iTunes_Control/iTunes/IC-Info.sidt`
> `/private/var/mobile/Library/FairPlay/iTunes_Control/iTunes/IC-Info.sisb`
> `/private/var/mobile/Library/FairPlay/iTunes_Control/iTunes/IC-Info.sisv`
> `/private/var/mobile/Library/FairPlay/iTunes_Control/iTunes/IC-Info.sidb`

`/iTunes_Control/iTunes/iTunesControl` 目录下的文件及时删除也不会影响应用的正常云信，并且在启动 `app` 之后下面的文件会进行重建。

- c. 企业签名阶段，早期的企业签名成本还是比较低的。由于 `dylib` 注入导致应用可以随意修改，加入广告。所以各种助手就有动力来进行企业签名分发。早期苹果证书封锁的也比较慢，所以收益还是非常可观的。
- d. 个人证书签名阶段。受苹果证书封锁越来越频繁的影响。于是助手们又开始转战个人证书，个人证书包括个人开发者证书和个人 `appleid`。最开始的时候苹果的 `appleid` 创建的 `mobile provision` 文件是一个月的有效期。所以不少助手为了成本都开始使用个人签名。后期有效期变为 `7` 天之后，又开始转战个人开发者签名。

通过 `appstore` 下载 `ipa`，早期的 `itunes` 通过两步进行的。

1. 下载 `ipa` 文件，此时的文件是没有任何的授权信息的，就是加密后的 `ipa` 文件。
2. 通过接口创建授权文件以及相关目录 `SC_Info`，该目录位于 `ipa` 的 `Payload/EmojiUltimate.app` 目录下。并且同时创建 `iTunesMetadata.plist` 文件。该文件包含了 `ipa` 购买的 `apple id` 的相关信息。

`SC_info` 目录结构：

关于 `ipa` 结构的文章可以看这个链接：`<mark class="encrypted">414K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2M7X3q4*7j5W2)9J5k6h3#2W2i4K6u0r3M7s2g2D9L8r3W2F1k6#2)9J5k6r3q4H3j5i4u0@1i4K6u0V1j5h3&6Q4x3X3c8A6L8%4y4Q4x3X3c8S2M7s2m8Q4x3V1j5`.</mark>

旧版的 `ipa` 在 `Frameworks` 目录下的支持库下并没有独立的 `SC_info`。新版的 `ipa` 在所有的库目录下都创建了 `SC_info`，具体是哪个版本变更的，由于长时间没有接触这个，我也不知道~~

虽然存在多个文件，但是文件的内容是一样的，通过 `md5` 就可以比对出来了，当然现在 `md5` 碰撞比较容易实现了，但是参考一下还是可以的。

3. 将授权文件，购买信息打包到 `ipa` 内。这个就是通过 `itunes` 最终下载到的 `ipa`。现在最新的 `itunes` 已经没有应用商店了~~

通过该文件可以看到，购买的 `appleid` 为 `cntwaymm@126.com`。各种助手也是通过该文件来判断应用的版本信息，是否是苹果的官方应用（如果要准确判断还需要依赖 `SC_info`）。不过在安装的过程中该文件并不参与校验。即使删除该文件也可以正常安装，并且该文件不参与签名。

众所周知，苹果一向以安全著称，那么既然最后的 `ipa` 是通过拼接合成的，那么会不会在下载的过程中被篡改？或者加入一些其他的功能？其实苹果早就想到了这一点了。即使是应用商店下载的 `ipa` 也是带数字签名的。

`ipa` 的签名信息分为两部分：

1. `ipa` 内所有文件的签名信息
2. 可执行文件的签名信息。

`ipa` 文件信息签名包括资源签名都位于 `Payload/EmojiUltimate.app/_CodeSignature` 下的 `CodeResources` 文件内：

文件部分内容：

对应文件的哈希值为 `sha1 + base64`，较新的 `ipa` 同时还会有 `sha256 + base64` 的签名方式。

新版签名数据

计算方法：

```python
with open(filepath, 'rb') as f:
    sha1obj = hashlib.sha1()
    sha1obj.update(f.read())
    hash = sha1obj.hexdigest()
    print("sha1:", hash)
    bs = base64.encodebytes(sha1obj.digest())
    print("hash:", bs)
```

通过计算可以发现，数值与 `plist` 文件中的数值是一样的：

看到这里，既然已经知道了计算方法，那么是不是可以修改文件，更新哈希值之后进行安装呢？如果尝试以下你就会发现这条路行不通！为什么？哈希明明是对的？

那是因为虽然当前文件的哈希是对的，但是由于文件内容变化，导致整个 `CodeResources` 文件的哈希值变了。而这个文件的哈希值则是记录在可执行文件的签名信息中的。

通过 `jtool` 可以便捷查看对应的签名信息：

标记的地方就是 `CodeResources` 的哈希值，通过文中的 `python` 代码也可以快速的计算该文件的 `sha1`，两者是一致的。

如果是新版本的 `ipa` 则还会有 `sha256` 的值。

这也就解释了，为什么下载的 `ipa` 是不能随便修改的，当然修改二进制文件最后的效果也是一样的，无法通过签名校验。

虽然如此，还是有一部分文件是可以修改的，那就是 `SC-info` 以及目录下的相关文件。这些文件对应的是记录的 `ipa` 的购买信息，以及授权信息。那么如果用两个不同的账号购买下载同一款应用，替换内部的 `sc-info` 文件是可以正常安装运行的。

至于为什么能通过签名校验，这个放在下一篇吧。

苹果开源代码中，关于签名的代码：`<mark class="encrypted">97bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6G2M7r3g2F1M7$3!0#2M7X3y4W2i4K6u0W2j5i4m8H3L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3V1k6e0k6h3y4#2M7X3W2@1P5g2)9J5c8W2y4W2j5%4g2J5K9i4c8&6i4K6u0V1y4e0f1@1y4K6q4Q4x3V1k6K6k6h3y4Q4x3V1k6e0k6h3y4#2M7X3W2@1P5g2)9J5c8W2c8G2L8$3I4Q4x3V1k6U0L8$3c8W2M7$3W2Y4L8W2)9J5k6h3y4Q4x3X3g2S2N6i4c8G2i4K6u0W2K9s2c8E0L8l9`.</mark>

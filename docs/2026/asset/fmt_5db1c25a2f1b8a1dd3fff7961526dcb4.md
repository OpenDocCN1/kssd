
# iOS 签名杂谈（二）

![wild\_bundle\_id](upload/attach/202006/211565\_Z9RE75MUAMSPHQ8.png)

具体有开发者账号的可以自己登陆苹果开发者后台查看对应的区别。另外如果没有开发者账号，对于普通的appleid试用开发者功能的`mobileprovision`文件创建不在本文的讨论范围内。文件创建流程可以自己跟踪xcode进行观察。

另外对于实际的签名过程至于是开发的`mp`文件还是分发的`mp`文件，在实际的使用效果中区别比较大的是推送模块。如果重新签名之后的文件需要使用推送功能，那么建议使用分发模式的`mp`文件， 并且在`bundleid`中配饰`push`功能并增加对应的`push`证书。

`mp`文件的主要作用有三个：

- 定义了`ipa`的`entitlement`权限信息
- 定义了授权设备列表。
- 授权文件自身的一些关键信息：有效期，开发者信息，`bundleid`信息，`uuid`信息，授权证书信息。

`mp`文件本身是一个二进制文件，如下所示：

![mobile provision](upload/attach/202006/211565\_SN5TX868QX3FN3C.png)

下面分别来说明这些信息：

----

### entitlement权限信息

该信息主要是定义了`ipa`的权限信息，并且在签名的过程中该权限信息虽然不要求与`ipa`二进制文件的权限信息完全一致，但是如果完全不一致可能会导致安装失败，闪退等问题。

osx的`mp`文件预览可以直接查看权限信息。

![mp enttitlement](upload/attach/202006/211565\_J5A3S2NG4T5WE9D.png)

另外通过`security`命令可以查看`mp`中的文本信息：

```bash
cms -D -i /Volumes/DATA/python\_projects/ios\_code\_sign/飞机流量查询\ FlightRadar24\ Pro-6.7.0\(越狱应用\)\_RESIGNED/Payload/Flightradar24.app/embedded.mobileprovision
```

可以看到展示的信息和预览是一致的：

![security](upload/attach/202006/211565\_77S2A599VRJR8ZB.png)

对于签名后的`ipa`可以通过`ldid`的命令查看签名后的`entitlement`信息：

```bash
ldid -e /Volumes/DATA/python\_projects/ios\_code\_sign/飞机流量查询\ FlightRadar24\ Pro-6.7.0\(越狱应用\)\_RESIGNED/Payload/Flightradar24.app/Flightradar24
```

内容如下：

![ldid](upload/attach/202006/211565\_ZQGF7SK5VAN35KN.png)

通过对比可以发现签名后的二进制文件的`entitlement`和`mp`中的`entitlement`文件内容大部分还是一致的。区别是并没有存储在一个单独的`<key>Entitlements</key>`目录下。

前面曾经提过并不要求`bundleid`完全一致，我这里的`bundleid`是`49U329UV5Q.mars.com.giants-software.fs18`，在苹果开发者后台创建的`bundleid`为`mars.com.giants-software.fs18`，但是在创建`mp`文件的时候系统会自动将开发者的`id`添加到`bundleid`前面。

而重签名的应用的`bundleid`是`se.resenatverket.FlightRadar24`，如下：

![bundleid](upload/attach/202006/211565\_E32ZXZ22RNY8SWC.png)

虽然两者不一样，但是不会影响正常的安装。并且对于使用自定义`resourcerules`签名的`ipa`通过`codesign`进行校验的时候会展示如下信息：

![codesign-v](upload/attach/202006/211565\_GBGVNUSQV6Z2D95.png)

签名结果显示使用了自定义的忽略规则，通过这个命令也可以知道`ipa`是否使用了自定义的`resourcerules`。

所以签名之后如果不能安装并不是`bundleid`的问题，可以从其它方面找一下原因。

另外提一下，虽然`ldid`有个`-s`参数可以给二进制写入`entitlement`信息，但是不建议通过该命令给`ipa`签名。如果是越狱系统不想使用`codesign`可以使用`ldid`给二进制文件写入权限，例如给`debug-server`写入权限。

如果是`ipa`签名可以通过`codesign`的`--entitlements path`参数写入`entitlement`信息：

> `--entitlements path`
>
> When signing, take the file at the given path and embed its contents in the signature as entitlement data. If the data at path does not already begin with a suitable binary ("blob") header, one is attached automatically.
>
> When displaying a signature, extract any entitlement data from the signature and write it to the path given. Use "-" to write to standard output. By default, the binary "blob" header is returned intact; prefix the path with a colon ":" to automatically strip it off. If the signature has no entitlement data, nothing is written (this is not an error).

更多关于`codesign`的参数可以通过命令`man codesign`查看详细说明，如果感觉不够详细可以查看苹果的官方文档。

----

### 授权设备列表

该信息存在于`mp`文件中的`ProvisionedDevices`字段下。对于授权设备列表有两种类型，这两种类型其实最根本的区别在于企业开发者和个人开发者的区别。

对于个人开发者只有一种形式，那么就是设备列表的形式，格式如下：

![ProvisionedDevices](upload/attach/202006/211565\_W9R5PEG5B7HUZMX.png)

该列表定义了那些设备可以安装签名后的`ipa`文件，对应的数据内容为设备的`uuid`，获取`uuid`主要有两种方法，1：使用各种助手，2：通过安装设备描述文件。具体的实现方法感兴趣可以自己搜索。

如果不在该列表内，那么签名后的`ipa`是无法安装的。

上面提到的是个人开发者，那么企业开发者还有另外一种形式，如下所示：

![all-devices](upload/attach/202006/211565\_E5BTZ57K58J5DW4.png)

对比就可以发现两者的区别，企业的`mp`文件多了一个`ProvisionsAllDevices`字段，去掉了`ProvisionedDevices`字段。新加的字段意味着，只要签名合法，那么所有的设备都可以安装签名后的`ipa`文件。这也是为什么企业开发者签名成了个大助手应用分发的主要方式。

---

### 授权文件自身的一些关键信息

其实该字段主要的信息是`mp`文件自身的有效期，该信息通过两部分实现，一个是创建日期，另外一个是`ttl`。展示为如下的两部分：

`CreationDate`：

![CreationDate](upload/attach/202006/211565\_H6N6WGJ94CT5X47.png)

`TTL`：

![ttl](upload/attach/202006/211565\_GF7F482UESQFDCA.png)

通过上面两部分就可以计算出`mp`文件的过期时间了。如果`mp`文件过期以后，签名的`ipa`就无法正常安装了。然而，`mp`过期并不需要重新签名，如果使用前面的自定义签名规则文件签名，只需要重新创建一个同样的`mp`文件压缩到`ipa`内替换即可继续安装了。

这里说的一样，意思是：

- 使用的同样的开发者账号，同样的`bundleid`创建的`mp`文件
- 开发者后台`bundleid`下的权限尽量不要修改，保持与过期之前一致。
- 开发者证书没有过期，并且开发者账号没有被封。

贴一下项目地址：

<<mark class="encrypted">a0eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6G2j5X3q4T1P5g2)9J5c8X3W2G2M7#2)9#2k6X3y4G2k6r3g2Q4y4h3k6K6K9h3N6F1</mark>>

一二中用到的工具和样本已经在项目中包含，有需要的可以自行下载。

第三篇的时候会简单的写一下，osx上的签名技巧。

---

上一篇中提到替换`SC-info`后的`ipa`依旧可以安装，至于原因其实也很简单。仔细查看`CodeResources`文件，就知道原因了。文件内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "283K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4H3M7r3I4W2i4K6u0W2j5$3!0E0i4K6u0r3c8q4c8p5M7#2)9J5c8W2m8J5L8%4m8W2M7Y4c8&6e0r3W2K6N6q4)9J5k6o6q4Q4x3X3f1H3i4K6u0W2k6s2c8V1">

<plist version="1.0">

<dict>

    <key>files</key>

    <dict>

        <key>EmojiUltimateViewController.nib</key>

        <data>

        ifQreSMmB3Uft3K3EF1/RLPF/zw=

        </data>

        <key>EmojiViewController.nib</key>

        <data>

        UUTdU18Tz6Fl6x7E2aGYcRxpH3A=

        </data>

        <key>English.lproj/Localizable.strings</key>

        <data>

        53Ka23KM2gkM/XGA2/7hVBG5jfc=

        </data>

        <key>MainWindow.nib</key>

        <data>

        UshXxpapPiP6lBtWe18OpnMv+lM=

        </data>

        <key>PkgInfo</key>

        <data>

        n57qDP4tZfLD1rCS43W0B4LQjzE=

        </data>

        <key>RemoteConfig.plist</key>

        <data>

        rQXI2OqpnGVOk5+BmA/F++FnAG0=

        </data>

        <key>de.lproj/Localizable.strings</key>

        <data>

        MAGm4wWT2b8996eIiqcJna3Jj5s=

        </data>

        <key>emoji.plist</key>

        <data>

        aybwK3Wkthu1Gk7XmEzyjXmGVRQ=

        </data>

        <key>emoji_ultimate_copy_button.png</key>

        <data>

        M7AjbQ3xPvVnFz4PB6kviXR9ePQ=

        </data>

        <key>emoji_ultimate_copy_button_background.png</key>

        <data>

        e7CSi/nIgilN4Skvg4kTVD+Un3M=

        </data>

        <key>emoji_ultimate_icon.png</key>

        <data>

        QREK+EwPbUuOJZf7HrAtqDG1a3o=

        </data>

        <key>emoji_ultimate_icon@2x.png</key>

        <data>

        ZRcuLgqQkQmFRh43aBR8snXi9HU=

        </data>

        <key>emoji_ultimate_view_background.png</key>

        <data>

        SPCyOB9McmjA9cYYma6lkXe23WE=

        </data>

        <key>es.lproj/Localizable.strings</key>

        <data>

        3ob4L+OaiqknLRq7FoZRSBvUIyU=

        </data>

        <key>fr.lproj/Localizable.strings</key>

        <data>

        t0XzVIa6xqvmp5KagcjmB3xR7pg=

        </data>

        <key>it.lproj/Localizable.strings</key>

        <data>

        anq4JMyR1PBXVZf50ZSs1DZo/Yg=

        </data>

        <key>nl.lproj/Localizable.strings</key>

        <data>

        c8FRK0aJIXOqcXMwQnmDmw3fWBQ=

        </data>

        <key>pt.lproj/Localizable.strings</key>

        <data>

        p2Bu2xab7q2mOKGkS3ilMjlbkg4=

        </data>

        <key>zh-hans.lproj/Localizable.strings</key>

        <data>

        GDxwv2JLXA1oNSOphz4DndPk5LM=

        </data>

        <key>zh-hant.lproj/Localizable.strings</key>

        <data>

        c3wmlLx60JrmuFeU/hOcLdepdAM=

        </data>

    </dict>

    <key>rules</key>

    <dict>

        <key>.*</key>

        <true/>

        <key>Info.plist</key>

        <dict>

            <key>omit</key>

            <true/>

            <key>weight</key>

            <real>10</real>

        </dict>

        <key>ResourceRules.plist</key>

        <dict>

            <key>omit</key>

            <true/>

            <key>weight</key>

            <real>100</real>

        </dict>

        <key>^SC_Info/.*\.sinf$</key>

        <dict>

            <key>omit</key>

            <true/>

            <key>weight</key>

            <integer>10000</integer>

        </dict>

        <key>^SC_Info/.*\.supp$</key>

        <dict>

            <key>omit</key>

            <true/>

            <key>weight</key>

            <integer>10000</integer>

        </dict>

    </dict>

</dict>

</plist>

```

该文件中除了定义的参与哈希运算的文件，同时还排除了一部分文件，排除的文件列表就是对应的`rules`下的内容：

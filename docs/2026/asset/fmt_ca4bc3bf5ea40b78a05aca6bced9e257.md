
# [原创] 某 iOS 游戏抓包、修改

这是一款音乐节奏型的游戏，游戏玩法为本地和联网混合。最近它更新了 v4.0.255 版本，应该<|message|>... 借此机会我们来研究一下。

## IPA 提取拆包

预备条件：已越狱的 iOS 设备。

1. 从 App Store 下载安装游戏

2. 使用 `a8aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6w2d9V1y4J5j5h3y4C8M7#2)9J5c8V1y4D9N6i4c8U0K9l9`.` 将游戏解密导出为 IPA

3. 将导出的 IPA 传到 PC 上

4. 用 7zip 或者别的 unzip 工具解压 IPA

## 简单的残片数值修改

我们先简单地试着修改残片值，看看对程序的修改流程是否能正常运作。

先用 IDA x64 加载主程序 `Arc-mobile` 等待自动分析完成。

`Shift-F7` 打开 Segments 界面，把名字中带 `const`, `string`, `data` 的区段 Segment permissions 全部改成只读的，这样 IDA 反编译时才好识别出字符串。

![](upload/attach/202208/898032\_AFNZPHP6HQYQURT.jpg)

通过一些简单的搜索可以了解到，残片值信息存在本地的 `AppData Container/Library/Preferences/moe.low.arc.plist` 文件的一个 `fr_v` 键值中，并且有一个相对的 `fr_k` 键值校验 `fr_v` 值的可信性，大概是某种哈希算法。但是这些都不重要，但存地修改本地缓存的值只能一次性生效，游戏中的数值修改仍然会复写掉你改的值。我们可以试试更加有效的更改残片值的方法。

在 IDA 的 Python 命令行输入 `'fr_v'.encode().hex(' ')` 得到这个字符串的 hex 值，然后在 IDA 的 Hex View 中 `Alt-B` 打开 Binary search 界面，搜索这串 hex 值。（有可能是字符串长度太短 IDA 没有把它认作字符串，故而使用二进制搜索而不是字符串搜索）

![](upload/attach/202208/898032\_MFX2D8NM93BUR8Y.jpg)

可以看到在 `__cstring` 区段找到了这个字符串。

![](upload/attach/202208/898032\_JWNMMCSHPPKZB55.jpg)

通过 XREF 我们可以定位到这个函数，简单看一下就能发现它读取 `fr_v` 和 `fr_k` 的键值，做了一些看似验证的操作，最后返回了 `fr_v` 的键值或者 `0`，大概验证失败的话就返回 `0` 了。

![](upload/attach/202208/898032\_G8R34HPP56DXC9S.jpg)

在确认一下这个函数是如何被调用的：

![](upload/attach/202208/898032\_EMHCYMXRDHV6R4J.jpg)

可以看到返回值直接被赋值到一个结构体上了，没有进一步的检验。那么想要修改残片值就太简单了，甚至完全不需要去理会 `fr_k` 是怎么校验的。

如果有安装 `keypatch` 之类的插件可以直接使用插件修改程序，不过为了照顾更多人，这里我们从头生成机器码。

打开 `Compiler Explorer`，随便编译一个返回 `999999999` 数值的函数，记得选择 `armv8-a` 的编译目标：

![](upload/attach/202208/898032\_DG2CFWXB8DFBH25.jpg)

每一行汇编上面一小行 hex 字符是对应的机器码，不过是用 Little-endian 的 DWORD 表示的，如果写入二进制文件需要 swap bytes。

回到 IDA 的汇编界面，把光标移到目标函数的开头一行，然后切换到 Hex View，按 `F2` 进入编辑模式，然后直接覆盖 Hex 值机器码，完后 مستقيم 按一次 `F2` 确认编辑：

![](upload/attach/202208/898032\_FT5SA9UB4HFQV3J.jpg)

回到汇编界面我们可以看到该函数被改为返回 `999999999` 了。 可以按 `P` 在此处重新建立并分析函数，就能 `F5` 看反编译代码了。

![](upload/attach/202208/898032\_9YFH69NE7HNQS2C.jpg)

接下来我们要把改动的部分写入到 `Arc-mobile` 文件中，这个操作位于 Edit - Patch program - Apply patches to input file...，注意你必须位于汇编界面或者 Hex 界面才能使用这个操作。

![](upload/attach/202208/898032\_M822ECBPCVKHY6H.jpg)

接下来我们需要 Sideload 我们修改后的 App，这一步的选择有很多，而且不一定需要越狱，比如 `AltStore`, `AltServerPatcher`, `Sideloadly`, `Cydia Impactor`, 甚至手动。因此就不详细展开了。

不过如果一切顺利，你安装好的游戏每次启动都会恢复到 `999999999` 个残片。做到这一点说明你已经成功掌握了修改 IPA 并安装测试的整个流程了。接下来我们来做些更有趣的改动。

## 更改 songlist 文件

网上有很多关于该游戏资源文件的格式介绍，我就不详细说明了，其中主要的曲目文件都在 `songs` 目录下，其中 `songlist` 文件列举了所有的曲目，是游戏主程序加载曲目的索引文件。为了防止修改 `songlist` 文件，游戏对其进行了哈希校验，其哈希值是写死在了 `Arc-mobile` 二进制文件中的。我们这就来看看。

首先 `Shift-F12` 打开字符串界面，`Ctrl-F` 搜索 `songlist`，找到 `songs/songlist` 字符串，然后通过 XREF 找到这样一个函数：

![](upload/attach/202208/898032\_ESRDZ8ZZ298C69B.jpg)

更值得注意的是这个函数后半部分有这样一段哈希值的对比代码：

![](upload/attach/202208/898032\_UPMQCVV7KRFQ4UK.jpg)

毫无疑问这个函数做的就是加载 `songs` 目录下的索引文件并进行校验，且一共有三个文件需要被校验：`songlist`, `packlist`, 以及 `unlocks`。我们现在就来让修改程序直接绕过校验。先来看一下这个哈希值字符串对比的汇编代码：

![](upload/attach/202208/898032\_4WSYUA6WJ967M48.jpg)

看到在调用完 `std::string::compare` 之后马上使用 `CBNZ` 即如果返回值 `W0 != 0`，也就是说字符串匹配不相等，则跳转到后面的地址。反过来说，如果要绕过这个校验，我们需要让这个代码执行字符串匹配时，也就是 `W0 == 0` 时的行为，也就是不进行跳转。那么一个很简单的 MOD 就是把这些 `CBNZ` 全部替换成 `NOP` 空语句。根本不需要去研究那些哈希值是怎么计算出来的。

再次打开 `Compiler Explorer`，这回把 `NOP` 翻译成机器码，即 `1F 20 03 D5`，然后替换掉 `CBNZ` 语句的机器码：

![](upload/attach/202208/898032\_4N285YKFWJ3MB6B.jpg)

然后我们可以看到反编译界面中的 `std::string::compare` 全都成了无作用的代码了。

![](upload/attach/202208/898032\_VUCHYVW5USXW9MJ.jpg)

现在我们就可以对 `songs` 目录下的文件随意更改了。比如把 `songlist` 中一首需要下载的歌曲的 `remote_dl` 字段删掉，再把下载好的 `ogg`、`aff` 等文件放到 song id 的文件夹中，即可把歌曲转变为本地歌曲，无需下载。 又比如修改 `unlocks` 文件可以改变歌曲的解锁方法，甚至让其变成无需解锁的歌曲。这方面的改动就交给读者自由探索了。

## HTTPS API 抓包

接下来我们做点稍有难度，抓取 HTTPS API 数据包。首先我们需要给 iOS 设备配置中间人代理，并且能替换 HTTPS 证书解密加密流量内容。这需要一个中间人代理软件，最好还能记录数据包。这一步有很多选择，比如 `Burp Suite`，`mitmproxy`，`Charles Proxy` 等等，我这里使用 Burp Suite 来演示。

首先需要配置 Burp Suite 的代理服务器，以及 iOS 上的网络代理设置，请参考这里的指示。 然后需要把 Burp Suite 的 HTTPS 代理证书加入到 iOS 的信任证书中，请参考这里的指示操作。做完这些后你应该能在 Burp Suite 的数据包记录界面看到一些抓取到的数据包了。

但是，当你打开游戏，进行一番操作后，在 Burp Suite 中却没有看到任何 API 相关的数据包。按经验来说，这是遇上 Cert Pinning 了，需要进行 Unpin。

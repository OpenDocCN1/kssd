
# 某交友 App 欣赏限制逆向分析

目标是看一下欣赏按钮的限制怎么突破一下，不然每天仅能点几个欣赏。

首先将 `apk` 拖入到 `jadx` 中，未发现有壳（能找到具体逻辑代码），但发现大量 `0oO` 混淆的混淆代码。

![](upload/attach/202210/962594_BQ8P7DWEF9KB8KX.png)

根据字符串搜索（今日次数已用完、欣赏、`follow`、`like` 等）找到了一个核心的代码，之前……

![](upload/attach/202210/962594_MCAC5D4KU3QTGJB.png)

但用 `Objection Hook` 一下这个类，再点欣赏按钮，发现这个类并没有被触发。也检查了 `jadx` 是否反混淆了类名，还是不行。

之后用了某大佬的 `EncryptStack` 插件打印了一下加密库调用的信息，从打印的堆栈信息中搜索关键词，找到了一些可能是欣赏按钮被点击的时候会调用的方法。

![](upload/attach/202210/962594_4CKRDM3FY9JQK2R.png)

![](upload/attach/202210/962594_FGTHFBWERGJQCBQ.png)

在 `jadx` 中搜索这个方法。

![](upload/attach/202210/962594_GT2UHHG6B7KTPQW.png)

发现反编译之后没有内容（我也不太清楚为啥）。感觉像是很重要的代码。

用 `Objection hook` 了一下这个方法之后，发现点欣赏按钮会被调用。再用 `frida` 脚本打印了一下它的参数和返回值。

![](upload/attach/202210/962594_UK8QVT3C4ZRZN28.png)

结果为：（第三行为点击欣赏的结果，其他为跳过按钮的结果）

![](upload/attach/202210/962594_E8D73CGA3MF8B57.png)

参数有 `eventv3`，感觉像是发送了某个事件。然后搜索了一下这个 `match_admire` 字符串。

![](upload/attach/202210/962594_JT3ZSZD8J5P35BJ.png)

发现只有两处调用了这个方法。第一个就是上面 `hook` 过的那个没有被调用、无效的类，所以点开第二个。

![](upload/attach/202210/962594_UDDMGGYUMCFC56P.png)

然后用 `objection hook` 了一下看看是否被调用，发现点欣赏按钮的时候调用了它和它的一些重载方法。看了一眼它的重载方法，在重载方法中看到一些重要逻辑代码。

![](upload/attach/202210/962594_4KPE8NZBQPSK27S.png)

![](upload/attach/202210/962594_UP6CAWU9N5KXEVD.png)

感觉这个应该就是欣赏限制的逻辑。

点开第一个条件。

![](upload/attach/202210/962594_GXZ2FQC77U8BP95.png)

看到里面都是一些 `vip` 判断的信息、`vip` 剩余时间之类的。应该是从接口返回的这个用户是否是 `vip` 之类的，提供给程序判断用户身份。

那岂不是可以修改参数，伪装成 `vip`？这样应该就可以无限点击欣赏了。

![](upload/attach/202210/962594_KQHBRX4RYDS52K8.png)

用 `frida hook` 这个方法，看参数和返回值。

![](upload/attach/202210/962594_EXFZ9H7V7QWJM8S.png)

所以修改了一下参数、`vip` 剩余天数之类的。

![](upload/attach/202210/962594_XG5PGKQ28HPWNMG.png)

因为类型转化暂时不太熟悉，所以看一下这个参数实体类 `JsonData` 中有没有构造方法之类的，重新 `new` 一个 `JsonData`。

找到了其中有一个 `create` 方法，参数是 `Object`，以及其中调用的 `init` 方法会判断参数类型。

![](upload/attach/202210/962594_YEZYM63SFN6DA6W.png)

![](upload/attach/202210/962594_VT4PVEKDFC7SSM2.png)

之前 `hook` 看到的类型是 `JSONObject`，原生的。所以构建了一个 `JSONObject`，再主动调用 `JsonData` 的 `create` 方法生成参数，再回传。

![](upload/attach/202210/962594_PN4FFEKKPBQSRE6.png)

果然替换过之后，绝大部分 `vip` 的功能都可以用了，个别的还有问题，别人看到你还是个普通用户……

第一次分析 App，完成后有点激动，哈哈哈，记录一下过程……

[招生]科锐逆向工程师培训（2026 年 11 月 13 日实地，远程教学同时开班，第 57 期）


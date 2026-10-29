
可以发现注册账号的协议十分简单，前三个我们可以随便拟造，后两个是固定的。

## 6.2 登录账号

同理分析登录账号的请求，可以看到只是比注册请求多了一个 `login` 参数。

![](upload/attach/202004/802108\_V3EVV4H37VJ7QYU.png)

## 6.3 更新积分

在进行积分消费的时候，观察到现有积分扣除后，会向服务器发送一次积分更新请求，抓包数据如下，发现有个 `time` 和 `sign`，猜测根据 `time` 生成 `sign`，然后会在服务器进行验证。

![](upload/attach/202004/802108\_7J6NP46P6PNQX5F.png)

找到构造请求的代码处。

![](upload/attach/202004/802108\_5VWT5HNFTNTU5Q2.png)

可以看到 `sign` 是根据 `time` 生成的，我们通过交叉索引找到生成 `sign` 的算法，可以发现只是简单的 `base64` 加密。

![](upload/attach/202004/802108\_744W47JNEHENBJK.png)

## 6.4 注册 VIP

上面我们提到怎么按破解内购的思路来 `hook` 改变支付逻辑，我们尝试一次并抓包。

![](upload/attach/202004/802108\_ADQMV6XHYP6SFYG.png)

可以看到第二个请求便是注册 VIP，我们找到相应的代码处。

![](upload/attach/202004/802108\_2HUWFEY69A9HUA4.png)

请求参数分别是 `name`、`endviptime`、`startviptime`、`memi1`、`verson`、`viptime`、`key`、`rightkey`，十分简单。

## 6.5 写一份协议

然后很自然的，我们可以轻松地自己写一份简单地协议，尝试过程中发现 `memi1` 也就是 `android_id` 非常容易被封，需要不断更换。。以及尝试了很久写注册 VIP 的请求都没有成功，可能在服务器还有别的验证。

```python
import base64
import time
import requests

requests.packages.urllib3.disable_warnings()

class tv:
    def __init__(self):
        self.root = 'http://39.108.64.125/WebRoot/superMaster/Server'
        self.memi1 = "9774d56d682e549c"
        self.rightkey = "376035775"
        self.key = "308202d5308201bda00302010202041669d9bf300d06092a864886f70d01010b0500301b310b3009060355040613023836310c300a06035504031303776569301e170d3136303731383038313935395a170d3431303731323038313935395a301b310b3009060355040613023836310c300a0603550403130377656930820122300d06092a864886f70d01010105000382010f003082010a028201010095f85892400aae03ca4ed9dcd838d162290ae8dd51939aac6ecfde8282f207c4cd9e507929a279e0a36f1e4847330cb53908c92915b2c6a93d7064be452d073a472093f7ca14f4ab68f827582fe0988e9e4bc8a6ea3b56001cbbbb760f9eec571b0bbc97392e65aaf08c686f0e2ba353896d48a37c36716239977bd0e4dd878025cab497d8164537aec9f6599eefb98577dce972a1b794e211226520e23497beec3fd8548bb5b4d263120d40115cca28116bac32378df5033f536a0d7367fef78c587fefed28c5c9b35ba684ed6e46d9369c40950cf7ad7236d10b7a51dfd2a8f218db72323bbd19f46947410b1191f263012ad4ba8f749223e37591254ee7f50203010001a321301f301d0603551d0e041604143d43284bd5e4b0d322c9962a5b70aad4dcbc3634300d06092a864886f70d01010b050003820101000f04c51ff763311aa011777ba2842b441b15c316373d1e1ed4116cf86e29d55c6ed3fa4c475251b1fb4fac57195dbca0166ebe565d9834552a3758b97c4528bab1f7ab82bb3a9faa932f5bc10943f3daf52e0fe5889ffb58a6be67ea1c9a2fb37dc8aa6f3af476039a467336991a4e52dccd520195cd473eb5b984e702ed9ff638a14c3abb575a7a80ae4062084d1138a06a20e173be9df32df631311b07352898706198ddebaaa011f0da8e5f288f7cfb77505bc943f6476d6cc1feef56b68137aad91f23c4bb772169539d05653a6f0d75f7192164e822b934322f3a975df677903b1667f5dc1e9ddb185da3281d31bfb8f67a84bd23bbcb398f8bb637dd72'

    def post(self, data=None):
        if data is None:
            data = {}
        return requests.post(url=self.root, data=data)

    def register(self, name, password):
        ret = self.post({
            'name': name,
            'pass': password,
            'memi1': self.memi1,
            'key': self.key,
            'rightkey': self.rightkey
        })
        print("Register response data: ")
        print(ret.content.decode('utf-8'))

    def login(self, name, password):
        ret = self.post({
            'name': name,
            'pass': password,
            'memi1': self.memi1,
            'key': self.key,
            'rightkey': self.rightkey,
            'login': 'login'
        })
        print("Login response data: ")
        print(ret.content.decode('utf-8'))

    def updateSocre(self, name, password, jifen):
        t = int(round(time.time() * 1000))
        sign = base64.b64encode(str(5 * t).encode('utf-8')).decode('utf-8')
        ret = self.post({
            'name': name,
            'pass': password,
            'jifen': jifen,
            'time': t,
            'sign': sign
        })
        print("UpdataScore response data: ")
        print(ret.content.decode('utf-8'))


if __name__ == "__main__":
    tv = tv()
    # 注册账号
    print(tv.register("mee4", "mee4"))
    # 登录账号
    print(tv.login("mee4", "mee4"))
    # 更新积分
    print(tv.updateSocre("mee4", "mee4", "1000"))
```

# 7. 结语

整个 `frida` 使用过程就是这样子，花时间整理不太容易，希望各位可以学到有用的东西能顺便点个赞就更好了（逃

***

更：评论区有问到 APP 要防止内购破解，可以做什么操作，简单写了些自己的看法，如果哪里有什么误解和偏见还请各位大佬指点下..

个人认为像这个 APP 虽然加了壳，也有根据 `time` 生成 `sign` 的算法然后把这两个值传到服务器去验证，也有获取 APP 的签名信息传到服务器防二次打包，还使用了防抓包的编程写法，但是都十分简单，一一解决起来十分容易，如果想防止内购破解，这些点都是必要的点，是都需要加强对抗的，根据木桶效应，一只水桶能装多少水取决于它最短的那块木板，APP 对抗同样也是。

首先是加的壳比较简单，`dex` 文件全部加载到内存中了轻松地可以 `dump` 出来，这样 APP 布置的第一道最重要的防线就没多少防护意义了，下面更是一败而溃了。

然后根据 `time` 生成 `sign` 的这么重要的算法不可放在 Java 层中去调用，不然反编译出来十分容易被分析出来利用构造协议，这个 APP 里面不仅放在 Java 层算法还只是简单的 `base64` 加密。逆过的一些防护优秀的 APP 是不加壳的，仅仅把根据 `time` 生成 `sign` 的算法放在 `native` 层进行对抗，分析起来难度十分大。

获取签名信息防二次打包这种重要的东西同样也是和生成 `sign` 算法一样不应该放在 Java 层的，应放在 `native` 层去对抗，以及获取签名的方式应该多样化，最好避免最常用的那种。

还有防抓包，感觉最好的解决方式还是对数据进行加密传输，加密算法放在 `native` 层。

还有反调试、防模拟器等等各种其他风险控制都要随着 APP 的防护需求去布置。

其实这样分析一下，很多时候不用对整个 APP 大动干戈地加壳，只需要对几个关键的点进行优雅地高强度对抗就能满足防护需求了。

## 目录

- 1. 前言
- 2. App 简单分析
- 3. 脱壳
  - 3.1 关于 `frida-unpack`
  - 3.2 `frida-unpack` 使用报错及解决方案
    - 3.2.1 使用
    - 3.2.2 `KeyError` 报错
    - 3.2.3 `readU8` 报错
    - 3.2.4 修改代码
  - 3.3 顺利脱壳
- 4. Hook 进行破解
  - 4.1 积分
  - 4.2 VIP
  - 4.3 `android_id`
- 5. 模拟抓包
  - 5.1 Charles 抓不到包的原因
  - 5.2 模拟抓包
  - 5.3 使用 Drony 配合 Charles 抓包
  - 5.4 `firda Hook` 完整代码
- 6. 协议分析
  - 6.1 注册账号
  - 6.2 登录账号
  - 6.3 更新积分
  - 6.4 注册 VIP
  - 6.5 写一份协议
- 7. 结语

# 1. 前言

`frida` 是一个十分强大的工具，已经学习它有一段时间了，但也只是零零碎碎的练习与使用。最近在对一个 APP 进行分析的过程中，使用 `frida` 完成了脱壳、`hook` 破解、模拟抓包、协议分析的操作，可以说是一条龙服务了，感觉十分有意义，学到了很多，对 `frida` 的理解和掌握程度也提高了不少，记录下来这次实战分享给各位正在学习 `frida` 的看雪用户。（在看雪上论坛水了这么久，也该反馈些东西了，逃

`frida` 入门这里就不多说了，论坛已经有很多优秀的入门帖子了，我也是看着这些帖子一点一点学习的。不过提一下我在安装时候踩的坑，当时折腾很久也安装不上心态被搞得有点爆炸。。

开始 Python 3.7 直接 `pip install frida` 和 `pip install frida-tools` 一直卡在 `Running setup.py install for frida ... –` 了，最后的解决好的办法是到 [ba6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3P5i4m8A6i4K6u0W2L8%4u0Y4i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5c8X3k6J5K9h3c8S2i4K6u0r3i4K6t1K6k6X3W2D9k6i4x3`](https://bbs.kanxue.com/elink@b54K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3P5i4m8A6i4K6u0W2L8%4u0Y4i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5c8X3k6J5K9h3c8S2i4K6u0r3i4K6t1K6k6X3W2D9k6i4x3`) 下载 `frida-xx.x.xx-py3.7-win-amd64.egg`，并把它放到安装的 Python 目录的 `\Python37\Lib\site-packages` 中。

然后找到对应的 `frida-tools` 版本，执行 `pip3 install frida-tools` 即可安装。如果不对应，执行命令可能会把 `frida-xx.x.xx-py3.7-win-amd64.egg` 删除，又卡在 `Running setup.py install for frida ... –` 地方，可以根据发布日期来判断相应的 `frida-tools` 版本。

然后 Python 3.7 安装 `easy_install`，执行 `easy_install frida-xx.x.xx-py3.7-win-amd64.egg`，即可在 Python 中 `import frida` 了。

# 2. App 简单分析

这是一个视频播放的 APP，里面有着各种卫视和 CCTV 的在线播放，其他栏中是一些新闻栏目和电影。

![](https://bbs.kanxue.com/upload/tmp/802108_RMSVN9PTTDBXMEY.png)

这些都是免费播放的，但是我们注意到了右上角的“积分：0”字样，说明情况不简单。我们点开个人栏，查看发现“在线吧”、“在线吧 2”里面需要积分消费才能进去。

![](https://bbs.kanxue.com/upload/attach/202004/802108_C66CZFND7U6QWC7.png)

而成人台里面又有一系列的栏目，每个进去都需要积分消费，点进去后会发现每个栏目里面都有一堆不可描述、胡里花哨、不符合核心价值观的影片或者直播分类，说明这是一个邪恶的 APP。

![](https://bbs.kanxue.com/upload/attach/202004/802108_722PZHBVHYJDMQT.png)

在获取积分里面，可以购买 VIP 和积分。

![](https://bbs.kanxue.com/upload/attach/202004/802108_PPJ9KUNCBEW942E.png)

然后经过简单测试发现加了 360 壳，并且使用 Charles 抓不到包。我们先使用 `firda` 脱壳拿到 DEX 文件，抓包问题后面再解决。

# 3. 脱壳

既然是 `frida` 的一条龙服务，我们尝试用 `frida` 来进行脱壳，这里我们直接使用 `frida-unpack`（[6fbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1M7%4c8E0j5i4c8Z5i4K6u0r3k6Y4u0A6k6r3q4Q4x3X3c8#2L8Y4m8S2j5$3D9`](https://bbs.kanxue.com/elink@e4cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1M7%4c8E0j5i4c8Z5i4K6u0r3k6Y4u0A6k6r3q4Q4x3X3c8#2L8Y4m8S2j5$3D9`)）。

## 3.1 关于 `frida-unpack`

`firda-unpack` 原理是利用 `frida` hook `libart.so` 中的 `OpenMemory` 方法，拿到内存中 DEX 的地址，计算出 DEX 文件的大小，从内存中将 DEX 导出。我们可以查看项目中的 `OpenMemory.js` 文件中的代码，更清晰直观地了解。

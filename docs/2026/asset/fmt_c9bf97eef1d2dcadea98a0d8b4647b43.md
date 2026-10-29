
对于这个调试，一个就是通过代理替换，定向到本地文件或自己的本地服务器，还一个就是可以自己保存网页，架设一个自己的服务器，这样可以随时改

代码来调试了。

本地调试的时候的，这个`js`路径要保持一致，有代码会检测这个路径格式：

![](https://bbs.kanxue.com/upload/attach/202205/44250_38UUXPRRHJUGQW4.jpg)

`TI`的这个`js`逻辑要比`IHG`的复杂：

1、 `post` 的 `sensor_data` 数据的，`IT` 有加密，`IHG` 没有。

2、 `IHG` 只用到了 `cookie` 中的 `_abck`，`TI` 多了个 `bm_sz`。

我没看过 `Akamai` 的低版本，从网上看有提到 `1.75` 版本，然后开 `IGH` 提交的 `sensor_data` 数据头部：

`key_ver: '7a74G7m23Vrp0o5c9231281.75'`

那这么看后面的数字就是版本号了，但是 `TI` 的是不同的，解密后，头部是这样的：

`7a74G7m23Vrp0o5c924489Ac8AWA+tulfQVzVpsCpdMA==`

前段时间版本升级了下的，头部变成这样：

`7a74G7m23Vrp0o5c924493oWV4VVGxQUElcrjPIc9GQQ==`

看起来前面一段是固定的，中间数字是时间相关的，后面一串是版本相关的，这个字符串是 `js` 里面的一个变量值。

看起来是个 `base64` 串，解了下也没看出个所以然，不管了，这个也不影响。

对于本地调试，修改过的代码要跳过校验，`js` 里面有逻辑会检查函数是否修改过，

下面这里是校验几个函数的 `hash`，对修改过的函数，校验不过就会走错误分支:

![](https://bbs.kanxue.com/upload/attach/202205/44250_AE489WYWN5C2CZU.jpg)

能调试后，单纯的逻辑逆向，相比二进制就简单些了，后面要做的就是用 `node` 来构造环境，生成需要发送的 `sensor_data` 数据，主要是一些浏览器相关的

环境构造，包括 `webgl` 信息、语音组件信息等，当然这个也要花点时间，有些需要跟浏览器的数据比较，如果发送的数据不对，就会返回类似这种错误：

`'ti_bm=Unknown%20Bot%20(8D4B186BE16BB4B3B59A0C24C27BAE69)%3aslow%3a%3aJavaScript%20Fingerprint%20Not%20Received%20(BETA)%3a%3a; path=/; domain=.ti.com'`

这个搞完后测试，发现可以加入购物车了：

![](https://bbs.kanxue.com/upload/attach/202205/44250_2BNYQ5AJQ6CJD32.jpg)

这个 `sensor_data post` 要模拟完全，不然可能出现协议成功率问题，持续添加可以后，那就是多账户问题了，这个可以通过差异化不同账号的请求环境

来处理，比如模拟不同的浏览器参数。

除了这个 `js` 逻辑外，还有个上面提到的头部 `js`，`pixel` 前缀的，这个会用来提交本地的环境参数的，

整个网站中，还有些反爬虫的处理，有的页面是直接返回的 `js`:

![](https://bbs.kanxue.com/upload/attach/202205/44250_GCKKSSTG9K9Z2XU.jpg)

这种就要解析出脚本来执行了。

现在回过头来看，要是替换掉 `miniblink` 里面的 `curl` 模块，那就是一个带源码的浏览器调试环境了，有个完整的浏览器环境，对于验证还是比较方便的。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

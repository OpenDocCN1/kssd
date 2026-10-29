
# [原创]某APP收费音频无会员绕过下载过程分析

## 0x00 背景介绍

在工作后的休闲时间我比较喜欢打开网络电台听一些有声书，大牛实事点评；不知道从什么时候开始网络上突然流行起了付费音频，很多付费音频都是由名家亲自参与制作，质量非常高 很受大众的喜欢；其中以某电台的付费内容最受欢迎，我也购买了好些套音频来跟着名家的脚步学习；名家制作的付费音频在选择的时候不容二说，直接购买就是；但是也不乏其中有很多的标题党 哗众取宠，能不能在不付费的情况下先进行一次试听呢

## 0x01 分析

为了方便分析，我特意选择了直接在浏览器中打开电台wap版；随便选择一个付费才能收听的音频，使用Chrome浏览器，调试模式，手机模式如下所示

![](https://bbs.kanxue.com/upload/attach/201703/352285_bvns3uzmwsxddyw.png)

音频播放URL：`513K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2Q4x3X3g2^5P5s2S2Q4x3X3g2U0L8$3#2Q4x3V1j5$3z5e0p5@1z5e0x3$3x3q4)9J5c8Y4y4G2N6h3&6V1i4K6u0r3x3K6t1I4y4K6x3@1x3o6V1`.

其中直接访问 `8d6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2Q4x3X3g2^5P5s2S2Q4x3X3g2U0L8$3#2Q4x3V1j5$3z5e0p5@1z5e0x3$3x3l9`.`. 可以显示当前主播的所有音频，也就是说 `69149360` 为主播ID， 而后面的 `32173409` 应该为当前的音频ID。

点击播放按钮通过chrome 调试 Network功能标签中，可以看到客户端会向服务端请求音频的真实地址用来播放； 通过分析发现以下两个请求比较关键。

### 第一个关键请求

```
GET 81fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2Q4x3X3g2j5h3q4S2j5i4K6u0W2j5$3!0E0i4K6u0r3L8h3!0T1K9h3I4W2i4K6u0r3N6s2u0S2j5$3E0Q4x3V1k6H3j5i4W2Q4x3V1j5K6x3U0p5%4x3K6b7H3z5g2)9K6c8X3c8W2N6X3W2U0k6g2)9K6c8s2m8U0i4K6t1$3j5h3#2H3i4K6y4n7N6h3W2V1i4K6y4p5i4K6t1$3j5h3#2H3i4K6y4n7N6r3!0C8k6h3&6Q4x3@1c8Q4x3U0k6S2L8i4m8Q4x3@1u0A6M7@1u0S2j5$3E0W2L8X3c8Q4x3@1c8X3j5h3I4K6k6g2)9J5y4X3q4E0M7q4)9K6b7W2)9#2k6W2)9K6c8o6p5@1z5e0l9K6y4U0b7#2z5e0x3%4y4U0V1`.
```

服务端返回：

```json
{
    "ret": 0,
    "msg": null,
    "errorCode": null,
    "trackId": 32173409,
    "uid": 69149360,
    "albumId": 6294413,
    "title": "演说之禅 | 耿人健 0309",
    "domain": "d85K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4#2k6r3W2G2i4K6u0W2M7r3q4&6i4K6u0W2P5s2S2Q4x3X3g2U0L8$3#2Q4x3U0k6I4N6h3!0@1i4K6y4n7i4K6u0o6",
    "duration": 2752,
    "totalLength": 22278931,
    "sampleDuration": 180,
    "sampleLength": 1702877,
    "isAuthorized": false,
    "apiVersion": "1.0.0",
    "buyKey": "fe4f133ccbf4b22dfa2a1e704ccbbda8",
    "seed": 3669,
    "fileId": "38*14*54*34*10*54*56*59*50*65*47*62*53*65*65*59*26*14*3*48*38*50*59*61*47*50*59*47*46*59*20*52*59*56*23*26*15*61*16*55*60*15*12*43*8*43*30*55*30*60*11*39*29*52*53*40*15*7*48*3*53*53*46*2*12*46*43*",
    "ep": "ixdsaY59SiQC2v0Mb4wd414PUk0i1ibGSddPKQ7mX3e0nu+O2qjckr8Kga7ahPJmVbQjgHJRfvE0jPb8wQMSjrkPPC9VE6CqX9LAvCdcqUKio+NbmGgY"
}
```

### 第二个关键请求

```
2deK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4#2k6r3W2G2i4K6u0W2M7r3q4&6i4K6u0W2P5s2S2Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5c8U0q4Q4x3X3f1H3i4K6u0W2x3q4)9J5c8Y4m8J5k6i4k6A6k6i4N6Q4x3V1j5I4y4K6l9J5z5o6M7%4i4K6u0r3k6%4u0G2N6i4l9I4i4K6u0r3e0e0l9I4i4K6u0r3x3o6c8Q4x3V1j5$3c8g2)9J5c8Y4N6w2k6@1A6y4L8r3A6m8d9X3#2S2P5r3q4n7K9V1u0m8g2W2m8*7c8e0R3K6d9X3k6#2L8K6R3^5y4q4)9J5k6h3@1@1j5g2)9K6c8Y4y4A6k6$3&6Q4x3@1b7%4j5U0u0T1x3e0V1K6k6U0V1#2k6o6x3K6x3r3j5$3x3e0j5H3x3e0y4U0j5K6q4X3k6U0M7H3z5h3t1H3x3g2)9J5y4X3q4E0M7q4)9K6b7X3u0#2P5g2)9#2k6X3E0W2P5g2)9K6c8r3k6W2y4r3j5I4x3K6y4U0j5$3u0X3y4r3t1J5x3X3c8X3j5e0u0S2x3h3f1%4x3o6c8U0j5$3u0T1k6r3p5^5i4K6t1$3j5h3#2H3i4K6y4n7N6r3W2E0k6i4y4@1j5h3#2H3i4K6y4p5x3e0b7&6x3o6x3$3y4o6f1%4y4e0f1$3y4K6l9J5x3q4)9J5y4X3q4E0M7q4)9K6b7Y4c8G2K9$3g2F1i4K6y4p5y4U0x3&6x3g2)9J5y4X3q4E0M7q4)9K6b7X3c8#2M7X3q4@1K9h3!0F1i4K6y4p5x3U0M7#2x3R3`.
```

第二个请求直接就是m4a 音频地址了，其中服务器地址是 `audio.pay.xx.com`，可以轻易的看出应该是从第一条GET中返回的json中获取的；那这一串参数是怎么得来的呢？

```
/download/1.0.0/preview/1702877/group1/M01/04/6E/wKgJMljAJmaxaBjBAVPzE83Jfuo884.m4a?sign=7b2b193f95d330f616013cc1ff709b01&buy_key=fe4f133ccbf4b22dfa2a1e704ccbbda8&timestamp=1490364575567020&token=6391&duration=2752
```

## 0x02 调试

作为一个优秀且长相帅气的看雪潜水多年的网友，遇到问题想到的第一解决方案就是 调(tiao)试(xi); 对付这种web型的协议密钥就不用祭出倚天剑(OD)和屠龙刀(IDA)了，直接利用chrome自带的调试功能以及Fiddler进行劫持就够了。过程如下：

从以上两条请求可以分析得出，音频的最终请求参数不是由服务器返回，而是由本地计算得到的；通常在web开发中js往往就承担了计算以及动态控制责任。由于本人对js不是很熟悉，无法快速定位于是使用了一个本办法：删除触发播放按钮的事件来定位，删除一个点击下播放，查看是否能播放，然后再删除一个再点击播放 如此循环；

在不断删除点击下，最终定位到请求w4a的处理时间在`all.js`中。打开`all.js` 发现所有js源码都在同一行，这完全无法分析，而且影响下js断点调试；于是我将js 格式化保存到本地，利用Fiddler 的AutoResponder功能劫持到本地来设置。由于站点使用https 来传输js，Fiddler无法劫持https内容，我们索性将整个请求页面一并劫持并修改源码`all.js` 去请求非https；Fiddler设置如下

![](https://bbs.kanxue.com/upload/attach/201703/352285_7ud56sc1ayqpj84.png)

`all.js`格式化之后变得特别清晰，于是乎我们先稍微看一遍`all.js` 看看哪里可能是解密的关键地方。干逆向这么久，觉得逆向的过程就是一个猜想和推到猜想的过程；以我C语言功力来理解js 在有可以的地方下断点(关于Chrome JS 调试请看这里： [b0bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3A6T1y4e0q4Q4x3X3g2F1k6i4c8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3y4e0R3#2y4K6m8Q4x3X3g2Z5N6r3@1`.](https://bbs.kanxue.com/elink@aeeK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3A6T1y4e0q4Q4x3X3g2F1k6i4c8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3y4e0R3#2y4K6m8Q4x3X3g2Z5N6r3@1`.))

![](https://bbs.kanxue.com/upload/attach/201703/352285_c7psggnpycrdhfb.png)

![](https://bbs.kanxue.com/upload/attach/201703/352285_qpybtw6g0hm379f.png)

点击播放，最终发现在 `success: function(t) {` 这个地方停下来

![](https://bbs.kanxue.com/upload/attach/201703/352285_himn572wsxqie24.png)

F11一步步分析得出结论如下

解密播放绝对路径：

KEY通过字典`o(s, r)` 运算由 `dg3utf1k6yxdwi09`得到 `xkt3a41psizxrh9l`

```js
t = "dg3utf1k6yxdwi09",e = [19, 1, 4, 7, 30, 14, 28, 8, 24, 17, 6, 35, 34, 16, 9, 10, 13, 22, 32, 29, 31, 21, 18, 3, 2, 23, 25, 27, 11, 20, 5, 15, 12, 0, 33, 26]function o(t, e) {        for (var i = [], o = 0; o < t.length; o++) {            for (var n = 0,            n = "a" <= t[o] && "z" >= t[o] ? t[o].charCodeAt(0) - 97 : t[o] - 0 + 26, a = 0; 36 > a; a++) if (e[a] == n) {                n = a;                break            }            i[o] = 25 < n ? n - 26 : String.fromCharCode(n + 97)        }        return i.join("")    }
```

再将KEY和第一条GET得到的json内容中的`ep`一起传入 `i(p, e(t.ep))` 中，经过取字典位移等运算得出密文，

```js
function e(t) {
        if (!t) return "";
        var e, i, o, n, a, t = t.toString(),
        s = [ - 1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 62, -1, -1, -1, 63, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, -1, -1, -1, -1, -1, -1, -1, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, -1, -1, -1, -1, -1, -1, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, -1, -1, -1, -1, -1];
        for (n = t.length, o = 0, a = ""; o < n;) {
            do e = s[255 & t.charCodeAt(o++)];
            while (o < n && -1 == e);
            if ( - 1 == e) break;
            do i = s[255 & t.charCodeAt(o++)];
            while (o < n && -1 == i);
            if ( - 1 == i) break;
            a += String.fromCharCode(e << 2 | (48 & i) >> 4);
            do {
                if (e = 255 & t.charCodeAt(o++), 61 == e) return a;
                e = s[e]
            } while ( o < n && - 1 == e );
            if ( - 1 == e) break;
            a += String.fromCharCode((15 & i) << 4 | (60 & e) >> 2);
            do {
                if (i = 255 & t.charCodeAt(o++), 61 == i) return a;
                i = s[i]
            } while ( o < n && - 1 == i );
            if ( - 1 == i) break;
            a += String.fromCharCode((3 & e) << 6 | i)
        }
        return a
    }

function i(t, e) {
        for (var i, o = [], n = 0, a = "", s = 0; 256 > s; s++) o[s] = s;
        for (s = 0; 256 > s; s++) n = (n + o[s] + t.charCodeAt(s % t.length)) % 256,
        i = o[s],
        o[s] = o[n],
        o[n] = i;
        for (var r = n = s = 0; r < e.length; r++) s = (s + 1) % 256,
        n = (n + o[s]) % 256,
        i = o[s],
        o[s] = o[n],
        o[n] = i,
        a += String.fromCharCode(e.charCodeAt(r) ^ o[(o[s] + o[n]) % 256]);
        return a
    }
```

使用json返回的`_randomSeed`通过算法得到字符串A "oRIBQLWKzamwPSuh\C3:1cdMDTYvJVeH\_q97fjX58p4nFsgxyr20t.NObi6-GEAlUk/Z"

```js
cg_hun: function() {
            this._cgStr = "";
            var t = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ/\\:._-1234567890",
            e = t.length,
            i = 0;
            for (i = 0; i < e; i++) {
                var o = this.ran(),
                n = o * t.length,
                a = parseInt(n);
                this._cgStr += t.charAt(a),
                t = t.split(t.charAt(a)).join("")
            }
        },
```

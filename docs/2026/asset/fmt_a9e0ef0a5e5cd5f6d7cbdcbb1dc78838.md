
| 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 | 17 | 18 | 19 | 20 | 21 | 22 | 23 | 24 | 25 | 26 | 27 | 28 | 29 | 30 | 31 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `{` | `"data": {` | `"bizFingerprintType": 3,` | `"tabInfo": [],` | `"classifyStyle": 1,` | `"categories": [` | `{` | `"internalId": "3127",` | `"secondList": [` | `{` | `"internalId": "3513",` | `"categoryImage": "a42K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3q4Y4k6g2)9J5k6h3#2A6M7%4y4X3M7X3g2K6K9q4)9J5k6h3y4F1i4K6u0r3y4e0j5%4x3U0R3@1j5U0g2U0x3K6N6X3y4r3x3#2j5e0R3I4y4h3k6X3k6e0x3$3k6X3c8S2x3h3t1@1y4o6g2Q4x3X3g2H3L8X3M7",` | `"icon": "热",` | `"name": "推荐",` | `"parentId": "3127"` | `},` | `{` | `"internalId": "3514",` | `"categoryImage": "4d7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3q4Y4k6g2)9J5k6h3#2A6M7%4y4X3M7X3g2K6K9q4)9J5k6h3y4F1i4K6u0r3x3K6u0W2k6o6l9$3j5$3x3K6x3o6x3&6y4r3t1H3z5o6V1#2k6e0M7%4x3h3u0T1x3$3p5%4z5h3f1#2z5h3g2Q4x3X3g2H3L8X3M7",` | `"icon": "惠",` | `"name": "会员特惠",` | `"parentId": "3127"` | `},` | `{` | `"internalId": "3129",` | `"categoryImage": "254K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3q4Y4k6g2)9J5k6h3#2A6M7%4y4X3M7X3g2K6K9q4)9J5k6h3y4F1i4K6u0r3x3e0V1%4x3h3x3J5k6r3t1H3z5e0x3^5y4o6l9K6z5o6R3$3y4r3k6V1k6r3x3^5j5U0t1@1z5e0M7I4y4o6q4Q4x3X3g2H3L8X3M7",` | `"icon": "新",` | `"name": "时令上新",` | `"parentId": "3127"` | `},` | `......` |

## 加密参数确定

通过 Postman 调试可知，`mfsig`不传或错传后不能正确请求数据，确认`mfsig`为核心加密签名。

![image-20210706153005140](https://bbs.kanxue.com/upload/attach/202107/928237_93PX528BQXY9CAK.jpg)

![image-20210706153036359](https://bbs.kanxue.com/upload/attach/202107/928237_WYDS2JSSWJ2PWK9.jpg)

## 初领妙道

### 小程序包反编译

#### 工具

##### wxUnpacker

小程序反编译工具 GitHub 地址：`fa5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6I4N6$3g2J5N6s2V1@1y4K6t1I4x3U0y4Q4x3V1k6%4P5r3q4H3M7q4g2F1M7r3q4U0K9$3g2J5`。

运行前提需要安装 Node 环境。

该工具运行需要一些 Node 依赖库，安装指引在 GitHub 的 `README.md` 文档中有。

#### 小程序包拉取

通过 RE 文件管理器 App 直捣微信小程序包路径：

`/data/data/com.tencent.mm/MicroMsg/${用户MD5}/appbrand/pkg/\_\*\_xxx.wxapkg`

利用 RE 文件管理器打成 ZIP 包，点击右上角按钮找到“发送”，通过 QQ、钉钉或者蓝牙等方式传送到个人电脑接收。

#### 小程序主子包判断

如今微信小程序单包体积不能超过 4M（小程序基础依赖包除外），如果项目内容过大，开发者会使用分包模式。

拿该电商来说，打开小程序一顿操作后，文件目录下发现四个包：

![image-20210706154412251](https://bbs.kanxue.com/upload/attach/202107/928237_789JX2TAR46Z88Z.jpg)

- `_2124598774_821.wxapkg`：3.3M，主包
- `_-588782754_76.wxapkg`：1.5M，子包
- `_152740959_13.wxapkg`：89k，子包
- `_1123949441_552.wxapkg`：14M，基础依赖包

#### 反编译执行

先反编译主包：

```bash
# 主包反编译
node wxWxapkg.js /Users/toretto/crack/wxapkg/missfresh_v3/_2124598774_821.wxapkg
```

再反编译子包，通过 `-s` 指定主包的路径，使子包反编译的内容复制到主包：

```bash
# 子包反编译
node wxWxapkg.js -s=/Users/toretto/crack/wxapkg/missfresh_v3/_2124598774_821 ../../wxapkg/missfresh_v3/_152740959_13.wxapkg

node wxWxapkg.js -s=/Users/toretto/crack/wxapkg/missfresh_v3/_2124598774_821 ../../wxapkg/missfresh_v3/_-588782754_76.wxapkg
```

![image-20210706154720473](https://bbs.kanxue.com/upload/attach/202107/928237_TSJPQ42N6AV88XZ.jpg)

看到 `File done` 即反编译成功。

![image-20210706154734258](https://bbs.kanxue.com/upload/attach/202107/928237_9GGZNX4U7N8Q6ZA.jpg)

用小程序开发工具打开。

![image-20210706154836938](https://bbs.kanxue.com/upload/attach/202107/928237_H3UXWAQ5XWWUQDW.jpg)

先在右上角详情中点击本地设置，勾选不校验合法域名、WebView、TLS 及 HTTPS 证书。

![image-20210706155122944](https://bbs.kanxue.com/upload/attach/202107/928237_MNME23V3V27C896.jpg)

完成上述操作后，一个简单代码分析环境就搭建好了。

## 渐入佳境

### 静态分析

经过抓包分析后，得知加密参数为`mfsig`，开发工具中全局搜索`mfsig`，发现并没有匹配的结果。

![image-20210706155415889](https://bbs.kanxue.com/upload/attach/202107/928237_D3QP9THWRUPKBVR.jpg)

转换思路，我们看到`mfsig`的取值均为`mfsw`开头，于是全局搜索这个，发现`mfsw`也无匹配结果，可以得出个结论：

该小程序的加密相关函数是特殊对待混淆过的，静态分析明文无法定位到。算是一个安全性比较好的 case。

小程序的代码一般发布时都会做混淆，一般而言单靠静态分析代码中的加密逻辑是很费时费力的，借助调试反而易于理解代码逻辑。

既然静态分析无果，这时就要体现动态调试分析的重要性了。

### 动态分析

#### 小程序编译

打开模拟器，并点击编译按钮，观察模拟器窗口和调试器的 Console 窗口中的报错提示。

期间会遇到几个很小的报错，逐步解决后成功看到主界面，接下来就可以调试了。

![image-20210706162859382](https://bbs.kanxue.com/upload/attach/202107/928237_XQYMH65TQV5EECS.jpg)

#### 动态分析过程

找到对应接口，打好断点后一步步调试分析。

定位到核心代码，反编译的代码格式比较乱，多行代码挤在同一行，不利于追步调试，可以点击左下角的 `{}` 按钮进行格式化。

![image-20210706160107480](https://bbs.kanxue.com/upload/attach/202107/928237_JHUVPSFBE4NKUDX.jpg)

通过调试器右侧的调试功能按键进行追踪调试。

![image-20210706160201085](https://bbs.kanxue.com/upload/attach/202107/928237_UGHW539224W77RV.jpg)

加密函数混淆过，需要些耐心一步步调试，纸上记录下加密过程。

大体上来看其实就是编码游戏，字符串转数组，数组转为字符串，再利用索引进行字符编码生成最后的`mfsig`。

## 元神初具

### 加密翻译

经过上述动态解析，将纸上记录下的加密流程进行整理，利用 Java 或 Python 进行翻译，实现一遍。

拿一个真实抓包接口的请求体参数进行测试，验证加密函数的正确性。

![image-20210706160955448](https://bbs.kanxue.com/upload/attach/202107/928237_C6JJENM42MM86F2.jpg)

得到的`mfsig`与接口中的完全一致，那么大功告成。

## 妙领天机

### 总结

1. 逆向需要耐心，也需要大胆的猜想和假设去不断尝试。混淆的代码笔者调试追溯了好几天，算是比较笨的。
2. 逆向工作会用到很多好用的工具，平时注意多收集一些好用的工具或者好博文，以事半功倍。
3. 本文旨在分享一些逆向技巧和思路，读者不可利用本文所述内容进行非法商业获取利益，若执意带来的法律责任由读者自行承担。

[[招生]科锐逆向工程师培训（2026年11月13日实地，远程教学同时开班，第57期）](https://bbs.kanxue.com/thread-51839.htm)

最后于 2021-7-6 17:02 被灵风_spirit 编辑，原因：格式修改。

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)

[#源码框架](https://bbs.kanxue.com/forum-161-1-127.htm)

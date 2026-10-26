
# [原创]某大厂生鲜超市加密协议分析

## 接口抓包分析

### 工具

**Charles**

关于 charles 的使用可翻阅我之前的 charles 专题文章

### 测试手机

**Nexus 5x**

**郑重声明，本文只分享思路，不做它用，为保护案例商家安全隐私，敏感信息用 xxx 代替**

### 接口：附近可服务的门店

`curl`

所有接口调用 url 均为<a href="elink@901K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3I4G2N6i4u0Q4x3X3g2^5P5s2S2^5P5s2S2^5i4K6u0W2j5$3!0E0i4K6u0r3j5i4m8A6"><mark class="encrypted">991K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3I4G2N6i4u0Q4x3X3g2^5P5s2S2^5P5s2S2^5i4K6u0W2j5$3!0E0i4K6u0r3j5i4m8A6</mark></a>，通过 `postbody` 参数 `functionId` 控制获取具体的数据

```bash
curl -H 'Host: colour.xxxxxxx.com' -H 'x-mlaas-at: wl=0' -H 'user-agent: xxxxxxxapp_android' -H 'content-type: application/x-www-form-urlencoded; charset=utf-8' --data-binary "commonExtend=&loginType=4&sign=b6beeee33ad4142cc54f3e55a045fbb1c70ecdfdbffa985b559cc36797d20357&screen=1794*1080&d_brand=LGE&body=%7B%22commonExtend%22%3A%22%22%2C%22data%22%3A%7B%22lon%22%3A%22120.02877%22%2C%22lat%22%3A%223x.278442%22%7D%2C%22appName%22%3A%22xxxxxxx%22%2C%22screen%22%3A%221794*1080%22%2C%22lon%22%3A%221xx.143936%22%2C%22platformId%22%3A%221%22%2C%22clientVersion%22%3A%223.6.4%22%2C%22storeId%22%3A%22232686%22%2C%22recommendSwitch%22%3A%22true%22%2C%22eu%22%3A%2275B6364667C69667%22%2C%22fv%22%3A%220333461727947597%22%2C%22osVersion%22%3A%228.1.0%22%2C%22partner%22%3A%22huawei%22%2C%22v%22%3A2%2C%22tenantId%22%3A%221%22%2C%22client%22%3A%22android%22%2C%22clientVersionBuild%22%3A%222110251117%22%2C%22model%22%3A%22Nexus5X%22%2C%22networkType%22%3A%22wifi%22%2C%22brand%22%3A%22LGE%22%2C%22lat%22%3A%223x.323437%22%7D&clientVersion=3.6.4&eu=75B6364667C69667&fv=0333461727947597&d_model=Nexus5X&functionId=xxxxxxx_platform_address_getPosition&t=1636957653670&partner=huawei&osVersion=8.1.0&build=2110251117&appid=****fresh_APP&client=xxxxxxx_android&lang=zh_CN&networkType=wifi" --compressed 'https://colour.xxxxxxx.com/api'
```

`postBody` URL 解码后

```text
commonExtend=&loginType=4&sign=b6beeee33ad4142cc54f3e55a045fbb1c70ecdfdbffa985b559cc36797d20357&screen=1794*1080&d_brand=LGE&body={"commonExtend":"","data":{"lon":"xxx.02877","lat":"xx.278442"},"appName":"xxxxxxx","screen":"1794*1080","lon":"xxx.143936","platformId":"1","clientVersion":"3.6.4","storeId":"232686","recommendSwitch":"true","eu":"75B6364667C69667","fv":"0333461727947597","osVersion":"8.1.0","partner":"huawei","v":2,"tenantId":"1","client":"android","clientVersionBuild":"2110251117","model":"Nexus5X","networkType":"wifi","brand":"LGE","lat":"xx.323437"}&clientVersion=3.6.4&eu=75B6364667C69667&fv=0333461727947597&d_model=Nexus5X&functionId=xxxxxxx_platform_address_getPosition&t=1636957653670&partner=huawei&osVersion=8.1.0&build=2110251117&appid=****fresh_APP&client=xxxxxxx_android&lang=zh_CN&networkType=wifi
```

接口正确返回 `responseBody`

```json
{
"code": "0",
"success": true,
"msg": null,
"data": {
"success": true,
"businessCode": 0,
"msg": null,
"type": 1,
"locationInfo": {
"addressExt": "浙江xxxxxxx",
"addressSummary": "浙江省xxx",
"storeId": null,
"lat": "xx.27x442",
"lon": "xxx.02x77",
"testShop": false
},
"defaultAddress": null,
"tenantShopInfoList": [{
"storeId": 232xxx,
"storeName": "华东****鲜云超",
"storeAddress": "江东中路与江东门北街交汇处",
"promiseInfo": "最快30分钟达 | 230.96KM",
"tenantDesc": "",
"businessInfo": "",
"tenantInfo": {
"tenantId": 1,
"tenantName": "****鲜",
"bigLogo": "http://img12.360buyimg.com/freshapp/jfs/t1/144864/26/9153/27011/5f6ae507E9dfc96a5/fc2f58d77bcbf2cd.png",
"smallLogo": "http://img12.360buyimg.com/freshapp/jfs/t1/143548/8/13335/6626/5fa4affbE87f4ded3/f46b57081818d3ba.png",
"circleLogo": "http://img12.360buyimg.com/freshapp/jfs/t1/140501/2/9085/6589/5f6ae50bE0508f49c/ae52b1fc1dc2aa59.png",
"contactTel": "4006068768",
"supportGiftCard": false,
"supportEmployeeCard": false,
"supportInvoiceCenter": false,
"supportBalance": false,
"clientInfo": null
},
"lon": "118.737681",
"lat": "32.036757",
"valid": true,
"freeBuy": false,
"delivery": false
}, {
"storeId": 196243,
"storeName": "华中****鲜云超",
"storeAddress": "光谷保利广场",
"promiseInfo": "最快30分钟达 | 539.52KM",
"tenantDesc": "",
"businessInfo": "",
"tenantInfo": {
"tenantId": 1,
"tenantName": "****鲜",
"bigLogo": "http://img12.360buyimg.com/freshapp/jfs/t1/144864/26/9153/27011/5f6ae507E9dfc96a5/fc2f58d77bcbf2cd.png",
"smallLogo": "http://img12.360buyimg.com/freshapp/jfs/t1/143548/8/13335/6626/5fa4affbE87f4ded3/f46b57081818d3ba.png",
"circleLogo": "http://img12.360buyimg.com/freshapp/jfs/t1/140501/2/9085/6589/5f6ae50bE0508f49c/ae52b1fc1dc2aa59.png",
"contactTel": "4006068768",
"supportGiftCard": false,
"supportEmployeeCard": false,
"supportInvoiceCenter": false,
"supportBalance": false,
"clientInfo": null
},
"lon": "114.410486",
"lat": "30.490744",
"valid": true,
"freeBuy": false,
"delivery": false
}],
"nearStore": false,
"fix": false,
"fixLat": null,
"fixLon": null
},
"extMap": {}
}
```

接口错误返回：接口有调用时效，会检测时间戳参数 `t`，时效性为五分钟，五分钟后再次调用返回异常

```json
{"code":"1","echo":"invalid signature"}
```

接口分析得知，有三个加密参数 `sign`、`eu`、`fv`

## 逆向分析

### 工具

`jadx`-反编译

`frida`+`objection` 动态调试

### `sign` 加密分析

`sign` 从字符串特征和长度来看，看起来像 `sha256`

`jadx` 打开 apk 搜索关键字“`HmacSha256`”，看到加密 `HmacSha256` 搜选出多个

![image-20211118200626723](upload/attach/202111/928237_Z4JWHHG8Y9VQFQU.jpg)

`frida`+`objection` 跟踪调用入参

需要 `objection` 个个追踪然后和抓包得到的 `sign` 比对，最终确定调用的为 `com.xxx.common.http.GatewaySignatureHelper.HMACSHA256`

![image-20211118200712661](upload/attach/202111/928237_Z7BTQEYMS685SPY.jpg)

![image-20211118200745887](upload/attach/202111/928237_JHAP4H77SAV466G.jpg)

`objection` 追踪

```bash
android hooking watch class_method com.xxx.common.http.GatewaySignatureHelper.HMACSHA256 --dump-args --dump-return --dump-backtrace
```

`--dump-backtrace` 追踪调用方法栈

```text
(agent) [172759] Called com.xxx.common.http.GatewaySignatureHelper.HMACSHA256([B, [B)
(agent) [172759] Backtrace:
com.xxx.common.http.GatewaySignatureHelper.HMACSHA256(Native Method)
com.xxx.common.http.GatewaySignatureHelper.signature(TbsSdkJava:60)
com.xxx.common.http.HttpRequest.paramHandler(TbsSdkJava:108)
com.xxx.common.http.HttpRequest.add(TbsSdkJava:9)
com.xstore.****fresh.modules.search.SearchRequest.getWareInfosIcon(TbsSdkJava:11)
com.xstore.****fresh.modules.productdetail.utils.GetWareInfoIconUtils.getWareInfoMsg(TbsSdkJava:7)
com.xstore.****fresh.modules.category.menulist.NewProductCategoryFragment.setListView(TbsSdkJava:34)
com.xstore.****fresh.modules.category.menulist.NewProductCategoryFragment.initView(TbsSdkJava:32)
com.xstore.****fresh.modules.category.menulist.NewProductCategoryFragment.onCreateView(TbsSdkJava:3)
androidx.fragment.app.Fragment.performCreateView(TbsSdkJava:4)
androidx.fragment.app.FragmentStateManager.createView(TbsSdkJava:15)
androidx.fragment.app.FragmentStateManager.moveToExpectedState(TbsSdkJava:23)
androidx.fragment.app.FragmentManager.executeOpsTogether(TbsSdkJava:34)
androidx.fragment.app.FragmentManager.removeRedundantOperationsAndExecute(TbsSdkJava:10)
androidx.fragment.app.FragmentManager.execPendingActions(TbsSdkJava:4)
androidx.fragment.app.FragmentManager$5.run(TbsSdkJava:1)
android.os.Handler.handleCallback(Handler.java:790)
android.os.Handler.dispatchMessage(Handler.java:99)
android.os.Looper.loop(Looper.java:164)
android.app.ActivityThread.main(ActivityThread.java:6494)
java.lang.reflect.Method.invoke(Native Method)
com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:438)
com.android.internal.os.ZygoteInit.main(ZygoteInit.java:807)
(agent) [172759] Arguments com.xxx.common.http.GatewaySignatureHelper.HMACSHA256([object Object], [object Object])
(agent) [172759] Return Value: b196066d6b926a9e032ea9ae1b0a52a048b03ac063ad55bfbc3fca0fe88959c6
```

上游重要的方法为这三个：

`com.xxx.common.http.GatewaySignatureHelper.HMACSHA256(Native Method)`

`com.xxx.common.http.GatewaySignatureHelper.signature(TbsSdkJava:60)`

`com.xxx.common.http.HttpRequest.paramHandler(TbsSdkJava:108)`

追 `signature`

```bash
android hooking watch class_method com.xxx.common.http.GatewaySignatureHelper.signature --dump-args --dump-return
```

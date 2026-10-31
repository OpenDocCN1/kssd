

# [原创]某宝联盟——淘口令解析

## 前言

本文的目的只有一个，就是学习更多的逆向技巧和思路。如果有人利用本文技术进行非法商业活动并获取利益，由此带来的法律责任均由操作者自行承担，与本文及作者无关。

在研究 `alimama` 的 `sgin` 过程中，无意中发现了淘口令解析。

## 工具准备

- Android Studio
- Android 模拟器
- `ddms`
- `jadx` 工具
- `xposed` 工程
- `Smali` 工程
- 在线淘口令解析（如有变更请自行百度）：
  `409K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0V1I4N6r3!0G2L8q4)9J5k6h3y4G2L8g2)9J5c8Y4c8C8L8r3A6^5`

## UI 界面

![](https://bbs.kanxue.com/upload/attach/201912/746297_CYPHJS22GW6TCPP.jpg)

## 全文搜索 `ClipboardManager`

使用过淘口令的人都知道，要让 `ali` 系的 App 能自动识别淘口令，前提必须是先复制淘口令到 Android 系统的粘贴板，所有我们就从 `ClipboardManager` 进行入手。

全文搜索 `ClipboardManager`，然后进行分析排查。

![](https://bbs.kanxue.com/upload/attach/201912/746297_BJ3XJSCT5FZ9QCR.jpg)

逐步排除，定位到关键代码：`TaoCodeTransferPresenter-itemInfoListener`

## `ddms` 抓取第一个位置：`TaoCodeTransferPresenter-itemInfoListener` 对象

分析后再用 `ddms` 进行轨迹跟踪，发现其实 `ali` 系的 App 大量使用了 `RxJava` 技术。

![](https://bbs.kanxue.com/upload/attach/201912/746297_W6Q26TTPU6ZNH8U.jpg)

```java
if (rxMtopResponse.isReqSuccess) {

    TaoCodeItemInfo taoCodeItemInfo = (TaoCodeItemInfo) rxMtopResponse.result;

    if (taoCodeItemInfo != null && !TextUtils.isEmpty(taoCodeItemInfo.getRawUrl())) {

        EventBus.getDefault().post(TaoCodeTransferEvent.success(TaoCodeTransferPresenter.this.mClipboardContent, taoCodeItemInfo));

        TaoCodeTransferPresenter.this.cleanClipboard();

        return;

    }

    return;

}
```

![](https://bbs.kanxue.com/upload/attach/201912/746297_VF36EVAEMZXU5TG.jpg)

## 第二个位置：`BaseActivity -> onTaoCodeTransferEvent`

![](https://bbs.kanxue.com/upload/attach/201912/746297_ER7K39E2QM9EXMF.jpg)

![](https://bbs.kanxue.com/upload/attach/201912/746297_5CM28H8DF8VKJW8.jpg)

## 第三个位置：`TransformTaoCodeDialog`

使用 `xposed` 进行 Hook 跟踪：

`private final RxMtopResult<TaoCodeItemInfo> itemInfoListener = new RxMtopResult<TaoCodeItemInfo>()` —— `TaoCodeTransferPresenter$2`

Hook 对应原始代码：

```java
private final RxMtopResult<TaoCodeItemInfo> itemInfoListener = new RxMtopResult<TaoCodeItemInfo>() {

    public void result(RxMtopResponse<TaoCodeItemInfo> rxMtopResponse) {

        。。。。

    }

};
```

![](https://bbs.kanxue.com/upload/attach/201912/746297_WT34VR3RTCKRCTE.jpg)

### 字段描述

- `pictUrl`：商品图片
- `rawUrl`：商品信息地址
- `title`：标题
- `couponAmount`：优惠券金额
- `money`：赚金额——转发优惠券
- `afterCouponAmount`：优惠后金额
- `price`：商品价格——优惠券前价格

![](https://bbs.kanxue.com/upload/attach/201912/746297_S673YCX2K8M9B7D.jpg)

![](https://bbs.kanxue.com/upload/attach/201912/746297_8HFQ9EXNZR5PQXP.jpg)

![](https://bbs.kanxue.com/upload/attach/201912/746297_FCBERJQPZ5QAZPS.jpg)

## 淘口令的例子

![](https://bbs.pediy.com/upload/attach/201912/746297_W33WGQSZ2ZTWYDY.jpg)

## Debug 方式启动

```bash
adb shell am start -D -n com.alimama.moon/com.alimama.moon.ui.WizardActivity
```

## Hook 分析构造淘口令对象

`TaoCodeTransferPresenter(@NonNull Context context)` 即 `this.itemInfoListener`。

![](https://bbs.kanxue.com/upload/attach/201912/746297_XNUDR5ESRGTRMMC.jpg)

## 开始构造 Hook 代码

```java
// new TaoCodeItemInfoRequest(this.mClipboardContent).sendRequest(this.itemInfoListener);

Object obj_mTaoCodeTransferPresenter = XposedHelpers.getObjectField(HookAlimama.appContext, "mTaoCodeTransferPresenter");

Object obj_itemInfoListener = XposedHelpers.getObjectField(obj_mTaoCodeTransferPresenter, "itemInfoListener");

Class<?> clz_TaoCodeItemInfoRequest = HookAlimama.myWechatloader.loadClass("com.alimama.union.app.taotokenConvert.TaoCodeItemInfoRequest");

String str_taokouling = "【第一卫数据线三合一充电线器一拖三手机快充苹果安卓三头多头二合一多功能三线多用华为一拖二车载快冲3三用】59dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0i4K6u0W2N6r3u0Q4x3X3g2U0L8W2)9J5c8X3S2Q4x3X3g2W2N6W2S2^5g2r3A6@1i4K6y4r3M7$3#2Q4x3@1b7H3j5e0j5#2y4$3u0Q4x3U0k6F1j5Y4y4H3i4K6y4n7i4@1f1&6i4@1u0n7i4K6W2q4i4@1f1K6i4K6R3J5i4K6W2q4i4@1f1$3i4K6V1J5i4K6R3K6i4@1x3J5i4@1t1H3i4@1f1&6i4K6S2r3i4K6R3^5i4@1f1K6i4K6R3K6i4K6V1H3i4@1f1$3i4K6S2q4i4@1p5#2i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4K6R3$3i4K6S2p5i4@1f1&6i4K6R3H3i4K6R3&6i4@1f1$3i4K6S2n7i4@1p5&6i4@1f1%4i4K6R3H3i4K6S2r3i4@1f1^5i4@1p5$3i4@1u0p5i4@1f1J5i4K6R3$3i4K6V1J5i4@1f1#2i4K6V1&6i4K6V1H3i4@1f1#2i4K6V1J5i4K6V1I4i4@1f1&6i4K6V1$3i4K6W2q4i4@1f1K6i4K6R3K6i4@1u0p5i4@1g2r3i4@1u0o6i4K6W2n7i4@1f1$3i4K6R3^5i4K6V1$3i4@1f1$3i4@1p5@1i4@1t1I4i4@1f1K6i4K6R3J5i4@1p5I4i4@1f1^5i4@1p5K6i4@1u0p5i4@1f1$3i4K6V1#2i4@1t1@1i4@1f1#2i4K6S2r3i4@1p5#2i4@1f1^5i4@1q4r3i4K6W2p5i4@1g2r3i4@1u0r3i4@1p5#2k6q4W2c8y4g2W2p5y4f1k6F1y4X3!0Q4c8f1k6Q4b7V1k6Q4b7e0g2Q4c8e0g2Q4z5e0m8Q4z5p5g2Q4c8e0g2Q4z5e0u0Q4z5e0q4Q4c8e0W2Q4z5e0k6Q4z5f1g2Q4c8e0k6Q4z5o6W2Q4z5p5u0Q4c8e0k6Q4z5f1y4Q4b7V1q4Q4c8e0g2Q4b7e0c8Q4b7e0W2Q4c8e0N6Q4z5p5y4Q4b7f1t1";
 
XposedHelpers.newInstance(clz_TaoCodeItemInfoRequest, str_taokouling);
```

## 添加回调代码

```java
private static void callbackInTaoCodeTransferPresenter$2_result(ClassLoader classLoader) {

    try {

        Class<?> clz_RxMtopResponse = classLoader.loadClass("com.alimama.union.app.rxnetwork.RxMtopResponse");

        XposedHelpers.findAndHookMethod(
            "com.alimama.union.app.taotokenConvert.TaoCodeTransferPresenter$2",
            classLoader,
            "result",
            clz_RxMtopResponse,
            new XC_MethodHook() {

                @Override
                protected void afterHookedMethod(MethodHookParam param) throws Throwable {

                    super.afterHookedMethod(param);

                    Object obj_result_param = param.args[0];

                    String retCode = (String) XposedHelpers.getObjectField(obj_result_param, "retCode");

                    if ("SUCCESS".equals(retCode)) {

                        Object obj_result_param_result_Field = XposedHelpers.getObjectField(obj_result_param, "result");

                        String picUrl = (String) XposedHelpers.getObjectField(obj_result_param_result_Field, "pictUrl");

                        String rawUrl = (String) XposedHelpers.getObjectField(obj_result_param_result_Field, "rawUrl");

                        String title = (String) XposedHelpers.getObjectField(obj_result_param_result_Field, "title");

                        Log.i(TAG, "picUrl= " + picUrl);

                        Log.i(TAG, "rawUrl= " + rawUrl);

                        Log.i(TAG, "title= " + title);
                    }
                }
            }
        );

    } catch (ClassNotFoundException e) {

        e.printStackTrace();

    }

}
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2019-12-10 19:20

被 `younghare` 编辑

，原因：


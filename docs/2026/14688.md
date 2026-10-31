
# [原创]我的微信数据监控研究发展过程

> 严重声明
>
> 本文的意图只有一个就是通过分析app学习更多的逆向技术，如果有人利用本文知识和技术进行非法操作进行牟利，带来的任何法律责任都将由操作者本人承担，和本文作者无任何关系，最终还是希望大家能够秉着学习的心态阅读此文

以下内容适用范围：已经获取root权限的手机

近两年来一直在做Android端微信的消息备份功能，一开始是调研自己实现，现在是公司业务需要，现已实现好友相关信息备份，监控消息、领取红包、监控发布朋友圈数据、监控朋友圈点赞回复等功能。

# 微信相关基本知识：

1. 聊天消息表、联系人表、头像信息表、联系人标签表、红包转账信息表等都存在`EnMicroMsg.db`数据库中，但该数据库被加密，需要密码才能查看。
2. `EnMicroMsg.db`地址是`/data/data/com.tencent.mm/MicroMsg/32位字符串/EnMicroMsg.db`，32位字符串是通过`MD5`加密(`"mm"+uin码`)得出的，如果有多个32位字符串的文件夹，说明登录过多个微信号，微信号不同，uin码不同。同一级的还有其他数据库，比如`SnsMicroMsg.db`，存放的是微信朋友圈相关表，该数据库无密码，可直接查看。
3. `EnMicroMsg.db`的密码是`MD5(imei码+uin码).substring(0, 7).toLowerCase()`得到，imei码是通过获取手机IMEI码得到，uin码是微信`SharedPerferences`中存储的，文件位置`\data\data\com.tencent.mm\shared\_prefs\auth\_info\_key\_prefs.xml`，通过解析xml文件来得到`\_auth\_uin`字段的值。
4. 当前登录微信的账号ID存放于`\data\data\com.tencent.mm\shared\_prefs\notify\_key\_pref\_no\_account.xml`文件中，通过解析xml文件来得到`login\_weixin\_username`字段的值。

## 1.静态监控

是我一开始参考网上文章后实现监控的方法，使用的是定时复制数据库，并读取的方法

定时复制微信的数据库文件到我的文件夹下，计算得到密码后，使用`sqlcipher`连接并传入密码来读取数据，然后发送到服务器。

## 2.动态监控

可以看到静态监控的方法并不完善，并不能做到实时监控，在我接触到`Xposed`后开始使用hook 拦截的方式来监控微信数据

可以说微信是我反编译大厂app后发现的少数不做加固的app之一，让我省去了脱壳的步骤

这里主要对监控发布朋友圈数据进行详细教程，包含xposed的大多数使用。

### -1 使用 当前Activity 这个app来查看发布朋友圈页面的包名和Activity。

![](https://bbs.kanxue.com/upload/attach/201901/816443_A43BEUZZMM4F8PQ.jpg)

当然也可以使用`adb`来查看

### -2 使用 `jadx`打开微信apk，本案例使用的是6.6.7版本，搜索类`SnsUploadUI`

![](https://bbs.kanxue.com/upload/attach/201901/816443_BZ8B7A9JYJE887V.png)

可以看到有一个`SnsEditText`的变量，很大把握可以确定这个变量就是发朋友圈的文本输入框，

### -3 找到发表按钮

看到发表按钮，可以确定它要么是一个`MenuItem`，要么是一个普通view，但总会有点击事件

先查找`SnsEditText`，发现它继承自`EditText`，直接在这个类中搜索 `nRk.getText` 的相关调用，很容易找到调用位置。

![](https://bbs.kanxue.com/upload/attach/201901/816443_WJ5HXH7MZVFD4X5.png)

### -4 进行hook

```
try {

    final Class<?> SnsUploadUIClass = cl.loadClass("com.tencent.mm.plugin.sns.ui.SnsUploadUI");

        XposedHelpers.findAndHookMethod(SnsUploadUIClass, "onCreate", Bundle.class,

            new XC_MethodHook() {

                @Override
                protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                    super.beforeHookedMethod(param);
                    Object oSnsUploadUI = param.thisObject;
                    XposedHelpers.findAndHookMethod("com.tencent.mm.plugin.sns.ui.SnsUploadUI$5$1",
                        mlpparam.classLoader, "pO", String.class,new XC_MethodHook() {
                            @Override
                            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                                super.beforeHookedMethod(param);
                                //获取SnsUploadUIClass中的nRk变量
                                Object nRk=XposedHelpers.findField(SnsUploadUIClass, "nRk").get(oSnsUploadUI);

                                //方法1
                                //强转为EditText  
                                EditText editText = (EditText) nRk;
                                //得到发表朋友圈的文本
                                String text=editText.getText().toString();

                                //方法2
                                //执行getText().toString()得到发表朋友圈的文本
                                String text=  XposedHelpers.callMethod(nRk,"getText").toString();

                                //todo上传数据
                            }
                    });
                }
            });
} catch (Exception e) {
    e.printStackTrace();
}
```

因为源码上有`SnsUploadUI.this.nRk.setText("")`这段，所以要在`beforeHookedMethod`中拦截

方法1是因为`SnsEditText`继承自`EditText`的，所以通过`findField`后直接强转成`EditText`，然后再获取字符串。

方法2是如果不知道`SnsEditText`是怎样的类的话，但我们知道`nRk`通过`nRk.getText().toString()`得到了文本信息，可以通过`XposedHelpers.callMethod`得到

需要说明的是`beforeHookedMethod`主要是截取方法参数，`afterHookedMethod`主要截取方法返回值，如果一个变量在方法中值被改变了，那就看你是要改变前的还是改变后的了。

至此拦截成功，使用`Toast`显示了一下。

![](https://bbs.kanxue.com/upload/attach/201901/816443_GPFDZKQNUEQ9YHT.jpg)

以上代码使用了xposed 的`findAndHookMethod`，`findField`，`callMethod`等方法，相对全面系统。当然还有`callStaticMethod`、`findAndHookConstructor`等常用方法，最好查看[Xposed Api文档](https://bbs.kanxue.com/elink@0e4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7r3W2Q4x3X3g2^5M7r3!0K6k6h3c8Q4x3X3g2A6L8X3k6G2i4K6u0r3M7X3g2X3k6i4u0W2L8X3y4W2i4K6u0r3M7r3q4U0K9$3q4Y4k6i4y4Q4x3X3g2Z5N6r3#2D9)进行学习，多多使用

使用xposed，最难的可能就是找到hook点，一般对于微信这种大量依赖数据库的，要么找到数据解析方法来拦截，要么找到数据入库或者查询操作自己解析判断来拦截

如果一开始无处下手，可以先找到它的数据库相关`insert`、`update`、`delete`、`query`操作，这样就可以通过打印的sql查询方法知道它每个页面查的是哪个数据表，然后分析数据表的结构和字段含义，再找到执行页面进行代码分析。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2019-1-30 13:59

被JovenHe编辑，原因：

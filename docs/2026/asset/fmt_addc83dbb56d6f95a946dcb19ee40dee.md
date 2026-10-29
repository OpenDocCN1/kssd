
```
#############################################
#
# 对于一些基本指令的添加
#
#############################################
# 代码混淆压缩比，在0~7之间，默认为5，一般不做修改
-optimizationpasses 5
# 混合时不使用大小写混合，混合后的类名为小写
-dontusemixedcaseclassnames
# 指定不去忽略非公共库的类
-dontskipnonpubliclibraryclasses
# 这句话能够使我们的项目混淆后产生映射文件
# 包含有类名->混淆后类名的映射关系
-verbose
# 指定不去忽略非公共库的类成员
-dontskipnonpubliclibraryclassmembers
# 不做预校验，preverify是proguard的四个步骤之一，Android不需要preverify，去掉这一步能够加快混淆速度。
-dontpreverify
# 保留Annotation不混淆
-keepattributes *Annotation*,InnerClasses
# 避免混淆泛型
-keepattributes Signature
# 抛出异常时保留代码行号
-keepattributes SourceFile,LineNumberTable
# 指定混淆是采用的算法，后面的参数是一个过滤器
# 这个过滤器是谷歌推荐的算法，一般不做更改
-optimizations !code/simplification/cast,!field/*,!class/merging/*
#############################################
#
# Android开发中一些需要保留的公共部分
#
#############################################
# 保留我们使用的四大组件，自定义的Application等等这些类不被混淆
# 因为这些子类都有可能被外部调用
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Appliction
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.app.backup.BackupAgentHelper
-keep public class * extends android.preference.Preference
-keep public class * extends android.view.View
-keep public class com.android.vending.licensing.ILicensingService
# 保留support下的所有类及其内部类
-keep class android.support.** {*;}
# 保留继承的
-keep public class * extends android.support.v4.**
-keep public class * extends android.support.v7.**
-keep public class * extends android.support.annotation.**
# 保留R下面的资源
-keep class **.R$* {*;}
# 保留本地native方法不被混淆
-keepclasseswithmembernames class * {
native <methods>;
}
# 保留在Activity中的方法参数是view的方法，
# 这样以来我们在layout中写的onClick就不会被影响
-keepclassmembers class * extends android.app.Activity{
public void *(android.view.View);
}
# 保留枚举类不被混淆
-keepclassmembers enum * {
public static **[] values();
public static ** valueOf(java.lang.String);
}
# 保留我们自定义控件（继承自View）不被混淆
-keep public class * extends android.view.View{
*** get*();
void set*(***);
public <init>(android.content.Context);
public <init>(android.content.Context, android.util.AttributeSet);
public <init>(android.content.Context, android.util.AttributeSet, int);
}
# 保留Parcelable序列化类不被混淆
-keep class * implements android.os.Parcelable {
public static final android.os.Parcelable$Creator *;
}
# 保留Serializable序列化的类不被混淆
-keepclassmembers class * implements java.io.Serializable {
static final long serialVersionUID;
private static final java.io.ObjectStreamField[] serialPersistentFields;
!static !transient <fields>;
!private <fields>;
!private <methods>;
private void writeObject(java.io.ObjectOutputStream);
private void readObject(java.io.ObjectInputStream);
java.lang.Object writeReplace();
java.lang.Object readResolve();
}
# 对于带有回调函数的onXXEvent、**On*Listener的，不能被混淆
-keepclassmembers class * {
void * (**On*Event);
void * (**On*Listener);
}
# webView处理，项目中没有使用到webView忽略即可
-keepclassmembers class fqcn.of.javascript.interface.for.webview {
public *;
}
-keepclassmembers class * extends android.webkit.webViewClient {
public void *(android.webkit.WebView, java.lang.String, android.graphics.Bitmap);
public boolean *(android.webkit.WebView, java.lang.String);
}
-keepclassmembers class * extends android.webkit.webViewClient {
public void *(android.webkit.webView, jav.lang.String);
}
```

### 3.2 so层混淆

这个之前已经写过了：

文档：函数混淆（JNI_Onload）.note

链接：`03cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3&6G2N6r3g2Q4x3X3g2&6L8%4g2V1j5h3!0Q4x3X3g2U0L8$3#2Q4x3V1k6F1L8%4c8W2M7$3S2S2M7X3g2Q4x3@1k6A6k6q4)9K6c8r3j5$3j5h3c8V1x3h3k6X3y4X3p5$3j5K6c8T1y4K6S2S2j5h3x3@1j5e0W2W2x3U0N6X3k6e0x3&6x3r3g2V1i4K6t1$3M7%4g2T1i4K6y4p5x3o6c8r3c8U0c8q4x3V1t1I4c8U0f1H3y4o6f1^5y4@1p5H3y4V1x3$3z5f1j5K6z5f1x3@1c8p5j5J5x3V1x3`.

### 3.3 签名验证

#### 3.3.1 在MainActivity.java中进行签名验证

```java
package com.example.signatureverify;
import androidx.appcompat.app.AppCompatActivity;
import android.content.Context;
import android.content.pm.PackageInfo;
import android.content.pm.PackageManager;
import android.content.pm.Signature;
import android.os.Bundle;
import android.widget.Toast;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Locale;
public class MainActivity extends AppCompatActivity {
@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate (savedInstanceState);
setContentView (R.layout.activity_main);
Context context =getApplicationContext ();
Toast.makeText (context, "我是正版", Toast.LENGTH_SHORT).show ();
String cert_sha1="59F8A6B86A367F0586F1A15DDDB63D75263C5D62"; // 通过调试提前获取apk的sha1签名
boolean is_org_app = false;
try {
is_org_app = isOrgApp(context,cert_sha1);
} catch (PackageManager.NameNotFoundException e) {
e.printStackTrace ();
} catch (NoSuchAlgorithmException e) {
e.printStackTrace ();
}
if(!is_org_app){
android.os.Process.killProcess ((android.os.Process.myPid ()));
//如果签名不一致，说明程序被修改了，直接退出
}
}
//比较签名
private boolean isOrgApp(Context context, String cert_sha1) throws PackageManager.NameNotFoundException, NoSuchAlgorithmException {
String current_sha1=getAppSha1(context);
current_sha1=current_sha1.replace (":","");
return cert_sha1.equals (current_sha1);
}
//生成sha1的签名
private String getAppSha1(Context context) throws PackageManager.NameNotFoundException, NoSuchAlgorithmException {
PackageInfo info=context.getPackageManager ().getPackageInfo (context.getPackageName (),PackageManager.GET_SIGNATURES);
byte[] cert =info.signatures[0].toByteArray ();
MessageDigest md =MessageDigest.getInstance ("SHA1");
byte[] publicKey=md.digest (cert);
StringBuffer hexString =new StringBuffer ();
for(int i=0;i<publicKey.length;i++){
String appendString=Integer.toHexString (0xFF&publicKey[i]).toUpperCase(Locale.US);
if(appendString.length ()==1){
hexString.append("0");
}
hexString.append(appendString);//签名的格式是11:22,所以需要加上":"
hexString.append (":");
}
String result=hexString.toString ();
return result.substring (0,result.length ()-1);
}
}
```

完成程序后，使用`AndroidKiller`进行修改时，将"我是正版"字符串修改成"我是盗版"，在运行程序时，程序会直接**闪退**

![在这里插入图片描述](upload/attach/202003/845506_6BEKNWS45BTFVDX.jpg)

#### 3.3.2 在So层中进行签名验证

`MainActivity`：

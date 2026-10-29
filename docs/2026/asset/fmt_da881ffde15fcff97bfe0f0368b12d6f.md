
```java
Normal:最低等级，系统会默认授予次权限，不会提示用户，低风险，所以的APP不能访问和共享此APP
Dangerous:系统在安装时声明此类权限的app会提示用户，是高风险，所以APP都能访问和共享此APP
Signature:权限表明的操作只针对使用同一个证书签名的app开放，是指具有相同签名的APP可以访问和共享此APP
SignatureOrSystem:系统images中APP和具有相同签名的APP可以访问和共享此APP,google建议不使用该选项，一般用于特定的一些功能
```

## 三、权限安全漏洞原理分析和复现

### 1. 漏洞原理

介绍权限漏洞之前，我们先讲解一下Android组件基本知识，前几篇帖子我们也介绍过，Android程序运行在独立的沙箱环境中，相互隔离，为了方便通信，Android提供了组件间的通信（ICC）机制，允许满足条件的组件相互传递数据，ICC的实现依赖于`Intent`和`Binder`机制，其底层是通过进程间通信来实现的。在组件的通信过程中，`Intent`是数据传播的载体，通信发起方发送`Intent`，携带执行的动作、动作相关的数据和附加数据等信息，通信接收方预先定义自身的`Intent-filter`，`intent-filter`中包含能够响应`intent`的动作和数据类型，声明能够响应的`intent`请求，应用程序框架在通信发起`Intent`后，负责找到能处理该`Intent`的接收方。

![image-20211026112045142](upload/attach/202110/905443\_CQTJYR7EGM743JE.png)

组件间通信的一个前提是接收方组件必须公开，Android中的组件分为公开组件和私有组件，具体有`export`是否可导出决定，而Android权限泄漏漏洞就是由于不合理的公开了组件或者没有对接收的`Intent`进行合法性校验导致，其表现为将一个拥有权限的API通过公开组件暴露给外界，攻击者可以构造数据访问这个API达到权限提升的效果，一般表现为两个方面：隐私获取和动作执行

```java
隐私获取：指的是攻击者通过构造数据访问存在权限泄露漏洞的程序组件，并让该组件返回隐私数据，这些数据包括短信消息、联系人信息、地理位置、邮件信息等等
动作执行：指的是攻击者构造数据访问存在权限泄露的程序组件，让该组件执行系统破坏等恶意的操作，例如结束系统服务、拨打电话、删除程序数据等
```

具体的示意图如下所示：

![image-20211026112525986](upload/attach/202110/905443\_X7CSYK5YCA2RU3A.png)

应用程序B是一个拥有权限P1的应用程序，包含组件C1，C1组件是公开给外界，意为`export = true`，并且C1组件可以访问权限P1对应的系统资源，这样另外一个没有权限P1的应用程序A，可以构造`Intent`数据去启动应用程序B的C1组件，进而访问到P1权限对应的系统资源，拥有权限P1的应用程序B没有对权限合理的保护，引发了权限泄漏，没有权限P1的应用程序A利用这个漏洞获得了权限，导致了权限泄漏

### 2. 漏洞案例

#### （1）下载文件权限泄漏漏洞

**目标程序：**

配置文件：

```xml
<!--网络-->
<uses-permission android:name="android.permission.INTERNET" />
<!-- 读写文件 -->
<uses-permission android:name="android.permission.READ\_EXTERNAL\_STORAGE" />
<uses-permission android:name="android.permission.WRITE\_EXTERNAL\_STORAGE" />
<receiver
android:name=".Download"
android:enabled="true"
android:exported="true">
<intent-filter>
<action android:name="com.example.down" />
</intent-filter>
</receiver>
```

代码段：

```java
public class Download extends BroadcastReceiver {
@Override
public void onReceive(Context context, Intent intent) {
// TODO: This method is called when the BroadcastReceiver is receiving
// an Intent broadcast.
if(intent.getAction().equals("com.example.down")){
String url = intent.getExtras().getString("url");
String fileName = intent.getExtras().getString("fileName");
Toast.makeText(context,"Downlaod start!"+url+"---"+fileName,Toast.LENGTH\_LONG).show();
Log.i("test",url+fileName);
}
//throw new UnsupportedOperationException("Not yet implemented");
}
}
```

**攻击程序：**

```java
public class MainActivity extends AppCompatActivity {
@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity\_main);
Intent intent = new Intent();
intent.setAction("com.example.down");
intent.putExtra("url","https://www.baidu.com/");
intent.putExtra("filename","baidu");
sendBroadcast(intent);
}
}
```

**效果显示：**

测试样例：

![image-20211026135228985](upload/attach/202110/905443\_7U75RR65J9CQ53T.png)

攻击样例：

![image-20211026135354598](upload/attach/202110/905443\_M69EJHUBWXPXRRM.png)

我们可以很明显发现，我们可以通过进程B发送广播到进程A，从而调用进程A的下载功能，开始下载

#### （2）猎豹清理大师内存清理权限泄漏漏洞

这个案例是一个很早之前的漏洞，现在参考意义不大，不过大家可以借助其学习其原理即可

**漏洞描述：**

```
Android应用程序猎豹清理大师存在自定义的权限android.permission.RESTART\_PACKAGES，该权限主要用来结束进程来达到清理内存的目的，当其他没有申请此权限的app向目标程序发送相应的intent时，便可以结束后台运行的部分app进程
```

**漏洞实现：**

![image-20211026140524902](upload/attach/202110/905443\_W6UJZHZDHE58BUD.png)

![image-20211026140633874](upload/attach/202110/905443\_DH7G67RJ8ANNPVQ.png)

![image-20211026140609720](upload/attach/202110/905443\_QPFH293V9SQQJ9Z.png)

这就是很典型的我们利用个人定义的权限，来攻击目标程序，达到权限泄漏漏洞的目标

### 3. 漏洞防护

#### （1）组件设置不可导出

我们从上面的两个案例可以发现，Android的权限泄漏漏洞往往和四大组件相互关联，我们对一些重要组件设置不可导出，就可以达到一定的保护效果

#### （2）设置权限级别

我们可以对自定义的权限一定要设置相应的保护级别，从上文我们可以得出：

```
android:protectionLevel="normal"，不需要动态申请
android:protectionLevel="dangerous"，需要动态申请
android:protectionLevel="signature|signatureOrSystem",需要两个程序签名相同才能访问，这样就可以很好的避免权限泄漏的风险
```

## 四、Android默认设置介绍

在上文中已经向大家详细介绍了Android中的权限安全漏洞的情况，接下来我们进一步了解Android中的默认设置以及默认设置引起的安全问题，Android中的默认设置主要分为：`AndroidManifest.xml`配置文件和`WebView`的默认设置

![image-20211026144059511](upload/attach/202110/905443\_Z8H8HDY5NRK5RCN.png)

#### （1）AndroidManifest.xml默认设置

这里我总结了`AndroidManifest.xml`的基本结构图

![image-20211026144059511](upload/attach/202110/905443\_XVVS4CMA78AYKVV.png)

这里我们主要关注`allowBackup`、`Debuggable`、组件默认导出

##### <1> allowBackup

Android API Level 8及其以上Android系统提供了应用程序的备份和恢复功能，这是由`AndroidManifest.xml`文件中的`allowBackup`属性决定，`allowBackup` 标志为 true 时，用户即可通过 `adb backup` 和 `adb restore` 来进行对应用数据的备份和恢复

Android 属性`allowBackup` 安全风险源于`adb backup`容许任何一个人能够打开USB调试开关，从Android手机中复制应用数据到外设，一旦数据被备份之后，所有应用数据都可被用户读取。`adb restore` 容许用户指定一个恢复的数据来源（即备份的应用数据）来恢复应用程序数据的创建

**安全影响：**

```
通讯录应用，一旦应用程序支持备份和恢复功能，攻击者即可通过 adb backup 和 adb restore 进行恢复新安装的同一个应用来查看聊天记录等信息
支付金融类应用，攻击者可通过此来进行恶意支付、盗取存款等
```

**allowBackup安全风险详情：**

```
1.allowBackup 风险位置：AndroidMannifest.xml 文件 android:allowBackup 属性
2.allowBackup 风险触发前提条件：未将 AndroidMannifest.xml 文件中的 android:allowBackup 属性值设为 false
3.allowBackup 风险原理：当 allowBackup 标志值为 true 时，即可通过 adb backup 和 adb restore 来备份和恢复应用程序数据
```

**数据备份：**

```java
命令：adb backup [-system|nosystem] -all [-apk|-noapk] [-shared|-noshared] -f <档案名称>[需要备份的应用包名]
[-system|nosystem]
这个指令告诉adb在备份时是否要连同系统一起备份，若没有打开的话，默认是-system表示会一起备份系统，若连系统一起备份，在还原时候会覆盖系统档案，这对已经升级的手机是非常不好的，-nosystem是建议要打上去的指令
-all
这个指令除非要备份单一App不然是一定要打上去的，表示是否备份全部的App，若有加上`-nosystem`，就只会备份你目前已经安装上去的App而不会连系统App一起备份
[-apk|-noapk]
默认是-noapk，这个参数的意思是，是否连安装的apk一起备份，若为-noapk，则只会备份apk的资料档
[-shared|-noshared]
默认是`-noshared`，表示是否连手机存储空间或是SD卡的档案一起备份
-f
用这个来选择备份文件存储在哪里，例如-f /backup/applock.ab将会使文件存储在根磁盘（Windows的C盘等等）下一个名为backup的文件夹里，并且备份文件名为applock.ab
```

例如我们直接备份

```java
adb backup -apk -shared -system -all -f applock.ab
```

就会在电脑默认路径下产生`applock.ab`文件，大小与系统有关

**查看数据：**

一般我们使用`abe`工具来解析`ab`文件，工具参考网址：[abe工具]( <mark class="encrypted">89eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6F1k6h3I4W2L8X3E0G2N6W2)9J5c8X3q4F1k6s2u0G2K9h3c8Q4x3X3c8T1j5h3y4C8N6i4m8Q4x3X3c8W2P5s2c8J5j5h3y4@1L8%4u0Q4x3V1k6J5k6h3I4W2j5i4y4W2M7#2)9J5z5b7`.`.</mark>

解压命令：

```java
java -jar ade.jar unpack 1.ab 1.tar
```

**恢复数据：**

```java
adb restore applock.ab
```

##### <2> debuggable

`android:debuggable`属性是指定应用程序是否能够被调试，即时是以用户模式运行在设备上的时候，如果设置为 true ，则能够被调试，否则不能调试，默认值是false

逆向工作人员在对一个apk程序进行逆向时，第一步往往就是要对`debuggable`属性进行绕过，否则就不能进行正常的调试，因此如果属性本身设为true，更容易导致该应用存在动态调试，泄露数据信息的风险

![image-20211026164320202](upload/attach/202110/905443\_Y2PA28AWXWQBUQY.png)

##### <3> exported

前几篇帖子，我们讲述四大组件过程中，经常会涉及到`exported`导出导致的漏洞，`exported`组件导出，会导致导出的组件被第三方APP任意调用，导致敏感信息，并可能受到绕过认证、恶意代码注入等风险，由于前面我们已经讲得很详细了，这里就不再做过多描述

#### （2）WebView默认设置

`WebView`是android中用来展示网页的重要控件，Android中经常会使用`WebView`来实现Web页面的展示，在`Activity`中启动自己的浏览器或者简单的展示一些在线内容等，而`WebView`上的漏洞问题也各种各样，过去几年内`WebView`中被披露的重大漏洞包括了任意代码执行漏洞、跨域、密码明文保存等，后面我们将`WebView`作为一个专题讲述上面存在的漏洞问题，这里我们只关注`WebView`中默认设置的安全问题

后面我们主要讲述`WebView`上几方面的默认设置漏洞

![image-20211026165646971](upload/attach/202110/905443\_U4EWKHYPP5BTR2W.png)

## 五、Android默认设置漏洞原理分析和复现

### 1. allowBackup

#### （1）备份安全设置信息泄露漏洞

上文我们已经详细的给大家讲述了`allowBackup`设置`true`的安全性，接下来我拿一个案例带大家深入了解：

##### <1> 漏洞复现

样本：`sieve.apk`

**信息设置：**

首先，我们将样本安装到手机上，并打开主界面

![image-20211026194532102](upload/attach/202110/905443\_E87D7X264BUETV4.png)

![image-20211026194611643](upload/attach/202110/905443\_J9PHD5K2CWDGJC6.png)

进入主程序我们发现，提示我们注册密码，而且需要至少16位，这里我们输入密码`zxcvbnm0123456789`

进入后，又要我们输入PIN码，我们输入`1234`

![image-20211026195015970](upload/attach/202110/905443\_WZ9AJS36S937GV8.png)

然后我们发现需要再次输入密码才能进入，我们再次输入密码

![image-20211026195112662](upload/attach/202110/905443\_2AXNMD6EW3276RK.png)

发现右上角可以添加账户密码信息，我们添加，密码设置`123456`

![image-20211026195256415](upload/attach/202110/905443\_ZE3RQ8Q6UGF2C5R.png)

![image-20211026195314419](upload/attach/202110/905443\_KSZSD8DSRGJVNK5.png)

这样我们就在apk中就存入了一些信息，接下来我们来看如何备份和恢复这些信息呢

**数据备份：**

我们先用`jadx-gui`打开目标程序，查看`allowbackup=true`

![image-20211026195645086](upload/attach/202110/905443\_2GUXW4F9SSJUSAV.png)

这里发现是可以备份的，所以我们输入备份命令

```java
adb backup -f allowBackup.ab -noapk com.mwr.example.sieve
```

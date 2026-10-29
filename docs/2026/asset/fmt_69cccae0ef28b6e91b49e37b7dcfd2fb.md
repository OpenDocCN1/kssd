
# [原创]攻防世界Mobile通关记录_apk逆向12_RememberOther_Ph0en1x-100_ill-intentions_wp

## 前言

业余时间写的，纯兴趣更新!

## apk逆向

### 题目信息

运行apk，随便输入，点击确定，弹框错误。

![](upload/attach/202210/847319_AN9PT7W5HCB4D8Z.png)

### 分析

将其拖入jeb中，查看类`MainActivity`中的`onCreate`方法。

```java
public void onCreate(Bundle arg3) {
super.onCreate(arg3);
this.setContentView(0x7F040019);
this.setTitle(0x7F06001D);
this.edit_userName = "Tenshine";
this.edit_sn = this.findViewById(0x7F0C0051);
this.btn_register = this.findViewById(0x7F0C0052);
this.btn_register.setOnClickListener(new View$OnClickListener() {
public void onClick(View arg5) {
if(!MainActivity.this.checkSN(MainActivity.this.edit_userName.trim(), MainActivity.this.edit_sn.getText().toString().trim())) {
Toast.makeText(MainActivity.this, 0x7F06001E, 0).show();
}
else {
Toast.makeText(MainActivity.this, 0x7F06001B, 0).show();
MainActivity.this.btn_register.setEnabled(false);
MainActivity.this.setTitle(0x7F060019);
}
}
});
}
```

即调用了`checkSN`方法。参数1为 `"Tenshine"`，参数2为我们输入的字符串。

看下`checkSN`的实现(具体看代码中的分析)

```java
private boolean checkSN(String str1, String input_str) {
boolean v7 = false;
if(str1 != null) {
try {
if(str1.length() == 0) {
return v7;
}
if(input_str == null) {
return v7;
}
if(input_str.length() != 22) {
return v7;
}
MessageDigest v1 = MessageDigest.getInstance("MD5");
v1.reset();
v1.update(str1.getBytes());//对参数1进行md5加密
String v3 = MainActivity.toHexString(v1.digest(), "");//并赋值给v3
StringBuilder v5 = new StringBuilder();
int v4;
for(v4 = 0; v4 < v3.length(); v4 += 2) { //将v3密文每隔一位进行舍弃
v5.append(v3.charAt(v4));
}
if(!"flag{" + v5.toString() + "}".equalsIgnoreCase(input_str)) {
return v7; //然后拼接上flag{}再与我们的输入将进行比较。
}
}
catch(NoSuchAlgorithmException v2) {
goto label_40;
}
v7 = true;
}
return v7;
label_40:
v2.printStackTrace();
return v7;
}
```

### 解题

```python
#coding:utf8
import hashlib
str1=b"Tenshine"
str2=hashlib.md5(str1).hexdigest()
print(str2)#b9c77224ff234f27ac6badf83b855c76
flag=""
for i in range(len(str2)):
if(i%2==0):
flag+=str2[i]
#flag="flag{"+flag+"}"
print(flag)#bc72f242a6af3857
```

## APK逆向-2

### 题目信息

安装apk安装失败

![](upload/attach/202210/847319_YRUEMUB6EY98N6V.png)

将其拖入jadx发现`AndroidManifest.xml`清单文件没有内容

![](upload/attach/202210/847319_CW6JUAUCAFJXDGM.png)

![](upload/attach/202210/847319_BB8DDQB92MWC58P.png)

### 分析

应该是`AndroidManifest.xml`文件被做了手脚。将其提取出来进行修改。

`apktool`提取失败。

```java
>apktool d APK逆向-2.apk
I: Using Apktool 2.6.1 on APK逆向-2.apk
I: Loading resource table...
I: Decoding AndroidManifest.xml with resources...
Exception in thread "main" brut.androlib.err.RawXmlEncounteredException: Could not decode XML
at brut.androlib.res.decoder.XmlPullStreamDecoder.decode(XmlPullStreamDecoder.java:145)
at brut.androlib.res.decoder.XmlPullStreamDecoder.decodeManifest(XmlPullStreamDecoder.java:151)
at brut.androlib.res.decoder.ResFileDecoder.decodeManifest(ResFileDecoder.java:159)
at brut.androlib.res.AndrolibResources.decodeManifestWithResources(AndrolibResources.java:193)
at brut.androlib.Androlib.decodeManifestWithResources(Androlib.java:141)
at brut.androlib.ApkDecoder.decode(ApkDecoder.java:109)
at brut.apktool.Main.cmdDecode(Main.java:175)
at brut.apktool.Main.main(Main.java:79)
Caused by: java.io.IOException: Expected: 0x001c0001, got: 0x01001c00
at brut.util.ExtDataInput.skipCheckChunkTypeInt(ExtDataInput.java:72)
at brut.androlib.res.decoder.StringBlock.read(StringBlock.java:50)
at brut.androlib.res.decoder.AXmlResourceParser.doNext(AXmlResourceParser.java:814)
at brut.androlib.res.decoder.AXmlResourceParser.next(AXmlResourceParser.java:98)
at brut.androlib.res.decoder.AXmlResourceParser.nextToken(AXmlResourceParser.java:108)
at org.xmlpull.v1.wrapper.classic.XmlPullParserDelegate.nextToken(XmlPullParserDelegate.java:105)
at brut.androlib.res.decoder.XmlPullStreamDecoder.decode(XmlPullStreamDecoder.java:138)
... 7 more
```

把apk后缀改成zip，将其拉出来。

找三个正常的清单文件(apk逆向，app1，app2),查看它们文件头哪些是共同的地方。

![](upload/attach/202210/847319_42NHY3JMQCXNTG9.png)

![](upload/attach/202210/847319_FARXAX6UJPY3Q2E.png)

![](upload/attach/202210/847319_U33XS3CFZDFRBY5.png)

```
第一行:偏移0x0-0x3 0x6-0xB 0xE-0xF
第二行:偏移0x1-0xB 0xE-0xF
第三行:偏移0x1-0xF
第四行:偏移0x1-0x7 0x9-0xB 0xD-0xF
```

然后再查看下本题

![](upload/attach/202210/847319_V84A59ZE422SRMV.png)

```
第一行:偏移0x0-0x3(一致) 0x6-0xB(不一致 0x8-0xB刚好反转) 0xE-0xF(一致)
第二行:偏移0x1-0xB(一致) 0xD-0xF(一致)
第三行:偏移0x1-0xF(不一致 0x2位置变成1了)
第四行:偏移0x1-0x7(一致) 0x9-0xB(一致) 0xD-0xF(一致)
```

所以这里我们将不一致的地方改为一致，即第一行的偏移`0x6-0xB`位置和第三行的偏移为偏移`0x1-0xF`

![](upload/attach/202210/847319_89U3RBM747F627H.png)

将其再覆盖掉apk中的清单文件，然后便可以安装成功了.

但安装成功后,却没有显示桌面图标.使用`am`进行打开`Mainactivity`界面

```
am start com.example.mmsheniq/.Mainactivity
```

![](upload/attach/202210/847319_FBQMNF5H7CB9ZTU.png)

同时看下`logcat`

![](upload/attach/202210/847319_GT7XC8B645HPFW9.png)

先不点安装，点空白处。

![](upload/attach/202210/847319_U84CFDAMHBHPA7P.png)

点击注册

![](upload/attach/202210/847319_6UMNJKGTJTFA3DF.png)

这里填过信息后点注册一直报错"请输入正确的身份证"。

本想好好分析下代码，可我懒死了.

### 解题

修复过`AndroidManifest.xml`文件后拖入jadx中，再看下清单文件中的内容。发现可疑的字符串，其实就是flag。

```
8d6efd232c63b7d2
```

![](upload/attach/202210/847319_PSC276EYCSTXAYH.png)

## RememberOther

### 题目信息

附件是个zip压缩包，解压后

![](upload/attach/202210/847319_YYFEEYVHJXCYW22.png)

查看`FoundItFun.docx`，发现只有一句话。

![](upload/attach/202210/847319_TUM2ERS3R2SV378.png)

### 分析

jadx

```java
this.btn_register.setOnClickListener(new View.OnClickListener() { // from class: com.droider.crackme0201.MainActivity.1
@Override // android.view.View.OnClickListener
public void onClick(View v) {
if (!MainActivity.this.checkSN(MainActivity.this.edit_userName.getText().toString().trim(), MainActivity.this.edit_sn.getText().toString().trim())) {
Toast.makeText(MainActivity.this, (int) R.string.unsuccessed, 0).show();
return;
}
Toast.makeText(MainActivity.this, (int) R.string.successed, 0).show();
MainActivity.this.btn_register.setEnabled(false);
MainActivity.this.setTitle(R.string.registered);
}
});
```

水啊水啊，如果`checkSN`方法返回true，则注册成功。

`checkSN`

```java
public boolean checkSN(String userName, String sn) {
try {
if (userName.length() == 0 && sn.length() == 0) {//什么都不输入就返回true 哈哈
return true;
}
if (userName == null || userName.length() == 0) {
return false;
}
if (sn == null || sn.length() != 16) {
return false;
}
MessageDigest digest = MessageDigest.getInstance("MD5");
digest.reset();
digest.update(userName.getBytes()); //对用户名进行md5加密
byte[] bytes = digest.digest();
String hexstr = toHexString(bytes, BuildConfig.FLAVOR);//BuildConfig.FLAVOR为空字符串
StringBuilder sb = new StringBuilder();
for (int i = 0; i < hexstr.length(); i += 2) {
sb.append(hexstr.charAt(i)); //隔一位取一位
}
String userSN = sb.toString();
return userSN.equalsIgnoreCase(sn); //用户名经过上面的加密后就再与输入的注册码sn相等即返回true
} catch (NoSuchAlgorithmException e) {
e.printStackTrace();
return false;
}
}
```

什么都不输入就会返回true的，返回`successed`对应的字符串。

```
<string name="successed">md5:b3241668ecbeb19921fdac5ac1aafa69</string>
```

对其进行解密得到

```
YOU_KNOW_
```

<a href="elink@d7eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6L8$3#2V1y4g2)9J5k6h3y4G2L8g2)9J5c8R3`.`.">MD5免费在线解密破解_MD5在线加密-SOMD5</a>

![](upload/attach/202210/847319_78HQZQJ74WNG278.png)

### 解题

但最终的flag需要在后面再添加上`ANDROID`即

```
YOU_KNOW_ANDROID
```

脑洞，无聊。

![](upload/attach/202210/847319_ZTHTSEU3WJYGPKF.png)

## Ph0en1x-100

### 题目信息

![](upload/attach/202210/847319_2Q9SM8TD2N6Z8C2.png)

### 分析

jadx

```java
public void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);
this.etFlag = (EditText) findViewById(R.id.flag_edit);
}
public void onGoClick(View v) {
String sInput = this.etFlag.getText().toString();
if (getSecret(getFlag()).equals(getSecret(encrypt(sInput)))) {
Toast.makeText(this, "Success", 1).show();
} else {
Toast.makeText(this, "Failed", 1).show();
}
}
```

如果`getSecret(getFlag())`的返回值和`getSecret(encrypt(sInput)))`相等就证明Success。

即`getFlag()`的返回值和`encrypt(sInput))`返回值相等，就证明Success。

这这两个函数都是在`phcm`的so库中实现的。

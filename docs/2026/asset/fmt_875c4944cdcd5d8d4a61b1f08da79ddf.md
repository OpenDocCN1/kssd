
# [原创] Android 简单加密壳

## APK 加固

### 参考

<mark class="encrypted">a7cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0p5#2M7r3u0Q4x3X3g2U0L8$3#2Q4x3X3g2U0L8W2)9J5c8X3#2&6i4K6u0r3j5$3!0#2M7Y4y4W2i4K6u0r3z5e0p5#2</mark>

https://www.kanxue.com/book-section_list-73.htm

## 开发中常用的加固手段

在开发阶段就嵌入安全代码&验证&混淆， 主要有以下几种方式

* `proguard`配置项(`release`版本默认开启) 对apk中源码进行混淆
* 对Apk反编译之后的`smali`进行混淆
* 对Apk中的字符串进行加密
* 对Apk中的文件进行校验

![image-20220415104418372](upload/attach/202208/890847_U3CHH5B3H7QBXXS.jpg)

### proguard混淆

* 开启之后会将系统库名&类名全部修改为随机字符串

![image-20220417170224448](upload/attach/202208/890847_U95AZW8Y789GPUC.jpg)

* `release`默认开启， 在项目目录`app`下的`build.gradle`中有配置信息`minifyEnabled`

`build.gradle(module)`配置 `minifyEnabled:true`

```
buildTypes {
release {
minifyEnabled true
proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
}
debug {
minifyEnabled true
proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
}
}
```

配置完后进行编译， 反编译就能观察到变化

`release` 版本生成: Build--> Generate Signed Bundle/APK --> APK 生成

* Proguard的混淆结果会输出到 `<module-name>/build/outputs/mapping/release或者debug/` 中的3个文件和一个混淆配置文件

![image-20220417164326902](upload/attach/202208/890847_596A3ZCD8RDZHP8.jpg)

* `mapping.txt`: 提供混淆前后类名&方法名&成员变量名的对应关系
* `seeds.txt`: 列出没有被混淆的类和成员
* `usage.txt`: 列出从APK中移除的代码（没有使用到）

`proguard`包括四个功能，`shrinker`（压缩）, `optimizer`（优化）,`obfuscator`（混淆）,`preverifier`（预校验）。

- `shrink`： 检测并移除没有用到的类，变量，方法和属性；
- `optimize`: 优化代码，非入口节点类会加上`private`/`static`/`final`, 没有用到的参数会被删除，一些方法可能会变成内联代码。
- `obfuscate`: 使用短又没有语义的名字重命名非入口类的类名，变量名，方法名。入口类的名字保持不变。
- `preverify`: 预校验代码是否符合Java1.6或者更高的规范(唯一一个与入口类不相关的步骤)

想要自定义`proguard`混淆可以在 `proguard-rules.pro`文件中

### smali代码乱序

通过对`smali`代码进行乱序来干扰代码逆向

```
goto :sign1
:sign3
指令3
goto :end
:sign2
指令2
goto :sign3
:sign1
指令1
goto :sign2
:end
```

**实例**

新建一个工程, 新建类`Hello.java`

```
public class Hello {
public static void main(String[] argc){
// 字符串的声明和定义
String a="1";
String b="2";
// 字符串拼接
String c = a+b;
// 打印字符串
System.out.println(c);
}
}
```

![image-20220417220751389](upload/attach/202208/890847_KMRQBZSQV33T6FJ.jpg)

> 禁用`mutidex`

>
> ```
> android {
> compileSdkVersion 21
> buildToolsVersion "21.1.0"
>
> defaultConfig {
> ...
> minSdkVersion 14
> targetSdkVersion 21
> ...
>
> // Enabling multidex support.
> multiDexEnabled false
> }
> ...
> }
> ```

编译生成APK

使用`Android Killer`(或者使用`apktool`)打开`apk`, 打开解压后的`smali`工程

```
java -jar apktool_latest.jar d app-debug.apk
```

观察`Hello.smali`代码如下(使用反汇编工具如`jd`和`jadx`也能够正常反汇编)

```
.class public Lcom/example/myapplication/Hello;
.super Ljava/lang/Object;
.source "Hello.java"
# direct methods
.method public constructor <init>()V
.locals 0
.line 3
invoke-direct {p0}, Ljava/lang/Object;-><init>()V
return-void
.end method
.method public static main([Ljava/lang/String;)V
// 字符串的声明和定义
.locals 4
.param p0, "argc" # [Ljava/lang/String;
.line 5
const-string v0, "1"
.line 6
.local v0, "a":Ljava/lang/String;
const-string v1, "2"
// 字符串拼接
.line 7
.local v1, "b":Ljava/lang/String;
new-instance v2, Ljava/lang/StringBuilder;
invoke-direct {v2}, Ljava/lang/StringBuilder;-><init>()V
invoke-virtual {v2, v0}, Ljava/lang/StringBuilder;->append(Ljava/lang/String;)Ljava/lang/StringBuilder;
invoke-virtual {v2, v1}, Ljava/lang/StringBuilder;->append(Ljava/lang/String;)Ljava/lang/StringBuilder;
invoke-virtual {v2}, Ljava/lang/StringBuilder;->toString()Ljava/lang/String;
move-result-object v2
// 打印字符串
.line 8
.local v2, "c":Ljava/lang/String;
sget-object v3, Ljava/lang/System;->out:Ljava/io/PrintStream;
invoke-virtual {v3, v2}, Ljava/io/PrintStream;->println(Ljava/lang/String;)V
.line 9
return-void
.end method
```

代码分为三部分, 对这三部分进行乱序

1. 字符串的声明和定义
2. 字符串拼接
3. 打印字符串

![image-20220417223420422](upload/attach/202208/890847_QQAUAMFPEGETX4Y.jpg)

对`smali`代码进行乱序之后，重新编译, 在`app-debug/build/dist/`目录下生成`apk`

```
java -jar apktool_latest.jar b app-debug
```

对`smali`代码进行乱序后，重新编译

![image-20211127184447788](upload/attach/202208/890847_4D7JBSBG6GMNWNS.jpg)

然后`dex2jar`就不能用了，但是`jadx`可以识破

### 对APK字符串进行加密

为了应对逆向分析中常用的字符串分析， 对使用到的字符串进行加密， 主要有如下方法：

* 编码混淆
* 加密处理, 在使用时才动态解密字符串

**普通方式定义字符串**

```
public class Hello {
public static void main(String[] argc){
String str = "Hello World";
System.out.println(str);
}
}
```

反编译的`smali`代码

```
.method public static main([Ljava/lang/String;)V
.registers 3
.param p0, "argc" # [Ljava/lang/String;
.line 5
const-string v0, "Hello World"
.line 6
.local v0, "str":Ljava/lang/String;
sget-object v1, Ljava/lang/System;->out:Ljava/io/PrintStream;
invoke-virtual {v1, v0}, Ljava/io/PrintStream;->println(Ljava/lang/String;)V
.line 7
return-void
.end method
```

可以看到 `"Hello World"` 被直接内联到代码里面

**编码混淆**

```
public class Hello {
public static void main(String[] argc){
byte[] strBytes = {0x48, 0x65, 0x6c, 0x6c, 0x6f, 0x57, 0x6f, 0x72, 0x6c, 0x64};
String str = new String(strBytes);
System.out.println(str);
}
}
```

反编译的`smali`代码

```
.method public static main([Ljava/lang/String;)V
.registers 4
.param p0, "argc" # [Ljava/lang/String;
.line 5
const/16 v0, 0xa
new-array v0, v0, [B
fill-array-data v0, :array_12
.line 6
.local v0, "strBytes":[B
new-instance v1, Ljava/lang/String;
invoke-direct {v1, v0}, Ljava/lang/String;-><init>([B)V
.line 7
.local v1, "str":Ljava/lang/String;
sget-object v2, Ljava/lang/System;->out:Ljava/io/PrintStream;
invoke-virtual {v2, v1}, Ljava/io/PrintStream;->println(Ljava/lang/String;)V
.line 8
return-void
:array_12
.array-data 1
0x48t
0x65t
0x6ct
0x6ct
0x6ft
0x57t
0x6ft
0x72t
0x6ct
0x64t
.end array-data
.end method
```

### 对apk中的文件进行校验

* 一般对`apk`中的`dex` & 签名 & `apk`本身进行校验
* 这种校验办法最终还是通过返回 `true/false`， 容易被暴力破解

#### 对apk中的dex文件进行校验

在演示过程中可以直接创建`crc_value`资源保存`dex`中的`crc`，然后与`dex`文件中的`crc`进行比对 在实践应用中可以从服务端远程获取`crc`校验值进行比较。

```
/*校验Dex CRC值*/
private void verifyDex(){
// 获取String.xml中的value, 实践中应该联网获取用于比对的CRC值
Long dexCrc = Long.parseLong(this.getString(R.string.crc_value));
String apkPath = this.getPackageCodePath();
Log.d(TAG, "verifyDex: PackageCodePath: " + apkPath);
try {
ZipFile zipFile = new ZipFile(apkPath);
ZipEntry dexEntry = zipFile.getEntry("classes.dex");
// 计算classes.dex的crc
long dexEntryCrc = dexEntry.getCrc();
Log.d(TAG, "verifyDex: dexEntryCrc: "+ dexEntryCrc);
if(dexCrc == dexEntryCrc){
Log.d(TAG, "dex has not been modified'");
}else{
Log.d(TAG, "dex has been modified");
}
}catch (Exception e){
e.printStackTrace();
}
}
```

#### 对apk中的apk进行校验

与`dex`校验不同， `apk`检验必须把计算好的`Hash`值放到网络服务器, 因为对APK的任何改动都会影响到最后的`Hash`值

```
/*校验APK MD5值
*与dex校验不同， apk检验必须把计算好的Hash值放到网络服务器, 因为对APK的任何改动都会影响到最后的Hash值
* */
private void verifyApk(){
// /data/app/com.example.myapplication-1/base.apk
String apkPath = this.getPackageCodePath();
MessageDigest msgDigest;
try {
// 获取apk并计算MD5值
msgDigest = MessageDigest.getInstance("MD5");
byte[] bytes = new byte[4096];
int count;
FileInputStream fis = new FileInputStream(new File(apkPath));
while((count = fis.read(bytes)) > 0){
msgDigest.update(bytes, 0, count);
}
// 计算出MD5值
BigInteger bInt = new BigInteger(1, msgDigest.digest());
String md5 = bInt.toString(16);
fis.close();
Log.d(TAG, "verifyApk: md5: "+ md5);
// 与服务端的MD5值进行对比
// code ....
}catch (Exception e){
e.printStackTrace();
}
```

#### 对apk中的签名进行校验

* 每个APK都会经过开发者的证书签名， 如果破解者对APK进行二次打包会用自己的签名
* 打包。必须要在在服务端存储MD5， 因为签名中含有对文件的hash

```
/*获取签名的Md5*/
public void verifySignature(){
String packageName = this.getPackageName();
PackageManager pm = this.getPackageManager();
PackageInfo pi;
String md5 = "";
try{
pi = pm.getPackageInfo(packageName, PackageManager.GET_SIGNATURES);
Signature[] s = pi.signatures;
// 计算出MD5值
MessageDigest msgDigest = MessageDigest.getInstance("MD5");
msgDigest.reset();
msgDigest.update(s[0].toByteArray());
BigInteger bInt = new BigInteger(1, msgDigest.digest());
md5 = bInt.toString();
}catch(Exception e){
e.printStackTrace();
}
Log.d(TAG, "verifySignature: md5" + md5);
// 与服务端的MD5值进行对比
// code ....
}
```

签名校验工具: `keytool`, 解压`apk/META_INF/CERT.RSA`

```
keytool -printcert -file CERT.RSA
```

## Android加固-壳

Android安全人鱼对APK加固主要有以下几个方面
